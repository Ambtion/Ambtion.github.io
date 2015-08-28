## 动画时间
CAAnimation作为IOS核心动画的抽象基类，其中有一个很重要的功能就是实现了一个叫CAMediaTiming的协议。

### CAMediaTiming协议
CAMediaTiming协议定义了在一段动画内用来控制逝去时间的属性的集合，CALayer和CAAnimation都实现了这个协议，所以时间可以被任意基于一个图层或者一段动画的类控制。

CAMediaTiming定义了8个属性。通过这8个属性的组合，准确的控制着动画时间。

* duration
* repeatCount
* repeatDuration
* autoreverses
* beginTime 
* speed
* timeOffset
* fillMode


####  duration
 




