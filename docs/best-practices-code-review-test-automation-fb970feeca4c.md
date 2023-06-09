# 最佳实践代码审查测试自动化。

> 原文：<https://itnext.io/best-practices-code-review-test-automation-fb970feeca4c?source=collection_archive---------0----------------------->

![](img/2d69033b7a5cc15562f62a0120101a10.png)

图片由[黑客资本](https://unsplash.com/photos/uv5_bsypFUM)

代码评审过程的主要目标是评估任何新代码的缺陷、错误和组织设定的质量标准。代码审查过程不应该只包含单方面的反馈。因此，代码评审过程的一个无形的好处是集体团队提高了编码技能。

D 正在进行的测试自动化是关于编写代码的。测试自动化代码很容易被当作“二等公民”。由于没有交付给客户，开发通常不太正式，并且可能缺乏组织中应用的审查和质量实践。最近，我一直在做大量的代码审查。我可能每个工作日花大约一个小时为我的团队处理评审，无论是作为评审者还是作者。所有的评论都专门涵盖了端到端的测试自动化:新的测试、旧的修复、配置变更和框架更新。我坚信测试自动化代码应该和它测试的产品代码一样接受同样的审查，因为测试自动化是一种产品。因此，应该应用所有相同的最佳实践。此外，我还寻找那些在测试自动化中比在其他软件领域中更频繁出现的问题。

ode 评审是软件开发过程中一个非常重要的现象。开源社区使它变得流行，现在它是任何开发团队的标准。如果正确执行，好处不仅在于减少错误数量和提高代码质量，还在于对程序员的培训效果。

尽管代码审查和拉取请求为开发人员所熟知，但是这些概念仍然没有被测试人员完全理解。在我工作过的大多数 scrum 团队中，默认情况下，测试人员不参与审查代码变更请求的过程。

代码审查过程至关重要，因为它从来都不是学校正式课程的一部分。你可能会学到编程语言和项目管理的细微差别，但是代码评审是一个随着组织的成长而发展的过程。

出于以下原因，代码审查至关重要:

*   确保代码中没有错误。
*   最小化你遇到问题的机会。
*   确认新代码符合指南。
*   提高新代码的效率。

代码评审进一步提高了其他团队成员的专业技能。由于高级开发人员通常会进行代码评审，初级开发人员可能会利用这些反馈来改进他们自己的编码。

C ode 审查等级。

1.拼写错误和样式问题(这是粗略查看代码时首先引起注意的事情)。这一项应该从使用自动化的代码评审过程中排除。

2.名称、函数、类、文件夹和文件的问题。

3.项目中代码的位置，即该函数是否位于所需的文件、类中。该类是否位于项目中正确位置的正确文件夹中。

4.以及这个代码是否有必要。也许这个功能已经实现了，只是人们不知道它的存在。使用一些解决这个问题的知名库可能比编写这个新代码好得多。

5.算法和方法。从原则上来说，你的方法适合这个系统。批评选择的算法或方法。他们试图更全面地了解正在发生的事情。

在代码审查期间，您应该注意的缺点列表:

*   **没有成功的证明**。自动化脚本需要成功运行才能通过审查，并且成功的证明(例如日志或屏幕截图)必须附在审查中。
*   **错别字和格式错误**。错别字和糟糕的格式反映了粗心，造成挫折，并损害声誉。它们对于行为驱动的开发框架来说尤其糟糕。
*   **硬编码值**。硬编码的值通常表示仓促的开发。有时，它们不是大问题，但是它们会削弱自动化代码库的灵活性。
*   **不正确的测试覆盖率**。令人惊讶的是，自动化脚本并没有真正覆盖预期的测试步骤。测试过程中的一个步骤可能会丢失，或者断言可能会产生误报。有时，断言甚至可能不会被执行！当回顾测试时，将原始的测试过程放在手边，并注意遗漏的覆盖面。
*   **不明评论。大多数评论者会给作者留下不清楚的意见，让他做出必要的修改。有必要提出明确的意见，为对话创造空间。**
*   **糟糕的代码布局**。自动化项目趋向于快速增长。随着新测试的出现，页面对象和数据模型等新的共享代码一直在增加。维护一个良好的、有组织的结构对于项目的可伸缩性和团队合作是必要的。测试用例应该按照功能区域来组织。公共代码应该从测试用例中抽象出来，放入共享库中。用于输入和日志记录的框架级代码应该与测试级代码分开。如果代码放在错误的地方，就很难找到或重用。这也可能造成依赖噩梦。例如，非 web 测试不应该依赖 Selenium WebDriver。确保新代码放在正确的位置。
*   **错误的配置更改**。一般来说，在单独的代码评审中提交任何配置变更，并向评审者提供为什么需要变更的详细解释。
*   **脆性**。下面是一些在回顾中要注意的脆性的例子:

1.  测试用例有足够的清理程序吗，即使在崩溃的时候？
2.  是否所有的异常都得到了适当的处理，甚至是意外的异常？
3.  Selenium WebDriver 总是被处理掉吗？
4.  如果断开，SSH 连接会自动重新连接吗？
5.  XPaths 是过于宽松还是过于严格？
6.  一个 REST API 响应代码 201 就和 200 一样好吗？

*   **忽略设计**。许多评审者忽略了整体设计，只关注自动化脚本的功能和可用性。但是这样做，他们错过了自动化脚本在整个系统中的角色，以及它如何适应已经存在的东西。
*   重复是测试自动化的最大问题。许多测试操作本质上是重复性的。工程师有时只是复制粘贴代码块，而不是寻找现有的方法或添加新的助手，以节省开发时间。此外，在大型代码库中很难找到满足即时需求的可重用部分。然而，好的代码评审应该发现代码冗余并提出更好的解决方案。

# 结论

在测试和质量保证过程中，代码审查阶段被严重低估了。如果你是一个在迭代开发框架内工作的测试人员，我绝对相信在熟悉需求之后，熟悉拉请求应该是你工作中不可或缺的一部分。另一方面，您的代码也应该被其他团队成员查看，这对于我个人来说是掌握测试自动化的一个关键因素。

![](img/231a99b48e540fa32e0c24d269f4d4da.png)

[https://test-engineer.site/](https://test-engineer.site/)

# 作者[安东·斯米尔诺夫](https://www.linkedin.com/in/vaskocuturilo/)