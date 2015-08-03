## 前言

上一章中我们学习的CALayer的基本属性，但是Core Animation图层兵不仅仅能作用于图片和颜色。在CALayer的官方介绍中，其中有一个继承体系结构，罗列了继承于CALayer的特殊图层，这些图层针对特殊的应用场景做了进一步的功能扩展，同时也针对各种的特殊场景做了优化。

## CALayer 专用图层

划线上面部分会有详细讲解，下面部分感兴趣请点击链接，查阅官网文档

*  [CAShapeLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAShapeLayer_class/index.html#//apple_ref/occ/cl/CAShapeLayer) 
* [CAGradientLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAGradientLayer_class/index.html#//apple_ref/occ/cl/CAGradientLayer)
* [CATextLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CATextLayer_class/index.html#//apple_ref/occ/cl/CATextLayer)
*  [CAReplicatorLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAReplicatorLayer_class/index.html#//apple_ref/occ/cl/CAReplicatorLayer)
*  [CATiledLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CATiledLayer_class/index.html#//apple_ref/occ/cl/CATiledLayer)
* [CAEmitterLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAEmitterLayer_class/index.html#//apple_ref/occ/cl/CAEmitterLayer)

******

* [CATransformLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CATransformLayer_class/index.html#//apple_ref/occ/cl/CATransformLayer)
*  [AVCaptureVideoPreviewLayer](https://developer.apple.com/library/prerelease/ios/documentation/AVFoundation/Reference/AVCaptureVideoPreviewLayer_Class/index.html#//apple_ref/occ/cl/AVCaptureVideoPreviewLayer)
*  [AVPlayerLayer](https://developer.apple.com/library/prerelease/ios/documentation/AVFoundation/Reference/AVPlayerLayer_Class/index.html#//apple_ref/occ/cl/AVPlayerLayer)
*  [AVSampleBufferDisplayLayer](https://developer.apple.com/library/prerelease/ios/documentation/AVFoundation/Reference/AVSampleBufferDisplayLayer_Class/index.html#//apple_ref/occ/cl/AVSampleBufferDisplayLayer)
*  [AVSynchronizedLayer](https://developer.apple.com/library/prerelease/ios/documentation/AVFoundation/Reference/AVSynchronizedLayer_Class/index.html#//apple_ref/occ/cl/AVSynchronizedLayer)
*  [CAEAGLLayer](https://developer.apple.com/library/prerelease/ios/documentation/QuartzCore/Reference/CAEAGLLayer_Class/index.html#//apple_ref/occ/cl/CAEAGLLayer)
*  [CAMetalLayer](https://developer.apple.com/library/prerelease/ios/documentation/Animation/Reference/CAMetalLayer_Ref/index.html#//apple_ref/occ/cl/CAMetalLayer)
* [CAScrollLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAScrollLayer_class/index.html#//apple_ref/occ/cl/CAScrollLayer)


###  CAShapeLayer

CAShapeLayer是一个通过矢量图形而不是bitmap来绘制的图层子类.使用CALayer有一下几个好处:

* 1 ： 渲染快速。CAShapeLayer使用了硬件加速，绘制同一图形会比用Core Graphics快很多
* 2 ： 高效使用内存。一个CAShapeLayer不需要像普通CALayer一样创建一个寄宿图形，所以无论有多大，都不会占用太多的内存。
* 3 ： 不会被图层边界剪裁掉。一个CAShapeLayer可以在边界之外绘制
* 4 ： CAShapeLayer的绘制是自动抗锯齿的，所以在使用CAShapeLayer的3D动画中，他不会像普通Layer一样出现像素化。然后需要注意的如果我们对CASharpLayer或者父类图层使用了图片处理，那么考虑到性能，CAShapLayer将会光栅化。官方原话如下： __However, certain kinds of image processing operations, such as CoreImage filters, applied to the layer or its ancestors may force rasterization in a local coordinate space__

CAShapeLayer具有一个很特殊特殊属性是path,path以及其他属性（比如颜色，宽度等）可以基本满足我们的图形绘制需求，同时在Core Animatin的框架中我们能够很容易的实现基于“绘制过程”的动画。比如：文字书写过程的动画。

* path  : The path defining the shape to be rendered. Animatable.

### CAGradientLayer
 CAGradientLayer是主要用来生成两种或更多颜色平滑渐变的。相较于Core Graphics的颜色渐变。CAGradientLayer的真正好处在于绘制使用了硬件加速。
 CAGradientLayer的主要熟悉是__colors__用于描述渐变颜色变化
 
 * colors : An array of CGColorRef objects defining the color of each gradient stop. Animatable.
 
### CAReplicatorLayer

CAReplicatorLayer是一个图层容器，主要是为内容图层（subLayers） 创建一系列内容的copy镜像，他支持设定copy数目，copy过程中应用的CATransform3D,颜色变化，值得注意的是这里应用的CATransform3D变化或者颜色变化都只针对上一个copy图层而言。

CAReplicatorLayer 的主要用处是：

* 1： 镜像放射（倒影）
* 2： 跟随动画 （适用于多个一摸一样的元素，比如loadding 框）

在参看文献中给出一篇关于CAReplicatorLayer动画的文章，有兴趣可以阅读下

### CATiledLayer
当我们要绘制一张特别大的图片，比如一个高像素的照片或者是地球表面的详细地图的时候，直接使用UIImage的-imageNamed:方法或者-imageWithContentsOfFile:方法会瞬间吞噬应用较大的内存，甚至引起AppCrash，这显示是不明智的。
其实绘制在ios设备上图片纹理是大小有限的，所有显示在屏幕上的图片最终都会被转化为OpenGL纹理，同时OpenGL有一个最大的纹理尺寸（这个取决于设备型号屏幕的大小）。
CATiledLayer就是专门为载入大图造成的性能问题提供了一个解决方案：将大图分解成小片然后将他们单独按需载入。

##### 什么情况下我们特别需要使用CATiledLayer

* 在ScrollView中绘制一张特别巨大的图片（比如超过2096×2096 pixels）
* 减少巨大图片占用的内存空间
* 使用缓存流觞的绘制一张巨图
* 平滑的缩放巨图
* 在ScrollView中平滑的缩放一些不规则绘制的图形

##### 使用CATiledLayer的优点

* 对UIScrollView进行完美适配，当我们使用UIScroollView展示，CATiledLayer利用缓存保证平移和缩放的流畅和快速，即使缓存没有完成，CATiledLayer也会保证缩放的流畅
* 通过使用简单的算法，以及较少的缓存，加快渲染速度（比使用UIImage / CGImageRef要快）
* 对于加载多块图片，自动绑定闪烁加载效果，优化视觉体验
* 总结一下以上这些优点都是自带的，不需要开发者在代码层级去设置什么，包括缓存的管理。开发成本低



### CATextLayer
当我们想要自定义绘制文本的适合，Core Graphics写入图层是一个合理的方案，但是如果我们想要越过视图，直接在图层上绘制，那么讲会十分的繁琐。幸运的是，Core Animation提供了一个CALayer的子类CATextLayer，它以图层的形式包含了UILabel几乎所有的绘制特性，并且额外提供了一些新的特性。
CATextLayer 相交于UILabel的优点

* 1：CATextLayer渲染速度快。在IOS6及其之前的版本，UILabel是通过webKit实现的，所以当文字多的适合绘制压力很大；CATextLayer则是直接使用Core Text框架实现渲染，并且使用硬件加速，所以使用CATextLayer的渲染速度非常快。
* 2：CATextLayer功能更加丰富。在IOS6之前的UILabel不支持富文本，但是CATextLayer支持富文本，行距等。

###  CAEmitterLayer

* 在iOS 5中，苹果引入了一个新的CALayer子类叫做CAEmitterLayer。CAEmitterLayer是一个高性能的粒子引擎，被用来创建实时例子动画如：烟雾，火，雨等等这些效果.

* CAEmitterLayer看上去像是许多CAEmitterCell的容器，这些CAEmitierCell定义了一个例子效果。你将会为不同的例子效果定义一个或多个CAEmitterCell作为模版，同时CAEmitterLayer负责基于这些模版实例化一个粒子流.
CAEmitterLayer中需要注意的几个属性
* 1： [RenderMode](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAEmitterLayer_class/index.html#//apple_ref/c/data/kCAEmitterLayerUnordered) 控制着在视觉上粒子图片是如何混合的 。其中kCAEmitterLayerAdditive，它实现了这样一个效果：合并例子重叠部分的亮度使得看上去更亮。默认是kCAEmitterLayerUnordered. 

* 2 :birthRate，lifetime和celocity，这些属性在CAEmitterCell中也有。这些属性会以相乘的方式作用在一起，这样你就可以用一个值来加速或者扩大整个例子系统

###  总结
*  本章挑选的讲了一些可能使用到的专用图层。如有兴趣，可以查阅官方CALayer的继承结构，
*  Core Aniamtion动画的关键体是Layer，所以针对Layer的属性动画都有一套基本的实现，后续我会讲述如何__自定义Layer属性实现动画___
*  UIView是对CALayer的基本封装，一般情况下部分CALayer的功能比对于的UIView更加丰富
*  Core Aniamtion 框架对CALayer的实现使用硬件加速，所以一般渲染速度较快
*  使用CALayer的专用图层实现功能，一般都是继承UIView，重载 + (Class)layerClass方法。也可以直接add到UIView的layer容器上，但是使用这种方法要注意隐式动画（后面会讲）


## 参考文献

### CAShapLayer
* [CAshapLayer class](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAShapeLayer_class/index.html#//apple_ref/occ/cl/CAShapeLayer)
* [CAShapeLayer disable antialiasing](http://stackoverflow.com/questions/10122978/cashapelayer-disable-antialiasing)
* [CAShapeLayer as a mask for CALayer: rounded corners are stretched](http://goobbe.com/questions/1951545/cashapelayer-as-a-mask-for-calayer-rounded-corners-are-stretched)
[disable anti-aliasing for UIBezierPath](http://www.4byte.cn/question/1010686/disable-anti-aliasing-for-uibezierpath.html)


###  CAGradientLayer
* [CAGradientLayer class](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAGradientLayer_class/index.html#//apple_ref/occ/instp/CAGradientLayer/colors)
* [Animated progress view with CAGradientLayer](https://nrj.io/animated-progress-view-with-cagradientlayer/)


### CAReplicatorLayer
* [CAReplicatorLayer Class](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAReplicatorLayer_class/index.html#//apple_ref/occ/cl/CAReplicatorLayer)
* [Creating animations with CAReplicatorLayer](http://www.ios-animations-by-emails.com/posts/2015-march)


### CATiledLayer
* [CATiledLayer Class](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CATiledLayer_class/index.html#//apple_ref/occ/cl/CATiledLayer)
* [CATiledLayer: how to use it, how it works, what it does](http://red-glasses.com/index.php/tutorials/catiledlayer-how-to-use-it-how-it-works-what-it-does/)
* [CATiledLayer private header](https://github.com/Ambtion/iOS-Headers/blob/master/iOS8.1/Frameworks/QuartzCore/CATiledLayer.h)
* [Catiledlayer Demon](http://www.cimgf.com/2011/03/01/subduing-catiledlayer/)



