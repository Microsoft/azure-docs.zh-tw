---
title: 適用於 Eclipse 的 Azure Service Fabric 外掛程式 | Microsoft Docs
description: 開始使用適用於 Eclipse 的 Service Fabric 外掛程式。
services: service-fabric
documentationcenter: java
author: rapatchi
manager: timlt
editor: ''
ms.assetid: bf84458f-4b87-4de1-9844-19909e368deb
ms.service: service-fabric
ms.devlang: java
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/06/2018
ms.author: rapatchi
ms.openlocfilehash: a9888a23088949b5373aa0eef7d4df3b3064466f
ms.sourcegitcommit: 99a6a439886568c7ff65b9f73245d96a80a26d68
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/31/2018
ms.locfileid: "39358580"
---
# <a name="service-fabric-plug-in-for-eclipse-java-application-development"></a>適用於 Eclipse Java 應用程式開發的 Service Fabric 外掛程式
Eclipse 是 Java 開發人員最普遍使用的整合式的開發環境 (IDE) 之一。 在本文中，我們將說明如何設定 Eclipse 開發環境來使用 Azure Service Fabric。 了解如何安裝 Service Fabric 外掛程式、建立 Service Fabric 應用程式，以及將 Service Fabric 應用程式部署到 Eclipse 中的本機或遠端 Service Fabric 叢集。 

> [!NOTE]
> Windows 上目前不支援 Eclipse 外掛程式。 

## <a name="install-or-update-the-service-fabric-plug-in-in-eclipse"></a>在 Eclipse 中安裝或更新 Service Fabric 外掛程式
您可以在 Eclipse 中安裝 Service Fabric 外掛程式。 外掛程式可協助簡化建置和部署 Java 服務的程序。

> [!IMPORTANT]
> Service Fabric 外掛程式需要 Eclipse Neon 或更新版本。 請參閱此注意事項之後的指示，以了解如何檢查您的 Eclipse 版本。 如果您已安裝較早版本的 Eclipse，可以從 [Eclipse 網站](https://www.eclipse.org)下載較新版本。 建議您不要在現有的 Eclipse 安裝之上進行安裝 (覆寫)。 您可以在執行安裝程式之前先移除舊版，或在不同的目錄中安裝較新版本。 
> 
> 在 Ubuntu 上，建議您直接從 Eclipse 網站安裝，而不要使用套件安裝程式 (`apt` 或 `apt-get`)。 這麼做可確保您會取得最新版的 Eclipse。 

從 [Eclipse 網站](https://www.eclipse.org)安裝 Eclipse Neon 版或更新版本。  同時安裝 Buildship 2.2.1 版或更新版本 (Service Fabric 外掛程式與舊版 Buildship 不相容)：
-   若要檢查已安裝的元件版本，請在 Eclipse 中移至 [說明] > [關於 Eclipse] > [安裝詳細資料]。
-   若要更新 Buildship，請參閱 [Eclipse Buildship：適用於 Gradle 的 Eclipse 外掛程式][buildship-update]。
-   若要檢查並安裝 Eclipse 的更新，請移至 [說明] > [檢查更新]。

若要安裝 Service Fabric 外掛程式，請在 Eclipse 中移至 [說明] > [安裝新軟體]。
1. 在 [使用] 方塊中，輸入 **http://dl.microsoft.com/eclipse**。
2. 按一下 [新增] 。

   ![適用於 Eclipse 的 Service Fabric 外掛程式][sf-eclipse-plugin-install]
3. 選取 Service Fabric 外掛程式，然後按 [下一步]。
4. 完成安裝步驟，然後接受 Microsoft 軟體授權條款。
  
如果您已安裝 Service Fabric 外掛程式，請安裝最新版本。 
1. 若要檢查可用的更新，前往 [說明] > [關於 Eclipse] > [安裝詳細資料]。 
2. 在已安裝外掛程式的清單中，選取 Service Fabric，然後按一下 [更新]。 將安裝可用的更新。
3. 更新 Service Fabric 外掛程式後，請同時重新整理 Gradle 專案。  以滑鼠右鍵按一下 [build.gradle]，然後選取 [重新整理]。

> [!NOTE]
> 如果安裝或更新 Service Fabric 外掛程式時很緩慢，可能是因為 Eclipse 設定。 Eclipse 會收集所有變更的中繼資料，以更新使用您 Eclipse 執行個體註冊的網站。 若要加速檢查及安裝 Service Fabric 外掛程式更新的程序，前往 [可用的軟體站台]。 清除所有網站的核取方塊，但指向 Service Fabric 外掛程式位置 (http://dl.microsoft.com/eclipse/azure/servicefabric)) 的網站除外。

> [!NOTE]
>如果 Eclipse 在 Mac 上未如預期一般運作，或需要您以進階使用者身分執行，請移至 **ECLIPSE_INSTALLATION_PATH** 資料夾，並瀏覽至子資料夾 **Eclipse.app/Contents/MacOS**。 執行 `./eclipse` 來啟動 Eclipse。


## <a name="create-a-service-fabric-application-in-eclipse"></a>在 Eclipse 中建立 Service Fabric 應用程式

1.  在 Eclipse 中，移至 [檔案] > [新增] > [其他]。 選取 **Service Fabric 外掛程式**，然後按 [下一步]。

    ![Service Fabric 新專案第 1 頁][create-application/p1]

2.  輸入專案的名稱，然後按 [下一步]。

    ![Service Fabric 新專案第 2 頁][create-application/p2]

3.  在範本清單中，選取 [服務範例]。 選取服務範本類型 (執行者、無狀態、容器或來賓二進位)，然後按 [下一步]。

    ![Service Fabric 新專案第 3 頁][create-application/p3]

4.  輸入服務名稱和服務詳細資訊，然後按一下 [完成]。

    ![Service Fabric 新專案第 4 頁][create-application/p4]

5. 當您建立第一個 Service Fabric 專案時，在 [開啟相關聯的觀點] 對話方塊中，按一下 [是]。

    ![Service Fabric 新專案第 5 頁][create-application/p5]

6.  新的專案看起來像這樣︰

    ![Service Fabric 新專案第 6 頁][create-application/p6]

## <a name="build-a-service-fabric-application-in-eclipse"></a>在 Eclipse 中建置 Service Fabric 應用程式

1.  以滑鼠右鍵按一下新的 Service Fabric 應用程式，然後選取 [Service Fabric]。

    ![Service Fabric 快顯功能表][publish/RightClick]

2. 在快顯功能表中，選取下列其中一個選項︰
    -   若要建置應用程式而不進行清除，按一下 [建置應用程式]。
    -   若要對應用程式執行清除建置，按一下 [重新建置應用程式]。
    -   若要清除所建置構件的應用程式，按一下 [清除應用程式]。
     
## <a name="deploy-a-service-fabric-application-to-the-local-cluster-with-eclipse"></a>使用 Eclipse 將 Service Fabric 應用程式部署到本機叢集

建置您的 Service Fabric 應用程式之後，請依照這些步驟將它部署到本機叢集。

1. 若您尚未啟動本機叢集，請依照[設定本機叢集](./service-fabric-get-started-linux.md#set-up-a-local-cluster)中的指示啟動您的本機叢集並確定它正確執行中。
2. 以滑鼠右鍵按一下您的 Service Fabric 應用程式，然後選取 [Service Fabric]。

    ![Service Fabric 快顯功能表][publish/RightClick]

3.  從快顯功能表中，按一下 [部署應用程式]。
4.  您可以在 [主控台] 視窗中查看部署作業的進度。
5.  若要檢查您的應用程式是否執行中，請在瀏覽器視窗中在您的本機叢集開啟 Service Fabric Explorer [http://localhost:19080/Explorer](http://localhost:19080/Explorer)。 展開 [應用程式] 節點，並確認您的應用程式執行中。 

若要了解如何在 Eclipse 中使用本機叢集針對您的應用程式進行偵錯，請 參閱[在 Eclipse 中針對 Java 服務進行偵錯](./service-fabric-debugging-your-application-java.md)。

您也可以使用 [發佈應用程式] 命令將您的應用程式部署到本機叢集：

1. 以滑鼠右鍵按一下您的 Service Fabric 應用程式，然後選取 [Service Fabric]。
2. 從快顯功能表中，按一下 [發佈應用程式]。
3. 在 [發佈應用程式] 視窗中，選擇 [PublishProfiles/Local.json] 做為目標設定檔，然後按一下 [發佈]。

    ![本機發佈對話方塊](./media/service-fabric-get-started-eclipse/localjson.png)

    根據預設值，Local.json 發佈檔案是設定為發佈到本機叢集。 如需有關發佈設定檔中存在之連線與端點參數的詳細資訊，請參閱下一節。

## <a name="publish-your-service-fabric-application-to-azure-with-eclipse"></a>使用 Eclipse 將您的 Service Fabric 應用程式發佈到 Azure

若要將您的應用程式發佈到雲端，請依照這些步驟執行：

1. 若要將您的應用程式發佈到雲端中的安全叢集，您需要使用 X.509 憑證來與您的叢集通訊。 在測試與開發環境中使用的憑證通常是叢集憑證。 在生產環境中，憑證應該是與叢集憑證不同的用戶端憑證。 您需要憑證與私密金鑰。 憑證 (與金鑰) 檔案必須是 PEM 格式。 您可以使用下列 openssl 命令從 PFX 檔案建立包含憑證與私密金鑰的 PEM 檔案：

    ```bash
    openssl pkcs12 -in your-cert-file.pfx -out your-cert-file.pem -nodes -passin pass:your-pfx-password
    ```

   若 PFX 檔案未受密碼保護，請為最後一個參數使用 `--passin pass:`。

2. 開啟 [PublishProfiles]目錄下的 **Cloud.json** 檔案。 您必須適當地為您的叢集設定叢集端點與安全性認證。

   - `ConnectionIPOrURL` 欄位包含您叢集的 IP 位址或 URL。 請注意，該值不包含 URL 配置 (`https://`)。
   - 根據預設值，`ConnectionPort` 欄位應該是 `19080`，除非您已為您的叢集變更此連接埠。
   - `ClientKey` 欄位應該指向您本機電腦上包含您的用戶端或叢集憑證之私密金鑰的 PEM 格式 .pem 或 .key 檔案。
   - `ClientCert` 欄位應該指向您本機電腦上包含您的用戶端或叢集 憑證之憑證資料的 PEM 格式 .pem 或 .crt 檔案。 

    ```bash
    {
         "ClusterConnectionParameters":
         {
            "ConnectionIPOrURL": "lnxxug0tlqm5.westus.cloudapp.azure.com",
            "ConnectionPort": "19080",
            "ClientKey": "[path_to_your_pem_file_on_local_machine]",
            "ClientCert": "[path_to_your_pem_file_on_local_machine]"
         }
    }
    ```

2. 以滑鼠右鍵按一下您的 Service Fabric 應用程式，然後選取 [Service Fabric]。
3. 從快顯功能表中，按一下 [發佈應用程式]。
3. 在 [發佈應用程式] 視窗中，選擇 [PublishProfiles/Cloud.json] 做為目標設定檔，然後按一下 [發佈]。

    ![雲端發佈對話方塊](./media/service-fabric-get-started-eclipse/cloudjson.png)

4.  您可以在 [主控台] 視窗中查看發佈作業的進度。
5.  若要檢查您的應用程式是否執行中，請在瀏覽器視窗中在您的 Azure 叢集開啟 Service Fabric Explorer。 針對上面的案例，這應該是：`https://lnxxug0tlqm5.westus.cloudapp.azure.com:19080/Explorer`。 展開 [應用程式] 節點，並確認您的應用程式執行中。 


在安全的 Linux 叢集上，若您的應用程式包含 Reliable Services 服務，您將必須設定可供您的服務用來呼叫 Service Fabric runtime API 的憑證。 若要深入了解，請參閱[將 Reliable Services 應用程式設定為在 Linux 叢集上執行](./service-fabric-configure-certificates-linux.md#configure-a-reliable-services-app-to-run-on-linux-clusters)。

如需有關如何將使用 Java 撰寫的 Service Fabric Reliable Services 應用程式部署到安全 Linux 叢集的快速逐步教學課程，請參閱[快速入門：部署 Java Reliable Services 應用程式](./service-fabric-quickstart-java-reliable-services.md)。

## <a name="deploy-a-service-fabric-application-by-using-eclipse-run-configurations"></a>使用 Eclipse 回合組態來部署 Service Fabric 應用程式

另一種方式是使用 Eclipse 回合組態來部署 Service Fabric 應用程式。

1. 在 Eclipse 中，移至 [執行] > [回合組態]。
2. 在 [等級專案] 下，選取 **ServiceFabricDeployer** 回合組態。
3. 在右窗格的 [引數] 索引標籤上，確定已適當地針對您的部署設定 **ip**、**port**、**clientCert** 與 **clientKey** 參數。 根據預設值，參數是設定為部署到本機叢集，如下列螢幕擷取畫面所示。 若要將您的應用程式發佈到 Azure，您可以修改參數以包含 Azure 叢集的端點詳細資料與安全性認證。 如需詳細資訊，請參閱上一節[使用 Eclipse 將您的 Service Fabric 應用程式發佈到 Azure](#publish-your-service-fabric-application-to-azure-with-eclipse)。

    ![[回合組態] 對話方塊本機](./media/service-fabric-get-started-eclipse/run-config-local.png)

5. 確定 [工作目錄] 指向您要部署的應用程式。 如果要變更應用程式，請按一下 [工作區] 按鈕，然後選取您要的應用程式。
6. 按一下 [套用]，然後按一下 [執行]。

幾分鐘內即可建置和部署您的應用程式。 您可以在 Service Fabric Explorer 中監視部署狀態。  

## <a name="add-a-service-fabric-service-to-your-service-fabric-application"></a>將 Service Fabric 服務新增至 Service Fabric 應用程式

若要將 Service Fabric 服務新增至現有 Service Fabric 應用程式，請執行下列步驟︰

1.  以滑鼠右鍵按一下您要新增服務的專案，然後按一下 [Service Fabric]。

    ![Service Fabric 新增服務第 1 頁][add-service/p1]

2.  按一下 [新增 Service Fabric 服務]，並完成步驟集合以將服務新增至專案。
3.  選取要新增至專案的服務範本，然後按 [下一步]。

    ![Service Fabric 新增服務第 2 頁][add-service/p2]

4.  輸入服務名稱 (以及視需要其他詳細資料)，然後按一下 [新增服務] 按鈕。  

    ![Service Fabric 新增服務第 3 頁][add-service/p3]

5.  新增服務之後，您的整體專案結構看起來類似下列專案︰

    ![Service Fabric 新增服務第 4 頁][add-service/p4]

## <a name="edit-manifest-versions-of-your-service-fabric-java-application"></a>編輯 Service Fabric Java 應用程式的資訊清單版本

若要編輯資訊清單版本，以滑鼠右鍵按一下專案，移至 **Service Fabric**，然後從功能表下拉式清單中選取 [編輯資訊清單版本...]。 在精靈中，您可以更新應用程式資訊清單、服務資訊清單的資訊清單版本，以及 **Code**、**Config** 和 **Data** 套件的版本。

如果您勾選 [自動更新應用程式和服務版本] 選項，然後更新版本，則會自動更新資訊清單版本。 若要提供範例，您先選取核取方塊，然後將 **Code** 的版本從 0.0.0 更新為 0.0.1 並按一下 [完成]，然後服務資訊清單版本和應用程式資訊清單版本將會自動更新為 0.0.1。

## <a name="upgrade-your-service-fabric-java-application"></a>升級 Service Fabric Java 應用程式

在升級案例中，假設您在 Eclipse 中使用 Service Fabric 外掛程式建立 **App1** 專案。 您使用外掛程式部署它，建立名為 **fabric:/App1Application** 的應用程式。 應用程式類型是 **App1AppicationType**，應用程式版本為 1.0。 現在您想要升級應用程式，但不要中斷可用性。

首先，變更您的應用程式，然後重建已修改的服務。 以服務 (和程式碼、組態或資料，視相關性而定) 已更新的版本，更新已修改之服務的資訊清單檔案 (ServiceManifest.xml)。 同時，也以應用程式已更新的版本號碼及已修改的服務，修改應用程式的資訊清單 (ApplicationManifest.xml)。  

若要使用 Eclipse 升級您的應用程式，您可以建立重複的回合組態設定檔。 然後，視需要使用它升級您的應用程式。

1.  前往 [執行] > [回合組態]。 在左窗格中，按一下 [等級專案] 左邊的小箭號。
2.  以滑鼠右鍵按一下 **ServiceFabricDeployer**，然後選取 [重複]。 輸入此設定的新名稱，例如，**ServiceFabricUpgrader**。
3.  在右窗格的 [引數] 索引標籤上，將 **-Pconfig='deploy'** 變更為 **-Pconfig='upgrade'**，然後按一下 [套用]。

此程序會建立並儲存您可以隨時用來升級應用程式的回合組態設定檔。 這也會從應用程式資訊清單檔中取得最新版更新應用程式類型版本的程序。

應用程式升級會花費幾分鐘的時間。 您可以在 Service Fabric Explorer 中監視應用程式升級。

## <a name="migrating-old-service-fabric-java-applications-to-be-used-with-maven"></a>移轉要搭配 Maven 使用的舊版 Service Fabric Java 應用程式
我們最近已將 Service Fabric Java 程式庫從 Service Fabric Java SDK 移至 Maven 存放庫。 雖然您使用 Eclipse 產生的新應用程式將產生最新的專案 (能夠搭配 Maven 運作)，但您可以更新現有的 Service Fabric 無狀態或動作項目 Java 應用程式 (先前使用 Service Fabric Java SDK)，以使用 Maven 提供的 Service Fabric Java 相依性。 請遵循[這裡](service-fabric-migrate-old-javaapp-to-use-maven.md)所述的步驟，確保舊版應用程式能搭配 Maven 運作。

## <a name="next-steps"></a>後續步驟

- 如需有關如何建置 Java Reliable Services 應用程式及將它部署到本機與部署到 Azure 的快速步驟，請參閱[快速入門：部署 Java Reliable Services 應用程式](./service-fabric-quickstart-java-reliable-services.md)。
- 若要了解如何在您的本機叢集上針對 Java 應用程式進行偵錯，請 參閱[在 Eclipse 中針對 Java 服務進行偵錯](./service-fabric-debugging-your-application-java.md)。
- 若要了解如何監視 Service Fabric 應用程式及針對此類應用程式進行診斷，請參閱[監視和診斷本機開發設定中的服務](./service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux.md)。

<!-- Images -->

[sf-eclipse-plugin-install]: ./media/service-fabric-get-started-eclipse/service-fabric-eclipse-plugin.png

[create-application/p1]:./media/service-fabric-get-started-eclipse/create-application/p1.png
[create-application/p2]:./media/service-fabric-get-started-eclipse/create-application/p2.png
[create-application/p3]:./media/service-fabric-get-started-eclipse/create-application/p3.png
[create-application/p4]:./media/service-fabric-get-started-eclipse/create-application/p4.png
[create-application/p5]:./media/service-fabric-get-started-eclipse/create-application/p5.png
[create-application/p6]:./media/service-fabric-get-started-eclipse/create-application/p6.png

[publish/Publish]: ./media/service-fabric-get-started-eclipse/publish/Publish.png
[publish/RightClick]: ./media/service-fabric-get-started-eclipse/publish/RightClick.png

[add-service/p1]: ./media/service-fabric-get-started-eclipse/add-service/p1.png
[add-service/p2]: ./media/service-fabric-get-started-eclipse/add-service/p2.png
[add-service/p3]: ./media/service-fabric-get-started-eclipse/add-service/p3.png
[add-service/p4]: ./media/service-fabric-get-started-eclipse/add-service/p4.png

<!-- Links -->
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship
