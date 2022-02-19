# Chapter 9：Virtual Memory

虚拟内存：硬件异常、硬件地址翻译、主存、磁盘文件和内核软件的交互，提供大的、一致的和私有的地址空间。

## 9.1 Physical and Virtual Addressing

物理寻址

![image-20220218160154957](Chapter 9：Virtual Memory.assets/image-20220218160154957.png)

虚拟寻址

![image-20220218160219646](Chapter 9：Virtual Memory.assets/image-20220218160219646.png)

生成虚拟地址访问主存。

地址翻译：将虚拟地址换为物理地址，需要内存管理单元(MMU)

## 9.2 Address Spaces

线性地址空间：整数连续

虚拟地址空间：从N=2的n次方生成虚拟地址。(n位地址空间)

物理地址空间：物理内存的M个字节，假设M=2的m次

## 9.3 VM as a Tool for Caching 

虚拟内存分割：虚拟页

物理内存分割：物理页/页帧

![image-20220218161315442](Chapter 9：Virtual Memory.assets/image-20220218161315442.png)

![image-20220218161356108](Chapter 9：Virtual Memory.assets/image-20220218161356108.png)

1、4、6已缓存，所以有对应的物理内存

### 9.3.1 DRAM Cache Organization

SRAM缓存：CPU和主存间的高速缓存

DRAM缓存：虚拟内存缓存

DRAM不命中需要磁盘服务，开销巨大。

SRAM不命中基于DRAM服务。

虚拟页：4KB~8MB，全相联，任何虚拟页放置在任何物理页。写回策略

### 9.3.2 Page Tables

操作系统软件、MMU中的地址翻译硬件和存放在物理地址的**页表**实现命中

![image-20220218162007454](Chapter 9：Virtual Memory.assets/image-20220218162007454.png)

一个有效位和n位地址字段组成。

有效位：虚拟页是否被缓存在DRAM中。

* 设置有效位，响应物理页起始位置
* 没有有效位，空地址：未分配，有地址：虚拟页在磁盘起始位置。

### 9.3.3 Page Hits

地址翻译硬件将虚拟地址作为索引定位PTE2，从内存读取，(根据有效位确定缓存在内存中)

### 9.3.4 Page Faults

DRAM缓存不命中。

通过有效位确定VP3未缓存，触发缺页异常，若VP4已修改，复制回磁盘，再将VP3替换到PP3位置，更新PTE3。重新启动缺页指令，能正常翻译地址

![image-20220218170224988](Chapter 9：Virtual Memory.assets/image-20220218170224988.png)

### 9.3.5 Allocating Pages

调用malloc，将VP5分配给PTE5

![image-20220218170803181](Chapter 9：Virtual Memory.assets/image-20220218170803181.png)

在工作集内，由于局部性，有很好的性能，

但工作集过大，会发生抖动，页面不断换进换出

### 9.4 VM as a Tool for Memory Management

每个进程提供独立的页表，独立的虚拟地址空间

![image-20220218173610120](Chapter 9：Virtual Memory.assets/image-20220218173610120.png)

![image-20220218174048803](Chapter 9：Virtual Memory.assets/image-20220218174048803.png)

![image-20220218174316034](Chapter 9：Virtual Memory.assets/image-20220218174316034.png)

## 9.5 VM as a Tool for Memory Protection

![image-20220218174509451](Chapter 9：Virtual Memory.assets/image-20220218174509451.png)

设置三个许可位，SUP为内核模式才能访问。

如果违反许可，触发“段错误”

## 9.6 Address Translation

![image-20220218174718180](Chapter 9：Virtual Memory.assets/image-20220218174718180.png)

![image-20220218174855747](Chapter 9：Virtual Memory.assets/image-20220218174855747.png)

页表基址寄存器指向当前页表。

通过VPN找到页表位置，得到物理页号。

再将虚拟页偏移作为物理页偏移。

拼接得到物理地址。

**命中步骤**

![image-20220218175310232](Chapter 9：Virtual Memory.assets/image-20220218175310232.png)

![image-20220218175326032](Chapter 9：Virtual Memory.assets/image-20220218175326032.png)

**缺页步骤**

![image-20220218175449235](Chapter 9：Virtual Memory.assets/image-20220218175449235.png)

![image-20220218175545872](Chapter 9：Virtual Memory.assets/image-20220218175545872.png)

### 9.6.1 Integrating Caches and VM

![image-20220218190454000](Chapter 9：Virtual Memory.assets/image-20220218190454000.png)

翻译后先从告诉缓存中查找，找不到再去内存找。

### 9.6.2 Speeding Up Address Translation with a TLB

翻译后备缓冲器(Translation Lookaside Buffer)

![image-20220218190813466](Chapter 9：Virtual Memory.assets/image-20220218190813466.png)

![image-20220218191011050](Chapter 9：Virtual Memory.assets/image-20220218191011050.png)

命中

![image-20220218191045874](Chapter 9：Virtual Memory.assets/image-20220218191045874.png)

不命中则从L1缓存中取出相应PTE

### 9.6.3 Multi-Level Page Tables

![image-20220218192937627](Chapter 9：Virtual Memory.assets/image-20220218192937627.png)

虚拟地址被划分为k个VPN和1个VPO。

![image-20220218193110710](Chapter 9：Virtual Memory.assets/image-20220218193110710.png)

### 9.6.4 Putting It Together: End-toEnd Address Translation

![image-20220218194019080](Chapter 9：Virtual Memory.assets/image-20220218194019080.png)

读虚拟地址0x03d4->00001111(VPN)011000(VPO)

得到VPO为0x14，PVN为0x0f

VPN中低2位为TLB索引0x3，高6位为TLB标记0x03

查TLB表得=>0x0D

得到物理地址为001011(CT)0110(CI)00(CO)

查告诉缓存得到索引5标记位为0d中偏移0有数据字节0x36，得到最终数据内容，传回给MMU传回CPU

## 9.7 Case Study :The Intel Core i7/Linux Memory System

### 9.7.1 Core i7 Address Translation

![image-20220218195804720](Chapter 9：Virtual Memory.assets/image-20220218195804720.png)

![image-20220218195855382](Chapter 9：Virtual Memory.assets/image-20220218195855382.png)

![image-20220218200340522](Chapter 9：Virtual Memory.assets/image-20220218200340522.png)

![image-20220218200432867](Chapter 9：Virtual Memory.assets/image-20220218200432867.png)

XD(禁止执行)位，禁止从某些内存页取指令。

A位(引用位)：每次访问页，设置

D位(修改位/脏位)：复制替换页时是否必须写回。

![image-20220218200805175](Chapter 9：Virtual Memory.assets/image-20220218200805175.png)

### 9.7.2 Linux Virtual Memory System

![image-20220218200955631](Chapter 9：Virtual Memory.assets/image-20220218200955631.png)

#### Linux Virtual Memory Areas

![image-20220218201616838](Chapter 9：Virtual Memory.assets/image-20220218201616838.png)

每个进程维护单独任务结构，任务结构包含或指向内核运行该进程所需所有信息(PID、用户栈指针、可执行目标文件名、程序计数器)

任务结构一条目指向mm_struct，描述虚拟内存当前状态。

pgd：指向第一级页表的基址

mmap：指向vm_area_struct(区域结构)的链表，描述当前虚拟空间的一个区域。

![image-20220218201948086](Chapter 9：Virtual Memory.assets/image-20220218201948086.png)

#### Linux Page Fault Exception Handling

1. 虚拟地址合法吗
2. 试图进行的内存访问合法吗
3. 选择牺牲页，交换。更新页表。

![image-20220218202243780](Chapter 9：Virtual Memory.assets/image-20220218202243780.png)

## 9.8 Memory Mapping

虚拟内存区域和磁盘对象关联，初始化虚拟内存区域

1. Linux文件系统中的普通文件。
2. 匿名文件。

### 9.8.1 Shared Objects Revisited（再看）

进程对区域任何写操作，共享的其他进程也是可见的

私有对象的改变，其他进程不可见。

![image-20220218202736703](Chapter 9：Virtual Memory.assets/image-20220218202736703.png)

只需要存放共享对象的一个副本。

私有对象使用“写时复制”技术：创建新副本，更新页表条目指向新副本，恢复可写权限。

![image-20220218203109959](Chapter 9：Virtual Memory.assets/image-20220218203109959.png)

### 9.8.2 The fork Function Revisiited

fork后，内核为新进程创建各种数据结构，分配卫衣PID。

创建mm_struct、区域结构和页表的原样**副本**(只读+写时复制)，虚拟内存也相同

两个进程任一写操作，写时复制创建新页面

### 9.8.3 The execve Function Revisited

`execve("a.out", NULL, NULL)`

加载并运行a.out如下步骤

![image-20220219102259482](Chapter 9：Virtual Memory.assets/image-20220219102259482.png)

![image-20220219102342333](Chapter 9：Virtual Memory.assets/image-20220219102342333.png)

### 9.8.4 User-Level Memory Mapping with the mmap Function

![image-20220219102503454](Chapter 9：Virtual Memory.assets/image-20220219102503454.png)

![image-20220219102709620](Chapter 9：Virtual Memory.assets/image-20220219102709620.png)

文件描述符fd在offset的位置以length的长度复制到虚拟内存中start的位置。

![image-20220219102759577](Chapter 9：Virtual Memory.assets/image-20220219102759577.png)

![image-20220219102915348](Chapter 9：Virtual Memory.assets/image-20220219102915348.png)

从start开始，由length删除

接下来对已删除区域引用会导致段错误。

##### 练习题9.5

![image-20220219103407300](Chapter 9：Virtual Memory.assets/image-20220219103407300.png)

![image-20220219103431906](Chapter 9：Virtual Memory.assets/image-20220219103431906.png)

检查参数，指定文件描述符fd

fstat：将参数fildes所指的文件状态，复制到参数buf所指的结构中(struct stat)

调用mmap进行映射，最后将bufp内容用write打印。

## 9.9 Dynamic Memory Allocation

![image-20220219105626357](Chapter 9：Virtual Memory.assets/image-20220219105626357.png)

动态内存分配器维护进程的**堆**区域

分配器风格：

![image-20220219105729205](Chapter 9：Virtual Memory.assets/image-20220219105729205.png)

### 9.9.1 The malloc and free Function

![image-20220219105827346](Chapter 9：Virtual Memory.assets/image-20220219105827346.png)

返回指向大小至少size字节内存块的指针。包含任何数据对象类型做对齐。

遇到问题返回NULL，设置errno

calloc：已初始化的动态内存，出师未为零

realloc：改变已分配块大小

![image-20220219110221186](Chapter 9：Virtual Memory.assets/image-20220219110221186.png)

将内核的brk指针增加incr扩展和收缩堆

![image-20220219110619420](Chapter 9：Virtual Memory.assets/image-20220219110619420.png)

ptr指向必须从mlloc、calloc或realloc获得已分配块的起始位置，否则free行为未定义。

**mlloc和free管理堆如下图**

![image-20220219110946985](Chapter 9：Virtual Memory.assets/image-20220219110946985.png)

(b)中由于双字对齐，malloc5个int需要分配6个int进行填充。

### 9.9.2 why Dynamic Memory Allocation？

直到程序运行才直到某些数据结构大小。

![image-20220219111405793](Chapter 9：Virtual Memory.assets/image-20220219111405793.png)

### 9.9.3 Allocator Requirements and Goals

![image-20220219111453758](Chapter 9：Virtual Memory.assets/image-20220219111453758.png)

目标1：最大化吞吐率

目标2：最大化内存利用率

互相牵制。

### 9.9.4 Fragmentation

内部碎片：已分配块比有效载荷大时发生(两者之差的和)。如9.34(b)

外部碎片：空闲内存合计起来足够满足一个分配请求，但没有单独空闲块足够大处理这个请求。如9.34(e)有6个字节空间但不连续。难以量化不可预测。

### 9.9.5 Implementation Issues

![image-20220219112757391](Chapter 9：Virtual Memory.assets/image-20220219112757391.png)

### 9.9.6 Implicit Free Lists

![image-20220219113944967](Chapter 9：Virtual Memory.assets/image-20220219113944967.png)

头部(块大小、是否已分配)、有效载荷、额外的填充

双字对齐后3位都是0，所以只需要29高位，释放后三位编码其他信息

![image-20220219114342517](Chapter 9：Virtual Memory.assets/image-20220219114342517.png)

隐式空闲链表，用链表来把各个已分配和空闲的块连接起来。

优点简单，缺点操作开销。

最小分配块为双字，只请求一字节仍分配两字节。

##### 练习题9.6

![image-20220219115546136](Chapter 9：Virtual Memory.assets/image-20220219115546136.png)

头部占4个字节，并且8字节对齐。

所以第一行  2+4=6，向8对齐,依此类推如下表所示

| 8    | 0x9  |
| ---- | ---- |
| 16   | 0x11 |
| 24   | 0x19 |
| 24   | 0x19 |

### 9.9.7 Placing Allocated Blocks

放置策略：

* 首次适配：从头开始搜索空闲链表
* 下一次适配：上次查询结束位置开始
* 最佳适配：检查每块，找到合适的最小

### 9.9.9 Getting Additional Heap Memory

1. 合并堆内存
2. 调用sbrk函数，申请额外的堆内存

### 9.9.10 Coalescing Free Blocks

![image-20220219120658311](Chapter 9：Virtual Memory.assets/image-20220219120658311.png)

立即合并：释放时合并(容易产生抖动)

推迟合并：等稍晚时间再合并

### 9.9.11 Coalescing with Boundary Tags（合并边界标记）

边界标记：在每个块尾部添加脚部，头部的副本，检查上一个块脚部，判断位置和状态。

![image-20220219144837070](Chapter 9：Virtual Memory.assets/image-20220219144837070.png)

![image-20220219145133132](Chapter 9：Virtual Memory.assets/image-20220219145133132.png)

四种情况对应下图4种处理方案。

![image-20220219145116360](Chapter 9：Virtual Memory.assets/image-20220219145116360.png)

潜在的缺陷就是多了内存开销

### 9.9.12 Putting It Together: Implementing a Simple Allocator

#### General Allocator Design

mem_heap和mem_brk之间的字节表示已分配的虚拟内存。

![image-20220219155936576](Chapter 9：Virtual Memory.assets/image-20220219155936576.png)

mm_init：初始化分配器

mm_malloc和mm_free和系统的有相同接口和语义

![image-20220219154311239](Chapter 9：Virtual Memory.assets/image-20220219154311239.png)

第一个字双边对齐不使用的填充自字

后面跟着特殊**序言块**，8字节已分配块，一个头部一个脚部组成。初始化时创建，永不释放。

中间都是可以分配的普通块

特殊**结尾快**：大小为零已分配块，只由一个头部组成。

#### Basic Constants and Macros for Manipulating the Free List

![image-20220219160800579](Chapter 9：Virtual Memory.assets/image-20220219160800579.png)

9行：大小和已分配位结合起来返回值

12行：读取和返回参数p引用的字

16-17行：地址p处头部或脚部返回大小和已分配位

剩下的是块指针操作。

#### Creating the Initial Free List

![image-20220219161204755](Chapter 9：Virtual Memory.assets/image-20220219161204755.png)

创建一个空的空闲链表，调用extend_heap扩展单字或双字对齐

extend_heap调用：1.堆初始化，2.请求额外的对空间(7~9行)

![image-20220219161717815](Chapter 9：Virtual Memory.assets/image-20220219161717815.png)

![image-20220219161726254](Chapter 9：Virtual Memory.assets/image-20220219161726254.png)

#### Freeing and Coalescing Blocks

![image-20220219163004410](Chapter 9：Virtual Memory.assets/image-20220219163004410.png)

序言块和结尾块也很好解决了麻烦边界问题。

#### Allocating Blocks

![image-20220219164241183](Chapter 9：Virtual Memory.assets/image-20220219164241183.png)

12-13：强制最小块大小

15：超过8字节请求

18：找合适块

19：可选分割

24-26：不能匹配，扩展堆

##### 练习题9.8 9.9

![image-20220219164111609](Chapter 9：Virtual Memory.assets/image-20220219164111609.png)

![image-20220219164313676](Chapter 9：Virtual Memory.assets/image-20220219164313676.png)

### 9.9.13 Explicit Free Lists

![image-20220219164654843](Chapter 9：Virtual Memory.assets/image-20220219164654843.png)

每个空闲块包含前驱和后继指针

时间从块总数到空闲块数量的现行时间。

空闲块排序策略：

1. 后进先出
2. 地址顺序：每个块地址都咸鱼后继地址（更高内存利用率）

### 9.9.14 Segregated Free Lists

维护多个空闲链表，每个链表的块有大致相等的大小。

![image-20220219172200875](Chapter 9：Virtual Memory.assets/image-20220219172200875.png)

#### Simple Segregated Storage

每个块就是这个大小类中最大元素的大小。  

{17-32}  则都是32大小的块

为空：系统申请

释放：插空闲链表头部

已分配的块不需要头脚部、链表单向

![image-20220219172651997](Chapter 9：Virtual Memory.assets/image-20220219172651997.png)

![image-20220219172714717](Chapter 9：Virtual Memory.assets/image-20220219172714717.png)

#### Segregated Fits(分离适配)

分配分割后剩余部分插入到适当的空闲链表。

#### Buddy System（伙伴系统

每个大小类都是2的幂

找到第一个可用，递归二分割块，放置响应空闲列表。

![image-20220219184142280](Chapter 9：Virtual Memory.assets/image-20220219184142280.png)

## 9.10 Gabage Collection

垃圾收集器：动态内存分配器，自动释放程序不需要的分配块。

### 9.10.1 Garbage Collector Basics

有向可达图，根节点和一组堆节点。

从根节点出发不可达，则为**垃圾**

![image-20220219184559493](Chapter 9：Virtual Memory.assets/image-20220219184559493.png)

释放，返回空闲链表，定期回收

对于C和C++不能精确维持可达图，不可达节点可能被标记成可达。

malloc找不到合适的空闲块，就调用垃圾回收器(代替应用调用free)，回收一些垃圾到空闲链表。失败申请额外内存空间。

### 9.10.2 Mark & Sweep Garbage Collectors

标记清除阶段组成。

调用mark函数标记，标记阶段结束，任何未标记已分配都认为不可达。

调用sweep函数，释放所有遇到未标记的分配块

![image-20220219185430952](Chapter 9：Virtual Memory.assets/image-20220219185430952.png)

![image-20220219185557528](Chapter 9：Virtual Memory.assets/image-20220219185557528.png)

### 9.10.3 Conservative Mark&Sweep for C Programs

平衡二叉树，左子树较小地址，右子树较大地址

![image-20220219185849310](Chapter 9：Virtual Memory.assets/image-20220219185849310.png)

可能不正确标记不可达的块，可能不会释放垃圾，导致不必要的外部碎片。

C语言收集器必须保守，因为不会用类型信息标记内存位置。

因此，像int、float可以伪装成指针。

垃圾回收期无法推断数据是int还是指针，必须保守标记可达。

## 9.11 Common Memory-Related Bugs in C Programs

### 9.11.1 Dereferencing Bad Pointers（间接引用坏指针）

![image-20220219191851737](Chapter 9：Virtual Memory.assets/image-20220219191851737.png)

### 9.11.2 Reading Unintialized Memory

![image-20220219192650928](Chapter 9：Virtual Memory.assets/image-20220219192650928.png)

误认为分配的y已经初始化为零，

应该进行显式的初始化。

### 9.11.3 Allowing Stack Buffer Overflows

![image-20220219192812762](Chapter 9：Virtual Memory.assets/image-20220219192812762.png)

gets复制任意长度串到缓冲区，必须用fgets限制输入串长度。

### 9.11.4 Assuming That Pointers and the Objects They Point to Are the Same Size(假设指针和它们指向的对象是相同大小的)

![image-20220219193033568](Chapter 9：Virtual Memory.assets/image-20220219193033568.png)

创建的int跟实际需要的int *在机器上大小不相等，则会超出A数组结尾位置。

### 9.11.5 Making Off-by-One Errors(错位错误)

![image-20220219193245949](Chapter 9：Virtual Memory.assets/image-20220219193245949.png)

创建n个元素指针，但是试图初始化n+1个元素。

### 9.11.6 Referencing a Pointer Instead of the Object It Points To（引用指针而不是指向的对象）

![image-20220219193438850](Chapter 9：Virtual Memory.assets/image-20220219193438850.png)

由于一元运算符--和*优先级相同，右向左结合，所以减少的是指针自己的值。

应该用(*size)--

### 9.11.7 Misunderstanding Pointer Arithmetic

![image-20220219193704184](Chapter 9：Virtual Memory.assets/image-20220219193704184.png)

指向数字的大小进行计算，而不是指针大小的字节计算。

### 9.11.8 Referencing Nonexistent Variables

![image-20220219193743680](Chapter 9：Virtual Memory.assets/image-20220219193743680.png)

栈在返回一个指针，这个指针值可能被其他函数调用溪修改，那么实际返回的又会是另一个栈帧条目

### 9.11.9 Referencing Data in Free Heap Blocks

![image-20220219194019668](Chapter 9：Virtual Memory.assets/image-20220219194019668.png)

在10行释放后再14行又引用了(也可能分配了其他数据)。

### 9.11.10 Introducing Memory Leaks

![image-20220219194204018](Chapter 9：Virtual Memory.assets/image-20220219194204018.png)

分配了堆块，不释放就返回

## 9.12 Summary

虚拟寻址引用主存：处理器产生虚拟地址，发送主存前翻译成物理地址，

三个作用

1. 自动缓存最近存放磁盘的虚拟地址空间内容，缺页则从磁盘复制
2. 简化内存管理、链接、共享数据、进程内存分配、程序加载、保护内存
3. TLB页表可以消除L1上页表条目开销

内存映射：讯内存和磁盘文件关联。  

mmap手工创建删除虚拟地址空间

malloc动态分配。

分配器：显式释放，隐式垃圾回收

有各种常见错误

