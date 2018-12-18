---
title: 教學課程 - 使用 Azure PowerShell 在擴展集中安裝應用程式 | Microsoft Docs
description: 了解如何搭配使用 Azure PowerShell 與自訂指令碼擴充功能，將應用程式安裝到虛擬機器擴展集
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: b74e001a0430adc45d06776f2c2f5383acae5856
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46960017"
---
# <a name="tutorial-install-applications-in-virtual-machine-scale-sets-with-azure-powershell"></a>教學課程 - 使用 Azure PowerShell 在虛擬機器擴展集中安裝應用程式
若要在擴展集的虛擬機器 (VM) 執行個體上執行應用程式，您需要先安裝應用程式元件和必要的檔案。 在先前的教學課程中，您已了解如何建立及使用自訂 VM 映像來部署您的 VM 執行個體。 此自訂映像已包含手動應用程式安裝和組態。 您也可以在部署好每個 VM 執行個體後，讓應用程式自動安裝到擴展集，或更新已在擴展集上執行的應用程式。 在此教學課程中，您將了解如何：

> [!div class="checklist"]
> * 自動將應用程式安裝到擴展集
> * 使用 Azure 自訂指令碼擴充功能
> * 更新在擴展集上執行的應用程式

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行此教學課程時，必須使用 Azure PowerShell 模組 6.0.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。 


## <a name="what-is-the-azure-custom-script-extension"></a>Azure 自訂指令碼擴充功能是什麼？
自訂指令碼擴充功能會在 Azure VM 上下載並執行指令碼。 此擴充功能適用於部署後組態、軟體安裝或其他任何組態/管理工作。 您可以從 Azure 儲存體或 GitHub 下載指令碼，或是在擴充功能執行階段將指令碼提供給 Azure 入口網站。

自訂指令碼延伸模組會與 Azure Resource Manager 範本整合，而且可與 Azure CLI、Azure PowerShell、Azure 入口網站或 REST API 搭配使用。 如需詳細資訊，請參閱[自訂指令碼延伸模組概觀](../virtual-machines/windows/extensions-customscript.md)。

若要查看作用中的自訂指令碼擴充功能，可建立擴展集來安裝 IIS Web 伺服器，並輸出 VM 執行個體的主機名稱。 自訂指令碼擴充功能定義會從 GitHub 下載範例指令碼、安裝必要的套件，然後將 VM 執行個體主機名稱寫入基本的 HTML 頁面。


## <a name="create-a-scale-set"></a>建立擴展集
現在使用 [New-AzureRmVmss](/powershell/module/azurerm.compute/new-azurermvmss) 建立虛擬機器擴展集。 為了將流量散發到個別的虛擬機器執行個體，也會建立負載平衡器。 負載平衡器包含在 TCP 連接埠 80 上分配流量的規則，同時允許 TCP 連接埠 3389 上的遠端桌面流量以及 TCP 連接埠 5985 上的 PowerShell 遠端處理。 出現提示時，請為擴展集中的 VM 執行個體提供適當的系統管理認證：

```azurepowershell-interactive
New-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet" `
  -Location "EastUS" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic"
```

建立及設定所有擴展集資源和 VM 需要幾分鐘的時間。


## <a name="create-custom-script-extension-definition"></a>建立自訂指令碼擴充功能的定義
Azure PowerShell 會使用雜湊表來儲存要下載的檔案和要執行的命令。 下列範例會使用 GitHub 中的範例指令碼。 首先，建立此組態物件，如下所示：

```azurepowershell-interactive
$customConfig = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis.ps1"
}
```

現在，使用 [Add-AzureRmVmssExtension](/powershell/module/AzureRM.Compute/Add-AzureRmVmssExtension) 來套用自訂指令碼擴充功能。 先前定義的組態物件會傳遞至此擴充功能。 使用 [Update-AzureRmVmss](/powershell/module/azurerm.compute/update-azurermvmss) 來更新和執行 VM 執行個體上的擴充功能。

```azurepowershell-interactive
# Get information about the scale set
$vmss = Get-AzureRmVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Add the Custom Script Extension to install IIS and configure basic website
$vmss = Add-AzureRmVmssExtension `
  -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.9 `
  -Setting $customConfig

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

擴展集中的每個 VM 執行個體會下載並執行 GitHub 中的指令碼。 在更複雜的範例中，您可以安裝多個應用程式元件和檔案。 如果相應增加擴展集，新的 VM 執行個體就會自動套用相同的自訂指令碼擴充功能定義，並安裝必要的應用程式。


## <a name="test-your-scale-set"></a>測試您的擴展集
若要查看作用中的 web 伺服器，可使用 [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 取得負載平衡器的公用 IP 位址。 下列範例會取得 myResourceGroup 資源群組中建立的 IP 位址：

```azurepowershell-interactive
Get-AzureRmPublicIpAddress -ResourceGroupName "myResourceGroup" | Select IpAddress
```

將負載平衡器的公用 IP 位址輸入網頁瀏覽器中。 負載平衡器會將流量散發至您的其中一個 VM 執行個體，如下列範例所示：

![IIS 中的基本網頁](media/tutorial-install-apps-powershell/running-iis.png)

讓網頁瀏覽器保持開啟，您就可以看到下一個步驟中的已更新版本。


## <a name="update-app-deployment"></a>更新應用程式部署
在擴展集的整個生命週期中，您可能需要部署更新版的應用程式。 透過自訂指令碼擴充功能，您可以參考已更新的部署指令碼，然後將擴充功能重新套用至擴展集。 在上一個步驟中建立擴展集時，`-UpgradePolicyMode` 已設定為「自動」。 此設定可讓擴展集中的 VM 執行個體自動更新並套用最新版的應用程式。

建立名為 customConfigv2 的新組態定義。 此定義會執行已更新的 v2 版應用程式安裝指令碼：

```azurepowershell-interactive
$customConfigv2 = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis-v2.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis-v2.ps1"
}
```

使用 [Add-AzureRmVmssExtension](/powershell/module/AzureRM.Compute/Add-AzureRmVmssExtension)，再次將自訂指令碼擴充功能組態套用到擴展集中的 VM 執行個體。 CustomConfigv2 定義會用來套用更新版的應用程式：

```azurepowershell-interactive
# Reapply the Custom Script Extension to install the updated website
$vmss = Add-AzureRmVmssExtension `
  -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.9 `
  -Setting $customConfigv2

# Update the scale set and reapply the Custom Script Extension to the VM instances
Update-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

擴展集中的所有 VM 執行個體都會自動以最新版的範例網頁來進行更新。 若要查看更新後的版本，請重新整理您瀏覽器中的網站：

![IIS 中的更新網頁](media/tutorial-install-apps-powershell/running-iis-updated.png)


## <a name="clean-up-resources"></a>清除資源
若要移除您的擴展集與其他資源，請使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 刪除資源群組及其所有資源。 `-Force` 參數會確認您想要刪除資源，而不另外對您提示將要進行此作業。 `-AsJob` 參數不會等待作業完成，就會將控制項傳回給提示字元。

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>後續步驟
在此教學課程中，您已了解如何使用 Azure PowerShell 自動安裝和更新擴展集上的應用程式：

> [!div class="checklist"]
> * 自動將應用程式安裝到擴展集
> * 使用 Azure 自訂指令碼擴充功能
> * 更新在擴展集上執行的應用程式

前往下一個教學課程，以了解如何自動調整擴展集。

> [!div class="nextstepaction"]
> [自動調整擴展集](tutorial-autoscale-powershell.md)
