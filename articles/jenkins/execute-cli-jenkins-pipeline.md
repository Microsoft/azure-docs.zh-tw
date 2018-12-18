---
title: 使用 Jenkins 執行 Azure CLI
description: 了解如何在 Jenkins 管線中使用 Azure CLI 將 Java Web 應用程式部署到 Azure
ms.service: jenkins
keywords: jenkins, azure, devops, app service, cli
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 6/7/2017
ms.openlocfilehash: b9ca8848da543bbfb27246109c3a4ab97eb6bc58
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46974913"
---
# <a name="deploy-to-azure-app-service-with-jenkins-and-the-azure-cli"></a>使用 Jenkins 和 Azure CLI 來部署到 Azure App Service
若要將 Java Web 應用程式部署到 Azure，您可以在 [Jenkins 管線](https://jenkins.io/doc/book/pipeline/)中使用 Azure CLI。 在本教學課程中，您會在 Azure VM 上建立 CI/CD 管線，包括如何︰

> [!div class="checklist"]
> * 建立 Jenkins VM
> * 設定 Jenkins
> * 在 Azure 中建立 Web 應用程式
> * 準備 GitHub 存放庫
> * 建立 Jenkins 管線
> * 執行管線並確認 Web 應用程式

本教學課程需要 Azure CLI 2.0.4 版或更新版本。 若要尋找版本，請執行 `az --version`。 如果您需要升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-and-configure-jenkins-instance"></a>建立及設定 Jenkins 執行個體
如果您還沒有 Jenkins Master，請從[解決方案範本](install-jenkins-solution-template.md)開始著手，此範本包含預設必要的 [Azure 認證](https://plugins.jenkins.io/azure-credentials)外掛程式。 

「Azure 認證」外掛程式可讓您將 Microsoft Azure 服務主體認證儲存在 Jenkins 中。 在 1.2 版中，我們已新增支援，因此 Jenkins 管線可以取得 Azure 認證。 

請確定您擁有的版本是 1.2 或更新版本：
* 在 Jenkins 儀表板中，按一下 [Manage Jenkins] \(管理 Jenkins\) -> [Plugin Manager] \(外掛程式管理員\)，然後搜尋 [Azure Credential] \(Azure 認證\)。 
* 如果版本比 1.2 版舊，請更新外掛程式。

Jenkins Master 中也必須有 Java JDK 和 Maven。 若要安裝，請使用 SSH 登入 Jenkins Master，然後執行下列命令：
```bash
sudo apt-get install -y openjdk-7-jdk
sudo apt-get install -y maven
```

## <a name="add-azure-service-principal-to-jenkins-credential"></a>將 Azure 服務主體新增到 Jenkins 認證

必須要有 Azure 認證，才能執行 Azure CLI。

* 在 Jenkins 儀表板中，按一下 [Credentials] \(認證\) -> [System] \(系統\) -> 。 按一下 [Global credentials(unrestricted)] \(全域認證 (不受限)\)。
* 按一下 [Add Credentials] \(新增認證\) 來填寫 [Subscription ID] \(訂用帳戶 ID\)、[Client ID] \(用戶端識別碼\)、[Client Secret] \(用戶端祕密\) 及 [OAuth 2.0 Token Endpoint] \(OAuth 2.0 權杖端點\)，以新增 [Microsoft Azure 服務主體](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json)。 請提供一個要在後續步驟中使用的識別碼。

![新增認證](./media/execute-cli-jenkins-pipeline/add-credentials.png)

## <a name="create-an-azure-app-service-for-deploying-the-java-web-app"></a>建立 Azure App Service 來部署 Java Web 應用程式

使用 [az appservice plan create](/cli/azure/appservice/plan#az-appservice-plan-create) CLI 命令，建立搭配**免費**定價層的 Azure App Service 方案。 Appservice 方案會定義用來託管應用程式的實體資源。 所有指派給 Appservice 方案的應用程式都會共用這些資源，從而讓您節省託管多個應用程式的成本。 

```azurecli-interactive
az appservice plan create \
    --name myAppServicePlan \ 
    --resource-group myResourceGroup \
    --sku FREE
```

方案準備妥當時，Azure CLI 會顯示類似下列範例的輸出：

```json
{ 
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "North Europe",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "app",
  "location": "North Europe",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  ...
  < Output has been truncated for readability >
} 
``` 

### <a name="create-an-azure-web-app"></a>建立 Azure Web 應用程式

 使用 [az webapp create](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) CLI 命令，在 `myAppServicePlan` App Service 方案中建立 Web 應用程式定義。 Web 應用程式定義會提供一個 URL 以存取您的應用程式，並設定數個選項將您的程式碼部署至 Azure。 

```azurecli-interactive
az webapp create \
    --name <app_name> \ 
    --resource-group myResourceGroup \
    --plan myAppServicePlan
```

使用您自己的唯一應用程式名稱來取代 `<app_name>` 預留位置。 這個唯一名稱會是 Web 應用程式預設網域名稱的一部分，因此，這個名稱在 Azure 的所有應用程式中必須是唯一的。 您可以先將自訂的網域名稱項目對應至 Web 應用程式，再將它公開給使用者。

Web 應用程式定義備妥之後，Azure CLI 會顯示類似下列範例的資訊： 

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
   ...
  < Output has been truncated for readability >
}
```

### <a name="configure-java"></a>設定 Java 

使用 [az appservice web config update](/cli/azure/webapp/config#az-appservice-web-config-update) 命令來設定您的應用程式需要的 Java 執行階段組態。

下列命令會將 Web 應用程式設定為在最新的 Java 8 JDK 和 [Apache Tomcat](http://tomcat.apache.org/) 8.0 上執行。

```azurecli-interactive
az webapp config set \ 
    --name <app_name> \
    --resource-group myResourceGroup \ 
    --java-version 1.8 \ 
    --java-container Tomcat \
    --java-container-version 8.0
```

## <a name="prepare-a-github-repository"></a>準備 GitHub 存放庫
開啟[適用於 Azure 的簡單 Java Web 應用程式](https://github.com/azure-devops/javawebappsample)存放庫。 為了將存放庫分支至您自己的 GitHub 帳戶，按一下右上角的 [分支] 按鈕。

* 在 GitHub Web UI 中，開啟 **Jenkinsfile** 檔案。 按一下鉛筆圖示來編輯此檔案，更新您 Web 應用程式的資源群組和名稱 (分別位於第 20 行和第 21 行)。

```java
def resourceGroup = '<myResourceGroup>'
def webAppName = '<app_name>'
```

* 變更第 23 行以更新您 Jenkins 執行個體中的認證識別碼

```java
withCredentials([azureServicePrincipal('<mySrvPrincipal>')]) {
```

## <a name="create-jenkins-pipeline"></a>建立 Jenkins 管線
在網頁瀏覽器中開啟 Jenkins，按一下 [New Item] \(新增項目\)。 

* 為作業提供一個名稱，然後選取 [Pipeline] \(管線\)。 按一下 [確定]。
* 接著，按一下 [Pipeline] \(管線\) 索引標籤。 
* 針對 [Definition] \(定義\)，選取 [Pipeline script from SCM] \(來自 SCM 的管線指令碼\)。
* 針對 [SCM]，選取 [Git]。
* 輸入您分支存放庫的 GitHub URL：https:\<您的分支存放庫\>.git
* 按一下 [儲存] 

## <a name="test-your-pipeline"></a>測試您的管線
* 移至您建立的管線，按一下 [Build Now] \(立即建置\)
* 應該在幾秒內就會順利完成一個組建，您可以移至該組建，然後按一下 [Console Output] \(主控台輸出\) 來查看詳細資料

## <a name="verify-your-web-app"></a>確認您的 Web 應用程式
若要確認 WAR 檔案是否已順利部署到您的 Web 應用程式， 請開啟網頁瀏覽器：

* 移至 http://&lt;app_name>.azurewebsites.net/api/calculator/ping  
您會看見：

        Welcome to Java Web App!!! This is updated!
        Sun Jun 17 16:39:10 UTC 2017

* 移至 http://&lt;app_name>.azurewebsites.net/api/calculator/add?x=&lt;x>&y=&lt;y> (以任何數字取代 &lt;x> 和 &lt;y>) 以取得 x 和 y 的總和

![計算機：加法](./media/execute-cli-jenkins-pipeline/calculator-add.png)

## <a name="deploy-to-azure-web-app-on-linux"></a>部署到 Linux 上的 Azure Web 應用程式
既然您已知道如何在 Jenkins 管線中使用 Azure CLI，現在就可以修改指令碼來部署到 Linux 上的 Azure Web 應用程式。

Linux 上的 Web 應用程式支援不同的方式來進行部署，也就是使用 Docker。 若要部署，您必須提供一個 Dockerfile，此檔案會將您的 Web 應用程式與服務執行階段封裝成 Docker 映像。 外掛程式會接著建置該映像，將它推送到 Docker 登錄，然後將該映像部署到您的 Web 應用程式。

* 依照[這裡](../app-service/containers/quickstart-nodejs.md)的步驟來建立在 Linux 上執行的 Azure Web 應用程式。
* 依照這篇[文章](https://docs.docker.com/engine/installation/linux/ubuntu/)中的指示，將 Docker 安裝在您的 Jenkins 執行個體上。
* 使用[這裡](/azure/container-registry/container-registry-get-started-azure-cli)的步驟在 Azure 入口網站中建立「容器登錄」。
* 在您所分支的相同[適用於 Azure 的簡單 Java Web 應用程式](https://github.com/azure-devops/javawebappsample)存放庫中，編輯 **Jenkinsfile2** 檔案：
    * 第 18 到 21 行：分別更新您的資源群組、Web 應用程式及 ACR。 
        ```
        def webAppResourceGroup = '<myResourceGroup>'
        def webAppName = '<app_name>'
        def acrName = '<myRegistry>'
        ```

    * 第 24 行：將 \<azsrvprincipal\> 更新成您的認證識別碼
        ```
        withCredentials([azureServicePrincipal('<mySrvPrincipal>')]) {
        ```

* 建立新的 Jenkins 管線，就像您在部署到 Windows 中的 Azure Web 應用程式時一樣，只是這次改用 **Jenkinsfile2**。
* 執行您的新作業。
* 若要確認，請在 Azure CLI 中，執行：

    ```
    az acr repository list -n <myRegistry> -o json
    ```

    您會收到下列結果︰
    
    ```
    [
    "calculator"
    ]
    ```
    
    移至 http://&lt;app_name>.azurewebsites.net/api/calculator/ping。 您會看見此訊息： 
    
        Welcome to Java Web App!!! This is updated!
        Sun Jul 09 16:39:10 UTC 2017

    移至 http://&lt;app_name>.azurewebsites.net/api/calculator/add?x=&lt;x>&y=&lt;y> (以任何數字取代 &lt;x> 和 &lt;y>) 以取得 x 和 y 的總和
    
## <a name="next-steps"></a>後續步驟
在本教學課程中，您已設定一個 Jenkins 管線，此管線會簽出 GitHub 存放庫中的原始程式碼。 請執行 Maven 來建置一個 WAR 檔案，然後使用 Azure CLI 來部署到 Azure App Service。 您已了解如何︰

> [!div class="checklist"]
> * 建立 Jenkins VM
> * 設定 Jenkins
> * 在 Azure 中建立 Web 應用程式
> * 準備 GitHub 存放庫
> * 建立 Jenkins 管線
> * 執行管線並確認 Web 應用程式
