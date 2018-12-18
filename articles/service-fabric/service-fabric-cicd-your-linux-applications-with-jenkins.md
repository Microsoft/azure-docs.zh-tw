---
title: 使用 Jenkins 連續建置和整合 Azure Service Fabric Linux 應用程式 | Microsoft Docs
description: 使用 Jenkins 連續建置和整合 Service Fabric Linux 應用程式
services: service-fabric
documentationcenter: java
author: sayantancs
manager: jpconnock
ms.service: service-fabric
ms.devlang: java
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 07/31/2018
ms.author: saysa
ms.openlocfilehash: f381285d29d70d6f5da6a6cd319c682cd0c6a235
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39444533"
---
# <a name="use-jenkins-to-build-and-deploy-your-linux-applications"></a>使用 Jenkins 建置和部署您的 Linux 應用程式
Jenkins 是連續整合和部署應用程式的熱門工具。 以下是使用 Jenkins 建置和部署 Azure Service Fabric 應用程式的方式。

## <a name="topic-overview"></a>主題概觀
本文涵蓋幾個設定 Jenkins 環境的可能方式，以及在應用程式建置完成後部署到 Service Fabric 叢集的不同方式。 請依照這些一般步驟順利設定 Jenkins、從 GitHub 提取變更、建置應用程式，並將它部署到您的叢集：

1. 確定您安裝了[必要條件](#prerequisites)。
1. 然後依照這些小節其中之一提供的步驟設定 Jenkins：
   * [在 Service Fabric 叢集內設定 Jenkins](#set-up-jenkins-inside-a-service-fabric-cluster)、 
   * [在 Service Fabric 叢集外設定 Jenkins](#set-up-jenkins-outside-a-service-fabric-cluster)，或
   * [在現有的 Jenkins 環境中安裝 Service Fabric 外掛程式](#install-service-fabric-plugin-in-an-existing-jenkins-environment)。
1. 設定好 Jenkins 之後，請依照[建立及設定 Jenkins 工作](#create-and-configure-a-jenkins-job)中的步驟設定 GitHub，以在對應用程式進行變更時觸發 Jenkins，以及透過建置步驟設定 Jenkins 工作管線，以從 GitHub 提取變更並建置您的應用程式。 
1. 最後，設定 Jenkins 工作建置後步驟，將應用程式部署到您的 Service Fabric 叢集。 有兩種方式可以設定 Jenkins 將您的應用程式部署到叢集：    
   * 對於開發和測試環境，請使用[使用叢集管理觀點設定部署](#configure-deployment-using-cluster-management-endpoint)。 這是最簡單的設定部署方法。
   * 對於生產環境，請使用[使用 Azure 認證設定部署](#configure-deployment-using-azure-credentials)。 Microsoft 建議針對生產環境使用此方法，因為使用 Azure 認證，您就可以限制 Jenkins 工作對 Azure 資源的存取權。 

## <a name="prerequisites"></a>必要條件

- 確定已在本機安裝 Git。 您可以根據您的作業系統，從 [Git 下載頁面](https://git-scm.com/downloads)安裝適當的 Git 版本。 如果您不熟悉 Git，請從 [Git 文件](https://git-scm.com/docs)深入了解。
- 本文針對要建置和部署的應用程式，使用 GitHub 的 *Service Fabric 使用者入門範例*：[https://github.com/Azure-Samples/service-fabric-java-getting-started](https://github.com/Azure-Samples/service-fabric-java-getting-started)。 您可以為此儲存機制建立分支來據以進行作業，或對指示稍微做一些修改，以使用您自己的 GitHub 專案。


## <a name="install-service-fabric-plugin-in-an-existing-jenkins-environment"></a>在現有的 Jenkins 環境中安裝 Service Fabric 外掛程式
如果您是將 Service Fabric 外掛程式新增到現有的 Jenkins 環境，需要下列項目：

- [Service Fabric CLI](service-fabric-cli.md) (sfctl)。

   > [!NOTE]
   > 確定在系統層級 (而不是在使用者層級) 安裝 CLI，Jenkins 才能執行 CLI 命令。 
   >

- 若要部署 Java 應用程式，請安裝 [Gradle 和 Open JDK 8.0](service-fabric-get-started-linux.md#set-up-java-development)。 
- 若要部署 .NET Core 2.0 應用程式，請安裝 [.NET Core 2.0 SDK](service-fabric-get-started-linux.md#set-up-net-core-20-development)。 

在您已經安裝環境所需的必要條件之後，便可以在 Jenkins 市集中搜尋 Azure Service Fabric 外掛程式並安裝。

在您安裝外掛程式之後，略過至[建立及設定 Jenkins 工作](#create-and-configure-a-jenkins-job)。


## <a name="set-up-jenkins-inside-a-service-fabric-cluster"></a>在 Service Fabric 叢集內設定 Jenkins

您可以在 Service Fabric 叢集內部或外部設定 Jenkins。 下列小節說明如何在叢集內設定它，同時又使用 Azure 儲存體帳戶來儲存容器執行個體的狀態。

### <a name="prerequisites"></a>必要條件
- 在 Service Fabric Linux 叢集中安裝 Docker。 Azure 中執行的 Service Fabric 叢集已經安裝 Azure。 如果您在本機執行叢集 (OneBox dev environment)，請檢查 Docker 是否已經使用 `docker info` 命令在電腦上安裝。 如果未安裝，請使用以下命令安裝：

   ```sh
   sudo apt-get install wget
   wget -qO- https://get.docker.io/ | sh
   ``` 

   > [!NOTE]
   > 請確定您已將 8081 連接埠指定為叢集上的自訂端點。 如果您使用本機叢集，請確定主機電腦上的連接埠 8081 已經開啟，且擁有公開的 IP 位址。
   >

### <a name="steps"></a>步驟
1. 使用以下命令複製應用程式：
   ```sh
   git clone https://github.com/suhuruli/jenkins-container-application.git
   cd jenkins-container-application
   ```

1. 在檔案共用中保存 Jenkins 容器的狀態：
   1. 以 `sfjenkinsstorage1` 之類的名稱，在和叢集**相同的區域**中建立 Azure 儲存體帳戶。
   1. 以 `sfjenkins` 之類的名稱，在儲存體帳戶下建立**檔案共用**。
   1. 針對該檔案共用按一下 [連接]，並記下 [正在從 Linux 連線] 底下顯示的值，該值看起來應該如下所示：

      ```sh
      sudo mount -t cifs //sfjenkinsstorage1.file.core.windows.net/sfjenkins [mount point] -o vers=3.0,username=sfjenkinsstorage1,password=<storage_key>,dir_mode=0777,file_mode=0777
      ```

   > [!NOTE]
   > 若要掛接 cifs 共用，您需要在叢集節點中安裝 cifs Utils 套件。      
   >

1. 將 `setupentrypoint.sh` 指令碼中的預留位置值更新為從步驟 2 取得的 Azure 儲存體詳細資料。
   ```sh
   vi JenkinsSF/JenkinsOnSF/Code/setupentrypoint.sh
   ```
   * 以來自上述步驟 2 中連線輸出的 `//sfjenkinsstorage1.file.core.windows.net/sfjenkins` 值取代 `[REMOTE_FILE_SHARE_LOCATION]`。
   * 以來自上述步驟 2 的 `vers=3.0,username=sfjenkinsstorage1,password=GB2NPUCQY9LDGeG9Bci5dJV91T6SrA7OxrYBUsFHyueR62viMrC6NIzyQLCKNz0o7pepGfGY+vTa9gxzEtfZHw==,dir_mode=0777,file_mode=0777` 值取代 `[FILE_SHARE_CONNECT_OPTIONS_STRING]`。

1. **僅限安全叢集：** 
   
   為了從 Jenkins 設定應用程式在安全叢集上的部署，叢集憑證必須可在 Jenkins 容器內進行存取。 在 **ContainerHostPolicies** 標記下的 *ApplicationManifest.xml* 檔案中，新增此憑證參考並使用叢集憑證的指紋值更新指紋值。

   ```xml
   <CertificateRef Name="MyCert" X509FindValue="[Thumbprint]"/>
   ```

   此外，將下列幾行新增到 *ApplicationManifest.xml* 檔案中的 **ApplicationManifest** (root) 標記底下，並使用叢集憑證的指紋值更新指紋值。

   ```xml
   <Certificates>
     <SecretsCertificate X509FindType="FindByThumbprint" X509FindValue="[Thumbprint]" />
   </Certificates> 
   ```

1. 連線至叢集並安裝容器應用程式。

   **安全叢集**
   ```sh
   sfctl cluster select --endpoint https://PublicIPorFQDN:19080  --pem [Pem] --no-verify # cluster connect command
   bash Scripts/install.sh
   ```
   上述命令採用 PEM 格式的憑證。 如果您的憑證格式為 PFX，可以使用以下命令轉換格式。 如果您的 PFX 檔案未受到密碼保護，請將 **passin** 參數指定為 `-passin pass:`。
   ```sh
   openssl pkcs12 -in cert.pfx -out cert.pem -nodes -passin pass:MyPassword1234!
   ``` 
   
   **不安全叢集**
   ```sh
   sfctl cluster select --endpoint http://PublicIPorFQDN:19080 # cluster connect command
   bash Scripts/install.sh
   ```

   這會在叢集上安裝 Jenkins 容器，您可以使用 Service Fabric Explorer 加以監視。

   > [!NOTE]
   > Jenkins 映像可能需要幾分鐘的時間才能下載到叢集上。
   >

1. 從瀏覽器中，前往 `http://PublicIPorFQDN:8081`。 它會提供登入所需之初始管理密碼的路徑。 
1. 查看 Service Fabric Explorer，以確定 Jenkins 容器是在哪個節點上執行。 透過安全殼層 (SSH) 登入此節點。
   ```sh
   ssh user@PublicIPorFQDN -p [port]
   ``` 
1. 使用 `docker ps -a` 取得容器執行個體識別碼。
1. 以 Secure Shell (SSH) 登入容器，並貼上您在 Jenkins 入口網站中看到的路徑。 例如，如果在入口網站中它顯示路徑 `PATH_TO_INITIAL_ADMIN_PASSWORD`，執行下列命令︰

   ```sh
   docker exec -t -i [first-four-digits-of-container-ID] /bin/bash   # This takes you inside Docker shell
   ```
   ```sh
   cat PATH_TO_INITIAL_ADMIN_PASSWORD # This displays the password value
   ```
1. 在 [Jenkins 入門] 頁面上，選擇 [選取要安裝的外掛程式] 選項，選取 [無] 核取方塊，然後按一下 [安裝]。
1. 建立使用者，或選擇繼續使用系統管理員身分。

在您設定 Jenkins 之後，略過至[建立及設定 Jenkins 工作](#create-and-configure-a-jenkins-job)。  

## <a name="set-up-jenkins-outside-a-service-fabric-cluster"></a>在 Service Fabric 叢集外設定 Jenkins

您可以在 Service Fabric 叢集內部或外部設定 Jenkins。 下列各節顯示如何在叢集外設定。

### <a name="prerequisites"></a>必要條件
- 確定您的電腦上已安裝 Docker。 使用下列命令可從終端機安裝 Docker︰

  ```sh
  sudo apt-get install wget
  wget -qO- https://get.docker.io/ | sh
  ```

  現在，當您在終端機執行 `docker info`，應該會在輸出中看到 Docker 服務正在執行。

### <a name="steps"></a>步驟
1. 提取 Service Fabric Jenkins 容器映像︰`docker pull rapatchi/jenkins:latest`。 此映像隨附於預先安裝的 Service Fabric Jenkins 外掛程式。
1. 執行容器映像︰`docker run -itd -p 8080:8080 rapatchi/jenkins:latest`
1. 取得容器映像執行個體的識別碼。 您可以使用命令 `docker ps –a` 列出所有 Docker 容器
1. 使用下列步驟登入 Jenkins 入口網站︰

   1. 從您的主機登入 Jenkins 殼層。 使用容器 ID 的前四個數字。 例如，如果容器 ID 為 `2d24a73b5964`, use `2d24`。

      ```sh
      docker exec -it [first-four-digits-of-container-ID] /bin/bash
      ```
   1. 從 Jenkins 殼層，取得容器執行個體的管理密碼：

      ```sh
      cat /var/jenkins_home/secrets/initialAdminPassword
      ```      
   1. 若要登入 Jenkins 儀表板，請在網頁瀏覽器中開啟以下 URL：`http://<HOST-IP>:8080`。 使用之前步驟的密碼將 Jenkins 解除鎖定。
   1. (選擇性。)第一次登入之後，您可以建立您自己的使用者帳戶，並針對下列步驟使用該帳戶，或者您可以繼續使用系統管理員帳戶。 如果您建立使用者，就需要繼續使用該使用者。
1. 設定 GitHub 以 Jenkins，方法為使用[產生新的 SSH 金鑰，並將它新增至 SSH 代理程式](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)中的步驟。
   * 使用 GitHub 所提供的指示產生 SSH 金鑰，並將 SSH 金鑰新增至裝載存放庫的 GitHub 帳戶。
   * 在 Jenkins Docker 殼層 (而非在主機上) 執行上述連結內所提到的命令。
   * 若要從主機登入 Jenkins 殼層，請使用下列命令：

      ```sh
      docker exec -t -i [first-four-digits-of-container-ID] /bin/bash
      ```

請確定 Jenkins 容器映像所裝載的叢集或電腦具有公開 IP 位址。 這可讓 Jenkins 執行個體接收來自 GitHub 的通知。

在您設定 Jenkins 之後，繼續下一個小節，[建立及設定 Jenkins 工作](#create-and-configure-a-jenkins-job)。

## <a name="create-and-configure-a-jenkins-job"></a>建立及設定 Jenkins 作業

本節中的步驟會說明如何設定 Jenkins 工作，以回應 GitHub 儲存機制中的變更，擷取變更，然後建置它們。 在本小節結尾處，會將您導向至設定工作的最終步驟，以依據您是要部署至開發/測試環境，或部署至生產環境來部署應用程式。 

1. 在 Jenkins 儀表板中，按一下 [新增項目]。
1. 輸入項目名稱 (例如，**MyJob**)。 選取 [自由樣式專案]，然後按一下 [確定]。
1. 隨即開啟 [工作組態] 頁面。 (若要從 Jenkins 儀表板前往組態，請按一下工作，然後按一下 [設定])。

1. 在 [一般] 索引標籤中，選取 [GitHub 專案] 的核取方塊，然後指定您的 GitHub 專案 URL。 此 URL 會裝載您想要與 Jenkins 連續整合、連續部署 (CI/CD) 流程整合的 Service Fabric Java 應用程式 (例如，`https://github.com/{your-github-account}/service-fabric-java-getting-started`)。

1. 在 [原始程式碼管理] 索引標籤中，選取 [Git]。 指定存放庫 URL，它會裝載您想要與 Jenkins CI/CD 流程整合的 Service Fabric Java 應用程式 (例如，`https://github.com/{your-github-account}/service-fabric-java-getting-started`)。 您也可以指定要建置一個分支 (例如，`/master`)。
1. 將您的 *GitHub* 儲存機制設定為與 Jenkins 通訊：

   a. 在您的 GitHub 儲存機制頁面中，前往 [設定] > [整合和服務]。

   b. 選取 [新增服務]、輸入 **Jenkins**，然後選取 [Jenkins-Github 外掛程式]。

   c. 輸入您的 Jenkins webhook URL (根據預設，它應該是 `http://<PublicIPorFQDN>:8081/github-webhook/`)。 按一下 [新增/更新服務]。

   d. 隨即會將測試事件傳送至您的 Jenkins 執行個體。 您應該會在 GitHub 中看到 webhook 所做的綠色勾號，而且您的專案將會建置。

1. 在 Jenkins 的 [建置觸發程序] 索引標籤中，選取您想要的建置選項。 對於此範例，您想要在推入至儲存機制時觸發組建，因此請選取 [GITScm 輪詢的 GitHub 勾點觸發程序]。 (在先前，此選項稱為**當變更推送至 GitHub 時建置**。)
1. 在 [建置] 索引標籤上，視您要建置 Java 應用程式或 .NET Core 應用程式，執行下列其中一個動作：

   * **對於 Java 應用程式：** 從 [新增建置步驟] 下拉式清單中，選取 [呼叫 Gradle 指令碼]。 按一下 [進階] 。 在進階功能表，針對您的應用程式指定 [根建置指令碼] 的路徑。 它會從指定的路徑挑選 build.gradle，並據以運作。 對於 [ActorCounter 應用程式](https://github.com/Azure-Samples/service-fabric-java-getting-started/tree/master/reliable-services-actor-sample/Actors/ActorCounter)，這是：`${WORKSPACE}/reliable-services-actor-sample/Actors/ActorCounter`.

     ![Service Fabric Jenkins 建置動作][build-step]

   * **對於 .NET Core 應用程式：** 從 [新增建置步驟] 下拉式清單中，選取 [執行殼層]。 在出現的命令方塊中，首先需要將目錄變更為 build.sh 檔案所在的路徑。 一旦目錄變更，就可以執行 build.sh 指令碼，並建置應用程式。

      ```sh
      cd /var/jenkins_home/workspace/[Job Name]/[Path to build.sh]  # change directory to location of build.sh file
      ./build.sh
      ```

     以下螢幕擷取畫面顯示使用 Jenkins 作業名稱 CounterServiceApplication 建置 [Counter Service](https://github.com/Azure-Samples/service-fabric-dotnet-core-getting-started/tree/master/Services/CounterService) 範例的命令範例。

      ![Service Fabric Jenkins 建置動作][build-step-dotnet]

1. 若要設定 Jenkins 在建置後動作中將您的應用程式部署到 Service Fabric 叢集，您需要該叢集憑證在 Jenkins 容器中的位置。 依據您的 Jenkins 容器是在叢集內部或外部執行選擇下列其中一項，並記下叢集憑證的位置：

   * **對於在叢集內執行的 Jenkins：** 您可在容器內回應 *Certificates_JenkinsOnSF_Code_MyCert_PEM* 環境變數的值，找到憑證的路徑。

      ```sh
      echo $Certificates_JenkinsOnSF_Code_MyCert_PEM
      ```
   
   * **對於在叢集外執行的 Jenkins：** 請依照這些步驟將叢集憑證複製到您的容器：
      1. 憑證必須是 PEM 格式。 如果您沒有 PEM 檔案，可以從憑證 PFX 檔案中建立一個。 如果您的 PFX 檔案未受密碼保護，請從您的主機執行以下命令：

         ```sh
         openssl pkcs12 -in clustercert.pfx -out clustercert.pem -nodes -passin pass:
         ``` 

      如果 PFX 檔案受密碼保護，請在 `-passin` 參數中包括密碼。 例如︰

         ```sh
         openssl pkcs12 -in clustercert.pfx -out clustercert.pem -nodes -passin pass:MyPassword1234!
         ``` 

      1. 若要為 Jenkins 容器取得容器 ID，請從主機執行 `docker ps`。
      1. 使用以下 Docker 命令將 PEM 檔案複製到您的容器：
    
         ```sh
         docker cp clustercert.pem [first-four-digits-of-container-ID]:/var/jenkins_home
         ``` 

就快要完成了！ 讓 Jenkins 作業維持開啟。 最後一項工作是，設定建置後步驟，將應用程式部署到您的 Service Fabric 叢集：

* 若要部署到開發或測試環境，請依照[使用叢集管理端點設定部署](#configure-deployment-using-cluster-management-endpoint)中的步驟進行。
* 若要部署到生產環境，請依照[使用 Azure 認證來設定部署](#configure-deployment-using-azure-credentials)中的步驟進行。

## <a name="configure-deployment-using-cluster-management-endpoint"></a>使用叢集管理端點設定部署
對於開發和測試環境，您可以使用叢集管理端點部署應用程式。 使用叢集管理端點設定建置後動作部署應用程式時，需要進行最少的設定。 如果您是要部署至生產環境，請略過至[使用 Azure 認證設定部署](#configure-deployment-using-azure-credentials)，以設定 Azure Active Directory 服務主體，以在部署期間使用。    

1. 在 Jenkins 作業中，按一下 [建置後動作] 索引標籤。 
1. 從 [建置後動作] 下拉式清單，選取 [部署 Service Fabric 專案]。 
1. 在 [Service Fabric 叢集組態] 底下，選取 [填入 Service Fabric 管理端點] 選項按鈕。
1. 對於 [管理主機]，請輸入叢集的連線端點；例如 `{your-cluster}.eastus.cloudapp.azure.com`。
1. 對於 [用戶端金鑰] 和 [用戶端憑證]，請輸入您 Jenkins 容器中 PEM 檔案的位置；例如 `/var/jenkins_home/clustercert.pem`。 (您已經在[建立及設定 Jenkins 作業](#create-and-configure-a-jenkins-job)的最後一個步驟複製憑證位置。)
1. 在 [應用程式組態] 底下，設定 [應用程式名稱]、[應用程式類型]，和 (相關) [應用程式資訊清單路徑] 欄位。

   ![Service Fabric Jenkins 建置後動作設定管理端點](./media/service-fabric-cicd-your-linux-application-with-jenkins/post-build-endpoint.png)

1. 按一下 [驗證組態 ]。 在驗證成功之後，按一下 [儲存]。 您的 Jenkins 作業管線現在已完整設定。 略過至 [後續步驟](#next-steps) 以測試部署。

## <a name="configure-deployment-using-azure-credentials"></a>使用 Azure 認證設定部署
對於生產環境，強烈建議您設定 Azure 認證以部署應用程式。 本節會說明如何設定 Azure Active Directory 服務主體，以用來在建置後動作中部署應用程式。 您可以指派服務主體給目錄中的角色，以限制 Jenkins 作業的權限。 

對於部署和測試環境，您可以設定 Azure 認證或叢集管理端點以部署應用程式。 如需有關如何設定叢集管理端點的詳細資料，請參閱[使用叢集管理端點設定部署](#configure-deployment-using-cluster-management-endpoint)。   

1. 若要建立 Azure Active Directory 服務主體，並在 Azure 訂用帳戶中指派權限給它，請依照[使用入口網站建立 Azure Active Directory 應用程式和服務主體](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal)中的步驟進行。 請注意以下事項：

   * 依照主題中的步驟進行時，請務必複製並儲存下列值：*應用程式 ID*、*應用程式金鑰*、*目錄 ID (租用戶 ID)*，和*訂用帳戶 ID*。 您需要它們才能在 Jenkins 中設定 Azure 認證。
   * 如果您沒有目錄的[必要權限](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) \(英文\)，將需要要求系統管理員授予權限，或為您建立服務主體，或者您需要針對 Jenkins 中的工作，在 [建置後動作] 中為叢集設定管理端點。
   * 在[建立 Azure Active Directory 應用程式](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal#create-an-azure-active-directory-application)[登入 URL] 輸入任何格式正確的 URL。
   * 在[指派應用程式給角色](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal#assign-application-to-role) 一節中，您可以將叢集資源群組的 *Reader* 角色指派給應用程式。

1. 回到 Jenkins 作業，按一下 [建置後動作] 索引標籤。
1. 從 [建置後動作] 下拉式清單，選取 [部署 Service Fabric 專案]。 
1. 在 [Service Fabric 叢集組態] 底下，選取 [選取 Service Fabric 叢集] 選項按鈕。 按下 [Azure 認證] 旁邊的 [新增]。 按一下 [Jenkins] 選取 Jenkins 認證提供者。
1. 在 Jenkins 認證提供者中，從 [種類] 下拉式清單中，選取 [Microsoft Azure 服務主體]。
1. 使用您在步驟 1 設定服務主體時儲存的值設定下列欄位：

   * [用戶端識別碼]：[應用程式識別碼]
   * [用戶端祕密]：[應用程式金鑰]
   * [租用戶識別碼]：[目錄識別碼]
   * [訂用帳戶識別碼]：[訂用帳戶識別碼]
1. 輸入您用來在 Jenkins 中選取認證的描述性 [識別碼]，和簡短的 [描述]。 然後按一下 [驗證服務主體]。 如果驗證成功，請按一下 [新增]。

   ![Service Fabric Jenkins 輸入 Azure 認證](./media/service-fabric-cicd-your-linux-application-with-jenkins/enter-azure-credentials.png)
1. 回到 [Service Fabric 叢集組態] 底下，確認您已經針對 [Azure 認證] 選取新的認證。 
1. 從 [資源群組] 下拉式清單中，針對您要部署應用程式的叢集，選取叢集的資源群組。
1. 從 [Service Fabric] 下拉式清單，選取您要部署應用程式的叢集。
1. 對於 [用戶端金鑰] 和 [用戶端憑證]，請輸入您 Jenkins 容器中 PEM 檔案的位置。 例如 `/var/jenkins_home/clustercert.pem` 。 
1. 在 [應用程式組態] 底下，設定 [應用程式名稱]、[應用程式類型]，和 (相關) [應用程式資訊清單路徑] 欄位。
    ![Service Fabric Jenkins 建置後動作設定 Azure 認證](./media/service-fabric-cicd-your-linux-application-with-jenkins/post-build-credentials.png)
1. 按一下 [驗證組態 ]。 在驗證成功之後，按一下 [儲存]。 您的 Jenkins 作業管線現在已完整設定。 繼續至 [後續步驟](#next-steps) 以測試部署。

## <a name="troubleshooting-the-jenkins-plugin"></a>對 Jenkins 外掛程式進行疑難排解

如果您遇到任何有關 Jenkins 外掛程式的錯誤，請在 [Jenkins JIRA](https://issues.jenkins-ci.org/) 的特定元件中提交問題。

## <a name="next-steps"></a>後續步驟
現在已設定 GitHub 和 Jenkins。 請考慮在您儲存機制分支中的 `reliable-services-actor-sample/Actors/ActorCounter` 專案中 https://github.com/Azure-Samples/service-fabric-java-getting-started \(英文\) 中進行一些範例變更。 將您的變更推送至遠端 `master` 分支 (或您已經設定使用的任何分支)。 這會觸發您設定的 Jenkins 作業 `MyJob`。 它會從 GitHub 擷取變更、建置它們並且將應用程式部署至您在建置後動作中指定的叢集。  

  <!-- Images -->
  [build-step]: ./media/service-fabric-cicd-your-linux-application-with-jenkins/build-step.png
  [build-step-dotnet]: ./media/service-fabric-cicd-your-linux-application-with-jenkins/build-step-dotnet.png

