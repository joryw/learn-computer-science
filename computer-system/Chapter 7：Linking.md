# Chapter 7：Linking

## 7.1 Compiler Drivers

![image-20220128153916501](Untitled.assets/image-20220128153916501.png)

首先C预处理器(C preprocessor cpp)将main.c翻译成ASCII码的中间文件main.i

接下来C编译器(C compiler ccl)，将main.i翻译成ASCII汇编文件main.s

最后汇编器(assembler as)将main.s翻译成可重定位目标文件main.o

通过链接器程序ld组合，创建可执行目标文件。

调用加载器函数，将代码和数据复制到内存。

## 7.2 Static Linking

1. Symbol resolution（符号解析）：目标文件定义和引用符号，每个符号对应一个函数、全局变量或静态变量。作用时将每个符号引用正好和一个符号定义关联
2. Relocation（重定位）：每个符号定义与内存位置关联起来。链接器产生重定位条目指令，执行重定位。

## 7.3 Object Files

Relocatable object file：与其他可重定位目标文件合并，创建可执行目标文件

Executable object file：可被直接复制到内存并执行

Shared object file：加载或运行时被动态加载进内存并链接。

编译器生成可重定位目标文件，链接器生成可执行目标文件。

目标文件是以文件形式存放在磁盘的目标模块(字节序列)

## 7.4 Relocatable Object Files

![image-20220128160338033](Untitled.assets/image-20220128160338033.png)

ELF头：描述生成该文件系统的字大小和字节顺序。剩余部分帮助链接器语法分析和解释目标文件信息

.text：已编译程序的机器代码

.rodata：只读数据，如printf语句的格式串和跳转表

.data：已初始化的全局和静态C变量。(局部变量运行时保存在栈中)

.bss：未初始化的静变量，以及所有被初始化为0的全局或静态变量。(占位符，不占据磁盘空间，内存分配时初始化为0。)

.symtab：符号表，定义和引用函数和全局变量信息。

.rel.text：一个.text节中位置的列表，和其他文件组合时需要修改。调用外部函数或引用全局变量都需要修改。

.rel.data：被模块引用或定义的所有全局变量的重定位信息。初始化时全局变量地址或外部定义函数地址都需要被修改。

.debug：调试符号表，定义的局部变量和类型定义，定义和引用的全局变量，原始C源文件。 -g调用才得到

.line：C源程序行号和.text节机器指令之间映射。 -g调用得到

.strtab：字符串表：包括.symtab和.debug中的符号表，节头部中的节名字。

节头部表：不同节位置和大小的描述。

## 7.5 Symbols and Symbol Tables

* 由模块m定义并能被其他模块引用的全局符号：

* 由其他模块定义并被模块m引用的全局符号：其他模块定义非静态C函数和全局变量
* 只被模块m定义和引用的局部符号：带static的C函数和全局变量

不包含本地非静态变量的任何符号，在栈中管理。

用编译器输出到汇编语言.s文件中的符号

![image-20220128165215685](Untitled.assets/image-20220128165215685.png)

name：字节偏移

value：符号地址

size：目标大小

type：数据或函数

binding：本地或者全局

section：分配目标文件的某个节

特殊的伪节：ABS：不该被重定位的符号、UNDEF：非定义的符号、COMMON：未初始化的全局变量

![image-20220129154711940](Untitled.assets/image-20220129154711940.png)

![image-20220202143127055](Chapter 7：Linking.assets/image-20220202143127055.png)

![image-20220202143204566](Chapter 7：Linking.assets/image-20220202143204566.png)

buf在main中声明，但实际上被用于外部调用，所以Symbol type 类型是外部类型。

bufp0指针地址引用，但在swap中作为全局变量使用。

## 7.6 Symbol Resolution（符号解析）

### 7.6.1 How Linkers Resolve Duplicate Symbol Names

函数和已初始化的全局变量是强引用，未初始化的全局变量是弱引用。

1. 不允许多个同名的强符号
2. 一个强符号和多个弱符号同名，那么选择强符号
3. 多个弱符号同名，那么从弱符号任意选一个



![image-20220202145258017](Chapter 7：Linking.assets/image-20220202145258017.png)

多个同名函数强符号(规则1)，不可用

![image-20220202145343887](Chapter 7：Linking.assets/image-20220202145343887.png)

多个同名全局变量强符号(规则1)，不可用

![image-20220202145501018](Chapter 7：Linking.assets/image-20220202145501018.png)

在一个模块x未初始化，则选择另一个模块定义的强符号(规则2)。

运行时函数f将x值做了修改，会给原模块带来破坏

![image-20220202150908313](Chapter 7：Linking.assets/image-20220202150908313.png)

两个弱引用，由于规则3，会任意选一个，造成不易察觉的错误。

![image-20220202151136282](Chapter 7：Linking.assets/image-20220202151136282.png)

一个用int，4个字节。 一个用double，8个字节。

f函数将x赋值为-0.0会将内存中x和y的内容覆盖。得到如下错误

![image-20220202151420001](Chapter 7：Linking.assets/image-20220202151420001.png)

这个错误只会触发连接器发出一条警告。执行很久才表现出来。

当编译器遇到弱全局，不确定其他模块是否定义了x， 所以把x分配给COMMON，决定权交给连接器。

x初始化为0则为强符号，则可以自信的分配给.bss

静态符号的构造必须唯一，可以分配成.data或.bss

### 7.6.2 Linking with Static Libraries

将所有相关的目标模块打包成为一个单独的文件，成为静态库(Static library)

如果不适用静态库

1. 让编译器辨认对标准函数的调用，直接生成对应的代码(增加编译器复杂性)
2. 将所有标准C函数放在单独可重定位目标模块，链接到可执行文件。(每个都要放副本到内存，内存浪费，修改也困难)
3. 每个标准函数创建独立可重定位文件，要求程序员显式链接合适的目标模块到她们的可执行文件，耗时容易出错

静态库通过指定单独文件使用库中定义的函数，只赋值被程序引用的目标模块

![image-20220202155604570](Chapter 7：Linking.assets/image-20220202155604570.png)

定义两个libvector库中的成员目标文件，通过AR工具，加入到这个库中

![image-20220202155653439](Chapter 7：Linking.assets/image-20220202155653439.png)

为使用这个库，调用addvec，vector.h定义了libvector.a中例程的函数原型

![image-20220202155344537](Chapter 7：Linking.assets/image-20220202155344537.png)

编译和链接输入文件main.o和libvector.a

![image-20220202154901830](Chapter 7：Linking.assets/image-20220202154901830.png)

复制addve.o到可执行文件，并且复制libc.a中的printf.o

### 7.6.3 How Linkers Use Static Libraries to Resolve References

连接器维护可重定位目标文件集合E，未解析的符号集合U，前面输入文件中已定义的符号集合D。初始时均为空。

* 链接器判断目标文件，添加到E，修改U和