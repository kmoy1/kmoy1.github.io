# NP-Complete Problems: Part 2

Last note we briefly introduced a shitload of problems that all had no polynomial-time (efficient) algorithm to solve it. Now with that in our arsenal we'll have plenty of material to actually discuss what NP-Complete problems are.

To sum up search problems in a sentence: some can be solved efficiently, while others can't.

<img src="C:\Users\Kevin\AppData\Roaming\Typora\typora-user-images\image-20200515000821680.png" alt="image-20200515000821680" style="zoom:33%;" />

The "easy problems" column has various algorithms: dynamic programming, network flow, graph search, greedy. There are many (different) reasons **why** these problems are easy. 

On the other hand, **all hard problems (left column) are all hard for the same reason**! At their core, they're all equivalent.

## P and NP

We defined a search problem by efficient checker algorithm ```C(I,S)```.

**The set of *all* search problems is NP** (nondeterministic polynomial time).  Many problems in NP are solvable in polynomial time (relative to input I). If I has a solution, the algorithm returns such a solution or reports none exist. 

**The set of all polynomial-solvable search problems is P** (polynomial).

Now comes an infamous unsolved question. Up to now, we just haven't been able to *find* any solutions to those problems on the left. Can those problems on the left column be *proved* as unsolvable in polynomial time? In other words, is $P \neq NP$? Probably. It's hard to believe a simple trick will crack all these hard problems, famously unsolved for decades and centuries. 

There is a good reason for mathematicians to believe that  $P \neq NP$: finding a proof for a given mathematical assertion is a search problem and is therefore in NP. So if $P = NP$, we could efficiently prove any theorem, and render all mathematicians jobless.

## Reductions

How do we *know* there isn't an efficient algorithm for NP-hard problems (besides human failure)? 

We can use reductions, which *translate* one search problem to another and prove core equivalence. They prove all those left-table columns are actually the same problem, except in different contexts. 

Reductions also show that **ALL these problems are the hardest search problems in NP**. So if even one of them has an efficient algorithm, all of them do. If we assume $P \neq NP$, all these search problems are NP-hard. 

### Search problem reduction

A reduction from search problem A to search problem B is specified by 2 polynomial-time algorithms $f$ and $h$:

- $f$ transforms any instance I of A into an instance f(I) of B
- $h$ maps any solution S of f(I) back into a solution h(S) of I.

If f(I) has no solution, then neither does I. Any algorithm for B can be converted into an algorithm for A by bracketing it between f and h.

### NP-Complete

A search problem is **NP-complete** if **all** other search problems, hard or easy, reduce to it.

The left column, i.e. problems like 3SAT and TSP, are all NP-Complete problems. 

## Two Ways to use Reductions

A reduction from problem A to problem B basically sums to this: If we know how to solve B efficiently, we can use this knowledge to solve A, since they are at core the same problem.

However, reductions from A to B can also mean: **if we know A is hard, we can prove that B is hard too**! 

Denote a reduction from A to B as  $A \rightarrow B$. Difficulty "flows" in the direction of the arrow, while efficient algorithms move in the opposite direction. Thus we know NP-complete problems are hard: all other search problems reduce to them, and thus each NP-complete problem contains the complexity of all search problems. **So if even one NP-complete problem is in P, then P = NP.**

Reductions also have a *composing* property:

If $A \rightarrow B$ and $B \rightarrow C$ then $A \rightarrow C$. 

We know any reduction is defined by $f$ and $h$. If $(f_{AB}, h_{AB})$ and $(f_{BC}, h_{BC})$ define $A \rightarrow B$ and $B \rightarrow C$, respectively, then $A \rightarrow C$ can be defined by $f_{BC} ◦ f_{AB}$ (maps A to C) and $h_{AB} ◦ h_{BC}$ (maps solution of C back to solution of A).

Thus once we know $A$ is NP-complete, if we can reduce it to $B$, we prove that $B$ is also NP-complete. Such a reduction establishes that all problems in NP reduce to B, via A.

## Factoring

Factoring, or finding all prime factors of an integer, is also a hard problem. 

But factoring is difficult for a different reason. Nobody believes that factoring is NP-complete. There's never a "no solution" to be reported- a number always has some prime factorization.



