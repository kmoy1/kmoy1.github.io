# Query Ops Run

Consider a relation $R(a, b, c)$ with $|R| = 1000$ tuples. We have an index on column $a$ with 50 unique values in the range $[1, 50]$ and an index on $b$ with 100 unique values in the range $[1, 100]$. (No index on $c$).

We can now *estimate* the output, i.e. number of tuples produced, by certain queries. Estimate below.

**Q1: SELECT * FROM R**

**A1: 1000 tuples**. Selects everything.

**Q2: SELECT * FROM R WHERE a = 42**

**A2: 20 tuples**

We now need to calculate the selectivity of the selection operator WHERE a = 42. In this case, selectivity is 1/# unique vals = 1/50, or around 1 tuple output per 50 scanned. Thus, around 1000 * (1/50) = 20 tuples produced. 

**Q3: SELECT * FROM R WHERE b = 42**

**A3: 10 tuples**. Now selectivity is 1/100, so 10 tuples.

**Q4: SELECT * FROM R WHERE c = 42**

**A4: 100 tuples**. Default selectivity is 1/10.

**Q5: SELECT * FROM R WHERE a <= 25**

**A5: 500 tuples** 

Selectivity of $a \le 25$ = Selectivity of  $a = 25$ + selectivity of $a < 25$. 1/50 + (25-1)/(50)  = 1/2. So 1/2 * 1000 = 500. We can also see that $a \le 25$ consists of half of the range of a, so 1/2 is sel

**Q6: SELECT * FROM R WHERE b <= 25**

**A6: 250 tuples**. $b \le 25$ makes 1/4 selectivity

**Q7: . SELECT * FROM R WHERE c <= 25**

**A7: 100 tuples**. Selectivity default 1/10.

**Q8: SELECT * FROM R WHERE a <= 25 AND b <= 25**

**A8: 125 tuples**. Selectivity is 1/2 * 1/4 = 1/8. 

**Q9: SELECT * FROM R WHERE a <= 25 OR b <= 25**

**A9: 625 tuples**. 

The selectivity of OR operators requires the inclusion-exclusion thing: Selectivity of a <= 25  + selectivity of b <=25 - selectivity of both = 1/2 + 1/4 - 1/8 = 5/8.

**Q10: SELECT * FROM R WHERE a = b**

**A10: 10 tuples ** Selectivity for column equality (and thus join conditions) is 1/(max values of A, max values of B). So we got 1/max(50,100) = 1/100.

**Q10: SELECT * FROM R WHERE a = c**

**A10: 20 tuples ** No information on c, so we treat it like it doesn't count: Sel = 1/max(a) = 1/50. 

## Single Table Access Plans

Given (quite a bit of) information on query, relations, and indexes on specific table columns, we can estimate optimal System R query plans!

```sql
SELECT *
FROM R, S, T
WHERE R.b = S.b AND S.c = T.c
AND R.a <= 50;
```

where $R(a,b), S(b,c), T(c,d)$ are our relations. Additionally, $[R] = 1000, |R| = 10000, [S] = 2000, |S| = 40000, [T] = 3000, |T| = 30000$. Assume we have these indexes built:

- Alt 2 unclustered index on R.a with 50 leaf pages
- Alt 2 clustered index on R.b with 100 leaf pages
- Alt 2 clustered indexes on S.b, T.c, and T.d (leaf page counts aren’t relevant)

Assume 2 IOs to reach the level above a leaf node (index height = 2) and that no index or data pages are ever cached. All indexes have keys in the range [1, 100] with 100 distinct values. 

For pass 1, we need to do single-table accesses (reads from disk) in an optimal way. 

**Q1: How many IOs to full scan $R$?**

**A1: 1000 IOs**. Full scans always read every page. 

**Q2: How many IOs to index scan $R$ on $R.a$**?

**A2: 5027 IOs ** Since R.a has an alt-2 unclustered index, our IO cost is (height of index) + (num of leaf pages read) + (num of data pages read). First, calculate our selectivity for R.a <= 50, which is 1/2. Remember selectivity will cut down the number of leaf and DPs read. We got 50 leaf pages on this index, and 1000 data pages, and 10000 records. Because unclustered, we gotta unfortunately take up an IO per record (disorganized smh).

So total, we got 2 + 0.5(50) + 0.5(10000) = 5027 IOs. 

**Q3: How many IOs to index scan $R$ on $R.b$**?

**A3: 1102 IOs**

R.b has an alt-2 clustered index. We do NOT use a join condition for these single table accesses. So with no conditions we assume selectivity 1, and we need to read all leaf/data pages. 

So total, we got 2 + (100) + (1000) = 1102 IOs. 

**Q4: How many pages from R advance at pass 0 for all of these access plans?**

**A4: 500 pages**. Our selectivity, which we calculated from our index scan on R.a, is 1/2, so 500 pages make it through.

Now let's say System R comes up with these plans for S and T: 

- Full scan on S: 2000 IOs
- Index scan on S.b: 1500 IOs
- Full scan on T: 3000 IOs
- Index scan on T.c: 3500 IOs
- Index scan on T.d: 3500 IOs

**Q5: Which plans advance?**

For R: Full Scan R is optimal for R, but index scan on R.b also advances because it produces an interesting order.

For S: Index scan on S.b is optimal, and it's also an interesting order. 

For T: Full scan on T is optimal. Index scan on T.c also advances because it has interesting order.

## Multi-Table Plans

Assume $B = 52$. Assume the access plan for $R$ is the index scan on R.b (S only had one access plan that advanced). We can calculate the IO cost for specific join algorithms on tables: 

**Q1: IO cost of $R \bowtie S$ on R.b = S.b, using BNLJ**?

**A1: 20500 IOs ** BNLJ's cost is $[R] + ceil([R]/B-2)*[S]$. However, note $[R] = 500$ now because of selectivity (index scan on R.b). So we got 500 + 500/50 * 2000 = 20500 IOs. 

**Q2: IO cost of $R \bowtie S$ on R.b = S.b, using SMJ?**

**A2: 2500 IOs**. Both these tables are sorted- so we just need to merge now, which costs $[R] + [S]$ IOs. 

Now for passes 2 to n it's a little more nuanced: we have to consider all length-i sets of tables to join, and we ONLY consider ones with join conditions (we avoid cross joins!)

For reference: 

```sql
SELECT *
FROM R, S, T
WHERE R.b = S.b AND S.c = T.c
AND R.a <= 50;
```

So for the above query we only got join conditions for {R,S} and {S,T}. So we'll only consider those! The optimal + optimal-interesting rule still applies as before. 

We also need to keep track of join conditions we've **already used** in joining tables in join plans: we can't just reuse a join conditions to say that that a (sort merge) join produces an interesting order, because we're NOT using that join condition again (we used it already for our join itself!)

