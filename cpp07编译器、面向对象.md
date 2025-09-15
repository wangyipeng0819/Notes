---
title: cpp07编译器、面向对象
date: 2024-12-11 14:00:01
categories:
- Cpp
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686c9d1fd91fc.png
---

`#define A B`   将标识A定义为B的别名    `#define 整数  int`       `整数  a{};`

`#define H`      定义一个标识符H， 代码中的H将会被删除掉   ，`int H a =>  int a;`         提高代码可阅读性的

`#undef H`      删除这个标识符的定义，以后用不了了，  删除的顺序是根据代码编译的顺序进行的。

**定义复杂表达式的宏**

```c++
#define _out_

#define SUM(X, Y) X+Y*3
#define RELEASE(x) delete[] x, x = nullptr;

// #define BIGGER(X, Y) (X>Y?X:Y)   // (X>Y?X:Y)必须加括号，不然处理不了
#define BIGGER(X, Y) ((X)>(Y)?(X):(Y))  // 更保险的方式，全都加上括号

#define SHOW(X) std::cout<<X
#define SHOW1(X) std::cout<<#X   // 处理成字符串

#define SHOW2(X,Y) void X##Y() {std::cout<<#X;}
SHOW2(test,22)  // 函数名字就叫test22
int main()
{
    std::cout << SUM(100,200);   // 100 + 200 * 3
    int* a = new int[50];
    RELEASE(a);
    
    std::cout << BIGGER(100, 200);
    SHOW(2333);   //  输出
    SHOW1(233sdjfkldsjf);
    
    test22();   // 就可以调用这个函数了
}
```

`const int width{100};`

`#define width 1000`

`#define` 的方式来定义常量存在一个问题，定义的常量并不具备类型，有时候并不安全

**`namespace` 命名空间**

把相关的函数，变量，结构体放进一个命名空间里面

```c++
// code.cpp
namespace t
{
    int v;
}
const int p{};
int main()
{
    t::v = 100;
    
    int p = 200;
    std::cout << p << std::endl;  // 输出200， 局部的
    std::cout << ::p << std::endl;  // 访问全局命名空间中的p  值为0
}
```

```c++
// t.cpp
extern int x;
void test()
{
    ::x;  // 就能访问到code.cpp 中的 x
}
```

全局命名空间，所有具有链接属性的对象，只要没有定义命名空间，就默认定义在全局命名空间中，全局命名空间中的成员的访问不用显示的指定，当局部名称覆盖了全局名称时才需要显式的指定全局命名空间。



命名空间是可以扩展的,命名空间可以放在多个头文件里面 。

```c++
namespace t
{
    int v;
}
namespace t
{
    int a;
}

int main()
{
   // t::// 这里会出来俩变量
}
```

命名空间的声明

```c++
// htc.h   头文件
namespace t
{
    extern int v;  // 这就是声明
}

//code.cpp
#include <iostream>
#include "cc.h"
// int t::v{ 20 };  这样也是可以的 
namespace t
{
    int v{200};  // 声明和定义分开写了
}
int main()
{
    std::cout << t::v;
}
```

命名空间的嵌套

```c++
// cc.h
#pragma once
namespace t
{
    extern int v;
    void test();

    namespace hack
    {
        void hack();
    }
}
// code.cpp
#include <iostream>
#include "cc.h"

namespace t
{
    int v{ 200 };
    void test()
    {
        std::cout << t::v << std::endl;
    }
}

void t::hack::hack()
{
    std::cout << "hack!!";
}

int main()
{
    t::test();
} 
```

未命名的命名空间  只针对本转换单元有效（待查询资料整理）

命名空间的别名

```c++
namespace a = t::hack;

int main()
{
    a::hack();  // 面对深度嵌套的命名空间，使用别名
}
```

```c++
namespace htd
{
    void sendSms();
    namespace hack
    {
        void hackServer();
    }
}
namespace hServer = htd::hack;
hServer::hackServer();
```

**未定义的命名空间和static**

未命名的命名空间是用`namespace {}`定义的，没有名字。

- 任何定义在未命名命名空间中的变量、函数或类型仅在当前编译单元（文件）中可见。
- 实际上，未命名命名空间的内容被编译器视为具有**内部链接** （类似于`static`）。

优点：提供了作用域隔离，防止名称冲突。相比static， 它可以包含多个声明(变量、函数、类型等)并形成一个逻辑分组。   建议使用未命名的命名空间代替`static`，因为它支持更复杂的结构和逻辑。

static 修饰的 变量或函数在当前文件中可见。

- `static`变量和函数具有**内部链接** ，即它们只能在定义它们的文件中访问。
- 主要用于标记单个实体的作用域，不支持逻辑分组。

![](https://bu.dusays.com/2024/11/30/674ab5bf6fd98.png)

**预处理指令逻辑**

```c++
#define VERSION 103
#define SENDSMS 1
#if VERSION==100&&SENDSMS   // 这里可以做一些计算的  #if VERSION==(100 + 1)
void sendSms() {}
#elif VERSION==101||SENDSMS
void sendSms() {}
#elif VERSION==102
void sendSms() {}
#elif VERSION==103
void sendSms() {}
#elif VERSION==104
void sendSms() {}
#endif
// #if 和 #endif 是成套出现的
```

**预定义宏**

标准预定义标识符  `__func__`   返回函数名字

```c++
int ave(int a, int b)
{
	std::cout << __func__ << std::endl;  // 输出ave
	return (a + b) / 2;
}

int main()
{
	ave(100, 1200);
	std::cout << __func__;  //  输出main
}
```

```c++
std::cout << __DATE__ << std::endl;  // 日期
std::cout << __TIME__ << std::endl;  // 时间
std::cout << __func__ << std::endl;  // 函数名
std::cout << __FILE__ << std::endl;  // 源文件名称
std::cout << __LINE__ << std::endl;  // 当前的行号
std::cout << __cplusplus << std::endl;  // 当翻译单元为C++时，__cplusplus为一个整数文本否则为未定义
```

![](https://bu.dusays.com/2024/11/30/674a850dab4fb.png)

```c++
#ifdef _CHAR_UNSIGNED
	std::cout << "char unsigned";
#else
  
#endif


int main()
{
    std::cout << __FUNCDNAME__ << std::endl;
    std::cout << __FUNCSIG__ << std::endl;
    std::cout << __TIMESTAMP__ << std::endl;
    std::cout << _WIN64 << std::
    //  _WIN64 仅在64位下才显示
#ifdef _WIN32
	std::cout << "win32\n";
#else
}
```

`_DEBUG`  在debug模式下可以使用，发布模式下没有了

**调试**

```c++
#define _DBG_FOR    // 通过注释这里就可以不调试 


#ifdef _DBG_FOR
#endif 
```

**assert断言**

![](https://bu.dusays.com/2024/11/30/674aafc634452.png)

```c++
#define NDEBUG
#include <iostream>
#include <cassert>

int main()
{
    std::cout << "请输入一个整数\n";
    int c;
    std::cin >> c;
    assert(c);   // 只要是0就会弹出来
    std::cout << 1000 / c;
}
```

**static_assert**

用于编译时检查条件

`static_assert(bool 表达式,"错误信息");`

`static_assert(bool表达式);`

与assert不同，static_assert主要是用来在编译时检查重要的条件。  因此检查的bool表达式中，只能用于常量

**面向对象的编程**

![https://bu.dusays.com/2024/11/30/674b125faff00.png](https://bu.dusays.com/2024/11/30/674b125faff00.png)

类      定义了对象的属性（成员变量）和行为（成员函数）

对象   是类的实例化，可以用来存储数据和调用函数。

```c++
#include <iostream>

struct NPC
{
	int hp;
	int mp;
	int damage;
};

struct MONSTER
{
	int hp;
	int mp;
	int damage;
	int price;
};

struct ROLE
{
	int hp;
	int mp;
	int damage;
	int diamond;
};

bool Act(NPC* acter, NPC* beacter)
{
	beacter->hp -= acter->damage;
	return beacter->hp > 0;
}

int main()
{
	MONSTER atm{ 1000,500,100,6000 };
	ROLE zs{ 1000,1000,50,1000 };
	ROLE ls{ 1000,500,100,600 };

	Act((NPC*)&atm, (NPC*)&ls);  // 只进行强制类型转换就可以了，没必要写很多函数了
	std::cout << ls.hp;
}
```

**一个例子**

```c++
// Role.cpp
#include "Role.h"

void Role::Act(Role& role)
{
    role.hp -= damage;
}

void Role::Init()
{
    hpRecover = 3;
}


Role* Role::bigger(Role* role)
{
   return role->lv > lv ? role : this;
}

Role& Role::SetLv(int newLv)
{
    lv = newLv;
    return *this;
}
Role& Role::SetHp(int newHp)
{
    hp = newHp;
    return *this;
}
Role& Role::SetDamage(int newDamage)
{
    damage = newDamage;
    return *this;
}
```

```c++
// main
#include <iostream>

#include "Role.h"

int main()
{
    int a = 200;
    int* p = &a;
    int& c = *p;

    Role user;
    Role monster;
    user.SetLv(100).SetDamage(50).SetHp(500).bigger(&monster)->bigger(&user);
    Role* biggerRole= user.bigger(&monster);
}
```

```C++
// Role.h
#pragma once
class Role
{
private:
	int hpRecover;
	int hp;
	int lv;
	void Init();
public:
	/*如果inline函数的定义不在类内部，也不在头文件中，而是放在某个.cpp文件中，
	编译器确实可能出现问题，因为它在链接阶段无法找到该函数的实现*/
	inline int GetHp();
	
	int damage;
	Role* bigger(Role* role);
	void Act(Role& role);

	Role& SetLv(int newLv);
	Role& SetHp(int newHp);
	Role& SetDamage(int newDamage);
};
```

内联函数，以Role.cpp里面的函数为准，重写原则

**this指针**

`this`指针是一个指向调用成员函数的对象的指针。每当你在一个类的成员函数中使用`this`时，它自动指向当前调用该成员函数的对象。

```c++
#include <iostream>
using namespace std;

class MyClass {
public:
    int x;

    MyClass(int val) : x(val) {}

    void show() {
        cout << "x = " << x << endl;   
        cout << "this = " << this << endl;
    }
};

int main() {
    MyClass obj(10);
    obj.show(); // x = 10   this指针的值是对象的地址
    return 0;
}
```

**this** 指针的特点

 · `this`指针是隐式传递给成员函数的。不需要显示地将它作为参数传递，它是C++编译器自动提供的。

 · `this`指针是只读的。你不能修改 `this`指针本身（既不能让 `this` 指针指向其他对象），但是可以通过它访问的对象的成员。

 · 对于非静态成员函数， `this` 指针总是存在的，而在静态成员函数中没有this指针，因为静态函数不依赖任何特定的对象。

 · `this` 指针的类型是ClassName*， 它指向调用成员函数的对象。

```c++
class Myclass{
public:
    int x;
    
    Myclass(int val) : x(val) {}
    
    void setX(int x){
        this->x = x;
    }
};
```

`this->x` 表示当前对象的成员变量x，而x则表示函数的参数。这样可以避免成员变量与函数参数名称冲突的问题。

`this`  指针的使用场景

1️⃣ 区分成员变量和局部变量

```c++
class MyClass{
public:
    int x;
    MyClass(int x)
    {
        this->x = x;
    }
};
```

2️⃣ 链式调用

`this` 指针的返回值可以用于链式调用，常见于实现流式操作接口（比如std::string的操作）。返回`*this` 意味着返回当前对象的引用，从而可以链式调用多个方法。

```C++
class MyClass{
public:
    int x;
    MyClass& setX(int val)
    {
        x = val;
        return *this; // 返回当前对象的引用，实现链式调用
    }
    void show()
    {
        std::cout << "x = " << x << std::endl;
    }
};

int main()
{
    MyClass obj(10);
    obj.setX(20).setX(30).show();  // 链式调用
    return 0;
}
```

3️⃣ this 指针的特殊情况

`this` 指针在静态成员函数中的不存在，静态成员函数不依赖于任何特定的对象，因此它们没有`this`指针。你不能在静态成员函数中使用`this`，因为静态成员函数是类的级别上的，而不是对象的级别上的。

```c++
class MyClass{
public:
    static void staticFunction()
    {
        // 不能使用this指针，因为staticFunction 是静态成员函数
        // this-> x = 10;
    }
}
```

4️⃣ `this` 指针与 `const` 修饰符

 · 在常量成员函数中，`this` 指针是 `const`类型的，这意味着你不能通过this指针访问的对象成员。

```c++
class MyClass{
public:
    int x;
    MyClass(int val) : x(val) {}
    void show() const
    {
        //不能修改成员变量
        // x = 10;  // 错误： 不能在const函数中修改对象的状态
        
    }
};
```

5️⃣ `this` 指针与`const`对象

对于常量对象，`this` 指针会被隐式地转换为`const` 类型，确保通过`this`指针访问的对象不会被修改。

```c++
void printObject(const MyClass& obj)
{
    std::cout << obj.x << std::endl;
}
```

6️⃣ `this` 指针的返回类型

由于 `this` 指针的类型是当前类的指针，所以可以将它作为返回值返回。这样通常用于链式调用或在类成员函数中传递对象的引用。

```c++
class MyClass{
public:
    int x;
    MyClass(int val) : x(val) {}
    
    MyClass* getThis()
    {
        return this;  // 返回当前对象的指针。
    }
}
```

`this`指针是C++中非常重要的特性，它允许成员函数访问当前对象的成员，并且有助于在类中实现链式调用、区分参数和成员变量等功能。理解`this`指针对于深入掌握C++至关重要，特别是在对象操作和内存管理方面。

**`const`**

`const` 在前面说明函数返回值是`const`  ，在后面说明这个函数是`const`

```c++
#include "Role.h"

int main()
{
	const Role user;

	Role monster;
	const Role* puser{ &user };
	// puser->damage = 2;  // 常量指针不能编译
	// puser->    这里不能调用成员函数
	puser->GetHp();  // GetHp()后面加上const 就可以定义了
	monster.GetHp();
}
```

```c++
class Role
{
public:
    int hp;
    int GetHP() const;   // const 成员函数内部一律不可以做任何改变的
}
```

`const` 对象不能以任何方式改变，这是`const` 原则，在这个原则下，`const`对象只能调用`const`成员函数。

在`const` 成员函数下，`this`指针也变成了`const`指针。



成员函数是`const`的情况下，想返回引用，只能在前面加上`const`，或者不返回引用，按值来传递。

1️⃣ 所有没有设计修改成员变量的成员函数，一律加上`const`

2️⃣ 利用函数重载

```C++
int GetDamage() const;  // const 对象，调用这个函数
int GetDamage();   // 非const 对象，调用这个函数
```

**const 类型转换**

```C++
void test(Role* p)   // 忘记加const了， 所以下面才进行类型转换
{
    p->SetHP(5000);
}
int main()
{
    const Role user;
    const Role* puser{ &user };
    test((Role*)(&user));  // C 语言的转换
    test(const_cast<Role*>(&user));   // c++ 的转换
    std::cout << puser->GetHP();
}
```

**mutable 关键字**

`mutable` 关键字是一个非常特殊的修饰符，它主要用于类的数据成员上。`mutable` 的作用是允许即使在 `const` 成员函数中，也可以修改被修饰的成员变量的值。`

1️⃣ `const` 成员函数的限制，指一个声明为 `const` 的成员函数，这意味着该成员函数承诺不会修改对象的状态，即不会修改对象的非 `mutable` 成员变量。

```c++
class MyClass{
public:
    int x;
    MyClass(int val) : x(val) {}
    void show() const{    // const 成员函数
        x = 10;    	//  错误， const 成员函数不能修改成员变量
    }
};
```

`show()` 被声明为 `const` 成员函数，它承诺不修改类的任何成员。因为 `x` 不是 `mutable` 成员变量，所以试图在 `const` 成员函数中修改 `x` 是不允许的，编译器会报错。



2️⃣  `mutable` 的作用

`mutable` 关键字的出现就是为了允许某些特定成员变量在 `const` 成员函数中修改。这对实现缓存、延迟计算、记录日志等场景非常有用。

```C++
class MyClass{
public:
    mutable int x;
    MyClass(int val) : x(val) {}
    void show() const{    // const 成员函数
        x = 10;    	//  允许：x 是 mutable， 可以在 const 函数中修改
    }
};
```

**构造函数**

```C++
#include <iostream>
class T
{
public:
    int hp;
    int mp;
    
    int GetMP() const { return mp; }
    void SetMP(int _lv) { lv = _lv; }
private:
    int lv;
};

int main()
{
    T t1{ 100,200 };  // 不能初始化lv
    t1.SetMP(109);  // 可以设置lv
}
```

构造函数在类被创建的时候自动被调用，一般用来创建新的类实例时执行初始化操作，构造函数与它所在的类同名，并且没有返回值，任何类都至少有一个构造函数；

```C++
#include <iostream>
class T
{
public:
    int hp;
    T()  // 无参数的构造函数
    {
        std::cout << "T()\n";
        hp = 100;
        mp = 200;
    }
    T(int _hp, int _mp,)  // 有参数的构造函数
    {
        std::cout << "T(int,int)\n";
        hp = _hp;
        mp = _mp;
    }
    
    T(T& t)   //  留下疑问，为什么要加个引用
    {
        hp = t.hp;
        mp = t.GetMP();
    }

private:
    int mp;
};

int main()
{
    T t1{100, 200};   // 调用有参数的构造函数
    T t2(t1);   // 调用的就是T(T& t)
    std::cout << "\n.............\n";
    std::cout << t1.hp << " " << t1.GetMP();
}
```

如果一个类没有明显的定义一个构造函数，那么编译器会自动的添加一个默认的构造函数，这个默认的构造函数是无参数、无返回值的函数

**`default`关键字**

```c++
Role() = default;
Role(int _lv = 500) { lv = _lv; }  // 如果有默认值的话编译器不能区分出来
```

不建议这么写

**`explicit` 关键字**

被 `explicit` 关键字修饰的构造函数会禁用类型转换，它可以防止编译器在进行类型转换时，自动调用某些构造函数。这是为了避免隐式类型转换（隐式构造或隐式转换）导致潜在的错误或不希望发生的行为。

隐式类型转换的风险

```C++
class MyClass
{
public:
    MyClass(int x)
    {
        std::cout << "----------" << x << std::endl;
    }
};

void foo(MyClass obj)
{
    std::cout << "In foo" << std::endl;
}

int main()
{
    foo(42);
}
```

在这个例子中，`foo(42)` 实际上是调用了 `MyClass` 的构造函数，并将 `42` 转换为 `MyClass` 类型对象。这种隐式转换可能是安全的，但它也可能会引入意外的行为，特别是在构造函数的逻辑较为复杂时。

```c++
#include <iostream>
class MyClass
{
public:
    explicit MyClass(int x)
    {
        std::cout << "----------" << x << std::endl;
    }
};

void foo(MyClass obj)
{
    std::cout << "In foo" << std::endl;
}

int main()
{
    foo(MyClass(42));  // 必须显示的调用构造函数
}
```

**深入理解构造函数**

使用成员初始化列表

```c++
class ROLE
{
private:
    int lv;
    int hpRecover;
    void init()
    {
        hpRecover = 3;
    }
public:
    //  效率更高  某些情况下只能用这种方式初始化
    ROLE(int _lv, int _damage) : lv{_lv},damage{_damage} {}
    int hp;
    int damage;
    void Act(ROLE& role);
};
```

使用成员函数初始化列表这样的方式构造类，要注意一个问题，即为成员赋值的顺序不是依据代码的顺序，而是成员变量在类的出现顺序。

```C++
class ROLE
{
private:
    int lv;
    int hpRecover;
public:
    ROLE(int _lv, int _damage) : lv{_lv},damage{_damage},hp{_lv*100},hpRecover{lv} {}
    int damage;
    void Act(ROLE& role);
};

/*
实际初始化的顺序
hp=lv*100
hpRecover = lv;
lv = _lv;
damage = _damage;
*/
```

**委托构造函数**

它允许一个构造函数调用另一个构造函数。委托构造函数通过将其构造逻辑委托给同一类中的另一个构造函数来减少代码重复，并提高代码的可维护性。

```C++
class ROLE
{
private:
    int lv;
    int hpRecover;
public:
    ROLE():ROLE(100,200) {}
    ROLE(int _lv, int _damage):lv{_lv},damage{_damage} {}
    int damage;
    void Act(ROLE& role);
};

// 委托构造函数初始化列表里不能初始化成员变量且只能调用一次同一个类的构造函数
```

```C++
#include <iostream>
#include <string>

class Person {
private:
    std::string name;
    int age;

public:
    // 委托构造函数：调用另一个构造函数
    Person() : Person("Unknown", 0)  // 委托之后，这里就不写name(name), age(age)这种了列表了
    {
        std::cout << "Default constructor called" << std::endl;
    }

    // 另一个构造函数
    Person(const std::string& name, int age) : name(name), age(age) {
        std::cout << "Parameterized constructor called" << std::endl;
    }

    void display() const {
        std::cout << "Name: " << name << ", Age: " << age << std::endl;
    }
};

int main() {
    Person p1;  // 将调用默认构造函数，进而委托给带参数的构造函数
    p1.display();

    Person p2("Alice", 30);  // 调用带参数的构造函数
    p2.display();

    return 0;
}
```

**副本构造函数**

```c++
Role role1;
Role role2(role1);
// 编译器为类指定了一个默认的副本构造函数，我们也可以手动指定副本构造函数。
```

```C++
// Role.h
Role(Role& rl) 
{
    hp = rl.hp;
}

// main.cpp
user.setHP(5000);
Role userA(user);  // 使用我们自己做的构造函数，只拷贝hp的值
```

`Role(Role& rl)`   引用问题

在 C++ 中，当你传递对象给函数或者在对象初始化时，如果参数类型不是引用（即传值方式），那么会发生**副本创建**，即编译器会调用副本构造函数来创建一个新的对象。传值会导致一次额外的复制操作，消耗额外的时间和资源。

使用引用类型参数（`Role& rl`）可以避免这种多余的复制，直接引用原有对象。因此，副本构造函数的参数一般定义为**常量引用**（`const Role& rl`），来避免对原对象进行修改。

如果副本构造函数的参数是按值传递，那么在调用副本构造函数时，编译器会尝试用这个值初始化一个新对象。这又会调用副本构造函数，而副本构造函数是用来构造副本的，这会导致递归调用，造成栈溢出。

通过传递引用，副本构造函数避免了重新调用自身，解决了递归调用的问题。

```c++
Role userA = user;  // userA还没有构造，把user当成参数
userA = user;   // 这里就不再构造函数了
```



**析构函数**

用来销毁一个类，析构函数没有参数，没有返回类型，使用default来定义

```c++
~类名()   // 没参数就不能重载
{
    
}
// 空的析构函数
~ROLE() = default;  
```

```c++
class Role
{
    int* ary;
public:
    Role()
    {
        ary = new int[100];
        std::cout << "\n类被创建";
    }
    ~Role()
    {
        delete[] ary;
        std::cout << "\n 类被销毁";   // 这样写防止内存泄露
    }
}

// std::string 就用到这种技术
```

**实现字符串类`hstring`**

`std::string` 设计一个类 `hstirng`

`hstring str("你好！");`    // 构造函数

`hstring strA(str);`   // 副本构造函数

```c++
#include <iostream>

class hstring
{
private:
	char* c_str;
	unsigned short len;
	unsigned short length(const char* str)
	{
		unsigned short len{};
		while (str[len++]);
		return len;
	}
public:
	hstring()
	{
		len = 1;
		c_str = new char[1] {0};
	}
	hstring(const char* str)
	{
		len = length(str) + 1;
		//c_str = (char*)str;  // 指向了常量的内存空间,所以运行起来会出错
		c_str = new char[len];
		memcpy(c_str, str, len);
	}
	char* Show() const
	{
		return c_str;   
	}

	~hstring() {}
};
int main()
{
	hstring str("你好！！");
	std::cout << str.Show();
}
```

私有成员变量的访问

在 C++ 中，类的私有变量（private members）只能在类的内部进行访问，不能直接从类的外部进行访问。私有成员是为了实现数据封装（Encapsulation）这一面向对象编程的基本原则，目的是隐藏类的内部实现细节，只允许通过公共接口（如公共成员函数）来访问和修改私有数据。

### 1. **如何访问类的私有成员**

C++ 提供了一些方法来访问和修改类的私有变量：

#### 1.1 **通过公共成员函数访问私有成员**

最常见的方式是通过公共成员函数（通常称为 getter 和 setter）来访问或修改私有成员。类的私有成员可以通过这些函数提供对外接口。

**示例代码**：

```c++
// cpp复制代码
#include <iostream>
using namespace std;

class MyClass {
private:
    int privateVar;  // 私有成员变量

public:
    // 公共构造函数
    MyClass(int val) : privateVar(val) {}

    // Getter：访问私有成员变量
    int getPrivateVar() const {
        return privateVar;
    }

    // Setter：修改私有成员变量
    void setPrivateVar(int val) {
        privateVar = val;
    }
};

int main() {
    MyClass obj(42);  // 创建对象并初始化私有成员
    cout << "Private Var: " << obj.getPrivateVar() << endl;  // 通过公共成员函数访问私有成员
    obj.setPrivateVar(100);  // 修改私有成员
    cout << "Private Var: " << obj.getPrivateVar() << endl;  // 再次访问修改后的值
    return 0;
}
```

在上述代码中，私有成员变量 `privateVar` 只能通过公共的 getter 函数 `getPrivateVar()` 和 setter 函数 `setPrivateVar()` 访问和修改。这是实现类的**数据封装**的一种常见方法。

#### 1.2 **通过友元函数（Friend Function）访问私有成员**

除了通过公共成员函数外，还可以通过定义**友元函数**来访问私有成员。友元函数不是类的成员函数，但它可以访问类的私有成员。友元函数被声明为类的朋友，拥有对该类私有成员的访问权限。

**示例代码**：

```c++
// cpp复制代码
#include <iostream>
using namespace std;

class MyClass {
private:
    int privateVar;  // 私有成员变量

public:
    MyClass(int val) : privateVar(val) {}

    // 声明友元函数
    friend void showPrivateVar(const MyClass& obj);
};

// 友元函数定义
void showPrivateVar(const MyClass& obj) {
    cout << "Private Var: " << obj.privateVar << endl;
}

int main() {
    MyClass obj(42);
    showPrivateVar(obj);  // 通过友元函数访问私有成员
    return 0;
}
```

在这个例子中，`showPrivateVar()` 是 `MyClass` 类的友元函数，它可以直接访问 `MyClass` 的私有成员 `privateVar`。

#### 1.3 **通过友元类（Friend Class）访问私有成员**

除了友元函数，C++ 还允许整个类成为另一个类的友元类。友元类可以访问被它声明为友元的类的私有成员。

**示例代码**：

```c++
// cpp复制代码
#include <iostream>
using namespace std;

class AnotherClass;  // 前向声明

class MyClass {
private:
    int privateVar;

public:
    MyClass(int val) : privateVar(val) {}

    // 声明 AnotherClass 为友元类
    friend class AnotherClass;
};

class AnotherClass {
public:
    void showPrivateVar(const MyClass& obj) {
        cout << "Private Var: " << obj.privateVar << endl;
    }
};

int main() {
    MyClass obj(42);
    AnotherClass anotherObj;
    anotherObj.showPrivateVar(obj);  // 通过友元类访问私有成员
    return 0;
}
```

在这个例子中，`AnotherClass` 是 `MyClass` 的友元类，因此 `AnotherClass` 可以直接访问 `MyClass` 的私有成员 `privateVar`。

#### 1.4 **通过指针和引用访问私有成员**

如果有指向对象的指针或引用，并且你能通过公共接口访问私有成员，那么你就可以通过这些接口来访问私有成员。值得注意的是，直接通过指针或引用访问私有成员是不可行的，除非使用 `friend` 函数或者 `getter` 和 `setter`。

**示例代码**：

```c++
// cpp复制代码
#include <iostream>
using namespace std;

class MyClass {
private:
    int privateVar;

public:
    MyClass(int val) : privateVar(val) {}

    // Getter
    int getPrivateVar() const {
        return privateVar;
    }
};

int main() {
    MyClass obj(42);
    MyClass* ptr = &obj;
    cout << "Private Var via pointer: " << ptr->getPrivateVar() << endl;  // 通过指针访问
    return 0;
}
```

### 2. **总结**

- **私有成员变量**（private members）是 C++ 中类的一部分，只能在类内部访问，不允许外部直接访问。
- **公共成员函数**（getter/setter）通常是访问私有成员的推荐方式。通过提供控制访问的接口，可以保护数据的安全性和一致性。
- **友元函数**和**友元类**允许外部的函数或类访问该类的私有成员，但这种方式通常会打破封装性，因此应该谨慎使用。
- 在实际应用中，通常建议将类的成员变量设为私有，并通过公共接口（getter、setter）进行访问和修改，以确保数据的有效性和安全性。

总之，访问类的私有变量应该遵循**封装性**的原则，避免直接在外部访问类的私有数据，而是通过设计好的接口来进行操作







在 C++ 中，`=` 运算符用于赋值操作，但是在不同的上下文中，`=` 运算符的含义和行为是不同的。你提到的两种情况：

```c++
// cpp复制代码
Role userA = user; // 这种形式是复制初始化
userA = user;       // 这种形式是赋值操作
```

这两者之间有很大的区别，我们将逐一分析它们的含义和不同之处。

### 1. `Role userA = user;` —— **复制初始化**（Copy Initialization）

这行代码属于**复制初始化**（Copy Initialization），它通常会调用**副本构造函数**。它的执行过程如下：

- **首先创建一个新对象 `userA`**，并用 `user` 对象初始化它。
- 在这个过程中，**副本构造函数**会被调用，通常情况下是 `Role(const Role& user)`。

也就是说，`Role userA = user;` 实际上是通过调用类的副本构造函数来构造 `userA` 对象的副本。这种赋值形式会复制 `user` 对象的状态到 `userA`。

### 副本构造函数的工作流程：

- 调用副本构造函数创建 `userA`。
- 将 `user` 对象的成员变量的值复制给 `userA`。
- 这通常会涉及成员的深拷贝（如果类中有动态分配内存等需要处理的资源）。

### 2. `userA = user;` —— **赋值操作**（Assignment）

这行代码属于**赋值操作**（Assignment），它会调用对象的**赋值运算符**。赋值运算符用于将一个已有对象的状态赋值给另一个对象。执行过程如下：

- 这里 `userA` 是已经存在的对象，而 `user` 是另一个对象。
- `userA = user;` 会调用类的赋值运算符，通常是 `Role& operator=(const Role& user)`，该运算符用于将 `user` 对象的状态复制到 `userA` 中。

### 赋值运算符的工作流程：

- 如果类的赋值运算符已定义，`operator=` 会被调用。
- 它会将 `user` 对象中的数据成员赋值给 `userA`，并可能涉及内存管理（例如，释放已有资源、分配新资源等）。
- 如果没有定义自定义的赋值运算符，编译器会提供一个默认的赋值运算符，该默认版本执行逐个成员的赋值操作。

### 关键区别

1. **操作对象不同：**
   - `Role userA = user;` 是创建一个新的对象 `userA` 并初始化它，这个过程调用的是副本构造函数。
   - `userA = user;` 是将已经存在的对象 `userA` 赋值为 `user`，这个过程调用的是赋值运算符。
2. **调用的函数不同：**
   - `Role userA = user;` 调用的是类的**副本构造函数**，它在创建新对象时用于初始化。
   - `userA = user;` 调用的是类的**赋值运算符**，它用于将一个已存在对象的值复制到另一个已存在对象。
3. **创建新对象与赋值对象：**
   - 在 `Role userA = user;` 中，`userA` 是通过副本构造函数创建的一个新对象。
   - 在 `userA = user;` 中，`userA` 已经存在，它的内容会被替换为 `user` 的内容。
4. **效率考虑：**
   - 副本构造函数通常用于创建新对象，因此它需要分配内存并初始化对象。
   - 赋值运算符通常在对象已经存在的情况下执行，它会处理已有对象的资源，可能会包括释放资源、重新分配内存等操作。

### 深入理解赋值运算符与副本构造函数

为了进一步理解它们的行为，我们可以看一下副本构造函数和赋值运算符的实现。

#### 副本构造函数

副本构造函数的目标是创建一个新对象，它接受一个对象作为参数，并用该对象的内容初始化新对象。典型的副本构造函数实现如下：

```
cpp复制代码class Role {
public:
    Role(const Role& other) {
        // 通过 other 来初始化当前对象
        this->name = other.name;  // 假设是简单的成员复制
        this->age = other.age;
        // 如果有动态内存分配，需要深拷贝
    }
};
```

#### 赋值运算符

赋值运算符用于将一个已存在的对象的状态赋值给另一个已存在的对象。典型的赋值运算符实现如下：

```
cpp复制代码class Role {
public:
    Role& operator=(const Role& other) {
        if (this != &other) {  // 防止自我赋值
            this->name = other.name;
            this->age = other.age;
            // 如果有动态内存分配，确保释放旧的资源并分配新资源
        }
        return *this;
    }
};
```

### 总结

- `Role userA = user;` 使用的是**副本构造函数**，其作用是创建一个新的对象 `userA` 并用 `user` 初始化它。
- `userA = user;` 使用的是**赋值运算符**，其作用是将已有对象 `user` 的内容赋值给已有对象 `userA`。

两者的主要区别在于：副本构造函数负责新对象的创建并进行初始化，而赋值运算符负责已有对象的内容赋值。因此，它们各自有不同的性能和功能特点，适用于不同的情境。

