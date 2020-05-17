# XJOIN

XJoin proceeds in three stages, each of which is performed by a separate thread.

- Stage 1: join records in memory hash tables (same as symmetric hash join)
- Stage 2: join records flushed to disk (due to memory constraints)
- Stage 3: (Started after all input records processed): Cleanup, which joins the relations created by stage 1 and 2

## Stage 1: In-Memory Joins

In XJoin, the records are organized in **partitions**. Each partition $P_X$ (partitions with hash value $X$) actually has two representative locations in memory *and* disk. Of course, $R$ and $S$ get their own (separate) record partitions. 

Like Symmetric Hash Join, when an input tuple arrives from a source, we first try partitioning it to memory. Then it is simply placed in its partition and we probe the other table's partitions and output matches.

NOTE: Basically, a relation's in-memory partitions are the same thing as a building hash table for that relation. 

However, if memory becomes *full*, then we flush one of the partitions (all of its records) to its corresponding disk partition, which frees up memory. 

We terminate, like SHJ, when all of our input records for $R$ and $S$ have been processed. 

## Stage 2: In-Disk Joins

This stage runs whenever stage 1 doesn't: i.e. when input records to stage 1 are delayed (stage 1 "blocks"). . Using optimizer, it'll pick a hash partition (based on optimizing output size + cost). We then probe the other relation's corresponding hash partition and output matches. 





