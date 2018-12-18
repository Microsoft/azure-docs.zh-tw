---
title: 在 Azure Stack 中使用 PowerShell 建立 Windows 虛擬機器 | Microsoft Docs
description: 在 Azure Stack 中使用 PowerShell 建立 Windows 虛擬機器。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 09/10/2018
ms.author: mabrigg
ms.custom: mvc
ms.openlocfilehash: 4795de2126a34907ecdec69e87a059dbadd0c3d0
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44380381"
---
# <a name="quickstart-create-a-windows-server-virtual-machine-by-using-powershell-in-azure-stack"></a>快速入門：在 Azure Stack 中使用 PowerShell 建立 Windows Server 虛擬機器

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以使用 Azure Stack PowerShell 建立 Windows Server 2016 虛擬機器。 請遵循本文中的步驟建立和使用虛擬機器。 本文也提供下列操作的步驟：

* 使用遠端用戶端連線到虛擬機器。
* 安裝 IIS 網頁伺服器，並且檢視預設首頁。
* 清除資源。

>[!NOTE]
 您可以從 Azure Stack 開發套件，或從 Windows 型外部用戶端 (如果您透過 VPN 連線) 來執行這篇文章中所述的步驟。

## <a name="prerequisites"></a>必要條件

* 請確定您的 Azure Stack 操作員已將 **Windows Server 2016** 映像新增到 Azure Stack 市集。

* Azure Stack 需要特定版本的 Azure PowerShell，才能建立和管理資源。 如果您尚未針對 Azure Stack 設定 PowerShell，請依照步驟來[安裝](azure-stack-powershell-install.md) PowerShell。

* 設定好 Azure Stack PowerShell 後，您必須連線到 Azure Stack 環境。 如需指示，請參閱[以使用者的身分使用 PowerShell 連線到 Azure Stack](azure-stack-powershell-configure-user.md)。

## <a name="create-a-resource-group"></a>建立資源群組

資源群組是在其中部署與管理 Azure Stack 資源的邏輯容器。 從您的開發套件或 Azure Stack 整合系統，執行下列程式碼區塊來建立資源群組。 本文件中的所有變數都有指派值，您可以使用這些值或指派新的值。

```powershell
# Create variables to store the location and resource group names.
$location = "local"
$ResourceGroupName = "myResourceGroup"

New-AzureRmResourceGroup `
  -Name $ResourceGroupName `
  -Location $location
```

## <a name="create-storage-resources"></a>建立儲存體資源

建立儲存體帳戶和儲存體容器來儲存 Windows Server 2016 映像。

```powershell
# Create variables to store the storage account name and the storage account SKU information
$StorageAccountName = "mystorageaccount"
$SkuName = "Standard_LRS"

# Create a new storage account
$StorageAccount = New-AzureRMStorageAccount `
  -Location $location `
  -ResourceGroupName $ResourceGroupName `
  -Type $SkuName `
  -Name $StorageAccountName

Set-AzureRmCurrentStorageAccount `
  -StorageAccountName $storageAccountName `
  -ResourceGroupName $resourceGroupName

# Create a storage container to store the virtual machine image
$containerName = 'osdisks'
$container = New-AzureStorageContainer `
  -Name $containerName `
  -Permission Blob
```

## <a name="create-networking-resources"></a>建立網路資源

建立虛擬網路、子網路和公用 IP 位址。 這些資源用來提供虛擬機器的網路連線能力。

```powershell
# Create a subnet configuration
$subnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
  -Name mySubnet `
  -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzureRmVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -Name MyVnet `
  -AddressPrefix 192.168.0.0/16 `
  -Subnet $subnetConfig

# Create a public IP address and specify a DNS name
$pip = New-AzureRmPublicIpAddress `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -AllocationMethod Static `
  -IdleTimeoutInMinutes 4 `
  -Name "mypublicdns$(Get-Random)"
```

### <a name="create-a-network-security-group-and-a-network-security-group-rule"></a>建立網路安全性群組和網路安全性群組規則

網路安全性群組可透過使用輸入和輸出規則來保護虛擬機器。 讓我們建立連接埠 3389 的輸入規則以允許傳入的遠端桌面連線，並建立連接埠 80 的輸入規則以允許傳入的 Web 流量。

```powershell
# Create an inbound network security group rule for port 3389
$nsgRuleRDP = New-AzureRmNetworkSecurityRuleConfig `
  -Name myNetworkSecurityGroupRuleRDP `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 1000 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 3389 `
  -Access Allow

# Create an inbound network security group rule for port 80
$nsgRuleWeb = New-AzureRmNetworkSecurityRuleConfig `
  -Name myNetworkSecurityGroupRuleWWW `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 1001 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access Allow

# Create a network security group
$nsg = New-AzureRmNetworkSecurityGroup `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -Name myNetworkSecurityGroup `
  -SecurityRules $nsgRuleRDP,$nsgRuleWeb
```

### <a name="create-a-network-card-for-the-virtual-machine"></a>建立虛擬機器的網路卡

網路卡可讓虛擬機器連線到子網路、網路安全性群組和公用 IP 位址。

```powershell
# Create a virtual network card and associate it with public IP address and NSG
$nic = New-AzureRmNetworkInterface `
  -Name myNic `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id `
  -NetworkSecurityGroupId $nsg.Id
```

## <a name="create-a-virtual-machine"></a>建立虛擬機器

建立虛擬機器組態。 此組態包括部署虛擬機器時使用的設定。 例如：認證、大小和虛擬機器映像。

```powershell
# Define a credential object to store the username and password for the virtual machine
$UserName='demouser'
$Password='Password@123'| ConvertTo-SecureString -Force -AsPlainText
$Credential=New-Object PSCredential($UserName,$Password)

# Create the virtual machine configuration object
$VmName = "VirtualMachinelatest"
$VmSize = "Standard_A1"
$VirtualMachine = New-AzureRmVMConfig `
  -VMName $VmName `
  -VMSize $VmSize

$VirtualMachine = Set-AzureRmVMOperatingSystem `
  -VM $VirtualMachine `
  -Windows `
  -ComputerName "MainComputer" `
  -Credential $Credential

$VirtualMachine = Set-AzureRmVMSourceImage `
  -VM $VirtualMachine `
  -PublisherName "MicrosoftWindowsServer" `
  -Offer "WindowsServer" `
  -Skus "2016-Datacenter" `
  -Version "latest"

$osDiskName = "OsDisk"
$osDiskUri = '{0}vhds/{1}-{2}.vhd' -f `
  $StorageAccount.PrimaryEndpoints.Blob.ToString(),`
  $vmName.ToLower(), `
  $osDiskName

# Sets the operating system disk properties on a virtual machine.
$VirtualMachine = Set-AzureRmVMOSDisk `
  -VM $VirtualMachine `
  -Name $osDiskName `
  -VhdUri $OsDiskUri `
  -CreateOption FromImage | `
  Add-AzureRmVMNetworkInterface -Id $nic.Id

# Create the virtual machine.
New-AzureRmVM `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -VM $VirtualMachine
```

## <a name="connect-to-the-virtual-machine"></a>連接至虛擬機器

若要遠端存取您在上一個步驟中所建立的虛擬機器，則需要其公用 IP 位址。 執行下列命令，以取得虛擬機器的公用 IP 位址：

```powershell
Get-AzureRmPublicIpAddress `
  -ResourceGroupName $ResourceGroupName | Select IpAddress
```

使用下列命令，建立使用虛擬機器的遠端桌面工作階段。 以虛擬機器的公用 IP 位址取代 IP 位址。 出現提示時，輸入您在建立虛擬機器時所使用的使用者名稱和密碼。

```powershell
mstsc /v <publicIpAddress>
```

## <a name="install-iis-via-powershell"></a>透過 PowerShell 安裝 IIS

您現已登入 Azure VM，可使用一行 PowerShell 來安裝 IIS，並啟用本機防火牆規則以允許 Web 流量通過。 開啟 PowerShell 提示字元並執行下列命令：

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

## <a name="view-the-iis-welcome-page"></a>檢視 IIS 歡迎使用頁面

安裝 IIS 後，在您的 VM 上開啟連接埠 80，即可使用所選的網頁瀏覽器來檢視預設的 IIS 歡迎使用畫面。 使用您在前一節中記載的 publicIpAddress 來瀏覽預設網頁。

![IIS 預設網站](./media/azure-stack-quick-create-vm-windows-powershell/default-iis-website.png)

## <a name="delete-the-virtual-machine"></a>刪除虛擬機器

當不再需要時，請使用下列命令來移除包含虛擬機器及其相關資源的資源群組：

```powershell
Remove-AzureRmResourceGroup `
  -Name $ResourceGroupName
```

## <a name="next-steps"></a>後續步驟

在這個快速入門中，您已部署簡單的 Windows 虛擬機器。 若要深入了解 Azure Stack 虛擬機器，請繼續移至 [Azure Stack 中虛擬機器的考量](azure-stack-vm-considerations.md)。
