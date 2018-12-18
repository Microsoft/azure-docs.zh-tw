---
title: 使用 cloud-init 在 Azure 上的 Linux VM 中更新和安裝封裝 | Microsoft Docs
description: 如何透過 Azure CLI 在建立期間使用 cloud-init 在 Linux 虛擬機器中更新及安裝套件
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
ms.date: 04/20/2018
ms.author: rclaus
ms.openlocfilehash: 84fab18d4e1f385f8770db52b18ac85151f48afd
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46988048"
---
# <a name="use-cloud-init-to-update-and-install-packages-in-a-linux-vm-in-azure"></a>使用 cloud-init 在 Azure 上的 Linux VM 中更新和安裝封裝
本文會示範如何在 Azure 佈建期間，使用 [cloud-init](https://cloudinit.readthedocs.io) 在 Linux 虛擬機器 (VM) 上或虛擬機器擴展集 (VMSS) 上更新封裝。 一旦 Azure 佈建資源，這些 cloud-init 指令碼就會在初次開機時執行。 如需深入了解 cloud-init 如何以原生方式在 Azure 和支援的 Linux 散發版本中運作，請參閱 [cloud-init 概觀](using-cloud-init.md)

## <a name="update-a-vm-with-cloud-init"></a>使用 cloud-init 更新 VM
基於安全考量，您應該設定 VM 以在第一次開機時套用最新的更新。 由於 cloud-init 可在不同的 Linux 散發版本上運作，所以不需要為套件管理員指定 `apt` 或 `yum`。 相反地，您可定義 `package_upgrade` 並讓 cloud-init 程序判斷使用中散發版本的適當機制。 此工作流程可讓您在不同的散發版本上使用相同的 cloud-init 指令碼。

若要查看作用中的升級程序，請在目前的殼層中建立名為 cloud_init_upgrade.txt 的檔案，然後貼上下列設定。 針對此案例，在 Cloud Shell 中 (而不是本機電腦上) 建立該檔案。 您可以使用任何您想要的編輯器。 輸入 `sensible-editor cloud_init_upgrade.txt` 可建立檔案，並查看可用的編輯器清單。 建議首先選擇使用 **nano** 編輯器。 請確定已正確複製整個 cloud-init 檔案，特別是第一行。  

```yaml
#cloud-config
package_upgrade: true
packages:
- httpd
```

部署此映像前，您必須使用 [az group create](/cli/azure/group#az_group_create) 命令建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

現在，請使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM 並以 `--custom-data cloud_init_upgrade.txt` 指定 cloud-init 檔案，如下所示：

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data cloud_init_upgrade.txt \
  --generate-ssh-keys 
```

以 SSH 連線到顯示於由上述命令所產生之輸出中的 VM 公用 IP 位址。 輸入您自己的 **publicIpAddress**，如下所示︰

```bash
ssh <publicIpAddress>
```

執行套件管理工具並檢查是否有更新。

```bash
sudo yum update
```

由於 cloud-init 會在開機時檢查是否有更新並予以安裝，因此應該已經沒有要套用的其他更新。  執行 `yum history` 並檢閱類似如下的輸出，您就會看到更新程序、更改過的套件數目，以及 `httpd` 的安裝。

```bash
Loaded plugins: fastestmirror, langpacks
ID     | Command line             | Date and time    | Action(s)      | Altered
-------------------------------------------------------------------------------
     3 | -t -y install httpd      | 2018-04-20 22:42 | Install        |    5
     2 | -t -y upgrade            | 2018-04-20 22:38 | I, U           |   65
     1 |                          | 2017-12-12 20:32 | Install        |  522
```

## <a name="next-steps"></a>後續步驟
如需其他設定變更的 cloud-init 範例，請參閱下列文件：
 
- [將其他 Linux 使用者新增至虛擬機器](cloudinit-add-user.md)
- [執行套件管理員以便在初次開機時更新現有的套件](cloudinit-update-vm.md)
- [變更虛擬機器本機的主機名稱](cloudinit-update-vm-hostname.md) 
- [安裝應用程式套件、更新組態檔案，以及插入金鑰](tutorial-automate-vm-deployment.md)