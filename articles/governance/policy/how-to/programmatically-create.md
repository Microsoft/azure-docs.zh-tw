---
title: 透過 Azure 原則以程式設計方式建立原則並檢視合規性資料
description: 本文會逐步引導您以程式設計方式建立及管理 Azure 原則的原則。
services: azure-policy
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: azure-policy
manager: carmonm
ms.openlocfilehash: dd7ec4f1d0c018a3c7eed19bea523f7c09bfea3e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46985311"
---
# <a name="programmatically-create-policies-and-view-compliance-data"></a>以程式設計方式建立原則並檢視合規性資料

本文會逐步引導您以程式設計方式建立及管理原則。 原則定義會對您的資源強制執行不同規則和影響。 強制作業可確保資源會符合您的公司標準及服務等級協定規範。

如需合規性相關資訊，請參閱[取得合規性資料](getting-compliance-data.md)。

## <a name="prerequisites"></a>必要條件

開始之前，請確定您已符合下列必要條件：

1. 請安裝 [ARMClient](https://github.com/projectkudu/ARMClient) (如果尚未安裝)。 此工具會將 HTTP 要求傳送至以 Azure Resource Manager 為基礎的 API。

1. 將您的 AzureRM PowerShell 模組更新為最新版本。 如需最新版本的詳細資訊，請參閱 [Azure PowerShell](https://github.com/Azure/azure-powershell/releases)。

1. 使用 Azure PowerShell 註冊 Policy Insights 資源提供者，以確保您的訂用帳戶可搭配資源提供者使用。 若要註冊資源提供者，您必須有權執行資源提供者的註冊動作作業。 這項作業包含在「參與者」和「擁有者」角色中。 執行下列命令以註冊資源提供者：

   ```azurepowershell-interactive
   Register-AzureRmResourceProvider -ProviderNamespace 'Microsoft.PolicyInsights'
   ```

   如需註冊及檢視資源提供者的詳細資訊，請參閱[資源提供者和類型](../../../azure-resource-manager/resource-manager-supported-services.md)。

1. 如果尚未安裝 Azure CLI，請先安裝。 您可以於[在 Windows 上安裝 Azure CLI](/cli/azure/install-azure-cli-windows) 中取得最新版本。

## <a name="create-and-assign-a-policy-definition"></a>建立及指派原則定義

要更清楚看見您資源的首要步驟，是透過您的資源建立及指派原則。 下一步是了解如何以程式設計方式建立及指派原則。 範例原則會使用 PowerShell、Azure CLI 和 HTTP 要求，來稽核已對所有公用網路開放的儲存體帳戶。

### <a name="create-and-assign-a-policy-definition-with-powershell"></a>使用 PowerShell 建立並指派原則定義

1. 使用下列 JSON 程式碼片段建立名稱為 AuditStorageAccounts.json 的 JSON 檔案。

   ```json
   {
       "if": {
           "allOf": [{
                   "field": "type",
                   "equals": "Microsoft.Storage/storageAccounts"
               },
               {
                   "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
                   "equals": "Allow"
               }
           ]
       },
       "then": {
           "effect": "audit"
       }
   }
   ```

   如需撰寫原則定義的詳細資訊，請參閱 [Azure 原則定義結構](../concepts/definition-structure.md)。

1. 使用 AuditStorageAccounts.json 檔案，執行下列命令來建立原則定義。

   ```azurepowershell-interactive
   New-AzureRmPolicyDefinition -Name 'AuditStorageAccounts' -DisplayName 'Audit Storage Accounts Open to Public Networks' -Policy 'AuditStorageAccounts.json'
   ```

   此命令會建立名為_對公用網路開放的稽核儲存體帳戶_的原則定義。 如需您可使用的其他參數詳細資訊，請參閱 [New-AzureRmPolicyDefinition](/powershell/module/azurerm.resources/new-azurermpolicydefinition)。

1. 建立原則定義之後，您可以執行下列命令來建立原則指派：

   ```azurepowershell-interactive
   $rg = Get-AzureRmResourceGroup -Name 'ContosoRG'
   $Policy = Get-AzureRmPolicyDefinition -Name 'AuditStorageAccounts'
   New-AzureRmPolicyAssignment -Name 'AuditStorageAccounts' -PolicyDefinition $Policy -Scope $rg.ResourceId
   ```

   以您想要的資源群組名稱取代 _ContosoRG_。

如需使用 Azure Resource Manager PowerShell 模組來管理資源原則的詳細資訊，請參閱 [AzureRM.Resources](/powershell/module/azurerm.resources/#policies)。

### <a name="create-and-assign-a-policy-definition-using-armclient"></a>使用 ARMClient 來建立及指派原則定義

請使用下列程序來建立原則定義。

1. 複製下列 JSON 程式碼片段以建立 JSON 檔案。 您會在下一個步驟中呼叫該檔案。

   ```json
   "properties": {
       "displayName": "Audit Storage Accounts Open to Public Networks",
       "policyType": "Custom",
       "mode": "Indexed",
       "description": "This policy ensures that storage accounts with exposure to Public Networks are audited.",
       "parameters": {},
       "policyRule": {
           "if": {
               "allOf": [{
                       "field": "type",
                       "equals": "Microsoft.Storage/storageAccounts"
                   },
                   {
                       "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
                       "equals": "Allow"
                   }
               ]
           },
           "then": {
               "effect": "audit"
           }
       }
   }
   ```

1. 使用下列其中一個呼叫來建立原則定義︰

   ```
   # For defining a policy in a subscription
   armclient PUT "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyDefinitions/AuditStorageAccounts?api-version=2016-12-01" @<path to policy definition JSON file>

   # For defining a policy in a management group
   armclient PUT "/providers/Microsoft.Management/managementgroups/{managementGroupId}/providers/Microsoft.Authorization/policyDefinitions/AuditStorageAccounts?api-version=2016-12-01" @<path to policy definition JSON file>
   ```

   使用您訂用帳戶的 ID 來取代上述 {subscriptionId}，或使用[管理群組](../../management-groups/overview.md)的 ID 來取代 {managementGroupId}。

   如需查詢結構的詳細資訊，請參閱[原則定義 – 建立或更新](/rest/api/resources/policydefinitions/createorupdate)和[原則定義 – 在管理群組建立或更新](/rest/api/resources/policydefinitions/createorupdateatmanagementgroup)

使用下列程序來建立原則指派，並在資源群組層級指派原則定義。

1. 複製下列 JSON 程式碼片段以建立 JSON 原則指派檔案。 以您自己的值取代 &lt;&gt; 符號中的範例資訊。

   ```json
   {
       "properties": {
           "description": "This policy assignment makes sure that storage accounts with exposure to Public Networks are audited.",
           "displayName": "Audit Storage Accounts Open to Public Networks Assignment",
           "parameters": {},
           "policyDefinitionId": "/subscriptions/<subscriptionId>/providers/Microsoft.Authorization/policyDefinitions/Audit Storage Accounts Open to Public Networks",
           "scope": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>"
       }
   }
   ```

1. 使用下列呼叫建立原則指派︰

   ```
   armclient PUT "/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.Authorization/policyAssignments/Audit Storage Accounts Open to Public Networks?api-version=2017-06-01-preview" @<path to Assignment JSON file>
   ```

   以您自己的值取代 &lt;&gt; 符號中的範例資訊。

   如需對 REST API 執行 HTTP 呼叫的詳細資訊，請參閱 [Azure REST API 資源](/rest/api/resources/)。

### <a name="create-and-assign-a-policy-definition-with-azure-cli"></a>使用 Azure CLI 建立並指派原則定義

若要建立原則定義，請使用下列程序：

1. 複製下列 JSON 程式碼片段以建立 JSON 原則指派檔案。

  ```json
  {
      "if": {
          "allOf": [{
                  "field": "type",
                  "equals": "Microsoft.Storage/storageAccounts"
              },
              {
                  "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
                  "equals": "Allow"
              }
          ]
      },
      "then": {
          "effect": "audit"
      }
  }
  ```

1. 執行下列命令以建立原則定義：

   ```azurecli-interactive
   az policy definition create --name 'audit-storage-accounts-open-to-public-networks' --display-name 'Audit Storage Accounts Open to Public Networks' --description 'This policy ensures that storage accounts with exposures to public networks are audited.' --rules '<path to json file>' --mode All
   ```

1. 使用下列命令以建立原則指派。 以您自己的值取代 &lt;&gt; 符號中的範例資訊。

   ```azurecli-interactive
   az policy assignment create --name '<name>' --scope '<scope>' --policy '<policy definition ID>'
   ```

您可以使用 PowerShell 與下列命令取得原則定義識別碼：

```azurecli-interactive
az policy definition show --name 'Audit Storage Accounts with Open Public Networks'
```

針對您所建立的原則定義，其原則定義識別碼應該類似下列範例：

```
"/subscription/<subscriptionId>/providers/Microsoft.Authorization/policyDefinitions/Audit Storage Accounts Open to Public Networks"
```

如需有關如何使用 Azure CLI 來管理資源原則的詳細資訊，請參閱 [Azure CLI 資源原則](/cli/azure/policy?view=azure-cli-latest)。

## <a name="next-steps"></a>後續步驟

如需本文中查詢與命令的詳細資訊，請檢閱以下文章。

- [Azure REST API 資源](/rest/api/resources/)
- [Azure RM PowerShell 模組](/powershell/module/azurerm.resources/#policies)
- [Azure CLI 原則命令](/cli/azure/policy?view=azure-cli-latest)
- [資源提供者 REST API 參考](/rest/api/policy-insights)
- [使用 Azure 管理群組來組織資源](../../management-groups/overview.md)