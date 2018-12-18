---
title: 在 Azure Cloud Shell 中從 Bash 使用 Terraform 進行部署 | Microsoft Docs
description: 在 Azure Cloud Shell 中從 Bash 使用 Terraform 進行部署
services: Azure
documentationcenter: ''
author: tomarcher
manager: routlaw
tags: azure-cloud-shell
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 11/15/2017
ms.author: tarcher
ms.openlocfilehash: 8512c04cb0efc698ca688724c3806291bb02d200
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46947634"
---
# <a name="deploy-with-terraform-from-bash-in-azure-cloud-shell"></a>在 Azure Cloud Shell 中從 Bash 使用 Terraform 進行部署
此文章逐步引導您使用 [Terraform AzureRM 提供者](https://www.terraform.io/docs/providers/azurerm/index.html)建立資源群組。 

[Hashicorp Terraform](https://www.terraform.io/) 是開放原始碼工具，可將 API 制訂到宣告式組態檔，以便與小組成員共用，供其編輯、檢閱和設定版本。 透過 AzureRM API，Microsoft AzureRM 提供者可用來與 Azure Resource Manager 所支援的資源互動。 

## <a name="automatic-authentication"></a>自動驗證
根據預設，Terraform 會安裝在 Cloud Shell 的 Bash 中。 此外，Cloud Shell 會自動驗證預設的 Azure CLI 訂用帳戶，以透過 Terraform Azure 模組部署資源。

Terraform 會使用所設定的預設 Azure CLI 訂用帳戶。 若要更新預設的訂用帳戶，請執行：

```azurecli-interactive
az account set --subscription mySubscriptionName
```

## <a name="walkthrough"></a>逐步介紹
### <a name="launch-bash-in-cloud-shell"></a>啟動 Cloud Shell 中的 Bash
1. 從您的慣用位置啟動 Cloud Shell
2. 確認您慣用的訂用帳戶已設定好

```azurecli-interactive
az account show
```

### <a name="create-a-terraform-template"></a>建立 Terraform 範本
使用您慣用的文字編輯器建立新的 Terraform 範本，並將其命名為 main.tf。

```
vim main.tf
```

將下列程式碼複製/貼到 Cloud Shell。

```
resource "azurerm_resource_group" "myterraformgroup" {
    name = "myRgName"
    location = "West US"
}
```

儲存檔案並結束文字編輯器。

### <a name="terraform-init"></a>Terraform 初始化
執行 `terraform init` 來開始。

```
justin@Azure:~$ terraform init

Initializing provider plugins...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.azurerm: version = "~> 0.2"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

[terraform init 命令](https://www.terraform.io/docs/commands/init.html)可用來初始化包含 Terraform 組態檔的工作目錄。 `terraform init` 命令是在撰寫新的 Terraform 組態或複製版本控制中的現有組態之後，所應該執行的第一個命令。 您可以安全地執行此命令多次。

### <a name="terraform-plan"></a>Terraform plan
預覽要使用 `terraform plan` 由 Terraform 範本所建立的資源。

```
justin@Azure:~$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + azurerm_resource_group.demo
      id:       <computed>
      location: "westus"
      name:     "myRGName"
      tags.%:   <computed>


Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

[terraform plan 命令](https://www.terraform.io/docs/commands/plan.html)可用來建立執行計畫。 Terraform 會執行重新整理 (除非明確停用)，然後判斷必須執行哪些動作才能達成組態檔中指定的所需狀態。 該計畫可使用 -out 來儲存，然後提供給 terraform apply 以確保只會執行預先計畫的動作。

### <a name="terraform-apply"></a>Terraform apply
使用 `terraform apply` 佈建 Azure 資源。

```
justin@Azure:~$ terraform apply
azurerm_resource_group.demo: Creating...
  location: "" => "westus"
  name:     "" => "myRGName"
  tags.%:   "" => "<computed>"
azurerm_resource_group.demo: Creation complete after 0s (ID: /subscriptions/mySubIDmysub/resourceGroups/myRGName)

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

[terraform apply 命令](https://www.terraform.io/docs/commands/apply.html)可用來套用所需的變更以達到組態所需的狀態。

### <a name="verify-deployment-with-azure-cli"></a>使用 Azure CLI 確認部署
執行 `az group show -n myRgName` 以確認該資源已成功佈建。

```azcliinteractive
az group show -n myRgName
```

### <a name="clean-up-with-terraform-destroy"></a>使用 terraform destroy 來清除
使用 [Terraform destroy 命令](https://www.terraform.io/docs/commands/destroy.html)清除所建立的資源群組，以清除 Terraform 所建立的基礎結構。

```
justin@Azure:~$ terraform destroy
azurerm_resource_group.demo: Refreshing state... (ID: /subscriptions/mySubID/resourceGroups/myRGName)

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  - azurerm_resource_group.demo


Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

azurerm_resource_group.demo: Destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 10s elapsed)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 20s elapsed)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 30s elapsed)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 40s elapsed)
azurerm_resource_group.demo: Destruction complete after 45s

Destroy complete! Resources: 1 destroyed.
```

您已成功透過 Terraform 建立 Azure 資源。 請瀏覽後續步驟，以繼續了解 Cloud Shell。

## <a name="next-steps"></a>後續步驟
[了解 Terraform Azure 提供者](https://www.terraform.io/docs/providers/azurerm/#)<br>
[Cloud Shell 中 Bash 的快速入門](quickstart.md)
