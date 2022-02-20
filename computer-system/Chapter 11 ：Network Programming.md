# Chapter 11 ：Network Programming

## 11.1 The Client-Server Programming Model

![image-20220220142801567](Chapter 11 ：Network Programming.assets/image-20220220142801567.png)

**客户端和服务器是进程**，而不是机器或主机

## 11.2 Networks

![image-20220220143011297](Chapter 11 ：Network Programming.assets/image-20220220143011297.png)

DMA传送，不通过处理器，直接内存到网络之间传送

![image-20220220143141531](Chapter 11 ：Network Programming.assets/image-20220220143141531.png)

集线器

每个主机能看到传送的帧，只有目的主机实际读取它

![image-20220220143314227](Chapter 11 ：Network Programming.assets/image-20220220143314227.png)

网桥将多个以太网连接较大局域网，**桥接以太网**

网桥之间利用聪明的分配算法，带宽更高

A->B 到达网桥X会丢弃

A->C X只会复制到Y，Y只复制到主机C

**路由器：**连接多个不兼容局域网，广域网



![image-20220220143745982](Chapter 11 ：Network Programming.assets/image-20220220143745982.png)

三台路由器连接两个局域网和两个广域网。

协议消除网络之间差异。

![image-20220220144021983](Chapter 11 ：Network Programming.assets/image-20220220144021983.png)

![image-20220220144145587](Chapter 11 ：Network Programming.assets/image-20220220144145587.png)

![image-20220220144630152](Chapter 11 ：Network Programming.assets/image-20220220144630152.png)

![image-20220220144334094](Chapter 11 ：Network Programming.assets/image-20220220144334094.png)

## 11.3 The Global IP Internet

![image-20220220144419483](Chapter 11 ：Network Programming.assets/image-20220220144419483.png)

TCP/IP协议(Transmission Control Protocol/Internet Protocol)：协议族，每个提供不同功能

混合使用套接字接口和Unix I/O接口，套接字接口实现为系统调用

因特网看做世界范围的主机集合

![image-20220220145201266](Chapter 11 ：Network Programming.assets/image-20220220145201266.png)

### 11.3.1 IP Addresses

![image-20220220151842788](Chapter 11 ：Network Programming.assets/image-20220220151842788.png)

通过上面的函数进行网络和主机字节顺序转换

hotnl：主->网络

ntohl：网络->主

`hostname -i` 确定主机地址

![image-20220220152251896](Chapter 11 ：Network Programming.assets/image-20220220152251896.png)

IP地址和点分十进制串转换

### 11.3.2 Internet Domain Names

![image-20220220152553289](Chapter 11 ：Network Programming.assets/image-20220220152553289.png)

定义了域名集合和IP地址集合之间映射，通过分布世界范围的数据库维护(DNS域名系统)，每个主机条目就是一个域名和IP地址的等价类。

多个域名可以映射同一个ip地址

多个域名可以映射同一组的多个ip地址。

### 11.3.3 Internet Connections

套接字：地址：端口表示

客户端：内核自动分配临时端口

服务器端：知名端口

![image-20220220152926961](Chapter 11 ：Network Programming.assets/image-20220220152926961.png)

## 11.4 The Sockets Interface

![image-20220220153351849](Chapter 11 ：Network Programming.assets/image-20220220153351849.png)

### 11.4.1 Socket Address Structures

套接字是通信的一个端点

有相应描述符的打开文件

![image-20220220153710005](Chapter 11 ：Network Programming.assets/image-20220220153710005.png)

总是以网络字节顺序(大端法)存放

connect、bind和accept函数要求指向与协议相关的套接字地址结构的指针。

### 14.4.2 The socket Function

创建套接字描述符

![image-20220220160112963](Chapter 11 ：Network Programming.assets/image-20220220160112963.png)

![image-20220220160210030](Chapter 11 ：Network Programming.assets/image-20220220160210030.png)

### 11.4.3 The connect Function

建立服务器连接

![image-20220220160253621](Chapter 11 ：Network Programming.assets/image-20220220160253621.png)

会阻塞直到连接成功或发生错误。

![image-20220220160431554](Chapter 11 ：Network Programming.assets/image-20220220160431554.png)

### 11.4.4 The bind Function

服务器和客户端建立连接

![image-20220220160513784](Chapter 11 ：Network Programming.assets/image-20220220160513784.png)

将addr中的服务器套接字地址和描述符sockfd联系起来

### 11.4.5 The listen Function

描述符被服务器使用而不是客户端

![image-20220220160635519](Chapter 11 ：Network Programming.assets/image-20220220160635519.png)

将sockfd从一个主动套接字转化为一个监听套接字，接受客户端连接请求。

backlog：开始拒绝请求前，未完成连接数量

### 11.4.6 The accept Function

等待来自客户端连接请求

![image-20220220160854983](Chapter 11 ：Network Programming.assets/image-20220220160854983.png)

![image-20220220161241598](Chapter 11 ：Network Programming.assets/image-20220220161241598.png)

![image-20220220161058904](Chapter 11 ：Network Programming.assets/image-20220220161058904.png)

1. 阻塞accept，等待连接到达监听描述符
2. 客户端调用connect
3. accept打开新的已连接描述符

### 11.4.7 Host and Service Conversion

#### The Getaddrinfo Function

将主机名、主机地址、服务名和端口号字符串转套接字地址结构

![image-20220220161910943](Chapter 11 ：Network Programming.assets/image-20220220161910943.png)

返回result指向addrinfo结构链表，指向对应host和service套接字地址结构

![image-20220220162013190](Chapter 11 ：Network Programming.assets/image-20220220162013190.png)

客户端调用后遍历列表尝试每个套接字地址，直到调用成功，建立连接。

服务器遍历套接字地址直到调用socket和bind成功

防止泄露最后调用freeaddrinfo释放链表。

![image-20220220162357043](Chapter 11 ：Network Programming.assets/image-20220220162357043.png)

![image-20220220162415957](Chapter 11 ：Network Programming.assets/image-20220220162415957.png)

![image-20220220162326372](Chapter 11 ：Network Programming.assets/image-20220220162326372.png)

![image-20220220162504410](Chapter 11 ：Network Programming.assets/image-20220220162504410.png)

#### The getnameinfo Function

将套接字地址转主机和服务名字符串。

![image-20220220170603399](Chapter 11 ：Network Programming.assets/image-20220220170603399.png)

![image-20220220170713894](Chapter 11 ：Network Programming.assets/image-20220220170713894.png)

![image-20220220170724523](Chapter 11 ：Network Programming.assets/image-20220220170724523.png)

### 11.4.8 Helper Functions for the Sockets Interface

#### The open_clientfd Function

建立与服务器的连接。

![image-20220220171227310](Chapter 11 ：Network Programming.assets/image-20220220171227310.png)

运行主机hostname 端口port上监听。

返回打开的套接字描述符

![image-20220220171701585](Chapter 11 ：Network Programming.assets/image-20220220171701585.png)

#### The open_listenfd Function

创建监听描述符，准备接收连接请求

![image-20220220171817168](Chapter 11 ：Network Programming.assets/image-20220220171817168.png)

打开和返回监听描述符，在端口port接收连接请求

![image-20220220171921590](Chapter 11 ：Network Programming.assets/image-20220220171921590.png)

调用getaddrinfo，遍历直到调用socket和bind成功。最后调用listen函数，将listenfd转换监听描述符返回

### 11.4.9 Example Echo Client and Server

![image-20220220172206944](Chapter 11 ：Network Programming.assets/image-20220220172206944.png)

连接后进入循环，从标准输入读取文本行，发送为服务器，从服务器读取回的到标准输出。遇EOF或Ctrl+D终止循环

循环终止，进程终止，自动关闭描述符， 24行显式关闭是良好编程习惯。

![image-20220220172556454](Chapter 11 ：Network Programming.assets/image-20220220172556454.png)

监听描述符后，循环等待客户端连接请求，输出已经连接的域名和IP地址，调用echo为客户端服务，echo返回后关闭已连接描述符，客户端服务器均关闭则连接终止。

clientaddr在accept返回前连接另一端客户端的套接字地址。

![image-20220220172839776](Chapter 11 ：Network Programming.assets/image-20220220172839776.png)

反复读写文本行，直到rio_readlineb遇到EOF

> EOF是内核检查到的条件，read函数返回零返回码就会出现，文件位置超出文件长度发生EOF，进程关闭连接一端发生EOF，另一端试图读取最后一个字节之后的字节检测到EOF

## 11.5 Web Servers

基于HTTP

HTML语言编写内容

### 11.5.2 Web Content

![image-20220220190848618](Chapter 11 ：Network Programming.assets/image-20220220190848618.png)

![image-20220220190949750](Chapter 11 ：Network Programming.assets/image-20220220190949750.png)

### 11.5.3 HTTP Transactions

#### HTTP Requests

![image-20220220191231342](Chapter 11 ：Network Programming.assets/image-20220220191231342.png)

5行：请求行

6行：请求报头

7行：空行

GET请求，直到服务器生成和返回URI标识的内容

#### HTTP Responses

8行：响应行

9~13：响应报头

14：终止空行

15-17：响应主体

![image-20220220191455862](Chapter 11 ：Network Programming.assets/image-20220220191455862.png)

### 11.5.4 Serving Dynamic Content

#### 服务器如何将参数传递给子进程

![image-20220220191949682](Chapter 11 ：Network Programming.assets/image-20220220191949682.png)

#### 服务器如何将其他信息传递给子进程

![image-20220220192021759](Chapter 11 ：Network Programming.assets/image-20220220192021759.png)

#### 子进程将它的输出发送到哪里

![image-20220220192139159](Chapter 11 ：Network Programming.assets/image-20220220192139159.png)

##### 练习题11.5

![image-20220220192249694](Chapter 11 ：Network Programming.assets/image-20220220192249694.png)

在子进程运行的CGI程序不需要显式地关闭输入输出流，当子进程终止时，内核会自动关闭所有描述符。

## 11.6  Putting It Together :The TINY Web Server

#### 主程序main

![image-20220220192538316](Chapter 11 ：Network Programming.assets/image-20220220192538316.png)

29行 open_listenfd打开一个监听套接字，无限循环

接收请求(32行)，执行事务(36)、关闭连接(37)

#### doit函数

![image-20220220193006813](Chapter 11 ：Network Programming.assets/image-20220220193006813.png)

处理HTTP事务

11-14:读解析请求行，用rio_readlineb

只支持GET请求，

23：表明静态内容还是动态内容

30：普通文件读权限，36：提供静态内容

39：动态内容，验证可执行文件，44：提供动态内容

#### clienterror函数

![image-20220220193442279](Chapter 11 ：Network Programming.assets/image-20220220193442279.png)

响应主体包含HTML文件，向浏览器用户解释错误

#### read_requesthdrs函数

![image-20220220193648717](Chapter 11 ：Network Programming.assets/image-20220220193648717.png)

读取并忽略报头信息。 通过6行检查回车和换行符

#### parse_uri函数

![image-20220220193752017](Chapter 11 ：Network Programming.assets/image-20220220193752017.png)

静态内容，清除CGI参数字符串，转Linux相对路径名

动态内容，抽取所有CGI参数，转为一个Linux相对文件名

#### serve_static函数

![image-20220220193927585](Chapter 11 ：Network Programming.assets/image-20220220193927585.png)

7-13：检查文件名后缀判断文件类型，发送响应行和响应报头给客户端。

请求文件内容复制到已连接描述符fd发送响应主体，18：只读方式打开filename，获得描述符

文件映射到内存就不需要描述符，20行关闭

21行执行文件传送，22行关闭防止泄露

#### serve_dynamic

![image-20220220194223122](Chapter 11 ：Network Programming.assets/image-20220220194223122.png)

派生子进程并运行CGI程序，提供各类型动态内容。

![image-20220220194400589](Chapter 11 ：Network Programming.assets/image-20220220194400589.png)

