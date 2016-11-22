# Mach-O 



## 前言

当我们使用 Xcode 编写 iOS 或 macOS **应用程序**时， `CMD+B` 即 `Build` 是少不了的。这里先不谈  `Build` 过程的细节（感兴趣的可以参考[这篇文章](https://objccn.io/issue-6-1/) ），我们关心的是 `Build` 的产物 **Application** 。



> 在 Xcode 导航栏 *File / Project Settings* 里可以看到它的缓存根路径，然后通过 ~/ *Project Name-xxx / Build / Products / xxxsimulator / Project Name* 就能找到它。但它不能被直接打开，我们可以右键后选择 *Show Package Contents* 来显示它所包含的内容，其中有我们想要的与工程同名的 Unix executable file，即可执行二进制文件。



对于可执行二进制文件，不同的操作系统平台有不同的格式规范，常见的如下表：

| Format name |    Operating system     |
| :---------: | :---------------------: |
|     PE      |         Windows         |
|     ELF     | Unix-like  (e.g. Linux) |
|   Mach-O    |       macOS, iOS        |



可以看到，在 macOS 和 iOS 中，使用的是 Mach-O 文件格式。

那什么是 Mach-O 呢？关于这个问题你可以查维基百科，但我们这里要引用的是 *Nick Kledzik* 在 [WWDC 2016 Optimizing App Startup Time](https://developer.apple.com/videos/play/wwdc2016/406/) 中的一段演讲内容：

> **Mach-O is a bunch of file types for different runtime executables.**
>
> File Types:
>
> * **Executable** — Main binary for application
> * **Dylib** — Dynamic library (aka DSO or DLL)
> * **Bundle** — Dylib that cannot be linked, only `dlopen()` , e.g. plug-ins



## Mach-O 文件结构

![](https://c1.staticflickr.com/9/8252/28915267544_5e9a50ff93_b.jpg)

### \#Mach Header

```c
/*
 * The 32-bit mach header appears at the very beginning of the object file for
 * 32-bit architectures.
 */

#define MH_MAGIC	0xfeedface  /* the mach magic number */	

struct mach_header {
	uint32_t  magic;
    cpu_type_t  cputype;
    cpu_subtype_t  cpusubtype;
    uint32_t  filetype; 		/* type of file */
    uint32_t  ncmds; 			
    uint32_t  sizeofcmds; 		
    uint32_t  flags;
}

 /* 
 * The layout of the file depends on the filetype.  For all but the MH_OBJECT
 * file type the segments are padded out and aligned on a segment alignment
 * boundary for efficient demand pageing.  The MH_EXECUTE, MH_FVMLIB, MH_DYLIB,
 * MH_DYLINKER and MH_BUNDLE file types also have the headers included as part
 * of their first segment.
 */

/* Constants for the filetype field of the mach_header */
#define	MH_OBJECT	0x1		/* relocatable object file */
#define	MH_EXECUTE	0x2		/* demand paged executable file */
#define	MH_FVMLIB	0x3		/* fixed VM shared library file */
#define	MH_CORE		0x4		/* core file */
#define	MH_PRELOAD	0x5		/* preloaded executable file */
#define	MH_DYLIB	0x6		/* dynamically bound shared library */
#define	MH_DYLINKER	0x7		/* dynamic link editor */
#define	MH_BUNDLE	0x8		/* dynamically bound bundle file */

/* Constants for the flags field of the mach_header */ 
#define	MH_NOUNDEFS	0x1		/* the object file has no undefined references */
#define	MH_INCRLINK	0x2		/* the object file is the output of an
					   		   incremental link against a base file
					   		   and can't be link edited again */
#define MH_DYLDLINK	0x4		/* the object file is input for the
					   		   dynamic linker and can't be staticly
					   	   	   link edited again */
#define MH_BINDATLOAD	0x8	/* the object file's undefined
					   		   references are bound by the dynamic
					           linker when loaded. */
...
```



### \#Load Commands

```c
/*
 * The load commands directly follow the mach_header.  The total size of all
 * of the commands is given by the sizeofcmds field in the mach_header.  All
 * load commands must have as their first two fields cmd and cmdsize.  The cmd
 * field is filled in with a constant for that command type.  Each command type
 * has a structure specifically for it.  The cmdsize field is the size in bytes
 * of the particular load command structure plus anything that follows it that
 * is a part of the load command (i.e. section structures, strings, etc.).  To
 * advance to the next load command the cmdsize can be added to the offset or
 * pointer of the current load command.  The cmdsize for 32-bit architectures
 * MUST be a multiple of 4 bytes and for 64-bit architectures MUST be a multiple
 * of 8 bytes (these are forever the maximum alignment of any load commands).
 * The padded bytes must be zero.  All tables in the object file must also
 * follow these rules so the file can be memory mapped.  Otherwise the pointers
 * to these tables will not work well or at all on some machines.  With all
 * padding zeroed like objects will compare byte for byte.
 */

struct load_command {
	uint32_t  cmd;		/* type of load command */
  	uint32_t  cmdsize;  /* total size of command in bytes */
}

/* Constants for the cmd field of all load commands, the type */
#define	LC_SEGMENT	0x1	/* segment of this file to be mapped */
#define	LC_SYMTAB	0x2	/* link-edit stab symbol table info */
#define	LC_SYMSEG	0x3	/* link-edit gdb symbol table info (obsolete) */
#define	LC_THREAD	0x4	/* thread */
#define	LC_UNIXTHREAD	0x5	/* unix thread (includes a stack) */
#define	LC_LOADFVMLIB	0x6	/* load a specified fixed VM shared library */
#define	LC_IDFVMLIB	0x7	/* fixed VM shared library identification */
#define	LC_IDENT	0x8	/* object identification info (obsolete) */
#define LC_FVMFILE	0x9	/* fixed VM file inclusion (internal use) */
#define LC_PREPAGE      0xa     /* prepage command (internal use) */
#define	LC_DYSYMTAB	0xb	/* dynamic link-edit symbol table info */
#define	LC_LOAD_DYLIB	0xc	/* load a dynamically linked shared library */
#define	LC_ID_DYLIB	0xd	/* dynamically linked shared lib ident */
#define LC_LOAD_DYLINKER 0xe	/* load a dynamic linker */
#define LC_ID_DYLINKER	0xf	/* dynamic linker identification */
#define	LC_PREBOUND_DYLIB 0x10	/* modules prebound for a dynamically */
				/*  linked shared library */
...
  
/*
 * The segment load command indicates that a part of this file is to be
 * mapped into the task's address space.  The size of this segment in memory,
 * vmsize, maybe equal to or larger than the amount to map from this file,
 * filesize.  The file is mapped starting at fileoff to the beginning of
 * the segment in memory, vmaddr.  The rest of the memory of the segment,
 * if any, is allocated zero fill on demand.  The segment's maximum virtual
 * memory protection and initial virtual memory protection are specified
 * by the maxprot and initprot fields.  If the segment has sections then the
 * section structures directly follow the segment command and their size is
 * reflected in cmdsize.
 */
struct segment_command { /* for 32-bit architectures */
	uint32_t	cmd;		/* LC_SEGMENT */
	uint32_t	cmdsize;	/* includes sizeof section structs */
	char		segname[16];	/* segment name */
	uint32_t	vmaddr;		/* memory address of this segment */
	uint32_t	vmsize;		/* memory size of this segment */
	uint32_t	fileoff;	/* file offset of this segment */
	uint32_t	filesize;	/* amount to map from the file */
	vm_prot_t	maxprot;	/* maximum VM protection */
	vm_prot_t	initprot;	/* initial VM protection */
	uint32_t	nsects;		/* number of sections in segment */
	uint32_t	flags;		/* flags */
};

/*
 * A segment is made up of zero or more sections.  Non-MH_OBJECT files have
 * all of their segments with the proper sections in each, and padded to the
 * specified segment alignment when produced by the link editor.  The first
 * segment of a MH_EXECUTE and MH_FVMLIB format file contains the mach_header
 * and load commands of the object file before its first section.  The zero
 * fill sections are always last in their segment (in all formats).  This
 * allows the zeroed segment padding to be mapped into memory where zero fill
 * sections might be. The gigabyte zero fill sections, those with the section
 * type S_GB_ZEROFILL, can only be in a segment with sections of this type.
 * These segments are then placed after all other segments.
 *
 * The MH_OBJECT format has all of its sections in one segment for
 * compactness.  There is no padding to a specified segment boundary and the
 * mach_header and load commands are not part of the segment.
 *
 * Sections with the same section name, sectname, going into the same segment,
 * segname, are combined by the link editor.  The resulting section is aligned
 * to the maximum alignment of the combined sections and is the new section's
 * alignment.  The combined sections are aligned to their original alignment in
 * the combined section.  Any padded bytes to get the specified alignment are
 * zeroed.
 *
 * The format of the relocation entries referenced by the reloff and nreloc
 * fields of the section structure for mach object files is described in the
 * header file <reloc.h>.
 */
struct section { /* for 32-bit architectures */
	char		sectname[16];	/* name of this section */
	char		segname[16];	/* segment this section goes in */
	uint32_t	addr;			/* memory address of this section */
	uint32_t	size;			/* size in bytes of this section */
	uint32_t	offset;			/* file offset of this section */
	uint32_t	align;			/* section alignment (power of 2) */
	uint32_t	reloff;			/* file offset of relocation entries */
	uint32_t	nreloc;			/* number of relocation entries */
	uint32_t	flags;			/* flags (section type and attributes)*/
	uint32_t	reserved1;		/* reserved (for offset or index) */
	uint32_t	reserved2;		/* reserved (for count or sizeof) */
};


/*
 * The symtab_command contains the offsets and sizes of the link-edit 4.3BSD
 * "stab" style symbol table information as described in the header files
 * <nlist.h> and <stab.h>.
 */
struct symtab_command {
	uint32_t	cmd;		/* LC_SYMTAB */
	uint32_t	cmdsize;	/* sizeof(struct symtab_command) */
	uint32_t	symoff;		/* symbol table offset */
	uint32_t	nsyms;		/* number of symbol table entries */
	uint32_t	stroff;		/* string table offset */
	uint32_t	strsize;	/* string table size in bytes */
};


/*
 * A program that uses a dynamic linker contains a dylinker_command to identify
 * the name of the dynamic linker (LC_LOAD_DYLINKER).  And a dynamic linker
 * contains a dylinker_command to identify the dynamic linker (LC_ID_DYLINKER).
 * A file can have at most one of these.
 * This struct is also used for the LC_DYLD_ENVIRONMENT load command and
 * contains string for dyld to treat like environment variable.
 */
struct dylinker_command {
	uint32_t	cmd;		/* LC_ID_DYLINKER, LC_LOAD_DYLINKER or LC_DYLD_ENVIRONMENT */
	uint32_t	cmdsize;	/* includes pathname string */
	union lc_str    name;   /* dynamic linker's path name */
};


/*
 * Dynamicly linked shared libraries are identified by two things.  The
 * pathname (the name of the library as found for execution), and the
 * compatibility version number.  The pathname must match and the compatibility
 * number in the user of the library must be greater than or equal to the
 * library being used.  The time stamp is used to record the time a library was
 * built and copied into user so it can be use to determined if the library used
 * at runtime is exactly the same as used to built the program.
 */
struct dylib {
    union lc_str  name;			    /* library's path name */
    uint32_t timestamp;			    /* library's build time stamp */
    uint32_t current_version;		/* library's current version number */
    uint32_t compatibility_version;	/* library's compatibility vers number*/
};

/*
 * A dynamically linked shared library (filetype == MH_DYLIB in the mach header)
 * contains a dylib_command (cmd == LC_ID_DYLIB) to identify the library.
 * An object that uses a dynamically linked shared library also contains a
 * dylib_command (cmd == LC_LOAD_DYLIB, LC_LOAD_WEAK_DYLIB, or
 * LC_REEXPORT_DYLIB) for each library it uses.
 */
struct dylib_command {
	uint32_t	cmd;		/* LC_ID_DYLIB, LC_LOAD_{,WEAK_}DYLIB, LC_REEXPORT_DYLIB */
	uint32_t	cmdsize;	/* includes pathname string */
	struct dylib	dylib;  /* the library identification */
};

...
...
...
  
/*
 * The entry_point_command is a replacement for thread_command.
 * It is used for main executables to specify the location (file offset)
 * of main().  If -stack_size was used at link time, the stacksize
 * field will contain the stack size need for the main thread.
 */
struct entry_point_command {
    uint32_t  cmd;			/* LC_MAIN only used in MH_EXECUTE filetypes */
    uint32_t  cmdsize;		/* 24 */
    uint64_t  entryoff;		/* file (__TEXT) offset of main() */
    uint64_t  stacksize;	/* if not zero, initial stack size */
};

...
```



### \#Raw Segment Data

* __PAGEZERO

  > ​

* __TEXT

  > ​

* __DATA

  > ​

* __LINKEDIT

  > ​



## Example



```c
~$ otool -l -h Demo_Mach_O_iOS

Mach header
      magic cputype  cpusubtype  caps    filetype ncmds sizeofcmds      flags
 0xfeedfacf 16777223          3  0x00           2    19       2504 0x00200085
Load command 0
      cmd LC_SEGMENT_64
  cmdsize 72
  segname __PAGEZERO
   vmaddr 0x0000000000000000
   vmsize 0x0000000100000000
  fileoff 0
 filesize 0
  maxprot 0x00000000
 initprot 0x00000000
   nsects 0
    flags 0x0
Load command 1
      cmd LC_SEGMENT_64
  cmdsize 712
  segname __TEXT
   vmaddr 0x0000000100000000
   vmsize 0x0000000000003000
  fileoff 0
 filesize 12288
  maxprot 0x00000007
 initprot 0x00000005
   nsects 8
    flags 0x0
Section
  sectname __text
   segname __TEXT
      addr 0x00000001000016e0
      size 0x0000000000000510
    offset 5856
     align 2^4 (16)
    reloff 0
    nreloc 0
     flags 0x80000400
 reserved1 0
 reserved2 0
Section
  sectname __stubs
   segname __TEXT
      addr 0x0000000100001bf0
      size 0x0000000000000036
    offset 7152
     align 2^1 (2)
    reloff 0
    nreloc 0
     flags 0x80000408
 reserved1 0 (index into indirect symbol table)
 reserved2 6 (size of stubs)
Section
  sectname __stub_helper
   segname __TEXT
      addr 0x0000000100001c28
      size 0x000000000000006a
    offset 7208
     align 2^2 (4)
    reloff 0
    nreloc 0
     flags 0x80000400
 reserved1 0
 reserved2 0
Section
  sectname __objc_methname
   segname __TEXT
      addr 0x0000000100001c92
      size 0x0000000000000a13
    offset 7314
     align 2^0 (1)
    reloff 0
    nreloc 0
     flags 0x00000002
 reserved1 0
 reserved2 0
Section
  sectname __objc_classname
   segname __TEXT
      addr 0x00000001000026a5
      size 0x0000000000000042
    offset 9893
     align 2^0 (1)
    reloff 0
    nreloc 0
     flags 0x00000002
 reserved1 0
 reserved2 0
Section
  sectname __objc_methtype
   segname __TEXT
      addr 0x00000001000026e7
      size 0x000000000000081d
    offset 9959
     align 2^0 (1)
    reloff 0
    nreloc 0
     flags 0x00000002
 reserved1 0
 reserved2 0
Section
  sectname __cstring
   segname __TEXT
      addr 0x0000000100002f04
      size 0x00000000000000a6
    offset 12036
     align 2^0 (1)
    reloff 0
    nreloc 0
     flags 0x00000002
 reserved1 0
 reserved2 0
Section
  sectname __unwind_info
   segname __TEXT
      addr 0x0000000100002fac
      size 0x0000000000000048
    offset 12204
     align 2^2 (4)
    reloff 0
    nreloc 0
     flags 0x00000000
 reserved1 0
 reserved2 0
Load command 2
      cmd LC_SEGMENT_64
  cmdsize 1032
  segname __DATA
   vmaddr 0x0000000100003000
   vmsize 0x0000000000001000
  fileoff 12288
 filesize 4096
  maxprot 0x00000007
 initprot 0x00000003
   nsects 12
    flags 0x0
Section
  sectname __nl_symbol_ptr
   segname __DATA
      addr 0x0000000100003000
      size 0x0000000000000010
    offset 12288
     align 2^3 (8)
    reloff 0
    nreloc 0
     flags 0x00000006
 reserved1 9 (index into indirect symbol table)
 reserved2 0
Section
  sectname __la_symbol_ptr
   segname __DATA
      addr 0x0000000100003010
      size 0x0000000000000048
    offset 12304
     align 2^3 (8)
    reloff 0
    nreloc 0
     flags 0x00000007
 reserved1 11 (index into indirect symbol table)
 reserved2 0
Section
  sectname __objc_classlist
   segname __DATA
      addr 0x0000000100003058
      size 0x0000000000000018
    offset 12376
     align 2^3 (8)
    reloff 0
    nreloc 0
     flags 0x10000000
 reserved1 0
 reserved2 0
Section
  sectname __objc_protolist
   segname __DATA
      addr 0x0000000100003070
      size 0x0000000000000010
    offset 12400
     align 2^3 (8)
    reloff 0
    nreloc 0
     flags 0x00000000
 reserved1 0
 reserved2 0
Section
  sectname __objc_imageinfo
   segname __DATA
      addr 0x0000000100003080
      size 0x0000000000000008
    offset 12416
     align 2^2 (4)
    reloff 0
    nreloc 0
     flags 0x00000000
 reserved1 0
 reserved2 0
Section
  sectname __objc_const
   segname __DATA
      addr 0x0000000100003088
      size 0x0000000000000d58
    offset 12424
     align 2^3 (8)
    reloff 0
    nreloc 0
     flags 0x00000000
 reserved1 0
 reserved2 0
Section
  sectname __objc_selrefs
   segname __DATA
      addr 0x0000000100003de0
      size 0x0000000000000040
    offset 15840
     align 2^3 (8)
    reloff 0
    nreloc 0
     flags 0x10000005
 reserved1 0
 reserved2 0
Section
  sectname __objc_classrefs
   segname __DATA
      addr 0x0000000100003e20
      size 0x0000000000000010
    offset 15904
     align 2^3 (8)
    reloff 0
    nreloc 0
     flags 0x10000000
 reserved1 0
 reserved2 0
Section
  sectname __objc_superrefs
   segname __DATA
      addr 0x0000000100003e30
      size 0x0000000000000008
    offset 15920
     align 2^3 (8)
    reloff 0
    nreloc 0
     flags 0x10000000
 reserved1 0
 reserved2 0
Section
  sectname __objc_ivar
   segname __DATA
      addr 0x0000000100003e38
      size 0x0000000000000018
    offset 15928
     align 2^3 (8)
    reloff 0
    nreloc 0
     flags 0x00000000
 reserved1 0
 reserved2 0
Section
  sectname __objc_data
   segname __DATA
      addr 0x0000000100003e50
      size 0x00000000000000f0
    offset 15952
     align 2^3 (8)
    reloff 0
    nreloc 0
     flags 0x00000000
 reserved1 0
 reserved2 0
Section
  sectname __data
   segname __DATA
      addr 0x0000000100003f40
      size 0x00000000000000b0
    offset 16192
     align 2^3 (8)
    reloff 0
    nreloc 0
     flags 0x00000000
 reserved1 0
 reserved2 0
Load command 3
      cmd LC_SEGMENT_64
  cmdsize 72
  segname __LINKEDIT
   vmaddr 0x0000000100004000
   vmsize 0x0000000000002000
  fileoff 16384
 filesize 6360
  maxprot 0x00000007
 initprot 0x00000001
   nsects 0
    flags 0x0
Load command 4
            cmd LC_DYLD_INFO_ONLY
        cmdsize 48
     rebase_off 16384
    rebase_size 216
       bind_off 16600
      bind_size 304
  weak_bind_off 0
 weak_bind_size 0
  lazy_bind_off 16904
 lazy_bind_size 248
     export_off 17152
    export_size 192
Load command 5
     cmd LC_SYMTAB
 cmdsize 24
  symoff 17368
   nsyms 144
  stroff 19752
 strsize 2992
Load command 6
            cmd LC_DYSYMTAB
        cmdsize 80
      ilocalsym 0
      nlocalsym 119
     iextdefsym 119
     nextdefsym 8
      iundefsym 127
      nundefsym 17
         tocoff 0
           ntoc 0
      modtaboff 0
        nmodtab 0
   extrefsymoff 0
    nextrefsyms 0
 indirectsymoff 19672
  nindirectsyms 20
      extreloff 0
        nextrel 0
      locreloff 0
        nlocrel 0
Load command 7
          cmd LC_LOAD_DYLINKER
      cmdsize 32
         name /usr/lib/dyld (offset 12)
Load command 8
     cmd LC_UUID
 cmdsize 24
    uuid 9E32701D-B786-3E79-8E7B-66F29C26304A
Load command 9
      cmd LC_VERSION_MIN_IPHONEOS
  cmdsize 16
  version 9.3
      sdk 9.3
Load command 10
      cmd LC_SOURCE_VERSION
  cmdsize 16
  version 0.0
Load command 11
       cmd LC_MAIN
   cmdsize 24
  entryoff 7008
 stacksize 0
Load command 12
          cmd LC_LOAD_DYLIB
      cmdsize 88
         name /System/Library/Frameworks/Foundation.framework/Foundation (offset 24)
   time stamp 2 Thu Jan  1 08:00:02 1970
      current version 1280.25.0
compatibility version 300.0.0
Load command 13
          cmd LC_LOAD_DYLIB
      cmdsize 56
         name /usr/lib/libobjc.A.dylib (offset 24)
   time stamp 2 Thu Jan  1 08:00:02 1970
      current version 228.0.0
compatibility version 1.0.0
Load command 14
          cmd LC_LOAD_DYLIB
      cmdsize 56
         name /usr/lib/libSystem.dylib (offset 24)
   time stamp 2 Thu Jan  1 08:00:02 1970
      current version 1226.10.1
compatibility version 1.0.0
Load command 15
          cmd LC_LOAD_DYLIB
      cmdsize 80
         name /System/Library/Frameworks/UIKit.framework/UIKit (offset 24)
   time stamp 2 Thu Jan  1 08:00:02 1970
      current version 3512.60.7
compatibility version 1.0.0
Load command 16
          cmd LC_RPATH
      cmdsize 40
         path @executable_path/Frameworks (offset 12)
Load command 17
      cmd LC_FUNCTION_STARTS
  cmdsize 16
  dataoff 17344
 datasize 24
Load command 18
      cmd LC_DATA_IN_CODE
  cmdsize 16
  dataoff 17368
 datasize 0

```

