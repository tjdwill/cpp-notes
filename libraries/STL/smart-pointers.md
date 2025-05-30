# On Smart Pointers

Raw pointers, while useful, present safety issues because it is possible to
access the memory address whose content is no longer valid (a.k.a *dangling
pointer*). We have no way of obtaining lifetime information about the object to
which the pointer refers (read about Rust's borrow checker for more
information). We also have the issue of ambiguous resource ownership. Another
way of thinking about ownership is "responsibility": who is reponsible for
managing the resource? Deleting it? With raw pointers, the answers to these
questions can be unclear. If more than one entity assumes responsiblility and
frees the resource, we have the potential for *double frees*. It takes careful,
meticulous attention-to-detail to ensure proper interaction with heap-allocated
objects.

The C++ Standard Library provides tools that make management much more explicit
and safe. These *smart pointers*, pointers that have awareness of ownership,
greatly enhances developer experience with memory management. These notes will
discuss the following smart pointers: `std::unique_ptr`, `std::shared_ptr`, and
`std::weak_ptr`. Introduced in C++11, all of these smart pointers are defined in
the [`<memory>`](https://en.cppreference.com/w/cpp/header/memory) header.

## [`std::unique_ptr`](https://en.cppreference.com/w/cpp/memory/unique_ptr)

The unique pointer, represented by `std::unique_ptr`, is a smart pointer that
semantically denotes sole ownership of a resource. Once constructed, the
developer is communicated there is only *one* owner of the resource, and that
owner is responsible for any cleanup and deletion behaviors. Indeed, the
`unique_ptr` is able to automatically delete the resource upon the pointer going
out of scope. 

If desired, the user can provide a custom deleter function during unique_ptr
construction that is called instead of the default behavior. This is useful in
cases where a file resource must be closed for example. Conventionally,
`unique_ptr`s are [recommended by
default](https://herbsutter.com/2013/05/29/gotw-89-solution-smart-pointers/) due
to minimized overhead. 


## [`std::shared_ptr`](https://en.cppreference.com/w/cpp/memory/shared_ptr)

Shared pointers represent shared ownership. Multiple pointers can own the same
object, and the object is not deleted until the program is certain there are no
owners left (*i.e.* that the last owner is being destroyed). Shared ownership is
implemented via reference counting, meaning an additional shared_ptr for an
object increases the ref count by 1 and the destruction of such a pointer
decrements the count. Like `unique_ptr`, `shared_ptr` allows for the
specification of a custom deleter.


### `std::make_shared` and `std::make_unique`

The STL provides factory functions for creating the respective pointers. Unless
there are extenuating circumstances like the need for a custom deleter or the
adoption of a raw pointer from an external source (i.e. you didn't create it),
is strongly recommended to use these functions instead of the classes'
constructors. For one, they are designed to prevent memory leaks, so if an
exception occurs before the destruction of the pointer, the resource is destoyed
instead of leaked. Secondly, obviate us of the need to use `new` and `delete`,
so those keywords can now be identified as "code-smells" or causes for concern
in a codebase. Finally, in the case of `shared_ptr`, `std::make_shared` is more
efficient because only one allocation is performed instead of two when using the
class' constructor. 

The tradeoff, however, is that the memory for the allocated object remains
unfreed until there are no remaining shared or weak pointers to it. In the
manual construction case, only the shared pointers affect when the memory is
freed.

## [`std::weak_ptr`](https://en.cppreference.com/w/cpp/memory/weak_ptr)

Weak pointers are *like* shared pointers except they don't contribute to the
object's ref count. Weak pointers don't own the object they point to. Unlike
shared pointers which are strong references (they keep the object alive), weak
pointers do not keep the object alive. 

### Important Features

- Query if the object is still alive (by querying ref count or using the
  specialized utility method)
- Ensure the object remains alive by generating a shared_ptr (via
  `weak_ptr.lock()`).

## Caution Points

- Don't use arrays `[]` in shared pointers because the deleter does not call
  `delete[]`.
- Don't manually construct two or more `shared_ptr` via
  `shared_ptr(some_raw_pointer)` because each will think it has sole ownership.
  If you want to make more shared_ptrs from an object, pass in the owning
  shared_ptr:

	```cpp int* a = new int; shared_ptr<int> sptr0 {a}; // shared_ptr<int> sptr1
	{a};  // BAD shared_ptr<int> sptr1 {sptr0}; // OK ```
- Watch out for circular references in which two objects each have
  shared_pointers of the other. 
    - The ref count will never be 0, so the objects are never destroyed.
    - Use a `std::weak_ptr` for one of them.
- If using a weak pointer for some logic, use `lock()` when you wish to interact
  with the object.
    - Ensures the object isn't invalidated underneath you.

## References/Read More

- Nicolai Josuttis: *The C++ Standard Library (2<sup>nd</sup> ed.)* Ch. 5.3 "Smart Pointers"
- Herb Sutter: [*GotW #89 Solution: Smart Pointers*](https://herbsutter.com/2013/05/29/gotw-89-solution-smart-pointers/)
- cppreference: [Memory Management Library](https://en.cppreference.com/w/cpp/memory)
- StackOverflow: [`make_shared` vs. `shared_ptr`](https://stackoverflow.com/a/20895705)
- Microsoft Learn: [STL Lectures](https://learn.microsoft.com/en-us/shows/c9-lectures-stephan-t-lavavej-standard-template-library-stl-/3-of-n)
    - Start at 32:15
- The Cherno: ["SMART POINTERS in C++ (std::unique_ptr, std::shared_ptr, std::weak_ptr)"](https://www.youtube.com/watch?v=UOB7-B2MfwA)
- The Cherno: ["Weak Pointers in C++ (std::weak_ptr)"](https://www.youtube.com/watch?v=M0GLQEfplxs)