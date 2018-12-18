---
title: Azure PowerShell 指令碼範例 - 在負載平衡器中開啟應用程式連接埠 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 在 Azure Load Balancer 中開啟 Service Fabric 應用程式的連接埠。
services: service-fabric
documentationcenter: ''
author: rwike77
manager: timlt
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: service-fabric
ms.workload: multiple
ms.devlang: na
ms.topic: sample
ms.date: 05/18/2018
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: 0549f5f2b5b0f8fdfc18b8c091c1065d6137b8c6
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34366166"
---
# <a name="open-an-application-port-in-the-azure-load-balancer"></a>在 Azure Load Balancer 中開啟應用程式連接埠

在 Azure 中執行的 Service Fabric 應用程式位於 Azure Load Balancer 後方。 這個範例指令碼會在 Azure Load Balancer 中開啟連接埠，以便 Service Fabric 應用程式可以與外部用戶端通訊。 視需要自訂參數。 如果您的叢集位於網路安全性群組中，也要[新增輸入網路安全性群組規則](service-fabric-powershell-add-nsg-rule.md)，以允許輸入流量。

如有需要，可隨同 [Service Fabric SDK](../service-fabric-get-started.md) 一起安裝 Service Fabric PowerShell 模組。 

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/service-fabric/open-port-in-load-balancer/open-port-in-load-balancer.ps1 "Open a port in the load balancer")]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [Get-AzureRmResource](/powershell/module/azurerm.resources/get-azurermresource) | 取得 Azure 資源。  |
| [Get-AzureRmLoadBalancer](/powershell/module/azurerm.network/get-azurermloadbalancer) | 取得 Azure Load Balancer。 |
| [Add-AzureRmLoadBalancerProbeConfig](/powershell/module/azurerm.network/add-azurermloadbalancerprobeconfig) | 將探查組態新增至負載平衡器。|
| [Get-AzureRmLoadBalancerProbeConfig](/powershell/module/azurerm.network/get-azurermloadbalancerprobeconfig) | 取得負載平衡器的探查組態。 |
| [Add-AzureRmLoadBalancerRuleConfig](/powershell/module/azurerm.network/add-azurermloadbalancerruleconfig) | 將規則組態新增至負載平衡器。 |
| [Set-AzureRmLoadBalancer](/powershell/module/azurerm.network/set-azurermloadbalancer) | 設定負載平衡器的目標狀態。 |

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [PowerShell 範例](../service-fabric-powershell-samples.md)中找到適用於 Azure Service Fabric 的其他 Azure PowerShell 範例。
