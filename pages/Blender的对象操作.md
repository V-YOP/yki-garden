- 对象即场景中出现的物体，包括能看到的物体，镜头，光源等。对象能进行变换Transformation，即缩放Scale，旋转Rotate和移动Grab，它们的快捷键还挺好记的，它们都支持按全局坐标系和按局部坐标系进行操作。
- ## 选择对象
- 关于选择对象的快捷键，常用的就不多说了，符合习惯，注意Blender关于选择的对象有两种状态——Active和Selected，同时能有多个Selected的对象，但只有一个Active的对象，Active可以认为是一种特殊的Selected。**Active对象绿色边框，Selected对象红色边框**。
- `Shift-鼠标左键`，**toggle对象的选择状态**，简单来说**就是设置对象为Active或者取消选中**：
	- Active -> 未选中
	- 未选中 -> Active
	- Selected -> Active
- `Shift-S`开启Cursor的一个菜单，可以调整Cursor，Active和Selection（即Selected的对象）之间的位置，如移动Cursor到Selection，移动Cursor到原点，到Active等。
- 似乎3D Cursor在创建对象和建模时更重要？
- ## 创建对象
- `Shift-A`呼出创建对象的菜单，**对象会创建于Cursor处**，在创建完对象后，3D Viewport左下角处会出现一个菜单能**调整刚才创建的对象的属性**。这里体现了Blender的设计哲学——**不展示dialog，而是直接用默认配置给你做操作，操作后你再按需更改**。
- DOING 拷贝对象
  :LOGBOOK:
  CLOCK: [2024-10-12 Sat 23:27:52]
  :END:
- ## 移动Grab
- `G`：**进入移动模式**
- `x`：在全局坐标系的X轴上进行移动
- `x-x`
- ## 缩放Scale
- `S`，**进入缩放模式**
- `x`：按全局坐标系的X轴缩放
-
- ## 旋转Rotate