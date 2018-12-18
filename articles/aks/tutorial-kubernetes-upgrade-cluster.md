---
title: Azure 上的 Kubernetes 教學課程 - 升級叢集
description: 在本 Azure Kubernetes Service (AKS) 教學課程中，您將了解如何將現有的 AKS 叢集升級至最新的可用 Kubernetes 版本。
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: tutorial
ms.date: 08/14/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: 1c0710be11b95b66d16661b5aff9cbf739ccda92
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48901923"
---
# <a name="tutorial-upgrade-kubernetes-in-azure-kubernetes-service-aks"></a>教學課程：在 Azure Kubernetes Service (AKS) 中升級 Kubernetes

依循應用程式和叢集的生命週期，您可能會想要升級至 Kubernetes 的最新可用版本，並使用新功能。 使用 Azure CLI 可以升級 Azure Kubernetes Service (AKS) 叢集。 若要將執行應用程式的中斷情況降到最低，則會在升級過程中仔細地[隔離並清空][kubernetes-drain] Kubernetes 節點。

在本教學課程 (七個章節的第七部分) 中升級了 Kubernetes 叢集。 您會了解如何：

> [!div class="checklist"]
> * 識別目前和可用的 Kubernetes 版本
> * 升級 Kubernetes 節點
> * 驗證升級成功

## <a name="before-you-begin"></a>開始之前

在先前的教學課程中，已將應用程式封裝成容器映像、將此映像上傳至 Azure Container Registry，並已建立 Kubernetes 叢集。 該應用程式接著便在 Kubernetes 叢集上執行。 如果您尚未完成這些步驟，而想要跟著做，請回到[教學課程 1 – 建立容器映像][aks-tutorial-prepare-app]。

在進行本教學課程時，您必須執行 Azure CLI 2.0.44 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][azure-cli-install]。

## <a name="get-available-cluster-versions"></a>取得可用的叢集版本

在升級叢集之前，請使用 [az aks get-upgrades][] 命令檢查哪些 Kubernetes 版本可進行升級：

```azurecli
az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster --output table
```

在下列範例中，目前的版本是 1.9.6，可用版本則顯示於 [升級] 資料行下方。

```
Name     ResourceGroup    MasterVersion    NodePoolVersion    Upgrades
-------  ---------------  ---------------  -----------------  ----------------------
default  myResourceGroup  1.9.9            1.9.9              1.10.3, 1.10.5, 1.10.6
```

## <a name="upgrade-a-cluster"></a>升級叢集

使用 [az aks upgrade][] 命令升級 AKS 叢集。 下列範例會將叢集升級至 Kubernetes *1.10.6* 版。

> [!NOTE]
> 您一次只能升級一個次要版本。 例如，您可以從 *1.9.6* 升級至 *1.10.3*，但無法直接從 *1.9.6* 升級至 *1.11.x*。 若要從 *1.9.6* 升級至 *1.11.x*，必須先從 *1.9.6* 升級至 *1.10.3*，然後再執行從 *1.10.3* 到 *1.11.x* 的升級。

```azurecli
az aks upgrade --resource-group myResourceGroup --name myAKSCluster --kubernetes-version 1.10.6
```

下列扼要的範例輸出顯示 *kubernetesVersion* 現在回報 1.10.6：

```json
{
  "agentPoolProfiles": [
    {
      "count": 3,
      "maxPods": 110,
      "name": "nodepool1",
      "osType": "Linux",
      "storageProfile": "ManagedDisks",
      "vmSize": "Standard_DS1_v2",
    }
  ],
  "dnsPrefix": "myAKSClust-myResourceGroup-19da35",
  "enableRbac": false,
  "fqdn": "myaksclust-myresourcegroup-19da35-bd54a4be.hcp.eastus.azmk8s.io",
  "id": "/subscriptions/<Subscription ID>/resourcegroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster",
  "kubernetesVersion": "1.10.6",
  "location": "eastus",
  "name": "myAKSCluster",
  "type": "Microsoft.ContainerService/ManagedClusters"
}
```

## <a name="validate-an-upgrade"></a>驗證升級

使用 [az aks show][] 命令確認升級是否成功，如下所示：

```azurecli
az aks show --resource-group myResourceGroup --name myAKSCluster --output table
```

下列範例輸出顯示 AKS 叢集執行的是 *KubernetesVersion 1.10.6*：

```
Name          Location    ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
------------  ----------  ---------------  -------------------  -------------------  ----------------------------------------------------------------
myAKSCluster  eastus      myResourceGroup  1.10.6               Succeeded            myaksclust-myresourcegroup-19da35-bd54a4be.hcp.eastus.azmk8s.io
```

## <a name="delete-the-cluster"></a>刪除叢集

由於這是教學課程系列的最後一個部分，您可以刪除 AKS 叢集。 Kubernetes 節點會在 Azure 虛擬機器 (VM) 上執行，所以即使您不使用叢集，這些節點仍會繼續產生費用。 請使用 [az group delete][az-group-delete] 命令來移除資源群組、容器服務以及所有相關資源。

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

> [!NOTE]
> 當您刪除叢集時，不會移除 AKS 叢集所使用的 Azure Active Directory 服務主體。 如需有關如何移除服務主體的步驟，請參閱[AKS 服務主體的考量和刪除][sp-delete]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已在 AKS 叢集中升級 Kubernetes。 您已了解如何︰

> [!div class="checklist"]
> * 識別目前和可用的 Kubernetes 版本
> * 升級 Kubernetes 節點
> * 驗證升級成功

跟著此連結以深入了解 AKS。

> [!div class="nextstepaction"]
> [AKS 概觀][aks-intro]

<!-- LINKS - external -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

<!-- LINKS - internal -->
[aks-intro]: ./intro-kubernetes.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[az aks show]: /cli/azure/aks#az-aks-show
[az aks get-upgrades]: /cli/azure/aks#az-aks-get-upgrades
[az aks upgrade]: /cli/azure/aks#az-aks-upgrade
[azure-cli-install]: /cli/azure/install-azure-cli
[az-group-delete]: /cli/azure/group#az-group-delete
[sp-delete]: kubernetes-service-principal.md#additional-considerations
