# Parallel Query Processing

In the real world, one computer running operations on an entire database won't cut it. Modern applications  need to be able to handle millions of requests over terabytes of data: a single, lone computer can't possibly respond to all those requests in a quick and timely manner.

What if we run our database on *multiple* computers? Parallelism is something that almost always helps with speed. **Parallel query processing** is the art of running a query on *multiple* machines.

## Parallel Architectures

**Shared memory** is when every CPU shares a single memory (RAM) and disk. 

**Shared disk** is when every CPU has its *own* memory, but shares disk. 

Sharing resources generally impedes system performance. Parallelism can be implemented much better if we have a **shared nothing** architecture, where machines have their own memory + disk. Now, they don't have to wait for a resource(s) to become available (for example, each computer can have their own local copy of a resource and don't have to wait for one computer to stop editing it).

## Types of Parallelism

**Intra-query parallelism** focuses on maximizing the speed of a *single* ("intra") query by spreading its work over multiple computers. Intra-query parallelism will be the main focus of this note. 

Inter-query parallelism, on the other hand, focuses on maximizing *overall system throughput* by parallelizing *multiple* queries on different machines. This is the heart of concurrency and is quite tricky to implement safely. 

### Types of Intra-query Parallelism

We can split intra-query parallelism into 2 more classes.

**Intra-operator parallelism** concerns making a single operator run as quickly as possible. For example, if we want to parallelize sorting (one operation), we can divide up the data onto several machines and having them sort the data in parallel. This maximizes speed of sorting immensely.

**Inter-operator parallelism** concerns maximizing a query's speed by running *operators* in parallel. For example, we can make sort-merge join run faster if we parallelize sorting $R$ and $S$. So thus we are parallelizing the entire query. 

### Types of Inter-operator Parallelism

We can split inter-operator parallelism into yet 2 more classes.

**Pipeline parallelism** occurs when operators form a pipeline on records: i.e. records are passed to the parent (higher in the query plan) operator immediately upon output. So not only do we not have to write these records back to disk, but the child operator can work on something else while the parent operator works on what the child just gave it.

**Bushy tree parallelism**, on the other hand, simply occurs when different branches of a query plan tree run in parallel. 

## Partitioning

Given our whole heap of data and a bunch of machines with their own disk, we can now begin to partition data pages to machines (a process known as **sharding**- each DP goes to only one machine). Let's look at some various **partitioning schemes for records** that determine which machine a record goes to.

### Range Partitioning

In **range partitioning**, each machine stores records that contain a specified range of allowed values. Range partitioning is good for lookups, because we can immediately lock in the machine with the required records. Parallel sorting and parallel sort merge join utilize range partitioning before actually performing the parallel sort.

### Hash Partitioning

In **hash partitioning**, records are hashed and sent to a machine that matches that hash value. This means that machines are basically the "buckets": equal values go to same machine, but close (not equal). Key lookup is still good, but range queries (get all records with id between 1 and 10) are not. We use hash partitioning in parallel hashing and parallel hash join.

### Round Robin Partitioning

Finally, in **round robin partitioning**, assign each subsequent record to the next machine, and cycle back. This ensures **load balancing: each machine gets equal number of records**, and thus we maximize parallelization! Con: since there isn't really an order anymore, every machine needs to be read from for every query. 

## Network Cost

Multiple machines communicating over a network always incurs some kind of **network cost** for the traffic sent. The **network cost** is the amount of data needed to be sent over a network for any given operation (like partitioning). Unlike memory-to-disk data transfers, we can now treat a record as the smallest possible unit of data sent over a network (opposed to a page).	

That's good enough for today. Next note we'll cover the sort/hash/join algorithms that utilize parallelism. 