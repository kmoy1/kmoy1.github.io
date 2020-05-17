# Recovery

**How do we make our DB handle failures?** 

Two important properties (ACID) in recovery are:

- **Durability**: Xact effects persist.
- **Atomicity**: Xact either commits (all ops done) or aborts (no ops done). *Never* leave Xact halfway done. 

## Force/No Force 

**Force policy **: When Xact finishes, **flush all modified data pages before committing. **Once a data page is flushed to disk it's permanently saved. This ensures durability. CON: lots of unnecessary writes. 

**No Force policy** states that we **only flush a page to disk when the page needs to be evicted from memory.** We prefer no force. However, this complicates durability because if the database crashes *after* the Xact commits, some pages that we wrote to might not get saved! So we'll **redo** Xact operations during recovery to ensure our actions persist. 

## Steal/No Steal

A **no steal policy** states pages cannot be evicted from memory (flushed) until Xact commits. This ensures atomicity: Xact operations are reflected in memory, and none of these operations get saved until all these pages are flushed at once. CON: every dirty page must stay in memory until a Xact commits. This will constrain how much memory we can use. 

A **steal policy** allows dirty pages to be flushed ("stolen") whenever (before commit). But note this is like saving one operation at a time- the exact opposite of atomicity. So we'll **undo** (bad) operations of aborted Xacts during recovery.

So we choose **steal/no force** as our best-performance policy.

## Write-Ahead Logging

A **log** describes the operations that the database has done through log records. **Each write operation (insert/delete/update) gets its own log (UPDATE) record**. 

An UPDATE log record looks like this:

<XID, pageID, offset, length, old_data , new data>

- XID: Xact ID that did op
- pageID: page modified
- offset: Start of edit on page
- length: How much data changed
- old_data: pointer to original data (used for UNDO)
- new_data: pointer to edited data (used for REDO)

There are also log records for COMMIT, ABORT, and END (Xact finished committing/aborting and is thus finished)

**Log pages** are treated like regular pages: edit in memory, flush to disk to save permanently. 

**Write Ahead Logging** (WAL) is the process and rules of flushing logs to disk. 

1. **Log records flushed *before* the dirty page gets flushed**. This key to atomicity: we know what happens for a write and can undo it in case of DB crash (needing to abort Xact). If we didn't flush the LR first, we obviously can't undo this op (because we don't know it happened!)
2. **All log records (in memory) MUST be flushed when a Xact commits**. This key to durability: we need something on disk for each operation when a Xact commits, so we can redo them all during recovery. 

## WAL Implementation

Log records have an important field called **LSN** (Log Sequence Number). The LSN is a unique increasing number that gives order to operation (op w/ log record LSN = 20 happened *after* op with record LSN = 10). 

Another field **prevLSN **stores the last op from the same Xact (useful for undoing Xact).	

The database also tracks **flushedLSN**, **which is the last log record flushed to disk**. Page flushed = page evicted from memory. If you think about it, the **flushedLSN indicates which log records should *not* be written to disk because they are already there!** (LSN $\le$ flushedLSN). We don't want to store duplicate log records, as that'd mess up the log (and redo/undo).

Finally, the **pageLSN** is the LSN of last op that dirtied page. 

So thus, **before page $i$ is flushed to disk, pageLSN must be $\le$ flushedLSN**. We must flush all log records before flushing page, and pageLSN is the last flushed record. If pageLSN > flushedLSN, we know we haven't flushed a log record yet.

## Aborting Xacts

At *crashpoint*, we need to abort in-progress Xacts. Some reasons to abort: deadlock, or user choice (perhaps Xact taking too long). We need to also ensure that because we aborted these Xacts, none of their ops persist in disk. 	

First, we log an ABORT record, signifying that we are starting the process for an Xact. Then we start at the Xact's **last log operation** (specified by pageLSN). For each operation undone, we log a **CLR Record**, which is a log record simply indicating we undo some Xact op. CLR Records are basically UPDATE records (stores previous state and new state), since it's technically writing a page, but it also indicates that this "undo write" happened from an ABORT. 

## Recovery Data Structures

We keep track of an **Xact table** that stores info about active Xacts. The Xact table maps Xacts (IDs) to **status and lastLSN**. Status is just if Xact is committing/running/aborting, and lastLSN is the **most recent op LSN**. 

We also maintain a **Dirty Page Table**, which keeps track of dirty pages (modified in memory, but not flushed yet). This table tells us what pages have operations that need to be redone, since these pages have ops done that haven't been permanently saved yet (flushing). The DPT maps page ID to **recLSN**: the **first** dirtying page op (can be for multiple Xacts). 

Both of these tables stored in memory. Memory is fucked up at crashpoint, so we need to reconstruct: ***checkpointing*** will make this easier. 

## ARIES Recovery Algorithm

When a database crashes, only disk persists. So we only have the data pages and logs flushed to crashpoint. From this information, it should restore itself so that all committed Xacts' ops have persisted (durability) and all transactions that didn’t finish before the crash are properly undone (atomicity). Thus we have 3 phases: 

1. Analysis: Reconstruct Xact table, DPT
2. Redo: Repeats Xact ops for committed Xacts.
3. Undo: Undo Xact ops, for running (and thus aborted) Xacts.

### Analysis Phase

Purpose: rebuild what the Xact Table and the DPT at crashpoint.

One problem with the analysis phase so far is that it requires the database to scan through the entire log.  In production databases this is not realistic as there could be thousands or millions of records. To speed up the analysis phase, we will use checkpointing. **Checkpointing writes the Xact table and the DPT to the log**. This way (like its name suggests) we can start analysis from the last checkpoint, instead of the beginning of the log.

Checkpointing writes a BEGIN_CHECKPT record (when checkpoint started), and END_CHECKPT record, when we finish writing Xact table and DPT to log (when checkpt finish). **The Xact table and DPT state can be at any state between BEGIN AND END CHECKPT**

### Redo Phase

Now, all dirtied pages should be in the DPT, along with their most first dirtying OP LSNs.

In redo, we repeat history (committed Xact ops) in order to reconstruct the state at crashpoint. I.e. we redo specific ops in log that might not have been saved to disk yet.

Begin at **smallest recLSN in the DPT**: first op that might not have been flushed (made it to disk). **Redo all UPDATE and CLR operations** unless:

- **Page not in DPT**: all changes (and thus this one!) have already been flushed to disk (which means they've been already saved permanently)
- **recLSN > LSN:** First dirty op the page occurred after this op: thus we know that current op already been flushed!
- **pageLSN (disk) ≥ LSN**: Most recent update to the page that made it to disk flushed, so an op occurring before this must have also been flushed. 

### Undo Phase

Finally, we start at the END of the log and go backwards: **undo every UPDATE (and only UPDATEs!) for each active Xact at crashpoint.** If a CLR for an UPDATE already exists, we don't undo that (because we'd be undoing an undone operation, which is basically a redo!)

For every undone UPDATE, we log a CLR. CLR records have an **undoNextLSN** **field, which stores the LSN of the next op to be undone for that Xact** (derived from prevLSN of current op).

Once we undo all an Xact's ops, log END record.

## Conclusion

Databases utilize a steal, no-force policy to guarantee recovery from crash. The database uses WAL policies to log all operations when running. The database utilizes this log in a 3 step process (analysis, redo, undo) in recovery and restores the database to its correct state at crashpoint.