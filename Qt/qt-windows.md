# Qt Windows

Firstly, in Qt, any `QWidget` that doesn't have a parent (`QWidget* parent == nullptr`) is a window.

## Shape-changing Windows

These windows are the type you see that in applications with basic and advanced options. On a button click, the hidden advanced options come into view, expanding the current window. 

To do so, here are the main things you need (assuming a custom class):

1. For each given widget (ex. combo box), connect the toggle button (ex. "Advanced >>>") signal with the visibility slot on the widget:
   
   ```cpp
    // button and widget are pointers
    QObject::connect(button, &QPushButton::toggled, widget, &QWidget::setVisible);
   ```

2. In the given containing parent's constructor, set the visibility of the widgets in question to hidden via `someWidget->hide()`.

3. Set the window's size constraint to fixed, meaning the user can't change the window size, but Qt can.

    ```cpp
    layout ()->setSizeConstraint(QLayout::SetFixedSize);
    ```

## Modal vs. Modeless Windows

A *modeless* window is a window that is allowed to run independently. It doesn't block the other windows. These windows typically have their signals connected to slots that implement some behavior or functionality. A modeless window allows the user to switch between it and other windows in the application at will. *Modal* windows, however, **do** block the other windows, effectively barring the application from running until the user is done with the window. Have you ever opened some dialog box in a program and attempted to click outside of it, only to have the window blink a few times and an alarm bell sound trigger? That was a modal window.

Unless explicitly stated otherwise through the `setModal()` method, dialog windows are modeless if `show()` is called. Additionally, you want to make use of the *raise* and *activateWindow* methods to ensure that the window is both properly raised to the front of the window queue and active, even if the window is already non-hidden. When `exec()` is called instead of `show()`, the window is modal.


