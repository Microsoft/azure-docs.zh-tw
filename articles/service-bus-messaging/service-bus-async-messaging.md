---
title: 服務匯流排非同步傳訊 |Microsoft Docs
description: 「Azure 服務匯流排」非同步傳訊說明。
services: service-bus-messaging
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: f1435549-e1f2-40cb-a280-64ea07b39fc7
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/26/2018
ms.author: spelluru
ms.openlocfilehash: 9bacce96e65a7aef611bec3ddae8b1872d5f9fae
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47391458"
---
# <a name="asynchronous-messaging-patterns-and-high-availability"></a>非同步傳訊模式和高可用性

有各種不同的方式可以實作非同步傳訊。 「Azure 服務匯流排」利用佇列、主題及訂用帳戶來支援透過儲存和轉送機制進行的非同步處理。 在正常 (同步) 作業中，您可將訊息傳送至佇列和主題，並接收來自佇列和訂用帳戶的訊息。 您撰寫的應用程式依存於這些實體永遠可用。 當實體健康狀態變更時，因為有許多不同的情況，您必須有辦法提供能滿足大多數需求的容量減少實體。

應用程式通常會使用非同步傳訊模式來啟用一些通訊案例。 您可以建置一個應用程式，讓用戶端可以傳送訊息至服務 (即使服務並未執行)。 對於經歷通訊暴增的應用程式，佇列可提供一個緩衝通訊的地方，協助平穩負載。 最後，您可取得簡單但有效率的負載平衡器，將訊息分散於多部電腦上。

為了維護這些實體的可用性，請考慮讓這些實體對持久的傳訊系統呈現無法使用的各種不同方式。 一般而言，我們會看到我們所撰寫的應用程式因下列情況而無法使用實體：

* 無法傳送訊息。
* 無法接收訊息。
* 無法管理實體 (建立、擷取、更新或刪除實體)。
* 無法連絡服務。

上述每個失敗有不同的失敗模式存在，可讓應用程式以降低某些能力的方式繼續執行工作。 例如，可傳送訊息但不會收到訊息的系統仍可以接收來自客戶的訂單，但無法處理這些訂單。 本主題討論可能發生的潛在問題，以及如何減輕這些問題。 服務匯流排已引進數個您必須選擇加入的緩和措施，而本主題也會討論以規則來控管使用這些選擇加入的緩和措施。

## <a name="reliability-in-service-bus"></a>服務匯流排的可靠性
有數種方法可以處理訊息和實體問題，而且有用來控管適當使用這些緩和措施的指導方針。 若要了解這些指導方針，您必須先了解服務匯流排中可能發生何種失敗。 由於 Azure 系統的設計，所有這些問題通常都很短暫。 概括而言，無法使用的各種原因如下︰

* 從服務匯流排所依賴的外部系統節流。 從與儲存體和運算資源的互動進行節流。
* 服務匯流排所依賴系統的問題。 例如，指定的儲存體部份可能會發生問題。
* 單一子系統上的服務匯流排失敗。 在此情況下，運算節點可進入不一致的狀態且本身必須重新啟動，導致它提供的所有實體平衡其他節點的負載。 這又會導致短期內的訊息處理速度緩慢。
* Azure 資料中心內的服務匯流排失敗。 這就是「災難性失敗」，系統有數分鐘或數小時的時間無法連線。

> [!NOTE]
> 「儲存體」一詞可表示 Azure 儲存體和 SQL Azure。
> 
> 

對於這些問題，服務匯流排有數種緩和措施。 下列各節會討論每個問題及其各自的緩和措施。

### <a name="throttling"></a>節流
使用服務匯流排，節流可達到協同合作的訊息速率管理。 每個個別的服務匯流排節點都容納許多實體。 每個實體都是從 CPU、記憶體、儲存體和其他面向提出系統需求。 當上述任何面向偵測到超出所定義臨界值的使用量時，服務匯流排可以拒絕指定的要求。 呼叫端會收到 [ServerBusyException][ServerBusyException] 並在 10 秒後重試。

在緩和措施中，程式碼必須讀取錯誤並停止任何訊息重試至少 10 秒。 因為此錯誤會發生於客戶應用程式的各個部分，因此預計每個部分都能獨立執行重試邏輯。 此程式碼可藉由在佇列或主題上啟用資料分割，以降低進行節流的機率。

### <a name="issue-for-an-azure-dependency"></a>Azure 相依性問題
Azure 中的其他元件可能會不時出現服務問題。 例如，當服務匯流排使用的系統正在升級時，該系統可能會暫時經歷降低的功能。 若要解決這幾類問題，服務匯流排會定期調查並實作緩和措施。 這些緩和措施的確會出現副作用。 例如，若要處理儲存體的暫時性問題，服務匯流排會實作可讓訊息傳送作業以一致的方式運作的系統。 基於緩和措施的本質，傳送的訊息可能需要 15 分鐘的時間才會出現在受影響的佇列或訂用帳戶中，而可供進行接收作業。 一般而言，大部分實體不會發生這個問題。 不過，基於 Azure 內服務匯流排中的實體數目，有一小部分的服務匯流排客戶有時會需要此緩和措施。

### <a name="service-bus-failure-on-a-single-subsystem"></a>單一子系統上的服務匯流排失敗
在任何應用程式中，有些情況會導致服務匯流排的內部元件變得不一致。 當服務匯流排偵測到這種情況時，它會從應用程式收集資料以協助診斷發生什麼狀況。 收集資料後，應用程式會在嘗試回到一致狀態時重新啟動。 這個程序非常迅速地發生，而且會導致實體呈現無法使用長達數分鐘，然而一般的停機時間短很多。

在這些情況下，用戶端應用程式會產生 [System.TimeoutException][System.TimeoutException] 或 [MessagingException][MessagingException] 例外狀況。 服務匯流排包含此問題的緩和措施 (採用自動用戶端重試邏輯形式)。 一旦重試期間期滿又未傳遞訊息，您便可以使用[配對的命名空間][paired namespaces]等其他功能來進行探索。 配對的命名空間有其他需要的注意事項 (請見該文章中的討論)。

### <a name="failure-of-service-bus-within-an-azure-datacenter"></a>Azure 資料中心內的服務匯流排失敗
Azure 資料中心失敗的最可能原因是服務匯流排或相依系統的升級部署失敗。 隨著平台日趨成熟，這種失敗的可能性已大幅降低。 發生資料中心失敗的原因也可能包含下列因素︰

* 電力中斷 (電源和供電失效)。
* 連線 (用戶端與 Azure 之間的網際網路中斷)。

在這兩種情況下，自然或人為災難都會造成此問題。 若要解決這個問題並確保您仍可傳送訊息，您可以使用[配對的命名空間][paired namespaces]，以允許在主要位置再次恢復良好狀況的期間，先將訊息傳送到次要位置。 如需詳細資訊，請參閱[將應用程式與服務匯流排中斷和災難隔絕的最佳做法 (英文)][Best practices for insulating applications against Service Bus outages and disasters]。

## <a name="paired-namespaces"></a>配對的命名空間
[配對的命名空間][paired namespaces]功能支援下列案例︰資料中心內的「服務匯流排」實體或部署變得無法使用。 雖然此事件不常發生，但分散式系統仍然必須準備好處理情況最糟的案例。 通常會因為服務匯流排所依賴的某個元素遇到短期問題而發生此事件。 為了維護中斷期間的應用程式可用性，服務匯流排使用者可以使用兩個不同的命名空間 (最好位於不同的資料中心) 來裝載其傳訊實體。 本節的其餘部分使用下列術語：

* 主要命名空間︰與您的應用程式互動進行傳送和接收作業的命名空間。
* 次要命名空間: 做為主要命名空間之備份的命名空間。 應用程式邏輯不會與此命名空間互動。
* 容錯移轉間隔︰在應用程式從主要命名空間切換至次要命名空間之前接受正常失敗的時間量。

配對的命名空間支援「傳送可用性」。 傳送可用性保留了傳送訊息的能力。 若要使用傳送可用性，您的應用程式必須符合下列需求：

1. 只會從主要命名空間接收訊息。
2. 傳送到指定佇列或主題的訊息可能不會按順序抵達。
3. 工作階段內的訊息可能不會按順序抵達。 這是工作階段的正常功能毀損。 這表示您的應用程式會使用工作階段以邏輯方式群組訊息。
4. 只會在主要命名空間上維護工作階段狀態。
5. 在次要佇列將所有訊息傳遞至主要佇列之前，主要佇列可以上線並開始接受訊息。

下列各節討論 API、如何實作 API 以及顯示使用此功能的範例程式碼。 請注意，這項功能有相關聯的計費暗示。

### <a name="the-messagingfactorypairnamespaceasync-api"></a>MessagingFactory.PairNamespaceAsync API
配對的命名空間功能會在 [Microsoft.ServiceBus.Messaging.MessagingFactory][Microsoft.ServiceBus.Messaging.MessagingFactory] 類別上包含 [PairNamespaceAsync][PairNamespaceAsync] 方法：

```csharp
public Task PairNamespaceAsync(PairedNamespaceOptions options);
```

當工作完成時，命名空間配對也會完成，而可供任何使用 [MessagingFactory][MessagingFactory] 執行個體建立的 [MessageReceiver][MessageReceiver]、[QueueClient][QueueClient] 或 [TopicClient][TopicClient] 採取動作。 [Microsoft.ServiceBus.Messaging.PairedNamespaceOptions][Microsoft.ServiceBus.Messaging.PairedNamespaceOptions] 是 [MessagingFactory][MessagingFactory] 物件所提供的不同配對類型的基底類別。 目前，唯一的衍生類別是名為 [SendAvailabilityPairedNamespaceOptions][SendAvailabilityPairedNamespaceOptions] 的類別，它會實作傳送可用性需求。 [SendAvailabilityPairedNamespaceOptions][SendAvailabilityPairedNamespaceOptions] 有一組建立在彼此之上的建構函式。 查看具有大部份參數的建構函式，以便了解其他建構函式的行為。

```csharp
public SendAvailabilityPairedNamespaceOptions(
    NamespaceManager secondaryNamespaceManager,
    MessagingFactory messagingFactory,
    int backlogQueueCount,
    TimeSpan failoverInterval,
    bool enableSyphon)
```

這些參數具有下列意義：

* *secondaryNamespaceManager*：次要命名空間的已初始化 [NamespaceManager][NamespaceManager] 執行個體，可供 [PairNamespaceAsync][PairNamespaceAsync] 方法用來設定次要命名空間。 命名空間管理員會用來取得命名空間中的佇列清單，並確定必要的待處理項目佇列存在。 如果這些佇列不存在，則會加以建立。 [NamespaceManager][NamespaceManager] 必須要能夠使用 **Manage** 宣告來建立權杖。
* *messagingFactory*︰次要命名空間的 [MessagingFactory][MessagingFactory] 執行個體。 [MessagingFactory][MessagingFactory] 物件可用來將訊息傳送給積存佇列，而如果 [EnableSyphon][EnableSyphon] 屬性設定為 **true**，還可從該積存佇列接收訊息。
* *backlogQueueCount*︰要建立的積存佇列數目。 此值必須至少為 1。 將訊息傳送至待處理項目時，會隨機選擇其中一個佇列。 如果您將此值設定為 1，則只能使用一個佇列。 當發生此情況且這一個積存佇列產生錯誤時，用戶端便無法嘗試不同的積存佇列，而可能無法傳送您的訊息。 我們建議將此值設定為較大的值並將此值預設為 10。 視您的應用程式每天傳送的資料量而定，您可以將此值變更為較大或較小的值。 每個待處理項目佇列最多可以保留 5 GB 的訊息。
* *failoverInterval*︰將任何單一實體切換至次要命名空間之前，您接受主要命名空間失敗的時間量。 容錯移轉會以逐一實體的方式進行。 單一命名空間中的實體經常留存在服務匯流排中的不同節點。 某一個實體失敗不表示另一個實體也失敗。 您可以將此值設定為 [System.TimeSpan.Zero][System.TimeSpan.Zero]，以在您的第一個非暫時性失敗後，立即容錯移轉至次要命名空間。 觸發容錯移轉計時器的失敗是 [IsTransient][IsTransient] 屬性為 false 的任何 [MessagingException][MessagingException]，或是 [System.TimeoutException][System.TimeoutException]。 其他例外狀況 (例如 [UnauthorizedAccessException][UnauthorizedAccessException]) 並不會導致容錯移轉，因為它們表示的是用戶端設定不正確。 [ServerBusyException][ServerBusyException] 不會導致容錯移轉，因為正確的模式是等待 10 秒鐘，然後重新傳送訊息。
* *enableSyphon*︰表示此特殊的配對應該也會將訊息從次要命名空間擷取回主要命名空間。 一般而言，傳送訊息的應用程式應將此值設定為 **false**；接收訊息的應用程式應將此值設定為 **true**。 原因是訊息接收端通常比訊息傳送端少。 視接收端的數目而定，您可以選擇讓單一應用程式執行個體處理 Syphon 職責。 使用許多接收端會牽涉到每個待處理項目佇列的計費。

若要使用程式碼，請建立主要 [MessagingFactory][MessagingFactory] 執行個體、次要 [MessagingFactory][MessagingFactory] 執行個體、次要 [NamespaceManager][NamespaceManager] 執行個體和 [SendAvailabilityPairedNamespaceOptions][SendAvailabilityPairedNamespaceOptions] 執行個體。 呼叫很簡單，如下所示：

```csharp
SendAvailabilityPairedNamespaceOptions sendAvailabilityOptions = new SendAvailabilityPairedNamespaceOptions(secondaryNamespaceManager, secondary);
primary.PairNamespaceAsync(sendAvailabilityOptions).Wait();
```

當 [PairNamespaceAsync][PairNamespaceAsync] 方法所傳回的工作完成時，即表示所有項目都已設定妥當並可供使用。 傳回工作之前，您可能尚未完成配對正常運作所需的所有背景工作。 因此，直到工作傳回時，您才能開始傳送訊息。 如果發生任何失敗，例如認證錯誤或無法建立待處理項目佇列，則會在工作完成時立即擲回這些例外狀況。 在工作傳回後，請檢查 [SendAvailabilityPairedNamespaceOptions][SendAvailabilityPairedNamespaceOptions] 執行個體上的 [BacklogQueueCount][BacklogQueueCount] 屬性，以確認已找到或建立佇列。 對於前面的程式碼，該作業會顯示如下:

```csharp
if (sendAvailabilityOptions.BacklogQueueCount < 1)
{
    // Handle case where no queues were created.
}
```

## <a name="next-steps"></a>後續步驟
既然您已了解服務匯流排中非同步傳訊的基本概念，請閱讀[配對的命名空間][paired namespaces]以取得更多詳細資料。

[ServerBusyException]: /dotnet/api/microsoft.servicebus.messaging.serverbusyexception
[System.TimeoutException]: https://msdn.microsoft.com/library/system.timeoutexception.aspx
[MessagingException]: /dotnet/api/microsoft.servicebus.messaging.messagingexception
[Best practices for insulating applications against Service Bus outages and disasters]: service-bus-outages-disasters.md
[Microsoft.ServiceBus.Messaging.MessagingFactory]: /dotnet/api/microsoft.servicebus.messaging.messagingfactory
[MessageReceiver]: /dotnet/api/microsoft.servicebus.messaging.messagereceiver
[QueueClient]: /dotnet/api/microsoft.servicebus.messaging.queueclient
[TopicClient]: /dotnet/api/microsoft.servicebus.messaging.topicclient
[Microsoft.ServiceBus.Messaging.PairedNamespaceOptions]: /dotnet/api/microsoft.servicebus.messaging.pairednamespaceoptions
[MessagingFactory]: /dotnet/api/microsoft.servicebus.messaging.messagingfactory
[SendAvailabilityPairedNamespaceOptions]: /dotnet/api/microsoft.servicebus.messaging.sendavailabilitypairednamespaceoptions
[NamespaceManager]: /dotnet/api/microsoft.servicebus.namespacemanager
[PairNamespaceAsync]: /dotnet/api/microsoft.servicebus.messaging.messagingfactory#Microsoft_ServiceBus_Messaging_MessagingFactory_PairNamespaceAsync_Microsoft_ServiceBus_Messaging_PairedNamespaceOptions_
[EnableSyphon]: /dotnet/api/microsoft.servicebus.messaging.sendavailabilitypairednamespaceoptions#Microsoft_ServiceBus_Messaging_SendAvailabilityPairedNamespaceOptions_EnableSyphon
[System.TimeSpan.Zero]: https://msdn.microsoft.com/library/system.timespan.zero.aspx
[IsTransient]: /dotnet/api/microsoft.servicebus.messaging.messagingexception#Microsoft_ServiceBus_Messaging_MessagingException_IsTransient
[UnauthorizedAccessException]: https://msdn.microsoft.com/library/system.unauthorizedaccessexception.aspx
[BacklogQueueCount]: /dotnet/api/microsoft.servicebus.messaging.sendavailabilitypairednamespaceoptions?redirectedfrom=MSDN#Microsoft_ServiceBus_Messaging_SendAvailabilityPairedNamespaceOptions_BacklogQueueCount
[paired namespaces]: service-bus-paired-namespaces.md
