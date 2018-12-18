---
title: 使用 Azure CLI 建立 Linux 環境 | Microsoft Docs
description: 使用 Azure CLI 從頭開始建立儲存體、Linux VM、虛擬網路和子網路、負載平衡器、NIC、公用 IP 以及網路安全性群組。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 4ba4060b-ce95-4747-a735-1d7c68597a1a
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/14/2017
ms.author: cynthn
ms.openlocfilehash: e618bce78a3a3ce080b6c24cc34a27c2f5fb39e9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46977548"
---
# <a name="create-a-complete-linux-virtual-machine-with-the-azure-cli"></a>使用 Azure CLI 來建立完整的 Linux 虛擬機器
若要在 Azure 中快速建立虛擬機器 (VM)，您可以使用單一的 Azure CLI 命令，此命令會使用預設值來建立任何必要的支援資源。 系統會自動建立虛擬網路、公用 IP 位址及網路安全性群組規則等資源。 若要在生產環境使用案例中對您的環境進行更多控制，您可以預先建立這些資源，然後再將 VM 新增到這些資源中。 本文將引導您了解如何建立 VM 及逐一建立每個支援資源。

請確定您已安裝最新的 [Azure CLI](/cli/azure/install-az-cli2) 並使用 [az login](/cli/azure/reference-index#az_login) 登入 Azure 帳戶。

在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包含 *myResourceGroup*、*myVnet* 和 *myVM*。

## <a name="create-resource-group"></a>建立資源群組
Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 您必須在建立虛擬機器和支援虛擬網路支援之前，先建立資源群組。 使用 [az group create](/cli/azure/group#az_group_create) 建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli
az group create --name myResourceGroup --location eastus
```

Azure CLI 命令的輸出預設是採用 JSON (JavaScript 物件標記法) 格式。 舉例來說，若要將預設輸出變更為清單或資料表，請使用 [az configure --output](/cli/azure/reference-index#az_configure)。 您也可以將 `--output` 新增到任何命令，以輸出格式進行一次變更。 下列範例顯示來自 `az group create` 命令的 JSON 輸出：

```json                       
{
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup",
  "location": "eastus",
  "name": "myResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="create-a-virtual-network-and-subnet"></a>建立虛擬網路和子網路
接著，您需在 Azure 中建立一個虛擬網路，以及一個可供您建立 VM 的子網路。 請使用 [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) 來建立一個名為 *myVnet* 且位址首碼為 *192.168.0.0/16* 的虛擬網路。 您還需新增一個名為 *mySubnet* 且位址首碼為 *192.168.1.0/24* 的子網路：

```azurecli
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 192.168.1.0/24
```

輸出會將子網路顯示為以邏輯方式建立在虛擬網路內︰

```json
{
  "addressSpace": {
    "addressPrefixes": [
      "192.168.0.0/16"
    ]
  },
  "dhcpOptions": {
    "dnsServers": []
  },
  "etag": "W/\"e95496fc-f417-426e-a4d8-c9e4d27fc2ee\"",
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet",
  "location": "eastus",
  "name": "myVnet",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "resourceGuid": "ed62fd03-e9de-430b-84df-8a3b87cacdbb",
  "subnets": [
    {
      "addressPrefix": "192.168.1.0/24",
      "etag": "W/\"e95496fc-f417-426e-a4d8-c9e4d27fc2ee\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet",
      "ipConfigurations": null,
      "name": "mySubnet",
      "networkSecurityGroup": null,
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "resourceNavigationLinks": null,
      "routeTable": null
    }
  ],
  "tags": {},
  "type": "Microsoft.Network/virtualNetworks",
  "virtualNetworkPeerings": null
}
```


## <a name="create-a-public-ip-address"></a>建立公用 IP 位址
現在，讓我們使用 [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) 來建立公用 IP 位址。 此公用 IP 位址可讓您從網際網路連線至 VM。 由於預設位址是動態位址，請使用 `--domain-name-label` 參數來建立具名的 DNS 項目。 下列範例會建立名為 *myPublicIP* 的公用 IP，其 DNS 名稱為 *mypublicdns*。 由於 DNS 名稱必須是唯一的，因此請提供您自己的唯一 DNS 名稱︰

```azurecli
az network public-ip create \
    --resource-group myResourceGroup \
    --name myPublicIP \
    --dns-name mypublicdns
```

輸出：

```json
{
  "publicIp": {
    "dnsSettings": {
      "domainNameLabel": "mypublicdns",
      "fqdn": "mypublicdns.eastus.cloudapp.azure.com",
      "reverseFqdn": null
    },
    "etag": "W/\"2632aa72-3d2d-4529-b38e-b622b4202925\"",
    "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP",
    "idleTimeoutInMinutes": 4,
    "ipAddress": null,
    "ipConfiguration": null,
    "location": "eastus",
    "name": "myPublicIP",
    "provisioningState": "Succeeded",
    "publicIpAddressVersion": "IPv4",
    "publicIpAllocationMethod": "Dynamic",
    "resourceGroup": "myResourceGroup",
    "resourceGuid": "4c65de38-71f5-4684-be10-75e605b3e41f",
    "tags": null,
    "type": "Microsoft.Network/publicIPAddresses"
  }
}
```


## <a name="create-a-network-security-group"></a>建立網路安全性群組
若要控制進出 VM 的流量，請將網路安全性群組套用至虛擬 NIC 或子網路。 下列範例會使用 [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create) 來建立名為 *myNetworkSecurityGroup* 的網路安全性群組：

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup
```

您需定義允許或拒絕特定流量的規則。 若要允許連接埠 22 上的輸入連線 (以支援 SSH 存取)，請使用 [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create) 建立輸入規則。 下列範例會建立名為 myNetworkSecurityGroupRuleSSH 的規則：

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRuleSSH \
    --protocol tcp \
    --priority 1000 \
    --destination-port-range 22 \
    --access allow
```

若要允許連接埠 80 上的輸入連線 (Web 流量)，請新增另一個網路安全性群組規則。 下列範例會建立一個名為 *myNetworkSecurityGroupRuleHTTP* 的規則：

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRuleWeb \
    --protocol tcp \
    --priority 1001 \
    --destination-port-range 80 \
    --access allow
```

使用 [az network nsg show](/cli/azure/network/nsg#az_network_nsg_show) 檢查網路安全性群組及規則：

```azurecli
az network nsg show --resource-group myResourceGroup --name myNetworkSecurityGroup
```

輸出：

```json
{
  "defaultSecurityRules": [
    {
      "access": "Allow",
      "description": "Allow inbound traffic from all VMs in VNET",
      "destinationAddressPrefix": "VirtualNetwork",
      "destinationPortRange": "*",
      "direction": "Inbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowVnetInBound",
      "name": "AllowVnetInBound",
      "priority": 65000,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "VirtualNetwork",
      "sourcePortRange": "*"
    },
    {
      "access": "Allow",
      "description": "Allow inbound traffic from azure load balancer",
      "destinationAddressPrefix": "*",
      "destinationPortRange": "*",
      "direction": "Inbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowAzureLoadBalancerInBou
      "name": "AllowAzureLoadBalancerInBound",
      "priority": 65001,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "AzureLoadBalancer",
      "sourcePortRange": "*"
    },
    {
      "access": "Deny",
      "description": "Deny all inbound traffic",
      "destinationAddressPrefix": "*",
      "destinationPortRange": "*",
      "direction": "Inbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/DenyAllInBound",
      "name": "DenyAllInBound",
      "priority": 65500,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    },
    {
      "access": "Allow",
      "description": "Allow outbound traffic from all VMs to all VMs in VNET",
      "destinationAddressPrefix": "VirtualNetwork",
      "destinationPortRange": "*",
      "direction": "Outbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowVnetOutBound",
      "name": "AllowVnetOutBound",
      "priority": 65000,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "VirtualNetwork",
      "sourcePortRange": "*"
    },
    {
      "access": "Allow",
      "description": "Allow outbound traffic from all VMs to Internet",
      "destinationAddressPrefix": "Internet",
      "destinationPortRange": "*",
      "direction": "Outbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/AllowInternetOutBound",
      "name": "AllowInternetOutBound",
      "priority": 65001,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    },
    {
      "access": "Deny",
      "description": "Deny all outbound traffic",
      "destinationAddressPrefix": "*",
      "destinationPortRange": "*",
      "direction": "Outbound",
      "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/defaultSecurityRules/DenyAllOutBound",
      "name": "DenyAllOutBound",
      "priority": 65500,
      "protocol": "*",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    }
  ],
  "etag": "W/\"3371b313-ea9f-4687-a336-a8ebdfd80523\"",
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup",
  "location": "eastus",
  "name": "myNetworkSecurityGroup",
  "networkInterfaces": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "resourceGuid": "47a9964e-23a3-438a-a726-8d60ebbb1c3c",
  "securityRules": [
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationPortRange": "22",
      "direction": "Inbound",
      "etag": "W/\"9e344b60-0daa-40a6-84f9-0ebbe4a4b640\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/securityRules/myNetworkSecurityGroupRuleSSH",
      "name": "myNetworkSecurityGroupRuleSSH",
      "priority": 1000,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    },
    {
      "access": "Allow",
      "description": null,
      "destinationAddressPrefix": "*",
      "destinationPortRange": "80",
      "direction": "Inbound",
      "etag": "W/\"9e344b60-0daa-40a6-84f9-0ebbe4a4b640\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup/securityRules/myNetworkSecurityGroupRuleWeb",
      "name": "myNetworkSecurityGroupRuleWeb",
      "priority": 1001,
      "protocol": "Tcp",
      "provisioningState": "Succeeded",
      "resourceGroup": "myResourceGroup",
      "sourceAddressPrefix": "*",
      "sourcePortRange": "*"
    }
  ],
  "subnets": null,
  "tags": null,
  "type": "Microsoft.Network/networkSecurityGroups"
}
```

## <a name="create-a-virtual-nic"></a>建立虛擬 NIC
虛擬網路介面卡 (NIC) 可透過程式設計方式供您使用，因為您可以將規則套用到 NIC 的使用上。 視 [VM 大小](sizes.md)而定，您可以將多個虛擬 NIC 附加至 VM。 在下列 [az network nic create](/cli/azure/network/nic#az_network_nic_create) 命令中，您會建立一個名為 *myNic* 的 NIC，並將它與網路安全性群組建立關聯。 公用 IP 位址 *myPublicIP* 也會與虛擬 NIC 建立關聯。

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --public-ip-address myPublicIP \
    --network-security-group myNetworkSecurityGroup
```

輸出：

```json
{
  "NewNIC": {
    "dnsSettings": {
      "appliedDnsServers": [],
      "dnsServers": [],
      "internalDnsNameLabel": null,
      "internalDomainNameSuffix": "brqlt10lvoxedgkeuomc4pm5tb.bx.internal.cloudapp.net",
      "internalFqdn": null
    },
    "enableAcceleratedNetworking": false,
    "enableIpForwarding": false,
    "etag": "W/\"04b5ab44-d8f4-422a-9541-e5ae7de8466d\"",
    "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic",
    "ipConfigurations": [
      {
        "applicationGatewayBackendAddressPools": null,
        "etag": "W/\"04b5ab44-d8f4-422a-9541-e5ae7de8466d\"",
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic/ipConfigurations/ipconfig1",
        "loadBalancerBackendAddressPools": null,
        "loadBalancerInboundNatRules": null,
        "name": "ipconfig1",
        "primary": true,
        "privateIpAddress": "192.168.1.4",
        "privateIpAddressVersion": "IPv4",
        "privateIpAllocationMethod": "Dynamic",
        "provisioningState": "Succeeded",
        "publicIpAddress": {
          "dnsSettings": null,
          "etag": null,
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP",
          "idleTimeoutInMinutes": null,
          "ipAddress": null,
          "ipConfiguration": null,
          "location": null,
          "name": null,
          "provisioningState": null,
          "publicIpAddressVersion": null,
          "publicIpAllocationMethod": null,
          "resourceGroup": "myResourceGroup",
          "resourceGuid": null,
          "tags": null,
          "type": null
        },
        "resourceGroup": "myResourceGroup",
        "subnet": {
          "addressPrefix": null,
          "etag": null,
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet",
          "ipConfigurations": null,
          "name": null,
          "networkSecurityGroup": null,
          "provisioningState": null,
          "resourceGroup": "myResourceGroup",
          "resourceNavigationLinks": null,
          "routeTable": null
        }
      }
    ],
    "location": "eastus",
    "macAddress": null,
    "name": "myNic",
    "networkSecurityGroup": {
      "defaultSecurityRules": null,
      "etag": null,
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNetworkSecurityGroup",
      "location": null,
      "name": null,
      "networkInterfaces": null,
      "provisioningState": null,
      "resourceGroup": "myResourceGroup",
      "resourceGuid": null,
      "securityRules": null,
      "subnets": null,
      "tags": null,
      "type": null
    },
    "primary": null,
    "provisioningState": "Succeeded",
    "resourceGroup": "myResourceGroup",
    "resourceGuid": "b3dbaa0e-2cf2-43be-a814-5cc49fea3304",
    "tags": null,
    "type": "Microsoft.Network/networkInterfaces",
    "virtualMachine": null
  }
}
```


## <a name="create-an-availability-set"></a>建立可用性設定組
可用性設定組可協助將 VM 分散到容錯網域和更新網域。 儘管您目前只建立一個 VM，但最佳做法是使用可用性設定組，這可讓您在未來更容易進行擴充。 

容錯網域定義一個虛擬機器群組，且群組內的虛擬機器會共用通用電源和網路交換器。 根據預設，可用性設定組內設定的虛擬機器最多可分散到三個容錯網域。 其中一個容錯網域中的硬體問題並不會影響每個執行您應用程式的 VM。

更新網域表示虛擬機器群組和可同時重新啟動的基礎實體硬體。 在計劃性維護期間，可能不會循序重新啟動更新網域，而只會一次重新啟動一個更新網域。

將多個 VM 放在一個可用性設定組中時，Azure 會自動將它們分散到容錯和更新網域。 如需詳細資訊，請參閱[管理 VM 的可用性](manage-availability.md)。

請使用 [az vm availability-set create](/cli/azure/vm/availability-set#az_vm_availability_set_create) 來建立 VM 的可用性設定組。 下列範例會建立名為 myAvailabilitySet 的可用性設定組：

```azurecli
az vm availability-set create \
    --resource-group myResourceGroup \
    --name myAvailabilitySet
```

輸出會指出容錯網域和更新網域：

```json
{
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/availabilitySets/myAvailabilitySet",
  "location": "eastus",
  "managed": null,
  "name": "myAvailabilitySet",
  "platformFaultDomainCount": 2,
  "platformUpdateDomainCount": 5,
  "resourceGroup": "myResourceGroup",
  "sku": {
    "capacity": null,
    "managed": true,
    "tier": null
  },
  "statuses": null,
  "tags": {},
  "type": "Microsoft.Compute/availabilitySets",
  "virtualMachines": []
}
```


## <a name="create-a-vm"></a>建立 VM
您已建立網路資源來支援可存取網際網路的 VM。 現在，請建立 VM 並使用 SSH 金鑰來保護它。 在此範例中，我們根據最新的 LTS 建立 Ubuntu VM。 您可以使用 [az vm image list](/cli/azure/vm/image#az_vm_image_list) 來找出其他映像，如[尋找 Azure VM 映像](cli-ps-findimage.md)所述。

指定要用於驗證的 SSH 金鑰。 如果您沒有 SSH 公開金鑰組，則可以[建立 SSH 公開金鑰組](mac-create-ssh-keys.md)，或使用 `--generate-ssh-keys` 參數來為您建立這些金鑰組。 如果您已經有金鑰組，此參數就會使用 `~/.ssh` 中的現有金鑰。

請使用 [az vm create](/cli/azure/vm#az_vm_create) 命令將我們的所有資源和資訊結合在一起來建立 VM。 下列範例會建立名為 myVM 的 VM。

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --location eastus \
    --availability-set myAvailabilitySet \
    --nics myNic \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

請使用您在建立公用 IP 位址時所提供的 DNS 項目，透過 SSH 連線到 VM。 當您建立 VM 時，輸出中會顯示這個 `fqdn`：

```json
{
  "fqdns": "mypublicdns.eastus.cloudapp.azure.com",
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-13-71-C8",
  "powerState": "VM running",
  "privateIpAddress": "192.168.1.5",
  "publicIpAddress": "13.90.94.252",
  "resourceGroup": "myResourceGroup"
}
```

```bash
ssh azureuser@mypublicdns.eastus.cloudapp.azure.com
```

輸出：

```bash
The authenticity of host 'mypublicdns.eastus.cloudapp.azure.com (13.90.94.252)' can't be established.
ECDSA key fingerprint is SHA256:SylINP80Um6XRTvWiFaNz+H+1jcrKB1IiNgCDDJRj6A.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'mypublicdns.eastus.cloudapp.azure.com,13.90.94.252' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.11.0-1016-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

azureuser@myVM:~$
```

您可以安裝 NGINX，然後查看傳送到 VM 的流量。 請依下列方式安裝 NGINX：

```bash
sudo apt-get install -y nginx
```

若要查看預設 NGINX 站台的實際運作情況，請開啟您的網頁瀏覽器，然後輸入 FQDN：

![您 VM 上的預設 NGINX 站台](media/create-cli-complete/nginx.png)

## <a name="export-as-a-template"></a>以範本形式匯出
如果您現在想要使用相同的參數來建立其他開發環境，或想要建立與其相符的生產環境，該怎麼辦？ Resource Manager 可使用定義了所有環境參數的 JSON 範本。 您可以透過參考此 JSON 範本來建置整個環境。 您可以[手動建立 JSON 範本](../../resource-group-authoring-templates.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)，或將現有的環境匯出來為您建立 JSON 範本。 使用 [az group export](/cli/azure/group#az_group_export) 來匯出您的資源群組，如下所示︰

```azurecli
az group export --name myResourceGroup > myResourceGroup.json
```

此命令會在您目前的工作目錄中建立 `myResourceGroup.json` 檔案。 當您從這個範本建立環境時，系統會提示您輸入所有資源名稱。 您可以透過將 `--include-parameter-default-value` 參數新增到 `az group export` 命令中，在您的範本檔案中填入這些名稱。 請編輯您的 JSON 範本以指定資源名稱，或 [建立 parameters.json 檔案](../../resource-group-authoring-templates.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 來指定資源名稱。

若要從您的範本建立環境，請使用 [az group deployment create](/cli/azure/group/deployment#az_group_deployment_create)，如下所示︰

```azurecli
az group deployment create \
    --resource-group myNewResourceGroup \
    --template-file myResourceGroup.json
```

您可以 [深入了解如何從範本進行部署](../../resource-group-template-deploy-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 請了解如何以累加方式更新環境、使用參數檔案，以及從單一儲存體位置存取範本。

## <a name="next-steps"></a>後續步驟
現在您已準備好開始使用多個網路元件和 VM。 您可以利用這裡介紹的核心元件，使用這個範例環境來建置您的應用程式。
