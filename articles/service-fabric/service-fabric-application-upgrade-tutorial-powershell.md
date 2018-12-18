---
title: 使用 PowerShell 進行 Service Fabric 應用程式升級 | Microsoft Docs
description: 本文會逐步解說使用 PowerShell 來部署 Service Fabric 應用程式、變更程式碼及執行升級的體驗。
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: ''
ms.assetid: 9bc75748-96b0-49ca-8d8a-41fe08398f25
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: subramar
ms.openlocfilehash: c1005d60df0b1cfd3b24be954ab4ff1b18c8f7a8
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44348764"
---
# <a name="service-fabric-application-upgrade-using-powershell"></a>使用 PowerShell 進行 Service Fabric 應用程式升級
> [!div class="op_single_selector"]
> * [PowerShell](service-fabric-application-upgrade-tutorial-powershell.md)
> * [Visual Studio](service-fabric-application-upgrade-tutorial.md)
> 
> 

<br/>

最常使用和建議的升級方法是監視輪流升級。  Azure Service Fabric 會根據健康狀態原則集，監視正在升級之應用程式的健康狀態。 當更新網域 (UD) 中的應用程式升級之後，Service Fabric 會評估應用程式健康狀態，並根據健康狀態原則繼續進行下一個更新網域或讓升級失敗。

可以使用受控或原生 API、PowerShell、Azure CLI、Java 或 REST，執行監視應用程式升級。 如需有關使用 Visual Studio 來執行升級的說明，請參閱 [使用 Visual Studio 升級您的應用程式](service-fabric-application-upgrade-tutorial.md)。

透過 Service Fabric 監視輪流升級，應用程式系統管理員即可設定 Service Fabric 用來判斷應用程式健康狀態良好的健康狀態評估原則。 此外，系統管理員也可設定當健康狀態評估失敗時採取的動作 (例如，進行自動回復)。本節會逐步解說使用 PowerShell 對其中一個 SDK 範例進行受監視的升級。 下列 Microsoft Virtual Academy 影片也會逐步引導您完成應用程式升級︰<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=OrHJH66yC_6406218965">
<img src="./media/service-fabric-application-upgrade-tutorial-powershell/AppLifecycleVid.png" WIDTH="360" HEIGHT="244">
</a></center>

## <a name="step-1-build-and-deploy-the-visual-objects-sample"></a>步驟 1：建置和部署視覺物件範例
在應用程式專案 **VisualObjectsApplication** 上按一下滑鼠右鍵，然後選取 [發佈] 命令來建置和發佈應用程式。  如需詳細資訊，請參閱 [Service Fabric 應用程式升級教學課程](service-fabric-application-upgrade-tutorial.md)。  或者，您可以使用 PowerShell 來部署您的應用程式。

> [!NOTE]
> 在 PowerShell 中使用任何 Service Fabric 命令之前，您必須先使用 `Connect-ServiceFabricCluster` Cmdlet 連接到叢集。 同樣地，它會假設已經在本機電腦上設定叢集。 請參閱 [設定 Service Fabric 開發環境](service-fabric-get-started.md)上的文章
> 
> 

在 Visual Studio 中建置專案後，您可以使用 PowerShell 命令 [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage) 將應用程式封裝複製到 ImageStore。 如果您想要在本機確認應用程式套件，使用 [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage) cmdlet。 下一個步驟是使用 [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype) Cmdlet 將應用程式註冊至 Service Fabric 執行階段。 下一個步驟是使用 [New-ServiceFabricApplication](/powershell/module/servicefabric/new-servicefabricapplication?view=azureservicefabricps) Cmdlet 啟動應用程式的執行個體。  這三個步驟類似於在 Visual Studio 中使用 [部署]  功能表項目。  一旦佈建完成，您應該清除從映像存放區複製的應用程式套件，以減少耗用的資源。  如果不再需要應用程式類型，應該基於相同原因將其取消註冊。 如需詳細資訊，請參閱[使用 PowerShell 部署與移除應用程式](service-fabric-application-upgrade-tutorial-powershell.md)。

現在，您可以使用 [Service Fabric Explorer 來檢視叢集與應用程式](service-fabric-visualizing-your-cluster.md)。 該應用程式有一個 Web 服務，透過在 Internet Explorer 的網址列中輸入 [http://localhost:8081/visualobjects](http://localhost:8081/visualobjects) ，即可瀏覽至該服務。  您應該會在畫面上看到一些浮動視覺物件四處移動。  此外，您可以使用 [Get-ServiceFabricApplication](/powershell/module/servicefabric/get-servicefabricapplication?view=azureservicefabricps) 檢查應用程式狀態。

## <a name="step-2-update-the-visual-objects-sample"></a>步驟 2：更新視覺物件範例
您可能會注意到在步驟 1 中已部署的版本，視覺物件不會旋轉。 讓我們將這個應用程式升級到其中的視覺物件也會旋轉的版本。

選取 VisualObjects 解決方案內的 VisualObjects.ActorService 專案，然後開啟 StatefulVisualObjectActor.cs 檔案。 在該檔案內，瀏覽至 `MoveObject` 方法，然後將 `this.State.Move()` 標記為註解，然後將 `this.State.Move(true)` 取消註解。 這項變更會在服務升級後旋轉物件。

我們也需要更新 *VisualObjects.ActorService* 專案的 **ServiceManifest.xml**檔案 (在 [PackageRoot] 底下)。 請將 *ServiceManifest.xml* 檔案中的 *CodePackage* 和服務版本及對應的行更新成 2.0。
您可以在對解決方案按一下滑鼠右鍵之後，使用 Visual Studio [編輯資訊清單檔案]  選項來進行資訊清單檔案變更。

進行變更之後，資訊清單應該會看起來如下 (醒目提示的部分即為所做的變更)：

```xml
<ServiceManifestName="VisualObjects.ActorService" Version="2.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

<CodePackageName="Code" Version="2.0">
```

現在，*ApplicationManifest.xml* 檔案 (可以在 **VisualObjects** 方案下的 **VisualObjects** 專案下找到)，會更新為 2.0 版的 **VisualObjects.ActorService** 專案。 此外，應用程式版本也會從 1.0.0.0 更新為 2.0.0.0。 *ApplicationManifest.xml* 看起來應該類似下列程式碼片段︰

```xml
<ApplicationManifestxmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VisualObjects" ApplicationTypeVersion="2.0.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">

 <ServiceManifestRefServiceManifestName="VisualObjects.ActorService" ServiceManifestVersion="2.0" />
```

現在只選取 [ActorService] 專案，然後以滑鼠右鍵按一下並選取 Visual Studio 中的 [組建] 選項建置專案。 如果您選取 [全部重建]，因為程式碼已變更，所以您要更新所有專案的版本。 接下來，在 [VisualObjectsApplication] 上按一下滑鼠右鍵，選取 [Service Fabric] 功能表，然後選擇 [封裝]，來封裝已更新的應用程式。 這個動作會建立可部署的應用程式封裝。  更新的應用程式已準備好進行部署。

## <a name="step-3--decide-on-health-policies-and-upgrade-parameters"></a>步驟 3：決定健康狀態原則並升級參數
請您熟悉[應用程式升級參數](service-fabric-application-upgrade-parameters.md)和[升級程序](service-fabric-application-upgrade.md)，以了解套用的各種升級參數、逾時和健康狀態準則。 對於此逐步解說，服務健康狀態評估準則會設定為預設值 (和建議值)，這表示升級之後，所有服務和執行個體應該是「健康狀態良好」  。  

但是，讓我們將 *HealthCheckStableDuration* 增加為 60 秒 (如此一來，在升級繼續至下一個更新網域之前，至少有 20 秒的時間服務是健康狀態良好的)。  同時也要將 *UpgradeDomainTimeout* 設為 1200 秒，將 *UpgradeTimeout* 設為 3000 秒。

最後，我們也將 *UpgradeFailureAction* 設定為回復。 此選項要求 Service Fabric 在升級期間如果遇到任何問題時要將應用程式回復為舊版。 因此在啟動升級 (在步驟 4) 時，會指定下列參數︰

FailureAction = Rollback

HealthCheckStableDurationSec = 60

UpgradeDomainTimeoutSec = 1200

UpgradeTimeout = 3000

## <a name="step-4-prepare-application-for-upgrade"></a>步驟 4：準備應用程式以進行升級
現在，應用程式已建置並且準備好進行升級。 如果您以系統管理員身分開啟 PowerShell 視窗並且輸入 [Get-ServiceFabricApplication](/powershell/module/servicefabric/get-servicefabricapplication?view=azureservicefabricps)，它應該會讓您知道它是已部署之 **VisualObjects** 的應用程式類型 1.0.0.0。  

應用程式封裝儲存在以下的相對路徑，您在其中解壓縮 Service Fabric SDK - *Samples\Services\Stateful\VisualObjects\VisualObjects\obj\x64\Debug*。 您應該會在該目錄中找到 "Package" 資料夾，這是應用程式封裝儲存的位置。 檢查時間戳記以確保它是最新組建 (您也可能需要適當地修改路徑)。

現在讓我們將更新的應用程式封裝複製到 Service Fabric ImageStore (Service Fabric 在其中儲存應用程式封裝)。 參數 *ApplicationPackagePathInImageStore* 會通知 Service Fabric 可以在哪裡找到應用程式封裝。 使用下列命令將更新的應用程式放在 "VisualObjects\_V2" (您可能需要再次適當地修改路徑)。

```powershell
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath .\Samples\Services\Stateful\VisualObjects\VisualObjects\obj\x64\Debug\Package -ApplicationPackagePathInImageStore "VisualObjects\_V2"
```

下一步是向 Service Fabric 註冊此應用程式，這可以使用 [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype?view=azureservicefabricps) 命令來執行：

```powershell
Register-ServiceFabricApplicationType -ApplicationPathInImageStore "VisualObjects\_V2"
```

如果上述命令不成功，很可能是您需要重建所有服務。 如步驟 2 中所述，您可能也必須更新您的 WebService 版本。

建議您在成功註冊應用程式之後，移除應用程式套件。  從映像存放區刪除應用程式套件會釋放系統資源。  保留未使用的應用程式套件會耗用磁碟儲存空間，並會導致應用程式效能問題。

```powershell
Remove-ServiceFabricApplicationPackage -ApplicationPackagePathInImageStore "VisualObjects\_V2" -ImageStoreConnectionString fabric:ImageStore
```

## <a name="step-5-start-the-application-upgrade"></a>步驟 5：開始應用程式升級
現在，您已準備好使用 [Start-ServiceFabricApplicationUpgrade](/powershell/module/servicefabric/start-servicefabricapplicationupgrade?view=azureservicefabricps)命令來啟動應用程式升級：

```powershell
Start-ServiceFabricApplicationUpgrade -ApplicationName fabric:/VisualObjects -ApplicationTypeVersion 2.0.0.0 -HealthCheckStableDurationSec 60 -UpgradeDomainTimeoutSec 1200 -UpgradeTimeout 3000   -FailureAction Rollback -Monitored
```


應用程式名稱如 *ApplicationManifest.xml* 檔案中所述。 Service Fabric 會使用這個名稱來識別要升級哪一個應用程式。 如果您設定的逾時太短，您可能會遇到失敗訊息，指出此問題。 請參閱疑難排解章節，或增加逾時。

現在，當應用程式升級進行時，您可以使用 Service Fabric Explorer 或 [Get-ServiceFabricApplicationUpgrade](/powershell/module/servicefabric/get-servicefabricapplicationupgrade?view=azureservicefabricps) PowerShell 命令監視進度。 

```powershell
Get-ServiceFabricApplicationUpgrade fabric:/VisualObjects
```

幾分鐘後，使用上述 PowerShell 命令所取得的狀態應該會顯示所有更新網域已升級 (完成)。 而且，您應該會在瀏覽器視窗中發現該視覺物件已經開始旋轉！

您可以嘗試從第 2 版升級到第 3 版，或練習從第 2 版升級為第 1 版。 從第 2 版移至第 1 版也視為一種升級。 練習逾時和健康狀態原則，讓自己更熟練。 當您部署至 Azure 叢集時，參數必須正確設定。 最好保守地設定逾時。

## <a name="next-steps"></a>後續步驟
[使用 Visual Studio 升級您的應用程式](service-fabric-application-upgrade-tutorial.md) 將引導您完成使用 Visual Studio 進行應用程式升級的步驟。

使用 [升級參數](service-fabric-application-upgrade-parameters.md)來控制您應用程式的升級方式。

了解如何使用 [資料序列化](service-fabric-application-upgrade-data-serialization.md)，以讓您的應用程式升級相容。

參考 [進階主題](service-fabric-application-upgrade-advanced.md)，以了解如何在升級您的應用程式時使用進階功能。

參考 [疑難排解應用程式升級](service-fabric-application-upgrade-troubleshooting.md)中的步驟，以修正應用程式升級中常見的問題。

