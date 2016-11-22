# Objective-C 引用计数原理

##引用计数如何存储
- 如果当前设备是64位环境并且使用 Objective-C 2.0，那么一些对象会使用`ISA`指针的一部分空间来存储它的引用计数；

- 否则Runtime会使用一张散列表来管理引用计数。



##TaggedPointer
判断当前对象是否在使用 TaggedPointer 是看标志位是否为 1：

```
#if SUPPORT_MSB_TAGGED_POINTERS
#	define TAG_MASK (1ULL << 63)
#else
#define TAG_MASK 1

inline bool
objc_object::isTaggedPointer()
{
#if SUPPORT_TAGGED_POINTERS
	return ((uintptr_t)this & TAG_MASK);
#else
	return false;
#endif
}
```

##isa 指针(NONPOINTER_IST)
用 64bit 存储一个内存地址显然是种浪费，毕竟很少有那么大内存的设备。于是可以优化存储方案，用一部分额外空间存储其他内容。`ISA`指针第一位为1即表示使用优化`isa`指针，这里列出不同架构下的64位环境中`isa`指针结构：

```
union isa_t
{
	isa_t() {}
	isa_t(uintptr_t value) : bits(value) {}
	
	Class cls;
	uintptr_t bits;
	
	__arm64__
	
	struct {
		uintptr_t indexed			: 1;
		uintptr_t has_assoc		: 1;
		uintptr_t has_cxx_dtor	: 1;
		uintptr_t shiftcls		: 30;
		uintptr_t magic			: 9;
		uintptr_t weakly_referenced : 1;
		uintptr_t deallocating	: 1;
		uintptr_t has_sidetable_rc : 1;
		uintptr_t extra_rc		: 19;
	};
	
	__x86_64__
	
	    struct {
        uintptr_t indexed           : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 44; // MACH_VM_MAX_ADDRESS 0x7fffffe00000
        uintptr_t weakly_referenced : 1;
        uintptr_t deallocating      : 1;
        uintptr_t has_sidetable_rc  : 1;
        uintptr_t extra_rc          : 14;
#       define RC_ONE   (1ULL<<50)
#       define RC_HALF  (1ULL<<13)
    };
```

`SUPPORT_NONPOINTER_ISA`用于标记是否支持优化的`isa`指针，其字面含义意思是`isa`的内容不再是类的指针了，而是包含了更多信息，比如引用计数，析构状态，被其他weak变量引用情况。判断方法也是根据设备类型：

```
// Define SUPPORT_NONPOINTER_ISA=1 to enable extra data in the isa field.
#if !__LP64__  ||  TARGET_OS_WIN32  ||  TARGET_IPHONE_SIMULATOR  ||  __x86_64__
#   define SUPPORT_NONPOINTER_ISA 0
#else
#   define SUPPORT_NONPOINTER_ISA 1
#endif
```





# 深入理解TaggedPointer

##前言

在2013年9月，苹果推出了iPhone5s，与此同时，iPhone5s配备了首个采用64位架构的A7双核处理器，为了节省内存和提高执行效率，苹果提出了Tagged Pointer的概念。对于64位程序，引入Tagged Pointer后，相关逻辑能减少一半的内存占用，以及3倍的访问速度提升，100倍的创建、销毁速度提升。本文从Tagged Pointer试图解决的问题入手，带领读者理解Tagged Pointer的实现细节和优势，最后指出了使用时的注意事项。


```
int main(int argc, char * argv[]) {
    @autoreleasepool {
        
        NSNumber *num1 = @1;
        NSNumber *num2 = @2;
        NSNumber *num3 = @3;
        
        NSNumber *num4 = @0xefffffffffffffff;
        
        NSLog(@"%p\n%p\n%p\n%p\n", num1, num2, num3, num4);
        
        
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

```
0xb000000000000012
0xb000000000000022
0xb000000000000032
0x7fd730400730
(lldb) p num4->isa
(Class) $0 = __NSCFNumber
(lldb) p num1->isa
error: Couldn't apply expression side effects : Couldn't dematerialize a result variable: couldn't read its memory
(lldb) 
```

可见，苹果确实是将值直接存储到了指针本身里面。
当8字节可以承载用于表示的数值时，系统就会以Tagged Pointer的方式生成指针，如果8字节承载不了时，则又用以前的方式生成普通的指针。

