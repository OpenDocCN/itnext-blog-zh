# Terraform 备忘单

> 原文：<https://itnext.io/terraform-cheat-sheet-3f7c5c55cfbc?source=collection_archive---------0----------------------->

## 命令参考与有用的例子！

有时，您只想直接找到需要与特定工具一起使用的命令，而不必浏览所有文档。在本帖中，我将重点介绍 Terraform CLI 中常用的命令，这样您就可以轻松地直接开始操作了！

![](img/adf5960398ad724195b194ba17340f53.png)

格伦·卡斯滕斯-彼得斯在 [Unsplash](https://unsplash.com/s/photos/list?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上的照片

本命令参考指南是在编写时使用 Terraform 的最新版本 1.1.9 编写的。随着时间的推移，可以添加新的命令和子命令，但不应该有太大的变化。您可以在此获得 Terraform 的最新版本:

[](https://www.terraform.io/downloads) [## HashiCorp 的下载| Terraform

### 软件包管理器 brew tap hashi corp/tap brew install hashi corp/tap/terraform MAC OS 二进制下载带宽由…提供

www.terraform.io](https://www.terraform.io/downloads) 

# 地形命令快速参考

## 内容

*   寻求帮助
*   显示你的地形版本
*   格式化您的 Terraform 代码
*   初始化您的目录
*   下载并安装模块
*   验证您的地形代码
*   规划您的基础设施
*   部署您的基础设施
*   摧毁你的基础设施
*   “污染”或“不污染”你的资源
*   刷新状态文件
*   查看您的状态文件
*   操作您的状态文件
*   将现有基础设施导入 Terraform 状态
*   获取提供商信息
*   管理您的工作区
*   查看您的输出
*   释放工作区的锁定
*   登录和注销远程主机(Terraform Cloud)
*   制作依赖关系图
*   测试你的表情

## **获得帮助**

`terraform -help` —获取可执行的命令列表及描述。可以与任何其他子命令一起使用，以获取更多信息。

`terraform fmt -help` —显示`fmt`命令的帮助选项。

## **显示你的地形版本**

`terraform version` —显示您的 Terraform 的当前版本，并在有更新版本可供下载时通知您。

## **格式化您的地形代码**

这应该是创建配置文件后运行的第一个命令，以确保使用 HCL 标准格式化代码。这使得跟踪更容易，并且有助于协作。

`terraform fmt` —使用 HCL 语言标准格式化您的 Terraform 配置文件。

`terraform fmt --recursive` —也格式化子目录中的文件

`terraform fmt --diff` —显示原始配置文件和格式更改之间的差异。

`terraform fmt --check` —在自动化 CI/CD 管道中很有用，检查标志可用于确保配置文件格式正确，否则退出状态将为非零。如果文件格式正确，退出状态将为零。

## **初始化你的目录**

`terraform init` —为了准备与 Terraform 一起使用的工作目录，`terraform init`命令执行后端初始化、子模块安装和插件安装。

`terraform init -get-plugins=false` —初始化工作目录，不要下载插件。

`terraform init -lock=false` —初始化工作目录，后端迁移时不要持有状态锁。

`terraform init -input=false` —初始化工作目录，禁用交互提示。

`terraform init -migrate-state` —重新配置后端，并尝试迁移任何现有状态。

`terraform init -verify-plugins=false` —初始化工作目录，不验证哈希公司签名的插件

关于`terraform init`命令的详细纲要，请看[我的帖子](https://faun.pub/terraform-init-command-overview-with-quick-usage-examples-752a5719c317)！

## **下载并安装模块**

注意这通常不是必需的，因为这是`terraform init`命令的一部分。

`terraform get` —下载并安装配置所需的模块。

`terraform get -update` —根据可用模块检查已安装模块的版本，如果可用，则安装新版本。

## **验证您的地形代码**

`terraform validate` —验证目录中的配置文件，并且不访问任何远程状态或服务。`terraform init`应该在此命令之前运行。

## **规划您的基础设施**

`terraform plan` —计划将生成一个执行计划，向您显示将采取什么行动，而不实际执行计划的行动。

`terraform plan -out=<path>` —将计划文件保存到给定路径。然后可以传递给`terraform apply`命令。

创建一个摧毁所有物体的计划，而不是通常的行动。

## **部署**你的**基础设施**

`terraform apply` —根据配置文件创建或更新基础架构。默认情况下，将首先生成一个计划，并需要在应用之前进行审批。

`terraform apply -auto-approve` —应用更改，无需以交互方式向计划键入“是”。在自动化 CI/CD 管道中很有用。

`terraform apply <planfilename>` —提供使用`terraform plan -out`命令生成的文件。如果提供，Terraform 将在没有任何确认提示的情况下采取计划中的措施。

`terraform apply -lock=false` —在地形应用操作期间，不要保持状态锁定。如果其他工程师可能对同一工作空间运行并发命令，请谨慎使用。

`terraform apply -parallelism=<n>` —指定并行运行的操作数量。

`terraform apply -var="domainpassword=password123"` —传入一个变量值。

`terraform apply -var-file="varfile.tfvars"` —传入包含在文件中的变量。

`terraform apply -target=”module.appgw.0"` —仅将更改应用于目标资源。

## **摧毁**你的**基础设施**

`terraform destroy` —破坏 Terraform 管理的基础设施。

`terraform destroy -target=”module.appgw.0"` —仅销毁目标资源。

`terraform destroy -auto-approve` —销毁基础设施，无需对计划交互式键入“是”。在自动化 CI/CD 管道中很有用。

## “污染”或“不污染”你的资源

使用`taint`命令将资源标记为未完全运行。它将被删除并重新创建。

`terraform taint vm1.name` —污染指定的资源实例。

`terraform untaint vm1.name` —取消对已经被污染的资源实例的着色。

## **刷新状态文件**

`terraform refresh` —用包含 Terraform 中被管理资源信息的更新元数据修改状态文件。不会修改您的基础设施。

## **查看**你的**状态文件**

`terraform show` —以人类可读的格式显示状态文件。

`terraform show <path to statefile>` —如果您想要读取特定的状态文件，您可以提供该文件的路径。如果没有提供路径，则显示当前状态文件。

## **操纵**你的**状态文件**

`terraform state` —以下子命令之一必须与该命令一起使用，以操作状态文件。

`terraform state list` —列出当前状态文件中跟踪的所有资源。

`terraform state mv` —移动状态中的项目，例如，当您需要告诉 Terraform 项目已被重命名时，这很有用，例如`terraform state mv vm1.oldname vm1.newname`

`terraform state pull > state.tfstate` —获取当前状态并将其输出到本地文件。

`terraform state push` —从本地状态文件更新远程状态。

`terraform state replace-provider hashicorp/azurerm customproviderregistry/azurerm` —替换提供者，这在切换到使用自定义提供者注册表时很有用。

`terraform state rm` —从状态文件中删除指定的实例。在 Terraform 之外手动删除资源时非常有用。

`terraform state show <resourcename>` —显示状态文件中指定的资源。

## 将现有基础设施导入 Terraform 状态

`terraform import vm1.name -i id123` —将 id 为 123 的虚拟机导入到 vm1.name 下配置文件中定义的配置中

`terraform import vm1.name -i id123 -allow-missing-config` —如果配置块不存在，导入并允许。

## **获取提供商信息**

`terraform providers` —显示配置文件中使用的提供者及其要求的树。

## 管理您的工作区

`terraform workspace` —以下子命令之一必须与 workspace 命令一起使用。当工程师想要测试稍微不同的代码版本时，工作区会很有用。不建议使用工作区来隔离或分离不同开发阶段之间的相同基础架构，例如开发/ UAT /生产，或不同的内部团队。

`terraform workspace show` —显示当前工作空间的名称。

`terraform workspace list` —列出您的工作区。

`terraform workspace select <workspace name>` —选择一个指定的工作空间。

`terraform workspace new <workspace name>` —创建一个具有指定名称的新工作区。

`terraform workspace delete <workspace name>` —删除指定的工作空间。

## 查看您的输出

`terraform output` —列出当前保存在状态文件中的所有输出。默认情况下，它们显示在`terraform apply`的末尾，如果您想单独查看它们，该命令会很有用。

`terraform output -state=<path to state file>` —列出保存在指定状态文件中的输出。

`terraform output -json` —以 JSON 格式列出保存在状态文件中的输出，以使它们成为机器可读的。

`terraform output vm1_public_ip` —列出状态文件中保存的特定输出。

## 释放工作区的锁定

`terraform force-unlock <lock_id>` —从您的工作区中删除具有指定锁 ID 的锁。当一个锁被“卡住”时，通常是在不完全的地形运行之后，这是很有用的。

## 登录和注销远程主机(Terraform Cloud)

`terraform login` —使用浏览器获取 terra form cloud(app . terra form . io)的 API 令牌。

`terraform login <hostname>` —登录到指定的主机。

`terraform logout` —默认情况下，对于 terra form Cloud(app . terra form . io)，在登录后删除本地存储的凭据。

`terraform logout <hostname>` —以指定的主机名登录后，删除本地存储的凭据。

## 制作依赖关系图

`terraform graph` —用点语言生成一个图形，显示状态文件中对象之间的依赖关系。这可以通过一个叫做 Graphwiz 的程序来渲染。

`terraform graph -plan=tfplan` —使用指定的计划文件(使用`terraform plan -out=tfplan`生成)生成依赖图。

`terraform graph -type=plan` —指定要输出的图形类型，可以是`plan, plan-refresh-only, plan-destroy,`或`apply`。

## 测试你的表情

`terraform console` —允许使用命令行在交互式控制台上测试和浏览表达式。例如 1+2:)

## 摘要

本指南将帮助您在使用 Terraform CLI 时直接获得所需的命令！

想要更多的 Terraform 内容？点击这里查看我在 Terraform 上的其他文章！

干杯！🍻

[](https://www.buymeacoffee.com/jackwesleyroper) [## Jack Roper 正在 Azure、Azure DevOps、Terraform、Kubernetes 和 Cloud tech 上写博客！

### 希望我的博客能帮到你，你会喜欢它的内容！我真的很喜欢写技术内容和分享…

www.buymeacoffee.com](https://www.buymeacoffee.com/jackwesleyroper)