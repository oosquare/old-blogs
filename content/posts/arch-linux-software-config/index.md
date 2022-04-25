---
title: "Arch Linux 常用软件配置"
subtitle: ""
date: 2022-03-17T22:28:02+08:00
draft: false
author: ""
authorLink: ""
description: ""
keywords: ""
license: "本文以 CC BY-NC 4.0 许可证发布"
comment: false
weight: 0

tags:
 - Linux
 - Arch Linux
 - 配置
categories:
 - Tech

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
lightgallery: true
seo:
  images: []

# See details front matter: /theme-documentation-content/#front-matter
---

记录一下安装 `Arch Linux` 后的常用软件安装以及配置。

## `yay`
使用 `yay` 从 `AUR` 上下载各种官方仓库所没有的包。

项目主页：<https://github.com/Jguer/yay>。

从源码安装：

```sh
$ pacman -S --needed git base-devel
$ git clone https://aur.archlinux.org/yay.git
$ cd yay
$ makepkg -si
```

或者下载二进制包：

```sh
$ pacman -S --needed git base-devel
$ git clone https://aur.archlinux.org/yay-bin.git
$ cd yay-bin
$ makepkg -si
```

用法与 `pacman` 一致。

## `debtap`
`debtap` 是 `AUR` 包，用于将 `deb` 包转换为 `pacman` 可以使用的包。

安装：

```sh
$ yay -S debtap
```

安装后可以编辑 `/usr/bin/debtap`，替换所有的 `http://ftp.debian.org/debian/dists` 为 `https://mirrors.ustc.edu.cn/debian/dists`，替换所有的 `http://archive.ubuntu.com/ubuntu/dists` 为 `https://mirrors.ustc.edu.cn/ubuntu/dists/`。

使用：
```sh
$ sudo debtap -u # 更新源列表
$ debtap -q package.deb # 转换 deb 包，-q 表示不要编辑除元数据之外的信息
$ sudo pacman -U package.tar.xz # 安装生成的包
```

## `Clementine`
一个跨平台的音乐播放器，不仅可以播放各种格式的音乐，还支持提取歌曲的元信息。

安装：

```sh
$ sudo pacman -S clementine
$ sudo pacman -S gst-plugins-good gst-plugins-base \
    gst-libav gst-plugins-bad gst-plugins-ugly
```

由于 `Clementine` 使用了 `GStreamer`，所以还要安装必要的插件，否则无法播放音乐。

## `Wireshark`
基于 `Qt` 编写的开源抓包工具。

可以编译源码，也可以直接用 `pacman` 安装：

```sh
$ sudo pacman -S wireshark
```

默认情况下只能使用 `root` 用户运行才可以访问网卡等设备，通过修改用户组设置使得常用用户可以直接使用：

```sh
$ sudo groupadd wireshark # 新建一个专用的用户组
$ sudo chgrp wireshark /usr/bin/dumpcap # 将dumpcap更改为wireshark用户组
$ sudo chmod 4755 /usr/bin/dumpcap # 4 表示执行时用户可以与所有者有相同权限
$ sudo gpasswd -a ctj12461 wireshark # 添加自己
```

附 `wireshark` 过滤出音乐的 `HTTP request` 的 `pattern`：

```
(tcp.port == 80 || udp.port == 80) && (http.request.uri contains "mp3" || http.request.uri contains "m4a" || http.request.uri contains "mp4" || http.request.uri contains "flac" || http.request.uri contains "ogg")
```

## `VS Code`
如果直接用包管理器安装 `code`，则会安装 `Code - OSS`，这个虽然也是 `VS Code`，但在协议上与 `Microsoft` 提供的 `Visual Studio Code` 不同，所以所带有的内容也有所差别，比如无法登陆 `Microsoft` 帐号，无法同步设置等。

如果有需要，可以用 `yay` 安装 `visual-studio-code-bin`：

```sh
$ yay -S visual-studio-code-bin
```

也是直接输入 `code` 运行。

## `WPS 2019 for Linux`
目前 `WPS` 对 `Microsoft Office` 的支持是最好的，而且还在稳定更新，推荐使用。

使用 `yay` 安装，并且要安装可选的依赖包：

```sh
$ yay -S wps-office-cn wps-office-mime-cn wps-office-mui-zh-cn # 安装中文环境的 WPS
$ yay -S ttf-wps-fonts wps-office-fonts # 安装字体
```

如果使用 `KDE`，可能会遇到字体模糊的情况，这是由缩放不为 100% 引起的问题，`WPS` 使用了 `Qt` 所以只要在运行前加上环境变量 `QT_SCREEN_SCALE_FACTORS=1` 即可，对于启动器或桌面上的 `Desktop Entry`，只要在 `/usr/share/applications` 下修改所有含 `wps` 的 `Desktop Entry` 文件即可，按照下面修改即可：

```sh
# /usr/share/applications/wps-office-wps.desktop
Exec=env QT_SCREEN_SCALE_FACTORS=1 /usr/bin/wps %U
```

这个方法还适用于其他的 `Qt` 程序。

## `Fcitx 5`
`Fcitx 5` 使用简单，比较推荐。

安装：

```sh
$ sudo pacman -S fcitx5-im fcitx5-configtool fcitx5-chinese-addons fcitx5-rime
```

添加环境变量到 `/etc/environment` 以正常使用 `fcitx5`：

```sh
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
INPUT_METHOD=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

如果无法开机启动，则执以下命令：

```sh
# 通过在 autostart 目录下添加启动项
$ cp /usr/share/applications/org.fcitx.Fcitx5.desktop ~/.config/autostart/
```

词库安装，可以自己选择：

```sh
$ sudo pacman -S fcitx5-pinyin-zhwiki
$ yay -S fcitx5-pinyin-sougou
$ yay -S fcitx5-pinyin-zhwiki-rime
$ yay -S fcitx5-pinyin-moegirl-rime
```

解决中文下按 `[` 和 `]` 输出为其他符号：编辑 `/usr/share/fcitx5/punctuation/punc.mb.zh_CN`，把 `[` 和 `]` 映射的字符修改为 `【` 和 `】`。

## `Icalingua++`
一个 `OICQ` 前端，基于已经被封杀的 `Icalingua`，拥有大多数实用功能。`GitHub` 项目主页：<https://github.com/icalingua-plus-plus/icalingua-plus-plus>。有各种安装方式，如 `AppImage`、`pacman`、`yay`。

若使用 `pacman` 安装，则需先下载软件包，比如是 `icalingua-2.6.1-1-x86_64.pkg.tar.zst`，则使用以下命令：

```sh
$ sudo pacman -U icalingua-2.6.1-1-x86_64.pkg.tar.zst
```

若使用 `yay`：

```sh
$ yay -S icalingua++
```

这里还是要说一句：tx nm*l。

## `Gwenview`
一个看图软件。

```sh
$ sudo pacman -S gwenview
```

## `Latte`
一个 `Dock` 软件，可以与 `KDE` 很好的集成在一起。

```sh
$ sudo pacman -S latte-dock
```

随后在桌面上右键点击添加小部件，选择 Latte 任务管理器即可，然后在 `Latte` 上右键添加部件，选择 `Launchpad Plasma Dark`，添加一个启动台。

## 顶部状态栏
在桌面上右键点击添加面板，选择应用程序菜单栏，然后右键菜单栏添加部件，从左往右依次是：Application Title、全局菜单、面板间隙、系统托盘，数字时钟、锁屏/注销、显示桌面，其中 Application Title 需要进一步配置，选择其配置中的 Apperance 选项卡中的 Text Type，改为 Application name。

配合 `Latte`，总体效果如下，应该是综合了美观和高效的美化配置了。

{{< image src="desktop.png" caption="桌面" >}}

## McMojave 图标
[项目主页](https://github.com/vinceliuice/McMojave-circle)，下载后运行安装脚本即可。

```sh
$ git clone https://github.com/vinceliuice/McMojave-circle
$ cd McMojave-circle
$ chmod +x install.sh
$ ./install.sh # 其他参数详见 GitHub 上的 README
```

## KDE 主题
安装 WhiteSure，[项目主页](https://github.com/vinceliuice/WhiteSur-kde)。

```sh
$ git clone https://github.com/vinceliuice/WhiteSur-kde
$ cd WhiteSur-kde
$ chmod +x install.sh
$ ./install.sh
```

安装后，在设置中选择全局主题为 WhiteSur-alt，Plasma 视觉风格为 WhiteSur-dark，窗口装饰元素为 Breeze 微风（默认的窗口装饰元素底部边框渲染可能有问题，可以自己考虑要不要用）。总之看自己喜好

## Kvantum
Kvantum 可以配置应用程序风格，比如背景透明等等。Kvantum 适用于很多主题，包括上面的 WhiteSur。

```sh
$ sudo pacman -S kvantum
```

一般安装一个全局主题后，Kvantum 中就可以修改其配置了。打开 Kvantum 后，选择变更/删除主题选项卡，然后在选择一个主题的下拉框中选择全局主题，比如 WhiteSur，然后点击配置主题选项卡，在技巧中勾选非活动窗口禁用模糊处理，在合成 & 一般外观中的不透明的应用的文本框中添加 clementine，具体尝试一下就知道如果不加，Clementine 会非常丑。

最后更改应用程序风格为 kvantum。现在只要是使用原生 UI 风格的应用，菜单等都会变为模糊透明。