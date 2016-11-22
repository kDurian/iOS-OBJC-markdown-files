# objc  category的秘密



## category 的真面目

objc 所有类和对象都是 C 结构体，category 当然也是一样，下面是 `runtime` 中 category 的结构：

```c
struct _category_t {
	const char *name;	// 1
  	struct _class_t *cls;	// 2
  	const struct _method_list_t *instance_methods;	// 3
  	const struct _method_list_t *class_methods;	// 4
   	const struct _protocol_list_t *protocols;	// 5
  	const struct _prop_list_t *properties;	// 6
};
```

1. `name` :   这里的 name 是类的名字，不是 category 小括号里的写的名字。

2. `cls` :   要扩展的类对象，编译期间这个值是 0，在 app 被 runtime 加载时才会根据 `name` 找到对应的类对象。

3. `instance_methods` :   这个 category 中所有的 `-` 方法。

4. `class_methods` :   这个 category 中所有的 `+` 方法。

5. `protocols` :   这个 category 中实现的 protocol，不常用，但支持。

6. `properties` ：  这个 category 所有的 property，这也是 category 里可以声明属性的原因，不过这个 property 不会 `@synthesize` 实例变量，一般有需求添加实例变量属性时会采用 `objc_setAssociatedObject` 和 `objc_getAssociatedObject` 方法绑定，不过这种方法生成的与普通的实例变量完全是两码事。

   ​

   ​

   ​