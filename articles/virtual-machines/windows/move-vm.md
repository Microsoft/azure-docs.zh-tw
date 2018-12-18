---
title: 移動 Azure 中的 Windows VM 資源 | Microsoft Docs
description: 在 Resource Manager 部署模型中將 Windows VM 移至另一個 Azure 訂用帳戶或資源群組。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 4e383427-4aff-4bf3-a0f4-dbff5c6f0c81
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/06/2017
ms.author: cynthn
ms.openlocfilehash: 168ba57399b2649af29820f7321dd0151618346e
ms.sourcegitcommit: e0834ad0bad38f4fb007053a472bde918d69f6cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37436475"
---
# <a name="move-a-windows-vm-to-another-azure-subscription-or-resource-group"></a>將 Windows VM 移至另一個 Azure 訂用帳戶或資源群組
本文將逐步引導您了解如何在資源群組或訂用帳戶之間移動 Windows VM。 如果您原本在個人訂用帳戶中建立 VM，而現在想要將它移至您的公司訂用帳戶以繼續工作，在訂用帳戶之間移動會很方便。

> [!IMPORTANT]
>此時，您無法移動受控磁碟。 
>
>移動過程中會建立新的資源識別碼。 移動 VM 之後，您必須更新工具和指令碼以使用新的資源識別碼。 
> 
> 

[!INCLUDE [virtual-machines-common-move-vm](../../../includes/virtual-machines-common-move-vm.md)]

## <a name="use-powershell-to-move-a-vm"></a>使用 PowerShell 移動 VM

若要將虛擬機器移至另一個資源群組，您必須確定也要移動所有的相依資源。 若要使用 Move-AzureRMResource Cmdlet，您需要每項資源的 ResourceId。 您可以使用 [Get-AzureRMResource](/powershell/module/azurerm.resources/get-azurermresource) Cmdlet 取得 ResourceId 清單。

```azurepowershell-interactive
 Get-AzureRMResource -ResourceGroupName <sourceResourceGroupName> | Format-table -Property ResourceId 
```

若要移動 VM，我們需要移動多個資源。 我們可以使用 Get-AzureRMResource 的輸出來建立以逗號分隔的 ResourceId 清單並傳遞至 [Move-AzureRMResource](/powershell/module/azurerm.resources/move-azurermresource)，以將資源移到目的地。 

```azurepowershell-interactive

Move-AzureRmResource -DestinationResourceGroupName "<myDestinationResourceGroup>" `
    -ResourceId <myResourceId,myResourceId,myResourceId>
```
    
若要將資源移到不同的訂用帳戶，請納入 **DestinationSubscriptionId** 參數。 

```azurepowershell-interactive
Move-AzureRmResource -DestinationSubscriptionId "<myDestinationSubscriptionID>" `
    -DestinationResourceGroupName "<myDestinationResourceGroup>" `
    -ResourceId <myResourceId,myResourceId,myResourceId>
```


系統會要求您確認您想要移動指定的資源。 

## <a name="next-steps"></a>後續步驟
您可以在資源群組和訂用帳戶之間移動許多不同類型的資源。 如需詳細資訊，請參閱 [將資源移動到新的資源群組或訂用帳戶](../../resource-group-move-resources.md)。    

