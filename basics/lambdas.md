# Lambda Functions

**NOTE**: As always, this is a bare-bones article with the information I've found useful.
It is not intended to be comprehensive.


Beginning in C++11, users have the ability to define anonymous function objects (*lambda expressions* or *lambdas* for short). Lambda functions are a way to define *closures* in C++—functions that are pre-defined with some context via zero or more encolosing variables.

According to [LearnCpp.com](https://www.learncpp.com/cpp-tutorial/introduction-to-lambdas-anonymous-functions/) lambdas are actually C++ functors (classes that overload the `operator()`. See [[functor]](#references) ).

## Structure

    <1> <2> <Optnl. 3> <Optnl. 4> <Optnl. 5>
    []  ()    mutable    throw()   -> type 
    {
        <6>
    } 

1. Capture clause (*lambda-introducer*)
2. Parameter list (*lambda declarator*)
3. Mutable specification (optional)
4. Exception-specification (optional)
5. Trailing return type (optional)
6. Lambda body


Here's a simple example:

```cpp
// This is a purely anonymous function.
// Note the use of `auto` assigning the lambda to an identifier.
auto isEven = [](int x) -> bool { return x % 2 == 0; };

assert(isEven(4));
```


### Capture Clause

The capture clause allows the lambda to access variables from its enclosing
scope, the scope in which it is defined. By default, a given captured variable
is passed by value (and made `const`), but if prefixed by an ampersand `&`, it
is captured by (mutable) reference. An empty capture clause `[]` indicates that no
variables from the enclosing scope are captured.

We can change the default behavior of the capture clause if desired:

- `[=]`: All variables from the outer scope referenced in the lambda body are captured by
  value.
- `[&]`: All variables from the outer scope referenced in the lambda body are captured by
  reference.

If the capture clause has more than one element, the behavior modifier must be first:

- `[=, &foo, &bar]`: capture `foo` and `bar` by reference
- `[&, foo, bar]`: capture `foo` and `bar` by value

Note that a capture clause of form `[=, foo] [=, =foo]` or `[&, &bar]` are invalid.

Also note that when defining lambdas within class member functions, we can capture `this`
to access data members and methods.

### Parameter List

Written like a normal function's parameter list. Starting from C++14, you can also define a
generic template by using `auto` as at least one parameter's type, with `auto` serving as a
template parameter.

### Mutable Specification

Lambdas are analogous `const` methods by default, so variable modification is prohibited.
However, if you include the `mutable` specification, the variables captured by value are
modifiable in the lambda body.

### Exception Specification

Specify if the lambda can throw an exception or not (ex. `noexcept`).

## References

- [Microsoft Learn](https://learn.microsoft.com/en-us/cpp/cpp/lambda-expressions-in-cpp?view=msvc-170)
- [functor](https://stackoverflow.com/questions/356950/what-are-c-functors-and-their-uses)
- [LearnCpp 20.6 - Introduction to
  lambdas](https://www.learncpp.com/cpp-tutorial/introduction-to-lambdas-anonymous-functions/)
- [LearnCpp 20.7 - Lambda Captures](https://www.learncpp.com/cpp-tutorial/lambda-captures/)
