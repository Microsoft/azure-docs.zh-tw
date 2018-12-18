---
title: 在 Azure 中的 Service Fabric 上為 Java 應用程式設定 Jenkins | Microsoft Docs
description: 在本教學課程中，了解如何使用 Jenkins 設定持續整合以部署 Java Service Fabric 應用程式。
services: service-fabric
documentationcenter: java
author: suhuruli
manager: msfussell
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: java
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/27/2018
ms.author: suhuruli
ms.custom: mvc
ms.openlocfilehash: b8c114ec3fe9b27d0318bf11a8b1fa8e3ce5f1c3
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43124975"
---
# <a name="tutorial-configure-a-jenkins-environment-to-enable-cicd-for-a-java-application-on-service-fabric"></a>教學課程︰設定 Jenkins 環境以便在 Service Fabric 上啟用 Java 應用程式的 CI/CD

本教學課程是一個系列的第五部分。 它會示範如何使用 Jenkins 將升級部署至您的應用程式。 在此教學課程中，Service Fabric Jenkins 外掛程式會搭配裝載 Voting 應用程式的 Github 存放庫使用，以將應用程式部署至叢集。

在本系列的第五部分中，您了解如何：
> [!div class="checklist"]
> * 在您的電腦上部署 Service Fabric Jenkins 容器
> * 設定 Jenkins 環境以便部署至 Service Fabric
> * 將您的應用程式升級

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * [建置 Java Service Fabric Reliable Services 應用程式](service-fabric-tutorial-create-java-app.md)
> * [在本機叢集上部署及偵錯應用程式](service-fabric-tutorial-debug-log-local-cluster.md)
> * [將應用程式部署至 Azure 叢集](service-fabric-tutorial-java-deploy-azure.md)
> * [設定應用程式的監視和診斷](service-fabric-tutorial-java-elk.md)
> * 設定 CI/CD

## <a name="prerequisites"></a>必要條件

* 從 [ Git 下載頁面](https://git-scm.com/downloads)在您的本機電腦上安裝 Git。 如需 Git 的詳細資訊，請參閱 [Git 文件](https://git-scm.com/docs)。
* 具備 [Jenkins](https://jenkins.io/) 的運用知識。
* 建立 [GitHub](https://github.com/) 帳戶並了解如何使用 GitHub。
* 在您的電腦上安裝 [Docker](https://www.docker.com/community-edition)。

## <a name="pull-and-deploy-service-fabric-jenkins-container-image"></a>提取並部署 Service Fabric Jenkins 容器映像

您可以在 Service Fabric 叢集內部或外部設定 Jenkins。 下列指示說明如何使用所提供的 Docker 印象在叢集外進行設定。 不過，也可以使用預先設定的 Jenkins 組建環境。 下列容器映像會隨著 Service Fabric 外掛程式一起安裝，並已準備好立即搭配 Service Fabric 使用。

1. 提取 Service Fabric Jenkins 容器映像︰``docker pull rapatchi/jenkins:v10``。 此映像隨附於預先安裝的 Service Fabric Jenkins 外掛程式。

1. 使用您的 Azure 憑證在已掛接的本機電腦上儲存所在的位置執行容器映像。

    ```bash
    docker run -itd -p 8080:8080 -v /service-fabric-java-quickstart/AzureCluster rapatchi/jenkins:v10
    ```

1. 取得容器映像執行個體的識別碼。 您可以使用命令 ``docker ps –a`` 列出所有 Docker 容器

1. 執行下列命令，擷取 Jenkins 執行個體的密碼：

    ```sh
    docker exec [first-four-digits-of-container-ID] cat /var/jenkins_home/secrets/initialAdminPassword
    ```

    如果容器識別碼是 2d24a73b5964，請使用 2d24。
    * 從入口網站 (``http://<HOST-IP>:8080``) 登入 Jenkins 儀表板需要此密碼
    * 第一次登入之後，您可以建立自己的使用者帳戶，或使用系統管理員帳戶。

1. 設定 GitHub 以 Jenkins，方法為使用[產生新的 SSH 金鑰，並將它新增至 SSH 代理程式](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)中提到的步驟。 因為命令是從 Docker 容器執行，所以請依照 Linux 環境的指示操作。
    * 使用 GitHub 所提供的指示來產生 SSH 金鑰。 接下來，將 SSH 金鑰新增至裝載存放庫的 GitHub 帳戶。
    * 在 Jenkins Docker 殼層 (而非在主機上) 執行上述連結內所提到的命令。
    * 若要從主機登入 Jenkins 殼層，請使用下列命令：

    ```sh
    docker exec -t -i [first-four-digits-of-container-ID] /bin/bash
    ```

    請確定 Jenkins 容器映像所裝載的叢集或電腦具有公開 IP。 具有公開 IP，可讓 Jenkins 執行個體接收來自 GitHub 的通知。

## <a name="create-and-configure-a-jenkins-job"></a>建立及設定 Jenkins 作業

1. 首先，如果您沒有可用於在 Github 上裝載 Voting 專案的存放庫，請建立一個。 在本教學課程的其餘部分中，存放庫稱為 **dev_test**。

1. 在 Jenkins 儀表板上建立**新項目**，網址是 ``http://<HOST-IP>:8080``。

1. 輸入項目名稱 (例如，**MyJob**)。 選取 [自由樣式專案]，然後按一下 [確定]。

1. 前往 [作業] 頁面，並按一下 [設定]。

   a. 在 [一般] 區段中，選取 [GitHub 專案] 的核取方塊，然後指定您的 GitHub 專案 URL。 此 URL 會裝載您想要與 Jenkins 連續整合、連續部署 (CI/CD) 流程整合的 Service Fabric Java 應用程式 (例如，``https://github.com/testaccount/dev_test``)。

   b. 在 [原始程式碼管理] 區段底下，選取 [Git]。 指定存放庫 URL，它會裝載您想要與 Jenkins CI/CD 流程整合的 Service Fabric Java 應用程式 (例如，*https://github.com/testaccount/dev_test.git*)。 您也可以在這裡指定要建置哪些分支 (例如，**/master**)。

1. 設定您的 *GitHub* (裝載存放庫者)，讓它能夠與 Jenkins 溝通。 請使用下列步驟：

   a. 移至您的 GitHub 儲存機制頁面。 移至 [設定] > [整合和服務]。

   b. 選取 [新增服務]、輸入 **Jenkins**，然後選取 [Jenkins-Github 外掛程式]。

   c. 輸入您的 Jenkins webhook URL (根據預設，它應該是 ``http://<PublicIPorFQDN>:8081/github-webhook/``)。 按一下 [新增/更新服務]。

   d. 隨即會將測試事件傳送至您的 Jenkins 執行個體。 您應該會在 GitHub 中看到 Webhook 旁邊有綠色勾號，以及您的專案組建。

   ![Service Fabric Jenkins 組態](./media/service-fabric-tutorial-java-jenkins/jenkinsconfiguration.png)

1. 在 [組建觸發程序] 區段下，選取您想要的建置選項。 針對此範例，您要每當發生某些推送至存放庫時觸發組建。 因此，您選取 [GITScm 輪詢的 GitHub 攔截觸發程序]。

1. 在 [建置] 區段底下，從下拉式清單 [新增建置步驟]，選取 [叫用 Gradle 指令碼] 選項。 在出現的小工具中開啟進階功能表，針對您的應用程式指定 [根建置指令碼] 的路徑。 它會從指定的路徑挑選 build.gradle，並據以運作。

    ![Service Fabric Jenkins 建置動作](./media/service-fabric-tutorial-java-jenkins/jenkinsbuildscreenshot.png)

1. 從 [建置後動作] 下拉式清單，選取 [部署 Service Fabric 專案]。 這裡您必須提供叢集詳細資料，當中會部署 Jenkins 編譯 Service Fabric 應用程式。 憑證的路徑是掛接磁碟區的位置 (/tmp/myCerts)。

    您也可以提供用來部署應用程式的其他詳細資料。 請參閱下列螢幕擷取畫面，以取得應用程式詳細資料的範例︰

    ![Service Fabric Jenkins 建置動作](./media/service-fabric-tutorial-java-jenkins/sfjenkins.png)

    > [!NOTE]
    > 如果您使用 Service Fabric 來部署 Jenkins 容器映像，這裡的叢集可能與裝載 Jenkins 容器應用程式的叢集相同。
    >

1. 按一下 [檔案] 。

## <a name="update-your-existing-application"></a>更新現有的應用程式

1. 使用 **Service Fabric Voting 範例 V2** 更新 VotingApplication/VotingWebPkg/Code/wwwroot/index.html 檔案中的 HTML 標題。

    ```html
    <div ng-app="VotingApp" ng-controller="VotingAppController" ng-init="refresh()">
        <div class="container-fluid">
            <div class="row">
                <div class="col-xs-8 col-xs-offset-2 text-center">
                    <h2>Service Fabric Voting Sample V2</h2>
                </div>
            </div>
        </div>
    </div>
    ```

1. 在 Voting/VotingApplication/ApplicationManifest.xml 檔案中將 **ApplicationTypeVersion** 和 **ServiceManifestVersion** 版本更新為 **2.0.0**。

    ```xml
    <?xml version="1.0" encoding="utf-8" standalone="no"?>
    <ApplicationManifest xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VotingApplicationType" ApplicationTypeVersion="2.0.0">
      <Description>Voting Application</Description>
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="VotingWebPkg" ServiceManifestVersion="2.0.0"/>
      </ServiceManifestImport>
      <ServiceManifestImport>
            <ServiceManifestRef ServiceManifestName="VotingDataServicePkg" ServiceManifestVersion="1.0.0"/>
        </ServiceManifestImport>
        <DefaultServices>
          <Service Name="VotingWeb">
             <StatelessService InstanceCount="1" ServiceTypeName="VotingWebType">
                <SingletonPartition/>
             </StatelessService>
          </Service>
       <Service Name="VotingDataService">
                <StatefulService MinReplicaSetSize="3" ServiceTypeName="VotingDataServiceType" TargetReplicaSetSize="3">
                    <UniformInt64Partition HighKey="9223372036854775807" LowKey="-9223372036854775808" PartitionCount="1"/>
                </StatefulService>
            </Service>
        </DefaultServices>
    </ApplicationManifest>
    ```

1. 在 Voting/VotingApplication/VotingWebPkg/ServiceManifest.xml 檔案中將 **ServiceManifest** 中的 **Version** 欄位和 **CodePackage** 標記中的 **Version** 欄位更新為 **2.0.0**。

    ```xml
    <CodePackage Name="Code" Version="2.0.0">
    <EntryPoint>
        <ExeHost>
        <Program>entryPoint.sh</Program>
        </ExeHost>
    </EntryPoint>
    </CodePackage>
    ```

1. 若要初始化可執行應用程式升級的 Jenkins 作業，請將您的新變更推送至您的 Github 存放庫。

1. 在 Service Fabric 總管中，按一下 [應用程式] 下拉式清單。 若要查看您的升級狀態，請按一下 [進行中的升級] 索引標籤。

    ![進行中的升級](./media/service-fabric-tutorial-create-java-app/upgradejava.png)

1. 如果您存取 **http://\<Host-IP>:8080**，現在會啟動並執行完整功能的 Voting 應用程式。

    ![本機 Voting 應用程式](./media/service-fabric-tutorial-java-jenkins/votingv2.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 在您的電腦上部署 Service Fabric Jenkins 容器
> * 設定 Jenkins 環境以便部署至 Service Fabric
> * 將您的應用程式升級

* 簽出其他 [Java 範例](https://github.com/Azure-Samples/service-fabric-java-getting-started)