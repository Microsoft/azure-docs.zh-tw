---
title: Azure Kubernetes Service (AKS) 中適用於輸出流量的靜態 IP 位址
description: 了解如何在 Azure Kubernetes Service (AKS) 叢集中，建立和使用適用於輸出流量的靜態公用 IP 位址
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 09/26/2018
ms.author: iainfou
ms.openlocfilehash: 175fa625a94626cde4d782abd1e9629530cab8b4
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47408499"
---
# <a name="use-a-static-public-ip-address-for-egress-traffic-in-azure-kubernetes-service-aks"></a>在 Azure Kubernetes Service (AKS) 中使用適用於輸出流量的靜態公用 IP 位址

根據預設，來自 Azure Kubernetes Service (AKS) 叢集的輸出 IP 位址會隨機指派。 例如，當您需要識別 IP 位址以存取外部服務時，這個設定並不理想。 您可能需要改為指派靜態 IP 位址，讓您可將其列入白名單以用於服務存取。

此文章示範如何在 AKS 叢集中，建立和使用靜態公用 IP 位址來與輸出流量搭配使用。

## <a name="before-you-begin"></a>開始之前

此文章假設您目前具有 AKS 叢集。 如果您需要 AKS 叢集，請參閱[使用 Azure CLI][aks-quickstart-cli] 或[使用 Azure 入口網站][aks-quickstart-portal]的 AKS 快速入門。

您也必須安裝並設定 Azure CLI 版本 2.0.46 或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][install-azure-cli]。

## <a name="egress-traffic-overview"></a>輸出流量概觀

來自 AKS 叢集的輸出流量會遵循 [Azure Load Balancer 慣例][outbound-connections]。 在建立 `LoadBalancer` 類型的第一個 Kubernetes 服務之前，AKS 叢集中的代理程式節點不屬於任何 Azure Load Balancer 集區。 在此設定中，節點不含任何執行個體層級的公用 IP 位址。 Azure 會將輸出流量轉譯為公用來源 IP 位址，這不是可設定或確定性的位址。

一旦建立 `LoadBalancer` 類型的 Kubernetes 服務，代理程式節點就會新增至 Azure Load Balancer 集區。 Azure 會將輸出流量轉譯為負載平衡器上設定的第一個公用 IP 位址。 此公用 IP 位址只適用於該資源的生命週期。 如果您刪除 Kubernetes LoadBalancer 服務，相關聯的負載平衡器和 IP 位址也會一併刪除。 如果您想要針對已重新部署的 Kubernetes 服務指派特定的 IP 位址或保留 IP 位址，您可以建立並使用靜態公用 IP 位址。

## <a name="create-a-static-public-ip"></a>建立靜態公用 IP

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

## <a name="create-a-service-with-the-static-ip"></a>建立具有靜態 IP 的服務

若要使用靜態公用 IP 位址建立服務，請將 `loadBalancerIP` 屬性和和靜態公用 IP 位址的值新增到 YAML 資訊清單。 建立名為 `egress-service.yaml` 的檔案，然後將下列 YAML 複製進來。 提供您在上一個步驟中所建立的自有公用 IP 位址。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: azure-egress
spec:
  loadBalancerIP: 40.121.183.52
  type: LoadBalancer
  ports:
  - port: 80
```

使用 `kubectl apply` 命令建立服務和部署。

```console
kubectl apply -f egress-service.yaml
```

此服務會在 Azure Load Balancer 上設定新的前端 IP。 如果您未設定任何其他 IP，則**所有**輸出流量現在都應該會使用此位址。 在 Azure Load Balancer 上設定多個位址時，輸出會使用該負載平衡器上的第一個 IP。

## <a name="verify-egress-address"></a>確認輸出位址

若要確認正在使用靜態公用 IP 位址，您可以使用 DNS 查閱服務，例如 `checkip.dyndns.org`。

啟動並附加至基本 *Debian* Pod：

```console
kubectl run -it --rm aks-ip --image=debian
```

若要存取容器內的網站，請使用 `apt-get` 來將 `curl` 安裝到容器。

```console
apt-get update && apt-get install curl -y
```

現在使用 curl 來存取 *checkip.dyndns.org* 網站。 輸出 IP 位址即會顯示，如下列範例輸出所示。 此 IP 位址會符合針對 loadBalancer 服務所建立和定義的靜態公用 IP 位址：

```console
$ curl -s checkip.dyndns.org

<html><head><title>Current IP Check</title></head><body>Current IP Address: 23.101.128.81</body></html>
```

## <a name="next-steps"></a>後續步驟

若要避免在 Azure Load Balancer 上維護多個公用 IP 位址，您可以改用輸入控制器。 輸入控制器會提供 SSL/TLS 終止、支援 URI 重寫及上游 SSL/TLS 加密等其他權益。 如需詳細資訊，請參閱[在 AKS 中建立基本的輸入控制器][ingress-aks-cluster]。

<!-- LINKS - internal -->
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[az-network-public-ip-list]: /cli/azure/network/public-ip#az-network-public-ip-list
[az-aks-show]: /cli/azure/aks#az-aks-show
[azure-cli-install]: /cli/azure/install-azure-cli
[ingress-aks-cluster]: ./ingress-basic.md
[outbound-connections]: ../load-balancer/load-balancer-outbound-connections.md#scenarios
[public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli