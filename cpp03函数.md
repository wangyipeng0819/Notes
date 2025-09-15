---
title: cpp03函数
date: 2024-12-11 13:56:28
categories:
- Cpp
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686c9d1fd91fc.png
---

**函数参数：指针参数**

- 如果需要传递大型数据结构，可以传递指针而不是整个数据结构，避免复制数据。

```c++
#include <iostream>

int Add(int* x, int* y)
{
    (*x) *= 100;
    (*y) *= 10;
    return (*x) + (*y);
}
int main()
{
    int x = 2, y = 1;
    int c = Add(&x, &y);
    //x 和  y 的值都改了  指针改的是内存中的数值
    std::cout << "c=" << c << "x=" << x << "y=" << y;
}
```

应用

```c++
#include <iostream>

struct Role
{
    int Hp;
    int Mp;
};

/*
int Exp(Role r1)
{
    return r1.Hp + r1.Mp;
}*/

int Exp(Role* r1)
{
    return r1->Hp + r1->Mp;
}

int main()
{
    Role r1{500,200};
    //c = Exp(r1);
    c = Exp(&r1);  // 效率大大提升
    std::cout << c;
}
```

避免修随意修改值，使用常量指针限定只读操作

游戏小程序

```c++
#include <iostream>

struct Role
{
    int Hp;
    int Mp;
    int damage;
};

bool Act(const Role* Acter, Role* beActer)
{
    beActer->Hp -= Acter->damage;
    return beActer->Hp <= 0;
}

int main()
{
    Role User{ 1000,1500,2222220 };
    Role Monster{ 1500,100,100 };

    if (Act(&Monster, &User))
        std::cout << "角色死亡！！";
    else if (Act(&User, &Monster)) std::cout << "怪物死亡！获得屠龙宝刀！";
    
}
```

**数组参数**

数组参数两种方式(等价，汇编代码也一样)

```c++
void Sort(int ary[], unsigned count)
{
    for (int i = 0; i < count; i ++ )
        std::cout << ary[i];
}

void Sort(int* ary. unsigned count)
{
    for (int i = 0; i < count; i ++ )
        std::cout << ary[i];
}
```



```c++
// 下面这俩都不能给到预期的结果
// 指针，能得到地址，但是得不到总大小，后面还得传入个数组大小才行  加个unsigned 
void Sort(int ary[])
{
    std::cout << sizeof(ary);
}

void Sort(int ary[])
{
    std::cout << sizeof(ary);
}
```

二维数组

```c++
// 必须指定后面的
void Sort(int ary[][2] unsigned count)
{
    
}

int main()
{
    int a[3][2]{{1,2},{3,4},{5,6}};
    Sort(a,3);
}
```

作业，改造Sort函数



**引用参数**

可以像指针一样，设定只能读不能写

```c++
struct Role
{
    int hp;
    int mp;
    int damage;
};

bool Act(const Role& Acter, Role& beAct)
{
    beAct.hp -= Acter.damage;
    return beAct.hp < 0;
}

int main()
{
    Role user{200,300,850};
    Role monster{800,300,50};
    if (Act(user,monster)) std::cout << "怪物死亡，获得。。。。";
}
```

指针和引用的区别

```c++
bool Act(Role& Acter, Role& beAct)
{
    return true;
}

bool Act(Role* Acter, Role* beAct)
{
    return true;
}

Act(nullptr, nullptr);
// 区别：指针可以传入nullptr空指针, 而引用不可以。
```

*&

```c++
// Role类型的指针的引用
bool Act(const Role& Acter, Role* beAct)
{
    beAct->hp -= Acter.damage;
    bool bEnd = beAct->hp<0;
    
    // 让beAct指向Acter
    beAct = (Role*)&Acter;
    return beAct->hp < 0;
}

int main()
{
    Role user{"奥特曼",200,300,500};
    Role monster{"小怪兽",800,300,50};
    
    Role* pRole = &monster;
    
    // 输出的name还是小怪兽，因为在函数作用内beAct指向了Acter,但是pRole没有指向Acter
    if (Act(user, pRole)) std::cout << pRole->Name << "怪物死亡。。。";
}


// 解决办法
bool Act(const Role& Acter, Role*& beAct);
// 这样 pRole也指向Acter了
```

完整的代码如下

```c++
#include <iostream>

struct Role
{
    char Name[0x20];
    int Hp;
    int Mp;
    int damage;
}*PROLE;

bool Act(const Role* Acter, PROLE& beActer)
{
    beAct->hp -= Acter->damage;
    bool bEnd = beAct->hp<0;
    
    
    beAct = (Role*)&Acter;
    return beAct->hp < 0;
}

int main()
{
    Role user{"奥特曼",200,300,500};
    Role monster{"小怪兽",800,300,50};
    
    PROLE pRole = &monster;
    
    if (Act(user, pRole)) std::cout << pRole->Name << "怪物死亡。。。";
}
```

**按值传递 vs 按引用传递**

```c++
bool Act(const Role* Acter, PROLE beActer)
{
    // beActer 是 PROLE 的拷贝，修改它不会影响原始的 PROLE
    beActer = (Role*)&Acter;
    return beActer->Hp < 0;
}
//在这种情况下，beActer 是 PROLE 的拷贝，修改 beActer 后不会影响 pRole。pRole 依然指向原始的 monster。

bool Act(const Role* Acter, PROLE& beActer)
{
    // beActer 是 PROLE 的引用，修改它会影响原始的 PROLE
    beActer = (Role*)&Acter;
    return beActer->Hp < 0;
}
```

```C++
int Add(int* x, int* y)
{
    (*x) *= 100;  // 修改 x 指向的值
    (*y) *= 10;   // 修改 y 指向的值
    return (*x) + (*y);  // 返回修改后的值之和
}
```

上面这个例子也是值传递，但因为通过指针解引用(*x 和 *y)修改了指针所指向的变量的值，所以下面的代码实际上是修改了原始变量的内容。所以，尽管是值传递，但是通过指针传递数据导致的效果是修改了原始数据。

- **指针本身是值传递**（函数内的指针 `x` 和 `y` 是原始指针的副本）。
- **指针所指向的内容是通过解引用修改的**（因此修改了原始数据）。

```c++
#include<iostream>

void swap(int& a, int& b)
{
	int tmp = ary[i];
    ary[i] = ary[i - 1];
    ary[i - 1] = tmp;
}

void Sort(int ary[], unsigned count, bool BigSort)
{
    for (int i = 1; i < count; i ++ )
        for (int i = 1; i < count; i ++ )
        {
            bool bcase = BigSort?ary[i] > ary[i - 1] : ary[i] < ary[i - 1];
            
            if (bcase) swap(ary[i], ary[i - 1]);
        }
}
```



**默认实参**

用户不指定值的时候，就使用默认的值

```c++
void Sort(int ary[], unsigned count, bool BigSort = true)
```

默认参数只能放在最后

```c++
int Add(int a = 100, b = 200); // 正确， 相当于Add(100,200);
int Add(int a = 100, int b);  // 错误
int Add(int a, int b = 100, int c = 250); // 正确
int Add(int a, int& b = 100);   //error   引用的本质是一个指针，指定100错了
```

**不定量参数**

main: 处理命令行选项 ，有时候需要给main传递实参，一种常见的情况是用户通过设置一组选项来确定函数所要执行的操作。假定main函数位于可执行文件prog之内，向程序传递下面的选项

`prog -d -o ofile data0`

这些命令行通过两个可选的形参传递给main函数

```c++
int main(int argc, cahr* argv[]) {...}
```

第二个形参是一个数组，它的元素是指向C风格字符串的指针：第一个形参argc表示数组中字符串的数量。因为第二个形参是数组，所以main函数也可以定义成：

```c++
int main()(int argc, char **argv) {...}
```

其中 argv 指向char* 。。

当实参传给main函数之后，argv的第一个元素指向程序的名字或者一个空字符串，接下来的元素一次传递命令行提供的实参。最后一个指针之后的元素保证为0 。

`argv[0] = "prog";`

`argv[1] = "-d";`

`argv[2] = "-o";`

`argv[3] = "ofile";`

`argv[4] = "data0";`

`argv[5] = 0;`

例题：输出程序名字和路径

```c++
#include <iostream>
#include <cstring>

int main(int argc, char* argv[]) {
    // argv[0] 是程序完整的路径（包括文件名）
    std::cout << "程序名字" << argv[0] << std::endl;

    const char* programName = strrchr(argv[0], '\\');
    // strrchr 是一个 字符串函数，用于查找路径中最后一次出现反斜杠\位置。
    // 如果找到反斜杠，返回指向它的指针   否则返回nullptr
    if (programName != nullptr)
    {
        programName++;
        std::cout << "程序名称：" << programName << std::endl;
    }
    else
        std::cout << "程序名称：" << argv[0] << std::endl;

    return 0;
}
```

```c++
// 使用刚学的find方法
#include <iostream>
#include <string>

int main(int argc, char* argv[])
{
    if(argc > 0) std::string programPath = argv[0];
    
    std::cout << "程序路径：" << programPath << std::endl;
    
    std::size_t lastSlash = programPath.rfind('\\');
    if (lastSlash == std::string::npos) lastSlash = programPath.rfind('/');
    
    if (lastSlash != std::string::npos)
    {
        std::string programName = programPath.substr(lastSlash + 1);
        std::cout << "程序名称： " << programName << std::endl;
    }
    else std::cout << "程序名称： " << programPath << std::endl;
    
    return 0;
}
```

**可变形参的函数**

使用 `...` 语法（C 风格的可变参数）

```c++
#include <iostream>
#include <cstdarg>  // 头文件支持可变参数

void printNumbers(int count, ...) {
    va_list args;
    // 可以使用char* args{}; 替代
    // 告诉args有多少个参数
    va_start(args, count);  // 初始化 args 以访问可变参数

    // 打印传入的所有参数
    for (int i = 0; i < count; ++i) {
        // 要读的指针  和  参数的类型
        // 每调用一次，就能获取下一个参数
        int num = va_arg(args, int);  // 获取下一个参数，类型为 int
        std::cout << num << " ";
    }

    va_end(args);  // 清理  释放args
    
    std::cout << std::endl;
}

int main() {
    printNumbers(3, 10, 20, 30);  // 打印 10 20 30
    printNumbers(5, 1, 2, 3, 4, 5);  // 打印 1 2 3 4 5
    return 0;
}
```

使用 `std::initializer_list`(C++11之后)

```c++
#include <iostream>
#include <initializer_list>

void printNumbers(std::initializer_list<int> nums) {
    for (int num : nums) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
}

int main() {
    printNumbers({10, 20, 30});  // 打印 10 20 30
    printNumbers({1, 2, 3, 4, 5});  // 打印 1 2 3 4 5
    return 0;
}
```

使用 `std::vector` 或者其他容器

```c++
#include <iostream>
#include <vector>

void printNumbers(const std::vector<int>& nums) {
    for (int num : nums) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
}

int main() {
    printNumbers({10, 20, 30});  // 打印 10 20 30
    printNumbers({1, 2, 3, 4, 5});  // 打印 1 2 3 4 5
    return 0;
}
```

使用 `std::function`  和 Lambda 表达式（高级用法）

```c++
#include <iostream>
#include <functional>

void applyFunction(std::function<void()> func) {
    func();
}

int main() {
    // Lambda 接受不定数量参数
    applyFunction([]() {
        std::cout << "Lambda with no arguments!" << std::endl;
    });
    return 0;
}
```

**一个小作业**

```c++
#include <iostream>
#include <string>

const char* ReadRef(const char* ref, const char* cmds)
{
    for (int i = 0; cmds[i]; i ++ )
    {
        if (cmds[i] == ref[0])
        {
            bool found = true;
            int  x;
            for (x = 0; ref[x]; x ++ )
            {
                if (ref[x] != cmds[i + x])
                {
                    found = false;
                    break;
                }
            }
            if (found) return &cmds[i + x];
        }
    }
    return nullptr;
}

int main(int argc, char* argv[]) {
    
    const char* id = nullptr;
    const char* pass = nullptr;
    const char* country = nullptr;

    
    for (int i = 1; i < argc; i++) {
        if (!id) id = ReadRef("id:", argv[i]);
        if (!pass) pass = ReadRef("pass:", argv[i]);
        if (!country) country = ReadRef("country:", argv[i]);
    }

    
    if (id && pass && country) {
        std::cout << "注册成功！\n";
        std::cout << "账号：" << id << "\n";
        std::cout << "密码：" << pass << "\n";
        std::cout << "国家：" << country << "\n";
    } else {
        std::cout << "参数不足！请按以下格式调用程序：\n";
        std::cout << "./program id:<your_id> pass:<your_pass> country:<your_country>\n";
    }

    return 0;
}
```

