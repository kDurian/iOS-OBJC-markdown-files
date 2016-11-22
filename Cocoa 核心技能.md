# Cocoa 核心技能



## 协议

[Working with Protocols](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithProtocols/WorkingwithProtocols.html)

### \#协议定义消息传递契约

类的接口声明了该类相关的方法和属性。相比之下，协议是用来声明独立于特定类的方法和属性。

定义协议的基本语法如下：

```objective-c
@protocol ProtocolName
// list of methods and properties
@end
```

协议可以包括实例方法和类方法的声明，属性也行。

举个例子，考虑一个用来显示饼状图的自定义视图类：

![](https://c5.staticflickr.com/9/8047/29614180276_0bf8e72bfd_b.jpg)

为了保证这个视图的可重复利用，所有决定显示的信息应该留给其他对象，数据源对象。这意味着只要和不不同的数据源通信就能使很多个同一视图类显示不同的信息。

饼状图的显示至少需要共有多少块、每块的大小和标题的信息。因此，该饼状图的数据源协议可能像这样：

```objective-c
@protocol XYZPieChartViewDataSource
- (NSUInteger)numberOfSegments;
- (CGFloat)sizeOfSegmentAtIndex:(NSUInteger)segmentIndex;
- (NSString *)titleForSegmentAtIndex:(NSUInteger)segmentIndex;
@end
```

饼状图类的接口需要一个记录数据源对象的属性。这个对象可以是任意类，因此这个基本的属性应该是 `id` 类型。唯一知道的是这个对象符合相关协议。

为视图声明数据源属性的语法如下：

```objective-c
@interface XYZPieChartView : UIView
@property (weak) id<XYZPieChartViewDataSource> dataSource;
...
@end
```

声明了一个 weak 的符合 `XYZPieChartViewDataSource` 协议的通用对象指针的属性。



### \#带有可选方法的协议

默认情况下，所有在协议中声明的方法都是必需的。这意味着任何符合该协议的类都必须实现这些方法。

在协议中同样可以指定可选方法。这些方法是当一个类需要时才实现的。

使用 `@optional` 指令将协议方法标记为可选的，就像：

```objective-c
@protocol XYZPieChartViewDataSource
- (NSUInteger)numberOfSegments;
- (CGFloat)sizeOfSegmentAtIndex:(NSUInteger)segmentIndex;
@optional
- (NSString *)titleForSegmentAtIndex:(NSUInteger)segmentIndex;
@end
```

没有标记指令的默认为必须的。



### \#在运行时检查可选方法的实现

如果一个方法在协议中被标记为可选的，在试图调用它之前必须检查代理对象是否实现了该方法。

接着上面的例子，饼状图可能用如下方式测试块标题方法的实现与否：

```objective-c
NSString *thisSegmentTitle;
if ([self.dataSource respondsToSelector:@selector(titleForSegmentAtIndex:)]) {
	thisSegmentTitle = [self.dataSource titleForSegmentAtIndex:index];
}
```



### \#协议继承其他协议

```objective-c
@protocol MyProtocol <NSObject>
...
@end
```

…...未完待续



## 对象拷贝

拷贝一个对象，会创建一个和原对象具有相同类和属性的新对象。当需要自己版本的对象数据时你会拷贝一个对象。



### \#对象拷贝的条件

如果一个对象的类遵守 `NSCopying` 协议，并且实现了其中的唯一一个方法，`copyWithZone:` ，那这个对象就是可被拷贝的。如果一个类有可变和不可变两种类型，可变的应该遵守 `NSMutableCopying` 协议，并实现 `mutableCopyWithZone:` 方法来确保拷贝对象的可变性。我们通过向一个对象发送 `copy` 或 `mutableCopy` 消息来复制它。这些消息会间接调用适当的 `NSCopying` 或 `NSMutableCopying` 协议方法。

对象拷贝分为深拷贝和浅拷贝。对于纯量属性，无论是深拷贝或浅拷贝，都是直接复制，但是在处理指针引用时，两者是有区别的。

![](https://c2.staticflickr.com/9/8546/29005992113_2518922ae6_b.jpg)



### \#集合类对象的 copy && mutableCopy

* **copy 不是浅拷贝，mutableCopy 不是深拷贝**


* **不可变集合对象的 copy && mutableCopy**

  ```objective-c
  NSArray *array = @[@1, @2, @3];
  NSArray *copyArray = [array copy];
  NSMutableArray *mutableCopyArray = [array mutableCopy];

  Console:
  array == copyArray != mutableCopyArray	// 指针值的比较
  ```

* **可变集合对象的 copy && mutableCopy**

  ```objective-c
  NSMutableArray *array = [NSMutableArray arrayWithObjects:@1, @2, @3, nil];
  NSArray *copyArray = [array copy];
  NSMutableArray *mutableCopyArray = [array mutableCopy];

  Console:
  array != copyArray != mutableCopyArray 	// 指针值比较
  ```

  |               | copy | mutableCopy |
  | :-----------: | :--: | :---------: |
  | **immutable** | 浅拷贝  |     深拷贝     |
  |  **mutable**  | 深拷贝  |     深拷贝     |

  > **注意：** 这里的深、浅只是相对于集合对象本身。
  >
  > * 集合对象是一个容器，里面是指向其它对象的指针，即使容器是空的，它在内存中也是有基础数据结构的。
  >
  >
  > * 浅拷贝是把集合在内存中的地址赋值给一个该集合类型的指针。
  >
  > * 深拷贝是把集合在内存中的数据复制一份到新的内存中，并把新的内存地址赋值给该集合的可变类型指针。
  >
  > * 如图：小智，兜里可能揣有若干个精灵球，每个精灵球对应一个宝可梦。
  >
  >   ![](https://c8.staticflickr.com/9/8470/29011918863_121fb0726b.jpg)
  >
  > ```c
  > 想象一下，把小智看成集合，精灵球看成集合中存放的对象指针，宝可梦当成对象指针指向的对象。
  >
  > 浅拷贝：画一张一模一样的图片。
  > 深拷贝：把小智和精灵球克隆一份，但宝可梦是没被克隆的，即被克隆出来的精灵球和原来的球都指向同一个宝可梦。
  > ```

​       

## 集合

集合是一个 Foundation 框架对象，它的主要功能是以 数组、字典或 sets 的形式存储对象。



### \#集合类

主要的类有：`NSArray`  `NSSet` `NSDictionary` ，具有以下共性：

* 它们只能持有对象，但对象类型没有限制。
* 它们对集合内的对象持有 `__strong` 引用。
* 它们不可变，但是有一个允许你改变集合内容的可变子类。
* 可以使用 `NSEnumerator` 或 快速枚举 遍历它们的内容。



### \#排序策略

集合都有一个特有的排序策略存储对象：

* **NSArray** 

  > `NSArray` 和 它的可变子类 `NSMutableArray` 使用从 0 开始的索引。

* **NSDictionary**

  > `NSDictionary` 和它的可变子类 `NSMutableDictionary` 使用键值对。

* **NSSet**

  > `NSSet` 和 它的可变子类 `NSMutableSet` 提供无序存储的对象。




## 集合编程指南

在 Cocoa 和 Cocoa Touch 中，集合是用来存储和管理一组对象的基础框架类。

![](https://c7.staticflickr.com/9/8146/29309096350_351aac8a37_b.jpg)



这些类使得管理一组对象的任务变得轻松起来。基础集合类在 OSX 和 iOS 中是高效并使用广泛的。



### \# At a Glance

集合类有几个共性。大多数集合类都只持有对象并都有一个可变和不可变的变体。

**所有的集合类共有的功能：**

* 枚举集合中的对象
* 判断集合中是否存在某个对象
* 访问集合中的单个元素

**可变集合还有一些额外的功能：**

* 添加对象到集合中
* 从集合中移除对象

虽然集合有很多共性，但也有重要的差异。最后，你会发现一些集合类比其它集合类更适用某个特定任务。



#### 索引访问和轻松枚举元素：数组

数组是可以通过索引访问的有序集合。因为顺序因素，你可能会使用数组来存储将要被 tableView 呈现的信息。



#### 数据与任意键值结合：字典

字典是允许通过键值对访问的无序集合。它也支持快速插入和删除的操作。



#### **快速插入，删除，成员资格检查：Sets**

Sets 是对象的无序集合。允许快速插入和删除。同时具有快速查看某个对象是否属于集合。



## Block

Blocks 是添加到 C、Objective-C 和 C++ 中的语言特性，它允许你创建可被传递给方法和函数的特殊代码片段，就像传值一样。它们也具备从作用域捕获值得能力，就像其他语言中的 闭包 或 匿名函数。



### \# Block 语法

定义 block 字面语法使用脱字符 (^)：

```objective-c
^ {
	NSLog(@"This is a block.");
}
```

正如函数和方法的定义一样，这里的花括号指明 block 的开始和结束。在这个例子中，block 没有返回任何值，也没有参数。

你可以声明一个变量来跟踪 block，跟使用函数指针指向一个 C 函数一样：

```objective-c
void (^simpleBlock)(void);
```

如果你不习惯使用 C 函数指针，这儿的语法可能看起来有点怪。这个例子声明了一个叫做 `simpleBlock` 的没有参数和返回值的 block 变量，意味着可以将下面的 block 字面值赋值给它：

```objective-c
simpleBlock = ^ {
	NSLog(@"This is a block");
};
```

就像任何其它变量的赋值过程，因此声明必须以右花括号后的冒号结束。也可以将声明和赋值过程结合起来：

```objective-c
void (^simpleBlock)(void) = ^ {
	NSLog(@"This is a block");
};
```

一旦声明并赋值了 block 变量，就可以调用该 block：


```objective-c
simpleBlock()
```


> **注意**：如果你试图调用一个没有被赋值的 block 变量（即 block 变量值为 nil），程序就会崩溃。



#### 带参数和返回值的 Block

Block 同样可以获得参数并返回值，就像方法和函数一样。

```objective-c
double (^multiplyTwoValues)(double, double) = ^(double firstValue, double secondValue) {
	return firstValue * secondValue;
}

double result = multiplyTwoValues(2, 4);
```



#### Block 可以从作用域中捕获值

不但可以包含可执行代码，block 同样具有从它的作用域中捕获值的能力。

如果你在一个方法的内部声明一个 block，例如：

```objective-c
- (void)testMethod {
	int anInteger = 42;
	
	void (^testBlock)(void) = ^{
		NSLog(@"Integer is: %i", anInteger);
	};
	
	testBlock();
}

/* console */
Integer is: 42
```

没有特殊声明，只有值被捕获。这意味着，如果你在 block 定义和调用之间改变了外部变量的值，就像：

```objective-c
int anInteger = 42;

void (^testBlock)(void) = ^ {
	NSLog(@"Integer is: %i", anInteger);
};

anInteger = 84;

testBlock();
```

不影响 block 捕获的值。输出如下：

```objective-c
Integer is: 42
```

这同时也意味着，block 不能改变原始变量的值，甚至包括捕获的值 ( 作为常量被捕获 )。



#### 使用 __block 变量共享存储

如果需要能够改变 block 内部捕获的变量值，你可以在原始变量声明时使用 __block 存储类型修饰符。意味着，这个变量存储在原始变量作用域和该作用域中声明的任意 block 共享的区域中。

重写前面的例子：

```objective-c
__block int anInteger = 42;

void (^testBlock)(void) = ^{
	NSLog(@"Integer is: %i", anInteger);
};

anInteger = 84;

testBlock();
```

因为 `anInteger` 被声明为 `__block` 变量， block 共享这块存储区域。这意味着输出将显示为：

```objective-c
Integer is: 84
```

它同样意味着 block 可以修改原始值：

```objective-c
__block int anInteger = 42;

void (^testBlock)(void) = ^{
        NSLog(@"Integer is: %i", anInteger);
        anInteger = 100;
    };

testBlock();
NSLog(@"Value of original variable is now: %i", anInteger);
```

这次，输出将变为：

```markdown
Integer is: 42
Value of original variable is now: 100
```



#### 将 Blocks 作为参数传递给方法或函数

这节之前的每个例子都会在 block 定以后立即调用。在实际中，更常见的是将 block 传递给函数或方法在别处调用。例如，你可能使用 GCD 在后台调用 block，或者定义一个代表被重复调用的任务的 block，例如当枚举一个集合时。并发和枚举在本节后面讨论。

Blocks 也用来作为回调，定义了当任务完成后将要执行的代码。举个例子，你的 app 可能需要创建一个执行复杂任务的对象来响应用户的动作，比如从 web 服务请求信息。因为这个任务可能持续很长一段时间，你应该在任务正在发生时显示某种进度指示器，然后任务一旦完成后隐藏它。

也许可以用 deledation 实现：

* 创建一个合适的 delegate protocol
* 实现必要的方法
* 将某个对象设置为该任务的 delegate
* 一旦任务完成后，通过该对象调用代理方法

然而 Blocks 可以简化它，因为你可以在创建任务时定义回调行为，就像：

```objective-c
- (IBAction)fetchRemoteInformation:(id)sender
{
	[self showProgressIndicator];
  	
  	XYZWebTask *task = ...
    
    [task beginTaskWithCallbackBlock: ^{
    	[self hideProgressIndicator];
    }];
}
```

这个例子调用一个方法来显示进度指示器，然后创建并启动任务。回调 block 制定了一旦任务完成后将要执行的代码；在这种情况下，它只调用一个方法来隐藏进度指示器。注意这个回调 block 为了在被调用时能够调用 `hideProgressIndicator` 捕获了 `self` 指针。

按照代码简洁性，block 使得在一个地方明确得看到任务完成前后将要发生的事变的容易，避免了通过追踪代理方法来查明将要发生的事的需要。



#### **Block** 应当总是作为方法的最后一个参数

只使用一个 block 参数的方法是坠吼的。如果方法还有一个非 block 参数，block 应该放在最后：

```objective-c
- (void)beginTaskWithName:(NSString *)name completion:(void(^)(void))callback;
```

这使得方法调用内联块时更容易阅读，就像：

```objective-c
[self beginTaskWithName:@"MyTask" completion:^{
	NSLog(@"The task is complete");
}];
```



#### 使用类型定义简化 Block 语法

如果需要定义不止一个具有同样签名的 block，你可能倾向于为该签名自定义类型。

例如，定义没有参数和返回值的简单 block 类型：

```objective-c
typedef void (^XYZSimpleBlock)(void);
```

之后就能用这个自定义类型作为方法参数或创建 block 变量：

```objective-c
XYZSimpleBlock anotherBlock = ^ {
	...
};
```

```objective-c
- (void)beginFetchWithCallbackBlock:(XYZSimpleBlock)callbackBlock {
	...
	callbackBlock();
}
```

类型定义在构建一个以 block 为参数或返回值为 block 的 block 时非常有用。考虑下面的例子：

```objective-c
void (^(^complexBlock)(void (^)(void)))(void) = ^ (void (^aBlock)(void)) {
	...
	return ^{
		...
	};
};
```

用类型定义来重写这段代码具有更好的可读性：

```
XYZSimpleBlock (^betterBlock)(XYZSimpleBlock) = ^ (XYZSimpleBlock aBlock) {
	...
	return ^{
    	...
	};
};
```

One is *complexBlock*, another is *betterBlock*.



#### 使用属性来追踪 Block 的对象

定义属性来追踪 block 的语法和定义一个 block 变量类似：


```objective-c
@interface XYZObject : NSObject
@property (copy) void (^blockProperty)(void);
@end
```

> **注意：** 必须要指明 `copy` 作为 property 属性，因为 blcok 需要被赋值来追踪它捕获的原始作用域之外的状态。在 ARC 下，你不需要担心这点，它会自动生成。

Block 属性的设置或调用就像其他 block 变量一样：

```objective-c
self.blockProperty = ^ {
	...
};
self.blockProperty();
```

使用类型定义进行 block 属性声明也是合理的，就像：

```objective-c
typedef void (^XYZSimpleBlock)(void);

@interface XYZObject : NSObject
@property (copy) XYZSimpleBlock blockProerty;
@end
```



#### 捕获self，避免强循环引用

Blocks 对所有的捕获对象都持有强引用，包括 `self` ，那意味着很容易就会产生循环引用，例如，对象持有捕获 `self` 的 `copy` block 属性：

```objective-c
@interface XYZBlockKeeper : NSObject
@property (copy) void(^block)(void);
@end
  
@implementation XYZBlockKeeper
- (void)configureBlock {
	self.block = ^{
    	[self doSomething];		// capturing a strong reference to self
      							// creates a strong reference cycle
	};
}
...
@end
```

为了避免这种问题，最好的做法是 捕获一个弱引用的 `self` ，就像：


```objective-c
- (void)configureBlock {
	XYZBlockKeeper * __weak weakSelf = self;
	self.block = ^{
		[weakSelf doSomething];
	};
}
```