---
title: Azure 服務匯流排端對端追蹤與診斷 | Microsoft Docs
description: 服務匯流排用戶端診斷與端對端追蹤的概觀
services: service-bus-messaging
documentationcenter: ''
author: lmolkova
manager: timlt
editor: ''
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/18/2017
ms.author: lmolkova
ms.openlocfilehash: 5489fa999f3427345c3ee9f07f904296de224e31
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42141291"
---
# <a name="distributed-tracing-and-correlation-through-service-bus-messaging"></a>透過服務匯流排傳訊進行分散式追蹤與相互關聯

微服務開發的其中一個常見問題，便是經由涉及處理程序的所有服務，追蹤來自用戶端之作業的能力。 此能力對於進行偵錯、效能分析、A/B 測試，以及其他一般診斷案例皆相當有用。
此問題的其中一部份，辨識追蹤作業的邏輯片段。 它包含訊息處理結果及延遲，以及外部相依性呼叫。 另外一部分，則是這些診斷事件於程序界線之外的關聯性。

當產生者透過佇列傳送訊息時，它通常會發生於其他邏輯作業的範圍中，並由其他用戶端或服務起始。 當取用者接收到訊息時，也會繼續相同的作業。 產生者與取用者 (以及其他處理該作業的服務) 應該都會發出遙測事件，以追蹤作業流程和結果。 若要將此類事件相互關聯並以端對端的方式追蹤作業，每個回報遙測的服務都必須為每個事件提供追蹤內容的戳記。

Microsoft Azure 服務匯流排傳訊已定義產生者與取用者應用來傳遞此類追蹤內容的裝載屬性。
該通訊協定是以 [HTTP 關聯性通訊協定](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md) \(英文\) 為基礎。

| 屬性名稱        | 說明                                                 |
|----------------------|-------------------------------------------------------------|
|  Diagnostic-Id       | 產生者針對佇列之外部呼叫的唯一識別碼。 請參閱 [HTTP 通訊協定中的 Request-Id](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md#request-id) \(英文\) 以了解邏輯依據、考量及格式 |
|  Correlation-Context | 作業內容，系統會將它傳播至涉及作業處理的所有服務。 如需詳細資訊，請參閱 [HTTP 通訊協定中的 Correlation-Context](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md#correlation-context) \(英文\) |

## <a name="service-bus-net-client-auto-tracing"></a>服務匯流排 .NET 用戶端自動追蹤

從 3.0.0 版開始，[適用於 .NET 的 Microsoft Azure 服務匯流排用戶端](/dotnet/api/microsoft.azure.servicebus.queueclient)會提供追蹤檢測點，可由追蹤系統或用戶端程式碼片段連結。
該檢測允許從用戶端追蹤針對服務匯流排傳訊服務的所有呼叫。 若訊息處理是透過[訊息處理常式模式](/dotnet/api/microsoft.azure.servicebus.queueclient.registermessagehandler)完成，則訊息處理也會進行檢測處理

### <a name="tracking-with-azure-application-insights"></a>透過 Azure Application Insights 進行追蹤

[Microsoft Application Insights](https://azure.microsoft.com/services/application-insights/) 能提供豐富的效能監視功能，包括自動要求和相依性追蹤。

請根據您的專案類型安裝 Application Insights SDK：
- [ASP.NET](../application-insights/app-insights-asp-net.md) - 安裝 2.5-beta2 版或更新版本
- [ASP.NET Core](../application-insights/app-insights-asp-net-core.md) - 安裝 2.2.0-beta2 版或更新版本。
這些連結提供安裝 SDK、建立資源及設定 SDK (若有需要) 的詳細資料。 針對非 ASP.NET 應用程式，請參閱[適用於主控台應用程式的 Azure Application Insights](../application-insights/application-insights-console.md) 一文。

如果您使用[訊息處理常式模式](/dotnet/api/microsoft.azure.servicebus.queueclient.registermessagehandler)來處理訊息，便無需採取任何動作，系統會自動追蹤由您的服務所完成的所有服務匯流排呼叫，並將它們與其他遙測項目相互關聯。 否則，請參考下列範例以進行手動的訊息處理追蹤。

#### <a name="trace-message-processing"></a>追蹤訊息處理

```csharp
private readonly TelemetryClient telemetryClient;

async Task ProcessAsync(Message message)
{
    var activity = message.ExtractActivity();
    
    // If you are using Microsoft.ApplicationInsights package version 2.6-beta or higher, you should call StartOperation<RequestTelemetry>(activity) instead
    using (var operation = telemetryClient.StartOperation<RequestTelemetry>("Process", activity.RootId, activity.ParentId))
    {
        telemetryClient.TrackTrace("Received message");
        try 
        {
           // process message
        }
        catch (Exception ex)
        {
             telemetryClient.TrackException(ex);
             operation.Telemetry.Success = false;
             throw;
        }

        telemetryClient.TrackTrace("Done");
   }
}
```

在此範例中，系統會針對每個已處理的訊息回報 `RequestTelemetry`，並具有時間戳記、持續期間及結果 (成功)。 遙測也具有相互關聯屬性的集合。
於訊息處理期間回報的巢狀追蹤和例外狀況，也會具有相互關聯屬性的戳記，以代表它們是 `RequestTelemetry` 的「子系」。

在您於訊息處理期間對支援的外部元件做出呼叫的情況下，系統也會對它們進行自動追蹤及相互關聯。 請參閱[使用 Application Insights .NET SDK 追蹤自訂作業](../application-insights/application-insights-custom-operations-tracking.md)以了解手動追蹤及相互關聯。

### <a name="tracking-without-tracing-system"></a>在沒有追蹤系統下進行追蹤
在您的追蹤系統不支援自動服務匯流排呼叫追蹤的情況下，可以考慮將該支援新增至追蹤系統或是您的應用程式中。 本節說明由服務匯流排 .NET 用戶端所傳送的診斷事件。  

服務匯流排 .NET 用戶端是使用 .NET 追蹤基本類型 [System.Diagnostics.Activity](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md) \(英文\) 和 [System.Diagnostics.DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md) \(英文\) 進行檢測。

`Activity` 會作為追蹤內容，而 `DiagnosticSource` 則為通知機制。 

如果沒有適用於 DiagnosticSource 事件的接聽程式，檢測將會關閉以避免產生檢測成本。n costs. DiagnosticSource 會將所有控制項賦予接聽程式：
- 接聽程式能控制要接聽的來源和事件
- 接聽程式能控制事件速率和取樣
- 事件會搭配裝載傳送，該承載能提供完整內容，使您可以在事件期間存取並修改 Message 物件

在繼續進行實作之前，請先熟悉 [DiagnosticSource 使用者指南](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md) \(英文\)。

讓我們在 ASP.NET Core 應用程式中針對服務匯流排事件建立接聽程式，並使它可以透過 Microsoft.Extension.Logger 寫入記錄。
它會使用 [System.Reactive.Core](https://www.nuget.org/packages/System.Reactive.Core) \(英文\) 程式庫來訂閱 DiagnosticSource (不使用它來訂閱 DiagnosticSource 的方式也十分簡單)

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory factory, IApplicationLifetime applicationLifetime)
{
    // configuration...

    var serviceBusLogger = factory.CreateLogger("Microsoft.Azure.ServiceBus");

    IDisposable innerSubscription = null;
    IDisposable outerSubscription = DiagnosticListener.AllListeners.Subscribe(delegate (DiagnosticListener listener)
    {
        // subscribe to the Service Bus DiagnosticSource
        if (listener.Name == "Microsoft.Azure.ServiceBus")
        {
            // receive event from Service Bus DiagnosticSource
            innerSubscription = listener.Subscribe(delegate (KeyValuePair<string, object> evnt)
            {
                // Log operation details once it's done
                if (evnt.Key.EndsWith("Stop"))
                {
                    Activity currentActivity = Activity.Current;
                    TaskStatus status = (TaskStatus)evnt.Value.GetProperty("Status");
                    serviceBusLogger.LogInformation($"Operation {currentActivity.OperationName} is finished, Duration={currentActivity.Duration}, Status={status}, Id={currentActivity.Id}, StartTime={currentActivity.StartTimeUtc}");
                }
            });
        }
    });

    applicationLifetime.ApplicationStopping.Register(() =>
    {
        outerSubscription?.Dispose();
        innerSubscription?.Dispose();
    });
}
```

在此範例中，接聽程式會記錄每個服務匯流排作業的持續期間、結果、唯一識別碼，以及開始時間。

#### <a name="events"></a>活動

針對每個作業，系統會傳送兩個事件：'Start' 和 'Stop'。 您應該只會對 'Stop' 事件感到興趣。 它們會提供作業的結果，並以 Activity 屬性的形式提供開始時間和持續期間。

每個裝載都會為接聽程式提供作業的內容，它會複寫 API 傳入參數和傳回值。 'Stop' 事件裝載具有 'Start' 事件裝載的所有屬性，因此您可以完全忽略 'Start' 事件。

所有事件也都有 'Entity' 和 'Endpoint' 屬性，下方表格將會省略它們
  * `string Entity`：實體 (佇列、主題等) 的名稱
  * `Uri Endpoint`：服務匯流排端點 URL

每個 'Stop' 事件都具有 `TaskStatus` 非同步作業為已完成的 `Status` 屬性，這也會在下方表格中省略以求精簡。

以下是檢測作業的完整清單：

| 作業名稱 | 追蹤的 API | 特定的裝載屬性|
|----------------|-------------|---------|
| Microsoft.Azure.ServiceBus.Send | [MessageSender.SendAsync](/dotnet/api/microsoft.azure.servicebus.core.messagesender.sendasync) | IList<Message> Messages：正在傳送之訊息的清單 |
| Microsoft.Azure.ServiceBus.ScheduleMessage | [MessageSender.ScheduleMessageAsync](/dotnet/api/microsoft.azure.servicebus.core.messagesender.schedulemessageasync) | Message Message：正在處理的訊息<br/>DateTimeOffset ScheduleEnqueueTimeUtc：已排程訊息位移<br/>long SequenceNumber：已排程訊息的序號 ('Stop' 事件裝載) |
| Microsoft.Azure.ServiceBus.Cancel | [MessageSender.CancelScheduledMessageAsync](/dotnet/api/microsoft.azure.servicebus.core.messagesender.cancelscheduledmessageasync) | long SequenceNumber：要取消之訊息的序號 | 
| Microsoft.Azure.ServiceBus.Receive | [MessageReceiver.ReceiveAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.receiveasync) |int RequestedMessageCount：可接收訊息的數目上限。<br/>IList<Message> Messages：已接收訊息的清單 ('Stop' 事件裝載) |
| Microsoft.Azure.ServiceBus.Peek | [MessageReceiver.PeekAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.peekasync) | int FromSequenceNumber：瀏覽訊息批次的起點。<br/>int RequestedMessageCount：要擷取之訊息的數目。<br/>IList<Message> Messages：已接收訊息的清單 ('Stop' 事件裝載) |
| Microsoft.Azure.ServiceBus.ReceiveDeferred | [MessageReceiver.ReceiveDeferredMessageAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.receivedeferredmessageasync) | IEnumerable<long> SequenceNumbers：包含要接收之序號的清單。<br/>IList<Message> Messages：已接收訊息的清單 ('Stop' 事件裝載) |
| Microsoft.Azure.ServiceBus.Complete | [MessageReceiver.CompleteAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.completeasync) | IList<string> LockTokens：包含要完成對應訊息之鎖定 Token 的清單。|
| Microsoft.Azure.ServiceBus.Abandon | [MessageReceiver.AbandonAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.abandonasync) | string LockToken：要放棄之對應訊息的鎖定 Token。 |
| Microsoft.Azure.ServiceBus.Defer | [MessageReceiver.DeferAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.deferasync) | string LockToken：要延後之對應訊息的鎖定 Token。 | 
| Microsoft.Azure.ServiceBus.DeadLetter | [MessageReceiver.DeadLetterAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.deadletterasync) | string LockToken：要進行無效信件處理之對應訊息的鎖定 Token。 | 
| Microsoft.Azure.ServiceBus.RenewLock | [MessageReceiver.RenewLockAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.renewlockasync) | string LockToken：要更新鎖定之對應訊息的鎖定 Token。<br/>DateTime LockedUntilUtc：以 UTC 格式呈現的新鎖定 Token 到期日和時間。 ('Stop' 事件裝載)|
| Microsoft.Azure.ServiceBus.Process | 於 [IReceiverClient.RegisterMessageHandler](/dotnet/api/microsoft.azure.servicebus.core.ireceiverclient.registermessagehandler) 中提供的訊息處理常式 Lambda 函式 | Message Message：正在處理的訊息。 |
| Microsoft.Azure.ServiceBus.ProcessSession | 於 [IQueueClient.RegisterSessionHandler](/dotnet/api/microsoft.azure.servicebus.iqueueclient.registersessionhandler) 中提供的訊息工作階段處理常式 Lambda 函式 | Message Message：正在處理的訊息。<br/>IMessageSession Session：正在處理的工作階段 |
| Microsoft.Azure.ServiceBus.AddRule | [SubscriptionClient.AddRuleAsync](/dotnet/api/microsoft.azure.servicebus.subscriptionclient.addruleasync) | RuleDescription Rule：提供要新增之規則的規則描述。 |
| Microsoft.Azure.ServiceBus.RemoveRule | [SubscriptionClient.RemoveRuleAsync](/dotnet/api/microsoft.azure.servicebus.subscriptionclient.removeruleasync) | string RuleName：要移除之規則的名稱。 |
| Microsoft.Azure.ServiceBus.GetRules | [SubscriptionClient.GetRulesAsync](/dotnet/api/microsoft.azure.servicebus.subscriptionclient.getrulesasync) | IEnumerable<RuleDescription> Rules：與訂用帳戶相關聯的所有規則。 (僅限 'Stop' 裝載) |
| Microsoft.Azure.ServiceBus.AcceptMessageSession | [ISessionClient.AcceptMessageSessionAsync](/dotnet/api/microsoft.azure.servicebus.isessionclient.acceptmessagesessionasync) | string SessionId：存在於訊息中的 sessionId。 |
| Microsoft.Azure.ServiceBus.GetSessionState | [IMessageSession.GetStateAsync](/dotnet/api/microsoft.azure.servicebus.imessagesession.getstateasync) | string SessionId：存在於訊息中的 sessionId。<br/>byte [] State：工作階段狀態 ('Stop' 事件裝載) |
| Microsoft.Azure.ServiceBus.SetSessionState | [IMessageSession.SetStateAsync](/dotnet/api/microsoft.azure.servicebus.imessagesession.setstateasync) | string SessionId：存在於訊息中的 sessionId。<br/>byte [] State：工作階段狀態 |
| Microsoft.Azure.ServiceBus.RenewSessionLock | [IMessageSession.RenewSessionLockAsync](/dotnet/api/microsoft.azure.servicebus.imessagesession.renewsessionlockasync) | string SessionId：存在於訊息中的 sessionId。 |
| Microsoft.Azure.ServiceBus.Exception | 任何經檢測的 API| Exception Exception：例外狀況執行個體 |

在每個事件中，您都可以存取保存目前作業內容的 `Activity.Current`。

#### <a name="logging-additional-properties"></a>記錄其他屬性

`Activty.Current` 會提供目前作業及其父系的詳細內容。 如需詳細資訊，請參閱 [Activity 文件](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md) \(英文\) 以取得詳細資料。
服務匯流排檢測會在 `Activity.Current.Tags` 中提供額外資訊，它們會保存 `MessageId` 和 `SessionId` (若有提供)。

追蹤 'Receive'、'Peek' 及 'ReceiveDeferred' 事件的活動，也可能會有 `RelatedTo` 標記。 它會保存結果所接收到訊息之 `Diagnostic-Id` 的相異清單。
這類作業可能會導致接收到數個不相關的訊息。 此外，`Diagnostic-Id` 於作業開始時為未知，因此 'Receive' 作業只能透過使用此標記來與 'Process' 作業相互關聯。 它在分析效能問題時相當有用，並可以檢查接收訊息所需花費的時間。

記錄 Tags 的有效方式為重複處理它們，因此若要將 Tags 新增至上述範例，將會如下所示： 

```csharp
Activity currentActivity = Activity.Current;
TaskStatus status = (TaskStatus)evnt.Value.GetProperty("Status");

var tagsList = new StringBuilder();
foreach (var tags in currentActivity.Tags)
{
    tagsList.Append($", "{tags.Key}={tags.Value}");
}

serviceBusLogger.LogInformation($"{currentActivity.OperationName} is finished, Duration={currentActivity.Duration}, Status={status}, Id={currentActivity.Id}, StartTime={currentActivity.StartTimeUtc}{tagsList}");
```

#### <a name="filtering-and-sampling"></a>篩選和取樣

在某些情況下，您可能只會想記錄部分的事件，以減少效能額外負荷或儲存空間耗用量。 您可以僅記錄 'Stop' 事件 (如上述範例所示)，或是對特定百分比的事件進行取樣。 
`DiagnosticSource` 提供搭配 `IsEnabled` 述詞來達成它的方式。 如需詳細資訊，請參閱 [DiagnosticSource 中以內容為基礎的篩選](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md#context-based-filtering) \(英文\)。

`IsEnabled` 可以針對單一作業呼叫數次，以將效能影響降至最低。

系統會於下列序列中呼叫 `IsEnabled`：

1. 以 `IsEnabled(<OperationName>, string entity, null)` 為例，`IsEnabled("Microsoft.Azure.ServiceBus.Send", "MyQueue1")`。 請注意到末端並沒有 'Start' 或 'Stop'。 請使用它來將特定作業或佇列篩選出來。 若回呼傳回 `false`，代表針對作業的事件並沒有傳送

  * 針對 'Process' 和 'ProcessSession' 作業，您也會接收到 `IsEnabled(<OperationName>, string entity, Activity activity)` 回呼。 請使用它來根據 `activity.Id` 或 Tags 屬性篩選事件。
  
2. 以 `IsEnabled(<OperationName>.Start)` 為例，`IsEnabled("Microsoft.Azure.ServiceBus.Send.Start")`。 檢查是否應該引發 'Start' 事件。 結果只會影響 'Start' 事件，但進一步的檢測並不會相依於它。

'Stop' 事件沒有 `IsEnabled`。

若某個作業結果為例外狀況，系統便會呼叫 `IsEnabled("Microsoft.Azure.ServiceBus.Exception")`。 您只能訂閱 'Exception' 事件並避免剩下的檢測。 在此情況下，您仍然必須處理這類例外狀況。 由於其他檢測皆已停用，您不應預期追蹤內容會有從取用者傳送至產生者的訊息。

您也可以使用 `IsEnabled` 來實作取樣策略。 以 `Activity.Id` 或 `Activity.RootId` 為基礎的取樣，能確保在所有嘗試上皆取得一致的取樣 (只要它是由追蹤系統或您自己的程式碼所傳播)。

在存在針對相同來源之多個 `DiagnosticSource` 接聽程式的情況下，只要其中一個接聽程式接受事件便已足夠，因此並無法保證會呼叫 `IsEnabled`。

## <a name="next-steps"></a>後續步驟

* [服務匯流排基本概念](service-bus-fundamentals-hybrid-solutions.md)
* [Application Insights 相互關聯](../application-insights/application-insights-correlation.md)
* [Application Insights 監視相依性](../application-insights/app-insights-asp-net-dependencies.md) 以查看 REST、SQL 或其他外部資源是否會降低您的效能。
* [使用 Application Insights .NET SDK 追蹤自訂作業](../application-insights/application-insights-custom-operations-tracking.md)
