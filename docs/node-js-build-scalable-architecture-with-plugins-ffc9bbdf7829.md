# Node.js 用插件构建可扩展架构

> 原文：<https://itnext.io/node-js-build-scalable-architecture-with-plugins-ffc9bbdf7829?source=collection_archive---------1----------------------->

![](img/c4511a9e8dc48ad8aecf852ecf220431.png)

照片由[埃德加·恰帕罗](https://unsplash.com/@echaparro?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/search/photos/organized?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄

任何服务器工程师的理想架构都是一组高度模块化和可重用的组件。然而，事实证明这种架构很难实现。当您的代码库不断增长时，尤其如此。一个可能有助于扩展代码的技巧是将可重用组件分成插件。在这篇简短的博文中，我想回顾一下使用这种方法的好处和原因。

# 什么是插件？

插件是以可扩展和可重用的方式扩展主应用程序功能的组件。Node.js 如此受欢迎的原因之一不仅仅是因为它卓越的架构，还因为它的大量插件包，由社区创建，为我们提供了几乎任何我们可能需要的功能。

# 分发插件

## 公共插件

如前所述，绝大多数插件都是以 npm 包的形式发布的，可以安装到项目的`node_modules`文件夹中。所有的公共插件都是通过 npm 发布的，任何人都可以使用。这种插件最突出的例子之一是 [express](https://www.npmjs.com/package/express) 。

## 私有插件

但是，插件不一定需要是公共的。插件也可以只为私人使用而编写，在组织内部，甚至是一个单独的项目。分发和共享私有插件有不同的方式:

*   **版本控制系统** —您可以简单地将您的插件添加到您的团队所使用的版本控制中。
*   **私有 npm 包** —许多组织使用 npm 来打包他们的插件，并且只在组织内部共享。

虽然内部插件可能使用主应用程序的依赖关系，但插件拥有自己的依赖关系图和`package.json`也是有益的。这是一个示例图:

如你所见，我们的项目中有三个`package.json`文件:一个用于主应用程序，两个用于`node_modules`文件夹中的每个插件。以下是您可能希望采用这种方法的三个原因:

1.  **NPM** —我们可以利用 NPM 将我们的插件作为私有包分发。
2.  **依赖关系图** —这些插件有自己的依赖关系图，这使得维护项目的依赖关系和解决冲突更加容易。
3.  **更容易导入** —导入这些插件要容易得多。我们不需要使用相对路径(`require('../../utils/moduleA')`)，只需要提供插件的名称(`require('pluginA')`)。

# 为什么要把你的代码组织成插件？

## 更好的结构

那么为什么要使用插件呢？除了前面的考虑之外，将应用程序的部分功能分离到一个插件中的主要原因之一是整体架构。通过把你的代码分成插件，你把你的代码分成既可重用又松散耦合的单元。它迫使你考虑封装，考虑插件的哪些部分应该暴露给主应用程序。总的来说，它使您的应用程序更具可伸缩性。

## 无服务器架构

无服务器架构可以说是 web 开发的未来。虽然这种体系结构提供了比传统服务器更好的可伸缩性，但是存在在无服务器功能之间共享公共组件的问题。

无服务器架构中的功能通常是执行特定计算或任务的相对较小的代码片段。它通常被设置为独立的微服务，默认情况下不与项目中的其他功能共享代码。

如前所述，由于每个函数都是独立的服务，所以在函数之间共享公共代码可能会很麻烦。原因是当部署时，每个功能运行在一个单独的环境中。

这就是打包你的插件并把它们作为 npm 模块分发的好处。你所需要做的就是将你的插件发布为一个私有包，并根据需要安装在你的函数中。

# 编写插件

让我们看看如何编写一个插件来扩展简单的无服务器功能。想象一下这种项目结构。

这里需要注意一些事情:

*   `serverlessF`文件夹包含我们的无服务器功能。这是一个简单的 Node.js 应用程序。
*   `utils`文件夹是为我们项目中多个功能共享的代码准备的。它由不同的插件组成，每个插件都有自己的 package.json。
*   `pluginA`作为私有包发布，由我们的无服务器功能安装。

下面是一个验证第三方令牌的简单插件的示例代码:

我们的插件公开了一个验证令牌的函数。这里一个有趣的技巧是第一行。如果你看上面的图表，你会注意到我们的 lambda 函数包含了`fetch.js`,对于我们的目的来说，它用于向第三方 API 发出各种获取请求。使用`module.parent.require('./fetch')`,我们能够通过模仿来自安装插件的父应用程序的`require`调用，在我们的插件中使用那个模块。

虽然使用`module.parent.require`看起来像是一个巧妙的技巧，但是很多时候建议不要依赖它。原因是它在我们的无服务器功能和插件之间创建了紧密耦合。如果我们将`fetch.js`移动到不同的子文件夹，我们的插件会崩溃。更好的方法是使用依赖注入为我们的插件提供一个`fetch`服务的实例:

**注意**:在编写插件时，避免硬连线依赖和创建紧密耦合尤为重要，否则插件的目的就落空了。你可以在这里阅读更多关于模块接线和依赖[的信息。](https://isamatov.com/node-module-wiring-dependencies/)

# 结论

作为一个小小的免责声明，你不必把你的应用程序的每一部分都分成一个插件，因为那样会增加不必要的复杂性。然而，如果你有一个很大的依赖图和/或一组内聚模块，将它们分成插件可能是值得的。此外，将代码打包成插件是在无服务器功能之间共享公共代码的好方法。

*原载于 2019 年 6 月 15 日*[*https://isamatov.com*](https://isamatov.com/node-build-scalable-architecture-with-plugins/)*。*