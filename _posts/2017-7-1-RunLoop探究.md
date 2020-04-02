# RunLoop 探究

RunLoop  简单来说就是一个事件驱动的大循环，如下代码表示

	int main(int argc, char * argv[]) {
     while (AppIsRunning) {
          id whoWakesMe = SleepForWakingUp();
          id event = GetEvent(whoWakesMe);
          HandleEvent(event);
     }
     return 0;

RunLoop 设计目的及优点:

1: 大循环维护线程生命周期，保证线程有任务时处于忙碌，无任务保持睡眠；节省机器资源

2: 抽象消息处理模式，提供高明的事件驱动处理方式


### RunLoopRef构造
	
	
		
	// mode数据结构
	struct __CFRunLoopRun {
 
	    CFMutableSetRef _commonModes;
	     // Set<CFStringRef>
	    CFMutableSetRef _commonModeItems; 
	    // Set<Source/Observer/Timer>
	    CFRunLoopModeRef _currentMode; 
	    // Current Runloop Mode
	    CFMutableSetRef _modes; 
	    // Set<CFRunLoopModeRef>
	    ...
	};	

	runloop数据结构
	struct __CFRunLoopMode {
   
    	CFStringRef _name; //ModeName
    	CFMutableSetRef _sources0;
    	CFMutableSetRef _sources1;
    	CFMutableArrayRef _observers;
    	CFMutableArrayRef _timers;
    	CFIndex _observerMask; //Mode状态
		...
	};


####  创建与退出
* 主线程自动创建，子线程默认不创建
* RunLoop退出条件: app退出；线程关闭；设置最大时间到期；modeItem为空
* 同一时间只能在一个mode，切换mode需要退出runloop重新进入
* 同一个Items(source,timer, Observer)可以添加不同model;mode被标记到commonModes


#### RunLoop 常见Mode

* kCFRunLoopDefaultMode: 默认 mode，通常主线程在这个 Mode 下运行。
* UITrackingRunLoopMode: 追踪mode，保证Scrollview滑动顺畅不受其他 mode 影响。
* UIInitializationRunLoopMode: 启动程序后的过渡mode，启动完成后就不再使用
* GSEventReceiveRunLoopMode: Graphic相关事件的mode，通常用不到
* kCFRunLoopCommonModes: 占位mode，作为标记DefaultMode和CommonMode用

 

#### RunLoop 事件源

* Input Source 
	* Port-Based Sources:与内核端口相关 (source1)
	* Custom Input Sources(source0)
	* Cocoa Perform Selector (为数据类型定义，实测Timer Source)
	
* Timer Source

* Run Loop Observers

source1 主要用于通过内核和其他线程发送的消息，主动唤醒runloop；
source0 主要处理event事件，标记处理，然后手动唤醒runloop;

Timer Source : NSTimer 注册到 RunLoop 后，RunLoop 会为其重复的时间点注册好事件回调；需要注意的是为了节省资源，runloop不能满足精确回调；一旦错过，本次回调丢弃；

Observers : 主要用于观察runloop如下状态
	
	typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
		    kCFRunLoopEntry = (1UL << 0),
		    kCFRunLoopBeforeTimers = (1UL << 1),
		    kCFRunLoopBeforeSources = (1UL << 2),
		    kCFRunLoopBeforeWaiting = (1UL << 5),
		    kCFRunLoopAfterWaiting = (1UL << 6),
		    kCFRunLoopExit = (1UL << 7),
		    kCFRunLoopAllActivities = 0x0FFFFFFFU
};

其中诸如自动释放池、UI渲染刷新、手势识别都是苹果注册Observers，监控Runloop状态实现的;

### RunLoop 事件序列

runloop代码实现如下:

	void CFRunLoopRun(void) {	
		    int32_t result;    
		    do {
				result = CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
				CHECK_FOR_FORK();
	        
	    	} while (kCFRunLoopRunStopped != result && kCFRunLoopRunFinished != result);
	}



CFRunLoopRunSpecific实现
	
	SInt32 CFRunLoopRunSpecific(CFRunLoopRef rl, CFStringRef modeName, CFTimeInterval seconds, Boolean returnAfterSourceHandled) {    
		
		// 0.1: 根据name获取currentMode
	    CFRunLoopModeRef currentMode = __CFRunLoopFindMode(rl, modeName, false);
	    
	    // 0.2: 如果currentMode为空，或者model没有ource/timer/observer
	    if (__CFRunLoopModeIsEmpty(rl, currentMode, rl->_currentMode))	return;
	   
	   
	  		 // 1.1 通知 Observers,即将进入RunLoop ，创建内存池
		if (currentMode->_observerMask & kCFRunLoopEntry ) __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopEntry);
		
		__CFRunLoopRun(rl, currentMode, seconds, returnAfterSourceHandled, previousMode) {
			
			// 1.2 通知 Observers Timer 回调
			__CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeTimers);
			
			// 1.3 通知 Observers Source0(非基于port) 回调
			__CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeSources);
			
			// 1.4 处理被加入的block中符合当前mode的block			__CFRunLoopDoBlocks(runloop, currentMode);
			
			// 1.5 触发source0(非基于port)回调
			sourceHandledThisLoop = __CFRunLoopDoSources0(runloop, currentMode, stopAfterHandle);
			
			// 1.6 触发了source0回调，再次处理加入的block(提高source0响应速度？)
			if (sourceHandledThisLoop) {
				__CFRunLoopDoBlocks(rl, rlm);
			}

			// 1.7 如果有 Source1 (基于port) 处于ready状态和等待回调状态，goto 2.0步奏
			
			if (__Source0DidDispatchPortLastTime) {
                Boolean hasMsg = __CFRunLoopServiceMachPort(dispatchPort, &msg)
                if (hasMsg) goto handle_msg;
            }

			// 1.8 没有消息处理，runloop即将进入sleep（OB释放并新建内存）
			__CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeWaiting)
		
			// 1.9 休眠等待一下信号激活 
					port-base 信号
					timer 回调
					runloop timeout
					被手动唤醒
					
			mach_msg(msg, MACH_RCV_MSG, port); 
	
			 // 1.10. 被唤醒，通知 Observers: RunLoop 的线程刚刚被唤醒了。
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopAfterWaiting);
            
            // 2.0 处理信号
           handle_msg:
           
           		// 2.1 如果是消息是timer，触发timer回调； 触发时间太早，丢弃当前触发
           		__CFRunLoopDoTimers(runloop, currentMode, mach_absolute_time())
           	
     		      	// 2.2 如果消息是dispatch 到main_quene的block，执行
					__CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(msg);

			
					// 2.3 如果是消息是source1，处理这个事件
					__CFRunLoopDoSource1(runloop, currentMode, source1, msg);
			// 执行加入到Loop的block
            __CFRunLoopDoBlocks(runloop, currentMode);
            
				
			
			 // 执行加入到Loop的block
            __CFRunLoopDoBlocks(runloop, currentMode);
 
 
            // 3.1 如果处理事件完毕、Runloop timeout、被外包终止，itmes为空,设置while参数退出Runloop
            
					
		}
		
	}


简化流程如下图: 

![Alt text](https://github.com/Ambtion/ambtion.github.io/blob/master/imageSource/runloop/runLoopSource.png?raw=ture)


# 参看文献

[官方文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)

[Runloop源码](https://opensource.apple.com/source/CF/CF-1151.16/CFRunLoop.c)

[Runloop 深入浅出，综合解答
](https://blog.csdn.net/hherima/article/details/51746125)

[iOS实时卡顿检测-RunLoop](https://www.jianshu.com/p/d0aab0eb8ce4)


 
