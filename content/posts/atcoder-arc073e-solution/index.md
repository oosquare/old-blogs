---
title: "AtCoder ARC 073E Ball Coloring 题解"
subtitle: ""
date: 2022-04-15T23:39:44+08:00
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
  - 贪心
  - C++
categories:
  - Olympiad in Information

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

题目链接：<https://atcoder.jp/contests/arc073/tasks/arc073_c>，题目大意如下：

> 给出 $n$ 个数对 $(a_i, b_i)$，可以给数对中的数染色，一种是 $a_i$ 红 $b_i$ 蓝，另一种则反过来。
> 定义以下 4 个变量：
> - $R_{max}$：所有被染为红色的数的最大值
> - $R_{min}$：所有被染为红色的数的最小值
> - $B_{max}$：所有被染为蓝色的数的最大值
> - $B_{min}$：所有被染为蓝色的数的最小值
>
> 要求最小化 $(R_{max} - R_{min})(B_{max} - B_{min})$。

这是一道比较巧妙的思维题。

## Part 1
直接考虑每个数对对答案的影响比较困难，但是可以发现所有的数中，最大值一定是 $R_{max}$ 或者 $B_{max}$，最小值一定是 $R_{min}$ 或者 $B_{min}$，又因为交换数的颜色对答案没有影响，所以可以考虑以下两种情况：

 - 最大值和最小值同色
 - 最大值和最小值异色

## Part 2
先考虑同色的情况。为了方便，默认 $a_i > b_i$。

先求出最大值和最小值的位置 $maxValPos, minValPos$，则最大值和最小值分别为 $a_{maxValPos}, b_{minValPos}$，再设 $up$ 和 $down$ 为另一种颜色的最大值和最小值。求出最大值和最小值后就可以先给 $up$ 和 $down$ 赋初值，也就是最值所在的数对中的另一个数，接下来考虑更新 $up$ 和 $down$。

对于其他的数对 $(a_i,b_i)$，首先可以确定他们都不会超过 $a_{maxValPos}, b_{minValPos}$，因此我们要尽可能让这些数对的数靠近 $up$ 和 $down$，让 $up$ 和 $down$ 的变动尽可能小。

可以分 4 种情况讨论：

 - 如果 $a_i$ 或 $b_i$ 满足在 $[down, up]$ 之间，则可以把在中间的这个数染成和 $up$ 和 $down$ 一样的颜色，显然 $up$ 和 $down$ 不变
 - 如果 $b_i < a_i < down$，此时必须更新 $down$，最优方案是用 $a_i$ 更新 $down$
 - 如果 $up < b_i < a_i$，此时必须更新 $up$，最优方案是用 $b_i$ 更新 $up$
 - 如果 $b_i < down \le up < a_i$，则无法即使计算出这个数对的影响，考虑稍后处理

按照上述方法初步更新 $up, down$ 后，再找出仍然满足 $b_i < down \le up < a_i$ 的数对，现在问题转化为给出若干区间 $[b_i, a_i]$，每个区间选择一个端点 $p_i$，使得这些端点和 $up, down$ 这两个点组成的区间 $[\min \\{p_i,up,down\\},\max \\{p_i,up,down\\}]$ 的长度最短。

考虑区间的关系，若一个区间被另一个区间完全覆盖，则这个区间对答案没有贡献，因为在被覆盖的情况下，大区间选择任意一个端点，小区间都可选择与大区间同一侧的端点，此时对答案没有影响，若选择相反方向，则可能使答案更劣。因此，我们可以去掉被包含的区间，剩下的就是两两相交的区间，且它们还满足一个很好的性质，就是左右端点都是单调递增的。

```plain
a b c            a b c
| | | [down, up] | | |
```

然后就是一个很显然的结论：将这些区间按照左端点排序后，一定存在一个分界点，满足分界点之前的区间都是选择右端点，分界点之后的区间都是选择左端点，证明就省略了，现在直接枚举分界点更新 $[up, down]$ 即可。

## Part 3
再考虑异色的情况，此时同样设出上一节的变量，但是 $up$ 是与 $a_{maxValPos}$ 同色的，$down$ 是与 $b_{minValPos}$ 同色的。为了最小化答案，就要使 $up$ 尽可能接近 $a_{maxValPos}$，$down$ 尽可能接近 $b_{minValPos}$。

所以找到最值后，先初始化 $up$ 为 $a_{minValPos}$，$down$ 为 $b_{maxValPos}$，然后再用 $a_i$ 更新 $up$，用 $b_i$ 更新 $down$。

时间复杂度为 $\mathcal O(n \log_2 n)$。

## Part 4
```cpp
#include <iostream>
#include <algorithm>

constexpr int MAX_N = 200000 + 10;
constexpr long long INF = 0x3f3f3f3f3f3f3f3f;

int n;
int a[MAX_N], b[MAX_N];
long long ans = INF;
std::pair<int, int> ranges[MAX_N], tmp[MAX_N];

inline bool between(int l, int r, int x) { return l <= x && x <= r; }

void updateUpAndDown(int &up, int &down, int maxValPos, int minValPos) {
    int tot = 0, tot2 = 0;

    for (int i = 1; i <= n; ++i) {
        if (i == maxValPos || i == minValPos)
            continue;

        if (b[i] < down && up < a[i])
            ranges[++tot] = { b[i], a[i] };
    }

    if (!tot)
        return;

    std::sort(ranges + 1, ranges + 1 + tot);

    for (int i = 1; i <= tot; ++i) {
        if (ranges[i].first == ranges[i + 1].first)
            continue;

        tmp[++tot2] = ranges[i];
    }

    tot = tot2;

    for (int i = 1; i <= tot; ++i)
        ranges[i] = tmp[i];

    if (up - ranges[1].first < ranges[tot].second - down)
        down = ranges[1].first;
    else
        up = ranges[tot].second;

    for (int i = 1; i < tot; ++i) {
        if (ranges[i].second - ranges[i + 1].first < up - down) {
            up = ranges[i].second;
            down = ranges[i + 1].first;
        }
    }
}

long long calcMaxMinColorSame() {
    int maxValPos = 0, minValPos = 0;
    int down, up;

    for (int i = 1; i <= n; ++i) {
        if (!maxValPos || a[maxValPos] < a[i]) {
            maxValPos = i;
            up = b[i];
        }

        if (!minValPos || b[minValPos] > b[i]) {
            minValPos = i;
            down = a[i];
        }
    }

    if (maxValPos == minValPos)
        return INF;

    if (down > up)
        std::swap(down, up);

    for (int i = 1; i <= n; ++i) {
        if (i == maxValPos || i == minValPos)
            continue;

        if (between(down, up, a[i]) || between(down, up, b[i]))
            continue;

        if (a[i] < down)
            down = a[i];
        else if (up < b[i])
            up = b[i];
    }

    updateUpAndDown(up, down, maxValPos, minValPos);

    return 1ll * (a[maxValPos] - b[minValPos]) * (up - down);
}

long long calcMaxMinColorNotSame() {
    int maxValPos = 0, minValPos = 0;
    int down, up;

    for (int i = 1; i <= n; ++i) {
        if (!maxValPos || a[maxValPos] < a[i]) {
            maxValPos = i;
            down = b[i];
        }

        if (!minValPos || b[minValPos] > b[i]) {
            minValPos = i;
            up = a[i];
        }
    }
    
    for (int i = 1; i <= n; ++i) {
        if (i == maxValPos || i == minValPos)
            continue;

        up = std::min(up, a[i]);
        down = std::max(down, b[i]);
    }

    return 1ll * (a[maxValPos] - up) * (down - b[minValPos]);
}

int main() {
    std::ios::sync_with_stdio(false);

    std::cin >> n;

    for (int i = 1; i <= n; ++i) {
        std::cin >> a[i] >> b[i];

        if (a[i] < b[i])
            std::swap(a[i], b[i]);
    }

    ans = calcMaxMinColorSame();
    ans = std::min(ans, calcMaxMinColorNotSame());
    std::cout << ans << std::endl;
    return 0;
}
```