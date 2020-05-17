# Network Flows

A **network** is a directed graph with edge **capacities**. For example, a network of pipelines where oil can be sent. Each pipeline (edge) has a maximum capacity it can handle, and there are no places to "cheat" and store oil. If a pipeline goes over capacity, it explodes and your entire shipping flow gets delayed and fucked up and gas prices rise by 2$ per gallon and whatever. 

**The goal: ship as much oil as possible from source $s$ to sink $t$, without making any pipelines explode.** 

## Maximizizing Flow

A network is represented by directed $G = ((V,E),s,t, c_e)$ : directed graph with a specified source and sink vertex, along with edge capacities $c_e$. Remember our goal.

A **flow** is just assigning values (flows) $f_e$ to each edge such that, for each edge $e$ of the network:

- $0 \le f_e \le c_e$ (pipeline doesn't explode)
- **Flow is conserved**: except for the source+sink node (where nothing enters source and none exits sink), the total flow entering a node $u$ is the same as the total flow exiting.

The goal: **assign edge flows** $f_e : e \in E$ that satisfies the above constraints AND maximizes a linear objective function, the total flow leaving the source node (and entering the sink node).

Hey, that sounds like LP!

Our objective function: for each vertex $i$ that is a neighbor of vertex $s$, we want $\max \sum f_{si}$: i.e. maximize the total flow leaving source node. 

Given $|E|$ edges, we have $|E|$ constraints for nonnegativity, $|E|$ constraints for max pipe capacity. For each node of the graph OTHER than $s$ and $t$, we also have one more *equality* constraint for flow conservation (flow in = flow out).

## A Closer Look

Start with zero flow from $s$. 

Repeat: choose an appropriate path from s to t, and increase flow along the edges of this path as much as possible.

If we choose a path that seems to *block* all other direct paths, simplex solves this by also **allowing paths to cancel existing flow**. If (b, a) isn’t in the original network, it effectively cancels some amount of flow previously assigned to     (a, b). 

To summarize, at each iteration simplex looks tries to find an s − t path. Edges (u, v) in each of these paths can be of two types:

- (u, v) is in the original network, and is not yet at full capacity.
- The reverse edge (v, u) is in network, where (u,v) exists and has flow.

If the current flow through an **edge** $(u,v)$ is $f$, then $(u, v)$ can take to $c_{uv} − f_{uv}$ additional units of flow without exploding. In the second case, $(u,v)$ can handle up to $f_{vu}$ more units (canceling some (or all) existing flow on (v, u)). 

A **residual network** $G^f = (V, E^f)$ captures these flow-increasing opportunities, and $E^f$ has exactly the two types of edges listed, with residual capacities $c^f$:

So **simplex actually be choosing s − t paths in the residual network**, and solves max flow this way. It proceeds in iterations, each time explicitly constructing Gf , finding a suitable s − t path in $G^f$ by using, say, a linear-time breadth-first search, and halting if there is no longer any such path along which flow can be increased.









