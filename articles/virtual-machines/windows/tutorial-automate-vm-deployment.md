---
title: 教學課程 - 在 Azure 中的 Windows VM 上安裝應用程式 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用自訂指令碼擴充功能執行指令碼，以及將應用程式部署至 Azure 中的 Windows 虛擬機器。
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/09/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 2a365321e35c84d5b09570a7eac41bec6cdeb143
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37928959"
---
# <a name="tutorial---deploy-applications-to-a-windows-virtual-machine-in-azure-with-the-custom-script-extension"></a>教學課程 - 使用自訂指令碼擴充功能將應用程式部署至 Azure 中的 Windows 虛擬機器

若要以快速且一致的方式設定虛擬機器 (VM)，通常需要某種形式的自動化。 自訂 Windows VM 的常見方法是使用 [Windows 的自訂指令碼擴充功能](extensions-customscript.md)。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 使用自訂指令碼擴充功能安裝 IIS
> * 使用自訂指令碼擴充功能建立 VM
> * 套用擴充功能之後檢視執行中的 IIS 網站

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，必須使用 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。


## <a name="custom-script-extension-overview"></a>自訂指令碼擴充功能概觀
自訂指令碼擴充功能會在 Azure VM 上下載並執行指令碼。 此擴充功能適用於部署後組態、軟體安裝或其他任何組態/管理工作。 您可以從 Azure 儲存體或 GitHub 下載指令碼，或是在擴充功能執行階段將指令碼提供給 Azure 入口網站。

「自訂指令碼擴充功能」會與 Azure Resource Manager 範本整合，您也可以使用 Azure CLI、PowerShell、Azure 入口網站或「Azure 虛擬機器 REST API」來執行它。

您可以搭配 Windows 和 Linux VM 使用自訂指令碼擴充功能。


## <a name="create-virtual-machine"></a>建立虛擬機器
首先，使用 [Get-credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 設定 VM 的系統管理員使用者名稱和密碼：

```azurepowershell-interactive
$cred = Get-Credential
```

現在您可以使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 建立 VM。 下列範例會在 *EastUS* 位置中建立名為 *myVM* 的 VM。 如果它們尚不存在，建立支援網路資源的資源群組 *myResourceGroupAutomate*。 為了允許 Web 流量，此 Cmdlet 也會開啟連接埠 *80*。

```azurepowershell-interactive
New-AzureRmVm `
    -ResourceGroupName "myResourceGroupAutomate" `
    -Name "myVM" `
    -Location "East US" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -OpenPorts 80 `
    -Credential $cred
```

系統需要花幾分鐘的時間來建立資源和 VM。


## <a name="automate-iis-install"></a>自動安裝 IIS
使用 [Set-AzureRmVMExtension](/powershell/module/azurerm.compute/set-azurermvmextension) 來安裝自訂指令碼擴充功能。 擴充功能會執行 `powershell Add-WindowsFeature Web-Server` 以安裝 IIS Web 伺服器，然後更新 Default.htm 頁面以顯示 VM 的主機名稱：

```azurepowershell-interactive
Set-AzureRmVMExtension -ResourceGroupName "myResourceGroupAutomate" `
    -ExtensionName "IIS" `
    -VMName "myVM" `
    -Location "EastUS" `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.8 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}'
```


## <a name="test-web-site"></a>測試網站
使用 [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) 取得負載平衡器的公用 IP 位址。 下列範例會取得稍早建立的 *myPublicIPAddress* IP 位址︰

```azurepowershell-interactive
Get-AzureRmPublicIPAddress `
    -ResourceGroupName "myResourceGroupAutomate" `
    -Name "myPublicIPAddress" | select IpAddress
```

接著，您可以在 Web 瀏覽器中輸入公用 IP 位址。 網站隨即顯示，包括負載平衡器分散流量之 VM 的主機名稱，如下列範例所示：

![執行中的 IIS 網站](./media/tutorial-automate-vm-deployment/running-iis-website.png)


## <a name="next-steps"></a>後續步驟

在本教學課程中，您將 VM 上的 IIS 安裝自動化。 您已了解如何︰

> [!div class="checklist"]
> * 使用自訂指令碼擴充功能安裝 IIS
> * 使用自訂指令碼擴充功能建立 VM
> * 套用擴充功能之後檢視執行中的 IIS 網站

前進至下一個教學課程，以了解如何建立自訂的 VM 映像。

> [!div class="nextstepaction"]
> [建立自訂的 VM 映像](./tutorial-custom-images.md)
