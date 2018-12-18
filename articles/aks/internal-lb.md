---
title: 建立 Azure Kubernetes Service (AKS) 內部負載平衡器
description: 了解如何建立和使用內部負載平衡器，透過 Azure Kubernetes Service (AKS) 公開您的服務。
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 07/12/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: 123fc08995416e0ff9c7e12a526deadc34b3a4a2
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39001390"
---
# <a name="use-an-internal-load-balancer-with-azure-kubernetes-service-aks"></a>搭配 Azure Kubernetes Service (AKS) 使用內部負載平衡器

內部負載平衡讓 Kubernetes 服務可供在與 Kubernetes 叢集相同之虛擬網路中執行的應用程式存取。 本文說明如何透過 Azure Kubernetes Service (AKS) 來建立和使用內部負載平衡器。 Azure Load Balancer 有兩種 SKU：基本和標準。 AKS 使用基本 SKU。

## <a name="create-an-internal-load-balancer"></a>建立內部負載平衡器

若要建立內部負載平衡器，使用服務類型 LoadBalancer 和 azure-load-balancer-internal 註釋來建立服務資訊清單，如下列範例所示：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-vote-front
```

一旦使用 `kubetctl apply` 部署之後，Azure 負載平衡器即會建立，並且可在與 AKS 叢集相同的虛擬網路上使用。

![AKS 內部負載平衡器的影像](media/internal-lb/internal-lb.png)

當您檢視服務詳細資料時，*EXTERNAL-IP* 資料行中的 IP 位址是內部負載平衡器的 IP 位址，如以下範例所示：

```
$ kubectl get service azure-vote-front

NAME               TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.248.59   10.240.0.7    80:30555/TCP   10s
```

## <a name="specify-an-ip-address"></a>指定 IP 位址

如果您想要搭配內部負載平衡器使用特定 IP 位址，請將 loadBalancerIP 屬性新增至負載平衡器規格中。指定的 IP 位址必須位於與 AKS 叢集相同的子網路，而且必須尚未指派給資源。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  loadBalancerIP: 10.240.0.25
  ports:
  - port: 80
  selector:
    app: azure-vote-front
```

當您檢視服務詳細資料時，*EXTERNAL-IP* 上的 IP 位址會反映指定的 IP 位址：

```
$ kubectl get service azure-vote-front

NAME               TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.184.168   10.240.0.25   80:30225/TCP   4m
```

## <a name="use-private-networks"></a>使用私人網路

當您建立 AKS 叢集時，您可以指定進階網路設定。 這種方法可讓您將叢集部署到現有的 Azure 虛擬網路和子網路。 其中一個案例是將 AKS 叢集部署到已連線至內部部署環境的私人網路，並執行只能在內部存取的服務。 如需詳細資訊，請參閱 [AKS 中的進階網路設定][advanced-networking]。

不需變更先前步驟，即可在使用私人網路的 AKS 叢集中部署內部負載平衡器。 負載平衡器已建立於與您的 AKS 叢集相同的資源群組中，但會連線到您的私人虛擬網路和子網路，如下列範例所示：

```
$ kubectl get service azure-vote-front

NAME               TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.1.15.188   10.0.0.35     80:31669/TCP   1m
```

> [!NOTE]
> 您可能需要對 AKS 叢集的服務主體，授與 Azure 虛擬網路資源部署所在資源群組的「網路參與者」角色。 使用 [az aks show][az-aks-show] 檢視服務主體，例如 `az aks show --resource-group myResourceGroup --name myAKSCluster --query "servicePrincipalProfile.clientId"`。 若要建立角色指派，請使用 [az role assignment create][az-role-assignment-create] 命令。

## <a name="specify-a-different-subnet"></a>指定不同的子網路

若要指定負載平衡器的子網路，請將 *azure-load-balancer-internal-subnet* 註釋新增到您的服務。 指定的子網路必須位於與您的 AKS 叢集相同的虛擬網路中。 部署後，負載平衡器 *EXTERNAL-IP* 位址會是所指定子網路的一部分。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
    service.beta.kubernetes.io/azure-load-balancer-internal-subnet: "apps-subnet"
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-vote-front
```

## <a name="delete-the-load-balancer"></a>刪除負載平衡器

刪除所有使用內部負載平衡器的服務後，負載平衡器本身也會遭到刪除。

## <a name="next-steps"></a>後續步驟

在 [Kubernetes 服務文件][kubernetes-services]中深入了解 Kubernetes 服務。

<!-- LINKS - External -->
[kubernetes-services]: https://kubernetes.io/docs/concepts/services-networking/service/

<!-- LINKS - Internal -->
[advanced-networking]: networking-overview.md
[deploy-advanced-networking]: networking-overview.md#configure-networking---cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create