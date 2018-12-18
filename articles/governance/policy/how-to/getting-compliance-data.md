---
title: 取得 Azure 原則中的合規性資料
description: Azure 原則評估和效果會決定合規性。 了解如何取得合規性詳細資料。
services: azure-policy
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: azure-policy
manager: carmonm
ms.custom: mvc
ms.openlocfilehash: 3fa185e741f1b14bf3f2e7413945b70b1ea1baaa
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970850"
---
# <a name="getting-compliance-data"></a>取得合規性資料

Azure 原則的其中一個最大優點，就是能夠針對訂用帳戶中的資源或訂用帳戶的[管理群組](../../management-groups/overview.md)，提供相關見解和控制權。 此控制權可運用在許多不同方面，例如防止在錯誤的位置建立資源、強制執行通用且一致的標記使用方式，或稽核現有的資源以取得適當的組態和設定。 在所有情況下，原則都會產生資料，讓您了解環境的合規性狀態。

有數種方式可存取原則和計畫指派所產生的合規性資訊：

- 使用 [Azure 入口網站](#portal)
- 透過[命令列](#command-line)指令碼

在查看這些報告合規性的方法之前，讓我們來看何時會更新合規性資訊，以及觸發評估週期的頻率和事件。

> [!WARNING]
> 如果合規性狀態報告為**未註冊**，請確認您已註冊 **Microsoft.PolicyInsights** 資源提供者，且使用者有適當的角色型存取控制 (RBAC) 權限，如[此處](../overview.md#rbac-permissions-in-azure-policy)所述。

## <a name="evaluation-triggers"></a>評估觸發程序

完成的評估週期結果會透過 `PolicyStates` 和 `PolicyEvents` 作業反映在 `Microsoft.PolicyInsights` 資源提供者中。 如需原則見解 REST API 之選項和功能的詳細資訊，請參閱[原則見解](/rest/api/policy-insights/)。

下列各種事件都會導致評估指派的原則和計畫：

- 將新的原則或計畫指派給範圍。 發生此情況時，需要約 30 分鐘的時間讓指派套用至定義的範圍。 套用之後，就會對新指派的原則或計畫開始該範圍內資源的評估週期，並根據原則或計畫所使用的效果，將資源標示為相容或不相容。 對範圍很大的資源評估大型原則或計畫可能需要一些時間，因此沒有預期何時將完成評估週期的預先定義。 完成之後，會在入口網站和 SDK 中提供更新的合規性結果。
- 更新已指派給範圍的原則或計畫。 此案例的評估週期和時間與範圍的新指派相同。
- 資源會透過 Resource Manager、REST、Azure CLI 或 Azure PowerShell 部署到具有指派的範圍。 在此案例中，個別資源的效果事件 (附加、稽核、拒絕、部署) 和合規性狀態資訊，約 15 分鐘後會在入口網站和 SDK 中變成可用。 此事件不會造成對其他資源的評估。
- 標準合規性評估週期。 每 24 小時會自動重新評估指派一次。 對範圍很大的資源評估大型原則或計畫可能需要一些時間，因此沒有預期何時將完成評估週期的預先定義。 完成之後，會在入口網站和 SDK 中提供更新的合規性結果。

## <a name="how-compliance-works"></a>合規性的運作方式

在指派中，如果資源沒有遵循原則或方案規則，則該資源**不符合規範**。 下表顯示不同的原則效果如何與結果合規性狀態的條件評估搭配使用：

| 資源狀態 | 效果 | 原則評估 | 合規性狀態 |
| --- | --- | --- | --- |
| exists | 拒絕、稽核、附加\*、DeployIfNotExist\*、AuditIfNotExist\* | True | 不相容 |
| exists | 拒絕、稽核、附加\*、DeployIfNotExist\*、AuditIfNotExist\* | False | 相容 |
| 新增 | 稽核、AuditIfNotExist\* | True | 不相容 |
| 新增 | 稽核、AuditIfNotExist\* | False | 相容 |

\* Append、DeployIfNotExist 和 AuditIfNotExist 效果需要 IF 陳述式為 TRUE。
這些效果也需要存在條件為 FALSE，以呈現不符合規範。 若為 TRUE，IF 條件會觸發相關資源的存在條件評估。

例如，假設您有資源群組 – ContosoRG，且部分的儲存體帳戶 (以紅色醒目提示) 會公開至公用網路。

![公開至公用網路的儲存體帳戶](../media/getting-compliance-data/resource-group01.png)

在此範例中，您必須注意安全性風險。 現在您已建立原則指派，其會針對 ContosoRG 資源群組中的所有儲存體帳戶來進行評估。 其會稽核三個不符合規範的儲存體帳戶，並因此將其狀態變更為**不符合規範**。

![已稽核不符合規範的儲存體帳戶](../media/getting-compliance-data/resource-group03.png)

除了**符合規範**和**不符合規範**以外，原則和資源有其他三種狀態：

- **衝突**：存在兩個或多個規則有衝突的原則 (例如，兩個原則附加使用不同值的相同標籤）。
- **未啟動**：尚未針對原則或資源開始評估週期。
- **未註冊**：Azure 原則資源提供者尚未註冊，或者登入的帳戶無權讀取合規性資料。

原則會使用原則規則定義中的 **type** 和 **name** 欄位來判斷資源是否相符。 如果是，則會被視為適用，且狀態為**符合規範**或**不符合規範**。 如果 **type** 或 **name** 是原則規則定義中的唯一屬性，則所有資源都會視為適用並將進行評估。

合規性百分比是透過將**符合規範**資源除以_總資源_來決定的。
_總資源_定義為**符合規範**、**不符合規範**和**衝突**資源的總和。 整體的合規性數字是**符合規範**之不同資源的總和除以所有不同資源的總和。 在下圖中，有 20 種適用的不同資源，只有一種是**不符合規範**。 因此，整體資源合規性為 19/20 或 95%。

![簡單的合規性範例](../media/getting-compliance-data/simple-compliance.png)

## <a name="portal"></a>入口網站

Azure 入口網站示範視覺化並了解您環境中合規性狀態的圖形化體驗。 在 [原則] 頁面上，[概觀] 選項提供有關原則和計畫合規性之可用範圍的詳細資料。 除了每個指派的合規性狀態和計數，還包含一個圖表，顯示過去七天的合規性。 [合規性] 頁面包含大部分相同的資訊 (圖表除外)，但提供額外的篩選和排序選項。

![原則合規性頁面](../media/getting-compliance-data/compliance-page.png)

您可以將一個原則或計畫指派給不同的範圍，但請注意表格中每個指派的範圍，以及指派給該範圍的類型定義。 也會提供每個指派的不符合規範的資源和不符合規範的原則數目。 按一下表格中的原則或計畫可讓您更深入地查看該特定指派的合規性。

![原則合規性詳細資料](../media/getting-compliance-data/compliance-details.png)

[資源合規性] 索引標籤上的資源清單 (預設為**不符合規範**，但您可以篩選) 反映目前指派之現有資源的評估狀態，建立資源的要求所觸發的事件 (附加、稽核、拒絕、部署) 則會顯示在 [事件] 索引標籤下方。

![原則合規性事件](../media/getting-compliance-data/compliance-events.png)

以滑鼠右鍵按一下您想要收集更多詳細資料的事件資料列，然後選取 [顯示活動記錄]。 活動記錄頁面隨即開啟，並會預先篩選至顯示指派和事件詳細資料的搜尋結果。 活動記錄檔提供有關這些事件的其他內容和資訊。

![原則合規性活動記錄](../media/getting-compliance-data/compliance-activitylog.png)

## <a name="command-line"></a>命令列

您可以直接使用 REST API (隨附於 [ARMClient](https://github.com/projectkudu/ARMClient)) 或 Azure PowerShell 搭配 REST API，來擷取入口網站中所提供的相同資訊。 如需 REST API 的完整詳細資訊，請參閱[原則見解](/rest/api/policy-insights/)參考。 REST API 參考頁面上有每個作業的 [試用] 綠色按鈕，可讓您直接在瀏覽器中試用。

若要在 Azure PowerShell 中使用下列範例，請透過此範例程式碼建構驗證權杖。 然後在範例中將 $restUri 取代為所需的字串，以擷取稍後可剖析的 JSON 物件。

```azurepowershell-interactive
# Login first with Connect-AzureRmAccount if not using Cloud Shell

$azContext = Get-AzureRmContext
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = New-Object -TypeName Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient -ArgumentList ($azProfile)
$token = $profileClient.AcquireAccessToken($azContext.Subscription.TenantId)
$authHeader = @{
    'Content-Type'='application/json'
    'Authorization'='Bearer ' + $token.AccessToken
}

# Define the REST API to communicate with
# Use double quotes for $restUri as some endpoints take strings passed in single quotes
$restUri = "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/summarize?api-version=2018-04-04"

# Invoke the REST API
$response = Invoke-RestMethod -Uri $restUri -Method POST -Headers $authHeader

# View the response object (as JSON)
$response
```

### <a name="summarize-results"></a>摘要結果

使用 REST API，即可依管理群組、訂用帳戶、資源群組、資源、計畫、原則、訂用帳戶層級指派或資源群組層級指派執行摘要。 以下是使用原則見解的[摘要訂用帳戶](/rest/api/policy-insights/policystates/summarizeforsubscription)在訂用帳戶層級摘要的範例：

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/summarize?api-version=2018-04-04
```

此輸出會摘要訂用帳戶。 在下列範例輸出中，摘要的合規性位於 **value.results.nonCompliantResources** 和 **value.results.nonCompliantPolicies** 下方。 此要求提供進一步詳細資料，包括組成不相容數目的每個指派及每個指派的定義資訊。 階層中的每個原則物件提供可在該層級用來取得其他詳細資料的 **queryResultsUri**。

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#summary",
    "@odata.count": 1,
    "value": [{
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#summary/$entity",
        "results": {
            "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2018-04-04&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=IsCompliant eq false",
            "nonCompliantResources": 15,
            "nonCompliantPolicies": 1
        },
        "policyAssignments": [{
            "policyAssignmentId": "/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77",
            "policySetDefinitionId": "",
            "results": {
                "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2018-04-04&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=IsCompliant eq false and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77'",
                "nonCompliantResources": 15,
                "nonCompliantPolicies": 1
            },
            "policyDefinitions": [{
                "policyDefinitionReferenceId": "",
                "policyDefinitionId": "/providers/microsoft.authorization/policydefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "effect": "deny",
                "results": {
                    "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2018-04-04&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=IsCompliant eq false and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77' and PolicyDefinitionId eq '/providers/microsoft.authorization/policydefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62'",
                    "nonCompliantResources": 15
                }
            }]
        }]
    }]
}
```

### <a name="query-for-resources"></a>查詢資源

使用上述範例，**value.policyAssignments.policyDefinitions.results.queryResultsUri** 會提供範例 URI，以取得特定原則定義的所有不相容資源。 檢視 **$filter** 值，IsCompliant 會等於 (eq) false，而 PolicyAssignmentId 會指定給原則定義，再指定給 PolicyDefinitionId 本身。
在篩選中包含 PolicyAssignmentId 的原因，是因為 PolicyDefinitionId 可能存在於各種範圍的數個原則或計畫指派中。 藉由指定 PolicyAssignmentId 和 the PolicyDefinitionId，我們就可以明確指出要尋找的結果。 之前針對 PolicyStates 使用了 **latest** (摘要訂用帳戶運算子之 **policyStatesSummaryResource** 的唯一允許值)，這會自動設定過去 24 小時的 [從] 和 [至] 時間範圍。

```http
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2018-04-04&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=IsCompliant eq false and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77' and PolicyDefinitionId eq '/providers/microsoft.authorization/policydefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62'
```

為求簡單明瞭，下列範例回應已修剪為只顯示一個不相容的資源 (請注意，@odata.count 實際上為 15，符合上述範例中不相容的資源計數)。 詳細的回應提供資源、原則 (或計畫) 和指派的數個相關資料。 請注意，您也可以查看哪些指派參數已傳遞至原則定義。

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
    "@odata.count": 15,
    "value": [{
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
        "timestamp": "2018-05-19T04:41:09Z",
        "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/rg-tags/providers/Microsoft.Compute/virtualMachines/linux",
        "policyAssignmentId": "/subscriptions/{subscriptionId}/resourceGroups/rg-tags/providers/Microsoft.Authorization/policyAssignments/37ce239ae4304622914f0c77",
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
        "effectiveParameters": "",
        "isCompliant": false,
        "subscriptionId": "{subscriptionId}",
        "resourceType": "/Microsoft.Compute/virtualMachines",
        "resourceLocation": "westus2",
        "resourceGroup": "RG-Tags",
        "resourceTags": "tbd",
        "policyAssignmentName": "37ce239ae4304622914f0c77",
        "policyAssignmentOwner": "tbd",
        "policyAssignmentParameters": "{\"tagName\":{\"value\":\"costCenter\"},\"tagValue\":{\"value\":\"Contoso-Test\"}}",
        "policyAssignmentScope": "/subscriptions/{subscriptionId}/resourceGroups/RG-Tags",
        "policyDefinitionName": "1e30110a-5ceb-460c-a204-c1c3969c6d62",
        "policyDefinitionAction": "deny",
        "policyDefinitionCategory": "tbd",
        "policySetDefinitionId": "",
        "policySetDefinitionName": "",
        "policySetDefinitionOwner": "",
        "policySetDefinitionCategory": "",
        "policySetDefinitionParameters": "",
        "managementGroupIds": "",
        "policyDefinitionReferenceId": ""
    }]
}
```

### <a name="view-events"></a>檢視事件

當建立或更新資源時，會產生原則評估結果。 結果稱為_原則事件_。 您可以使用下列 URI 來檢視與訂用帳戶建立關聯的最新原則事件。

```http
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyEvents/default/queryResults?api-version=2018-04-04
```

您的結果類似下列範例：

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyEvents/$metadata#default",
    "@odata.count": 1,
    "value": [{
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyEvents/$metadata#default/$entity",
        "NumAuditEvents": 16
    }]
}
```

如需有關查詢原則事件的詳細資訊，請參閱[原則事件](/rest/api/policy-insights/policyevents)參考文章。

### <a name="azure-powershell"></a>Azure PowerShell

在 PowerShell 資源庫中，會以 [AzureRM.PolicyInsights](https://www.powershellgallery.com/packages/AzureRM.PolicyInsights) 形式提供適用於原則的 Azure PowerShell 模組。 您可以使用 PowerShellGet，透過 `Install-Module -Name AzureRM.PolicyInsights` 來安裝模組 (請確定您已安裝最新的 [Azure PowerShell](/powershell/azure/install-azurerm-ps))：

```azurepowershell-interactive
# Install from PowerShell Gallery via PowerShellGet
Install-Module -Name AzureRM.PolicyInsights

# Import the downloaded module
Import-Module AzureRM.PolicyInsights

# Login with Connect-AzureRmAccount if not using Cloud Shell
Connect-AzureRmAccount
```

模組有三個 Cmdlet：

- `Get-AzureRmPolicyStateSummary`
- `Get-AzureRmPolicyState`
- `Get-AzureRmPolicyEvent`

範例：取得其不相容資源數目最高之最上層指派原則的狀態摘要。

```azurepowershell-interactive
PS> Get-AzureRmPolicyStateSummary -Top 1

NonCompliantResources : 15
NonCompliantPolicies  : 1
PolicyAssignments     : {/subscriptions/{subscriptionId}/resourcegroups/RG-Tags/providers/micros
                        oft.authorization/policyassignments/37ce239ae4304622914f0c77}
```

範例：取得最近評估資源的狀態記錄 (預設是依時間戳記遞減排序)。

```azurepowershell-interactive
PS> Get-AzureRmPolicyState -Top 1

Timestamp                  : 5/22/2018 3:47:34 PM
ResourceId                 : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Network/networkInterfaces/linux316
PolicyAssignmentId         : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Authorization/policyAssignments/37ce239ae4304622914f0c77
PolicyDefinitionId         : /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62
IsCompliant                : False
SubscriptionId             : {subscriptionId}
ResourceType               : /Microsoft.Network/networkInterfaces
ResourceLocation           : westus2
ResourceGroup              : RG-Tags
ResourceTags               : tbd
PolicyAssignmentName       : 37ce239ae4304622914f0c77
PolicyAssignmentOwner      : tbd
PolicyAssignmentParameters : {"tagName":{"value":"costCenter"},"tagValue":{"value":"Contoso-Test"}}
PolicyAssignmentScope      : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags
PolicyDefinitionName       : 1e30110a-5ceb-460c-a204-c1c3969c6d62
PolicyDefinitionAction     : deny
PolicyDefinitionCategory   : tbd
```

範例：取得所有不相容虛擬網路資源的詳細資料。

```azurepowershell-interactive
PS> Get-AzureRmPolicyState -Filter "ResourceType eq '/Microsoft.Network/virtualNetworks'"

Timestamp                  : 5/22/2018 4:02:20 PM
ResourceId                 : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Network/virtualNetworks/RG-Tags-vnet
PolicyAssignmentId         : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Authorization/policyAssignments/37ce239ae4304622914f0c77
PolicyDefinitionId         : /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62
IsCompliant                : False
SubscriptionId             : {subscriptionId}
ResourceType               : /Microsoft.Network/virtualNetworks
ResourceLocation           : westus2
ResourceGroup              : RG-Tags
ResourceTags               : tbd
PolicyAssignmentName       : 37ce239ae4304622914f0c77
PolicyAssignmentOwner      : tbd
PolicyAssignmentParameters : {"tagName":{"value":"costCenter"},"tagValue":{"value":"Contoso-Test"}}
PolicyAssignmentScope      : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags
PolicyDefinitionName       : 1e30110a-5ceb-460c-a204-c1c3969c6d62
PolicyDefinitionAction     : deny
PolicyDefinitionCategory   : tbd
```

範例：取得特定日期之後出現之不相容虛擬網路資源的相關事件。

```azurepowershell-interactive
PS> Get-AzureRmPolicyEvent -Filter "ResourceType eq '/Microsoft.Network/virtualNetworks'" -From '2018-05-19'

Timestamp                  : 5/19/2018 5:18:53 AM
ResourceId                 : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Network/virtualNetworks/RG-Tags-vnet
PolicyAssignmentId         : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Authorization/policyAssignments/37ce239ae4304622914f0c77
PolicyDefinitionId         : /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62
IsCompliant                : False
SubscriptionId             : {subscriptionId}
ResourceType               : /Microsoft.Network/virtualNetworks
ResourceLocation           : eastus
ResourceGroup              : RG-Tags
ResourceTags               : tbd
PolicyAssignmentName       : 37ce239ae4304622914f0c77
PolicyAssignmentOwner      : tbd
PolicyAssignmentParameters : {"tagName":{"value":"costCenter"},"tagValue":{"value":"Contoso-Test"}}
PolicyAssignmentScope      : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags
PolicyDefinitionName       : 1e30110a-5ceb-460c-a204-c1c3969c6d62
PolicyDefinitionAction     : deny
PolicyDefinitionCategory   : tbd
TenantId                   : {tenantId}
PrincipalOid               : {principalOid}
```

您可以透過 Azure PowerShell Cmdlet `Get-AzureRmADUser` 使用 [PrincipalOid] 欄位來取得特定使用者。 以您從上一個範例取得的回應取代 **{principalOid}**。

```azurepowershell-interactive
PS> (Get-AzureRmADUser -ObjectId {principalOid}).DisplayName
Trent Baker
```

## <a name="log-analytics"></a>Log Analytics

如果您有一個 [Log Analytics](../../../log-analytics/log-analytics-overview.md) 工作區，其中 `AzureActivity` 解決方案已繫結至您的訂用帳戶，則您也可以使用簡單的 Kusto 查詢和 `AzureActivity` 表格來檢視評估週期中不相容的結果。 由於 Log Analytics 中有不相容的詳細資料，這也表示可設定警示來監看特定資源、資源群組是否不相容，或甚至不相容項目的閾值，例如過去 24 小時內超過 10 個。

![使用 Log Analytics 的原則合規性](../media/getting-compliance-data/compliance-loganalytics.png)

## <a name="next-steps"></a>後續步驟

- 在 [Azure 原則範例](../samples/index.md)檢閱範例
- 檢閱[原則定義結構](../concepts/definition-structure.md)
- 檢閱[了解原則效果](../concepts/effects.md)
- 了解如何[以程式設計方式建立原則](programmatically-create.md)
- 探索如何[補救不符合規範的資源](remediate-resources.md)
- 檢閱[使用 Azure 管理群組來組織資源](../../management-groups/overview.md)，以了解何謂管理群組