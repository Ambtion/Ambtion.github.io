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


####  duration

	对象最基本的持续时间
	
这个基本属性和其他属性的结合共同决定动画的时间持续时间。在其他的属性的是默认值的前提下，一般duration的大小会是动画的持续时间

##### autoreverses
	动画是否在播放完成后倒叙播放。默认是NO,当设置YES，动画从A到B然后从B到A，这意味着动画执行了两次。
#### repeatCount
	动画重复次数，可以是分数，当设置浮分数，重复执行动画的部分，比如设置0.5，那么重复执行动画的一半。
	
#### repeatDuration
	和repeat count类似，但很少用到的就是repeat duration了。它将会根据给定的一个duration简单的重复动画。经过一个repeat duration时，如果它小于动画的duration，那么动画就会提前结束（repeat duration之后结束）。比如动画时间是1.5,repeat duration是2.0，那么执行1.5秒动画后，再重复执行0.5.
	
#### beginTime
	

	


 
	
 




