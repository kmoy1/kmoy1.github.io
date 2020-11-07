# Parallel Query Run

- **Interquery parallelism**: parallelize multiple queries (max speed of total throughput)
- **Intraquery parallelism**: max speed of a single query (parallelism of query *operators*).
- Pros and Cons of key partitioning:
  - PROS: search/update operations (which require searching on the key) more efficient- we'll know which computer the date resides on if exists. 
  - CONS: overhead to additional insertions and updates

Let's say we have $m=3$ machines and $B=5$ buffer pages per machine. And finally, $N=63$ data pages with no duplicates.

**Minimum number of passes to sort data?**

Partitioning data perfectly gives $N_m = 21$ pages per machine. If we do external sorting, we'll require $1+ log_{4}(ceil(21/5)) = 1 + log_4(5) = 3$ passes. Pass 0 produces $B-1$ or 5 sorted runs. Pass 1 produces 2 sorted runs. Pass 2 produces 1. 

So we need 4 passes total.

**Minimum number of passes to hash data**?

Partitioning data perfectly gives $N_m = 21$ pages per machine (hash partitioning). 

The first partitioning pass reserves 1 input buffer and $B-1=4$ output buffers (partitions). So each machine partitions 21 pages into 4 partitions, giving each partition 6 pages each. This wont fit in a 5-page memory, so we recursively partition again to give 5 sub-partitions of 2 pages each. This'll work. We need 1 pass to write all the shit to memory (read all pages in each partition, construct hash table in memory, flush)

SO we need 4 passes overall.

**Don't use round robin partitioning if we have duplicate keys. This might send those duplicate keys to different machines. **

Now let's say that relation R has $[R]$ pages and relation S has $[S]$ pages.

If we have m machines with B buffer pages each, what is the number of passes in order to perform sort merge join (in terms of R, S, m, and B)? Consider reading over either relation to be a pass.

(1 pass to partition each of the two tables across machines) + (the number of passes needed to sort R) + (the number of passes to sort S) + (1 final merge sort pass, going through both tables).

- partitioning R and S across machines takes TWO passes (one across each table)
- Sorting R takes standard sorting passes. Same with S. 
- Finally, the sort merge pass iterates through both tables, so TWO passes. 