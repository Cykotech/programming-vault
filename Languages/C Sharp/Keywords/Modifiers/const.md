The `const` keyword is used to define a constant field or variable that cannot be modified or reassigned. Constants can be numerical, Boolean, strings, or `null`. Because constants cannot be reassigned, they must be declared upon initialization. [[String Interpolation|Interpolated Strings]] can be constant if all contained expressions are also constant.

```cs
const double GravitationalConstant = 9.81;

const string Language = "C#";
const string Platform = ".NET";

const string FullProductName = $"{Platform} - Language: {Language}";
```

Constant declarations can also declare multiple variables

```cs
const int X = 1, Y = 2, Z = 3;
```

The [[static]] modifier is not permitted in a constant declaration.