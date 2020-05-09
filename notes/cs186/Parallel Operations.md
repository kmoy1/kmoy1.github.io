# Parallel Operations

Now that we've understood the basics of parallelism and its purpose (SPEED), let's enhance the sorting/hashing/join algorithms we know with parallelism. 

## Parallel Sorting

There are (three) steps for parallel sorting: 

1. Range partition the table's records => machines
2. Locally sort on each machine
3. Concatenate (MERGE) everything to make a fully sorted table!

## Parallel Hashing

Incredibly similar to its sorting counterpart, the steps:

1. Hash partition the table's records => machines
2. Locally hash on each machine
3. Concatenate (MERGE) everything to make a full hash table. 

## Parallel Sort-Merge Join

Remember that sort-merge join on tables $R$ and $S$ consisted of sorting $R$ and $S$ then using markers to join all matching records in sorted $S$ for each record in $R$, creating a fully sorted joined table. 

Parallelism will speed things up. Steps:

1. Using the **same ranges as join column**, range partition records of $R$, $S$. This is just to ensure matching records in $R$ and $S$ go to the same machine to be joined. 
2. Local SMJ on each machine
3. Concatenate sorted mini-joins to produce full SMJ table

## Parallel Grace Hash Join

Steps:

1. Using the same hash function $H_f$ for *both* tables, hash partition $R,S$ records ON THE JOIN COLUMN (again, preserving that matching records => same machine)
2. Local GHJ on each machine.
3. Concatenate mini-joins to produce full GHJ table

## Broadcast Join

If we have a *massive* table and want to join it with a relatively miniscule table, **broadcast join** might be the way to go. 

Let’s say $R$ is a 1,000,000 page table, round robin partitioned. Let's say $S$ is a 100 page table that is not partitioned (all on a single machine). We could do one of the parallel join algorithms discussed above, but those require partitioning *all* records of both $R$ and $S$. Given the size of $R$, we don't really want to do that (massive network cost).

A broadcast join solves this problem because we send a copy of the smaller relation ($S$) to *every* machine, i.e. ALL the records of $S$ go to every machine. Then, each machine does a local join, and concatenates, as usual. Even though each machine might be doing more work compared to if we did parallel SMJ/GHJ, the point is we **avoid partitioning a fatass relation over the network**. So we save a bunch of IOs. 

## Symmetric Hash Join

All the join algorithms discussed above share an unfortunate quality: they are all **pipeline breakers**. Not only do these joins take a while, but there's no "streaming" of records allowed: the only valid output is the fully joined table, which means that *every single record* of $R$ and $S$ must be processed before output. This prevents the wonderful pipeline parallelism we discussed in the previous note. 

However, there might be a solution with **Symmetric Hash Join**! Let's say we want to join $R$ and $S$ again.

1. Build 2 hash tables $H_R$ and $H_S$ from records in $R$ and $S$. 
2. For each record $r_i$ in $R$, probe $H_S$ for all matches and output any found. For each record $s_j$ in $S$, probe $H_R$ for all matches and output any found.
3. Add record to corresponding hash table. 

Here, every output tuple will be generated exactly once: when the second matching tuple arrives! It doesn't matter which record arrives first. Thus, SHJ is not a pipeline breaker.

## Hierarchical Aggregation

**Hierarchical aggregation** is the parallelism of aggregation operations (i.e. SUM, COUNT, AVG). 

To parallelize COUNT, each machine individually counts their records. The machines all send their counts to another machine which sums all sums for final sum.

Parallelizing AVG is a little trickier: the average of a bunch of averages isn’t necessarily the average of the data set. To parallelize AVG each machine must calculate the sum of all the values and the count. They then send these values to the coordinator machine. The coordinator machine adds up the sums to calculate the overall sum and then adds up the counts to calculate the overall count. It then divides the sum by the count to calculate the final average.

