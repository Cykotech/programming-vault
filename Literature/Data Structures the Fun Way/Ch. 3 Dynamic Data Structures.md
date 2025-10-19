# Introduction

Basic data structures give us a place to store data, but they lack the ability to properly respond to changes in that data. For example, if we change the value of an element in a sorted array, we will need to resort that array. Or if we need to change the size of the array by shrinking or growing it, static data structures are not optimal.

# The Limitations of Arrays

Arrays are great at storing multiple values, but suffer from big fallback, their size and layout are fixed in memory at the time of creation. In order to modify the array, we need to create a new array by overwriting the existing array and copying the data. 

To circumnavigate this limitation, most programming languages offer a dynamic "array" that grows and shrinks as you add and remove data. But these are typically wrappers around a static array. See [[Lists]] for an example. While these array wrappers are convenient, they can lead to a series of inefficiencies because of the changes in memory used behind the scenes.

An array can't just dynamically grow to add a single value because the slot in memory beyond the end of the array might already be occupied. So instead of risking deleting an important variable, we must reallocate a new chunk of memory for a new array of the required size.

If we know we are going to add a large amount of values, we can amortize the cost of the operation by using a strategy like array doubling. This can be expensive though because we now have to copy every element in the old array to the new array, or we might waste space by adding more elements than we actually need leading to empty space in memory.

```cs
public int[] ArrayDouble(int[] arr)
{
	int length = arr.Length;
	int[] newArr = new Array(length * 2);
	
	int i = 0;
	
	while (i < length)
	{
		newArr[i] = arr[i];
		i++;
	}
	
	return newArr;
}
```

An array's fixed nature also complicates adding new values in the middle of the array. If there is space available for the new element, we still have to shift over each existing element that is to the right of the space in which we want to add the new element. Or if the array is full, we now need to reallocate space for the new array, but need to make two copies from the old array. One for all the elements to the left of the new element, and the other for the elements to the right.

# Pointers and References

A pointer is a variable that stores the address in memory where data is stored. Pointers make sharing this data with different parts of a program easier and more efficient. Instead of passing a large piece of data around, we share a pointer and our code will know where to look when it wants to access or modify data. Pointers can also assume a null value which denotes that the pointer doesn't actually point to anything yet. Some languages offer the raw pointers so that you can access the actual address in memory. While others use variable names as an abstract reference to that address.

# Linked List

Linked Lists are very simple dynamic data structures that share a lot of characteristics with arrays. The primary difference being that instead of storing the data in memory contiguously as an array does, a linked list is composed of a chain of nodes linked by pointers. The core of a node is very simple. It contains the value and a pointer to the next node in the list.

```cs
class LinkedListNode<T>
{
	T Value { get; set; }
	LinkedListNode<T> Next { get; set; }
}
```

If you were to visualize a linked list, you could imagine a series of linked bins where each bin stores a single value with an arrow pointing to the next bin. The last node in a list will usually be `null`, noting that the node doesn't point to a valid node.

The important thing to consider about a linked list is that each node does not care about it's absolute position in the list, just it's relative position. So even if the nodes are scattered all throughout a computer's memory, it's easy to traverse the list using the pointers. But including the pointers brings one drawback when compared to arrays. The inclusion pointers requires more memory per node. An array stores elements at *K &times; N* bytes, where *K* equals the length of the array and *N* is the size of each element. A linked list however stores nodes at *K &times; (M + N)* bytes, where *M* is the size of a pointer.

But the power of linked lists lies in the fact that they do not require a contiguous block of memory. When you want to add a new node to the list, you can just grab memory from where ever it exists instead of having to reallocate a new block of memory.

Linked lists are typically stored and accessed through a single pointer to the start or head of the list. As long as that pointer is accessible, any program can traverse the list as needed.

```cs
public LinkedListNode<T> Lookup (LinkedListNode<T> head, int element)
{
	LinkedListNode<T> current = head;
	int count = 0;
	
	while (count < element && current != null)
	{
		current = current.Next;
		count++;
	}
	
	return current;
}
```

Another tradeoff to consider is the higher computing overhead when accessing values in a linked list compared to arrays. Arrays access their elements with a single computational offset. Linked lists need to iterate through every element starting at the head node till either the desired node is found or the end of the list is reached.

The tradeoffs of each data structure is very important to consider. Arrays are very valuable in scenarios like binary search with a sorted list, whereas linked lists are very inefficient with binary search.

# Operations on Linked Lists

The most important and powerful characteristic of a linked list is it's ability to dynamically rearrange the data contained within it. Each time you want to perform certain operations on an array, you either need to shift every element around or allocate a new array. A linked list simply requires reassignment of pointers.

## Inserting into a Linked List

A perfect example of this is insertion. When you insert a new element into an at capacity array, you need allocate a new block of memory. Also if the new element needs to be inserted anywhere but the end of the array, you must traverse the array and shift every element down.

A linked list however, only requires us to find the location for the new node. Adding a new node to the head of a list simply requires setting the new node's `Next` pointer to the current head of the list. Adding to the end of the list is also as simple as setting the tail node's pointer to the new node. Typically, this would require traversing the whole list to find the tail node, but there are methods to avoid this.

Adding a new node in the middle of a linked list requires only the computation of traversing the list. Once you locate where the new node is to be inserted, adding the new node only requires updating two pointers. The previous node will now point to the new node and the new node will point to the next node in the list.

```cs
// First find your desired node

public void LinkedListInsert (LinkedListNode<T> previous, LinkedListNode<T> newNode)
{
	newNode.Next = previous.Next;
	previous.Next = newNode;
}
```

One final consideration to take when inserting into a linked list is ensuring that the entire list points to the proper node when inserting before the current head of the list and ensuring that you don't insert past the end of the list.

```cs
public LinkedListNode<T> LinkedListInsert (LinkedListNode<T> head, int index, T data)
{
	// Consideration for inserting at the head of a list
	if (index == 0)
	{
		var newHead = new LinkedListNode(value);
		newHead.Next = head;
		return newHead;
	}
	
	LinkedListNode<T>? current = head;
	LinkedListNode<T>? previous = null;
	int count = 0;
	
	while (count < index && current != null)
	{
		previous = current;
		current = current.Next;
		count++;
	}
	
	// Check that you did not run off the end of the list
	if (count < index)
		// Produce error or produce logic to handle
	
	var newNode = new LinkedListNode(value);
	newNode.Next = previous.Next;
	previous.Next = newNode;
	
	return head;
}
```

It is important to return the head because the potential for changes allows us to account for insertions before the head. We could also wrap the head node in a composite linked list data structure to perform operations on that directly.

## Deleting from a Linked List

In order to delete an element in a linked list, the only required operations include deleting the desired node, and updating the previous node's pointer. In the case of an array, we would have to shift every element right of the deleted element to left on space. And just like with insertion, we must take special considerations when handling the head and end of a list. When deleting the current head of a list, we must be sure to return the new head. And when dealing with the end of list, you must consider skipping the deletion if the desired index is past the end, or returning an error.

```cs
public LinkedListNode<T>? LinkedListDelete(LinkedListNode<T>? head, int index)
{
	if (head == null)
		return null;
		
	if (index == 0)
	{
		var newHead = head.Next;
		head.Next = null;
		return newHead;
	}
	
	var current = head;
	LinkedListNode<T>? previous = null;
	int count = 0;
	
	while (count < index && current != null)
	{
		previous = current;
		current = current.Next;
		count++;
	}
	
	if (current != null)
	{
		previous.Next = current.Next;
		current.Next = null;
	}
	else
	{
		// Handle with an error
	}
	
	return head;
}
```

# Doubly Linked Lists

Linked lists form the basis of many pointer based structures. One of the most common extensions is the doubly linked list. The core functionality is the same as a standard linked list, but now every node tracks it's own previous node.

```cs
class DoublyLinkedListNode<T>
{
	T Value { get; set; }
	DoublyLinkedListNode<T> Next { get; set; }
	DoublyLinkedListNode<T> Previous { get; set; }
}
```

This new information makes operations involving previous nodes simpler. Instead of having to track the previous node ourselves or retraversing the list to find the previous node, each node now contains a pointer to the previous node. The code to perform an operation on a Doubly Linked List is similar to a standard Linked List, you just need to include updating a node's `previous` pointer as well.

# Arrays and Linked Lists of Items

While arrays and linked lists are different data structures, we can combine some of their concepts to increase the size of the structures that we are storing. For example, we want to store the list of names of people attending a party. How many characters do we need each element to store? What do we do about the waste when one person's name is significantly shorter than the limit? What happens if we want to store more data than what we initially planned for like music preferences for each guest? A great solution is to create an array that stores pointers. A pointer will always be a fixed size allowing us to store larger and more dynamic data elsewhere. It's very important to consider the trade offs of different data structures and consider how to use the strengths of one structure to improve on the weaknesses of another.

[[Ch. 2 Binary Search|Previous Chapter]]
[[Ch. 4 Stacks and Queues|Next Chapter]]