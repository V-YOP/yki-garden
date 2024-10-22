- 所谓的**着色Shading**就是配置材质Material，[[Blender]]中，材质Material是一个基于节点的系统（就像门电路，有输入输出），可以随意调整得到复杂的效果。纹理Texture是材质中的一种节点，允许是图像或者程序生成。**材质等于着色器Shader、效果Effect和材质Texture的组合**。
- 该节点系统包含多种节点：
	- `Shader Node`：着色器节点，处理物体和光照的交互方式，**着色器节点是材质的最重要的部分之一**，注意这个着色器并非是计算机图形学领域的着色器（底层还是它，但不知道封装了多少层，考虑不要当作同义词）
	- ……其他节点，包括输入、输出（如材质输出）、材质等。
- 关于材质，有一些规则：
	- 材质必须被一个或多个对象引用，没有被引用的材质会被GC掉（引用它的对象称为User，材质可以定义Fake User，即一个虚拟的引用，防止它GC掉）；材质的名称是材质的唯一标识符
	  logseq.order-list-type:: number
	- 对象可以有零个或多个材质
	  logseq.order-list-type:: number
	- 无论是拷贝还是链接对象，材质都会被重用，通过`Object-Realtions-Make Single User`把材质“本地化”。
	  logseq.order-list-type:: number
	- 可以通过Append或Link引用其他blend文件中的材质
	  logseq.order-list-type:: number
- 上面说对象引用材质，但更实践地来说，**对象包含材质槽Material Slots，对象的每个面属于特定材质槽，材质槽引用（Assign）材质**。默认地，对象的所有面属于同一个材质槽，但可以在Shader Editor中修改它以允许物体的各个面持有不同材质。
- 材质相关配置见`Properties-Material` tab页，调整该页的配置，实际上就是调整`Shader Editor`中相应Shader Node的配置。
- # 着色器节点Shader
  :LOGBOOK:
  CLOCK: [2024-10-22 Tue 00:40:44]
  :END:
- Blender提供了一堆着色器，其中BSDF这个词经常出现，它指的是Bidirectional Scattering Distribution Function，一种处理物体对光照的反映的**数学模型**。
- 有如下可能常用的着色器：
	- Diffuse BSDF：简单不透明漫反射
	  logseq.order-list-type:: number