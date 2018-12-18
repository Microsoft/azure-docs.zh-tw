---
title: 上傳一般化的 VHD 以在 Azure 中建立多部 VM| Microsoft Docs
description: 將一般化 VHD 上傳至 Azure 儲存體帳戶，建立搭配 Resource Manager 部署模型使用的 Windows VM。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 05/18/2017
ms.author: cynthn
ROBOTS: NOINDEX
ms.openlocfilehash: be2ec6df33f5756dc080195bfad32e0c9079453c
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/19/2018
ms.locfileid: "31603195"
---
# <a name="upload-a-generalized-vhd-to-azure-to-create-a-new-vm"></a>將一般化 VHD 上傳至 Azure，以建立新的 VM。

本主題涵蓋將一般化的非受控磁碟上傳至儲存體帳戶，然後建立使用上傳磁碟的新 VM。 一般化 VHD 映像已使用 Sysprep 移除您所有的個人帳戶資訊。 

如果您想要從儲存體帳戶中的特殊化 VHD 建立 VM，請參閱[從特殊化 VHD 建立 VM](sa-create-vm-specialized.md)。

本主題涵蓋使用儲存體帳戶的內容，但我們建議客戶改用受控磁碟。 如需如何使用受控磁碟來準備、上傳及建立新 VM 的完整逐步解說，請參閱[使用受控磁碟從上傳至 Azure 的一般化 VHD 建立新的 VM](upload-generalized-managed.md)。



## <a name="prepare-the-vm"></a>準備 VM

一般化 VHD - 已使用 Sysprep 移除您所有的個人帳戶資訊。 如果您想要使用 VHD 作為建立新 VM 映像的來源，您應該︰
  
  * [準備要上傳至 Azure 的 Windows VHD](prepare-for-upload-vhd-image.md). 
  * 使用 Sysprep 一般化虛擬機器

### <a name="generalize-a-windows-virtual-machine-using-sysprep"></a>使用 Sysprep 將 Windows 虛擬機器一般化
本節說明如何將 Windows 虛擬機器一般化以做為映像。 Sysprep 會移除您的所有個人帳戶資訊以及其他項目，並準備電腦以做為映像。 如需 Sysprep 的詳細資訊，請參閱 [如何使用 Sysprep：簡介](http://technet.microsoft.com/library/bb457073.aspx)。

請確定 Sysprep 支援電腦上執行的伺服器角色。 如需詳細資訊，請參閱 [Sysprep Support for Server Roles (伺服器角色的 Sysprep 支援)](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)

> [!IMPORTANT]
> 如果您是第一次在將 VHD 上傳至 Azure 之前執行 Sysprep，請確定您已[準備好 VM](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 再執行 Sysprep。 
> 
> 

1. 登入 Windows 虛擬機器。
2. 以系統管理員身分開啟 [命令提示字元] 視窗。 切換至 **%windir%\system32\sysprep** 目錄，然後執行 `sysprep.exe`。
3. 在 [系統準備工具] 對話方塊中，選取 [進入系統全新體驗 (OOBE)]，並確認已勾選 [一般化] 核取方塊。
4. 在 [關機選項] 中選取 [關機]。
5. 按一下 [SERVICEPRINCIPAL] 。
   
    ![啟動 Sysprep](./media/upload-generalized-managed/sysprepgeneral.png)
6. Sysprep 完成時，會關閉虛擬機器。 

> [!IMPORTANT]
> 在您完成將 VHD 上傳到 Azure 或從 VM 建立映像之前，請勿重新啟動 VM。 如果 VM 意外重新啟動，請執行 Sysprep 來重新將它一般化。
> 
> 


## <a name="upload-the-vhd"></a>上傳 VHD

將 VHD 上傳到 Azure 儲存體帳戶。

### <a name="log-in-to-azure"></a>登入 Azure
如果尚未安裝 Azure PowerShell 1.4 版或更高版本，請參閱 [How to install and configure Azure PowerShell (如何安裝和設定 Azure PowerShell)](/powershell/azure/overview)。

1. 開啟 Azure PowerShell，並登入您的 Azure 帳戶。 這會開啟一個可供您輸入 Azure 帳戶認證的快顯視窗。
   
    ```powershell
    Connect-AzureRmAccount
    ```
2. 取得您可用訂用帳戶的訂用帳戶識別碼。
   
    ```powershell
    Get-AzureRmSubscription
    ```
3. 使用訂用帳戶識別碼來設定正確的訂用帳戶。 使用正確訂用帳戶的識別碼取代 `<subscriptionID>`。
   
    ```powershell
    Select-AzureRmSubscription -SubscriptionId "<subscriptionID>"
    ```

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
 
### <a name="start-the-upload"></a>開始上傳 

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


## <a name="create-a-new-vm"></a>建立新的 VM 

您現在可以使用已上傳的 VHD 建立新的 VM。 

### <a name="set-the-uri-of-the-vhd"></a>設定 VHD 的 URI

要使用之 VHD 的 URI 格式如下：https://**mystorageaccount**.blob.core.windows.net/**mycontainer**/**MyVhdName**.vhd。 在此範例中，名為 **myVHD** 的 VHD 位於容器 **mycontainer** 中的儲存體帳戶 **mystorageaccount**。

```powershell
$imageURI = "https://mystorageaccount.blob.core.windows.net/mycontainer/myVhd.vhd"
```


### <a name="create-a-virtual-network"></a>建立虛擬網路
建立[虛擬網路](../../virtual-network/virtual-networks-overview.md)的 vNet 和子網路。

1. 建立子網路。 下列範例會在資源群組 **myResourceGroup** 中建立名為 **mySubnet** 的子網路，而其位址首碼為 **10.0.0.0/24**。  
   
    ```powershell
    $rgName = "myResourceGroup"
    $subnetName = "mySubnet"
    $singleSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24
    ```
2. 建立虛擬網路 下列範例會在**美國西部**位置中建立名為 **myVnet** 的虛擬網路，其位址首碼為 **10.0.0.0/16**。  
   
    ```powershell
    $location = "West US"
    $vnetName = "myVnet"
    $vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location `
        -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet
    ```    

### <a name="create-a-public-ip-address-and-network-interface"></a>建立公用 IP 位址和網路介面
若要能夠與虛擬網路中的虛擬機器進行通訊，您需要 [公用 IP 位址](../../virtual-network/virtual-network-ip-addresses-overview-arm.md) 和網路介面。

1. 建立公用 IP 位址。 此範例會建立名為 **myPip** 的公用 IP 位址。 
   
    ```powershell
    $ipName = "myPip"
    $pip = New-AzureRmPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $location `
        -AllocationMethod Dynamic
    ```       
2. 建立 NIC。 此範例會建立名為 **myNic** 的 NIC。 
   
    ```powershell
    $nicName = "myNic"
    $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $location `
        -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id
    ```

### <a name="create-the-network-security-group-and-an-rdp-rule"></a>建立網路安全性群組和 RDP 規則
若要能夠使用 RDP 登入 VM，您必須有可在連接埠 3389 上允許 RDP 存取的安全性規則。 

此範例會建立名為 **myNsg** 的 NSG，其包含的規則 **myRdpRule** 可允許透過連接埠 3389 的 RDP 流量。 如需 NSG 的詳細資訊，請參閱[使用 PowerShell 對 Azure 中的 VM 開啟連接埠](nsg-quickstart-powershell.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

```powershell
$nsgName = "myNsg"

$rdpRule = New-AzureRmNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
    -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
    -SourceAddressPrefix Internet -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange 3389

$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $rgName -Location $location `
    -Name $nsgName -SecurityRules $rdpRule
```


### <a name="create-a-variable-for-the-virtual-network"></a>建立虛擬網路的變數
為已完成的虛擬網路建立變數。 

```powershell
$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName $rgName -Name $vnetName
```

### <a name="create-the-vm"></a>建立 VM
下列 PowerShell 指令碼示範如何設定虛擬機器組態，以及使用已上傳的 VM 映像做為新安裝的來源。



```powershell
# Enter a new user name and password to use as the local administrator account 
    # for remotely accessing the VM.
    $cred = Get-Credential

    # Name of the storage account where the VHD is located. This example sets the 
    # storage account name as "myStorageAccount"
    $storageAccName = "myStorageAccount"

    # Name of the virtual machine. This example sets the VM name as "myVM".
    $vmName = "myVM"

    # Size of the virtual machine. This example creates "Standard_D2_v2" sized VM. 
    # See the VM sizes documentation for more information: 
    # https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
    $vmSize = "Standard_D2_v2"

    # Computer name for the VM. This examples sets the computer name as "myComputer".
    $computerName = "myComputer"

    # Name of the disk that holds the OS. This example sets the 
    # OS disk name as "myOsDisk"
    $osDiskName = "myOsDisk"

    # Assign a SKU name. This example sets the SKU name as "Standard_LRS"
    # Valid values for -SkuName are: Standard_LRS - locally redundant storage, Standard_ZRS - zone redundant
    # storage, Standard_GRS - geo redundant storage, Standard_RAGRS - read access geo redundant storage,
    # Premium_LRS - premium locally redundant storage. 
    $skuName = "Standard_LRS"

    # Get the storage account where the uploaded image is stored
    $storageAcc = Get-AzureRmStorageAccount -ResourceGroupName $rgName -AccountName $storageAccName

    # Set the VM name and size
    $vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize

    #Set the Windows operating system configuration and add the NIC
    $vm = Set-AzureRmVMOperatingSystem -VM $vmConfig -Windows -ComputerName $computerName `
        -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
    $vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id

    # Create the OS disk URI
    $osDiskUri = '{0}vhds/{1}-{2}.vhd' `
        -f $storageAcc.PrimaryEndpoints.Blob.ToString(), $vmName.ToLower(), $osDiskName

    # Configure the OS disk to be created from the existing VHD image (-CreateOption fromImage).
    $vm = Set-AzureRmVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri `
        -CreateOption fromImage -SourceImageUri $imageURI -Windows

    # Create the new VM
    New-AzureRmVM -ResourceGroupName $rgName -Location $location -VM $vm
```

## <a name="verify-that-the-vm-was-created"></a>確認已建立 VM
完成時，在 [Azure 入口網站](https://portal.azure.com)的 [瀏覽] > [虛擬機器] 底下，或是使用下列 PowerShell 命令，應該就可以看到新建立的 VM：

```powershell
    $vmList = Get-AzureRmVM -ResourceGroupName $rgName
    $vmList.Name
```

## <a name="next-steps"></a>後續步驟
若要使用 Azure PowerShell 管理新的虛擬機器，請參閱 [使用 Azure Resource Manager 與 PowerShell 管理虛擬機器](tutorial-manage-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。


