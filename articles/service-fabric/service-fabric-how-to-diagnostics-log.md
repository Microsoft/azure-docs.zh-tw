---
title: 從 Azure 或獨立叢集中的 .NET Service Fabric 應用程式產生記錄事件
description: 了解如何為裝載在 Azure 叢集或獨立叢集上的 .NET Service Fabric 應用程式新增記錄功能。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/27/2018
ms.author: ryanwi
ms.openlocfilehash: 42a6430162f3bafd3ec3ce2a3c523f6f5755914a
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39001373"
---
# <a name="add-logging-to-your-service-fabric-application"></a>為 Service Fabric 應用程式新增記錄功能

您的應用程式必須有足夠的資訊，才能在發生問題時以抽絲剝繭的方式進行偵錯。 記錄功能是您可以為 Service Fabric 應用程式新增的最重要功能之一。 發生失敗時，良好的記錄功能可為您提供一個調查失敗的方向。 藉由分析記錄模式，您可以找出改善應用程式效能或設計的方式。 本文件會示範幾個不同的記錄選項。

## <a name="eventflow"></a>EventFlow

[EventFlow 程式庫](https://github.com/Azure/diagnostics-eventflow)套件可讓應用程式定義要收集哪些診斷資料，以及應將這些資料輸出至何處。 診斷資料可以是包括從效能計數器到應用程式追蹤在內的任何資料。 它會在與應用程式相同的處理序中執行，因此可將通訊額外負荷降到最低。 如需有關 EventFlow 和 Service Fabric 的詳細資訊，請參閱[使用 EventFlow 進行 Azure Service Fabric 事件彙總](service-fabric-diagnostics-event-aggregation-eventflow.md)。

### <a name="using-structured-eventsource-events"></a>使用結構化 EventSource 事件

依使用案例定義訊息事件可讓您在事件內容中，封裝事件的相關資料。 您可以更容易根據所指定事件屬性的名稱或值來進行搜尋和篩選。 建構檢測輸出可讓您更容易讀取，但需要更多考量和時間來為每個使用案例定義事件。 

某些事件定義可以在整個應用程式內共用。 例如，應用程式內的許多不同服務會重複使用方法啟動或停止事件。 特定網域服務有自己的獨特事件，例如，訂單系統可能有 **CreateOrder** 事件。 此方法可能產生很多事件，專案小組之間可能需要協調識別碼。 

```csharp
[EventSource(Name = "MyCompany-VotingState-VotingStateService")]
internal sealed class ServiceEventSource : EventSource
{
    public static readonly ServiceEventSource Current = new ServiceEventSource();

    // The instance constructor is private to enforce singleton semantics.
    private ServiceEventSource() : base() { }

    ...

    // The ServiceTypeRegistered event contains a unique identifier, an event attribute that defined the event, and the code implementation of the event.
    private const int ServiceTypeRegisteredEventId = 3;
    [Event(ServiceTypeRegisteredEventId, Level = EventLevel.Informational, Message = "Service host process {0} registered service type {1}", Keywords = Keywords.ServiceInitialization)]
    public void ServiceTypeRegistered(int hostProcessId, string serviceType)
    {
        WriteEvent(ServiceTypeRegisteredEventId, hostProcessId, serviceType);
    }

    // The ServiceHostInitializationFailed event contains a unique identifier, an event attribute that defined the event, and the code implementation of the event.
    private const int ServiceHostInitializationFailedEventId = 4;
    [Event(ServiceHostInitializationFailedEventId, Level = EventLevel.Error, Message = "Service host initialization failed", Keywords = Keywords.ServiceInitialization)]
    public void ServiceHostInitializationFailed(string exception)
    {
        WriteEvent(ServiceHostInitializationFailedEventId, exception);
    }

    ...

```

### <a name="using-eventsource-generically"></a>以一般方式使用 EventSource

因為定義特定事件可能很困難，許多人會以一組常用的參數定義少量事件，且通常會將資訊輸出為字串。 大部分的結構層面會遺失，因此更難搜尋和篩選結果。 此方法定義的少量事件通常對應至記錄層級。 下列程式碼片段定義偵錯和錯誤訊息：

```csharp
[EventSource(Name = "MyCompany-VotingState-VotingStateService")]
internal sealed class ServiceEventSource : EventSource
{
    public static readonly ServiceEventSource Current = new ServiceEventSource();

    // The Instance constructor is private, to enforce singleton semantics.
    private ServiceEventSource() : base() { }

    ...

    private const int DebugEventId = 10;
    [Event(DebugEventId, Level = EventLevel.Verbose, Message = "{0}")]
    public void Debug(string msg)
    {
        WriteEvent(DebugEventId, msg);
    }

    private const int ErrorEventId = 11;
    [Event(ErrorEventId, Level = EventLevel.Error, Message = "Error: {0} - {1}")]
    public void Error(string error, string msg)
    {
        WriteEvent(ErrorEventId, error, msg);
    }

    ...

```

使用混合式的結構化和泛型檢測也可行。 結構化檢測用於報告錯誤和計量。 泛型事件可用來產生詳細記錄，讓工程師用於疑難排解。

## <a name="microsoftextensionslogging"></a>Microsoft.Extensions.Logging

ASP.NET Core 記錄 ([Microsoft.Extensions.Logging NuGet 套件](https://www.nuget.org/packages/Microsoft.Extensions.Logging)) 是一個為應用程式提供標準記錄 API 的記錄架構。 您可以在 ASP.NET Core 記錄中插入對其他記錄後端的支援。 這可針對所處理應用程式中的記錄提供各種支援，而無須變更太多程式碼。

1. 將 **Microsoft.Extensions.Logging** NuGet 套件新增至您要檢測的專案。 此外，也新增任何提供者套件。 如需詳細資訊，請參閱 [ASP.NET Core 中的記錄](https://docs.microsoft.com/aspnet/core/fundamentals/logging)。
2. 針對 **Microsoft.Extensions.Logging**，將 **using** 指示詞新增至您的服務檔案。
3. 在服務類別內定義私用變數。

   ```csharp
   private ILogger _logger = null;
   ```

4. 在服務類別的建構函式中，新增此程式碼：

   ```csharp
   _logger = new LoggerFactory().CreateLogger<Stateless>();
   ```

5. 開始用您的方法檢測程式碼。 以下是幾個範例：

   ```csharp
   _logger.LogDebug("Debug-level event from Microsoft.Logging");
   _logger.LogInformation("Informational-level event from Microsoft.Logging");

   // In this variant, we're adding structured properties RequestName and Duration, which have values MyRequest and the duration of the request.
   // Later in the article, we discuss why this step is useful.
   _logger.LogInformation("{RequestName} {Duration}", "MyRequest", requestDuration);
   ```

### <a name="using-other-logging-providers"></a>使用其他記錄提供者

某些協力廠商提供者會使用上節所述的方法，包括 [Serilog](https://serilog.net/)、[NLog](http://nlog-project.org/) 和 [Loggr](https://github.com/imobile3/Loggr.Extensions.Logging)。 您可以將這些全部插入 ASP.NET Core 記錄中，也可以分開使用。 Serilog 有一項功能讓記錄器傳來的所有訊息更豐富。 這項功能可用來輸出服務名稱、類型和資料分割資訊。 若要在 ASP.NET Core 基礎結構中使用這項功能，請執行下列步驟：

1. 將 **Serilog****Serilog.Extensions.Logging****Serilog.Sinks.Literate** 及 **Serilog.Sinks.Observable** NuGet 套件新增至專案。 
2. 建立 `LoggerConfiguration` 和記錄器執行個體。

   ```csharp
   Log.Logger = new LoggerConfiguration().WriteTo.LiterateConsole().CreateLogger();
   ```

3. 將 `Serilog.ILogger` 引數新增至服務建構函式，並傳遞新建立的記錄器。

   ```csharp
   ServiceRuntime.RegisterServiceAsync("StatelessType", context => new Stateless(context, Log.Logger)).GetAwaiter().GetResult();
   ```

4. 在服務建構函式中，為 **ServiceTypeName**、**ServiceName**、**PartitionId** 及 **InstanceId** 建立屬性豐富器。

   ```csharp
   public Stateless(StatelessServiceContext context, Serilog.ILogger serilog)
       : base(context)
   {
       PropertyEnricher[] properties = new PropertyEnricher[]
       {
           new PropertyEnricher("ServiceTypeName", context.ServiceTypeName),
           new PropertyEnricher("ServiceName", context.ServiceName),
           new PropertyEnricher("PartitionId", context.PartitionId),
           new PropertyEnricher("InstanceId", context.ReplicaOrInstanceId),
       };

       serilog.ForContext(properties);

       _logger = new LoggerFactory().AddSerilog(serilog.ForContext(properties)).CreateLogger<Stateless>();
   }
   ```

5. 檢測程式碼，就如同在沒有 SeriLog 的情況下使用 ASP.NET Core 一樣。

   >[!NOTE]
   >我們建議您「不要」在上述範例中使用靜態 `Log.Logger`。 Service Fabric 可以在單一處理序內裝載相同服務類型的多個執行個體。 如果您使用靜態 `Log.Logger`，屬性豐富器的最後一個寫入器會顯示所有執行中執行個體的值。 這是為什麼 _logger 變數是服務類別私人成員變數的其中一個原因。 此外，您還必須讓 `_logger` 可供各個服務可能用到的通用程式碼使用。

## <a name="next-steps"></a>後續步驟

- 深入了解 [Service Fabric 中的應用程式監視](service-fabric-diagnostics-event-generation-app.md)。
- 深入了解如何使用 [EventFlow](service-fabric-diagnostics-event-aggregation-eventflow.md) 和 [Microsoft Azure 診斷](service-fabric-diagnostics-event-aggregation-wad.md)進行記錄。










