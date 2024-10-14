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


