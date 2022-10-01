---
title: 使用 WSL2 部署 Arch Linux 环境
date: 2022-10-01T09:38:08+08:00

categories: tech
tags:
  - wsl
  - archlinux
  - linux

image: cover.jpg

draft: false
---

最近入了 Arch Linux 的坑，手上没有空闲的机子可以装着玩，虚拟机又有点膈应的感觉，就找了找资料在 WSL2 上装了体验一下。

关于 WSL1 和 WSL2 的区别，不在本文的讨论范围内，可以前往[微软的官方说明](https://learn.microsoft.com/zh-cn/windows/wsl/compare-versions)查看。

本文中，以 `>` 开头的命令为 Powershell 命令，`#` 开头为 Arch 中的 `root` 用户命令，`$` 开头为 Arch 中的普通用户命令。

## 准备工作

- Windows 10 2004 版本或以上（我使用的是 Windows 11）
- 可选功能中打开**适用于 Linux 的 Windows 子系统**和**虚拟机平台**

首先前往 [**这里**](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi) 下载 WSL 的内核更新包，不然没法安装。

安装后将 WSL 的默认版本设置为 WSL2：
```powershell
> wsl --set-default-version 2
```

## 安装 Arch

前往 [项目仓库 (yuk7/ArchWSL)](https://github.com/yuk7/ArchWSL) 的 [Releases](https://github.com/yuk7/ArchWSL/releases) 页面下载最新的 `Arch.zip`，并解压到具有写权限的空文件夹里。留意 `Arch.exe` 的名称，`.exe` 前的部分会作为稍后在 WSL 内注册的发行版名称，例如改名成 `ArchLinux.exe`，稍后的实例就叫 `ArchLinux`。

{{< tip note "多个 Arch 共存" >}}
由于这里的 Arch 是通过文件安装，只要将 `exe` 文件复制并改名即可。注意改名后的文件不要和已有的实例名称重复。
{{</ tip >}}

双击打开 `Arch.exe`，程序会自动注册 rootfs 和注册表配置。

## 创建默认用户

直接使用 `root` 用户很不安全，这里我们创建一个新的用户代替 `root` 进行日常使用。

```
# useradd -m -G wheel -s /bin/bash bs
```

这里的 `bs` 是用户名称，将其换成你自己的。然后为新用户设置密码：

```
# passwd bs
```

为了使新用户可以使用 `sudo` 权限，编辑 `/etc/sudoers` 文件，找到 `root ALL=(ALL:ALL) ALL`，在其下添加一行 `bs ALL=(ALL:ALL) ALL` （`bs` 同样换成刚刚的用户名）；并找到 `#%wheel ALL=(ALL:ALL) ALL`，将其取消注释（去掉最前面的 `#`）

{{< tip note "找不到编辑器" >}}
如果没有自己常用的编辑器（如 vim），可以先跳到下面的 Pacman 配置小节，然后自行安装。
{{</ tip >}}

改好了是这样的：

```
root ALL=(ALL:ALL) ALL
bs ALL=(ALL:ALL) ALL

%wheel ALL=(ALL:ALL) ALL
```

然后使用 `exit` 退出 Arch 交互界面，打开 Powershell，切换到 Arch 目录，输入以下命令将默认登录用户切换为刚刚创建的新用户：

```powershell
> Arch.exe config --default-user bs
```

## 配置 Pacman

打开 Arch.exe 进入命令行。

由于众所周知的原因，国内访问 Arch 的软件源十分缓慢，这里将源换为[清华大学开源镜像站的源](https://mirrors.tuna.tsinghua.edu.cn/help/archlinux/)。

编辑 `/etc/pacman.d/mirrorlist`，往下翻找到 China 一栏，将 `https://mirrors.tuna.tsinghua.edu.cn` 行取消注释。也可以多取消几个注释作为后备。如果懒得找也可以直接在文件开头添加：

```
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```

初始化 Pacman 的密钥环：

{{< tip note "未创建新用户" >}}
如果刚刚没有创建用来代替 `root` 的新用户，请将本节命令中的 `sudo` 去除，
{{</ tip >}}

```bash
$ sudo pacman-key --init
$ sudo pacman-key --populate
$ sudo pacman -Syy archlinux-keyring
```

执行滚动更新：

```bash
$ sudo pacman -Syu
```

{{< tip success 搭建完成 >}}
这样就配置好了一个基本的 Arch Linux 环境了，但仍然是一个空壳子，接下来给它配置一些常用服务。
{{</ tip >}}

## 安装包管理器 yay

yay 是一个广泛使用的 AUR Helper，解决了 pacman 无法安装 AUR 软件的问题，同时完全兼容 pacman 操作。

首先安装必要软件包：

```bash
$ sudo pacman -S base-devel git
```

安装 `yay-bin`（如果直接安装 `yay` 则需要自己编译）：

```bash
$ cd ~
$ git clone https://aur.archlinux.org/yay-bin.git
$ cd yay-bin
$ makepkg -si
$ yay
```

## 安装 oh-my-zsh

oh-my-zsh 是一个常用的 zsh 框架，提供了许多美观的主题和实用插件，能极大提升 shell 下的效率。

```bash
$ yay -S zsh curl
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装基本插件，这里我安装了自己常用的 `zsh-autosuggestions` 和 `zsh-syntax-highlighting` 来提供自动补全和语法高亮。

```bash
$ cd ~/.oh-my-zsh/plugins
$ git clone https://github.com/zsh-users/zsh-autosuggestions.git --depth 1
$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git --depth 1
```

编辑 `~/.zshrc`，找到 `plugins`，在 `git` 后面添加插件名称，使用空格或空行分隔。也可以往上找到 `ZSH_THEME` 并修改主题。

![最终效果，主题使用 ys](omz-preview.png)
