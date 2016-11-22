## weak  reference 原理

```objective-c
@interface Durian : NSObject
@property (weak) Nycat *cat;
@end
  
@interface Nycat : NSObject
@end
  

{
  	... 
  	Durian *du = [Durian new];
	Nycat *cat = [Nycat new];
	du.cat = cat;  // weak reference
}
```



weak reference 表示当该实例变量所指向的对象被释放时，该实例变量自动置为 nil。

而对象的释放是从 dealloc 