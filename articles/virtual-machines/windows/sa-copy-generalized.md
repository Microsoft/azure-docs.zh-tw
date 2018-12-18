---
title: 在 Azure 中建立一般化 VM 非受控映像 | Microsoft Docs
description: 建立一般化的 Windows VM 未受管理的映像，在 Azure 中用以建立多個 VM 複本。
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
ms.date: 05/23/2017
ms.author: cynthn
ROBOTS: NOINDEX
ms.openlocfilehash: b416acd9a2a3b03502b7eca11eade9dbd56f3afe
ms.sourcegitcommit: c52123364e2ba086722bc860f2972642115316ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34072044"
---
# <a name="how-to-create-an-unmanaged-vm-image-from-an-azure-vm"></a>如何從 Azure VM 建立非受控 VM 映像

本文章涵蓋使用儲存體帳戶的內容。 建議您使用受控磁碟和受管理的映像，不要使用儲存體帳戶。 如需詳細資訊，請參閱[在 Azure 中擷取一般化 VM 的受控映像](capture-image-resource.md)。

本文示範如何使用 Azure PowerShell 建立使用儲存體帳戶的一般化 Azure VM 的映像。 然後可以使用映像來建立另一個 VM。 此映像包含作業系統磁碟與連結到虛擬機器的資料磁碟。 映像不包含虛擬網路資源，因此您需要在建立新的 VM 時設定這些資源。 

## <a name="prerequisites"></a>先決條件
您需要安裝 Azure PowerShell 1.0.x 版或更新版本。 如果您尚未安裝 PowerShell，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 以了解安裝步驟。

## <a name="generalize-the-vm"></a>一般化 VM 
本節說明如何將 Windows 虛擬機器一般化以做為映像。 將 VM 一般化會移除您的所有個人帳戶資訊，以及其他項目，並準備電腦作為映像。 如需 Sysprep 的詳細資訊，請參閱 [如何使用 Sysprep：簡介](http://technet.microsoft.com/library/bb457073.aspx)。

請確定 Sysprep 支援電腦上執行的伺服器角色。 如需詳細資訊，請參閱 [Sysprep Support for Server Roles (伺服器角色的 Sysprep 支援)](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)

> [!IMPORTANT]
> 如果您是第一次將 VHD 上傳至 Azure，請確定您已[準備好 VM](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 再執行 Sysprep。 
> 
> 

您也可以使用 `sudo waagent -deprovision+user` 將 Linux VM 一般化，然後使用 PowerShell 來擷取該 VM. 如需有關使用 CLI 來擷取 VM 的資訊，請參閱[如何使用 Azure CLI 來一般化和擷取 Linux 虛擬機器](../linux/capture-image.md)。


1. 登入 Windows 虛擬機器。
2. 以系統管理員身分開啟 [命令提示字元] 視窗。 切換至 **%windir%\system32\sysprep** 目錄，然後執行 `sysprep.exe`。
3. 在 [系統準備工具] 對話方塊中，選取 [進入系統全新體驗 (OOBE)]，並確認已勾選 [一般化] 核取方塊。
4. 在 [關機選項] 中選取 [關機]。
5. 按一下 [確定]。
   
    ![啟動 Sysprep](./media/upload-generalized-managed/sysprepgeneral.png)
6. Sysprep 完成時，會關閉虛擬機器。 

> [!IMPORTANT]
> 在您完成將 VHD 上傳到 Azure 或從 VM 建立映像之前，請勿重新啟動 VM。 如果 VM 意外重新啟動，請執行 Sysprep 來重新將它一般化。
> 
> 

## <a name="log-in-to-azure-powershell"></a>登入 Azure PowerShell
1. 開啟 Azure PowerShell，並登入您的 Azure 帳戶。
   
    ```powershell
    Connect-AzureRmAccount
    ```
   
    這會開啟一個可供您輸入 Azure 帳戶認證的快顯視窗。
2. 取得您可用訂用帳戶的訂用帳戶識別碼。
   
    ```powershell
    Get-AzureRmSubscription
    ```
3. 使用訂用帳戶識別碼來設定正確的訂用帳戶。
   
    ```powershell
    Select-AzureRmSubscription -SubscriptionId "<subscriptionID>"
    ```

## <a name="deallocate-the-vm-and-set-the-state-to-generalized"></a>解除配置 VM 並將狀態設定為一般化

> [!IMPORTANT] 
> 一旦將標記標示為一般化之後，您就無法從 VM 新增、編輯或移除標記。 如果您要將標記新增至 VM，請確定先新增標記，再將其標示為一般化。
> 

1. 解除配置 VM 資源。
   
    ```powershell
    Stop-AzureRmVM -ResourceGroupName <resourceGroup> -Name <vmName>
    ```
   
    Azure 入口網站中 VM 的 [狀態] 會從 [已停止] 變更為 [已停止 (已解除配置)]。
2. 將虛擬機器的狀態設定為 [一般化] 。 
   
    ```powershell
    Set-AzureRmVm -ResourceGroupName <resourceGroup> -Name <vmName> -Generalized
    ```
3. 檢查 VM 的狀態。 VM 的 [OSState/一般化] 區段中的 [DisplayStatus] 應設定為 [VM 一般化]。  
   
    ```powershell
    $vm = Get-AzureRmVM -ResourceGroupName <resourceGroup> -Name <vmName> -Status
    $vm.Statuses
    ```

## <a name="create-the-image"></a>建立映像

使用這個命令，在目的地儲存體容器中建立非受控虛擬機器映像。 此映像會建立在與原始虛擬機器相同的儲存體帳戶中。 `-Path` 參數會將來源 VM 的 JSON 範本複本儲存到本機電腦。 `-DestinationContainerName` 參數是要用以保存映像的容器名稱。 如果此容器不存在，則會為您建立。
   
```powershell
Save-AzureRmVMImage -ResourceGroupName <resourceGroupName> -Name <vmName> `
    -DestinationContainerName <destinationContainerName> -VHDNamePrefix <templateNamePrefix> `
    -Path <C:\local\Filepath\Filename.json>
```
   
您可以從 JSON 檔案範本取得映像的 URL。 請依序前往 [資源]  >  [storageProfile]  >  [osDisk]  >  [映像]  >  [uri] 區段，取得您映像的完整路徑。 映像的 URL 如下所示： `https://<storageAccountName>.blob.core.windows.net/system/Microsoft.Compute/Images/<imagesContainer>/<templatePrefix-osDisk>.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.vhd`
   
您也可以在入口網站中驗證 URI。 系統會將映像複製到儲存體帳戶中名為 **system** 的容器中。 

## <a name="create-a-vm-from-the-image"></a>從映像建立 VM

現在您可以從非受控映像建立一或多部 VM。

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
下列 PowerShell 會完成虛擬機器設定，並使用非受控映像當做新安裝的來源。

</br>

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

### <a name="verify-that-the-vm-was-created"></a>確認已建立 VM
完成時，在 [Azure 入口網站](https://portal.azure.com)的 [瀏覽] > [虛擬機器] 底下，或是使用下列 PowerShell 命令，應該就可以看到新建立的 VM：

```powershell
    $vmList = Get-AzureRmVM -ResourceGroupName $rgName
    $vmList.Name
```

## <a name="next-steps"></a>後續步驟
若要使用 Azure PowerShell 管理新的虛擬機器，請參閱 [使用 Azure Resource Manager 與 PowerShell 管理虛擬機器](tutorial-manage-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。


