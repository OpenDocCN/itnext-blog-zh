# 如何在 Kubernetes 上构建和操作开源数据栈

> 原文：<https://itnext.io/how-to-build-and-operate-an-open-source-data-stack-on-kubernetes-c1ec34742afd?source=collection_archive---------4----------------------->

当前的共识是在 Kubernetes 上部署您的开源数据堆栈，因为它提供了无缝扩展和使用标准工具操作复杂基础设施的能力。

![](img/b89a3902b01ccfa9eb7a9b335e066de4.png)

在 Kubernetes 上构建和操作开源数据栈的指南。照片由[马丁·亚当斯](https://unsplash.com/@martinadams?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit) / [Unsplash](https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit) 拍摄

当前的数据工程生态系统充满了来自开源和第三方解决方案的各种工具。虽然在选择哪条道路(选择开源或第三方供应商)上还没有达成共识，但我认为探索构建开源数据栈的可能性是很有趣的(而且，在当前的市场状态下，这确实是重新考虑如何设计数据栈并开始探索开源替代方案的最佳时机)。

目前的共识是在 Kubernetes 上部署您的开源数据栈，因为它提供了无缝扩展的能力，并使用标准工具操作复杂的基础设施。

操作大规模系统近十年来，我亲眼目睹了自行部署开源应用程序的挑战。另一方面，我意识到，对于组织来说，自托管其数据堆栈以更好地处理与其工作领域相关的隐私和安全问题是多么有价值。

虽然这对开发人员来说肯定是一个挑战，但我认为作为一个组织，使用正确的工具和流程部署自托管的开源数据堆栈肯定是合理的。

在本文中，您将了解到:

*   在数据堆栈的每一层都有最常见的开源工具可供选择
*   为什么您应该选择在 Kubernetes 上自托管您的数据堆栈
*   在部署开源数据堆栈时，我看到了三个潜在的挑战
*   构建开源数据栈

首先，您需要为数据堆栈的每一层选择正确的开源工具。好的开源工具有很好的文档记录，有强大的社区存在，并且正在被积极开发。良好的社区支持渠道和贡献计划的加分。

# 数据摄取

对于数据摄取，一个流行的解决方案是 [Airbyte](https://airbyte.com/) ，这是一个开源的 EL(T)平台，可以帮助您将不同来源的数据复制到您的数据仓库、数据湖和数据库。由于其易用性和低启动门槛，它正迅速成为复制作业的行业标准。

我经常看到公司使用的另一个摄取工具是 [Singer](https://www.singer.io) ，这是一个开源的 ETL 工具，可以为您组织的所有数据提供数据提取和整合功能。

# 数据转换

[dbt](https://www.getdbt.com/) 已经成为数据转换的主要发展方向。它不仅是一个很好的开源解决方案，而且安装和运行起来也很简单。他们还有一个由 25，000 多名数据从业者组成的强大社区，可以回答您在实施过程中可能遇到的任何问题。

我见过的另一个有趣的选项是[data form](https://dataform.co/)——更具体地说是他们的开源 SDK——一个帮助数据团队开发、测试和部署基于 SQL 的数据工作流到仓库的框架。

# 数据编排

拥有一个编排引擎来调度作业并管理它们之间的大量依赖关系是当今数据堆栈的重要组成部分。多年来， [Apache Airflow](https://www.youtube.com/watch?v=rGQ-Vzy8aso) 一直是数据工程师在数据管道中编排作业的标准。

最近， [Dagster](https://dagster.io/) 作为气流的替代物，在这个领域获得了越来越多的关注。与 Airflow 不同，Dagster 在构建时考虑了完整的软件开发生命周期，允许开发人员在部署到产品之前，首先在本地协调他们的管道。我想强调的另一个关键特性是 Dagster 的现代界面，它允许开发人员可视化 Dag 的状态和他们输出的任何日志。

在接下来的几个月中，您需要关注的一个工具是[perfect](https://www.prefect.io/)，这是一个旨在处理现代数据基础设施的工作流管理系统。

# 数据仓库/分析引擎

对于数据存储层，有三种常见的开源解决方案，这些年来受到了广泛的关注。

*   [**Clickhouse:**](https://clickhouse.com/) 高性能，易于设置，在简单的硬件上运行良好。如果您想要建立一个列数据存储，并且您确切地知道您想要存储哪种类型的数据，这是一个不错的选择。
*   [](https://prestodb.io/)****/**[**Trino**](https://trino.io/):位于现有数据库之上的强大 SQL 查询引擎。当您不需要底层存储解决方案而想要一个更通用的解决方案时，这两种解决方案都很有用。**
*   **[**Apache Spark**](https://spark.apache.org/)**:**数据处理行业最大的开源项目之一。自 2009 年首次发布以来，统一分析引擎一直是一些最大公司的数据和机器学习业务的主要组成部分。**

# **数据可视化**

**一旦您以一种可展示的方式获得了数据，您仍然需要一个 BI 团队来查询数据，以便为内部和外部的利益相关者提供分析。这个领域最流行的三个开源解决方案是 [Apache 超集](https://superset.apache.org/)、 [Lightdash](https://www.lightdash.com/) 和 [Metabase](https://www.metabase.com/) 。**

**Apache 超集和 Metabase 非常相似，这种选择通常可以归结为您更喜欢哪种用户体验。众所周知，Metabase 有更直观的 UI，而 Superset 可能更强大，但学习曲线更陡。**

**另一方面，lightdash 是一个有趣的选择，尤其是当您在组织内部大量使用 dbt 时，因为 BI 所需的一切都作为代码编写在您的 dbt 项目中。**

# **关于在 Kubernetes 上运行数据基础设施栈的误解**

**![](img/cfcb77c21d24a6e23fcc91992585d040.png)**

**关于在 Kubernetes 上托管数据堆栈的常见误解。巴勃罗·加西亚·萨尔达尼亚 / [Unsplash](https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit) 拍摄的照片**

**虽然自托管您的数据堆栈并不是一个新概念，但是仍然有一些误解需要解决。弄清楚如何部署开源数据栈并不像以前那样困难，尤其是如果你有经验或者愿意学习 Helm 和 Kubernetes。您可能会启动 Metabase 或 Airbyte 之类的应用程序，但这可能比您预期的要长。**

**我想说明的是，如果你完全是从零开始构建，学习 Kubernetes 的斜坡时间是很长的，所以在大多数情况下，在短时间内学习 Kubernetes 对开发人员来说并不是微不足道的。然而，在过去的几年中，数据基础架构环境已经成熟了很多，在云中安装应用程序的能力也比以前好得多。**

**我经常听到的另一个常见误解是，扩展应用程序具有挑战性。我认为这需要一些澄清。有状态系统(例如数据仓库)非常难以扩展，因为您需要管理数据分区和数据仓库特有的其他问题。**

**然而，作为一个整体社区，可以肯定地说，我们已经找到了如何扩展纯计算系统的方法(这是大量数据软件所考虑的)。)使用合适的基础设施扩展计算系统并不太复杂，只需调整几个转盘。**

**您将使用一个 Kubernetes 部署或一个 Kubernetes 有状态集，并根据您获得的流量调整副本。此外，现在存在非常强大的自动扩展结构，因此扩展不再像以前那样具有挑战性。挑战更多地在于了解要监控和调整的适当指标。**

# **为什么您应该在 Kubernetes 上自托管您的开源数据栈**

**![](img/742e6b5fbfbafb3b5306889067fe3386.png)**

**考虑在 Kubernetes 上自托管开源数据栈的三个理由。Claudio Sc 拍摄的照片**

**我经常被问到为什么组织应该考虑在 Kubernetes 上运行他们的数据堆栈。虽然这样做有很多好处，但我认为重要的是强调我在 Kubernetes 上运行您的数据堆栈时看到的三个主要好处。**

1.  ****节省成本:**在我看来，最大的好处是组织节省的成本，尤其是当你的业务开始成熟时。大规模运行基础架构的大型组织每年实际上可以减少超过 100 万美元的云开支。我还看到了令人难以置信的成本节约，尤其是在与托管服务层配对时。通常，如果您要购买托管服务解决方案，通常会有 40%的计算加价，如果您正在进行大规模批处理作业，成本会迅速增加。**
2.  ****更简单的安全模式:**一切都是一个加固的网络，无需担心隐私和合规性问题。当您开始谈论 GDPR 和 CCPA 环境时，能够围绕产品分析套件实现强大的法规遵从性和隐私性尤其有帮助，因为这些环境很难大规模全面实施。**
3.  ****在操作上更容易扩展到多个解决方案:**在大多数情况下，您可能需要将 Airbyte 与其他解决方案(如 Superset、Airflow 和 Presto)一起运行。要有效地做到这一点既有挑战性又费时。如果您致力于一起运行所有这些应用程序，自托管的 Kubernetes 模型将变得非常强大，因为它将所有应用程序统一在一个单一的环境中。开发人员可以在 web UI 的基础上创建统一的管理体验。另一个好处是应用程序升级过程是统一的。升级像 Kubeflow 这样的工具所涉及的复杂性需要 Kubernetes 级别的大量依赖项。这些依赖性可能会与您的应用程序(如 Airflow)部署冲突，后者需要类似于 K9 或 Istio 的东西。有了统一的平台，这一过程得到了简化，开发人员可以轻松地在所有应用程序的依赖关系之间进行差异检查，并验证它们在任何给定时间都是可升级的。**

# **部署开源数据栈的三大挑战**

**![](img/d4b11ec8bb989f8fb86308d4d14c814e.png)**

**下面是部署开源数据栈时可能遇到的三个常见挑战。照片由[迈克·什切潘斯基](https://unsplash.com/@youngprodigy3?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit) / [Unsplash](https://unsplash.com/?utm_source=ghost&utm_medium=referral&utm_campaign=api-credit) 拍摄**

**我坦率地承认，对于一些工程团队来说，运行自托管数据堆栈并不总是合理的，并且可能具有挑战性，尤其是如果您之前没有任何建立和运行自托管堆栈的经验。**

**虽然这样做肯定是可能的，但我想快速指出我在部署自托管开源数据堆栈时遇到的三个主要挑战。**

1.  ****管理升级周期:**使升级具有挑战性的是依赖性的复杂性。这方面的一个例子是 [Kubeflow](https://www.kubeflow.org/) ，它有大约 15 个部署在引擎盖下运行。运行 Kubeflow 时，您需要运行诸如 [Knative](https://knative.dev/docs/) 、 [Istio](https://istio.io/) 和 Kubernetes 版本依赖等应用程序，以确保升级的正确安装。如果集群中发生任何变化，您整个流部署都会中断。管理升级生命周期非常困难，事实是这需要手动操作。在烘焙、升级和交付之前，您必须在所有目标云中进行手动验证。而且，在跟踪发布到上游开源项目的安全补丁和新版本方面，您最终不得不进行手工操作。**
2.  ****应对安全挑战:**使用开源软件的一个挑战是弄清楚如何让一切都安全。许多开源软件都有一个关于安全的坏故事。许多开源应用程序没有登录——因此它们没有任何身份验证解决方案，即使有，也是过时的、偷偷进入企业计划或没有正确实现。安全性的另一个极其复杂的部分是您的解决方案的软件供应链管理。您必须实现图像扫描，以确保您了解 docker 图像中可能存在的任何漏洞。基础映像包含安全漏洞是很常见的，消除这些漏洞并不是一项简单的任务。实施网络安全仍然很困难。服务网格应该是这方面的解决方案，但是当你开始推动它们时，我们已经看到了兼容性问题，特别是在与数据库操作者的交互中，事情开始变得糟糕。**
3.  ****不同工具的集成是棘手的:**目前这个过程仍然是手工的，需要一些思考。共享网络层是一个真正的大赢家，这也是我们喜欢运行单集群的原因。这也是开发商越来越看好 Kubernetes 的原因。这是一个一致的操作环境，能够为您的所有应用程序添加某种整体安全配置文件。然而，请记住，实际上这并不是一个完整的解决方案。有大量其他应用程序需要您手动集成，但没有一个真正的基础架构级别可以解决这一问题。**

# **结论**

**这篇文章来源于 2022 年 3 月 15 日我对 Kubernetes 上的数据社区负责人 [Bart Farrell](https://www.linkedin.com/today/author/bart-farrell) 的采访。如需进一步了解背景，请点击查看完整一集[。](https://youtu.be/KzaQNvXhIMs)**

**是否有我遗漏的工具或解决方案应该添加到列表中？如有任何问题或意见，请联系本人 mjg@plural.sh。**

**而且，如果你喜欢我们在 Plural 做的事情，并且想为我们的开源项目做贡献，你可以在这里查看我们的源代码。**

***原载于 2022 年 7 月 6 日*[*https://www . plural . sh*](https://www.plural.sh/blog/how-to-build-and-operate-an-open-source-data-stack-on-kubernetes/)*。***