---
title: 使用 cloud-init 為 Azure 上的 Linux 虛擬機器設定主機名稱 | Microsoft Docs
description: 如何透過 Azure CLI 在建立期間使用 cloud-init 來自訂 Linux VM
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
ms.openlocfilehash: e985111a28805f861242240a5c2e3d7b6664be4e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996106"
---
# <a name="use-cloud-init-to-set-hostname-for-a-linux-vm-in-azure"></a>使用 cloud-init 為 Azure 上的 Linux 虛擬機器設定主機名稱
本文會示範如何在 Azure 佈建期間，使用 [cloud-init](https://cloudinit.readthedocs.io) 在虛擬機器 (VM) 上或虛擬機器擴展集 (VMSS) 上設定特定的主機名稱。 一旦 Azure 佈建資源，這些 cloud-init 指令碼就會在初次開機時執行。 如需深入了解 cloud-init 如何以原生方式在 Azure 和支援的 Linux 散發版本中運作，請參閱 [cloud-init 概觀](using-cloud-init.md)

## <a name="set-the-hostname-with-cloud-init"></a>使用 cloud-init 設定主機名稱
依預設，當您在 Azure 中建立新的虛擬機器時，主機名稱會和虛擬機器名稱相同。  若要在您使用 [az vm create](/cli/azure/vm#az_vm_create) 於 Azure 中建立虛擬機器時，執行 cloud-init 指令碼來變更此預設主機名稱，請使用 `--custom-data` 參數來指定 cloud-init 檔案。  

若要查看作用中的升級程序，請在目前的殼層中建立名為 cloud_init_hostname.txt 的檔案，然後貼上下列設定。 針對此案例，在 Cloud Shell 中 (而不是本機電腦上) 建立該檔案。 您可以使用任何您想要的編輯器。 輸入 `sensible-editor cloud_init_hostname.txt` 可建立檔案，並查看可用的編輯器清單。 建議首先選擇使用 **nano** 編輯器。 請確定已正確複製整個 cloud-init 檔案，特別是第一行。  

```yaml
#cloud-config
hostname: myhostname
```

部署此映像前，您必須使用 [az group create](/cli/azure/group#az_group_create) 命令建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

現在，請使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM 並以 `--custom-data cloud_init_hostname.txt` 指定 cloud-init 檔案，如下所示：

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data cloud_init_hostname.txt \
  --generate-ssh-keys 
```

建立之後，Azure CLI 隨即顯示 VM 的相關資訊。 使用 `publicIpAddress` 來以 SSH 連線到您的 VM。 輸入您自己的位址，如下所示︰

```bash
ssh <publicIpAddress>
```

若要查看 VM 名稱，請使用 `hostname` 命令，如下所示：

```bash
hostname
```

VM 所報告的主機名稱應該會和 cloud-init 檔案中所設定的值相同，如下列範例輸出所示：

```bash
myhostname
```

## <a name="next-steps"></a>後續步驟
如需其他設定變更的 cloud-init 範例，請參閱下列文件：
 
- [將其他 Linux 使用者新增至虛擬機器](cloudinit-add-user.md)
- [執行套件管理員以便在初次開機時更新現有的套件](cloudinit-update-vm.md)
- [變更虛擬機器本機的主機名稱](cloudinit-update-vm-hostname.md) 
- [安裝應用程式套件、更新組態檔案，以及插入金鑰](tutorial-automate-vm-deployment.md)