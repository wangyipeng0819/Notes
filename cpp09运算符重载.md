---
title: cpp09运算符重载
date: 2024-12-11 14:01:00
categories:
- Cpp
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686c9d1fd91fc.png
---



运算符重载的主要目的是为了让目标代码更方便使用和维护，而不是提升开发效率，重载运算符未必能提升开发效率。

① 让类也支持原生的运算  比如 + -  * / 

② 提升对程序的控制权  比如重载new  delete new[]   delete[]

它允许你为自定义类型（如类或结构体）定义操作符的行为，使得这些操作符可以像内建类型一样工作。通过重载运算符，可以使自定义对象之间的运算更加直观和简洁。运算符重载是通过定义特殊的成员函数或友元函数来实现的



**初探运算符重载**

```C++
#include <iostream>

class Person
{
	friend bool operator<(Person& psa, Person& psb);
	friend bool operator<(Person& psa, unsigned short _Age);
	unsigned short Age;
public:
	Person(unsigned short _Age) :Age{ _Age }{}

	// 获得私有变量
	unsigned short GetAge() { return Age; }
	bool operator>(Person& person);
};

// 类的成员函数来实现运算符重载
bool Person::operator>(Person & person)
{
	return Age > person.Age;
}

// 非类的成员函数实现运算符重载
bool operator<(Person& psa, Person& psb)
{
	return psa.GetAge() < psb.Age;
}

// 跟数字比
bool operator<(Person& psa, unsigned short _Age)
{
	return psa.Age < _Age;
}

int main()
{
	Person Man(20);
	Person Woman(50);
	// 调用的是 Person::operator>(Person & person)
	// 
	if (Man>Woman) std::cout << " 你找到富婆了！！" << std::endl;

	// 调用operator<(Person& psa, unsigned short _Age)
	// 
	if (Man < 10) std::cout << " 你找到富婆了！！" << std::endl;


	// 类的非成员函数实现运算符重载  声明为友元函数了
	// Man < Woman <=> operator<(Man,Woman)
	if (Man<Woman) std::cout << " 你找到富婆了！！" << std::endl;
}
```

！！！记住理解那些等价关系

**运算符重载的原则和时机**

![](https://bu.dusays.com/2024/12/03/674e63385333d.png)

不能把加法重载为减法！！！

![](https://bu.dusays.com/2024/12/03/674e65fd43456.png)

![](https://bu.dusays.com/2024/12/03/674e65fd5013e.png)

![](https://bu.dusays.com/2024/12/03/674e65fd5ca10.png)

传递值或者传递引用  建议传递引用  尤其是右值引用，不修改在前面加上`const` 就可以了



**重载赋值运算符**

默认赋值运算符重载

```c++
/*
Role x, y;
....
y = x;
*/
class Role
{
public:
    int hp;
    int mp;
    Role& operator=(const Role& role);
    {
        hp = role.hp;
        mp = role.mp;
        return *this;
    }
    // Role& operator=(const Role& role) = delete;   那么编译器就不再重载了
    // main.cpp中  y = x不再正确了
};
```

**实现赋值运算符的重载**

！！！operator=只能用类的成员函数来实现

```c++
#include <iostream>
class Role
{
public:
    int hp;
    int mp;
    Role& operator=(const Role& role);
    
};
Role& Role::operator=(const Role& role)
{
        hp = role.hp;
        mp = role.mp;
    // 而且返回值是空的时候void Role::operator=(const Role& role)
    // 如果注释掉 return *this;  
    // 那么就不能 z = y = x;
    // z.operator=(y.operator=(x));   因为返回值为空，所以这个操作不能执行了
    // 返回值编程Role 类型也没问题，但是引用节省开销
        return *this;
}

int main()
{
    Role x, y;
    x.hp = 100;
    y.mp = 200;
    y = x;
    // z = y = x;
}
```

当类里面有指针的时候，往往不能采用默认运算符，必须重载（例子如下）

```C++
int main()
{
    Role x, y, z;
    x.hp = 100;
    y.mp = 200;
    
    z = y =x;
    std::cout << y.hp << "//////" << y.mp;
    char strA[]{ "aaaaabbbccccc" };
    hstring str{ strA };
    // 如果strA的内存空间改变了，那么str没有值了，因为内存空间不属于str
    strA[0] = 0;
    std::cout << str.c_str();
}
```

**重载赋值运算符的代码如下**

```c++
#include <iostream>
#include "hstring.h"


unsigned short hstring::GetStrLen(const char* str) const
{
	unsigned short len = 0;
	while (str[len] != '\0') {  // 遍历字符串直到遇到 '\0'
		len++;
	}
	return len;
}
// 拷贝  函数
void hstring::CopyStrs(char* dest, const char* source)
{
	unsigned short len = GetStrLen(source);
	if (len > usmlen)
	{
		delete[] cstr;  // 释放之前分配的内存
		cstr = new char[len];  // 重新分配内存
		usmlen = len; // 修正内存长度 
	}
	memcpy(cstr, source, len);
	cstr[len] = '\0';
	uslen = len;  // 字符串长度修正

}


// 缓冲区
// 默认构造函数 
hstring::hstring()
{
	usmlen = 0x32;  // 内存空间等于32字节
	uslen = 0;
	cstr = new char[usmlen];
	cstr[0] = '\0';  // 初始化为空字符串
}


// 初始化hstring类对象的构造函数。
// 带参数的构造函数
// 可以接受一个或者多个参数，在创建对象时，通过这些参数来初始化对象的数据成员。这使得对象在创建之初就能被赋予特定的值。
hstring::hstring(const char* str) :hstring()
{
	CopyStrs(cstr, str);
}

// 拷贝构造函数  使用一个已经存在的同类型对象初始化新创建的对象。它的参数是一个常量引用，指向同类型的对象。
hstring::hstring(const hstring& str):hstring()
{
	CopyStrs(cstr, str.cstr);
}
hstring& hstring::operator=(const hstring& str)
{
	if (this != &str)
	{
		CopyStrs(cstr, str.cstr);
	}
	
	return *this;
}

hstring& hstring::operator=(const long long& value)
{
    if (cstr != nullptr)
    {
        delete[] cstr;
        cstr = nullptr;
        usmlen = 0;
        uslen = 0;
    }

    long long tempValue = value;
    bool isNegative = false;
    // 计算数字的位数
    unsigned short count = 0;

    // 判断负号
    if (tempValue < 0)
    {
        count++;  // 负数的话  先把符号算进去
        isNegative = true;
        tempValue = -tempValue;
    }

    long long temp = tempValue;
    do
    {
        count++;
        temp = temp / 10;
    } while (temp > 0);

    // 先判断是否为负数再判断是否为0  如果为0 数值位数就是1
    if (tempValue == 0) count = 1;

    uslen = count;       
    usmlen = uslen + 1;       
    cstr = new char[usmlen];

    int index = usmlen - 2; // usmlen-1是\0
    cstr[usmlen - 1] = '\0';

    if (isNegative)
    {
        cstr[0] = '-';  
    }
    // 这里tempValue已经是个非负数了
    while (tempValue > 0)
    {
        char charValue = (char)(tempValue % 10 + '0');
        cstr[index--] = charValue;
        tempValue = tempValue / 10;
    }
    return *this;
}

```

**重载<< 和>>**

![](https://bu.dusays.com/2024/12/07/67545d0215102.png)







·

**重载括号()**

![](https://bu.dusays.com/2024/12/03/674eb16dbd58f.png)

**重载二元运算符**

![](https://bu.dusays.com/2024/12/03/674eb16d9fbeb.png)

