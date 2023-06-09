# 热重装原生 ES2015 模块

> 原文：<https://itnext.io/hot-reloading-native-es2015-modules-dc54cd8cca01?source=collection_archive---------5----------------------->

正如我的同事曾经讽刺地评论的那样，曾经有一段时间 JavaScript 只能在浏览器中工作，而现在它本质上是一种编译语言。现在，不要误解我，像 webpack、rollup 或 package 这样的工具给现代前端开发带来了巨大的优势。即使现在浏览器已经支持本地模块，你仍然应该捆绑你的代码用于生产。但是发展呢？JavaScript bundlers 也改进了一些东西:我特别喜欢的一个改进是热重载。事实上，我非常喜欢它，所以我决定看看是否有可能使用原生 ES2015 模块实现类似的东西。

对于那些不熟悉的人来说，热重装就是不用重装整个页面就能替换更新的模块。通常，整个系统包括:

*   文件监视器
*   WebSocket 或 EventSource 连接，通知客户端文件的更改
*   处理热更新的运行时代码。

你可以在 [webpack](https://webpack.js.org/concepts/hot-module-replacement/) 或[package](https://parceljs.org/hmr.html)文档中找到关于这个特性的内部和公共 API 的更多细节。对于本文，我将提供一个小例子:

我们从`text.mjs`模块中导入`text`变量，并将其放入文件`body`中。然后我们订阅`text.mjs`更新，并保持正文与模块内容同步。

现在，我们将尝试在不捆绑的情况下为原生 ES2015 模块实现此功能。

总有一天， [WHATWG Loader 规范](https://github.com/whatwg/loader)将会完成，并有望提供所有必要的 API 来轻松实现热模块替换。但是即使在今天，原生 JavaScript 模块也有一些有趣的属性，这在某些情况下是可能的。首先，导入是动态绑定:

在这个例子中，每次我们改变`counter`，所有其他依赖于它的模块都接收新值，就好像`counter`和`increment`函数都在同一个模块中定义一样。

第二个属性，对重载部件很重要:模块名是 URL，不同的 URL 被认为是不同的模块，即使它们最终解析为同一个文件。为了演示它，让我们稍微改变一下前面的例子:

由于`counter`现在是从不同的 URL 导入的，`increment`调用不会改变它。`counter.mjs`和`counter.mjs?someQueryParameter`是两个独立的模块。如果我们在浏览器开发工具中查看网络选项卡，我们也会看到它们是两个不同的请求:

![](img/f4e4fe1e0775b65fc8b0fa17f4edd6d1.png)

了解了这一点，我们可以实现一个服务器，它将能够热重装 ES2015 模块，但有一些限制(见下文)。它将按以下方式工作:

*   服务器将运行文件监视器和 WebSocket 端点。每次模块改变时，它将向所有连接的客户端广播改变的模块名称和修改时间。
*   在前端，我们将实现小型 WebSocket 客户端，这将允许我们订阅特定模块的更新。
*   在`module.mjs?mtime=<anything>` 航线下，我们将提供原始模块。
*   在`module.mjs`下，我们将生成并提供具有自我更新能力的代理模块，而不是原始源代码:

1.  我们从原始模块导入每个导出的名称。
2.  我们将导入分配给模块局部变量。我们使用可变的`let`绑定，这样我们可以在以后重新分配它们。
3.  我们以原始名称导出模块局部变量。
4.  我们订阅原始模块更新通知。
5.  每次我们收到更新，我们用一个新的`mtime`查询参数重新导入模块。由于 URL 语义，这将导致浏览器加载更新的模块。
6.  我们将模块局部变量更新为我们在步骤 5 之后收到的值。

这个代理模块是这种技术的核心思想。因为它具有相同的导出，并且在相同的 URL 下提供服务，所以它可以透明地替换原来的。由于 ES2015 模块的动态绑定特性，每个消费者都会自动收到代理模块导出的所有更新。

这种方法也有两个局限性:

1.  如果出口名称或编号发生变化，我们无法更新模块。在这种情况下，我们将退回到整页重载
2.  如果原始模块导出可变绑定，就像在我们的`counter`例子中，我们不能生成代理模块:因为我们将所有的导出重新分配给不同的变量，它们不能像在原始模块中那样正确地更新。因此，我们不会为这些模块生成代理，如果它们发生变化，我们会重新加载整个页面。

本文还遗漏了一些东西:在最后一个演示中，有一个 API 对模块重载(`accept`和`dispose` 回调)做出反应，更新通知传播到模块父节点，直到它们被处理。

你可以在我的 [GitHub repo](https://github.com/SevInf/heiss) 找到源代码和演示，或者用这个 [npm 包](https://www.npmjs.com/package/heiss)在你自己的代码上试试。让我知道你的想法。这种或任何类似的方法有任何前景吗？