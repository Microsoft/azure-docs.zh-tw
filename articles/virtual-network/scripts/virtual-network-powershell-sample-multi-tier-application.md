---
title: Azure PowerShell 指令碼範例 - 為多層式應用程式建立網路 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 為多層式應用程式建立虛擬網路。
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
ms.openlocfilehash: 3a57827616e4722b8520dcddb64e4e67fa8c79c9
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31599884"
---
# <a name="create-a-network-for-multi-tier-applications-script-sample"></a>為多層式應用程式建立網路的指令碼範例

此指令碼範例會建立一個具有前端和後端子網路的虛擬網路。 傳送到前端子網路的流量會限制為 HTTP 和 SSH，而傳送到後端子網路的流量則限制為 MySQL 且連接埠為 3306。 執行此指令碼之後，您將有兩部虛擬機器，每個子網路中各有一部，可供您在其中部署 Web 伺服器和 MySQL 軟體。

您可以從 Azure [Cloud Shell](https://shell.azure.com/powershell) 或從本機的 PowerShell 安裝來執行指令碼。 如果您在本機使用 PowerShell，此指令碼需要使用 AzureRM PowerShell 模組 5.4.1 版或更新版本。 若要尋找已安裝的版本，請執行 `Get-Module -ListAvailable AzureRM`。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼


[!code-azurepowershell-interactive[main](../../../powershell_scripts/virtual-network/virtual-network-multi-tier-application/virtual-network-multi-tier-application.ps1  "Virtual network for multi-tier application")]

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
| [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) | 建立 Azure 虛擬網路和前端子網路。 |
| [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) | 建立後端子網路。 |
| [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) | 建立公用 IP 位址以從網際網路存取 VM。 |
| [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) | 建立虛擬網路介面，並將它們連結到虛擬網路的前端和後端子網路。 |
| [New-AzureRmNetworkSecurityGroup](/powershell/module/azurerm.network/new-azurermnetworksecuritygroup) | 建立與前端和後端子網路關聯的網路安全性群組 (NSG)。 |
| [New-AzureRmNetworkSecurityRuleConfig](/powershell/module/azurerm.network/new-azurermnetworksecurityruleconfig) |建立對特定子網路允許或封鎖特定連接埠的 NSG 規則。 |
| [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) | 建立虛擬機器，並將 NIC 連結到每個 VM。 此命令也會指定要使用的虛擬機器映像和系統管理認證。 |
| [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 刪除資源群組及其包含的所有資源。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

可以在[虛擬網路 PowerShell 範例](../powershell-samples.md)中找到其他的虛擬網路 PowerShell 指令碼範例。
