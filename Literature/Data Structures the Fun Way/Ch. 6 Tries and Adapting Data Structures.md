# Introduction

Binary search trees are an excellent structure at storing data in an organized fashion. But they are only one method of implementing a tree like structure. Instead of using a greater than or less than comparison, we can modify how the tree splits data to optimize the tree for the problem. Strings are a common data type that need to have more optimal structures. Tries are trees that split on a single character, reducing the operation cost on comparing strings in deep trees.

# Binary Search Trees of Strings

Before a solution can be optimized, you must first understand the limitations of the current approach.

## Strings in Trees

Binary search trees can store anything that is sortable. Strings can be sorted in alphabetical order. The greater than/less than comparison is reused to define which string proceeds the other in the alphabet. At first glance, a binary search tree offers an excellent comparison method to sorting string data. The worst case scenario for navigating the tree is still proportional to the depth. But there is an important factor that must also be considered, the cost of [[Ch. 1 Information in Memory#Strings|string comparison]]. The cost of the comparison at each node scales with the length of the target string and the length of the string at each node. This can get costly very quickly.

## The Cost of String Comparisons

To be able to determine the proper order in which strings should be stored, each string must be compared sequentially until a difference is found. If the only difference lies at the end of two long strings, then the entirety of both strings must be compared.

But another key piece of information will help optimize our tree. In many languages, there are only a limited number of characters that are combined in specific ways. The English alphabet contains 26 letters, if numbers and symbols are not included in the the string, then there are only a limited number of combinations that can occur.

# Tries

Tries are a tree-like data structure that branch strings based on their prefixes. On top of comparing strings at their current prefix, we also allow the tree to now split more than once. So instead of each node representing a single value, each node now represents a predefined prefix of characters. And instead of the root node containing an arbitrary value, it represents an empty prefix.

The branch of a trie can be represented with an array of pointers. There are more memory efficient representations due to some prefixes only having a limited amount of possible next characters. Just like a binary search tree, a child node only needs to be created when the data needing to be stored at that location is inserted.

But unlike a binary search tree, a node does not necessarily represent a valid entry. Leaf nodes will always represent valid entries, but internal nodes might just represent prefixes along the route to other entries. A common approach to managing this is by using a boolean marked as true if the node is an entry and false otherwise.

```cs
public class TrieNode
{
	public bool IsEntry { get; set; }
	public char[] Children { get; init; }
}
```

As with a binary search tree, a Trie can be wrapped with an interface to make accessing the trie through the root node more accessible.

```cs
public class Trie
{
	public TrieNode Root { get; init; }
}
```

The root of a trie will always be non-null. When the trie is created, the root will be initialized to be an empty string that does not represent.

## Searching Tries

Searching tries simplifies string comparison such that at each node, at each level we only need to compare the next character in the string, the sequence of the prefix has already been calculated. The complication of this approach is because this comparison changes at each level. You must be sure to properly increment the index of the string with each level. The trie wrapper allows us to abstract the initial counter and root node from the initial call.

```cs
public TrieNode TrieSearch(Trie trie, string target)
{
	return TrieNodeSearch(trie.Root, target, 0)
}

public TrieNode TrieNodeSearch(TrieNode current, string target, int index)
{
	if (index == target.Length)
		if (current.IsEntry)
			return current;
		else
			return null;
	
	char nextLetter = target[index];
	int nextIndex = current.Children.IndexOf(nextLetter);
	TrieNode? nextChild = current.Children[nextIndex];
	
	if (nextChild == null)
		return null;
	else
		return TrieNodeSearch(nextChild, target, index + 1);
}
```

Tries only contain nodes that include data, which drastically improves our search function. At each step, we are only required to perform a single look up, checking for an existing child and then either proceeding to the next node in the trie or having our search determine that the target string does not exist. A trie can contain entire dictionary worth of words, and our search would still require us to visit 6 nodes for the word `house`.

## Adding and Removing Nodes

Just like binary trees, tries adapt as data is inserted into the tree. Each character in a string progresses further down the tree to search for the desired location. But unlike binary trees, multiple internal nodes might be added to insert a single string.

The top level function will call the recursive function starting with the root node and set the current depth to 0.

```cs
public void Insert(Trie trie, string value)
{
	TrieNodeInsert(trie.Root, value, 0);
}
```

The creation of the root node does not require a special case since it is inserted at the creation of the trie.

```cs
public void TrieNodeInsert(TrieNode current, string value, int index)
{
	if (index == value.Length)
		current.IsEntry == true;
	else
	{
		char nextLetter = value[index];
		int nextIndex = current.Children.IndexOf(nextLetter);
		TrieNode? nextChild = current.Children[nextIndex];
		
		if (nextChild == null)
		{
			current.Children[nextIndex] = new TrieNode();
			TrieNodeInsert(current.Children[nextIndex], value, index + 1);
		}
		else
			TrieNodeInsert(nextChild, value, index + 1);
	}
}
```

Removing nodes follows a similar process but reversed. Instead of creating nodes from the last branch till we reach the target, we search for the target and work our way backwards till we find the first parent that contains other children. Our recursive function will return a Boolean value which will then notify the parent that the branch can be pruned.

```cs
public void TrieNodeDelete(TrieNode current, string target, int index)
{
	if (index == target.Length)
		if (current.IsEntry)
			current.IsEntry == false;
	else
	{
		char nextLetter = target[index];
		int nextIndex = current.Children.IndexOf(nextLetter);
		TrieNode? nextChild = current.Children[nextIndex];
		
		if (nextChild != null)
		{
			if (TrieNodeDelete(nextChild, target, index + 1))
				churrent.Children[nextIndex] = null;
		}
	}
	
	if (current.IsEntry)
		return false;
	
	foreach (var child in current.Children)
	{
		if (child != null)
			return false;
	}
	
	return true;
}
```

Since deletion requires making a round trip from the root to a single leaf, the cost of deletion is proportional to the length of the target string.

# Conclusion

It is important to remember that the overhead of a trie is much more costly than a binary search tree. A trie's value increases as more entries with similar prefixes are inserted into the trie. Remember that the look up cost of an entry scales independently of the number of entries and that string comparison is an expensive operation.

In a real world example, a trie could be used as part of a spell checking feature. When constantly editing an active word document, it is important to remember with each new word to check, the cost of comparing strings will add up quickly. Each edit in the document will require a look up in the trie. A trie can be further optimized in certain scenarios where there can be certain restrictions on parts of the string, such as a serial number of a commercial product.

The key takeaway of a trie is not to be the most accessible data structure, but to optimize certain operations and algorithms that can be expensive when run many times over the lifetime of an application.

[[Ch. 5 Binary Search Trees|Previous Chapter]]
[[Ch. 7 Priority Queues and Heaps|Next Chapter]]