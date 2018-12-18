---
title: 快速入門 - 使用 Azure PowerShell 建立 Linux VM | Microsoft Docs
description: 在本快速入門中，您會了解如何使用 Azure PowerShell 來建立 Linux 虛擬機器
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/24/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 45b1597d3b61a9386fc015b5a7272d948fa5772b
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394777"
---
# <a name="quickstart-create-a-linux-virtual-machine-in-azure-with-powershell"></a>快速入門：使用 PowerShell 在 Azure 中建立 Linux 虛擬機器

Azure PowerShell 模組用於從 PowerShell 命令列或在指令碼中建立和管理 Azure 資源。 本快速入門說明如何使用 Azure PowerShell 模組，在 Azure 中部署執行 Ubuntu 的 Linux 虛擬機器 (VM)。 若要查看作用中的 VM，接著要以 SSH 連線至 VM，並安裝 NGINX 網頁伺服器。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-powershell.md](../../../includes/cloud-shell-powershell.md)]

如果您選擇在本機安裝和使用 PowerShell，則在執行本教學課程時，必須使用 Azure PowerShell 模組 5.7.0 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure PowerShell 模組](/powershell/azure/install-azurerm-ps)。 如果您在本機執行 PowerShell，則也需要執行 `Connect-AzureRmAccount` 以建立與 Azure 的連線。

最後，名稱為 id_rsa.pub 的公開 SSH 金鑰必須儲存在 Windows 使用者設定檔的 .ssh 目錄中。 如需建立及使用 SSH 金鑰的詳細資訊，請參閱[建立適用於 Azure 的 SSH 金鑰](ssh-from-windows.md)。

## <a name="create-a-resource-group"></a>建立資源群組

使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立 Azure 資源群組。 資源群組是在其中部署與管理 Azure 資源的邏輯容器：

```azurepowershell-interactive
New-AzureRmResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

## <a name="create-virtual-network-resources"></a>建立虛擬網路資源

建立虛擬網路、子網路和公用 IP 位址。 這些資源可用來提供 VM 的網路連線能力，並可將它連線到網際網路：

```azurepowershell-interactive
# Create a subnet configuration
$subnetConfig = New-AzureRmVirtualNetworkSubnetConfig -Name "mySubnet" -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzureRmVirtualNetwork -ResourceGroupName "myResourceGroup" -Location "EastUS" `
-Name "myVNET" -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

# Create a public IP address and specify a DNS name
$pip = New-AzureRmPublicIpAddress -ResourceGroupName "myResourceGroup" -Location "EastUS" `
-AllocationMethod Static -IdleTimeoutInMinutes 4 -Name "mypublicdns$(Get-Random)"
```

建立 Azure 網路安全性群組及流量規則。 網路安全性群組可使用輸入和輸出規則來保護 VM。 下列範例會針對允許 SSH 連線的 TCP 通訊埠 22 建立輸入規則。 為了允許傳入的網路流量，還會針對 TCP 通訊埠 80 建立輸入規則。

```azurepowershell-interactive
# Create an inbound network security group rule for port 22
$nsgRuleSSH = New-AzureRmNetworkSecurityRuleConfig -Name "myNetworkSecurityGroupRuleSSH"  -Protocol "Tcp" `
-Direction "Inbound" -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
-DestinationPortRange 22 -Access "Allow"

# Create an inbound network security group rule for port 80
$nsgRuleWeb = New-AzureRmNetworkSecurityRuleConfig -Name "myNetworkSecurityGroupRuleWWW"  -Protocol "Tcp" `
-Direction "Inbound" -Priority 1001 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
-DestinationPortRange 80 -Access "Allow"

# Create a network security group
$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName "myResourceGroup" -Location "EastUS" `
-Name "myNetworkSecurityGroup" -SecurityRules $nsgRuleSSH,$nsgRuleWeb
```

使用 [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) 建立虛擬網路介面卡 (NIC)。 虛擬 NIC 會將 VM 連線至子網路、網路安全性群組和公用 IP 位址。

```azurepowershell-interactive
# Create a virtual network card and associate with public IP address and NSG
$nic = New-AzureRmNetworkInterface -Name "myNic" -ResourceGroupName "myResourceGroup" -Location "EastUS" `
-SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id
```

## <a name="create-a-virtual-machine"></a>建立虛擬機器

虛擬機器組態包括部署 VM 時所使用的設定，例如 VM 映像、大小和驗證選項。 如下所示地定義 SSH 認證、OS 資訊與 VM 大小：

```azurepowershell-interactive
# Define a credential object
$securePassword = ConvertTo-SecureString ' ' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("azureuser", $securePassword)

# Create a virtual machine configuration
$vmConfig = New-AzureRmVMConfig -VMName "myVM" -VMSize "Standard_D1" | `
Set-AzureRmVMOperatingSystem -Linux -ComputerName "myVM" -Credential $cred -DisablePasswordAuthentication | `
Set-AzureRmVMSourceImage -PublisherName "Canonical" -Offer "UbuntuServer" -Skus "16.04-LTS" -Version "latest" | `
Add-AzureRmVMNetworkInterface -Id $nic.Id

# Configure SSH Keys
$sshPublicKey = Get-Content "$env:USERPROFILE\.ssh\id_rsa.pub"
Add-AzureRmVMSshPublicKey -VM $vmconfig -KeyData $sshPublicKey -Path "/home/azureuser/.ssh/authorized_keys"
```

現在，使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 合併上述要建立的組態定義：

```azurepowershell-interactive
New-AzureRmVM -ResourceGroupName "myResourceGroup" -Location eastus -VM $vmConfig
```

## <a name="connect-to-virtual-machine"></a>連線至虛擬機器

部署完成後，以 SSH 連線至 VM。 為了查看 VM 的運作，接著會安裝 NGINX Web 伺服器。

若要查看 VM 的公用 IP 位址，請使用 [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) Cmdlet：

```azurepowershell-interactive
Get-AzureRmPublicIpAddress -ResourceGroupName "myResourceGroup" | Select "IpAddress"
```

使用 SSH 用戶端來連線至 VM。 您可以從網頁瀏覽器使用 Azure Cloud Shell，或者，如果您使用 Windows，則可以使用 [Putty](ssh-from-windows.md) 或[適用於 Linux 的 Windows 子系統](/windows/wsl/install-win10)。 提供 VM 的公用 IP 位址：

```bash
ssh azureuser@IpAddress
```

出現提示時，登入使用者名稱為 azureuser。 如果 SSH 金鑰搭配使用複雜密碼，則需要在提示時輸入該複雜密碼。

## <a name="install-web-server"></a>安裝 Web 伺服器

若要查看作用中的 VM，請安裝 NGINX 網頁伺服器。 若要更新套件來源並安裝最新的 NGINX 套件，請從 SSH 工作階段執行下列命令：

```bash
# update packages
sudo apt-get -y update

# install NGINX
sudo apt-get -y install nginx
```

完成時，`exit` (結束) SSH 工作階段

## <a name="view-the-web-server-in-action"></a>檢視 Web 伺服器的運作

在已安裝 NGINX 且現在已開放從網際網路連往 VM 的連接埠 80 後，請使用所選的網頁瀏覽器來檢視預設的 NGINX 歡迎頁面。 使用上一個步驟所取得 VM 的公用 IP 位址。 下列範例示範預設的 NGINX 網站：

![預設 NGINX 網站](./media/quick-create-cli/nginx.png)

## <a name="clean-up-resources"></a>清除資源

若不再需要，您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) Cmdlet 來移除資源群組、VM 和所有相關資源：

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name "myResourceGroup"
```

## <a name="next-steps"></a>後續步驟

在此快速入門中，您已部署簡單的虛擬機器、建立網路安全性群組和規則，並已安裝基本的 Web 伺服器。 若要深入了解 Azure 虛擬機器，請繼續 Linux VM 的教學課程。

> [!div class="nextstepaction"]
> [Azure Linux 虛擬機器教學課程](./tutorial-manage-vm.md)
