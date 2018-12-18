---
title: 在 Windows Server 上升級獨立的 Azure Service Fabric 叢集 | Microsoft Docs
description: 升級執行獨立 Azure Service Fabric 叢集的 Service Fabric 程式碼和/或組態，包括設定叢集更新模式。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: 66296cc6-9524-4c6a-b0a6-57c253bdf67e
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/15/2017
ms.author: dekapur
ms.openlocfilehash: 93b79b7adacdec18912d28bb9725e2dc77737d59
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43050758"
---
# <a name="upgrade-your-standalone-azure-service-fabric-cluster-on-windows-server"></a>在 Windows Server 上升級獨立的 Azure Service Fabric 叢集 
> [!div class="op_single_selector"]
> * [Azure 叢集](service-fabric-cluster-upgrade.md)
> * [獨立叢集](service-fabric-cluster-upgrade-windows-server.md)
>
>

對於現代化系統來說，升級能力攸關產品是否能長期成功。 Azure Service Fabric 叢集是您擁有的資源。 本文說明如何確定叢集一律會執行支援版本的 Service Fabric 程式碼和組態的方法。

## <a name="control-the-service-fabric-version-that-runs-on-your-cluster"></a>控制叢集上執行的 Service Fabric 版本
若要設定叢集，讓其在 Microsoft 發行新版本時下載 Service Fabric 更新，請將 fabricClusterAutoupgradeEnabled 叢集組態設定為 true。 若要選取要讓叢集執行的受支援 Service Fabric 版本，請將 fabricClusterAutoupgradeEnabled 叢集組態設定為 false。

> [!NOTE]
> 請確定您的叢集一律執行支援的 Service Fabric 版本。 當 Microsoft 宣布發行新版本的 Service Fabric 時，從宣布當日起至少 60 天後，舊版就會標示為結束支援。 新的版本會於 [Service Fabric 小組部落格上](https://blogs.msdn.microsoft.com/azureservicefabric/)發佈。 那時就有新的版本可選擇。
>
>

只有當您使用生產樣式節點組態，其中每個 Service Fabric 節點是配置在不同實體或虛擬機器時，才能將叢集升級到新的版本。 如果您擁有的開發叢集在單一的實體或虛擬機器上有多個 Service Fabric 節點，您必須以新版本重新建立叢集。

兩個不同的工作流程可以將您的叢集升級至最新版本或支援的 Service Fabric 版本。 一個工作流程適用於具備連線能力而可自動下載最新版本的叢集。 另一個工作流程則適用於不具備連線能力因而無法下載最新 Service Fabric 版本的叢集。

### <a name="upgrade-clusters-that-have-connectivity-to-download-the-latest-code-and-configuration"></a>升級具有連線能力而可下載最新程式碼和組態的叢集
如果您的叢集節點與 [Microsoft 下載中心](http://download.microsoft.com)的網際網路連線，則可以使用這些步驟將您的叢集升級至支援的版本。

對於可以連線至 [Microsoft 下載中心](http://download.microsoft.com)的叢集，Microsoft 會定期檢查新版本 Service Fabric 的可用性。

當新的 Service Fabric 版本可用時，系統會將套件下載至本機叢集並佈建以進行升級。 除了通知客戶這個新版本之外，系統會顯示明確的叢集健全狀況警告，如下所示：

「現行的叢集版本 [version#] 將於 [date] 結束支援。」

叢集執行最新版本後，警告就會消失。

#### <a name="cluster-upgrade-workflow"></a>叢集升級工作流程
當您看到叢集健全狀況警告時，請執行下列操作︰

1. 從具有系統管理員存取權的任何電腦，將叢集連接至列為叢集中節點的所有電腦。 執行此指令碼所在的電腦不一定是叢集的一部分。

    ```powershell

    ###### connect to the secure cluster using certs
    $ClusterName= "mysecurecluster.something.com:19000"
    $CertThumbprint= "70EF5E22ADB649799DA3C8B6A6BF7FG2D630F8F3"
    Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
        -X509Credential `
        -ServerCertThumbprint $CertThumbprint  `
        -FindType FindByThumbprint `
        -FindValue $CertThumbprint `
        -StoreLocation CurrentUser `
        -StoreName My
    ```

2. 取得您可以升級的 Service Fabric 版本清單。

    ```powershell

    ###### Get the list of available Service Fabric versions
    Get-ServiceFabricRegisteredClusterCodeVersion
    ```

    您應該會看到如下的輸出：

    ![取得 Service Fabric 版本][getfabversions]
3. 使用 [Start-ServiceFabricClusterUpgrade](https://docs.microsoft.com/powershell/module/servicefabric/start-servicefabricclusterupgrade) Windows PowerShell 命令開始將叢集升級為可用版本。

    ```Powershell

    Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion <codeversion#> -Monitored -FailureAction Rollback

    ###### Here is a filled-out example

    Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion 5.3.301.9590 -Monitored -FailureAction Rollback

    ```
   若要監視升級的進度，您可以使用 Service Fabric Explorer 或執行下列 PowerShell 命令：

    ```powershell

    Get-ServiceFabricClusterUpgrade
    ```

    如果不符合叢集健康狀態原則，則會回復升級。 若要為 Start-ServiceFabricClusterUpgrade 命令指定自訂的健康狀態原則，請參閱 [Start-ServiceFabricClusterUpgrade](https://docs.microsoft.com/powershell/module/servicefabric/start-servicefabricclusterupgrade) 的文件。

    在解決導致復原的問題後，請依照先前所述的相同步驟再次起始升級。

### <a name="upgrade-clusters-that-have-no-connectivity-to-download-the-latest-code-and-configuration"></a>升級*沒有連線能力*因而無法下載最新程式碼和組態的叢集
如果您的叢集節點未與 [Microsoft 下載中心](http://download.microsoft.com)的網際網路連線，則可以使用這些步驟將您的叢集升級至支援的版本。

> [!NOTE]
> 如果您正在執行的叢集未連線至網際網路，您必須關注 Service Fabric 小組部落格，以便得知新版本的消息。 系統不會顯示叢集健康狀態警告來提醒您有新版本。  
>
>

#### <a name="auto-provisioning-vs-manual-provisioning"></a>自動佈建與手動佈建
若要啟用自動下載與註冊最新版程式碼的功能，請設定 Service Fabric 更新服務。 如需指示，請參閱[獨立套件](service-fabric-cluster-standalone-package-contents.md)中的 Tools\ServiceFabricUpdateService.zip\Readme_InstructionsAndHowTos.txt。
如需進行手動程序，請遵循以下指示。

請修改叢集組態以將下列屬性設定為 false，再開始進行組態升級：

        "fabricClusterAutoupgradeEnabled": false,

如需使用方式的詳細資料，請參閱 [Start-ServiceFabricClusterConfigurationUpgrade PowerShell command](https://docs.microsoft.com/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade)。 在您開始組態升級之前，請確實更新 JSON 中的 'clusterConfigurationVersion'。

```powershell

    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>

```

#### <a name="cluster-upgrade-workflow"></a>叢集升級工作流程

1. 從叢集中的其中一個節點執行 [Get-ServiceFabricClusterUpgrade](https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricclusterupgrade)，並記下 *TargetCodeVersion*。

2. 從與網際網路連線的電腦執行下列程式碼，以根據目前版本列出所有升級相容版本 ，並從相關聯的下載連結下載對應封裝：

    ```powershell

    ###### Get list of all upgrade compatible packages  
    Get-ServiceFabricRuntimeUpgradeVersion -BaseVersion <TargetCodeVersion as noted in Step 1> 
    ```

3. 從具有系統管理員存取權的任何電腦，將叢集連接至列為叢集中節點的所有電腦。 執行此指令碼所在的電腦不一定是叢集的一部分。

    ```powershell

   ###### Get the list of available Service Fabric versions
    Copy-ServiceFabricClusterPackage -Code -CodePackagePath <name of the .cab file including the path to it> -ImageStoreConnectionString "fabric:ImageStore"

   ###### Here is a filled-out example
    Copy-ServiceFabricClusterPackage -Code -CodePackagePath .\MicrosoftAzureServiceFabric.5.3.301.9590.cab -ImageStoreConnectionString "fabric:ImageStore"

    ```
4. 將下載的套件複製到叢集映像存放區。

5. 註冊所複製的套件。

    ```powershell

    ###### Get the list of available Service Fabric versions
    Register-ServiceFabricClusterPackage -Code -CodePackagePath <name of the .cab file>

    ###### Here is a filled-out example
    Register-ServiceFabricClusterPackage -Code -CodePackagePath .\MicrosoftAzureServiceFabric.5.3.301.9590.cab

     ```
6. 開始將叢集升級為可用版本。

    ```Powershell

    Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion <codeversion#> -Monitored -FailureAction Rollback

    ###### Here is a filled-out example
    Start-ServiceFabricClusterUpgrade -Code -CodePackageVersion 5.3.301.9590 -Monitored -FailureAction Rollback

    ```
   您可以在 Service Fabric Explorer 上或執行下列 PowerShell 命令來監視升級進度：

    ```powershell

    Get-ServiceFabricClusterUpgrade
    ```

    如果不符合叢集健康狀態原則，則會回復升級。 若要為 Start-ServiceFabricClusterUpgrade 命令指定自訂的健康狀態原則，請參閱 [Start-ServiceFabricClusterUpgrade](https://docs.microsoft.com/powershell/module/servicefabric/start-servicefabricclusterupgrade) 的文件。

    在解決導致復原的問題後，請依照先前所述的相同步驟再次起始升級。


## <a name="upgrade-the-cluster-configuration"></a>升級叢集組態
起始組態升級之前，您可以執行獨立套件中的 PowerShell 指令碼來對新叢集組態 JSON 進行測試：

```powershell

    TestConfiguration.ps1 -ClusterConfigFilePath <Path to the new Configuration File> -OldClusterConfigFilePath <Path to the old Configuration File>

```
或是使用此指令碼：

```powershell

    TestConfiguration.ps1 -ClusterConfigFilePath <Path to the new Configuration File> -OldClusterConfigFilePath <Path to the old Configuration File> -FabricRuntimePackagePath <Path to the .cab file which you want to test the configuration against>

```

某些組態無法升級，例如端點、叢集名稱、節點 IP 等。新的叢集組態 JSON 會對照舊的叢集組態 JSON 來進行測試，如有任何問題，就會在 PowerShell 視窗中擲回錯誤。

若要升級叢集組態升級，請執行 [Start-ServiceFabricClusterConfigurationUpgrade](https://docs.microsoft.com/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade)。 組態升級是按升級網域逐一處理。

```powershell

    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>

```

### <a name="cluster-certificate-config-upgrade"></a>叢集憑證組態升級  
叢集憑證可用於叢集節點之間的驗證。 在進行憑證變換時請格外小心，因為若發生失敗，將會讓叢集節點之間無法通訊。

從技術角度來看，有四個支援選項：  

* 單一憑證升級：升級路徑為「憑證 A (主要) -> 憑證 B (主要) -> 憑證 C (主要) -> ...」。

* 雙重憑證升級：升級路徑為「憑證 A (主要) -> 憑證 A (主要) 和 B (次要) -> 憑證 B (主要) -> 憑證 B (主要) 和 C (次要) -> 憑證 C (主要) -> ...」。

* 憑證類型升級：指紋型憑證設定 <-> CommonName 型憑證設定。 例如，憑證指紋 A (主要) 和指紋 B (次要) -> 憑證 CommonName C。

* 憑證簽發者指紋升級：升級路徑為「憑證 CN=A,IssuerThumbprint=IT1 (主要) -> 憑證 CN=A,IssuerThumbprint=IT1,IT2 (主要) -> 憑證 CN=A,IssuerThumbprint=IT2 (主要)」。


## <a name="next-steps"></a>後續步驟
* 了解如何自訂一些 [Service Fabric 叢集設定](service-fabric-cluster-fabric-settings.md)。
* 了解如何[相應放大和相應縮小叢集](service-fabric-cluster-scale-up-down.md)。
* 了解[應用程式升級](service-fabric-application-upgrade.md)。

<!--Image references-->
[getfabversions]: ./media/service-fabric-cluster-upgrade-windows-server/getfabversions.PNG
