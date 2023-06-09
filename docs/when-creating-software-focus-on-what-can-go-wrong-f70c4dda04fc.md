# 当创建软件时，关注可能出错的地方

> 原文：<https://itnext.io/when-creating-software-focus-on-what-can-go-wrong-f70c4dda04fc?source=collection_archive---------2----------------------->

## 避免愚蠢和聪明一样多

![](img/b6ec119be75576e08a34b4a1034c297c.png)

罗伯特·拉兹洛摄于[佩克斯](https://www.pexels.com/photo/bird-water-animal-lake-7025851/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)

> *“值得注意的是，像我们这样的人通过努力保持不愚蠢，而不是努力变得非常聪明，获得了多少长期优势。”查理·芒格*

收集需求和创建软件的常见方法是关注它需要做什么，它应该如何工作。阻止用户变笨和软件帮助他们完成工作一样重要。

软件不仅仅要工作，它还必须不允许用户陷入混乱或卡住。

关注快乐之路和满足需求将意味着你理解了软件的一面。它让你任由事情出错，因为你没有询问或考虑软件在你的设计中应该做什么。

需求中一个常见的弱点是他们没有考虑软件不应该让用户做什么。

就像技术债务一样，阻止问题的最好时机是在问题发生之前。如果你提前考虑，你可以避免问题或者阻止问题发生。

如果你不去寻找，就很难看到你的解决方案中的风险或遗漏的需求。

# 颠倒问题

> “颠倒，总是颠倒:把一个情况或问题颠倒过来。倒过来看。如果我们所有的计划都失败了会怎么样？我们不想去哪里，怎么去呢？不要寻找成功，而是列出如何失败的清单——懒惰、嫉妒、怨恨、自怜、权利和所有自我挫败的心理习惯。避开这些品质，你就会成功。告诉我，我会死在哪里，就是说，所以我不去那里。”查理·芒格

将问题转化的想法是一种从不同的角度来看待你正在创建的软件的工具，一个在你创建软件之前阻止错误的机会。

思考错误和失误有助于你避免它们，而不是专注于快乐的道路，然后在用户做疯狂的事情时遇到问题，因为他们可以这样做。

我发现反转这个术语很难理解。不清楚我们在努力做什么。相反，把它想象成

*   这个软件可能发生的最糟糕的事情是什么
*   我在努力避免做什么

从你不想发生的事情开始往回走可以帮助你避免最大的错误。

这是一个很好的想法，问什么是这个软件可以工作的最好方式，什么是这个软件可以工作的最差方式。解决方案会在中间的某个地方，但是你会更好的理解软件想要达到的目标，软件的目的是什么。

在谈判中，克里斯·沃斯将此称为问题背后的问题。这凸显了一个事实，人们很少说话算数。在需求中有很多没有说出来，这导致了假设。当你在假设的基础上构建软件时，就有出错的风险。

要求没有错，是少了一些要求。您需要揭示需求的目的，以帮助您在构建之前找到缺失的需求。

# 反问题的例子

您正在创建一个案例管理系统，供内部用户提出问题供 IT 部门解决。

您的重点是创建一个案例管理系统，用户可以通过电子邮件或电话记录案例。

如果您将它留在那里，您可以很容易地创建一个简单的案例管理系统。而是把问题反过来，会发生什么不好的事情？我们试图避免什么？

*   没有详细信息来联系提出此案的人。用户需要输入有效的电子邮件或电话号码，否则我们无法联系他们。
*   24 小时不回复。我们需要在 24 小时内做出回应。
*   用户要求他们不被允许的软件。这将是浪费时间。对于一些软件，我们需要得到他们经理的批准。
*   无人处理的案子，比如有人去度假了。如果有人去度假，他们的箱子需要别人来拿。

# **技术设计**

这是一个很好的心态来看待你的技术设计，什么可能会出错。这是在构建之前对设计进行逻辑压力测试的一种形式。你越早发现问题，解决问题就越容易、越快。

在软件建立之前，你可以改变设计和成本只是设计，将没有代码或依赖代码来改变。改变软件是昂贵的，因为它可能有依赖的代码、单元测试、测试、文档、培训、部署。

就您的设计提出以下问题:

*   如果服务、组件或解决方案的某个部分不可用，该怎么办？
*   如果我把这个比例放大 100/200/300%会怎么样？还能用吗？
*   什么会导致速度变慢？它需要多快？
*   如果更多的用户需要使用它会怎样？预计 5 年后的使用情况如何？
*   这会让某人不被允许做一些他们不应该做的事情(安全)吗？
*   我们最少需要多少数据？如果我们没有数据呢？

这些问题旨在扭转你的思维，找到你试图回避的问题。把问题反过来，换个角度就是帮你避免错误。

另一个倒置是不假设客户或开发人员知道什么是超出范围的。始终在规范文档中添加假设和范围外部分。

软件开发是失败者的游戏，尽量减少你的错误是一个优势，因为这些是真正减缓软件开发和粉碎计划的原因。

> “值得注意的是，像我们这样的人通过努力保持不愚蠢，而不是努力变得非常聪明，获得了多少长期优势。民间有句谚语说得很有道理，“淹死的是强壮的游泳者。”—查理·芒格