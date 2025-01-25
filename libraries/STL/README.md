# C++ Standarad Template Library (STL)

The Standard Library for C++ has a lot of functionality. It can be overwhelming to consider in its totality,
so I'm going to list some STL components to consider:


## Containers

- `std::vector`
  - Dynamically-sized container w/ contiguous elements and constant access time.   
- `std::array`
  - Fixed-size container w/ contiguous elements and constant access time. 
- `std::list`; `std::forward_list`
  - Doubly- and Singly- linked lists 
- `std::deque`; `std::queue`; `std::stack`
- `std::map`; `std:set` (and unordered versions)
  - Associative containers 
- `std::string`; `std::string_view`

## Algorithms

Research the `<algorithm>` component. 
Related: `<iterator>` and `<ranges>`. 

## Memory Management

- Smart Pointers
  - `std::unique_ptr`, `std::shared_ptr`, and `std::weak_ptr`

## Input/Output

- `<iostream>`
  - I/O for standard files (stdin, stdout, and stderr)
- `<fstream>`
  - File I/O
- `<sstream>`
  - String I/O (useful for building strings or parsing from a string)
- `<format>` (related to text, so I'm placing it here)

 ## Numeric Based

 - `<random>`
 - `<cmath>`
   - Basic math functions
  
## Date and Time

 - `<chrono>`

## General Utilities
 
 - `std::optional`, `std::expected`, `std::variant`
 - `std::tuple`, `std::pair`

## Multithreading 

- `std::jthread`
- `std::stop_source` and `std::stop_token`
- Synchronization primitives (mutexes, locks, latches, semaphores, etc.)

  Obviously, there is *much* more in the STL, but this seems to be a solid list for building broad familiarity.
