# Introduction

Stacks and queues are data structures that change how stored data is retrieved based on the order it was inserted. Stacks fetch the most recently inserted data (LIFO) and queues fetch the data that was inserted first (FIFO). Stacks are at the core of depth-first search, where you can look further down a path until you reach a dead-end before looking down another path. Queues enable breadth-first search which is where you shallowly explore a path before going deeper.

# Stacks

A stack is a last-in, first-out (LIFO) structure which operates like a stack of papers. When you add a new paper to the stack, the only immediately viewable page is the top page. A stack typically supports two operations:

- **Push**: Add a new element to the top of the stack
- **Pop**: Remove the element from the top of the stack and return it

For example, if you push the numbers `1, 2, 3, 4, 5` onto your stack, when you pop these numbers off, you would receive them in the order `5, 4, 3, 2, 1`.

Stacks can be implemented with either arrays or linked lists.

## Stacks as Arrays

When stacks are implemented as arrays, you would define an additional variable to track the top of the stack which corresponds to the index of the last item in the array. This variable can be initiated to -1 as an indication that the stack is empty.

```cs
public class Stack<T>
{
	int ArraySize { get; set; }
	int Top { get; set; }
	Array<T> Values { get; set; }
}
```

When adding elements to a fixed size array, you must be sure to consider the position of last element in relation to the size of the array. If there is no more space, you would need to define a behavior to increase the size of the array. [[Ch. 3 Dynamic Data Structures#^array-doubling|Array doubling]] is a great example for this use case, but you must still consider the additional cost of memory usage as you insert more and more data.

```cs
public void Push(Stack<T> stack, T value)
{
	if (stack.Top == stack.ArraySize - 1)
	{
		stack.Values = ArrayDouble(stack.Values);
		stack.ArraySize = stack.Values.Length;
	}
	
	stack.Top++;
	stack.Values[stack.Top] = value;
}
```

When popping an element off the stack, you must only check that the stack is not empty. Otherwise you just decrement the top variable and return the element at the initial top index.

```cs
public T? Pop(Stack<T> stack)
{
	T? value = null;
	
	if (stack.Top > -1)
	{
		value = stack.Values[stack.Top];
		stack.Top--;
	}
	
	return value;
}
```

And remember, since elements are only being added and removed from the end of an array, there is no need to shift other elements around. As long as there is room in the array, insertions and removals can be performed at a constant cost, but there is additional cost paid when you need to resize the array, so preallocating an array that is large enough can be helpful.

## Stacks as Linked Lists

A linked list is an alternative approach to implementing a stack. A simple pointer to the head of the list, serves as the top of the stack.

```cs
public class Stack<T>
{
	LinkedListNode<T> Top { get; set; }
}
```

Instead of adding to the end of an array and resizing as needed, the linked list approach requires creating a new node and updating pointers.

```cs
public void Push(Stack<T> stack, T value)
{
	var node = new LinkedListNode<T>(value);
	node.Next = stack.Top;
	stack.Top = node;
}
```

Popping of the stack is just as simple in an array-based stack. The head node is moved to the next node in the stack, and the old head is returned. You must still check that the stack is not empty.

```cs
public T Pop(Stack<T> stack)
{
	T? value = null;
	
	if (stack.Top != null)
	{
		value = stack.Top.Value;
		stack.Top = stack.Top.Next; 
	}
	
	return value;
}
```

The new cost of using a stack backed with a linked list, is the memory cost of storing the pointers. But this now comes with the benefit of allowing your stack to grow and shrink with your data.

# Queues

A queue is a first in, first out (FIFO) data structure. A queue operates like a line for a roller coaster. The first person in line is the first person that rides the coaster. New people join the line by getting to the back. The two supported operations for a queue are:

- **Enqueue**: Add a new element to the back of the queue
- **Dequeue**: Remove the element from the front of the queue and return it

If 5 elements are enqueued in the order `1, 2, 3, 4, 5`, then when dequeued, they would be retrieved in the same order. Queues are very useful when you want to be able to process data in the order it arrives. Like stacks, queues can also be implemented as arrays or linked lists.

## Queues as Arrays

When you implement a queue as an array, the two indices you want to track are the first and last indices. When we enqueue an element, we add it to the end of the array and increment the back index. When an element is dequeued, we remove the front element and increment the front index.

Dequeueing from a fixed array runs into a big dilemma, an empty space quickly fills up at the front of the array. We can solve this with two methods. Shift each element forward every time you dequeue, or wrap the array. Shifting each element comes at a huge cost of shifting every element with every dequeue operation. When you wrap the array, you must start to consider how to handle the indicies as you increment past the end of the array. Wrapping does add complexity, but it avoids the cost of shifting elements.

## Queues as Linked Lists

Queues are better implemented as linked lists. With a linked list, we already maintain a pointer to the head of the list, which can be used as the front of our queue. We just now need to maintain a pointer to the tail of the list to use as the back of our queue.

```cs
public class Queue<T>
{
	LinkedListNode<T> Front { get; set; }
	LinkedListNode<T> Back { get; set; }
}
```

Insertions and deletions in queues are very similar to stacks as they are just updating your special pointer nodes.

```cs
public void Enqueue(Queue<T> queue, T value)
{
	var node = new LinkedListNode<T>(value);
	
	if (queue.Back == null)
		queue.Front = node;
	else
		queue.Back.Next = node;
		
	queue.Back = node;
}

public T Dequeue(Queue<T> queue)
{
	if (queue.Front == null)
		return null;
	
	var value = queue.Front.Value;
	queue.Front = queue.Front.Next;
	
	if (queue.Front == null)
		queue.Back == null;
	
	return value;
}
```

Both operations require a constant number of operations regardless of the size of the queue.

# The Importance of Order

The order in which we handle insertions and deletions can change how an algorithm behaves. When processing incoming network requests, we want to handle earlier requests, so we would use a queue. However, most programming languages use stacks to process function calls. When a new function is called, the current state is pushed onto the stack, and then jumps to a new function. When execution of a function is completed, that function is then popped off the stack.

Search algorithm behavior varies greatly on the data structure we choose as well.

## Depth-First Search

Depth-First Search is an algorithm that explores the depth of a path till the end is hit. The algorithm then returns to the last branching point, and checks other paths. Depth-First Search uses a stack by exploring the most recently inserted option. When browsing the web, you are presented with 3 different web pages. If you explore each page using Depth-First Search, you would follow every link until the displayed page has no links. You would then back track to the first page that has any unexplored links until you reach the last link on the third page.

![[Depth_First_Search.png]]

## Breadth-First Search

Breadth-First Search will explore the original 3 pages just as thoroughly as Depth-First Search, but the order in which the pages are explored will change. Instead of clicking each link as you come across it, you would save each link to a list, continue to read the web page, then proceed down the list adding new links to the bottom of the list till all 3 web pages are explored.

![[Breadth_First_Search.png]]

[[Ch. 3 Dynamic Data Structures|Previous Chapter]]
[[Ch. 5 Binary Search Trees|Next Chapter]]