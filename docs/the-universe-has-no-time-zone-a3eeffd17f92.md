# 宇宙没有时区

> 原文：<https://itnext.io/the-universe-has-no-time-zone-a3eeffd17f92?source=collection_archive---------0----------------------->

![](img/4bdc3798c819595615dd26477d6d50ed.png)

很久很久以前，宇宙诞生了。这被广泛认为是不受欢迎的举动，并使许多人愤怒。

雪上加霜的是，人类创造了“时区”，以简化对他们的时间的推理。看，几个世纪以来，我们人类已经习惯了*当地时间*，即使我们没有太多的衡量标准。我们很高兴地说，对 T4 我来说，日出发生在早上 6 点。但是对我来说是早上 6 点，对日本的某人来说可能是早上 8 点。因此，世界走到了一起，并将地球分成了时区。这样，一个时区的每个人都可以忽略其他人，或者更确切地说，可以参考他们的当地时间，而不必太担心地球上的其他人。

时区是在大约公元 1675 年创建的，位于英格兰格林威治的[皇家天文台](https://www.rmg.co.uk/royal-observatory)在其全盛时期，就像所有的英国事物*一样，是时间本身的中心。于是就有了 GMT，格林威治标准时间。也称为 UTC，协调世界时。请不要问我 CUT 是怎么变成 UTC 的。*

继续前进，世界，因为它是，接受 GMT 时间，或格林威治地区的时间，作为世界时间，没有任何时区。

但是，如果您不在 GMT，那么您所在位置的当地时间将由 GMT 的一个偏移量来标识。因此，如果在印度，时间是 08:30:24 (hh:mm:ss)，这意味着现在是印度指定时区 IST 的 08:30:24，定义为 **UTC+05:30** 。

更进一步，当事件发生在 IST 的 08:30:24 时，这意味着格林威治时间，GMT / UTC 时区是 03:00:24 (08:30:24 - 05:30:00)。

因此，名称 **hh:mm:ss + th:tm** 意味着，由 **hh:mm:ss** 部分表示的时间是**当地时间**，为了获得 UTC 时间，我们必须从中减去 **th:tm** 。

# 问题是

大约在 20 世纪 70 年代，在所有的计算机革命之前，对于正常的应用程序来说，编程时间和日期没有太多问题。你做了一个软件，配置成在给定的时区使用，人们对它很满意。其他时区的用户也同样配置了他们的软件，他们也很高兴。

当一个时区的软件与其他时区的软件交互时，问题就出现了。你看，如果你不小心编程你的软件，很好地照顾时区，你可能会以损坏的时间/日期值为你的数据而告终。

然后互联网出现了。其要点是让整个工厂的网络能够相互通信。

现在，日期和时间的编码问题变得复杂了。很多次了。以至于即使是现在，在 21 世纪，我们仍然有正确的日期和时间编程的问题。

# 解决方案

有几种方法可以简化时区的日期和时间编码。一些简单的规则可以使软件中关于日期和时间的推理更容易处理。

*   确保*代码中的所有*时间戳都用时区定义为*。*
*   始终依赖运行软件的系统提供的时区信息。
*   转换为 UTC 后，始终存储所有时间戳。注意，这只是针对存储部分。根据您的应用程序，您可以通过用户输入或运行环境，在校正相关时区后向用户显示时间戳。

# 进入节点时间

NodaTime 是一个. NET 库，它极大地简化了。NET 平台。图书馆有一种不同的方法来考虑日期、时间和时区。

*   在内部，野田时间将时间视为一条单一的、非相对论性的线性时间线，“零”点是 Unix 纪元(UTC 1970 年 1 月 1 日午夜)。这意味着，如果野田时间图书馆被拟人化，它就是一个漂浮的外星人，比如说在我们的太阳周围，保持着一个始于 Unix 时代的秒表。现在，由于 UTC 也处于“零”时区参考中，一个在 UTC 时区格林威治的人，拥有一个在同一 Unix 纪元开始的秒表，他的秒表上的刻度将与外星人拥有的秒表完全匹配。野田时间称秒表的数字为**瞬间。**这是野田时间保持的全球时间线。
*   外星人或人类的秒表每一秒都在滴答作响。这意味着，Noda 时间库中的一种即时数据类型是一个数字，它显示自 Unix 纪元以来已经过去了多少纳秒。这个数字对于太阳附近的外星人和 UTC 区域的人类是一样的。
*   因为即时类型只是纳秒，所以它没有隐含的日历系统。因此，为了获得给定日历系统中的本地时间，程序员可以使用工具将这些即时值转换成所需的参考时区和时间段。

让我们看一些代码。假设您有一个数据库，在本例中是 PostgreSQL，它有一个带时区列的时间戳。要创建 NodaTime 时区的日期时间，我们可以执行以下操作

要将实体插入数据库，我们可以这样做。

要检索同一个实体，我们可以这样做。

BrandEntity 对象定义如下

因此，我们已经介绍了使用 NodaTime 获取、存储和检索时区日期时间对象。一定要查看它的文档，了解一些有用的方法和操作符。

**注意**:你必须像这样配置 Npgsql 来使用 NodaTime。对于 ASP.NET 核心 MVC 应用程序，这可以在启动类 ConfigureServices 方法中完成。

```
NpgsqlConnection.GlobalTypeMapper.UseNodaTime();
```

我强烈建议每个人都使用 NodaTime。NET 项目。您可以在中了解为什么会有这样一个库，以及它为什么会取代 DateTime 实用程序。网本身在[这个博客帖子](https://blog.nodatime.org/2011/08/what-wrong-with-datetime-anyway.html)由乔恩斯基特。