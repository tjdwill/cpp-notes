# Inheritance in C++

Date: 6 November 2024

Coming from Python, I understand the basics of inheritance in that a user-defined type
adopts behaviors and representative data members from some set of ancestral types.
Inheritance can be quite involved from what I've seen, so I want to be sure I understand
the *why* before concerning myself with the technical *how*.

Inheritance in programming is a concept use to describe **Is-A** relationships. Some entity
`X` **is a** form of entity `Y`. Programmers wield inheritance to establish a common set of
behaviors for inheriting subclasses.  In C++, inheritance is often used to implement
*polymorphism* which, to my knowledge, is the idea that, as long as some set of entities of
possibly-distinct types adheres to a common behavioral contract, we don't care which type
among the set is in active use when invoking said behavior.

## When to use Inheritance

**NOTE**: This will likely be revised and refined as I learn more.

The common answer to this question is "when expressing an is-a" relationship, but I want to
explore some more. So far, whenever I come across inheritance, the goal was to establish
that inheriting types implement some behavior. In other words, they *can do* some behavior
`K`. While inheriting classes also inherit the data members, I've never seen an instance
where this was the *reason* for using inheritance. 

When I think about the concept's use in C++ and Python, it seems that the focus of
inheritance is on a class's behavior, not its structure. An analog to this would be Rust's
concept of *traits*. Rust doesn't support inheritance, but it still achieves polymorphism
via adherence to traits, defined behavioral specifications. Perhaps inheritance should be
used to have a set of abstract types representing some behavior or operational concept
which are then inherited from to guarantee behavior in a concrete type?

Think about the fact that even in implementation, the focus in C++ is primarily on methods
(virtual or non-virtual); the focus is on the *behavior*. 

## Access Specifiers and Inheritance

First, let's review what the access specifiers are. An *access specifier* is a keyword that determines the level of access to a class member. We have three such specifiers: `public`, `private`, and `protected`. `public` members (data, functions, classes, enums, etc. ) are accessible by everyone. `private` members are only accessible by members within the class itself. Finally, `protected` members are accessible by class members *and* derived classes. By default, `class` members are `private`, and `struct` members are `public`.

When inheriting from a class, one is able to specify the type of inheritance. Inheritance type specifies how a derived class exports the accessibility of derived members. There are three types: `public`, `private`, and `protected`. 

```cpp
class Foo
{
};
class Bar : <access specifier> Foo  // inherit from `Foo`
{
};
```

`public` inheritance is the most commonly-used inheritance type (and the one you should select by default). If a class `Bar` inherits from `Foo` publicly,  any subsequent `Baz` that inherits from `Bar` will experience the same access of `Foo`'s members: `public` members remains `public`, `protected` members remain `protected`, and `private` members of `Foo` are still inaccessible to derived classes. 

`protected` inheritance is the *least* used. Under this inheritance type, when `Bar` inherits `Foo`, `Foo`'s `public` members become `protected` to subsequent subclasses of `Bar`.

Finally, with `private` inheritance, both `public` and `protected` members of `Foo` become `private` within `Bar`, such that these members are inaccessible to subclasses of `Bar`. This specification is the *language* default for classes, but we pretty much always inherit publicly in practice.
### Access vs. Visibility

It is important to note the distinction between "access" and "visibility".  Even if an entity has private access, it is still visible. In his talk about Back to Basics C++ Software Design, Klaus Iglberger gave an example of this distinction. Given the following:

```cpp
// https://godbolt.org/z/E9xMsW8ss
class Foo
{
public:
	auto foo(int x) const -> double;
private:
	auto foo(double x) const -> double;

	int mX {};
};

int main()
{
	Foo test{};
	test.foo(5.3);  // resolves to the private overload; compiler error
}
```

When first encountering this example, I assumed the public method would be called, and the input argument narrowed into an `int`, but that assumption was incorrect. As we can see, the private method is still visible and selectable via name resolution, but it is inaccessible due to the `private` specifier, so the compiler throws. 

## `final`

The [`final`](https://en.cppreference.com/w/cpp/language/final) keyword has two uses. When specified on a virtual function, it means this function cannot be overridden by derived classes. When used in a class definition, it means that class itself cannot be derived from. 

## Additional Remarks

- Because I had this question at one point, know that only one object is created when a derived object is constructed, even if base class constructors are called. One way to think about it is that the derived class has dedicated sections where base-class-relevant data goes. It's not 100% precise (or even accurate), but that way of thinking helped clarify the concept for me. 