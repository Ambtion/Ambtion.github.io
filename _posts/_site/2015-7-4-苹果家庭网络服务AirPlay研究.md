## [AirPlay概要](https://www.apple.com/airplay/)  

## 1：什么是AirPlay ？
 苹果用于家庭网络共享数据的协议。

## 2 AirPlay用来干什么 ？
 目前主要iphone,ipad，ipod touch 到Apple TV 已经苹果授权的其他硬件设备的数据传输（PS：以上是官方说法。使用AirSearve(实现AirPlay协议的一个客户端，支持os，ios系统，ios系统需要越狱安装）也可是现实多种苹果设备间的数据传输。

## 3 AirPlay 协议传输数据方式

 照片和幻灯片的投射 （Photos）
 音频留的投射 （Stream audio）
 视频流的投射 （videos）
 屏幕内容的投射 （AirPlay Mirroring）

## 4 AirPlay 在使用的过程中不需要任何配置，仅仅需要在同一局域网环境下就可以放心对方。如何实现的？

  AirPlay的 “Search Searver”服务是基于[零配置联网](http://baike.baidu.com/link?url=JV849r3u0e_8Y80LR4KoCSpmiO8VPxYHNpepo4K6AXfSHtKUyuc7lnk446eLZAf8fn7RGF7WMITqa_PS_DtYr_)协议的。

  接受AirPlay协议的硬件设备会公开两部分内容。在这两部分内容中，其他设备可以找到并且读取一些基本信息，包括设备名字，设备支持的内容类型等。

  参看文献： http://nto.github.io/AirPlay.html

## 5 ios App开发中AirPlay的使用方式 

 目前苹果开放的SDK中，只有当App是用于播放媒体的适合，用户可以在App内显示"AirPlay 设备列表”按钮。
  屏幕镜像接口未找到
  官方给出的Demon如下：

  你可以使用如下代码显示"AirPlay 设备列表”按钮.

     MPVolumeView *volumeView = [ [MPVolumeView alloc] init] ; 
     [view addSubview:volumeView];

 如果你想仅仅显示AirPlay按钮，而不显示音量大小，你可以使用如下代码

     MPVolumeView *volumeView = [ [MPVolumeView alloc] init] ;
     [volumeView setShowsVolumeSlider:NO];
     [volumeView sizeToFit];
     [view addSubview:volumeView];


 PS：只有AirPlay设备列表不为空的适合，按钮才才能显示，当然，越狱系统可以下载响应组件强制显示

 参看文献 [AirPlay Guide](https://developer.apple.com/library/ios/documentation/AudioVideo/Conceptual/AirPlayGuide/EnrichYourAppforAirPlay/EnrichYourAppforAirPlay.html#//apple_ref/doc/uid/TP40011045-CH6-SW1)

## 以上内容如有错误，请大家指出
