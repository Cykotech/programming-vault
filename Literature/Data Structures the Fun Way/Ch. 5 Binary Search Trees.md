# Introduction

Binary search trees are a dynamic data structure that is built on the concept of [[Ch. 2 Binary Search|binary search]]. Sorted arrays make additions and deletions expensive. A binary search tree solves this problem by blending algorithmic efficiency and dynamic adaptability.

![[Binary_Search_Tree.png]]

# Binary Search Tree Structure

Trees are a hierarchical data structure that are composed of chains of nodes. These are a natural evolution of linked lists where each node is permitted to have two `next` nodes. The typical data stored within a tree node are the value itself, a pointer to the left and right children nodes, and sometimes a pointer to the parent node.

```cs
public class TreeNode<T>
{
	public T Value { get; set; }
	public TreeNode<T>? Left { get; set; }
	public TreeNode<T>? Right { get; set; }
	public TreeNode<T>? Parent { get; set; }
}
```

Sometimes additional data is needed to be stored. You can use the tree to store the data you wish to sort by, a timestamp for example, and a pointer to the remainder of the data.

Binary search trees begin at the start of a single root node which then branches into multiple paths using the left and right pointers. This root node is used as the entry point to the data structure. Like linked lists, a binary search tree's nodes might be scattered throughout a computer's memory. Because each node is linked through pointers, the efficiency of sorting and accessibility is extremely powerful. That power is stated in the tree's primary property:

> For any node *N*, the value of any node in *N*'s left subtree is less than *N*'s value, and the value of any node in *N*'s right subtree is greater than *N*'s value.

Essentially, each node added to a tree defines the tree's structure and organization. Each node to the left of a node, will always be a lesser value than the current node, whereas the node's right nodes will always be of greater value. This definition however, restricts a binary search tree to unique values. You can design a tree to accept duplicate values.

# Searching Binary Search Trees

When searching binary trees, we make repeated steps starting at the root node. At each node, we compare the current value to the target value and move down to either the left or right node depending on if it is less or greater than respectively. This process is repeated until we either find the target node or there are no children in the correct direction. If there are no children, then the target value does not exist in the tree.

## Iterative and Recursive Searches

This search can be implemented iteratively or recursively.

A recursive approach can be implemented by using a function that either returns the current value if it is the target, or returns a call to the search function using the appropriate node to the left or right as the next step.

```cs
public TreeNode<T>? FindValue(TreeNode<T>? current, T target)
{
	if (current == null)
		return null;
		
	if (current.Value == target)
		return current;
		
	if (target < current.Value && current.Left != null)
		return FindValue(current.Left, target);
		
	if (target > current.Value && current.Right != null)
		return FindValue(current.Right, target);
		
	return null;
}
```

We return null at the end of the function in the case that a dead end is reached without finding the target to denote that the value does not exist in the tree.

This strategy allows us to *prune* half of the remaining values at each step because each subtree can only contain values that are less than or greater than the current value. For example, if we are looking for the number 63, and the current node contains 50, we can exclude the entire left subtree because all of the values contained with in will all be less than 50.

An iterative approach employs the same strategy, but instead of returning function calls, the function will now use a `while` loop to update a pointer to the current node.

```cs 
public TreeNode<T>? FindValue(TreeNode<T> root, T target)
{
	var current = root;
	
	while (current != null && current.Value != target)
	{
		if (target < current.Value)
			current = current.Left;
		else
			current = current.Right;
	}
	
	return current;
}
```

This approach will make the same steps to move further down the tree. However, when the loop is exited, current will either equal `null`, denoting that the value does not exist in the tree, or current will contain a pointer to the target value.

The computational cost for both implementations is proportional to the depth of the value in the tree. Structuring the tree to minimize depth is the key to efficiency.

## Searching Trees vs. Searching Sorted Arrays

Building and searching a binary tree can add a lot of complexity and overhead when compared to searching a sorted array. But when dealing with dynamic sets of data, a binary tree's flexibility is very poweful. When new data is constantly inserted or deleted, operations are less costly on a tree than array that needs to be reconstructed and recallocated on every operation. Being aware of your data and selecting the appropriate data structure to use for it is key for performance. A binary tree keeps your data in a constant organized and sorted state each time an operation is performed compared to an array where elements have to be sorted on each operation.

# Modifying Binary Search Trees

In a binary tree, the root node always requires special care. When performing any operation, the start is always the root node. A tree can be simplified by wrapping the root node with a simple data structure.

```cs
public class BinarySearchTree<T>
{
	TreeNode<T> Root { get; set; }
}
```

The importance of this wrapper is that we can include some top level functions to handle any cases where the root node is involved directly. For example, if the root node is `null`, then the tree is empty.

```cs
public TreeNode<T>? FindTreeNode(BinarySearchTree<T> tree, T target)
{
	if (tree.Root == null)
		return null;
		
	return FindValue(tree.Root, target);
}
```

## Adding Nodes

The operation of adding a node to a tree is very similar to searching the tree. The primary difference is handling the termination case of when the algorithm reaches the dead end. When adding a value, we want to create a new node when the dead end is reached compared to returning `null` in the search operation. As with searching, we want to include the case where the root node can be `null`.

```cs
public void InsertTreeNode(BinarySearchTree<T> tree, T value)
{
	if (tree.Root == null)
		tree.Root = new(value);
	else
		InsertNode(tree.Root, value);
}

public void InsertNode(TreeNode<T> current, T value)
{
	if (value == current.Value)
		// Update current node as needed or handle duplicate value
		return;
	
	if (value < current.Value)
	{
		if (current.Left != null)
			InsertNode(current.Left, value);
		else
		{
			current.Left = new(value);
			current.Left.Parent = current;
		}
	}
	else
	{
		if (current.Right != null)
			InsertNode(current.Right, value);
		else
		{
			current.Right = new(value);
			current.Right.Parent = current;
		}
	}
}
```

The algorithm for inserting takes the same steps to find the proper location for the new value, except we are intentionally looking for a node with no children in the correct direction. When that node is found, we set the appropriate child pointer equal to a new node containing the new value, and then set that new node's parent pointer node to the current node if necessary.

## Removing Nodes

Removing a node in a binary tree is where the complexity occurs. There are three scenarios to consider: removing a leaf node, removing an internal node with one child, and removing an internal node with two children. The complexity increases with the number of children.

Removing a leaf node is simple, we just delete the node and update the parent's pointer to that node to reflect that it no longer exists. Removing leaf nodes shows why storing a pointer to the parent is valuable. We can simply search for the node to delete, follow the pointer back to the parent, and update the parent's child pointer.

If the target node has a single child, we promote the child to be the child of the target's parent. This strategy of deletion is successful even if the child node has it's own subtree, because all of the nodes in the subtree were already sorted through the desired parent.

Complexity increases signifigantly when the target node has two children.