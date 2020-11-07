

# Distributed Transactions

For databases with lighter workloads, all data can usually be housed all on a single machine. However, as we've discussed in previous notes on parallelism, **shared nothing architecture** ensures each machine receives a *partition* of the data, which is distributed by range/hash key/load balance. In such *distributed databases*, we need **distributed transactions** to execute queries, which perform reads and writes on data that exist on different machines.

## Distributed Locking

In shared-nothing, every machine has its own independent data: thus, **every machine can maintain its own local lock table**. Coarser-grained locks (locks on entire tables or even the database) can either be distributed to all nodes with a partition or be centralized at some pre-determined node.

In this design, **2-phase locking** is performed at every node using local locks in order to guarantee serializability between different transactions. 

What about handling deadlock? Remember that because each node's data is independent, each node has its own waits-for graph. So the **waits-for graphs for each node must be unioned** to find cycles as transactions can be blocked by other transactions executing on different nodes.

## Two-Phase Commit

In a distributed database, **consensus** is the rule that all nodes all do the exact same thing, in unison, on any piece of data. 

Consensus is implemented through **Two Phase Commit** and enforces the property that all nodes have the same "view" of the data. The same way atomicity enforces that a transaction either commits or aborts on a single resource, two phase commit ensures a distributed transaction either commits or aborts on *all* its resources (over all nodes). **If** **consensus is not enforced, concurrency problems arise**: for example, if some machines commit the transaction while others abort it, this would cause different viewpoints on "shared" resources for different nodes.

For every distributed transaction, we have a **coordinator node** responsible for maintaining consensus among all transaction's nodes. At commit time, this coordinator node starts Two Phase Commit:

### Phase 1: Preparation

2 PC’s first phase is the **preparation phase.**

Remember that for a *single* distributed transaction, we have a coordinator node and a bunch of participant notes.

1. coordinator node broadcasts "prepare" message to participants, i.e. tells participants to either prepare for commit or abort. Remember we're enforcing universal atomicity here. 
2. Participants generate a prepare/abort record, then flush this record to disk. 
3. If a participant node flushes prepare record, send YES to coordinator. If node flush abort record, send  NO
4. Coordinator generates a commit record if **unanimous** yes votes or an abort record otherwise and flushes record to disk. Remember we want to enforce universal atomicity- if even a single abort record comes from one of the nodes, we gotta trash everything and abort the transaction entirely. 

### Phase 2:  Commit/Abort phase

1. Coordinator broadcasts (sends message to every node in the db) if it flushed COMMIT/ABORT. 
2. Participant nodes follow suit and flush same record as coord
3. Participants send an ACK message to coordinator
4. Once all participants send ACK, coordinator generates END record and flushes the record at some point later.

## Distributed Recovery: Flushed Commit/Prepare Records

**2PC maintains consensus even in the presence of node failures.** For example, if a node were to fail at any point during 2PC, when the node comes back online, it should still end up doing the same things and having the same data view as the other nodes in the database.

How? 

The 2PC protocol tells us to log (flush) prepare, commit, and abort records. This is more than enough to help us with recovery. Let’s look at what happens at each possible failure point in the protocol, for either any participant node or the coordinator. We'll mention these in chronological order.

### Failure 1: Recovering Participant, No Prepare Record

The first possible failure point is in the first step of phase 1: participant node errors, begins recovery, and sees no prepare record flushed by it to disk. This probably means that the participant has not even started 2PC yet. We also know it hasn't sent out any vote (YES/NO) messages to the coordinator yet, since those happen *after* flushing the prepare record to disk (and we didn't find this prepare record). Thus because it hasn't voted yet, we can safely abort the transaction (locally). This will, of course, lead to the entire thing getting aborted for everyone, since the unanimous vote is broken. x

### Failure 2: Recovering Participant, Prepare Record

Next, we could have a recovering participant that *has* flushed a prepare record. A lot of things could have happened between logging prepare and crashing.

We don’t know if the coordinator made a commit decision or not (step 4 of prep). So recovery asks the coordinator if it did (”Did the coordinator log a commit?”). Upon response (commit or abort), the recovering participant resumes 2PC from phase 2.

### Failure 3: Recovering Coordinator, Commit Record

Another failure point is located at the end of the prep phase, where the coordinator node is recovering, and a commit record exists in the log. So obviously we *want* to commit this Xact, but we don’t know if we broadcasted our decision yet (started phase 2).

Thus, the solution is to simply re-run phase 2, by broadcasting commit messages and going from there. 

### Failure 4: Recovering Participant, Commit Record

In phase 2, we can have a recovering participant who flushed a commit record already. So now we just need to send the ACK to the coordinator. 

### Failure 5: Recovering Coordinator, End Record

If the coordinator fucks up AFTER it flushes the end record, we're good, because everyone already finished Xact (finished phase 2). No need for any recovery steps!

## Distributed Recovery: Flushed Abort Records

We could handle recovering nodes with flushed ABORT records in the same way as before (e.g. tell participants, send acks). This works fine, but can we save some work here? What if nodes just didn’t bother recovering aborted transactions? I.e. what if nodes just...did nothing?

This will work if we set a **universal standard** (among participants and coordinator) that no log records => abort. This optimization is called **presumed abort**. This means abort records *never* have to be flushed in *either* phase by *anybody*.

Thus action in points of failure for nodes flushing abort records are far simpler: DO NOTHING. Thus the below failure points assume no presumed abort.

### Failure 1: Recovering Participant, Abort Record (Phase 1)

Manually vote "NO" to the coord.

### Failure 2: Recovering Coordinator, Abort Record

Run phase 2 over again (broadcast abort)

### Failure 3: Recovering Participant, Abort Record (Phase 2)

Send back ACK to coord.

## Summary

Overall, **2PC recovery decision is commit iff coordinator has logged a commit record**(end of phase 1). Since 2PC requires unanimous agreement, all nodes must be active (not errored/recovering) to finish the commit/abort process. Same for recovery: all failed nodes must eventually come back alive. 

