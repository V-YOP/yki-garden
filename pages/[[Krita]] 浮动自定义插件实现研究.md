- 这个的目的是为了让所有常用操作能直接显示在光标旁边，使得最少的鼠标移动
- 需求：
	- 按特定快捷键在指定位置或鼠标位置去显示我的Widget（配置页）
	  logseq.order-list-type:: number
	- 能够**借**原生DockWidget的内容
	  logseq.order-list-type:: number
	- 能够随意设置所有子组件的位置，大小
	  logseq.order-list-type:: number
	- 能够设置透明背景，背景的鼠标事件能够穿透
	  logseq.order-list-type:: number
	- 允许定义多个Layout，按不同快捷键去显示不同Layout。不考虑多个Layout同时显示的情况，一次只能显示一个Layout
	  logseq.order-list-type:: number
	- 提供一些自定义的Widget，如各种ToolButton，笔刷等
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
	- DOING 透明背景
	  logseq.order-list-type:: number
	  :LOGBOOK:
	  CLOCK: [2024-09-17 Tue 21:55:54]
	  :END:
- ## 关于透明背景
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