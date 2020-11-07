# Decomposition of Graphs

Graphs represent (and solve!) a lot of practical problems. A classic example is a map: how many colors do we need to color every state in the US such that adjacent states have different colors?

We can represent this with graphs! Our graph has one vertex for each state, edges between neighboring states. Now, can we assign a color to each vertex such that no *adjacent* vertices (connected by one edge) are colored the same? 

We call this the **graph coloring problem**, and it actually has many practical uses. Suppose a university needs to schedule examinations for all its classes and wants minimize time slots used. The only constraint is that two exams cannot be scheduled concurrently if some student will be taking both of them. Draw one vertex per exam. Then, draw an edge between two vertices if a conflict exists (student ). Think of each time slot as having its own color. Then, assigning time slots is exactly the same as coloring this graph!

Some basic operations on graphs arise with such frequency, and in such a diversity of contexts, that a lot of effort has gone into finding efficient procedures for them. This chapter is devoted to some of the most fundamental of these algorithms—those that uncover the basic connectivity structure of a graph

