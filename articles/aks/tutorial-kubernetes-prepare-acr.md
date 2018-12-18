---
title: Azure 上的 Kubernetes 教學課程 - 準備 ACR
description: AKS 教學課程 - 準備 ACR
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: tutorial
ms.date: 02/22/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: 4ad5dcb8dbb11f1d6e12e3c19eab5da68009df58
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39430751"
---
# <a name="tutorial-deploy-and-use-azure-container-registry"></a>教學課程：部署和使用 Azure Container Registry

Azure Container Registry (ACR) 是以 Azure 為基礎的私人登錄，用於裝載 Docker 容器映像。 本教學課程 (2/7 部分) 逐步解說如何部署 Azure Container Registry 執行個體，以及將容器映像推送至該執行個體。 完成的步驟包括：

> [!div class="checklist"]
> * 部署 Azure Container Registry (ACR) 執行個體
> * 標記 ACR 的容器映像
> * 將映像上傳至 ACR

在後續教學課程中，此 ACR 執行個體會與 AKS 中的 Kubernetes 叢集整合。

## <a name="before-you-begin"></a>開始之前

在[上一個教學課程][aks-tutorial-prepare-app]中，我們已針對簡單的 Azure Voting 應用程式建立容器映像。 如果您尚未建立 Azure Voting 應用程式映像，請回到[教學課程 1 – 建立容器映像][aks-tutorial-prepare-app]。

本教學課程需要您執行 Azure CLI 2.0.27 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][azure-cli-install]。

## <a name="deploy-azure-container-registry"></a>部署 Azure Container Registry

部署 Azure Container Registry 時，您必須先有資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

使用 [az group create][az-group-create] 命令來建立資源群組。 在此範例中，`eastus` 區域中會建立名為 `myResourceGroup` 的資源群組。

```azurecli
az group create --name myResourceGroup --location eastus
```

使用 [az acr create][az-acr-create] 命令建立 Azure Container Registry。 登錄名稱在 Azure 內必須是唯一的，且包含 5-50 個英數字元。

```azurecli
az acr create --resource-group myResourceGroup --name <acrName> --sku Basic
```

在本教學課程的其餘部分，我們使用 `<acrName>` 作為容器登錄名稱的預留位置。

## <a name="container-registry-login"></a>Container Registry 登入

使用 [az acr login][az-acr-login] 命令登入 ACR 執行個體。 您必須在建立容器登錄時，為容器登錄提供唯一名稱。

```azurecli
az acr login --name <acrName>
```

此命令在完成之後會傳回「登入成功」訊息。

## <a name="tag-container-images"></a>標記容器映像

若要查看目前的映像清單，請使用 [docker images][docker-images] 命令。

```console
docker images
```

輸出：

```
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
azure-vote-front             latest              4675398c9172        13 minutes ago      694MB
redis                        latest              a1b99da73d05        7 days ago          106MB
tiangolo/uwsgi-nginx-flask   flask               788ca94b2313        9 months ago        694MB
```

每個容器映像都必須標記登錄的 loginServer 名稱。 將容器映像推送到映像登錄時，此標籤可用於路由傳送。

使用 [az acr list][az-acr-list] 命令來取得 loginServer 名稱。

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

現在，以容器登錄的 loginServer 標記 `azure-vote-front` 映像。 此外，將 `:v1` 新增至映像名稱的結尾。 此標籤指示映像版本。

```console
docker tag azure-vote-front <acrLoginServer>/azure-vote-front:v1
```

標記之後，請執行 [docker images][docker-images] 以驗證作業。

```console
docker images
```

輸出：

```
REPOSITORY                                           TAG                 IMAGE ID            CREATED             SIZE
azure-vote-front                                     latest              eaf2b9c57e5e        8 minutes ago       716 MB
mycontainerregistry082.azurecr.io/azure-vote-front   v1            eaf2b9c57e5e        8 minutes ago       716 MB
redis                                                latest              a1b99da73d05        7 days ago          106MB
tiangolo/uwsgi-nginx-flask                           flask               788ca94b2313        8 months ago        694 MB
```

## <a name="push-images-to-registry"></a>將映像推送到登錄

將 `azure-vote-front`映像推送至登錄。

使用下列範例，以您環境中的 loginServer 取代 ACR loginServer 名稱。

```console
docker push <acrLoginServer>/azure-vote-front:v1
```

這需要幾分鐘的時間完成。

## <a name="list-images-in-registry"></a>列出登錄中的映像

若要傳回已推送至 Azure Container Registry 的映像清單，請使用 [az acr repository list][az-acr-repository-list] 命令。 以 ACR 執行個體名稱更新命令。

```azurecli
az acr repository list --name <acrName> --output table
```

輸出：

```azurecli
Result
----------------
azure-vote-front
```

而後若要查看特定映像的標籤，請使用 [az acr repository show-tags][az-acr-repository-show-tags] 命令。

```azurecli
az acr repository show-tags --name <acrName> --repository azure-vote-front --output table
```

輸出：

```azurecli
Result
--------
v1
```

當教學課程完成時，私人 Azure Container Registry 執行個體中已儲存容器映像。 在後續的教學課程中，此映像會從 ACR 部署到 Kubernetes 叢集。

## <a name="next-steps"></a>後續步驟

在本教學課程中，已備妥可用於 AKS 叢集中的 Azure Container Registry。 已完成下列步驟：

> [!div class="checklist"]
> * 已部署 Azure Container Registry 執行個體
> * 已標記 ACR 的容器映像
> * 已將映像上傳至 ACR

請前進到下一個教學課程，了解如何在 Azure 中部署 Kubernetes 叢集。

> [!div class="nextstepaction"]
> [部署 Kubernetes 叢集][aks-tutorial-deploy-cluster]

<!-- LINKS - external -->
[docker-images]: https://docs.docker.com/engine/reference/commandline/images/

<!-- LINKS - internal -->
[az-acr-create]: /cli/azure/acr#create
[az-acr-list]: /cli/azure/acr#list
[az-acr-login]: https://docs.microsoft.com/cli/azure/acr#az-acr-login
[az-acr-list]: https://docs.microsoft.com/cli/azure/acr#az-acr-list
[az-acr-repository-list]: /cli/azure/acr/repository#list
[az-acr-repository-show-tags]: /cli/azure/acr/repository#show-tags
[az-group-create]: /cli/azure/group#az-group-create
[azure-cli-install]: /cli/azure/install-azure-cli
[aks-tutorial-deploy-cluster]: ./tutorial-kubernetes-deploy-cluster.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
