---
title: 线性DP
date: 2025-08-05 23:07:53
tags:
- 算法
categories:
- 数据结构
cover: https://bu.dusays.com/2025/08/05/68921f23a5a4e.png
---

### AcWing 898. 数字三角形

```
内部存储这样存更好操作，下面就是一个三角形在数组中的存储的样子
7
3 8
8 1 0 
2 7 4 4
4 5 2 6 5
```

```c++
#include <bits/stdc++.h>

using namespace std;

const int N = 510, INF = 1e9;

int n;
int a[N][N];
int f[N][N] = {-INF};


int main()
{
    scanf("%d", &n);
    
    for (int i = 1; i <= n; i ++ )
        for (int j = 1; j <= i; j ++ )
            scanf("%d", &a[i][j]);
            
    // for(int i = 0; i <= n; i ++ )
    // 初始化到i+1，是因为下面一层的状态方程要用到i+1
    //     for(int j = 0; j <= i + 1; j ++ )
    //         f[i][j] = -INF;
            
    f[1][1] = a[1][1];
    for (int i = 2; i <= n; i ++ )  // 从第二层开始
        for (int j = 1; j <= i; j ++ )
            f[i][j] = max(f[i - 1][j - 1] + a[i][j], f[i - 1][j] + a[i][j]);
    
    int res = -INF;
    for (int i = 1; i <= n; i ++ ) res = max(res, f[n][i]);
    
    printf("%d\n", res);
    return 0;
}
```

### AcWing 895. 最长上升子序列

1≤N≤1000，
−10^9≤数列中的数≤10^9

```c++
#include <bits/stdc++.h>

using namespace std;

const int N = 1010;

int n;
int a[N], f[N];

int main()
{
    scanf("%d", &n);
    
    for (int i = 1; i <= n; i ++ )  scanf("%d", &a[i]);
    
    for (int i = 1; i <= n; i ++ )
    {
        f[i] = 1;  // 相当于进行一个初始化， 对f[i]初始化为1，只有a[i]一个数值
        for (int j = 1; j < i; j ++ )
            if (a[j] < a[i])
                f[i] = max(f[i], f[j] + 1);
    }
    
    int res = 0;
    for (int i = 1; i <= n; i ++ ) res = max(res, f[i]);
    
    printf("%d", res);
    return 0;
}
```

### AcWing 896. 最长上升子序列 II

维护数组`q`，考虑进来一个元素`ai`，(1)  元素大于等于`q[len]`，直接将元素插入到`q`数组末尾

(2)  元素小于`q`数组末尾元素，找到第一个大于它的元素，用`ai`替换它



```c++
#include <bits/stdc++.h>

using namespace std;

const int N = 100010;

int n;
int a[N];
int q[N];

int main()
{
    scanf("%d", &n);
    for (int i = 0; i < n; i ++ ) scanf("%d", &a[i]);

    int len = 0;
    for (int i = 0; i < n; i ++ )
    {
        int l = 0, r = len;
        while (l < r)
        {
            // 对q 数组进行二分查找，查找结束后r 满足 q[r] < a[i]，于是 a[i] 可以放在 r+1 位置上。
            int mid = l + r + 1 >> 1;
            if (q[mid] < a[i]) l = mid;
            else r = mid - 1;
        }
        // 如果 r+1 > len，说明发现了更长的上升子序列。
        len = max(len, r + 1);
        // 把 a[i] 放到 q[r + 1]，更新该长度为 r+1 的最小结尾值；
        q[r + 1] = a[i];
    }

    printf("%d\n", len);

    return 0;
}
```



### AcWing 897. 最长公共子序列



### AcWing 902. 最短编辑距离



### AcWing 899. 编辑距离