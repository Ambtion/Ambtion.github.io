## 前言

上一章中我们学习的CALayer的基本属性，但是Core Animation图层兵不仅仅能作用于图片和颜色。在CALayer的官方介绍中，其中有一个继承体系结构，罗列了继承于CALayer的特殊图层，这些图层针对特殊的应用场景做了进一步的功能扩展，同时也针对各种的特殊场景做了优化。

## CALayer 专用图层

*  [CAShapeLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAShapeLayer_class/index.html#//apple_ref/occ/cl/CAShapeLayer) 
* [CATextLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CATextLayer_class/index.html#//apple_ref/occ/cl/CATextLayer)
* [CAGradientLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAGradientLayer_class/index.html#//apple_ref/occ/cl/CAGradientLayer)
*  [CAReplicatorLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAReplicatorLayer_class/index.html#//apple_ref/occ/cl/CAReplicatorLayer)
* [CAScrollLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAScrollLayer_class/index.html#//apple_ref/occ/cl/CAScrollLayer)
* [CATransformLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CATransformLayer_class/index.html#//apple_ref/occ/cl/CATransformLayer)
****
*  [AVCaptureVideoPreviewLayer](https://developer.apple.com/library/prerelease/ios/documentation/AVFoundation/Reference/AVCaptureVideoPreviewLayer_Class/index.html#//apple_ref/occ/cl/AVCaptureVideoPreviewLayer)
*  [AVPlayerLayer](https://developer.apple.com/library/prerelease/ios/documentation/AVFoundation/Reference/AVPlayerLayer_Class/index.html#//apple_ref/occ/cl/AVPlayerLayer)
*  [AVSampleBufferDisplayLayer](https://developer.apple.com/library/prerelease/ios/documentation/AVFoundation/Reference/AVSampleBufferDisplayLayer_Class/index.html#//apple_ref/occ/cl/AVSampleBufferDisplayLayer)
*  [AVSynchronizedLayer](https://developer.apple.com/library/prerelease/ios/documentation/AVFoundation/Reference/AVSynchronizedLayer_Class/index.html#//apple_ref/occ/cl/AVSynchronizedLayer)
*  [CAEAGLLayer](https://developer.apple.com/library/prerelease/ios/documentation/QuartzCore/Reference/CAEAGLLayer_Class/index.html#//apple_ref/occ/cl/CAEAGLLayer)
*  [CAEmitterLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAEmitterLayer_class/index.html#//apple_ref/occ/cl/CAEmitterLayer)
*  [CAMetalLayer](https://developer.apple.com/library/prerelease/ios/documentation/Animation/Reference/CAMetalLayer_Ref/index.html#//apple_ref/occ/cl/CAMetalLayer)
*  [CATiledLayer](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CATiledLayer_class/index.html#//apple_ref/occ/cl/CATiledLayer)


###  CAShapeLayer

CAShapeLayer是一个通过矢量图形而不是bitmap来绘制的图层子类.使用CALayer有一下几个好处:

* 1 ： 渲染快速。CAShapeLayer使用了硬件加速，绘制同一图形会比用Core Graphics快很多
* 2 ： 高效使用内存。一个CAShapeLayer不需要像普通CALayer一样创建一个寄宿图形，所以无论有多大，都不会占用太多的内存。
* 3 ： 不会被图层边界剪裁掉。一个CAShapeLayer可以在边界之外绘制
* 4 ： CAShapeLayer的绘制是自动抗锯齿的，所以在使用CAShapeLayer的3D动画中，他不会像普通Layer一样出现像素化。然后需要注意的如果我们对CASharpLayer或者父类图层使用了图片处理，那么考虑到性能，CAShapLayer将会光栅化。官方原话如下： __However, certain kinds of image processing operations, such as CoreImage filters, applied to the layer or its ancestors may force rasterization in a local coordinate space__

CAShapeLayer具有一个很特殊特殊属性是path,path以及其他属性（比如颜色，宽度等）可以基本满足我们的图形绘制需求，同时在Core Animatin的框架中我们能够很容易的实现基于“绘制过程”的动画。比如：文字书写过程的动画。

PS:core Aniamtio动画的关键体是Layer，所以针对Layer的属性动画都有一套基本的实现，后续我会讲述如何__自定义Layer属性实现动画___

### 



## 参考文献

### CAShapLayer
* [CAshapLayer class](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsImaging/Reference/CAShapeLayer_class/index.html#//apple_ref/occ/cl/CAShapeLayer)
* [CAShapeLayer disable antialiasing](http://stackoverflow.com/questions/10122978/cashapelayer-disable-antialiasing)
* [CAShapeLayer as a mask for CALayer: rounded corners are stretched](http://goobbe.com/questions/1951545/cashapelayer-as-a-mask-for-calayer-rounded-corners-are-stretched)
[disable anti-aliasing for UIBezierPath](http://www.4byte.cn/question/1010686/disable-anti-aliasing-for-uibezierpath.html)







