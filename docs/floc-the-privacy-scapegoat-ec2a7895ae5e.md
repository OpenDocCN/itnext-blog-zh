# FLoC:隐私的替罪羊

> 原文：<https://itnext.io/floc-the-privacy-scapegoat-ec2a7895ae5e?source=collection_archive---------2----------------------->

如果你在过去几个月里读过关于隐私或网络技术的文章，你可能听说过 FLoC。对于那些不太熟悉的人来说:这是谷歌提议的网络标准，用浏览器生成的兴趣分组(“群组”)取代广告跟踪 cookies 谷歌声称这项技术将[保护用户隐私](https://web.dev/floc/)，其他人都声称[是隐私噩梦](https://www.eff.org/deeplinks/2021/03/googles-floc-terrible-idea)。

在过去的几个月里，FLoC 一直是隐私和网络新闻周期中的热门话题，有来自[every](https://www.wired.com/story/google-floc-privacy-ad-tracking-explainer/)[major](https://www.theverge.com/2021/3/30/22358287/privacy-ads-google-chrome-floc-cookies-cookiepocalypse-finger-printing)[news](https://www.bloomberg.com/news/newsletters/2021-03-08/google-is-taking-away-the-cookies-and-plans-to-floc-us-all-instead)[outlet](https://www.forbes.com/sites/zakdoffman/2021/05/01/stop-using-google-chrome-on-your-iphone-android-macbook-and-pc/)的文章，无数的在线讨论，以及来自[EFF](https://www.eff.org/deeplinks/2021/03/googles-floc-terrible-idea)以及 DuckDuckGo 和 Mozilla 等众多公司的严厉回应。普遍的共识是，出于概念和技术原因，FLoC 是不好的，它可能会允许广告商单独针对用户，尽管它明确声称不会这样做。谷歌方面似乎基本上无视了这些评论，并在 Chrome 上实现了一个测试版，这让用户更加愤怒。

但是在所有这些中，有一些东西已经丢失了。

当被几家不同的新闻媒体问及 Edge 浏览器的 FLoC 计划时，微软给出了一个奇怪的冗长而曲折的回答:

> “我们相信在未来，网络可以为人们提供隐私、透明和控制，同时还支持负责任的商业模式，以创建一个充满活力、开放和多样化的生态系统。像谷歌一样，我们支持给予用户明确同意的解决方案，并且不绕过消费者的选择。这也是为什么我们不支持利用未经同意的用户身份信号的解决方案，如指纹识别。该行业正在发展，将会出现不需要个人用户 ID 的基于浏览器的提议，以及基于同意和第一方关系的基于 ID 的提议。我们将继续与社区一起探索这些方法。例如，最近，我们很高兴地介绍了一种可能的方法，正如我们的[鹦鹉提案](https://github.com/WICG/privacy-preserving-ads/blob/main/Parakeet.md)中所描述的。该提案不是最终版本，而是一份不断发展的文件。”

我马上总结一下，但首先考虑一下新闻媒体在面对这种回应时选择的一些标题:

[微软 Edge 成为最新一款禁用谷歌 FLoC 的浏览器](https://www.techradar.com/news/microsoft-edge-becomes-latest-browser-to-disable-googles-floc)

[火狐、Edge、Safari 和其他浏览器不会使用谷歌新的 FLoC 广告技术](https://www.theverge.com/2021/4/16/22387492/google-floc-ad-tech-privacy-browsers-brave-vivaldi-edge-mozilla-chrome-safari)

没有人想和谷歌的新追踪机制 FLoC (Android Police)有任何关系

仅仅从标题来看，你会认为微软已经对 FLoC 说了一个响亮的“不”。事实当然不是这样，因为这个回复实际上是在说‘我们支持总体想法，但我们还不知道我们要做什么’。但是在最后还有更重要的东西塞在里面:鹦鹉提案。

你会问，什么是鹦鹉？好吧，我做了我们所有的艰苦工作，阅读了密集，罗嗦的规格。本质上，从最终用户的角度来看，它与 FLoC 是一样的，关键的改进是，是浏览者而不是网站向广告网络发出请求，这意味着网站不能添加超出鹦鹉定义的阈值的任意数据。不幸的是，它完全忽略了一个事实，即浏览器(和服务器)可以在任何时候发出任意的网络请求，这些请求可以用来添加额外的识别数据。

事实上，我会告诉你一个网站如何绕过鹦鹉的这个限制:

1.  用户前往[https://cool-site.com。](https://cool-site.com.)
2.  cool-site.com 使用 JS 从用户的浏览器中收集尽可能多的识别信息，例如用户代理、浏览器版本、屏幕尺寸等。它还可能产生一个随机字节作为熵的来源。它将此数据发送回 cool-site.com 服务器，后者会添加 IP 地址，然后将此数据发送到广告网络。
3.  cool-site.com 发起了一个鹦鹉请求，只使用熵的一个字节作为兴趣。鹦鹉填充了更多的兴趣，用户的位置，一些用户代理参数，站点识别广告单元 ID，等等。
4.  广告网络现在可以组合原始站点、粗略位置、兴趣字节、一些粗略的用户代理参数，以及最重要的请求时间戳，以几乎 100%的成功率将冷站点服务器的请求与长尾小鹦鹉请求相匹配，这意味着对手可以容易地将附加的识别信息添加到长尾小鹦鹉请求中。

除此之外，鹦鹉还:

*   提议使用广告商付费访问的托管服务，在主机(如微软)的金钱利益和用户利益之间造成明显的冲突；
*   使用特定兴趣、位置和用户代理数据作为标识符，而不是 FLoC 的群组哈希，使其开箱即用时更容易识别，
*   需要信任不透明的托管服务来剥离数据，并且本身不收集任何此类数据。

因此，我们这里有一个 WICG 提案，由一家几乎与谷歌一样大的公司推动，与 FLoC 有类似的，在某些方面更糟糕的隐私问题。但这就是它令人担忧的地方:与 FLoC 形成鲜明对比的是，据我所知，只有[一篇面向消费者的文章](https://mspoweruser.com/microsoft-prefers-their-parakeet-to-googles-floc/)在标题中提到了长尾小鹦鹉——在一个专门迎合微软、*和*粉丝的网站上，它仍然在标题中提到 FLoC。显然，新闻媒体不认为长尾小鹦鹉是一个像絮一样有市场的话题，但是为什么呢？也许人们认为现代的微软是一家“好的”**公司，而谷歌是“大坏蛋”。或者，考虑到这是一个阴谋，弗洛克被有目的地用作替罪羊，以阻止人们调查其他提议。也许——很可能——这甚至不是最初的目标，但随着时间的推移，它演变成了那个角色。**

**不幸的是，这并不意味着长尾小鹦鹉完全没有被注意到，因为虽然没有面向消费者的文章，但实际上有无数针对广告行业人士的文章。广告高管们意识到，除了主流 Chrome，FLoC 可能很难在其他地方生存，他们显然也在关注其他选择。所以我决定看看我们还遗漏了什么。**

**事实证明，WICG 正在考虑的其他基于兴趣的广告提案数量惊人:[斑鸠](https://github.com/WICG/turtledove)、[雏鸟](https://github.com/WICG/turtledove/blob/main/FLEDGE.md)、[麻雀](https://github.com/WICG/sparrow)、[鸽子](https://github.com/google/ads-privacy/tree/master/proposals/dovekey)、[帕罗特](https://github.com/prebid/identity-gatekeeper/blob/master/proposals/PARRROT.md)、[燕鸥](https://github.com/WICG/turtledove/blob/main/TERN.md)和[海燕](https://github.com/w3c/web-advertising/blob/main/PETREL.md)等等。他们似乎都有相同的关注点:在广告行业杂志和报纸上发表了几篇文章，而主流新闻媒体上却没有。事实上，其中一些甚至是由谷歌赞助的，而仍然很少有甚至没有消费者文章。**

**所以不幸的是，似乎在某种程度上 FLoC 已经完成了它的目标——它只是和我们(可能还有谷歌)最初想的目标不一样。**