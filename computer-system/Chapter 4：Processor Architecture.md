# Chapter 4：Processor Architecture

每个时钟周期执行一条完整的Y86-64指令

创建流水线化处理器，每条指令分解为五步

### 4.1 The Y86-64 Instruction Set Architecture

### 4.1.1 Programmer-Visible State

![image-20220108165402353](Chapter 4：Processor Architecture.assets/image-20220108165402353.png)

状态码Stat：表明程序执行的总体状态

### 4.1.2 Y86-64 Instructions

指令编码长度1到10个字节不等。fn是某个整数操作、数据传送条件、分支条件

![image-20220108165820927](Chapter 4：Processor Architecture.assets/image-20220108165820927.png)

立即数(i)、寄存器(r)、内存(m)。

第一个字母：源类型

第二个字母：目标类型

* 4个mov
* 4个整数操作：OPq
* 7个跳转指令：jXX
* 6个条件传送：comvXX

### 4.1.3 Instruction Encoding

高4为是代码部分，低4位是功能部分

![image-20220108170615212](Chapter 4：Processor Architecture.assets/image-20220108170615212.png)

寄存器文件：小的、以寄存器ID作为地址的随机访问存储器。

![image-20220108170723858](Chapter 4：Processor Architecture.assets/image-20220108170723858.png)

示例：

![image-20220108171025414](Chapter 4：Processor Architecture.assets/image-20220108171025414.png)

##### 练习4.1

![image-20220108173714130](Chapter 4：Processor Architecture.assets/image-20220108173714130.png)

按表翻译，注意小端法。练习4.2思路相同

#### CISC和RISC

![image-20220108173946378](Chapter 4：Processor Architecture.assets/image-20220108173946378.png)

### 4.1.4 Y86-64 Exceptions

![image-20220108174126878](Chapter 4：Processor Architecture.assets/image-20220108174126878.png)

处理器调用异常处理程序处理。

### 4.1.5 Y86-64 Programs

![image-20220108174437123](Chapter 4：Processor Architecture.assets/image-20220108174437123.png)

区别：

* 2-3行常数加载寄存器，不能用立即数
* 8-9行，内存读取将其于寄存器相加
* 设置了条件码，就不用testq指令。

用Y86-64汇编代码编写完整程序文件的例子

![image-20220108174640935](Chapter 4：Processor Architecture.assets/image-20220108174640935.png)

"." 汇编器伪指令

.pos 0 从地址0开始产生代码  39行的.pos指明栈地址0x200

8-13行，，生命了4个字的数组，分别为值。 .align指定8字节对齐

**汇编结果如下**

![image-20220108175202453](Chapter 4：Processor Architecture.assets/image-20220108175202453.png)

## 4.2 Logic Design and the Hardware Control Language HCL

### 4.2.1 Logic Gates

![image-20220108180604805](Chapter 4：Processor Architecture.assets/image-20220108180604805.png)

逻辑门只对单个位数进行操作

### 4.2.2 Combinational Circuits and HCL Boolean Expressions(组合电路和HCL布尔表达式)

限制

* 必须连接选项之一：1.一个系统输入  2.某个存储单元输出 3.某个逻辑门输出
* 两个或多个逻辑门输出不能连接一起
* 必须无环

![image-20220108192142327](Chapter 4：Processor Architecture.assets/image-20220108192142327.png)

![image-20220108192237809](Chapter 4：Processor Architecture.assets/image-20220108192237809.png)

异或

![image-20220108192427890](Chapter 4：Processor Architecture.assets/image-20220108192427890.png)

### 3.2.3 Word-Level Combinational Circuits and HCL Integer Expressions

A==B就是测试64位是否都相等

![image-20220108192603159](Chapter 4：Processor Architecture.assets/image-20220108192603159.png)

![image-20220108192911445](Chapter 4：Processor Architecture.assets/image-20220108192911445.png)

![image-20220108192923799](Chapter 4：Processor Architecture.assets/image-20220108192923799.png)

四路复用器

![image-20220108193205389](Chapter 4：Processor Architecture.assets/image-20220108193205389.png)

![image-20220108193152086](Chapter 4：Processor Architecture.assets/image-20220108193152086.png)

### 4.2.5 Memory and Clocking

* 时钟寄存器(寄存器)
* 随机访问寄存器(内存)

硬件寄存器：输入和输出连接的电路的其他部分

程序寄存器：可寻址的字

![image-20220108194314646](Chapter 4：Processor Architecture.assets/image-20220108194314646.png)

寄存器操作：时钟跳动进入下一个值

![image-20220108194336288](Chapter 4：Processor Architecture.assets/image-20220108194336288.png)

寄存器文件：两个读端口，一个写端口

![image-20220108194513555](Chapter 4：Processor Architecture.assets/image-20220108194513555.png)

随机访问存储器

## 4.3 Sequential Y86-64 Implementations

### 4.3.1 Organizing Processing into Stages

* 取指(fetch)：内存读取指令字节，地址是PC值
* 译码(decode)：从寄存器文件读入最多两个操作数
* 执行(execute)：ALU操作
* 访存(memory)：写入内存
* 写回(write back)： 最多写2个结果到寄存器文件
* 更新PC(PC update)：设置吓一跳指令地址

整数操作指令

![image-20220108195833900](Chapter 4：Processor Architecture.assets/image-20220108195833900.png)

![image-20220108202622210](Chapter 4：Processor Architecture.assets/image-20220108202622210.png)

![image-20220108202859449](Chapter 4：Processor Architecture.assets/image-20220108202859449.png)

![image-20220108203228677](Chapter 4：Processor Architecture.assets/image-20220108203228677.png)

### 4.3.2 SEQ Hardware Structure

![image-20220108203943742](Chapter 4：Processor Architecture.assets/image-20220108203943742.png)

完整结构：

![image-20220108204217755](Chapter 4：Processor Architecture.assets/image-20220108204217755.png)

* 白色方框：时钟寄存器
* 浅蓝色方框：硬件单元
* 灰色圆角：控制逻辑块
* 白色圆圈：线路名字
* 中等粗线：宽度为字长
* 细线：宽度为字节
* 虚线：单个位

![image-20220108204853459](Chapter 4：Processor Architecture.assets/image-20220108204853459.png)

更完整的计算。

### 4.3.3 SEQ Timing(SEQ 时序)

PRINCIPLE：No reading back

never needs to read back the **state** **updated** by an instruction in order to complete the processing of this instruction

也就是要读更新过的数据。任何指令试图读之前，都会更新。

示例：

![image-20220108205850516](Chapter 4：Processor Architecture.assets/image-20220108205850516.png)

![image-20220108205903064](Chapter 4：Processor Architecture.assets/image-20220108205903064.png)

第4步会取出数据执行je指令，因为ZF为0，不选择分支。计数器产生新值0x01f

### 4.3.4 SEQ Stage Implementations

不同指令代码、功能码、寄存器名字、ALU操作和状态码值

![image-20220108210543633](Chapter 4：Processor Architecture.assets/image-20220108210543633.png)

#### Fetch Stage

![image-20220108211130351](Chapter 4：Processor Architecture.assets/image-20220108211130351.png)

imem error：地址不合法

Instr vaild：发现不合法指令

Need regids：包括一个寄存器指示符字吗？（1~8/2~9产生valC）

Need valC：包括一个常数字吗

imem error、imem error访存阶段用来产生状态码

PC increment：根据need regids跟need_valC值产生valP(p+1+r+8i)

#### Decode and Write-Back Stages

![image-20220108212717799](Chapter 4：Processor Architecture.assets/image-20220108212717799.png)

寄存器文件：A、B读端口  M、E（E是栈，M是寄存器）写端口（每个端口都有地址连接（寄存器ID，为0xF表明不需要访问）和数据连接）

写端口地址输入  dst

读端口地址输入 src

![image-20220108213202903](Chapter 4：Processor Architecture.assets/image-20220108213202903.png)

RRSP 是%rsp栈寄存器

![image-20220108213350018](Chapter 4：Processor Architecture.assets/image-20220108213350018.png)

#### Execute Stage

![image-20220108213638441](Chapter 4：Processor Architecture.assets/image-20220108213638441.png)

![image-20220108213659127](Chapter 4：Processor Architecture.assets/image-20220108213659127.png)

![image-20220108213936884](Chapter 4：Processor Architecture.assets/image-20220108213936884.png)

cond根据条件码和功能码确定是否进行条件分支或条件数据传送

### Memory Stage

![image-20220108214300090](Chapter 4：Processor Architecture.assets/image-20220108214300090.png)

![image-20220108214313987](Chapter 4：Processor Architecture.assets/image-20220108214313987.png)

##### 练习题4.27

![image-20220108214806789](Chapter 4：Processor Architecture.assets/image-20220108214806789.png)

![image-20220108214835929](Chapter 4：Processor Architecture.assets/image-20220108214835929.png)

#### PC Update Stage

![image-20220108214923766](Chapter 4：Processor Architecture.assets/image-20220108214923766.png)

根据指令代码和分支标志，从valC、valM、valP选

![image-20220108215009617](Chapter 4：Processor Architecture.assets/image-20220108215009617.png)

## 4.4General Principles of Pipelining

吞吐量：单位时间执行总量

延迟：服务一个所需时间

### 4.4.1 Computational Pipelines

![image-20220108221347155](Chapter 4：Processor Architecture.assets/image-20220108221347155.png)

![image-20220114175941905](Chapter 4：Processor Architecture.assets/image-20220114175941905.png)

延迟：从头到尾执行一条指令所需时间

![image-20220114180148592](Chapter 4：Processor Architecture.assets/image-20220114180148592.png)

### 4.4.3 Limitations of Pipelining

#### Nonuniform Partitioning(不一致的划分)

![image-20220114181055987](Chapter 4：Processor Architecture.assets/image-20220114181055987.png)

由于时长不一致，由最长的B决定，时钟周期必须设为150+20=170ps

#### Dimishing Returns of Deep Pipelining（流水线过深，收益反而下降）

![image-20220114182144995](Chapter 4：Processor Architecture.assets/image-20220114182144995.png)

由于流水线寄存器延迟，吞吐量没有加倍，成了制约因素。

### 4.4.4 Pipelining a System with Feedback

![image-20220114182736180](Chapter 4：Processor Architecture.assets/image-20220114182736180.png)

增加了阶段，则本来l1的输出作为l2的输入，三阶段变成l1输出作为l4输入。

## 4.5 Pipelined Y86-64 Implementations

### 4.5.1 SEQ+： Rearanging the Computation Stages（重新安排计算阶段）

将PC阶段计算从时钟结束移动到时钟开始时

![image-20220114191345730](Chapter 4：Processor Architecture.assets/image-20220114191345730.png)

### 4.5.2 Inserting Pipeline Registers



**PIPE-**



![image-20220114191703165](Chapter 4：Processor Architecture.assets/image-20220114191703165.png)

![image-20220114191849820](Chapter 4：Processor Architecture.assets/image-20220114191849820.png)

![image-20220114191859525](Chapter 4：Processor Architecture.assets/image-20220114191859525.png)

![image-20220114191953898](Chapter 4：Processor Architecture.assets/image-20220114191953898.png)

大写前缀是流水线寄存器，小写前缀是流水线阶段

### 4.5.4 Next PC Prediction

总是预测选择了条件分支

从**不选择分支valP 、选择分支valC、ret到达流水线寄存器W(W_valM)**选择最终PC值

### 4.5.5 Pipeline Hazards

分为数据和控制两部分

![image-20220114200425663](Chapter 4：Processor Architecture.assets/image-20220114200425663.png)

周期6完成了对常数3的写入，周期7从寄存器中从寄存器中取出值，所以并没有造成数据危害

如果去掉一个nop，则还没有写回寄存器，就被取出，则发生错误。

去掉两个nop，则得到的两个操作数都是错误的。

#### Avoid Data Hazards by Stalling（暂停）

![image-20220114201115460](Chapter 4：Processor Architecture.assets/image-20220114201115460.png)

用bubble，就像nop，不会改变寄存器、内存、条件码或程序状态。用标号"D"和"E"之间的箭头表示。

性能并不是很好

#### Avoiding Data Hazards by Forwarding（转发）

![image-20220114201536930](Chapter 4：Processor Architecture.assets/image-20220114201536930.png)

发现未写入寄存器文件的值，直接使用该值，而不是去取寄存器的值。

![image-20220114203528307](Chapter 4：Processor Architecture.assets/image-20220114203528307.png)

同样，两个操作数也可以用数据转发实现。

![image-20220114203719787](Chapter 4：Processor Architecture.assets/image-20220114203719787.png)

除了将ALU产生的以及其目标为写端口E的值进行转发，还可以从内存中读出的以及其目标为写端口M的值，如上图，一共有五个不同的转发源

#### Load/Use Data Hazards

![image-20220114204240643](Chapter 4：Processor Architecture.assets/image-20220114204240643.png)

当加载的数据在下一个指令就要被使用，可以同时使用暂停和转发技术，先用bubble将加载指令推迟到访存阶段，而使用指令到译码阶段。然后通过转发完成计算。这种暂停成为 load interlock

#### Avoiding Control Hazards

无法根据取指阶段确定下一条指令地址。发生在ret和跳转指令(预测错误)

![image-20220114204743548](Chapter 4：Processor Architecture.assets/image-20220114204743548.png)

ret指令经过译码执行和访存阶段，会用bubble进行停止，直到写回阶段，选择返回地址作为指令的取指地址。

![image-20220114211321413](Chapter 4：Processor Architecture.assets/image-20220114211321413.png)

当发现分支预测错误，往第一个取值的执行和第二个取值的译码阶段加入bubble，实现指令取消，再继续执行下条指令。

### 4.5.6 Exception Handling

1. halt指令
2. 有非法指令和功能码组合的指令
3. 取指或数据读写试图访问一个非法地址

基本原则:

1. 由流水线中最深的指令引起的异常,优先级最高。
2. 分支预测错误的异常指令不会导致异常停止。
3. 异常指令之后的所有指令不能影响系统状态。（禁止更新条件码寄存器或是数据内存）

### 4.5.7 PIPE Stage Implementations

![image-20220114212431670](Chapter 4：Processor Architecture.assets/image-20220114212431670.png)

HCL代码区别在于加上了前缀，表明信号源来自寄存器D(译码阶段产生)

#### PC Selection and Fetch Stage

![image-20220114212603846](Chapter 4：Processor Architecture.assets/image-20220114212603846.png)

PC从三个程序计数器源选择(错误预测分支、ret、其他情况)

![image-20220114212744830](Chapter 4：Processor Architecture.assets/image-20220114212744830.png)

函数调用或跳转，选择valC

![image-20220114212828813](Chapter 4：Processor Architecture.assets/image-20220114212828813.png)

#### Decode and Write-Back stages

![image-20220114213014246](Chapter 4：Processor Architecture.assets/image-20220114213014246.png)

寄存器ID来自写回阶段，而不是译码阶段。

![image-20220114213333529](Chapter 4：Processor Architecture.assets/image-20220114213333529.png)

五个转发源

不满足转发条件，则选择d_rvalA作为输出(寄存器端口A读出的值)

![image-20220114213438121](Chapter 4：Processor Architecture.assets/image-20220114213438121.png)

优先级由上述检测寄存器ID顺序决定。最早流水线转发源以较高优先级（也就是最近的指令）

![image-20220114214447988](Chapter 4：Processor Architecture.assets/image-20220114214447988.png)

上图所示，第一条指令进入到访存阶段，第二条指令进入到执行阶段，从最早流水线原则，应该选择执行阶段的值作为转发数据的值

#### Execute Stage

![image-20220114214946519](Chapter 4：Processor Architecture.assets/image-20220114214946519.png)

#### Memory Stage

![image-20220114215109563](Chapter 4：Processor Architecture.assets/image-20220114215109563.png)

#### 4.5.8 Pipeline Control Logic

Load/use hazards：读和使用值之间，必须暂停一个周期

Processing ret：暂停直到ret到达写回阶段

Mispredicted branches：错误预测必须取消指令，并从跳转后面那条指令开始取指

Exceptions：进制后面指令更新程序员可见的状态，异常到达写回时，停止执行。

#### Desired Handling of Special Control Cases（特殊控制情况所期望的处理）

![image-20220115153728593](Chapter 4：Processor Architecture.assets/image-20220115153728593.png)

在ret的时候，连续取出了三个不正确的指令，直到ret进入到写回阶段，找到正确的pc地址

当异常发生，防止修改程序员可见状态

1. 禁止执行阶段指令设置条件码
2. 向内存阶段中插入气泡，以禁止向数据内存中写入
3. 当写回阶段有异常指令时，暂停写回阶段，因而暂停流水线

![image-20220115154235448](Chapter 4：Processor Architecture.assets/image-20220115154235448.png)

出现内存异常，禁止更新条件码，往后续访存阶段插入bubble，并在写回阶段暂停异常指令

#### Detecting Special Control Conditions

![image-20220115154642951](Chapter 4：Processor Architecture.assets/image-20220115154642951.png)

#### Pipeline Control Mechanisms

![image-20220115154900014](Chapter 4：Processor Architecture.assets/image-20220115154900014.png)

stall为1，禁止更新State

bubble为1，设置为固定的复位配置，等效于nop指令

都为1，看成出错

![image-20220115155157106](Chapter 4：Processor Architecture.assets/image-20220115155157106.png)

各个流水线寄存器在三种特殊情况下应该采取的行动

#### Combinations of Control Conditions

![image-20220115155536094](Chapter 4：Processor Architecture.assets/image-20220115155536094.png)

组合可能同时出现。

组合A的ret出现在不选择分支

![image-20220115155754775](Chapter 4：Processor Architecture.assets/image-20220115155754775.png)

组合A的处理与预测错误分支类似，只是取指阶段是暂停

组合B加载设置寄存器%rsp，ret用这个寄存器作为源操作数，弹出地址，应该将ret指令阻塞在译码阶段。

![image-20220115160028835](Chapter 4：Processor Architecture.assets/image-20220115160028835.png)

既有stall又有bubble，希望ret推迟一个周期，用stall

#### Control Logic Implementation

![image-20220115160345117](Chapter 4：Processor Architecture.assets/image-20220115160345117.png)

![image-20220115160355236](Chapter 4：Processor Architecture.assets/image-20220115160355236.png)

![image-20220115160437029](Chapter 4：Processor Architecture.assets/image-20220115160437029.png)

### 4.5.9 Performance Analysis

![image-20220115162236073](Chapter 4：Processor Architecture.assets/image-20220115162236073.png)

一共处理Ci条指令和Cb个气泡

![image-20220115162341750](Chapter 4：Processor Architecture.assets/image-20220115162341750.png)

lp(load penalty)

mp(mispredicted branch penalty)

rp(ret penalty)

![image-20220115162505548](Chapter 4：Processor Architecture.assets/image-20220115162505548.png)

### 4.5.10 unfinished Business

#### Multicycle Instrutions

如整数乘法、除法、浮点运算等

#### Interfacing with the Memory System

用暂停处理短时间的高速缓存不命中和用异常处理长时间的缺页，能顾及所有存储器引起的不可预测性。

## 4.6 Summary

ISA：顺序指令(RISC、CISC)，

SEQ处理器：每个时钟周期执行一条指令，通过五个阶段

SEQ+：引入流水线

PIPE-：添加流水线寄存器
