---
title: 在 Azure 中建立受控映像 | Microsoft Docs
description: 在 Azure 中建立一般化 VM 或 VHD 的受控映像。 映像可用來建立多部使用受控磁碟的 VM。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 04/10/2018
ms.author: cynthn
ms.openlocfilehash: 4445787fd559c6d0a6dfc891910cb9a139a6907e
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31602564"
---
# <a name="create-a-managed-image-of-a-generalized-vm-in-azure"></a>在 Azure 中建立一般化 VM 的受控映像

您可以從在儲存體帳戶中儲存為受控磁碟或非受控磁碟的一般化 VM，建立受控映像資源。 映像接著便可用來建立多個 VM。 

## <a name="generalize-the-windows-vm-using-sysprep"></a>使用 Sysprep 將 Windows VM 一般化

Sysprep 會移除您的所有個人帳戶資訊以及其他項目，並準備電腦以做為映像。 如需 Sysprep 的詳細資訊，請參閱 [如何使用 Sysprep：簡介](http://technet.microsoft.com/library/bb457073.aspx)。

請確定 Sysprep 支援電腦上執行的伺服器角色。 如需詳細資訊，請參閱 [Sysprep Support for Server Roles (伺服器角色的 Sysprep 支援)](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)

> [!IMPORTANT]
> 當您在 VM 上執行 sysprep 之後，系統就會將它視為*一般化*且無法將它重新啟動。 將 VM 一般化的程序是無法復原的。 如果您需要讓原始的 VM 保持運作，就應該取得 [VM 的複本](create-vm-specialized.md#option-3-copy-an-existing-azure-vm)，然後將複本一般化。 
>
> 如果您是第一次在將 VHD 上傳至 Azure 之前執行 Sysprep，請確定您已[準備好 VM](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 再執行 Sysprep。  
> 
> 

1. 登入 Windows 虛擬機器。
2. 以系統管理員身分開啟 [命令提示字元] 視窗。 切換至 **%windir%\system32\sysprep** 目錄，然後執行 `sysprep.exe`。
3. 在 [系統準備工具] 對話方塊中，選取 [進入系統全新體驗 (OOBE)]，並確認已勾選 [一般化] 核取方塊。
4. 在 [關機選項] 中選取 [關機]。
5. 按一下 [SERVICEPRINCIPAL] 。
   
    ![啟動 Sysprep](./media/upload-generalized-managed/sysprepgeneral.png)
6. Sysprep 完成時，會關閉虛擬機器。 不要重新啟動 VM。


## <a name="create-a-managed-image-in-the-portal"></a>在入口網站中建立受控映像 

1. 開啟[入口網站](https://portal.azure.com)。
2. 在左側功能表中，按一下 [虛擬機器]，然後從清單中選取 VM。
3. 在該 VM 的頁面中，按一下上方功能表中的 [擷取]。
3. 在 [名稱] 中，輸入您要用於映像的名稱。
4. 在 [資源群組] 中，選取 [新建] 並輸入名稱，或是選取 [使用現有項目]，然後從下拉式清單中選取要使用的資源群組。
5. 如果您想要在建立映像之後刪除來源 VM，請選取 [自動在建立映像後刪除此虛擬機器]。
6. 完成之後，請按一下 [建立]。
16. 建立映像之後，您會在資源群組的資源清單中看見它顯示為 [映像] 資源。



## <a name="create-an-image-of-a-vm-using-powershell"></a>使用 Powershell 建立 VM 的映像

直接從 VM 建立映像，可確保映像包含 VM 的所有相關磁碟，包括 OS 磁碟和任何資料磁碟。 此範例示範如何從使用受控磁碟的 VM 建立受控映像。


開始之前，請確定您擁有最新版的 AzureRM.Compute PowerShell 模組。 本文需要 AzureRM 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。


> [!NOTE]
> 如果您想要將映像儲存於區域復原的儲存體中，則需要在支援[可用性區域](../../availability-zones/az-overview.md)且在映像設定中包含 `-ZoneResilient` 參數的區域中建立它。


1. 建立一些變數。

    ```azurepowershell-interactive
    $vmName = "myVM"
    $rgName = "myResourceGroup"
    $location = "EastUS"
    $imageName = "myImage"
    ```
2. 確定已解除配置 VM。

    ```azurepowershell-interactive
    Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName -Force
    ```
    
3. 將虛擬機器的狀態設定為 [一般化] 。 
   
    ```azurepowershell-interactive
    Set-AzureRmVm -ResourceGroupName $rgName -Name $vmName -Generalized
    ```
    
4. 取得虛擬機器。 

    ```azurepowershell-interactive
    $vm = Get-AzureRmVM -Name $vmName -ResourceGroupName $rgName
    ```

5. 建立映像組態。

    ```azurepowershell-interactive
    $image = New-AzureRmImageConfig -Location $location -SourceVirtualMachineId $vm.ID 
    ```
6. 建立映像。

    ```azurepowershell-interactive
    New-AzureRmImage -Image $image -ImageName $imageName -ResourceGroupName $rgName
    ``` 
## <a name="create-an-image-from-a-managed-disk-using-powershell"></a>使用 PowerShell 從受控磁碟建立受控映像

如果您只想要建立 OS 磁碟的映像，您也可以將受控磁碟識別碼指定為 OS 磁碟來建立映像。

    
1. 建立一些變數。 

    ```azurepowershell-interactive
    $vmName = "myVM"
    $rgName = "myResourceGroup"
    $location = "EastUS"
    $snapshotName = "mySnapshot"
    $imageName = "myImage"
    ```

2. 取得 VM。

   ```azurepowershell-interactive
   $vm = Get-AzureRmVm -Name $vmName -ResourceGroupName $rgName
   ```

3. 取得受控磁碟的識別碼。

    ```azurepowershell-interactive
    $diskID = $vm.StorageProfile.OsDisk.ManagedDisk.Id
    ```
   
3. 建立映像組態。

    ```azurepowershell-interactive
    $imageConfig = New-AzureRmImageConfig -Location $location
    $imageConfig = Set-AzureRmImageOsDisk -Image $imageConfig -OsState Generalized -OsType Windows -ManagedDiskId $diskID
    ```
    
4. 建立映像。

    ```azurepowershell-interactive
    New-AzureRmImage -ImageName $imageName -ResourceGroupName $rgName -Image $imageConfig
    ``` 


## <a name="create-an-image-from-a-snapshot-using-powershell"></a>使用 Powershell 從快照集建立映像

您可以從一般化 VM 的快照集建立受控映像。

    
1. 建立一些變數。 

    ```azurepowershell-interactive
    $rgName = "myResourceGroup"
    $location = "EastUS"
    $snapshotName = "mySnapshot"
    $imageName = "myImage"
    ```

2. 取得快照集。

   ```azurepowershell-interactive
   $snapshot = Get-AzureRmSnapshot -ResourceGroupName $rgName -SnapshotName $snapshotName
   ```
   
3. 建立映像組態。

    ```azurepowershell-interactive
    $imageConfig = New-AzureRmImageConfig -Location $location
    $imageConfig = Set-AzureRmImageOsDisk -Image $imageConfig -OsState Generalized -OsType Windows -SnapshotId $snapshot.Id
    ```
4. 建立映像。

    ```azurepowershell-interactive
    New-AzureRmImage -ImageName $imageName -ResourceGroupName $rgName -Image $imageConfig
    ``` 


## <a name="create-image-from-a-vhd-in-a-storage-account"></a>從儲存體帳戶中的 VHD 建立映像

從儲存體帳戶中的一般化 OS VHD 建立受控映像。 您需要儲存體帳戶中 VHD 的 URI，其格式為 https://*mystorageaccount*.blob.core.windows.net/*container*/*vhd_filename.vhd*。 在此範例中，我們使用的 VHD 位於名為 *vhdcontainer* 之容器的 *mystorageaccount* 中，且 VHD 檔案名稱為 *osdisk.vhd*。


1.  首先，設定一般參數：

    ```azurepowershell-interactive
    $vmName = "myVM"
    $rgName = "myResourceGroup"
    $location = "EastUS"
    $imageName = "myImage"
    $osVhdUri = "https://mystorageaccount.blob.core.windows.net/vhdcontainer/osdisk.vhd"
    ```
2. 停止\解除配置 VM。

    ```azurepowershell-interactive
    Stop-AzureRmVM -ResourceGroupName $rgName -Name $vmName -Force
    ```
    
3. 將 VM 標示為一般化。

    ```azurepowershell-interactive
    Set-AzureRmVm -ResourceGroupName $rgName -Name $vmName -Generalized 
    ```
4.  使用一般化 OS VHD 建立映像。

    ```azurepowershell-interactive
    $imageConfig = New-AzureRmImageConfig -Location $location
    $imageConfig = Set-AzureRmImageOsDisk -Image $imageConfig -OsType Windows -OsState Generalized -BlobUri $osVhdUri
    $image = New-AzureRmImage -ImageName $imageName -ResourceGroupName $rgName -Image $imageConfig
    ```

    
## <a name="next-steps"></a>後續步驟
- 您現在可以[從一般化受控映像建立 VM](create-vm-generalized-managed.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。  

