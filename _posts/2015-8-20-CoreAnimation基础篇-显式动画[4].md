## 显示动画

隐式动画是在iOS平台创建动态用户界面的一种直接方式，也是UIKit动画机制的基础，不过它并不能涵盖所有的动画类型。在本章中，我们将研究显式动画的框架结构，实现方式，已经显式动画的类型。

IOS系统一个很重要的特色就是它的动画流畅，优美。为了实现这一点，苹果在这个系统中做了很多工作。其中，为了开发者能够快速方便的使用动画，苹果做了一系列的封装，为我们提供了各种不同的“动画武器”，满足开发者的基本需求。当我们选定合适的“动画武器”后，仅仅需要设置几个关键值（比如时间，开始，结束状态）就能实现基本的动画，下图是苹果为开发者封装设计的动画模块。

![CAAniamtion](https://github.com/Ambtion/ambtion.github.io/blob/master/imageSource/CoreAnimaiton/CoreAnimation/CAAnimaiton_Frameworlk.png?raw=ture)

在这个模块中，我们需要明确理解的知识点分别是:

* 1: CAAnimation是做什么的，它实现的NSCoding,CAMediaTiming,CAAction协议分别实现了什么功能
* 2: CAAnimaitonGrop，CAPropertyAnimaiton，CATranstion各自的应用场景
* 3: CABAsicAnimaiton,CAKeyframeAnmation的各自应用场景

### CAAnimation

CAAnimation是一个动画的抽象类,他将动画的共性抽取，然后通过对一些协议的支持，构建动画的基本通道。CAAnimation仅仅提供动画通道，要实现动画，需要使用它的子类。

* 1: 对CAMediaTiming协议的支持，提供动画时间。像duration，beginTime和repeatCount这些时间相关的属性都在这个类中，这些属性结合起来，能够准确灵活的控制着动画时间(__详情可以参考文献controlling Animation Timing__)
* 2: 对CAAction协议的支持，提供获取动画方式的通道(__详情可以参看隐式动画篇幅__)
* 3: 对NSCoding的支持，满足CAAnimation的键值容器特性（个人猜测可能和动画获取的方式有关。同时这一属性能够使我们在开发中对每一个动画快速识别，比如为CAAnimation赋一个ID）
* 4: 定义CAAnimationDelegate,在动画开始或者结束的时候能够回调

###  CAAnimaitonGrop

有时间，我们需要在对一个CALayer执行Position动画的同时，执行一个透明的opacity动画。简单的将两个动画行为添加到图层中，并不能完全的保证2个行为处于一个动画空间，或者是所受到的影响完全一样（比如同一时间的其他动画干扰）。CAAnimaitonGrop就是为了满足这个应用场景而设计的。

需要注意的是CAAnimaitonGrop不会改变其所包含的动画行为的时间。举个例子：

有一个CABAsicAnimaiton A, A.duraiton = 5s. 
有一个CAAnimaitonGrop B,B包含A
当B.duration = 10s。 动画执行时间是10s,执行5s
当B.duration = 4s。  动画执行时间是4s,执行4s
	
	
###  CAPropertyAnimaiton

####  CABAsicAnimaiton
####  CAKeyframeAnmation

###  CATranstion



### 参看文献

##### [controlling Animation Timing](http://ronnqvi.st/controlling-animation-timing/)

#### [Animation Types and Timing Programming Guide](https://developer.apple.com/library/prerelease/mac/documentation/Cocoa/Conceptual/Animation_Types_Timing/Introduction/Introduction.html)

#### [Key-Value Coding Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Conceptual/KeyValueCoding/Articles/KeyValueCoding.html)

