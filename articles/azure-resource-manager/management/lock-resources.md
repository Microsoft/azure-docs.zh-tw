---
title: 鎖定資源以防止變更
description: 藉由套用鎖定給所有使用者和角色，防止使用者更新或刪除 Azure 資源。
ms.topic: conceptual
ms.date: 11/11/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 0e8fc74b2da0c253ec9c5bf34ec7543398aea48f
ms.sourcegitcommit: fc8ce6ff76e64486d5acd7be24faf819f0a7be1d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98802438"
---
# <a name="lock-resources-to-prevent-unexpected-changes"></a>鎖定資源以防止非預期的變更

身為系統管理員，您可能需要鎖定訂用帳戶、資源群組或資源，以防止組織中的其他使用者不小心刪除或修改重要資源。 您可以將鎖定層級設定為 **CanNotDelete** 或 **ReadOnly**。 在入口網站中，鎖定分別名為 [刪除] 和 [唯讀]。

* **CanNotDelete** 表示經過授權的使用者仍然可以讀取和修改資源，但無法刪除資源。
* **ReadOnly** 表示經過授權的使用者可以讀取資源，但無法刪除或更新資源。 套用這個鎖定類似於限制所有經過授權使用者的權限是由「讀取者」角色所授與。

## <a name="how-locks-are-applied"></a>如何套用鎖定

當您在父範圍套用鎖定時，該範圍內的所有資源都會都繼承相同的鎖定。 甚至您稍後新增的資源都會繼承父項的鎖定。 繼承中限制最嚴格的鎖定優先順序最高。

不同於角色型存取控制，您可以使用管理鎖定來對所有使用者和角色套用限制。 若要瞭解如何設定使用者和角色的許可權，請參閱 [azure 角色型存取控制 (AZURE RBAC) ](../../role-based-access-control/role-assignments-portal.md)。

Resource Manager 鎖定只會套用於管理平面發生的作業，亦即要傳送至 `https://management.azure.com` 的作業。 鎖定並不會限制資源執行自己函式的方式。 限制資源的變更，但沒有限制資源的作業。 例如，SQL Database 邏輯伺服器上的 ReadOnly 鎖定可防止您刪除或修改伺服器。 它不會讓您無法在該伺服器上的資料庫中建立、更新或刪除資料。 允許資料的交易，因為這些作業不會傳送到 `https://management.azure.com`。

## <a name="considerations-before-applying-locks"></a>套用鎖定前的考量

套用鎖可能導致意外的結果，因為某些看來不會修改資源的作業實際上需要執行被鎖定阻止的動作。 鎖定會防止任何需要對 Azure Resource Manager API 進行 POST 要求的作業。 鎖定阻止的動作常見範例像是：

* **儲存體帳戶** 上的唯讀鎖定會防止所有使用者列出金鑰。 清單金鑰作業是透過 POST 要求進行處理，因為傳回的金鑰可用於寫入作業。

* **App Service** 資源上的唯讀鎖定會防止 Visual Studio 伺服器總管顯示該資源的檔案，因為該互動需要寫入存取權。

* 包含 **虛擬機器** 的 **資源群組** 上的唯讀鎖定，會防止所有使用者啟動或重新開機虛擬機器。 這些作業需要 POST 要求。

* 無法刪除 **資源群組** 上的鎖定，可防止 Azure Resource Manager [自動刪除](../templates/deployment-history-deletions.md) 歷程記錄中的部署。 如果您在歷程記錄中到達800部署，您的部署將會失敗。

* **Azure 備份服務** 所建立之 **資源群組** 上的無法刪除鎖定會導致備份失敗。 服務最多支援 18 個還原點。 當鎖定時，備份服務無法清除還原點。 如需詳細資訊，請參閱[常見問題 - 備份 Azure VM](../../backup/backup-azure-vm-backup-faq.yml)。

* **訂用帳戶** 上的唯讀鎖定會阻止 **Azure Advisor** 正常運作。 Advisor 無法儲存其查詢的結果。

## <a name="who-can-create-or-delete-locks"></a>誰可以建立或刪除鎖定

若要建立或刪除管理鎖定，您必須擁有 `Microsoft.Authorization/*` 或 `Microsoft.Authorization/locks/*` 動作的存取權。 在內建角色中，只有 **擁有者** 和 **使用者存取管理員** 被授與這些動作的存取權。

## <a name="managed-applications-and-locks"></a>受控的應用程式和鎖定

某些 Azure 服務 (例如 Azure Databricks) 使用[受控的應用程式](../managed-applications/overview.md)來實作服務。 在此情況下，服務將建立兩個資源組。 一個資源群組包含服務的概觀，而且不會被鎖定。 另一個資源群組包含服務的基礎結構，並已鎖定。

如果您嘗試刪除基礎結構資源群組，會收到一個錯誤訊息，指出資源組已鎖定。 如果您嘗試刪除基礎結構資源群組的鎖定，就會收到錯誤，指出無法刪除鎖定，因為其是由系統應用程式所擁有。

請改為刪除服務，這也會刪除基礎結構資源群組。

針對受控應用程式，選取您所部署的服務。

![選取服務](./media/lock-resources/select-service.png)

請注意，此服務包含 **受控資源群組** 的連結。 該資源群組會保留基礎結構並加以鎖定。 無法將其直接刪除。

![顯示受控群組](./media/lock-resources/show-managed-group.png)

若要刪除服務的所有資料，包括鎖定的基礎結構資源群組，請選取服務的 [刪除]。

![刪除服務](./media/lock-resources/delete-service.png)

## <a name="configure-locks"></a>設定鎖定

### <a name="portal"></a>入口網站

[!INCLUDE [resource-manager-lock-resources](../../../includes/resource-manager-lock-resources.md)]

### <a name="arm-template"></a>ARM 範本

使用 Azure Resource Manager 範本 (ARM 範本) 部署鎖定時，您必須留意鎖定的範圍和部署的範圍。 若要在部署範圍套用鎖定，例如鎖定資源群組或訂用帳戶，請不要設定 [範圍] 屬性。 鎖定部署範圍內的資源時，請設定 [範圍] 屬性。

下列範本會將鎖定套用至它所部署的資源群組。 請注意，鎖定資源上沒有範圍屬性，因為鎖定的範圍符合部署的範圍。 此範本會部署在資源群組層級。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {  
    },
    "resources": [
        {
            "type": "Microsoft.Authorization/locks",
            "apiVersion": "2016-09-01",
            "name": "rgLock",
            "properties": {
                "level": "CanNotDelete",
                "notes": "Resource Group should not be deleted."
            }
        }
    ]
}
```

若要建立資源群組並加以鎖定，請在訂用帳戶層級部署下列範本。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "rgName": {
            "type": "string"
        },
        "rgLocation": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Resources/resourceGroups",
            "apiVersion": "2019-10-01",
            "name": "[parameters('rgName')]",
            "location": "[parameters('rgLocation')]",
            "properties": {}
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2020-06-01",
            "name": "lockDeployment",
            "resourceGroup": "[parameters('rgName')]",
            "dependsOn": [
                "[resourceId('Microsoft.Resources/resourceGroups/', parameters('rgName'))]"
            ],
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "type": "Microsoft.Authorization/locks",
                            "apiVersion": "2016-09-01",
                            "name": "rgLock",
                            "properties": {
                                "level": "CanNotDelete",
                                "notes": "Resource group and its resources should not be deleted."
                            }
                        }
                    ],
                    "outputs": {}
                }
            }
        }
    ],
    "outputs": {}
}
```

將鎖定套用至資源群組內的 **資源** 時，請新增 [範圍] 屬性。 將 [範圍] 設定為要鎖定之資源的名稱。

下列範例示範建立應用程式服務方案的範本、網站和網站上的鎖定。 鎖定的範圍會設定為網站。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "hostingPlanName": {
      "type": "string"
    },
    "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]"
    }
  },
  "variables": {
    "siteName": "[concat('ExampleSite', uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2020-06-01",
      "name": "[parameters('hostingPlanName')]",
      "location": "[parameters('location')]",
      "sku": {
        "tier": "Free",
        "name": "f1",
        "capacity": 0
      },
      "properties": {
        "targetWorkerCount": 1
      }
    },
    {
      "type": "Microsoft.Web/sites",
      "apiVersion": "2020-06-01",
      "name": "[variables('siteName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', parameters('hostingPlanName'))]"
      ],
      "properties": {
        "serverFarmId": "[parameters('hostingPlanName')]"
      }
    },
    {
      "type": "Microsoft.Authorization/locks",
      "apiVersion": "2016-09-01",
      "name": "siteLock",
      "scope": "[concat('Microsoft.Web/sites/', variables('siteName'))]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites', variables('siteName'))]"
      ],
      "properties": {
        "level": "CanNotDelete",
        "notes": "Site should not be deleted."
      }
    }
  ]
}
```

### <a name="azure-powershell"></a>Azure PowerShell

您可以使用 Azure PowerShell 以 [New-AzResourceLock](/powershell/module/az.resources/new-azresourcelock) 命令鎖定已部署的資源。

若要鎖定資源，請提供資源的名稱、其資源類型，以及其資源群組名稱。

```azurepowershell-interactive
New-AzResourceLock -LockLevel CanNotDelete -LockName LockSite -ResourceName examplesite -ResourceType Microsoft.Web/sites -ResourceGroupName exampleresourcegroup
```

若要鎖定資源群組，請提供資源群組的名稱。

```azurepowershell-interactive
New-AzResourceLock -LockName LockGroup -LockLevel CanNotDelete -ResourceGroupName exampleresourcegroup
```

若要取得鎖定的相關資訊，請使用 [Get-AzResourceLock](/powershell/module/az.resources/get-azresourcelock)。 若要取得訂用帳戶中的所有鎖定，請使用︰

```azurepowershell-interactive
Get-AzResourceLock
```

若要取得資源的所有鎖定，請使用︰

```azurepowershell-interactive
Get-AzResourceLock -ResourceName examplesite -ResourceType Microsoft.Web/sites -ResourceGroupName exampleresourcegroup
```

若要取得資源群組的所有鎖定，請使用︰

```azurepowershell-interactive
Get-AzResourceLock -ResourceGroupName exampleresourcegroup
```

若要刪除資源的鎖定，請使用：

```azurepowershell-interactive
$lockId = (Get-AzResourceLock -ResourceGroupName exampleresourcegroup -ResourceName examplesite -ResourceType Microsoft.Web/sites).LockId
Remove-AzResourceLock -LockId $lockId
```

若要刪除資源群組的鎖定，請使用：

```azurepowershell-interactive
$lockId = (Get-AzResourceLock -ResourceGroupName exampleresourcegroup).LockId
Remove-AzResourceLock -LockId $lockId
```

### <a name="azure-cli"></a>Azure CLI

您可使用 [az lock create](/cli/azure/lock#az-lock-create) 命令，透過 Azure CLI 來鎖定已部署的資源。

若要鎖定資源，請提供資源的名稱、其資源類型，以及其資源群組名稱。

```azurecli
az lock create --name LockSite --lock-type CanNotDelete --resource-group exampleresourcegroup --resource-name examplesite --resource-type Microsoft.Web/sites
```

若要鎖定資源群組，請提供資源群組的名稱。

```azurecli
az lock create --name LockGroup --lock-type CanNotDelete --resource-group exampleresourcegroup
```

若要取得鎖定的相關資訊，請使用 [az lock list](/cli/azure/lock#az-lock-list)。 若要取得訂用帳戶中的所有鎖定，請使用︰

```azurecli
az lock list
```

若要取得資源的所有鎖定，請使用︰

```azurecli
az lock list --resource-group exampleresourcegroup --resource-name examplesite --namespace Microsoft.Web --resource-type sites --parent ""
```

若要取得資源群組的所有鎖定，請使用︰

```azurecli
az lock list --resource-group exampleresourcegroup
```

若要刪除資源的鎖定，請使用：

```azurecli
lockid=$(az lock show --name LockSite --resource-group exampleresourcegroup --resource-type Microsoft.Web/sites --resource-name examplesite --output tsv --query id)
az lock delete --ids $lockid
```

若要刪除資源群組的鎖定，請使用：

```azurecli
lockid=$(az lock show --name LockSite --resource-group exampleresourcegroup  --output tsv --query id)
az lock delete --ids $lockid
```

### <a name="rest-api"></a>REST API

您可以使用[管理鎖定的 REST API](/rest/api/resources/managementlocks)，來鎖定已部署的資源。 此 REST API 可讓您建立及刪除鎖定，以及抓取現有鎖定的相關資訊。

若要建立鎖定，請執行：

```http
PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/locks/{lock-name}?api-version={api-version}
```

範圍可以是訂用帳戶、資源群組或資源。 lock-name 是您想要命名鎖定的任何名稱。 api-version 請使用 **2016-09-01**。

在要求中，包含指定鎖定屬性的 JSON 物件。

```json
{
  "properties": {
  "level": "CanNotDelete",
  "notes": "Optional text notes."
  }
}
```

## <a name="next-steps"></a>後續步驟

* 若要了解如何有邏輯地組織您的資源，請參閱[使用標記來組織您的資源](tag-resources.md)。
* 您可以使用自訂原則，在訂用帳戶內套用限制和慣例。 如需詳細資訊，請參閱[何謂 Azure 原則？](../../governance/policy/overview.md)。
* 如需關於企業如何使用 Resource Manager 有效地管理訂用帳戶的指引，請參閱 [Azure 企業 Scaffold - 規定的訂用帳戶治理](/azure/architecture/cloud-adoption-guide/subscription-governance)。
