# 构建 FFmpeg WebAssembly 版本(= ffmpeg.wasm):第 1 部分准备

> 原文：<https://itnext.io/build-ffmpeg-webassembly-version-ffmpeg-js-part-1-preparation-ed12bf4c8fac?source=collection_archive---------4----------------------->

![](img/8c0c378d4f8e771bf56ebf20e4685f63.png)

> 2020 年 8 月更新:修改故事，使其在 MacOS 中工作。

在这一部分，您将学习:

1.  这个系列的背景
2.  如何用 docker(以及在 MacOS 中不用 Docker)构建原生 FFmpeg

# 这个系列的背景

这一系列故事旨在达到以下目的:

1.  对于想学习如何使用 Emscripten 将 C/C++库编译成 JavaScript 的人来说，这是一本指南(希望是目前为止最有用和详细的)
2.  个人笔记

# 为什么是 FFmpeg？

FFmpeg 是一个免费的开源项目，由一个庞大的软件套件库和程序组成，用于处理视频、音频和其他多媒体文件和流。(来自维基百科)

这是一个非常有用的库，没有一个 JavaScript 库具有完全相同的功能。如果你在谷歌上搜索“ffmpeg.js ”,你会发现很少有现存的库和我们将要构建的完全一样:

*   ffmpeg . js:[https://github.com/Kagami/ffmpeg.js](https://github.com/Kagami/ffmpeg.js/)
*   https://github.com/bgrins/videoconverter.js

在大多数情况下，这些库都很棒，随时可用，但是它们存在以下问题:

1.  FFmpeg 和 Emscripten 的版本都已过时。
2.  多年未进行有效维护。*(Kagami/ffmpeg . js 2020 年 4 月继续开发)*

我曾考虑接管其中一个库，但由于这些年来发生了太多的变化，我决定从头开始，同时编写了这一系列教程来帮助人们学习如何在真实的 C/C++库中使用 Emscripten。

# 如何用 Docker 构建原生 FFmpeg

首先，我们需要从它的存储库中克隆 FFmpeg 源代码，因为`master`分支正在开发中，我们最好选择一个特定的版本来编译。

在我写这个故事的时候，FFmpeg 的最新稳定版本是 n4.3.1，所以我们将在整个故事中使用这个版本。

在完成了存储库的克隆之后，是时候用 GCC 构建以确保它能够工作了。

实际上，如果你很急的话，可以跳过这一部分，但是根据我自己的经验，最好是先熟悉一下库的构建系统。

构建和安装 FFmpeg 的指令可以在资源库的根目录下的 [**INSTALL.md**](https://github.com/FFmpeg/FFmpeg/blob/n4.3.1/INSTALL.md) 中找到:

```
Installing FFmpeg:1\. Type `./configure` to create the configuration. A list of configure options is printed by running `configure -help`.`configure` can be launched from a directory different from the FFmpeg sources to build the objects out of tree. To do this, use an absolute path when launching `configure`, e.g. `/ffmpegdir/ffmpeg/configure`.2\. Then type `make` to build FFmpeg. GNU Make 3.81 or later is required.3\. Type `make install` to install all binaries and libraries you built.NOTICE
 - - -- Non system dependencies (e.g. libx264, libvpx) are disabled by default.
```

因为我们不需要实际安装 FFmpeg，所以只需要第 1 步和第 2 步。

有两种构建方式，一种是原生方式，需要你安装软件包(例如 emsdk，Node.js)。大多数时候它是有效的，但是有时你可能会面临由于软件包版本和操作系统的变化而难以解决的错误。另一种方法是使用 [Docker](https://www.docker.com/) ，它提供了一个稳定和静态的构建环境。强烈推荐使用 Docker，因为它可以节省您安装(和删除)软件包的时间。

我不会在这里介绍如何安装包，但是我把脚本分成了`build.sh`和`build-with-docker.sh`，你可以自己安装所有的包并运行`build.sh`。

> 为了确保本教程能够最大限度地覆盖环境，我使用 Github 动作来测试它是否能在 Linux 和 MacOS 上工作。对于 Linux 用户，我会使用 Docker way / `build-with-docker.sh`来构建。对于 MacOS 用户，由于 Github Actions 不支持 Docker，所以我会使用 native way / `build.sh`来构建。

现在，让我们用下面的内容创建一个名为`build.sh`的文件。

要以本机方式构建，只需运行命令:

```
$ bash build.sh
```

要使用 Docker 构建，创建一个名为`build-with-docker.sh`的文件，包含以下内容:

和运行命令:

```
$ bash build-with-docker.sh
```

> `--disable-x86asm`是禁用 x86 汇编特性所必需的，因为我们不打算使用它。
> 
> 根据你的网速和电脑的硬件规格，完成编译可能需要 10~30 分钟。
> 
> 当 gcc 9 引入更多的约束时，你在编译过程中看到大量的警告是正常的。

编译原生 FFmpeg 应该需要一些时间。如果一切正常，您应该可以使用以下命令运行`ffmpeg`:

```
$ ./ffmpeg
```

您应该会看到类似这样的内容:

> ffmpeg 版本 n4.3.1 版权所有 2000–2019 FFmpeg 开发者
> 用 gcc 8 (GCC)
> 构建的配置:—disable-x86 ASM
> libavutil 56。22.100 / 56.22.100
> libavcodec 58。35.100 / 58.35.100
> libavformat 58。20.100 / 58.20.100
> libavdevice 58。5.100 / 58.5.100
> libavfilter 7。40.101 / 7.40.101
> libswscale 5。3.100 / 5.3.100
> libswresample 3。3.100 / 3.3.100
> 超快速音视频编码器
> 用法:ffmpeg[options][[infile options]-I infile]…{[outfile options]outfile }…
> 
> 使用-h 获得完整的帮助，或者更好的是，运行' man ffmpeg '

你可以访问这里的知识库，看看它是如何详细工作的:[https://github.com/ffmpegwasm/FFmpeg/tree/n4.3.1-p1](https://github.com/ffmpegwasm/FFmpeg/tree/n4.3.1-p1)

并且可以在这里随意下载建造神器:[https://github.com/ffmpegwasm/FFmpeg/releases/tag/n4.3.1-p1](https://github.com/ffmpegwasm/FFmpeg/releases/tag/n4.3.1-p1)

现在我们已经完成了准备，下面我们继续进入[构建 FFmpeg WebAssembly 版本(= ffmpeg.wasm): Part.2 用 Emscripten 编译](https://medium.com/@jeromewus/build-ffmpeg-webassembly-version-ffmpeg-js-part-2-compile-with-emscripten-4c581e8c9a16)开始用 Emscripten 编译 FFmpeg。😃