---
title: cpp06不同语言联合编程
date: 2024-12-11 13:58:40
categories:
- Cpp
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686c9d1fd91fc.png
---

​                                                              **此章节干货较少，笔记也比较混乱，理解为主**

**static 和 inline**

静态变量

```c++
#include <iostream>
int ave(int a, int b)
{
    static int count;    // 初始化为0   生命周期跟全局变量一样（局部变量）  （不会放在栈上面）
    std::cout << count ++ << std::endl;
    return (a + b) / 2;
}
// 静态变量（无论是局部的还是全局的）通常被存储在 数据段（data segment）中，而不是栈或堆中。静态变量的内存分配在程序启动时就已完成，并且在整个程序运行期间都保持有效，直到程序退出。
```

内联函数

**内联函数**（Inline Function）是 C++ 中的一种函数优化技术，它的作用是让编译器将函数的代码直接嵌入到调用该函数的地方，而不是通过函数调用的机制进行跳转。这种优化有助于减少函数调用的开销（尤其是对于短小的函数），从而提高程序的执行效率。

递归函数不能用，本身就需要栈空间。

可以用一个inline声明一个内联函数   内联函数会建议编译器把这个函数处理成内联代码以提升性能

始终是建议，具体编译器是否采纳，由编译器决定。。。

```c++
inline int add(int a, int b)
{
    return a + b;
}

int main()
{
    int a = 1 + 2;   // 加上inline之后，直接 把相应的代码放在这里了  不调用函数了   直接a + b
}
```

**从编译器的角度理解定义和声明**

函数声明（也称为函数原型）是告诉编译器函数的名称、返回类型和参数类型的声明，但不包含函数体。它主要用于在调用函数之前告诉编译器函数的存在及其接口。

```c++
#include <iostream>

int ave(int a, int b);  //  

int main()
{
    int c = ave(1, 2);
}

int ave(int a, int b)
{
    return a + b;
}
```

函数定义是函数的实际实现，包含了函数的返回类型、参数以及函数体。函数定义包括了函数的逻辑实现，并且会占用内存资源。

- **函数声明**只提供函数的接口信息，不包含实现，通常用于让编译器在调用函数之前知道函数的存在。
- **函数定义**提供了函数的完整实现，包含了函数的所有细节。

**变量和函数的不同（声明  定义）**

`extern int x;`  // 声明变量 x     声明，默认是extern

`extern` 关键字用于声明变量而不定义它。这样可以在多个文件之间共享变量。例如，当你需要在多个源文件中使用同一个全局变量时，可以使用 `extern` 来声明该变量。

x86  32位windows操作系统下内存分配







**头文件和源文件**

头文件用于声明类、函数、变量、常量等信息，但不包含具体的实现

头文件只有在调用的时候才被编译，主要是用来写声明的，也可以写定义。

函数只要是在一个工程里就不能重复定义（重载除外）

情况 1️⃣ 静态函数，只在所在的cpp文件中有效，加上 `static` 关键字，那么不同的cpp文件中的静态函数都有各自的空间，如果不加，那么整个工程里面的相同函数名的函数都用的同一块空间。



情况 2️⃣ 内联函数  也是一样的



如果是变量的话，要用 `extern` 关键字，只声明不定义，那写在头文件里没问题。

静态变量也是在各自的源文件里有效。`static`关键字





重复调用的情况不会出现问题，原因是 `#pragma once`  但有的编译器不支持这个命令了 

**头文件保护机制**（Include Guard）。头文件保护机制的目的是防止头文件被多次包含，避免重复定义和潜在的编译错误。

```c++
#ifndef _HEMATH_
#define _HEMATH_


#endif
```

1. **`#ifndef _HEMATH_`**：
   - `#ifndef` 是 "if not defined" 的缩写，意思是“如果未定义”。
   - 它检查是否已定义名为 `_HEMATH_` 的宏。如果没有定义，那么接下来的代码会被编译；如果已经定义，则跳过。
   - 这个宏通常是头文件的唯一标识符，用来确保头文件只被包含一次。
2. **`#define _HEMATH_`**：
   - `#define` 用于定义宏 `_HEMATH_`。一旦定义了这个宏，后续的 `#ifndef` 就会失败，避免重复包含该头文件。
   - 宏 `_HEMATH_` 在第一次包含该头文件时定义，以后任何包含此头文件的地方都会跳过其中的内容。
3. **`#endif`**：
   - 这是 `#ifndef` 的结束标志，表示条件编译的结束。它标志着条件保护代码块的结束。

```c++
// 示例：`hemath.h`
#ifndef _HEMATH_
#define _HEMATH_

int add(int a, int b);
int subtract(int a, int b);

#endif
```

```c++
// 在源文件中使用
#include "hemath.h" // 第一次包含，宏 _HEMATH_ 没有定义，会进入条件编译区块
#include "hemath.h" // 第二次包含，宏 _HEMATH_ 已定义，直接跳过头文件内容
```

这样，第二次 `#include` 时，头文件的内容就不会被重新包含，从而避了重复定义的问题。

- 宏名字（`_HEMATH_`）可以自定义，但最好使用独特且具有全局唯一性的名字，通常使用项目名或文件名来避免与其他库的宏冲突。
- 另外，也可以考虑使用 `#pragma once` 来替代传统的 `#ifndef` 方式，这样做更简洁，并且现代编译器通常会优化它。



C语言不支持函数重载，所以有多个源文件包含相同的函数，C语言把函数名加_  但是C++有函数重载不能这样处理，.c 文件和 .cpp 文件若有相同的函数，也不会出错，因为编译器处理之后 的名字不一样

1️⃣ 定义在C语言里面的函数，不能在C++中直接访问 （**要使用extern关键字才行**）

```C++
extern "C" int ave();  // 加上这个就能在cpp文件里面运行c中的函数了

// 写法1️⃣
extern "C"
{
    int ave();
    int pve();
}

// 写法2️⃣   直接把头文件 用extern "C"
extern "C"
{
    #include "e.h"  // 头文件里面的全是C的风格
}
```

通过这种手段，C++可以调用C语言代码

2️⃣**在C语言中调用C++的函数**

```c++
#include <iostream>
extern "C" int xve() {} // 把C++ 中的函数定义成C风格即可
```

为了让C语言文件知道C++中定义的函数，在C++的头文件中使用 `extern "C"` 来声明这些函数。这样，C语言和C++都可以包含这个头文件。



```c++
// example.h (C++头文件)

#ifdef __cplusplus
extern "C" {
#endif

int add(int a, int b);  // 声明C++中的add函数

#ifdef __cplusplus
}
#endif
```

3️⃣ 在C语言代码中，使用extern "C" 声明该函数。

```c
// example.c (C文件)

#include <stdio.h>

extern int add(int a, int b);  // 声明C++中的add函数

int main() {
    int result = add(2, 3);  // 调用C++中的函数
    printf("Result: %d\n", result);
    return 0;
}
```

**C 和 c++ 源文件混用的问题**

如果.c 文件 和 .cpp 文件重名（都叫a），那编译的时候，会有俩a.obj，一个文件下目录下有俩a.obj  那么就会出问题，尽量不要让C++和C文件重名

**创建自己的SDK**

```c++
#include <iostream>
#include <edoyun.h>

// 方法一  配好包含目录和库目录之后，写下面的指令即可
//#pragma comment(lib,"edoyun.lib")
// 方法二  链接器->输入->附加依赖项
int main()
{
	std::cout << edoyun::GetVersion();
	std::cout << ave(100,300);
}
```

![](https://bu.dusays.com/2024/11/28/67480783670e4.png)



**函数调用约定**

函数调用约定是函数调用与被调用者之间的一种协议，这个协议主要规定了以下两个内容：

如何传递参数    如何恢复栈平衡

```c++
int __cdecl ave(int a, int b)
{
    return a + b;
}

int main()
{
    ave(1, 2);
}

// __cdecl 参数入栈顺序 从右到左
// 堆栈平衡：谁调用谁平衡
// 正因为__cdecl这种堆栈平衡方式，能够支持不定量参数
```

```c++
int __stdccall ave(int a, int b)
{
    return a + b;
}
int main()
{
    ave(1, 2);
}
// __stdcall 参数入栈顺序  从右到左
// 堆栈平衡：函数自己恢复栈平衡
// Windows 编程中 WINAPI CALLBACK 都是 __stdcall 的宏
// 生成的函数名会加下划线，后面跟@和参数尺寸
```

![](https://bu.dusays.com/2024/11/28/674823c0b7270.png)

![](https://bu.dusays.com/2024/11/28/674823ba10bfe.png)







**递归函数**

**递归函数**是指在函数的定义中，调用了它自身的函数。递归的核心思想是将复杂的问题分解为更简单的同类问题。通过递归，问题可以被拆解成较小的子问题，直到达到最简单的基本情况（即递归的终止条件）。

```C++
int ave(int x)
{
    if (x == 1) return x;
    return x * ave(x - 1);
}
```

示例

```c++
#include <iostream>
#include <string>

using std::string;

typedef struct Role
{
	string Id;
	int Exp;
}*PROLE;

int GetDataCount(string str)
{
	int icount{};
	for (int i = 0; i < str.length(); i++)
	{
		if (str[i] == ';')
		{
			icount++;
			i += 3;
		}
	}
	return icount / 2;
}

void GetStringData(PROLE prole, string& strData, int istart)
{
	istart = strData.find("id=", istart);
	if (istart == std::string::npos) return;
	int iend = strData.find(";", istart + 3);
	prole->Id = strData.substr(istart + 3, iend - istart - 3);
	istart = iend + 1;

	istart = strData.find("exp=", istart);
	iend = strData.find(";", istart + 4);
	prole->Exp = std::stoi(strData.substr(istart + 4, iend - istart - 4));
	istart = iend + 1;

	return GetStringData(++prole, strData, istart);
}

int main()
{
	string strData = "id=Tomy Clare;exp=9523;id=Sunny;exp=9523;id=DyBaby;exp=25301;id=Simple;exp=25301;id=Bkacs11;exp=2100;id=DumpX;exp=36520;";

	int icount{ GetDataCount(strData) };
	PROLE pRole = new Role[icount];
	GetStringData(pRole, strData, 0);

	for (int i = 1; i < icount; i++)
	{
		for (int y = 1; y < icount; y++)
		{
			if (pRole[y].Exp > pRole[y - 1].Exp)
			{
				Role tmp = pRole[y - 1];
				pRole[y - 1] = pRole[y];
				pRole[y] = tmp;
			}
		}
	}
	for (int i = 0; i < icount; i++)
	{
		std::cout << pRole[i].Id << "  " << pRole[i].Exp << std::endl;
	}
}
```

**编译器**

![](https://bu.dusays.com/2024/11/28/674823b859eb4.png)

1️⃣ 未定义行为

C++标准未作规定的行为，称为未定义行为，未定义行为的结果是不确定的，具体在不同的编译器下会有不同的效果；

`c=2*a++ + ++a*6;`

这里先算`a++` 还是先算 `++ a` 就是一个未定义行为

`int x = -25602;`   `x=x>>2;`  x的结果在不同编译器下是不确定的，这也属于未定义行为。





​								**One Definition Rule**

ODR是一系列规则，而不是一个规则，程序中定义的每个对象都对应着自己的规则；

但是基本上来讲，任何变量、函数、类、枚举、模板，概念在每个转换单元中都只允许有一个定义；非inline的函数或者变量，在程序中，有且只有一个定义

仅在自己的转换单元有效：

1️⃣ `const`类型的变量 ：**全局const变量**，没有特别声明，只在当前转换单元内可见。**静态const变量**，在类或函数中的作用域内有效，不会暴露到其他地方。**外部const变量**，使用`extern`关键字使得变量拥有外部链接属性。 

```c++
const int a = 10;  // 只在当前源文件有效

extern const int b;  // 在其他文件中也可以引用，但变量定义只能在一个文件中
```

2️⃣ `static` 类型的变量 ：**全局static变量**  仅在定义它的源文件内有效，不能被其他源文件访问。**局部static变量** 只在函数内有效，但它们的生命周期持续整个程序运行周期，只有在第一次调用时初始化，之后的调用将保留上一次修改的值。

```C++
static int global_var = 10;  // 只能在当前源文件有效

void func() {
    static int local_static = 0;  // 只在该函数调用中有效，但其生命周期持续整个程序
}
```

 3️⃣ `inline` 变量：在当前转换单元内生效，但是可以跨多个转换单元共享。`inline`变量的作用类似于常量，允许在多个源文件中定义和使用，而不发生重复定义链接错误。

`inline`变量通常用在头文件中定义常量或其他需要跨多个文件共享的变量。`inline`变量遵循一翻译单元一实例的规则，保证多个源文件中不会出现重复定义的问题。

```C++
// header.h
inline int a = 1350;  // 内联变量，多个翻译单元可以共享这个变量。

// cpp文件
#include "header.h"
int main()
{
    a = 2000;   // 修改inline 变量
    return 0;
}
```

总结：仅在当前翻译单元生效的：`static` 和 默认的 `const` 变量。  可以跨多个翻译单元共享：`inline` 变量。

名称的链接属性：

1️⃣ 内部链接属性：该名称仅在本转换单元中有效。

2️⃣ 外部链接属性：该名称在其他的转换单元中也有效。

3️⃣ 无链接属性： 该名称仅仅能够用于该名称的作用域内访问。