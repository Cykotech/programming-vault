Prevents further modifications to an object. A FrozenError will be raised if modification is attempted. There is no way to unfreeze a frozen object. The returned value is `self`.

```rb
a = [ "a", "b", "c" ]
a.freeze
a << "z" # => FrozenError: can't modify frozen Array
```