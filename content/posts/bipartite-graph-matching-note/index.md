---
title: "二分图匹配学习笔记"
subtitle: ""
date: 2022-02-22T13:31:06+08:00
draft: false
author: "ctj12461"
authorLink: ""
description: ""
keywords: ""
license: "本文以 CC BY-NC-SA 4.0 许可证发布"
comment: false
weight: 0

tags:
  - OI
  - 图论
  - 二分图
  - 建模
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

## 二分图
### 定义
如果一个图 $G=(V,E)$ 中的结点可以被分为两个部分，且两个部分之内的点互相没有连边，则称这种图为二分图。如果每个结点的度数相等且都为 $k$，则称这种二分图为 $k$ - 正则二分图。

### 判断
对图的结点进行黑白染色，相连的结点染不同的颜色，若最后满足每条边的两个端点颜色不相同，则这个图是二分图。树也是二分图。

### 染色
这里的染色不同于判断的染色，它指的是对边进行染色。若染 $k$ 种颜色，则称其为二分图的 $k$ 染色。

对于一个 $k$ - 正则二分图来说，它一定可以被 $k$ 染色，而对于一般的二分图，若其最大度数为 $k$，则它也可以被 $k$ 染色。

### 匹配
在二分图的边集中选出一个非空子集，且这个非空子集内没有两条边连接了一个相同的端点，则这个非空子集被称为二分图的一个匹配，同样对于一般图也有类似的概念。

若一个匹配的所含的边数最大，则称这个匹配为最大匹配。

若二分图两边的结点个数相同，且存在一个匹配满足其中的边连接了所有的点，则这个匹配被称为这个二分图的完美匹配或者完备匹配。

若每条边有边权，且一个匹配的所含的边的权值和最大，则这个匹配被称为这个二分图的最大权匹配。

## 匈牙利算法
### 交替路
假设在二分图最大匹配的过程中，已经找到了一些边作为一个匹配，则一条交替路就是一个匹配边和非匹配边交替连接组成的简单路径。

根据二分图的定义，假设一条交替路的起点在左边，且第一条边是匹配边，则这条交替路中的匹配边一定都是从左边到右边，而非匹配边一定是从右边到左边。

交替路上的结点最多连接一条匹配边和一条非匹配边，所以如果把交替路上的匹配边变为非匹配边，非匹配边变为匹配边，则仍然满足条件，也就是交替路边集的匹配边集的补集也可以是匹配。

### 增广路
匈牙利算法的核心就是找增广路，这条增广路是一条交替路。每次增广从左边开始，第一条边是非匹配边，最后一条边也是非匹配边，根据上文的描述，这样的路径的边数为奇数，非匹配边个数比匹配边个数多 $1$，若把匹配边与非匹配边反转，则反转后的边集仍然是一个合法的匹配，且比原来的匹配的边数多了 $1$。如果能找到一条这样的增广路，则此次增广成功。

具体来说，比如一个左边的结点 $x$ 找到了一个右边的结点 $y$，它没有与其他左边的结点匹配过，则 $e(x,y)$ 可以作为一个匹配边，反转边集前，有 $1$ 条匹配边，$0$ 条非匹配边，增广成功。

如果右边的结点 $y$ 已经有匹配了，那么就可以让原来 $y$ 的匹配 $x'$ 新找一个点 $y'$，如果能够找到这个 $y'$，那么 $e(x',y')$ 成为一个匹配边，$y$ 就无需和 $x'$ 匹配了，也就是说 $y$ 已经变为了一个没有匹配的点，$x$ 就可以与 $y$ 匹配了。如果 $x'$ 找不到 $y'$，那么 $x'$ 就要保持原样，和 $y$ 匹配，那么 $x$ 就不可以和 $y$ 匹配，只能继续寻找下一个可能的结点。

对于 $x'$，它找 $y'$ 的过程和 $x$ 找 $y$ 的过程是一样的，所以可以递归实现。

总结一下，一个结点 $x$ 的寻找过程可以分为两种情况：
1. 找到一个未匹配结点 $y$，与它匹配
2. 让一个已匹配的结点 $y$ 和 $y$ 的另一边的结点 $x'$ 取消匹配，让 $x'$ 找新匹配点 $y'$，若取消成功，则 $x$ 与 $y$ 可以匹配

可以发现 $x$ 寻找成功，则匹配边都变成了非匹配边，原来的非匹配边都变成了匹配边，也就是找增广路的过程，然后对这条增广路的匹配边与非匹配边反转，匹配边数加 $1$。就这样对于左边的每个结点都增广，尝试找到左边的每个结点对应的匹配，最后就找到了最大匹配。

时间复杂度是 $O(|V||E|)$ 的，一般情况下跑不满这个上界。

### 代码实现
```cpp
bool augment(int x) {
    for (int y : graph[x]) {
        if (vis[y])
            continue;
        // 无论是否增广成功，对于当前增广路上的结点，下次都可以不用再走这个点
        vis[y] = true;
        // y 没有匹配或者取消 y 的匹配成功
        if (!match[y] || augment(match[y])) {
            match[y] = x;
            return true; // 找到匹配点，增广成功
        }
    }

    return false; // 没有找到结点，增广失败
}

int hungary() {
    int res = 0;

    for (int i = 1; i <= n; ++i)
        match[i] = 0;
    
    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= n; ++j) // 每次增广都要清空
            vis[j] = false;
        
        if (augment(i))
            ++res;
    }

    return res;
}
```

## 二分图最大匹配性质
### 相关概念
假设有一个图 $G=(V,E)$，下面是一些常见的概念：

- **边覆盖**：边覆盖是 $E$ 的一个子集 $E'$，$\forall\ x \in E$，满足 $x$ 是边 $e$ 的端点，且 $e \in E'$，也就是 $E'$ 中的边的端点的并集是 $V$
- **独立集**：独立集是 $V$ 的一个子集 $V'$，$\forall\ x,y \in V$，满足 $e(x,y) \notin E$，也就是独立集中的点互不相连
- **团**：团是图 $G$ 的一个子图 $G'=(V',E')$，$V' \subseteq V$，$E' \subseteq E$，且团是完全图，即 $\forall\ x,y \in V$，满足 $e(x,y) \in E'$，也就是团中的点互相连边
- **点覆盖**：点覆盖是 $V$ 的一个子集 $V'$，$\forall\ e(x,y) \in V$，满足 $x \in V'$ 或 $y \in V'$，也就是 $V'$ 中的点所连的边的并集是 $E$

### 等式
若一个图是二分图，则有如下性质：
- 若二分图不存在孤立的点，则 $|$最小边覆盖$| = |V| - |$最大匹配$|$
- $|$最大独立集$| = |$补图的最大团$| = |V| - |$最大匹配$|$
- $|$最小点覆盖$| = |$最大匹配$|$

### 与 DAG 的关系
对于一个 DAG $G$，构造二分图 $G'$，满足 $G$ 中的有向边 $e(x,y)$ 与 $G'$ 中的边 $e(x,y')$ 一一对应，其中 $x$ 和 $y'$ 在两边。此时满足 $|G$ 的最小路径覆盖$| = |G'$ 的最小点覆盖$|$。路径覆盖指选出一些路径 $p$ 组成一个集合 $P$，满足图中所有点都被至少一个 $p$ 经过，且 $p \in P$。

### 建模相关
可以通过一些转化把题目转化为二分图的问题，用上面的性质解决，详情见下面的例题。

## 例题
### Luogu P2055 假期的宿舍
> 学校放假了……有些同学回家了，而有些同学则有以前的好朋友来探访，那么住宿就是一个问题。
> 
> 比如 A 和 B 都是学校的学生，A 要回家，而 C 来看 B，C 与 A 不认识。我们假设每个人只能睡和自己直接认识的人的床。那么一个解决方案就是 B 睡 A 的床而 C 睡 B 的床。而实际情况可能非常复杂，有的人可能认识好多在校学生，在校学生之间也不一定都互相认识。
> 
> 我们已知一共有 $n$ 个人，并且知道其中每个人是不是本校学生，也知道每个本校学生是否回家。问是否存在一个方案使得所有不回家的本校学生和来看他们的其他人都有地方住。
>
> $T$ 组数据，$1\le T\le 20$，$1\le n\le 50$。

每个人只能睡一张床，一张床也只能被一个人睡，所以这就是一个二分图最大匹配的问题，把留在学校的学生和外校的学生放在左边，本校的学生的床放在右边，如果 A 和 B 是朋友，则它们可以睡对方的床（如果有），就把 A 向 B 的床连边，把 B 向 A 的床连边，求最大匹配即可。

```cpp
#include <iostream>
#include <vector>
using namespace std;

constexpr int MAX_N = 50 + 5;

int t, n;
bool local[MAX_N], home[MAX_N];
vector<int> graph[MAX_N];
int match[MAX_N];
bool vis[MAX_N];

void clear() {
    for (int i = 1; i <= n; ++i) {
        graph[i].clear();
        match[i] = 0;
    }
}

bool augment(int x) {
    for (int y : graph[x]) {
        if (vis[y])
            continue;

        vis[y] = true;

        if (!match[y] || augment(match[y])) {
            match[y] = x;
            return true;
        }
    }

    return false;
}

bool hungary() {
    int tot = 0, cnt = 0;

    for (int i = 1; i <= n; ++i) {
        if (local[i] && home[i])
            continue;

        ++tot;

        for (int j = 1; j <= n; ++j)
            vis[j] = false;

        if (augment(i))
            ++cnt;
    }

    return tot == cnt;
}

void solve() {
    cin >> n;

    for (int i = 1; i <= n; ++i)
        cin >> local[i];

    for (int i = 1; i <= n; ++i)
        cin >> home[i];

    clear();

    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= n; ++j) {
            bool fri;
            cin >> fri;

            if (fri) {
                if (local[j])
                    graph[i].push_back(j);

                if (local[i])
                    graph[j].push_back(i);
            }
        }

        if (local[i] && !home[i])
            graph[i].push_back(i);
    }

    cout << (hungary() ? "^_^" : "T_T") << endl;
}

int main() {
    ios::sync_with_stdio(false);

    cin >> t;

    while (t--)
        solve();

    return 0;
}
```

### Luogu P6268 舞会
> 某学校要召开一个舞会。已知学校所有 $n$ 名学生中，有 $m$ 对男生和女生互相跳过舞，一个男生或一个女生可能和多个人互相跳过舞。在这个舞会上，要求被邀请的学生中的任何一对男生和女生互相都不能跳过舞。求这个舞会最多能邀请多少个学生参加。
>
> $1\le n\le 1000$，$1\le m\le 2000$。

把跳过舞的男生和女生连边，因为不可能和同性跳舞，所以可以把这个图看成二分图，男生在左边，女生在右边，要求选出的同学之间没有互相跳过舞，显然同性可以满足这个条件，对于异性，只要没有互相跳过舞，也就是在图中没有连边，这个问题就是求二分图的最大独立集。建出图后先染色，分出二分图的两部分，没有连边的点可以分到任意一遍，然后利用 $|$最大独立集$| = |V| - |$最大匹配$|$ 求出答案即可。

```cpp
#include <iostream>
#include <vector>
using namespace std;

constexpr int MAX_N = 1000 + 10;
constexpr int MAX_M = 2000 + 10;

int n, m;
int col[MAX_N];
vector<int> tmp[MAX_N], graph[MAX_N];
bool vis[MAX_N];
int match[MAX_N];

void link(vector<int> graph[], int x, int y) {
    graph[x].push_back(y);
}

void coloring(int x, int now) {
    if (vis[x])
        return;

    col[x] = now;
    vis[x] = true;

    for (int y : tmp[x]) {
        if (!now)
            link(graph, x, y);
        
        coloring(y, now ^ 1);
    }
}

bool augment(int x) {
    for (int y : graph[x]) {
        if (vis[y])
            continue;

        vis[y] = true;

        if (!match[y] || augment(match[y])) {
            match[y] = x;
            return true;
        }
    }

    return false;
}

int hungary() {
    int res = 0;

    for (int i = 1; i <= n; ++i) {
        if (col[i])
            continue;
        
        for (int j = 1; j <= n; ++j)
            vis[j] = false;
        
        if (augment(i))
            ++res;
    }

    return res;
}

int main() {
    ios::sync_with_stdio(false);
    cin >> n >> m;

    for (int i = 1; i <= m; ++i) {
        int x, y;
        cin >> x >> y;
        ++x;
        ++y;
        link(tmp, x, y);
        link(tmp, y, x);
    }

    for (int i = 1; i <= n; ++i)
        if (!vis[i])
            coloring(i, 0);
        
    cout << n - hungary() << endl;
    return 0;
}
```

### Luogu P1129 矩阵游戏
> 小 Q 是一个非常聪明的孩子，除了国际象棋，他还很喜欢玩一个电脑益智游戏――矩阵游戏。矩阵游戏在一个 $n \times n$ 黑白方阵进行（如同国际象棋一般，只是颜色是随意的）。每次可以对该矩阵进行两种操作：
> - 行交换操作：选择矩阵的任意两行，交换这两行（即交换对应格子的颜色）。
> - 列交换操作：选择矩阵的任意两列，交换这两列（即交换对应格子的颜色）。
>
> 游戏的目标，即通过若干次操作，使得方阵的主对角线（左上角到右下角的连线）上的格子均为黑色。
>
> 对于某些关卡，小 Q 百思不得其解，以致他开始怀疑这些关卡是不是根本就是无解的！于是小 Q 决定写一个程序来判断这些关卡是否有解。
>
> $T$ 组数据，$1\le T\le 20$，$1\le n\le 200$。

首先，如果有两个黑色格子在同一行或同一列，则不可能通过任何操作将它们拆分到不同行或不同列，又因为主对角线上的格子两两不在同一行也不在同一列，则游戏有解的条件就是存在 $n$ 个黑色格子两两不在同一行也不在同一列。由于在同一行或同一列的格子最多只能对答案有 $1$ 的贡献，所以可以强制只选择一行或一列中的一个格子。

考虑把每一行建一个结点放在左边，用 $x$ 表示，每一列建一个结点放在右边，用 $y'$表示，如果格子 $x,y$ 为黑色，则连一条边 $e(x,y')$。这样做一个二分图最大匹配，就满足了每一行或每一列最多有 $1$ 的贡献的条件，如果最后存在完美匹配，则游戏有解，否则无解。

```cpp
#include <iostream>
#include <vector>
using namespace std;

constexpr int MAX_N = 200 + 10;

int t, n;
vector<int> graph[MAX_N];
bool vis[MAX_N];
int match[MAX_N];

void link(int x, int y) {
    graph[x].push_back(y);
}

void clear() {
    for (int i = 1; i <= n; ++i) {
        graph[i].clear();
        match[i] = 0;
    }
}

bool augment(int x) {
    for (int y : graph[x]) {
        if (vis[y])
            continue;

        vis[y] = true;

        if (!match[y] || augment(match[y])) {
            match[y] = x;
            return true;
        }
    }

    return false;
}

int hungary() {
    int res = 0;

    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= n; ++j)
            vis[j] = false;
        
        if (augment(i))
            ++res;
    }

    return res;
}

void solve() {
    cin >> n;
    clear();

    for (int i = 1; i <= n; ++i) {
        for (int j = 1; j <= n; ++j) {
            int col;
            cin >> col;

            if (col)
                link(i, j);
        }
    }

    cout << (hungary() == n ? "Yes" : "No") << endl;
}

int main() {
    ios::sync_with_stdio(false);
    
    cin >> t;

    while (t--)
        solve();
        
    return 0;
}
```

### Luogu P1963 变换序列
> 给出一个序列 $0,1,2,\dots, n-1$，一个变换序列 $T$ 可以 $i$ 变为 $T_i$，$T$ 可以视为一个排列且 $T_i \in [0,n-1]$。$\forall\ x,y \in [0,n-1]$，定义它们的距离 $D(x,y)=\min(|x-y|,n-|x-y|)$，即把 $x,y$ 看成环上的点时的距离。给出每个 $D(i,T_i)$，求出一个满足条件的序列 $T$ 且字典序最小，或判断无解。
> 
> $1\le n\le 10000$。

设 $d_i=D(i,T_i)$，则 $i$ 对于的可能的 $T_i$ 为 $x-d_i,x+d_i,x-(n-d_i),x+(n-d_i)$ 且满足在 $[0,n-1]$ 之内。同时注意判断这些 $T_i$ 带入 $D(i,T_i)$ 的计算式时是否能够满足条件，即 $d_i\le n-d_i$，如果不能满足，那么这就不是合法的 $D(i,T_i)$，也就无解了。

然后就是二分图最大匹配了，每个 $i$ 从左边向右边可能的 $T_i$ 连边，可以先求出一个可行解。在求可行解的过程中，为了能够让某些点能够匹配，我们断开了一些已匹配点的匹配关系，所以这个可行解不一定满足字典序最小。

我们可以再跑一遍最大匹配，让从前往后贪心地让每个点先断开已有的匹配，重新选择编号更小的点，这个过程就是增广的过程，同时根据贪心的思想，更前面的点选择的编号更小一定比后面选这个编号更优，如果在重新增广的过程中遇到要更改更前面的点的匹配时，直接退出即可，即使现在无法换到更小的编号，也能够保证总体上的答案更优。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

constexpr int MAX_N = 1e4 + 10;

int n, d[MAX_N];
std::vector<int> graph[MAX_N];
bool vis[MAX_N], fixed[MAX_N];
int match[MAX_N], ans[MAX_N];

inline void link(int x, int y) {
    graph[x].push_back(y);
}

bool augment(int x) {
    for (int y : graph[x]) {
        if (vis[y])
            continue;
        
        vis[y] = true;

        if (match[y] == -1 || augment(match[y])) {
            match[y] = x;
            return true;
        }
    }

    return false;
}

bool exchange(int x) {
    if (fixed[x])
        return false;
    
    for (int y : graph[x]) {
        if (vis[y])
            continue;
        
        vis[y] = true;

        if (match[y] == -1 || exchange(match[y])) {
            match[y] = x;
            return true;
        }
    }

    return false;
}

bool hungary() {
    int res = 0;

    for (int i = 0; i < n; ++i)
        match[i] = -1;

    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < n; ++j)
            vis[j] = false;
        
        if (augment(i))
            ++res;
    }

    if (res != n)
        return false;
    
    for (int i = 0; i < n; ++i) {
        int other = -1;

        for (int j = 0; j < n; ++j) {
            vis[j] = false;
            
            if (match[j] == i)
                other = j;
        }

        match[other] = -1;

        if (!exchange(i))
            match[other] = i;
        
        fixed[i] = true;
    }

    return true;
}

int main() {
    std::ios::sync_with_stdio(false);
    
    std::cin >> n;

    for (int i = 0; i < n; ++i) {
        std::cin >> d[i];
        
        if (d[i] > n - d[i])
            continue;
        
        if (i - d[i] >= 0)
            link(i, i - d[i]);
        
        if (i + d[i] < n)
            link(i, i + d[i]);
        
        if (i - (n - d[i]) >= 0)
            link(i, i - (n - d[i]));
        
        if (i + (n - d[i]) < n)
            link(i, i + (n - d[i]));
    }

    if (hungary()) {
        for (int i = 0; i < n; ++i)
            ans[match[i]] = i;
        
        for (int i = 0; i < n; ++i)
            std::cout << ans[i] << " ";
        
        std::cout << std::endl;
    } else {
        std::cout << "No Answer" << std::endl;
    }

    return 0;
}
```

### UVa 1663 Purifying Machine
> 给出 $m$ 个长度为 $n$ 的模板 `01` 串，某些串含有至多一个 `*`，表示此处可以匹配 `0` 或 `1`，根据这些模板构造出一个数字集合，满足其中的数字的二进制表示都为 $n$ 位，且能够匹配每一个模板串，要求你构造一些 `01` 串，最多也可以含有至多一个 `*`，使得这些数字能够和至少一个 `01` 串匹配，且该集合外的数字均不能和任意一个 `01` 串匹配，求最小的 `01` 串个数。
>
> $1\le n\le 10$，$1\le m\le 1000$。

容易证明不存在 $3$ 个及以上的数字在二进制表示下只有 $1$ 位互不相同，所以可以把每个数字看作结点，每个含 `*` 的 `01` 串看作连接两个匹配的数字的结点的边，则这个图是二分图。

但是这个二分图并不一定连通，对于那些孤立的结点（数字），只能用固定的 `01` 串取匹配它们。对于剩下的连通的结点，要有最少的含 `*` 的 `01` 串匹配每个结点，相当于选出最少的边，使得每个结点都是这个边集中的某条边的端点，这就是最小边覆盖问题。

先对二分图染色，做一遍最大匹配，再利用 $|$最小边覆盖$| = |V| - |$最大匹配$|$ 求出连通部分的答案，再加上孤立部分的答案就是总答案了。

```cpp
#include <iostream>
#include <vector>
using namespace std;

const int MAX_N = 10;
const int MAX_NODE = (1 << MAX_N);

int n, m;
vector<int> tmp[MAX_NODE], graph[MAX_NODE];
int match[MAX_NODE];
bool selected[MAX_NODE], exist[MAX_NODE], vis[MAX_NODE], col[MAX_NODE];
int selectedTot, graphTot;
char str[MAX_N + 1];

void link(vector<int> graph[], int x, int y) {
    graph[x].push_back(y);
}

bool augment(int x) {
    for (int i = 0; i < (int) graph[x].size(); ++i) {
        int y = graph[x][i];

        if (vis[y])
            continue;

        vis[y] = true;

        if (match[y] == -1 || augment(match[y])) {
            match[y] = x;
            return true;
        }
    }

    return false;
}

int hungary() {
    int res = 0;

    for (int i = 0; i < (1 << n); ++i)
        match[i] = -1;

    for (int i = 0; i < (1 << n); ++i) {
        if (!exist[i] || col[i])
            continue;

        for (int j = 0; j < (1 << n); ++j)
            vis[j] = false;

        if (augment(i))
            ++res;
    }

    return res;
}

void coloring(int x, bool now) {
    if (vis[x])
        return;

    col[x] = now;
    vis[x] = true;

    for (int i = 0; i < (int) tmp[x].size(); ++i) {
        int y = tmp[x][i];

        if (!now)
            link(graph, x, y);

        coloring(y, !now);
    }
}

int convert(char str[], int len) {
    int res = 0;

    for (int i = len - 1; i >= 0; --i)
        res = (res << 1) + (str[i] - '0');

    return res;
}

void clear() {
    selectedTot = graphTot = 0;

    for (int i = 0; i < (1 << n); ++i) {
        vis[i] = selected[i] = exist[i] = col[i] = false;
        tmp[i].clear();
        graph[i].clear();
    }
}

void input() {
    cin >> n >> m;
    clear();

    for (int i = 1; i <= m; ++i) {
        cin >> str;

        for (int j = 0, k = n - 1; j < k; ++j, --k)
            swap(str[j], str[k]);

        int pos = -1;

        for (int j = 0; j < n; ++j) {
            if (str[j] == '*') {
                pos = j;
                break;
            }
        }

        if (pos == -1) {
            int id = convert(str, n);
            selected[id] = true;
        } else {
            str[pos] = '0';
            int id = convert(str, n);
            selected[id] = true;
            str[pos] = '1';
            id = convert(str, n);
            selected[id] = true;
            str[pos] = '*';
        }
    }
}

void build() {
    for (int i = 0; i < (1 << n); ++i)
        if (selected[i])
            ++selectedTot;

    for (int i = 0; i < (1 << n); ++i) {
        for (int j = 0; j < n; ++j) {
            int id1 = (i & (~(1 << j))), id2 = (i | (1 << j));

            if (selected[id1] && selected[id2]) {
                exist[id1] = exist[id2] = true;
                link(tmp, id1, id2);
                link(tmp, id2, id1);
            }
        }
    }

    for (int i = 0; i < (1 << n); ++i) {
        if (!exist[i])
            continue;

        ++graphTot;
        coloring(i, false);
    }
}

bool solve() {
    input();

    if (n == 0 && m == 0)
        return true;

    build();

    cout << ((selectedTot - graphTot) + (graphTot - hungary())) << endl;
    return false;
}

int main() {
    ios::sync_with_stdio(false);

    bool end = false;

    do
        end = solve();
    while (!end);

    return 0;
}
```