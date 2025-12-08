# Introduction

[[Ch. 4 Stacks and Queues|Stacks and queues]] are a simple data structure that access data based on the order it was inserted. Priority queues take this a step further by adding an additional property of retrieval order.

# Priority Queues

Priority queues store items and enable retrieval based on a determined priority. This dynamic nature allows insertions and retrievals to be intertwined. The most basic operations a priority queues supports are:

- Add an item and assign a priority score
- Look up the item with the highest priority
- Remove the item with the highest priority

Other useful functions would include checking if the queue is empty or returning the number of items stored.

A priority queue assigns priority based on its use case. Some scenarios might be obvious while other others might need to be determined by an algoritm. Network requests can be processed by the age of the initial request or by a static property as part of the data of the request.

Priority queues can be suboptimally implemented with primitive data structures like a linked list or array. A linked list makes the highest priority easily accessible as we would just need to find the head of the list, but adding new items can be expensive as inserting a low priority item in a long queue would require us to traverse the entire list. If the list was kept unsorted we could make insertions easy, but now looking up the highest priority now becomes an expensive operation. Maintaining a priority queue built on a linked list causes a lot of careful decisions to be made based on how the data is handled. A data structure that solves this problem is the heap.

# Max Heaps

A max heap is another variation of the binary search tree that stores data based on the max heap property. The max heap property is what sorts inserted data by greater or less priority, just like a binary tree stores data that is greater than or less than. Unlike binary trees, the heap has no preference for storing data to the left or right, moving down a level in the heap only decreases the priority of the data. The heap's simplicity makes very optimal for priority queues.

While heaps are commonly visualized as a tree, they are typically implemented with arrays. In an array-based implementation, the root node will always be stored at index 1, while each node's children will be store relative to their parent's index, which will always be *2i* and *2i + 1*. The 0-th index skipped so that we can adhere to this convention. Because children are indexed on a simple mathmatical calculation, we can find a node's parent just the same by using `Math.Floor(i/2)`. Since the maximum value is always stored in a fixed location (i = 1), look up of that value will always be in constant time.

Adding and removing elements from an array presents the issue of [[Ch. 3 Dynamic Data Structures#^array-doubling|resizing an array]]. This would waste the efficiency of the heap. We can circumvent this by allocating a large array, and tracking the last inserted element in the heap as a virtual end of the array and enable expanding the heap as simple as updating a pointer. Overallocation, however, does come at the cost of wasted memory.

```cs
public class Heap<T>
{
	public T[] Data { get; set; }
	public int Size { get; set; }
	public int Last { get; set; }
}
```

Using an array as the core of a tree-like structure such as a heap is interesting. A binary tree could not be implemented efficiently with an array as it would require a largely empty array with empty or deep branches. A heap will pack the array, leaving the data constantly balanced and easily accessible as insertions and retrievals are performed.

## Adding Elements to a Heap

When adding new elements to a heap, we must maintain the max heap property. New elements must be inserted below higher priority elements while still be sorted above lower priority elements. Inserting new elements in the middle of an array is an expensive linear operation. Since the array is packed, we can insert the new element at the end of the array and bubble up the element until the new node is less than or equal to its parent, maintaining the max heap property.

```cs
public void Insert(Heap<T> heap, T value)
{
	if (heap.Last == heap.Size - 1)
		// Increase heap size
	
	heap.Last++;
	heap.Data[heap.Last] = value;
	
	int current = heap.Last;
	int parent = Math.Floor(current / 2);
	
	while (parent >= 1 && heap.Data[parent] < heap.Data[current])
	{
		T temp = heap.Data[parent];
		heap.Data[parent] = heap.Data[current];
		heap.Data[current] = temp;
		current = parent;
		parent = Math.Floor(current / 2);
	}
}
```

Heap insertions are inherently inexpensive. In the worst case, you have to swap a new entry from the last index to the root node, but this is still a small fraction of the heap's values. By design, a heap is a balanced binary tree. This means that each new entry will require at most *log$_2$(N)* swaps.

## Removing Highest-Priority Elements from Heaps

Looking up and removing the highest priority element is the core operation of a heap. To remove this node, the heap property must first be broken, then restored. The first step is swapping the root and last nodes in the tree. In an array implementation, this corresponds to swapping the first and last elements in the array. Since we are removing the highest-priority, this swap fills the gap created by the retrieval. To fix the heap property, we start at the incorrect root node and walk it down the tree. At each level, we compare this node to the current location's children and swap it with the larger of the two. The operation terminates when there are either no children remaining or niether child is larger than the current node. The worst case scenario for this operation is the same as an insertion.

```cs
public T Remove(Heap<T> heap)
{
	if (heap.Last == 0)
		return null;
	
	T result = heap.Data[1];
	heap.Data[1] = heap.Data[heap.Last];
	heap.Data[heap.Last] = null;
	heap.Last--;
	
	int i = 1;
	
	while (i <= heap.Last)
	{
		int swap = i;
		
		if (2i <= heap.Last && heap.Data[swap] < heap.Data[2i])
			swap = 2i;
			
		if (2i + 1 <= heap.Last && heap.Data[swap] < heap.Data[2i + 1])
			swap = 2i + 1;
		
		if (i != swap)
		{
			T temp = heap.Data[i];
			heap.Data[i] = heap.Data[swap];
			heap.Data[swap] = temp;
			i = swap;
		}
		else
			break;
	}
	
	return result;
}
```

## Storing Auxiliary Information

Typically heaps need to store information beyond the priority on which the heap is sorted. An additional record can be implemented to improve the use of the heap. Helper functions can then be implemented to sort the priorities.

```cs
public record TaskRecord(float Priority, string TaskName, bool Completed);

public static bool IsLessThan(TaskRecord a, TaskRecord b) => a.Priority < b.Priority;

while (parent >= 1 && IsLessThan(heap.Data[parent], heap.Data[current]))
```

# Updating Priorities

Sometimes a heap needs to support dynamic priority changes. To manage this, we would use a similar process to adding or removing nodes from the heap. We first determine if we are increasing or decreasing the priority, then bubble the node up if it is increasing, or let the node sink if the priority is decreasing.

```cs
public void UpdateValue(Heap<T> heap, int index, float value)
{
	T oldValue = heap.Data[index];
	heap.Data[index] = value;
	
	if (oldValue < value)
		// Bubble the element up using the insertion operation
	else
		// Drop the element down the heap using the removal operation
}
```

At this point, the only remaining question is how to find the node we wish to update. This can be solved by using a [[hash table]].

# Min Heaps

A min heap modifies the heap property to order elements by lowest priority. When inserting elements, we bubble up lower values and sink higher values when removing.

# Heapsort

Heaps are more powerful than just the ability to retrieve data based on priority. We can also use a heap to sort data in an algorithm called heapsort. Heapsort uses a heap to rearrange the elements in an array in the desired sorted order. This occurs in two phases:

1. Building a max/min heap from all the items
2. Extracting all the items from the heap in decreasing sorted order and storing them in a new array.

```cs
public T[] Heapsort(T[] unsorted)
{
	int length = unsorted.Length;
	Heap<T> tempHeap = new Heap<T>(Size = length);
	T[] result = new T[length];
	
	int j = 0;
	
	while (j < length)
	{
		Insert(tempHeap, unsorted[j]);
		j++;
	}
	
	j = 0;
	
	while (j < n)
	{
		result[j] = Remove(tempHeap);
		j++;
	}
	
	return result;
}
```

The runtime of this algorithm is bound by the worst case of the most expensive operation of a heap. Insertions and retrievals are of equal cost. To construct a heap of a predetermined amount of items binds this algorithm to a runtime of *Nlog$_2$(N)*.

# Conclusion

While heaps are simple variations of a binary tree, the modifications required to support heap operations sacrifice our ability to efficiently search our data. But in turn, we now have the capability to consistently walk down a single path for each operation performed on the heap. In fact, doubling the amount of nodes in a heap typically adds a single iteration to the next operation as heaps are self balancing.

[[Ch. 6 Tries and Adapting Data Structures|Previous Chapter]]
[[Ch. 8 Grids|Next Chapter]]