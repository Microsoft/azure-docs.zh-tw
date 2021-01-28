---
title: 在 .NET Azure Functions 中使用相依性插入
description: 了解如何在 .NET 函式中使用相依性插入來註冊和使用服務
author: ggailey777
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 01/27/2021
ms.author: glenga
ms.reviewer: jehollan
ms.openlocfilehash: 66e2cd22f4bcb95be65d6d04345dcac622436a04
ms.sourcegitcommit: 4e70fd4028ff44a676f698229cb6a3d555439014
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98955083"
---
# <a name="use-dependency-injection-in-net-azure-functions"></a>在 .NET Azure Functions 中使用相依性插入

Azure Functions 支援相依性插入 (DI) 軟體設計模式，這是在類別及其相依性之間達成[控制反轉 (IoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) 的技術。

- Azure Functions 中的相依性插入是以 .NET Core 相依性插入功能為基礎。 建議熟悉 [.NET Core 相依性插入](/aspnet/core/fundamentals/dependency-injection) (機器翻譯)。 您覆寫相依性的方式，與在取用方案上使用 Azure Functions 讀取設定值的方式有所不同。

- 從 Azure Functions 2.x 開始支援相依性插入。

## <a name="prerequisites"></a>Prerequisites

您必須安裝下列 NuGet 套件，才能使用相依性插入：

- [Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions/)

- [Microsoft.NET.Sdk.Functions](https://www.nuget.org/packages/Microsoft.NET.Sdk.Functions/) 套件 1.0.28 版或更新版本

- [DependencyInjection](https://www.nuget.org/packages/Microsoft.Extensions.DependencyInjection/) (目前僅限2.x 版及更早版本的支援) 

## <a name="register-services"></a>註冊伺服器

若要註冊服務，請建立方法來設定元件並將其新增至 `IFunctionsHostBuilder` 執行個體。  Azure Functions 主機會建立 `IFunctionsHostBuilder` 的執行個體，並將其直接傳入方法。

若要註冊方法，請新增 `FunctionsStartup` 組件屬性，以指定啟動期間所使用的類型名稱。

```csharp
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using Microsoft.Extensions.DependencyInjection;

[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.Services.AddHttpClient();

            builder.Services.AddSingleton<IMyService>((s) => {
                return new MyService();
            });

            builder.Services.AddSingleton<ILoggerProvider, MyLoggerProvider>();
        }
    }
}
```

此範例會使用在啟動時註冊 `HttpClient` 所需的 [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http/) 套件。

### <a name="caveats"></a>警示

在執行階段處理啟動類別的前後會執行一系列註冊步驟。 因此，請記住下列項目：

- 啟動類別僅用於設定和註冊。 避免使用啟動時在啟動程序期間註冊的服務。 例如，請勿嘗試在啟動期間註冊的記錄器中記錄訊息。 註冊程序的這個時間點太早，還無法提供服務使用。 執行 `Configure` 方法之後，Functions 執行階段會繼續註冊其他相依性，這可能會影響服務的運作方式。

- 相依性插入容器只會保留明確註冊的類型。 唯一作為可插入類型使用的服務是在 `Configure` 方法中所設定服務。 因此，Function 特定類型 (例如 `BindingContext` 和 `ExecutionContext`) 無法在設定期間使用或作為可插入的類型使用。

## <a name="use-injected-dependencies"></a>使用插入的相依性

您可使用建構函式插入，讓相依性可供函式使用。 使用「函式插入」時，您不需要針對插入的服務或函數類別使用靜態類別。

下列範例示範如何將 `IMyService` 和 `HttpClient` 相依性插入 HTTP 觸發的函式中。

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;
using System.Net.Http;
using System.Threading.Tasks;

namespace MyNamespace
{
    public class MyHttpTrigger
    {
        private readonly HttpClient _client;
        private readonly IMyService _service;

        public MyHttpTrigger(HttpClient httpClient, IMyService service)
        {
            this._client = httpClient;
            this._service = service;
        }

        [FunctionName("MyHttpTrigger")]
        public async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            var response = await _client.GetAsync("https://microsoft.com");
            var message = _service.GetMessage();

            return new OkObjectResult("Response from function with injected dependencies.");
        }
    }
}
```

此範例會使用在啟動時註冊 `HttpClient` 所需的 [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http/) 套件。

## <a name="service-lifetimes"></a>服務存留期

Azure Functions 應用程式提供與 [ASP.NET 相依性插入](/aspnet/core/fundamentals/dependency-injection#service-lifetimes)相同的服務存留期。 針對 Functions 應用程式，不同服務存留期的行為如下所示：

- **暫時性**：每次服務解析時，就會建立暫時性服務。
- **限域**：限域服務存留期符合函式執行存留期。 針對每個函式執行建立一次範圍服務。 稍後在執行期間對該服務提出的要求會重複使用現有服務執行個體。
- **單一**：單一服務存留期符合主機存留期，並會在該執行個體上跨函式執行重複使用。 建議連線和用戶端使用單一存留期服務，例如 `DocumentClient` 或 `HttpClient` 執行個體。

請前往 GitHub 檢視或下載[不同服務存留期的範本](https://github.com/Azure/azure-functions-dotnet-extensions/tree/main/src/samples/DependencyInjection/Scopes)。

## <a name="logging-services"></a>記錄服務

如果您需要自己的記錄提供者，請將自訂類型註冊為的實例 [`ILoggerProvider`](/dotnet/api/microsoft.extensions.logging.iloggerfactory) ，此實例可透過「 [記錄檔抽象概念](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Abstractions/) 」 NuGet 套件取得。

Azure Functions 會自動新增 Application Insights。

> [!WARNING]
> - 請勿新增 `AddApplicationInsightsTelemetry()` 至服務集合，該集合會註冊與環境所提供之服務衝突的服務。
> - `TelemetryConfiguration` `TelemetryClient` 如果您使用內建的 Application Insights 功能，請不要註冊您自己的。 如果您需要設定自己 `TelemetryClient` 的實例，請透過插入， `TelemetryConfiguration` 如 c # 函式 [中記錄自訂遙測](functions-dotnet-class-library.md?tabs=v2%2Ccmd#log-custom-telemetry-in-c-functions)所示，建立一個實例。

### <a name="iloggert-and-iloggerfactory"></a>ILogger<T> 和 ILoggerFactory

主機會將 `ILogger<T>` 和 `ILoggerFactory` 服務插入至函式。  不過，根據預設，這些新的記錄篩選器會篩選掉函數記錄。  您需要修改檔案 `host.json` ，以加入其他篩選準則和類別。

下列範例示範如何 `ILogger<HttpTrigger>` 使用公開給主機的記錄來新增。

```csharp
namespace MyNamespace
{
    public class HttpTrigger
    {
        private readonly ILogger<HttpTrigger> _log;

        public HttpTrigger(ILogger<HttpTrigger> log)
        {
            _log = log;
        }

        [FunctionName("HttpTrigger")]
        public async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req)
        {
            _log.LogInformation("C# HTTP trigger function processed a request.");

            // ...
    }
}
```

下列範例檔案會 `host.json` 加入記錄篩選。

```json
{
    "version": "2.0",
    "logging": {
        "applicationInsights": {
            "samplingSettings": {
                "isEnabled": true,
                "excludedTypes": "Request"
            }
        },
        "logLevel": {
            "MyNamespace.HttpTrigger": "Information"
        }
    }
}
```

如需有關記錄層級的詳細資訊，請參閱 [設定記錄層級](configure-monitoring.md#configure-log-levels)。

## <a name="function-app-provided-services"></a>函數應用程式提供的服務

函式主機會註冊許多服務。 下列服務可安全地作為應用程式中的相依性：

|服務類型|存留期|描述|
|--|--|--|
|`Microsoft.Extensions.Configuration.IConfiguration`|單一|執行階段設定|
|`Microsoft.Azure.WebJobs.Host.Executors.IHostIdProvider`|單一|負責提供主機執行個體的識別碼|

如有其他要相依的服務，請[在 GitHub 上建立並張貼問題](https://github.com/azure/azure-functions-host)。

### <a name="overriding-host-services"></a>覆寫主機服務

目前不支援覆寫主機所提供的服務。  如有要覆寫的服務，請[在 GitHub 上建立並張貼問題](https://github.com/azure/azure-functions-host)。

## <a name="working-with-options-and-settings"></a>使用選項和設定

[應用程式設定](./functions-how-to-use-azure-function-app-settings.md#settings)中定義的值位於 `IConfiguration` 執行個體中，其可供讀取啟動類別中的應用程式設定值。

您可將值從 `IConfiguration` 執行個體擷取到自訂類型中。 將應用程式設定值複製到自訂類型會使這些值得以插入，以供輕鬆地測試服務。 讀入設定執行個體的設定必須是簡單的索引鍵/值組。

請考慮下列類別，其中包含名稱與應用程式設定一致的屬性：

```csharp
public class MyOptions
{
    public string MyCustomSetting { get; set; }
}
```

而 `local.settings.json` 檔案可能會建構自訂設定，如下所示：
```json
{
  "IsEncrypted": false,
  "Values": {
    "MyOptions:MyCustomSetting": "Foobar"
  }
}
```

您可從 `Startup.Configure` 方法內部使用下列程式碼，以將值從 `IConfiguration` 執行個體擷取到自訂類型中：

```csharp
builder.Services.AddOptions<MyOptions>()
    .Configure<IConfiguration>((settings, configuration) =>
    {
        configuration.GetSection("MyOptions").Bind(settings);
    });
```

呼叫 `Bind` 會將具有相符屬性名稱的值從設定複製到自訂執行個體中。 選項執行個體現在可供 IoC 容器用來插入函式中。

選項物件會作為一般 `IOptions` 介面的執行個體插入函式中。 您可使用 `Value` 屬性來存取在設定中找到的值。

```csharp
using System;
using Microsoft.Extensions.Options;

public class HttpTrigger
{
    private readonly MyOptions _settings;

    public HttpTrigger(IOptions<MyOptions> options)
    {
        _settings = options.Value;
    }
}
```

如需使用選項的詳細資訊，請參閱 [ASP.NET Core 中的選項模式](/aspnet/core/fundamentals/configuration/options)。

## <a name="using-aspnet-core-user-secrets"></a>使用 ASP.NET Core 的使用者秘密

在本機開發時，ASP.NET Core 提供 [秘密管理員工具](/aspnet/core/security/app-secrets#secret-manager) ，可讓您將秘密資訊儲存在專案根目錄之外。 這會讓秘密不會不慎認可到原始檔控制。 Azure Functions Core Tools (3.0.3233 版或更新版本) 會自動讀取 ASP.NET Core 秘密管理員所建立的秘密。

若要將 .NET Azure Functions 專案設定為使用使用者密碼，請在專案根目錄中執行下列命令。

```bash
dotnet user-secrets init
```

然後使用 `dotnet user-secrets set` 命令來建立或更新秘密。

```bash
dotnet user-secrets set MySecret "my secret value"
```

若要存取函式應用程式程式碼中的使用者秘密值，請使用 `IConfiguration` 或 `IOptions` 。

## <a name="customizing-configuration-sources"></a>自訂設定來源

> [!NOTE]
> 從 Azure Functions 主機版本2.0.14192.0 和3.0.14191.0 開始，可以開始進行設定來源自訂。

若要指定其他設定來源，請覆寫函式 `ConfigureAppConfiguration` 應用程式類別中的方法 `StartUp` 。

下列範例會新增基底的設定值，以及選用的環境專屬應用程式佈建檔。

```csharp
using System.IO;
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void ConfigureAppConfiguration(IFunctionsConfigurationBuilder builder)
        {
            FunctionsHostBuilderContext context = builder.GetContext();

            builder.ConfigurationBuilder
                .AddJsonFile(Path.Combine(context.ApplicationRootPath, "appsettings.json"), optional: true, reloadOnChange: false)
                .AddJsonFile(Path.Combine(context.ApplicationRootPath, $"appsettings.{context.EnvironmentName}.json"), optional: true, reloadOnChange: false)
                .AddEnvironmentVariables();
        }
    }
}
```

將設定提供者加入的 `ConfigurationBuilder` 屬性 `IFunctionsConfigurationBuilder` 。 如需使用設定提供者的詳細資訊，請參閱 [ASP.NET Core 中](/aspnet/core/fundamentals/configuration/#configuration-providers)的設定。

`FunctionsHostBuilderContext`從取得 `IFunctionsConfigurationBuilder.GetContext()` 。 使用此內容可取得目前的環境名稱，並解析函數應用程式資料夾中的設定檔位置。

根據預設，設定檔（例如 *appsettings.js* ）不會自動複製到函數應用程式的輸出檔案夾。 更新 *.csproj* 檔案以符合下列範例，以確定檔案已複製。

```xml
<None Update="appsettings.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>      
</None>
<None Update="appsettings.Development.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    <CopyToPublishDirectory>Never</CopyToPublishDirectory>
</None>
```

> [!IMPORTANT]
> 針對在耗用量或高階方案中執行的函式應用程式，修改觸發程式中使用的設定值可能會導致調整錯誤。 類別對這些屬性所做的任何變更都會導致函式 `FunctionsStartup` 應用程式啟動錯誤。

## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱下列資源：

- [如何監視函數應用程式](functions-monitoring.md)
- [函式的最佳做法](functions-best-practices.md)
