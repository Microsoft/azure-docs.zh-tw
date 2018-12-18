---
title: 搭配 Azure CLI 使用內部 DNS 進行 VM 名稱解析 | Microsoft Docs
description: 如何搭配 Azure CLI 在 Azure 上建立虛擬網路介面卡並使用內部 DNS 進行 VM 名稱解析
services: virtual-machines-linux
documentationcenter: ''
author: vlivech
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: article
ms.date: 02/16/2017
ms.author: v-livech
ms.openlocfilehash: acfdd9070b49805c20b8ef921b5387c151448aa1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46961496"
---
# <a name="create-virtual-network-interface-cards-and-use-internal-dns-for-vm-name-resolution-on-azure"></a>在 Azure 上建立虛擬網路介面卡並使用內部 DNS 進行 VM 名稱解析

此文章說明如何搭配 Azure CLI，使用虛擬網路介面卡 (VNic) 與 DNS 標籤名稱來設定 Linux VM 的靜態內部 DNS 名稱。 靜態 DNS 名稱是用於永久基礎結構服務，例如 Jenkins 組建伺服器，它用於這份文件或 Git 伺服器。

這些需求包括：

* [一個 Azure 帳戶](https://azure.microsoft.com/pricing/free-trial/)
* [SSH 公開金鑰和私密金鑰檔案](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="quick-commands"></a>快速命令
如果您需要快速完成工作，接下來這一節將詳細說明所需的命令。 每個步驟的詳細資訊和內容可在文件其他地方找到，從[這裡](#detailed-walkthrough)開始。 若要執行這些步驟，您需要安裝最新的 [Azure CLI](/cli/azure/install-az-cli2)，並且使用 [az login](/cli/azure/reference-index#az_login) 來登入 Azure 帳戶。

先決條件︰資源群組、虛擬網路與子網路、具有 SSH 輸入的網路安全性群組。

### <a name="create-a-virtual-network-interface-card-with-a-static-internal-dns-name"></a>使用靜態內部 DNS 名稱來建立虛擬網路介面卡
使用 [az network nic create](/cli/azure/network/nic#az_network_nic_create) 來建立 vNic。 `--internal-dns-name` CLI 旗標是用來設定 DNS 標籤，可為虛擬網路介面卡 (vNic) 提供靜態 DNS 名稱。 下列範例會建立名為 `myNic` 的 vNic、將它連接到 `myVnet` 虛擬網路，以及建立名為 `jenkins` 的內部 DNS 名稱記錄：

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --internal-dns-name jenkins
```

### <a name="deploy-a-vm-and-connect-the-vnic"></a>部署 VM 並連接 vNic
使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 在部署至 Azure 的期間，`--nics` 旗標會將 vNic 連接到 VM。 下列範例會使用「Azure 受控磁碟」建立名為 `myVM` 的 VM，然後連結上一步驟中名為 `myNic` 的 vNic：

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --nics myNic \
    --image UbuntuLTS \
    --admin-username azureuser \
    --ssh-key-value ~/.ssh/id_rsa.pub
```

## <a name="detailed-walkthrough"></a>詳細的逐步解說

Azure 上完整的連續整合和連續部署 (CiCd) 基礎結構需要特定的伺服器是靜態或長時間執行的伺服器。 虛擬網路和「網路安全性群組」之類的 Azure 資產，最好是當作很少部署的靜態且長久資源。 虛擬網路部署之後可供新的部署重複使用，完全不會對基礎結構造成負面影響。 您可以稍後為您的開發或測試環境，將 Git 儲存機制伺服器或是會傳遞 CiCd 的 Jenkins 自動化伺服器新增到此虛擬網路中。  

僅可在 Azure 虛擬網路內解析內部 DNS 名稱。 因為 DNS 名稱是內部的，它們不會解析至外部網際網路，為基礎結構提供額外安全性。

在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包含 `myResourceGroup`、`myNic` 和 `myVM`。

## <a name="create-the-resource-group"></a>建立資源群組
首先，使用 [az group create](/cli/azure/group#az_group_create)建立資源群組。 下列範例會在 `westus` 位置建立名為 `myResourceGroup` 的資源群組：

```azurecli
az group create --name myResourceGroup --location westus
```

## <a name="create-the-virtual-network"></a>建立虛擬網路

下一步是建置要在其中啟動 VM 的虛擬網路。 在本逐步解說中，此虛擬網路包含一個子網路。 如需 Azure 虛擬網路的詳細資訊，請參閱[建立虛擬網路](../../virtual-network/manage-virtual-network.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json#create-a-virtual-network)。 

使用 [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) 建立虛擬網路。 下列範例會建立名為 `myVnet` 的虛擬網路和名為 `mySubnet` 的子網路：

```azurecli
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 192.168.1.0/24
```

## <a name="create-the-network-security-group"></a>建立網路安全性群組
「Azure 網路安全性群組」相當於網路層的防火牆。 如需有關「網路安全性群組」的詳細資訊，請參閱[如何在 Azure CLI 中建立 NSG](../../virtual-network/tutorial-filter-network-traffic-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 

使用 [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create) 建立網路安全性群組。 下列範例會建立名為 `myNetworkSecurityGroup` 的網路安全性群組：

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup
```

## <a name="add-an-inbound-rule-to-allow-ssh"></a>新增輸入規則以允許 SSH
使用 [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create) 新增網路安全性群組的連入規則。 下列範例會建立名為 `myRuleAllowSSH` 的規則：

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myRuleAllowSSH \
    --protocol tcp \
    --direction inbound \
    --priority 1000 \
    --source-address-prefix '*' \
    --source-port-range '*' \
    --destination-address-prefix '*' \
    --destination-port-range 22 \
    --access allow
```

## <a name="associate-the-subnet-with-the-network-security-group"></a>將子網路與網路安全性群組建立關聯
若要將將子網路與「網路安全性群組」建立關聯，請使用 [az network vnet subnet update](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_update)。 下列範例會將子網路名稱 `mySubnet` 與名為 `myNetworkSecurityGroup` 的「網路安全性群組」建立關聯：

```azurecli
az network vnet subnet update \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name mySubnet \
    --network-security-group myNetworkSecurityGroup
```


## <a name="create-the-virtual-network-interface-card-and-static-dns-names"></a>建立虛擬網路介面卡與靜態 DNS 名稱
Azure 非常有彈性，但是若要使用 DNS 名稱來進行 VM 名稱解析，您必須建立包含 DNS 標籤的虛擬網路介面卡 (VNic)。 VNic 很重要，因為您可以在整個基礎結構週期中，藉由將它們連接到不同的 VM 來重複使用它們。 此方法既可讓 VM 為暫時性，又可將 vNic 保持為靜態資源。 藉由使用 vNic 上的 DNS 標籤，我們便能夠從 VNet 中的其他 VM 啟用簡單名稱解析。 使用可解析的名稱，會讓其他 VM 依據 DNS 名稱 `Jenkins` 存取自動化伺服器，或以 `gitrepo` 存取 Git 伺服器。  

使用 [az network nic create](/cli/azure/network/nic#az_network_nic_create) 來建立 vNic。 下列範例會建立名為 `myNic` 的 vNic、將它連接到名為 `myVnet` 的 `myVnet` 虛擬網路，以及建立名為 `jenkins` 的內部 DNS 名稱記錄：

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --internal-dns-name jenkins
```

## <a name="deploy-the-vm-into-the-virtual-network-infrastructure"></a>將 VM 部署至虛擬網路基礎結構
我們現在有一個虛擬網路和子網路、一個作為防火牆而會封鎖所有輸入流量 (用於 SSH 的連接埠 22 除外) 來保護子網路的「網路安全性群組」，以及一個 vNic。 您現在可以在這個現有的網路基礎結構內部署 VM。

使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 下列範例會使用「Azure 受控磁碟」建立名為 `myVM` 的 VM，然後連結上一步驟中名為 `myNic` 的 vNic：

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --nics myNic \
    --image UbuntuLTS \
    --admin-username azureuser \
    --ssh-key-value ~/.ssh/id_rsa.pub
```

我們使用 CLI 旗標來呼叫現有的資源，以指示 Azure 將 VM 部署在現有的網路內。 重申一次，VNet 和子網路部署之後，就可以在 Azure 區域內保持為靜態或永久性資源。  

## <a name="next-steps"></a>後續步驟
* [直接使用 Azure CLI 命令，建立自訂的 Linux VM 環境](create-cli-complete.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [使用範本在 Azure 上建立 Linux VM](create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
