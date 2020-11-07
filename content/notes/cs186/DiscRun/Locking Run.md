# **Locking** Disc Run

Given a schedule, we can make a dependency graph and tell if the schedule is conflict serializable, as well as give conflict equivalent serial schedules.

Given a schedule, we can make a "waits-for" graph and tell if the schedule has deadlock: Xacts waiting on each other to release locks on a resource. 

Given a schedule (that shows locking), we can tell if execution uses 2PL/strict 2PL/neither. 

## Multigranularity Locking

Let's say a Xact T1 wants to scan a table R and write (update) some tuples

**What kinds of locks should T1 have on R, the pages of R, and the updated tuples?**

Well, eventually, T1 wants an X lock on the tuples. 

But our protocol requires that we hold an IX lock on parent, i.e. pages of R. 

So we obtain **SIX lock** on R, IX lock on page of R with tuples, X lock on tuple(s).

**S lock compatible with IX?**

No, because of the matrix. But let's think why.

Let's say T1 wants an S lock on R, and T2 wants an IX lock on R. 

This implies T1 will read the entire object (i.e. all child nodes) so S locks are placed on all children. However, IX implies that there will eventually be an X lock on a child. 

So some child of R will be written by T2 and read by T1. This can't be right. INCOMPATIBLE. Btw, SIX locks try to enforce different locks on different trees. 

Now consider a table which contains two pages with three tuples each:

- Page 1 contains Tuples 1, 2, and 3 
- Page 2 contains Tuples 4, 5, and 6

Let's say T1 has an IX lock on the table, an IX lock on Page 1, and an X lock on Tuple 1. 

**Which locks could be granted to a second transaction T2 for Tuple 2?**

T2 can place an IX or IS lock on page 1! However, he won't be able to touch tuple 1, only touch tuple 2. Which means T2 can place X or S lock on tuple 2.

Now let's say T1 IS-locked table and S-locked Page 1.

**What locks could be granted to a second transaction T2 for Page 1?**

S locks are only compatible with S and IS locks. 

