# Introduction

Sometimes data needs to be sorted by multiple values. This requires the use of a multi-dimensional data structure. A grid is a common base for most spatial data structures.

# Introducing Nearest-Neighbor Search

Nearest-neighbor search is a search algorithm that retrieves the data point closest to the given search target. The definition of nearest-neighbor search is as follows:

> Given a set of N data points *X = {x$_1$, x$_2$, ... , x$_N$}*, a target value *x'*, and a distance function *dist(x, y)*, find the point *x$_i$ ∈ X* that minimizes *dist(x', x$_i$)*

Where [[Ch. 2 Binary Search#Binary Search Algorithm|binary search]] cares about finding an exact match, nearest-neighbor only wants to find the closest match. This makes a good base algorithm for many multi-dimensional data structures. 

## Nearest-Neighbor Search with Linear Scan

Starting with a modified version of [[Ch. 2 Binary Search#Linear Scan|linear scan]], we can see how this simple approach can highlight the benefits of more complex and efficient algorithms. Consider searching numbers by comparing the absolute distance: *dist(x, y) = |x - y|*. A visualization such as a number line can help by displaying the distance between two data points and generalizing the algorithm in multiple dimensions.

Scanning a line of numbers linearly, each comparison will compute the distance between the two values. When the numbers are visualized on a line, it's easy to see the numbers as sorted, but this approach does not care if the numbers are sorted as it will make every comparison regardless of order.

```cs
public float LinearClosestNeighbor(float[] arr, float target, Func<float, float, float> dist)
{
	int length = arr.Length;
	
	if (length == 0)
		return null;
	
	float candidate = arr[0];
	float closest = dist(target, candidate);
	
	int i = 1;
	
	while (i < length)
	{
		float current = dist(target, arr[i]);
		
		if (current < closest)
		{
			closest = current;
			candidate = arr[i];
		}
		
		i++;
	}
	
	return candidate;
}
```

## Searching Spatial Data

Now consider comparing distances on a map. There are multiple starting points to compare to your target. Each point contains two values.

```cs
public struct Point
{
	public float X;
	public float Y;
}
```

When comparing two points, the typical formula will be the Euclidean distance.

> *dist = √((x$_1$ - x$_2$)$^2$ + (y$_1$ - y$_2$)$^2$))*

This operation can be performed with linear scan, but as with one-dimensional data, this becomes inefficient as the data set grows larger. We can adapt our data into a different structure so that we can naturally eliminate large amounts of candidates. When operating on multiple search criteria, it is important to be able to consider all the values at once, as one dimension might yield a potential candidate that wouldn't even be close in another dimension.

# Grids

