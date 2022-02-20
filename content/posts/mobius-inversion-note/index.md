---
title: "莫比乌斯反演"
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
  - 数论
  - 莫比乌斯反演
  - 莫比乌斯函数
  - 数论方块
  - 积性函数
  - Dirichlet 卷积
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
  enable: false
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
0 & \exist p^2 \mid n\\\\
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

Q.E.D.

#### 与常数函数的卷积
这个性质十分重要，是反演的基础。这个性质可以写成 $I * \mu= \epsilon$，或者：
$$
\sum_{d\mid n}I(\frac{n}{d})\mu(d)=\epsilon(n)
$$

经常简记为：

$$
\sum_{d\mid n} \mu(d) = [n=1]
$$

证明如下：

考虑 $d$ 作为 $n$ 的约数，其自身对整体的贡献，设 $n,d$ 的唯一分解分别为 $n=\prod_{i=1}^{k} p_i^{c_i}$，$d=\prod_{i=1}^{k} p_i^{c'_i}$。

1. 若 $\exist\ p_i$ 满足 $p_i^2 \mid d$，则 $\mu(d)=0$，容易发现这样的 $d$ 的唯一分解中，存在一个 $c'_i>1$。
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

Q.E.D.

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