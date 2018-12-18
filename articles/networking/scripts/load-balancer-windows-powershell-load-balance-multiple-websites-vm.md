---
title: Azure PowerShell 指令碼範例 - 使用 Azure PowerShell 進行多個網站的負載平衡 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 將多個網站負載平衡至相同的虛擬機器
services: load-balancer
documentationcenter: load-balancer
author: georgewallace
manager: timlt
editor: tysonn
tags: ''
ms.assetid: ''
ms.service: load-balancer
ms.devlang: powershell
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 05/16/2017
ms.author: gwallace
ms.openlocfilehash: 44cb76a09cdaacd1bffd653bae6a5735a81dc6f6
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31597814"
---
# <a name="load-balance-multiple-websites"></a>進行多個網站的負載平衡

此指令碼範例使用可用性設定組成員的兩個虛擬機器 (VM) 建立虛擬網路。 負載平衡器會將兩個不同 IP 位址的流量傳送至這兩個 VM。 執行指令碼之後，您可以將 Web 伺服器軟體部署至 VM 並主控多個網站 (各有自己的 IP 位址)。

您可以視需要使用 [Azure PowerShell 指南](https://docs.microsoft.com/powershell/azureps-cmdlets-docs/) \(英文\) 中的指示來安裝 Azure PowerShell，然後執行 `Connect-AzureRmAccount` 來建立與 Azure 的連線。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/load-balancer/load-balance-multiple-web-sites-vm/load-balance-multiple-web-sites-vm.ps1  "Load balance multiple web sites")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、VM 和所有相關資源。

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令建立資源群組、虛擬網路、負載平衡器和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立用來存放所有資源的資源群組。 |
| [New-AzureRmAvailabilitySet](/powershell/module/azurerm.compute/new-azurermavailabilityset) | 建立 Azure 可用性設定組以提供高可用性。 |
| [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) | 建立子網路組態。 此組態可使用於虛擬網路建立程序。 |
| [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) | 建立虛擬網路。 |
| [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) | 建立公用 IP 位址。 |
| [New-AzureRmLoadBalancerFrontendIpConfig](/powershell/module/azurerm.network/new-azurermloadbalancerfrontendipconfig) | 建立負載平衡器的前端 IP 設定。 |
| [New-AzureRmLoadBalancerBackendAddressPoolConfig](/powershell/module/azurerm.network/new-azurermloadbalancerbackendaddresspoolconfig) | 建立負載平衡器的後端位址集區設定。 |
| [New-AzureRmLoadBalancerProbeConfig](/powershell/module/azurerm.network/new-azurermloadbalancerprobeconfig) | 建立 NLB 探查。 NLB 探查可用來監視 NLB 集合中的每部 VM。 如有任何 VM 變得無法存取，就不會將流量路由至該 VM。 |
| [New-AzureRmLoadBalancerRuleConfig](/powershell/module/azurerm.network/new-azurermloadbalancerruleconfig) | 建立 NLB 規則。 在此範例中，會為連接埠 80 建立規則。 當 HTTP 流量抵達 NLB 時，就會經由連接埠 80 將其路由傳送至 NLB 集合的其中一部 VM。 |
| [New-AzureRmLoadBalancer](/powershell/module/azurerm.network/new-azurermloadbalancer) | 建立負載平衡器。 |
| [New-AzureRmNetworkInterfaceIpConfig](/powershell/module/azurerm.network/new-azurermnetworkinterfaceipconfig) | 定義虛擬網路介面的進階功能。 |
| [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) | 建立網路介面。 |
| [New-AzureRmVMConfig](/powershell/module/azurerm.compute/new-azurermvmconfig) | 建立 VM 組態。 此組態包括 VM 名稱、作業系統和系統管理認證等資訊。 建立 VM 時會使用此組態。 |
| [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) | 建立虛擬機器。 |
|[Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 移除資源群組及其內含的所有資源。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 的詳細資訊，請參閱 [Azure PowerShell 文件](https://docs.microsoft.com/powershell/azure/overview)。

您可以在 [Azure 網路概觀文件](../powershell-samples.md?toc=%2fazure%2fnetworking%2ftoc.json)中找到其他網路 PowerShell 指令碼範例。
