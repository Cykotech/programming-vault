# Introduction

Variables are where a program stores individual pieces of data. These are names that represent the location or address of the data in memory. Some languages can refer to these as pointers because they point to the location of the data.

Never forget that naming your variables something meaningful is very important. *Stuff, Misc, Important* don't convey the actual meaning of the data.

```
magicNumber = 69
// What does this mean?

meaningOfLife = 42
```

Many programming languages associate types with variable data. This affects how much data is allocated for the variable. Integers and floating-point numbers will require more space in memory than a Boolean value, but are also fixed amounts of memory as compared to a string.

```cs
int number = 23.5;
bool isSmart = True;
string word = "The Bird";
```

When you need to store many different variables together because of a relation or association to each other, you'll want to create a composite data structure. Most languages will refer to these as a struct or an object.

```cs
public struct CoffeeRecord {
	string Name,
	string Brand,
	int Rating,
	float CostPerPound,
	bool IsDarkRoast,
	string OtherNotes,
}
```

A good real world example of why data should be grouped in a composite data structure is business cards. A business card contains a person's name, phone number, email address, and other important information. And compare that to handing a person 5 pieces of paper each with a different piece of information on it.

# Arrays

An [[array]] is a good way to store a collection of data that would typically be a list. Arrays are essentially rows of variables stored in equally sized contiguous blocks of memory. Each block can store a value of the defined type. Consider lockers in a school highway a real life example of an array.

A value in an array, or element, is accessible by using an index, which is the element's location in memory relative to the start of the array. Because indices are relative to the start of an array, most arrays are zero-indexed.

```cs
int[] nums = [3, 14, 1, 5, 9, 26, 5, 3, 5]
		//    0  1   2  3  4  5   6  7  8
```

With data indexed in an array, it is easier to find the data in memory compared to an arbitrarily named variable. Consider a school locker named *Jeremy's Locker*. You would have to randomly check lockers till you find the correct match. Compared to if you have the locker number. You would be able to quickly narrow down the amount of lockers to search.

While arrays are a composite data structure, if there needs to be a change made to the global state of the array, the change would have to be applied to every element within the array. If you wanted to add a new locker in the middle of a row, you would have to take most or all of the existing lockers out, and shift all the removed lockers down one location. The common practice for adding an item to an array, is to juggle values using a temporary value.

```cs
var temp = arr[i];
arr[i] = arr[j];
arr[j] = temp;
```

If you did not use this temporary or dummy variable, you would end up with both elements containing the same value.

## Insertion Sort

A good way to understand the power of an array's structure is with the use of an algorithm, such as [[insertion sort]]. 

Insertion sort works by sorting a subset of an array, then expanding the subset till the entire array is sorted. A good real life example would be sorting books on a bookshelf. If you start at the left most book, then as you move right down the shelf, you would insert the book in the correct location on the left so that when you reach the end of the shelf, your entire shelf would be sorted.

```cs
void InsertionSort(int[] arr)
{
	int currIndex = 1;
	
	while (currIndex < arr.Length)
	{
		var currElement = arr[currIndex];
		int insertingIndex = i - 1;
		
		while (insertingIndex >= 0 && arr[insertingIndex] > currElement)
		{
			arr[insertingIndex + 1] = arr[insertingIndex];
			insertingIndex--;
		}
		
		arr[insertingIndex + 1] = currElement;
		currIndex++;
	}
}
```

Insertion sort, however, is not efficient. In the worst case scenario, we could iterate through all of the elements in front of the current element. So this algorithm scales proportionally with the square of the number of items in an array.

But this simple algorithm provides key insight into the functionality of arrays. It demonstrates how swapping elements, inserting a new value, and iterating over the collection.

## Strings

Strings are special kinds of arrays that are represented as an ordered list of characters. Each of their elements contains a single character, which can be either a letter, a number, symbol, space, or a special indicator. The end of a string is typically denoted by special character. Characters can be accessed individually by using their index.

Every language has it's own different implementation for strings. Some use a simple array of characters, others use an object that serves as a wrapper class of an array. In the case of an object, the object provides additional functionality such as dynamic resizing or substring searching.

Measuring equality of strings is also interesting. Compared to integers, equality of strings is more complex, because you have to iterate through all the characters in both strings to determine equality.

```cs
public bool StringEquality(string string1, string string2)
{
	if (string1.Length != string2.Length)
		return false;
		
	int n = string1.Length;
	int i = 0;
	
	while (i < n && string1[i] == string2[i])
		i++;
	
	return i == n;
}
```

The cost of this operation grows proportionally with the length of the input strings. Consider comparing two books character by character attempting to find a mismatch. Many programming languages provide a string class that allow direct comparisons, but under the hood, each character must still be iterated over.

[[Ch. 2 Binary Search|Next Chapter]]