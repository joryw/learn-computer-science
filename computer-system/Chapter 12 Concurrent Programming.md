# Chapter 12 Concurrent Programming

## 12.1 Concurrent Programming with Processes

![image-20220221212001745](Untitled.assets/image-20220221212001745.png)

![image-20220221212013743](Untitled.assets/image-20220221212013743.png)

![image-20220221212031181](Untitled.assets/image-20220221212031181.png)

派生子进程进行服务，同时父进程释放已连接描述符，否则会发生内存泄露。子进程并发向各自客户端提供服务

### 12.1.1 A Concurrent Serve Based on Processes

![image-20220221213804653](Untitled.assets/image-20220221213804653.png)

![image-20220221214234755](Untitled.assets/image-20220221214234755.png)

共享文件表，但不共享用户地址空间。

![image-20220221214544587](Untitled.assets/image-20220221214544587.png)

引用计算从2减少到1，仍可以使用

终止时，内核关闭所有打开的描述符

### 12.2 Concurrent Programming with I/O Multiplexing

使用select函数，要求内核挂起进程，只有一个或多个I/O事件发生后，才将控制返回给应用程序。

![image-20220221214853986](Untitled.assets/image-20220221214853986.png)

处理类型为fd_set集合

![image-20220221215541439](Untitled.assets/image-20220221215541439.png)

![image-20220221215554740](Untitled.assets/image-20220221215554740.png)

### 12.2.1 A Concurrent Event-Driven Server Based on I/O Multiplexing

![image-20220221215924243](Untitled.assets/image-20220221215924243.png)

![image-20220221215937046](Untitled.assets/image-20220221215937046.png)

![image-20220221220002485](Untitled.assets/image-20220221220002485.png)

![image-20220221220116868](Untitled.assets/image-20220221220116868.png)

![image-20220221220133143](Untitled.assets/image-20220221220133143.png)

![image-20220221220251437](Untitled.assets/image-20220221220251437.png)

![image-20220221220306008](Untitled.assets/image-20220221220306008.png)

![image-20220221220316030](Untitled.assets/image-20220221220316030.png)

![image-20220221220411610](Untitled.assets/image-20220221220411610.png)

![image-20220221220516058](Untitled.assets/image-20220221220516058.png)

![image-20220221220623693](Untitled.assets/image-20220221220623693.png)

即可作为输入也可作为输出，所以每次调用重新初始化。

## 12.3 Concurrent Programming with Threads

![image-20220221221032968](Untitled.assets/image-20220221221032968.png)

### 12.3.2 Posix Threads

![image-20220221221234844](Untitled.assets/image-20220221221234844.png)

主线程创建对等线程，等待它终止

### 12.3.3 Creating Threads

![image-20220221221434918](Untitled.assets/image-20220221221434918.png)

![image-20220221221446106](Untitled.assets/image-20220221221446106.png)

### 12.3.4 Terminating Threads

![image-20220221221526801](Untitled.assets/image-20220221221526801.png)

### 12.3.5 Reaping Terminated Threads

![image-20220221222431025](Untitled.assets/image-20220221222431025.png)

等待指定而不是任意1

### 13.3.6 Detaching Threads 

![image-20220221222635951](Untitled.assets/image-20220221222635951.png)

### 12.3.7  Initializing Threads

![image-20220221222825243](Untitled.assets/image-20220221222825243.png)

### 12.3.8 A Concurrent Server Based on Threads

![image-20220222104338421](Untitled.assets/image-20220222104338421.png)

多个线程运行在同一个进程中，共享同一个描述符表，引用数为1，只需要关闭一次即可。

![image-20220222104952661](Untitled.assets/image-20220222104952661.png)

## 12.4 Shared Variables in Threaded Programs

### 12.4.1 Threads Memory Model

有自己独立的线程上下文：线程ID、栈、栈指针、程序计数器、条件码和通用目的寄存器值。

### 12.4.2 Mapping Variables to Memory

![image-20220222110358283](Untitled.assets/image-20220222110358283.png)

![image-20220222110700706](Untitled.assets/image-20220222110700706.png)

栈变量线程私有，全局和静态变量是共享的。

![image-20220222110959650](Untitled.assets/image-20220222110959650.png)

* prt：全局变量，能被主程序写和对等线程读
* cnt：静态变量，内存中只有一个实例，被两个对等线程读写
* i.m：主线程栈本地自动变量，对等线程不会在栈中引用它，不是共享的
* msgs.m：主线程栈自动变量，通过ptr间接引用
* myid.0和，yid.1：分别驻留对等线程0和1的栈中

这里ptr、cnt和msgs被对于一个线程引用，是共享的

## 12.5 Synchronizing Threads with Semaphores

![image-20220222111431234](Untitled.assets/image-20220222111431234.png)

中间操作了共享变量cnt，会受到多个线程影响

![image-20220222111553331](Untitled.assets/image-20220222111553331.png)

无法预测按照正确的指令顺序运行

### 12.5.1 Process Graphs

![image-20220222112246529](Untitled.assets/image-20220222112246529.png)

![image-20220222112258443](Untitled.assets/image-20220222112258443.png)

![image-20220222112447252](Untitled.assets/image-20220222112447252.png)

穿过不安全区，是不安全的，经过轨迹或者绕开则安全。

### 12.5.2 Semaphores

![image-20220222112809655](Untitled.assets/image-20220222112809655.png)

### 12.5.3 Using Semaphores for Mutual Exclusion

![image-20220222113154491](Untitled.assets/image-20220222113154491.png)

通过PV操作的结合，创建了禁止区(-1部分)，正好包含了不安全区，所以没有一条轨迹能够进入不安全区，每次运行均能得到正确结果

### 12.5.4 Using Semaphores to Schedule Shared Resources

#### 1.生产者-消费者问题

![image-20220222113434428](Untitled.assets/image-20220222113434428.png)

#### 2.读者-写者问题

一组并发线程访问一个共享对象

写者独占，读者可无限个

1. 读者优先：读者不断进来可能导致饥饿问题
2. 写者优先

![image-20220222114100058](Untitled.assets/image-20220222114100058.png)

mutex用于读者计数，w用于控制写者是否能进行操作。

### 12.5.5 Putting It Together : A Concureent Server Based on Prethreading

![image-20220222114449574](Untitled.assets/image-20220222114449574.png)

从缓冲区中取出已连接描述符，让工作者线程进行操作。

## 12.7 Other Concurrency Issues

### 12.7.1 Thread Safety

![image-20220222115818917](Untitled.assets/image-20220222115818917.png)

![image-20220222115846669](Untitled.assets/image-20220222115846669.png)

![image-20220222115858186](Untitled.assets/image-20220222115858186.png)

### 12.7.2 Reentrancy

当被多个线程调用，不会引用任何共享数据。

![image-20220222120018167](Untitled.assets/image-20220222120018167.png)

### 12.7.3 Using Existing Library Functions in Threaded Programs

![image-20220222120259137](Untitled.assets/image-20220222120259137.png)

### 12.7.4 Races

![image-20220222120516016](Untitled.assets/image-20220222120516016.png)

在创建线程时引入参数，参数i在进行i++执行前后导致局部变量myid复制的线程id可能得到不正确的线程ID，

![image-20220222120657562](Untitled.assets/image-20220222120657562.png)

为每个整数ID分配独立的块，传递给线程指向这个块的指针

![image-20220222120903130](Untitled.assets/image-20220222120903130.png)

![image-20220222120916127](Untitled.assets/image-20220222120916127.png)

![image-20220222120924819](Untitled.assets/image-20220222120924819.png)

### 12.7.5 Deadlocks

![image-20220222120952159](Untitled.assets/image-20220222120952159.png)

![image-20220222121134149](Untitled.assets/image-20220222121134149.png)

![image-20220222121204663](Untitled.assets/image-20220222121204663.png)

![image-20220222121233316](Untitled.assets/image-20220222121233316.png)

每个线程先对s枷锁，然后对t加锁

练习题12.15

![image-20220222143451340](Untitled.assets/image-20220222143451340.png)

![image-20220222143509821](Untitled.assets/image-20220222143509821.png)