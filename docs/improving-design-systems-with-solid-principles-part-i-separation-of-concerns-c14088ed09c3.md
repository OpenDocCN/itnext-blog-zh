# 用坚实的原则改进设计系统。第一部分:关注点分离

> 原文：<https://itnext.io/improving-design-systems-with-solid-principles-part-i-separation-of-concerns-c14088ed09c3?source=collection_archive---------3----------------------->

![](img/60609307c3dd40524e1af27aac20db6a.png)

不久前，如果有人提到“设计模式”，我的脑海中会浮现出苏格兰短裙和圆点图案，但随着我越来越多地处理和扩展现实生活中的界面，我越来越确信这些原则不仅在前端开发中有它们的位置，而且实际上应该像它们在后端开发中一样定义它。

用户界面应该对变化有弹性，不管风是从尾部还是从头部吹来都没有关系——适应变化的产品需求和设计需求应该是一种无痛的开发体验。坚实的原则可以帮助我们构建更好的设计系统和组件库，方法是应用既定的准则来设计经得起未来考验且可测试的前端解决方案。

我不会详细解释固体代表什么，因为已经有很多关于这个主题的文章了:这里的，这里的，这里的。相反，我想把重点放在我从错误中学到了什么，以及我建立的管理工作的规则上。

为了简洁起见，我不会用代码来说明这些想法，而是将在一系列文章中继续这一思路，重点关注我在这里所做的假设。

# 关注点分离

前端应用程序承担着各种各样的责任，并且经常有太多的理由要改变。当从一个可靠的角度来看一个设计系统时，我们应该看看组成我们应用程序的不同层，并找到分离或抽象逻辑的方法来满足单一责任原则和同等程度的其他 4 个原则。

## 控制反转

*   **依赖层:**这些年来我们看到的无数失败足以证明，虽然我们应该避免重新发明轮子，但如果轮子变平了，我们也应该有更换轮子的选择。我们的库应该以这样一种方式设计，允许依赖关系的替换，而不需要全面的重构任务。

> ***设计系统组件不应该直接依赖第三方依赖。***

*   **服务层:**只要有可能，服务单件不应该直接从其他模块导入，因为这会使测试复杂化，并使应用程序与单个具体实现保持耦合。相反，应该反转或注入服务依赖关系。

> ***设计系统组件不应该直接导入服务单件。***

## **错误处理**

*   **错误处理层:**不幸的是，错误是每个应用程序的自然组成部分，它们的严重性以及对应用程序和用户流的影响各不相同。虽然错误处理很重要，但是单个组件可能应该避免捕捉和处理错误，因为它们应该保持沉默，不知道应用程序及其父组件使用什么逻辑来处理和报告错误。错误处理程序要么用 props 注入到组件中，要么在更高的级别被捕获。组件不应该知道它们的运行时环境，因此记录到控制台也是不允许的

> 设计系统组件不应该试图自己处理错误。

## 从外部服务获取和传播数据

*   **传输层:**与外部服务的通信不应该依赖于任何特定的客户端实现。无论您是使用`fetch`还是`axios`，或者是通过 web 套接字还是 HTTP 进行通信，都不应该影响应用程序的其他部分。

> ***设计系统组件应该不关心应用程序如何与外界通信。***

*   **API 层:**外部服务更改其规范、添加或修改调用端点和签名、更改响应代码等等是很常见的。设计系统不应该对 API 实现细节做出假设，应该提供足够的抽象层次。后端使用 RESTful API 还是 GraphQL 对单个组件来说并不重要。

> ***设计系统组件不应该知道数据来自哪里。***

*   **转换层:**数据模型发生变化，新版本的 API 规范开始启用，通常要求前端应用程序适应这些变化。因此，前端模型不应该与后端 dto 耦合，以避免深入组件树进行更改。

> ***设计系统组件不应该关心外部数据流的格式或范围，而应该只以他们喜欢的格式消费他们需要的东西。***

*   **同步层:**处理异步数据是大多数应用程序的日常痛苦。获取、重新获取和缓存应该由应用程序中的专用层来处理，组件应该接收解析的数据或者负责挂起子组件的呈现。如前所述，组件应该独立于用于处理异步数据的库:`useSWR`、`useQuery`、`useAsync`等都做同样的事情，但都有自己的注意事项，组件不应该处理它们的怪癖。

> 理想情况下，设计系统组件应该是无状态、同步和受控的。

## 状态管理

*   **存储层:**组件应该能够独立存在，而不应该被绑定到全局存储机制的细节上。消费者可以使用 Redux、反冲、上下文或其他任何东西来管理他们的状态。如前所述，组件应该尽量保持无状态，并通过容器和/或更高级的组件从外部进行控制。状态突变应该通过注入到 props 组件中的处理程序来实现。就目前情况而言，组件甚至不应该知道它们正在改变某些东西——它们应该简单地使用回调将事件发送到外部世界，这些事件发生了什么与组件无关。

> *设计系统组件不应直接访问全局应用程序状态。*

*   **缓存层:**本地存储不是通用的，可能在运行时环境中不可用。尽管浏览器中有许多可用的存储机制，但应用程序不应该总是依赖于它们的可用性或可访问性(由于隐私偏好、供应商不一致、大小限制等)。缓存应该是可选的，并且应该总是发生在组件范围之外。

> *设计系统组件不应直接访问缓存。*

## 运行时环境

*   **运行时层:**现代应用在各种环境下构建、渲染和运行:浏览器、无头浏览器、SSR 服务器、CI/CD 环境。设计系统应该是通用的，不针对任何一个特定的环境，也不依赖于特定于运行时的 API 的可用性。窗户不总是窗户。

> *设计系统组件不应假设其运行时环境。*

*   **视口层:**智能手表、手机、平板电脑、台式机——设计系统应该能够在各种屏幕尺寸上工作。单个组件试图适应屏幕尺寸可能不是一个好主意——他们应该从上面得到指示，或者设计系统应该提供适合各种屏幕尺寸的多个组件。向组件收取提供功能和适应各种屏幕尺寸的费用扩大了它们的责任范围。

> *设计系统组件应该在设计上做出响应，同时保持与它们被渲染的视口大小无关。*

*   **动画层:**硬件加速是一块难啃的骨头。在某些情况下，没有动画比一个起伏不定的更好。没有人关心屏幕阅读器上的动画。此外，动画应该是上下文相关的，符合用户体验的整体流程。

> *设计系统组件不应执行自己的动画。*

*   **辅助功能层:**鼠标、键盘、触摸屏、阅读器——在各种设备类型上导航应用程序并与之交互应该是可能的。组件应该使用语义标记并遵循 ARIA 指南，以确保最大的兼容性、可访问性和互操作性。

> *设计系统组件应该可以在所有类型的设备上访问，而不需要知道使用什么类型的设备来访问它们。*

*   **本地化层:**一个设计系统不应该成为本地化和国际化的瓶颈。一个组件没有理由偏爱英语而不是法语，英寸而不是厘米，或者用其他地区的细微差别和特性使人困惑和沮丧(看着你，美国)。

> *设计系统组件应该在所有语言环境中无缝工作，而不需要知道它们在什么语言环境中使用。*

## **DOM**

*   **层次层:**为了确保可组合性，组件不应该对它们在 DOM 中的安装位置以及它们相对于其他节点的位置做出假设。例如，给单个组件增加边距违反了几个基本原则。

> 组件不应该知道它们在 DOM 中的位置，也不应该知道它们的兄弟或父组件。

*   **3D 定位层:**CSS 树中元素的堆叠应该是集中的，由应用程序的需求决定，而不是由组件的类型决定。

> *组件不应该管理它们自己的 z 索引，也不应该知道它们在堆栈中的位置。*

## 介绍会；展示会

*   **组合层:**设计系统应该提供一组原语，可以用来组合其他组件，然后可以用来组合进一步的组件:原子设计原则是确保组件可重用的一个很好的起点。

> *组件应该能够以任意顺序排列它们的子组件。*

*   **样式层:**组合的组件应该能够向外部世界表示它们的组合和状态(例如，禁用的、活动的、无效的)(例如，使用块-元素-修改器类系统)，但是它们不应该有任何内部内联样式，因为那会违反多重实体原则。组件可以根据它们的变体来调整它们的组成，但是它们不应该有任何线索来解释变体是如何转化成它们的视觉外观的。消费者应该能够关闭所有的样式(例如，在屏幕阅读器上)并且仍然理解应用程序。

> *组件应该能够没有任何风格的存在，并且不应该关心它们在外界看来如何。*

*   **主题层:**外观决策应该在设计系统令牌的层次上做出。更改应该级联到整个组件库或其子集，而组件内部不需要任何额外的更改。你应该能够通过仅仅改变一些变量来重塑你的企业，因此盲目地跟随被大肆宣传的 CSS/JS 框架的例子将会使你未来的生活变得复杂，并且不会使你的库在你自己的组织之外可用。

> *组件应该存在于一个黑盒中，而不知道外界存在的颜色和绝对度量单位。*

# 结论

今天就这样了。我已经开始编写一些代码来说明这些原则，并且会在时间允许的情况下与大家分享。如果你不同意我的任何结论，欢迎留下评论，让我们一起讨论。不管这些假设多么主观，我认为它们确实反映了我多年来经历的现实生活中的挑战。