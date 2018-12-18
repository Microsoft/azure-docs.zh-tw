---
title: 使用加速網路來建立 Azure 虛擬機器 | Microsoft Docs
description: 了解如何使用加速網路建立 Linux 虛擬機器。
services: virtual-network
documentationcenter: ''
author: gsilva5
manager: gedegrac
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 01/04/2018
ms.author: gsilva
ms.openlocfilehash: de69cdf69f30639d048dccd7d433c86f6cb9db7b
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2018
ms.locfileid: "33894174"
---
# <a name="create-a-windows-virtual-machine-with-accelerated-networking"></a>建立使用加速網路的 Windows 虛擬機器

在本教學課程中，您將了解如何使用加速網路來建立 Windows 虛擬機器 (VM)。 若要建立使用加速網路的 Linux VM，請參閱[建立使用加速網路的 Linux VM](create-vm-accelerated-networking-cli.md)。 加速網路可以對 VM 啟用 Single Root I/O Virtualization (SR-IOV)，大幅提升其網路效能。 這個高效能路徑會略過資料路徑的主機，進而減少延遲、抖動和 CPU 使用率，供支援的 VM 類型中最嚴苛的網路工作負載使用。 下圖顯示兩部 VM 之間的通訊，一部具備加速網路而另一步沒有︰

![比較](./media/create-vm-accelerated-networking/accelerated-networking.png)

如果沒有加速網路，進出 VM 的所有網路流量都必須周遊主機和虛擬交換器。 虛擬交換器對網路流量提供所有原則強制執行，例如網路安全性群組、存取控制清單、隔離性以及其他網路虛擬化服務。 若要深入了解虛擬交換器，請閱讀 [Hyper-V Network Virtualization and Virtual Switch (Hyper-V 網路虛擬化和虛擬交換器)](https://technet.microsoft.com/library/jj945275.aspx) 文章。

如果使用加速網路，網路流量就會先送達 VM 的網路介面 (NIC)，然後轉送到 VM。 虛擬交換器套用的所有網路原則現在皆已卸載，並在硬體中套用。 在硬體中套用原則會讓 NIC 略過主機和虛擬交換器，同時在主機中維護套用的所有原則，直接將網路流量轉送到 VM。

加速網路的優點只適用於已啟用此功能的 VM。 為了獲得最佳結果，最好在至少兩部連線到相同 Azure 虛擬網路 (VNet) 的 VM 上啟用此功能。 當透過 VNet 通訊或連線內部部署時，此功能對整體延遲的影響可以降到最低。

## <a name="benefits"></a>優點
* **較低的延遲 / 較高的每秒封包數目 (pps)：** 從資料路徑移除虛擬交換器會減少主機中封包在處理原則時所花的時間，並增加 VM 內可處理的封包數目。
* **減少抖動︰** 虛擬交換器處理視需要套用的原則數量和正在進行處理的 CPU 工作負載而定。 將原則強制執行卸載到硬體透過將封包直接傳遞到 VM、移除主機到 VM 的通訊，以及所有軟體插斷和環境切換，而減少變化。
* **降低 CPU 使用率︰** 略過主機中的虛擬交換器可減少處理網路流量的 CPU 使用率。

## <a name="limitations-and-constraints"></a>限制和條件約束

### <a name="supported-operating-systems"></a>受支援的作業系統
您可以從 Azure 資源庫直接使用下列發行版本： 
* **Windows Server 2016 Datacenter** 
* **Windows Server 2012 R2 Datacenter** 

### <a name="supported-vm-instances"></a>支援的 VM 執行個體
大多數一般用途和具有 2 個以上 vCPU 的計算最佳化執行個體大小，皆支援加速網路。  這些支援的系列為：D/DSv2 和 F/Fs

在支援超執行緒的執行個體中，加速網路可在具有 4 個以上 vCPU 的 VM 執行個體上進行支援作業。 支援的系列為：D/DSv3、E/ESv3、Fsv2 和 Ms/Mms

如需 VM 執行個體的詳細資訊，請參閱 [Windows VM 大小](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

### <a name="regions"></a>區域
適用於所有公用 Azure 區域和 Azure 政府雲端。

### <a name="enabling-accelerated-networking-on-a-running-vm"></a>在執行中的 VM 上啟用加速網路
支援的 VM 大小若沒啟用加速網路，則只能在停止或解除配置 VM 時啟用此功能。

### <a name="deployment-through-azure-resource-manager"></a>透過 Azure Resource Manager 進行部署
虛擬機器 (傳統) 無法透過加速網路部署。

## <a name="create-a-windows-vm-with-azure-accelerated-networking"></a>建立使用加速網路的 Azure Windows VM

雖然本文提供使用 Azure PowerShell 來建立具有加速網路之虛擬機器的步驟，但您也可以[使用 Azure 入口網站來建立具有加速網路的虛擬機器](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 在入口網站中建立虛擬機器時，請在 [設定] 下的 [加速的網路] 底下，選取 [已啟用]。 除非您已選取[支援的作業系統](#supported-operating-systems)和 [VM 大小](#supported-vm-instances)，否則啟用加速網路的選項不會出現在入口網站中。 建立虛擬機器之後，您必須完成[確認作業系統中已安裝驅動程式](#confirm-the-driver-is-installed-in-the-operating-system)中的指示動作。

## <a name="create-a-virtual-network"></a>建立虛擬網路

安裝 [Azure PowerShell](/powershell/azure/install-azurerm-ps) 5.1.1 或更新版本。 若要尋找您目前安裝的版本，請執行 `Get-Module -ListAvailable AzureRM`。 如果您需要安裝或升級，請從 [PowerShell 資源庫](https://www.powershellgallery.com/packages/AzureRM)安裝最新版的 AzureRM 模組。 在 PowerShell 工作階段中，使用 [Connect-AzureRmAccount](/powershell/module/azurerm.profile/connect-azurermaccount) 登入 Azure 帳戶。

在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包含 *myResourceGroup*、*myNic* 和 *myVM*。

使用 [New-AzureRmResourceGroup](/powershell/module/AzureRM.Resources/New-AzureRmResourceGroup)建立資源群組。 下列範例會在 *centralus* 位置建立名為 *myResourceGroup* 的資源群組：

```powershell
New-AzureRmResourceGroup -Name "myResourceGroup" -Location "centralus"
```

首先，請使用 [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/AzureRM.Network/New-AzureRmVirtualNetworkSubnetConfig) 來建立子網路設定。 下列範例會建立名為 mySubnet 的子網路：

```powershell
$subnet = New-AzureRmVirtualNetworkSubnetConfig `
    -Name "mySubnet" `
    -AddressPrefix "192.168.1.0/24"
```

透過 *mySubnet* s子網路使用 [New-AzureRmVirtualNetwork](/powershell/module/AzureRM.Network/New-AzureRmVirtualNetwork) 建立虛擬網路。

```powershell
$vnet = New-AzureRmVirtualNetwork -ResourceGroupName "myResourceGroup" `
    -Location "centralus" `
    -Name "myVnet" `
    -AddressPrefix "192.168.0.0/16" `
    -Subnet $Subnet
```

## <a name="create-a-network-security-group"></a>建立網路安全性群組

首先，使用 [New-AzureRmNetworkSecurityRuleConfig](/powershell/module/AzureRM.Network/New-AzureRmNetworkSecurityRuleConfig) 建立網路安全性群組規則。

```powershell
$rdp = New-AzureRmNetworkSecurityRuleConfig `
    -Name 'Allow-RDP-All' `
    -Description 'Allow RDP' `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 100 `
    -SourceAddressPrefix * `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389
```

使用 [New-AzureRmNetworkSecurityGroup](/powershell/module/AzureRM.Network/New-AzureRmNetworkSecurityGroup) 建立網路安全性群組，並指派 *Allow-RDP-All* 安全性規則給它。 除了 *Allow-RDP-All* 規則之外，網路安全性群組還包含數個預設規則。 一個預設規則會停用來自網際網路的所有輸入存取，這也是將 *Allow-RDP-All* 指派給網路安全性原則的原因，讓您在建立虛擬機器之後，能夠從遠端連線。

```powershell
$nsg = New-AzureRmNetworkSecurityGroup `
    -ResourceGroupName myResourceGroup `
    -Location centralus `
    -Name "myNsg" `
    -SecurityRules $rdp
```

使用 [Set-AzureRmVirtualNetworkSubnetConfig](/powershell/module/AzureRM.Network/Set-AzureRmVirtualNetworkSubnetConfig) 將網路安全性群組與 *mySubnet* 子網路建立關聯。 網路安全性群組中的規則適用於子網路中部署的所有資源。

```powershell
Set-AzureRmVirtualNetworkSubnetConfig `
    -VirtualNetwork $vnet `
    -Name 'mySubnet' `
    -AddressPrefix "192.168.1.0/24" `
    -NetworkSecurityGroup $nsg
```

## <a name="create-a-network-interface-with-accelerated-networking"></a>使用加速網路建立網路介面
使用 [New-AzureRmPublicIpAddress](/powershell/module/AzureRM.Network/New-AzureRmPublicIpAddress) 建立公用 IP 位址。 如果您不打算從網際網路存取虛擬機器，那麼您就不需要公用 IP 位址，但如果要完成本文中的步驟，就會需要公用 IP 位址。

```powershell
$publicIp = New-AzureRmPublicIpAddress `
    -ResourceGroupName myResourceGroup `
    -Name 'myPublicIp' `
    -location centralus `
    -AllocationMethod Dynamic
```

建立具 [New-AzureRmNetworkInterface](/powershell/module/AzureRM.Network/New-AzureRmNetworkInterface) 且啟用加速網路的網路介面，並將公用 IP 位址指派給網路介面。 下列範例會在 *myVnet* 虛擬網路的 *mySubnet*子網路中建立名為 *myNic* 的網路介面，並指派*myPublicIp* 公用 IP 位址給它：

```powershell
$nic = New-AzureRmNetworkInterface `
    -ResourceGroupName "myResourceGroup" `
    -Name "myNic" `
    -Location "centralus" `
    -SubnetId $vnet.Subnets[0].Id `
    -PublicIpAddressId $publicIp.Id `
    -EnableAcceleratedNetworking
```

## <a name="create-the-virtual-machine"></a>建立虛擬機器

使用 [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential) 將您的 VM 認證設定為 `$cred` 變數︰

```powershell
$cred = Get-Credential
```

首先，使用 [New-AzureRmVMConfig](/powershell/module/azurerm.compute/new-azurermvmconfig) 來定義您的 VM。 下列範例會使用支援加速網路的 VM 大小來定義名為 *myVM* 的 VM (*Standard_DS4_v2*)：

```powershell
$vmConfig = New-AzureRmVMConfig -VMName "myVm" -VMSize "Standard_DS4_v2"
```

如需所有 VM 大小和特性的清單，請參閱 [Windows VM 大小](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

使用 [Set-AzureRmVMOperatingSystem](/powershell/module/azurerm.compute/set-azurermvmoperatingsystem) 和 [Set-AzureRmVMSourceImage](/powershell/module/azurerm.compute/set-azurermvmsourceimage) 建立其餘的 VM 組態。 下列範例會建立 Windows Server 2016 VM：

```powershell
$vmConfig = Set-AzureRmVMOperatingSystem -VM $vmConfig `
    -Windows `
    -ComputerName "myVM" `
    -Credential $cred `
    -ProvisionVMAgent `
    -EnableAutoUpdate
$vmConfig = Set-AzureRmVMSourceImage -VM $vmConfig `
    -PublisherName "MicrosoftWindowsServer" `
    -Offer "WindowsServer" `
    -Skus "2016-Datacenter" `
    -Version "latest"
```

使用 [Add-AzureRmVMNetworkInterface](/powershell/module/azurerm.compute/add-azurermvmnetworkinterface) 連結您先前建立的網路介面：

```powershell
$vmConfig = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic.Id
```

最後，使用 [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) 建立 VM：

```powershell
New-AzureRmVM -VM $vmConfig -ResourceGroupName "myResourceGroup" -Location "centralus"
```

## <a name="confirm-the-driver-is-installed-in-the-operating-system"></a>確認作業系統中已安裝驅動程式

一旦您在 Azure 中建立 VM，請與 VM 連線，然後確認驅動程式已安裝在 Windows 中。

1. 從網際網路瀏覽器開啟 Azure [入口網站](https://portal.azure.com)，並以您的 Azure 帳戶登入。
2. 在 Azure 入口網站頂端包含「搜尋資源」文字的方塊中，輸入 myVm。 當搜尋結果中出現 **myVm** 時，按一下它。 如果在 [連線] 按鈕底下看到 [建立中]，則表示 Azure 還未建立好 VM。 請在 [連線] 按鈕底下已沒有 [建立中] 字樣時，才按下概觀左上角的 [連線]。
3. 輸入您在[建立虛擬機器](#create-the-virtual-machine)中所輸入的使用者名稱和密碼。 如果您從未連線到 Azure 中的 Windows VM，請參閱[連線至虛擬機器](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json#connect-to-virtual-machine)。
4. 以滑鼠右鍵按一下 Windows 的 [開始] 按鈕，然後按一下 [裝置管理員]。 展開 [網路介面卡] 節點。 確認 **Mellanox ConnectX-3 Virtual Function Ethernet Adapter** 有出現，如下圖所示︰

    ![裝置管理員](./media/create-vm-accelerated-networking/device-manager.png)

現在已啟用您 VM 的加速網路。

## <a name="enable-accelerated-networking-on-existing-vms"></a>在現有的 VM 上啟用加速網路
如果您已建立不含加速網路的 VM，那麼在現有 VM 上啟用此功能是可能的。  VM 必須符合前面也說明過的下列必要條件，才能支援加速網路：

* VM 必須是支援加速網路的大小
* VM 必須是支援的 Azure 資源庫映像 (及 Linux 的核心版本)
* 可用性設定組中的所有 VM 或 VMSS 必須先停止/解除配置，然後才能在任何 NIC 上啟用加速網路

### <a name="individual-vms--vms-in-an-availability-set"></a>個別 VM 與可用性設定組中的多個 VM
首先要停止/解除配置 VM 或可用性設定組中的所有 VM：

```azurepowershell
Stop-AzureRmVM -ResourceGroup "myResourceGroup" `
    -Name "myVM"
```

請務必注意，如果您的 VM 是個別建立，沒有使用可用性設定組，則您只需要停止/解除配置該個別 VM，即可啟用加速網路。  如果您的 VM 是使用可用性設定組建立的，則包含在可用性設定組中的所有 VM 皆必須先停止/解除配置之後，才能在任何 NIC 上啟用加速網路。 

停止之後，在您 VM 的 NIC 上啟用加速網路：

```azurepowershell
$nic = Get-AzureRmNetworkInterface -ResourceGroupName "myResourceGroup" `
    -Name "myNic"

$nic.EnableAcceleratedNetworking = $true

$nic | Set-AzureRmNetworkInterface
```

重新啟動您 VM 或可用性設定組中的所有 VM，然後確認加速網路已啟用： 

```azurepowershell
Start-AzureRmVM -ResourceGroup "myResourceGroup" `
    -Name "myVM"
```

### <a name="vmss"></a>VMSS
VMSS 稍有不同，但會遵循相同的工作流程。  首先，停止 VM：

```azurepowershell
Stop-AzureRmVmss -ResourceGroupName "myResourceGroup" ` 
    -VMScaleSetName "myScaleSet"
```

停止 VM 後，更新網路介面下的加速網路屬性：

```azurepowershell
$vmss = Get-AzureRmVmss -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet"

$vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].EnableAcceleratedNetworking = $true

Update-AzureRmVmss -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet" `
    -VirtualMachineScaleSet $vmss
```

請注意，VMSS 會升級 VM，其使用三種不同的設定 (自動、輪流及手動) 來套用更新。  將這些指示中的原則設為自動，以便 VMSS 在重新啟動之後，立即採用變更。  將其設定為自動，讓所做的變更可立即採用： 

```azurepowershell
$vmss.UpgradePolicy.AutomaticOSUpgrade = $true

Update-AzureRmVmss -ResourceGroupName "myResourceGroup" `
    -VMScaleSetName "myScaleSet" `
    -VirtualMachineScaleSet $vmss
```

最後，重新啟動 VMSS：

```azurepowershell
Start-AzureRmVmss -ResourceGroupName "myResourceGroup" ` 
    -VMScaleSetName "myScaleSet"
```

重新啟動後，等候升級完成，但是一旦完成後，VF 會出現在 VM 內。  (請確定您是使用支援的 OS 和 VM 大小。)

### <a name="resizing-existing-vms-with-accelerated-networking"></a>調整使用加速網路的現有 VM 大小

針對已啟用加速網路的 VM，您只能將其大小調整為其他支援加速網路的 VM。  

您無法使用調整作業將已啟用加速網路的 VM 大小，調整為不支援加速網路的 VM 執行個體。  相反的，若要為這些 VM 的其中一個調整大小，請執行下列動作： 

* 停止/解除配置 VM 或可用性設定組/VMSS 中的所有 VM。
* 必須在 VM 或可用性設定組/VMSS 中所有 VM 的 NIC 上停用加速網路。
* 一旦停用加速網路之後，VM/可用性設定組/VMSS 就可以移到不支援加速網路的新大小，並重新啟動。  



