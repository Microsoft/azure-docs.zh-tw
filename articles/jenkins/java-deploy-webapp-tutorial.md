---
title: 使用 Jenkins 將 Web 應用程式部署至 Azure
description: 使用 Jenkins 與 Docker 設定 Java Web 應用程式從 GitHub 到 Azure App Service 的持續整合。
ms.service: jenkins
keywords: jenkins, azure, devops, app service, 持續整合, ci, 持續部署, cd
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 07/31/2018
ms.openlocfilehash: 81959159d4860512c184ada25930da814b5ae044
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946573"
---
# <a name="set-up-continuous-integration-and-deployment-to-azure-app-service-with-jenkins"></a>使用 Jenkins 設定 Azure App Service 的持續整合和部署

本教學課程使用 Jenkins，設定 [Spring Boot](http://projects.spring.io/spring-boot/) 架構開發的 Java Web 應用程式範例到 [Linux 上的 Azure App Service Web 應用程式](/azure/app-service/containers/app-service-linux-intro)的持續整合與部署 (CI/CD)。

您會在本教學課程中執行下列工作：

> [!div class="checklist"]
> * 安裝部署至 Azure App Service 時所需的 Jenkins 外掛程式。
> * 定義 Jenkins 作業，以在發送新認可時，從 GitHub 儲存機制組建 Docker 映像。
> * 定義適用於 Linux 的新 Azure Web 應用程式，並加以設定以部署推送至 Azure Container Registry 的 Docker 映像。
> * 設定 Azure App Service Jenkins 外掛程式。
> * 透過手動組建將範例應用程式部署到 Azure App Service。
> * 觸發 Jenkins 組建，並藉由推送變更至 GitHub 更新 Web 應用程式。

## <a name="before-you-begin"></a>開始之前

若要完成本教學課程，您需要：

* 已設定 JDK 和 Maven 工具的 [Jenkins](https://jenkins.io/)。 如果您沒有 Jenkins 系統，請立即從 [Jenkins 解決方案範本](/azure/jenkins/install-jenkins-solution-template)在 Azure 中建立一個。
* [GitHub](https://github.com) 帳戶。
* [Azure CLI](/cli/azure) (不論是從本機命令列，還是在 [Azure Cloud Shell](/azure/cloud-shell/overview) 中)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="install-jenkins-plug-ins"></a>安裝 Jenkins 外掛程式

1. 開啟網頁瀏覽器至您的 Jenkins Web 主控台，並從左側功能表選取 [管理 Jenkins]，然後選取 [管理外掛程式]。
2. 選取 [可用] 索引標籤。
3. 搜尋並選取下列外掛程式旁的核取方塊：   

    - [Azure App Service 外掛程式](https://plugins.jenkins.io/azure-app-service)
    - [GitHub 分支來源外掛程式](https://plugins.jenkins.io/github-branch-source)

    如果未顯示外掛程式，請檢查 [已安裝] 索引標籤，確定其尚未安裝。

1. 選取 [在重新啟動後立即下載並安裝]，以啟用 Jenkins 組態中的外掛程式。

## <a name="configure-github-and-jenkins"></a>設定 GitHub 與 Jenkins

將 Jenkins 設定為在推送新認可至您帳戶中的儲存機制時收到 [GitHub Webhook](https://developer.github.com/webhooks/)。

1. 選取 [管理 Jenkins]，然後選取 [設定系統]。 在 [GitHub] 區段中，確定已選取 [管理勾點]，然後選取 [管理其他 GitHub 動作]，並選擇 [將登入和密碼轉換成權杖]。
2. 選取 [從登入和密碼] 選項按鈕，並輸入您的 GitHub 使用者名稱及密碼。 選取 [建立權杖認證]，以建立新的 [GitHub 個人存取權杖](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)。   
   ![從登入和密碼建立 GitHub PAT](media/jenkins-java-quickstart/create_github_credentials.png)
3.  從 GitHub 伺服器設定中的 [認證] 下拉式清單選取新建的權仗。 選取 [測試連接]，以確認驗證正常運作。   
   ![設定 PAT 之後，驗證 GitHub 的連線](media/jenkins-java-quickstart/verify_github_connection.png)

> [!NOTE]
> 如果您的 GitHub 帳戶已啟用雙因素驗證，請建立 GitHub 上的權仗，並設定 Jenkins 以使用該權仗。 如需完整的詳細資料，請檢閱 [Jenkins GitHub 外掛程式](https://wiki.jenkins.io/display/JENKINS/Github+Plugin)說明文件。

## <a name="fork-the-sample-repo-and-create-a-jenkins-job"></a>派生範例儲存機制，並建立 Jenkins 作業 

1. 開啟 [Spring Boot 範例應用程式儲存機制](https://github.com/spring-guides/gs-spring-boot-docker) ，並選取右上角的 [派生]，將其派生至您擁有的 GitHub 帳戶。   
    ![從 GitHub 派生](media/jenkins-java-quickstart/fork_github_repo.png)
1. 在 Jenkins Web 主控台中，選取 [新增項目]，為該項目指定名稱 **MyJavaApp**，選取 [自由樣式專案]，然後選取 [確定]。   
    ![新增 Jenkins 自由樣式專案](media/jenkins-java-quickstart/jenkins_freestyle.png)
2. 在 [一般] 區段下，選取 [GitHub 專案]，然後輸入您的分支存放庫 URL，例如 https://github.com/raisa/gs-spring-boot-docker
3. 在 [原始碼管理] 區段中，選取 [Git]，輸入您的分支存放庫 `.git` URL，例如 https://github.com/raisa/gs-spring-boot-docker.git
4. 在 [組建觸發程序] 下，選取 [GITScm 輪詢的 GitHub 勾點觸發程序]。
5. 在 [組建] 區段下，選取 [新增組建步驟]，然後選擇 [叫用最上層 Maven 目標]。 在 [目標] 欄位中輸入 `package`。
6. 選取 [ **儲存**]。 您可以藉由從專案頁面中選取 [立即組建]，以測試您的作業。

## <a name="configure-azure-app-service"></a>設定 Azure App Service 

1. 使用 Azure CLI 或 [Cloud Shell](/azure/cloud-shell/overview)，建立新的 [Linux 上的 Web 應用程式](/azure/app-service/containers/app-service-linux-intro)。 本教學課程中的 Web 應用程式名稱是 `myJavaApp`，但是您必須為自己的應用程式使用唯一名稱。
   
    ```azurecli-interactive
    az group create --name myResourceGroupJenkins --location westus
    az appservice plan create --is-linux --name myLinuxAppServicePlan --resource-group myResourceGroupJenkins 
    az webapp create --name myJavaApp --resource-group myResourceGroupJenkins --plan myLinuxAppServicePlan --runtime "java|1.8|Tomcat|8.5"
    ```

2. 建立 [Azure Container Registry](/azure/container-registry/container-registry-intro)，以儲存 Jenkins 建立的 Docker 映像。 本教學課程中使用的容器登錄名稱是 `jenkinsregistry`，但是您自己的容器登錄需使用專屬的唯一名稱。 

    ```azurecli-interactive
    az acr create --name jenkinsregistry --resource-group myResourceGroupJenkins --sku Basic --admin-enabled
    ```
3. 設定 Web 應用程式，以執行推送至容器登錄的 Docker 映像，並指定在該容器中執行的應用程式監聽連接埠 8080 上的要求。   

    ```azurecli-interactive
    az webapp config container set -c jenkinsregistry/webapp --resource-group myResourceGroupJenkins --name myJavaApp
    az webapp config appsettings set --resource-group myResourceGroupJenkins --name myJavaApp --settings PORT=8080
    ```

## <a name="configure-the-azure-app-service-jenkins-plug-in"></a>設定 Azure App Service Jenkins 外掛程式

1. 在 Jenkins Web 主控台中，選取您建立的 [MyJavaApp] 作業，然後在頁面左側選取 [設定]。
2. 向下捲動至 [建置後動作]，選取 [新增建置後動作]，然後選擇 [發佈 Azure Web 應用程式]。
3. 在 [Azure 設定檔組態] 下，選取 [Azure 認證] 旁的 [新增]，並選擇 [Jenkins]。
4. 在 [新增認證] 對話方塊中，從 [種類] 下拉式清單中選取 [Microsoft Azure 服務主體]。
5. 從 Azure CLI 或 [Cloud Shell](/azure/cloud-shell/overview) 建立 Active Directory 服務主體。
    
    ```azurecli-interactive
    az ad sp create-for-rbac --name jenkins_sp --password secure_password
    ```

    ```json
    {
        "appId": "BBBBBBBB-BBBB-BBBB-BBBB-BBBBBBBBBBB",
        "displayName": "jenkins_sp",
        "name": "http://jenkins_sp",
        "password": "secure_password",
        "tenant": "CCCCCCCC-CCCC-CCCC-CCCCCCCCCCC"
    }
    ```
6. 將服務主體的認證輸入 [新增認證] 對話方塊。 如果您不知道您的 Azure 訂用帳戶識別碼，您可以從 CLI 查詢：
     
     ```azurecli-interactive
     az account list
     ```

     ```json
        {
            "cloudName": "AzureCloud",
            "id": "AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA",
            "isDefault": true,
            "name": "Visual Studio Enterprise",
            "state": "Enabled",
            "tenantId": "CCCCCCCC-CCCC-CCCC-CCCC-CCCCCCCCCCC",
            "user": {
            "name": "raisa@fabrikam.com",
            "type": "user"
            }
     ```

    ![設定 Azure 服務主體](media/jenkins-java-quickstart/azure_service_principal.png)
6. 藉由選取 [驗證服務主體]，透過 Azure 驗證服務主體驗證。 
7. 選取 [新增] 以儲存認證。
8. 當您回到 [發佈 Azure Web 應用程式] 組態時，從 [Azure 認證] 下拉式清單選取剛剛新增的服務主體認證。
9. 在 [應用程式設定] 中，從下拉式清單選擇您的資源群組與 Web 應用程式名稱。
10. 選取 [透過 Docker 發佈] 選項按鈕。
11. 為 [Dockerfile 路徑] 輸入 `complete/Dockerfile`。
12. 在 [Docker 登錄 URL] 欄位中輸入 `https://jenkinsregistry.azurecr.io`。
13. 選取 [登錄認證] 旁的 [新增]。 
14. 在 [使用者名稱] 中，輸入您為 Azure Container Registry 建立的管理員使用者名稱。
15. 在 [密碼] 欄位中輸入 Azure Container Registry 的密碼。 您可以從 Azure 入口網站或透過下列 CLI 命令，取得您的使用者名稱和密碼：

    ```azurecli-interactive
    az acr credential show -n jenkinsregistry
    ```
    ![新增您的容器登錄認證](media/jenkins-java-quickstart/enter_acr_credentials.png)
15. 選取 [新增] 以儲存認證。
16. 在 [發佈 Azure Web 應用程式] 之 [應用程式設定] 面板的 [登錄認證] 下拉式清單中，選取新建立的認證。 已完成的建置後動作應該如下圖所示：   
    ![Azure App Service 部署的建置後動作設定](media/jenkins-java-quickstart/appservice_plugin_configuration.png)
17. 選取 [儲存] 以儲存作業設定。

## <a name="deploy-the-app-from-github"></a>從 GitHub 部署應用程式

1. 從 Jenkins 專案中，選取 [立即組建]，以將範例應用程式部署至 Azure。
2. 建置完成後，您的應用程式會依其發佈的 URL 存留在 Azure 上，例如 http://myjavaapp.azurewebsites.net。   
   ![在 Azure 上檢視已部署的應用程式](media/jenkins-java-quickstart/hello_docker_world_unedited.png)

## <a name="push-changes-and-redeploy"></a>推送變更並重新部署

1. 從您的 Github 分支，在網頁上瀏覽至 `complete/src/main/java/Hello/Application.java`。 從 GitHub 介面的右側選取 [編輯此檔案] 連結。
2. 對於 `home()` 方法進行下列變更，並認可對儲存機制主要分支所做的變更。
   
    ```java
    return "Hello Docker World on Azure";
    ```
3. 新組建會在 Jenkins 上啟動，由儲存機制之 `master` 分支上的新認可觸發。 完成時，請在 Azure 上重新載入您的應用程式。     
      ![在 Azure 上檢視已部署的應用程式](media/jenkins-java-quickstart/hello_docker_world.png)

## <a name="troubleshooting-the-jenkins-plugin"></a>對 Jenkins 外掛程式進行疑難排解

如果您遇到任何有關 Jenkins 外掛程式的錯誤，請在 [Jenkins JIRA](https://issues.jenkins-ci.org/) 的特定元件中提交問題。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 Azure VM 做為組建代理程式](/azure/jenkins/jenkins-azure-vm-agents)