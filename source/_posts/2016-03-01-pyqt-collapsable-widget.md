---
layout: post
title:  "PyQt Collapsable Widget"
date:   2016-03-01 22:30:00
excerpt: "I wanted to add a PyQt collapsable widget to my PyQt UI in Maya (like the frameLayout command). Turned out it does not exist until Qt5, so I had to make it myself..."
categories: 
- fix
tags:
- pyqt
- widget
- collapsable
- python
- framelayout
- maya
comments: true
---

Maya frameLayout
----------------

I wanted to add a submenu in my UI for a Maya script. But I wanted it to be **collapsable**, so that the user could show/hide the info. I had used the *Maya* **frameLayout** command so I was looking for something similar in PyQt, like this:

![Maya frameLayout command]({{site.url}}/assets/images/posts/pyqt-collapsable-widget/maya_framelayout.gif)

After several researches, I found out that this kind of layout was integrated only with Qt5. So for my PyQt4 UI, I had to do a personnalized widget.

With the help from [this post](https://groups.google.com/d/msg/python_inside_maya/vO1pvA4YhF0/WpXMlkpgl54J), especially [this chunk](http://pastebin.com/qYgDDYsB), I came up with the following code:

---

PyQt FrameLayout
----------------

{% highlight python %}
from PyQt4 import QtGui, QtCore

class FrameLayout(QtGui.QWidget):
    def __init__(self, parent=None, title=None):
        QtGui.QFrame.__init__(self, parent=parent)

        self._is_collasped = True
        self._title_frame = None
        self._content, self._content_layout = (None, None)

        self._main_v_layout = QtGui.QVBoxLayout(self)
        self._main_v_layout.addWidget(self.initTitleFrame(title, self._is_collasped))
        self._main_v_layout.addWidget(self.initContent(self._is_collasped))

        self.initCollapsable()

    def initTitleFrame(self, title, collapsed):
        self._title_frame = self.TitleFrame(title=title,
                                            collapsed=collapsed)

        return self._title_frame

    def initContent(self, collapsed):
        self._content = QtGui.QWidget()
        self._content_layout = QtGui.QVBoxLayout()

        self._content.setLayout(self._content_layout)
        self._content.setVisible(not collapsed)

        return self._content

    def addWidget(self, widget):
        self._content_layout.addWidget(widget)

    def initCollapsable(self):
        QtCore.QObject.connect(self._title_frame,
                               QtCore.SIGNAL('clicked()'),
                               self.toggleCollapsed)

    def toggleCollapsed(self):
        self._content.setVisible(self._is_collasped)
        self._is_collasped = not self._is_collasped
        self._title_frame._arrow.setArrow(int(self._is_collasped))

    ############################
    #           TITLE          #
    ############################
    class TitleFrame(QtGui.QFrame):
        def __init__(self, parent=None, title="", collapsed=False):
            QtGui.QFrame.__init__(self, parent=parent)

            self.setMinimumHeight(24)
            self.move(QtCore.QPoint(24, 0))
            self.setStyleSheet("border:1px solid rgb(41, 41, 41); ")

            self._hlayout = QtGui.QHBoxLayout(self)
            self._hlayout.setContentsMargins(0, 0, 0, 0)
            self._hlayout.setSpacing(0)

            self._arrow = None
            self._title = None

            self._hlayout.addWidget(self.initArrow(collapsed))
            self._hlayout.addWidget(self.initTitle(title))

        def initArrow(self, collapsed):
            self._arrow = FrameLayout.Arrow(collapsed=collapsed)
            self._arrow.setStyleSheet("border:0px")

            return self._arrow

        def initTitle(self, title=None):
            self._title = QtGui.QLabel(title)
            self._title.setMinimumHeight(24)
            self._title.move(QtCore.QPoint(24, 0))
            self._title.setStyleSheet("border:0px")

            return self._title

        def mousePressEvent(self, event):
            self.emit(QtCore.SIGNAL('clicked()'))

            return super(FrameLayout.TitleFrame, self).mousePressEvent(event)


    #############################
    #           ARROW           #
    #############################
    class Arrow(QtGui.QFrame):
        def __init__(self, parent=None, collapsed=False):
            QtGui.QFrame.__init__(self, parent=parent)

            self.setMaximumSize(24, 24)

            # horizontal == 0
            self._arrow_horizontal = (QtCore.QPointF(7.0, 8.0),
                                      QtCore.QPointF(17.0, 8.0),
                                      QtCore.QPointF(12.0, 13.0))
            # vertical == 1
            self._arrow_vertical = (QtCore.QPointF(8.0, 7.0),
                                    QtCore.QPointF(13.0, 12.0),
                                    QtCore.QPointF(8.0, 17.0))
            # arrow
            self._arrow = None
            self.setArrow(int(collapsed))

        def setArrow(self, arrow_dir):
            if arrow_dir:
                self._arrow = self._arrow_vertical
            else:
                self._arrow = self._arrow_horizontal

        def paintEvent(self, event):
            painter = QtGui.QPainter()
            painter.begin(self)
            painter.setBrush(QtGui.QColor(192, 192, 192))
            painter.setPen(QtGui.QColor(64, 64, 64))
            painter.drawPolygon(*self._arrow)
            painter.end()
{% endhighlight %}

---

Previews
--------

#### PyQt FrameLayout from Python
<center>
      <img src="https://raw.githubusercontent.com/By0ute/pyqt-collapsable-widget/master/images/pyqt_collapsable_widget.gif">
</center>


#### PyQt FrameLayout from Maya
<center>
      <img src="https://raw.githubusercontent.com/By0ute/pyqt-collapsable-widget/master/images/pyqt_collapsable_widget_maya.gif">
</center>


---

**You can find the code of my github <i class="fa fa-github"></i> [here](https://github.com/By0ute/pyqt-collapsable-widget)**