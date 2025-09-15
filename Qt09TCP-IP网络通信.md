---
title: Qt09TCP-IP网络通信
date: 2025-03-13 10:10:23
tags:
- 网络编程
- Qt学习
categories:
- Qt
cover: https://bu.dusays.com/2025/03/13/67d2dbab1ed00.png
---



TCP/IP通信（即SOCKET通信）是通过网线将**服务器Server端**和**客户机Client端**进行连接，在遵循ISO/OSI模型的四层层级构架的基础上通过TCP/IP协议建立的通讯。控制器可以设置为服务器端或客户端。

关于TCP/IP协议可详看：[TCP/IP协议详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/33889997)

 总的来说，TCP/IP通讯有两个部分：

- **客户端**和**服务器**
- **QTcpServer（监听套接字）**和**QTcpSocket（通讯套接字）**

监听套接字，顾名思义，监听关于各种通讯的状态，一旦进行通讯，监听套接字会启动通讯套接字，进行通讯

客户端使用connectToHost函数主动连接服务器后，服务器会触发newConnectio这个槽函数，并进行取出QTcpServer（监听套接字），将相关内容取出并赋给QTcpSocket（通讯套接字）。
客户端向服务器发送数据，触发readyRead（），进行处理，彼此传递时，原理都是这样的。

**对双方来说都起作用的部分：**

1. 一旦建立连接，就会触发connected，服务器特殊一点，触发的是newConnectio
2. 互传数据也是一样的，一旦接受到，就会触发readyread

服务器中，需要监听套接字以及通讯套接字，监听套接字用于监听客户端是否给服务器发送请求

本篇博文做了初步的学习与尝试，编写了一个客户端和服务器基于窗口通信以及文件传输的小例程。

### <font color="FF0000">一，客户端</font>

客户端的代码比服务器稍简单，总的来说，使用QT中的**QTcpSocket类**与服务器进行通信只需要以下5步：

**（1）创建QTcpSocket套接字对象**

```
    socket = new QTcpSocket(this);
```

**（2）使用这个对象连接服务器**

```
    QString ip = ui.lineEdit_ip->text();//获取ip
    int port = ui.lineEdit_2->text().toInt();//获取端口数据
    socket->connectToHost(ip, port);
```

**（3）使用write函数向服务器发送数据**

```
    QByteArray data = ui.lineEdit_3->text().toUtf8();//获取lineEdit控件中的数据并发送给服务器
    socket->write(data);
```

 **（4）当socket接收缓冲区有新数据到来时，会发出readRead()信号，因此为该信号添加槽函数以读取数据**

```
 connect(socket, &QTcpSocket::readyRead, this, &QTcpClinet::ReadData);
void QTcpClinet::ReadData()
{
    QByteArray buf = socket->readAll();
    ui.textEdit->append(buf);
}
```

**（5）断开与服务器的连接（关于close()和disconnectFromHost()的区别，可以按F1看帮助）**

```
socket->disconnectFromHost();
```

**客户端例程：（新建一个qt项目QTcpClinet（客户机））**

- **ui界面**

![](https://bu.dusays.com/2025/03/13/67d2dbe2f1a93.png)

本地回路ip：127.0.0.1 可以连接到本地ip（电脑内部循环的ip）

如果要和局域网其他ip连接 -> 在运行（win+R）+cmd+ipconfig ->ipv4地址 查看本机ip

- **QTcpClinet.h**

```
#include <QtWidgets/QWidget>
#include "ui_QTcpClinet.h"
#include"QTcpSocket.h"
#pragma execution_character_set("utf-8")
class QTcpClinet : public QWidget
{
    Q_OBJECT

public:
    QTcpClinet(QWidget *parent = Q_NULLPTR);
    ~QTcpClinet();
public slots:
    void on_btn_connect_clicked();
    void ReadData();
    void on_btn_push_clicked();
private:
    Ui::QTcpClinetClass ui;
    QTcpSocket* socket;//创建socket指针
};
```

- **QTcpClinet.cpp**

```c++
#include "QTcpClinet.h"

QTcpClinet::QTcpClinet(QWidget *parent)
    : QWidget(parent)
{
    ui.setupUi(this);
    socket = new QTcpSocket(this);
}

QTcpClinet::~QTcpClinet()
{
    delete this->socket;//回收内存
}

void QTcpClinet::on_btn_connect_clicked()
{
  if (ui.btn_connect->text()==tr("连接服务器"))
  {
    QString ip = ui.lineEdit_ip->text();//获取ip
    int port = ui.lineEdit_2->text().toInt();//获取端口数据
    //取消已有的连接
    socket->abort();
    //连接服务器
    socket->connectToHost(ip, port);
    bool isconnect = socket->waitForConnected();//等待直到连接成功
    //如果连接成功
    if (isconnect)
    {
        ui.textEdit->append("The connection was successful!!");
        ui.btn_push->setEnabled(true);//按钮使能
        //修改按键文字
        ui.btn_connect->setText("断开服务器连接");
        //接收缓冲区（服务器）信息
        connect(socket, &QTcpSocket::readyRead, this, &QTcpClinet::ReadData);
    }
    else
    {
        ui.textEdit->append("The connection falied!!");
    }
  }
  else
  {
      //断开连接
      socket->disconnectFromHost();
      ui.btn_connect->setText("连接服务器");
      ui.btn_push->setEnabled(false);//关闭发送按钮使能
  }

}

//接收缓冲区信息函数
void QTcpClinet::ReadData()
{
    QByteArray buf = socket->readAll();
    ui.textEdit->append(buf);
}
//发送按钮事件
void QTcpClinet::on_btn_push_clicked()
{
    QByteArray data = ui.lineEdit_3->text().toUtf8();//获取lineEdit控件中的数据并发送给服务器
    socket->write(data);
    //判断是否写入成功
    bool iswrite = socket->waitForBytesWritten();
    if (iswrite)
    {
        //写入成功
    }
    else
    {
        //没有写入成功
    }
}
```

### <font color="FF0000">二，服务器（需要一直运行哦）</font>

服务器除了使用到了**QTcpSocket类**，还需要用到**QTcpSever类**。即便如此，也只是比客户端复杂一点点，用到了6个步骤：

**（1）创建QTcpSever对象**

```
    server = new QTcpServer(this);
```

**（2）侦听一个端口，使得客户端可以使用这个端口访问服务器**

```
    server->listen(QHostAddress::Any, 6677);//监听所有ip和6677端口
```

**（3）当服务器被客户端访问时，会发出newConnection()信号，因此为该信号添加槽函数，并用一个QTcpSocket对象接受客户端访问**

```
connect(server, &QTcpServer::newConnection, this, &TcpServer::ClientConnect);
void TcpServer::ClientConnect()
{
    //解析所有客户连接
    while (server->hasPendingConnections())
    {
        //连接上后通过socket（QTcpSocket对象）获取连接信息
        socket = server->nextPendingConnection();
        QString str = QString("[ip:%1,port:%2]").arg(socket->peerAddress().toString()).arg(socket->peerPort());//监听客户端是否有消息发送
        connect(socket, &QTcpSocket::readyRead, this, &TcpServer::ReadData1);
    }
}
```

**（4）使用socket的write函数向客户端发送数据**

```
socket->write(data);
```

**（5）当socket接收缓冲区有新数据到来时，会发出readRead()信号，因此为该信号添加槽函数以读取数据**

```
//监听客户端是否有消息发送
connect(socket, &QTcpSocket::readyRead, this, &TcpServer::ReadData1);
//获取客户端向服务器发送的信息
void TcpServer::ReadData1()
{
    QByteArray buf = socket->readAll();//readAll最多接收65532的数据
    QString str = QString("[ip:%1,port:%2]").arg(socket->peerAddress().toString()).arg(socket->peerPort());
    ui.textEdit_server->append(str +QString(buf));
    //socket->write("ok");//服务器接收到信息后返回一个ok
}
```

**（6）取消侦听**

```
server->close();
```

 **服务器例程：（添加一个新的qt项目TcpServer（服务器））**

- **ui界面**

![](https://bu.dusays.com/2025/03/13/67d2dc19a7d8f.png)

- **TcpServer.h**

```
#include <QtWidgets/QWidget>
#include"ui_TcpServer.h"
#include"qtcpserver.h"
#include"qtcpsocket.h"

class TcpServer : public QWidget
{
    Q_OBJECT

public:
    TcpServer(QWidget *parent = Q_NULLPTR);
    ~TcpServer();
public slots:
    void on_btn_server_clicked();
    void on_btn_listen_clicked();
private:
    Ui::TcpServerClass ui;
    QTcpServer* server;
    QTcpSocket* socket;//一个客户端对应一个socket
    void ClientConnect();
    void ReadData1();
    
};
```

- **TcpServer.cpp**

```
#include "TcpServer.h"
#include"qstring.h"
#include"qdebug.h"
#pragma execution_character_set("utf-8")
TcpServer::TcpServer(QWidget *parent)
    : QWidget(parent)
{
    ui.setupUi(this);
    server = new QTcpServer(this);
   //客户机连接信号槽
    connect(server, &QTcpServer::newConnection, this, &TcpServer::ClientConnect);
}

TcpServer::~TcpServer()
{
    server->close();
    server->deleteLater();
}

void TcpServer::on_btn_listen_clicked()
{
    if (ui.btn_listen->text()=="侦听")
    {
        //从输入框获取端口号
        int port = ui.lineEdit_port->text().toInt();
        //侦听指定端口的所有ip
        if (!server->listen(QHostAddress::Any, port))
        {
            //若出错，则输出错误信息
            qDebug() << server->errorString();
            return;
        }
        //修改按键文字
        ui.btn_listen->setText("取消侦听");    
    }
    else
    {
        socket->abort();
        //取消侦听
        server->close();
        //修改按键文字
        ui.btn_listen->setText("侦听");
    }
}

void TcpServer::ClientConnect()
{
    //解析所有客户连接
    while (server->hasPendingConnections())
    {
        //连接上后通过socket获取连接信息
        socket = server->nextPendingConnection();
        QString str = QString("[ip:%1,port:%2]").arg(socket->peerAddress().toString()).arg(socket->peerPort());
        //提示连接成功
        ui.textEdit_server->append(str+"Connect to the server");
        //复选框选项为连接服务器的ip
        ui.comboBox->addItem(str);
        //将socket地址放入combobox属性内
        //ui.comboBox->setItemData(ui.comboBox->count()-1, QVariant((int)socket));
        //监听客户端是否有消息发送
        connect(socket, &QTcpSocket::readyRead, this, &TcpServer::ReadData1);
    }
}

//获取客户端向服务器发送的信息
void TcpServer::ReadData1()
{
    QByteArray buf = socket->readAll();//readAll最多接收65532的数据
    QString str = QString("[ip:%1,port:%2]").arg(socket->peerAddress().toString()).arg(socket->peerPort());
    ui.textEdit_server->append(str +QString(buf));
}

//服务器向客户端发送信息
void TcpServer::on_btn_server_clicked()
{
  if(ui.comboBox->count()== 0)return;
  //QTcpSocket* skt=  (QTcpSocket*)ui.comboBox->itemData(ui.comboBox->currentIndex()).value<int>();
  socket->write(ui.lineEdit1->text().toUtf8());
}
```

**注意**：write中需要写入char类型的元素或QByteArray类型的元素

![](https://bu.dusays.com/2025/03/13/67d2dc2f4307e.png)

**效果展示：**

![](https://bu.dusays.com/2025/03/13/67d2dc5b93345.gif)

### <font color="FF0000">三，TCP/IP文件传输</font>

上文实现了消息的传输，由于**socket->readAll();（readAll最多接收65532的数据）**，因此对于大文件的传输用此方法是不可取的。

**TCP/IP文件传输的思路：**

1. 客户端和服务器连接
2. 客户端选择文件，并发送文件给服务器（发送的是文件的帧头，格式：文件名&大小）
3. 服务器触发readyRead，然后解析文件帧头（获取文件名和大小），并返回客户端一个ok消息
4. 客户端触发readyRead，然后发送文件数据，通过progressBar显示进度
5. 服务器再次触发readyRead，接收文件数据，并保存（通过ishead判断接收的是文件帧头还是文件数据）

**代码实现：**

新建服务器项目（TcpServer）

- **TcpServer.h**

```
#pragma once

#include <QtWidgets/QWidget>
#include "ui_TcpServer.h"
#include"qtcpserver.h"
#include"qtcpsocket.h"
#pragma execution_character_set("utf-8")
class TcpServer : public QWidget
{
    Q_OBJECT

public:
    TcpServer(QWidget *parent = Q_NULLPTR);
    void hasConnect();
private:
    Ui::TcpServerClass ui;
    QTcpServer* server;
    QTcpSocket* socket;
    bool ishead;
    QString fileName;
    int fileSize;//接收文件的总大小
    int recvSize;//当前接收文件的大小
    QByteArray filebuf;//当前接收的文件数据
};
```

- **TcpServer.cpp**

```
#include "TcpServer.h"
#include"qfile.h"
TcpServer::TcpServer(QWidget *parent)
    : QWidget(parent)
{
    ishead = true;
    ui.setupUi(this);
    server = new QTcpServer(this);
    //监听1122端口的ip
    server->listen(QHostAddress::Any, 1122);
    //如果有用户连接触发槽函数
    connect(server, &QTcpServer::newConnection, this, &TcpServer::hasConnect);
}

void TcpServer::hasConnect()
{
    while (server->hasPendingConnections()>0)//判断当前连接了多少人
    {
        //用socket和我们的客户端连接，一个客户端对应一个套接字socket
        socket = server->nextPendingConnection();
        //服务器界面上输出客户端信息
        ui.textEdit->append(QString("%1：新用户连接").arg(socket->peerPort()));
        //如果客户端发送信息过来了，触发匿名函数
        connect(socket, &QTcpSocket::readyRead, [=]() {
            QByteArray buf = socket->readAll();
            //用一个标志位ishead判断是头还是数据位
            if (ishead)
            {
                //如果是头，解析头（文件名，文件大小）
                QString str = QString(buf);
                ui.textEdit->append(str);
                QStringList strlist = str.split("&");
                fileName = strlist.at(0);//解析帧头文件名
                fileSize = strlist.at(1).toInt();//解析帧头文件大小
                ishead = false;//下次接收到的文件就是我们的数据
                recvSize = 0;
                filebuf.clear();
                socket->write("ok");
            }
            else
            {
                //根据文件名和文件大小接收和保存文件
                filebuf.append(buf);
                recvSize += buf.size();//每接收一次文件，当前文件大小+1
                //当接收文件大小等于总文件大小，即文件数据接收完毕
                if (recvSize>=fileSize)
                {
                    //保存文件
                    QFile file(ui.lineEdit->text() + fileName);
                    file.open(QIODevice::WriteOnly);
                    file.write(filebuf);
                    file.close();
                    ishead = true;
                }
            }
            });
    }
}
```

新建客户端项目（QTcpClient）

- **QTcpClient.h**

```
#include <QtWidgets/QWidget>
#include"ui_QTcpClient.h"
#include"qtcpsocket.h"
#pragma execution_character_set("utf-8")
class QTcpClient : public QWidget
{
    Q_OBJECT

public:
    QTcpClient(QWidget *parent = Q_NULLPTR);
public slots:
    void on_btn_connect_clicked();
    void on_btn_choose_clicked();
    void on_btn_open_clicked();
    
private:
    Ui::QTcpClientClass ui;
    QTcpSocket* socket;
};
```

- **QTcpClient.cpp**

```
#include "QTcpClient.h"
#include"qfiledialog.h"
#include"qfileinfo.h"
QTcpClient::QTcpClient(QWidget *parent)
    : QWidget(parent)
{
    ui.setupUi(this);
    socket = new QTcpSocket(this);
    
}
void QTcpClient::on_btn_connect_clicked()
{
    QString ip = ui.lineEdit_ip->text();//获取ip
    int port = ui.lineEdit_port->text().toInt();//获取端口数据
    socket->connectToHost(ip, port);//连接服务器
    //等待连接成功
    if (socket->waitForConnected())
    {
        ui.textEdit->append("<font color='green'>连接服务器成功！</font>");    
        ui.btn_open->setEnabled(true);
        
        //如果服务器发送信息到客户端，触发匿名函数
        connect(socket, &QTcpSocket::readyRead, [=]() {
            //读取服务器发送的信息（即缓冲区信息）
            QByteArray buf = socket->readAll();
            if (buf=="ok")
            {
                QFile file = (ui.label_path->text());
                if (!file.open(QIODevice::ReadWrite))
                {
                    //读取文件失败
                    return;
                }
                qint64 currentlen = 0;//当前已经发送的大小
                qint64 allLength = file.size();//总文件大小
                do
                {
                    char data[1024];
                    qint64 msize = file.read(data, 1024);//读文件放入打他数组中，返回读取到的大小
                    socket->write(data, msize);//把读取到的data数据发送给服务器
                    currentlen += msize;//实时获取当前发送的文件大小
                    ui.progressBar->setValue(currentlen *100 / allLength);//更新界面进度条
                } while (currentlen < allLength);//当发送文件等于文件大小时，发送完毕，循环结束
            }
            });

    }
    else
    {
        ui.textEdit->append("<font color='red'>连接服务器失败！</font>");
    }
}
//选择文件事件
void QTcpClient::on_btn_choose_clicked()
{
    QString path = QFileDialog::getOpenFileName(this, "打开文件", "", "(*.*)");
    ui.label_path->setText(path);
}
//发送文件事件
void QTcpClient::on_btn_open_clicked()
{
    QFileInfo info(ui.label_path->text());
    //用QFileInfo：：fileName，size获取文件名和大小 格式：文件名&大小
    //服务器用该格式解析文件名和大小
    QString head = QString("%1&%2").arg(info.fileName()).arg(info.size());
    //将该格式发送给服务器 toUtf8：QString转QByteArray或char类型
    socket->write(head.toUtf8()); 
}
```

**效果展示：**

![](https://bu.dusays.com/2025/03/13/67d2dc9a83bfe.gif)

<br>

<br>

<br>

<span style="color: red; font-size: 20px;">转载自：[唯有自己强大](https://www.cnblogs.com/xyf327)          如有侵权，在下方评论  立刻删除。</span>
