---
title: 在 Azure 中從特製化磁碟建立 VM | Microsoft Docs
description: 藉由連結特製化非受控磁碟，在 Resource Manager 部署模型中建立新的 VM。
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
ms.date: 05/23/2017
ms.author: cynthn
ROBOTS: NOINDEX
ms.openlocfilehash: da1fa2b182888e623f8df734c9119e208433e2bd
ms.sourcegitcommit: 909469bf17211be40ea24a981c3e0331ea182996
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "34012704"
---
# <a name="create-a-vm-from-a-specialized-vhd-in-a-storage-account"></a>從儲存體帳戶中的特製化 VHD 建立 VM

藉由使用 Powershell 連結特製化非受控磁碟作為 OS 磁碟來建立新的 VM。 特製化磁碟是現有 VM 中的 VHD 複本，可從原始的 VM 維護使用者帳戶、應用程式和其他狀態資料。 

您有兩個選擇：
* [上傳 VHD](sa-create-vm-specialized.md#option-1-upload-a-specialized-vhd)
* [複製現有 Azure VM 的 VHD？](sa-create-vm-specialized.md#option-2-copy-an-existing-azure-vm)

## <a name="before-you-begin"></a>開始之前
如果您使用 PowerShell，請確定您擁有最新版的 AzureRM.Compute PowerShell 模組。 執行下列命令來安裝它。

```powershell
Install-Module AzureRM.Compute 
```
如需詳細資訊，請參閱 [Azure PowerShell 版本控制](/powershell/azure/overview)。


## <a name="option-1-upload-a-specialized-vhd"></a>選項 1：上傳特製化 VHD

您可以上傳從特製化的 VM，以在內部部署虛擬化工具，像是從另一個雲端匯出 HYPER-V 或在 VM 建立 VHD。

### <a name="prepare-the-vm"></a>準備 VM
您可以上傳使用內部部署 VM 建立的特製化 VHD，或上傳從另一個雲端匯出的 VHD。 特製化的 VHD 會從原始的 VM 維護使用者帳戶、應用程式和其他狀態資料。 如果您想要使用 VHD 現狀建立新的 VM，請確定完成下列步驟。 
  
  * [準備要上傳至 Azure 的 Windows VHD](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). **不要**使用 Sysprep 一般化 VM。
  * 移除任何 VM 上 (也就是 VMware 工具) 已安裝的來賓虛擬化工具和代理程式。
  * 確認已透過 DHCP 設定 VM 提取其 IP 位址和 DNS 設定。 這可確保伺服器在啟動時取得 VNet 內的 IP 位址。 


### <a name="get-the-storage-account"></a>取得儲存體帳戶
您需要一個 Azure 中的儲存體帳戶來裝載上傳的 VM 映像。 您可以使用現有的儲存體帳戶或建立新帳戶。 

若要顯示可用的儲存體帳戶，請輸入︰

```powershell
Get-AzureRmStorageAccount
```

如果您想要使用現有的儲存體帳戶，請移至[上傳 VM 映像](#upload-the-vm-vhd-to-your-storage-account)一節。

如果您需要建立儲存體帳戶，請依照下列步驟操作：

1. 您需要在當中建立儲存體帳戶的資源群組名稱。 若要找出您訂用帳戶中的所有資源群組，請輸入︰
   
    ```powershell
    Get-AzureRmResourceGroup
    ```

    若要在**美國中部**區域建立名為 **myResourceGroup** 的資源群組，請輸入︰

    ```powershell
    New-AzureRmResourceGroup -Name myResourceGroup -Location "West US"
    ```

2. 使用 [New-AzureRmStorageAccount](/powershell/module/azurerm.storage/new-azurermstorageaccount) Cmdlet，在此資源群組中建立名為 **mystorageaccount**的儲存體帳戶：
   
    ```powershell
    New-AzureRmStorageAccount -ResourceGroupName myResourceGroup -Name mystorageaccount -Location "West US" `
        -SkuName "Standard_LRS" -Kind "Storage"
    ```
   
### <a name="upload-the-vhd-to-your-storage-account"></a>將 VHD 上傳至儲存體帳戶
使用 [Add-AzureRmVhd](/powershell/module/azurerm.compute/add-azurermvhd) Cmdlet，將映像上傳到儲存體帳戶中的容器。 這個範例會將檔案 **myVHD.vhd** 從 `"C:\Users\Public\Documents\Virtual hard disks\"` 上傳至 **myResourceGroup** 資源群組中名為 **mystorageaccount** 的儲存體帳戶。 檔案會放入名為 **mycontainer** 的容器，新的檔案名稱會是 **myUploadedVHD.vhd**。

```powershell
$rgName = "myResourceGroup"
$urlOfUploadedImageVhd = "https://mystorageaccount.blob.core.windows.net/mycontainer/myUploadedVHD.vhd"
Add-AzureRmVhd -ResourceGroupName $rgName -Destination $urlOfUploadedImageVhd `
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

視您的網路連線和 VHD 檔案大小而定，此命令可能需要一些時間才能完成。


## <a name="option-2-copy-the-vhd-from-an-existing-azure-vm"></a>選項 2：從現有 Azure VM 複製 VHD

您可以將 VHD 複製到另一個儲存體帳戶，以在建立新的重複 VM 時使用。

### <a name="before-you-begin"></a>開始之前
請確定您︰

* 已取得**來源和目的地儲存體帳戶**的相關資訊。 針對來源 VM，您需要有儲存體帳戶和容器名稱。 容器名稱通常會是 **vhd**。 您也需要有一個目的地儲存體帳戶。 如果您還沒有，可以使用入口網站 ([所有服務] > [儲存體帳戶] > [新增])，或使用 [New-AzureRmStorageAccount](/powershell/module/azurerm.storage/new-azurermstorageaccount) Cmdlet 建立一個。 
* 已下載並安裝 [AzCopy 工具](../../storage/common/storage-use-azcopy.md)。 

### <a name="deallocate-the-vm"></a>解除配置 VM
解除配置 VM，這會釋出要複製的 VHD。 

* **入口網站**︰ 按一下 [虛擬機器]  >  [myVM] > [停止]
* **Powershell**：使用 [Stop-AzureRmVM](/powershell/module/azurerm.compute/stop-azurermvm) 來停止 (解除配置) **myResourceGroup** 資源群組中名為 **myVM** 的 VM。

```powershell
Stop-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM
```

Azure 入口網站中 VM 的 [狀態] 會從 [已停止] 變更為 [已停止 (已解除配置)]。

### <a name="get-the-storage-account-urls"></a>取得儲存體帳戶 URL
您需要來源和目的地儲存體帳戶的 URL。 URL 看起來像是：`https://<storageaccount>.blob.core.windows.net/<containerName>/`。 如果您已經知道儲存體帳戶和容器名稱，可以直接取代方括號中的資訊來建立您的 URL。 

您可以使用 Azure 入口網站或 Azure PowerShell 來取得 URL：

* **入口網站**︰按一下 **>**[所有服務] > [儲存體帳戶] > *儲存體帳戶* > [Blob]，而您的來源 VHD 檔可能在 **vhds** 容器中。 按一下容器的 [屬性]，複製標示為[URL] 的文字。 您會需要來源和目的地容器的 URL。 
* **Powershell**：使用 [Get-AzureRmVM](/powershell/module/azurerm.compute/get-azurermvm) 來取得 **myResourceGroup** 資源群組中名為 **myVM** 之 VM 的資訊。 在結果中，查看 [儲存體設定檔] 區段的 [VHD URI]。 URI 的第一個部分是容器的 URL，最後一個部分是 VM 的作業系統 VHD 名稱。

```powershell
Get-AzureRmVM -ResourceGroupName "myResourceGroup" -Name "myVM"
``` 

## <a name="get-the-storage-access-keys"></a>取得儲存體存取金鑰
找出來源和目的地儲存體帳戶的存取金鑰。 如需存取金鑰的詳細資訊，請參閱 [關於 Azure 儲存體帳戶](../../storage/common/storage-create-storage-account.md)。

* **入口網站**︰按一下 [所有服務]  > [儲存體帳戶]  > [儲存體帳戶] > [存取金鑰]。 複製標示為 [金鑰1] 的金鑰。
* **Powershell**：使用 [Get-AzureRmStorageAccountKey](/powershell/module/azurerm.storage/get-azurermstorageaccountkey) 來取得 **myResourceGroup** 資源群組中 **mystorageaccount** 儲存體帳戶的儲存體金鑰。 複製標示 [金鑰1] 的金鑰。

```powershell
Get-AzureRmStorageAccountKey -Name mystorageaccount -ResourceGroupName myResourceGroup
```

### <a name="copy-the-vhd"></a>複製 VHD
您可以使用 AzCopy 在儲存體帳戶之間複製檔案。 針對目的地容器，如果指定的容器不存在，它會為您建立。 

若要使用 AzCopy，在您的本機電腦上開啟命令提示字元，瀏覽至安裝 AzCopy 的資料夾。 它會類似 C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy。 

若要複製容器內的所有檔案，使用 **/S** 選項。 這可以用來複製作業系統 VHD 及其所有的資料磁碟 (如果它們是在相同容器中)。 這個範例示範如何將 **mysourcestorageaccount** 儲存體帳戶中**mysourcecontainer** 容器中的所有檔案，複製到 **mydestinationstorageaccount** 儲存體帳戶中的 **mydestinationcontainer**。 儲存體帳戶和容器的名稱請取代為您自己的。 使用您自己的金鑰取代 `<sourceStorageAccountKey1>` 和 `<destinationStorageAccountKey1>`。

```
AzCopy /Source:https://mysourcestorageaccount.blob.core.windows.net/mysourcecontainer `
    /Dest:https://mydestinationatorageaccount.blob.core.windows.net/mydestinationcontainer `
    /SourceKey:<sourceStorageAccountKey1> /DestKey:<destinationStorageAccountKey1> /S
```

如果要複製包含多個檔案之容器中的特定 VHD，也可以使用 /Pattern 選項指定檔案名稱。 在此範例中，只會複製名為 **myFileName.vhd** 的檔案。

```
AzCopy /Source:https://mysourcestorageaccount.blob.core.windows.net/mysourcecontainer `
  /Dest:https://mydestinationatorageaccount.blob.core.windows.net/mydestinationcontainer `
  /SourceKey:<sourceStorageAccountKey1> /DestKey:<destinationStorageAccountKey1> `
  /Pattern:myFileName.vhd
```


完成時，您將收到如下的訊息：

```
Finished 2 of total 2 file(s).
[2016/10/07 17:37:41] Transfer summary:
-----------------
Total files transferred: 2
Transfer successfully:   2
Transfer skipped:        0
Transfer failed:         0
Elapsed time:            00.00:13:07
```

### <a name="troubleshooting"></a>疑難排解
* 當您使用 AZCopy 時，如果您看到「伺服器無法驗證要求」錯誤，請確定授權標頭值的格式正確 (包含簽章)。 如果您正在使用金鑰 2 或次要儲存體金鑰，請嘗試使用主要或第 1 個儲存體金鑰。

## <a name="create-the-new-vm"></a>建立新 VM 

您必須建立新的 VM 所要使用的網路和其他 VM 資源。

### <a name="create-the-subnet-and-vnet"></a>建立子網路和 VNet

建立 [虛擬網路](../../virtual-network/virtual-networks-overview.md)的 vNet 和 subNet。

1. 建立子網路。 此範例會在資源群組 **myResourceGroup** 中建立名為 **mySubnet** 的子網路，並將子網路位址首碼設定為 **10.0.0.0/24**。
   
    ```powershell
    $rgName = "myResourceGroup"
    $subnetName = "mySubNet"
    $singleSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24
    ```
2. 建立 vNet。 此範例會將虛擬網路名稱設定為 **myVnetName**、位置設定為 **美國西部**，及虛擬網路的位址首碼設定為 **10.0.0.0/16**。 
   
    ```powershell
    $location = "West US"
    $vnetName = "myVnetName"
    $vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location `
        -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet
    ```    
### <a name="create-the-network-security-group-and-an-rdp-rule"></a>建立網路安全性群組和 RDP 規則
若要能夠使用 RDP 登入 VM，您必須有可在連接埠 3389 上允許 RDP 存取的安全性規則。 因為新 VM 的 VHD 是從現有的特製化 VM 所建立，在建立 VM 之後，您可以從具有使用 RDP 之登入權限的來源虛擬機器使用現有帳戶。
這需要在建立相關聯網路介面之前完成。  
此範例會將 NSG 名稱設定為 **myNsg** 以及將 RDP 規則名稱設定為 **myRdpRule**。

```powershell
$nsgName = "myNsg"

$rdpRule = New-AzureRmNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
    -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
    -SourceAddressPrefix Internet -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange 3389
$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $rgName -Location $location `
    -Name $nsgName -SecurityRules $rdpRule
    
```

如需端點和 NSG 規則的詳細資訊，請參閱[使用 PowerShell 對 Azure 中的 VM 開啟連接埠](nsg-quickstart-powershell.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

### <a name="create-a-public-ip-address-and-nic"></a>建立公用 IP 位址和 NIC
若要能夠與虛擬網路中的虛擬機器進行通訊，您需要 [公用 IP 位址](../../virtual-network/virtual-network-ip-addresses-overview-arm.md) 和網路介面。

1. 建立公用 IP。 在此範例中，公用 IP 位址名稱會設定為 **myIP**。
   
    ```powershell
    $ipName = "myIP"
    $pip = New-AzureRmPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $location `
        -AllocationMethod Dynamic
    ```       
2. 建立 NIC。 在此範例中，NIC 名稱會設定為 **myNicName**。 這個步驟也會讓稍早建立的網路安全性群組與此 NIC 相關聯。
   
    ```powershell
    $nicName = "myNicName"
    $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName `
    -Location $location -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id
    ```

### <a name="set-the-vm-name-and-size"></a>設定 VM 名稱和大小

此範例將 VM 名稱設定為 "myVM"，將 VM 大小設定為 "Standard_A2"。
```powershell
$vmName = "myVM"
$vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize "Standard_A2"
```

### <a name="add-the-nic"></a>新增 NIC
    
```powershell
$vm = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic.Id
```
    
    
### <a name="configure-the-os-disk"></a>設定 OS 磁碟

1. 為您所上傳或複製的 VHD 設定 URI。 在此範例中，VHD 檔案 **myOsDisk.vhd** 保存在容器 **myContainer** 中的儲存體帳戶 **myStorageAccount**。

    ```powershell
    $osDiskUri = "https://myStorageAccount.blob.core.windows.net/myContainer/myOsDisk.vhd"
    ```
2. 新增 OS 磁碟。 在此範例中，建立 OS 磁碟時，"osDisk" 一詞會附加至 VM 名稱而形成 OS 磁碟名稱。 這個範例也指定這個以 Windows 為基礎的 VHD，應該附加至 VM 作為 OS 磁碟。
    
    ```powershell
    $osDiskName = $vmName + "osDisk"
    $vm = Set-AzureRmVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri -CreateOption attach -Windows
    ```

選擇性︰如果您有資料磁碟需要附加至 VM，請使用資料 VHD 的 URL 和適當的邏輯單元編號 (Lun) 新增資料磁碟。

```powershell
$dataDiskName = $vmName + "dataDisk"
$vm = Add-AzureRmVMDataDisk -VM $vm -Name $dataDiskName -VhdUri $dataDiskUri -Lun 1 -CreateOption attach
```

使用儲存體帳戶時，資料和作業系統磁碟的 URL 近似於︰`https://StorageAccountName.blob.core.windows.net/BlobContainerName/DiskName.vhd`。 您可以藉由下列方法在入口網站上找到它：瀏覽至目標儲存體容器，按一下複製的作業系統或資料 VHD，然後複製 URL 的內容。


### <a name="complete-the-vm"></a>完成 VM 

使用我們剛才建立的組態建立 VM。

```powershell
#Create the new VM
New-AzureRmVM -ResourceGroupName $rgName -Location $location -VM $vm
```

如果此命令成功，您會看到如下的輸出︰

```powershell
RequestId IsSuccessStatusCode StatusCode ReasonPhrase
--------- ------------------- ---------- ------------
                         True         OK OK   

```

### <a name="verify-that-the-vm-was-created"></a>確認已建立 VM
從 [Azure 入口網站](https://portal.azure.com)的 [所有服務] > [虛擬機器] 下，或是使用下列 PowerShell 命令，應可看到新建立的 VM：

```powershell
$vmList = Get-AzureRmVM -ResourceGroupName $rgName
$vmList.Name
```

## <a name="next-steps"></a>後續步驟
登入至新的虛擬機器。 如需詳細資訊，請參閱 [如何連接和登入執行 Windows 的 Azure 虛擬機器](connect-logon.md)。

