# Creating Menus and Toolbars in Qt

This is from Chapter 3 of Blanchette and Summerfield's *C++ GUI Programming with Qt 4*.


The basis of toolbars and menus are *actions*. The simple process is:

- Create and set up the actions.
- Create menus and populate with actions
- Create toolsbars and populate with actions


## Creating actions

Actions are created via the `QAction` class. We instantiate a `QAction` object with its parent being
the containing window (ex. the main window). Then, set the action's icon, shortcut key, accelerator
(the combination typ. triggered with `ALT+<Key>` where <Key> is the character prefixed by `&` in a
QString), and status tip using the respecive methods. Here's an example from the book:

```cpp
newAction = new QAction(tr("&New"), this);
newAction->setIcon(QIcon(":/images/new.png"));
newAction->setShortcut(QKeySequence::New);
newAction->setStatusTip(tr("Create a new spreadsheet file"));
connect(newAction, &QAction::triggered, this, &MainWindow::newFile);  // connects the action with some functionality.
```

Some actions are checkable, meaning instead of simply triggering a command, they toggle some option
or functionality. This is controlled by the `setCheckable` and `setChecked` methods of `QAction`.
