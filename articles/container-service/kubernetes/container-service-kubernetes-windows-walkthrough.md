---
title: 快速入門 - 適用於 Windows 的 Azure Kubernetes 叢集
description: 快速了解如何在 Azure Container Service 中使用 Azure CLI 建立適用於 Windows 的 Kubernetes 叢集。
services: container-service
author: dlepow
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 07/18/2017
ms.author: danlep
ms.custom: H1Hack27Feb2017, mvc, devcenter
ms.openlocfilehash: 24659f60e44dc306451528cc7bfb823ad14268a6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946148"
---
# <a name="deploy-kubernetes-cluster-for-windows-containers"></a>部署適用於 Windows 容器的 Kubernetes 叢集

[!INCLUDE [aks-preview-redirect.md](../../../includes/aks-preview-redirect.md)]

Azure CLI 可用來從命令列或在指令碼中建立和管理 Azure 資源。 本指南詳述使用 Azure CLI 在 [Azure Container Service](../container-service-intro.md) 中部署 [Kubernetes](https://kubernetes.io/docs/home/) 叢集。 部署叢集之後，您要使用 Kubernetes`kubectl` 命令列工具與其連線，且您要部署第一個 Windows 容器。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

> [!NOTE]
> 支援 Windows 容器在 Azure Container Service 上的 Kubernetes 處於預覽階段。 
>

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯群組。 

下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

## <a name="create-kubernetes-cluster"></a>建立 Kubernetes 叢集
使用 [az acs create](/cli/azure/acs#az-acs-create) 命令，在 Azure Container Service 中建立 Kubernetes 叢集。 

下列範例會建立名為 myK8sCluster 的叢集，使用一個 Linux 主要節點和兩個 Windows 代理程式節點。 這個範例會建立連線到 Linux 主機所需的 SSH 金鑰。 此範例會以 azureuser 作為系統管理使用者名稱，並以 myPassword12 作為 Windows 節點上的密碼。 將這些值更新為適合您環境的值。 



```azurecli-interactive 
az acs create --orchestrator-type=kubernetes \
    --resource-group myResourceGroup \
    --name=myK8sCluster \
    --agent-count=2 \
    --generate-ssh-keys \
    --windows --admin-username azureuser \
    --admin-password myPassword12
```

幾分鐘之後，此命令就會完成，且會顯示您部署的相關資訊。

## <a name="install-kubectl"></a>安裝 kubectl

若要從用戶端電腦連線到 Kubernetes 叢集，請使用 [`kubectl`](https://kubernetes.io/docs/user-guide/kubectl/) (Kubernetes 命令列用戶端)。 

如果您是使用 Azure CloudShell，就已安裝 `kubectl`。 如果您想要在本機進行安裝，可以使用 [az acs kubernetes install-cli](/cli/azure/acs/kubernetes#install-cli) 命令。

下列 Azure CLI 範例會將 `kubectl` 安裝到您的系統。 在 Windows 上，以系統管理員身分執行此命令。

```azurecli-interactive 
az acs kubernetes install-cli
```


## <a name="connect-with-kubectl"></a>使用 kubectl 連線

若要將 `kubectl` 設定為連線到 Kubernetes 叢集，請執行 [az acs kubernetes get-credentials](/cli/azure/acs/kubernetes#get-credentials) 命令。 下列範例會下載 Kubernetes 叢集的叢集設定。

```azurecli-interactive 
az acs kubernetes get-credentials --resource-group=myResourceGroup --name=myK8sCluster
```

若要確認從您電腦至叢集的連線，請嘗試執行：

```azurecli-interactive
kubectl get nodes
```

`kubectl` 會列出主要和代理程式節點。

```azurecli-interactive
NAME                    STATUS                     AGE       VERSION
k8s-agent-98dc3136-0    Ready                      5m        v1.5.3
k8s-agent-98dc3136-1    Ready                      5m        v1.5.3
k8s-master-98dc3136-0   Ready,SchedulingDisabled   5m        v1.5.3

```

## <a name="deploy-a-windows-iis-container"></a>部署 Windows IIS 容器

您可以在 Kubernetes Pod 內執行 Docker 容器，其中包含一個或多個容器。 

此基本範例使用 JSON 檔案來指定 Microsoft Internet Information Server (IIS) 容器，然後使用 `kubctl apply` 命令來建立 Pod。 

建立名為 `iis.json` 的本機檔案，並複製下列文字。 此檔案會告訴 Kubernetes 使用來自 [Docker 中樞](https://hub.docker.com/r/microsoft/iis/)的公用容器映像，在 Windows Server 2016 Nano Server 上執行 IIS。 該容器會使用連接埠 80，但一開始只能在叢集網路中存取。

 ```JSON
 {
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "iis",
    "labels": {
      "name": "iis"
    }
  },
  "spec": {
    "containers": [
      {
        "name": "iis",
        "image": "microsoft/iis:nanoserver",
        "ports": [
          {
          "containerPort": 80
          }
        ]
      }
    ],
    "nodeSelector": {
     "beta.kubernetes.io/os": "windows"
     }
   }
 }
 ```

若要啟動 Pod，請輸入：
  
```azurecli-interactive
kubectl apply -f iis.json
```  

若要追蹤部署，請輸入：
  
```azurecli-interactive
kubectl get pods
```

部署 Pod 時，狀態會是 `ContainerCreating`。 可能需要幾分鐘，容器才能進入 `Running` 狀態。

```azurecli-interactive
NAME     READY        STATUS        RESTARTS    AGE
iis      1/1          Running       0           32s
```

## <a name="view-the-iis-welcome-page"></a>檢視 IIS 歡迎使用頁面

若要使用公用 IP 位址將 Pod 向全球公開，請輸入下列命令：

```azurecli-interactive
kubectl expose pods iis --port=80 --type=LoadBalancer
```

使用此命令，Kubernetes 會以服務的公用 IP 位址來建立服務和 Azure Load Balancer 規則。 

執行下列命令來查看服務的狀態。

```azurecli-interactive
kubectl get svc
```

IP 位址一開始會顯示為 `pending`。 幾分鐘之後，`iis` 的外部 IP 位址會完成設定：
  
```azurecli-interactive
NAME         CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE       
kubernetes   10.0.0.1       <none>          443/TCP        21h       
iis          10.0.111.25    13.64.158.233   80/TCP         22m
```

若要在外部 IP 位址查看預設的 IIS 歡迎使用頁面，您可以使用指定的網頁瀏覽器：

![瀏覽至 IIS 的影像](./media/container-service-kubernetes-windows-walkthrough/kubernetes-iis.png)  


## <a name="delete-cluster"></a>刪除叢集
若不再需要叢集，您可以使用 [az group delete](/cli/azure/group#az-group-delete) 命令來移除資源群組、容器服務和所有相關資源。

```azurecli-interactive 
az group delete --name myResourceGroup
```


## <a name="next-steps"></a>後續步驟

在本快速入門中，您部署了 Kubernetes 叢集、與 `kubectl` 連線，並使用 IIS 容器部署了 Pod。 若要深入了解 Azure Container Service，請繼續進行 Kubernetes 教學課程。

> [!div class="nextstepaction"]
> [管理 ACS Kubernetes 叢集](container-service-tutorial-kubernetes-prepare-app.md)
