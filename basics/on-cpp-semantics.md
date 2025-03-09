# On C++ Semantics

## Copy Semantics

## Reference Semantics

## Move Semantics

## With `auto`

On numerous occasions, I've fallen victim to C++'s use of implicit copy (value) semantics.
I expect to mutate an object, only to have mutated a copy. Both occasions of this occurring
was due to lack of specifiers when declaring an `auto` variable. To fix, ensure you attach
the reference specifier to the declaration:

```cpp
auto var = obj.get_mut(); // will return a copy!

auto& var = obj.get_mut(); // will return a reference!


// assume `some_vector` is a vector of strings..

for (auto const& x: some_vector) // returns references to constant vector elements.
    std::cout << x;

for (auto x: some_vector)  // tries to copy elements.
    std::cout << "Copied: " << x << \n";
```

