# 带有表情符号的函数式编程基础

> 原文：<https://itnext.io/an-emoji-lovers-guide-to-functional-programming-part-1-241d8d4c9223?source=collection_archive---------0----------------------->

![](img/340a0bbf86d5ec4926929328322c8f60.png)

## 表情符号爱好者的函数式编程指南

## 学习数组映射和闭包的函数基础

*用表情符号和 JavaScript 学习函数式编程。代码示例应该足够简单，不需要任何先验知识就可以理解，但是我可以想象它看起来有点奇怪。还有，JavaScript 实际上不允许表情符号作为 JavaScript 变量名。出于这个原因，这些代码示例不会在没有修改的情况下运行。*

*   [*用食物表情符号制作便便。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-1-241d8d4c9223)
*   [*把暴风云变成晴朗的云。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-2-800b438c7ce3)
*   [*用减速器造独角兽！*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-3-ef78e3156e)
*   [*管道操作员现在！*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-4-735c17ca4113)
*   [*为食肉动物过滤肉类。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-5-a6bc3324a839)
*   [*使用递归与归约。*](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-6-5c1d441d36af)

函数式编程可能很难理解。这里有一个快速使用食物表情符号的方法:

```
💩 = [🍔, 🍟, 🍪].reduce(eat)
```

漂亮吧？如果你把汉堡、薯条和饼干放在减肥器里，你最终会得到便便！

如果你不明白这一点，我们可以从一些简单的基本数学开始:

```
😂 = 😢 + 😀😶 = 🤐 - 😐
```

我们再举一个函数的例子。我们有一排云。我们将处理这个数组，然后返回另一个数组。

```
[🌩️, ⛈️] = [☁️, 🌧️].map(add(⚡))
```

最好的函数式编程！我们拿了一些云表情符号，给每一个都加了闪电，用闪电做了云！就像魔法一样！

在我们进入疯狂小镇之前，让我们看一个使用不同范例的例子:

```
addLighting = item => item + ⚡
🌩️ = addLightning(☁️)addSun = item => item + ☀️
⛅ = addSun(☁️)
```

这段代码虽然有效，但并不枯燥(不要重复)。对于我们想要做的每个`add`操作，我们必须手工创建一个全新的函数。相反，我们可以尝试一个更好的解决方案:

```
add = (item, additive) => item + additive🌩️ = add(☁️, ⚡)⛅ = add(☁️, ☀️)
```

太棒了。我们已经创建了一个接受多个参数并返回一个值的通用函数。但是我们还有一个问题！如果我们想对数组中的每一项都调用`add`，就像我们之前对`map`所做的那样，该怎么办？强制性地这样做，看起来会像这样:

```
clouds = [☁️, 🌧️]
stormyClouds = []add = (item, additive) => item + additivefor (let i = 0, l = clouds.length; i < l; i++) {
    stormyClouds
    .push(
        add(clouds[i], ⚡)
    )
}
```

看起来足够简单，但是`for`循环确实很乱。我们不仅在循环的每一轮对`i`进行了变异，而且还将值推入了循环之外的`stormyClouds`数组。

JavaScript 也有`for-in`循环，为你跟踪`i`；所以，你的`for`循环可以干净很多。

```
for (let i in clouds) {
    stormyClouds
    .push(
        add(clouds[i], ⚡)
    )
}
```

但是，当我们真正关心的是该指数的值时，为什么要传递这个指数呢？foreach 循环，在 JavaScript 中称为`for-of`，让我们更接近函数式方法:

```
for (let cloud of clouds) {
    stormyClouds
    .push(
        add(cloud, ⚡)
    )
}
```

既然我们已经在这里了，我们可以通过用`forEach`替换`for-of`来快速清理这个问题:

```
clouds
.forEach(cloud => {
    stormyClouds
    .push(
        add(cloud, ⚡)
    )
})
```

略好，但不满意。由于`forEach`接受一个函数，我们可以在其他地方创建该函数，然后像这样传递它:

```
addLightning = cloud => {
    stormyClouds
    .push(
        add(cloud, ⚡)
    )
}clouds.forEach(addLightning)
```

虽然`addLighting`函数使单元测试变得简单，但我们还是回到了自己身上。现在我们回到创建一次性函数。如果你想写一个`addSun`，你将会复制粘贴很多相同的代码。我们的代码仍然不干燥，但是我们一般地写它。这是怎么回事？

这就是我们需要一个了结的地方。我们与`map`一起使用的原始`add`函数实际上是这样的:

```
add = additive => item => item + additive
```

这个`add`函数允许我们生成函数。它接受一个`additive`，然后返回另一个接受`item`的函数。关键是第一次调用`add`实际上什么都不做；它只是存储`additive`，所以当内部函数被调用时，它可以访问`additive`，即使它从未被传入！

现在我们可以即时生成`addLighting`和`addSun`:

```
addLightning = add(⚡)
🌩️ = addLightning(☁️)addSun = add(☀️)
⛅ = addSun(☁️)
```

这是另一种看待它的方式:

```
🌩️ = add(⚡)(☁️)⛅ = add(☀️)(☁️)
```

很奇怪吧？为什么我们要返回一个函数，然后调用那个函数，而我们只需要两个参数就可以了呢？是因为复制粘贴的问题。使用闭包，我们可以删除大量样板文件，创建返回其他通用函数的通用函数。

使用过程方法，我们通常会这样写:

```
clouds = [☁️, 🌧️]
stormyClouds = []
sunnyClouds = []add = (item, additive)=> item + additiveaddLightning = cloud => {
    stormyClouds
    .push(
        add(cloud, ⚡)
    )
}addSun = cloud => {
    sunnyClouds
    .push(
        add(cloud, ☀️)
    )
}for(let i = 0, l = clouds.length; i < l; i++) {
    addLightning(clouds[i])
}for(let i = 0, l = clouds.length; i < l; i++) {
    addSun(clouds[i])
}[🌩️, ⛈️] = stormyClouds
[⛅, 🌦️] = sunnyClouds
```

但是使用函数式编程，我们可以编写非常简洁的代码来做完全相同的事情:

```
add = additive => item => item + additive[🌩️, ⛈️] = [☁️, 🌧️].map(add(⚡))
[⛅, 🌦️] = [☁️, 🌧️].map(add(☀️))
```

超级超级强大！相同的功能只需要 3 行代码！为了更深入一点，我们可以编写一个`subtract`函数:

```
subtract = subtractor => item => item - subtractor
```

现在我们可以反过来做同样的操作:

```
[☁️, 🌧️] = [🌩️, ⛈️️].map(subtract(⚡))
[☁️, 🌧️] = [⛅, 🌦️].map(subtract(☀️))
```

俏皮！让我们更进一步。我们实际上可以`map`数组，然后通过链接这些函数再次`map`它:

```
[⛅, 🌦️] = (
    [🌩️, ⛈️️]
    .map(subtract(⚡))
    .map(add(☀️))
)
```

曾经风雨交加的现在阳光明媚。减去闪电，加上太阳。每次调用`map`时，它都会返回一个新数组。你第一次运行它时，我们得到的是常规的无闪电云。第二次运行时，它会添加 sun。超级简单，超级强大。

**感受力量！**

## [点击此处进入第 2 部分！](https://medium.com/@Sawtaytoes/an-emoji-lovers-guide-to-functional-programming-part-2-800b438c7ce3)

# 更多阅读

如果您对与函数式编程相关的更多主题感兴趣，您应该看看我的其他文章:

*   [安全重构旧代码:第 1 部分](/how-to-safely-refactor-old-code-part-1-a1a853263fec)
*   [使用 Redux 的秘密:createNamespaceReducer](https://medium.com/@Sawtaytoes/the-secret-to-using-redux-createnamespacereducer-d3fed2ccca4a)
*   [在 React 组件中使用 Redux 还原剂](https://medium.com/@Sawtaytoes/using-redux-reducers-in-react-components-4e92985dd9cb)
*   [Redux-Observable 可以解决你的状态问题](https://medium.com/@Sawtaytoes/redux-observable-can-solve-your-state-problems-15b23a9649d7)
*   [RxJS 和可观察的 Flic 按钮](https://medium.com/flicblog/flic-buttons-and-the-observable-customization-using-rxjs-2214bc53d407)

*我感谢* ***贾斯汀·苏拉德*** *在工作中鼓舞人心的懈怠状态:*

```
💩 = [🍔, 🍟, 🍪].reduce(eat)
```