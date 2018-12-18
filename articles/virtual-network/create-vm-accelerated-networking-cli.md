---
title: 使用加速網路來建立 Azure 虛擬機器 | Microsoft Docs
description: 了解如何建立已啟用加速網路的 Linux 虛擬機器。
services: virtual-network
documentationcenter: na
author: gsilva5
manager: gedegrac
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/02/2018
ms.author: gsilva
ms.custom: ''
ms.openlocfilehash: b6aaf98ca3b5581691b6c70783be5250b506056c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990955"
---
# <a name="create-a-linux-virtual-machine-with-accelerated-networking"></a>使用加速網路建立 Linux 虛擬機器

在本教學課程中，您將了解如何使用加速網路建立 Linux 虛擬機器 (VM)。 若要建立使用加速網路的 Windows VM，請參閱[建立使用加速網路的 Windows VM](create-vm-accelerated-networking-powershell.md)。 加速網路可以對 VM 啟用 Single Root I/O Virtualization (SR-IOV)，大幅提升其網路效能。 這個高效能路徑會略過資料路徑的主機，進而減少延遲、抖動和 CPU 使用率，供支援的 VM 類型中最嚴苛的網路工作負載使用。 下圖顯示兩部 VM 之間的通訊，一部具備加速網路而另一步沒有︰

![比較](./media/create-vm-accelerated-networking/accelerated-networking.png)

如果沒有加速網路，進出 VM 的所有網路流量都必須周遊主機和虛擬交換器。 虛擬交換器對網路流量提供所有原則強制執行，例如網路安全性群組、存取控制清單、隔離性以及其他網路虛擬化服務。 若要深入了解虛擬交換器，請閱讀 [Hyper-V Network Virtualization and Virtual Switch (Hyper-V 網路虛擬化和虛擬交換器)](https://technet.microsoft.com/library/jj945275.aspx) 文章。

如果使用加速網路，網路流量就會先送達 VM 的網路介面 (NIC)，然後轉送到 VM。 虛擬交換器套用的所有網路原則現在皆已卸載，並在硬體中套用。 在硬體中套用原則會讓 NIC 略過主機和虛擬交換器，同時在主機中維護套用的所有原則，直接將網路流量轉送到 VM。

加速網路的優點只適用於已啟用此功能的 VM。 為了獲得最佳結果，最好在至少兩部連線到相同 Azure 虛擬網路 (VNet) 的 VM 上啟用此功能。 當透過 VNet 通訊或連線內部部署時，此功能對整體延遲的影響可以降到最低。

## <a name="benefits"></a>優點
* **較低的延遲 / 較高的每秒封包數目 (pps)：** 從資料路徑移除虛擬交換器會減少主機中封包在處理原則時所花的時間，並增加 VM 內可處理的封包數目。
* **減少抖動︰** 虛擬交換器處理視需要套用的原則數量和正在進行處理的 CPU 工作負載而定。 將原則強制執行卸載到硬體透過將封包直接傳遞到 VM、移除主機到 VM 的通訊，以及所有軟體插斷和環境切換，而減少變化。
* **降低 CPU 使用率︰** 略過主機中的虛擬交換器可減少處理網路流量的 CPU 使用率。

## <a name="supported-operating-systems"></a>受支援的作業系統
您可以從 Azure 資源庫直接使用下列發行版本： 
* **Ubuntu 16.04** 
* **SLES 12 SP3** 
* **RHEL 7.4**
* **CentOS 7.4**
* **CoreOS Linux**
* **具有反向移植核心的 Debian "Stretch"**
* **Oracle Linux 7.4**

## <a name="limitations-and-constraints"></a>限制和條件約束

### <a name="supported-vm-instances"></a>支援的 VM 執行個體
大多數一般用途和具有 2 個以上 vCPU 的計算最佳化執行個體大小，皆支援加速網路。  這些支援的系列為：D/DSv2 和 F/Fs

在支援超執行緒的執行個體中，加速網路可在具有 4 個以上 vCPU 的 VM 執行個體上進行支援作業。 支援的系列為：D/DSv3、E/ESv3、Fsv2 和 Ms/Mms。

如需 VM 執行個體的詳細資訊，請參閱 [Linux VM 大小](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

### <a name="regions"></a>區域
適用於所有公用 Azure 區域和 Azure 政府雲端。

### <a name="network-interface-creation"></a>網路介面建立 
您只能對新的 NIC 啟用加速網路。 無法對現有 NIC 來啟用。
### <a name="enabling-accelerated-networking-on-a-running-vm"></a>在執行中的 VM 上啟用加速網路
支援的 VM 大小若沒啟用加速網路，則只能在停止或解除配置 VM 時啟用此功能。  
### <a name="deployment-through-azure-resource-manager"></a>透過 Azure Resource Manager 進行部署
虛擬機器 (傳統) 無法透過加速網路部署。

## <a name="create-a-linux-vm-with-azure-accelerated-networking"></a>建立使用 Azure 加速網路的 Linux VM

雖然本文提供使用 Azure CLI 來建立具有加速網路之虛擬機器的步驟，但您也可以[使用 Azure 入口網站來建立具有加速網路的虛擬機器](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 在入口網站中建立虛擬機器時，請在 [設定] 下的 [加速的網路] 底下，選取 [已啟用]。 除非您已選取[支援的作業系統](#supported-operating-systems)和 [VM 大小](#supported-vm-instances)，否則啟用加速網路的選項不會出現在入口網站中。 建立虛擬機器之後，您必須完成[確認加速網路已確實啟用](#confirm-that-accelerated-networking-is-enabled)中的指示動作。

### <a name="create-a-virtual-network"></a>建立虛擬網路

請安裝最新的 [Azure CLI](/cli/azure/install-azure-cli)，並使用 [az login](/cli/azure/reference-index#az_login) 來登入 Azure 帳戶。 在下列範例中，請以您自己的值取代範例參數名稱。 範例參數名稱包含 *myResourceGroup*、*myNic* 和 *myVm*。

使用 [az group create](/cli/azure/group#az_group_create) 來建立資源群組。 下列範例會在 *centralus* 位置建立名為 *myResourceGroup* 的資源群組：

```azurecli
az group create --name myResourceGroup --location centralus
```

選取 [Linux 加速網路](https://azure.microsoft.com/updates/accelerated-networking-in-expanded-preview)中所列出的支援 Linux 區域。

使用 [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) 建立虛擬網路。 下列範例會建立名為 *myVnet* 的虛擬網路和子網路：

```azurecli
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefix 192.168.0.0/16 \
    --subnet-name mySubnet \
    --subnet-prefix 192.168.1.0/24
```

### <a name="create-a-network-security-group"></a>建立網路安全性群組
使用 [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create) 建立網路安全性群組。 下列範例建立名為 myNetworkSecurityGroup 的網路安全性群組：

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup
```

網路安全性群組包含數個預設規則，其中一種會停用來自網際網路的所有輸入存取。 開啟連接埠以允許 SSH 透過 [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create) 存取虛擬機器：

```azurecli
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNetworkSecurityGroup \
  --name Allow-SSH-Internet \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --source-address-prefix Internet \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range 22
```

### <a name="create-a-network-interface-with-accelerated-networking"></a>使用加速網路建立網路介面

使用 [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) 建立公用 IP 位址。 如果您不打算從網際網路存取虛擬機器，那麼您就不需要公用 IP 位址，但如果要完成本文中的步驟，就會需要公用 IP 位址。

```azurecli
az network public-ip create \
    --name myPublicIp \
    --resource-group myResourceGroup
```

使用已啟用的加速網路並透過 [az network nic create](/cli/azure/network/nic#az_network_nic_create) 建立網路介面。 下列範例會在 *myVnet* 虛擬網路的 *mySubnet* 子網路中建立名為 *myNic* 的網路介面，並將 *myNetworkSecurityGroup* 網路安全性群組與網路介面建立關聯：

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic \
    --vnet-name myVnet \
    --subnet mySubnet \
    --accelerated-networking true \
    --public-ip-address myPublicIp \
    --network-security-group myNetworkSecurityGroup
```

### <a name="create-a-vm-and-attach-the-nic"></a>建立 VM 並連結 NIC
當您建立 VM 時，請指定您使用 `--nics` 所建立的 NIC。 選取列於 [Linux 加速網路](https://azure.microsoft.com/updates/accelerated-networking-in-expanded-preview)中的大小和發行版本。 

使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 下列範例會使用 UbuntuLTS 映像建立名為 *myVM* 的 VM，以及支援加速網路的大小 (*Standard_DS4_v2*)：

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --size Standard_DS4_v2 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --nics myNic
```

如需所有 VM 大小和特性的清單，請參閱 [Linux VM 大小](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。

建立 VM 後，系統將傳回與下列範例輸出類似的輸出。 請記下 **publicIpAddress**。 在後續步驟中，將會使用此位址來存取 VM。

```azurecli
{
  "fqdns": "",
  "id": "/subscriptions/<ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "centralus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "192.168.0.4",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
}
```

### <a name="confirm-that-accelerated-networking-is-enabled"></a>確認加速網路已確實啟用

使用下列命令來對 VM 建立 SSH 工作階段。 使用指派至您建立之虛擬機器的公用 IP 位址來取代 `<your-public-ip-address>`，如果您在建立 VM 時，為 `--admin-username` 使用不同的值，則請取代 *azureuser*。

```bash
ssh azureuser@<your-public-ip-address>
```

從 Bash 殼層中，輸入 `uname -r`，並確認核心版本是下列其中一個版本或更高版本：

* **Ubuntu 16.04**: 4.11.0-1013
* **SLES SP3**: 4.4.92-6.18
* **RHEL**: 7.4.2017120423
* **CentOS**: 7.4.20171206


確認 Mellanox VF 裝置已使用 `lspci` 命令向 VM 公開。 傳回的輸出大致如下列所示：

```bash
0000:00:00.0 Host bridge: Intel Corporation 440BX/ZX/DX - 82443BX/ZX/DX Host bridge (AGP disabled) (rev 03)
0000:00:07.0 ISA bridge: Intel Corporation 82371AB/EB/MB PIIX4 ISA (rev 01)
0000:00:07.1 IDE interface: Intel Corporation 82371AB/EB/MB PIIX4 IDE (rev 01)
0000:00:07.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 02)
0000:00:08.0 VGA compatible controller: Microsoft Corporation Hyper-V virtual VGA
0001:00:02.0 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
```

使用 `ethtool -S eth0 | grep vf_` 命令檢查 VF (虛擬函式) 上的活動。 如果您收到如下列範例輸出的輸出，則加速網路將會啟用並運作。

```bash
vf_rx_packets: 992956
vf_rx_bytes: 2749784180
vf_tx_packets: 2656684
vf_tx_bytes: 1099443970
vf_tx_dropped: 0
```
現在已啟用您 VM 的加速網路。

## <a name="enable-accelerated-networking-on-existing-vms"></a>在現有的 VM 上啟用加速網路
如果您已建立不含加速網路的 VM，那麼在現有 VM 上啟用此功能是可能的。  VM 必須符合前面也說明過的下列必要條件，才能支援加速網路：

* VM 必須是支援加速網路的大小
* VM 必須是支援的 Azure 資源庫映像 (及 Linux 的核心版本)
* 可用性設定組中的所有 VM 或 VMSS 必須先停止/解除配置，然後才能在任何 NIC 上啟用加速網路

### <a name="individual-vms--vms-in-an-availability-set"></a>個別 VM 與可用性設定組中的多個 VM
首先要停止/解除配置 VM 或可用性設定組中的所有 VM：

```azurecli
az vm deallocate \
    --resource-group myResourceGroup \
    --name myVM
```

請務必注意，如果您的 VM 是個別建立，沒有使用可用性設定組，則您只需要停止/解除配置該個別 VM，即可啟用加速網路。  如果您的 VM 是使用可用性設定組建立的，則包含在可用性設定組中的所有 VM 皆必須先停止/解除配置之後，才能在任何 NIC 上啟用加速網路。 

停止之後，在您 VM 的 NIC 上啟用加速網路：

```azurecli
az network nic update \
    --name myNic \
    --resource-group myResourceGroup \
    --accelerated-networking true
```

重新啟動您 VM 或可用性設定組中的所有 VM，然後確認加速網路已啟用： 

```azurecli
az vm start --resource-group myResourceGroup \
    --name myVM
```

### <a name="vmss"></a>VMSS
VMSS 稍有不同，但會遵循相同的工作流程。  首先，停止 VM：

```azurecli
az vmss deallocate \
    --name myvmss \
    --resource-group myrg
```

停止 VM 後，更新網路介面下的加速網路屬性：

```azurecli
az vmss update --name myvmss \
    --resource-group myrg \
    --set virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].enableAcceleratedNetworking=true
```

請注意，VMSS 會升級 VM，其使用三種不同的設定 (自動、輪流及手動) 來套用更新。  將這些指示中的原則設為自動，以便 VMSS 在重新啟動之後，立即採用變更。  將其設定為自動，讓所做的變更可立即採用： 

```azurecli
az vmss update \
    --name myvmss \
    --resource-group myrg \
    --set upgradePolicy.mode="automatic"
```

最後，重新啟動 VMSS：

```azurecli
az vmss start \
    --name myvmss \
    --resource-group myrg
```

重新啟動後，等候升級完成，但是一旦完成後，VF 會出現在 VM 內。  (請確定您是使用支援的 OS 和 VM 大小。)

### <a name="resizing-existing-vms-with-accelerated-networking"></a>調整使用加速網路的現有 VM 大小

針對已啟用加速網路的 VM，您只能將其大小調整為其他支援加速網路的 VM。  

您無法使用調整作業將已啟用加速網路的 VM 大小，調整為不支援加速網路的 VM 執行個體。  相反的，若要為這些 VM 的其中一個調整大小，請執行下列動作： 

* 停止/解除配置 VM 或可用性設定組/VMSS 中的所有 VM。
* 必須在 VM 或可用性設定組/VMSS 中所有 VM 的 NIC 上停用加速網路。
* 一旦停用加速網路之後，VM/可用性設定組/VMSS 就可以移到不支援加速網路的新大小，並重新啟動。  

