---
title: 對 Azure Service Fabric 應用程式進行封裝 | Microsoft Docs
description: 如何在將 Service Fabric 應用程式部署至叢集之前對它進行封裝。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: mani-ramaswamy
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: ryanwi
ms.openlocfilehash: 24cb1fd0666b404d92dfb803f55c850226ff59b6
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34205805"
---
# <a name="package-an-application"></a>封裝應用程式
本文說明如何對 Service Fabric 應用程式進行封裝，並使其準備好進行部署。

## <a name="package-layout"></a>封裝版面配置
應用程式資訊清單、一或多個服務資訊清單及其他必要的封裝檔案必須以特定的配置組織後，才能部署到 Service Fabric 叢集。 在本文中的範例資訊清單必須組織成下列目錄結構：

```
PS D:\temp> tree /f .\MyApplicationType

D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat
```

命名資料夾以符合每個對應元素的 **名稱** 屬性。 例如，如果服務資訊清單包含名稱為 **MyCodeA** 和 **MyCodeB** 的兩個程式碼封裝，則同名的兩個資料夾會包含每個程式碼封裝所需的二進位檔。

## <a name="use-setupentrypoint"></a>使用 SetupEntryPoint
使用 **SetupEntryPoint** 的一般案例，是當您必須在服務啟動之前執行可執行檔，或必須使用提高的權限來執行作業時。 例如︰

* 設定及初始化服務可執行檔需要的環境變數。 這不僅限於透過 Service Fabric 程式設計模型撰寫的可執行檔。 例如，npm.exe 部署 node.js 應用程式，需要設定某些環境變數。
* 透過安裝安全性憑證設定存取控制。

如需有關如何設定 **SetupEntryPoint** 的詳細資訊，請參閱[設定服務設定進入點的原則](service-fabric-application-runas-security.md)

<a id="Package-App"></a>
## <a name="configure"></a>設定
### <a name="build-a-package-by-using-visual-studio"></a>使用 Visual Studio 建置封裝
如果您使用 Visual Studio 2015 來建立您的應用程式，您可以使用 [封裝] 命令來自動建立符合上述版面配置的封裝。

若要建立封裝，請以滑鼠右鍵按一下方案總管中的應用程式專案，然後選擇 [封裝] 命令，如下所示：

![使用 Visual Studio 封裝應用程式][vs-package-command]

封裝完成時，您會在 [輸出]  視窗中找到封裝的位置。 當您在 Visual Studio 中部署或偵錯應用程式時，封裝步驟會自動執行。

### <a name="build-a-package-by-command-line"></a>透過命令列建置封裝
使用 `msbuild.exe` 以程式設計方式封裝您的應用程式也是可行的。 深究其背後的原理，由於 Visual Studio 執行它，因此輸出也會相同。

```shell
D:\Temp> msbuild HelloWorld.sfproj /t:Package
```

## <a name="test-the-package"></a>測試封裝
您可以使用 [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage?view=azureservicefabricps) 命令，透過 PowerShell 在本機上驗證封裝結構。
此命令會檢查有無資訊清單剖析問題，並驗證所有參考。 這個命令只會驗證封裝中目錄與檔案的結構正確性。
除了檢查所有必要檔案是否都在之外，它不會驗證任何程式碼或資料封裝內容。

```
PS D:\temp> Test-ServiceFabricApplicationPackage .\MyApplicationType
False
Test-ServiceFabricApplicationPackage : The EntryPoint MySetup.bat is not found.
FileName: C:\Users\servicefabric\AppData\Local\Temp\TestApplicationPackage_7195781181\nrri205a.e2h\MyApplicationType\MyServiceManifest\ServiceManifest.xml
```

這個錯誤顯示程式碼封裝中遺漏服務資訊清單 *SetupEntryPoint* 中參考的 **MySetup.bat** 檔案。 加入遺漏的檔案之後，應用程式驗證就會通過：

```
PS D:\temp> tree /f .\MyApplicationType

D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │       MySetup.bat
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat

PS D:\temp> Test-ServiceFabricApplicationPackage .\MyApplicationType
True
PS D:\temp>
```

如果您的應用程式已定義[應用程式參數](service-fabric-manage-multiple-environment-app-configuration.md)，您可以在 [Test-ServiceFabricApplicationPackage (英文)](/powershell/module/servicefabric/test-servicefabricapplicationpackage?view=azureservicefabricps) 中傳遞它們以進行適當的驗證。

如果您知道將部署應用程式的叢集，建議您傳送 `ImageStoreConnectionString` 參數。 在此情況下，該封裝也會針對已在叢集中執行的舊版應用程式進行驗證。 例如，驗證可以偵測具有相同版本但不同內容的封裝是否已經部署。  

一旦應用程式正確封裝好並通過驗證，請考慮壓縮封裝以便能更快速地進行部署作業。

## <a name="compress-a-package"></a>壓縮封裝
當封裝很大或有許多檔案時，您可以壓縮它以加快部署速度。 壓縮會減少檔案數目並降低封裝大小。
如果是壓縮的應用程式封裝，[上傳應用程式封裝](service-fabric-deploy-remove-applications.md#upload-the-application-package)可能會比上傳未壓縮的封裝花費更長的時間，尤其當壓縮作業是在複製過程中進行時，更是如此。 使用壓縮，[註冊應用程式類型](service-fabric-deploy-remove-applications.md#register-the-application-package)和[取消註冊應用程式類型](service-fabric-deploy-remove-applications.md#unregister-an-application-type)會比較快。

已壓縮及未壓縮套件的部署機制皆相同。 如果封裝已壓縮，它會以壓縮的形式儲存在叢集映像存放區中，並在應用程式執行之前於節點上解壓縮。
壓縮會將有效的 Service Fabric 封裝以壓縮的版本取代。 該資料夾必須允許寫入權限。 在已經壓縮的封裝上執行壓縮將不會產生任何變化。

您可以執行 PowerShell 命令 [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps)，並搭配 `CompressPackage` 參數來壓縮封裝。 您可以使用相同的命令，並搭配 `UncompressPackage` 參數來將封裝解壓縮。

下列命令會在不將封裝複製到映像存放區的情況下壓縮封裝。 您可以在不搭配 `SkipCopy` 旗標的情況下使用 [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps)，視需求將壓縮的封裝複製到一或多個 Service Fabric 叢集。
該套件現在包含 `code`、`config` 及 `data` 套件的 ZIP 壓縮檔案。 因為許多內部作業需要，所以應用程式資訊清單和服務資訊清單不會經過壓縮。 例如，某些驗證的封裝共用作業、應用程式類型名稱和版本擷取作業都需要存取資訊清單。 對資訊清單進行壓縮，將會使這些作業效率不佳。

```
PS D:\temp> tree /f .\MyApplicationType

D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │       MySetup.bat
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat
PS D:\temp> Copy-ServiceFabricApplicationPackage -ApplicationPackagePath .\MyApplicationType -CompressPackage -SkipCopy

PS D:\temp> tree /f .\MyApplicationType

D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
       ServiceManifest.xml
       MyCode.zip
       MyConfig.zip
       MyData.zip

```

除此之外，您可以使用 [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps)，同時壓縮並複製封裝。
如果封裝很大，請提供足夠的逾時，使封裝可以完成壓縮並上傳至叢集。
```
PS D:\temp> Copy-ServiceFabricApplicationPackage -ApplicationPackagePath .\MyApplicationType -ApplicationPackagePathInImageStore MyApplicationType -ImageStoreConnectionString fabric:ImageStore -CompressPackage -TimeoutSec 5400
```

Service Fabric 會在內部針對驗證計算應用程式封裝的總和檢查碼。 使用壓縮時，會針對每個封裝的壓縮版本計算總和檢查碼。 從同一個應用程式封裝產生新的 zip 會建立不同的總和檢查碼。 若要避免驗證錯誤，請使用[差異佈建](service-fabric-application-upgrade-advanced.md)。 如果使用此選項，請勿在新版本中包含沒有變更的封裝， 而是要從新的服務資訊清單直接參考這些封裝。

如果無法使用差異佈建而又必須包含封裝，請產生新版本的 `code`、`config` 和 `data` 封裝，以避免總和檢查碼不符。 如果使用壓縮的封裝，不論先前的版本是否使用壓縮，也必須為沒有變更的封裝產生新的版本。

封裝現已正確完成封裝、驗證及壓縮 (若有需要)，因此已準備好[部署](service-fabric-deploy-remove-applications.md)至一或多個 Service Fabric 叢集。

### <a name="compress-packages-when-deploying-using-visual-studio"></a>使用 Visual Studio 進行部署時壓縮套件
您可以將 `CopyPackageParameters` 元素新增到發行設定檔，並將 `CompressPackage` 屬性設定為 `true`，來指示 Visual Studio 在部署時壓縮套件。

``` xml
    <PublishProfile xmlns="http://schemas.microsoft.com/2015/05/fabrictools">
        <ClusterConnectionParameters ConnectionEndpoint="mycluster.westus.cloudapp.azure.com" />
        <ApplicationParameterFile Path="..\ApplicationParameters\Cloud.xml" />
        <CopyPackageParameters CompressPackage="true"/>
    </PublishProfile>
```

## <a name="create-an-sfpkg"></a>建立 sfpkg
從 6.1 版開始，Service Fabric 允許從外部存放區佈建。
如果使用此選項，就不需要將應用程式封裝複製到映像存放區。 您可以改為建立 `sfpkg` 並將它上傳到外部存放區，然後在佈建時對 Service Fabric 提供下載 URI。 同一個封裝可以佈建到多個叢集。 從外部存放區佈建可節省將封裝複製到每個叢集所需的時間。

`sfpkg` 檔案是一個包含初始應用程式封裝的 zip，副檔名為 ".sfpkg"。
在 zip 中，應用程式封裝可能是已經過壓縮或未壓縮的。 zip 內應用程式封裝的壓縮是在 code、config 和 data 封裝層級完成，如[先前所述](service-fabric-package-apps.md#compress-a-package)。

若要建立 `sfpkg`，請先建立一個包含原始已壓縮或未壓縮應用程式封裝的資料夾。 然後，使用任何公用程式，以副檔名 ".sfpkg" 壓縮資料夾。 例如，使用 [ZipFile.CreateFromDirectory](https://msdn.microsoft.com/library/hh485721(v=vs.110).aspx)。

```csharp
ZipFile.CreateFromDirectory(appPackageDirectoryPath, sfpkgFilePath);
```

`sfpkg` 必須以頻外方式上傳至 Service Fabric 以外的外部存放區。 外部存放區可以是公開 REST http 或 https 端點的任何存放區。 佈建時，Service Fabric 會執行 GET 作業下載 `sfpkg` 應用程式封裝，所以存放區必須允許封裝的「讀取」權限。

若要佈建套件，請使用外部佈建，這會需要下載 URI 和應用程式類型資訊。

>[!NOTE]
> 根據映像存放區相對路徑的佈建目前不支援 `sfpkg` 檔案。 因此，不應該將 `sfpkg` 複製到映像存放區。

## <a name="next-steps"></a>後續步驟
[部署與移除應用程式][10]說明如何使用 PowerShell 來管理應用程式執行個體。

[管理多個環境的應用程式參數][11]說明如何為不同的應用程式執行個體設定參數和環境變數。

[設定應用程式的安全性原則][12]說明如何依據安全性原則執行服務來限制存取。

<!--Image references-->
[vs-package-command]: ./media/service-fabric-package-apps/vs-package-command.png

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: service-fabric-deploy-remove-applications.md
[11]: service-fabric-manage-multiple-environment-app-configuration.md
[12]: service-fabric-application-runas-security.md
