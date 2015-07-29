# CoreAmimation-基础篇-[CALayer](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CALayer_class/)


### CALayer和树形结构

* __CALayer和UIView的最大不同是CALayer无法支持事件交互__
  
  在IOS开发中，所有的视图都是以UIView（Mac是NSView）作为基类派生而来，通过树形结构管理视图显示。UIView可以处理触摸事件，支持2D矩阵映射，支持基与Core Graphics的绘制，同时也能实现一些简答的基本动画。CALayer在概念上和UIView一样，他也是通过树形结构管理视图，支持矩阵映射，支持Core Graphics绘制。同时也支持动画实现。

* __UIView是对CALayer的封装，这种设计主要是为了职责分离，重用代码__
  
  每一个UIView都有一个CALayer的实例，他们的树形结构一一对应。实际上UIView只是的对CALayer进行了一次封装，暴露部分CALayer支持的功能，同时额外的支持事件响应,苹果之所以使用这个种UIView，CALayer对应的设计模式，主要是为了做到职责分离。由于Mac的事件和移动设备的事件全然不同，所以考虑到代码的复用，所以将他们的事件却别分离。

* 视图的层级有四种树形结构

	* UIView视图树
	* CALayer视图树  
	* 呈现树（主要用于动画，后面讲）
	* 渲染树（主要用于动画，后面讲）

### 图层能力

图层不能像视图那样处理触摸事件，那么他能做哪些视图不能做的呢？这里有一些UIView没有暴露出来的CALayer的功能：

* 阴影，圆角，带颜色的边框
* 3D变换
* 非矩形范围
* 透明遮罩
* 多级非线性动画


### 图层的内容属性

* #### contents属性
 	IOS设置一个CGImageRef，Mac OS X 10.6及其以后，你可以设置一个 NSImage，
 需要注意点的是如果一个图层和UIView绑定，尽量避免设置content属性，以免影响视图后续UI更新

* #### contentGravity
	等同于ViewContentModel
	
* ####  contentsScale
	如果contentsScale设置为1.0，将会以每个点1个像素绘制图片，如果设置为2.0，则会以每个点2个像素绘制图片，这就是我们熟知的Retina屏幕  

* #### contentsRect
	CALayer的contentsRect属性允许我们在图层边框里显示寄宿图的一个子域。可以控制显示内容

* #### contentsCenter
	contentsCenter是一个Rect区域，他的指定content的拉伸效果。和局部拉伸图片的函数效果一样。
	
#### 自定义图层内容

* #### UIView

  继承UIView 实现 drawRect: 自定义绘制内容。（__注意：当系统检查到DrawRect被调用的时候，系统会为视图分配一个寄宿图，这个寄宿图的大小等于视图大小乘以contentsScale。所以不要在自定义的视图中写一个空的DrawRect函数__）
* #### CALayer

##### 通过CALayerDelegate协议

   当需要被重绘时，CALayer会请求它的代理给他一个寄宿图来显示。它通过调用下面这个方法做到的:

	-(void)displayLayer:(CALayerCALayer *)layer;（可以直接设置contents）
	
如果代理不实现-displayLayer:方法，CALayer就会转而尝试调用下面这个方法：

	-(void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;
	
##### 通过继承CALayer

继承CALayer可以通过两个方法捕捉系统绘制过程

	- (void)display
	- (void)drawInContext:(CGContextRef)theContext
	
其中display函数主要是用于

* 1：创建一个绘制的上下文
* 2：将draw绘制内容生成一个CGImageRef,然后写寄宿图


值得注意的是display方法会创建一个视图图形上下文不会自动设置UIKit上下文。为了使用UIKit来绘图，所以CALayer绘制值中，我们需要调用UIGraphicsPushContext（）方法指定接收到的上下文为当前上下文。

### 图层的几何属性

####  图层属性

CALayer提供了3个比较重要的属性来确定图层位置，其中frame通过position(center)和bouds计算确定，

* #### frame   
* #### bounds   
* #### position

CALayer还有一个特别重要的几何属性：锚点（anchorPoint）

* #### anchorPoint

 关于锚点我们需要明确几点：
 
###### 1：视图center属性和图层的position属性都指定了anchorPoint相对于父图层的位置
###### 2：图层的anchorPoint通过position来控制它的frame的位置
###### 3：所有的矩阵变化是以锚点为中心点变化的，锚点是所有视图或者图层移动选择的”手柄”
###### 4：锚点的使用单位坐标描述，也就是图层的相对坐标，图层左上角是{0, 0}，右下角是{1, 1}，因此默认坐标是{0.5, 0.5}
 
 和UIView不同的是CALayer处于三维坐标系中，所以CALayer还有另外两个属性，zPosition和anchorPointZ
 
 * #### zPosition
 * #### anchorPointZ
 
 关于z坐标系属性需要明确的一点
 修改图层的z坐标系属性影响的仅仅是图层的展现效果。不影响事件响应链。所以当我们修改z坐标属性，很可能出现一个A视图挡在B视图前面，且都可以响应事件的前提下，B视图反而优先与A视图响应事件。
 
#### 视图坐标转换

CALayer提供了一套不同坐标系下转换坐标的方法：

	- (CGPoint)convertPoint:(CGPoint)point fromLayer:(CALayer *)layer; 
	- (CGPoint)convertPoint:(CGPoint)point toLayer:(CALayer *)layer; 
	- (CGRect)convertRect:(CGRect)rect fromLayer:(CALayer *)layer;
	- (CGRect)convertRect:(CGRect)rect toLayer:(CALayer *)layer;

PS：对应的UIView也有一套封装的方法。函数前缀名字都是convertPoint 或者convertRect。

#### 视图变换
* #### CATransform3D
这里需要明确的几点是：

###### 1：视图矩阵变换是二维空间，图层变换是三维空间。
###### 2：矩阵变换可以拼接，我们仅仅需要调用系统函数即可，不需要太深入理解矩阵计算。
###### 3：在3D变换中有需要理解“[透视投影](http://baike.baidu.com/link?url=weyHf8BG2dzTzSaIUI3fSKD3SherW3T532Iiv8c-HDY3QlqvkDPAYVjHk0SZ3tWl0ovbAWOhaXtYNUhypPGXu_)”的概念CATransform3D的透视效果通过一个矩阵中一个很简单的元素来控制：m34。m34的默认值是0，我们可以通过设置m34为-1.0 / d来应用透视效果，d代表了想象中视角相机和屏幕之间的距离，通常d=500-1000就可。

* #### sublayerTransform
sublayerTransform会自动应用于Layer上的所有子图层，默认是identity transform

### 图层的其他属性
* ##### conrnerRadius 圆角
* ##### shadowOpacity 阴影透明度
* #### shadowColor 阴影颜色
* #### shadowOffset 阴影偏移
* #### shadowRadius 控制阴影边界过度，值越大，边界越模糊
* #### shadowPath通过CGPath自定阴影形状
* #### mask mask是视图的一个子图层，他有图层所有的属性，他自定图层显示的内容



#### PS：本章主要讲述的是CALayer的常见属性，以及和动画关联较大的属性，其他具体属性可以参阅查考文献中的Apple CALayer Class


### 参考文献
[View Programming Guide for iOS](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html#//apple_ref/doc/uid/TP40009503-CH2-SW1)

[Apple CALayer Class](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CALayer_class/#//apple_ref/occ/instp/CALayer/doubleSided)

[CoreAnimation 图层内容](http://www.dreamingwish.com/article/coreanimation-programming-guide-e-the-content-layer.html)

[CoreAniamtion 图层绘制](http://blog.csdn.net/slyfy27/article/details/17505441)
 










	















 
 
 
 


