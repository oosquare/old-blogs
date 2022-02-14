---
title: "Luogu P3647 连珠线题解"
subtitle: ""
date: 2022-02-04T19:41:12+08:00
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
  - DP
  - 树形 DP
  - C++

categories:
  - 题解

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

这里给出一种无需换根的 DP 思路。

## Part 1

我们从结点的添加方式入手，可以把结点分为三类：

- 初始的结点，这种结点只有一个
- 通过 Append 添加的结点，以下简称 A 类结点
- 通过 Insert 添加的结点，以下简称 I 类结点

不换根的思路核心就在于保证转移时的过程符合结点的添加规则。为了方便，我们可以给每条线定向。同时假设 $1$ 为根，$x$ 为当前结点，$y$ 为 $x$ 的一个儿子结点，$fa$ 为 $x$ 的父亲结点。很显然接下来可以分 $6$ 类讨论：

1. 若 $x$ 为初始结点，则其子结点、$fa$ 都应该连向  $x$
2. 若 $x$ 为 A 类结点
   1.  $x$ 向 $fa$ 连一条红线，子结点连向 $x$
   2. $x$ 向其中一个子结点 $y$ 连一条红线，其他子结点以及 $fa$ 连向  $x$

3. 若 $x$ 为 I 类结点
   1. $x$ 取代一个子结点 $y$ 连向 $fa$ 的红线，然后换成蓝线
   2. $x$ 取代两个个子结点 $y_1$ 和 $y_2$ 之间的红线，然后换成蓝线
   3. $x$ 取代 $fa$ 连向一个子结点 $y$ 的红线，然后换成蓝线

上面的分类讨论中的“连向”可以看成 Append 或 Insert，根据题意计算就可以了。

## Part 2

接下来根据分类讨论设出状态以及写出方程：

```cpp
long long f[MAXN][6];

/**
 * f[x][0]: x is initial
 * f[x][1]: x -> fa
 * f[x][2]: x -> y
 * f[x][3]: y -> x -> fa
 * f[x][4]: y1 -> x -> y2
 * f[x][5]: fa -> x -> y
 */
```

设 $cont(x)$ 表示 $x$ 向其父结点连线的贡献，则 $cont(x)=\\max\\{f[x][1],f[x][3]+len(x,fa(x))\\}$。
$$
f[x][0]=\\sum_{y\\in son(x)}cont(y)\\\\
f[x][1]=\\sum_{y\\in son(x)}cont(y)\\\\
f[x][2]=\\sum_{y\\in son(x)}cont(y)+\\max_{y\\in son(x)}\\{\\max\\{f[y][0], f[y][2], f[y][4], f[y][5] + len(y,x)\\}-cont(y)\\}\\\\
f[x][3]=\\sum_{y\\in son(x)}cont(y)+\\max_{y\\in son(x)}\\{f[y][1]+len(y,x)-cont(y)\\}\\\\
f[x][4]=\\sum_{y\\in son(x)}cont(y)+\\\\\\max_{y_1,y_2\\in son(x)\\wedge y_1\\ne y_2}\\{
f[y_1][1]+len(y_1,x)-cont(y_1)+\\max\\{f[y_2][0], f[y_2][2], f[y_2][4]\\}+len(y_2,x)-cont(y_2)\\}\\\\
f[x][5]=\\sum_{y\\in son(x)}cont(y)+\\max_{y\\in son(x)}\\{\\max\\{f[y][0], f[y][2], f[y][4]\\}+len(y,x)-cont(y)\\}
$$
上面的转移都可以看成先从子结点按照 $cont(x)$ 转移，在钦定某个结点按照特定的方式转移，记录下最优的差值在加到 $f$ 里即可。$f[x][4]$ 有两个变量，可以考虑枚举其中一个，用 `std::multiset<T>` 维护另外一个的最值即可。

时间复杂度 $O(n \\log_2 n)$。

## Part 3

```cpp
#include <iostream>
#include <vector>
#include <set>
using namespace std;

constexpr int MAXN = 200000 + 10;
constexpr int INFINITY = 0x7fffffff;

int n;
vector<pair<int, int>> tree[MAXN];
long long f[MAXN][6];

/**
 * f[x][0]: x is initial
 * f[x][1]: x -> fa
 * f[x][2]: x -> y
 * f[x][3]: y -> x -> fa
 * f[x][4]: y1 -> x -> y2
 * f[x][5]: fa -> x -> y
 */

void link(int x, int y, int l) {
    tree[x].push_back({y, l});
    tree[y].push_back({x, l});
}

void dfs(int x, int fa) {
    if (tree[x].size() == 1 && fa != 0) {
        f[x][2] = f[x][3] = f[x][4] = f[x][5] = -INFINITY;
        return;
    }

    long long sum = 0, delta[4] = {-INFINITY, -INFINITY, -INFINITY, -INFINITY};
    multiset<long long> val;
    val.insert(-INFINITY);

    for (auto [y, l] : tree[x]) {
        if (y == fa)
            continue;

        dfs(y, x);

        long long cont = max(f[y][1], f[y][3] + l);
        sum += cont;

        for (int i = 0; i < 6; ++i)
            f[x][i] += cont;

        delta[0] = max(delta[0], max(max(max(f[y][0], f[y][2]), f[y][4]), f[y][5] + l) - cont);
        delta[1] = max(delta[1], f[y][1] + l - cont);
        delta[3] = max(delta[3], max(max(f[y][0], f[y][2]), f[y][4]) + l - cont);
        val.insert(f[y][1] + l - cont);
    }

    for (auto [y, l] : tree[x]) {
        if (y == fa)
            continue;

        int cont = max(f[y][1], f[y][3] + l);
        val.erase(val.find(f[y][1] + l - cont));
        delta[2] = max(delta[2], *val.rbegin() + max(max(f[y][0], f[y][2]), f[y][4]) + l - cont);
        val.insert(f[y][1] + l - cont);
    }

    f[x][2] += delta[0];
    f[x][3] += delta[1];
    f[x][4] += delta[2];
    f[x][5] += delta[3];

    for (int i = 0; i < 6; ++i)
        if (f[x][i] < 0)
            f[x][i] = -INFINITY;
}

int main() {
    ios::sync_with_stdio(false);
    cin >> n;

    if (n == 1) {
        cout << 0 << endl;
        return 0;
    }

    for (int i = 1; i < n; ++i) {
        int x, y, l;
        cin >> x >> y >> l;
        link(x, y, l);
    }

    dfs(1, 0);
    cout << max(max(f[1][0], f[1][2]), f[1][4]) << endl;
    return 0;
}
```