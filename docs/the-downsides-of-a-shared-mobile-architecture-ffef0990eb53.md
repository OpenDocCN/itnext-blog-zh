# 共享移动架构的缺点

> 原文：<https://itnext.io/the-downsides-of-a-shared-mobile-architecture-ffef0990eb53?source=collection_archive---------3----------------------->

![](img/d6587bcaf1ebe3a9981bdcd03d18e061.png)

照片由 [Unsplash](https://unsplash.com/) 上的[泰勒 B](https://unsplash.com/@carstyler)**拍摄**

在我的[上一篇文章](https://www.alessandrorecchi.com/shared-mobile-architecture-pros/)中，我们讨论了当我的团队决定在 Android 和 iOS 上设计相同架构的新功能时，我发现的所有好处。在本文中，我将反其道而行之，向您展示这种方法的所有缺点。

没有什么是完美的，与您分享我们使用这种方法的好处以及我们面临的缺点和问题是正确的。作为奖励，我也会试着给你一些如何克服这些问题的建议。

在这篇文章中，我将告诉你我们遇到的问题，我们可以将它们分为两类:**人员&流程**问题和**技术**问题。你可能会惊讶地发现，第一类比第二类更受关注。这是每个开发人员越资深就会发现的事情。**人&过程**问题通常比**技术**问题更难解决，也更微妙(我在这里做了一点概括，我肯定这在 NASA 是不同的)。

# 带人上船

![](img/7e2bb531ac6878856c7f9d6fb41f4c44.png)

照片由[this is 工程](https://unsplash.com/@thisisengineering) 在 [Unsplash](https://unsplash.com/) 上拍摄

当决定用一个统一的架构开发一些东西时，你将面临的第一个问题是来自其他人的反对。为什么？因为并不总是有人愿意改变。几乎可以肯定的是，新的做事方式将要求来自一个平台或者在某些情况下来自两个平台的开发人员以不同的方式做事。你团队中的人肯定会问你“为什么我要以不同的方式做事？我们现在做事的方式很好”。

不幸的是，我既没有一个解决方案，也没有一个神奇的公式让你去说服人们，新的做事方式会更好，甚至对于一个实验周期来说足够好。这取决于你。要知道，说服他人并让他人参与进来是高级开发人员和领导者的关键技能，你越早开始练习越好。

# 厨师太多了

第二个问题更微妙，如果你有一个像加满油的机器一样完美工作的团队，它可能不会发生。我称之为“厨师太多”的问题。当有太多人在处理同一个问题**时，很难统一每个人对解决方案的看法**。

这是一个棘手的问题，既可能发生在设计阶段，也可能发生在实现阶段，但希望不要同时发生在这两个阶段，我建议您在认识到解决方案的决策不是在正常的时间跨度内做出的之后，尽快尝试并修复它。

让我告诉你我们发生了什么。我不得不说，这次我在设计阶段没有看到任何问题。当然，有很多对话和来回，但感谢我们的经理[汉尼斯·多尔夫曼](http://hannesdorfmann.com/)，决定是及时做出的。因此，我能给你的避免设计阶段出现问题的唯一建议是**有一个强有力的技术领导者**给你指明正确的方向，或者有一个像一个单元一样工作的自组织团队。

不幸的是，在实施阶段并不顺利。我们很快注意到，拉取请求(PRs)被批准和合并的时间比平常要长。在大多数文章中，你可以找到关于编程风格和细节的长篇对话。我们试图编写代码，以匹配来自 2 个不同平台的 4 个人的风格和偏好。让我告诉你，这是一件超级困难的事情，如果不是不可能的话。几个星期后，我们认识到我们有一个问题，并决定通过以更好的方式定义跨平台评审者的目的来解决它。跨平台评审员是第二道防线，确保公关遵循我们同意的设计，但更重要的是，实现与其他平台的实现基本一致。

请注意，我写的代码大多是一致的，而不是完全相同的，因为试图写完全相同的代码(直到单个构造)会大大降低我们的速度。我认为，如果你想要在两个平台上有完全相同的代码，那么你应该在跨平台开发上做更多的研究。就是这样！经过这次调整后，实现阶段进行得更加顺利。

# 长期训练

我们将讨论的最后一个人的问题是关于长期的纪律。一件事是留出大量的时间来同时设计和实现一个完整的特性，但是几个月后，当随机的改进标签开始出现时，会发生什么呢？我相信您也经历过两个平台的目标或范围略有不同的 sprints。可能一个平台有时间开改善票，另一个没有。接下来会发生什么？

你可能会想，既然特性是一起设计的，那么来自一个平台的开发人员可以首先设计和实现特性，甚至可以添加来自另一个平台的开发人员作为评审人员，以保持跨平台评审人员的精神，这样一切都会很好。

**在这里要非常小心，因为你可以使用统一的架构来撤销你在一开始所做的所有努力**。首先，由于该票证不属于其他平台开发人员的目标或范围，他/她可能没有时间对其进行适当的审查。此外，由于设计和实现主要是在一方完成的，其他开发人员可能不会对结果 100%满意，因为如果他们出现在设计阶段，他们会提出不同的想法。最后，我敢打赌这样做将引入一些轻微的实现差异。在原项目发布 6 个月后实施新票时，同样的事情也发生在我们身上。

每次发生这样的事情，你都冒着增加小差异的风险，这些小差异会逐渐导致整个特性的指数级差异。保持强大，为两个平台有一个一致的 sprint 目标而奋斗，这样你就可以继续在相同的主题上工作。

# 选择错误的架构

当选择以统一的方式开发一个特性时，只有一个主要的技术问题，这也是通常想到的第一件事:“如果我选择了错误的架构怎么办？”。让我告诉你，这并不像你想象的那么严重。让我们来探索一下。

![](img/c948feaf5d4b1dfc1f24b4bc1defdf23.png)

[马克老板](https://unsplash.com/@vork)在 [Unsplash](https://unsplash.com/) 上的照片

为两个不同平台上的新特性选择一个架构和一个设计并不容易。有许多活动部分需要考虑，如平台差异、解决方案复杂性以及满足需求。可能会发生这样的情况，你犯了一个错误，结果两个应用程序的设计都是错误的。

真的有听起来那么糟糕吗？有点，但也不尽然。比方说，你设法实现了这个特性，但是你不得不用一个糟糕的设计来实现它。恭喜你！你刚刚介绍了技术债，欢迎来到黑暗面。这意味着从长远来看，你将不得不从两个平台的开发者那里花时间来修复它，甚至完全重构它。

你想知道比两个平台都有技术债更糟糕的是什么吗？只在一个平台上有技术债务，而在另一个平台上没有问题。迷茫？这是个好兆头。

只在一个平台上有技术债务意味着将来只有一半的团队可以实现新的东西，而另一半将忙于修复过去的错误。现在，你创造了一个不一致的团队，一个更难管理的团队。

我们在[之前的文章](https://www.alessandrorecchi.com/shared-mobile-architecture-pros/)中详细讨论了为什么不一致是不好的，所以我现在不打算重复。虽然这可能看起来违反直觉，但我相信**在两个平台上引入技术债务比只在一个**上引入要好。至少，如果你在两个平台上都有技术债务，你可以更容易地与你的产品经理合作，给出正确的优先级，并尽早解决它。

# 意识是关键

这篇文章的重点是非常明确地告诉你，没有什么是完美的，每条道路都有自己的起伏。然而，更重要的是，在知道未来风险的情况下做出决定或**选择一条道路。在软件开发中，意识是关键。如果你知道可能会发生什么，你就可以做好准备，减轻可能出现的问题。**

尽管上两篇文章完全基于我自己的经验，但是看到这些确切的问题和解决方案一次又一次地从不同的人那里出现还是很有趣的。最新的例子来自务实的工程师 Gergely Orosz。他写了一本名为《大规模构建移动应用:39 个工程挑战》的书，其中有一章是关于跨平台开发的。那里强调的问题和好处与这里和[上一篇文章](https://www.alessandrorecchi.com/shared-mobile-architecture-pros/)中发现的非常相似。

你觉得怎么样？你曾经面临过这些问题吗？你是怎么解决的？请在下面的评论中告诉我，或者发微博给我 [@_alerecchi](https://twitter.com/_alerecchi)

*原载于 2021 年 6 月 16 日*[*https://www.alessandrorecchi.com*](https://www.alessandrorecchi.com/downsides-shared-mobile-architecture/)T22。