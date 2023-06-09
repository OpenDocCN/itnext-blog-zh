# 转换器加速 JavaScript 数组

> 原文：<https://itnext.io/using-transducers-to-speed-up-javascript-arrays-92677d000096?source=collection_archive---------1----------------------->

![](img/95b6856b235e66a8285efc34c765d4dd.png)

图片由[森石油](https://unsplash.com/@simson_petrol?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 拍摄

## 处理大型 JavaScript 数组真的很慢

在本文中，我将介绍使用转换器提高大型 JavaScript 数组处理速度的方法。本文建立在我上一篇文章的发现之上: [加速 JavaScript 数组处理](/speed-up-javascript-array-processing-8d601c57bb0d) *。*

***TL；dr*** *JavaScript 数组处理很慢，但是有了传感器，你可以解决很多这样的问题。尽管如此，你还是要首先考虑你的原生选择。*

我从未想过我会需要它，但我实际上需要使用我的解决方案 [*加速 JavaScript 数组处理*](/speed-up-javascript-array-processing-8d601c57bb0d) 来加速加载我最近正在编写的一个 Chrome 插件。

这个插件在一个列表中加载了超过 26，800 个条目，并对它们进行了一些处理。令人惊讶的是，即使有几个`filter`和`map`函数，它还是非常快。直到我开始做一些高级处理，需要将数组的大小复制到 53，700 多项。这时，我注意到速度下降了许多个数量级。

***免责声明:*** *所有代码示例都是杜撰的，但我把它们做得和实际实现差不多。这些例子的处理时间比我的实际实现要快得多，但是在较慢的机器上，您会看到更慢的速度。*

# 一切都很好

在进行任何更改之前，让我们看一下原始代码:

这可能可以通过使用传感器来改善，但我认为没有必要，因为处理时间平均为 8.5 毫秒

尽管如此，让我们看看传感器版本:

根据我以前的文章，您会认为这样会更快，但事实并非如此。时钟为 22.1 毫秒，列表中没有足够的项目使传感器成为比本地`Array`方法更快的替代方法，也没有足够的数据转换。这很简单，所以除非我们增加更多的条目或操作符，否则不会提高速度。

# 经济放缓

这个插件的加载几乎是即时的，但是突然之间，这个小小的改变让它花费了 35 秒，除了处理一个 JavaScript 数组之外，什么也没做。怎么会？我做了什么？

基于我处理需求时的一个错误以及 Chrome 如何处理这个字符串数组，我需要在列表中添加两倍的项目。

这就是变化的样子:

***免责声明:*** *虽然在这个伪代码例子中断章取义没有意义，但是项目的需求需要第二个* `*concat*` *版本。*

我将每个基准测试运行 6 次。平均 12300 毫秒，这是迄今为止最无聊的基准测试。谈慢！

但是，让我们来看看解决这个问题的一些方法。我知道`concat`函数每次运行时都会创建一个新数组。这肯定会成为问题的一部分，所以我利用了`concat`需要两个参数的事实:

这个减少`concat`语句数量的微妙变化让我们减少了一半:6600 毫秒。您可能已经猜到了这个数字，因为将创建的数组数量减少一半会将总处理时间减少一半。

我们实际上可以进一步降低！我们可以简单地给它一个数组，而不是让`concat`检查我们传递的每个参数。如果您不知道，`concat`实际上会展平它给出的任何数组:

通过这一简单的更改，我们现在的时间减少到了可以忍受的 2600 毫秒。为什么？我只能猜测这是因为我们提前创建了一个数组，而`concat`不需要自己处理`arguments`。这是一种使用参数分布的老方法，只在老的`function`定义中有效，在粗箭头函数中无效。

如果您有兴趣，可以在这篇 MDN 文章中找到更多信息:

[](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments) [## 参数对象

### arguments 是一个类似数组的对象，可以在函数内部访问，它包含传递给…

developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments) 

# 加速

正如您所看到的，虽然这段代码看起来很现代，很实用，但是由于 reducer 的使用方式，它仍然很糟糕。RxJS 传感器来救援！

## 为什么要用 RxJS？

和上一篇文章一样，我们将再次使用 RxJS。您可能知道，我熟悉 RxJS，并且已经围绕`subscribe`编写了方法，所以我可以像 JavaScripts 的`Array`方法一样同步使用它。

虽然我可以为这个项目使用现成的转换器库，但它是一个大量使用 RxJS 的 Chrome 插件。引入另一个库增加了项目的复杂性，因为我已经签约做这项工作，我想减少另一个人必须学习维护项目的工具数量。引入 RxJS 本来就不确定，但是由于这个特定插件的异步特性，它已经降低了很多复杂性。

## 我们去了格子呢！

回到我们的传感器版本，让我们再试一次:

2300ms？真的吗？考虑到我们在上一篇文章中的数字，这不是我所期望的速度提升。

这就是这个测试失败的地方。换能器是一种完全不同的思考数组处理的方式。在引擎盖下，传感器是一个`reduce`循环中的一组包装函数。这意味着，拥有 2 个`reduce`函数可能会使速度变慢很多；其实大 O(n)。

利用`mergeMap`，我们最终得到了一个更合理的 70.6 毫秒。我们已经完全摆脱了阵列式的`reduce`，改用更快的`mergeMap`换能器。现在，我们处理所有的值，只在最后把它们放入一个数组。这是加速的关键。我们都减少了创建数组的次数(减少到一次),并尽可能地延迟它(直到最后)。

这是你所期待的，对吗？但愿不会！我认为我们还可以进一步降低。我们必须能够让它更接近我们最初的 8.5 毫秒；否则，我认为这是一个失败。

现在，由于最初的传感器测试是 22.1 毫秒，那么我们只能合理地假设这是我们能做的最好的。因为我们要复制数组大小，所以 40 毫秒是我们的目标。我们还有很长的路要走。

复制我们的数组逻辑和几个`mergeMap`操作符，这是我想到的:

我想，因为我们只创建了一个数组，这将是非常快，但事实并非如此。平均 51.5 毫秒，显然没有达到我们的数字。

另外，注意我是如何复制`filter`逻辑的？那是暂时的。在实际情况下，可以将这些操作符组合起来，创建一个新的操作符，在两种情况下都可以使用。这样你就不会有将来的维护问题。正如你所知道的，我确实在处理两个列表之前尝试过处理过滤器逻辑，但是结果比较慢，因为我必须创建一个中间数组。

现在我带你去看我在那个插件中使用的解决方案——真正的解决方案。这很复杂也很难看，但是在插件中，我实际上能够回到原来的处理速度，这实际上比只做一个`map`要快。

这与之前的版本非常相似，除了我们使用了`forkJoin`而不是`mergeMap`。不同之处在于它返回两个结果的数组。我们实际上是在创建两个较小的数组，然后将它们合并成一个较大的数组，但是在前面的例子中，我们是从一堆已处理的项目中创建一个大数组。

在这一点上，我不能告诉你为什么这个版本更快，因为这超出了我对 RxJS 的了解。我本以为之前的版本会更快，但显然不是。

尽管如此，我们还是获得了 41.2 毫秒，与我们的估计相符。但在我看来还不够好。我想我们可以把这个时间缩短到至少 17 毫秒。

# 突然意识到

这种双重处理的方法让我想到，为什么不在`map`身上做同样的事情呢？事实上，它可能读起来更快更容易！

看起来是这样的:

是的，它在我们估计的 19.8 毫秒的范围内。这是我一直在寻找的解决方案，但我决定走一条不同的路线，因为我没有意识到它可以有多简单。说真的，如果你看到一件事花了 12 秒以上，你的第一反应会是什么？

# 结论

正如您所看到的，即使在我们优化的`reduce`示例中，转换器也可以带来一些主要的速度优势，但是除非您有大量的项目或大量的管道操作符，否则它们在 JavaScript 的本地`Array`方法上所能实现的将会受到限制。

但这并不是说所有的传感器库都是这样的。我使用的是 RxJS，它被设计用来处理随时间变化的值。它并没有像这样针对数组处理进行优化，只是与我以前的文章联系在一起，这些文章也以有趣的方式使用 RxJS。如果我使用真正的传感器库，我想知道这段代码会快多少。

老实说，我对这篇文章的结果感到惊讶。不管出于什么原因，Chrome 插件的处理速度慢了很多，所以 transducer 版本实际上对处理速度有很大的影响，但不管出于什么原因，Node.js v10 已经进行了一些优化，这大大加快了原生`Array`方法的速度。或者，他们减缓了传感器的方法😡。我永远不会知道！好消息是，传感器版本最终只慢了大约 10-20 毫秒。我敢打赌，改变这些结果不需要太多的东西。

## 转折点

好吧，我很好奇，又做了一些基准测试。当将最快的阵列和换能器方法与 500，000 个项目的列表进行比较时。传感器方法快了 100 毫秒。终于！一些符合我期望的东西。

有了这个特定的数据集，我能够找到临界点:大约 250，000 个项目。除此之外，差距还会大幅扩大。现在我很好奇处理器速度和可用内存如何影响这些基准测试。它会支持传感器还是原生阵列方法？

# 更多阅读

如果你对更多与 RxJS 相关的话题感兴趣，你应该看看我的其他文章:

*   [加速 JavaScript 数组处理](/speed-up-javascript-array-processing-8d601c57bb0d)
*   [现在是管道操作员！](/an-emoji-lovers-guide-to-functional-programming-part-4-735c17ca4113)
*   [处理缓存和 AJAX 竞争条件](/handling-cache-and-ajax-race-conditions-4cb152db8764)
*   [RxJS 中的无损背压](/lossless-backpressure-in-rxjs-b6de30a1b6d4)
*   [Redux-Observable 可以解决你的状态问题](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7)