---
title: 使用 ASP.NET Core 的服務通訊 |Microsoft Docs
description: 了解如何在無狀態和具狀態的 Reliable Services 中使用 ASP.NET Core。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''
ms.assetid: 8aa4668d-cbb6-4225-bd2d-ab5925a868f2
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 08/29/2018
ms.author: vturecek
ms.openlocfilehash: 384d0fa32b64706c9d9d9baa0e2e0bbb2ac3c522
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44719591"
---
# <a name="aspnet-core-in-service-fabric-reliable-services"></a>Service Fabric Reliable Services 中的 ASP.NET Core

ASP.NET Core 是新的開放原始碼和跨平台架構，可建置現代化雲端網際網路連線的應用程式，例如 web 應用程式、IoT 應用程式，以及行動後端。 

本文是深度指南，說明使用**Microsoft.ServiceFabric.AspNetCore.*** NuGet 套件集在Service Fabric Reliable Services 中裝載 ASP.NET Core 服務。

如需 Service Fabric 中 ASP.NET Core 的簡介教學課程，以及如何安裝開發環境設定的指示，請參閱[建立 .NET 應用程式](service-fabric-tutorial-create-dotnet-app.md)。

本文的其餘部分假設您十分熟悉 ASP.NET Core。 如果不是，我們建議您閱讀 [ASP.NET Core 基礎](https://docs.microsoft.com/aspnet/core/fundamentals/index)。

## <a name="aspnet-core-in-the-service-fabric-environment"></a>Service Fabric 環境中的 ASP.NET Core

ASP.NET Core 和 Service Fabric 應用程式可以在 .NET Core 以及完整的 .NET Framework 上執行。 可在 Service Fabric 中以兩種方式使用 ASP.NET Core︰
 - **裝載作為客體可執行檔**。 這主要用於在 Service Fabric 上執行現有的 ASP.NET Core 應用程式而無須變更程式碼。
 - **在 Reliable Service 內部執行**. 這可與 Service Fabric 執行階段更緊密整合，並可允許具狀態 ASP.NET Core 服務。

本文的其餘部分將說明如何使用隨附於 Service Fabric SDK 的 ASP.NET Core 整合元件，在 Reliable Service 內部使用 ASP.NET Core。 

## <a name="service-fabric-service-hosting"></a>Service Fabric 服務裝載

在 Service Fabric 中，您服務的一或多個執行個體及/或複本會在服務主機處理序中執行，這是執行您的服務程式碼的可執行檔。 您身為服務作者，擁有服務主機處理序，且 Service Fabric 會為您啟動及監視。

傳統 ASP.NET (直到 MVC 5) 已透過 System.Web.dll 與 IIS 緊密結合。 ASP.NET Core 能區別網頁伺服器與 web 應用程式。 這可讓 web 應用程式在不同的 web 伺服器之間具有可攜性，也可讓 web 伺服器自我裝載，這表示您的 web 伺服器可以在自己的程序中啟動，而不是在專用的 web 伺服器軟體所擁有的處理序 (例如 IIS) 中啟動。 

為了將 Service Fabric 服務和 ASP.NET 結合為客體可執行檔或結合至 Reliable Service 之中，您必須能夠在服務主機處理序中啟動 ASP.NET。 ASP.NET Core 自我裝載可讓您執行這項操作。

## <a name="hosting-aspnet-core-in-a-reliable-service"></a>在 Reliable Service 中裝載 ASP.NET Core
一般而言，自我裝載的 ASP.NET Core 應用程式會在應用程式的進入點建立 WebHost，例如 `Program.cs` 方法中的 `static void Main()`。 在此情況下，WebHost 的生命週期會繫結至處理序的生命週期。

![在處理序中裝載 ASP.NET Core][0]

不過，應用程式進入點不是在 Reliable Service 中建立 WebHost 的正確位置，因為應用程式進入點僅用來使用 Service Fabric 執行階段登錄服務類型，因此它可能會建立該服務類型的執行個體。 WebHost 應該建立在 Reliable Service 之內。 在服務主機處理序中，服務執行個體及/或複本可以通過多個生命週期。 

Reliable Service 執行個體是由您衍生自 `StatelessService` 或 `StatefulService` 的服務類別代表。 服務的通訊堆疊包含在您服務類別中的 `ICommunicationListener` 實作。 `Microsoft.ServiceFabric.AspNetCore.*` NuGet 封裝包含 `ICommunicationListener` 的實作，可在 Reliable Service 中啟動和管理 Kestrel 或 HttpSys 的 ASP.NET Core WebHost。

![在 Reliable Service 中裝載 ASP.NET Core][1]

## <a name="aspnet-core-icommunicationlisteners"></a>ASP.NET Core ICommunicationListeners
`Microsoft.ServiceFabric.AspNetCore.*` NuGet 套件中 Kestrel 和 HttpSys 的 `ICommunicationListener` 實作有類似的使用模式，但針對每個 web 伺服器執行的動作稍有不同。 

這兩種通訊接聽程式都會提供使用下列引數的建構函式︰
 - **`ServiceContext serviceContext`** 包含關於執行服務資訊的 `ServiceContext` 物件。
 - **`string endpointName`**：在 ServiceManifest.xml 中的 `Endpoint` 組態名稱。 這是兩個通訊接聽程式主要的不同之處︰HttpSys **需要** `Endpoint` 組態，而 Kestrel 不需要。
 - **`Func<string, AspNetCoreCommunicationListener, IWebHost> build`**：您所實作並在其中建立並傳回 `IWebHost` 的 lambda。 這可讓您以您通常會在 ASP.NET Core 應用程式的方式設定 `IWebHost`。 Lambda 提供為您根據 Service Fabric 整合選項所產生的 URL 和您提供的 `Endpoint` 組態。 然後該 URL 可以修改或依現狀使用以啟動 web 伺服器。

## <a name="service-fabric-integration-middleware"></a>Service Fabric 整合中介軟體
`Microsoft.ServiceFabric.AspNetCore` NuGet 套件包含 `IWebHostBuilder` 上的 `UseServiceFabricIntegration` 擴充方法，其會新增 Service Fabric 的中介軟體。 此中介軟體會設定 Kestrel 或 HttpSys `ICommunicationListener`，以使用 Service Fabric 命名服務註冊唯一服務 URL，然後驗證用戶端要求以確保用戶端連線至正確的服務。 這在共用主機環境 (例如 Service Fabric) 中是必要的，其中多個 web 應用程式可以在相同的實體或虛擬機器上執行，但請勿使用唯一的主機名稱，以避免用戶端不小心連線到錯誤的服務。 此案例會在下一節中詳細說明。

### <a name="a-case-of-mistaken-identity"></a>誤用身分識別的案例
不論通訊協定，服務複本會接聽唯一 IP:port 組合。 一旦服務複本開始在 IP:port 端點上接聽，它會向 Service Fabric 命名服務報告該端點位址，用戶端或其他服務可以在其中探索。 如果服務使用動態指派的應用程式連接埠，服務複本可能會針對先前碰巧位於相同實體或虛擬機器上的其他服務，而誤用其 IP:port 端點。 這可能會造成用戶端不小心連線至錯誤的服務。 如果以下一連串事件發生，則可能發生這個情況︰

 1. 服務 A 透過 HTTP 接聽 10.0.0.1:30000。 
 2. 用戶端解析服務 A 並取得位址 10.0.0.1:30000
 3. 服務 A 移到另一個節點。
 4. 服務 B 置於 10.0.0.1 且碰巧使用相同的連接埠 30000。
 5. 用戶端使用快取的位址 10.0.0.1:30000 嘗試連線到服務 A。
 6. 用戶端現在已成功連接至 B，不瞭解其連線到錯誤的服務。

這會不定時造成 難以診斷的 Bug。 

### <a name="using-unique-service-urls"></a>使用唯一的服務 URL
若要避免這個問題，服務可以使用唯一識別碼將端點張貼至命名服務，並在用戶端要求期間驗證該唯一識別碼。 這在非惡意租用戶受信任環境中的服務之間為合作式動作。 這在惡意租用戶環境中不會提供安全服務驗證。

在受信任環境中，由 `UseServiceFabricIntegration` 法所新增的中介軟體會自動將唯一識別項附加到已張貼至命名服務的地址，並在每個要求上驗證該識別項。 如果識別項不符合，中介軟體會立即傳回 HTTP 410 不存在的回應。

使用動態指派連接埠的服務應該要使用此中介軟體。

使用固定的唯一連接埠之服務在合作的環境中沒有這個問題。 固定的唯一連接埠通常是用於對外開放的服務，需要用戶端應用程式連線的知名通訊埠。 例如，大部分對網際網路開放的 web 應用程式會使用 web 瀏覽器連接的連接埠 80 或 443。 在此情況下，不應該啟用唯一識別碼。

下圖顯示已啟用中介軟體的要求流程︰

![Service Fabric ASP.NET Core 整合][2]

Kestrel 和 HttpSys `ICommunicationListener` 實作以完全相同的方式使用這項機制。 雖然 HttpSys 可以使用基礎 http.sys 連接埠共用功能，根據唯一的 URL 路徑內部區分要求，該功能不供 HttpSys`ICommunicationListener` 實作使用，因為會導致在稍早所述的案例中的 HTTP 503 和 HTTP 404 錯誤狀態碼。 這會使用戶端難以判斷錯誤的意圖，因為 HTTP 503 和 HTTP 404 已經常用來表示其他錯誤。 因此，Kestrel 和 HttpSys `ICommunicationListener` 實作會在 `UseServiceFabricIntegration` 擴充方法所提供的中介軟體上標準化，讓用戶端只需在 HTTP 410 回應上執行服務端點重新解析動作。

## <a name="httpsys-in-reliable-services"></a>Reliable Services 中的 HttpSys
HttpSys 可在 Reliable Service 中使用，方法為匯入 **Microsoft.ServiceFabric.AspNetCore.HttpSys** NuGet 封裝。 此套件包含 `HttpSysCommunicationListener`，實作`ICommunicationListener`，可讓您使用 HttpSys 作為 web 伺服器，在 Reliable Service 內部建立 ASP.NET Core WebHost。

HttpSys 會建置在 [Windows HTTP Server API](https://msdn.microsoft.com/library/windows/desktop/aa364510(v=vs.85).aspx)上。 這會使用 IIS 用來處理 HTTP 要求並將它們路由傳送到執行 web 應用程式之處理程序的 http.sys 的核心驅動程式。 這可讓相同實體或虛擬機器上的多個處理序在相同的連接埠上裝載 web 應用程式，由唯一的 URL 路徑或主機名稱區分。 這些功能對於在 Service Fabric 中於相同叢集裝載多個網站很有用。

>[!NOTE]
>HttpSys 實作只適用於 Windows 平台。

下圖說明 HttpSys 如何在 Windows 上使用 http.sys 核心驅動程式進行連接埠共用︰

![http.sys][3]

### <a name="httpsys-in-a-stateless-service"></a>無狀態服務中的 HttpSys
若要在無狀態服務中使用 `HttpSys`，請覆寫 `CreateServiceInstanceListeners` 方法並傳回 `HttpSysCommunicationListener` 執行個體︰

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    return new ServiceInstanceListener[]
    {
        new ServiceInstanceListener(serviceContext =>
            new HttpSysCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
                new WebHostBuilder()
                    .UseHttpSys()
                    .ConfigureServices(
                        services => services
                            .AddSingleton<StatelessServiceContext>(serviceContext))
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
                    .UseStartup<Startup>()
                    .UseUrls(url)
                    .Build()))
    };
}
```

### <a name="httpsys-in-a-stateful-service"></a>具狀態服務中的 HttpSys

由於基礎 http.sys 連接埠共用功能很複雜，`HttpSysCommunicationListener` 目前不是設計用於具狀態服務。 如需詳細資訊，請參閱下一節的使用 HttpSys 動態連接埠配置。 針對具狀態服務，Kestrel 是建議的 web 伺服器。

### <a name="endpoint-configuration"></a>端點組態

使用 Windows HTTP Server API，包括 HttpSys 的 Web 伺服器需要 `Endpoint` 組態。 使用 Windows HTTP 伺服器 API 的 web 伺服器都必須先保留其 URL 與http.sys (這通常使用 [netsh](https://msdn.microsoft.com/library/windows/desktop/cc307236(v=vs.85).aspx) 工具來完成)。 這個動作需要您服務預設未具有的提高權限。 *ServiceManifest.xml* 中的 `Endpoint` 設定之 `Protocol` 屬性的 "Http" 或 "https" 選項會特別用來指示 Service Fabric 執行階段，以使用[強式萬用字元](https://msdn.microsoft.com/library/windows/desktop/aa364698(v=vs.85).aspx) URL 前置詞代表您註冊 URL 與 http.sys。

例如，若要保留服務的 `http://+:80`，應該在 ServiceManifest.xml 中使用下列組態︰

```xml
<ServiceManifest ... >
    ...
    <Resources>
        <Endpoints>
            <Endpoint Name="ServiceEndpoint" Protocol="http" Port="80" />
        </Endpoints>
    </Resources>

</ServiceManifest>
```

端點名稱必須傳遞給 `HttpSysCommunicationListener` 建構函式︰

```csharp
 new HttpSysCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
 {
     return new WebHostBuilder()
         .UseHttpSys()
         .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
         .UseUrls(url)
         .Build();
 })
```

#### <a name="use-httpsys-with-a-static-port"></a>搭配使用 HttpSys 與靜態連接埠
搭配使用靜態連接埠與 HttpSys，在 `Endpoint` 組態中提供通訊埠編號︰

```xml
  <Resources>
    <Endpoints>
      <Endpoint Protocol="http" Name="ServiceEndpoint" Port="80" />
    </Endpoints>
  </Resources>
```

#### <a name="use-httpsys-with-a-dynamic-port"></a>搭配使用 HttpSys 與動態連接埠
若要使用動態指派的連接埠與 HttpSys，省略 `Endpoint` 組態中的 `Port` 屬性︰

```xml
  <Resources>
    <Endpoints>
      <Endpoint Protocol="http" Name="ServiceEndpoint" />
    </Endpoints>
  </Resources>
```

依 `Endpoint` 組態配置的動態連接埠只提供每個主機處理序一個連接埠。 目前的 Service Fabric 裝載模型可讓多個服務執行個體及/或複本裝載在相同的程序中，這表示當透過 `Endpoint` 組態配置時，每個都會共用相同的連接埠。 多個 HttpSys 執行個體可以使用基礎 http.sys 連接埠共用功能共用連接埠，但由於其為用戶端要求所帶來的複雜性，`HttpSysCommunicationListener` 並不支援。 針對動態連接埠使用量，Kestrel 是建議的 web 伺服器。

## <a name="kestrel-in-reliable-services"></a>Reliable Services 中的 Kestrel
Kestrel 可在 Reliable Service 中使用，方法為匯入 **Microsoft.ServiceFabric.AspNetCore.Kestrel** NuGet 封裝。 此套件包含 `KestrelCommunicationListener`，實作`ICommunicationListener`，可讓您使用 Kestrel 做為 web 伺服器，在 Reliable Service 內部建立 ASP.NET Core WebHost。

Kestrel 根據 libuv 是 ASP.NET Core 的跨平台 web 伺服器，為跨平台非同步 I/O 程式庫。 不同於 HttpSys，Kestrel 不會使用集中式端點管理員，例如 http.sys。 且不同於 HttpSys，Kestrel 不支援多個處理序之間的連接埠共用。 Kestrel 的每個執行個體必須使用唯一的連接埠。

![kestrel][4]

### <a name="kestrel-in-a-stateless-service"></a>無狀態服務中的 Kestrel
若要在無狀態服務中使用 `Kestrel`，請覆寫 `CreateServiceInstanceListeners` 方法並傳回 `KestrelCommunicationListener` 執行個體︰

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    return new ServiceInstanceListener[]
    {
        new ServiceInstanceListener(serviceContext =>
            new KestrelCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
                new WebHostBuilder()
                    .UseKestrel()
                    .ConfigureServices(
                        services => services
                            .AddSingleton<StatelessServiceContext>(serviceContext))
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
                    .UseStartup<Startup>()
                    .UseUrls(url)
                    .Build();
            ))
    };
}
```

### <a name="kestrel-in-a-stateful-service"></a>具狀態服務中的 Kestrel
若要在具狀態服務中使用 `Kestrel`，請覆寫 `CreateServiceReplicaListeners` 方法並傳回 `KestrelCommunicationListener` 執行個體︰

```csharp
protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new ServiceReplicaListener[]
    {
        new ServiceReplicaListener(serviceContext =>
            new KestrelCommunicationListener(serviceContext, (url, listener) =>
                new WebHostBuilder()
                    .UseKestrel()
                    .ConfigureServices(
                         services => services
                             .AddSingleton<StatefulServiceContext>(serviceContext)
                             .AddSingleton<IReliableStateManager>(this.StateManager))
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
                    .UseStartup<Startup>()
                    .UseUrls(url)
                    .Build();
            ))
    };
}
```

在此範例中，`IReliableStateManager` 的單一執行個體會提供給 WebHost 相依性插入容器。 這並非絕對必要，但它可讓您在 MVC 控制器動作方法中使用 `IReliableStateManager` 和可靠的集合。

`Endpoint` 組態名稱**不**提供給具狀態服務中的 `KestrelCommunicationListener`。 我們會在下列各節中詳細說明這個部分。

### <a name="endpoint-configuration"></a>端點組態
`Endpoint` 組態不需要使用 Kestrel。 

Kestrel 是簡單的獨立 web 伺服器；不同於 HttpSys (或 HttpListener)，它不需要 *ServiceManifest.xml* 中的 `Endpoint` 組態，因為它不需要在啟動之前註冊 URL。 

#### <a name="use-kestrel-with-a-static-port"></a>搭配使用 Kestrel 與靜態連接埠
靜態連接埠可以在 ServiceManifest.xml 的 `Endpoint` 組態中設定以供與 Kestrel 搭配使用。 雖然這並非絕對必要，它會提供兩個潛在的好處︰
 1. 如果連接埠不在應用程式連接埠範圍中，它會由 Service Fabric 透過 OS 防火牆開啟。
 2. 透過 `KestrelCommunicationListener` 提供給您的 URL 會使用此連接埠。

```xml
  <Resources>
    <Endpoints>
      <Endpoint Protocol="http" Name="ServiceEndpoint" Port="80" />
    </Endpoints>
  </Resources>
```

如果設定了 `Endpoint`，其名稱必須傳遞至 `KestrelCommunicationListener` 建構函式︰ 

```csharp
new KestrelCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) => ...
```

如果未使用 `Endpoint` 組態，請省略 `KestrelCommunicationListener` 建構函式中的名稱。 在此情況下，會使用動態連接埠。 如需詳細資訊，請參閱下一節。

#### <a name="use-kestrel-with-a-dynamic-port"></a>搭配使用 Kestrel 與動態連接埠
Kestrel 無法在 ServiceManifest.xml 中從 `Endpoint` 組態使用自動連接埠指派，因為來自 `Endpoint` 組態的自動連接埠指派會針對每個主機處理序指派唯一的連接埠，且單一主機處理序可以包含多個 Kestrel 執行個體。 因為 Kestrel 不支援連接埠共用，這並不會如每個 Kestrel 執行個體必須在唯一連接埠上開啟一般運作。

若要使用動態通訊埠指派與 Kestrel，請完全省略 ServiceManifest.xml 中的 `Endpoint` 組態，且不要將端點名稱傳遞至 `KestrelCommunicationListener` 建構函式︰

```csharp
new KestrelCommunicationListener(serviceContext, (url, listener) => ...
```

在此組態中，`KestrelCommunicationListener` 會自動從應用程式連接埠範圍選取未使用的連接埠。

## <a name="scenarios-and-configurations"></a>案例和組態
本章節描述下列案例，並提供建議的 web 伺服器、通訊埠組態、Service Fabric 整合選項，以及其他設定的組合，來達成正常運作的服務︰
 - 外部公開 ASP.NET Core 無狀態服務
 - 內部公開 ASP.NET Core 無狀態服務
 - 內部公開 ASP.NET Core 具狀態服務

**外部公開**的服務是公開可以從叢集外部連線的端點，通常是透過負載平衡器。

**僅供內部使用**的服務是其端點僅可從叢集內連線。

> [!NOTE]
> 具狀態服務端點通常應該不會公開至網際網路。 位於察覺不到 Service Fabric 服務解析度之負載平衡器後的叢集 (例如 Azure Load Balancer)，將無法公開具狀態服務，因為負載平衡器將無法找出並將流量路由到適當的具狀態服務複本。 

### <a name="externally-exposed-aspnet-core-stateless-services"></a>外部公開 ASP.NET Core 無狀態服務
Kestrel 建議用於前端服務的 web 伺服器，這些服務會公開於外部、網際網路 HTTP 端點。 在 Windows 中，HttpSys 可用來提供連接埠共用功能，讓您使用相同的連接埠在相同的節點集合上裝載多個以主機名稱或路徑區別的 Web 服務，不需要依賴前端 Proxy 或閘道來提供 HTTP 路由。
 
當公開至網際網路時，無狀態服務應使用可透過負載平衡器連線的已知且穩定端點。 這是您提供給使用者的應用程式 URL。 建議使用下列組態：

|  |  | **注意事項** |
| --- | --- | --- |
| Web 伺服器 | Kestrel | Kestrel 是慣用的 Web 伺服器，因為 Windows 和 Linux 均支援它。 |
| 連接埠組態 | 靜態 | 已知的靜態連接埠應在 ServiceManifest.xml 的 `Endpoints` 組態中設定，例如 HTTP 為 80 或 443 為 HTTPS。 |
| ServiceFabricIntegrationOptions | None | 設定 Service Fabric 整合中介軟體時，應使用 `ServiceFabricIntegrationOptions.None` 選項，以便服務不會嘗試驗證唯一識別項的傳入要求。 應用程式的外部使用者不會知道中介軟體所使用的唯一識別資訊。 |
| 執行個體計數 | -1 | 在典型的使用案例中，執行個體計數設定應該設為 "-1"，以便在接收來自負載平衡器之流量的所有節點上可以使用執行個體。 |

如果多個外部公開的服務共用相同的節點集，則 HttpSys 可搭配使用唯一但穩定的 URL 路徑。 這可以透過修改設定 IWebHost 時提供的 URL 來完成。 請注意這僅適用於 HttpSys。

 ```csharp
 new HttpSysCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
 {
     url += "/MyUniqueServicePath";
 
     return new WebHostBuilder()
         .UseHttpSys()
         ...
         .UseUrls(url)
         .Build();
 })
 ```

### <a name="internal-only-stateless-aspnet-core-service"></a>僅供內部使用的無狀態 ASP.NET Core 服務
只會從叢集內呼叫的無狀態服務應該使用唯一的 URL 並動態指派連接埠，以確保多個服務之間的合作。 建議使用下列組態：

|  |  | **注意事項** |
| --- | --- | --- |
| Web 伺服器 | Kestrel | 雖然可能會針對內部的無狀態服務使用 HttpSys，但 Kestrel 是建議的伺服器，可允許多個服務執行個體共用主機。  |
| 連接埠組態 | 動態指派 | 多個具狀態服務的複本可能會共用主機處理序或主機作業系統，因此需要唯一的連接埠。 |
| ServiceFabricIntegrationOptions | UseUniqueServiceUrl | 使用動態連接埠指派，此設定可防止稍早所述的誤用識別問題。 |
| InstanceCount | any | 執行個體計數設定可以設定為操作本服務所需的任何值。 |

### <a name="internal-only-stateful-aspnet-core-service"></a>僅供內部使用的具狀態 ASP.NET Core 服務
只會從叢集內呼叫的具狀態服務應該使用動態指派連接埠，以確保多個服務之間的合作。 建議使用下列組態：

|  |  | **注意事項** |
| --- | --- | --- |
| Web 伺服器 | Kestrel | `HttpSysCommunicationListener`不是設計由複本共用主機處理序的具狀態服務使用。 |
| 連接埠組態 | 動態指派 | 多個具狀態服務的複本可能會共用主機處理序或主機作業系統，因此需要唯一的連接埠。 |
| ServiceFabricIntegrationOptions | UseUniqueServiceUrl | 使用動態連接埠指派，此設定可防止稍早所述的誤用識別問題。 |

## <a name="next-steps"></a>後續步驟
[使用 Visual Studio 偵錯 Service Fabric 應用程式](service-fabric-debugging-your-application.md)

<!--Image references-->
[0]:./media/service-fabric-reliable-services-communication-aspnetcore/webhost-standalone.png
[1]:./media/service-fabric-reliable-services-communication-aspnetcore/webhost-servicefabric.png
[2]:./media/service-fabric-reliable-services-communication-aspnetcore/integration.png
[3]:./media/service-fabric-reliable-services-communication-aspnetcore/httpsys.png
[4]:./media/service-fabric-reliable-services-communication-aspnetcore/kestrel.png
