# TopCoder SRM 561 Orienteering Solution

**Author:** Zhong Zhixian
**Keywords:** Graph Theory, Math, Probability & Expectation

---

## Problem Summary

Given an unrooted tree represented as a grid graph, and a subset of its nodes \$C\$, we randomly choose a subset \$S\$ of size \$K\$ from \$C\$ with equal probability. Let \$\mathrm{length}\$ be the shortest path length (counted as the number of edges traversed, counting multiplicities) that visits all nodes in \$S\$. Given the tree, the set \$C\$, and an integer \$K\$, compute the expected value of \$\mathrm{length}\$.

Constraints:

* Number of nodes in the tree \$\leq 50 \times 50\$
* \$|C| \leq 300\$
* \$2 \leq K \leq |C|\$

## Solution

### Understanding \$\mathrm{length}\$

Since the tree is a grid tree, we can use tree properties:

* There is a unique simple path between any two nodes.
* Every edge is a bridge (cut-edge).

### Key Properties

**Property 1:** For any \$S \subseteq C\$ and \$a, b \in S\$, a path from \$a\$ to \$b\$ visiting all nodes in \$S\$ exists.

**Property 2:** Let \$P\$ be the shortest path visiting all nodes in \$S\$. If we order \$S = {v\_0, v\_1, ..., v\_{|S|-1}}\$ by the first time each node is visited in \$P\$, then \$P\$ starts at \$v\_0\$ and ends at \$v\_{|S|-1}\$.

**Property 3:** For all \$i \in \[1, |S|)\$, the path from \$v\_{i-1}\$ to \$v\_i\$ is a simple path in \$P\$.

**Property 4:** An edge \$e \in P\$ if and only if removing \$e\$ disconnects some pair \$u,v \in S\$.

**Property 5:** Any edge appears in \$P\$ at most 2 times.

**Property 6:** An edge appears exactly once in \$P\$ if and only if it is in the simple path from \$v\_0\$ to \$v\_{|S|-1}\$.

Let \$E\_S\$ be the set of edges that appear in simple paths between all pairs in \$S\$. Let \$\mathrm{path}(u,v)\$ be the simple path between \$u\$ and \$v\$.

Let \$P\$ be the shortest path visiting all nodes in \$S\$, starting at \$u\$ and ending at \$v\$:

$$
\mathrm{length}(P) = 2|E_S| - |\mathrm{path}(u,v)|
$$

To minimize \$\mathrm{length}(P)\$, maximize \$|\mathrm{path}(u,v)|\$.
Let \$D\_S\$ be the max pairwise distance in \$S\$, then:

$$
\mathrm{length} = 2|E_S| - D_S
$$

### Expected Value

We want:

$$
E(\mathrm{length}) = 2E(|E_S|) - E(D_S)
$$

#### Expected |E\_S|

Let \$E\$ be all edges in the tree.

$$
E(|E_S|) = \sum_{e \in E} P(e \in E_S)
$$

Let \$e\$ disconnect tree into \$A\$ and \$B\$ when removed, with \$s\$ points from \$C\$ in \$A\$.

$$
P(e \notin E_S) = \frac{\binom{s}{K}}{\binom{|C|}{K}} + \frac{\binom{|C|-s}{K}}{\binom{|C|}{K}}
$$

$$
P(e \in E_S) = 1 - \frac{\binom{s}{K}}{\binom{|C|}{K}} - \frac{\binom{|C|-s}{K}}{\binom{|C|}{K}}
$$

Define function:

```python
# Returns f(a, b, K) = C(a, K) / C(a + b, K)
def f(a, b, K):
    x = 1.0
    for i in range(K):
        x *= (a - i) / (a + b - i)
    return x
```

So:

$$
E(|E_S|) = \sum_{e \in E} \left(1 - f(s_e, |C| - s_e, K) - f(|C| - s_e, s_e, K) \right)
$$

To compute \$s\_e\$, convert the unrooted tree to rooted, and compute subtree counts:

```python
def dfs(v, parent, C, tree, s):
    s[v] = int(v in C)
    for child in tree[v]:
        if child != parent:
            dfs(child, v, C, tree, s)
            s[v] += s[child]
```

### Expected D\_S

There are \$\le 300^2\$ pairs \$u,v\$. For each, calculate probability that \$u,v\$ is the furthest pair in a random \$K\$-subset of \$C\$.
To break ties uniquely, define:

$$
d(u,v) = |\mathrm{path}(u,v)| + 2^{-h(u)} + 2^{-h(v)}
$$

Where \$h(u)\$ is a unique ID for node \$u\$. This ensures distinctiveness of \$d(u,v)\$.

Final result:

$$
E(\mathrm{length}) = 2E(|E_S|) - E(D_S)
$$
