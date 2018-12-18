---
title: 教學課程 - Azure 中 Linux VM 的高可用性 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用 Azure CLI 在可用性設定組中部署高可用性的虛擬機器
documentationcenter: ''
services: virtual-machines-linux
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: tutorial
ms.date: 08/24/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: ee1c3cb079d77fe8e510695514e77c8ae9795c30
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46950500"
---
# <a name="tutorial-create-and-deploy-highly-available-virtual-machines-with-the-azure-cli"></a>教學課程：使用 Azure CLI 建立及部署高可用性的虛擬機器

在本教學課程中，您會學到如何使用稱為「可用性設定組」的功能，增加 Azure 虛擬機器解決方案的可用性和可靠性。 可用性設定組可確保您在 Azure 上部署的 VM 會分散到多個各自獨立的硬體叢集中。 這麼做可以確保當 Azure 發生硬體或軟體故障時，受到影響的只會是一部分的 VM 子集，您整體的解決方案則會維持可用且正常運作。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立可用性設定組
> * 在可用性設定組中建立 VM
> * 檢查可用的 VM 大小

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.30 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="availability-set-overview"></a>可用性設定組概觀

可用性設定組是一種可在 Azure 中使用的邏輯群組功能，用以確保其中所放置的 VM 資源在部署到 Azure 資料中心時會彼此隔離。 Azure 可確保您在可用性設定組中所放置的 VM，會橫跨多部實體伺服器、計算機架、儲存體單位和網路交換器來執行。 如果硬體或 Azure 軟體發生故障時，只有一部分的 VM 子集會受到影響，整體的應用程式則會保持運作狀態，可供客戶繼續使用。 如果您想要建置可靠的雲端解決方案，就一定要使用可用性設定組功能。

請想想典型的 VM 架構解決方案，在此解決方案中，您可能有 4 個前端 Web 伺服器，並使用 2 個後端 VM 來裝載資料庫。 在使用 Azure 時，建議您先定義兩個可用性設定組，再部署 VM︰一個可用性設定組用來放置「Web」層，另一個可用性設定組用來放置「資料庫」層。 當您建立新的 VM 時，您便可以將可用性設定組指定為 az vm create 命令的參數，Azure 會自動確保您在可用性設定組內所建立的 VM，會跨多個實體硬體資源來隔離。 如果其中一個用來執行 Web 伺服器或資料庫伺服器 VM 的實體硬體發生問題，Web 伺服器和資料庫 VM 的其他執行個體會繼續執行，因為它們位於不同硬體上。

如果您想要在 Azure 內部署可靠的 VM 架構解決方案，請使用可用性設定組。


## <a name="create-an-availability-set"></a>建立可用性設定組

您可以使用 [az vm availability-set create](/cli/azure/vm/availability-set#az_vm_availability_set_create) 來建立可用性設定組。 在此範例中，我們會針對 myResourceGroupAvailability 資源群組中名為 myAvailabilitySet 的可用性設定組，將更新和容錯網域數目設定為 2。

首先，使用 [az group create](/cli/azure/group#az-group-create) 建立資源群組，然後建立可用性設定組：

```azurecli-interactive
az group create --name myResourceGroupAvailability --location eastus

az vm availability-set create \
    --resource-group myResourceGroupAvailability \
    --name myAvailabilitySet \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 2
```

可用性設定組可讓您跨容錯網域和更新網域來隔離資源。 **容錯網域**代表以隔離方式集合在一起的伺服器、網路和儲存體資源。 在前面的範例中，在部署 VM 時，可用性設定組會分散到至少兩個容錯網域。 可用性設定組也會分散到兩個**更新網域**。 兩個更新網域可確保當 Azure 執行軟體更新時，VM 資源是隔離的，以免在 VM 上執行的所有軟體同時更新。


## <a name="create-vms-inside-an-availability-set"></a>建立位於可用性設定組內的 VM

您必須將 VM 建立於可用性設定組內，才能確保 VM 會在硬體中正確地分散。 建立可用性設定組之後，就無法在其中新增現有的 VM。

透過 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM 時，使用 `--availability-set` 參數來指定可用性設定組的名稱。

```azurecli-interactive
for i in `seq 1 2`; do
   az vm create \
     --resource-group myResourceGroupAvailability \
     --name myVM$i \
     --availability-set myAvailabilitySet \
     --size Standard_DS1_v2  \
     --vnet-name myVnet \
     --subnet mySubnet \
     --image UbuntuLTS \
     --admin-username azureuser \
     --generate-ssh-keys
done
```

可用性設定組內有兩部虛擬機器。 由於它們位於相同的可用性設定組內，Azure 會確保 VM 和其所有資源 (包括資料磁碟) 會分散到隔離開來的實體硬體中。 這樣的分佈方式有助於確保 VM 解決方案整體有更高的可用性。

移至 [資源群組] > [myResourceGroupAvailability] > [myAvailabilitySet]，即可在入口網站中檢視可用性設定組分佈方式。 VM 會分散到兩個容錯和更新網域，如下列範例所示：

![入口網站中的可用性設定組](./media/tutorial-availability-sets/fd-ud.png)

## <a name="check-for-available-vm-sizes"></a>檢查可用的 VM 大小

稍後可以將其他 VM 新增至可用性設定組，其中的 VM 大小可用於硬體上。 針對可用性設定組，使用 [az vm availability-set list-sizes](/cli/azure/vm/availability-set#az-vm-availability-set-list-sizes) 列出硬體叢集上所有可用的大小。

```azurecli-interactive
az vm availability-set list-sizes \
     --resource-group myResourceGroupAvailability \
     --name myAvailabilitySet \
     --output table
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立可用性設定組
> * 在可用性設定組中建立 VM
> * 檢查可用的 VM 大小

請前進到下一個教學課程，以了解虛擬機器擴展集。

> [!div class="nextstepaction"]
> [建立虛擬機器擴展集](tutorial-create-vmss.md)
