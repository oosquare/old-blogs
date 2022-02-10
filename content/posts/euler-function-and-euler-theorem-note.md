---
title: "欧拉函数 & 欧拉定理学习笔记"
subtitle: ""
date: 2022-02-10T08:37:50+08:00
draft: false
author: ""
authorLink: ""
description: ""
keywords: ""
license: ""
comment: false
weight: 0

tags:
  - OI
  - 数学
  - 数论
  - 欧拉函数
  - 欧拉定理
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
lightgallery: false
seo:
  images: []

# See details front matter: /theme-documentation-content/#front-matter
---

欧拉函数 $\varphi(n)$ 是一个重要的数论函数，它表示 $[1, n]$ 中与 $n$ 互质的数的个数。

## 欧拉函数性质
### 积性函数
积性函数的定义是：如果一个函数 $f(n)$ 满足 $\gcd(a,b)=1$ 时 $f(ab)=f(a)f(b)$，则 $f(n)$ 为积性函数。
若无需 $\gcd(a,b)=1$ 就有 $f(ab)=f(a)f(b)$，则它为完全积性函数。

$$
n = \prod p_i^{c_i}\\\\
f(n) = \prod f(p_i^{c_i})
$$

一般来说，各种积性函数都是可以使用欧拉筛计算出来的，尽管方法各不相同。

$\varphi(n)$ 也是积性函数，证明就省略了。

### 计算公式
因为 $\varphi(n)$ 是积性函数，所以我们可以先研究 $\varphi(p^k)$ 怎么计算。$p^k$ 的约数有 $1, p, p^2, \dots, p^k$，而 $p^2, \dots, p^k$ 都含有 $p$ 这个约数，如果一个数不与 $p^k$ 互质，则一点有 $p$ 这个约数，我们只有知道小于等于 $p^k$ 的数中 $p$ 的倍数有多少即可。由此得出 $\varphi(n) = p^k - \frac{p^k}{p} = p^k - p^{k-1}=p^k\frac{p-1}{p}$。

根据 $\varphi(n)$ 的积性函数性质，得出对于任意的 $n$：

$$
\begin{aligned}
\varphi(n) & = \prod p_i^{c_i} \frac{p_i - 1}{p_i} \\\\
& = n \prod \frac{p_i-1}{p_i} \\\\
& = n \prod (1 - \frac{1}{p_i}) 
\end{aligned}
$$

### 求和式
#### Formula 1

$$
n = \sum_{d \mid n} \varphi(d)
$$

考虑设函数 $f(x, n)=\sum_{i=1}^{n}[\gcd(i,n)=x]$，则：

$$
n=\sum_{d \mid n} f(d,n)
$$

又因为若 $\gcd(i,n)=x$，则 $\gcd(\frac{i}{x},{\frac{n}{x}})=1$，所以 $f(x,n)=\varphi(\frac{n}{x})$。带回原式：

$$
n=\sum_{d \mid n}\varphi(\frac{n}{d})
$$

因为 $d$ 与 $\frac{n}{d}$ 是一起出现的，所以上式等价于原式。

#### Formula 2
$$
\sum_{i=1}^{n} \sum_{j=1}^{n} [\gcd(i,j)=1]=\sum_{i=1}^{n}2\varphi(i)-1
$$

若我们仅考虑 $j\le i$，则可以固定每个 $i$ 考虑，对于每个 $i$，与其互质的 $j$ 有 $\varphi(i)$ 个，全部的和显然为 $\sum_{i=1}^{n} \varphi(i)$。对于 $j \ge i$ 的情况，可以发现 $j,i$ 相当于前一种情况的 $i,j$，变换求和顺序后其实是等价的，即：

$$
\sum_{i=1}^{n} \sum_{j=i}^{n} [\gcd(i,j)=1]=\sum_{j=1}^{n} \sum_{i=1}^{j} [\gcd(i,j)=1]
$$

左右能够枚举到的 $(i,j)$ 是一样的，又因为只有 $1$ 与本身互质，则原式等价于两种情况相加再扣除一个 $1$ 的重复。

#### Formula 3
$$
\sum_{i=1}^{n}i[\gcd(i,n)=1]=\frac{n\varphi(n)}{2}
$$

根据更相减损术 $\gcd(a,b)=\gcd(b,a-b)$，可以知道若 $\gcd(i,n)=1$，则 $\gcd(n-i,n)=1$，也就是与 $n$ 互质的数是成对出现的，且每一对的和都为 $n$，考虑头尾相加，仿照等差数列的求和公式：

$$
\begin{aligned}
& \sum_{i=1}^{n}i[\gcd(i,n)=1] + \sum_{i=1}^{n}(n-i)[\gcd(n-i,n)=1]\\\\
= &\sum_{i=1}^{n} n[\gcd(i,n)=1]\\\\
= &n\varphi(n)
\end{aligned}\\\\
$$

所以两边除以 $2$ 得出：

$$
\sum_{i=1}^{n}i[\gcd(i,n)=1]=\frac{n\varphi(n)}{2}
$$

### 筛法
使用欧拉筛时随便求出即可。假设循环到 $i$：
- 若 $i$ 为质数，则 $\varphi(i)=i-1$
- 若 $p_j \nmid i$，则 $\gcd(p_j,i)=1$，根据积性函数性质，$\varphi(ip_j)=\varphi(i)\varphi(p_j)=(p_j-1)\varphi(i)$
- 若 $p_j \mid i$，则 $\gcd(p_j,i)\ne 1$，$i$ 中已有  $p_j$ 这个约数，根据计算公式，只要乘一次$\frac{p_j-1}{p_j}$，则 $\varphi(ip_j)=p_j\varphi(i)$

## 代码实现
### 单次求解
```cpp
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
```

### 欧拉筛
```cpp
void sieve(int n) {
    notPrime[1] = true;

    for (int i = 2; i <= n; ++i) {
        if (!notPrime[i]) {
            prime[++tot] = i;
            phi[i] = i - 1;
        }

        for (int j = 1; j <= tot && i * prime[j] <= n; ++j) {
            notPrime[i * prime[j]] = true;

            if (i % prime[j]) {
                phi[i * prime[j]] = phi[i] * (prime[j] - 1);
            } else {
                phi[i * prime[j]] = phi[i] * prime[j];
                break;
            }
        }
    }
}
```

## 欧拉定理
### 定理内容
#### 普通
$$
a^{\varphi(m)} \equiv 1 \pmod{m}\ (\gcd(a,m)=1)
$$

#### 扩展
$$
a^b \equiv
\begin{cases}
a^{b \bmod \varphi(m)} & \gcd(a,m)=1\\\\
a^{b} & \gcd(a,m)\ne 1 \wedge b < \varphi(m)\\\\
a^{b \bmod \varphi(m) + \varphi(m)} & \gcd(a,m)\ne 1 \wedge b \ge \varphi(m)
\end{cases}
\pmod{m}
$$

### 应用
#### 逆元
$$
a^{\varphi(m)}\equiv a\cdot a^{\varphi(m)-1}\equiv 1 \pmod{m}\ (\gcd(a,m)=1)
$$

当 $\gcd(a,m)=1$ 时，$a^{\varphi(m)-1}$ 就是 $a$ 的逆元。费马小定理就是其特殊情况。

#### 降幂
有时幂的指数恒大，快速幂的时间复杂度也无法接受，这时可以使用扩展欧拉定理对指数进行取模减小，优化算法。

#### 幂次幂
有时题目要求类似 $a^{b_1^{b_2^{\cdots}}}$ 的式子，一般直接处理指数与底数的关系是不可做的，这时就可以使用扩展欧拉定理，同时由于复合欧拉函数 $\varphi(\varphi(\dots\varphi(n)\dots))$ 的值变为 $1$ 的次数是 $O(\log_2 n)$ 的，所以也只需要用 $O(\log_2 n)$ 扩展欧拉定理，某一层的指数就会变为 $0$，配合快速幂，总体的时间复杂度为 $O(\log_2^2 n)$。

## 例题
### Luogu P2158 仪仗队
> 给出一个 $n \times n$ 的点阵，从左下角观察，一个点能够被观察到当且仅当其与左下角的点的连线上没有其他的点，就能够被观察到的点的个数（左下角的点不算）。
>
> {{< image src = "https://cdn.luogu.com.cn/upload/pic/1149.png" width = 400 height = 300 >}}
>
> $1\le n \le 40000$。

容易看出左上部分和右下部分的答案相同，先考虑右下，对点阵建坐标系，左下角为 $(0,1)$，则半平面 $x-y \ge 0$ 中的点 $(x, y)$ 若满足 $\gcd(x,y)=1$，则可以被观察到。这部分答案为 $\sum_{i=1}^{n-1}\varphi(i)$，左上部分答案也是这个，图中还有一个位于 $(1,2)$ 的点未被算到，再加 $1$ 即可，总答案为：

$$
\sum_{x=1}^{n-1} 2\varphi(x)+1
$$

欧拉筛求出欧拉函数后直接加即可。

```cpp
#include <iostream>
using namespace std;

constexpr int MAXN = 40000 + 10;

int n, ans = 1;
bool notPrime[MAXN];
int tot, prime[MAXN], phi[MAXN];

void sieve(int n) {
    notPrime[1] = true;

    for (int i = 2; i <= n; ++i) {
        if (!notPrime[i]) {
            prime[++tot] = i;
            phi[i] = i - 1;
        }

        for (int j = 1; j <= tot && i * prime[j] <= n; ++j) {
            notPrime[i * prime[j]] = true;

            if (i % prime[j]) {
                phi[i * prime[j]] = phi[i] * (prime[j] - 1);
            } else {
                phi[i * prime[j]] = phi[i] * prime[j];
                break;
            }
        }
    }
}

int main() {
    cin >> n;

    if (n == 1) {
        cout << 0 << endl;
        return 0;
    }

    sieve(n - 1);

    for (int i = 1; i <= n - 1; ++i)
        ans += 2 * phi[i];

    cout << ans << endl;
    return 0;
}
```

### Luogu P2303 Longge 的问题
> 给定一个整数 $n$，你需要求出 $\sum_{i=1}^{n} \gcd(i,n)$。
>
> $1\le n< 2^{32}$

推式子，考虑枚举 $d$ 作为 $\gcd(i,n)$：

$$
\begin{aligned}
\sum_{i=1}^{n} \gcd(i,n)&=\sum_{d \mid n}d\sum_{i=1}^{n}[\gcd(i,n)=d]\\\\
&=\sum_{d \mid n}d\sum_{i=1}^{n}[\gcd(\frac{i}{d},\frac{n}{d})=1]\\\\
&=\sum_{d \mid n}d\sum_{i=1}^{\frac{n}{d}}[\gcd(i,\frac{n}{d})=1]\\\\
&=\sum_{d \mid n}d\varphi(\frac{n}{d})
\end{aligned}
$$

枚举约数 $d$，再计算 $\varphi$ 即可，时间复杂度 $O(d(n)\varphi(n))$，$d(n)$ 表示 $n$ 的约数个数，实际上跑得很快。

```cpp
#include <iostream>
using namespace std;

long long n, ans;

long long phi(long long x) {
    long long res = x;

    for (long long i = 2; i * i <= n; ++i) {
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

int main() {
    long long i;
    
    for (i = 1; i * i < n; ++i)
        if (n % i == 0)
            ans += i * phi(n / i) + (n / i) * phi(i);

    if (i * i == n)
        ans += i * phi(i);

    cout << ans << endl;
    return 0;
}
```

### Luogu P2350 外星人
> 设 $\varphi^x(n)$ 为对 $n$ 求 $x$ 次欧拉函数，即：
> $$
> \varphi^x(n)=
> \begin{cases}
> \varphi(n) & x=1\\\\
> \varphi^{x-1}(\varphi(n)) & x>1
> \end{cases}
> $$
> 现在给出 $n$ 的唯一分解 $\prod_{i=1}^{m}p_i^{q_i}$，求出一个最小的 $x$ 使得 $\varphi^x(n)=1$。
>
> $test$ 组数据，$1\le test\le 50$，$1\le p_i\le 10^5$，$1\le q_i\le 10^9$，$1\le m\le 2000$。

首先可以找一些规律，对于 $2^k$ 这一类数，显然最小的 $x=k$，因为每套一层欧拉函数就会使 $k$ 减小 $1$。

对于其他不等于 $2$ 的质数 $p$，$\varphi(p)=p-1=2^{k'}\prod p_i'^{c_i'}$，$\varphi^2(p)=2^{k'+k'-1}\prod p_i''^{c_i''}$，$\varphi^3(p)=2^{k'+k''+k'''-2}\prod p_i'''^{c_i'''}$，依此类推，发现几个性质：
1. 每个互质的 $p_i^{c_i}$ 同时减小 $c_i$，根据欧拉函数的公式可以得出
2. 每次每个 $p_i^{c_i}\ (p_i\ne 2)$ 都会使前面的某些 $p_j^{c_j}$ 的 $c_j$ 增加，且 $p_j<p_i$，且一定有一个 $p_j=2$。
3. 对于 $2^k$，不考虑后面的 $p_i^{c_i}$，每次 $k$ 会被减小 $1$
4. 最后一个被消去的质因数一定是 $2$

前 $3$ 个都很好理解，最后一个使用反证法，假设存在一个质因数 $p$ 大于 $2$，且经过 $t$ 次操作后，$2$ 的次数变为 $0$，而 $p$ 的次数大于 $0$，这种情况下若要满足最后一个被消去的质因数不是 $2$，则 $\varphi^{t+1}(p)$ 不能有 $2$ 的约数，与性质 $2$ 矛盾，故假设不成立。

所以，总共的消除次数就是 $2$ 的次数，这启发我们计算每个质因数 $p_i$ 最后能够产生多少个 $2$。设 $f[i]$ 表示 $i$ 能够产生的 $2$ 的个数，则：

$$
f[2]=1\\\\
f[p]=f[p-1]\\\\
f[p^k]=kf[p]\\\\
f[ap]=f[a]+f[p]\\\\
$$

其中 $p$ 为质数，这个 DP 在欧拉筛时求出即可。最后答案为：

$$
\sum_{i=1}^{m}qf[p]+[2\notin\\{p_1,p_2,\dots,p_m\\}]
$$

注意特判最开始的 $n$ 没有 $2$ 的约数的情况，这个时候要先求一次欧拉函数来获得 $2$ 的约数。

```cpp
#include <iostream>
using namespace std;

constexpr int MAXP = 1e5 + 10;

int t, m;
int f[MAXP], prime[MAXP], tot;
bool notprime[MAXP];

void preprocess(int n) {
    notprime[0] = notprime[1] = true;
    f[1] = 1; // 虽然 1 不能产生 2，但是为了方便可以这么写

    for (int i = 2; i <= n; ++i) {
        if (!notprime[i]) {
            prime[++tot] = i;
            f[i] = f[i - 1];
        }

        for (int j = 1; j <= tot; ++j) {
            if (i * prime[j] > n)
                break;

            notprime[i * prime[j]] = true;
            f[i * prime[j]] = f[i] + f[prime[j]];

            if (i % prime[j] == 0)
                break;
        }
    }
}

void solve() {
    m = read();
    bool found = false;
    long long ans = 0;

    for (int i = 1; i <= m; ++i) {
        int p, q;
        cin >> p >> q;
        found |= (p == 2);
        ans += 1ll * f[p] * q;
    }

    ans += !found;
    cout << ans << endl;
}

int main() {
    ios::sync_with_stdio(false);
    preprocess(MAXP - 10);
    cin >> t;

    while (t--)
        solve();

    return 0;
}
```

### Luogu P4139 上帝与集合的正确用法
> 给出 $p$，求以下式子：
> $$
> 2^{2^{2^{\cdots}}} \bmod p
> $$
> 可以看作 $\infty$ 个 $2$，尽管这种式子不是很严谨。
>
> $T$ 组数据，$1\le T \le 10^3$，$1\le p\le 10^7$。

使用扩展欧拉定理，原式可以写成：

$$
2^{2^{2^{\cdots}}} \bmod p=
\begin{cases}
2^{2^{2^{\cdots}} \bmod \varphi(p)} \bmod p & \gcd(2,p)=1\\\\
2^{2^{2^{\cdots}} \bmod \varphi(p) + \varphi(p)} \bmod p & \gcd(2,p)\ne 1
\end{cases}
$$

这里把 $2^{2^{2^{\cdots}}}$ 视为 $\infty$，所以不存在 $2^{2^{2^{\cdots}}}<\varphi(p)$ 的情况。设 $f(p) = 2^{2^{2^{\cdots}}} \bmod p$，就可以把上面的式子写成这样：

$$
f(p)=
\begin{cases}
2^{f(\varphi(p))} \bmod p & \gcd(2,p)=1\\\\
2^{f(\varphi(p)) + \varphi(p)} \bmod p & \gcd(2,p)\ne 1
\end{cases}
$$

当 $p=1$ 时，$f(p)=0$。先筛出 $\varphi(p)$，再用递归实现 $f(p)$，$f(p)$ 只会递归 $(\log_2 p)$ 层，可以再加上记忆化搜索。时间复杂度 $O(p+T\log_2^2 p)$。

```cpp
#include <iostream>
using namespace std;

constexpr int MAXP = 1e7 + 10;

int t, p;
int buc[MAXP];
int prime[MAXP], phi[MAXP], tot;
bool notPrime[MAXP];

int gcd(int x, int y) {
    int t;

    while (y) {
        t = x;
        x = y;
        y = t % y;
    }

    return x;
}

int power(int x, int y, int p) {
    int res = 1;
    
    for (; y; y /= 2) {
        if (y % 2)
            res = 1ll * res * x % p;
        
        x = 1ll * x * x % p;
    }

    return res;
}

void preprocess(int n) {
    notPrime[1] = true;

    for (int i = 2; i <= n; ++i) {
        if (!notPrime[i]) {
            prime[++tot] = i;
            phi[i] = i - 1;
        }

        for (int j = 1; j <= tot && i * prime[j] <= n; ++j) {
            notPrime[i * prime[j]] = true;

            if (i % prime[j]) {
                phi[i * prime[j]] = phi[i] * (prime[j] - 1);
            } else {
                phi[i * prime[j]] = phi[i] * prime[j];
                break;
            }
        }
    }
}

int calc(int p) {
    if (buc[p] != -1)
        return buc[p];

    int res;

    if (gcd(2, p) == 1)
        res = power(2, calc(phi[p]), p);
    else
        res = power(2, calc(phi[p]) + phi[p], p);
    
    buc[p] = res;
    return res;
}

int main() {
    ios::sync_with_stdio(false);
    preprocess(MAXP - 1);

    for (int i = 0; i < MAXP; ++i)
        buc[i] = -1;
    
    buc[1] = buc[2] = 0;

    cin >> t;

    while (t--) {
        int x;
        cin >> x;
        cout << calc(x) << endl;
    }
    
    return 0;
}
```

### Luogu P3747 相逢是问候
> 维护一个长度为 $n$ 的数组，这个数组的下标为从 $1$ 到 $n$ 的正整数。
>
> 一共有 $m$ 个操作，可以分为两种：
> 1. `0 l r`，对区间 $[l,r]$ 中的每一个数进行赋值：$a_i\longleftarrow c^{a_i}$，$c$ 为一常数。
> 2. `1 l r`，查询区间 $[l,r]$ 中的 $a_i$ 之和对 $p$ 取模的值。
>
> $1\le n,m\le 5\times 10^4$，$1\le p\le 10^8$，$0<c<p$，$0\le a_i < p$。

思路与上一题有异曲同工之处，每个数最多被赋值 $O(\log_2 p)$ 次后就固定不变了，用线段树维护区间和以及区间的最小操作次数，每次修改只处理仍然需要计算的部分，总时间复杂度 $O(n\log_2^3 p)$，常数小就可以过去了。

考虑去掉一个 $O(\log_2 p)$。由于 $\varphi(p)$ 减小得很快，仅需要 $O(\log_2 p)$ 次，意味着快速幂的模数也很少，考虑对每个模数 $M$ 预处理出 $p_1[i]=M^i$，$p_2[i]=M^{10000i}$，然后就可以 $O(1)$ 求幂了。时间复杂度 $O(n\log_2^2 p)$。

```cpp
#include <iostream>
using namespace std;

constexpr int MAXN = 5e4 + 10;
constexpr int INFINITY = 0x3f3f3f3f;

struct Node {
    int left, right;
    int cnt, sum;
};

int n, m, p, c;
int a[MAXN];
int seq[MAXN], g[MAXN], tot;
Node tree[MAXN * 4];
int p1[10001], p2[10001], p3[32][10001], p4[32][10001];

int gcd(int x, int y) {
    int t;

    while (y) {
        t = x;
        x = y;
        y = t % y;
    }

    return x;
}

int powerImpl(int x, int y) {
    if (y == INFINITY)
        return INFINITY;

    int res = 1;

    for (; y; y /= 2) {
        if (y % 2) {
            if (x == INFINITY || 1ll * res * x >= p)
                return INFINITY;

            res = res * x;
        }

        if (x == INFINITY || 1ll * x * x >= p)
            x = INFINITY;
        else
            x = x * x;
    }

    return res;
}

int power(int y) {
    int y1 = y % 10000, y2 = y / 10000;
    
    if (p1[y1] == INFINITY || p2[y2] == INFINITY || 1ll * p1[y1] * p2[y2] >= p)
        return INFINITY;
    
    return p1[y1] * p2[y2];
}

int power(int y, int id) {
    int y1 = y % 10000, y2 = y / 10000;
    return 1ll * p3[id][y1] * p4[id][y2] % seq[id];
}

int powerImpl(int x, int y, int p) {
    int res = 1;

    for (; y; y /= 2) {
        if (y % 2)
            res = 1ll * res * x % p;

        x = 1ll * x * x % p;
    }

    return res;
}

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

void preprocess() {
    int x = p;

    while (seq[tot] != 1) {
        seq[++tot] = x;
        x = phi(x);
    }

    for (int i = 1; i <= tot; ++i)
        g[i] = gcd(seq[i], c);

    for (int i = 0; i <= 10000; ++i) {
        p1[i] = powerImpl(c, i);
        p2[i] = powerImpl(c, i * 10000);
    }

    for (int i = 1; i <= tot; ++i) {
        for (int j = 0; j <= 10000; ++j) {
            p3[i][j] = powerImpl(c, j, seq[i]);
            p4[i][j] = powerImpl(c, j * 10000, seq[i]);
        }
    }
}

// 核心部分，利用扩展欧拉定理求幂次幂
// 返回值一个是答案，一个是用于处理与 phi(p) 的大小关系
pair<int, int> calc(int dep, int lim, int a) {
    // phi 的值已经为 1
    if (dep == tot) {
        int prod = a;

        for (int i = lim; i >= dep; --i) {
            prod = power(prod);

            // 用无穷大表示超过 p 的情况
            if (prod == INFINITY)
                return { 0, INFINITY };
        }

        return { 0, prod };
    }

    // 已经到 a_i，无法继续递归
    if (dep == lim + 1)
        return { a % seq[dep], a };

    auto [res, prod] = calc(dep + 1, lim, a);
    
    // seq[i] 表示对 p 求 i 次欧拉函数的结果
    // g[i] 表示 c 与 seq[i] 的 gcd
    if (g[dep] == 1)
        res = power(res, dep);
    else if (prod >= seq[dep + 1])
        res = power(res + seq[dep + 1], dep);
    else
        res = power(prod, dep);

    prod = power(prod);
    return { res, prod };
}

inline void pushup(int x) {
    tree[x].sum = (tree[x * 2].sum + tree[x * 2 + 1].sum) % p;
    tree[x].cnt = min(tree[x * 2].cnt, tree[x * 2 + 1].cnt);
}

void build(int x, int l, int r) {
    tree[x].left = l;
    tree[x].right = r;

    if (l == r) {
        tree[x].sum = a[l];
        return;
    }

    int mid = (l + r) / 2;
    build(x * 2, l, mid);
    build(x * 2 + 1, mid + 1, r);
    pushup(x);
}

void modify(int x, int l, int r) {
    if (tree[x].cnt >= tot || tree[x].right < l || r < tree[x].left)
        return;

    if (tree[x].left == tree[x].right) {
        ++tree[x].cnt;
        tree[x].sum = calc(1, tree[x].cnt, a[tree[x].left]).first;
        return;
    }

    int mid = (tree[x].left + tree[x].right) / 2;

    if (l <= mid)
        modify(x * 2, l, r);

    if (mid < r)
        modify(x * 2 + 1, l, r);

    pushup(x);
}

int query(int x, int l, int r) {
    if (l <= tree[x].left && tree[x].right <= r)
        return tree[x].sum;

    int mid = (tree[x].left + tree[x].right) / 2, res = 0;

    if (l <= mid)
        res = (res + query(x * 2, l, r)) % p;

    if (mid < r)
        res = (res + query(x * 2 + 1, l, r)) % p;

    return res;
}

int main() {
    ios::sync_with_stdio(false);
    cin >> n >> m >> p >> c;
    preprocess();

    for (int i = 1; i <= n; ++i)
        cin >> a[i];

    build(1, 1, n);

    for (int i = 1; i <= m; ++i) {
        int op, l, r;
        cin >> op >> l >> r;

        if (op == 0)
            modify(1, l, r);
        else
            cout << query(1, l, r) << endl;
    }

    return 0;
}
```