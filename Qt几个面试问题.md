---
title: Qt几个面试问题
date: 2025-03-07 18:19:53
tags:
- 面试问题
- Qt学习
categories:
- Qt
cover: https://bu.dusays.com/2025/03/10/67cef42578e27.png
---

#### QT多线程





#### QML

**QML（Qt Meta-Object Language）** 是 **Qt框架** 中用于构建用户界面（UI）的声明式编程语言。它专注于界面设计与用户交互，与C++后端逻辑结合，适用于开发跨平台、动态且现代化的应用程序（如桌面、移动及嵌入式应用）。以下是其核心特性与作用：

- ECMAScript
- Qt 对象系统
- Qt Quick 标准库

用于定义和创建动态可视化界面



Qt Quick 是一种基于 Qt 的用户界面设计技术。它允许开发人员使用 QML（Qt Meta-Object Language）语言和 JavaScript 以声明式的方式创建动态的、高效的、流畅的用户界面。Qt Quick 技术不同于传统的基于部件（widget-based）的用户界面设计，其设计哲学是将界面的各个元素抽象出来，然后通过组合这些元素来实现各种不同的界面和交互效果。



#### 多线程的实现：信号和槽

api函数   成员函数、**2.2 信号槽**  **2.3静态函数**

**派生QThread类对象的方法（重写Run函数）**



启动事件循环：当一个程序调用 exec()（3456例如 QApplication::exec()）后，当前线程会进入一个阻塞状态，并开始无限循环地监听和分发事件（如鼠标点击、键盘输入、定时器触达、网络响应等）。



**使用信号与槽方式来实现多线程**

种方法存在一个局限性，只有一个run()函数能够在线程中去运行，但是当有多个函数在同一个线程中运行时，就没办法了，至少实现起来很麻烦

从 QObject 派生，在这个类中添加一个公共的成员函数(working)，函数体就是我们要子线程中执行的业务逻辑

- 在主线程中创建一个 QThread 对象，这就是子线程的对象
- 在主线程中创建工作的类对象（千万不要指定给创建的对象指定父对象）
- 将 MyWork 对象移动到创建的子线程对象中，需要调用 QObject 类提供的 moveToThread() 方法
- 启动子线程，调用 start(), 这时候线程启动了，但是移动到线程中的对象并没有工作
- 调用 MyWork 类对象的工作函数，让这个函数开始执行，这时候是在移动到的那个子线程中运行的

### 共享内存  队列

在Qt中，**共享内存（Shared Memory）** 是一种允许不同进程访问同一块内存区域的机制。这是实现进程间通信（IPC）的一种高效方式，尤其适用于需要**高频数据交换**的场景（如实时数据传输）。Qt提供了 **`QSharedMemory`** 类来简化共享内存的操作。Qt提供了 **`QSharedMemory`** 类来简化共享内存的操作

### QT 事件过滤器

在Qt中，**事件过滤器（Event Filter）** 是一种强大的机制，允许一个对象监视或拦截另一个对象的事件处理流程。通过事件过滤器，可以在事件到达目标对象前进行预处理或完全拦截，这在需要自定义事件行为或跨组件交互时非常实用。

继承 `QObject` 并重写 `eventFilter()` 方法。该方法在目标对象的事件被处理前调用。

#### **安装过滤器到目标对象**

目标对象通过 `installEventFilter()` 注册过滤器，使其事件被监控。

### QT定时器

Qt中有两种方法来使用定时器，一种是定时器事件，另一种是使用信号和槽。

- 利用对void timerEvent(QTimerEvent* e)事件的重写。
- 启动定时器 int QObject::startTimer ( int interval ) ;

开启一个定时器，返回值为int类型。他的参数interval是毫秒级别。当开启成功后会返回这个定时器的ID, 并且每隔interval 时间后会进入timerEvent 函数。直到定时器被杀死（killTimer）

- timerEvent的返回值是定时器的唯一标识。可以和e->timerId比较
- void killTimer(int id); //停止 ID 为 id 的计时器，ID 由 startTimer()函数返回

**方式：**

1. 利用定时器类QTimer
2. 创建定时器对象 QTimer *timer=new QTimer(this)
3. 启动定时器timer->start( 500） //参数：每隔n毫秒发送一个信号（timeout）
4. 用connect连接信号和槽函数（自定义槽）
5. 暂停 timer->stop

**实例：**启动label 每隔0.5秒计时

### QT和MFC底层机制



QT

**事件驱动模型**：

- **事件循环**：每个 Qt 应用程序有一个全局事件队列（`QEventLoop`），管理用户输入、定时器、网络事件等。

- 事件分发

  ：

  - Qt 将操作系统的原生事件（如鼠标点击、按键）封装为 `QEvent` 对象（如 `QMouseEvent`）。
  - 通过 `QObject::event()` 和 `QWidget::eventFilter()` 分发事件。

- 信号槽机制

  ：

  - 通过元对象系统（Meta-Object System，由 `moc` 生成）实现松耦合通信。
  - 类型安全，支持跨线程通信。

MFC

核心：基于 Win32 API，依赖 Windows 的 HWND（窗口句柄）和消息队列。
消息传递：
Windows 消息（如 WM_PAINT、WM_CLOSE）通过 消息循环（PeekMessage/GetMessage）分发给窗口过程（WndProc）。
MFC 封装了消息处理流程，通过 消息映射宏（如 ON_WM_PAINT()）将消息与成员函数绑定。

**QJSON**

Json 是一种轻量级的数据交换格式。它使得人们很容易进行阅读和编写。

QJson 是 Json的一种扩展，也就是一种简单的数据结构。

JSON 基于两种结构：1️⃣名称：值       2️⃣值的有序列表

```C++
#include <QJsonDocument>  // 读取和写入Json类
#include <QJsonValue>   // Json值有下面几种形式：空、布尔值、浮点数、字符串、数组、对象（可以是另一个Json）、未定义
#include <QJsonObject>// JSON对象是键值对的列表中，其中键是唯一的字符串，值由QJsonValue表示。
#include <QJsonArray>  // JSON 数组是一个值列表。可通过从数组中插入和删除QJsonValue 来操作列表。
#include <QJsonParseError> // 报告Json解析过程中的错误。
```

#### QT图表功能

Qt Charts是基于Qt Graphics View实现的一个图表的组件，可以用来在QT GUI程序中添加现在风格的、可交互的、以数据为中心的图表，可以用作QWidget或者 QGraphicsWidget，也可用在QML中。支持的图标类型有：折线图跟曲线图、面积图、饼图、柱状图等。

#### QT QSS 

​    QT样式表，用来实现对控件外观的自定义。

#### QT  信号只声明没有定义

​	**在 Qt 中，自定义信号（Signal）只需要在头文件中声明，无需手动在源文件中实现（定义）。** 这是因为 Qt 的元对象系统（Meta-Object System，由 `Q_OBJECT` 宏和元对象编译器 `moc` 实现）**会自动生成信号的底层代码**，负责信号的触发与槽的连接机制。

**Qt元对象系统MOC的责任**

信号的“实现”由 moc 完成
信号本质是一个“标记”，当你在代码中使用 `emit mySignal();` 时：`emit` 关键字向 moc 生成的代码下发触发指令。moc 负责管理信号与槽的连接关系，并将参数传递给槽函数。

**信号的本质**：信号是接口而非功能函数   信号的目的是触发事件通知，而非具体实现逻辑。用户只需要声明信号的格式（名称、参数类型），告知元对象系统如何传递数据。

```c++
signals:
	void healthChanged(int newHealth);  // 声明信号
	void positionUpdated(double x, double y); // 血量
```

**发射信号** `emit`：在需要的位置调用`emit`，时间的逻辑处理由接收该信号的槽函数实现

```c++
void Aladdin::takeDamage(int damage)
{
    m_health -= damage;
    emit healthChanged(m_health); // 触发信号，通知外部当前的血量
}
```

对比槽函数（需要手动实现）

#### 信号为何必须是 `void` 类型？

信号的作用是触发事件通知，而不是返回值。若想传递数据，应通过参数传递，而非返回值。

#### **信号是否可重载？**

是的，允许通过参数类型或数量重载信号。连接时需使用 `QOverload` 或 `static_cast` 明确指定重载版本。

#### **为何有时无法触发信号？**

- 未正确连接信号与槽（`connect` 错误）。
- 未添加 `Q_OBJECT` 宏或未重新生成 moc。
- 发射信号的对象被提前销毁。

#### QT自定义控件的使用流程

备注下
