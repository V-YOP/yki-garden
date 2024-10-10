- DOING complete me
  :LOGBOOK:
  CLOCK: [2024-10-09 Wed 10:36:33]
  :END:
- 第n次尝试学习 [[Blender]]。
- 在Blender中，所有界面都是特定的Editor，它行为像IDE的侧边栏，或Qt的DockWidget，这给予了Blender界面的极大的灵活性。每个Editor都是……当前正在编辑的场景的一个侧面，或者说它们负责处理特定领域/方面的数据。
- Editor总是属于特定Area的，Area即一个特定的显示区域，Area中的Editor能够在不改变Area范围的情况下进行替换，只需点击Editor左上角的按钮选择新的Editor即可，想从界面中移除一个Area，就右击该按钮，选`Header-Close Area`。每个Editor都有Header和Menu（可能为空），Menu是Editor自己的子菜单，仅关心该Editor中的操作。Editor左上的按钮能切换该Area的Editor
- 最顶部的Layout，Modeling，Sculpting称为Workspace，它们都是特定Editor的集合，使得在处理不同领域的问题时能方便地进行切换。
- 视图操作，即View Navigation，其实更适合翻译成视图导航，但这很怪，还是叫操作吧。所有Navigation操作都是在`3D Viewport` Editor中进行的，这个Editor顾名思义，是用来编辑和查看3D场景的，它包含诸多模式，如对象模式，编辑模式，雕刻模式等。
- Blender 中存在**视图view** 和**镜头camera**，视图供用户浏览场景，而不影响渲染，镜头则是一个**对象object**，定义渲染图像时的视角。因为镜头是对象，所以视图和镜头的操作是有差别的。但是能够将镜头和视图临时地**双向绑定**，使得两者之间操作能互通，也可以调整视图使视角和镜头一致或者反之。
- 视图有一个中心，许多视图操作会和中心相关，不改变中心位置，只改变镜头和中心的相对关系，如缩放，旋转等，而有的操作则会改变中心，如平移，重设中心等，此外视图也能像FPS游戏一样使用WASD操作。
-
- ## 缩放
- 缩放是简单的，鼠标滚轮即可，注意很多可用按键都直接显示在最底下的status bar。
- ## 平移
-
- ## 旋转
- ## 轴向视角
- ## 重设中心
- ## 飞行模式