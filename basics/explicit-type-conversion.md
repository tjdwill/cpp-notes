# Type Conversion

In C++, we may need to convert between types. For example, sometimes we may want to interpret an
unsigned integer as a signed integer or float, or we may want to interpret a derived class as a base
class. In C, there was the `(<type here>) object` cast mechanism, but this is not recommended in C++
because of the lack of semantic information and the difficulty in finding uses, especially in larger
codebases.

Instead, C++ provides named conversion types:

- `static_cast`: Converts between related types (ex. floating-point to integral type (and
  vice-versa) or conversions defined by constructors/conversion operators)
    ```cpp
    int to_int { static_cast<int>(3.14) };
    ```
- `dynamic_cast`: Run-time-checked  conversion of pointers and references into a class hierarchy
    - Convert between base classes and derived classes
- `reinterpret_cast`: Interpret the bits of an object as another potentially unrelated type.
- `const_cast`: bypasses `const` or `volatile` to provide write access to the object.