---
title: Azure 上的 Kubernertes 教學課程 - 部署應用程式
description: AKS 教學課程 - 部署應用程式
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: tutorial
ms.date: 02/22/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: e0e349361afaac9aec816d7f5d158322d6f4e691
ms.sourcegitcommit: d7725f1f20c534c102021aa4feaea7fc0d257609
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37100939"
---
# <a name="tutorial-run-applications-in-azure-kubernetes-service-aks"></a>教學課程：執行 Azure Kubernetes Service (AKS) 中的應用程式

在本教學課程 (七個章節的第四部分) 中，已在 Kubernetes 叢集中部署一個應用程式範例。 完成的步驟包括：

> [!div class="checklist"]
> * 更新 Kubernetes 資訊清單檔
> * 在 Kubernetes 中執行應用程式
> * 測試應用程式

在後續的教學課程中，此應用程式會相應放大並且更新。

本教學課程假設讀者已了解 Kubernetes 的基本概念，如需 Kubernetes 的詳細資訊，請參閱 [Kubernetes 文件][kubernetes-documentation]。

## <a name="before-you-begin"></a>開始之前

在先前的教學課程中，已將應用程式封裝成容器映像、將此映像上傳至 Azure Container Registry，並已建立 Kubernetes 叢集。

若要完成本教學課程中的內容，您需要預先建立的 `azure-vote-all-in-one-redis.yaml` Kubernetes 資訊清單檔。 在先前的教學課程中，此檔案已連同應用程式原始程式碼一起下載。 請確認您擁有複製的存放庫，而且已將目錄變更為複製的目錄。

如果您尚未完成這些步驟，而想要跟著做，請回到[教學課程 1 – 建立容器映像][aks-tutorial-prepare-app]。

## <a name="update-manifest-file"></a>更新資訊清單檔

在本教學課程中，我們一直使用 Azure Container Registry (ACR) 來儲存容器映像。 執行應用程式之前，您需要在 Kubernetes 資訊清單檔中更新 ACR 登入伺服器名稱。

請使用 [az acr list][az-acr-list] 命令來取得 ACR 登入伺服器名稱。

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

我們已預先建立登入伺服器名稱為 `microsoft` 的資訊清單檔。 使用任何文字編輯器開啟該檔案。 在此範例中，我們使用 `nano` 開啟檔案。

```console
nano azure-vote-all-in-one-redis.yaml
```

將 `microsoft` 取代為 ACR 登入伺服器名稱。 您可以在資訊清單檔的第 **47** 行找到這個值。

```yaml
containers:
- name: azure-vote-front
  image: microsoft/azure-vote-front:v1
```

上述程式碼就會變成：

```yaml
containers:
- name: azure-vote-front
  image: <acrName>.azurecr.io/azure-vote-front:v1
```

儲存並關閉檔案。

## <a name="deploy-application"></a>部署應用程式

使用 [kubectl apply][kubectl-apply] 命令來執行應用程式。 此命令會剖析資訊清單檔，並建立已定義的 Kubernetes 物件。

```azurecli
kubectl apply -f azure-vote-all-in-one-redis.yaml
```

輸出：

```
deployment "azure-vote-back" created
service "azure-vote-back" created
deployment "azure-vote-front" created
service "azure-vote-front" created
```

## <a name="test-application"></a>測試應用程式

已建立 [Kubernetes 服務][kubernetes-service]，會將應用程式公開至網際網路。 此程序需要數分鐘的時間。

若要監視進度，請使用 [kubectl get service][kubectl-get] 命令搭配 `--watch` 引數。

```azurecli
kubectl get service azure-vote-front --watch
```

一開始，*azure-vote-front* 服務的 *EXTERNAL-IP* 會顯示為 *pending*。

```
azure-vote-front   10.0.34.242   <pending>     80:30676/TCP   7s
```

當 *EXTERNAL-IP* 位址從 *pending* 變成一個 *IP 位址*之後，請使用 `CTRL-C` 來停止 kubectl 監看式流程。

```
azure-vote-front   10.0.34.242   52.179.23.131   80:30676/TCP   2m
```

若要查看應用程式，請瀏覽至外部 IP 位址。

![Azure 上 Kubernetes 叢集的影像](media/container-service-kubernetes-tutorials/azure-vote.png)

如果應用程式並未載入，可能是因為您的映像登錄發生授權問題。

請依照下列步驟來[允許透過 Kubernetes 祕密存取](https://docs.microsoft.com/azure/container-registry/container-registry-auth-aks#access-with-kubernetes-secret)。

## <a name="next-steps"></a>後續步驟

在本教學課程中，是將 Azure 投票應用程式部署到 AKS 中的 Kubernetes 叢集。 完成的工作包括：

> [!div class="checklist"]
> * 下載 Kubernetes 資訊清單檔
> * 在 Kubernetes 中執行應用程式
> * 測試應用程式

請前往下一個教學課程，以了解如何調整 Kubernetes 應用程式和基礎 Kubernetes 基礎架構。

> [!div class="nextstepaction"]
> [調整 Kubernetes 應用程式和基礎結構][aks-tutorial-scale]

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-documentation]: https://kubernetes.io/docs/home/
[kubernetes-service]: https://kubernetes.io/docs/concepts/services-networking/service/

<!-- LINKS - internal -->
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[aks-tutorial-scale]: ./tutorial-kubernetes-scale.md
[az-acr-list]: /cli/azure/acr#list
