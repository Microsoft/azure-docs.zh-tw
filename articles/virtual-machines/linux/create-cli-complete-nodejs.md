---
title: 使用 Azure 傳統 CLI 建立完整的 Linux 環境 | Microsoft Docs
description: 使用 Azure 傳統 CLI，從頭開始建立儲存體、Linux VM、虛擬網路與子網路、負載平衡器、NIC、公用 IP 以及網路安全性群組。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 4ba4060b-ce95-4747-a735-1d7c68597a1a
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/09/2017
ms.author: cynthn
ms.openlocfilehash: 560d1c55b159ed817c0b080171862c28ebe73f3e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46952795"
---
# <a name="create-a-complete-linux-environment-with-the-azure-classic-cli"></a>使用 Azure 傳統 CLI 建立完整的 Linux 環境
在這篇文章中，我們將建立一個簡單的網路，當中包含一個負載平衡器，以及一組對開發和簡單運算而言相當實用的 VM。 我們將以逐個命令的方式逐步完成程序命令，直到您具備兩個可供您透過網際網路從任何地方連線的有效、安全 Linux VM 為止。 然後您便可以繼續著手更複雜的網路和環境。

在過程中，您將了解 Resource Manager 部署模型提供給您的相依性階層，以及它提供多少功能。 在您了解系統建置的方式之後，您就可以使用 [Azure Resource Manager 範本](../../resource-group-authoring-templates.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)更快地建置系統。 此外，在您了解環境的組件彼此如何搭配運作之後，就可以更輕鬆地建立範本來將它們自動化。

此環境包含：

* 兩個位於可用性設定組內的 VM。
* 一個在連接埠 80 有負載平衡規則的負載平衡器。
* 可保護 VM 防止不必要流量的網路安全性群組 (NSG) 規則。

若要建立此自訂環境，您需要處於 Resource Manager 模式 (`azure config mode arm`) 的最新 [Azure 傳統 CLI](../../cli-install-nodejs.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 您也需要 JSON 剖析工具。 此範例使用 [jq](https://stedolan.github.io/jq/)。


## <a name="cli-versions-to-complete-the-task"></a>用以完成工作的 CLI 版本
您可以使用下列其中一個 CLI 版本來完成工作︰

- [Azure 傳統 CLI](#quick-commands)：適用於傳統與資源管理部署模型的 CLI (本文章)
- [Azure CLI](create-cli-complete.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)：適用於資源管理部署模型的新一代 CLI


## <a name="quick-commands"></a>快速命令
如果您需要快速完成工作，下列章節詳細說明上傳 VM 至 Azure 的基本命令。 每個步驟的詳細資訊和內容可在文件其他地方找到，從[這裡](#detailed-walkthrough)開始。

確定您已登入 [Azure 傳統 CLI](../../cli-install-nodejs.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)，而且使用的是 Resource Manager 模式：

```azurecli
azure config mode arm
```

在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包含 `myResourceGroup`、`mystorageaccount` 和 `myVM`。

建立資源群組。 下列範例會在 `westeurope` 位置建立名為 `myResourceGroup` 的資源群組：

```azurecli
azure group create -n myResourceGroup -l westeurope
```

使用 JSON 剖析器來確認資源群組：

```azurecli
azure group show myResourceGroup --json | jq '.'
```

建立儲存體帳戶。 下列範例會建立名為 `mystorageaccount` 的儲存體帳戶。 (儲存體帳戶名稱必須是唯一的，因此請提供您自己的唯一名稱。)

```azurecli
azure storage account create -g myResourceGroup -l westeurope \
  --kind Storage --sku-name GRS mystorageaccount
```

使用 JSON 剖析器來確認儲存體帳戶：

```azurecli
azure storage account show -g myResourceGroup mystorageaccount --json | jq '.'
```

建立虛擬網路 下列範例會建立名為 `myVnet`的虛擬網路：

```azurecli
azure network vnet create -g myResourceGroup -l westeurope\
  -n myVnet -a 192.168.0.0/16
```

建立子網路。 下列範例會建立名為 `mySubnet`的子網路：

```azurecli
azure network vnet subnet create -g myResourceGroup \
  -e myVnet -n mySubnet -a 192.168.1.0/24
```

使用 JSON 剖析器來確認虛擬網路和子網路：

```azurecli
azure network vnet show myResourceGroup myVnet --json | jq '.'
```

建立公用 IP。 下列範例會建立名為 `myPublicIP` 的公用 IP，DNS 名稱為`mypublicdns`。 (DNS 名稱必須是唯一的，因此請提供您自己的唯一名稱。)

```azurecli
azure network public-ip create -g myResourceGroup -l westeurope \
  -n myPublicIP  -d mypublicdns -a static -i 4
```

建立負載平衡器。 下列範例會建立名為 `myLoadBalancer` 的負載平衡器：

```azurecli
azure network lb create -g myResourceGroup -l westeurope -n myLoadBalancer
```

建立負載平衡器的前端 IP 集區，並與公用 IP 建立關聯。 下列範例會建立名為 `mySubnetPool` 的前端 IP 集區：

```azurecli
azure network lb frontend-ip create -g myResourceGroup -l myLoadBalancer \
  -i myPublicIP -n myFrontEndPool
```

建立負載平衡器的後端 IP 集區。 下列範例會建立名為 `myBackEndPool` 的後端 IP 集區：

```azurecli
azure network lb address-pool create -g myResourceGroup -l myLoadBalancer \
  -n myBackEndPool
```

建立負載平衡器的 SSH 輸入網路位址轉譯 (NAT) 規則。 下列範例會建立兩個負載平衡器，`myLoadBalancerRuleSSH1` 和 `myLoadBalancerRuleSSH2`：

```azurecli
azure network lb inbound-nat-rule create -g myResourceGroup -l myLoadBalancer \
  -n myLoadBalancerRuleSSH1 -p tcp -f 4222 -b 22
azure network lb inbound-nat-rule create -g myResourceGroup -l myLoadBalancer \
  -n myLoadBalancerRuleSSH2 -p tcp -f 4223 -b 22
```

建立負載平衡器的 Web 輸入 NAT 規則。 下列範例會建立名為 `myLoadBalancerRuleWeb` 的負載平衡器規則：

```azurecli
azure network lb rule create -g myResourceGroup -l myLoadBalancer \
  -n myLoadBalancerRuleWeb -p tcp -f 80 -b 80 \
  -t myFrontEndPool -o myBackEndPool
```

建立負載平衡器健全狀況探查。 下列範例會建立名為 `myHealthProbe` 的 TCP 探查：

```azurecli
azure network lb probe create -g myResourceGroup -l myLoadBalancer \
  -n myHealthProbe -p "tcp" -i 15 -c 4
```

使用 JSON 剖析器來確認負載平衡器、IP 集區及 NAT 規則：

```azurecli
azure network lb show -g myResourceGroup -n myLoadBalancer --json | jq '.'
```

建立第一個網路介面卡 (NIC)。 使用您自己的 Azure 訂用帳戶識別碼取代 `#####-###-###` 區段。 當檢查您建立的資源時，會在 **jq** 的輸出中註明您的訂用帳戶識別碼。 您也可以使用 `azure account list`來檢視您的訂用帳戶識別碼。

下列範例會建立名為 `myNic1` 的 NIC：

```azurecli
azure network nic create -g myResourceGroup -l westeurope \
  -n myNic1 -m myVnet -k mySubnet \
  -d "/subscriptions/########-####-####-####-############/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool" \
  -e "/subscriptions/########-####-####-####-############/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/inboundNatRules/myLoadBalancerRuleSSH1"
```

建立第二個 NIC。 下列範例會建立名為 `myNic2` 的 NIC：

```azurecli
azure network nic create -g myResourceGroup -l westeurope \
  -n myNic2 -m myVnet -k mySubnet \
  -d "/subscriptions/########-####-####-####-############/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool" \
  -e "/subscriptions/########-####-####-####-############/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/inboundNatRules/myLoadBalancerRuleSSH2"
```

使用 JSON 剖析器來確認兩個 NIC：

```azurecli
azure network nic show myResourceGroup myNic1 --json | jq '.'
azure network nic show myResourceGroup myNic2 --json | jq '.'
```

建立網路安全性群組。 下列範例會建立名為 `myNetworkSecurityGroup` 的網路安全性群組：

```azurecli
azure network nsg create -g myResourceGroup -l westeurope \
  -n myNetworkSecurityGroup
```

新增兩個網路安全性群組的輸入規則。 下列範例會建立兩個規則，名為 `myNetworkSecurityGroupRuleSSH` 和 `myNetworkSecurityGroupRuleHTTP`：

```azurecli
azure network nsg rule create -p tcp -r inbound -y 1000 -u 22 -c allow \
  -g myResourceGroup -a myNetworkSecurityGroup -n myNetworkSecurityGroupRuleSSH
azure network nsg rule create -p tcp -r inbound -y 1001 -u 80 -c allow \
  -g myResourceGroup -a myNetworkSecurityGroup -n myNetworkSecurityGroupRuleHTTP
```

使用 JSON 剖析器來確認網路安全性群組和輸入規則：

```azurecli
azure network nsg show -g myResourceGroup -n myNetworkSecurityGroup --json | jq '.'
```

將網路安全性群組繫結至兩個 NIC︰

```azurecli
azure network nic set -g myResourceGroup -o myNetworkSecurityGroup -n myNic1
azure network nic set -g myResourceGroup -o myNetworkSecurityGroup -n myNic2
```

建立可用性設定組。 下列範例會建立名為 `myAvailabilitySet` 的可用性設定組：

```azurecli
azure availset create -g myResourceGroup -l westeurope -n myAvailabilitySet
```

建立第一個 Linux VM。 下列範例會建立名為 `myVM1` 的 VM：

```azurecli
azure vm create \
    --resource-group myResourceGroup \
    --name myVM1 \
    --location westeurope \
    --os-type linux \
    --availset-name myAvailabilitySet \
    --nic-name myNic1 \
    --vnet-name myVnet \
    --vnet-subnet-name mySubnet \
    --storage-account-name mystorageaccount \
    --image-urn canonical:UbuntuServer:16.04.0-LTS:latest \
    --ssh-publickey-file ~/.ssh/id_rsa.pub \
    --admin-username azureuser
```

建立第二個 Linux VM。 下列範例會建立名為 `myVM2` 的 VM：

```azurecli
azure vm create \
    --resource-group myResourceGroup \
    --name myVM2 \
    --location westeurope \
    --os-type linux \
    --availset-name myAvailabilitySet \
    --nic-name myNic2 \
    --vnet-name myVnet \
    --vnet-subnet-name mySubnet \
    --storage-account-name mystorageaccount \
    --image-urn canonical:UbuntuServer:16.04.0-LTS:latest \
    --ssh-publickey-file ~/.ssh/id_rsa.pub \
    --admin-username azureuser
```

使用 JSON 剖析器來確認已建置的所有項目︰

```azurecli
azure vm show -g myResourceGroup -n myVM1 --json | jq '.'
azure vm show -g myResourceGroup -n myVM2 --json | jq '.'
```

將您的新環境匯出成範本，以便快速重新建立新的執行個體：

```azurecli
azure group export myResourceGroup
```

## <a name="detailed-walkthrough"></a>詳細的逐步解說
接下來的詳細步驟將說明您建置環境時每個命令的功能。 當您建置自己的自訂開發環境或生產環境時，這些概念會相當有用。

確定您已登入 [Azure 傳統 CLI](../../cli-install-nodejs.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)，而且使用的是 Resource Manager 模式：

```azurecli
azure config mode arm
```

在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包含 `myResourceGroup`、`mystorageaccount` 和 `myVM`。

## <a name="create-resource-groups-and-choose-deployment-locations"></a>建立資源群組並選擇部署位置
Azure 資源群組是邏輯部署實體，當中包含用來啟用資源部署邏輯管理的組態資訊和中繼資料。 下列範例會在 `westeurope` 位置建立名為 `myResourceGroup` 的資源群組：

```azurecli
azure group create --name myResourceGroup --location westeurope
```

輸出：

```azurecli                        
info:    Executing command group create
+ Getting resource group myResourceGroup
+ Creating resource group myResourceGroup
info:    Created resource group myResourceGroup
data:    Id:                  /subscriptions/guid/resourceGroups/myResourceGroup
data:    Name:                myResourceGroup
data:    Location:            westeurope
data:    Provisioning State:  Succeeded
data:    Tags: null
data:
info:    group create command OK
```

## <a name="create-a-storage-account"></a>建立儲存體帳戶
您需要儲存體帳戶來用於您的 VM 磁碟和任何您想要新增的額外資料磁碟。 您幾乎是在建立資源群組之後立即建立儲存體帳戶。

在這裡，我們使用 `azure storage account create` 命令，其中會傳遞帳戶的位置、控制它的資源群組，以及您想要的儲存體支援類型。 下列範例會建立名為 `mystorageaccount` 的儲存體帳戶：

```azurecli
azure storage account create \  
  --location westeurope \
  --resource-group myResourceGroup \
  --kind Storage --sku-name GRS \
  mystorageaccount
```

輸出：

```azurecli
info:    Executing command storage account create
+ Creating storage account
info:    storage account create command OK
```

若要使用 `azure group show` 命令來檢查我們的資源群組，讓我們使用 [jq](https://stedolan.github.io/jq/) 工具來搭配 `--json` Azure CLI 選項。 (您可以使用 **jsawk** 或任何您慣用的語言程式庫來剖析 JSON。)

```azurecli
azure group show myResourceGroup --json | jq '.'
```

輸出：

```json
{
  "tags": {},
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup",
  "name": "myResourceGroup",
  "provisioningState": "Succeeded",
  "location": "westeurope",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "resources": [
    {
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
      "name": "mystorageaccount",
      "type": "storageAccounts",
      "location": "westeurope",
      "tags": null
    }
  ],
  "permissions": [
    {
      "actions": [
        "*"
      ],
      "notActions": []
    }
  ]
}
```

若要使用 CLI 來調查儲存體帳戶，您必須先設定帳戶名稱和金鑰。 以您選擇的名稱取代下列範例中的儲存體帳戶名稱︰

```bash
export AZURE_STORAGE_CONNECTION_STRING="$(azure storage account connectionstring show mystorageaccount --resource-group myResourceGroup --json | jq -r '.string')"
```

然後您就可以輕鬆地檢視您的儲存體資訊︰

```azurecli
azure storage container list
```

輸出：

```azurecli
info:    Executing command storage container list
+ Getting storage containers
data:    Name  Public-Access  Last-Modified
data:    ----  -------------  -----------------------------
data:    vhds  Off            Sun, 27 Sep 2015 19:03:54 GMT
info:    storage container list command OK
```

## <a name="create-a-virtual-network-and-subnet"></a>建立虛擬網路和子網路
接著，您將需要建立一個在 Azure 中執行的虛擬網路，以及一個可供您建立 VM 的子網路。 下列範例會建立名為 `myVnet` 的虛擬網路，位址首碼為 `192.168.0.0/16`：

```azurecli
azure network vnet create --resource-group myResourceGroup --location westeurope \
  --name myVnet --address-prefixes 192.168.0.0/16
```

輸出：

```azurecli
info:    Executing command network vnet create
+ Looking up virtual network "myVnet"
+ Creating virtual network "myVnet"
+ Loading virtual network state
data:    Id                              : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet
data:    Name                            : myVnet
data:    Type                            : Microsoft.Network/virtualNetworks
data:    Location                        : westeurope
data:    ProvisioningState               : Succeeded
data:    Address prefixes:
data:      192.168.0.0/16
info:    network vnet create command OK
```

同樣地，讓我們使用 --json 選項 `azure group show` 和 `jq` 來查看我們如何建置資源。 我們現在有 `storageAccounts` 資源和 `virtualNetworks` 資源。  

```azurecli
azure group show myResourceGroup --json | jq '.'
```

輸出：

```json
{
  "tags": {},
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup",
  "name": "myResourceGroup",
  "provisioningState": "Succeeded",
  "location": "westeurope",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "resources": [
    {
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet",
      "name": "myVnet",
      "type": "virtualNetworks",
      "location": "westeurope",
      "tags": null
    },
    {
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
      "name": "mystorageaccount",
      "type": "storageAccounts",
      "location": "westeurope",
      "tags": null
    }
  ],
  "permissions": [
    {
      "actions": [
        "*"
      ],
      "notActions": []
    }
  ]
}
```

現在讓我們在要於其中部署 VM 的 `myVnet` 虛擬網路中建立子網路。 我們將使用 `azure network vnet subnet create` 命令搭配我們已經建立的資源︰`myResourceGroup` 資源群組和 `myVnet` 虛擬網路。 在下列範例中，我們會新增名為 `mySubnet` 的子網路，子網路位址首碼為 `192.168.1.0/24`：

```azurecli
azure network vnet subnet create --resource-group myResourceGroup \
  --vnet-name myVnet --name mySubnet --address-prefix 192.168.1.0/24
```

輸出：

```azurecli
info:    Executing command network vnet subnet create
+ Looking up the subnet "mySubnet"
+ Creating subnet "mySubnet"
+ Looking up the subnet "mySubnet"
data:    Id                              : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet
data:    Type                            : Microsoft.Network/virtualNetworks/subnets
data:    ProvisioningState               : Succeeded
data:    Name                            : mySubnet
data:    Address prefix                  : 192.168.1.0/24
data:
info:    network vnet subnet create command OK
```

由於子網路是以邏輯方式位於虛擬網路內，因此我們將使用略為不同的命令來尋找子網路資訊。 我們使用的命令是 `azure network vnet show`，但是我們將繼續使用 `jq`來檢查 JSON 輸出。

```azurecli
azure network vnet show myResourceGroup myVnet --json | jq '.'
```

輸出：

```json
{
  "subnets": [
    {
      "ipConfigurations": [],
      "addressPrefix": "192.168.1.0/24",
      "provisioningState": "Succeeded",
      "name": "mySubnet",
      "etag": "W/\"974f3e2c-028e-4b35-832b-a4b16ad25eb6\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet"
    }
  ],
  "tags": {},
  "addressSpace": {
    "addressPrefixes": [
      "192.168.0.0/16"
    ]
  },
  "dhcpOptions": {
    "dnsServers": []
  },
  "provisioningState": "Succeeded",
  "etag": "W/\"974f3e2c-028e-4b35-832b-a4b16ad25eb6\"",
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet",
  "name": "myVnet",
  "location": "westeurope"
}
```

## <a name="create-a-public-ip-address"></a>建立公用 IP 位址
現在，讓我們建立將指派給您負載平衡器的公用 IP 位址 (PIP)。 它可讓您使用 `azure network public-ip create` 命令從網際網路連線到您的 VM。 由於預設位址是動態位址，因此我們將使用 `--domain-name-label` 選項在 **cloudapp.azure.com** 網域中建立具名的 DNS 項目。 下列範例會建立名為 `myPublicIP` 的公用 IP，DNS 名稱為`mypublicdns`。 因為 DNS 名稱必須是唯一的，因此請提供您自己的唯一 DNS 名稱︰

```azurecli
azure network public-ip create --resource-group myResourceGroup \
  --location westeurope --name myPublicIP --domain-name-label mypublicdns
```

輸出：

```azurecli
info:    Executing command network public-ip create
+ Looking up the public ip "myPublicIP"
+ Creating public ip address "myPublicIP"
+ Looking up the public ip "myPublicIP"
data:    Id                              : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP
data:    Name                            : myPublicIP
data:    Type                            : Microsoft.Network/publicIPAddresses
data:    Location                        : westeurope
data:    Provisioning state              : Succeeded
data:    Allocation method               : Dynamic
data:    Idle timeout                    : 4
data:    Domain name label               : mypublicdns
data:    FQDN                            : mypublicdns.westeurope.cloudapp.azure.com
info:    network public-ip create command OK
```

公用 IP 位址也是最上層資源，因此您可以使用 `azure group show`來查看它。

```azurecli
azure group show myResourceGroup --json | jq '.'
```

輸出：

```json
{
"tags": {},
"id": "/subscriptions/guid/resourceGroups/myResourceGroup",
"name": "myResourceGroup",
"provisioningState": "Succeeded",
"location": "westeurope",
"properties": {
    "provisioningState": "Succeeded"
},
"resources": [
    {
    "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP",
    "name": "myPublicIP",
    "type": "publicIPAddresses",
    "location": "westeurope",
    "tags": null
    },
    {
    "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet",
    "name": "myVnet",
    "type": "virtualNetworks",
    "location": "westeurope",
    "tags": null
    },
    {
    "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
    "name": "mystorageaccount",
    "type": "storageAccounts",
    "location": "westeurope",
    "tags": null
    }
],
"permissions": [
    {
    "actions": [
        "*"
    ],
    "notActions": []
    }
]
}
```

您可以使用完整的 `azure network public-ip show` 命令來調查更多資源詳細資料，包括子網域的完整網域名稱 (FQDN)。 公用 IP 位址資源已經以邏輯方式配置，但尚未指派特定位址。 若要取得 IP 位址，您將需要一個負載平衡器，而我們尚未建立此負載平衡器。

```azurecli
azure network public-ip show myResourceGroup myPublicIP --json | jq '.'
```

輸出：

```json
{
"tags": {},
"publicIpAllocationMethod": "Dynamic",
"dnsSettings": {
    "domainNameLabel": "mypublicdns",
    "fqdn": "mypublicdns.westeurope.cloudapp.azure.com"
},
"idleTimeoutInMinutes": 4,
"provisioningState": "Succeeded",
"etag": "W/\"c63154b3-1130-49b9-a887-877d74d5ebc5\"",
"id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP",
"name": "myPublicIP",
"location": "westeurope"
}
```

## <a name="create-a-load-balancer-and-ip-pools"></a>建立負載平衡器和 IP 集區
在建立負載平衡器的情況下，您將可以把流量分散到多個 VM。 它也會藉由執行多個 VM 以在進行維護或發生大量負載時回應使用者要求，為您的應用程式提供備援。 下列範例會建立名為 `myLoadBalancer` 的負載平衡器：

```azurecli
azure network lb create --resource-group myResourceGroup --location westeurope \
  --name myLoadBalancer
```

輸出：

```azurecli
info:    Executing command network lb create
+ Looking up the load balancer "myLoadBalancer"
+ Creating load balancer "myLoadBalancer"
data:    Id                              : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer
data:    Name                            : myLoadBalancer
data:    Type                            : Microsoft.Network/loadBalancers
data:    Location                        : westeurope
data:    Provisioning state              : Succeeded
info:    network lb create command OK
```

我們的負載平衡器很空，因此讓我們建立一些 IP 集區。 我們想要為負載平衡器建立兩個 IP 集區，一個用於前端，一個用於後端。 前端 IP 集區會公開顯示。 它也是我們指派稍早所建立 PIP 的位置。 然後，我們將使用後端集區作為我們 VM 要連接的位置。 這樣一來，流量便可以經由負載平衡器流向 VM。

首先，讓我們建立前端 IP 集區。 下列範例會建立名為 `myFrontEndPool` 的前端集區：

```azurecli
azure network lb frontend-ip create --resource-group myResourceGroup \
  --lb-name myLoadBalancer --public-ip-name myPublicIP \
  --name myFrontEndPool
```

輸出：

```azurecli
info:    Executing command network lb frontend-ip create
+ Looking up the load balancer "myLoadBalancer"
+ Looking up the public ip "myPublicIP"
+ Updating load balancer "myLoadBalancer"
data:    Name                            : myFrontEndPool
data:    Provisioning state              : Succeeded
data:    Private IP allocation method    : Dynamic
data:    Public IP address id            : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP
info:    network lb mySubnet-ip create command OK
```

請注意我們如何使用 `--public-ip-name` 參數來傳入我們稍早建立的 `myPublicIP`。 將公用 IP 位址指派給負載平衡器可讓您透過網際網路連線到您的 VM。

接下來，讓我們建立第二個 IP 集區，這次是針對我們的後端流量。 下列範例會建立名為 `myBackEndPool` 的後端集區：

```azurecli
azure network lb address-pool create --resource-group myResourceGroup \
  --lb-name myLoadBalancer --name myBackEndPool
```

輸出：

```azurecli
info:    Executing command network lb address-pool create
+ Looking up the load balancer "myLoadBalancer"
+ Updating load balancer "myLoadBalancer"
data:    Name                            : myBackEndPool
data:    Provisioning state              : Succeeded
info:    network lb address-pool create command OK
```

我們可以使用 `azure network lb show` 並檢查 JSON 輸出，來查看負載平衡器的執行情況：

```azurecli
azure network lb show myResourceGroup myLoadBalancer --json | jq '.'
```

輸出：

```json
{
  "etag": "W/\"29c38649-77d6-43ff-ab8f-977536b0047c\"",
  "provisioningState": "Succeeded",
  "resourceGuid": "f1446acb-09ba-44d9-b8b6-849d9983dc09",
  "outboundNatRules": [],
  "inboundNatPools": [],
  "inboundNatRules": [],
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer",
  "name": "myLoadBalancer",
  "type": "Microsoft.Network/loadBalancers",
  "location": "westeurope",
  "mySubnetIPConfigurations": [
    {
      "etag": "W/\"29c38649-77d6-43ff-ab8f-977536b0047c\"",
      "name": "myFrontEndPool",
      "provisioningState": "Succeeded",
      "publicIPAddress": {
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP"
      },
      "privateIPAllocationMethod": "Dynamic",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/mySubnetIPConfigurations/myFrontEndPool"
    }
  ],
  "backendAddressPools": [
    {
      "etag": "W/\"29c38649-77d6-43ff-ab8f-977536b0047c\"",
      "name": "myBackEndPool",
      "provisioningState": "Succeeded",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool"
    }
  ],
  "loadBalancingRules": [],
  "probes": []
}
```

## <a name="create-load-balancer-nat-rules"></a>建立負載平衡器 NAT 規則
若要讓流量流經負載平衡器，我們需要建立指定輸入或輸出動作的網路位址轉譯 (NAT) 規則。 您可以指定要使用的通訊協定，然後依需要將外部連接埠對應到內部連接埠。 在我們的環境中，建立一些規則以允許 SSH 透過負載平衡器流向 VM。 我們將設定 TCP 連接埠 4222 和 4223 以導向 VM 上的 TCP 連接埠 22 (我們會在稍後建立)。 下列範例會建立名為 `myLoadBalancerRuleSSH1` 的規則以將 TCP 連接埠 4222 對應至連接埠 22：

```azurecli
azure network lb inbound-nat-rule create --resource-group myResourceGroup \
  --lb-name myLoadBalancer --name myLoadBalancerRuleSSH1 \
  --protocol tcp --frontend-port 4222 --backend-port 22
```

輸出：

```azurecli
info:    Executing command network lb inbound-nat-rule create
+ Looking up the load balancer "myLoadBalancer"
warn:    Using default enable floating ip: false
warn:    Using default idle timeout: 4
warn:    Using default mySubnet IP configuration "myFrontEndPool"
+ Updating load balancer "myLoadBalancer"
data:    Name                            : myLoadBalancerRuleSSH1
data:    Provisioning state              : Succeeded
data:    Protocol                        : Tcp
data:    mySubnet port                   : 4222
data:    Backend port                    : 22
data:    Enable floating IP              : false
data:    Idle timeout in minutes         : 4
data:    mySubnet IP configuration id    : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/mySubnetIPConfigurations/myFrontEndPool
info:    network lb inbound-nat-rule create command OK
```

針對進行 SSH 的第二個 NAT 規則，重複此程序。 下列範例會建立名為 `myLoadBalancerRuleSSH2` 的規則以將 TCP 連接埠 4223 對應至連接埠 22：

```azurecli
azure network lb inbound-nat-rule create --resource-group myResourceGroup \
  --lb-name myLoadBalancer --name myLoadBalancerRuleSSH2 --protocol tcp \
  --frontend-port 4223 --backend-port 22
```

讓我們繼續進行，並針對 web 流量建立 TCP 連接埠 80 的 NAT 規則，以將規則連結到我們的 IP 集區。 如果我們將規則連結到 IP 集區，而不是將規則個別連結到 VM，則我們可在 IP 集區中新增或移除 VM。 負載平衡器會自動調整流量的流動。 下列範例會建立名為 `myLoadBalancerRuleWeb` 的規則以將 TCP 連接埠 80 對應至連接埠 80：

```azurecli
azure network lb rule create --resource-group myResourceGroup \
  --lb-name myLoadBalancer --name myLoadBalancerRuleWeb --protocol tcp \
  --frontend-port 80 --backend-port 80 --frontend-ip-name myFrontEndPool \
  --backend-address-pool-name myBackEndPool
```

輸出：

```azurecli
info:    Executing command network lb rule create
+ Looking up the load balancer "myLoadBalancer"
warn:    Using default idle timeout: 4
warn:    Using default enable floating ip: false
warn:    Using default load distribution: Default
+ Updating load balancer "myLoadBalancer"
data:    Name                            : myLoadBalancerRuleWeb
data:    Provisioning state              : Succeeded
data:    Protocol                        : Tcp
data:    mySubnet port                   : 80
data:    Backend port                    : 80
data:    Enable floating IP              : false
data:    Load distribution               : Default
data:    Idle timeout in minutes         : 4
data:    mySubnet IP configuration id    : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/mySubnetIPConfigurations/myFrontEndPool
data:    Backend address pool id         : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool
info:    network lb rule create command OK
```

## <a name="create-a-load-balancer-health-probe"></a>建立負載平衡器健全狀況探查
健全狀況探查會定期檢查受負載平衡器保護的 VM，以確定它們依定義的方式運作及回應要求。 否則，就從將它們從作業中移除，以確保不會將使用者導向到它們。 您可以定義健全狀況探查的自訂檢查，以及間隔和逾時值。 如需有關健全狀況探查的詳細資訊，請參閱 [負載平衡器探查](../../load-balancer/load-balancer-custom-probe-overview.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 下列範例會建立名為 `myHealthProbe` 的 TCP 健全狀況探查：

```azurecli
azure network lb probe create --resource-group myResourceGroup \
  --lb-name myLoadBalancer --name myHealthProbe --protocol "tcp" \
  --interval 15 --count 4
```

輸出：

```azurecli
info:    Executing command network lb probe create
warn:    Using default probe port: 80
+ Looking up the load balancer "myLoadBalancer"
+ Updating load balancer "myLoadBalancer"
data:    Name                            : myHealthProbe
data:    Provisioning state              : Succeeded
data:    Protocol                        : Tcp
data:    Port                            : 80
data:    Interval in seconds             : 15
data:    Number of probes                : 4
info:    network lb probe create command OK
```

在這裡，我們指定了 15 秒的健全狀況檢查間隔。 最多可錯過 4 次探查 (1 分鐘)，負載平衡器才會將該主機視為已不再運作。

## <a name="verify-the-load-balancer"></a>確認負載平衡器
現在，負載平衡器設定已完成。 以下是您所採取的步驟：

1. 您建立了負載平衡器。
2. 您建立了前端 IP 集區並為它指派公用 IP 位址。
3. 您建立了 VM 可以連接的後端 IP 集區。
4. 您建立了可允許透過 SSH 連接到 VM 以進行管理的 NAT 規則，以及可允許將 TCP 連接埠 80 用於 Web 應用程式的規則。
5. 您新增了健全狀況探查來定期檢查 VM。 這個健全狀況探查可確保使用者不會嘗試存取已不再運作或提供內容的 VM。

讓我們檢閱您負載平衡器現在的樣子︰

```azurecli
azure network lb show --resource-group myResourceGroup \
  --name myLoadBalancer --json | jq '.'
```

輸出：

```json
{
  "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
  "provisioningState": "Succeeded",
  "resourceGuid": "f1446acb-09ba-44d9-b8b6-849d9983dc09",
  "outboundNatRules": [],
  "inboundNatPools": [],
  "inboundNatRules": [
    {
      "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
      "name": "myLoadBalancerRuleSSH1",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/inboundNatRules/myLoadBalancerRuleSSH1",
      "mySubnetIPConfiguration": {
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/mySubnetIPConfigurations/myFrontEndPool"
      },
      "protocol": "Tcp",
      "mySubnetPort": 4222,
      "backendPort": 22,
      "idleTimeoutInMinutes": 4,
      "enableFloatingIP": false,
      "provisioningState": "Succeeded"
    },
    {
      "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
      "name": "myLoadBalancerRuleSSH2",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/inboundNatRules/myLoadBalancerRuleSSH2",
      "mySubnetIPConfiguration": {
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/mySubnetIPConfigurations/myFrontEndPool"
      },
      "protocol": "Tcp",
      "mySubnetPort": 4223,
      "backendPort": 22,
      "idleTimeoutInMinutes": 4,
      "enableFloatingIP": false,
      "provisioningState": "Succeeded"
    }
  ],
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer",
  "name": "myLoadBalancer",
  "type": "Microsoft.Network/loadBalancers",
  "location": "westeurope",
  "mySubnetIPConfigurations": [
    {
      "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
      "name": "myFrontEndPool",
      "provisioningState": "Succeeded",
      "publicIPAddress": {
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/publicIPAddresses/myPublicIP"
      },
      "privateIPAllocationMethod": "Dynamic",
      "loadBalancingRules": [
        {
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/loadBalancingRules/myLoadBalancerRuleWeb"
        }
      ],
      "inboundNatRules": [
        {
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/inboundNatRules/myLoadBalancerRuleSSH1"
        },
        {
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/inboundNatRules/myLoadBalancerRuleSSH2"
        }
      ],
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/mySubnetIPConfigurations/myFrontEndPool"
    }
  ],
  "backendAddressPools": [
    {
      "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
      "name": "myBackEndPool",
      "provisioningState": "Succeeded",
      "loadBalancingRules": [
        {
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/loadBalancingRules/myLoadBalancerRuleWeb"
        }
      ],
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool"
    }
  ],
  "loadBalancingRules": [
    {
      "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
      "name": "myLoadBalancerRuleWeb",
      "provisioningState": "Succeeded",
      "enableFloatingIP": false,
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/loadBalancingRules/myLoadBalancerRuleWeb",
      "mySubnetIPConfiguration": {
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/mySubnetIPConfigurations/myFrontEndPool"
      },
      "backendAddressPool": {
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool"
      },
      "protocol": "Tcp",
      "loadDistribution": "Default",
      "mySubnetPort": 80,
      "backendPort": 80,
      "idleTimeoutInMinutes": 4
    }
  ],
  "probes": [
    {
      "etag": "W/\"62a7c8e7-859c-48d3-8e76-5e078c5e4a02\"",
      "name": "myHealthProbe",
      "provisioningState": "Succeeded",
      "numberOfProbes": 4,
      "intervalInSeconds": 15,
      "port": 80,
      "protocol": "Tcp",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/probes/myHealthProbe"
    }
  ]
}
```

## <a name="create-an-nic-to-use-with-the-linux-vm"></a>建立要與 Linux VM 搭配使用的 NIC
您可以透過程式設計方式提供 NIC，因為您可以將規則套用到 NIC 的使用上。 您也可以有多個 NIC。 在下列 `azure network nic create` 命令中，您會將 NIC 連結到負載後端 IP 集區，並將它與 NAT 規則建立關聯以允許 SSH 流量。

使用您自己的 Azure 訂用帳戶識別碼取代 `#####-###-###` 區段。 當檢查您建立的資源時，會在 `jq` 的輸出中註明您的訂用帳戶識別碼。 您也可以使用 `azure account list`來檢視您的訂用帳戶識別碼。

下列範例會建立名為 `myNic1` 的 NIC：

```azurecli
azure network nic create --resource-group myResourceGroup --location westeurope \
  --subnet-vnet-name myVnet --subnet-name mySubnet --name myNic1 \
  --lb-address-pool-ids /subscriptions/########-####-####-####-############/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool \
  --lb-inbound-nat-rule-ids /subscriptions/########-####-####-####-############/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/inboundNatRules/myLoadBalancerRuleSSH1
```

輸出：

```azurecli
info:    Executing command network nic create
+ Looking up the subnet "mySubnet"
+ Looking up the network interface "myNic1"
+ Creating network interface "myNic1"
data:    Id                              : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic1
data:    Name                            : myNic1
data:    Type                            : Microsoft.Network/networkInterfaces
data:    Location                        : westeurope
data:    Provisioning state              : Succeeded
data:    Enable IP forwarding            : false
data:    IP configurations:
data:      Name                          : Nic-IP-config
data:      Provisioning state            : Succeeded
data:      Private IP address            : 192.168.1.4
data:      Private IP allocation method  : Dynamic
data:      Subnet                        : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet
data:      Load balancer backend address pools:
data:        Id                          : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool
data:      Load balancer inbound NAT rules:
data:        Id                          : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/inboundNatRules/myLoadBalancerRuleSSH1
data:
info:    network nic create command OK
```

您可以直接檢查資源來查看詳細資料。 您可以使用 `azure network nic show` 命令來檢查資源︰

```azurecli
azure network nic show myResourceGroup myNic1 --json | jq '.'
```

輸出：

```json
{
  "etag": "W/\"fc1eaaa1-ee55-45bd-b847-5a08c7f4264a\"",
  "provisioningState": "Succeeded",
  "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic1",
  "name": "myNic1",
  "type": "Microsoft.Network/networkInterfaces",
  "location": "westeurope",
  "ipConfigurations": [
    {
      "etag": "W/\"fc1eaaa1-ee55-45bd-b847-5a08c7f4264a\"",
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic1/ipConfigurations/Nic-IP-config",
      "loadBalancerBackendAddressPools": [
        {
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool"
        }
      ],
      "loadBalancerInboundNatRules": [
        {
          "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/inboundNatRules/myLoadBalancerRuleSSH1"
        }
      ],
      "privateIPAddress": "192.168.1.4",
      "privateIPAllocationMethod": "Dynamic",
      "subnet": {
        "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet"
      },
      "provisioningState": "Succeeded",
      "name": "Nic-IP-config"
    }
  ],
  "dnsSettings": {
    "appliedDnsServers": [],
    "dnsServers": []
  },
  "enableIPForwarding": false,
  "resourceGuid": "a20258b8-6361-45f6-b1b4-27ffed28798c"
}
```

現在，我們將建立的第二個 NIC，其中會再次將它連結到我們的後端 IP 集區。 這一次，第二個 NAT 規則將允許 SSH 流量。 下列範例會建立名為 `myNic2` 的 NIC：

```azurecli
azure network nic create --resource-group myResourceGroup --location westeurope \
  --subnet-vnet-name myVnet --subnet-name mySubnet --name myNic2 \
  --lb-address-pool-ids  /subscriptions/########-####-####-####-############/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/backendAddressPools/myBackEndPool \
  --lb-inbound-nat-rule-ids /subscriptions/########-####-####-####-############/resourceGroups/myResourceGroup/providers/Microsoft.Network/loadBalancers/myLoadBalancer/inboundNatRules/myLoadBalancerRuleSSH2
```

## <a name="create-a-network-security-group-and-rules"></a>建立網路安全性群組和規則
我們現在會建立網路安全性群組和管理 NIC 存取權的輸入規則。 網路安全性群組可以套用至 NIC 或子網路。 您要定義規則以控制進出 VM 的流量。 下列範例會建立名為 `myNetworkSecurityGroup` 的網路安全性群組：

```azurecli
azure network nsg create --resource-group myResourceGroup --location westeurope \
  --name myNetworkSecurityGroup
```

讓我們加入 NSG 的輸入規則以允許連接埠 22 上的輸入連線 (以支援 SSH)。 下列範例會建立名為 `myNetworkSecurityGroupRuleSSH` 的規則以允許連接埠 22 上的 TCP︰

```azurecli
azure network nsg rule create --resource-group myResourceGroup \
  --nsg-name myNetworkSecurityGroup --protocol tcp --direction inbound \
  --priority 1000 --destination-port-range 22 --access allow \
  --name myNetworkSecurityGroupRuleSSH
```

現在讓我們加入 NSG 的輸入規則以允許連接埠 80 上的輸入連線 (以支援 web 流量)。 下列範例會建立名為 `myNetworkSecurityGroupRuleHTTP` 的規則以允許連接埠 80 上的 TCP︰

```azurecli
azure network nsg rule create --resource-group myResourceGroup \
  --nsg-name myNetworkSecurityGroup --protocol tcp --direction inbound \
  --priority 1001 --destination-port-range 80 --access allow \
  --name myNetworkSecurityGroupRuleHTTP
```

> [!NOTE]
> 輸入規則是輸入網路連線的篩選器。 在此範例中，我們將 NSG 繫結至 VM 虛擬 NIC，這意謂著任何傳送給連接埠 22 的要求都會傳遞到 VM 上的 NIC。 這個輸入規則與網路連線相關，而不是與端點 (在傳統部署中會相關的對象) 相關。 若要開啟連接埠，您必須將 `--source-port-range` 保留設定為 '\*' (預設)，才能接受來自 **「任何」** 要求連接埠的輸入要求。 連接埠通常是動態的。
>
>

## <a name="bind-to-the-nic"></a>繫結至 NIC
將 NSG 繫結至 NIC。 我們需要將 NIC 與我們的網路安全性群組連線。 執行這兩個命令來連結我們的兩個 NIC：

```azurecli
azure network nic set --resource-group myResourceGroup --name myNic1 \
  --network-security-group-name myNetworkSecurityGroup
```

```azurecli
azure network nic set --resource-group myResourceGroup --name myNic2 \
  --network-security-group-name myNetworkSecurityGroup
```

## <a name="create-an-availability-set"></a>建立可用性設定組
可用性設定組可協助將 VM 分散到容錯網域和升級網域。 為 VM 建立可用性設定組。 下列範例會建立名為 `myAvailabilitySet` 的可用性設定組：

```azurecli
azure availset create --resource-group myResourceGroup --location westeurope
  --name myAvailabilitySet
```

容錯網域定義一個虛擬機器群組，且群組內的虛擬機器會共用通用電源和網路交換器。 根據預設，可用性設定組內設定的虛擬機器最多可分散到三個容錯網域。 這裡的概念是，其中一個容錯網域中的硬體問題將不會影響每個執行您應用程式的 VM。 將多個 VM 放在一個可用性設定組中時，Azure 會自動將它們分散到容錯網域。

升級網域表示虛擬機器群組和可同時重新啟動的基礎實體硬體。 在計劃性維護期間，可能不會循序重新啟動升級網域，而只會一次重新啟動一個升級網域。 同樣地，將多個 VM 放在一個可用性設定網站中時，Azure 會自動將它們分散到升級網域。

深入了解 [管理 VM 的可用性](manage-availability.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

## <a name="create-the-linux-vms"></a>建立 Linux VM
您已建立儲存體和網路資源來支援可存取網際網路的 VM。 現在，讓我們建立這些 VM，並利用沒有密碼的 SSH 金鑰來保障其安全。 在此情況下，我們要根據最新的 LTS 建立 Ubuntu VM。 我們會使用 `azure vm image list`來找出該映像資訊 (如 [尋找 Azure VM 映像](../windows/cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)所述)。

我們之前使用 `azure vm image list westeurope canonical | grep LTS`命令來選取映像。 在此案例中，我們會使用 `canonical:UbuntuServer:16.04.0-LTS:16.04.201608150`。 針對最後一個欄位，我們會傳遞 `latest` ，如此在未來我們就一律會取得最新的組建。 (我們使用的字串是 `canonical:UbuntuServer:16.04.0-LTS:16.04.201608150`)。

對於任何已經使用 **ssh-keygen -t rsa -b 2048**在 Linux 或 Mac 上建立 ssh rsa 公用和私密金鑰組的人來說，這下一個步驟都是相當熟悉的步驟。 如果您的 `~/.ssh` 目錄中沒有任何憑證金鑰組，您可以建立它們︰

* 藉由使用 `azure vm create --generate-ssh-keys` 選項來自動建立。
* 藉由使用 [自行建立的指示](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)來手動建立。

或者，您也可以使用 `--admin-password` 方法，在 VM 建立後驗證 SSH 連線。 這個方法通常較不安全。

我們會使用 `azure vm create` 命令將所有資源和資訊結合在一起來建立 VM：

```azurecli
azure vm create \
  --resource-group myResourceGroup \
  --name myVM1 \
  --location westeurope \
  --os-type linux \
  --availset-name myAvailabilitySet \
  --nic-name myNic1 \
  --vnet-name myVnet \
  --vnet-subnet-name mySubnet \
  --storage-account-name mystorageaccount \
  --image-urn canonical:UbuntuServer:16.04.0-LTS:latest \
  --ssh-publickey-file ~/.ssh/id_rsa.pub \
  --admin-username azureuser
```

輸出：

```azurecli
info:    Executing command vm create
+ Looking up the VM "myVM1"
info:    Verifying the public key SSH file: /home/ahmet/.ssh/id_rsa.pub
info:    Using the VM Size "Standard_DS1"
info:    The [OS, Data] Disk or image configuration requires storage account
+ Looking up the storage account mystorageaccount
+ Looking up the availability set "myAvailabilitySet"
info:    Found an Availability set "myAvailabilitySet"
+ Looking up the NIC "myNic1"
info:    Found an existing NIC "myNic1"
info:    Found an IP configuration with virtual network subnet id "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet" in the NIC "myNic1"
info:    This is an NIC without publicIP configured
info:    The storage URI 'https://mystorageaccount.blob.core.windows.net/' will be used for boot diagnostics settings, and it can be overwritten by the parameter input of '--boot-diagnostics-storage-uri'.
info:    vm create command OK
```

您可以使用預設 SSH 金鑰來立即連線到 VM。 請確定您指定的連接埠正確，因為我們要通過的是負載平衡器。 (針對第一個 VM，我們設定了將連接埠 4222 轉送到 VM 的 NAT 規則。)

```bash
ssh ops@mypublicdns.westeurope.cloudapp.azure.com -p 4222
```

輸出：

```bash
The authenticity of host '[mypublicdns.westeurope.cloudapp.azure.com]:4222 ([xx.xx.xx.xx]:4222)' can't be established.
ECDSA key fingerprint is 94:2d:d0:ce:6b:fb:7f:ad:5b:3c:78:93:75:82:12:f9.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[mypublicdns.westeurope.cloudapp.azure.com]:4222,[xx.xx.xx.xx]:4222' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-34-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.

ops@myVM1:~$
```

繼續進行，並以相同方式建立第二個 VM：

```azurecli
azure vm create \
  --resource-group myResourceGroup \
  --name myVM2 \
  --location westeurope \
  --os-type linux \
  --availset-name myAvailabilitySet \
  --nic-name myNic2 \
  --vnet-name myVnet \
  --vnet-subnet-name mySubnet \
  --storage-account-name mystorageaccount \
  --image-urn canonical:UbuntuServer:16.04.0-LTS:latest \
  --ssh-publickey-file ~/.ssh/id_rsa.pub \
  --admin-username azureuser
```

您現在可以使用 `azure vm show myResourceGroup myVM1` 命令來檢查您已建立的項目。 此時，您的 Ubuntu VM 是在 Azure 中的負載平衡器後方執行，您只能利用 SSH 金鑰組來登入 (因為密碼已被停用)。 您可以安裝 nginx 或 httpd、部署 Web 應用程式，然後查看經由負載平衡器流向兩個 VM 的流量。

```azurecli
azure vm show --resource-group myResourceGroup --name myVM1
```

輸出：

```azurecli
info:    Executing command vm show
+ Looking up the VM "TestVM1"
+ Looking up the NIC "myNic1"
data:    Id                              :/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM1
data:    ProvisioningState               :Succeeded
data:    Name                            :myVM1
data:    Location                        :westeurope
data:    Type                            :Microsoft.Compute/virtualMachines
data:
data:    Hardware Profile:
data:      Size                          :Standard_DS1
data:
data:    Storage Profile:
data:      Image reference:
data:        Publisher                   :canonical
data:        Offer                       :UbuntuServer
data:        Sku                         :16.04.0-LTS
data:        Version                     :latest
data:
data:      OS Disk:
data:        OSType                      :Linux
data:        Name                        :clib45a8b650f4428a1-os-1471973896525
data:        Caching                     :ReadWrite
data:        CreateOption                :FromImage
data:        Vhd:
data:          Uri                       :https://mystorageaccount.blob.core.windows.net/vhds/clib45a8b650f4428a1-os-1471973896525.vhd
data:
data:    OS Profile:
data:      Computer Name                 :myVM1
data:      User Name                     :ops
data:      Linux Configuration:
data:        Disable Password Auth       :true
data:
data:    Network Profile:
data:      Network Interfaces:
data:        Network Interface #1:
data:          Primary                   :true
data:          MAC Address               :00-0D-3A-24-D4-AA
data:          Provisioning State        :Succeeded
data:          Name                      :LmyNic1
data:          Location                  :westeurope
data:
data:    AvailabilitySet:
data:      Id                            :/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/availabilitySets/myAvailabilitySet
data:
data:    Diagnostics Profile:
data:      BootDiagnostics Enabled       :true
data:      BootDiagnostics StorageUri    :https://mystorageaccount.blob.core.windows.net/
data:
data:      Diagnostics Instance View:
info:    vm show command OK

```


## <a name="export-the-environment-as-a-template"></a>將環境匯出為範本
您現在已經建立這個環境，如果您想要使用相同的參數來建立其他開發環境，或想要建立與其相符的生產環境時，該怎麼辦？ Resource Manager 可使用定義了所有環境參數的 JSON 範本。 您可以藉由參考此 JSON 範本來建置整個環境。 您可以 [手動建立 JSON 範本](../../resource-group-authoring-templates.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)，或將現有的環境匯出來為您建立 JSON 範本：

```azurecli
azure group export --name myResourceGroup
```

此命令會在您目前的工作目錄中建立 `myResourceGroup.json` 檔案。 當您從這個範本建立環境時，系統會提示您輸入所有資源名稱，包括負載平衡器、網路介面或 VM 的名稱。 您可以藉由將 `-p` 或 `--includeParameterDefaultValue` 參數新增至稍早所示的 `azure group export` 命令中，在您的範本檔案中填入這些名稱。 請編輯您的 JSON 範本以指定資源名稱，或 [建立 parameters.json 檔案](../../resource-group-authoring-templates.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 來指定資源名稱。

從範本建立環境：

```azurecli
azure group deployment create --resource-group myNewResourceGroup \
  --template-file myResourceGroup.json
```

您可以 [深入了解如何從範本進行部署](../../resource-group-template-deploy-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 請了解如何以累加方式更新環境、使用參數檔案，以及從單一儲存體位置存取範本。

## <a name="next-steps"></a>後續步驟
現在您已準備好開始使用多個網路元件和 VM。 您可以利用這裡介紹的核心元件，使用這個範例環境來建置您的應用程式。
