# Interesting Tidbits

Here are some things I've found noteworthy while learning.

## Rounding to Nearest Size Boundary

I originally saw this while studying Bloomberg's BSL library. In `bsl/bslstl/bsl_string.h`, there's
the following snippet that's similar to this:

```cpp
SIZE = ((20 + sizeof(TYPE) - 1) & ~(sizeof(TYPE) - 1));
```
The comment on that line (ln. 1029-1032), mentions that the code rounds the buffer to a word
boundary. I didn't know why it worked, so I did some research.
[This StackOverflow comment explains it for a similar code segment](https://stackoverflow.com/a/8178170):

>  `#define _INTSIZE(n) ((sizeof(n) + sizeof(int) - 1) & ~(sizeof(int) - 1))`
>
>> The code first adds three to the number.
>>
>> Then it zeroes out the last two bits to round down to a multiple of fours. Just like you can
>> round down to the nearest multiple of 100 in decimal by replacing the last two digits with
>> zeroes.
>> 
>> If the number is already a multiple of four, adding three to it and then rounding down to the
>> nearest multiple of four leaves it alone, as desired. If the number is 1, 2, or 3 more than a
>> multiple of 4, adding 3 to it raises it above the next multiple of 4, which it then rounds down
>> to, exactly as desired.

So, in other words, the act of adding `sizeof(TYPE) - 1` and then masking the value with the binary
inverse of `sizeof(TYPE) - 1` rounds the value to the nearest multiple of `sizeof(TYPE)`. 

The interesting takeaway for me was the use of bit operations to produce such a result.

---

## Converting Recursion to Iteration

Many different sources I've come across state that *any* recursion can represented as iteration. 
The following is an example of doing so when we have multiple recursive calls in the function body.

### Problem

> Represent postorder tree traversal using a stack implementation.

The recursive implementation is pretty trivial (pseudo-code):

```python
# Recursive Implementation
def traverse(node):
    if (node.left)
        traverse(node.left)
    if (node.right)
        traverse(node.right)
    # process the "root"
    visit(node)
```

Implementing an iterative approach, took more thought.
This was tough to think about because I didn't know how to achieve it with oen stack, come back up
the tree to handle the right subtrees, nor did I know how to prevent an infinite loop when pushing
nodes on the stack based on if they had valid children (nodes already visited would be added again).
Eventually, I was able to arrive at an answer. The thing about recursion is that, when you make a
recursive call, a new stack frame is allocated. This means that the state of the current frame has
to be stored somewhere such that, upon return, we can continue execution. In other words, to
successfully solve this problem using a stack implementation, I need to store both relevant
parameters *and* the execution state. I did this with two stacks (This will be pseudocode-like):

```python
def traverse(node):
    """Iterative implementation of postorder traversal."""
    enum States { LEFT, RIGHT, ROOT }
    
    node_stack = []
    state_stack = []
    node_stack.push(node)
    state_stack.push(States.LEFT)
    
    while (not node_stack.isEmpty() and not state_stack.isEmpty()):
        state = state_stack.pop()
        curr = node_stack.pop()

        switch(state):
            case States.LEFT:
                - push 'curr' onto node_stack
                - advance the state of current node
                    - state_stack.push(States.RIGHT)
                - left child exists?
                    - push left child onto stack
                    - push States.LEFT onto state_stack
                break
            case States.RIGHT:
                - push 'curr' onto node_stack
                - push States.ROOT onto state_stack (advances the state of curr)
                - right child exists?
                    - push right child onto node_stack
                    - push States.LEFT onto state_stack 
                break
            case States.ROOT:
                # Process the root
                visit(curr)
                break
    # end
```

We see that the state tracking is used to determine which path of the execution should be taken.
Each branch advances the state of the current node and initializes the state of any child node
found.


### Further Reading

- [More general recursion to iteration](https://stackoverflow.com/a/8512072)
- [Recursion to Iteration
  Series](https://blog.moertel.com/tags/recursion-to-iteration%20series.html)
- [From Recursive to Iterative Functions](https://www.baeldung.com/cs/convert-recursion-to-iteration)

## Range-based Iteration over Containers Simultaneously

With a combination of [structured binding](https://en.cppreference.com/w/cpp/language/structured_binding) (C++17) and [`std::ranges::views::zip`](https://en.cppreference.com/w/cpp/ranges/zip_view) (C++23), we can iterate over multiple containers using range-based for loops (RBFL). The `zip` view creates a container in which the *ith* element is a tuple of the *ith* element of each constituent containeer. Its number of elements is the minimum of the number of elements across all constituent containers. Here is a modified example from CppReference:

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
