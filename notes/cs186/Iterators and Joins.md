# Iterators and Joins

Let's look at a SQL JOIN statement, when we join two tables $R$ and $S$.  

For each record $r_i$ in $R$ we find all records $s_j$ in $S$ that match the join condition we have specified and write $< r_i , s_j >$ as a new row in the table, horizontally concatenated. This will produce one new relation based on join condition matches.

