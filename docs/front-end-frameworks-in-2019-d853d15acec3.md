# 2019 年的前端框架

> 原文：<https://itnext.io/front-end-frameworks-in-2019-d853d15acec3?source=collection_archive---------0----------------------->

## 我对当今最重要的前端框架的看法，以及对未来的展望

![](img/6faa7438568cb7071203f3173c60529d.png)

Artem Sapegin 在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

现在是 2019 年，写 Javascript 从来没有这么好玩。

在过去的几年里，社区给了我们自动启动我们项目的构建器，格式化我们代码的 linters 和纠正我们的 types 我们可以使用 WebAssembly、WebWorkers/ServiceWorkers，我们的应用程序可以在几乎任何设备上运行。

由于 Webpack、Parcel 和 Rollup 等工具，我们今天拥有的可能性简直太棒了。

并且，**我们有大量的框架和库**。成吨。这难道不是一个神奇的编码世界吗？我也这么认为

但是有了这么多，我们经常要为雇主要求我们开始的下一个项目选择堆栈，然后，像通常发生的那样，为它辩护。

**注意**:这个帖子并不是要煽动一场战争或者类似的事情。相反，我想邀请读者通过讨论我的观点和意见来反思框架的现状。

# 标准化和成熟度

虽然我们可以肯定地看到每天都有新的东西出现，但不可否认的是，过去几年疯狂涌入的新库和框架已经减少，整个行业似乎已经达到一定程度的标准化和成熟。

事实上，业界似乎已经同意:

*   组件很酷
*   反应是冷静的
*   Typescript 很酷
*   组件中的状态(大部分)是个坏主意

## 概念和范例的一致性

当不得不学习新框架时，最困难的事情通常不是它的 API，而是它不同的编程范式。

当我在 2013 年学习 AngularJS 时，我来自一个 JQuery 背景，我不得不改变我关于构建用户界面的整个观点。这是非常不同的。

虽然每个库都有各种各样的 API 和实现，但我们可以说，由于以上几点的标准化，我们不需要完全改变我们编写和思考代码的方式。

例如，React 无疑普及了组件的概念，并且(多亏了 Redux)将状态放在了组件之外。

然而，其他框架已经注意到了这一点，并随后实现了类似的概念:组件也是 *Angular* 和 *Vue* 中的构建块，它们都有自己的 Redux 实现(NgRx 和 Vuex)，这是目前这些框架中管理状态最常用的模式。

这并不是说你很快就能学会这两者——而是因为他们都分享了架构背后的核心概念，所以你会轻松很多。

一切都感觉“有点相似”。

## 所以——不再有 Javascript 疲劳了？

不完全是。

Javascript 仍然是一种发展非常迅速的语言，有一个蓬勃发展的社区，我们仍然需要不时地重新学习、更新和重构。

但与几年前相比，这是一个更加标准、稳定和成熟的行业，这是一件好事。

# 大的:反应，Vue 和角

众所周知，我们有看起来是最大竞争对手的三大框架:React、Vue 和 Angular。

*   React 已经成为最常用的框架，原因有很多；除此之外，还有各种具有兼容 API 但占用空间较小的框架，如 Preact 和 Inferno
*   Angular 在历史上是强大的，并且已经发布了一个健壮的新版本，该版本仍然试图在非企业团队和用户中找到爱
*   Vue 看起来有其他两家公司的长处，没有任何科技巨头的支持，有一个大的、热情的社区

这些框架的质量在某种程度上已经稳定下来；他们各有所长，也有争议——但总的来说，我想说他们之间的差异往往是个人观点的问题。

## 选择框架:观点问题，还是客观差异？

无论您是否关心寿命、社区规模、性能、捆绑包大小和生态系统，这些框架都提供了类似的结果。

那么如何选择选择什么技术呢？如果你选择一个而不是另一个，你真的会错过吗？

*   如果你的团队已经有了使用框架的经验，安全的答案是继续使用那个框架，除非你有真正的理由不这样做
*   一般来说，不会。我不认为选择一个会让你错过另一个拥有的令人难以置信的功能或性能

这通常取决于您的偏好:正如我在上面所说的，这些框架在某些方面是相似的，但是也有一些基本的区别。例如:

*   用*钩子*对禁止类的最佳实践做出反应
*   相反，在 Angular 中，您几乎只是在编写类:如果您更喜欢一种方法，这是一个很大的区别
*   Vue 非常灵活，让您可以自由选择。可以写 JSX，Angular 启发的组件 API 等。鉴于 Vue 提供的灵活性，它是一个非常安全的选择

## 工具和社区

在社区和工具方面，他们似乎也不相上下:

*   它们都可以由服务器渲染
*   它们都可以是懒惰装载的
*   它们都可以作为移动应用程序本机运行(Nativescript，React Native)
*   他们有很棒的设计系统的 UI 组件库
*   除了 Angular，Vue 和 React 都有第三方静态站点生成器

在写这篇文章的时候，我想说的是，对于一个框架来说，很少有其他框架没有类似的工具或库的。

# “消失的框架”的兴起

另一方面，我们也有大量较小的突出项目，如 Svelte 和 Stencil(以及许多其他项目)。

所谓的“消失的框架”是极其令人兴奋的项目，原因很简单:它们编译你写的代码，没有框架；因此，应用程序将变得更小、更快。

对于某些项目，这些因素可能极其重要。

## 何时使用更小的、消失的框架？

什么时候使用这些较小的框架有意义？

*   当然，当你只是比其他人更喜欢他们的时候…
*   对于构建公司范围内的 UI 组件:由于能够编译为 Web 组件，这些框架非常适合与其他项目共享代码，可能是用 big 3 或普通 Javascript 编写的
    。例如， *Ionic* 能够通过用模板编写组件，而不是从头开始重新编写，快速将其框架不仅发布到 Angular，还发布到其他框架
*   面向普遍使用低功耗设备或慢速网络的受众的应用程序

## **成熟的企业应用怎么样？**

嗯，也许吧。

我没有足够的经验来回答这个问题。但是我认为随着越来越多的公司和项目采用它们，也许我们会看到这些框架在大范围内如何比较。

# 外卖食品

现在是做开发者的大好时机，好到选择一个框架都是有争议的事情。

*   框架在概念和最佳实践方面大多是稳定的——尽管它们有明显的差异
*   尽管框架和库的发展速度趋于稳定，Javascript 仍然很难跟上
*   选择一个框架而不是另一个框架不会让你错过比其他框架更重要的东西。根据您的喜好，看看每个框架的未来发展中哪一个最吸引您
*   注意新来者:所谓的“消失的框架”,如 Svelte 和 Stencil，正在提议成为该领域中可能的新玩家，而像 Ionic 这样的成功项目就是一个例子，说明如何利用这些类型的框架来构建跨框架组件

如果您需要任何澄清，或者如果您认为有些事情不清楚或错误，请留下评论！随时欢迎反馈。

我希望你喜欢这篇文章！如果有，请关注我的 [*中的*](https://medium.com/@.gc?source=post_page---------------------------) ， [*Twitter*](https://twitter.com/home?source=post_page---------------------------) *或我的* [*网站*](https://frontend.consulting/) *以获取更多关于软件开发、前端、RxJS、Typescript 等方面的文章！*

# 了解更多信息

[](https://blog.bitsrc.io/let-everyone-in-your-company-see-your-reusable-components-270cd3213fe9) [## 让你公司的每个人共享你的可重用组件

### 以可视化的方式分享你现有的技术，帮助 R&D、产品、营销和其他所有人共同建设。

blog.bitsrc.io](https://blog.bitsrc.io/let-everyone-in-your-company-see-your-reusable-components-270cd3213fe9) [](https://blog.bitsrc.io/sharing-components-with-angular-and-bit-b68896806c18) [## 与角度和位共享组件

### Bit 简介:构建和共享角度组件

blog.bitsrc.io](https://blog.bitsrc.io/sharing-components-with-angular-and-bit-b68896806c18) [](https://blog.bitsrc.io/how-to-easily-share-react-components-between-projects-3dd42149c09) [## 如何在项目和应用程序之间共享 React UI 组件

### 帮助您在团队应用程序之间组织、共享和同步 React 组件的简单指南。

blog.bitsrc.io](https://blog.bitsrc.io/how-to-easily-share-react-components-between-projects-3dd42149c09) [](https://blog.bitsrc.io/simplify-complex-ui-by-implementing-the-atomic-design-in-react-with-bit-f4ad116ec8db) [## 使用 React 和 Bit 的原子设计:简化复杂的 UI

### 使用带有 React + Bit 的原子设计来简化复杂的 UI。

blog.bitsrc.io](https://blog.bitsrc.io/simplify-complex-ui-by-implementing-the-atomic-design-in-react-with-bit-f4ad116ec8db)