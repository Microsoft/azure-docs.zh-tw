---
title: Azure PowerShell 指令碼範例 - 讓兩個虛擬網路對等互連 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 讓兩個虛擬網路對等互連
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
ms.openlocfilehash: 4fe4777b3adf585b2b6ddc862023905e756404d9
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31599973"
---
# <a name="peer-two-virtual-networks-script-sample"></a>讓兩個虛擬網路對等互連的指令碼範例

此指令碼範例會在同一個區域中建立兩個虛擬網路，並透過 Azure 網路連接這兩個虛擬網路。 執行此指令碼之後，您將會在兩個虛擬網路之間建立對等互連。

您可以從 Azure [Cloud Shell](https://shell.azure.com/powershell) 或從本機的 PowerShell 安裝來執行指令碼。 如果您在本機使用 PowerShell，此指令碼需要使用 AzureRM PowerShell 模組 5.4.1 版或更新版本。 若要尋找已安裝的版本，請執行 `Get-Module -ListAvailable AzureRM`。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurepowershell-interactive[main](../../../powershell_scripts/virtual-network/peer-two-virtual-networks/peer-two-virtual-networks.ps1 "Peer two networks")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源：

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、虛擬機器和所有相關資源。 下表中的每個命令都會連結至命令特定的文件：

| 命令 | 注意 |
|---|---|
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立用來存放所有資源的資源群組。 | 
| [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork)| 建立 Azure 虛擬網路和子網路。 |
| [Add-AzureRmVirtualNetworkPeering](/powershell/module/azurerm.network/add-azurermvirtualnetworkpeering) | 在兩個虛擬網路之間建立對等互連。  |
| [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

可以在[虛擬網路 PowerShell 範例](../powershell-samples.md)中找到其他的虛擬網路 PowerShell 指令碼範例。
