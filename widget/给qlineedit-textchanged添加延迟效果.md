# 给QLineEdit textChanged添加延迟效果

![screenshot](https://i0.wp.com/i.imgur.com/tKUpSKx.gif)


如果textChanged触发的太频繁会让用户觉得一直在卡，所以

```python
# source: https://wiki.qt.io/Delay_action_to_wait_for_user_interaction
class DelayedExecutionTimer(QObject):  

    triggered = pyqtSignal(str)
 
    def __init__(self, parent):
        super(DelayedExecutionTimer, self).__init__(parent)
        # The minimum delay is the time the class will wait after being triggered before emitting the triggered() signal
        # (if there is no key press for this time: trigger)
        self.minimumDelay = 700
        self.minimumTimer = QTimer(self)
        self.minimumTimer.timeout.connect(self.timeout)
 
    def timeout(self):
        self.minimumTimer.stop()
        self.triggered.emit(self.string)
 
    def trigger(self, string):
        self.string = string
        self.minimumTimer.stop()
        self.minimumTimer.start(self.minimumDelay)
```
使用方法
```python
def __textChangeTriggered(text):
    print 'textChange trigged:', text
 
if __name__ == '__main__':
    app = QtGui.QApplication(sys.argv)
    w = QtGui.QLineEdit()
 
    filter_delay = DelayedExecutionTimer(w)
    w.textChanged[str].connect(filter_delay.trigger)
    filter_delay.triggered[str].connect(__textChangeTriggered)
 
    w.show()
    sys.exit(app.exec_())
```

其效果是当textChanged 信号发生时，__textChangeTriggered 并不会马上执行，而是会有一定的延迟，只要用户还在输入，minimumTimer就会被不断的stop，并重新初始化，直到用户停止输入，再经过minimumDelay之后，slot才会被执行

增强版
-----
再加入一个maxTimer，当maxDelay的时间到了的时候，即使用户正在输入，slot也会被执行，这个意义在于如果用户一直在打字，我们偶尔也希望slot执行一下下，不要一直等待输入完成 

```python
class DelayedExecutionTimer(QtCore.QObject):
    triggered = QtCore.pyqtSignal(str)
 
    def __init__(self, maxDelay=2000, minDelay=500, parent=None):
        super(DelayedExecutionTimer, self).__init__(parent)
        # The min delay is the time the class will wait after being triggered before emitting the triggered() signal
        # (if there is no key press for this time: trigger)
        self.minDelay = minDelay
        self.maxDelay = maxDelay
        self.minTimer = QtCore.QTimer(self)
        self.maxTimer = QtCore.QTimer(self)
        self.minTimer.timeout.connect(self.timeout)
        self.maxTimer.timeout.connect(self.timeout)
 
    def timeout(self):
        self.minTimer.stop()
        self.maxTimer.stop()
        self.triggered.emit(self.string)
 
    def trigger(self, string):
        self.string = string
        if not self.maxTimer.isActive():
            self.maxTimer.start(self.maxDelay)
        self.minTimer.stop()
        self.minTimer.start(self.minDelay)
```