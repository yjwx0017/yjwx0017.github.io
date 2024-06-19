title: 使用 vcpkg 安装 C++ 依赖库
categories: 日志
tags: [vcpkg]
date: 2021-11-12 16:15:00
---
[包管理工具Vcpkg 的使用][1]
[教程：从清单文件安装依赖项][2]

Linux下开发，用到什么依赖库，直接运行`apt install xxx` 安装即可，好用到爆炸。Python、Node.js等也都有类似的包管理器。而VS环境下需要自己下载各种依赖库的源码、编译，折腾半天，最后还不一定能用。

而 vcpkg 的诞生，给我们带来了福音。

## 简介

vcpkg 是 Microsoft 的跨平台开源软件包管理器，极大地简化了 Windows、Linux 和 macOS 上第三方库的配置与安装。如果项目要使用第三方库，建议通过 vcpkg 来安装它们。vcpkg 同时支持开源和专有库。

vcpkg 为我们处理好了各种软件包的依赖关系和编译配置，安装软件包时软件包源码被下载下来，并通过 CMake 在本地编译。目前官方和社区提供了对各种平台和编译器的支持。

## 安装

```
git clone https://github.com/microsoft/vcpkg
cd vcpkg 
.\bootstrap-vcpkg.bat
```

## 经典模式

在经典模式下，使用 vcpkg 作为命令行接口在全局安装目录中安装依赖项。 该目录通常位于 `%VCPKG_ROOT%/installed`，其中 `%VCPKG_ROOT%` 是 vcpkg 安装目录所在的位置。

查询软件包：

```
.\vcpkg.exe search xxx
```

安装软件包：

```
.\vcpkg.exe install xxx
```

## 清单模式

清单模式是大多数用户推荐的工作流。

在清单模式下，在名为 vcpkg.json 的清单文件中声明项目的直接依赖项。

清单文件有自己的 vcpkg_installed 目录，可在其中安装依赖项，与所有包都安装在全局 `%VCPKG_ROOT%/installed` 目录中的经典模式不同。每个项目都可以有自己的清单和自己的一组依赖项，这些依赖项不会与其他项目的依赖项发生冲突。

假设项目需要以下依赖库：cxxopts、fmt、range-v3。

在项目所在的同一目录中创建一个名为 vcpkg.json 的文件：

``` json
{
  "dependencies": [
    "cxxopts",
    "fmt",
    "range-v3"
  ]
}
```

如果想要手动安装依赖项，只需在包含清单文件的目录中运行 `vcpkg install`。

该命令完成后，所有生成的包都将出现在 vcpkg_installed 目录中。 此目录的具体位置取决于构建系统；通常，在构建系统的默认输出文件夹内，或 vcpkg.json 文件旁边。

  [1]: https://zhuanlan.zhihu.com/p/153199835
  [2]: https://learn.microsoft.com/zh-cn/vcpkg/consume/manifest-mode