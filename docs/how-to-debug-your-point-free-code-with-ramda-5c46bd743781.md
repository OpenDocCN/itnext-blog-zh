# 如何用 Ramda 调试你的无点代码

> 原文：<https://itnext.io/how-to-debug-your-point-free-code-with-ramda-5c46bd743781?source=collection_archive---------2----------------------->

![](img/77d858e3381918ec3971a0d7781bfa0a.png)

大约六个月前，我开始在日常生活中使用 [Ramda](https://ramdajs.com/) ，这是一个用于编写函数式 JavaScript 的库，总的来说，我认为它帮助我编写了更简单、更容易推理、更不容易出错的代码。这有多种原因，特别是不变性是其设计的核心，它通过功能组合鼓励可重用性。然而，无论你是一个多么优秀的程序员，你都不可避免地会遇到错误，当你遇到错误时，你通常希望这种体验尽可能的顺畅。

如果你像我一样，喜欢到处粘贴`console.log`语句来弄清楚发生了什么(我知道，我知道——我应该使用我的调试器)，那么从命令式编程方式到声明式编程方式的范式转变可能会在一开始让你感到困惑，特别是如果你试图做一些[没有要点的事情](http://randycoulman.com/blog/2016/06/21/thinking-in-ramda-pointfree-style/)。今天，我们将学习一个简单的技巧，它允许你窥视你的程序，并观察程序中任何给定步骤的值。

## 把它插在你的烟斗里抽吧

在 Ramda 中，您会经常发现自己通过管道将一个函数的返回值传递给另一个函数，以便对给定的输入应用一系列转换。Ramda 提供了一个方便的`[pipe](https://ramdajs.com/docs/#pipe)`函数来完成这个任务。

```
const add1AndMultiplyBy2 = R.pipe(
  x => x + 1,
  x => x * 2,
);add1AndMultiplyBy2(1); // returns 4
```

让我们假设这两个 lambdas 实际上是可重用的，并在我们的管道外部将它们定义为独立的实用程序。

```
const add1 = x => x + 1;
const multiplyBy2 = x => x * 2;
```

然后我们可以使我们的`add1AndMultiplyBy2`函数完全无点。

```
const add1AndMultiplyBy2 = R.pipe(
  add1,
  multiplyBy2,
);
```

太好了——现在我们有了干净、优雅的代码，它对每一步发生的事情都是透明的。会出什么问题呢？

那么，如果在我们不知道的情况下，我们的`add1AndMultiplyBy2`函数被用字符串`'1'`而不是数字`1`调用，会怎么样呢？

```
add1AndMultiplyBy2('1'); // returns 22
```

没错—这将返回数字`22`。向任何本能地知道这一点的人脱帽致敬。对于其他想知道发生了什么的人，让我们先来看看在没有管道的情况下，如何用普通的 JavaScript 编写我们的函数。

```
function add1AndMultiplyBy2 (num) {
  const plus1 = add1(num);
  const plus1Times2 = multiplyBy2(plus1); return plus1Times2;
}
```

为了弄清这个神秘的`22`的真相，您可能要做的第一件事就是在对`add1`的调用后贴一个`console.log`来观察它的返回值。

```
function add1AndMultiplyBy2 (num) {
  const plus1 = add1(num);
  console.log(plus1); // logs '11'
  const plus1Times2 = multiplyBy2(plus1); return plus1Times2;
}
```

啊哈！狡猾，狡猾的 JavaScript！它将数字`1`与字符串`'1'`连接在一起，产生`'11'`，然后传递给我们的`multiplyBy2`函数，通过类型强制的魔力，产生我们的最终返回值`22`。谜团解开了。

很好——但是当使用 Ramda 时，如何得出这个不太明显的结论呢？如果我们使用漂亮的`pipe`回到更简洁的`add1AndMultiplyBy2`版本，我们会注意到没有地方可以粘贴`console.log`。

## R.tap()来救援！

幸运的是，Ramda 提供了一个名为`[tap](https://ramdajs.com/docs/#tap)`的函数，它用一个提供的值运行一个给定的函数，然后返回值，不变。

可以把它想象成一个`[identity](https://ramdajs.com/docs/#identity)`函数(它除了返回提供给它的值之外什么也不做)，但是具有执行一些副作用的能力。

那么这有什么用呢？现在，我们可以`console.log`管道中任何给定步骤的值，以观察出问题时发生了什么。

```
const add1AndMultiplyBy2 = R.pipe(
  add1,
  R.tap(x => console.log('WTF', x)), // logs 'WTF' and '11'
  multiplyBy2,
);add1AndMultiplyBy2('1'); // returns 22
```

这一切，*没有打破我们的管道*。

这最后一点很重要，因为如果我们在调用`add1`后简单地传递`console.log`作为`pipe`中的下一个函数，我们实际上会改变最终结果，因为`console.log`返回`undefined`。

```
const add1AndMultiplyBy2 = R.pipe(
  add1,
  console.log, // returns undefined
  multiplyBy2, // undefined * 2??
);add1AndMultiplyBy2('1'); // returns NaN, duh
```

这可能会导致更多的混乱，特别是因为在管道中间返回`undefined`经常会抛出一个大错误，这时你可能会忍不住开始拔头发。

## 包装东西

由于使用`tap`将一些东西记录到控制台对于调试来说非常实用，所以让我们继续为这个特殊的用例创建一个包装器。

```
const log = msg => R.tap(x => console.log(msg, x));
```

现在我们可以在管道函数中的任何地方使用我们方便的`log`函数，就像我们使用常规的`console.log`一样，甚至不需要记住`tap`是如何工作的。

```
const add1AndMultiplyBy2 = R.pipe(
  add1,
  log('value after add1: '), // logs "value after add1: 11"
  multiplyBy2,
);add1AndMultiplyBy2('1');
```

就这样结束了！😎

1.  关于 JavaScript 中类型强制的更多信息，请查阅凯尔·辛普森的[类型和语法](https://github.com/getify/You-Dont-Know-JS/blob/master/types%20&%20grammar/README.md#you-dont-know-js-types--grammar)一书。