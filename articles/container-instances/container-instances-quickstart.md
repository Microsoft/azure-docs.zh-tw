---
title: 快速入門 - 在 Azure Container Instances 中執行應用程式
description: 在本快速入門中，您會使用 Azure CLI 將 Docker 容器中所執行的應用程式部署至 Azure Container Instances
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: quickstart
ms.date: 10/02/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: 7db3d9a076fe9ff5b8bbf970705b82a3f0d5ce54
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48855658"
---
# <a name="quickstart-run-an-application-in-azure-container-instances"></a>快速入門：在 Azure Container Instances 中執行應用程式

使用 Azure Container Instances 在 Azure 中簡潔且快速地執行 Docker 容器。 您不需要部署虛擬機器，也不需要使用完整的容器協調流程平台，如 Kubernetes。 在本快速入門中，您會使用 Azure 入口網站在 Azure 中建立容器，並使用完整網域名稱 (FQDN) 讓其應用程式可供使用。 執行單一部署命令的幾秒之後，您可以瀏覽至執行中的應用程式：

![在瀏覽器中檢視部署至 Azure Container Instances 的應用程式][aci-app-browser]

如果您沒有 Azure 訂用帳戶，請在開始前建立 [[免費帳戶]][azure-account]。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

您可以使用 Azure Cloud Shell 或安裝在本機的 Azure CLI 來完成此快速入門。 如果您想要在本機使用，您需要 2.0.27 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI][azure-cli-install]。

## <a name="create-a-resource-group"></a>建立資源群組

Azure 容器執行個體和所有 Azure 資源相同，都必須部署到資源群組中。 資源群組可讓您組織和管理相關的 Azure 資源。

首先，使用下列 [az group create][az-group-create] 命令，在 *eastus* 位置中建立名為 *myResourceGroup* 的資源群組：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-container"></a>建立容器

有了資源群組之後，現在您可以在 Azure 中執行容器。 若要使用 Azure CLI 建立容器執行個體，請在 [az container create][az-container-create] 命令中提供資源群組名稱、容器執行個體名稱和 Docker 容器映像。 您可以指定一或多個要開啟的連接埠和 (或) DNS 名稱標籤，以將您的容器公開至網際網路。 在本快速入門中，您會部署附有 DMS 名稱標籤的一個容器，其中裝載以 Node.js 撰寫的小型 Web 應用程式。

執行下列命令以啟動容器執行個體。 `--dns-name-label` 值在您建立執行個體所在的 Azure 區域中必須是唯一的。 如果出現「DNS 名稱標籤無法使用」錯誤訊息，請嘗試使用不同的 DNS 名稱標籤。

```azurecli-interactive
az container create --resource-group myResourceGroup --name mycontainer --image microsoft/aci-helloworld --dns-name-label aci-demo --ports 80
```

您應該會在幾秒內獲得 Azure CLI 的回應，指出部署已。 請使用 [az container show][az-container-show] 命令查看其狀態：

```azurecli-interactive
az container show --resource-group myResourceGroup --name mycontainer --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" --out table
```

當您執行命令時，就會顯示容器的完整網域名稱 (FQDN) 及其佈建狀態。

```console
$ az container show --resource-group myResourceGroup --name mycontainer --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" --out table
FQDN                               ProvisioningState
---------------------------------  -------------------
aci-demo.eastus.azurecontainer.io  Succeeded
```

如果容器的 `ProvisioningState` 為 [成功]，請在瀏覽器中瀏覽至其 FQDN。 如果您看到如下的網頁，恭喜您！ 您已將 Docker 容器中執行的應用程式成功部署至 Azure。

![顯示在 Azure 容器執行個體中執行之應用程式的瀏覽器螢幕擷取畫面][aci-app-browser]

如果一開始應用程式並未顯示，您可能需要等候幾秒鐘讓 DNS 傳播完成，然後再次嘗試重新整理瀏覽器。

## <a name="pull-the-container-logs"></a>提取容器記錄

如果您要對容器或其執行的應用程式進行疑難排解 (或只是要檢視其輸出)，請先檢視容器執行個體的記錄。

使用 [az container logs][az-container-logs] 命令提取容器執行個體記錄：

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer
```

輸出會顯示容器的記錄，且應該會顯示您在瀏覽器中檢視應用程式時產生的 HTTP GET 要求。

```console
$ az container logs --resource-group myResourceGroup --name mycontainer
listening on port 80
::ffff:10.240.255.105 - - [01/Oct/2018:18:25:51 +0000] "GET / HTTP/1.0" 200 1663 "-" "-"
::ffff:10.240.255.106 - - [01/Oct/2018:18:31:04 +0000] "GET / HTTP/1.1" 200 1663 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
::ffff:10.240.255.106 - - [01/Oct/2018:18:31:04 +0000] "GET /favicon.ico HTTP/1.1" 404 150 "http://aci-demo.eastus.azurecontainer.io/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36"
```

## <a name="attach-output-streams"></a>附加輸出資料流

除了檢視記錄外，您還可以將您的本機標準輸出和標準錯誤資料流附加至容器的這些項目。

首先，請執行 [az container attach][az-container-attach] 命令將本機主控台附加至容器的輸出資料流：

```azurecli-interactive
az container attach --resource-group myResourceGroup -n mycontainer
```

在附加之後，重新整理瀏覽器幾次以產生一些額外的輸出。 作業完成後，請使用 `Control+C` 將主控台中斷連結。 您應該會看到如下所示的輸出：

```console
$ az container attach --resource-group myResourceGroup -n mycontainer
Container 'mycontainer' is in state 'Running'...
(count: 1) (last timestamp: 2018-03-15 21:17:59+00:00) pulling image "microsoft/aci-helloworld"
(count: 1) (last timestamp: 2018-03-15 21:18:05+00:00) Successfully pulled image "microsoft/aci-helloworld"
(count: 1) (last timestamp: 2018-03-15 21:18:05+00:00) Created container with id 3534a1e2ee392d6f47b2c158ce8c1808d1686fc54f17de3a953d356cf5f26a45
(count: 1) (last timestamp: 2018-03-15 21:18:06+00:00) Started container with id 3534a1e2ee392d6f47b2c158ce8c1808d1686fc54f17de3a953d356cf5f26a45

Start streaming logs:
listening on port 80
::ffff:10.240.255.105 - - [15/Mar/2018:21:18:26 +0000] "GET / HTTP/1.1" 200 1663 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.146 Safari/537.36"
::ffff:10.240.255.105 - - [15/Mar/2018:21:18:26 +0000] "GET /favicon.ico HTTP/1.1" 404 150 "http://aci-demo.eastus.azurecontainer.io/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.146 Safari/537.36"
::ffff:10.240.255.107 - - [15/Mar/2018:21:18:44 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.146 Safari/537.36"
::ffff:10.240.255.107 - - [15/Mar/2018:21:18:47 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.146 Safari/537.36"
```

## <a name="clean-up-resources"></a>清除資源

當容器使用完畢後，請使用 [az container delete][az-container-delete] 命令來移除容器：

```azurecli-interactive
az container delete --resource-group myResourceGroup --name mycontainer
```

若要確認否已將容器刪除，請執行 [az container list](/cli/azure/container#az-container-list) 命令：

```azurecli-interactive
az container list --resource-group myResourceGroup --output table
```

**mycontainer** 不應該出現在命令的輸出中。 如果您的資源群組中沒有任何其他容器，則不會顯示任何輸出。

當您使用完 *myResourceGroup* 資源群組和其中包含的所有資源後，請使用 [az group delete][az-group-delete] 命令加以刪除：

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已使用公用 Docker Hub 登錄中的映像建立 Azure 容器執行個體。 如果您想要建置容器映像，並從私人的 Azure 容器登錄進行部署，請繼續進行 Azure Container Instances 教學課程。

> [!div class="nextstepaction"]
> [Azure 容器執行個體教學課程](./container-instances-tutorial-prepare-app.md)

若要試用在 Azure 上的協調流程系統中執行容器的選項，請參閱 [Service Fabric][service-fabric] 或 [Azure Kubernetes Service (ACS)][container-service] 快速入門。

<!-- IMAGES -->
[aci-app-browser]: ./media/container-instances-quickstart/aci-app-browser.png

<!-- LINKS - External -->
[app-github-repo]: https://github.com/Azure-Samples/aci-helloworld.git
[azure-account]: https://azure.microsoft.com/free/
[node-js]: http://nodejs.org

<!-- LINKS - Internal -->
[az-container-attach]: /cli/azure/container#az-container-attach
[az-container-create]: /cli/azure/container#az-container-create
[az-container-delete]: /cli/azure/container#az-container-delete
[az-container-list]: /cli/azure/container#az-container-list
[az-container-logs]: /cli/azure/container#az-container-logs
[az-container-show]: /cli/azure/container#az-container-show
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[azure-cli-install]: /cli/azure/install-azure-cli
[container-service]: ../aks/kubernetes-walkthrough.md
[service-fabric]: ../service-fabric/service-fabric-quickstart-containers.md
