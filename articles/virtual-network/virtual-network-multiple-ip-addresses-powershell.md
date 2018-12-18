---
title: Azure 虛擬機器的多個 IP 位址 - PowerShell | Microsoft Docs
description: 了解如何使用 PowerShell 對虛擬機器指派多個 IP 位址 | Resource Manager.
services: virtual-network
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: c44ea62f-7e54-4e3b-81ef-0b132111f1f8
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/24/2017
ms.author: jdial;annahar
ms.openlocfilehash: 4c74833933642ec67bdd2a77d073b083d54a3038
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38678607"
---
# <a name="assign-multiple-ip-addresses-to-virtual-machines-using-powershell"></a>使用 PowerShell 對虛擬機器指派多個 IP 位址

[!INCLUDE [virtual-network-multiple-ip-addresses-intro.md](../../includes/virtual-network-multiple-ip-addresses-intro.md)]

本文說明如何使用 PowerShell 透過 Azure Resource Manager 部署模型建立虛擬機器 (VM)。 無法將多個 IP 位址指派給透過傳統部署模型建立的資源。 若要深入了解 Azure 部署模型，請參閱[了解部署模型](../resource-manager-deployment-model.md)文章。

[!INCLUDE [virtual-network-multiple-ip-addresses-scenario.md](../../includes/virtual-network-multiple-ip-addresses-scenario.md)]

## <a name = "create"></a>建立有多個 IP 位址的 VM

後續步驟說明如何使用多個 IP 位址建立範例 VM，如案例中所述。 請依據您的實作需求來變更變數值。

1. 開啟 PowerShell 命令提示字元，在單一 PowerShell 工作階段內完成本節中其餘的步驟。 如果您尚未安裝和設定 PowerShell，請先完成 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 文章中的步驟。
2. 使用 `Connect-AzureRmAccount` 命令來登入您的帳戶。
3. 使用您選擇的名稱和位置來取代 *myResourceGroup* 和 *westus*。 建立資源群組。 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

    ```powershell
    $RgName   = "MyResourceGroup"
    $Location = "westus"

    New-AzureRmResourceGroup `
    -Name $RgName `
    -Location $Location
    ```

4. 在與資源群組相同的位置中，建立虛擬網路 (VNet) 和子網路：

    ```powershell
    
    # Create a subnet configuration
    $SubnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
    -Name MySubnet `
    -AddressPrefix 10.0.0.0/24

    # Create a virtual network
    $VNet = New-AzureRmVirtualNetwork `
    -ResourceGroupName $RgName `
    -Location $Location `
    -Name MyVNet `
    -AddressPrefix 10.0.0.0/16 `
    -Subnet $subnetConfig

    # Get the subnet object
    $Subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name $SubnetConfig.Name -VirtualNetwork $VNet
    ```

5. 建立網路安全性群組 (NSG) 和規則。 NSG 會使用輸入和輸出規則來保護 VM。 在此情況下，會建立連接埠 3389 的輸入規則，以允許連入的遠端桌面連線。

    ```powershell
    
    # Create an inbound network security group rule for port 3389

    $NSGRule = New-AzureRmNetworkSecurityRuleConfig `
    -Name MyNsgRuleRDP `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 1000 `
    -SourceAddressPrefix * `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389 -Access Allow
    
    # Create a network security group
    $NSG = New-AzureRmNetworkSecurityGroup `
    -ResourceGroupName $RgName `
    -Location $Location `
    -Name MyNetworkSecurityGroup `
    -SecurityRules $NSGRule
    ```

6. 定義 NIC 的主要 IP 組態。 如果您未使用先前定義的值，請將 10.0.0.4 變更成您所建立之子網路中的有效位址。 指派靜態 IP 位址之前，建議您先確認該位址尚未處於使用中。 輸入命令 `Test-AzureRmPrivateIPAddressAvailability -IPAddress 10.0.0.4 -VirtualNetwork $VNet`。 如果位址可用，則輸出會傳回 *True*。 如果無法使用，則輸出會傳回 *False* 和可用位址的清單。 

    在下列命令中，**以要使用的唯一 DNS 名稱取代 <replace-with-your-unique-name>**。 該名稱在 Azure 區域內的所有公用 IP 位址中必須是唯一的。 這是選擇性參數。 如果您只想要使用公用 IP 位址來連接到 VM，則可以將它移除。

    ```powershell
    
    # Create a public IP address
    $PublicIP1 = New-AzureRmPublicIpAddress `
    -Name "MyPublicIP1" `
    -ResourceGroupName $RgName `
    -Location $Location `
    -DomainNameLabel <replace-with-your-unique-name> `
    -AllocationMethod Static
        
    #Create an IP configuration with a static private IP address and assign the public IP ddress to it
    $IpConfigName1 = "IPConfig-1"
    $IpConfig1     = New-AzureRmNetworkInterfaceIpConfig `
    -Name $IpConfigName1 `
    -Subnet $Subnet `
    -PrivateIpAddress 10.0.0.4 `
    -PublicIpAddress $PublicIP1 `
    -Primary
    ```

    當您指派多個 IP 組態給 NIC 時，必須將一個組態指派為 *-Primary*。

    > [!NOTE]
    > 公用 IP 位址需要少許費用。 若要深入了解 IP 位址定價，請閱讀 [IP 位址定價](https://azure.microsoft.com/pricing/details/ip-addresses) 頁面。 訂用帳戶中可使用的公用 IP 位址數目有限制。 若要深入了解限制，請參閱 [Azure 限制](../azure-subscription-service-limits.md#networking-limits)文章。

7. 為 NIC 定義次要 IP 組態。 您可以視需要新增或移除組態。 每個 IP 組態都必須有一個指派的私人 IP 位址。 每個組態可以視需要有一個指派的公用 IP 位址。

    ```powershell
    
    # Create a public IP address
    $PublicIP2 = New-AzureRmPublicIpAddress `
    -Name "MyPublicIP2" `
    -ResourceGroupName $RgName `
    -Location $Location `
    -AllocationMethod Static
        
    #Create an IP configuration with a static private IP address and assign the public IP ddress to it
    $IpConfigName2 = "IPConfig-2"
    $IpConfig2     = New-AzureRmNetworkInterfaceIpConfig `
    -Name $IpConfigName2 `
    -Subnet $Subnet `
    -PrivateIpAddress 10.0.0.5 `
    -PublicIpAddress $PublicIP2
        
    $IpConfigName3 = "IpConfig-3"
    $IpConfig3 = New-AzureRmNetworkInterfaceIpConfig `
    -Name $IPConfigName3 `
    -Subnet $Subnet `
    -PrivateIpAddress 10.0.0.6
    ```

8. 建立 NIC 並將三個 IP 組態指派給它：

    ```powershell
    
    $NIC = New-AzureRmNetworkInterface `
    -Name MyNIC `
    -ResourceGroupName $RgName `
    -Location $Location `
    -NetworkSecurityGroupId $NSG.Id `
    -IpConfiguration $IpConfig1,$IpConfig2,$IpConfig3
    ```

    >[!NOTE]
    >雖然在本文中是將所有組態都指派給一個 NIC，但您也可以指派多個 IP 組態給連結到 VM 的每個 NIC。 閱讀[建立具有多個 NIC 的 VM](../virtual-machines/windows/multiple-nics.md) 文章以了解如何建立具有多個 NIC 的 VM。

9. 輸入下列命令來建立 VM：

    ```powershell
    
    # Define a credential object. When you run these commands, you're prompted to enter a sername and password for the VM you're reating.
    $cred = Get-Credential
    
    # Create a virtual machine configuration
    $VmConfig = New-AzureRmVMConfig `
    -VMName MyVM `
    -VMSize Standard_DS1_v2 | `
    Set-AzureRmVMOperatingSystem -Windows `
    -ComputerName MyVM `
    -Credential $cred | `
    Set-AzureRmVMSourceImage `
    -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer `
    -Skus 2016-Datacenter `
    -Version latest | `
    Add-AzureRmVMNetworkInterface `
    -Id $NIC.Id
    
    # Create the VM
    New-AzureRmVM `
    -ResourceGroupName $RgName `
    -Location $Location `
    -VM $VmConfig
    ```

10. 完成本文的[將 IP 位址新增至 VM 作業系統](#os-config)一節中適用於您的作業系統的步驟，將私人 IP 位址新增至 VM 作業系統。 請勿將公用 IP 位址新增至作業系統。

## <a name="add"></a>將 IP 位址新增至 VM

您可以完成後續步驟，將私人和公用 IP 位址新增至 Azure 網路介面。 下列各節中的範例假設您已經有一個 VM，其具有本文中的[案例](#Scenario)所述的三項 IP 組態，您不需要進行設定。

1. 開啟 PowerShell 命令提示字元，在單一 PowerShell 工作階段內完成本節中其餘的步驟。 如果您尚未安裝和設定 PowerShell，請先完成 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 文章中的步驟。
2. 將下列 $Variables 的「值」變更為您想要對其新增 IP 位址的 NIC 名稱，以及 NIC 所在的資源群組和位置︰

    ```powershell
    $NicName  = "MyNIC"
    $RgName   = "MyResourceGroup"
    $Location = "westus"
    ```

    如果您不知道要變更的 NIC 名稱，請輸入下列命令，然後變更先前變數的值︰

    ```powershell
    Get-AzureRmNetworkInterface | Format-Table Name, ResourceGroupName, Location
    ```
3. 輸入下列命令建立變數，並將它設定為現有 NIC︰

    ```powershell
    $MyNIC = Get-AzureRmNetworkInterface -Name $NicName -ResourceGroupName $RgName
    ```
4. 在下列命令中，將 *MyVNet* 和 *MySubnet* 變更成 NIC 所連線的 VNet 和子網路的名稱。 輸入命令來擷取 NIC 所連線的 VNet 和子網路物件︰

    ```powershell
    $MyVNet = Get-AzureRMVirtualnetwork -Name MyVNet -ResourceGroupName $RgName
    $Subnet = $MyVnet.Subnets | Where-Object { $_.Name -eq "MySubnet" }
    ```
    如果您不知道 NIC 所連接的 VNet 或子網路名稱，輸入下列命令︰
    ```powershell
    $MyNIC.IpConfigurations
    ```
    在輸出中，尋找與下列範例輸出類似的文字︰
    
    ```
    "Id": "/subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVNet/subnets/MySubnet"
    ```
    在此輸出中，*MyVnet* 是 VNet 而 *MySubnet* 是 NIC 所連線的子網路。

5. 根據您的需求，完成下列其中一個章節中的步驟︰

    **新增私人 IP 位址**

    若要將私人 IP 位址新增至 NIC，您必須建立 IP 組態。 下列命令會建立具有靜態 IP 位址 10.0.0.7 的組態。 指定靜態 IP 位址時，它必須是子網路未使用的位址。 建議您先輸入 `Test-AzureRmPrivateIPAddressAvailability -IPAddress 10.0.0.7 -VirtualNetwork $myVnet` 命令來測試地址，確定它為可用。 如果 IP 位址可用，則輸出會傳回 *True*。 如果無法使用，則輸出會傳回 False 和可用位址的清單。

    ```powershell
    Add-AzureRmNetworkInterfaceIpConfig -Name IPConfig-4 -NetworkInterface `
    $MyNIC -Subnet $Subnet -PrivateIpAddress 10.0.0.7
    ```
    使用唯一組態名稱和私人 IP 位址 (適用於具有靜態 IP 位址的組態)，視需要建立最多的組態。

    完成本文的[將 IP 位址新增至 VM 作業系統](#os-config)一節中適用於您的作業系統的步驟，將私人 IP 位址新增至 VM 作業系統。

    **新增公用 IP 位址**

    將公用 IP 位址與新的 IP 組態或現有的 IP 組態產生關聯，便可新增公用 IP 位址。 視需要完成後續其中一節的步驟。

    > [!NOTE]
    > 公用 IP 位址需要少許費用。 若要深入了解 IP 位址定價，請閱讀 [IP 位址定價](https://azure.microsoft.com/pricing/details/ip-addresses) 頁面。 訂用帳戶中可使用的公用 IP 位址數目有限制。 若要深入了解限制，請參閱 [Azure 限制](../azure-subscription-service-limits.md#networking-limits)文章。
    >

    - **將公用 IP 位址資源與新的 IP 組態產生關聯**
    
        每當您在新的 IP 組態中新增公用 IP 位址時，也必須新增私人 IP 位址，因為所有的 IP 組態都必須有一個私人 IP 位址。 您可以新增現有的公用 IP 位址資源，或建立一個新的資源。 若要建立新的公用 IP 位址資源，請輸入下列命令：
    
        ```powershell
        $myPublicIp3 = New-AzureRmPublicIpAddress `
        -Name "myPublicIp3" `
        -ResourceGroupName $RgName `
        -Location $Location `
        -AllocationMethod Static
        ```

        若要建立具有靜態私人 IP 位址和相關 *myPublicIp3* 公用 IP 位址資源的新 IP 組態，請輸入下列命令︰

        ```powershell
        Add-AzureRmNetworkInterfaceIpConfig `
        -Name IPConfig-4 `
        -NetworkInterface $myNIC `
        -Subnet $Subnet `
        -PrivateIpAddress 10.0.0.7 `
        -PublicIpAddress $myPublicIp3
        ```

    - **將公用 IP 位址資源與現有的 IP 組態產生關聯**

        公用 IP 位址資源只能與尚未關聯的 IP 組態產生關聯。 您可以輸入下列命令，判斷 IP組態是否有相關聯的公用 IP 位址︰

        ```powershell
        $MyNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary
        ```

        您會看到類似以下的輸出：

        ```     
        Name       PrivateIpAddress PublicIpAddress                                           Primary
        
        IPConfig-1 10.0.0.4         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress    True
        IPConfig-2 10.0.0.5         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress   False
        IpConfig-3 10.0.0.6                                                                     False
        ```

        由於 IpConfig 3 的 **PublicIpAddress** 欄是空白，目前沒有與其相關聯的公用 IP 位址資源。 您可以將現有的公用 IP 位址資源新增至 IpConfig-3，或輸入下列命令以建立一個︰

        ```powershell
        $MyPublicIp3 = New-AzureRmPublicIpAddress `
        -Name "MyPublicIp3" `
        -ResourceGroupName $RgName `
        -Location $Location -AllocationMethod Static
        ```

        輸入下列命令，將公用 IP 位址資源與名為 *IpConfig-3* 的現有 IP 組態產生關聯：
    
        ```powershell
        Set-AzureRmNetworkInterfaceIpConfig `
        -Name IpConfig-3 `
        -NetworkInterface $mynic `
        -Subnet $Subnet `
        -PublicIpAddress $myPublicIp3
        ```

6. 輸入下列命令，可設定具有新 IP 組態的 NIC︰

    ```powershell
    Set-AzureRmNetworkInterface -NetworkInterface $MyNIC
    ```

7. 輸入下列命令，以檢視指派給 NIC 的私人 IP 位址和公用 IP 位址資源︰

    ```powershell   
    $MyNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary
    ```
8. 完成本文的[將 IP 位址新增至 VM 作業系統](#os-config)一節中適用於您的作業系統的步驟，將私人 IP 位址新增至 VM 作業系統。 請勿將公用 IP 位址新增至作業系統。

[!INCLUDE [virtual-network-multiple-ip-addresses-os-config.md](../../includes/virtual-network-multiple-ip-addresses-os-config.md)]
