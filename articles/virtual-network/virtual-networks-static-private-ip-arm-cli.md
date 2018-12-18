---
title: 設定 VM 的私人 IP 位址 - Azure CLI | Microsoft Docs
description: 了解如何使用 Azure 命令列介面 (CLI) 設定虛擬機器的私人 IP 位址。
services: virtual-network
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 40b03a1a-ea00-454c-b716-7574cea49ac0
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/16/2017
ms.author: jdial
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 0ffcf483b2852ec87c263573a97b4508d5b5d39c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46997466"
---
# <a name="configure-private-ip-addresses-for-a-virtual-machine-using-the-azure-cli"></a>使用 Azure CLI 設定虛擬機器的私人 IP 位址

[!INCLUDE [virtual-networks-static-private-ip-selectors-arm-include](../../includes/virtual-networks-static-private-ip-selectors-arm-include.md)]

[!INCLUDE [virtual-networks-static-private-ip-intro-include](../../includes/virtual-networks-static-private-ip-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

本文涵蓋之內容包括資源管理員部署模型。 您也可以 [管理傳統部署模型中的靜態私人 IP 位址](virtual-networks-static-private-ip-classic-cli.md)。

[!INCLUDE [virtual-networks-static-ip-scenario-include](../../includes/virtual-networks-static-ip-scenario-include.md)]

> [!NOTE]
> 下列範例 Azure CLI 命令預期現有的簡單環境。 如果您想要執行如本文件中所顯示的命令，請先建置 [建立 vnet](quick-create-cli.md)中所說明的測試環境。

## <a name="specify-a-static-private-ip-address-when-creating-a-vm"></a>建立 VM 時指定靜態私人 IP 位址

若要在名為 *TestVNet* 之 VNet 的 *FrontEnd* 子網路中建立名為 *DNS01* 的 VM，且其靜態私人 IP 為 *192.168.1.101*，請完成下列步驟：

1. 安裝並設定最新的 [Azure CLI](/cli/azure/install-azure-cli) (若您尚未這樣做)，並使用 [az login](/cli/azure/reference-index#az_login) 來登入 Azure 帳戶。

2. 使用 [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) 命令來建立 VM 的公用 IP。 輸出後顯示的清單可說明所使用的參數。

    > [!NOTE]
    > 視您的環境而定，您可能會想要或需要為此步驟和後續步驟中的引數使用不同的值。

    ```azurecli
    az network public-ip create \
    --name TestPIP \
    --resource-group TestRG \
    --location centralus \
    --allocation-method Static
    ```

    預期的輸出：

   ```json
   {
        "publicIp": {
            "idleTimeoutInMinutes": 4,
            "ipAddress": "52.176.43.167",
            "provisioningState": "Succeeded",
            "publicIPAllocationMethod": "Static",
            "resourceGuid": "79e8baa3-33ce-466a-846c-37af3c721ce1"
        }
    }
    ```

   * `--resource-group`︰要在其中建立公用 IP 之資源群組的名稱。
   * `--name`：公用 IP 的名稱。
   * `--location`：要在其中建立公用 IP 的 Azure 區域。

3. 執行 [az network nic create](/cli/azure/network/nic#az_network_nic_create) 命令以建立具有靜態私人 IP 的 NIC。 輸出後顯示的清單可說明所使用的參數。 
   
    ```azurecli
    az network nic create \
    --resource-group TestRG \
    --name TestNIC \
    --location centralus \
    --subnet FrontEnd \
    --private-ip-address 192.168.1.101 \
    --vnet-name TestVNet
    ```

    預期的輸出：
   
    ```json
    {
        "newNIC": {
            "dnsSettings": {
            "appliedDnsServers": [],
            "dnsServers": []
            },
            "enableIPForwarding": false,
            "ipConfigurations": [
            {
                "etag": "W/\"<guid>\"",
                "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC/ipConfigurations/ipconfig1",
                "name": "ipconfig1",
                "properties": {
                "primary": true,
                "privateIPAddress": "192.168.1.101",
                "privateIPAllocationMethod": "Static",
                "provisioningState": "Succeeded",
                "subnet": {
                    "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                    "resourceGroup": "TestRG"
                }
                },
                "resourceGroup": "TestRG"
            }
            ],
            "provisioningState": "Succeeded",
            "resourceGuid": "<guid>"
        }
    }
    ```
    
    參數：

    * `--private-ip-address`：NIC 的靜態私人 IP 位址。
    * `--vnet-name`：要在其中建立 NIC 之 VNet 的名稱。
    * `--subnet`：要在其中建立 NIC 之子網路的名稱。

4. 執行 [azure vm create](/cli/azure/vm/nic#az_vm_nic_create) 命令以使用之前建立的公用 IP 和 NIC 來建立 VM。 輸出後顯示的清單可說明所使用的參數。
   
    ```azurecli
    az vm create \
    --resource-group TestRG \
    --name DNS01 \
    --location centralus \
    --image Debian \
    --admin-username adminuser \
    --ssh-key-value ~/.ssh/id_rsa.pub \
    --nics TestNIC
    ```

    預期的輸出：
   
    ```json
    {
        "fqdns": "",
        "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/DNS01",
        "location": "centralus",
        "macAddress": "00-0D-3A-92-C1-66",
        "powerState": "VM running",
        "privateIpAddress": "192.168.1.101",
        "publicIpAddress": "",
        "resourceGroup": "TestRG"
    }
    ```
   
   基本 [az vm create](/cli/azure/vm#az_vm_create) 參數以外的參數。

   * `--nics`︰VM 所連接至之 NIC 的名稱。
   
建議您不要靜態指派在 VM 作業系統內已指派給 Azure 虛擬機器的私人 IP，除非必要，例如[將多個 IP 位址指派給 Windows VM](virtual-network-multiple-ip-addresses-cli.md) 時。 如果您確實手動設定作業系統內的私人 IP 位址，請確保它的位址與指派給 Azure [網路介面](virtual-network-network-interface-addresses.md#change-ip-address-settings)的私人 IP 位址相同，否則您可能會失去與虛擬機器的連線。 深入了解[私人 IP 位址](virtual-network-network-interface-addresses.md#private)設定。

## <a name="retrieve-static-private-ip-address-information-for-a-vm"></a>擷取 VM 的靜態私人 IP 位址資訊

執行下列 Azure CLI 命令，以觀察私人 IP 配置方法和私人 IP 位址的值：

```azurecli
az vm show -g TestRG -n DNS01 --show-details --query 'privateIps'
```

預期的輸出：

```json
"192.168.1.101"
```

若要顯示該 VM 之 NIC 的特定 IP 資訊，請查詢 NIC，特別是：

```azurecli
az network nic show \
-g testrg \
-n testnic \
--query 'ipConfigurations[0].{PrivateAddress:privateIpAddress,IPVer:privateIpAddressVersion,IpAllocMethod:p
rivateIpAllocationMethod,PublicAddress:publicIpAddress}'
```

輸出會像這樣：

```json
{
    "IPVer": "IPv4",
    "IpAllocMethod": "Static",
    "PrivateAddress": "192.168.1.101",
    "PublicAddress": null
}
```

## <a name="remove-a-static-private-ip-address-from-a-vm"></a>移除 VM 的靜態私人 IP 位址

您無法從 Azure CLI 的 NIC 移除 Azure Resource Manager 部署的靜態私人 IP 位址。 您必須：
- 建立使用動態 IP 的新 NIC
- 在新建立 NIC 的 VM 上設定 NIC。 

若要變更先前命令中使用之 VM 的 NIC，請完成下列步驟：

1. 執行 **azure network nic create** 命令來建立使用新 IP 位址動態配置 IP 的新 NIC。 因為未指定任何 IP 位址，配置方法為**動態**。

    ```azurecli
    az network nic create     \
    --resource-group TestRG     \
    --name TestNIC2     \
    --location centralus     \
    --subnet FrontEnd    \
    --vnet-name TestVNet
    ```        
   
    預期的輸出：

    ```json
    {
        "newNIC": {
            "dnsSettings": {
            "appliedDnsServers": [],
            "dnsServers": []
            },
            "enableIPForwarding": false,
            "ipConfigurations": [
            {
                "etag": "W/\"<guid>\"",
                "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC2/ipConfigurations/ipconfig1",
                "name": "ipconfig1",
                "properties": {
                "primary": true,
                "privateIPAddress": "192.168.1.4",
                "privateIPAllocationMethod": "Dynamic",
                "provisioningState": "Succeeded",
                "subnet": {
                    "id": "/subscriptions/<guid>/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                    "resourceGroup": "TestRG"
                }
                },
                "resourceGroup": "TestRG"
            }
            ],
            "provisioningState": "Succeeded",
            "resourceGuid": "0808a61c-476f-4d08-98ee-0fa83671b010"
        }
    }
    ```

2. 執行 **azure vm set** 命令來變更 VM 所使用的 NIC。
   
    ```azurecli
    azure vm set -g TestRG -n DNS01 -N TestNIC2
    ```

    預期的輸出：
   
    ```json
    [
        {
            "id": "/subscriptions/0e220bf6-5caa-4e9f-8383-51f16b6c109f/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/TestNIC3",
            "primary": true,
            "resourceGroup": "TestRG"
        }
    ]
    ```

    > [!NOTE]
    > 如果 VM 夠大而可容納多個 NIC，請執行 **azure network nic delete** 命令，以刪除舊 NIC。

## <a name="next-steps"></a>後續步驟

了解如何管理 [IP 位址設定](virtual-network-network-interface-addresses.md)。