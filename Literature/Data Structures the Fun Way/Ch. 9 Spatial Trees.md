# Introduction

Grids are already an excellent structure for when it is no longer efficient to organize data in a single dimension. Quadtrees and k-d trees are two data structure classes that allow us to adapt our structure to enable efficient operations on tighter distribution of data or scale our structure beyond two dimensions.

# Quadtrees

Grids, while they offer the convenience of storing two-dimensional data, come with a heavy overhead of either creating too many bins, which may become empty, or storing too many points in one bin. To solve this, we can establish an additional structure when needed. We can partition off another bin when one becomes too full. The structure used to achieve this is a *uniform quadtree*.

A uniform quadtree marries the branching structure of trees to grids. The root node represents the entire space covered by the tree. Each of the four branches in the tree represent an equally sized quadrant of the root space. Each branch can be commonly labelled as NorthWest, NorthEast, SouthWest, and SouthEast. Internal nodes can store up to four pointers to children, along with the associated metadata (number of points in the branch, regional spatial bounds, etc.). Leaf nodes store the list of points within the region. A single structure can be used for both internal and leaf nodes. A 2x2 grid can be used to store the pointers to child nodes for internal nodes while also used as the array of points for leaf nodes.

![[Quadtree.png]]

```cs
class QuadTreeNode
{
	public bool IsLeaf { get; set; }
	public int Count { get; set; }
	public float XMin { get; set; }
	public float XMax { get; set; }
	public float YMin { get; set; }
	public float YMax { get; set; }
	QuadTreeNode[][] Children { get; set; }
	Point[] Points { get; set; }
}

struct Point
{
	public float X;
	public float Y;
}
```

Instead of explicitly storing the bounds of each node, the bounds can be derived from the number of splits in the sequence. This brings the decision of which is valued more, saving memory or having quicker lookups.  The power of a quadtree is adaptive branching at each level, creating a hierarchical grid. The wrapper class for a quadtree contains a root node that defines the boundaries of the entire tree along with an empty list of children and points.

![[Quadtree_Partition.png]]

```cs
class Quadtree
{
	public QuadTreeNode Root { get; set; }
}
```

## Building Uniform Quadtrees

As a quadtree is built, the space is recursively subdivided with no awareness of what points might be arbitrarily close or duplicated. Additional logic must be created so that our quadtree can designate a node as a leaf node and store the points. Some of the common tests to determine if a node should be subdivided include:

- **Are there enough points to justify a split?**: If the number of points within the defined node is too small, the overhead of adding a new node to the tree would be too expensive compared to checking the list points in the current node.
- **Are the spatial bounds large enough to justify a split?**: If the points in the node meant to be divided are clustered to close together, then subdividing the node could result in multiple subdivisions without separating the points.
- **Has a maximum depth been reached?**: The quadtree can be limited to reach a certain level to preserve computation costs on excessive subdivision.

When constructing quadtrees from already existing data, it is typically best practice to iteratively add the data points to an empty quadtree.

![[Quadtree_Branching.png]]

## Adding Points

Adding points begins with calling the function from the wrapper class to ensure that we do not attempt to insert into a non-null node.

```cs
public bool QuadTreeInsert(QuadTree tree, float x, float y)
{
	if (x < tree.Root.XMin || x > tree.Root.XMax)
		return false;
	
	if (y < tree.Root.YMin || y > tree.Root.YMax)
		return false;
	
	QuadTreeNodeInsert(tree.Root, x, y);
	return true;
}
```

Once the point is determined to be within the bounds of the quadtree, the insert method is then called on the root node of the tree.

```cs
public void QuadTreeNodeInsert(QuadTreeNode node, float x, float y)
{
	node.NumPoints++;
	
	float xBinSize = (node.XMax - node.XMin) / 2.0;
	float yBinSize = (node.YMax - node.YMin) / 2.0;
	int xBin = Math.Floor((x - node.XMin) / xBinSize);
	int yBin = Math.Floor((y - node.Ymin) / yBinSize);
	
	if (!node.IsLeaf)
	{
		if (node.Children[xBin][yBin] == null)
		{
			node.Children[xBin][yBin] = new(
				node.Xmin + xBin * xBinSize,
				node.Xmin + (xBin + 1) * xBinSize,
				node.Ymin + yBin * yBinSize,
				node.Ymin + (yBin + 1) * yBinSize);
		}
		
		QuadTreeNodeInsert(node.Children[xBin][yBin], x, y);
		return;
	}
	
	node.Points.Append(new(x, y));
	
	if (node.ShouldSplit)
	{
		node.IsLeaf = false;
		
		foreach(var point in node.Points)
		{
			QuadTreeNodeInsert(node, point.X, point.Y);
		}
		
		node.NumPoints = node.NumPoints - node.Points.Length;
		node.Points = [];
	}
}
```

It is important to note that because how a quadtree divides the area, the tree structure does not change as points are inserted as the subdivided areas are already predetermined.

## Removing Points

Removing points from a quadtree is similar to insertion, but with some added complexity. The leaf node that contains the point to be deleted is determined, then the tree is proceeded back towards the root node removing splits that are no longer necessary. This can involve extracting any children's points, which may have children themselves, and merging them with the current parent.

An additional consideration is if a quadtree contains abitrarily close or duplicate points. To handle floating point number's limited precision, the [[Ch. 8 Grids#^ce0585|distance formula]] used with grids will evaluate which point is to be deleted. A helper function is also used to determine if a node is no longer needed to be divided.

```cs
public Point[] QuadTreeNodeCollapse(QuadTreeNode node)
{
	if (node.IsLeaf)
		return node.Points;
	
	for (int i = 0; i < node.Children.Length; i++)
	{
		for (int j = 0; j < node.Children[i].Length; j++)
		{
			if (node.Children[i][j] != null)
			{
				var subPoints = QuadTreeNodeCollapse(node.Children[i][j]);
				foreach (var point in subPoints)
				{
					node.Points.Append(point);
				}
				node.Children[i][j] == null;
			}
		}
	}
	
	node.IsLeaf = true;
	return node.Points;
}
```

Then the quadtree wrapper provides a top level function.

```cs
public bool QuadTreeDelete(QuadTree tree, float x, float y)
{
	if (x < tree.Root.XMin || x > tree.Root.XMax)
		return false;
		
	if (y < tree.Root.YMin || y > tree.Root.YMax)
		return false;
	
	return QuadTreeNodeDelete(tree.Root, x, y);
}

public bool QuadTreeNodeDelete(QuadTreeNode node, float x, float y)
{
	if (node.IsLeaf)
	{
		int i = 0;
		
		while (i < node.Points.Length)
		{
			if (approxEqual(node.Points[i].X, node.Points[i].Y, x, y))
			{
				// remove point from node.Points
				node.NumPoints--;
				return true;
			}
			
			i++;
		}
		
		return false;
	}
	
	float xBinSize = (node.XMax - node.XMin) / 2.0;
	float yBinSize = (node.YMax - node.YMin) / 2.0;
	int xBin = Math.Floor((x - node.Xmin) / xBinSize);
	int yBin = Math.Floor((y - node.Ymin) / yBinSize);
	
	if (node.Children[xBin][yBin] == null)
		return false;
	
	if (QuadTreeNodeDelete(node.Children[xBin][yBin], x, y))
	{
		node.NumPoints--;
		
		if (node.Children[xBin][yBin].NumPoints == 0)
			node.Children[xBin][yBin] == null;
		
		if (!node.ShouldSplit)
			node.Points = QuadTreeNodeCollapse(node);
		
		return true;
	}
	
	return false;
}
```

The current implementation only takes into consideration of points that fall into a single, specific bin. There might need to be other calculations for points that fall on the boundary of other bins.

## Searching Uniform Quadtrees

Searching a quadtree starts at the root node. Each node is determined whether it can potentially contain a node closer than the current best candidate. Each internal node has it's children nodes explored recursively, and leaf nodes have their points tested directly. If a node is determined to not be able to contain a better candidate, then the branch is then pruned. Just as with grids, we can accomplish this with the same [[Ch. 8 Grids#^3dbe93|distance]] computation.

The search is initialized with a point at Infinity so that the algorithm has something to compare to at the first step. To optimize the search, the quadrant that would potentially contain the target point needs to be determined and then recursively proceed down each level till we reach a leaf node. Once a real candidate is obtained, branches can now be pruned. If the candidate is in the SouthWest child of the NorthWest quadrant, and none of the other children can contain a point closer than the current candidate, then we can eliminate all of those children in that quadrant. The same concept applies to the other quadrants. If the NorthEast quadrant cannot contain a closer point than the current candidate, then we do not need to proceed down that branch. We only proceed down branches that meet the minimum distance is less than our current candidate.

## Nearest Neighbor Search Code

The minimum distance calcluation is the same as with grids.

```cs
public float MinDist(QuadTreeNode node, float x, float y)
{
	float xDist = 0.0;
	if (x < node.XMin)
		xDist = node.XMin - x;
	if (x > node.XMax)
		xDist = x - node.XMax;
		
	float yDist = 0.0;
	if (y < node.YMin)
		yDist = node.YMin - y;
	if (y > node.YMax)
		yDist = y - node.YMax;
		
	return Math.Sqrt(xDist ^ 2 + yDist ^ 2);
}
```

A thin wrapper is then used to establish the infinite start distance.

```cs
public Point QuadTreeNearestNeighbor(QuadTree tree, float x, float y)
{
	return QuadTreeNodeNearestNeighbor(tree.Root, x, y, Infinity);
}
```

And the actual search function.

```cs
public Point QuadTreeNodeNearestNeighbor(QuadTreeNode node, float x, float y, float bestDist)
{
	if (MinDist(node, x, y) >= bestDist)
		return null;
	
	Point bestCandidate = null;
	
	if (node.IsLeaf)
	{
		foreach(Point point in node.Points)
		{
			float dist = Distance(x, y, point.X, point.Y);
			
			if (dist < bestDist)
			{
				bestDist = dist;
				bestCandidate = point;
			}
		}
		
		return bestCandidate;
	}
	
	float xBinSize = (node.XMax - node.XMin) / 2.0;
	float yBinSize = (node.YMax - node.YMin) / 2.0;
	
	int xBin = Math.Floor((x - node.XMin) / xBinSize);
	
	if (xBin < 0)
		xBin = 0;
	
	if (xBin > 1)
		xBin = 1;
	
	int yBin = Math.Floor((y - node.YMin) / yBinSize);
	
	if (yBin < 0)
		yBin = 0;
	
	if (yBin > 1)
		yBin = 1;
		
	for (int i = xBin; i < (xBin + 1) % 2; i++)
	{
		for (int j = yBin; j < (yBin + 1) % 2; j++)
		{
			if (node.Children[i][j] != null)
			{
				Point quadBest = QuadTreeNodeNearestNeighbor(node.Children[i][j], x, y, bestDist);
				
				if (quadBest != null)
				{
					bestCandidate = quadBest;
					bestDist = Distance(x, y, quadBest.X, quadBest.Y);
				}
			}
		}
	}
	
	return bestCandidate;
}
```

# k-d Trees

Quadtrees solve the problem of dynamic splits within two dimensions, but still present inefficienies in higher dimensional data sets. Octtrees offer a three-dimensional variant, with 8 subtrees at each level. But when the splits are among *D*-dimensions, each level will contain 2$^D$ subtrees at each level. A five-dimensional octree will have 32 subtrees at each level.

## k-d Tree Structure

A *k-d tree* further integrates the branching qualities of a binary tree with the spatial partitioning of a quadtree. A k-d tree splits along a single dimension and provides each internal node with two children. The left child contains all the points that are less than or equal to the specified value of the split dimension, and the right child contains all the points greater than the value.

K-d trees do not have the same grid-like structure of a quadtree, but is empowered by the branching factor of binary search trees. This allows the k-d tree to scale to higher-dimensional data sets.

K-d trees dynamically adjust based on the data at each node. A split is not constrained to the midpoint of any dimension. Instead of partitioning space into 2$^d$ equally sized cells, a dimension and value is selected based upon suitability for the data at that node. Internal nodes then store the dimension that is being partitioned (*splitDim*) and the value that represents the partition (*splitVal*). The points are then assigned to the left and right children based on partition value.

![[K_D_Tree.png]]

Instead of splitting on alternating dimensions, the tree structure can be tailored by choosing splits from additional criteria that can improve search functions. A split can be made based on the widest dimension, or based on the distribution of points along any dimension. A k-d tree's flexible structure requires additional care when dealing with a node's spatial bounds. While each dimension may start as a perfect square, the shape and size of each node will be different. The area of a node is tracked by the definition of it's spatial bounds, which is calculated by the minimum and maximum value in the current dimension. Since there are an arbitrary count of dimensions, the bounds of each dimension is stored in two arrays, *xMin* & *xMax*. All points in the k-d tree satisfy this statement:

> *xMin*[d] <= *pt*[d] <= *xMax*[d] FOR ALL *d*

Because of the inherent complexity, k-d nodes store significant amounts of information. The high cost of storing this information is offset by flexibility and algorithmic efficiency.

```cs
class KDTreeNode
{
	public bool IsLeaf { get; set; }
	public int NumDimensions { get; set; }
	public int NumPoints { get; set; }
	public float[] XMin { get; set; }
	public float[] XMax { get; set; }
	public int SplitDim { get; set; }
	public float SplitVal { get; set; }
	public KDTreeNode Left { get; set; }
	public KDTreeNode Right { get; set; }
	public Point[] Points { get; set; }
}

class KDTree
{
	public int NumDimensions { get; set; }
	public KDTreeNode Root { get; set; }
}

class Point
{
	public float[] Values { get; set; }
}
```

In order to maintain consistency with each operation, it is helpful to store the number of dimensions in the wrapper class so that it remains fixed after creation. It is also important to remember that choosing a partitioning strategy really highlights the strengths of a k-d tree. As points cluster together, partitions can be performed in a way to provide the most information. While quadtrees will always partition to equally sized areas, k-d trees can partition to evenly divide points into separate nodes or keep a tight group of points together.

## Tighter Spatial Bounds

A spatial tree's pruning power can be increased by evaluating the bounding box of all points already contained within a node. By using tighter spatial bounds, a split at each node is not performed on the entire space represented by the node, but by the space that contains actual points.

```cs
public (float[], float[]) ComputeBoundingBox(Point[] points)
{
	int numPoints = points.Length;
	
	if (numPoints == 0)
		throw new Exception();
	
	int numDims = points[0].Length;
	
	float[] L = new float[numDims];
	float[] H = new float[numDims];
	int d = 0;
	
	while (d < numDims)
	{
		L[d] = points[0].Values[d];
		H[d] = points[0].Values[d];
		d++;
	}
	
	int i = 1;
	
	while (i < numPoints)
	{
		d = 0;
		
		while (d < numDims)
		{
			if (L[d] > points[i].Values[d])
				L[d] = points[i].Values[d];
			
			if (H[d] = points[i].Values[d])
				H[d] = points[i].Values[d];
			
			d++;
		}
		
		i++;
	}
	
	return (L, H);
}
```

The additional benefit of computing a bounding box is that it can confirm that each point contains the appropriate amount of dimensions. Tracking bounds will also add complexity with each dynamic change. Adding a point could increase the bound of any given node, and removing a point could shrink the bounds as well.

## Building k-d Trees

Construction of a k-d tree is a recursive process not dissimilar to a binary search tree. The tree begins at the first level and divides all of the data points by selecting a dimension and appropriate value. Each level repeats the process until a termination point is met. Either a minimum number of points, a minimum width, or maximum depth. The most common tests are a minimum number of points and a minimum width, but either limiting the width of a node or the depth of the tree are essential to preventing infinite recursion.

The wrapper class will provide the method for validating that every data point has the appropriate amount of dimensions.

```cs
public void BuildKDTree(KDTree tree, Point[] points)
{
	foreach (var point in points)
	{
		if (point.Values.Length != tree.NumDimensions)
			// Handle with an error
	}
	
	if (points.Length > 0)
	{
		tree.Root = new();
		RecursiveBuildKDTree(tree.Root, tree.NumDimensions, points);
	}
	else
		tree.Root = null;
}
```

The difference between building a k-d tree and a quadtree is that a dimension must be determined to split along. If included, the bounding box for each level must also be computed over *D* dimensions.

```cs
public void RecursiveBuildKDTree(KDTreeNode node, int numDims, Point[] points)
{
	node.NumPoints = points.Length;
	node.NumDimensions = numDims;
	node.Left = null;
	node.Right = null;
	node.Points = [];
	node.SplitDim = -1;
	node.SplitVal = 0.0;
	node.IsLeaf = true;
	
	(node.XMin, node.XMax) = ComputeBoundingBox(points);
	
	float maxWidth = 0.0;
	int d = 0;
	
	while (d < node.NumDimensions)
	{
		if (node.XMax[d] - node.XMin[d] > maxWidth)
			maxWidth = node.XMax[d] - node.XMin[d];
			
		d++;
	}
	
	if (!node.ShouldSplit)
	{
		foreach(var point in points)
			node.Points.Append(point);
		
		return;
	}
	
	node.SplitDim = "Chosen Split Dimension";
	node.SplitVal = "Chosen Split Value";
	node.IsLeaf = false;
	
	Point[] leftPoints = [];
	Point[] rightPoints = [];
	
	foreach (var point in points)
	{
		if (point.Values[node.SplitDim] <= node.SplitVal)
			leftPoints.Append(point);
		else
			rightPoints.Append(point);
	}
	
	node.Left = new();
	RecursiveBuildKDTree(node.Left, numDims, leftPoints);
	
	node.Right = new();
	RecursiveBuildKDTree(node.Right, numDims, rightPoints);
}
```

A common strategy for choosing the dimension to split and the value to split at is to select the middle of the widest dimension.

```cs
float maxWidth = 0.0;
int splitDim = 0;
int d = 0;

while (d < node.NumDimensions)
{
	if (node.XMax[d] - node.XMin[d] > maxWidth)
	{
		maxWidth = node.XMax[d] - node.XMin[d];
		splitDim = d;
	}
	
	d++;
}
```

Bulk construction of a k-d tree is more efficient as the structure of the tree can be adapted with a large amount of existing data. Compared to inserting an abitrary point after construction that might make subset comparisions tricky.

## k-d Tree Operations

Insertions, deletions, and searches of k-d trees are very similar to that of quadtrees. The major difference is that instead of directly comparing a fixed two dimensional partition, the tree uses the defined split at each node to explore children.

### Insertion

At each level of an insertion, the current split dimension and split value are used to determine which child to proceed down. Once a leaf node is reached, the node is evaluated to determine if a split is necessary. Also if the bounds are being tracked, then they must be recalculated at each level if necessary. With insertion, the bounding box can only grow.

```cs
int d = 0;
while (d < node.NumDimensions)
{
	if (x[d] < node.XMin[d])
		node.XMin[d] = x[d];
		
	if (x[d] > node.XMax[d])
		node.Xmax[d] = x[d];
	
	d++;
}
```

### Deletion

When deleting a point, the same steps are taken to search for the desired point as insertion. After the node is deleted, the tree returns to the root and recalculates the bounding box along the way.  Deletion can only cause the bounding box to shrink. Internal nodes are also evaluated to see if they can be collapsed.

### Search

The primary change in searching through a k-d tree from a quadtree is in the testing for which branches to prune. The Euclidean distance formula and [[Ch. 8 Grids#^efea7d|minimum distance]] calculation will continue to be applied.

```cs
public float KDTreeNodeMinDist(KDTreeNode node, Point point)
{
	float distSum = 0.0;
	int d = 0;
	
	while (d < node.NumDimensions)
	{
		float diff = 0.0;
		
		if (point.Values[d] < node.XMin[d])
			diff = node.XMin[d] - point.Values[d];
		
		if (point.Values[d] > node.XMax[d])
			diff = point.Values[d] - node.XMax[d];
			
		distSum += (diff * diff);
		d++;
	}
	
	return Math.Sqrt(distSum);
}
```

[[Ch. 8 Grids|Previous Chapter]]
[[Ch. 10 Hash Tables|Next Chapter]]