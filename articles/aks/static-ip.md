---
title: 搭配 Azure Kubernetes Service (AKS) 負載平衡器使用靜態 IP 位址
description: 了解如何搭配 Azure Kubernetes Service (AKS) 負載平衡器來建立和使用靜態 IP 位址。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 09/26/2018
ms.author: iainfou
ms.openlocfilehash: 8aab091ed992a946cd78ecf4f0c8fdfff4185a08
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47407547"
---
# <a name="use-a-static-public-ip-address-with-the-azure-kubernetes-service-aks-load-balancer"></a>搭配 Azure Kubernetes Service (AKS) 負載平衡器使用靜態公用 IP 位址

根據預設，指派給 AKS 叢集所建立之負載平衡器資源的公用 IP 位址，只有在該資源的生命週期內有效。 如果您刪除 Kubernetes 服務，相關聯的負載平衡器和 IP 位址也會一併刪除。 如果您想要針對已重新部署的 Kubernetes 服務指派特定的 IP 位址或保留 IP 位址，您可以建立並使用靜態公用 IP 位址。

此文章示範如何建立靜態公用 IP 位址，並將它指派給您的 Kubernetes 服務。

## <a name="before-you-begin"></a>開始之前

此文章假設您目前具有 AKS 叢集。 如果您需要 AKS 叢集，請參閱[使用 Azure CLI][aks-quickstart-cli] 或[使用 Azure 入口網站][aks-quickstart-portal]的 AKS 快速入門。

您也必須安裝並設定 Azure CLI 版本 2.0.46 或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][install-azure-cli]。

## <a name="create-a-static-ip-address"></a>建立靜態 IP 位址

當您建立靜態公用 IP 位址來與 AKS 搭配使用時，必須在**節點**資源群組中建立 IP 位址資源。 使用 [az aks show][az-aks-show] 命令來取得資源群組名稱，並新增 `--query nodeResourceGroup` 查詢參數。 下列範例會在 *myResourceGroup* 資源群組名稱中，取得 AKS 叢集名稱 *myAKSCluster* 的節點資源群組：

```azurecli
$ az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv

MC_myResourceGroup_myAKSCluster_eastus
```

現在使用 [az network public ip create][az-network-public-ip-create] 命令來建立靜態公用 IP 位址。 指定在上一個命令中所取得的節點資源群組名稱，然後指定 IP 位址資源的名稱，例如 *myAKSPublicIP*：

```azurecli
az network public-ip create \
    --resource-group MC_myResourceGroup_myAKSCluster_eastus \
    --name myAKSPublicIP \
    --allocation-method static
```

IP 位址即會顯示，如下列扼要範例輸出所示：

```json
{
  "publicIp": {
    "dnsSettings": null,
    "etag": "W/\"6b6fb15c-5281-4f64-b332-8f68f46e1358\"",
    "id": "/subscriptions/<SubscriptionID>/resourceGroups/MC_myResourceGroup_myAKSCluster_eastus/providers/Microsoft.Network/publicIPAddresses/myAKSPublicIP",
    "idleTimeoutInMinutes": 4,
    "ipAddress": "40.121.183.52",
    [..]
  }
````

您稍後可以使用 [az network public-ip list][az-network-public-ip-list] 命令來取得公用 IP 位址。 指定節點資源群組的名稱，然後查詢 *ipAddress*，如下列範例所示：

```azurecli
$ az network public-ip list --resource-group MC_myResourceGroup_myAKSCluster_eastus --query [0].ipAddress --output tsv

40.121.183.52
```

## <a name="create-a-service-using-the-static-ip-address"></a>使用靜態 IP 位址建立服務

若要使用靜態公用 IP 位址建立服務，請將 `loadBalancerIP` 屬性和和靜態公用 IP 位址的值新增到 YAML 資訊清單。 建立名為 `load-balancer-service.yaml` 的檔案，然後將下列 YAML 複製進來。 提供您在上一個步驟中所建立的自有公用 IP 位址。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: azure-load-balancer
spec:
  loadBalancerIP: 40.121.183.52
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-load-balancer
```

使用 `kubectl apply` 命令建立服務和部署。

```console
kubectl apply -f load-balancer-service.yaml
```

## <a name="troubleshoot"></a>疑難排解

如果 Kubernetes 服務資訊清單 *loadBalancerIP* 屬性中所定義的靜態 IP 位址不存在，或尚未在節點資源群組中建立，則負載平衡器服務建立即會失敗。 若要進行疑難排解，請使用 [kubectl describe][kubectl-describe] 命令來檢閱服務建立事件。 提供 YAML 資訊清單中所指定的服務名稱，如下列範例所示：

```console
kubectl describe service azure-load-balancer
```

Kubernetes 服務資源的相關資訊即會顯示。 下列範例輸出結尾的 *Events* 指出「找不到使用者提供的 IP 位址」。 在這些案例中，請確認您已在節點資源群組中建立靜態公用 IP 位址，以及 Kubernetes 服務資訊清單中所指定的 IP 位址正確無誤。

```
Name:                     azure-load-balancer
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=azure-load-balancer
Type:                     LoadBalancer
IP:                       10.0.18.125
IP:                       40.121.183.52
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32582/TCP
Endpoints:                <none>
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type     Reason                      Age               From                Message
  ----     ------                      ----              ----                -------
  Normal   CreatingLoadBalancer        7s (x2 over 22s)  service-controller  Creating load balancer
  Warning  CreatingLoadBalancerFailed  6s (x2 over 12s)  service-controller  Error creating load balancer (will retry): Failed to create load balancer for service default/azure-load-balancer: user supplied IP Address 40.121.183.52 was not found
```

## <a name="next-steps"></a>後續步驟

若要進一步控制應用程式的網路流量，您可能想要改為[建立輸入控制器][aks-ingress-basic]。 您也可以[使用靜態公用 IP 位址建立輸入控制器][aks-static-ingress]。

<!-- LINKS - External -->
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe

<!-- LINKS - Internal -->
[aks-faq-resource-group]: faq.md#why-are-two-resource-groups-created-with-aks
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[az-network-public-ip-list]: /cli/azure/network/public-ip#az-network-public-ip-list
[az-aks-show]: /cli/azure/aks#az-aks-show
[aks-ingress-basic]: ingress-basic.md
[aks-static-ingress]: ingress-static-ip.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
