# 解释:Rust 在 Web 开发中的未来

> 原文：<https://itnext.io/explained-rust-futures-for-web-development-b1d0632490e7?source=collection_archive---------1----------------------->

## *正如我在上一篇文章* *中解释的那样，期货是异步环境的一部分。我们需要语法、类型和运行时来以非阻塞的方式运行代码。期货是 Rust 异步故事的典型部分。*

# 两种观点

如果你来自 NodeJS，Rust 的未来没有多大意义。在 NodeJS 中，一切都是异步发生的。因此，为了让你能够说“嘿，我真的需要等待这个 GET HTTP 调用的答案”，你把`.then()`放在了`Promise`上，这样你就可以确保当 HTTP 调用结束时，你只是执行了`.then()`中的代码。

在 Rust 中，默认情况下，一切都是阻塞和同步的，所以你可能会问自己:“为什么要为复杂性而烦恼，这正是我最初想要的！”

Rust 是一种系统编程语言。因此，要在 Rust 中编写应用程序，您必须始终身兼两职。“系统 Hat"(⛑”和“程序员帽”(🎩).系统 Hat(⛑)让你从机器的角度考虑什么是最好的🎩)负责软件的语法和编写方式。

如果你来自 NodeJS，系统是由 Googles V8 运行时负责的，所以你可以专注于语法。在 Rust 中，你可以从 crates 那里得到帮助，尽管你需要自己做出某些决定。

这就是我们为什么要使用期货的原因。因此，您需要在应用程序中将 Futures 作为一种类型来处理，然后确保使用运行时来实际执行它们。如果你在消费期货(例如当你使用的一个箱子返回一个`Future`)，你必须理解数据来自哪里。在这种情况下，需要程序员帽**和**系统帽。

# `Future`究竟是什么？🎩

Rust 中的一个`Future`实际上是一个`trait`，如果你想实现它，看起来是这样的:

```
trait Future {
    type Item;
    type Error; fn poll(&mut self) -> Poll<Self::Item, Self::Error>;
}
```

`poll`方法很重要。这将从你的运行时得到调用，并将返回`[Async::Ready](https://docs.rs/futures/0.1/futures/enum.Async.html)` [或](https://docs.rs/futures/0.1/futures/enum.Async.html) `[Async::NotReady](https://docs.rs/futures/0.1/futures/enum.Async.html)`。

如果您想从远程位置或文件系统获取不同的杂志期，您可以为此创建并返回自己的`Future`:

```
struct Magazine {
    issues: Vec<u8>
}
```

还有`impl`上面的`Future`特质:

```
impl Future for Magazine {
    // here we return a single byte
    type Item = u8;
    type Error = io::Error; // this method is getting called from the runtime. Everytime we can read
    // a byte into the buffer, we return `Async::Ready`
    fn poll(&mut self) -> Poll<Self::Item, Self::Error> {
        let mut buffer = [0;1];
        match self.0.poll_read(&mut buf) {
            Ok(Async::Ready(_num_bytes_read)) => Ok(Async::Ready(buffer[0])),
            Ok(Async::NotReady) => Ok(Async::NotReady),
            Err(e) => Err(e)
        }
    }
}
```

运行时会用你请求的信息填充缓冲区，一旦缓冲区满了，`Future`就会返回`Async::Ready`。

这就是铁锈中的一个`Future`的基本精髓。

# 什么时候使用期货？🎩

> *为一个类型实现* `*Future*` *trait 是你告诉机器“嘿，这可能要花点时间，在执行这部分代码时考虑一下”的方式。*

Rust 是一种同步语言。所有的事情都是一行一行地发生，并且一直等到每个结果都被处理完。这不是一个问题，除非你想做需要更长时间处理的任务。

如果你用 Rust 创建一个微服务，你需要为一些数据获取另外三个微服务，然后你合并数据并把它写入数据库，这是一个你想考虑使用 Futures 的用例。

# 为什么使用期货可以节省时间？⛑

在我们的测试场景中，比方说，每个 HTTP 调用可能需要 1 分钟。现在，不是等待 3 分钟来完成所有的调用，而是希望并行运行它们，这样您仍然只需等待 1 分钟而不是 3 分钟。

因此，您创建了三个方法，每个方法返回一个`Future`，收集它们，然后将它们传递给运行时(例如，通过`tokio::run`传递给 tokio)。

# 期货怎么用？🎩 + ⛑

如果板条箱返回一个`Future`，一旦`Future`准备好，您可以使用`[.and_then()](https://docs.rs/futures/0.1.26/futures/future/struct.AndThen.html)`处理结果。在`Future`上使用`[.map()](https://docs.rs/futures/0.1.26/futures/future/struct.Map.html)`可以改变`Future`的类型。

在异步 Rust 世界中，我们必须处理不同类型的数据。例如，我们不是使用字符串和数字，而是处理值流。您很可能会处理流。

一个`Stream`是一个未来的延伸。未来是关于产生一个单一的值，而`Stream`是产生值，只要它们存在。

*   [流](https://docs.rs/futures/0.1.26/futures/stream/index.html):一个`Stream`就像一个`Future`一样，是一个你可以在一个类型上`impl`的特征。这允许您迭代返回值(即`Some(_)`或`None`)。
*   [接收器](https://docs.rs/futures/0.1.26/futures/sink/index.html):用于连续(以异步方式)将数据写入套接字或文件系统

所以流是用来读数据的，接收器是用来写数据的。在我们的网络生态系统中有两种类型的流:

*   字节流(如 HTTP 主体或 TCP 流)
*   消息流(如 WebSocket 帧或 UDP 数据包)，其中每个消息都有固定的大小

# 代码示例

让我们看一个用例，您正在使用的板条箱返回一个`Future`。在做 HTTP 调用的时候，`reqwest`就是一个很好的例子。在从`reqwest`返回的`Future`上，您可以使用`.and_then()`来处理结果(🎩):

```
// We have to use "r#" before "async" because "async" is a reserved keyword.
use reqwest::r#async::Client;// The return type has to be `Future<Item=(), Error=()>` to be able
// to use `tokio::run`. 
// If it has a different type, you have to use `tokio::block_on`
fn fetch_data() -> impl Future<Item=(), Error=()> {
    Client::new()
        .get(url)
        .send()
        .and_then(|res| {
            res.into_body().concat2()
        })
        .map_err(|err| println!("request error: {}", err))
        .map(|body| {
            // here you can use the body to write it to a file
            // or return it via Ok()
            // If you return it via for example Ok(users)
            // then you need to adjust the return type in impl Future<Item=TYPE
            // Examples can be found here: 
            // https://github.com/gruberb/futures_playground // For now, lets just turn the body into a Vector
            let v = body.to_vec();
        })
}
```

一旦你创建了返回一个`Future` ( `fetch_data()`)，**的方法，你必须把它传递给一个运行时**，比如 tokio (⛑):

```
tokio::run(fetch_data());
```

# 高级概述

1.  您从外部板条箱收到一个`Future`
2.  一个`Future`很可能返回一个`Stream`值，所以你必须将这个`Stream`形成一个你可以同步处理的类型(比如一个向量或者字符串)
3.  您通过`-> impl Future<Item=(), Error=()>`从一个方法中返回整个未来，其中大括号`()`是您想要返回的实际类型的占位符
4.  您通过`tokio::run(method())`将方法传递给运行时，如 tokio
5.  `run`调用将启动运行时，设置所需的资源，然后将这个未来放在一个线程池中，并开始轮询您的未来
6.  然后，它会尝试将工作传递给操作系统
7.  每次运行时轮询你的`Future`，如果你的`Future`正在等待的底层 I/O 资源没有准备好，它将返回`NotReady​`。运行时看到这个`NotReady​`返回值并让你的`Future`休眠
8.  一旦来自底层 I/O 资源的事件到来，运行时将检查该 I/O 资源是否与您的`Future`相关联，并再次开始轮询。这一次，您的`Future`将能够返回一个带有值的`Ready`，因为底层 I/O 资源已经提供了一个值
9.  然后，运行时会将`Future`的状态设置为就绪，并处理代码的`.and_then()`部分

与 NodeJS 不同的是，`Future`是通过`tokio::run`而不是之前执行的。在 Node 中，只要你写了一个`Promise`，对象就会被立即返回。

# 期货有什么不一样或者难的？⛑

让我们看一下上面的例子:

*   我们用我们的请求`Client::new()`和`.send()`创建一个新的客户端
*   我们会得到一个`[Response](https://github.com/seanmonstar/reqwest/blob/master/src/async_impl/response.rs#L24)`回来:

```
pub struct Response {
    status: StatusCode,
    headers: HeaderMap,
    url: Box<Url>,
    body: Decoder,
    ...
}
```

*   机身本身就是一个解码器，可以通过`[.into_body()](https://github.com/seanmonstar/reqwest/blob/master/src/async_impl/response.rs#L113)`变成一个`Body`。
*   `Body`本身[实现](https://github.com/seanmonstar/reqwest/blob/master/src/async_impl/decoder.rs#L155)一个[流](https://docs.rs/futures/0.1.26/futures/stream/trait.Stream.html)(如前所述)。
*   现在我们可以从 Rust 的 Futures API 中找到答案:我们可以通过`[.concat2()](https://docs.rs/futures/0.1.26/futures/stream/struct.Concat2.html)`将一个字节流转换成一个条目

```
...
        .and_then(|res| {
            res.into_body().concat2()
        })
...
```

*   我们将`.map()`部分中的这一项作为`body`
*   在你的代码编辑器的帮助下，你会发现这个`body`实际上是一个`[Chunk](https://github.com/hyperium/hyper/blob/master/src/body/chunk.rs)`，从库`[Hyper](https://github.com/hyperium/hyper)`返回
*   我们现在可以把这个`Chunk`变成一个`[Vector](https://doc.rust-lang.org/std/vec/struct.Vec.html)`

```
...
        .map(|body| {
            let v = body.to_vec();
            // do whatever with v
        })
...
```

从那时起，我们回到了“正常的”铁锈地带，可以忘记刚刚发生的事情🙃。

> *你可以在这个* [*GitHub 资源库*](https://github.com/gruberb/futures_playground) *中找到完整的例子。在那里我接收到一个* `*JSON*` *并把它写到一个文件中。*

这也是为什么一开始处理期货如此艰难的原因之一。你必须考虑比 NodeJS 更低的层次。此外，`async/await`语法[还不是最终的](https://internals.rust-lang.org/t/await-syntax-discussion-summary/9914)，这导致了更多的锅炉代码。

这些心理步骤有助于你在处理期货交易时不迷失方向:

1.  我从这个库中得到的返回类型或值是什么？
2.  我如何访问这个响应的值的`Stream`?
3.  当我通过`.concat2()`收集所有值时，库将把这个`Stream`变成什么？
4.  我如何将这种新类型转换成 Vector 或另一种 Rust std 格式，以便可以将其传递给同步方法？

![](img/a1af3570fda438757f4b661f7781cf11.png)

基本上，你总是想弄清楚如何访问值流，收集它们，然后处理结果对象。

# 如何执行多个`Future`？🎩

一般来说，你想把你的值收集成`Streams`，这样对于你从`Stream`中得到的每一项，你可以派生出一个新的`Future`来处理它。

Rust Futures API 有一个名为`[FuturesUnordered](https://docs.rs/futures/0.1.26/futures/stream/futures_unordered/struct.FuturesUnordered.html)`的方法，您可以使用它来添加多个`Future`:

```
use futures::stream::futures_unordered::FuturesUnordered;
use hyper::{client::ResponseFuture, Client};fn setup_requests() -> FuturesUnordered<ResponseFuture> {
    let mut list_of_futures = FuturesUnordered::new(); let client = Client::new(); let first = client.get(URL);
    list_of_futures.push(first); let second = client.get(URL);
    list_of_futures.push(second); list_of_futures
}
```

在这个例子中，我们使用`[hyper](https://github.com/hyperium/hyper)`进行 HTTP 调用。你可以在 Github 上找到剩下的代码[。](https://github.com/chicagohaskell/async-futures-talk/blob/master/rustlb/examples/multi-http/src/main.rs)

如果您使用的是`reqwest`，语法看起来会略有不同。这里你`.join()`多个请求，并作为“一个未来”返回。

```
fn fetch() -> impl Future<Item=(), Error=()> {
    let client = Client::new(); let json = |mut res : Response | {
        res.json::<STRUCT_TYPE>()
    }; let request1 =
        client
            .get(URL)
            .send()
            .and_then(json); let request2 =
        client
            .get(URL)
            .send()
            .and_then(json); request1.join(request2)
        .map(|(res1, res2)|{
            println!("{:?}", res1);
            println!("{:?}", res2);
        })
        .map_err(|err| {
            println!("stdout error: {}", err);
        })
}
```

完整代码也可以在 [GitHub](https://github.com/seanmonstar/reqwest/blob/master/examples/async_multiple_requests.rs) 上找到。

# 期货的未来如何？🎩 + ⛑

期货价格稳定在 1.37，大约在 6 月份。语法和运行时也有变化，这将有助于减少您必须编写的代码量，以便将这个值流从`Future`中取出并转换成同步 Rust 格式。

您还可以使用`[Runtime](https://github.com/rustasync/runtime)` crate，它可以为您节省几乎所有的样板代码。尽管经历上述过程有助于你更深层次地理解未来。

# 摘要

如果您执行异步操作，比如从操作系统获取文件或向远程服务器发出 HTTP 请求，那么 Futures 允许您以非阻塞的方式处理返回值。

如果它是同步的，您将不得不阻塞正在运行操作的线程，并等待结果，直到您继续。为了以异步方式实现这一点，我们有一个运行时，它自己创建线程，并承担未来。当操作系统向运行时返回值时，它将填充`Future`中的值。

一旦`Future`完成，运行时设置`Async::Ready`，代码的`.and_then()`部分将被执行。