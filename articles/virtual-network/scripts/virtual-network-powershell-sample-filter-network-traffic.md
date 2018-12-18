---
title: Azure PowerShell 指令碼範例 - 篩選 VM 網路流量 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 篩選輸入和輸出 VM 網路流量。
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: powershell
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/20/2018
ms.author: jdial
ms.openlocfilehash: 2241c86d069b3b30d869e7594dd1a7a03335b335
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31603992"
---
# <a name="filter-inbound-and-outbound-vm-network-traffic-script-sample"></a>篩選輸入和輸出 VM 網路流量的指令碼範例

此指令碼範例會建立一個具有前端和後端子網路的虛擬網路。 傳送到前端子網路的輸入流量會限制為 HTTP 及 HTTPS，而從後端子網路傳送到網際網路的輸出流量則不受允許。 執行此指令碼之後，您會有一部具有兩個 NIC 的虛擬機器。 每個 NIC 會連線到不同的子網路。

您可以從 Azure [Cloud Shell](https://shell.azure.com/powershell) 或從本機的 PowerShell 安裝來執行指令碼。 如果您在本機使用 PowerShell，此指令碼需要使用 AzureRM PowerShell 模組 5.4.1 版或更新版本。 若要尋找已安裝的版本，請執行 `Get-Module -ListAvailable AzureRM`。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurepowershell-interactive[main](../../../powershell_scripts/virtual-network/filter-network-traffic/filter-network-traffic.ps1  "Filter VM network traffic")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源：

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
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

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

可以在[虛擬網路 PowerShell 範例](../powershell-samples.md)中找到其他的虛擬網路 PowerShell 指令碼範例。
