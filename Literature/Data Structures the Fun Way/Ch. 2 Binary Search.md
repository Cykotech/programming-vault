# Introduction

Binary search is an algorithm for efficiently searching through a sorted list by repeatedly dividing the list in half determining which half contains the target value. Binary search demonstrates how clever algorithms can use a data's structure to achieve significant computational savings.

# The Problem

Before any problem can be solved, it must first be defined. The focus here will be finding a single item in a list that matches the given target value.

> Given a set of *N* data points *X = {x$_1$, x$_2$, ..., x$_N$}* and a target value *x'*, find a point *x$_i$ ∈ X* such that *x' = x$_i$*, or indicate that no such point exists.

# Linear Scan

To better understand the value of binary search, look at a less efficient algorithm such as linear scan. Linear scan iterates through every element in an array from the beginning till either the target element is found or the end of the array is reached. This is usually implemented with a simple `while` loop.

```cs
public int LinearScan(int[] arr, int target)
{
	int i = 0;
	while(i < arr.Length)
	{
		if (arr[i] == target)
			return i;
			
		i++;
	}
	
	return -1;
}
```

Because this algorithm goes through every element, the algorithm scales directly with the number of elements in the input array. This is a brute force solution, guaranteed to find the target without consideration of performance.

# Binary Search Algorithm

Binary search is an algorithm that finds a target `v` in a sorted list, and so only works when the data is pre sorted. This algorithm operates by finding the mid point of the list and determining which half of the list contains `v`. The algorithm then repeats this operation on the half that might contain `v` until only one value remains.

The key to efficient algorithms is taking advantage of the data's structure or information. In the case of binary search, we are using the fact that the data is sorted in a predetermined order. Take an array of integers as the example.

```cs
int[] nums = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
```

If we are looking for the number 7, when we look at the element at the midpoint, 5, since we know that 7 is greater than 5, we don't have to search the lower half of the array. We can only do this because we know that the numbers are already sorted in increasing order.

Binary search tracks the current search area using two indices, `lowIndex` and `highIndex`. These two points are how we determine where the midpoint is. At the start of each iteration, the algorithm chooses the midpoint as such:

```cs
int midIndex = Math.Floor((highIndex + lowIndex) / 2);
```

We then compare `v` to `arr[midIndex]`. Of course if these two values are equal, we can return this value and terminate the algorithm. If `v` is less than our midpoint, we would then make our `highIndex` equal to `midIndex - 1`, and if `v` is greater, `lowIndex` would then be set equal to `midIndex + 1`. Note that even if the `lowIndex` or `highIndex` may point to the target, we cannot reliably call that comparison as this algorithm only cares about the value pointed by the `midIndex`.

## Absent Values

One other consideration that must be taken into account is if the target value does not exist in the array. In linear scan, we can confirm the values absence by reaching the end of the list. We can confirm this in binary search by testing our bounds instead. Since with each step, we are moving one of the bounds past the midpoint, we can continue to iterate through the array until `highIndex < lowIndex` as this will confirm we've tested the entire array.

## Implementing Binary Search

Binary search can be implemented in a single `while` loop. And just like linear search, we will also return `-1`, in the case of an absent value.

```cs
public int BinarySearch(int[] arr, int target)
{
	int highIndex = arr.Length - 1;
	int lowIndex = 0;
	
	while (lowIndex <= highIndex)
	{
		int midIndex = Math.Floor((highIndex + lowIndex) / 2);
		
		if (arr[midIndex] == target)
			return midIndex;
		
		if (arr[midIndex] < target)
			lowIndex = midIndex + 1;
		
		else
			highIndex = midIndex - 1;
	}
	
	return -1;
}
```

# Runtime

We can tell almost immediately that binary search is faster than linear scan. The relative speed of an algorithm is determined by the data it is run on. There are definitely conditions that linear scan can and will beat binary search, i.e. the target element is at the beginning of the list. And then there are also times where binary search is unnecessary. For a list of two elements, we don't need to find a midpoint, we can just look directly at the two elements.

An algorithm is analyzed based on it's average and worst-case performance. This is done through [[Big-O notation]]. Linear scan's worst case is if the target is at the end of the list or the target is absent. The number of iterations in this case will always be `N` where `N` is the number of elements in the array. Where as binary search removes half the data with each step. This makes binary search scale logarithmically with the size of the data. Granted there is more computing work being done in calculating new midpoints and setting the bounds of each search area, but with larger sets of data, checking less elements far outweighs this cost.

[[Ch. 1 Information in Memory|Previous Chapter]]
[[Ch. 3 Dynamic Data Structures|Next Chapter]]