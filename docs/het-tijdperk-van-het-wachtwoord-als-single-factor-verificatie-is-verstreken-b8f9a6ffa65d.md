# 单一因素验证后的密码期限

> 原文：<https://itnext.io/het-tijdperk-van-het-wachtwoord-als-single-factor-verificatie-is-verstreken-b8f9a6ffa65d?source=collection_archive---------0----------------------->

![](img/cadfd56084fa3c404529806590e7c7ae.png)

您的密码不符合要求；它必须至少包含十六个字符，包括两个大写字母、四个数字、一个字符和五个重音字符。很多人都知道把你的密码都写在安全的笔记本上。现在，我们的设备上有“钱包”密码箱，如 Wallet、LastPass 和 1Password。虽然(互联网)安全性日益受到人们的关注，但密码的使用仍是相当长的一段时间吗？

在过去的几十年里，已经出现了一些替代传统密码的方法。在我看来，现在是验证的时候了。我认为，一个新的数字时代，它带来的所有安全挑战要求一种新的身份验证方式。因此，在本文中，我将简要介绍传统密码的主要安全陷阱，随后是最新的发展情况和解决此问题的可能方法。

## 人类是最弱的一环

敏感、机密、组织和个人信息的(数字)世界隐藏在密码的大门之外。黑客和邪恶的犯罪分子正在大规模地测试这种安全机制的力量，以获取这些宝贵的信息以获得有利可图的利益。我的同事 tim doets 最近的一篇文章“T0”网络犯罪攻击#1:暴力攻击，详细介绍了几种攻击性的身份验证方法。

即使密码设置得如此安全，我们还是要处理进入的环境的安全问题。举例来说，银行对黑客来说是一个有吸引力的目标，今年早些时候，在一次[对孟加拉中央银行的袭击中，这些数字窃贼勒索了 2，700 万欧元。黑客也可以模仿](https://www.linkit.nl/knowledge-base/125/Een_bank_zonder_firewall_en_verbonden_met_simpele_routers)[公共 WiFi 热点](https://www.linkit.nl/knowledge-base/149/10_tips_voor_veilig_gebruik_openbare_Wi_fi_hotspots)，从而使您下意识地将机密信息(包括密码)直接发送给他们。对于不熟悉的用户来说，它与原始用户没有任何区别

但最大的风险并不是密码的强弱而是人与人之间的差异。忘记注销、接收和下载恶意文件，或简单地释放您的设备(包括密码)，因为您的新同事或邻居的计算机不执行此操作。它仍然可以从你的肩膀上看出来。

## 我们的数据不再安全了吗？

**当前的方法**
虽然传统的密码在顶部输入，但替代方法已经发展。[生物鉴别](https://www.linkit.nl/knowledge-base/120/De_rol_van_biometrie_binnen_multifactor_authenticatie)，如面部扫描、指纹、虹膜或语音识别等安全性越来越普遍。特别是当您解锁智能手机或登录到银行数据时，请按下指纹的可追溯性。但它也受到了像 ellit Williams-hacker(t26)这样的专家的攻击安全专家甚至被告知这可能是最不安全的安全措施(T5)。你今天只留下了几处指纹？

此外，密码可以“哈希”而不是指纹。同样，就密码的[散列 t10 而言，像布鲁斯·施耐尔这样的学者认为，标准应该适应当今的发展。虽然哈希函数在被环境保存之前使用实际的加密来保护密码，但它需要时间来查找该哈希，从而使实际的密码浮出水面。

也是大数据，在其中存储用户行为并进行帐户持有人识别，在](https://books.google.nl/books?id=mU6fpT1sXCoC&lpg=PT12&pg=PT174&redir_esc=y%23v=onepage&q&f=false)[安全性的复杂性](https://www.linkit.nl/knowledge-base/123/Authenticatie_veiliger_maken_met_behulp_van_Big_Data)中起着越来越大的作用。

个人而言，我更喜欢相关系统，多因素身份验证。请记住，在插入个人第二个设备或工具进入要求的环境之前需要执行的其他操作(例如输入密码)。在包含敏感信息(如银行和个人信息)的环境中，通常会出现这种情况。例如，在财务复写之前，通常需要在您的行动电话上输入(tan)程式码，或使用智慧卡或识别码。

**展望未来**
有多种身份验证形式，存在着各种形式的优缺点。根据最近的一项研究，[Gartner report](https://www.gartner.com/doc/3177132/decision-point-architecting-singlefactor-twofactor)发现，通过将生物识别层添加到当前的主导应用程序中，密码保护可以通过组合安全方法获得增强:多因素身份验证。在这里，它不是一个或两个。如果一个黑客通过攻击或者由于身份信息泄漏而获得了信息，那么每一步都需要另外一个独立的步骤。虽然您在任何地方都留下了指纹，并且可能会无意中保护了您的安全，但无论如何，从所有不正确获得的密码中获取这些指纹都是很困难且费时的。

目前，我正在等待这些技术在安全环境体系结构中的发展。另一方面，这种变革和实施带来了新的指导方针和成本，同时也需要随时保持易用性和隐私价值的平衡。

## 结论是

无论是使用暴力攻击、数据泄漏还是工具，密码都已被破解。

除了提供密码之外，这种传统的个人资料保护方式目前仍是最主要的使用方式。依我之见，以密码为基础的保全性，以及上述的替代方案，都不足以取代或取代现有的保全性。

专家们还在计划中讨论安全标准的制定，以及使用生物鉴别作为指纹是否更安全的方法。现在是不是该把密码传给下一代的安全人员了？

在我看来，最先进的解决方案是多层身份验证的努力，最好是密码和生物识别的结合。

尽管我们必须用现有的条带化，并等待多因素身份验证成为标准体系结构。现在，我要在半个小时内把我的 Bol.com 密码下载到我的加密硬盘上。