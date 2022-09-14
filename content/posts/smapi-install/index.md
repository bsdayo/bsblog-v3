---
title: 星露谷模组加载器 —— SMAPI 的安装
date: 2022-09-13T19:34:50+08:00

categories: tech

draft: false
---

一直听说[星露谷](https://store.steampowered.com/app/413150/Stardew_Valley/?l=schinese)对于 Mod 的支持十分完善，这次打算自己尝试一番。

{{< tip info 注意 >}}
本文仅针对 Windows 平台，其他平台方法大同小异，可以直接参照[官方 Wiki](https://zh.stardewvalleywiki.com/%E6%A8%A1%E7%BB%84:%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97/%E5%85%A5%E9%97%A8)。
{{</ tip >}}

首先到 [SMAPI 官网](https://smapi.io/)下载最新的安装包，可以选择从 NexusMods (N 网) 下载，也可以直接下载 (Direct download)。截至本文，SMAPI 的最新版本为 `3.16.2`。

把下载下来的 `SMAPI-<版本>-installer.zip` 解压到任意位置，打开里面的 `install on Windows.bat`，安装脚本会自动检测游戏的安装位置。如果没有找到会提示以下信息：

```
Oops, couldn't find the game automatically.

Type the file path to the game directory (the one containing 'Stardew Valley.dll'), then press enter.
```

出现这种情况，首先找到星露谷的安装目录，若为 Steam 安装可以直接右键库里的星露谷，选择管理 - 浏览本地文件，然后复制打开的文件夹路径（例如我的是 `D:\Steam\steamapps\common\Stardew Valley`），再粘贴进终端内并回车。

```
What do you want to do?

[1] Install SMAPI.
[2] Uninstall SMAPI.

Type 1 or 2, then press enter.
```
输入 1 并回车继续安装。日后如果想要卸载 SMAPI，可以再次运行此脚本并选择 2 卸载。

如果出现以下信息，则为安装成功：

```
SMAPI is installed! If you use Steam, set your launch options to enable achievements (see smapi.io/install):
    "D:\Steam\steamapps\common\Stardew Valley\StardewModdingAPI.exe" %command%

If you don't use Steam, launch StardewModdingAPI.exe in your game folder to play with mods.
```

提示信息中指出，如果使用 Steam 启动游戏，可以设置启动项来确保安装 SMAPI 后也能正常获得成就。

{{< tip warn 注意 >}}
在配置Steam启动项之前，需要确保您的游戏路径里没有中文，否则将设置失败。
{{</ tip >}}

首先复制终端窗口中给出的启动指令（如上文，为 `"D:\Steam\steamapps\common\Stardew Valley\StardewModdingAPI.exe" %command%`），Steam 中右键星露谷打开属性，在“通用”选项卡中把复制的指令粘贴进“启动选项”一栏，保存即可。

至此已经完成了 SMAPI 的安装，至于添加 Mod 下次再讲 ~~（咕咕咕（（~~
