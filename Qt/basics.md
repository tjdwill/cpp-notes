# Qt Basics

Version: 5.15.2
Module(s): Widgets

The basics of a Qt program are simple. First, Qt is it's own entity. Rather than considering it a library for C++, it's more of an extension to the language, having its own meta-object compiler and implementing its own types and keywords.

At the root of any application is the `QApplication` object. For any Qt GUI application, there is exactly one `QApplication` object. This class is specifically for the `QtWidgets` library, so it assumes use of `QtWidgets` (or, I'm assuming, `QtQml`). The purpose of the class is to handle widget initialization and finalization. It's the event listener of the Qt application. Additionally, it:

- Initializes the application
- Parses the command line arguments
- Defines the apps look (this is the "native look and feel" part)
- Manages cursor handling

This object *must* be created before any other user interface object or argument parsing.

## Widgets

For Qt, every element of the user interface is an individual widget. Widgets all inherit from the `QWidget` class which itself inherits from the `QObject` class. Each `QObject` has a parent and can have multiple children. A given parent is a pointer to a `QObject`. Qt handles object deletion on its own; we don't need to do it ourselves. Meaning, if you consider a given parent the root of an object tree, deleting the parent will delete the entire tree. If a branch or inner-node of the tree is deleted *before* the parent, it is unregistered from the parent, preventing double-frees.

Any widget that is *not* embedded in a parent is called a **window**.

### Widget Communication: Signals and Slots

For interactive, responsive widgets, they need to be able to exchange information. Qt does this through the concepts of **signals** and **slots**. Essentially, an object can send a signal when some event happens. It can pass information to whatever slots receive the signal. Slots, in turn, are functions (i.e. class methods) that run in response to some signal.

There is essentially no limit to the number of signals a slot can associate with. Similarly, there's no limit to the number of slots a given signal associates with. Signals can also connect with one another. The second signal will fire immediately following the first.

Signals and slots are connected via the `QObject::connect` static method. Note that if two widgets are connected bilaterally (signal ObjA -> slot ObjB; signal ObjB -> slot ObjA), Qt prevents infinite recursion by checking if the widget is already set to the received value.


## The use of `new` in Qt

So, when I saw that example code kept instantiating pointers to `QWidget`s instead of instantiating them directly, I paused because manual `new` means manual `delete` in my head. However, as long as the widget is connected to a parent, it will be handled once the parent is deleted. This allows heap-allocated widgets to be used with ease-of-mind. 
According to KDAB's Jesper K. Pedersen, you want to heap-allocate widget-related objects in order to have them live longer than the current stack frame. Value-related objects such as `QString`s or `QColor`s, however, do not need to be heap allocated because they are primarily used for configuration via function calls (I'm paraphrasing and extrapolating here).

## Layouts

There are multiple layout manager classes in Qt. Among others, we have:

0. `QHBoxLayout`: lays out widgets horizontally from left to right
1. `QVBoxLayout`: lays out widgets vertically from top to bottom
2. `QGridLayout`: lays widgets on a grid
3. `QFormLayout`: two-column layout (labels : field\_widgets)
4. `QStackedLayout`: page-based layout

- Widget addition order matters; it affects where it appears in the sequence.
