---
title: Azure PowerShell 指令碼範例 - 變更 RDP 連接埠範圍 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 變更已部署之叢集的 RDP 連接埠範圍。
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
ms.date: 03/19/2018
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: 83fb6cc03f605a60b06f31fa6ddd82cd4e3e899e
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30180182"
---
# <a name="update-the-rdp-port-range-values"></a>更新 RDP 連接埠範圍值

此指令碼範例會在叢集已部署完成之後，變更叢集節點 VM 上的 RDP 連接埠範圍值。  為了讓基礎 VM 不會循環，所以使用 Azure PowerShell。  此指令碼會取得叢集之資源群組中的 `Microsoft.Network/loadBalancers` 資源，並更新 `inboundNatPools.frontendPortRangeStart` 和 `inboundNatPools.frontendPortRangeEnd` 值。 視需要自訂參數。

您可以視需要使用 [Azure PowerShell 指南 (英文)](/powershell/azure/overview) 中的指示來安裝 Azure PowerShell。 

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/service-fabric/change-rdp-port-range/change-rdp-port-range.ps1 "Update the RDP port range values")]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [Get-AzureRmResource](/powershell/module/azurerm.resources/get-azurermresource) | 取得 `Microsoft.Network/loadBalancers` 資源。 |
|[Set-AzureRmResource](/powershell/module/azurerm.resources/set-azurermresource)|更新 `Microsoft.Network/loadBalancers` 資源。|

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure PowerShell 範例](../service-fabric-powershell-samples.md)中找到適用於 Azure Service Fabric 的其他 Azure PowerShell 範例。
