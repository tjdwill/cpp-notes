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
3. Set the Status tip.
4. Connect the action's `triggered()` signal to some desired slot.
5. Add the action to any desired menu and/or toolbar.
