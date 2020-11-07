# Transactions and Locking: Part 2

Last note we introduced concurrency and how we utilized transactions, locks, and schemes such as 2PL and conflict dependency graphs to ensure and verify conflict serializability. 

## Deadlock

Remember that the Lock Manager puts lock requests into the Wait Queue if the locks conflict with what's currently held.

Remember also that **queued locks have to wait for currently held locks to be released before they even get the chance to lock on.**

So let's consider an interesting scenario. Let's say $T1$ and $T2$ both hold S locks on a resource and they *both* request a lock upgrade (to X). In our lock upgrade algorithm, we place both of those lock requests at the front of the queue. 

However, there's a problem: $T1$ and $T2$ are waiting **on each other** to release their respective (S) lock so it can get an X lock. This unfortunate situation is **deadlock**, where there's a cycle of Xacts waiting for each others' lock release. The end result is $T1$ and $T2$ never get their X lock upgrades. 

### Avoiding Deadlock

What are some ways we can prevent deadlock in the first place? 

We can give Xacts **priority** by age (now - start time). 

Thus, now, if $T_i$ wants a lock that $T_j$ holds, we have two options, depending on which Xact has higher priority:

- **Wait-Die**: If $T_i$ has higher priority, $T_i$ waits for $T_j$; otherwise (lower priority) $T_i$ aborts. This is the "capitalism" implementation because the Xact currently holding the lock $T_j$ is given the "benefit of the doubt".
- **Wound-Wait**:  If $T_i$ has higher priority, $T_j$ aborts; otherwise (lower priority) $T_i$ waits. This is the "dog eat dog" implementation because the requesting Xact $T_i$ is given more opportunities, and can immediately "wound" $T_j$ by aborting it if it has higher priority. 

### Detecting Deadlock

A lot of aborts in the wait-die and wound-wait implementations. Instead, what if we had a way to **detect** deadlocks, and if we found one, we **abort one of the Xacts in the deadlock** (releasing its held locks and voiding its lock request) **so the other Xacts can proceed.** 

We use a **waits-for graph** to detect deadlock(s) again with one node per Xact. We say $T_i$ **waits for ** $T_j$, and create an edge $T_i \rightarrow T_j$ if:

- $T_j$ has a lock on resource X
- $T_i$ tries to acquire a lock on resource X, but it's incompatible, so $T_j$ must release its lock before $T_i$'s lock grantable. 

Of course, if there's a cycle- i.e. multiple Xacts wait for each other- we know we have deadlock.

Periodic checks are made for cycles, and if one is found, we will ”shoot” (abort) one of the cycle Xacts.

## Lock Granularity

The ***granularity*** of a lock specifies how "specific" the resource locked is: we can lock a tuple, or a page, or a table, or even an entire database. Multigranularity locking allows us to place locks at different levels of the **database tree**: 

<img src="C:\Users\Kevin\Documents\BerkeleyShit\website\kmoy1.github.io\notes\cs186\LockGranularity.PNG" alt="LockGranularity" style="zoom:50%;" />

The root node represents the database. The database has two child nodes representing two tables. Table 1 contains two pages A and B, while table 2 has one page with $m$ records. Records are leaves of this tree.

**When we place a lock on a node, we implicitly lock all of its children.** Obviously, if $T1$ wants to write Table 1, we don't want $T2$ writing one of its pages (which is the same as writing table 1). 

### Intent Locks

We now introduce the **intent** lock, which states an Xact *intends* to get a lock of some type at a *finer granularity*- i.e. some child node. 

So we have some new locks:

- **Intent-Shared** (IS):  Xact wants to get S lock(s) on children.
- **Intent-Exclusive** (IX): Xact wants to get X lock(s) on children. NOTE: 2 Xacts can hold an IX lock on the same resource, but *only if they place X locks on two different child nodes*. It's up to LockMan to enforce this. 
- **Shared-Intent-Exclusive** (SIX): S and IX locks combined. Xact wants to prevent other Xacts from modifying children but allows reading them (S locks). An S lock prevents any other X locks on children, of course. But for the IX lock property, **we also allow other Xacts to claim S locks (NOT X) on children we don't have an X lock on**. Again, LockMan needs to enforce this.

So this means we're enforcing that S and X locks must be prefaced with INTENT locks on parent nodes (for the same Xact)! 

Note that S locks are actually incompatible with SIX locks, because then the S lock implicitly S locks all children, which prevents X locks (the intent of the SIX lock). Only IS is compatible with SIX, since it's the way other Xacts can claim S locks on non-X-locked child nodes. 

Below is the **compatibility matrix** of two potential Xacts claiming locks on the same resource: 

<img src="C:\Users\Kevin\Documents\BerkeleyShit\website\kmoy1.github.io\notes\cs186\CompatibilityMatrix.PNG" alt="CompatibilityMatrix" style="zoom:33%;" />

 ### Locking Protocol

With these 3 new locks, let's go over our new locking rules:

1. Each Xact starts from the root (the entire database), and traverses down. 
2. If an Xact wants to hold an S OR IS lock on a node, it must also hold an IS or IX lock on parent
3. If an Xact wants to hold an X OR IX lock on a node, it must also hold an IX or SIX lock on parent
4. **Xact locks are released bottom-up (leaf to root)**
5. 2PL and lock compatibility (matrix) enforced
6. Above protocol equivalent to directly setting locks at *leaf* levels.

