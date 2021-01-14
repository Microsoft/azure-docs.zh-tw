---
title: 將資料磁碟與 Windows VM 中斷連結 - Azure
description: 使用 Resource Manger 部署模型，將資料磁碟與 Azure 中的虛擬機器中斷連結。
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 01/08/2020
ms.author: cynthn
ms.openlocfilehash: cae75c88b4803912565e010f744a7757a3b98f04
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98201547"
---
# <a name="how-to-detach-a-data-disk-from-a-windows-virtual-machine"></a>如何從 Windows 虛擬機器卸離資料磁碟

當不再需要某個連接至虛擬機器的資料磁碟時，卸離此資料磁碟很簡單。 這會將磁碟從虛擬機器中卸離，但這不會將它從儲存體中移除。

> [!WARNING]
> 將磁碟中斷連結時，並不會自動將它刪除。 如果您已訂閱「進階」儲存體，您將會繼續因該磁碟而導致產生儲存體費用。 如需詳細資訊，請參閱[使用進階儲存體時的定價和計費](../disks-types.md#billing)。

如果您想要再次使用磁碟上現有的資料，您可以將磁碟重新連接至相同或其他虛擬機器。

 

## <a name="detach-a-data-disk-using-powershell"></a>使用 PowerShell 來中斷資料磁碟連結

您可以使用 PowerShell 來「熱」移除資料磁碟，但是在從 VM 卸離磁碟之前，請確定沒有任何項目正在使用該磁碟。

在此範例中，我們會從 **myResourceGroup** 資源群組中的 **myVM** 移除名為 **myDisk** 的磁碟。 首先使用 [Remove-AzVMDataDisk](/powershell/module/az.compute/remove-azvmdatadisk) Cmdlet 來移除磁碟。 接著，您會使用 [Update-AzVM](/powershell/module/az.compute/update-azvm) Cmdlet 來更新虛擬機器的狀態，以完成移除資料磁碟的程序。

```azurepowershell-interactive
$VirtualMachine = Get-AzVM `
   -ResourceGroupName "myResourceGroup" `
   -Name "myVM"
Remove-AzVMDataDisk `
   -VM $VirtualMachine `
   -Name "myDisk"
Update-AzVM `
   -ResourceGroupName "myResourceGroup" `
   -VM $VirtualMachine
```

磁碟仍留在儲存體中，但不再連結至虛擬機器。

## <a name="detach-a-data-disk-using-the-portal"></a>使用入口網站來中斷資料磁碟連結

您可「熱」移除資料磁碟，但請先確定沒有任何項目正在使用該磁碟，再與 VM 中斷連結。

1. 在左窗格中，選取 [虛擬機器]。
1. 選取虛擬機器，其具有想要與其中斷連結的資料磁碟。
1. 在 [設定] 下，選取 [磁碟]。
1. 在 [ **磁片** ] 窗格中，按一下您想要中斷連結之資料磁片的最右邊，然後按一下 [ **X** 刪除] 按鈕。
1. 選取頁面頂端的 [儲存] 來儲存變更。

磁碟仍留在儲存體中，但不再連結至虛擬機器。

## <a name="next-steps"></a>後續步驟

如果您想要重複使用該資料磁碟，只要 [將它連結至另一個 VM](attach-managed-disk-portal.md)
