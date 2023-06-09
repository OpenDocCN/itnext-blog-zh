# 循环中的承诺呢？

> 原文：<https://itnext.io/https-medium-com-popov4ik4-what-about-promises-in-loops-e94c97ad39c0?source=collection_archive---------0----------------------->

![](img/a6bb46bb07c720dde009164c8717ff41.png)

彬彬有礼的绅士

我听说人们“并不真正理解承诺”，每个月至少几次，每次我都认为我不是那些人之一。

在这里，我将描述我的团队最近面临的一个案例及其解决方式。只是说，虽然我不是回调和实现承诺的“老方法”的大粉丝，但我确实喜欢*异步/等待*方式。

将会有我们已经经历的步骤的描述，所以如果你需要**最终解决方案**——滚动到文章的结尾。

# 问题描述

那么首先，我来描述一下这个问题。我们有一个值数组，我们需要迭代这个数组，并将数组的每个元素作为参数之一传递给一个函数(函数返回一个*承诺*)。第二个参数是数据库连接。最后，我们需要将一组值(不是承诺)传递给另一个函数。此外，该函数返回一个对象，该对象的值将在同一个循环中使用。主要目的是获得最佳的性能结果和良好的代码可读性。

# 初始版本

第一版代码:

上面的例子有一个很好的*异步/等待*语法，而我们假装使用*承诺*。 **asyncForEach()** 正在等待回调，立即解析它，并将解析的值推送到 finalArray。在这种情况下，来自 **asyncFunction()** 的值被立即解析，因此我们并没有真正从*承诺*中受益。这里我们有一个漂亮的主流*异步/等待*语法:)为了提高性能，我们需要将所有的*承诺*推送到一个数组中，然后一次性解决，就像这里的[所示的](https://hackernoon.com/async-await-essentials-for-production-loops-control-flows-limits-23eb40f171bd)。

但是在这种情况下，我们的代码看起来会像这样:

这种方法会稍微提高性能。然而，可读性会更差，并且会出现额外的代码(另一个循环)。此外，我们仍然没有获得最佳性能(迭代解析的值需要额外的时间/资源)。所以，它决定不实现它，并试图想出一个更“promisified”的方式。

# 回调时间到了？

更好的做法是给*一个承诺*一个一旦解决就需要做的行动的指示。**来了然后()**。我们可以将函数调用推送到一个数组中而不解析它，并将值操作的指令包含到回调中。这意味着一旦*承诺*将被解析(在 **forEach()** 循环之后),我们真的不需要担心另一个循环来迭代每个值，因为在解析期间它已经在那里了。这是代码:

回调得到一个*承诺*，一旦到达 **Promise.all()** ，回调就被解决。没有额外的循环，没有时间浪费，没有额外的代码。这种情况下的性能明显好于最初的情况。

如果它仍然不工作，不要担心:可能你仍然在使用一个 *MySQL* 连接 *Promises* 。使用*池*代替。每次循环迭代时都打开连接，考虑到 **asyncFunction()** 与 **myArray** 的每个元素并行(几乎)运行，这是不“健康”的。使用 **createPool()** 将同时有许多到数据库的连接，并将重用已经完成前一次运行的连接。*池*是异步操作的完美解决方案(除非你不考虑限制和/或试图终止应用程序与 *MySQL* 数据库的连接)。但是 **then()** 是“老办法”，所以不够好。

# 最终解决方案

那么如何才能摆脱回调，提高可读性，保持性能不变呢？以下是最终解决方案:

或者:

如果你喜欢添加新的 npm 包，看看 [*蓝鸟*](https://www.npmjs.com/package/bluebird) ，它有 **Promise.map()** ，这两个包都可以做——**promise . all()**和 **map()** 。

# 结论

最终的解决方案具有最好的性能，代码可读性足够好，并且没有额外的代码。这个案例帮助我更好地理解了*承诺*，但我仍然不确定我是否是“真正理解承诺”的人之一。