# 过早的优化是有害的，除非

> 原文：<https://itnext.io/premature-optimization-is-evil-unless-881e6ada726a?source=collection_archive---------1----------------------->

作为软件开发人员，当我们开始在这个疯狂而复杂的观点和模式世界中的旅程时，我们被告知许多事情。我总是喜欢听到的一句话是过早的优化是万恶之源。而且这是真的，尤其是对于初创企业！一家初创企业需要快速、敏捷和斗志昂扬。你应该不断迭代你的产品，查看使用数据，识别模式和趋势，解决客户问题。对于那些说，“但是，如果我们有大量的人马上使用我们的服务，如果我们一夜之间有数百万的客户，那该怎么办…”恭喜你。这是一个很好的问题，这意味着你建立了人们想要使用的东西，虽然你可能看到或听到的性能不是最好的，但客户正在使用它，你可以开始优化。

我可以在这里结束这篇文章，但我想谈谈更重要的事情——在你开始写第一行代码之前就进行优化。而不是迷失在 NoSQL 对 SQL，或者文档对关系对图形的争论中。让我们再次关注一家初创公司应该做什么……发布代码和产品。我见过许多初创公司从 NoSQL 开始他们的项目，最终意识到他们并不真正了解数据将如何被访问和/或使用。这导致您的工程师在他们的服务层或客户端做大量繁重的工作。老实说，如果您不知道您的访问模式和您的数据将如何被使用，您应该坚持使用 RDBMS。我知道文档存储、图形数据库、宽列等都有有效的用例。，但是您刚才说您不知道您的访问模式是什么或者它们看起来会是什么样子，您需要 RDBMS 和 SQL 的灵活性。我将称之为预优化#1，你可以稍后感谢我。

既然我们已经确定我们的初创企业应该使用 RDBMS，我们就可以专注于运输代码，并开始捕获客户使用模式的数据。那么你用 Postgres，MySQL，Spanner，蟑螂，Planetscale 哪个数据库呢？老实说，没关系，他们都会做得很好，都有自己的长处和短处。但是，作为一家 1B 电子商务初创公司的前首席信息官/首席技术官，我曾经为客户开发过许多大型复杂的应用程序，包括职业体育联盟和大型财富 500 强公司，我可以告诉你，我会选择 Planetscale 来启动我的新项目。

Planetscale 真的把 DX 打出了公园。非块模式迁移、数据库分支和 Vitess 的底层技术——由 Youtube 构思并提供支持——简直令人惊叹。

![](img/d7caac4bf35b0a22c114b79655c2e2ee.png)

数据库分支

在我提供建议的最近一次创业中，我们将 Planetscale 作为最新工具添加到他们的工具箱中，使他们的团队能够持续开发产品并将其运送给客户。如果你还没有机会，我强烈推荐你去看看 Planetscale，并接受他们慷慨的免费服务。我称之为预优化#2，你可以以后再感谢我。至于何时优化项目的其他方面，只有您和您的团队知道，但您不需要担心的一件事是您的数据库的伸缩能力。