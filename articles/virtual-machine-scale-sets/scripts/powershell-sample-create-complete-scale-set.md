---
title: Azure PowerShell 範例 - 建立完整的虛擬機器擴展集 | Microsoft Docs
description: Azure PowerShell 範例
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/29/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: c92b922adeb2e3cd80a271062704feb3bdb4c949
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38652151"
---
# <a name="create-a-complete-virtual-machine-scale-set-with-powershell"></a>使用 PowerShell 建立完整的虛擬機器擴展集
此指令碼會建立執行 Windows Server 2016 的虛擬機器擴展集。 此時會設定和建立個別資源，而不是使用 [New-AzureRmVmss 中提供於此處的內建資源建立選項](powershell-sample-create-simple-scale-set.md)。 執行指令碼之後，您可以透過 RDP 存取 VM 執行個體。

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼
[!code-powershell[main](../../../powershell_scripts/virtual-machine-scale-sets/complete-scale-set/complete-scale-set.ps1 "Create a complete virtual machine scale set")]

## <a name="clean-up-deployment"></a>清除部署
執行下列命令來移除資源群組、擴展集和所有相關資源。

```powershell
Remove-AzureRmResourceGroup -Name $resourceGroupName
```

## <a name="script-explanation"></a>指令碼說明
此指令碼會使用下列命令來建立部署。 下表中的每個項目都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立用來存放所有資源的資源群組。 |
| [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) | 建立子網路組態。 此組態可使用於虛擬網路建立程序。 |
| [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) | 建立虛擬網路。 |
| [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) | 建立公用 IP 位址。 |
| [New-AzureRmLoadBalancerFrontendIpConfig](/powershell/module/azurerm.network/new-azurermloadbalancerfrontendipconfig) | 建立負載平衡器的前端 IP 組態。 |
| [New-AzureRmLoadBalancerBackendAddressPoolConfig](/powershell/module/azurerm.network/new-azurermloadbalancerbackendaddresspoolconfig) | 建立負載平衡器的後端位址集區設定。 |
| [New-AzureRmLoadBalancerInboundNatRuleConfig](/powershell/module/azurerm.network/new-azurermloadbalancerinboundnatruleconfig) | 建立負載平衡器的輸入 NAT 規則組態。 |
| [New-AzureRmLoadBalancer](/powershell/module/azurerm.network/new-azurermloadbalancer) | 建立負載平衡器。 |
| [Add-AzureRmLoadBalancerProbeConfig](/powershell/module/azurerm.network/new-azurermloadbalancerprobeconfig) | 建立負載平衡器的探查組態。 |
| [Add-AzureRmLoadBalancerRuleConfig](/powershell/module/azurerm.network/new-azurermloadbalancerruleconfig) | 建立負載平衡器的規則組態。 |
| [Set-AzureRmLoadBalancer](/powershell/module/AzureRM.Network/Set-AzureRmLoadBalancer) | 使用提供的資訊更新負載平衡器。 |
| [New-AzureRmVmssIpConfig](/powershell/module/AzureRM.Compute/New-AzureRmVmssIpConfig) | 建立擴展集 VM 執行個體的 IP 組態。 VM 執行個體會連線至負載平衡器後端集區、NAT 集區和虛擬網路子網路。 |
| [New-AzureRmVmssConfig](/powershell/module/AzureRM.Compute/New-AzureRmVmssConfig) | 建立擴展集組態。 此組態會包含多項資訊，例如要建立的 VM 執行個體數目、VM SKU (大小)，以及升級原則模式等。 此組態可藉由其他 Cmdlet 來新增，並在擴展集建立期間使用。 |
| [Set-AzureRmVmssStorageProfile](/powershell/module/AzureRM.Compute/Set-AzureRmVmssStorageProfile) | 定義要用於 VM 執行個體的映像，並將其新增至擴展集組態。 |
| [Set-AzureRmVmssOsProfile](/powershell/module/AzureRM.Compute/Set-AzureRmVmssStorageProfile) | 定義系統管理使用者名稱和密碼認證，以及 VM 的命名前置詞。 請將這些值新增至擴展集組態。 |
| [Add-AzureRmVmssNetworkInterfaceConfiguration](/powershell/module/AzureRM.Compute/Add-AzureRmVmssNetworkInterfaceConfiguration) | 根據 IP 組態，將虛擬網路介面新增至 VM 執行個體。 請將這些值新增至擴展集組態。 |
| [New-AzureRmVmss](/powershell/module/AzureRM.Compute/New-AzureRmVmss) | 根據擴展集組態中提供的資訊來建立擴展集。 |
| [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 移除資源群組及其內含的所有資源。 |

## <a name="next-steps"></a>後續步驟
如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure 虛擬機器擴展集文件](../powershell-samples.md)中找到其他的虛擬機器擴展集 PowerShell 指令碼範例。
