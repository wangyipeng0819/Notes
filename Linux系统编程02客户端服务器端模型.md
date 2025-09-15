---
title: Linux系统编程02客户端服务器端模型
date: 2025-01-12 14:30:02
categories:
- linux
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686ca567e2bda.png
---

### 简单的TCP 客户端 - 服务器模型

#### `run_client`函数

```c++
void run_client()
{
    // 创建一个TCP套接字
    int client = socket(PF_INET, SOCK_STREAM, 0);
    /*
    PF_INET 指定协议族为 IPv4
    SOCK_STREAM：指定套接字类型为流套接字
    0：指定协议类型，这里使用默认协议
    如果socket函数成功创建套接字，会返回一个非负整数作为套接字描述符；如果失败，返回-1。
    */
    // 定义一个struct sockaddr_in类型的变量addr，用于存储地址信息。
    struct sockaddr_in servaddr;
    
    // 初始化服务器地址结构体
    memset(&servaddr, 0, sizeof(servaddr));
    
    //指定地址族为 IPv4，AF代表 “Address Family”，AF_INET和前面的PF_INET相对应。
    servaddr.sin_family = AF_INET;
    // 设置服务器IP地址为本地回环地址127.0.0.1
    servaddr.sin_addr.s_addr = inet_addr("127.0.0.1");
    // 设置服务器端口号为9527
    servaddr.sin_port = htons(9527);

    // 连接到服务器
    // 将客户端套接字（client）连接到指定的服务器地址（servaddr）。
    int ret = connect(client, (struct sockaddr*)&servaddr, sizeof(servaddr));
    // 连接成功，ret的值为 0；如果连接失败，ret的值为 -1，并且会设置相应的错误码。
    if (ret == 0)
    {
        // 输出当前文件、行号和函数名，表示连接成功
        printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
        char buffer[256] = "hello,hello\n";
        // 向服务器发送消息
        // client是套接字描述符，buffer是要发送的数据缓冲区，sizeof(buffer)表示要发送的数据长度。
        write(client, buffer, sizeof(buffer));
        // 清空缓冲区
        memset(buffer, 0, sizeof(buffer));
        // 从服务器读取响应
        read(client, buffer, sizeof(buffer));
        // 输出服务器的响应
        std::cout << buffer;
    }
    else {
        // 输出连接失败的错误信息
        printf("%s(%d):%s %d\n", __FILE__, __LINE__, __FUNCTION__, ret);
    }
    // 关闭套接字
    close(client);
    // 输出客户端完成信息
    std::cout << "client done!!" << std::endl;
}
```

#### `run_server()`函数

```c++
void lession63_()
{
    int server, client;
    struct sockaddr_in seraddr, clientaddr;
    socklen_t cliaddrlen;

    // 创建一个TCP套接字
    server = socket(PF_INET, SOCK_STREAM, 0);
    if (server < 0) {
        // 输出错误信息并返回
        std::cout << "error！！\n";
        return;
    }
    // 初始化服务器地址结构体
    memset(&seraddr, 0, sizeof(seraddr));

    // 设置地址族为IPv4
    seraddr.sin_family = AF_INET;
    // 服务器绑定到所有网络接口上
    seraddr.sin_addr.s_addr = inet_addr("0.0.0.0");
    // 设置服务器端口号为9527
    seraddr.sin_port = htons(9527);

    // 将套接字绑定到指定地址和端口
    int ret = bind(server, (struct sockaddr*)&seraddr, sizeof(seraddr));
    if (ret == -1)
    {
        // 输出绑定失败信息并关闭套接字
        std::cout << "bind failed!" << std::endl;
        close(server);
        return;
    }

    // 开始监听连接，最大连接数为3
    // 使用listen函数使服务器套接字server开始监听传入的连接请求，最大允许连接数为 3。
    // listen函数返回 0 表示成功，返回 -1 表示失败。
    ret = listen(server, 3);
    if (ret == -1)
    {
        // 输出监听失败信息并关闭套接字
        std::cout << "listen failed!" << std::endl;
        close(server);
        return;
    }

    char buffer[1024];
    while (1)
    {
        // 清空缓冲区
        memset(buffer, 0, sizeof(buffer));
        // 输出当前文件、行号和函数名
        printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
        // 接受客户端连接
        client = accept(server, (struct sockaddr*)&clientaddr, &cliaddrlen);
        if (client == -1)
        {
            // 输出接受客户端连接失败信息并关闭服务器
            std::cout << "client failed!" << std::endl;
            close(server);
            return;
        }
        // 输出当前文件、行号和函数名
        printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
        // 从客户端读取消息
        read(client, buffer, sizeof(buffer));
        // 将接收到的消息回显给客户端
        ssize_t len = write(client, buffer, strlen(buffer));
        if (len!= (ssize_t)strlen(buffer))
        {
            // 输出写回显失败信息并关闭服务器
            std::cout << "write failed!" << std::endl;
            close(server);
            return;
        }
        // 输出当前文件、行号和函数名
        printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
        // 关闭与客户端的连接
        close(client);
    }
    // 关闭服务器套接字
    close(server);
    // 输出当前文件、行号和函数名
    printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
}
```

#### `lession63()`函数

```c++
void lession63()
{
    // 创建子进程
    pid_t pid = fork();
    if (pid == 0)
    {
        // 子进程，作为客户端运行
        sleep(1);
        // 运行客户端函数两次
        run_client();
        run_client();
    }
    else if (pid > 0)
    {
        // 父进程，作为服务器端运行
        lession63_();
        int status = 0;
        // 等待子进程结束
        wait(&status);
    }
    else {
        // 输出fork失败信息
        std::cout << "fork failed!" << pid << std::endl;
    }
}
```

#### 总结

**TCP连接客户端的流程**

1️⃣创建套接字

2️⃣初始化服务器地址

3️⃣连接服务器。connect 成功返回0，失败返回-1

4️⃣通信   write read

5️⃣关闭套接字 （close关闭）

**TCP连接服务器的流程**

1️⃣创建套接字

2️⃣初始化服务器地址

3️⃣绑定套接字，`bind` 函数将服务器套接字 `server` 绑定到指定的地址和端口。

4️⃣监听连接，`listen`函数使得服务器套接字开始监听传入的连接请求

5️⃣`accept`接受并处理客户端连接，返回一个新的套接字描述符用于与客户端通信。   `read`、`write` 进行通信。

6️⃣关闭服务器套接字，`close(server)`

#### 回声服务器

**回声服务器**：将从客户端收到的数据原样返回给客户端，即“回声”。

客户端部分 分为 发送  和 接受两部分

```c++
void run_client64()
{
    // 创建一个TCP套接字，使用IPV4地址族
	int client = socket(PF_INET, SOCK_STREAM, 0);
    // 定义一个存储服务器地址信息的结构体
	struct sockaddr_in servaddr;
	memset(&servaddr, 0, sizeof(servaddr));
    // 设置地址族为IPV4
	servaddr.sin_family = AF_INET;
    // 将点分十进制的IP地址  转换为二进制形式并存储到结构体中
	servaddr.sin_addr.s_addr = inet_addr("127.0.0.1");
	// 将端口号从主机字节序转换为网络字节序
    servaddr.sin_port = htons(9527);
	int ret = connect(client, (struct sockaddr*)&servaddr, sizeof(servaddr));
	while (ret == 0)
	{
		printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
		// 定义一个缓冲区用于存储用户输入的消息
        char buffer[256] = "";
		fputs("Input message(Q to quit):", stdout);
        // 从标准输入读取一行消息到缓冲区
		fgets(buffer, sizeof(buffer), stdin);
		if ((strcmp(buffer, "q\n") == 0 || strcmp(buffer, "Q\n") == 0))
		{
			break;
		}
        // 获取用户输入消息的长度
		size_t len = strlen(buffer);
        // 获取用户输入消息的长度
		size_t send_len = 0;
        
		while (send_len < len)
		{
            // 向服务器发送消息，从缓冲区的未发送部分开始发送
			ssize_t ret = write(client, buffer + send_len, len -send_len);
			// 如果发送失败
            if (ret <= 0)
			{
				fputs("write failed!\n", stdout);
				close(client);
				std::cout << "client done!!" << std::endl;
				return;
			}
            // 更新已发送的字节数
			send_len += (size_t)ret;
		}
		
		memset(buffer, 0, sizeof(buffer));
        // 已接收的字节数
		size_t read_len = 0;
        // 循环接收服务器的响应，直到接收的字节数达到发送的消息长度
		while (read_len < len)
		{
            // 从服务器读取响应消息
			ssize_t ret = read(client, buffer + read_len, len - read_len);
			// 读取失败
            if (ret <= 0)
			{
				fputs("read failed!\n", stdout);
				close(client);
				std::cout << "client done!!" << std::endl;
				return;
			}
            // 更新已经接收的字节数
			read_len += (size_t)ret;
		}
		std::cout << "from server " << buffer;
	}
    // 关闭套接字
	close(client);
    // 输出客户端结束信息
	std::cout << "client done!!" << std::endl;
}
```

服务器端

```c++
void server64()
{
    // 定义服务器和客户端的套接字描述符
	int server, client;
    // 定义存储服务器和客户端地址信息的结构体
	struct sockaddr_in seraddr, clientaddr;
    // 定义客户端地址结构体的长度
	socklen_t cliaddrlen;
	// 创建一个 TCP 套接字，使用 IPv4 地址族
	server = socket(PF_INET, SOCK_STREAM, 0);
	if (server < 0) {
		std::cout << "create socket failed！！\n";
		return;
	}
	memset(&seraddr, 0, sizeof(seraddr));
	seraddr.sin_family = AF_INET;
	seraddr.sin_addr.s_addr = inet_addr("0.0.0.0");
	seraddr.sin_port = htons(9527);
	int ret = bind(server, (struct sockaddr*)&seraddr, sizeof(seraddr));
	// 如果绑定失败
    if (ret == -1)
	{
        // 输出错误信息
		std::cout << "bind failed!" << std::endl;
        // 关闭套接字
		close(server);
		return;
	}
    // 开始监听客户端连接  在最大允许3个连接的请求排队
	ret = listen(server, 3);
	if (ret == -1)
	{
		std::cout << "listen failed!" << std::endl;
		close(server);
		return;
	}
	char buffer[1024];
    // 处理2个客户端连接
	for (int i = 0; i < 2; i ++ )
	{
		memset(buffer, 0, sizeof(buffer));
		printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
         // 接受客户端连接请求，返回一个新的套接字描述符
		client = accept(server, (struct sockaddr*)&clientaddr, &cliaddrlen);
		if (client == -1)
		{
			std::cout << "client failed!" << std::endl;
			close(server);
			return;
		}
		printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
		// 已读取的字节数
        ssize_t len = 0;
        // 循环读取客户端发送的消息
		while (( len = read(client, buffer, sizeof(buffer)) ) > 0)
		{
            // 将接收到的消息原样返回给客户端
			len = write(client, buffer, len);
			
            if (len != (ssize_t)strlen(buffer))
			//if (len != write(client, buffer, len))
			{
				std::cout << "write failed! len: " <<
                    len << "buffer: " << buffer << std::endl;
				close(server);
				return;
			}
			memset(buffer, 0, len);
		}
		if (len <= 0)
		{
            // 输出错误信息
			std::cout << "read failed!" << std::endl;
            // 关闭服务器套接字
			close(server);
			return;
		}
		printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
		close(client);  // 可以注释
	}
	close(server);  // 服务关闭的时候，客户端自动关闭
	printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
}
```

**主函数：启动服务器和客户端**

```c++
// 回声服务器
void lession64()
{
    // 创建一个子进程
	pid_t pid = fork();
    // 如果是子进程
	if (pid == 0)
	{
		server64();
	} 
	else if (pid > 0) // 如果是父进程
	{
        // 启动两次客户端程序
		for (int i = 0; i < 2; i ++)
			//sleep(1);
			run_client64();
        // 用于存储子进程的退出状态
		int status = 0;
        // 等待子进程结束
		wait(&status);
	}
	else {
		std::cout << "fork failed!" << pid << std::endl;
	}
}
```

**回声服务器存在的问题**

（上述代码已经更正了）代码有个错误的假设，每次调用read、write函数时都会以<font color="FF0000">字符串为单位执行实际的I/O操作。</font>每次调用write函数都会传递一个字符串，这种假设也算合理。但是，多次调用write函数传递的字符串有可能一次性传递到服务器端。此时客户端有可能从服务器端收到多个字符串，这不是我们希望看到的结果。

需要考虑下面的情况：<font color="FF0000">字符串太长，需要分2个数据包发送！</font>







#### 问题汇总

**客户端设置服务器地址**

客户端设置服务器地址是为了明确要连接的目标服务器位置。它通过`struct sockaddr_in`结构体来存储服务器的网络地址信息，包括 `IP` 地址和端口号。

**服务器端设置服务器地址**

服务器端设置自己的地址主要是为了将服务器套接字绑定到一个特定的网络接口和端口上，以便接收来自客户端的连接请求。



**`connect()`函数  和  `bind()`函数   的区别**

- `connect`函数主要用于客户端套接字与服务器端套接字建立连接。它尝试将客户端的网络端点（由 IP 地址和端口号标识）与服务器的网络端点进行关联，从而建立起一条通信链路，使得客户端和服务器可以互相发送和接收数据。

  返回0：客户端与服务器之间的连接已经建立，可以开始进行数据读写操作了。

  返回-1：执行 失败的时候会返回-1，errno变量来指示具体错误原因。

- `bind`函数主要用于服务器端将套接字绑定到一个特定的本地网络端点（包括 IP 地址和端口号）。它的作用是让服务器在特定的网络地址和端口上监听客户端的连接请求，使得客户端能够准确地找到服务器并与之建立连接。

**`write`函数**

- `fd`：文件描述符，它可以代表一个打开的文件、套接字等。在网络编程中，通常是通过 `socket` 函数创建并连接好的套接字描述符。
- `buf`：指向要写入数据的缓冲区的指针。
- `count`：要写入的字节数。

返回值大于0：当 `write` 函数成功写入部分或全部数据时，它会返回实际写入的字节数。这个返回值可能小于、等于 `count` 参数指定的字节数。

- **等于 `count`**：表示成功写入了请求的所有字节。例如，你希望写入 10 个字节的数据，`write` 函数返回 10，说明这 10 个字节都成功写入到了文件描述符所关联的目标中。
- **小于 `count`**：可能是因为某些原因（如磁盘空间不足、网络缓冲区已满等），无法一次性写入所有请求的字节。在这种情况下，你可能需要再次调用 `write` 函数，从上次写入结束的位置继续写入剩余的数据。

返回值等于0：通常出现了一些异常情况

返回值为-1 ：`write`函数执行失败时，它会返回-1，并且会设置`errno`变量来指示具体的错误原因。

**这俩部分设置是否一样？**

- **客户端角度**
  - 客户端设置的服务器 IP 地址必须是服务器实际绑定并监听的 IP 地址之一。如果服务器绑定了特定的 IP 地址（例如`192.168.1.100`），客户端就需要使用这个 IP 地址来连接服务器。不过，如果服务器绑定的是`0.0.0.0`（表示监听所有本地网络接口），客户端可以使用服务器所在主机的任何一个有效本地 IP 地址来连接。
- **服务器角度**
  - 服务器绑定的 IP 地址决定了它在哪些网络接口上监听客户端连接。如果服务器有多个网络接口（例如有多个网卡，每个网卡有不同的 IP 地址），绑定`0.0.0.0`意味着在所有接口上监听；而绑定特定的 IP 地址则只在该接口监听。

**主机字节序和网络字节序**

大端字节序也成为网络字节序，数据的高位字节存于低地址，低位字节存于高地址。

小端字节序：数据低位字节存于低地址，高位字节存于高地址。



### 简单的 `UDP` 客户端 - 服务器模型

#### `server`()函数

```c++
int server(int argc, char* argv[])
{
	int ser_sock = -1;
	char message[512];
	struct sockaddr_in servaddr, clientaddr;
	socklen_t clientlen = 0;

	// 检查命令行参数
	if (argc != 2)
	{
		printf("usage:%s  <port>\n", argv[0]);
		handle_error("argement is error!:");
	}
    
    // 创建UDP套接字
	ser_sock = socket(PF_INET, SOCK_DGRAM, 0); // UDP
	if (ser_sock == -1)
	{
		handle_error("create socket failed:");
	}
	memset(&servaddr, 0, sizeof(servaddr));
    // 初始化服务器地址
	servaddr.sin_family = AF_INET;
	servaddr.sin_addr.s_addr = htonl(INADDR_ANY); // 0.0.0.0  全网段监听
    // 设置servaddr的端口号为从命令行参数argv[1]中获取的端口号
    // 通过htons函数将其转换为网络字节序。
	servaddr.sin_port = htons((short)atoi(argv[1]));
    
    // 绑定套接字到地址
	if (bind(ser_sock, (struct sockaddr*)&servaddr, sizeof(servaddr)) == -1)
	{
		handle_error("bind failed:");
	}
    // 消息处理循环
	for (int i = 0; i < 10; i++)
	{
		clientlen = sizeof(clientaddr);
        
        // 使用recvfrom函数从客户端接收数据
        /*
        ser_sock：服务器套接字描述符。
        0：表示默认的标志位。
        (struct sockaddr*)&clientaddr：用于存储客户端地址的结构体指针。
        
        recvfrom 函数返回接收到的数据长度。
        */
		ssize_t len = recvfrom(ser_sock, message, sizeof(message), 0, (struct sockaddr*)&clientaddr, &clientlen);
        
        // 使用sendto函数将接收到的数据回显给客户端
		sendto(ser_sock, message, len, 0, (struct sockaddr*)&clientaddr, clientlen);
	}
    // 关闭套接字
	close(ser_sock);
	return 0;
}
```

#### `client()`函数

```c++
int client(int argc, char* argv[])
{
	// 变量声明
	int client_sock;
	struct sockaddr_in serv_addr;
	socklen_t serv_len = sizeof(serv_addr);
	char message[512] = "";

	if (argc != 3)
	{
		printf("usage:%s is port\n", argv[0]);
		handle_error("argement error:");
	}
    // 创建 UDP 套接字
    // UDP 套接字是无连接的，通过SOCK_DGRAM参数来指定创建 UDP 类型的套接字。
	client_sock = socket(PF_INET, SOCK_DGRAM, 0);
	if (client_sock == -1)
	{
		handle_error("socket create failed！");
	}
	memset(&serv_addr, 0, sizeof(serv_addr));
    // 初始化服务器地址
	serv_addr.sin_family = AF_INET;
	serv_addr.sin_addr.s_addr = inet_addr(argv[1]);
	serv_addr.sin_port = htons((short)atoi(argv[2]));

    // 消息处理循环
	while (1)
	{
		printf("Input message(q to quit):");
		scanf("%s", message);
		if ((strcmp(message, "q") == 0) || (strcmp(message, "Q") == 0)) {
			printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
			break;
		}
		// 分配了地址
		printf("%s(%d):%s\n", __FILE__, __LINE__, __FUNCTION__);
		ssize_t len = sendto(client_sock, message, strlen(message), 0, (sockaddr*)&serv_addr, serv_len);
		memset(message, 0, len);
		recvfrom(client_sock, message, sizeof(message), 0, (sockaddr*)&serv_addr, &serv_len);
		printf("recv:%s\n", message);
	}
    // 关闭套接字并返回
	close(client_sock);
	return 0;
}
```

#### 总结

**`UDP`客户端的流程**

1️⃣ 创建套接字

2️⃣ 设置服务器地址端口

3️⃣ 使用`sendto`函数发送数据、`recvfrom`函数来接收数据。

4️⃣ 关闭套接字

**`UDP`服务器端的流程**

1️⃣ 创建套接字

2️⃣ 绑定地址和端口

3️⃣ 接受、处理数据

4️⃣ 关闭套接字

#### 问题汇总

`UDP`服务器端/客户端的实现方法。但如果仔细观察`UDP`客户端会发现，它缺少把`IP`和端口分配给套接字的过程。

`TCP`客户端调用connect函数自动完成此过程，`UDP`调用`sendto`函数时自动分配`IP`和端口号



**套接字类型只能在创建时决定，以后不能再更改。**

**`SO_SNDBUF &SO_RCVBUF`**

`SO_RCVBUF`是输入缓冲大小相关可选项，`SO_SNDBUF`是输出缓冲大小相关可选项。用这2 个可选项既可以读取当前I/O缓冲大小，也可以进行更改。通过下列示例读取创建套接字时默认的I/O缓冲大小。



**如何使用`socket`、`getsockopt`和`setsockopt`函数来获取和设置套接字的属性**

```c++
void lession76()
{
    // 声明了两个整数变量，分别用于存储 TCP 套接字描述符和 UDP 套接字描述符。
	int tcp_sock, udp_sock;
    
    // 声明并初始化一个整数变量optval，用于存储套接字选项的值。
	int optval = 0;
	socklen_t len = sizeof(optval);

    // 分别创建TCP/UDP 套接字
	tcp_sock = socket(PF_INET, SOCK_STREAM, 0);
	udp_sock = socket(PF_INET, SOCK_DGRAM, 0);

	printf("SOCK_STREAM:%d\n", SOCK_STREAM);
	printf("SOCK_DGRAM:%d\n", SOCK_DGRAM);

    // 获取套接字类型
    // 使用getsockopt函数获取 TCP 套接字的类型。
    // SOL_SOCKET表示通用套接字选项级别，SO_TYPE表示要获取的选项是套接字类型。
    // 获取到的值存储在optval中。
	getsockopt(tcp_sock, SOL_SOCKET, SO_TYPE, (void*)&optval, &len);
	printf("tcp_sock type is :%d\n", optval);
	optval = 0;
	getsockopt(udp_sock, SOL_SOCKET, SO_TYPE, (void*)&optval, &len);
	printf("udp_sock type is :%d\n", optval);
	optval = 0;
    // 获取和设置 TCP 套接字发送缓冲区大小
	getsockopt(tcp_sock, SOL_SOCKET, SO_SNDBUF, (void*)&optval, &len);
	printf("tcp_sock send buffer size is :%d\n", optval);
	optval = 1024 * 16;
	setsockopt(tcp_sock, SOL_SOCKET, SO_SNDBUF, (void*)&optval, len);
	getsockopt(tcp_sock, SOL_SOCKET, SO_SNDBUF, (void*)&optval, &len);
	printf("*tcp_sock send buffer size is :%d\n", optval);

    // 关闭套接字
	close(tcp_sock);
	close(udp_sock);
}
```

**套接字的多种可选项**

在客户端控制台输入Q消息，或通过CTRL+C终止程序。

​	在客户端输入Q消息调用close函数，向服务器端发送FIN消息并经过四次挥手的过程。

<font color="#FF0000">服务器端和客户端已建立连接的状态下，</font>向服务器端控制台输入CTRL+C，即强制关闭服务器端。

​	如果是CTRL+C，服务器端重新运行的时候将产生问题。如果用同一端口号重新运行服务器端，将输出“bind error” 消息，并且无法再次运行。这种情况下，大约三分钟后即可重新运行服务器端。

![](https://bu.dusays.com/2025/03/05/67c7e28519e2f.png)

上述两种方法唯一的区别就是谁先传输FIN消息。

![](https://bu.dusays.com/2025/03/05/67c7e2b6a33fb.png)

​	假设上图中主机A是服务器端，因为是主机A向B发送FIN消息，故可以想象成服务器端1在控制台输入CTRL+C。但问题是，套接字经过四次挥手过程后并非立即消除，而是要经过一段时间的Time-wait状态。当然，只有先断开连接的（先发送FIN消息的）主机才经过Time-wait状态。因此，若服务器端先断开连接，则无法立即重新运行。套接字处在Time-wait过程时，相应端口是正在使用的状态。因此，就像之前验证过的，bind函数调用过程中当然会发生错误。

​	不论是服务器端还是客户端都会有Time-wait过程。先断开连接的套接字必然会经过Time-wait过程。但是无需考虑客户端Time-wait状态。因为客户端套接字的端口号是任意指定的。

**<font color="#FF0000">为什么会有Time-wait状态？</font>**

​	如上图中假设主机A向主机B传输ACK消息（SEQ5001、ACK7502）后立即消除套接字。但最后这条ACK消息在传递途中丢失，未能传给主机B。这时会发生什么?主机B会认为之前自己发送的FIN消息（SEQ 7501、ACK 5001）未能抵达主机A，继而试图重传。但此时主机A已是完全终止的状态，因此主机B永远无法收到从主机A最后传来的ACK消息。相反，**若主机A的套接字处在Time-wait状态**，则会向主机B重传最后的ACK消息，主机B也可以正常终止。基于这些考虑，先传输FIN消息的主机应经过Time-wait过程。

**地址再分配**

![](https://bu.dusays.com/2025/03/05/67c7e5e01f1b4.png)

​	如上图所示，在主机A的四次握手过程中，如果最后的数据丢失，则主机B会认为主机A未能收到自己发送的FIN消息，因此重传。这时，收到FIN消息的主机A将重启Time-wait计时器。因此，如果网络状况不理想，Time-wait状态将持续。

​	解决方案就是在套接字的可选项中更改SO_REUSEADDR的状态。适当调整该参数，可将Time-wait状态下的套接字端口号重新分配给新的套接字。SO_REUSEADDR的默认值为0（假），这就意味着无法分配Time-wait状态下的套接字端口号。因此需要将这个值改成1（真）。