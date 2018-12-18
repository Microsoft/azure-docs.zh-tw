---
title: Azure PowerShell 指令碼範例 - 建立 Azure 防火牆測試環境
description: Azure PowerShell 指令碼範例 - 建立 Azure 防火牆測試環境。
services: virtual-network
author: vhorne
ms.service: firewall
ms.devlang: powershell
ms.topic: sample
ms.date: 8/13/2018
ms.author: victorh
ms.openlocfilehash: 7f1986a9a59087d084577e980233ff87360a17e0
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49390103"
---
# <a name="create-an-azure-firewall-test-environment"></a>建立 Azure 防火牆測試環境

此指令碼範例會建立防火牆和測試網路環境。 網路有一個 VNet，包含三個子網路：AzureFirewallSubnet 和 ServersSubnet 以及 JumpboxSubnet。 ServersSubnet 和 JumpboxSubnet 中各有一個 2 核心 Windows Server。

防火牆位於 AzureFirewallSubnet 中，並使用應用程式規則集合進行設定，其中包含單一規則可允許存取 www.microsoft.com。

所建立的使用者定義路由會將 ServersSubnet 中的網路流量指向通過防火牆，該防火牆會套用防火牆規則。

您可以從 Azure [Cloud Shell](https://shell.azure.com/powershell) 或從本機的 PowerShell 安裝來執行指令碼。 

如果您在本機執行 PowerShell，此指令碼需要使用最新的 AzureRM PowerShell 模組版本 (6.9.0 或更新版本)。 若要尋找已安裝的版本，請執行 `Get-Module -ListAvailable AzureRM`。 

如果您需要升級，則可以使用 `PowerShellGet`，這會內建於 Windows 10 和 Windows Server 2016。

> [!NOTE]
>其他 Windows 版本會要求您安裝 `PowerShellGet` 後才能加以使用。 您可以執行 `Get-Module -Name PowerShellGet -ListAvailable | Select-Object -Property Name,Version,Path` 來判斷它是否安裝在您的系統上。 如果輸出是空白的，則您必須安裝最新的 [Windows Management Framework](https://www.microsoft.com/download/details.aspx?id=54616)。

如需詳細資訊，請參閱[使用 PowerShellGet 在 Windows 上安裝 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-6.4.0)

任何透過 Web Platform Installer 完成的現有 Azure PowerShell 安裝，都會與 PowerShellGet 安裝衝突，且必須加以移除。

請記住，如果您在本機執行 PowerShell，則也必須執行 `Connect-AzureRmAccount`，以建立與 Azure 的連線。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼


[!code-azurepowershell-interactive[main](../../../powershell_scripts/firewall/create-fw-test.ps1  "Create a firewall test environment")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源：

```powershell
Remove-AzureRmResourceGroup -Name AzfwSampleScriptEastUS -Force
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來建立資源群組、虛擬網路及網路安全性群組。 下表中的每個命令都會連結至命令特定的文件：

| 命令 | 注意 |
|---|---|
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立用來存放所有資源的資源群組。 |
| [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) | 建立子網路設定物件 |
| [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) | 建立 Azure 虛擬網路和前端子網路。 |
| [New-AzureRmNetworkSecurityRuleConfig](/powershell/module/azurerm.network/new-azurermnetworksecurityruleconfig) | 建立要指派給網路安全性群組的安全性規則。 |
| [New-AzureRmNetworkSecurityGroup](/powershell/module/azurerm.network/new-azurermnetworksecuritygroup) |建立對特定子網路允許或封鎖特定連接埠的 NSG 規則。 |
| [Set-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/set-azurermvirtualnetworksubnetconfig) | 將 NSG 與子網路建立關聯。 |
| [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) | 建立公用 IP 位址以從網際網路存取 VM。 |
| [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) | 建立虛擬網路介面，並將它們連結到虛擬網路的前端和後端子網路。 |
| [New-AzureRmVMConfig](/powershell/module/azurerm.compute/new-azurermvmconfig) | 建立 VM 組態。 此組態包括 VM 名稱、作業系統和系統管理認證等資訊。 建立 VM 時會使用此組態。 |
| [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) | 建立虛擬機器。 |
|[Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 移除資源群組及其內含的所有資源。 |
|[New-AzureRmFirewall](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermfirewall?view=azurermps-6.9.0)| 建立新的 Azure 防火牆。|
|[Get-AzureRmFirewall](https://docs.microsoft.com/powershell/module/azurerm.network/get-azurermfirewall?view=azurermps-6.9.0)|取得 Azure 防火牆物件。|
|[New-AzureRmFirewallApplicationRule](https://docs.microsoft.com/powershell/module/azurerm.network/new-azurermfirewallapplicationrule?view=azurermps-6.9.0)|建立新的 Azure 防火牆應用程式規則。|
|[Set-AzureRmFirewall](https://docs.microsoft.com/powershell/module/azurerm.network/set-azurermfirewall?view=azurermps-6.9.0)|認可 Azure 防火牆物件變更。|


## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

