# Const Correctness in C++

The `const` keyword in C++ denotes that the entity to which it descibes is intended to be
immutable. The idea of const correctness is that one should aggressively use the keyword to
denote the semantic intention of the parameters and objects used within functions. This
allows both the programmer and the compiler to reason about the program more readily. An
example of this is in concurrency. If we are only considering `const` (i.e. read-only)
objects for a given function, no race conditions are possible, so we don't have to spend a large amount of time
stressing about it.

The keyword also has actual compiler-level guarantees in terms of what methods an object
can invoke. For `const` objects (either declared as such during instantiation or via a
const reference), only `const` methods can be called because these methods signal that the
object is not modified during their execution.

## How do I label `const`ness?

Easily, just use the `const` keyword:

```c++

void print(std::string const& s);  // free function that doesn't modify its input
class Bar
{
    auto count() const -> int;     // const member function that returns some count.
    auto update(int amount);       // non-const member function. Const objects calling this
                                   // function results in a compile-time error.
};
```

Note where I wrote the `const` keywords. For member functions, the keyword comes after the
function parameter list. `const` attaches to whatever is immediately left of it. If nothing is on its
left, it attaches to its immediate right. For consistency's sake, I employ *East-style const* in which I only write `const` to the right of whatever it is describing. I find this method more intuitive and easier to decipher. 

Take a look at [the following article on East/West const](https://ianyepan.github.io/posts/cpp-const/) to learn the difference between the two. Regardless of what you (or your workteam) chooses, be consistent.

## A note on mutability

Sometimes, we actually *do* want to modify something within a const member function (ex.
some internal data member). The way to do this is by declaring the **data member** as
`mutable`. This allows the data member to be modified even within `const` functions.

## `const` and `auto`

Because this has bitten me multiple times, I make a note here. If you want a constant (or mutable) reference to an object via `auto`, you **must** actually specify that on the declaration. So for example:

```c++
#include <iostream>
#include <vector>


void print_vec(std::vector<int> const& v)
{
    for (auto i: v)  // Value Semantics (COPIES)
        std::cout << i << '\n';
}

int main()
{
    std::vector<int> v { 0, 1, 2, 3 };
    print_vec(v);
}
```

Each element in the vector is *copied*. To actually obtain a reference, we need to specify such:

```c++
#include <iostream>
#include <vector>


void print_vec(std::vector<int> const& v)
{
    for (auto const& i: v)  // Reference Semantics
        std::cout << i << '\n';
}

int main()
{
    std::vector<int> v { 0, 1, 2, 3 };
    print_vec(v);
}
```

This is important to internalize, particularly in cases where you expect to modify some mutable object. `auto` would modify a *copy*, so the changes would not be reflected on the intended object. You would have to declare your worker variable's type as `auto&`.

## Helpful Reading

- [ISOCPP Const Correctness](https://isocpp.org/wiki/faq/const-correctness)
- [East/West Const](https://ianyepan.github.io/posts/cpp-const/)
- [Const Correctness](https://arne-mertz.de/2016/07/const-correctness/) by Arne Mertz
