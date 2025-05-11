# Layouts and Positioning in Qt

Date: 11 May 2025

As shown in [the basic notes](/libraries/Qt/basics.md#installing-a-layout), the layout system is
simple to work as long as our use is a simple `addWidget()` call. However, when it comes to managing
sizes and relative positions between widgets in a given layout, we need to know a little more
about how Qt thinks of size as well as about the facilities provided to specify the designer's intent.

## Sizing 

Here are *some* size-related [properties of a `QWidget`](https://doc.qt.io/qt-6.5/qwidget.html#properties):

- **size** (`QSize`)
	- size of the widget excluding any window frame.
	- "By default, composite widgets which do not provide a size hint will be sized according to the space requirements of their child widgets." ([source](https://doc.qt.io/qt-6.5/qwidget.html#size-hints-and-size-policies))
	- **width**
	- **height**
- **sizeHint** (`const QSize`)
	- The recommended size for the widget (no size if value is invalid (width and/or height is negative))
	- **minimumSizeHint**
		- Recommended minimum size of the widget; ignored if `minimumSize()` is set.
	- A widget must *have* a layout in order to return a valid size hint, meaning the widget must have a layout that, in theory, manages at least one child widget
- **sizePolicy** (`QSizePolicy`):
	- Default layout behavior of the widget; its willingness to be resized (can be specified vertically and horizontally)
		- Can also set stretch factors horizontally and vertically for specifying relative sizes between widgets.
	- `QSizePolicy::Fixed`
		- the widget's size hint is the only acceptable size. The widget never grows or shrinks.
	- `QSizePolicy::Minimum`
		- The widget's size hint is sufficient. The widget can't be made smaller than that, and it can be expanded (though there's no real benefit in doing so.)
    - `QSizePolicy::Maximum`
		- The size hint specifies the maximum size. The widget can't expand, but it can shrink if space is needed for other widgets.
	- `QSizePolicy::Preferred`
		- size hint is the preferred size, but the widget can be expanded or shrunk as needed.
	- `QSizePolicy::Expanding`
		- size hint is reasonable for the widget size, and the widget can be shrunk down, but if there's extra space available, the widget should get as much space as possible.
	- `QSizePolicy::MinimumExpanding`
		- Basically `QSizePolicy::Expanding`, but the widget can't shrink; its size hint specifies the minimum size.
	- `QSizePolicy::Ignored`
		- `sizeHint()` is ignored entirely, and the widget takes up as much space as possible.
- **maximumSize** (`QSize`)
	- max size in pixels
	- **maximumHeight**
	- **maximumWidth**
- **minimumSize** (`QSize`)
	- Specifying this overrides the minimum size defined by a layout. 
	- **minimumHeight**
	- **minimumWidth**

Each widget has a size that is represented by an integer-precision type known as a `QSize`. For my
purposes, I don't *really* need to know much more about this type other than it exists. The
important part is the concept of a size hint. Widgets have a property called a `sizeHint` that
communicates their preferred sizes. Of note, `QWidget`s that don't have a layout return invalid size
hints. Therefore, to ensure a given widget has a size hint, one must either give the widget a layout
(which implies the widget in question is now a *composite*, a container for other widgets) such that
it has a valid size hint or implement `sizeHint()` for that widget as an override.

In other words, calculating a given widget's `sizeHint` is a recursive method in which the
termination is through widgets with pre-implemented size hints (ex. `QPushButton`). If a given
`QWidget` has neither a layout nor a manual `sizeHint` implementation, Qt won't be able to
automatically manage its size and positioning as desired.

### QtDesigner Empty `QWidget` Sizing Trick

When designing custom widgets in QtDesigner, the program requires that each widget has a layout in
order to be properly sized. However, we may want to place an empty container widget to be populated
later in code. To give that widget a proper size, we have to trick QtDesigner. 

1. If already set, break the top-level form's layout. 
2. Give the desired container widget a dummy child widget that has a valid size hint (ex.
   `QPushButton`).
3. Add your desired layout to the container widget, including any desired stretch factors and/or
   size policies.
4. Resize the container widget to take up the approximate amount of space in the form. 
5. Remove the dummy widget
6. Set the top-level form's layout.
7. Iterate as needed. Once the container widget has a layout, we don't need the dummy widget any
   more. However, if you break the layout of the container widget, you will need a new dummy widget
   to set it again.

Say the container widget has object name `d_ContainerWidget` in some form `MyCoolDisplay`. Then,
when you need to place the desired widget `myFancyWidget` in the container:

```cpp
// MyCoolDisplay.h

#include <ui_MyCoolDisplay.h>  // generated header

class MyCoolDisplay : QWidget
{
Q_OBJECT
public:
    MyCoolDisplay(QWidget* parent=nullptr);

private:
    // Keep an instance of the custom widget from QtDesigner
    // One could inherit from it instead, but the C++ community "prefer[s] composition over inheritance".
    ::Ui::MyCoolDisplay d_Ui {};  
};

// ------------------------------------------------------------------------
// MyCoolDisplay.cpp

#include "MyCoolDisplay.h"
#include <widgets/SomeFancyWidget.h>

MyCoolDisplay::MyCoolDisplay(QWidget* parent)
	: QWidget{parent}
{
	d_Ui.setupUi(this);
	
	// Add your widget to the container
	SomeFancyWidget* myFancyWidget = new SomeFancyWidget();
	d_Ui.d_ContainerWidget->layout()->addWidget(myFancyWidget);
}
```
## Automatic Management via Layouts

For automatic positioning and widget sizing, Qt provides the concept of layouts. Layouts
automatically manage the sizes and relative positions of widgets they house. Read about layout
management [at the Qt Documentation](https://doc.qt.io/qt-6.5/layout.html). A quick summary is that
layouts handle:

- child widget positioning
- default and minimum window sizes
- resizing
- changes such as font appearance, widget visibility, and widget count.

### Concrete Layout Types

The `QLayout` class is an abstract base class, so we really don't want to construct this one in
practice. Qt provides four specialized layout types (some of which have further specializations)
that make our lives easier: `QBoxLayout`, `QGridLayout`, `QFormLayout`, and `QStackedLayout`.

[`QBoxLayout`](https://doc.qt.io/qt-6/qboxlayout.html) is likely the mostly commonly-used layout
type. Specifically, its specialization classes `QHBoxLayout` and `QVBoxLayout` are near ubiquitous
in Qt. The two derived classes allow users to lay out widgets in a row or a column, respectively. It
also enables us to add potentially-expanding empty space to control the space distribution in the
layout.

[`QGridLayout`](https://doc.qt.io/qt-6/qgridlayout.html) allows more fine-grained control over a
grid-like layout. A given widget can span multiple rows and columns as desired by the designer, and
one can specify the relative stretch factors among given rows and columns (via `setRowStrech()` and
`setColumnStrech()`, respectively) to specify which cells get the most space.

[`QFormLayour`](https://doc.qt.io/qt-6/qformlayout.html) provides a more convenient form-like
(*label* : *field*) layout in lieu of `QGridLayout`.

[`QStackedLayout`](https://doc.qt.io/qt-6/qstackedlayout.html) provides a means to have an
assortment of widgets as "pages" and switch between them.
See the [`QStackedWidget` page](https://doc.qt.io/qt-6/qstackedwidget.html) for more information.


Note that if a given layout `foo` is not top-level, meaning it does not manage all of the widget's
area and children, you have to add the layout to its parent layout first via
`parentLayout->addLayout(foo)`. Once this is done, you may then add widgets to `foo`.