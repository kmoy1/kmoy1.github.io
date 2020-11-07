# Query Optimization 

**Query Optimization** aims to find the query plan that minimizes the number of IOs needed to execute it. Recall that a query plan is simply some sequence of operations that will get us the correct result (relation) for a query.

[Insert Sailors, Reserves query plan]

For example, the above query plan consists of, in order, a join, then row selection, then column projection.

## Selectivity Estimation

**We don't every know the true IO cost of a query plan until it's executed.** This means 2 things:

1. No guarantee we will find the optimal query plan - must use heuristics and **estimations** to (hopefully) find a good enough one.
2. We need a way estimate query plan IO cost.

**Selectivity estimation** is a tool used in estimating a query plan’s cost. Basically, operators are given **selectivities** based on (approximately) what percentage of pages make it "through" the operator onto the operator above it. Naturally, operators with very low selectivities (very few percentage of pages make it through) will probably want to be applied first in our optimal query plan (since future operators will work on fewer pages).

Some important selectivity estimation formulas below:

- $X=a$: 1/(unique vals in X)
- $X=Y$: 1/max(unique vals in X, unique vals in Y)
- $X>a$: (max(X) - a) / (max(X) - min(X) + 1)
- cond1 AND cond2: Selectivity(cond1) * Selectivity(cond2)

### Selectivity of Joins

Consider joining $A$ and $B$ on A.id = B.id. A conditionless join is just a cartesian product, which just joins every row from $A$ with every row from $B$, producing a total of $[A][B]$ pages. We can treat the join condition as just a simple AND $X=Y$ from the formulas above. Thus, the selectivity for a join on a common column id is:

$$\frac{1}{\text{max(unique vals for A.id, unique vals for B.id)}}$$

And thus the approximate total number of pages that make it through the join:

$$\frac{[A][B]}{\text{max(unique vals for A.id, unique vals for B.id)}}$$

## Common Heuristics

**Heuristics** are just rules that we follow in cutting down possible plans. Here are the big 3:

1. Push down projects (π) and selects (σ) as far as they can go
2. Only consider left deep plans
3. Do not consider cross joins *unless* they are the only option.

### Pushing Down Projects and Selects

Why push down projects and selects as far down the plan? In other words, why make projection and selection happen as early as possible? 

We know pushing down selection reduces the number of pages the higher operators deal with, since selection always cuts down records which make up pages. Less records always means less pages.

What about pushing down projection, though? Note projection doesn't reduce the number of records, but instead reduces the *size* of the records, by eliminating columns! Thus we can fit more records on a page, and thus there are fewer pages! (Remember from a few notes back that pages just squeeze records together for the most part). Unfortunately, projection (unlike selection) can't be pushed down freely: for example, we can't eliminate a column that's used in a SELECT or WHERE clause higher up in the plan.

### Consider only Left-Deep Plans

A left deep plan is a plan where **all right tables in any join are base tables.**

Why ONLY consider them? Two main reasons:

1. Greatly reduces plan space (number of plans to consider)
2. **Pipelineable**: Pages in a left deep plan can be pipelined, which means that joins do NOT have to be written to disk, and instead just passed up to the next (join) operator. So we save a ton of IOs. 

### No Cross Joins

Cross joins generally output a ton of pages which makes higher-up operators perform many IOs. We want to avoid that whenever possible.

## System R

The **System R query optimizer** uses all of the heuristics that we mentioned in the previous section, and consists of two passes. 

### Pass 1: Accessing Base Tables

The first pass of System R determines how to access tables **optimally** or **interestingly**. Optimal table accesses are simply means of accessing tables (from disk) that minimize IO costs. Interesting table accesses are means of accessing tables (from disk) such that the table(s) are **sorted** on a column either used in a future GROUP BY, ORDER BY, or another join.

To access a table during pass 1, we can either perform a full scan OR an index scan *for each index built on that table*. If we have a pushed-down SELECT operation, either scan will also only return rows only if they match the SELECT condition specified. Because we're only reading *individual* tables right now, conditions involving multiple tables (join conditions) are skipped in pass 1. 

Full scans always take $[P]$ IOs for table $P$. 

Index scans, however, depend on two things:

- How records are stored (alt 1/2/3 index)
- Index clustered or not

### Alt-1 Index Scan

**Alternative 1 index IO cost: (height of index (or cost to reach level above leaf)) + (num leaf pages read)**

Remember that in alt-1 indexes, leaf pages contain the records themselves. **These records are sorted in index leaf pages**, so we don't have to read *all* leaves: we can just find our starting point and scan right. (NOTE: Leaf and leaf page is basically the same thing).

### Example:  Alternative 1 Index Scan Cost

Let's say we have a table $A$ with $[A]$ pages. We have an height-2 alt-1 index built on A's column $C1$. Our SQL query has TWO conditions on $A$: $C1 > 5$ and $C2 < 6$ . Let's say the range of $C1$ and $C2$ was $[1,10]$. 

We want to find out (approximately) the IO cost of an index scan (on $C1$) to access $A$. 

First, we notice that we have a select condition, since it is a condition on individual columns of a table. This is pushed down and handled first, by heuristic 1. We first calculate the selectivity of this SELECT condition: (0.5)*(0.5) = 0.25. 

Here is the key part. We can't use $C2$'s condition to narrow down what pages we look at in our index because the index is only built on $C1$! On a full scan we would consider both, because our table is "built on" all columns! So we have a "true" selectivity of 0.5, contributed to only by $C1$. Thus the number of leaf pages read is $0.5[A]$. We also need to traverse down $2$ index pages to find our starting leaf. 

### Alt-2/3 Index Scan

Remember that leaf pages now contain **pointers** to data pages. 

For **alt-2 OR alt-3 indexes, IO cost: (height of index) + (num of leaf pages read) + (num of data pages read).**

We apply selectivity to reduce BOTH leaf pages and data pages read. Here, clustering/unclustering of the index(es) also matters. Recall that a clustered index sorts the data pages (records) on the same condition the index was built on: unclustered indices do not bother with this. Thus with a clustered index, we can actually apply selectivity condition on the sorted data pages to reduce the number of data pages read to selectivity * # DPs. Unclustered indices have to read every record (*selectivity still), since they're scattered unordered all over the data pages. 

### Example:  Alternative 2 Index Scan Cost

Let's say we have a table $B$ with $[B]$ pages and $|B|$ records. We have an height-2 alt-2 index built on B's column $C1$, and this index has $[L]$ leaf pages. Our SQL query has TWO conditions on $B$: $C1 > 5$ and $C2 < 6$. Range of $C1$ and $C2$ stays at $[1,10]$. 

Again, we want to find the IO cost of an index scan (on $C1$) to access $B$. Again, since our index is only built on $C1$, we are forced to ignore the selectivity of $C2$. 

If index clustered, IO Cost = $2 + 0.5[L] +0.5[B]$ . We traverse 2 index nodes, then read $0.5[L]$ leaf pages, then $0.5[B]$ pages. If index unclustered, IO Cost = $2 + 0.5[L] +0.5|B|$ . We traverse 2 index nodes, then read $0.5[L]$ leaf pages, then $0.5|B|$ records.

## Pass 1 Final Step: Survival of the Fittest

After we calculate all the possible access plans for a table in our query, we have to decide which one advances to the next pass (to complete the rest of the query). Remember that we want **optimal** plans (in terms of IOs) OR **optimal interesting** plans. Interesting plans are the set of plans with an **interesting order**, of which output tables satisfy two criteria:

- Table is sorted
- Table is sorted on column used in a *future* ORDER BY, GROUP BY, or join in the query.

Of all these interesting plans, we want to consider (and thus advance) the one that minimizes IOs. Obviously, getting a table already sorted on a column used later in a GROUP BY/ORDER BY will save *massive* amounts of IOs- only a single pass of the table is needed. Tables sorted on a column used in later joins are also valuable because (if) we use sort merge join for this later join, the "sort" process is already done and we save IOs. Full scans are never interesting: its output is almost always unsorted. Index scans sort output, however, by the index build column- so it definitely can produce interesting tables depending on what the index col is and if it's used later in the query.

### Example: Pass 1

Let's say we had this query: 

```sql
SELECT * FROM players 
INNER JOIN teams ON players.teamid = teams.id 
ORDER BY fname;
```

The first pass tries to find good plans to access *both* the players and teams table. Let's say System R produced these access patterns in pass 1:

1. Full Scan players (100 IOs) 
2. Index Scan players.age (90 IOs) 
3. Index Scan players.teamid (120 IOs) 
4. Full Scan teams (300 IOs) 
5. Index Scan teams.record (400 IOs)

Which ones advance? 

Well, first we find the optimal plan for players and teams. This is plan 2 for players, where we have players sorted by player age, and plan 4 for teams, where we just full scan it. 

We also advance the optimal interesting plan for each table. For players, this is plan 3, because we use players.teamid in a later join. We would also advance a plan that sorted teams by id, but that isn't given. 

Thus, plans 2, 3, and 4 advance.



So that's it for pass 1: accessing base tables optimally or interestingly. The rest System R's passes handle finding good plans to join the tables together. Read about it in note 2.



