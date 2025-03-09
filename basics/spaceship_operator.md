# Spaceship Operator `<=>`

C++20 introduced a new comparison operator which has been coined the *spaceship operator* `<=>`.
Previously, defining overloaded comparison operators required one method per operation for a minimum
count of 6 (`<`, `>`, `==`, `<=`, `>=`, and `!=`) for a given type. The count increases when
additional types are compared to the new type. Additionally, the operators were mostly implemented
in terms of the `<` operator, which [is an unsuitable primitive for multiple
reasons](https://brevzin.github.io/c++/2019/07/28/comparisons-cpp20/#comparisons-in-c98-thru-c17).

The spaceship operator introduces comparison categories, solving the problem of the unsuitable
primitive, and also enabling C++ developers to develop comparison overloads using a maximum of two
methods per type. The six traditional operators are derived by the compiler as a function of the
`<=>` and `==` implementations (though, importantly, not actually generated as functions). A `<=>`
comparison returns a category object taking on [one of the following
values](https://brevzin.github.io/c++/2019/07/28/comparisons-cpp20/#a-new-ordering-primitive-):

- `strong_ordering`: where equality implies that `b` can be substituted for `a` (and vice-versa) in
  most functions.
    - `strong_ordering::greater`
    - `strong_ordering::equal`
    - `strong_ordering::less`
- `weak_ordering`: "equality" implies equivalence under some metric or scenario (the example given
  in the article is that of case-insensitive string comparison)
    - `weak_ordering::greater`
    - `weak_ordering::equivalent`
    - `weak_ordering::less`
- `partial_ordering`: some values can't be compared, so are unordered (ex. NaN)
    - `partial_ordering::greater`
    - `partial_ordering::unordered`
    - `partial_ordering::less`

All three category objects are compared to the zero literal `0` to evaluate the expression:

```cpp
strong_ordering::less < 0     // true
strong_ordering::less == 0    // false
strong_ordering::less != 0    // true
strong_ordering::greater >= 0 // true

partial_ordering::less < 0    // true
partial_ordering::greater > 0 // true

// unordered is a special value that isn't
// comparable against anything
partial_ordering::unordered < 0  // false
partial_ordering::unordered == 0 // false
partial_ordering::unordered > 0  // false
```

Meaning, any `a @ b` is turned into `(a <=> b) @ 0` where `@` is a relational operator (ex. `<`).


## Cool Actions

`==` and `<=>` are defined as the new primary operators. One cool feature of the primary operators
is that they are reversible. Meaning if we define some member function implementing `==` for a
non-Self type (ex. `Foo::operator==(double d)`), the compiler will use this function for both the
`foo == d` and the `d == foo` cases, saving the amount of functions we need to implement. The two
primary operators (and **only** the primary operators) are symmetric. In the case of the `<=>`
operator, however, there is a caveat: the symmetric case evaluates to `0 <=> (a <=> b)` (see the
linked article for more). Addtionally, the compiler doesn't even need to generate new functions to
handle these cases.

Also, the [comparisons can be
defaulted](https://brevzin.github.io/c++/2019/07/28/comparisons-cpp20/#defaulting-comparisons),
making lexicographical comparison implementations a breeze.
