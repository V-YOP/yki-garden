- #Blender 这篇开始就更多地关注理论了，常用的快捷键在前面已经学习地比较熟悉了。
- **对象有多种类型**，不同类型会包含不同类型的数据：
	- 网格Mesh，多边形模型，由顶点，边和面组成
	  logseq.order-list-type:: number
	- 集合Collection，顾名思义；注意集合的Active和其他对象的Active是分开的，新增的对象总是增加到Active的Collection，而当前选择的对象所属集合不一定是Active的。左上角也会写上当前Active的Collection；快捷键`M`允许快速移动对象到特定Collection。
	  logseq.order-list-type:: number
		- 此外，Collection在导入其它blend文件时也会有意义，用户能直接导入整个Collection或导入部分对象。
	- ……其他类型，如Curve，Surface，Camera等
	  logseq.order-list-type:: number
- 编辑操作允许对对象的数据进行编辑。如果说对象模式主要是布置场景和对对象进行变换的话，**编辑模式主要是用来编辑Mesh**，更具体地说，是编辑Mesh对象的顶点Vertex（复数是Vertices），边Edge和面Face，它们都可以是编辑的元素，但最本质上，它们**编辑的都是点集**，只不过线和面包含多个点罢了（对工具的心智模型可以这么建立吗？）。编辑模式会提供多种工具来进行编辑。
- 在编辑模式下，能够选择去编辑点或线或面，**但按住`Shift`再选择的话可以同时编辑点线面中一个或多个**，这个注释写的很清楚。选择了特定点线面后，便能够使用变换和其他左侧看到的工具去进行建模操作了。
- 在编辑模式下能够选择物体的点、线、面，**这里的“选择”和对象的Active或Selected在行为上很多是一样的**（从这个角度出发，我们选择一整个对象的时候，实际上是选择了物体的所有点）——可以通过`反引号键`和`Shift-S`菜单去让镜头聚焦选择的点线面。**通过这种方式，我们能够将3D Cursor置于对象的点，线和面上**。当我们需要对齐对象时，我们总应当使用这种方式来最精准地对齐。
- 建模时可以用X光模式（右上角按钮，或者`Alt-Z`），从而能选择到物体不可见部分的点线面，此时适合用框选`B`的方式去进行选择。
- ## 对象原点origin
- 对象有一个原点**作为其移动，旋转和缩放的根据**，对象的位置也是按原点位置去标识的，正方体的原点默认在体的正中心。**可以通过3D Cursor设置对象原点**，只需右键对象然后`Set Origin-Origin to 3D Cursor`。
- 原点对编辑很有用——想象你调整一个放置在地面上的物体的大小，此时如果原点仍旧是物体的中心，放大后物体下面就会和地面重叠了，而如果原点正好在物体的底面上，则放大后仍旧好好地置于地面上。**原点究竟放到哪里要根据建模任务的需要**。
- # 建模Modeling
- 建模更具体来说是叫多边形建模polygon modeling。从建模的角度出发，可以认为对象是一个顶点集合和线集合组成的**线框**，**面就是一堆线首尾相连包围而成的东西**，而非数学上的平面——这带来一个重要的点：**面可以不是平的**，但实际操作中似乎应当保证面是平的，不然渲染结果会受影响（因为法线之类的东西算不对）。
- DOING 在这里插入总结——建模有哪些类型的工具，负责怎样的业务，使用的心智模型，它们的快捷键...
  :LOGBOOK:
  CLOCK: [2024-10-14 Mon 16:50:23]
  CLOCK: [2024-10-14 Mon 16:50:27]
  :END:
- ## 挤出Extrude
	- **Extrude是最常用的多边形建模工具**，它的行为是**复制选中的元素，并将复制后的元素和原来的元素一一对应连接上**，下图就是他的效果，连接线用红圈标出来了。Extrude按+号时会**按法线方向**把复制后的元素进行移动，但也可以不理会它，任意去移动。
	- 这里并非是完全地复制——能够观察到，如果一个点周围所有点都被选择了，此时进行extrude时，它不会被复制而是被移动……总之不用关心这种细节，能理解就行。
	- 关于法线方向，Blender会根据点旁边的三个面和线旁边的两个面去计算点、线的法线方向。
	- ![image.png](../assets/image_1728953682660_0.png){:height 517, :width 761}
	- Extrude如此常用，以至于它有一个快捷键`E`，选择特定元素后按E就可以进行extrude，**注意按下`E`之后节点就被复制了并进入移动模式，即使后面按下`Esc`，Esc只取消移动操作**。奇妙的是，**只有面会进入沿法线的移动模式，其他的只能任意移动**。Extrude也可以用来让面沿法线移动。
	- Extrude自己也有多种模式，可以通过左边菜单选择当前工具，但更方便的方法恐怕是选择元素后 `Alt+E`唤出菜单：
		- Extrude Region：默认行为，各个元素按平均的法线方向去extrude。
		  logseq.order-list-type:: number
		- Extrude Individual：各个面按自己的法向分别extrude，各个面之间不会有连接
		  logseq.order-list-type:: number
		- Extrude Along Normals：各个面按法向分别extrude，**这些面中，不同面中相交的线会按两个面的平均法线运动，不相交则按该面的法线运动**，运动距离会是一样的，但因为角度有差别，所以结果的面不会和原面是同一法线：
		  logseq.order-list-type:: number
			- ![image.png](../assets/image_1728961968854_0.png)
		- Extrude Manifold：**特别适合向内挤**，向内extrude时能够**智能移除重叠面**，避免自相交等。它适合向内挤压时移除重叠面，保持外观合理，但不能直接用于“打通”——尝试打通（即两边对称向内挤直到刚好重叠）时，extrude manifold（仍）会留下一个面（但不会造成多余的点，わーい），这时候手动删除就行了，至少此时没有重复的点：
		  logseq.order-list-type:: number
			- ![image.png](../assets/image_1728971613237_0.png){:height 415, :width 821}
			- [官方文档](https://docs.blender.org/manual/zh-hans/4.2/modeling/meshes/tools/extrude_manifold.html)的附图更形象，它提到Extrude Manifold能够**融并正交边**，这话似乎是说，如果它发现在移动过程中创建的面和其他的面完全共面，则把这里的其他的面扩大或缩小，详情见附图。
			- ![modeling_meshes_tools_extrude-manifold_example.gif](../assets/modeling_meshes_tools_extrude-manifold_example_1729132966757_0.gif)
			-
- ## 环切Loop Cut
	- Loop Cut不改变原来的形状，它就像在物体上**缠一个橡皮筋包围它，创建对应的顶点和边缘**（或者理解为切一刀）。环切不会创建新的形状，而是只是在旧的形状上增加新的点和边。
	- `Ctrl-R`进入Loop Cut模式，此时：
		- 鼠标置于要切分的边上，能够看到切面预览
		  logseq.order-list-type:: number
		- `鼠标滚轮`调整切面数量，它们距离相等
		  logseq.order-list-type:: number
		- `鼠标左键`应用切面，此时会进入切面的移动模式
		  logseq.order-list-type:: number
		- `Esc`以取消移动，做均等切分，否则应用移动
		  logseq.order-list-type:: number
	-
	-