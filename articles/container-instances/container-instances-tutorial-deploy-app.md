---
title: Azure 容器執行個體教學課程 - 部署應用程式
description: Azure 容器執行個體教學課程第 3 部分 (共 3 部分) - 部署應用程式
services: container-instances
author: mmacy
manager: jeconnoc
ms.service: container-instances
ms.topic: tutorial
ms.date: 03/21/2018
ms.author: marsma
ms.custom: mvc
ms.openlocfilehash: 5a68baa0c04dd90236e99cf010c96b1876fb4638
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39425705"
---
# <a name="tutorial-deploy-a-container-to-azure-container-instances"></a>教學課程：將容器部署至 Azure Container Instances

這是三部分系列的最後一個教學課程。 在系列的較前段，我們[已建立容器映像](container-instances-tutorial-prepare-app.md)並[推送至 Azure Container Registry](container-instances-tutorial-prepare-acr.md)。 本文會將容器部署至 Azure Container Instances，以完成本系列。

在本教學課程中，您：

> [!div class="checklist"]
> * 將容器從 Azure Container Registry 部署至 Azure Container Instances
> * 在瀏覽器中檢視執行中的應用程式
> * 顯示容器的記錄

## <a name="before-you-begin"></a>開始之前

[!INCLUDE [container-instances-tutorial-prerequisites](../../includes/container-instances-tutorial-prerequisites.md)]

## <a name="deploy-the-container-using-the-azure-cli"></a>使用 Azure CLI 來部署容器

在本節中，您會使用 Azure CLI 來部署[第一個教學課程](container-instances-tutorial-prepare-app.md)內建且在[第二個教學課程](container-instances-tutorial-prepare-acr.md)中推送至 Azure Container Registry 的映像。 請先確定您已完成這些教學課程後，再繼續執行。

### <a name="get-registry-credentials"></a>取得登錄認證

當您部署私人容器登錄 (類似[第二個教學課程](container-instances-tutorial-prepare-acr.md)中建立的容器登錄) 中裝載的映像時，必須提供登錄的認證。

首先，取得容器登錄登入伺服器的完整名稱 (以您的登錄名稱取代 `<acrName>`)：

```azurecli
az acr show --name <acrName> --query loginServer
```

接著，取得容器登錄密碼：

```azurecli
az acr credential show --name <acrName> --query "passwords[0].value"
```

### <a name="deploy-container"></a>部署容器

現在使用 [az container create][az-container-create] 命令來部署容器。 以您從先前兩個命令取得的值取代 `<acrLoginServer>` 和 `<acrPassword>`。 以您的容器登錄名稱取代 `<acrName>`。

```azurecli
az container create --resource-group myResourceGroup --name aci-tutorial-app --image <acrLoginServer>/aci-tutorial-app:v1 --cpu 1 --memory 1 --registry-login-server <acrLoginServer> --registry-username <acrName> --registry-password <acrPassword> --dns-name-label aci-demo --ports 80
```

在幾秒內，您應該會從 Azure 收到首次回應。 `--dns-name-label` 值在您建立容器執行個體所在的 Azure 區域中必須是唯一的。 如果您在執行命令時收到 **DNS 名稱標籤**錯誤訊息，請修改上方命令中的值。

### <a name="verify-deployment-progress"></a>確認部署進度

若要檢視部署的狀態，請使用 [az container show][az-container-show]：

```azurecli
az container show --resource-group myResourceGroup --name aci-tutorial-app --query instanceView.state
```

重複執行 [az container show][az-container-show] 命令，直到狀態從 Pending (暫止) 變更為 Running (執行中) 為止，這應該會在一分鐘內完成。 當容器狀態為 *Running* (執行中) 時，請繼續進行下一個步驟。

## <a name="view-the-application-and-container-logs"></a>檢視應用程式和容器記錄

部署成功之後，請使用 [az container show][az-container-show] 命令來顯示容器的完整網域名稱 (FQDN)：

```bash
az container show --resource-group myResourceGroup --name aci-tutorial-app --query ipAddress.fqdn
```

例如︰
```console
$ az container show --resource-group myResourceGroup --name aci-tutorial-app --query ipAddress.fqdn
"aci-demo.eastus.azurecontainer.io"
```

若要查看執行中的應用程式，在您慣用的瀏覽器中瀏覽至顯示的 DNS 名稱：

![瀏覽器中的 Hello World 應用程式][aci-app-browser]

您也可以檢視容器的記錄輸出：

```azurecli
az container logs --resource-group myResourceGroup --name aci-tutorial-app
```

範例輸出︰

```bash
$ az container logs --resource-group myResourceGroup --name aci-tutorial-app
listening on port 80
::ffff:10.240.0.4 - - [21/Jul/2017:06:00:02 +0000] "GET / HTTP/1.1" 200 1663 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36"
::ffff:10.240.0.4 - - [21/Jul/2017:06:00:02 +0000] "GET /favicon.ico HTTP/1.1" 404 150 "http://aci-demo.eastus.azurecontainer.io/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36"
```

## <a name="clean-up-resources"></a>清除資源

如果您已不再需要這個教學課程系列中建立的任何資源，便可以執行 [az group delete][az-group-delete] 命令來移除資源群組及其包含的所有資源。 此命令除了會刪除執行中的容器和所有相關資源之外，也會刪除您所建立的容器登錄。

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已完成將容器部署至 Azure Container Instances 的程序。 已完成下列步驟：

> [!div class="checklist"]
> * 使用 Azure CLI 從 Azure Container Registry 部署容器
> * 在瀏覽器中檢視應用程式
> * 檢視容器記錄

您現在已有基本概念，請繼續深入了解 Azure Container Instances，例如容器群組的運作方式：

> [!div class="nextstepaction"]
> [Azure Container Instances 中的容器群組](container-instances-container-groups.md)

<!-- IMAGES -->
[aci-app-browser]: ./media/container-instances-quickstart/aci-app-browser.png

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show
[az-group-delete]: /cli/azure/group#az-group-delete
[azure-cli-install]: /cli/azure/install-azure-cli
[prepare-app]: ./container-instances-tutorial-prepare-app.md
