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
给出 $n + 1$ 个点 $(x_1,y_1),(x_2,y_2),\dots,(x_n,y_n),(x_{n+1},y_{n+1})$，要求出过这些点的 $n$ 次多项式函数（也称该函数的度为 $n$），先考虑对每个点构造一个函数，第 $i$ 个点的函数为 $f_i(x)$，满足：

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

接下来就是找到一种合适的形式来表示 $f_i(x)$ 的分段规定，可以让 $f_i(x)$ 含有一些因式，满足 $x=x_j,j\ne i$ 时，其值为 $0$，$x=x_i$ 时，其值为 $y_i$。显然下面这种满足条件：

$$
f_i(x) = y_i \prod_{i\ne j} \frac{x-x_j}{x_i-x_j}
$$

所以最终的 $f(x)$ 的解析式为：

$$
f(x) = \sum_{i=1}^{n + 1} y_i \prod_{i\ne j} \frac{x-x_j}{x_i-x_j}
$$

求单个函数值的时间复杂度是 $\Theta(n^2)$。

### 前后缀积优化
在一些题目中，插值所需的 $x$ 是连续的，当 $x\in \mathrm{N^*}$ 时，我们可以把公式写成这样：

$$
f(x) = \sum_{i=1}^{n + 1} y_i \prod_{i\ne j} \frac{x-j}{i-j} 
= \sum_{i=1}^{n + 1} y_i \frac{pre[i - 1]suf[i+1]}{(i-1)!(n-i+1)!(-1)^{n-i+1}}\\\\
pre[i]=\prod_{j=1}^{i} (k-j) = (k-i)pre[i-1]\\\\
suf[i]=\prod_{j=i}^{n+1} (k-j)= (k-i)suf[i+1]\\\\
$$

特殊地，$pre[0] = 1$，$suf[n+2] = 1$。

这样单次插值的时间复杂度就是 $\Theta(n)$。
当然，前后缀积优化还适用于 $x_i-x_{i-1}=d\ (d\ne 0,x>1)$ 的情况，即 $x_i$ 是一个公差不为 $0$ 的等差数列。

### 重心拉格朗日插值
如果题面要求动态地加入插值点升高次数，上述方法不够优秀，从公式中可以发现，我们重复计算了许多东西，考虑把它们提取出来：

$$
f(x) = \sum_{i=1}^{n + 1} y_i \prod_{i\ne j} \frac{x-x_j}{x_i-x_j}
= \sum_{i=1}^{n + 1} \frac{y_i}{x-x_i} \prod_{j=1}^{n+1}(x-x_j) \prod_{i\ne j}\frac{1}{x_i-x_j}
$$

定义以下函数：

$$
w(i)=\prod_{i\ne j}\frac{1}{x_i-x_j}\\\\
c(x)=\prod_{j=1}^{n+1}(x-x_j)
$$

其中 $w(i)$ 也被称为重心权。

最终公式为：

$$
f(x)=c(x) \sum_{i=1}^{n + 1} w(i) \frac{y_i}{x-x_i}
$$

显然只要 $\Theta(1)$ 更新 $c(x)$，$\Theta(nt)$ 求出 $w(n+2)$ 以及更新 $w(1\dots n+1)$。最后 $\Theta(nt)$ 求出新的 $f(x)$。$\Theta(t)$ 为求逆元的复杂度，一般为 $\Theta(\log_2 M)$，$M$ 为模数。

具体地，对于 $\forall i\in [1,n+1]$，$w(i)$ 都除以 $x_i-x_{n+2}$，而 $w(n+2)$ 只需扫一遍即可。

## 代码实现
### 普通拉格朗日插值
```cpp
int interpolation(int deg, int k, int x[], int y[]) {
    int res = 0;

    for (int i = 1; i <= deg + 1; ++i) {
        int num = y[i], den = 1;

        for (int j = 1; j <= deg + 1; ++j) {
            if (i == j)
                continue;

            num = 1ll * num * (k - x[j] + MOD) % MOD;
            den = 1ll * den * (x[i] - x[j] + MOD) % MOD;
        }

        res = (res + 1ll * num * power(den, MOD - 2)) % MOD;
    }

    return res;
}
```

### 前后缀积优化
```cpp
inline int sgnInv(int n) {
    static int inv[2] = {power(1, MOD - 2), power(MOD - 1, MOD - 2)};
    return inv[n % 2];
}

void preprocess(int deg) {
    fac[0] = 1;

    for (int i = 1; i <= deg + 1; ++i)
        fac[i] = 1ll * fac[i - 1] * i % MOD;
    
    inv[deg + 1] = power(fac[deg + 1], MOD - 2);

    for (int i = deg; i >= 0; --i)
        inv[i] = 1ll * inv[i + 1] * (i + 1) % MOD;
}

int interpolation(int deg, int k, int x[], int y[]) {
    int res = 0;

    pre[0] = suf[deg + 2] = 1;

    for (int i = 1; i <= n + 1; ++i)
        pre[i] = 1ll * pre[i - 1] * (k - i + MOD) % MOD;
    
    for (int i = n + 1; i >= 1; --i)
        suf[i] = 1ll * suf[i + 1] * (k - i + MOD) % MOD;

    for (int i = 1; i <= n + 1; ++i) {
        int num = 1ll * y[i] * pre[i - 1] % MOD * suf[i + 1] % MOD;
        int den = 1ll * inv[i - 1] * inv[deg - i + 1] % MOD * sgnInv(deg - i + 1) % MOD % MOD;
        res = (1ll * res + 1ll * num * den % MOD) % MOD;
    }

    return res
}
```

### 重心拉格朗日插值
```cpp
int interpolationAdd(int nx, int ny) {
    ++deg;
    x[deg + 1] = nx;
    y[deg + 1] = ny;

    c = 1ll * c * (k - nx + MOD) % MOD;
    
    for (int i = 1; i <= deg; ++i)
        w[i] = 1ll * w[i] * power(x[i] - x[deg + 1] + MOD, MOD - 2) % MOD;
    
    for (int i = 1; i <= deg; ++i)
        w[deg + 1] = 1ll * w[deg + 1] * (x[deg + 1] - x[i] + MOD) % MOD;
    
    w[deg + 1] = power(w[deg + 1], MOD - 2);

    int res = 0;

    for (int i = 1; i <= deg + 1; ++i)
        res = 1ll * y[i] * w[i] % MOD * power(k - x[i]) % MOD;
    
    res = 1ll * res * c % MOD;
    return res;
}
```

## 例题
### Luogu P4593 教科书般的亵渎
> 给出一个单调递增的长度为 $n$ 的数列，每次操作可以使数列的值全部减少 $x$，满足 $x$ 恰好是操作开始前数列开头的值域连续段全部被减为小于等于 $0$，操作结束后，将该连续段删除，并获得 $1+2^k+\cdots +x^k$ 的分数。题面给出数列中没有出现的数，共 $m$ 个。
>
> $1\le n \le 10^{13}$，$1\le m \le 10$。

把原题面转化为上述模型后，我们需要求 $f(x)=\sum_{i=1}^{x}i^k$。

这里给出一个结论：**一个 $n$ 次函数的前缀和为 $n+1$ 次。**

所以考虑先求出 $\forall i\in [1,k+2]$ 的 $f(i)$，插值得到 $k+1$ 次的 $f(x)$。

至于题目中的减操作，$\Theta(m^2)$ 模拟每一段即可。若使用前后缀积优化，则时间复杂度为 $\Theta(m^2k)$。
```cpp
#include <iostream>
#include <algorithm>
#include <utility>
#include <numeric>
#include <vector>
using namespace std;

using ll = long long;

constexpr int maxm = 50 + 10;
constexpr int mod = 1e9 + 7;

int t, m;
ll n, a[maxm];
int x[maxm], y[maxm];
int fac[maxm], inv[maxm], pre[maxm], suf[maxm];

int power(int x, int y) {
    int res = 1;

    for (; y; y /= 2) {
        if (y % 2)
            res = 1ll * res * x % mod;

        x = 1ll * x * x % mod;
    }

    return res;
}

inline int sgnInv(int n) {
    static int inv[2] = {power(1, mod - 2), power(mod - 1, mod - 2)};
    return inv[n % 2];
}

void preprocess() {
    fac[0] = 1;

    for (int i = 1; i <= maxm - 1; ++i)
        fac[i] = 1ll * fac[i - 1] * i % mod;
    
    inv[maxm - 1] = power(fac[maxm - 1], mod - 2);

    for (int i = maxm - 2; i >= 0; --i)
        inv[i] = 1ll * inv[i + 1] * (i + 1) % mod;
}

int interpolation(ll x0, int deg, int x[], int y[]) {
    int ans = 0;

    pre[0] = suf[deg + 2] = 1;

    for (int i = 1; i <= deg + 1; ++i)
        pre[i] = 1ll * pre[i - 1] * (x0 - i + mod) % mod;
    
    for (int i = deg + 1; i >= 1; --i)
        suf[i] = 1ll * suf[i + 1] * (x0 - i + mod) % mod;

    for (int i = 1; i <= deg + 1; ++i) {
        int num = 1ll * y[i] * pre[i - 1] % mod * suf[i + 1] % mod;
        int den = 1ll * inv[i - 1] * inv[deg - i + 1] % mod * sgnInv(deg - i + 1) % mod % mod;
        ans = (1ll * ans + 1ll * num * den % mod) % mod;
    }

    return (ans + mod) % mod;
}

int calc(ll x0) {
    if (x0 <= m + 3)
        return y[x0];

    return interpolation(x0, m + 2, x, y);
}

int solve() {
    int ans = 0;

    for (int i = 1; i <= m + 3; ++i) {
        x[i] = i;
        y[i] = (y[i - 1] + power(i, m + 1)) % mod;
    }

    for (int i = 1; i <= m; ++i) {
        ans = (ans + calc(n)) % mod;

        for (int j = i; j <= m; ++j)
            ans = (ans - power(a[j], m + 1) + mod) % mod;

        for (int j = i + 1; j <= m; ++j)
            a[j] -= a[i];

        n -= a[i];
    }

    if (n != 0)
        ans = (ans + calc(n)) % mod;

    return ans;
}

int main() {
    ios::sync_with_stdio(false);
    preprocess();
    cin >> t;

    while (t--) {
        cin >> n >> m;

        for (int i = 1; i <= m; ++i)
            cin >> a[i];

        sort(a + 1, a + 1 + m);
        cout << solve() << endl;
    }

    return 0;
}
```

### Luogu P3270 成绩比较
> G 系共有 $N$ 位同学，$M$ 门必修课。这 $N$ 位同学的编号为 $0$ 到 $N-1$ 的整数，其中 B 神的编号为 $0$ 号。这 $M$ 门必修课编号为 $0$ 到 $M-1$ 的整数。一位同学在必修课上可以获得的分数是 $1$ 到 $U_i$ 中的一个整数。
> 
> 如果在每门课上 A 获得的成绩均小于等于 B 获得的成绩，则称 A 被 B 碾压。在 B 神的说法中，G 系共有 $K$ 位同学被他碾压（不包括他自己），而其他 $N-K-1$ 位同学则没有被他碾压。D 神查到了 B 神每门必修课的排名。
>
> 这里的排名是指：如果 B 神某门课的排名为 $R$，则表示有且仅有 $R-1$ 位同学这门课的分数大于 B 神的分数，有且仅有 $N-R$ 位同学这门课的分数小于等于 B 神（不包括他自己）。
>
> 请你求出所有可能的成绩种类数。
>
> $1\le N\le 100$，$1\le M\le 100$，$1\le U_i\le 10^9$，$1\le R_i\le N$。

考虑 DP。先只考虑成绩的相对关系，即先不管每个人的成绩具体是多少。设 $f[i][j]$ 表示前 $i$ 门课程 B 神能够碾压 $j$ 个人的方案数：

$$
f[i][j]=\sum_{k=j}^{N-1} f[i-1][k] \binom{k}{j} \binom{n-k-1}{r[i]-1-k+j}\\\\
f[0][N-1] = 1
$$

这个式子意思是前 $i-1$ 门课中，B 神可以碾压 $k$ 人，但现在其中的 $k-j$ 人这门课成绩比 B 神高，无法被碾压，有 $\binom{k}{j}$ 种方案，为了保证 B 神排名为 $r[i]$，除了刚才的 $k-j$ 人，还要有 $r[i]-1-k+j$ 人排在 B 神前面，这些人来自未被碾压的 $n-k-1$ 人中有 $\binom{n-k-1}{r[i]-1-k+j}$ 种方案。

由于每门课的排名固定互相独立，所以设 $d(i)$ 为第 $i$ 门课的人的排名固定时成绩方案数，则有：

$$
d(i)=\sum_{j=1} j^{N-R_i} (U_i-j)^{R_i-1}
$$

这里的算法是枚举 B 神的分数，然后统计每个人可能的分数的方案数。可以发现 $j^{N-R_i} (U_i-j)^{R_i-1}$ 是一个 $N-1$ 次多项式，则 $d(i)$ 是一个 $n$ 次多项式，插值解决。

最后定义 $f[i][j]$ 为考虑成绩的具体取值的方案数：

$$
f[i][j]=d(i)\sum_{k=j}^{n-1} f[i-1][k] \binom{k}{j} \binom{n-k-1}{r[i]-1-k+j}\\\\
f[0][N-1]=1
$$

最后答案为 $f[M][K]$。时间复杂度 $\Theta(MN^2)$。

```cpp
#include <iostream>
using namespace std;

constexpr int MAXN = 100 + 10;
constexpr int MOD = 1e9 + 7;

int n, m, k;
int u[MAXN], r[MAXN];
int c[MAXN][MAXN], d[MAXN], f[MAXN][MAXN];

int power(int x, int y) {
    int res = 1;

    for (; y; y /= 2) {
        if (y % 2)
            res = 1ll * res * x % MOD;

        x = 1ll * x * x % MOD;
    }

    return res;
}

int interpolation(int deg, int k, int x[], int y[]) {
    int res = 0;

    for (int i = 1; i <= deg + 1; ++i) {
        int num = y[i], den = 1;

        for (int j = 1; j <= deg + 1; ++j) {
            if (i == j)
                continue;

            num = 1ll * num * (k - x[j] + MOD) % MOD;
            den = 1ll * den * (x[i] - x[j] + MOD) % MOD;
        }

        res = (res + 1ll * num * power(den, MOD - 2)) % MOD;
    }

    return res;
}

int calc(int i) {
    int x[MAXN] = { 0 }, y[MAXN] = { 0 };

    for (int j = 1; j <= min(u[i], n + 1); ++j) {
        x[j] = j;
        y[j] = (y[j - 1] + 1ll * power(j, n - r[i]) * power(u[i] - j, r[i] - 1)) % MOD;
    }

    if (u[i] <= n + 1)
        return y[u[i]];

    return interpolation(n, u[i], x, y);
}

int main() {
    ios::sync_with_stdio(false);
    cin >> n >> m >> k;

    for (int i = 1; i <= m; ++i)
        cin >> u[i];

    for (int i = 1; i <= m; ++i)
        cin >> r[i];

    for (int i = 0; i <= n; ++i)
        c[i][0] = c[i][i] = 1;

    for (int i = 1; i <= n; ++i)
        for (int j = 1; j < i; ++j)
            c[i][j] = (c[i - 1][j] + c[i - 1][j - 1]) % MOD;
            
    for (int i = 1; i <= m; ++i)
        d[i] = calc(i);

    f[0][n - 1] = 1;

    for (int i = 1; i <= m; ++i) {
        for (int j = 1; j < n; ++j) {
            for (int k = j; k < n; ++k) {
                if (k - j > r[i] - 1)
                    continue;

                int cnt = 1ll * c[k][j] * c[n - k - 1][r[i] - 1 - (k - j)] % MOD;
                int pre = 1ll * d[i] * f[i - 1][k] % MOD * cnt % MOD;
                f[i][j] = (f[i][j] + pre) % MOD;
            }
        }
    }

    cout << f[m][k] << endl;
    return 0;
}
```

### Luogu P4463 calc
> 定义一个长度为 $n$ 的正整数数列 $a_n$ 的权值为 $\prod_{i=1}a_i$，现在要求你求出所有满足以下条件的数列的权值并对质数 $p$ 取模：
> 1. $\forall i,j \in [1,n]\wedge i\ne j$，$a_i\ne a_j$。
> 2. $\forall i \in [1,n]$，$a_i\in [1,k]$。
>
> $1\le k\le 10^9$，$1\le n\le 500$，$2 \le p \le 10^9$，$p>k>n+1$。

设 $f[i][j]$ 为不考虑数的顺序情况下，在数列的前 $i$ 个数都属于 $[1,j]$ 的权值和，考虑选或不选 $j$ 这个数，则有：

$$
f[i][j]=f[i][j-1]+j\cdot f[i-1][j-1]\\\\
f[0][j]=1
$$

最后答案为 $f[n][k]$。可以想到需要插值，现在重点是要知道 $f[n][x]$ 的次数。

设 $d[i][j]$ 为 $f[i][j]$ 的次数.
- 首先可以知道 $d[0][j]=0$。
- 对于 $i=1$，仅考虑 $f[1][j]\longleftarrow j\cdot f[0][j-1]$，则 $d[1][j] = 1$，而考虑 $f[1][j]\longleftarrow f[1][j-1]$，相当于做前缀和，则 $d[1][j] = 2$。
- 对于 $i=2$，仅考虑 $f[2][j]\longleftarrow j\cdot f[1][j-1]$，则 $d[2][j] = 3$，而考虑 $f[2][j]\longleftarrow f[2][j-1]$，相当于做前缀和，则 $d[2][j] = 4$。
- etc.

最后得到 $d[n][x]=2n$，则需要处理出 $2n+1$ 个 $f[n][x_i]$。
注意这还没考虑数的顺序，对数列进行全排列，答案为 $n!f[n][k]$。

时间复杂度 $\Theta(2n^2+8n^3)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

constexpr int MAXN = 500 + 10;

int n, k, p, f[MAXN][MAXN * 2];
int x[MAXN * 2], y[MAXN * 2];

void DP() {
	for (int i = 0; i <= 2 * n; ++i)
		f[0][i] = 1;

	for (int i = 1; i <= n; ++i)
		for (int j = 1; j <= 2 * n; ++j)
			f[i][j] = (f[i][j - 1] + 1ll * j * f[i - 1][j - 1]) % p;
}

int power(int x, int y) {
    int res = 1;
    while (y) {
        if (y % 2)
            res = 1ll * res * x % p;
        x = 1ll * x * x % p;
        y /= 2;
    }
    return res % p;
}

int inv(int x) {
    return power(x, p - 2);
}

int interpolation(int deg, int k, int x[], int y[]) {
    int ans = 0;

    for (int i = 1; i <= deg + 1; ++i) {
        int num = y[i], den = 1;

        for (int j = 1; j <= deg + 1; ++j) {
            if (i == j)
                continue;

            num = 1ll * num * (k - x[j] + p) % p;
            den = 1ll * den * (x[i] - x[j] + p) % p;
        }
        ans = (ans + 1ll num * inv(den) % p) % p;
    }

    return ans;
}

int main() {
    cin >> k >> n >> p;
    DP();

	for (int i = 1; i <= 2 * n + 1; ++i) {
		x[i] = i;
		y[i] = f[n][i];
	}

    int ans = interpolation(2 * n, k % p, x, y);
    
    for (int i = 1; i <= n; ++i)
    	ans = 1ll * ans * i % p;
    
    cout << ans << endl;
    return 0;
}
```