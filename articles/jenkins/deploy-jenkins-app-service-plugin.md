---
title: 使用 Jenkins 外掛程式來部署到 Azure App Service
description: 了解如何在 Jenkins 中使用 Azure App Service Jenkins 外掛程式將 Java Web 應用程式部署到 Azure
ms.service: jenkins
keywords: jenkins, azure, devops, app service
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 07/31/2018
ms.openlocfilehash: 5f76d18662105df6d278e09e047baa13773ab4ac
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49319348"
---
# <a name="deploy-to-azure-app-service-by-using-the-jenkins-plugin"></a>使用 Jenkins 外掛程式來部署到 Azure App Service 

若要將 Java Web 應用程式部署到 Azure，您可以在 [Jenkins 管線](/azure/jenkins/execute-cli-jenkins-pipeline)中使用 Azure CLI，或者，也可以使用 [Azure App Service Jenkins 外掛程式](https://plugins.jenkins.io/azure-app-service)。 Jenkins 外掛程式 1.0 版支援透過下列方式，使用 Azure App Service 的 Web Apps 功能來進行連續部署：
* 檔案上傳。
* 適用於 Linux 上 Web Apps 的 Docker。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 設定 Jenkins 以透過檔案上傳來部署 Web 應用程式。
> * 設定 Jenkins 以部署適用於容器的 Web Apps。

## <a name="create-and-configure-a-jenkins-instance"></a>建立及設定 Jenkins 執行個體

如果您還沒有 Jenkins Master，請從[解決方案範本](install-jenkins-solution-template.md)開始著手，此範本包含「Java 開發套件」(JDK) 第 8 版和下列必要的 Jenkins 外掛程式：

* [Jenkins Git 用戶端外掛程式](https://plugins.jenkins.io/git-client) 2.4.6 版 
* [Docker Commons 外掛程式](https://plugins.jenkins.io/docker-commons) 1.4.0 版
* [Azure 認證](https://plugins.jenkins.io/azure-credentials) 1.2 版
* [Azure App Service](https://plugins.jenkins.io/azure-app-service) 0.1 版

您可以使用 Jenkins 外掛程式來部署 Web Apps 所支援、以任何語言 (例如 C#、PHP、Java 及 Node.js) 撰寫的 Web 應用程式。 在本教學課程中，我們會使用[適用於 Azure 的簡單 Java Web 應用程式](https://github.com/azure-devops/javawebappsample)。 若要將儲存機制分支到您自己的 GitHub 帳戶，請選取 GitHub 介面右上角的 [Fork] \(分支\) 按鈕。  

> [!NOTE]
> 必須要有 Java JDK 和 Maven，才能建置 Java 專案。 如果您使用代理程式來進行持續整合，請將這些元件安裝在 Jenkins Master 上。 如果您要部署 Java SE 應用程式，組建伺服器上還需要有 ZIP。
>

若要安裝這些元件，請使用 SSH 登入 Jenkins 執行個體，然後執行下列命令：

```bash
sudo apt-get install -y openjdk-7-jdk
sudo apt-get install -y maven
```

若要部署到「適用於容器的 Web App」，請在 Jenkins Master 上或在用於組建的 VM 代理程式上安裝 Docker。 如需相關指示，請參閱[在 Ubuntu 上安裝 Docker](https://docs.docker.com/engine/installation/linux/ubuntu/) \(英文\)。

## <a name="service-principal"></a>將 Azure 服務主體新增到 Jenkins 認證

您必須要有 Azure 服務主體，才能部署到 Azure。 


1. 若要建立 Azure 服務主體，請使用 [Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json) 或 [Azure 入口網站](/azure/azure-resource-manager/resource-group-create-service-principal-portal)。
2. 在 Jenkins 儀表板上，選取 [Credentials] \(認證\) > [System] \(系統\)。 然後，選取 [Global credentials(unrestricted)] \(全域認證 (不受限)\)。
3. 若要新增 Microsoft Azure 服務主體，請選取 [新增認證]。 為 [訂用帳戶 ID]、[用戶端識別碼]、[用戶端密碼] 及 [OAuth 2.0 權杖端點] 欄位輸入值。 將 [識別碼] 欄位設定為 **mySp**。 我們會在本文的後續步驟中用到此識別碼。


## <a name="configure-jenkins-to-deploy-web-apps-by-uploading-files"></a>設定 Jenkins 以透過上傳檔案來部署應用程式

若要將專案部署到 Web Apps，您可以透過檔案上傳來上傳組建成品。 Azure App Service 支援多個部署選項。 Azure App Service Jenkins 外掛程式可讓您輕鬆使用根據檔案類型所衍生的部署選項。 

* 若為 Java EE 應用程式，會使用 [WAR 部署](/azure/app-service/app-service-deploy-zip#deploy-war-file)。
* 若為 Java SE 應用程式，會使用 [ZIP 部署](/azure/app-service/app-service-deploy-zip#deploy-zip-file)。
* 若為其他語言，則會使用 [Git 部署](/azure/app-service/app-service-deploy-local-git)。

在於 Jenkins 中設定作業之前，您必須要有 Azure App Service 方案和 Web 應用程式，才能執行 Java 應用程式。


1. 使用 `az appservice plan create` [Azure CLI 命令](/cli/azure/appservice/plan#az-appservice-plan-create)來建立搭配**免費**定價層的 Azure App Service 方案。 App Service 方案會定義用來裝載您應用程式的實體資源。 指派給 App Service 方案的所有應用程式會共用這些資源。 共用資源可協助您在裝載多個應用程式時節省成本。
2. 建立 Web 應用程式。 您可以使用 [Azure 入口網站](/azure/app-service-web/web-sites-configure) 或下列 `az` Azure CLI 命令：
    ```azurecli-interactive 
    az webapp create --name <myAppName> --resource-group <myResourceGroup> --plan <myAppServicePlan>
    ```
    
3. 設定您應用程式所需的 Java 執行階段組態。 下列 Azure CLI 命令會將 Web 應用程式設定為在最新的 JDK 8 和 [Apache Tomcat](http://tomcat.apache.org/) 8.0 版上執行：
    ```azurecli-interactive
    az webapp config set \
    --name <myAppName> \
    --resource-group <myResourceGroup> \
    --java-version 1.8 \
    --java-container Tomcat \
    --java-container-version 8.0
    ```

### <a name="set-up-the-jenkins-job"></a>設定 Jenkins 作業

1. 在 Jenkins 儀表板上建立新的**自由樣式**專案。
2. 將 [Source Code Management] \(原始程式碼管理\) 欄位設定為使用您的[適用於 Azure 的簡單 Java Web 應用程式](https://github.com/azure-devops/javawebappsample)本機分支。 提供 [Repository URL] \(儲存機制 URL\) 值。 例如： http://github.com/&ltyour_ID>/javawebappsample。
3. 使用 Maven 藉由新增**執行 shell** 命令來新增一個建置專案的步驟。 在此範例中，我們需要一個額外的命令來將目標資料夾中的 \*.war 檔案重新命名為 **ROOT.war**：   
    ```bash
    mvn clean package
    mv target/*.war target/ROOT.war
    ```

4. 選取 [發行 Azure Web 應用程式] 來新增建置後動作。
5. 輸入 **mySp** 作為 Azure 服務主體。 此主體在先前的步驟中已儲存為 [Azure 認證](#service-principal)。
6. 在 [應用程式組態] 區段中，選擇您訂用帳戶中的資源群組和 Web 應用程式。 Jenkins 外掛程式會自動偵測 Web 應用程式是採用 Windows 架構還是 Linux 架構。 若為 Windows Web 應用程式，就會顯示 [發行檔案] 選項。
7. 填入您要部署的檔案。 例如，如果您使用 Java，請指定 WAR 套件。 使用選擇性的 [來源目錄] 和 [目標目錄] 參數來指定用於上傳檔案的來源和目標資料夾。 Azure 上的 Java Web 應用程式會在 Tomcat 伺服器中執行。 因此，就 Java 而言，您需將 WAR 套件上傳到 [webapps] 資料夾。 在此範例中，請將 [來源目錄] 值設定為 **target**，將 [目標目錄] 值設定為 **webapps**。
8. 如果您想要部署到生產環境以外的位置，您也可以設定 [位置] 名稱。
9. 儲存專案並建置該專案。 建置完成時，您的 Web 應用程式就會部署到 Azure。

### <a name="deploy-web-apps-by-uploading-files-using-jenkins-pipeline"></a>透過使用 Jenkins 管線上傳檔案來部署 Web 應用程式

Azure App Service Jenkins 外掛程式是符合管線需求的外掛程式。 您可以參考以下 GitHub 儲存機制中的範例。

1. 在 GitHub 介面中，開啟 **Jenkinsfile_ftp_plugin** 檔案。 若要編輯此檔案，請選取鉛筆圖示。 針對您的 Web 應用程式，分別更新第 11 行和第 12 行上的 **resourceGroup** 和 **webAppName** 定義：
    ```java
    def resourceGroup = '<myResourceGroup>'
    def webAppName = '<myAppName>'
    ```

2. 將第 14 行上的 **withCredentials** 定義設定為您 Jenkins 執行個體中的認證識別碼：
    ```java
    withCredentials([azureServicePrincipal('<mySp>')]) {
    ```

### <a name="create-a-jenkins-pipeline"></a>建立 Jenkins 管線

1. 在網頁瀏覽器中開啟 Jenkins。 選取 [新增項目]。
2. 為作業提供一個名稱，然後選取 [Pipeline] \(管線\)。 選取 [確定] 。
3. 選取 [Pipeline] \(管線\) 索引標籤。
4. 針對 [Definition] \(定義\) 值，選取 [Pipeline script from SCM] \(來自 SCM 的管線指令碼\)。
5. 針對 [SCM] 值，選取 [Git]。 輸入您分支儲存機制的 GitHub URL。 例如： https://&lt;your_forked_repo>.git。
6. 將 [Script Path] \(指令碼路徑\) 值更新成 **Jenkinsfile_ftp_plugin**。
7. 按一下 [Save] \(儲存\) 並執行作業。

## <a name="configure-jenkins-to-deploy-web-app-for-containers"></a>設定 Jenkins 以部署適用於容器的 Web App

Linux 上的 Web Apps 支援使用 Docker 來進行部署。 若要使用 Docker 來部署您的 Web 應用程式，您必須提供一個 Dockerfile，此檔案會將您的 Web 應用程式與服務執行階段封裝成 Docker 映像。 接著，Jenkins 外掛程式會建置該映像，將它推送到 Docker 登錄，然後將該映像部署到您的 Web 應用程式。

Linux 上的 Web Apps 也支援 Git 和檔案上傳等傳統部署方法，但僅適用於內建語言 (.NET Core、Node.js、PHP 和 Ruby)。 若為其他語言，則需要將您的應用程式程式碼和服務執行階段一起封裝到 Docker 映像，然後使用 Docker 來部署。

在於 Jenkins 中設定作業之前，您必須在 Linux 上擁有一個 Web 應用程式。 您還需要有一個容器登錄，以便儲存和管理您的私人 Docker 容器映像。 您可以使用 DockerHub 來建立容器登錄。 在此範例中，我們使用 Azure Container Registry。

* [在 Linux 上建立 Web 應用程式](../app-service/containers/quickstart-nodejs.md)。
* Azure Container Registry 是一個受控 [Docker 登錄](https://docs.docker.com/registry/)服務，此服務以開放原始碼的 Docker Registry 2.0 版為基礎。 [建立 Azure 容器登錄](/azure/container-registry/container-registry-get-started-azure-cli)。 您也可以使用 DockerHub。

### <a name="set-up-the-jenkins-job-for-docker"></a>設定 Docker 的 Jenkins 作業

1. 在 Jenkins 儀表板上建立新的**自由樣式**專案。
2. 將 [Source Code Management] \(原始程式碼管理\) 欄位設定為使用您的[適用於 Azure 的簡單 Java Web 應用程式](https://github.com/azure-devops/javawebappsample)本機分支。 提供 [Repository URL] \(儲存機制 URL\) 值。 例如： http://github.com/&ltyour_ID>/javawebappsample。
3. 使用 Maven 藉由新增**執行 shell** 命令來新增一個建置專案的步驟。 在命令中包含下列一行：
    ```bash
    mvn clean package
    ```

4. 選取 [發行 Azure Web 應用程式] 來新增建置後動作。
5. 輸入 **mySp** 作為 Azure 服務主體。 此主體在先前的步驟中已儲存為 [Azure 認證](#service-principal)。
6. 在 [應用程式組態] 區段中，選擇您訂用帳戶中的資源群組和 Linux Web 應用程式。
7. 選擇 [透過 Docker 發行]。
8. 填寫 **Dockerfile** 路徑值。 您可以保留預設值 /Dockerfile。
針對 [Docker 登錄 URL] 值，如果您使用 Azure Container Registry，請使用 https://&lt;yourRegistry>.azurecr.io 格式來輸入 URL。 如果您使用 DockerHub，請將值保留空白。
9. 針對 [登錄認證] 值，新增容器登錄的認證。 您可以在 Azure CLI 中執行下列命令，以取得使用者識別碼和密碼。 第一個命令會啟用系統管理員帳戶：
    ```azurecli-interactive
    az acr update -n <yourRegistry> --admin-enabled true
    az acr credential show -n <yourRegistry>
    ```

10. [進階] 索引標籤中的 Docker 映像名稱和標記值是選擇性的值。 預設會從您在 Azure 入口網站的 [Docker 容器] 設定中 中設定的映像名稱，取得映像名稱的值。 標記會從 $BUILD_NUMBER 產生。
    > [!NOTE]
    > 請務必在 Azure 入口網站中指定映像名稱，或是在 [進階] 索引標籤中提供 [Docker 映像] 值。針對此範例，請將 [Docker 映像] 值設定為 &lt;your_Registry>.azurecr.io/calculator，並將 [Docker 映像標記] 值保留空白。

11. 如果您使用內建的 Docker 映像設定，部署將會失敗。 請在 Azure 入口網站的 [Docker 容器] 設定中，將 Docker 組態變更為使用自訂映像。 針對內建映像，請使用檔案上傳方法來進行部署。
12. 與檔案上傳方法類似，您也可以選擇**生產環境**以外的不同**位置**。
13. 儲存並建置專案。 系統會將您的容器映像推送到登錄中，並且部署 Web 應用程式。

### <a name="deploy-web-app-for-containers-by-using-jenkins-pipeline"></a>使用 Jenkins 管線來部署適用於容器的 Web App

1. 在 GitHub 介面中，開啟 **Jenkinsfile_container_plugin** 檔案。 若要編輯此檔案，請選取鉛筆圖示。 針對您的 Web 應用程式，分別更新第 11 行和第 12 行上的 **resourceGroup** 和 **webAppName** 定義：
    ```java
    def resourceGroup = '<myResourceGroup>'
    def webAppName = '<myAppName>'
    ```

2. 將第 13 行變更為您的容器登錄伺服器：
    ```java
    def registryServer = '<registryURL>'
    ```

3. 將第 16 行變更為使用您 Jenkins 執行個體中的認證識別碼：
    ```java
    azureWebAppPublish azureCredentialsId: '<mySp>', publishType: 'docker', resourceGroup: resourceGroup, appName: webAppName, dockerImageName: imageName, dockerImageTag: imageTag, dockerRegistryEndpoint: [credentialsId: 'acr', url: "http://$registryServer"]
    ```

### <a name="create-a-jenkins-pipeline"></a>建立 Jenkins 管線    

1. 在網頁瀏覽器中開啟 Jenkins。 選取 [新增項目]。
2. 為作業提供一個名稱，然後選取 [Pipeline] \(管線\)。 選取 [確定] 。
3. 選取 [Pipeline] \(管線\) 索引標籤。
4. 針對 [Definition] \(定義\) 值，選取 [Pipeline script from SCM] \(來自 SCM 的管線指令碼\)。
5. 針對 [SCM] 值，選取 [Git]。 輸入您分支儲存機制的 GitHub URL。 例如： https://&lt;your_forked_repo>.git。
7. 將 [Script Path] \(指令碼路徑\) 值更新成 **Jenkinsfile_container_plugin**。
8. 按一下 [Save] \(儲存\) 並執行作業。

## <a name="verify-your-web-app"></a>確認您的 Web 應用程式

1. 若要確認 WAR 檔案是否已順利部署到您的 Web 應用程式，請開啟網頁瀏覽器。
2. 移至 http://&lt;your_app_name>.azurewebsites.net/api/calculator/ping。 以您的 Web 應用程式名稱取代 &lt;your_app_name>。 您會看見此訊息：
    ```
    Welcome to Java Web App!!! This is updated!
    Sun Jun 17 16:39:10 UTC 2017
    ```

3. 移至 http://&lt;your_app_name>.azurewebsites.net/api/calculator/add?x=&lt;x>&y=&lt;y>。 以任意數字取代 &lt;x> 和 &lt;y> 來得出 x + y 的總和。 計算機會顯示總和：![計算機：加法](./media/execute-cli-jenkins-pipeline/calculator-add.png)

### <a name="for-azure-app-service-on-linux"></a>針對 Linux 上的 Azure App Service

1. 若要確認您的 Web 應用程式，請在 Azure CLI 中執行下列命令：
    ```CLI
    az acr repository list -n <myRegistry> -o json
    ```
    隨即會顯示下列訊息：
    ```CLI
    ["calculator"]
    ```
    
2. 移至 http://&lt;your_app_name>.azurewebsites.net/api/calculator/ping。 以您的 Web 應用程式名稱取代 &lt;your_app_name>。 您會看見此訊息： 
    ```
    Welcome to Java Web App!!! This is updated!
    Sun Jul 09 16:39:10 UTC 2017
    ```

3. 移至 http://&lt;your_app_name>.azurewebsites.net/api/calculator/add?x=&lt;x>&y=&lt;y>。 以任意數字取代 &lt;x> 和 &lt;y> 來得出 x + y 的總和。
    
## <a name="troubleshooting-the-jenkins-plugin"></a>對 Jenkins 外掛程式進行疑難排解

如果您遇到任何有關 Jenkins 外掛程式的錯誤，請在 [Jenkins JIRA](https://issues.jenkins-ci.org/) 的特定元件中提交問題。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已使用 Azure App Service Jenkins 外掛程式來部署到 Azure。

您已了解如何︰

> [!div class="checklist"]
> * 將 Jenkins 設定為透過檔案上傳部署 Azure App Service 
> * 設定 Jenkins 以部署到適用於容器的 Web App 
