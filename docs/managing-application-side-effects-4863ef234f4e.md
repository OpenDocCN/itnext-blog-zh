# 管理应用程序副作用:Redux-Saga 简介

> 原文：<https://itnext.io/managing-application-side-effects-4863ef234f4e?source=collection_archive---------8----------------------->

![](img/9b49bd96eb8dca464deb77872282b0b4.png)

## 开始之前…

本文由两部分组成:首先了解副作用以及它们与 Redux 的关系，然后深入研究 Redux-Saga 的基本原理。如果你只是对快速理解 Redux-Saga 感兴趣，请随意跳到 Redux-Saga 部分。但是如果你仍然不确定 Redux-Saga 是否适合你，那么本文的第一部分可能会帮助你做出决定。

## 什么是副作用？

Redux 作为 UI web 应用程序中处理状态管理的主要方法越来越受欢迎。在一个项目中采用 Redux 通常会得到这样的结果(尤其是对于第一次使用 Redux 的开发人员):

1.  阅读并浏览 [Redux 教程](https://redux.js.org/basics/basic-tutorial)
2.  为应用程序的一小部分写一些动作创建者和缩减者
3.  尝试发出一个 AJAX 请求，并将结果数据存储在 Redux 中
4.  开始搜索堆栈溢出，找出为什么这个简单的任务会如此混乱
5.  质疑人生选择

他们将会发现，Redux 对于什么是“核心”功能的概念非常有限。在纯 Redux 应用程序中，应用程序遵循以下顺序:

1.  调度一个动作
2.  减压器改变商店
3.  重复

这个应用程序流是同步和确定性的(因为 reducers 被限制为同步和确定性的)。通常，这个流程也完全不足以处理现代 UI web 应用程序执行的所有可能的任务。

从 Redux 的角度来看，任何发生在正常流程之外的事情都被认为是一个**副作用**，因此完全由开发人员来决定他们应该如何建模和实现这些任务，以及他们应该如何与 Redux 的准系统应用程序流程进行交互。这包括以下内容:

*   与异步 API 的交互
*   通过 AJAX 请求获取/发布数据
*   设置超时和间隔
*   调度操作以响应其他操作

例如，假设我们有一个带有`getUser(id)`方法的 API 客户端，它将调用一个 REST API `/users/:id`来返回一个用户配置文件。我们希望将这些数据存储到 Redux 中，这样就可以跨应用程序访问这些数据。

让我们为此设计动作和状态。第一反应可能是创建一个动作，`GET_USER`，然后用一个 reducer 处理这个动作。然而，用户数据将被异步检索(因为它是一个 AJAX 请求)，所以我们将遵循 [Redux 关于设计异步动作和状态的建议](https://redux.js.org/advanced/async-actions#async-actions):

现在我们有了一组基本的动作创建者，我们想实际上把它们连接起来并发出 API 请求(这是 Redux 的副作用)。让我们探索几个选项，使用这个 Redux 设置作为我们的基础。

# 使用 Redux 时如何实现副作用？

由于 [Redux 对如何实现副作用并不固执己见](https://redux.js.org/faq/actions#how-can-i-represent-side-effects-such-as-ajax-calls-why-do-we-need-things-like-action-creators-thunks-and-middleware-to-do-async-behavior)，因此出现了许多管理它们的模式。我喜欢将这些分为两类:完全在 Redux 生命周期之外运行的模式(**Redux-外部**)，以及与 Redux 生命周期交错的模式(**以 Redux 为中心的**)。

# 冗余外部模式

实现副作用最直接的方法(也是大多数开发人员首先会采用的方法)是独立于 Redux 编写和触发它们，并让这个独立的代码[调用 Redux](https://redux.js.org/api/store#store-methods) (通过`dispatch`或`getState`)。

## redux-外部模式:视图框架

假设您正在使用一个框架来呈现或控制应用程序，将副作用实现作为代码的一部分存在似乎是很自然的。例如，如果您使用 React，常见的方法是在挂载组件时触发副作用，并通过绑定的动作创建器将副作用的不同异步转换绑定到 Redux:

通过将 Redux 绑定到各自的视图或服务层，可以使用其他常见的 JS 框架编写类似的模式。如果没有框架，这种模式看起来就像让 DOM 事件处理程序触发副作用代码。

**好处**

*   通过将副作用逻辑与副作用结果的主要消费者联系起来，会更容易理解
*   不涉及添加新的库/依赖项

**不利方面**

*   更难测试和重用代码(有时；这可以通过仔细的预见和计划来避免)
*   通常通过给先前的纯逻辑增加副作用来增加部件/控制器逻辑的长度和复杂性
*   强制将副作用绑定到组件生命周期，而不是独立运行
*   不容易对应用程序流中发生的事件做出反应，例如被分派的操作

> ***注:*** *在撰写本文时，*[***React Hooks API***](https://reactjs.org/docs/hooks-intro.html)*仍处于 alpha 开发阶段。这似乎是一种很有前途的方法，可以使这种模式更容易在 React 中测试和重用，并最小化组件代码的复杂性。然而，它仍然会受到副作用与组件生命周期相关的不利影响。*

# 以冗余为中心的模式

虽然使用 Redux-External 模式更容易实现，但是它缺乏触发更复杂副作用的能力，比如调度动作来响应其他动作(是的，您可以使用`[subscribe](https://redux.js.org/api/store#a-id-subscribe-a-subscribelistener-subscribe)`方法，但是 Dan Abramov 说[您可能不应该使用](https://github.com/reduxjs/redux/issues/303#issuecomment-125184409))。对于像这样的副作用，我们应该使用一种通过某种中间件将其自身绑定到 Redux 的模式。

以下对以 Redux 为中心的副作用模式的描述很大程度上是受 Gosha Arinich 的文章[**3 Redux 应用**](https://goshakkk.name/redux-side-effect-approaches/) 中常见的副作用处理方法的启发，为了方便起见，在此重新描述。

## 以 Redux 为中心的模式#1:智能动作创建者

在标准的 Redux 实现中，动作创建者是纯粹的**，这意味着他们将简单地创建并返回一个对象，可选地基于传递给动作创建者的一些参数。在这种模式中，我们可以决定让动作创建者也执行期望的副作用。这通常是通过让动作创建者返回简单对象之外的内容，并使用中间件来处理这些非对象动作来实现的:**

**[**Redux-Thunk**](https://github.com/reduxjs/redux-thunk) 是这种模式的常用实现。**

****好处****

*   **不需要学习标准 JavaScript 之外的任何新概念或心理模型**

****不利方面****

*   **动作创作者不再纯粹，使他们更难理解和测试**
*   **除了简单的副作用之外，经常会导致回调地狱(尽管 async-await 可以帮助解决这个问题)**
*   **智能动作创建者只能在被调用时运行，而不是对任意动作做出反应**

## **以冗余为中心的模式#2:智能中间件+专门的动作**

**我们可以将工作转移到中间件中，让我们的动作为中间件提供特殊的指令，而不是在动作创建器中运行副作用。这类似于**智能动作创建者**模式，除了动作创建者仍然是纯粹的，现在中间件将拦截动作并处理执行中的副作用:**

**[**Redux-Promise**](https://github.com/redux-utilities/redux-promise) 是这种模式的常用实现。**

****好处****

*   **动作创作者保持纯洁，所以更容易测试**

****缺点****

*   **更难概括来处理任何期望的副作用；每种副作用通常都需要自己特定的中间件来处理**

## **以 Redux 为中心的模式#3: Redux 挂钩/监听器**

**我们可以采用前面的模式，并通过完全去除动作的任何特殊化来进一步推广它。如果我们编写定制的中间件来监听被调度的通用动作，并独立于 Redux 生命周期执行副作用，那么我们的动作创建者和动作将保持简单和纯粹，副作用将完全由该中间件描述:**

**实际上，这种模式的实现很少让开发人员直接编写每个中间件；他们通常会创建一些抽象的模型来进行开发。 [**Redux-Saga**](https://github.com/redux-saga/redux-saga) 是这种模式的常用实现。**

****好处****

*   **动作很简单，不包含副作用的业务逻辑**
*   **允许许多听众对一个动作做出反应**
*   **包含副作用逻辑，如果实现正确，很容易测试**

****不利方面****

*   **这种模式的实现在复杂性和有用性方面有很大的不同**
*   **会有一个陡峭的学习曲线**

# **我应该选择哪种模式？**

**一如既往，这将取决于你的项目的需求和目标。这些模式按照它们的伸缩方式大致排序(不幸的是，也按照复杂性排序)。对于小而简单的项目，使用 Redux-External 方法通常就足够了。随着应用程序的增长，越来越多的部分变得相互依赖，您可能希望切换到能够更好地管理代码的模式。**

**如果你还在读这篇文章(很高兴你还在这里！)，那么您可能对您当前使用的模式不满意，并且有兴趣看看还有什么。在 [**广阔**](https://www.expanse.co/join-us/) 这里，我们已经将 [**Redux-Saga**](https://github.com/redux-saga/redux-saga) 用于我们的应用程序，它帮助我们组织我们的代码库并简化我们实现副作用的方式。不过，离开地面可能会很困难；本文的其余部分致力于使学习曲线变得更容易接近。**

# **还原传奇**

**Redux-Saga 是 **Redux Hooks/Listeners** 模式的一个实现，定义了一个“Saga”编程模型来处理副作用。**

**Sagas 是与应用程序一起运行的“线程”或“子例程”。像线程一样，它们可以在任何上下文中暂停、启动或取消(通常使用 Redux 操作)。此外，因为 Redux-Saga 是建立在 **Redux Hooks/Listeners** 模式之上的，所以 Saga 也可以完全访问 Redux 状态，可以“阻塞”或“等待”Redux 动作，并且可以调度它们自己的动作。**

## **ES6 发电机**

**使用 Redux-Saga 的最大学习曲线之一是它依赖于 [**ES6 生成器**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*#Description):React/Redux 开发人员不常用的语言特性。**

> **"为什么我不能用普通代码写传奇？"**

**生成器有一些好处，比如允许以同步方式编写异步流(类似于 async/await)。但主要的特点是，它们允许传奇“持续运行”，而不会阻塞主执行线程。与在 sagas 中直接编写阻塞调用不同，阻塞调用用一个**效果**来描述，并用一个`yield`来“包装”，这将执行控制返回给 Redux-Saga。这允许 Redux-Saga 以增量方式运行 Saga，将执行与主应用程序交错(在了解了更多关于生成器的知识后，这将更有意义)。**

> **"那么 ES6 发电机是如何工作的呢？"**

**生成器是可以自行暂停并将执行控制返回给“控制”函数(从这一点开始称为“控制器”)的函数。他们还在生成器和控制器之间使用**双向消息传递**来共享两个函数之间的上下文。**

**下面的幻灯片演示了这种控制器/发电机关系的一个简单示例，以 [NumberWang](https://youtu.be/0obMRztklqU) 为调:**

> **" Redux-Saga 如何使用 ES6 发电机？"**

**在上面的关系中， **Redux-Saga 中间件充当控制器，**和**应用的 Saga 是生成器**。无论 saga 产生什么，都将被 Redux-Saga 中间件解释为指令，并在指令完成时将控制权归还给 Saga:**

## **还原传奇效应**

**在前面的例子中，saga 向 Redux-Saga 传递了一个承诺，Redux-Saga 将其解释为“解析这个承诺，然后提取并发回结果”的指令。与 async/await 不同，这不是 ES6 生成器内置的功能。这是因为 Redux-Saga 的中间件实现(控制器)决定对所有 promise 类型的消息遵循这些指令。**

****Redux-Saga 将从 Saga 生成的所有消息视为指令。**事实上，它在库中有一整套内置指令，你可以传递给它，这些被称为 [**效果**](https://redux-saga.js.org/docs/basics/DeclarativeEffects.html) 。这些效果允许发送 Redux-Saga 的各种自定义指令，例如等待或调度 Redux 操作，等待解决的承诺，或者组合多个效果。**

**这是之前的 Redux-Saga 示例，现在使用效果作为指示，而不是承诺:**

> **"我如何开始使用 Redux-Saga？"**

**当然，学习编写生成器函数和使用 Redux-Saga 的最好方法是自己编写一些 Saga。如果你犹豫要不要马上运行`npm i redux-saga`,我已经整理了一个包含一系列任务的教学资源库，这些任务将向你介绍生成器函数、应用程序副作用以及使用 Redux-Saga 效果和 API。这个 repo 还包含每个任务的“无 sagas”实现(使用 Redux-External 模式),以便您可以比较这种方法将如何改变您的代码的外观和组织方式。**

**[](https://github.com/jlmart88/introduction-to-redux-saga) [## JL mart 88/redux-saga 简介

### 通过动手实施任务介绍 Redux-Saga—JL mart 88/Redux-Saga 简介

github.com](https://github.com/jlmart88/introduction-to-redux-saga) 

> ***注:*** *以上资源库使用的是*[***TypeScript***](https://www.typescriptlang.org/index.html)***，*** *但是即使你不认识 TypeScript 也还是可以用！按照* `*README.md*` *中的说明运行项目，不进行类型检查，这样您就可以编写普通的 ES6 JavaScript，而不会受到 transpiler 的影响。或者，你也可以以此为契机，* [*跳上打字列车*](https://medium.com/@jtomaszewski/why-typescript-is-the-best-way-to-write-front-end-in-2019-feb855f9b164) *😉。*** 

****其他资源:****

*   **[Redux-Saga 文档](https://redux-saga.js.org/)**
*   **[redux 应用中 3 种常见的副作用处理方法](https://goshakkk.name/redux-side-effect-approaches/) — Gosha Arinich**
*   **什么时候应该使用传奇？ —菲利克斯·克拉克**

**想展示你新磨练出来的还原传奇技能吗？ [**广阔天地正在招人**](https://www.expanse.co/join-us/) ！我们通过为世界上最大的组织提供实时、全面的全球互联网优势视图，来帮助保护这些组织。如果你有学习和成长的欲望，喜欢解决有挑战性的问题，给我们写封短信吧！**

***原载于 2019 年 4 月 13 日 https://www.expanse.co**的* [*。*](https://www.expanse.co/managing-application-side-effects-an-introduction-to-redux-saga/)**