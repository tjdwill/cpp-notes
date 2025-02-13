# Sorting Algorithms

According to Robert Sedgewick of *Algorithms in C* (3<sup>rd</sup> edition), we want to "sort
*files* of *items* containing *keys*. The goal is to rearrange the items such that the keys are
sorted according to some well-defined ordering criteria. In other words, to sort a collection of
items, we sort the keys associated with said items. Based on the key type, we can even sort
disparate items (think back to the section on addition in Chrystal's *Algebra*).

*Internal Sorting*
: A sorting method whose file(s) of items can fit into memory

*External Sorting*
: A sorting method whose file(s) must be sorted from an external store (disk or tape)o

*Nonadaptive Sort*
: The sequence of operations performed does not depend on the order of the data.

*Adaptive Sort*
: Operation sequences are determined on the outcomes of data comparisons

*Stable*
: A sorting method thta preserves the relative order of itesm with duplicated keys

*Indirect Sort*
: Rather than move objects around in memory, shuffle pointers (or indices) to said items

The elementary sorting algorithms (selection-, insertion-, and bubble-sort) are all `O(N^2)` in
terms of time complexity, but Sedgewick professes these methods may be more sufficient to the more
complex algorithms when considering a small sort space (< 100 from what I've surmised). Complex
algorithms may have large overhead that make them infeasible for small cases.

**Performance Characteristics**

- Time complexity - Based on number of comparisons and exchanges
- Space complexity - How much extra spacee needed (3 types)
    - Little to no extra space: in-place sorting
    - Linked list/pointer/array-index representation: extra memory for the extra bookkeeping
    - Holds copy of entire array

## Selection Sort

Basic idea:

    0. Find smallest unsorted element in the search space.
    1. Exchange it with the element in the first position of the unsorted space.
    2. Decrease the unsorted space by one.

Cons:

    - Run time depends very little on initial sorting order of the items.
        - Non-adaptive sorT?

Pros:

    - Great for sorting files with huge items and small keys (Sedgewick says it's the method of choice for this use case)

## Insertion Sort

Assume that for a given item `arr[i]`, everything to the left of `arr[i]` is sorted. Find the
elements within `0 <= j, k <= i-1` such that `arr[j] < arr[i] < arr[k]` and `k = j+1`. Insert
`arr[i]` in between `arr[j], arr[k]` moving everything after to the right if needed.


# TBD