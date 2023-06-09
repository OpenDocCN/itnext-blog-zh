# 设计固体:反应性

> 原文：<https://itnext.io/designing-solidjs-reactivity-75180a4c74b4?source=collection_archive---------1----------------------->

![](img/d8ff2efabf88b08f4d2740977bfbbec9.png)

IStock

[*SolidJS*](https://github.com/ryansolid/solid) *是一个高性能的 JavaScript UI 库。本系列文章深入探讨了设计该库的技术和决策。使用 Solid 不需要了解这些内容。今天的文章重点是固体的反应系统。*

2019 年前端开发的热点按钮话题。这在很大程度上归功于苗条的哈里斯和富有的哈里斯。他并不是第一个这样说的人，React 自己团队的成员也说过，包括丹·阿布拉莫夫:

> 反应并不完全是被动的

我认为我们不应该对[的反应](https://reactjs.org/)这么苛刻。在这一点上，我想观众中的许多人都在问，“被动反应到底意味着什么？”有学术功能反应式编程(FRP)，如 RxJS，处理数据流。还有像 MobX 或者 [Svelte](https://svelte.dev/) 这样的 UI 库。然后当然还有[反应](https://reactjs.org/)。我读过很多解释，比如老维基百科的回答:

> **反应式编程**是一种[编程范式](https://en.wikipedia.org/wiki/Programming_paradigm)，围绕[数据流](https://en.wikipedia.org/wiki/Dataflow_programming)和变更传播。这意味着应该可以用所使用的编程语言轻松地表达静态或动态数据流，并且底层执行模型将自动通过数据流传播更改。

这是真的，但也有点多。它实际上是一个基于事件的系统。你以“不要打电话给我们，我们会打电话给你”的方式编写代码。所有反应式系统仍然需要一些东西来触发更新事件，然后调用您的代码来处理该更新。这不同于经典系统，因为它的模型不是基于轮询，或者一些连续的代码执行循环(是的，这发生在后台)。你的代码对一些事件做出反应。

解决方案之间的真正区别在于变更的粒度。也就是说，你一共准备了多少零钱？您是否将一个数据点与 DOM 节点上的一个属性绑定在一起，以便在数据更新时设置该属性？还是通过虚拟 DOM 渲染器和 diff/patch 引擎移动整个状态树？这些都是重要的考虑因素，并指导你如何攻击这个主题。但是我想首先确定所有的 UI 系统在很多方面都比你想象的更接近。我将强调不同库所采取的权衡以及在设计固体反应性中所扮演的角色。

# 反应钩

我发现 React Hooks 的那天，我太激动了。记得看过[文章](https://medium.com/@dan_abramov/making-sense-of-react-hooks-fdbde8803889)又重读了一遍。看着丹的视频挂掉每一个例子。我不是不明白那是什么。不，我很好奇他们是如何实现的，但是当我看到第一个例子的时候，我立刻明白了他们的意图。我只是完全不相信。 [React](https://reactjs.org/) 团队不仅能够模仿一个适当的细粒度反应系统的 API，而且他们还通过 API 简单地解决了一个我已经挣扎了多年的问题。最重要的是，他们认为这是[反应](https://reactjs.org/)发展的前进方向！

当我看到 React Hooks 时，我并不觉得看到了什么特别新的东西，我以前没有看到过十几次。KnockoutJS 在 2010 年有一个非常相似的 API，没有人会为了那个 API 而跳过去。具有讽刺意味的是，正是 React 的流行为这些库钉上了棺材，并开始了我创建 Solid 的旅程。2014 年，我讨厌 React、它的类生命周期和它的虚拟 DOM。我已经习惯于通过数据定义来组织我的组件，并将其抽象成可重用的块。我看到了不祥之兆，知道是时候更进一步了。

React 永远不会成为最好的声明性数据库。钩子规则和所有围绕闭包的复杂性都是其技术的结果。事实上它不是细粒度的。有了细粒度的库，你的组件执行一次，只有你的钩子执行多次。与 React 完全相反。您的组件每次都执行，并且您将更改列入白名单。

事实上，在流行的库里， [Vue](https://vuejs.org/) ，是使用钩子模式的明显领先者。它们建立在一个细粒度的反应系统上，该系统提供给一个粗粒度的虚拟 DOM。这是天造地设的一对，不用想也知道。埃文和他的公司不可能避免采取这种方式。当然，社区的反应非常消极，这本身就说明了很多问题。

因此，在这个领域没有明确的赢家的情况下，我决定如何去尝试传递最好的声明性数据(Hooks)库的火炬。

# 命令查询责任分离

我前面提到 React 已经解决了我曾经遇到的一个问题。细粒度反应库中的 getter/setter 总是很难使用。他们需要处理拦截访问和更改。最经典的形式是单个函数，其中不带参数调用 if 是一个 get，带一个参数调用 if 是一个 set。

```
const count = observable(0);// read count
count();// set count
count(10);
```

这总是一个笨拙的语法，因为你永远不清楚你到底在做什么。你在调用函数吗？你在获取一个值吗？公平地说，这在这些类型的库中并不重要，因为执行上下文中的一切都会被跟踪，但是当涉及到重构时，如果不小心的话，可能会产生一些非常糟糕的反应循环。

较新的库使用了显式的`.get`和`.set`方法，但是总是很冗长。然后出现了代理，它消除了所有这些开销，但隐藏了更多的数据。这是一个很好的进步，但是有了嵌套就很难跟上变化了。它到底有没有深度可观测性？在粗粒度的系统中，比如由 Vue 或 React 使用的虚拟 DOMs，这没什么大不了的，但是对于 Solid 来说，它从来都不太适合我。然后我看到:

```
const [count, setCount] = useState(0);
```

这是如此简单，但它解决了许多问题。首先，这些并不是不明确的，因为它们是特定于功能而命名的。一个获取值，另一个设置值。使用起来也不冗长。你想怎么叫就怎么叫。没有`.set`。最后，也是最重要的，读和写是分开的。这是以前没有人做过的事情，也是控制的关键。您可以将该值传递给后代代码，但不能更改它，或者您可以传递 setter，但不能读取它。您可以创建更改管道，其中只暴露第一个 setter 和最后一个 getter，并保持相同的干净模式。当我看到这个的时候，我知道这就是 Solid 的 API。

以前我们有所有这些可变的原子，现在我们有有目的的方法。实际控制数据流的能力是 React 钩子最大的进步。它是命令查询责任分离(CQRS)，也称为 Flux，以更精细的粒度应用于本地状态。简而言之，通过分离读/写，我们可以实施单向数据流。人们对面向对象编程充满了诗意，但这是一种没有类的模式，无法提供更高级别的控制。然而，大多数“反应式”图书馆似乎完全忽略了这一点，他们如此渴望轻松，以至于忘记了我们在 21 世纪初已经尝试过这一点，而且非常糟糕。

Vue 在这里尤其引人注目。他们已经有能力向这个组合 API(Hooks)方向发展很多年了。他们随时都可能暴露他们的反应系统。他们不需要反应来发现钩子来移动。Vue 一直藏在引擎盖下。我猜他们没有，因为他们渴望远离那些 2010 年代早期的图书馆。我不怪他们。现在气氛已经变了，我担心这个教训还没有被吸取。

钩子和细粒度计算之间还有一个区别。获取简单值需要一个代理对象或访问函数。没有办法仅仅将`count`作为一个值传递回去，并且能够跟踪依赖关系。于是我有了一个好主意，如果我们编译呢？

# 编译缺点

[Svelte](https://svelte.dev/) 的存在开启了一场有趣的讨论，关于我们能用 JavaScript 框架设计做什么。简单语法和缩减运行时间的承诺令人难以置信。不幸的是，即使是编译也不能真正帮助我们。它还有很多其他的好处，但是在反应性方面，它不是免费的。让我们来看一个假设的例子。

为什么我在这里看编译？为了避免输入`()`或者`.value`。但是编译器怎么知道我的值是反应性的呢？在基础层面上，让我们让一切都是反应性的。事实上我们可以做得更好。让我们将它限制为在计算的上下文中访问的值。对于局部变量，这可以无缝地工作，但是对于道具或全局存储呢？像定制钩子这样的复合行为呢？

就是这么回事。对于传入的任何数据，我们不知道是否应该将其视为包装的。如果我们不能控制上下文之外的更改传播，那么我们在本地包装它也没有关系。我们可以假设它已经包装好了吗？这可能是一个危险的假设。不，我们只加一个尾随的`$`来表示。嗯……我们又回到使用指示器了。

让我们从不同的角度来看这个问题。我如何表明我正在将值而不是反应容器传递给子组件？我是否在某些情况下包括了`$`而在其他情况下没有？这基本上挫败了本汇编的目的。也许我出去的时候会把所有东西都包起来。

所以结论是我可以局部优化，但我总是在边界上付出最大的代价。如果我更进一步，我意识到我甚至不需要复杂的反应系统，因为我可以使用静态分析来排序我的更新。事实上，为什么还要独立运行它们，因为那样我就需要一个订阅系统了。我可以将更新绑定到本地组件。由于我尽可能避免特殊的指示符语法，并且总是在边界处换行，所以尝试像 CQRS 这样的东西来控制数据流是不可能的，也是不切实际的。

我的朋友们，这就是我们如何到达苗条的 T2 的。这一切都很符合逻辑，但代价很大。为了简单和简洁而完全拥抱编译的动机不仅限制了我们的表达能力，而且强加了非常真实的性能边界。参见[UI 组件的真实成本](https://medium.com/better-programming/the-real-cost-of-ui-components-6d2da4aba205)和 [JavaScript UI 编译器:比较苗条和坚实](https://medium.com/@ryansolid/javascript-ui-compilers-comparing-svelte-and-solid-cbcba2120cea)以获得更深入的观点。尽管如此，在这个领域，如果有几件事情发生变化，性能可伸缩性考虑可能并不总是如此。然而，试图将反应原子表示为单个值总是会有问题，因为一旦我们离开当前范围，就必须区分反应引用和它返回的值。

# 粒度

我们已经讨论了许多流行的方法，但是如果你一直在跟踪的话，没有一种方法最终能比组件更好地处理粒度。Svelte 编译成一个简单的每组件更新程序。React Hooks 为您提供了一个 API，以细粒度方式表示您的更改，但每次仍会重新执行您的函数组件。Vue 从一个细粒度的反应系统开始，只是将整个组件模板包装在一个计算节点中。这些系统没有一个真正符合我的目标。从技术的角度来看，Vue 可能是最接近的，但在我看来，考虑到对婴儿步伐的阻力，在不可否认的明显正确的方向上，以及它在这项技术上已经存在多长时间，我觉得在未来几年内，他们不太可能走得足够远。

那么为什么强调粒度呢？它减少了在运行时计算成本的需要。Svelte 明白降低这一成本的重要性。他们通过局部优化来解决这个问题。但除此之外，在一个更普遍的系统中，这意味着在更具体的层面上将反应效应与数据联系起来。如果改变`state.url`直接更新一个锚标签，你不需要区分整个模板。当然，前期的布线成本可能会很高。但是 web UI 的大部分成本来自 DOM 操作。因此，如果您避免跟踪实际的 DOM 并预编译高效的代码，执行开销甚至比插入元素的巨大开销还难以衡量。在互操作性和调试方面，使用真正的 DOM 节点还有其他一些好处。

现在 Svelte 的方法本身并不是低效的。在单个更新例程上进行简单的数据失效比较也能提高一些最快的库，但是一旦接近边界和更深的嵌套，就会出现新的问题。Svelte 对此有解决方案，但是他们的存储技术与他们的组件状态管理解决方案略有不同。我知道希望 Solid 的基本原语是可传递的。“钩子”才是重要的。这些原语在任何地方都应该工作一致。实际上，组件远没有那么重要。

# 组件之死？

不完全是。我们需要将代码容器化。但是将组件作为唯一的粒度级别有很多限制。它们不再是你组织代码和思考的方式，而是成为你的应用程序的一个功能块。它们不再是一个模块化/封装的故事，而是变成了别的东西。React 已经到了那个地步。你只能在一个问题上投入更多的组件。它促使你发明解决问题的方法，否则这些方法可能不存在。我的意思是，当处理 6 个组件与 2 个组件时，适当的钻井会差多少。

在像 Solid 这样完全细粒度的库中，大部分工作是在组件初始化期间完成的。本质上，主要部分只需要执行一次。所以，当我意识到在初始化之后保留组件的动机微乎其微时，我明白了。所有的钩子都在那个状态上关闭，并且永远不会丢失它们的引用，直到它们的上下文被释放。它们不需要额外的重量。没有需要反应式包裹和展开的边界。就执行代码而言，组件是一种幻觉。

我需要解决的唯一问题是如何确保以一种通用的方式处理 props，这种方式可以推迟执行，直到在正确的上下文中被访问。因此，组件消费者可以传递一个反应值或静态值，它仍然可以工作。幸运的是`props`是对象，所以我可以在这里使用 getter/setter 属性。这比做反应包装要便宜得多。基本上，在不知道所包含的数据是否是反应性的情况下，我们可以将执行推迟到重要的时候。这也是懒惰评估`props.children`的一个简单机制，但是我将等到下一篇关于模板的文章来深入探讨更详细的细节。

这意味着原始独立性的完全容器。基本上一切都是函数。通过编写和调用函数来处理编写钩子、存储或组件。任何函数都可以包含原语，因为它们独立于渲染层次，DOM 更新只是副作用。基本上，代码组织不是库的功能关注点。我知道这有点吓人。有些人认为 React 太自由了。有最佳实践，但没有规则。我希望 Solid 能更进一步地接受它。

# 结论

查看这些其他库并探索反应性的含义对于 Solid 找到它的位置是至关重要的。

React 有助于用明确的有目的的 API 形成 Solid 的强大身份。他们的 API 设计的质量和思想在竞争中脱颖而出。他们凭借模式的力量促使整个行业的人们重新思考整体反应的能力是强大的。我最大的愿望之一就是有一天 Solid 也能达到类似的高标准。

Svelte 帮助 Solid 从根本上理解了反应性的本质。当你去掉所有移动的部分，你就能真正评估什么是重要的。只有在这样做之后，我才意识到为什么以及哪里需要添加东西。也许不是工具里的所有东西都能“激发欢乐”，但是重新发现爱是一件很强大的事情。苗条的身材使坚实的身体习惯于不害怕重新思考标准。

Vue 让我真正明白了妥协的代价。取悦所有人可以让你受欢迎，但也可以让你一事无成。这不一定有什么错，但它让我明白了一些事情。我更愿意写一个有分歧但人们感觉强烈的库，而不是一个每个人都喜欢但无论如何都不会引起强烈情绪的库。Vue 真的帮助 Solid 看到了拥抱其真实本质的价值，包括缺点和所有。

总之，这是一次非常令人兴奋的旅程，我学到了很多东西。设计一个库/框架是采取一堆微观决策，并编织一个一致和直观的叙事。当这些决定中的任何一个都可能产生持久的影响时，这并不总是一项简单的任务。下次见。

[](https://github.com/ryansolid/solid) [## 瑞安固体/固体

### 一个用于构建用户界面的声明式、高效且灵活的 JavaScript 库。-瑞安固体/固体

github.com](https://github.com/ryansolid/solid) [](https://medium.com/@ryansolid/designing-solidjs-dualities-69ee4c08aa03) [## 设计固体:二元性

### 看对立面能否帮助我们重新定义看待整体问题空间的方式？

medium.com](https://medium.com/@ryansolid/designing-solidjs-dualities-69ee4c08aa03) [](https://medium.com/javascript-in-plain-english/designing-solidjs-immutability-f1e46fe9f321) [## 设计固体:不变性

### 反应式状态管理可以既是不可变的又是最高效的吗？

medium.com](https://medium.com/javascript-in-plain-english/designing-solidjs-immutability-f1e46fe9f321)