---
title: Azure Container Service 教學課程 - 調整應用程式
description: Azure Container Service 教學課程 - 調整應用程式
services: container-service
author: dlepow
manager: jeconnoc
ms.service: container-service
ms.topic: tutorial
ms.date: 09/14/2017
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: 792d9b1409b9571474f47da4940724df7a764d82
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39429877"
---
# <a name="scale-kubernetes-pods-and-kubernetes-infrastructure"></a>調整 Kubernetes Pod 和 Kubernetes 基礎結構

[!INCLUDE [aks-preview-redirect.md](../../../includes/aks-preview-redirect.md)]

如果您一直都依照教學課程操作，就會在 Azure Container Service 中有一個正常運作 Kubernetes 叢集，並已部署「Azure 投票」應用程式。 

在本教學課程中 (七個章節的第五部分)，您會將應用程式中的 Pod 相應放大，然後嘗試進行 Pod 自動調整。 您也會了解如何調整 Azure VM 代理程式節點的數目，以變更叢集的工作負載裝載容量。 完成的工作包括：

> [!div class="checklist"]
> * 手動調整 Kubernetes Pod
> * 設定執行應用程式前端的自動調整 Pod
> * 調整 Kubernetes Azure 代理程式節點

在後續的教學課程中，會更新 Azure Vote 應用程式，且會將 Log Analytics 設定為監視 Kubernetes 叢集。

## <a name="before-you-begin"></a>開始之前

在先前的教學課程中，已將應用程式封裝成容器映像、將此映像上傳至 Azure Container Registry，並已建立 Kubernetes 叢集。 該應用程式接著便在 Kubernetes 叢集上執行。 

如果您尚未完成這些步驟，而想要跟著做，請回到[教學課程 1 – 建立容器映像](./container-service-tutorial-kubernetes-prepare-app.md)。 

## <a name="manually-scale-pods"></a>手動調整 Pod

目前為止，已部署 Azure Vote 前端與 Redis 執行個體，每個都有單一複本。 若要確認，請執行 [kubectl get](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) 命令。

```azurecli-interactive
kubectl get pods
```

輸出：

```bash
NAME                               READY     STATUS    RESTARTS   AGE
azure-vote-back-2549686872-4d2r5   1/1       Running   0          31m
azure-vote-front-848767080-tf34m   1/1       Running   0          31m
```

使用 [kubectl scale](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#scale) 命令來手動變更 `azure-vote-front` 部署中的 Pod 數目。 以下範例會將數目增加到 5 個。

```azurecli-interactive
kubectl scale --replicas=5 deployment/azure-vote-front
```

執行 [kubectl get pods](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) 以確認 Kubernetes 是否正在建立 Pod。 在大約一分鐘後，其他 Pod 便已在執行：

```azurecli-interactive
kubectl get pods
```

輸出：

```bash
NAME                                READY     STATUS    RESTARTS   AGE
azure-vote-back-2606967446-nmpcf    1/1       Running   0          15m
azure-vote-front-3309479140-2hfh0   1/1       Running   0          3m
azure-vote-front-3309479140-bzt05   1/1       Running   0          3m
azure-vote-front-3309479140-fvcvm   1/1       Running   0          3m
azure-vote-front-3309479140-hrbf2   1/1       Running   0          15m
azure-vote-front-3309479140-qphz8   1/1       Running   0          3m
```

## <a name="autoscale-pods"></a>自動調整 Pod

Kubernetes 支援[水平 Pod 自動調整](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)，可根據 CPU 使用率或其他選取的計量來調整部署中的 Pod 數目。 

若要使用自動調整程式，必須為您的 Pod 定義 CPU 要求和限制。 在 `azure-vote-front` 部署中，前端容器會要求 0.25 個 CPU，限制為 0.5 個 CPU。 設定看起來會像這樣：

```YAML
resources:
  requests:
     cpu: 250m
  limits:
     cpu: 500m
```

下列範例使用 [kubectl autoscale](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#autoscale) 命令來自動調整 `azure-vote-front` 部署中的 Pod 數目。 在這裡，如果 CPU 使用率超過 50%，自動調整程式就會增加 Pod，最多可達 10 個。


```azurecli-interactive
kubectl autoscale deployment azure-vote-front --cpu-percent=50 --min=3 --max=10
```

若要查看自動調整程式的狀態，請執行下列命令：

```azurecli-interactive
kubectl get hpa
```

輸出：

```bash
NAME               REFERENCE                     TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
azure-vote-front   Deployment/azure-vote-front   0% / 50%   3         10        3          2m
```

幾分鐘之後，如果 Azure Vote 應用程式上的負載降到最低，Pod 複本數目就會自動降低成 3 個。

## <a name="scale-the-agents"></a>調整代理程式

如果您在上一個教學課程中使用預設命令來建立 Kubernetes 叢集，它就會有 3 個代理程式節點。 如果您打算在叢集上增加或減少容器工作負載，可以手動調整代理程式的數目。 請使用 [az acs scale](/cli/azure/acs#az-acs-scale) 命令，並使用 `--new-agent-count` 參數來指定代理程式的數目。

下列範例會在名為 *myK8sCluster* 的 Kubernetes 叢集中，將代理程式節點的數目增加到 4 個。 此命令需要幾分鐘的時間來完成。

```azurecli-interactive
az acs scale --resource-group=myResourceGroup --name=myK8SCluster --new-agent-count 4
```

此命令輸出會在 `agentPoolProfiles:count` 值中顯示代理程式的數目：

```azurecli
{
  "agentPoolProfiles": [
    {
      "count": 4,
      "dnsPrefix": "myK8SCluster-myK8SCluster-e44f25-k8s-agents",
      "fqdn": "",
      "name": "agentpools",
      "vmSize": "Standard_D2_v2"
    }
  ],
...

```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已在 Kubernetes 叢集中使用不同的調整功能。 涵蓋的工作包括：

> [!div class="checklist"]
> * 手動調整 Kubernetes Pod
> * 設定執行應用程式前端的自動調整 Pod
> * 調整 Kubernetes Azure 代理程式節點

請前往下一個教學課程，以了解如何更新 Kubernetes 中的應用程式。

> [!div class="nextstepaction"]
> [更新 Kubernetes 中的應用程式](./container-service-tutorial-kubernetes-app-update.md)

