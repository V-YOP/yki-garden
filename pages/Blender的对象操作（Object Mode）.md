- #Blender 对象即场景中出现的物体，包括能看到的物体，镜头，光源，对象集合等。对象能进行变换Transformation，即缩放Scale，旋转Rotate和移动Grab，它们的快捷键还挺好记的，它们都支持按全局坐标系和按局部坐标系进行操作。变换是对象的一种……表达形式，**变换不改变对象本身**。
  id:: 670a87ba-c425-4206-92f6-bdf6a3804697
- 对象能够在`N`菜单中进行操作，修改它的属性，也可在Properties Editor中修改，但更多的时候用的还是快捷键。
- 下面的操作都在3D Viewport的**Object Mode**进行，该模式适合用来布置整个场景。
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
- `Shift-D`**拷贝对象**，拷贝时会**进入移动模式**。
  :LOGBOOK:
  CLOCK: [2024-10-12 Sat 23:27:52]
  :END:
- `Alt-D`**引用链接对象**，**这样创建的对象，和原来的对象共享同样的数据**（对Mesh对象而言就是Mesh数据，Mesh反映物体的几何形状，同时包含Material信息，这个后面再表）
- `Object-Relations-Make SIngle User-Object & Data & Materials`允许**“断开”链接**，本质上就是把Mesh或其他类型的书序（以及Mesh中包含的Material）给复制一份（或者叫local化？）。
- ## 移动Grab
- `G`：**进入移动模式**，**对象默认会在和视图平行的平面上进行运动**（所以可以实现调整视图为轴向来方便移动）
- `ESC`，取消操作
- `x`：切换到这样的状态，**在全局坐标系的X轴上进行移动**
- `x-x`：切换到这样的状态，**在局部坐标系的X轴上移动，即自己的X轴上移动**
- `Shift-X`：切换到这样的状态，**在全局坐标系的YoZ平面上移动**
- `Shift-X-X`：切换到这样的状态，**在局部坐标系的YoZ平面上移动**
- 上面的对其他坐标轴也适用
- `Shift`，按住时移动地会更慢，适用于更精确的操作
- `Ctrl`，按住时以1m的粒度对齐坐标轴，同时按下`Shift`时以0.1m为粒度
- ## 缩放Scale
- `S`，**进入缩放模式**，其他的同移动，Ctrl的粒度为0.1，Shift+Ctrl粒度为0.01
- ## 旋转Rotate
- `R`，进入旋转模式，其他的同移动，Ctrl的粒度是5度，Shift+Ctrl粒度为1度
- ## Pivot