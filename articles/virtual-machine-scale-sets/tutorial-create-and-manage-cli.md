---
title: 教學課程 - 建立及管理 Azure 虛擬機器擴展集 | Microsoft Docs
description: 了解如何使用 Azure CLI 建立虛擬機器擴展集，以及一些常見的管理工作，例如如何啟動和停止執行個體，或變更擴展集容量。
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
ms.openlocfilehash: f00f7934bfb78c16f9e93dc2fcd94025e5f806a1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993273"
---
# <a name="tutorial-create-and-manage-a-virtual-machine-scale-set-with-the-azure-cli"></a>教學課程 - 使用 Azure CLI 建立及管理虛擬機器擴展集
虛擬機器擴展集可讓您部署和管理一組相同、自動調整的虛擬機器。 在虛擬機器擴展集生命週期期間，您可能需要執行一或多個管理工作。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 建立及連線至虛擬機器擴展集
> * 選取及使用 VM 映像
> * 檢視及使用特定 VM 執行個體大小
> * 手動調整擴展集
> * 執行常見的擴展集管理工作

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，則在本教學課程中，您必須執行 Azure CLI 2.0.29 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 


## <a name="create-a-resource-group"></a>建立資源群組
Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 資源群組必須在虛擬機器擴展集之前建立。 使用 [az group create](/cli/azure/group#az_group_create) 命令來建立資源群組。 在此範例中，會在 eastus 區域中建立名為 myResourceGroup 的資源群組。 

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

在本教學課程中，當您建立或修改擴展集時，都會指定資源群組名稱。


## <a name="create-a-scale-set"></a>建立擴展集
您可以使用 [az vmss create](/cli/azure/vmss#az_vmss_create) 命令建立虛擬機器擴展集。 下列範例會建立名為 myScaleSet 的擴展集，以及產生 SSH 金鑰 (如果不存在)︰

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

建立及設定所有擴展集資源和 VM 執行個體需要幾分鐘的時間。 為了將流量散發到個別的虛擬機器執行個體，也會建立負載平衡器。


## <a name="view-the-vm-instances-in-a-scale-set"></a>檢視擴展集中的 VM 執行個體
若要檢視擴展集中的 VM 執行個體清單，請使用 [az vmss list-instances](/cli/azure/vmss#az_vmss_list_instances)，如下所示：

```azurecli-interactive
az vmss list-instances \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --output table
```

下列範例輸出顯示擴展集中的兩個 VM 執行個體：

```bash
  InstanceId  LatestModelApplied    Location    Name          ProvisioningState    ResourceGroup    VmId
------------  --------------------  ----------  ------------  -------------------  ---------------  ------------------------------------
           1  True                  eastus      myScaleSet_1  Succeeded            MYRESOURCEGROUP  c059be0c-37a2-497a-b111-41272641533c
           3  True                  eastus      myScaleSet_3  Succeeded            MYRESOURCEGROUP  ec19e7a7-a4cd-4b24-9670-438f4876c1f9
```


輸出中的第一個資料行會顯示 *InstanceId*。 若要檢視特定 VM 執行個體的其他資訊，請將 `--instance-id` 參數新增至 [az vmss get-instance-view](/cli/azure/vmss#az_vmss_get_instance_view)。 下列範例顯示 VM 執行個體 *1* 的相關資訊：

```azurecli-interactive
az vmss get-instance-view \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --instance-id 1
```


## <a name="list-connection-information"></a>列出連線資訊
一個公用 IP 位址指派給將流量路由至個別 VM 執行個體的負載平衡器。 根據預設，網路位址轉譯 (NAT) 規則會新增至 Azure 負載平衡器，以將遠端連線流量轉送至每個 VM 上指定的連接埠。 若要連線至擴展集中的 VM 執行個體，您可以對指派的公用 IP 位址和連接埠號碼建立遠端連線。

若要列出連線至擴展集中的 VM 執行個體的位址和連接埠，請使用 [az vmss list-instance-connection-info](/cli/azure/vmss#az_vmss_list_instance_connection_info)：

```azurecli-interactive
az vmss list-instance-connection-info \
  --resource-group myResourceGroup \
  --name myScaleSet
```

下列範例輸出顯示執行個體名稱、負載平衡器的公用 IP 位址，以及 NAT 規則將流量轉送到的連接埠號碼：

```bash
{
  "instance 1": "13.92.224.66:50001",
  "instance 3": "13.92.224.66:50003"
}
```

透過 SSH 連線至您的第一個 VM 執行個體。 請使用 `-p` 參數指定您的公用 IP 位址和連接埠號碼，如先前的命令所示：

```azurecli-interactive
ssh azureuser@13.92.224.66 -p 50001
```

登入該 VM 執行個體後，您可以視需要執行一些手動組態變更。 目前，請如常關閉 SSH 工作階段：

```bash
exit
```


## <a name="understand-vm-instance-images"></a>了解 VM 執行個體映像
當您在教學課程一開始建立擴展集時，您為 VM 執行個體指定了 *UbuntuLTS* 的 `--image`。 Azure Marketplace 包含許多可用來建立 VM 執行個體的映像。 若要查看最常用的映像清單，請使用 [az vm image list](/cli/azure/vm/image#az_vm_image_list) 命令。

```azurecli-interactive
az vm image list --output table
```

下列範例輸出顯示 Azure 上最常用的 VM 映像。 *UrnAlias* 可用來指定這些在您建立擴展集時最常用的映像之一。

```bash
Offer          Publisher               Sku                 Urn                                                             UrnAlias             Version
-------------  ----------------------  ------------------  --------------------------------------------------------------  -------------------  ---------
CentOS         OpenLogic               7.3                 OpenLogic:CentOS:7.3:latest                                     CentOS               latest
CoreOS         CoreOS                  Stable              CoreOS:CoreOS:Stable:latest                                     CoreOS               latest
Debian         credativ                8                   credativ:Debian:8:latest                                        Debian               latest
openSUSE-Leap  SUSE                    42.2                SUSE:openSUSE-Leap:42.2:latest                                  openSUSE-Leap        latest
RHEL           RedHat                  7.3                 RedHat:RHEL:7.3:latest                                          RHEL                 latest
SLES           SUSE                    12-SP2              SUSE:SLES:12-SP2:latest                                         SLES                 latest
UbuntuServer   Canonical               16.04-LTS           Canonical:UbuntuServer:16.04-LTS:latest                         UbuntuLTS            latest
WindowsServer  MicrosoftWindowsServer  2016-Datacenter     MicrosoftWindowsServer:WindowsServer:2016-Datacenter:latest     Win2016Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2012-R2-Datacenter  MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest  Win2012R2Datacenter  latest
WindowsServer  MicrosoftWindowsServer  2012-Datacenter     MicrosoftWindowsServer:WindowsServer:2012-Datacenter:latest     Win2012Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2008-R2-SP1         MicrosoftWindowsServer:WindowsServer:2008-R2-SP1:latest         Win2008R2SP1         latest
```

若要檢視完整清單，請加入 `--all` 引數。 您也可以依 `--publisher` 或 `–-offer` 來篩選此映像清單。 在下列範例中，會篩選出供應項目符合 *CentOS* 的所有映像的清單：

```azurecli-interactive
az vm image list --offer CentOS --all --output table
```

下列的扼要的輸出會顯示部分可用的 CentOS 7.3 映像：

```azurecli-interactive 
Offer    Publisher   Sku   Urn                                 Version
-------  ----------  ----  ----------------------------------  -------------
CentOS   OpenLogic   7.3   OpenLogic:CentOS:7.3:7.3.20161221   7.3.20161221
CentOS   OpenLogic   7.3   OpenLogic:CentOS:7.3:7.3.20170421   7.3.20170421
CentOS   OpenLogic   7.3   OpenLogic:CentOS:7.3:7.3.20170517   7.3.20170517
CentOS   OpenLogic   7.3   OpenLogic:CentOS:7.3:7.3.20170612   7.3.20170612
CentOS   OpenLogic   7.3   OpenLogic:CentOS:7.3:7.3.20170707   7.3.20170707
CentOS   OpenLogic   7.3   OpenLogic:CentOS:7.3:7.3.20170925   7.3.20170925
```

若要部署使用特定映像的擴展集，請使用 *Urn* 資料行中的值。 指定映像時，可以使用 *latest* 來取代映像版本號碼，這會選取最新的散發版本。 在下列範例中，會使用 `--image` 引數來指定最新版的 CentOS 7.3 映像。

建立及設定所有擴展集資源和 VM 執行個體需要幾分鐘的時間，而您並不需要部署下列擴展集：

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSetCentOS \
  --image OpenLogic:CentOS:7.3:latest \
  --admin-user azureuser \
  --generate-ssh-keys
```


## <a name="understand-vm-instance-sizes"></a>了解 VM 執行個體大小
VM 執行個體大小 (或 *SKU*) 會決定可供 VM 執行個體使用的計算資源 (例如 CPU、GPU 和記憶體) 數量。 擴展集中的 VM 執行個體必須針對預期的工作負載適當設定大小。

### <a name="vm-instance-sizes"></a>VM 執行個體大小
下表會將一般 VM 大小分類成各種使用案例。

| 類型                     | 一般大小           |    說明       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [一般用途](../virtual-machines/linux/sizes-general.md)         |Dsv3、Dv3、DSv2、Dv2、DS、D、Av2、A0-7| 平衡的 CPU 對記憶體。 適用於開發/測試及小型到中型應用程式和資料解決方案。  |
| [計算最佳化](../virtual-machines/linux/sizes-compute.md)   | Fs、F             | CPU 與記憶體的比例高。 適用於中流量應用程式、網路設備，以及批次處理。        |
| [記憶體最佳化](../virtual-machines/linux/sizes-memory.md)    | Esv3、Ev3、M、GS、G、DSv2、DS、Dv2、D   | 記憶體與核心的比例高。 適用於關聯式資料庫、中型到大型快取，以及記憶體內分析。                 |
| [儲存體最佳化](../virtual-machines/linux/sizes-storage.md)      | Ls                | 高磁碟輸送量及 IO。 適用於巨量資料、SQL 及 NoSQL 資料庫。                                                         |
| [GPU](../virtual-machines/linux/sizes-gpu.md)          | NV、NC            | 以大量圖形轉譯和視訊編輯為目標的特製化 VM。       |
| [高效能](../virtual-machines/linux/sizes-hpc.md) | H、A8-11          | 我們的最強大 CPU VM，可搭配選用的高輸送量網路介面 (RDMA)。 

### <a name="find-available-vm-instance-sizes"></a>尋找可用的 VM 執行個體大小
若要查看特定區域中可用的 VM 執行個體大小清單，請使用 [az vm list-sizes](/cli/azure/vm#az_vm_list_sizes) 命令。

```azurecli-interactive
az vm list-sizes --location eastus --output table
```

其輸出類似於下列扼要的範例，顯示指派給每個 VM 大小的資源：

```azurecli-interactive
  MaxDataDiskCount    MemoryInMb  Name                      NumberOfCores    OsDiskSizeInMb    ResourceDiskSizeInMb
------------------  ------------  ----------------------  ---------------  ----------------  ----------------------
                 4          3584  Standard_DS1_v2                       1           1047552                    7168
                 8          7168  Standard_DS2_v2                       2           1047552                   14336
[...]
                 1           768  Standard_A0                           1           1047552                   20480
                 2          1792  Standard_A1                           1           1047552                   71680
[...]
                 4          2048  Standard_F1                           1           1047552                   16384
                 8          4096  Standard_F2                           2           1047552                   32768
[...]
                24         57344  Standard_NV6                          6           1047552                   38912
                48        114688  Standard_NV12                        12           1047552                  696320
```

### <a name="create-a-scale-set-with-a-specific-vm-instance-size"></a>建立具有特定 VM 執行個體大小的擴展集
當您在教學課程一開始建立擴展集時，您為 VM 執行個體提供了預設的 VM SKU *Standard_D1_v2*。 您可以根據 [az vm list-sizes](/cli/azure/vm#az_vm_list_sizes) 的輸出指定不同的 VM 執行個體大小。 下列範例會使用 `--vm-sku` 參數指定 VM 執行個體大小 *Standard_F1*，以建立擴展集。 建立及設定所有擴展集資源和 VM 執行個體需要幾分鐘的時間，而您並不需要部署下列擴展集：

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSetF1Sku \
  --image UbuntuLTS \
  --vm-sku Standard_F1 \
  --admin-user azureuser \
  --generate-ssh-keys
```


## <a name="change-the-capacity-of-a-scale-set"></a>變更擴展集的容量
當您在教學課程一開始建立擴展集時，您依預設部署了兩個 VM 執行個體。 您可以在使用 [az vmss create](/cli/azure/vmss#az_vmss_create) 時指定 `--instance-count` 參數，以變更要隨擴展集建立的執行個體數目。 若要增加或減少您現有擴展集中的 VM 執行個體數目，您可以手動變更容量。 擴展集會建立或移除所需的 VM 執行個體數目，然後設定用來分散流量的負載平衡器。

若要手動增加或減少擴展集中的 VM 執行個體數目，請使用 [az vmss scale](/cli/azure/vmss#az_vmss_scale)。 下列範例會將擴展集中的 VM 執行個體數目設為 *3*：

```azurecli-interactive
az vmss scale \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --new-capacity 3
```

如果需要幾分鐘的時間才能更新您擴展集的容量。 若要查看擴展集中目前包含的執行個體數目，請使用 [az vmss show](/cli/azure/vmss#az_vmss_show) 並查詢 *sku.capacity*：

```azurecli-interactive
az vmss show \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --query [sku.capacity] \
    --output table
```


## <a name="common-management-tasks"></a>一般管理工作
現在，您可以建立擴展集、列出連線資訊，以及連線至 VM 執行個體。 您已了解如何對 VM 執行個體使用不同的作業系統映像、選取不同的 VM 大小，或手動調整執行個體數目。 在日常管理工作中，您可能需要停止、啟動或重新啟動擴展集中的 VM 執行個體。

### <a name="stop-and-deallocate-vm-instances-in-a-scale-set"></a>停止及解除配置擴展集中的 VM 執行個體
若要停止擴展集中的一或多個 VM 執行個體，請使用 [az vmss stop](/cli/azure/vmss#az_vmss_stop)。 `--instance-ids` 參數可讓您指定一或多個要停止的 VM 執行個體。 如果您未指定執行個體識別碼，則會停止擴展集中的所有 VM 執行個體。 下列範例會停止執行個體 *1*：

```azurecli-interactive
az vmss stop --resource-group myResourceGroup --name myScaleSet --instance-ids 1
```

已停止的 VM 執行個體會維持配置，並繼續產生計算費用。 如果您轉而想要解除配置 VM 執行個體，而且只要產生儲存體費用，請使用 [az vmss deallocate](/cli/azure/vmss#az_vmss_deallocate)。 下列範例會停止並解除配置執行個體 *1*：

```azurecli-interactive
az vmss deallocate --resource-group myResourceGroup --name myScaleSet --instance-ids 1
```

### <a name="start-vm-instances-in-a-scale-set"></a>啟動擴展集中的 VM 執行個體
若要啟動擴展集中的一或多個 VM 執行個體，請使用 [az vmss start](/cli/azure/vmss#az_vmss_start)。 `--instance-ids` 參數可讓您指定一或多個要啟動的 VM 執行個體。 如果您未指定執行個體識別碼，則會啟動擴展集中的所有 VM 執行個體。 下列範例會啟動執行個體 *1*：

```azurecli-interactive
az vmss start --resource-group myResourceGroup --name myScaleSet --instance-ids 1
```

### <a name="restart-vm-instances-in-a-scale-set"></a>重新啟動擴展集中的 VM 執行個體
若要重新啟動擴展集中的一或多個 VM 執行個體，請使用 [az vmss restart](/cli/azure/vmss#az_vm_restart)。 `--instance-ids` 參數可讓您指定一或多個要重新啟動的 VM 執行個體。 如果您未指定執行個體識別碼，則會重新啟動擴展集中的所有 VM 執行個體。 下列範例會重新啟動執行個體 *1*：

```azurecli-interactive
az vmss restart --resource-group myResourceGroup --name myScaleSet --instance-ids 1
```


## <a name="clean-up-resources"></a>清除資源
刪除資源群組時，將會同時刪除其內含的所有資源，例如 VM 執行個體、虛擬網路和磁碟。 `--no-wait` 參數不會等待作業完成，就會將控制項傳回給提示字元。 `--yes` 參數會確認您想要刪除資源，而不另外對您提示將要進行此作業。

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何使用 Azure CLI 執行一些基本的擴展集建立和管理工作：

> [!div class="checklist"]
> * 建立及連線至虛擬機器擴展集
> * 選取及使用 VM 映像
> * 檢視及使用特定 VM 大小
> * 手動調整擴展集
> * 執行常見的擴展集管理工作

請前往下一個教學課程，了解擴展集磁碟。

> [!div class="nextstepaction"]
> [使用資料磁碟搭配擴展集](tutorial-use-disks-cli.md)
