- 这个的目的是为了让所有常用操作，界面能直接显示在光标旁边，使得最少的鼠标移动。
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
	- DONE 借原生DockWidget
	  logseq.order-list-type:: number
	  :LOGBOOK:
	  CLOCK: [2024-09-17 Tue 21:54:12]
	  CLOCK: [2024-09-17 Tue 21:54:19]--[2024-09-19 Thu 11:21:16] =>  37:26:57
	  :END:
	- DONE **编辑**模式……这个恐怕是最难的一步
	  logseq.order-list-type:: number
	  :LOGBOOK:
	  CLOCK: [2024-09-17 Tue 21:54:26]--[2024-09-19 Thu 14:03:06] =>  40:08:40
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
- ## 关于借Docker
	- 之前学习的时候以为只能通过layout去添加子组件，其实只要设置parent就行了，此时就是相对于父组件的绝对布局。Krita的原生DockWidget中，其Layout都是空的，实际组件内容从`widget`方法去获取到。为此，要拿到其中的widget，只需要设置它的parent即可。这里直接抽象成一个类去包装借来的组件，并保证它始终充满自己。
	- 下面的代码得在Krita的Scripter中执行，其中一个要点是要让QApplication引用它的实例，不然它会被GC掉。**在自己被删除、关闭时还回组件的功能似乎无效，得进一步研究**。
	- ```python
	  """在打开文档后在Scripter中开始执行，需要避免窗口被GC"""
	  from PyQt5.QtGui import QCloseEvent, QResizeEvent
	  from krita import *
	  from PyQt5.QtCore import *
	  from PyQt5.QtCore import QEvent, QObject, Qt
	  from PyQt5.QtGui import *
	  from PyQt5.QtWidgets import *
	  from PyQt5.QtWidgets import QWidget
	  
	  class BorrowWidget(QWidget):
	      def __init__(self, dock_widget: DockWidget, parent: QWidget | None = None) -> None:
	          super().__init__(parent)
	          self.__borrowed_widget = None
	          self.__dock_widget = dock_widget
	          self.setMinimumSize(100, 100) # 给定一个基准大小
	          self.destroyed.connect(self.return_back)
	  
	      def resizeEvent(self, a0: QResizeEvent) -> None:
	          if not self.__borrowed_widget:
	              return super().resizeEvent(a0)
	          self.__borrowed_widget.setGeometry(self.rect()) # rect 方法返回 (0, 0, *self.size())
	          return super().resizeEvent(a0)
	  
	      def borrow(self):
	          if self.__borrowed_widget:
	              return
	          self.__borrowed_widget = self.__dock_widget.widget()
	          self.__dock_widget.setWidget(QLabel('Borrowed'))
	          self.__borrowed_widget.setParent(self)
	          qInfo(f'{self.__borrowed_widget.geometry()=}')
	          self.__borrowed_widget.setSizePolicy(QSizePolicy.Expanding, QSizePolicy.Expanding)
	          self.__borrowed_widget.show()
	          self.adjustSize()
	          self.show()
	  
	      def return_back(self):
	          if not self.__borrowed_widget:
	              return
	          self.__dock_widget.setWidget(self.__borrowed_widget)
	          self.__borrowed_widget = None
	          self.hide()
	          
	  if __name__ == '__main__':
	      win = QWidget()
	      QApplication.instance().win = win # 避免被GC
	      win.setFixedSize(1000, 800)
	      dock = next(i for i in Krita.instance().dockers() if i.objectName() == 'sharedtooldocker')
	  
	      container = BorrowWidget(dock, win)
	  
	      borrow_btn = QPushButton(win)
	      borrow_btn.clicked.connect(container.borrow)
	      borrow_btn.setText('borrow')
	  
	      return_back_btn = QPushButton(win)
	      return_back_btn.clicked.connect(container.return_back)
	      return_back_btn.setText('return back')
	  
	      borrow_btn.setGeometry(10, 10, 120, 50)
	      return_back_btn.setGeometry(150, 10, 120, 50)
	  
	      container.setGeometry(10, 100, 600, 300)
	  
	      win.show()
	  ```
- ## 关于编辑模式
	- 编辑模式的话，使用eventFilter去监听在组件上的QEvent.MouseMove, QEvent.MouseButtonPress, QEvent.MouseButtonRelease事件，鼠标按下时，根据按下的位置是在边界还是在内部去决定是调整大小还是移动位置，并记录初始按下的位置，在后续的移动事件中根据鼠标和原本按下位置的偏移量去操作组件geometry，实际上还是比较简单的，下面贴上该EventFilter（实现上还有提升空间）（注意EventFilter对象也需要专门去引用它，不然会被GC掉而且没半点痕迹）：
		- ```python
		  class _EditGeometryEventFilter(QObject):
		          def __init__(self, parent: QObject | None = None) -> None:
		              super().__init__(parent)
		              self.__editing_mode = False
		  
		              self.__editing_widget = None
		              self.__editing_widget_original_geo: Optional[QRect] = None
		              self.__editing_global_pos_start: Optional[QPoint] = None
		  
		              self.__editing_anchor: Literal['B', 'T', 'R', 'L', 'LT', 'RT', 'LB', 'RB', 'INNER', None] = None
		          
		          @property
		          def editing_mode(self):
		              return self.__editing_mode
		          
		          @editing_mode.setter
		          def editing_mode(self, v: bool):
		              self.__editing_mode = v
		  
		          def get_editing_ahchor(self, wh: QSize, offset: QPoint, border_threshold = 15) -> Literal['B', 'T', 'R', 'L', 'LT', 'RT', 'LB', 'RB', 'INNER']:
		              border_threshold = min(wh.width() // 4, wh.height() // 4, border_threshold)
		  
		              close_to_vertical_border = offset.x() <= border_threshold or abs(wh.width() - offset.x()) <= border_threshold
		              close_to_horizontal_border = offset.y() <= border_threshold or abs(wh.height() - offset.y()) <= border_threshold
		              
		              if not close_to_horizontal_border and not close_to_vertical_border:
		                  return 'INNER'
		  
		              horizontal_label = 'L' if offset.x() * 2 < wh.width() else 'R'
		              vertical_label = 'T' if offset.y() * 2 < wh.height() else 'B'
		  
		              if close_to_horizontal_border and not close_to_vertical_border:
		                  return vertical_label
		              if close_to_vertical_border and not close_to_horizontal_border:
		                  return horizontal_label
		              return f'{horizontal_label}{vertical_label}'
		  
		          def __setting_cursor(self, child_widget: QWidget, event: QMouseEvent):
		              # 鼠标移动出组件时，重设光标
		              if event.type() == QEvent.Leave:
		                  child_widget.unsetCursor()
		                  return 
		              # 否则，设置光标
		              match self.get_editing_ahchor(child_widget.size(), event.pos()):
		                  case 'INNER':
		                      cursor = Qt.ClosedHandCursor # 抓手
		                  case 'L' | 'R':
		                      cursor = Qt.SizeHorCursor # 横向调整大小
		                  case 'T' | 'B':
		                      cursor = Qt.SizeVerCursor # 纵向调整大小
		                  case 'LT' | 'RB':
		                      cursor = Qt.SizeFDiagCursor # 左上到右下对角线
		                  case 'RT' | 'LB':
		                      cursor = Qt.SizeBDiagCursor # 左上到右下对角线
		                  case _: raise NotImplementedError("Impossible")
		              child_widget.setCursor(QCursor(cursor))
		  
		          def __stop_editing(self):
		              self.__editing_widget = None
		              self.__editing_widget_original_geo = None
		              self.__editing_global_pos_start = None
		              self.__editing_anchor = None
		  
		          def __handle_mouse_event(self, child_widget: QWidget, event: QMouseEvent):
		              # 不在编辑状态时，设置光标
		              if not self.__editing_widget and event.type() in (QEvent.Leave, QEvent.MouseMove):
		                  self.__setting_cursor(child_widget, event)
		                  return
		              # 无视离开事件
		              if event.type() == QEvent.Leave:
		                  return
		              
		              # self.__editing_widget只是个标识，不使用，目标是为了避免编辑到其它的 widget
		              if self.__editing_widget is not None and self.__editing_widget is not child_widget:
		                  return
		      
		              # 如果是鼠标按下，设置状态，设置光标
		              if event.type() == QEvent.MouseButtonPress:
		                  print('按下')
		                  self.__setting_cursor(child_widget, event)
		                  self.__editing_anchor = self.get_editing_ahchor(child_widget.size(),event.pos())
		                  self.__editing_widget = child_widget
		                  self.__editing_widget_original_geo = child_widget.geometry()
		                  self.__editing_global_pos_start = event.globalPos()
		                  return
		  
		              if event.type() == QEvent.MouseButtonRelease:
		                  # 如果是鼠标抬起，重设状态，重设光标    
		                  print('抬起')
		                  child_widget.unsetCursor()
		                  self.__stop_editing()
		                  return
		  
		              offset = event.globalPos() - self.__editing_global_pos_start
		              geo = QRect(self.__editing_widget_original_geo)
		              print(self.__editing_anchor)
		              # 鼠标移动事件，狠角色要来了
		              # 如果是编辑位置，问题容易一些——获取当前鼠标位置相对于原位置的偏移量，并据此设置geometry
		              # 编辑大小时，虽然让人感觉很虚，但是QRect里有很多方法让我不需要自己实现任何多余计算逻辑
		              match self.__editing_anchor:
		                  case 'INNER':
		                      geo.moveTo(geo.x() + offset.x(), geo.y() + offset.y())
		                  case 'L':
		                      geo.setLeft(geo.left() + offset.x())
		                  case 'R':
		                      geo.setRight(geo.right() + offset.x())
		                  case 'T': 
		                      geo.setTop(geo.top() + offset.y())
		                  case 'B': 
		                      geo.setBottom(geo.bottom() + offset.y())
		                  case 'LT':
		                      geo.setTopLeft(geo.topLeft() + offset)
		                  case 'RT':
		                      geo.setTopRight(geo.topRight() + offset)
		                  case 'LB':
		                      geo.setBottomLeft(geo.bottomLeft() + offset)
		                  case 'RB':
		                      geo.setBottomRight(geo.bottomRight() + offset)
		              
		              child_widget.setGeometry(geo)
		  
		          def eventFilter(self, obj: QObject, event: QEvent) -> bool:
		              """
		              eventFilter返回True表示我处理完了，不再传递给下一个eventFilter或者组件本身，
		              """
		              assert isinstance(obj, QWidget)
		              if not self.editing_mode:
		                  return False
		              if event.type() in (QEvent.FocusAboutToChange,):
		                  obj.unsetCursor()
		                  self.__stop_editing()
		                  return False
		              if event.type() not in (QEvent.MouseMove, QEvent.Leave, QEvent.MouseButtonPress, QEvent.MouseButtonRelease):
		                  return False
		              self.__handle_mouse_event(obj, event)
		  
		              return event.type() in (QEvent.MouseButtonPress, QEvent.MouseButtonRelease)
		  ```
	- 这里需要注意——eventFilter的执行顺序是：
		- 自己的eventFilter
		  logseq.order-list-type:: number
		- 如果返回False的话，转给自己的相应事件处理方法，返回True就不再转发（这就是为什么eventFilter叫Filter）
		  logseq.order-list-type:: number
		- that's it。你以为会可以冒泡给父组件？没这事，只能调父类的同方法把事件传递给父类，和父组件无关。**Qt没有冒泡机制**。
		  logseq.order-list-type:: number
	- 问题就在这里——eventFIlter没法递归地禁止所有子组件的操作；GPT建议可以使用`setEnabled(False)`，或`setAttribute(Qt.WA_TransparentForMouseEvents, True)`，我觉得还有一种法子是在组件上直接盖一个透明的组件去阻止它的交互。但这有必要把所有子组件再包一层。