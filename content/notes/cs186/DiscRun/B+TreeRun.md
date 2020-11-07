# B+ Trees Discussion Run

Let's say we have this B+ Tree below. It is **order 1**, so each node can have 1 or 2 entries, and thus 2 or 3 pointers. Leaf nodes can have up to 2 entries. 

<img src="C:\Users\Kevin\Documents\BerkeleyShit\website\kmoy1.github.io\notes\cs186\B+Tree.PNG" alt="B+Tree" style="zoom:33%;" />

**Q1: Max number of *insertions* we can do without changing the height of the tree?**

**A1: 12 insertions**

Notice the height of this tree is 2. The maximum number of entries for a tree with height $h$ and order $d$ is $2d*(2d+1)^h$. Thus the maximum number of entries for this tree of height 3 (order obviously can't change) is $2*3^2 = 18$. 

Remember that **entries are all in the leaf nodes**. Right now there are 6 entries. Thus we can add a maximum of 18-6=12. 

**Q2: Min number of insertions to *change* tree height?**	

**A2: 3 insertions**

We know B+ tree height increases iff the root splits. So we just need to find an insertion sequence that makes the root split as fast as possible. One such insertion sequence is 1,4,5.

## Indices

Remember a **clustered index** orders the data pages (generally) by index. An **unclustered index** has no such ordering and results in a fuckload of pointers everywhere. This means range queries on a unclustered index are much more inefficient: there wouldn't be any caching and we'd need about an IO **per record,** which is ghastly for large relations. 

**Alternative 1 index**: stores entire rows (records) in leaf pages.

**Alternative 2 index:** (key, recordId) pairs. Each key occurs once. 

**Alternative 3 index:** (key, [rid1, rid2,...]) pairs. Each key occurs once. 

**Q1: Can you have two clustered indices on separate columns? Does it help?**

**A1: You can, but it's kind of inefficient.** You'd have to keep two copies of the data (one for each index), and keep them updated.

## Unclustered Shithole

``` sql
CREATE TABLE Submissions (
    record_id integer UNIQUE,
    assignment_id integer,
    student_id integer,
    time_submitted integer,
    grade_received byte,
    comment text,
    regrade_request text,
    PRIMARY KEY(assignment_id, student_id));
CREATE INDEX SubmissionLookupIndex ON Submissions (
    assignment_id, student_id);
```

Given the above submissions table and an Alt-2 *unclustered* index on TWO columns (assignment_id, student_id), with height 3. Assume table takes 12 MB on disk, and page size is 64 KB.

**Q1:  How many IOs to full scan Submissions?**

**A1: 192 IOs**. A full scan ignores indices and always reads in every page from the table. We have 12 * 1024 / 64 = 192 pages in the submissions table, so 192 IOs. 

**Q2:How many IOs to run UPDATE Students SET grade_received=85 WHERE assignment_id=20 AND student_id=12345;? **

**A2: 6 IOs**

Now we are simply updating our index on a *specific* key, so unclustered/not doesn't really matter since we're just following pointers. 3 IOs to traverse to leaf. 1 IO to read leaf and find data pages with records. 2 IOs to read + write data pages. 6 total.

**Q3: In the *worst case*, how many IOs for equality search on grade_received (SELECT * WHERE grade_received = 100) ?**

**A3: 192 IOs (Full Scan)**

We assume any record can match grade_received in the worst case. So we have to scan all records, and thus all pages. Full Scan => 192. 

## Bulk Loading

​	



