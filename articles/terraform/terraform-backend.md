---
title: 使用 Azure 儲存體作為 Terraform 後端
description: 在 Azure 儲存體中儲存 Terraform 狀態的簡介。
services: terraform
author: neilpeterson
ms.service: terraform
ms.topic: article
ms.date: 09/13/2018
ms.author: nepeters
ms.openlocfilehash: 48351dc0351b7a717a610ac9552e11362707e150
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46128002"
---
# <a name="store-terraform-state-in-azure-storage"></a>在 Azure 儲存體中儲存 Terraform 狀態

Terraform 狀態用來與 Terraform 設定協調已部署的資源。 Terraform 可使用狀態來了解要新增、更新或刪除哪些 Azure 資源。 根據預設，只有在執行 *Terraform apply* 時Terraform 狀態才會儲存在本機。 基於以下幾個理由，此設定並非理想的設定：

- 本機狀態在小組或共同作業環境中無法正常運作
- Terraform 狀態可能包含機密資訊
- 將狀態儲存在本機會增加意外刪除的機率

Terraform 包含狀態後端的概念，也就是 Terraform 狀態的遠端儲存體。 使用狀態後端時，狀態檔案會儲存在 Azure 儲存體這類資料存放區中。 本文件詳細說明如何設定及使用 Azure 儲存體做為 Terraform 狀態後端。

## <a name="configure-storage-account"></a>設定儲存體帳戶

若要使用 Azure 儲存體做為後端，必須先建立儲存體帳戶。 可以透過 Azure 入口網站、PowerShell、Azure CLI 或 Terraform 本身建立儲存體帳戶。 使用下列範例，透過 Azure CLI 設定儲存體帳戶。

```azurecli-interactive
#!/bin/bash

RESOURCE_GROUP_NAME=tstate
STORAGE_ACCOUNT_NAME=tstate$RANDOM
CONTAINER_NAME=tstate

# Create resource group
az group create --name $RESOURCE_GROUP_NAME --location eastus

# Create storage account
az storage account create --resource-group $RESOURCE_GROUP_NAME --name $STORAGE_ACCOUNT_NAME --sku Standard_LRS --encryption-services blob

# Get storage account key
ACCOUNT_KEY=$(az storage account keys list --resource-group $RESOURCE_GROUP_NAME --account-name $STORAGE_ACCOUNT_NAME --query [0].value -o tsv)

# Create blob container
az storage container create --name $CONTAINER_NAME --account-name $STORAGE_ACCOUNT_NAME --account-key $ACCOUNT_KEY

echo "storage_account_name: $STORAGE_ACCOUNT_NAME"
echo "container_name: $CONTAINER_NAME"
echo "access_key: $ACCOUNT_KEY"
```

記下儲存體帳戶名稱、容器名稱和儲存體存取金鑰。 設定遠端狀態時需要使用這些值。

## <a name="configure-state-backend"></a>設定狀態後端

執行時 *Terraform init* 會設定 Terraform 狀態後端。 若要設定狀態後端，下列資料則是必要項目。

- storage_account_name - Azure 儲存體帳戶的名稱。
- container_name - Blob 容器的名稱。
- key - 要建立的狀態存放區檔案名稱。
- access_key - 儲存體存取金鑰。

這些值都可以在 Terraform 組態檔或命令列中指定，但還是建議您為 `access_key` 使用環境變數。 使用環境變數可防止系統將金鑰寫入至磁碟。

使用 Azure 儲存體存取金鑰的值建立名為 `ARM_ACCESS_KEY` 的環境變數。

```console
export ARM_ACCESS_KEY=<storage access key>
```

若要進一步保護 Azure 儲存體帳戶存取金鑰，請將它儲存在 Azure Key Vault 中。 接著便可使用類似下列的命令來設定環境變數。 如需 Azure Key Vault 的詳細資訊，請參閱 [Azure Key Vault 文件][azure-key-vault]。

```console
export ARM_ACCESS_KEY=$(az keyvault secret show --name terraform-backend-key --vault-name myKeyVault --query value -o tsv)
```

若要設定 Terraform 使用後端，請在 Terraform 設定中使用一種 *azurerm* 類型加入 *backend* 設定。 將 *storage_account_name*、*container_name* 和 *key* 的值新增至設定區塊。

下列範例設定 Terraform 後端及建立 Azure 資源群組。 請以您環境中的值取代這些值。

```json
terraform {
  backend "azurerm" {
    storage_account_name  = "tstate09762"
    container_name        = "tstate"
    key                   = "terraform.tfstate"
  }
}

resource "azurerm_resource_group" "state-demo-secure" {
  name     = "state-demo"
  location = "eastus"
}
```

現在，使用 *Terraform init* 初始化設定，然後使用 *Terraform apply* 執行設定。 完成時，您可以在 Azure 儲存體 Blob 中找到狀態檔案。

## <a name="state-locking"></a>狀態鎖定

使用 Azure 儲存體 Blob 做為狀態儲存體時，執行任何寫入狀態的作業之前 Blob 會自動鎖定。 此設定可防止多個並行處理狀態作業，並行作業可能會導致損毀。 如需詳細資訊，請參閱 Terraform 文件中的[「狀態鎖定」][terraform-state-lock]。

透過 Azure 入口網站或其他 Azure 管理工具，檢查 Blob 時，可以看到鎖定圖示。

![帶有鎖定圖示的 Azure Blob](media/terraform-backend/lock.png)

## <a name="encryption-at-rest"></a>待用加密

根據預設，儲存在 Azure Blob 中的資料會先經過加密，再保存至儲存體基礎結構。 Terraform 需要狀態時，系統會從後端擷取狀態，並儲存在您開發系統上的記憶體中。 在此設定中，Azure 儲存體中的狀態受到保護，且系統不會將其寫入本機磁碟中。

如需 Azure 儲存體加密的詳細資訊，請參閱[待用資料的 Azure 儲存體服務加密][azure-storage-encryption]。

## <a name="next-steps"></a>後續步驟

如需深入了解 Terraform 後端設定，請參閱 [Terraform 後端文件][terraform-backend]。

<!-- LINKS - internal -->
[azure-key-vault]: ../key-vault/quick-create-cli.md
[azure-storage-encryption]: ../storage/common/storage-service-encryption.md

<!-- LINKS - external -->
[terraform-azurerm]: https://www.terraform.io/docs/backends/types/azurerm.html
[terraform-backend]: https://www.terraform.io/docs/backends/
[terraform-state-lock]: https://www.terraform.io/docs/state/locking.html
