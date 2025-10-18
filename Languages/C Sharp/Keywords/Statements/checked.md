A `checked` statement is used for checking and preventing overflow in integral arithmetic operations and conversions. When arithmetic overflow occurs, an [[OverflowException]] is thrown. If the expression is a constant, the exception is thrown at compile time.

```cs
int a = int.MaxValue;

try
{
	checked
	{
		Console.WriteLine(a + 3);
	}
}
catch (OverflowException e)
{
	Console.WriteLine(e.Message); 
	// Arithemetic operation resulted in an overflow.
}
```

The default behavior is `unchecked`, which wraps an integer to the opposite end of the integer type's range.

```cs
Console.WriteLine(a + 3);
// 2
```

