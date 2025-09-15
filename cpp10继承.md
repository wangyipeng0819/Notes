---
title: cpp10继承
date: 2024-12-11 14:01:26
categories:
- Cpp
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686c9d1fd91fc.png
---

**继承的概念**

```c++
class Base
{};
class Derive:public Base//公有继承
{}；
class Derive2:protected Base //保护继承
{};
class Derive3:private Base//私有继承
{};
//上面的代码中，Derive，Derive2，Derive3都继承自Base基类
//区别就是继承方式不同
```

**多继承**

如果省略继承方式，默认为`private`

```c++
class <派生类名>:<继承方式1><基类名1>,<继承方式2><基类名2>,....
{  
};
```





![](https://bu.dusays.com/2024/12/07/6753b6c38f762.png)

private 和 protected 区别：类继承，private无法被访问  protected可以被访问。

继承的时候，私有变量是占内存空间的，但是没办法访问。。。

**继承中的访问控制**

![](https://bu.dusays.com/2024/12/07/6753c254715ca.png)

![](https://bu.dusays.com/2024/12/07/6753c2ca2a6ed.png)

![](https://bu.dusays.com/2024/12/07/6753c3c88b103.png)

一般来说，尽量设计类的成员变量为private，如果需要访问这些成员变量，应该提供setter以及getter函数。

**继承中的构造函数**

```c++
class object
{
    
};
class mapObject :public object
{
    
};
class actObject :public mapObject
{
    
};
// 构造顺序为object->mapObject->actObject
```

```c++
#include <iostream>

class object
{
public:
	int x;
	int y;
	object()
	{
		std::cout << "object was created\n";
	}
	object(const object& obj)
	{
		std::cout << "object was created by copy\n";
	}
};

class mapObject :public object
{
public:
	int mapId;
	mapObject()
	{
		std::cout << " mapObject was created\n";
	}
	mapObject(const mapObject& obj)
	{
		std::cout << " mapObject was created by copy\n";
	}
	mapObject(int id) :mapId{ id }
	{
		
	}
};

class actObject :public mapObject
{
public:
	int damage;
	// 不能加上mapId  因为现在还没有构造，不认为mapId是actObject的成员
	actObject():mapObject{100},damage{100}
	{
		// 可以在里面写
		mapId = 3939;
		std::cout << " actObject was created\n";
	}
	// mapObject{obj}没有做类型转换就传进去了
	actObject(const actObject& obj):mapObject{obj}
	{
		std::cout << "actObject" << std::endl;
	}
};

int main()
{
	actObject obj{};
	std::cout << "------------" << std::endl;
	actObject obj2 = obj;  // 副本构造函数
}
```





**继承构造函数**

允许派生类从基类继承构造函数。这意味着派生类可以使用基类的构造函数来初始化从基类继承来的数据成员，而不需要在派生类中重新定义类似的构造函数（默认构造函数和副本构造函数无法继承）

语法：`using <基类名称>::<基类名称>;`

```c++
#include <iostream>

class object
{
public:
	int x;
	int y;
	object()
	{
		std::cout << "object was created\n";
	}
	object(const object& obj)
	{
		std::cout << "object was created by copy\n";
	}
};

class mapObject :public object
{
public:
	int mapId;
	mapObject()
	{
		std::cout << " mapObject was created\n";
	}
	mapObject(const mapObject& obj)
	{
		std::cout << " mapObject was created by copy\n";
	}
	mapObject(int id) :mapId{ id }
	{
		std::cout << id;
	}
	mapObject(int id, int id2) :mapId{ id }
	{
		std::cout << id;
	}
};

class actObject :public mapObject
{
public:
	// 这样就继承了mapObject中的构造函数   但是默认构造函数、副本构造函数没办法继承
	using mapObject::mapObject;
	int damage;
	// 不能加上mapId  因为现在还没有构造，不认为mapId是actObject的成员
	actObject():mapObject{100},damage{100}
	{
		// 可以在里面写
		mapId = 3939;
		std::cout << " actObject was created\n";
	}
	// mapObject{obj}没有做类型转换就传进去了
	actObject(const actObject& obj):mapObject{obj}
	{
		std::cout << "actObject" << std::endl;
	}
};

int main()
{
	actObject obj{200};
	actObject objx{ 200,300 };

	std::cout << "------------" << std::endl;
	actObject obj2 = obj;  // 副本构造函数
}
```

**继承中的析构函数与重名问题**

构造的时候由基类开始构造的，析构的时候恰恰相反，先析构派生类的

继承中的成员变量名称重复的问题

1️⃣ 函数名相同，参数不同

​	using 基类::函数名

不会出错，有个作用域的问题，不存在变量名重复的问题，访问的时候 使用作用域的符号 `object::x`

```c++
mapObject MAP;
MAP.x = 2500;
MAP.object::x = 3500; // 使用作用域来解决这个问题
```

2️⃣ 函数名相同，参数相同

​	基类::函数名

函数存在一个重载的问题，

```c++
// 参数不同名字相同   直接 在派生类里面引入函数
using object::showX;
// 不引入的话会被覆盖掉

// 如果基类和派生类的函数名参数以及返回类型都一样， using已经区分不出来了，只能通过作用域来区别
// object类
int ShowX(int x)
{
    
}

// mapObject 类
using object::showX;
int ShowX(int x)
{
    
}

// actObject 类
int ShowX(int x)
{
    
}
// main函数
mapObject MAP:
MAP.ShowX(1500);  // 调用派生类中的函数
MAP.object::ShowX(2500);  // 调用基类中的函数

actObject obj;
obj.ShowX("222");
obj.mapObject::ShowX(33);
obj.mapObject::object::ShowX(44);

// 也可以直接调用,没什么层次感
obj.object::ShowX(55);

// 直接这么操作即可
```

**多重继承问题**

多重继承带来的重复继承的问题

```c++
#include <iostream>

class Wolf
{
public:
	void bite()
	{
		std::cout << "Wolf Bite!\n";
	}
	void eat()
	{
		std::cout << "Wolf eat!\n";
	}
};

class Man 
{
public:
	void eat()
	{
		std::cout << "Man eat!\n";
	}
};

class WolfMan :public Wolf, public Man
{
public:
	void Change()
	{
		IsWolf = !IsWolf;
	}
	void eat()
	{
		if (IsWolf) Wolf::eat();
		else Man::eat();
	}
private:
	bool IsWolf = false;
};

int main()
{
	WolfMan Jack;
	Jack.bite();
	Jack.eat();
	Jack.Change();  // 变身
	Jack.eat();   //  
}
```

派生类之间的内存并不是重叠的

```c++
// class MoveObject
class MoveObject
{
public:
	int x;
	int y;
};

// main
int main()
{
	WolfMan Jack;
	Jack.bite();
	Jack.Wolf::x = 250;
	Jack.Wolf::y = 250;

	std::cout << Jack.Wolf::x << std::endl;  // 250
	std::cout << Jack.Man::x << std::endl;  // -858993460
}
```

**虚基类**，如果发现已经继承一次了，下次就不会再继承的

```c++
// class Man :public virtual MoveObject   这样就是虚基类
// class Wolf:public virtual MoveObject

// main
int main()
{
	WolfMan Jack;
	Jack.bite();
	Jack.Wolf::x = 250;

	std::cout << Jack.Wolf::x << std::endl;
    std::cout << Jack.x << std::endl;    // 这三个都是指向的一个x
	std::cout << Jack.Man::x << std::endl;  // 虚基类，这样都是250，它俩x指向的是同一个地方，没有歧义了
}
```

**从内存的角度来理解继承**

未完待续。。。