---
title: Azure 原則範例 - 已核准的 VM 映像
description: 此範例原則會要求只能在環境中部署已核准的自訂映像。
services: azure-policy
author: DCtheGeek
manager: carmonm
ms.service: azure-policy
ms.topic: sample
ms.date: 09/18/2018
ms.author: dacoulte
ms.custom: mvc
ms.openlocfilehash: d4216c785155ac5462dbcb1b48bf58e7bc718601
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46965361"
---
# <a name="approved-vm-images"></a>已核准的 VM 映像

此原則會要求只能在環境中部署已核准的自訂映像。 您需指定已核准的映像識別碼陣列。

您可以利用下列各項部署此範例原則：

- [Azure 入口網站](#azure-portal)
- [Azure PowerShell](#azure-powershell)
- [Azure CLI](#azure-cli)
- [REST API](#rest-api)

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-policy"></a>範例原則

### <a name="policy-definition"></a>原則定義

撰寫好的完整 JSON 原則定義，可供 REST API、[部署至 Azure] 按鈕使用，也可以在入口網站中手動建立。

[!code-json[full](../../../../policy-templates/samples/compute/allowed-custom-images/azurepolicy.json "Complete policy definition (JSON)")]

> [!NOTE]
> 如果要在入口網站中手動建立原則，請使用上述的 **properties.parameters** 和 **properties.policyRule** 部分。 以大括號 `{}` 將兩個區段包住，使其成為有效的 JSON。

### <a name="policy-rules"></a>原則規則

定義原則規則的 JSON (由 Azure CLI 和 Azure PowerShell 使用)。

[!code-json[rule](../../../../policy-templates/samples/compute/allowed-custom-images/azurepolicy.rules.json "Policy rules (JSON)")]

### <a name="policy-parameters"></a>原則參數

定義原則參數的 JSON (由 Azure CLI 和 Azure PowerShell 使用)。

[!code-json[parameters](../../../../policy-templates/samples/compute/allowed-custom-images/azurepolicy.parameters.json "Policy parameters (JSON)")]

## <a name="parameters"></a>參數

|名稱 |類型 |欄位 |說明 |
|---|---|---|---|
|imageIds |陣列 |Microsoft.Compute/imageIds |已核准的 VM 映像清單|

透過 PowerShell 或 Azure CLI 建立指派時，可以使用 `-PolicyParameter` (PowerShell) 或 `--params` (Azure CLI) 在字串中或透過檔案將參數值當作 JSON 傳遞。
PowerShell 也支援 `-PolicyParameterObject` ，其要求將名稱/值雜湊表傳遞給 Cmdlet，其中 [名稱] 是參數名稱，而 [值] 是在指派過程中傳遞的單一值或值陣列。

此範例在參數中，只有允許 _YourResourceGroup_ 資源群組中的 _ContosoStdImage_ 或位於「美國中部」Windows Server 2016 Datacenter 的 2018 年 5 月份映像版本。

```json
{
    "imageIds": {
        "value": [
            "/subscriptions/<subscriptionId>/resourceGroups/YourResourceGroup/providers/Microsoft.Compute/images/ContosoStdImage",
            "/Subscriptions/<subscriptionId>/Providers/Microsoft.Compute/Locations/centralus/Publishers/MicrosoftWindowsServer/ArtifactTypes/VMImage/Offers/WindowsServer/Skus/2016-Datacenter/Versions/2016.127.20180510"
        ]
    }
}
```

## <a name="azure-portal"></a>Azure 入口網站

[![部署至 Azure](../media/deploy/deploybutton.png)](https://portal.azure.com/?#blade/Microsoft_Azure_Policy/CreatePolicyDefinitionBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-policy%2Fmaster%2Fsamples%2FCompute%2Fallowed-custom-images%2Fazurepolicy.json)
[![部署至 Azure Gov](../media/deploy/deployGovbutton.png)](https://portal.azure.us/?#blade/Microsoft_Azure_Policy/CreatePolicyDefinitionBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-policy%2Fmaster%2Fsamples%2FCompute%2Fallowed-custom-images%2Fazurepolicy.json)

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [sample-powershell-install](../../../../includes/sample-powershell-install-no-ssh.md)]

### <a name="deploy-with-azure-powershell"></a>使用 Azure PowerShell 部署

```azurepowershell-interactive
# Create the Policy Definition (Subscription scope)
$definition = New-AzureRmPolicyDefinition -Name 'allowed-custom-images' -DisplayName 'Approved VM images' -description 'This policy governs the approved VM images' -Policy 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/Compute/allowed-custom-images/azurepolicy.rules.json' -Parameter 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/Compute/allowed-custom-images/azurepolicy.parameters.json' -Mode All

# Set the scope to a resource group; may also be a subscription or management group
$scope = Get-AzureRmResourceGroup -Name 'YourResourceGroup'

# Set the Policy Parameter (JSON format)
$policyparam = '{ "imageIds": { "value": [ "/subscriptions/<subscriptionId>/resourceGroups/YourResourceGroup/providers/Microsoft.Compute/images/ContosoStdImage", "/Subscriptions/<subscriptionId>/Providers/Microsoft.Compute/Locations/centralus/Publishers/MicrosoftWindowsServer/ArtifactTypes/VMImage/Offers/WindowsServer/Skus/2016-Datacenter/Versions/2016.127.20180510" ] } }'

# Create the Policy Assignment
$assignment = New-AzureRmPolicyAssignment -Name 'allowed-custom-images-assignment' -DisplayName 'Approved VM images Assignment' -Scope $scope.ResourceId -PolicyDefinition $definition -PolicyParameter $policyparam
```

### <a name="remove-with-azure-powershell"></a>使用 Azure PowerShell 移除

執行下列命令以移除先前的指派和定義：

```azurepowershell-interactive
# Remove the Policy Assignment
Remove-AzureRmPolicyAssignment -Id $assignment.ResourceId

# Remove the Policy Definition
Remove-AzureRmPolicyDefinition -Id $definition.ResourceId
```

### <a name="azure-powershell-explanation"></a>Azure PowerShell 說明

部署和移除指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件：

| 命令 | 注意 |
|---|---|
| [New-AzureRmPolicyDefinition](/powershell/module/azurerm.resources/new-azurermpolicydefinition) | 建立新的 Azure 原則定義。 |
| [Get-AzureRmResourceGroup](/powershell/module/azurerm.resources/get-azurermresourcegroup) | 取得單一資源群組。 |
| [New-AzureRmPolicyAssignment](/powershell/module/azurerm.resources/new-azurermpolicyassignment) | 建立新的 Azure 原則指派。 在此範例中，我們會未它提供定義，但它也可以採取主動。 |
| [Remove-AzureRmPolicyAssignment](/powershell/module/azurerm.resources/remove-azurermpolicyassignment) | 移除現有的 Azure 原則指派。 |
| [Remove-AzureRmPolicyDefinition](/powershell/module/azurerm.resources/remove-azurermpolicydefinition) | 移除現有的 Azure 原則定義。 |

## <a name="azure-cli"></a>Azure CLI

[!INCLUDE [sample-cli-install](../../../../includes/sample-cli-install.md)]

### <a name="deploy-with-azure-cli"></a>使用 Azure CLI 進行部署

```azurecli-interactive
# Create the Policy Definition (Subscription scope)
definition=$(az policy definition create --name 'allowed-custom-images' --display-name 'Approved VM images' --description 'This policy governs the approved VM images' --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/Compute/allowed-custom-images/azurepolicy.rules.json' --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/Compute/allowed-custom-images/azurepolicy.parameters.json' --mode All)

# Set the scope to a resource group; may also be a subscription or management group
scope=$(az group show --name 'YourResourceGroup')

# Set the Policy Parameter (JSON format)
policyparam='{ "imageIds": { "value": [ "/subscriptions/<subscriptionId>/resourceGroups/YourResourceGroup/providers/Microsoft.Compute/images/ContosoStdImage", "/Subscriptions/<subscriptionId>/Providers/Microsoft.Compute/Locations/centralus/Publishers/MicrosoftWindowsServer/ArtifactTypes/VMImage/Offers/WindowsServer/Skus/2016-Datacenter/Versions/2016.127.20180510" ] } }'

# Create the Policy Assignment
assignment=$(az policy assignment create --name 'allowed-custom-images-assignment' --display-name 'Approved VM images Assignment' --scope `echo $scope | jq '.id' -r` --policy `echo $definition | jq '.name' -r` --params "$policyparam")
```

### <a name="remove-with-azure-cli"></a>使用 Azure CLI 移除

執行下列命令以移除先前的指派和定義：

```azurecli-interactive
# Remove the Policy Assignment
az policy assignment delete --name `echo $assignment | jq '.name' -r`

# Remove the Policy Definition
az policy definition delete --name `echo $definition | jq '.name' -r`
```

### <a name="azure-cli-explanation"></a>Azure CLI 說明

| 命令 | 注意 |
|---|---|
| [az policy definition create](/cli/azure/policy/definition?view=azure-cli-latest#az-policy-definition-create) | 建立新的 Azure 原則定義。 |
| [az group show](/cli/azure/group?view=azure-cli-latest#az-group-show) | 取得單一資源群組。 |
| [az policy assignment create](/cli/azure/policy/assignment?view=azure-cli-latest#az-policy-assignment-create) | 建立新的 Azure 原則指派。 在此範例中，我們會未它提供定義，但它也可以採取主動。 |
| [az policy assignment delete](/cli/azure/policy/assignment?view=azure-cli-latest#az-policy-assignment-delete) | 移除現有的 Azure 原則指派。 |
| [az policy definition delete](/cli/azure/policy/definition?view=azure-cli-latest#az-policy-definition-delete) | 移除現有的 Azure 原則定義。 |

## <a name="rest-api"></a>REST API

有數個工具可用來與 Resource Manager REST API 互動，例如 [ARMClient](https://github.com/projectkudu/ARMClient) 或 PowerShell。 在[原則定義結構](../concepts/definition-structure.md#aliases)的 [別名] 區段中，可以找到從 PowerShell 呼叫 REST API 的範例。

### <a name="deploy-with-rest-api"></a>使用 REST API 進行部署

- 建立原則定義 (訂用帳戶範圍)。 使用要求本文的[原則定義](#policy-definition) JSON。

  ```http
  PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyDefinitions/allowed-custom-images?api-version=2016-12-01
  ```

- 建立原則指派 (資源群組範圍)

  ```http
  PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/YourResourceGroup/providers/Microsoft.Authorization/policyAssignments/allowed-custom-images-assignment?api-version=2017-06-01-preview
  ```

  使用要求本文的下列 JSON 範例：

  ```json
  {
      "properties": {
          "displayName": "Approved VM images Assignment",
          "policyDefinitionId": "/subscriptions/<subscriptionId>/providers/Microsoft.Authorization/policyDefinitions/allowed-custom-images",
          "parameters": {
              "imageIds": {
                  "value": [
                      "/subscriptions/<subscriptionId>/resourceGroups/YourResourceGroup/providers/Microsoft.Compute/images/ContosoStdImage",
                      "/Subscriptions/<subscriptionId>/Providers/Microsoft.Compute/Locations/centralus/Publishers/MicrosoftWindowsServer/ArtifactTypes/VMImage/Offers/WindowsServer/Skus/2016-Datacenter/Versions/2016.127.20180510"
                  ]
              }
          }
      }
  }
  ```

### <a name="remove-with-rest-api"></a>使用 REST API 移除

- 移除原則指派

  ```http
  DELETE https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyAssignments/allowed-custom-images-assignment?api-version=2017-06-01-preview
  ```

- 移除原則定義

  ```http
  DELETE https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyDefinitions/allowed-custom-images?api-version=2016-12-01
  ```

### <a name="rest-api-explanation"></a>REST API 說明

| 服務 | 群組 | 作業 | 注意 |
|---|---|---|---|
| 資源管理 | 原則定義 | [建立](/rest/api/resources/policydefinitions/createorupdate) | 在訂用帳戶建立新的 Azure 原則定義。 替代方法：[建立管理群組](/rest/api/resources/policydefinitions/createorupdateatmanagementgroup) |
| 資源管理 | 原則指派 | [建立](/rest/api/resources/policyassignments/create) | 建立新的 Azure 原則指派。 在此範例中，我們會未它提供定義，但它也可以採取主動。 |
| 資源管理 | 原則指派 | [刪除](/rest/api/resources/policyassignments/delete) | 移除現有的 Azure 原則指派。 |
| 資源管理 | 原則定義 | [刪除](/rest/api/resources/policydefinitions/delete) | 移除現有的 Azure 原則定義。 替代方法：[刪除管理群組](/rest/api/resources/policydefinitions/deleteatmanagementgroup) |

## <a name="next-steps"></a>後續步驟

- 檢閱其他 [Azure 原則範例](index.md)
- 檢閱 [Azure 原則定義結構](../concepts/definition-structure.md)
- 如需虛擬機器的更多 Azure 原則範例，請參閱[將原則套用到 Windows VM](../../../virtual-machines/windows/policy.md)