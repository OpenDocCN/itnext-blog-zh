# 使用 Haskell 的 AWS Lambda 自定义运行时

> 原文：<https://itnext.io/aws-lambda-custom-runtimes-with-haskell-791a6d3fb9?source=collection_archive---------4----------------------->

![](img/afcd45cb767e47bdf0dc16009808acf5.png)

## 自定义运行时

AWS 在 2018 年 11 月的年度 re:Invent 大会上为我们带来的重大消息是为 Lambda 引入了自定义运行时。这确实很重要，因为它允许我们用任何编程语言编写 Lambda 函数，而不需要将二进制代码作为已经支持的运行时(node、python、go 等)的子进程来运行。).有一个关于如何在 bash 中编写 runtime/lambda 的官方教程，以及一些现成的 C++和 Rust 的运行时。

让我们重复一下运行时实际上做了什么，以及它如何适应像 Haskell 这样的编译语言。bash 教程以及可能所有动态/解释语言的运行时都利用了这样一个事实，即运行时可以加载并运行处理程序的代码。虽然这很好，因为开发人员可以专注于只编写处理程序代码，并让运行时在某个地方，但这可能会导致冷启动的一部分。我想大部分时间花在了实例化所有需要的基础设施上，但也有一部分时间花在了启动和运行运行时上，尤其是如果处理程序本身使用了一些动态加载的库。

Rust 和 C++的官方运行时采用的方法是将整个运行时放入一个库中，并将其与处理程序一起编译成一个二进制文件。

对于已经有一些编写 lambdas 的经验的人来说，这可能有点令人困惑，但这是完全有意义的。在这种方法中，我们只需将运行 lambda 所需的所有东西打包到二进制文件中(实际上是所有东西，但我们将在后面探讨)。

因此，让我们看看是否可以一步一步地用 Haskell 创建一个可以在 lambda 上本地运行的 Lambda 运行时🙂

## 环境设置

为了解决这个问题，让我们创建一个本地开发环境。为了运行我们想要的一切，我们需要:

- `stack`(可从[此处](https://docs.haskellstack.org/en/stable/README/#how-to-install) )
- `docker`
- `aws` cli 工具(仅当我们实际想要在 Lambda 上运行代码时)

第一步是创建一个堆栈项目并对其进行配置。

```
*$ stack new hs-lambda
$ cd hs-lambda
$ stack config set resolver lts-12.14*
```

现在进入适当的拉姆达的东西。我们将从执行环境开始，它是:

*   操作系统—亚马逊 Linux
*   AMI-amzn-AMI-hvm-2017 . 03 . 1 . 2017 08 12-x86 _ 64-GP2
*   Linux 内核—4 . 14 . 77–70.59 . amzn 1 . x86 _ 64
*   用于 JavaScript 的 AWS SDK—2 . 290 . 0
*   Python 的 SDK(Boto 3)-3–1 . 7 . 74 Boto core-1 . 10 . 74

对我们来说，重要的是我们编译的代码将在**Amazon Linux v 2017 . 03 . 1 . 2017 08 12**上运行，因为我们希望它成为我们的编译环境，以实现最大的二进制兼容性。

因此，让我们基于 AWS 映像构建一个容器:

```
*$ docker run -rm -name $(basename $(pwd)) -v $(pwd):/mnt -ti amazonlinux:2017.03.1.20170812 /bin/bash*
```

这将启动一个 bash 会话，并将我们的当前目录挂载到容器内的`/mnt`目录中。我们也想在这里使用`stack`,所以让我们安装它和一些依赖项:

```
*bash-4.2# cd /mnt
bash-4.2# curl -sSL* [*https://get.haskellstack.org/*](https://get.haskellstack.org/) *| sh
bash-4.2# yum install -y gcc make xz libffi zlib zlib-devel gmp gmp-devel — suggested by stack install script*
```

到目前为止，我们已经部分建立了编译和开发环境。现在，让我们完成开发环境的设置。我们将只使用我们的容器进行最终编译，并在我们的 dev box 上做其他事情。让容器保持运行，打开另一个终端窗口，将`cd`指向我们的`hs-lambda`目录。

## 履行

`stack new`创建了一个基本的项目框架，我们准备写一些代码。让我们打开`app/Main.hs`文件，回到文档中，看看我们的运行时应该做什么。出于本文的目的，我们将创建一个基本的无服务器 echo 服务器🙂

首先，引用 AWS 文档告诉我们如何让我们的定制运行时工作:

> *你可以用任何编程语言实现 AWS Lambda 运行时。*
> 
> *运行时是在调用函数时运行 Lambda 函数处理程序方法的程序。您可以将运行时以名为 bootstrap 的可执行文件的形式包含在您的函数部署包中。*

从上面我们知道，我们需要一个名为`bootstrap`的二进制文件放在我们的 lambda zip 包中，这个文件应该包含运行时。让我们更进一步，检查我们的运行时必须经历的所有步骤。

1.  **检索设置** —读取环境变量以获取关于功能和环境的详细信息。

*   `_HANDLER` -处理器的位置，来自功能配置。标准格式是`file.method`，其中`file`是不带扩展名的文件名，`method`是文件中定义的方法或函数的名称。
*   `LAMBDA_TASK_ROOT` -包含功能代码的目录。
*   `AWS_LAMBDA_RUNTIME_API` -运行时 API 的主机和端口

有关可用变量的完整列表，请参见 Lambda 函数可用的环境变量

2.**初始化函数** —加载处理程序文件并运行它包含的任何全局或静态代码。函数应该一次性创建像 SDK 客户端和数据库连接这样的静态资源，并在多次调用中重用它们。

3.**处理错误** —如果出现错误，调用初始化错误 API 并立即退出。

我们将查看这些要点，看看它们是否都适用。

1.  我们的计划是创建一个带有自定义运行时的基本的最小 lambda 函数，所以我们将把所有东西放入我们的`bootstrap`二进制文件中。在这种情况下，我们不需要加载任何处理程序，所以我们真的不需要读取`_HANDLER`和`LAMBDA_TASK_ROOT`变量。我们唯一需要的是`AWS_LAMBDA_RUNTIME_API`的值。
2.  同样，我们不会加载任何处理程序，所以初始化 SDK 等。可以在我们的`bootstrap`代码中完成。
3.  我们可能应该做一些错误处理，但由于这只是一个玩具例子，我们将跳过它。为了简单起见。

好了，到目前为止，我们只需要读取一个环境变量并运行一些代码来初始化昂贵的东西，如数据库连接、SDK 等。没什么难的。

完成初始化后，我们可以进入处理任务的主循环。像往常一样，让我们后退一步，阅读文档:

1.  **获取事件** —调用下一个调用 API 来获取下一个事件。响应正文包含事件数据。响应头包含请求 ID 和其他信息。
2.  **传播跟踪头** —从 API 响应中的`Lambda-Runtime-Trace-Id`头获取 X 射线跟踪头。为 X-Ray SDK 使用的环境变量设置相同的值。
3.  **创建一个上下文对象** —使用环境变量和 API 响应头中的上下文信息创建一个对象。
4.  **调用函数处理程序** —将事件和上下文对象传递给处理程序。
5.  **处理响应** —调用调用响应 API 来发布来自处理程序的响应。
6.  **处理错误** —如果出现错误，调用调用错误 API。
7.  **清理** —释放未使用的资源，向其他服务发送数据，或者在获取下一个事件之前执行额外的任务。

现在，这很容易。我们只是:

*   向一个端点发出 HTTP 请求并接收一个事件
*   读取一些头文件并设置一些环境变量
*   创建一些额外的上下文
*   处理事件时考虑(或不考虑)上下文
*   生成一个输出，并通过另一个 HTTP 请求发布它，无论它是一个正确的响应还是一个错误(尽管端点不同)
*   必要时清理

关于 API 的一些细节可以在[这里](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-api.html)找到。在我们的例子中，我们制作了一个 echo 服务器，所以我们只需要简单地获取事件并发回它。代码很简单:

## 汇编

除了错误处理(我们跳过它，因为它只是一个玩具)之外，我们几乎拥有了我们需要的一切。现在这还不能编译，因为我们使用了一些外部库。我们需要将它们添加到我们的`package.yaml`文件中。我们需要做的另一个小改变是重命名我们的可执行文件。我们可以稍后手动完成，但是让我们自动完成。`package.yaml`的相关部分现在应该看起来像这样:

现在我们已经准备好在本地编译和运行了(这可能需要一些时间)。

```
*$ stack build
$ stack exec bootstrap
bootstrap: AWS_LAMBDA_RUNTIME_API: getEnv: does not exist (no  environment variable)*
```

有用！我们有一个错误，我们丢失了一个环境变量，但现在没问题。让我们尝试在我们的容器中编译它(这也可能需要一段时间)。

```
*bash-4.2# stack setup
bash-4.2# stack build
bash-4.2# ldd .stack-work/install/x86_64-linux/lts-12.14/8.4.3/bin/bootstrap
 linux-vdso.so.1 => (0x00007ffe14dae000)
 libm.so.6 => /lib64/libm.so.6 (0x00007fca5f2f1000)
 libpthread.so.0 => /lib64/libpthread.so.0 (0x00007fca5f0d5000)
 libz.so.1 => /lib64/libz.so.1 (0x00007fca5eebf000)
 librt.so.1 => /lib64/librt.so.1 (0x00007fca5ecb7000)
 libutil.so.1 => /lib64/libutil.so.1 (0x00007fca5eab4000)
 libdl.so.2 => /lib64/libdl.so.2 (0x00007fca5e8b0000)
 libgmp.so.10 => /usr/lib64/libgmp.so.10 (0x00007fca5e63a000)
 libc.so.6 => /lib64/libc.so.6 (0x00007fca5e26d000)
 /lib64/ld-linux-x86–64.so.2 (0x00007fca5f5f3000)*
```

一切正常，但是我们的二进制文件有一个问题,`ldd`命令强调了这个问题。我们的二进制是动态链接的。这很好，因为它降低了文件大小，但在我们无法控制的系统上，不能安装任何库或任何东西，这是一个问题。还记得我们如何安装`zlib`和`gmp`吗？在 AWS 上运行我们的 lambda 的 VM 上将缺少这两个。有两种选择。我们可以捆绑缺失的库，将我们的二进制文件与那些库链接起来，或者编译一个静态链接的二进制文件。虽然第二种方法会产生一个更大的文件，但我们将继续使用它，因为它使整个事情更加健壮。

为了让它工作，我们将不得不安装一些库的静态版本，而不仅仅是丢失的那些。经过反复试验，我发现我们只需要三个。

```
*bash-4.2# yum install -y glibc-static gmp-static zlib-static*
```

现在网上有很多关于如何用 ghc/cabal/stack 编译一个静态二进制的[资源](https://vadosware.io/post/static-binaries-for-haskell-a-convoluted-approach/)。我选择了一个看起来很干净的，并且有一天可能会被用在工具上的。当我们运行`stack build`时，它根据我们`package.yaml`和`stack.yaml`中的内容为我们生成了一个`hs-lambda.cabal`文件。Stack 会警告我们不要乱动`.cabal`文件。事实上，自从它进入由`stack new`创建的`.gitignore`文件后，git 甚至没有跟踪它。然而，就目前而言，使静态链接工作的最好方法是在`executable`部分的`.cabal`文件中添加一行。

现在我们将重新编译。

```
*bash-4.2# stack clean
bash-4.2# stack build -ghc-options=’-fPIC’ — this will create a statically linked binary
bash-4.2# ldd .stack-work/install/x86_64-linux/lts-12.14/8.4.3/bin/bootstrap
 not a dynamic executable*
```

我们完了。如你所见，我们的二进制文件是独立的，可以在任何 linux 发行版上运行。

## 部署

现在，我们可以回到我们的开发箱，将整个东西打包并部署它。

```
*$ zip function.zip .stack-work/install/x86_64-linux/lts-12.14/8.4.3/bin/bootstrap
 adding: bootstrap (deflated 73%)
$ aws lambda create-function -function-name test-custom-runtime -zip-file fileb://function.zip -handler function.handler -runtime provided -role arn:aws:iam::123456789:role/aws-lambda-role
 {
 “FunctionName”: “test-custom-runtime”,
 “FunctionArn”: “arn:aws:lambda:eu-central-1:123456789:function:test-custom-runtime”,
 “Runtime”: “provided”,
 “Role”: “arn:aws:iam::123456789:role/aws-lambda-role”,
 “Handler”: “function.handler”,
 “CodeSize”: 4667617,
 “Description”: “”,
 “Timeout”: 3,
 “MemorySize”: 128,
 “LastModified”: “2019–01–15T21:40:01.625+0000”,
 “CodeSha256”: “TMVO8YwRTW3tu9CKpT5ShPe+iVVemrq9xKW2iDdS9Lc=”,
 “Version”: “$LATEST”,
 “TracingConfig”: {
 “Mode”: “PassThrough”
 },
 “RevisionId”: “d8579e6e-bc80–4951-be5f-cabb5d93f0b6”
 }*
```

我们准备好测试它了。

```
*$ aws lambda invoke -function-name ‘test-custom-runtime’ -payload ‘{“text”:”Hello”}’ response.txt
 {
 “StatusCode”: 200,
 “ExecutedVersion”: “$LATEST”
 }$ cat response.txt
 {“text”:”Hello”}*
```

有用！如您所见，我们的 lambda 运行成功，并将我们的有效负载反馈给了我们。

*原载于 2019 年 1 月 31 日*[*【https://www.tooploox.com】*](https://www.tooploox.com/blog/aws-lambda-custom-runtimes-with-haskell)*。*