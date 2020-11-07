# NP-Complete Problems: Part 1

All algorithms so far (shortest paths, MSTs, etc.) are ***efficient***, because since runtime grows polynomially to input size.

The efficiency of these algorithms is important because our possibility space is exponential.

- n boys can be matched with n girls in n! different ways
- a graph with n vertices has $n^{n−2}$ spanning trees.
- A typical graph has an exponential number of paths from s to t. 

Technically, we can solve all these problems by just checking each possible solution one by one. But an algorithm whose running time is $2^n$ (or worse) is useless in practice. Efficient algorithms are all about cleverly narrowing down this search space!

We still haven't solved everything though. Some problems we just haven't found efficient solutions for yet. **The fastest algorithms we know for such problems are all exponential**—not much better than an *exhaustive search*. We now introduce some important examples. 

## Satisfiability

Satisfiability (SAT) is a canonical hard problem. 

Here's an example problem: (x ∨ y ∨ z) (x ∨ -y) (y ∨ -z) (z ∨ -x) (-x ∨ -y ∨ -z). 

This is a *Boolean formula* in **conjunctive normal form** (CNF). Clauses are the parentheses, each consisting of a **disjunction of literals**, where a literal is some variable or negation. The goal: **finding a satisfying truth assignment** is an assignment of Booleans to all literals such that *each clause evaluates to true*. 

**SAT Problem**: for any Boolean Formula, find a satisfying truth assignment or report no solution.

Can we solve our problem given above?  (x ∨ y ∨ z) (x ∨ -y) (y ∨ -z) (z ∨ -x) (-x ∨ -y ∨ -z)

It's easy to argue that no solution for this *specific* problem exists: the three middle clauses force x, y, z (all literals) to have the same value, which makes the last clause or the first clause false. 

But how do we decide this in general? We can obviously test each truth assignment but this is absolutely an exponential possibility space: in fact, for $n$ literals there are *exactly* $2^n$ possible truth assignments.

A **search problem** challenges us, given instance $I$ (some input problem, like the Boolean formula in CNF), to find some  solution $S$. If no such solution exists, we must say so.

Search problems also require solutions be **quickly verifiable**. $S$ must be concise, length(S) polynomially bounded by input problem $I$. Clearly, in SAT S is just an assignment to variables, so this is true. 

Formally, a solution $S$ is concise if there's some polynomial-time algorithm ```C(S,I)``` that checks if S is a solution of I. For SAT, ```C(S,I)``` is very simple indeed: just check if every clause in $I$ evaluates to true given the assignment $S$.

We might actually think of this (efficient) checker algorithm as *defining* the search problem, i.e. each algorithm is specified by some ```C(S,I)```. Each checker algorithm C is unique to the search problem.

Basically, nobody has really found an efficient (non-exponential) solution to general SAT yet. 

However, we have come up with two algorithms that *do* solve specific versions of SAT (SAT with some constraints)

- **Horn Formulas** are Boolean formulas where all clauses contain at most one positive literal (everything else negated). A greedy algorithm has been developed to solve Horn Formulas.
- **2SAT** problems have formulas with only two literals in each clause. We can actually construct a graph from this formula, and by finding the SCCs, find a solution in linear time.
- **3SAT** has formulas with three literals- hard again.

## Traveling Salesman Problem (TSP)

Given $n$ vertices labeled 1 to n, all $n(n-1)/2$ distances between every pair of vertices, and a distance budget $b$, we want to find a **tour** (cycle that hits every vertex once) of total distance $\le b$. Another way of looking at the solution is some permutation of vertices $\tau(1)...\tau(n)$ where $\tau(i)$ specifies the vertex (number) traveled to at step $i$ . If we specify $d(x,y)$ as the distances between vertices $x$ and $y$ in our graph, our goal:

$d(\tau(1), \tau(2)) + d(\tau(2), \tau(3)) + ... + d(\tau(n), \tau(1)) \le b$

Defining TSP as a search problem is easy: given a weighted graph instance, find *any* tour within the budget or report no solution.

But isn't TSP an *optimization* problem? There's only a single minimal solution- why make it a search problem, which implies there are multiple tours that work?

There's actually a good reason which we'll dive into later in *reduction*, which proves these problems are actually at heart the same problem. Optimization problems and search problems actually reduce to each other, so their difficulty stays the same! 

Think about it like this for TSP: If we find an algorithm that gives us an optimal tour, we can easily solve the search problem by just checking if that tour is within budget (if it's not, no solution). Conversely, if we find an algorithm that gives us a tour with cost less than some arbitrary budget $b$ (the search problem), we can just set b as the actual optimal cost and get our optimal tour (solving optimization!). So knowing algorithm for optimization gives us algorithm for search, and vice versa!

We can easily find the optimal tour cost, by the way, by binary search on all the tour costs. 

Why is a **budget** needed in this problem if we're actually searching for an *optimal* tour cost? The solution to a search problem should be easy to check: polynomial-time checkable. Specifying a budget makes it easy to check a solution: just add up tour edges and ensure it's less than $b$. It's not so easy to check if a solution is *optimal* or not. 

Unfortunately, we also don't have any polynomial-time algorithms (yet) to solve general TSP. Of course, we could try all (n − 1)! tours, and even use dynamic programming, but both are still exponential algorithms. 

What about the MST problem? We can make it a search problem given a distance matrix and a bound $b$, and are asked to find a tree $T$ with total weight ≤ b. Search TSP can be thought of as a harder version of search MST in this case: for TSP, our "tree" is not allowed to branch and thus becomes a path. This extra restriction makes TSP much harder to solve.

## Euler, Rudrata

**Euler paths** are paths in a graph that hit each edge exactly once. We know there's a Euler path in a graph if BOTH:

- the graph is connected
- every vertex except start+end vertices has even degree

Search problem form: Given a graph, find a path that contains each edge exactly once or report none exists. This search problem can be solved in polynomial time. 

Knights tour: Can a knight hop and touch each square on the board exactly once? We can model this as a graph with 64 vertices, and each vertex (square) connected by edge if separated by a knight move. 

**Rudrata cycle**: cycle that hits all ***vertices*** exactly once. Search problem form: given a graph, find a cycle that visits each vertex exactly once - or report no solution. Isn't this a simpler TSP? Yes. And still, no polynomial-time algorithm has been found for it. 

**Rudrata path**: find a ***path*** that hits all vertices exactly once (doesn't need to cycle back to starting vertex). We will later prove that finding the Rudrata path and Rudrata cycle are at heart the same problem- both unsolvable in polynomial time.

## Cuts and Bisections

A **cut**: set of edges whose removal leaves a graph disconnected- i.e. creates 2 or more disjoints sets of vertices.

**Minimum cut problem**: given graph G and budget b, find a cut with $\le b$ edges. This problem is solvable in polynomial time with $n − 1$ max-flow computations: if we give each edge capacity 1, the smallest max flow between some fixed node and every other node corresponds to the smallest cut. In many graphs, the smallest cut is just all edges adjacent to a vertex, leaving it as a singleton. 

What about (small) cuts that partition vertices into (nearly) equal-sized sets? The BALANCED CUT problem: given a graph with n vertices and a budget b, partition the vertices into two sets S and T such that |S|, |T| ≥ n/3 and such that there are at most b edges between S and T. This is another hard problem.

We can represent a digital picture as a graph of pixels with different "parts" of a picture (rocks, sky, grass) as subgraphs with a lot of interconnecting edges. That way, we can try to use multiple balanced cuts to separate these parts of the picture. 

## Integer LP

Linear programming is efficiently solvable both in practice and in theory. INTEGER LINEAR PROGRAMMING (ILP) is when solutions must be integral- this is *not* efficiently solvable. ILP search problem form: Given linear inequalities Ax ≤ b, where $A \in  R^{m \times n}$ and $b \in R^m$; objective function $c \in R^n$; and **goal** $g$ (like a TSP budget, but opposite: we want to *surpass* this goal), we want to find an ILP solution $x$ that contains nonnegative integers such that $Ax ≤ b$ and $c · x ≥ g$. 

However, we can simplify further: constraint $c · x ≥ g$ can be included into $Ax ≤ b$. 

So, updated ILP search problem definition: given $A$ and $b$, find a nonnegative integer vector $x$ satisfying $Ax ≤ b$, or report that none exists. No efficient algorithm exists yet. 

A special case of ILP are ZERO-ONE EQUATIONS (ZOE): given a *binary* $A \in R^{m\times n}$, find a *binary* vector $x$ satisfying $Ax = 1$, where 1 is a vector of m 1s. This is indeed a special case of ILP, and is also hard. 

## 3D Matching

BIPARTITE MATCHING problem: given a **bipartite graph** with $n$ nodes on each side, find a set of $n$ disjoint edges (match up each node on each side), or declare no matching. Reduction to maximum flow is a way to solve. 

3D MATCHING considers matchings on TRIPARTITE graphs now. Consider $n$ boys and $n$ girls, but also $n$ pets. Intuitively, a triple (b, g, p) means that boy b, girl g, and pet p get along well together. **We want to find n disjoint triples and thereby create n harmonious households.** 

## Independent Set

INDEPENDENT SET problem: Given a graph and integer $g$, find $g$ **independent vertices**: a set vertices which have no edges between any two. The independent set problem can be solved efficiently on trees, but for general graphs no polynomial algorithm is known.

## Vertex Cover

VERTEX COVER problem: Given a graph and a budget $b$, find $b$ vertices that cover (touch) every edge. (Kinda related to INDEPENDENT SET...)

VERTEX COVER is a special case of SET COVER problem: Given a set $E$, several subsets of $E$  as $S_1,... S_m$, and  budget $b$,  select $b$ of these subsets that **union to $E$.** So VERTEX COVER is just when E = edges of a graph, and for each vertex $i$, subset $S_i$ contains the adjacent edges to vertex $i$. 

## Clique

CLIQUE problem: Given a graph and a goal $g$, find a set of $g$ vertices such that all possible edges between them are present. 

## Longest Path

LONGEST PATH PROBLEM: Given a graph G with nonnegative edge weights and $s$ and $t$, along with a goal $g$, find a s-t path with total weight $\ge g$. Obviously we need to stop "cheating" so we require a simple path with no repeated vertices. No efficient algorithm known.

## Knapsack, Subset Sum

KNAPSACK problem: Given $n$ integer weights $w_1,...,w_n$ and $n$ integer values $v_1,...v_n$ (for n items), as well as weight capacity $W$ and a goal $g$, find a set of items whose total weight $\le W$ and total value $\ge g$. 

Using DP, we've come up with an algorithm with runtime $O(nW)$. This is unfortunately exponential, since it involves $W$ rather than $\log W$. The exhaustive algorithm looks at all $2^n$ subsets of items. No polynomial algorithm exists. 

UNARY KNAPSACK problem: KNAPSACK but integers are coded in unary—for instance, by writing IIIIIIIIIIII for 12.  Interestingly, now we *do* have a polynomial algorithm to solve!! How does just changing the representation of numbers affect our search process at all?? 

Suppose now each item’s value $v_i$ is equal to its weight $w_i$: (in binary), and goal g = capacity W. This special case is the same as finding a subset of a given set of integers that adds up to exactly W. It's a special case of KNAPSACK, so it can't possibly be harder. But could it be polynomial? This problem is called SUBSET SUM, and is also very hard. :(

Conceptually simple problems like SUBSET SUM and 3SAT are invaluable in reductions we make between problems.