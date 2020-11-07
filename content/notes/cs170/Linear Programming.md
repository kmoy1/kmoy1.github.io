# Linear Programming

In LP, we are given a set of variables, and we want to assign a set of values to them such that we:

- maximize OR minimize objective function
- satisfy linear constraints (equations or inequalities)

### Example

Objective function: $\max x_1 + 6x_2$

Constraints: $x_1 \le 200$, $x_2 \le 300$, $x_1 + x_2 \le 400$, $x_1,x_2 \ge 0$

A linear equation with $x_1$ and $x_2$ is a 2D line. A linear inequality specifies a *half-space*, shaded region on one side of the line. 

Since we have 5 inequalities in the example above, we got 5 half spaces: the intersection of these 5 spaces is our set of ***feasible solutions*.** This creates a 5-vertex polygon. We usually want to find the point in this polygon that fulfills the goal of our objective function (maximize/minimize). 

Let's say our above objective function represented profit. Then possible profits lie on the line $x_1 + 6x_2 = c$. As we increase our profit (c), this “profit line” moves parallel to itself, up and to the right. We want to move up this line as far as possible while still touching the feasible region (since these points are only what's allowed). The optimum solution will be the very last feasible point the profit line touches. **This optimal point MUST be a vertex of the feasible region polygon**.

It's possible there's an optimal EDGE for our objective function. In this case, the optimum solution would not be unique (infinite solutions) but there would certainly be an optimum vertex.

For linear programs, the optimum is always achieved at a feasible region vertex. The only exceptions are LPs with no optimums. This can happen in two ways: 

- LP itself is **infeasible**: constraints are are *impossible* to satisfy. For example, $x_1 \le 12$, $x_1 \ge 13$.  
- LP feasible region is **unbounded**, arbitrarily high objective values. EX: $\max x_1 + x_2$ with $x_1,x_2 \ge 0$. 

## Solving LPs with Simplex

The **simplex algorithm** starts at some vertex (most likely (0,0)) and iteratively hops to the next vertex in the feasible region if it increases objective value. **When hitting a vertex with no better neighbors, simplex declares it to be optimal and halts**. 

Why does this imply a globally optimal vertex? Think of the profit line passing through this vertex. All of the neighbors of this vertex lie BELOW this line (or else that neighbor would be more optimal and simplex would have jumped to that). Since all the vertex’s neighbors lie below the line, every other vertex must too.

