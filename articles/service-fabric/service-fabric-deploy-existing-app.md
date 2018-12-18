---
title: 將現有的可執行檔部署至 Azure Service Fabric | Microsoft Docs
description: 了解如何將現有應用程式封裝為來賓可執行檔，使其可以部署至 Service Fabric 叢集。
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: ''
ms.assetid: d799c1c6-75eb-4b8a-9f94-bf4f3dadf4c3
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: na
ms.date: 07/02/2017
ms.author: mfussell
ms.openlocfilehash: 99d34d59bb9d55ff074d454fe4544917c4e91110
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34205977"
---
# <a name="package-and-deploy-an-existing-executable-to-service-fabric"></a>將現有可執行檔封裝和部署至 Service Fabric
在封裝現有可執行檔作為[來賓可執行檔](service-fabric-guest-executables-introduction.md)時，您可以選擇使用 Visual Studio 專案範本，或是[手動建立應用程式套件](#manually)。 使用 Visual Studio 時，就可讓 [新增專案範本] 為您建立應用程式套件的結構和資訊清單檔案。

> [!TIP]
> 將現有 Windows 可執行檔封裝成服務的最簡單方式，就是使用 Visual Studio，而在 Linux 上則是使用 Yeoman
>

## <a name="use-visual-studio-to-package-and-deploy-an-existing-executable"></a>使用 Visual Studio 來封裝及部署現有的可執行檔
Visual Studio 會提供 Service Fabric 服務範本，協助您將來賓可執行檔部署至 Service Fabric 叢集。

1. 選擇 [檔案]  >  [新增專案]，然後建立 Service Fabric 應用程式。
2. 選擇 [來賓執行檔] 做為服務範本。
3. 按一下 [瀏覽] 以選取內含執行檔的資料夾，並填入其餘參數以建立服務。
   * 「Code Package Behavior」。 可設定為將資料夾的所有內容複製到 Visual Studio 專案，這在執行檔沒有變更時很有用。 如果您預期會變更可執行檔，並想要以動態方式取得新組建，則可以選擇改為連結到資料夾。 在 Visual Studio 中建立應用程式專案時，您可以使用連結的資料夾。 這會從專案內連結到來源位置，讓您可以在來源目的地更新來賓執行檔。 在組建時使這些更新會成為應用程式套件的一部分。
   * 「Program」指定應執行以便啟動服務的執行檔。
   * 「Arguments」指定應傳遞至執行檔的引數。 這可以是具有引數的參數清單。
   * 「WorkingFolder」指定即將啟動之程序的工作目錄。 您可以指定三個值：
     * `CodeBase` 指定工作目錄即將設為應用程式套件中的 code 目錄 (先前檔案結構中所示的 `Code` 目錄)。
     * `CodePackage` 指定工作目錄即將設為應用程式套件中的根目錄 (先前檔案結構中所示的 `GuestService1Pkg`)。
     * `Work` 指定檔案放在名為 work 的子目錄中。
4. 指定服務的名稱，然後按一下 [確定]。
5. 如果服務需要用來進行通訊的端點，您現在可以將 protocol、port 和 type 新增至 ServiceManifest.xml 檔案。 例如：`<Endpoint Name="NodeAppTypeEndpoint" Protocol="http" Port="3000" UriScheme="http" PathSuffix="myapp/" Type="Input" />`。
6. 您現在可以藉由在 Visual Studio 中偵錯方案，對本機叢集執行封裝和發佈動作。 準備好時，即可將應用程式發佈至遠端叢集，或將方案簽入到原始檔控制。
7. 請參閱[檢查您的執行中應用程式](#check-your-running-application)，以了解如何檢視 Service Fabric Explorer 中執行的來賓執行檔服務。

如需逐步解說範例，請參閱[使用 Visual Studio 建立第一個客體可執行檔應用程式](quickstart-guest-app.md)。

## <a name="use-yeoman-to-package-and-deploy-an-existing-executable-on-linux"></a>使用 Yoeman 在 Linux 上封裝及部署現有的可執行檔

在 Linux 上建立及部署來賓可執行檔的程序與部署 csharp 或 java 應用程式的程序相同。

1. 在終端機中，輸入 `yo azuresfguest`。
2. 為您的應用程式命名。
3. 為您的服務命名並提供詳細資料，包括可執行檔的路徑，以及叫用它時所必須使用的參數。

Yeoman 會建立應用程式套件，其中包含適當的應用程式和資訊清單檔案，以及安裝和解除安裝指令碼。

<a id="manually"></a>

## <a name="manually-package-and-deploy-an-existing-executable"></a>手動封裝和部署現有執行檔
手動封裝來賓執行檔的程序是基於下列步驟：

1. 建立套件目錄結構。
2. 新增應用程式的程式碼和組態檔。
3. 編輯服務資訊清單檔。
4. 編輯應用程式資訊清單檔。

<!--
>[AZURE.NOTE] We do provide a packaging tool that allows you to create the ApplicationPackage automatically. The tool is currently in preview. You can download it from [here](http://aka.ms/servicefabricpacktool).
-->

### <a name="create-the-package-directory-structure"></a>建立套件目錄結構
您可以如先前＜應用程式套件檔案結構＞一節所述開始建立目錄結構。

### <a name="add-the-applications-code-and-configuration-files"></a>新增應用程式的程式碼和組態檔
建立目錄結構之後，您可以在 code 和 config 目錄之下新增應用程式的程式碼和組態檔。 您也可以在 code 和 config 目錄之下建立其他目錄或子目錄。

Service Fabric 會進行應用程式根目錄內容的 `xcopy`，所以除了建立兩個最上層目錄 code 和 settings 以外，沒有預先定義的結構可使用  (您可以選擇不同的名稱。 下一節中有更多詳細資訊)。

> [!NOTE]
> 請確定您包含應用程式需要的所有檔案和相依項目。 Service Fabric 會將應用程式套件的內容，複製到即將部署應用程式服務的叢集中的所有節點上。 該套件應包含應用程式需要執行的所有程式碼。 請勿假設已經安裝相依項目。
>
>

### <a name="edit-the-service-manifest-file"></a>編輯服務資訊清單檔
下一步就是編輯服務資訊清單檔以包含下列資訊：

* 服務類型的名稱。 這是 Service Fabric 用來識別服務的識別碼。
* 用來啟動應用程式 (ExeHost) 的命令。
* 必須執行才能安裝應用程式 (SetupEntrypoint) 的任何指令碼。

以下是 `ServiceManifest.xml` 檔案的範例：

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" Name="NodeApp" Version="1.0.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <ServiceTypes>
      <StatelessServiceType ServiceTypeName="NodeApp" UseImplicitHost="true"/>
   </ServiceTypes>
   <CodePackage Name="code" Version="1.0.0.0">
      <SetupEntryPoint>
         <ExeHost>
             <Program>scripts\launchConfig.cmd</Program>
         </ExeHost>
      </SetupEntryPoint>
      <EntryPoint>
         <ExeHost>
            <Program>node.exe</Program>
            <Arguments>bin/www</Arguments>
            <WorkingFolder>CodePackage</WorkingFolder>
         </ExeHost>
      </EntryPoint>
   </CodePackage>
   <Resources>
      <Endpoints>
         <Endpoint Name="NodeAppTypeEndpoint" Protocol="http" Port="3000" Type="Input" />
      </Endpoints>
   </Resources>
</ServiceManifest>
```

接下來幾節說明您需要更新的不同檔案部分。

#### <a name="update-servicetypes"></a>更新 ServiceTypes
```xml
<ServiceTypes>
  <StatelessServiceType ServiceTypeName="NodeApp" UseImplicitHost="true" />
</ServiceTypes>
```

* 您可以挑選任何您要用於 `ServiceTypeName` 的名稱。 此值在 `ApplicationManifest.xml` 檔案中用來識別服務。
* 指定 `UseImplicitHost="true"`。 此屬性會告知 Service Fabric，此服務是以獨立式 (Self-Contained) 應用程式為基礎，所以 Service Fabric 只要將它當作程序啟動並監視其健康狀況即可。

#### <a name="update-codepackage"></a>更新 CodePackage
CodePackage 元素指定服務程式碼的位置 (和版本)。

```xml
<CodePackage Name="Code" Version="1.0.0.0">
```

`Name` 元素用來在包含服務程式碼的應用程式套件中指定目錄的名稱。 `CodePackage` 也有 `version` 屬性。 這可用來指定程式碼的版本，而在 Service Fabric 中利用應用程式生命週期管理基礎結構，也可能用來升級服務的程式碼。

#### <a name="optional-update-setupentrypoint"></a>選擇性︰更新 SetupEntrypoint
```xml
<SetupEntryPoint>
   <ExeHost>
       <Program>scripts\launchConfig.cmd</Program>
   </ExeHost>
</SetupEntryPoint>
```
SetupEntryPoint 元素用來指定任何應在服務的程式碼啟動前執行的可執行檔或批次檔。 這是選擇性步驟，因此如果不需要初始化，則不需納入此元素。 每次重新啟動服務時，就會執行 SetupEntrypoint。

只有一個 SetupEntryPoint，所以如果應用程式的 setup 需要多個指令碼，則必須將 setup 指令碼組合在單一批次檔中。 SetupEntryPoint 可以執行任何類型的檔案：執行檔、批次檔、PowerShell Cmdlet。 如需詳細資訊，請參閱[設定 SetupEntryPoint](service-fabric-application-runas-security.md)。

在上述範例中，SetupEntryPoint 會執行位於 code 目錄的 `scripts` 子目錄中的批次檔 `LaunchConfig.cmd` (假設 WorkingFolder 元素已設為 CodeBase)。

#### <a name="update-entrypoint"></a>更新 EntryPoint
```xml
<EntryPoint>
  <ExeHost>
    <Program>node.exe</Program>
    <Arguments>bin/www</Arguments>
    <WorkingFolder>CodeBase</WorkingFolder>
  </ExeHost>
</EntryPoint>
```

服務資訊清單檔中的 `EntryPoint` 元素用來指定如何啟動服務。 `ExeHost` 元素指定應用來啟動服務的可執行檔 (和引數)。

* `Program` 指定應啟動服務的執行檔名稱。
* `Arguments` 指定應傳遞至可執行檔的引數。 這可以是具有引數的參數清單。
* `WorkingFolder` 指定即將啟動之程序的工作目錄。 您可以指定三個值：
  * `CodeBase` 指定工作目錄即將設為應用程式套件中的 code 目錄 (先前檔案結構中的 `Code` 目錄)。
  * `CodePackage` 指定工作目錄即將設為應用程式套件中的根目錄 (先前檔案結構中的 `GuestService1Pkg`)。
    * `Work` 指定檔案放在名為 work 的子目錄中。

WorkingFolder 適合用來設定正確的工作目錄，以便應用程式或初始化指令碼可使用相對路徑。

#### <a name="update-endpoints-and-register-with-naming-service-for-communication"></a>更新端點，並向命名服務註冊以進行通訊
```xml
<Endpoints>
   <Endpoint Name="NodeAppTypeEndpoint" Protocol="http" Port="3000" Type="Input" />
</Endpoints>

```
在前述範例中，`Endpoint` 元素指定應用程式可以接聽的端點。 在此範例中，Node.js 應用程式會接聽 http 連接埠 3000。

此外，您可以要求 Service Fabric 將此端點發佈至命名服務，讓其他服務可以探索這項服務的端點位址。 這可讓您在服務 (來賓可執行檔) 之間通訊。
發佈的端點位址格式為 `UriScheme://IPAddressOrFQDN:Port/PathSuffix`。 `UriScheme` 和 `PathSuffix` 是選擇性的屬性。 `IPAddressOrFQDN` 是此執行檔所在節點的 IP 位址或完整網域名稱，系統會自動為您計算。

在下列範例中，部署服務後，您在 Service Fabric Explorer 中會看到服務執行個體已發佈的端點，類似於 `http://10.1.4.92:3000/myapp/`。 或者，如果這是本機電腦，則您會看到 `http://localhost:3000/myapp/`。

```xml
<Endpoints>
   <Endpoint Name="NodeAppTypeEndpoint" Protocol="http" Port="3000"  UriScheme="http" PathSuffix="myapp/" Type="Input" />
</Endpoints>
```
您可以使用這些位址搭配[反向 Proxy](service-fabric-reverseproxy.md)，就能在服務之間進行通訊。

### <a name="edit-the-application-manifest-file"></a>編輯應用程式資訊清單檔
設定 `Servicemanifest.xml` 檔案之後，您需要對 `ApplicationManifest.xml` 檔進行一些變更，以確保使用正確的服務類型和名稱。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="NodeAppType" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="NodeApp" ServiceManifestVersion="1.0.0.0" />
   </ServiceManifestImport>
</ApplicationManifest>
```

#### <a name="servicemanifestimport"></a>ServiceManifestImport
在 `ServiceManifestImport` 元素中，您可以指定要包含在應用程式中的一個或多項服務。 `ServiceManifestName` 指定 `ServiceManifest.xml` 檔案所在的目錄名稱，用來參考服務。

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="NodeApp" ServiceManifestVersion="1.0.0.0" />
</ServiceManifestImport>
```

## <a name="set-up-logging"></a>設定記錄
對於來賓可執行檔，最好能夠查看主控台記錄檔，以查明應用程式和設定指令碼是否顯示任何錯誤。
在 `ServiceManifest.xml` 檔案中，可使用 `ConsoleRedirection` 元素設定主控台重新導向。

> [!WARNING]
> 切勿在實際部署的應用程式中使用主控台重新導向原則，因為這可能會影響應用程式容錯移轉。 僅將此原則用於本機開發及偵錯。  
>
>

```xml
<EntryPoint>
  <ExeHost>
    <Program>node.exe</Program>
    <Arguments>bin/www</Arguments>
    <WorkingFolder>CodeBase</WorkingFolder>
    <ConsoleRedirection FileRetentionCount="5" FileMaxSizeInKb="2048"/>
  </ExeHost>
</EntryPoint>
```

`ConsoleRedirection` 可用來將主控台輸出 (stdout 和 stderr) 重新導向到工作目錄。 這提供了在 Service Fabric 叢集中設定或執行應用程式期間確認沒有發生任何錯誤的功能。

`FileRetentionCount` 決定工作目錄中儲存多少個檔案。 例如，值為 5 表示工作目錄中會儲存先前 5 次執行的記錄檔。

`FileMaxSizeInKb` 指定記錄檔的大小上限。

記錄檔會儲存在服務的其中一個工作目錄中。 若要判斷檔案的位置，使用 Service Fabric Explorer 來判斷服務執行所在的節點以及目前使用的工作目錄。 本文稍後會說明此程序。

## <a name="deployment"></a>部署
最後一個步驟是[部署應用程式](service-fabric-deploy-remove-applications.md)。 下列 PowerShell 指令碼示範如何將應用程式部署至本機開發叢集，並啟動新的 Service Fabric 服務。

```PowerShell

Connect-ServiceFabricCluster localhost:19000

Write-Host 'Copying application package...'
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath 'C:\Dev\MultipleApplications' -ImageStoreConnectionString 'file:C:\SfDevCluster\Data\ImageStoreShare' -ApplicationPackagePathInImageStore 'nodeapp'

Write-Host 'Registering application type...'
Register-ServiceFabricApplicationType -ApplicationPathInImageStore 'nodeapp'

New-ServiceFabricApplication -ApplicationName 'fabric:/nodeapp' -ApplicationTypeName 'NodeAppType' -ApplicationTypeVersion 1.0

New-ServiceFabricService -ApplicationName 'fabric:/nodeapp' -ServiceName 'fabric:/nodeapp/nodeappservice' -ServiceTypeName 'NodeApp' -Stateless -PartitionSchemeSingleton -InstanceCount 1

```

>[!TIP]
> 如果封裝很大或有許多檔案，請在將它複製到映像存放區之前[壓縮封裝](service-fabric-package-apps.md#compress-a-package)。 請在[這裡](service-fabric-deploy-remove-applications.md#upload-the-application-package)閱讀更多資訊。
>

Service Fabric 服務可以各種「組態」部署。 例如，它可部署為單一或多個執行個體，也可以 Service Fabric 叢集的每個節點上有一個服務執行個體的方式部署。

`New-ServiceFabricService` Cmdlet 的 `InstanceCount` 參數用來指定應在 Service Fabric 叢集中啟動多少個服務執行個體。 您可以根據要部署的應用程式類型來設定 `InstanceCount` 值。 兩個最常見的案例包括：

* `InstanceCount = "1"`。 在此案例中，叢集中只部署一個服務執行個體。 Service Fabric 的排程器會決定即將部署服務的節點。
* `InstanceCount ="-1"`。 在此案例中，Service Fabric 叢集中的每個節點上都部署一個服務執行個體。 結果，叢集中的每個節點都有一個 (且僅只一個) 服務執行個體。

這是前端應用程式 (例如 REST 端點) 很有用的組態，因為用戶端應用程式只需要「連線」到叢集中的任何節點，即可使用端點。 當 Service Fabric 叢集的所有節點都連線到負載平衡器時，也可使用此組態。 如此便可將用戶端流量分散於在叢集中所有節點上執行的服務。

## <a name="check-your-running-application"></a>來查執行中的應用程式
在 Service Fabric Explorer 中，找出執行服務的節點。 在此範例中，它是在 Node1 上執行：

![服務執行所在的節點](./media/service-fabric-deploy-existing-app/nodeappinsfx.png)

如果您巡覽至節點並瀏覽至應用程式，您會看到基本節點資訊，包括它在磁碟上的位置。

![在磁碟上的位置](./media/service-fabric-deploy-existing-app/locationondisk2.png)

如果您使用「伺服器總管」來瀏覽至目錄，您可以找到工作目錄和服務的記錄檔資料夾，如以下螢幕擷取畫面所示： 

![記錄檔的位置](./media/service-fabric-deploy-existing-app/loglocation.png)

## <a name="next-steps"></a>後續步驟
在本文中，您已經學會如何封裝來賓可執行檔並部署至 Service Fabric。 請參閱下列文章以了解相關資訊和工作。

* [封裝和部署來賓可執行檔的範例](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)，包括封裝工具預先發行版本的連結
* [兩個客體可執行檔 (C# 和 nodejs) 使用 REST 透過命名服務進行通訊的範例](https://github.com/Azure-Samples/service-fabric-containers)
* [部署多個來賓可執行檔](service-fabric-deploy-multiple-apps.md)
* [使用 Visual Studio 建立第一個 Service Fabric 應用程式](service-fabric-create-your-first-application-in-visual-studio.md)
