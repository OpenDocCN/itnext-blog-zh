# 用减速器建造独角兽！

> 原文：<https://itnext.io/an-emoji-lovers-guide-to-functional-programming-part-3-ef78e3156e?source=collection_archive---------1----------------------->

![](img/340a0bbf86d5ec4926929328322c8f60.png)

## 表情符号爱好者的函数式编程指南

## 深入探讨数组缩减、函数组合和 curry。

*用表情符号和 JavaScript 学习函数式编程。代码示例应该足够简单，不需要任何先验知识就可以理解，但是我可以想象它看起来有点奇怪。还有，JavaScript 实际上不允许表情符号作为 JavaScript 变量名。出于这个原因，这些代码示例不会在没有修改的情况下运行。*

*   [*用食物表情符号制作便便。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-1-241d8d4c9223)
*   [*把暴风云变成晴朗的云。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-2-800b438c7ce3)
*   [*用减速器造独角兽！*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-3-ef78e3156e)
*   [*现在是管道操作员！*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-4-735c17ca4113)
*   [*为食肉动物过滤肉类。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-5-a6bc3324a839)
*   [*用归约器递归。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-6-5c1d441d36af)

让我们从用传统方法制作独角兽开始:

```
unicornParts = [🐴, 🤘, 🌈, ✨, 🎶]
unicorn = 🐴add = item => item + unicornunicornParts.forEach(add)🦄 = unicorn
```

这类似于我们的数组`reduce`在幕后做的事情。注意，在这个`forEach`例子中，当在循环的每次迭代中将外部`unicorn`变量设置为一个新值时会产生突变。为了消除这种变异，我们可以使用我们的函数`add`而不用任何循环:

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

乱七八糟的，而且你还得逆向编写函数。如果我们用管子呢？：

```
pipeReducer = (item, change) => change(item)
pipe = (...functions) => functions.reduce(pipeReducer)🦄 = pipe(add(🤘), add(🌈), add(🎶), add(✨))(🐴)
```

好些了，但还没到那一步。使用`map`我们可以进一步改善这一点:

```
add = additive => item => item + additiveunicornParts = [🤘, 🌈, ✨, 🎶].map(add)🦄 = pipe(...unicornParts)(🐴)
```

我们已经大大降低了复杂性，但似乎仍有一些问题。因为我们已经在数组上映射了`add`，我们可以尝试直接使用 ol' `pipeReducer`而不是调用`pipe`。

```
add = additive => item => item + additivepipeReducer = (item, change) => change(item)🦄 = (
    [🤘, 🌈, ✨, 🎶]
    .map(add)
    .reduce(pipeReducer, 🐴)
)
```

这就更难解释了。马离独角兽零件太远了，而且是主料！我们至少可以让它以一种有意义的方式排序:

```
add = additive => item => item + additivepipeReducer = (item, change) => change(item)
initialValue = ???🦄 = (
    [🐴, 🤘, 🌈, ✨, 🎶]
    .map(add)
    .reduce(pipeReducer, initialValue)
)
```

由于 JavaScript 实际上并不支持表情符号，而且我们也没有表情符号数据类型，所以也没有可以使用的起始值，比如:`0`、`''`、`{}`、`[]`、`new Map()`等等。因此，虽然这个例子在视觉上更有意义，但我们不能使用它。相反，我们必须尝试一些更复杂的东西:

```
pipeReducer = (item, change) => change(item)🦄 = (
    [🐴]
    .concat([🤘, 🌈, ✨, 🎶].map(add))
    .reduce(pipeReducer)
)
```

这个方法绝对有代码味。很明显，我们看到了减速器的力量，但肯定有问题。因为`add`包不住马，所以我们在设计中发现了一个瑕疵。相反，我们想使用`add`创建一个缩减器:

```
add = additive => item => item + additivesum = (combined, item) => add(combined)(item)🦄 = [🐴, 🤘, 🌈, ✨, 🎶].reduce(sum)
```

感谢我们对通用`add`函数的重用，我们的代码看起来非常干净。我打赌我们还能做得更好！我们最大的问题是，无论何时我们想要这种功能，都必须从头开始创建像`sum`这样的函数。我们真正需要的是一种像`add`这样的函数生成器，并创建一种通用的方法来生成`sum`:

```
generateReducer = func => (combined, item) => func(combined)(item)sum = generateReducer(add)
```

虽然这看起来没问题，但它非常有限，因为它只能接受两个值。我们真正想要构建的被称为`uncurry`。与之相反的是，`curry`，是一个基于参数列表创建函数生成器的函数。JavaScript 有一个名为`bind`的内置 currying 函数，但是它与我们需要的有点不同，因为我们不关心`this`上下文。

既然需要`uncurry`，我们先写一个简单的`curry`:

```
curry = func => first => second => func(first, second)
```

这个版本只支持最多有两个参数的函数，在传递给我们的函数之后，它只创建一个闭包。它可以把我们原来的不闭合`add`:

```
add = (additive, item) => item + additive
```

并将其转换成我们一直使用的闭包版本:

```
add = additive => item => item + additive
```

但是我们真的希望咖喱有更多的功能。取而代之，我们可以编写我们的`curry`来不断返回规范的函数生成器，直到所有的参数都被赋值(也称为`autocurry`:

这个版本首先包装了我们原来的`add`函数，返回另一个函数。当用任意数量的参数调用这个函数时，我们检查传入了多少个参数。我们要么调用带有所有参数的原始函数，要么返回另一个跟踪所有传入参数的 curried 函数。这意味着我们可以同时使用闭包版本和非闭包版本:

```
add = (additive, item) => item + additivecurriedAdd = curry(add)💩 = curriedAdd(👱, 🍫)💩 = curriedAdd(👱)(🍫)
```

您可能已经意识到，如果我们只是从头开始，就没有理由从闭包中创建函数生成器；尽管，这并不完全正确。在很多情况下，您会希望编写闭包，而不是依赖于`curry`:

*   如果你没有可用的`curry`，你将不得不像以前一样自己编写或创建函数生成器。
*   它创造了许多间接性和魔力，而这对于包括你自己在内的其他程序员来说可能并不明显。
*   总是用`curry`包装你的函数是很麻烦的，相反，编写闭包可能会更容易。
*   如果需要微优化，currying 会在代码周围添加更多的函数包装器和逻辑，最终在数万次调用后会明显变慢。
*   像`pipe`这样接受未知数量参数的函数将无法用于我们的`autocurry`示例，因为它需要已知数量的参数。使用`...args`时，`func.length`将始终返回`0`。

我们学习`curry`的主要原因是为了学习`uncurry`，所以让我们深入了解一下如何扩展我们简单的`sum`方法来创建一个函数，该函数采用函数生成器并返回一个多参数函数:

`uncurry`比`curry`复杂很多。它从用另一个函数包装我们的`add`开始。如果您调用我们的 wrapped `add`，它将接受我们传递的任何参数，直到超过参数的数量。如果我们的`add`函数返回一个函数，那么我们将把剩下的参数传递给那个函数，并重复这个过程，直到我们用完了传递的所有参数:

```
add = additive => item => item + additive💩 = add(👱)(🍓) uncurriedAdd = uncurry(add)💩 = uncurriedAdd(👱, 🍓)
```

有了工具箱中的`curry`和`uncurry`，我们现在可以使用`uncurry`和`add`来生成`sum`:

```
sum = uncurry(add)🦄 = [🐴, 🤘, 🌈, ✨, 🎶].reduce(sum)
```

就这么简单，对吗？有点吧。最初的问题是不能用`add`来`map`所有的事情，这是因为我们的`add`不是很通用。我们需要考虑这样一个事实，即我们可能没有将`item`传递到我们返回的函数中:

```
add = additive => (item = ???) => (
    item + additive
)
```

虽然我们遇到了同样的问题，即没有一个假表情数据类型的默认值，但我们可以使用一个函数三元组来代替:

```
add = additive => item => (
    item
    ? item + additive
    : additive
)
```

现在我们可以做一个 map-reduce 来创建`add`包装器并使用我们的`pipeReducer`。

```
add = additive => item => (
    item
    ? item + additive
    : additive
)pipeReducer = (item, change) => change(item)🦄 = (
    [🐴, 🤘, 🌈, ✨, 🎶]
    .map(add)
    .reduce(pipeReducer, null)
)
```

当然，这不像我们之前的`sum`函数那么简洁，但是如果你手头没有`uncurry`，map-reduce 可能就是你问题的解决方案。

**感受号角！**

## [点击此处进入第 4 部分！](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-4-735c17ca4113)

# 更多阅读

如果您对与函数式编程相关的更多主题感兴趣，您应该看看我的其他文章:

*   [安全重构旧代码:第 1 部分](/how-to-safely-refactor-old-code-part-1-a1a853263fec)
*   [使用 Redux 的秘密:createNamespaceReducer](https://medium.com/@Sawtaytoes/the-secret-to-using-redux-createnamespacereducer-d3fed2ccca4a)
*   [在 React 组件中使用 Redux 还原剂](https://medium.com/@Sawtaytoes/using-redux-reducers-in-react-components-4e92985dd9cb)
*   [Redux-Observable 可以解决你的状态问题](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7)
*   [RxJS 和可观察的 Flic 按钮](https://medium.com/flicblog/flic-buttons-and-the-observable-customization-using-rxjs-2214bc53d407)