---
title: 快速入門 - 將 Azure Container Registry 事件傳送至事件方格
description: 在本快速入門中，您可以針對容器登錄啟用事件方格事件，然後將容器映像推送和刪除事件傳送至範例應用程式。
services: container-registry
author: mmacy
manager: jeconnoc
ms.service: container-registry
ms.topic: article
ms.date: 08/23/2018
ms.author: marsma
ms.openlocfilehash: 6ff83885ba80f0399f7b085970b1191e8e4cd999
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746503"
---
# <a name="quickstart-send-container-registry-events-to-event-grid"></a>快速入門：將容器登錄事件傳送至事件方格

Azure 事件方格是完全受控的事件路由服務，可使用發佈-訂閱模型提供統一的事件耗用量。 在本快速入門中，您可以使用 Azure CLI 來建立容器登錄、訂閱登錄事件，然後部署範例 Web 應用程式來接收事件。 最後，您會觸發容器映像 `push` 和 `delete` 事件，並在範例應用程式中檢視事件裝載。

當您完成本文中的步驟之後，自您的容器登錄傳送至事件方格的事件會出現在範例 Web 應用程式中：

![呈現包含三個已接收事件之範例 Web 應用程式的網頁瀏覽器][sample-app-01]

如果您沒有 Azure 訂用帳戶，請在開始前建立 [[免費帳戶]][azure-account]。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

本文中的 Azure CLI 命令已針對 **Bash** 殼層進行格式化。 如果您使用不同的殼層 (例如 PowerShell 或命令提示字元)，您可能需要據以調整行接續字元或變數指派行。 本文使用變數來將需要編輯的命令數量降至最低。

## <a name="create-a-resource-group"></a>建立資源群組

Azure 資源群組是一種邏輯容器，您可在其中部署與管理 Azure 資源。 下列 [az group create][az-group-create] 命令會在 *eastus* 區域中建立名為 *myResourceGroup* 的資源群組。 如果您想要針對資源群組使用不同名稱，請將 `RESOURCE_GROUP_NAME` 設定為不同的值。

```azurecli-interactive
RESOURCE_GROUP_NAME=myResourceGroup

az group create --name $RESOURCE_GROUP_NAME --location eastus
```

## <a name="create-a-container-registry"></a>建立容器登錄庫

接下來，使用下列命令來將容器登錄部署到資源群組。 執行 [az acr create][az-acr-create] 命令之前，先將 `ACR_NAME` 設定為您的登錄名稱。 此名稱在 Azure 內必須是唯一的，且長度限制為 5-50 個英數字元。

```azurecli-interactive
ACR_NAME=<acrName>

az acr create --resource-group $RESOURCE_GROUP_NAME --name $ACR_NAME --sku Basic
```

一旦建立登錄之後，Azure CLI 就會傳回類似下列的輸出：

```json
{
  "adminUserEnabled": false,
  "creationDate": "2018-08-16T20:02:46.569509+00:00",
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myregistry",
  "location": "eastus",
  "loginServer": "myregistry.azurecr.io",
  "name": "myregistry",
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

## <a name="create-an-event-endpoint"></a>建立事件端點

在本節中，您可以使用位於 GitHub 存放庫的 Resource Manager 範本，來將預先建置的範例 Web 應用程式部署到 Azure App Service。 您稍後會訂閱登錄的事件方格事件，並指定此應用程式作為要將事件傳送至其中的端點。

若要部署範例應用程式，請將 `SITE_NAME` 設為您 Web 應用程式的唯一名稱，然後執行下列命令。 網站名稱在 Azure 中必須是唯一的，因為它會構成 Web 應用程式完整網域名稱 (FQDN) 的一部分。 在後續小節中，您會在網頁瀏覽器中瀏覽至應用程式的 FQDN，以檢視您的登錄事件。

```azurecli-interactive
SITE_NAME=<your-site-name>

az group deployment create \
    --resource-group $RESOURCE_GROUP_NAME \
    --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" \
    --parameters siteName=$SITE_NAME hostingPlanName=$SITE_NAME-plan
```

成功部署之後 (可能需要幾分鐘的時間)，開啟瀏覽器，然後瀏覽至您的 Web 應用程式以確定它正在執行：

`http://<your-site-name>.azurewebsites.net`

您應該會看到範例應用程式已呈現，且其中未顯示任何事件訊息：

![呈現範例 Web 應用程式且未顯示任何事件的網頁瀏覽器][sample-app-02]

[!INCLUDE [event-grid-register-provider-cli.md](../../includes/event-grid-register-provider-cli.md)]

## <a name="subscribe-to-registry-events"></a>訂閱登錄事件

在事件方格中，您可以訂閱「主題」，以告知它您想要追蹤的事件，以及要將它們傳送至何處。 下列 [az eventgrid event-subscription create][az-eventgrid-event-subscription-create] 命令會訂閱您所建立的容器登錄，並指定您的 Web 應用程式 URL 作為應將事件傳送至其中的端點。 您在先前小節中所填入的環境變數會在此處重複使用，因此不需進行任何編輯。

```azurecli-interactive
ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)
APP_ENDPOINT=https://$SITE_NAME.azurewebsites.net/api/updates

az eventgrid event-subscription create \
    --name event-sub-acr \
    --resource-id $ACR_REGISTRY_ID \
    --endpoint $APP_ENDPOINT
```

完成訂閱時，您應該會輸出類似下列的內容：

```JSON
{
  "destination": {
    "endpointBaseUrl": "https://eventgridviewer.azurewebsites.net/api/updates",
    "endpointType": "WebHook",
    "endpointUrl": null
  },
  "filter": {
    "includedEventTypes": [
      "All"
    ],
    "isSubjectCaseSensitive": null,
    "subjectBeginsWith": "",
    "subjectEndsWith": ""
  },
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myregistry/providers/Microsoft.EventGrid/eventSubscriptions/event-sub-acr",
  "labels": null,
  "name": "event-sub-acr",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "topic": "/subscriptions/<Subscription ID>/resourceGroups/myresourcegroup/providers/microsoft.containerregistry/registries/myregistry",
  "type": "Microsoft.EventGrid/eventSubscriptions"
}
```

## <a name="trigger-registry-events"></a>觸發登錄事件

現在，範例應用程式已啟動且正在執行，而您已使用事件方格來訂閱登錄，您已準備好產生某些事件。 在本節中，您將使用 ACR 組建來建置容器映像並推送至您的登錄。 ACR 組建是一個 Azure Container Registry 功能，可讓您在雲端中建置容器映像，而不需在本機電腦上安裝 Docker 引擎。

### <a name="build-and-push-image"></a>建置與推送映像

執行下列 Azure CLI 命令，從 GitHub 存放庫的內容建置容器映像。 根據預設，ACR 組建會將已成功建置的映像自動推送至您的登錄，其會產生 `ImagePushed` 事件。

```azurecli-interactive
az acr build --registry $ACR_NAME --image myimage:v1 https://github.com/Azure-Samples/acr-build-helloworld-node.git
```

當 ACR 組建建置您的映像，接著加以推送時，您應該會看到類似下列的輸出。 為了簡潔起見，已將下列範例輸出截斷。

```console
$ az acr build -r $ACR_NAME --image myimage:v1 https://github.com/Azure-Samples/acr-build-helloworld-node.git
Sending build context to ACR...
Queued a build with build ID: aa2
Waiting for build agent...
2018/08/16 22:19:38 Using acb_vol_27a2afa6-27dc-4ae4-9e52-6d6c8b7455b2 as the home volume
2018/08/16 22:19:38 Setting up Docker configuration...
2018/08/16 22:19:39 Successfully set up Docker configuration
2018/08/16 22:19:39 Logging in to registry: myregistry.azurecr.io
2018/08/16 22:19:55 Successfully logged in
Sending build context to Docker daemon  94.72kB
Step 1/5 : FROM node:9-alpine
...
```

若要確認建置的映像位於您的登錄中，請執行下列命令以檢視 "myimage" 存放庫中的標記：

```azurecli-interactive
az acr repository show-tags --name $ACR_NAME --repository myimage
```

您所建置之映像的 "v1" 標記應該會出現在輸出中，如下所示：

```console
$ az acr repository show-tags --name $ACR_NAME --repository myimage
[
  "v1"
]
```

### <a name="delete-the-image"></a>刪除映像

現在，使用 [az acr repository delete][az-acr-repository-delete] 命令刪除映像，藉以產生 `ImageDeleted` 事件：

```azurecli-interactive
az acr repository delete --name $ACR_NAME --image myimage:v1
```

您應該會看到類似下列的輸出，要求確認是否要刪除資訊清單和相關聯的映像：

```console
$ az acr repository delete --name $ACR_NAME --image myimage:v1
This operation will delete the manifest 'sha256:f15fa9d0a69081ba93eee308b0e475a54fac9c682196721e294b2bc20ab23a1b' and all the following images: 'myimage:v1'.
Are you sure you want to continue? (y/n): y
```

## <a name="view-registry-events"></a>檢視登錄事件

您現在已將映像推送至登錄，接著將其刪除。 瀏覽至您的事件方格檢視器 Web 應用程式，而您應該會看到 `ImageDeleted` 和 `ImagePushed` 事件。 您也可能會看到因執行[訂閱登錄事件](#subscribe-to-registry-events)一節中的命令而產生的訂閱驗證事件。

下列螢幕擷取畫面顯示包含三個事件的範例應用程式，而 `ImageDeleted` 事件已展開以顯示其詳細資料。

![顯示含有 ImagePushed 和 ImageDeleted 事件之範例應用程式的網頁瀏覽器][sample-app-03]

恭喜！ 如果您看到 `ImagePushed` 和 `ImageDeleted` 事件，表示您的登錄正在將事件傳送至事件方格，而事件方格正在將那些事件轉送至您的 Web 應用程式端點。

## <a name="clean-up-resources"></a>清除資源

一旦您完成使用在本快速入門中建立的資源之後，您可以使用下列 Azure CLI 命令來將它們全部刪除。 當您刪除資源群組時，即會永久刪除所有包含在內的資源。

**警告**：此作業無法復原。 執行此命令之前，請確定您不再需要群組中的任何資源。

```azurecli-interactive
az group delete --name $RESOURCE_GROUP_NAME
```

## <a name="event-grid-event-schema"></a>Event Grid 事件結構描述

您可以在事件方格文件中找到 Azure Container Registry 事件訊息結構描述參考：

[Container Registry 的 Azure Event Grid 事件結構描述](../event-grid/event-schema-container-registry.md)

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已部署容器登錄、使用 ACR 組建來建置映像、刪除它，並已使用範例應用程式從事件方格中取用您的登錄事件。 接下來，前往 ACR 組建教學課程，以深入了解如何在雲端中建置容器映像，包括基底映像更新上的自動化組建：

> [!div class="nextstepaction"]
> [使用 ACR 組建在雲端中建置容器映像](container-registry-tutorial-quick-build.md)

<!-- IMAGES -->
[sample-app-01]: ./media/container-registry-event-grid-quickstart/sample-app-01.png
[sample-app-02]: ./media/container-registry-event-grid-quickstart/sample-app-02-no-events.png
[sample-app-03]: ./media/container-registry-event-grid-quickstart/sample-app-03-with-events.png

<!-- LINKS - External -->
[azure-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[sample-app]: https://github.com/dbarkol/azure-event-grid-viewer

<!-- LINKS - Internal -->
[az-acr-create]: /cli/azure/acr/repository#az-acr-create
[az-acr-repository-delete]: /cli/azure/acr/repository#az-acr-repository-delete
[az-eventgrid-event-subscription-create]: /cli/azure/eventgrid/event-subscription#az-eventgrid-event-subscription-create
[az-group-create]: /cli/azure/group#az-group-create
