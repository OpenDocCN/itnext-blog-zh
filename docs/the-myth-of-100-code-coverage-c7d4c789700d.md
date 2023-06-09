# 100%代码覆盖率的神话

> 原文：<https://itnext.io/the-myth-of-100-code-coverage-c7d4c789700d?source=collection_archive---------4----------------------->

## 代码只存在于你对它的测试中

![](img/ddd0c74a4c9949f7f491cfa1507a6bde.png)

图片右上角的两个虚构的无头人[的设计作品，面对着左边的一群士兵。每个士兵都骑在马上。无头人没有头，他们的面部特征在胸前](https://en.wikipedia.org/wiki/Headless_men)

代码覆盖率并没有显示全部情况。这篇文章展示了 100%代码覆盖率的神话是如何不可逆转地破坏你的代码设计的。

我们开始吧。

对数据传输对象的测试增加了代码覆盖率。然而，它们不执行应用程序的任何有意义的业务行为。它们只是技术细节。例如，更改访问器方法的名称更有可能需要更改多个测试，并导致不必要的维护，而不会提供任何商业价值。

[工作代码](http://jsfiddle.net/fagnerbrack/v89bwp2o/)显示了一个名为“purchase books command”的类的“user id”访问器方法的测试示例

在上面的例子中，对`PurchaseBooksCommand`的`userId()`访问器方法的更改破坏了对`PurchaseBooksCommand`和`BookStoreCommandHander`的测试。

[工作代码](http://jsfiddle.net/fagnerbrack/kjo5smxp/)显示了当您将方法“user id”重命名为“client id”时，不止一个测试失败的例子

同样，对 React 组件内部功能的测试增加了代码覆盖率。然而，它们也增加了测试代码和组件内部代码之间的耦合。改变组件 [**的内部结构而不改变行为**](https://medium.com/@fagnerbrack/how-to-refactor-a-public-interface-317ed18d38a3) 很可能也需要改变测试，这是不必要的工作。

[伪代码示例](https://gist.github.com/FagnerMartinsBrack/928ccfacba428f5bc5de0927de6df696)展示了 React 组件的测试。该测试获取“counter”组件的实例，并调用方法“increment”两次。断言验证 React 组件的内部状态是否具有数字为 2 的“count”属性。

> 测试覆盖的代码行数并不是代码覆盖率的有用度量。

对一个使用数据传输对象作为你的应用程序的一个基本特性的参数的类的测试，比如说买书，可能不会增加 DTO 的代码覆盖率。然而，他们确保测试**只执行**属于[业务领域](https://en.wikipedia.org/wiki/Business_domain)的代码。

“练习”意味着测试执行代码并断言一个行为。这是这篇文章中的一个基本术语。如果测试只执行没有断言的代码，它会增加代码覆盖率，但不能保证当代码行为改变时会看到测试失败。

[工作代码](http://jsfiddle.net/fagnerbrack/aupesbzg/)展示了“书店命令处理程序”的一个良好测试的例子该测试验证用户是否可以启动购买图书的过程，而不是命令的属性是否有效。

同样，使用 React 组件的呈现状态的测试可能不会增加内部函数调用的代码覆盖率。然而，针对呈现状态的断言确保了当您[重构](https://medium.com/@fagnerbrack/how-to-refactor-a-public-interface-317ed18d38a3)组件的内部代码时，测试不太可能中断。

[伪代码示例](https://gist.github.com/FagnerMartinsBrack/dc5dca24f08c5692f7990705c9c668a8)展示了一个 React 组件的测试。该测试呈现组件，该组件返回一个 API 来查询内存中呈现的 DOM。该测试查询一个具有类“count button”的元素，并断言具有类“current-count”的元素具有正确的计数。

> 测试覆盖的有意义行为的数量是代码覆盖率的一个更好的度量。

根据[古德哈特定律](https://en.wikipedia.org/wiki/Goodhart%27s_law)，当一个客观的衡量标准成为系统中的一个目标时，人们倾向于利用这个衡量标准来假装他们正在取得更好的结果:

> 当一个度量成为目标时，它就不再是一个好的度量
> 
> — [古德哈特定律](https://en.wikipedia.org/wiki/Goodhart%27s_law)

如果您使用代码行作为测试覆盖率的客观度量，人们倾向于玩游戏，并试图编写最佳代码来满足该度量。代码的结局比系统根本没有任何测试更糟糕。

那么，如何快速验证所有的测试是否覆盖了正确的内容呢？我希望有一个非黑即白的回应。我希望有一个神奇的工具，你可以扔向代码，它会突出**所有的**测试问题。

可惜没有。

验证应用程序行为的代码覆盖率的最接近的方法是[突变测试](https://rachelcarmena.github.io/2017/09/01/do-we-have-a-good-safety-net-to-change-this-legacy-code.html)。然而，这并不能突出 100%的测试问题。

测试覆盖率很大程度上依赖于应用程序的设计，而应用程序的设计很大程度上依赖于你试图解决的问题的类型。

你的设计是为了让人类理解代码，你的测试是为了让人类不会搞砸。机器没有这些问题，它们总是正确地执行代码，从不出错。因此，没有任何[静态分析](https://en.wikipedia.org/wiki/Static_program_analysis)工具可以理解你的设计；只有具有软件设计技能的人脑才能查看代码，理解设计，并判断测试是否覆盖了正确的内容。

如果你只是为了优化机器的覆盖率而编写测试，你最终会得到只有机器才能理解的无用的不可读的代码。

> 业务行为的 100%测试覆盖率和代码行的 100%测试覆盖率之间有很大的区别。

有意义的代码覆盖的一个很好的例子是当你设计你的模型来[隔离 HTTP 请求的副作用](https://medium.com/@fagnerbrack/how-to-mock-the-network-with-nock-and-javascript-to-test-parts-of-a-system-you-need-a-good-design-9432c9c05a30)。在这种情况下，您需要以测试代码和应用程序代码之间耦合较少的方式来设计代码:

[上一篇](https://gist.github.com/FagnerMartinsBrack/d7f6e000d3feec3b7019c7f156029892)[文章](https://medium.com/@fagnerbrack/how-to-mock-the-network-with-nock-and-javascript-to-test-parts-of-a-system-you-need-a-good-design-9432c9c05a30)中的伪代码示例，它将“get 请求”注入“HTTP 服务器数据源”测试断言来自“HTTP 服务器数据源”的方法“find posts title”返回正确的结果。

另一个有意义的代码覆盖率的例子是当[你设计你的模型来减少嵌套导入](https://medium.com/@fagnerbrack/the-test-should-verify-if-the-code-solves-the-problem-not-if-it-runs-afea37a3a6e)的数量。在这种情况下，您需要以这样一种方式设计代码，使得应用程序的组件之间的直接耦合更少。最好将依赖项作为函数参数或类构造函数来传递，这样就可以测试模型，而不需要模拟导入。

如果测试模拟了应用程序如何使用被测代码，并且您在测试代码中模拟了导入，那么您应该在应用程序代码中做同样的事情。如果这听起来很荒谬，因为没有人在应用程序代码中嘲笑导入，那是因为它确实如此！嘲笑导入是一种[糟糕的代码味道](https://medium.com/@fagnerbrack/code-smell-92ebb99a62d0)，API 的可插拔性不足以让应用程序**消费它**，这是一个你应该重新设计的暗示:

[上一篇](https://gist.github.com/FagnerMartinsBrack/fc7721773c4336c1cc7bbb40599c0da1)[帖子](https://medium.com/@fagnerbrack/the-test-should-verify-if-the-code-solves-the-problem-not-if-it-runs-afea37a3a6e)中的伪代码示例，展示了一个大写字母“Prefill”的函数，它接收两个参数，一个“存根提供者”和一个“映射逻辑”实例。“映射逻辑”接收一个“用于简单匹配”的参数该代码将“prefill”调用的结果以小写形式存储在一个名为“Prefill”的变量中。代码将“prefill”变量作为一个函数调用，并将“表单域”作为唯一的参数。代码将结果存储在一个名为“预填充表单域”的变量中断言测试“预填充表单字段”的字符串表示是否正确。

在测试环境中，**客户端**是任何一段使用你的模型的代码。测试是模型的“第一个客户”,模拟应用程序如何使用它。应用程序是“第二个客户端”

代码是一段文本，只有当机器执行它时才变得有用。没有一个**客户端**机器可以证明它的有用性，代码就只是一堆没有目的的文本。对于所有实际手段来说，它是不存在的。执行代码的机器应该运行在你的计算机上，而不是生产环境中。这允许快速和早期的反馈。

> 问题不在于 100%的代码覆盖率。废话百分百是。

适当的 100%覆盖意味着实现 100%的业务用例。这是为了涵盖你的模型的外部 API，而不是像[数据传输对象](https://en.wikipedia.org/wiki/Data_transfer_object)的方法或 React 组件内部的技术细节。

尽管 100%的业务需求测试覆盖率是重要的度量标准，但是没有办法通过静态分析工具来检索这些信息。代码覆盖率是让人类控制代码，而不是机器。

优秀的设计技巧和解决问题而不是技术细节的心态是迈向有意义的测试覆盖率的第一步。

毕竟只有有了好的设计，才能做到 100%。

![](img/2b0b11a1cf62c107626e391585222e1f.png)

感谢阅读。如果您有任何反馈，请通过 [Twitter](https://twitter.com/FagnerBrack) 、[脸书](https://www.facebook.com/fagner.brack)或 [Github](http://github.com/FagnerMartinsBrack) 联系我。

感谢[雷切尔·m·卡梅纳](https://twitter.com/bberrycarmen)和[杰夫·格里格](https://twitter.com/JeffGrigg1) 对这篇文章的深刻见解。