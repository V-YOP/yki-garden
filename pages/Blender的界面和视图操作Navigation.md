- 第n次尝试学习 [[Blender]]，这次要把笔记做好，使得之后再尝试捡起来的时候只要以这里为单一真相来源即可。学习顺序是先实践再理论（但3D建模部分要先把理论稍微过一遍）。首先是布局和视图操作。
- 在Blender中，所有顶层组件都是特定的Editor，如展示3D场景的`3D Viewport`，展示元素及其父子关系的`Outliner`等，它行为像IDE的侧边栏，或Qt的DockWidget，能够随意进行调整，这给予了Blender界面的极大的灵活性。每个Editor都是当前正在编辑的场景的一个侧面，或者说它们负责处理特定领域/方面的数据。
- Editor总是属于特定Area的，Area即一个特定的显示区域，Area中的Editor能够在不改变Area范围的情况下进行替换，只需点击Editor左上角的按钮选择新的Editor即可，想从界面中移除一个Area（等于是移除该Editor），就右击该按钮，选`Header-Close Area`。Area能够最大化（`Ctrl-Space`）和占满全屏幕`Ctrl-Alt-Space`（最大化然后不显示顶层菜单和status bar），再次使用相应快捷键返回原状态。
- 每个Editor都有Header和Menu（可能为空），Menu是Editor自己的子菜单，仅关心该Editor中的操作。Header如果隐藏了会显示一个小三角在Editor顶部使得后续能再唤出来。
- Area的布局好像和之前研究的Qt的DockWidget的布局是类似的（但没有标签页），任何复杂布局能和只能通过多次的垂直分和水平分去达到：
  collapsed:: true
	- {{embed ((6703f099-d439-43bb-8148-0af7d3f740c3))}}
- 从Area的四个边角向Area内拖动，则能够对Editor进行水平或竖直切分，向外移动则能够移动Area的位置，向外移动的行为则覆盖掉该方向的Area。
- Editor可以单独掏出来自成一个窗口
- 最顶部的Layout，Modeling，Sculpting称为Workspace，它们都是特定Editor的集合，使得在处理不同领域的问题时能方便地进行切换。
- ---
- 视图操作，即View Navigation，其实更适合翻译成视图导航，但这很怪，还是叫操作吧。所有Navigation操作都是在`3D Viewport` Editor中进行的，这个Editor顾名思义，是用来编辑和查看3D场景的，它包含诸多模式，如对象模式，编辑模式，雕刻模式等。
- Blender 中存在**视图view** 和**镜头camera**，视图供用户浏览场景，而不影响渲染，镜头则是一个**对象object**，定义渲染图像时的视角。因为镜头是对象，所以视图和镜头的操作是有差别的。但是能够将镜头和视图临时地**双向绑定**，使得两者之间操作能互通，也可以调整视图使视角和镜头一致或者反之。
- 视图有一个虚拟的，不可见的**中心**，许多视图操作会和中心相关，不改变中心位置，只改变镜头和中心的相对关系，如缩放，旋转等，而有的操作则会改变中心，如平移，重设中心等，此外视图也能像FPS游戏一样使用WASD操作。**在`N`菜单中能够将中心锁定到游标位置，这在某些时候会更符合直觉**。
- 视图可以选择透视投影和正交投影，正交投影就是从无穷远处观察物体，使得物体完全不会有透视变形。
- 因为我的键盘上没有小键盘，所以我会避免任何使用小键盘的情况……**如果真的有常用的东西，可以用收藏菜单去处理，它按`Q`去呼出**，我把调整重心到3D光标这个加进去了。
- ## 缩放Zoom
- `鼠标滚轮`，注意很多可用按键都直接显示在最底下的status bar，它会展示当前鼠标所在Editor的部分快捷键。这里其实叫缩放不是太准确——相机的位置确确实实是接近和远离中心了。
- ## 绕中心运动Orbit
- `鼠标中键-拖动`然后拖动，使得视角能绕着中心运动，运动过程中相机离中心的距离是不变的。
- ## 滚转 Roll
- 视图模式下只有使用小键盘快捷键才能进行滚转，`Shift-小键盘4`和`Shift-小键盘6`。不使用小键盘的话就只能绑定到镜头对象，然后用调整滚转角的方式去进行滚转，这里不表。
- ## 平移 Pan
- `Shift-鼠标中键`能够将视图在屏幕平面上进行平移。
- ## 对齐Align View和重设中心
- 对齐会比较实用，能够调整视角到对齐，对齐和重设中心的功能会稍有重合，这里写一起
- `反引号键`是**绝杀**——按下它呼出菜单，允许切换到轴向，和切换到相机视角，同时支持重设中心到当前正在选择的对象。实际上关于对齐相关操作，用这个菜单很多时候就是**足够**了。
- `Alt-鼠标中键`，**替代小键盘**，按住Alt，然后按下鼠标中键进行**短暂**拖动，其将**调整到相应方向的坐标轴方向**上。
- `鼠标中键-Alt-拖动`，按住鼠标中键进入Orbit状态后，按住Alt再拖动的话，在接近**坐标轴**方向以及两两和三三轴的45度方向时会**吸附**上去。
- `View-Frame Selected`和`Numpad .`**重设中心到所选对象**
- `Home`重设视图到**能看到场景中所有对象**
- `View-Align View-Center View To Cursor`**重设中心到3D游标**
- `Shift-鼠标右键`**设置3D游标 3D Cursor**
- `Shift-C`**重置**一切——重设游标，同时重设中心和缩放使能够看到**整个场景**
- `Shift-B`——调整视角使适合**看到框选的对象**
- `Ctrl-Alt-Numpad 0` **将相机对齐到视图**
- `/`切换到**局部视图**，只展示当前正选择的对象，同时重设中心到它
- ## 飞行模式
- `Shift-反引号键`开启**FPS模式**，按`Tab`能提供重力，从而切换到步行模式，在这个模式下就是熟悉的感觉了。
- `鼠标滚轮`调整移动速度，有时候速度会莫名变得非常慢，不知道原因。