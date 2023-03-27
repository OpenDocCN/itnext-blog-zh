# Flutter Web + Firebase 托管

> 原文：<https://itnext.io/flutter-web-firebase-hosting-45d7e3fc50f9?source=collection_archive---------2----------------------->

![](img/266943b3a985ebbe07644e7a5aa62745.png)

作为一名开发人员，这是前所未有的好时机；你把一个想法或概念变成现实的容易程度是因为缺乏一个更好的词——惊人。

在这篇文章中，我们将开始部署一个 Flutter web 应用程序到 Firebase 主机的旅程。

# 颤振构型

1.  创建一个新的 flutter 项目，并为 web 构建它

![](img/2a5295779fb87cd9e8ec0c887f907282.png)![](img/09e93c1dcd8316d53fed016753af5508.png)![](img/8ec8621274333ce7ad709aec4d51fc93.png)

颤动的部分已经完成了！:)

# 火基配置

1.  创建一个新的 firebase 项目[https://console.firebase.google.com/](https://console.firebase.google.com/)

![](img/954c945736619a99cd240182fd4e1499.png)

2.接下来，我们将安装 firebase 工具

![](img/d0f3142fbb85185161d3a9e45d281150.png)![](img/6967333597e6bac9166769e2b8ed07a1.png)

3.您运行下面的命令并使用您的凭据登录:

![](img/87a7a294c235c66ea0b3c69dc1c25605.png)

4.在你的 flutter 项目根目录下运行 **firebase init** 。

![](img/75bd47b61c2f8bfd2660c8f241e10058.png)

5.选择托管并选择相应的项目。

![](img/20f08f214b62de450bbe56189f44cb6d.png)

6.在提示“您希望将什么用作您的公共目录？”键入“ **build/web** ”，然后对“配置为单页应用程序(将所有 URL 重写为/index.html)”键入“是”

![](img/69d700e49d9833ed4c53724f636ffa11.png)

# 部署时间到了

```
firebase deploy --only hosting
```

![](img/271b45834bb1d40e7f5e8cc3e9411749.png)![](img/1493d2e72425cef54add73b257889fcb.png)

# 结论

如今部署静态网站超级简单方便。Firebase 托管使它变得非常简单。

您可以阅读 Firebase 文档并添加您的自定义域。

# 如果这有帮助的话:请给我一个掌声，它让我保持动力😀

⥅跟我上[中型](https://medium.com/@codememory101)。

⥅在推特上关注我。

📢在社交媒体上分享。