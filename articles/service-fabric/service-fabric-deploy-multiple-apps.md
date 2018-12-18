---
title: 將使用 MongoDB 的 Node.js 應用程式部署至 Azure Service Fabric | Microsoft Docs
description: 如何封裝多個來賓可執行檔以部署至 Azure Service Fabric 叢集的逐步解說
services: service-fabric
documentationcenter: .net
author: mikkelhegn
manager: timlt
editor: ''
ms.assetid: b76bb756-c1ba-49f9-9666-e9807cf8f92f
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/23/2018
ms.author: mikhegn
ms.openlocfilehash: 9a7ab3881cd1058a60ff7d5f6e50c296f042e76e
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34206074"
---
# <a name="deploy-multiple-guest-executables"></a>部署多個來賓可執行檔
本文說明如何封裝多個來賓可執行檔並部署至 Azure Service Fabric。 若要建置和部署單一 Service Fabric 套件，請閱讀如何[將來賓可執行檔部署至 Service Fabric](service-fabric-deploy-existing-app.md)。

雖然本逐步解說示範的是如何部署使用 MongoDB 做為資料存放區並具有 Node.js 前端的應用程式，但是您可以將這些步驟套用到任何與另一個應用程式具有相依性的應用程式。   

您可以使用 Visual Studio 來產生包含多個來賓可執行檔的應用程式套件。 請參閱[使用 Visual Studio 封裝現有應用程式](service-fabric-deploy-existing-app.md)。 新增第一個來賓可執行檔之後，在應用程式專案上按一下滑鼠右鍵，然後選取 [新增] -> [新的 Service Fabric 服務] 將第二個來賓可執行檔專案新增至方案。 請注意：如果您選擇在 Visual Studio 專案中連結來源，則建置 Visual Studio 方案將可確保應用程式封裝是最新的，並且含有來源中的變更。 

## <a name="samples"></a>範例
* [封裝和部署來賓可執行檔的範例](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [兩個客體可執行檔 (C# 和 nodejs) 使用 REST 透過命名服務進行通訊的範例](https://github.com/Azure-Samples/service-fabric-containers)

## <a name="manually-package-the-multiple-guest-executable-application"></a>手動封裝多個來賓可執行檔應用程式
或者，您可以手動封裝來賓可執行檔。 對於手動封裝，本文使用 Service Fabric 封裝工具，您可在 [http://aka.ms/servicefabricpacktool](http://aka.ms/servicefabricpacktool) 取得。

### <a name="packaging-the-nodejs-application"></a>封裝 Node.js 應用程式
本文假設 Service Fabric 叢集中的節點上未安裝 Node.js。 因此，您需要在封裝之前，先將 Node.exe 新增至您節點應用程式的根目錄。 Node.js 應用程式 (使用 Express Web 架構和 Jade 範本引擎) 的目錄結構看起來應該與以下類似：

```
|-- NodeApplication
    |-- bin
        |-- www
    |-- node_modules
        |-- .bin
        |-- express
        |-- jade
        |-- etc.
    |-- public
        |-- images
        |-- etc.
    |-- routes
        |-- index.js
        |-- users.js
    |-- views
        |-- index.jade
        |-- etc.
    |-- app.js
    |-- package.json
    |-- node.exe
```

在下一個步驟中，您將為 Node.js 應用程式建立應用程式封裝。 以下程式碼會建立包含 Node.js 應用程式的 Service Fabric 應用程式封裝。

```
.\ServiceFabricAppPackageUtil.exe /source:'[yourdirectory]\MyNodeApplication' /target:'[yourtargetdirectory] /appname:NodeService /exe:'node.exe' /ma:'bin/www' /AppType:NodeAppType
```

以下是所使用之參數的描述：

* **/source** 指向應封裝之應用程式的目錄。
* **/target** 定義應在其中建立封裝的目錄。 這個目錄必須是與來源目錄不同的目錄。
* **/appname** 定義現有應用程式的應用程式名稱。 請務必了解這會轉譯成資訊清單中的服務名稱，而不是轉譯成 Service Fabric 應用程式名稱。
* **/exe** 定義 Service Fabric 應啟動的可執行檔，在此例中為 `node.exe`。
* **/ma** 定義要用來啟動可執行檔的引數。 由於未安裝 Node.js，因此 Service Fabric 需要執行 `node.exe bin/www`來啟動 Node.js Web 伺服器。  `/ma:'bin/www'` 會告訴封裝工具使用 `bin/www` 當做 node.exe 的引數。
* **/AppType** 定義 Service Fabric 應用程式類型名稱。

如果您瀏覽至 /target 參數中指定的目錄，您可以看到工具已建立可完整運作的 Service Fabric 封裝，如以下所示：

```
|--[yourtargetdirectory]
    |-- NodeApplication
        |-- C
              |-- bin
              |-- data
              |-- node_modules
              |-- public
              |-- routes
              |-- views
              |-- app.js
              |-- package.json
              |-- node.exe
        |-- config
              |--Settings.xml
        |-- ServiceManifest.xml
    |-- ApplicationManifest.xml
```
所產生的 ServiceManifest.xml 現在有個描述應該如何啟動 Node.js Web 伺服器的區段，如以下程式碼片段所示：

```xml
<CodePackage Name="C" Version="1.0">
    <EntryPoint>
        <ExeHost>
            <Program>node.exe</Program>
            <Arguments>'bin/www'</Arguments>
            <WorkingFolder>CodePackage</WorkingFolder>
        </ExeHost>
    </EntryPoint>
</CodePackage>
```
在此範例中，Node.js Web 伺服器會接聽通訊埠 3000，所以您需要更新 ServiceManifest.xml 檔案中的端點資訊，如以下所示。   

```xml
<Resources>
      <Endpoints>
         <Endpoint Name="NodeServiceEndpoint" Protocol="http" Port="3000" Type="Input" />
      </Endpoints>
</Resources>
```
### <a name="packaging-the-mongodb-application"></a>封裝 MongoDB 應用程式
既然您已封裝 Node.js 應用程式，您可以繼續封裝 MongoDB。 如先前所提到的，您現在進行的步驟並非 Node.js 和 MongoDB 專用的步驟。 事實上，它們適用於所有要封裝在一起以做為一個 Service Fabric 應用程式的應用程式。  

為了封裝 MongoDB，您會想要確定您封裝 Mongod.exe 和 Mongo.exe。 這兩個二進位檔都位於 MongoDB 安裝目錄的 `bin` 目錄中。 目錄結構類似於下方的結構。

```
|-- MongoDB
    |-- bin
        |-- mongod.exe
        |-- mongo.exe
        |-- anybinary.exe
```
Service Fabric 需要使用類似於下方的命令來啟動 MongoDB，因此封裝 MongoDB 時，您需要使用 `/ma` 參數。

```
mongod.exe --dbpath [path to data]
```
> [!NOTE]
> 如果您將 MongoDB 資料目錄放在節點的本機目錄中，當節點發生失敗時，將不會保留資料。 您應該使用永久性儲存體或實作 MongoDB 複本集以防止資料遺失。  
>
>

在 PowerShell 或命令殼層中，我們會使用下列參數來執行封裝工具：

```
.\ServiceFabricAppPackageUtil.exe /source: [yourdirectory]\MongoDB' /target:'[yourtargetdirectory]' /appname:MongoDB /exe:'bin\mongod.exe' /ma:'--dbpath [path to data]' /AppType:NodeAppType
```

為了將 MongoDB 新增至您的 Service Fabric 應用程式封裝，您必須確定 /target 參數指向已經包含應用程式資訊清單及 Node.js 應用程式的同一個目錄。 您也需要確定您是使用相同的 ApplicationType 名稱。

讓我們瀏覽至該目錄並檢查已建立的工具。

```
|--[yourtargetdirectory]
    |-- MyNodeApplication
    |-- MongoDB
        |-- C
            |--bin
                |-- mongod.exe
                |-- mongo.exe
                |-- etc.
        |-- config
            |--Settings.xml
        |-- ServiceManifest.xml
    |-- ApplicationManifest.xml
```
如您所見，工具已將新資料夾 [MongoDB] 新增至包含 MongoDB 二進位檔的目錄。 如果您開啟 `ApplicationManifest.xml` 檔案，您將可以看到封裝現在包含 Node.js 應用程式和 MongoDB。 以下程式碼會顯示應用程式資訊清單的內容。

```xml
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="MyNodeApp" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="MongoDB" ServiceManifestVersion="1.0" />
   </ServiceManifestImport>
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="NodeService" ServiceManifestVersion="1.0" />
   </ServiceManifestImport>
   <DefaultServices>
      <Service Name="MongoDBService">
         <StatelessService ServiceTypeName="MongoDB">
            <SingletonPartition />
         </StatelessService>
      </Service>
      <Service Name="NodeServiceService">
         <StatelessService ServiceTypeName="NodeService">
            <SingletonPartition />
         </StatelessService>
      </Service>
   </DefaultServices>
</ApplicationManifest>  
```

### <a name="publishing-the-application"></a>發佈應用程式
最後一個步驟是要使用以下 PowerShell 指令碼，將應用程式發佈至本機 Service Fabric 叢集：

```
Connect-ServiceFabricCluster localhost:19000

Write-Host 'Copying application package...'
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath '[yourtargetdirectory]' -ImageStoreConnectionString 'file:C:\SfDevCluster\Data\ImageStoreShare' -ApplicationPackagePathInImageStore 'NodeAppType'

Write-Host 'Registering application type...'
Register-ServiceFabricApplicationType -ApplicationPathInImageStore 'NodeAppType'

New-ServiceFabricApplication -ApplicationName 'fabric:/NodeApp' -ApplicationTypeName 'NodeAppType' -ApplicationTypeVersion 1.0  
```

將應用程式順利發佈至本機叢集之後，您便可以透過我們在 Node.js 應用程式的服務資訊清單中輸入的連接埠 (例如 http://localhost:3000) 存取 Node.js 應用程式。

在本教學課程中，您已看到如何輕鬆地將兩個現有應用程式封裝成一個 Service Fabric 應用程式。 您也會了解如何將其部署到 Service Fabric，以便讓它能夠從一些 Service Fabric 功能 (例如高可用性和健康情況系統整合) 獲益。


## <a name="adding-more-guest-executables-to-an-existing-application-using-yeoman-on-linux"></a>在 Linux 上使用 Yeoman 將更多來賓可執行檔新增至現有的應用程式

若要將其他服務新增至已使用 `yo` 建立的應用程式，請執行下列步驟︰ 
1. 將目錄變更為現有應用程式的根目錄。  例如，如果 `MyApplication` 是 Yeoman 所建立的應用程式，則為 `cd ~/YeomanSamples/MyApplication`。
2. 執行 `yo azuresfguest:AddService` 並提供必要的詳細資料。

## <a name="next-steps"></a>後續步驟
* 了解如何使用 [Service Fabric 部署容器和容器概觀](service-fabric-containers-overview.md)
* [封裝和部署來賓可執行檔的範例](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [兩個客體可執行檔 (C# 和 nodejs) 使用 REST 透過命名服務進行通訊的範例](https://github.com/Azure-Samples/service-fabric-containers)
