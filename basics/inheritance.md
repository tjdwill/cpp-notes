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
