# Visual Studio 代码:使用 Gulp 在保存时将 Less 自动编译为 CSS

> 原文：<https://itnext.io/visual-studio-code-auto-compile-less-to-css-on-save-using-gulp-2fa15bc7d954?source=collection_archive---------4----------------------->

![](img/dce14290cb4db3c5256dded4d175e65b.png)

照片由[内森·杜姆劳](https://unsplash.com/@nate_dumlao?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

我们开发接口的速度越来越快，使用保存编译来编译更少的 CSS 代码。

没有编辑器插件。没有肮脏的把戏。没有难懂的代码。

我将为您提供一个干净的 Javascript 代码片段，以及几秒钟内完成设置的简单步骤。

# 设置

您需要:

*   **一**一**项目目录** : `mkdir project`
*   **一饮而尽:**

在全球和您的项目中:

`npm install -g gulp`或`yarn global add gulp`

`cd project`

`npm install gulp`或`yarn add gulp`

*   仅在您的项目中

`cd project`

`npm install gulp-less`或`yarn add gulp-less`

你完了。前进。

# 简短的解释

**Gulp** 是一个 Javascript 任务运行器，可以让你自动化多余的任务。

它有很多非常有用的内置功能。

其中，我们将使用文件监视器，它允许我们在文件改变时执行任务。

**Gulp-Less** 是一个 Gulp 插件，专门用于将较少的文件编译成 CSS。它为我们提供了一个更少的编译器。

# 工作片段

*您可以根据自己的需求轻松定制。*

**首先，**在你的项目目录的根目录下创建一个文件，命名为`gulpfile.js`。

这就是 Gulp 期望你如何组织你的任务，所以不要用不同的名称。

**然后，**将这段代码复制粘贴到`gulpfile.js`:

**最后，**打开一个终端(使用 VSCode 中的`cmd`或`Ctrl + Shift + ù`)并运行该命令:

`gulp default`

**你完成了。**

每次你修改一个位于你的`srcDir`中的 Less 文件，它都会编译它并生成位于你的`dstDir`中的相应 CSS 文件。

*我所说的“修改文件”是指使用* `*Ctrl + S*` *或* `*File > Save*`

# 进一步的细节:它是如何工作的？

首先，我们只是导入两个模块:`gulp | gulp-less`。

然后，我们定义 Less 文件的位置，以及我们希望 CSS 文件生成的位置。

`Line 9`:我们正在定义一个名为“less”的 Gulp 任务，由我们的自定义函数定义。

`Line 11`:我们命令大口取`srcDir`中所有的`.less`文件作为源。(使用`*.less`

`Line 12`:我们命令 Gulp 将所有这些文件放入来自`gulp-less`模块的`less`函数(使用`pipe`)。这个函数将把每个文件编译成它们的 CSS 副本。

`Line 13`:我们命令 Gulp 从较小的源文件中取出所有动态创建的 CSS 文件，并保存到`srcDir`。

然而，这项任务本身并不能解决任何问题。这只是一个在某处定义的任务，它本身不会对文件变化做出反应。

**这就是为什么:**

`Line 17`:当位于`srcDir`的文件发生变化时，我们告诉 Gulp 执行`less`任务。

**最后，**运行`gulp default`将运行`default`任务，每次文件改变时，任务本身运行`less`任务。

# **更进一步**

起初，我不想开始学习海洋中的另一项技术，但是伙计，(相信我)值得花时间去学习。

Gulp 可以让你自动化大量的事情，比如:

*   文件缩小/美化。
*   浏览器兼容性的自动前缀。
*   保存时刷新浏览器。
*   许多其他耗时的任务你最好让 Gulp 为你处理😃

此外，VSCode 有一个`Tasks`选项卡，您可以从中运行 gulp 任务。这也是为什么`gulpfile.js`命名很重要。

更进一步，VSCode 允许你定义运行任务的快捷键，这样输入命令或者点击`Tasks`就不再是你的痛苦了。

[使用键盘快捷键运行任务请参见此链接](https://code.visualstudio.com/docs/editor/tasks#_binding-keyboard-shortcuts-to-tasks)

> 注意，运行`gulp default`会一直运行并观察文件，直到你关闭编辑器/命令行解释器。

# 感谢阅读

尽我所能在今年的每一周为您提供有价值的内容。

我肯定会感谢 50 次鼓掌👼

查看我的上一篇文章:

 [## 使用 Google Analytics 跟踪您网站上的定制事件

### 在跟踪用户方面，谷歌分析是一个非常强大的解决方案。但是，每个人都包含的小脚本…

dmware.fr](https://dmware.fr/track-custom-events-on-your-website-using-google-analytics/) 

欢迎致电 **david.mellul@outlook.fr.**

个人网站更多有价值的内容: [https://dmware.fr](https://dmware.fr)

![](img/4a6d158585592dad923f1c41ee8b582e.png)

[吴怡](https://unsplash.com/@takeshi2?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上的《白色马克杯里的卡布奇诺，白色泡沫艺术在木桌上》