# Introduction to GIL

GIL is _mostly_ a statically typed language:
it enforces strict typing at compile time,
except when you choose not to by using dynamic typing.

GIL could be compiled or it could be interpreted.
The type system is designed to allow for efficient
machine-level implementation and execution,
i.e. an integer is just a machine word.

But it's also expressive and pragmatic enough for quick hacks.

## High-level structure

Here's the simplest possible GIL program, which does nothing and exits successfully:

```
func main(args: [string]): int {
    return 0
}
```

Every GIL program must include exactly one `main()` function with this signature.
`main()` takes a single parameter, a list of strings
which is conventionally called `args`.
And it returns a single integer value
(which becomes the exit status of the OS process that runs this program).

## Functions

The building block of a GIL program is functions.
All code must be defined in a function -- nothing executes outside of one.
The simplest possible function takes no parameters,
does nothing, and returns nothing:

```
func do_nothing() {
}
```

The absence of a return type indicates that this is a _procedure_,
a function that returns no value.
Inside a procedure, the `return` statement is optional and always bare.

Function names consist of lowercase letters, underscores, and digits
(but they cannot start with a digit).

Most functions have one or more parameters and a return value,
whose types must all be declared:

```
func greeting(name: string): string {
    return "hello " + name
}
```

When we need to distinguish functions that return a value from procedures,
we'll call them _normal functions_.
Simplifying a bit, every normal function must have an explicit return value.
Every reachable code path must have a `return` statement,
and the function must return a value of the correct type.
(There are situations, notably infinite loops and exceptions,
where the rules around reachable code and `return` statements are a little loose.)

```
// error: function 'invalid' contains a code path with no return statement
func invalid(n: int): int {
    if n < 100 {
         return n * 2
    }
}
```

## Variables and Values

A variable, whether it is a function parameter or defined inside a function,
always has type and a value.
Declaring the type and assigning the value are technically separate:

```
n: int
n = 50
```

But this is so common that you can do them together:

```
n: int = 50
```

If you don't explicitly assign a value,
the variable will take the _default value_ for its type:
integer 0, boolean false, float 0.0, etc.
So the second example is a smidge more efficient, because `n` is only assigned one value.
In the first example, the default value for `int` (0) is assigned,
and then 50 is assigned
(although a clever compiler would hopefully optimize this away).

If you don't explicitly declare a type, GIL infers a type at compile time:

```
num = 50              // inferred type: int
size = 1.5            // inferred type: float
name = "Bob"          // inferred type: string
```

GIL can infer types from more complex expressions:

```
another_num = (num * 2) + 3   // inferred type: int

g = greeting(name)            // inferred type: string
```

The value of a variable always lives in a contiguous area of memory.
(This does not preclude the creation of complex data structures like linked lists!
We'll cover this below, when we talk about pointers.)

## Primitive Types

So far we've only seen two of GIL's builtin types: `int` and `string`.

`int` is a signed integer the size of one machine word:
on a 32-bit machine it's a 32-bit integer,
and on a 64-bit machine it's a 64-bit integer.

GIL has a rich complement of numeric types:
`uint` is an unsigned machine word;
and `int8`, `uint8`, `int16`, `uint16`, etc (up to 64-bit)
specify the exact number of bits.

Numeric literals always use the machine-specific type, `int` or `float`.
If you specifically want (say) an unsigned 32-bit integer,
you need to say so explicitly:

```
num: uint32 = 50
```

`byte` is an alias for `uint8` -- if it were not a builtin type,
you could define it yourself with

```
type byte: uint8
```

There are also `bool`, `float`, `float32`, and `float64` types.

This is the set of GIL's _primitive types_,
which are supported directly in hardware of any modern processor.

## Alias Types

With `byte`, we've seen one alias type which is built-in to the language.
You can define as many alias types as you like:

```
type user_id: int8
type group_id: int8
```

After this, types `user_id` and `group_id` are _nearly_ equivalent to `int8`.
Specifically the same pattern of bits in memory has the same meaning:
00000001 is 1, 11111111 is -1, etc.

But these types are not interchangeable:

```
func lookup_username(user_id: user_id): string {
    ...
}

// error: lookup_username() takes parameter of type user_id, not group_id
func main(args: [string]): int {
    n: group_id = 17
    name = lookup_username(n)
}
```

(This example also demonstrates a peculiar but useful feature:
types and variables live in separate namespaces,
so it's valid to declare a variable `user_id` of type `user_id`.)

We'll discuss alias types further when we talk about
type casting and type conversion, below.

## Composed Types

It's rare that you can build useful software without combining
many values into one larger data structure.
That's what composed types are for.

The most common composed type is `string`,
but we need to cover a few more things before learning how GIL implements strings.

The simplest composed type is the _dynamic array_,
which is usually just called an array.
(GIL also has fixed-size multidimensional arrays,
but those are mostly useful in specialized mathematical and scientific programs.)

Here's an array of integers:
```
values = [4, -1, 8, 10]
```

This single line of code actually involves two distinct types.
The _concrete_ type of `values` is `array[int]`: a dynamic array of `int`.
That determines how the data is represented in memory:
as 5 machine words, one for the length and one for the 4 values.
For example, on a 32-bit machine, this array would take up 20 contiguous bytes in memory:
```
00000004          // length as a 32-bit integer
00000004          // values[0]
ffffffff          // values[1] (2's complement signed 32-bit integer)
00000008          // values[2]
0000000a          // values[3]
```

(It's actually a bit more complex than that:
to allow dynamic resizing, the `values` array is a _pointer_ to
that 20-byte region of memory.
We'll cover pointers below.)

But the inferred type for `values` is not `array[int]`, it is `list[int]`:
a list of integers.
This is such a common thing to declare that it has a shorthand notation, `[int]`.

Thus, the above code is equivalent to
```
values: [int]       // aka list[int], aka list of integer
values = [4, 1, 8, 9]
```

`list[int]` is an abstract type, which has nothing to do with bits in memory.
We'll talk more about abstract types in a bit.
Arrays are only special because they are the default implementation of lists.

But first, it's useful to know about GIL's other builtin composed types.
Next up is the map, which associates _keys_ and _values_.
The default implementation of map is the hashmap.

```
age: {string: int}                // abstract type: map from string to int
age = {"bob": 32, "dan": 21}      // concrete type: hashmap from string to int
```

Finally, there is the set:

```
members: {string}                 // abstract type: set of string
members = {"tim", "jed"}          // concrete type: hashset of string
```

## Abstract Types (Interfaces)

`int`, `float`, `bool`, and `array[int]` are all concrete types.
The actual bits in memory, and how they are interpreted, are part of the definition of the type.

Interfaces, also known as abstract types, don't work that way.
Instead, an interface specifies _what you can do_ with a value.

For example, you could define an interface `Collection` to mean
"any value with a method called `.len()` that returns an int":

```
type Collection: interface {
    len(): int
}
```

Strings, arrays, hashmaps, and hashsets all implement this interface.
This means that you can declare a function to take `Collection`,
and pass any value to it that implements that interface.

```
func double_len(collection: Collection): int {
    return collection.len() * 2
}

func main(args: [string]): int {
    // string and array of float both implement Collection
    println(double_len("foobar"))
    println(double_len([4.0, 1.3, -5.2]))

    // error: bool does not implement Collection
    println(double_len(false))
}
```

An interface can specify any number of methods, operators, and fields.

The simplest possible interface is the empty interface,
which specifies nothing at all.
This is another builtin type, which would be declared as

```
type any: interface {}
```

Every value of any type in a GIL program implements this interface.

```
x: any
x = 3               # int implements the empty interfaace
x = true            # bool implements the empty interface
x = "foo"           # string implements the empty interface
x = ["foo", "bar"]  # array[string] implements the empty interface
```

Experienced developers will have figured out by now that
`array[int]` and `array[bool]` are distinct types.
(Likewise for `list[int]` and `list[bool]`.)
Ignore that for now: we'll cover that later, when we get to _generic types_.

## Pointers

## Strings

