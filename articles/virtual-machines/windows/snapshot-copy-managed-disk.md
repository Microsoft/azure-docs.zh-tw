---
title: 在 Azure 中建立 VHD 的快照集 | Microsoft Docs
description: 了解如何建立 Azure VHD 的複本作為備份，或用來針對問題進行疑難排解。
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 15eb778e-fc07-45ef-bdc8-9090193a6d20
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 04/10/2018
ms.author: cynthn
ms.openlocfilehash: 7d45fd749fea4036d944d740541d8b8607553835
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34658150"
---
# <a name="create-a-snapshot"></a>建立快照集

製作作業系統或資料磁碟 VHD 的快照集作為備份，或是用來針對 VM 問題進行疑難排解。 快照集是完整的 VHD 唯讀複本。 

## <a name="use-azure-portal-to-take-a-snapshot"></a>使用 Azure 入口網站建立快照集 

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 從左上方開始，按一下 [建立資源] 並搜尋**快照集**。
3. 在 [快照集] 刀鋒視窗中，按一下 [建立]。
4. 輸入快照集的 [名稱]。
5. 選取現有的[資源群組](../../azure-resource-manager/resource-group-overview.md#resource-groups)，或輸入新群組的名稱。 
6. 選取 Azure 資料中心的 [位置]。  
7. 在 [來源磁碟] 中，選取要建立快照集的受控磁碟。
8. 選取用來儲存快照集的 [帳戶類型]。 除非需要儲存在高效能磁碟上，否則建議選取 **Standard_LRS**。
9. 按一下頁面底部的 [新增] 。

## <a name="use-powershell-to-take-a-snapshot"></a>使用 PowerShell 建立快照集

下列步驟示範如何取得要複製的 VHD 磁碟、建立快照集設定，以及使用 [New-AzureRmSnapshot Cmdlet](/powershell/module/azurerm.compute/new-azurermsnapshot) 建立磁碟的快照集。 

開始之前，請確定您擁有最新版的 AzureRM.Compute PowerShell 模組。 本文需要 AzureRM 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

設定部分參數。 

 ```azurepowershell-interactive
$resourceGroupName = 'myResourceGroup' 
$location = 'eastus' 
$vmName = 'myVM'
$snapshotName = 'mySnapshot'  
```

取得 VM。

 ```azurepowershell-interactive
$vm = get-azurermvm `
   -ResourceGroupName $resourceGroupName `
   -Name $vmName
```

建立快照集組態。 在此範例中，我們將建立作業系統磁碟的快照集。

 ```azurepowershell-interactive
$snapshot =  New-AzureRmSnapshotConfig `
   -SourceUri $vm.StorageProfile.OsDisk.ManagedDisk.Id `
   -Location $location `
   -CreateOption copy
```
   
> [!NOTE]
> 如果您想要將快照集儲存於區域復原的儲存體中，則需要在支援[可用性區域](../../availability-zones/az-overview.md)且包含 `-SkuName Standard_ZRS` 參數的區域中建立它。   

   
建立快照集。

```azurepowershell-interactive
New-AzureRmSnapshot `
   -Snapshot $snapshot `
   -SnapshotName $snapshotName `
   -ResourceGroupName $resourceGroupName 
```




## <a name="next-steps"></a>後續步驟

從快照集建立受控磁碟，然後連結新的受控磁碟作為 OS 磁碟，以從快照集建立虛擬機器。 如需詳細資訊，請參閱[從快照集建立 VM](./../scripts/virtual-machines-windows-powershell-sample-create-vm-from-snapshot.md?toc=%2fpowershell%2fmodule%2ftoc.json) 範例。
