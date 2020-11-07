# NP-Complete Reductions: Part 1

We shall now prove all our problems we discussed in the previous notes- TSP, 3SAT, Independent Set, etc.- are all NP-complete, by proving they all reduce to each other. 

## Rudrata S-T Path -> Rudrata Cycle

RUDRATA CYCLE problem: Given a graph, find a **cycle** that hits each vertex exactly once. 

RUDRATA (s,t)-PATH problem: Given a graph and two specified s,t vertices, find a path starting at s and ending at t that hits each vertex exactly once. 

Well, you can see RUDRATA CYCLE is just a special version of Rudrata (s,t)-PATH. Is RUDRATA cycle easier? No, and we prove this by reduction. 

First, we must map an instance of A to B via $f$. We define such instances first: Rudrata (s,t)-Path as (G = (V, E), s,t) and Rudrata cycle as (G' = (V',E')). Now we map st path to cycle: Simply add an additional vertex x and two new edges {s, x} and {x,t} to graph G (st-path graph), and that's our new instance G' (of Rudrata cycle). So we've completed $f$, which maps an instance of A to B.

Now we tackle $h$: given a solution to B, map it back to a solution of A. This means given *any* Rudrata cycle in G', how do we map it to a Rudrata (s,t)-path in G? Easy: delete the edges {s, x} and {x,t} from the cycle. We can tell this works, but how?

We have to show $h$ works when B has a solution AND when it doesn't: 

- Solution found (Rudrata cycle exists): Added vertex x has only two neighbors, s and t, so any Rudrata cycle in G' traverse the edges {t, x} and {x, s} **consecutively**. The *rest* of the cycle then hits every other vertex from s to t. Thus deleting the edges {t, x} and {x, s} from the cycle gives a Rudrata path from s to t IN our original graph G.
- No solution found (Rudrata cycle DNE): We must show a RUDRATA (s,t)-PATH in G has no solution either. We try proving the **contrapositive**: if a Rudrata (s,t)-path is in G, then Rudrata cycle is in G'. Just add edges {t, x} and {x, s} to G, which creates the cycle. 

Finally, we must check $f$ and $h$ take time polynomial in the size of the instance A: (G,s,t). This is easy: f only adds a vertex and two edges, and $h$ only deletes two edges. Both take constant time. 

## 3SAT -> Independent Set

One can hardly think of two more different problems. 

3SAT problem: Given Boolean formula with clauses with $\le 3$ literals, find a satisfying truth assignment. 

INDEPENDENT SET problem: Given a graph and target $g$, find some set of $g$ pairwise non-adjacent vertices. 

First, we must map an instance of A to B via $f$. Mapping a Boolean formula to a graph with a target. What???

A satisfying truth assignment has at ONE literal from each clause as true. But these have to be *consistent*: if we choose  -x as True, we cannot choose x as True in another clause. So if we find a *consistent choice of literals*, one from each clause, this gives us a truth assignment. 

So let us now create our instance mapping, by representing a clause (x ∨ y ∨ z) as a triangle graph with 3 vertices labeled x, y, z. Why triangle? A triangle is a fully connected graph, we must pick ONE (to assign true) in the independent set. A clause with two literals is just a single edge joining 2 literal nodes. (Clauses with one literal must be true and can be ignored). In the resulting graph, an independent set needs to pick *one* literal from each group. We want to enforce one literal picked from each clause: so we set goal $g$ = number of clauses. 

Now, we must enforce preventing conflict (choosing x and -x) in different clauses. So we draw an edge between any two vertices that correspond to opposite literals. Now our $f$ is complete: we've mapped 3SAT to a valid INDEPENDENT SET graph with $g$ as the number of clauses!

So now we need to construct $h$: given a solution to INDEPENDENT SET, i.e. an an independent set S of g vertices in G, efficiently recover a truth assignment. Assign literal $x$ a value of true if S contains vertex $x$, and a value of false if S contains a vertex $-x$ (if S contains neither, then x can take either value). S has $g$ vertices, so it'll have one vertex per clause. Thus our created truth assignment satisfies those literals, and thus satisfies all clauses.

We still need to prove that given NO solution to INDEPENDENT set, we can't have a satisfying truth assignment. Again, proving the contrapositive: Given a satisfying truth assignment, there's an independent set in the correlated graph. This is easy: for each clause, pick one literal marked true, and add the corresponding vertex to the set. S must be independent: the only way it won't be is if we have connected vertices in our set, and because we assumed a satisfying truth assignment this can never happen: x and -x can certainly never be in our set, and we only pick one vertex from each clause. 

## SAT -> 3SAT

These reductions are amazing in that a general problem actually *reduces to a specific version of that problem*! 

We want to prove our problem (SAT) is still hard hard even with input restrictions: in this case, clauses restricted to have $≤ 3$ literals. In our reduction, we want to **modify the given instance to get rid of everything that doesn't obey the restriction/constraint** (clauses with $\ge 4$ literals). However, we also want to keep the instance essentially the same: given the modified instance solution, we can get an original instance solution.

So now we must come up with $f$, i.e. reduce a general instance of SAT to a special version (3SAT). 

Given instance $I$ of SAT, for any clause with more than three literals, i.e. $(a_1 \or a_2 ... \or a_k)$ where $k \ge 4$, we want to create a bunch of 3-literal clauses: 

$(a_1 \or a_2 \or y_1) (-y_1\or a_3 \or y_2) (-y_2 \or a_4 \or y_3)...(-y_{k-3} \or a_{k-1} \or a_k)$

where $y_i$ is just a new literal we introduce (we make as many new clauses with new variables as we want, as long as we don't violate the number of literals allowed *in* clauses). We call this created 3SAT instance $I'$. Converting $I$ (SAT) to $I'$ (3SAT) clearly takes polynomial time. 

We need to prove $(a_1 \or a_2 ... \or a_k)$ is equivalent to the 3-clauses chain. This means proving we maintain satisfiability, i.e. any satisfying assignment of $(a_1 \or a_2 ... \or a_k)$ will allow some setting for our added  $y_i$'s such that $(a_1 \or a_2 \or y_1) (-y_1\or a_3 \or y_2) (-y_2 \or a_4 \or y_3)...(-y_{k-3} \or a_{k-1} \or a_k)$ is satisfied, and vice versa.

We start with the reverse: suppose that the chain of 3-clauses are all satisfied (all the clauses are true). This means *at least* one $a_i$ must be true- if they were all false, then an impossible chain effect follows: $y_1$ would have to be true, then $y_2$ true (since $y_1$'s negation is in the immediate next clause) and so on, forcing the last clause false. But this means $(a_1 \or a_2 ... \or a_k)$ must be true, and is thus satisfied!

Conversely, if $(a_1 \or a_2 ... \or a_k)$ is satisfied, we want to prove our chain of 3-clauses is too. We now *know* some $a_i$ must be true. So if we just set $y_1,..., y_{i−2}$ to true and the rest to false, this ensures all 3-clauses are true, since each 3-clause  that *doesn't* contain $a_i$ either contains $y_1$ OR contains a negation of the $y_i$ we set to false, making them true.

Thus, we've finished $f$, which maps any SAT to an equivalent 3SAT. 

## 3SAT -> (3,3)SAT

3,3-SAT PROBLEM is just 3SAT but the additional restriction that no literal can appear in more than 3 clauses. STILL hard!

First, creating $f$. We must somehow find a way to catch literals appearing more than 3 times. Suppose in 3SAT literal $x$ appears in $k \ge 4$ clauses. **We make a bunch of new literals $x_i$ representing occurrence $i$ of $x$**, i.e. replace $k$ instances of $x$ with a new variable. This will allow us to keep track of a literal's number of occurrences, and also satisfies the 3-clause limit!

Finally, because $x_i$'s all represent a single literal $x$, we need to ensure they all have the same value. So we add clauses:

$(-x_1 ∨ x_2) (-x_2 ∨ x_3)...(-x_k ∨ x_1)$

to ensure each $x_i$ must have the same value. Additionally, no literal $x_i$ will appear more than twice in our added clauses and thus no more than 3 times total. Hence 3SAT is satisfiable iff (3,3)-SAT instance is satisfiable, so they reduce to each other. 

## Independent Set -> Vertex Cover

Notice that set $S$ is a vertex cover of $G = (V, E)$ (S's vertices hit every edge) iff the remaining nodes, $V − S$, are an independent set of $G$. Otherwise, there will be some edge not touched by $S$'s vertices that connects 2 vertices in $V-S$. 

So for $f$, we don't have to do shit.

To solve $(G, g)$ of INDEPENDENT SET, simply look for a vertex cover of $G$ with $|V|-g$ nodes. If such a vertex cover exists, all nodes **not** in it form our independent set. If no such vertex cover exists, then G cannot possibly have an independent set of size g. 

Thus for $h$, given a vertex cover of G, we just add every node NOT in it to form a valid independent set of G.

## Independent Set -> Clique

Define the **complement** of a graph G = (V, E) to be G' = (V, E'), where E' contains every possible (unordered) edge of G NOT in E.

$f$ just transforms $G$ to $G'$. 

Then node set S is an independent set of G iff S is a clique of G'. To paraphrase, these nodes have no edges between them in G iff they have all possible edges between them in G'. Beauty. This means $h$ just takes every node in the clique of G' and returns them as the independent set of G: which leaves us with something cool: **the solution to both A and B is identical.**

## Conclusion

We've looked at some basic (with the exception of maybe 3SAT-> Independent Set) reductions this note. Next note we'll look at some more complicated ones. 

