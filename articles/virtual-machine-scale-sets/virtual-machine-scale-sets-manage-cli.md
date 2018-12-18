---
title: 使用 Azure CLI 管理虛擬機器擴展集 | Microsoft Docs
description: 用於管理虛擬機器擴展集 (例如如何啟動和停止執行個體，或變更擴展集容量) 的一般 Azure CLI 命令。
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
ms.topic: article
ms.date: 05/29/2018
ms.author: cynthn
ms.openlocfilehash: 014e6ff2bc769675e47743cfdd4fbf7fc6234974
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46999829"
---
# <a name="manage-a-virtual-machine-scale-set-with-the-azure-cli"></a>使用 Azure CLI 管理虛擬機器擴展集
在虛擬機器擴展集生命週期期間，您可能需要執行一或多個管理工作。 此外，您可以建立指令碼來自動化各種生命週期工作。 此文章詳述一些可讓您執行這些工作的一般 Azure CLI 命令。

若要完成這些管理工作，您需要最新的 Azure CLI。 如需相關資訊，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 如果您需要建立虛擬機器擴展集，可以[使用 Azure CLI 建立擴展集](quick-create-cli.md)。


## <a name="view-information-about-a-scale-set"></a>檢視擴展集的相關資訊
若要檢視擴展集的整體資訊，請使用 [az vmss show](/cli/azure/vmss#az_vmss_show)。 下列範例取得 myResourceGroup 資源群組中 myScaleSet 擴展集的相關資訊。 輸入您自己的名稱，如下所示：

```azurecli
az vmss show --resource-group myResourceGroup --name myScaleSet
```


## <a name="view-vms-in-a-scale-set"></a>檢視擴展集中的 VM
若要檢視擴展集中的 VM 執行個體清單，請使用 [az vmss list-instances](/cli/azure/vmss#list-instances)。 下列範例列出 myResourceGroup 資源群組中，名稱為 myScaleSet 擴展集內的所有虛擬機器執行個體。 針對這些名稱提供您自己的值：

```azurecli
az vmss list-instances \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --output table
```

若要檢視特定 VM 執行個體的其他資訊，請將 `--instance-id` 參數新增至 [az vmss get-instance-view](/cli/azure/vmss#get-instance-view)，並指定要檢視的執行個體。 下列範例檢視 myScaleSet 擴展集和 myResourceGroup 資源群組中 VM 執行個體 *0* 的相關資訊。 輸入您自己的名稱，如下所示：

```azurecli
az vmss get-instance-view \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --instance-id 0
```


## <a name="list-connection-information-for-vms"></a>列出 VM 的連線資訊
若要連線至擴展集中的 VM，您可以 SSH 或 RDP 到指派的公用 IP 位址和連接埠號碼。 網路位址轉譯 (NAT) 規則預設會新增至 Azure 負載平衡器，以將遠端連線流量轉送給每部虛擬機器。 若要列出連線至擴展集中 VM 執行個體的位址和連接埠，請使用 [az vmss list-instance-connection-info](/cli/azure/vmss#list-instance-connection-info)。 下列範例列出 myScaleSet 擴展集和 myResourceGroup 資源群組中虛擬機器執行個體的連線資訊。 針對這些名稱提供您自己的值：

```azurecli
az vmss list-instance-connection-info \
    --resource-group myResourceGroup \
    --name myScaleSet
```


## <a name="change-the-capacity-of-a-scale-set"></a>變更擴展集的容量
上述命令顯示您擴展集和 VM 執行個體的相關資訊。 若要增加或減少擴展集中的執行個體數目，您可以變更容量。 擴展集會建立或移除所需的 VM 數目，然後設定接收應用程式流量的 VM。

若要查看擴展集中目前擁有的執行個體數目，請使用 [az vmss show](/cli/azure/vmss#az_vmss_show)並查詢 sku.capacity：

```azurecli
az vmss show \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --query [sku.capacity] \
    --output table
```

然後，您可以使用 [az vmss scale](/cli/azure/vmss#az_vmss_scale)，手動增加或減少擴展集中的虛擬機器數目。 下列範例會將擴展集中的 VM 數目設定為 5：

```azurecli
az vmss scale \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --new-capacity 5
```

如果需要幾分鐘的時間才能更新您擴展集的容量。 如果您減少擴展集的容量，則會先移除具有最高執行個體識別碼的 VM。


## <a name="stop-and-start-vms-in-a-scale-set"></a>停止和啟動擴展集中的 VM
若要停止擴展集中的一或多個 VM，請使用 [az vmss stop](/cli/azure/vmss#az-vmss-stop)。 `--instance-ids` 參數可讓您指定停止一或多個 VM。 如果您未指定執行個體識別碼，則會停止擴展集中的所有 VM。 若要停止多個 VM，請以空格分隔每個執行個體識別碼。

下列範例停止 myScaleSet 擴展集和 myResourceGroup 資源群組中的執行個體 *0*。 提供您自己的值，如下所示︰

```azurecli
az vmss stop --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```

已停止的 VM 會維持配置，並繼續產生計算費用。 如果您改為想要解除配置 VM，而且只要產生儲存體費用，請使用 [az vmss deallocate](/cli/azure/vmss#az_vmss_deallocate)。 若要解除配置多個 VM，請以空格分隔每個執行個體識別碼。 下列範例停止和解除配置 myScaleSet 擴展集和 myResourceGroup 資源群組中的執行個體 *0*。 提供您自己的值，如下所示︰

```azurecli
az vmss deallocate --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```


### <a name="start-vms-in-a-scale-set"></a>啟動擴展集中的 VM
若要啟動擴展集中的一或多個 VM，請使用 [az vmss start](/cli/azure/vmss#az_vmss_start)。 `--instance-ids` 參數可讓您指定啟動一或多個 VM。 如果您未指定執行個體識別碼，則會啟動擴展集中的所有 VM。 若要啟動多個 VM，請以空格分隔每個執行個體識別碼。

下列範例啟動 myScaleSet 擴展集和 myResourceGroup 資源群組中的執行個體 *0*。 提供您自己的值，如下所示︰

```azurecli
az vmss start --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```


## <a name="restart-vms-in-a-scale-set"></a>重新啟動擴展集中的 VM
若要重新啟動擴展集中的一或多個 VM，請使用 [az vmss restart](/cli/azure/vmss#az_vmss_restart)。 `--instance-ids` 參數可讓您指定重新啟動一或多個 VM。 如果您未指定執行個體識別碼，則會重新啟動擴展集中的所有 VM。 若要重新啟動多個 VM，請以空格分隔每個執行個體識別碼。

下列範例重新啟動 myScaleSet 擴展集和 myResourceGroup 資源群組中的執行個體 *0*。 提供您自己的值，如下所示︰

```azurecli
az vmss restart --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```


## <a name="remove-vms-from-a-scale-set"></a>移除擴展集中的 VM
若要移除擴展集中的一或多個 VM，請使用 [az vmss delete-instances](/cli/azure/vmss#delete-instances)。 `--instance-ids` 參數可讓您指定移除一或多個 VM。 如果您指定 * 表示執行個體識別碼，則會移除擴展集中的所有 VM。 若要移除多個 VM，請以空格分隔每個執行個體識別碼。

下列範例移除 myScaleSet 擴展集和 myResourceGroup 資源群組中的執行個體 *0*。 提供您自己的值，如下所示︰

```azurecli
az vmss delete-instances --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```


## <a name="next-steps"></a>後續步驟
擴展集的其他一般工作包含如何[部署應用程式](virtual-machine-scale-sets-deploy-app.md)以及[升級 VM 執行個體](virtual-machine-scale-sets-upgrade-scale-set.md)。 您也可以使用 Azure CLI [設定自動調整規則](virtual-machine-scale-sets-autoscale-overview.md)。
