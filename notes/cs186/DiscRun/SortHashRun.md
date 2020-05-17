# Sorting/Hashing Discussion Run

**Q1: With $B = 4$ buffer pages $N=108$ pages of records to sort, how many passes to sort?**

**A1: 4 passes.**

We know the number of passes is $1 + \lceil \log_{B-1}(\lceil N/B \rceil) \rceil  = 1 + \log_3(27) = 4$. 

The first pass is used to sort all individual pages, creating 27 sorted runs of 4 pages each. The next 3 passes merge these 27 runs into 1 sorted run of pages. 

**Q2: How many sorted runs would each pass produce?**

**A2: Pass 0 produces 27, Pass 1 produces 9, Pass 2 produces 3, Pass 3 **produces 1. Notice we're merging $B-1 = 3$ runs at a time.

**Q3: What is the IO cost of sorting? ** 

**A3: 864 IOs**

We know that each pass takes $2N$ IOs to read and write all the pages operated on in memory. So with $N=108$ and 4 passes, we have $4 * 2 * 108 = 864$ IOs total. 

**Q4: If pages started already individually sorted, how many passes and IOs to sort the file?**

**A3: Still 864 IOs**

We still need Pass 0 to produce $\lceil N/B \rceil$ runs of $B$ pages- starting off sorted doesn't change the run-creation step. 4 passes and 864 IOs remains required.

**Q5: Now let's say we wanted to set an upper limit $p$ on the number of passes to sort $N$ pages with $B$ buffer pages. Given $N$ pages and $p$, min number of buffer pages $B$ needed? (Write expression involving N and p)**

**A5: $B(B-1)^{p-1} \ge N$**

We can start with the inequality $1 + \lceil \log_{B-1}(\lceil N/B \rceil) \rceil \le p$.

$\log_{B-1}(N/B)\le p-1$

$(N/B) \le (B-1)^{p-1}$

$N \le B(B-1)^{p-1}$

We notice that if we wanted to set a hard $p=1$, then we require $B \ge N$ i.e. we need to fit all of our pages into memory then apply whatever sorting algorithm.

## Hashing

For hashing, we still want to hash $N$ pages utilizing $B$ buffer pages. 

The first "pass" is the partitioning pass 1. We hash partition $N$ pages into $B-1$ partitions, and recursively do so until all partitions can fit in memory, i.e. $\le B$ pages big.

Our first pass requires us to write $\sum_{i=1}^{B-1}\text{(pages in partition i}) $. We will also have to read and write additional pages for recursive partitions (*read* initial oversized partition, *write* recursive partitions). Finally, after all our partitioning we read all pages, build in-memory hash table, and write all pages in this table. 

**Q1: Why hashing over sorting?**

**A1: Sometimes we don't care about overall order- just grouping like values together.  EX: Removing duplicates, GROUP BY without ORDER BY**

Suppose we have $B$ buffer pages and can process $B(B-1)$ pages of data with hashing in 2 passes- i.e. one partitioning pass, one hash-table-building pass. 

**Q2: How many input buffers did we use? How many partitions after pass 0? How many pages per partition?**

**A2: 1, $B-1$, $B$**

We always have 1 input buffer for external hashing. Since we know we processed of $B(B-1)$ pages, and we know we had 1 partitioning pass, we know we produced $B-1$ FULL partitions of $B$ pages after pass 0- a perfect load balance. 

**Q3: To process exactly $B(B − 1)$ pages with external hashing, is it likely that you’ll have to perform recursive external hashing (i.e. more than 1 partitioning pass)? Why or why not? **

**A3: Very very likely. **

To use only one partitioning pass, we'd need an absolutely perfect hash f(x) that perfectly load balances $B(B-1)$ pages into B - 1 partitions and completely fill them all. This is just not possible in practice: some partitions will probably have $\ge B$ pages. 

Assume $N = 100$ pages using $B = 10$ buffer pages. Suppose in the initial partitioning pass, the pages are unevenly hashed into partitions of 10, 20, 20, and 50 pages. Assuming different uniform hash functions are used for every recursive partitioning pass.

**Q4: How many IOs in this scenario?**

**A4: 634 IOs**

P-Pass 0: 100 IOs to read in all N=100 pages. We write all our partitions (10, 20, 20, 50) to disk: another 100 IOs.

P-Pass 1: We can immediately build a hash table with the size-10 partition: we read and write 10 pagess for 20 IOs. 

3 partitions are too big: ones with 20 and 50 pages. 

For a 20-page partition, we read and recursively partition into 9 partitions of size 3. So this requires us to read 20 pages and write 27 for 47 IOs total. Then building the hash table requires us to read and write 27 again. So in total, 20 + 27 + 27 + 27 = 101 IOs. The other 20-page partition also takes 101 IOs (same process). 

For a 50-page partition, we recursively partition into 9 partitions of size 6. Thus read 50, write 54, read 54, write 54 to get 50+54+54+54 = 212 IOs.

Total, we need 212 + 101+101 +20 + 200 (initial pass 0) = 634 IOs total. Whew! 

