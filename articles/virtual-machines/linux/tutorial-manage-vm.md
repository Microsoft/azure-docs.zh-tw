---
title: 教學課程 - 使用 Azure CLI 來建立和管理 Linux VM | Microsoft Docs
description: 在本教學課程中，您將了解如何使用 Azure CLI 在 Azure 中建立和管理 Linux VM
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/23/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 01e8f74f409271810652a9202b634762ad88dcea
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46981241"
---
# <a name="tutorial-create-and-manage-linux-vms-with-the-azure-cli"></a>教學課程：使用 Azure CLI 來建立和管理 Linux VM

Azure 虛擬機器提供完全可設定且彈性的計算環境。 本教學課程涵蓋基本的「Azure 虛擬機器」部署項目，例如選取 VM 大小、選取 VM 映像、部署 VM。 您會了解如何：

> [!div class="checklist"]
> * 建立及連線到 VM
> * 選取及使用 VM 映像
> * 檢視及使用特定 VM 大小
> * 調整 VM 的大小
> * 檢視及了解 VM 狀態

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.30 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-resource-group"></a>建立資源群組

使用 [az group create](https://docs.microsoft.com/cli/azure/group#az_group_create) 命令來建立資源群組。 

Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 資源群組必須在虛擬機器之前建立。 在此範例中，會在 eastus 區域中建立名為 myResourceGroupVM 的資源群組。 

```azurecli-interactive 
az group create --name myResourceGroupVM --location eastus
```

在建立或修改 VM 時，會指定資源群組，在本教學課程的整個過程中可以看到此操作。

## <a name="create-virtual-machine"></a>建立虛擬機器

使用 [az vm create](https://docs.microsoft.com/cli/azure/vm#az_vm_create) 命令建立虛擬機器。 

建立虛擬機器時，有數個可用的選項，例如作業系統映像、磁碟大小及系統管理認證。 下列範例會建立名為 myVM 的 VM，而該 VM 執行 Ubuntu Server。 VM 上會建立名為 azureuser 的使用者帳戶，而如果預設金鑰位置 (~/.ssh) 沒有 SSH 金鑰，則會加以產生：

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupVM \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

系統可能需要幾分鐘的時間來建立 VM。 建立 VM 之後，Azure CLI 就會輸出 VM 的相關資訊。 請記下 `publicIpAddress`，此位址可用來存取虛擬機器。 

```azurecli-interactive 
{
  "fqdns": "",
  "id": "/subscriptions/d5b9d4b7-6fc1-0000-0000-000000000000/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroupVM"
}
```

## <a name="connect-to-vm"></a>連接到 VM

您現在可以在 Azure Cloud Shell 中使用 SSH 連線到 VM，也可以從本機電腦與 VM 連線。 請使用先前步驟中記下的 `publicIpAddress` 來取代範例 IP 位址。

```bash
ssh azureuser@52.174.34.95
```

在登入 VM 後，您就可以安裝和設定應用程式。 完成時，請像平常一樣地關閉 SSH 工作階段：

```bash
exit
```

## <a name="understand-vm-images"></a>了解 VM 映像

Azure Marketplace 包含許多可用來建立 VM 的映像。 在先前的步驟中，是使用 Ubuntu 映像來建立虛擬機器。 在此步驟中，則是使用 Azure CLI 來搜尋 Marketplace 中的 CentOS 映像，然後使用此映像來部署第二部虛擬機器。 

若要查看最常用的映像清單，請使用 [az vm image list](/cli/azure/vm/image#az_vm_image_list) 命令。

```azurecli-interactive 
az vm image list --output table
```

此命令輸出會傳回 Azure 上最常用的 VM 映像。

```bash
Offer          Publisher               Sku                 Urn                                                             UrnAlias             Version
-------------  ----------------------  ------------------  --------------------------------------------------------------  -------------------  ---------
WindowsServer  MicrosoftWindowsServer  2016-Datacenter     MicrosoftWindowsServer:WindowsServer:2016-Datacenter:latest     Win2016Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2012-R2-Datacenter  MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest  Win2012R2Datacenter  latest
WindowsServer  MicrosoftWindowsServer  2008-R2-SP1         MicrosoftWindowsServer:WindowsServer:2008-R2-SP1:latest         Win2008R2SP1         latest
WindowsServer  MicrosoftWindowsServer  2012-Datacenter     MicrosoftWindowsServer:WindowsServer:2012-Datacenter:latest     Win2012Datacenter    latest
UbuntuServer   Canonical               16.04-LTS           Canonical:UbuntuServer:16.04-LTS:latest                         UbuntuLTS            latest
CentOS         OpenLogic               7.3                 OpenLogic:CentOS:7.3:latest                                     CentOS               latest
openSUSE-Leap  SUSE                    42.2                SUSE:openSUSE-Leap:42.2:latest                                  openSUSE-Leap        latest
RHEL           RedHat                  7.3                 RedHat:RHEL:7.3:latest                                          RHEL                 latest
SLES           SUSE                    12-SP2              SUSE:SLES:12-SP2:latest                                         SLES                 latest
Debian         credativ                8                   credativ:Debian:8:latest                                        Debian               latest
CoreOS         CoreOS                  Stable              CoreOS:CoreOS:Stable:latest                                     CoreOS               latest
```

新增 `--all` 引數即可查看完整的清單。 您也可以依 `--publisher` 或 `–-offer` 來篩選此映像清單。 在此範例中，是以符合 CentOS 的供應項目作為條件來篩選此清單的所有映像。 

```azurecli-interactive 
az vm image list --offer CentOS --all --output table
```

部分輸出：

```azurecli-interactive 
Offer             Publisher         Sku   Urn                                     Version
----------------  ----------------  ----  --------------------------------------  -----------
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.201501         6.5.201501
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.201503         6.5.201503
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.201506         6.5.201506
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.20150904       6.5.20150904
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.20160309       6.5.20160309
CentOS            OpenLogic         6.5   OpenLogic:CentOS:6.5:6.5.20170207       6.5.20170207
```

若要使用特定映像部署 VM，請記下 Urn 資料行中的值，其中包含可用來[識別](cli-ps-findimage.md#terminology)映像的發行者、供應項目、SKU 和版本號碼 (選擇性)。 指定映像時，可以使用 “latest” 來取代映像版本號碼，這會選取最新的散發版本。 在此範例中，是使用 `--image` 引數來指定最新版的 CentOS 6.5 映像。  

```azurecli-interactive 
az vm create --resource-group myResourceGroupVM --name myVM2 --image OpenLogic:CentOS:6.5:latest --generate-ssh-keys
```

## <a name="understand-vm-sizes"></a>了解 VM 大小

虛擬機器大小會決定可供虛擬機器使用的計算資源 (例如 CPU、GPU 和記憶體) 數量。 您必須針對預期的工作負載，來適當設定虛擬機器的大小。 如果工作負載增加，可以調整現有虛擬機器的大小。

### <a name="vm-sizes"></a>VM 大小

下表會將大小分類成各種使用案例。  

| 類型                     | 大小           |    說明       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [一般用途](sizes-general.md)         |Dsv3、Dv3、DSv2、Dv2、DS、D、Av2、A0-7| 平衡的 CPU 對記憶體。 適用於開發/測試及小型到中型應用程式和資料解決方案。  |
| [計算最佳化](sizes-compute.md)   | Fs、F             | CPU 與記憶體的比例高。 適用於中流量應用程式、網路設備，以及批次處理。        |
| [記憶體最佳化](../virtual-machines-windows-sizes-memory.md)    | Esv3、Ev3、M、GS、G、DSv2、DS、Dv2、D   | 記憶體與核心的比例高。 適用於關聯式資料庫、中型到大型快取，以及記憶體內分析。                 |
| [儲存體最佳化](../virtual-machines-windows-sizes-storage.md)      | Ls                | 高磁碟輸送量及 IO。 適用於巨量資料、SQL 及 NoSQL 資料庫。                                                         |
| [GPU](sizes-gpu.md)          | NV、NC            | 以大量圖形轉譯和視訊編輯為目標的特製化 VM。       |
| [高效能](sizes-hpc.md) | H、A8-11          | 我們的最強大 CPU VM，可搭配選用的高輸送量網路介面 (RDMA)。 


### <a name="find-available-vm-sizes"></a>尋找可用的 VM 大小

若要查看特定區域中可用的 VM 大小清單，請使用 [az vm list-sizes](/cli/azure/vm#az_vm_list_sizes) 命令。 

```azurecli-interactive 
az vm list-sizes --location eastus --output table
```

部分輸出：

```azurecli-interactive 
  MaxDataDiskCount    MemoryInMb  Name                      NumberOfCores    OsDiskSizeInMb    ResourceDiskSizeInMb
------------------  ------------  ----------------------  ---------------  ----------------  ----------------------
                 2          3584  Standard_DS1                          1           1047552                    7168
                 4          7168  Standard_DS2                          2           1047552                   14336
                 8         14336  Standard_DS3                          4           1047552                   28672
                16         28672  Standard_DS4                          8           1047552                   57344
                 4         14336  Standard_DS11                         2           1047552                   28672
                 8         28672  Standard_DS12                         4           1047552                   57344
                16         57344  Standard_DS13                         8           1047552                  114688
                32        114688  Standard_DS14                        16           1047552                  229376
                 1           768  Standard_A0                           1           1047552                   20480
                 2          1792  Standard_A1                           1           1047552                   71680
                 4          3584  Standard_A2                           2           1047552                  138240
                 8          7168  Standard_A3                           4           1047552                  291840
                 4         14336  Standard_A5                           2           1047552                  138240
                16         14336  Standard_A4                           8           1047552                  619520
                 8         28672  Standard_A6                           4           1047552                  291840
                16         57344  Standard_A7                           8           1047552                  619520
```

### <a name="create-vm-with-specific-size"></a>建立特定大小的 VM

在先前的 VM 建立範例中，並未提供大小，因此是採用預設大小。 您可以在建立 VM 時，使用 [az vm create](/cli/azure/vm#az_vm_create) 和 `--size` 引數來選取 VM 大小。 

```azurecli-interactive 
az vm create \
    --resource-group myResourceGroupVM \
    --name myVM3 \
    --image UbuntuLTS \
    --size Standard_F4s \
    --generate-ssh-keys
```

### <a name="resize-a-vm"></a>調整 VM 的大小

在部署 VM 之後，可以調整其大小以增加或減少資源配置。 您可以使用 [az vm show](/cli/azure/vm#az_vm_show) 檢視 VM 目前的大小：

```azurecli-interactive
az vm show --resource-group myResourceGroupVM --name myVM --query hardwareProfile.vmSize
```

在調整 VM 大小之前，請先檢查目前的 Azure 叢集上是否有所需的大小可用。 [az vm list-vm-resize-options](/cli/azure/vm#az_vm_list_vm_resize_options) 命令會傳回大小清單。 

```azurecli-interactive 
az vm list-vm-resize-options --resource-group myResourceGroupVM --name myVM --query [].name
```
如果有所需的大小可用，即可從已開機狀態調整 VM 的大小，但是會在作業期間重新開機。 請使用 [az vm resize]( /cli/azure/vm#az_vm_resize) 命令來執行調整大小作業。

```azurecli-interactive 
az vm resize --resource-group myResourceGroupVM --name myVM --size Standard_DS4_v2
```

如果目前的叢集上沒有所需的大小，則必須先將 VM 解除配置，才能進行調整大小作業。 請使用 [az vm deallocate]( /cli/azure/vm#az_vm_deallocate) 命令將 VM 停止並解除配置。 請注意，重新開啟 VM 電源之後，可能會移除暫存磁碟上的任何資料。 公用 IP 位址也會變更，除非使用的是靜態 IP 位址。 

```azurecli-interactive 
az vm deallocate --resource-group myResourceGroupVM --name myVM
```

一旦解除配置，便可以調整大小。 

```azurecli-interactive 
az vm resize --resource-group myResourceGroupVM --name myVM --size Standard_GS1
```

調整大小之後，即可重新啟動 VM。

```azurecli-interactive 
az vm start --resource-group myResourceGroupVM --name myVM
```

## <a name="vm-power-states"></a>VM 電源狀態

Azure VM 的電源狀態可以是許多電源狀態的其中一種。 這個狀態代表從 Hypervisor 的觀點來看，VM 的目前狀態。 

### <a name="power-states"></a>電源狀態

| 電源狀態 | 說明
|----|----|
| 啟動中 | 表示虛擬機器正在啟動。 |
| 執行中 | 表示虛擬機器正在執行。 |
| 停止中 | 表示虛擬機器正在停止。 | 
| 已停止 | 表示虛擬機器已停止。 處於已停止狀態的虛擬機器仍然會產生運算費用。  |
| 正在解除配置 | 表示虛擬機器正在解除配置。 |
| 已解除配置 | 表示虛擬機器已從 Hypervisor 中移除，但仍可在控制平面中使用。 處於「已解除配置」狀態的虛擬機器不會產生計算費用。 |
| - | 表示虛擬機器的電源狀態不明。 |

### <a name="find-power-state"></a>尋找電源狀態

若要擷取特定 VM 的狀態，請使用 [az vm get-instance-view](/cli/azure/vm#az_vm_get_instance_view) 命令。 請務必為虛擬機器和資源群組指定有效的名稱。 

```azurecli-interactive 
az vm get-instance-view \
    --name myVM \
    --resource-group myResourceGroupVM \
    --query instanceView.statuses[1] --output table
```

輸出：

```azurecli-interactive 
ode                DisplayStatus    Level
------------------  ---------------  -------
PowerState/running  VM running       Info
```

## <a name="management-tasks"></a>管理工作

在虛擬機器的生命週期內，您可以執行一些管理工作，例如啟動、停止或刪除虛擬機器。 此外，您可以建立指令碼來自動執行重複或複雜的工作。 使用 Azure CLI，可以從命令列或在指令碼中執行許多常見的管理工作。 

### <a name="get-ip-address"></a>取得 IP 位址

此命令會傳回虛擬機器的私人和公用 IP 位址。  

```azurecli-interactive 
az vm list-ip-addresses --resource-group myResourceGroupVM --name myVM --output table
```

### <a name="stop-virtual-machine"></a>停止虛擬機器

```azurecli-interactive 
az vm stop --resource-group myResourceGroupVM --name myVM
```

### <a name="start-virtual-machine"></a>啟動虛擬機器

```azurecli-interactive 
az vm start --resource-group myResourceGroupVM --name myVM
```

### <a name="delete-resource-group"></a>刪除資源群組

刪除資源群組會同時刪除其內含的所有資源，例如 VM、虛擬網路和磁碟。 `--no-wait` 參數不會等待作業完成，就會將控制項傳回給提示字元。 `--yes` 參數會確認您想要刪除資源，而不另外對您提示將要進行此作業。

```azurecli-interactive 
az group delete --name myResourceGroupVM --no-wait --yes
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解基本的 VM 建立和管理，像是如何：

> [!div class="checklist"]
> * 建立及連線到 VM
> * 選取及使用 VM 映像
> * 檢視及使用特定 VM 大小
> * 調整 VM 的大小
> * 檢視及了解 VM 狀態

請前進到下一個教學課程，以了解 VM 磁碟。  

> [!div class="nextstepaction"]
> [建立和管理 VM 磁碟](./tutorial-manage-disks.md)
