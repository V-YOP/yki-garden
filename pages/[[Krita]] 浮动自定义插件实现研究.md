- 这个的目的是为了让所有常用操作能直接显示在光标旁边，使得最少的鼠标移动。
- 需求：
	- 按特定快捷键在**鼠标位置**去显示我的Widget（配置页）
	  logseq.order-list-type:: number
	- 能够**借**原生DockWidget的内容
	  logseq.order-list-type:: number
	- **能够随意设置所有子组件的位置，大小**，**不考虑重叠**
	  logseq.order-list-type:: number
	- **能够设置透明背景，背景的鼠标事件能够穿透**
	  logseq.order-list-type:: number
	- 允许定义多个Layout，按不同快捷键去显示不同Layout。**不考虑多个Layout同时显示的情况，一次只能显示一个Layout**
	  logseq.order-list-type:: number
	- 提供一些自定义的Widget，如各种ToolButton，笔刷，绘画时间等，**先不考虑配置**
	  logseq.order-list-type:: number
- 需要研究：
	- DOING 借原生DockWidget
	  logseq.order-list-type:: number
	  :LOGBOOK:
	  CLOCK: [2024-09-17 Tue 21:54:12]
	  CLOCK: [2024-09-17 Tue 21:54:19]
	  :END:
	- DOING **编辑**模式……这个恐怕是最难的一步
	  logseq.order-list-type:: number
	  :LOGBOOK:
	  CLOCK: [2024-09-17 Tue 21:54:26]
	  :END:
	- DONE 透明背景且背景处无视鼠标操作
	  logseq.order-list-type:: number
	  :LOGBOOK:
	  CLOCK: [2024-09-17 Tue 21:55:54]--[2024-09-19 Thu 09:16:14] =>  35:20:20
	  CLOCK: [2024-09-19 Thu 09:16:14]--[2024-09-19 Thu 09:16:15] =>  00:00:01
	  :END:
- ## 一般论
	- 该组件实现时不注册DockWidget，而是使用普通的QWidget，设置其父对象为Extension以**避免它被GC**。
	- **组件要为透明背景，并且占据整个屏幕**，使得允许组件距离鼠标任意远都能显示到；组件要stayOnTop以避免失焦，**组件只能再次通过快捷键关闭**
	- 组件提供一个编辑模式，编辑模式下，在屏幕上显示提示性信息，中间显示一个十字，并且背景绘制半透明蒙版阻止操作；所有子组件相对于十字的位置即是唤起组件时子组件相对于鼠标的位置；**先不考虑组件设置的组件（先手写 json 去处理）**。
	- 后面的demo中，对应插件的widget总叫Dashboard，还没决定该具体怎么叫。
- ## 关于透明背景，和编辑模式的十字线标识
  collapsed:: true
	- 透明背景，且置顶是容易实现的，只需要在最外层组件上添加相应标识符，该操作会自动地让透明背景不响应鼠标事件，也不聚焦，但Krita的鼠标滚轮会有Bug，但反正我也不用滚轮，其他的画布快捷键都正常。
	- ```python
	  dashboard.setAttribute(Qt.WA_TranslucentBackground, True) # 透明背景，必须和无边框结合使用
	  dashboard.setWindowFlag(Qt.FramelessWindowHint, True) # 无边框
	  dashboard.setWindowFlag(Qt.WindowStaysOnTopHint, True) # 置顶
	  ```
	- 此外，半透明背景，以及十字的绘制的话，**十字比较复杂，需要用子组件的形式，并且需要控制始终置顶且无法操作**。半透明背景通过**重写paintEvent**去操作。虽然GPT死不认账，但是**Qt似乎是根据透明度决定允不允许操作穿过它**。下面是一个可直接运行的pyqt示例，提供一个checkbox表示是否是编辑模式，非编辑模式下是全透明背景，鼠标可穿透，编辑模式下显示十字和半透明背景，鼠标不可穿透。
	- ```python
	  from typing import override
	  from PyQt5.QtWidgets import QWidget, QApplication, QPushButton
	  from PyQt5.QtGui import QPaintEvent, QPainter, QPen, QResizeEvent, QColor
	  from PyQt5.QtCore import QChildEvent, Qt, QPoint, QTimer
	  import sys
	  
	  
	  class CrossOverlay(QWidget):
	      """
	      十字线组件
	      """
	      def __init__(self, parent=None):
	          super().__init__(parent)
	          self.setAttribute(Qt.WA_TransparentForMouseEvents)  # 让鼠标事件穿透该组件
	          self.setAttribute(Qt.WA_NoSystemBackground)  # 禁止系统背景绘制
	          self.setStyleSheet("background: transparent;")  # 设置背景透明
	          self.raise_()  # 将这个覆盖层放到顶层
	  
	      def paintEvent(self, event):
	          # 绘制十字在顶层组件上
	          painter = QPainter(self)
	          painter.setRenderHint(QPainter.Antialiasing)
	  
	          pen = QPen(Qt.white, 2)
	          painter.setPen(pen)
	  
	          rect = self.rect()
	          center = rect.center()
	  
	          # 绘制水平线
	          painter.drawLine(QPoint(rect.left(), center.y()), QPoint(rect.right(), center.y()))
	  
	          # 绘制垂直线
	          painter.drawLine(QPoint(center.x(), rect.top()), QPoint(center.x(), rect.bottom()))
	  
	          painter.end()
	  
	  class Dashboard(QWidget):
	      def __init__(self) -> None:
	          super().__init__()
	          self.cross = CrossOverlay(self)
	          self.cross.setGeometry(0,0,30,30)
	          self.__editing_mode = False
	          self.setAttribute(Qt.WA_TranslucentBackground, True) # 透明背景，必须和无边框结合使用
	          self.setWindowFlag(Qt.FramelessWindowHint, True) # 无边框
	          self.setWindowFlag(Qt.WindowStaysOnTopHint, True) # 置顶
	          self.__repaint_me()
	  
	      def __repaint_me(self):
	          if self.__editing_mode:
	              self.cross.show()
	          else:
	              self.cross.hide()
	          self.repaint()
	  
	      @override
	      def paintEvent(self, a0: QPaintEvent) -> None:
	          if not self.__editing_mode:
	              return super().paintEvent(a0)
	          # 创建一个绘制器，绘制一个半透明的背景
	          painter = QPainter(self)
	  
	          # 设置半透明背景颜色（例如：黑色，带有一定透明度）
	          painter.setBrush(QColor(49, 54, 59, 127))  # ARGB, 150 为透明度
	          painter.setPen(Qt.NoPen)  # 去掉边框线条
	  
	          # 绘制一个矩形作为背景
	          painter.drawRect(self.rect())
	          
	      # 新组件添加时，把十字线往上移以保证它始终置顶（注意添加十字线组件本身也会触发这个事件，很好笑
	      @override
	      def childEvent(self, a0: QChildEvent) -> None:
	          super().childEvent(a0)
	          if a0.added() and hasattr(self, 'cross'):
	              self.cross.raise_()
	  
	      # 组件大小修改时，保证十字线始终处在中间
	      @override
	      def resizeEvent(self, a0: QResizeEvent) -> None:
	          super().resizeEvent(a0)
	          geo = self.cross.geometry()
	          geo.moveCenter(self.rect().center())
	          self.cross.setGeometry(geo)
	      
	      @property
	      def editing_mode(self):
	          return self.__editing_mode
	  
	      @editing_mode.setter
	      def editing_mode(self, x: bool):
	          self.__editing_mode = x
	          self.__repaint_me()
	  
	  # 测试程序
	  app = QApplication(sys.argv)
	  window = Dashboard()
	  window.setMinimumSize(300, 300)
	  
	  child1 = QWidget(window)
	  child1.setStyleSheet("background-color: red;")
	  child1.setGeometry(50, 50, 120, 120)
	  
	  child2 = QWidget(window)
	  child2.setStyleSheet("background-color: blue;")
	  child2.setGeometry(130, 130, 120, 120)
	  
	  child3 = QPushButton(window) # 检查十字线是否穿透鼠标
	  child3.setText('center11')
	  child3.setGeometry(130, 130, 40, 40)
	  child3.clicked.connect(lambda: print('clicked'))
	  
	  def interval_toggle_editing_mode():
	      size = window.size()
	      window.setMinimumSize(size.width() + 10, size.height() + 10) # 测试大小调整时十字线是否调整（虽然没必要啦）
	      window.editing_mode = not window.editing_mode
	      QTimer.singleShot(1000, interval_toggle_editing_mode)
	  window.show()
	  interval_toggle_editing_mode()
	  sys.exit(app.exec_())
	  
	  ```
- ## 关于借DockWidget的内容
	- 之前学习的时候以为只能通过layout去添加子组件，其实只要设置parent就行了，此时就是相对于父组件的绝对布局。Krita的原生DockWidget中，其Layout都是空的，实际组件内容从`widget`方法去获取到。为此，要拿到其中的widget，只需要设置它的parent即可。这里直接抽象成一个类，让
- ---
- GPT:::
- ## 关于透明背景
  collapsed:: true
	- 方法二：事件过滤器
	  
	  你可以为 `QDockWidget` 安装一个事件过滤器，在过滤器中自定义处理鼠标事件，判断是否在透明区域，如果在透明区域则将事件过滤掉，否则正常处理。
	  
	  下面是一个示例：
	- ```python
	  from PyQt5.QtCore import Qt, QEvent
	  from PyQt5.QtWidgets import QApplication, QMainWindow, QDockWidget
	  
	  class TransparentDockWidget(QDockWidget):
	      def __init__(self, *args, **kwargs):
	          super().__init__(*args, **kwargs)
	          self.setStyleSheet("""
	              QDockWidget {
	                  background: transparent;
	              }
	          """)
	          # 安装事件过滤器
	          self.installEventFilter(self)
	  
	      def eventFilter(self, obj, event):
	          # 过滤鼠标事件，判断是否是透明区域，如果是则忽略事件
	          if event.type() in [QEvent.MouseButtonPress, QEvent.MouseButtonRelease, QEvent.MouseMove]:
	              if self.is_transparent_area(event.pos()):
	                  return True  # 阻止 DockWidget 响应事件
	          return super().eventFilter(obj, event)
	  
	      def is_transparent_area(self, pos):
	          # 自定义逻辑，判断pos是否在透明区域
	          # 这里可以根据你设计的UI判断哪些区域需要穿透，哪些区域不需要
	          return True  # 例子中返回True表示整个区域都透明
	  
	  app = QApplication([])
	  window = QMainWindow()
	  
	  dock_widget = TransparentDockWidget("Dock", window)
	  window.addDockWidget(Qt.LeftDockWidgetArea, dock_widget)
	  
	  window.show()
	  app.exec_()
	  
	  ```
- ## 关于子组件的编辑
  collapsed:: true
	- 这个可以先在普通Qt上写个原型看看，不考虑同时操作多个Widget的需求，不考虑任何界面和操作上的优化，功能最优先，毕竟编辑总是少数的。
	- 要在 PyQt 中创建一个类似 Dashboard 的组件，提供编辑模式以允许用户调整子组件的位置和大小，并能够保存这些修改，可以参考以下思路进行实现。
	- ### 实现步骤
	- **编辑模式的切换**：当进入编辑模式时，允许子组件自由调整位置和大小；当退出编辑模式时，子组件的位置和大小固定。
	- **拖动和调整子组件大小**：需要为每个子组件启用鼠标事件来实现拖动和调整大小的功能。
	- **保存和恢复组件的布局**：在用户编辑完布局后，保存子组件的位置和大小信息（例如保存到 JSON 文件），下次启动时恢复这些设置。
	- ### 具体实现步骤
	- #### 1. 创建 Dashboard Widget
	  
	  首先，创建一个包含子组件的容器，比如 `QWidget` 或 `QFrame`，作为 Dashboard 的容器。子组件可以是按钮、图表、面板等。
	- #### 2. 处理拖动和调整大小
	  
	  为了实现组件的拖动和调整大小，需要为每个子组件安装事件过滤器，监听鼠标事件。以下代码处理了鼠标拖动和大小调整。
	- ```python
	  class DashboardWidget(QWidget):
	      def __init__(self):
	          super().__init__()
	          self.child_widgets = []
	          self.edit_mode = False
	          self.dragging = False
	          self.resizing = False
	          self.current_widget = None
	          self.offset = QPoint()
	          self.init_ui()
	  
	      def init_ui(self):
	          # 添加子组件，保持和上面一样
	          btn1 = QPushButton('Button 1', self)
	          btn1.setGeometry(50, 50, 100, 50)
	          self.child_widgets.append(btn1)
	  
	          btn2 = QPushButton('Button 2', self)
	          btn2.setGeometry(200, 150, 100, 50)
	          self.child_widgets.append(btn2)
	  
	          self.show()
	  
	      def toggle_edit_mode(self):
	          self.edit_mode = not self.edit_mode
	          for widget in self.child_widgets:
	              if self.edit_mode:
	                  widget.setMouseTracking(True)
	                  widget.installEventFilter(self)
	              else:
	                  widget.removeEventFilter(self)
	  
	      def eventFilter(self, obj, event):
	          if self.edit_mode and event.type() == event.MouseButtonPress:
	              self.current_widget = obj
	              self.offset = event.pos()
	              if event.pos().x() > obj.width() - 10 and event.pos().y() > obj.height() - 10:
	                  self.resizing = True
	              else:
	                  self.dragging = True
	              return True
	  
	          elif self.edit_mode and event.type() == event.MouseMove:
	              if self.dragging:
	                  new_pos = obj.pos() + event.pos() - self.offset
	                  obj.move(new_pos)
	              elif self.resizing:
	                  new_width = event.pos().x()
	                  new_height = event.pos().y()
	                  obj.setGeometry(obj.x(), obj.y(), new_width, new_height)
	              return True
	  
	          elif self.edit_mode and event.type() == event.MouseButtonRelease:
	              self.dragging = False
	              self.resizing = False
	              return True
	  
	          return super().eventFilter(obj, event)
	  
	  ```
-