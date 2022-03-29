---
title: "莫比乌斯反演学习笔记"
subtitle: ""
date: 2022-02-20T11:23:07+08:00
draft: false
author: ""
authorLink: ""
description: ""
keywords: ""
license: "本文以 CC BY-NC 4.0 许可证发布"
comment: false
weight: 0

tags:
  - OI
  - 数学
  - 数论
  - 莫比乌斯反演
  - 莫比乌斯函数
  - 数论分块
  - 积性函数
  - 筛法
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

莫比乌斯反演可以用于优化一类式子的计算。

## 莫比乌斯函数
### 定义
莫比乌斯函数的定义如下：

$$
\mu(n)=
\begin{cases}
0 & \exists\ p^2 \mid n \wedge p>1\\\\
(-1)^k & n=\prod_{i=1}^{k} p_i
\end{cases}
$$

也就是说，如果 $n$ 含有平方约数，则 $\mu(n)=0$，否则 $\mu(n)=(-1)^k$，其中 $k$ 为 $n$ 中的本质不同的质约数个数。

### 性质
#### 积性函数
$\mu(n)$ 为积性函数，证明如下：

设 $a,b$ 满足 $\gcd(a,b)=1$。

1. 若 $a,b$ 其中一个数含有平方约数，则它们的乘积也一定含有平方约数，此时它们的莫比乌斯函数值都为 $0$，即 $\mu(ab)=\mu(a)\mu(b)=0$。
2. 否则 $a,b$ 都不含有平方约数，则设它们的质约数个数分别为 $k_a,k_b$，则 $\mu(a)=(-1)^{k_a},\mu(b)=(-1)^{k_b}$，因为满足 $\gcd(a,b)=1$，故 $ab$ 的质约数个数为 $k_a+k_b$，$\mu(ab)=(-1)^{k_a+k_b}=(-1)^{k_a}(-1)^{k_b}=\mu(a)\mu(b)$。

`Q.E.D.`

#### 与常数函数的卷积
这个性质十分重要，是反演的基础。这个性质可以写成 $I * \mu= \epsilon$，或者：

$$
\sum_{d\mid n}I(\frac{n}{d})\mu(d)=\epsilon(n)
$$

经常简记为：

$$
\sum_{d\mid n} \mu(d) = [n=1]
$$

$\epsilon$ 是 `Dirchlet` 卷积的单位元函数。

证明如下：

考虑 $d$ 作为 $n$ 的约数，其自身对整体的贡献，设 $n,d$ 的唯一分解分别为 $n=\prod_{i=1}^{k} p_i^{c_i}$，$d=\prod_{i=1}^{k} p_i^{c'_i}$。

1. 若 $\exists\ p_i$ 满足 $p_i^2 \mid d$，则 $\mu(d)=0$，容易发现这样的 $d$ 的唯一分解中，存在一个 $c'_i>1$。
2. 若 $n\ne 1$，则 $k\ne 0$，对答案有贡献的 $d$ 一定满足 $\forall\ i\in [1,k],c_i\in [0,1]$，也就是每个 $p_i$ 选一次或不选。枚举 $d$ 选的质约数个数 $i$：
$$
\sum_{i=0}^{k} (-1)^i \binom{k}{i}=
\sum_{i=0}^{k} \binom{k}{i} 1^{k-i} (-1)^i=
[1+(-1)]^k=0
$$
3. 特殊考虑 $n=1$，此时 $k=0$：
$$
\sum_{i=0}^{k} (-1)^i \binom{k}{i}=
(-1)^0 \binom{0}{0}=1
$$

综上，$\sum_{d\mid n} \mu(d) = \sum_{i=0}^{k} (-1)^i \binom{k}{i} = [n=1]$

`Q.E.D.`

#### 容斥系数
$\mu(n)$ 的本质其实是容斥系数，比如这个问题：长度为 $n$ 的整数序列，每个数的取值范围为 $[1,m]$，求满足序列元素的 $\gcd$ 为 $1$ 的序列个数。

直接计算比较难，可以用容斥原理，先设 $f(x)$ 为序列元素的 $\gcd$ 为 $x$ 的倍数的序列个数，再考虑重复。显然 $f(x)=\lfloor \frac{m}{x} \rfloor^{n}$。假设我们用全集的大小 $f(1)$ 减去不合法的部分，对 $f$ 的自变量取值进行讨论：

1. 若 $x=x_0^k$ 且 $k>1$，显然 $x$ 是 $x_0$ 的倍数且  $x_0$ 对应的序列集合包含了 $x$ 的情况，所以 $x$ 不需要再被重复计算。
2. 若 $x=ab$，则 $a,b$ 对应的序列集合包含了 $x$ 的情况
    1. 若 $a,b$ 对答案都有贡献，则 $x$ 的贡献被多算了一次
    2. 若 $a,b$ 其中一个对答案都有贡献，则 $x$ 没有被重复计算
    3. 若 $a,b$ 其中对答案都没有贡献，则 $x$ 也不需要对答案有贡献
3. 若 $x=abc$，则 $a,b,c$ 对应的序列集合包含了 $x$ 的情况
    1. 若 $a,b,c$ 对答案都有贡献，根据情况 2，$x$ 对应的序列集合含于 $a,b,c,ab,bc,ab$ 中，而 $ab,bc,ab$ 都把 $a,b,c$ 的集合中的 $x$ 的情况扣掉了，所以现在要把 $x$ 的贡献加回去。
    2. 若 $a,b,c$ 其中一个对答案都没有贡献，则同情况 2，不需要加贡献
    3. 若仅 $a,b,c$ 其中一个对答案都有贡献，则同情况 1，不需要加贡献
    4. 若 $a,b,c$ 其中对答案都没有贡献，则 $x$ 也不需要对答案有贡献
4. etc.

可以看出一个规律：
1. 若 $x$ 为某个数的贡献且不等于那个数，则相当于含有平方约数，可以相当于它的贡献乘 $0$
2. 否则考虑它能够拆成 $x=abc\cdots$，若其中一个的贡献为 $0$，则 $x$ 本身也不需要对答案有贡献，也就是其中一个数有平方约数，它就不需要有贡献
3. 若都有 $a,b,c,\dots$ 都有贡献，则贡献是正还是负，取决于 $a,b,c,\dots$ 的个数，为奇数则为负贡献，否则为正贡献。

这与 $\mu(n)$ 的定义完全一致，所以我们完全可以把 $\mu(n)$ 当成容斥系数。

## 莫比乌斯反演
### 反演公式
#### 约数反演公式

$$
f(n)=\sum_{d\mid n} g(d) \implies g(n) = \sum_{d\mid n} \mu(\frac{n}{d}) f(d)
$$

#### 倍数反演公式

$$
f(n)=\sum_{n\mid d} g(d) \implies g(n) = \sum_{n\mid d} \mu(\frac{d}{n}) f(d)
$$

### 常见结论
$$
\epsilon(n) = \sum_{d\mid n}\mu(d)\\\\
\varphi(n) = \sum_{d\mid n}\mu(\frac{n}{d})d = \sum_{d\mid n}\mu(d)\frac{n}{d}\\\\
d(n) = \sum_{x\mid n} I(\frac{n}{x})I(x) = \sum_{x\mid n}1\\\\
\sigma(n) = \sum_{d\mid n} I(\frac{n}{d})d = \sum_{d\mid n} d
$$

或者是 `Dirchlet` 卷积形式：

$$
\epsilon = I * \mu\\\\
\varphi = \mu * id\\\\
d = I * I\\\\
\sigma = I * id
$$

### 应用
如果一个函数 $g(n)$ 难以计算，但知道了它与另一个函数 $f(x)$ 的关系，如 `Dirchlet` 卷积的形式，则可以通过莫比乌斯反演得到。或者可以通过莫比乌斯反演对式子进行变换化简，方便计算。

## 例题
### Luogu P2522 Problem b
> 对于给出的 $n$ 个询问，每次求有多少个数对 $(x,y)$，满足 $a \le x \le b$，$c \le y \le d$，且 $\gcd(x,y) = k$。
> 
> $1\le n,k\le 5\times 10^4$，$1\le a\le b\le 5\times 10^4$，$1\le c\le d\le 5\times 10^4$。

设 $S(n,m)=\sum_{i=1}^{n} \sum_{j=1}^{m} [\gcd(i,j)=k]$，则每次询问的答案为 $S(b,d)-S(a-1,d)-S(b,c-1)+S(a-1,c-1)$。接下来进行 $S(n,m)$ 的推导：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
\begin{aligned}
S(n,m) & = \sum_{i=1}^{n} \sum_{j=1}^{m} [\gcd(i,j)=k]\\\\
& = \sum_{i=1}^{n} \sum_{j=1}^{m} [\gcd(\frac{i}{k},\frac{i}{k})=1]\\\\
& = \sum_{i=1}^{\floor{\frac{n}{k}}} \sum_{j=1}^{\floor{\frac{m}{k}}} [\gcd(i,j)=1]\\\\
\end{aligned}
$$

注意到 $[\gcd(i,j)=1]$ 与 $\epsilon(\gcd(i,j))$ 等价，用 $\epsilon(\gcd(i,j))$ 替换：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
\begin{aligned}
S(n,m) & = \sum_{i=1}^{\floor{\frac{n}{k}}} \sum_{j=1}^{\floor{\frac{m}{k}}} [\gcd(i,j)=1]\\\\
& = \sum_{i=1}^{\floor{\frac{n}{k}}} \sum_{j=1}^{\floor{\frac{m}{k}}} \epsilon(\gcd(i,j))\\\\
& = \sum_{i=1}^{\floor{\frac{n}{k}}} \sum_{j=1}^{\floor{\frac{m}{k}}} \sum_{d\mid \gcd(i,j)} \mu(d)\\\\
\end{aligned}
$$

变换求和顺序，先枚举 $d$，则 $d\mid \gcd(i,j)$，也就是 $d\mid i \wedge d\mid j$，则后续枚举 $i,j$ 也要满足能够被 $d$ 整除：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
\begin{aligned}
S(n,m) & = \sum_{i=1}^{\floor{\frac{n}{k}}} \sum_{j=1}^{\floor{\frac{m}{k}}} \sum_{d\mid \gcd(i,j)} \mu(d)\\\\
& = \sum_{d=1}^{\min\left(\floor{\frac{n}{k}},\floor{\frac{m}{k}}\right)} \mu(d) \sum_{i=1}^{\floor{\frac{n}{k}}} [d\mid i]\sum_{j=1}^{\floor{\frac{m}{k}}} [d\mid j]\\\\
& = \sum_{d=1}^{\min\left(\floor{\frac{n}{k}},\floor{\frac{m}{k}}\right)} \mu(d) \floor{\frac{\floor{\frac{n}{k}}}{d}} \floor{\frac{\floor{\frac{m}{k}}}{d}}\\\\
& = \sum_{d=1}^{\min\left(\floor{\frac{n}{k}},\floor{\frac{m}{k}}\right)} \mu(d) \floor{\frac{n}{dk}} \floor{\frac{m}{dk}}\\\\
\end{aligned}
$$

后面的 $\newcommand\floor[1]{\left\lfloor #1 \right\rfloor} \floor{\frac{n}{dk}} \floor{\frac{m}{dk}}$ 用数论分块优化，这个式子就仅有 $O(\sqrt{n}+\sqrt{m})$ 种取值，且相同的取值都在连续的一段区间，所以可以预处理 $\sum_{d=1}^{\min(n,m)} \mu(d)$，每一段区间就是 $\mu(d)$ 的区间和乘后面的式子。

认为 $a,b,c,d$ 同阶，则时间复杂度 $O(a+n\sqrt{a})$。

```cpp
#include <iostream>
using namespace std;

constexpr int MAX_N = 5e4;

int n, t, mu[MAX_N + 10], prime[MAX_N + 10], notPrime[MAX_N + 10];

void sieve(int n) {
    mu[1] = 1;
    notPrime[1] = 1;
    for (int i = 1; i <= n; ++i) {
        if (!notPrime[i]) {
            prime[++t] = i;
            mu[i] = -1;
        }
        for (int j = 1; j <= t; ++j) {
            if (i * prime[j] > n)
                break;
            notPrime[i * prime[j]] = true;
            mu[i * prime[j]] = i % prime[j] == 0 ? 0 : -mu[i];
            if (i % prime[j] == 0)
                break;
        }
    }
    for (int i = 1; i <= n; ++i)
        mu[i] += mu[i - 1];
}
// 这里其实算的是 gcd(i, j) = 1 的个数
// 也就是 S(n, m) = calc(n / k, m / k)
int calc(int n, int m) {
    int res = 0;
    for (int l = 1, r; l <= min(n, m); l = r + 1) {
        r = min(n / (n / l), m / (m / l));
        res += (mu[r] - mu[l - 1]) * (n / l) * (m / l);
    }
    return res;
}

int main() {
    sieve();
    cin >> n;
    
    for (int i = 1; i <= n; ++i) {
        int a, b, c, d, k, res = 0;
        cin >> a >> b >> c >> d >> k;
        res += calc(b / k, d / k);
        res -= calc(b / k, (c - 1) / k);
        res -= calc((a - 1) / k, d / k);
        res += calc((a - 1) / k, (c - 1) / k);
        cout << res << endl;
    }

    return 0;
}
```

### Luogu P2257 YY 的 GCD
> 给出 $n,m$，求满足 $1\le x\le n,1\le y\le m$ 且 $\gcd(x,y)$ 为质数的有序数对 $(x,y)$ 个数。
>
> 有 $t$ 组数据，$1\le t\le 10^4$，$1\le n,m\le 10^7$。

设质数集为 $\mathrm{P}$，则我们要求出以下式子：

$$
\sum_{i=1}^{n} \sum_{j=1}^{m} [\gcd(i,j)\in \mathrm{P}]
$$

考虑枚举 $p=\gcd(i,j) \in \mathrm{P}$，则：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
\begin{aligned}
\sum_{i=1}^{n} \sum_{j=1}^{m} [\gcd(i,j)\in \mathrm{P}]
& = \sum_{p\in \mathrm{P}} \sum_{i=1}^{n} \sum_{j=1}^{m} [\gcd(i,j) = p]\\\\
& = \sum_{p\in \mathrm{P}} \sum_{d=1}^{\min\left(\floor{\frac{n}{p}},\floor{\frac{m}{p}}\right)} \mu(d) \floor{\frac{n}{dp}} \floor{\frac{m}{dp}}
\end{aligned}
$$

这样子做绝对会 `TLE`，因为如果使用这个形式进行计算，必须要枚举 $p$，所以进行优化，设 $t=dp$，则 $d=\frac{t}{p}$，所以式子变成这样：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
\begin{aligned}
\sum_{i=1}^{n} \sum_{j=1}^{m} [\gcd(i,j)\in \mathrm{P}]
& = \sum_{p\in \mathrm{P}} \sum_{d=1}^{\min\left(\floor{\frac{n}{p}},\floor{\frac{m}{p}}\right)} \mu(d) \floor{\frac{n}{dp}} \floor{\frac{m}{dp}}\\\\
& = \sum_{p\in \mathrm{P}} \sum_{p\mid t} \mu(\frac{t}{p}) \floor{\frac{n}{t}} \floor{\frac{m}{t}}\\\\
& = \sum_{t=1}^{\min(n,m)} \floor{\frac{n}{t}} \floor{\frac{m}{t}} \sum_{p \mid t} \mu(\frac{t}{p})
\end{aligned}
$$

可以把后面的 $\sum_{p \mid t} \mu(\frac{t}{p})$ 看成一个与 $t$ 有关的函数，那么我们就可以通过枚举每个质数 $p$，让它去贡献它的每一个倍数，这个可以 $O(n \log_2 n)$ 预处理，再 $O(n)$ 预处理前缀和，然后每次询问就可以直接数论分块了。

同样认为 $n,m$ 同阶，则时间复杂度 $O(n \log_2 n+t\sqrt{n})$。

```cpp
#include <iostream>
using namespace std;

constexpr int MAX_N = 1e7 + 10;

int prime[MAX_N], notPrime[MAX_N], mu[MAX_N];
long long sum[MAX_N];
int n, m, t, l;

void sieve() {
    notPrime[0] = notPrime[1] = true;
    mu[1] = 1;
    for (int i = 2; i < MAX_N; ++i) {
        if (!notPrime[i]) {
            prime[++l] = i;
            mu[i] = -1; 
        }
        for (int j = 1; j <= l; ++j) {
            if (i * prime[j] > MAX_N - 1)
                break;
            notPrime[i * prime[j]] = true;
            mu[i * prime[j]] = i % prime[j] ? -mu[i] : 0;
            if (i % prime[j] == 0)
                break;
        }
    }
    for (int i = 1; i <= l; ++i)
        for (int j = 1; prime[i] * j < MAX_N; ++j)
            sum[prime[i] * j] += mu[j];
    for (int i = 2; i < MAX_N; ++i)
        sum[i] += sum[i - 1];
}

long long calc(int n, int m) {
    long long res = 0;
    for (int l = 1, r; l <= min(n, m); l = r + 1) {
        r = min(n / (n / l), m / (m / l));
        res += (sum[r] - sum[l - 1]) * (n / l) * (m / l);
    }
    return res;
}

int main() {
    sieve();
    cin >> t;
    for (int i = 1; i <= t; ++i) {
        cin >> n >> m;
        cout << calc(n, m) << endl;
    }
    return 0;
}
```

### Luogu P3704 数字表格
> 设 $f_i$ 为斐波那契数列的第 $i$ 项，求:
> $$
> \prod_{i=1}^{n} \prod_{j=1}^{m} f_{\gcd(i,j)} \bmod (10^9+7)
> $$
>
> $t$ 组数据，$1\le t\le 10^3$，$1\le n,m\le 10^6$。

先枚举 $d$ 作为 $\gcd(i,j)$，考虑每个 $f(d)$ 会被乘几次，也就是把式子化成这样：

$$
\prod_{d=1}^{\min(n,m)} f(d)^{\sum_{i=1}^{n} \sum_{j=1}^{m} [\gcd(i,j)=d]}
$$

首先可以化简指数，设指数为 $t(d)$：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
t(d)=\sum_{k=1}^{\min\left(\floor{\frac{n}{d}},\floor{\frac{m}{d}}\right)} \mu(k) \floor{\frac{n}{dk}} \floor{\frac{m}{dk}}\\\\
\prod_{d=1}^{\min(n,m)} f(d)^{\sum_{i=1}^{n} \sum_{j=1}^{m} [\gcd(i,j)=d]} = \prod_{d=1}^{\min(n,m)} f(d)^{t(d)}\\\\
$$

现在已经可以使用数论分块套数论分块 `A` 了这道题了，只要再预处理一个前缀积及其逆元，时间复杂度 $O(n+tn^{\frac{3}{4}}\log_2 n)$。你没有看错，可以 $\Theta(n)$ 预处理斐波那契数列的前缀积的逆元。

这个算法跑得巨慢，实现不好就会 `TLE`。进行用上面的技巧进行优化，设 $p=dk$，则 $k=\frac{p}{d}$，则：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
t(d) = \sum_{d\mid p} \mu(\frac{p}{d}) \floor{\frac{n}{p}} \floor{\frac{m}{p}}
$$

先枚举 $p$，现在 $p$ 从指数上变到了底数上，所以加变为乘，也就是变换求积顺序：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
\begin{aligned}
\prod_{d=1}^{\min(n,m)} f(d)^{t(d)}
& = \prod_{p=1}^{\min(n,m)} \prod_{d\mid p} f(d)^{\mu(\frac{p}{d}) \floor{\frac{n}{p}} \floor{\frac{m}{p}}}\\\\
& = \prod_{p=1}^{\min(n,m)} \left(\prod_{d\mid p} f(d)^{\mu(\frac{p}{d})}\right)^{\floor{\frac{n}{p}} \floor{\frac{m}{p}}}
\end{aligned}
$$

所以现在只要预处理出 $\prod_{d\mid p} f(d)^{\mu(\frac{p}{d})}$，再做前缀积，很明显可以 $O(n\log_2 n)$ 预处理这个式子，用上文的贡献法实现即可。

至于求斐波那契数列前缀积的逆元的方法，其实适用于任意的正整数序列。具体可以看[这里](/contents/)。

```cpp
#include <iostream>
using namespace std;

constexpr int MAX_N = 1e6 + 10;
constexpr int MOD = 1e9 + 7;

int t, n, m;
int prime[MAX_N], mu[MAX_N], tot;
bool notPrime[MAX_N];
int fib[MAX_N], ifib[MAX_N];
int prod[MAX_N], iprod[MAX_N];
int tmp[MAX_N];

int power(int x, int y) {
    int res = 1;

    for (; y; y /= 2) {
        if (y % 2)
            res = 1ll * res * x % MOD;

        x = 1ll * x * x % MOD;
    }

    return res;
}

void preprocessMu(int n) {
    notPrime[1] = true;
    mu[1] = 1;

    for (int i = 2; i <= n; ++i) {
        if (!notPrime[i]) {
            prime[++tot] = i;
            mu[i] = -1;
        }

        for (int j = 1; j <= tot && i * prime[j] <= n; ++j) {
            notPrime[i * prime[j]] = true;

            if (i % prime[j]) {
                mu[i * prime[j]] = -mu[i];
            } else {
                mu[i * prime[j]] = 0;
                break;
            }
        }
    }
}

void preprocessFib(int n) {
    fib[1] = 1;

    for (int i = 2; i <= n; ++i)
        fib[i] = (fib[i - 1] + fib[i - 2]) % MOD;

    tmp[0] = 1;

    for (int i = 1; i <= n; ++i)
        tmp[i] = 1ll * tmp[i - 1] * fib[i] % MOD;

    int itmp = power(tmp[n], MOD - 2);

    for (int i = n; i >= 1; --i) {
        ifib[i] = 1ll * itmp * tmp[i - 1] % MOD;
        itmp = 1ll * itmp * fib[i] % MOD;
    }
}

void preprocessProd(int n) {
    for (int i = 1; i <= n; ++i)
        prod[i] = 1;

    for (int i = 1; i <= n; ++i) {
        if (!mu[i])
            continue;

        if (mu[i] == 1) {
            for (int j = i, k = 1; j <= n; j += i, ++k)
                prod[j] = 1ll * prod[j] * fib[k] % MOD;
        } else {
            for (int j = i, k = 1; j <= n; j += i, ++k)
                prod[j] = 1ll * prod[j] * ifib[k] % MOD;
        }
    }

    prod[0] = 1;

    for (int i = 1; i <= n; ++i) {
        tmp[i] = prod[i];
        prod[i] = 1ll * prod[i - 1] * prod[i] % MOD;
    }
    
    iprod[n] = power(prod[n], MOD - 2);

    for (int i = n - 1; i >= 0; --i)
        iprod[i] = 1ll * iprod[i + 1] * tmp[i + 1] % MOD;
}

void solve() {
    int res = 1;

    for (int l = 1, r; l <= min(n, m); l = r + 1) {
        r = min(n / (n / l), m / (m / l));
        res = 1ll * res * power(1ll * prod[r] * iprod[l - 1] % MOD, 1ll * (n / l) * (m / l) % (MOD - 1)) % MOD; 
    }

    cout << res << endl;
}

int main() {
    ios::sync_with_stdio(false);
    preprocessMu(MAX_N - 10);
    preprocessFib(MAX_N - 10);
    preprocessProd(MAX_N - 10);

    cin >> t;

    while (t--) {
        cin >> n >> m;
        solve();
    }

    return 0;
}
```

### SPOJ GCDMAT GCD OF MATRIX
> 给出 $n,m$，并给出 $t$ 组询问，每组询问给出 $i_1,j_1,i_2,j_2$，求：
> $$
> \sum_{i=i_1}^{i_2} \sum_{j=j_1}^{j_2} \gcd(i,j)
> $$
>
> $1\le t\le 500$，$1\le n,m\le 5\times 10^4$，$1\le i_1\le i_2 \le n$，$1\le j_1\le j_2 \le m$。

先设 $S(n,m)=\sum_{i=1}^{n} \sum_{i=1}^{m} \gcd(i,j)$，这是一个比较套路的式子，用到了 $\varphi = \mu * id$ 的结论：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
\begin{aligned}
S(n,m) & = \sum_{d=1}^{\min(n,m)} d \sum_{i=1}^{n} \sum_{j=1}^{m} [\gcd(i,j)=d]\\\\
& = \sum_{d=1}^{\min(n,m)} d \sum_{k=1}^{\min\left(\floor{\frac{n}{d}},\floor{\frac{m}{d}}\right)} \mu(k) \floor{\frac{n}{dk}} \floor{\frac{m}{dk}}\\\\
\end{aligned}
$$

设 $p=dk$：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
\begin{aligned}
S(n,m)
& = \sum_{d=1}^{\min(n,m)} d \sum_{k=1}^{\min\left(\floor{\frac{n}{d}},\floor{\frac{m}{d}}\right)} \mu(k) \floor{\frac{n}{dk}} \floor{\frac{m}{dk}}\\\\
& = \sum_{d}^{\min(n,m)} d \sum_{d \mid p}
 \mu(\frac{p}{d}) \floor{\frac{n}{p}}\floor{\frac{m}{p}}\\\\
& = \sum_{p=1}^{\min(n,m)} \floor{\frac{n}{p}}\floor{\frac{m}{p}} \sum_{d \mid p} \mu(\frac{p}{d}) d \\\\
& = \sum_{p=1}^{\min(n,m)} \varphi(p) \floor{\frac{n}{p}}\floor{\frac{m}{p}}
\end{aligned}
$$

时间复杂度 $O(n+t\sqrt{n})$。

```cpp
#include <iostream>
using namespace std;

constexpr int MAX_N = 1e6 + 10;
constexpr int MOD = 1e9 + 7;

int t, n, m, i1, i2, j1, j2;
int prime[MAX_N], phi[MAX_N], tot;
bool notPrime[MAX_N];

void preprocess(int n) {
    notPrime[1] = 1;
    phi[1] = 1;

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

    for (int i = 2; i <= n; ++i) {
        phi[i] = phi[i - 1] + phi[i];

        if (phi[i] >= MOD)
            phi[i] -= MOD;
    }
}

int calc(int n, int m) {
    int res = 0;

    for (int l = 1, r, e = min(n, m), val; l <= e; l = r + 1) {
        r = min(n / (n / l), m / (m / l));
        val = 1ll * (n / l) * (m / l) % MOD;
        res = (res + 1ll * (phi[r] - phi[l - 1] + MOD) * val) % MOD;
    }

    return res;
}

int main() {
    ios::sync_with_stdio(false);
    cin >> t;
    cin >> n >> m;
    preprocess(min(n, m));

    while (t--) {
        cin >> i1 >> j1 >> i2 >> j2;
        long long ans = calc(i2, j2);
        ans -= calc(i1 - 1, j2);
        ans -= calc(i2, j1 - 1);
        ans += calc(i1 - 1, j1 - 1);
        cout << (ans % MOD + MOD) % MOD << endl;
    }

    return 0;
}
```

### Luogu P6156 简单题
> 给出 $n, k$，求：
> $$
> \sum_{i=1}^{n} \sum_{j=1}^{n} (i+j)^k f(\gcd(i,j)) \gcd(i,j)\\\\
> f(n) =
> \begin{cases}
> 0 & \exists\ p^2 \mid n \wedge p>1\\\\
> 1 & \text{otherwise}
> \end{cases}
> $$
>
> $1\le n\le 5\times 10^6$，$1\le k\le 10^{18}$。

首先容易发现 $f(n)=\mu^2(n)$。设答案为 $\text{ans}$，先枚举 $d=\gcd(i,j)$：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
\begin{aligned}
\text{ans} & = \sum_{i=1}^{n} \sum_{j=1}^{n} (i+j)^k \mu^2(\gcd(i,j)) \gcd(i,j)\\\\
& = \sum_{d=1}^{n} \mu^2(d)d \sum_{i=1}^{n} \sum_{j=1}^{n} (i+j)^k [\gcd(i,j)=d]\\\\
& = \sum_{d=1}^{n} \mu^2(d)d^{k+1} \sum_{i=1}^{n} \sum_{j=1}^{n} (\frac{i}{d}+\frac{j}{d})^k [\gcd(\frac{i}{d},\frac{j}{d})=1]\\\\
& = \sum_{d=1}^{n} \mu^2(d)d^{k+1} \sum_{i=1}^{\floor{\frac{n}{d}}} \sum_{j=1}^{\floor{\frac{n}{d}}} (i+j)^k [\gcd(i,j)=1]\\\\
& = \sum_{d=1}^{n} \mu^2(d)d^{k+1} \sum_{t=1}^{\floor{\frac{n}{d}}}\mu(t) \sum_{i=1}^{\floor{\frac{n}{d}}} \sum_{j=1}^{\floor{\frac{n}{d}}} (i+j)^k [t\mid i][t\mid j]\\\\
& = \sum_{d=1}^{n} \mu^2(d)d^{k+1} \sum_{t=1}^{\floor{\frac{n}{d}}}\mu(t) t^k \sum_{i=1}^{\floor{\frac{n}{d}}} \sum_{j=1}^{\floor{\frac{n}{d}}} (\frac{i}{t}+\frac{j}{t})^k [t\mid i][t\mid j]\\\\
& = \sum_{d=1}^{n} \mu^2(d)d^{k+1} \sum_{t=1}^{\floor{\frac{n}{d}}}\mu(t) t^k \sum_{i=1}^{\floor{\frac{n}{dt}}} \sum_{j=1}^{\floor{\frac{n}{dt}}} (i+j)^k\\\\
\end{aligned}
$$

设 $f(n) = \sum_{i=1}^{n} \sum_{j=1}^{n} (i+j)^k$，$p=dt$：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
\begin{aligned}
\text{ans} & = \sum_{d=1}^{n} \mu^2(d)d^{k+1} \sum_{t=1}^{\floor{\frac{n}{d}}}\mu(t) t^k \sum_{i=1}^{\floor{\frac{n}{dt}}} \sum_{j=1}^{\floor{\frac{n}{dt}}} (i+j)^k\\\\
& = \sum_{d=1}^{n} \mu^2(d)d^{k+1} \sum_{t=1}^{\floor{\frac{n}{d}}}\mu(t) t^k f\left(\floor{\frac{n}{dt}}\right)\\\\
& = \sum_{p=1}^{n} f\left(\floor{\frac{n}{p}}\right) \sum_{d\mid p} \mu^2(d) \mu\left(\frac{p}{d}\right) d^{k+1} \left(\frac{p}{d}\right)^k\\\\
& = \sum_{p=1}^{n} f\left(\floor{\frac{n}{p}}\right) p^k \sum_{d\mid p} \mu^2(d) \mu\left(\frac{p}{d}\right) d
\end{aligned}
$$

设 $g(n)=\sum_{d\mid n} \mu^2(d) \mu\left(\frac{n}{d}\right) d$：

$$
\newcommand\floor[1]{\left\lfloor #1 \right\rfloor}
\begin{aligned}
\text{ans} & = \sum_{p=1}^{n} f\left(\floor{\frac{n}{p}}\right) g(p) p^k\\\\
& = \sum_{p=1}^{n} f\left(\floor{\frac{n}{p}}\right) g(p) id_k(p)
\end{aligned}
$$

这个式子当中，出现了三个函数，依次考虑怎么求它们。

#### 求 $f(n)$
对于 $f(n) = \sum_{i=1}^{n} \sum_{j=1}^{n} (i+j)^k$，有两种方法：

**方法 1**

考虑作为底数的 $i+j$ 的每个值出现了多少次，$i+j\in [2,2n]$，为了方便，我们把 $1$ 也算进去，~~打表~~发现每个值的出现次数满足如下函数关系：

$$
cnt(i)=
\begin{cases}
i-1 & i \le n+1\\\\
2n-i+1 & i > n+1
\end{cases}
$$

于是设 $pre[i]=\sum_{j=1}^{i} (j-1)j^k$，$suf[i]=\sum_{j=i}^{2n} (2n-j+1)j^k$，$sum[i]=\sum_{j=1}^{i} j^k$，则：

$$
f(i)=pre[i+1]+(suf[i+2]-suf[2i+1])-(2n-2i)(sum[2i]-sum[i+1])
$$

可以看成以 $i+1$ 为分界，前面的用 $pre[i+1]$，后面先用 $suf[i+2]-suf[2i+1]$ 算出在 $[i+2,2i]$ 中的和，因为 $suf[i]$ 以 $2n$ 为参考，还包含 $[i+1,2n]$ 中的值，现在以 $2i$ 为参考，所以会多算 $(2n-2i)(sum[2i]-sum[i+1])$。

**方法 2**

上述式子有更简单的表示方法，设 $s_1(n)=\sum_{i=1}^{n} i^k$，$s_2(n)=\sum_{i=1}^{n}s_1(i)$，则 $f(n)=s_2(2n)-2s_2(n)$，~~证明省略，事实证明方法 1 更容易发现。~~

两种方法预处理的时间复杂度都是 $\Theta(n)$。

#### 求 $g(n)$
根据积性函数的判断性质：若 $f(n),g(n)$ 均为积性函数，则下列函数也为积性函数：

$$
h_1(n)=f(n)g(n)\\\\
h_2(n)=\sum_{d\mid n} f(\frac{n}{d}) g(d)
$$

分析 $g(n)=\sum_{d\mid n} \mu^2(d) \mu\left(\frac{n}{d}\right) d$ 的表达式：$g(n)$ 是 $\mu^2(n)n$ 和 $\mu(n)$ 的 `Dirchlet` 卷积，$\mu^2(n)n$ 由三个函数 $\mu(n),\mu(n),n$ 三个函数相乘得到，故 $g(n)$ 为积性函数。

分析 $g(n)$ 的性质，可以先从 $g(p^k)$ 的值入手，其中 $p$ 为质数：
1. $k=0$，则 $g(p^k)=g(1)=\mu^3(1)=1$
2. $k=1$，则 $g(p^k)=g(p)=\mu^2(1)\mu(p)+\mu^2(p)\mu(1)p=p-1$
3. $k=2$，则 $g(p^k)=g(p^2)=\mu^2(1)\mu(p^2)+\mu^2(p)\mu(p)p+\mu^2(p^2)\mu(1)=-p$
4. $k\ge 3$，根据抽屉原理，$d$ 和 $\frac{p^k}{d}$ 中至少有一个的 $p$ 的指数大于 $1$，故 $\mu^2(d)$ 和 $\mu\left(\frac{p^k}{d}\right)$ 至少有一个为 $0$，相乘后 $g(p^k)=0$

所以在欧拉筛时根据 $i\cdot prime[j]$ 与 $prime[j]$ 的关系进行计算即可。时间复杂度 $\Theta(n)$。

#### 求 $id_k(n)$
再考虑求 $id_k(n)$，这是个完全积性函数，所以随便筛一下就可以了，比如在遇到质数 $p$ 时用快速幂求一下 $id_k(p)$，其他时候就令 $id_k(i\cdot prime[j])=id_k(i)id_k(prime[j])$ 即可，时间复杂度 $\Theta(n+\pi(n)\log_2 k)\approx \Theta(n)$。

然后预处理一下 $g(n)id_k(n)$ 的前缀和，数论分块求解即可，总时间复杂度 $O(n+\sqrt{n})$。这种方法也可以通过 $1\le n\le 10^7$ + 多组询问的加强版。

```cpp
#include <iostream>
using namespace std;

constexpr int MAX_N = 5e6 + 10;
constexpr int MAX_SIZE = 1e7 + 10;
constexpr int MOD = 998244353;

int n;
long long k;
int prime[MAX_SIZE], tot;
bool notPrime[MAX_SIZE];
int prod[MAX_SIZE], f[MAX_N], g[MAX_SIZE], sum[MAX_N];
int fPre[MAX_SIZE], fSuf[MAX_SIZE], fSum[MAX_SIZE];
int ans;

int power(int x, long long y) {
    int res = 1;

    for (; y; y /= 2) {
        if (y % 2)
            res = 1ll * res * x % MOD;

        x = 1ll * x * x % MOD;
    }

    return res;
}

void sieve(int n) {
    n *= 2;
    prod[1] = 1;
    g[1] = 1;

    for (int i = 2; i <= n; ++i) {
        if (!notPrime[i]) {
            prime[++tot] = i;
            prod[i] = power(i, k);
            g[i] = i - 1;
        }

        for (int j = 1; j <= tot && 1ll * i * prime[j] <= n; ++j) {
            const int next = i * prime[j];
            notPrime[next] = true;
            prod[next] = 1ll * prod[i] * prod[prime[j]] % MOD;

            if (i % prime[j]) {
                g[next] = g[i] * (prime[j] - 1);
            } else if ((i / prime[j]) % prime[j]) {
                g[next] = -g[i / prime[j]] * prime[j];
                break;
            } else {
                g[next] = 0;
                break;
            }
        }
    }

    n /= 2;

    for (int i = 1; i <= n; ++i) {
        sum[i] = (sum[i - 1] + 1ll * prod[i] * g[i]) % MOD;
        sum[i] = (sum[i] + MOD) % MOD;
    }
}

void preprocess(int n) {
    for (int i = 1; i <= 2 * n; ++i) {
        fPre[i] = (fPre[i - 1] + 1ll * prod[i] * (i - 1)) % MOD;
        fSum[i] = (fSum[i - 1] + prod[i]) % MOD;
    }

    for (int i = 2 * n; i >= 1; --i)
        fSuf[i] = (fSuf[i + 1] + 1ll * prod[i] * (2 * n - i + 1)) % MOD;

    for (int i = 1; i <= n; ++i) {
        int mid = i + 1, res = 0;
        res = ((fPre[mid] + fSuf[mid + 1]) % MOD - fSuf[2 * i + 1]) % MOD;
        res = (res - 1ll * (2 * n - 2 * i) * (fSum[2 * i] - fSum[mid])) % MOD;
        res = (res + MOD) % MOD;
        f[i] = res;
    }
}

int main() {
    ios::sync_with_stdio(false);

    cin >> n >> k;

    sieve(n);
    preprocess(n);

    for (int l = 1, r; l <= n; l = r + 1) {
        r = n / (n / l);
        ans = (ans + 1ll * (sum[r] - sum[l - 1] + MOD) * f[n / l]) % MOD;
    }

    cout << ans << endl;
    return 0;
}
```