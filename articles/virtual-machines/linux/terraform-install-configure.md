---
title: 安裝和設定 Terraform 以便與 Azure 搭配使用 | Microsoft Docs
description: 了解如何安裝和設定 Terraform 以建立 Azure 資源
services: virtual-machines-linux
documentationcenter: virtual-machines
author: echuvyrov
manager: jeconnoc
editor: na
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 06/19/2018
ms.author: echuvyrov
ms.openlocfilehash: 0943bd1bffb3df7beda97ea0619f1aced4ca3a41
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946777"
---
# <a name="install-and-configure-terraform-to-provision-vms-and-other-infrastructure-into-azure"></a>安裝和設定 Terraform 以在 Azure 中佈建 VM 和的其他基礎結構
 
Terraform 使用[簡易範本化語言](https://www.terraform.io/docs/configuration/syntax.html)，提供簡單的方法來定義、預覽及部署雲端基礎結構。 本文說明在 Azure 中使用 Terraform 來佈建資源的必要步驟。

若要深入了解如何搭配 Azure 使用 Terraform，請造訪 [Terraform 中樞](/azure/terraform)。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Terraform 預設會安裝在 [Cloud Shell](/azure/terraform/terraform-cloud-shell) 中。 如果您選擇在本機安裝 Terraform，請完成下一個步驟，否則請繼續進行[設定 Terraform 以存取 Azure](#set-up-terraform-access-to-azure)。

## <a name="install-terraform"></a>安裝 Terraform

若要安裝 Terraform，請將適合您作業系統的套件[下載](https://www.terraform.io/downloads.html)到個別的安裝目錄中。 此下載包含單一可執行檔，您也應該為其定義全域路徑。 如需如何在 Linux 和 Mac 上設定路徑的指示，請移至[此網頁](https://stackoverflow.com/questions/14637979/how-to-permanently-set-path-on-linux)。 如需如何在 Windows 上設定路徑的指示，請移至[此網頁](https://stackoverflow.com/questions/1618280/where-can-i-set-path-to-make-exe-on-windows)。

使用 `terraform` 命令確認路徑組態。 可用的 Terraform 選項清單會隨即顯示，如下列輸出範例所示：

```bash
azureuser@Azure:~$ terraform
Usage: terraform [--version] [--help] <command> [args]
```

## <a name="set-up-terraform-access-to-azure"></a>設定 Terraform 對 Azure 的 存取權

若要讓 Terraform 將資源佈建至 Azure，請建立 [Azure AD 服務主體](/cli/azure/create-an-azure-service-principal-azure-cli)。 此服務主體會授與您的 Terraform 指令碼，讓您可以在 Azure 訂用帳戶中佈建資源。

如果您有多個 Azure 訂用帳戶，請先使用 [az account show](/cli/azure/account#az-account-show) 查詢帳戶，以取得訂用帳戶識別碼和租用戶識別碼值的清單：

```azurecli-interactive
az account show --query "{subscriptionId:id, tenantId:tenantId}"
```

若要使用選取的訂用帳戶，請使用 [az account set](/cli/azure/account#az-account-set) 為此工作階段設定訂用帳戶。 設定 `SUBSCRIPTION_ID` 環境變數，以保存從您要使用之訂用帳戶傳回的 `id` 欄位值：

```azurecli-interactive
az account set --subscription="${SUBSCRIPTION_ID}"
```

現在，您可以建立要與 Terraform 搭配使用的服務主體。 使用 [az ad sp create-for-rbac]/cli/azure/ad/sp#az-ad-sp-create-for-rbac)，並將 [範圍] 設為您的訂用帳戶，如下所示：

```azurecli-interactive
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/${SUBSCRIPTION_ID}"
```

隨即傳回您的 `appId`、`password`、`sp_name` 與 `tenant`。 記下 `appId` 與 `password`。

## <a name="configure-terraform-environment-variables"></a>設定 Terraform 環境變數

若要設定 Terraform 以使用您的 Azure AD 服務主體，請設定下列環境變數，然後讓 [Azure Terraform 模組](https://registry.terraform.io/modules/Azure)使用這些變數。 如果使用 Azure 公用以外的 Azure 雲端，您也可以設定環境。

- `ARM_SUBSCRIPTION_ID`
- `ARM_CLIENT_ID`
- `ARM_CLIENT_SECRET`
- `ARM_TENANT_ID`
- `ARM_ENVIRONMENT`

您可以使用下列範例殼層指令碼來設定這些變數：

```bash
#!/bin/sh
echo "Setting environment variables for Terraform"
export ARM_SUBSCRIPTION_ID=your_subscription_id
export ARM_CLIENT_ID=your_appId
export ARM_CLIENT_SECRET=your_password
export ARM_TENANT_ID=your_tenant_id

# Not needed for public, required for usgovernment, german, china
export ARM_ENVIRONMENT=public
```

## <a name="run-a-sample-script"></a>執行指令碼範例

在空的目錄中建立檔案 `test.tf`，並在下列指令碼中貼上。

```tf
provider "azurerm" {
}
resource "azurerm_resource_group" "rg" {
        name = "testResourceGroup"
        location = "westus"
}
```

儲存檔案，然後初始化 Terraform 部署。 此步驟會下載建立 Azure 資源群組所需的 Azure 模組。

```bash
terraform init
```

輸出類似於下列範例：

```bash
* provider.azurerm: version = "~> 0.3"

Terraform has been successfully initialized!
```

您可以使用 Terraform 指令碼 `terraform plan` 來預覽要完成的動作。 準備好建立資源群組時，即可套用您的 Terraform 計劃，如下所示：

```bash
terraform apply
```

輸出類似於下列範例：

```bash
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + azurerm_resource_group.rg
      id:       <computed>
      location: "westus"
      name:     "testResourceGroup"
      tags.%:   <computed>

azurerm_resource_group.rg: Creating...
  location: "" => "westus"
  name:     "" => "testResourceGroup"
  tags.%:   "" => "<computed>"
azurerm_resource_group.rg: Creation complete after 1s
```

## <a name="next-steps"></a>後續步驟

在本文中，您已安裝 Terraform 或使用 Cloud Shell 來設定 Azure 認證，以及開始在 Azure 訂用帳戶中建立資源。 若要在 Azure 中建立更完整的 Terraform 部署，請參閱下列文章：

> [!div class="nextstepaction"]
> [使用 Terraform 建立 Azure VM](terraform-create-complete-vm.md)
