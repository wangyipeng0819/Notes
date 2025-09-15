---
title: cpp08类
date: 2024-12-11 14:00:28
categories:
- Cpp
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686c9d1fd91fc.png
---

**静态成员变量**

static  关键字声明一个类的静态成员变量，类的静态成员变量的特点：

1️⃣ 所有类的实例中，共享类中的静态成员变量

2️⃣ 类的静态成员变量在没有类的实例的情况下，依然可以访问

3️⃣ 类的静态成员变量并不完全属于类

```c++
class T
{
public:
    static int count;   // 内存空间不属于类
    int hp;
};
int T::count = 100;  //定义 必须加上T::

int main()
{
    T::cout << 350;
    T t1;
}
```

静态成员变量的初始化

```c++
class T
{
    inline static int count{};  // 利用inline  在类里面定义count
public:
    int hp;
};
```

```c++
class T
{
    inline static int count{};
public:
    int hp;
    T()
    {
        count ++;
    }
    void GetCount()
    {
        return count;
    }
};

T t4;
int main()
{
    T t1, t2, t3;
    T t5;
    std::cout << t1.GetCount();
}
```

```c++
#include <iostream>

class T
{
    inline static int count{};
public:
    int hp;
    T()
    {
        count ++;
    }
    void GetCount()
    {
        return count;
    }
    ~T()
    {
        count--;
    }
};

T t4;
int main()
{
    T t1, t2, t3;
    T t5;
    std::cout << t1.GetCount();
}
```

**静态成员常量**

```c++
class T
{
public:
    const static int count{250};  // 有时候可以在这里定义，有时候不可以，我个人觉得在外面定义
    int hp;
    T()
    {
        count ++;
    }
    void GetCount()
    {
        return count;
    }
    ~T()
    {
        count--;
    }
};
```

**静态成员函数**

![](https://bu.dusays.com/2024/12/02/674da3de88eb6.png)

```c++
// 第一条
class T
{
     inline static int count{};
public:
    int hp;
    static void Test() // const 第三条，不需要const来限定
    {
        // hp++;  第二条  没有类的实例不能访问这个   一 二  条冲突
        // this->hp++;  避免访问非静态成员变量
        count ++ ;
    }
};

int main()
{
    T::Test();
}
```

**友元类**

友元类是一个可以访问另一个类的私有成员（`private`）和保护成员（`protected`）的类。即使是私有或保护成员，友元类也可以直接访问它们，打破了类的封装性（encapsulation）的一部分。友元类的机制为类提供了一种灵活的方式来控制哪些外部类可以访问其私有成员。

友元会破坏类的封装性，没有更好选择的情况下再使用友元，友元咧不是一种平等的关系，你能访问它的私有变量，它不一样能访问你的私有变量

```C++
class T
{
    int hp;  // 默认为私有的
    int mp;
    void GetMP()
    {
        std::cout << mp;
    }
    friend void SetHP(T& t1);  // 可以有多个友元
};

void SetHP(T& t1)
{
    t1.hp = 100;
    t1.GetMP();
}
```

```c++
#include <iostream>
class T1;  // 不在这里声明，那T中的T1编译器识别不出来
class T
{
    int hp;  // 默认为私有的
    int mp;
    void GetMP()
    {
        std::cout << mp;
    }
    friend void SetHP(T& t1, T1& t2);  // 可以有多个友元
    friend void SetMP(T& t1, T1& t2);
};

class T1
{
    int hp;  // 默认为私有的
    int mp;
    void GetMP()
    {
        std::cout << mp;
    }
    friend void SetHP(T& t1, T1& t2);  // 可以有多个友元
    friend void SetMP(T& t1, T1& t2); // 放在public 或者 private 都可以
};

void SetMP(T& t1, T1& t2)
{
    t1.hp = 100;
    t1.GetMP();
}

void SetHP(T& t1, T1 t2)
{
    t1.hp = 100;
    t1.GetMP();
}
```

**嵌套类**

1️⃣ 用法

2️⃣ 作用域

```c++
#include <iostream>
class Role
{
	int hp;
	static void test() {}
public:
    static void testx() {}
	enum class WeaponLv
	{
		normal = 0,
		high,
		rare,
		myth
	};

	Role()
	{
		Weapon::test1();
	
	}

	class Weapon
	{
		int tt1;
		Weapon* ReturnW();
	public:
        Weapon()
        {
            testx();  // 可以访问Role静态成员函数
        }
		static void test1() {}

	public:
		Weapon()
		{
			Role role;
			role.hp++;
		}
		short lv;
		WeaponLv wlv;
	};
	
	int mp;
	Weapon leftHands;
};

Role::Weapon* Role::Weapon::ReturnW()  // 注意作用域
{
	return this;
}
int a1;
int main()
{
	int x=250;

	class T
	{
		int hp;
		int mp;
		int count;
		void GetHP()
		{
			a1++;
			//x++;
		}
		static int GetCount()
		{
			
		}
	};
	T t1;
}
```

访问权限问题：嵌套类可以访问外层类的所有成员 ，外层类仅能访问嵌套类的共有成员

枚举类型也可以放进去，但有个作用域的问题

**局部类**

**定义在函数内的类称为局部类**：局部类的定义必须写在类内、 局部类中不允许使用静态成员变量、局部类可以访问全局变量。

静态成员变量不能定义在类里面，定义在外面又受作用域的问题，矛盾

虽然有`inline`关键字， 但避免那么写    局部类都少用

**嵌套类模块化问题**

```c++
// Role.h
#pragma once
// 这里不要Skill.h的
class Role
{
public:
    class Skill;
};
```

```C++
// Skill.h
#pragma once
#include "Role.h"
class Role::Skill
{
public:
    int hp;
    int mp;
};
```

```C++
#include <iostream>
#include "Role.h"
#include "Skill.h"

int main()
{
    
}
```



malloc 和 new 的本质区别：malloc仅仅分配内存，new除了分配内存以外还会调用构造函数

free() 没有调用析构函数，delete不进释放内存空间，会调用类的析构函数

![](https://bu.dusays.com/2024/12/02/674dba5f2954b.png)

![](https://bu.dusays.com/2024/12/02/674dba5f505ef.png)

![](https://bu.dusays.com/2024/12/02/674dba5f2e452.png)

**从底层理解类**

![](https://bu.dusays.com/2024/12/02/674dbde4a10b1.png)

![](https://bu.dusays.com/2024/12/02/674dbe7601fb6.png)

![](https://bu.dusays.com/2024/12/02/674dbfcc11236.png)

**类的自定义函数调用**







**项目**

```c++
#include <iostream>

class HPMed
{
public:
    int Recover{ 100 };
};

class Role
{
    int hp;
    int maxHp;
public:
    Role()
    {
        hp = 1000;
        maxHp = 3500;
    }
    void GetHP()
    {
        std::cout << "HP:" << hp << "\\" << maxHp<<"\n";
    }
    void EatMed(HPMed& hpMed)
    {
        hp += hpMed.Recover;
        hp = hp > maxHp ? maxHp : hp;
    }
};

int main()
{
    Role user;
    HPMed med;
    //user = user + med;
    user.GetHP();
    while (1)
    {
        int a;
        std::cin >> a;
        user.EatMed(med);
        system("cls");
        user.GetHP();
    } 
}
```

