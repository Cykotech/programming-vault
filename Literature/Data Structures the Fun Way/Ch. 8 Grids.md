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

^ce0585

This operation can be performed with linear scan, but as with one-dimensional data, this becomes inefficient as the data set grows larger. We can adapt our data into a different structure so that we can naturally eliminate large amounts of candidates. When operating on multiple search criteria, it is important to be able to consider all the values at once, as one dimension might yield a potential candidate that wouldn't even be close in another dimension.

# Grids

Grids are a structure designed for two-dimensional data sets. Similar to how arrays have a fixed set of elements, grids have a fixed set of cells or bins. Regardless of the application of the grid, since the data is two-dimensional, we typically visualize a grid with x and y axes. Unlike arrays, each bin is not restricted to a single value. A bin is bound with a high and low value that defines the range of data points that would be stored within that bin. Grids will use each point's coordinates to determine which bin will store the data.

## Grid Structure

As with every data structure the top level of the data structure includes additional information for defining each bin and how to interact with them. The first definition would include the number of bins along the x and y axes. Then we define the bounds of each bin by using a width property. This can be precomputed to help simplify the structure.

```cs
float xBinWidth = (XEnd - XStart) / NumXBins;
float yBinWdith = (YEnd - YStart) / NumYBins;
```

There is other useful information that can be included in the grid, such as, the number of points stored or the number of empty bins.

```cs
class Grid<T>
{
	public int NumXBins { get; set; }
	public int NumYBins { get; set; }
	public float XStart { get; set; }
	public float XEnd { get; set; }
	public float XBinWidth { get; set; }
	public float YStart { get; set; }
	public float YEnd { get; set; }	
	public float YBinWidth { get; set; }
	public GridPoint[][] Bins { get; set; }
}
```

In a fixed size grid, we can map a point using a simple equation.

```cs
int xBin = Math.Floor((x - XStart) / XBinWidth);
int yBin = Math.Floor((y - YStart) / YBinWidth);
```

With previous data structures, each container only contained one value, with grids, each bin might potentially contain multiple values, so it is important that each bin contains a proper data structure to store it's own points. This is commonly done with a [[Ch. 3 Dynamic Data Structures#Linked List|linked list]]. Each bin would store the pointer to the head of a list which would contain all the points in that bin.

```cs
class GridPoint
{
	public float X { get; set; }
	public float Y { get; set; }
	public GridPoint Next { get; set; }
}
```

## Building Grids and Inserting Points

A grid begins construction by allocating the empty data structure, then iteratively inserting points using a simple for loop. The higher level structure is fixed during construction and does not change when data is added.

```cs
public bool GridInsert(Grid grid, float X, float Y)
{
	int xBin = Math.Floor((x - grid.XStart) / grid.XBinWidth);
	int yBin = Math.Floor((y - grid.YStart) / grid.YBinWidth);
	
	if (xbin < 0 || xbin >= grid.NumXBins)
		return false;
	
	if (ybin < 0 || ybin >= grid.NumYBins)
		return false;
	
	GridPoint nextPoint = grid.Bins[xBin][yBin];
	grid.Bins[xBin][yBin] = new GridPoint(x, y);
	grid.Bins[xBin][yBin].Next = nextPoint;
}
```

## Deleting Points

The approach to deleting a point is similar. The difficulty arises in determining which point in the bin to delete. There might be values that are arbitrarily close or duplicate points. Because floating point variables have limited precision, we can use a helper function to help determine approximate equality.

```cs
public bool ApproxEqual(float x1, float y1, float x2, float y2)
{
	if (Math.Abs(x1 - x2) > threshold)
		return false;
	
	if (Math.Abs(y1 - y2) > threshold)
		return false
	
	return true;
}
```

Each grid's threshold will be determined by the actual data stored and the precision of the programming language. Generally, it should be large enough to account for the accuracy of the the float.

The deletion operation will consist of finding the correct bin, then traversing the contained list till a match is found, and splicing the match out of the list. The function will return a boolean based on successful deletion.

```cs
public bool Delete(Grid grid, float x, float y)
{
	int xBin = Math.Floor((x - grid.XStart) / grid.XBinWidth);
	int yBin = Math.Floor((y - grid.YStart) / grid.YBinWidth);
	
	if (xBin < 0 || xbin > grid.NumXBins)
		return false;
	if (ybin < 0 || ybin > grid.NumYBins)
		return false;
		
	if (grid.Bins[xBin][yBin] == null)
		return false;
		
	GridPoint? current = grid.Bins[xBin][yBin];
	GridPoint? previous = null;
	
	while (current != null)
	{
		if (ApproxEqual(x, y, current.X, current.Y))
		{
			if (previous == null)
				grid.Bins[xBin][yBin] = current.Next;
			else
				previous.Next = current.Next;
			
			return true;
		}
		
		previous = current;
		current = current.Next;
	}
	
	return false;
}
```

# Searches Over Grids

Implementing a nearest-neighbor search begins by pruning the cells in the grid that would be too far to consider, minimizing the amount of computations. Then we either scan each remaining bin linearly, or use an expanding search.

## Pruning Bins

The spatial structure of a grid enables limitations to how many points get checked. Once a candidate neighbor is established, it's distance to the desired value is then used to prune bins. We can use the bounds of each bin to determine if any point within the current bin could be closer than the current best candidate. If not, the bin can be ignored. This can be accomplished with a helper function that calculates Euclidean distance.

```cs
public float Distance(float x1, float, y1, float x2, float y2)
{
	return Math.Sqrt((x1 - x2)^2 + (y1 - y2)^2);
}
```

We then use simple mathematics to evaluate bins for the closest possible point. If the closest possible point is further than the current best candidate, that bin can be automatically eliminated. If a point falls outside of the desired cell, then we can consider the Euclidean distance and consider each dimension independently. We calculate the minimum distance to shift both point's dimensions to be within the cell's range.

```
 xMin = xStart + xBin \* xBinWidth
 xMax = xStart + (xBin + 1) \* xBinWidth
 yMin = yStart + yBin \* yBinWidth
 yMax = yStart + (yBin + 1) \* yBinWidth
```

 The distance would then be calculated as follows:

> *MinDist = √(${x_{dist}}^2$ + ${y_{dist}}^2$)*

^3dbe93

The distances applied to this formula are as follows: ^efea7d

- If the point is lower than the dimension's minimum: `distance = min - point`
- If the point is within the dimension's bounds: `distance = 0`
- If the point is greater than the dimension's maximum: `distance = point - max`

If the minimum distance to any possible point within the bin is greater than the current best candidate, then the bin can be ignored.

```cs
public float MinDistToBin(Grid grid, int xBin, int yBin, float x, float y)
{
	if (xBin < 0 || x > grid.NumXBins)
		return Infinity;
	
	if (yBin < 0 || y > grid.NumYBins)
		return Infinity;
	
	float xMin = grid.XStart + xBin * grid.xBinWidth;
	float xMin = grid.XStart + (xBin + 1) * grid.xBinWidth;
	float xDist = 0;
	
	if (x < xMin)
		xDist = xMin - x;
		
	if (x > xMax)
		xDist = x - xMax;
	
	float yMin = grid.YStart + yBin * grid.yBinWidth;
	float yMin = grid.YStart + (yBin + 1) * grid.yBinWidth;
	float yDist = 0;
	
	if (y < yMin)
		yDist = yMin - y;
		
	if (y > yMax)
		yDist = y - yMax;
		
	return Math.Sqrt(xDist ^ 2 + yDist ^ 2);
}
```

## Linear Scan Over Bins

Scanning through each bin linearly is a guaranteed but inefficient method to finding the nearest neighbor.

```cs
public GridPoint GridLinearScanNN(Grid grid, float x, float y)
{
	float bestDist = Infinity;
	GridPoint? bestCandidate = null;
	
	int xBin = 0;
	
	while (xBin < grid.NumXBins)
	{
		int yBin = 0;
		
		while (yBin < grid.NumYBins)
		{
			if (MinDistToBin(grid, xBin, yBin, x, y) < bestDist)
			{
				GridPoint curr = grid.Bins[xBin][yBin];
				
				while (curr != null)
				{
					float dist = Distance(x, y, current.X, current.Y);
					
					if (dist < bestDist)
					{
						bestDist = dist;
						bestCandidate = current;
					}
					
					current = current.Next;
				}
			}
			
			yBin++;
		}
		
		xBin++;
	}
	
	return bestCandidate;
}
```

This algorithm is great in the fact that it enables the elimination of entire bins. If a bin contains a large number of points that are close, but invalid, it saves a lot of time. But if a bin contains a small amount of points that are far, we waste time by considering the bin. Unlike insertions, linear scan works well with target points that exist inside or outside the bounds of the grid as the target point does not need to be mapped on the grid.

## Ideal Expanding Search over Bins

Linear scan is the first step to optimally searching our grid for our nearest neighbor by eliminating entire bins that might not contain a valid candidate. But an unnecessary amount of computations are performed by considering bins that far from our target point, even if we've already found best candidate. Improving this search will begin by considering bins in an order based on their proximity to the target. The search starts with the bin that would contain the target. The search then expands outwards from the neighboring bins along the x and y axes only, then to each of those bins neighbors, all the way until the nearest neighbor is found.

Once a candidate is found, an evaluation of all other bins that might contain a point within the radius of that candidate must be performed. There might be a closer neighbor just within the bounds of a neighboring bin. Once all bins within this radius have been evaluated, then we have our best candidate. The tradeoff to this exact, ideal approach is complexity. This could implemented with nested `for` loops and additional logic for ensuring we don't escape the bounds of the grid and also knowing when to stop scanning.

## Simplified Expanding Search

A simplified version of this search will use an expanding diamond instead of a perfect spiral. For this implementation, we can use the Manhattan distance to count the steps between grid cells:

> *d = |xbin$_1$ - xbin$_2$| + |ybin$_1$ - ybin$_2$|*

An important consideration for this method is that it can be suboptimal for grids with nonequilateral bins.

The first iteration of this approach starts the same as the spiral, evaluate the bin in which the target would fall in. Each subsequent iteration then evaluates all the bins one step further than the last. For now, each bin we check will be scanned linearly to find the closest point in each bin. The steps for checking bins will differ as we need to follow the pattern of an expanding diamond. After each iteration, if there are no cells that can be considered valid, then it can be concluded that there are no valid points for candidacy `d` steps away from the target.

```cs
public GridPoint? GridCheckBin(Grid grid, int xBin, int ybin, float x, float y, float threshold)
{
	if (xBin < 0 || xBin >= grid.NumXBins)
		return null;
		
	if (yBin < 0 || yBin >= grid.NumYBins)
		return null;
	
	GridPoint? bestCandidate = null;
	float bestDist = threshold;
	GridPoint current = grid.Bins[xBin][yBin];
	
	while (current != null)
	{
		float dist = Distance(x, y, current.X, current.Y);
		
		if (dist < bestDist)
		{
			bestDist = dist;
			bestCandidate = current;
		}
		
		current = current.Next;
	}
	
	return bestCandidate;
}

public GridPoint? GridSearchExpanding(Grid grid, float x, float y)
{
	float bestDist = Infinity;
	GridPoint? bestPoint = null;
	
	int xb = Math.Floor((x - grid.XStart) / grid.XBinWidth);
	
	if (xb < 0)
		xb = 0;
	
	if (xb >= grid.NumXBins)
		xb = grid.NumXBins - 1;
		
	int yb = Math.Floor((y - grid.YStart) / grid.YBinWidth);
	
	if (yb < 0)
		yb = 0;
	
	if (yb >= grid.NumYBins)
		yb = grid.NumYBins - 1;
	
	int steps = 0;
	bool explore = true;
	
	while (explore)
	{
		explore = false;
		
		int xOff = steps * -1;
		
		while (xOff <= steps)
		{
			int yOff = steps - Math.Abs(xOff);
			
			if (MinDistToBin(grid, xb + xOff, yb - yOff, x, y) < bestDist)
			{
				GridPoint point = GridCheckBin(grid, xb + xOff, yb - yOff, x, y, bestDist);
				
				if (point != null)
				{
					bestDist = Distance(x, y, point.X, point.Y);
					bestPoint = point;
				}
				
				explore = true;
			}
			
			if (MinDistToBin(grid, xb + xOff, yb + yOff, x, y) < bestDist && yOff != 0)
			{
				GridPoint point = GridCheckBin(grid, xb + xOff, yb - yOff, x, y, bestDist);
				
				if (point != null)
				{
					bestDist = Distance(x, y, point.X, point.Y);
					bestPoint = point;
				}
				
				explore = true;
			}
			
			xOff++;
		}
		
		steps++;
	}
	
	return bestPoint;
}
```

# The Importance of Grid Size

A grid's bin size massively impacts the efficiency of search algorithms on our grid. Currently, when a bin gets checked, each point in a bin is scanned linearly, meaning that larger bins turns into longer runtimes per bin. However, making the bins too small, can cause issues with wasted memory of excessive empty bins. The optimal grid size can be determined by multiple factors, including number of points and distribution.

# Beyond Two Dimensions

Grids can be scaled to higher dimensions by using the same formula introducing a *z* coordinate.

> *dist = √((x$_1$ - x$_2$)$^2$ + (y$_1$ - y$_2$)$^2$ + (z$_1$ - z$_2$)$^2$)*

Or in a more general sense, we can define the distance of *d*-dimensional data, where *d* represents the dimension of the *i*th point.

> *dist(x$_1$, x$_2$) = √((∑$_d$(x$_1$\[d] - x$_2$\[d])$^2$)*

Higher dimensions introduces a new challenge to grids, the memory required to store. A 5x5 grid would require a total of 25 1x1 bins. Adding a third dimension of the same size increases the total bins to 125. As the number of bins increases exponentially, the number of empty bins increases as well, meaning the potential for wasted calculations increases.

# Beyond Spatial Data

Spatial data is easily visualized as points on a map. It's easy to consider what is closer when physical distances are the values. But when the values in our grid are attributes of a data object, we must reconsider how define "closeness". Points on our grid still have values to which we can calculate the "distance" between them, but sometimes we want to consider one attribute more important than the other. We can add another value to our Euclidean distance formula to consider the weight (*w*) of these attributes.

> *dist(x$_1$, x$_2$) = √((∑$_d$w$_d$(x$_1$\[d] - x$_2$\[d])$^2$)*

[[Ch. 7 Priority Queues and Heaps|Previous Chapter]]
[[Ch. 9 Spatial Trees|Next Chapter]]
