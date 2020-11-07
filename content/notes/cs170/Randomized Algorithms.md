# Randomized Algorithms

Most algorithms are *deterministic*, meaning they don't utilize randomness as a part of its process. Most algorithms take an input and, with machine-like exactness, always produce some correct, defined output. 

However, *randomized algorithms* utilize randomness, i.e. generate random bits, as a part of execution. We can formalize a randomized algorithm as $A(x,r)$, where $x$ = usual input and $r \in \{0,1\}^R$ for some arbitrary $R$. The randomness of $R$ might cause $A(x,r)$ to run differently on the same input $x$. 

There are two classes of randomized algorithms: **Las Vegas** and **Monte Carlo**, fittingly named.

## Las Vegas

**In Las Vegas algorithms, correctness is guaranteed, but the runtime $t(x, r)$ is a random variable.** This means we'll always output the right answer but the time taken might be shitty. 

So we want to understand the runtime distribution of $A(x,r)$. Let $T(n)$ be the **expected runtime** of a Las Vegas algorithm $A(x,r)$ with inputs of size n. "Expected runtime" in this case actually refers to the worst case:

$$T(n) = \max_{|x|=n}E_r(t(x,r))$$

i.e. the maximum expected runtime of $A$ on an input $x$ of size $n$ (overall all size-n inputs).

An example of a Las Vegas algorithm is QuickSort. QuickSort always sorts the input correctly, but its runtime is a random variable. In cases like this, we take the *average* runtime (same as expected!) as $O(n \log n)$. However, note that its worst possible runtime is $O(n^2)$. 

## Monte Carlo

In Monte Carlo algorithms, we flip the script:  **correctness is now a random variable, while runtime is bounded.** This gets a little trickier because now we have to have some measure of how often our algorithm is "correct". 

Let $\Delta(x, r)$ be an “indicator random variable” for the event that $A(x, r)$ returns something *incorrect*: (∆(x, r) = 1 when $A(x, r)$ is incorrect and ∆(x, r) = 0 otherwise). Then, $E(\Delta(x, r)) = P(\Delta(x, r) = 1) = \delta > 0$ for some arbitrary $\delta$. This gives us **expected correctness probability **, or an algorithm correctness measure. 

Polling is an example of a Monte Carlo procedure. For an election with two candidates $A$ and $B$, let's say we don't poll and instead get every legal voters' vote choices and record each as a name-candidate pair into a list $x$. We want a procedure that, given $x$, estimates the *true* proportion of A's voters. We'll declare our procedure "correct" if its estimate is $\pm p$  from the true proportion ($p$ is some arbitrary error). Then we'd just tally votes in $x$ and return our results. 

That's nice, but I ain't tryna talk to every voter in the US cuh. So instead we poll: use randomness $r$ to get a random sample of population, then tally. 

Note that "runtime" (time taken to tally everyone in sample) is always bounded by the sample size, and can be made arbitrary.  However our sample estimate might not be good enough to be deemed "correct"! For example, we might get unlucky and select 1000 republicans (all voting for B) which will skewing our estimate, and thus $\Delta(x, r) > 0$.

## QuickSort

Recalling QuickSort's algorithm, which sorts an array $A[1...n]$:

```
Algorithm QuickSort(A[1 . . . n]):
	if i = 1, then return A
	else:
		1. pick a uniformly random pivot ∈ {1, . . . , n}
		2. L ← {i : A[i] < A[pivot]}
		3. R ← {i : A[i] > A[pivot]}
		4. return [QuickSort(L), A[pivot], QuickSort(R)]
```

NOTE: obtaining $L, R$ (numbers in $A$ less than, greater than pivot) can be done in $O(n)$ time via for loop.

For any fixed $A$, QuickSort(A) runtime is a random variable. Worst case is $O(n^2)$ which can occur, for example, if pivot is always chosen as $\min(A)$. However, if the pivot is chosen optimally each time (median) then recurrence relation $T(n) = 2T(n/2) + O(n)$ solves to $T(n) = O(n\log n)$ (expected runtime).

If we denote $t(A, r)$ as QuickSort(A) runtime, what is then the expected runtime $E(t(A,r))$? 

For every recursive call on some array of size $l$, we spend $\Theta(l)$ time at that level, and make $l-1 = \Theta(l)$ comparisons (l-1 numbers compared to pivot). Thus **runtime $t(A, r)$ is equal to the number of comparisons performed** (up to a constant factor). Furthermore, for any pair of indices $i \neq j \in \{1,...n\}$, $A[i]$ is compared to $A[j]$ either once or never through QuickSort. Items compared always include the pivot, and afterwards the non-pivot is put into either $L$ or $R$ and can never possibly compare with the pivot again (since we recursively call QuickSort on $L$ and $R$).

Let's define indicator $X_{i,j}$ which tells us if $A[i]$ is ever compared with $A[j]$. Since the number of comparisons bounds the runtime, we know:

$$t(A,r) \le C\sum_{i<j} X_{i,j}$$

Now we can suitably find the **expected runtime**: 

Can we bound $E(X_{i,j})$?

What is the probability that the ith and jth smallest elements ($a_i, a_j$) are ever compared? Both elements start off in the same recursive subproblem. 