---
title: 快速入門 - 使用 Azure CLI 在 Azure 中建立私用的 Docker 登錄
description: 快速了解如何使用 Azure CLI 建立私用的 Docker 容器登錄。
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: quickstart
ms.date: 03/03/2018
ms.author: danlep
ms.custom: H1Hack27Feb2017, mvc
ms.openlocfilehash: 57c72056b669865278fa8109cd7f4963a1f0887a
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48855233"
---
# <a name="quickstart-create-a-container-registry-using-the-azure-cli"></a>快速入門：使用 Azure CLI 建立容器登錄

Azure Container Registry 是用於儲存私用 Docker 容器映像的受控 Docker 容器登錄服務。 本指南詳述如何使用 Azure CLI 建立 Azure Container Registry 執行個體、將容器映像推送到登錄中，最後將容器從登錄部署至 Azure 容器執行個體 (ACI)。

此快速入門需要您執行 Azure CLI 2.0.27 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][azure-cli]。

您也必須在本機上安裝 Docker。 Docker 提供可輕鬆在任何 [macOS][docker-mac]、[Windows][docker-windows] 或 [Linux][docker-linux] 系統上設定 Docker 的套件。

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create][az-group-create] 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組。

```azurecli
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-container-registry"></a>建立容器登錄庫

在本快速入門中，您會建立「基本」登錄。 有數個不同的 SKU 提供 Azure Container Registry，簡略說明於下表。 如需個別項目更詳細的資訊，請參閱[容器登錄 SKU][container-registry-skus]。

[!INCLUDE [container-registry-sku-matrix](../../includes/container-registry-sku-matrix.md)]

使用 [az acr create][az-acr-create] 命令建立 ACR 執行個體。 登錄名稱在 Azure 內必須是唯一的，且包含 5-50 個英數字元。 下列範例中使用 *myContainerRegistry007*。 請將此更新為唯一的值。

```azurecli
az acr create --resource-group myResourceGroup --name myContainerRegistry007 --sku Basic
```

建立登錄時，輸出大致如下：

```json
{
  "adminUserEnabled": false,
  "creationDate": "2017-09-08T22:32:13.175925+00:00",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myContainerRegistry007",
  "location": "eastus",
  "loginServer": "myContainerRegistry007.azurecr.io",
  "name": "myContainerRegistry007",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

在本快速入門的其餘部分，`<acrName>` 是容器登錄名稱的預留位置。

## <a name="log-in-to-acr"></a>登入 ACR

發送和提取容器映像之前，您必須登入 ACR 執行個體。 若要這樣做，請使用 [az acr login][az-acr-login] 命令。

```azurecli
az acr login --name <acrName>
```

完成後，此命令會傳回 `Login Succeeded` 訊息。

## <a name="push-image-to-acr"></a>推送映像到 ACR

若要推送映像到 Azure Container Registry，您必須先有映像。 如果您還沒有任何本機容器映像，請執行下列命令，從 Docker 中樞提取現有的映像。

```bash
docker pull microsoft/aci-helloworld
```

您必須使用 ACR 登入伺服器的完整名稱來標記映像，才能將映像推送至您的登錄。 請執行下列命令，取得 ACR 執行個體的完整登入伺服器名稱。

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

使用 [docker tag][docker-tag] 命令來標記映像。 將 `<acrLoginServer>` 取代為 ACR 執行個體的登入伺服器名稱。

```bash
docker tag microsoft/aci-helloworld <acrLoginServer>/aci-helloworld:v1
```

最後，使用 [docker push][docker-push] 將映像推送到 ACR 執行個體。 將 `<acrLoginServer>` 取代為 ACR 執行個體的登入伺服器名稱。

```bash
docker push <acrLoginServer>/aci-helloworld:v1
```

## <a name="list-container-images"></a>列出容器映像

下列範例會列出登錄中的存放庫：

```azurecli
az acr repository list --name <acrName> --output table
```

輸出：

```bash
Result
----------------
aci-helloworld
```

下列範例會列出在 **aci-helloworld** 的標籤上。

```azurecli
az acr repository show-tags --name <acrName> --repository aci-helloworld --output table
```

輸出：

```bash
Result
--------
v1
```

## <a name="deploy-image-to-aci"></a>將映像部署至 ACI

若要從您建立的登錄部署容器執行個體，您必須在部署時提供登錄認證。 在生產案例中，您應該使用[服務主體][container-registry-auth-aci]進行容器登錄存取，但若要盡快完成本快速入門，請使用下列命令在您的登錄上啟用管理使用者：

```azurecli
az acr update --name <acrName> --admin-enabled true
```

啟用系統管理員後，使用者名稱會與您的登錄名稱相同，您可以利用此命令擷取密碼：

```azurecli
az acr credential show --name <acrName> --query "passwords[0].value"
```

若要部署需要 1 個 CPU 核心和 1 GB 記憶體的容器映像，請執行下列命令。 以您從先前命令取得的值取代 `<acrName>`、`<acrLoginServer>` 和 `<acrPassword>`。

```azurecli
az container create --resource-group myResourceGroup --name acr-quickstart --image <acrLoginServer>/aci-helloworld:v1 --cpu 1 --memory 1 --registry-username <acrName> --registry-password <acrPassword> --dns-name-label aci-demo --ports 80
```

您應會從 Azure Resource Manager 得到首次回應，其中包含容器的詳細資料。 若要監視容器的狀態以及查看它何時執行，請重複 [az container show][az-container-show]。 此作業所需的時間應該不到一分鐘。

```azurecli
az container show --resource-group myResourceGroup --name acr-quickstart --query instanceView.state
```

## <a name="view-the-application"></a>檢視應用程式

部署成功至 ACI 之後，請使用 [az container show][az-container-show] 命令來擷取容器的 FQDN：

```azurecli
az container show --resource-group myResourceGroup --name acr-quickstart --query ipAddress.fqdn
```

範例輸出：`"aci-demo.eastus.azurecontainer.io"`

若要查看執行中的應用程式，請在您慣用的瀏覽器中瀏覽至該公用 IP 位址。

![瀏覽器中的 Hello World 應用程式][aci-app-browser]

## <a name="clean-up-resources"></a>清除資源

若不再需要，您可以使用 [az group delete][az-group-delete] 命令移除資源群組、ACR 執行個體和所有容器映像。

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已使用 Azure CLI 建立 Azure Container Registry、將容器映像推送至登錄，並透過 Azure 容器執行個體啟動其執行個體。 請繼續進行 Azure 容器執行個體教學課程，以深入了解 ACI。

> [!div class="nextstepaction"]
> [Azure 容器執行個體教學課程][container-instances-tutorial-prepare-app]

<!-- IMAGES> -->
[aci-app-browser]: ../container-instances/media/container-instances-quickstart/aci-app-browser.png


<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[azure-cli]: /cli/azure/install-azure-cli
[az-container-show]: /cli/azure/container#az-container-show
[container-instances-tutorial-prepare-app]: ../container-instances/container-instances-tutorial-prepare-app.md
[container-registry-skus]: container-registry-skus.md
[container-registry-auth-aci]: container-registry-auth-aci.md
