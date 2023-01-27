---
filename: "motion-study-in-minecraft"
title: "Minecraft 运动学研究"
subtitle: ""
date: 2022-08-06T21:06:58+08:00
draft: false
author: "ctj12461"
authorLink: ""
authorEmail: ""
description: ""
keywords: ""
license: ""
comment: false
weight: 0

tags:
  - Minecraft
  - 物理
  - 实验
  - Python
categories:
  - Game

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
lightgallery: true
seo:
  images: []

# See details front matter: /theme-documentation-content/#front-matter
---

在造红石炮时，往往需要用到一些运动学的知识。以下是我对 Minecraft 的运动学的一些研究，应该会对设计红石炮有所帮助，

## 前置知识
- 在 Minecraft 中，我们通常只考虑实体的运动，而方块则与运动没有一点关系。实体的运动大体与高中物理相同，但需要考虑空气阻力的作用，这使得 Minecraft 中的运动更加真实但又更加复杂。
- 在 Minecraft 中，长度单位有格和米 $(\text{m})$ 两种，$1$ 格 $=1\ \text{m}$，所以它们在数值上是相同的，所以一般直接使用米作为长度单位。
- 时间单位有三种，分别是游戏刻 $(\text{gt})$、红石刻 $(\text{rt})$ 和秒 $(\text{s})$，它们有这样的换算关系：$1\ \text{gt}=0.5\ \text{rt}=1\ \text{s}$。这三种单位都很常用。
- Minecraft 中不同实体的重力加速度可能是不同的。
- 尽管现实中的运动是连续的，但由于计算机和 Minecraft 的实现原因，Minecraft 的运动是离散的，其每 $1\ \text{gt}$ 计算一次，因此，各种复杂的计算可以通过计算每 $1\ \text{gt}$ 的运动情况再累加的方法实现。

## 竖直方向上的运动
### 理论计算
这里以 TNT 为研究对象。在 [Minecraft Wiki](https://minecraft.fandom.com/wiki/Entity#Motion_of_entities) 上有各种实体的相关参数，查阅得知 TNT 的重力加速度 $a = 0.04\ \operatorname{m/gt^2}=16\ \operatorname{m/s^2}$，阻力系数 $k=0.02\ \operatorname{gt^{-1}}=0.4\ \operatorname{gt^{-1}}$。原文中提到的是 Drag，中文版中翻译为阻力，但其实它是表示在一定速度时产生的产生空气阻力的加速度，即 $a_f=kv$，所以我觉得阻力系数会更恰当。

前面提到 Minecraft 中的运动可以拆分为每个 $\operatorname{gt}$ 进行计算，所以计算过程就像是数列的计算。

这里我们研究 TNT 在竖直方向上初速度为 $0$ 的运动。设 $v(t)$ 表示在 $t\ \operatorname{gt}$ 时的速度，$h(t)$ 为此时下落的高度。则 $v(0) = 0$。

假设每 $\operatorname{gt}$ 内 TNT 做匀变速直线运动，则运动状态有如下关系：

$$
\begin{split}
&v(t) = v(t - 1) + [a - kv(t - 1)]\\\\
&h(t) = h(t - 1) + \frac{v(t - 1) + v(t)}{2}\\\\
\end{split}
$$

使用 Python 3 实现计算：

```python
v = [0.0] * 81
h = [0.0] * 81
k = 0.02
a = 0.04

for i in range(1, 81):
    v[i] = (1 - k) * v[i - 1] + a
    h[i] = h[i - 1] + (v[i - 1] + v[i]) / 2

    if i % 8 == 0:
        print(f"i = {i}, t = {i / 20} s, h = {h[i]:.1f} m")
```

输出结果如下：

```plain
i = 8, t = 0.4 s, h = 1.2 m
i = 16, t = 0.8 s, h = 4.7 m
i = 24, t = 1.2 s, h = 10.0 m
i = 32, t = 1.6 s, h = 16.9 m
i = 40, t = 2.0 s, h = 25.1 m
i = 48, t = 2.4 s, h = 34.5 m
i = 56, t = 2.8 s, h = 44.9 m
i = 64, t = 3.2 s, h = 56.2 m
i = 72, t = 3.6 s, h = 68.1 m
i = 80, t = 4.0 s, h = 80.7 m
```

### 实验验证
在 Minecraft 中进行验证，可以通过中继器延时控制活塞的伸缩，用活塞挡住 TNT，从而控制 TNT 的实际下落时间。TNT 在 $4\ \text{s}$ 后会爆炸，可以用方块搭建一面墙作为标尺，利用 TNT 爆炸破坏墙面来确定其位置。

{{< image src="experiment-1-design-1.png" caption="实验装置设计" >}}

{{< image src="experiment-1-design-2.png" caption="实验装置设计" >}}

为了减小实验误差，可采取如下措施：

- 选择爆炸抗性较高的方块
- 适当增大墙与 TNT 下落轨迹的距离，因为 TNT 伤害按照球体进行计算，拉长距离可以让损害的面积减小，TNT 中心的高度更精确
- 计算好延迟，**特别是活塞**

进行三次实验，对每个时刻的下落高度取平均值。

{{< image src="experiment-1-result.png" caption="实验结果" >}}

进行数据处理，并与理论分析得到的数据进行比较：

|      **下落时间 $t/\text{s}$**       | $0.8$ | $1.2$  | $1.6$  | $2.0$  | $2.4$  | $2.8$  | $3.2$  | $3.6$  | $4.0$  |
| :----------------------------------: | :---: | :----: | :----: | :----: | :----: | :----: | :----: | :----: | :----: |
|  **第一组下落高度 $h_1/\text{m}$**   | $4.5$ | $10.2$ | $17.5$ | $25.6$ | $35.3$ | $45.3$ | $56.4$ | $68.0$ | $79.9$ |
|  **第二组下落高度 $h_2/\text{m}$**   | $4.6$ | $10.7$ | $17.3$ | $25.8$ | $34.9$ | $45.2$ | $56.4$ | $69.1$ | $79.7$ |
|  **第三组下落高度 $h_3/\text{m}$**   | $5.0$ | $10.5$ | $17.1$ | $25.7$ | $35.0$ | $45.8$ | $56.9$ | $68.6$ | $79.7$ |
|    **平均下落高度 $h/\text{m}$**     | $5.0$ | $10.4$ | $17.3$ | $25.7$ | $35.1$ | $45.4$ | $56.6$ | $68.6$ | $79.8$ |
|    **理论下落高度 $h_0/\text{m}$**    | $4.7$ | $10.0$ | $16.9$ | $25.1$ | $34.5$ | $44.9$ | $56.2$ | $68.1$ | $80.7$ |
| **下落高度差值 $\Delta h/\text{m}$** | $0.3$ | $0.4$  | $0.4$  | $0.6$  | $0.6$  | $0.5$  | $0.4$  | $0.5$  | $0.9$  |

由于通过方块的损毁来采集数据，而损毁区域的大小大概可以用一个 $4\times 4$ 的矩形框起来，再加上爆炸有很多不确定因素，所以系统误差较大，而 $\Delta h$ 大多数都小于 $0.9\ \text{m}$，在可接受范围内，所以这种计算方式是可行的。

### 应用
Minecraft 中的速度是可以合成和分解的，所以竖直方向上的运动不管在什么时候都可以拿出来独立分析。

我们可以通过该理论计算任何实体在竖直方向上的运动情况，尤其是在红石炮中对 TNT 弹头的飞行时间进行计算和调整，以及发射时相关时序的控制。

## 水平方向上的运动
其实水平方向上的运动与竖直方向上的大体类似，但是有点实体在水平方向（$x,z$ 轴）的阻力系数与竖直方向的不同，所以需要注意。

但是考虑到实际应用，一般情况下都是涉及到红石炮这种装置，其在水平方向上能够提供往往非常巨大，阻力远小于动力，且红石炮的弹头在飞行过程中在水平方向上不受重力，可以近似为匀速直线运动，不需要用到这么复杂的理论。