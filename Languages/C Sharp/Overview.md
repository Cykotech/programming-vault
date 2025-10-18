C# is a strictly typed Object Oriented programming language owned and maintained by Microsoft. It is powered by the [[NET|.NET]] framework.

# Variables

When you declare a variable in C#, you must declare a type. You can allow the complier to infer the type using the `var` keyword. But once the variable is initialized, there are no conventional methods to convert the type of the variable. 

```cs
string word = "string";
int number = 3;
bool yes = true;

var unknown = 12;
```

Variables are able to be reassigned and mutable by default. You can prevent this by declaring a variable with the [[Modifiers#^cdd756|const]] keyword. When dealing with objects or references, you would in turn use the [[Modifiers#^54c585|readonly]] keyword.

```cs
const string permanent = "You can't change me!"; 

// Not allowed
permanent = "Invalid reassignment";
permanent += " You can't add this on";

readonly List<int> nums = [1, 2];

// Allowed
nums.Add(3);
nums.Remove(2);

// Not allowed
nums = new List<int>();
```

Variables can also be made to be nullable. This allows variables to only accept the defined type, but can also accept `null` in the case that no data is found.

```cs
string? name = GetNameOrNull();
```

# Types

## Value Types

### Numeric

C# supports integral and floating-point numeric types. Each split into different types that store different sizes of numbers based on bit size.

<table class="csharp-integral"><caption>Integral Types</caption><thead><td>Type</td><td>Size</td><td>Range</td></thead><tr><td><code>byte</code></td><td>8-bit</td><td>0 to 255</td></tr><tr><td><code>sbyte</code></td><td>8-bit</td><td>-128 to 127</td></tr><tr><td><code>short</code></td><td>16-bit</td><td>-32,768 to 32,767</td></tr><tr><td><code>ushort</code></td><td>16-bit</td><td>0 to 65,535</td></tr><tr><td><code>int</code></td><td>32-bit</td><td>-2,147,483,648 to 2,147,483,647</td></tr><tr><td><code>uint</code></td><td>32-bit</td><td>0 to 4,294,967,295</td></tr><tr><td><code>long</code></td><td>64-bit</td><td>-9,223,372,036,854,775,808 to 9,223,372,036,854,775,807</td></tr><tr><td><code>ulong</code></td><td>64-bit</td><td>0 to 18,446,744,073,709,551,615</td></tr></table>

<table class="csharp-float"><thead><caption>Floating-Point Types</caption><td>Type</td><td>Size</td><td>Approx Range</td><td>Precision</td></thead><tr><td><code>float</code></td><td>32-bit</td><td>±1.5e−45 to ±3.4e38</td><td>~7 digits</td></tr><tr><td><code>double</code></td><td>64-bit</td><td>±5.0e−324 to ±1.7e308</td><td>~15-16 digits</td></tr><tr><td><code>decimal</code></td><td>128-bit</td><td>±1.0e−28 to ±7.9e28</td><td>28–29 digits (high precision, base-10)</td></tr></table>

Knowing the ranges of a numeric type is important. If an operation is performed on a numeric type and the result is beyond the range of the type, C# will overflow the number and the result will wrap around.

```cs
short num = 32766;
num += 10;
Console.WriteLine(num);
// -32760
```

You can prevent this behavior by using a [[Statements#^ba24e9|checked]] statement and produce an [[Exceptions#^58df5d|OverflowException]].

The floating-point types can measure digits after a decimal point up to a number of digits depending on the type used. Any digits beyond that precision point are rounded or truncated.

```cs
float f = 1.123456789f;
Console.WriteLine(f);  
// Output: 1.123457   (rounded at ~7 digits)

double d = 1.1234567890123456789;
Console.WriteLine(d);  
// Output: 1.123456789012346   (rounded at ~16 digits)

decimal m = 1.1234567890123456789012345678m;
Console.WriteLine(m);  
// Output: 1.1234567890123456789012345678 (kept all 28 digits)
```

Read More: [[Value Types#^e22ef6|Integers]] [[Value Types#^18fb60|Floating Points]]
### Char 

Chars are single Unicode characters that use 16-bit UTF-16 encoding. Chars are declared with single quotes. `''`

```cs
char letter = 'a';
char digit = '5';
char symbol = '#';
```

Read More: [[Value Types#^54c204|Char]]
### Boolean

The Boolean type is a simple type that has only two possible values, `true` or `false`. The default value of a Boolean is `false`.

```cs
bool isActive = true;
bool isComplete = false;
```

Other values cannot be implicitly converted to a Boolean value. When evaluating a condition, your value must explicitly be a Boolean.

```cs
if (isActive)
{
	Console.WriteLine("Active");
}

int x = 0;
// if (x) { } // Compile Error
```

There are 4 operators that are used for performing comparison operations on Boolean values.

<table><thead><td>Operator</td><td>Example</td><td>Result</td></thead><tr><td><code>! (NOT)</code></td><td><code>!true</code></td><td><code>false</code></td></tr><tr><td><code>&& (AND)</code></td><td><code>true && false</code></td><td><code>false</code></td></tr><tr><td><code>|| (OR)</code></td><td><code>true || false</code></td><td><code>true</code></td></tr><tr><td><code>^ (XOR)</code></td><td><code>true ^ true</code></td><td><code>false</code></td></tr></table>

Read More: [[Value Types#^bd7f34|Boolean]]
### Enumeration

An enumeration type, or `enum` is a group of named constants that are supported by an integral type (default: `int`). The supporting type can be changed to any other integral type. The default value starts at 0 and increments by 1, however you can assign custom values. An unassigned `enum` with use 0 as it's default.

```cs
enum Day : short {
	None = 0,
	Monday = 1,
	Tuesday = 2, 
	Wednesday = 3,
	Thursday = 4, 
	Friday = 5,
	Saturday = 6,
	Sunday = 7
}

Day defaultDay;
// None

Day humpDay = Day.Wednesday;

Console.WriteLine((int)humpDay); // 3
Console.WriteLine(humpDay.ToString()); // "Wednesday"
```

Read More: [[Value Types#^f8ff8b|Enumeration]]
### Structs

Structs are lighter types that are designed to be immutable. Structs are intended not to require inheritance, but can inherit from [[interfaces]] only. Because of their lightweight nature, structs should only typically be 16 bytes or less for improved performance.

```cs
public struct Point
{
	public int X;
	public int Y;
	
	public Point(int x, int y)
	{
		X = x;
		Y = y;
	}
}
```

Structs as value types are very powerful as when you assign them to a new variable, the value is copied, not the reference. A struct allows a parameterless constructor, which then initializes the struct's fields to their default value. i.e. an `int` field will be initialized to `0`.

```cs
Point p1 = new Point(3,4);
Point p2 = p1;
p2.X = 10;

// p1.X = 3 p1.Y = 4
// p2.X = 10 p2.Y = 4
```

Because struct copies are independent, mutable structs can cause many issues and confusion, best practice concludes that structs should be made immutable by either making the struct [[readonly]] or make it's fields [[init]] only.

```cs
public readonly struct Coordinate
{
	public int X { get; init; }
	public int Y { get; init; }
	
	public Coordinate(int x, int y)
	{
		X = x;
		Y = y;
	}
}
```

Read More: [[Value Types#^d224b5|Structs]]
### Tuples

Tuples are a lightweight data structure used for grouping values together. Tuples don't need an explicitly defined type or custom object. By default, you can reference to individual items in the tuple using `.ItemX`. You can also provide your tuple items names for readability sake.

```cs
var tuple = (1, "apple", true);

// tuple.Item1 = 1
// tuple.Item2 = "apple"
// tuple.Item3 = true

var person = (Name: "Alice", Age: 30);

// person.Name = "Alice"
// person.Age = 30
```

Tuples also support deconstruction. You can define a variable name for each item within a tuple.

```cs
var (x, y) = (5, 10);
```

Read More: [[Value Types#^c8981a|Tuples]]
## Reference Types

### Object

The object type is the base type of all types. Regardless of whether a variables type is a value or reference type, it ultimately inherits from `object`. What's important to know about the object type is that it contains important utility methods that are passed to every type in C#.

You can wrap a value type in an `object` reference. This is called boxing. In order to unbox the variable back to it's original type, you must explicitly cast the variable.

```cs
int x = 5;
object boxed = x;
int unboxed = (int)boxed;
```

The use of the object type has been phased out since the introduction of [[generics]] and the [[dynamic]] keyword.

Read More: [[Reference Types#^1c373f|Object]]

### String

Strings represent a sequence of char values and are declared with double quotes. `""` Once a string is created the contents cannot be changed. Whenever an operation is performed on a string, a new string is created. Strings can be combined or concatenated using simple addition.

```cs
string name = "Bob";

// A new string is constructed from scratch using the constructor
name += "by";

string greeting = "Hello, " + name;

// "Hello, Bobby"
```

You can also embed values in your strings using string interpolation. An interpolated string is declared with `$""`. Then when you want to embed a value, you'd inject it into the string with `{}`.

```cs
int age = 32;
string sentence = $"I am {age} years old.";
```

When you want to ignore escape characters or maintain formatting, a verbatim string will get used. A verbatim string is declared with `@""`.

```cs
string path = @"C:\Users\Cykotech\Documents";
string multiline = @"Line1
Line2
Line3";
```

When you want to include special characters or whitespace, you need to use a special escape sequence.

```cs
string text = "Line1\nLine2\tTabbed\\Backslash\"Quote\"";
```

Read More: [[Reference Types#^35fe17|Strings]]

### Record

Records are designed to be immutable, value-based data models. Comparison between records are based on the data within the object, not the memory identity. Declaring a record's primary constructor provides you an immutable type with auto-properties. You can also define an immutable record with `init` setters.

```cs
public record Person(string FirstName, string LastName);

public record Car
{
	public string Make { get; init; }
	public string Model { get; init; }
}
```

Read More: [[Reference Types#^b80075|Records]]

### Class

A class is a type used to store various data and behaviors. Classes are the core of Object-Oriented Programming practices. Classes support two types of data storage, fields and properties. Fields are used for raw storage and are usually private to prevent accidental mutation or reassignment by external classes. Properties allow controlled access to fields through getters and setters and can provide additional logic to retrieving data.

```cs
public class Person
{
	public string FirstName { get; set; }
	public string LastName { get; set; }
	
	private int _age; // field
	public int Age // property
	{
		get => _age;
		set
		{
			if (value < 0) throw new ArgumentException("Age must be
			positive");
			_age = value;
		}
	}
	
	public void Greet()
	{
		Console.WriteLine($"Hello, I'm {FirstName} {LastName}");
	}
}
```

While other types offer constructors, class constructors are powerful because you can add additional logic beyond property assignment. You define a classes constructor by defining a method without a return type that has the same name as the class.

```cs
public class Car
{
	public string Make { get; }
	public string Model { get; }
	
	public Car(string make, string model)
	{
		Make = make;
		Model = model;
	}
}
```

Classes can also derive their implementations from other classes through inheritance. Inheritance practices also provide ways to override the parent class's methods using the [[virtual]], [[override]], and [[abstract]] keywords.

```cs
// This class cannot be instatiated only inherited

public abstract class Shape
{
	public abstract double Area(); 
}

public class Circle : Shape
{
	public double Radius { get; set; }
	public override double Area() => Math.PI * Radius * Radius;
}
```

Class member access can also be controlled using various keywords.

<table><thead><td>Keyword</td><td>Access</td></thead><tr><td><code>public</code></td><td>Everywhere</td></tr><tr><td><code>private</code></td><td>Only with in the class</td></tr><tr><td><code>protected</code></td><td>Within the class and any derived classes</td></tr><tr><td><code>internal</code></td><td>Within the same assembly</td></tr><tr><td><code>protected internal</code></td><td>Within a subclass or the same assembly</td></tr><tr><td><code>private protected</code></td><td>Within a subclass of the same assembly</td></tr></table>

See More [[Reference Types#^b23261|Classes]]

### Interfaces

An interface is a defined contract, in which any class or struct that inherits the interface must provide a concrete implementation of any members defined. You can define a default implementation for any members of an interface that will be implemented upon inheritance of the interface.

```cs
public interface IAttack
{
	void Attack();
	
	void LogDamage(int damage)
	{
		Console.WriteLine($"Target recieved {damage} points of damage");
	}
}

public interface IDefend
{
	void Defend();
}

public class Warrior : IAttack, IDefend
{
	public void Attack() => Console.WriteLine("Attacks");
	public void Defend() => Console.WriteLine("Defends");
}
```

See More [[Reference Types#^826a5e|Interfaces]]

### Collections and Arrays

C# provides two different types for storing large amounts of data, arrays and collections. Arrays are fixed-size and cannot be resized without recreating the array. Collections include many different types of collections that are dynamic in size and are optimized in different ways to either improve performance or improve data organization.

<table class="csharp-collections"><thead><td>Collection</td><td></td></thead><tr><td><code>List&lt;T&gt;</code></td><td>Dynamic Array</td></tr><tr><td><code>Dictionary&lt;TKey, TValue&gt;</code></td><td>Key/Value Pairs</td></tr><tr><td><code>Queue&lt;T&gt;</code></td><td>FIFO</td></tr><tr><td><code>Stack&lt;T&gt;</code></td><td>LIFO</td></tr><tr><td><code>HashSet&lt;T&gt;</code></td><td>Unique Values</td></tr></table>

All collections can be iterated through using a [[Statements#^c26ed8|foreach]] statement. 

```cs
foreach (var item in list)
{
	Console.WriteLine(item);
}
```

See More [[Reference Types#^235f95|Collections and Arrays]]

# Keywords

C# provides predefined reserved identifiers that serve a specific purpose for the compiler. These keywords cannot be used as user-defined identifiers unless prefixed with `@`. i.e. `@if`

