---
title: "拉格朗日插值学习笔记"
subtitle: ""
date: 2022-02-07T23:53:43+08:00
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
  - 多项式
  - 拉格朗日插值
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

拉格朗日插值是众多插值算法中的一种，插值是通过一些点来求出过这些点的多项式函数的过程。

## 算法思想
### 构造函数
给出 $n + 1$ 个点 $(x_1,y_1),(x_2,y_2),\dots,(x_n,y_n),(x_{n+1},y_{n+1})$，要求出过这些点的 $n$ 次多项式函数，先考虑对每个点构造一个函数，第 $i$ 个点的函数为 $f_i(x)$，满足：

$$
f_i(x) =
\begin{cases}
y_i & x = x_i\\\\
0 & x \ne x_i
\end{cases}
\ (x\in \\{x_1,x_2,\dots,x_n,x_{n+1}\\})
$$

最后将这些函数组合为最终的函数：

$$
f(x) = \sum_{i=1}^{n+1}f_i(x)
$$

接下来就是找到一种合适的形式来表示 $f_i(x)$ 的分段规定，可以让 $f_i(x)$ 含有一些因式，满足 $x=x_j,j\ne i$ 时，其值为 $0$，$x=x_i$ 时，其值为 $y_i$。显然下面这种满足条件:

$$
f_i(x) = y_i \prod_{i\ne j} \frac{x-x_j}{x_i-x_j}
$$

所以最终的 $f(x)$ 的解析式为：

$$
f(x) = \sum_{i=1}^{n + 1} y_i \prod_{i\ne j} \frac{x-x_j}{x_i-x_j}
$$
