# WebUI 常用插件和Embeddings
	- SD WebUI支持直接在界面中安装扩展——它自带一个“扩展市场”，同时支持使用git仓库去安装插件。
	- [界面汉化](https://github.com/dtlnor/stable-diffusion-webui-localization-zh_CN)
	  logseq.order-list-type:: number
	- [图像长宽比指定](https://github.com/thomasasfk/sd-webui-aspect-ratio-helper)：允许指定生成图像为常用长宽比并锁定，交换长宽
	  logseq.order-list-type:: number
	- [提示词增强插件](https://github.com/physton/sd-webui-prompt-all-in-one)
	  logseq.order-list-type:: number
	- [EasyNegative](https://github.com/MeccaStudio/EasyNegative)：常用反向提示词embedding
	  logseq.order-list-type:: number
	- [OneButtonPrompt](https://github.com/Airjen/OneButtonPrompt)：根据配置随机生成提示词
	  logseq.order-list-type:: number
- # 相关资料：
	- ## 模型下载渠道
		- <https://civitai.com>，C站需要魔法网络
		- <https://www.liblib.art>，国内好用的模型网站
		- <https://tusi.cn>，相对没有liblib好用
- # 术语
	- **大模型/Checkpoint**：深度学习的模型，在它刚开始训练和训练完成后，它的“形状”是没有变化的，变化的只有其内部的权重，因此我们使用的大模型，总是某个**检查点**下的大模型，所以这里就直接叫它Checkpoint；以此类推，微调后的大模型也叫Checkpoint。
	  logseq.order-list-type:: number
		- 大模型主要可以分为真实凤，二次元和2.5D（厚涂之类的）。
		- 大模型分为普通/SD模型和SDXL模型；常说的SD1.5/2.0/2.1等是SD模型，普通模型的基础分辨率是`512x512`，**生图时务必用模型本身的基础分辨率去出图，再在该分辨率的基础上做放大操作**，但可以做细微调整，如使用`768x512`出宽屏图等；SDXL模型基础分辨率是`1024x1024`，因此会占用更多显存（起码得16G），体积也更大——普通模型大小大概是2-3G，SDXL模型在此基础上翻倍。SDXL兼容性一般也没有SD模型好。
	- **Lora**：必须依附在大模型上使用，做**微调**，来自于对特定风格图片的训练，体积小，但能提高对特定风格/主题图像的生成效果。Lora也分为普通Lora和SDXL Lora。
	  logseq.order-list-type:: number
	- **VAE**：用于编解码图像的特征空间，通常用于调整生成图片的色彩饱和度和对比度，调整VAE能够使图像更鲜艳或柔和。
	  logseq.order-list-type:: number
	- **ControlNet**：允许用户提供更多的输入，如人体姿态图，线稿，深度图等，使得能更精确地约束和控制生成的图像，避免出现错误的人体结构，多几根手指等情况，同时保证**创作一致性**，比如生成同一个角色的多个角度的图像等。
	  logseq.order-list-type:: number
	- **Hires.fix**：一种**后处理**技术，**在图像完全生成完毕后**提升图像分辨率并修复细节。
	  logseq.order-list-type:: number
		- Hires.fix会先绘制低分辨率（原始）的图像，然后使用算法放大分辨率，再使用高分辨率重绘。算法既有普通的
	- **Refiner**：在图像生成达到某个进度（通常是80%）后介入并接管图像生成；Refiner同样依赖SD模型，但其专注于做细化和提升图像质量，而非完全生成图像。
	  logseq.order-list-type:: number
- # webUI 的采样器和Hires.fix
	- ## 采样器
		- 采样器是第一个参数，重要性可见一斑。采样器的判断基准——速度>质量。
		- `Euler [a]`：老派采样器，**带`a`的表示画面不会收敛，采样步数更高时画面仍旧会有随机性**
		- `DPM`，大部分时候使用的都是`DPM`采样器，`DPM++XXX`等都是一代采样器，`DPM2`为二代采样器，二代的算法速度慢一倍。
		- `DPM`有诸多变体：
			- `Karras`：在迭代8步以后开始**噪点更少**，因此能只用更少的迭代步数
			  logseq.order-list-type:: number
			- `Exponential`：不收敛，速度和Karras类似，细节会丢失一些，**画面更柔和，干净**
			  logseq.order-list-type:: number
			- `Heun`：Euler的改进，质量更好，速度慢一倍
			  logseq.order-list-type:: number
			- `++2S/M`，M是S的升级，不使用S……
			  logseq.order-list-type:: number
			- `++3M`：速度和2M一致，但需要>30步的采样步数才有好结果，CFG需更小
			  logseq.order-list-type:: number
			- `SDE`：随机微分方程，不收敛，高品质，慢
			  logseq.order-list-type:: number
		- 可以使用：
			- `DPM++2M Karras`：速度快，质量ok
			  logseq.order-list-type:: number
			- `DPM++SDE Karras`：不收敛，高品质，慢，适合真实系图片，追求画质
			  logseq.order-list-type:: number
			- `DPM++2M SDE Karras`：是`2M`和`SDE`的折中，不收敛
			  logseq.order-list-type:: number
			- 上述的3M版本
			  logseq.order-list-type:: number
	- ## Hires.fix
		- Hires.fix会**先绘制低分辨率（原始）的图像，然后使用算法放大分辨率，再使用高分辨率重绘**。可以配置Hires.fix的放大算法，以及迭代步数（默认同生成时的迭代步数），重绘幅度。这个重绘是真的重绘——重绘幅度调大了可能会完全改变画面，小了会导致图片有点糊（算法也对此有影响）。
		- Hires.fix的问题——慢，**爆显存**。
		- 迭代步数可以指定5-30之间，不用太高，后面的不明显。重绘幅度好像越大噪点越少。
		- 重绘甚至能修改使用的模型和采样算法，但需要配置界面去显示。
		- webUI提供了一些放大算法，也可以自己下载一些放大算法模型。
		- 关于放大算法（效果均是对二次元来说）：
			- `Latent`系列：把图像转换成潜向量，Latent整个都会有一层雾蒙蒙的光晕，饱和度更高，显存消耗少，速度快，**但非常不清晰**。
			  logseq.order-list-type:: number
			- `Nearest`：最近邻插值，速度快，但边缘感觉很硬且有瑕疵，会出现描边线，感觉得放特别大然后再压缩才好看。
			  logseq.order-list-type:: number
			- `Lanczos`：质量不错，速度较快，色块质感很舒服，但仍旧稍显模糊，有种二次元厚涂感
			  logseq.order-list-type:: number
			- `LDSR`：一种潜向量算法，**细节拉满**，超级慢（而且需要下载，我下不动，还没测试）
			  logseq.order-list-type:: number
			- `R-ESRGAN 4x+`：一种基于对抗神经网络的算法，轮廓边缘会更加锐利（比UltraSharp更锐利），但非轮廓部分会很糊，但放到二次元上好像和lanczos没啥差别……？
			  logseq.order-list-type:: number
			- `R-ESRGAN 4X+ Anime6B`：特别适合二次元，无论是薄涂还是半厚涂，相较于R-ESRGAN 4x+细节更少更软，更平滑，但比其它的噪点都更少，其它的都会有一种类似纸的质感，这个基本看不出来。
			  logseq.order-list-type:: number
			- `4x-UltraSharp`：ESRGAN 的升级，似乎能无脑用，细节比Lanczos更丰富些，但更硬。
			  logseq.order-list-type:: number
				- [4x-UltraSharp](https://civitai.com/models/116225/4x-ultrasharp)需要手动下载，放到`models/ESRGAN`即可
		- 选择上：
			- 折中，选择4x-ultrasharp
			  logseq.order-list-type:: number
			- 要锐度，选择R-ESRGAN 4X+
			  logseq.order-list-type:: number
			- 不想过于锐化，整点厚涂风，或更柔软的质感，选Lanczos
			  logseq.order-list-type:: number
			- 二次元选 R-ESRGAN 4X+ Anime6B
			  logseq.order-list-type:: number
		- 同时，webUI在附加功能里也提供了仅放大和批量放大的功能，允许使用上面的放大算法进行放大，但这时候**似乎要倾向去选择使用深度学习的算法**，lanczos会糊，因为它没重绘嘛。
- # 图生图
	- 图生图会额外以一张图片作为输入，同时会增加一些相关参数，包括缩放模式，重绘尺寸，重绘幅度等，其他的和文生图一样，但是**似乎没有Hires.fix……**这tm怎么用？
	- 图生图功能这里也提供了**图片反推关键字**的功能，但需要下载。
	- 图生图主要是利用上它的**局部重绘**功能用于精修图片——用蒙版圈出不想改变或想改变的内容，然后进行重绘。
	- 重绘有这样的参数：
		- 蒙版内容处理，对蒙版内容，是填充空白还是保留原来的像素，还是使用空白潜空间，还是使用其潜空间噪声，前者能得到更接近原图的效果，后者
		  logseq.order-list-type:: number
	- 重绘能设置参考——是直接不参考（空白或空白潜空间），还是以原图信息（原图像素，或潜空间信息）去参考，后者的话显然能更接近原图效果，前者更多用于去除掉画面内容或完全新增内容。
	- 这玩意儿似乎不和controlnet结合的话效果会很差……