---
title: 在本機部署和升級 Azure Service Fabric 服務 | Microsoft Docs
description: 了解如何設定本機 Service Fabric 叢集、將現有的應用程式部署至該叢集，然後升級該應用程式。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: 60a1f6a5-5478-46c0-80a8-18fe62da17a8
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/03/2017
ms.author: ryanwi
ms.openlocfilehash: 0243f6ca96b5e2c20b40bf6e0fc11e8d760db475
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44053358"
---
# <a name="get-started-with-deploying-and-upgrading-applications-on-your-local-cluster"></a>在您的本機叢集上開始部署和升級應用程式
Azure Service Fabric SDK 包含完整的本機開發環境，可讓您快速地在本機叢集上開始部署和管理應用程式。 在本文中，您會從 Windows PowerShell 建立本機叢集、將現有的應用程式部署至該叢集，然後將應用程式升級為新版本。

> [!NOTE]
> 本文假設您已經 [設定開發環境](service-fabric-get-started.md)。
> 
> 

## <a name="create-a-local-cluster"></a>建立本機叢集
Service Fabric 叢集代表一組您可以部署應用程式的硬體資源。 通常，叢集是由任意數量的電腦 (從 5 部到數千部) 所組成。 不過，Service Fabric SDK 包含可在一部電腦上執行的叢集組態。

請務必了解 Service Fabric 本機叢集不是模擬器。 它會執行在多部電腦的叢集上找到的相同平台程式碼。 唯一的差別在於它會在一部電腦上執行通常分散於五部電腦的平台程序。

SDK 提供兩種方式來設定本機叢集：Windows PowerShell 指令碼和 [本機叢集管理員] 系統匣應用程式。 在本教學課程中，我們會使用 PowerShell 指令碼。

> [!NOTE]
> 如果您已藉由從 Visual Studio 部署應用程式來建立本機叢集，您可以略過本節。
> 
> 

1. 以系統管理員身分啟動新的 PowerShell 視窗。
2. 從 SDK 資料夾執行叢集設定指令碼：
   
    ```powershell
    & "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\ClusterSetup\DevClusterSetup.ps1"
    ```
   
    叢集設定需要一些時間。 設定完成後，您應該會看到輸出類似於：
   
    ![叢集設定輸出][cluster-setup-success]
   
    您現在已準備好將應用程式部署至您的叢集。

## <a name="deploy-an-application"></a>部署應用程式
Service Fabric SDK 包含一組豐富的架構以及用來建立應用程式的開發人員工具。 如果您有興趣學習如何在 Visual Studio 中建立應用程式，請參閱 [在 Visual Studio 中建立第一個 Service Fabric 應用程式](service-fabric-create-your-first-application-in-visual-studio.md)。

在本教學課程中，您會使用現有的範例應用程式 (稱為 WordCount)，如此您即可專注於平台的管理層面：部署、監視及升級。

1. 以系統管理員身分啟動新的 PowerShell 視窗。
2. 匯入 Service Fabric SDK PowerShell 模組。
   
    ```powershell
    Import-Module "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\Tools\PSModule\ServiceFabricSDK\ServiceFabricSDK.psm1"
    ```
3. 建立目錄以儲存您下載和部署的應用程式，例如 C:\ServiceFabric。
   
    ```powershell
    mkdir c:\ServiceFabric\
    cd c:\ServiceFabric\
    ```
4. [將 WordCount 應用程式下載](http://aka.ms/servicefabric-wordcountapp) 至您建立的位置。  注意︰Microsoft Edge 瀏覽器會使用 .zip  副檔名來儲存檔案。  請將副檔名變更為 .sfpkg 。
5. 連接到本機叢集：
   
    ```powershell
    Connect-ServiceFabricCluster localhost:19000
    ```
6. 使用 SDK 的部署命令來建立新的應用程式，並提供應用程式套件的名稱和路徑。
   
    ```powershell  
   Publish-NewServiceFabricApplication -ApplicationPackagePath c:\ServiceFabric\WordCountV1.sfpkg -ApplicationName "fabric:/WordCount"
    ```
   
    如果順利執行，您應該會看到如下的輸出：
   
    ![將應用程式部署至本機叢集][deploy-app-to-local-cluster]
7. 若要查看作用中的應用程式，請啟動瀏覽器並瀏覽至 [http://localhost:8081/wordcount/index.html](http://localhost:8081/wordcount/index.html)。 您應該會看到：
   
    ![部署應用程式 UI][deployed-app-ui]
   
    WordCount 應用程式很簡單。 它包含的用戶端 JavaScript 程式碼可產生隨機五個字元的「單字」，這些字接著會透過 ASP.NET Web API 轉送至應用程式。 具狀態服務會追蹤已計算的字數。 這些單字會根據單字的第一個字元進行分割。 您可以在[傳統快速入門範例](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/classic/Services/WordCount)中找到 WordCount 應用程式的原始程式碼。
   
    我們所部署的應用程式包含四個資料分割。 所以開頭為 A 到 G 的單字會儲存在第一個磁碟分割中，而開頭為 N 到 H 的單字會儲存在第二個資料分割中，依此類推。

## <a name="view-application-details-and-status"></a>檢視應用程式詳細資料和狀態
我們現已部署應用程式，讓我們在 PowerShell 中看看一些應用程式詳細資料。

1. 查詢叢集上所有已部署的應用程式：
   
    ```powershell
    Get-ServiceFabricApplication
    ```
   
    假設您只部署 WordCount 應用程式，您會看到類似下面的內容：
   
    ![在 PowerShell 中查詢所有已部署的應用程式][ps-getsfapp]
2. 查詢 WordCount 應用程式中包含的服務集合，進而移至下一個層級。
   
    ```powershell
    Get-ServiceFabricService -ApplicationName 'fabric:/WordCount'
    ```
   
    ![在 PowerShell 中列出應用程式的服務][ps-getsfsvc]
   
    此應用程式是由兩個服務所組成：Web 前端服務以及可管理文字的具狀態服務。
3. 最後，看看 WordCountService 的資料分割清單：
   
    ```powershell
    Get-ServiceFabricPartition 'fabric:/WordCount/WordCountService'
    ```
   
    ![在 PowerShell 中檢視服務資料分割][ps-getsfpartitions]
   
    您所使用的命令集 (例如所有的 Service Fabric PowerShell 命令) 適用於任何您可能連接的叢集 (本機或遠端)。
   
    若要以更具視覺效果的方式與叢集互動，您可以在瀏覽器中瀏覽至 [http://localhost:19080/Explorer](http://localhost:19080/Explorer)，以使用 Web 型 Service Fabric Explorer 工具。
   
    ![在 Service Fabric 總管中檢視應用程式詳細資料][sfx-service-overview]
   
   > [!NOTE]
   > 若要深入了解 Service Fabric Explorer，請參閱 [使用 Service Fabric Explorer 視覺化叢集](service-fabric-visualizing-your-cluster.md)。
   > 
   > 

## <a name="upgrade-an-application"></a>升級應用程式
Service Fabric 會在應用程式推展於叢集時監視其健康狀態，進而提供不需停機的升級。 執行 WordCount 應用程式的升級。

新版的應用程式現在只會計算以母音開頭的單字。 進行升級時，我們會看到應用程式的行為出現兩項變更。 第一，計數成長的速率應該變慢，因為計算的單字比較少。 第二，由於第一個資料分割有兩個母音 (A 和 E)，而其他每個資料分割只包含一個母音，其計數最後應該會開始超越其他資料分割。

1. [將 WordCount 第 2 版套件下載](http://aka.ms/servicefabric-wordcountappv2) 到您下載第 1 版套件的相同位置。
2. 返回 PowerShell 視窗並使用 SDK 的升級命令在叢集中註冊新版本。 然後開始升級 fabric:/WordCount 應用程式。
   
    ```powershell
    Publish-UpgradedServiceFabricApplication -ApplicationPackagePath C:\ServiceFabric\WordCountV2.sfpkg -ApplicationName "fabric:/WordCount" -UpgradeParameters @{"FailureAction"="Rollback"; "UpgradeReplicaSetCheckTimeout"=1; "Monitored"=$true; "Force"=$true}
    ```
   
    開始升級時，您應該會在 PowerShell 中看到下列輸出。
   
    ![在 PowerShell 中的升級進度][ps-appupgradeprogress]
3. 當升級正在進行時，您可能會發現從 Service Fabric 總管監視其狀態更加輕鬆。 啟動瀏覽器視窗並瀏覽至 [http://localhost:19080/Explorer](http://localhost:19080/Explorer)。 展開左邊樹狀目錄中的 [應用程式]，然後選擇 [WordCount]，最後選擇 [fabric:/WordCount]。 在 [基本功能] 索引標籤中，您會在升級繼續進行叢集的升級網域時的狀態。
   
    ![在 Service Fabric 總管中的升級進度][sfx-upgradeprogress]
   
    透過每個網域繼續升級時，系統會執行健康狀態檢查，以確保應用程式運作正常。
4. 如果您對 fabric:/WordCount 應用程式中的服務集合重新執行先前的查詢，請注意，雖然 WordCountService 版本已變更，但 WordCountWebService 版本維持不變：
   
    ```powershell
    Get-ServiceFabricService -ApplicationName 'fabric:/WordCount'
    ```
   
    ![在升級後查詢應用程式服務][ps-getsfsvc-postupgrade]
   
    此範例強調 Service Fabric 如何管理應用程式升級。 只會影響已變更的服務 (或這些服務中的程式碼/組態套件) 集合，讓升級程序變得更快速且更可靠。
5. 最後，返回瀏覽器來觀察新應用程式版本的行為。 如同預期，計數進度比較緩慢，而第一個磁碟分割最後會有稍微多一些的磁碟區。
   
    ![在瀏覽器中檢視新版的應用程式][deployed-app-ui-v2]

## <a name="cleaning-up"></a>清除
在我們做結論之前，請務必記得本機叢集是真實的。 應用程式會繼續在背景中執行，直到您將它們移除。  視應用程式的本質而定，執行中應用程式可能會佔用您電腦上的大量資源。 您有數個選項可管理應用程式和叢集：

1. 若要移除個別應用程式及其所有資料，請執行下列命令︰
   
    ```powershell
    Unpublish-ServiceFabricApplication -ApplicationName "fabric:/WordCount"
    ```
   
    或者，從 Service Fabric Explorer 的 [動作] 功能表或左窗格中應用程式清單檢視的內容功能表來刪除應用程式。
   
    ![在 Service Fabric 總管中刪除應用程式][sfe-delete-application]
2. 在叢集中刪除應用程式之後，將 WordCount 應用程式類型的 1.0.0 和 2.0.0 版取消註冊。 刪除作業會從叢集的映像存放區中移除應用程式套件，包括程式碼和設定。
   
    ```powershell
    Remove-ServiceFabricApplicationType -ApplicationTypeName WordCount -ApplicationTypeVersion 2.0.0
    Remove-ServiceFabricApplicationType -ApplicationTypeName WordCount -ApplicationTypeVersion 1.0.0
    ```
   
    或者，在 Service Fabric Explorer 中為應用程式選擇 [解除佈建類型]  。
3. 若要關閉叢集，但保留應用程式資料及追蹤，請按一下系統匣應用程式中的 [停止本機叢集]  。
4. 若要完全刪除叢集，請按一下系統匣應用程式中的 [移除本機叢集]  。 此選項會導致下次您在 Visual Studio 中按 F5 鍵時發生其他緩慢部署。 只有在您計劃一陣子不使用本機叢集或者需要回收資源時，才將本機叢集移除。

## <a name="one-node-and-five-node-cluster-mode"></a>1 個節點和 5 個節點叢集模式
開發應用程式時，您常會發現自己快速反覆撰寫程式碼、偵錯、變更程式碼和偵錯。 為了協助最佳化這個程序，本機叢集可以用兩種模式執行︰1 個節點或 5 節點。 這兩個叢集模式各有優點。 5 個節點叢集模式可讓您使用實際的叢集。 您可以測試容錯移轉案例，使用更多您的服務的執行個體和複本。 1 個節點叢集模式已最佳化，可進行服務的快速部署和註冊，進而協助您使用 Service Fabric 執行階段快速驗證程式碼。

1 個節點叢集或 5 個節點叢集模式皆不是模擬器。 本機開發叢集會執行在多部電腦的叢集上找到的相同平台程式碼。

> [!WARNING]
> 變更叢集模式時，目前的叢集會從您的系統移除，而建立新的叢集。 當您變更叢集模式時，在叢集中儲存的資料會遭到刪除。
> 
> 

若要將模式變更為 1 個節點叢集，請選取 Service Fabric 本機叢集管理員中的**交換器叢集模式**。

![切換叢集模式][switch-cluster-mode]

或者，您也可以使用 PowerShell 變更叢集模式︰

1. 以系統管理員身分啟動新的 PowerShell 視窗。
2. 從 SDK 資料夾執行叢集設定指令碼：
   
    ```powershell
    & "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\ClusterSetup\DevClusterSetup.ps1" -CreateOneNodeCluster
    ```
   
    叢集設定需要一些時間。 設定完成後，您應該會看到輸出類似於：
   
    ![叢集設定輸出][cluster-setup-success-1-node]

## <a name="next-steps"></a>後續步驟
* 您現在已部署並升級某些預先建置的應用程式，您可以 [嘗試在 Visual Studio 中建立自己的應用程式](service-fabric-create-your-first-application-in-visual-studio.md)。
* 本文中在本機叢集上執行的所有動作也可以在 [Azure 叢集](service-fabric-cluster-creation-via-portal.md) 上執行。
* 本文中執行的是基本升級。 若要深入了解 Service Fabric 升級的功用和彈性，請參閱 [升級文件](service-fabric-application-upgrade.md) 。

<!-- Images -->

[cluster-setup-success]: ./media/service-fabric-get-started-with-a-local-cluster/LocalClusterSetup.png
[extracted-app-package]: ./media/service-fabric-get-started-with-a-local-cluster/ExtractedAppPackage.png
[deploy-app-to-local-cluster]: ./media/service-fabric-get-started-with-a-local-cluster/DeployAppToLocalCluster.png
[deployed-app-ui]: ./media/service-fabric-get-started-with-a-local-cluster/DeployedAppUI-v1.png
[deployed-app-ui-v2]: ./media/service-fabric-get-started-with-a-local-cluster/DeployedAppUI-PostUpgrade.png
[sfx-app-instance]: ./media/service-fabric-get-started-with-a-local-cluster/SfxAppInstance.png
[sfx-two-app-instances-different-partitions]: ./media/service-fabric-get-started-with-a-local-cluster/SfxTwoAppInstances-DifferentPartitionCount.png
[ps-getsfapp]: ./media/service-fabric-get-started-with-a-local-cluster/PS-GetSFApp.png
[ps-getsfsvc]: ./media/service-fabric-get-started-with-a-local-cluster/PS-GetSFSvc.png
[ps-getsfpartitions]: ./media/service-fabric-get-started-with-a-local-cluster/PS-GetSFPartitions.png
[ps-appupgradeprogress]: ./media/service-fabric-get-started-with-a-local-cluster/PS-AppUpgradeProgress.png
[ps-getsfsvc-postupgrade]: ./media/service-fabric-get-started-with-a-local-cluster/PS-GetSFSvc-PostUpgrade.png
[sfx-upgradeprogress]: ./media/service-fabric-get-started-with-a-local-cluster/SfxUpgradeOverview.png
[sfx-service-overview]: ./media/service-fabric-get-started-with-a-local-cluster/sfx-service-overview.png
[sfe-delete-application]: ./media/service-fabric-get-started-with-a-local-cluster/sfe-delete-application.png
[cluster-setup-success-1-node]: ./media/service-fabric-get-started-with-a-local-cluster/cluster-setup-success-1-node.png
[switch-cluster-mode]: ./media/service-fabric-get-started-with-a-local-cluster/switch-cluster-mode.png
