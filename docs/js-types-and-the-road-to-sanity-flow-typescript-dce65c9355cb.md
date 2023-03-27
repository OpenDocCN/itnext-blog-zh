# JS，类型和理智之路(flow，typescript)

> 原文：<https://itnext.io/js-types-and-the-road-to-sanity-flow-typescript-dce65c9355cb?source=collection_archive---------3----------------------->

不久前，我还被在 JS 上打字的想法折磨着。不是因为这是社区中的新宣传，而是因为我可以想象这样的实现有什么好处。

即使在一个使用 react-redux 不断增长的中小型项目中，您也可以看到，您必须从一个引用到另一个引用才能理解一个动作是如何被调用的。即使在测试中，这也会造成很大的麻烦，尤其是当你试图测试不是你写的代码时。最重要的是，当你开发应用程序时，当大量的错误出现时，类型检查会给你带来安全。

所以我想我可以更深入一点，这样我就可以得到一个现实的想法，并根据我的发现来实施。

# 今日 JS

在潜入深水之前，让我们先想想我们要解决的问题。JS 不是一种强类型语言，但正因为如此，它可以让每个人都有机会快速、毫无问题地构建应用程序。

> 但是 JS 最好的长处真的会很快变成它的弱点。

尽管 JS 不是一种强类型语言，但它定义了七种内置类型。这里有一个关于这些如何被称为类型、标记或子类型的争论，但是让我们坚持类型。这些类型如下

*   `null`
*   `undefined`
*   `boolean`
*   `number`
*   `string`
*   `object`
*   `symbol`-ES6 中新增！

我们可以看到这些类型就是他们声称使用的`typeof`。在这里，在我调查的最开始，你可以看到这个问题。当惨叫看似没问题的时候，总有一个..酪

```
typeof undefined     === "undefined"; // true
typeof true          === "boolean";   // true
typeof 42            === "number";    // true
typeof "42"          === "string";    // true
typeof { life: 42 }  === "object";    // true

// added in ES6!
typeof Symbol()      === "symbol";    // true
```

但是 null 呢？

```
typeof null === ???;
```

Null 是一种非常特殊且容易出错的情况，因为它的类型是 **object** …我知道它看起来很可怕，对吗？但是等一下，还有更多。

即使是内置函数，比如布尔型、字符串型、数组型等等，你也会发现它的局限性。参见下面的例子

```
var testVar = “i dont need types”                   === string
var testVar = new String(“i don’t need types”)      === objectvar testVar = false                                 === boolean
var testVar = new Boolean(false)                    === object
```

你可以看到，如果你用一个布尔值或者一个字符串来创建一个变量，这个变量的类型对应于你所传递的内容。另一方面，如果你创建了内置函数，那么它们的类型就是 object。

如果这还不够奇怪的话，我收集了更多的例子来说服你。

```
10 + 5                                   === 15
'10' + 5                                 === '105'
'10' - 5                                 === 5               
( + '10')                                === 10
```

所以，你不能指望你自己的数字。

![](img/f2432107e8872921e2523468783fe65a.png)

我知道那是什么感觉！

# 类型需求

所以根据上面的例子，显然需要类型。这容易吗？

在研究 JS 项目的类型之前，您需要考虑几个利弊。

## 优势

*   编译器上的错误
*   为下一个开发人员记录的代码
*   避免 javascript 自己不直观地转换类型

## 不足之处

*   更多的发展时间
*   学习曲线(尤其是对于打字稿这种新语言)
*   开发人员池

# 额外——除此之外，您还可以利用 chrome v8 的速度

记住这些，你必须看到你的选择。在 JS 社区中，有两个主要的参与者。流和类型脚本。

![](img/cd8937f860e16690b277df2456f91f2e.png)

我将跳过解释这两个的部分，直接看最后的结果。

在**流**的情况下，你可以逐渐地导入到一个项目中，而不会破坏你的代码。不过这需要特别注意代码和形状，因为如果流程中还没有某些东西，它不会通知您。在这种情况下，您将需要使用 flow-coverage-report，这增加了项目的复杂性。流量社区不断抱怨每一步的突破性变化。这些让心流很难运作。

在打字稿的情况下，你必须改变**一切……**我是认真的。您必须将所有文件更改为 ts、tsx 等类型脚本的文件扩展名。你还必须安装你的模块所需的所有类型，例如 react、redux 等(2 年前你可能会遇到一个问题，但现在你的模块有类型已经是很标准的了)。最后，您必须用类型定义所有现有的代码。相当痛苦。但是(总有但是)社区比流量大 10 倍。

如果你在一年前问我，我可能会选择 Flow 而不是 Typescript，因为所有已经描述过的过程，但 ATM 我认为 2019 年是 Typescript 年。

这是因为

*   被许多大公司选中，超过 Jest 等流量。
*   终于拥有了模块所需的所有类型
*   react create app 提供了用现成的 typescript 项目创建应用程序的选项

# 一段时间后在 React-Redux-RXjs 项目上使用 Typescript 和我的发现/建议

在使用了 3.5 个月之后，我总结出以下 5 个问题。

## 非常棘手的类型检查 redux 生命周期

检查生命周期变得越来越难，但是当你发现你的公式变得越来越简单。原因是您需要单独定义所有的操作，这样您就可以说 action1 将返回该操作。参见下面的例子

然后，通过将所有动作定义为一个类型，可以帮助您将该类型导入到缩减器中，并说明在该缩减器中采取的动作将是该类型中的一个。

现在你在 Reducer 中定义了 initialState 是用户(或者你可能有一些不同的东西，比如 UserReducer 等等)，这个 reducer 采取的动作是 UserActionTypes，并且返回一个用户(或者如所说的 UserReducer 依赖于你)，如果在这之间有任何变化，你都会知道；) .这里最重要的是，如果你的有效负载是一个带键的对象，那么每个开关盒都知道这个属性是否存在，并告诉你一个错误！一开始很难，但之后，新鸭子会用复制粘贴来拯救它。

## 在文件结构上区分类型变得越来越困难

如前所述，我有一个包含所有所需类型的 types.ts 文件。但是有些情况下，当你想规范化一些 API 的数据，或者你在一个 utils 文件中有一个需要在应用程序间共享的属性。我的建议是不要污染 types.ts 文件，将逻辑和文件夹中的类型分开，这样如果你有一个需要类型的 util 文件，就可以将它们放在一个文件夹中。

## Redux-Observable 缺少关于如何对 epic 进行类型检查的文档

我发现很难对我的史诗进行打字检查。原因是我找不到任何文档来源来给出一个很好的例子，这个例子对于理解它的人来说不会太复杂。我使用的解决方案很简单，但我花了一些时间来弄清楚，并确保我的 epic 类型检查正确。

## 连接和合成不再产生相同的结果

我是这样的人，当我有几个 HOC 时，我不会在一个大的 HOC 链中使用它们，而是使用 redux compose。

```
compose(hoc1, hoc2)(Component)
```

这会给你一个疯狂的错误消息，我不能让它工作，因为 compose 返回的不是一个组件。

```
Type error: JSX element type 'Component' does not have any construct or call signatures.  TS2604
```

## 不要害怕在需要的时候使用它们

你完全不用害怕。在我的例子中，我使用情感来设计我的组件，所以任何对我来说不重要的东西都不必进行类型检查。但是不要过度。

另一方面，除了我面临的问题，我还保留了一套规则，以保持它足够简单

## 每只鸭子的类型文件

在每个 duck 上有一个 types.ts 来保存模型、动作和 reducer 结果，但是正如已经说过的，不要用其他类型污染该文件，因为您可以分离逻辑。

## 在每个反应组件上设置道具，并在必要时导出

和 prop-types 库一样，我用 typescript 保持了相同的逻辑。此外，我还分离了任何会污染带有任何 HOC 的组件的逻辑，因此任何带有 HOC 的连接或任何额外的内容都会转到该文件夹的 index.ts，如 redux connect、redux forms 等。在这种情况下，您可以在组件文件中分离您的 props，并使用 Props 和 ReduxProps 之类的东西来提高可读性。

## 始终使用 React。足球俱乐部

我总是在定义反应。即使我不使用孩子或其他任何东西来扩展我的道具，为将来使用提供正确的价值

最后，在我看来，有一个或大或小的项目，安全总比后悔好。如果为 MVP 编写代码或快速编写代码不是您的首要考虑，那么使用类型、流或 Typescript 比普通 Javascript 更好。

> 如果你喜欢这篇文章，可以考虑点击 clap👏并且你可以随时访问[我的其他文章](https://medium.com/@panagiotisvrs)🔥

# **链接**

[https://github . com/getify/You-Dont-Know-JS/blob/master/types % 20% 26% 20g rammar/CH3 . MD](https://github.com/getify/You-Dont-Know-JS/blob/master/types%20%26%20grammar/ch3.md)

[https://github . com/getify/You-Dont-Know-JS/blob/master/types % 20% 26% 20g rammar/ch1 . MD](https://github.com/getify/You-Dont-Know-JS/blob/master/types%20%26%20grammar/ch1.md)