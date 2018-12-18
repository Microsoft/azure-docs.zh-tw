---
title: 診斷虛擬機器網路路由問題 - Azure CLI | Microsoft Docs
description: 在本文章中，您將了解如何使用 Azure 網路監看員的下一個躍點功能，來診斷虛擬機器網路路由問題。
services: network-watcher
documentationcenter: network-watcher
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose virtual machine (VM) network routing problem that prevents communication to different destinations.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 04/20/2018
ms.author: jdial
ms.custom: ''
ms.openlocfilehash: 8b881e51bfac25b83a828ad1f44fcd6d7da1c791
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948040"
---
# <a name="diagnose-a-virtual-machine-network-routing-problem---azure-cli"></a>診斷虛擬機器網路路由問題 - Azure CLI

在本文章中，您可部署虛擬機器 (VM)，然後檢查送至 IP 位址和 URL 的通訊。 您可判斷通訊失敗的原因及其解決方式。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0.28 版或更新版本。 若要尋找已安裝的版本，請執行 `az --version`。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 驗證 CLI 版本之後，請執行 `az login` 以建立與 Azure 的連線。 本文章中的 CLI 命令會經過格式化以在 Bash 殼層中執行。

## <a name="create-a-vm"></a>建立 VM

您必須先建立資源群組來包含 VM，才能建立 VM。 使用 [az group create](/cli/azure/group#az-group-create) 來建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

使用 [az vm create](/cli/azure/vm#az-vm-create) 建立 VM。 如果預設金鑰位置中還沒有 SSH 金鑰，此命令將會建立這些金鑰。 若要使用一組特定金鑰，請使用 `--ssh-key-value` 選項。 下列範例會建立名為 myVm 的 VM：

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm \
  --image UbuntuLTS \
  --generate-ssh-keys
```

建立 VM 需要幾分鐘的時間。 在建立 VM 且 CLI 傳回輸出之前，請勿繼續進行其餘步驟。

## <a name="test-network-communication"></a>測試網路通訊

若要測試網路監看員的網路通訊，您必須先在欲測試的 VM 所在區域啟用網路監看員，然後使用網路監看員的下一個躍點功能來測試通訊。

### <a name="enable-network-watcher"></a>啟用網路監看員

如果您已在美國東部地區啟用網路監看員，請跳至[使用下一個躍點](#use-next-hop)。 使用 [az network watcher configure](/cli/azure/network/watcher#az-network-watcher-configure) 命令在美國東部地區建立網路監看員：

```azurecli-interactive
az network watcher configure \
  --resource-group NetworkWatcherRG \
  --locations eastus \
  --enabled
```

### <a name="use-next-hop"></a>使用下一個躍點

Azure 會自動建立通往預設目的地的路由。 您可以建立覆寫預設路由的自訂路由。 有時候，自訂路由可能會導致通訊失敗。 當流量的目的地為特定位址時，若要測試來自 VM 的路由，請使用 [az network watcher show-next-hop](/cli/azure/network/watcher?view=azure-cli-latest#az-network-watcher-show-next-hop) 判斷下一個路由躍點。

測試從 VM 輸出至 www.bing.com 的其中一個 IP 位址通訊：

```azurecli-interactive
az network watcher show-next-hop \
  --dest-ip 13.107.21.200 \
  --resource-group myResourceGroup \
  --source-ip 10.0.0.4 \
  --vm myVm \
  --nic myVmVMNic \
  --out table
```

幾秒之後，輸出會通知您 [nextHopType] 是 [網際網路]，而 [routeTableId] 是 [系統路由]。 此結果可讓您知道通往目的地的路由是有效的。

測試從 VM 輸出至 172.31.0.100 的通訊：

```azurecli-interactive
az network watcher show-next-hop \
  --dest-ip 172.31.0.100 \
  --resource-group myResourceGroup \
  --source-ip 10.0.0.4 \
  --vm myVm \
  --nic myVmVMNic \
  --out table
```

傳回的輸出會通知您 [nextHopType] 是 [無]，而 [routeTableId] 也是 [系統路由]。 此結果可讓您知道，雖然通往目的地的系統路由是有效的，但沒有下一個躍點可將流量路由至目的地。

## <a name="view-details-of-a-route"></a>檢視路由的詳細資料

若要分析進一步路由，請使用 [az network nic show-effective-route-table](/cli/azure/network/nic#az-network-nic-show-effective-route-table) 命令檢閱網路介面的有效路由：

```azurecli-interactive
az network nic show-effective-route-table \
  --resource-group myResourceGroup \
  --name myVmVMNic
```

傳回的輸出中包含下列文字：

```azurecli
{
  "additionalProperties": {
    "disableBgpRoutePropagation": false
  },
  "addressPrefix": [
    "0.0.0.0/0"
  ],
  "name": null,
  "nextHopIpAddress": [],
  "nextHopType": "Internet",
  "source": "Default",
  "state": "Active"
},
```

當您使用 `az network watcher show-next-hop` 命令測試[使用下一個躍點](#use-next-hop)中輸出至 13.107.21.200 的通訊時，**addressPrefix** 為 0.0.0.0/0** 的路由會用來將流量路由至該位址，這是因為輸出中沒有其他路由包含此位址。 根據預設，如果位址並未指定在另一個路由的位址前置詞中，則會路由至網際網路。

但是當您使用 `az network watcher show-next-hop` 命令測試輸出至 172.31.0.100 的通訊時，結果收到通知指出您已沒有下一個躍點類型。 您也會在傳回的輸出中看到下列文字：

```azurecli
{
  "additionalProperties": {
    "disableBgpRoutePropagation": false
      },
  "addressPrefix": [
    "172.16.0.0/12"
  ],
  "name": null,
  "nextHopIpAddress": [],
  "nextHopType": "None",
  "source": "Default",
  "state": "Active"
},
```

如同來自 `az network watcher nic show-effective-route-table` 命令的輸出中所示，雖然有通往 172.16.0.0/12 前置詞的預設路由，且其中包含 172.31.0.100 位址，但 [nextHopType] 是 [無]。 Azure 會建立通往 172.16.0.0/12 的預設路由，但不會指定下一個躍點類型，除非有理由這麼做。 例如，如果您將 172.16.0.0/12 位址範圍新增至虛擬網路的位址空間，則 Azure 會將此路由的 **nextHopType** 變更為 **Virtual network**。 執行檢查就會看到 **Virtual network** 顯示為 **nextHopType**。

## <a name="clean-up-resources"></a>清除資源

您可以使用 [az group delete](/cli/azure/group#az-group-delete) 來移除不再需要的資源群組，以及其所包含的所有資源：

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>後續步驟

在本文章中，您已建立 VM 並診斷來自 VM 的網路路由。 您已了解 Azure 會建立數個預設路由，並測試通往兩個不同目的地的路由。 深入了解 [Azure 中的路由](../virtual-network/virtual-networks-udr-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)以及如何[建立自訂路由](../virtual-network/manage-route-table.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-route)。

針對輸出 VM 連線，您也可以使用網路監看員的[連線疑難排解](network-watcher-connectivity-cli.md)功能，以判斷 VM 與端點之間的延遲，以及所允許和拒絕的網路流量。 您可以使用網路監看員的連線監視功能，以監視某段時間內 VM 與端點 (例如 IP 位址或 URL) 之間的通訊情形。 若要了解方法，請參閱[監視網路連線](connection-monitor.md)。