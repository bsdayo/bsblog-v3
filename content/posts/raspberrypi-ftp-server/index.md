---
title: 树莓派搭建 FTP 服务器实现局域网文件共享
date: 2022-09-14T20:34:14+08:00

categories: tech
tags:
  - raspberrypi

image: cover.jpg

draft: false
---

## 起因

经常需要在电脑和手机之间互传文件，想了不少办法，最早是使用 QQ 等聊天工具进行文件传输，效率十分低下；后来找到了 [SnapDrop](https://snapdrop.net/) 这款工具，可以以类似面对面快传的方式在局域网内传输文件，提高了些许效率，但还是感觉到些许别扭。于是开始找寻一种更为优雅的文件传输方式，想到手里有一个半吃灰的树莓派，就开始着手搭建自己的 FTP 文件服务器。

{{< tip quote "什么是 FTP" >}}
文件传输协议（File Transfer Protocol，缩写 **FTP**）是一个用于在计算机网络上在客户端和服务器之间进行文件传输的应用层协议。
{{</ tip >}}

相比 WebDav 等其他传输协议，纯 FTP 的安全性略显低下，但对于小型自用而言已经绰绰有余。

## 搭建

{{< tip info 关于树莓派 >}}
本文中使用的树莓派信息如下：  
型号：4B  
RAM：4GB  
系统：Ubuntu Server 22.04
{{</ tip >}}

首先安装软件包 `vsftpd`

```bash
$ sudo apt install vsftpd -y
```

安装过程中会自动创建一个名为 `ftp` 的用户，我们给它设置一个密码作为稍后的登录密码（若打算开启匿名登录可跳过）：

```bash
$ sudo passwd ftp
```

{{< tip note "FTP 文件存放位置" >}}
FTP 服务开启后的文件存储位置就是 `frp` 用户的家目录，位于 `/srv/ftp`
{{</ tip >}}

密码更改完成后，选用喜欢的编辑器打开 FTP 服务器配置文件 `/etc/vsftpd.conf`，按照需求取消注释（去掉行首的 `#` 号）或修改相应选项（如果找不到被注释的选项在哪里也可以直接在文件末尾添加，格式为 `选项=值`）。这里列出几个常用的，`YES/NO` 分别对应是/否：

**通用**

**`local_enable`**：是否允许本地用户登录  
**`write_enable`**：是否允许写入操作（上传文件、新建文件夹等）  
**`utf8_filesystem`**：启用 UTF-8 支持

**匿名登录相关**

**`anonymous_enable`**：是否允许匿名登录  
**`anon_upload_enable`**：是否允许匿名用户上传文件  
**`anon_mkdir_write_enable`**：是否允许匿名用户新建文件夹

**[Chroot](https://zh.wikipedia.org/wiki/Chroot) 相关**

**`chroot_local_user`**：是否将连接 FTP 的本地用户的根目录限制为其家目录（`/home/<username>`）  
**`chroot_list_enable`**：启用 chroot 白名单，只有在其中的用户**才会**被 chroot（如果把 `chroot_local_user` 设置为 `YES` 将变为黑名单，只有在其中的用户**不会**被 chroot）  
**`chroot_list_file`**：chroot 黑/白名单的文件路径，文件中填写用户名，一行一个

注：如果只设置 `chroot_local_user` 为 `YES` 而不设置黑/白名单，会对所有连接 FTP 的本地用户进行 chroot。

修改好配置后重启 FTP 服务：

```bash
$ sudo systemctl restart vsftpd
```

## 连接

查看树莓派 IP，可以使用 `ip addr` 或 `ifconfig` 等等方式。这里举例为 `192.168.0.114`，后续命令中记得替换成自己的。

使用 FTP 工具进行连接测试，这里先使用 Linux 和 Windows 都自带的 `ftp` 命令：

```bash
$ ftp 192.168.0.114
```

提示输入用户名和密码，用户名输入 `ftp`，密码输入之前设置的密码，若设置了匿名就直接留空回车。

```
Connected to 192.168.0.114.
220 (vsFTPd 3.0.5)
Name (192.168.0.114:bs): ftp
331 Please specify the password.
Password:
230 Login successful.
ftp>
```

成功进入 FTP Shell 后可以输入 `ls`，`pwd` 等命令进行测试，使用 `quit` 退出。

{{< tip success 搭建完成 >}}
一个简单的 FTP 服务器已经搭建完毕，可以使用各种 FTP 工具连接使用。

如果搭建过程中遇到问题，可以参考文章后面的问题解决部分。
{{</ tip >}}

## 体验
搭建完成后，我分别使用移动端的 MT 管理器、和 Windows 自带的网络位置进行了测试。在 MT 管理器下连接后操作十分流畅，上传/下载的速度也很快。而 Windows 自带的网络位置则十分难用，打开文件只打开了一个空的 Edge，复制文件十分缓慢，新建文件夹需要延迟很久，删除文件也经常删不掉。~~fkms~~

但尽管如此，通过 FTP 传输文件仍然带来了一种全新的体验：手机上将文件移进”文件夹“，转眼就能在电脑上移出来使用，做到了几近原生、无缝的传输体验。相信如果解决了 Windows 端的工具问题，效率会大大提升。

## 问题解决
### 无法登录进 FTP Shell
{{< tip quote "Ubuntu 官网的说明" >}}
To allow users with a shell of /usr/sbin/nologin access to FTP, but have no shell access, edit /etc/shells adding the nologin shell.

This is necessary because, by default vsftpd uses PAM for authentication, and the /etc/pam.d/vsftpd configuration file contains:

`auth`&emsp;`required`&emsp;&emsp;`pam_shells.so`

The shells PAM module restricts access to shells listed in the /etc/shells file.

摘自[这里](https://ubuntu.com/server/docs/service-ftp)
{{</ tip >}}

解决办法：编辑 `/etc/shells`，在文件末尾添加一行 `/usr/sbin/nologin`

### 550 create directory operation failed
如果设置了 `write_enable` 后仍不能创建文件夹，需要确保根目录具有可执行权限。执行以下命令：
```bash
$ chmod 777 -R /srv/ftp
```

### vsftpd: refusing to run with writable root inside chroot

如果设置了 chroot 后无法新建文件夹/上传文件，并提示以上信息，是因为 `2.3.5` 版本后 `vsftpd` 加强了安全检查，用户的主目录不能再有写权限，如果检查到就会报错。可以在配置文件中添加 `allow_writeable_chroot=YES` 来解决。

参考自[这里](https://blog.csdn.net/bluishglc/article/details/42399439)
