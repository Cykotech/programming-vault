# Introduction

Caches demonstrate the cost of accessing data across various media. A step further in this problem involves accessing new blocks of data. B-trees are a self-balancing tree-based structure that account for the cost of retrieving blocks of new data. B-trees store multiple pieces of data within a single node. This enables the cost of extracting all of the values to be paid at once up front, and once stored all of these values can be accessed quickly. The further tradeoff is increased complexity within each node.

When trying to index a massive set of data, that index might contain pointers to many different images and files. When the data set is larger than the index, the index might need to be stored in multiple blocks of slow storage. B-trees solve this by combining the indexing and keys while minimizing retrieval costs. B-trees define the tree structure in a way the helps prevent it from becoming horribly imbalanced.

# B-tree Structure

B-trees allow a node to have multiple branches similar to [[Ch. 6 Tries and Adapting Data Structures|tries]] and [[Ch. 9 Spatial Trees|quadtrees]]. B-tree nodes are packed with keys so that multiple partitions can be tracked and maximizing the amount of data that can be retrieved by retrieving a block of keys at one time.

The size of a B-tree node is defined with a size parameter *k*, which provides the bounds of the number elements a non-root node can contain. Each non-root node will store between *k* and *2k* sorted keys. The root node, however, can store between 0 and *2k* keys. Like [[Ch. 5 Binary Search Trees|binary search trees]], internal nodes define the ranges for its branches using the values of its keys. Nodes store pointers for each possible split point, before and after each key. This allows between *k + 1* and *2k + 1* children. This divides the values stored in child node to contain the values that come after the previous split, but before the proceding split.

```cs
class BTree
{
	public BTreeNode Root { get; set; }
	public int K { get; set; }
}

class BTreeNode
{
	public int K { get; set; }
	public int Size { get; set; }
	public bool IsLeaf { get; set; }
	public T[] Keys { get; set; }
	public BTreeNodes[] Children { get; set; }
}
```

The first complication is that a node's keys and children are stored in separate and differently sized arrays. This means how a key at index *i* maps to its adjacent child pointers needs to be defined. The pointers must be defined so that the values of the keys at or below *children[i]* are less than *keys[i]* but greater than *keys[i - 1]*.

# Searching B-trees

B-trees are searched in the same manner as all tree-based structures, start at the root node, and walk down the tree till the target is found. Each node scans the it's keys until either the target is found or the value of the keys exceeds the value of our target. Then the node either proceeds to the previous child or if the node is a leaf, the target is determined to not be in the tree.

```cs
public T? Search (BTree tree, T target)
{
	return BTreeNodeSearch(tree.Root, target);
}

public T? NodeSearch (BTreeNode node, T target)
{
	int i = 0;
	
	while (i < node.Size && target >= node.Keys[i])
	{
		if (target == node.Keys[i])
			return node.Keys[i];
		
		i++;
	}
	
	if (node.IsLeaf)
		return null;
	
	return NodeSearch(node.Children[i], target);
}
```

B-tree nodes can eventually contain large amounts of keys so scanning through a node's keys linearly, can be simplified by using something like [[Ch. 2 Binary Search|binary search]]. Instead of walking down the tree after a single comparison, multiple comparisons must be performed at each level. This tradeoff is ideal for b-trees as they are optimized to reduce the number of nodes fetched during an operation. Instead of fetching a new node for each comparison from an expensive data store, a larger node containing more data is fetched so that multiple comparisons can be performed in local memory quickly. Each comparison can still prune large sections of the tree since each node contains up to *2k* keys and *2k + 1* children.

# Adding Keys

Adding keys to a B-tree is a more complex process compared to other tree-based structures as the structure needs to remained balanced by maintaining the number of keys in a node between *k* and *2k*. The two approaches for handling full nodes are as such:

1. Split as the algorithm proceeds down the tree and never call insertion on a full node.
2. Temporarily insert into a full node and split nodes back up the tree, resulting in a 2-stage algorithm.

To perform an insertion, the algorithm must first proceed down the tree and find the location to store the new key. Then the algorithm must then walk back up the tree splitting overfull nodes. A split will always add more branches, but will not always increase the depth. Depth will only increase when the root node gets split.

## The Addition Algorithm

The first stage of the algorithm recursively decends the tree, searching for the the location to store the new key. If a matching key is found along the way, that key's data can be updated. Otherwise, the algorithm will continue down the tree to the leaf where the key will be stored.

First, a helper function is defined for inserting a key into a non-full node. This function will also take in a child pointer representing the child after the new key so that this function can be reused for splitting. This parameter will be null for a leaf node.

```cs
public void AddKey (BTreeNode node, T key, BTreeNode nextChild)
{
	int i = node.Size - 1;
	
	while (i >= 0 && key < node.Keys[i])
	{
		node.Keys[i + 1] = node.Keys[i];
		
		if (!node.IsLeaf)
			node.Children[i + 2] = node.Children[i + 1];
		
		i--;
	}
	
	node.Keys[i + 1] = key;
	
	if (!node.IsLeaf)
		node.Children[i + 2] = node.Children[i + 1];
	
	node.Size++;
}
```

While linearly shifting elements in an array is expensive, this is an acceptable tradeoff so that the number of node accesses is minimized.

Another necessary helper is to check if a node is overfull.

```cs
public bool IsOverFull(BTreeNode node) => node.Size == (2 * node.K + 1);
```

One more helper to perform the actual split is added that takes a node and the index of a child and splits the child. Everything proceeding the index remains in the original child, while precedeing the index is added to a new child. The key at the index itself is added to the parent node.

```cs
public void Split (BTreeNode node, int childIndex)
{
	BTreeNode oldChild = node.Children[childIndex];
	BTreeNode newChild = new BTreeNode(node.K);
	newChild.IsLeaf = oldChild.IsLeaf;
	
	int splitIndex = Math.Floor(oldChild.Size / 2);
	T splitKey = oldChild.Keys[splitIndex];
	
	int newIndex = 0;
	int oldIndex = splitIndex + 1;
	
	while (oldIndex < oldChild.Size)
	{
		newChild.Keys[newIndex] = oldChild.Children[oldIndex];
		oldChild.Keys[oldIndex] = null;
		
		if (!oldChild.IsLeaf)
		{
			newChild.Children[newIndex] = oldChild.Children[oldIndex];
			oldChild.Children[oldIndex] = null;
		}
		
		newIndex++;
		oldIndex++;
	}
	
	if (!old.Child.IsLeaf)
	{
		newChild.Children[newIndex] = oldChild.Children[oldChild.Size];
		oldChild.Children[oldChild.Size] = null
	}
	
	oldChild.Keys[splitIndex] = null;
	AddKey(node, splitKey, newChild);
	
	newChild.Size = oldChild.Size - splitIndex - 1;
	oldChild.Size = splitIndex;
}
```

With all the helpers defined, they can all be used within the actual insert operation.

```cs
public void Insert (BtreeNode node)
{
	int i = 0;
	
	while (i < node.Size && key >= node.Keys[i])
	{
		if (key == node.keys[i])
		{
			// Update data
			return;
		}
		
		i++;
	}
	
	if (node.IsLeaf)
		AddKey(node, key, null);
	else
	{
		Insert(node.Children[i], key);
		
		if (IsOverFull(node.Children[i]))
			Split(node, i);
	}
}
```

Extra storage gets used so that the code can be simpler. A node is allowed to temporarily overfill allowing for *2k + 1* keys and *2k + 2* children until `Split()` gets called. The buffer is created by allocating large enough arrays for the keys and children.

The only remaining special case is for when the root node needs to be split. This function will be defined in the wrapper class for the b-tree.

```cs
public void Insert (BTree tree, T key)
{
	Insert(tree.Root, key);
	
	if (IsOverFull(tree.Root))
	{
		BTreeNode newRoot = new BTreeNode(tree.K);
		newRoot.IsLeaf = false;
		newRoot.Size = 0;
	}
	
	newRoot.Children[0] = tree.Root;
	Split(newRoot, 0);
	tree.Root = newRoot;
}
```

The process of inserting a key involves a single round trip from the root node to a leaf node and back. The number of nodes accessed is proportional to the depth of the tree. The self-balancing nature of b-trees maintains all leaf nodes at the same depth with internal nodes containing a maximum branching factor of *k + 1*. This means a node retrevial scales logarithmically in *N*. The total work requires scales at $k * log_k(N)$.

## Examples of Adding Keys



# Removing Keys

## Fixing Under-full Nodes

## Finding the Minimum Value Key

## The Removal Algorithm

## Examples of Removing Keys

