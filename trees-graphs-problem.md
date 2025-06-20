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
