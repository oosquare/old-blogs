---
title: "高斯消元学习笔记"
subtitle: ""
date: 2022-02-06T16:06:35+08:00
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
  - 线性代数
  - 高斯消元
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

高斯消元主要用于求解线性方程组的解，同时可以解决某些有后效性的 DP 问题，

#### 算法思想
##### 增广矩阵
为了更方便地求解方程组，可以将系数和常数项放入矩阵，接下来就可以用一些矩阵的操作来消元了。

比如有一个方程组如下：

$$
\begin{array}{l}
3x+2y+3z=10\\\\
3x+y+4z=12\\\\
x+y+z=4
\end{array}
$$

我们可以这么用矩阵表示:

$$
\begin{bmatrix}
2 & 2 & 3 \\\\
3 & 1 & 4 \\\\
1 & 1 & 1 
\end{bmatrix}
\begin{bmatrix}
10 \\\\
12 \\\\
1
\end{bmatrix}
$$

在理论分析和实现代码时，为了方便，通常把两个矩阵拼在一起，这个矩阵就是增广矩阵：

$$
\begin{bmatrix}
2 & 2 & 3 & 10 \\\\
3 & 1 & 4 & 12 \\\\
1 & 1 & 1 & 1 
\end{bmatrix}
$$

增广矩阵的第 $i$ 行代笔第 $i$ 个方程，第 $i$ 列表示第 $i$ 个未知数的系数或常数项，接下来用 $x_i$ 表示第 $i$ 个未知数。

##### 初等行变换
初等行变换是指行之间的加减乘等运算。消元的本质就是利用初等行变换，使未知数的系数变为 $0$。具体地，初等行变换是两个行中的元素逐一进行运算，以上面的矩阵为例，记 $row_i$ 表示第 $i$ 行，则 $row_2 \longleftarrow row_2 - \frac{2}{3}row_1$ 的结果就是：

$$
\begin{bmatrix}
0 & -2 & -\frac{1}{2} & -3
\end{bmatrix}
$$

这个操作使该行的第 $1$ 个元素变为了 $0$，从方程的意义上考虑，就是消去了一个元。

##### 三角矩阵和对角矩阵
三角矩阵是指一个矩阵的一个三角部分全部为 $0$，对角矩阵则是对角线之外的元素均为 $0$。高斯消元中，三角矩阵一般指下三角矩阵，对角矩阵一般指除常数项部分为对角矩阵的矩阵。

假设得到了一个对角矩阵：

$$
\begin{bmatrix}
k_1 & 0 & 0 & c_1\\\\
0 & k_2 & 0 & c_2\\\\
0 & 0 & k_3 & c_3
\end{bmatrix}
$$

那就相当于是一个个的一元线性方程，直接解出即可。而对角矩阵可以通过三角矩阵得出：

$$
\begin{bmatrix}
k_{11} & k_{12} & k_{13} & c_1\\\\
0 & k_{22} & k_{23} & c_2\\\\
0 & 0 & k_{33} & c_3
\end{bmatrix}
$$

首先可以解出 $x_3$，然后将 $x_3$ 带入第 $2$ 个方程，则 $k_{23}x_3$ 就成为了常数项，移项一下得到：

$$
\begin{bmatrix}
k_{11} & k_{12} & k_{13} & c_1\\\\
0 & k_{22} & 0 & c_2 - k_{23}x_3\\\\
0 & 0 & k_{33} & c_3
\end{bmatrix}
$$

同理带入 $x_2,x_3$ 到第 $1$ 个方程：

$$
\begin{bmatrix}
k_{11} & 0 & 0 & c_1 - k_{12}x_2 - k_{13}x_3\\\\
0 & k_{22} & 0 & c_2 - k_{23}x_3\\\\
0 & 0 & k_{33} & c_3
\end{bmatrix}
$$

##### 具体步骤与实例
只要从上往下将每行与它下面的行进行运算，每枚举一行就消去一个元，枚举第 $i$ 行则消去 $x_i$，即通过初等行变换把第 $i + 1$ 到 $n$ 的方程的 $x_i$ 系数变为 $0$。最后从下往上带入即可。

以上面的矩阵为例，这里就模拟到得到三角矩阵：

$$
\begin{bmatrix}
2 & 2 & 3 & 10 \\\\
3 & 1 & 4 & 12 \\\\
1 & 1 & 1 & 1 
\end{bmatrix}
$$

**Step 1:** $row_2 \longleftarrow row_2 - \frac{2}{3}row_1$
$$
\begin{bmatrix}
2 & 2 & 3 & 10 \\\\
0 & -2 & -\frac{1}{2} & -3 \\\\
1 & 1 & 1 & 1 
\end{bmatrix}
$$

**Step 2:** $row_3 \longleftarrow row$