---
title: 監視 Azure IoT 中樞的健康情況 | Microsoft Docs
description: 使用「Azure 監視器」和「Azure 資源健康狀態」來監視您的「IoT 中樞」並快速診斷問題
author: kgremban
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/09/2018
ms.author: kgremban
ms.openlocfilehash: d3c32c2258f7542a02549fbc531aa9e8293d0235
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996293"
---
# <a name="monitor-the-health-of-azure-iot-hub-and-diagnose-problems-quickly"></a>監視 Azure IoT 中樞的健康情況並快速診斷問題

實作「Azure IoT 中樞」的企業會期望其資源有可靠的效能。 為了協助您對作業維持密切的監看，「IoT 中樞」已與 [Azure 監視器][lnk-AM]和 [Azure 資源健康狀態][lnk-ARH]完全整合。 這兩項服務會一前一後協力運作，為您提供讓 IoT 解決方案以健全狀態啟動並執行所需的資料。 

「Azure 監視器」是您所有 Azure 服務之監視和記錄功能的單一來源。 您可以將 Azure 監視器產生的診斷記錄傳送給 Log Analytics、事件中樞或 Azure 儲存體來進行自訂處理。 Azure 監視器的計量和診斷設定可讓您檢視資源的效能。 請繼續閱讀本文來了解如何搭配您的 IoT 中樞來[使用 Azure 監視器](#use-azure-monitor)。 

> [!IMPORTANT]
> 不保證使用 Azure 監視器診斷記錄透過 IoT 中樞服務發出的事件是可靠的或按順序的。 某些事件可能會遺失，或未按順序傳遞。 診斷記錄也不是即時的，而且它可能需要幾分鐘的時間才能將事件記錄到您所選擇的目的地。

「Azure 資源健康狀態」可協助您在 Azure 問題影響您的資源時，進行診斷並取得支援。 個人化儀表板可提供您「IoT 中樞」的目前和過去健全狀態。 請繼續閱讀本文來了解如何搭配您的 IoT 中樞來[使用 Azure 資源健康狀態](#use-azure-resource-health)。 

除了與這兩項服務整合之外，「IoT 中樞」也提供自己的計量，可供您用來了解 IoT 資源的狀態。 若要深入了解，請參閱[了解 IoT 中樞計量][lnk-metrics]。

## <a name="use-azure-monitor"></a>使用 Azure 監視器

「Azure 監視器」提供資源層級的診斷資訊，這意謂著您可以監視在 IoT 中樞內進行的作業。 

「Azure 監視器」的診斷設定會取代「IoT 中樞」作業監視器。 如果您目前使用作業監視，則應該移轉您的工作流程。 如需詳細資訊，請參閱[從作業監視移轉至診斷設定][lnk-migrate]。

若要深入了解「Azure 監視器」所監視的特定計量和事件，請參閱 [Azure 監視器支援的計量][lnk-AM-metrics]和[Azure 診斷記錄支援的服務、結構描述和類別][lnk-AM-schemas]。

[!INCLUDE [iot-hub-diagnostics-settings](../../includes/iot-hub-diagnostics-settings.md)]

### <a name="understand-the-logs"></a>了解記錄

「Azure 監視器」會追蹤「IoT 中樞」中進行的各種不同作業。 每個類別都有一個結構描述，當中定義該類別中事件的回報方式。 

#### <a name="connections"></a>連線

連線類別會追蹤來自 IoT 中樞的裝置連線和中斷連線事件以及錯誤。 若想識別未經授權的連線嘗試，以及在連線品質不佳之區域內的裝置中斷連線時進行追蹤，就很適合追蹤此類別。

> [!NOTE]
> 針對可靠的裝置連線狀態，勾選 [裝置活動訊號][lnk-devguide-heartbeat]。

```json
{
    "time": "UTC timestamp",
    "resourceId": "Resource Id",
    "operationName": "deviceConnect",
    "category": "Connections",
    "level": "Information",
    "properties": "{\"deviceId\":\"<deviceId>\",\"protocol\":\"<protocol>\",\"authType\":\"{\\\"scope\\\":\\\"device\\\",\\\"type\\\":\\\"sas\\\",\\\"issuer\\\":\\\"iothub\\\",\\\"acceptingIpFilterRule\\\":null}\",\"maskedIpAddress\":\"<maskedIpAddress>\"}", 
    "location": "Resource location"
}
```

#### <a name="cloud-to-device-commands"></a>雲端到裝置的命令

雲端到裝置的命令類別會追蹤在 IoT 中樞發生，且與雲端到裝置訊息管線相關的錯誤。 這個類別包括在傳送雲端到裝置訊息 (例如未經授權的寄件者)、接收雲端到裝置訊息 (例如超過傳遞計數) 和接收雲端到裝置訊息意見反應 (例如意見反應已過期) 時所發生的錯誤。 如果已成功傳遞雲端到裝置訊息，但裝置未適當處理雲端到裝置訊息，此類別不會捕捉來自此裝置的錯誤。

```json
{
    "time": " UTC timestamp",
    "resourceId": "Resource Id",
    "operationName": "messageExpired",
    "category": "C2DCommands",
    "level": "Error",
    "resultType": "Event status",
    "resultDescription": "MessageDescription",
    "properties": "{\"deviceId\":\"<deviceId>\",\"messageId\":\"<messageId>\",\"messageSizeInBytes\":\"<messageSize>\",\"protocol\":\"Amqp\",\"deliveryAcknowledgement\":\"<None, NegativeOnly, PositiveOnly, Full>\",\"deliveryCount\":\"0\",\"expiryTime\":\"<timestamp>\",\"timeInSystem\":\"<timeInSystem>\",\"ttl\":<ttl>, \"EventProcessedUtcTime\":\"<UTC timestamp>\",\"EventEnqueuedUtcTime\":\"<UTC timestamp>\", \"maskedIpAddresss\": \"<maskedIpAddress>\", \"statusCode\": \"4XX\"}",
    "location": "Resource location"
}
```

#### <a name="device-identity-operations"></a>裝置身分識別作業

裝置身分識別作業類別會追蹤您嘗試在其 IoT 中樞的身分識別登錄中建立、更新或刪除項目時所發生的錯誤。 佈建案例就很適合追蹤此類別。

```json
{
    "time": "UTC timestamp",
    "resourceId": "Resource Id",
    "operationName": "get",
    "category": "DeviceIdentityOperations",
    "level": "Error",    
    "resultType": "Event status",
    "resultDescription": "MessageDescription",
    "properties": "{\"maskedIpAddress\":\"<maskedIpAddress>\",\"deviceId\":\"<deviceId>\", \"statusCode\":\"4XX\"}",
    "location": "Resource location"
}
```

#### <a name="routes"></a>路由

訊息路由類別會在訊息路由評估期間追蹤發生的錯誤以及 IoT 中樞所認知的端點健全狀況。 此類別包含的事件像是：當規則評估為「未定義」、當 IoT 中樞將端點標示為無效、及其他任何從端點收到錯誤。 此類別不包含有關訊息本身的特定錯誤 (例如裝置節流錯誤)，這些是在「裝置遙測」類別下報告。

```json
{
    "time": "UTC timestamp",
    "resourceId": "Resource Id",
    "operationName": "endpointUnhealthy",
    "category": "Routes",
    "level": "Error",
    "properties": "{\"deviceId\": \"<deviceId>\",\"endpointName\":\"<endpointName>\",\"messageId\":<messageId>,\"details\":\"<errorDetails>\",\"routeName\": \"<routeName>\"}",
    "location": "Resource location"
}
```

#### <a name="device-telemetry"></a>裝置遙測

裝置遙測類別會追蹤在 IoT 中樞發生，且與遙測管線相關的錯誤。 此類別包括在傳送遙測事件 (例如節流) 和接收遙測事件 (例如未經授權的讀取器) 時所發生的錯誤。 這個類別無法捕捉裝置本身所執行之程式碼所造成的錯誤。

```json
{
    "time": "UTC timestamp",
    "resourceId": "Resource Id",
    "operationName": "ingress",
    "category": "DeviceTelemetry",
    "level": "Error",
    "resultType": "Event status",
    "resultDescription": "MessageDescription",
    "properties": "{\"deviceId\":\"<deviceId>\",\"batching\":\"0\",\"messageSizeInBytes\":\"<messageSizeInBytes>\",\"EventProcessedUtcTime\":\"<UTC timestamp>\",\"EventEnqueuedUtcTime\":\"<UTC timestamp>\",\"partitionId\":\"1\"}", 
    "location": "Resource location"
}
```

#### <a name="file-upload-operations"></a>檔案上傳作業

檔案上傳類別會追蹤在 IoT 中樞發生且與檔案上傳功能相關的錯誤。 此類別包括︰

* SAS URI 所發生的錯誤，例如當它在裝置通知中樞已完成上傳之前就到期時。
* 裝置所報告的失敗上傳。
* IoT 中樞通知訊息建立期間在儲存體中找不到檔案時所發生的錯誤。

此類別無法捕捉直接發生在裝置將檔案上傳到儲存體時的錯誤。

```json
{
    "time": "UTC timestamp",
    "resourceId": "Resource Id",
    "operationName": "ingress",
    "category": "FileUploadOperations",
    "level": "Error",
    "resultType": "Event status",
    "resultDescription": "MessageDescription",
    "durationMs": "1",
    "properties": "{\"deviceId\":\"<deviceId>\",\"protocol\":\"<protocol>\",\"authType\":\"{\\\"scope\\\":\\\"device\\\",\\\"type\\\":\\\"sas\\\",\\\"issuer\\\":\\\"iothub\\\",\\\"acceptingIpFilterRule\\\":null}\",\"blobUri\":\"http//bloburi.com\"}",
    "location": "Resource location"
}
```

#### <a name="cloud-to-device-twin-operations"></a>雲端到裝置對應項作業

雲端到裝置對應項作業類別會追蹤裝置對應項上服務起始的事件。 這些作業可能包括取得對應項、更新或取代標記，以及更新或取代所需的屬性。 

```json
{
    "time": "UTC timestamp",
    "resourceId": "Resource Id",
    "operationName": "read",
    "category": "C2DTwinOperations",
    "level": "Information",
    "durationMs": "1",
    "properties": "{\"deviceId\":\"<deviceId>\",\"sdkVersion\":\"<sdkVersion>\",\"messageSize\":\"<messageSize>\"}", 
    "location": "Resource location"
}
```

#### <a name="device-to-cloud-twin-operations"></a>裝置到雲端對應項作業

裝置到雲端對應項作業類別會追蹤裝置對應項上裝置起始的事件。 這些作業可能包括取得對應項、更新回報的屬性，以及訂閱所需的屬性。

```json
{
    "time": "UTC timestamp",
    "resourceId": "Resource Id",
    "operationName": "update",
    "category": "D2CTwinOperations",
    "level": "Information",
    "durationMs": "1",
    "properties": "{\"deviceId\":\"<deviceId>\",\"protocol\":\"<protocol>\",\"authenticationType\":\"{\\\"scope\\\":\\\"device\\\",\\\"type\\\":\\\"sas\\\",\\\"issuer\\\":\\\"iothub\\\",\\\"acceptingIpFilterRule\\\":null}\"}", 
    "location": "Resource location"
}
```

#### <a name="twin-queries"></a>對應項查詢

對應項查詢類別會針對在雲端中起始之裝置對應項的查詢要求，進行回報。 

```json
{
    "time": "UTC timestamp",
    "resourceId": "Resource Id",
    "operationName": "query",
    "category": "TwinQueries",
    "level": "Information",
    "durationMs": "1",
    "properties": "{\"query\":\"<twin query>\",\"sdkVersion\":\"<sdkVersion>\",\"messageSize\":\"<messageSize>\",\"pageSize\":\"<pageSize>\", \"continuation\":\"<true, false>\", \"resultSize\":\"<resultSize>\"}", 
    "location": "Resource location"
}
```

#### <a name="jobs-operations"></a>作業的操作

作業操作類別會針對用來更新裝置對應項或對多部裝置叫用直接方法的作業要求，進行回報。 這些要求是在雲端中起始的。 

```json
{
    "time": "UTC timestamp",
    "resourceId": "Resource Id",
    "operationName": "jobCompleted",
    "category": "JobsOperations",
    "level": "Information",
    "durationMs": "1",
    "properties": "{\"jobId\":\"<jobId>\", \"sdkVersion\": \"<sdkVersion>\",\"messageSize\": <messageSize>,\"filter\":\"DeviceId IN ['1414ded9-b445-414d-89b9-e48e8c6285d5']\",\"startTimeUtc\":\"Wednesday, September 13, 2017\",\"duration\":\"0\"}", 
    "location": "Resource location"
}
```

#### <a name="direct-methods"></a>直接方法

直接方法類別會追蹤傳送給個別裝置的「要求-回應」互動。 這些要求是在雲端中起始的。 

```json
{
    "time": "UTC timestamp",
    "resourceId": "Resource Id",
    "operationName": "send",
    "category": "DirectMethods",
    "level": "Information",
    "durationMs": "1",
    "properties": "{\"deviceId\":\"<deviceId>\", \"RequestSize\": 1, \"ResponseSize\": 1, \"sdkVersion\": \"2017-07-11\"}", 
    "location": "Resource location"
}
```

### <a name="read-logs-from-azure-event-hubs"></a>從 Azure 事件中樞讀取記錄

在透過診斷設定來設定事件記錄功能之後，您可以建立會將記錄讀出的應用程式，以便根據記錄中的資訊來採取行動。 以下範例程式碼會從事件中樞擷取記錄：

```csharp
class Program 
{ 
    static string connectionString = "{your AMS eventhub endpoint connection string}"; 
    static string monitoringEndpointName = "{your AMS event hub endpoint name}"; 
    static EventHubClient eventHubClient; 
//This is the Diagnostic Settings schema 
    class AzureMonitorDiagnosticLog 
    { 
        string time { get; set; } 
        string resourceId { get; set; } 
        string operationName { get; set; } 
        string category { get; set; } 
        string level { get; set; } 
        string resultType { get; set; } 
        string resultDescription { get; set; } 
        string durationMs { get; set; } 
        string callerIpAddress { get; set; } 
        string correlationId { get; set; } 
        string identity { get; set; } 
        string location { get; set; } 
        Dictionary<string, string> properties { get; set; } 
    }; 
    static void Main(string[] args) 
    { 
        Console.WriteLine("Monitoring. Press Enter key to exit.\n"); 
        eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, monitoringEndpointName); 
        var d2cPartitions = eventHubClient.GetRuntimeInformation().PartitionIds; 
        CancellationTokenSource cts = new CancellationTokenSource(); 
        var tasks = new List<Task>(); 
        foreach (string partition in d2cPartitions) 
        { 
            tasks.Add(ReceiveMessagesFromDeviceAsync(partition, cts.Token)); 
        } 
        Console.ReadLine(); 
        Console.WriteLine("Exiting..."); 
        cts.Cancel(); 
        Task.WaitAll(tasks.ToArray()); 
    } 
    private static async Task ReceiveMessagesFromDeviceAsync(string partition, CancellationToken ct) 
    { 
        var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition, DateTime.UtcNow); 
        while (true) 
        { 
            if (ct.IsCancellationRequested) 
            { 
                await eventHubReceiver.CloseAsync(); 
                break; 
            } 
            EventData eventData = await eventHubReceiver.ReceiveAsync(new TimeSpan(0,0,10)); 
            if (eventData != null) 
            { 
                string data = Encoding.UTF8.GetString(eventData.GetBytes()); 
                Console.WriteLine("Message received. Partition: {0} Data: '{1}'", partition, data); 
                var deserializer = new JavaScriptSerializer(); 
                //deserialize json data to azure monitor object 
                AzureMonitorDiagnosticLog message = new JavaScriptSerializer().Deserialize<AzureMonitorDiagnosticLog>(result); 
 
            } 
        } 
    } 
} 
```

## <a name="use-azure-resource-health"></a>使用 Azure 資源健康狀態

使用「Azure 資源健康狀態」來監視您的 IoT 中樞是否已啟動並執行。 您也可以了解區域性服務中斷是否對您的 IoT 中樞造成影響。 若要了解有關您「Azure IoT 中樞」健全狀態的特定詳細資料，建議您[使用 Azure 監視器](#use-azure-monitor)。 

「Azure IoT 中樞」會指出區域層級的健康情況。 如果有區域性服務中斷影響到您的 IoT 中樞，健全狀態就會顯示為 [不明]。 若要深入了解「Azure 資源健康狀態」所執行的特定健康情況檢查，請參閱 [Azure 資源健康狀態中的資源類型和健康情況檢查][lnk-ARH-checks]。

若要檢查您 IoT 中樞的健康情況，請依照下列步驟：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 瀏覽至 [服務健全狀況] > [資源健康狀態]。
1. 從下拉式方塊中，選取您的訂用帳戶和 [IoT 中樞]。

若要深入了解如何解譯健康情況資料，請參閱[Azure 資源健康狀態概觀][lnk-ARH]

## <a name="next-steps"></a>後續步驟

- [了解 IoT 中樞計量][lnk-metrics]
- [搭配連接 IoT 中樞和信箱的 Azure Logic Apps 進行 IoT 遠端監視和通知][lnk-monitoring-notifications]


[lnk-AM]: ../azure-monitor/index.yml
[lnk-ARH]: ../service-health/resource-health-overview.md
[lnk-metrics]: iot-hub-metrics.md
[lnk-migrate]: iot-hub-migrate-to-diagnostics-settings.md
[lnk-AM-metrics]: ../monitoring-and-diagnostics/monitoring-supported-metrics.md
[lnk-AM-schemas]: ../monitoring-and-diagnostics/monitoring-diagnostic-logs-schema.md
[lnk-ARH-checks]: ../service-health/resource-health-checks-resource-types.md
[lnk-monitoring-notifications]: iot-hub-monitoring-notifications-with-azure-logic-apps.md
[lnk-devguide-heartbeat]: iot-hub-devguide-identity-registry.md#device-heartbeat
