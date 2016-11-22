# 深入解析Objc中方法的结构

```c
struct objc_class {
	isa_t isa;
	Class super_class;
	cache_t cache;
	class_data_bits_t bits; // class_rw_t * plus custom rr/alloc flags
	
	class_rw_t *data() {
		return bits.data();
	}
};
```
- `isa`是指向元类的指针
- `super_class`指向当前类的父类
- `cache`用于缓存指针和`vtable`，加速方法的调用
- `bits`就是存储类的方法、属性和遵循的协议等信息的地方

##`class_data_bits_t`结构体

```
struct class_data_bits_t {
	uintptr_t bits;
	
public:

#define FAST_DATA_MASK 0x00007ffffffffff8UL

	class_rw_t * data() {
		return (class_rw_t *)(bits & FAST_DATA_MASK);
	}
	
	bool isSwift() {
		return getBit(FAST_IS_SWIFT);
	}
	
	bool hasDefaultRR() {
		return getBit(FAST_HAS_DEFAULT_RR);
	}
	
	bool requiresRawIsa() {
		return getBit(FAST_REQUIRES_RAW_ISA);
	}
};
```
> x86_64架构，macOS只使用了其中的47位来为对象分配地址。而且由于地址要按字节对齐，所以掩码后三位为0。


将`bits`与`FAST_DATA_MASK`进行位运算，只取其中的`[3, 47]`位转换成`class_rw_t *`返回。
因为`class_rw_t *`指针只存于第`[3, 47]`位，所以可以使用最后三位来存储关于当前类的其他信息：


```
伪代码：
union class_data_bits_t {
	uintptr_t bits;
	
	struct {
		uintptr_t is_swift : 1,
		uintptr_t has_default_rr : 1,
		uintptr_t requires_raw_isa : 1,
		uintptr_t class_rw_t : 44,
		uintprt_t extra : 17
	};
}
```

##`class_rw_t`和`class_ro_t`
ObjC类中的属性、方法还有遵循的协议等信息都保存在`class_rw_t`中：

```
struct class_rw_t {
	uint32_t flags;
	uint32_t version;
	
	const class_ro_t *ro; //常量指针
	
	method_array_t methods;
	property_array_t properties;
	protocol_array_t protocols;
	
	Class firstSubclass;
	Class nextSiblingClass;
	
	char *demangledName;
};

```

> 指针常量：指针值是常量，即指针指向的地址不可以变化，但是指向地址的内容可以变化

<br>
> 常量指针：指针值可以变化，但是指针指向的地址的内容是常量，不可以变化

其中还有一个指向常量的指针`ro`，其中存储了**当前类在编译期就已经确定的属性、方法以及遵循的协议。**

```
struct class_ro_t {
	uint32_t flags;
	uint32_t instanceStart;
	uint32_t instanceSize;
#ifdef __LP64__
	uint32_t reserved;
#endif

	const uint8_t *ivarLayout;
	
	const char *name;
	method_list_t *baseMethodList;
	protocol_list_t *baseProtocols;
	const ivar_list_t *ivars;
	
	const uint8_t *weakIvarLayout;
	property_list_t *baseProperties;
	
	method_list_t *baseMethods() const {
		return baseMethodList;
	}
};
```


##编译后内存中类的结构
因为类在内存中的位置是编译期就确定的，先运行一次代码获取`NyanCat`在内存中的地址。

```
#import <Foundation/Foundation.h>
#import "NyanCat.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Class cls = [NyanCat class];
        NSLog(@"%p", cls);
    }
    return 0;
}

输出：
0x100001270
```
接下来，在整个Objc运行时初始化之前，也就是`_objc_init`方法中加入一个断点：


```
(lldb) p (objc_class *)0x100001270
(objc_class *) $0 = 0x0000000100001270
(lldb) p (class_data_bits_t *)0x100001290 //即bits在objc_class的偏移是32byte
(class_data_bits_t *) $1 = 0x0000000100001290
(lldb) p $1->data()
(class_rw_t *) $2 = 0x00000001000011e0
(lldb) p (class_ro_t *)$2 //将 class_rw_t 强制转化为 class_ro_t
(class_ro_t *) $3 = 0x00000001000011e0
(lldb) p *$3
(class_ro_t) $4 = {
	flags = 
	...
}
```
##总结
1.类在内存中的位置是在编译期间决定的
2.类的方法、属性以及协议在编译期间存放到了『错误』的位置，知道`realizeClass`执行之后，才放到了`class_rw_t`指向的只读区域`class_ro_t`，这样我们就可以在运行时为`class_rw_t`添加方法，也不会影响类的只读结构。
3.在`class_ro_t`中的属性在运行期间就不能改变了，再添加方法时，会修改`class_rw_t`中的`methods`列表，而不是`class_ro_t`中的`baseMethods`，对于方法的添加会在之后的文章中分析。


##realizeClass(Class cls)主要代码
```
static Class realize(Class cls) {
	const class_ro_t *ro;
	class_rw_t *rw;
	Class supercls;
	Class metacls;
	bool isMeta;
	
	ro = (const class_ro_t *)cls->data();
	rw = (class_rw_t *)calloc(sizeof(class_rw_t), 1);
```


