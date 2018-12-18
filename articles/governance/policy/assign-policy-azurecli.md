---
title: 使用 Azure CLI 建立原則指派，以識別 Azure 環境中的不相容資源
description: 使用 PowerShell 建立 Azure 原則指派，以識別不相容資源。
services: azure-policy
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: quickstart
ms.service: azure-policy
ms.custom: mvc
ms.openlocfilehash: 4954ca42af1755ea62e7142048d48805397b6a0a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46968476"
---
# <a name="create-a-policy-assignment-to-identify-non-compliant-resources-in-your-azure-environment-with-the-azure-cli"></a>使用 Azure CLI 建立原則指派，以識別 Azure 環境中的不相容資源

了解 Azure 中合規性的第一個步驟是識別您資源的狀態。
本快速入門會逐步引導您完成程序來建立原則指派，以識別出未使用受控磁碟的虛擬機器。

在此程序結束時，您將會成功識別出未使用受控磁碟的虛擬機器。 它們「不符合」原則指派的規範。

Azure CLI 可用來從命令列或在指令碼中建立和管理 Azure 資源。 本指南使用 Azure CLI 來建立原則指派，以及識別 Azure 環境中不符合規範的資源。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

本快速入門會要求您執行 Azure CLI 2.0.4 版或更新版本，以在本機安裝和使用 CLI。 若要尋找版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。

## <a name="prerequisites"></a>必要條件

使用 Azure CLI 註冊 Policy Insights 資源提供者。 註冊資源提供者，以確保您的訂用帳戶可搭配它使用。 若要註冊資源提供者，您必須有權執行資源提供者的註冊動作作業。 這項作業包含在「參與者」和「擁有者」角色中。 執行下列命令以註冊資源提供者：

```azurecli-interactive
az provider register --namespace 'Microsoft.PolicyInsights'
```

如需註冊及檢視資源提供者的詳細資訊，請參閱[資源提供者和類型](../../azure-resource-manager/resource-manager-supported-services.md)

請安裝 [ARMClient](https://github.com/projectkudu/ARMClient) (如果尚未安裝)。 此工具會將 HTTP 要求傳送至以 Azure Resource Manager 為基礎的 API。

## <a name="create-a-policy-assignment"></a>建立原則指派

在本快速入門中，您會建立一個原則指派，並且指派 **Audit VMs that do not use managed disks (稽核沒有受控磁碟的虛擬機器)** 定義。 此原則定義會識別與原則定義中設定之條件不相符的資源。

執行下列命令以建立原則指派：

```azurecli-interactive
az policy assignment create --name 'audit-vm-manageddisks' --display-name 'Audit Virtual Machines without Managed Disks Assignment' --scope '<scope>' --policy '<policy definition ID>'
```

上述命令會使用下列資訊：

- **Name** - 指派的實際名稱。  例如，我們使用了 audit-vm-manageddisks。
- **DisplayName** - 原則指派的顯示名稱。 在此案例中，您會使用 Audit Virtual Machines without Managed Disks (稽核沒有受控磁碟指派的虛擬機器)。
- **原則** – 原則定義識別碼，這是您用來建立指派的根基。 在此案例中，即為原則定義「Audit VMs that do not use managed disks (稽核沒有受控磁碟的虛擬機器)」的 ID。 若要取得原則定義識別碼，請執行此命令：`az policy definition list --query "[?displayName=='Audit VMs that do not use managed disks']"`
- **範圍** – 範圍會決定在哪些資源或資源群組上強制執行原則指派。 範圍從訂用帳戶到資源群組。 請務必將 &lt;scope&gt; 取代為您的資源群組。

## <a name="identify-non-compliant-resources"></a>識別不相容的資源

若要檢視這個新指派之下不符合規範的資源，請執行下列命令來取得原則指派識別碼：

```azurepowershell-interactive
$policyAssignment = Get-AzureRmPolicyAssignment | Where-Object { $_.Properties.DisplayName -eq 'Audit Virtual Machines without Managed Disks Assignment' }
$policyAssignment.PolicyAssignmentId
```

如需有關原則指派識別碼的詳細資訊，請參閱 [Get-AzureRMPolicyAssignment](/powershell/module/azurerm.resources/get-azurermpolicyassignment)。

接下來，執行下列命令，以取得不合規資源的資源識別碼，而這些識別碼會輸出到 JSON 檔案中：

```
armclient post "/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2017-12-12-preview&$filter=IsCompliant eq false and PolicyAssignmentId eq '<policyAssignmentID>'&$apply=groupby((ResourceId))" > <json file to direct the output with the resource IDs into>
```

您的結果類似下列範例：

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
    "@odata.count": 3,
    "value": [{
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachineId>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine2Id>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionName>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine3ID>"
        }

    ]
}
```

結果類似於您通常在 Azure 入口網站檢視中看到列在 [不符合規範的資源] 之下的內容。

## <a name="clean-up-resources"></a>清除資源

此集合中的其他指南是以本快速入門為基礎。 如果您打算繼續進行後續的教學課程，請勿清除在此快速入門中建立的資源。 如果您不打算繼續，請藉由執行以下命令來刪除您所建立的指派：

```azurecli-interactive
az policy assignment delete --name 'audit-vm-manageddisks' --scope '/subscriptions/<subscriptionID>/<resourceGroupName>'
```

## <a name="next-steps"></a>後續步驟

在這個快速入門中，您指派原則定義以識別 Azure 環境中的不相容資源。

若要深入了解如何指派原則並確保您在**未來**建立的資源符合規範，請繼續進行教學課程：

> [!div class="nextstepaction"]
> [建立及管理原則](./tutorials/create-and-manage.md)