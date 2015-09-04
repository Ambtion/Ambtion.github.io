## 动画时间
CAAnimation作为IOS核心动画的抽象基类，其中有一个很重要的功能就是实现了一个叫CAMediaTiming的协议。

### CAMediaTiming协议
CAMediaTiming协议定义了在一段动画内用来控制逝去时间的属性的集合，CALayer和CAAnimation都实现了这个协议，所以时间可以被任意基于一个图层或者一段动画的类控制。

CAMediaTiming定义了8个属性。通过这8个属性的组合，准确的控制着动画时间。

* duration
* autoreverses
* repeatCount
* repeatDuration
* beginTime 
* speed
* timeOffset
* fillMode

#### 持续和重复
Core Amimation主要用过__duration__控制动画持续时间，__autoreverses__,__repeatCount__,__repeatDuration__控制动画的重复周期。需要注意的是repeatCount和repeatDuration可能会相互冲突，所以一半只对其中一个指定非零值。

#####  duration
	duration是一个CFTimeInterval的类型（类似于NSTimeInterval的一种双精度浮点类型），对将要进行的动画的一次迭代指定了时间.duration默认是0.但是“0”仅仅代表了“默认”，也就是duration是0.25秒。
##### autoreverses
	动画是否在播放完成后倒叙播放。默认是NO,当设置YES，动画从A到B然后从B到A，这意味着动画执行了两次。
##### repeatCount
	动画重复次数，可以是分数，当设置浮分数，重复执行动画的部分，比如设置0.5，那么重复执行动画的一半。
##### repeatDuration
	和repeat count类似，但很少用到的就是repeat duration了。它将会根据给定的一个duration简单的重复动画。经过一个repeat duration时，如果它小于动画的duration，那么动画就会提前结束（repeat duration之后结束）。比如动画时间是1.5,repeat duration是2.0，那么执行1.5秒动画后，再重复执行0.5.
	
#### 相对时间

每次讨论到Core Animation，时间都是相对的，每个动画都有它自己描述的时间，可以独立地加速，延时或者偏移。

##### beginTime
	指定了动画开始之前的的延迟时间。这里的延迟从动画添加到可见图层的那一刻开始测量，默认是0（就是说动画会立刻执行）。
##### speed
	是时间的倍数，默认1.0，减少它会减慢图层/动画的时间，增加它会加快速度。大于0，动画正常播放，小于零，动画倒叙播放。如果2.0的速度，那么对于一个duration为1的动画，实际上在0.5秒的时候就已经完成了。
##### timeOffset
	增加beginTime导致的延迟动画不同，增加timeOffset只是让动画快进到某一点执行。设置timeOffset为0.5意味着动画将从一半的地方开始，到动画的末尾结束，然后动画跳到并执行0.5动画。这看起来有点像我们把动画的0.5s切下来，然后放到最后.
	
##### fillMode

	使用动画开始或结束的值用来填充开始之前和结束之后的时间。
	
### 层级关系时间

图层时间实现的原理和坐标系的实现一样，他具有两个特点

* 图层时间是相对的，图层的时间是一个相对于父图层的时间测量。对图层调整时间将会影响到它本身和子图层的动画，但不会影响到父图层

* 所有的动画都被按照层级组合。对CALayer或者CAGroupAnimation调整duration和repeatCount/repeatDuration属性并不会影响到子动画。但是beginTime，timeOffset和speed属性将会影响到子动画。然而在层级关系中，beginTime指定了父图层开始动画（或者组合关系中的父动画）和对象将要开始自己动画之间的偏移

### 全局时间和本地时间
CoreAnimation有一个全局时间的概念。通过下面函数可以获得

	CFTimeInterval time = CACurrentMediaTime();
	
这个时间在设备上所有进程都是全局的--但是在不同设备上并不是全局的--不过这已经足够对动画的参考点提供便利。

每个CALayer和CAAnimation实例都有自己本地时间的概念，是根据父图层/动画层级关系中的beginTime，timeOffset和speed属性计算。就和转换不同图层之间坐标关系一样，CALayer同样也提供了方法来转换不同图层之间的本地时间。如下：
	
	- (CFTimeInterval)convertTime:(CFTimeInterval)t fromLayer:(CALayer *)l; 	
	- (CFTimeInterval)convertTime:(CFTimeInterval)t toLayer:(CALayer *)l;
	
	
### 参看文献
[CAMediaTiming Protocol Reference](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAMediaTiming_protocol/#//apple_ref/doc/constant_group/Fill_Modes)
[Controlling Animation Timing](http://ronnqvi.st/controlling-animation-timing/)
