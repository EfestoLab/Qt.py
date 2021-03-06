## Caveats

There are cases where Qt.py is not handling incompatibility issues.

- [QtCore.QAbstractItemModel.createIndex](CAVEATS.md#qtcoreqabstractmodelcreateindex)
- [QtCore.QItemSelection](CAVEATS.md#qtcoreqitemselection)
- [QtCore.Slot](CAVEATS.md#qtcoreslot)
- [QtWidgets.QAction.triggered](CAVEATS.md#qtwidgetsqactiontriggered)
- [QtGui.QRegExpValidator](CAVEATS.md#qtguiqregexpvalidator)

<br>
<br>

**Tests**

Code blocks in this document are automatically tested at each commit before being accepted into the project. In order for your code to run successfully, follow these guidelines.

1. Each caveat MUST contain (1) a header, (2) description, (3) one or more examples and (4, optional) a solution.
1. Each caveat MUST have a header prefixed with four hashtags, e.g. `#### My Heading`.
1. Each example MAY NOT use more than one (1) binding at a time, e.g. both PyQt5 and PySide.
1. Each example MUST visualise return value and any exceptions thrown.
1. An example MUST reside under a heading, e.g. `#### My Heading`
1. The first line of each example MUST be `# MyBinding`, where `MyBinding` is the binding you intend to test with, such as `PySide` or `PyQt4`.
1. Examples MAY indicate either Python 2 or 3 as `# MyBinding, Python2`
1. Examples MUST be in [doctest](https://docs.python.org/2.7/library/doctest.html) format. See other caveats for samples.
1. Examples MUST `import Qt` (where appropriate), NOT e.g. `import PyQt5`.
1. Examples MAY include `untested` in which case the continuous integration mechanism will look the other way, e.g. `# PyQt4, untested`


<br>
<br>
<br>


#### QtGui.QAbstractItemModel.createIndex

In PySide, somehow the last argument (the id) is allowed to be negative and is maintained. While in PyQt4 it gets coerced into an undefined unsigned value.

```python
# PySide
>>> from Qt import QtGui
>>> model = QtGui.QStandardItemModel()
>>> index = model.createIndex(0, 0, -1)
>>> int(index.internalId()) == -1
True
```

```python
# PyQt4
>>> from Qt import QtGui
>>> model = QtGui.QStandardItemModel()
>>> index = model.createIndex(0, 0, -1)
>>> int(index.internalId()) == 18446744073709551615
True

```

##### Usecase

I had been using the id as an index into a list. But the unexpected return value from PyQt4 broke it by being invalid. The workaround was to always check that the returned id was between 0 and the max size I expect.  

– @justinfx


<br>
<br>
<br>


#### QtCore.QItemSelection

PySide has the `QItemSelection.isEmpty` and `QItemSelection.empty` attributes while PyQt4 only has the `QItemSelection.isEmpty` attribute.

```python
# PySide2
>>> from Qt import QtCore
>>> func = QtCore.QItemSelection.isEmpty
>>> func = QtCore.QItemSelection.empty
```

```python
# PyQt5
>>> from Qt import QtCore
>>> func = QtCore.QItemSelection.isEmpty
>>> func = QtCore.QItemSelection.empty
Traceback (most recent call last):
...
AttributeError: type object 'QItemSelection' has no attribute 'empty'
```

##### Workaround

They both support the `len(selection)` operation.

```python
# PyQt4
>>> from Qt import QtCore
>>> selection = QtCore.QItemSelection()
>>> len(selection)
0
```

```python
# PySide
>>> from Qt import QtCore
>>> selection = QtCore.QItemSelection()
>>> len(selection)
0
```


<br>
<br>
<br>


#### QtCore.Slot

PySide allows for a `result=None` keyword param to set the return type. PyQt4 crashes:

```python
# PySide
>>> from Qt import QtCore, QtGui
>>> slot = QtCore.Slot(QtGui.QWidget, result=None)
```

```python
# PyQt4, Python2
>>> from Qt import QtCore, QtGui
>>> slot = QtCore.Slot(QtGui.QWidget)
>>> slot = QtCore.Slot(QtGui.QWidget, result=None)
Traceback (most recent call last):
...
TypeError: string or ASCII unicode expected not 'NoneType'
```

```python
# PyQt4, Python3
>>> from Qt import QtCore, QtGui
>>> slot = QtCore.Slot(QtGui.QWidget)
>>> slot = QtCore.Slot(QtGui.QWidget, result=None)
Traceback (most recent call last):
...
TypeError: bytes or ASCII string expected not 'NoneType'
```


<br>
<br>
<br>


#### QtWidgets.QAction.triggered

PySide cannot accept any arguments. In PyQt4, `QAction.triggered` signal requires a bool arg.

```python
# PySide
>>> from Qt import QtCore, QtWidgets
>>> obj = QtCore.QObject()
>>> action = QtWidgets.QAction(obj)
>>> action.triggered.emit()  # Note the return value (!)
True
>>> action.triggered.emit(True)
Traceback (most recent call last):
...
TypeError: triggered() only accepts 0 arguments, 2 given!
```

```python
# PyQt4
>>> from Qt import QtCore, QtWidgets
>>> obj = QtCore.QObject()
>>> action = QtWidgets.QAction(obj)
>>> action.triggered.emit(True)
>>> action.triggered.emit()
Traceback (most recent call last):
...
TypeError: QAction.triggered[bool] signal has 1 argument(s) but 0 provided
```


<br>
<br>
<br>


#### QtGui.QRegExpValidator

| Affects       | Version
|:--------------|:-----------------
| PyQt4         | <= 4.8.4

In PySide, the constructor for `QtGui.QRegExpValidator()` can just take a `QRegExp` instance, and that is all.

In PyQt4 you are required to pass some form of a parent argument, otherwise you get a TypeError:

```python
# PySide, untested
>>> from Qt import QtCore, QtGui
>>> regex = QtCore.QRegExp("\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}")
>>> validator = QtGui.QRegExpValidator(regex)
>>> validator = QtGui.QRegExpValidator(regex, None)
Traceback (most recent call last):
...
TypeError: ...
```

```python
# PyQt4, untested
>>> from Qt import QtCore, QtGui
>>> regex = QtCore.QRegExp("\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}")
>>> validator = QtGui.QRegExpValidator(regex, None)
>>> validator = QtGui.QRegExpValidator(regex)
Traceback (most recent call last):
...
TypeError: ...
```
