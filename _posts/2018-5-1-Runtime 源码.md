
## Rumtime 源码分析

Objective-C 是一个动态语言，这意味着它不仅需要一个编译器，也需要一个运行时系统来动态得创建类和对象、进行消息传递和转发。Runtime是基于此目的的一个用C和汇编编写的运行系统；

Runtime系统开源，下载地址点[apple](https://opensource.apple.com/source/objc4/) or [git](https://github.com/opensource-apple/objc4)

## 启动
App 冷启动概况为三大步骤:

1.dyld加载可执行文件、递归加载依赖库
2.Runtime 初始化可执行文件
3.main

其中Runtime主要执行如下步骤

* map_images 解析处理文件内容
* load_images 准备load函数，回调所有类与类别的load方法

### map_images
	
Runtime 加载时，会调用_objc_init函数，向dyld注册三个函数指针。
map_images 主要负责了runtime的初始化工作


	void _objc_init(void) {
	
		 ...
		 _dyld_objc_notify_register(&map_images, load_images, unmap_image);
	}
	
	
	void map_images(unsigned count, const char * const paths[],
           const struct mach_header * const mhdrs[]){
          ...
   		 return map_images_nolock(count, paths, mhdrs);
	}
	
	void  map_images_nolock(unsigned mhCount, const char * const mhPaths[],
                  const struct mach_header * const mhdrs[]){
        	...
        	_read_images(hList, hCount, totalClasses, unoptimizedTotalClasses);
	}
	
	
**_read _images** 函数中完成大量的初始化操作，精简的流程如下

	void _read_images(header_info **hList, uint32_t hCount, int totalClasses, int unoptimizedTotalClasses){
	
		1. 根据当前类数量实例化类存储Hash表
			int namedClassesSize = 
            (isPreoptimized() ? unoptimizedTotalClasses : totalClasses) * 4 / 3;
        gdb_objc_realized_classes =
            NXCreateMapTable(NXStrValueMapPrototype, namedClassesSize);
            
        2. 从编译后的ClassList读取classref_t，添加到gdb_objc_realized_classes (哈希表)
			classref_t *classlist = _getObjc2ClassList(hi, &count);
			for (i = 0; i < count; i++) {
				Class cls = (Class)classlist[i];
				// 如有哈希已有替换最新，无直接添加
         	   Class newCls = readClass(cls, headerIsBundle, headerIsPreoptimized); 
         	 }
            
        3. 将未映射Class和Super Class重映射

        	// 重映射类
			Class *classrefs = _getObjc2ClassRefs(hi, &count);
			for (i = 0; i < count; i++) {
                remapClassRef(&classrefs[i]);
            }
            // 重映射父类
            classrefs = _getObjc2SuperRefs(hi, &count);
            for (i = 0; i < count; i++) {
                remapClassRef(&classrefs[i]);
            }
		
		4. 将所有SEL都注册到SEL的哈希表
			SEL *sels = _getObjc2SelectorRefs(hi, &count);
			for (i = 0; i < count; i++) {
				// sel_cname函数内部就是将SEL强转为常量字符串
				const char *name = sel_cname(sels[i]);
				sels[i] = sel_registerNameNoLock(name, isBundle);
   		      }
		
		5. 读取ProtocolList将所有Protocol都添加到protocol_map 哈希表中，对所有Protocol重映射
			protocol_t **protolist = _getObjc2ProtocolList(hi, &count);
			for (i = 0; i < count; i++) {
				readProtocol(protolist[i], cls, protocol_map, isPreoptimized, isBundle);
			}
	
			protocol_t **protolist = _getObjc2ProtocolRefs(hi, &count);
			for (i = 0; i < count; i++) {
				remapProtocolRef(&protolist[i]);
			}
		
		
		6. 初始化所有非懒加载的类，进行ro、rw操作
			classref_t *classlist = _getObjc2NonlazyClassList(hi, &count);
			for (i = 0; i < count; i++) {
				realizeClass(cls);
			}
			
		7. 处理类和元类的category,存储在哈希表中，class为key，category_list 为value
			category_t **catlist = _getObjc2CategoryList(hi, &count)
			for (i = 0; i < count; i++) {
				addUnattachedCategoryForClass(cat, cls, hi);
				..
				addUnattachedCategoryForClass(cat, cls->ISA(), hi);
			}
			
			
	}
	
### load images

load 类方法调用时机在main函数之前。load 方法由系统调用，且整个生命周期只会调用一次；所以适合在load方法中执行只执行一次的操作；

值得注意的是 **如果一个类通过category重新原有方法，会覆盖原有方法；但是load是个例外，所有Category和原类的load方法都会被执行**


	void load_images(const char *path __unused, const struct mach_header *mh){
		...
		// 将类load按照继承者链的顺序添加到loadable_class中
		// 将category load 添加到loadable_category中
		prepare_load_methods((const headerType *)mh);
		
		// 调用类的load
		// 调用category的load
		// 注意这个顺序 类别的load总要等类load执行完成
		call_load_methods();
		
	}
### initialize	
和load 类似的方法是initialize; initialize 是在第一次调用类方法时，才会调用；这个函数也是runtime主动调用，自己不可调用。值得注意的有两点

* initialize 可被category覆盖
* initialize 可能永远不被调用

向对象发送消息时候，会调用到lookUpImpOrForward 函数；该函数中会判断当前类是否被初始化，如没有则初始化；

	/***********************************************************************
	* realizeClass
	* Performs first-time initialization on class cls, 
	* including allocating its read-write data.
	* Returns the real class structure for the class. 
	* Locking: runtimeLock must be write-locked by the caller
	**********************************************************************/

	IMP lookUpImpOrForward(Class cls, SEL sel, id inst, 
	                       bool initialize, bool cache, bool resolver){
		...
		
		if (!cls->isRealized()) {
		// 初始化，分配空间空间，通过class_ro_t 实现class_rw_t
   	     realizeClass(cls);
	   }
		
		if (initialize  &&  !cls->isInitialized()) {
		如果没有initialize,执行initialize；调用initialize并设置标志位
        _class_initialize (_class_getNonMetaClass(cls, inst));
	}
	
_class_initialize 执行类initialize 方法，并且设置标记位
	
	void _class_initialize(Class cls) {
	
		1. // 递归初始化父类
		supercls = cls->superclass;
		if (supercls  &&  !supercls->isInitialized()) {
			_class_initialize(supercls);   
		}
		
		2. // 通过objc_msgSend()函数调用initialize方法
		callInitialize(cls);
	}	
	
## 运行
在向对象发送消息时，最终lookUpImpOrForward会被调到，这个函数主要有两个作用

* 消息查找：查找到方法执行的IMP并返回
* 消息转发：执行小组转发机制

### 消息查找 & 消息转发
继续看lookUpImpOrForward 源码

		IMP lookUpImpOrForward(Class cls, SEL sel, id inst, 
	                       bool initialize, bool cache, bool resolver){
	                       
			...
	
			1. 缓存查找，有退出
			imp = cache_getImp(cls, sel);
		    if (imp) goto done;
		    
		   2. 当前类方法类别查找，加入缓存并有退出
			Method meth = getMethodNoSuper_nolock(cls, sel);
		   if (meth) {
				log_and_fill_cache(cls, meth->imp, sel, inst, cls);
				imp = meth->imp;
				goto done;
        }
        
			3. 循环异常查找父类的缓存或方法列表，有加入缓存退出
			for (Class curClass = cls->superclass;curClass != nil;curClass = curClass->superclass){
				imp = cache_getImp(curClass, sel);
				if (imp != (IMP)_objc_msgForward_impcache) {
					log_and_fill_cache(cls, imp, sel, inst, curClass);
					goto done;
				} else {
					break;
				}
				
				Method meth = getMethodNoSuper_nolock(curClass, sel);
				if (meth) {
					log_and_fill_cache(cls, meth->imp, sel, inst, curClass);
					imp = meth->imp;
					goto done;
	          	 }
			}

			4.如果没有找到，则尝试动态方法解析
			if (resolver  &&  !triedResolver) {
				// 执行 [cls resolveInstanceMethod:sel] 或者 [nonMetaClass resolveClassMethod:sel]
				_class_resolveMethod(cls, sel, inst);
				triedResolver = YES;
				
				// 跳转到步奏1
				goto retry; 
			}
			
			5. 如果还没有IMP被发现，则进入消息转发阶段
			imp = (IMP)_objc_msgForward_impcache;
			cache_fill(cls, sel, imp, inst);
			
	}


总结:

* 如果IMP为找到，触发动态方法解析，且会触发一次；触发完成，再次执行查找流程；值得注意的是，源码中对于 resolveInstanceMethod | resolveClassMethod 返回值不参与逻辑；但是最好跟进实际情况返回，保持良好习惯
* 消息转发中forwardingTargetForSelector 用于单对象转发
* 消息转发中判断methodSignatureForSelector 用于多对象转发，如实现签名,签名规则点[这里](https://www.jianshu.com/p/e4237de0aedb)，触发forwardInvocation;


## 参考
[oc 类方法和实例方法底层实现](https://www.jianshu.com/p/7ce4c55890e8)
[深入探究SEL,Method,IMP
](https://www.jianshu.com/p/94040934bfdc)
[RuntimeAnalyze](https://github.com/DeveloperErenLiu/RuntimeAnalyze)
[method签名](https://www.jb51.net/article/105280.htm)

