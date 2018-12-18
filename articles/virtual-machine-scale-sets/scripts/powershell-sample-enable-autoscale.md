---
title: Azure PowerShell 範例 - 啟用主機型自動調整 | Microsoft Docs
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
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: ca484a6f1b2dafa3688efe0e50bb2c35f954a8b6
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38697410"
---
# <a name="automatically-scale-a-virtual-machine-scale-set-with-powershell"></a>使用 PowerShell 自動調整虛擬機器擴展集
此指令碼會建立執行 Windows Server 2016 的虛擬機器擴展集，並在 CPU 負載變更時，使用主機型計量來進行自動調整。

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼
[!code-powershell[main](../../../powershell_scripts/virtual-machine-scale-sets/auto-scale-host-metrics/auto-scale-host-metrics.ps1 "Automatically scale a virtual machine scale set")]

## <a name="clean-up-deployment"></a>清除部署
執行下列命令來移除資源群組、擴展集和所有相關資源。

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明
此指令碼會使用下列命令來建立部署。 下表中的每個項目都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [New-AzureRmVmss](/powershell/module/azurerm.compute/new-azurermvmss) | 建立虛擬機器擴展集和所有支援的資源，包括虛擬網路、負載平衡器和 NAT 規則。 |
| [Get-AzureRmVmss](/powershell/module/azurerm.compute/get-azurermvmss) | 取得虛擬機器擴展集的相關資訊。 |
| [Add-AzureRmVmssExtension](/powershell/module/azurerm.compute/add-azurermvmssextension) | 新增自訂指令碼的 VM 擴充來安裝基本 Web 應用程式。 |
| [Update-AzureRmVmss](/powershell/module/azurerm.compute/update-azurermvmss) | 更新虛擬機器擴展集模型以套用 VM 擴充。 |
| [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) | 取得指派給負載平衡器使用的公用 IP 位址資訊。 |
| [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 移除資源群組及其內含的所有資源。 |

## <a name="next-steps"></a>後續步驟
如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure 虛擬機器擴展集文件](../powershell-samples.md)中找到其他的虛擬機器擴展集 PowerShell 指令碼範例。