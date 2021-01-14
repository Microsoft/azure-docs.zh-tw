---
title: 從 Azure 中的特製化 VHD 建立 Windows VM
description: 使用 Resource Manager 部署模型，藉由連結特製化受控磁碟作為 OS 磁碟，建立新的 Windows VM。
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 10/10/2019
ms.author: cynthn
ms.openlocfilehash: 577b8527d95a6410deca8002cddfaae31be1fc97
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98197484"
---
# <a name="create-a-windows-vm-from-a-specialized-disk-by-using-powershell"></a>使用 PowerShell 從特製化磁碟建立 Windows VM

連結特製化受控磁碟作為 OS 磁碟，以建立新的虛擬機器。 特製化磁碟是來自現有 VM 的虛擬硬碟 (VHD) 複本，其中包含來自您原始 VM 的使用者帳戶、應用程式及其他狀態資料。 

您有幾種選項：
* [使用現有受控磁碟](#option-1-use-an-existing-disk)。 如果您有無法正常運作的 VM，此選項相當有用。 您可以刪除 VM，然後重複使用受控磁碟來建立新的 VM。 
* [上傳 VHD](#option-2-upload-a-specialized-vhd) 
* [使用快照集來複製現有的 Azure VM](#option-3-copy-an-existing-azure-vm)

您也可以使用 Azure 入口網站，[從特定的 VHD 建立新的虛擬機器](create-vm-specialized-portal.md)。

本文說明如何使用受控磁碟。 如果您有需要使用儲存體帳戶的舊版部署，請參閱[從儲存體帳戶中的特製化 VHD 建立 VM](/previous-versions/azure/virtual-machines/windows/sa-create-vm-specialized)。

> [!IMPORTANT]
> 
> 當您使用特製化磁片來建立新的 VM 時，新的 VM 會保留原始 VM 的電腦名稱稱。 其他電腦特定的資訊 (例如，CMID) 也會保留下來，而在某些情況下，這項重複的資訊可能會造成問題。 複製 VM 時，請留意您的應用程式會依賴哪些類型的電腦特定資訊。  
> 因此，如果您想要建立多個 Vm，請不要使用特製化磁片。 然而，對於較大型的部署，請[建立映像](capture-image-resource.md)，然後[使用該映像來建立多個 VM](create-vm-generalized-managed.md)。

我們建議您將來自單一 VHD 或快照集的並行部署數目限制為 20 部 VM。 

## <a name="option-1-use-an-existing-disk"></a>選項 1：使用現有的磁碟

如果您已刪除虛擬機器，而想要重複使用 OS 磁碟建立新的虛擬機器，請使用 [Get-AzDisk](/powershell/module/az.compute/get-azdisk)。

```powershell
$resourceGroupName = 'myResourceGroup'
$osDiskName = 'myOsDisk'
$osDisk = Get-AzDisk `
-ResourceGroupName $resourceGroupName `
-DiskName $osDiskName
```
現在，您可以將此磁碟當作 OS 磁碟連結至[新的虛擬機器](#create-the-new-vm)。

## <a name="option-2-upload-a-specialized-vhd"></a>選項 2：上傳特製化 VHD

您可以上傳從特製化的 VM，以在內部部署虛擬化工具，像是從另一個雲端匯出 HYPER-V 或在 VM 建立 VHD。

### <a name="prepare-the-vm"></a>準備 VM
依現況使用 VHD 來建立新 VM。 
  
  * [準備要上傳至 Azure 的 Windows VHD](prepare-for-upload-vhd-image.md). 請 **勿** 使用 Sysprep 將 VM一般化。
  * 移除已安裝在 VM 上的所有客體虛擬化工具和代理程式 (例如 VMware 工具)。
  * 確認已將 VM 設定成從 DHCP 取得 IP 位址和 DNS 設定。 這可確保伺服器在啟動時取得虛擬網路內的 IP 位址。 


### <a name="upload-the-vhd"></a>上傳 VHD

您現在可以直接將 VHD 上傳至受控磁碟。 如需相關指示，請參閱[使用 Azure PowerShell 將 VHD 上傳至 Azure](disks-upload-vhd-to-managed-disk-powershell.md)。

## <a name="option-3-copy-an-existing-azure-vm"></a>選項 3：複製現有的 Azure VM

您可以藉由建立 VM 的快照集，然後使用該快照集來建立新的受控磁碟和新的 VM，建立使用受控磁碟的 VM 複本。

如果您想要將現有的 VM 複製到另一個區域，您可以使用 azcopy，[在另一個區域中建立磁碟複本](disks-upload-vhd-to-managed-disk-powershell.md#copy-a-managed-disk)。 

### <a name="take-a-snapshot-of-the-os-disk"></a>製作 OS 磁碟的快照集

您可以建立整個 VM (包含所有磁碟) 的快照集或僅包含單一磁碟的快照集。 下列步驟說明如何使用 [New-AzSnapshot](/powershell/module/az.compute/new-azsnapshot) Cmdlet 來建立僅包含您 VM 之 OS 磁碟的快照集。 

首先，設定一些參數。 

 ```powershell
$resourceGroupName = 'myResourceGroup' 
$vmName = 'myVM'
$location = 'westus' 
$snapshotName = 'mySnapshot'  
```

取得 VM 物件。

```powershell
$vm = Get-AzVM -Name $vmName `
   -ResourceGroupName $resourceGroupName
```
取得 OS 磁碟名稱。

 ```powershell
$disk = Get-AzDisk -ResourceGroupName $resourceGroupName `
   -DiskName $vm.StorageProfile.OsDisk.Name
```

建立快照集組態。 

 ```powershell
$snapshotConfig =  New-AzSnapshotConfig `
   -SourceUri $disk.Id `
   -OsType Windows `
   -CreateOption Copy `
   -Location $location 
```

建立快照集。

```powershell
$snapShot = New-AzSnapshot `
   -Snapshot $snapshotConfig `
   -SnapshotName $snapshotName `
   -ResourceGroupName $resourceGroupName
```


若要使用此快照集來建立必須具備高效能的 VM，請將 `-AccountType Premium_LRS` 參數新增至 New-AzSnapshotConfig 命令。 此參數建立的快照集會儲存為「進階受控磁碟」。 「進階受控磁碟」比「標準磁碟」費用高，因此請先確定您需要「進階磁碟」，再使用此參數。

### <a name="create-a-new-disk-from-the-snapshot"></a>從快照集建立新的磁碟

請使用 [New-AzDisk](/powershell/module/az.compute/new-azdisk)，從快照集建立受控磁碟。 這個範例會使用 *myOSDisk* 作為磁碟名稱。

建立新 VM 的新資源群組。

```powershell
$destinationResourceGroup = 'myDestinationResourceGroup'
New-AzResourceGroup -Location $location `
   -Name $destinationResourceGroup
```

設定 OS 磁碟名稱。 

```powershell
$osDiskName = 'myOsDisk'
```

建立受控磁碟。

```powershell
$osDisk = New-AzDisk -DiskName $osDiskName -Disk `
    (New-AzDiskConfig  -Location $location -CreateOption Copy `
    -SourceResourceId $snapshot.Id) `
    -ResourceGroupName $destinationResourceGroup
```


## <a name="create-the-new-vm"></a>建立新 VM 

建立新 VM 所要使用的網路和其他 VM 資源。

### <a name="create-the-subnet-and-virtual-network"></a>建立子網路和虛擬網路

建立[虛擬網路](../../virtual-network/virtual-networks-overview.md)和 VM 的子網路。

1. 建立子網路。 此範例會在資源群組 *myDestinationResourceGroup* 中建立名為 *mySubnet* 的子網路，並將子網路位址首碼設定為 *10.0.0.0/24*。
   
    ```powershell
    $subnetName = 'mySubNet'
    $singleSubnet = New-AzVirtualNetworkSubnetConfig `
       -Name $subnetName `
       -AddressPrefix 10.0.0.0/24
    ```
    
2. 建立虛擬網路 此範例會將虛擬網路名稱設定為 *myVnetName*、將位置設定為 *美國西部*，以及將虛擬網路的位址首碼設定為 *10.0.0.0/16*。 
   
    ```powershell
    $vnetName = "myVnetName"
    $vnet = New-AzVirtualNetwork `
       -Name $vnetName -ResourceGroupName $destinationResourceGroup `
       -Location $location `
       -AddressPrefix 10.0.0.0/16 `
       -Subnet $singleSubnet
    ```    
    

### <a name="create-the-network-security-group-and-an-rdp-rule"></a>建立網路安全性群組和 RDP 規則
為了能夠使用遠端桌面通訊協定 (RDP) 來登入 VM，您將需要有可在連接埠 3389 上允許 RDP 存取的安全性規則。 在我們的範例中，新 VM 的 VHD 是建立自現有的特製化 VM，因此您可以針對 RDP 使用存在於來源虛擬機器上的的帳戶。

此範例會將網路安全性群組 (NSG) 名稱設定為 *myNsg*，以及將 RDP 規則名稱設定為 *myRdpRule*。

```powershell
$nsgName = "myNsg"

$rdpRule = New-AzNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
    -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
    -SourceAddressPrefix Internet -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange 3389
$nsg = New-AzNetworkSecurityGroup `
   -ResourceGroupName $destinationResourceGroup `
   -Location $location `
   -Name $nsgName -SecurityRules $rdpRule
    
```

如需有關端點和 NSG 規則的詳細資訊，請參閱[使用 PowerShell 對 Azure 中的 VM 開啟連接埠](nsg-quickstart-powershell.md)。

### <a name="create-a-public-ip-address-and-nic"></a>建立公用 IP 位址和 NIC
若要能夠與虛擬網路中的虛擬機器進行通訊，您將需要[公用 IP 位址](../../virtual-network/public-ip-addresses.md)和網路介面。

1. 建立公用 IP。 在此範例中，公用 IP 位址名稱會設定為 *myIP*。
   
    ```powershell
    $ipName = "myIP"
    $pip = New-AzPublicIpAddress `
       -Name $ipName -ResourceGroupName $destinationResourceGroup `
       -Location $location `
       -AllocationMethod Dynamic
    ```       
    
2. 建立 NIC。 在此範例中，NIC 名稱會設定為 *myNicName*。
   
    ```powershell
    $nicName = "myNicName"
    $nic = New-AzNetworkInterface -Name $nicName `
       -ResourceGroupName $destinationResourceGroup `
       -Location $location -SubnetId $vnet.Subnets[0].Id `
       -PublicIpAddressId $pip.Id `
       -NetworkSecurityGroupId $nsg.Id
    ```
    


### <a name="set-the-vm-name-and-size"></a>設定 VM 名稱和大小

此範例將 VM 名稱設定為 myVM，將 VM 大小設定為 Standard_A2。

```powershell
$vmName = "myVM"
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize "Standard_A2"
```

### <a name="add-the-nic"></a>新增 NIC
    
```powershell
$vm = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id
```
    

### <a name="add-the-os-disk"></a>新增 OS 磁碟 

請使用 [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk) 將 OS 磁碟新增至組態。 這個範例將磁碟大小設定為 *128GB*，並附加受控磁碟作為 *Windows* OS 磁碟。
 
```powershell
$vm = Set-AzVMOSDisk -VM $vm -ManagedDiskId $osDisk.Id -StorageAccountType Standard_LRS `
    -DiskSizeInGB 128 -CreateOption Attach -Windows
```

### <a name="complete-the-vm"></a>完成 VM 

請使用 [New-AzVM](/powershell/module/az.compute/new-azvm) 搭配剛才建立的組態來建立 VM。

```powershell
New-AzVM -ResourceGroupName $destinationResourceGroup -Location $location -VM $vm
```

如果此命令成功，您將看到類似以下的輸出︰

```powershell
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK   

```

### <a name="verify-that-the-vm-was-created"></a>確認已建立 VM
您應該可在 [Azure 入口網站](https://portal.azure.com)的 [瀏覽] > [虛擬機器] 下，或透過使用下列 PowerShell 命令，看到新建立的 VM。

```powershell
$vmList = Get-AzVM -ResourceGroupName $destinationResourceGroup
$vmList.Name
```

## <a name="next-steps"></a>後續步驟
登入至新的虛擬機器。 如需詳細資訊，請參閱 [如何連接和登入執行 Windows 的 Azure 虛擬機器](connect-logon.md)。