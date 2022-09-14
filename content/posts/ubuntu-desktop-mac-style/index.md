---
title: Mac风格 Ubuntu Desktop 美化教程
date: 2021-07-27T23:25:10+08:00

categories: tech
tags:
  - ubuntu
  - linux

image: cover.jpg

description: 将 Ubuntu Desktop 美化为 Mac 风格，基于 Ubuntu Desktop 20.04
draft: false
---

前些天把树莓派 4B 的系统换成了 Ubuntu Desktop 21.04，使用没什么问题，驱动支持也非常好，遂美化之。

本教程适用于任何 Ubuntu Desktop 系统（18.04 及以上），不仅限于树莓派。

先看看美化前的效果：

![美化前 1](before-beautify-1.jpg)
![美化前 2](before-beautify-2.jpg)

再看看美化后：

![美化后 1](after-beautify-1.jpg)
![美化后 2](after-beautify-2.jpg)

## 准备工作

- 一台安装了 Ubuntu Desktop 系统（18.04 版本及以上）的设备
- 手

使用前请将软件源更新到最新版本：

```bash
$ sudo apt update
```

## 安装美化管理工具

打开终端，安装 `gnome-tweaks` 工具

```bash
$ sudo apt install gnome-tweaks chrome-gnome-shell -y
```

随后打开 [https://extensions.gnome.org/](https://extensions.gnome.org/) ，点击 `Click here to install browser extension` 安装浏览器插件

![安装浏览器插件](install-browser-extension.png)

随后搜索 [User Themes](https://extensions.gnome.org/extension/19/user-themes/) 插件安装，点击页面右上角的开关，把`OFF`改成`ON`就可以了，如果弹出确认窗口就点确认

![安装插件](user-theme-extension.png)

## Dock 栏

Ubuntu Desktop 自带的 Dock 栏并不是很好用，可定制性也不强，这里使用 `Dash to Dock` 插件增强。

向上面安装 `User Theme` 插件一样，在 [https://extensions.gnome.org/](https://extensions.gnome.org/) 中搜索 [Dash to Dock](https://extensions.gnome.org/extension/307/dash-to-dock/) 并安装。

然后去 Gnome Tweaks 里启用：

![启用 Dash to Dock 插件](enable-dash-to-dock.png)

随后你可以右键 Dock 栏上的应用抽屉图标，打开 Dash to Dock 设置，这里可以配置一系列设置，如位置、图标大小、透明度等等。

![Dash to Dock 设置](dash-to-dock-settings.png)

## Mac 风格主题安装

文首那种 Mac 风格的美化包是 [WhiteSur-gtk-theme](https://github.com/vinceliuice/WhiteSur-gtk-theme/)。

作者建议使用脚本方式安装，这样可以体验主题的全部功能，例如锁屏美化，Dock 栏美化等等。方法如下：

确保你安装了 `git`：

```bash
$ sudo apt install git -y
```

将 GitHub 上的[源码仓库](https://github.com/vinceliuice/WhiteSur-gtk-theme/)克隆到本地：

```bash
$ git clone https://github.com/vinceliuice/WhiteSur-gtk-theme.git
```

执行安装：

```bash
$ cd WhiteSur-gtk-theme
$ chmod +x ./install.sh    # 赋予可执行权限
$ ./install.sh
```

这会以默认的方式自动安装 WhiteSur 主题到系统。（同时安装亮/暗色，所有选项保持默认）

### 自定义安装

#### 基本配置

这里列出了基本的主题配置选项

##### 主题模式

使用 `-c` 或 `--color` 选项选择主题模式（亮/暗色），可以重复使用

```bash
$ ./install.sh -c light            # 只安装亮色
$ ./install.sh -c light -c dark    # 亮暗色一起安装
```

默认：亮暗色一起安装

##### 主题色

使用 `-t` 或 `--theme` 选项选择主题色，可以重复使用

```bash
$ ./install.sh -t red            # 只安装红色
$ ./install.sh -t red -t blue   # 安装红色和蓝色
$ ./install.sh -t all            # 安装所有
```

默认：安装 `default` 颜色

![主题模式/主题色](whitesur/colors-themes.png)

##### 文件管理侧边栏宽度

使用 `-s` 或 `--sidebar` 选项更改文件管理的侧边栏宽度

```bash
$ ./install.sh -s 220
```

默认：使用 `default` 宽度

![文件管理侧边栏宽度](whitesur/sidebars.png)

##### 活动窗口管理器图标

使用 `-i` 或 `--icon` 选项更改活动窗口管理器的图标（最左上角的那个）

```bash
$ ./install.sh -i ubuntu
```

默认：使用苹果 logo

![活动窗口管理器图标](whitesur/icons.png)

##### 文件管理器样式

使用 `-N` 或 `--nautilus-style` 选项更改文件管理器的样式

```bash
$ ./install.sh -N mojave
```

默认：`default` 样式

![文件管理器样式](whitesur/nautilus.png)

#### 进阶配置

这里列出了一些进阶配置，如 Firefox 浏览器美化，锁屏美化，Dash to Dock 美化等等

需要用到 `./tweaks.sh`，先赋予可执行权限：

```bash
$ chmod +x ./tweaks.sh
```

##### Firefox 火狐浏览器

使用 `-f` 或 `--firefox` 选项更改 Firefox（火狐浏览器）的界面样式

```bash
$ ./tweaks.sh -f             # 使用默认样式
$ ./tweaks.sh -f monterey    # 使用 Monterey 样式
```

![WhiteSur 样式](whitesur/firefox-whitesur.jpg)

![Monterey 样式](whitesur/firefox-monterey.jpg)

##### Dash to Dock

使用 `-d` 或 `--dash-to-dock` 选项更改 Dash to Dock 的样式

```bash
$ ./tweaks.sh -d            # 根据主题选择
$ ./tweaks.sh -d -c dark    # 使用暗色
```

![Dash to Dock 样式](whitesur/dash-to-dock.png)

##### 锁屏样式

使用 `-g` 或 `--gdm` 选项更改 Dash to Dock 的样式，需要以 root 身份运行。
`-b`、`-n`、`-N` 选项可以搭配使用。

```bash
$ sudo ./tweaks.sh -g               # 安装锁屏样式
$ sudo ./tweaks.sh -g -b "your picture.jpg"    # 自定义锁屏壁纸
$ sudo ./tweaks.sh -g -b default    # 使用默认锁屏壁纸
$ sudo ./tweaks.sh -g -b blank      # 不使用壁纸
$ sudo ./tweaks.sh -g -n            # 不模糊壁纸
$ sudo ./tweaks.sh -g -N            # 不调暗壁纸
```

![锁屏样式](whitesur/gdm.jpg)

更多选项可以输入 `./tweaks.sh -h` 来查看。

## 图标

图标也可以使用 WhiteSur 的，来自于同一个作者。

![预览 1](whitesur-icons/preview-1.jpg)
![预览 2](whitesur-icons/preview-2.jpg)

### 安装

与主题本体类似，需要使用 `git` 将仓库克隆到本地：

```bash
$ git clone https://github.com/vinceliuice/WhiteSur-icon-theme.git
$ cd WhiteSur-icon-theme
$ chmod +x ./install.sh
$ ./install.sh
```

下面列出了一些可用选项：

- `-d`, `--dest` 定制主题安装目录 （默认为 `$HOME/.themes`）
- `-n`, `--name` 定制图标主题名 （默认为 `WhiteSur`）
- `-t`, `--theme` 定制主题色 [default/purple/pink/red/orange/yellow/green/grey/all] （默认为蓝色）
- `-a`, `--alternative` 为软件中心和文件管理器安装图标
- `-b`, `--bold ` 安装加粗版本（推荐在高分屏下使用）
- `--black` 安装黑色面板主题图标
- `-h`, `--help` 显示帮助信息

### 应用主题

有时主题不会自动应用至系统，这时需要手动打开一开始安装的美化管理工具。应用列表里找到 `优化` 应用或者终端输入 `gnome-tweaks`

![Gnome Tweaks 工具](gnome-tweaks.png)

都选择 WhiteSur 即可。如果配置后不生效，重启后即可看到效果。

## 参考

- [WhiteSur-gtk-theme GitHub 仓库](https://github.com/vinceliuice/WhiteSur-gtk-theme)
