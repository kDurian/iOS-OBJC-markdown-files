# 重温C++ 对象和类
##10.2 抽象和类
###10.2.1 类型是什么
简而言之，指定基本类型完成了3项工作：

* 决定数据对象需要的内存数量
* 决定如何解释内存中的位(long 和 float 在内存中占用的位数相同，但将它们转换为数值的方法不同)
* 决定可使用数据对象执行的操作或方法

对于内置类型来说，有关操作的信息被内置到编译器中。但在C++中定义用户自定义的类型时，必须自己提供这些信息。付出这些劳动换来了根据实际需要定制新数据类型的强大功能和灵活性。

###10.2.2 C++中的类
stocks.cpp的第一部分

```
#include <iostream>
#include <cstring>

class Stock {
	char company[30]; // private by default
	int shares;
	double share_val;
	double total_val;
	void set_tot() {  // private by default
		total_val = shares * share_val;
	}
public:
	void acquire(const char *co, int n, double pr);
	void buy(int num, double price);
	void sell(int num, double price);
	void update(double price);
	void show();
};		
```

无论类成员是数据成员还是成员函数，都可以在类的公有部分或私有部分声明它。但由于隐藏数据是OOP主要的目标之一，因此数据项通常放在私有部分，组成类接口的成员函数放在公有部分；否则，就无法从程序中调用这些函数。

> **类和结构**
> 类描述看上去很像是包含成员函数以及public和private可见性标签的结构声明。实际上，C++对结构进行了扩展，使之具有与类相同的特性。
>
> 它们之间惟一的区别是，结构的默认访问类型是public，而类为private。

###10.2.3 实现类成员函数
特殊的特征：

* 定义成员函数时，使用作用域解析操作符(::)来标识函数所属的类
* 类方法可以访问类的private组件

stocks.cpp 第二部分

```
void Stock::acquire(const char *co, int n, double pr)
{
	std::strncpy(company, co, 29);
	company[29] = '\0';
	if (n < 0) {
		std::cerr<<"Number of shares can't be negative."
		<<company<<"shares set to 0.\n";
	   shares = 0;
	}
	else shares = n;
	share_val = pr;
	set_tot();
}

void Stock::buy(int num, double price)
{
	if (num < 0)
	{
		std::cerr<<"Number of shares purchased can't be negative."
		<<"Transaction is aborted.\n";
	}
	else 
	{
		shares += num;
		share_val = price;
		set_tot();
	}
}

void Stock::sell(int num, double price)
{
	using std::cerr;
	if (num < 0)
	{
		cerr<<"Number of shares sold can't be negative."
		<<"Transaction is aborted.\n";
	}
	else if (num > shares)
	{
		cerr<<"You can't sell more than you have!"
		<<"Transaction is aborted.\n";
	}
	else 
	{
		shares -=num;
		share_val = price;
		set_tot();
	}
}

void Stock::update(double price)
{
	share_val = price;
	set_tot();
}

void Stock::show()
{
	using std::cout:
	using std::endl;
	cout<<"Company: "<<company<<" Shares: "<<shares<<endl
	<<" Share Price: $"<<share_val
	<<" Total Worth: $"<<total_val<<endl;
}
	
```
所创建的每个新对象都有自己的存储空间，用于存储其内部变量和类成员；但同一个类的所有对象共享同一组类方法，即每种方法只有一个副本。

##10.3 类的构造函数和析构函数
###10.3.1 声明和定义构造函数

```
// constructor prototype with some default arguments
Stock(const char *co, int n = 0, double pr = 0.0);
```
注意，没有返回类型。原型位于类声明的公有部分。
下面是构造函数的一种可能定义：

```
Stock::Stock(const char *co, int n, double pr)
{
	std::strncpy(company, co, 29);
	company[29] = '\0';
	if (n < 0) {
		std::cerr<<"Number of shares can't be negative."
		<<company<<"shares set to 0.\n";
	   shares = 0;
	}
	else shares = n;
	share_val = pr;
	set_tot();
}
```
上述代码和本章前面的acquire()函数相同。区别在于，程序声明对象时，将自动调用构造函数。

###10.3.2 使用构造函数
C++提供了两种使用构造函数来初始化对象的方式。

第一种方式是显示调用构造函数：

```
Stock food = Stock("World Cabbage", 250, 1.25);
```

另一种方式是隐式调用构造函数：

```
Stock garment("Furry Mason", 50, 2.5);
```
这种格式更紧凑，它与下面的显示调用等价：

```
Stock garment = Stock("Furry Mason", 50, 2.5);
```

模板类

vector<int> nums;
将导致编译器定义名为vector<int>的类，并创建一个类型为Vector<int>的nums对象。定义类时，编译器将使用int替换


模板使得算法独立于存储的数据类型，而迭代器使算法独立于使用的容器类型。因此，它们都是STL通用方法的重要组成部分。


1.

```
double *find_ar(double *ar, int n, const double &val)
{
	for (int i = 0; i < n; i++) 
	{
		if (ar[i] == val)
			return &ar[i];
	}	
	return 0;
}
```
该函数使用下标来遍历数组。尽管如此，这种算法仍然与一种特定的数据结构(数组)关联在一起。

<br>

2.

下面来看搜索另一种数据结构——链表的情况。链表由被链接在一起的Node结构组成：

```
struct Node {
	double item;
	Node *p_next;
};
```
假设有一个指向链表的第一个节点的指针，每个节点的p_next指针都指向下一个节点，链表最后一个节点的p_next指针被设置为0：

```
Node *find_ll(Node *head, const double &val)
{
	Node *start;
	for (start = head; start != 0; start = start->p_next)
	{
		if (start->item == val)
			return start;
	}
	return 0;
}
```
同样，也可以使用模板将这种算法推广到支持 == 操作符的任何数据类型的链表。不过，这种算法也是与特定的数据结构——链表关联在一起。

从实现细节上看，这两个find函数的算法是不同的：

* 一个使用数组索引来遍历元素
* 另一个则将start重置为start->p_next。

不过，从广义上说，这两种算法是相同的：
>将值依次与容器中的每个值进行比较，直到找到匹配的为止


通用编程技术旨在使用同一个find函数来处理数组、链表或任何其他容器类型。即函数不仅独立于容器中存储的数据类型，而且独立于容器本身的数据结构。模板提供了存储在容器中的数据类型的通用表示，因此还需要遍历容器中的值的通用表示，迭代器正是这样的通用表示。

要实现find函数，迭代器应具备哪些特征呢？下面是一个简短的特征列表：

* 应能够对迭代器执行解除引用的操作，以便能够访问它引用的值。即如果p是一个迭代器，则应对 *p 进行定义。
* 应能够将一个迭代器赋值给另一个。即如果p和q都是迭代器，则应对表达式p=q进行定义。
* 应能够将一个迭代器与另一个进行比较，看它们是否相等。即如果p和q都是迭代器，则应对p == q 和 p != q进行定义。
* 应能够使用迭代器遍历容器中的所有元素，这可以通过为迭代器p定义++P和p++来实现。


实际上，STL按功能的强弱定义了多种级别的迭代器。顺便说一句，常规指针就能满足迭代器的要求：

```
typedef double *iterator;
iterator find_ar(iterator ar, int n, const double &val)
{
	for (int i = 0; i < n; i++, ar++)
	{
		if (*ar == val)
			return ar;
		return 0;
	}
}
```


```
typedef double *iterator;
iterator find_ar(iterator begin, iterator end, const double &val)
{	
	iterator ar;
	for (ar = begin; ar != end; ar++)
	{
		if (*ar == val)
			return ar;
		return end;
	}
}
```

对于find_ll()函数，可以定义一个迭代器类，其中定义了操作符*和++:

```
stuct Node {
	double item;
	Node *p_next;
};

class iterator
{
	Node *pt;
	
public:
	iterator():pt(0){}
	iterator(Node *pn): pt(pn){}
	
	double operator *() // * 操作符重载
	{
		return pt->item;
	}	
	
	iterator & operator++()  // ++it 操作符重载
	{
		pt = pt->p_next;
		return *this;
	}
	
	iterator operator++(int) // it++  操作符重载
	{
		iterator tmp = *this;
		pt = pt->p_next;
		return tmp;
	}
// ... operator==(), operator!=()...etc.
};
```
这里重点不是如何定义iterator类，而是有了这样的类后，第二个find函数就可以这样编写:

```
iterator find_ll(iterator head, const double &val)
{
	iterator start;
	for(start = head; start != 0; ++start)
	{
		if (*start == val)
			return start;
	}
	return 0;
}
```

STL遵循上面介绍的方法。首先，每个容器类(vector、list、deque等)定义了相应的迭代器类型。对于其中的某个类，迭代器可能是指针；而对于另一个类，则可能是对象。不管实现方式如何，迭代器都将提供所需的操作：如*和++(有些类需要的操作可能比其他类多)。其次，每个容器类都有一个超尾标记，当迭代器递增到超越容器的最后一个值后，这个值将被赋给迭代器。每个容器类都有begin()和end()方法，它们分别返回一个指向容器的第一个元素和超尾位置的迭代器。每个容器类都使用++操作，让迭代器从指向第一个元素逐步指向超尾位置，从而遍历容器中的每一个元素。


使用容器类时，无须知道其迭代器是如何实现的，也无须知道超尾是如何实现的，而只需要知道它有迭代器，其begin()返回一个指向第一个元素的迭代器，end()返回一个指向超尾位置的迭代器即可。例如，假设要打印vector<double>对象中的值，则可以这样做:

```
vector<double> scores;


vector<double>::iterator pr;
for (pr = scores.begin(); pr != scores.end(); pr++)
	cout<< *pr << endl;
	
其中代码行:
vector<double>::iterator pr;
将pr的类型声明为vector<double>类的迭代器。如果要使用list<double>类模板来存储分数，则代码如下:

list<double>::iterator pr;
for (pr = scores.begin(); pr != scores.end(); pr++)
	cout<< *pr << endl;
```

唯一不同的是pr的类型。因此，STL通过为每个类定义适当的迭代器，并以统一的风格设计类，能够对内部表示决然不同的容器，编写相同的代码。

基于算法的要求，设计基本迭代器的特征和容器特征。

###迭代器类型
STL定义了5种迭代器，并根据所需的迭代器类型对算法进行了描述。这5种迭代器分别是输入迭代器、输出迭代器、正向迭代器、双向迭代器和随机访问迭代器。




