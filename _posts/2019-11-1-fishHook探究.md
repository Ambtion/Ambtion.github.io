# 前言
[fishhook](https://github.com/facebook/fishhook)是fackbook开源的一个用来hook C函数的hook库。需要注意的是fishhook只能hook iOS系统的C函数，自己编写的C函数是无法hook的。

## 原理

App启动时候会去链接很多动态库；如UIKit，UIFoundation等。但是动态库不参与前期静态编译，所以在可执行文件的代码端中，并不包含动态库相关的汇编指令；因此，动态库的函数地址一开始是无法知道的，所以这些函数地址存放在__ DATA,__ la_ symbol_ prt与__ DATA,__ nl_ symbol_prt表中，也就是所谓的PIC(位置无关代码)。fishhook的原理就是遍历__ DATA,__ la_ symbol_ prt与__ DATA,__ nl_ symbol_prt,匹配相同的函数符号（函数名词），从而改变函数实现地址；对于内部c函数 hook，fishhook 不支持，但是我们可以通过内部c函数的地址（symtab中)；

## 逻辑实现

### fihshook 入口函数如下:
	
	int rebind_symbols(struct rebinding rebindings[], size_t rebindings_nel) {
			
		if (!_rebindings_head->next) {
			// 初次调用，注册dyld回调

			// 注册时候，所有已经加载的镜像会回调函数
			// 注册后，新image加载时，调用
			 _dyld_register_func_for_add_image();
		} else {
		
			// 对所有Image执行_rebind_symbols_for_image
			uint32_t c = _dyld_image_count();
			for (uint32_t i = 0; i < c; i++) {
				_rebind_symbols_for_image(_dyld_get_image_header(i), _dyld_get_image_vmaddr_slide(i));
			}
		
		}
	}

### 实现逻辑
_rebind _symbols _for _image 主要实现的逻辑就是更具符号名称，查找到函数实现地址，替换为replace的函数，保留原有的函数地址
大体实现逻辑如下:

* 遍历Load commond，查找到__LINKEDIT，计算Image起始地址
* 获得symtab、strtab、indirect Symbols 地址
* 二次遍历Load commond。查找__la_symbol_prt以及
__nl_symbol_prt,获得函数实现列表indirect_symbol_bindings
* 遍历indirect Symbols，通过symtab与strtab 查找到函数符号，替换indirect_symbol_bindings对应位置函数实现；

### 查找关系

1.indirect Symbols 中存储的是对于函数在symbol中的位置，官方给出如下对应关系：
	
	/* section the indirect symbol table entries, in corresponding order in the
    *  indirect symbol table, start at the index stored in the reserved1 field
    *  of the section structure.
    */
    
也就是讲indirect Symbols + section_t.reserved1作为起始地址，获取的内容为函数在symbol中的index

2.symbol table[index] 获取的是nlist_64结构体，结构体定义如下
	
		struct nlist_64 {
		 	union {
        		uint32_t  n_strx; /* index into the string 				table */
	   	 	} n_un;
		}		

	
nlist_64. n_un. n_strx 记录函数符号在strtab中的地址。

至此，我们可以通过给定的函数名称，找到对应函数；

3.la_symbol_ptr 或 nl_symblo_ptr 中存储的是函数实现地址；与indirect Symbols对应；
通过section.addr + slide(ASLR地址)获得函数表的起始地址indirect_symbol_bindings；
访问indirect_symbol_bindings[index],修改函数实现地址即可；

到此，fishhook完成系统C函数的hook；

## 文献

[fishhook官方文档](https://github.com/facebook/fishhook)

[Mach-O实现static函数的动态调用](https://www.jianshu.com/p/48afe039019c)

[深入剖析iOS动态链接库](https://www.jianshu.com/p/1de663f64c05)

[fishhook源码解析](https://www.jianshu.com/p/54946d7e8d36)


