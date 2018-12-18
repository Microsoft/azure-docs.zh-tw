---
title: 搭配 Azure Container Service 引擎和 Swarm 模式使用 CI/CD
description: 搭配 Docker Swarm 模式、Azure Container Registry 及 Azure DevOps 使用 Azure Container Service 引擎，以持續傳遞多容器 .NET Core 應用程式
services: container-service
author: diegomrtnzg
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 05/27/2017
ms.author: diegomrtnzg
ms.custom: mvc
ms.openlocfilehash: 296c097ee3302eaa39210274b16c6352866eac8a
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44299571"
---
# <a name="full-cicd-pipeline-to-deploy-a-multi-container-application-on-azure-container-service-with-acs-engine-and-docker-swarm-mode-using-azure-devops"></a>使用 Azure DevOps 的完整 CI/CD 管線，搭配 ACS 引擎和 Docker Swarm 模式在 Azure Container Service 上部署多容器應用程式

*本文是以[使用 Azure DevOps 的完整 CI/CD 管線，搭配 Docker Swarm 在 Azure Container Service 上部署多容器應用程式](container-service-docker-swarm-setup-ci-cd.md)一文為基礎*

現在，為雲端開發現代化應用程式的其中一個最大挑戰是要能持續傳遞這些應用程式。 在本文中，您會了解如何實作完整的持續整合和部署 (CI/CD) 管線，使用： 
* 使用 Docker Swarm 模式的 Azure Container Service 引擎
* Azure Container Registry
* Azure DevOps

本文是以一個使用 ASP.NET Core 開發的簡單應用程式 (可在 [GitHub](https://github.com/jcorioland/MyShop/tree/docker-linux) 上取得) 為依據。 該應用程式由四個不同的服務組成：三個 Web API 和一個 Web 前端：

![MyShop 範例應用程式](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/myshop-application.png)

目標是使用 Azure DevOps 在 Docker Swarm 模式叢集中持續傳遞此應用程式。 下圖詳述此持續傳遞管線：

![MyShop 範例應用程式](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/full-ci-cd-pipeline.png)

以下是個步驟的簡短說明：

1. 程式碼變更認可至原始程式碼存放庫 (在此為 GitHub) 
2. GitHub 觸發 Azure DevOps 中的組建 
3. Azure DevOps 取得最新版本的來源，並建置構成應用程式的所有映像 
4. Azure DevOps 將每個映像推送到使用 Azure Container Registry 服務建立的 Docker 登錄 
5. Azure DevOps 觸發新發行 
6. 該版本在 Azure Container Service 叢集主要節點上使用 SSH 執行一些命令 
7. 叢集上的 Docker Swarm 模式提取最新版本的映像 
8. 使用 Docker 堆疊部署應用程式的最新版本 

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前，您需要完成下列工作：

- [使用 ACS 引擎在 Azure Container Service 中建立 Swarm 模式叢集](https://github.com/Azure/azure-quickstart-templates/tree/master/101-acsengine-swarmmode)
- [連接到 Azure 容器服務中的 Swarm 叢集](../container-service-connect.md)
- [建立 Azure 容器登錄](../../container-registry/container-registry-get-started-portal.md)
- [具有 Azure DevOps 組織並已建立專案](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization-msa-or-work-student)
- [在您的 GitHub 帳戶建立該 GitHub 存放庫的分叉 (英文)](https://github.com/jcorioland/MyShop/tree/docker-linux)

>[!NOTE]
> Azure Container Service 中的 Docker Swarm Orchestrator 會使用舊版獨立 Swarm。 整合式 [Swarm 模式](https://docs.docker.com/engine/swarm/) (Docker 1.12 及更高版本) 目前不是 Azure Container Service 中支援的 Orchestrator。 基於這個理由，我們會使用 [ACS 引擎](https://github.com/Azure/acs-engine/blob/master/docs/swarmmode.md) (社群貢獻的[快速入門範本](https://azure.microsoft.com/resources/templates/101-acsengine-swarmmode/)) 或 [Azure Marketplace](https://azuremarketplace.microsoft.com) 的 Docker 解決方案。
>

## <a name="step-1-configure-your-azure-devops-organization"></a>步驟 1：設定您的 Azure DevOps 組織 

在本節中，您將設定 Azure DevOps 組織。 若要設定 Azure DevOps 服務端點，請在您的 Azure DevOps 專案中，按一下工具列中的 [設定] 圖示，然後選取 [服務]。

![開啟服務端點](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/services-vsts.PNG)

### <a name="connect-azure-devops-and-azure-account"></a>連線 Azure DevOps 與 Azure 帳戶

設定 Azure DevOps 專案與 Azure 帳戶之間的連線。

1. 在左側，按一下 [新增服務端點] > [Azure Resource Manager]。
2. 若要授權讓 Azure DevOps 使用您的 Azure 帳戶，請選取您的 [訂用帳戶]，並按一下 [確定]。

    ![Azure DevOps - 授權 Azure](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-azure.PNG)

### <a name="connect-azure-devops-and-github"></a>連線 Azure DevOps 與 GitHub

設定您的 Azure DevOps 專案與 GitHub 帳戶之間的連線。

1. 在左側，按一下 [新增服務端點] > [GitHub]。
2. 若要授權讓 Azure DevOps 與您的 GitHub 帳戶搭配使用，請按一下 [授權]，並依照開啟視窗中的程序操作。

    ![Azure DevOps - 授權 GitHub](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-github.png)

### <a name="connect-azure-devops-to-your-azure-container-service-cluster"></a>將 Azure DevOps 連線到 Azure Container Service 叢集

進入 CI/CD 管線的最後步驟是在 Azure 中設定 Docker Swarm 叢集的外部連線。 

1. 針對 Docker Swarm 叢集新增 [SSH] 類型的端點。 然後輸入您的 Swarm 叢集 (主要節點) 的 SSH 連線資訊。

    ![Azure DevOps - SSH](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-ssh.png)

已經完成所有設定。 在下一步中，您會建立建置應用程式並部署到 Docker Swarm 叢集的 CI/CD 管線。 

## <a name="step-2-create-the-build-pipeline"></a>步驟 2：建立組建管線

在此步驟中，您會設定 Azure DevOps 專案的組建管線，並定義容器映像的組建工作流程

### <a name="initial-pipeline-setup"></a>初始管線設定

1. 若要建立組建管線，請連線到您的 Azure DevOps 專案，然後按一下 [組建與發行]。 在 [組件定義] 區段中，按一下 [+ 新增]。 

    ![Azure DevOps - 新增組建管線](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/create-build-vsts.PNG)

2. 選取 [空白流程]。

    ![Azure DevOps - 新增空的組建管線](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/create-empty-build-vsts.PNG)

4. 然後，按一下 [變數] 索引標籤，建立兩個新的變數：**RegistryURL** 和 **AgentURL**。 貼上登錄與叢集代理程式 DNS 的值。

    ![Azure DevOps - 組建變數組態](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-build-variables.png)

5. 在 [組建定義] 頁面中，開啟 [觸發程序] 索引標籤，並使用您在必要條件中建立的 MyShop 專案分叉持續整合來設定組建。 接著，選取 [批次變更]。 確定選取 *docker-linux* 為**分支規格**。

    ![Azure DevOps - 組建存放庫組態](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-github-repo-conf.PNG)


6. 最後，按一下 [選項] 索引標籤，並將預設代理程式佇列設定為 [Hosted Linux Preview] \(託管的 Linux 預覽)。

    ![Azure DevOps - 主機代理程式組態](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-build-agent.png)

### <a name="define-the-build-workflow"></a>定義組建工作流程
接下來的步驟會定義組建工作流程。 首先，您需要設定程式碼的來源。 若要這麼做，請選取 [GitHub] 以及 [儲存機制] 和 [分支] \(docker-linux)。

![Azure DevOps - 設定原始程來源](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-source-code.png)

要為 *MyShop* 應用程式建置五個容器映像。 每個映像都是使用專案資料夾中的 Dockerfile 來建置：

* ProductsApi
* Proxy
* RatingsApi
* RecommandationsApi
* ShopFront

每個映像需要兩個 Docker 步驟，一個是建置映像，另一個是將映像推送至 Azure Container Registry。 

1. 若要在組建工作流程中新增步驟，按一下 [+ 加入建置步驟] 然後選取 [Docker]。

    ![Azure DevOps - 新增組建步驟](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-build-add-task.png)

2. 針對每個映像，設定一個使用 `docker build` 命令的步驟。

    ![Azure DevOps - Docker 組建](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-docker-build.png)

    針對建置作業，選取您的 Azure Container Registry、[Build an image] \(建置映像) 動作，以及定義每個映像的 Dockerfile。 將 [工作目錄] 設為 Dockerfile 根目錄，定義 [映像名稱]，然後選取 [包含最新標記]。
    
    映像名稱格式必須為：```$(RegistryURL)/[NAME]:$(Build.BuildId)```。 以映像名稱取代 **[NAME]**：
    - ```proxy```
    - ```products-api```
    - ```ratings-api```
    - ```recommendations-api```
    - ```shopfront```

3. 針對每個映像，設定使用 `docker push` 命令的第二個步驟。

    ![Azure DevOps - Docker 推送](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-docker-push.png)

    針對推送作業，選取您的 Azure Container Registry，選取 [Push an image] \(推送映像) 動作，並輸入在上一個步驟中建置的**映像名稱**，然後選取 [包含最新標記]。

4. 針對這五個映像個別設定好建置和推送步驟之後，請在建置工作流程中再新增三個步驟。

   ![Azure DevOps - 新增命令列工作](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-build-command-task.png)

      1. 命令列工作會使用 bash 指令碼以 RegistryURL 變數取代 docker-compose.yml 檔案中的 *RegistryURL*。 
    
          ```-c "sed -i 's/RegistryUrl/$(RegistryURL)/g' src/docker-compose-v3.yml"```

          ![Azure DevOps - 使用登錄 URL 更新 Compose 檔案](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-build-replace-registry.png)

      2. 命令列工作會使用 bash 指令碼以 AgentURL 變數取代 docker-compose.yml 檔案中的 *AgentURL*。
  
          ```-c "sed -i 's/AgentUrl/$(AgentURL)/g' src/docker-compose-v3.yml"```

     3. 一個工作，會將更新的 Compose 檔案卸除成組建構件，讓它能夠在版本中使用。 請參閱以下畫面了解詳細資料。

         ![Azure DevOps - 發佈成品](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-publish.png) 

         ![Azure DevOps - 發佈 Compose 檔案](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-publish-compose.png) 

5. 按一下 [儲存並排入佇列] 以測試組建管線。

   ![Azure DevOps - 儲存並排入佇列](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-build-save.png) 

   ![Azure DevOps - 新增佇列](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-build-queue.png) 

6. 如果**建置**正確，您會看見這個畫面：

  ![Azure DevOps - 建置成功](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-build-succeeded.png) 

## <a name="step-3-create-the-release-pipeline"></a>步驟 3︰建立發行管線

Azure DevOps 可讓您[跨環境管理發行](https://www.visualstudio.com/team-services/release-management/)。 您可以啟用持續部署以確保應用程式會順利部署到不同環境 (例如開發、測試、進入生產階段前和生產)。 您可以建立代表 Azure Container Service Docker Swarm 模式叢集的環境。

![Azure DevOps - 發行至 ACS](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-release-acs.png) 

### <a name="initial-release-setup"></a>初始發行設定

1. 若要建立發行管線，請按一下 [發行] > [+ 發行]

2. 若要設定構件來源，請按一下 [構件] > [連結構件來源]。 在此，請將這個新的發行管線連結到您在上一個步驟中定義的組建。 之後，就可以在發行程序中取得 docker-compose.yml 檔案。

    ![Azure DevOps - 發行成品](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-release-artefacts.png) 

3. 若要設定發行觸發程序，請按一下 [觸發程序]，然後選取 [持續部署]。 在相同的構件來源上設定觸發程序。 此設定可確保順利完成建置後，就會開始新的發行。

    ![Azure DevOps - 發行觸發程序](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-release-trigger.png) 

4. 若要設定發行變數，請按一下 [變數]，然後選取 [+變數] 以登錄資訊建立三個新的變數：**docker.username**、**docker.password** 和 **docker.registry**。 貼上登錄與叢集代理程式 DNS 的值。

    ![Azure DevOps - 組建存放庫組態](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-release-variables.png)

    >[!IMPORTANT]
    > 如上一個畫面所示，按一下 docker.password 中的 [鎖定] 核取方塊。 這項設定對限制密碼很重要。
    >

### <a name="define-the-release-workflow"></a>定義發行工作流程

發行工作流程由您新增的兩個工作組成。

1. 設定工作並使用您先前設定的 SSH 連線來安全地將 Compose 檔案複製到 Docker Swarm 主要節點上的 [deploy] 資料夾。 請參閱以下畫面了解詳細資料。
    
    來源資料夾：```$(System.DefaultWorkingDirectory)/MyShop-CI/drop```

    ![Azure DevOps - 發行 SCP](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-release-scp.png)

2. 設定第二個工作以在主要節點上執行 Bash 命令以執行 `docker` 和 `docker stack deploy` 命令。 請參閱以下畫面了解詳細資料。

    ```docker login -u $(docker.username) -p $(docker.password) $(docker.registry) && export DOCKER_HOST=:2375 && cd deploy && docker stack deploy --compose-file docker-compose-v3.yml myshop --with-registry-auth```

    ![Azure DevOps - 發行 Bash](./media/container-service-docker-swarm-mode-setup-ci-cd-acs-engine/vsts-release-bash.png)

    在主要節點上執行的命令會使用 Docker CLI 和 Docker-Compose CLI 執行下列工作：

    - 登入 Azure Container Registry (它使用 [變數] 索引標籤中定義的三個組建變數)
    - 定義 **DOCKER_HOST** 變數來與 Swarm 端點 (:2375) 搭配使用
    - 瀏覽到先前安全複製工作建立的 [deploy] 資料夾，其中包含 docker-compose.yml 檔案 
    - 執行提取新映像的 `docker stack deploy` 命令，然後建立容器。

    >[!IMPORTANT]
    > 如先前畫面所示，讓 [在 STDERR 上失敗] 核取方塊保持未選取。 這個設定能讓我們完成發行程序，因為 `docker-compose` 會在標準錯誤輸出上印出數個診斷訊息 (例如容器已停止或已刪除)。 如果您選取該核取方塊，即使一切正常運作，Azure DevOps 仍會報告發行期間出現錯誤。
    >
3. 儲存這個新的發行管線。

## <a name="step-4-test-the-cicd-pipeline"></a>步驟 4：測試 CI/CD 管線

您已經完成設定，現在可以開始測試這個新的 CI/CD 管線。 最簡單的測試方法是更新原始程式碼，然後將變更認可到您的 GitHub 存放庫。 推送程式碼的數秒之後，您會看到新的組建在 Azure DevOps 中執行。 一旦順利完成，就會觸發新的發行，並且在 Azure Container Service 叢集中部署新版的應用程式。

## <a name="next-steps"></a>後續步驟

* 如需 CI/CD 與 Azure DevOps 的相關詳細資訊，請參閱 [Azure DevOps 組建概觀](https://www.visualstudio.com/docs/build/overview)。
* 如需 ACS 引擎的詳細資訊，請參閱 [ACS Engine GitHub repo](https://github.com/Azure/acs-engine) (ACS 引擎 GitHub 儲存機制)。
* 如需 Docker Swarm 模式的詳細資訊，請參閱 [Swarm mode overview](https://docs.docker.com/engine/swarm/) (Swarm 模式概觀)。
