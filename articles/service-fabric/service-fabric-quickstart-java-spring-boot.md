---
title: 在 Azure 中的 Service Fabric 上建立 Spring Boot 應用程式 | Microsoft Docs
description: 在本教學課程中，您會使用 Spring Boot 範例應用程式來部署適用於 Azure Service Fabric 的 Spring Boot 應用程式。
services: service-fabric
documentationcenter: java
author: suhuruli
manager: msfussell
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: java
ms.topic: quickstart
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/23/2017
ms.author: suhuruli
ms.custom: mvc, devcenter
ms.openlocfilehash: ea74d7893a55e37b5a198de17d71754b6f23a09e
ms.sourcegitcommit: 0bb8db9fe3369ee90f4a5973a69c26bff43eae00
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48870055"
---
# <a name="quickstart-deploy-a-java-spring-boot-application-to-service-fabric"></a>快速入門：將 Java Spring Boot 應用程式部署到 Service Fabric

Azure Service Fabric 是一個分散式系統平台，可讓您部署及管理微服務與容器。

此快速入門說明如何將 Spring Boot 應用程式部署到 Service Fabric。 本快速入門使用 Spring 網站上的 [Getting Started](https://spring.io/guides/gs/spring-boot/) 範例。 本快速入門使用熟悉的命令列工具，引導您將 Spring Boot 範例部署為 Service Fabric 應用程式。 完成後，您就可以在 Service Fabric 上使用 Spring Boot Getting Started 範例。

![應用程式螢幕擷取畫面](./media/service-fabric-quickstart-java-spring-boot/springbootsflocalhost.png)

在此快速入門中，您可了解如何：

* 將 Spring Boot 應用程式部署到 Service Fabric
* 將應用程式部署到本機叢集
* 將應用程式部署到 Azure 中的叢集
* 跨多個節點相應放大應用程式
* 無法使用時，執行服務的容錯移轉

## <a name="prerequisites"></a>必要條件

若要完成本快速入門：

1. 安裝 Service Fabric SDK 和 Service Fabric 命令列介面 (CLI)

    a. [Mac](https://docs.microsoft.com/azure/service-fabric/service-fabric-cli#cli-mac)
    
    b. [Linux](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started-linux#installation-methods)

1. [安裝 Git](https://git-scm.com/)
1. 安裝 Yeoman

    a. [Mac](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started-mac#create-your-application-on-your-mac-by-using-yeoman)

    b. [Linux](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started-linux#set-up-yeoman-generators-for-containers-and-guest-executables)
1. 設定 Java 環境

    a. [Mac](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started-mac#create-your-application-on-your-mac-by-using-yeoman)
    
    b.  [Linux](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started-linux#set-up-java-development)

## <a name="download-the-sample"></a>下載範例

在終端機視窗中執行下列命令，將 Spring Boot Getting Started 範例應用程式複製到本機電腦。

```bash
git clone https://github.com/spring-guides/gs-spring-boot.git
```

## <a name="build-the-spring-boot-application"></a>建置 Spring Boot 應用程式 
1. 在 `gs-spring-boot/complete` 目錄中，執行下列命令以建置應用程式 

    ```bash
    ./gradlew build
    ``` 

## <a name="package-the-spring-boot-application"></a>封裝 Spring Boot 應用程式 
1. 在複本的 `gs-spring-boot` 目錄中，執行 `yo azuresfguest` 命令。 

1. 針對每個提示，輸入下列詳細資料。

    ![Yeoman 項目](./media/service-fabric-quickstart-java-spring-boot/yeomanspringboot.png)

1. 在 `SpringServiceFabric/SpringServiceFabric/SpringGettingStartedPkg/code` 資料夾中，建立名為 `entryPoint.sh` 的檔案。 將下列項目新增至 `entryPoint.sh` 檔案。 

    ```bash
    #!/bin/bash
    BASEDIR=$(dirname $0)
    cd $BASEDIR
    java -jar gs-spring-boot-0.1.0.jar
    ```

1. 在 `gs-spring-boot/SpringServiceFabric/SpringServiceFabric/SpringGettingStartedPkg/ServiceManifest.xml` 檔案中新增 [端點] 資源

    ```xml 
        <Resources>
          <Endpoints>
            <Endpoint Name="WebEndpoint" Protocol="http" Port="8080" />
          </Endpoints>
       </Resources>
    ```

    **ServiceManifest.xml** 現在如下所示： 

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <ServiceManifest Name="SpringGettingStartedPkg" Version="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" >

       <ServiceTypes>
          <StatelessServiceType ServiceTypeName="SpringGettingStartedType" UseImplicitHost="true">
       </StatelessServiceType>
       </ServiceTypes>

       <CodePackage Name="code" Version="1.0.0">
          <EntryPoint>
             <ExeHost>
                <Program>entryPoint.sh</Program>
                <Arguments></Arguments>
                <WorkingFolder>CodePackage</WorkingFolder>
             </ExeHost>
          </EntryPoint>
       </CodePackage>
        <Resources>
          <Endpoints>
            <Endpoint Name="WebEndpoint" Protocol="http" Port="8080" />
          </Endpoints>
       </Resources>
     </ServiceManifest>
    ```

在這個階段，您已經針對可以部署到 Service Fabric 的 Spring Boot Getting Started 範例，建立 Service Fabric 應用程式。

## <a name="run-the-application-locally"></a>在本機執行應用程式

1. 執行下列命令，以在 Ubuntu 機器上啟動本機叢集：

    ```bash
    sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
    ```

    如果使用 Mac，請從 Docker 映像啟動本機叢集 (假設您已遵循[必要條件](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started-mac#create-a-local-container-and-set-up-service-fabric)針對 Mac 設定您的本機叢集)。 

    ```bash
    docker run --name sftestcluster -d -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 -p 8080:8080 mysfcluster
    ```

    啟動本機叢集需要一些時間。 若要確認該叢集完全啟動，請存取位於 **http://localhost:19080** 的 Service Fabric Explorer。 五個狀況良好的節點表示本機叢集已啟動並執行。 
    
    ![本機叢集狀況良好](./media/service-fabric-quickstart-java-spring-boot/sfxlocalhost.png)

1. 瀏覽到 `gs-spring-boot/SpringServiceFabric` 資料夾。
1. 執行下列命令，以連接到本機叢集。

    ```bash
    sfctl cluster select --endpoint http://localhost:19080
    ```
1. 執行 `install.sh` 指令碼。

    ```bash
    ./install.sh
    ```

1. 開啟您最愛的網頁瀏覽器，並存取 **http://localhost:8080** 以存取應用程式。

    ![本機應用程式前端](./media/service-fabric-quickstart-java-spring-boot/springbootsflocalhost.png)

您現在可以存取已部署至 Service Fabric 叢集的 Spring Boot 應用程式。

## <a name="deploy-the-application-to-azure"></a>將應用程式部署至 Azure

### <a name="set-up-your-azure-service-fabric-cluster"></a>設定 Azure Service Fabric 叢集

若要將應用程式部署到 Azure 中的叢集，請建立您自己的叢集。

合作對象叢集是 Azure 上裝載的免費、限時 Service Fabric 叢集，由 Service Fabric 小組所執行。 您可以使用合作對象叢集來部署應用程式，並了解平台。 叢集會使用單一的自我簽署憑證，以確保節點對節點和用戶端對節點的安全性。

登入並加入 [Linux 叢集](http://aka.ms/tryservicefabric)。 藉由按一下 [PFX] 連結，將 PFX 憑證下載至您的電腦。 按一下**讀我檔案**連結以尋找憑證密碼，以及關於如何設定各種環境以使用憑證的指示。 請將 [歡迎] 頁面和 [讀我檔案] 頁面保持為開啟，您將在下列步驟使用其中的某些指示。

> [!Note]
> 每小時可用的合作對象叢集數目有限。 如果您在嘗試註冊合作對象叢集時收到錯誤，可以等候一段時間再重試，也可以依照[在 Azure 上建立 Service Fabric 叢集](service-fabric-tutorial-create-vnet-and-linux-cluster.md)中的步驟，在您的訂用帳戶中建立叢集。
>
> Spring Boot 服務設定為在連接埠 8080 上接聽傳入流量。 請確定您的叢集中已開啟該連接埠。 如果您使用合作對象叢集，此連接埠已開啟。
>

Service Fabric 提供了數項可用來管理叢集及其應用程式的工具：

* Service Fabric Explorer，此為以瀏覽器為基礎的工具。
* Service Fabric 命令列介面 (CLI)，此 CLI 會在 Azure CLI 之上執行。
* PowerShell 命令。

在此快速入門中，您會使用 Service Fabric CLI 和 Service Fabric Explorer。

若要使用 CLI，必須根據您所下載的 PFX 檔案建立 PEM 檔案。 若要轉換此檔案，請使用下列命令。 (對於合作對象叢集，您可以從 [讀我檔案] 頁面上的指示中，複製您的 PFX 檔案適用的特定命令。)

```bash
openssl pkcs12 -in party-cluster-1486790479-client-cert.pfx -out party-cluster-1486790479-client-cert.pem -nodes -passin pass:1486790479
``` 

若要使用 Service Fabric Explorer，必須將您從合作對象叢集網站下載的憑證 PFX 檔案匯入至憑證存放區 (Windows 或 Mac) 或瀏覽器本身 (Ubuntu)。 您需要 PFX 私密金鑰密碼；此密碼可從 [讀我檔案] 頁面取得。

請使用您最熟悉的方法在您的系統上匯入憑證。 例如︰

* 在 Windows 上：按兩下 PFX 檔案，並依照提示在您的個人存放區中安裝憑證：`Certificates - Current User\Personal\Certificates`。 或者，您可以使用 **ReadMe** 指示中的 PowerShell 命令。
* 在 Mac 上：按兩下 PFX 檔案，並依照提示在您的 Keychain 中安裝憑證。
* 在 Ubuntu 上：Mozilla Firefox 是 Ubuntu 16.04 中的預設瀏覽器。 若要將憑證匯入 Firefox 中，請按一下瀏覽器右上角的功能表按鈕，然後按一下 [選項]。 在 [喜好設定] 頁面上，使用搜尋方塊搜尋「憑證」。 按一下 [檢視憑證]，選取 [您的憑證] 索引標籤上，按一下 [匯入]，並依照提示匯入憑證。

   ![在 Firefox 上安裝憑證](./media/service-fabric-quickstart-java-spring-boot/install-cert-firefox.png)

### <a name="deploy-the-application-using-cli"></a>使用 CLI 部署應用程式

既然應用程式和叢集已經就緒，您可以將其直接從命令列部署到叢集。

1. 瀏覽到 `gs-spring-boot/SpringServiceFabric` 資料夾。
1. 執行下列命令，以連接到 Azure 叢集。

    ```bash
    sfctl cluster select --endpoint https://<ConnectionIPOrURL>:19080 --pem <path_to_certificate> --no-verify
    ```
1. 執行 `install.sh` 指令碼。

    ```bash
    ./install.sh
    ```

1. 開啟您的網頁瀏覽器，並存取 **http://\<ConnectionIPOrUrl>:8080** 以存取應用程式。

    ![本機應用程式前端](./media/service-fabric-quickstart-java-spring-boot/springbootsfazure.png)

您現在可以存取在 Azure 上的 Service Fabric 叢集中執行的 Spring Boot 應用程式。

## <a name="scale-applications-and-services-in-a-cluster"></a>調整叢集中的應用程式和服務

您可以在整個叢集內調整服務，以因應服務的負載變更。 您可以藉由變更叢集中執行的執行個體數目來調整服務。 您可以透過多種方式調整服務，例如，可以使用 Service Fabric CLI (sfctl) 中的指令碼或命令。 在下列步驟中，請使用 Service Fabric Explorer。

Service Fabric Explorer 會在所有 Service Fabric 叢集中執行，並可藉由瀏覽至叢集 HTTP 管理連接埠 (19080) 從瀏覽器存取，例如 `http://localhost:19080`。

若要調整 Web 前端服務，請執行下列動作：

1. 在您的叢集中開啟 Service Fabric Explorer，例如 `http://localhost:19080`。
1. 按一下樹狀檢視中 **fabric:/SpringServiceFabric/SpringGettingStarted** 節點旁邊的省略符號 (三個點)，然後選擇 [調整服務]。

    ![Service Fabric Explorer 的 [調整服務]](./media/service-fabric-quickstart-java-spring-boot/sfxscaleservicehowto.png)

    您現在可以選擇調整服務的執行個體數目。

1. 將數字變更為 **3**，然後按一下 [縮放服務]。

    使用命令列調整服務的替代方式如下所示。

    ```bash
    # Connect to your local cluster
    sfctl cluster select --endpoint https://<ConnectionIPOrURL>:19080 --pem <path_to_certificate> --no-verify

    # Run Bash command to scale instance count for your service
    sfctl service update --service-id 'SpringServiceFabric~SpringGettingStarted' --instance-count 3 --stateless 
    ``` 

1. 按一下樹狀檢視中的 **fabric:/SpringServiceFabric/SpringGettingStarted** 節點，然後展開資料分割節點 (以 GUID 表示)。

    ![Service Fabric Explorer 調整服務完成](./media/service-fabric-quickstart-java-spring-boot/sfxscaledservice.png)

    此服務有三個執行個體，且樹狀檢視會顯示執行個體在其中執行的節點。

藉由這項簡單的管理工作，您已讓前端服務可用來處理使用者負載的資源倍增。 請務必了解，您不需要多個服務執行個體，就能讓服務確實可靠地執行。 如果服務失敗，Service Fabric 可確保會有新的服務執行個體在叢集中執行。

## <a name="fail-over-services-in-a-cluster"></a>叢集中的容錯移轉服務

若要示範服務容錯移轉，可以使用 Service Fabric Explorer 來模擬節點重新啟動。 請確定您的服務只有一個執行個體正在執行。

1. 在您的叢集中開啟 Service Fabric Explorer，例如 `http://localhost:19080`。
1. 按一下執行您服務執行個體之節點旁邊的省略符號 (三個點)，然後重新啟動節點。

    ![Service Fabric Explorer 重新啟動節點](./media/service-fabric-quickstart-java-spring-boot/sfxhowtofailover.png)
1. 您服務的執行個體會移至其他節點，而且您的應用程式不會有任何停機時間。

    ![Service Fabric Explorer 重新啟動節點成功](./media/service-fabric-quickstart-java-spring-boot/sfxfailedover.png)

## <a name="next-steps"></a>後續步驟

在此快速入門中，您已了解如何：

* 將 Spring Boot 應用程式部署到 Service Fabric
* 將應用程式部署到本機叢集
* 將應用程式部署到 Azure 中的叢集
* 跨多個節點相應放大應用程式
* 無法使用時，執行服務的容錯移轉

若要深入了解如何在 Service Fabric 中使用 Java 應用程式，請繼續進行教學課程以了解 Java 應用程式。

> [!div class="nextstepaction"]
> [部署 Java 應用程式](./service-fabric-tutorial-create-java-app.md)
