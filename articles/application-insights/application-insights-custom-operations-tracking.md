---
title: 使用 Azure Application Insights .NET SDK 追蹤自訂作業 | Microsoft Docs
description: 使用 Azure Application Insights .NET SDK 追蹤自訂作業
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: TBD
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: article
ms.date: 06/30/2017
ms.author: sergkanz;mbullwin
ms.openlocfilehash: e66d5e8674d6564d3910daddb1119c22a8b8bd9f
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2018
---
# <a name="track-custom-operations-with-application-insights-net-sdk"></a>使用 Application Insights .NET SDK 追蹤自訂作業

Azure Application Insights SDK 會自動追蹤相依服務的連入 HTTP 要求和呼叫，例如 HTTP 要求、SQL 查詢。 追蹤和相互關聯要求與相依性，可讓您深入了解橫跨所有微服務 (合併此應用程式) 的整體應用程式回應能力和可靠性。 

有一類應用程式模式無法以一般方式支援。 適當監視這類模式時，需要進行手動程式碼檢測。 本文涵蓋可能需要手動檢測的一些模式，例如自訂佇列處理和執行長時間執行背景工作。

本文提供有關如何使用 ApplicationInsights SDK 追蹤自訂作業的指引。 本文件相關於：

- 適用於 .NET (也稱為 Base SDK) 的 Application Insights 版本 2.4+。
- 適用於 Web 應用程式 (執行 ASP.NET) 的 Application Insights 版本 2.4+。
- Application Insights for ASP.NET Core 版本 2.1+。

## <a name="overview"></a>概觀
作業是應用程式執行的邏輯部分。 它具有名稱、開始時間、持續時間、結果和執行的內容，例如使用者名稱、屬性和結果。 如果作業 A 是由作業 B 起始，則作業 B 設為 A 的父代。作業只能有一個父代，但是可以有多個子系作業。 如需有關作業和遙測相互關聯的詳細資訊，請參閱 [Azure Application Insights 遙測相互關聯](application-insights-correlation.md)。

在 Application Insights.NET SDK 中，作業是由抽象類別 [OperationTelemetry](https://github.com/Microsoft/ApplicationInsights-dotnet/blob/develop/src/Microsoft.ApplicationInsights/Extensibility/Implementation/OperationTelemetry.cs) 及其子系 [RequestTelemetry](https://github.com/Microsoft/ApplicationInsights-dotnet/blob/develop/src/Microsoft.ApplicationInsights/DataContracts/RequestTelemetry.cs) 和 [DependencyTelemetry](https://github.com/Microsoft/ApplicationInsights-dotnet/blob/develop/src/Microsoft.ApplicationInsights/DataContracts/DependencyTelemetry.cs) 描述。

## <a name="incoming-operations-tracking"></a>傳入作業追蹤 
Application Insights Wb SDK 會針對在 IIS 管線中執行的 ASP.NET 應用程式和所有的 ASP.NET Core 應用程式，自動收集 HTTP 要求。 其他平台和架構有社群支援的解決方案。 不過，如果任何標準或社群支援的解決方案不支援應用程式，您可以用手動方式進行檢測。

接收佇列中項目的背景工作是另一個需要自訂追蹤的範例。 對於某些佇列，將訊息新增至此佇列的呼叫會作為相依性追蹤。 但是，不會自動收集描述處理訊息的高階作業。

我們來看看追蹤這類作業的方式。

在較高的層級中，工作是建立 `RequestTelemetry` 並且設定已知的屬性。 作業完成之後，您會追蹤遙測。 下列範例示範此工作。

### <a name="http-request-in-owin-self-hosted-app"></a>Owin 自我裝載應用程式中的 HTTP 要求
在此範例中，追蹤內容會根據[相互關聯的 HTTP 通訊協定](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md)傳播。 您應該預期會收到該處所述的標題。

```csharp
public class ApplicationInsightsMiddleware : OwinMiddleware
{
    private readonly TelemetryClient telemetryClient = new TelemetryClient(TelemetryConfiguration.Active);
    
    public ApplicationInsightsMiddleware(OwinMiddleware next) : base(next) {}

    public override async Task Invoke(IOwinContext context)
    {
        // Let's create and start RequestTelemetry.
        var requestTelemetry = new RequestTelemetry
        {
            Name = $"{context.Request.Method} {context.Request.Uri.GetLeftPart(UriPartial.Path)}"
        };

        // If there is a Request-Id received from the upstream service, set the telemetry context accordingly.
        if (context.Request.Headers.ContainsKey("Request-Id"))
        {
            var requestId = context.Request.Headers.Get("Request-Id");
            // Get the operation ID from the Request-Id (if you follow the HTTP Protocol for Correlation).
            requestTelemetry.Context.Operation.Id = GetOperationId(requestId);
            requestTelemetry.Context.Operation.ParentId = requestId;
        }

        // StartOperation is a helper method that allows correlation of 
        // current operations with nested operations/telemetry
        // and initializes start time and duration on telemetry items.
        var operation = telemetryClient.StartOperation(requestTelemetry);

        // Process the request.
        try
        {
            await Next.Invoke(context);
        }
        catch (Exception e)
        {
            requestTelemetry.Success = false;
            telemetryClient.TrackException(e);
            throw;
        }
        finally
        {
            // Update status code and success as appropriate.
            if (context.Response != null)
            {
                requestTelemetry.ResponseCode = context.Response.StatusCode.ToString();
                requestTelemetry.Success = context.Response.StatusCode >= 200 && context.Response.StatusCode <= 299;
            }
            else
            {
                requestTelemetry.Success = false;
            }

            // Now it's time to stop the operation (and track telemetry).
            telemetryClient.StopOperation(operation);
        }
    }
    
    public static string GetOperationId(string id)
    {
        // Returns the root ID from the '|' to the first '.' if any.
        int rootEnd = id.IndexOf('.');
        if (rootEnd < 0)
            rootEnd = id.Length;

        int rootStart = id[0] == '|' ? 1 : 0;
        return id.Substring(rootStart, rootEnd - rootStart);
    }
}
```

相互關聯的 HTTP 通訊協定也會宣告 `Correlation-Context` 標題。 不過，為了簡單起見在這裡省略。

## <a name="queue-instrumentation"></a>佇列檢測
雖然有[相互關聯的 HTTP 通訊協定](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md)可隨著 HTTP 要求傳遞相互關聯詳細資訊，但每個佇列通訊協定都必須定義相同的詳細資訊如何連同佇列訊息傳遞。 有些佇列通訊協定 (例如 AMQP) 允許傳遞其他中繼資料，其他 (例如 Azure 儲存體佇列) 則需要將內容編碼為 	訊息承載。

### <a name="service-bus-queue"></a>服務匯流排佇列
Application Insights 會使用[適用於 .NET 的 Microsoft Azure 服務匯流排用戶端](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus/) 3.0.0 版本和更新版本來追蹤服務匯流排通訊呼叫。
如果您使用[訊息處理常式模式](/dotnet/api/microsoft.azure.servicebus.queueclient.registermessagehandler)來處理訊息，便無需採取任何動作，系統會自動追蹤由您的服務所完成的所有服務匯流排呼叫，並將它們與其他遙測項目相互關聯。 如果您手動處理訊息，請參閱[使用 Microsoft Application Insights 追蹤服務匯流排用戶端](../service-bus-messaging/service-bus-end-to-end-tracing.md)。

如果您使用 [WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) 套件，請進一步閱讀 - 下列範例示範如何追蹤 (和相互關聯) 服務匯流排的呼叫，因為服務匯流排佇列使用 AMQP 通訊協定，而 Application Insights 不會自動追蹤佇列作業。
相互關聯識別碼會傳入訊息屬性。

#### <a name="enqueue"></a>加入佇列

```csharp
public async Task Enqueue(string payload)
{
    // StartOperation is a helper method that initializes the telemetry item
    // and allows correlation of this operation with its parent and children.
    var operation = telemetryClient.StartOperation<DependencyTelemetry>("enqueue " + queueName);
    operation.Telemetry.Type = "Queue";
    operation.Telemetry.Data = "Enqueue " + queueName;

    var message = new BrokeredMessage(payload);
    // Service Bus queue allows the property bag to pass along with the message.
    // We will use them to pass our correlation identifiers (and other context)
    // to the consumer.
    message.Properties.Add("ParentId", operation.Telemetry.Id);
    message.Properties.Add("RootId", operation.Telemetry.Context.Operation.Id);

    try
    {
        await queue.SendAsync(message);
        
        // Set operation.Telemetry Success and ResponseCode here.
        operation.Telemetry.Success = true;
    }
    catch (Exception e)
    {
        telemetryClient.TrackException(e);
        // Set operation.Telemetry Success and ResponseCode here.
        operation.Telemetry.Success = false;
        throw;
    }
    finally
    {
        telemetryClient.StopOperation(operation);
    }
}
```

#### <a name="process"></a>Process
```csharp
public async Task Process(BrokeredMessage message)
{
    // After the message is taken from the queue, create RequestTelemetry to track its processing.
    // It might also make sense to get the name from the message.
    RequestTelemetry requestTelemetry = new RequestTelemetry { Name = "Dequeue " + queueName };

    var rootId = message.Properties["RootId"].ToString();
    var parentId = message.Properties["ParentId"].ToString();
    // Get the operation ID from the Request-Id (if you follow the HTTP Protocol for Correlation).
    requestTelemetry.Context.Operation.Id = rootId;
    requestTelemetry.Context.Operation.ParentId = parentId;

    var operation = telemetryClient.StartOperation(requestTelemetry);

    try
    {
        await ProcessMessage();
    }
    catch (Exception e)
    {
        telemetryClient.TrackException(e);
        throw;
    }
    finally
    {
        // Update status code and success as appropriate.
        telemetryClient.StopOperation(operation);
    }
}
```

### <a name="azure-storage-queue"></a>Azure 儲存體佇列
下列範例示範如何追蹤 [Azure 儲存體佇列](../storage/queues/storage-dotnet-how-to-use-queues.md)作業，並且讓產生者、取用者與 Azure 儲存體之間的遙測相互關聯。 

儲存體佇列有 HTTP API。 HTTP 要求的 Application Insights 相依性收集器會追蹤對佇列的所有呼叫。
請確保您在 `applicationInsights.config` 中具有 `Microsoft.ApplicationInsights.DependencyCollector.HttpDependenciesParsingTelemetryInitializer`。 如果沒有，請如同[在 Azure Application Insights SDK 中篩選和前置處理](app-insights-api-filtering-sampling.md)所述，以程式設計方式新增它。

如果您手動設定 Application Insights，請務必建立並初始化 `Microsoft.ApplicationInsights.DependencyCollector.DependencyTrackingTelemetryModule`，類似於：
 
```csharp
DependencyTrackingTelemetryModule module = new DependencyTrackingTelemetryModule();

// You can prevent correlation header injection to some domains by adding it to the excluded list.
// Make sure you add a Storage endpoint. Otherwise, you might experience request signature validation issues on the Storage service side.
module.ExcludeComponentCorrelationHttpHeadersOnDomains.Add("core.windows.net");
module.Initialize(TelemetryConfiguration.Active);

// Do not forget to dispose of the module during application shutdown.
```

您可能也想要相互關聯 Application Insights 作業識別碼與儲存體要求識別碼。 如需如何設定及取得儲存體要求用戶端和伺服器要求識別碼的詳細資訊，請參閱[監視、診斷 Azure 儲存體及進行移難排解](../storage/common/storage-monitoring-diagnosing-troubleshooting.md#end-to-end-tracing)。

#### <a name="enqueue"></a>加入佇列
因為 Azure 儲存體佇列支援 HTTP API，Application Insights 會自動追蹤所有作業與佇列。 在許多情況下，此檢測應該就足夠了。 不過，若要讓取用者端追蹤與生產者追蹤相互關聯，您必須傳遞一些相互關聯內容，類似於我們在「相互關聯的 HTTP 通訊協定」中的作業方式。 

這個範例顯示如何追蹤 `Enqueue` 作業。 您可以：

 - **相互關聯重試 (如果有的話)**：全部都有一個通用父代，也就是 `Enqueue` 作業。 否則會作為連入要求的子系追蹤。 如果佇列有多個邏輯要求，可能難以找到導致重試的呼叫。
 - **相互關聯儲存體記錄 (必要時)**：與 Application Insights 遙測相互關聯。

`Enqueue` 作業是父代作業 (例如，連入 HTTP 要求) 的子系。 HTTP 相依性呼叫是 `Enqueue` 作業的子系，是連入要求的孫系：

```csharp
public async Task Enqueue(CloudQueue queue, string message)
{
    var operation = telemetryClient.StartOperation<DependencyTelemetry>("enqueue " + queue.Name);
    operation.Telemetry.Type = "Queue";
    operation.Telemetry.Data = "Enqueue " + queue.Name;

    // MessagePayload represents your custom message and also serializes correlation identifiers into payload.
    // For example, if you choose to pass payload serialized to JSON, it might look like
    // {'RootId' : 'some-id', 'ParentId' : '|some-id.1.2.3.', 'message' : 'your message to process'}
    var jsonPayload = JsonConvert.SerializeObject(new MessagePayload
    {
        RootId = operation.Telemetry.Context.Operation.Id,
        ParentId = operation.Telemetry.Id,
        Payload = message
    });
    
    CloudQueueMessage queueMessage = new CloudQueueMessage(jsonPayload);

    // Add operation.Telemetry.Id to the OperationContext to correlate Storage logs and Application Insights telemetry.
    OperationContext context = new OperationContext { ClientRequestID = operation.Telemetry.Id};

    try
    {
        await queue.AddMessageAsync(queueMessage, null, null, new QueueRequestOptions(), context);
    }
    catch (StorageException e)
    {
        operation.Telemetry.Properties.Add("AzureServiceRequestID", e.RequestInformation.ServiceRequestID);
        operation.Telemetry.Success = false;
        operation.Telemetry.ResultCode = e.RequestInformation.HttpStatusCode.ToString();
        telemetryClient.TrackException(e);
    }
    finally
    {
        // Update status code and success as appropriate.
        telemetryClient.StopOperation(operation);
    }
}  
```

如果您因為其他原因，而想要減少您應用程式回報的遙測資料量，或不想追蹤 `Enqueue` 作業，您可以直接使用 `Activity` API：

- 建立 (和啟動) 新的 `Activity`，而不是啟動 Application Insights 作業。 您不需要在上面指派作業名稱以外的任何屬性。
- 將 `yourActivity.Id` 序列化成為訊息承載，而不是 `operation.Telemetry.Id`。 您也可以使用 `Activity.Current.Id`。


#### <a name="dequeue"></a>清除佇列
類似於 `Enqueue`，Application Insights 會自動追蹤儲存體佇列的實際 HTTP 要求。 不過 `Enqueue` 作業可能發生於父代內容，例如連入要求內容。 Application Insights SDK 會使這類作業 (及其 HTTP 部分) 與父代要求和相同範圍中報告的其他遙測自動相互關聯。

`Dequeue` 作業有些麻煩。 Application Insights SDK 會自動追蹤 HTTP 要求。 不過，它在剖析訊息之前並不知道相互關聯內容。 不可能相互關聯 HTTP 要求來取得包含遙測其餘部分的訊息。

在許多情況下，將佇列的 HTTP 要求與其他追蹤相互關聯也很有用。 下列範例提供如何執行的示範：

```csharp
public async Task<MessagePayload> Dequeue(CloudQueue queue)
{
    var telemetry = new DependencyTelemetry
    {
        Type = "Queue",
        Name = "Dequeue " + queue.Name
    };

    telemetry.Start();

    try
    {
        var message = await queue.GetMessageAsync();

        if (message != null)
        {
            var payload = JsonConvert.DeserializeObject<MessagePayload>(message.AsString);

            // If there is a message, we want to correlate the Dequeue operation with processing.
            // However, we will only know what correlation ID to use after we get it from the message,
            // so we will report telemetry after we know the IDs.
            telemetry.Context.Operation.Id = payload.RootId;
            telemetry.Context.Operation.ParentId = payload.ParentId;

            // Delete the message.
            return payload;
        }
    }
    catch (StorageException e)
    {
        telemetry.Properties.Add("AzureServiceRequestID", e.RequestInformation.ServiceRequestID);
        telemetry.Success = false;
        telemetry.ResultCode = e.RequestInformation.HttpStatusCode.ToString();
        telemetryClient.TrackException(e);
    }
    finally
    {
        // Update status code and success as appropriate.
        telemetry.Stop();
        telemetryClient.Track(telemetry);
    }

    return null;
}
```

#### <a name="process"></a>Process

在下列範例中，追蹤連入訊息的方式類似於追蹤連入 HTTP 要求的方式：

```csharp
public async Task Process(MessagePayload message)
{
    // After the message is dequeued from the queue, create RequestTelemetry to track its processing.
    RequestTelemetry requestTelemetry = new RequestTelemetry { Name = "Dequeue " + queueName };
    // It might also make sense to get the name from the message.
    requestTelemetry.Context.Operation.Id = message.RootId;
    requestTelemetry.Context.Operation.ParentId = message.ParentId;

    var operation = telemetryClient.StartOperation(requestTelemetry);

    try
    {
        await ProcessMessage();
    }
    catch (Exception e)
    {
        telemetryClient.TrackException(e);
        throw;
    }
    finally
    {
        // Update status code and success as appropriate.
        telemetryClient.StopOperation(operation);
    }
}
```

同樣地，可能會檢測其他佇列作業。 預覽 (Peek) 作業應以類似清除佇列作業的方式進行檢測。 不一定要檢測佇列管理作業。 Application Insights 會追蹤 HTTP 這類作業，這在大多數情況下已足夠。

檢測訊息刪除時，請務必設定作業 (相互關聯) 識別碼。 或者，您可使用 `Activity` API。 您就不需要在遙測項目上設定作業識別碼，因為 Application Insights SDK 會為您設定：

- 取得佇列中的項目後，建立新的 `Activity`。
- 使用 `Activity.SetParentId(message.ParentId)` 讓取用者和產生者記錄相互關聯。
- 啟動 `Activity`。
- 使用 `Start/StopOperation` 協助程式追蹤清除佇列、處理和刪除作業。 從相同的非同步控制流程 (執行內容) 執行此作業。 如此一來，這些作業就會正確地相互關聯。
- 停止 `Activity`。
- 使用 `Start/StopOperation` 或手動呼叫 `Track` 遙測。

### <a name="batch-processing"></a>批次處理
有些佇列中，您可以使用一個要求清除佇列多個訊息。 處理這類訊息可能是獨立的，並且屬於不同的邏輯作業。 在此情況下，不可能使 `Dequeue` 作業與特定訊息處理相互關聯。

每個訊息應該在自己的非同步控制流程中處理。 如需詳細資訊，請參閱[連出相依性追蹤](#outgoing-dependencies-tracking)一節。

## <a name="long-running-background-tasks"></a>長時間執行的背景工作
有些應用程式可能會應使用者要求，啟動長時間執行的作業。 就追蹤/檢測觀點而言，這與要求或相依性檢測並無不同： 

```csharp
async Task BackgroundTask()
{
    var operation = telemetryClient.StartOperation<RequestTelemetry>(taskName);
    operation.Telemetry.Type = "Background";
    try
    {
        int progress = 0;
        while (progress < 100)
        {
            // Process the task.
            telemetryClient.TrackTrace($"done {progress++}%");
        }
        // Update status code and success as appropriate.
    }
    catch (Exception e)
    {
        telemetryClient.TrackException(e);
        // Update status code and success as appropriate.
        throw;
    }
    finally
    {
        telemetryClient.StopOperation(operation);
    }
}
```

在此範例中，`telemetryClient.StartOperation` 會建立 `RequestTelemetry` 並填滿相互關聯內容。 假設您有一項父代作業，由排程作業的連入要求所建立。 只要 `BackgroundTask` 在與連入要求相同的非同步控制流程中啟動，它就會與該父代作業相互關聯。 `BackgroundTask` 和所有巢狀遙測項目將會自動與造成它的要求相互關聯，即使在要求結束後亦然。

從沒有任何相關聯作業 (`Activity`) 的背景執行緒啟動工作時，`BackgroundTask` 沒有任何父代。 不過，它可以有巢狀作業。 工作回報的所有遙測項目會與在 `BackgroundTask` 中建立的 `RequestTelemetry` 相互關聯。

## <a name="outgoing-dependencies-tracking"></a>連出相依性追蹤
您可以追蹤自己的相依性種類或 Application Insights 不支援的作業。

服務匯流排佇列或儲存體佇列中的 `Enqueue` 方法可作為這類自訂追蹤的範例。

自訂相依性追蹤的一般方法如下：

- 呼叫 `TelemetryClient.StartOperation` (擴充) 方法，以填滿相互關聯所需的 `DependencyTelemetry` 屬性和其他一些屬性 (開始時間戳記、持續時間)。
- 在 `DependencyTelemetry` 上設定其他自訂屬性，例如名稱與您需要的任何其他內容。
- 進行相依性呼叫並且等候。
- 完成時使用 `StopOperation` 停止作業。
- 處理例外狀況。

```csharp
public async Task RunMyTaskAsync()
{
    using (var operation = telemetryClient.StartOperation<DependencyTelemetry>("task 1"))
    {
        try 
        {
            var myTask = await StartMyTaskAsync();
            // Update status code and success as appropriate.
        }
        catch(...) 
        {
            // Update status code and success as appropriate.
        }
    }
}
```

處置作業會導致作業停止，因此您可以執行而不是呼叫 `StopOperation`。

*警告*：在某些情況下，未處理的例外狀況可能導致[無法](https://docs.microsoft.com/dotnet/csharp/language-reference/keywords/try-finally)呼叫 `finally`，因此可能無法追蹤作業。

### <a name="parallel-operations-processing-and-tracking"></a>平行的作業處理和追蹤

`StopOperation` 只會停止已啟動的作業。 如果目前執行中作業不符合您想要停止的作業，則 `StopOperation` 不會有任何動作。 如果您以平行方式在相同的執行內容中啟動多項作業，可能會發生這種情形：

```csharp
var firstOperation = telemetryClient.StartOperation<DependencyTelemetry>("task 1");
var firstOperation = telemetryClient.StartOperation<DependencyTelemetry>("task 1");
var firstTask = RunMyTaskAsync();

var secondOperation = telemetryClient.StartOperation<DependencyTelemetry>("task 2");
var secondTask = RunMyTaskAsync();

await firstTask;

// FAILURE!!! This will do nothing and will not report telemetry for the first operation
// as currently secondOperation is active.
telemetryClient.StopOperation(firstOperation); 

await secondTask;
```

請務必一律以相同的**非同步**方法來呼叫 `StartOperation` 和處理作業，以隔離平行執行的作業。 如果作業是同步的 (或不是非同步的)，請包裝處理序並使用 `Task.Run` 追蹤：

```csharp
public void RunMyTask(string name)
{
    using (var operation = telemetryClient.StartOperation<DependencyTelemetry>(name))
    {
        Process();
        // Update status code and success as appropriate.
    }
}

public async Task RunAllTasks()
{
    var task1 = Task.Run(() => RunMyTask("task 1"));
    var task2 = Task.Run(() => RunMyTask("task 2"));
    
    await Task.WhenAll(task1, task2);
}
```

## <a name="next-steps"></a>後續步驟

- 了解 Application Insights 中[遙測相互關聯](application-insights-correlation.md)的基本概念。
- 如需 Application Insights 類型和資料模型，請參閱[資料模型](application-insights-data-model.md)。
- 向 Application Insights 報告自訂[事件和計量](app-insights-api-custom-events-metrics.md)。
- 請查看內容屬性集合的標準[設定](app-insights-configuration-with-applicationinsights-config.md#telemetry-initializers-aspnet)。
- 查看 [System.Diagnostics.Activity 使用者指南](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md)以了解如何使遙測相互關聯。
