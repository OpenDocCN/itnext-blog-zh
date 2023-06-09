# Swift 中的依赖注入变得简单！

> 原文：<https://itnext.io/property-wrappers-and-dependencies-in-swift-59799d08c2fc?source=collection_archive---------2----------------------->

![](img/463fa0971cd6a797c9f3128bccd0df29.png)

在 [Unsplash](https://unsplash.com/s/photos/code-swift-dependency?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上由 [Hitesh Choudhary](https://unsplash.com/@hiteshchoudhary?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 拍摄的照片

大家好，由于工作繁忙，我已经离开一段时间了。

今天我想写一些关于 Swift 5.1 中一个令人兴奋的“新”特性的东西，这个特性叫做[属性包装器](https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md)。有了这个“新”特性，我们可以使用一些花哨的 DSL 构建器来创建一个小的(140 行)依赖注入库。

在这里的 [Pinch](https://pinch.nl/en/) ，我们使用构造注入来满足我们所有的依赖；如果我们有很多属性需要传递，这就变得很乏味了。我们尝试了一些开源框架；不知何故，我们总是回到构造函数注入。

随着这一新功能在 Swift 中的实施，我想尝试一下我的实施。下面的片段是我想要实现的最终目标。确保代码在一个文件中；这使得我们能够在库中隐藏一些实现逻辑。

我们使用一个定制的包装器将我们的依赖注入到我们的 ViewController 中。应该自动解决这些约束。我们可以从创建一个包含所有依赖项的类开始。

我们创建一个字典来保存我们所有的工厂，这样我们就可以有一个查找表来创建我们想要的实例。我们确实需要确保构造函数是私有的；这迫使我们使用我们将在以后构建的 DSL。

当对象取消初始化时，我们确保从内存中删除所有工厂。

随着我们的基础的实现，我们创建了一个包含内部工作的服务。它允许创建实例，并设置“全局”或“一次性”周期；这意味着当我们请求一个实例时，我们总是返回同一个实例，或者每次访问它时创建一个新的实例。

实现了基础之后，我们可以对依赖类进行私有扩展。因此它可以处理注册和解析我们的依赖关系。

这段代码将受益于错误处理❤

在 resolve 方法中，我们基于所请求的服务创建一个 ObjectIdentifier。我们做一些检查，看看我们是否需要一个全局实例，并采取相应的行动。

register 方法将服务(基于对象标识符)保存到字典中。

是时候创建一个公共扩展了。我们创建一个静态主变量来保存我们的静态实例。这是使我们的属性包装器容易解析我们的依赖性所需要的

这段代码将受益于错误处理❤

是时候创建一个公共扩展了。

我们创建一个 fileprivate 静态变量来保存我们的“主”实例；这允许属性包装器快速解析我们的依赖关系。我们创建了一些接受服务的 DSL 方法，并创建了一些方便的初始化器。最后，我们创建一个构建方法，用新的版本替换主实例。

剩下的就是实现我们的属性包装器。

这段代码将受益于错误处理❤

我们在包装器中存储了对已解析依赖项的引用(这确保了当我们访问变量时，它不会每次都被解析为一个新的实例)。

当我们访问变量时，我们首先检查我们是否已经创建了一个实例，否则就返回它。如果这是第一次，我们调用闭包来解决依赖性。

让我们看看我们的实现是否有效。用下面的依赖图想象下面的场景。

当我们运行这个例子时，我们可以看到我们的实现正在工作！

瞧，这就结束了我对 Swift 5.1 中引入的新属性包装器的深入研究。这个例子缺乏基本的错误处理，可能还可以改进，但是应该能让你对如何编写自己的代码有所了解。

你有什么问题或评论吗？请在评论中告诉我们。