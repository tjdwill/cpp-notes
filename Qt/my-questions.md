# Questions About Qt

While certainly informative, reading doesn't quite answer every question, so in this
document I record questions that arise during practical use of the Qt framework.

**Q:** If creating a custom subclass of some Qt widget, should I make all constituent
pointers data members? Sometimes, the objects don't need further programmatic interaction
after being instantiated. Doing so would also increase the class size. On the other hand,
*not* doing so would mean there can be many more pointers and "phantom" objects used within
a widget than accounted for in the class definition. If the design of the given class
changes, it may be difficult to track down stray objects. What to do?

**A:** No, you don't need to make *every* single widget a data member of the class. For
one, this can become unwieldy. Secondly, if you confine the creation of component widgets
to method calls (ex. if making a navigation bar, have some method called `createNavBar()`),
it should be fairly simple to find where changes should be made.

**Q:** When communicating between widgets across the multiple levels of the parent-child
hierarchy (ex. child <-> grandparent), is it better to connect the two directly, or should
signals be propagated—chaining signal calls up and down each direct parent or child until
it reaches the destination? The former can lead to really disorganized dependency graphs,
while the latter could introduce extra clutter code.

**A:** If those are the only two options, I think the propagation would end up being
cleaner. However, it may be worthwhile to instead expose the pointer to said object through
the intermediate parent (ex. how buttons are exposed in a `QDialogButtonBox` via the
`button` method). This way, we can directly connect to the signal in question without
having to introduce multiple levels of propagation. This also ends up encouraging a more
flexible design (imagine if the lowest signal's type signature changes when using
propagation).

**Q: Should function-scoped widgets be created on the stack or the heap (via `new`)?**

**A:** After attempting to do this, especially when the widget itself has a variable number of
components based on a boolean, allocating on the heap is easier. If this temporary
widget is to be created and used within a custom QtWidget subclass:

- Create the widget with `this` as its parent.
- Add all components and connections as needed using pointer-based syntax (normal Qt
 style).
- At the end of all operations, set the created object to be deleted via
 `createdObject->deleteLater()`.

This way, if I somehow forget to delete the object at the end of the function, it's
still deleted later because it is set as a child of the enclosing custom class instance.
Bottomline, the heap-based Qt style seems easier and more consistent.

**Q: If creating an app that utilizes multiple constituent custom widgets, should the
sub-widgets implement app logic?"

**A:** Personally, I think the answer is "no." After trying this the first time, I realized
that I wouldn't have a way to coordinate actions implemented in the main widget. Also, if
the lower widgets implement app logic, said logic will be connected via signals and slots
in the lower widget, which, again may be undesirable for `QAction` coordination.

Instead, I've found that lower-level widgets should basically be non-functional; the focus
should be on the structure. An exception may be in page-based widgets (ex. `QStackedWidget`,
`QTabWidget`) where the
page-switching logic can be implemented within the defining class. These classes should
expose important components such as push buttons in the `public` interface in order to
allow higher-level objects to connect to relevant signals and slots. If the idea of making
data members public is uncomfortable (and it is), remember that Qt Designer-generated
classes make all data members `public`. I am proposing the same thing for classes I write
myself. Do Qt things the Qt way.
