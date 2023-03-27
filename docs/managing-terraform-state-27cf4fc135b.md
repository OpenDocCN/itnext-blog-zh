# 管理地形状态

> 原文：<https://itnext.io/managing-terraform-state-27cf4fc135b?source=collection_archive---------3----------------------->

## 最佳实践和示例

在本文中，我们将研究如何管理 Terraform 状态。首先，我们将介绍什么是 Terraform 状态以及为什么需要它，然后看一些存储、组织和隔离状态文件的最佳实践。然后，我们将继续研究如何利用数据源引用远程状态，最后，如何使用`terraform state`命令来操作状态文件的内容。

![](img/068d9dae81b35ccf4406516a823593cf.png)

照片由[阿维·理查兹](https://unsplash.com/@avirichards?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)在 [Unsplash](https://unsplash.com/s/photos/remote?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) 上拍摄

## 什么是地形状态？

Terraform 在一个状态文件中记录它所创建的资源的信息。这使得 Terraform 能够知道哪些资源在其控制之下，以及何时更新和销毁它们。默认情况下，terraform 状态文件名为`terraform.tfstate`，保存在运行 Terraform 的同一目录中。它是在运行`terraform apply.`之后创建的。这个文件的实际内容是配置中定义的资源的 JSON 格式映射，以及那些存在于您的基础设施中的资源。当 Terraform 运行时，它可以使用这种映射来比较基础设施和代码，并根据需要进行调整。

## 存储状态文件

默认情况下，状态文件存储在运行 Terraform 的本地目录中。如果您使用 Terraform 进行测试或用于个人项目，这是很好的(只要您的状态文件是安全的和备份的！)，但是，当在一个团队中处理 Terraform 项目时，这就成了一个问题，因为多个人需要访问状态文件。此外，当使用自动化和 CI/CD 管道来运行 Terraform 时，状态文件需要是可访问的，并且将许可给予运行管道的服务主体，以便能够访问保存状态文件的存储帐户容器。这使得共享存储成为保存状态文件的理想选择，可以根据需要授予权限。Azure 存储帐户或亚马逊 S3 桶是一个完美的选择。你也可以使用 [Spacelift 之类的工具为你管理你的状态](https://docs.spacelift.io/vendors/terraform/state-management)。

你应该远程存储你的状态文件，而不是在你的本地机器上！然后可以使用`terraform`块中的`backend`块引用远程状态文件的位置(通常在`main.tf`文件中)。

以下示例显示了在 Azure 中使用存储帐户的配置:

```
terraform { backend "azurerm" {
    resource_group_name  = "terraform-rg"
    storage_account_name = "terraformsa"
    container_name       = "terraformstate"
    key                  = "terraform.tfstate"
  }}
```

将状态文件存储在源代码控制中并不是一个好主意。这是因为 Terraform 状态文件包含所有纯文本数据，其中可能包含机密。将秘密存储在除安全位置之外的任何地方都不是一个好主意，并且绝对不应该放在源代码控制中。为了避免这种情况，例如在 Azure 中，可以引用 Azure Key Vault。其次，大多数版本控制系统不允许锁定文件，当多人试图同时访问文件时，这可能会导致问题。最后，在版本控制中存储状态文件可能会引入人为错误，因为每次运行 Terraform 时都需要提取最新的状态文件。这可能导致使用旧的状态文件，并对基础结构进行意外更改。

相反，使用远程后端解决了这些挑战，本机支持锁定，将自动加载状态文件，并且可以使用加密来保护磁盘上和传输中的状态文件。Azure Storage 或亚马逊 S3 等远程后端存储旨在实现高可用性。它们还支持版本控制，因此如果文件损坏，您可以回滚到以前的状态文件。最后，它使用起来很便宜，通常状态文件非常小，通常可以放入空闲层。

## 隔离和组织状态文件

**国家文件应该隔离，以减少“爆炸半径”。**通常，项目被构建在单个文件夹中，并且对所有资源使用单个状态文件。这会立即带来风险，因为配置中的错误可能会更改状态文件，并对所有资源造成不必要的后果。一个更好的方法是为基础设施的各个部分使用多个状态文件。在逻辑上将资源彼此分开，并在后端给它们自己的状态文件，这意味着对一个资源的更改不会影响另一个。不同环境的不同状态文件也是一个好主意。

例如，假设我们的项目中有一个 Azure SQL 数据库、Azure VNET 和 Azure Web App。基础设施在 3 个环境中是相同的:开发、UAT(用户验收测试)和生产。对于每个环境，这 3 种资源中的每一种都有自己的状态文件。这为每个环境提供了强有力的隔离。更进一步，您可能希望在不同的订阅中为每个环境使用完全独立的存储帐户。假设我们将所有状态文件保存在一个存储帐户中，远程后端的容器文件夹结构最终可能看起来像下面的示例:

```
terraformstate--development
  --webapp.tfstate
  --sqldb.tfstate
  --vnet.tfstate--UAT
  --webapp.tfstate
  --sqldb.tfstate
  --vnet.tfstate--production
  --webapp.tfstate
  --sqldb.tfstate
  --vnet.tfstate
```

开发环境中 Azure Web App 的后端配置:

```
terraform { backend "azurerm" {
    resource_group_name  = "terraform-rg"
    storage_account_name = "terraformsa"
    container_name       = "terraformstate"
    key                  = "development\webapp.tfstate"
  }}
```

对开发中的 Azure Web 应用程序进行更改不会影响生产中的 Azure Web 应用程序。

使用 Terraform 工作空间也可以实现隔离。这些在生产环境中不太有用，但有利于快速测试隔离环境。[不鼓励使用工作空间来管理实际环境](https://www.terraform.io/language/state/workspaces#when-to-use-multiple-workspaces.)。Terraform 默认有一个工作空间(称为 default！).状态文件被隔离到每个工作区。

您可以使用`terraform workspace new`命令创建一个新的工作空间。

```
$ terraform workspace new developmentCreated and switched to workspace "development"! You're now on a new, empty workspace. Workspaces isolate their state, so if you run "terraform plan" Terraform will not see any existing state for this configuration.
```

使用工作空间时有用的命令快速参考:

`terraform workspace list` —列出您的工作区。

`terraform workspace new [workspace name]` —创建新的工作区。

`terraform workspace select [workspace name]`-切换到指定的工作区。

## 地形远程状态

`terraform_remote_state`是一个数据源，可用于直接从远程状态文件中获取详细信息。当您需要引用存储在不同状态文件中的配置输出时，这很有用。当在您的配置中定义了一个`output`块时，其内容包含在状态文件中。这些细节可以在项目的其他地方引用。

例如，再次考虑我们有一个 Azure SQL 数据库和 Azure Web 应用程序。这些的配置文件应该设置为使用不同的状态文件。当创建 web 应用程序需要连接的数据库时，我们添加了一个`output`块来公开数据库创建后的结果 ID:

```
output "sqldb_id" {
  value       = azurerm_sql_database.example.id
  description = "Database ID"
}
```

现在，我们可以通过使用`terraform_remote_state`配置数据块，在 Web 应用程序配置代码中引用结果 SQL 数据库 ID:

```
data "terraform_remote_state" "dev_sqldb" {   
  backend = "azurerm"

  config = {     
    storage_account_name = "terraformsa"     
    container_name       = "terraformstate"     
    key                  = "development/sqldb.tfstate"   
  }
}
```

然后在配置中需要的地方引用输出的名称:

```
data.terraform_remote_state.dev_sqldb.outputs.sqldb_id
```

## 操作状态文件

有时有必要直接与状态文件交互，或者检查其内容，当项目被错误导入或不再存在于真实基础结构中时删除项目，或者导入已经存在的项目以将其置于 Terraform 管理之下。

`terraform state`命令可用于执行高级状态管理。所有修改状态的状态管理命令在进行修改之前创建状态的时间戳备份。

有用的`terraform state`命令:

`terraform state list` —列出状态文件的内容。

`terraform state rm` —从状态文件中删除一个项目。

`terraform state show` —显示状态文件中的资源。

要将基础设施中由其他方法创建的现有项目导入状态文件，使其处于地形控制之下，可使用`terraform import`命令。Terraform 文档页面上的每个资源都有一个导入部分，详细说明了如何对特定资源使用该命令。比如在[azure RM _ sql _ database](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/sql_database)docs 页面上，声明可以使用`resource id`导入 SQL 数据库，并给出了一个例子。

```
terraform import azurerm_sql_database.database1 /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.Sql/servers/myserver/databases/database1
```

在使用该命令之前，需要将相应的`resource`块`azurerm_sql_database.database1`写入配置文件，设置与实际基础设施相匹配。导入资源可能非常耗时，因此从一开始就避免手动创建资源通常是一个明智的想法！

## 摘要

理解 Terraform 状态管理和最佳实践是精通 Terraform 的关键。作为一般规则，状态文件应远程存储，并以这样的方式隔离和组织，即对于资源和环境的逻辑组存在单独的状态文件，以便在发生任何错误时减少“爆炸半径”。`terraform_remote_state`数据源可用于从状态文件中引用`outputs`。最后，`terraform state`和`terraform import`命令可以用来操作状态文件的内容。

有关 Terraform 状态的更多信息，请查看 Terraform 文档:

[](https://www.terraform.io/language/state) [## HashiCorp 的 State | Terraform

### 搜索 Terraform 文档 Terraform 必须存储有关托管基础架构和配置的状态。这个…

www.terraform.io](https://www.terraform.io/language/state) 

想要更多的 Terraform 内容？[点击这里查看我在 Terraform 上的其他文章](https://medium.com/@jackwesleyroper/terraform-related-articles-index-52fab8f11a0b)！

干杯！🍻

[](https://www.buymeacoffee.com/jackwesleyroper) [## Jack Roper 正在 Azure、Azure DevOps、Terraform、Kubernetes 和 Cloud tech 上写博客！

### 希望我的博客能帮到你，你会喜欢它的内容！我真的很喜欢写技术内容和分享…

www.buymeacoffee.com](https://www.buymeacoffee.com/jackwesleyroper) 

*最初发表于*[*space lift . io*](https://spacelift.io/)*。*