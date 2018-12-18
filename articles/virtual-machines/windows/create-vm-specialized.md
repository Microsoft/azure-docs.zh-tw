---
title: 從 Azure 中的特製化 VHD 建立 Windows VM | Microsoft Docs
description: 在 Resource Manager 部署模型中連結特製化受控磁碟作為 OS 磁碟，以建立新的 Windows VM。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 3b7d3cd5-e3d7-4041-a2a7-0290447458ea
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 01/09/2018
ms.author: cynthn
ms.openlocfilehash: 34bfe7733c60337d6ab7d81c498d2fb0fd15e1fd
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43338481"
---
# <a name="create-a-windows-vm-from-a-specialized-disk-using-powershell"></a>使用 PowerShell 從特製化磁碟建立 Windows 虛擬機器

連結特製化受控磁碟作為 OS 磁碟，以建立新的虛擬機器。 特製化磁碟是現有 VM 中的虛擬硬碟 (VHD) 複本，可從原始的 VM 維護使用者帳戶、應用程式和其他狀態資料。 

當您使用特製化 VHD 來建立新的 VM 時，新的 VM 會保留原始 VM 的電腦名稱。 系統也會保留其他電腦特定資訊，在某些情況下，此重複資訊可能會造成問題。 複製 VM 時，請留意您的應用程式會依賴哪些類型的電腦特定資訊。

您有幾種選項：
* [使用現有受控磁碟](#option-1-use-an-existing-disk)。 如果您有無法正常運作的虛擬機器，這會十分實用。 您可以刪除虛擬機器，重複使用受控磁碟來建立新的虛擬機器。 
* [上傳 VHD](#option-2-upload-a-specialized-vhd) 
* [使用快照集複製現有的 Azure 虛擬機器](#option-3-copy-an-existing-azure-vm)

您也可以使用 Azure 入口網站，[從特定的 VHD 建立新的虛擬機器](create-vm-specialized-portal.md)。

本主題說明如何使用受控磁碟。 如果您的舊版部署需要使用儲存體帳戶，請參閱[從儲存體帳戶中的特殊化 VHD 建立 VM](sa-create-vm-specialized.md)

## <a name="before-you-begin"></a>開始之前
如果您使用 PowerShell，請確定您擁有最新版的 AzureRM.Compute PowerShell 模組。 

```powershell
Install-Module AzureRM -RequiredVersion 6.0.0
```
如需詳細資訊，請參閱 [Azure PowerShell 版本控制](/powershell/azure/overview)。

## <a name="option-1-use-an-existing-disk"></a>選項 1. 使用現有磁碟

如果您已刪除虛擬機器，而想要重複使用 OS 磁碟建立新的虛擬機器，請使用 [Get-AzureRmDisk](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermdisk?view=azurermps-6.8.1)。

```powershell
$resourceGroupName = 'myResourceGroup'
$osDiskName = 'myOsDisk'
$osDisk = Get-AzureRmDisk `
-ResourceGroupName $resourceGroupName `
-DiskName $osDiskName
```
現在，您可以將此磁碟當作 OS 磁碟連結至[新的虛擬機器](#create-the-new-vm)。

## <a name="option-2-upload-a-specialized-vhd"></a>選項 2：上傳特製化 VHD

您可以上傳從特製化的 VM，以在內部部署虛擬化工具，像是從另一個雲端匯出 HYPER-V 或在 VM 建立 VHD。

### <a name="prepare-the-vm"></a>準備 VM
如果您想要使用 VHD 現狀建立新的 VM，請確定完成下列步驟。 
  
  * [準備要上傳至 Azure 的 Windows VHD](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). **不要**使用 Sysprep 一般化 VM。
  * 移除 VM 上 (如 VMware 工具) 已安裝的任何來賓虛擬化工具和代理程式。
  * 確認已透過 DHCP 設定 VM 提取其 IP 位址和 DNS 設定。 這可確保伺服器在啟動時取得 VNet 內的 IP 位址。 


### <a name="get-the-storage-account"></a>取得儲存體帳戶
您需要一個 Azure 中的儲存體帳戶來儲存已上傳的 VHD。 您可以使用現有的儲存體帳戶或建立新帳戶。 

若要顯示可用的儲存體帳戶，請輸入︰

```powershell
Get-AzureRmStorageAccount
```

如果您想要使用現有的儲存體帳戶，請移至[上傳 VHD](#upload-the-vhd-to-your-storage-account) 一節。

如果您需要建立儲存體帳戶，請依照下列步驟操作：

1. 您需要在當中建立儲存體帳戶的資源群組名稱。 若要找出您訂用帳戶中的所有資源群組，請輸入︰
   
    ```powershell
    Get-AzureRmResourceGroup
    ```

    若要在*美國中部*區域建立名為 *myResourceGroup* 的資源群組，請輸入︰

    ```powershell
    New-AzureRmResourceGroup `
       -Name myResourceGroup `
       -Location "West US"
    ```

2. 使用 [New-AzureRmStorageAccount](/powershell/module/azurerm.storage/new-azurermstorageaccount) Cmdlet，在此資源群組中建立名為 *mystorageaccount*的儲存體帳戶：
   
    ```powershell
    New-AzureRmStorageAccount `
       -ResourceGroupName myResourceGroup `
       -Name mystorageaccount `
       -Location "West US" `
       -SkuName "Standard_LRS" `
       -Kind "Storage"
    ```

### <a name="upload-the-vhd-to-your-storage-account"></a>將 VHD 上傳至儲存體帳戶 
使用 [Add-AzureRmVhd](/powershell/module/azurerm.compute/add-azurermvhd) Cmdlet，將 VHD 上傳至儲存體帳戶中的容器。 這個範例會將檔案 *myVHD.vhd* 從 `"C:\Users\Public\Documents\Virtual hard disks\"` 上傳至 *myResourceGroup* 資源群組中名為 *mystorageaccount* 的儲存體帳戶。 此檔案會儲存在名為 *mycontainer* 的容器中，而新的檔案名稱會是 *myUploadedVHD.vhd*。

```powershell
$resourceGroupName = "myResourceGroup"
$urlOfUploadedVhd = "https://mystorageaccount.blob.core.windows.net/mycontainer/myUploadedVHD.vhd"
Add-AzureRmVhd -ResourceGroupName $resourceGroupName `
   -Destination $urlOfUploadedVhd `
   -LocalFilePath "C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd"
```


如果成功，您會得到看起來如以下的回應：

```powershell
MD5 hash is being calculated for the file C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd.
MD5 hash calculation is completed.
Elapsed time for the operation: 00:03:35
Creating new page blob of size 53687091712...
Elapsed time for upload: 01:12:49

LocalFilePath           DestinationUri
-------------           --------------
C:\Users\Public\Doc...  https://mystorageaccount.blob.core.windows.net/mycontainer/myUploadedVHD.vhd
```

視您的網路連線和 VHD 檔案大小而定，此命令可能需要一些時間才能完成

### <a name="create-a-managed-disk-from-the-vhd"></a>從 VHD 建立受控磁碟

使用 [New-AzureRMDisk](/powershell/module/azurerm.compute/new-azurermdisk)，從儲存體帳戶中的特製化 VHD 建立受控磁碟。 此範例會使用 **myOSDisk1** 作為磁碟名稱，將磁碟 *Standard_LRS* 放入儲存體，並使用 *https://storageaccount.blob.core.windows.net/vhdcontainer/osdisk.vhd* 作為來源 VHD 的 URI。

建立新 VM 的新資源群組。

```powershell
$destinationResourceGroup = 'myDestinationResourceGroup'
New-AzureRmResourceGroup -Location $location `
   -Name $destinationResourceGroup
```

從已上傳的 VHD 建立新的 OS 磁碟。 

```powershell
$sourceUri = (https://storageaccount.blob.core.windows.net/vhdcontainer/osdisk.vhd)
$osDiskName = 'myOsDisk'
$osDisk = New-AzureRmDisk -DiskName $osDiskName -Disk `
    (New-AzureRmDiskConfig -AccountType Standard_LRS  `
    -Location $location -CreateOption Import `
    -SourceUri $sourceUri) `
    -ResourceGroupName $destinationResourceGroup
```

## <a name="option-3-copy-an-existing-azure-vm"></a>選項 3：複製現有的 Azure 虛擬機器

您可以製作 VM 的快照集，然後使用該，快照集來建立新的受控磁碟和新的 VM，進而建立使用受控磁碟的 VM 複本。


### <a name="take-a-snapshot-of-the-os-disk"></a>製作 OS 磁碟的快照集

您可以製作整個 VM (包括所有磁碟) 或僅只單一磁碟的快照集。 下列步驟說明如何使用 [New-AzureRmSnapshot](/powershell/module/azurerm.compute/new-azurermsnapshot) Cmdlet，製作僅只您 VM 之 OS 磁碟的快照集。 

設定部分參數。 

 ```powershell
$resourceGroupName = 'myResourceGroup' 
$vmName = 'myVM'
$location = 'westus' 
$snapshotName = 'mySnapshot'  
```

取得 VM 物件。

```powershell
$vm = Get-AzureRmVM -Name $vmName `
   -ResourceGroupName $resourceGroupName
```
取得 OS 磁碟名稱。

 ```powershell
$disk = Get-AzureRmDisk -ResourceGroupName $resourceGroupName `
   -DiskName $vm.StorageProfile.OsDisk.Name
```

建立快照集組態。 

 ```powershell
$snapshotConfig =  New-AzureRmSnapshotConfig `
   -SourceUri $disk.Id `
   -OsType Windows `
   -CreateOption Copy `
   -Location $location 
```

建立快照集。

```powershell
$snapShot = New-AzureRmSnapshot `
   -Snapshot $snapshotConfig `
   -SnapshotName $snapshotName `
   -ResourceGroupName $resourceGroupName
```


如果您計劃使用快照集來建立必須是高效能的 VM，請在 New-AzureRmSnapshot 命令中使用參數 `-AccountType Premium_LRS`。 此參數建立的快照集會儲存為進階受控磁碟。 進階受控磁碟的價格高於「標準」受控磁碟。 因此，使用該參數之前，請確定您真的需要「進階」受控磁碟。

### <a name="create-a-new-disk-from-the-snapshot"></a>從快照集建立新的磁碟

使用 [New-AzureRMDisk](/powershell/module/azurerm.compute/new-azurermdisk)，從快照集建立受控磁碟。 這個範例會使用 *myOSDisk* 作為磁碟名稱。

建立新 VM 的新資源群組。

```powershell
$destinationResourceGroup = 'myDestinationResourceGroup'
New-AzureRmResourceGroup -Location $location `
   -Name $destinationResourceGroup
```

設定 OS 磁碟名稱。 

```powershell
$osDiskName = 'myOsDisk'
```

建立受控磁碟。

```powershell
$osDisk = New-AzureRmDisk -DiskName $osDiskName -Disk `
    (New-AzureRmDiskConfig  -Location $location -CreateOption Copy `
    -SourceResourceId $snapshot.Id) `
    -ResourceGroupName $destinationResourceGroup
```


## <a name="create-the-new-vm"></a>建立新 VM 

建立新 VM 所要使用的網路和其他 VM 資源。

### <a name="create-the-subnet-and-vnet"></a>建立子網路和 VNet

建立 [虛擬網路](../../virtual-network/virtual-networks-overview.md)的 vNet 和 subNet。

建立子網路。 此範例會在資源群組 **myDestinationResourceGroup** 中建立名為 **mySubnet** 的子網路，並將子網路位址首碼設定為 **10.0.0.0/24**。
   
```powershell
$subnetName = 'mySubNet'
$singleSubnet = New-AzureRmVirtualNetworkSubnetConfig `
   -Name $subnetName `
   -AddressPrefix 10.0.0.0/24
```

建立 vNet。 此範例會將虛擬網路名稱設定為 **myVnetName**、位置設定為 **美國西部**，及虛擬網路的位址首碼設定為 **10.0.0.0/16**。 
   
```powershell
$vnetName = "myVnetName"
$vnet = New-AzureRmVirtualNetwork `
   -Name $vnetName -ResourceGroupName $destinationResourceGroup `
   -Location $location `
   -AddressPrefix 10.0.0.0/16 `
   -Subnet $singleSubnet
```    


### <a name="create-the-network-security-group-and-an-rdp-rule"></a>建立網路安全性群組和 RDP 規則
若要能夠使用 RDP 登入 VM，您必須有可在連接埠 3389 上允許 RDP 存取的安全性規則。 因為新 VM 的 VHD 是建立自現有的特製化 VM，您可以使用來自 RDP 來源虛擬機器的帳戶。

此範例會將 NSG 名稱設定為 **myNsg** 以及將 RDP 規則名稱設定為 **myRdpRule**。

```powershell
$nsgName = "myNsg"

$rdpRule = New-AzureRmNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
    -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
    -SourceAddressPrefix Internet -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange 3389
$nsg = New-AzureRmNetworkSecurityGroup `
   -ResourceGroupName $destinationResourceGroup `
   -Location $location `
   -Name $nsgName -SecurityRules $rdpRule
    
```

如需端點和 NSG 規則的詳細資訊，請參閱[使用 PowerShell 對 Azure 中的 VM 開啟連接埠](nsg-quickstart-powershell.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

### <a name="create-a-public-ip-address-and-nic"></a>建立公用 IP 位址和 NIC
若要能夠與虛擬網路中的虛擬機器進行通訊，您需要 [公用 IP 位址](../../virtual-network/virtual-network-ip-addresses-overview-arm.md) 和網路介面。

建立公用 IP。 在此範例中，公用 IP 位址名稱會設定為 **myIP**。
   
```powershell
$ipName = "myIP"
$pip = New-AzureRmPublicIpAddress `
   -Name $ipName -ResourceGroupName $destinationResourceGroup `
   -Location $location `
   -AllocationMethod Dynamic
```       

建立 NIC。 在此範例中，NIC 名稱會設定為 **myNicName**。
   
```powershell
$nicName = "myNicName"
$nic = New-AzureRmNetworkInterface -Name $nicName `
   -ResourceGroupName $destinationResourceGroup `
   -Location $location -SubnetId $vnet.Subnets[0].Id `
   -PublicIpAddressId $pip.Id `
   -NetworkSecurityGroupId $nsg.Id
```



### <a name="set-the-vm-name-and-size"></a>設定 VM 名稱和大小

此範例將 VM 名稱設定為 myVM，將 VM 大小設定為 Standard_A2。

```powershell
$vmName = "myVM"
$vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize "Standard_A2"
```

### <a name="add-the-nic"></a>新增 NIC
    
```powershell
$vm = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic.Id
```
    

### <a name="add-the-os-disk"></a>新增 OS 磁碟 

使用 [Set-AzureRmVMOSDisk](/powershell/module/azurerm.compute/set-azurermvmosdisk) 將 OS 磁碟新增至組態。 這個範例將磁碟大小設定為 *128GB*，並附加受控磁碟作為 *Windows* OS 磁碟。
 
```powershell
$vm = Set-AzureRmVMOSDisk -VM $vm -ManagedDiskId $osDisk.Id -StorageAccountType Standard_LRS `
    -DiskSizeInGB 128 -CreateOption Attach -Windows
```

### <a name="complete-the-vm"></a>完成 VM 

使用 [New-AzureRMVM](/powershell/module/azurerm.compute/new-azurermvm) 以及我們剛才建立的組態建立 VM。

```powershell
New-AzureRmVM -ResourceGroupName $destinationResourceGroup -Location $location -VM $vm
```

如果此命令成功，您會看到如下的輸出︰

```powershell
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK   

```

### <a name="verify-that-the-vm-was-created"></a>確認已建立 VM
從 [Azure 入口網站](https://portal.azure.com)的 [瀏覽] > [虛擬機器]下，或是使用下列 PowerShell 命令，應可看到新建立的 VM：

```powershell
$vmList = Get-AzureRmVM -ResourceGroupName $destinationResourceGroup
$vmList.Name
```

## <a name="next-steps"></a>後續步驟
登入至新的虛擬機器。 如需詳細資訊，請參閱 [如何連接和登入執行 Windows 的 Azure 虛擬機器](connect-logon.md)。

