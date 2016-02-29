# Core Location

在移动互联网中`位置服务`就一直处于一个非常重要的位置。苹果对其也尤为重视，在最初的iPhone手机中，Maps.app 和Core Location Api就一直存在。并且，之后的WWDC都在逐步的升级、添加Core Location库的功能。

Core Locaiton库的核心是[`CLLocationManager`](https://developer.apple.com/library/ios/documentation/CoreLocation/Reference/CLLocationManager_Class/)。Core Locaiton库的能力都是基本都是通过CLLocationManager的实例来实现的。

## `定位权限`

在IOS8以前，位置服务的权限是二元的：给与不给。用户可以通过Settings.app中设置决定是否允许用户定位。在IOS8修改了这个问题，添加了更加细化的场景。它把位置服务权限拆分成了 2 个不同的授权。

### IOS8+系统

主要添加了“授权分级”（`使用中授权`和`始终授权`）

* 授权注册

    Info.plist中的”NSLocationUsageDescription拆分为两个不同的关键字`（NSLocationWhenInUseUsageDescription` 和 `NSLocationAlwaysUsageDescription`

*  授权请求函数

		[self.locationManager requestWhenInUseAuthorization];
		[self.locationManager requestAlwaysAuthorization];


值得注意的是IOS8之后，Info.plist中的”NSLocationUsageDescription“是必填的。如果没有添加对应的关键字就调用函数请求授权，那么将不会有任何的弹出提示给用户。（系统将认为App不需要位置授权）

### **IOS9+系统**
* Blue Bar 一直保留

IOS8系统后，App获得`使用中授权`权限，当App进入后台，手机顶部会闪现`Blue Bar`提示用户App正在后台定位，之后`Blue Bar`消失，然而此时App依旧可以后台定位。IOS9针对`Blue Bar`再次优化，即只要在`使用中授权`前提下后台定位，那么`blue Bar`一直保留，以便明确告诉用户App使用定位情况。


* 同一App中允许多个location manager，一些只能在前台定位，另一些可在后台定位（并可随时禁止其后台定位）


IOS9通过allowsBackgroundLocationUpdates(缺少NO)控制后台定位是否打开。设置NO的时候，即使使用后台定位（Background not  Suspended），App也会遵循系统后台App挂起策略（一段时间后App挂起，定位关闭)；


## `定位性能`
   
   尽管苹果对`定位模块`做了很多优化，包括基站缓冲，GPS唤醒策略等等（详细可看iOS定位原理和使用建议）；定位依旧是一个非常消耗电量的操作。所以何时打开定位功能的何时关闭定位尤为重要。同时，官方也给出一些使用定位的建议：

1. 设置`Location Manager`的`pausesLocationUpdatesAutomatically`为YES；从而使`Location Manager`能够在合适的时间暂定定位（关闭硬件定位）
2. 选择`Location manager`的`activityType`的合适场景。针对不同频率的定位场景，苹果定义了多种activityType，比如驾车导航，步行导航。选择合适的`activityType`能使系统合理的调整定位休眠时机，从而减少电量的消耗。
3. 如果使用的是后台定位，可以使用`allowDeferredLocationUpdatesUntilTraveled:timeout`函数推迟事件唤醒时间，从而减少功耗。关`allowDeferredLocationUpdatesUntilTraveled:timeout`的详细用法请参考文献

## `定位服务`

到目前为止，Core Location框架提供了如下的后台定位能力，来提高Core Locaiton的定位能力

1. Location
2. Significant Location Changes
3. CLVisit
4. Regions
5. Beacon

下图是Core Location提供能力对应的Api,以及是否需要background和background能否获得位置的关系表
![APIs and technologies for which an iOS app can specify location updates](https://github.com/Ambtion/ambtion.github.io/blob/master/imageSource/CoreLocation/backLocation.png?raw=ture)
	

#### Significant Location Changes
* `系统级服务`,要求用户`始终定位`的授权。所谓的系统级能力的意思是：即使用户Kill掉App或者App是挂起状态，该能力依旧可以获得用户地理位置事件的回调，从而后他唤醒App（10s，可以通过`beginBackgroundTaskWithName:expirationHandler:`获取更多的后台时间）
* 该服务要求`蜂窝硬件能力`的支持，同时事件频率低于标准定位能力。主要用于获取用户`初始定位信息`和`用户位置信息何时变化`（个人推测通过`蜂窝基站变化`确定用户位置信息变化）
* Demon[地址](https://github.com/Ambtion/SignificantChangeUpdates.git)

#### CLVisit
* `系统级服务`,要求用户`始终定位`的授权
* 该服务比`Significant Location Changes`频率更低。当用户到达一个地方一段何时时间（具体时间多少官方没有给出），系统会分发用户到达某个地方的，这个时间具有一定的延时。
* 由于使用方法比较简单，这里不给出Demon.

#### Regions
* `系统级服务`,要求用户`始终定位`的授权
* 该服务	频率高于`Significant Location Changes`，却低于普通定位。该服务主要用户监控用户`进出`围栏的动作。使用该服务，需要先向系统注册一个围栏（`最大限制20个`），围栏是由中心点（经纬度）、半径（几十米到几千公里，注意最大的半径范围设备有关，可以通过`CLLocationManager`的`maximumRegionMonitoringDistance`获得）组成。
* Demon[地址](https://github.com/Ambtion/FenceDemon.git)

#### Beacon
* `系统级服务`
* 这是一个需要`支持Beacon协议的硬件设备`配合的功能。当用户进入`Beacon设备`的范围内的时候，系统分发事件通知App,获得`Beacon设备`信息。一般用于商场广告的精准推送
 

PS:在IOS8以前，通过Settings.app禁止App后台刷新【文献9】可以阻止用户位置事件的分发。在IOS8+以后，禁止App后台刷新无法可以阻止用户位置事件的分发。

## `参考文献`

1.  [Using Core Location in iOS 4 WWDC 2010](http://asciiwwdc.com/2010/sessions/115)
2.  [What’s New in Core Location WWDC 2013](http://asciiwwdc.com/2013/sessions/307) 
3. [What's New in Core Location WWDC 2015](http://asciiwwdc.com/2015/sessions/714)
4. [AllowsBackgroundLocationUpdates](http://stackoverflow.com/questions/30808192/allowsbackgroundlocationupdates-in-cllocationmanager-in-ios9)
5. [iOS定位原理和使用建议](http://www.2cto.com/kf/201404/289744.html)
6. [Deferring Location Updates While Your App Is in the Background
](https://developer.apple.com/library/prerelease/watchos/documentation/UserExperience/Conceptual/LocationAwarenessPG/CoreLocation/CoreLocation.html#//apple_ref/doc/uid/TP40009497-CH2-SW14)
7. [Location and Maps Programming Guide](https://developer.apple.com/library/prerelease/watchos/documentation/UserExperience/Conceptual/LocationAwarenessPG/CoreLocation/CoreLocation.html#//apple_ref/doc/uid/TP40009497-CH2-SW1)
8. [CLLocationManager Class Reference](https://developer.apple.com/library/ios/documentation/CoreLocation/Reference/CLLocationManager_Class/#//apple_ref/doc/uid/TP40007125-CH3-SW60)
9. [How to Disable Background App Refresh](http://www.gottabemobile.com/2014/10/14/how-to-stop-ios-8-apps-from-refreshing-in-the-background/)
10. [GSM蜂窝基站定位基本原理浅析](http://www.cnblogs.com/magicboy110/archive/2010/12/10/1902741.html)
11. [When startMonitoringSignificantLocationChanges call back](http://stackoverflow.com/questions/8290707/startmonitoringsignificantlocationchanges-lack-of-accuracy)




