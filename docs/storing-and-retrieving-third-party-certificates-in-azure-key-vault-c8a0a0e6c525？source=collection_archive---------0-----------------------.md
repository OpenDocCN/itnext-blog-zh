# 在 Azure Key Vault 中存储和检索第三方证书

> 原文：<https://itnext.io/storing-and-retrieving-third-party-certificates-in-azure-key-vault-c8a0a0e6c525?source=collection_archive---------0----------------------->

![](img/47999568806a6f633863b82a7449b5d1.png)

照片由[克里斯·巴尔巴利斯](https://unsplash.com/@cbarbalis?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

存储、管理和续订证书可能会很麻烦。它们什么时候到期？我把它们存放在哪里？谁可以访问它们，这是如何控制的？我如何跟踪这些证书的过期日期，这样我们就不会因为证书过期而陷入严重的中断([嘿，记住这个](https://www.theverge.com/2020/2/3/21120248/microsoft-teams-down-outage-certificate-issue-status))？

Azure Key Vault 可以生成证书并自动更新它们，这使得上面列出的大多数问题都不是问题。困难在于我们无法控制属于可信第三方的证书的生成和更新过程。幸运的是，Key Vault 让这变得非常简单。

# 支持的文件类型

密钥库支持 PFX 或 PEM 格式的文件。如果你不熟悉术语，不要担心。我将在这里分析这两种格式的区别👇以问答的形式。

> 什么是 PFX？PFX 代表什么？

PFX 代表“个人信息交换”这是 PKCS#12 标准的文件格式。可以把 PKCS 看作是一系列标准，这些标准规定了我们应该如何存储和共享证书、密钥和其他安全数据。这个特殊的标准(#12)说我们应该能够**将私有密钥和公共密钥证书存储在一个单独的加密文件中。**以这种方式生成的文件可以使用`.pfx`或`.p12`作为文件扩展名。

> PEM 是什么？这和 PFX 有什么不同？

PEM 代表“隐私增强邮件”这是一种基于 IETF 在 1993 年定义的同名标准的文件格式。标准本身从未被广泛采用，但文件名和首字母缩略词仍然存在。

许多类型的数据可以存储在 PEM 文件中。该标准定义了证书、私钥和证书签名请求的存储，所有这些都可以存储在——您猜对了——PEM 文件中。

如果`.pem`扩展名表明文件是 PEM 格式的，我们怎么知道里面是什么？这由文件中的单行标题表示，如下所示:

```
-----BEGIN CERTIFICATE-----
```

包含密钥的文件将以`-----BEGIN RSA PRIVATE KEY-----`开头，证书签名请求……好了，你明白了！

PEM 格式文件的扩展名变化很大。文件可以以扩展名`.cer`、`.crt`或`.key`结尾；这些都是 PEM 格式的文件，这里的扩展名是告诉用户文件内容的一种便捷方式。如果您不确定，可以通过阅读文件内容和检查文件头来验证文件中存储的内容。

**通常，PEM 文件包含证书或私钥。**如果使用 openssl 生成自签名证书和私钥，默认情况下，它会将密钥和证书作为两个独立的 PEM 文件发出。

> 所以 PFX 包含私钥和证书，而 PEM 文件可以包含任何东西？

✅没错！PEM 文件非常灵活，可以包含各种数据。正如我前面说过的，通常在一个 PEM 文件中您只会看到一个密钥或一个证书。PFX 将在同一文件中包含证书和私钥。

> 所以…我可以把我的证书(以 PEM 格式)上传到 Key Vault，就这样？

没有❌.您需要上传证书和私钥。因此，PFX 文件更容易导入到密钥库中，因为它们同时包含证书和密钥。如果您将密钥和证书作为单独的 PEM 文件，您必须在将它们导入到密钥库之前将它们合并。我个人还不能将密钥和证书连接成一个文件并上传到 Key Vault。根据我的经验，PFX 是最好的选择。

# 导入证书

![](img/c24904b0ca970dcb118e97f73bfa4527.png)

照片由[布鲁诺·托佐](https://unsplash.com/@brunnotozzo?utm_source=medium&utm_medium=referral)在 [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral) 上拍摄

为了演示这是如何工作的，我将使用 openssl 生成一个简单的自签名证书和私钥:

```
$ openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -subj '/CN=localhost'
Generating a 4096 bit RSA private key
..................................................................++
........................................................................................................................++
writing new private key to 'key.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
```

出现提示时，提供私钥的密码。这保证了至少你的私钥是加密的，即使有人能够得到它。现在让我们使用 Azure CLI 创建一个新的 Azure 密钥库:

```
$ az login
$ az group create --name mykeyvault --location 
$ az keyvault create --location uksouth --name mykeyvault \
  --resource-group MyResourceGroup
```

出于论证的目的，这是当您尝试上载没有相应私钥的证书时发生的情况:

```
$ az keyvault certificate import --vault-name mykeyvault -n mycert -f cert.pemPrivate key is not specified in the specified X.509 PEM certificate content. Please specify private key in the X.509 PEM certificate content.
```

这将失败，因为我们只提供了证书。让我们按照以下步骤将两个 PEM 文件(证书和私钥)合并成一个 PFX:

1.  输入如下所示的 openssl 命令
2.  按 Enter 键后，首先会提示您输入之前设置的密码(key.pem 的通行短语)。这允许 openssl 解密私钥，这样它就可以将它与证书结合起来。
3.  接下来，会要求您设置导出密码。该密码加密生成的 PFX。如果有人获得此密码，他们就可以解密文件并获得您的私钥。选择一个密码，并在出现提示时输入两次。
4.  生成 PFX 后，将其上传到密钥库！

所有这些步骤看起来都像这样:

```
$ openssl pkcs12 -export -out certificate.pfx -inkey key.pem -in cert.pem
Enter pass phrase for key.pem:
Enter Export Password:
Verifying - Enter Export Password:- - - - - -$ az keyvault certificate import --vault-name mykeyvault -n mycert -f certificate.pfx --password "<password>"{
  "attributes": {
    "created": "2020-09-19T22:33:19+00:00",
    "enabled": true,
    "expires": "2021-09-19T22:25:51+00:00",
    "notBefore": "2020-09-19T22:25:51+00:00",
    "recoveryLevel": "Recoverable+Purgeable",
    "updated": "2020-09-19T22:33:19+00:00"
  },
  "cer": "MIIEpDCCAowCCQDnMpQhIoHbwjANBgkqhkiG9w0BAQsFADAUMRIwEAYDVQQDDAlsb2NhbGhvc3QwHhcNMjAwOTE5MjIyNTUxWhcNMjEwOTE5MjIyNTUxWjAUMRIwEAYDVQQDDAlsb2NhbGhvc3QwggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIKAoICAQDDZj...
```

那有效！现在，我们已经将示例证书和密钥导入到了密钥库中。如果我们在门户中查看它，我们可以看到到期日期和指纹:

![](img/28ff9de494057919ce5cc75f64d5fc05.png)

# 正在检索证书

从密钥库中获取证书就像上传证书一样简单。要仅下载证书(意味着*没有*私钥)，运行以下命令:

```
$ az keyvault certificate download --vault-name mykeyvault -n mycert -f downloadedcert.pem
```

如果您想要证书*和*私钥，您可以通过运行下面的代码来检索原始的 PFX。请注意，它是 Base64 编码的，因此我们必须在将文件传递给 openssl 之前先对其进行解码:

```
$ az keyvault secret download --vault-name mykeyvault -n mycert --file downloaded.pfx# Decode from Base64
$ cat downloaded.pfx | base64 -d > decoded.pfx# Verify we can read the file
$ openssl pkcs12 -in decoded.pfx# Retrieve the certificate
$ openssl pkcs12 -in decoded.pfx -out mycert.pem -nokeys -nodes# Retrieve the private key
$ openssl pkcs12 -in decoded.pfx -out mycert.key -nocerts -nodes
```

**注意:当您在 Azure Key Vault 中导入证书时，它会从您的 PFX 文件中删除密码，以及从您的私钥中删除密码！**当系统提示输入导入密码和 PEM 密码时，只需按 Enter 键。然后你就可以阅读这个文件了。

如果要防止证书上的私钥被导出，可以设置一个高级策略配置来实现这一目的:

![](img/9eb9d07e6086bb5bd4035aaa81a29fcc.png)

# 结论

*   将证书导入密钥库时，密码将从文件中删除。这很重要！任何在 PFX 上需要密码的代码都不能使用密码来解密它，因为 Key Vault 会将密码取出并设置为空字符串。
*   **在证书过期之前，Key Vault 不会通知您。你必须自己设置。我目前正在用 Terraform 自动创建一个逻辑应用程序。要在 Azure 门户中做到这一点，[遵循本教程](https://docs.microsoft.com/en-us/azure/key-vault/general/event-grid-logicapps)。**
*   权限通过访问策略进行管理。这些应该尽可能严格地只允许那些需要访问您的证书的人。在撰写本文时，通过 Azure RBAC 控制[数据平面对密钥库资源的访问正在预览中。一旦这被提升到 GA，我会推荐使用 Azure RBAC。](https://docs.microsoft.com/en-us/azure/key-vault/general/secure-your-key-vault#data-plane-and-azure-rbac-preview)
*   如果您有许多证书，您可以查询密钥保管库帐户中的所有证书，并按从早到晚的顺序获取它们的到期日期:

```
$ az keyvault certificate list --vault-name mykeyvault \
  --query "[].{Name:name,Expires:attributes.expires,ID:id,Thumbprint:x509Thumbprint}" \
  -o table | sort -k2 -n -
```

在 Key Vault 中导入和导出第三方证书非常简单。一旦它们被导入，Key Vault 将为您提供所有到期日期的统一概述，并与其他 Azure 产品无缝集成。唯一需要指出的潜在“陷阱”是在从密钥库中导出证书时删除文件和私钥上的密码。因此，您应该绝对确定您的密钥保管库帐户上的访问策略(以及最终的 IAM 角色)尽可能严格，或者您的证书上设置了高级策略配置。