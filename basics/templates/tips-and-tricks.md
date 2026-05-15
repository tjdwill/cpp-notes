# Template Tips and Tricks

## "Iterate" over a list of types

Say we have a set of template specializations whose test code is all the same. 
Instead of writing a test for every specialization, it is useful to have a way
to execute the same code for a set of types.

One way to do that is through recursion over a parameter pack:

```cpp
// The base case that terminates the recursion
template<typename T>
void someTest(){
    // Code that depends on the type here.
}

template<typename First, typename Second, typename... RemainingTypes >
void someTest(){
    someTest<First>();
    someTest<Second, RemainingTypes...>();
}

void runTest(){
    // Run the test for each type in the parameter list.
    someTest<int, std::string, bool, SomeCustomClass>();
}
```

Note that the recursive version of `someTest()` has three template parameters.
This is necessary to properly resolve any ambiguities in template resolution.
Consider what happens when the we finally get down to a two remaining types:

```cpp
someTest<bool, SomeCustomClass>()
{
    someTest<bool>();
    someTest<SomeCustomClass, /* RemainingTypes is empty */>();
}
```
Both function calls resolve to the version of someTest() that takes one template
parameter, completing the recursion. However, if the template function instead
had the form:

```cpp
template<typename First, typename... RemainingTypes>
void someTest(){
    someTest<First>();
    someTest<RemainingTypes...>();
}
```
When we get down to the last type in the parameter pack, there is no way to
resolve the empty parameter pack case for the second function call. The code
won't compile.
