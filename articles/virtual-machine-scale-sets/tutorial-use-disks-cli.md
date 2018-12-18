---
title: 教學課程 - 使用 Azure CLI 建立及使用擴展集所適用的磁碟 | Microsoft Docs
description: 了解如何使用 Azure CLI 來建立及使用虛擬機器擴展集所適用的受控磁碟，包括如何新增、準備、列出及中斷連結磁碟。
services: virtual-machine-scale-sets
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: ae26e851e60f654a502779fe2a8b41e13dc47915
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993624"
---
# <a name="tutorial-create-and-use-disks-with-virtual-machine-scale-set-with-the-azure-cli"></a>教學課程：使用 Azure CLI 建立及使用虛擬機器擴展集所適用的磁碟
虛擬機器擴展集會使用磁碟來儲存 VM 執行個體的作業系統、應用程式和資料。 當您建立及管理擴展集時，請務必選擇預期的工作負載所適用的磁碟大小和組態。 此教學課程將說明如何建立及管理 VM 磁碟。 在此教學課程中，您將了解如何：

> [!div class="checklist"]
> * OS 磁碟和暫存磁碟
> * 資料磁碟
> * 標準和進階磁碟
> * 磁碟效能
> * 連結及準備資料磁碟

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，則在此教學課程中，您必須執行 Azure CLI 2.0.29 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。


## <a name="default-azure-disks"></a>預設 Azure 磁碟
建立或調整擴展集後，有兩個磁碟會自動連結到各個 VM 執行個體。

**作業系統磁碟** - 作業系統磁碟可裝載 VM 執行個體的作業系統，其大小可以高達 2 TB。 OS 磁碟預設會標示為 /dev/sda。 OS 磁碟的磁碟快取組態已針對 OS 效能進行最佳化。 因為此組態，OS 磁碟**不得**裝載應用程式或資料。 請對應用程式和資料使用資料磁碟，此文章稍後會詳細說明。

**暫存磁碟** - 暫存磁碟會使用與 VM 執行個體位於相同 Azure 主機的固態磁碟機。 暫存磁碟的效能非常好，可用於暫存資料處理等作業。 不過，如果 VM 執行個體移至新的主機，則會移除儲存在暫存磁碟上的任何資料。 暫存磁碟的大小取決於 VM 執行個體大小。 暫存磁碟會標示為 /dev/sdb，其掛接點為 /mnt。

### <a name="temporary-disk-sizes"></a>暫存磁碟大小
| 類型 | 一般大小 | 暫存磁碟大小上限 (GiB) |
|----|----|----|
| [一般用途](../virtual-machines/linux/sizes-general.md) | A、B 和 D 系列 | 1600 |
| [計算最佳化](../virtual-machines/linux/sizes-compute.md) | F 系列 | 576 |
| [記憶體最佳化](../virtual-machines/linux/sizes-memory.md) | D、E、G 和 M 系列 | 6144 |
| [儲存體最佳化](../virtual-machines/linux/sizes-storage.md) | L 系列 | 5630 |
| [GPU](../virtual-machines/linux/sizes-gpu.md) | N 系列 | 1440 |
| [高效能](../virtual-machines/linux/sizes-hpc.md) | A 和 H 系列 | 2000 |


## <a name="azure-data-disks"></a>Azure 資料磁碟
如果您需要安裝應用程式和儲存資料，您可以新增額外的資料磁碟。 資料磁碟應使用於任何需要持久且有回應之資料儲存體的情況。 每個資料磁碟皆具有 4 TB 的最大容量。 VM 執行個體的大小會決定可連結的資料磁碟數目。 每個 VM vCPU 可以連結兩個資料磁碟。

### <a name="max-data-disks-per-vm"></a>每部 VM 的資料磁碟上限
| 類型 | 一般大小 | 每部 VM 的資料磁碟上限 |
|----|----|----|
| [一般用途](../virtual-machines/linux/sizes-general.md) | A、B 和 D 系列 | 64 |
| [計算最佳化](../virtual-machines/linux/sizes-compute.md) | F 系列 | 64 |
| [記憶體最佳化](../virtual-machines/linux/sizes-memory.md) | D、E、G 和 M 系列 | 64 |
| [儲存體最佳化](../virtual-machines/linux/sizes-storage.md) | L 系列 | 64 |
| [GPU](../virtual-machines/linux/sizes-gpu.md) | N 系列 | 64 |
| [高效能](../virtual-machines/linux/sizes-hpc.md) | A 和 H 系列 | 64 |


## <a name="vm-disk-types"></a>VM 磁碟類型
Azure 提供兩種類型的磁碟。

### <a name="standard-disk"></a>標準磁碟
標準儲存體以 HDD 作為後盾，既可提供符合成本效益的儲存體，又可保有效能。 標準磁碟適合用於具成本效益的開發和測試工作負載。

### <a name="premium-disk"></a>進階磁碟
進階磁碟是以 SSD 為基礎的高效能、低延遲磁碟為後盾。 建議將這些磁碟用於執行生產工作負載的 VM。 進階儲存體支援 DS 系列、DSv2 系列、GS 系列和 FS 系列 VM。 當您選取磁碟大小時，其值會上調為下一個類型。 例如，如果磁碟大小少於 128 GB，則磁碟類型為 P10。 如果磁碟大小介於 129 GB 與 512 GB 之間，則大小為 P20。 如果超過 512 GB，則大小為 P30。

### <a name="premium-disk-performance"></a>進階磁碟效能
|進階儲存體磁碟類型 | P4 | P6 | P10 | P20 | P30 | P40 | P50 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 磁碟大小 (上調) | 32 GB | 64 GB | 128 GB | 512 GB | 1,024 GB (1 TB) | 2,048 GB (2 TB) | 4,095 GB (4 TB) |
| 每一磁碟的 IOPS 上限 | 120 | 240 | 500 | 2,300 | 5,000 | 7,500 | 7,500 |
每一磁碟的輸送量 | 25 MB/秒 | 50 MB/秒 | 100 MB/秒 | 150 MB/秒 | 200 MB/秒 | 250 MB/秒 | 250 MB/秒 |

雖然上表指出每個磁碟的最大 IOPS，但可藉由分割多個資料磁碟來達到較高等級的效能。 例如，Standard_GS5 VM 最高可達到 80,000 IOPS。 如需每部 VM 的最大 IOPS 詳細資訊，請參閱 [Linux VM 大小](../virtual-machines/linux/sizes.md)。


## <a name="create-and-attach-disks"></a>建立和連結磁碟
您可以在建立擴展集時建立並連結磁碟，或使用現有的擴展集。

### <a name="attach-disks-at-scale-set-creation"></a>在建立擴展集時連結磁碟
首先，使用 [az group create](/cli/azure/group#az_group_create) 命令來建立資源群組。 在此範例中，會在 eastus 區域中建立名為 myResourceGroup 的資源群組。

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

使用 [az vmss create](/cli/azure/vmss#az_vmss_create) 命令建立虛擬機器擴展集。 下列範例會建立名為 myScaleSet 的擴展集，以及產生 SSH 金鑰 (如果不存在)。 系統會使用 `--data-disk-sizes-gb` 參數建立兩個磁碟。 第一個磁碟的大小為 *64* GB，第二個磁碟則為 *128* GB：

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy automatic \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 64 128
```

建立及設定所有擴展集資源和 VM 執行個體需要幾分鐘的時間。

### <a name="attach-a-disk-to-existing-scale-set"></a>將磁碟連結至現有的擴展集
您也可以將磁碟連結至現有的擴展集。 請使用在先前的步驟中建立的擴展集，透過 [az vmss disk attach](/cli/azure/vmss/disk#az_vmss_disk_attach) 新增另一個磁碟。 下列範例會連結另一個 *128* GB 的磁碟：

```azurecli-interactive
az vmss disk attach \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --size-gb 128
```


## <a name="prepare-the-data-disks"></a>準備資料磁碟
您所建立並連結至擴展集 VM 執行個體的磁碟，是原始磁碟。 磁碟必須完成準備工作，才能用於您的資料與應用程式。 若要準備磁碟，您可以建立分割區、建立檔案系統，並將其掛接。

若要自動執行跨擴展集中多個 VM 執行個體的程序，您可以使用 Azure 自訂指令碼延伸模組。 此延伸模組可在每個 VM 執行個體的本機位置執行指令碼，以準備已連結的資料磁碟 (舉例而言)。 如需詳細資訊，請參閱[自訂指令碼延伸模組概觀](../virtual-machines/linux/extensions-customscript.md)。

下列範例會使用 [az vmss extension set](/cli/azure/vmss/extension#az_vmss_extension_set) 在每個 VM 執行個體上執行 GitHub 範例存放庫中的指令碼，以準備所有已連結的原始資料磁碟：

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/prepare_vm_disks.sh"],"commandToExecute":"./prepare_vm_disks.sh"}'
```

若要確認磁碟已正確備妥，請透過 SSH 連線至其中一個 VM 執行個體。 請使用 [az vmss list-instance-connection-info](/cli/azure/vmss#az_vmss_list_instance_connection_info) 列出擴展集的連線資訊：

```azurecli-interactive
az vmss list-instance-connection-info \
  --resource-group myResourceGroup \
  --name myScaleSet
```

使用您自己公用 IP 位址和連接埠號碼連線至第一個 VM 執行個體，如下列範例所示：

```azurecli-interactive
ssh azureuser@52.226.67.166 -p 50001
```

檢查 VM 執行個體上的分割區，如下所示：

```bash
sudo fdisk -l
```

下列範例輸出顯示三個磁碟已連結至 VM 執行個體 - */dev/sdc*、*/dev/sdd* 和 */dev/sde*。 這三個磁碟都有可使用所有可用空間的單一分割區：

```bash
Disk /dev/sdc: 64 GiB, 68719476736 bytes, 134217728 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa47874cb

Device     Boot Start       End   Sectors Size Id Type
/dev/sdc1        2048 134217727 134215680  64G 83 Linux

Disk /dev/sdd: 128 GiB, 137438953472 bytes, 268435456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd5c95ca0

Device     Boot Start       End   Sectors  Size Id Type
/dev/sdd1        2048 268435455 268433408  128G 83 Linux

Disk /dev/sde: 128 GiB, 137438953472 bytes, 268435456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x9312fdf5

Device     Boot Start       End   Sectors  Size Id Type
/dev/sde1        2048 268435455 268433408  128G 83 Linux
```

檢查 VM 執行個體的檔案系統和掛接點，如下所示：

```bash
sudo df -h
```

下列範例輸出顯示這三個磁碟已正確地將其檔案系統掛接在 */datadisks* 下：

```bash
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        30G  1.3G   28G   5% /
/dev/sdb1        50G   52M   47G   1% /mnt
/dev/sdc1        63G   52M   60G   1% /datadisks/disk1
/dev/sdd1       126G   60M  120G   1% /datadisks/disk2
/dev/sde1       126G   60M  120G   1% /datadisks/disk3
```

您的擴展集中每個 VM 執行個體上的磁碟都會以相同的方式自動準備。 當您的擴展集要相應增加時，必要的資料磁碟會連結至新的 VM 執行個體。 此外也會執行自訂指令碼延伸模組，以自動準備磁碟。

關閉 VM 執行個體的 SSH 連線：

```bash
exit
```


## <a name="list-attached-disks"></a>列出已連結的磁碟
若要檢視已連結至擴展集之磁碟的相關資訊，請使用 [az vmss show](/cli/azure/vmss#az_vmss_show) 並查詢 *virtualMachineProfile.storageProfile.dataDisks*：

```azurecli-interactive
az vmss show \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --query virtualMachineProfile.storageProfile.dataDisks
```

此時會顯示磁碟大小、儲存層和 LUN (邏輯單元編號) 的相關資訊。 下列範例輸出會詳盡顯示三個連結至擴展集的資料磁碟：

```json
[
  {
    "additionalProperties": {},
    "caching": "None",
    "createOption": "Empty",
    "diskSizeGb": 64,
    "lun": 0,
    "managedDisk": {
      "additionalProperties": {},
      "storageAccountType": "Standard_LRS"
    },
    "name": null
  },
  {
    "additionalProperties": {},
    "caching": "None",
    "createOption": "Empty",
    "diskSizeGb": 128,
    "lun": 1,
    "managedDisk": {
      "additionalProperties": {},
      "storageAccountType": "Standard_LRS"
    },
    "name": null
  },
  {
    "additionalProperties": {},
    "caching": "None",
    "createOption": "Empty",
    "diskSizeGb": 128,
    "lun": 2,
    "managedDisk": {
      "additionalProperties": {},
      "storageAccountType": "Standard_LRS"
    },
    "name": null
  }
]
```


## <a name="detach-a-disk"></a>中斷連結磁碟
當您不再需要給定的磁碟時，您可以從擴展集將其中斷連結。 磁碟會從擴展集中的所有 VM 執行個體移除。 若要從擴展集將磁碟中斷連結，請使用 [az vmss disk detach](/cli/azure/vmss/disk#az_vmss_disk_detach)，並指定磁碟的 LUN。 LUN 會顯示在上一節提及之 [az vmss show](/cli/azure/vmss#az_vmss_show) 的輸出中。 下列範例會從擴展集將 LUN *2* 中斷連結：

```azurecli-interactive
az vmss disk detach \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --lun 2
```


## <a name="clean-up-resources"></a>清除資源
若要移除您的擴展集和磁碟，請使用 [az group delete](/cli/azure/group#az_group_delete) 刪除資源群組及其所有資源。 `--no-wait` 參數不會等待作業完成，就會將控制項傳回給提示字元。 `--yes` 參數會確認您想要刪除資源，而不另外對您提示將要進行此作業。

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>後續步驟
在此教學課程中，您已了解如何使用 Azure CLI 來建立及使用擴展集所適用的磁碟：

> [!div class="checklist"]
> * OS 磁碟和暫存磁碟
> * 資料磁碟
> * 標準和進階磁碟
> * 磁碟效能
> * 連結及準備資料磁碟

前往下一個教學課程，了解如何將自訂映像用於擴展集 VM 執行個體。

> [!div class="nextstepaction"]
> [將自訂映像用於擴展集 VM 執行個體](tutorial-use-custom-image-cli.md)