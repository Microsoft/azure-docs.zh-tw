---
title: 教學課程 - 使用 Azure CLI 自動調整擴展集 | Microsoft Docs
description: 了解如何使用 Azure CLI 隨著 CPU 需求的增加和減少自動調整虛擬機器擴展集
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
ms.date: 05/18/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 7c3d536cd4fb99d6d83b973989279d289e8434a8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46995362"
---
# <a name="tutorial-automatically-scale-a-virtual-machine-scale-set-with-the-azure-cli"></a>教學課程：如何使用 Azure CLI 自動調整虛擬機器擴展集

當建立擴展集時，您會定義您想要執行的 VM 執行個體數目。 當您的應用程式需求變更時，您可以自動增加或減少 VM 執行個體數目。 自動調整的能力可讓您在整個應用程式的生命週期中，跟上客戶的需求或對應用程式效能變更做出回應。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 使用擴展集的自動調整
> * 建立及使用自動調整規則
> * 對 VM 執行個體進行壓力測試，並觸發自動調整規則
> * 在需求降低時重新自動相應縮小

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.32 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-scale-set"></a>建立擴展集

使用 [az group create](/cli/azure/group#create) 來建立資源群組，如下所示：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

現在使用 [az vmss create](/cli/azure/vmss#create) 建立虛擬機器擴展集。 下列範例會建立執行個體計數為「2」的擴展集，以及產生 SSH 金鑰 (如果不存在)︰

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --instance-count 2 \
  --admin-username azureuser \
  --generate-ssh-keys
```

## <a name="define-an-autoscale-profile"></a>定義自動調整設定檔

若要對擴展集啟用自動調整，您必須先定義自動調整設定檔。 此設定檔會定義預設、最小和最大擴展集容量。 這些限制可讓您藉由不繼續建立 VM 執行個體來控制成本，並且在可接受的效能與保留在相應縮小事件中的執行個體數目下限之間取得平衡。 使用 [az monitor autoscale create](/cli/azure/monitor/autoscale#az-monitor-autoscale-create) 建立自動調整設定檔。 下列範例會設定 *2* 個 VM 執行個體的預設和最小容量，以及最多 *10* 個：

```azurecli-interactive
az monitor autoscale create \
  --resource-group myResourceGroup \
  --resource myScaleSet \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 2 \
  --max-count 10 \
  --count 2
```

## <a name="create-a-rule-to-autoscale-out"></a>建立自動相應放大的規則

如果您的應用程式需求增加，您擴展集內 VM 執行個體上的負載也會跟著增加。 如果這樣的負載增加會持續而非只是短暫的需求，您就可以設定自動調整規則來增加擴展集中的 VM 執行個體數目。 建立這些 VM 執行個體並部署應用程式後，擴展集就會開始透過負載平衡器將流量分散給它們。 您可以控制要監視哪些計量 (例如 CPU 或磁碟)、應用程式負載必須符合給定的閾值多久，以及要將多少個 VM 執行個體新增至擴展集。

我們將使用 [az monitor autoscale rule create](/cli/azure/monitor/autoscale/rule#az-monitor-autoscale-rule-create) 建立規則，以在平均 CPU 負載在 5 分鐘期間內大於 70% 時，增加擴展集中的 VM 執行個體數目。 當此規則觸發時，VM 執行個體數目會增加三個。

```azurecli-interactive
az monitor autoscale rule create \
  --resource-group myResourceGroup \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 3
```

## <a name="create-a-rule-to-autoscale-in"></a>建立自動相應縮小的規則

在夜晚或週末，您的應用程式需求可能會減少。 若這樣的負載減少會持續一段時間，您就可以設定自動調整規則來減少擴展集中的 VM 執行個體數目。 這個相應縮小的動作可減少執行擴展集的成本，因為只會執行符合目前需求所需的執行個體數目。

使用 [az monitor autoscale rule create](/cli/azure/monitor/autoscale/rule#az-monitor-autoscale-rule-create) 建立規則，以在平均 CPU 負載在 5 分鐘期間內低於 30% 時，減少擴展集中的 VM 執行個體數目。 下列範例定義以 1 為單位相應縮小 VM 執行個體數目的規則：

```azurecli-interactive
az monitor autoscale rule create \
  --resource-group myResourceGroup \
  --autoscale-name autoscale \
  --condition "Percentage CPU < 30 avg 5m" \
  --scale in 1
```

## <a name="generate-cpu-load-on-scale-set"></a>在擴展集上產生 CPU 負載

若要測試自動調整規則，請在擴展集中的 VM 執行個體上產生 CPU 負載。 這種模擬的 CPU 負載會使自動調整相應放大，並增加 VM 執行個體數目。 當模擬的 CPU 負載隨後減輕時，自動調整規則即會相應縮小，並減少 VM 執行個體數目。

首先，請使用 [az vmss list-instance-connection-info](/cli/azure/vmss#az_vmss_list_instance_connection_info)，列出連線至擴展集中的 VM 執行個體的位址和連接埠：

```azurecli-interactive
az vmss list-instance-connection-info \
  --resource-group myResourceGroup \
  --name myScaleSet
```

下列範例輸出顯示執行個體名稱、負載平衡器的公用 IP 位址，以及網路位址轉譯 (NAT) 規則將流量轉送到的連接埠號碼：

```json
{
  "instance 1": "13.92.224.66:50001",
  "instance 3": "13.92.224.66:50003"
}
```

透過 SSH 連線至您的第一個 VM 執行個體。 請使用 `-p` 參數指定您自己的公用 IP 位址和連接埠號碼，如先前的命令所示：

```azurecli-interactive
ssh azureuser@13.92.224.66 -p 50001
```

登入之後，安裝 **stress** 公用程式。 請啟動 10 個會產生 CPU 負載的 **stress** 背景工作。 這些背景工作會執行 420 秒，這就足以讓自動調整規則實作所需的動作。

```azurecli-interactive
sudo apt-get -y install stress
sudo stress --cpu 10 --timeout 420 &
```

當 **stress** 顯示類似於 stress: info: [2688] dispatching hogs: 10 cpu, 0 io, 0 vm, 0 hdd 的輸出時，請按 Enter 鍵返回提示。

若要確認 **stress** 產生了 CPU 負載，請使用 **top** 公用程式檢查作用中的系統負載：

```azuecli-interactive
top
```

結束 **top**，然後關閉 VM 執行個體的連線。 **stress** 會繼續對 VM 執行個體執行。

```azurecli-interactive
Ctrl-c
exit
```

使用先前的 [az vmss list-instance-connection-info](/cli/azure/vmss#az_vmss_list_instance_connection_info) 中所列的連接埠號碼，連線至第二個 VM 執行個體：

```azurecli-interactive
ssh azureuser@13.92.224.66 -p 50003
```

安裝並執行 **stress**，然後在第二個 VM 執行個體上啟動十個背景工作。

```azurecli-interactive
sudo apt-get -y install stress
sudo stress --cpu 10 --timeout 420 &
```

同樣地，當 **stress** 顯示類似於 stress: info: [2713] dispatching hogs: 10 cpu, 0 io, 0 vm, 0 hdd 的輸出時，請按 Enter 鍵返回提示。

關閉第二個 VM 執行個體的連線。 **stress** 會繼續對 VM 執行個體執行。

```azurecli-interactive
exit
```

## <a name="monitor-the-active-autoscale-rules"></a>監視作用中的自動調整規則

若要監視擴展集中的 VM 執行個體數目，請使用 **watch**。 約需要 5 分鐘的時間，自動調整規則才會開始執行相應放大程序，以回應 **stress** 對每個 VM 執行個體產生的 CPU 負載：

```azurecli-interactive
watch az vmss list-instances \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --output table
```

達到 CPU 閾值時，自動調整規則就會增加擴展集中的 VM 執行個體數目。 下列輸出顯示擴展集自動相應放大時所建立的三個 VM：

```bash
Every 2.0s: az vmss list-instances --resource-group myResourceGroup --name myScaleSet --output table

  InstanceId  LatestModelApplied    Location    Name          ProvisioningState    ResourceGroup    VmId
------------  --------------------  ----------  ------------  -------------------  ---------------  ------------------------------------
           1  True                  eastus      myScaleSet_1  Succeeded            myResourceGroup  4f92f350-2b68-464f-8a01-e5e590557955
           2  True                  eastus      myScaleSet_2  Succeeded            myResourceGroup  d734cd3d-fb38-4302-817c-cfe35655d48e
           4  True                  eastus      myScaleSet_4  Creating             myResourceGroup  061b4c90-0d73-49fc-a066-19eab0b3d95c
           5  True                  eastus      myScaleSet_5  Creating             myResourceGroup  4beff8b9-4e65-40cb-9652-43899309da27
           6  True                  eastus      myScaleSet_6  Creating             myResourceGroup  9e4133dd-2c57-490e-ae45-90513ce3b336
```

初始 VM 執行個體的 **stress** 停止時，平均 CPU 負載即會恢復正常。 再經過 5 分鐘之後，自動調整規則會相應縮小 VM 執行個體數目。 相應縮小動作會先移除具有最高識別碼的 VM 執行個體。 當擴展集使用可用性設定組或可用性區域時，相應縮小動作會平均散發到這些 VM 執行個體上。 下列範例輸出顯示擴展集自動相應縮小時所刪除的一個 VM 執行個體：

```bash
           6  True                  eastus      myScaleSet_6  Deleting             myResourceGroup  9e4133dd-2c57-490e-ae45-90513ce3b336
```

使用 `Ctrl-c` 結束 watch。 擴展集會繼續每隔 5 分鐘相應縮小一次，並移除一個 VM 執行個體，直到達到最小執行個體計數 (兩個) 為止。

## <a name="clean-up-resources"></a>清除資源

若要移除您的擴展集與其他資源，請使用 [az group delete](/cli/azure/group#az_group_delete) 刪除資源群組及其所有資源。 `--no-wait` 參數不會等待作業完成，就會將控制項傳回給提示字元。 `--yes` 參數會確認您想要刪除資源，而不另外對您提示將要進行此作業。

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解如何使用 Azure CLI 自動相應縮小或相應放大擴展集：

> [!div class="checklist"]
> * 使用擴展集的自動調整
> * 建立及使用自動調整規則
> * 對 VM 執行個體進行壓力測試，並觸發自動調整規則
> * 在需求降低時重新自動相應縮小

如需有效虛擬機器擴展集的更多範例，請參閱下列 Azure CLI 範例指令碼：

> [!div class="nextstepaction"]
> [Azure CLI 的擴展集指令碼範例](cli-samples.md)
