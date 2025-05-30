# C++ Ranges: A Non-Expert's Primer

10 April 2025

Outside of highly technical and exhaustive *cppreference.com* entries, written
material on C++ ranges (`<ranges>`) is sparse. I'm going to record some thoughts
and questions I have during my attempt to learn about C++ ranges, and I hope to
expand these notes in the future.

## Intro to Ranges

Firstly, **what is a range**? From what I've learned thus far, a range is an
abstraction over iterators. Ranges are pairs of iterators, *begin* and
*sentinel*, which characterize where the range begins and ends, respectively.
Like iterators (since ranges are built on top of them), ranges work as
half-intervals `[begin, sentinel)` such that the range stops at—but does not
include—the data associated with `sentinel` among the elements to be processed.
Unlike iterators, the two constituent iterators in a range need not have the
same type; they only need to be comparable. Also, based on talks given at C++
conferences, STL <u>are</u> ranges. I thought it noteworthy that they <u>are</u>
views rather than "having" or "producing" them. 

Along that train-of-thought, containers being ranges enables more succinct
syntax with range-based algorithm implementations (see, among others, the
[constrained
algorithms](https://en.cppreference.com/mwiki/index.php?title=cpp/algorithm/ranges&oldid=178068)
page on  *cppreference.com*). Take a look at the following:

```cpp
std::vector<int> arr {1, 0, 3, 4, 5, 2};
// previous, iterator-based algorithm interface
// std::sort(arr.begin(), arr.end());

// range-based sort
std::ranges::sort(arr);  // note we just pass the object in!
```

## What is a View?

A large majority of the utility I've seen from the ranges library is found in
*views*, types accessible through the `std::ranges::views` namespace (aliased to
`std::views` in the library). These are cheaply copyable, typically non-owning
ranges. By "cheaply", I mean that one must be able to copy, delete, and/or move
in constant time `O(1)`. This property is not guaranteed for owning-ranges (ex.
creating an owning vector requires that you copy the entire container, an `O(n)`
operation). 

The main selling point around views is that they are **composable** and
**lazy**. Multiple operations can be chained to create a resulting view via
*range adaptors*, functions that produce views. The resulting view is not
iterated over until the user requests it to be. This laziness is important for
composition. Multiple (potentially infinite) views can be chained together
without incurring the costs of evaluation up front. This is the complete
opposite behavior of that seen in the base STL algorithms which are evaluated
immediately, rendering consecutive operations potentially expensive.

Chaining range adaptors is simple, especially for those familiar with the
Unix-style command line: we use "pipes" `|`. For example, the following snippet
produces the squares that are odd:

```cpp
#include <iostream>
#include <ranges>
#include <vector>


int main()
{
    std::vector<int> v { 0, 1, 2, 3, 4, 5 };
    auto square = [](int x) -> int { return x*x; };
    auto isOdd = [](int x) -> bool { return !(x % 2 == 0); };
    auto oddSquared = v
            | std::views::transform(square)    // note the pipes!
            | std::views::filter(isOdd);
    for (const int x : oddSquared) {
        std::cout << x << " "; 
    }
	std::cout << "\n";      // 1, 9, 25
	
	return 0;
}
```

Pretty nice!  One thing to note is that, due to internal caching, ranges cannot
be passed by `const&` (see Jeff Garland's talk linked at the bottom of this
article ~27:00 in).

## What are some useful range adaptors?

I'm by no means an experienced user of ranges, but here are some useful functions and adaptors I've seen and/or used. Note that some of these are introduced in C++23:

1. `views::filter`: a [`view`](https://en.cppreference.com/w/cpp/ranges/view
   "cpp/ranges/view") that   consists of the elements of a
   [`range`](https://en.cppreference.com/w/cpp/ranges/range "cpp/ranges/range")
   that satisfies a predicate
2. `views::transform`: a [`view`](https://en.cppreference.com/w/cpp/ranges/view
   "cpp/ranges/view") of a   sequence that applies a transformation function to
   each element
3. `views::split`: a [`view`](https://en.cppreference.com/w/cpp/ranges/view
   "cpp/ranges/view") over the   subranges obtained from splitting another
   [`view`](https://en.cppreference.com/w/cpp/ranges/view "cpp/ranges/view")
   using a delimiter
4. `views::{all, iota}`  
5. `views::take`  
6. `views::cartesian_product`  
7. `views::reverse`  
8. `views::zip`   
9. `views::{drop, drop_while}`  

Here are some useful range-based algorithms:
1. `ranges::to`
2. `ranges::sort`
3. `ranges::reverse`
4. `ranges::fold_left`
5. `ranges::{max, min, min_element, max_element, ...}`
6. `ranges::{find, for_each}`
7. `ranges::[all|any|none]_of`

There's a lot more to cover such as concepts, custom range implementation,
projections, and more, but I'm not there yet, especially with generic
programming. Maybe in the future.

## Interesting Uses
### Range-based Iteration over Containers Simultaneously

With a combination of [structured
binding](https://en.cppreference.com/w/cpp/language/structured_binding) (C++17)
and
[`std::ranges::views::zip`](https://en.cppreference.com/w/cpp/ranges/zip_view)
(C++23), we can iterate over multiple containers using range-based for loops
(RBFL). The `zip` view creates a container in which the *i<sup>th</sup>* element
is a tuple of the *i<sup>th</sup>* element of each constituent container. Its
number of elements is the minimum of the number of elements across all
constituent containers. Here is a modified example from CppReference:

```cpp
// https://godbolt.org/z/Pf66PcM1W
#include <array>
#include <iostream>
#include <list>
#include <ranges>
#include <string>
#include <tuple>
#include <vector>
 
void print(auto const rem, auto const& range)
{
    for (std::cout << rem; auto const& elem : range)
        std::cout << elem << ' ';
    std::cout << '\n';
}
 
int main()
{
    auto x = std::vector{1, 2, 3, 4};
    auto y = std::list<std::string>{"α", "β", "γ", "δ", "ε"};
    auto z = std::array{'A', 'B', 'C', 'D', 'E', 'F'};
 
    print("Source views:", "");
    print("x: ", x);
    print("y: ", y);
    print("z: ", z);
 
    print("\nzip(x,y,z):", "");

    // Structured Binding with Range-based for loops.
    for (const auto& [num, str, chr] : std::views::zip(x, y, z))
    {
        std::cout << num << ' '
                  << str << ' '
                  << chr << '\n'; 
    }
}

/*
Output
------
Source views: 
x: 1 2 3 4 
y: α β γ δ ε 
z: A B C D E F 

zip(x,y,z): 
1 α A
2 β B
3 γ C
4 δ D
*?
```

## For more...

- Watch Jeff Garland's [*Effective Ranges: A Tutorial for Using C++2x
  Ranges*](https://www.youtube.com/watch?v=QoaVRQvA6hI) talk. It's the most
  approachable resource I've come across for ranges thus far.
- Read:
	- Microsoft's material on [Ranges](https://learn.microsoft.com/en-us/cpp/standard-library/ranges?view=msvc-170) and  [Range Adaptors](https://learn.microsoft.com/en-us/cpp/standard-library/range-adaptors?view=msvc-170)
	- Hannes Hauswedell's [*A beginner's guide to C++ Ranges and Views.*](https://hannes.hauswedell.net/post/2019/11/30/range_intro/) - *cppreference.com*
		- [Ranges library](https://en.cppreference.com/mwiki/index.php?title=cpp/ranges&oldid=180956)
		- [Standard library header `<ranges>`](https://en.cppreference.com/w/cpp/header/ranges) 
		- [Constrained algorithms](https://en.cppreference.com/mwiki/index.php?title=cpp/algorithm/ranges&oldid=178068)
	