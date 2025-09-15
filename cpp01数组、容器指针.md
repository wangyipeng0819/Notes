---
title: cpp01数组、容器指针
date: 2024-12-11 13:54:17
categories:
- Cpp
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686c9d1fd91fc.png
---

**数组**：

std::array中，可以通过at()来访问数组的内容，如果越界了就会抛出越界异常 



数组安全：由于数组的本质是向操作系统申请了一块内存，因此越界的数组将会访问到不该访问的地址，这种越界将会造成程序崩溃，BUG错误，更可怕的是，数组越界漏洞，可能会让攻击者拿到操作系统的控制权。

**容器**：

```c++
std::vector<数据类型> 变量名;
std::vector<int> s;
std::vector<int> s{1,2,3};
std::vector<int> s(5);  // 设置了5个大小的容器
std::vector<int> s(5,100);  // 这个容器拥有五个元素，每个元素的初始值为100
```

容器的几个新用法：

```c++
std::vector<int> s;
s.push_back(值);  //将值添加到容器末尾
s.pop_back();  //将末尾的值删除掉
s.insert(s.begin()+2, 2); // 在指定位置插入元素
s.assign(10,100); //将s重新初始化为拥有10个元素  每个元素位100的容器
s.erase(s.begin() + 2);   // 删除指定位置的元素
s.clear(); // 将容器清空
s.empty(); //看看是不是空的
```

**容器例子**

```c++
// vector 遍历
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    for (auto v : vec) {
        std::cout << v << " ";  // 输出元素值
    }
    std::cout << std::endl;

    return 0;
}

/*-------------------------------------------------------------------*/
// 使用引用避免拷贝
#include <iostream>
#include <vector>

int main() {
    std::vector<std::string> vec = {"Alice", "Bob", "Charlie"};

    for (auto& name : vec) {  // 使用引用避免拷贝
        name += " (modified)";
    }

    for (const auto& name : vec) {  // 使用常量引用只读访问
        std::cout << name << std::endl;
    }

    return 0;
}

// 遍历 map 容器
#include <iostream>
#include <map>

int main() {
    std::map<std::string, int> m = {{"Alice", 25}, {"Bob", 30}, {"Eve", 35}};

    // c++17及以上才可以
    for (const auto& [key, value] : m) {  // 使用结构化绑定
        std::cout << key << ": " << value << std::endl;
    }
    return 0;
}

// 遍历数组
#include <iostream>

int main() {
    int arr[] = {10, 20, 30, 40, 50};

    for (auto num : arr) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    return 0;
}

// 遍历unordered_map
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> um = {{"One", 1}, {"Two", 2}, {"Three", 3}};

    for (const auto& pair : um) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}
```

**结构化绑定**

```c++
// std::pair  通过first second 来访问pair成员
#include <iostream>
#include <map>

int main() {
    std::pair<int, std::string> p = {1, "Alice"};

    // 使用结构化绑定
    auto [id, name] = p;

    std::cout << "ID: " << id << ", Name: " << name << std::endl;
    // 输出  ID: 1, Name: Alice

    return 0;
}
```

```c++
// std::tuple
#include <iostream>
#include <tuple>

int main() {
    std::tuple<int, double, std::string> t = {1, 3.14, "Hello"};

    // 使用结构化绑定
    auto [a, b, c] = t;

    std::cout << a << ", " << b << ", " << c << std::endl;
    return 0;
}
```

```c++
//遍历std::map
#include <iostream>
#include <map>

int main() {
    std::map<std::string, int> m = {{"Alice", 30}, {"Bob", 25}, {"Charlie", 35}};

    for (const auto& [key, value] : m) {  // 解构键值对
        std::cout << key << ": " << value << std::endl;
    }

    return 0;
}
```

```c++
// 解构数组
#include <iostream>

int main() {
    int arr[3] = {10, 20, 30};

    auto [x, y, z] = arr;  // 解构数组
    std::cout << x << ", " << y << ", " << z << std::endl;

    return 0;
}
```

解构用户自定义类型

```c++
#include <iostream>
#include <string>

struct Person {
    int id;
    std::string name;
    double salary;
};

int main() {
    Person p = {101, "Alice", 75000.0};

    auto [id, name, salary] = p;  // 解构绑定
    std::cout << "ID: " << id << ", Name: " << name << ", Salary: " << salary << std::endl;

    return 0;
}
```

特殊应用：非聚合类型（涉及重载）

```c++
#include <iostream>
#include <tuple>

class Person {
private:
    int id;
    std::string name;
    double salary;

public:
    Person(int i, std::string n, double s) : id(i), name(n), salary(s) {}

    friend struct std::tuple_size<Person>;
    friend auto std::get<0>(const Person&) -> int;
    friend auto std::get<1>(const Person&) -> std::string;
    friend auto std::get<2>(const Person&) -> double;
};

// 特化 std::tuple_size 和 std::get
namespace std {
    template <> struct tuple_size<Person> : std::integral_constant<size_t, 3> {};
    template <> auto get<0>(const Person& p) { return p.id; }
    template <> auto get<1>(const Person& p) { return p.name; }
    template <> auto get<2>(const Person& p) { return p.salary; }
}

int main() {
    Person p(101, "Bob", 85000.0);

    auto [id, name, salary] = p;
    std::cout << "ID: " << id << ", Name: " << name << ", Salary: " << salary << std::endl;

    return 0;
}
```

![](https://bu.dusays.com/2024/11/18/673ab200dc43b.png)

**指针**

 指针语法：数据类型 * 变量名称

```c++
//指针必须初始化  指针其实就是存储的别的变量的内存地址
int* a;  //指针是内存地址  建议第一种写法
int *b;  //两种写法

// 要操作的内存地址
// 要操作的内存大小
int* pa = &a;
```

**指针**

```c++
int a[]{ 10001,20001,30001,40001 };
int* ptr{ &a[0] };

//指针+1是 1 * 数据类型的大小
std::cout << ptr << std::endl;			//ptr的值就是a[0]的地址
std::cout << &a[0] << std::endl;
std::cout << *ptr << std::endl;			//*ptr就是a[0]的值
std::cout << (*ptr)++ << std::endl;		//先输出*ptr的值  为10001，然后对*ptr  加  1
std::cout << a[0] << std::endl;  		// a[0]变为10002
std::cout << "------------分界线----------------" << std::endl;

std::cout << ptr << std::endl;			//ptr的值就是a[0]的地址
std::cout << *ptr++ << std::endl;		//++的优先级高，先对ptr++，在对ptr做解引用的操作
std::cout << a[0] << std::endl;			// a[0]的值不变
std::cout << *ptr << std::endl;			//ptr进行加一操作之后，指向a[1],  输出20001
std::cout << ptr << std::endl;			//ptr的值增加了一个int类型的大小，4个字节
std::cout << "------------分界线----------------" << std::endl;

//指针的指针，指针也是个变量，本身也有地址，现在要操作ptr的内存
//不考虑前面进行的操作
int** pptr{ &ptr };										//这样定义就是指针的指针
std::cout << "数组a[0]的地址为： " << ptr << std::endl;
std::cout << "数组a[0]的地址为： " << &a[0] << std::endl;	//	这里ptr和&a[0]输出的相同
std::cout << "ptr的地址为：" << pptr << std::endl;		//这里输出ptr的地址
std::cout << *pptr << std::endl;			//*pptr解引用pptr，得到ptr的值，即a[0]地址
std::cout << **pptr << std::endl;		//**pptr进一步解引用，得到a[0]的值

*pptr = &a[1];
std::cout << *pptr << std::endl;		//使用*pptr把ptr的值改成a[1]数组的内存地址



//类似的，可以定义下面这种
int*** ppptr{ &pptr };
std::cout << ***ppptr << std::endl;
std::cout << "------------分界线----------------" << std::endl;
```

![](https://bu.dusays.com/2024/11/13/6734530db4262.png)

**常量指针**

```C++
//const 变量类型*  只能指向一个常量
// 特点，指针的指向可以修改，但是指针指向的值不可以修改
const int a{ 100 };
const int b{ 200 };
int c{ 300 };
const int* p{ &a };
// *p = 500;  不可以修改值
std::cout << *p << std::endl;
p = &b;  // 可以修改指向
p = &c;  // 常量指针也可以指向非常量变量
// *p = 2000;  常量指针就是不让改
```

**指针常量**

```C++
//特点：指针的指向不可以改，指针指向的值可以改（内存中的数据可以更改）
int a{ 100 };
int b{ 200 };
int* const p{ &a };
//p = &b;   不能修改指向
*p = 999;  //可以修改指针指向的内存空间
std::cout << a << std::endl; //  这时候a输出 999；
```

**指向常量的常量指针**

```C++
//const 变量类型* const
//特点：指针的指向和指针指向的值都不可以修改 
const int a{ 100 };
const int b{ 200 };
const int* const p{ &a };
//p = &b;  不能修改指向
//*p = 999;  不能修改内存空间里面的值
```

**补充（指针有关的类型转换 ）**

```c++
const int a{ 100 };
const int b{ 200 };
//int* pa{ &a };   常量
int* pa{ (int*)&a };   //通过转换是可以的
*p = 9500;	//这种操作是允许的，但是a的值不变
std::cout << *pa << std::endl;
```

**指针数组补充**

指针数组是**一个数组**，数组的每个元素都是指针。

```c++
int* p[5];
```

 p 是一个包含5个指针的数组。  每个指针可以指向一个int类型的变量。

```c++
int a = 10, b = 20, c = 30;
int* p[3];
p[0] = &a;
p[1] = &b;
p[2] = &c;

// 通过指针数组来访问变量。
printf("a = %d, b = %d, c = %d", *p[0], *p[1], *p[2]);
// p[0] 是第一个指针， *p[0] 则访问指针指向的内容。
```

数组指针 是**一个指针**，它指向一个数组。

```c++
int (*p)[5];
```

p 是一个指针，指向包含5个整型元素的数组。

```c++
int a[5] = {1, 2, 3, 4, 5};
int (*p)[5];
p = &a;   // p 指向 数组 a

// 这样来访问数组 
printf("First element: %d\n", (*p)[0]);
printf("Second element: %d\n", (*p)[1]);
```

数组指针是一个指针，它指向整个数组。

使用*p 解引用数组指针，可以获得数组本身， 然后可以通过下标操作访问数组元素。

数组指针在多维数组操作中非常常用。  





```c++
int a[5]{ 1,2,3,4,5 };
int* ptrA{ a };
int* ptrB{ &a[0] };   //从汇编语言得出，a放的就是a[0]的地址
int* ptrC{ a + 1 };

std::cout << ptrC[1] << std::endl;  //输出的 是3
std::cout << ptrA[1] << std::endl;  //也可以这样访问a[1]
std::cout << sizeof(a) <<std::endl;  //输出大小是20  处理a的时候把它当作数组处理
std::cout << sizeof(ptrA) <<std::endl; //输出大小是4
//底层来说a是个指针，明面上还是数组
//指针可以当数组来用，数组可以当指针来读
```

```c++
//多维数组在内存中不存在，只是我们告诉内存这么访问而已
int test[2][5]
{
	{1001,1002,1003,1004,1005},
	{2001,2002,2003,2004,2005}
};
int* ptest{ (int*)test };
std::cout << test[1][4] << std::endl;
std::cout << ptest[9] << std::endl;
```

```c++
int test[2][5]
{
	{1001,1002,1003,1004,1005},
	{2001,2002,2003,2004,2005}
};

int* ptestA[5]; 	//  指针数组，  5个int类型的指针
int (*ptest)[5]{test};	//  首先是一个指针，类型是数组指针，每一行能存储5个数据
ptest = ptest + 1;
std::cout << ptest << std::endl; //比原来增加了6 * int 的大小，因为ptest是个数组指针，逻辑是6
std::cout << ptest[0][1] << std::endl; //数组指针可以使用二维数组的方式访问数据
std::cout << sizeof(ptest) << std::endl;  //大小是4字节 ，所以ptest就是个指针，不是数组
```

**数组再补充**

```c++
int test[2][5]
{
	{1001,1002,1003,1004,1005},
	{2001,2002,2003,2004,2005}
};

std::cout << test[0] << std::endl; //这是个地址
//a + [] * int
```

**C语言内存分配**

```c
int* p = (int*)malloc(x * sizeof(int));
int* pa = (int*)calloc(x, sizeof(int));
std::cout << "------------------------" << std::endl;
int* p = (int*)malloc(4);
p = (int*)realloc(p, 8);  //重新分配了大小,原来输入的值还在
//原来分配了100，后面更改为50，那内存地址不会变
//原来分配了100，后面改成200，内存地址会变，但是会把原来的内存的内容拷贝到新的内存中

free(p);
free(pa);

//p 和 pa的值成了悬挂指针，
//所以free之后改成p = 0, pa = 0;
```

**C++内存分配**

```c++
//数据类型* 指针变量名称 = new 数据类型;
int* pa = new int;
//数据类型* 指针变量名称 = new 数据类型[数量];
int* p = new int[5];

---------------------------------------------------
p = new int[x];
*p = 500;
p[0] = 500;

//两种释放内存的方式，不要混用
delete pa;
delete[] p;


--------------------------------------------------
//内存复制
int a[5]{1001,1002,1003,1004,1005};
int* p = new int[5];
memcpy(p,a,5*sizeof(int));

//设置内存
//memset 可以指定内存区域每一个字节的值都设置为val,
int* pa = new int[100];
//这里就是设置成0 了
memset(p,0,100*sizeof(int));

void* meeset(void* _dst, int val, size_t size)
// val范围是0x 00 --- 0x ff   如果设置0x1234，那么只会保存34
//memset可以将指定内存区域每一个字节都设置为val，_size为要设置的长度（字节）
```

使用动态内存分配风险

如果释放内存后没有清零，很危险，使用new可能会报错

重复释放

内存碎片

不推荐C语言和C++释放内存的语句混用

**引用**

```c++
//数据类型& 变量名称{引用对象的名称};
//引用类型一定要初始化，就相当于一个别名
int a{500};
int b{100};
int& la{a};
la = 500; // 相当于 a = 500;
//它们所有的地址都是一样的
//引用设置之后就不能更改了

la = b; // a 的值编程了100
```

**智能指针**

```c++
std::unique_ptr<int[]> intPtr{std::make_unique<int[]>(5)};  // 是有五个这种变量
std::unique_ptr<int> intPtrA{std::make_unique<int>(5)};     // 把指针初始化为5

intPtr.reset();  // 将指针地址清零，然后把内存空间还给操作系统

int* a = new int[5];
a = intPtr.get();  // 将intPtr的内存地址赋值给a

a = intPtr.release();  // 把指针设置为0，但还会范围占用内存的地址
```

**智能指针的转移**

```c++
std::unique_ptr<int[]> intPtr{std::make_unique<int[]>(5)}; 
std::unique_ptr<int[]> intPtrA{};  

intPtrA = std::move(intPtr);  // intPtr被清零了，失效了，intPtrA指向原来intPtr指向的位置
```

**共享智能指针**

```c++
int* a{};

//语法
std::shared_ptr<类型> 变量名称{};
std::shared_ptr<int> ptrA{}; // 设置为空
std::shared_ptr<int> ptrB{std::make_shared<int>(5)}; //将这个指针初始化为5
//std::make_shared不支持数组
std::shared_ptr<int[]> ptrC{new int[5]{1,2,3,4,5}};
std::cout << ptrA << " " << ptrC[0] << " " << *ptrB;
/*----------------------------------------------------*/
//如何共享
std::shared_ptr<int> ptrA{std::make_shared<int>(5)};
std::shared_ptr<int> ptrB{ptrA};
//两个指针可以指向同一个地方
std::cout << ptrB << " " << *ptrB << std::endl;
std::cout << ptrA << " " << *ptrA;

//假设有  A  B C  三个指针指向这块区域，只有当所有的指针都释放了，才会释放内存空间
// 会有一个计数器，来记录有几个指针指向这块区域
long std::shared_ptr.use_count(); // .use_count() 会返回当前指针共有多少个对象调用

std::cout << ptrA.use_count() << std::endl;

//会将当前共享指针设置为空，同时如果当前智能指针是最后一个拥有该指针的对象，那么将释放内存。
std::shared_ptr.reset();

ptrB.reset();
std::cout << ptrB << " ";//*ptrB使用不了，因为被释放了
```

![](https://bu.dusays.com/2024/11/16/67389b688486f.png)

**指针和结构体**

通过指针访问自定义数据类型（基础部分）

```c++
typedef struct Role
{
    int HP;
    int MP;
}* PRole;
// 上面不写*号相当于给Role改了个名字，写了*号就是类型指针的意思
//以前声明指针需要Role*    现在只需要PRole即可
int main()
{
    Role user;
    PRole puser = &user;
    
    //  使用指针偏移来访问数据
    puser->HP = 50;
    puser->MP = 100;
    user.HP = 50;
    user.MP = 50;
    std::cout << (*puser).HP << std::endl;
    std::cout << puser->HP << std::endl;
}
```

**内存对齐问题**

```c++
typedef struct Role
{
    char op;
    int HP;
    int MP;
    short x;
    short x1;
}* PRole;

int main()
{
    Role user;
    PRole puser = &user;
    std::cout << sizeof(Role);
    // 注销掉 x1 和不注销 x1 的结果是一样的，这里涉及到内存对齐问题。
}
```

内存对齐是指在内存中存储数据时，数据的起始地址按特定的规则对齐，而不是任意的内存地址。

- 目的：提高 CPU 访问内存的效率，因为现代 CPU 通常按字节、字或更大单位（如 4 字节或 8 字节）读取内存。
- 结果：可能会在数据之间引入填充字节（padding），以满足对齐规则

对齐原则

每个成员的地址必须是**对齐系数**的整数倍。

- 对齐系数 = **成员大小** 或 **默认对齐数**，取较小值。

结构体总大小必须是**结构体最大对齐系数**的倍数。

<u>指定几个字节对齐</u>

```c++
#pragma pack(2)  // 指定 2 字节对齐
typedef struct
{
    char a;
    int b;
    short c;
}PackedExample;
#packma pack(); // 恢复默认对齐

//直接输出这个结构体占据的大小
printf("sizeof(PackedExample): %zu\n", sizeof(PackedExample));
```

**指针安全**

悬挂指针和野指针

1. **悬挂指针**: 悬挂指针产生于指针所指向的内存已被释放或者失效后，指针本身没有及时更新或清空。在该内存释放之后，任何通过这个悬挂指针的引用或操作都是不安全的，因为这块内存可能已经重新分配给了其他的数据。

   示例：当一个指针指向动态分配（比如使用`malloc`或`new`）的内存，并且随后该内存被释放掉（使用`free`或`delete`），而没有将指针设置为`NULL`，此时这个指针就变成了悬挂指针。

2. **野指针**: 野指针通常是指未初始化的指针，它没有被设置为任何有效的地址。由于它可能指向任意位置，对野指针的解引用是危险的，并且可能会导致难以预测的行为甚至程序崩溃。

   示例：声明了一个指针变量但是没有给它赋予确定的初始值，然后就开始使用这个指针。

尽管两者看似相似，但是产生原因和解决方式有所不同：

- **悬挂指针问题**可以通过确保指针在释放关联的内存资源后立即被设为`NULL`来避免。
- **野指针问题**则需要确保每个指针变量在使用前都被明确初始化为一个合法的地址或`NULL`。

处理这两种类型的指针时，编程中的最佳实践是始终确保你的指针在声明后得到适当的初始化，在资源被释放之后更新状态，并且在解引用之前检查其有效性。



指针存在的俩问题：

1️⃣ 指针没有了，内存空间还在

2️⃣ 内存空间释放了，指针还有

野指针（悬挂指针）

```c++
// 野指针
int main()
{
    int* p;
    {
        int* a = new int[5];
        p = a;
        a[2] = 555;
    }
    std::cout << p[2]; // 内存空间还在，没有释放
    // 这就是野指针  （悬挂指针）
}
```

改正方法：使用 智能指针

```c++
int* p;
{
    std::unique_ptr<int[]> a{ std::make_unique<int[]>(50) };
    a[2] = 250;
    p = a.get();
    std::cout << p[2];  // 输出250
} // 括号结束后，内存空间会释放掉

std::cout << p[2]; // 输出-572662307,  说明内存空间被释放了
```

另外一种情况，存在栈里面，括号  之后栈空间没有被回收（如果分配到函数里面，那么就不会出现这种问题）

```c++
int* p;
{
    int a[5]{1, 2, 3, 4, 5};
    p = a;
    std::cout << p[0] << std::endl;  // 输出1
}
std::cout << p[0] << std::endl;  // 输出1
```

内存空间释放了，但是指针还在

```c++
int* p;
{
    int* a = new int[5];
    p = a;
    std::cout << p[0] << std::endl;
    delete[] a;
}
std::cout << p[0];   // p指针还在
```

补充知识：.get()函数

1️⃣std::shared_ptr` 和 `std::unique_ptr    在智能指针中，.get() 用于获取所管理的原始指针。返回指正指针内部所管理的原始指针，但不会更改其所有权或者生命周期管理。

2️⃣在输入流(std::istream)中，.get()用于从输入流中读取字符。

```c++
// 程序示例
#include <iostream>
#include <sstream>

int main()
{
	std::istringstream input("HEllo World");
	char c;
	while (input.get(c))  std::cout << c;
	
	return 0;
}
```

**堆和栈**

堆的本质就是空闲内存，C++中把堆称为自由存储区，只要是你的程序加载后，没有占用空闲的内存，都是自由存储区，我们用new或者malloc申请的一块新内存区域，都是操作系统从堆上操作的。

栈是程序编译时就已经确定大小的一段内存区域，主要是用于临时变量的存储，栈的效率高于堆，但是容量有限.

**汇编知识（补充）**

在汇编语言中，`LEA`（Load Effective Address）指令用于将一个内存地址加载到寄存器中，而不是直接访问该内存地址的值。简单来说，`LEA` 计算内存地址的有效值，并将其存入目标寄存器，而不是从该内存地址加载数据。