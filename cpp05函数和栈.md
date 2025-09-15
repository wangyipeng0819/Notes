---
title: cpp05函数和栈
date: 2024-12-11 13:57:50
categories:
- Cpp
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686c9d1fd91fc.png
---

栈的本质是一段提前分配好的内存空间，主要就是用来存放临时变量！只需要管理好栈的读写就可以避免频繁的内存分配和不必要的内存浪费！

```C++
int Ave(int a, int b)
{
    a = a + 250;
    return a + b;
}
int Add(int a, int b)
{
    int c = 250;
    int d = Ave(a, b);
    c = c + d;
    return c;
}
int main()
{
    std::cout << Add;
    system("pause");
    int x = Add(250, 50);
}
```

CPU 的寄存器：







`return`  的值 一般使用 eax 来传递

![](https://bu.dusays.com/2024/11/27/6747130a5ebff.png)

![](https://bu.dusays.com/2024/11/27/674713b137197.png)

