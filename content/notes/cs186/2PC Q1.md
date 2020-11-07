# 2PC With Logging: Questions

One of the key points to understand about distributed transaction operation is **2-phase commit**: the Xact commit protocol followed that ensures sufficient recovery for all nodes in the network, if any of them were to error at any time during commit. The following questions concern potential scenarios during execution of 2PC.

### Pre-Vote Crashes

Suppose the coordinator sends PREPARE messages to the participants and crashes **before receiving any votes**. Assume presumed abort. 

**Q1: Coordinator Recovery Steps?**

**A1: Coordinator aborts the transaction locally by UNDOing its actions.** We log undo records for each action undone and finally a concluding abort record.

Before we think about this, let's locate where we're at in 2PC. We know we're in the beginning of the preparation phase (phase 1), right after the coordinator node broadcasts "prepare". 

After the coordinator restarts, the recovery process sees an Xact was executing at crashpoint. Additionally, it looks at the coordinator log and sees no commit records- or any records at *all*. So with presumed abort, the coordinator must presume abort as well, and aborts the Xact on its end.  

**Q2: Participant who voted NO (before the coordinator crashed) recovery steps?**

**A2: Participant aborts the transaction locally**. 

If *any* participant sends NO, the transaction will be aborted. Note a participant gives zero fucks about what happens with a coordinator in this case. 

**Q3: Participant who voted YES (before the coordinator crashed) recovery steps?**

**A3: Depends on the coordinator. **

This situation is a little trickier. We can't just commit the Xact locally because some other participant might have aborted. Obviously, we can't just abort either (what if everyone else also voted YES?)

So it depends on the coordinator. The recovery process will periodically contact the coordinator for instructions. After the coordinator recovers,  we know it aborts the Xact (since it didn't log anything) and will answer ”abort” upon receiving an inquiry message. Participant nodes then (all) abort.

### Post-Vote Crashes

Now let's say our coordinator crashes **AFTER** receiving all participant votes. Let's say every participant voted YES except one (one NO vote), and the coordinator didn't log records after coming back online. 

**Q1: How does this affect how participant and coordinator nodes recover?** 

First, the coordinator's behavior isn't gonna change, because it still sees no log records and presumes abort. We abort Xact locally, and since there is no information about the participant IDs in the log,

For participants who vote NO, the participant aborts.

For participants who vote YES, the same logic follows from before: the coordinator comes back online and sees nothing in its logs, so it aborts locally. Participants who vote YES will *ping* the coordinator for instructions, and the coordinator will tell it to abort. 

**Thus, the recovery process stays the same, no matter if the coordinator crashes before or after receiving votes.** 

NOTE: obviously, presumed abort means less log records flushed to disk (compared to no presumed abort). However, if a transaction is successful (commits), the number of flushed records stays consistent (no presumed commit).

### Post-Vote Crashes: Participants

Now lets say a *participant* crashes after receiving prepare message and replying YES. Let's also say every other participant also voted YES without crashing. 

**Q1: What is the recovery process (post-restart) for this lone participant?** 

Upon restarting, the recovery process locates preparation state of participant. Again, it periodically pings the coordinator site for further instructions on how to handle the Xact. Since everyone voted YES and the coordinator got all these votes, coordinator will respond to COMMIT the Xact. The participant will then commit the Xact locally. 

### 2PC Runtime Calculation

Suppose we have one coordinator and three participants. It takes 30ms for a coordinator to send messages to all participants; 5, 10, and 15ms for participant 1, 2, and 3 to send a message to the coordinator respectively; and 10ms for each machine to generate and flush a record. Assume for the same message, each participant receives it from the coordinator at the same time. 

**Q1: How long does the whole 2PC process (from the beginning to the coordinator’s final log flush) take for a successful commit in the best case?**

**A1: 130 ms**

During phase 1, we take 30 ms to broadcast our prepare message. We take 10 ms for each machine to generate and flush a (PREPARE/ABORT) record. We wait 15 ms for the lagging participant 3 to vote (parallelism). Finally, the coordinator needs to take 10 ms to flush COMMIT/ABORT. So phase 1 total is 30 + 10 + 15 + 10 = 65 ms. 

During phase 2, we take 30 ms for a coordinator to broadcast COMMIT/ABORT again. Then, 10 ms for each participant flushing that record.  We wait 15 ms for the lagging participant 3 to send ACK. Finally, 10 ms for coordinator to flush END record. So phase 2 total is 30 + 10 + 15 + 10 = 65 ms.

Adding that up gives a total of 130 ms. 