# Sort-Merge Join

If we want to join $R$ and $S$, sometimes sorting $R$ and $S$ first helps, *especially* if we want our joined table to be sorted on a specific column. The following explains the SMJ algorithm:

1. Sort $R$ and $S$. 
2. Initialize two markers $r_i$ and $s_j$ at the start of $R$ and $S$, and advance until we get a match ($\theta(r_i,s_j) = \text{True})$. Join and write record to disk.
3. **Mark** $s_j$ and advance $s_j$ to subsequent rows of $S$, i.e. $s_{j+1}, s_{j+2}...$, match with $r_i$. Join and write these records to disk. Stop when you hit a row in $S$ that's not a match.
4. Advance $r_i$ and reset $s_j$ to the marked spot, and repeat step 2. We want to mark this in case we have **duplicate** records of $R$ each  joining to multiple records in $S$.

**Average IO cost of SMJ: (sort $R$) + (sort $S$) + ($[R] + [S]$)**

(Java) SMJ pseudocode:

```java
while (r != null and s != null) {
	if (!mark) {
		while (r < s) { advance r }
		while (r > s) { advance s }
		// mark start of block of S
		mark = s
	}
	if (r == s) {
		result = <r, s>
		advance s
		yield result
	}
	else {	
		reset s to mark
		advance r
		mark = NULL
	}
}
```

Ultimately, this produces a table sorted on a column of $S$. 