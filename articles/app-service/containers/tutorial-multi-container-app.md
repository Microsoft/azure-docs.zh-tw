---
title: 在適用於容器的 Web 應用程式中建立多容器 (預覽) 應用程式
description: 了解如何透過 Docker Compose 和 Kubernetes 組態檔以及 WordPress 與 MySQL 應用程式在 Azure 上使用多個容器。
keywords: azure 應用程式服務, web 應用程式, linux, docker, compose, 多容器, 多重容器, 適用於容器的 web 應用程式, 多個容器, 容器, kubernetes, wordpress, 適用於 mysql 的 azure db, 具有容器的生產資料庫
services: app-service
documentationcenter: ''
author: msangapu
manager: jeconnoc
editor: ''
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 06/25/2018
ms.author: msangapu
ms.custom: mvc
ms.openlocfilehash: c8002b10e2c7c0823cd59bf300283d04fca5ce2e
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49390205"
---
# <a name="tutorial-create-a-multi-container-preview-app-in-web-app-for-containers"></a>教學課程：在適用於容器的 Web 應用程式中建立多容器 (預覽) 應用程式

[適用於容器的 Web 應用程式](app-service-linux-intro.md)提供彈性的 Docker 映像使用方式。 在本教學課程中，您將了解如何使用 WordPress 和 MySQL 建立多容器應用程式。 您會在 Cloud Shell 中完成本教學課程，但您也可以在本機使用 [Azure CLI](/cli/azure/install-azure-cli) 命令列工具 (2.0.32 或更新版本) 來執行這些命令。

在本教學課程中，您將了解如何：
> [!div class="checklist"]
> * 將 Docker Compose 組態轉換為使用適用於容器的 Web 應用程式
> * 將 Kubernetes 組態轉換為使用適用於容器的 Web 應用程式
> * 將多容器應用程式部署至 Azure
> * 新增應用程式設定
> * 對您的容器使用永續性儲存體
> * 連線到適用於 MySQL 的 Azure 資料庫
> * 針對錯誤進行疑難排解

[!INCLUDE [Free trial note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要有 [Docker Compose](https://docs.docker.com/compose/) 或 [Kubernetes](https://kubernetes.io/) 的使用經驗。

## <a name="download-the-sample"></a>下載範例

在本教學課程中，您會使用 [Docker](https://docs.docker.com/compose/wordpress/#define-the-project) 中的 Compose 檔案，但您會加以修改，使其包含適用於 MySQL 的 Azure 資料庫、永續性儲存體和 Redis。 組態檔位於 [Azure 範例](https://github.com/Azure-Samples/multicontainerwordpress)中。

[!code-yml[Main](../../../azure-app-service-multi-container/docker-compose-wordpress.yml)]

在 Cloud Shell 中，建立教學課程目錄並變更為此目錄。

```bash
mkdir tutorial

cd tutorial
```

下一步，執行下列命令，將範例應用程式存放庫複製到您的教學課程目錄。 然後，變更為 `multicontainerwordpress` 目錄。

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

<!-- [!INCLUDE [app-service-plan](app-service-plan-linux.md)] -->

下列範例會在**標準**定價層 (`--sku S1`) 和 Linux 容器 (`--is-linux`) 中，建立名為 `myAppServicePlan` 的 App Service 方案。

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux
```

建立 App Service 方案後，Cloud Shell 會顯示類似下列範例的資訊：

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

## <a name="docker-compose-configuration-options"></a>Docker Compose 組態選項

在本教學課程中，您會使用 [Docker](https://docs.docker.com/compose/wordpress/#define-the-project) 中的 Compose 檔案，但您會加以修改，使其包含適用於 MySQL 的 Azure 資料庫、永續性儲存體和 Redis。 或者，您也可以使用 [Kubernetes 組態](#use-a-kubernetes-configuration-optional)。 組態檔位於 [Azure 範例](https://github.com/Azure-Samples/multicontainerwordpress)中。

下列清單顯示適用於容器的 Web 應用程式中支援和不支援的 Docker Compose 組態選項：

### <a name="supported-options"></a>支援的選項

* 命令
* entrypoint
* Environment
* 映像
* 連接埠
* restart
* 服務
* 磁碟區

### <a name="unsupported-options"></a>不支援的選項

* build (不允許)
* depends_on (已忽略)
* networks (已忽略)
* secrets (已忽略)

> [!NOTE]
> 公開預覽中也會忽略任何未明確標註的其他選項。

### <a name="docker-compose-with-wordpress-and-mysql-containers"></a>用於 WordPress 和 MySQL 容器的 Docker Compose

## <a name="create-a-docker-compose-app"></a>建立 Docker Compose 應用程式

在 Cloud Shell 中，使用 [az webapp create](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) 命令，在 `myAppServicePlan` App Service 方案中建立多容器 [Web 應用程式](app-service-linux-intro.md)。 別忘了將 _\<app_name>_ 取代為唯一的應用程式名稱。

```bash
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

建立 Web 應用程式後，Cloud Shell 會顯示類似下列範例的輸出：

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

瀏覽至已部署的應用程式 (位於 `http://<app_name>.azurewebsites.net`)。 此應用程式可能需要數分鐘才能載入。 如果發生錯誤，請再等待數分鐘，然後重新整理瀏覽器。 如果您在到問題而想要進行疑難排解，請檢閱[容器記錄](#find-docker-container-logs)。

![適用於容器的 Web 應用程式上的範例多容器應用程式][1]

**恭喜**，您已在適用於容器的 Web 應用程式中建立多容器應用程式。 接下來，您必須設定應用程式以使用適用於 MySQL 的 Azure 資料庫。 請勿在此時安裝 WordPress。

## <a name="connect-to-production-database"></a>連線至生產資料庫

建議您不要在生產環境中使用資料庫容器。 本機容器無法進行調整。 您將使用可調整的「適用於 MySQL 的 Azure 資料庫」。

### <a name="create-an-azure-database-for-mysql-server"></a>建立適用於 MySQL 的 Azure 資料庫伺服器

使用 [`az mysql server create`](/cli/azure/mysql/server?view=azure-cli-latest#az-mysql-server-create) 命令建立適用於 MySQL 伺服器的 Azure 資料庫。

在下列命令中，在您看見 _&lt;mysql_server_name>_ 預留位置的地方，取代成您自己的 MySQL 伺服器名稱 (有效字元有 `a-z`、`0-9`、`-`)。 這個名稱是 MySQL 伺服器主機名稱 (`<mysql_server_name>.database.windows.net`) 的一部分，必須是全域唯一的。

```azurecli-interactive
az mysql server create --resource-group myResourceGroup --name <mysql_server_name>  --location "South Central US" --admin-user adminuser --admin-password My5up3rStr0ngPaSw0rd! --sku-name B_Gen4_1 --version 5.7
```

建立伺服器需要幾分鐘的時間才能完成。 建立 MySQL 伺服器後，Cloud Shell 會顯示類似下列範例的資訊：

```json
{
  "administratorLogin": "adminuser",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "<mysql_server_name>.database.windows.net",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/<mysql_server_name>",
  "location": "southcentralus",
  "name": "<mysql_server_name>",
  "resourceGroup": "myResourceGroup",
  ...
}
```

### <a name="configure-server-firewall"></a>設定伺服器防火牆

使用 [`az mysql server firewall-rule create`](/cli/azure/mysql/server/firewall-rule?view=azure-cli-latest#az-mysql-server-firewall-rule-create) 命令，建立 MySQL 伺服器的防火牆規則來允許用戶端連線。 當起始 IP 和結束 IP 都設為 0.0.0.0 時，防火牆只會為其他 Azure 資源開啟。

```azurecli-interactive
az mysql server firewall-rule create --name allAzureIPs --server <mysql_server_name> --resource-group myResourceGroup --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!TIP]
> [僅使用您的應用程式所用的輸出 IP 位址](../app-service-ip-addresses.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json#find-outbound-ips)，讓您的防火牆規則更具限制性。
>

### <a name="create-the-wordpress-database"></a>建立 WordPress 資料庫

```bash
az mysql db create --resource-group myResourceGroup --server-name <mysql_server_name> --name wordpress
```

資料庫建立後，Cloud Shell 會顯示類似於下列範例的資訊：

```json
{
  "additionalProperties": {},
  "charset": "latin1",
  "collation": "latin1_swedish_ci",
  "id": "/subscriptions/12db1644-4b12-4cab-ba54-8ba2f2822c1f/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/<mysql_server_name>/databases/wordpress",
  "name": "wordpress",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.DBforMySQL/servers/databases"
}
```

### <a name="configure-database-variables-in-wordpress"></a>設定 WordPress 中的資料庫變數

若要將 WordPress 應用程式連線至這個新的 MySQL 伺服器，您必須設定幾個 WordPress 專屬環境變數，包括 `MYSQL_SSL_CA` 所定義的 SSL CA 路徑。 下方的[自訂映像](https://docs.microsoft.com/azure/app-service/containers/tutorial-multi-container-app#use-a-custom-image-for-mysql-ssl-and-other-configurations)中提供來自 [DigiCert](http://www.digicert.com/) 的 [Baltimore CyberTrust Root](https://www.digicert.com/digicert-root-certificates.htm)。

若要進行這些變更，請在 Cloud Shell 中使用 [az webapp config appsettings set](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令。 應用程式設定為區分大小寫和空格分隔。

```bash
az webapp config appsettings set --resource-group myResourceGroup --name <app_name> --settings WORDPRESS_DB_HOST="<mysql_server_name>.mysql.database.azure.com" WORDPRESS_DB_USER="adminuser@<mysql_server_name>" WORDPRESS_DB_PASSWORD="My5up3rStr0ngPaSw0rd!" WORDPRESS_DB_NAME="wordpress" MYSQL_SSL_CA="BaltimoreCyberTrustroot.crt.pem"
```

應用程式設定建立後，Cloud Shell 會顯示類似於下列範例的資訊：

```json
[
  {
    "name": "WORDPRESS_DB_HOST",
    "slotSetting": false,
    "value": "<mysql_server_name>.mysql.database.azure.com"
  },
  {
    "name": "WORDPRESS_DB_USER",
    "slotSetting": false,
    "value": "adminuser@<mysql_server_name>"
  },
  {
    "name": "WORDPRESS_DB_NAME",
    "slotSetting": false,
    "value": "wordpress"
  },
  {
    "name": "WORDPRESS_DB_PASSWORD",
    "slotSetting": false,
    "value": "My5up3rStr0ngPaSw0rd!"
  },
  {
    "name": "MYSQL_SSL_CA",
    "slotSetting": false,
    "value": "BaltimoreCyberTrustroot.crt.pem"
  }
]
```

### <a name="use-a-custom-image-for-mysql-ssl-and-other-configurations"></a>使用 MySQL SSL 和其他組態的自訂映像

根據預設，適用於 MySQL 的 Azure 資料庫會使用 SSL。 WordPress 需要額外設定才能將 SSL 用於 MySQL。 WordPress 的「官方映像」並未提供額外設定，但有[自訂映像](https://hub.docker.com/r/microsoft/multicontainerwordpress/builds/)可供您使用。 在實務上，您會將所需的變更新增至您自己的映像。

自訂映像以 [Docker Hub 中的 WordPress](https://hub.docker.com/_/wordpress/) 的「官方映像」為基礎。 「適用於 MySQL 的 Azure 資料庫」的這個自訂映像中已做了下列變更：

* [將用於 SSL 的 Baltimore Cyber Trust Root 憑證檔案新增至 MySQL。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L61)
* [對 WordPress wp-config.php 中的 MySQL SSL 憑證授權單位憑證使用應用程式設定。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L163)
* [為 MySQL SSL 所需的 MYSQL_CLIENT_FLAGS 新增 WordPress 定義。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L164)

已為 Redis 做了下列變更 (用於稍後的章節)：

* [新增 Redis v4.0.2 的 PHP 擴充功能。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/Dockerfile#L35)
* [新增檔案解壓縮所需的解壓縮功能。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L71)
* [新增 Redis 物件快取 1.3.8 WordPress 外掛程式。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L74)
* [對 WordPress wp-config.php 中的 Redis 主機名稱使用應用程式設定。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L162)

若要使用自訂映像，您必須更新 docker-compose-wordpress.yml 檔案。 在 Cloud Shell 中，輸入 `nano docker-compose-wordpress.yml`，以開啟 nano 文字編輯器。 將 `image: wordpress` 變更為使用 `image: microsoft/multicontainerwordpress`。 您不再需要資料庫容器。 請從組態檔中移除 `db`、`environment`、`depends_on` 和 `volumes` 區段。 您的檔案看起來應該會類似於下列程式碼：

```yaml
version: '3.3'

services:
   wordpress:
     image: microsoft/multicontainerwordpress
     ports:
       - "8000:80"
     restart: always
```

儲存您的變更並結束 nano。 使用 `^O` 命令進行儲存，以及使用 `^X` 來結束作業。

### <a name="update-app-with-new-configuration"></a>使用新組態更新應用程式

在 Cloud Shell 中，使用 [az webapp config container set](/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set) 命令重新設定多容器 [Web 應用程式](app-service-linux-intro.md)。 別忘了將 _\<app_name>_ 取代為您先前建立的 Web 應用程式名稱。

```bash
az webapp config container set --resource-group myResourceGroup --name <app_name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

應用程式重新設定後，Cloud Shell 會顯示類似下列範例的資訊：

```json
[
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICB3b3JkcHJlc3M6CiAgICAgaW1hZ2U6IG1zYW5nYXB1L3dvcmRwcmVzcwogICAgIHBvcnRzOgogICAgICAgLSAiODAwMDo4MCIKICAgICByZXN0YXJ0OiBhbHdheXM="
  }
]
```

### <a name="browse-to-the-app"></a>瀏覽至應用程式

瀏覽至已部署的應用程式 (位於 `http://<app_name>.azurewebsites.net`)。 應用程式現在會使用適用於 MySQL 的 Azure 資料庫。

![用於容器的 Web App 上的範例多容器應用程式][1]

## <a name="add-persistent-storage"></a>新增永續性儲存體

您的多個容器現在會在適用於容器的 Web 應用程式中執行。 不過，如果您此時安裝 WordPress，並於稍後重新啟動應用程式，您會發現 WordPress 安裝已消失。 之所以發生此狀況，是因為您的 Docker Compose 組態目前指向容器內的儲存位置。 安裝到容器中的檔案在應用程式重新啟動後將不會保存。 在本節中，您會將永續性儲存體新增至 WordPress 容器。

### <a name="configure-environment-variables"></a>設定環境變數

若要使用永續性儲存體，您必須在 App Service 內啟用此設定。 若要進行這項變更，請在 Cloud Shell 中使用 [az webapp config appsettings set](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令。 應用程式設定為區分大小寫和空格分隔。

```bash
az webapp config appsettings set --resource-group myResourceGroup --name <app_name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=TRUE
```

應用程式設定建立後，Cloud Shell 會顯示類似於下列範例的資訊：

```json
[
  < JSON data removed for brevity. >
  {
    "name": "WORDPRESS_DB_NAME",
    "slotSetting": false,
    "value": "wordpress"
  },
  {
    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
    "slotSetting": false,
    "value": "TRUE"
  }
]
```

### <a name="modify-configuration-file"></a>修改組態檔

在 Cloud Shell 中，輸入 `nano docker-compose-wordpress.yml`，以開啟 nano 文字編輯器。

`volumes` 選項會將檔案系統對應至容器內的目錄。 `${WEBAPP_STORAGE_HOME}` 是 App Service 中與您應用程式的永續性儲存體相對應的環境變數。 您將在 volumes 選項中使用此環境變數，讓 WordPress 檔案安裝到永續性儲存體中，而不是容器中。 請對檔案進行下列修改：

在 `wordpress` 區段中新增 `volumes` 選項，讓其看起來類似於下列程式碼：

```yaml
version: '3.3'

services:
   wordpress:
     image: microsoft/multicontainerwordpress
     volumes:
      - ${WEBAPP_STORAGE_HOME}/site/wwwroot:/var/www/html
     ports:
       - "8000:80"
     restart: always
```

### <a name="update-app-with-new-configuration"></a>使用新組態更新應用程式

在 Cloud Shell 中，使用 [az webapp config container set](/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set) 命令重新設定多容器 [Web 應用程式](app-service-linux-intro.md)。 別忘了將 _\<app_name>_ 取代為唯一的應用程式名稱。

```bash
az webapp config container set --resource-group myResourceGroup --name <app_name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

在命令執行後，將會顯示類似於下列範例的輸出：

```json
[
  {
    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
    "slotSetting": false,
    "value": "TRUE"
  },
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICBteXNxbDoKICAgICBpbWFnZTogbXlzcWw6NS43CiAgICAgdm9sdW1lczoKICAgICAgIC0gZGJfZGF0YTovdmFyL2xpYi9teXNxbAogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgTVlTUUxfUk9PVF9QQVNTV09SRDogZXhhbXBsZXBhc3MKCiAgIHdvcmRwcmVzczoKICAgICBkZXBlbmRzX29uOgogICAgICAgLSBteXNxbAogICAgIGltYWdlOiB3b3JkcHJlc3M6bGF0ZXN0CiAgICAgcG9ydHM6CiAgICAgICAtICI4MDAwOjgwIgogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgV09SRFBSRVNTX0RCX1BBU1NXT1JEOiBleGFtcGxlcGFzcwp2b2x1bWVzOgogICAgZGJfZGF0YTo="
  }
]
```

### <a name="browse-to-the-app"></a>瀏覽至應用程式

瀏覽至已部署的應用程式 (位於 `http://<app_name>.azurewebsites.net`)。

WordPress 容器現在會使用適用於 MySQL 的 Azure 資料庫和永續性儲存體。

## <a name="add-redis-container"></a>新增 Redis 容器

 WordPress 的「官方映像」未包含 Redis 的相依性。 此[自訂映像](https://github.com/Azure-Samples/multicontainerwordpress)中已有要搭配使用 Redis 與 WordPress 所需的相依性和額外設定，可供您使用。 在實務上，您會將所需的變更新增至您自己的映像。

自訂映像以 [Docker Hub 中的 WordPress](https://hub.docker.com/_/wordpress/) 的「官方映像」為基礎。 Redis 的這個自訂映像中已做了下列變更：

* [新增 Redis v4.0.2 的 PHP 擴充功能。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/Dockerfile#L35)
* [新增檔案解壓縮所需的解壓縮功能。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L71)
* [新增 Redis 物件快取 1.3.8 WordPress 外掛程式。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L74)
* [對 WordPress wp-config.php 中的 Redis 主機名稱使用應用程式設定。](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L162)

將 Redis 容器新增至組態檔底部，使其看起來類似於下列範例：

[!code-yml[Main](../../../azure-app-service-multi-container/compose-wordpress.yml)]

### <a name="configure-environment-variables"></a>設定環境變數

若要使用 Redis，您必須在 App Service 內啟用此設定 `WP_REDIS_HOST`。 這是讓 WordPress 與 Redis 主機進行通訊的*必要設定*。 若要進行這項變更，請在 Cloud Shell 中使用 [az webapp config appsettings set](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令。 應用程式設定為區分大小寫和空格分隔。

```bash
az webapp config appsettings set --resource-group myResourceGroup --name <app_name> --settings WP_REDIS_HOST="redis"
```

應用程式設定建立後，Cloud Shell 會顯示類似於下列範例的資訊：

```json
[
  < JSON data removed for brevity. >
  {
    "name": "WORDPRESS_DB_USER",
    "slotSetting": false,
    "value": "adminuser@<mysql_server_name>"
  },
  {
    "name": "WP_REDIS_HOST",
    "slotSetting": false,
    "value": "redis"
  }
]
```

### <a name="update-app-with-new-configuration"></a>使用新組態更新應用程式

在 Cloud Shell 中，使用 [az webapp config container set](/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set) 命令重新設定多容器 [Web 應用程式](app-service-linux-intro.md)。 別忘了將 _\<app_name>_ 取代為唯一的應用程式名稱。

```bash
az webapp config container set --resource-group myResourceGroup --name <app_name> --multicontainer-config-type compose --multicontainer-config-file compose-wordpress.yml
```

在命令執行後，將會顯示類似於下列範例的輸出：

```json
[
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICBteXNxbDoKICAgICBpbWFnZTogbXlzcWw6NS43CiAgICAgdm9sdW1lczoKICAgICAgIC0gZGJfZGF0YTovdmFyL2xpYi9teXNxbAogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgTVlTUUxfUk9PVF9QQVNTV09SRDogZXhhbXBsZXBhc3MKCiAgIHdvcmRwcmVzczoKICAgICBkZXBlbmRzX29uOgogICAgICAgLSBteXNxbAogICAgIGltYWdlOiB3b3JkcHJlc3M6bGF0ZXN0CiAgICAgcG9ydHM6CiAgICAgICAtICI4MDAwOjgwIgogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgV09SRFBSRVNTX0RCX1BBU1NXT1JEOiBleGFtcGxlcGFzcwp2b2x1bWVzOgogICAgZGJfZGF0YTo="
  }
]
```

### <a name="browse-to-the-app"></a>瀏覽至應用程式

瀏覽至已部署的應用程式 (位於 `http://<app_name>.azurewebsites.net`)。

完成相關步驟並安裝 WordPress。

### <a name="connect-wordpress-to-redis"></a>將 WordPress 連線至 Redis

登入 WordPress 管理。在左側導覽中選取 [外掛程式]，然後選取 [已安裝的外掛程式]。

![選取 WordPress 外掛程式][2]

在此處顯示所有的外掛程式

在 [外掛程式] 頁面中，尋找 [Redis 物件快取]，然後按一下 [啟動]。

![啟動 Redis][3]

按一下 [設定] 。

![按一下 [設定]][4]

按一下 [啟用物件快取] 按鈕。

![按一下 [啟用物件快取] 按鈕][5]

WordPress 會連線至 Redis 伺服器。 連線**狀態**會顯示在相同頁面上。

![WordPress 會連線至 Redis 伺服器。 連線 **狀態** 會顯示在相同頁面上。][6]

**恭喜**，您已將 WordPress 連線至 Redis。 已可用於生產環境的應用程式現在會使用**適用於 MySQL 的 Azure 資料庫、永續性儲存體和 Redis**。 您現在可以將 App Service 方案相應放大至多個執行個體。

## <a name="use-a-kubernetes-configuration-optional"></a>使用 Kubernetes 組態 (選用)

在本節中，您將了解如何使用 Kubernetes 組態來部署多個容器。 請確實依照先前步驟建立[資源群組](#create-a-resource-group)和 [App Service 方案](#create-an-azure-app-service-plan)。 由於大部分的步驟都與 Compose 區段的步驟類似，因此系統已為您合併組態檔。

### <a name="supported-kubernetes-options-for-multi-container"></a>多個容器支援的 Kubernetes 選項

* args
* 命令
* containers
* 映像
* name
* 連接埠
* 規格

> [!NOTE]
>公開預覽中不支援任何未明確標註的其他 Kubernetes 選項。
>

### <a name="kubernetes-configuration-file"></a>Kubernetes 設定檔

您會在教學課程的這個部分使用 kubernetes-wordpress.yml。 我們將其顯示於此供您參考：

[!code-yml[Main](../../../azure-app-service-multi-container/kubernetes-wordpress.yml)]

### <a name="create-an-azure-database-for-mysql-server"></a>建立適用於 MySQL 的 Azure 資料庫伺服器

使用 [`az mysql server create`](/cli/azure/mysql/server?view=azure-cli-latest#az-mysql-server-create) 命令，在適用於 MySQL 的 Azure 資料庫中建立伺服器。

在下列命令中，在您看見 _&lt;mysql_server_name>_ 預留位置的地方，取代成您自己的 MySQL 伺服器名稱 (有效字元有 `a-z`、`0-9`、`-`)。 這個名稱是 MySQL 伺服器主機名稱 (`<mysql_server_name>.database.windows.net`) 的一部分，必須是全域唯一的。

```azurecli-interactive
az mysql server create --resource-group myResourceGroup --name <mysql_server_name>  --location "South Central US" --admin-user adminuser --admin-password My5up3rStr0ngPaSw0rd! --sku-name B_Gen4_1 --version 5.7
```

建立 MySQL 伺服器後，Cloud Shell 會顯示類似下列範例的資訊：

```json
{
  "administratorLogin": "adminuser",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "<mysql_server_name>.database.windows.net",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/<mysql_server_name>",
  "location": "southcentralus",
  "name": "<mysql_server_name>",
  "resourceGroup": "myResourceGroup",
  ...
}
```

### <a name="configure-server-firewall"></a>設定伺服器防火牆

使用 [`az mysql server firewall-rule create`](/cli/azure/mysql/server/firewall-rule?view=azure-cli-latest#az-mysql-server-firewall-rule-create) 命令，建立 MySQL 伺服器的防火牆規則來允許用戶端連線。 當起始 IP 和結束 IP 都設為 0.0.0.0 時，防火牆只會為其他 Azure 資源開啟。

```azurecli-interactive
az mysql server firewall-rule create --name allAzureIPs --server <mysql_server_name> --resource-group myResourceGroup --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!TIP]
> [僅使用您的應用程式所用的輸出 IP 位址](../app-service-ip-addresses.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json#find-outbound-ips)，讓您的防火牆規則更具限制性。
>

### <a name="create-the-wordpress-database"></a>建立 WordPress 資料庫

如果您尚未建立[適用於 MySQL 伺服器的 Azure 資料庫](#create-an-azure-database-for-mysql-server)，請加以建立。

```bash
az mysql db create --resource-group myResourceGroup --server-name <mysql_server_name> --name wordpress
```

資料庫建立後，Cloud Shell 會顯示類似於下列範例的資訊：

```json
{
  "additionalProperties": {},
  "charset": "latin1",
  "collation": "latin1_swedish_ci",
  "id": "/subscriptions/12db1644-4b12-4cab-ba54-8ba2f2822c1f/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/<mysql_server_name>/databases/wordpress",
  "name": "wordpress",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.DBforMySQL/servers/databases"
}
```

### <a name="create-a-multi-container-app-kubernetes"></a>建立多容器應用程式 (Kubernetes)

在 Cloud Shell 中，使用 [az webapp create](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) 命令，在 `myResourceGroup` 資源群組和 `myAppServicePlan` App Service 方案中建立多容器 [Web 應用程式](app-service-linux-intro.md)。 別忘了將 _\<app_name>_ 取代為唯一的應用程式名稱。

```bash
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name> --multicontainer-config-type kube --multicontainer-config-file kubernetes-wordpress.yml
```

建立 Web 應用程式後，Cloud Shell 會顯示類似下列範例的輸出：

```json
{
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

### <a name="configure-database-variables-in-wordpress"></a>設定 WordPress 中的資料庫變數

若要將 WordPress 應用程式連線至這個新的 MySQL 伺服器，您必須設定幾個 WordPress 專屬環境變數。 若要進行這項變更，請在 Cloud Shell 中使用 [az webapp config appsettings set](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令。 應用程式設定為區分大小寫和空格分隔。

```bash
az webapp config appsettings set --resource-group myResourceGroup --name <app_name> --settings WORDPRESS_DB_HOST="<mysql_server_name>.mysql.database.azure.com" WORDPRESS_DB_USER="adminuser@<mysql_server_name>" WORDPRESS_DB_PASSWORD="My5up3rStr0ngPaSw0rd!" WORDPRESS_DB_NAME="wordpress" MYSQL_SSL_CA="BaltimoreCyberTrustroot.crt.pem"
```

應用程式設定建立後，Cloud Shell 會顯示類似於下列範例的資訊：

```json
[
  {
    "name": "WORDPRESS_DB_HOST",
    "slotSetting": false,
    "value": "<mysql_server_name>.mysql.database.azure.com"
  },
  {
    "name": "WORDPRESS_DB_USER",
    "slotSetting": false,
    "value": "adminuser@<mysql_server_name>"
  },
  {
    "name": "WORDPRESS_DB_NAME",
    "slotSetting": false,
    "value": "wordpress"
  },
  {
    "name": "WORDPRESS_DB_PASSWORD",
    "slotSetting": false,
    "value": "My5up3rStr0ngPaSw0rd!"
  }
]
```

### <a name="add-persistent-storage"></a>新增永續性儲存體

您的多個容器現在會在適用於容器的 Web 應用程式中執行。 資料將在重新啟動後清除，因為檔案不會持續保存。 在本節中，您會將永續性儲存體新增至 WordPress 容器。

### <a name="configure-environment-variables"></a>設定環境變數

若要使用永續性儲存體，您必須在 App Service 內啟用此設定。 若要進行這項變更，請在 Cloud Shell 中使用 [az webapp config appsettings set](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) 命令。 應用程式設定為區分大小寫和空格分隔。

```bash
az webapp config appsettings set --resource-group myResourceGroup --name <app_name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=TRUE
```

應用程式設定建立後，Cloud Shell 會顯示類似於下列範例的資訊：

```json
[
  {
    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
    "slotSetting": false,
    "value": "TRUE"
  }
]
```

### <a name="browse-to-the-app"></a>瀏覽至應用程式

瀏覽至已部署的應用程式 (位於 `http://<app_name>.azurewebsites.net`)。

應用程式現在會在適用於容器的 Web 應用程式中執行多個容器。

![用於容器的 Web App 上的範例多容器應用程式][1]

**恭喜**，您已在適用於容器的 Web 應用程式中建立多容器應用程式。

若要使用 Redis，請遵循[將 WordPress 連線至 Redis](#connect-wordpress-to-redis) 中的步驟。

## <a name="find-docker-container-logs"></a>尋找 Docker 容器記錄

如果您在使用多個容器時發生問題，您可以瀏覽至下列位置存取容器記錄：`https://<app_name>.scm.azurewebsites.net/api/logs/docker`。

您會看到類似於下列範例的輸出：

```json
[
   {
      "machineName":"RD00XYZYZE567A",
      "lastUpdated":"2018-05-10T04:11:45Z",
      "size":25125,
      "href":"https://<app_name>.scm.azurewebsites.net/api/vfs/LogFiles/2018_05_10_RD00XYZYZE567A_docker.log",
      "path":"/home/LogFiles/2018_05_10_RD00XYZYZE567A_docker.log"
   }
]
```

您會看到每個容器的記錄，以及父處理序的記錄。 請將各自的 `href` 值複製到瀏覽器中，以檢視記錄。

[!INCLUDE [Clean-up section](../../../includes/cli-script-clean-up.md)]

在本教學課程中，您已了解如何：
> [!div class="checklist"]
> * 將 Docker Compose 組態轉換為使用適用於容器的 Web 應用程式
> * 將 Kubernetes 組態轉換為使用適用於容器的 Web 應用程式
> * 將多容器應用程式部署至 Azure
> * 新增應用程式設定
> * 對您的容器使用永續性儲存體
> * 連線到適用於 MySQL 的 Azure 資料庫
> * 針對錯誤進行疑難排解

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [對適用於容器的 Web 應用程式使用自訂 Docker 映像](tutorial-custom-docker-image.md)

<!--Image references-->
[1]: ./media/tutorial-multi-container-app/azure-multi-container-wordpress-install.png
[2]: ./media/tutorial-multi-container-app/wordpress-plugins.png
[3]: ./media/tutorial-multi-container-app/activate-redis.png
[4]: ./media/tutorial-multi-container-app/redis-settings.png
[5]: ./media/tutorial-multi-container-app/enable-object-cache.png
[6]: ./media/tutorial-multi-container-app/redis-connected.png
