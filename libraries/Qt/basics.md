# Qt Basics 

Version: 5.15.2 
Module(s): Widgets, Core, Gui

The basics of a Qt program are simple. First, Qt is its own entity. Rather than
considering it a library for C++, it's more of an extension to the language, having its own
meta-object compiler and implementing its own types and keywords.

At the root of any application is the `QApplication` object. For any Qt GUI application,
there is exactly one `QApplication` object. This class is specifically for the `QtWidgets`
library, so it assumes use of `QtWidgets` (or, I'm assuming, `QtQml`). The purpose of the
class is to handle widget initialization and finalization. It's the event listener of the
Qt application. Additionally, it:

- Initializes the application
- Parses the command line arguments
- Defines the app's look (this is the "native look and feel" part)
- Manages cursor handling

This object *must* be created before any other user interface object or argument parsing.

## Widgets

For Qt, every element of the user interface is an individual widget. Widgets all inherit
from the `QWidget` class which itself inherits from the `QObject` class. Each `QObject` has
a parent and can have multiple children. A given parent is a pointer to a `QObject`. Qt
handles object deletion on its own; we don't need to do it ourselves. Meaning, if you
consider a given parent the root of an object tree, deleting the parent will delete the
entire tree. If a branch or inner-node of the tree is deleted *before* the parent, it is
unregistered from the parent, preventing double-frees.

Any widget that is *not* embedded in a parent is called a **window**.

### Widget Communication: Signals and Slots

For interactive, responsive widgets, they need to be able to exchange information. Qt does
this through the concepts of **signals** and **slots**. Essentially, an object can send a
signal when some event happens. It can pass information to whatever slots receive the
signal. Slots, in turn, are functions (i.e. class methods) that run in response to some
signal.

There is essentially no limit to the number of signals a slot can associate with.
Similarly, there's no limit to the number of slots a given signal associates with. Signals
can also connect with one another. The second signal will fire immediately following the
first.

Signals and slots are connected via the `QObject::connect` static method. Note that if two
widgets are connected bilaterally (signal ObjA -> slot ObjB; signal ObjB -> slot ObjA), Qt
prevents infinite recursion by checking if the widget is already set to the incoming value.


## The use of `new` in Qt

So, when I saw that example code kept instantiating pointers to `QWidget`s instead of
instantiating them directly, I paused because manual `new` means manual `delete` in my
head. However, as long as the widget is connected to a parent, it will be handled once the
parent is deleted. This allows heap-allocated widgets to be used with ease-of-mind.
According to KDAB's Jesper K. Pedersen, you want to heap-allocate widget-related objects in
order to have them live longer than the current stack frame. Value-related objects such as
`QString`s or `QColor`s, however, do not need to be heap allocated because they are
primarily used for configuration via function calls (I'm paraphrasing and extrapolating
here).

To summarize, if the object in question has a parent, you're allocating it with `new`. 

## Layouts

*Layout managers* are entities in charge of managing the positions, sizes, and spacings of
widgets under their jurisdiction. The ability to nest these managers within one another
enables the creation of complex dialog widget design.  There are multiple layout manager
classes in Qt. Among others, we have:

0. `QHBoxLayout`: lays out widgets horizontally from left to right by default
1. `QVBoxLayout`: lays out widgets vertically from top to bottom by default
2. `QGridLayout`: lays widgets on a grid
3. `QFormLayout`: two-column layout (labels : field\_widgets)
4. `QStackedLayout`: page-based layout

- Widget addition order matters; it affects where it appears in the sequence.
- Regarding resource management, widgets are re-parented once the layout manager is
  installed to a widget. 
- Layout managers *are not* widgets themselves; they don't derive from `QWidget`
    - They are invisible as a result.

### Installing a Layout

Once you've created the main layout, install it to the desired widget by passing the layout
pointer to the `QWidget::setLayout()` function (all derived classes have this method.)

```cpp
// ...    
QLabel* label { new QLabel(tr("Type &Here:")) };
QLineEdit* lineEdit { new QLineEdit }; 
label->setBuddy(lineEdit); 

QHBoxLayout* line { new QHBoxLayout };
line->addWidget(label);
line->addWidget(lineEdit);

QWidget* myWidget { new QWidget };
myWidget->setLayout(line);
// ...
```

## Dialogs

*Dialog* windows are used to communicate with the application user. They can present
information to the user as well as retrieve information via data input.
The base class for dialogs is [`QDialog`](https://doc.qt.io/qt-5/qdialog.html). Study the
common slots, signals, and methods.

While one can build dialogs from scratch using Qt's C++ API, the recommended method is to
design the dialog visually via QtDesigner, subsequently generating the class from the
design file (`.ui` extension). The typical pattern is:

1. Design widget (let's call it `Foo`) with QtDesigner
2. Add custom functionality to the class by creating a subclass of `Ui::Foo` (generated
   classes are under the `Ui` namespace) and `QDialog`.
    
    ```cpp
    // foo.h
    #include  "ui_foo.h"  // this is generated by UIC
    #include <QDialog>
    
    class Foo : public QDialog, public Ui::Foo
    {
        Q_OBJECT // never forget this.
        /* define slots, signals, methods, data members, etc. */
    public:
        Foo(QWidget* parent=nullptr);
    };

    // foo.cpp
    #include "foo.h"
    #include <QtWidgets>

    Foo::Foo(QWidget* parent)
        : QDialog(parent)
    {
        setupUi(this);
        /* other setup actions here */
    }
    ```

3. Use the implemented subclass. 

The approach displayed in item two is called the *Multiple Inheritance Approach*. However,
there are also the *Direct* and *Single Inheritance* approaches as well. [Read about all
three of the approaches here.](https://doc.qt.io/qt-5/designer-using-a-ui-file.html)


### Shape-changing Dialogs

Dialogs that expand or contract based on user interactivity. Here are the basics:

- Connect relevant widgets' visibility slots to a togglable button (see
  [QPushButton::setCheckable()](https://doc.qt.io/qt-6/qabstractbutton.html#checkable-prop))'s `toggled(bool)` signal
- Hide the widgets when instantiating the object (call `widget->hide()`).
- In the constructor, set the housing widget's layout size constraint to "fixed" (`layout()->setSizeConstraint(QLayout::Fixed)` in order to prevent user size
  manipulation (the application will be in charge of resizes for that widget).

# Actions

As part of making a menu, we use actions to give the menu items functionality. Actions are
items in Qt that are used to provide one single behavior for multiple invocation methods
(menu, toolbar, and/or keyboard shortcut). The basic action is represented by [the `QAction`
type](https://doc.qt.io/qt-5/qaction.html). The class allows the programmer to set many
different properties: icon, menu text, shortcut combination, status text, "What's This?"
text, and tooltip text. 

The basic process is as follows:

0. Instantiate the `QAction*` via `new` (in true Qt style)
1. Set the icon via pathing (hint: it's helpful to leverage `rcc` to embed the image into the
   application)
2. Set the shortcut
3. set the Status tip.
4. Connect the action's `triggered()` signal to some desired slot.
5. Add the action to any desired menu and/or toolbar.
 Some actions are checkable meaning they are togglable and have checkmarks indicating their
 state.
