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
   - `someActionPointer->setShortcut(QKeySequence const&)`
   - We can set our own custome kwyboard commands, but try to find a fit from among [`QKeySequence::StandardKey`](https://doc.qt.io/qt-6/qkeysequence.html#StandardKey-enum)
3. Set the Status tip.
4. Connect the action's `triggered()` signal to some desired slot.
5. Add the action to any desired menu and/or toolbar.

We instantiate a `QAction` object with its parent being
the containing window (ex. the main window). Then, set the action's icon, shortcut key, accelerator
(the combination typ. triggered with `ALT+<Key>` where <Key> is the character prefixed by `&` in a
QString), and status tip using the respecive methods. Here's an example from the book (
Chapter 3 of Blanchette and Summerfield's *C++ GUI Programming with Qt 4*):

```cpp
newAction = new QAction(tr("&New"), this);
newAction->setIcon(QIcon(":/images/new.png"));
newAction->setShortcut(QKeySequence::New);
newAction->setStatusTip(tr("Create a new spreadsheet file"));
connect(newAction, &QAction::triggered, this, &MainWindow::newFile);  // connects the action with some functionality.
```

## Togglable Actions

We can set actions to be togglable on instantiation. This communicates that the action is in either
the on or off state (think: word processing software with "Bold" toggle). To declare an action as
togglable, set `actionPtr->setCheckable(true)`. 

Qt automatically handles the state switches upon user interaction. It is not necessary for the
developer to manually switch the state via `QAction::setChecked(bool)`. 