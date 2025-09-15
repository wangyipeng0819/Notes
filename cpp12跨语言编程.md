---
title: cpp12跨语言编程
date: 2024-12-11 14:01:43
categories:
- Cpp
tags:
- 笔记
cover: https://bu.dusays.com/2025/07/08/686c9d1fd91fc.png
---

### C++ 调用汇编代码

**使用汇编代码的时机和意义**

![](https://bu.dusays.com/2024/12/09/675697d600c3a.png)

**Microsoft c++ x86 内联汇编**

![](https://bu.dusays.com/2024/12/09/675697d600c3a.png)

![](https://bu.dusays.com/2024/12/09/67569ae9428a8.png)

![](https://bu.dusays.com/2024/12/09/67569ae99537e.png)

```c++
#include <iostream>

int main()
{
    int a = 250;

    // 不需要加分号   使用__asm来分割
    __asm mov eax,a
    __asm add eax,1
    __asm mov a,eax

    std::cout << a;  // 251
    // 这样省略了很多__asm
    __asm
    {
        mov eax, a
        add eax, 1
        mov a, eax
    }

    std::cout << a;  // 252
}
```

**__asm 中汇编使用规范**

_asm支持

Pentium 4 和 AMD Athlon 所有操作码

支持MMX指令集

可以利用_emit创建目标处理器支持的其他指令

如果_emit生成修改寄存器的值，编译器无法确定哪些寄存器受到影响，这个时候编译器容易做出错误的判断，程序可能产生不可预测的行为。

asm 与  段引用  __asm中必须通过寄存器来引用段  不能通过段名称来访问

![](https://bu.dusays.com/2024/12/09/6756a0231248e.png)

```c++
int a[1000];
int count;
int arySize;
int typeAry;
_asm 
{
    mov count,LENGTH a
    mov arySize,SIZE a
    mov typeAry,TYPE a
}

std::cout << "元素个数为： " << count << std::endl;  // 1000
std::cout << "元素大小为： " << arySize << std::endl; // 4000
std::cout << "元素大小为： " << typeAry << std::endl;  // 4
```

**__asm的调试**

/Zi 编译选项 可以使用源代码调试内联汇编程序，可以在C/C++/汇编代码上设置断点

把多条汇编语言放在同一行可能会妨碍调试

**__asm中 C/C++使用规范**

![](https://bu.dusays.com/2024/12/09/6756a02410305.png)

**__asm代码优化和寄存器的注意事项**

_fastcall 不建议包含_asm代码

esp、ebp跟栈有关系，不能随便改

**_declspec(naked)**