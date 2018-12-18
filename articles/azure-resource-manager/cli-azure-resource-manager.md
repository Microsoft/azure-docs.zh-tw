---
title: 使用 Azure CLI 管理資源 | Microsoft Docs
description: 使用 Azure 命令列介面 (CLI) 來管理 Azure 資源和群組
editor: ''
manager: timlt
documentationcenter: ''
author: tfitzmac
services: azure-resource-manager
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: vm-multiple
ms.devlang: na
ms.topic: conceptual
ms.date: 10/06/2017
ms.author: tomfitz
ms.openlocfilehash: dd111c33cbd348a05ed0f0c04f7325347612e54d
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47106307"
---
# <a name="use-the-azure-cli-to-manage-azure-resources-and-resource-groups"></a>使用 Azure CLI 管理 Azure 資源和資源群組

在本文中，您會了解如何使用 Azure CLI 和 Azure Resource Manager 管理您的解決方案。 如果您不熟悉如何使用 Resource Manager，請參閱 [Resource Manager 概觀](resource-group-overview.md)。 本文著重於管理工作。 您將：

1. 建立資源群組
2. 將資源加入資源群組
3. 將標籤加入資源
4. 根據名稱或標籤值查詢資源
5. 套用和移除資源的鎖定
6. 刪除資源群組

本文不會說明如何將 Resource Manager 範本部署到您的訂用帳戶。 如需該資訊，請參閱[使用 Resource Manager 範本與 Azure CLI 來部署資源](resource-group-template-deploy-cli.md)。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

若要在本機安裝並使用 CLI，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。

## <a name="set-subscription"></a>設定訂用帳戶

如果您有多個訂用帳戶，可以切換成其他訂用帳戶。 首先，我們來看看您帳戶的所有訂用帳戶。

```azurecli-interactive
az account list
```

它會傳回您的已啟用與已停用訂用帳戶清單。

```json
[
  {
    "cloudName": "AzureCloud",
    "id": "<guid>",
    "isDefault": true,
    "name": "Example Subscription One",
    "registeredProviders": [],
    "state": "Enabled",
    "tenantId": "<guid>",
    "user": {
      "name": "example@contoso.org",
      "type": "user"
    }
  },
  ...
]
```

請注意，其中一個訂用帳戶已標示為預設。 此訂用帳戶是您目前作業的內容。 若要切換成其他訂用帳戶，請使用 **az account set** 命令提供訂用帳戶名稱。

```azurecli-interactive
az account set -s "Example Subscription Two"
```

若要顯示目前的訂用帳戶內容，請使用無任何參數的 **az account show**：

```azurecli-interactive
az account show
```

## <a name="create-a-resource-group"></a>建立資源群組

將任何資源部署至訂用帳戶之前，您必須建立將包含該資源的資源群組。

若要建立資源群組，請使用 **az group create** 命令。 此命令會使用 **name** 參數來指定資源群組的名稱，並使用 **location** 參數來指定其位置。

```azurecli-interactive
az group create --name TestRG1 --location "South Central US"
```

輸出的格式如下：

```json
{
  "id": "/subscriptions/<subscription-id>/resourceGroups/TestRG1",
  "location": "southcentralus",
  "managedBy": null,
  "name": "TestRG1",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

如果您稍後需要取出資源群組，請使用下列命令：

```azurecli-interactive
az group show --name TestRG1
```

若要取得訂用帳戶中的所有資源群組，請使用：

```azurecli-interactive
az group list
```

## <a name="add-resources-to-a-resource-group"></a>將資源加入資源群組

若要將資源加入資源群組，您可以使用 **az resource create** 命令，或是您建立之資源類型的特定命令 (例如 **az storage account create**)。 您可能會發現使用資源類型的特定命令會比較容易，因為它包含新資源所需的屬性參數。 若要使用 **az resource create**，您必須知道要設定的所有屬性而不需經過提示。

不過，透過指令碼新增資源可能會在日後造成混淆，因為新的資源不在 Resource Manager 範本中。 範本可讓您可靠且重複地部署您的解決方案。

下列命令會建立儲存體帳戶。 請不要使用範例所顯示的名稱，而是為儲存體帳戶提供唯一的名稱。 名稱的長度必須介於 3 到 24 個字元，而且只能使用數字和小寫字母。 如果使用範例所顯示的名稱，您會收到錯誤，因為該名稱已在使用中。

```azurecli-interactive
az storage account create -n myuniquestorage -g TestRG1 -l westus --sku Standard_LRS
```

如果您稍後需要取出此資源，請使用下列命令：

```azurecli-interactive
az storage account show --name myuniquestorage --resource-group TestRG1
```

## <a name="add-a-tag"></a>新增標記

標籤可讓您根據不同的屬性組織您的資源。 例如，您在不同的資源群組中可能有數個資源，屬於相同的部門。 您可以對這些資源套用部門標籤和值，將它們標示為屬於相同的類別目錄。 或者，您可以標記資源是在生產或測試環境中使用。 在本文中，您只會對一個資源套用標籤，但在您的環境中，很有可能會對所有資源套用標籤。

下列命令對您的儲存體帳戶套用兩個標籤︰

```azurecli-interactive
az resource tag --tags Dept=IT Environment=Test -g TestRG1 -n myuniquestorage --resource-type "Microsoft.Storage/storageAccounts"
```

標籤會以一個物件的方式更新。 若要對已經包含標籤的資源新增標籤，請先擷取現有的標籤。 將新的標籤加入包含現有標籤的物件，然後對資源重新套用所有標籤。

```azurecli-interactive
jsonrtag=$(az resource show -g TestRG1 -n myuniquestorage --resource-type "Microsoft.Storage/storageAccounts" --query tags)
rt=$(echo $jsonrtag | tr -d '"{},' | sed 's/: /=/g')
az resource tag --tags $rt Project=Redesign -g TestRG1 -n myuniquestorage --resource-type "Microsoft.Storage/storageAccounts"
```

## <a name="search-for-resources"></a>搜尋資源

使用 **az resource list** 命令取出不同搜尋條件的資源。

* 若要依名稱取得資源，請提供 **name** 參數︰

  ```azurecli-interactive
  az resource list -n myuniquestorage
  ```

* 若要取得資源群組中的所有資源，請提供 **resource-group** 參數︰

  ```azurecli-interactive
  az resource list --resource-group TestRG1
  ```

* 若要取得有某個標籤名稱和值的所有資源，請提供 **tag** 參數︰

  ```azurecli-interactive
  az resource list --tag Dept=IT
  ```

* 若要取得有特定資源類型的所有資源，請提供 **resource-type** 參數︰

  ```azurecli-interactive
  az resource list --resource-type "Microsoft.Storage/storageAccounts"
  ```

## <a name="get-resource-id"></a>取得資源識別碼

許多命令會採用資源識別碼作為參數。 若要取得資源識別碼並且儲存在變數中，請使用：

```azurecli-interactive
webappID=$(az resource show -g exampleGroup -n exampleSite --resource-type "Microsoft.Web/sites" --query id --output tsv)
```

## <a name="lock-a-resource"></a>鎖定資源

當您必須確保重要資源不會意外被刪除或修改時，就可以對資源進行鎖定。 您可以指定 **CanNotDelete** 或 **ReadOnly**。

若要建立或刪除管理鎖定，您必須擁有 `Microsoft.Authorization/*` 或 `Microsoft.Authorization/locks/*` 動作的存取權。 在內建角色中，只有「擁有者」和「使用者存取管理員」被授與這些動作的存取權。

若要套用鎖定，請使用下列命令：

```azurecli-interactive
az lock create --lock-type CanNotDelete --resource-name myuniquestorage --resource-group TestRG1 --resource-type Microsoft.Storage/storageAccounts --name storagelock
```

在上述範例中鎖定的資源要到鎖定移除之後才能刪除。 若要移除鎖定，請使用︰

```azurecli-interactive
az lock delete --name storagelock --resource-group TestRG1 --resource-type Microsoft.Storage/storageAccounts --resource-name myuniquestorage
```

如需設定鎖定的詳細資訊，請參閱[使用 Azure Resource Manager 鎖定資源](resource-group-lock-resources.md)。

## <a name="remove-resources-or-resource-group"></a>移除資源或資源群組
您可以移除資源或資源群組。 當您移除資源群組時，也會移除該資源群組內的所有資源。

* 若要從資源群組中刪除資源，請對於要刪除的資源類型使用 delete 命令。 此命令會刪除資源，但不會刪除資源群組。

  ```azurecli-interactive
  az storage account delete -n myuniquestorage -g TestRG1
  ```

* 若要刪除資源群組和其所有資源，請使用 **az group delete** 命令。

  ```azurecli-interactive
  az group delete -n TestRG1
  ```

這兩個命令都會要求您確認您想要移除資源或資源群組。

## <a name="next-steps"></a>後續步驟
* 若要了解如何建立 Resource Manager 範本，請參閱[編寫 Azure Resource Manager 範本](resource-group-authoring-templates.md)。
* 若要了解部署範本的相關資訊，請參閱[使用 Azure Resource Manager 範本部署應用程式](resource-group-template-deploy-cli.md)。
* 您可以將現有的資源移動到新的資源群組。 如需範例，請參閱 [將資源移至新的資源群組或訂用帳戶](resource-group-move-resources.md)。
* 如需關於企業如何使用 Resource Manager 有效地管理訂用帳戶的指引，請參閱 [Azure 企業 Scaffold - 規定的訂用帳戶治理](/azure/architecture/cloud-adoption-guide/subscription-governance)。