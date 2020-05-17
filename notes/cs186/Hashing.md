# Hashing

Sometimes for data we just want to group like values together (GROUP BY or removing duplicates), but we don't really care about overall order. **Hashing** is a way to group like values together. If we could just fit all our data in memory we'd simply create a hash table and be done with this. 

But we can't. Let's see how we can work with the constraints of memory to hash. 

## General Strategy

Like merging sorted runs, we'll **concatenate hash tables** in our algorithm.

We do have to account for something: if two separate hash tables both have the same value, concatenation will probably not group those values together! So we need to guarantee that **if a certain value is currently in memory, all of its values are in memory at the same time**. In other words, if "Xanax" is in memory all occurrences of "Xanax" must be in memory, or we can't build the hash table. 

## Algorithm

Divide and conquer- the opposite of our sorting algorithm!

The initial “divide” phase is simply hash partitioning passes: reading in data and writing it to partitions. Our “conquer” phase will be actually constructing the hash tables. 

### Divide (Partitioning) Pass

Utilizing $B$ memory pages, we use 1 input buffer and $B-1$ output buffers, each output buffer representing a partition that we hash each record in the input buffer to. Each hash partition contains records that all have the same hash value (according to some hash function). 

When any hash partition fills fills up we flush the page to disk.

Each flushed buffer page is placed adjacent to the one flushed before it in disk- a means of "concatenation".

**Most importantly, a partition will contain ALL occurrences of a particular value (next to each other)**. This is ensured because all duplicates will hash to the same partition via hash f(x).

### Conquer (Build Hash Table) Pass

Now we build a hash table out of the partitions that **fit in memory**, i.e. partitions $\le B$ pages. Partitions that don't fit ($> B$ pages) will simply be recursively partitioned with a **different** hash function (into sub-partitions hopefully $\le B$ pages big). This will count as **another partitioning pass**: we need to read and write these pages to disk. 

Note if we reused our hash function we'd just hash everything to a single (oversized) partition again, so that gets nowhere. 

Once all partitions fit in memory, we simply **build a hash table for each partition and write each table to disk**.

## Analysis 

Unfortunately, unlike sorting, we don't know how many passes we need to fit all partitions into memory (we don't know how large partitions will be). So there's no general IO formula. 

All good. **It's possible for the number of pages in the table to increase after a partitioning pass**. Let's consider an example. 

Consider the following 3-page table which contains 2 records (integers) per page. We want to hash this table:

$$[1,2][1,4][3,4]$$

If we have $B=3$, i.e. we have 2 partitions that take hash values 0 and 1. Let's say our hash function is

$h(x) =
\begin{cases}
0,  & \text{if $x$ is odd} \\
1, & \text{if $x$ is even}
\end{cases}$

So our first partitioning pass will give partitions:

Partition 1: $[1,1],[3]$ (2 pages)

Partition 2: $[4,2],[4]$

So if partition pages contain less-than-full data pages, we can end up with more than what we started with. 

Therefore we need to count IOs painfully and step-by-step: by going through each pass and see adding IOs for every read and write. 

Here's a general *algorithm* to do so:

- $m$ partitioning passes
- $r_i$ pages read in for partitioning pass $i$
- $w_i$ pages written for partitioning pass $i$
- $X$ total pages *after partitioning* to build our hash table out of

$$\sum_{i=1}^{m}(r_i + w_i) + 2X$$ then gives the total number of IOs for hashing $N$ pages. 

Some important properties:

- $r_0 = N$. We always read all $N$ pages in the first partitioning pass. 
- $r_i \le w_i$. It's possible to write more pages (above example) than read pages, but never read more pages than write (that would mean we lost some records in partitioning).
- $w_{i+1} \ge r_i$. We will not read in more pages than what was written during the previous partitioning pass. The worst case is that every single partition from pass $i$ is too big and must be repartitioned, so we'd have to read in every page. 
- $X \ge N$: the number of pages making our hash table is at least as big as the number of data pages we started with. **Partitioning passes can only increase the number of data pages!**