# Angular ngRx 存储单元测试

> 原文：<https://itnext.io/ngrx-store-testing-cheat-sheet-59e069eb47c?source=collection_archive---------0----------------------->

## 有角的

## 测试你的角度存储现在变得很容易与这个卑微的手册🧪

![](img/007be298430d86c7e229914b14a38924.png)

亚历克斯·康德拉蒂耶夫在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄的照片

# 先决条件[📝](https://emojipedia.org/memo/)

阅读这篇文章意味着你对 Angular、Typescript 和 NgRx Store 有一个基本的了解。另外，你了解通量模式(Redux)。

 [## NgRx 文档

ngrx.io](https://ngrx.io/guide/store) ![](img/8d140abd2ef1203a7024a8de42183a88.png)

NgRx 商店图解(来自 [ngrx.io](https://ngrx.io/guide/store)

# 样品商店🏪

为了深入单元测试 Angular ngRx 存储，我创建了一个简单的存储。我们只有一个`data`字段。该属性将通过服务获取。这里没有实现这个服务，但是它可以从 REST api 中检索数据。

在我们的商店中，我们将管理两种类型的命令(也称为动作):

*   一个 ***LoadData*** 命令来触发检索
*   一个 ***SetData*** 动作(带有一个有效载荷)将获取的数据保存到存储器中

## 行动🎬

这里，我们定义了获取和设置`data`的两个动作。

## 还原剂🔍

我们的缩减器指定了`SetData`动作发生时会发生什么。不出所料，它只是设置了`data`。

## 效果🌪️

这是一个非常常见的效果用例，我们加载数据，就像我们在应用程序引导时所做的那样。

当`LoadData`动作发生时，由于`SetData`动作，从服务中获取`data`对象并存储在存储中。

## 选择器🎯

有点过度设计，但这是测试选择器的好方法:

# 单元测试我们的商店🧪

现在是考验的时候了！💥

我们将使用 Jasmine 作为单元测试框架，但这不是强制性的。

## 行动🎬

测试动作非常容易，因为我们只需要检查动作对象的良好创建。

## 还原剂🔍

在我们的 reducer 中，我们将有 2 个测试用例:

*   初始状态
*   `SetData`行动

由于我们将设置数据，我们需要一点配置:从`initialState`中重置`data`属性，以确保它在每个测试的最开始保持不变。这里，我们使用 Jasmine 框架中的`afterEach`方法。

## Effect️s🌪️

测试效果需要更多的配置，因为我们需要模拟商店和数据提供者`DataService`。为了实现这一点，您可以在测试模块的定义中找到设置(第 20–24 行)。

第 23–24 行允许定义一个包含函数列表的模拟对象`DataService`。然后，第 29 行，我们定义一个函数将返回什么值。

最后，我们必须通过向动作管道`actions$`添加动作来触发效果。如第 44–45 行所述，通过订阅效果来操作检查。

## 选择器🎯

由于选择器只是浏览商店特性的一种方式，我们可以创建一个类似于商店的复杂对象来检查选择器是否获取了正确的属性。

在这里，由于工厂的存在，创建状态和实体变得更加容易，使之更适应每个测试用例。

# 特性模块状态呢？🤔

如上所述，可以对每个状态独立地进行单元测试功能模块状态。

然而，你会面临一些关于特性模块中单元测试选择器的问题。它需要改变状态，将我们的特征状态封装到一个根状态中。

# 小费🃏

`MockStore`类提供了一个非常有用的属性:`scannedActions$` *。*这是一个可观察对象，我们可以在其上添加一个订户来检查所有调度的操作。

当你**在一个效果**中调度几个动作时，这可能是有用的。

# 最后的话🔚

对 ngRx 商店进行单元测试实际上非常容易，对吧！？由于通量模式，商店的每个部分都是自给自足的，可以独立测试。

如果您遇到了本文迄今为止没有涉及的特定案例，请随意发表评论。