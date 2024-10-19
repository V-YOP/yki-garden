- Modifier是对象的属性，它提供**非破坏性编辑**功能，就像图层蒙版，**可插拔**。同时它也可以`apply`，把自己的效果永久地应用到对象上，但做这事儿之前应当做备份。
- Modifier在Properties Editor中管理，选择Mesh Object后，能看到一个扳手图标，那里就是Modifier。可以有多个Modifier，它们**应用的顺序从上到下**。Modifier可以设置在Edit Mode，viewport和render时的可见性，对象在Outlines中也可以调整后两者。
- 下面学习各种不同的Modifier。Modifier的话关键就在于它的各种**配置项**了。
- ## 细分曲面 Subdivison Surface
- 可能是最常用的修改器之一。细分曲面为选择的多边形添加一大堆新面以对多边形进行**平滑**。
- **一个常见的建模工作流是先从低多边形的版本开始，然后再增加它的多边形数量**。
  id:: 6712fc8d-bce2-4ecb-a932-772005db4d35
- ![image.png](../assets/image_1729306064470_0.png){:height 256, :width 641}
- 决定模型光滑程度的是**子面数**以及**算法**，总的来说，有如下配置项：
	- Level，**细分等级**——对每个原来的面，细分得到的面数为$$2^{Level}$$。viewport和render时能设置不同的细分等级，这是优化策略。调太大会增加计算量，甚至让blender崩溃。
	  logseq.order-list-type:: number
	- Algorithm，**细分算法**：
	  logseq.order-list-type:: number
		- **Catmull-Clark**：**根据**原有网格的顶点和边去生成曲面，适用于有机形态
		  logseq.order-list-type:: number
		- **Simple**：**仅细分面**，不改变顶点，即**不改变原始形状**
		  logseq.order-list-type:: number
	- Quality，细分质量，关系到细分的正确性，高准确度会增加计算开销
	  logseq.order-list-type:: number
	- DOING 其他的
	  logseq.order-list-type:: number
	  :LOGBOOK:
	  CLOCK: [2024-10-19 Sat 16:34:13]
	  CLOCK: [2024-10-19 Sat 16:34:15]
	  CLOCK: [2024-10-19 Sat 16:34:15]
	  :END:
- 在看到细化时出现错误的形状的时候，检查是否是**法线出现问题**，通过右上角的菜单选择查看面的法线，如果确实存在错误的法线（如果某个面看不到法线，说明它的法线是向内的，错了），选择它，然后`Shift-N`修复法线。
- （使用非Simple的算法）创建细分曲面时，它会尝试参考原有的多边形，具体来说是顶点和边去进行细分，避免做出太大的改动。因此**可以通过创建新的边来修改曲面细化的行为**，这特别适用于要让某些部分的转折变得更硬的时候，这时候要使用Loop Cut工具（想不到吧！），创建更多的**边环**让细分算法作参考。
- ## 镜像Mirror
- 镜像修改器允许沿X，Y，Z轴（以及多个轴）进行镜像镜像会以物体原点位置为参考点，同时允许设置自动merge。**镜像没什么好说的**，试试总能看明白。
- ## 数组Array
- 字面意思，沿任意方向（但仅一个方向）进行任意的重复，重复可以通过指定重复次数或最大长度等来确定重复次数。
- 数组能选择一个**参考对象**，Blender会计算原对象相对于参考对象的**偏移**，然后每次创建新对象的时候，**应用该偏移**，从而能达到间距逐渐增大，逐渐旋转，逐渐缩放以及他们的组合等效果。参考对象可以使用**空对象**——它不可见，但能够进行变换。
- 数组有三种偏移，它们分别是（该顺序同时也是它们的应用顺序）：
	- 相对偏移Relative Offset，以自己尺寸的百分比进行相对偏移。
	  logseq.order-list-type:: number
	- 固定偏移Constant Offset，以固定尺寸进行偏移
	  logseq.order-list-type:: number
	- 对象偏移Object Offset，即上面说的参考对象
	  logseq.order-list-type:: number
- ## 布尔 Bool
- 使用布尔运算去创建对象，从这个修改器的角度看来，**对象不是线框，而是真真正正的几何体了**，布尔运算允许对这些几何体进行交集Intersection，差集Substraction，并集Union运算，从而得到新的几何体。布尔运算必须依赖另一个对象的形状去发挥作用·。
- 布尔运算最典型的用途之一是给对象**打圆孔**。