# Chapter 8：Exceptional Control Flow

## 8.1Exceptions

异常：控制流突变，用来响应处理器状态中某些变化。

![image-20220217151822009](Chapter 8：Exceptional Control Flow.assets/image-20220217151822009.png)

通过异常表的条转变，间接过程调用，到专门处理这类时间的操作系统子程序(异常处理程序)

![image-20220217152006548](Chapter 8：Exceptional Control Flow.assets/image-20220217152006548.png)

### 8.1.1 Exception Handling

![image-20220217152216062](Chapter 8：Exceptional Control Flow.assets/image-20220217152216062.png)

会有一张带有非负整数异常号的异常表，

异常表的起始地址放在异常表基址寄存器的特殊CPU寄存器中。

![image-20220217152616261](Chapter 8：Exceptional Control Flow.assets/image-20220217152616261.png)

### 8.1.2 Classes of Exceptions

![image-20220217152814710](Chapter 8：Exceptional Control Flow.assets/image-20220217152814710.png)

#### Interrupts

异步，处理器外部的I/O设备的信号

![image-20220217153049973](Chapter 8：Exceptional Control Flow.assets/image-20220217153049973.png)

其他同步发生，执行当前指令结果，称为故障指令

#### Traps and System Calls

用户程序和内核提供过程接口，称为**系统调用**

执行syscall到异常处理程序的陷阱，解析参数。

![image-20220217153403208](Chapter 8：Exceptional Control Flow.assets/image-20220217153403208.png)

#### Faults

可能修正，否则终止。比如缺页异常

![image-20220217155037771](Chapter 8：Exceptional Control Flow.assets/image-20220217155037771.png)

#### Aborts

不可恢复的致命错误造成，通常一些硬件错误

![image-20220217155406503](Chapter 8：Exceptional Control Flow.assets/image-20220217155406503.png)

### 8.1.3 Exceptions in Linux/x86-64 Systems

#### Linux/x86-64 Faults and Aborts

![image-20220217155456311](Chapter 8：Exceptional Control Flow.assets/image-20220217155456311.png)

* 一般保护错误：程序引用了未定义的虚拟内存区域，或者因为程序试图写一个只读的文本段。不会尝试修复，报告为“段故障”
* 机器检查：导致故障的指令在执行检测到致命的硬件错误。

#### Linux/x86-64 System Calls

![image-20220217155834081](Chapter 8：Exceptional Control Flow.assets/image-20220217155834081.png)

系统调用的参数通过通用寄存器而不是栈传递，

%rax包含返回值，-4095到-1的负数返回表明发生错误。

![image-20220217160331544](Chapter 8：Exceptional Control Flow.assets/image-20220217160331544.png)

采用了系统级函数write而不是函数printf

## 8.2 Processes

进程：一个执行中程序的实例。

程序运行在某个进程的**上下文**，由正确运行所需状态组成(代码、数据、栈……)

用shell执行可执行目标文件，就会创建一个新的进程。

![image-20220217160657158](Chapter 8：Exceptional Control Flow.assets/image-20220217160657158.png)

### 8.2.1 Logical Control Flow

![image-20220217161040348](Chapter 8：Exceptional Control Flow.assets/image-20220217161040348.png)

逻辑流交错执行，被强占(暂时挂起)

### 8.2.2 Concurret Flows

上图中，A、B并发，A、C并发，但B和C没有并发。

### 8.2.3 Private Address Space

这个空间中某个地址相关联的内存字节不能被其他进程读写

![image-20220217161804328](Chapter 8：Exceptional Control Flow.assets/image-20220217161804328.png)

### 8.2.4 User and Kernel Modes

**内核模式**的进程可以执行指令集中任何指令。

没有设置**模式位**，则运行在**用户模式**

用户->内核：中断、故障或者陷入系统调用。

/proc文件系统：允许用户模式进程访问内核数据结构内容。

### 8.2.5 Context Switches

内核为每个进程维持一个**上下文(context)**。

上下文切换：

1. 保存当前进程上下文
2. 恢复某个先前被抢占的进程被保存的上下文
3. 将控制传递给新恢复的进程

执行系统调用可能发生上下文切换。

中断也可能引发上下文切换。

![image-20220217162744159](Chapter 8：Exceptional Control Flow.assets/image-20220217162744159.png)

进程A需要从磁盘取数据，相对较长的时间，所以切换到进程B执行，接收到磁盘中断信号，则切换回A进程继续执行。

## 8.3 System Call Error Handling

错误处理包装函数

![image-20220217163414080](Chapter 8：Exceptional Control Flow.assets/image-20220217163414080.png)

![image-20220217163431656](Chapter 8：Exceptional Control Flow.assets/image-20220217163431656.png)

![image-20220217163440396](Chapter 8：Exceptional Control Flow.assets/image-20220217163440396.png)

## 8.4 Process Control

### 8.4.1 Obtaining Process IDs

![image-20220217164254350](Chapter 8：Exceptional Control Flow.assets/image-20220217164254350.png)

### 8.4.2 Creating and Terminating Processes

三种终止原因

1. 收到终止进程信号
2. 从主程序返回
3. 调用exit函数

![image-20220217164348058](Chapter 8：Exceptional Control Flow.assets/image-20220217164348058.png)

exit函数以**status退出状态**终止

![image-20220217164548427](Chapter 8：Exceptional Control Flow.assets/image-20220217164548427.png)

父进程通过调用**fork函数**创建新运行的子进程

* 子进程得到父进程用户级虚拟地址空间相同(但独立)的副本，包含代码和数据段、堆、共享库及用户栈。

* 子进程获得与父进程任何打开文件描述符相同的副本，子进程可以读写父进程中打开的任何文件。

* 最大区别在于不同的PID

fork调用一次，返回两次

* 一次在调用进程(父进程)中，返回子进程PID
* 一次是在新创建的子进程中,返回0

#### 案例

![image-20220217165339688](Chapter 8：Exceptional Control Flow.assets/image-20220217165339688.png)

![image-20220217165317597](Chapter 8：Exceptional Control Flow.assets/image-20220217165317597.png)

![image-20220217165307215](Chapter 8：Exceptional Control Flow.assets/image-20220217165307215.png)

进程图：拓扑排序

![image-20220217165437352](Chapter 8：Exceptional Control Flow.assets/image-20220217165437352.png)

两次调用fork

![image-20220217165705213](Chapter 8：Exceptional Control Flow.assets/image-20220217165705213.png)

### 8.4.3 Reaping Child Processes(回收子进程)

僵死(zombie)进程：终止但未被回收的进程，直到被父进程回收(reaped)

* 父进程终止：内核安排init进程(PID为1)成为它孤儿进程的养父

![image-20220217170520086](Chapter 8：Exceptional Control Flow.assets/image-20220217170520086.png)

等待集合中的一个子进程终止，返回已终止子进程的PID，此时子进程已经被回收，内核删除所有痕迹。

#### Determining the Members of the Wait Set

![image-20220217170636872](Chapter 8：Exceptional Control Flow.assets/image-20220217170636872.png)

#### Modifying the Default Behavior

![image-20220217170911298](Chapter 8：Exceptional Control Flow.assets/image-20220217170911298.png)

#### Checking the Exit Status of a Reaped Child

![image-20220217171103399](Chapter 8：Exceptional Control Flow.assets/image-20220217171103399.png)

#### Error Conditions

If the calling process has no children, then waitpid returns −1 and sets errno to ECHILD. If the waitpid function was interrupted by a signal, then it returns −1 and sets errno to EINTR

##### 练习题8.3

![image-20220217173627126](Chapter 8：Exceptional Control Flow.assets/image-20220217173627126.png)

![image-20220217173648795](Chapter 8：Exceptional Control Flow.assets/image-20220217173648795.png)

waitpid需要等待所有子进程其中一个出现终止(exit)才会继续执行

#### The wait Function

![image-20220217173827100](Chapter 8：Exceptional Control Flow.assets/image-20220217173827100.png)

等价于调用waitpid(-1, &status, 0)

#### Examples of Using waitpid

![image-20220217174647342](Chapter 8：Exceptional Control Flow.assets/image-20220217174647342.png)

![image-20220217174659333](Chapter 8：Exceptional Control Flow.assets/image-20220217174659333.png)

10-12行，创建N个子进程，并且仅对子进程执行exit

15-20行，父进程进入挂起，等待子进程终止，每当有一个子进程终止，得到进程号，并且检查是否是正常终止。

24行：当所有子进程都退出，最后执行waitpid，得到errno=ECHILD，则正常终止，否则通过unix_error输出错误信息

子进程回收顺序是特定计算机系统的属性

![image-20220217191528382](Chapter 8：Exceptional Control Flow.assets/image-20220217191528382.png)

消除不确定性，用特定顺序回收

### 8.4.4 Putting Processes to Sleep

![image-20220217192549345](Chapter 8：Exceptional Control Flow.assets/image-20220217192549345.png)

挂起指定时间，到了返回0，否则返回剩下秒数。

![image-20220217192641814](Chapter 8：Exceptional Control Flow.assets/image-20220217192641814.png)

调用函数休眠，直到进程收到信号

##### 练习题8.5

![image-20220217193043420](Chapter 8：Exceptional Control Flow.assets/image-20220217193043420.png)

![image-20220217193103150](Chapter 8：Exceptional Control Flow.assets/image-20220217193103150.png)

### 8.4.5 Loading and Running Programs

![image-20220217193149948](Chapter 8：Exceptional Control Flow.assets/image-20220217193149948.png)

当前进程的上下文中加载并运行一个新程序。

只当出现错误，如找不到filename，才返回，否则调用一次不返回。

![image-20220217193352811](Chapter 8：Exceptional Control Flow.assets/image-20220217193352811.png)

参数列表和环境变量

![image-20220217193446121](Chapter 8：Exceptional Control Flow.assets/image-20220217193446121.png)

![image-20220217193629747](Chapter 8：Exceptional Control Flow.assets/image-20220217193629747.png)

There are three arguments to function main, each stored in a register according to the x86-64 stack discipline:

(1) argc, which gives the **number** of non-null pointers in the argv[] array;

(2) argv,which points to the **first** entry in the argv[] array; 

(3) envp, which points to the **first** entry in the envp[] array.

![image-20220217194001497](Chapter 8：Exceptional Control Flow.assets/image-20220217194001497.png)

查找环境遍历，存在返回指针，不存在返回NULL

![image-20220217194056854](Chapter 8：Exceptional Control Flow.assets/image-20220217194056854.png)

unsetenv：删除字符串

setenv：用newvalue替换oldvalue，不存在则把“name=newvalue”添加

> fork：新的子进程运行相同程序，execve：当前上下文加载新的程序，覆盖地址空间，相同PID。

##### 练习题8.6

![image-20220217200023940](Chapter 8：Exceptional Control Flow.assets/image-20220217200023940.png)

![image-20220217200045104](Chapter 8：Exceptional Control Flow.assets/image-20220217200045104.png)

## 8.5 Signals

信号：允许进程和内核中断其他进程

![image-20220217200842462](Chapter 8：Exceptional Control Flow.assets/image-20220217200842462.png)

### 8.5.1 Signal Terminology(信号术语)

发送信号：1.内核检测到系统事件，2.进程调用了kill函数

接收信号：强迫以某种方式对信号做出反应

![image-20220217201144010](Chapter 8：Exceptional Control Flow.assets/image-20220217201144010.png)

信号处理捕获信号的基本思想

**一种类型至多只有一个待处理信号**

**信号阻塞，待处理信号不会被接收**

### 8.5.2 Sending Signals

#### Process Groups

每个进程属于一个进程组。

![image-20220217201522475](Chapter 8：Exceptional Control Flow.assets/image-20220217201522475.png)

返回当前进程组ID

![image-20220217201512965](Chapter 8：Exceptional Control Flow.assets/image-20220217201512965.png)

一个子进程和父进程在一个进程组，通过setpgid改变进程组

把pid进程组改为pgid，

如果pid为0.则使用进程的PID

如果pgid为0，则pid指定的进程的PID作为进程组ID

eg：15213位调用进程

`setpgid(0, 0)`

创建新的进程组，进程组ID为15213，把进程15213加入到进程组

#### Sending Signals with the /bin/kill Program

`bin/kill -9 15213`

发送信号9给进程15213

`bin/kill -9 -15213`

发送信号到进程组PID中每个进程

#### Sending Signals from the keyboard

`ls | sort` 

创建两个进程组成的前台作业。通过Unix管道连接。为每个作业创建独立的进程组。

![image-20220217202255724](Chapter 8：Exceptional Control Flow.assets/image-20220217202255724.png)

前台作业的父进程创建两个子进程都是进程组20的成员

Ctrl+C发送SIGINT信号到前台进程组每个进程，终止前台作业。

Ctrl+Z 挂起前台作业

#### Sending Signals with the kill Function

![image-20220217202501213](Chapter 8：Exceptional Control Flow.assets/image-20220217202501213.png)

![image-20220217203332226](Chapter 8：Exceptional Control Flow.assets/image-20220217203332226.png)

#### Sending Signals with the alarm Function

![image-20220217203431740](Chapter 8：Exceptional Control Flow.assets/image-20220217203431740.png)

secs秒后发送SIGALRM信号给调用进程

### 8.5.3 Receiving Signals

进程p从内核切换到用户态，检查未被阻塞待处理信号的集合，

* 为空， 则控制传递到p的逻辑控制流下一条指令

* 非空，选择集合中通常最小k，强制p接收信号k，触发进程采取某种行为。再传递到下一条指令。通常是下面的一种

![image-20220217204304121](Chapter 8：Exceptional Control Flow.assets/image-20220217204304121.png)

可以参考图8-26

![image-20220218134425545](Chapter 8：Exceptional Control Flow.assets/image-20220218134425545.png)

signal函数可以修改和信号相关联的默认信号(SIGSTOP和SIGKILL不能修改)

![image-20220217204603674](Chapter 8：Exceptional Control Flow.assets/image-20220217204603674.png)

下面案例执行Ctrl+C时发送SIGINT信号修改为捕获信号，输出一条消息，然后终止该进程

![image-20220218135027537](Chapter 8：Exceptional Control Flow.assets/image-20220218135027537.png)

![image-20220218135039029](Chapter 8：Exceptional Control Flow.assets/image-20220218135039029.png)

信号处理程序可以被其他信号处理程序中断

##### 练习题8.7

![image-20220218135249244](Chapter 8：Exceptional Control Flow.assets/image-20220218135249244.png)

![image-20220218135647449](Chapter 8：Exceptional Control Flow.assets/image-20220218135647449.png)

![image-20220218135701794](Chapter 8：Exceptional Control Flow.assets/image-20220218135701794.png)

首先判断是否输入参数，作为sleep函数的参数。

其次中断信号捕获到SIGINT，hander执行return，（为了后续能执行snooze函数）。

然后再执行snooze函数。

最后执行exit终止。

### 8.5.4 Blocking and Unblocking Signals

![image-20220218140217060](Chapter 8：Exceptional Control Flow.assets/image-20220218140217060.png)

![image-20220218140606689](Chapter 8：Exceptional Control Flow.assets/image-20220218140606689.png)

### 8.5.5 Writing Signal Handers

#### Safe Signal Handling

**G0：处理程序要尽可能简单**

**G1：在处理程序中只调用异步信号安全的函数(1.可重入 2.不能被信号处理程序中断)**

printf不在此列，唯一安全是使用write函数，下例开发SIO安全包

![image-20220218141445372](Chapter 8：Exceptional Control Flow.assets/image-20220218141445372.png)

![image-20220218141457246](Chapter 8：Exceptional Control Flow.assets/image-20220218141457246.png)

此时handler就可以调用SIO安全包版本的Sio_puts打印输出，不会被中断

**G2：保存和恢复errno**

**G3：阻塞所有信号，保护对共享全局数据结构的访问**

**G4：用volatile声明全局变量**

**G5：用sig_atomic_t声明标志，保证读写是原子不可中断的**

规则是保守不总是严格必须的。

#### Correct Signal Handling

存在一个未处理的信号表明**至少**有一个信号到达了

SIGCHLD信号，只要有一个子进程终止或停止，就会发送给父进程(自由做其他事)

![image-20220218143339444](Chapter 8：Exceptional Control Flow.assets/image-20220218143339444.png)

创建了三个子进程，当捕获到SIGCHLD信号，父进程就会去执行waitpid回收子进程，

![image-20220218143350339](Chapter 8：Exceptional Control Flow.assets/image-20220218143350339.png)

没有解决排队等待，实际执行只回收了两个子进程，是由于有待处理的信号时，第三个信号就已经到达了，从而被丢弃。而后也不会再次执行，

不可用用信号对进程中发生的事件计数

![image-20220218143927188](Chapter 8：Exceptional Control Flow.assets/image-20220218143927188.png)

这里用while循坏表示每次都尽可能多的回收子进程。

##### 练习题8.8

![image-20220218144529711](Chapter 8：Exceptional Control Flow.assets/image-20220218144529711.png)

![image-20220218144538506](Chapter 8：Exceptional Control Flow.assets/image-20220218144538506.png)

counter初始值为2，首先父进程先打印2，然后创建子进程，子进程进入循环，父进程发送终止子进程信号，并且等待子进程终止，

子进程接收到信号发生中断，执行处理程序，打印“1”，然后终止子进程

父进程回收子进程后继续，打印3（初始值2+1）

最终结果为“213”

#### Portable Signal Handling

不同系统有不同信号处理语义

* signal函数的语义各不同

* 系统调用可以被中断

![image-20220218144954210](Chapter 8：Exceptional Control Flow.assets/image-20220218144954210.png)

允许用户在设置信号处理时，明确指定要的信号处理语义

![image-20220218145259386](Chapter 8：Exceptional Control Flow.assets/image-20220218145259386.png)

利用包装函数调用sigaction，

![image-20220218145107797](Chapter 8：Exceptional Control Flow.assets/image-20220218145107797.png)

### 8.5.6 Synchronizing Flows to Avoid Nasty Concurrency Bugs

![image-20220218150240718](Chapter 8：Exceptional Control Flow.assets/image-20220218150240718.png)

上面的案例会出现，创建子进程先完成作业成为僵死进程，而父进程还没有addjob的时候就受到SIGCHLD信号，执行了deletejob，而后再执行addjob添加了不存在的子进程

这属于竞争的同步错误案例

![image-20220218150712680](Chapter 8：Exceptional Control Flow.assets/image-20220218150712680.png)

解决方案通过调用fork之前阻塞父进程中的SIGCHLD信号，再执行了addjob后再解除阻塞。

注意子进程也集成了阻塞集合，所以在调用execve之前，要解除子进程中阻塞的信号。

### 8.5.7 Explicitly Waiting for Signals(显式等待信号)

![image-20220218151328880](Chapter 8：Exceptional Control Flow.assets/image-20220218151328880.png)

暂时用mask替换当前阻塞集合，挂起该进程，直到受到一个信号，要么运行一个程序(从程序返回)，要么终止该进程(不返回直接终止)。

![image-20220218151516259](Chapter 8：Exceptional Control Flow.assets/image-20220218151516259.png)

![image-20220218151714110](Chapter 8：Exceptional Control Flow.assets/image-20220218151714110.png)

减少了循环和使用pause或sleep造成的浪费

## 8.6 Nonlocal Jumps

从一个函数转移动另一个正在执行的函数，不需要经过正常的调用。

![image-20220218152116460](Chapter 8：Exceptional Control Flow.assets/image-20220218152116460.png)

![image-20220218152343580](Chapter 8：Exceptional Control Flow.assets/image-20220218152343580.png)

在上图案例中，通过setjmp保存环境变量，调用foo()，foo()依次调用bar，当函数遇到错误可以执行longjmp从setjmp中返回，通过switch在代码中某个位置进行处理。

如果中间函数预计在结尾释放数据结构，则可能被跳过从而造成泄露。

sig版本是可以被信号处理程序使用的版本

![image-20220218152927395](Chapter 8：Exceptional Control Flow.assets/image-20220218152927395.png)

当捕获到Ctrl+C信号，会执行handler通过非本地跳转到达sigsetjmp位置，得到1的返回值，进入else进行重启。

必须在调用了sigsetjmp之后才能调用siglongjum

> 可以把try中的catch子句看成类似setjmp，throw类似longjmp

## 8.7 Tools for Manipulating Processes

![image-20220218153244578](Chapter 8：Exceptional Control Flow.assets/image-20220218153244578.png)

## 8.8 Summary

四种类型：中断（外部IO设备设置中断管脚）、故障(重启)、终止（不返回）、陷阱（系统调用）

进程：逻辑控制流、私有地址空间

创建子进程

非本地跳转规避正常调用/返回栈规则，从一个函数分支到另一个函数。

