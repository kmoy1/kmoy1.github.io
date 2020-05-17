# Transactions and Locking: Part 1

**	Many problems can arise if we don't handle concurrency:

- **Inconsistent Reads**: Reading only part of what was updated, because of bad timing.

  For example, let's say a User 1 updates tables A and B, but after User 1 finishes updating table A (and before updating table B) User 2 swoops in and reads tables A and B. User 2 won't be reading an updated table B!

- **Lost Update**: User 2 overwrites user 1's updates to A (e.g. they edit the same record in a relation).

- **Dirty Reads**: One user reads an update that was never committed (aborted)

  Let's say User 1 updates table A, but he says fuck this shit and aborts. If User 2 reads the table before the update is **rolled back** (undone), he'll be reading something dirty. 

## Transactions (Xacts)

**Transactions** are sequences of actions that should be executed as a single, logical, ***atomic*** unit. Think of it like a recipe or a blueprint. 

A transaction, if implemented correctly with concurrency control in the DBMS, will guarantee the **ACID properties**:

- **Atomicity**: Xact either commits or aborts. Either every action happens or none happen. 
- **Consistency**: Xact's effects always are consistent on the database resources: no randomness.
- **Isolation**: Each Xact's actions are isolated from other Xacts. In reality, this isn't how it works, but the **DBMS ensures the illusion that each Xact executes isolated**.
- **Durability**: Even in case of failure, if an Xact is committed, its effects persist. 

## Concurrency Control

This note focuses on enforcing isolation in Xacts. To do this, we introduce **transaction schedules** which, for a given transaction, show order of operations in list form. Such operations, for our purposes, include: Begin, Read, Write, Commit and Abort.

If we want to enforce isolation, we need to focus on one Xact at a time. Ideally, then, we'd like to run through all of an Xact's ops before starting the next Xact.

A **serial schedule** is a schedule of transactions that run completely through, one after the other (SERIALLY). For example, if we have two Xacts T1 and T2, a serial schedule would mean T1’s ops run completely before T2 even begins.

It's actually pretty inefficient to run Xacts one at a time, though. This is where providing the *illusion* of isolation comes in. We want to be *like* a serial schedule (because we know it's correct) while utilizing the performance benefits of concurrency- running Xacts simultaneously. To do this, we establish the notion of **equivalent schedules**, schedules which must follow 3 rules:

1. Same Xacts
2. Same Xact op order
3. Same effects on database

A schedule equivalent to a serial schedule is **serializable**, and has the same effects of that serial schedule. 

The first two properties are easy money to check. What about property 3? We could always just run through each schedule and compare resulting database states. However, that seems like unnecessary work- can we avoid this? 

Certainly. We do so by looking for **conflicting operations** in the schedules. Conflicting operations involve operations (read/write) from different Xacts operating on the same resource. Also, at least one of these operations *must* be a write. **If the two schedules order every pair of conflicting operations in the same way, they have the same effects on the database.** Not only are these schedules equivalent, but they are **conflict equivalent**. (It is possible for 2 schedules to be equivalent and not conflict equivalent).

A schedule that is *conflict equivalent* to some serial schedule **conflict serializable.** (stronger than serializable).

### Conflict Dependency Graph

So our way of proving serializability for a schedule is proving conflict serializability- proving something stronger. A **dependency graph**, with a node per Xact, will tell us if the schedule is conflict serializable or not. 

We say that node $T_j$ **depends** on node $T_i$, and we thus draw an arrow from $T_i$ to $T_j$, if: 

- One of $T_i$'s ops conflicts with an operation $T_j$'s ops (both operate on same resource)
- $T_i$'s op appears earlier than $T_j$'s op.

**A schedule is conflict serializable (and thus serializable) iff its dependency graph is acyclic.** 	

## 2 Phase Locking

**Locks** are "reserve tickets" on resources: what allows an Xact to read and write data safely to that resource. 

Let's say Xact $T_1$ is currently reading resource $A$. That Xact must ensure that $T_2$ isn't currently *modifying* $A$ also (other Xacts can read concurrently, though). So Xacts wanting to read a resource will request a Shared (S) lock on that resource. Xacts wanting to write (edit) a resource will request an Exclusive (X) lock on that resource.

For a resource, as many S locks as possible are allowed, but only one X lock is allowed. 

### Two Phase Locking

**Two phase locking (2PL) is a scheme that ensures conflict serializable schedules.** Two rules:

- Xacts must get S lock *before* reading, X lock before writing.
- **After an Xact releases a lock, it cannot get new locks**. This is the key to enforcing serializability through locking!

However, we must also be mindful of **cascading aborts**, which this scheme doesn't prevent. For example, let's say $T1$ writes $A$ (acquires X lock on A) and then releases the lock on $A$ (without committing). Then, $T2$ reads $A$. If $T1$ aborts, then $T2$ must also abort because it made a dirty read on $A$ (uncommitted update). One Xact aborting forcing another to abort certainly isn't isolation. 

Our solution is **Strict Two Phase Locking** (Strict 2PL), which is the same as 2PL, except **all Xact locks get released together, and only when the Xact completes.** So even after an Xact is done reading/writing a resource, it can't release its lock on it until it commits. So in our above case, $T2$ won't be able to read $A$ because $T1$ isn't done and still holds an $X$ lock on $A$, and the dirty read is avoided. 

## Lock Management

So now we know that Xacts are gonna be making a bunch of **lock requests** for the various database resources. How does the Lock Manager manage these requests? 

The Lock Manager keeps a **hash table keyed by locked resource name**. Each resource name maps to a granted set of locks (i.e. Xacts holding the locks on the resource), current lock type, and a wait queue of lock requests: (Xact, requested lock) pairs where the requested locks conflicts with the current held lock.

Here's an example of a Lock Manager hash table: 	

![LockManHashTable](C:\Users\Kevin\Documents\BerkeleyShit\website\kmoy1.github.io\notes\cs186\LockManHashTable.PNG)This table tells us *exactly* what we need to do for various lock requests. For a given incoming lock request,  the LockMan checks for conflicting locks in both the granted set and wait queue. If there's a conflict anywhere, straight to the gulag (wait queue). Otherwise (no conflict), then we grant the lock and add the Xact to the granted set (and change the mode if necessary).

### Lock Upgrade Requests

Xacts can also get greedy and request a **lock upgrade**: i.e. Xact with S lock requesting upgrade to X lock. Such **upgrade requests are added to the front of the wait queue**.

Now when do those queue locks get to hold the resource? After a lock is released, the wait queue is iterated  through. For each lock in the queue, LockMan checks if it's compatible with any current locks on the resource. If it is we grant that lock, **otherwise we stop immediately.**

### Queue Skipping

Note that this implementation does ***not*** allow **queue skipping**, where upon an incoming lock request, we first check compatibility with currently held locks. If the lock cannot be granted, then put it at the back of the queue. **When a lock is released, grant *any* queued lock request's locks that are compatible with currently held locks.** (Basically, we just don't return immediately if we find an incompatible queue lock request)

So we've covered concurrency, transactions, and locks in this note. That's good enough.

The next note handles deadlock and lock granularity. 