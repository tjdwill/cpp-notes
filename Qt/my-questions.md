# Questions About Qt

While certainly informative, reading doesn't quite answer every question, so in this
document I record questions that arise during practical use of the Qt framework.

0. If creating a custom subclass of some Qt widget, should I make all constituent pointers
   data members? Sometimes, the objects don't need further programmatic interaction after
   being instantiated. Doing so would also increase the class size. On the other hand,
   *not* doing so would mean there can be many more pointers and "phantom" objects used
   within a widget than accounted for in the class definition. If the design of the given
   class changes, it may be difficult to track down stray objects. What to do?

1. When communicating between widgets across the parent-child hierarchy (ex. child <->
   grandparent), is it better to connect the two directly, or should signals be
   propagated—chaining signal calls up and down each direct parent or child until it
   reaches the destination? The former can lead to really disorganized dependency graphs,
   while the latter could introduce extra clutter code.
