---
title: Azure PowerShell 指令碼範例 - 為多層式應用程式建立網路 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 為多層式應用程式建立虛擬網路。
services: virtual-network
documentationcenter: virtual-network
author: georgewallace
manager: timlt
editor: tysonn
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: powershell
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 05/16/2017
ms.author: gwallace
ms.openlocfilehash: cff445f7657d5661f8577d9f6be7072eed2c1c28
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31597277"
---
# <a name="create-a-network-for-multi-tier-applications"></a>為多層式應用程式建立網路

此指令碼範例會建立一個具有前端和後端子網路的虛擬網路。 傳送到前端子網路的流量會限制為 HTTP 和 SSH，而傳送到後端子網路的流量則限制為 MySQL 且連接埠為 3306。 執行此指令碼之後，您將有兩部虛擬機器，每個子網路中各有一部，可供您在其中部署 Web 伺服器和 MySQL 軟體。

您可以視需要使用 [Azure PowerShell 指南](https://docs.microsoft.com/powershell/azureps-cmdlets-docs/) \(英文\) 中的指示來安裝 Azure PowerShell，然後執行 `Connect-AzureRmAccount` 來建立與 Azure 的連線。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼


[!code-powershell[main](../../../powershell_scripts/virtual-network/virtual-network-multi-tier-application/virtual-network-multi-tier-application.ps1  "Virtual network for multi-tier application")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源。

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來建立資源群組、虛擬網路及網路安全性群組。 下表中的每個命令都會連結至命令特定的文件。

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

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](https://docs.microsoft.com/powershell/azure/overview)。

您可以在 [Azure 網路概觀文件](../powershell-samples.md?toc=%2fazure%2fnetworking%2ftoc.json)中找到其他網路 PowerShell 指令碼範例。
