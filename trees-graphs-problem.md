# Trees & Graphs - Structure, traversal and Cycle Detection

## Lowest Common Ancestor

### Problem Summary
Given a binary tree and two nodes `p` and `q`, return the **lowest node** in the tree that is an ancestor of both.
A node can be an ancestor of itself.

### Code
```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
  if (root == null || root == p || root == q) {
    return root;
  }

  TreeNode left = lowestCommonAncestor(root.left, p, q);
  TreeNode right = lowestCommonAncestor(root.right, p, q);

  if (left != null && right != null) {
    return root; // p and q found in different subtrees
  }

  return (left != null) ? left : right; // return non-null side
}
```

### Key Pattern / Idea
Use postorder traversal (left-right-root) to:
- recursively search for `p` and `q`
- **bubble up** the node that first "sees" both - thats the LCA

### Common Pitfalls
- Forgetting to check `root == p || root == q` early
- Misinterpreting when one side is null - it just means both targets are in the other subtree
- Thinking you need to track parent pointers - unnecessary with recursion

### Transferable Takeaway
This is a classic case of:
- recursing on both subtrees
- making a decision **after** recursion
- combining the results based on the presence or absence of key nodes

Same structure works for:
- Subtree searches
- Distance-between-nodes problems
- Counting or aggregating results bottom up



## Diameter of a Tree: Finding the longest path between nodes

### Problem Summary
Given a binary tree, return the length of the longest path between any two nodes. The path may or may not pass through the root, and is measured in number of edges.

### Code
```java
int diameter = 0;

public int diameterOfBinaryTree(TreeNode root) {
  dfs(root);
  return diameter;
}

private int dfs(TreeNode root) {
  if (node == null) return 0;

  int left = dfs(node.left);
  int right = dfs(node.right);

  diameter = Math.max(diameter, left + right); // update max path seen so far

  return Math.max(left, right) + 1; // return height of this subtree
}
```

### Key Pattern / Idea
Use **postorder dfs** to compute:
- `leftHeight` and `rightHeight` at each node
- the **diameter** through a node  = `left + right`
- the max of all these values given the diameter of a tree

> Height = longest path from node to leaf (in edges)

### Common Pitfalls
- confusing **height** with **diameter** (height is one side, diameter is both)
- returning `left + right` instead of updating a seperate diameter variable
- forgetting that diameter is in edges, not nodes

### Transferable Takeaway
This is a common DFS pattern:
- recurse on subtrees
- combine their results at each node
- update a global variable to track the best solution seen so far



## Serialize and Deserialize a Binary Tree: Preorder DFS

### Problem Summary
- Serialization: convert a binary tree to a string that fully preserves its structure (including nulls)
- Deserialization: convert that string back into the exact same tree structure

This is essential for:
- Saving trees to disk
- Transmitting trees between systems
- Ensuring structure isn't lost even if node values are identical

### Serialization - Preorder traversal with null markers
```java
public String serialize(TreeNode root) {
  StringBuilder sb = new StringBuilder();
  serializeHelper(root, sb);
  return sb.toString();
}

private void serializeHelper(TreeNode node, StringBuilder sb) {
  if (node == null) {
    sb.append("#,");
    return;
  }

  sb.append(node.val).append(",");
  serializeHelper(node.left, sb);
  serializeHelper(node.right, sb);
}
```
- Nulls are marked with `"#"` to preserve structure
- Preorder means **node -> left -> right**


### Deserialization - match preorder structure recursively
```java
public TreeNode deserialize(String data) {
  Queue<String> tokens = new LinkedList<>(Arrays.asList(data.split(",")));
  return deserializeHelper(tokens);
}

private TreeNode deserializeHelper(Queue<String> tokens) {
  String val = tokens.poll();

  if (val.equals("#")) {
    return null;
  }

  TreeNode node = new TreeNode(Integer.parseInt(val));
  node.left = deserializeHelper(tokens);
  node.right = deserializeHelper(tokens);

  return node;
}
```
- Use a queue to walk through tokens in order
- Rebuild the tree in preorder (just like it was serialized)

### Common Pitfalls
- Not marking nulls ("#") -> leads to ambiguity
- Forgetting preorder during reconstruction
- Using postorder/inorder without redesigning format
- Assuming all node values are unique (they don't have to be)

### Transferable Takeaway
Preorder with null markers is a common approach to:
- preserve full tree structure in a linear format
- recursively reconstruct heirarchical data

Also applies to:
- Expression trees
- Abstract syntax trees (AST)
- JSON/XML-like data parsing



## Cycle Detection in a Directed Graph using DFS

### Problem Summary
Detect whether a **directed graph** contains a **cycle**.
- Input: `n` nodes (0 to n-1), list of directed edges `[from, to]`
- Output: `true` if a cycle exists, else `false`

### Key Idea / Pattern
**Use DFS with recursion tracking**.
Track two states:
- `visited`: nodes that are fully explored
- `recStack`: nodes currently in the **DFS call stack**
If during DFS we reach a node already in `recStack`, a **back edge** (-> cycle) is found.

### Graph Representation (Adjacency List)
```java
Map<Integer, List<Integer>> graph = new HashMap<>();

for (int i = 0; i < n; i++) {
  graph.put(i, new ArrayList<>());
}

for (int[] edge : edges) {
  int from = edge[0];
  int to = edge[1];
  graph.get(from).add(to);
}
```

### Why This Works
- A node in the `recStack` means we're still exploring it
- Reaching that node again means we've looped - `cycle found`.
- Once DFS finishes for a node, we remove it from `recStack`


### Why the HashMap structure works:
```java
Map<Integer, List<Integer>> graph = new HashMap<>();
```
We use a HashMap to get **fast, direct access** to any node's neighbours.
- Lookup: `graph.get(2)` gives you all neighbours of node `(2)` in O(1).

We use an ArrayList inside to store just the list of connected nodes (neighbours).
- Example: if `graph.get(2) = [0, 3]`, then node `2` has edges `2 -> 0` and `2 -> 3`.

Think of it like:
> HashMap = index of people

> List = people they follow (edges)

### DFS Helper Method
```java
private boolean hasCycleDFS(
  int node,
  Map<Integer, List<Integer>> graph,
  boolean[] visited,
  boolean[] recStack
) {
  if (recStack[node]) {
    return true;
  }

  if (visited[node]) {
    return false;
  }

  visited[node] = true;
  recStack[node] = true;

  for (int neighbour : graph.get(node)) {
    if (hasCycleDFS(neighbour, graph, visited, recStack)) {
      return true;
    }
  }

  recStack[node] = false;
  return false;
}
```

### Main Cycle Detection Method
```java
public boolean hasCycle(int n, int[][] edges) {
  Map<Integer, List<Integer>> graph = new HashMap<>();

  for (int i = 0; i < n; i++) {
    graph.put(i, new ArrayList<>());
  }

  for (int[] edge : edges) {
    int from = edge[0], to = edge[1];
    graph.get(from).add(to);
  }

  boolean[] visited = new boolean[n];
  boolean[] recStack = new boolean[n];

  for (int i = 0; i < n; i++) {
    if (!visited[i]) {
      if (hasCycleDFS(i, graph, visited, recStack)) {
        return true;
      }
    }

  return false;
}
```

### Common Pitfalls
- Using a real stack instead of `recStack[]` - not needed, slower
- Not removing node from `recStack` after exploring all paths
- Forgetting to DFS from **all nodes** in case the graph is disconnected

### Transferable Takeaway
> When using DFS on a **directed graph**, tracking the **recursion stack** lets you track **back edges**, which signal cycles.
This pattern shows up in:
- Course scheduling problems
- Deadlock detection
- Topological sort cycle validation



## Cycle Detection in an Undirected Graph (DFS)

### Problem Summary
Given an undirected graph with `n` nodes and a list of edges, detect whether it contains a cycle.
  Input:
    - `n` (int): number of nodes labeled `0` to `n - 1`
    - edges (int[][]): list of undirected edges `[u, v]`
  Output:
    - `true` if a cycle exists
    - `false` otherwise

### Key Pattern / Idea
Use **DFS** and track the **parent** of each node to detect cycles.
For each unvisited neighbour:
  - recurse into it
  - if you reach a node that is already visited and not the parent, you've found a cycle

> This is different from directed graphs - no need for a recursion stack

### Graph Representation (Adjacency List)
```java
Map<Integer, List<Integer>> graph = new HashMap<>();

for (int i = 0; i < n; i++) {
  graph.put(i, new ArrayList<>());
}

for (int[] edge : edges) {
  int u = edge[0], v = edge[1];
  graph.get(u).add(v);
  graph.get(v).add(u); // undirected: add both directions
}
```

### DFS Cycle Detection Code
```java
private boolean hasCycleDFS(
  int node,
  int parent,
  Map<Integer, List<Integer>> graph,
  boolean[] visited
) {
  visited[node] = true;

  for (int neighbor : graph.get(node)) {
    if (!visited[neighbor]) {
      if (hasCycleDFS(neighbor, node, graph, visited)) {
        return true;
      }
    } else if (neighbor != parent) {
      return true;
    }
  }

  return false;
}
```

### Has Cycle - Main Method
```java
public boolean hasCycle(int n, int[][] edges) {
  Map<Integer, List<Integer>> graph = new HashMap<>();

  for (int i = 0; i < n; i++) {
    graph.put(i, new ArrayList<>());
  }

  for (int[] edge : edges) {
    int u = edge[0];
    int v = edge[1];
    graph.get(u).add(v);
    graph.get(v).add(u); // undirected graph
  }

  boolean[] visited = new boolean[n];

  for (int i = 0; i < n; i++) {
    if (!visited[i]) {
      if hasCycleDFS(i, -1, graph, visited) {
        return true;
      }
    }
  }

  return false;
}
```
> We pass -1 as the initial parent for the first node - it has no parent.

> This loop ensures all components are checked, not just the connected part.

### Common Pitfalls
- Failing to track the parent -> false cycle detection
- Not adding both `u -> v` and `v -> u` in undirected graph
- Assuming one connected component - must check all nodes

### Transferable Takeaway
> In **undirected graphs**, track the **parent** to avoid falsely detecting a cycle when revisiting the edge you just came from.

Pattern applies to:
- graph validation
- network loop detection
- connectivity problems


## Cycle Detection in Undirected Graph using Union-Find

### Problem Summary
Input:
- `n` (int) number of nodes, labeled `0` to `n-1`
- `edges` (int[][]): list of undirected edges, each as [u, v]

Output:
- Return `true` if adding any edge creates a cycle
- Otherwise, return `false`

### Key Pattern / Idea
Union-Find is used to track **connected components** (subgroups of connected nodes).
- Each node starts in its own group
- When we process an edge [u, v], we:
  1. use `find()` to get the **root parent** (representative) of `u` and `v`
  2. if the roots are **equal** -> they're already connected -> **adding the edge creates a cycle**
  3. if the roots are **different** -> we `union()` them -> **merge the groups**

> If two nodes are already in the same group, connecting them again means a cycle



### Initialise Parent Array
Each node is its own parent at the start:
```java
int[] parent = new int[n];

for (int i = 0; i < n; i++) {
  parent[i] = i; // node starts in its own set
}
```

### find() Method - with Path Compression
```java
private int find(int[] parent, int node) {
  if (parent[node] != node) {
    parent[node] = find(parent, parent[node]); // path compression
  }
  return parent[node];
}
```
- recursively finds the **root parent** of the node
- flattens the tree for faster future lookups

### `union()` Method - Merge Two Sets
```java
private void union(int[] parent, int u, int v) {
  int rootU = find(parent, u);
  int rootV = find(parent, v);

  if (rootU != rootV) {
    parent[rootU] = rootV; // connect the groups
  }
}
```
- Only unions if nodes are in different groups
- Keeps disjoint sets up to date

### Core Java Code
```java
public boolean hasCycle(int n, int[][] edges) {
  int[] parent = new int[n];

  for (int i = 0; i < n; i++) {
    parent[i] = i; // each node is its own parent
  }

  for (int[] edge : edges) {
    int u = edge[0];
    int v = edge[1];

    int rootU = find(parent, u);
    int rootV = find(parent, v);

    if (rootU == rootV) {
      return true; // cycle found
    }

    union(parent, u, v); // merge sets
  }

  return false; // no cycles found
}
```

### Common Pitfalls
- forgetting to initialise `parent[i] = i`
- using assignment instead of `union()` to merge sets
- not compressing paths in `find()` (hurts performance)
- re-checking edges unnecessarily - only check if roots match

### Transferable Takeaway
Union-Find is the go-to technique for:
- cycle detection in undirected graphs
- tracking disjoint sets
- Kruskal's MST algorithm
- Connectivity checks in dynamic networks
It's fast, memory-efficient, and non-recursive - perfect for large-scale graphs



## Shortest Path in an Unweighted Graph Using BFS

### Problem Summary
Input:
- `n` nodes labeled `0` to `n-1`
- a list of `edges` (as int[][]), each edge `[u, v]`
- a **start node** and a **target node**

Output:
- return the **length of the shortest path** from `start` to `target`, measured in number of edges
- return `-1` if the target is not reachable

### Why BFS?
> BFS explores nodes in **layers**. In an unweighted graph, the **first time** BFS reaches a node, it's through the **shortest path**.

- BFS guarantees shortest path in edge count
- No need for edge weights or Dijkstra
- Very efficient for sparse graphs

### Key Idea / Pattern
Use **Breadth First Search (BFS)** to explore graph in layers.
- Use a queue for level-by-level traversal
- Track visited nodes to avoid reprocessing
- Track steps (or levels) to count the shortest distance


### Graph Representation (Adjacency List)
```java
Map<Integer, List<Integer>> graph = new HashMap<>();

for (int i = 0; i < n; i++) {
  graph.put(i, new ArrayList<>());
}

for (int[] edge : edges) {
  int u = edge[0];
  int v = edge[1];

  graph.get(u).add(v);
  graph.get(v).add(u); // for undirected graphs
}
```

### BFS Code for Shortest Path
```java
public int shortestPath(int n, int[][] edges, int start, int target) {
  Map<Integer, List<Integer>> graph = new HashMap<>();

  for (int i = 0; i < n; i++) {
    graph.put(i, new ArrayList<>());
  }

  for (int[] edge : edges) {
    int u = edge[0];
    int v = edge[1];
    graph.get(u).add(v);
    graph.get(v).add(u); // undirected
  }

  Queue<Integer> queue = new LinkedList<>();
  boolean[] visited = new boolean[n];

  queue.add(start);
  visited[start] = true;
  int steps = 0;

  while (!queue.isEmpty()) {
    int size = queue.size();

    for (int i  = 0; i < size; i++) {
      int node = queue.poll();

      if (node == target) {
        return steps;
      }

      for (int neighbor : graph.get(node)) {
        if (!visited[neighbor]) {
          queue.add(neighbor);
          visited[neighbor] = true;
        }
      }
    }

    steps++;
  }

  return -1;
}
```

### Common Pitfalls
- Using DFS - it doesn't guarantee the shortest path
- Forgetting to mark nodes as visited (can cause infinite loop)
- Not counting `steps` level-by-level

### Transferable Takeaway
> Use BFS for shortest path in **unweighted** graphs. The layer where a node is first reached equals its shortest path distance.

Applies to:
- Maze solvers
- Word ladder problems
- Network hops / social distance


## Topological Sort: Linear Ordering in DAGs

### Problem Summary
Input:
  - A **Directed Acyclic Graph (DAG)** represented as:
    - number of nodes `n`
    - list of directed edges `[from, to]`

Output:
  - A list of nodes in **topological order**, where for every edge `u -> v`, node `u` appears **before** node `v`

### Why Use Topological Sort?
Topological sort gives a **valid ordering of tasks** where each task's **dependencies are resolved before it's processed**.

> Think: What can I do now that all my prerequesites are done?

### Requirements
Graph must be a DAG:
  - **Directed** -> edges have direction
  - **Acyclic** -> no cycles allowed (cycles break dependency chains)

### Graph Construction
```java
Map<Integer, List<Integer>> graph = new HashMap<>();

for (int i = 0; i < n; i++) {
  graph.put(i, new ArrayList<>());
}

for (int edge : edges) {
  int from = edge[0];
  int to = edge[1];

  graph.get(from).add(to);
}
```
> Direction matters: `from -> to` means `from` must come **before** `to`.


## Topological Sort using DFS

### Problem Summary
Input:
  - `n` nodes labeled `0` to `n - 1`
  - a list of **directed edges** `int[][] edges`, where each edge is `[u, v]`

Output:
  - A list of nodes in **topological order** - where for every edge `u -> v`, node `u` appears **before** node `v`

### Key Idea / Pattern
> Perform a **post-order DFS** - after visiting all descendants of a node **push it onto a stack**. When all DFS calls finish, **popping from the stack** gives a valid topological ordering.

  - Works only on **DAGs** (Diredted Acyclic Graphs)
  - Uses a `visited[]` array to prevent revisiting
  - Uses a `stack` to reverse the post-order


### DFS Helper
```java
private void dfs(
  int node,
  Map<Integer, List<Integer>> graph,
  boolean[] visited,
  Stack<Integer> stack) {
    visited[node] = true;

    for (int neighbor : graph.get(node)) {
      if (!visited[neighbour]) {
        dfs(neighbour, graph, visited, stack);
      }
    }

    stack.push(node); // post-order: add after visiting all children
}
```

### Main Method
```java
public List<Integer> topologicalSort(int n, int[][] edges) {
  Map<Integer, List<Integer>> graph = new HashMap<>();

  for (int i = 0; i < n; i++) {
    graph.put(i, new ArrayList<>());
  }

  for (int[] edge : edges) {
    int from = edge[0];
    int to = edge[1];
    graph.get(from).add(to);
  }

  boolean[] visited = new boolean[n];
  Stack<Integer> stack = new Stack<>();

  for (int i = 0; i < n; i++) {
    if (!visited[i]) {
      dfs(i, graph, visited, stack);
    }
  }

  List<Integer> result = new ArrayList<>();
  while (!stack.isEmpty()) {
    result.add(stack.pop());
  }

  return result;
}
```

### Common Pitfalls
- Using DFS pre-order (topological order must be **post-order**)
- Forgetting to visit disconnected components
- Forgetting to push the node after visiting all children
- Running on graphs with cycles (not allowed in topological sort)

### Transferable Takeaway
> Topological sort via **DFS post-order** is a clean way to solve ordering problems in DAGs.

Applies to:
  - Task scheduling
  - Course prerequisite planning
  - Package installation order
  - Build systems


## BFS-Based Topological Sort (Kahn's Algorithm)
```java
public List<Integer> topologicalSortKahn(int n, int[][] edges) {
  Map<Integer, List<Integer>> graph = new HashMap<>();
  int[] inDegree = new int[n];

  for (int i = 0; i < n; i++) {
    graph.put(i, new ArrayList<>());
  }

  for (int edge : edges) {
    graph.get(edge[0]).add(edge[1]);
    inDegree[edge[1]]++;
  }

  Queue<Integer> queue = new LinkedList<>();
  for (int i = 0; i < n; i++) {
    if (inDegree[i] == 0) {
      queue.add(i);
    }
  }

  List<Integer> result = new ArrayList<>();
  while (!queue.isEmpty()) {
    int node = queue.poll();
    result.add(node);

    for (int neighbor : graph.get(node)) {
      inDegree[neighbor]--;
      if (inDegree[neighbor] == 0) {
        queue.add(neighbor);
      }
    }
  }

  return result.size() == n ? result : new ArrayList<>();
}
```

### Common Pitfalls
- Forgetting `inDegree[to]++` when adding edges
- Failing to start from all `inDegree == 0` nodes
- Not checking for cycles - must verify `result.size() == n` at the end

### Transferable Takeaway
> Kahn's algorithm is a **safe, iterative way** to compute topological order in DAGs while also detecting cycles.

It's ideal for:
- Dependency resolution (packages, tasks, courses)
- Scheduling with clear "must come before" rules
- Detecting invalid dependancy graphs
