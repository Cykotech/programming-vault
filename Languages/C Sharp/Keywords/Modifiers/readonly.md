The `readonly` keyword can be used in the following contexts:
- Field declaration
- A `readonly struct` type definition
- Instance member declaration within a structure type
- A `ref readonly` method return

In field declaration, a field cannot be reassigned once the containing object's constructor exits. In the case of a value type, that field is immutable and cannot be modified. If the `readonly` field is a reference type, the reference cannot be changed through reassignment, but is mutable only if the original object is mutable.

```cs
class Age
{
	private readonly int _year;
	private readonly Person _person;

	Age(int year)
	{
		_year = year;
	}

	void ChangeYear()
	{
		_year = 1967;
		// Compile error
	}

	void ChangePerson()
	{
		_person = new Person();
		// Compile error
	}

	void ModifyPerson()
	{
		_person.Name = "John Doe";
		// Allowed if the Person type is mutable
	}
}
```

In a `readonly struct`, all members must be either `readonly` or [[init]] only.

```cs
public readonly struct Coords
{
	public Coords(double x, double y)
	{
		X = x;
		Y = y;
	}

	public double X { get; init; }
	public double Y { get; init; }
}
```

In a [[Struct]], you can declare a `readonly` instance member. This is typically done in a case where the struct is not `readonly`, when you want to define members that don't modify the state of the struct. A `readonly` member can call non-`readonly` members, but cannot modify them.

```cs
public readonly double Sum()
{
	return X + Y;
}
```

