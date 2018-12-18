---
title: 設定 VM 的私人 IP 位址 - Azure PowerShell | Microsoft Docs
description: 了解如何使用 PowerShell 設定虛擬機器的私人 IP 位址。
services: virtual-network
documentationcenter: na
author: jimdial
manager: timlt
editor: tysonn
tags: azure-resource-manager
ms.assetid: d5f18929-15e3-40a2-9ee3-8188bc248ed8
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/23/2016
ms.author: jdial
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: b0e8153f1d0cecd4efe66dc7cce64addd6ed62aa
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38307664"
---
# <a name="configure-private-ip-addresses-for-a-virtual-machine-using-powershell"></a>使用 PowerShell 設定虛擬機器的私人 IP 位址

[!INCLUDE [virtual-networks-static-private-ip-selectors-arm-include](../../includes/virtual-networks-static-private-ip-selectors-arm-include.md)]

[!INCLUDE [virtual-networks-static-private-ip-intro-include](../../includes/virtual-networks-static-private-ip-intro-include.md)]

Azure 有兩個部署模型：Azure Resource Manager 和傳統。 Microsoft 建議透過 Resource Manager 部署模型建立資源。 若要深入了解兩個模型的差異，請閱讀[了解 Azure 部署模型](../azure-resource-manager/resource-manager-deployment-model.md)。 本文涵蓋之內容包括資源管理員部署模型。 您也可以 [管理傳統部署模型中的靜態私人 IP 位址](virtual-networks-static-private-ip-classic-ps.md)。

[!INCLUDE [virtual-networks-static-ip-scenario-include](../../includes/virtual-networks-static-ip-scenario-include.md)]

以下的範例 PowerShell 命令會預期已根據上述案例建立簡單的環境。 如果您想要執行如本文件中所示的命令，請先建置[建立虛擬網路](quick-create-powershell.md)中所述的測試環境。

## <a name="create-a-vm-with-a-static-private-ip-address"></a>建立具有靜態私人 IP 位址的 VM
若要在名為 TestVNet 之 VNet 的FrontEnd子網路中建立名為 DNS01 且其靜態私人 IP 為 192.168.1.101 的 VM，請遵循下列步驟：

1. 針對儲存體帳戶、位置、 資源群組和要使用的認證設定變數。 您必須輸入 VM 的使用者名稱和密碼。 儲存體帳戶和資源群組必須已經存在。

    ```powershell
    $stName  = "vnetstorage"
    $locName = "Central US"
    $rgName  = "TestRG"
    $cred    = Get-Credential -Message "Type the name and password of the local administrator account."
    ```

2. 擷取虛擬網路以及您想要在其中建立 VM 的子網路。

    ```powershell
    $vnet   = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet
    $subnet = $vnet.Subnets[0].Id
    ```

3. 必要時，請建立公用 IP 位址從網際網路存取 VM。

    ```powershell
    $pip = New-AzureRmPublicIpAddress -Name TestPIP -ResourceGroupName $rgName `
    -Location $locName -AllocationMethod Dynamic
    ```

4. 使用您想要指派給 VM 的靜態私人 IP 位址建立 NIC。 請確定該 IP 來自您要新增 VM 的子網路範圍。 這是本文中將私人 IP 設定為靜態的主要步驟。

    ```powershell
    $nic = New-AzureRmNetworkInterface -Name TestNIC -ResourceGroupName $rgName `
    -Location $locName -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id `
    -PrivateIpAddress 192.168.1.101
    ```

5. 使用 NIC 建立 VM：

    ```powershell
    $vm = New-AzureRmVMConfig -VMName DNS01 -VMSize "Standard_A1"
    $vm = Set-AzureRmVMOperatingSystem -VM $vm -Windows -ComputerName DNS01 `
    -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
    $vm = Set-AzureRmVMSourceImage -VM $vm -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
    $vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id
    $osDiskUri = $storageAcc.PrimaryEndpoints.Blob.ToString() + "vhds/WindowsVMosDisk.vhd"
    $vm = Set-AzureRmVMOSDisk -VM $vm -Name "windowsvmosdisk" -VhdUri $osDiskUri `
    -CreateOption fromImage
    New-AzureRmVM -ResourceGroupName $rgName -Location $locName -VM $vm 
    ```

建議您不要靜態指派在 VM 作業系統內已指派給 Azure 虛擬機器的私人 IP，除非必要，例如[將多個 IP 位址指派給 Windows VM](virtual-network-multiple-ip-addresses-powershell.md) 時。 如果您確實手動設定作業系統內的私人 IP 位址，請確保它的位址與指派給 Azure [網路介面](virtual-network-network-interface-addresses.md#change-ip-address-settings)的私人 IP 位址相同，否則您可能會失去與虛擬機器的連線。 深入了解[私人 IP 位址](virtual-network-network-interface-addresses.md#private)設定。 請勿手動指派在虛擬機器作業系統內已指派給 Azure 虛擬機器的公用 IP 位址。

## <a name="retrieve-static-private-ip-address-information-for-a-network-interface"></a>擷取網路介面的靜態私人 IP 位址資訊
如果要檢視使用上述指令碼所建立之 VM 的靜態私人 IP 位址資訊，請執行下列 PowerShell 命令並查看 *PrivateIpAddress* 和 *PrivateIpAllocationMethod* 的值：

```powershell
Get-AzureRmNetworkInterface -Name TestNIC -ResourceGroupName TestRG
```

預期的輸出：

    Name                 : TestNIC
    ResourceGroupName    : TestRG
    Location             : centralus
    Id                   : /subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC
    Etag                 : W/"[Id]"
    ProvisioningState    : Succeeded
    Tags                 : 
    VirtualMachine       : {
                             "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/DNS01"
                           }
    IpConfigurations     : [
                             {
                               "Name": "ipconfig1",
                               "Etag": "W/\"[Id]\"",
                               "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC/ipConfigurations/ipconfig1",
                               "PrivateIpAddress": "192.168.1.101",
                               "PrivateIpAllocationMethod": "Static",
                               "Subnet": {
                                 "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd"
                               },
                               "PublicIpAddress": {
                                 "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/TestPIP"
                               },
                               "LoadBalancerBackendAddressPools": [],
                               "LoadBalancerInboundNatRules": [],
                               "ProvisioningState": "Succeeded"
                             }
                           ]
    DnsSettings          : {
                             "DnsServers": [],
                             "AppliedDnsServers": [],
                             "InternalDnsNameLabel": null,
                             "InternalFqdn": null
                           }
    EnableIPForwarding   : False
    NetworkSecurityGroup : null
    Primary              : True

## <a name="remove-a-static-private-ip-address-from-a-network-interface"></a>從網路介面移除靜態私人 IP 位址
若要移除上述指令碼中新增至 VM 的靜態私人 IP 位址，請執行下列 PowerShell 命令：

```powershell
$nic=Get-AzureRmNetworkInterface -Name TestNIC -ResourceGroupName TestRG
$nic.IpConfigurations[0].PrivateIpAllocationMethod = "Dynamic"
Set-AzureRmNetworkInterface -NetworkInterface $nic
```

預期的輸出：

    Name                 : TestNIC
    ResourceGroupName    : TestRG
    Location             : centralus
    Id                   : /subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC
    Etag                 : W/"[Id]"
    ProvisioningState    : Succeeded
    Tags                 : 
    VirtualMachine       : {
                             "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/WindowsVM"
                           }
    IpConfigurations     : [
                             {
                               "Name": "ipconfig1",
                               "Etag": "W/\"[Id]\"",
                               "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC/ipConfigurations/ipconfig1",
                               "PrivateIpAddress": "192.168.1.101",
                               "PrivateIpAllocationMethod": "Dynamic",
                               "Subnet": {
                                 "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd"
                               },
                               "PublicIpAddress": {
                                 "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/TestPIP"
                               },
                               "LoadBalancerBackendAddressPools": [],
                               "LoadBalancerInboundNatRules": [],
                               "ProvisioningState": "Succeeded"
                             }
                           ]
    DnsSettings          : {
                             "DnsServers": [],
                             "AppliedDnsServers": [],
                             "InternalDnsNameLabel": null,
                             "InternalFqdn": null
                           }
    EnableIPForwarding   : False
    NetworkSecurityGroup : null
    Primary              : True

## <a name="add-a-static-private-ip-address-to-a-network-interface"></a>將靜態私人 IP 位址新增至網路介面
若要將靜態私人 IP 位址新增至使用上述指令碼建立之 VM，請執行下列命令：

```powershell
$nic=Get-AzureRmNetworkInterface -Name TestNIC -ResourceGroupName TestRG
$nic.IpConfigurations[0].PrivateIpAllocationMethod = "Static"
$nic.IpConfigurations[0].PrivateIpAddress = "192.168.1.101"
Set-AzureRmNetworkInterface -NetworkInterface $nic
```

建議您不要靜態指派在 VM 作業系統內已指派給 Azure 虛擬機器的私人 IP，除非必要，例如[將多個 IP 位址指派給 Windows VM](virtual-network-multiple-ip-addresses-powershell.md) 時。 如果您確實手動設定作業系統內的私人 IP 位址，請確保它的位址與指派給 Azure [網路介面](virtual-network-network-interface-addresses.md#change-ip-address-settings)的私人 IP 位址相同，否則您可能會失去與虛擬機器的連線。 深入了解[私人 IP 位址](virtual-network-network-interface-addresses.md#private)設定。 請勿手動指派在虛擬機器作業系統內已指派給 Azure 虛擬機器的公用 IP 位址。

## <a name="change-the-allocation-method-for-a-private-ip-address-assigned-to-a-network-interface"></a>針對指派至網路介面的私人 IP 位址變更配置方法

私人 IP 位址是透過靜態或動態配置方法指派至 NIC。 啟動原先處於已停止 (已解除配置) 狀態的 VM 之後，動態 IP 位址可能變更。 即使 VM 從已停止 (已解除配置) 狀態重新啟動之後，如果 VM 裝載的服務需要相同的 IP 位址，這可能會造成問題。 靜態 IP 位址會一直保留，直到刪除 VM 為止。 若要變更 IP 位址的配置方法，請執行下列指令碼，將配置方法從動態變更為靜態。 如果目前私人 IP 位址的配置方法是靜態，請先將 *Static* 變更為 *Dynamic*，再執行指令碼。

```powershell
$RG = "TestRG"
$NIC_name = "testnic1"

$nic = Get-AzureRmNetworkInterface -ResourceGroupName $RG -Name $NIC_name
$nic.IpConfigurations[0].PrivateIpAllocationMethod = 'Static'
Set-AzureRmNetworkInterface -NetworkInterface $nic 
$IP = $nic.IpConfigurations[0].PrivateIpAddress

Write-Host "The allocation method is now set to"$nic.IpConfigurations[0].PrivateIpAllocationMethod"for the IP address" $IP"." -NoNewline
```

如果您不知道的 NIC 的名稱，您可以輸入下列命令，檢視資源群組內的 NIC 清單︰

```powershell
Get-AzureRmNetworkInterface -ResourceGroupName $RG | Where-Object {$_.ProvisioningState -eq 'Succeeded'} 
```

## <a name="next-steps"></a>後續步驟

了解如何管理 [IP 位址設定](virtual-network-network-interface-addresses.md)。