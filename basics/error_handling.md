# Error Handling in C++

Though critical in all of our software, error handling is still a hotly-debated topic in multiple language communities. Being a language of multiple paradigms, C++ has multiple methods of error handling with various tradeoffs. Here are the following methods:

- Exceptions
- Error codes
- `std::expected<T, E>`

## General Observations

What interests me about these mechanisms is that, while all three have their differences,
they are used in similar ways. Perform a fallible operation, check for an error, and handle
error if it exists. Otherwise, continue to next instruction. Also, for exceptions
and algebraic types, there is a tendency for propagation: errors instantiated at
some lower-level "bubbles up" to be handled at a higher level. They also tend to
inspire a desire for a generalized error that encapsulates the more specified
versions of the type. For example, with exceptions, it's common to have more
specific exception types to inherit from a general `Exception` type. As a result,
if the programmer creates some exception type `Foo` that inherits from `Exception`,
all such exceptions will be caught if using `Exception` as the basis of error
handling. This way, if we don't aren't actually concerned about *which* exception
occurred, the error is still handled.

## Error Codes

Error codes form a classic error handling method. It's used extensively in C
programming since C has no other error handling methods.
