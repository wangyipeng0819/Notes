---
title: Linux系统编程03进程通信
date: 2025-01-12 14:30:04
categories:
- linux
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686ca567e2bda.png
---

### 进程的概念以及应用

​	占用内存空间的正在运行的程序即为进程

​	从操作系统的角度看，进程是程序流的基本单位，若创建多个进程，则操作系统将同时运行。有时一个程序运行过程中也会产生多个进程。

CPU核的个数与进程数   ：一个CPU中可能包含多个运算设备（核）。核的个数与可同时运行的进程数相同。

若进程数超过了核数，进程将分时使用CPU资源。

进程ID：1 要分配给操作系统启动后的（用于协助操作系统）首个进程，因此用户进程无法得到ID值1

**创建进程**

通过调用fork函数创建进程

```c++
pid_t pid = fork();
```

fork函数将创建调用的进程副本，也就是说，是复制正在运行的、调用fork函数的进程。另外两个进程都将执行fork调用后的语句（fork函数返回后）。同一个进程、复制相同的内存空间，之后的程序要根据fork函数的返回值加以区分。即利用fork函数的如下特点区分程序执行流程。

​	父进程：fork函数返回子进程ID.

​	子进程：fork函数返回0

**僵尸进程**

产生僵尸进程的原因：

​	调用fork函数产生子进程的终止方式。

 1. 传递参数并调用exit函数。

 2. main函数中执行return语句并返回值。

    向exit函数传递的参数值和main函数的return语句返回值都是会传递给操作系统。而操作系统不会销毁子进程，直到这些值传递给产生改子进程的父进程。处在这种状态下的进程就是僵尸进程。将子进程变成僵尸进程的正是操作系统。

僵尸进程怎么销毁呢？     应该向创建子进程的父进程传递子进程的exit参数值或return语句的返回值。

​	操作系统不会主动把这些值传递给父进程。只有父进程主动发起请求（函数调用）时，操作系统才会传递该值。如果父进程未主动要求获得子进程的结束状态值，操作系统将一直保存，并让子进程长时间处于僵尸进程状态。也就是说，父母要负责收回自己生的孩子。我们来创建一个僵尸进程。

##### **信号处理**

我们已经知道了进程创建及销毁方法，但还有一个问题没解决。

"子进程究竟何时终止?调用waitpid函数后要无休止地等待吗?"

父进程往往与子进程一样繁忙，因此不能只调用waitpid函数以等待子进程终止。

方法：

***\*向操作系统求助\****

子进程终止的识别主体是操作系统，因此，若操作系统能把如下信息告诉正忙于工作的父进程，将有助于构建高效的程序。

"嘿，父进程!你创建的子进程终止了!"

此时父进程将暂时放下工作，处理子进程终止相关事宜。这就是信号处理（Signal Handling）机制。此处的"信号"是在**特定事件**发生时由**操作系统**向进程发送的消息。另外，为了响应该消息，执行与消息相关的自定义操作的过程称为"信号处理"。

![](https://bu.dusays.com/2025/03/05/67c8037a2462e.png)

防止僵尸进程的代码

```c++
// 处理SIGCHLD 信号函数
void read_childproc(int sig)
{
    pid_t pid;
    int status;
    // 回收僵尸进程
    pid = waitpid(-1,&status,WNOHANG);
    printf("Removed proc id: %d \n", pid);
}


// main函数中防止僵尸进程的代码
pid_t pid;
struct sigaction act;
socklen_t adr_sz;

// 函数名字就是一个指针，这里就是指针的赋值
// act.sa_handler是一个函数指针
act.sa_handler = read_childproc;
sigemptyset(&act.sa_mask);
act.sa_flags = 0;
state = sigaction(SIGCHLD,&act,0);
if (state == -1)
{
    error_handling("sigaction() error");
}
```

R SIGALRM∶已到通过调用alarm函数注册的时间。

R SIGINT∶输入CTRL+C。  

R SIGCHLD∶子进程终止。（英文为child）

1、"子进程终止则调用mychild函数。"

代码：signal(SIGCHLD, mychild);

此时mychild函数的参数应为int，返回值类型应为void。对应signal函数的第二个参数。另外，常数SIGCHLD表示子进程终止的情况，应成为signal函数的第一个参数。

2、"已到通过alarm函数注册的时间，请调用timeout函数。"

3、"输入CTRL+C时调用keycontrol函数。"

```c++
void lession82()
{
	pid_t pid = fork();
	if (pid > 0)
	{
        // 父进程睡眠30秒  给子进程足够的时间来执行。
		sleep(30);
		int status;
		waitpid(pid, &status, 0);
	}
	else {
		printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
		exit(-1);
	}
}
```

**信号处理函数**

```c++
void signal_func(int sig)
{
    switch (sig)
    {
    case SIGALRM:  // 闹钟信号
        printf("tid %d pid %d\n", pthread_self(), getpid());
        alarm(2);
        break;
    case SIGINT:   // 通常由用户按下 Ctrl + C 产生
        printf("Ctrl + C press...");
        exit(0);
        break;
    }
}

void lession83()
{
    // 打印当前的线程id和  进程id
    printf("=====tid %d pid %d\n", pthread_self(), getpid());
    
    // 把signal_func函数分别注册为SIGALRM信号、SIGINT信号的处理函数
    signal(SIGALRM, signal_func);
    signal(SIGINT, signal_func);
    
    // 使用 alarm(1) 设置一个 1 秒的闹钟，1 秒后会触发 SIGALRM 信号。
    alarm(1);
    while (true) {
        sleep(3);
        printf("=====tid %d pid %d\n", pthread_self(), getpid());
    }
}
```

**利用Sigaction函数 进行信号处理**

​	singnal在UNIX系列的不同操作系统中可能存在区别，但sigaction完全相同。



**进程通信：为了方便进程交换信息，在内核提供缓冲区进行数据交换的机制。**

操作系统提供了两个进程可以同时访问的内存空间。

交换数据时基于开发 / 权限 的

缓冲区两边像是有俩齿轮一样，如果是比作商店的话，一边是放进去东西，一边是拿出去东西，形成了消息处理的流水线。

#### 进程间通信：管道

**单管道**

![](https://bu.dusays.com/2025/01/12/6783c9daa4ad9.png)

为了完成进程间通信，需要创建管道。管道并非属于进程的资源，而是和套接字一样，属于操作系统（也就不是fork函数的复制对象）。所以，两个进程通过操作系统提供的内存空间进行通信。



父进程调用该函数时将**创建管道**，同时获取对应于出入口的文件描述符号

`Filedes[0]` 通过管道接收数据时使用的文件描述符，即管道出口。

`Filedes[1]` 通过管道传输数据时使用的文件描述符，即管道入口。

父进程创建子进程的时候，自己的资源镜像复制到子进程里面，所以父进程创建了管道子进程是有这个信息的。

```c++
void singlePipe()
{
	// 两个文件描述符
	int fds[2] = { -1,-1 };
	char str[128] = "send by sub process!!!\n";
	char buf[128] = "";
	pipe(fds);
	pid_t pid = fork();
	if (pid == 0)
	{
		write(fds[1], str, sizeof(str));
	}
	else {
		read(fds[0], buf, sizeof(buf));
		printf("%s(%d): %s server: %s\n", __FILE__, __LINE__, __FUNCTION__, buf);
	}

}
```

**双管道**

1个管道无法完成双向通信任务，有时候需要创建2个管道，各自负责不同的数据流动即可。

![](https://bu.dusays.com/2025/01/12/6783d16c0a2b6.png)

```c++
void doublePipe()
{
    // 两个方向各有俩文件描述符
	int s2c[2], c2s[2];
	pipe(s2c);
	pipe(c2s);
	pid_t pid = fork();
	if (pid == 0)
	{
		char buffer[256] = "";
		write(c2s[1], "hello,i am subprocess!\n", 23);
		read(s2c[0], buffer, sizeof(buffer));
		printf("%s(%d): %s server: %s\n", __FILE__, __LINE__, __FUNCTION__, buffer);
	}
	else {
		char buffer[256] = "";

		read(c2s[0], buffer, sizeof(buffer));
		write(s2c[1], "hello,i am mainprocess\n", 23);
		printf("%s(%d): %s server: %s\n", __FILE__, __LINE__, __FUNCTION__, buffer);
	}
	printf("%s(%d): %s server: %d\n", __FILE__, __LINE__, __FUNCTION__, getpid());
}
```

#### 进程间通信：FIFO

​	对比pipe管道，他已经可以完成在两个进程之间通信的任务，不过它似乎完成的不够好，也可以说是不够彻底。它只能在两个有**亲戚关系**的进程之间进行通信，这就大大限制了pipe管道的应用范围。

​	`fifo`管道的本质是操作系统中的命名文件，当然Linux的理念就是万物皆文件，它在操作系统中以命名文件的形式存在，我们可以在操作系统中看见`fifo`管道，在你有权限的情况下，甚至可以读写他们。

内核会针对`fifo`文件开辟一个缓冲区，操作FIFO文件，可以操作缓冲区，实现进程通信。一旦使用`mkfifo`创建了一个FIFO，就可以使用open打开它，常见的文件IO函数都可以用于`FIFO`。如：`close`、`read`、`write`、`unlink`等 .

这样的话  一个进程对应一个管道，大大减少了管道的数量。

**打开FIFO文件的时候，read端会阻塞等待write端打开open，write端同理，也会阻塞等待另外一端打开。**

大概就是下图所示

![](https://bu.dusays.com/2025/03/05/67c85594c01ab.png)

#### 进程间通信：共享内存

​	共享内存允许不同进程之间**共享同一段逻辑内存**，对于这段内存，它们都能访问，或者修改它，没有任何限制。所以它是**进程间传递大量数据**的一种非常有效的方式。“共享内存允许不同进程之间共享同一段逻辑内存”，这里是逻辑内存。也就是说共享内存的进程访问的可以不是同一段物理内存，这个没有明确的规定，但是大多数的系统实现都将进程之间的共享内存安排为同一段物理内存。

**使用共享内存的步骤通常是：**

1）创建或获取一段共享内存；

2）将上一步创建的共享内存映射到该进程的地址空间；

3）访问共享内存；

4）将共享内存从当前的进程地址空间分离；

5）删除这段共享内存；

```c++
void lession90()
{
	pid_t pid = fork();
	if (pid > 0)
	{
		// . 表明当前的路径 
		//  ftok(".",1)文件目录不一样   后面的数字也不一样 
        // 用于创建或获取一个共享内存段。如果IPC_CREAT标志被设置，且指定的内存段不存在，则创建一个新的共享内存段。
		int shm_id = shmget(ftok(".",1), sizeof(STUDENT), IPC_CREAT | 0666);
		if (shm_id == -1)
		{
			printf("%s(%d): %s create share memeory failed!\n", __FILE__, __LINE__, __FUNCTION__);
			return;
		}
		// 使用函数shmat()来映射共享内存
		PSTUDENT pStu = (PSTUDENT)shmat(shm_id, NULL, 0);
		pStu->id = 666666;
		strcpy(pStu->name, "买买提");
		pStu->age = 18;
		pStu->sex = true;
		pStu->sig = 99;
		// 两边需要同步   不同步直接删除映射就拿不到数据
		while (pStu->sig == 99)
		{
			usleep(1000);
		}
		// 删除映射
		shmdt(pStu);
		shmctl(shm_id, IPC_RMID, NULL);
	}
	else { // 子进程
		usleep(500000); // 等待父进程写入
		int shm_id = shmget(ftok(".", 1), sizeof(STUDENT), IPC_CREAT | 0666);
		if (shm_id == -1)
		{
			printf("%s(%d): %s create share memeory failed!\n", __FILE__, __LINE__, __FUNCTION__);
			return;
		}
		// 使用函数shmat()来映射共享内存
		PSTUDENT pStu = (PSTUDENT)shmat(shm_id, NULL, 0);
		while (pStu->sig != 99)
		{
			usleep(1000);
		}
		printf("%d, %s,%d,%s\n", pStu->id, pStu->name, pStu->age, pStu->sex ? "男" : "女");
		pStu->sig = 0;
		shmdt(pStu);
		shmctl(shm_id, IPC_RMID, NULL);
	}
}
```



#### 进程间通信：信号量

- 假设没有信号量，父子进程可能会同时访问共享内存，导致数据不一致。例如，父进程可能正在写入数据，而子进程同时尝试读取尚未完全写入的数据。通过信号量的 P 操作（`semop`函数，`sem_op`为 - 1）和 V 操作（`semop`函数，`sem_op`为 1），可以确保在某一时刻只有一个进程能够访问共享内存中的关键区域。比如，子进程在读取共享内存中的学生信息之前，先对信号量进行 P 操作，等待父进程完成写入并执行 V 操作释放资源后，子进程才能读取，从而避免了数据竞争和不一致性。

原来共享内存有很严重的时间差，降低了效率。

为了防止出现因多个程序同时访问一个共享资源而引发的一系列问题，我们需要一种方法，它可以通过生成并使用令牌来授权，在任一时刻只能有一个执行线程访问代码的临界区域。临界区域是指执行数据更新的代码需要独占式地执行。而信号量就可以提供这样的一种访问机制，让一个临界区同一时间只有一个线程在访问它，也就是说信号量是用来调协进程对共享资源的访问的。

```c++
void lession91()
{
	pid_t pid = fork();
	if (pid > 0) {//父进程
		// 使用ftok函数生成一个唯一的键值key，用于创建信号量。
		key_t key = ftok(".", 2);

		// semget函数创建一个信号量集，包含 2 个信号量，权限为0666。
		int sem_id = semget(key, 2, IPC_CREAT | 0666);

		//使用semctl函数初始化两个信号量的值为 0。
		semctl(sem_id, 0, SETVAL, 0);
		semctl(sem_id, 1, SETVAL, 0);
		//调用shmget函数创建一个共享内存段，大小为sizeof(STUDENT)，权限为0666。
		int shm_id = shmget(ftok(".", 1), sizeof(STUDENT), IPC_CREAT | 0666);
		if (shm_id == -1) {
			printf("%s(%d):%s create share memeory failed!\n", __FILE__, __LINE__, __FUNCTION__);
			return;
		}

		//将共享内存段附加到进程的地址空间，返回一个指向共享内存的指针pStu。
		//映射
		PSTUDENT pStu = (PSTUDENT)shmat(shm_id, NULL, 0);
		pStu->id = 666666;
		strcpy(pStu->name, "abcdefghijklmn");
		pStu->age = 18;
		pStu->sex = true;
		//信号量
		sembuf sop = {
			.sem_num = 0,
			.sem_op = 1  // 为正数，semop就是V操作
		};

		// 获取信号量的值，
		printf("%s(%d):%s %d\n", __FILE__, __LINE__, __FUNCTION__, semctl(sem_id, 0, GETVAL));
		semop(sem_id, &sop, 1);//V操作
		printf("%s(%d):%s %d\n", __FILE__, __LINE__, __FUNCTION__, semctl(sem_id, 0, GETVAL));
		sop.sem_num = 1;
		sop.sem_op = -1;  // 为负数，semop为P操作
		printf("%s(%d):%s %d\n", __FILE__, __LINE__, __FUNCTION__, semctl(sem_id, 1, GETVAL));
		semop(sem_id, &sop, 1);//P操作
		printf("%s(%d):%s %d\n", __FILE__, __LINE__, __FUNCTION__, semctl(sem_id, 1, GETVAL));

		//将共享内存段从进程的地址空间分离
		shmdt(pStu);
		//删除共享内存段。
		shmctl(shm_id, IPC_RMID, NULL);
		//删除信号量
		semctl(sem_id, 0, IPC_RMID);
		semctl(sem_id, 1, IPC_RMID);
		sleep(10);
	}
	else {//子进程
		key_t key = ftok(".", 2);
		//semget和shmget函数获取已存在的信号量和共享内存段。
		int sem_id = semget(key, 2, IPC_CREAT);
		int shm_id = shmget(ftok(".", 1), sizeof(STUDENT), IPC_CREAT | 0666);
		if (shm_id == -1) {
			printf("%s(%d):%s create share memeory failed!\n", __FILE__, __LINE__, __FUNCTION__);
			return;
		}

		sembuf sop = {
			.sem_num = 0,
			.sem_op = -1//P操作
		};
		printf("%s(%d):%s %d\n", __FILE__, __LINE__, __FUNCTION__, semctl(sem_id, 0, GETVAL));
		semop(sem_id, &sop, 1);//P操作
		printf("%s(%d):%s %d\n", __FILE__, __LINE__, __FUNCTION__, semctl(sem_id, 0, GETVAL));
		PSTUDENT pStu = (PSTUDENT)shmat(shm_id, NULL, 0);
		//信号量
		printf("%d ,%s,%d,%s\n", pStu->id, pStu->name, pStu->age, pStu->sex ? "male" : "female");
		sop.sem_num = 1;
		sop.sem_op = 1;//V操作
		printf("%s(%d):%s %d\n", __FILE__, __LINE__, __FUNCTION__, semctl(sem_id, 1, GETVAL));
		semop(sem_id, &sop, 1);//V操作
		printf("%s(%d):%s %d\n", __FILE__, __LINE__, __FUNCTION__, semctl(sem_id, 1, GETVAL));
		sleep(10);
		//共享内存段分离
		shmdt(pStu);
		shmctl(shm_id, IPC_RMID, NULL);
	}
}
```



#### 进程间通信：消息队列

**消息队列**提供了一种从一个进程向另一个进程发送一个数据块的方法。 每个数据块都被认为含有一个类型，接收进程可以独立地接收含有不同类型的数据结构。我们可以通过发送消息来避免命名管道的同步和阻塞问题。但是消息队列与命名管道FIFO一样，每个数据块都有一个最大长度的限制。

1️⃣ `msgget`函数  创建和访问一个消息队列   在示例中，子进程和父进程都有

2️⃣ `msgsnd`函数  用来把消息添加到队列。

3️⃣ `msgrcv`函数  用来从一个消息队列获取消息。

4️⃣ `msgctl` 函数 用来控制消息队列，它与共享内存的`shmctl`函数类似，删除消息队列。

```c++
void lession92()
{
	pid_t pid = fork();
	if (pid > 0)
	{
		// msg 的接受
		// 有的linux系统无法实现
		int msg_id = msgget(ftok(".", 3), IPC_CREAT | 0666);
		printf("%s(%d):%s %d %d\n", __FILE__, __LINE__, __FUNCTION__, msg_id, errno);
		printf("%s\n", strerror(errno));
		if (msg_id == -1) return;
		MSG msg;
		memset(&msg, 0, sizeof(msg));
		while (true)
		{
			//msgrcv 函数的参数依次为：消息队列 ID、接收消息的缓冲区、消息数据部分的大小、期望接收的消息类型（这里为 1）、接收标志（这里为 0）。
			ssize_t ret = msgrcv(msg_id, &msg, sizeof(msg.data), 1, 0);
			if (ret == -1)
			{
				usleep(1000);
				printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
			}
			else break;
		}
		printf("%d name: %s age: %d msg: %s\n",
			msg.data.id, msg.data.name, msg.data.age, msg.data.message);
		getchar();
		msgctl(msg_id, IPC_RMID, 0);
	}
	else {
		int msg_id = msgget(ftok(".", 3), IPC_CREAT | 0666);
		MSG msg;
		memset(&msg, 0, sizeof(msg));
		msg.type = 1;
		msg.data.id = 6666;
		strcpy(msg.data.name, "Bingo");
		msg.data.age = 18;
		strcpy(msg.data.message, "hello world!");
		printf("***%d name: %s age: %d msg: %s\n",
			msg.data.id, msg.data.name, msg.data.age, msg.data.message);
		//msgsnd 函数将消息发送到消息队列
		msgsnd(msg_id, &msg, sizeof(msg.data), 0);
		sleep(1);
		//使用 msgctl 函数删除消息队列。
		msgctl(msg_id, IPC_RMID, 0);
	}
}
```

待续。。。。

