# clang rewrite 



## .m

```objective-c
@protocol PersonDelegate <NSObject>
- (void)dance;
@optional
- (void)gay;
@end

@interface Person : NSObject<PersonDelegate>
@property(nonatomic, strong) NSString *name;
@property(nonatomic, assign) NSUInteger age;
@end

@implementation Person
- (void)dance{
    NSLog(@"prefer dancing~");
}
@end
  
  
@interface Person (Strong)
- (void)playSoccer;
+ (BOOL)haveStrongInspiron;
@end
  
@implementation Person (Strong)
- (void)playSoccer
{
    NSLog(@"play soccer~");
}

+ (BOOL)haveStrongInspiron
{
    NSLog(@"have strong inspiron");
}
@end
```



## **clang -rewrite-objc  .m**



### **objc_class**

#### \#_class_t

```c
struct _class_t {
	struct _class_t *isa;
	struct _class_t *superclass;
	void *cache;
	struct _class_ro_t *ro;
};

struct _class_t OBJC_CLASS_$_Person = {
	&OBJC_METACLASS_$_Person,
	&OBJC_CLASS_$_NSObject,
	0,
	&_OBJC_CLASS_RO_$_Person,
};

struct _class_t OBJC_METACLASS_$_Person = {
	&OBJC_METACLASS_$_NSObject,
	&OBJC_METACLASS_$_NSObject,
	0,
	&_OBJC_METACLASS_RO_$_Person,
};
```



#### \#_class_ro_t

```c
struct Person_IMPL {
	struct NSObject_IMPL NSObject_IVARS;
  	NSString *_name;
  	NSUInteger _age;
};

struct _class_ro_t {
	unsigned int flags;
  	unsigned int instanceStart;
  	unsigned int instanceSize;
  	unsigned int reserved;
   	const unsigned char *ivarLayout;
  	const char *name;
  	const struct _method_list_t *baseMethods;
  	const struct _objc_protocols_list *baseProtocols;
  	const struct _ivar_list_t *ivars;
  	const unsigned char *weakIvarLayout;
  	const struct _prop_list_t *properties;
};

struct _class_ro_t _OBJC_CLASS_RO_$_Person = {
	0,
  	__OFFSETOFIVAR__(struct Person, _name),
  	sizeof(struct Person_IMPL),
  	0,
  	0,
  	"Person",
  	(const struct _method_list_t *)&_OBJC_$_INSTANCE_METHODS_Person,
  	(const struct _objc_protocol_list *)&_OBJC_CLASS_PROTOCOLS_$_Person,
  	(const struct _ivar_list_t *)&_OBJC_$_INSTANCE_VARIABLES_Person,
  	0,
  	(const struct _prop_list_t *)&_OBJC_$_PROP_LIST_Person,
};

struct _class_ro_t _OBJC_METACLASS_RO_$_Person = {
	1,
  	sizeof(struct _class_t),
  	sizeof(struct _class_t),
  	0,
  	0,
  	"Person",
  	0,
  	0,
  	0,
  	0,
  	0,
};
```



#### \#_category_t 

```c
struct _category_t {
    const char *name;
    struct _class_t *cls;
    const struct _method_list_t *instance_methods;
    const struct _method_list_t *class_methods;
    const struct _protocol_list_t *protocols;
    const struct _prop_list_t *properties;
};

static struct _method_list_t _OBJC_$_CATEGORY_INSTANCE_METHODS_Person_$_Strong = {
	sizeof(_objc_method),
  	1,
  	{
      	{
          (struct objc_selector *)"playSoccer",
          "v16@0:8",
          (void *)_I_Person_Strong_playSoccer
      	}
  	}
};

static struct _method_list_t _OBJC_$_CATEGORY_Class_METHODS_Person_$_Strong = {
	sizeof(_objc_method),
  	1,
  	{
      	{
          (struct objc_selector *)"haveStrongInspiron",
          "c16@0:8",
          (void *)_I_Person_Strong_haveStrongInspiron
      	}
  	}
};

static struct _category_t _OBJC_$_CATEGORY_Person_$_Strong =
{
    "Person",
    &OBJC_CLASS_$_Person,
    (const struct _method_list_t *)&_OBJC_$_CATEGORY_INSTANCE_METHODS_Person_$_Strong,
    (const struct _method_list_t *)&_OBJC_$_CATEGORY_CLASS_METHODS_Person_$_Strong,
    0,
    0,
};
```



#### \#_protocol_list_t 

```c
struct _protocol_t {
	void *isa;
	const char *name;
	const struct _protocol_list_t *protocol_list;	// super protocols;
	const struct method_list_t *instance_methods;
	const struct method_list_t *class_methods;
	const struct method_list_t *optionalInstanceMethods;
	const struct method_list_t *optionalClassMethods;
	const struct _prop_list_t *properties;
	const unsigned int size;	// sizeof(struct _protocol_t)
	const unsigned flags;
	const char *extendedMethodTypes;
};

struct _protocol_list_t {
	long protocol_count;
  	struct _protocol_t *super_protocols[1];
};

static struct _method_list_t _OBJC_PROTOCOL_INSTANCE_METHODS_NSObject = {
	sizeof(_objc_method),
  	19,
  	   {
       	{(struct objc_selector *)"isEqual:", "c24@0:8@16", 0},
        {(struct objc_selector *)"class", "#16@0:8", 0},
        {(struct objc_selector *)"self", "@16@0:8", 0},
       	{(struct objc_selector *)"performSelector:", "@24@0:8:16", 0},
        {(struct objc_selector *)"performSelector:withObject:", "@32@0:8:16@24", 0},
        {(struct objc_selector *)"performSelector:withObject:withObject:", "@40@0:8:16@24@32", 0},
        {(struct objc_selector *)"isProxy", "c16@0:8", 0},
        {(struct objc_selector *)"isKindOfClass:", "c24@0:8#16", 0},
        {(struct objc_selector *)"isMemberOfClass:", "c24@0:8#16", 0},
        {(struct objc_selector *)"conformsToProtocol:", "c24@0:8@16", 0},
        {(struct objc_selector *)"respondsToSelector:", "c24@0:8:16", 0},
        {(struct objc_selector *)"retain", "@16@0:8", 0},
        {(struct objc_selector *)"release", "Vv16@0:8", 0},
        {(struct objc_selector *)"autorelease", "@16@0:8", 0},
        {(struct objc_selector *)"retainCount", "Q16@0:8", 0},
        {(struct objc_selector *)"zone", "^{_NSZone=}16@0:8", 0},
        {(struct objc_selector *)"hash", "Q16@0:8", 0},
        {(struct objc_selector *)"superclass", "#16@0:8", 0},
        {(struct objc_selector *)"description", "@16@0:8", 0}
       }
};

static struct _method_list_t _OBJC_PROTOCOL_OPT_INSTANCE_METHODS_NSObject = {
	sizeof(_objc_method),
  	1,
  	{
      	{
          	(struct objc_selector *)"debugDescription",
          	"@16@0:8",
          	0
      	}
  	}
};

static struct _prop_list_t _OBJC_PROTOCOL_PROPERTIES_NSObject = {
	sizeof(_prop_t),
  	4,
    {
      	{"hash","TQ,R"},
        {"superclass","T#,R"},
        {"description","T@\"NSString\",R,C"},
        {"debugDescription","T@\"NSString\",R,C"}
    }
};

struct _protocol_t _OBJC_PROTOCOL_NSObject = {
	0,
    "NSObject",
    0,
    (const struct method_list_t *)&_OBJC_PROTOCOL_INSTANCE_METHODS_NSObject,
    0,
    (const struct method_list_t *)&_OBJC_PROTOCOL_OPT_INSTANCE_METHODS_NSObject,
    0,
    (const struct _prop_list_t *)&_OBJC_PROTOCOL_PROPERTIES_NSObject,
    sizeof(_protocol_t),
    0,
    (const char **)&_OBJC_PROTOCOL_METHOD_TYPES_NSObject
};

static struct _protocol_list_t _OBJC_PROTOCOL_REFS_PersonDelegate = {
	1,
  	&_OBJC_PROTOCOL_NSObject
};

static struct _method_list_t _OBJC_PROTOCOL_INSTANCE_METHODS_PersonDelegate = {
	sizeof(_objc_method),
  	1,
  	{
     	{
         	(struct objc_selector *)"dance",
          	"v16@0:8",
          	0
     	}
  	}
};

struct _protocol_t _OBJC_PROTOCOL_PersonDelegate = {
	0,
  	"PersonDelegate",
  	(const struct _protocol_list_t *)&_OBJC_PROTOCOL_REFS_PersonDelegate,
  	(const struct method_list_t *)&_OBJC_PROTOCOL_INSTANCE_METHODS_PersonDelegate,
  	0,
  	0,
  	sizeof(_protocol_t),
  	0,
  	(const char **)&_OBJC_PROTOCOL_METHOD_TYPES_PersonDelegate
};

static struct _protocol_list_t _OBJC_CLASS_PROTOCOLS_$_Person = {
	1,
  	&_OBJC_PROTOCOL_PersonDelegate
};
```



#### _method_list_t 

```c
struct _objc_method {
	struct objc_selector *_cmd;
  	const char *method_type;
 	void *_imp;
};

struct _method_list_t {
	unsigned int entsize;
  	unsigned int method_count;
  	struct _objc_method method_list[4];
};

static struct _method_list_t  _OBJC_$_INSTANCE_METHODS_Person = {
	sizeof(_objc_method),
  	4,
  	{
      	{
          	(struct objc_selector *)"name",
          	"@16@0:8",
          	(void *)_I_Person_name
      	},
      	{
          	(struct objc_selector *)"setName",
          	"v24@0:8@16",
          	(void *)_I_Person_setName_
		},
      	{
			(struct objc_selector *)"age",
          	"Q16@0:8",
          	(void *)_I_Person_age
        }
      	{
          	(struct objc_selector *)"setAge",
          	"v24@0:8Q16",
          	(void *)_I_Person_setAge_
      	}
	}
};
```



#### _ivar_list_t 

```c
struct _ivar_t {
	unsigned long int *offset;
  	const char *name;
  	const char *type;
  	unsigned int alignment;
  	unsigned int size;
};

struct _ivar_list_t {
	unsigned int entsize;
  	unsigned int count;
  	struct _ivar_t ivar_list[2];
};

static struct _ivar_list_t _OBJC_$_INSTANCE_VARIABLES_Person = {
	sizeof(_ivar_t),
  	2,
  	{
      	{
          	(unsigned long int *)&OBJC_IVAR_$_Person$_name,
          	"_name",
          	"@\"NSString\"", 
          	3, 
          	8
      	},
      	{
			(unsigned long int *)&OBJC_IVAR_$_Person$_age,
          	"_age",
          	"Q",
          	3,
          	8
        }
  	};
};
```



#### _prop_list_t 

```c
struct _prop_t {
	const char *name;
  	const char *attributes;
};

struct _prop_list_t {
	unsigned int entsize;
  	unsigned int count_of_properties;
  	struct _prop_t prop_list[2];
};

static struct _prop_list_t _OBJC_$_PROP_LIST_Person = {
	sizeof(_prop_t),
  	2,
  	{
    	{
        	"name",
          	"T@\"NSString\",&,N,V_name"
    	},
      	{
        	"age",
          	"TQ,N,V_age"
      	}
  	}
};
```



#### (void *)_imp

```c
static NSString *_I_Person_name(Person *self, SEL _cmd) {
	return (*(NSString **)((char *)self + OBJC_IVAR_$_Person$_name));
}

static void _I_Person_setName_(Person *self, SEL _cmd, NSString *name) {
	(*(NSString **)((char *)self + OBJC_IVAR_$_Person$_name)) = name;
}

static NSUInteger _I_Person_age(Person *self, SEL _cmd) {
  	return (*(NSUInteger *)((char *)self + OBJC_IVAR_$_Person$_age));
}

static void _I_Person_setAge_(Person *self, SEL _cmd, NSUInteger age) {
	(*(NSUInteger *)((char *)self + OBJC_IVAR_$_Person$_age)) = age;
}
```



#### ivar offset

```c
typedef struct objc_object Person;

unsigned long int OBJC_IVAR_$_Person$_name = __OFFSETOFIVAR__(struct Person, _name);
unsigned long int OBJC_IVAR_$_Person$_age = __OFFSETOFIVAR__(struct Person, _age);
```

