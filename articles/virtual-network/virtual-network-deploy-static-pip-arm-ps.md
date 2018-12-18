---
title: 建立具有靜態公用 IP 位址的 VM - Azure PowerShell | Microsoft Docs
description: 了解如何使用 PowerShell 建立具有靜態公用 IP 位址的 VM。
services: virtual-network
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: ad975ab9-d69f-45c1-9e45-0d3f0f51e87e
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/15/2016
ms.author: jdial
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 68656db0b76a29e7ab36fd6fa9ad4647712233ee
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38696578"
---
# <a name="create-a-vm-with-a-static-public-ip-address-using-powershell"></a>使用 PowerShell 建立具有靜態公用 IP 位址的 VM

> [!div class="op_single_selector"]
> * [Azure 入口網站](virtual-network-deploy-static-pip-arm-portal.md)
> * [PowerShell](virtual-network-deploy-static-pip-arm-ps.md)
> * [Azure CLI](virtual-network-deploy-static-pip-arm-cli.md)
> * [PowerShell (傳統)](virtual-networks-reserved-public-ip.md)

[!INCLUDE [virtual-network-deploy-static-pip-intro-include.md](../../includes/virtual-network-deploy-static-pip-intro-include.md)]

> [!NOTE]
> Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用 Resource Manager 部署模型，Microsoft 建議大部分的新部署使用此模型，而不是傳統部署模型。

[!INCLUDE [virtual-network-deploy-static-pip-scenario-include.md](../../includes/virtual-network-deploy-static-pip-scenario-include.md)]

[!INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## <a name="start-your-script"></a>啟動指令碼
[這裡](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/IaaS-Story/03-Static-public-IP/virtual-network-deploy-static-pip-arm-ps.ps1)可以下載所使用之完整的 PowerShell 指令碼。 請遵循下列步驟來變更指令碼來讓指令碼在環境中運作。

根據要用於部署的值來變更下列變數的值。 下列值對應至本文中使用的案例：

```powershell
# Set variables resource group
$rgName                = "IaaSStory"
$location              = "West US"

# Set variables for VNet
$vnetName              = "WTestVNet"
$vnetPrefix            = "192.168.0.0/16"
$subnetName            = "FrontEnd"
$subnetPrefix          = "192.168.1.0/24"

# Set variables for storage
$stdStorageAccountName = "iaasstorystorage"

# Set variables for VM
$vmSize                = "Standard_A1"
$diskSize              = 127
$publisher             = "MicrosoftWindowsServer"
$offer                 = "WindowsServer"
$sku                   = "2012-R2-Datacenter"
$version               = "latest"
$vmName                = "WEB1"
$osDiskName            = "osdisk"
$nicName               = "NICWEB1"
$privateIPAddress      = "192.168.1.101"
$pipName               = "PIPWEB1"
$dnsName               = "iaasstoryws1"
```

## <a name="create-the-necessary-resources-for-your-vm"></a>為 VM 建立必要的資源
建立 VM 之前，您需要供 VM 使用的資源群組、VNet、公用 IP 及 NIC。

1. 建立新的資源群組。

    ```powershell
    New-AzureRmResourceGroup -Name $rgName -Location $location
    ```

2. 建立 VNet 和子網路。

    ```powershell
    $vnet = New-AzureRmVirtualNetwork -ResourceGroupName $rgName -Name $vnetName `
        -AddressPrefix $vnetPrefix -Location $location

    Add-AzureRmVirtualNetworkSubnetConfig -Name $subnetName `
        -VirtualNetwork $vnet -AddressPrefix $subnetPrefix

    Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
    ```

3. 建立公用 IP 資源。 

    ```powershell
    $pip = New-AzureRmPublicIpAddress -Name $pipName -ResourceGroupName $rgName `
        -AllocationMethod Static -DomainNameLabel $dnsName -Location $location
    ```

4. 使用公用 IP 為上述建立之子網路中的 VM 建立網路介面 (NIC)。 請注意，第一個 Cmdlet 會從 Azure 擷取 VNet，這是必要的，因為已執行 `Set-AzureRmVirtualNetwork` 來變更現有的 VNet。

    ```powershell
    $vnet = Get-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName
    $subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name $subnetName
    $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName `
        -Subnet $subnet -Location $location -PrivateIpAddress $privateIPAddress `
        -PublicIpAddress $pip
    ```

5. 建立裝載 VM 作業系統磁碟機的儲存體帳戶。

    ```powershell
    $stdStorageAccount = New-AzureRmStorageAccount -Name $stdStorageAccountName `
    -ResourceGroupName $rgName -Type Standard_LRS -Location $location
    ```

## <a name="create-the-vm"></a>建立 VM
現在所有必要的資源已準備就緒，您可以建立新的 VM。

1. 為 VM 建立設定物件。

    ```powershell
    $vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize
    ```

2. 取得 VM 本機系統管理員帳戶的認證。

    ```powershell
    $cred = Get-Credential -Message "Type the name and password for the local administrator account."
    ```

3. 建立 VM 設定物件。

    ```powershell
    $vmConfig = Set-AzureRmVMOperatingSystem -VM $vmConfig -Windows -ComputerName $vmName `
        -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
    ```

4. 設定 VM 的作業系統映像。

    ```powershell
    $vmConfig = Set-AzureRmVMSourceImage -VM $vmConfig -PublisherName $publisher `
        -Offer $offer -Skus $sku -Version $version
    ```

5. 設定作業系統磁碟。

    ```powershell
    $osVhdUri = $stdStorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/" + $osDiskName + ".vhd"
    $vmConfig = Set-AzureRmVMOSDisk -VM $vmConfig -Name $osDiskName -VhdUri $osVhdUri -CreateOption fromImage
    ```

6. 將 NIC 新增至 VM。

    ```powershell
    $vmConfig = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic.Id -Primary
    ```

7. 建立 VM。

    ```powershell
    New-AzureRmVM -VM $vmConfig -ResourceGroupName $rgName -Location $location
    ```

8. 儲存指令碼檔案。

## <a name="run-the-script"></a>執行指令碼

進行任何必要的變更之後，執行先前的指令碼。 幾分鐘後，會建立虛擬機器。

## <a name="set-ip-addresses-within-the-operating-system"></a>設定作業系統內的 IP 位址

您絕不應手動指派在虛擬機器作業系統內已指派給 Azure 虛擬機器的公用 IP 位址。 建議您不要靜態指派在 VM 作業系統內已指派給 Azure 虛擬機器的私人 IP，除非必要，例如[將多個 IP 位址指派給 Windows VM](virtual-network-multiple-ip-addresses-powershell.md) 時。 如果您確實手動設定作業系統內的私人 IP 位址，請確保它的位址與指派給 Azure [網路介面](virtual-network-network-interface-addresses.md#change-ip-address-settings)的私人 IP 位址相同，否則您可能會失去與虛擬機器的連線。 深入了解[私人 IP 位址](virtual-network-network-interface-addresses.md#private)設定。

## <a name="next-steps"></a>後續步驟

任何網路流量均可流入和流出本文所建立的 VM。 您可以在網路安全性群組內定義輸入和輸出安全性規則，以限制網路介面和 (或) 子網路可以流入和流出的流量。 若要深入了解網路安全性群組，請參閱[網路安全性群組概觀](security-overview.md)。