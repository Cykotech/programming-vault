# Introduction

Binary search trees are a dynamic data structure that is built on the concept of [[Ch. 2 Binary Search|binary search]]. Sorted arrays make additions and deletions expensive. A binary search tree solves this problem by blending algorithmic efficiency and dynamic adaptability.

![[Binary_Search_Tree.png]]

# Binary Search Tree Structure

Trees are a hierarchical data structure that are composed of chains of nodes. These are a natural evolution of linked lists where each node is permitted to have two `next` nodes. The typical data stored within a tree node are the value itself, a pointer to the left and right children nodes, and sometimes a pointer to the parent node.

```cs
public class TreeNode<T>
{
	public T Value { get; set; }
	public TreeNode<T> Left { get; set; }
	public TreeNode<T> Right { get; set; }
	public TreeNode<T>? Parent { get; set; }
}
```

Sometimes additional data is needed to be stored.