---
title: Linux系统编程01-基础知识
date: 2025-01-09 14:55:59
categories:
- linux
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686ca567e2bda.png
---

### 一、套接字编程

**TCP服务端**

1️⃣ 创建套接字socket()

```c++
int sockfd = socket(PF_INET, SOCK_STREAM, 0);
// PF_INET 指定协议族为 IPv4
// SOCK_STREAM：指定套接字类型为流套接字
// 0：指定协议类型，这里使用默认协议
// 如果socket函数成功创建套接字，会返回一个非负整数作为套接字描述符；如果失败，返回-1。
```

2️⃣ 存储地址信息

```c++
// 定义一个struct sockaddr_in类型的变量addr，用于存储地址信息。
struct sockaddr_in addr;

//指定地址族为 IPv4，AF代表 “Address Family”，AF_INET和前面的PF_INET相对应。
addr.sin_family = AF_INET; 

// INADDR_ANY，表示该套接字将绑定到服务器上的所有网络接口。
addr.sin_addr.s_addr = INADDR_ANY;

// htons函数将主机字节序的端口号 9527 转换为网络字节序，并存储在sin_port字段中。
addr.sin_port = htons(9527);
```

②分配套接字地址bind()

```c++
// 调用bind函数将创建的套接字sock绑定到指定的地址和端口。
bind(sockfd, (struct sockaddr*)&addr, sizeof(addr));
// 如果bind函数失败，返回-1。
```

③等待连接请求状态listen()

```c++
listen(sockfd, 5);  // 5 是请求队列的最大长度
```

④允许连接accept()

```c++
int client_sock = accept(sockfd, (struct sockaddr*)&client_addr, &client_len);
```

⑤数据交换read()/write()

```c++
char buffer[1024];
read(client_sock, buffer, sizeof(buffer));  // 接收客户端的数据
write(client_sock, "Hello, Client!", 15);   // 发送数据到客户端
```

⑥断开连接close()    关闭服务器套接字

```c++
close(client_sock);
close(sockfd);
```

**客户端套接字编程流程**

①创建套接字socket()

```c++
int sockfd = socket(PF_INET, SOCK_STREAM, 0);
```

②设置服务器地址

```c++
struct sockaddr_in server_addr;
server_addr.sin_family = AF_INET;
server_addr.sin_port = htons(9527);  // 服务器端的端口号
server_addr.sin_addr.s_addr = inet_addr("192.168.0.1");  // 服务器的 IP 地址
```

③连接服务器（connect）

```c++
connect(sockfd, (struct sockaddr*)&server_addr, sizeof(server_addr));
```

④数据交换read()/write()

```c++
write(sockfd, "Hello, Server!", 15);  // 向服务器发送数据
read(sockfd, buffer, sizeof(buffer));  // 从服务器读取数据
```

⑤断开连接close()    关闭服务器套接字

```c++
close(sockfd);
```

**客户端套接字地址信息在哪里**？

网络数据交换必须分配IP和端口。  调用connect函数时，在操作系统内核 IP使用计算机的IP，端口随机

即 客户端IP地址和端口在调用connect函数时自动分配，无需调用标记的bind函数进行分配。

客户端在listen之后  close之前connect才有效

**迭代服务器**

普通服务器TCP的缺点：启动一次服务程序，只能给一个客户端服务。迭代服务器比较原始，它的原型可以描述成：

```c++
while(1)
{
    new_fd = 服务器 accept客户端的连接(new_fd = accept(listenfd,xx,xx));
    //逻辑处理
    // 在这个new_fd上给客户端发送消息
    // 关闭new_fd
}
```

**基本工作流程**

1. **等待连接** ：服务器会一直监听一个端口，等待客户端的连接。
2. **接受连接** ：当客户端连接时，服务器会接受连接并建立一个新的套接字与客户端通信。
3. **处理请求** ：服务器会处理客户端的请求，执行一些操作，或者提供一些服务。
4. **关闭连接** ：处理完一个客户端的请求后，服务器关闭与该客户端的连接。
5. **继续等待** ：服务器回到监听状态，等待下一个客户端的连接请求。

**write函数和read函数**

①write函数   `ssize_t write(int fd, const void *buf, size_t count);`

`write`函数用于向文件描述符`fd`所指向的文件、套接字或其他I/O设备中写入数据。



### TCP底层原理

`write`函数调用后并非立即传输数据，read函数调用后也并非马上接收数据。`write`函数调用瞬间，数据将移至输出缓冲；`read`函数调用瞬间，从输入缓冲读取数据。

A：I/O缓冲在每个套接字中单独存在。

B：I/O缓冲在创建套接字时自动生成。

C：即使关闭套接字也会**继续传递输出缓冲中遗留的数据**。

D：关闭套接字将丢失输入缓冲中的数据。



"客户端输入缓冲50字节，而服务器端传输了100字节。"

填满输入缓冲前迅速调用read函数读取数据，就能腾出一部分空间，问题就解决了。

其实不会发生这类问题，因为TCP会控制数据流。TCP有滑动窗口协议。

**数据收发也是如此，因此TCP中不会因为缓冲溢出而丢失数据。**

**但是会因为缓冲而影响传输效率。**

#### TCP内部原理

**TCP通信的三大步骤**

1. 三次握手建立连接；
2. 开始通信，进行数据交换；
3. 四次挥手断开连接；

利用TCP三次握手进行攻击。

断开连接有可能卡在中间两次发数据包的地方。



### UDP编程

#### UDP基本原理

4层TCP/IP模型中，第二层传输层分为TCP和UDP。

只考虑可靠性，TCP比UDP更好。但UDP在结构上比TCP更简洁。UDP不会发送类似ACK的应答消息，也不会想SEQ那样给数据包分配序号。因此UDP的性能有时比TCP高出很多。

为了提供可靠的数据传输服务，TCP在不可靠的IP层进行流控制，而UDP就缺少这种流控制机制。

TCP的速度无法超过UDP，但在收发某些类型的数据时有可能接近UDP。例如，每次交换的数据量越大，TCP的传输速率越接近UDP的传输速率。

**TCP比UDP慢的原因**

① 收发数据前后进行的连接设置及清除过程。

② 收发数据过程中为保证可靠性而添加的流控制。

**UDP 服务端**

UDP   服务器/客户端不像TCP那样在连接状态下交换数据，因此与TCP不同，无需经过连接过程。也就是说，不必调用TCP连接过程中调用的listen函数和accept函数。UDP中只有**创建套接字的过程和数据交换的过程**。

UDP的服务端和客户端都只需要一个套接字，而TCP中，套接字之间是一一对应的关系。若要向10个客户端提供服务，则除了守门的服务器套接字外，还需要10个服务器端套接字。

UDP套接字绑定本地的IP地址和端口号，这个套接字就可以接受来自任何主机发送到该端口的数据报。

创建好TCP套接字后，传输数据时无需再添加地址信息。但是UDP套接字不会保持连接状态（UDP套接字只有简单的邮筒功能），因此每次传输数据都要添加目标地址信息。这相当于寄信前在信件中填写地址。填写地址并传输数据时调用的UDP相关函数。

```C++
#include <sys/socket.h>
ssize_t sendto(int sock,void*buff,size_t nbytes,int flags,struct sockaddr *to, socklen_t addrlen);
/*
sock     用于传输数据的UDP套接字文件描述符。
buff     保存待传输数据的缓冲地址值。
nbytes   待传输的数据长度，以字节为单位。
flags    可选项参数，若没有则传递0。
to       存有目标地址信息的sockaddr结构体变量的地址值
addrlen  传递给参数to的地址值结构体变量长度。


上述函数与之前的TCP输出函数最大的区别在于，此函数需要向它传递目标地址信息。接下来介绍接收UDP数据的函数。UDP数据的发送端并不固定，因此该函数定义为可接收发送端信息的形式，也就是将同时返回UDP数据包中的发送端信息。
*/
```

```c++
#include<sys/socket.h>
ssize_t recvfrom(int sock,  void *buff,size_t nbytes,  int flags,
                 struct sockaddr*from,   socklen_t*addrlen);
// →成功时返回接收的字节数，失败时返回-1。
/*
sock用于接收数据的UDP套接字文件描述符。
buff保存接收数据的缓存地址值
nbytes 可接收的最大字节数，故无法超过参数buf所指的缓冲大小。
flags可选项参数，若没有则传入0。
from存有发送端地址信息的sockaddr结构体变量的地址值。
addrlen 保存参数from的结构体变量长度的变量地址值。
编写UDP程序时最核心的部分就在于上述两个函数，这也说明二者在UDP数据传输中的地位。
*/
```



UDP相关的函数

DDos攻击大多利用UDP





**发生地址分配错误**（重点）

客户端是connect，所以不会出现bind  failed，因为有几万个端口可以用，所以connect失败很难





**I/O缓冲区大小**







**TCP_NODELAY**

监控键盘的输入







### Linux系统编程：进程

#### 并发服务器的实现方法

使其同时相所有发起请求的客户端提供服务，以提高平均满意度。

而且，**网络程序中数据通信时间比CPU运算时间占比更大，因此，向多个客户端提供服务是一种有效利用CPU的方式。**

1. 多进程服务器：通过创建多个进程提供服务。
2. 多路复用服务器：通过捆绑并统一管理I/O对象提供服务。
3. 多线程服务器：通过生成与客户端等量的线程提供服务。

**进程**：占用内存空间正在运行的程序 就是进程

从操作系统的角度来看，**进程是程序流的基本单位**，若创建多个进程，则操作系统将同时运行。有时候一个程序运行过程中也会产生多个进程。