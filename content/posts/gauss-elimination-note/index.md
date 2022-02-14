---
title: "高斯消元学习笔记"
subtitle: ""
date: 2022-02-06T16:06:35+08:00
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

## 算法思想
### 增广矩阵
为了更方便地求解方程组，可以将系数和常数项放入矩阵，接下来就可以用一些矩阵的操作来消元了。

比如有一个方程组如下：

$$
\begin{cases}
3x+2y+3z=10\\\\
3x+y+4z=12\\\\
x+y+z=4
\end{cases}
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

### 初等行变换
初等行变换是指行之间的加减乘等运算。消元的本质就是利用初等行变换，使未知数的系数变为 $0$。具体地，初等行变换是两个行中的元素逐一进行运算，以上面的矩阵为例，记 $row_i$ 表示第 $i$ 行，则 $row_2 \longleftarrow row_2 - \frac{2}{3}row_1$ 的结果就是：

$$
\begin{bmatrix}
0 & -2 & -\frac{1}{2} & -3
\end{bmatrix}
$$

这个操作使该行的第 $1$ 个元素变为了 $0$，从方程的意义上考虑，就是消去了一个元。

### 三角矩阵和对角矩阵
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

### 具体步骤与实例
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

**Step 2:** $row_3 \longleftarrow row_3 - \frac{1}{2}row_1$
$$
\begin{bmatrix}
2 & 2 & 3 & 10 \\\\
0 & -2 & -\frac{1}{2} & -3 \\\\
0 & 0 & -\frac{1}{2} & -4
\end{bmatrix}
$$

**Step 3:** $row_3 \longleftarrow row_3 - 0\cdot row_2$

增广矩阵不变。

### 高斯-约旦消元
由于计算机上浮点数的精度有限，要尽可能选择绝对值较大的数作为除数，所以枚举到第 $i$ 行时，可以找到 $x_i$ 的系数最大的方程，与当前行交换，这种方法叫做高斯-约旦消元法。

### 解的判断
方程组有可能有无数解（含自由元）或无解。得到一个对角矩阵后，若存在对角元素 $k_{ii} = 0$，而常数项 $c_i \ne 0$，则无解，否则若存在对角元素 $k_{ii} = 0$，而常数项 $c_i = 0$，则含自由元。

## 代码实现
```cpp
class EquationGroup {
public:
    enum class Result {
        UNIQUE, INFINITY, NO_SOLUTION
    };

    void init(int n) {
        this->n = n;
        e = vector<vector<double>>(n + 1, vector<double>(n + 2));
    }

    vector<double> &operator[](int x) {
        return e[x];
    }

    pair<vector<double>, Result> solve() {
        transform();
        Result res = calc();
        vector<double> x(n + 1);

        for (int i = 1; i <= n; ++i)
            x[i] = e[i][n + 1];
        
        return {x, res};
    }
private:
    vector<vector<double>> e;
    int n; // 未知数个数

    void transform() {
        for (int i = 1; i <= n; ++i) {
            int p = i;
            // 选择 e[][i] 绝对值最大的一行进行交换
            for (int j = i + 1; j <= n; ++j)
                if (abs(e[p][i]) < abs(e[j][i]))
                    p = j;
            
            if (p != i)
                for (int j = 1; j <= n + 1; ++j)
                    swap(e[i][j], e[p][j]);
            // 初等行变换
            for (int j = i + 1; j <= n; ++j) {
                double rate = e[j][i] / e[i][i];

                for (int k = 1; k <= n + 1; ++k)
                    e[j][k] -= e[i][k] * rate;
            }
        }
    }

    Result calc() {
        Result res = Result::UNIQUE;
        
        for (int i = n; i >= 1; --i) {
            // 把解 x_i 存在 e[i][n + 1] 中
            for (int j = i + 1; j <= n; ++j)
                e[i][n + 1] -= e[i][j] * e[j][n + 1];
            
            if (!e[i][i]) {
                e[i][n + 1] /= e[i][i];
            } else {
                if (e[i][n + 1])
                    res = Result::NO_SOLUTION;
                else if (res != Result::NO_SOLUTION)
                    res = Result::INFINITY;
            }
        }

        return res;
    }
};
```

## 例题
### Luogu P4035 球形空间产生器
> 给出一个 $n + 1$ 个 $n$ 维空间的点，求这 $n + 1$ 个点构成的 $n$ 维球体的球心。
> 
> $1 \le n \le 10$。

设球心为 $(x_1,x_2,\dots,x_n)$，半径为 $r$，对于球体的某个点 $p$，有如下关系：

$$
\sum_{i = 1}^{n} (p_i - x_i) ^ 2 = r ^ 2\\\\
$$ 

展开并移项得到：

$$
\sum_{i = 1}^{n} 2p_ix_i + (r ^ 2 - \sum_{i = 1}^{n} x_i ^ 2) = \sum_{i = 1}^{n} p_i ^ 2
$$

把括号看成一个整体，显然是一个 $n + 1$ 元线性方程组，直接高斯消元求解。

```cpp
#include <iostream>
#include <iomanip>
#include <cmath>
using namespace std;

constexpr int MAXN = 10 + 5;

int n;
double pos[MAXN][MAXN], mat[MAXN][MAXN];

void preprocess() {
    for (int i = 1; i <= n + 1; ++i) {
        double sum = 0;

        for (int j = 1; j <= n; ++j) {
            mat[i][j] = 2 * pos[i][j];
            sum += pos[i][j] * pos[i][j];
        }
        
        mat[i][n + 1] = 1;
        mat[i][n + 2] = sum;
    }
}

void transform(int n) {
    for (int i = 1; i <= n; ++i) {
        int p = i;

        for (int j = i + 1; j <= n; ++j)
            if (abs(mat[p][i]) < abs(mat[j][i]))
                p = j;
        
        if (i != p)
            for (int j = i; j <= n + 1; ++j)
                swap(mat[i][j], mat[p][j]);

        for (int j = i + 1; j <= n; ++j) {
            double rate = mat[j][i] / mat[i][i];

            for (int k = i; k <= n + 1; ++k)
                mat[j][k] -= rate * mat[i][k];
        }
    }
}

void calc(int n) {
    for (int i = n; i >= 1; --i) {
        for (int j = i + 1; j <= n; ++j)
            mat[i][n + 1] -= mat[i][j] * mat[j][n + 1];
        
        mat[i][n + 1] /= mat[i][i];
    }
}

void guass(int n) {
    transform(n);
    calc(n);
}

int main() {
    ios::sync_with_stdio(false);

    cin >> n;

    for (int i = 1; i <= n + 1; ++i)
        for (int j = 1; j <= n; ++j)
            cin >> pos[i][j];

    preprocess();
    guass(n + 1);

    for (int i = 1; i <= n; ++i)
        cout << fixed << setprecision(3) << mat[i][n + 2] << " ";

    cout << endl;
    return 0;
}
```

### Codeforces 24D Broken robot
> $n$ 行 $m$ 列的矩阵，$(1, 1)$ 是矩阵的左上角，$(n, m)$ 是矩阵的右下角。现在你在 $(x, y)$，每次等概率向左，右，下走或原地不动，但不能走出去，问走到最后一行期望的步数。（原地不动也算一步）
> 
> $1 \le n, m \le 10 ^ 3$，$1 \le x \le n$，$1 \le y \le m$。

先考虑 $m = 1$ 的情况，有 $\frac{1}{2}$ 的概率留在原地，$\frac{1}{2}$ 的概率向下走，期望步数为 $2 (m - x)$。

若 $m > 1$，设 $f[i][j]$ 为从 $(i, j)$ 走到最后一行的期望步数，则

$$
f[i][j] = 
\begin{cases}
\frac{f[i][j + 1] + f[i + 1][j] + f[i][j]}{3} + 1 & j = 1\\\\
\frac{f[i][j - 1] + f[i][j + 1] + f[i + 1][j] + f[i][j]}{4} + 1 & 1 < j < m\\\\
\frac{f[i][j - 1] + f[i + 1][j] + f[i][j]}{3} + 1 & j = m
\end{cases}
$$

移项可得：

$$
\begin{cases}
2f[i][j] - f[i][j + 1] - f[i + 1][j] = 3 & j = 1\\\\
3f[i][j] - f[i][j - 1] - f[i][j + 1] - f[i + 1][j] = 4 & 1 < j < m\\\\
2f[i][j] - f[i][j - 1] - f[i + 1][j] = 3 & j = m
\end{cases}
$$

于是每行的转移就可以解决了。然而暴力消元的复杂度不正确，考虑到每个方程都至多只有三个系数不为 $0$，每一行只会消去下一行的一个元（可以手画一个矩阵），利用这个性质，消元可以做到 $\Theta(m)$，总的复杂度就是 $\Theta(m (n-x))$。

```cpp
#include <iostream>
#include <iomanip>
using namespace std;

constexpr int MAXN = 1e3 + 5;

// * * 0 0 0 0 | *
// * * * 0 0 0 | *
// 0 * * * 0 0 | *
// 0 0 * * * 0 | *
// 0 0 0 * * * | *
// 0 0 0 0 * * | *

int n, m, x, y;
double mat[MAXN][MAXN], f[MAXN];

void fill() {
    mat[1][1] = 2;
    mat[1][2] = -1;
    mat[1][m + 1] = 3 + f[1];

    for (int i = 2; i < m; ++i) {
        mat[i][i] = 3;
        mat[i][i - 1] = -1;
        mat[i][i + 1] = -1;
        mat[i][m + 1] = 4 + f[i];
    }

    mat[m][m] = 2;
    mat[m][m - 1] = -1;
    mat[m][m + 1] = 3 + f[m];
}

void gauss() {
    for (int i = 1; i < m; ++i) {
        double rate = mat[i + 1][i] / mat[i][i];
        mat[i + 1][i] = 0;
        mat[i + 1][i + 1] -= rate * mat[i][i + 1];
        mat[i + 1][m + 1] -= rate * mat[i][m + 1];
    }

    mat[m][m + 1] /= mat[m][m];

    for (int i = m - 1; i >= 1; --i)
        mat[i][m + 1] = (mat[i][m + 1] - mat[i + 1][m + 1] * mat[i][i + 1]) / mat[i][i];
    
    for (int i = 1; i <= m; ++i)
        f[i] = mat[i][m + 1];
}

int main() {
    ios::sync_with_stdio(false);
    cin >> n >> m >> x >> y;
    
    if (m == 1) {
        cout << 2 * (n - x) << endl;
        return 0;
    }

    for (int i = n - 1; i >= x; --i) {
        fill();
        gauss();
    }

    cout << setprecision(8) << f[y] << endl;
    return 0;
}
```

### Luogu P3232 游走
> 给定一个 $n$ 个点 $m$ 条边的无向连通图，顶点从 $1$ 编号到 $n$，边从 $1$ 编号到 $m$。
> 
> 小 Z 在该图上进行随机游走，初始时小 Z 在 $1$ 号顶点，每一步小 Z 以相等的概率随机选择当前顶点的某条边，沿着这条边走到下一个顶点，获得等于这条边的编号的分数。当小 Z 到达 $n$ 号顶点时游走结束，总分为所有获得的分数之和。现在，请你对这 $m$ 条边进行编号，使得小 Z 获得的总分的期望值最小。
> 
> $2 \le n \le 500$，$1 \le m \le 125000$。

设 $f[i]$ 表示走到 $i$ 的期望次数，$deg[i]$ 为点 $x$ 的度，则对于一条边 $(x, y)$，其期望次数为：

$$
\frac{f[x]}{deg[x]} + \frac{f[y]}{deg[y]}
$$

考虑 $f[i]$ 的转移，设 $j$ 为 $i$ 的上一个结点，则：

$$
f[i] = \sum\frac{f[j]}{deg[j]} + [i = 1]
$$

因为初始在点 $1$，则期望次数会多 $1$。把这个方程，由于 $n$ 是终点，不参加转移，设 $f[n] = 0$。于是前 $n - 1$ 个转移方程组成一个方程组，解出它即可。

最后算出每条边的期望次数，根据排序不等式，次数越多的边应该编号越小，这样答案更优。

```cpp
#include <iostream>
#include <iomanip>
#include <cmath>
#include <vector>
#include <algorithm>
using namespace std;

constexpr int MAXN = 500 + 10;
constexpr int MAXM = 125000 + 10;

struct Equation {
    double a[MAXN];

    double &operator[](int x) {
        return a[x];
    }
};

struct EquationGroup {
    Equation e[MAXN];  
    int n;

    void init(int n) {
        this->n = n;
    }

    Equation &operator[](int x) {
        return e[x];
    }

    void transform() {
        for (int i = 1; i <= n; ++i) {
            int p = i;
            
            for (int j = i + 1; j <= n; ++j)
                if (abs(e[p][i]) < abs(e[j][i]))
                    p = j;
            
            if (p != i)
                for (int j = 1; j <= n + 1; ++j)
                    swap(e[i][j], e[p][j]);
            
            for (int j = i + 1; j <= n; ++j) {
                double rate = e[j][i] / e[i][i];

                for (int k = 1; k <= n + 1; ++k)
                    e[j][k] -= e[i][k] * rate;
            }
        }
    }

    void calc() {
        for (int i = n; i >= 1; --i) {
            for (int j = i + 1; j <= n; ++j)
                e[i][n + 1] -= e[i][j] * e[j][n + 1];
            
            e[i][n + 1] /= e[i][i];
        }
    }

    vector<double> solve() {
        transform();
        calc();
        vector<double> res(n + 1);

        for (int i = 1; i <= n; ++i)
            res[i] = e[i][n + 1];
        
        return res;
    }
};

struct Edge {
    int x, y;
    double p;

    bool operator<(const Edge &rhs) const {
        return p > rhs.p;
    }
};

int n, m;
EquationGroup e;
vector<int> graph[MAXN];
int deg[MAXN];
Edge edges[MAXM];
double ans;

void link(int x, int y) {
    graph[x].push_back(y);
    graph[y].push_back(x);
    ++deg[x];
    ++deg[y];
}

int main() {
    ios::sync_with_stdio(false);
    cout << fixed << setprecision(3);
    cin >> n >> m;

    for (int i = 1; i <= m; ++i) {
        auto &[x, y, p] = edges[i];
        cin >> x >> y;
        link(x, y);
    }

    e.init(n - 1);

    for (int i = 1; i < n; ++i) {
        for (int j : graph[i])
            if (j != n)
                e[i][j] = -1.0 / deg[j];
        
        e[i][i] = 1;
    }

    e[1][n] = 1;

    auto f = e.solve();

    for (int i = 1; i <= m; ++i) {
        auto &[x, y, p] = edges[i];
        p = (x != n ? f[x] / deg[x] : 0) + (y != n ? f[y] / deg[y] : 0);
    }

    sort(edges + 1, edges + 1 + m);

    for (int i = 1; i <= m; ++i)
        ans += i * edges[i].p;

    cout << ans << endl;
    return 0;
}
```

### Luogu P3211 XOR 和路径
> 给定一个无向连通图，有 $n$ 个点和 $m$ 条边，其节点编号为 $1$ 到 $n$，其边的权值为非负整数。从 $1$ 号节点开始，以相等的概率随机选择一个有连边的点，并沿这条边走到下一个节点，重复这个过程，直到走到 $n$ 号节点为止，便得到一条从 $1$ 号节点到 $n$ 号节点的路径，请你求出该算法得到的路径的 XOR 和的期望值。
>
> 设 $(x, y, w)$ 为图中的一条边，$1 \le x, y \le n$，$0 \le w \le 10 ^ 9$，$2 \le n \le 100$，$1 \le m \le 10000$。图中可能有重边或自环。

首先可能想到设 $f[x]$ 为 $x$ 点的 XOR 期望，方程为

$$
f[x] = \sum_y \frac{f[y] \oplus w(x, y)}{deg[x]}
$$

但是这种方程是无法转移的。先考虑答案是怎么得到的假设有一条路径 $P = \\{w_1, w_2, \dots, w_k\\}$，其对答案的贡献为：

$$
p(w_1 \oplus w_2 \oplus \cdots \oplus w_k)
$$

如果按位考虑，分子的结果只会是 $0$ 或 $1$，只需要求出分子的结果为 $1$ 的概率。

设 $f[x][0/1]$ 为点 $x$ 到点 $n$ 的路径异或值为 $0/1$ 的概率。于是可以写成方程：

$$
f[x][0] = \sum_y \frac{f[y][w(x, y)]}{deg[x]}\\\\
f[x][1] = \sum_y \frac{f[y][w(x, y) \oplus 1]}{deg[x]}\\\\
$$

设当前枚举的位数为 $b$，则对答案的贡献为 $2 ^ b f[1][1]$。上述转移显然可以高斯消元解决，时间复杂度 $\Theta(n ^ 3 \log_2 w)$。

注意连接自环不要连两遍。

```cpp
#include <iostream>
#include <vector>
#include <cmath>
#include <iomanip>
using namespace std;

constexpr int MAXN = 100 + 10;

class EquationGroup {
public:
    void init(int n) {
        this->n = n;
        e = vector<vector<long double>>(n + 1, vector<long double>(n + 2, 0));
    }

    vector<long double> &operator[](int x) {
        return e[x];
    }

    vector<long double> solve() {
        transform();
        calc();
        vector<long double> res(n + 1);

        for (int i = 1; i <= n; ++i)
            res[i] = e[i][n + 1];
        
        return res;
    }
private:
    int n;
    vector<vector<long double>> e;

    void transform() {
        for (int i = 1; i <= n; ++i) {
            int p = i;

            for (int j = i + 1; j <= n; ++j)
                if (abs(e[p][i]) < abs(e[j][i]))
                    p = j;
            
            if (p != i)
                for (int j = i; j <= n + 1; ++j)
                    swap(e[i][j], e[p][j]);
            
            for (int j = i + 1; j <= n; ++j) {
                long double rate = e[j][i] / e[i][i];

                for (int k = i; k <= n + 1; ++k)
                    e[j][k] -= rate * e[i][k];
            }
        }
    }

    void calc() {
        for (int i = n; i >= 1; --i) {
            for (int j = i + 1; j <= n; ++j)
                e[i][n + 1] -= e[i][j] * e[j][n + 1];

            e[i][n + 1] /= e[i][i];
        }
    }
};

int n, m, mx;
EquationGroup e;
vector<pair<int, int>> graph[MAXN];
int deg[MAXN];
long double ans;

void link(int x, int y, int w) {
    graph[x].push_back({y, w});
    ++deg[x];

    if (x != y) {
        graph[y].push_back({x, w});
        ++deg[y];
    }
}

long double calc(int b) {
    e.init(2 * n);

    for (int x = 1; x < n; ++x) {
        e[x][x] += deg[x];
        e[x + n][x + n] += deg[x];

        for (auto [y, w] : graph[x]) {
            int w1 = ((w >> b) & 1);
            e[x][y + w1 * n] += -1;
            e[x + n][y + (w1 ^ 1) * n] += -1;
        }
    }

    e[n][n] = 1;
    e[n][2 * n + 1] = 1;
    e[n + n][2 * n] = 1;
    e[n + n][2 * n + 1] = 0;

    auto f = e.solve();
    return f[1 + n] * (1 << b);
}

int main() {
    ios::sync_with_stdio(false);
    cout << fixed << setprecision(3);
    cin >> n >> m;

    for (int i = 1; i <= m; ++i) {
        int x, y, w;
        cin >> x >> y >> w;
        link(x, y, w);
        mx = max(mx, w);
    }

    mx = (int) log2(mx);
    
    for (int i = 0; i <= mx; ++i)
        ans += calc(i);
    
    cout << ans << endl;
    return 0;
}
```

