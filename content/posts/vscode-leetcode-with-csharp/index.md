---
title: 使用 VSCode 打造 LeetCode 的 C# 刷题环境
date: 2022-11-20T20:51:55+08:00

categories: tech
tags:
  - vscode
  - leetcode
  - csharp

image: cover.jpg

draft: false
---

最近终于接触了大名鼎鼎的 LeetCode 刷题平台，开始苦痛之路（x

LeetCode 自带的网页编辑器体验并不好，代码补全、高亮等都不是很舒服，于是打算使用本地开发。尝试在 Rider 中找到了 IntelliJ 平台通用的 LeetCode 插件，但是对于 C# 支持并不是很好，有些水土不服，而且 Rider 也不很适合单个文件开发。最后我找到了 VSCode 的一款 LeetCode 刷题扩展，基本符合了我的需求。

这篇文章将简单记录一下结合 .NET / C# 使用的过程。

## 准备工作
1. 首先安装 [Visual Studio Code](https://code.visualstudio.com/download)。
2. 打开 VSCode，在左侧的扩展市场找到以下扩展并安装：
  - [**C#** _by Microsoft_](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)：基于 OmniSharp 提供对 C# 的语言支持，包括自动补全、语法高亮等。
  - [**LeetCode** _by 力扣 LeetCode_](https://marketplace.visualstudio.com/items?itemName=LeetCode.vscode-leetcode)：LeetCode 官方提供的刷题扩展，拥有题库查看、测试、提交等功能。[扩展的中文文档](https://github.com/LeetCode-OpenSource/vscode-leetcode/blob/master/docs/README_zh-CN.md)


## 配置 LeetCode 扩展和 .NET
新建一个文件夹并在 VSCode 中打开，这将作为我们 LeetCode 的工作区。

在左侧找到 LeetCode 图标打开侧边栏，首先点击上方的小地球标志切换为国内的站点（leetcode.cn），然后点击 Sign in to LeetCode，输入账号密码登录。加载好后侧边栏会显示账号目前可用的题库。

随便打开一道，点击页面右下角的 Code Now 按钮，提示选择使用的语言，这里选择 csharp (C#)，扩展会自动创建一个名为 `ID.题目名称.cs` 的文件，例如下面这样：

```csharp
/*
 * @lc app=leetcode.cn id=1 lang=csharp
 *
 * [1] 两数之和
 */

// @lc code=start
public class Solution {
    public int[] TwoSum(int[] nums, int target) {

    }
}
// @lc code=end
```

这个文件中，最上方的一块注释包含了题目的ID、标题，和用于 LeetCode 扩展识别的题目元数据。然后是两行特殊的注释，`// @lc code=start` 和 `// @lc code=end`。这两行中间的所有内容都会作为提交或测试题目时的提交内容，就像是网页编辑器中一样。而这两行之外的内容都不会提交，因此可以写一些本地测试用代码。

我们发现自动生成的文件直接放在根目录下，而且没有文件夹分隔。这样会显得很杂乱——尤其是需要为一道题编写多个题解的时候。LeetCode 扩展提供了一系列组件帮助我们自定义文件的生成路径。

打开设置页面，搜索 leetcode 并找到 **Leetcode: Workspace Folder** 设置项，将值更改为 LeetCode 工作区的目录。这一项指定了 LeetCode 文件生成的相对目录。

然后找到 **LeetCode: File Path** 项，点击进入 settings.json 中编辑。VSCode 会自动生成一些默认配置，我们对照修改即可：

```json
"leetcode.filePath": {
  "default": {
    "folder": "",
    "filename": "${id}-${kebab-case-name}.${ext}"
  }
}
```

修改 `folder` 和 `filename` 的值，其中 `folder` 的值将作为文件目录名的模板（相对于工作区根目录），`filename` 的值将作为生成文件名的模板。可以使用 `${xxx}` 作为预定义的插值，可以查看[官方文档](https://github.com/LeetCode-OpenSource/vscode-leetcode/wiki/%E8%87%AA%E5%AE%9A%E4%B9%89%E9%A2%98%E7%9B%AE%E6%96%87%E4%BB%B6%E7%9A%84%E7%9B%B8%E5%AF%B9%E6%96%87%E4%BB%B6%E5%A4%B9%E8%B7%AF%E5%BE%84%E5%92%8C%E6%96%87%E4%BB%B6%E5%90%8D)了解可使用的插值。

我根据个人习惯修改成了如下形式：

```json
"leetcode.filePath": {
  "default": {
    "folder": "src/${id}-${PascalCaseName}",
    "filename": "Solution-.${ext}"
  }
}
```

这样生成的文件路径会形如 `src/1-两数之和/Solution-.cs`。

{{< tip note 关于命名 >}}
这里我在 `Solution` 后加了一个横杠，方便有多个题解的同时存在。只需重命名并在 `-` 后面加上序号即可，例如 `Solution-1.cs` `Solution-2.cs`。
{{</ tip >}}

文件生成好了，但我们还不能正式开始编写代码。不信试试，你会发现除了少数几个关键字外根本没有代码提示...

其实解决办法很简单，我们只需要在项目里创建一个 `csproj` 文件，将工作区内的 cs 文件表示成一个项目就可以了（一个 `.csproj` 就可以，不用带上解决方案 `.sln` 文件）。

{{< tip note "csproj 文件的存放位置" >}}
这里我将 `LeetCode.csproj` 放在了 `src` 目录下。由于 `csproj` 文件的位置会影响到 `bin`、`obj` 文件夹的位置，所以你的位置如果不一样，记得同步更改后文的 `launch.json`。
{{</ tip >}}

以下是基本的内容：
```xml
<!-- LeetCode.csproj -->
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net6.0</TargetFramework>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
    </PropertyGroup>
</Project>
```

目前观察下来 LeetCode 在执行 C# 代码时是会带上 `ImplictUsings` 的，所以这里我们也带上，减少麻烦。同时根据官网说明，LeetCode 使用的是 .NET 6，因此我们指定 `TargetFramework` 为 `net6.0`。

创建完项目文件后重启 VSCode，让 OmniSharp 服务重新加载一下。此时再编写代码就会有代码提示了。

{{< tip note "报了几十个莫名其妙的错误？" >}}
如果遇到诸如无法识别 `System` 命名空间等奇葩问题，可以删除生成的 `bin` 和 `obj` 文件夹，然后重新手动执行 `dotnet restore`。 
{{</ tip >}}

## 调试配置
有时我们通不过测试，但又不知道哪里出现了问题，这时我们就可以对程序 debug。得益于 VSCode 高度自定义化的调试配置，即使是 LeetCode 题解这样奇怪的项目结构也是可以调试的。

由于每个文件都是一个题解，常规的 .NET 调试方法并不适用，而所有题解共用一个入口点就过于麻烦了。这里采用的方法是将题解分隔至不同的命名空间，并在调试编译时指定 `StartupObject` 来指定启动的 `Main` 方法，从而执行单个题解的调试。

首先我们在刚刚生成的 cs 文件顶端添加一行 `namespace LeetCode.P1.S1;`。这是我个人的命名方法，代表 1 号题目 (**P**roblem) 的第 1 个题解 (**S**olution)。你也可以修改成自己的样式，注意同时需要在后文的 `tasks.json` 中同步更改。

在工作区根目录新建 `.vscode` 文件夹，并在其中新建以下文件和内容：

`.vscode/tasks.json`
```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "command": "dotnet",
            "type": "process",
            "args": [
                "build",
                "${workspaceFolder}/src/LeetCode.csproj",
                "-p:StartupObject=LeetCode.P${input:problem}.S${input:solution}.Program",
                "-p:GenerateFullPaths=true",
                "/consoleloggerparameters:NoSummary"
            ],
            "problemMatcher": "$msCompile"
        }
    ],
    "inputs": [
        {
            "id": "problem",
            "description": "Please enter the problem ID.",
            "default": "",
            "type": "promptString"
        },
        {
            "id": "solution",
            "description": "Please enter the solution number.",
            "default": "1",
            "type": "promptString"
        }
    ]
}
```

在 `dotnet build` 的选项中，我们指定了 `-p:StartupObject=LeetCode.P${input:problem}.S${input:solution}.Program`。这采用了 VSCode 的配置语法，执行该 task 时，VSCode 会展示两个输入框，一个是题目ID，一个是题解ID。例如用户输入 123、1 后会拼接成 `-p:StartupObject=LeetCode.P123.S1.Program`。选项意味着将程序的入口点设置为 `LeetCode.P123.S1.Program` 中的 `Main` 方法。

底下的 `inputs` 块是对上面两个输入框数据的描述。

然后我们创建 `.vscode/launch.json` 文件：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": ".NET Core Launch (console)",
            "type": "coreclr",
            "request": "launch",
            "preLaunchTask": "build",
            "program": "${workspaceFolder}/src/bin/Debug/net6.0/LeetCode.dll",
            "cwd": "${workspaceFolder}",
            "console": "internalConsole",
            "stopAtEntry": false
        }
    ]
}
```

这个文件由 VSCode 自动生成的修改而来，只动了 `program` 项，将其指定为编译后的 DLL 文件路径。

在题解文件中，`// @lc code=end` 行下方（这样就不会把测试代码提交至 LeetCode）添加一个 `Program` 类，并添加一个 `Main` 静态方法，在其中实例化 `Solution` 类并调用题目方法。整个文件如下（以题目 9.回文数 举例）：

```csharp
/*
 * @lc app=leetcode.cn id=9 lang=csharp
 *
 * [9] 回文数
 */

// 指定命名空间
namespace LeetCode.P9.S1;

// 以下内容会被提交至 LeetCode
// @lc code=start
public class Solution
{
    public bool IsPalindrome(int x)
    {
        // 题解代码...
        // 你可以在任意地方打上断点调试。

        return true;
    }
}
// @lc code=end
// 以上内容会被提交至 LeetCode

public static class Program
{
    // 程序从这里开始
    public static void Main()
    {
        // 实例化 Solution 类
        var solution = new Solution();

        // 调用方法，进入调试
        solution.IsPalindrome(1000021);
    }
}
```

按 F5 开启调试，提示输入题目 ID ( 9 ) 和题解序号 ( 1 )，随后成功启动调试。

## 一些小技巧
对于测试，每次手动输入 `namespace` 和 `Program` 之类的东西还是有点麻烦。我们可以编写一些 code snippets 来达到自动补全的效果。

新建 `.vscode/csharp.code-snippets` 文件，输入以下内容：

```json
{
  "SolutionNamespace": {
    "scope": "csharp",
    "prefix": "ns",
    "body": [
      "namespace LeetCode.P$1.S$2;"
    ],
    "description": "Solution Namespace"
  },
  "TestProgram": {
    "scope": "csharp",
    "prefix": "tp",
    "body": [
      "public static class Program",
      "{",
      "    public static void Main()",
      "    {",
      "        var solution = new Solution();",
      "        ",
      "        $1",
      "    }",
      "}"
    ],
    "description": "Test Program"
  }
}
```

这样输入 `ns` 并 Tab 就能自动补全命名空间，输入 `tp` 就能自动补全测试类。