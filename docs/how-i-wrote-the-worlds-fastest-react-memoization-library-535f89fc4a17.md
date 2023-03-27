# 我如何编写世界上最快的反应记忆库

> 原文：<https://itnext.io/how-i-wrote-the-worlds-fastest-react-memoization-library-535f89fc4a17?source=collection_archive---------1----------------------->

![](img/23c5c0b42389c9865c2de2d6c46592ca.png)

[*点击这里在 LinkedIn 上分享这篇文章*](https://www.linkedin.com/cws/share?url=https%3A%2F%2Fitnext.io%2Fhow-i-wrote-the-worlds-fastest-react-memoization-library-535f89fc4a17)

事实上，我曾经尝试过写最慢的一个，而“t [最快的 JavaScript 记忆库](https://community.risingstack.com/the-worlds-fastest-javascript-memoization-library/)是一年前写的。甚至没有一个——有这么多的库，一个比一个快，很难把它们都记住。

> 为什么要多写一个？因为你不能使用或处理现有的。

## 记忆。

这是减少或完全跳过不必要计算的常见模式。工作非常简单——

> 你有一个函数。你用一些参数调用 than 函数。下一次你做同样的事情——只是重复使用已知的结果。

所有的图书馆都这么做。唯一的区别是它们如何处理函数 arity(参数的数量)，它们可以存储多少结果，它们的速度有多快。

默认情况下，lodash.memoize 只“看到”第一个参数，而 fast-memoize 字符串化所有参数，并使用 JSON 作为缓存键。

速度也很重要。Ramda 比没有记忆的代码快 **100 倍**，lodash 比 ramda 快 **100 倍**，纳米记忆比 lodash 快 **100 倍**。

> 但是你不需要速度。你需要记忆。

上面的比较是针对斐波那契数列计算的。所有这些库都很擅长基于参数来记忆函数结果，打破所有内存限制，因为长缓存大小通常是不受限制的。

关于这个问题的第一个*调用*(对我来说)是由一个名为 [memoize-one](https://github.com/alexreardon/memoize-one) 的库发出的，作者是 [Alex Reardon](https://medium.com/u/ea5e41121e55?source=post_page-----535f89fc4a17--------------------------------) 。主要意图很清楚—**它记忆一个结果**。因为你可能不需要更多。

> 这其实是 React/Redux 世界需要的东西。只有中断更新的能力，没有任何副作用(像内存缓存中的内存泄漏)

> 应该不会很快。应该能用。

但是，你知道，你仍然必须写一个“特殊的”代码来正确地记忆你需要的东西。

```
const mapStateToProps = state => ({
   todos: state.todos.filter(todo => todo.active)
});
```

^:每次状态改变时，这个都会产生一个新的 todos 数组。

```
const filterTodos = memoize(todos => todos.filter(todo => todo.active));const mapStateToProps = state => ({
   todos: filterTodos(state.todos)
});
```

^这个不会。现在它只会对`state.todos`物体的变化做出反应。

```
const filterTodos = memoize(todos => todos.filter(todo => todo.active));
const getTodos = todos => todos.map(todo => todo.text )const mapStateToProps = state => ({
   todos: getTodos(filterTodos(state.todos))
});
```

^这个仍然会对`state.todos`物体的变化做出反应，但是应该只对`todo.text`的变化做出反应。

普通库也有一个共同的“限制”，比如一个[重选](https://github.com/reactjs/reselect) ( **选择器和记忆**库)，它也只能存储一个“最后”结果。并且，如果你有**几个**不同的**连接组件**，都从一个商店选择一些东西，但是具有不同的道具——你将总是禁用你的记忆功能([链接到问题](https://github.com/reactjs/reselect#accessing-react-props-in-selectors)，[链接到变通办法](https://github.com/threadheap/reselect-weakmap-memoize)，以及[另一个变通办法](https://github.com/toomuchdesign/re-reselect))。**有** **可** **只有一个！**他们只会一直“缓存缺失”。

> 编写一个适当记忆的代码——可能是一项艰巨的工作。

你将最终获得重新选择级联和非常具体的记忆技术，这将使你能够为你的项目创建一个“更好的”记忆。

> 不是“更快”，而是“请尽可能多地记忆案例”

## MobX

我一直喜欢 MobX 的一点——懒惰。我可以偷懒，写一段代码，这段代码会被偷懒执行…什么也不做。

你不需要想，“哦，当这个事件被调度时，Redux 将触发所有的 ConnectedComponents，mapStateToProps 所有的东西，并且可能重新绘制应用程序的一半，因为我的一个选择器每次运行都产生一个唯一的值。

> 不是唯一的，而是不可改变的。就{…originalStateProp}

你知道，由于这种**低级优化**你，除了你必须提供什么，**但没有**——Vue 和 Angular 可以更快开箱。我是说 React/Redux 可能很烂。MobX——石头！

## Immer.js

哦。Immer 是一股清新的空气。它使用现代 javascript 的能力来增强 reducers，让您只需编写代码。让您只编写您需要编写的代码，而不考虑如何编写代码(并保持不变性)。

Immer 比" not"-immer 慢得多，但是你不需要这些每秒千万次的减速器，如果没有它你可能会有的。仅仅每秒 100 万就已经不错了。

# 但是记忆化呢？

所以我建立了一个内存化库，它与 MobX 和 Immer.js 共享一些东西。它只是工作，解决你的问题。不知何故。神奇地。隐形的。

正如我在开始所说的——我尝试过构建最慢的记忆库，它是最快的记忆库。

我称之为— **记忆状态**。

[](https://github.com/theKashey/memoize-state) [## Kashey/memoize 国家

### memo ize-state-mapStateToProps 的正确记忆

github.com](https://github.com/theKashey/memoize-state) 

它是**慢速**cos——它使用 javascript `Proxy`来观察内存化的函数正在做什么。它比普通的记忆库多 100 倍的代码。

它是**快速的**cos——当它必须决定它应该返回被记忆的值还是必须刷新它——它可以**只比较被使用的部分参数**,使它成为…

> “请尽可能多地记住案例”

只要它“经常”记忆，它在计算上花费的时间就会更少，工作速度也会更快…

我可能应该贴两个例子。

第一个很简单。它像所有记忆功能一样工作。

但是第二个…它会**忽略第二个元素中的值变化**，因为你没有读取它，甚至没有返回那个元素。你不需要它。

> Memoize-state 不会对参数的变化做出反应，memoized 函数不会“使用”。因此，它将记忆更多的情况。

只需想象更常见的情况— redux 的 mapStateToProps。

没有特别的逻辑。没有选择器。没有“参数级”记忆。而且你不会相信我，**这些函数是相等的**！你可以在任何地方应用内存状态！它会跟踪你提供的参数的用法。

> 记忆状态不是“内部”记忆，不是“功能”记忆，而是**“外部”记忆**。只要用 memoize-state 包装任何东西并让它被记忆。任何纯粹的功能，只要有可能，都应该被记忆！

通过相同的机制，它解决了具有多个组件的*重选问题*——只需将整个 mapStateToProps… **包装两次**。道具没有区别的情况下，每个人一次，每个组件第二次，如果他们的选择器不同。我有一个软件包，可以自动完成。

[](https://github.com/theKashey/beautiful-react-redux) [## 卡希/美丽反应还原

### 美丽-反应-还原-还原🚀，Redux🤘，Redux🔥

github.com](https://github.com/theKashey/beautiful-react-redux) 

我在这里只能说出不好的东西——只要你不能使用像重选级联这样的东西，你就不会使用它们(经验法则)。因此，您也将丢失缓存级联。结果是——有些东西会被计算两次。

> 没有汽车能取代你。你是施展魔法的法师。

## 反应-记忆

Memoize-state 工作起来非常简单，对用户来说是不可见的，因此我在另一个库中使用了它，并考虑到了 Memoize。正如丹·阿布拉莫夫所提议的。

我构建的库不是基于这个规范，只要你的记忆函数是“外部的”就没有必要。

它只是将所有的道具(除了`compute`)传递给`compute`函数，并通过子函数(又名 renderProps)渲染结果。

仅此而已。其他的都藏在引擎盖下。你应该相信记忆状态。检查 repo——它包含一个指向 codesandox 示例的链接。

[](https://github.com/theKashey/react-memoize) [## 记忆/反应-记忆

### 反应式内存缓存(又名内存化)是一种非常强大的优化技术

github.com](https://github.com/theKashey/react-memoize) 

## 记忆状态

编写这个库不是一项简单的任务。我写了它，花了两天时间，测试它，在 twitter 上发布，发现这个库不工作，我的意思是完全不工作，并花了两个多星期在研发上

我修好了它。接下来我写了这篇文章。发现我犯的错误越来越少。修好它们。发现没几个“客户”，仔细检查这次真的“搞定”了。

它实际上是如何工作的——它只是用代理(来自 [proxyequal](https://github.com/theKashey/proxyequal) 库)包装所有给定的参数，而**监视**使用情况。

在 memoized 函数执行后，memoize-state 将从提供的参数中知道所有使用的“子键”。子键，不可能检测到有人使用(或未使用)了键(参数)本身。

然后，当记忆函数被第二次调用时——它将不得不比较的不是对象本身，而是整体的一小部分。这是我犯的第一个错误。

如果你使用了`a.b.c.d`，你必须“重视”——只比较`.d` **让所有其他路径都“灵活”**，否则这种记忆将不起作用。我花了一周时间才明白为什么我写的简单比较函数在现实中不起作用。

错过的点——我们还必须`deproxyfy`结果，在被记忆的函数内确定跟踪代码的范围，不要让它泄露出去……并跟踪函数返回的键。

在这个例子中，memoize-state 应该对`state.a.b.c`作出反应，只要这是所使用的“最深”键，并且只要它被返回，就对`state.a.b`作出反应。但不是`state.a`——它应该保持“灵活”。

还有代理中的代理(或两次记忆)，这在记忆中应该是非常准确的，以防止误报缓存。

目前所有边缘情况都已解决并测试，**记忆状态稳定**。

## 速度

Memoize-state 将基准作为 CI 的一部分。很难理解这个库的性能——它总是在记忆功能的“成本”和记忆糖的“成本”之间取得平衡。

但是让我们试试。

1.  “标准”记忆。3 个整数参数的函数。没有变化。

> memoize-one x 6703353 操作/秒
> lodash.memoize x 3095017 操作/秒
> fast-memoize x 1013601 操作/秒
> memoize-state x 4007493 操作/秒

不坏。比 lodash 更好。Fast-memoize 有点糟糕，因为将 3 个旧的普通类型(int)转换成 JSON 来存储这些东西——成本很高。

*这是一个悬而未决的问题，cos lodash 做同样的转换。*

2.“标准”记忆。3 个整数参数的函数。所有变化。

> memoize-one x 21573 操作/秒
> lodash.memoize x 35272 操作/秒
> fast-memoize x 19195 操作/秒
> memoize-state x 19436 操作/秒

不是最好的。但是这是边缘情况，当你不需要任何记忆的时候。

3.“国家”案。随机改变类似状态的物体。

> memo ize-one x 9921
> lodash . memo ize x 93547
> fast-memo ize x 90966
> memo ize-state x 1301700

~ 13 万 vs ~9000。它只是忽略了大部分的随机状态变化，因为长记忆函数只使用了一个子集。

4.“大州”案。只是提供大的状态。

> memo ize-one x 10523
> lodash . memo ize x 258
> 快速记忆 x 255
> 记忆状态 x 81181

80000 对 300。

lodash memoize 和 fast-meoize 将状态转换为 json。坏主意。memo ize-一个做浅相等，但很多值。记忆-状态执行狙击手比较。

## 缓存命中

原始性能测试不仅包含每秒操作数，还包含“缓存命中”参数。这可能更重要。

> 你的记忆功能可以很慢，只要它比“错过的”记忆和树渲染导致的 react/redux/redraw 快。

正确的重选级联可以有 100%的缓存命中率，但是很难编写正确的级联，调试它，使它保持最新。嗯，只是需要时间。

memoize-one 的“缓存命中”接近理想状态。它将尽可能多地记忆案例。

它比普通的记忆库大 10 倍，比普通的记忆库慢 10 倍，但是，你知道，你的应用程序会同样快 10 倍。没有任何时间花在优化上。

> 这就是目标。没有什么“特殊”的事情需要你去做。

## 浏览器支持

Memoize-state 和所有基于它的库都需要 ES6 代理。IE11 和 React-Native 都不支持，你得多填。不是问题。

[](https://github.com/tvcutsem/harmony-reflect) [## tvcutsem/和声-反射

### 用于 ES6 反射和代理对象的 harmony-reflect - ES5 shim

github.com](https://github.com/tvcutsem/harmony-reflect) 

## 结论

记忆状态不能代替“普通的”记忆函数，对于“普通的旧类型”——字符串和整数作为参数的情况，只要它非常复杂。请随意使用任何其他低级库。

如果你传递一个对象——状态、道具，或者只是把你所有的变量放入对象中并传递那个对象，它就会发光。闪亮亮！

> const easy win = memoizeState(your function)

这种“不可见的记忆化”帮助我构建了两个非常有用的库，并在开发周期和运行时节省了大量时间。).

你能用它建造什么？

[](https://github.com/theKashey/memoize-state) [## Kashey/memoize 国家

### memo ize-state-mapStateToProps 的正确记忆

github.com](https://github.com/theKashey/memoize-state)