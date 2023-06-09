# 组件 vs (UI)集成 vs E2E 测试

> 原文：<https://itnext.io/component-vs-ui-integration-vs-e2e-tests-f02b575339dc?source=collection_archive---------2----------------------->

UI 测试分类。阅读此处或[dev .](https://dev.to/noriste/component-vs-ui-integration-vs-e2e-tests-3i0d)上的内容。

![](img/472d84f2b2e01f7623a47ed96932370e.png)

由[杰里米·毕晓普](https://unsplash.com/@jeremybishop)在 [Unsplash](https://unsplash.com) 上拍摄的照片

我正在 GitHub 上做一个大的 [UI 测试最佳实践](https://github.com/NoriSte/ui-testing-best-practices)项目，我分享这个帖子来传播它并有直接的反馈。

谈到 UI 测试(仅涉及 UI，而非底层 JavaScript 代码)，有三种主要的测试类型:

组件测试
UI 集成测试
端到端(E2E)测试

# 组件测试

UI 的单元测试，他们在一个隔离的环境中测试每一个组件。

独立开发组件很重要，因为它允许您将组件与相应的容器/用途隔离开来。一个组件的存在是为了隔离一个单一的行为/内容(单一责任原则[),因此，隔离编码是有益的。](https://www.wikiwand.com/en/Single_responsibility_principle)

有许多方法和工具可以独立开发组件，但是由于其有效性和生态系统，story book[成为了标准选择。](https://storybook.js.org)

组件有三种类型的契约:生成的输出( **HTML** )、它们的可视方面( **CSS** )和外部 API(**道具和回调**)。测试每一个方面都可能很麻烦，这就是故事片派上用场的地方。它允许您自动化:

**快照测试**:快照是组件生成的输出，是包含所有渲染 HTML 的字符串。如果生成的 HTML 意外或无意地发生了变化，快照测试就会失败，您可以选择这些变化是有意的还是无意的。

**视觉回归测试**:组件的视觉方面将**与前一个进行逐个像素的比较**，再次提示您选择是否接受更改。

这些测试由 [Storyshots](https://www.npmjs.com/package/@storybook/addon-storyshots) 在每一个故事书页面(又名“故事”)上自动启动。

**回调测试**:一个小的 React 容器应用程序通过一些回调来呈现组件。然后，模拟用户交互，并传递预期要调用的回调。 [React 测试库](https://testing-library.com/docs/react-testing-library/)是这类测试的标准库

**交互/状态测试**:一些与组件的交互期望正确的状态管理。这种测试必须从消费者的角度来写，而不是从内在的角度(例如。用户填充时输入字段的值，而不是内部组件状态)。交互/状态测试应该在键盘事件触发后断言输入字段值。

# UI 集成测试

他们在一个真实的浏览器中运行整个应用程序**,而不需要访问真实的服务器**。这些测试是每个前端开发人员的王牌。他们速度极快，更少暴露于随机故障或假阴性。Cypress 非常适合 UI 集成测试。

前端应用程序不知道没有真正的服务器:每个 AJAX 调用都会被测试工具立即解决。静态 JSON 响应(称为“fixtures”)用于模拟服务器响应。Fixtures 允许我们测试前端状态，模拟每一个可能的后端状态。

另一个有趣的效果是:fixture**允许你在没有后端**应用程序的情况下工作。你可以把 UI 集成测试想象成“仅前端测试”。

最成功的测试套件的核心是大量的 UI 集成测试，考虑最适合你的前端应用的测试类型。

# 端到端(E2E)测试

他们运行整个应用程序，与真正的服务器进行交互。从用户交互(其中一个“端”)到业务数据(另一个“端”):一切都必须按照设计工作。E2E 测试通常很慢，因为

他们需要一个**工作后端**应用程序，通常与前端应用程序一起启动。没有服务器你就不能启动它们，所以你要依靠后端开发人员来工作

他们需要**可靠的数据**，在每次测试前播种并清理数据

这就是为什么 E2E 考试不适合作为唯一/主要的考试类型。它们非常重要，因为它们测试一切(前端+后端),但是必须小心使用，以避免脆弱和长达一小时的测试套件。

在一个包含大量 UI 集成测试的完整套件中，您可以将 E2E 测试视为“后端测试”。您应该通过它们测试哪些流程？

快乐之路:你需要确保，至少，用户能够完成基本操作

对你的企业有价值的一切:幸福之路与否，测试你的企业所关心的一切(显然，对它们进行优先排序)

经常发生故障的一切:系统的薄弱环节也必须受到监控

识别/定义测试的类型有助于对它们进行分组，限制它们的范围，以及选择是否在整个应用程序和部署管道中运行它们。

同样， [Cypress](https://www.cypress.io/) 是我进行 E2E 测试的首选工具。

# 明智地命名它们

您可以编写许多不同的 UI 测试，并且用一种通用的方式命名
测试文件是一个好习惯。

这很有用，因为通常你只需要运行一种类型的测试，情况可能是:
-在开发过程中， 您只需要运行其中的一些
—您正在更改一些相关的组件，您需要检查生成的标记没有改变
—您正在更改一个全局 CSS 规则，您只需要运行可视化测试
—您正在更改一个应用流程，您需要运行整个应用集成测试
—您的 DevOps 同事需要确保一切正常运行，并且最简单的方法是 这样做只是启动 E2E 测试
-您的构建管道只需要运行集成和 E2E 测试
-您的监控管道需要一个脚本来启动 E2E 和监控测试

如果您明智地命名了您的测试，那么启动其中的某一类测试将会非常容易。

柏树:

```
cypress run — spec \”cypress/integration/**/*.e2e.*\”
```

玩笑:

```
jest — testPathPattern=e2e\\.*$
```

没有一种全局的方式来命名测试文件，建议可以用:
-您正在测试的主题
-测试的种类(*集成*、 *e2e* 、*监控*、*组件*等等。)
-选择的测试后缀(*测试*、*规格*等。)
-文件扩展名(。js，。ts，。jsx，。tsx 等。)
都用句号隔开。

一些例子可以是
-认证。e2e.test 。ts
-认证。**集成测试**。ts
-现场。**监测**。js
-登录。**组件测试**。tsx
等

你好👋我是 Stefano Magni，我是一名热情的 JavaScript 开发人员和 T2 赛普拉斯大使。
我喜欢创造高质量的产品，测试和自动化一切，学习和分享我的知识，帮助别人，在会议上发言和面对新的挑战。
我在意大利比特币初创公司 [Conio](https://conio.com/it/?source=post_page---------------------------) 工作。
你可以在 [Twitter](https://twitter.com/NoriSte?source=post_page---------------------------) 、 [GitHub](https://github.com/NoriSte?source=post_page---------------------------) 、 [LinkedIn](https://www.linkedin.com/in/noriste/?source=post_page---------------------------) 上找到我。你可以找到我最近所有的文章/演讲等等。这里。