# 🔥帮助我从亚马逊和 LinkedIn 获得服务请求前端的系统设计概念

> 原文：<https://itnext.io/system-design-concepts-that-helped-me-get-sr-frontend-offers-from-amazon-linkedin-9e100f3ce7d2?source=collection_archive---------1----------------------->

## 如果你刚刚开始你的系统设计之旅，这个概念的汇编将帮助你从基础开始。

![](img/cedf95a8ba10a61a936981575db1cd4f.png)

# 内容

*   [**简介**](#08d3)
*   [**客户机-服务器模式**](#9f22)
*   [**网络协议**](#f515)
*   [**存储**](#3ff8)
*   [**延迟和吞吐量**](#c33f)
*   [**可用性**](#d34a)
*   [**缓存**](#4496)
*   [**代理**](#7642)
*   [**负载均衡器**](#36ca)
*   [**散列**](#72bb)
*   [**数据库**](#add9)
*   [**键值存储**](#6be7)
*   [**专门的存储范例**](#d01a)
*   [**复制和分片**](#fedf)
*   [**领袖选举**](#9629)
*   [**点对点**](#5058)
*   [**轮询和流式传输**](#810d)
*   [**配置**](#fa78)
*   [**限速**](#dfdf)
*   [**伐木**](#3743)
*   [**发布/订阅模式—流**](#dad0)
*   [MapReduce](#e20e)
*   [**安全**](#d1b5)
*   [**结论**](#6a82)
*   [**了解更多**](#3035)

# 介绍

这篇博文是前端面试备忘单的延续。如果你还没看过，先看看吧👇

[](/frontend-interview-cheatsheet-that-helped-me-to-get-offer-on-amazon-and-linkedin-cba9584e33c7) [## 🔥帮助我获得亚马逊和 LinkedIn 录用通知的前端面试备忘单

### 如果你正在准备一个前端面试，想快速更新你的领域知识，这个备忘单将…

itnext.io](/frontend-interview-cheatsheet-that-helped-me-to-get-offer-on-amazon-and-linkedin-cba9584e33c7) 

这里的列表会给你一个系统设计面试概念的细粒度介绍，你可能需要掌握。希望你会喜欢阅读，别忘了在社交媒体上关注我:[](https://easy-web.medium.com/subscribe)**和 [**推特**](https://twitter.com/easy_web_org)**

**如果本文收集到 **1000 个掌声**👏，我会贴上**系统设计面试模板**，让你轻松准备面试**

# ****客户端-服务器模式****

*   **[**DNS**](https://aws.amazon.com/route53/what-is-dns/)**—域名系统将域名重定向到 IP 地址。客户端向 DNS 发送查询并返回 IP 地址；****
*   ****[**IP 地址**](https://whatismyipaddress.com/ip-address) —连接到互联网的每台机器的数字地址；最大允许偏差(0–255)。192.0.0.1—本地主机。192.169..c.d****
*   ****[**端口**](https://whatismyipaddress.com/port) —监听多个进程不碰撞。0–65525.2 ⁶.0–1023 端口被系统占用(22:安全外壳，53: DNS 查找；80:HTTP；443: HTTPS)****

# ******网络协议******

*   ****[**IP**](https://www.mozilla.org/en-US/products/vpn/more/what-is-an-ip-address/) —互联网协议—互联网上机器之间通信的规则。使用 IP 数据包****
*   ****[**TCP**](https://www.sdxcentral.com/resources/glossary/transmission-control-protocol-tcp/) —传输控制协议——保证数据包按顺序传递，会知道是否发生了故障。通过附加的 TCP 报头(订单信息)，建立在 IP 之上****
*   ****[**IP 包**](https://www.easytechjunkie.com/what-is-an-ip-packet.htm) —机器间通信的最小数据单元。由报头(源、接收者、数据包大小、IPv4 协议版本)和有效载荷组成。来自数据包⁶字节的大小。不能保证所有的数据包都被**收到并订购了 HTTP - > TCP - > IP******

# ******储存******

*   ******数据库** —在磁盘或内存中存储数据的服务器。记录和查询；****
*   ****[**磁盘**](https://www.webopedia.com/definitions/disk/) —如果进程死亡，存储并保存数据。硬盘—速度更慢，价格更低。SSD —更快、更贵、非易失性；****
*   ****[**内存**](https://en.wikipedia.org/wiki/Random-access_memory) — RAM 快速存取，易失性，进程死亡时擦除数据；****
*   ****[**持久数据**](https://dzone.com/articles/what-is-persistent-data) —如果进程死亡(中断)，保留数据；****

# ******延迟和吞吐量******

*   ****[**潜伏期**](https://developer.mozilla.org/en-US/docs/Web/Performance/Understanding_latency)**——**完成操作所需的时间(毫秒):****

****1)从 RAM 中读取 1mb—0.25 毫秒；****

****2)从 SSD 读取 1mb—1 ms；****

****3)通过网络传输 1mb(1gb/s)—10ms；****

****4)从 HDD 中读取 1mb—20 ms；****

****5)转移 1 Mb 洲际往返—150 ms；****

*   ****[**吞吐量**](https://en.wikipedia.org/wiki/Throughput) **—** 操作系统每秒可以处理的数量(RPS，QPS)****

# ******可用性—可靠性******

*   ****[](https://www.techopedia.com/definition/990/availability)**—服务在时间点(每年)的访问；1) 99% (87.8 小时)；2) 99.9% (8.8 小时)；3) 99.99% (52.6 分钟)4) **99.999% (5.3 分钟)** ( **5 个九**)；******
*   ****[**冗余**](https://computersciencewiki.org/index.php/Redundancy) —复制系统以提高可用性(确保没有单点故障—使用冗余)。**被动**冗余(如果一台机器死亡，另一台机器工作)，主动**冗余(如果一台机器死亡，另一台机器控制功能)******
*   ****[](https://en.wikipedia.org/wiki/Service-level_agreement)**(服务级别——协议)服务的保证，由多个 SLO 组成******
*   ******[](https://en.wikipedia.org/wiki/Service-level_objective)**(服务级别——目标)服务的保证(可用性)********

# ********缓存********

*   ********可以缓存什么—** 客户端，服务器，中间(服务器-数据库)，API，硬件(CPU)。频繁使用的操作，繁重的操作/计算；******
*   ****[**缓存**](https://en.wikipedia.org/wiki/Cache_(computing)) —存储数据以提供更快访问的硬件或软件(繁重计算、API 响应、API 请求的结果)；****
*   ****[**直写缓存**](https://www.geeksforgeeks.org/write-through-and-write-back-in-cache/#:~:text=In%20write%2Dthrough%2C%20data%20is,power%20outage%20or%20system%20failure).) —在缓存和内存中同步更新缓存时，如果进程死亡，恢复数据。优点—更简单可靠；缺点—当缓存中没有太多写入时效果更好；****
*   ****[**写回缓存**](https://www.geeksforgeeks.org/write-through-and-write-back-in-cache/#:~:text=In%20write%2Dthrough%2C%20data%20is,power%20outage%20or%20system%20failure).) —缓存与内存异步更新时；****
*   ****如果我们关心缓存的陈旧性(数据的准确性)——>我们使用本地缓存；**如果准确性很重要(浏览量、点赞数)** —使用第三方缓存(Redis)****
*   ****[**缓存** **命中**](https://www.techopedia.com/definition/6306/cache-hit#:~:text=A%20cache%20hit%20is%20a,already%20contains%20the%20requested%20data.) —在缓存中找到请求的数据****
*   ****[**缓存未命中**](https://www.techopedia.com/definition/6308/cache-miss) —由于纯设计，未找到存储的缓存****
*   ****[**缓存驱逐策略**](https://en.wikipedia.org/wiki/Cache_replacement_policies) —缓存时将被移除的规则( **LRU** (最近)、 **LFU** (频繁)、 **FIFO** (队列))****
*   ****[**内容交付网络**](https://en.wikipedia.org/wiki/Content_delivery_network) **(CDN)** —缓存你的服务器的第三方服务(基于地区)，总是更好的延迟。PoP(存在点)。 **Cloudflare，谷歌云 CDN。******

# ******代理—安全性、可靠性******

*   ****[**转发代理**](https://www.jscape.com/blog/bid/87783/forward-proxy-vs-reverse-proxy)**——**客户端(隐藏 IP)VPN；****
*   ****[**反向代理**](https://www.jscape.com/blog/bid/87783/forward-proxy-vs-reverse-proxy)**——**服务器拦截器(负载均衡、领袖选举)(日志记录、缓存、过滤请求、负载均衡、安全屏蔽)；****
*   ******代理工具—Nginx；******

# ******负载平衡器—安全性、可靠性、性能******

*   ****在服务器之间分配流量。具有不同选择策略的多个负载平衡器；****
*   ****[**服务器选择策略**](https://www.caida.org/workshops/wide/0503/slides/toshiyuki.pdf)——(权重——如果有的服务器更强大)循环、随机选择、基于性能、基于 IP、基于 API 路径；****
*   ******热点** —将过多的流量分配到一台机器上，导致次优的分片密钥/哈希函数****
*   ******LB 文库—Nginx；******

# ******散列法******

*   ****[**哈希**](https://www.educative.io/edpresso/what-is-hashing)**——**将一段数据转换成哈希——通常是整数；****
*   ****[**一致哈希**](https://en.wikipedia.org/wiki/Consistent_hashing) —如果哈希表的大小被调整，最小化要重新映射的键的数量。(添加或删除服务器时的负载均衡器)( **Rendezvous —最高随机散列**)；****
*   ****[](https://www.encryptionconsulting.com/education-center/what-is-sha/)****——**安全哈希算法，哈希函数的集合。人气 SHA-3。******

# ******数据库******

*   ****[**ACID**](https://www.bmc.com/blogs/acid-atomic-consistent-isolated-durable/)—**database**(数据库事务的):原子性——一个事务中的所有操作都是成功或失败的，不是一个中间状态；一致性—每个事务保证数据处于更新状态(强一致性)，(最终一致性)将在一段时间后更新；隔离——多个事务的执行将与其顺序执行相同；持久性—事务处于非易失状态，不会崩溃****
*   ****[**关系数据库**](https://aws.amazon.com/relational-database/) —以表格、行、列以及它们之间的关系组织起来的数据库。****
*   ****[**非关系数据库**](https://docs.microsoft.com/en-us/azure/architecture/data-guide/big-data/non-relational-data) —非表格，为特定目的而特别组织的。****
*   ****[**SQL**](https://www.w3schools.com/sql/sql_intro.asp) —结构化查询语言；****
*   ******SQL** **数据库—** 支持 SQL 的关系数据库；****
*   ****[**NoSQL**](https://en.wikipedia.org/wiki/NoSQL) —不支持 SQL 的数据库****
*   ****[**数据库索引**](https://en.wikipedia.org/wiki/Database_index) **—** 一种数据结构，改善数据检索，减缓数据写入(因为为列创建索引)；****
*   ****[**强稠度**](https://www.geeksforgeeks.org/eventual-vs-strong-consistency-in-distributed-databases/) —与酸有关。(每笔交易保证更新数据)—PostgreSQL；****
*   ****[](https://www.geeksforgeeks.org/eventual-vs-strong-consistency-in-distributed-databases/)**——(事务最终(以后)会知道更新的数据)；******

# ********键值存储********

*   ******用于*缓存的 NoSQL 数据库*和*动态配置* ( **Etcd** —强一致性，用于领袖选举)、 **Redis** —缓存，限速器)、 **Zookeeper** (强一致性，领袖选举，高可用)；******

# ******专用存储范例******

*   ****[**Blob 存储**](https://www.enterprisestorageforum.com/software/what-is-blob-storage/)**—Blob(二进制大对象)的键值存储通常媒体:图片、音频、视频(谷歌云存储、亚马逊 S3)；******
*   ******[**时间序列数据库**](https://en.wikipedia.org/wiki/Time_series_database) —存储和分析时间索引的数据，持续创建(日志、物联网、股票):**普罗米修斯、InfluxDB、石墨；********
*   ****[**图形数据库**](https://aws.amazon.com/nosql/graph/) —当数据有深度关系连接(社交网络)(Neo4j)****
*   ****[**Cypher**](https://neo4j.com/developer/cypher/)—Neo4j 创造的图形查询语言，比 SQL 简单****
*   ****[**空间数据库**](https://en.wikipedia.org/wiki/Spatial_database) —位置、地图；使用四叉树进行快速查询(地区/位置)；(酒店，地图)****

# ******复制和分片******

*   ****[**复制**](https://www.techopedia.com/definition/1236/replication) —将数据从一台服务器复制到另一台服务器；(减少单点故障，**减少** **延迟**)，主数据库和副本必须更新(同步、异步)；****
*   ****[**分片**](https://www.digitalocean.com/community/tutorials/understanding-database-sharding) —数据分区(当数据库很大时)，将数据库分割成碎片以**增加吞吐量**。(基于客户区域、基于数据类型(支付)、基于列的散列)。如果次优分片策略，会有**热点******

# ******领导者选举—可靠性******

*   ****服务器选举执行主要操作(系统重要操作)的主服务器的过程。并且其他节点在重选后也知道谁是领导者；****
*   ****[**共识算法**](https://en.wikipedia.org/wiki/Consensus_(computer_science)) —选择领导者，同步所有服务器 **Paxos，Raft。Etcd 和 Zookeeper —** 实现领导者选举的键值数据库****

# ******点对点******

*   ****机器分担工作量，增加系统吞吐量。(文件分发)****
*   ****[**八卦协议**](https://en.wikipedia.org/wiki/Gossip_protocol) —机器之间进行通信以分担工作量的过程。将文件分割成数据块(blobs)，创建一个哈希映射来通知哪个对等体拥有哪个数据块。**分布式哈希表。北海巨妖******

# ******轮询和流式传输******

*   ******用例**:聊天、股票、温度****
*   ****[**轮询**](https://en.wikipedia.org/wiki/Polling_(computer_science)) —取间隔数据(API)；****
*   ****[**流**](https://www.geeksforgeeks.org/what-is-web-socket-and-how-it-is-different-from-the-http/)**——**创建一个持续检索(监听)数据的连接(API) Web 套接字****

# ******配置******

*   ******JSON 还是 YAML******
*   ******可以是静态的** —不变的，优点是不变的难以打破系统，缺点是—需要重建****
*   ******动态**(存储在 KV — Redis 中)优点—灵活、应用速度快，缺点—如果不进行测试，很容易破坏系统****

# ******速率限制—安全性、可靠性******

*   ****减少垃圾邮件系统，避免 DoS 和 d DoS(分布式)拒绝服务攻击；****
*   ****由缓存实现，难以使用服务器缓存，因为负载平衡器不知道如何重定向；****
*   ****在反向代理(负载平衡器)中使用外部键值数据库(Redis)；****
*   ****[**速率限制**](https://en.wikipedia.org/wiki/Rate_limiting) 规则可能简单，也可能复杂(基于层)，具体取决于系统；****

# ******记录—可靠性，增长******

*   ****[**日志**](https://en.wikipedia.org/wiki/Logging_(software)) —从事件中收集信息。所有事件都将记录在 STDOUT、STDERR 中。保存到时序数据库: **InfluxDB** ，**普罗米修斯******
*   ****[**监视器**](https://en.wikipedia.org/wiki/Program_process_monitoring) —用 Grafana 可视化；****
*   ****[**警报**](https://sematext.com/blog/monitoring-alerting/) —如果监控系统出现尖峰，则创建警报，例如，在空闲时发布；****
*   ****在系统成长过程中，日志记录非常重要****

# ******发布/订阅模式—流******

*   ****[](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)**发布/订阅模式；******
*   ********用例**:聊天消息、新闻提要、股票价格、通知；******
*   ****保证:**至少一次交付、持久存储、订购、可重复使用；******
*   ****发布者服务器(P) —主题频道(T) —消息—订阅者(P)->(T)—m1—m2-->(S)****
*   ******幂等运算**——不管调用多少次，结果都一样；****
*   ****[**Apache Kafka**](https://kafka.apache.org/)—LinkedIn 的流媒体系统，Google Cloud Sub/Pub 保证**至少一次交付；******

# ******MapReduce —可扩展性大数据******

*   ******文件系统** —组织数据的抽象(层次，树— Unix 文件系统)。**分布式文件系统** —相同但机器间拆分数据— **Google 文件系统**、 **Hadoop** **分布式文件系统；******
*   ****[**MapReduce**](https://en.wikipedia.org/wiki/MapReduce) —分发数据:**快速、高效、容错**；数据集在多台机器之间拆分- > ( **映射**)对于每一个 chunk 到 key:val->(**Shuffle**)reorganize->(**Reduce**)转换为更有意义的数据；****
*   ******先决条件** —我们有分布式文件系统，我们有拆分成块的数据集，在多台机器之间划分，有中央控制(知道发生了什么，并与 map/reduce 工作人员通信)；服务器发送映射数据；****
*   ****如果出现故障，中央控制将重新运行 map/reduce(等幂运算)****

# ******安全******

*   ****[**中间人攻击**](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) —拦截并变异从客户端到服务器的私有 IP 包。加密和 HTTPS 保护；****
*   ****[**对称加密**](https://www.techslang.com/definition/what-is-symmetric-encryption/) —使用一个密钥加密和解密数据。比不对称更快。其算法是**高级加密标准** (AES)的一部分；****
*   ****[**高级加密标准**](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)**——**标准对称加密算法(AES — 128、AES — 192、AES—256)；****
*   ****[**非对称加密**](https://cheapsslsecurity.com/blog/what-is-asymmetric-encryption-understand-with-simple-examples/) —公钥和私钥进行加密和解密。用公钥加密的数据(可以共享)，可能只能用私钥解密(需要安全)。比对称慢；****
*   ****[**HTTPS**](https://en.wikipedia.org/wiki/HTTPS) —安全连接。要求服务器拥有 **SSL 证书**并使用 **TLS** 在服务器-客户端之间通信(加密数据)；****
*   ****[**TLS**](https://en.wikipedia.org/wiki/Transport_Layer_Security) —构建在 TCP 之上的传输层安全协议；****
*   ****[**SSL 证书**](https://www.godaddy.com/help/what-is-an-ssl-certificate-542) —服务器被**认证机构**授予数字证书。包含服务器公钥。在 **HTTPS 连接中建立 **TLS 握手**；******
*   **[**认证机构**](https://en.wikipedia.org/wiki/Certificate_authority)**—****验证公钥证书来源的实体；****
*   ****[**TLS 握手**](https://docs.microsoft.com/en-us/windows/win32/secauthn/tls-handshake-protocol#:~:text=The%20Transport%20Layer%20Security%20(TLS,Handshake%20Protocol%20manages%20the%20following%3A&text=Authentication%20of%20the%20server%20and%20optionally%2C%20the%20client) —建立客户端与服务器之间的安全连接。客户端发送一串随机字节(**客户端 hello** ) - >服务器发送另一串随机字节(**服务器 hello** ) +带公钥的 SSL 证书- >客户端向证书颁发机构验证证书，发送用随机字节公钥串加密的**premaster secret**->客户端和服务器使用客户端 hello、服务器 hello 和 premaster secret 生成会话密钥(使用对称加密)，对通信过程中的所有数据进行加密；****

# ****结论****

****刷新基础只是系统设计面试准备的第一步。接下来是学习一个系统设计面试模板，如果你分享一点点你的爱❤️看起来像 **1000 我就来分享一下👏鼓掌。******

> ****在 medium 和 twitter 上关注我，获取更多面试准备资料和见解。****

****[](https://easy-web.medium.com/subscribe) [## 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。

### 每当维塔利·舍甫琴科发表文章时，就收到一封电子邮件。通过注册，您将创建一个中型帐户，如果您还没有…

easy-web.medium.com](https://easy-web.medium.com/subscribe) [](https://easy-web.medium.com/membership) [## 通过我的推荐链接加入 Medium 维塔利·舍甫琴科

### 作为一个媒体会员，你的会员费的一部分会给你阅读的作家，你可以完全接触到每一个故事…

easy-web.medium.com](https://easy-web.medium.com/membership) [](https://twitter.com/easy_web_org) [## JavaScript 不可用。

### 编辑描述

twitter.com](https://twitter.com/easy_web_org) [](https://www.facebook.com/easyweb.org) [## 简易网页

### 关于 Web 技术的教程、趋势和见解。让每个人都可以使用 web 开发。查尔斯·莫里斯街 374 号

www.facebook.com](https://www.facebook.com/easyweb.org) 

# 了解更多信息

[](/frontend-interview-cheatsheet-that-helped-me-to-get-offer-on-amazon-and-linkedin-cba9584e33c7) [## 🔥帮助我获得亚马逊和 LinkedIn 录用通知的前端面试备忘单

### 如果你正在准备一个前端面试，想快速更新你的领域知识，这个备忘单将…

itnext.io](/frontend-interview-cheatsheet-that-helped-me-to-get-offer-on-amazon-and-linkedin-cba9584e33c7) [](/how-to-become-a-sr-frontend-engineer-in-amazon-without-leetcode-9b7ec604a12) [## 没有 LeetCode 如何成为亚马逊高级前端工程师🔥

### 这不是 clickbait，这是一个关于我如何从讨厌 LeetCode 到在大型科技公司获得多个邀请的故事。

itnext.io](/how-to-become-a-sr-frontend-engineer-in-amazon-without-leetcode-9b7ec604a12) [](/top-18-web-3-0-trends-every-frontend-developer-has-to-follow-in-2022-2861f9b63627) [## 🔥2022 年每个前端开发人员都必须遵循的 18 大 Web 3.0 趋势

### 这个列表将展示 Web 3.0 可以带来的新机会，并可能激发下一个百万美元的想法。😜

itnext.io](/top-18-web-3-0-trends-every-frontend-developer-has-to-follow-in-2022-2861f9b63627)****