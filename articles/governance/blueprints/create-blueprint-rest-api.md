---
title: 使用 REST API 建立 Azure 藍圖
description: Azure 藍圖可用來建立、定義和部署成品。
services: blueprints
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: quickstart
ms.service: blueprints
manager: carmonm
ms.custom: mvc
ms.openlocfilehash: b873ee869b2044977ebefcfd65331567c24e7ec8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46974199"
---
# <a name="define-and-assign-an-azure-blueprint-with-rest-api"></a>使用 REST API 定義和指派 Azure 藍圖

了解如何在 Azure 中建立及指派藍圖，有助於組織定義一致性的常見模式，且能根據 Resource Manager 範本、原則、安全性等，開發出可重複使用並能快速部署的設定。 在本教學課程中，您將了解如何使用 Azure 藍圖在您的組織中處理藍圖的建立、發佈和指派等常見工作，例如：

> [!div class="checklist"]
> - 建立新藍圖並新增各種支援成品
> - 變更仍在**草稿**狀態的現有藍圖
> - 將藍圖標示為**已發佈**，代表藍圖已可供指派
> - 將藍圖指派給現有的訂用帳戶
> - 檢查指派的藍圖的狀態和進度
> - 移除已指派給訂用帳戶的藍圖

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free)。

## <a name="getting-started-with-rest-api"></a>開始使用 REST API

如果您不熟悉 REST API，請先詳閱 [Azure REST API 參考](/rest/api/azure/)，以對 REST API 有一般的了解，特別是要求 URI 和要求本文。 本文使用這些概念提供使用 Azure 藍圖的方向，而且假設您已具備其使用知識。 [ARMClient](https://github.com/projectkudu/ARMClient) 這類工具及其他元件可自動處理授權，建議初學者使用它們。

### <a name="rest-api-and-powershell"></a>REST API 和 PowerShell

如果您還沒有發出 REST API 呼叫的工具，請考慮使用 PowerShell 取得這些指示。 以下是向 Azure 進行驗證的範例標頭。 產生驗證標頭 (有時也稱為**持有人權杖**)，並提供 REST API URI 以連接到任何參數或**要求本文**:

```powershell-interactive
# Login first with Connect-AzureRmAccount if not using Cloud Shell

$azContext = Get-AzureRmContext
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = New-Object -TypeName Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient -ArgumentList ($azProfile)
$token = $profileClient.AcquireAccessToken($azContext.Subscription.TenantId)
$authHeader = @{
    'Content-Type'='application/json'
    'Authorization'='Bearer ' + $token.AccessToken
}

# Invoke the REST API
$restUri = 'https://management.azure.com/subscriptions/{subscriptionId}?api-version=2016-06-01'
$response = Invoke-RestMethod -Uri $restUri -Method Get -Headers $authHeader
```

取代上面 **$restUri** 變數中的 `{subscriptionId}`，以取得您訂用帳戶的相關資訊。 $Response 變數會保存 `Invoke-RestMethod` Cmdlet 的結果，它可使用 [Convertfrom-json](/powershell/module/microsoft.powershell.utility/convertfrom-json) 這類的 Cmdlet 加以剖析。 如果 REST API 服務端點應該有**要求本文**，請提供一個 JSON 格式的變數給 `Invoke-RestMethod` 的 `-Body` 參數。

## <a name="create-a-blueprint"></a>建立藍圖

定義合規性標準模式的第一個步驟，即是以可用的資源規劃藍圖。 在此範例中，會建立名為「MyBlueprint」的藍圖，用以設定訂用帳戶的角色和原則指派、新增資源群組、在資源群組上建立 Resource Manager 的範本和角色指派。

> [!NOTE]
> 使用 REST API 時，會先建立_藍圖_物件。 對於要新增的具有參數的每個_成品_，需要在初始_藍圖_上預先定義參數。

在每個 REST API URI 中有一些變數，需要您以自己的值取代它們：

- `{YourMG}` - 以您的管理群組名稱取代
- `{subscriptionId}` - 以您的訂用帳戶 ID 取代

1. 建立初始_藍圖_物件。 **要求本文**包含藍圖的屬性、要建立的任何資源群組，以及所有藍圖層級參數，它們是在指派期間設定，而且稍後步驟中新增的成品會使用它們。

   - REST API URI

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint?api-version=2017-11-11-preview
     ```

   - 要求本文

     ```json
     {
         "properties": {
             "description": "This blueprint sets tag policy and role assignment on the subscription, creates a ResourceGroup, and deploys a resource template and role assignment to that ResourceGroup.",
             "targetScope": "subscription",
             "parameters": {
                 "storageAccountType": {
                     "type": "string",
                     "metadata": {
                         "displayName": "storage account type.",
                         "description": null
                     }
                 },
                 "tagName": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The name of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "tagValue": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The value of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "contributors": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Contributor role at the subscription"
                     }
                 },
                 "owners": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Owner role at the resource group"
                     }
                 }
             },
             "resourceGroups": {
                 "storageRG": {
                     "description": "Contains the resource template deployment and a role assignment."
                 }
             }
         }
     }
     ```

1. 在訂用帳戶中新增角色指派。 **要求本文**定義成品的_種類_，屬性對應至角色定義識別碼，而且主體身分識別當作值的陣列傳遞。 在下列範例中，授與指定角色的主體身分識別是設定給藍圖指派期間設定的參數。

   - REST API URI

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/roleContributor?api-version=2017-11-11-preview
     ```

   - 要求本文

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
             "principalIds": "[parameters('contributors')]"
         }
     }
     ```

1. 在訂用帳戶中新增原則指派。 **要求本文**定義成品的_種類_、對應至原則或方案定義的屬性，以及設定用來定義藍圖參數 (藍圖指派期間設定) 的原則指派。

   - REST API URI

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/policyTags?api-version=2017-11-11-preview
     ```

   - 要求本文

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "description": "Apply tag and its default value to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "[parameters('tagName')]"
                 },
                 "tagValue": {
                     "value": "[parameters('tagValue')]"
                 }
             }
         }
     }
     ```

1. 在訂用帳戶為儲存標記新增另一個儲存體標記 (重複使用 _storageAccountType_ 參數)。 這個新增的原則指派成品示範藍圖上定義的參數可用於一個以上的成品。 在範例中，**storageAccountType** 用來在資源群組上設定一個標記，提供下個步驟中建立的儲存體帳戶的相關資訊。

   - REST API URI

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/policyStorageTags?api-version=2017-11-11-preview
     ```

   - 要求本文

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "description": "Apply storage tag and the parameter also used by the template to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "StorageType"
                 },
                 "tagValue": {
                     "value": "[parameters('storageAccountType')]"
                 }
             }
         }
     }
     ```

1. 在資源群組下新增範本。 Resource Manager 範本的**要求本文** 包含範本的一般 JSON 元件，使用 **properties.resourceGroup** 定義目標資源群組，並將 **storageAccountType**、**tagName** 和 **tagValue** 藍圖參數提供給範本以重複使用它們。 藍圖參數是透過定義 **properties.parameters** 來提供給範本使用，而在 JSON 範本內，索引鍵/值是用來插入值。 藍圖和範本參數的名稱可以相同，但是用不同的名字是為了說明每個物件是如何從藍圖傳遞到範本成品。

   - REST API URI

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/templateStorage?api-version=2017-11-11-preview
     ```

   - 要求本文

     ```json
     {
         "kind": "template",
         "properties": {
             "template": {
                 "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                 "contentVersion": "1.0.0.0",
                 "parameters": {
                     "storageAccountTypeFromBP": {
                         "type": "string",
                         "defaultValue": "Standard_LRS",
                         "allowedValues": [
                             "Standard_LRS",
                             "Standard_GRS",
                             "Standard_ZRS",
                             "Premium_LRS"
                         ],
                         "metadata": {
                             "description": "Storage Account type"
                         }
                     },
                     "tagNameFromBP": {
                         "type": "string",
                         "defaultValue": "NotSet",
                         "metadata": {
                             "description": "Tag name from blueprint"
                         }
                     },
                     "tagValueFromBP": {
                         "type": "string",
                         "defaultValue": "NotSet",
                         "metadata": {
                             "description": "Tag value from blueprint"
                         }
                     }
                 },
                 "variables": {
                     "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
                 },
                 "resources": [{
                     "type": "Microsoft.Storage/storageAccounts",
                     "name": "[variables('storageAccountName')]",
                     "apiVersion": "2016-01-01",
                     "tags": {
                        "[parameters('tagNameFromBP')]": "[parameters('tagValueFromBP')]"
                     },
                     "location": "[resourceGroup().location]",
                     "sku": {
                         "name": "[parameters('storageAccountTypeFromBP')]"
                     },
                     "kind": "Storage",
                     "properties": {}
                 }],
                 "outputs": {
                     "storageAccountSku": {
                         "type": "string",
                         "value": "[variables('storageAccountName')]"
                     }
                 }
             },
             "resourceGroup": "storageRG",
             "parameters": {
                 "storageAccountTypeFromBP": {
                     "value": "[parameters('storageAccountType')]"
                 },
                 "tagNameFromBP": {
                     "value": "[parameters('tagName')]"
                 },
                 "tagValueFromBP": {
                     "value": "[parameters('tagValue')]"
                 }
             }
         }
     }
     ```

1. 在資源群組下新增角色指派。 類似於先前的角色指派項目，下列範例為**擁有者**角色使用定義識別碼，並為它提供藍圖的不同參數。

   - REST API URI

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/roleOwner?api-version=2017-11-11-preview
     ```

   - 要求本文

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "resourceGroup": "storageRG",
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
             "principalIds": "[parameters('owners')]"
         }
     }
     ```

## <a name="publish-a-blueprint"></a>發佈藍圖

將成品新增到藍圖之後，即可發佈藍圖。 發佈後即可將藍圖指派給訂用帳戶。

- REST API URI

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/versions/{BlueprintVersion}?api-version=2017-11-11-preview
  ```

`{BlueprintVersion}` 的值是字母、數字和連字號組成的字串 (無空格或其他特殊字元)，最大長度為 20 個字元。 使用如 **v20180622 135541** 這類唯一且包含資訊的字串。

## <a name="assign-a-blueprint"></a>指派藍圖

使用 REST API 發佈藍圖後，即可將藍圖指派給訂用帳戶。 將您建立的藍圖指派給管理群組階層下的其中一個訂用帳戶。 **要求本文**指定要指派的藍圖，提供名稱和位置給藍圖定義中的任何資源群組，以及提供在藍圖中定義且用於一或多個連接成品的所有參數。

1. 在目標訂用帳戶上提供 Azure 藍圖服務主題**擁有者**角色。 AppId 是靜態的 (`f71766dc-90d9-4b7d-bd9d-4499c4331c3f`)，但服務主體識別碼則依租用戶而各為不同。 使用下列 REST API 可要求租用戶的詳細資料。 它使用具有不同授權的 [Azure Active Directory 圖形 API](../../active-directory/develop/active-directory-graph-api.md)。

   - REST API URI

     ```http
     GET https://graph.windows.net/{tenantId}/servicePrincipals?api-version=1.6&$filter=appId eq 'f71766dc-90d9-4b7d-bd9d-4499c4331c3f'
     ```

1. 將藍圖部署指派給訂用帳戶以執行它。 因為**參與者**和**擁有者**參數需要主體的 objectId 陣列被授與角色指派，所以請使用 [Azure Active Directory 圖形 API](../../active-directory/develop/active-directory-graph-api.md) 收集 objectId 以用於您自己的使用者、群組或服務主題的**要求本文**中。

   - REST API URI

     ```http
     PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Blueprint/blueprintAssignments/assignMyBlueprint?api-version=2017-11-11-preview
     ```

   - 要求本文

     ```json
     {
         "properties": {
             "blueprintId": "/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint",
             "resourceGroups": {
                 "storageRG": {
                     "name": "StorageAccount",
                     "location": "eastus2"
                 }
             },
             "parameters": {
                 "storageAccountType": {
                     "value": "Standard_GRS"
                 },
                 "tagName": {
                     "value": "CostCenter"
                 },
                 "tagValue": {
                     "value": "ContosoIT"
                 },
                 "contributors": {
                     "value": [
                         "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
                         "38833b56-194d-420b-90ce-cff578296714"
                     ]
                 },
                 "owners": {
                     "value": [
                         "44254d2b-a0c7-405f-959c-f829ee31c2e7",
                         "316deb5f-7187-4512-9dd4-21e7798b0ef9"
                     ]
                 }
             }
         },
         "identity": {
             "type": "systemAssigned"
         },
         "location": "westus"
     }
     ```

## <a name="unassign-a-blueprint"></a>取消指派藍圖

如果不再需要藍圖，或因為模式、原則、設計有所更新，而已經用較新的藍圖取代舊藍圖，可以將藍圖從訂用帳戶中移除。 移除藍圖時，會將指派為該藍圖一部份的成品保留下來。 若要移除藍圖指派，請使用下列 REST API 作業：

- REST API URI

  ```http
  DELETE https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Blueprint/blueprintAssignments/assignMyBlueprint?api-version=2017-11-11-preview
  ```

## <a name="delete-a-blueprint"></a>刪除藍圖

若要移除藍圖本身，請使用下列 REST API 作業：

- REST API URI

  ```http
  DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint?api-version=2017-11-11-preview
  ```

## <a name="next-steps"></a>後續步驟

- 深入了解[藍圖生命週期](./concepts/lifecycle.md) (英文)
- 了解如何使用[靜態和動態參數](./concepts/parameters.md) (英文)
- 了解如何自訂[藍圖排序順序](./concepts/sequencing-order.md) (英文)
- 了解如何使用[藍圖資源鎖定](./concepts/resource-locking.md) (英文)
- 瞭解如何[更新現有的指派](./how-to/update-existing-assignments.md) (英文)
- 使用[一般疑難排解](./troubleshoot/general.md)來解決藍圖指派期間發生的問題