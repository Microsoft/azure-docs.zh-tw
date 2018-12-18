---
title: 使用 Azure API 管理、事件中樞和 Runscope 監視 API | Microsoft Docs
description: 藉由連接「Azure API 管理」、「Azure 事件中樞」和 Runscope 來進行 HTTP 記錄和監視，示範 log-to-eventhub 原則的範例應用程式
services: api-management
documentationcenter: ''
author: darrelmiller
manager: erikre
editor: ''
ms.assetid: c528cf6f-5f16-4a06-beea-fa1207541a47
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 01/23/2018
ms.author: apimpm
ms.openlocfilehash: 4c58be8f501e72027e1692ceb73552a3f252f92a
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38603173"
---
# <a name="monitor-your-apis-with-azure-api-management-event-hubs-and-runscope"></a>利用 Azure API 管理、事件中樞及 Runscope 監視您的 API
[API 管理服務](api-management-key-concepts.md) 提供許多功能，以增強傳送至 HTTP API 之 HTTP 要求的處理。 不過，要求和回應的存在都是暫時的。 提出要求並透過 API 管理服務送到您的後端 API。 您的 API 會處理此要求，而回應會傳回給 API 取用者。 API 管理服務會保留一些有關 API 的重要統計資料，以顯示在 Azure 入口網站儀表板上，但除此之外，詳細資料會消失。

藉由在「API 管理」服務中使用 log-to-eventhub 原則，您便可以將任何來自要求和回應的詳細資料傳送到 [Azure 事件中樞](../event-hubs/event-hubs-what-is-event-hubs.md)。 您想要從傳送到您的 API 的 HTTP 訊息產生事件的原因包羅萬象。 範例包括更新稽核線索、使用量分析、例外狀況警示和第三方整合。   

本文將示範如何擷取整個 HTTP 要求和回應訊息、將它傳送到事件中樞，然後將該訊息轉送給可提供 HTTP 記錄和監視服務的第三方服務。

## <a name="why-send-from-api-management-service"></a>為什麼要從 API 管理服務傳送？
您可以撰寫可插入 HTTP API 架構的 HTTP 中介軟體，以擷取 HTTP 要求和回應並將其饋送至記錄和監視系統。 此方法的缺點是 HTTP 中介軟體必須整合到後端 API 中，而且必須符合 API 的平台。 如果有多個 API，則每個 API 都必須部署中介軟體。 後端 API 無法升級通常有一些原因。

使用 Azure API 管理服務來與記錄基礎結構整合，提供了一個集中式且平台獨立的解決方案。 在某種程度上藉助 Azure API 管理的 [異地複寫](api-management-howto-deploy-multi-region.md) 功能，也可加以擴充。

## <a name="why-send-to-an-azure-event-hub"></a>為什麼要傳送到 Azure 事件中樞？
合理提問，為何建立 Azure 事件中樞專屬的原則？ 我可能想要在許多不同的地方記錄我的要求。 為什麼不能直接將要求傳送到最終目的地？  這是一個選項。 不過，從 API 管理服務提出記錄要求時，就必須考慮記錄訊息對 API 效能有何影響。 增加系統元件的可用執行個體或利用異地複寫功能，可以處理逐漸增加的負載。 不過，如果記錄基礎結構的要求開始變慢而低於負載，則流量短期突增可能會導致延遲要求。

Azure 事件中樞已設計用來輸入大量資料，其能夠處理的事件數目遠高於大部分 API 所處理的 HTTP 要求數目。 事件中樞可作為您的 API 管理服務與基礎結構之間有點複雜的緩衝區，將會儲存及處理訊息。 這可確保您的 API 效能不會因為記錄基礎結構而受到影響。  

資料一旦傳遞至事件中樞，就會保存起來並等待事件中樞取用者進行處理。 事件中樞不在乎其處理方式，而只在乎如何確保成功傳遞訊息。     

事件中樞能夠將事件串流至多個取用者群組。 如此一來，即可由不同的系統來處理事件。 這麼做能夠支援許多整合案例，而不會對在 API 管理服務中處理 API 要求造成額外延遲，因為只需要產生一個事件。

## <a name="a-policy-to-send-applicationhttp-messages"></a>用來傳送應用程式/http 訊息的原則
事件中樞接受簡單字串形式的事件資料。 該字串的內容由您決定。 若要能夠封裝 HTTP 要求並將它傳送至事件中樞，我們需要以要求或回應資訊來格式化字串。 在這類情況下，如果有我們可重複使用的現有格式，我們就不需要撰寫自己的剖析程式碼。 一開始，我考慮使用 [HAR](http://www.softwareishard.com/blog/har-12-spec/) 來傳送 HTTP 要求和回應。 不過，這種格式最適合用於儲存 JSON 格式的一連串 HTTP 要求。 其中包含了一些必要元素，讓透過網路傳遞 HTTP 訊息的案例增加了不必要的複雜度。  

替代選項是使用如 HTTP 規格 [RFC 7230](http://tools.ietf.org/html/rfc7230) 中所述的 `application/http` 媒體類型。 此媒體類型會使用與透過網路用來實際傳送 HTTP 訊息完全相同的格式，但整個訊息可以放在另一個 HTTP 要求的本文中。 在我們的案例中，我們只是會以此本文作為我們的訊息來傳送到事件中樞。 [Microsoft ASP.NET Web API 2.2 用戶端](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client/)程式庫中有一個剖析器，可以剖析此格式並將它轉換成原生 `HttpRequestMessage` 和 `HttpResponseMessage` 物件，相當方便。

為了能夠建立此訊息，我們需要在 Azure API 管理中利用以 C# 為基礎的[原則運算式](https://msdn.microsoft.com/library/azure/dn910913.aspx)。 以下是可將 HTTP 要求訊息傳送到 Azure 事件中樞的原則。

```xml
<log-to-eventhub logger-id="conferencelogger" partition-id="0">
@{
   var requestLine = string.Format("{0} {1} HTTP/1.1\r\n",
                                               context.Request.Method,
                                               context.Request.Url.Path + context.Request.Url.QueryString);

   var body = context.Request.Body?.As<string>(true);
   if (body != null && body.Length > 1024)
   {
       body = body.Substring(0, 1024);
   }

   var headers = context.Request.Headers
                          .Where(h => h.Key != "Authorization" && h.Key != "Ocp-Apim-Subscription-Key")
                          .Select(h => string.Format("{0}: {1}", h.Key, String.Join(", ", h.Value)))
                          .ToArray<string>();

   var headerString = (headers.Any()) ? string.Join("\r\n", headers) + "\r\n" : string.Empty;

   return "request:"   + context.Variables["message-id"] + "\n"
                       + requestLine + headerString + "\r\n" + body;
}
</log-to-eventhub>
```

### <a name="policy-declaration"></a>原則宣告
此原則運算式有一些值得一提的特別之處。 log-to-eventhub 原則有一個名為 logger-id 的屬性，這是指已在 API 管理服務中建立的記錄器名稱。 在 [如何在 Azure API 管理中將事件記錄至 Azure 事件中樞](api-management-howto-log-event-hubs.md)文件中可以找到如何在 API 管理服務中設定事件中樞記錄器的詳細資訊。 第二個屬性是一個選擇性參數，其指示事件中樞要在哪個資料分割中儲存訊息。 事件中樞使用資料分割來達到延展性，而且需要至少兩個資料分割。 只保證在一個資料分割內的訊息會依序傳遞。 如果我們未指示事件中樞要在哪一個資料分割中放置訊息，它會使用循環配置資源演算法來分散負載。 不過，這可能導致一些訊息未依順序處理。  

### <a name="partitions"></a>分割數
若要確保我們的訊息依序傳遞給取用者並利用資料分割的負載分散功能，我選擇將 HTTP 要求訊息傳送到一個資料分割，將 HTTP 回應訊息傳送到第二個資料分割。 如此可確保負載平均分散，而且我們可以保證所有要求和所有回應都會依序取用。 回應有可能在對應要求之前取用，但這不成問題，因為我們有不同的機制可讓要求與回應相互關聯，而且我們知道要求一律會出現在回應之前。

### <a name="http-payloads"></a>HTTP 裝載
在建置 `requestLine` 之後，我們會查看是否應該截斷要求本文。 要求本文會被截斷成只有 1024 個字元。 您可以增加此值，不過個別的事件中樞訊息受限於 256 KB，因此有些 HTTP 訊息本文可能會無法放入單一訊息中。 進行記錄和分析時，可以從 HTTP 要求行和標頭衍生大量資訊。 此外，許多 API 要求只會傳回小型本文，所以相較於降低傳輸、處理和儲存成本來保留所有的本文內容，截斷大型本文所造成的資訊值遺失相當微小。 有關處理本文的最後一個注意事項，就是我們必須將 `true` 傳遞至 As<string>() 方法，因為我們正在閱讀本文內容，但也希望後端 API 能夠讀取本文。 我們將 true 傳遞至這個方法，造成本文進行緩衝處理，以便第二次讀取本文。 一定要留意您的 API 是否上傳大型檔案或使用長輪詢。 在這些情況下，最好完全避免讀取本文。   

### <a name="http-headers"></a>HTTP 標頭
HTTP 標頭可以轉換成採用簡單索引鍵/值組格式的訊息格式。 我們已選擇刪除某些安全性機密欄位，以避免不必要地洩漏認證資訊。 API 金鑰及其他認證不太可能用於分析。 如果我們想要分析使用者及其使用的特定產品，我們可以從 `context` 物件取得該方式，然後將它加入至訊息。     

### <a name="message-metadata"></a>訊息中繼資料
建立要傳送至事件中樞的完整訊息時，第一行實際上不屬於 `application/http` 訊息。 第一行是額外的中繼資料，由訊息是要求或回應訊息以及用來使回應與要求相互關聯的訊息識別碼所組成。 使用如下所示的另一個原則，即可建立訊息識別碼：

```xml
<set-variable name="message-id" value="@(Guid.NewGuid())" />
```

我們可能已建立要求訊息，將它儲存在變數中，直到傳回回應為止，然後將要求和回應當作單一訊息傳送。 不過，獨立傳送要求和回應並使用訊息識別碼讓兩者相互關聯，我們便可取得訊息大小方面的更多彈性，還能夠利用多個資料分割並同時維護訊息順序，而要求將會更快出現在我們的記錄儀表板中。 在某些情況下，有效的回應也可能永遠不會傳送到事件中樞，這可能是因為 API 管理服務發生嚴重要求錯誤，但我們仍有這一筆要求記錄。

用於傳送回應 HTTP 訊息的原則看起來類似要求，所以完整的原則組態如下所示：

```xml
<policies>
  <inbound>
      <set-variable name="message-id" value="@(Guid.NewGuid())" />
      <log-to-eventhub logger-id="conferencelogger" partition-id="0">
      @{
          var requestLine = string.Format("{0} {1} HTTP/1.1\r\n",
                                                      context.Request.Method,
                                                      context.Request.Url.Path + context.Request.Url.QueryString);

          var body = context.Request.Body?.As<string>(true);
          if (body != null && body.Length > 1024)
          {
              body = body.Substring(0, 1024);
          }

          var headers = context.Request.Headers
                               .Where(h => h.Key != "Authorization" && h.Key != "Ocp-Apim-Subscription-Key")
                               .Select(h => string.Format("{0}: {1}", h.Key, String.Join(", ", h.Value)))
                               .ToArray<string>();

          var headerString = (headers.Any()) ? string.Join("\r\n", headers) + "\r\n" : string.Empty;

          return "request:"   + context.Variables["message-id"] + "\n"
                              + requestLine + headerString + "\r\n" + body;
      }
  </log-to-eventhub>
  </inbound>
  <backend>
      <forward-request follow-redirects="true" />
  </backend>
  <outbound>
      <log-to-eventhub logger-id="conferencelogger" partition-id="1">
      @{
          var statusLine = string.Format("HTTP/1.1 {0} {1}\r\n",
                                              context.Response.StatusCode,
                                              context.Response.StatusReason);

          var body = context.Response.Body?.As<string>(true);
          if (body != null && body.Length > 1024)
          {
              body = body.Substring(0, 1024);
          }

          var headers = context.Response.Headers
                                          .Select(h => string.Format("{0}: {1}", h.Key, String.Join(", ", h.Value)))
                                          .ToArray<string>();

          var headerString = (headers.Any()) ? string.Join("\r\n", headers) + "\r\n" : string.Empty;

          return "response:"  + context.Variables["message-id"] + "\n"
                              + statusLine + headerString + "\r\n" + body;
     }
  </log-to-eventhub>
  </outbound>
</policies>
```

`set-variable` 原則會建立一個可供 `<inbound>` 區段和 `<outbound>` 區段中的 `log-to-eventhub` 原則存取的值。  

## <a name="receiving-events-from-event-hubs"></a>從事件中樞接收事件
使用 [AMQP 通訊協定](http://www.amqp.org/)可從 Azure 事件中樞接收事件。 Microsoft 服務匯流排團隊已提供用戶端程式庫，以便取用事件。 支援兩種不同的方法：一個方法是成為「直接取用者」，另一個方法是使用 `EventProcessorHost` 類別。 在 [事件中樞程式設計指南](../event-hubs/event-hubs-programming-guide.md)中可找到這兩種方法的範例。 簡而言之，差別在於：`Direct Consumer` 給您完整控制權，而 `EventProcessorHost` 會替您做一些繁雜工作，但會假設您將如何處理這些事件。  

### <a name="eventprocessorhost"></a>EventProcessorHost
在此範例中，我們將使用 `EventProcessorHost` 以求簡化，但是它可能不是此特定案例的最佳選擇。 `EventProcessorHost` 會努力確定您不必擔心特定事件處理器類別內的執行緒問題。 不過，在我們的案例中，我們只是將訊息轉換成另一種格式，並使用非同步方法將它傳遞到另一個服務。 不需要更新共用狀態，因此沒有執行緒問題的風險。 在大部分的情況下， `EventProcessorHost` 可能是最佳選擇，當然也是比較容易的選項。     

### <a name="ieventprocessor"></a>IEventProcessor
使用 `EventProcessorHost` 時的中心概念是建立 `IEventProcessor` 介面的實作，其中包含 `ProcessEventAsync` 方法。 該方法的本質如下所示：

```csharp
async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
{

   foreach (EventData eventData in messages)
   {
       _Logger.LogInfo(string.Format("Event received from partition: {0} - {1}", context.Lease.PartitionId,eventData.PartitionKey));

       try
       {
           var httpMessage = HttpMessage.Parse(eventData.GetBodyStream());
           await _MessageContentProcessor.ProcessHttpMessage(httpMessage);
       }
       catch (Exception ex)
       {
           _Logger.LogError(ex.Message);
       }
   }
    ... checkpointing code snipped ...
}
```

系統會將 EventData 物件清單傳遞至此方法，而我們會逐一查看該清單。 每個方法的位元組會被剖析為 HttpMessage 物件，而該物件會傳遞至 IHttpMessageProcessor 的執行個體。

### <a name="httpmessage"></a>HttpMessage
`HttpMessage` 執行個體包含三個資料片段：

```csharp
public class HttpMessage
{
   public Guid MessageId { get; set; }
   public bool IsRequest { get; set; }
   public HttpRequestMessage HttpRequestMessage { get; set; }
   public HttpResponseMessage HttpResponseMessage { get; set; }

... parsing code snipped ...

}
```

`HttpMessage` 執行個體包含 `MessageId` GUID，它可讓我們將 HTTP 要求連接到對應的 HTTP 回應和一個布林值 (以識別物件是否包含 HttpRequestMessage 和 HttpResponseMessage 的執行個體)。 從 `System.Net.Http` 使用內建 HTTP 類別，我才能夠利用 `System.Net.Http.Formatting` 內含的 `application/http` 剖析程式碼。  

### <a name="ihttpmessageprocessor"></a>IHttpMessageProcessor
`HttpMessage` 執行個體會接著轉送到 `IHttpMessageProcessor` 的實作，這是我所建立的介面，用於分離事件的接收及解譯與 Azure 事件中樞及其實際處理。

## <a name="forwarding-the-http-message"></a>轉送 HTTP 訊息
在此範例中，我認為將 HTTP 要求推送至 [Runscope](http://www.runscope.com)很有趣。 Runscope 是專門從事 HTTP 偵錯、記錄和監視的雲端架構服務。 該服務有免費層，因此可輕易試用，它可讓我們即時看到流經 API 管理服務的 HTTP 要求。

`IHttpMessageProcessor` 實作如下所示：

```csharp
public class RunscopeHttpMessageProcessor : IHttpMessageProcessor
{
   private HttpClient _HttpClient;
   private ILogger _Logger;
   private string _BucketKey;
   public RunscopeHttpMessageProcessor(HttpClient httpClient, ILogger logger)
   {
       _HttpClient = httpClient;
       var key = Environment.GetEnvironmentVariable("APIMEVENTS-RUNSCOPE-KEY", EnvironmentVariableTarget.User);
       _HttpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("bearer", key);
       _HttpClient.BaseAddress = new Uri("https://api.runscope.com");
       _BucketKey = Environment.GetEnvironmentVariable("APIMEVENTS-RUNSCOPE-BUCKET", EnvironmentVariableTarget.User);
       _Logger = logger;
   }

   public async Task ProcessHttpMessage(HttpMessage message)
   {
       var runscopeMessage = new RunscopeMessage()
       {
           UniqueIdentifier = message.MessageId
       };

       if (message.IsRequest)
       {
           _Logger.LogInfo("Sending HTTP request " + message.MessageId.ToString());
           runscopeMessage.Request = await RunscopeRequest.CreateFromAsync(message.HttpRequestMessage);
       }
       else
       {
           _Logger.LogInfo("Sending HTTP response " + message.MessageId.ToString());
           runscopeMessage.Response = await RunscopeResponse.CreateFromAsync(message.HttpResponseMessage);
       }

       var messagesLink = new MessagesLink() { Method = HttpMethod.Post };
       messagesLink.BucketKey = _BucketKey;
       messagesLink.RunscopeMessage = runscopeMessage;
       var runscopeResponse = await _HttpClient.SendAsync(messagesLink.CreateRequest());
       _Logger.LogDebug("Request sent to Runscope");
   }
}
```

我能夠利用 [Runscope 的現有用戶端程式庫](http://www.nuget.org/packages/Runscope.net.hapikit/0.9.0-alpha)，所以可輕鬆地將 `HttpRequestMessage` 和 `HttpResponseMessage` 執行個體推送到它們的服務中。 若要存取 Runscope API，您需有一個帳戶和 API 金鑰。 在 [建立應用程式來存取 Runscope API](http://blog.runscope.com/posts/creating-applications-to-access-the-runscope-api) 螢幕錄製影片中可找到取得 API 金鑰的指示。

## <a name="complete-sample"></a>完整範例
範例的[原始程式碼](https://github.com/darrelmiller/ApimEventProcessor)和測試位於 GitHub 上。 您需要 [API 管理服務](get-started-create-service-instance.md)、[已連線的事件中樞](api-management-howto-log-event-hubs.md)及[儲存體帳戶](../storage/common/storage-create-storage-account.md)，才能自行執行此範例。   

此範例只是簡單的主控台應用程式，可接聽來自事件中樞的事件，將它們轉換為 `HttpRequestMessage` 和 `HttpResponseMessage` 物件，然後再轉送到 Runscope API。

在下列動畫影像中，您可以看見正在開發人員入口網站中對 API 提出的要求、顯示正在接收、處理和轉送訊息的主控台應用程式，然後是顯示在 Runscope 流量偵測器中的要求和回應。

![示範將要求轉送到 Runscope](./media/api-management-log-to-eventhub-sample/apim-eventhub-runscope.gif)

## <a name="summary"></a>總結
Azure API 管理服務提供了一個理想位置，可供擷取您的 API 的雙向 HTTP 流量。 Azure 事件中樞是一個可高度擴充、低成本的解決方案，用來擷取該流量並將它饋送到次要處理系統中，以便進行記錄、監視和其他複雜的分析。 連線到第三方監視系統 (像是 Runscope) 就像數十行程式碼一樣簡單。

## <a name="next-steps"></a>後續步驟
* 深入了解 Azure 事件中樞
  * [開始使用 Azure 事件中樞](../event-hubs/event-hubs-c-getstarted-send.md)
  * [使用 EventProcessorHost 接收訊息](../event-hubs/event-hubs-dotnet-standard-getstarted-receive-eph.md)
  * [事件中樞程式設計指南](../event-hubs/event-hubs-programming-guide.md)
* 深入了解 API 管理和事件中樞的整合
  * [如何將事件記錄到 Azure API 管理中的 Azure 事件中樞](api-management-howto-log-event-hubs.md)
  * [記錄器實體參考](https://docs.microsoft.com/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-logger-entity)
  * [log-to-eventhub 原則參考](https://msdn.microsoft.com/library/azure/dn894085.aspx#log-to-eventhub)
