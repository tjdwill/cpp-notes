# Creating Menus and Toolbars in Qt

This is from Chapter 3 of Blanchette and Summerfield's *C++ GUI Programming with
Qt 4*.


The basis of toolbars and menus are *actions*. The simple process is:

- Create and set up the actions.
- Create menus and populate with actions
- Create toolsbars and populate with actions


## Creating actions

See [the notes on QActions](./actions.md).

## Creating Menus

Menus come in two forms (semantically): drop-down and context. Both serve the
same function in that they provide a list of commands/actions, but they are used
differently. Drop-down menus are `QMenu`s that "live" in a `QMenuBar`. The menu
bar positions itself at the top of its parent widget, allowing the developer to
provide the functinoality commonly seen in many applications (File Menu, Help,
Options, etc.).

Context Menus are the menus we see when right-clicking in the application. The
menu items are *context dependent* (hence the name) whereas drop-downs are
typically more "global". 

**NOTE**: To be clear, context menus and drop-down menus are both the `QMenu`s.
They are the same object type used in different use-cases. The major differences
between the two, to me, are:

- Context menus are typically placed at the right-click location (the cursor).
- Context menus are context-dependent (the listed actions may vary)
- Drop-down menus have a static location: the menu bar.
- Drop-down menu items tend to be static in terms of presence within the menu.

### Context Menu

Though I've yet to create one, it would appear that context menus are as simple
as creating and populating the `QMenu` as usual followed by calling
[`QMenu::popup()`](https://doc.qt.io/qt-6/qmenu.html#popup) or
[`QMenu::exec()`](https://doc.qt.io/qt-6/qmenu.html#exec-1) with the desired
position point (and possibly the action you would like to appear at that point.
See the docs).

1. Create menu `new QMenu`
2. Give it actions via `QMenu::addAction()`
3. Call `QMenu::exec()` (for synchronous execution) or `QMenu::popup()` (for
   asynchronous execution)
    - Position: for the current mouse position, use the `QCursor::pos()` static
      method.
    - Note that position is always specified w.r.t the global reference frame
      (global screen coordinates; primary screen). If you want to set the
      context menu's location to be aligned relative to a widget, use
      [`QWidget::maptoGlobal()`](https://doc.qt.io/qt-6/qwidget.html#mapToGlobal)

### QMenuBar

Typically, we interact with the `QMenuBar` in the context of a `QMainWindow`.
Just call the main window's `menuBar()` method to receive the handle to the
`QMenuBar` object. Add menus and separators as desired.

However, we can also create menu bars ourselves. It is improtant to note that we
don't need to set the layout or position of the menu bar manually; the menu bar
is automatically moved to the top of its parent widget. Study the following
example program I wrote:

```cpp
// qmenubar.cpp
// A program to practice manually adding a QMenuBar to a widget.
#include <QtWidgets>
#include <QApplication>

int main(int argc, char* argv[])
{
    QApplication app { argc, argv };
    
    QWidget* window { new QWidget() };

    // Setting the `window` widget as parent will cause the
    // QMenuBar to automatically set itself at the top of
    // the widget.
    //
    // Note that this program *could* have been written such
    // that the menubar is the top-level widget (no
    // parent). QMenuBar inherits from QWidget, so it can
    // be a window.
    QMenuBar* menubar { new QMenuBar(window) };
    QMenu* fileMenu { new QMenu(QObject::tr("&File")) };
    
    QAction* closeAction { new QAction(QObject::tr("&Exit")) };
    closeAction->setShortcut(QKeySequence::Quit);
    QObject::connect(
            closeAction, &QAction::triggered,
            window, &QWidget::close
    );
    // Add the action to the menu and the menu to the
    // menubar. 
    fileMenu->addAction(closeAction);
    menubar->addMenu(fileMenu);

    window->setMinimumSize(QSize(640, 480));
    window->setWindowTitle(QObject::tr("QMenu Practice"));
    window->show();

    return app.exec();
}
```