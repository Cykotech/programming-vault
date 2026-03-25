#dsa
# Introduction

A hash table is a dynamic data structure hyper optimized for insertions and lookups. Hash tables use mathematical functions to point to the location of the data. These are best used when the goal is to store data for quick searches and retrievals.

Arrays provide a compact structure for storing individual pieces of data and mechanism for efficient retrieval, but only when we know the item's index. [[Ch. 2 Binary Search|Binary search]] demonstrates how complex finding an data can be when don't know the index. While the search itself is efficient, this requires the array to be in a constantly sorted state with each insertion and deletion.

A hash table enables our data to be retrieved with only knowing one specific value of the data which is used as a key.

# Storage and Search with Keys

An ideal example would include storing integer values in an array at the same index as the number. i.e. The number 9 would be stored at index 9. This makes finding this number extremely efficient as accessing an array's index directly can be done in constant time.

The trade off for this approach is that now there needs to be a space for every possible number. Credit card numbers contain 16 digits. Storing every possible 16 digit number would require an array of 10$^6$. This will lead to excess unused space.

[[Ch. 3 Dynamic Data Structures#Arrays and Linked Lists of Items|Linked lists]] are an example of how our data structures don't need to store the data directly but instead a pointer to our data. Entire composite data structures can be very difficult to use as the stored value in a hash table. The solution is to derive a single value from the data to use as a key. The key can range from various identifiers such as names or ids.

Using these unique identifiers now presents an indexing problem. Arrays use integer values as their indexes so finding data stored based on a string or a floating point value becomes impossible. Hash tables use mathematical computations to convert these identifiers into integral values that can be used to index our data.
# Hash Tables

The mathematical mappings used to compress the large key identifiers are referred to as hash functions. Hash functions convert key *k* to a value that can be indexed into bin *b*. A simple example is to perform a modulo operation on the hash value using the number of bins.

> *f(k) = k % b*

Storing credit card numbers becomes simpler using this method when attempting to index using the last two digits of each number. But presents the new problem of having multiple values stored in one location. If multiple credit card numbers end with the value of 10, and we 100 bins, all of these numbers will be stored in the same bin. A hash table's structure includes the solution of how to handle any such collisions. The hash functions sole job is to take the identifier and convert it into an indexable value.
## Collisions

The most optimal hash functions can't avoid all collisions. This is a natural problem presented when trying to convert a large number of keys to a smaller set size of bins. The most common approaches are chaining and linear probing.

## Chaining

Chaining is a collision strategy for employing further structure within each bin. Instead of a bin storing a single piece of data, it can point to the head of a linked list.

```cs
class HashTable
{
	public int Size { get; set; }
	public ListNode[] Bins { get; set; }
}

class ListNode<T>
{
	public T Key { get; set; }
	public T Value { get; set; }
	public ListNode<T> Next { get; set; }
}
```

Each bin now contains every piece of data that has been indexed to that bin. This also makes insertions simple.

```cs
public void Insert(HashTable hash, T key, T value)
{
	int hashValue = HashFunction(key, hash.Size);
	
	if (hash.Bins[hashValue] == null)
	{
		hash.Bins[hashValue] = new ListNode(key, value);
		return;
	}
	
	ListNode current = hash.Bins[hashValue];
	
	while (current.Key != key && current.Next != null)
	{
		current = current.Next;
	}
	
	if (current.Key == key)
		current.Value = value;
	else
		current.Next = new ListNode(key, value);
}
```

Searches are equally simple as well.

```cs
public ListNode? Lookup (HashTable hash, T key)
{
	int hashValue = HashFunction(key, hash.Size);
	
	if (hash.Bins[hashValue] == null)
		return null;
	
	ListNode current = hash.Bins[hashValue];
	
	while (current.Key != key && current.Next != null)
	{
		current = current.Next;
	}
	
	if (current.Key == key)
		return current;
	
	return null;
}
```

Deletions are simple as well, as after the lookup, removing the item follows the removal process of the storage data structure. In this case, splicing the node out of the linked list.

```cs
public ListNode? Delete(HashTable hash, T key)
{
	int hashValue = HashFunction(key, hash.Size);
	
	if (hash.Bins[hashValue] == null)
		return null;
	
	ListNode? current = hash.Bins[hashValue];
	ListNode? prev = null;
	
	while (current.Key != key && current.Next != null)
	{
		prev = current;
		current = current.Next;
	}
	
	if (current.Key == key)
	{
		if (prev != null)
		{
			prev.Next = current.Next;
		}
		else
		{
			hash.Bins[hashValue] = current.Next;
		}
		
		return current;
	}
	
	return null;
}
```

With this approach, while looking up data results in a runtime equivalent of the data structure used to store the data after the indexing, each operation is now performed on a smaller subset of the data.

## Linear Probing

A different strategy for handling collisions is to utilize adjacent bins. If the bin initially selected to store data is full, the hash table can look in the next bin. The process repeats until an empty bin is found. When applied to other operations of the hash table, searches start at the corresponding bin and proceed till either the target is found or determined to not be in the hash table.

Hash tables that implement linear probing require a different structure.

```cs
class HashTableEntry
{
	public T Key { get; set; }
	public T Value { get; set; }
}

class HashTable
{
	public int Size { get; set; }
	public int NumKeys { get; set; }
	public HashTableEntry[] Bins { get; set; }
}
```

It is important to track this information as when the number of keys is equal to the size of the hash table, the hash table must consider resizing, but must do so carefully as the hash function has mapped all of the existing keys based on the original size of the table.

```cs
public bool Insert(HashTable hash, T key, T value)
{
	int index = HashFunction(key, hash.Size);
	int count = 0;
	
	HashTableEntry current = hash.Bins[index];
	
	while (current != null && current.Key != key && count != hash.Size)
	{
		index++;
		
		if (index >= hash.Size)
			index = 0;
		
		current = hash.Bins[index];
		count++
	}
	
	if (count == hash.Size)
		return false;
		
	if (current == null)
	{
		hash.Bins[index] == new HashTableEntry(key, value);
		hash.NumKeys++;
	}
	else
	{
		hash.Bins[index].Value = value;
	}
	
	return true;
}
```

Searching follows the same pattern.

```cs
public T? Lookup(HashTable hash, T key)
{
	int index = HashFunction(key, hash.Size);
	int count = 0;
	
	HashTableEntry? current = hash.Bins[index];
	
	while (current != null && current.Key != key && count != hash.Size)
	{
		index++;
		
		if (index >= hash.Size)
			index = 0;
		
		current = hash.Bins[index];
		count++;
	}
	
	if (current != null && current.Key == key)
		return current.Value;
	
	return null;
}
```

With most linear probing implementations, deletions are complex as when you delete an entry, it becomes difficult to find entries indexed based on that first entry's location. It then becomes a decision to either reindex the hash table with each deletion or insert dummy data.

The advantage of linear probing is that the array stores all of the hash table's data directly instead of adding the overhead of a linked list or other data structure. But this comes with a drawback of potentially iterating through the entire hash table as it fills up.

# Hash Functions

A good hash function draws the line between hash tables and linked lists. Hash functions should uniformly map keys throughout bins instead of overloading a select few. Hash functions should meet these key criteria:

- **Be deterministic**: A hash function needs to map the same key to the same bin every time. If the same input does not produce the same index every time, then when data is inserted into the hash table, it cannot be looked up.
- **Have a predefined range for a given size**: The hash function needs to produce an output for a limited range corresponding to the number of buckets. For a table of *b* buckets, the hash function should map within the range of *[0, b - 1]*. The hash function should also vary the range with the size of the hash table. If a larger table is needed, the function should correspondingly adjust the range.

## Handling Non-Numeric Keys

Numeric keys are simple to implement hash functions for, they can use simple mathematical functions. But hash functions become complex when using keys such as strings. Typically, non-numeric keys will be transformed into a numeric value, then can be mathematically indexed. Strings for example will be first converted to the ASCII value which provides the numeric value that can be indexed. A common strategy is to use *Horner's method* to convert a string key to a numerical index.

```cs
public int StringHash(string key, int size)
{
	int total = 0;
	
	foreach (char c in key)
	{
		total = CONST * total + (int)c;
	}
	
	return total % size;
}
```

`CONST` is typically a prime number larger than any of the possible input characters in a string.

## An Example Use Case

Hash tables are particularly useful in tracking sets of items when uniqueness or counts are required. Most programming languages have their own implementation built in.

Stacks and queues can be replaced with a hash table when implementing [[Ch. 4 Stacks and Queues#The Importance of Order|breadth-first search and depth-first search]]. The hash table can be used to track nodes that have already been visited. This is an excellent use case for a hash table as they are optimized for fast insertions and lookups.

[[Ch. 9 Spatial Trees|Previous Chapter]]
[[Ch. 11 Caches|Next Chapter]]