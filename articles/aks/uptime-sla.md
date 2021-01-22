---
title: Azure Kubernetes Service (AKS) 與運作時間 SLA
description: 了解 Azure Kubernetes Service (AKS) API 伺服器的選用運作時間 SLA 供應項目。
services: container-service
ms.topic: conceptual
ms.date: 01/08/2021
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 9f8f697da7499d370c96b77e7e543dec9fbafa3e
ms.sourcegitcommit: 52e3d220565c4059176742fcacc17e857c9cdd02
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98664090"
---
# <a name="azure-kubernetes-service-aks-uptime-sla"></a>Azure Kubernetes Service (AKS) 運作時間 SLA

運作時間 SLA 是一項選用功能，可為叢集啟用受財務支援的高層級 SLA。 運作時間 SLA 保證使用[可用性區域][availability-zones]的叢集能保有 Kubernetes API 伺服器端點 99.95% 的可用性，而未使用可用性區域的叢集則保有 99.9% 的可用性。 AKS 會在更新和容錯網域之間使用主要節點複本，以確保符合 SLA 需求。

需要 SLA 以符合合規性需求或需要將 SLA 延伸至其終端使用者的客戶，應該啟用這項功能。 客戶若有重要工作負載可因較高層級的運作時間 SLA 獲益，也可能受益於這項功能。 將運作時間 SLA 功能與可用性區域搭配使用，可讓 Kubernetes API 伺服器的運作時間具有更高的可用性。  

客戶仍可建立具有 99.5% 服務等級目標 (SLO) 的無限制免費叢集，並視需要選擇慣用的 SLO 或 SLA 運作時間。

> [!Important]
> 針對具有輸出鎖定的叢集，請參閱[限制輸出流量](limit-egress-traffic.md)以開啟適當的連接埠。

## <a name="region-availability"></a>區域可用性

* 執行時間 SLA 適用于公用區域，以及 [支援 AKS](https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service)的 Azure Government 區域。
* 在支援 AKS 的所有公用區域中， [私人 AKS][private-clusters] 叢集都可以使用執行時間 SLA。

## <a name="sla-terms-and-conditions"></a>SLA 條款及條件

運作時間 SLA 是付費功能，可對個別叢集啟用。 運作時間 SLA 定價取決於離散叢集的數目，而不是個別叢集的大小。 如需詳細資訊，請檢視[運作時間 SLA 定價詳細資料](https://azure.microsoft.com/pricing/details/kubernetes-service/)。

## <a name="before-you-begin"></a>開始之前

* 安裝 [Azure CLI](/cli/azure/install-azure-cli) 2.8.0 版或更新版本

## <a name="creating-a-new-cluster-with-uptime-sla"></a>使用執行時間 SLA 建立新的叢集

> [!NOTE]
> 目前，如果您啟用執行時間 SLA，則無法從叢集移除。

若要透過運作時間 SLA 建立新叢集，請使用 Azure CLI。

下列範例會在 eastus  位置建立名為 myResourceGroup  的資源群組：

```azurecli-interactive
# Create a resource group
az group create --name myResourceGroup --location eastus
```
使用 [`az aks create`][az-aks-create] 命令來建立 AKS 叢集。 下列範例會建立名為 myAKSCluster  並包含一個節點的叢集。 這項作業需要幾分鐘的時間才能完成：

```azurecli-interactive
# Create an AKS cluster with uptime SLA
az aks create --resource-group myResourceGroup --name myAKSCluster --uptime-sla --node-count 1
```
在幾分鐘之後，此命令就會完成，並以 JSON 格式傳回叢集的相關資訊。 下列 JSON 程式碼片段顯示 SKU 的付費層，指出您的叢集已啟用執行時間 SLA：

```output
  },
  "sku": {
    "name": "Basic",
    "tier": "Paid"
  },
```

## <a name="modify-an-existing-cluster-to-use-uptime-sla"></a>修改現有的叢集以使用執行時間 SLA

您可以選擇更新現有的叢集，以使用執行時間 SLA。

如果您已使用先前的步驟建立 AKS 叢集，請刪除資源群組：

```azurecli-interactive
# Delete the existing cluster by deleting the resource group 
az group delete --name myResourceGroup --yes --no-wait
```

建立新的資源群組：

```azurecli-interactive
# Create a resource group
az group create --name myResourceGroup --location eastus
```

建立新的叢集，而不使用執行時間 SLA：

```azurecli-interactive
# Create a new cluster without uptime SLA
az aks create --resource-group myResourceGroup --name myAKSCluster--node-count 1
```

使用 [`az aks update`][az-aks-update] 命令來更新現有的叢集：

```azurecli-interactive
# Update an existing cluster to use Uptime SLA
 az aks update --resource-group myResourceGroup --name myAKSCluster --uptime-sla
 ```

 下列 JSON 程式碼片段顯示 SKU 的付費層，指出您的叢集已啟用執行時間 SLA：

 ```output
  },
  "sku": {
    "name": "Basic",
    "tier": "Paid"
  },
  ```

## <a name="clean-up"></a>清除

若要避免產生費用，請清除您所建立的任何資源。 若要刪除叢集，請使用 [`az group delete`][az-group-delete] 命令來刪除 AKS 資源群組：

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```


## <a name="next-steps"></a>後續步驟

使用[可用性區域][availability-zones]提高 AKS 叢集工作負載的高可用性。

將您的叢集設定為[限制輸出流量](limit-egress-traffic.md)。

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
[region-availability]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service

<!-- LINKS - Internal -->
[vm-skus]: ../virtual-machines/sizes.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[faq]: ./faq.md
[availability-zones]: ./availability-zones.md
[az-aks-create]: /cli/azure/aks?#az-aks-create
[limit-egress-traffic]: ./limit-egress-traffic.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-update]: /cli/azure/aks?view=azure-cli-latest&preserve-view=true#az_aks_update
[az-group-delete]: /cli/azure/group#az-group-delete
[private-clusters]: private-clusters.md
