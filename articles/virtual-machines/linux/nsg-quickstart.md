---
title: 使用 Azure CLI 對 Linux VM 開啟連接埠 | Microsoft Docs
description: 了解如何使用 Azure Resource Manager 部署模型和 Azure CLI 對 Linux VM 開啟連接埠 / 建立端點
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
ms.assetid: eef9842b-495a-46cf-99a6-74e49807e74e
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 12/13/2017
ms.author: cynthn
ms.openlocfilehash: 4244520fa30fece53e1d01e50044cfeca496066c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46973332"
---
# <a name="open-ports-and-endpoints-to-a-linux-vm-with-the-azure-cli"></a>使用 Azure CLI 針對 Linux VM 開啟連接埠和端點

您可以透過在子網路或 VM 網路介面上建立網路篩選，對 Azure 中的虛擬機器 (VM) 開啟連接埠或建立端點。 您可將控制輸入和輸出流量的這些篩選器放在可接收流量的資源所附加的網路安全性群組上。 讓我們使用連接埠 80 上的 Web 流量的常見範例。 這篇文章說明如何使用 Azure CLI 來開啟連接埠至 VM。 


若要建立網路安全性群組和規則，您需要安裝 [Azure CLI](/cli/azure/install-az-cli2)，並且使用 [az login](/cli/azure/reference-index#az_login) 登入 Azure 帳戶。

在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包括 myResourceGroup、myNetworkSecurityGroup 和 myVnet。


## <a name="quickly-open-a-port-for-a-vm"></a>快速開啟 VM 的連接埠
如果您需要在開發/測試案例中快速開啟 VM 的連接埠，您可以使用 [az vm open-port](/cli/azure/vm#az_vm_open_port) 命令。 此命令會建立網路安全性群組、新增規則，並將其套用到 VM 或子網路。 下列範例會在名為 myResourceGroup 資源群組中名為 myVM 的 VM 上開啟連接埠 80。

```azure-cli
az vm open-port --resource-group myResourceGroup --name myVM --port 80
```

若要更深入地控制規則，例如定義來源 IP 位址範圍，請繼續進行本文中的其他步驟。


## <a name="create-a-network-security-group-and-rules"></a>建立網路安全性群組和規則
使用 [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create) 建立網路安全性群組。 下列範例會在 eastus 位置中建立名為 myNetworkSecurityGroup 的網路安全性群組：

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --location eastus \
    --name myNetworkSecurityGroup
```

使用 [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create)新增規則以允許流向您 Web 伺服器的 HTTP 流量 (或針對自己的案例 (例如 SSH 存取或資料庫連接) 進行調整)。 下列範例會建立名為 myNetworkSecurityGroupRule 的規則以允許連接埠 80 上的 TCP 流量︰

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRule \
    --protocol tcp \
    --priority 1000 \
    --destination-port-range 80
```


## <a name="apply-network-security-group-to-vm"></a>將網路安全性群組套用至 VM
使用 [az network nic update](/cli/azure/network/nic#az_network_nic_update)將「網路安全性群組」與 VM 的網路介面 (NIC) 建立關聯。 下列範例將名為 myNic 的現有 NIC 與名為 myNetworkSecurityGroup 的網路安全性群組建立關聯：

```azurecli
az network nic update \
    --resource-group myResourceGroup \
    --name myNic \
    --network-security-group myNetworkSecurityGroup
```

或者，您也可以使用 [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update)將「網路安全性群組」與虛擬網路的子網路建立關聯，而不是只與單一 VM 上的網路介面建立關聯。 下列範例將 myVnet 虛擬網路中名為 mySubnet 的現有子網路，與名為 myNetworkSecurityGroup 的網路安全性群組建立關聯：

```azurecli
az network vnet subnet update \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name mySubnet \
    --network-security-group myNetworkSecurityGroup
```

## <a name="more-information-on-network-security-groups"></a>網路安全性群組的詳細資訊
這裡的快速命令可讓您使流向您 VM 的流量開始正常運作。 「網路安全性群組」提供許多絕佳的功能和細微性來控制對您資源的存取。 您可以深入了解 [建立網路安全性群組和 ACL 規則](tutorial-virtual-network.md#secure-network-traffic)。

針對高可用性 Web 應用程式，您應該將 VM 放在 Azure Load Balancer 後方。 此負載平衡器會將流量分散到所有 VM，並具有提供流量篩選的網路安全性群組。 如需詳細資訊，請參閱[如何平衡 Azure 中 Linux 虛擬機器的負載以建立高可用性應用程式](tutorial-load-balancer.md)。

## <a name="next-steps"></a>後續步驟
在此範例中，您建立了簡單的規則來允許 HTTP 流量。 您可以從下列文章中，找到有關建立更詳細環境的資訊︰

* [Azure Resource Manager 概觀](../../azure-resource-manager/resource-group-overview.md)
* [什麼是網路安全性群組 (NSG)？](../../virtual-network/security-overview.md)
