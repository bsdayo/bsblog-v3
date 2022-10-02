---
title: SkiaSharp 的 Type Initializer Exception 解决记录
date: 2022-10-02T14:46:48+08:00

categories: tech
tags:
  - skiasharp
  - csharp

image:

draft: false
---

给树莓派重新装了系统，重新把自己的 bot 挂上去，发现生成图片的时候会报出奇怪的错误：

```
TypeInitializationException: The type initializer for 'SkiaSharp.SKImageInfo' threw an exception.
   at SkiaSharp.SKImageInfo..ctor(Int32 width, Int32 height)
   ...
```

查询一番后，大多都是说是 SkiaSharp 的版本问题，升级就好，但是我已经在使用最新版本了）最后在[这里](https://github.com/mono/SkiaSharp/issues/964#issuecomment-541399218)找到了解决方案。

**报错的原因是：系统缺少了 `libfontconfig`，导致字体相关操作无法完成。**

那就很简单了，使用 `yay` 找找相关的包：

```bash
$ yay libfontconfig

2 extra/libfontenc 1.1.6-1 (14.2 KiB 35.3 KiB)
    X11 font encoding library
1 extra/fontconfig 2:2.14.0-1 (340.5 KiB 1.0 MiB)
    Library for configuring and customizing font access
==> 要安装的包 (示例: 1 2 3, 1-3 或 ^4)
==> 1
```

看来就是这个 `extra/fontconfig` 了，安装上去，重启应用再试了试，生成图片一切正常了。
