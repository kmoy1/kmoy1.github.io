# Lower Bounds

Usually, for a given problem (max flow, TSP, 3SAT, etc.) we identified some sort of computational ***resource*** like time or space (memory) and developed algorithms that *upper bounded* the amount of that resource we needed to solve the problem.

Formally, for a specific problem (say 3SAT) we can define a function 

$$t(n) = min_A max_{|x|=n} \text{(r(A,x))}$$

where $r(A,x)$ is the running time of algorithm $A$ on input $x$. 

This equation is a little tricky: let's break it down. We take the minimum over all algorithms $A$ (that solve the problem), and take the maximum over all length-$n$ inputs. So $t(n)$ is the running time of the *fastest algorithm* operating on the *slowest length-n input*. **In other words, we take the minimum of all possible algorithms' worst case runtimes.**

Thus if we produce any (correct) algorithm $A$ and bound $A$’s worst case runtime, we can provide an upper bound on $t(n)$. 

$t(n)$ (as given above) is **non-uniform**: the algorithm it represents for $n$ isn't always the same, as it needs to be the one that minimizes runtime. Actually, our definition above has issues, since it would allow "cheating" algorithms, since the space of $A$ can grow with $n$; for example, the minimizer algorithm $A_n$ for input size $n$ could simply be a lookup table for all length-n inputs. How can we remedy this? Two ways:

- TAccept the non-unformity of algorithms, but bound $A$'s **description complexity** (e.g. bounding source code length).
- Enforce uniformity of algorithms by **asking uniform questions**. In uniform computation, we enforce that $A$ *cannot* change with $n$. A uniform question could for example be the following: for some function $f(n)$, say $f(n) = n^2$ for example, is there a **correct fixed algorithm** $A$ (used for *all* size $n$ inputs) such that the worst-case running time of $A$ on size $n$ inputs is $O(f(n))$?  Or does every fixed correct $A$ have worst-case running time $ω(f(n))$ ($A$ is **lower bounded** by $f(n)$)?

But we'll focus on specific well-defined questions in this note.

Before we determine something like an optimal algorithm or a lower-bounded running time for a specific problem, we must first formally define an algorithm.

For this we need to have a **computational model** that describes how exactly computation works.

Although this topic is for another day, humanity has so far been generally pretty bad at proving lower bounds. Let's look at some computational models and lower bounds in them. For the last model, communication complexity, we will actually show how some lower bounds are proven!

## Comparison-Based Model

This is the model done by most sorting algorithms. An algorithm in this model operates on $n$ **comparable inputs** $x_1,...,x_n$, via a comparator that, for simplicity, we denote as “<”.  Let's also assume all these values are distinct. 

Now we can **model any given algorithm as a binary tree** (a “decision tree”) in which **each node is given some *pair* $(x_i, x_j)$ where $i \neq j$.** 

The algorithm’s naturally starts at the root node. It compares $x_i$ and $x_j$ and goes left if $x_i < x_j$ and right otherwise. The leaves of the tree correspond to return values, i.e. when the algorithm reaches a leaf it knows the answer and has finished. Algorithm complexity can then be determined by the maximum depth of any leaf (max height of tree), i.e. maximum number of possible comparisons.

We can prove that *any* comparison-based sorting algorithm takes $Ω(n \log n)$ time, by proving the height of the algorithm tree is $Ω(n \log n)$. We can have at most $2^d$ leaves for a height-$d$ binary tree. There are $n!$ ways to order an $n$-element array, and our algorithm has to find the correct one, i.e. **our algorithm tree needs at least one distinct leaf for each order.**

Thus we need $2^d \ge n!$, which means we need $d \ge \log (n!)$. We also know that $n! \ge (n/2)^{n/2}$, so we require that $d \ge (n/2)^{n/2} \ge \Omega(n \log n)$. Thus for a correct algorithm tree, we have $d \ge \Omega(n \log n)$, and have successfully lower-bounded algorithmic complexity!

Note that our algorithm tree implementation makes for non-uniform computation (because it grows as $n$ grows), which the lower bound *still* holds for. 



## Circuit Complexity

The **circuit model** concerns $n$ input bits ($x_1,...,x_n$) into logic gates (AND, OR, NOT) via wires. The output of these logic gates are then wired into other gates, and so on. However, **the circuit should be acyclic**: gate output should never have a path back to its own input. Eventually we all link to one final output gate, outputs our final Boolean answer.

What notions of "complexity" are there for a circuit? Two common ones:

- **Depth**: length of the longest path from any input (denoted as $x_i$) to the output gate
- **Size**: Total number of wires

The number of circuits with size s (s wires) is $2^{O(s \log s)}$, whereas the number of functions on $n$ bits is $2^{2^n}$ ($2^n$ possible arrangements of n bits, i.e. $2^n$ possible inputs, each mapped to either output 0 or 1). Thus if we want our algorithm to capture all functions with size-s circuits, there would need to be at least as many size-s circuits as functions, which means $2^{Cs \log s} ≥ 2^{2^n}$ , which after simplifying implies **there exist functions that require circuit size exponential in n**, specifically of size $s = Ω(2^n/n)$. Overall, this means that some function that takes $n$ bits will require exponential wires to represent as a circuit. 

But even though we proved it, we haven't be able to find such a function yet. 

Why do we care about circuit complexity? Let $P/\text{poly}$ denote the class of all problems with size $n$ inputs that can be solved by circuits of size $s(n) ≤ \text{poly}(n)$ (circuit size polynomially bounded by $n$). We know $P \subset P/\text{poly}$. Thus if we can show that some NP-complete problem (3SAT) is **not** in $P/\text{poly}$, then we've proven that $P \neq NP$!

## Cell Probe Model 

The **cell probe model** models data structures and particularly to prove lower bounds on things like update time, query time, or memory.

Say our structure has memory $M$ consuming $S$ machine words of space, where each word is either 32 or 64 bits. Whenever the data structure receives an operation request (update/query), it must do some computation in addition to memory read/writes to process it. **In the cell probe model, we only count memory read/writes.** 

The algorithm and memory *communicate*. Messages from algorithm to memory will either be $\text{read}(i)$, where memory responds with $M[i]$, or $\text{write}(i,\Delta)$, in which case the memory will update M[i]← ∆. So algorithm will either update or query memory at a specific location for each message. In this case, $i$ is $\lceil \log_2(S) \rceil $ bits, and ∆ is $w$ bits. The number of "messages" send back and forth to perform an operation is exactly the number of memory read/writes the DS performs.

Let us denote $t_q$ as "query time", or the number of back-and-forth communication rounds for querying. Same for $t_u$, i.e. update time. 

The cell probe model is essentially a two-player “communication game”. Player 1 is the algorithm, whose takes in an input UPDATE or QUERY. Player 2 is the memory, whose input are memory contents. The algorithm's messages are $\le \lceil \log_2(S) \rceil + w$ bits long (memory index + value). The memory’s messages $\le w$ bits (value).

So that leads us to the question: what is the minimum number of "communication rounds" required to solve a given problem? Sounds like a lower bound problem! *Communication complexity* is relevant in proving lower bounds on the number of rounds (time) needed. 

**Cell probe lower bounds** are actually the *golden standard* to proving data structure lower bounds: we only count memory reads and writes.

## Branching Programs

A **branching program** models computing some function $f : \{0, 1\}^n → Y$ (mapping a binary string to numeric output).

Our model is a DAG with a unique source node and some number of sink nodes. Each sink node represents one element of $Y$. 

