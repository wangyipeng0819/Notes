---
title: Linux系统编程04线程与多线程
date: 2025-01-12 21:36:22
categories:
- linux
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686ca567e2bda.png
---

### 线程的概念

线程是在进程中产生的一个执行单元，是CPU调度和分配的最小单元，其在同一个进程中与其他线程并行运行，他们可以共享进程内的资源，比如内存、地址空间、打开的文件等等。

   **线程**是CPU调度和分派的基本单位

   **进程**是分配资源的基本单位

![](https://bu.dusays.com/2025/01/13/67841d43278bb.png)

进程：正在运行的程序（狭义）

是处于执行期的程序以及它所管理的资源（如打开的文件、挂起的信号、进程状态、地址空间等等）的总称，从操作系统核心角度来说，进程是操作系统调度除CPU时间片外进行的资源分配和保护的基本单位，它有一个独立的虚拟地址空间，用来容纳进程映像(如与进程关联的程序与数据)，并以进程为单位对各种资源实施保护，如受保护地访问处理器、文件、外部设备及其他进程(进程间通信) 

#### 为什么使用多线程？

1️⃣ **避免阻塞**：单个进程只有一个主线程，当主线程阻塞的时候，整个进程也就阻塞了，无法再去做其它的一些功能了。

2️⃣ **避免CPU空转**：应用程序经常会涉及到 `RPC`，数据库访问，磁盘IO等操作，这些操作的速度比CPU慢很多，而在等待这些响应时，CPU却不能去处理新的请求，导致这种单线程的应用程序性能很差。

3️⃣ **提升效率**：一个进程要独立拥有 4 GB的虚拟地址空间，而多个线程可以共享同一地址空间，线程的切换比进程的切换要快得多。

![](https://bu.dusays.com/2025/01/13/67841d4328da3.png)

补充：RPC：分为两部分，用户调用接口 + 具体网络协议。后者由框架来实现。

具体网络协议是框架来实现的，把开发者要发出和接收的内容以某种应用层协议打包进行网络收发。

·HTTP也是一种网络协议，但包的内容是固定的，必须是：请求行 + 请求头 + 请求体

·RPC是一种自定义网络协议，由具体框架来定，比如SRPC里支持的RPC协议有：SRPC/thrift/BRPC/tRPC

简单介绍到这里，后面具体学习了再谈！！！

### 线程的创建与运行

**`pthread_create`函数**

```c++
#include<pthread.h>
// 线程的创建
int pthread_create(
pthread_t*restrict thread, 
const pthread_attr_t * restrict attr,
void *(* start_routine)(void *),
void*restrict arg
);
thread
// 保存新创建线程ID的变量地址值。线程与进程相同，也需要用于区分不同线程的ID。
attr
// 用于传递线程属性的参数，传递NULL时，创建默认属性的线程。

start_routine
// 相当于线程main函数的、在单独执行流中执行的函数地址值（函数指针）。

arg
// 通过第三个参数传递调用函数时包含传递参数信息的变量地址值。

```

`pthread_join`函数
调用`pthread_join`函数的进程（或线程）将进入等待状态，直到第一个参数为ID的线程终止为止。而且可以得到线程的main函数返回值，所以该函数比较有用。下面通过示例了解该函数的功能。

```c++
# include<pthread.h>
int pthread_join(pthread_t thread, void ** status);
```

#### 示例

```c++
#include<pthread.h>
void* threadEntry(void* arg)
{
	const char* msg = "i am from thread";
	for (int i = 0; i < 5; i++)
	{
		printf("%s(%d):%s thread begin: %s\n", __FILE__, __LINE__, __FUNCTION__, arg);
	}
	
	return (void*)msg;
}

void lession94()
{
	pthread_t tid;
	const char* pInfo = "hello world"; // 全局的
	int ret = pthread_create(&tid, NULL, threadEntry,(void*)pInfo);
	if (ret != -1)
	{
		void* result = NULL;
		pthread_join(tid, &result);
		printf("%s(%d):%s from thread: %s\n", __FILE__, __LINE__, __FUNCTION__, result);
	}
}
```

不要使用局部变量

### 线程同步：互斥量

两个线程直接访问一个全局变量并进行对应的运算，结果是不确定的

**线程同步**用于解决线程访问顺序引发的问题。需要同步的情况可以从如下两方面考虑。

**1**、同时访问同一内存空间时发生的情况。

**2**、需要指定访问同一内存空间的线程执行顺序的情况。

#### 示例

```c++
int num = 0;
pthread_mutex_t mutex;
void* thread_inc(void* arg)
{
    // 实时性高，但抢占差
	for (int i = 0; i < 100000; i++)
	{
		pthread_mutex_lock(&mutex);
		num++;
		pthread_mutex_unlock(&mutex);
		if (i % 10000) printf("%s(%d):%s num is: %d\n", __FILE__, __LINE__, __FUNCTION__, num);
	}
	return NULL;
}

void* thread_dec(void* arg)
{
    // 实时性差，但抢占高
	pthread_mutex_lock(&mutex);
	for (int i = 0; i < 100000; i++)
	{
		num--;
	}
	pthread_mutex_unlock(&mutex);
	return NULL;
}
//互斥量
void lession95()
{
	pthread_mutex_init(&mutex, NULL);

	pthread_t thread_id[50];
	for (int i = 0; i < 50; i++)
	{
		if (i % 2) pthread_create(thread_id + i, NULL, thread_inc, NULL);
		else pthread_create(thread_id + i, NULL, thread_dec, NULL);
	}
	for (int i = 0; i < 50; i++)
	{
		pthread_join(thread_id[i], NULL);
	}
	printf("%s(%d):%s num is: %d\n", __FILE__, __LINE__, __FUNCTION__, num);
	pthread_mutex_destroy(&mutex);
}
```

### 线程同步：信号量

信号量与互斥量极为相似，在互斥量的基础上很容易理解信号量

初始值为 1 的信号量，可以用作互斥锁（把信号值始终控制为1）

信号量创建及销毁方法如下

```c++
#include <semaphore.h>
// 参数一  创建信号量时传递保存信号量的变量地址值，销毁时传递需要销毁的信号量变量地址值。
// 参数二  决定几个线程
// 参数三  指定初始值
int sem_init(sem_t* sem, int pshared, unsigned int value);
int sem_destroy(sem_t* sem);
```

```c++
#include<semaphore.h>
int sem_post(sem_t*sem);  // V 操作
int sem_wait(sem_t * sem);  // P操作
```

### 线程的销毁

**销毁线程的2种方法**

Linux线程并不是在首次调用的线程main函数返回时自动销毁，所以用如下2种方法之一加以明确。否则由线程创建的内存空间将一直存在。

1、调用pthread_join函数。

2、调用pthread_detach函数。

之前调用过pthread_join函数。调用该函数时，不仅会等待线程终止，还会引导线程销毁。但该函数的问题是，线程终止前，调用该函数的线程将进入阻塞状态。因此，通常通过如下函数调用引导线程销毁。

```c++
#include <pthread.h>
int pthread_detach(pthread_t thread);
```

调用 **pthread_detach**函数不会引起线程终止或进入阻塞状态，可以通过该函数引导销毁线程创建的内存空间。

`pthread_detach(pthread_self());` 

`pthread_exit(0) ;`

调用以上线程函数来进行自动销毁无需外部的操作

### 多线程并发服务器的实现

**聊天室项目：网络编程 + 多线程 + 线程同步实现的 聊天服务器和客户端**

以前进程是，每有客户端连接，fork一次，现在是开一个线程来和客户端进行沟通，主的在等待

```c++
int clnt_socks[100] = { 0 };
int clnt_cnt = 0;
pthread_mutex_t mutex1;

// 向所有已连接的客户端发送消息
void send_msg(const char* msg, ssize_t str_len)
{
	pthread_mutex_lock(&mutex1);
	for (int i = 0; i < clnt_cnt; i++) {
		if (clnt_socks[i] >= 0)
			write(clnt_socks[i], msg, str_len);
	}
	pthread_mutex_unlock(&mutex1);
}

void* handle_clnt(void* arg)
{
	pthread_detach(pthread_self());
	int clnt_sock = *(int*)arg;
	char msg[1024] = "";

	ssize_t str_len = 0;
	while ((str_len = read(clnt_sock, msg, sizeof(msg))) > 0) {
		//TODO:过滤敏感词
		send_msg(msg, str_len);
	}
	pthread_mutex_lock(&mutex1);
	//for (int i = 0; i < clnt_cnt; i++) {//TDDO:优化逻辑
	//	if (clnt_sock == clnt_socks[i]) {
	//		clnt_socks[i] = -1;
	//		break;
	//	}
	//}
	*(int*)arg = -1;
	pthread_mutex_unlock(&mutex1);
	close(clnt_sock);
	pthread_exit(NULL);
}

void server98() {
	int serv_sock, clnt_sock;
	struct sockaddr_in serv_adr, clnt_adr;
	socklen_t clnt_adr_sz = sizeof(clnt_adr);
	serv_sock = socket(PF_INET, SOCK_STREAM, 0);
	memset(&serv_adr, 0, sizeof(serv_adr));
	serv_adr.sin_family = AF_INET;
	serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
	serv_adr.sin_port = htons(9527);
    
    // 初始化一个互斥锁，保护共享资源。NULL表示默认属性
	pthread_mutex_init(&mutex1, NULL);
	if (bind(serv_sock, (struct sockaddr*)&serv_adr, sizeof(serv_adr)) == -1) {
		printf("bind error %d %s", errno, strerror(errno));
		return;
	}
	if (listen(serv_sock, 5) == -1) {
		printf("listen error %d %s", errno, strerror(errno));
		return;
	}

	for (;;) {//while(1)等价
        //accept函数会阻塞，直到有客户端连接。如果成功，返回一个新的套接字描述符clnt_sock
		clnt_sock = accept(serv_sock, (struct sockaddr*)&clnt_adr, &clnt_adr_sz);
		if (clnt_sock == -1) {
			printf("accept error %d %s", errno, strerror(errno));
			break;
		}
        // 锁定互斥锁mutex1，以防止多个线程同时访问共享资源clnt_socks和clnt_cnt。
		pthread_mutex_lock(&mutex1);
		clnt_socks[clnt_cnt++] = clnt_sock;
        // 解锁互斥锁mutex1。
		pthread_mutex_unlock(&mutex1);
		pthread_t thread;
        // 创建一个新的线程，执行handle_clnt函数
// 并将新接受的客户端套接字描述符的地址作为参数传递给handle_clnt函数。NULL表示使用默认的线程属性。
		pthread_create(&thread, NULL, handle_clnt, &clnt_socks[clnt_cnt - 1]);
	}
	close(serv_sock);
    // 销毁互斥锁mutex1。
	pthread_mutex_destroy(&mutex1);
}
sem_t semid;
char name[64] = "[DEFAULT]";
void* client_send_msg(void* arg) {
	pthread_detach(pthread_self());
    // 从传入的参数中获取客户端套接字描述符
	int sock = *(int*)arg;
	char msg[256] = "";
	char buffer[1024];
	while (true) {
		memset(buffer, 0, sizeof(buffer));
		fgets(msg, sizeof(msg), stdin);
		if ((strcmp(msg, "q\n") == 0) || (strcmp(msg, "Q\n") == 0)) {
			break;
		}
		snprintf(buffer, sizeof(msg), "%s %s", name, msg);
		write(sock, buffer, strlen(buffer));
	}
	sem_post(&semid);  // V操作
	pthread_exit(NULL);
}

void* client_recv_msg(void* arg) {
	pthread_detach(pthread_self());
	int sock = *(int*)arg;
	char msg[256] = "";
	while (true) {
        // 套接字sock读取数据，存储到msg中，str_len为读取的字节数。
		ssize_t str_len = read(sock, msg, sizeof(msg));
		if (str_len <= 0)break;
		fputs(msg, stdout);
		memset(msg, 0, str_len);
	}
	sem_post(&semid);  // V 操作
	pthread_exit(NULL); // 线程结束
}

void client98() {
	int sock = socket(PF_INET, SOCK_STREAM, 0);
	struct sockaddr_in serv_addr;
	memset(&serv_addr, 0, sizeof(serv_addr));
	serv_addr.sin_family = AF_INET;
	serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
	serv_addr.sin_port = htons(9527);
	fputs("input your name:", stdout);
	scanf("%s", name);
	if (connect(sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) == -1) {
		printf("connect error %d %s", errno, strerror(errno));
		return;
	}
    // 定义两个线程标识符thsend和threcv，分别用于发送和接收消息的线程。
	pthread_t thsend, threcv;
    // 初始化信号量semid。0表示信号量是进程内共享的，-1是信号量的初始值。
	sem_init(&semid, 0, -1);
    
    // 给线程绑定主函数
    // (void*)&sock  参数
	pthread_create(&thsend, NULL, client_send_msg, (void*)&sock);
	pthread_create(&threcv, NULL, client_recv_msg, (void*)&sock);
	sem_wait(&semid);  // P操作
	close(sock);
}

void lession98(const char* arg)
{
	if (strcmp(arg, "s") == 0) {
		server98();
	}
	else {
		client98();
	}
}
```

