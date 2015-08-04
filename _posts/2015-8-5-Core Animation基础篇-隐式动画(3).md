## 隐式动画

前两章我们讨论了Core Animation框架的CALayer的基本属性，已经专用图层，也就是Core Animation除动画以为可以做到的所有事情。但是动画是Core Animation库的非常显著的特性，从这章起我们就开始探讨Core Animation的动画属性。

在Core Animation框架中，动画主要分为两个类型

* 隐式动画
* 显式动画

本章我们主要探讨Core Animation的隐式动画。在理解隐式动画前，我们需要明白一个概念--事务

### 事务

#### 事务是什么

Core Animaiton 基于一个假设，即屏幕上任何东西都是可以（或者可能）动画的。动画并不需要我们打开，相反需要明确的关闭，否则他将一直存在。

当我们修改CALayer的Animatable属性的时候（细心的话，在前面查阅CALayer或者专用图层的时候我们会发现对于CALayer的属性有一个Animatable的描述），他并不会理解在屏幕上显示出来，而是会平滑的过度到新的值。而且这种行为是默认的，不需要额外的操作。

#### 事务如何工作
		
		
	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *layerView;
	@property (nonatomic, weak) IBOutlet CALayer *colorLayer;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
	    //create sublayer
    	self.colorLayer = [CALayer layer];
	    self.colorLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    	self.colorLayer.backgroundColor = [UIColor blueColor].CGColor;
	    //add it to our view
    	[self.layerView.layer addSublayer:self.colorLayer];
	}

	- (IBAction)changeColor
	{
    	//randomize the layer background color
	    CGFloat red = arc4random() / (CGFloat)INT_MAX;
    	CGFloat green = arc4random() / (CGFloat)INT_MAX;
	    CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    	self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;                                                                                       ￼
	}

	@end
	
上述代码执行，我们可以清晰的看到一个平滑的过度动画，这个其实就是隐式动画。之所以叫隐式动画是因为我们并没有任何关于动画的操作。我们仅仅修改了一个属性，然而Core Animation自动决定控制一个动画，保证属性的平滑过度。同样Core Animation支持显示动画，后面探究。

一个动画有三个重要因素

* 对象体，即动画实施的操作体，比如图层的bounds,Transform等（这也是我们前面探索CALayer的原因）
* 动画时间
* 动画行为 

我们修改属性，明确了__对象体__，然而隐式动画又是如何确定动画时间和动画行文这两个重要因素的？实际上动画执行的时间取决于当前事务的设置，动画类型取决于图层行为。

实际上事务是Core Animaiton用来包含一系列属性动画集合的机制，任何用事务制定修改的属性都不会立刻发生变化，而是当事务一旦提交开始一个用一个动画过度到新的属性。

事务是通过[CATransaction](https://developer.apple.com/library/prerelease/mac/documentation/GraphicsImaging/Reference/CATransaction_class/#//apple_ref/occ/clm/CATransaction/flush)类来做管理。这个类的设计有些奇怪，不像你从它的命名预期的那样去管理一个简单的事务，而是管理了一叠你不能访问的事务。CATransaction没有属性或者实例方法，并且也不能用+alloc和-init方法创建它。但是可以用+begin和+commit分别来入栈或者出栈。

任何可以做动画的图层属性都会被添加到栈顶的事务，我们可以通过+setAnimationDuration:方法设置当前事务的动画时间，或者通过+animationDuration方法来获取值（默认0.25秒）。当然我们也可以通过 + setDisableActions: 禁用某个属性的动画行为

Core Aniamtion会在每个Run Loop周期中自动开始一个新的事务(Run Loop是iOS负责收集用户输入，处理定时器或者网络事件并且重新绘制屏幕的东西)，即使你不显式的用[CATransaction begin]开始一次事务，任何在一次Run Loop中改变的属性都会集中起来，然后做一个0.25秒的动画。

在修改使用 +setAnimationDuration:方法设置当前事务的动画时间的时候，一定要主要当前修改不要影响同一事务的其他动画，比如屏幕旋转。避免的最好方法是自己调用[CATransaction begin]开始一次事务开始一个事务。（ps：这种嵌套有点类型内存池的概念）

### 图层行为

通过前面两张的探究，我们知道视图其实是对图层的保证，那么既然图层有隐式动画的属性，视图是否也是如此那？

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *layerView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    //set the color of our layerView backing layer directly
	    self.layerView.layer.backgroundColor = [UIColor blueColor].CGColor;
	}

	- (IBAction)changeColor
	{
    	//begin a new transaction
	    [CATransaction begin];
    	//set the animation duration to 1 second
	    [CATransaction setAnimationDuration:1.0];
    	//randomize the layer background color
	    CGFloat red = arc4random() / (CGFloat)INT_MAX;
    	CGFloat green = arc4random() / (CGFloat)INT_MAX;
	    CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    	self.layerView.layer.backgroundColor = [UIColor 	colorWithRed:red green:green blue:blue alpha:	1.0].CGColor;
    	//commit the transaction
	    [CATransaction commit];
	}
	
	@end
	
当我们允许上述代码，发现视图是直接发生改变，而不是允许一个0.25秒的隐式动画。发生了什么呢？隐式动画好像被UIView关联图层给禁用了。

试想一下，如果UIView的属性都有动画特性的话，那么无论在什么时候修改它，我们都应该能注意到的。所以，如果说UIKit建立在Core Animation（默认对所有东西都做动画）之上，那么隐式动画是如何被UIKit禁用掉呢？

我们知道Core Animation通常对CALayer的所有属性（可动画的属性）做动画，但是UIView把它关联的图层的这个特性关闭了。为了更好说明这一点，我们需要知道隐式动画是如何实现的。

我们把改变属性时CALayer自动应用的动画称作行为，当CALayer的属性被修改时候，它会调用-actionForKey:方法，传递属性的名称。剩下的操作都在CALayer的头文件中有详细的说明，实质上是如下几步：

* 1: 检查是否实现CALayerDelegate协议指定的-actionForLayer:forKey方法。如果有，直接调用并返回结果。
* 2: 如果1没有实现，图层直接从actions属性的字典中读取对应的方法。
* 3: 如果2中没有行为，那么图层从stype属性的字典中读取对应的方法。
* 4: 如果3中没有行为，那么图层将会直接调用定义了每个属性的标准行为的-defaultActionForKey:方法。

所以一轮完整的搜索结束之后，-actionForKey:要么返回空（这种情况下将不会有动画发生），要么是CAAction协议对应的对象，最后CALayer拿这个结果去对先前和当前的值做动画

于是这就解释了UIKit是如何禁用隐式动画的：每个UIView对它关联的图层都扮演了一个委托，并且提供了-actionForLayer:forKey的实现方法。当不在一个动画块的实现中，UIView对所有图层行为返回nil，但是在动画block范围之内，它就返回了一个非空值。

当然返回nil并不是禁用隐式动画唯一的办法，CATransaction有个方法叫做+setDisableActions:，可以用来对所有属性打开或者关闭隐式动画。



### 图层树形状结构-呈现与模型

CALayer的属性行为其实很不正常，因为改变一个图层的属性并没有立刻生效，而是通过一段时间渐变更新。这是怎么做到的呢？

当我们改变一个图层的属性，属性值的确是立刻更新的（如果你读取它的数据，你会发现它的值在你设置它的那一刻就已经生效了），但是屏幕上并没有马上发生改变。这是因为你设置的属性并没有直接调整图层的外观，相反，他只是定义了图层动画结束之后将要变化的外观。












