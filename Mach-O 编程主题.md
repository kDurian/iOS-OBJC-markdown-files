# Mach-O 编程主题



## 简介

OSX 支持多种应用程序生态环境，每种都有它自己的规则、惯例和文件格式。在 OSX 中，内核扩展、命令行工具、应用程序、框架和库（共享和静态）的实现都是基于 Mach-O ( Mach object ) 文件。

OSX 的运行时体系结构规定了目标文件如何在文件系统中布局和程序如何与内核通信。OSX 采用的目标文件格式是 Mach-O。

一个 Mach-O 文件有以下数据区域：

* **头:** 指定文件的目标架构，比如 PPC, PPC64, IA-32, or x86-64.
* **加载命令:** 指定文件的逻辑结构和虚拟内存中的布局
* **原始数据段:** 包含定义在加载命令中定义的段的原始数据

> 下面的列表描述了 OSX 支持的其他运行时环境：
>
> - *Classic*
> - *LaunchCFMApp*
> - *Java virtual machine*
> - *OSX kernel* 

这个文档讨论了你应当如何使用 Mach-O 文件格式。它讲述了：

* 你可以构建什么类型的程序
* 程序是如何加载和执行的
* 你可以如何改变程序加载和执行的方式
* 如何在运行时加载代码
* 如何在运行时加载和链接代码

如果你创建或加载了 *Bundles*、共享库或者框架，你可能会想阅读和理解这份文档中的一切。



### 谁应该阅读这份文档

如果你想为 OSX 写开发工具，你需要理解这份文档传达的信息。

这份文档对于共享库和框架的开发者也是有用的，需要在运行时加载代码的应用程序开发者也同样适用。



### 文档的组织结构

这份文档包含以下文章：

* [创建 Mach-O 文件](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachOTopics/1-Articles/building_files.html#//apple_ref/doc/uid/TP40001828-SW1) 

  > 描述了Mac 应用程序是如何创建的和你可以开发的应用程序类型

* [执行 Mach-O 文件](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachOTopics/1-Articles/executing_files.html#//apple_ref/doc/uid/TP40001829-SW1)

  > 提供了 OSX 动态加载过程的概述

* [运行时加载代码](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachOTopics/1-Articles/loading_code.html#//apple_ref/doc/uid/TP40001830-SW1)

  > 描述了如何使用共享库和框架和如何在运行时加载插件

* [间接寻址](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachOTopics/1-Articles/indirect_addressing.html#//apple_ref/doc/uid/TP40004919-SW1)

  > 解释了 Mach-O 文件如何引用定义在另一个 Mach-O 文件中的符号

* [位置无关代码](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachOTopics/1-Articles/dynamic_code.html#//apple_ref/doc/uid/TP40002528-SW1)

  > 讨论了动态链接器加载代码到随机虚拟存储地址的方法

* [x86-64 Code Model](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachOTopics/1-Articles/x86_64_code.html#//apple_ref/doc/uid/TP40005044-SW1)

  > 描述了 OSX x86-64 用户空间代码模型和 Symtem V x86-64 代码模型的区别

这份文档也包含了修订记录和索引。



## 创建 Mach-O 文件

为了生成程序，开发者将源代码转换成目标文件。然后这些目标文件被打包成可执行代码或者静态库。OSX 包含了将源代码转换成可运行的应用程序或可以被一个或多个应用程序使用的静态库的工具。

这篇文章大致描述了 Mac 应用程序是如何构建的，并且深入讨论了你能够构建的应用程序类型。它讲述了 Mach-O 文件构建过程中涉及的工具，说明了你可以构建的 Mach-O 文件类型，并谈及了在 OSX 运行时环境中最小的可链接代码和数据单元，模块。也描绘了由模块集合包装成的静态存档库。



### 工具 — 构建和运行 Mach-O 文件

内核实际上使用 *动态链接器* （位于 **/usr/lib/dyld** 特殊标记的动态共享库 ）在运行时执行加载和绑定程序的工作。内核把程序和动态链接器加载到一个新的进程并执行它们。

在整个文档中，抽象讨论以下工具：

* **编译器** 

  > 一个将高级语言编写的源代码翻译成包含二进制机器码和数据的中级目标文件的工具。除非另有说明，本书把机器语言汇编器当做编译器。

* **静态链接器**

  > 一个将中级目标文件联合成最终产品的工具。

不论你是使用 Xcode，标准命令行工具或者第三方工具集来开发你的应用程序，明白下面的每个工具的角色可以提高你对 Mach-O 运行时体系结构的理解和促进与其他 OSX 开发者对这个话题的交流。

这些标准工具包括以下几种：

* **编译器驱动程序**

  > */usr/bin/gcc* ,  包含 编译、汇编和链接 C、C++和 Objective-C 源代码模块的支持。

* **C++ 编译器驱动程序**

  > */usr/bin/c++* , 自动链接 C++ 运行时函数到输出文件。

* **汇编器**

  > */usr/bin/as* , 从汇编语言代码生成中间对象文件。

* **静态链接器**

  > */usr/bin/ld* , 编译器用来结合 Mach-O 可执行文件。

* **库生成工具**

  > */usr/bin/libtool* , 根据给定参数，创建静态库或动态共享库。

用来分析 Mach-O 文件的工具：

* **/usr/bin/lipo**

  > 可以让你创建和分析包含不止一个架构的 images 二进制文件

* **/usr/bin/file**

  > 文件类型显示工具，显示某个文件的类型。对于多架构文件，它将显示组成归档文件的每个 images 的类型。

* **/usr/bin/otool**

  > 目标文件显示工具，列出 Mach-O 文件中指定 section 和 segment 的内容。

* **/usr/bin/pagestuff**

  > 页面分析工具，显示组成指定 image 的每个逻辑页，包括每个逻辑页中包含的 section 和 symbol 名称。这个工具对于多架构二进制文件是无效的。

* **/usr/bin/nm**

  > 符号表显示工具，查看目标文件符号表中的内容。



### 产品 — 可以构建的 Mach-O 文件类型

在 OSX 中，一个典型的应用程序的可执行代码来源于多种类型的文件。主要的可执行文件通常包含程序的核心逻辑，包括 main 函数入口。应用程序主要的功能通常是在主要的可执行文件的代码中实现的。

其它包含可执行代码的文件有：

* **中间目标文件**

  > 这些文件不是最终产品，它们是更大的目标文件的基本构建块，通常情况下，编译器为每个输入的源代码文件生成的代码和数据创建一个中间目标文件。然后使用静态链接器将目标文件组合到动态链接器。集成开发环境，例如 Xcode 通常隐藏这个层面的细节。

* **动态共享库**

  > 包含供应用程序动态引用的可重复使用的可执行代码模块，当应用程序启动时将被动态链接器加载。 共享库通常用于存储供应用程序使用的大量代码。

* **框架**

  > 包含共享库和关联资源，比如图像文件、开发文档和编程接口的目录。

* **伞型框架**

  > 包含不止一种子框架的特殊框架类型。例如，Cocoa 伞型框架就包含了 Application Kit 框架和 Foundation 框架。

* **静态文档库**

  > 包含可被静态链接器在运行时添加到你的应用程序的可重用代码模块。静态文档库通常包含仅仅被几种应用程序使用的少量代码，或者因某些原因很难被共享库维护的代码。

* **Bundles**

  > 应用程序可以使用动态链接功能在运行时加载的可执行文件。**Bundles** 可以实现插件功能。在 OSX 中*bundle* 术语有两层含义：
  >
  > * 包含可执行代码的实际目标文件
  > * 包含目标文件和相关资源的目录文件。例如，OSX 中的软件被打包成 *bundles*。并且，因为这些 *bundles* 在 *Finder* 中是以一个单独文件而非目录显示的，应用程序 *bundles* 也被当作 *application packages*。
  >
  > 后者使用的更加普遍。然而，除非另有说明，本文档是指前者。

* **内核扩展**

  > 静态绑定的 Mach-O 文件，类似于 *Bundles*。内核扩展是被加载到内核地址空间的，因此构建起来和其它 Mach-O 文件不同。

默认情况下，静态链接器在 */System/Library/Frameworks*下搜索框架和伞型框架，在 */usr/lib* 下搜索共享库和静态文档库。



## 执行 Mach-O 文件

这篇文章提供了 OSX 动态加载过程的概述。在 OSX 中加载和链接一个程序的过程主要涉及两个实体：

* **OSX 内核**
* **动态链接器**

当你运行一个程序，内核为程序创建一个进程，然后加载程序和动态链接共享库，通常是 */usr/lib/dyld*，到程序的地址空间。然后内核执行动态链接器中的代码加载程序引用的库文件。



### 启动应用

当你从 Finder 或 Dock 中启动应用，或者在 shell 中运行程序，系统最后都会代你调用两个函数，`fork` 和 `execve` 。`fork` 函数创建一个进程；`execve` 函数加载和执行程序。

> 有几个变种 exec 函数，例如，`execl` ，`execv` 和 `exect` ，它们向程序传递参数和环境变量的方式稍有不同。在 OSX 中，这些 exec 函数最终都会调用内核 `execve` 。

另外：

如果想像 Finder 或 Dock 一样，在当前程序中打开别的程序，需要使用 [Launch Services Framework](https://developer.apple.com/library/mac/documentation/Carbon/Conceptual/LaunchServicesConcepts/LSCIntro/LSCIntro.html#//apple_ref/doc/uid/TP30000999) 。



### 分支和执行进程

为了使用 BSD 系统调用创建一个进程，当前进程必须调用 `fork` 系统调用。`fork` 调用创建一个当前进程的逻辑拷贝，然后将新进程的 ID 返回到当前进程。

为了运行一个不同的可执行文件，你的进程必须调用带有替代可执行文件位置的路径名的`execve` 系统调用。`execve` 系统调用用一个不同的可执行文件替换当前内存中的程序。

一个 Mach-O 可执行文件包含一个由一组加载命令组成的文件头。对于使用共享库或框架的程序，其中一个命令指出了用来加载程序的链接器位置。如果你使用 Xcode，常常是 **/usr/lib/dyld**，标准的 OSX 动态链接器。

当`execve` 响应后，内核首先加载指定的程序文件并检查位于文件开始处的 `mach_header` 结构体。内核确认是有效的 Mach-O 文件，解释存储在文件头中的加载命令。然后内核加载 由加载命令指定的动态链接器到内存中，并在程序文件上执行动态链接器。

动态链接器加载所有的主程序依赖的所有共享库并绑定足够多的符号来使程序启动。然后再调用入口函数。在构建时期，静态链接器添加目标文件 `/usr/lib/crt1.o` 中的标准入口函数到主要可执行文件。这个函数为内核设置了运行时状态环境，静态初始化C++ 对象，初始化 Objective-C 运行时，然后调用程序的 `main` 函数。



### 找到输入符号































> [Mach-O Programming Topics](https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html#//apple_ref/doc/uid/TP40001827-SW1)

