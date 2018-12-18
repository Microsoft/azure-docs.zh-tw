---
title: 使用 Azure 原則來限制 VM 延伸模組安裝 | Microsoft Docs
description: 使用 Azure 原則來限制延伸模組部署。
services: virtual-machines-linux
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/23/2018
ms.author: roiyz;cynthn
ms.openlocfilehash: 15a6a7f4753d51118d23d2e3c021010218d2d2d7
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451828"
---
# <a name="use-azure-policy-to-restrict-extensions-installation-on-windows-vms"></a>使用 Azure 原則來限制 Windows VM 上的延伸模組安裝

如果您想要防止在 Windows VM 上使用或安裝特定的延伸模組，就可以使用 PowerShell 來建立 Azure 原則，以限制某個資源群組內 VM 的延伸模組。 

本教學課程會使用 Cloud Shell 內的 Azure PowerShell，這會不斷更新至最新版本。 如果您選擇在本機安裝和使用 PowerShell，本教學課程會要求使用 Azure PowerShell 模組版本 3.6 或更新版本。 執行 ` Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 

## <a name="create-a-rules-file"></a>建立規則檔

為了限制可安裝的延伸模組，您必須要有一個[規則](/azure/azure-policy/policy-definition#policy-rule)來提供識別延伸模組的邏輯。

以下範例說明如何在 Azure Cloud Shell 中建立規則檔來拒絕安裝 'Microsoft.Compute' 所發佈的延伸模組，但如果您是在本機 PowerShell 中運作，則也可以建立本機檔案，然後將路徑 ($home/clouddrive) 取代成您電腦上該本機檔案的路徑。

在 [Cloud Shell](https://shell.azure.com/powershell) 中，輸入：

```azurepowershell-interactive
nano $home/clouddrive/rules.json
```

將下列 .json 複製並貼到檔案中。

```json
{
    "if": {
        "allOf": [
            {
                "field": "type",
                "equals": "Microsoft.Compute/virtualMachines/extensions"
            },
            {
                "field": "Microsoft.Compute/virtualMachines/extensions/publisher",
                "equals": "Microsoft.Compute"
            },
            {
                "field": "Microsoft.Compute/virtualMachines/extensions/type",
                "in": "[parameters('notAllowedExtensions')]"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

完成時，按 **Ctrl + O**，然後按 **Enter** 以儲存檔案。 按 **Ctrl + X** 以關閉檔案並結束。

## <a name="create-a-parameters-file"></a>建立參數檔案

您也需要一個[參數](/azure/azure-policy/policy-definition#parameters)檔，以建立供您用於傳遞要封鎖之延伸模組清單的結構。 

以下範例說明如何在 Cloud Shell 中為 VM 建立參數檔，但如果您是在本機 PowerShell 中運作，則也可以建立本機檔案，然後將路徑 ($home/clouddrive) 取代成您電腦上該本機檔案的路徑。

在 [Cloud Shell](https://shell.azure.com/powershell) 中，輸入：

```azurepowershell-interactive
nano $home/clouddrive/parameters.json
```

將下列 .json 複製並貼到檔案中。

```json
{
    "notAllowedExtensions": {
        "type": "Array",
        "metadata": {
            "description": "The list of extensions that will be denied.",
            "strongType": "type",
            "displayName": "Denied extension"
        }
    }
}
```

完成時，按 **Ctrl + O**，然後按 **Enter** 以儲存檔案。 按 **Ctrl + X** 以關閉檔案並結束。

## <a name="create-the-policy"></a>建立原則

原則定義是一個用來儲存您所要使用之設定的物件。 原則定義會使用規則檔和參數檔來定義原則。 請使用 [New-AzureRmPolicyDefinition](/powershell/module/azurerm.resources/new-azurermpolicydefinition) Cmdlet 來建立原則定義。

 原則規則和參數是您在 Cloud Shell 中以 .json 檔案形式建立並儲存的檔案。


```azurepowershell-interactive
$definition = New-AzureRmPolicyDefinition `
   -Name "not-allowed-vmextension-windows" `
   -DisplayName "Not allowed VM Extensions" `
   -description "This policy governs which VM extensions that are explicitly denied."   `
   -Policy 'C:\Users\ContainerAdministrator\clouddrive\rules.json' `
   -Parameter 'C:\Users\ContainerAdministrator\clouddrive\parameters.json'
```




## <a name="assign-the-policy"></a>指派原則

以下範例會使用 [New-AzureRMPolicyAssignment](/powershell/module/azurerm.resources/new-azurermpolicyassignment) 將原則指派給資源群組。 任何建立在 **myResourceGroup** 資源群組中的 VM 都將無法安裝「VM 存取代理程式」或「自訂指令碼」延伸模組。 

請使用 [Get-AzureRMSubscription | Format-Table](/powershell/module/azurerm.profile/get-azurermsubscription) Cmdlet 來取得您的訂用帳戶 ID，以用來取代範例中的 ID。

```azurepowershell-interactive
$scope = "/subscriptions/<subscription id>/resourceGroups/myResourceGroup"
$assignment = New-AzureRMPolicyAssignment `
   -Name "not-allowed-vmextension-windows" `
   -Scope $scope `
   -PolicyDefinition $definition `
   -PolicyParameter '{
    "notAllowedExtensions": {
        "value": [
            "VMAccessAgent",
            "CustomScriptExtension"
        ]
    }
}'
$assignment
```

## <a name="test-the-policy"></a>測試原則

若要測試原則，請嘗試使用「VM 存取」延伸模組。 以下範例應該會失敗並出現「Set-AzureRmVMAccessExtension : 原則不允許資源 'myVMAccess'」訊息。

```azurepowershell-interactive
Set-AzureRmVMAccessExtension `
   -ResourceGroupName "myResourceGroup" `
   -VMName "myVM" `
   -Name "myVMAccess" `
   -Location EastUS 
```

在入口網站中，密碼變更應該會失敗並出現「因為違反了原則，所以範本部署失敗。」 訊息。

## <a name="remove-the-assignment"></a>移除指派

```azurepowershell-interactive
Remove-AzureRMPolicyAssignment -Name not-allowed-vmextension-windows -Scope $scope
```

## <a name="remove-the-policy"></a>移除原則

```azurepowershell-interactive
Remove-AzureRmPolicyDefinition -Name not-allowed-vmextension-windows
```
    
## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱 [Azure 原則](../../azure-policy/azure-policy-introduction.md)。
