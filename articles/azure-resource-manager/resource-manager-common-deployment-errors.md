---
title: 針對常見的 Azure 部署錯誤進行疑難排解 | Microsoft Docs
description: 說明如何解決使用 Azure Resource Manager 將資源部署至 Azure 時的常見錯誤。
services: azure-resource-manager
documentationcenter: ''
tags: top-support-issue
author: tfitzmac
manager: timlt
editor: tysonn
keywords: 部署錯誤, azure 部署, 部署至 azure
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/16/2018
ms.author: tomfitz
ms.openlocfilehash: 8da582750b5e20ddd7018f59292e7342f1628c8c
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39425378"
---
# <a name="troubleshoot-common-azure-deployment-errors-with-azure-resource-manager"></a>使用 Azure Resource Manager 針對常見的 Azure 部署錯誤進行疑難排解

本文說明可能會遇到的一些常見 Azure 部署錯誤，並且提供解決錯誤的資訊。 如果您找不到部署錯誤的錯誤碼，請參閱[尋找錯誤碼](#find-error-code)。

## <a name="error-codes"></a>錯誤碼

| 錯誤碼 | 緩和 | 詳細資訊 |
| ---------- | ---------- | ---------------- |
| AccountNameInvalid | 遵循儲存體帳戶的命名限制。 | [解析儲存體帳戶名稱](resource-manager-storage-account-name-errors.md) |
| AccountPropertyCannotBeSet | 檢查可用儲存體帳戶屬性。 | [storageAccounts](/azure/templates/microsoft.storage/storageaccounts) |
| AllocationFailed | 叢集或區域沒有可用的資源或無法支援所要求的 VM 大小。 稍後重試要求，或要求不同的 VM 大小。 | [Linux 的佈建和配置問題](../virtual-machines/linux/troubleshoot-deployment-new-vm.md)、[Windows 的佈建和配置問題](../virtual-machines/windows/troubleshoot-deployment-new-vm.md)以及[配置失敗疑難排解](../virtual-machines/windows/allocation-failure.md)|
| AnotherOperationInProgress | 等候並行作業完成。 | |
| AuthorizationFailed | 您的帳戶或服務主體沒有完成部署的足夠存取權。 請檢查您的帳戶所屬的角色以及它針對部署範圍的存取權。 | [Azure 角色型存取控制](../role-based-access-control/role-assignments-portal.md) |
| BadRequest | 您傳送的部署值不符合資源管理員的預期。 請查看內部狀態訊息，以取得疑難排解的說明。 | [範本參考](/azure/templates/)和[支援位置](resource-manager-templates-resources.md#location) |
| 衝突 | 您要求的作業在資源的目前狀態下不允許。 例如，只有在建立 VM 時或解除配置 VM 之後，才可調整磁碟大小。 | |
| DeploymentActive | 等候此資源群組的並行部署完成。 | |
| DeploymentFailed | DeploymentFailed 錯誤是一般錯誤，不會提供您解決錯誤所需的詳細資料。 尋找錯誤碼的錯誤詳細資料，以提供更多資訊。 | [尋找錯誤碼](#find-error-code) |
| DeploymentQuotaExceeded | 如果每個資源群組的部署達到 800 個數量限制，請從歷程記錄中刪除不再需要的部署。 您可以使用 [az group deployment delete](/cli/azure/group/deployment#az-group-deployment-delete) (Azure CLI) 或 [Remove-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/remove-azurermresourcegroupdeployment) (PowerShell) 以從歷程記錄中刪除項目。 從部署歷程記錄中刪除項目時，並不會影響部署資源。 | |
| DnsRecordInUse | DNS 記錄名稱必須是唯一的。 請提供不同的名稱，或是修改現有的記錄。 | |
| ImageNotFound | 檢查 VM 映像設定。 |  |
| InUseSubnetCannotBeDeleted | 當您嘗試更新資源時，可能會遇到這個錯誤，但是藉由刪除和建立資源即可處理要求。 請務必指定所有不變的值。 | [更新資源](/azure/architecture/building-blocks/extending-templates/update-resource) |
| InvalidAuthenticationTokenTenant | 取得適當租用戶的存取權杖。 您只能從您的帳戶所屬的租用戶取得權杖。 | |
| InvalidContentLink | 您最有可能嘗試連結至無法使用的巢狀範本。 再次確認您為巢狀範本提供的 URI。 如果儲存體帳戶中已有範本，請確定 URI 可存取。 您可能需要傳送 SAS 權杖。 | [連結的範本](resource-group-linked-templates.md) |
| InvalidParameter | 您為資源提供的其中一個值與預期值不相符。 此錯誤的原因可能是許多不同情況。 例如，密碼強度不足，或 blob 名稱不正確。 請檢查錯誤訊息，以判斷哪個值需要修正。 | |
| InvalidRequestContent | 您的部署值包含未預期的值，或遺漏必要值。 請確認您的資源類型值。 | [範本參考](/azure/templates/) |
| InvalidRequestFormat | 執行部署時啟用偵錯記錄，並且確認要求的內容。 | [偵錯記錄](#enable-debug-logging) |
| InvalidResourceNamespace | 請檢查您在 **type** 屬性中指定的資源命名空間。 | [範本參考](/azure/templates/) |
| InvalidResourceReference | 資源不存在或未正確地參考。 檢查是否需要新增相依性。 確認您使用 **reference** 函式包括案例的必要參數。 | [解析相依性](resource-manager-not-found-errors.md) |
| InvalidResourceType | 請檢查您在 **type** 屬性中指定的資源類型。 | [範本參考](/azure/templates/) |
| InvalidSubscriptionRegistrationState | 向資源提供者註冊訂用帳戶。 | [解析註冊](resource-manager-register-provider-errors.md) |
| InvalidTemplate | 請檢查錯誤的範本語法。 | [解析無效的範本](resource-manager-invalid-template-errors.md) |
| InvalidTemplateCircularDependency | 移除不必要的相依性。 | [解析循環相依性](resource-manager-invalid-template-errors.md#circular-dependency) |
| LinkedAuthorizationFailed | 檢查您的帳戶是否屬於與部署所在之資源群組相同的租用戶。 | |
| LinkedInvalidPropertyId | 資源的資源識別碼未正確地解析。 請檢查您為資源識別碼提供所有必要值，包含訂用帳戶識別碼、資源群組名稱、資源類型、父代資源名稱 (如有需要) 和資源名稱。 | |
| LocationRequired | 提供資源的位置。 | [設定位置](resource-manager-templates-resources.md#location) |
| MismatchingResourceSegments | 請確定巢狀資源的名稱和類型都有正確的區段數目。 | [解析資源區段](resource-manager-invalid-template-errors.md#incorrect-segment-lengths)
| MissingRegistrationForLocation | 檢查資源提供者註冊狀態和支援的位置。 | [解析註冊](resource-manager-register-provider-errors.md) |
| MissingSubscriptionRegistration | 向資源提供者註冊訂用帳戶。 | [解析註冊](resource-manager-register-provider-errors.md) |
| NoRegisteredProviderFound | 檢查資源提供者註冊狀態。 | [解析註冊](resource-manager-register-provider-errors.md) |
| NotFound | 您可以嘗試與父代資源並行部署相依資源。 檢查是否需要新增相依性。 | [解析相依性](resource-manager-not-found-errors.md) |
| OperationNotAllowed | 部署嘗試進行超過訂用帳戶、資源群組或區域配額的作業。 可能的話，請修改您的部署，以維持在配額內。 否則，請考慮要求變更您的配額。 | [解析配額](resource-manager-quota-errors.md) |
| ParentResourceNotFound | 請確定父代資源在建立子系資源之前即已存在。 | [解析父代資源](resource-manager-parent-resource-errors.md) |
| PrivateIPAddressInReservedRange | 指定的 IP 位址包含 Azure 所需的位址範圍。 變更 IP 位址以避免保留的範圍。 | [IP 位址](../virtual-network/virtual-network-ip-addresses-overview-arm.md) |
| PrivateIPAddressNotInSubnet | 指定的 IP 位址在子網路範圍之外。 變更 IP 位址，使其落在子網路範圍內。 | [IP 位址](../virtual-network/virtual-network-ip-addresses-overview-arm.md) |
| PropertyChangeNotAllowed | 有些屬性無法在已部署的資源上變更。 更新資源時，將您的變更限制為允許的屬性。 | [更新資源](/azure/architecture/building-blocks/extending-templates/update-resource) |
| RequestDisallowedByPolicy | 您的訂用帳戶包含會讓您無法在部署期間嘗試執行某個動作的資源原則。 尋找封鎖動作的原則。 可能的話，請修改您的部署以符合原則的限制。 | [解析原則](resource-manager-policy-requestdisallowedbypolicy-error.md) |
| ReservedResourceName | 提供不包含保留名稱的資源名稱。 | [唯一的資源名稱](resource-manager-reserved-resource-name.md) |
| ResourceGroupBeingDeleted | 等候刪除完成。 | |
| ResourceGroupNotFound | 檢查部署的目標資源群組名稱。 它必須已經存在於您的訂用帳戶中。 檢查訂用帳戶內容。 | [Azure CLI](/cli/azure/account?#az-account-set) [PowerShell](/powershell/module/azurerm.profile/set-azurermcontext) |
| ResourceNotFound | 您的部署會參考無法解析的資源。 確認您使用 **reference** 函式包括案例的必要參數。 | [解析參考](resource-manager-not-found-errors.md) |
| ResourceQuotaExceeded | 部署嘗試建立資源，這些資源超過訂用帳戶、資源群組或區域的配額。 可能的話，請修改您的基礎結構，以維持在配額內。 否則，請考慮要求變更您的配額。 | [解析配額](resource-manager-quota-errors.md) |
| SkuNotAvailable | 選取可供您選取之位置使用的 SKU (例如 VM 大小)。 | [解析 SKU](resource-manager-sku-not-available-errors.md) |
| StorageAccountAlreadyExists | 提供儲存體帳戶的唯一名稱。 | [解析儲存體帳戶名稱](resource-manager-storage-account-name-errors.md)  |
| StorageAccountAlreadyTaken | 提供儲存體帳戶的唯一名稱。 | [解析儲存體帳戶名稱](resource-manager-storage-account-name-errors.md) |
| StorageAccountNotFound | 檢查您嘗試使用的訂用帳戶、資源群組和儲存體帳戶名稱。 | |
| SubnetsNotInSameVnet | 虛擬機器只能有一個虛擬網路。 在部署多個 NIC 時，請確定它們屬於相同的虛擬網路。 | [多個 NIC](../virtual-machines/windows/multiple-nics.md) |
| TemplateResourceCircularDependency | 移除不必要的相依性。 | [解析循環相依性](resource-manager-invalid-template-errors.md#circular-dependency) |
| TooManyTargetResourceGroups | 減少單一部署的資源群組數目。 | [跨資源群組部署](resource-manager-cross-resource-group-deployment.md) |

## <a name="find-error-code"></a>尋找錯誤碼

您可能會收到兩種錯誤類型︰

* 驗證錯誤
* 部署錯誤

驗證錯誤來自可在部署之前判斷的情況。 其中包含您範本中的語法錯誤，或者嘗試部署會超出您訂用帳戶配額的資源。 部署程序期間出現的情況下會發生驗證錯誤。 其中包括嘗試存取以平行方式部署的資源。

這兩種錯誤類型都會傳回錯誤碼，以供您針對部署進行疑難排解。 這兩種錯誤類型都會出現在[活動記錄](resource-group-audit.md)中。 不過，驗證錯誤不會出現在部署歷程記錄中，因為部署永遠不會啟動。

### <a name="validation-errors"></a>驗證錯誤

透過入口網站進行部署時，您在提交值之後看到驗證錯誤。

![顯示入口網站驗證錯誤](./media/resource-manager-common-deployment-errors/validation-error.png)

選取訊息以取得詳細資訊。 在下圖中，您會看到 **InvalidTemplateDeployment** 錯誤，以及指出某個原則封鎖了部署的訊息。

![顯示驗證詳細資料](./media/resource-manager-common-deployment-errors/validation-details.png)

### <a name="deployment-errors"></a>部署錯誤

當作業通過驗證，但在部署期間失敗時，您會收到部署錯誤。

若要使用 PowerShell 查看部署錯誤的代碼和訊息，請使用：

```azurepowershell-interactive
(Get-AzureRmResourceGroupDeploymentOperation -DeploymentName exampledeployment -ResourceGroupName examplegroup).Properties.statusMessage
```

若要使用 Azure CLI 查看部署錯誤的代碼和訊息，請使用：

```azurecli-interactive
az group deployment operation list --name exampledeployment -g examplegroup --query "[*].properties.statusMessage"
```

在入口網站中選取通知。

![通知錯誤](./media/resource-manager-common-deployment-errors/notification.png)

您會看到更多與部署相關的詳細資料。 選取選項來尋找與錯誤相關的詳細資訊。

![部署失敗](./media/resource-manager-common-deployment-errors/deployment-failed.png)

您會看到錯誤訊息和錯誤碼。 請注意，有兩個錯誤碼。 第一個錯誤碼 (**DeploymentFailed**) 是一般錯誤，不會提供您解決錯誤所需的詳細資料。 第二個錯誤碼 (**StorageAccountNotFound**) 會提供您需要的詳細資料。 

![錯誤詳細資料](./media/resource-manager-common-deployment-errors/error-details.png)

## <a name="enable-debug-logging"></a>啟用偵錯記錄

有時您需要更多關於要求和回應的資訊，以了解哪裡出了錯。 在部署期間，您可以要求在部署期間記錄其他資訊。 

### <a name="powershell"></a>PowerShell

在 PowerShell 中，將 **DeploymentDebugLogLevel** 參數設定為 [All]、[ResponseContent] 或 [RequestContent]。

```powershell
New-AzureRmResourceGroupDeployment `
  -Name exampledeployment `
  -ResourceGroupName examplegroup `
  -TemplateFile c:\Azure\Templates\storage.json `
  -DeploymentDebugLogLevel All
```

檢查具有下列 Cmdlet 的要求內容︰

```powershell
(Get-AzureRmResourceGroupDeploymentOperation `
-DeploymentName exampledeployment `
-ResourceGroupName examplegroup).Properties.request `
| ConvertTo-Json
```

或者，具有下列項目的回應內容︰

```powershell
(Get-AzureRmResourceGroupDeploymentOperation `
-DeploymentName exampledeployment `
-ResourceGroupName examplegroup).Properties.response `
| ConvertTo-Json
```

此資訊可協助您判斷範本中的值是否會正確設定。

### <a name="azure-cli"></a>Azure CLI

目前，Azure CLI 不支援開啟偵錯記錄功能，但您可以擷取偵錯記錄。

使用下列命令檢查部署作業︰

```azurecli
az group deployment operation list \
  --resource-group examplegroup \
  --name exampledeployment
```

使用下列命令檢查要求內容︰

```azurecli
az group deployment operation list \
  --name exampledeployment \
  -g examplegroup \
  --query [].properties.request
```

使用下列命令檢查回應內容︰

```azurecli
az group deployment operation list \
  --name exampledeployment \
  -g examplegroup \
  --query [].properties.response
```

### <a name="nested-template"></a>巢狀範本

若要記錄巢狀範本的偵錯資訊，請使用 **debugSetting** 項目。

```json
{
    "apiVersion": "2016-09-01",
    "name": "nestedTemplate",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "{template-uri}",
            "contentVersion": "1.0.0.0"
        },
        "debugSetting": {
           "detailLevel": "requestContent, responseContent"
        }
    }
}
```

## <a name="create-a-troubleshooting-template"></a>建立疑難排解範本

在某些情況下，針對範本進行疑難排解的最簡單方式是測試其組件。 您可以建立簡化範本，以便專注於您認為是錯誤成因的組件。 例如，假設您在參考資源時收到錯誤。 不需要處理整個範本，而是建立一個會傳回可能造成問題之組件的範本。 它可協助您判斷是否傳入正確的參數、正確地使用範本函式，以及取得所預期的資源。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageName": {
        "type": "string"
    },
    "storageResourceGroup": {
        "type": "string"
    }
  },
  "variables": {},
  "resources": [],
  "outputs": {
    "exampleOutput": {
        "value": "[reference(resourceId(parameters('storageResourceGroup'), 'Microsoft.Storage/storageAccounts', parameters('storageName')), '2016-05-01')]",
        "type" : "object"
    }
  }
}
```

或者，假設您遇到您認為與未正確設定之相依性有關的部署錯誤。 將其細分為簡化範本以測試您的範本。 首先，建立可部署單一資源 (例如 SQL Server) 的範本。 當您確定已正確定義該資源時，加入依存該資源的資源 (例如 SQL Database)。 當您正確定義這兩個資源時，加入其他相依的資源 (例如稽核原則)。 在每個測試部署之間，刪除資源群組以確保您充分測試相依性。


## <a name="next-steps"></a>後續步驟
* 若要了解稽核動作，請參閱 [使用 Resource Manager 來稽核作業](resource-group-audit.md)。
* 若要了解部署期間可採取哪些動作來判斷錯誤，請參閱 [檢視部署作業](resource-manager-deployment-operations.md)。
