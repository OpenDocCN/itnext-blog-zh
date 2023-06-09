# 强制关闭 Golang 中的 TCP 连接

> 原文：<https://itnext.io/forcefully-close-tcp-connections-in-golang-e5f5b1b14ce6?source=collection_archive---------0----------------------->

![](img/ba612d4247a57469dd0236e97fc9c42d.png)

何塞·安东尼奥·加列戈·巴斯克斯在 [Unsplash](https://unsplash.com/s/photos/reset?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄的照片

使用 TCP 服务器是熟悉底层网络通信的好方法。当编写直接通过 TCP 套接字通信的应用程序时，需要大量的套接字管理步骤。对于典型的基于 HTTP 的应用程序，您通常不必采取这些操作。

有时，这些额外的管理步骤包括必须强制关闭 TCP 会话。

今天的文章将讨论两种关闭 TCP 会话的方法；传统的默认关闭和使用`SetLinger()`方法的强制关闭。本文还将解释为什么这些方法不同，以及何时应该使用每种方法。

## Go 中的基本 TCP 服务器

我们将在本文中使用的编程语言是 Golang，但是讨论的概念是通用的，适用于整个网络。

下面是 Go 中一个基本的 TCP 服务器。该服务器监听单个端口，回显发送给它的任何数据，当然，为了更好地显示关闭的连接，它将在一段时间后关闭连接。

我们可以将上面的代码分成四个主要部分。

第一个通过开始监听端口`9000`上的 TCP 连接来创建`net.Listener`。在内部，Go 告诉系统内核在所有可用的接口上绑定端口`9000`。

如果成功，代码转到第二部分，程序通过调用`net.Listener.Accept()`方法等待新的 TCP 连接。`Accept()`方法将等待新的连接到达，并将该连接作为`net.Conn`返回。从这里，我们的节目开始第三和第四部分。

在第三部分，我们的代码开始一个新的 goroutine 这个 goroutine 采用被称为`c`的`net.Conn`，并开始读取和写入连接。对于写入这个 TCP 连接的所有内容，这个 goroutine 将写回同一个连接。

第四部分是本文的重点。这里，我们的程序创建了另一个 goroutine，但是这个 goroutine 使用了`time.After`来等待 15 秒。一旦这 15 秒结束，我们的 goroutine 将通过`defer`函数调用`net.Conn.Close()`。实际上，从服务器端关闭我们的 TCP 会话。

## Go 中的一个基本 TCP 客户端

为了连接到我们的 TCP 服务器，我们将使用下面的客户端代码，也是用 Go 编写的。

我们的客户可以分为三个不同的部分。

首先，我们使用`net.Dial()`方法打开一个 TCP 连接到我们的 TCP 服务器正在监听的同一个`localhost:9000`地址。

利用返回的`net.Conn`，第二部分使用`net.Conn.Write()`方法编写我们的示例消息。

最后，第三部分只是简单地循环一个`net.Conn.Read()`方法来连续读取从 TCP 服务器发送的数据。

## 运行我们的客户端和服务器

我们的客户机和服务器已经准备好了，让我们看看运行它们时会发生什么。

下面的输出来自我们的 TCP 服务器。

从上面我们可以看到，我们的服务器看到了一个打开的 TCP 会话，过了一会儿，输出一个错误，指出我们的服务器试图从一个关闭的网络连接中读取。

考虑到我们有一个从 TCP 连接读取的 goroutine，而在另一个 goroutine 中，我们关闭了那个连接，这种行为是可以预料的。

因此，对于本练习，该错误表明我们成功关闭了服务器端的会话。

现在让我们来看看 TCP 客户端的输出。

从这个输出中，我们可以看到我们的 TCP 会话被打开了，不久之后(与来自 TCP 服务器的时间戳相匹配)，我们从连接中收到了一个 **EOF** 错误。同样，这也是我们所期望的。当远程端很好地关闭了我们的 TCP 会话时，我们的客户端收到了一个错误类型(`io.EOF`)。

## 学习`net.Conn.Close()`如何工作

根据 Go 文档，默认情况下，在执行`net.Conn.Close()`方法后；在后台，操作系统将完成所有数据的发送，然后关闭 TCP 会话。

这意味着，当我们执行我们的`net.Conn.Close()`方法时，我们对其执行的 TCP 会话将启动一个连接终止序列，其中包括处理(丢弃)任何未完成的数据。也就是说，直到我们接收到最后的`FIN-ACK`包。

为了更好地了解这种行为，让我们来看一个使用`tcpdump`命令的 TCP 客户端和服务器通信的网络捕获。

在研究网络通信时,`tcpdump`命令是一个方便的工具。它允许我们捕获和查看网络数据包。

在上面的例子中，我使用了`-i`标志将我的网络接口指定为`lo0`我的环回接口。我使用`port 9000`过滤网络捕获，只捕获与端口`9000`通信或来自端口`9000`的流量。

在第`3`行，我们可以看到客户端(源端口`50796`)向端口`9000`(TCP 服务器)发送了一个`SYN`数据包(显示为`Flags [S]`)。在接下来的几行中，我们可以看到服务器发回一个`SYN-ACK`(显示为`Flags [S.]`)，客户端发送一个`ACK`(显示为`Flags [.]`)来确认`SYN-ACK`数据包。然后完成 3 次 TCP 握手。

从这里开始，我们的客户机和服务器来回发送数据，直到第`11`行，在这里我们的 TCP 服务器向 TCP 客户机发送一个`FIN`包。

在接下来的两行中，我们可以看到 TCP 客户端发送了另一个`ACK`，后跟一个`FIN-ACK`。TCP 服务器用确认`FIN-ACK`的`ACK`来回复。

从网络捕获中，我们可以看到一个标准的 TCP 闭包是如何工作的。我们可以看到，即使我们的程序可能已经执行了`net.Conn.Close()`方法，在操作系统的后台，连接仍然是活动的，直到我们接收到最后的`FIN-ACK`包。

## 强制关闭 TCP 会话

虽然上面的例子已经展示了默认行为，但是执行更“强有力”的套接字关闭是可能的。在 Go 中，我们可以使用`net.TCPConn.SetLinger()`方法来控制这种行为。

该方法通过对操作系统的系统调用来改变`SO_LINGER`套接字选项值。

为了展示这一切是如何工作的，让我们对我们最初的 TCP 服务器做一些简单的修改。

在很大程度上，这个版本的 TCP 服务器与以前的相同，只有一个关键的区别。在第`59`行运行`net.Conn.Close()`之前，我们的应用程序正在执行`net.TCPConn.SetLinger()`，将值`0`传递给方法。

我们通过向`net.TCPConn.SetLinger()`方法传递一个值来定义操作系统在关闭 TCP 连接时的行为。

传递的值可以被认为是以秒为单位的计时器值。

在某些操作系统中，当传递大于`0`的任何内容时，这将导致连接以与默认行为相同的方式运行，只是在定义的秒数后，任何未完成的流量都会被拒绝。

当精确设置为`0`时，操作系统将立即关闭连接并丢弃任何未完成的数据包。为了更好地理解发生了什么，让我们使用仍在运行的`tcpdump`命令来看看这个例子。

通过上面的输出，我们可以再次看到通过`SYN`、`SYN-ACK`和`ACK`三次握手建立了一个 TCP 会话。我们也可以在这个截图中看到闭包，但是这一次，闭包过程看起来有点不同。

TCP 服务器没有发送`FIN`数据包，而是向 TCP 客户端发送了`RST`(显示为`Flags [R.]`)数据包。

这是对话中的最后一个数据包；此流中没有确认。这是故意的。

`RST`数据包是一种特殊类型的数据包，用于“重置”TCP 连接。这是发送方告诉远程方它不会接受或接收该连接的新数据的一种方式。TCP 客户端不需要确认这个关闭，因为 TCP 服务器将拒绝所有接收到的数据。

## 摘要

至此，我们已经探索了关闭 TCP 会话的默认过程，并展示了强制关闭 TCP 会话的过程。

我们还深入研究了操作系统中的底层过程，以及这两种方法在网络级别上的不同之处。

然而，我们还没有探究为什么我们应该知道这种区别，以及何时应该使用每个过程。

一般来说，作为一个 TCP 服务器，关闭一个 TCP 会话时，最好使用默认的`net.Conn.Close()`进程，不改变任何`SO_LINGER`选项。这适用于大多数用例，如超时、响应最终消息或任何其他典型行为。

A `RST`应该被保留用于当客户端行为是非典型的或者当所有通信必须完成时。一个例子可能是一个应用程序有许多，许多短期连接。

使用默认方式关闭时，每个连接都会经历一系列连接状态，如`FIN_WAIT_1`、`FIN_WAIT_2`、`TIME_WAIT`等。，使连接在系统上停留一段时间。夺取资源。使用强制关闭会立即关闭这些连接，释放资源，如打开的文件等。

另一个例子是当一个连接对一个保留的或特殊的端口打开时。在这些情况下，通常使用`RST`数据包来表示不希望的行为。

对于大多数应用程序和关闭场景，TCP 客户端应该启动连接关闭。因此，TCP 服务器通常会将`RST`数据包用于需要服务器启动连接关闭的特殊情况。