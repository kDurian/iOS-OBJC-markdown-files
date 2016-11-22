## Mach-O Universal Files



![Mach-O Universal Files](https://c5.staticflickr.com/9/8519/29091937332_d8dcc13578_b.jpg)



> You may have also heard of universal files, what are they? Well suppose you build an iOS app, for a 64 bit, and now you have this Mach-O file, so what happens the next code when you say you also want to build it for 32 bit devices? When you rebuild, Xcode will build another separate Mach-O file, this one built for 32 bits, RB7. And then those two files are merged into a third file, called the Mach-O universal file. And that has a header at the start, and all the header has a list of all the architectures and what their offsets are in the file.



## Fat Header

`Fat Header`的数据结构在`<mach-o/fat.h>`头文件中定义：

```objective-c
#define FAT_MAGIC	0xcafebabe

struct fat_header {
	uint32_t  magic; 		/* FAT_MAGIC */
  	uint32_t  nfat_arch;	/* number of structs that follow */
};

struct fat_arch {
	cpu_type_t	cputype;		/* cpu specifier (int) */
  	cpu_subtype_t  cupsubtype;  /* machine specifier (int) */
  	uint32_t  offset;			/* file offset to this object file */
  	uint32_t  size;				/* size of this object file */
  	uint32_t  align;			/* alignment as a power of 2 */
};
```



## Mach-O Image Files

![Mach-O Image](https://c1.staticflickr.com/9/8480/29164947496_e195bc5bfd_z.jpg)

###  Mach Header

```objective-c
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



### Load Command

```objective-c
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
  
```

…… 待续



### LINKEDIT

The __LINKEDIT segment contains raw data used by the dynamic linker, such as symbol, string, and relocation table entries.

#### Symbol Table

A symbol table is simply a list of names in an object. The names in the list may be names of functions, initialised/uninitialized memory regions, or other things depending on the object format. The symbol table does not need to be mapped into a running process and is only useful for debugging. The symbol table(and other sections) may be removed from an object when you use strip.

A symbol is a generic representation of the location of a function, data variable, or constant in an executable file. 

```c
/*
 * This is the symbol table entry structure for 64-bit architectures.
 */

struct nlist_64 {
	union {
    	uint32_t n_strx; /* index into the string table */
	}n_un;
  	uint8_t n_type;   	 /* type flag, see below */
  	uint8_t n_sect;		 /* section number or NO_SECT */
  	uint16_t n_desc;	 /* see <mach-o/stab.h> */
  	uint64_t n_value;	 /* value of this symbol (or stab offset) */
};
```

> **What is a dynamic symbol table?**
>
> Shared objects in both Mach-O and ELF have a  symbol table listing only functions that are exported by the object.
>
> This table is used during dynamic linking and is mapped into the process' address sapce when the object is loaded, unlike the symbol table which is just used for debugging.
>
> The dynamic symbol table is a subset of the symbol table. 





>  [WWDC/2016/Session/406 Optimizing App Startup Time]( https://developer.apple.com/videos/play/wwdc2016/406/)
>
>  [Github/vedon](https://github.com/vedon/Digging-Mach-O-file/blob/master/Mach-O%20File%20Overview(terminal%20tool)/Mach-O%20file%20info.md)
>
>  [Dynamic symbol table duel:ELF vs Mach-O, round 2](http://timetobleed.com/dynamic-symbol-table-duel-elf-vs-mach-o-round-2/)





