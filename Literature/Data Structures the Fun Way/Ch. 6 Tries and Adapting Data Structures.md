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

But unlike a binary search tree, a node does not necessarily represent a valid entry.