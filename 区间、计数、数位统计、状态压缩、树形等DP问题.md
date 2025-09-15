---
title: 区间、计数、数位统计、状态压缩、树形等DP问题
date: 2025-08-05 23:10:13
tags:
- 算法
categories:
- 数据结构
cover: https://bu.dusays.com/2025/08/05/6892229d3737e.png
---

## 区间DP

### AcWing 282. 石子合并





## 计数类DP

### AcWing 900. 整数划分





## 数位统计DP

### AcWing 338. 计数问题

| 数字 | 1    | 11   | 21   | 31   | 41   | 51   | 61   | 71   | 81   | 91   | 101  | 111  | 231  | 991  |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| ans  | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   | 11   | 12   | 24   | 100  |

发现一个规律：假设x的每一位分别为abcdefg，那么当g == 1时，我们的答案就是abcdef + 1

假如g < 1的，那最后就不会统计到abcdef1这个数字，答案就是abcdefg

如果g > 1，那么答案也是abcdef + 1

所以统计个位数的规律我们可以视为 `n/10+(n%10>=1)`



| x    | 100  | 200  | 300  | 1000 | 1600 | 161a(9=>a>=0) | 1650   |
| ---- | ---- | ---- | ---- | ---- | ---- | ------------- | ------ |
| n    | 10   | 20   | 30   | 100  | 160  | 160+(a+1)     | 160+10 |



```c++
#include <bits/stdc++.h>

using namespace std;

const int N = 10;

// 将num数组存的数合成一个整树 
int get(vector<int> num, int l, int r)
{
    int res = 0;
    for (int i = l; i >= r; i -- )
        res = res * 10 + num[i];
    return res;
}

int power10(int x)
{
    int res = 1;
    while (x -- ) res *= 10;
    return res;
}

int count(int n, int x)
{
    if (!n) return 0;
    
    vector<int> num;
    while (n)
    {
        num.push_back(n % 10);
        n /= 10;
    }
    n = num.size();
    int res = 0;
    
    for (int i = n - 1 - !x; i >= 0; i -- )
    {
        if (i < n - 1)
        {
            res += get(num, n - 1, i + 1) * power10(i);
            if (!x) res -= power10(i);
        }
        if (num[i] == x) res += get(num, i - 1, 0) + 1;
        else if (num[i] > x) res += power10(i);
    }
    return res;
}

int main()
{
    int a, b;
    while (cin >> a >> b, a)
    {
        if (a > b) swap(a, b); // 从小到大
        
        for (int i = 0; i <= 9; i ++ )
            cout << count(b, i) - count(a - 1, i) << ' ';
        cout << endl;
    }
    
    return 0;
}
```

这个使用了类似前缀和的方式 ，求a-b之间的数字，我们找到计算的规律后，直接用(0,b)-(0,a- 1)的，就得到答案了

## 状态压缩DP

### AcWing 291. 蒙德里安的梦想

```c++
// 朴素版的写法
#include <bits/stdc++.h>

using namespace std;

const int N = 12, M = 1 << N;

int n, m;
long long f[N][M];  //  f[i][j]表示处理到第i列且状态为j时的方案数
bool st[M];

// (j&k) == 0   两列的状态不能有重叠
//两列的并集  (j | k) 不存在连续奇数个0

int main()
{
    while (cin >> n >> m, n || m)  // 循环读取输入，当n和m不同时为0时继续
    {
        for (int i = 0; i < 1 << n; i ++ )
        {
            int cnt = 0;
            st[i] = true; // 初始假设状态i合法
            for (int j = 0; j < n; j ++ )   // 检查状态i的每一位（从低位到高位）
                if (i >> j & 1)  // 检查i列的第j位是否为1
                { // 如果是1，检查之前连续0的个数是否为奇数（cnt & 1），若是则状态不合法
                    if (cnt & 1) st[i] = false;
                    cnt = 0;  // 重置计数器cnt = 0
                }
                else cnt ++ ;
            if (cnt & 1) st[i] = false;
        }
        memset(f, 0, sizeof f);
        f[0][0] = 1;
        for (int i = 1; i <= m; i ++)  // 遍历每一列i（从1到m）
            for (int j = 0; j < 1 << n; j ++ )  // 枚举当前列i的所有可能状态j
                for (int k = 0; k < 1 << n; k ++ ) // 枚举前一列i-1的所有可能状态k
                    if ((j & k) == 0 && st[j | k]) 
                        f[i][j] += f[i - 1][k];
        cout << f[m][0] << endl;
    }
    return 0;
}
```

```cpp
// 去除无效状态的优化写法
#include <bits/stdc++.h>

using namespace std;

typedef long long LL;

const int N = 12, M = 1 << N;

int n, m;
LL f[N][M];
vector<int> state[M];// state[i]存储所有能与状态i相邻的合法状态(所有能转移到状态i的合法状态)
bool st[M];          // 判断每列横方块放置的合法状态，用十进制存储，用它的二进制表示

int main()
{
    //预处理st，能放入竖方块的合法状态 ：用二进制数状态表示，用二进制数的十进制状态存储
    while (cin >> n >> m, n || m)  
    {
        for (int i = 0; i < 1 << n; i ++ ) //检查0到2^(n-1)种状态，这里是遍历一列中的状态
        {
            int cnt = 0;  // 记录当前连续的空位数量
            bool is_valid = true;
            for (int j = 0; j < n; j ++ )
                if (i >> j & 1)  // 用于检查第i列的第j位（从右往左数，最低位为第0位）是否为1。
                {
                    if (cnt & 1) // 如果之前的空位数是奇数，无法放满竖方块
                    {
                        is_valid = false;
                        break;
                    }
                }
            	else cnt ++ ;  // 如果第j行是空的，计数器加1
            if (cnt & 1) is_valid = false;  // 检查最后一组连续空位，如果是奇数则不合法
            st[i] = is_valid;  // 记录状态i的合法性
        }
        
        for (int i = 0; i < 1 << n; i ++ )    // 遍历所有可能的状态
        {
            // 清空上一次计算的结果
            // 如果不清空，当处理第二组输入时
            // state[i]中会同时包含上一组输入的状态和新输入的状态
            // 例如，第一次 n = 3 时计算的状态和第二次 n = 4 时计算的状态会混在一起，导致错误
            state[i].clear();
            for (int j = 0; j < 1 << n; j ++ )
                if ((i & j) == 0 && st[i | j])
                    state[i].push_back(j);
        }
        memset(f, 0, sizeof f);
        f[0][0] = 1;
        for (int i = 1; i <= m; i ++ )
            for (int j = 0; j < 1 << n; j ++ )
                for (auto k : state[j])
                    f[i][j] += f[i - 1][k];  // 累加所有可能的转移方案数
        // 输出最终结果：第m列状态为0(第0~m-1列全部填满且没有伸出)的方案数
        cout << f[m][0] << endl; 
    }
    return 0;
}
```

**别人的注释**

```c++
#include <cstring>          
#include <iostream>         
#include <algorithm>        
#include <vector>           
using namespace std;        
typedef long long LL;
const int N = 12, M = 1 << N;   // 定义常量N为12(最大行数+1)，M为2^N(最大可能的状态数)
int n, m; 
LL f[N][M];             

// 状态计算：从i-1列到第i列的方案固定了，但是从i-2列到第i-1列的伸法没有固定
// 此处从i-2列伸出到第i-1列的状态设为k(它也是j的相邻状态)，它与j满足一定条件时，是合法的
// 需满足的条件为：
// 1. k和j不能在同一行为1（填充冲突, 此时两个冒头的为0）
// 2. 第i-1空着的小方块能被竖着的1*2的小方块塞满（列方向上连续的空着的方块数为偶数）

bool st[M];             // st[i]表示状态i是否合法(能否用竖方块填满剩余空位)
vector<int> state[M];   // state[i]存储所有能与状态i相邻的合法状态(所有能转移到状态i的合法状态)

// 初始状态：因为从“-1列”（不存在）伸出到第0列的方案数显然为0，但是0也是一种解，所以f[0][0] = 1, 这是这个状态转移的递归基
// 显然f[0][u] (u != 0) 值都为0
// 最终输出为f[m][0], 意为0~m-1列(即整个棋盘)已经全部摆好，且从m-1列伸出到m列的行数为0
int main()
{
    while(cin >> n >> m, n || m)
    {
        for(int i = 0; i < 1 << n; i ++)   // 遍历所有可能的状态(0到2^n-1)
        {
            int cnt = 0;                   // 记录当前连续的空位数量
            bool is_valid = true;          // 标记当前状态是否合法
            for(int j = 0; j < n; j ++)    // 遍历该状态的每一位(每一行)
            {
                if(i >> j & 1)             // 如果第j行放了横方块
                {                      
                    if(cnt & 1)            // 如果之前的空位数是奇数，无法放满竖方块
                    {
                        is_valid = false;  // 标记状态为不合法
                        break;
                    }
                }
                else  cnt ++;              // 如果第j行是空的，计数器加1
            } 
            if(cnt & 1)  is_valid = false; // 检查最后一组连续空位，如果是奇数则不合法
            st[i] = is_valid;              // 记录状态i的合法性
        }


        for(int i = 0; i < 1 << n; i ++)      // 遍历所有可能的状态
        {
            // 清空上一次计算的结果
            // 如果不清空，当处理第二组输入时：
            // state[i]中会同时包含上一组输入的状态和新输入的状态
            // 例如，第一次n=3时计算的状态和第二次n=4时计算的状态会混在一起，导致错误
            state[i].clear();

            for(int j = 0; j < 1 << n; j ++)  // 遍历所有可能的相邻状态
                if((i & j) == 0 && st[i | j])
                    // i表示从第i-1列伸出到第i列的二进制数, j表示从第i-2列伸出到第i-1列的二进制数
                    // (i & j) == 0 意为状态i与其相邻状态j没有重叠（不在同一行的第i-1列为1）
                    // i | j 表示的就是第i-1列的二进制数，st[i|j]表示的就是第i-1列二进制数确定时，其能否用竖方块填满剩余空位(合法)
                    state[i].push_back(j);    // 将合法的相邻状态j添加到state[i]中
        }

        // 初始化dp数组为0
        // 初始状态：第0列空状态的方案数为1
        memset(f, 0, sizeof f); 
        f[0][0] = 1;

        for(int i = 1; i <= m; i ++)          // 从第1列开始遍历到第m列
            for(int j = 0; j < 1 << n; j++)   // 遍历第 i 列的所有可能状态
                for(auto k : state[j])        // 遍历所有能够转移到状态j的前一列状态k
                    f[i][j] += f[i - 1][k];   // 累加所有可能的转移方案数

        cout << f[m][0] << endl;              // 输出最终结果：第m列状态为0(第0~m-1列全部填满且没有伸出)的方案数
    }

    return 0;
}
```



### AcWing 91. 最短Hamilton路径

给定一张 n 个点的带权无向图，点从 0∼n−1 标号，求起点 0 到终点 n−1 的最短 Hamilton 路径。

Hamilton 路径的定义是从 0 到 n−1 不重不漏地经过每个点恰好一次。

```c++

```



## 树形DP

### AcWing 285. 没有上司的舞会



## 记忆化搜索

### AcWing 901. 滑雪