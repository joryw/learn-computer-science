# Chapter 3：Machine-Level Representation of Programs

摩尔定律：晶体管数量每18个月翻一倍

## 3.2 Program Encodings

### 3.2.1Machine-Level Code

* 程序计数器(Program counter)：指示下个要执行的地址
* 寄存器文件(register file)：16个，存地址或整数数据。
* 条件码(condition code)：最近执行的算术或逻辑指令的状态信息
* 一组向量寄存器：一个或多个整数或浮点数的值

程序内存：可执行机器代码、操作系统需要的信息、管理过程调用和返回的运行时栈，用户分配的内存块。

## 3.3 Data Formats

![image-20220105153447943](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105153447943.png)

## 3.4 Accessing Information

![image-20220105153807978](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105153807978.png)

### 3.4.1 Operand Specifiers(操作数指示符)

![image-20220105154149637](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105154149637.png)

立即数：表示常数值

寄存器：表示某个寄存器的此内容

内存引用：根据计算的地址访问某个内存地址。

### 3.4.2 Data Movement Instructions

移位操作指令，根据后缀确定移动的字节数。

**movl作为以寄存器作为目的时，把该寄存器高位4字节设置为0**

最后一条指令用于处理64位立即数数据。常规的movq只能表示32位补码数字立即数作为源操作数。

![image-20220105155541355](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105155541355.png)

**五种操作组合**

![image-20220105155840966](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105155840966.png)

**示例**

![image-20220105160859124](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105160859124.png)

较小源值复制到较大目的时，有一下两类数据移动指令。

movz：零扩展，把剩余字节填充为0

movs：符号扩展，源操作最高位进行复制。

其中源为**寄存器或内存**，目的为**寄存器**

![image-20220105160536172](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105160536172.png)

![image-20220105160547593](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105160547593.png)

**注意：没有把4字节源零扩展到8字节目的指令，可以用movl实现**

cltq指令，总是以寄存器%eax作为源，%rax作为符号扩展结果的目的。与`movslq %eax, %rax`一致

**示范**

![image-20220105163413294](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105163413294.png)

**练习3.3**

![image-20220105165211051](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105165211051.png)

EBX 是**"基地址"(base)**寄存器, 在内存寻址时存放基地址。

### 3.4.3 Data Movement Example

**练习题3.4**

![image-20220105170654519](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105170654519.png)

![image-20220105170701381](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105170701381.png)

1. 如果从字节小->字节大
   1. 有符号->无符号，进行符号扩展
   2. 有符号->无符号，进行零扩展
2. 如果从字节大->字节小，不用进行扩展，使用源长度后缀复制给rax，再根据目的长度，调整后缀。

**练习题3.5**

![image-20220105171514564](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105171514564.png)

解析：将内存地址的内容（用*表示地址的内容），用寄存器暂存，然后再将寄存器的内容，存入另一个内存地址，作为地址内容。

### 3.4.4 Pushing and Popping Stack Data

1. 后进先出
2. 对栈顶进行插入和删除
3. 栈向下增长，栈顶地址是最低的。栈顶在底部

![image-20220105172708196](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105172708196.png)

压栈等价于下述内容

![image-20220105172450869](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105172450869.png)

弹栈等价下述

![image-20220105172625451](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105172625451.png)

## 3.5 Arithmetic and Logical Operations

每个指令都带有不同大小操作数变种(如addb、addw、addl、addq)

分为4组：**加载有效地址、一元操作、二元操作和移位**

![image-20220105173000509](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105173000509.png)

### 3.5.1 Load Effective Address

实际上是movq操作，并诶呦引用内存，而是将有效地址写入目的操作数。

假如：%rdx值为x，则` leaq 7(%rdx,%rdx,4), %rax`将%rax设置为5x+7。目的操作数必须是寄存器。

**案例**

![image-20220105173734800](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105173734800.png)

可以理解为是对寄存器的**地址计算的结果**，然后复制给另一个寄存器。

而传统的移位操作会将计算后的**地址内容**，复制过去。

### 3.5.2 Unary and Binary Operations (一元和二元操作)

 the instruction `subq %rax,%rdx` decrements register %rdx by the value in %rax  （目的为被除数，源为除数）

第一个操作数：立即数、寄存器或内存地址

第二个操作数：寄存器或内存地址

### 3.5.3 Shift Operations

当寄存器为0xFF，salb移7位，salw移15位，sall移31位，salq移63位

### 3.5.4 Discussion

**练习题3.11**

![image-20220105195231107](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105195231107.png)

### 3.5.5 Special Arithmetic Operations

![image-20220105200025083](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105200025083.png)

1. 由%rdx(高64)和%rax(低64)共同组成的128位
2. 要求由一个参数必须在寄存器%rax中，另一个作为源操作数给出。
3. 除法需要将参数存在rax，对rax使用cqto对高64位进行符号扩展，得到128位的%rdx和%rax结合。最终模数存在%rdx，结果存在%rax

**乘法案例：**

![image-20220105200353373](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105200353373.png)

**除法案例：**

需要将%rdx存的数据暂存到另一个位置，将%rdx参与除法计算

![image-20220105200823972](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105200823972.png)

**练习题3.12**

![image-20220105201855307](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105201855307.png)

无符号扩展，采用movl $0, %rdx。将高8位设置为0。

最后再用无符号除法指令。

## 3.6 Control

### 3.6.1 Condition Codes

![image-20220105202205693](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105202205693.png)

**案例**

![image-20220105202355308](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105202355308.png)

**leadq指令不改变任何条件码。其他图3-10列出指令都会设置条件码**

![image-20220105203304893](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105203304893.png)

1. CMP跟SUB行为一样，但是不改变目的寄存器，只设置条件码

2. TEST类似AND，一般用来测试两个操作数是否相同（`testq %rax. %rax` 检查%rax是负数、零还是正数）。或者一个是掩码，指示哪些位应该测试。

### 3.6.2 Accessing the Condition Codes

1. 某些组合，将字节设置为0或1
2. 条件跳转
3. 条件传送数据

![image-20220105203605702](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105203605702.png)

**案例：**

![image-20220105204050944](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105204050944.png)

#### 练习题3.13

![image-20220105205421880](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105205421880.png)

![image-20220105205430068](Chapter 3：Machine-Level Representation of Programs.assets/image-20220105205430068.png)

通过cmp跟set两个分别判断位数及有无符号。

### 3.6.3 Jump Instructions

直接(`jmp .L1`)/间接(`jmp *%rax`、`jmp *(%rax)`)

![image-20220106105854881](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106105854881.png)

### 3.6.4 Jump Instruction Encodings

![image-20220106110310869](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106110310869.png)

链接后，重定位了地址

![image-20220106110527110](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106110527110.png)

**rep ret作用：**

通过跳转到达ret，处理器不能正常处理，所以只能通过在rep进行空操作，再执行ret，此时不改变代码其他行为。

#### 练习题3.15

![image-20220106112250674](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106112250674.png)

jmp操作的下一个地点作为起点，

字节码里最后一部分作为指令下一个地点的偏移量。

进行求和，得到跳转后的地址

### 3.6.5 Implementing Conditional Branches with Conditional Control（条件控制实现条件分支）

![image-20220106113237237](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106113237237.png)

使用goto语句描述汇编代码程序控制流。实际上是一个不好的变成风格，使代码难读和调试。

![image-20220106113515265](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106113515265.png)

从汇编的角度，会产生以上的语句。

#### 练习题3.16

![image-20220106114825796](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106114825796.png)

对于&&中的多个条件判断，需要拆分成多个if分支，逐个验证。

#### 练习题3.17

![image-20220106120056912](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106120056912.png)

![image-20220106120103353](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106120103353.png)

这里可以模仿一下3.16(b)的书写方式。B问要关注的是没有else的情况下哪种能减少跳转。

### 3.6.6 Implementing Conditional Branches with Conditional Moves

![image-20220106143501994](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106143501994.png)

条件数据传送性能更好，（条件跳转，只有分支条件求职完成，才能决定分支往哪走。猜测错误会浪费时钟周期，导致性能下降）

**预测错误的处罚**

![image-20220106143959592](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106143959592.png)

**条件传送指令**

![image-20220106144542655](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106144542655.png)

**示例：**

![image-20220106144634117](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106144634117.png)

![image-20220106144716952](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106144716952.png)

**反例：**

![image-20220106145008481](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106145008481.png)

在这个案例中，当测试为假时，由于在mov对xp进行了间接引用，间接导致了引用空指针的错误。这种情况是非法的。所以必须要用分支代码编译。

在选择哪种条件分支中，需要权衡浪费的计算和分支预测错误造成性能处罚的相对性能。

经验而言，即时分支预测错误开销大于复杂计算，GCC还是会使用条件控制转移。

##### 练习题3.20

![image-20220106151454353](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106151454353.png)

进行除法计算的时候，由于需要进行舍入处理，需要先添加一定的偏置量，再进行除法计算。

##### 练习题3.21

![image-20220106152423374](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106152423374.png)

![image-20220106152454121](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106152454121.png)

注意：当汇编语言翻译为x >= y，则实际上在C语言中是x<y的条件

### 3.6.7 Loops

#### Do-While Loops

![image-20220106152705017](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106152705017.png)

##### 示例

![image-20220106152810919](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106152810919.png)

##### 练习题3.22

![image-20220106153122008](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106153122008.png)

阶乘表：

![image-20220106153141494](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106153141494.png)

通过两种方式判断

1.  x/n不等于 (n - 1) !
2. 是否为100的倍数

数据long计算，直到20！才溢出。

#### While Loops

格式：

![image-20220106154850085](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106154850085.png)

示例：

![image-20220106154934144](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106154934144.png)

另一种是翻译成do-while形式

![image-20220106155749562](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106155749562.png)

示例：

![image-20220106155717691](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106155717691.png)

##### 练习题3.25

![image-20220106160335510](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106160335510.png)

![image-20220106160355726](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106160355726.png)

##### 练习题3.25

![image-20220106161842965](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106161842965.png)

#### For Loops

for循环产生的代码是while循环两种翻译之一，取决于优化的等级

![image-20220106162642941](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106162642941.png)

##### 练习题3.28

![image-20220106164653886](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106164653886.png)

**答案:**

![image-20220106164714508](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106164714508.png)

##### 练习题3.29

![image-20220106165640235](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106165640235.png)

这里需要考虑continue在statement中，会跳过i++操作，导致无法进行更新i值，所以使用goto，跳转到更新位置。

#### 3.6.8 switch Statements

多重分支，跳转表数据结构更高效（当开关数量多，值范围跨度小使用）

![image-20220106172344981](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106172344981.png)

通过`subq &100 %rsi`，调整index范围在0~6。 大于6直接跳转到默认代码处

跳转表如下图

![image-20220106172740656](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106172740656.png)

##### 练习题3.31

![image-20220106200659636](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106200659636.png)

## 3.7 Procedures

### 3.7.1 The Run-Time Stack

![image-20220106185649298](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106185649298.png)

### 3.7.2 Control Transfer

![image-20220106185753461](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106185753461.png)

**详细示例**

![image-20220106190146114](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106190146114.png)

每个栈的长度为0x08

### 3.7.3 Data Transfer

寄存器传参(参数1-6)

![image-20220106195013669](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106195013669.png)

参数7-n放在栈上

栈传递方式

![image-20220106195325270](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106195325270.png)

##### 练习题3.33

![image-20220106200716379](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106200716379.png)

![image-20220106200804218](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106200804218.png)

### 3.7.4 Local Storage  on the Stack

局部数据存在内存情况

* 寄存器不够存
* 使用地址运算符&
* 数组或结构，通过引用访问

**示例**

![image-20220106204209754](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106204209754.png)

在调用proc之前，先将函数参数加载到寄存器。

![image-20220106204330794](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106204330794.png)

参数7和8位于栈指针偏移量为8和16位置。

#### 3.7.5  Local Storage in Registers

保存寄存器的值，要么不改变，要么压栈，返回前弹出。

除了栈指针%rsp，都分类为调用者保存寄存器。任何函数都能修改。

**示例：**

![image-20220106205639104](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106205639104.png)

将%rpb和%rbx的值压入栈中暂存，然后将x和返回的结果存入到寄存器%rpb和%rbx中进行暂存。计算完毕后，将压入栈中的数据弹回给寄存器。

第一次调用，保存x的值，第二次调用，保存Q(y)的值

#### 3.7.6 Recursive Procedures

1. 要用到%rbx寄存器，先将寄存器原来的值存在栈上，最后恢复
2. 将调用函数返回结果存在%rax寄存器
3. 调用函数自身，形成递归

![image-20220106210329491](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106210329491.png)

## 3.8 Array Allocation and Access

### 3.8.1 Basic Orinciples

![image-20220106211255872](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106211255872.png)

### 3.8.2 Pointer Arithmetic

如表所示：计算指针地址和指针值

![image-20220106211737571](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106211737571.png)

### 3.8.3 Nested Arrays

![image-20220106213037315](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106213037315.png)

行优先存储

![image-20220106213138025](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106213138025.png)

![image-20220106213238583](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106213238583.png)

![image-20220106213731719](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106213731719.png)

### 3.8.4 Fixed-Size Arrays

![image-20220106214353293](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106214353293.png)

示例：

![image-20220106215519116](Chapter 3：Machine-Level Representation of Programs.assets/image-20220106215519116.png)

### 3.8.5 Variable-Size Arrays

![image-20220107134720862](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107134720862.png)

## 3.9 Heterogeneous Data Structures(异质的数据结构)

结构(structure)

联合(union)

### 3.9.1 Structures

![image-20220107135028736](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107135028736.png)

![image-20220107135044219](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107135044219.png)

**案例：**

![image-20220107135505508](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107135505508.png)

##### 练习题3.41

![image-20220107141142030](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107141142030.png)

##### 练习题3.42

![image-20220107142553239](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107142553239.png)

### 3.9.2 Unions

多种类型引用一个对象（即联合的对象在同样长度的数据，可以有多种不同的类型表示）

![image-20220107143814985](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107143814985.png)

![image-20220107143826446](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107143826446.png)

下面的例子，通过一个枚举，来表示这是一个内部节点还是叶子节点。

* 假如是内部节点，则会有左右子树，分别为8个字节，
* 假如是叶子节点，则会有节点值，由16个字节的double数组组成。

type占4个字节，type跟联合之间需要4个字节填充，一共需要**24个字节**

![image-20220107144010986](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107144010986.png)

字节顺序也很重要，下面的例子，如果是小端机器，word0是低4位，word1是高4位

![image-20220107144353261](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107144353261.png)

##### 练习题3.43

![image-20220107150551597](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107150551597.png)

![image-20220107151141189](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107151141189.png)

### 3.9.3 Data Alignment（数据对齐）

任何K字节基本对象(起始)地址必须是K的倍数(由前后类型字节的长度决定当前字节的对齐量)

![image-20220107151348862](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107151348862.png)

如下例，有int类型， 所以整体需要按照4字节对齐

![image-20220107151542667](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107151542667.png)

按照最小字节则总共需要9个字节，不符合对齐原则，通过增加间隙。最终达到12字节

![image-20220107151649501](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107151649501.png)

同理

![image-20220107151707724](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107151707724.png)

![image-20220107151717812](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107151717812.png)

> 指针的字节数
>
> 指针即为地址，指针几个字节跟语言无关，而是跟系统的寻址能力有关。譬如以前是16为地址，指针即为2个字节，现在一般是32位系统，所以是4个字节，以后64位，则就为8个字节

##### 练习题3.44

![image-20220107155106125](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107155106125.png)

* 不理解为什么`struct P4 a[2]`占用24个字节。

##### 练习题3.45

![image-20220107164104576](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107164104576.png)

![image-20220107164128360](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107164128360.png)

## 3.10 Combining Control and Data in Machine-Level Programs(在机器级程序中将控制与数据结合起来)

### 3.10.1 Understanding Pointers

`int *ip`  指向int类型的指针

`char **cpp` 指向的对象本身就是一个char类型的指针

`void *` 通用指针

* 指针值：对象的地址
* 用&运算符创建：放在赋值左边。
* 用于间接引用指针：结果为（内存引用）值
* 数组与指针紧密联系：数组引用`a[3]`与指针运算和间接引用`*(a + 3)`一样效果
* 强制类型转换，只改变类型，不改变值。`(int *)p+7=p+28` 而`(int *)(p + 7)=p+7`  强转优先级高于加法。
* 指针可指向函数

### 3.10.2 Using the GDB Debugger

![image-20220107170831081](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107170831081.png)

### 3.10.3 Out-of Bounds Memory References and Buffer Overflow(内存越界和缓冲区溢出)

gets没法确定保存整个字符串分配了足够的空间。

分配8个字节，只要输入不超过7个字节（包括结尾null），能放入buf分配的空间，过长会覆盖某些信息

![image-20220107171222283](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107171222283.png)

超过23会破坏返回指针的值

![image-20220107171340928](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107171340928.png)

#### 练习题3.46

![image-20220107174250941](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107174250941.png)

![image-20220107174307361](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107174307361.png)

![image-20220107174320708](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107174320708.png)

1. 注意压栈操作会分配空间暂存寄存器内容。
2. 根据栈分配的空间大小，开始输入字符串长度。
3. 不要忽略字符串结尾的null，为0x00

### 3.10.4 Thwarting Buffer Overflow Attacks(对抗缓冲区溢出攻击)

#### Stack Randomization

栈位置在程序每次运行都变化

地址空间布局随机化(ASLR)： 每次运行程序，程序代码、库函数、栈、全局变量和堆数据，都会加载到不同区域。

通过插入大量nop，对程序计数器加1，只要能猜中某个地址，就能到达攻击代码。

##### 练习题3.47

![image-20220107191213273](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107191213273.png)

上面练习说明了栈随机化并不能很大程度阻止攻击。

#### Stack Corruption Detection（栈破坏检测）

栈保护机制：在栈中任意局部缓冲区与栈状态之间保存一个特殊的金丝雀(canary)值(哨兵值)。通过检查是否被改变，确定程序是否异常。

![image-20220107191619830](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107191619830.png)

下图案例，通过第3行从内存读出值，存入栈中偏移8位置。

`%fs:40`是段寻址方式，存在特殊段，标志为“只读”，不能被覆盖。

第11行用xorq比较金丝雀是否被修改过。

![image-20220107191947261](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107191947261.png)

##### 练习题3.48

![image-20220107194318547](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107194318547.png)

![image-20220107194214555](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107194214555.png)

#### Limiting Executable Code Regions(限制可执行代码区域)

编译器产生的代码是可执行的，其他部分限制为允许读写

“NX”位可以将检查页是否可执行交给硬件，可以减少性能损失

有些程序要求动态产生和执行代码能力。

取决于语言和操作系统。

### 3.10.5 Supporting Variable-Size Stack Frames（支持变长栈帧）

p数组长度由第一个参数n给出，所以每次调用函数分配的空间都不同

![image-20220107195613358](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107195613358.png)

上图第2行`movq %rsp, %rbp`可以将当前栈位置作为基指针保存。第3行分配16字节内存。（前8个存i，后8个未使用）

第5-11行在栈上分配8n字节，并放置好数组p

第15行看出，i保存在rbp-8位置。

第20行leave，将帧指针恢复之前的值



下图%rbp作为帧指针(基指针)：指向保存值那一时刻的栈位置，可以用固定长度的局部变量(如i)相对于%rbp偏移量引出。

![image-20220107195745881](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107195745881.png)

栈帧长可变情况下，才使用帧指针

##### 练习题3.49

![image-20220107215049927](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107215049927.png)

## 3.11 Floating-Point Code

![image-20220107220750202](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107220750202.png)

每个都是256位(32字节)，只用来保存浮点数，只使用低32位或64位

### 3.11.1 Floating-Point Movement and Coversion Operations（浮点传送和转换操作）

![image-20220107222810195](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107222810195.png)

内存和寄存器传送用vmovss和vmovsd。

寄存器和寄存器传送用movaps和vmovapd

![image-20220107223453493](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107223453493.png)

![image-20220107223513993](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107223513993.png)

先用xmm1暂存v1，xmm0作为返回值存储v2，最后再返回xmm0寄存器。

浮点数转整数会进行截断。有双操作数（浮点数转整数）和三操作数（整数转浮点数）

![image-20220107223859452](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107223859452.png)

两个浮点数之间转换：单精度转双精度如下

![image-20220107224903096](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107224903096.png)

vunpcklps交叉放置两个XMM寄存器的值，存在第三个寄存器中

[s3,s2,s1,s0]跟[d3,d2,d1,d0]得到[s1,d1,s0,d0]

![image-20220107224931899](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107224931899.png)

双精度转单精度

![image-20220107225137069](Chapter 3：Machine-Level Representation of Programs.assets/image-20220107225137069.png)

假设[x1,x0]，用vmovddup得到[x0,x0]，用vcvtpd2psx得到[0.0,0.0,x0,x0]，这样操作比起单个指令，更有明显意义

**示例**

![image-20220108144856498](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108144856498.png)

### 3.11.2 Floating-point Code in Procedures(过程中的浮点数)

* %xmm0~%xmm7传8个参数，按序排列，多的放栈
* %xmm0放返回值
* 所有XMM都是调用者保存，被调用者不用保存就覆盖任意一个

double *和long * 都是指针，用rdi而不是xmm0。

### 3.11.3 Floating-Point Arithmetic Operations

![image-20220108151702708](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108151702708.png)

![image-20220108151831324](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108151831324.png)

![image-20220108151844716](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108151844716.png)

##### 练习题3.53

![image-20220108152853796](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108152853796.png)

### 3.11.4 Defining and Using Floating-Point Constants

![image-20220108153621176](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108153621176.png)

小端法0位低四位，1077936128为高四位

### 3.11.5 Using Bitwise Operations in Floating-Point Code

![image-20220108153832620](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108153832620.png)

##### 练习题3.56

![image-20220108154549173](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108154549173.png)

![image-20220108154555796](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108154555796.png)

### 3.11.6 Floating-Point Comparison Operations

![image-20220108155615051](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108155615051.png)

S2在XMM寄存器，S1在XMM寄存器或内存

![image-20220108155749899](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108155749899.png)

CF零标志位，ZF进位标志位，PF奇偶标志位

当任一操作数为NaN，就会出现无序。

**示例**

![image-20220108160255831](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108160255831.png)

分为 0 (NEG), 1 (ZERO), 2 (POS), and 3 (OTHER)四种情况

##### 练习题3.57

![image-20220108162253157](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108162253157.png)

![image-20220108162317499](Chapter 3：Machine-Level Representation of Programs.assets/image-20220108162317499.png)

可以新建临时局部变量a、d去暂存参数*ap *dp，进行计算。

