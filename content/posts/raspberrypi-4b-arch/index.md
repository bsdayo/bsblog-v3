---
title: 在树莓派 4B 上安装 Arch Linux ARM
date: 2022-10-02T10:51:14+08:00

categories: tech
tags:
  - raspberrypi
  - archlinux
  - linux

image:

draft: false
---
## 前言

自从在虚拟机、[WSL2](../wsl2-arch/) 上相继安装过 Arch Linux 之后，我将目光投向了家里仅剩的一个还运行着 Ubuntu 的 Linux 设备 —— 树莓派 4B。

话不多说，直接开搞（

## 准备

首先由于启动方式的不同，Arch Linux ARM for Raspberry Pi 并不能使用我们一般用的 RPi Imager 烧录。要安装 Arch，首先得准备一台已经装好了的 Linux 设备。可以使用虚拟机，这里我直接用树莓派的备用系统（一张烧好了官方系统的 SD 卡，作为后备还原系统）安装。

随便 pick 一张适合做系统的 SD 卡插入电脑，开始我们的安装。

## 安装

使用 `lsblk` 命令查看连接到设备的 SD 卡设备号，我这里为 `/dev/sda`。

确认无误后，使用 `sudo fdisk /dev/sda` （之后的 `/dev/sda` 都换成你的 SD 卡）开始分区。

步骤如下：

1. 输入 `o`，清除 SD 卡上现有的所有分区
2. 输入 `n`，新建一个分区，输入 `1` 代表创建设备上的第一个分区，直接回车使用默认起始位置，然后输入 `+200M` 指定结束位置。
3. 输入 `t`，然后输入 `c`，将刚刚创建的分区类型切换为 W95 FAT32 (LBA)。
4. 输入 `n`，再新建一个分区，输入 `2`，代表创建设备上的第二个分区，回车两次使用默认的起始和结束位置。
5. 输入 `w`，将刚刚的改动写入 SD 卡。

格式化新创建的分区：

```bash
$ mkfs.vfat /dev/sda1
$ mkfs.ext4 /dev/sda2
```

挂载分区：

```bash
$ mkdir boot root
$ mount /dev/sda1 boot
$ mount /dev/sda2 root
```

下载 rootfs 包，并将其解压到 `root` 目录下：

```bash
$ wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-aarch64-latest.tar.gz
$ bsdtar -xpf ArchLinuxARM-rpi-armv7-latest.tar.gz -C root
$ sync
```

{{< tip note 下载太慢... >}}
如果下载太慢，也可以使用国内源下载，将 url 换为 [https://mirrors.tuna.tsinghua.edu.cn/archlinuxarm/os/ArchLinuxARM-rpi-aarch64-latest.tar.gz](https://mirrors.tuna.tsinghua.edu.cn/archlinuxarm/os/ArchLinuxARM-rpi-aarch64-latest.tar.gz)
{{</ tip >}}

将启动文件复制到 `boot` 目录下：

```bash
$ mv root/boot/* boot
```

更新 `fstab`：

```bash
$ sed -i 's/mmcblk0/mmcblk1/g' root/etc/fstab
```

卸载 SD 卡：

```bash
$ umount boot root
```

接下来把 SD 卡插入树莓派，连上网线，在路由器查看树莓派的 IP 地址，然后使用 SSH 链接。默认用户名称和密码都为 `alarm`。登录上后切换到 `root` 用户（默认密码为 `root`），编辑 `/etc/pacman.d/mirrorlist` 换源：

```
$ su root
# nano /etc/pacman.d/mirrorlist
```

在文件开头添加以下内容：

```
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxarm/$arch/$repo
```

也可以自行寻找其他的源添加，Ctrl+X 保存退出。

接着初始化 pacman 密钥：

```
# pacman-key --init
# pacman-key --populate archlinuxarm
```

安装基本包：

```
# pacman -Syyu base-devel git
```

然后就完成了 Arch Linux ARM 在树莓派上的安装，接下来照例安装一下基本环境。

## 配置（可选）

默认用户名字叫 `alarm`，不太好听，改个自己的（`bs` 换成用户名）：

```
# useradd -m -g users -G wheel -s /bin/bash bs
```

安装 `yay` 包管理器

```bash
$ cd ~
$ git clone https://aur.archlinux.org/yay-bin.git
$ cd yay-bin
$ makepkg -si
```

基本软件包

```bash
$ yay -S vim tree neofetch
```

安装 `oh-my-zsh`

```bash
$ yay -S zsh curl
$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

常用 zsh 插件

```bash
$ cd ~/.oh-my-zsh/plugins/
$ git clone https://github.com/zsh-users/zsh-autosuggestions.git --depth 1
$ git clone https://github.com/zsh-users/zsh-syntax-higlighting.git --depth 1
```

编辑 `~/.zshrc`，修改 `plugins` 项

```
plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```

## 连接 WiFi

我们目前还是通过有线网连接到的网络，可以使用 `NetworkManager` 连接到 WiFi：

```bash
$ yay -S networkmanager
$ sudo systemctl enable NetworkManager
$ sudo systemctl start NetworkManager
```

显示附近的 WiFi：

```bash
$ nmcli device wifi list
```

连接 WiFi：

```bash
$ nmcli device wifi connect WiFi名称 password 密码
```

使用 `ip addr` 查看，发现已经连接到 WiFi了。
