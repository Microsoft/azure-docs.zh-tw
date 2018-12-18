---
title: 在 Azure 中建立連接多個 NIC 的 Linux VM | Microsoft Docs
description: 了解如何使用 Azure CLI 或 Resource Manager 範本，來建立連接多個 NIC 的 Linux VM。
services: virtual-machines-linux
documentationcenter: ''
author: iainfoulds
manager: jeconnoc
editor: ''
ms.assetid: 5d2d04d0-fc62-45fa-88b1-61808a2bc691
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 06/07/2018
ms.author: iainfou
ms.openlocfilehash: 4982de352af2ce33f4dbf6dba00ff9296cc9b873
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46999746"
---
# <a name="how-to-create-a-linux-virtual-machine-in-azure-with-multiple-network-interface-cards"></a>如何在 Azure 中建立有多個網路介面卡的 Linux 虛擬機器


此文章詳述如何使用 Azure CLI 建立具有多個 NIC 的 VM。

## <a name="create-supporting-resources"></a>建立支援資源
請安裝最新的 [Azure CLI](/cli/azure/install-az-cli2) 並使用 [az login](/cli/azure/reference-index#az_login) 來登入 Azure 帳戶。

在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包含 myResourceGroup、mystorageaccount 和 myVM。

首先，使用 [az group create](/cli/azure/group#az_group_create) 建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli
az group create --name myResourceGroup --location eastus
```

使用 [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) 建立虛擬網路。 下列範例會建立名為 myVnet 的虛擬網路和名為 mySubnetFrontEnd 的子網路：

```azurecli
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefix 10.0.0.0/16 \
    --subnet-name mySubnetFrontEnd \
    --subnet-prefix 10.0.1.0/24
```

使用 [az network vnet subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) 建立後端流量的子網路。 下列範例會建立名為 mySubnetBackEnd 的子網路：

```azurecli
az network vnet subnet create \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name mySubnetBackEnd \
    --address-prefix 10.0.2.0/24
```

使用 [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create) 建立網路安全性群組。 下列範例建立名為 myNetworkSecurityGroup 的網路安全性群組：

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup
```

## <a name="create-and-configure-multiple-nics"></a>建立及設定多個 NIC
使用 [az network nic create](/cli/azure/network/nic#az_network_nic_create) 建立兩個 NIC。 下列範例會建立兩個連接網路安全性群組的 NIC (名為 myNic1 和 myNic2)，以及一個連接到各個子網路的 NIC：

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic1 \
    --vnet-name myVnet \
    --subnet mySubnetFrontEnd \
    --network-security-group myNetworkSecurityGroup
az network nic create \
    --resource-group myResourceGroup \
    --name myNic2 \
    --vnet-name myVnet \
    --subnet mySubnetBackEnd \
    --network-security-group myNetworkSecurityGroup
```

## <a name="create-a-vm-and-attach-the-nics"></a>建立 VM 並附加 NIC
當您建立 VM 時，指定您使用 `--nics` 建立的 NIC。 當您選取 VM 大小時也需多加注意。 您可以新增至 VM 的 NIC 總數是有限制的。 深入了解 [Linux VM 大小](sizes.md)。

使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 下列範例會建立名為 myVM 的 VM。

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --size Standard_DS3_v2 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --nics myNic1 myNic2
```

完成[針對多個 NIC 設定客體作業系統](#configure-guest-os-for- multiple-nics)中的步驟，將路由資料表新增至客體作業系統。

## <a name="add-a-nic-to-a-vm"></a>將 NIC 新增至 VM
先前的步驟建立了一個有多個 NIC 的 VM。 您也可以使用 Azure CLI 將 NIC 新增至現有的 VM。 不同的 [VM 大小](sizes.md) 支援不同數量的 NIC，因此可據以調整您的 VM。 如有需要，您可以[調整 VM 的大小](change-vm-size.md)。

使用 [az network nic create](/cli/azure/network/nic#az_network_nic_create) 建立另一個 NIC。 下列範例會建立一個名為 myNic3 的 NIC，此 NIC會連線到後端子網路與在先前步驟中建立的網路安全性群組：

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic3 \
    --vnet-name myVnet \
    --subnet mySubnetBackEnd \
    --network-security-group myNetworkSecurityGroup
```

若要將 NIC 新增至現有的 VM，請先使用 [az vm deallocate](/cli/azure/vm#az_vm_deallocate) 解除配置 VM。 下列範例會解除配置名為 myVM 的 VM：


```azurecli
az vm deallocate --resource-group myResourceGroup --name myVM
```

使用 [az vm nic add](/cli/azure/vm/nic#az_vm_nic_add) 新增 NIC。 下列範例將 myNic3 新增至 myVM：

```azurecli
az vm nic add \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --nics myNic3
```

使用 [az vm start](/cli/azure/vm#az_vm_start) 啟用 VM：

```azurecli
az vm start --resource-group myResourceGroup --name myVM
```

完成[針對多個 NIC 設定客體作業系統](#configure-guest-os-for- multiple-nics)中的步驟，將路由資料表新增至客體作業系統。

## <a name="remove-a-nic-from-a-vm"></a>從 VM 中移除 NIC
若要將 NIC 從現有的 VM 中移除，請先使用 [az vm deallocate](/cli/azure/vm#az_vm_deallocate) 解除配置 VM。 下列範例會解除配置名為 myVM 的 VM：

```azurecli
az vm deallocate --resource-group myResourceGroup --name myVM
```

使用 [az vm nic remove](/cli/azure/vm/nic#az_vm_nic_remove) 移除 NIC。 下列範例會將 myNic3 從 myVM 中移除：

```azurecli
az vm nic remove \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --nics myNic3
```

使用 [az vm start](/cli/azure/vm#az_vm_start) 啟用 VM：

```azurecli
az vm start --resource-group myResourceGroup --name myVM
```


## <a name="create-multiple-nics-using-resource-manager-templates"></a>使用 Resource Manager 範本建立多個 NIC
Azure Resource Manager 範本會使用宣告式 JSON 檔案來定義您的環境。 您可以閱讀 [Azure Resource Manager 概觀](../../azure-resource-manager/resource-group-overview.md)。 Resource Manager 範本提供一種方式，可在部署期間建立資源的多個執行個體，例如建立多個 NIC。 您使用 *copy* 來指定要建立的執行個體數目：

```json
"copy": {
    "name": "multiplenics"
    "count": "[parameters('count')]"
}
```

深入了解[使用 *copy* 建立多個執行個體](../../resource-group-create-multiple.md)。 

您也可以使用 `copyIndex()`，然後在資源名稱後面附加一個數字，讓您能夠建立 `myNic1`、`myNic2`，依此類推。以下顯示附加索引值的範例：

```json
"name": "[concat('myNic', copyIndex())]", 
```

您可以閱讀 [使用 Resource Manager 範本建立多個 NIC](../../virtual-network/template-samples.md)的完整範例。

完成[針對多個 NIC 設定客體作業系統](#configure-guest-os-for- multiple-nics)中的步驟，將路由資料表新增至客體作業系統。

## <a name="configure-guest-os-for-multiple-nics"></a>針對多個 NIC 設定客體作業系統

先前的步驟已建立虛擬網路和子網路、連結 NIC，然後建立 VM。 並未建立公用 IP 位址以及允許 SSH 流量的網路安全性群組規則。 若要針對多個 NIC 設定客體作業系統，您需要允許遠端連線，並且在 VM 本機執行命令。

若要允許 Web 流量，請使用 [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create) 建立網路安全性群組規則，如下所示：

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name allow_ssh \
    --priority 101 \
    --destination-port-ranges 22
```

使用 [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) 建立公用 IP 位址，並使用 [az network nic ip-config update](/cli/azure/network/nic/ip-config#az-network-nic-ip-config-update) 將它指派給第一個 NIC：

```azurecli
az network public-ip-address create --resource-group myResourceGroup --name myPublicIP

az network nic ip-config update \
    --resource-group myResourceGroup \
    --nic-name myNic1 \
    --name ipconfig1 \
    --public-ip-addres myPublicIP
```

若要檢視 VM 的公用 IP 位址，請使用 [az vm show](/cli/azure/vm#az-vm-show)，如下所示：

```azurecli
az vm show --resource-group myResourceGroup --name myVM -d --query publicIps -o tsv
```

現在透過 SSH 連線至 VM 的公用 IP 位址。 上一個步驟中提供的預設使用者名稱為 azureuser。 提供您自己的使用者名稱和公用 IP 位址：

```bash
ssh azureuser@137.117.58.232
```

若要對次要網路介面進行雙向傳送，您必須手動將持續性路由新增至每個次要網路介面的作業系統。 在本文中，eth1 是次要介面。 將持續性路由新增至作業系統的指示，會因散發套件而有所不同。 如需相關指示，請參閱您的散發套件文件。

將路由新增至作業系統時，無論網路介面位於哪個子網路，閘道位址均為 *.1*。 例如，如果網路介面獲派 10.0.2.4 地址，則您為路由指定的閘道為 10.0.2.1。 您可以為路由的目的地定義特定網路；或是如果希望介面的所有流量通過指定的閘道，請指定目的地為 0.0.0.0。 每個子網路的閘道均由虛擬網路管理。

新增次要介面的路由後，請使用 `route -n` 確認路由位於您的路由表中。 下列範例輸出適用於本文中將兩個網路介面新增至 VM 的路由表：

```bash
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.1.1        0.0.0.0         UG    0      0        0 eth0
0.0.0.0         10.0.2.1        0.0.0.0         UG    0      0        0 eth1
10.0.1.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 eth1
168.63.129.16   10.0.1.1        255.255.255.255 UGH   0      0        0 eth0
169.254.169.254 10.0.1.1        255.255.255.255 UGH   0      0        0 eth0
```

請在重新開機後再次檢查您的路由表，透過重新啟動確認您新增的路由持續存在。 若要測試連線能力，您可以輸入下列命令，其中 eth1 是次要網路介面的名稱：

```bash
ping bing.com -c 4 -I eth1
```

## <a name="next-steps"></a>後續步驟
當您嘗試建立一個有多個 NIC 的 VM 時，請檢閱 [Linux VM 大小](sizes.md)。 注意每個 VM 大小所支援的 NIC 數目上限。

若要進一步保護您的 VM，請使用 Just-In-Time 虛擬機器存取。 這項功能會視需要開啟 SSH 流量的網路安全性群組規則，並持續一段定義的期間。 如需詳細資訊，請參閱[使用 Just-In-Time 管理虛擬機器存取](../../security-center/security-center-just-in-time.md)。
