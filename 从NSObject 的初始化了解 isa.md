# 从NSObject 的初始化了解 isa

```
struct objc_object {
	isa_t isa;
};

struct objc_class {
	isa_t isa;
	Class super_class;
	cache_t cache;
	class_data_bits_t bits;
};
```
##结构体`isa_t`


```
#define ISA_MASK        0x00007ffffffffff8ULL
#define ISA_MAGIC_MASK  0x001f800000000001ULL
#define ISA_MAGIC_VALUE 0x001d800000000001ULL
#define RC_ONE   (1ULL<<56)
#define RC_HALF  (1ULL<<7)

union isa_t {
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;

    struct {
        uintptr_t indexed           : 1;
        uintptr_t has_assoc         : 1;
        uintptr_t has_cxx_dtor      : 1;
        uintptr_t shiftcls          : 44;
        uintptr_t magic             : 6;
        uintptr_t weakly_referenced : 1;
        uintptr_t deallocating      : 1;
        uintptr_t has_sidetable_rc  : 1;
        uintptr_t extra_rc          : 8;
    };
};
```
`isa_t`是一个`union`类型的结构体，对`union`不熟悉的读者可以看到这个stackoverflow上的[回答](http://stackoverflow.com/questions/252552/why-do-we-need-c-unions)。也就是说其中的`cls`、`bits`还有结构体共用同一块地址空间。

##`isa`的初始化
我们可以通过`isa`初始化的方法`initIsa`来初步了解64位的bits的作用，当我们对一个Objc对象分配内存时，其方法调用栈中包含了下面的两个方法：

```
inline void
objc_object::initInstanceIsa(Class cls, bool hasCxxDtor)
{
	initIsa(cls, true, hasCxxDtor);
}

inline void 
objc_object::initIsa(Class cls, bool indexed, bool hasCxxDtor)
{
	if(!indexed) {
		isa.cls = cls;
	} else {
		isa.bits = ISA_MAGIC_VALUE;
		isa.has_cxx_dtor = hasCxxDtor;
		isa.shiftcls = (uintptr_t)cls >> 3;
	}
}
```
1. 对整个`isa`的bits进行设置，传入`ISA_MAGIC_VALUE`

> \#define ISA_MAGIC_VALUE 0X001d800000000001ULL


```
union isa_t {
	struct {
		indexed = 1;
		has_assoc = 0;
		has_cxx_dtor = 0;
		shiftcls = 00000000000000000000000000000000000000000000;// 44
		magic = 110111; // 6
		weakly_referenced = 0;
		deallocating = 0;
		has_sidetable_rc = 0;
		extra_rc = 00000000; // 8
	}
}
```
2.设置`hasCxxDtor`

3.设置`shiftcls`

```
/***********************************************************************
* class_createInstance
* fixme
* Locking: none
**********************************************************************/

id 
class_createInstance(Class cls, size_t extraBytes)
{
    return _class_createInstanceFromZone(cls, extraBytes, nil);
}

static __attribute__((always_inline)) 
id
_class_createInstanceFromZone(Class cls, size_t extraBytes, void *zone, 
                              bool cxxConstruct = true, 
                              size_t *outAllocatedSize = nil)
{
    if (!cls) return nil;
    
    printf("%p\n", cls); // 100000000001110101110000011111000

    assert(cls->isRealized());

    // Read class's info bits all at once for performance
    bool hasCxxCtor = cls->hasCxxCtor();
    bool hasCxxDtor = cls->hasCxxDtor();
    bool fast = cls->canAllocIndexed();

    size_t size = cls->instanceSize(extraBytes);
    if (outAllocatedSize) *outAllocatedSize = size;

    id obj;
    if (!UseGC  &&  !zone  &&  fast) {
        obj = (id)calloc(1, size);
        if (!obj) return nil;
        obj->initInstanceIsa(cls, hasCxxDtor);
    } 
    else {
#if SUPPORT_GC
        if (UseGC) {
            obj = (id)auto_zone_allocate_object(gc_zone, size,
                                                AUTO_OBJECT_SCANNED, 0, 1);
        } else 
#endif
        if (zone) {
            obj = (id)malloc_zone_calloc ((malloc_zone_t *)zone, 1, size);
        } else {
            obj = (id)calloc(1, size);
        }
        if (!obj) return nil;
        printf("%p\n", obj); // 1011101100000000000000100000000001110101110000011111001

        // Use non-indexed isa on the assumption that they might be 
        // doing something weird with the zone or RR.
        obj->initIsa(cls);
    }

    if (cxxConstruct && hasCxxCtor) {
        obj = _objc_constructOrFree(obj, cls);
    }

    return obj;
}

/// 结论

printf("%p\n", cls);
> 100000000001110101110000011111000
printf("%p\n", obj);
> 1011101100000000000000100000000001110101110000011111001

1.所有类指针的最后三位二进制都是0
2.
```

####各变量的含义

| 变量名               | 含义                                      |
| ----------------- | --------------------------------------- |
| indexed           | 0表示普通的`isa`指针，1表示使用优化，存储类信息             |
| has_assoc         | 表示该对象是否包含associated object, 如果没有，则析构时更快 |
| has_cxx_dtor      | 表示该对象是否有C++或ARC的析构函数，如果没有，则析构时更快        |
| shiftcls          | 类的指针                                    |
| magic             | 固定值为0xd2，用于在调试时分辨对象是否未完成初始化             |
| weakly_referenced | 表示该对象是否有被弱引用过，如果没有，则析构时更快               |
| deallocating      | 表示该对象是否正在析构                             |
| has_sidetable_rc  | 表示该对象的引用计数值是否过大无法存储在`isa`指针             |
| extra_rc          | 存储引用计数值减一后的结果                           |

000100000000001110101110000011111(33)
0000000001011101100000000000000100000000001110101110000011111001



