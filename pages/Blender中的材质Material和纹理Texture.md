- 所谓的**着色Shading**就是配置材质Material，[[Blender]]中，材质Material是一个基于节点的系统（就像门电路，有输入输出），可以随意调整得到复杂的效果。纹理Texture是材质中的一种节点，允许是图像或者程序生成。材质相关配置见`Properties-Material` tab页。**材质等于着色器Shader、效果Effect和材质Texture的组合**。
- 关于材质，有一些规则：
	- 材质必须被一个或多个对象引用，没有被引用的材质会被GC掉；材质的名称是材质的唯一标识符
	  logseq.order-list-type:: number
	- 对象可以有零个或多个材质
	  logseq.order-list-type:: number
	- 拷贝对象时，
	  logseq.order-list-type:: number
	- 可以通过Append或Link引用其他blend文件中的材质
	  logseq.order-list-type:: number
-