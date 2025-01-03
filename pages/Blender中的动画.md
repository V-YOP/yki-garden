- 第七章 ((67152902-8a66-485f-8eb4-fa3478830e63))
- [[Blender]]的一大领域便是制作动画而非单张图片。**关键帧Keyframe**是动画（即使是其他领域的动画，如日本动画，迪士尼3D动画）的一个**不可或缺**的重要概念，每个关键帧是一个完整的画面，**连续的画面可以认为就是不同的关键帧之间的切换**。在Blender中，**关键帧属于特定对象**，表示该对象在特定时间点上的**状态**。Blender自动处理关键帧之间的**插值Interpolation**，当然这里也可以进行细微的调整。关键帧安排在**时间线Timeline**中，它就像视频的时间轴。
- 实际上，可以说，**一切动画的绘制过程，就是先绘制关键帧，然后通过“插值”得到关键帧之间的过渡**。
- 在Krita中，关键帧以对象在特定时间点的**状态**，即属性为表现。这里的属性可能是transformation或其它。比如，第一个关键帧中，值为0，第二个关键帧中，值为100，Blender就会自动处理两个关键帧之间值从0到100的转化，即插值。不止是数值属性，甚至下拉菜单属性也能设置关键帧……很神奇。
- 关键帧在时间轴上。**时间轴以帧为单位**。帧速率即播放时帧的切换的速率，帧率使用**每秒帧数FPS（Frame Per Second）**来衡量，我们说电影是24帧，游戏是60帧，就是说它们的帧速率是24FPS，60FPS。帧速率是场景的属性。