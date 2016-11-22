## After _objc_init

map_2\_images

_read_images



load_images 	number 1

…...

load_images  number 101



map_2\_images

_read_images

map_2\_images

_read_images



load_images  number 102

…...

load_images  number 111



### map_2\_images  

```c
// struct objc_class
struct bucket_t {
private:
  	uintptr_t _key;
  	IMP _imp;
};

struct cache_t {
	struct bucket_t *_buckets;
  	uint32_t _mask;
  	uint32_t _occupied;
};

struct class_data_bits_t {
  	uintptr_t bits;
};

struct class_rw_t {
	uint32_t flags;
  	uint32_t version;
  	const class_ro_t *ro;
  	method_array_t methods;
  	property_array_t properties;
  	protocol_array_t protocols;
  	Class firstSubclass;
  	Class nextSiblingClass;
  	char *demangledName;
};

struct class_ro_t {
	uint32_t flags;
  	uint32_t instanceStart;
  	uint32_t instanceSize;
  	uint32_t reserved;
  	const uint8_t *ivarLayout;
  	const char *name;
  	method_list_t *baseMethodList;
  	protocol_list_t *baseProtocols;
  	const ivar_list_t *ivars;
  	const uint8_t *weakIvarLayout;
  	property_list_t *baseProperties;
};

struct objc_class {
	union isa_t isa;
  	Class superclass;
  	cache_t cache;
  	class_data_bits_t bits;  // class_rw_t * plus custom rr/alloc flags
};

// protocol_list_t 
struct protocol_list_t {
	uintptr_t count;
  	uintptr_t list[0]; // protocol_t *
};

struct protocol_t : objc_object {
  	union isa_t isa;
	const char *mangledName;
  	struct protocol_list_t *protocols;
  	method_list_t *instanceMethods;
  	method_list_t *classMethods;
  	method_list_t *optionalInstanceMethods;
  	method_list_t *optionalClassMethods;
  	property_list_t *instance_Properties;
  	uint32_t size;
  	uint32_t flags;
};

// category_list
struct category_list {
	uint32_t count;
  	uint32_t reserved;
  	locstamped_category_t list[0];
};

struct locstamped_category_t {
	category_t *cat;
  	struct header_info *hi;
};

struct category_t {
	const char *name;
  	classref_t cls;
  	struct method_list_t *instanceMethods;
  	struct method_list_t *classMethods;
  	struct protocol_list_t *protocols;
  	strut property_list_t *instanceProperties;
};


struct objc_image_info {
	uint32_t version;
  	uint32_t flags;
};

struct header_info {
 	struct header_info *next;
  	const headerType *mhdr;
  	const objc_image_info *info;
  	const char *fname;
  	bool loaded;
  	bool inSharedCache;
  	bool allClassesRealized;
  	
  	//func
  	...   
};

struct MapPair {
	const void *key;
  	const void *value;
};

struct NSMapTable {
	...
    unsigned count;
  	...
  	struct MapPair buckets[];
};

static struct NSMapTable *namedSelectors;  // 静态全局变量，作用域为所在源文件，objc_sel.mm

struct header_info *FirstHeader = 0;	// 非静态全局变量，作用域为所有源文件 libobjc.dylib
struct header_info *LastHeader = 0;
int HeaderCount = 0;

const char *
map_2_images(enum dyld_image_states state, uint32_t infoCount,
             const struct dyld_image_info infoList[])
{
//    dyld_image_info info = infoList[0];
//    printf("func name: %s\n", __FUNCTION__);
    rwlock_writer_t lock(runtimeLock);
    return map_images_nolock(state, infoCount, infoList);
}

const char *
map_images_nolock(enum dyld_image_states state, uint32_t infoCount,
                  const struct dyld_image_info infoList[]) 
{
	static bool firstTime = YES;
    static bool wantsGC = NO;
    uint32_t i;
    header_info *hi;
    header_info *hList[infoCount];
    uint32_t hCount;
    size_t selrefCount = 0;
  	
  	i = infoCount;
  	hCount = 0;
  	while(i--) {
      	const struct mach_header *mhdr = infoList[i].imageLoadAddress;
      	hi = addHeader(mhdr) {				// list all image that has objc contents 
          	size_t info_size = 0;
          	unsigned long seg_size;
          	const objc_image_info *image_info = _getObjcImageInfo(mhdr, &info_size);
          	const uint8_t *objc_segment = getsegmentdata(mhdr, "__OBJC", &seg_size);
          	if(!objc_segment && !image_info) return NULL;
          	
          	hi = (header_info *)calloc(sizeof(head_info), 1);   //在堆上
          	hi->mhdr = mhdr;
          	hi->info = image_info;
          	hi->fname = dyld_image_path_containing_address(hi->mhdr);
          	hi->loaded = true;
          	hi->isSharedCache = false;
          	hi->allClassesRealized = NO;
          
          	if (hi->mhdr->filetype == MH_DYLIB && _hasObjcContents(hi)) {
              	dlopen(hi->fname, RTLD_NOLOAD);
          	}
          
          	appendHeader(hi) {
				HeaderCount++;
              	hi->next = NULL;
              	if (!Firstheader) {
                    // list is empty
					FirstHeader = LastHeader = hi;
                } else {
                  	if (!LastHeader) {
                      	...
                  	}
                  	LastHeader->next = hi;
                  	LastHeader = hi;
                }
            }
      	}
      	if (!hi) {
          	// no objc data in this entry, 该 .dylib 中没有与 objc 有关的数据信息
          	continue;
      	}
      
      	struct NXMapTable *namedSelectors = NSCreatedMapTable();
      	NXMapInsert(nameSelectors, const char *name, SEL sel);
      	/*  objc_sel.mm   void sel_init(bool wantsGC, size_t selrefCount)
      		load;
      		initialize;
      		retain;
      		release;
      		autorelease;
      		retainCount;
      		alloc;
      		dealloc;
      		copy;
      		new;
      		...
      	*/
      	
      	hList[hCount++] = hi;  // Images that contain objc contents. Include main exec.
  	}
  
  	_read_images(hList, hCount);
  
  	return NULL;
}

static bool preoptimized; //YES if this image's dyld shared cache optimizations are valid.
struct NXMapTable *gdb_objc_realized_classes;

void _read_images(header_info **hList, uint32_t hCount)
{
//    printf("func name: %s\n", __FUNCTION__); 
    header_info *hi;
    uint32_t hIndex;
    size_t count;
    size_t i;
    Class *resolvedFutureClasses = nil;
    size_t resolvedFutureClassCount = 0;
    static bool doneOnce;
  
#define EACH_HEADER	(hIndex = 0; hIndex < hCount && (hi = hList[hIndex]); hIndex++)

  	// Count classes. Size various table based on the total.
  	int total = 0;
  	int unoptimizedTotal = 0;
  	for (EACH_HEADER) {
      	if (_getObjc2ClassList(hi, &count)) {
			total += (int)count;
          	if (!hi->inSharedCache) unoptimizedTotal += count;
        }
  	}
  
  	gdb_objc_realized_classes = NSCreateMapTable(...);
  
  	for (EACH_HEADER) {
      	bool headerIsBundle = hi->isBundle();
        bool headerIsPreoptimized = hi->isPreoptimized(); //image's dyld shared cache optimizations are valid
    	classref_t *classlist = _getObjc2ClassList(hi, &count);
      	for (i = 0; i < count; i++) {
          	Class cls = (Class)classlist[i];
          	Class newCls = readClass(cls, headerIsBundle, headerIsPreoptimized)
	        {
				const char *mangledName = cls->mangledName(){
                  	if (isRealized() || ifFuture()) {
                      	return data()->ro->name;
                  	}else {
                      	return ((class_ro_t *)data())->name;
                  	}
				}
              
              	Class replacing = nil;
              	
              	addNamedClass(cls, mangledName, replacing){
                  	NSMapInsert(gdb_objc_realized_classes, name, cls);
              	}
              
              	return cls;
            }
      	}
    }
  
  	/*
  	static const char *sel_cname(SEL sel) {
      	return (const char *)(void *)sel;
  	}
  	*/
  	
  	// Fix up @selector references
  	static size_t UnfixedSelectors;
  	sel_lock();
  	for (EACH_HEADER) {
      	SEL *sels = _getObjc2SelectorRefs(hi, &count);
      	UnfixedSelectors += count;
      	for (int i = 0; i < count; i++) {
          	const char *name = sel_cname(sels[i]);
          	sels[i] = sel_registerNameNoLock(name, isBundle){
              	SEL result = 0;
              	result = sel_alloc(name, copy) {
                  	return (SEL)(copy ? strdup(name) : name); 
              	}
              	NXMapInsert(namedSelectors, sel_getName(result), result);
          	}
      	}
  	}
  	sel_unlock();
  
  	for (EACH_HEADER) {
      	extern objc_class OBJC_CLASS_$_Protocol;
      	Class cls = (Class)&OBJC_CLASS_$_Protocol;
      	NSMapTable *protocol_map = protocols(); // NSCreateMapTable();
      	protocol_t **protolist = _getObjc2ProtocolList(hi, &count);
      	for (int i = 0; i < count; i++) {
          	protocol_t *newproto = protolist[i];
          	readProtocol(newproto, cls, protocol_map, isPreoptimized, isBundle){
              	newproto->initIsa(protocol_class);
              	NSMapInsert(protocol_map, newproto->mangledName, newproto);
          	}
      	}
  	}
  
  	for (EACH_HEADER) {
		classref_t *classlist = _getObjc2NonlazyClassList(hi, &count);
      	for (int i = 0; i < count; i++) {
          	Class cls = classlist[i];
          	if (!cls) continue;
          	
          	realizeClass(cls) { // Performs first-time initialization on class cls, including allocating its read-write data. Returns the real class structure for the class.
              	
              	const class_ro_t *ro;
              	class_rw_t *rw;
              	Class supercls;
              	Class metacls;
              	bool isMeta;
              
              	if (!cls) return nil;  // 终止条件  NSObject.supercls = nil  NSObject.isa = NSObjectMeta
              	if (cls->isRealized()) return cls; // 终止条件  NSObjectMeta.supercls = NSObject  NSObjectMeta.isa = self  
              
              	ro = (const class_ro_t *)cls->data();
              	rw = (class_rw_t *)calloc(sizeof(class_rw_t), 1);  // 64byte in heap
              	rw->ro = ro;
              	rw->flags = RW_REALIZED | RW_REALIZING;
              	cls->setData(rw);
              	
              	isMeta = ro->flags & RO_META;
              	rw->version = isMeta ? 7 : 0;
              
              	supercls = realizeClass(remapClass(cls->superclass));  // 递归
              	metacls = realizeClass(remapClass(cls->ISA()));	// 递归
              
             	methodizeClass(cls) {
                  	bool isMeta = cls->isMetaClass();
                  	auto rw = cls->data();
                  	auto ro = rw->ro;
                  	
                  	method_list_t *list = ro->baseMethods;
                  	if (list) {
                      	rw->methods.attachLists(&list, 1);
                  	}
                  
                  	property_list_t *proplist = ro->baseProperties;
                  	if (proplist) {
                      	rw->properties.attachLists(&proplist, 1);
                  	}
                  
                  	protocol_list_t *protolist = ro->baseProtocols;
                  	if (protolist) {
						rw->protocols.attachLists(&protolist, 1);
                    }
                  
                  	category_list *cats = unattachedCategoriesForClass(cls, true);
                  	attachCategories(cls, cats, false){
						bool isMeta = cls->isMetaClass();
                      	
                      	method_list_t **mlists = (method_list_t **)malloc(cats->count * sizeof(*mlists));
                      	property_list_t **proplists = (property_list_t **)malloc(cats->count * sizeof(*proplists));
                      	protocol_list_t **protolists = (protocol_list_t **)malloc(cats->count * sizeof(*protolists));
                      
                      	int mcount = 0;
                      	int propcount = 0;
                      	int protocount = 0;
                      	int i = cats->count;  // 类的分类个数
                      
                      	while(i--) {
                          	auto& entry = cats->list[i];
                          	method_list_t *mlist = entry.cat->methodsForMeta(isMeta);
                          	if (mlist) {
                              	mlists[mcount++] = mlist;
                          	}
                          
                          	property_list_t *proplist = entry.cat->propertiesForMeta(isMeta);
                          	if (proplist) {
                              	proplists[propcount++] = proplist;
                          	}
                          
                          	protocol_list_t *protolist = entry.cat->protocols;
                          	if (protolist) {
                              	protolists[protocount++] = protolist;
                          	}
                      	}
                      	auto rw = cls->data();
                      	rw->methods.attachLists(mlists, mcount);
                      	rw->properties.attachLists(proplists, propcount);
                      	rw->protocols.attachLists(protolists, protocount);
                    }
             	}
          	}
      	}
    }
}
  	

```



### First time

 ![Screen Shot 2016-09-16 at 3.07.41 AM](/Users/jtliu/Desktop/Screen Shot 2016-09-16 at 3.07.41 AM.png)

```c
#define	MH_EXECUTE	0x2		/* demand paged executable file */
struct mach_header_64 {
	uint32_t	magic;		/* mach magic number identifier */
	cpu_type_t	cputype;	/* cpu specifier */
	cpu_subtype_t	cpusubtype;	/* machine specifier */
	uint32_t	filetype;	/* type of file */
	uint32_t	ncmds;		/* number of load commands */
	uint32_t	sizeofcmds;	/* the size of all the load commands */
	uint32_t	flags;		/* flags */
	uint32_t	reserved;	/* reserved */
};

struct dyld_image_info {
	const struct mach_header*	imageLoadAddress;	/* base address image is mapped into */
	const char*					imageFilePath;		/* path dyld used to load the image */
	uintptr_t					imageFileModDate;	/* time_t of image file */			
};


const struct mach_header *imageLoadAddress = 0x100000000;
uint32_t filetype = MH_EXECUTE
imageFilePath = '/Users/jtliu/Desktop/SourceCache/objc/Build/Products/Debug/debug-objc'
```



### Second time

 ![Screen Shot 2016-09-16 at 3.10.29 AM](/Users/jtliu/Desktop/Screen Shot 2016-09-16 at 3.10.29 AM.png)

```c
#define	MH_DYLIB	0x6		/* dynamically bound shared library */

const struct mach_header *imageLoadAddress = 0x00007fff8d3c3000;
uint32_t filetype = MH_DYLIB
imageFilePath = 	"/System/Library/Frameworks/ApplicationServices.framework/Versions/A/Frameworks/SpeechSynthesis.framework/Versions/A/SpeechSynthesis"
```

### Third time

 ![Screen Shot 2016-09-16 at 3.10.42 AM](/Users/jtliu/Desktop/Screen Shot 2016-09-16 at 3.10.42 AM.png)



