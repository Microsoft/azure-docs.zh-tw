---
title: 在 Linux 上建立適用於 Apache Tomcat 伺服器的 Azure Service Fabric 容器 | Microsoft Docs
description: 建立 Linux 容器以公開在 Azure Service Fabric 上於 Apache Tomcat 伺服器上執行的應用程式。 以您的應用程式和 Apache Tomcat 伺服器建置 Docker 映像，將該映像推送到容器登錄，建置並部署 Service Fabric 容器應用程式。
services: service-fabric
documentationcenter: .net
author: JimacoMS2
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 6/08/2018
ms.author: v-jamebr
ms.openlocfilehash: 29208bcbdbe6ad01d0e1ac7343bd921f3287260a
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39580250"
---
# <a name="create-service-fabric-container-running-apache-tomcat-server-on-linux"></a>在 Linux 上建立執行 Apache Tomcat 伺服器的 Service Fabric 容器
Apache Tomcat 是一個熱門且開放原始碼的 Java Servlet 和 Java Server 技術實作。 此文章說明如何搭配 Apache Tomcat 和簡單的 Web 應用程式來建置容器，將該容器部署至執行 Linux 的 Service Fabric 叢集，然後連線至 Web 應用程式。  

若要深入了解 Apache Tomcat，請參閱 [Apache Tomcat 首頁](http://tomcat.apache.org/) \(英文\)。 

## <a name="prerequisites"></a>必要條件
* 執行下列項目的開發電腦︰
  * [Service Fabric SDK 和工具](service-fabric-get-started-linux.md)。
  * [Docker CE for Linux](https://docs.docker.com/engine/installation/#prior-releases). 
  * [Service Fabric CLI](service-fabric-cli.md)

* 位於 Azure Container Registry 中的容器登錄。 您可以使用 [Azure 入口網站](../container-registry/container-registry-get-started-portal.md)或 [Azure CLI](./service-fabric-tutorial-create-container-images.md#deploy-azure-container-registry) 來在 Azure 訂用帳戶中建立容器登錄。 

## <a name="build-a-tomcat-image-and-run-it-locally"></a>建置 Tomcat 映像並於本機執行它
請依照本節中的步驟以 Apache Tomcat 映像和簡單 Web 應用程式為基礎建置 Docker 映像，並在本機系統上的容器中執行它。 
 
1. 將[搭配 Java 的 Service Fabric 快速入門](https://github.com/Azure-Samples/service-fabric-java-getting-started) \(英文\) 範例存放庫複製到開發電腦上。

   ```bash
   git clone https://github.com/Azure-Samples/service-fabric-java-getting-started.git
   ```

1. 將目錄變更至 Apache Tomcat 伺服器範例目錄 (*service-fabric-java-getting-started/container-apache-tomcat-web-server-sample*)：

   ```bash
   cd service-fabric-java-getting-started/container-apache-tomcat-web-server-sample
   ```

1. 以位於 Docker Hub 上的官方 [Tomcat 映像](https://hub.docker.com/_/tomcat/) \(英文\) 和 Tomcat 伺服器範例為基礎建立 Docker 檔案。 在 *service-fabric-java-getting-started/container-apache-tomcat-web-server-sample* 目錄中，建立名為 *Dockerfile* (無副檔名) 的檔案。 將下列內容新增至 *Dockerfile* 並儲存變更：

   ```
   FROM library/tomcat

   EXPOSE 8080

   COPY ./ApacheTomcat /usr/local/tomcat
   ```

   如需詳細資訊，請參閱 [Dockerfile 參考](https://docs.docker.com/engine/reference/builder/) \(英文\)。


4. 執行 `docker build` 命令來建立可執行 Web 應用程式的映像：

   ```bash
   docker build . -t tomcattest
   ```

   此命令會使用 Dockerfile 中的指示建立新映像，並將此映像命名 (-t 標記) 為 `tomcattest`。 若要建置容器映像，系統會先從 Docker Hub 下載基礎映像，然後將應用程式加入它。 

   建置命令完成後，執行 `docker images` 命令以查看新映像的資訊︰

   ```bash
   $ docker images
    
   REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
   tomcattest                    latest              86838648aab6        2 minutes ago       194 MB
   ```

5. 先確認您的容器化應用程式在本機執行，再將它推送至容器登錄：
 
   ```bash
   docker run -itd --name tomcat-site -p 8080:8080 tomcattest.
   ```
   
   * `--name` 會為容器命名，讓您可以使用易記名稱 (而非其識別碼) 來參考它。
   * `-p` 會指定容器和主機 OS 之間的連接埠對應。 

   > [!Note]
   > 您以 `-p` 參數開啟的連接埠，應該是您的 Tomcat 應用程式接聽要求的連接埠。 目前的範例已在 *ApacheTomcat/conf/server.xml* 檔案中設定連接器，以在連接埠 8080 上接聽 HTTP 要求。 此連接埠已對應至主機上的連接埠 8080。 

   若要了解其他參數，請參閱 [Docker run 文件](https://docs.docker.com/engine/reference/commandline/run/) \(英文\)。

1. 若要測試容器，請開啟瀏覽器並輸入下列其中一個 URL。 針對每個 URL，您將會看見不同版本的 "Hello World!" 歡迎畫面。

   - http://localhost:8080/hello 
   - http://localhost:8080/hello/sayhello 
   - http://localhost:8080/hello/sayhi 

   ![Hello world /sayhi](./media/service-fabric-get-started-tomcat/hello.png)

2. 停止容器並將它從開發電腦中刪除：

   ```bash
   docker stop tomcat-site
   docker rm tomcat-site
   ```

## <a name="push-the-tomcat-image-to-your-container-registry"></a>將 Tomcat 映像推送至容器登錄
在您確認 Tomcat 映像已在開發電腦上的容器中執行之後，請將它推送至容器登錄中的存放庫。 此文章使用 Azure Container Registry 來儲存映像，但您可以修改步驟來使用您偏好的其他容器登錄。 本文會假設登錄名稱為 *myregistry*，且完整登錄名稱為 myregistry.azurecr.io。 請針對您的案例變更這些內容。 

1. 使用您的[登錄認證](../container-registry/container-registry-authentication.md)執行 `docker login` 登入容器登錄庫。

   下列範例會傳遞 Azure Active Directory [service principal](../active-directory/develop/app-objects-and-service-principals.md) 的識別碼和密碼。 例如，您可能基於自動化案例已指派服務主體到您的登錄庫。 或者，您可以使用登錄使用者名稱和密碼進行登入。

   ```bash
   docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
   ```

2. 下列命令會建立映像的標籤或別名，包含登錄的完整路徑。 這個範例會指定 `samples` 命名空間中的映像，以避免登錄根目錄雜亂。

   ```bash
   docker tag tomcattest myregistry.azurecr.io/samples/tomcattest
   ```

3. 將映像推送至容器登錄：

   ```bash
   docker push myregistry.azurecr.io/samples/tomcattest
   ```

## <a name="build-and-deploy-the-service-fabric-container-application"></a>建置並部署 Service Fabric 容器應用程式
您已將 Tomcat 映像推送至容器登錄，現在您可以建置並部署會從登錄提取 Tomcat 映像，並將它在叢集中作為容器化服務執行的 Service Fabric 容器應用程式。 

1. 在本機複製品之外 (在 *service-fabric-java-getting-started* 樹狀目錄之外) 建立新的目錄。 切換到它，並使用 Yeoman 針對容器應用程式建立 scaffold： 

   ```bash
   yo azuresfcontainer 
   ```
   當系統提示時，輸入下列值：

   * Name your application (為應用程式命名)：ServiceFabricTomcat
   * Name of the application service (應用程式服務的名稱)：TomcatService
   * Input the Image Name (輸入映像名稱)：提供容器映像在容器登錄中的 URL，例如 myregistry.azurecr.io/samples/tomcattest。
   * Commands (命令)：將此保留空白。 因為此映像已定義工作負載進入點，您不需要明確指定輸入命令 (命令會在容器內執行，這會讓容器在啟動後繼續執行)。
   * Number of instances of guest container application (客體容器應用程式的執行個體數目)：1

   ![容器的 Service Fabric Yeoman 產生器](./media/service-fabric-get-started-tomcat/yo-generator.png)

10. 在服務資訊清單 (*ServiceFabricTomcat/ServiceFabricTomcat/TomcatServicePkg/ServiceManifest.xml*) 中，在根 **ServiceManfest** 標記下方新增下列 XML，以開啟應用程式接聽要求的連接埠。 **Endpoint** 標記會宣告端點的通訊協定和連接埠。 針對此文章，容器化服務會接聽連接埠 8080： 

    ```xml
    <Resources>
      <Endpoints>
        <!-- This endpoint is used by the communication listener to obtain the port on which to 
         listen. Please note that if your service is partitioned, this port is shared with 
         replicas of different partitions that are placed in your code. -->
        <Endpoint Name="endpointTest" Port="8080" Protocol="tcp"/>
      </Endpoints>
    </Resources>
    ```

11. 在應用程式資訊清單 (*ServiceFabricTomcat/ServiceFabricTomcat/ApplicationManifest.xml*) 中，在 **ServiceManifestImport** 標記下方新增下列 XML。 將 **RepositoryCredentials** 標記中的 **AccountName** 和 **Password** 取代為容器登錄的名稱，以及登入它所需的密碼。

    ```xml
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <PortBinding ContainerPort="8080" EndpointRef="endpointTest"/>
        <RepositoryCredentials AccountName="myregistry" Password="=P==/==/=8=/=+u4lyOB=+=nWzEeRfF=" PasswordEncrypted="false"/>
      </ContainerHostPolicies>
    </Policies>
    ```

    **ContainerHostPolicies** 標記會指定用於啟動容器主機的原則。
    
    * **PortBinding** 標記會設定容器連接埠至主機連接埠的對應原則。 **ContainerPort** 屬性會設定為 8080，因為容器會公開連接埠 8080 (如 Dockerfile 中所指定)。 **EndpointRef** 屬性會設定為 "endpointTest"，也就是在上個步驟中定義於服務資訊清單中的端點。 因此，在通訊埠 8080 上針對服務的連入要求，會對應到容器上的連接埠 8080。 
    * **RepositoryCredentials** 標記會指定容器向提取映像的目標 (私人) 存放庫進行驗證所需的認證。 若您將從公用存放庫提取映像，便不需要此原則。
    

12. 在 *ServiceFabricTomcat* 資料夾中，連線至您的 Service Fabric 叢集。 

    * 若要連線至本機 Service Fabric 叢集，請執行：

       ```bash
       sfctl cluster select --endpoint http://localhost:19080
       ```
    
    * 若要連線至安全的 Azure 叢集，請先確定用戶端憑證已以 .pem 檔案形式存在於 *ServiceFabricTomcat* 目錄中，然後執行： 

       ```bash
       sfctl cluster select --endpoint https://PublicIPorFQDN:19080 -pem your-certificate.pem -no-verify
       ```
       在上述命令中，將 `your-certificate.pem` 取代為用戶端憑證檔案的名稱。 在開發與測試環境中，經常會使用叢集憑證作為用戶端憑證。 如果您的憑證未自我簽署，請省略 `-no-verify` 參數。 
       
       叢集憑證通常會以 .pfx 檔案的形式下載至本機。 如果您尚未擁有 PEM 格式的憑證，則可以執行下列命令來從 .pfx 檔案建立 .pem 檔案：

       ```bash
       openssl pkcs12 -in your-certificate.pfx -out your-certificate.pem -nodes -passin pass:your-pfx-password
       ```

       如果您的 .pfx 檔案未受密碼保護，請針對最後一個參數使用 `-passin pass:`。


13. 執行範本中所提供的安裝指令碼來將應用程式部署至您的叢集。 指令碼會將應用程式套件複製到叢集的映像存放區、註冊應用程式類型，並建立應用程式的執行個體。

       ```bash
       ./install.sh
       ```

    在執行安裝指令碼之後，請開啟瀏覽器並瀏覽至 Service Fabric Explorer：
    
    * 在本機叢集上，請使用 http://localhost:19080/Explorer (若使用 Mac OS X 上的 Vagrant，請將 *localhost* 取代為 VM 的私人 IP)。
    * 在安全的 Azure 叢集上，使用 https://PublicIPorFQDN:19080/Explorer。 
    
    展開 [應用程式] 節點，並注意到有一個針對您應用程式類型的項目 (**ServiceFabricTomcatType**)，以及另一個針對該類型第一個執行個體的項目。 應用程式可能需要幾分鐘的時間才能完全部署，請耐心等候。

    ![Service Fabric Explorer](./media/service-fabric-get-started-tomcat/service-fabric-explorer.png)


1. 若要在 Tomcat 伺服器上存取該應用程式，請開啟瀏覽器視窗並輸入下列任一 URL。 如果您是部署至本機叢集，請為 *PublicIPorFQDN* 使用 *localhost*。 針對每個 URL，您將會看見不同版本的 "Hello World!" 歡迎畫面。

   * http://PublicIPorFQDN:8080/hello  
   * http://PublicIPorFQDN:8080/hello/sayhello
   * http://PublicIPorFQDN:8080/hello/sayhi

## <a name="clean-up"></a>清除
使用範本中提供的解除安裝指令碼來刪除叢集中的應用程式執行個體，並將應用程式類型取消註冊。

```bash
./uninstall.sh
```

將映像推送至容器登錄之後，您可以從開發電腦刪除本機映像︰

```
docker rmi tomcattest
docker rmi myregistry.azurecr.io/samples/tomcattest
```

## <a name="next-steps"></a>後續步驟
* 如需其他 Linux 容器功能的快速步驟，請參閱[在 Linux 建立第一個 Service Fabric 容器應用程式](service-fabric-get-started-containers-linux.md)。
* 如需 Linux 容器的詳細步驟，請參閱[建立 Linux 容器應用程式](service-fabric-tutorial-create-container-images.md)教學課程。
* 深入了解如何[在 Service Fabric 上執行容器](service-fabric-containers-overview.md)。


