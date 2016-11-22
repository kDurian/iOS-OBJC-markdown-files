# Block，面向对象中的异类

### 前言

引用苹果官方文档中 [Working with Blocks](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html) 中的一段话：

> Blocks are a language-level feature added to C, Objective-C and C++, which allow you to create distinct segments of code that can be passed around to methods or functions as if they were values. Blocks are Objective-C objects, which means they can be added to collections like `NSArray` or `NSDictionary`. They also have the ability to capture values from the enclosing scope, making them similar to *closures* or *lambdas* in other programming languages.

* block 是以 Objective-C 语言层面的功能被添加进来的
* 可以将 block 传递给方法和函数，好像传参一样
* block 是 Objective-C 中的对象
* block 能从当前的作用域内捕获值



### Block 语法

#### 定义 Block

在 Objective-C 中 ，block 的语法很另类，新手看了可能会抓狂。我就抓过，直冒冷汗那种。不信你~~来抓一下~~，来看一下，👇就是定义简单 block 的语法：

```objective-c
^{
     NSLog(@"It's a block, are you scared?");
 }
```

😓  花括号还好理解，但 ' ^ '  。。。显然这里不能读作异或。其实，它有很多中文名字，参考维基百科 [脱字符](https://zh.wikipedia.org/wiki/%E8%84%B1%E5%AD%97%E7%AC%A6) 。我们不妨就以它的英文名称呼它， caret symbol 。

#### 使用 Block

当我们在程序中只定义 block 会怎样呢？

```objective-c
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        ^{											// ⚠️ Expression result unused
            NSLog(@"It's a block, are you scared?");
        };
    }
    return 0;
}
```

非常遗憾，会警告，表达式结果没有使用。下面的代码同样会报这个错误：

```objective-c
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
      
		[[NSObject alloc] init];  // ⚠️ Expression result unused
      
    }
    return 0;
}
```

也许，我们可以尝试用一个变量来引用它：

```objective-c
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
      
		void (^block)();  // variable
      
		block = ^{
			NSLog(@"It's a block, are you scared?");
		};
    }
    return 0;
}
```

就像：

```objective-c
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
      
      	NSObject *obj; // variable
      
		obj = [[NSObject alloc] init];  
    }
    return 0;
}
```

它们有点相似。对，正如前言所说，`Block 是 Objective-C 中的对象` ，我们会在后面拆穿 block 的真面目。在那之前，我们该想想如何让 block 内的代码执行了，呵呵，非常简单，变量名后面加对圆括号：

```objective-c
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
      
		void (^block)();  // variable
      
		block = ^{
			NSLog(@"It's a block, are you scared?");
		};
      
      	block();
    }
    return 0;
}
```

结果如下：

`It's a block, are you scared?`	



### 拆穿 Block

##### 祭出 clang -rewrite-objc

`main.m`

```objective-c
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
		void (^block)();
		block = ^{
			NSLog(@"It's a block, are you scared?");
		};
      	block();
    }
    return 0;
}
```

执行 `clang -rewrite-objc main.m` ，将其重写为 C 代码，去掉干扰信息后，是下面这样的：

```c
struct __block_impl {
    void *isa;
    int Flags;
    int Reserved;
    void *FuncPtr;
};

struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
        impl.isa = &_NSConcreteStackBlock;
        impl.Flags = flags;
        impl.FuncPtr = fp;
        Desc = desc;
    }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    printf("It's block. Are you scared?");
}

static struct __main_block_desc_0 {
    size_t reserved;
    size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};

// main 
int main(int argc, const char * argv[]) {
  
        void (*block)(); 												// 1

        block = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));										       // 2

        ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);															   // 3
        
    return 0;
}
```



`main` 函数中的第二行：

去掉干扰信息，简化如下：

```c++
struct __main_block_impl_0 tmp = __main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA);

struct __main_block_impl_0 *block = &tmp;
```

分析：在栈上创建并初始化一个 __main_block_impl\_0 结构体对象，然后用 block 变量指向它。



`main` 函数中的第三行：

```
(*block->impl.FuncPtr)(block);
```

分析：执行函数指针指向的函数，并将 block 作为参数。



## 再谈 block 

你也许认为 block 语法很特殊，其实，对于编译器它就是普通的 C 源码。支持 block 的编译器将 block 源码转化为 标准 C 源码，然后正常编译。

这仅仅是概念上的解释。事实上，编译器从没有生成人类可读的源代码。但是，clang (LLVM 编译器) 有生成人类可读代码的功能。

这节我们通过转化的源代码，对 block 的实现方式一探究竟。

### 转化源码

`clang -rewrite-objc ` 可以将带有 block 语法的源码转化成标准 C++ 源代码。尽管生成的是 C++ 风格代码，但除了使用结构体构造器之外，其他的都是 C 语言风格。

```objective-c
int main() {
	void (^block)() = ^{printf("Block\n");};
    block();
  	return 0;
}
```

👆的代码将被转换成👇

```c++
struct __block_impl {
	void *isa;
  	int Flags;
  	int Reserved;
  	void *FuncPtr;
};

struct __main_block_impl_0 {
	struct __block_impl impl;
  	struct __main_block_desc_0 *Desc;
  
  	__mian_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    	impl.isa = &_NSConcreteStackBlock;
      	impl.Flags = flags;
      	impl.FuncPtr = fp;
      	Desc = desc;
  	}
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
 	printf("Block\n");
}

static struct __main_block_desc_0 {
	unsigned long reserved;
  	unsigned long Block_size;
}__main_block_desc_0_DATA = {
	0,
  	sizeof(struct __main_block_impl_0)
};

int main() {
 	void (*blk)(void) = (void (*)(void))&__main_block_impl_0((void *)__main_block_func_0, 	  						&__main_block_desc_0_DATA);
  
 	((void (*)(struct __block_impl *))((struct __block_impl *)blk)->FuncPtr)((struct 		__block_impl *)blk);
  
 	return 0;
} 
```

`main` 函数中的第二行：

去掉干扰信息，简化如下：

```c++
struct __main_block_impl_0 tmp = __main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA);

struct __main_block_impl_0 *block = &tmp;
```

分析：在栈上创建并初始化一个 __main_block_impl\_0 结构体对象，然后用 block 变量指向它。



`main` 函数中的第三行：

```
(*block->impl.FuncPtr)(block);
```

分析：执行函数指针指向的函数，并将 block 作为参数。



### 捕获自动变量

```c
int main() {
	int dmy = 256;
  	int val = 10;
  	const char *fmt = "val = %d\n";
  	void (^blk)(void) = ^{
      	printf(fmt, val);
  	};
  	return 0;
}
```

`clang -rewrite-objc` 

```c++
struct __main_block_impl_0 {
	void *isa;
  	int Flags;
  	int Reserved;
  	void *FuncPtr;
  	struct __main_block_desc_0 *Desc;
  	const char *fmt;
  	int val;

  	__main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, const char *_fmt, int _val, int flags=0) : fmt(_fmt), val(_val) {
      	impl.isa = _NSÇoncreteStackBlock;
      	impl.Flags = flags;
      	impl.FuncPtr = fp;
      	Desc = desc;
  	}
}

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
	const char *fmt = __cself->fmt;
  	int val = __cself->val;
  	printf(fmt, val);
}

int main() {
  	int dmy = 256;
  	int val = 10;
  	const char *fmt = "val = %d\n";
  	void (*blk)(void) = &__main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA, fmt, val);
}
```

注意，只有被 Block 使用的变量被捕获了。在这个例子中，"dmy" 没有用到，也就没有被添加到 __main_block_impl\_0 中。



###  可写变量

这节我们展示如何在 block 中写变量。

先看个例子：

```c
int val = 0;
void (^blk)() = ^{
  	val = 1;
};
```

这在编译器中会报错：`error: variable is not assignable (missing __block type specifier)` 

那我们确实想在 block 中更改捕获的自动变量，怎么办。不要慌，有两种选择：

* 使用其它类型的变量
* 使用 __block  说明符

#### 静态 或 全局变量

* static variables
* static global variables
* global variables

Block 字面的匿名函数部分可以简单的转化成 C 函数。在转化的函数中，静态全局变量 和 全局变量都是可以存取的。

但静态变量不同，因为转化的函数是在定义 block 的函数外声明的，它无法对静态变量进行访问。我们可以测试一下：

```c
int global_var = 1;
static int static_global_var = 2;

int main() {
  	static int static_var = 3;
  	void (^blk)(void) = ^{
    	global_var *= 1;
      	static_global_var *= 2;
      	static_var *= 3;
  	};
  	return 0;
}
```

`clang -rewrite-objc ` 摘取关键信息：

```c
struct __main_block_impl_0 {
	...
    int *static_var;
  	__main_block_impl_0(..., _static_var) {
      	static_var = _static_var;
  	}
};

static void __main_block_func_0(... *__cself) {
  	global_var *= 1;			
  	static_global_var *= 2;		
  	
  	int *static_var = __cself->static_var;
  	(*static_var) *= 3;
}

int global_var = 1;
static int static_global_var = 2;

int main() {
	static int static_var = 3;  // data section
  	blk = __main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA, &static_var);
  	
}
```



#### __block 说明符

```c
__block int val = 10;
void (^blk)(void) = ^{val = 1;};
```

`clang -rewrite-objc `

```c
struct __Block_byref_val_0 {
	void *__isa;
  	__Block_byref_val_0 *__forwarding;
  	int __flags;
  	int __size;
  	int val;
};

struct __main_block_impl_0 {
	...
    struct __Block_byref_val_0 *val;
  	__main_block_impl_0(..., __Block_byref_val_0 *_val)  {
      ...
      val = _val->forwarding; // isEqualTo: val = _val;
  	}
};

static void __main_block_func_0(... *__cself) {
  	__Block_byref_val_0 *val = __cself->val;
  	(val->__forwarding->val) = 1;
  
  	isEqualTo:
  	__cself->val->__forwarding->val = 1;
}

int main() {
	struct __Block_byref_val_0 val = {		// on the stack
      	0,
      	&val;  // pointer to self
      	0,
      	sizeof(__Block_byref_val_0);
      	10;
	}; 
  
  	blk = &__main_block_impl_0(..., &val);
}
```

当自动变量前面加上 __block 修饰符，源码变得大了很多。下面，我们讨论为什么多了一个 \_\_block 修饰符就必须要这么多的代码。

在原始源码中，我们是这样使用 __block 修饰符：

```c
__block int val = 10;
```

然后被转换成：

```c
__Block_byref_val_0 val = {
	0,
  	&val
    0,
  	sizeof(__Block_byref_val_0),
  	10
};
```

出人意料的是，它被转换成一个结构体实例，也在栈上，就像 Block 一样。同时，原始自动变量的值被存储在新的结构体实例的成员变量上。

Block 中，对 __block 变量赋值是如何工作的呢？

```c
^{val = 1;};
```

这行代码被转换成下面：

```c
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
	__Block_byref_val_0 *val = __cself->val;
  	(val->forwarding->val) = 1;
}
```

我们知道，在 Block 中为一个静态变量赋值，使用的是指向静态变量的指针。然而，为了给一个 __block 变量赋值，会更加复杂。代表 Block 的 *\_\_main_block_impl\_0* 结构体实例中有一个指针变量，指向代表 \_\_block 变量的 *\_\_Block_byref_val\_0* 结构体实例，如下图中`1` 号箭头。

而且，`__Block_byref_val_0` 结构体实例有一个指向 `__Block_byref_val_0` 结构体实例的指针成员变量`__forwarding` ，如下图中 `2` 号箭头。

![](https://ooo.0o0.ooo/2016/11/20/58322aee97aa7.jpeg)

```c
void (^blk)(void) = ^{val = 1;};   // 这句会自动调用 _Block_copy 
```

![](https://ooo.0o0.ooo/2016/11/20/5832277d3cf1c.jpeg)



#### 存储 Block 的内存段

在之前的某个小节中，我们知道 Block 是 Objective-C 对象。其 isa 指针指向的是其所属的类，共有一下三种：

* _NSConcreteStackBlcok
* _NSConcreteGlobalBlock
* _NSConcreteMallocBlock

 'stack'、 'global'、 'malloc' 分别出了该对象所在的内存区域，如下图：

![](https://ooo.0o0.ooo/2016/11/20/5831affad73e2.jpeg)



#### 捕获对象

之前举的例子都是整型变量，接下来，我们看一下在 Block 中使用对象会发生什么。

```c
- (void)testBlock {
	typedef void(^blk_t)(id obj);
	blk_t blk;
	{
		NSObject *obj = [NSObject new];
    	data.obj = obj;
        
    	id array = [[NSMutableArray alloc] init];
    	blk = [^(id obj){
            		[array addObject:obj];
            		NSLog(@"array count: %lu", (unsigned long)[array count]);
        		}  copy];   
	};
	blk([[NSObject alloc] init]);
	blk([[NSObject alloc] init]);
	blk([[NSObject alloc] init]);
}

// 输出
array count: 1
array count: 2
array count: 3
```

*Converted source code* :

```c
struct __main_block_impl_0 {
	...
    id __strong array;
  	__main_block_impl_0(..., _array) {
      	array = _array;
      	...
  	}
};

static void __main_block_func_0(... *__cself, id obj) {
  	id __strong array = __cself->array;
  	[array addObject:obj];
  	NSLog(@"array count: %ld", [array count]);
}

static void __main_block_copy_0(struct __main_block_impl_0 *dst, ... *src) {
  	_Block_object_assign(&dst->array, src->array, BLOCK_FIELD_IS_OBJECT);
}

static void __main_block_dispose_0(struct __main_block_impl_0 *src) {
  	_Block_object_dispose(src->array, BLOCK_FIELD_IS_OBJECT);
}

testBlock() {
	blk_t blk;
	{
  		id __strong array = [[NSMutableArray alloc] init];
  		blk = &__main_block_impl_0(..., array, ...);
  		blk = [blk copy];
	}
	(*blk->impl.FuncPtr)(blk, [[NSObject alloc] init]); 
	(*blk->impl.FuncPtr)(blk, [[NSObject alloc] init]); 
	(*blk->impl.FuncPtr)(blk, [[NSObject alloc] init]);
} 
```

可以看到，当我们在 Block 中使用对象时，会将引用该对象的自动变量捕获进来，作为 Block 结构体的成员变量，并且使用 「__strong」 所有权修饰符。

```c
struct __main_block_impl_0 { 
	struct __block_impl impl; 
	struct __main_block_desc_0* Desc; 
 	id __strong array; 
}; 
```

*Clang 4.0 OBJECTIVE-C AUTOMATIC REFERENCE COUNTING (ARC)* [Ownership-qualified fields of structs and unions](http://clang.llvm.org/docs/AutomaticReferenceCounting.html#ownership-qualified-fields-of-structs-and-unions) :

> C does not give us very good language tools for managing the lifetime of aggregates, so it is more convenient to simply forbid them. It is still possible to manage this with a `void*` or an `__unsafe_unretained` object.

虽然编译器不能侦测 C 结构体初始化或销毁的时机来进行内存管理，但是 Objective-C 运行时库可以侦测到 Block 从栈拷贝到堆中，和堆中 Block 销毁的动作。因此，Block 的结构体中有被 *__strong* /  *__weak*  所有权修饰符修饰的变量，编译器可以正确地初始化和释放它们。为此，*__main_block_desc_0* 结构体中的 *copy* / *dispose* 被添加进来，并用函数 ***___main_block_copy_0*** 和 ***__main_block_dispose_0*** 来给它们赋值。

```c
static void __main_block_copy_0(struct __main_block_impl_0 *dst, ... *src) {
  	_Block_object_assign(&dst->array, src->array, BLOCK_FIELD_IS_OBJECT);
}
```

```c
static void __main_block_dispose_0(struct __main_block_impl_0 *src) {
  	_Block_object_dispose(src->array, BLOCK_FIELD_IS_OBJECT);
}
```

*_Block_object_assign* 和 *_Block_object_dispose* 分别相当于对对象作 *retain* 和 *release* 操作。

#### __block  变量 和 对象

__block 说明符可以用来修饰任意类型的自动变量。我们来看看它用来修饰一个 id 类型的自动变量。

```c
__block id obj = [[NSObject alloc] init];
```

等价于：

```c
__block id __strong obj = [[NSObject alloc] init];  //ARC is enabled, __strong is default.
```

`clang -rewrite-objc` 后，变成👇一大坨：

```c
/* struct for __block variable */
struct __Block_byref_obj_0 {
	void *__isa;
  	__Block_byref_obj_0 *__forwarding;
  	int __flags;
  	int __size;
  	void (*__Block_byref_id_object_copy)(void *, void *);
  	void (*__Block_byref_id_object_dispose)(void *);
  	id obj;
};

static void __Block_byref_id_object_copy_131(void *dst, void *src) {
 _Block_object_assign((char*)dst + 40, *(void * *) ((char*)src + 40), 131);
}
static void __Block_byref_id_object_dispose_131(void *src) {
 _Block_object_dispose(*(void * *) ((char*)src + 40), 131);
}

__Block_byref_obj_0 obj = {
  (void*)0,
  (__Block_byref_obj_0 *)&obj, 
  33554432, 
  sizeof(__Block_byref_obj_0), 
  __Block_byref_id_object_copy_131, 
  __Block_byref_id_object_dispose_131, 
  [[NSObject alloc] init],
};
```

当 Block 捕获自动变量，并且 Block 从栈被拷贝到堆上时，*_Block_object_assign* 函数被调用，因此 Block 获取了被捕获变量引用对象的所有权。当在堆上的 Block 对象释放时，*_Block_object_dispose* 函数被调用来释放被捕获变量指向的对象。

当 id 类型的自动变量被 *__strong* 和 *__block*  修饰时，会发生同样的事情。当 __block 变量从栈上拷贝到堆上时，*\_Block_object_assign* 函数被调用，因此同样在堆上的 Block 获取 _block 变量的所有权。



#### 总结

![](https://ooo.0o0.ooo/2016/11/22/5834578463a06.jpeg)



![](https://ooo.0o0.ooo/2016/11/22/58345785cd5ef.jpeg)

