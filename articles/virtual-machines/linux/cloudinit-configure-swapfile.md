---
title: 使用 cloud-init 在 Linux 虛擬機器上設定分頁檔 | Microsoft Docs
description: 如何透過 Azure CLI，在建立期間使用 cloud-init 在 Linux 虛擬機器中設定分頁檔
services: virtual-machines-linux
documentationcenter: ''
author: rickstercdn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: article
ms.date: 11/29/2017
ms.author: rclaus
ms.openlocfilehash: 2a5a878b7c8c3b6126d90b978241fbcb237d8db7
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946301"
---
# <a name="use-cloud-init-to-configure-a-swapfile-on-a-linux-vm"></a>使用 cloud-init 在 Linux 虛擬機器上設定分頁檔
本文會示範如何使用 [cloud-init](https://cloudinit.readthedocs.io) 在各種 Linux 發行套件上設定分頁檔。 傳統上是由 Linux 代理程式 (WALA) 根據發行套件的需求來設定分頁檔。  本文將概述在佈建期間，使用 cloud-init 依需求建置分頁檔的流程。  如需深入了解 cloud-init 如何以原生方式在 Azure 和支援的 Linux 散發版本中運作，請參閱 [cloud-init 概觀](using-cloud-init.md)

## <a name="create-swapfile-for-ubuntu-based-images"></a>建立 Ubuntu 型映像的分頁檔
依預設，Azure 上的 Ubuntu 資源庫映像不會建立分頁檔。 若要使用 cloud-init 在虛擬機器佈建期間啟用分頁檔設定 - 請參閱 Ubuntu wiki 上的 [AzureSwapPartitions 文件](https://wiki.ubuntu.com/AzureSwapPartitions)。

## <a name="create-swapfile-for-red-hat-and-centos-based-images"></a>建立 Red Hat 和 CentOS 型映像的分頁檔

在您目前的殼層中，建立名為 cloud_init_swapfile.txt的檔案，並貼上下列設定。 針對此案例，在 Cloud Shell 中 (而不是本機電腦上) 建立該檔案。 您可以使用任何您想要的編輯器。 輸入 `sensible-editor cloud_init_swapfile.txt` 可建立檔案，並查看可用的編輯器清單。 建議首先選擇使用 **nano** 編輯器。 請確定已正確複製整個 cloud-init 檔案，特別是第一行。  

```yaml
#cloud-config
disk_setup:
  ephemeral0:
    table_type: gpt
    layout: [66, [33,82]]
    overwrite: true
fs_setup:
  - device: ephemeral0.1
    filesystem: ext4
  - device: ephemeral0.2
    filesystem: swap
mounts:
  - ["ephemeral0.1", "/mnt"]
  - ["ephemeral0.2", "none", "swap", "sw", "0", "0"]
```

部署此映像前，您必須使用 [az group create](/cli/azure/group#az_group_create) 命令建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

現在，請使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM 並以 `--custom-data cloud_init_swapfile.txt` 指定 cloud-init 檔案，如下所示：

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data cloud_init_swapfile.txt \
  --generate-ssh-keys 
```

## <a name="verify-swapfile-was-created"></a>確認已建立分頁檔
以 SSH 連線到顯示於由上述命令所產生之輸出中的 VM 公用 IP 位址。 輸入您自己的 **publicIpAddress**，如下所示︰

```bash
ssh <publicIpAddress>
```

一旦您使用 SSH 連線至虛擬機器，就必須檢查是否已建立分頁檔

```bash
swapon -s
```

此命令的輸出如下所示：

```text
Filename                Type        Size    Used    Priority
/dev/sdb2  partition   2494440 0   -1
```

> [!NOTE] 
> 如果您現有的 Azure 映像已設定分頁檔，但您想要變更新映像的分頁檔設定，那麼您應該要移除現有的分頁檔。 如需詳細資訊，請參閱＜透過 cloud-init 自訂映像來進行佈建＞的文件。

## <a name="next-steps"></a>後續步驟
如需其他設定變更的 cloud-init 範例，請參閱下列文件：
 
- [將其他 Linux 使用者新增至虛擬機器](cloudinit-add-user.md)
- [執行套件管理員以便在初次開機時更新現有的套件](cloudinit-update-vm.md)
- [變更虛擬機器本機的主機名稱](cloudinit-update-vm-hostname.md) 
- [安裝應用程式套件、更新組態檔案，以及插入金鑰](tutorial-automate-vm-deployment.md)
