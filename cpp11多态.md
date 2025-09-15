---
title: cpp11多态
date: 2024-12-11 14:01:33
categories:
- Cpp
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686c9d1fd91fc.png
---

**多态概念**

多态是面向对象编程中的一个重要概念，它允许在基类的指针或引用指向派生类对象时，调用不同的函数实现。换句话说，多态是指同一接口在不同对象中有不同的实现方式。

多态的基本分类：

①  编译时多态（静态多态）：通过函数重载和运算符重载来实现。

② 运行时多态 （动态多态）：通过继承和虚函数来实现。

对象多态

① 向上转型   父类==>子类  人是动物 

② 向下转型   子类==>父类  动物是人  相当于用子类代替父类  （大神和沙雕之间二象性）

```c++
// 30多态概念.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。

#include <iostream>

class Animal
{
public:
    int  age;
};

class Human :public Animal
{
public:
    int Money;
    void doit()
    {
        std::cout << "Do it~!!!";
    }
};

int main()
{
    Human laow{};
    laow.age = 50;
    laow.Money = 3000;

    // 向上转型
    //Animal anm = laow;  // 可能发生内存切片的问题
    //std::cout << anm.age << std::endl;
    // 使用指针,没有内存截断问题
   
    //Human* human = anm1;  // human的成员变量要比anm1多
    Animal* anm1 = &laow;
    // 向下转型   更大的目的是调用里面的函数
    Human* human = (Human*)anm1;

    human->doit();
}
```

**方法多态**

①  静态多态： 函数重载、函数模板

```c++
#include <iostream>

class Animal
{
public:
    int  age;
};

class Human :public Animal
{
public:
    int Money;
    void doit()
    {
        std::cout << "Do it~!!!";
    }
};

void BeAct(Human* R)
{

}
void BeAct(Animal* anm)
{

}

int main()
{
    Human laow;
    Animal dog;

    BeAct(&laow);
    BeAct(&dog);  // 静态多态   编译的时候决定使用哪个函数
}
```

②  动态多态

```c++
// 30多态概念.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。

#include <iostream>

class Animal
{
public:
    int  age;
    void BeAct(Animal* anm)
    {
        std::cout << "动物被攻击！！" << std::endl;
    }
};

class Human :public Animal
{
public:
    int Money;
    void doit()
    {
        std::cout << "Do it~!!!";
    }
    void BeAct(Animal* anm)
    {
        std::cout << "人物被攻击！！" << std::endl;

    }
};

int main()
{
    Human laow;
    Animal dog;

    

    int id;
    Animal* bA;
    std::cin >> id;
    if (id)
    {
        bA = &dog;
    }
    else bA = &laow;
    bA->BeAct(&dog);  // 编译的时候不确定，就叫做动态多态 ！！！
}
```

**`virtual关键字`**

它是一个在基类中声明，并且希望在派生类中重写（或覆盖）的方法。通过虚函数，可以实现多态性，使得程序能够在运行时根据对象的实际类型来调用适当的函数，而不是在编译时就决定调用哪个函数。

![](https://bu.dusays.com/2024/12/08/67550c89eea41.png)

override 后缀可以强制要求检查函数是重载

final 后缀可以终止函数的重载

```c++
// 31多态虚函数.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//

#include <iostream>
class MoveObject
{
public:
    int x;
    int y;
    // virtual只能放在类内部声明
    // 不加上virtual 就是静态绑定
    void virtual Move();
// 如果这里设置成private，那么这个函数在派生类里面用不了
// private:
    // 虚函数在派生类和基类中的返回值要求基本一致，但是当返回类型为类类型的指针和引用时除外
    virtual MoveObject* Move1()
    {
        x++;
        y++;
        return this;
    }
};

void MoveObject::Move()
{
    x++;
    y++;
}


class NPCObject:public MoveObject
{
public:
    // 这种不是覆盖   而是动态绑定
    // 不重写 也能运行，因为继承了基类的Move函数，调用基类中的Move函数
    void Move()
    {
        x++;
        y++;
        std::cout << "我是NPC\n";
    }
    //void Move(int a);  // 如果参数不一样，那就是一个正常的函数
    // 如果这么写的话，那么调用的就是MoveObject中的Move函数


    // 虚函数在派生类和基类中的返回值要求基本一致，但是当返回类型为类类型的指针和引用时除外
    // MoveObject派生除了NPCObject，那么这里可以用MoveObject或者NPCObject
    // 如果NPCObject类不再派生的话，后面加上final可以
    NPCObject* Move1()   // 协变
    {
        x++;
        y++;
        return this;
    }
    /*只要是派生类的虚函数都要加上override关键字*/
};

class MonsterObject :public MoveObject
{
public:
    void Move()
    {
        x++;
        y++;
        std::cout << "我是怪物\n";
    }

    MonsterObject* Move1()   // 协变
    {
        x++;
        y++;
        return this;
    }
};

// 这是以前的写法
/*
* 
void Move(MonsterObject* obj)
{
    obj->Move();
}

void Move(NPCObject* obj)
{
    obj->Move();
}*/

// 现在多态的写法， 只写一个基类即可
// 调用类的对象时无法使用虚函数的，必须使用基类指针来实现虚函数的调用

// 这样写调用的是MoveObject中的Move
/*void Move(MoveObject obj)
{
    obj.Move();
}*/
// 引用的本质也是个指针
void Move(MoveObject& obj)
{
    obj.Move();
}

int main()
{
    MonsterObject snake;
    NPCObject zsf;
    // 如果没有virtual，那么就是静态绑定，什么都不会输出
    // 调用的是MoveObject 中的 Move函数
    Move(snake);
    Move(zsf);
}
// 建议虚函数做成私有的，不破坏封装，可以通过基类指针访问
```

**虚函数详解**

```c++
// 虚函数详解.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//

#include <iostream>

class MoveObject
{
public:
    int x;
    int y;

    MoveObject()
    {
        // 构造函数，先构造基类再构造派生类
        //  派生类还没有构造，所以调用的就是MoveObject中的Move
        std::cout << this << std::endl;
        this->Move();
    }
    virtual void Move()
    {
        std::cout << "MoveObject Moving~\n";
    }

    void test()
    {
        // 相当于this->Move()
        // Move又是一个虚函数，所以调用的就是下面MonsterObject中的Move函数
        Move();
    }

    ~MoveObject()
    {
        // 只能做静态绑定，调用基类中的Move函数
        Move();
    }
};

class MonsterObject :public MoveObject
{
public:
    void Move() override
    {
        std::cout << "Monster Moving\n";
    }
    MonsterObject()
    {
        std::cout << this << std::endl;
        MoveObject::test();
    }
    ~MonsterObject()
    {
        // 先析构派生类  再析构基类  
        Move();
    }
};


// 面试重点
int main()
{
    MonsterObject snake;
    // 调用的是Monster中的Move
    snake.test();
}
```

**调用虚函数基类版本**

```c++
MoveObject::Move();  // 这样就能调用基类版本的Move函数了
```

**默认实参在虚函数重点错误**

```c++
// 基类
virtual void AutoMove(int step = 2)
{
    std::cout << "auto move" << step << std::endl;
}
// 派生类
void AutoMove(int step = 3) override
{
    // 编译的时候已经决定了step的值,在这里改没什么用
    std::cout << "~~~~~~auto move" << step << std::endl;
}

// main()
MonsterObject snake;

MoveObject* p = &snake;
p->AutoMove();
// 调用的是派生类的AutoMove  但是step的值在编译的时候就已经确定了，在派生类里面改是没有用的
```

**释放含有虚函数的派生对象**

```c++
// 虚函数详解.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//

#include <iostream>

class MoveObject
{
public:
    int x;
    int y;

    MoveObject()
    {
        std::cout << this << std::endl;
        this->Move();
    }
    virtual void Move()
    {
        std::cout << "MoveObject Moving~\n";
    }

    virtual void AutoMove(int step = 2)
    {
        std::cout << "auto move" << step << std::endl;
    }

    void test()
    {
        Move();
    }

    virtual ~MoveObject()
    {
        std::cout << " Moveobj析构函数！！" << std::endl;
        Move();
    }
    // 析构函数中没有内容的话可以换种写法
    // virtual ~MoveObject() = default;
};

class MonsterObject :public MoveObject
{
public:
    void Move() override
    {
        std::cout << "Monster Moving\n";
    }
    MonsterObject()
    {
        std::cout << this << std::endl;
        MoveObject::Move();
    }
    void AutoMove(int step = 3) override
    {
        // 编译的时候已经决定了step的值,在这里改没什么用
        std::cout << "~~~~~~auto move" << step << std::endl;
    }
    ~MonsterObject()
    {
        std::cout << " Monsterobj析构函数！！" << std::endl;
        Move();
    }
    
};


// 面试重点
int main()
{
    // MonsterObject snake;

    MoveObject* p = new MonsterObject();
    p->AutoMove();
    delete p; // 触发析构函数，先析构派生类再析构基类

    // 析构函数不是虚函数，所以调用MoveObject中的析构函数
    //p->~MoveObject()
    // 但是没有释放内存，造成内存泄露，所以做一个虚析构函数
}
```

在类里面，析构函数和构造函数是静态绑定，其余都是动态绑定的。

**对象多态详解**

```c++
// 33对象多态详解.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//

#include <iostream>
class MoveObject
{
public:
    int x;
};

/*virtual*/
class MonsterObject :public MoveObject
{};

class NPCObject :public MoveObject
{};


class Wolf :public MonsterObject
{};

class Man :public MonsterObject
{};

class WolfMan :public Wolf,public MoveObject,public Man
{};


int main()
{
    MonsterObject monster;
    MoveObject* _move = &monster;  // 隐式类型转换，向上转型

    // _move = new MoveObject();
    // 编译器不确定_pmonster指向的是什么
    // MonsterObject* _pmonster = _move; // 隐式类型转换不被允许  向下转型不允许
    //可以强制，但是基类得是可以访问的才行
    MonsterObject* _pmonster = (MonsterObject*)_move; // 强制类型转换  向下转换
    MonsterObject* _pmonsterA = static_cast<MonsterObject*>(_move); // 静态强制类型转换
    // 虚基类不能强制转换，类是虚拟的

    /*------------------------------------------------------------*/

    WolfMan wolfman;

    void* ptr = &wolfman;
    Wolf* pwlf = &wolfman;
    // Man* pman = &wolfman;
    // 存在类型转换
    std::cout << "ptr: " << ptr << std::endl;
    std::cout << "pwlf: " << pwlf << std::endl;
    //std::cout << "pman: " << pman << std::endl;


    std::cout << "----------------------" << std::endl;
    void* ptr1 = &wolfman;
    Wolf* pwlf1 = (Wolf*)ptr1;  // void* => Wolf*
    //Man* pman1 = (Man*)ptr1;  // 指针   类型 和内存地址
    std::cout << "ptr1: " << ptr1 << std::endl;
    std::cout << "pwlf1: " << pwlf1 << std::endl;
    //std::cout << "pman1: " << pman1 << std::endl;

    std::cout << "----------------------" << std::endl;
    void* ptr2 = &wolfman;
    Wolf* pwlf2 = (Wolf*)&wolfman;  // void* => Wolf*  Wolfman*=>Wolf
    //Man* pman2 = (Man*)&wolfman;    // 类型
    std::cout << "ptr2: " << ptr2 << std::endl;
    std::cout << "pwlf2: " << pwlf2 << std::endl;
    //std::cout << "pman2: " << pman2 << std::endl;


    //_move = &wolfman; // 多重继承  编译器不知道怎么转，尽量避免多重继承
    std::cout << "----------------------" << std::endl;
    WolfMan wolfman1;
    wolfman1.::Wolf::MonsterObject::MoveObject::x = 2500;
    wolfman1.::MoveObject::x = 3500;
    MoveObject* _move1 = static_cast<MoveObject*>(&wolfman1);

    std::cout << _move1->x << std::endl;
}
```

强制类型转换都是有风险的！！！

**动态强制转换**

![](https://bu.dusays.com/2024/12/08/67555d8e3fb89.png)

```c++
// 33对象多态详解.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//

#include <iostream>
class MoveObject
{
public:
    int x;
    virtual void test() { }
    virtual void Move()
    {
        //if (dynamic_cast<MonsterObject*>(this))
        // 不推荐写法，非常依赖派生类
    }
};

/*virtual*/
class MonsterObject :public MoveObject
{
public:
    void Move() {};
};

class NPCObject :public MoveObject
{
public:
    void Move() {};
};


class Wolf :public MonsterObject
{};

class Man :public MonsterObject
{};

class boss
{

};

class WolfMan : public Man//,public boss
{};



int main()
{

    // 无论 向上转还是向下转，推荐无损转，使用指针或者引用来转
    MonsterObject monster;
    MoveObject* _move =(MoveObject*)&monster;
    MoveObject& lMove = monster;

    
    //_move = new MoveObject();
    //MonsterObject* _pmonsterA = dynamic_cast<MonsterObject*>(_move);
    //auto _pmonsterA = dynamic_cast<MonsterObject*>(_move);
    //MonsterObject和NPCObject之间没有继承关系，_move不能被转换为NPCObject*。
    //auto _pmonsterB = dynamic_cast<NPCObject*>(_move);
    
    //if (_pmonsterA != nullptr) _pmonsterA->MonsterMove();
    //if (_pmonsterB != nullptr) _pmonsterB->NPCMove();
    
    // 直接这样使用就可以了，避免过度使用 dynamic_cast
    //_move->Move();

    // 不应该转换  没有空引用 
    //NPCObject& lNPC = dynamic_cast<NPCObject&>(monster);
    
    //没有关系的类转换   跨类转换
    WolfMan wlfman;
    MoveObject* pMove = &wlfman;
    auto p = dynamic_cast<WolfMan*>(pMove);
    auto p1 = dynamic_cast<boss*>(pMove);

    std::cout << "p: " << p << std::endl;
    std::cout << "p1: " << p1 << std::endl;
}
```



**抽象类**

```c++
#include <iostream>

// 拥有纯虚函数的类称为抽象类，因为该类的函数没有实现，因此不能创建抽象类的实例
// 但是可以使用抽象类的指针和引用作为返回或参数！！
// 抽象类的构造函数因为不能实际使用，所以一般推荐把抽象类的构造函数定义为protected
// 抽象类的派生类如果没有定义纯虚函数，则该派生类依然是抽象类
class Animal
{

    void virtual Move() = 0;  // 这个虚函数没什么实际内容，就设计成纯虚函数
    void virtual fly() = 0;
    void virtual Notice() = 0;  // 发送通知
    void virtual Eat() = 0;

    // 一个类里面都是纯虚函数，没有提供具体内容，这就是接口类

protected:
    Animal() {}  // 抽象类的构造函数因为不能实际使用，所以一般推荐把抽象类的构造函数定义为protected
};

class Dog :public Animal
{
    void virtual Move()
    {

    }
    // 这里没有fly函数的，所以这个类也是抽象类
};

class Cat :public Animal
{
    void virtual Move()
    {

    }
};


//Animal GetAnm() { }  不能创建实体

int main()
{
    // 拥有纯虚函数的类称为抽象类，因为该类的函数没有实现，因此不能创建抽象类的实例
    //Animal anm;
    //Animal* anm1 = new Cat(); // 指针可以，实体不行

}
```

**类的成员函数的指针**

```c++
// 类的成员函数的函数指针.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//

#include <iostream>

// 这里限定了类Wolf
class Wolf;
typedef void (Wolf::* PGROUP)();


class Wolf
{
public:
    static void Count()
    {

    }

    Wolf()
    {
        pGroup = &Wolf::Group0;
        (this->*pGroup)();
    }

    void Group0()
    {
        std::cout << "一阶段！！！" << std::endl;
    }
    void Group1()
    {
        std::cout << "二阶段！！！" << std::endl;
    }
    void Group2()
    {
        std::cout << "三阶段！！！" << std::endl;
    }

    PGROUP pGroup;
};

typedef void(*COUNT)();

int main()
{
    PGROUP pFunction = &Wolf::Group2;
    Wolf* pWolf = new Wolf();
    (pWolf->*pFunction)();  // 成员函数的函数指针

    COUNT _count = &Wolf::Count;
    _count();
}
```

```c++
class Wolf; // 类的前置声明
// 定义了一个新类型PGROUP，它是指向Wolf类的成员函数指针，指向的成员函数没有参数且返回类型为void
// PGROUP 是一个类型别名，用于指向Wolf类中没有参数且返回类型为void的成员函数。
typedef void (Wolf::* PGROUP)();  
```

```c++
Wolf()
{
    pGroup = &Wolf::Group0;  // 构造函数中初始化pGroup
    (this->*pGroup)();        // 调用pGroup指向的成员函数
    PGROUP pGroup;  // 成员函数指针，指向Wolf类的成员函数
}
```

```c++
// COUNT是一个类型别名，用于定义指向没有参数且返回类型为void 普通函数的指针
// COUNT 是一个指向无参数无返回值函数的指针。
typedef void(*COUNT)();
```

```c++
// main函数
int main()
{
    PGROUP pFunction = &Wolf::Group2;  // 定义一个成员函数指针，指向Group2
    Wolf* pWolf = new Wolf();  // 创建一个Wolf对象（会调用构造函数）
    (pWolf->*pFunction)();  // 通过成员函数指针调用Group2

    COUNT _count = &Wolf::Count;  // 定义一个普通函数指针，指向静态成员函数Count
    _count();  // 调用静态成员函数Count
}
```

成员函数指针：指向类成员函数时，需要使用::*语法并制定类名。在调用的时候，必须通过类实例（this指针）来调用。

静态成员函数：静态成员函数不依赖于对象实例，可以像普通函数一样直接通过类名或函数指针调用。

成员函数与普通函数指针的不同：成员函数指针需要对象实例来调用，而普通函数指针则不需要。

**多态：虚函数的实现**

（面试可能问到）

```c++
// ConsoleApplication1.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
#include <iostream>
#include <Windows.h>

void Hack()
{
    std::cout << "劫持了！！！1" << std::endl;
}
class AIM
{
public:
    int HP;
    virtual void Eat()
    {
        std::cout << "AIM" << std::endl;
    }
    virtual void Die()
    {
        std::cout << "AIM_die" << std::endl;
    }
};

class Wolf :public AIM
{
public:
    virtual void Eat()
    {
        std::cout << "Wolf" << std::endl;
    }

    virtual void Die()
    {
        std::cout << "WOLF-DIE" << std::endl;
    }

    void Sound()
    {
        std::cout << "aoaoaoaoa~~~~" << std::endl;
    }
};

int main()
{
    AIM* wolf = new Wolf();
    wolf->Die(); // 调用的是Wolf类中的函数
    
    // 输出8   多了4个字节
    std::cout << sizeof(AIM) << std::endl;

    // 测试下多出的四个字节是头部还是尾部
    std::cout << wolf << " " << &wolf->HP << std::endl; // 加在头部了

    unsigned* vtable = (unsigned*)wolf;
    std::cout << std::hex << "Vtable:  " << vtable[0] << std::endl;

    unsigned* func = (unsigned*)vtable[0];

    std::cout << std::hex << "eat: " << func[0] << std::endl;
    std::cout << std::hex << "die: " << func[1] << std::endl;
    
    
    // 通过修改虚函数表的数据可以实现劫持
    DWORD old;
    VirtualProtect(func, 8, PAGE_EXECUTE_READWRITE, &old);
    func[0] = (unsigned)Hack;
    func[1] = (unsigned)Hack;
    wolf->Eat();
    // 以上只是影响的Wolf类  

    AIM* _aim = new AIM();
    _aim->Eat();

    AIM* _aim1 = new Wolf();
    _aim1->Eat();

	// 只有通过指针访问函数才会调用虚函数表
    Wolf wl;
    wl.Sound();  // 这个不涉及指针，能够调用我们想要的函数
    wl.Eat();
}
```

![](https://bu.dusays.com/2024/12/09/67568e837d723.png)

 虚表的性质

1️⃣ 同一个类的多个实例都指向同一个虚函数表

2️⃣ 通过修改虚函数表的数据可以实现劫持

3️⃣ 只有通过指针访问函数才会调用虚函数表

