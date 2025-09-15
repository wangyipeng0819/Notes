---
title: cpp04函数进阶
date: 2024-12-11 13:57:24
categories:
- Cpp
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686c9d1fd91fc.png
---

### 函数返回-返回指针和引用

#### 返回指针不要返回局部变量

```c++
#include <iostream>

int clen(const char* str)
{
    int i;
    for (i = 0; str[i]; i ++ )
    return ++i;
}

char* cstr(const char* str)
{
    int len = clen(str);
   // char strRt[0x20];   // 这就是错误示范，返回了一个局部变量
    // 只要不释放，一直在堆上面
    char* strRt = new char[len];
    memcpy(strRt, str, len);
    return strRt;
}
```

**项目设计**

![](https://bu.dusays.com/2024/11/24/6742aa525f2f5.png)

```c++
#include <iostream>

typedef struct Role
{
	char* name;
	int Hp;
	int maxHp;
	int Mp;
	int maxMp;
	int lv;
}*PROLE, ROLE;
int clen(const char* str)
{
	int i;
	for (i = 0; str[i]; i++);
	return ++i;
}
char* cstr(const char* str)
{
	// 不要返回一个局部变量
	int len = clen(str);
	// 只要不释放，一直在堆上面
	char* strRt = new char[len];
	memcpy(strRt, str, len);
	return strRt;
}

ROLE CreateMonster(const char* str, int Hp, int Mp)
{
    // 申请了空间，把它返回之后，还得赋值到 role上面，造成了性能损失
	Role rt{ cstr(str), Hp, Hp, Mp, Mp, 1 };
	return rt;
}

int main()
{
	char* str;
	str = cstr("123");
	std::cout << str;

	ROLE role = CreateMonster("aoteman", 1500, 1500);

	std::cout << role.name << std::endl;
	std::cout << role.Hp << std::endl;
}

/*---------------改进方法1️⃣  返回指针--------------------*/
PROLE CreateMonster(const char* str, int Hp, int Mp)
{
    // 这下返回的是4字节的指针
	PROLE rt = new ROLE{ cstr(str), Hp, Hp, Mp, Mp, 1 };
	return rt;
}

int main()
{
	char* str;
	str = cstr("123");
	std::cout << str;

	PROLE role = CreateMonster("aoteman", 1500, 1500);
    // 这样的话性能折损几乎没有了
	std::cout << role->name << std::endl;
	std::cout << role->Hp << std::endl;
}

/*---------------改进方法2️⃣  返回引用--------------------*/
ROLE& CreateMonster(const char* str, int Hp, int Mp)
{
	PROLE rt = new{ cstr(str), Hp, Hp, Mp, Mp, 1 };
    // 由于是引用，所以不能传空指针
	return *rt;
}

int main()
{
	char* str;
	str = cstr("123");
	std::cout << str;

	ROLE& role = CreateMonster("aoteman", 1500, 1500);

	std::cout << role.name << std::endl;
	std::cout << role.Hp << std::endl;
}
```

#### 数组的引用（补充）

```c++
int a;
int& b = a;
int c[100];
// int& d[100] = c;  error
int (&e)[100] = a;  // 必须为100 才行   而且得告诉编译器，e是个引用
```

```c++
void ave(int(&art)[100])
{
    std::cout << sizeof(art) << std::endl;
    for (auto x : art);
}

int main()
{
    int a[100];
    int c[100];
    int (&b)[100] = c;
    
    ave(a);
}
```

#### 右值引用

```c++
#include <iostream>

void Add1(int&& a)
{
    std::cout << a;
}

void Add2(int& a)
{
    std::cout << a;
}

int main()
{
    int c = 320 + 250;
    int& d = c;
    // 右值引用 &&
    // int&& e = d;  右值引用你就不能指向内存中有位置的对象，
    int&& e = 320;   // 右值引用可以指向临时的东西
    
    e = 1500;
    
    int x = c + 100 + 200;
    Add2(x);
    // 使用右值引用，减少不必要的内存浪费
    Add1(c + 100 + 200)
}
```

```c++
Role CrateMonster()
{
    Role rt{100,200};
    return rt;
}

void show(Role&& r1)
{
    std::cout << r1.hp;
}

int main()
{
    Show(CreateMonster());
    // 原来的方法是创建一个额外的变量来保存
    // ROLE role = CreateMonster("aoteman", 1500, 1500);
}
```

![](https://bu.dusays.com/2024/11/24/6742beef8067c.png)

右边的部分，`Role r1` 是需要创建变量的，有内存开销。  `Role& r1`无法传递局部变量（临时量）。

左边部分，虽然Role CreateRole()分配到栈里，但跑到下一个函数，这部分内容要擦除的。



#### 函数的本质（底层）

![](https://bu.dusays.com/2024/11/24/6742c2f421215.png) push  有个作用  传参的    ，把参数推到栈区

![](https://bu.dusays.com/2024/11/24/6742c5e9d7b54.png)

函数的名字就是一个内存地址  代码就是二进制数据

```c++
int Add(int a, int b)
{
    return a + b;
}

int main()
{
    std::cout << Add;  //函数的名字就是一个内存地址
    
    char* str = (char*)Add;
    for (int i = 0; i < 30; i ++ )
    {
        printf("%02X\n", (unsigned char)str[i]);
        // 打印出来的内容就是命令对应的二进制
    }
    
    // 测试下能不能写入
    // str[0] = 25; 出错  常量一样，，不让写
}
```

#### 函数指针

函数返回类型  (*函数指针变量名)(参数类型  参数名称, ...  参数类型  参数名称);

`int (*pAdd)(int a, int b)`

```c++
#include <iostream>

// 新用法
// 声明一个函数指针类型
typedef char(*pfAdd)(int, int);
using pFAdd = char (*)(int, int);


int AddX(int a, int b)
{
    return a + b;
}

int Add1(int a, int b)
{
    return a + b;
}

int main()
{
    // 类型转换  上面声明的 两个类型
    pfAdd pAdd1 = (pfAdd)Add1;
    pFAdd pAdd2 = (pFAdd)AddX;
    // 类型不一样的时候使用强制类型转换
    // 声明函数指针
    int (*pxAdd)(int, int) = AddX;
    std::cout << pxAdd(1, 2) << std::endl;


    std::cout << sizeof(pAdd1) << " " << pAdd1(30, 35);
}
```

Tips

![](https://bu.dusays.com/2024/11/24/6742d6cc78d8f.png)

#### 从函数的角度彻底认识栈

栈十分重要，底层很重要

1️⃣ 栈是预先分配好的，连续的内存空间，局部变量就放在栈空间上，通过控制esp来实现局部变量的创建和释放

```c++
int Add(int a, int b)
{
    int c = 250;
    int d = Ave(a, b);
    c = c + d;
    return c;
}
```

2️⃣ 栈平衡如果被破坏，函数就可能不能返回到预期的位置，同理，利用这个原理，我们也可以控制目标程序进入指定的位置，来获取目标操作系统的控制权限，这也就是栈攻击的技术原理，同时编写代码时也要积极预防栈攻击。

```c++
00C61073 6A 32             push   32h
00C61075 68 FA 00 00 00    push   0FAh
00C6107A E8 A1 FF FF FF    call   00C61020
00C6107F 83 C4 08          add    esp,8
00C61082 89 45 FC          mov    dword ptr [ebp-4],eax
```

3️⃣ 当我们逆向的时候，可以通过函数头部的sub esp, x来判断这个函数有多少局部变量



4️⃣ 关于CPU寄存器的说明

​	eax： 函数的执行结果会通过eax来传递

​	esp： 栈顶，栈顶以下的值代表着是可以访问的局部变量。

​	ebp： 栈底

​	eip：  CPU执行的位置

#### 函数重载（C++内容）

函数重载（Function Overloading）是编程中一种允许在同一个作用域中定义多个同名函数的技术。这些函数根据其参数的个数或类型的不同，能够执行不同的操作。



```c++
int ave(int *a, int b)
{
    return 0;
}

int ave(int a[], int b)
{
    return 0;
}
// 这种是不可以的，int* a 和  int a[]其实是一样的 
// 函数重载使用时不能出现歧义

int ave(int& a, int& b)
{
    return 0;
}

int ave(int a, int b)
{
    return 0;
}


int main()
{
    int a = 100, b = 100;
    ave(a, b);  // 这种就是不行，出现歧义了  上面两个函数都适用
}
```

```c++
int ave(int& a, int& b)
{
    return 0;
}


float ave(float a, float b)
{
    return 0;
}

int main()
{
    char a = 100, b = 100;
    std::cout << ave(a, b) << std::endl;  // 自动转成float
    std::cout << ave((int)a, (int)b) << std::endl;
    // 这地方的 转换是临时 的变量，没有固定的内存地址
}
```

```c++
int ave(int a, int b)
{
    return 0;
}

int ave(const int a, const int b)
{
    return 0;
}
// 不可以重载，是以值传参，不能修改内存里面的值，所以const形同虚设
```

```c++
int ave(int& a, int& b)
{
    return 0;
}

int ave(const& int a, const& int b)
{
    return 0;
}
// 可以重载，
int main()
{
    int a = 100, b = 100;
    const int c = 20, d = 10;
    std::cout << ave(a, b) << std::endl;  //  非调用第一个
    std::cout << ave(c, d) << std::endl;  //  常量执行第二个
}
```

// 函数重载的时候不能默认参数   编译器分不清

#### 函数模板（C++内容）

先说下什么是模板，模板就是编写“`类型参数化`”的代码，等到编译时再根据具体类型生成对应版本。

模板在编译时实例化，当编译器遇到模板的具体使用时，例如调用模板函数或创建模板类实例时，编译器会生成相应的代码。这一过程称为模板的实例化。



1️⃣函数模板  ，定义通用的函数逻辑，`template<typename T> T add(T a, T b)`

```c++
template<typename T>
T add(T a, T b) {
    return a + b;
}

int main() {
    std::cout << add(3, 4) << "\n";      // 推导出 T=int
    std::cout << add(3.5, 2.1) << "\n";  // 推导出 T=double
}
```

2️⃣类模板，定义通用的类或数据结构，`template<typename T> class MyVector`

```c++
template<typename T>
class Box {
public:
    T value;
    Box(T val) : value(val) {}
    void show() { std::cout << value << "\n"; }
};

int main() {
    Box<int> intBox(10);
    Box<std::string> strBox("hello");

    intBox.show();   // 输出 10
    strBox.show();   // 输出 hello
}
```

下面主要介绍下**函数模板**

函数模板（Function Template）是 C++ 中的一种通用编程技术，允许程序员编写可以操作多种数据类型的函数，而无需为每种数据类型单独重写代码。函数模板通过引入模板参数，使函数的类型可以在调用时动态确定。

```c++
template <typename type1> type1 ave(type1 a, type1 b)
{
    return (a + b) / 2;
}
```

```c++
#include <iostream>

template <typename type1>
type1 ave(type1 a, type1 b, type1 c)
{
	return (a + b + c) / 3;
}

type1 ave1(type1* a, type1* b, type1* c)
{
	return (a + b + c) / 3;
}
// 把type1当作一种类型来使用   指针  数组  都可以

int main()
{
	std::cout << ave(12.0f, 250.0f, 35.33f) << std::endl;
    std::cout << ave(11, 12, 13) << std::endl;
    std::cout << ave((char)11, (char)12, (char)13) << std::endl;
}
```

```c++
// 指定函数参数的模板
template <typename type1> type1 ave(type1 a, type1 b)
{
    return (a + b) / 2;
}
int main()
{
	std::cout << ave<int>(12.0f, 250.0f, 35.33f) << std::endl;  // 转换成int类型了
    // 相当于执行下面的代码
    // int ave(int a, int b);   而不是 float ave(float a, float b);
}
```

```c++
//函数重载大于函数模板

template<typename type1>
type1 bigger(int a, int b)
{
    return a > b ? a : b;
}
// 例外
//template<>
int* bigger(int* a, int* b)
{
    return *a > *b ? a : b;
}
float* bigger(float* a, float* b)
{
    return *a > *b ? a : b;
}

template<typename type1>
type1 ave(type1 a, type1 b, type1 c)
{
    return (a + b + c) / 2;
}

int main()
{
    // 本来是想着减少内存开销，传入指针，结果发现，传入的是地址
    int a{ 100 }, b{ 200 }, c{ 300 };
    c = *bigger(&a, &b);
    c = *ave(&a, &b, &c);   //相当于把内存地址加起来除以3
}
```

关键字**auto decltype**

1️⃣auto 不保留const属性  `const int a{ 10 };` `auto b = a;`  这里的b就是int类型 。

2️⃣ auto 会优先推断为值类型而非引用类型。

```c++
int b{200};
int& l{b};
auto c = l;   // 这里l还是int类型
```

3️⃣ auto 利用函数返回值来确定类型的时候，函数会执行。

```c++
auto x = ave(1,2); // ave(1,2)会执行    x的类型根据ave函数的返回类型来确定
```

根据特性2️⃣

```c++
#inlcude <iostream>


// auto 拖尾函数
auto bigger(int& a, int& b)->float   // 转成float类型了
    
int& bigger(int& a, int& b)  // 这里改成auto不行，会把&转成int
{
    return a > b ? a : b;
}
int main()
{
    int a{100};
    int b{200};
    
    // 能这么写是因为返回值不是临时变量，左值
    // bigger(a, b) = 500;  // 比较a 和 b 哪个数大，大的 改成500;
}
```

**decltype关键字**

```c++
int a{};
unsigned b{};
decltype(a-b) x;  // 相当于unsigned x;    有符号数和无符号运算，结果为无符号数
```

如果表达式没有经历过运算，那么类型是一样的，  可以保留引用属性  （引用必须初始化）

（没有固定内存地址的情况）如果经历过运算了，会根据结果的类型来推断类型

（有固定内存地址的情况）

```c++
int a{100};
int* pa{&a};

decltype(*pa) x = a;// x 是int 类型的引用
```

![](https://bu.dusays.com/2024/11/24/674327db2d723.png)

`decltype(pa[5]) x;`  相当于int& x    pa[5]是有地址的，只是不归我们访问而已

![](https://bu.dusays.com/2024/11/24/674328e67c835.png)

#### ![](https://bu.dusays.com/2024/11/24/67432a50d20d6.png)

```c++
auto bigger(int& a, int& b)->decltype(a > b ? a : b)
{
    return a > b ? a : b;
}
```

```c++
decltype(auto) bigger(int& a, int& b)
{
    return a > b ? a : b;
}
```

#### 推断函数模板返回类型

```c++
template<typename T1, typename T2>
T1 ave(T2 a, T1 b)   // T1
{
    return (a + b) / 2;
}

int main()
{
    // 返回类型是由后面的决定的
    ave(100.02f, 200);
}
```

```c++
#include <iostream>

template<typename T1, typename T2>
delctype(auto) bigger(T1& a, T2& b)
{
    return a > b ? a : b;
}

int main()
{
    float a = 50;
    int b = 50000;
    // float& lx = b;
    
    bigger(a, b);  // 结果是个float引用，但是不能指向int类型
    
    std::cout << b;
}
```

#### 函数模板参数的默认值

 

```C++
#include <iostream>
// 返回值默认为int类型, 建议写在前面
// 另一种写法,  可以不指定了，跟着T1 走
// template<typename T1, typename T2, typename TR = T1>
template<typename TR = int, typename T1, typename T2>
TR ave(T1 a, T2 b)
{
    return (a + b) / 2;
}
```

函数模板可以有非类型的模板参数

```c++
template<typename type1, int max, int min>
bool AddHp(type1& hp, type1 damage)
{
    hp -= damage;
    if (hp > max) hp = max;
    return hp < min;
}
```

实践，试下这个函数

```C++
#include <iostream>
template<int max, int min, typename type1>   // 建议将 非类型参数写在前面
bool ChangeHp(type1& hp, type1 damage)  // max 和 min 不是变量
{
    hp -= damage;
    if (hp > max) hp = max;
    return hp < min;
}

int main()
{
    int hp = 2500;
    // const int x = 2500;
    // ChangHp<x,1000>(hp, 100);  // 这样会报错，x 是常量才可以，这也说明了max和min不是变量
    ChangHp<2000,1000>(hp, 100);   // <>里面的数据需要指定
    std::cout << hp;
}
```

```c++
//  指定默认值
#include <iostream>
// 指定默认值之后，在main里面就不需要<>来指定max  min的值了
template<int max = 2000, int min = 1000, typename type1>
bool ChangeHp(type1& hp, type1 damage) 
{
    hp -= damage;
    if (hp > max) hp = max;
    return hp < min;
}

int main()
{
    int hp = 2500;
    ChangHp(hp, 100);
    std::cout << hp;
}
```

将非类型参数写在后面

```C++
// 将max 和 min 转换为type1类型
template<typename type1, type1 max = 2000, type1 min = 1000, > type1 类型
bool ChangeHp(type1& hp, type1 damage) 
{
    hp -= damage;
    if (hp > max) hp = max;
    return hp < min;
}
```

![](https://bu.dusays.com/2024/11/25/6743c6535c905.png)

**处理不同元素数组的模板函数**

```C++
template<typename T, short count>
T ave(T(&ary)[count])
{
    T all{};
    for (int i = 0; i < count; i ++ )
        all += ary[i];
    return all / count;
}
int main()
{
    int a[5]{1,2,3,4,5};
    std::cout << ave(a);  // count的值是通过数组大小自动推导出来的
    
    // 显式指定模板参数
    //std::cout << ave<int, 5>(a) << std::endl;
    
    // 只指定 count 的值
    std::cout << ave<decltype(a[0]), 5>(a) << std::endl;
    
}
```

**小项目：万能排序工具**

```c++
#include <iostream>
template<typename T>
void Swap(T& a, T& b)
{
    T tmp{ a };
    a = b;
    b = tmp;
}

template<typename T>
void Sort(T* ary, unsigned count, bool BigSort = true)
{
    for (unsigned i = 1; i < count; i ++ )
        for (unsigned i = 1; i < count; i ++ )
        {
            bool bcase = BigSort > ary[i] > ary[i - 1] : ary[i] < ary[i - 1];
            if (bcase) Swap(ary[i], ary[i - 1]);
        }
}

int main()
{
    int a[6]{ 123, 343, 434, 1, 3454, 33};
    short a1[6]{ 123, 343, 434, 1, 3454, 33};
    
    std::string a2[5]{ "12","asd","sdf","sesdas","sdasd" };
    
    Sort(a1, 6);
	Sort(a, 6);
	Sort(a2, 5);
	for (auto x : a)std::cout << x << std::endl;
	for (auto x : a1)std::cout << x << std::endl;
	for (auto x : a2)std::cout << x << std::endl;
}
```

