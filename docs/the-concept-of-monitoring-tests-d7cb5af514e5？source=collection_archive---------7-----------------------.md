# “监控测试”的概念

> 原文：<https://itnext.io/the-concept-of-monitoring-tests-d7cb5af514e5?source=collection_archive---------7----------------------->

小 E2E 测试检查很少(但至关重要)的技术细节。阅读此处或[dev .](https://dev.to/noriste/the-concept-of-monitoring-tests-4l5j)上的内容。

![](img/a5d11c3408009f63fd0cf913bc49f7a9.png)

由 [Rainer Bleek](https://unsplash.com/@brain1966?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 在 [Unsplash](https://unsplash.com/?source=post_page---------------------------) 上拍摄的照片

我正在 GitHub 上做一个大的 [UI 测试最佳实践](https://github.com/NoriSte/ui-testing-best-practices?source=post_page---------------------------)项目，我分享这个帖子来传播它并有直接的反馈。

几个月前，我在基于《盖茨比》的 business.conio.com 网站工作。除了分享一些我写的插件之外，我想尽可能地提高网站的性能。幸运的是，盖茨比在谈到性能时表现突出，但你知道，这永远不够。

我推了推科诺的 DevOps👋)尽可能地利用 AWS/S3 的能力，为网站用户提供 [Brotli](https://www.wikiwand.com/en/Brotli) 压缩的和永久缓存的静态资产。结果是一个超高性能的产品，但道路并不容易，因为由于铲斗配置，有时 **Brotli 压缩被打破**。

这个错误很微妙，因为该网站使用了大量 JavaScript 特性。网站**看起来**可以工作，但是如果压缩设置不正确，联系表单就不会工作。错误的症状只是控制台中显示的一个错误。

这种错误很容易用检查表单是否工作的 E2E 测试识别出来(显然是从用户的角度来看)，但是我不能依赖这样的测试，因为测试套件非常慢。毕竟，每个 E2E 测试都很慢。
更多:DevOps 需要频繁的反馈，他改变了 S3 配置很多次，在几秒钟内**而不是几分钟内**收到反馈会很好。

因为检查非常简单，而且我讨厌手工测试，所以我写了一个简单的测试来检查 Brotli 压缩。我使用了 [Cypress](https://www.cypress.io) ，并编写了如下测试

```
// extract the main JS file from the source code of the page. I removed the regex matching part
const getMainJsUrl = pageSource => "/app-<example-hash>.js"context("The Brotli-compressed assets should be served with the correct content encoding", () *=>* {
  *const* test = *url* *=>* {
   cy.request(url)
    .its("body")
    .then(getMainJsUrl) // retrieves the app.js URL
    .then(*appUrl* *=>* cy.request({url: url + appUrl, headers: {"Accept-Encoding": "br"}})
    .its("headers.content-encoding")
    .should("equal", "br"))
  } it("staging", () *=>* test(urls.staging))
  it("production", () *=>* test(urls.production))
})
```

一旦写好，我可以提供一个专门的脚本，只启动这个测试(排除所有标准的 E2E 测试)。Et voilà:我可以通过一个超快速的测试来持续监控 Brotli 压缩！

缓存管理呢？我们也遇到了一些麻烦，我添加了一些专门的测试

```
const shouldNotBeCached = (xhr) => cy.wrap(xhr)
  .its("headers.cache-control")
  .should("equal", "public,max-age=0,must-revalidate")const shouldBeCached = (xhr) => cy.wrap(xhr)
  .its("headers.cache-control")
  .should("equal", "public,max-age=31536000,immutable")context('Site monitoring', () => {
  context('The HTML should not be cached', () => {
    const test = url =>
      cy.request(url)
        .then(shouldNotBeCached)

    it("staging", () => test(urls.staging))
    it("production", () => test(urls.production))
  })

  context('The static assets should be cached', () => {
    const test = url =>
      cy.request(url)
        .its("body")
        .then(getMainJsUrl)
        .then(appUrl => url+appUrl)
        .then(cy.request)
        .then(shouldBeCached)

    it('staging', () => test(urls.staging))
    it('production', () => test(urls.production))
  })
}
```

我喜欢这些小测试，因为在几秒钟内，它们就能检查出对用户体验至关重要的东西。我可以睡得很好，我们**永远**免受这些问题**。**

# **监控测试还能检查什么？**

**用 Gatsby 配置(针对不同的环境有许多条件和定制)制造混乱太容易了。第一个，也是最关键的，需要监控的东西是最简单的:robots.txt 和 sitemap.xml 文件。**

***robots.txt* 文件必须禁止暂存站点抓取并允许生产站点抓取:**

```
context('The robots.txt file should disallow the crawling of the staging site and allow the production one', () *=>* {
  *const* test = (*url*, *content*) *=>* cy.request(`${url}/robots.txt`)
      .its("body")
      .should("contain", content) it('staging', () *=>* test(urls.staging, "Disallow: /"))
  it('production', () *=>* test(urls.production, "Allow: /"))
})
```

**与静态资产一样，sitemap.xml 文件不能被缓存:**

```
context('The sitemap.xml should not be cached', () *=>* {
  *const* test = *url* *=>* cy.request(`${url}/sitemap.xml`)
      .then(shouldNotBeCached) it('staging', () *=>* test(urls.staging))
  it('production', () *=>* test(urls.production))
})
```

**由于一个错误的构建过程而出现的错误，我又编写了一个监控测试:有时除了主页之外，所有页面都包含 404 页面的相同内容。测试如下:**

```
context('An internal page should not contain the same content of the 404 page', () *=>* {
  *const* pageNotFoundContent = "Page not found"
  *const* test = *url* *=>* {
    cy.request(`${url}/not-found-page`)
      .its("body")
      .should("contain", pageNotFoundContent)
    cy.request(`${url}/about`)
      .its("body")
      .should("not.contain", pageNotFoundContent)
  } it('staging', () *=>* test(urls.staging))
  it('production', () *=>* test(urls.production))
})
```

# **运行它们**

**我用 [Cypress](https://www.cypress.io) 编写了测试并运行它们——如果你将文件命名为 *xxx，那么运行起来会非常简单。* ***监控*** *.test.js:***

```
cypress run — spec \"cypress/integration/**/*.**monitoring**.*\"
```

# **为什么把它们和标准的 E2E 测试分开？**

**嗯，因为:**

*   **监控测试不是从用户的角度编写的，E2E 测试是。但是对于 E2E 测试，我可能会有一个“联系人表单应该工作”的失败测试，而对于监控测试，我可能会有一个“brotli 压缩应该工作”的失败测试(越来越有用)。我总是更喜欢面向用户的测试，但是，当某些东西经常失败时，我希望保持检查**
*   **监控测试不贵，E2E 测试不贵:E2E 测试非常慢，它们会堵塞你的管道队列，而且基于你如何实施它们，它们会影响你的分析。这就是为什么我通常不对`production`环境运行它们，而只对`staging`环境运行它们。监控测试在两种环境下运行，没有缺点**

**你可以在我的这个要点中找到所有的测试**

**如果你有更多的想法或问题，请留下评论👍**

**你好👋我是 Stefano Magni，我是一名热情的 JavaScript 开发人员和 T2 赛普拉斯大使。
我喜欢创造高质量的产品，测试和自动化一切，学习和分享我的知识，帮助别人，在会议上发言和面对新的挑战。
我在意大利比特币初创公司 [Conio](https://conio.com/it/?source=post_page---------------------------) 工作。
你可以在 [Twitter](https://twitter.com/NoriSte?source=post_page---------------------------) 、 [GitHub](https://github.com/NoriSte?source=post_page---------------------------) 、 [LinkedIn](https://www.linkedin.com/in/noriste/?source=post_page---------------------------) 上找到我。你可以找到我最近所有的文章/演讲等等。这里。**