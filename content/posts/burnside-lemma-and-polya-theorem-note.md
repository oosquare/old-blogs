---
title: "Burnside 引理 & Polya 定理学习笔记"
subtitle: ""
date: 2022-02-12T19:14:24+08:00
draft: true
author: ""
authorLink: ""
description: ""
keywords: ""
license: ""
comment: false
weight: 0

tags:
  - OI
  - 群论
  - 置换群
  - Burnside 引理
  - Polya 定理
  - C++
categories:
  - 笔记

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

toc:
  enable: true
math:
  enable: true
lightgallery: true
seo:
  images: []

# See details front matter: /theme-documentation-content/#front-matter
---

Burnside 引理和 Polya 定理主要用于解决计算本质不同方案数的计数问题。

## 群论
### 基本定义
群可以看成是一个由集合和某个二元运算组成的二元组 $(S,\cdot)$，这里的 $\cdot$ 就代表一个二元运算，这个二元运算需要满足结合律、存在单位元和逆元。群有很多的种类，比如置换群、循环群、矩阵群等，其中置换群就是我们接下来要介绍的。

### 性质
若一个二元组 $(S,\cdot)$ 是群，则它满足以下性质：
1. 封闭性：$\forall x,y\in S$，若 $z=x\cdot y$，则 $z\in S$
2. 结合律：$\forall x,y,z\in S$，$(x\cdot y)\cdot z=x\cdot (y \cdot z)$
3. 单位元：存在唯一一个元素 $e \in S$ 满足 $\forall x \in S$，$e\cdot x=x$
4. 逆元：$\forall x \in S$，都可以找到唯一一元素 $x^{-1}$ 满足 $x \cdot x^{-1} = e$，这个元素称为 $x$ 的逆元

举一个简单的例子，$(\mathrm{Z},+)$ 就是一个群，无论取哪两个元素进行加法运算，结果仍然是整数；加法满足结合律；单位元 $e = 0$；任意一个元素 $x$ 的逆元 $x^{-1} = -x$。

## 置换与置换群
### 置换的基本定义
顾名思义，置换群就是置换组成的群。置换可以看成一个有限集合到自身的双射，具体一点，假设有两个长度相同的集合 $p,q$（这里假设集合可以有序），则置换 $f$ 作用于 $p$ 就是生成一个新的集合 $p'$，满足 $p'_i=p_{q_i}$。

置换表示成下面这样：

$$
f=
\begin{pmatrix}
p_1, p_2, p_3, \dots, p_n\\\\
p_{q_1}, p_{q_2}, p_{q_3}, \dots p_{q_n}
\end{pmatrix}
$$

有时我们会看到这样的置换：

$$
f=
\begin{pmatrix}
1, 2, 3, \dots, n\\\\
p_1, p_2, p_3, \dots, p_n
\end{pmatrix}
$$

这种置换也经常被简写为：

$$
f=
\begin{pmatrix}
p_1, p_2, p_3, \dots, p_n
\end{pmatrix}
$$

### 置换的乘法
假设有两个置换 $f,g$，定义它们的乘法为 $f \cdot g$：
$$
f=
\begin{pmatrix}
1, 2, 3, \dots, n\\\\
p_1, p_2, p_3, \dots, p_n
\end{pmatrix}
,
g=
\begin{pmatrix}
p_1, p_2, p_3, \dots, p_n\\\\
q_1, q_2, q_3, \dots, q_n
\end{pmatrix}
\\\\
f \cdot g =
\begin{pmatrix}
1, 2, 3, \dots, n\\\\
q_1, q_2, q_3, \dots, q_n
\end{pmatrix}
$$

### 循环置换
循环置换是特殊的置换，它表示为下面这样：

$$
f=
\begin{pmatrix}
1, 2, 3, \dots n - 1, n\\\\
2, 3, 4, \dots, n, 1
\end{pmatrix}
$$

若两个置换不含相同的元素，则称这两个置换不相交。任何的置换都可以拆分为若干不相交的循环置换的乘积。比如下面的例子：

$$
\begin{pmatrix}
a_1, a_2, a_3, a_4, a_5, a_6\\\\
a_2, a_1, a_5, a_4, a_6, a_3
\end{pmatrix}
=
\begin{pmatrix}
a_1, a_2\\\\
a_2, a_1
\end{pmatrix}
\cdot
\begin{pmatrix}
a_3, a_5, a_6\\\\
a_5, a_6, a_3
\end{pmatrix}
\cdot
\begin{pmatrix}
a_4\\\\
a_4
\end{pmatrix}
$$

要证明这个定理，可从把每个元素看成结点，上往下连有向边，因为上下两行的下标都是排列，则每个元素都在每一行有且仅出现一次，也就是入度和出度均为 $1$，显然只可能构成若干个简单环，每个简单环就代表一个循环置换。

### 置换群
由置换和置换的乘法组成的群就是置换群，置换经过乘法运算后显然还是置换，多个置换的结合顺序先后不影响答案，单位元是恒等置换，即上下两行的元素排列相同，每个置换的逆元是交换上下两行的置换。

## 本质不同计数问题与置换
这一类的问题通常会给出一些规定，定义了哪些状态是本质相同的。比如要求给一个 $2 \times 2$ 的正方形染色，可选颜色有两种，如果一种方案可以通过旋转正方形得到另外一种，则这两种方案等价。

如果我们把正方形的每个格子看成集合的一个元素，则可以通过定义置换来表示旋转，如果一个状态经过某个置换的作用后得到了另外一个状态，则它们等价。

## Burnside 引理
### 不动点
对于一个置换 $f$，若 $s$ 满足 $f(s) = s$，则称 $s$ 为 $f$ 的不动点。设有一个集合 $X$ 表示所有的状态，则记 $X^f$ 为 $f$ 作用于 $X$ 中的元素得到的不动点子集，记 $c_1(f) = |X^f|$。

### 等价类
若两个集合经过置换作用后相等，则它们属于同一个等价类。

### 引理内容
设 $A$，$B$ 为有限集合，$X$ 为 $A$ 到 $B$ 的映射组成的集合，$G$ 为 $A$ 上的置换群，$X/G$ 为 $X$ 中的映射经过 $G$ 中所有的置换作用后的等价类集合，则有：

$$
|X/G|=\frac{1}{|G|}\sum_{f\in G}|X^f|=\frac{1}{|G|}\sum_{f\in G}c_1(f)
$$

即 $G$ 中所有的置换作用于 $X$ 的不动点个数的均值。

直接理解这个公式比较难，我们使用上面的染色问题来解释。这里的 $A$ 就是表示 $2 \times 2$ 正方形中的每个格子，$B$ 代表两种颜色，$X$ 就是代表染色方案，即一种染色方案就是格子对于上一种颜色的映射，$X$ 就是所有这些映射的集合。$G$ 作为 $A$ 上的置换群，代表旋转正方形的各种操作，$X/G$ 是等价类集合，即本质不同的染色方案。

这个引理的证明比较复杂，需要较好的群论基础，还要了解轨道-稳定子定理，这里就不展开了，~~反正也不考证明。~~

### 对正方形染色问题的解决
现在具体分析一下这个问题。旋转方法有 $4$ 种，分别是：
1. 顺时针旋转 $0$，此时为恒等置换，随意染色，共 $2^4=16$ 种
2. 顺时针旋转 $\frac{\pi}{2}$，此时仅有全部颜色相同的方案为不动点，共 $2$ 种
3. 顺时针旋转 $\pi$，此时只要对角的颜色相同即可，共 $2$ 对对角，每队 $2$ 种方案，共 $2^2 = 4$ 种
4. 顺时针旋转 $\frac{3\pi}{2}$，这种情况同第二种，共 $2$ 种

带入公式得：

$$
|X/G|=\frac{1}{4}\times (16+2+4+2)=6
$$

可以自己枚举验证一下。

## Polya 定理
### 定理内容
Polya 定理是 Burnside 引理的具体化，同时也有一定的限制条件。在前置条件于 Burnside 引理相同的情况下，给出 Polya 定理的公式：

$$
|X/G|=\frac{1}{|G|}\sum_{f\in G} |B|^{\tau(f)}
$$

其中 $\tau(f)$ 表示 $f$ 的循环置换个数。根据上文任何置换都可以拆分为若干不相交循环置换的乘积，循环置换之间互不影响，故循环置换之内的元素染色方案必须相同，这样才能满足经过置换作用后仍然保持不变，即不动点，而循环置换之间可以不同，每个循环置换可以有 $|B|$ 种选择（按照上文的例子就是颜色数），通过乘法得出不动点总数为 $|B|^{\tau(f)}$。

一个理解循环置换的方法就是把它看成进行有向图连边后的环，可以通过找环来得出循环置换个数。

### 用 Polya 定理解决正方形染色问题
同样分 $4$ 类讨论：
1. 顺时针旋转 $0$，$\tau(f)=4$，共 $2^4=16$ 种
2. 顺时针旋转 $\frac{\pi}{2}$，$\tau(f)=1$，共 $2$ 种
3. 顺时针旋转 $\pi$，$\tau(f)=2$，共 $2^2 = 4$ 种
4. 顺时针旋转 $\frac{3\pi}{2}$，$\tau(f)=1$，共 $2$ 种

$$
|X/G|=\frac{1}{4}\times (16+2+4+2)=6
$$

### Polya 定理的限制
有时题目会在染色方面做限制，例如某种颜色的使用次数，或颜色之间的互相制约等等，此时 Polya 定理的简单计数就不管用了，需要通过 DP 或组合数学的方法求出不动点个数，带入 Burnside 引理公式计算。

## 例题
### Luogu P4980 【模板】Pólya 定理
> 给定一个 $n$个点，$n$ 条边的环，有 $n$ 种颜色，给每个顶点染色，问有多少种本质不同的染色方案，答案对 $10^9+7$ 取模。
>
> 注意本题的本质不同，定义为：只需要不能通过旋转与别的染色方案相同。
>
> $1\le n \le 10^9$，$1\le t \le 10^3$。

容易想到旋转操作可以表示成 $n$ 种置换，第 $i$ 种代表旋转顺时针 $\frac{2\pi i}{n}$，即将每个点向前移动 $i$ 格。考虑第 $i$ 个置换拆分的循环置换个数，列出置换表达式：

$$
f_i=
\begin{pmatrix}
1,2,3,\dots,n-1,n\\
n-i+1,n-i+1,n-i+3,\dots n-i-1,n-i
\end{pmatrix}
$$

考虑在环上跳的模型，设起点为 $S$，终点为 $T$，一次跳 $i$ 格，跳了 $x$ 次，则可以列出它们的关系：

$$
S + ix \equiv T \pmod{n}
$$

根据线性同余方程的知识，可以得出方程有解，满足：

$$
ix+ny=\gcd(i,n),\ \gcd(i,n) \mid (T-S)
$$

也就是，$T$ 与 $S$ 之间的路程是 $\gcd(i,n)$ 的倍数，若 $S$ 固定，则有 $\frac{n}{gcd(i,n)}$ 个对应的 $T$ 构成了循环，共有 $\frac{n}{\frac{n}{\gcd(i,n)}}=\gcd(i,n)$ 个循环，即 $\tau(f_i)=\gcd(i,n)$。

使用 Polya 定理，带入公式，最终答案为：

$$
\begin{aligned}
\frac{1}{n}\sum_{i=1}^{n} n^{\gcd(i,n)} & = \frac{1}{n} \sum_{d\mid n}n^d \sum_{i=1}^{n} [\gcd(i,n)=d]\\\\
&= \sum_{d\mid n} n^{d-1} \sum_{i=1}^{\frac{n}{d}} [\gcd(i,\frac{n}{d})=1]\\\\
&= \sum_{d\mid n} n^{d-1} \varphi(\frac{n}{d})
\end{aligned}
$$

单次询问时间复杂度 $O(d(n)(\log_2 n+\sqrt{n}))$。

```cpp
#include <iostream>
using namespace std;

constexpr int MOD = 1e9 + 7;

int t, n;

int phi(int x) {
    int res = x;

    for (int i = 2; i * i <= x; ++i) {
        if (x % i)
            continue;
        
        res = res / i * (i - 1);

        while (x % i == 0)
            x /= i;
    }

    if (x > 1)
        res = res / x * (x - 1);

    return res;
}

int power(int x, int y) {
    int res = 1;

    for (; y; y /= 2) {
        if (y % 2)
            res = 1ll * res * x % MOD;
        
        x = 1ll * x * x % MOD;
    }

    return res;
}

int main() {
    ios::sync_with_stdio(false);
    cin >> t;

    while (t--) {
        cin >> n;
        int ans = 0;

        for (int i = 1; i * i <= n; ++i) {
            if (n % i)
                continue;

            if (i * i == n) {
                ans = (ans + 1ll * power(n, i - 1) * phi(n / i)) % MOD;
            } else {
                ans = (ans + 1ll * power(n, i - 1) * phi(n / i)) % MOD;
                ans = (ans + 1ll * power(n, n / i - 1) * phi(i)) % MOD;
            }
        }

        cout << ans << endl;
    }

    return 0;
}
```

### Luogu P2561 黑白瓷砖
> 给出一个边长为 $n$ 的等边三角形，三角形是有 $\frac{n(n+1)}{2}$ 个六边形组成的，对每个六边形黑白染色，若某一方案通过旋转、对称或旋转和对称同时进行可以得到另一个方案，则它们本质相同。求本质不同的方案数。
> 
> $1\le n\le 20$。

根据图观察得出有 $6$ 种置换方案，分别为：
1. 不做任何操作，$\tau(f_1)=\frac{n(n+1)}{2}$
2. 顺时针旋转 $\frac{2\pi}{3}$，$\tau(f_2) = \lceil \frac{n(n+1)}{6} \rceil$，三条边上关于中心旋转等角度旋转得到的三个点
3. 顺时针旋转 $\frac{4\pi}{3}$，$\tau(f_3) = \lceil \frac{n(n+1)}{6} \rceil$，同上
4. 对称，$\tau(f_4) = \sum_{i=1}^{n} \lceil \frac{i}{2} \rceil$，每行的左右两边对称的一对为一个循环置换
5. 先顺时针旋转 $\frac{2\pi}{3}$ 再对称，等价于换一条边做对称，同上
6. 先顺时针旋转 $\frac{4\pi}{3}$ 再对称，等价于再换一条边做对称，同上

同时进行的情况不管是先旋转还是先对称，都是一样的，多次旋转或多次对称的情况等价于以上 $6$ 种中的某些情况。

需要高精度计算，给出 `Python 3` 代码：

```python
from math import ceil

n = int(input())

ex = int(ceil(n * (n + 1) / 6))
ans = 2 ** (n * (n + 1) // 2)
ans += 2 ** (ex + 1) # 2, 3

ex = 0

for i in range(1, n + 1):
    ex += int(ceil(i / 2))

ans += 3 * (2 ** ex) # 4, 5, 6
ans //= 6
print(ans)
```

### Luogu P1446 Cards
> 给出 $n$ 张牌，给每张牌染上红、绿、蓝三种颜色，要求这三种牌恰好有 $Sr,Sg,Sb$ 张，以及给出 $m$ 种洗牌方法，每种洗牌方法用一个排列 $X$ 表示，第 $i$ 张牌变为原来的第 $X_i$ 张牌。求本质不同的染色方案数。
>
> $1\le n,m \le 60$，$m+1<p<100$，$Sr,Sg,Sb\le 20$，$n = Sr + Sg + Sb$。

由于对染色加上了限制，Polya 定理不能用了，考虑 Burnside 引理，并通过 DP 求出每组置换的不动点个数。显然每组置换为题目中的每种洗牌方式，另外要加上恒等置换。

对于一种置换，求出其循环置换个数 $e$，及循环置换的作用的集合的元素个数 $c[i]$。显然为了满足不动点的要求，每个循环置换内的颜色应该相同。

设 $f[i][r][g][b]$ 为选到了第 $i$ 个循环置换且三种颜色分别用了 $r,g,b$ 个的方案数，有如下状态转移方程：

$$
f[0][0][0][0] = 1\\\\
f[i][r][g][b]\longleftarrow f[i-1][r-c[i]][g][b]\ (r\ge c[i])\\\\
f[i][r][g][b]\longleftarrow f[i-1][r][g-c[i]][b]\ (g\ge c[i])\\\\
f[i][r][g][b]\longleftarrow f[i-1][r]][g][b-c[i]]\ (b\ge c[i])
$$

则不动点个数为 $f[e][Sr][Sg][Sb]$。最终答案为：

$$
\frac{1}{m} \sum_{x\in \mathrm{perm}} f[e[x]][Sr][Sg][Sb]
$$

时间复杂度 $\Theta(nmSrSgSb)$。

```cpp
#include <iostream>
#include <sstream>
#include <vector>
#include <string>
using namespace std;

constexpr int MAX_M = 60 + 5;
constexpr int MAX_S = 20 + 5;

int n, m, sr, sg, sb, p;
vector<int> perm[MAX_M];
bool vis[MAX_M];
int f[2][MAX_S][MAX_S][MAX_S], ans;

int power(int x, int y) {
    int res = 1;

    for (; y; y /= 2) {
        if (y % 2)
            res = 1ll * res * x % p;

        x = 1ll * x * x % p;
    }

    return res;
}

int dfs(int now, const vector<int> & perm) {
    if (vis[now])
        return 0;
    
    vis[now] = true;
    return dfs(perm[now], perm) + 1;
}

vector<int> findLoop(const vector<int> & perm) {
    vector<int> res = { 0 };

    for (int i = 1; i <= n; ++i)
        vis[i] = false;

    for (int i = 1; i <= n; ++i)
        if (!vis[i])
            res.push_back(dfs(i, perm));
    
    return res;
}

int dp(const vector<int> & loop) {
    int e = loop.size() - 1;
    int res = 0;

    for (int r = 0; r <= sr; ++r)
        for (int g = 0; g <= sg; ++g)
            for (int b = 0; b <= sb; ++b)
                f[0][r][g][b] = 0;
    
    f[0][0][0][0] = 1;

    for (int i = 1; i <= e; ++i) {
        int now = (i & 1), pre = (now ^ 1);

        for (int r = 0; r <= sr; ++r) {
            for (int g = 0; g <= sg; ++g) {
                for (int b = 0; b <= sb; ++b) {
                    int & val = f[now][r][g][b] = 0;

                    if (r - loop[i] >= 0)
                        val = (val + f[pre][r - loop[i]][g][b]) % p;
                    
                    if (g - loop[i] >= 0)
                        val = (val + f[pre][r][g - loop[i]][b]) % p;
                    
                    if (b - loop[i] >= 0)
                        val = (val + f[pre][r][g][b - loop[i]]) % p;
                    
                    if (i == e)
                        res = (res + val) % p;
                }
            }
        }
    }

    return res;
}

int main() {
    ios::sync_with_stdio(false);
    cin >> sr >> sg >> sb >> m >> p;
    n = sr + sg + sb;

    for (int i = 1; i <= m; ++i) {
        perm[i].push_back(0);

        for (int j = 1; j <= n; ++j) {
            int x;
            cin >> x;
            perm[i].push_back(x);
        }
    }    
    // 判断是否已经存在循环置换
    bool found = false;

    for (int i = 1; i <= m; ++i) {
        bool found2 = true;

        for (int j = 1; j <= n; ++j) {
            if (perm[i][j] != j) {
                found2 = false;
                break;
            }
        }

        if (found2) {
            found = true;
            break;
        }
    }

    if (!found) {
        ++m;

        for (int i = 0; i <= n; ++i)
            perm[m].push_back(i);
    }

    for (int i = 1; i <= m; ++i)
        ans = (ans + dp(findLoop(perm[i]))) % p;
    
    ans = 1ll * ans * power(m, p - 2) % p;
    cout << ans << endl;
    return 0;
}
```