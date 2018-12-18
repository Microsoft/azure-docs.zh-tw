---
title: 使用 Azure CLI 建立具有多個 IP 位址的 VM | Microsoft Docs
description: 了解如何使用 Azure 命令列介面 (CLI) 將多個 IP 位址指派給虛擬機器。
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/17/2016
ms.author: jimdial
ms.openlocfilehash: bd8a9e651c3b2369ae29be966ba4c9cbe0ae579b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46972907"
---
# <a name="assign-multiple-ip-addresses-to-virtual-machines-using-the-azure-cli"></a>使用 Azure CLI 對虛擬機器指派多個 IP 位址

[!INCLUDE [virtual-network-multiple-ip-addresses-intro.md](../../includes/virtual-network-multiple-ip-addresses-intro.md)]

本文說明如何使用 Azure CLI 透過 Azure Resource Manager 部署模型建立虛擬機器 (VM)。 無法將多個 IP 位址指派給透過傳統部署模型建立的資源。 若要深入了解 Azure 部署模型，請參閱[了解部署模型](../resource-manager-deployment-model.md)文章。

[!INCLUDE [virtual-network-multiple-ip-addresses-scenario.md](../../includes/virtual-network-multiple-ip-addresses-scenario.md)]

## <a name = "create"></a>建立有多個 IP 位址的 VM

後續步驟說明如何建立具有多個 IP 位址的範例虛擬機器，如案例中所述。 視您的實作而定，變更 "" 中的變數值和 IP 位址類型。 

1. 如果尚未安裝 [Azure CLI](/cli/azure/install-azure-cli)，請先安裝它。
2. 完成[建立 Linux VM 的 SSH 公用和私用金鑰組](../virtual-machines/linux/mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-network%2ftoc.json)中的步驟，為 Linux VM 建立 SSH 公用和私用金鑰組。
3. 從命令殼層使用命令 `az login` 進行登入，然後選取您使用的訂用帳戶。
4. 在 Linux 或 Mac 電腦上執行後續的指令碼以建立 VM。 該指令碼會建立資源群組、一個虛擬網路 (VNet)、一個具有三個 IP 組態的 NIC，以及連接了兩個 NIC 的 VM。 NIC、公用 IP 位址、虛擬網路和 VM 資源必須全都位於相同的位置和訂用帳戶。 雖然資源不需要全都位於相同的資源群組中，但在下列指令碼中，它們卻是如此。

```bash
    
#!/bin/sh
    
RgName="myResourceGroup"
Location="westcentralus"
az group create --name $RgName --location $Location
    
# Create a public IP address resource with a static IP address using the `--allocation-method Static` option. If you
# do not specify this option, the address is allocated dynamically. The address is assigned to the resource from a pool
# of IP adresses unique to each Azure region. Download and view the file from
# https://www.microsoft.com/en-us/download/details.aspx?id=41653 that lists the ranges for each region.

PipName="myPublicIP"

# This name must be unique within an Azure location.
DnsName="myDNSName"

az network public-ip create \
--name $PipName \
--resource-group $RgName \
--location $Location \
--dns-name $DnsName\
--allocation-method Static

# Create a virtual network with one subnet

VnetName="myVnet"
VnetPrefix="10.0.0.0/16"
VnetSubnetName="mySubnet"
VnetSubnetPrefix="10.0.0.0/24"

az network vnet create \
--name $VnetName \
--resource-group $RgName \
--location $Location \
--address-prefix $VnetPrefix \
--subnet-name $VnetSubnetName \
--subnet-prefix $VnetSubnetPrefix

# Create a network interface connected to the subnet and associate the public IP address to it. Azure will create the
# first IP configuration with a static private IP address and will associate the public IP address resource to it.

NicName="MyNic1"
az network nic create \
--name $NicName \
--resource-group $RgName \
--location $Location \
--subnet $VnetSubnet1Name \
--private-ip-address 10.0.0.4
--vnet-name $VnetName \
--public-ip-address $PipName
    
# Create a second public IP address, a second IP configuration, and associate it to the NIC. This configuration has a
# static public IP address and a static private IP address.

az network public-ip create \
--resource-group $RgName \
--location $Location \
--name myPublicIP2 \
--dns-name mypublicdns2 \
--allocation-method Static

az network nic ip-config create \
--resource-group $RgName \
--nic-name $NicName \
--name IPConfig-2 \
--private-ip-address 10.0.0.5 \
--public-ip-name myPublicIP2

# Create a third IP configuration, and associate it to the NIC. This configuration has  static private IP address and   # no public IP address.

azure network nic ip-config create \
--resource-group $RgName \
--nic-name $NicName \
--private-ip-address 10.0.0.6 \
--name IPConfig-3

# Note: Though this article assigns all IP configurations to a single NIC, you can also assign multiple IP configurations
# to any NIC in a VM. To learn how to create a VM with multiple NICs, read the Create a VM with multiple NICs 
# article: https://docs.microsoft.com/azure/virtual-network/virtual-network-deploy-multinic-arm-cli.

# Create a VM and attach the NIC.

VmName="myVm"

# Replace the value for the following **VmSize** variable with a value from the
# https://docs.microsoft.com/azure/virtual-machines/virtual-machines-linux-sizes rticle. The script fails if the VM size
# is not supported in the location you select. Run the `azure vm sizes --location estcentralus` command to get a full list
# of VMs in US West Central, for example.

VmSize="Standard_DS1"

# Replace the value for the OsImage variable value with a value for *urn* from the utput returned by entering the
# `az vm image list` command.

OsImage="credativ:Debian:8:latest"

Username="adminuser"

# Replace the following value with the path to your public key file. If you're creating a Windows VM, remove the following
# line and you'll be prompted for the password you want to configure for the VM.

SshKeyValue="~/.ssh/id_rsa.pub"

az vm create \
--name $VmName \
--resource-group $RgName \
--image $OsImage \
--location $Location \
--size $VmSize \
--nics $NicName \
--admin-username $Username \
--ssh-key-value $SshKeyValue
```

除了建立具有 NIC (其中含有 3 個 IP 組態) 的 VM 之外，該指令碼還會建立：

- 單一的進階受控磁碟 (預設)，但有其他選項可讓您選擇可以建立的磁碟類型。 如需詳細資訊，請閱讀[使用 Azure CLI 建立 Linux VM](../virtual-machines/linux/quick-create-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 一文。
- 具有一個子網路和兩個公用 IP 位址的虛擬網路。 或者，您可以使用「現有」虛擬網路、子網路、NIC 或公用 IP 位址資源。 若要了解如何使用現有網路資源，而不是另外建立資源，請輸入 `az vm create -h`。

公用 IP 位址需要少許費用。 若要深入了解 IP 位址定價，請閱讀 [IP 位址定價](https://azure.microsoft.com/pricing/details/ip-addresses) 頁面。 訂用帳戶中可使用的公用 IP 位址數目有限制。 若要深入了解限制，請參閱 [Azure 限制](../azure-subscription-service-limits.md#networking-limits)文章。

在 VM 建立後，請輸入 `az network nic show --name MyNic1 --resource-group myResourceGroup` 命令來檢視 NIC 組態。 輸入 `az network nic ip-config list --nic-name MyNic1 --resource-group myResourceGroup --output table` 以檢視與 NIC 相關聯之 IP 組態的清單。

完成本文的[將 IP 位址新增至 VM 作業系統](#os-config)一節中適用於您的作業系統的步驟，將私人 IP 位址新增至 VM 作業系統。

## <a name="add"></a>將 IP 位址新增至 VM

您可以完成後續步驟，將其他私人和公用 IP 位址新增至現有的 Azure 網路介面。 範例以本文章所述的[案例](#Scenario)為基礎。

1. 開啟命令殼層，並在單一工作階段內完成本章節的其餘步驟。 如果您尚未安裝並設定 Azure CLI，請完成 [Azure CLI 安裝](/cli/azure/install-az-cli2?toc=%2fazure%2fvirtual-network%2ftoc.json)一文中的步驟，並使用 `az-login` 命令登入您的 Azure 帳戶。

2. 根據您的需求，完成下列其中一個章節中的步驟︰

    **新增私人 IP 位址**
    
    若要將私人 IP 位址新增至 NIC，您必須使用隨後的命令建立 IP 組態。 靜態 IP 位址必須是未使用的子網路位址。

    ```bash
    az network nic ip-config create \
    --resource-group myResourceGroup \
    --nic-name myNic1 \
    --private-ip-address 10.0.0.7 \
    --name IPConfig-4
    ```
    
    使用唯一組態名稱和私人 IP 位址 (適用於具有靜態 IP 位址的組態)，視需要建立最多的組態。

    **新增公用 IP 位址**
    
    新增公用 IP 位址的方法是將它與新的 IP 組態或現有的 IP 組態產生關聯。 視需要完成後續其中一節的步驟。

    公用 IP 位址需要少許費用。 若要深入了解 IP 位址定價，請閱讀 [IP 位址定價](https://azure.microsoft.com/pricing/details/ip-addresses) 頁面。 訂用帳戶中可使用的公用 IP 位址數目有限制。 若要深入了解限制，請參閱 [Azure 限制](../azure-subscription-service-limits.md#networking-limits)文章。

    - **將資源與新的 IP 組態產生關聯**
    
        每當您在新的 IP 組態中新增公用 IP 位址時，也必須新增私人 IP 位址，因為所有的 IP 組態都必須有一個私人 IP 位址。 您可以新增現有的公用 IP 位址資源，或建立一個新的資源。 若要建立新的公用 IP 位址資源，請輸入下列命令：
    
        ```bash
        az network public-ip create \
        --resource-group myResourceGroup \
        --location westcentralus \
        --name myPublicIP3 \
        --dns-name mypublicdns3
        ```

        若要建立具有靜態私人 IP 位址和相關 *myPublicIP3* 公用 IP 位址資源的新 IP 組態，請輸入下列命令︰

        ```bash
        az network nic ip-config create \
        --resource-group myResourceGroup \
        --nic-name myNic1 \
        --name IPConfig-5 \
        --private-ip-address 10.0.0.8
        --public-ip-address myPublicIP3
        ```

    - **將資源與現有的 IP 組態產生關聯** 公用 IP 位址資源只能與尚未相關聯的 IP 組態產生關聯。 您可以輸入下列命令，判斷 IP組態是否有相關聯的公用 IP 位址︰

        ```bash
        az network nic ip-config list \
        --resource-group myResourceGroup \
        --nic-name myNic1 \
        --query "[?provisioningState=='Succeeded'].{ Name: name, PublicIpAddressId: publicIpAddress.id }" --output table
        ```

        傳回的輸出︰
    
            Name        PublicIpAddressId
            
            ipconfig1   /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP1
            IPConfig-2  /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP2
            IPConfig-3

        由於 IpConfig-3 的 **PublicIpAddress** 欄是空白的，目前沒有與其相關聯的公用 IP 位址資源。 您可以將現有的公用 IP 位址資源新增至 IpConfig-3，或輸入下列命令以建立一個︰

        ```bash
        az network public-ip create \
        --resource-group  myResourceGroup
        --location westcentralus \
        --name myPublicIP3 \
        --dns-name mypublicdns3 \
        --allocation-method Static
        ```
    
        輸入下列命令，將公用 IP 位址資源與名為 *IpConfig-3* 的現有 IP 組態產生關聯：
    
        ```bash
        az network nic ip-config update \
        --resource-group myResourceGroup \
        --nic-name myNic1 \
        --name IPConfig-3 \
        --public-ip myPublicIP3
        ```

3. 輸入下列命令，以檢視指派給 NIC 的私人 IP 位址和公用 IP 位址資源識別碼︰

    ```bash
    az network nic ip-config list \
    --resource-group myResourceGroup \
    --nic-name myNic1 \
    --query "[?provisioningState=='Succeeded'].{ Name: name, PrivateIpAddress: privateIpAddress, PrivateIpAllocationMethod: privateIpAllocationMethod, PublicIpAddressId: publicIpAddress.id }" --output table
    ```

    傳回的輸出︰ <br>
    
        Name        PrivateIpAddress    PrivateIpAllocationMethod   PublicIpAddressId
        
        ipconfig1   10.0.0.4            Static                      /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP1
        IPConfig-2  10.0.0.5            Static                      /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP2
        IPConfig-3  10.0.0.6            Static                      /subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP3
    

4. 依照本文的[將 IP 位址新增至 VM 作業系統](#os-config)一節中的指示，將您新增至 NIC 的私人 IP 位址新增至 VM 作業系統。 請勿將公用 IP 位址新增至作業系統。

[!INCLUDE [virtual-network-multiple-ip-addresses-os-config.md](../../includes/virtual-network-multiple-ip-addresses-os-config.md)]
