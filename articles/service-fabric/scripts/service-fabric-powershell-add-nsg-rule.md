---
title: Azure PowerShell 指令碼範例 - 新增網路安全性群組規則 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 新增網路安全性群組，以便允許特定連接埠上的輸入流量。
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
ms.date: 11/28/2017
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: fd3c648ee63c45bef305658832a4d31dfdb213be
ms.sourcegitcommit: 29bac59f1d62f38740b60274cb4912816ee775ea
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/29/2017
ms.locfileid: "25987189"
---
# <a name="add-an-inbound-network-security-group-rule"></a>新增輸入網路安全性群組規則

本範例指令碼會建立網路安全性群組規則，以允許連接埠 8081 上的輸入流量。  指令碼會取得叢集所在的 `Microsoft.Network/networkSecurityGroups` 資源、建立新的網路安全性設定規則，並更新網路安全性群組。 視需要自訂參數。

您可以視需要使用 [Azure PowerShell 指南](/powershell/azure/overview)中的指示來安裝 Azure PowerShell。 

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/service-fabric/add-inbound-nsg-rule/add-inbound-nsg-rule.ps1 "Update the RDP port range values")]

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [Get-AzureRmResource](/powershell/module/azurerm.resources/get-azurermresource) | 取得 `Microsoft.Network/networkSecurityGroups` 資源。 |
|[Get-AzureRmNetworkSecurityGroup](/powershell/module/azurerm.network/get-azurermnetworksecuritygroup)| 依名稱取得網路安全性群組。|
|[Add-AzureRmNetworkSecurityRuleConfig](/powershell/module/azurerm.network/add-azurermnetworksecurityruleconfig)| 將網路安全性規則設定新增到網路安全性群組中。 |
|[Set-AzureRmNetworkSecurityGroup](/powershell/module/azurerm.network/set-azurermnetworksecuritygroup)| 設定網路安全性群組的目標狀態。|

## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。
