---
title: 搭配 Azure Container Service 和 Swarm 使用 CI/CD
description: 搭配 Docker Swarm、Azure Container Registry 及 Azure DevOps 使用 Azure Container Service，以持續傳遞多容器 .NET Core 應用程式
services: container-service
author: jcorioland
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 12/08/2016
ms.author: jucoriol
ms.custom: mvc
ms.openlocfilehash: 3b91c269104e740add1d3a5b8ecaee93ca269188
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44302821"
---
# <a name="full-cicd-pipeline-to-deploy-a-multi-container-application-on-azure-container-service-with-docker-swarm-using-azure-devops-services"></a>使用 Azure DevOps Services 搭配 Docker Swarm 在 Azure Container Service 上部署多容器應用程式的完整 CI/CD 管線

為雲端開發現代化應用程式的其中一個最大挑戰是要能持續傳遞這些應用程式。 在本文中，您會了解如何搭配 Docker Swarm、Azure Container Registry 及 Azure Pipelines 使用 Azure Container Service 來實作完整的持續整合與部署 (CI/CD) 管線。

本文是以一個使用 ASP.NET Core 開發的簡單應用程式 (可在 [GitHub](https://github.com/jcorioland/MyShop/tree/acs-docs) 上取得) 為依據。 該應用程式由四個不同的服務組成：三個 Web API 和一個 Web 前端：

![MyShop 範例應用程式](./media/container-service-docker-swarm-setup-ci-cd/myshop-application.png)

目標是使用 Azure DevOps Services 在 Docker Swarm 叢集中持續傳遞此應用程式。 下圖詳述此持續傳遞管線：

![MyShop 範例應用程式](./media/container-service-docker-swarm-setup-ci-cd/full-ci-cd-pipeline.png)

以下是個步驟的簡短說明：

1. 程式碼變更認可至原始程式碼存放庫 (在此為 GitHub) 
1. GitHub 觸發 Azure DevOps Services 中的組建 
1. Azure DevOps Services 取得最新版本的來源，並建置構成應用程式的所有映像 
1. Azure DevOps Services 將每個映像推送到使用 Azure Container Registry 服務建立的 Docker 登錄 
1. Azure DevOps Services 觸發新發行 
1. 該版本在 Azure Container Service 叢集主要節點上使用 SSH 執行一些命令 
1. 叢集上的 Docker Swarm 提取最新版本的映像 
1. 使用 Docker Compose 部署應用程式的最新版本 

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前，您需要完成下列工作：

- [在 Azure 容器服務中建立 Swarm 叢集](container-service-deployment.md)
- [連接到 Azure 容器服務中的 Swarm 叢集](../container-service-connect.md)
- [建立 Azure 容器登錄](../../container-registry/container-registry-get-started-portal.md)
- [具有 Azure DevOps Services 組織並已建立專案](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization-msa-or-work-student)
- [在您的 GitHub 帳戶建立該 GitHub 存放庫的分叉 (英文)](https://github.com/jcorioland/MyShop/)

[!INCLUDE [container-service-swarm-mode-note](../../../includes/container-service-swarm-mode-note.md)]

您也需要已經安裝 Docker 的 Ubuntu (14.04 or 16.04) 機器。 Azure DevOps Services 在 Azure Pipelines 程序執行期間會使用此機器。 建立此機器的其中一個方法是使用可在 [Azure Marketplace (英文)](https://azure.microsoft.com/marketplace/partners/canonicalandmsopentech/dockeronubuntuserver1404lts/) 中取得的映像。 

## <a name="step-1-configure-your-azure-devops-services-organization"></a>步驟 1：設定您的 Azure DevOps Services 組織 

在本節中，您將設定 Azure DevOps Services 組織。

### <a name="configure-an-azure-devops-services-linux-build-agent"></a>設定 Azure DevOps Services Linux 組建代理程式

若要建立 Docker 映像，並將映像從 Azure DevOps Services 組建推送到 Azure 容器登錄，您需要註冊 Linux 代理程式。 安裝選項如下：

* [在 Linux 上部署代理程式 (英文)](https://www.visualstudio.com/docs/build/admin/agents/v2-linux)

* [使用 Docker 執行 Azure DevOps Services 代理程式](https://hub.docker.com/r/microsoft/vsts-agent)

### <a name="install-the-docker-integration-azure-devops-services-extension"></a>安裝 Docker Integration Azure DevOps Services 擴充功能

Microsoft 提供 Azure DevOps Services 擴充功能，以在 Azure Pipelines 程序中與 Docker 搭配使用。 此擴充功能可在 [Azure DevOps Services Marketplace](https://marketplace.visualstudio.com/items?itemName=ms-vscs-rm.docker) 中取得。 按一下 [安裝] 以將此擴充功能新增到您的 Azure DevOps Services 組織：

![安裝 Docker 整合](./media/container-service-docker-swarm-setup-ci-cd/install-docker-vsts.png)

系統會要求您使用認證連線到 Azure DevOps Services 組織。 

### <a name="connect-azure-devops-services-and-github"></a>連線 Azure DevOps Services 與 GitHub

設定您的 Azure DevOps Services 專案與 GitHub 帳戶之間的連線。

1. 在您的 Azure DevOps Services 專案中，按一下工具列中的 [設定] 圖示，然後選取 [服務]。

    ![Azure DevOps Services - 外部連線](./media/container-service-docker-swarm-setup-ci-cd/vsts-services-menu.png)

1. 在左側，按一下 [新增服務端點] > [GitHub]。

    ![Azure DevOps Services - GitHub](./media/container-service-docker-swarm-setup-ci-cd/vsts-github.png)

1. 若要授權讓 Azure DevOps Services 與您的 GitHub 帳戶搭配使用，請按一下 [授權]，並依照開啟視窗中的程序操作。

    ![Azure DevOps Services - 授權 GitHub](./media/container-service-docker-swarm-setup-ci-cd/vsts-github-authorize.png)

### <a name="connect-azure-devops-services-to-your-azure-container-registry-and-azure-container-service-cluster"></a>將 Azure DevOps Services 連線到您的 Azure 容器登錄和 Azure Container Service 叢集

進入 CI/CD 管線的最後一步是設定外部連線，來連線到您的容器登錄和您在 Azure 中的 Docker Swarm 叢集。 

1. 在您 Azure DevOps Services 專案的 [服務] 設定中，新增 [Docker 登錄] 類型的服務端點。 

1. 在開啟的快顯視窗中，輸入您的 Azure 容器登錄的 URL 和認證。

    ![Azure DevOps Services - Docker 登錄](./media/container-service-docker-swarm-setup-ci-cd/vsts-registry.png)

1. 針對 Docker Swarm 叢集新增 [SSH] 類型的端點。 然後輸入您的 Swarm 叢集的 SSH 連線資訊。

    ![Azure DevOps Services - SSH](./media/container-service-docker-swarm-setup-ci-cd/vsts-ssh.png)

已經完成所有設定。 在下一步中，您會建立建置應用程式並部署到 Docker Swarm 叢集的 CI/CD 管線。 

## <a name="step-2-create-the-build-pipeline"></a>步驟 2：建立組建管線

在此步驟中，您會設定 Azure DevOps Services 專案的組建管線，並定義容器映像的組建工作流程

### <a name="initial-pipeline-setup"></a>初始管線設定

1. 若要建立組建管線，請連線到您的 Azure DevOps Services 專案，然後按一下 [組建與發行]。 

1. 在 [組件定義] 區段中，按一下 [+ 新增]。 選取 [空白] 範本。

    ![Azure DevOps - 新增組建管線](./media/container-service-docker-swarm-setup-ci-cd/create-build-vsts.png)

1. 使用 GitHub 存放庫來源設定新組建，選取 [持續整合]，然後選取註冊 Linux 代理程式所在的代理程式佇列。 按一下 [建立] 以建立組建管線。

    ![Azure DevOps Services - 建立組建管線](./media/container-service-docker-swarm-setup-ci-cd/vsts-create-build-github.png)

1. 在 [組建定義] 頁面中，先開啟 [儲存機制] 索引標籤，並使用您在必要條件中建立的 MyShop 專案分叉來設定組建。 請確定您是選取 *acs-docs* 做為 [預設分支]。

    ![Azure DevOps Services - 組建存放庫組態](./media/container-service-docker-swarm-setup-ci-cd/vsts-github-repo-conf.png)

1. 在 [觸發程序] 索引標籤上，將組建設為在每次認可之後觸發。 選取 [持續整合] 和 [批次變更]。

    ![Azure DevOps Services - 組建觸發程序組態](./media/container-service-docker-swarm-setup-ci-cd/vsts-github-trigger-conf.png)

### <a name="define-the-build-workflow"></a>定義組建工作流程
接下來的步驟會定義組建工作流程。 要為 *MyShop* 應用程式建置五個容器映像。 每個映像都是使用專案資料夾中的 Dockerfile 來建置：

* ProductsApi
* Proxy
* RatingsApi
* RecommandationsApi
* ShopFront

您需要為每個映像新增兩個 [Docker] 步驟，一個是建置映像，另一個是將映像推送至 Azure 容器登錄。 

1. 若要在組建工作流程中新增步驟，按一下 [+ 加入建置步驟] 然後選取 [Docker]。

    ![Azure DevOps Services - 新增組建步驟](./media/container-service-docker-swarm-setup-ci-cd/vsts-build-add-task.png)

1. 針對每個映像，設定一個使用 `docker build` 命令的步驟。

    ![Azure DevOps Services - Docker 組建](./media/container-service-docker-swarm-setup-ci-cd/vsts-docker-build.png)

    針對建置作業，選取您的 Azure 容器登錄，選取 [Build an image] \(建置映像) 動作，以及定義每個映像的 Dockerfile。 將 [Build context] \(組件內容) 設為 Dockerfile 根目錄，並定義 [Image Name] \(映像名稱)。 
    
    如上一個畫面顯示，使用您 Azure 容器登錄的 URI 做為映像名稱的開頭。 (您也可以使用組建變數將映像的標籤參數化，就像此範例中的組建識別碼一樣。)

1. 針對每個映像，設定使用 `docker push` 命令的第二個步驟。

    ![Azure DevOps Services - Docker 推送](./media/container-service-docker-swarm-setup-ci-cd/vsts-docker-push.png)

    針對推送作業，選取您的 Azure 容器登錄，選取 **[Push an image]**  \(推送映像) 動作，並輸入在上一個步驟中建置的 **[Image Name]**  \(映像名稱)。

1. 針對這五個映像個別設定好建置和推送步驟之後，請在建置工作流程中再新增兩個步驟。

    a. 命令列工作會使用 bash 指令碼以目前的組建識別碼取代 docker-compose.yml 檔案中的 *BuildNumber*。請參閱以下畫面了解詳細資料。

    ![Azure DevOps Services - 更新 Compose 檔案](./media/container-service-docker-swarm-setup-ci-cd/vsts-build-replace-build-number.png)

    b. 一個工作，會將更新的 Compose 檔案卸除成組建構件，讓它能夠在版本中使用。 請參閱以下畫面了解詳細資料。

    ![Azure DevOps Services - 發佈 Compose 檔案](./media/container-service-docker-swarm-setup-ci-cd/vsts-publish-compose.png) 

1. 按一下 [儲存] 並命名您的組建管線。

## <a name="step-3-create-the-release-pipeline"></a>步驟 3︰建立發行管線

Azure DevOps Services 可讓您[跨環境管理發行](https://www.visualstudio.com/team-services/release-management/)。 您可以啟用持續部署以確保應用程式會順利部署到不同環境 (例如開發、測試、進入生產階段前和生產)。 您可以建立代表 Azure Container Service Docker Swarm 叢集的新環境。

![Azure DevOps Services - 發行至 ACS](./media/container-service-docker-swarm-setup-ci-cd/vsts-release-acs.png) 

### <a name="initial-release-setup"></a>初始發行設定

1. 若要建立發行管線，請按一下 [發行] > [+ 發行]

1. 若要設定構件來源，請按一下 [構件] > [連結構件來源]。 在此，請將這個新的發行管線連結到您在上一個步驟中定義的組建。 如此一來，就可以在發行程序中取得 docker-compose.yml 檔案。

    ![Azure DevOps Services - 發行成品](./media/container-service-docker-swarm-setup-ci-cd/vsts-release-artefacts.png) 

1. 若要設定發行觸發程序，請按一下 [觸發程序]，然後選取 [持續部署]。 在相同的構件來源上設定觸發程序。 此設定可確保一旦順利完成建置，就會立即開始新的發行。

    ![Azure DevOps Services - 發行觸發程序](./media/container-service-docker-swarm-setup-ci-cd/vsts-release-trigger.png) 

### <a name="define-the-release-workflow"></a>定義發行工作流程

發行工作流程由您新增的兩個工作組成。

1. 設定工作並使用您先前設定的 SSH 連線來安全地將 Compose 檔案複製到 Docker Swarm 主要節點上的 [deploy] 資料夾。 請參閱以下畫面了解詳細資料。

    ![Azure DevOps Services - 發行 SCP](./media/container-service-docker-swarm-setup-ci-cd/vsts-release-scp.png)

1. 設定第二個工作以在主要節點上執行 Bash 命令以執行 `docker` 和 `docker-compose` 命令。 請參閱以下畫面了解詳細資料。

    ![Azure DevOps Services - 發行 Bash](./media/container-service-docker-swarm-setup-ci-cd/vsts-release-bash.png)

    在主要節點上執行的命令會使用 Docker CLI 和 Docker-Compose CLI 執行下列工作：

    - 登入 Azure 容器登錄 (它使用 [變數] 索引標籤中定義的三個組建變數)
    - 定義 **DOCKER_HOST** 變數來與 Swarm 端點 (:2375) 搭配使用
    - 瀏覽到先前安全複製工作建立的 [deploy] 資料夾，其中包含 docker-compose.yml 檔案 
    - 執行提取新映像的 `docker-compose` 命令，停止服務並移除服務，然後建立容器。

    >[!IMPORTANT]
    > 如先前畫面所示，讓 [在 STDERR 上失敗] 核取方塊保持未選取。 這個設定很種要，因為 `docker-compose` 會在標準錯誤輸出上印出數個診斷訊息 (例如容器已停止或已刪除)。 如果您選取該核取方塊，即使一切正常運作，Azure DevOps Services 仍會報告發行期間出現錯誤。
    >
1. 儲存這個新的發行管線。


>[!NOTE]
>這個部署包含幾段停機時間，因為我們會停止舊服務並執行新服務。 執行藍綠部署可以避免此情況。
>

## <a name="step-4-test-the-cicd-pipeline"></a>步驟 4. 測試 CI/CD 管線

您已經完成設定，現在可以開始測試這個新的 CI/CD 管線。 最簡單的測試方法是更新原始程式碼，然後將變更認可到您的 GitHub 存放庫。 推送程式碼的數秒之後，您會看到新的組建在 Azure DevOps Services 中執行。 一旦順利完成，就會觸發新的發行，並且在 Azure Container Service 叢集中部署新版的應用程式。

## <a name="next-steps"></a>後續步驟

* 如需 CI/CD 與 Azure DevOps Services 的相關詳細資訊，請參閱 [Azure DevOps Services 組建概觀](https://www.visualstudio.com/docs/build/overview)。
