# Error Handling in C++

Though critical in all of our software, error handling is still a hotly-debated
topic in multiple language communities. Being a language of multiple paradigms,
C++ has multiple methods of error handling with various tradeoffs. Here are the
following methods:

- Exceptions
- Error codes
- `std::expected<T, E>`
- Assertions
- Logging

## General Observations

What interests me about these mechanisms is that, while all three have their
differences, they are used in similar ways. Perform a fallible operation, check
for an error, and handle error if it exists. Otherwise, continue to next
instruction. Also, for exceptions and algebraic types, there is a tendency for
propagation: an error instantiated at some lower-level "bubbles up" to be
handled at a higher level. They also tend to inspire a desire for a generalized
error that encapsulates the more specified versions of the type. For example,
with exceptions, it's common to have more specific exception types to inherit
from a general `Exception` type. As a result, if the programmer creates some
exception type `Foo` that inherits from `Exception`, all such exceptions will be
caught if using `Exception` as the basis of error handling. This way, if we
aren't actually concerned about *which* exception occurred, the error is still
handled.

## Exceptions

One area where exceptions are less useful is concurrent code. For example,
multi-processed programs can't share exceptions. In those cases, it is more
reasonable to return error codes or algebraic data types to denote error.

## Error Codes

Error codes form a classic error handling method. It's used extensively in C
programming since C has no other error handling methods.

## `std::expected<T, E>`

This is basically an error code, but it becomes part of a procedure's return
signature.

## Assertions

Assertions are used to verify expected state before, during, and after the
execution of a function. I've learned that assertions are mostly for developers;
they are used to detect logical errors that result from incorrect code. These
errors should not happen. 

## Logging

While not a traditional entry in this conversation, logging, a more generalized
version of print debugging, is a form of handling errors given that the purpose
of many error handling efforts is to detect, diagnose, and fix bugs.

Specifically, logging is useful in contexts where code execution isn't
necessarily deterministic like with concurrent code. Logs allow the developer to
trace through execution to detect synchronization errors, errant file accesses,
incorrect program state, etc.

It is also a useful method for forensic purposes. Some errors only occur
sporadically, so having a means of capturing a potentially once-in-a-blue-moon
error is vital in long-running applications.