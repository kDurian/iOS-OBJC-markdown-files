# ObjC源码 与 C代码

##NyanCat.m

```
#import <Foundation/Foundation.h>

@interface NyanCat : NSObject
@property(nonatomic, strong) NSString *name;
@property(nonatomic, assign) int age;

+ (void)nyan1;
- (void)nyan2;
@end


@implementation NyanCat
+ (void)nyan1 {
    printf("class method~");
}

- (void)nyan2 {
    printf("instance method~");
}
@end

```

##NyanCat.cpp

```
typedef struct objc_object NyanCat;
typedef struct {} _objc_exc_NyanCat;

extern "C" unsigned long OBJC_IVAR_$_NyanCat$_name;
extern "C" unsigned long OBJC_IVAR_$_NyanCat$_age;

struct NyanCat_IMPL {
	struct NSObject_IMPL NSObject_IVARS;
	int _age;
	NSString *__name;
};

//@implementation NyanCat

static void _C_NyanCat_nyan1(Class self, SEL _cmd)
{
	printf("class method~");
}

static void _I_NyanCat_nyan2(NyanCat *self, SEL _cmd)
{
	printf("instance method~");
}

static NSString * _I_NyanCat_name(NyanCat *self, SEL _cmd)
{
	return (*(NSString **)((char *)self + OBJC_IVAR_$_NyanCat$_name));
}

static void _I_NyanCat_setName_(NyanCat * self, SEL _cmd, NSString *name)
{
    (*(NSString **)((char *)self + OBJC_IVAR_$_NyanCat$_name)) = name;
}

static int _I_NyanCat_age(NyanCat * self, SEL _cmd)
{
    return (*(int *)((char *)self + OBJC_IVAR_$_NyanCat$_age));
}

static void _I_NyanCat_setAge_(NyanCat * self, SEL _cmd, int age)
{
    (*(int *)((char *)self + OBJC_IVAR_$_NyanCat$_age)) = age;
}

//@end

struct _prop_t {
	const char *name;
	const char *attributes;
};

struct _objc_method {
	struct objc_selector *_cmd;
	const char *method_type;
	void *_imp;
};

struct _protocol_t {
	void *isa;
	const char *protocol_name;
	const struct _protocol_list_t *protocol_list;
	const struct method_list_t *instance_methods;
	const struct method_list_t *class_methods;
	const struct method_list_t *optionalInstanceMethods;
	const struct method_list_t *optionalClassMethods;
	const struct _prop_list_t *properties;
	const unsigned int size;
	const unsigned int flags;
	const char **extendedMethodTypes;
};

struct _ivar_t {
	unsigned long int *offset; // pointer to ivar offset location
	const char *name;
	const char *type;
	unsigned int alignment;
	unsigned int size;
};

struct _class_ro_t {
	unsigned int flags;
	unsigned int instanceStart;
	unsigned int instanceSize;
	unsigned int reserved;
	const unsigned char *ivarLayout;
	const char *name;
	const struct _method_list_t *baseMethods;
	const struct _objc_protocol_list *baseProtocols;
	const struct _ivar_list_t *ivars;
	const unsigned char *weakIvarLayout;
	const struct _prop_list_t *properties;
};

struct _class_t {
	struct _class_t *isa;
	struct _class_t *superclass;
	void *cache;
	void *vtable;
	struct _class_ro_t *ro;
};

struct _category_t {
	const char *name;
	struct _class_t *cls;
	const struct _method_list_t *instance_methods;
	const struct _method_list_t *class_methods;
	const struct _protocol_list_t *protocols;
	const struct _prop_list_t *properties;
};

struct objc_cache {
	unsigned int mask;
	unsigned int occupied;
	Method buckets[1];
};

extern "C" struct objc_cache _objc_empty_cache;

extern "C" unsigned long int OBJC_IVAR_$_NyanCat$_age = __OFFSETOFIVAR__(	struct NyanCat, _age);
extern "C" unsigned long int OBJC_IVAR_$_NyanCat$_name = __OFFSETOFIVAR__(struct NyanCat, _name);

static struct /*_ivar_list_t*/ {
	unsigned int entsize;
	unsigned int count;
	struct _ivar_t ivar_list[2];
}
_OBJC_$_INSTANCE_VARIABLES_NyanCat = {
	sizeof(_ivar_t),
	2,
	{
	{(unsigned long int *)&OBJC_IVAR_$_NyanCat$_age, "_age", "i", 2, 4},
	{(unsigned long int *)&OBJC_IVAR_$_NyanCat$_name, "_name", "@\"NSString\"", 3, 8}
	}
};

static struct /*_method_list_t*/ {
	unsigned int entsize; // sizeof(struct _objc_method)
	unsigned int method_count;
	struct _objc_method method_list[5];
}
_OBJC_$_INSTANCE_METHODS_NyanCat = {
	sizeof(_objc_method),
	5,
	{
	{(struct objc_selector *)"nyan2", "v16@0:8", (void *)_I_NyanCat_nyan2},
	{(stuct objc_selector *)"name", "@16@0:8", (void *)_I_NyanCat_name},
	{(struct objc_selector *)"setName:", "v24@0:8@16", (void *)_I_NyanCat_setName_},
	{(struct objc_selector *)"age", "i16@0:8", (void *)_I_NyanCat_age},
	{(struct objc_selector *)"setAge", "v20@0:8i16", (void *)_I_NyanCat_setAge_}
	}
};

static struct /*_method_list_t*/ {
	unsigned int entsize;  // sizeof(struct _objc_method)
	unsigned int method_count;
	struct _objc_method method_list[1];
}
_OBJC_$_CLASS_METHODS_NyanCat = {
	sizeof(_objc_method),
	1,
	{
	{(struct objc_selector *)"nyan1", "v16@0:8", (void *)_C_NyanCat_nyan1}
	}
};

static struct /*_prop_list_t*/ {
	unsigned int entsize;  // sizeof(struct _prop_t)
	unsigned int count_of_properties;
	struct _prop_t prop_list[2];
}
_OBJC_$_PROP_LIST_NyanCat = {
	sizeof(_prop_t),
	2,
	{
	{"name","T@\"NSString\",&,N,V_name"},
	{"age","Ti,N,V_age"}
	}
};

static struct _class_ro_t _OBJC_METACLASS_RO_$_NyanCat = {
	1, 
	sizeof(struct _class_t),
	sizeof(struct _class_t),
	(unsigned int)0,
	0,
	"NyanCat",
	(const struct _method_list_t *)&_OBJC_$_CLASS_METHODS_NyanCat,
	0,
	0,
	0,
	0,
};

static struct _class_ro_t _OBJC_CLASS_RO_$_NyanCat = {
	0, 
	__OFFSETOFIVAR__(struct NyanCat, _age), 
	sizeof(struct NyanCat_IMPL), 
	(unsigned int)0, 
	0, 
	"NyanCat",
	(const struct _method_list_t *)&_OBJC_$_INSTANCE_METHODS_NyanCat,
	0, 
	(const struct _ivar_list_t *)&_OBJC_$_INSTANCE_VARIABLES_NyanCat,
	0, 
	(const struct _prop_list_t *)&_OBJC_$_PROP_LIST_NyanCat,
};

extern "C" struct _class_t OBJC_METACLASS_$_NSObject;

extern "C" struct _class_t OBJC_METACLASS_$_NyanCat = {
	0, // &OBJC_METACLASS_$_NSObject,
	0, // &OBJC_METACLASS_$_NSObject,
	0, // (void *)&_objc_empty_cache,
	0, // unused, was (void *)&_objc_empty_vtable,
	&_OBJC_METACLASS_RO_$_NyanCat,
};

extern "C" struct _class_t OBJC_CLASS_$_NSObject;

extern "C" struct _class_t OBJC_CLASS_$_NyanCat = {
	0, // &OBJC_METACLASS_$_NyanCat,
	0, // &OBJC_CLASS_$_NSObject,
	0, // (void *)&_objc_empty_cache,
	0, // unused, was (void *)&_objc_empty_vtable,
	&_OBJC_CLASS_RO_$_NyanCat,
};


static void OBJC_CLASS_SETUP_$_NyanCat( void )
{
	OBJC_METACLASS_$_NyanCat.isa = &OBJC_METACLASS_$_NSObject;
	OBJC_METACLASS_$_NyanCat.superclass = &OBJC_METACLASS_$_NSObject;
	OBJC_METACLASS_$_NyanCat.cache = &_objc_empty_cache;
	OBJC_CLASS_$_NyanCat.isa = &OBJC_METACLASS_$_NyanCat;
	OBJC_CLASS_$_NyanCat.superclass = &OBJC_CLASS_$_NSObject;
	OBJC_CLASS_$_NyanCat.cache = &_objc_empty_cache;
}


static void *OBJC_CLASS_SETUP[] =
{
	(void *)&OBJC_CLASS_SETUP_$_NyanCat,
};

static struct _class_t *L_OBJC_LABEL_CLASS_$ =
{
	&OBJC_CLASS_$_NyanCat,
};

static struct IMAGE_INFO
{
    unsigned version;
    unsigned flag;
} _OBJC_IMAGE_INFO = { 0, 2 };
```

##核心代码
```

struct _class_t OBJC_ClASS_$_NyanCat = {
	isa = &OBJC_METACLASS_$_NyanCat,
	superclass = &OBJC_CLASS_$_NSObject,
	cache = (void *)&_objc_empty_cache,
	ro = &_OBJC_CALSS_RO_$_NyanCat,
};

struct _class_ro_t _OBJC_CLASS_RO_$_NyanCat {
    0,
    __OFFSETOFIVAR__(struct NyanCat, _age),
    sizeof(struct NyanCat_IMPL),
    (unsigned int)0,
    0,
    "NyanCat",
    (const struct _method_list_t *)&_OBJC_$_INSTANCE_METHODS_NyanCat,
    0,
    (const struct _ivar_list_t *)&_OBJC_$_INSTANCE_VARIABLES_NyanCat,
    0,
    (const struct _prop_list_t *)&_OBJC_$_PROP_LIST_NyanCat,
};

struct _class_t OBJC_METACLASS_$_NyanCat = {
	isa = &OBJC_METACLASS_$_NSOBJECT,
	superclass = &OBJC_METACLASS_$_NSOBJECT,
	cache = (void *)&_objc_empty_cache,
	ro = &_OBJC_METACLASS_RO_$_NyanCat,
};

struct _class_ro_t _OBJC_METACLASS_RO_$_NyanCat {
    1,
    sizeof(struct _class_t),
    sizeof(struct _class_t),
    (unsigned int)0,
    0,
    "NyanCat",
    (const struct _method_list_t *)&_OBJC_$_CLASS_METHODS_NyanCat,
    0,
    0,
    0,
    0,
};

```

