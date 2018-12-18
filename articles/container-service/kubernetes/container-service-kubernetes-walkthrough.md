---
title: 快速入門 - 適用於 Linux 的 Azure Kubernetes 叢集
description: 快速了解如何在 Azure Container Service 中使用 Azure CLI 建立適用於 Linux 的 Kubernetes 叢集。
services: container-service
author: neilpeterson
manager: jeconnoc
ms.service: container-service
ms.topic: quickstart
ms.date: 02/26/2018
ms.author: nepeters
ms.custom: H1Hack27Feb2017, mvc, devcenter
ms.openlocfilehash: 2186bd8c28851552bdbd80a74ba7cefd9f7a82d7
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46950347"
---
# <a name="deploy-kubernetes-cluster-for-linux-containers"></a>部署適用於 Linux 容器的 Kubernetes 叢集

[!INCLUDE [aks-preview-redirect.md](../../../includes/aks-preview-redirect.md)]

本快速入門中會使用 Azure CLI 來部署 Kubernetes 叢集。 接著，在叢集上部署和執行多容器應用程式，其中包含 Web 前端和 Redis 執行個體。 完成後，即可透過網際網路來存取應用程式。 

本文件中使用的範例應用程式是以 Python 撰寫。 這裡詳述的概念和步驟可用來將任何容器映像部署到 Kubernetes 叢集中。 在 [GitHub](https://github.com/Azure-Samples/azure-voting-app-redis.git) 上可取得此專案相關的程式碼、Dockerfile 和預先建立之 Kubernetes 資訊清單檔案。

![瀏覽至 Azure 投票的影像](media/container-service-kubernetes-walkthrough/azure-vote.png)

本快速入門假設讀者已了解 Kubernetes 的基本概念，如需 Kubernetes 的詳細資訊，請參閱 [Kubernetes 文件]( https://kubernetes.io/docs/home/)。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯群組。 

下列範例會在 westeurope 位置建立名為 myResourceGroup 的資源群組。

```azurecli-interactive 
az group create --name myResourceGroup --location westeurope
```

輸出：

```json
{
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup",
  "location": "westeurope",
  "managedBy": null,
  "name": "myResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="create-kubernetes-cluster"></a>建立 Kubernetes 叢集

使用 [az acs create](/cli/azure/acs#az-acs-create) 命令，在 Azure Container Service 中建立 Kubernetes 叢集。 下列範例會建立名為 *myK8sCluster* 的叢集，其中包含一個 Linux 主要節點和三個 Linux 代理程式節點。

```azurecli-interactive 
az acs create --orchestrator-type kubernetes --resource-group myResourceGroup --name myK8sCluster --generate-ssh-keys
```

在有限試用之類的某些情況下，Azure 訂用帳戶只擁有 Azure 資源的有限存取權。 如果部署因可用核心受限而失敗，請將 `--agent-count 1` 加入 [az acs create](/cli/azure/acs#az-acs-create) 命令來減少預設代理程式的數量。 

幾分鐘之後，此命令就會完成，並以 json 格式傳回叢集的相關資訊。 

## <a name="connect-to-the-cluster"></a>連接到叢集

若要管理 Kubernetes 叢集，請使用 Kubernetes 命令列用戶端：[kubectl](https://kubernetes.io/docs/user-guide/kubectl/)。 

如果您是使用 Azure CloudShell，則已安裝 kubectl。 如果您想要在本機進行安裝，可以使用 [az acs kubernetes install-cli](/cli/azure/acs/kubernetes#install-cli) 命令。

若要設定 kubectl 來連線到 Kubernetes 叢集，請執行 [az acs kubernetes get-credentials](/cli/azure/acs/kubernetes#get-credentials) 命令。 此步驟會下載憑證並設定 Kubernetes CLI 以供使用。

```azurecli-interactive 
az acs kubernetes get-credentials --resource-group=myResourceGroup --name=myK8sCluster
```

若要驗證叢集的連線，請使用 [kubectl get](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) 命令來傳回叢集節點的清單。

```azurecli-interactive
kubectl get nodes
```

輸出：

```bash
NAME                    STATUS                     AGE       VERSION
k8s-agent-14ad53a1-0    Ready                      10m       v1.6.6
k8s-agent-14ad53a1-1    Ready                      10m       v1.6.6
k8s-agent-14ad53a1-2    Ready                      10m       v1.6.6
k8s-master-14ad53a1-0   Ready,SchedulingDisabled   10m       v1.6.6
```

## <a name="run-the-application"></a>執行應用程式

Kubernetes 資訊清單檔會定義所需的叢集狀態，包括哪些容器映像應在執行中。 例如，資訊清單可用來建立執行 Azure 投票應用程式所需的所有物件。 

建立名為 `azure-vote.yml` 的檔案，並複製到下列 YAML。 如果您在 Azure Cloud Shell 中作業，可以使用 vi 或 Nano 建立這個檔案，猶如使用虛擬或實體系統。

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: azure-vote-back
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: azure-vote-back
    spec:
      containers:
      - name: azure-vote-back
        image: redis
        ports:
        - containerPort: 6379
          name: redis
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-back
spec:
  ports:
  - port: 6379
  selector:
    app: azure-vote-back
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: azure-vote-front
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: azure-vote-front
    spec:
      containers:
      - name: azure-vote-front
        image: microsoft/azure-vote-front:v1
        ports:
        - containerPort: 80
        env:
        - name: REDIS
          value: "azure-vote-back"
---
apiVersion: v1
kind: Service
metadata:
  name: azure-vote-front
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: azure-vote-front
```

使用 [kubectl create](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create) 命令來執行應用程式。

```azurecli-interactive
kubectl create -f azure-vote.yml
```

輸出：

```bash
deployment "azure-vote-back" created
service "azure-vote-back" created
deployment "azure-vote-front" created
service "azure-vote-front" created
```

## <a name="test-the-application"></a>測試應用程式

執行應用程式時會建立 [Kubernetes 服務](https://kubernetes.io/docs/concepts/services-networking/service/)，此服務會向網際網路公開前端應用程式。 此程序需要數分鐘的時間完成。 

若要監視進度，請使用 [kubectl get service](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) 命令搭配 `--watch` 引數。

```azurecli-interactive
kubectl get service azure-vote-front --watch
```

一開始，*azure-vote-front* 服務的 **EXTERNAL-IP** 會顯示為 *pending*。 當 EXTERNAL-IP 位址從 *pending* 變成一個「IP 位址」之後，請使用 `CTRL-C` 來停止 kubectl 監看式流程。 
  
```bash
azure-vote-front   10.0.34.242   <pending>     80:30676/TCP   7s
azure-vote-front   10.0.34.242   52.179.23.131   80:30676/TCP   2m
```

您現在可以瀏覽至外部 IP 位址來查看 Azure 投票應用程式。

![瀏覽至 Azure 投票的影像](media/container-service-kubernetes-walkthrough/azure-vote.png)  

## <a name="delete-cluster"></a>刪除叢集
若不再需要叢集，您可以使用 [az group delete](/cli/azure/group#az-group-delete) 命令來移除資源群組、容器服務和所有相關資源。

```azurecli-interactive 
az group delete --name myResourceGroup --yes --no-wait
```

## <a name="get-the-code"></a>取得程式碼

在本快速入門中，已使用預先建立的容器映像來建立 Kubernetes 部署。 相關的應用程式程式碼、Dockerfile 和 Kubernetes 資訊清單檔案，都可以在 GitHub 上取得。

[https://github.com/Azure-Samples/azure-voting-app-redis](https://github.com/Azure-Samples/azure-voting-app-redis.git)

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已部署 Kubernetes 叢集，並將多容器應用程式部署到此叢集。 

若要深入了解 Azure Container Service，並逐步完成部署範例的完整程式碼，請繼續 Kubernetes 叢集教學課程。

> [!div class="nextstepaction"]
> [管理 ACS Kubernetes 叢集](./container-service-tutorial-kubernetes-prepare-app.md)
