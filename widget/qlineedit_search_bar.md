# QLineEdit 搜索框

想实现的效果是，在左侧加一个搜索图标，右侧当有文字输入时，显示一个清除内容的叉

![screenshot](https://i0.wp.com/i.imgur.com/I5SJDPE.gif)

```python
import sys
from PyQt4 import QtGui, QtCore
 
class SearchLineEdit(QtGui.QLineEdit):
 
    def __init__(self,  parent=None):
        super(SearchLineEdit, self).__init__(parent)
 
        b64_data = 'iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAAz0lEQVQ4ja3SPUoEQRTE8R+DwcQbi/HiGTaewLMYzS1kQRAx9AB6jp1wowFPIBgabjgmNbgf0+Mu+IcHTXdVvUd380uNFlvsUtvs1f7gBj0GdHhIddnro5mkjuAbDZbpOtLkrC9N0qZLg3t84elI00TTmmCbUZcxXxcm7aI9oMItNrjDOz4LAZtoD7jaWz8WjLNU+MDqDO0q2hP2L7HE7CUeP+OUeXzGRaY+4dyP9IbXUsjcV17EPKSKISWqmAb/HPJ8ScBUyO7SgDHkJeb1DzfURQMGQROeAAAAAElFTkSuQmCC'
        data = QtCore.QByteArray.fromBase64(b64_data)
        self.searchButtonPixmap = QtGui.QPixmap()
        self.searchButtonPixmap.loadFromData(data)
 
        b64_data = 'iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAABwklEQVQ4jXXTQYhNYRQH8N97Jkk2IyRJL2PhvaVkMdl8NAs73ctkMVlYiIVm+SZFKbMYGxRZSikp90t2SJelWciCuZNihMZkIStZTRb3e7recOpu7nf+55z//39Oy1CEEDbhGCbQ2Tw6Ch/wFA+KGH8289tD4Cks4TYO4Hv6xnEH7/Msm2xiWg3wLM7jCfplWb5uJuZZth9zOIQLRYyXYV2j81Vcw8myLL8OU6uqarnX7d7FVvR73W5VVdXbVuK8hFc4Upbl6jB4aJI2nmEvxkYwiS1p7NWUcB8rmC5iXM2zbAbHcbiI8UeeZX28xNERtdrvGpzXo6d2YiTPss+YxRdsSDnz+ISJNjpYHIxYxPgLAQs40wCHIsaVlCNhOn/Z2CjyDY8av56rd2FNtNNDb0ioGcykzguYws2kjzzLJBE/ttUbtieEsK+h8tnB2A06p7Az9RjHLjxuhRA2qm18g4nkxI5EZTkV3YZOEeN8avACuzHWghDCCdzDDUz/bxcS+BZOIy9ijM1VvohLSbA+5suyHAAHY8/hIPpFjFdo3EIqMonr2K72eTFdYy/xX8a5ItY+rimQimxA5t/n/DDtyZ/4DUr2o9PRIDtUAAAAAElFTkSuQmCC'
 
        data = QtCore.QByteArray.fromBase64(b64_data)
        self.clearButtonPixmap = QtGui.QPixmap()
        self.clearButtonPixmap.loadFromData(data)
 
        self.clearButton = QtGui.QToolButton(self)
        self.clearButton.setIcon(QtGui.QIcon(self.clearButtonPixmap))
        self.clearButton.setStyleSheet('border: 0px; padding: 0px;')
        self.clearButton.setCursor(QtCore.Qt.ArrowCursor)
 
        frameWidth = self.style().pixelMetric(
            QtGui.QStyle.PM_DefaultFrameWidth)
 
        clearButtonSize = self.clearButton.sizeHint()
 
        self.setMinimumSize(max(self.minimumSizeHint().width(), clearButtonSize.width() + frameWidth*2 + 2),
                            max(self.minimumSizeHint().height(), clearButtonSize.height() + frameWidth*2 + 2))
 
        self.setStyleSheet('''
                            QLineEdit{
                                border: 2px solid gray;
                                border-radius: 12px;
                                padding-right: %dpx;
                                padding-left: %dpx;
                            }
                           ''' % (clearButtonSize.width() + frameWidth + 2,
                                  self.searchButtonPixmap.width() +
                                  frameWidth + 2
                                  )
                           )
 
        self.clearButton.setVisible(False)
        self.clearButton.clicked.connect(self.clearButtonClicked)
 
        self.textChanged.connect(self.__updateClearButton)
 
    def resizeEvent(self, event):
        buttonSize = self.clearButton.sizeHint()
        frameWidth = self.style().pixelMetric(
            QtGui.QStyle.PM_DefaultFrameWidth)
        self.clearButton.move(self.rect().right() - frameWidth - buttonSize.width(),
                              (self.rect().bottom() - buttonSize.height() + 1)/2)
        super(SearchLineEdit, self).resizeEvent(event)
 
    def __updateClearButton(self):
            self.clearButton.setVisible(bool(self.text()))
 
    def clearButtonClicked(self):
        self.clear()
 
    def paintEvent(self, event):
        super(SearchLineEdit, self).paintEvent(event)
 
        painter = QtGui.QPainter(self)
        h = self.searchButtonPixmap.height()
        right_border = 3
        painter.drawPixmap(
            right_border+2, (self.height()-h)/2, self.searchButtonPixmap)
 
if __name__ == '__main__':
    app = QtGui.QApplication(sys.argv)
    win = SearchLineEdit()
    win.show()
    sys.exit(app.exec_())
```

* QByteArray.fromBase64()用来读取base64编码的png图片
* clearButton 是一个QToolButton,仅当输入框有内容的时候才出现
* searchButtonPixmap是左侧的放大镜图标，是在paintEvent中画出来的
* 输入框是圆角的是因为有css border-radius