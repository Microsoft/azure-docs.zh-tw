---
title: 使用 Docker Compose 組態在適用於容器的 Azure Web 應用程式中建立多容器 (預覽) 應用程式
description: 利用幾分鐘的時間在適用於容器的 Azure Web 應用程式中部署第一個多容器應用程式
keywords: azure 應用程式服務, web 應用程式, linux, docker, compose, 多容器, 多重容器, 適用於容器的 web 應用程式, 多個容器, 容器, kubernetes, wordpress, 適用於 mysql 的 azure db, 具有容器的生產資料庫
services: app-service\web
documentationcenter: ''
author: msangapu
manager: jeconnoc
editor: ''
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 06/22/2018
ms.author: msangapu
ms.custom: mvc
ms.openlocfilehash: 1d54d914edb306b89dc6f2604aa5af3d17cb535d
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45576777"
---
# <a name="create-a-multi-container-preview-app-using-web-app-for-containers"></a>透過用於容器的 Web App 建立多容器 (預覽) 應用程式

[適用於容器的 Web 應用程式](app-service-linux-intro.md)提供彈性的 Docker 映像使用方式。 本快速入門說明如何使用 Docker Compose 組態，在 [Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview) 中將多容器應用程式部署至用於容器的 Web App。 至於 Kubernetes 和使用「適用於 MySQL 的 Azure DB」的完整端對端解決方案，請遵循[多容器教學課程](tutorial-multi-container-app.md)。

您將會在 Cloud Shell 中完成本快速入門，但您也可以在本機使用 [Azure CLI](/cli/azure/install-azure-cli) (2.0.32 或更新版本) 來執行這些命令。 

![適用於容器的 Web 應用程式上的範例多容器應用程式][1]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="download-the-sample"></a>下載範例

在本快速入門中，您會使用從 [Docker](https://docs.docker.com/compose/wordpress/#define-the-project) 取得的 Compose 檔案。 組態檔位於 [Azure 範例](https://github.com/Azure-Samples/multicontainerwordpress)中。

[!code-yml[Main](../../../azure-app-service-multi-container/docker-compose-wordpress.yml)]

在 Cloud Shell 中，建立快速入門目錄並變更為此目錄。

```bash
mkdir quickstart

cd quickstart
```

下一步，執行下列命令，將範例應用程式存放庫複製到您的快速入門目錄。 然後，變更為 `multicontainerwordpress` 目錄。

```bash
git clone https://github.com/Azure-Samples/multicontainerwordpress

cd multicontainerwordpress
```

## <a name="create-a-resource-group"></a>建立資源群組

[!INCLUDE [resource group intro text](../../../includes/resource-group.md)]

在 Cloud Shell 中，使用 [`az group create`](/cli/azure/group?view=azure-cli-latest#az-group-create) 命令來建立資源群組。 下列範例會在*美國中南部*位置中建立名為 *myResourceGroup* 的資源群組。 若要查看**標準**層中 Linux 上之 App Service 的所有支援位置，請執行 [`az appservice list-locations --sku S1 --linux-workers-enabled`](/cli/azure/appservice?view=azure-cli-latest#az-appservice-list-locations) 命令。

```azurecli-interactive
az group create --name myResourceGroup --location "South Central US"
```

您通常會在附近的區域中建立資源群組和資源。

當命令完成時，JSON 輸出會顯示資源群組屬性。

## <a name="create-an-azure-app-service-plan"></a>建立 Azure App Service 方案

在 Cloud Shell 中，使用 [`az appservice plan create`](/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create) 命令在資源群組中建立 App Service 方案。

下列範例會在**標準**定價層 (`--sku S1`) 和 Linux 容器 (`--is-linux`) 中，建立名為 `myAppServicePlan` 的 App Service 方案。

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux
```

建立 App Service 方案後，Azure CLI 會顯示類似下列範例的資訊：

```json
{
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "South Central US",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "linux",
  "location": "South Central US",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  < JSON data removed for brevity. >
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
}
```

## <a name="create-a-docker-compose-app"></a>建立 Docker Compose 應用程式

在 Cloud Shell 終端機中，使用 [az webapp create](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) 命令，在 `myAppServicePlan` App Service 方案中建立多容器 [Web 應用程式](app-service-linux-intro.md)。 別忘了將 _\<app_name>_ 取代為唯一的應用程式名稱。

```bash
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name> --multicontainer-config-type compose --multicontainer-config-file compose-wordpress.yml
```

建立 Web 應用程式後，Azure CLI 會顯示類似下列範例的輸出：

```json
{
  "additionalProperties": {},
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app_name>.azurewebsites.net",
  "enabled": true,
  < JSON data removed for brevity. >
}
```

### <a name="browse-to-the-app"></a>瀏覽至應用程式

瀏覽至已部署的應用程式 (位於 `http://<app_name>.azurewebsites.net`)。 此應用程式可能需要數分鐘才能載入。 如果發生錯誤，請再等待數分鐘，然後重新整理瀏覽器。

![適用於容器的 Web 應用程式上的範例多容器應用程式][1]

**恭喜**，您已在適用於容器的 Web 應用程式中建立多容器應用程式。

[!INCLUDE [Clean-up section](../../../includes/cli-script-clean-up.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [在用於容器的 Web App 中建立多容器 WordPress 應用程式](tutorial-multi-container-app.md)

<!--Image references-->
[1]: ./media/tutorial-multi-container-app/azure-multi-container-wordpress-install.png