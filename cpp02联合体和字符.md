---
title: cpp02联合体和字符
date: 2024-12-11 13:55:52
categories:
- Cpp
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686c9d1fd91fc.png
---

**联合体语法**

```c++
//示范
union USER
{	
    short sHP;
    int nHP;
};
// 通过union 可以创建一个联合体，union中的成员变量共享内存，因此union的数据类型大小由其最大的成员变量决定。
```

实践一下

```c++
// 联合体

#include <iostream>

union USER
{
	short sHP;
	int nHP;
	double fHP;
};

union {
	short sHP;
	int nHP;
	double fHP;
} ls;// 不重用，临时的 

typedef struct Test {
	short sHP;
	int nHP;
	double fHP;
};

int main()
{
	// 内存分配   没有初始化
	USER user;
	Test t;
	std::cout << sizeof(user) << std::endl; 
	std::cout << sizeof(t) << std::endl;  



	user.sHP = 100;//[100][0][][][][][][]
	std::cout << user.sHP << std::endl;  // 100

	//[100][0][？][？][？][？][？][？]
	// 访问4个格子，但有俩没有初始化，不知道是什么值
	std::cout << user.nHP << std::endl;  
	user.nHP = 0;
	std::cout << user.nHP << std::endl;
}
```

未命名的联合体是供临时使用的！

**C 语言中 字符串的拼接**

```c
char str[0x10] = "123";
char strB[0x10] = "456";

char strC[0x20];
memcpy(strC, str, strlen(str));
memcpy(strC + strlen(str), strB, strlen(strB) + 1);

std::cout << strC;
```

**C++ 中 字符串拼接**

```c++
string str{"12345"};
// 不必考虑溢出
std::cout << str << std::endl;

string str1{ "123455", 3 }; // 只要前面三个字符串
std::cout << str1 << std::endl;  // 输出123


string str2{ "0123456",2,3 };  // 从2开始截取3位
std::cout << str2 << std::endl;
// 对中文支持的不太好
string str3{ "你好啊啊啊", 3 };
std::cout << str3 << std::endl;

string str4(6, 'a');  // 6个  a
string str5(6, 65);  //转成大写的A
std::cout << str4 << std::endl;
std::cout << str5 << std::endl;


// 连接字符串
string str6, str7;
str6 = "123";
str7 = "456";

str6 = str7 + " " + "123";  // 这里一定要有string 类型的变量在这里
std::cout << str6 << std::endl;

// 把数字转换成字符串
int age;
std::cin >> age;

string st;
st = " 用户的年龄是：";
str7 = st + std::to_string(age);
std::cout << str7;
```

**C++ 中字符串拼接（进阶）**

```c++
#include <iostream>
using std::string;

#define SoftName "EDY"
#define SoftVersion "2.0"

int main()
{
	string str;

	//str = "1233" + "1222";
	// 不能直接连接两个字符串
	str = string{ "!22" } + "1233";  // 把字符串放到临时变量里面就可以了

	 //str = "1233" + "bcbc" + string{ "!22" };   error
	// 前面两个加法是直接的字符串的字面量
	//改进
	str = "abc" + ("bcd" + string{ "123" }); // 这样就可以了

	str += "aka";  // 没问题
	//str += "aka" + "bka";    +的优先级大于   +=
	// 解决办法如下
	str += "aka" + string{ "bka" };


	string strB;
	// 连接
	strB = "!23""233";
	// 唯一用途
	strB = SoftName SoftVersion;
    
    // 字符串连接  字符
    string strC{"123"};
    char a;
    std::cin >> a;
    str += a;  // 允许这样连接
    // 注意事项
    str += a + 'o';  // 两个字符  是俩常量，可以相加    字符a 与 字符o相加之后再连接
    // 相当于
    str += char(a + 'o');
    // 字符串不能相加的原因是， 字符串是两个char类型的数组
    
    
    //拼接方法
    string strD{"123"};
    strD.append("456");  // 在字符串后面拼接上456
    strD.append("222").append("dfddd");  // 无限  加append
    // 拼接的时候也可以有选择
    strD.append("123455", 2); // 从2开始拼接   跟字符串初始化用法一样
}
```

**截取字符串**

```c++
.substr(起始位置,要截取的长度);

std::string str{"123456"};
std::string strsub{str.substr(1)};  // strsub = "23456"
std::string strsubA{str.substr(1, 3)}; // strsubA = "234";

string strB;
// 也可以连着截取，但是要放到一个新的string里面
strB = str.substr(7).substr(3);  // 提取从索引7开始的子字符串（包括索引 7 的字符）， 直到字符串末尾。.substr(3) 在上一步的基础上，从索引 3 开始提取子字符串 （包括索引3 的字符）， 直到字符串末尾。
// 字符串的索引也是从0开始的
// substr不会改变str的值，而append会改变
```

**计算字符串长度**

```c++
string str{"222222"};
// 对中文字符串长度的计算还是不准确
std::cout << str.length();
```

下面是一个正常计算字符串的方法

```c++
#include <iostream>
#include <locale>
int main() {
    
    // 设置区域为中文   确保宽字符输入/输出流能正确处理中文字符。
    setlocale(LC_ALL, "chs");
    wchar_t wstr[255]; // 定义宽字符数组
    std::wcout << L"请输入一个字符串: ";
    std::wcin.getline(wstr, 255); // 使用宽字符输入

    int length = 0;
    for (int i = 0; wstr[i] != L'\0'; ++i) { // 遍历宽字符数组
        ++length;
    }

    std::wcout << L"字符串的长度是: " << length << std::endl;
    return 0;
}
```

**字符串比较**

1️⃣ 比较相同的内容

```c++
char* str_1{(char*)"123456"};
char* str_2{(char*)"123456"};

std::cout << (str1 == str2) << std::endl;  // 地址相等，进行优化之后指向相同的位置
```

```c++
char* str_1{(char*)"123456"};
char* str_2 = new char[7];
std::cin >> str_2;

std::cout << (str1 == str2) << std::endl;  // 输入123456  是不相等的
```

```c++
string str_1{"123456"};
string str_2;

std::cin >> str_2;
std::cout << (str1 == str2) << std::endl;   // 手动输入123456   是相等的
```

2️⃣ 比较不同的内容

```c++
string str_1{"abcdef"};
string str_2{"bcdefg"};

// 从第一个字符开始比较，如果比出来大小后面就不比较了
if(str_1 > str_2)
{
    std::cout << "大于";
}
else std::cout << "小于等于";
```

```c++
string str_1{"abcdef"};
str1.compare("bcdefg"); // 返回一个int类型的值
// 如果str1比另外一个字符串小返回负数 -1， 如果相等返回 0  如果大于返回 1
```

3️⃣ 截取一段之后进行比较

```c++
string strA{"abc cdef"};
strA.compare(5,4,"cdef"); // 截取的是strA   compare(起始位置,参与比较的长度,被比较的字符串);
```

```C++
string strA{"abc cdef"};
strA.compare(5,4,"cdef ghijklm", 0, 4); // 前面截图的strA， 后面截取的是"cdef ghijklm"
```

```c++
string strA{"username:5620;studentId:655555"};

std::cout << strA.find("studentId:"); // 返回字符串的起始位置
std::cout << strA.substr(strA.find("studentId:")); // 输出  studentId:655555
std::cout << strA.substr(strA.find("studentId:") + 10); //输出655555
```

补充  find()  方法 的用法

![](https://bu.dusays.com/2024/11/21/673ee8f79c446.png)

```c++
str.find("sdajklfdkasjlkfjlas", 0, 10); //在字符串 str 中，从位置 0 开始，查找字符串 "sdajklfdkasjlkfjlas" 的前 10 个字符是否存在。
```

倒着搜索

```c++
// 表示从字符串 str 的 索引 0 位置向前（从右到左查找）寻找目标字符串 "sdajklfdkasjlkfjlas"
str.rfind("sdajklfdkasjlkfjlas", 0); 
```



**字符串的应用--小项目**

![](https://bu.dusays.com/2024/11/21/673ef4d76891d.png)

```c++
#include <iostream>
#include <string>
using std::string;
int main()
{
	string str{ "id=user;pas=632105;role=阿森;" };
	string strIn;
	string strOut;
	while (true)
	{
		std::cout << "请输入您要查阅的属性：\n";
		std::cin >> strIn;
		int lfind = str.find(strIn + "=");
		if (lfind == std::string::npos)
			std::cout << "查找的属性不存在";
		else
		{
			int lend = str.find(";", lfind);
			strOut = str.substr(lfind + strIn.length() + 1, lend - lfind - strIn.length() - 1);
			std::cout << strOut << std::endl;
		}
	}
}
```

**插入字符串**

```c++
// 语法
.insert();
string id{"id=;"};
id.insert(3,"testId");
std::cout << id;
```

![](https://bu.dusays.com/2024/11/21/673f1c5e64bea.png)

```c++
// 方法二
string id{"id="};
// 从索引插入6个*
id.insert(3,6,'*');
std::cout << id << std::endl;

id.insert(3, "testid");
std::cout << id;
```

```c++
string id{"id="};
// 3 是从id字符串索引3开始插入，后面的第一个6是从字符串"killertestid"索引6开始插入6个字符到字符串id中
id.insert(3,"killertestid", 6, 6);
std::cout << id << std::endl;
```

```c++
string id{"id="};
// 在id字符串索引3开始插入 ，  然后从字符串"killertestid123456"的k开始插入6个
id.insert(3,"killertestid123456",6);
```

**指针数组字符串**

str和 str[0] 不是一个位置，具体跟数组有区别，涉及到运算符重载

str的地址和str[0]的地址差4

```c++
string str{"12344"};
// 不转int会当作字符串来输出
std::cout << (int)&str << " " << (int)&str[0] << " " << (int)&str[1];
```

```c++
string str{"12345"};
std::cout << str[0] << std::endl;

std::cout << (int)&str << " " << (int)&str[0] << " " << (int)&str[1] << std::endl;
str = str + "534894539823478923";
// str  跟 str[0]地址相差就很多了，需要重新分配内存地址
std::cout << (int)&str << " " << (int)&str[0] << " " << (int)&str[1] << std::endl;

// .c_str() 返回的是 const char*，不能直接通过指针修改字符串内容。
const char* baseStr = str.c_str();
// baseStr[0] = '5';   不允许修改
std::cout << (int)&str << " " << (int)&str[0] << " " << (int)&str[1] << " " << (int)baseStr << std::endl;


// C++ 17 之后 .data()方法得到的是一个char* 的指针

const char* dataStr = str.data();
// dataStr[0] = '5';   不允许修改
std::cout << (int)&str << " " << (int)&str[0] << " " << (int)&str[1] << " " << (int)baseStr << std::endl;
```



- 返回一个 `const char*` 指针，指向以空字符 `\0` 结尾的 C 风格字符串。
- 返回的字符串数据与 `std::string` 对象内部保持一致，但不能修改返回值（它是 `const` 的）。

**字符串替换**

```c++
// 语法1
.replace(要替换的内容的起始位置，要替换的长度，“替换后的内容”);

string strId{"id=user;"};
// 替换从索引 3 开始的 4 个字符为 "zhangsan"
strId.replace{3,4,"zhangsan"};
std::cout << strId;  // id=zhangsan

// 语法2
.replace(要替换的内容的起始位置，要替换的长度，替换后的字符长度，‘字符’);
// 替换从索引 3 开始的 4 个字符，用 6 个 '*' 替换
str.replace(3,4,6,'*');
// str的内容为：id=******


// 语法3
.replace(要替换的内容的起始位置，要替换的长度，"替换后的内容"，替换后内容的长度节选);
// 替换从索引 3 开始的 4 个字符，用 "zhangsan!pkaq" 的前 8 个字符替换
str.replace(3,4,"zhangsan!pkaq",8);

// 语法4
.replace(要替换的内容的起始位置，要替换的长度，"替换后的内容"，替换后的内容的起始位置，替换后内容的长度节选);
// 替换从索引 3 开始的 4 个字符，用 "zhangsan!pkaq" 的从第9个字符开始截取，截取4个
str.replace(3,4,"zhangsan!pkaq",9，,4);
```

**删除字符串内容**

```c++
// 语法1
.erase(要删除的起始位置,要删除的起始长度);

string str{"id=user;"};
str.erase(3,4);
// id=;

// 语法2
.erase(要删除的起始位置); // 从起始位置开始删除所有内容
string str{"id=user;"};
str.erase(3);
// id = 

.erase();   //删除字符串所有内容   擦除
.clear();   //删除字符串所有内容   清零
```

**字符串（补充）**

计算字符串长度

```c++
string strIn;
std::cin >> strIn;

int length{0};
for (int i = 0; strIn[i]; i ++ )
{
    // 在 ASCII 编码中，单字节字符（如英文、数字）均为非负数（0-127）
    if (strIn[i] < 0) i ++ ;
    length ++ ;
}
// 这样就能统计出字符个数了
std::cout << length;
```

字符串转换

``` c++
string cIn = "123";
int x = std::stoi(cIn); // 转换成int类型
```

![](https://bu.dusays.com/2024/11/21/673f31c813942.png)

字符串流

```c++
#include <iostream>
#include <string>
#include <sstream>
using std::string;
using std::stringstream;

int main()
{
    stringstream strS;
    // 这是按照16进制来输出的
    strS << "你好" << "123 [" << std::hex << 12530 << "]";
    string strX = strS.str();
    std::cout << strX;
}
```

**字符串项目**

```c++
#include <iostream>
#include <string>
using std::string;

typedef struct Role
{
    string Id;
    int Exp;
}*PROLE;

int main()
{
    string strData = "id=Tomy Clare;exp=9523;id=Sunny;exp=9523;id=DyBaby;exp=25301;id=Simple;exp=25301;id=Bkacs11;exp=2100;id=Dudu;exp=2122;";

    int istart{}, iend{}, icount{};
    for (int i = 0; i < strData.length(); i++)
    {
        if (strData[i] == ';')
        {
            icount++;
            i += 3;
        }
    }
    PROLE pRole = new Role[icount / 2];
    icount = 0;
    do
    {
        // 查找到id=   则把istart 将更新为 "id=" 首次出现的索引
        istart = strData.find("id=", istart);
        if (istart == std::string::npos) break;

        iend = strData.find(";", istart + 3);
        // 从istart开始截取，截取长度就是iend-istart-3
        pRole[icount].Id = strData.substr(istart + 3, iend - istart - 3);
        istart = iend + 1;
        iend = strData.find(";", istart + 4);
        // 在这里 icount ++ 省去后面再++ 了

        pRole[icount++].Exp = std::stoi(strData.substr(istart + 4, iend - istart - 4));
        istart = iend + 1;
    } while (true);

    // 冒泡排序  经典冒泡排序
    for (int i = 0; i < icount - 1; i++) {
        for (int j = 0; j < icount - i - 1; j++) {
            if (pRole[j].Exp < pRole[j + 1].Exp ||
                (pRole[j].Exp == pRole[j + 1].Exp && pRole[j].Id > pRole[j + 1].Id)) {
                Role temp = pRole[j];
                pRole[j] = pRole[j + 1];
                pRole[j + 1] = temp;
            }
        }
    }

    for (int i = 0; i < icount; i++)
    {
        std::cout << pRole[i].Id << " " << pRole[i].Exp << std::endl;
    }
}
```