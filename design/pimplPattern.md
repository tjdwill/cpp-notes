# The Pointer to Implementation (pImpl) Pattern

Recently, I came across this design pattern that I want to write notes about. The *Pointer to
Implementation*, or pImpl pattern is known by many names, including the "Cheshire Cat", "Opaque
Pointer", and, for those familiar with the Gang of Four book on design patterns, the "Bridge
Pattern". As a quick 411, encapsulation, data hiding, and speedy build times are all ideal sin the
C++ world, and the pImpl pattern provides a mechanism by which all three can be further achieved. 

As I understand it, pImpl is a pattern in which the implementation details of a class are hidden,
accessible only behind a privately-held pointer (hence, *opaque*. We don't "know" size information
about the thing behind the pointer without looking at its definition). Essentially, for a given
class `Foo` , we split it into a public interface and a private implementation that holds `Foo`'s
data members:

```cpp
// some header file
class Foo
{
public:
	// define public interface
	~Foo(); // must only *declare* the destructor
private:
	class Private; // stores implementation-specific data
	std::unique_ptr<Private> dImpl;
}
```

We declare an incomplete class, `Foo::Private`, to serve as our private implementation. We then
declare a unique pointer data member that owns the private implementation. Critically, `Foo`'s
destructor is *declared* (but not defined) in the header file, to then be defined in the `.cpp`
file. This sequence is done because `Foo::Private` is forward declared; the unique pointer doesn't
have enough information regarding how to delete a `Private` object, so the destructor can't be
defaulted or defined within the header. Compilation errors abound if one attempts to do so. As a
final note on this topic, note that because we define the destructor, we must also handle all of the
other special member functions in accordance with the Rule of Five (Six if we include the default
constructor).

So, great. What do we get from this? Why would we take the time to implement a class this way? Well,
now we can change the implementation of class without consumers needing to recompile all components
that use that class. If we were to change the definition within the header file, any file that
includes the class's header must recompile accordingly. By hiding the data members behind a pointer,
we can change how to class is represented as often as we want, only needing to recompile and relink
the implementation file. In other words, this helps compile times. 

Also, though I've yet to reach the level where this is a concern, this pattern helps keep a stable
application binary interface (ABI). Basically, because the class's sole data member is a pointer,
its size and alignment remain stable and unchanging. Therefore, barring any additional changes to
the header, changes to the Private implementation **won't** change the public class's size. If
interested about ABI, the Qt framework has a great article about their use of the pImpl pattern [at
the following link](https://wiki.qt.io/D-Pointer).

## Implementation Questions

So, while it's nice to know what pImpl is, I found that I had a lot of questions when programming it
myself. Here's what I learned in the process:


### What methods should be implemented in the private implementation class?

It depends. [CppReference claims private methods should go in
`Private`](https://en.cppreference.com/w/cpp/language/pimpl), which is a good rule-of-thumb.
However, there are cases, such as in legacy code, where private methods rely on (call) protected
functions. In this case, if the private methods were moved to `Private`, `Private` would require a
pointer to the public class that owns it to invoke these functions. Now, this isn't *necessarily* a
problem; Qt makes extensive use of exactly this (see the above link). However, some people may be
hesitant to introduce that cyclic relationship between the two classes.

Instead, we can simply keep the private methods that use public-class-specific information (like
protected functions) on the public class itself, implementing them with dereferences of the `dImpl`
pointer to access data members. Ultimately, it doesn't matter what goes where when considering the
desired compilation benefit. However, in terms of both maintenance and design, placement makes a
difference. As a personal anecdote, placement affected whether or not I could default special member
functions in one case. 

### Should subclasses have access to the base class's dImpl?

No. The pImpl pattern is an implementation detail, so consumers should not have access or even
knowledge about it. Put another way, `Private` should implement functionality specific to the base
class. As a result, the subclasses are then free to have their own `SomeDerived::Private` classes
without overriding the base class's version.

### Why not make `Private` a `friend`?

1. The classes are now truly coupled (would still need a pointer to the base to access
   instance-specific methods).
2. Readers will have a more difficult time following along, partly because `friend` use tends to be
   discouraged. 
3. Semantically, it's less faithful to our intent. Declaring `Private` as `friend` means it is no
   longer under the public class's namespace. 

### Is the pImpl pointer the only data member?

Typically, yes. There may be special exceptions, but I can't think of any at the time of writing.

---

Ultimately, the way I think of the relationship between `Foo::Private` and `Foo`  is that it lies on
a spectrum. At one extreme, `Foo::Private` is simply a class that contains data members, and `Foo`
implements all of its methods using said data members. On the other side, `Foo::Private` implements
the actual logic of all of `Foo`'s behavior, and  `Foo` forwards all calls from its public interface
to call those private implementations. As always, it depends.

## More Reading

- [*Opaque Pointer Pattern in C++*](https://danielsieger.com/blog/2024/08/02/cpp-opaque-pointer-pattern.html) by Daniel Siegler
- [*How to implement the pimpl idiom by using unique_ptr*](https://www.fluentcpp.com/2017/09/22/make-pimpl-using-unique_ptr/) by Jonathan
  Boccara (Fluent C++)
