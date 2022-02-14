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
**Theorem 1：** $\forall a, b \in \mathrm{Z}$，一定存在 $x,y\in \mathrm{Z}$，使得 $ax+by=\gcd(a,b)$。

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
    
    x *= c / d;
    x = (x % (p / d) + (p / d)) % (p / d);
    return x;
}
```

## 中国剩余定理及其扩展
### 中国剩余定理
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

`CRT` 的算法流程如下：
1. 令 $M=\prod_{i=1}^{n} m_i$，$M$ 不对任何数取模
2. 对于 $m_i$，令 $M_i=\prod_{i\ne j}=\frac{M}{m_i}$，$M_i^{-1}$ 为 $M_i$ 对 $m_i$ 的逆元，$c_i=M_iM_i^{-1}$，$c_i$ 不能对 $m_i$ 取模
3. 令 $x=\sum_{i=1}^{n}a_ic_i \bmod M$，则 $x$ 为模 $M$ 意义下的唯一解

证明算法的正确性：

对于 $\forall i\ne j$，$m_i\mid M_j$，故 $m_i\mid a_jc_j$，所以在模 $m_i$ 情况下，$x \equiv a_ic_i \pmod{m_i}$。因为 $M_i^{-1}$ 为 $M_i$ 对 $m_i$ 的逆元，故在模 $m_i$ 的同余式中 $c_i\equiv 1 \pmod{m_i}$，$a_ic_i\equiv a_i \pmod{m_i}$，即 $x \equiv a_i \pmod{m_i}$，满足第 $i$ 个方程。

解在模 $M$ 意义下的唯一性证明：

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
