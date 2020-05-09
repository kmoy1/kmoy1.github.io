# Grace Hash Join

The fundamental part of join algorithms is their means of finding matching records of $R$ and $S$. Lookups are synonymous with hash tables. We can construct a hash table that is $B-2$ pages (1 buffer page needed for input (current page in $S$), 1 buffer page needed for output records) big on the records of $R$, **fit it into memory**, and then simply find all of $R$'s matching records for each record in $S$. 

This is called **Naive Hash Join**. Its cost is $[R] + [S]$ I/Os. 

Damn- this join algorithm is efficient, cheap, and simple. Unfortunately, the biggest problem with this is that **Naive Hash Join relies on R being able to fit entirely into memory** (specifically, having $[R] ≤ B − 2$ pages). If $R$ is a massive table, this is obviously gonna be a problem. 

This is where **Grace Hash Join** comes in.

We repeatedly hash $R$ and $S$ into $B-1$ buffers until we get hash partitions ≤ $B − 2$ pages big (which we can then fit into memory). More specifically, consider each pair of corresponding partitions $R_i$ and $S_i$. If Ri and Si are both > B-2 pages big, hash *both* partitions once again into smaller ones. Otherwise, if either $R_i$ or $S_i ≤ B-2$ pages, load the smaller partition into memory, build the hash table, and perform NHJ with the other (larger) partition.

**I/O cost of GHJ = cost of hashing + cost of Naive Hash Join on the subsections.** The cost of hashing can change based on how many times we need to repeatedly hash on how many partitions. 

Cost of hashing a partition $P$ = read IOs for all pages in $P$  + write IOs for all resulting sub-partitions.

