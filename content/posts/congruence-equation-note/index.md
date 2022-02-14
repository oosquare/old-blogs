---
title: "同余方程学习笔记"
subtitle: ""
date: 2022-02-13T21:40:02+08:00
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
  - 数论
  - 同余
  - 中国剩余定理
  - BSGS
  - 原根
  - N 次剩余
categories:
  - draft

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

解各种同余方程是同余问题的一个重要部分，本文介绍各种同余方程的求解方法。

## 二元线性不定方程
### 形式
不定方程的范围很广泛，只要没有确定的解的方程都可以叫不定方程。本文主要讨论数论中的不定方程。

$$
ax+by=c\ (a,b,c,x,y\in \mathrm{Z})
$$

形如上面这种形式，满足各项的系数和解均为整数的方程就是数论中的不定方程。

### 求解
**Theorem 1**：$\forall a, b \in \mathrm{Z}$，一定存在 $x,y\in \mathrm{Z}$，使得 $ax+by=\gcd(a,b)$。

这个定理又叫做裴蜀定理，根据这个定理，可以得出不定方程 $ax+by=c$ 有解的充要条件就是 $\gcd(a,b)\mid c$。记 $d=\gcd(a,b)$，则我们可以先求出 $ax'+by'=d$ 的解，再令 $x=\frac{c}{d}x',y=\frac{c}{d}y'$，就得出原方程的一组解。

求 $x',y'$ 的过程可以用扩展欧几里得算法，即 `exgcd` 解决。

**Theorem 2**：若 $\gcd(a,b)=1$，且 $x_0,y_0$ 是方程 $ax+by=c$ 的一组解，则该方程的通解为 $x=x_0+kb,y=y_0-ka$，其中 $k\in \mathrm{Z}$。

换句话说，$x,y$ 这两个解是具有周期性的。根据这个定理，我们知道一定有一个 $x\in [0,b)$，而对于方程 $ax+by=c$，则一定有一个 $x\in [0,\frac{b}{\gcd(a,b)})$，由此可以求一个不定方程的一个元的最小非负整数解，具体地，假设知道了一个解 $x_0$，则最小非负整数 $x=(x_0 \bmod m + m) \bmod m$，其中 $m=\frac{b}{\gcd(a,b)}$。

总结一下，求解二元线性不定方程 $ax+by=c$ 的流程如下：
1. 求出 $ax+by=\gcd(a,b)$ 的一组解 $x',y'$，并根据 $\gcd(a,b)\mid c$ 判断方程有无解
2. 令 $x=\frac{c}{\gcd(a,b)}x',y=\frac{c}{\gcd(a,b)}y'$，$x,y$ 为 $ax+by=c$ 的一组解
3. 若需要求 $x$ 的最小非负整数解，则将 $x=(x_0 \bmod m + m) \bmod m$ 作为最终的答案，其中 $m=\frac{b}{\gcd(a,b)}$

## 线性同余方程
### 形式
$$
ax \equiv c \pmod{p}
$$

形如这样的方程，就被称线性同余方程。

### 求解
任意的线性同余方程都可以与一个二元线性不定方程互相转化，比如上面的方程就与二元线性不定方程有如下关系：

$$
ax \equiv c \pmod{p} \iff ax+py=c
$$

所以可以把线性同余方程转化为一个二元线性不定方程，求出 $x$ 即可，包括有无解的判定条件都是一样的。

以下代码展示了如何求线性同余方程的最小非负整数解：

### 代码
```cpp
template <typename T> T exgcd(T a, T b, T & x, T & y) {
    if (b == 0) {
        x = 1;
        y = 0;
        return a;
    } else {
        T d = exgcd(b, a % b, x, y);
        T t = x;
        x = y;
        y = t - a / b * y;
        return d;
    }
}

template <typename T> optional<T> solveEquation(T a, T c, T p) {
    a = (a % p + p) % p;
    c = (c % p + p) % p;
    T x, y;
    T d = exgcd(a, p, x, y);

    if (c % d != 0)
        return nullopt;
    
    x = (1ll * (c / d) * x % (p / d) + (p / d)) % (p / d);
    return x;
}
```

## 中国剩余定理
### 形式
$$
\begin{cases}
x \equiv a_1 \pmod{m_1}\\\\
x \equiv a_2 \pmod{m_2}\\\\
\cdots\\\\
x \equiv a_n \pmod{m_n}\\\\
\end{cases}
\\\\
\forall i\ne j,\gcd(m_i,m_j)=1
$$

形如上面这样的线性同余方程组且满足每个方程的模数两两互质，就可以用中国剩余定理解决。中国剩余定理也叫做 `CRT (Chinese Remainder Theorem)`。

### 求解
1. 令 $M=\prod_{i=1}^{n} m_i$，$M$ 不对任何数取模
2. 对于 $m_i$，令 $M_i=\prod_{i\ne j}=\frac{M}{m_i}$，$M_i^{-1}$ 为 $M_i$ 对 $m_i$ 的逆元，$c_i=M_iM_i^{-1}$，$c_i$ 不能对 $m_i$ 取模
3. 令 $x=\sum_{i=1}^{n}a_ic_i \bmod M$，则 $x$ 为模 $M$ 意义下的唯一解

### 算法正确性证明
对于 $\forall i\ne j$，$m_i\mid M_j$，故 $m_i\mid a_jc_j$，所以在模 $m_i$ 情况下，$x \equiv a_ic_i \pmod{m_i}$。

因为 $M_i^{-1}$ 为 $M_i$ 对 $m_i$ 的逆元，故在模 $m_i$ 的同余式中 $c_i\equiv 1 \pmod{m_i}$，$a_ic_i\equiv a_i \pmod{m_i}$，即 $x \equiv a_i \pmod{m_i}$，满足第 $i$ 个方程。

### 解在模 $M$ 意义下的唯一性证明

设 $x,y$ 不相等，$x,y<M$，但都满足方程组，则可以有下面的方程组：

$$
\begin{cases}
x - y\equiv 0 \pmod{m_1}\\\\
x - y\equiv 0 \pmod{m_2}\\\\
\cdots\\\\
x - y\equiv 0 \pmod{m_n}\\\\
\end{cases}
$$

则 $\forall i,m_i\mid |x-y|$，所以 $M \mid |x-y|$，即 $|x-y|=kM$，因为 $|x-y|<M$，故 $k=0$，推出 $x=y$，与假设矛盾。

### 代码
```cpp
int exgcd(int a, int b, int & x, int & y) {
    if (b == 0) {
        x = 1;
        y = 0;
        return a;
    } else {
        int d = exgcd(b, a % b, x, y);
        int t = x;
        x = y;
        y = t - (a / b) * y;
        return d;
    }
}

int solveEquation(int a, int c, int p) {
    int x, y;
    exgcd(a, p, x, y);
    x = (1ll * x * c % p + p) % p;
    return x;
}

inline int inv(int x, int p) {
    return solveEquation(x, 1, p);
}

int chineseRemainderTheorem(int n, int a[], int m[]) {
    int mod = 1, ans = 0;

    for (int i = 1; i <= n; ++i)
        mod *= m[i];
    
    for (int i = 1; i <= n; ++i) {
        int modi = mod / m[i];
        ans = (ans + 1ll * a[i] * modi * inv(modi)) % mod; // 用 exgcd 求逆元
    }

    return ans;
}
```

## 扩展中国剩余定理
### 形式
$$
\begin{cases}
a_1 x \equiv c_1 \pmod{m_1}\\\\
a_2 x \equiv c_2 \pmod{m_2}\\\\
\cdots\\\\
a_n x \equiv c_n \pmod{m_n}\\\\
\end{cases}
$$

相比普通中国剩余定理求解的方程组，少了模数互质的限制。

### 求解
考虑合并方程。设当前要合并前 $i$ 个方程，显然可以转化为合并前 $i-1$ 个方程与第 $i$ 个方程。

设 $x_i$ 为合并前 $i$ 个方程的解，$d_i=\operatorname{lcm}(m_1,m_2,\dots,m_i)$。构造 $x_i=sx_{i-1}+t$，我们要使 $x_i$ 能够满足前 $i-1$ 个方程，就要满足以下条件：
1. $\forall j < i$，$a_j(sx_{i-1}+t)\equiv a_j x_{i-1} \pmod{m_j}$
2. $a_i(sx_{i-1}+t) \equiv c_i \pmod{m_i}$

最简单的办法就是令 $s=1,t=d_{i-1}y_i$，并求一个合适的 $y_i$，也就是要求出以下方程的解 $y_i$：
$$
a_i d_{i-1}y_i \equiv c_i - a_i x_{i-1} \pmod{m_i}
$$

用 `exgcd` 解决，若该线性同余方程无解，则方程组无解，否则求出前 $i$ 个方程的解 $x_i$，进行下一次合并。

假设各个 $m_i$ 同阶，时间复杂度 $O(n \log_2 m_i)$。

### 代码
```cpp
int gcd(int a, int b) {
    return b == 0 ? a : gcd(b, a % b);
}

int exgcd(int a, int b, int & x, int & y) {
    if (b == 0) {
        x = 1;
        y = 0;
        return a;
    } else {
        int d = exgcd(b, a % b, x, y);
        int t = x;
        x = y;
        y = t - (a / b) * y;
        return d;
    }
}

optional<int> solveEquation(int a, int c, int p) {
    a = (a % p + p) % p;
    c = (c % p + p) % p;
    int x, y, d = exgcd(a, p, x, y);
    
    if (c % d)
        return nullopt;
    
    x *= (c / d);
    int m = p / d;
    x = (x % m + m) % m;
    return x;
}

optional<int> exChineseRemainderTheorem(int n, int a[], int c[], int m[]) {
    int x = 0, d = 1;
    
    for (int i = 1; i <= n; ++i) {
        auto oy = solveEquation(a[i] * d, c[i] - a[i] * x, m[i]);

        if (!oy.has_value())
            return nullopt;
        
        int y = oy.value();
        int g = gcd(d, m[i]);
        d = d / g * m[i];
        x = (x + 1ll * (d / m[i] * g) * y) % d;
    }

    return x;
}
```

## BSGS
### 离散对数
与离散对数相对的是我们熟悉的连续对数，也就是最经常接触的对数。而离散对数是在同余中的一个概念。

若在模 $m$ 意义下，$x$ 满足以下等式：

$$
a^x=b \pmod{m}
$$

则 $x$ 为以 $a$ 为底的 $b$ 的离散对数。注意离散对数不同于连续对数，一个数的离散对数有很多个，并不是唯一的。

如果 $\gcd(a,m)=1$，则我们可以使用 `BSGS` 算法求解一个数的最小的离散对数。

### 求解
因为 $\gcd(a,m)=1$，则 $a^k$ 对 $m$ 的逆元一定存在。所以我们可以把 $a^x$ 写成 $a^{ps-q}$ 并直接将 $a^{-q}$ 提取出来，即：

$$
a^{ps} \equiv ba^q \pmod{m}
$$

考虑暴力的思路，可以枚举两边的 $p,q$，如果有同余的两个 $a^{ps},a^q$，就更新答案。

考虑到枚举会计算许多重复的东西，可以用某种数据结构维护映射表，存下每个 $(a^q,q)$，再枚举左边找相应的同余的项。我们先假设 $x=ps-q \in [0,m)$，则当 $s=\lceil\sqrt{m}\rceil$ 时，时间复杂度最优，为 $\Theta(\sqrt{m})$ 或 $\Theta(\sqrt{m}\log_2 \sqrt{m})$，具体为哪种复杂度，取决于是用哈希表还是平衡树实现。 

### 可解性证明
至于为什么一定有 $x\in [0,m)$，使用抽屉原理证明：

对于任意的 $x$，$a^x \bmod m$ 最多只有 $m$ 种取值，而在 $x\in [0,m]$ 时，一共有 $m+1$ 种指数，则根据抽屉原理得出必定存在一对 $x,y$ 满足 $a^x \equiv a^y \pmod{m}$。设 $t=|x-y|$，则 $a^x a^k=a^{x+t}a^k$，因为已经保证 $\gcd(a,m)=1$，则逆元一定存在，所以可以规定 $t\in\mathrm{Z}$，所以指数具有周期性，一定可以得到 $x\in [0,m)$。

### 代码
```cpp
int power(int x, int y, int m) {
    int res = 1;

    for (; y; y /= 2) {
        if (y % 2)
            res = 1ll * res * x % m;
        
        x = 1ll * x * x % m;
    }

    return res;
}

optional<int> bsgs(int a, int b, int m) {
    a %= m;
    b %= m;
    
    unordered_map<int, int> buc;
    int s = ceil(sqrt(m)), prod = b, base = power(a, s, m);
    buc[b] = 0;

    for (int i = 0; i < s; ++i) {
        prod = 1ll * prod * a % m;
        buc[prod] = i;
    }

    prod = 1;

    for (int i = 0; i <= s; ++i) {
        auto it = buc.find(prod);

        if (it != buc.end() && i * s - it->second >= 0)
            return i * s - it->second;
            
        prod = 1ll * prod * base % m;
    }

    return nullopt;
}
```

## 扩展 BSGS
### 形式
$$
a^x \equiv b \pmod{m}
$$

其中 $a,m$ 不一定互质。这种情况下，逆元不一定存在，也就不可以把指数拆成两个部分来优化。

考虑先把式子化为 $\gcd(a,m)=1$ 的形式。方法是不断提取出 $a$ 与 $m$ 互质的部分，任何整个式子同除以这个数。

$$
a \equiv c \pmod{m} \iff \frac{a}{d} \equiv \frac{c}{d} \pmod{\frac{m}{d}}\ (d\mid a \wedge d\mid c \wedge d\mid m)
$$

这个式子是同余式的基本性质。所以按照这样的方法提取：

$$
a^x \equiv b \pmod{m}\\\\
\Downarrow\\\\
\frac{a}{d_1}a^{x-1} \equiv \frac{b}{d_1} \pmod{\frac{m}{d_1}}\\\\
\Downarrow\\\\
\frac{a}{d_1 d_2}a^{x-2} \equiv \frac{b}{d_1 d_2} \pmod{\frac{m}{d_1 d_2}}\\\\
\Downarrow\\\\
\vdots\\\\
\Downarrow\\\\
\frac{a}{d_1 d_2\cdots d_k}a^{x-k} \equiv \frac{b}{d_1 d_2\cdots d_k} \pmod{\frac{m}{d_1 d_2\cdots d_k}}\\\\
$$

假设现在 $a$ 与 $\frac{m}{d_1 d_2\cdots d_k}$ 互质，就可以用 `BSGS` 求出最后一个方程的解 $x-k$，再加上 $k$ 就是 $x$ 了。

需要注意两点：
1. 若在提取过程中，出现 $d_1 d_2 \cdots d_i \nmid b$，则方程无法再继续提取，方程无解
2. 有可能答案在 $[0,k)$ 内，只要 $O(k)$ 枚举检验即可。

```cpp
int gcd(int a, int b) {
    int t;

    while (b) {
        t = a;
        a = b;
        b = a % b;
    }
    
    return a;
}

int power(int x, int y, int m) {
    int res = 1;

    for (; y; y /= 2) {
        if (y % 2)
            res = 1ll * res * x % m;
        
        x = 1ll * x * x % m;
    }

    return res;
}

optional<int> bsgs(int a, int b, int m) {
    a %= m;
    b %= m;
    
    unordered_map<int, int> buc;
    int s = ceil(sqrt(m)), prod = b, base = power(a, s, m);
    buc[b] = 0;

    for (int i = 0; i < s; ++i) {
        prod = 1ll * prod * a % m;
        buc[prod] = i;
    }

    prod = 1;

    for (int i = 0; i <= s; ++i) {
        auto it = buc.find(prod);

        if (it != buc.end() && i * s - it->second >= 0)
            return i * s - it->second;
            
        prod = 1ll * prod * base % m;
    }

    return nullopt;
}

optional<int> exBsgs(int a, int b, int m) {
    a = (a % m + m) % m;
    b = (b % m + m) % m;
    
    if (m == 1 || b == 1)
        return 0;
        
    int prod = 1, d, k = 0;

    while (true) {
        d = gcd(a, m);
        
        if (d == 1)
            break;
        
        if (b % d)
            return nullopt;
        
        b /= d;
        m /= d;
        ++k;
        prod = prod * (a / d) % m;

        if (prod == b)
            return k;
    }

    auto res = bsgs(a, b, m);

    if (!res.has_value())
        return nullopt;
    
    return res.value() + k;
}
```

## 阶与原根
### 阶
对于一对互质的整数 $a,m$，若 $r$ 满足 $r \ne 0$ 且 $a^r \equiv 1 \pmod{m}$ 且不存在任何小于 $r$ 的正整数满足该同余式，则称 $r$ 为 $a$ 对 $m$ 的阶，记作 $r = \delta_m(a)$。

**Theorem 1**：$0 < \delta_m(a) \le \varphi(m)$。

若 $\gcd(a,m)=1$，则欧拉定理一定成立，即 $a^{\varphi(m)} \equiv 1 \pmod{m}$，显然 $0 < \delta_m(a) \le \varphi(m)$。

**Theorem 2**：若 $a^k \equiv 1 \pmod{m}$，则 $\delta_m(a) \mid k$。

假设满足 $\delta_m(a) \nmid k$ 但 $a^k \equiv 1 \pmod{m}$，则 $k=s\delta_m(a)+t$，其中 $t \in (0,\delta_m(a))$，所以：

$$
a^k \equiv (a^{\delta_m(a)})^sa^t \equiv a^t \pmod{m}
$$

根据阶的定义，$a^t \not\equiv 1 \pmod{m}$，所以矛盾，故 $\delta_m(a) \mid k$。

### 原根
若 $a$ 满足 $\delta_m(a)=\varphi(m)$，则 $a$ 为 $m$ 的原根。