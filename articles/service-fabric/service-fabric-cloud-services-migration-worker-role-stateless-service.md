---
title: 將 Azure 雲端服務應用程式轉換成 Service Fabric | Microsoft Docs
description: 本指南會比較雲端服務的 Web 角色和背景工作角色以及 Service Fabric 的無狀態服務，以協助您從雲端服務移轉到 Service Fabric。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''
ms.assetid: 5880ebb3-8b54-4be8-af4b-95a1bc082603
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/02/2017
ms.author: vturecek
ms.openlocfilehash: 4eed3825d52fe52025077980e21f3763cc5751ac
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44049944"
---
# <a name="guide-to-converting-web-and-worker-roles-to-service-fabric-stateless-services"></a>將 Web 角色和背景工作角色轉換成 Service Fabric 無狀態服務的指南
本文說明如何將雲端服務的 Web 角色和背景工作角色移轉至 Service Fabric 無狀態服務。 對於整體架構會大致保持相同的應用程式來說，這是最簡單的雲端服務至 Service Fabric 移轉路徑。

## <a name="cloud-service-project-to-service-fabric-application-project"></a>雲端服務專案到 Service Fabric 應用程式專案
 雲端服務專案和 Service Fabric 應用程式專案結構類似，且兩者皆能代表應用程式的部署單位，也就是說，兩者各自會定義所部署來執行應用程式的完整封裝。 雲端服務專案包含一個或多個 Web 角色和背景工作角色。 同樣地，Service Fabric 應用程式專案也包含一個或多個服務。 

兩者的差別在於，雲端服務專案會結合應用程式部署與 VM 部署，因此其中會包含 VM 組態設定，而 Service Fabric 應用程式專案則只會定義將要部署到 Service Fabric 叢集中一組現有 VM 的應用程式。 部署 Service Fabric 叢集時，可透過 Resource Manager 範本或 Azure 入口網站來進行部署，叢集本身只會部署一次，但可在此叢集中部署多個 Service Fabric 應用程式。

![Service Fabric 和雲端服務專案的比較][3]

## <a name="worker-role-to-stateless-service"></a>背景工作角色至無狀態服務
從概念上來說，背景工作角色代表無狀態的工作負載，這表示是工作負載的每個執行個體都是相同的，隨時都可將要求路由傳送到任何執行個體。 每個執行個體應該不會記得先前的要求。 工作負載的運作狀態是由外部狀態存放區 (例如 Azure 資料表儲存體或 Azure Document DB) 負責管理。 在 Service Fabric 中，這類工作負載是以無狀態服務來代表。 只要將背景工作角色程式碼轉換成無狀態服務，就能以最簡單的方式將背景工作角色移轉到 Service Fabric。

![背景工作角色至無狀態服務][4]

## <a name="web-role-to-stateless-service"></a>Web 角色至無狀態服務
和背景工作角色類似，Web 角色也代表無狀態的工作負載，因此在概念上也能對應至 Service Fabric 無狀態服務。 不過，和 Web 角色不同的是，Service Fabric 不支援 IIS。 若要將 Web 應用程式從 Web 角色移轉至無狀態服務，必須先移動到可以自我裝載且不仰賴 IIS 或 System.Web (例如 ASP.NET Core 1) 的 Web 架構。

| **應用程式** | **支援** | **移轉路徑** |
| --- | --- | --- |
| ASP.NET Web Forms |否 |轉換為 ASP.NET Core 1 MVC |
| ASP.NET MVC |移轉 |升級至 ASP.NET Core 1 MVC |
| ASP.NET Web API |移轉 |使用自我裝載的伺服器或 ASP.NET Core 1 |
| ASP.NET Core 1 |是 |N/A |

## <a name="entry-point-api-and-lifecycle"></a>進入點 API 和生命週期
背景工作角色和 Service Fabric 服務 API 提供類似的進入點： 

| **進入點** | **背景工作角色** | **Service Fabric 服務** |
| --- | --- | --- |
| Processing |`Run()` |`RunAsync()` |
| VM 啟動 |`OnStart()` |N/A |
| VM 停止 |`OnStop()` |N/A |
| 開啟接聽程式以接收用戶端要求 |N/A |<ul><li> `CreateServiceInstanceListener()` (針對無狀態)</li><li>`CreateServiceReplicaListener()` (針對具狀態)</li></ul> |

### <a name="worker-role"></a>背景工作角色
```csharp

using Microsoft.WindowsAzure.ServiceRuntime;

namespace WorkerRole1
{
    public class WorkerRole : RoleEntryPoint
    {
        public override void Run()
        {
        }

        public override bool OnStart()
        {
        }

        public override void OnStop()
        {
        }
    }
}

```

### <a name="service-fabric-stateless-service"></a>Service Fabric 無狀態服務
```csharp

using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.ServiceFabric.Services.Communication.Runtime;
using Microsoft.ServiceFabric.Services.Runtime;

namespace Stateless1
{
    public class Stateless1 : StatelessService
    {
        protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
        {
        }

        protected override Task RunAsync(CancellationToken cancelServiceInstance)
        {
        }
    }
}

```

兩者都有可從中開始處理的主要「執行」覆寫。 Service Fabric 服務將 `Run`、`Start` 和 `Stop` 結合為單一進入點 `RunAsync`。 當 `RunAsync` 啟動時，服務應該會開始運作，而當 `RunAsync` 方法的 CancellationToken 收到信號時，則應該會停止運作。 

背景工作角色和 Service Fabric 服務的生命週期與存留期之間有幾個主要差異：

* **生命週期：** 最大的差異是背景工作角色是 VM，因此其生命週期繫結至 VM，且包含 VM 啟動和停止時的事件。 Service Fabric 服務的生命週期則和 VM 的生命週期不同，因此不包含主機 VM 或機器啟動和停止時的事件，因為它們彼此不相關。
* **存留期：** 背景工作角色執行個體會在 `Run` 方法結束時回收。 不過，Service Fabric 服務中的 `RunAsync` 方法可以執行到完成為止，且服務執行個體會維持啟動狀態。 

Service Fabric 為接聽用戶端要求的服務提供選擇性的通訊設定進入點。 RunAsync 和通訊進入點都是 Service Fabric 服務中的選擇性覆寫 (服務可選擇只接聽用戶端要求或只執行處理迴圈，或兩者都選擇)，這就是 RunAsync 方法不必重新啟動服務執行個體就可以結束的原因，因為它可以繼續接聽用戶端要求。

## <a name="application-api-and-environment"></a>應用程式 API 和環境
雲端服務環境 API 提供目前 VM 執行個體的資訊和功能，以及關於其他 VM 角色執行個體的資訊。 Service Fabric 則提供其執行階段的相關資訊，以及關於服務目前執行所在之節點的某些資訊。 

| **環境工作** | **雲端服務** | **Service Fabric** |
| --- | --- | --- |
| 組態設定和變更通知 |`RoleEnvironment` |`CodePackageActivationContext` |
| 本機儲存體 |`RoleEnvironment` |`CodePackageActivationContext` |
| 端點資訊 |`RoleInstance` <ul><li>目前的執行個體︰`RoleEnvironment.CurrentRoleInstance`</li><li>其他角色和執行個體︰`RoleEnvironment.Roles`</li> |<ul><li>`NodeContext` (針對目前的節點位址)</li><li>`FabricClient` 和 `ServicePartitionResolver` (針對服務端點探索)</li> |
| 環境模擬 |`RoleEnvironment.IsEmulated` |N/A |
| 同時變更事件 |`RoleEnvironment` |N/A |

## <a name="configuration-settings"></a>組態設定
雲端服務中的組態設定已針對 VM 角色設定，並套用至該 VM 角色的所有執行個體。 這些設定是 ServiceConfiguration.*.cscfg 檔案中所設定的索引鍵/值組，並可直接透過 RoleEnvironment 進行存取。 在 Service Fabric 中，設定會個別套用至每個服務和每個應用程式，而不是套用至 VM，因為 VM 可以裝載多個服務和應用程式。 服務包含三個封裝：

* **程式碼：** 包含服務的可執行檔、二進位檔、DLL 和服務需要執行的任何其他檔案。
* **組態：** 服務的所有組態檔和設定。
* **資料：** 與服務相關聯的靜態資料檔案。

這些封裝每一個都可以獨立設定版本和升級。 和雲端服務類似，組態封裝可以透過 API 以程式設計方式存取，並有提供事件來通知服務其組態封裝已變更。 Settings.xml 檔案可用於索引鍵/值組的組態和程式設計存取，類似於 App.config 檔案的應用程式設定區段。 不過，和雲端服務不同的是，Service Fabric 組態封裝可以包含任何格式的任何組態檔，無論是 XML、JSON、YAML 或自訂的二進位格式。 

### <a name="accessing-configuration"></a>存取組態
#### <a name="cloud-services"></a>雲端服務
透過 `RoleEnvironment`即可存取 ServiceConfiguration.*.cscfg 中的組態設定。 這些設定可全域提供給相同雲端服務部署中的所有角色執行個體使用。

```csharp

string value = RoleEnvironment.GetConfigurationSettingValue("Key");

```

#### <a name="service-fabric"></a>Service Fabric
每個服務都有自己的個別組態封裝。 可供叢集中所有應用程式存取的全域組態設定沒有內建機制。 使用組態封裝內的 Service Fabric 特殊組態檔 Settings.xml 時，Settings.xml 中的值可以在應用程式層級覆寫，實現應用程式層級的組態設定。

透過服務的 `CodePackageActivationContext`即可在每個服務執行個體內存取組態設定。

```csharp

ConfigurationPackage configPackage = this.Context.CodePackageActivationContext.GetConfigurationPackageObject("Config");

// Access Settings.xml
KeyedCollection<string, ConfigurationProperty> parameters = configPackage.Settings.Sections["MyConfigSection"].Parameters;

string value = parameters["Key"]?.Value;

// Access custom configuration file:
using (StreamReader reader = new StreamReader(Path.Combine(configPackage.Path, "CustomConfig.json")))
{
    MySettings settings = JsonConvert.DeserializeObject<MySettings>(reader.ReadToEnd());
}

```

### <a name="configuration-update-events"></a>組態更新事件
#### <a name="cloud-services"></a>雲端服務
當環境發生變更 (例如組態變更) 時，就會使用 `RoleEnvironment.Changed` 事件來通知所有角色執行個體。 使用此事件可取用組態更新，卻又不會回收角色執行個體或重新啟動背景工作角色處理序。

```csharp

RoleEnvironment.Changed += RoleEnvironmentChanged;

private void RoleEnvironmentChanged(object sender, RoleEnvironmentChangedEventArgs e)
{
   // Get the list of configuration changes
   var settingChanges = e.Changes.OfType<RoleEnvironmentConfigurationSettingChange>();
foreach (var settingChange in settingChanges) 
   {
      Trace.WriteLine("Setting: " + settingChange.ConfigurationSettingName, "Information");
   }
}

```

#### <a name="service-fabric"></a>Service Fabric
服務中的三個封裝類型 (程式碼、組態和資料) 每個都有可在封裝已更新、新增或移除時通知服務執行個體的事件。 服務可以包含多個各類型的封裝。 例如，服務可以有多個組態封裝，每個都個別設定版本並可升級。 

這些事件可供取用服務封裝中的變更而不必重新啟動服務執行個體。

```csharp

this.Context.CodePackageActivationContext.ConfigurationPackageModifiedEvent +=
                    this.CodePackageActivationContext_ConfigurationPackageModifiedEvent;

private void CodePackageActivationContext_ConfigurationPackageModifiedEvent(object sender, PackageModifiedEventArgs<ConfigurationPackage> e)
{
    this.UpdateCustomConfig(e.NewPackage.Path);
    this.UpdateSettings(e.NewPackage.Settings);
}

```

## <a name="startup-tasks"></a>啟動工作
啟動工作是應用程式啟動前所採取的動作。 啟動工作通常用來以提高的權限執行安裝指令碼。 雲端服務和 Service Fabric 皆支援啟動工作。 兩者的主要差異是，雲端服務中的啟動工作繫結至 VM，因為它是角色執行個體的一部分，而 Service Fabric 中的啟動工作則繫結至服務，而不會繫結至任何特定 VM。

| Service Fabric | 雲端服務 |
| --- | --- | --- |
| 組態位置 |ServiceDefinition.csdef |
| 權限 |「有限」或「提高」 |
| 排序 |「簡單」、「背景」、「前景」 |

### <a name="cloud-services"></a>雲端服務
雲端服務中的啟動進入點是在 ServiceDefinition.csdef 中針對每個角色進行設定。 

```xml

<ServiceDefinition>
    <Startup>
        <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple" >
            <Environment>
                <Variable name="MyVersionNumber" value="1.0.0.0" />
            </Environment>
        </Task>
    </Startup>
    ...
</ServiceDefinition>

```

### <a name="service-fabric"></a>Service Fabric
Service Fabric 中的啟動進入點是在 ServiceManifest.xml 中針對每個服務進行設定。

```xml

<ServiceManifest>
  <CodePackage Name="Code" Version="1.0.0">
    <SetupEntryPoint>
      <ExeHost>
        <Program>Startup.bat</Program>
      </ExeHost>
    </SetupEntryPoint>
    ...
</ServiceManifest>

``` 

## <a name="a-note-about-development-environment"></a>開發環境的注意事項
雲端服務和 Service Fabric 都會使用專案範本來與 Visual Studio 整合，並支援偵錯、設定以及部署到本機和 Azure。 雲端服務和 Service Fabric 也都提供本機開發執行階段環境。 兩者的差異在於，雲端服務的開發執行階段會模擬其執行所在的 Azure 環境，但 Service Fabric 不會使用模擬器，而是使用完整的 Service Fabric 執行階段。 您在本機開發機器執行的 Service Fabric 環境和在生產階段時執行的是相同的環境。

## <a name="next-steps"></a>後續步驟
請深入了解 Service Fabric Reliable Services 以及雲端服務與 Service Fabric 應用程式架構之間的基本差異，以了解如何利用 Service Fabric 的整組功能。

* [開始使用 Service Fabric Reliable Services](service-fabric-reliable-services-quick-start.md)
* [雲端服務和 Service Fabric 之間差異的概念指南](service-fabric-cloud-services-migration-differences.md)

<!--Image references-->
[3]: ./media/service-fabric-cloud-services-migration-worker-role-stateless-service/service-fabric-cloud-service-projects.png
[4]: ./media/service-fabric-cloud-services-migration-worker-role-stateless-service/worker-role-to-stateless-service.png
