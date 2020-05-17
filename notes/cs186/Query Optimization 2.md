# Query Optimization: System R Passes 2-n

Now that we have accessed (read from disk) all the base tables, we want to find optimal ways to join them together. These are handled with the rest of the passes of System R.

**At pass $i$, we try to join $i$ tables, using the tables from pass $i-1$ and pass 1**.

For example, pass 2 tries to find good ways to join two (base) tables together, both from pass 1. Pass 5 tries to join 5 tables: 4 from pass 4, and one from pass 1, adhering to the left-deep plan heuristic. **We always try to join joined tables with *one* base table.**

We're joining $i$ tables at pass $i$: thus, we want to consider all length- $i$ sets of tables that don't need a cross join to join- i.e. sets of tables with a join condition. Thus, pass $i$ will produce *at least* one **join plan** for each of these sets. Identical to pass 1, we advance optimal and optimal interesting join plans we find. 

System R also considers each possible join algorithm the database can implement. For example, in pass 2, we might see plan 1 as A CNLJ B and plan 2 as A SMJ B. Recall that sort merge join's output table is sorted on the columns in the join condition. Thus, only plans where SMJ is used for the last join in the set can possibly produce an interesting order, are only possible if SMJ is used for the last join in the set. 

## Example: Passes 2 and 3

### Pass 2

Let's say we have this query. Assume pass 1 is done, and we have accessed all our base tables. 

```sql
SELECT *
FROM A INNER JOIN B
ON A.aid = B.bid
INNER JOIN C
ON b.did = c.cid
ORDER BY c.cid;
```

On to pass 2. We want all length-2 sets of tables joinable WITHOUT cross joins. Only joins of $\{A,B\}$ and $\{B,C\}$ will be considered: we never consider joining $\{A,C\}$ because there isn't a join condition and we'd need a cross join. For simplicity, assume only SMJ and CNLJ are implemented, and pass 1 *only* returned a full scan for each table. 

Say System R returns these join plans for pass 2:

1. A CNLJ B (estimated cost: 1000)
2. B CNLJ A (estimated cost: 1500)
3. A SMJ B (estimated cost: 2000)
4. B CNLJ C (estimated cost: 800)
5. C CNLJ B (estimated cost: 600)
6. C SMJ B (estimated cost: 1000)

Which plans advance? 

Well, first we want the optimal join plan *for each length-i set of tables*. For $\{A,B\}$ we see this is plan 1. For $\{B,C\}$ we see this is plan 5.

We also want the optimal interesting plans. We look at all SMJ plans, and see that only plan 6 is interesting because it will sort on c.cid, which is used in a later ORDER BY. 

Thus, joins 1, 5, and 6 will advance to pass 3.

### Pass 3

Same logic as before. Now, though, we want to consider all possible ways to join THREE tables without cross joins: in particular, all ways to join 2 tables with a third remaining base table.

Again, the query for reference: 	

```sql
SELECT *
FROM A INNER JOIN B
ON A.aid = B.bid
INNER JOIN C
ON b.did = c.cid
ORDER BY c.cid;
```

Now, there's only one length-3 set of tables to consider join plans for: $\{A,B,C\}$. 

Pass 3 Plans:

1. Join 1 {A, B} CNLJ C (estimated cost: 10,000)
2. Join 1 {A, B} SMJ C (estimated cost: 12,000)
3. Join 5 {B, C} CNLJ A (estimated cost 8,000)
4. Join 5 {B, C} SMJ A (estimated cost: 20,000)
5. Join 6 {B, C} CNLJ A (estimated cost: 22,000)
6. Join 6 {B, C} SMJ A (estimated cost: 18,000)

Note order is now important because we are only considering left deep plans, so base tables must be on the right.

Considering IO cost optimality, Plan 3 will advance. However, we still want to advance optimal interesting joins. Because plan 2 SMJs with C, sorting cid in output, which is used in a later ORDER BY, that is interesting and will advance. 

Thus plans 2 and 3 will advance, and we've finished the system R join passes for this query.