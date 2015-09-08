## 动画缓冲

我们知道动画的运行过程其实是有速率的产生的，理想情况下动画过程没有任何阻力前提（有点像高中的物理试题），运动过程是匀速的。这种恒定速度的动画，是最简单实现的，同时也是最不真实的动画。

现实生活中的任何一个物体都会在运动中加速或者减速，为了能够控制动画的加速和减速，Core Animation内嵌了一系列标准函数提供给我们使用。

### CAMediaTimingFunction

在CAAnimaiton抽象类中，有一个CAMediaTimingFunction的属性，他的定义如下：

	A timing function defining the pacing of the animation. Defaults to
	nil indicating linear pacing.
	@property(strong) CAMediaTimingFunction *timingFunction;
	
同样也可以使用CATransaction的+setAnimationTimingFunction:方法来控制动画运动过程中的速度。

CAMediaTimingFunction主要提供了两种初始化方法

	+ (instancetype)functionWithName:(NSString *)name;
	+ (instancetype)functionWithControlPoints:(float)c1x :(float)c1y :(float)c2x :(float)c2y;


值得注意的是当使用CAKeyframeAnimation控制关键帧实现动画的时候，CAKeyframeAnimation有一个NSArray类型的timingFunctions，我们可以用它来对每次动画的步骤指定不同的计时函数，但是指定函数的个数一定要等于keyframes数组的元素个数减一，因为它是描述每一帧之间动画速度的函数。

### 系统提供的速度函数
	
使用CAMediaTimingFunction最简单的方法就是使用functionWithName：,这个一个系统封装好的初始化函数，在这个函数中苹果提供了几种常用的速度轨迹
	
	kCAMediaTimingFunctionLinear 
	kCAMediaTimingFunctionEaseIn 
	kCAMediaTimingFunctionEaseOut 
	kCAMediaTimingFunctionEaseInEaseOut
	
* kCAMediaTimingFunctionLinear 常量创建了一个线性的计时函数。同时也是timingFunction的默认值
* kCAMediaTimingFunctionEaseIn 常量创建了一个慢慢加速然后突然停止的方法
* kCAMediaTimingFunctionEaseOut常量创建了一个突然加速然后慢慢减速停止的方法
* kCAMediaTimingFunctionEaseInEaseOut常量创建了一个慢慢加速然后再慢慢减速的过程.
*  
### 自定义速度函数	

除了+functionWithName:之外，CAMediaTimingFunction同样有另一个构造函数，一个有四个浮点参数的+functionWithControlPoints::::

CAMediaTimingFunction函数的主要原则在于它把输入的时间转换成起点和终点之间成比例的改变。



[](http://cubic-bezier.com/)
[](http://blog.csdn.net/likendsl/article/details/7852658)	
### 参考文档

[CAMediaTimingFunction](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/CAMediaTimingFunction_class/)

				

