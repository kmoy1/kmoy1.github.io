# Final Run 1

## ANTI Joins

What join algorithms can be used for this ANTI-JOIN? 

```sql
SELECT * FROM R, S WHERE R.a != S.b;
```

**We need join algorithms that can take generic join conditions** $\theta(r_i,s_j)$. Only **PNLJ** and **BNLJ** will work for this! Index-Nested Loop join utilizes indices which only allow return matches for equality. Grace Hash Join also only works for equality: the probing phase joins by equality only (although I guess it could be adjusted to probe for inequality?? That's kind of stupid and defeats the purpose of hashing, though). Sort Merge Join *certainly* cannot join by inequality- the whole point of the sorting process was so it could return matches to $R$ in a linear move-down fashion.

## BNLJ: Buffer Twist

Let's say BNLJ was invented such that only $B/2$ buffer pages could be used (instead of $B-2$).  Min IO cost of joining R, S where $[R] = 70$ pages, $[S] = 50$ pages, and $B = 4$? 

With $B/2$-size chunks of $R$, our formula now becomes: 

$[R] + ceil(R/(B/2))*[S]$

which gives $70 + 35*50$ = 5320 IOs. However, **we're asked for the MIN IO cost, so we can switch around R and S just fine**: 

$[S] + ceil(S/(B/2))*[R]$ which gives $50 + 25*70$ = 1800 IOs. **Thus min IO cost is 1800 IOs.**

## Grace Hash Join With Dupes

Let's say we find 2 files $G, S$ where $[G] = 110$ pages, $[S] = 25$ pages, and $B = 6$ pages. 

**80% of values in G’s join column are duplicates while the other 20% are unique values**. S’s join column consists of unique values.

Assume a perfect hash function is used to partition G and all duplicates go to partition 1. Assume an *imperfect* hash function is used to create the following partitions of S:

- S partition 1: 6 pages
- S partition 2: 3 pages
- S partition 3: 4 pages
- S partition 4: 8 pages
- S partition 5: 4 pages

Thus we read 25 pages and write 25 pages. Total: 50 IOs for partitioning pass 1.

Total IO cost of GHJ assuming that perfect hash functions are used on recursive partitioning passes?

First, we have $B-1 = 5$ partitions in all partitioning phases. We need these partitions to be $\le B-2 = 4$ pages big (since we reserve 2 pages for input/output buffer for the other relation during probing). 

Now let's align each of $R$'s partitions with S:

- R partition 1: 88 pages 
- R partition 2: 22/4 = 6 pages
- R partition 3: 6 pages
- R partition 4: 6 pages
- R partition 5: 6 pages

ALSO, this partitioning could work as well: 

- R partition 1: 88 + 5 = 93 pages
- R partition 2: 22/4 = 5 pages
- R partition 3: 5 pages
- R partition 4: 5 pages
- R partition 5: 5 pages

And standard GHJ on the rest. 

## Sort Merge Join with Indexes (!!)

We have indexes on the join columns between $R$ and $S$:

- R: Alt 2 unclustered index, height=2, 10 leaf pages.
- S: Alt 2 clustered index, height=2, 12 leaf pages.

Let $[R] = 30$, 4 records on each page ($|R| = 120$ records). Let $[S] = 40$, 5 records on each page.

**What is the average IO cost of performing Sort Merge Join on R and S?**

The cost of SMJ is (sort R) + (sort S) + ([R] + [S]). **But we have indices: our shit is already sorted.** So if we could just find a way to iterate through pages of R and S (and merge) *without needing to load them in memory*, i.e. just using indexes directly, we're good. 

Iterating over R’s sorted records: 2 to reach leaf, 12 for traversing the leaf pages, and, since we're unclustered, we need to iterate through all records (120). So iterating over R's records takes 132 IOs.  

Iterating over $S$'s sorted records: 2 to reach leaf, 10 for leaf pages, and 40 to access data pages. 

## Concussion Recovery

A property of transactions is **durability**, which refers to the idea that we will never lose the
result of some transaction.

One way to ensure this durability is to use the **force policy**, which states that we want to flush all dirtied DPs, i.e. "save" all Xact ops, before committing. 

However, the downside to using the force policy is **performance**. This makes for a lot of unnecessary writes: we'd much rather only flush pages when we have to (when page must be evicted).

In the forward processing portion of ARIES recovery (as covered in this course), we choose
to use the **no-force policy**, which allows us to avoid immediately flushing dirty pages to disk upon
commit.

However, our decision to use this policy complicates **durability**, which is a key part of ACID.

To overcome this problem, we will need to **redo** operations during recovery.

## Log Recovery: Analysis and UNDO.

Let's our shit fucked up, and we see this log up to crashpoint:

<img src="C:\Users\Kevin\AppData\Roaming\Typora\typora-user-images\image-20200514001814566.png" alt="image-20200514001814566" style="zoom: 50%;" />

Here are the Xact tables and DPT in end_checkpt:

<img src="C:\Users\Kevin\AppData\Roaming\Typora\typora-user-images\image-20200514001949099.png" alt="image-20200514001949099" style="zoom:50%;" />

**What would the Xact Table and DPT look like at the end of analysis?**

Remember the steps for Analysis:

Start at given Xact table and DPT, and start from BEGIN_checkpt.

Besides END records, add the Xact to Xact table (if necessary). Update lastLSN.

If COMMIT/ABORT record, update status in Xact table.

If UPDATE record, add page to the DPT (if necessary), set recLSN equal to the LSN (do this only once obviously).

If END record boot Xact from Xact table. 

If UNDO CLR record, it's treated like an update record, so update Xact table (and DPT if necessary)

**Standard follow process from here produces:**

 <img src="C:\Users\Kevin\Documents\BerkeleyShit\website\kmoy1.github.io\notes\cs186\CheckptTables1.PNG" alt="CheckptTables1" style="zoom:30%;" />

**Complete the undo phase, specifically,** **what log records are emitted?** 

The next record we want to undo is determined by the maximum of the priority queue of prevLSNs, which we keep updating after each undo. Each Xact's prevLSN starts at lastLSN. 

The aborting Xact with the highest lastLSN is where we start (since we wanna start as high up as possible). So we start at LSN 100. This is an abort record, we get the prevLSN and add this back to the priority
queue (add LSN 90, T2 back into the priority queue).

Next up is LSN 90: UPDATE. We undo this, write CLR log record **CLR: Undo T2 LSN 90** with prevLSN as 100, undoNextLSN as 50. add (LSN 50, T2) to queue. 

Next up is LSN 70: CLR for T1 this time. We don't undo CLRs to undo. add (LSN 10, T1) to queue. 

Next up is LSN 50: UPDATE. We undo this, write CLR log record **CLR: Undo T2 LSN 50** with prevLSN as 130, and UNDONEXTLSN as prevLSN of undone op: 20. Add (LSN 20, T2) to queue.

Next up is LSN 20: UPDATE. We undo this, write CLR log record **CLR: Undo T2 LSN 20** with prevLSN as 140, and undonextLSN as 0. **The minute our undonextLSN is 0, we know we finished undoing all Xact ops, so we write an END T2** record. 

Next up is LSN 10: UPDATE. We undo this, set prevLSN as 70 and undoNextLSN as 0. END T1.

DONE.

## SQL

Consider the following schema for Octograder:

```sql
CREATE TABLE edx_students (
    edx_id INTEGER PRIMARY KEY,
    name VARCHAR,
    email VARCHAR UNIQUE
);
CREATE TABLE assignments (
    assignment_name VARCHAR PRIMARY KEY,
    due_date TIMESTAMP
);
CREATE TABLE submissions (
    submission_id INTEGER PRIMARY KEY,
    edx_id INTEGER REFERENCES edx_students(edx_id),
    assignment_name VARCHAR REFERENCES assignments(assignment_name),
    submission_time TIMESTAMP,
    submission_file VARCHAR
);
```

NOTE: Students can submit assignments multiple times.

**What queries find the number of students who submitted hw5?**

SELECT COUNT(*)
FROM submissions
WHERE assignment_name = 'hw5';  

No. THIS ERRORS, as there's no accompanying GROUP BY. 

SELECT COUNT(edx_id)
FROM submissions
WHERE assignment_name = 'hw5'; 

Still errors. No accompanying group by. 

**SELECT COUNT(DISTINCT edx_id)**
**FROM submissions**
**GROUP BY assignment_name**
**HAVING assignment_name = 'hw5';** 

YES. This counts the number of DISTINCT IDs in all submissions for HW5, i.e. the number of students that submitted it. 

**What queries find the submission_id for all late hw1 submissions (disregard slip mins)?**

```sql
SELECT submission_id
FROM submissions S, assignments A
WHERE S.assignment_name = A.assignment_name
AND S.assignment_name = 'hw1'
AND S.submission_time > A.due_date;
```

YES. This links a submission to assignment, which allows to compare submission time to due date (and thus determine lateness) via ```S.submission_time > A.due_date```

```sql
SELECT submission_id
FROM submissions S INNER JOIN assignments A
ON S.assignment_name = A.assignment_name
AND S.submission_time > A.due_date
WHERE S.assignment_name = 'hw1';
```

YES: This also works. INNER JOIN ON does the same shit. The ON and AND is basically the same fucking thing lol

```sql
SELECT submission_id
FROM (
    SELECT due_date
    FROM assignments
    WHERE assignment_name = 'hw1'
) AS A1, submissions S
WHERE S.submission_time > A1.due_date;
```

Unfortunately, this query doesn't restrain non-hw1 submissions. The inner query just gets a single-column table of (repeated) rows of the hw1 due date. Joining this with all submissions submitted past this due date will give all late submissions but is also going to give every submission of hw2, hw3, hw4........

**What queries find all the emails of students who didn’t submit hw5?**

```sql
SELECT email
FROM (
    SELECT edx_id
    FROM submissions
    WHERE assignment_name = 'hw5'
) AS S FULL OUTER JOIN edx_Students E
ON S.edx_id = E.edx_id
WHERE S.edx_id IS NULL;
```

YES: First, we make a table S, which gives all EDX IDs (of students) who submitted hw5. Then, we FULL OUTER join this with edx_students, so that all EDX IDs in edx_students (has every student's EDX ID) can match up with those who submitted. The records where the left join has edx_id NULL means that those students EDX ID didn't show up in S, and thus they didn't submit hw5!

```sql
SELECT email
FROM edx_students E LEFT OUTER JOIN (
    SELECT edx_id
    FROM submissions S
    WHERE S.assignment_name = 'hw5'
) AS S1
ON E.edx_id = S1.edx_id
WHERE E.edx_id IS NULL;
```

Unfortunately, this doesn't work because a left outer join matches every left record with all matches in right, so it is impossible for left records (E) to contain null values.

