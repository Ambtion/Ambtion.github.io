# Core Animation 简介
## 关于Core Animation
Core Animation是IOS与OS平台上负责图像渲染和动画的基本设施。Core Animatio可以动画视图和其他可视的元素。使用Core Animation，我们仅仅需要设计几个动画的关键值，（比如动画时间，开发位置，结束位置）就可以启动Core Animation。Core Animation将大部分的绘图任务交给图形硬件处理，从而提升图像渲染速度。

![image](https://github.com/Ambtion/ambtion.github.io/blob/master/imageSource/CoreAnimaiton/CoreAnimation/CoreAnimaiton_FrameWorlk.png?raw=ture)

如图:Core Animation位于AppKit和UIKit的底层。它被紧密的集成到了Cocoa和Cocoa Touch视图工作流中。虽然被紧密的集成，Core Animation也存在扩展功能的接口，这些接口暴露给了应用的视图。使用这些接口能让你更细粒度地控制应用中的动画。


## Core Animation的工作内容

### 1：Core Animation 管理应用内容

Core Animaiton 并不是一个绘制系统，他只是负责将内容合成到硬件上。Core Animation的核心对象是图层（CALayer），图层负责管理，操作视图内容，将捕捉到的视图内容放入缓冲，然后Core Animation合成传输给硬件显示。


### 2：图层属性的改变触发动画

#### 隐式动画
Core Animation创建的大部分动画都包含对图层属性的更改。和视图一样，图层对象也具有边框矩形、坐标原点、尺寸、不透明度、变换矩阵以及许多其他面向可视的属性（如backgroundColor）。这些视图的改变都会触发隐式动画（触发指定关闭）。从而实现一个过度的动画。当然我们也可以修改动画动作，从而修改隐式动画。（动作对象是实现了一个预定义接口的常规对象。Core Animation使用动作对象实现与图层关联的常规的默认动画集合，__当属性发生变化的时候,Core Animation自动检索并执行动画动作__）

#### 显式动画

一般情况下，当我们指定动画的3元素， 动画体（图层的frame）,动画轨迹（动画的开始态，结束态）,已经动画时间。那么我们就可以得到一个显示的动画，动画的颗粒过程将由Core Animation提我们完成。当然如果要求对动画有较高的掌控，那么我们也可以指定动画轨迹。


### 3：管理图层的图层结构

有组织的图层之间存在着父子关系。图层的组织会影响到图层的可见内容。



## 总结
本节主要是对Core Animation的一个概况描述。

* Core Animaition的核心是CALayer
* Core Animaition的主要作用是控制动画颗粒，从而减少开发者使用动画的成本
* Core Animattion管理图层的图层结构，同时负责将内容合成到硬件中，提高绘制效率

## 参看文献

### [Core Animation Programming Guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html)

### [View Programming Guide for iOS](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009503)



