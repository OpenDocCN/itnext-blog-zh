# 如何在 Dart 中使用流(第 1 部分)

> 原文：<https://itnext.io/how-to-use-streams-in-dart-part-1-4503fec0cdd7?source=collection_archive---------1----------------------->

## 利用 Dart 类处理流数据

![](img/9c9993fe7391b7838eb070ef44a5e400.png)

*照片由* [*杰罗姆 Prax*](https://unsplash.com/photos/U_m-mPOZzMI?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) *上*[*Unsplash*](https://unsplash.com/search/photos/stream?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

对于大多数钻研 Dart ( *或任何其他语言)的程序员来说，流的概念已经被证明是一个很难理解的话题，部分原因是它需要通过几次尝试和例子来掌握。在本文中，我将试图揭开 Dart 中流的使用的神秘面纱，同时用我们在本系列中进一步学到的知识构建一些有形的东西。*

# 什么是流？

查看 Dart 文档，其定义为:

> 异步数据事件的来源。流提供了一种接收事件序列的方式。每个事件要么是一个数据事件，也称为流的一个元素，要么是一个错误事件，这是一个通知，表明某些事情已经失败。当一个流发出它的所有事件时，一个单独的“done”事件将通知侦听器已经到达结尾。
> 
> **api.dartlang.org**

作为一个概念，流指的是*通道*，数据通过该通道从 A 点流向 b 点。在该通道中，我们能够对到达 b 点之前“读入”的数据执行各种转换。当以*块*的形式传输数据而不是一次传输全部数据时，该*通道*非常有用。

在 Dart 中使用流的方式是通过 SDK 提供的一组助手类。这些助手类提供了实用方法，将数据推送到流中，并通知流的侦听器捕获任何添加的数据。

代表流的最通用的类叫做`Stream<T>`。通常我们不直接使用这个类，因为它被 Dart 库中的其他类暴露了。将此视为与数据流经的*通道*交互的接口。

# 流控制器的基本示例

一个`StreamController<T>`包含一个流，它允许消费者向它发送数据、完成和错误事件。我们将通过执行`streamController.stream`来访问这个流，允许我们调用在它的[文档](https://api.dartlang.org/stable/2.1.1/dart-async/Stream-class.html)中定义的任何方法。

这里有一个关于`StreamController<T>`类的例子:

```
var streamController = **StreamController**();// Accessing the stream and listening for data event
streamController.**stream**.**listen**((**data**) {
  print('Got eem! $**data**');
});
```

上面的代码片段允许我们观察流*通道*中传入的数据块。然后，我们通过将数据打印到控制台来响应这些数据。

所以我猜接下来的**问题**是:*我们如何触发数据监听器事件？* **答案:** *通过给流喂数据！这是通过另一个名为`EventSink<T>`的类实现的。这个对象包含一个`add()`方法，用于向流提供数据:*

```
streamController.**sink**.**add**('Added this string');// Result
// Got eem! Added this string
```

流上的`listen()`方法也可以捕捉错误消息。这是因为每当您侦听流时，都会生成一个`StreamSubscription<T>`对象。这个对象是**能够处理各种事件**的原因，例如数据、错误和完成(当在流上调用 close()方法时*)。*

下面是`listen()`方法的完整定义:

```
**StreamSubscription<T>** listen (
  void **onData**(**T** event), 
  {
    Function **onError**,
    void **onDone**(), // Invoked when the stream is closed
    bool **cancelOnError** // Kills the stream when an error occurs
  });
```

下面是我们如何调用“错误”和“完成”事件:

```
streamController.**sink**.**addError**('Houston, we have a problem!'); // Got an error! Houston, we have a problem!streamController.**sink**.**close**(); // Mission complete!
```

→ [**在镖靶上试试这个**](https://dartpad.dartlang.org/3baf3a9c229dcfa962878905e478a1a7)

# 通过库公开的流

虽然`StreamController<T>`允许我们对自己实例化的流进行细粒度的控制，但是有内置的 dart 库在幕后使用流。例如，看看这个设置服务器的代码片段:

```
import 'dart:io';void main() async {
  var **server** = await **HttpServer.bind**('localhost', 8080); // HttpServer exposes a Stream<T> interface
  **server**.**listen**((HttpRequest **request**) {
    request.response.write('Hello, World!');
    request.response.close();
  });
}
```

上面的代码片段实例化了一个用于创建 web 服务器的`HttpServer`。这个类公开了一个`Stream<T>`接口，这意味着我们现在可以监听这个流，它将包含当用户访问我们的服务器时产生的请求对象。

下面是 web 浏览器中显示的另一个流示例:

```
import 'dart:html';void main() {
  var **button** = querySelector('button');

  // `onClick` is a Stream<T> instance that receives user click data events
  button.**onClick**.**listen**((_) => print('Button clicked!'));
}
```

在浏览器中发生的用户交互，如*点击*、*滚动*、*输入*等，被作为流中捕获的“数据”事件发出。换句话说，HTML 元素也公开了一个`Stream<T>`接口，用于处理页面上的用户交互。

还有很多使用流的类，这里的要点是，通常你不会直接实例化`Stream<T>`对象，而是通过 SDK 中的各种库类为你实例化。

# 结论

流提供了一种处理大块数据的强大方式。因为这是以异步方式运行的，所以我们获得了以非阻塞方式运行代码的好处。我建议通读文档，特别是包含异步编程类的 **dart:async** 库，比如 Streams 和 Futures。

在本系列的下一部分中，我们将研究如何在流上执行转换，以及演示一个以使用流🧱为中心的通用设计模式

# 继续阅读

→ [**如何在 Dart 中使用 Streams(第二部)**](https://creativebracket.com/how-to-use-streams-in-dart-2/)

# 进一步阅读

*   [dart:异步库文档](https://api.dartlang.org/stable/2.1.1/dart-async/dart-async-library.html)
*   [**egghead . io 上免费飞镖课**](https://egghead.io/instructors/jermaine-oppong)

# 分享是关怀🤗

如果你喜欢读这篇文章，请通过各种社交渠道分享。此外，检查并 [**订阅我的 YouTube 频道**](https://youtube.com/c/CreativeBracket) ( *也点击铃铛图标*)观看 Dart 上的视频。

[**订阅我的电子邮件简讯**](http://eepurl.com/gipQBX) 下载我的免费**Dart 入门电子书**并在新内容发布时得到通知。

**喜欢，分享一下** [**跟着我**](https://twitter.com/creativ_bracket) 😍有关 Dart 的更多内容。