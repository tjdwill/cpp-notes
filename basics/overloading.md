# Overloading in C++

Date: 26 February 2025

C++ allows function overloading, meaning we can reuse names for functions. This allows us to
implement multiple versions of the same function for convenience. An example of this is found in
constructors. C++ allows us to specify more than one way of constructing a class by varying the
input parameter signature. This is a key point: overloading is only allowed when the input
parameters change. If the parameter signature remains the same but the return type varies, the code
will not compile.

```cpp
void foo(int d);
// auto foo(int d) -> int; // ERROR! 
void foo(double d);
void foo(std::string s, std::vector<int> v);
```

## Operator Overloading

A result of overloading capability is *operator overloading*, where a developer can use C++ syntax
such as `operator+` or `operator<<` to provide convenient syntax for their custom types. An example
application of this is in linear algebra libraries. Instead of creating methods for addition and
multiplication, one can simply overload the `+` and `*` operators, providing the desired behavior
while maintaining a nice syntax (which works to increase readability).

Common overloads include ([among others](https://en.wikipedia.org/wiki/Operators_in_C_and_C%2B%2B#Operators)):

- `operator +`
- `operator -`
- `operator *`
- `operator ()`
- `operator []`
- `operator <<`
- `operator >>`

A great example of overloading the latter two operators `<<` and `>>` is found in the `iostream`
library.

**Update**: 

Here is a GitHub Gist that is useful to view: https://gist.github.com/beached/38a4ae52fcadfab68cb6de05403fa393

### Member vs. Non-member Operator Overloads

[Relevant StackOverflow Post](https://stackoverflow.com/questions/4622330/operator-overloading-member-function-vs-non-member-function)

I thought this was interesting, so I'm including it here. Even though we can define operator
overloads to be member functions for some class (they are just functions, after all), it is
recommended to make **binary** operator overloads non-member functions (generally). We do this for
binary operators to preserve symmetry. Let's say we have some class `Foo` that we want to make
summable with `int`s:

```cpp
class Foo 
{
    /* ... */
public:
    RetType operator+ (int j) 
    {
        /* ... */
    };
};
```

The problem with this implementation is that we can only write an addition expression in one
direction; the `Foo` object has to appear on the left. This is because the `this` pointer is always
the first implicit argument of a non-static member function. Instead, we can define non-member
functions that preserve symmetry, allowing `Foo` to appear to the right of the binary operator:

```cpp
// If we need implementation details
class Foo
{
public:
    friend auto operator+ (Foo const& f, int k) -> RetType;
    friend auto operator+ (int k, Foo const& f,) -> RetType;
};

/*
*   Friend implementations here
*/

//------------------------------------------------------------------------------
// If we don't need implementation details:
class Foo
{
    /*  ...  */
};
auto operator+ (Foo const& f, int k) -> RetType;
auto operator+ (int k, Foo const& f) -> RetType;
```

Where the latter implementation can just call the previous one with the arguments swapped:

```cpp
auto operator+ (int k, Foo const& f) -> RetType
{
    return f + k;  // calls operator +(Foo const&, int);
}
```

An exception to this rule is the spaceship operator `<=>` in which the compiler can generate the
corresponding symmetric comparison function from the member function. 
See the [Notes on Spaceship Operator](./spaceship_operator.md#cool-actions) for more information.

## Resolution

[Read about overload resolution here](https://en.cppreference.com/w/cpp/language/overload_resolution). 

The following is reminiscent to something I came across in an interview:

```cpp
auto foo(std::string const& s) -> std::string;  // 1
void foo(std::string& s);                       // 2

std::string str {"Hi"};

foo(str);  // Calls 2
```

I was confused as to why version 2 was called since I would have thought the more const-correct
version would be called instead. After reading the above link (the section "Ranking of implicit
conversion sequences" 3f), I learned that C++ actually chooses the version whose `cv`-qualification
(const-volatile) is *less* qualified. Therefore, it selects the version that takes a mutable reference.