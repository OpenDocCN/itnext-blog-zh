# 管道操作员现在！

> 原文：<https://itnext.io/an-emoji-lovers-guide-to-functional-programming-part-4-735c17ca4113?source=collection_archive---------0----------------------->

![](img/340a0bbf86d5ec4926929328322c8f60.png)

## 表情符号爱好者的函数式编程指南

## 使用正式的 ECMAScript 管道操作符。

*用表情符号和 JavaScript 学习函数式编程。代码示例应该足够简单，不需要任何先验知识就可以理解，但是我可以想象它看起来有点奇怪。还有，JavaScript 实际上不允许表情符号作为 JavaScript 变量名。出于这个原因，这些代码示例不会在没有修改的情况下运行。*

*   [*用食物表情符号制作便便。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-1-241d8d4c9223)
*   [*把暴风云变成晴朗的云。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-2-800b438c7ce3)
*   [*用减速器造独角兽！*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-3-ef78e3156e)
*   [*现在的管道操作员是！*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-4-735c17ca4113)
*   [*为食肉动物过滤肉类。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-5-a6bc3324a839)
*   [*使用递归与归约。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-6-5c1d441d36af)

我们一直在使用的`pipe`函数实际上是 pipeline 的简称。当你读到这篇文章的时候，它可能已经发布了，我们目前正处于 JavaScript 中[管道操作符的第一阶段](https://github.com/tc39/proposal-pipeline-operator)。

让我们回到 unicorn 的例子，如果不遍历一个数组，链接我们的`add`函数会有一个大问题。

```
add = additive => item => item + additive🦄 = (
    add(🎶)(
        add(✨)(
            add(🌈)(
                add(🤘)(🐴)
            )
        )
    )
)
```

有时候我们不想遍历数组。也许我们想要一个`subtract`和一个`add`混合在一起，当循环一个函数时会变得稍微复杂一点。当然，我们可以添加一个三元组，并拆分我们在`map`上调用的函数，但这不会非常干净地解决所有情况。

这是我们之前用自己的`pipe`函数得到的解决方案:

```
🦄 = pipe(
    🐴,
    add(🤘),
    add(🌈),
    add(✨),
    add(🎶)
)
```

使用本地管道操作符，我们可以进一步改进:

```
🦄 = 🐴 |> add(🤘) |> add(🌈) |> add(✨) |> add(🎶)🦄 = (
    🐴
    |> add(🤘)
    |> add(🌈)
    |> add(✨)
    |> add(🎶)
)
```

管道操作符获取一个值，然后将其传递给链中的每个函数。看看这段代码，很容易推理并以正确的顺序完成我们需要的任务。

回到我们最初的管道，它返回一个函数；管道运营商没有。这是否意味着管道运营商势在必行？不。但这确实意味着如果我们想要像以前一样的功能管道，我们需要包装它:

```
rockBand = item => (
    item
    |> add(🤘)
    |> add(🎶)
)luckyCharms = item => (
    item
    |> add(🌈)
    |> add(✨)
)kidsTheseDays = item => rockBand(luckyCharms(item))🦄 = kidsTheseDays(🐴)
```

我们可以用不同的方式写`kidsTheseDays`,实现同样的事情:

```
kidsTheseDays = item => (
    item 
    |> rockBand
    |> luckyCharms
)🦄 = kidsTheseDays(🐴)
```

注意我们如何使用功能管道来连接其他功能管道。这将导致一些奇妙的代码重用！一个额外的好处是，只有在我们传递了`item`之后，处理才会发生。

因为所有管道操作符(`|>`)都可以是我们常规的旧`pipe`函数的独立参数，所以让我们用 JavaScript 的原生`pipe`实用程序`Promise`制作一个独角兽:

```
rockBand = item => (
    Promise.resolve(item)
    .then(add(🤘))
    .then(add(🎶))
)luckyCharms = item => (
    Promise.resolve(item)
    .then(add(🌈))
    .then(add(✨))
)kidsTheseDays = item => (
    Promise.resolve(item)
    .then(rockBand)
    .then(luckyCharms)
)promiseICanHasUnicorn = kidsTheseDays(🐴)promiseICanHasUnicorn
.then(console.log)
// 🦄
```

使用`Promise.resolve`，我们能够做与管道操作符完全相同的事情，并且我们获得了内置的错误处理的额外好处。

如果你还没有使用过承诺，你可能会意识到它们是异步发生的；因此，您实际上对它们的执行没有多少控制权。在我们的例子中，我们没有对 unicorn 做任何事情，但是在普通的代码库中，您通常会一直进行处理，直到您触发了副作用，比如写入数据库或呈现到 DOM。

虽然 JavaScript 的本机`Promise`允许我们像使用管道操作符一样使用管道，但它固有的异步性可能不符合您的特定限制。在这种情况下，你可以像这样使用`await`:

```
🦄 = await kidsTheseDays(🐴)
```

当直接处理承诺时，您不需要指定`async`关键字，因为它只是将您的返回值包装在承诺中的语法糖。这样，我们就有了一个不一定是异步的原生 JavaScript `pipe`。

与`async-await`有取舍写法。`Promise`是处理副作用的功能性方法。使用`async-await`将你的功能性的基于承诺的代码转变为一步一步的命令式方法。这里需要一个`await`的例子看起来很棒，但是当你围绕其他函数创建函数时，`async-await`就变成了一种疾病。

每个包含`await`的函数都需要定义为`async`。任何调用链上的`async`函数的函数在被调用之前也需要有`await`，然后也需要被包装在`async`中。这意味着深入代码库的一个`async-await`会导致链上所有更高的功能需要相同的处理。因此，当需要异步操作时，最好使用基于`Promise`的管道，而不是`async-await`。

**感受未来！**

## [点击此处进入第 5 部分！](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-5-a6bc3324a839)

# 更多阅读

如果您对与函数式编程相关的更多主题感兴趣，您应该看看我的其他文章:

*   [安全重构旧代码:第 1 部分](/how-to-safely-refactor-old-code-part-1-a1a853263fec)
*   [使用 Redux 的秘密:createNamespaceReducer](https://medium.com/@Sawtaytoes/the-secret-to-using-redux-createnamespacereducer-d3fed2ccca4a)
*   [在 React 组件中使用 Redux 还原剂](https://medium.com/@Sawtaytoes/using-redux-reducers-in-react-components-4e92985dd9cb)
*   [Redux-Observable 可以解决你的状态问题](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7)
*   [RxJS 和可观察的 Flic 按钮](https://medium.com/flicblog/flic-buttons-and-the-observable-customization-using-rxjs-2214bc53d407)