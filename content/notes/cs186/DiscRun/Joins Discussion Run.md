# Joins Discussion Run

Let's say we had two tables to join:

- Companies: (company_id, industry, ipo_date)
- NYSE: (company_id, date, trade, quantity)

We want to join on C.company_id = N.company_id. Assume $B=20$ buffer pages, company_id is a primary key, and for each tuple in Companies, there's 4 matching tuples in NYSE. 

Companies: $[C] = 50$ pages, $|C| = 2500$ tuples total (50 tuples per page)

NYSE: $[N] = 100$ pages,  $|N| = 10000$ tuples total (100 tuples per page)

Finally, assume we got 2 *unclustered* B+ indexes on C.company_id and N.company_id. Assume 2 IOs to access a leaf and find record. 

**Q1: How many IOs for simple nested loops join?**

**A1: 250050 IOs. Yikes.**

This is by far the shittiest join: find all record matches in $S$ for each record in $R$. Thus we have $[R] + |R|[S] = 50 + 2500 * 100 = 250050$ IOs. 

**Q2: How many IOs for block nested loops join?**

**A2: 350 IOs**

This one's much better: we're now using $B-2$ = 18 page chunks for $R$. So for each of these chunks in $R$, we'll iterate through all pages of $S$ once. This gives $[R] + \lceil ([R]/B-2) \rceil [S] = 50 + ceil(50/18) * 100 = 350 IOs$.

**Q3: How many IOs for index nested loops join?**

**A3: 15050 IOs.**

INLJ goes through each record in $R$ and immediately outputs matching records in $S$ via index. So the IO cost is $[R] + |R|∗\text{(cost to look up matching records in S)}$. What's the cost to look up records in our NYSE index? 2 IOs. 

However, we must now also account for the fact that we have FOUR matches per record in $R$: So we add 1 IO for each of these records, since our index is unclustered. In total, that gives us $50 + 2500 * (6) = 15050$ IOs.

**Q4: How many IOs for (ordinary) sort merge join?**

**A4: 750 IOs**

The cost of SMJ is (cost to sort $S$) + (cost to sort $R$) + ($[R] + [S]$).

It takes $4*50 = 200$ IOs to sort $R$ (2 passes). It takes $4*100 = 400$ IOs to sort $S$ (still 2 passes). Finally, joining takes 100 + 50 = 150 IOs. Thus in total, SMJ on NYSE and Companies takes 200 + 400 + 150 = 750 IOs. 

**Q5: How many IOs for (Grace) hash join?**

**A5: 492 IOs**.

Remember GHJ's cost was (cost of hashing) + (cost of NHJ on partitions). We have $B-1 = 19$ partitions.

For $S	$, we simply read $[R] = 100$ pages (100 IOs) and write $19(6) = 114$ pages (114 IOs). For $R$ we read $[R] = 50$ pages and write $19(3) = 57$ pages. Finally, since all of these partitions fit into memory, we simply build and probe for 114 + 57 = 171 IOs. Thus in total, for partitioning (cost of hashing) and build + probing (NHJ on partitions), we have 321 + 171 = 492 IOs. 

## Grace Hash Join

Let's say Catalog and Transactions tables need to be joined. $[C]=100, |C| = 2000$. $[T] = 50, |T| = 2500$. 

Also assume $B = 10$. 

**Q1: How many partitioning passes for GHJ?**

**A1: 1 pass ** 

We have $B-1 = 9$ partitions per pass. We also need these partitions to fit in memory, i.e. $\le B-2 = 8$ pages. We stop immediately if either tables' recursive partitions stop. Immediately, we see that $ceil(50/9) = 6 \le 8$, so we only need 1 pass. 

**Q2: How many IOs for GHJ?**

**A2: 474 IOs**

Since we know we only need 1 pass: 

We read 100 pages, write 12*9 = 108. Read 50 pages, write 6 * 9 = 54. Building and probing takes 108 + 54 = 162 IOs.

Thus total, we need 100 + 108 + 50 + 54 + 162 = 312  + 162 = 474 IOs. 

Now consider above, when we only have $B=8$ buffer pages. 

Q1: How many partitioning passes for GHJ?**

**A1: 2 passes now**.

Now we need our partitions to be smaller than 6. Unfortunately, $ceil(50/7) = 7 > 6$. We need 2 passes.  



