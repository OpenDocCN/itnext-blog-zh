# 使用 Scala 期货处理利率限制

> 原文：<https://itnext.io/handling-rate-limits-with-scala-futures-8904538bca7f?source=collection_archive---------0----------------------->

## 为了走得更快，在走得快时减速。

![](img/c4b568f668e33b9fe9f879633cfb4245.png)

(来源:[派克斯](https://www.pexels.com/photo/yellow-and-black-road-concrete-barrier-638487/))

当开发/集成外部 API 时，速率限制是生活中不幸的一部分。速率限制在其实现中各不相同，但通常归结为对特定时间窗口内可以发出的请求的数量(以及可能的类型)的某种预定义限制。

在编写具有多个集成点的并发代码时，处理这个问题可能会很困难。你的第一反应可能是[引入重试](https://hackernoon.com/exponential-back-off-with-scala-futures-7426340d0069)；然而，这并不能确保公平性，并导致数据竞争。为了正确处理和遵守速率限制，解决方案必须具有以下特性:

*   请求的串行化(以确保公平性)
*   跟踪配额使用情况
*   检测与超出配额相关的错误
*   计划重试，同时保持序列化
*   支持超时(以防队列变大)
*   跟踪队列长度和估计等待时间

除了第一条，这些都是有意义的。值得澄清的是，这里的序列化只包括请求执行顺序，不包括完成顺序。这意味着请求的并行执行仍然存在。当然，如果不是这样的话，我在这篇文章中使用`Future` s 就没有什么意义了。😉

# 马具

如果我们不能把一堆代码转储到一个 REPL 中去玩，那么展示这些代码就没有意思了。这篇文章中的代码没有任何依赖关系，我在每一节的末尾都放了一个链接，链接到一个要点，这是该点的累积代码，您可以将其粘贴到您的 REPL 中(protip: type `:paste`到 REPL 中以转储大块代码)。

当然，这篇文章只为你自己的 API 调用提供了包装，所以为了确保我们还有东西可以玩，这里有一个模拟 API 调用，我们可以在文章中使用。

我们可以将它复制粘贴到我们的 Scala REPL 中，开始享受乐趣吧！

# 序列化请求

![](img/ce1b77e5c7392f576dd596c72167ad36.png)

(来源:[像素](https://www.pexels.com/photo/blueberry-bowl-breakfast-cereal-216951/))

是时候提出要求了！

> 呃…

开玩笑，无论如何…为了使我们的请求连续，我们需要某种 FIFO 数据结构，我们可以用它来控制我们的请求；所以，一个队列。然而，我们还需要在某个时间跨度内限制当前活动请求的数量(我们的基本速率限制支持)。

让我们从一些基本的会计结构和一个用于总结这一切的类开始:

我们的班级拥有我们开始学习所需的一切。该类的输入是一个限制和一个持续时间。这构成了我们限速行为的基础。在类内部，我们有一些东西需要跟踪。首先我们有`requestQueue`,顾名思义，它是一个等待请求的队列。现在，请求被表示为`RequestQueueItem`,这将在后面解释。

我们还有`requestCount`和`windowStopTime`字段，用于确保对于从`timeFrame`得到的给定时间窗口，我们不会超过我们的限制。

`request`是我们今天编写的 API 中的 star 方法。它相当简单，因为它需要一个产生`Future`的函数。这个提供的函数是用户定义的代码，用于向我们的速率受限 API 发出请求。`request`然后创建一个新的`Promise`对象，并将其`Future`传递回客户端。现在我们的代码可以控制`Future`的执行，并且只在我们需要的时候将结果返回给客户端(而不是直接组合`Future`并返回)。

最后，对`tryRequest`的调用检查是否可以发出请求，并发出请求。这是为了确保当队列有容量时，一个请求被添加到队列中就被触发，而不是使用某种调度代码。

让我们看看 `hasCapacity`是如何工作的:

`hasCapacity`检查我们是否可以在当前时间窗口内提出请求。但是，如果它注意到我们在时间窗口之外，就会创建一个新窗口，并重置我们的请求计数器。

我们初始代码的最后一位是通过`makeRequest`请求的实际执行:

这个有点长，但还是很容易分解。如果我们有一个请求，我们增加请求计数，然后执行用户提供的请求函数。如果这导致失败，我们将请求重新排队，并尝试发出另一个请求。如果我们成功了，那么我们就把结果返回给用户。最后，我们尝试提出另一个请求。我们在未来之外这样做是为了允许并行执行，并确保我们使用分配的限制(例如，高吞吐量+高延迟)。

注意，在失败的情况下，我们必须进行同步，因为我们在`Future`的回调中从不同的上下文中修改队列的状态。

[到目前为止代码的要点。](https://gist.github.com/JohnMurray/b4392a913b8f56c428d57555c1e1cf63)

让我们将它粘贴到 REPL 中，看看会发生什么:

```
scala> val service = new ApiService[String](5, 10.seconds)scala> (1.to(10)).map({ _ => service.request(
  () => MockAPI.simpleService
)}).toListres5: List[scala.concurrent.Future[String]] = List(
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(<not completed>),
  Future(<not completed>),
  Future(<not completed>),
  Future(<not completed>),
  Future(<not completed>)
)
```

因为我们的 MockAPI 即时返回，所以我们可以看到它像预期的那样工作。给定 10 个要执行的项目，我们只取回 5 个。剩下的 5 个必须等待下一个执行窗口。但是，如果我们等待 10 秒钟左右，然后重新检查他们的状态，会怎么样呢？

```
scala> res5res6: List[scala.concurrent.Future[String]] = List(
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(<not completed>),
  Future(<not completed>),
  Future(<not completed>),
  Future(<not completed>),
  Future(<not completed>)
)
```

好吧，那似乎不起作用。问题是，一旦达到容量，一旦新的时间窗口打开，就没有流程再次启动请求。也就是说，除非有人提出另一个要求。这是不理想的。让我们看看如何解决这个问题。

这太酷了。我们从 Java 标准库中提取了一些非常方便的实用程序，允许我们异步调度任务。现在，如果我们在当前时间窗口内用完了容量，我们可以安排一个任务在下一个窗口开始时进行重新检查。如果由于某种原因，我们的能力发生变化，我们可以取消任务。这段代码的一个棘手之处是使用了`@volatile`，这确保了读取的是`recheckFut`的一致版本(因为我们在多线程中更新它)。虽然这确实产生了某种竞争条件，但是排列很容易理解，并且比引入额外的锁更好(在我看来)。

现在，如果我们重新运行前面的示例，等待 10 秒钟，我们应该会看到所有的任务都完成了。

```
scala> (1.to(10)).map({ _ => service.request(
  () => MockAPI.simpleService
)}).toListres5: List[scala.concurrent.Future[String]] = ...
// Wait 10 seconds and then look at res5res5: List[scala.concurrent.Future[String]] = List(
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6))
)
```

现在情况看起来好多了，但仍有一个问题不太明显。如果用户提供的函数失败(出于任何原因)，那么我们假设这是一个与速率限制相关的失败，并对请求重新排队。现实情况是，对于网络上任何种类的 API 请求，速率限制只是我们可能遇到的一小类错误。我们将在下一节进一步探讨这一点。

[到目前为止的代码要点。](https://gist.github.com/JohnMurray/884884fef9a47fe65aecacf27fe0a4e0)

# 检测配额错误

用户需要一种方法来定义什么是速率限制错误，什么不是。此外，用户不应该为了检测速率限制异常而局限于失败的`Future`。这可以很容易地用下面的类型来表示:

```
Either[Throwable, T] => Boolean
```

我们可以保持相同的默认行为，同时将其扩展为可由用户覆盖:

省去打字，`LimitDetector`代表我们的`PartialFunction`。`request`现在获取一个`LimitDetector`并将其放入队列项中以备后用。如果用户没有指定，则使用默认值。默认值最初由类设置(到我们最初的行为)，但是可以使用`withDefaultLimitDetection`在实例级设置。现在更新`makeRequest`功能:

除了对方法进行一些一般性的重组以使其更具可读性(减少嵌套)，主要的变化是在请求完成(`result.onComplete`)时，我们检查成功*和*失败案例以确定是否需要重试。由于这种错误处理是特定于超出速率的错误，我们将考虑正在创建的无限重试循环的预期行为。

[到目前为止代码的要点。](https://gist.github.com/JohnMurray/83a0970b82d2984343609d52491e1b23)

让我们使用默认的极限检测来尝试这个新代码:

```
scala> val service = new ApiService[String](5, 10.seconds)scala> (1.to(10)).map({ _ => service.request(
     |   () => MockAPI.throwingRateLimitedService
     | )()}).toListres3: List[scala.concurrent.Future[String]] = List(
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(<not completed>),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(<not completed>),
  Future(<not completed>),
  Future(<not completed>),
  Future(<not completed>),
  Future(<not completed>),
  Future(<not completed>))
```

我们实际上可以看到，5 个初始请求中有 2 个失败了，并在后台重试。如果我们在几秒钟后查看`res3`的结果，我们可能会看到类似这样的内容:

```
scala> res3
res8: List[scala.concurrent.Future[String]] = List(
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(<not completed>),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(Success(1, 2, 3, 4, 5, 6)),
  Future(<not completed>),
  Future(Success(1, 2, 3, 4, 5, 6)),   
  Future(<not completed>),
  Future(<not completed>),
  Future(<not completed>),
  Future(<not completed>))
```

你可以看到取得了进展，但可能没有你想象的那么多。这是因为失败的请求会在队列的最前面重新排队。如果他们继续失败，那么就不会取得什么进展。由于被重新排队的请求的异步性质，出现了一些无序的进展。最终，所有任务都将完成。

同样，我们可以使用添加的极限检测代码来确保相同的代码工作。

```
(1.to(10)).map({ _ =>
  service.request(() => MockAPI.nonThrowingRateLimitedService) {
    case Left(_)    => false
    case Right(msg) => msg == "RATELIMIT_EXCEEDED"
  }
}).toList
```

我会让你在你的 REPL 里玩这个。

# 支持超时

![](img/490009bf0de2e6d88ad6ddccd332b950.png)

(肯·劳伦斯在 [Unsplash](https://unsplash.com/search/photos/otu-of-time?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄)

无论我们喜欢什么，税率限制都是一个必须解决的问题。有时这意味着您可能想要进行比您的能力更多的 API 调用。虽然适当的排队和重试会有所帮助…

> 什么！我一直在关注你告诉我我们建立的奇特的队列和重试机制不是答案吗？！？(╯ □ )╯︵ ┻━┻

请冷静下来。这些事情*是*重要的，但是当请求继续比您的配额允许的更快时，您可能想要提供某种快速失败机制，以便您的应用程序可以优雅地失败。

当然，另一种选择是忽略这个问题，让请求堆积起来，直到队列变得如此之大，以至于耗尽了内存并使机器崩溃。这仅发生在您的响应时间之后，假设这些 API 调用的结果传播到客户端，粗略地拍摄。

> 天啊，你没必要这么刻薄。我会选择第一个选项。

不错的选择！我们可以简单地向我们的`request`函数添加另一个参数来指定我们希望等待的最大时间:

请注意，我们已经向我们的`RequestQueueItem`对象添加了一个新字段:

`request`方法现在需要等待最长时间(默认为永远)，然后计算一个期限，让我们*开始*执行请求。我们已经陈述了我们的意图，但是如果我们的截止日期到了，我们仍然希望发生一些事情:

*   我们不想浪费资源来执行一个超过截止日期的请求
*   我们应该完成用户的`Future`作为一个超时的，失败的未来
*   我们应该将请求从队列中移除，以避免对象堆积和耗尽内存

为了有效地做到这一点，我们可以创建一个函数来整理我们的队列，并每隔 N 个时间单位调用它。时间单位由客户端配置，其工作原理类似于我们的`windowStopTime`和`timeFrame`值。

有了这些新值，我们可以编写一个简单的清理程序来执行每个`cleanupTimeFram`:

`cleanupRequests`检查是否到了清理队列的时间。如果是，它从队列中删除所有过时的请求，用一个`TimeoutException`完成它们，并重新设置下一次清理的时间。

最后一件要注意的事情是确保我们不会对已经超过时间限制的请求执行工作，但是在被弹出队列之前设法避免了清理功能:

`makeRequest`函数现在会快速丢弃超过截止日期的请求，直到找到有效的请求或者队列为空。

# 结论

我们有了它，一个简单的速率限制服务，围绕 Scala `Future` s 工作。

[https://gist . github . com/John Murray/34 E3 beb 7 F5 eed 4935 f 70 DC 45 b 0256067](https://gist.github.com/JohnMurray/34e3beb7f5eed4935f70dc45b0256067)

出发前，请注意以下几点:

*   这里给出的代码是为了演示的目的。如果你喜欢，可以随意使用它，但是要注意它没有经过很好的测试。还有一些我(个人)会改变的地方，比如时间窗口上使用的 1 秒粒度。
*   上面的解决方案只假设了一个应用程序/机器。我认为这是显而易见的，但是如果你想在分布式环境中使用这样的代码，你需要考虑到这一点。无论是通过配额分区还是某种数据共享(假设您的窗口足够大，可以进行协调)。
*   这段代码中的重试只是围绕速率限制错误。如果你有兴趣了解更多关于 Scala futures 的退休信息，请参见我的另一篇文章[Scala Futures](https://hackernoon.com/exponential-back-off-with-scala-futures-7426340d0069)的指数后退。
*   计算和公开/报告等待时间估计值并将其公开给客户可能是一个有趣的练习(如果您正在寻找一个有趣的玩具项目来玩)。

# 约翰·默里的其他职位

[](https://hackernoon.com/systems-programming-d5917e41353f) [## 系统编程

### 用无术语的探索揭开神秘面纱。

hackernoon.com](https://hackernoon.com/systems-programming-d5917e41353f) [](https://hackernoon.com/im-bored-of-language-x-514be05b59cd) [## 我厌倦了 X 语言

### 我想走得快

hackernoon.com](https://hackernoon.com/im-bored-of-language-x-514be05b59cd)