#dsa
# Introduction

Caches are a data structure designed to alleviate high data access by storing data closer to a computation. Algorithmic efficiency is very much effected by the cost of data access. How data is stored is not the only factor in this regard, but the type of storage that is used. Caches can be used to increase performance of websites. The browser will often cache commonly accessed data if a web page consists of large amounts of data such as images or videos. Instead of redownloading the logo of a website every time a new page loads, the browser will store the image in the local harddrive to reduce load times.

How the data is selected to be stored in a cache is the key. If every piece of data was stored in the cache, then the cache wouldn't be needed in the first place. One such strategy is the Least Recently Used (LRU) cache.

# Introducing Caches

Computer storage can be commonly thought of as a bookshelf, where accessing a single book is a simple task if you know where to look. But as the size of programs grow, the bookshelf grows to the size of a multi-story library. The most commonly read books can be stored on a shelf at the entrance, while books that might not have ever been read might be stored in the basement.

The organization of data is not the only single factor to consider, but where the data is stored. Different types of storage have different trade offs, such size available and speed of access. CPU memory is incredibly fast, but offers limited storage space, Random-Access Memory (RAM) offers a slightly larger size but at slightly lower speeds, and an external hard drive has a massive size, but is signifigantly slower than the previous stores. Speeds decrease signifigantly when it comes to accessing data on the internet.

Caches are the step before accessing any expensive data. Before the distant server is called out to, the cache gets checked to validate whether the data is already stored locally. If the data is found, this is called a *cache hit*, and the request can be stopped, otherwise a *cache miss* is returned and the data needs to be retrieved from the source.

It is important to consider how much data is stored in a cache as more data will require larger and slower storage. As the cache fills up, there needs to be a determination as to which data is replaced, or *evicted* from the cache. Different caches require different eviction strategies varying from counting how often data is accessed to predicting when the data will be used again.

# LRU Eviction and Caches

LRU cache stores recently used information and present a simple, common approach to understanding caches. The intuition behind this strategy is that data is re-accessed when it was recently needed, such as with a large website that uses a company logo on every page. LRU caches contain a set amount of storage of the most recently used items and evict the least recently used data when the cache fills up. 

## Building an LRU Cache

The LRU eviction policy requires two operations: looking up an abitrary element and finding the least recently used element to delete. Both operations must be performed rapidly.

An LRU cache can be built using both a [[Ch. 10 Hash Tables|hash table]] and a [[Ch. 4 Stacks and Queues|queue]]. The hash table enables fast lookups of data stored in the cache, and the queue tracks which item has not been used recently. Instead of scanning the entire cache for the latest timestamp, the cache can dequeue the earliest item, providing an efficient method to determine which data to remove from the cache.

```cs
class LRUCache
{
	public HashTable Hash { get; set; }
	public Queue Q { get; set; }
	public int MaxSize { get; set; }
	public int CurrentSize { get; set; }
}
```

Each entry in the hash table stores a composite data structure containing at least 3 pieces of information: the key, the value, and a pointer to the corresponiding node in the queue. The pointer is essential as nodes in the queue will need to be looked up and modified as data is accessed in the cache.

```cs
class CacheEntry
{
	public T Key { get; set; }
	public T Value { get; set; }
	public QueueListNode Node { get; set; }
}
```

The only defined lookup function will check if data exists in the cache, if there is a cache hit, the data will be returned and it's recency will be updated, otherwise the program will go to the expensive data store, add the data to the cache, and evict the least recently used data if necessary.

```cs
public T Lookup(LRUCache cache, T key)
{
	CacheEntry entry = cache.HashTable.Lookup(key);
	
	if (entry == null)
	{
		if (cache.CurrentSize >= cache.MaxSize)
		{
			T keyToRemove = cache.Queue.Dequeue();
			cache.HashTable.Remove(keyToRemove);
			cache.CurrentSize--;
		}
		
		T data = Fetch();
		
		cache.Queue.Enqueue(key);
		entry = new CacheEntry(key, data, cache.Queue.Back);
		cache.HashTable.Insert(key, entry);
		cache.CurrentSize++;
	}
	else
	{
		cache.Queue.Remove(entry.Node);
		cache.Queue.Enqueue(key);
		
		entry.Node = cache.Queue.Back;
	}
	
	return entry.Value;
}
```

## Updating an Element's Recency

In order for a remove operation to be supported, the queue needs to support updating an element's position. The queue in an LRU cache will use a doubly linked list to enable efficient requeuing of items in the middle of the queue. 

```cs
class QueueListNode
{
	public T Value { get; set; }
	public QueueListNode Prev { get; set; }
	public QueueListNode Next { get; set; }
}
```

Extra logic is provided to the standard operations of the queue to support the doubly linked list.

```cs
public void Enqueue (Queue q, T value)
{
	QueueListNode node = new QueueListNode(value);
	
	if (q.Back  == null)
	{
		q.Front = node;
		q.Back = node;
	}
	else
	{
		q.Back.Next = node;
		node.Prev = q.Back;
		q.Back = node;
	}
}

public T Dequeue (Queue q, T value)
{
	if (q.Front == null)
		return null;
		
	T value = q.Front.Value;
	q.Front = q.Front.Next;
	
	if (q.Front == null)
		q.Back = null;
	else
		q.Front.Prev = null;
	
	return value;
}
```

The `RemoveNode` operation is then added. Because the queue is maintained as a doubly linked list, the queue does not need to be scanned multiple times during this operation.

```cs
public void RemoveNode (Queue q, QueueListNode node)
{
	if (node.Prev != null)
		node.Prev.Next = node.Next;
	
	if (node.Next != null)
		node.Next.Prev = node.Prev;
	
	if (node == q.Front)
		q.Front = q.Front.Next;
	
	if (node == q.Back)
		q.Back = q.Back.Prev;
}
```

It is important to remember to maintain the pointer from the cache's hash table entries to the corresponding nodes in the queue to support efficient lookups for updating data's recency.

# Other Eviction Strategies

Alternative strategies for evicting data from a cache include: most recently used, least frequently used, and predictive eviction. Each strategy has it's own tradeoffs and which one to select when implementing a cache will vary depending on the scenario.

LRU is optimal when there are bursts of commonly used data, but the distribution of the data varies over time. Most Recently Used (MRU) caches are useful when data is less likely to be consumed multiple times. Time of access is not the only metric to determine eviction. Least Frequently Used (LFU) caches will count how many times data has been accessed from the cache, and discard the lowest count upon eviction. This is useful when the accessed data is expected to remain stable.

The predictive eviction strategy involves a forward-thinking approach that tries to predict which elements will be needed in the future. Instead of relying on counts or timestamps, a model is built to predict which entries will be accessed in the future. The efficiency of this cache is determined by the accuracy of the model. The primary downside of a predictive cache is the added complexity of building the model.

[[Ch. 10 Hash Tables|Previous Chapter]]
[[Ch. 12 B-Trees|Next Chapter]]