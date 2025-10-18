Ruby is an object-oriented language with an emphasis concise syntax. It is very commonly paired with it's powerful [[Rails]] framework.
# Syntax

## Variables

Declaring variables does not require any special keywords as many other languages. All that is required is the name of the variable.

```rb
x = 3
word = "string"
list = [1, 2, 3]
```

Variables are mutable by design. You can prevent object mutation by using .[[freeze]], but this does not prevent reassignment. Ruby also has constant variables, but only by convention. By writing a variable's name in caps, you will not prevent reassignment, but you will receive a warning when attempted.

```rb
string = "hello"
string.freeze

string.upcase! # => FrozenError: can't modify frozen String

NAME = "Jon"
NAME = "Jane" # Warning: already initialized constant NAME
```

## Data Types

### Numbers

Ruby has 3 main numeric types which all inherit from the [[Numeric]] class.

<table class="ruby-numeric"><thead><td>Type</td><td>Description</td><td>Example</td></thead><tr><td>Integer</td><td>Represents whole numbers</td><td><code>x = 2</code></td></tr><tr><td>Float</td><td>Represents decimal numbers using floating-point precision</td><td><code>y = 3.14</code></td></tr><tr><td>Rational</td><td>Represents fractions (numerator/denominator)</td><td><code>z = Rational(2, 3) # => (2/3)</code></td></tr><tr><td><a href="/Numeric/Complex">Complex</a></td><td>Represents complex numbers with real and imaginary parts</td><td><code>i = Complex(2, 3) # => 2 + 3i</code></td></tr></table>

### Symbols

Ruby introduces a unique concept called Symbols that are immutable, string-like objects. The primary use of these are as identifiers or labels. Symbols are singletons, only one copy can exist in memory.

```rb
# As hash key labels
person = {
	name: "Joe",
	age: 30
}

# As method names/arguments
send(:puts, "Hello")
```

### Booleans

Like many other languages Ruby also supports Boolean values. Something is considered true if it is an instantiation of a class.

<table class="ruby-boolean"><tr><td><code>0</code></td><td>True</td><td><code>Numeric()</code></td></tr><tr><td><code>[]</code></td><td>True</td><td><code>Array()</code></td></tr><tr><td><code>{}</code></td><td>True</td><td><code>Hash()</code></td></tr><tr><td><code>""</code></td><td>True</td><td><code>String()</code></td></tr></table>

Ruby supports the standard comparison and logical expressions/operators as seen in many other languages.

```rb
5 > 3 # => true
nil.nil? # => true
!true # => false
true && false # => false
true || false # => true
```

