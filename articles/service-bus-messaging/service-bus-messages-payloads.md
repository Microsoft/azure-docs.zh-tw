---
title: Azure 服務匯流排訊息、承載及序列化 | Microsoft Docs
description: 服務匯流排訊息承載概觀
services: service-bus-messaging
documentationcenter: ''
author: clemensv
manager: timlt
editor: ''
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/26/2018
ms.author: spelluru
ms.openlocfilehash: 00c7605b09c32328a8324b13b8151a258a39dc22
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48857596"
---
# <a name="messages-payloads-and-serialization"></a>訊息、承載和序列化

Microsoft Azure 服務匯流排的功能就是處理訊息。 訊息會以索引鍵/值組屬性形式來傳遞承載以及中繼資料，並對服務匯流排和應用程式描述承載並提供處理指示。 有時，單獨使用中繼資料就足以傳遞傳送者想要與接收者溝通的資訊，因此承載就會維持空白。

適用於 .NET 和 Java 的官方服務匯流排用戶端物件模型會反映抽象的服務匯流排訊息結構，而此訊息結構會對應至服務匯流排支援的傳送與接收網路通訊協定。
 
服務匯流排訊息是由服務匯流排絕不會在服務端上以任何形式處理的二進位承載區段，以及兩組屬性所組成。 「訊息代理程式屬性」會由系統預先定義。 這些預先定義的屬性可能會控制訊息代理程式內的訊息層級功能，或是對應到一般和標準化的中繼資料項目。 「使用者屬性」是由應用程式定義和設定的索引鍵/值組集合。
 
下表列出預先定義的訊息代理程式屬性。 所有官方用戶端 API 均會使用這些名稱，同時也會在 HTTP 通訊協定對應的 [BrokerProperties](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Properties_) JSON 物件中使用。
 
括弧中列出 AMQP 通訊協定層級所使用的對等名稱。 

| 屬性名稱                         | 說明                                                                                                                                                                                                                                                                                                                                                                                                                               |
|---------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|  [ContentType](/dotnet/api/microsoft.azure.servicebus.message.contenttype) (content-type)           | 選擇性地說明訊息的承載，並提供遵循 RFC2045 第 5 節格式的描述項；例如 `application/json`。                                                                                                                                                                                                                                                                                             |
|  [CorrelationId](/dotnet/api/microsoft.azure.servicebus.message.correlationid#Microsoft_Azure_ServiceBus_Message_CorrelationId) (correlation-id)       | 讓應用程式能夠指定訊息的內容以供相互關聯之用；例如，反映正要回覆之訊息的 **MessageId**。                                                                                                                                                                                                                                                                  |
| [DeadLetterSource](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.deadlettersource)                      | 只會設定於已成為無效信件且後續會從無效信件佇列自動轉送到另一個實體的訊息中。 指示其中訊息已成為無效信件的實體。 這個屬性是唯讀的。                                                                                                                                                                                                                                  |
| [DeliveryCount](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.deliverycount)                         | 針對此訊息嘗試進行的傳遞次數。 當訊息鎖定到期時或接收者明確放棄此訊息時，這個計數就會遞增。 這個屬性是唯讀的。                                                                                                                                                                                                                                                  |
| [EnqueuedSequenceNumber](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.enqueuedsequencenumber)                | 對於已自動轉送的訊息，此屬性會反映訊息原始提交時首次指派給訊息的序號。 這個屬性是唯讀的。                                                                                                                                                                                                                                                                |
| [EnqueuedTimeUtc](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.enqueuedtimeutc)                       | 在實體中接受並儲存訊息的 UTC 時刻。 當接收者不想信任傳送者的時鐘時，這個值可用來作為權威且中立的抵達時間指標。 這個屬性是唯讀的。                                                                                                                                                                                                   |
|  [ExpiresAtUtc](/dotnet/api/microsoft.azure.servicebus.message.expiresatutc) (absolute-expiry-time) | 由於其到期而將訊息標示為移除且無法再從實體中擷取的 UTC 時刻。 到期會由 **TimeToLive** 屬性所控制，而這個屬性會從 EnqueuedTimeUtc + TimeToLive 計算而得。 這個屬性是唯讀的。                                                                                                                                                                           |
| [ForcePersistence](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.forcepersistence)                      | 對於已設定 [EnableExpress](/dotnet/api/microsoft.servicebus.messaging.queuedescription.enableexpress) 旗標的佇列或主題，可設定這個屬性來指示訊息必須保存於磁碟中，之後才能進行認可。 這是適用於所有非 Express 實體的標準行為。                                                                                                                                                                                                         |
| [Label](/dotnet/api/microsoft.azure.servicebus.message.label) (subject)                       | 這個屬性可讓應用程式以標準化方式向接收者指出訊息目的，類似於電子郵件主旨行。                                                                                                                                                                                                                                                                                  |
| [LockedUntilUtc](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.lockeduntilutc)                        | 對於在鎖定下接收的訊息 (查看鎖定接收模式，不是預先安置的)，這個屬性反映訊息會維持鎖定在佇列/訂用帳戶中的結束 UTC 時刻。 當鎖定到期時，[DeliveryCount](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.deliverycount) 就會遞增，而訊息就再次可供擷取。 這個屬性是唯讀的。                                                                                                                         |
| [LockToken](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.locktoken)                             | 鎖定權杖是「查看鎖定」接收模式中由訊息代理程式所持有鎖定的參考。 權杖可透過[延遲](message-deferral.md) API 以將鎖定永久固定，然後從一般傳遞狀態流程中取出訊息。 這個屬性是唯讀的。                                                                                                                                                               |
| [MessageId](/dotnet/api/microsoft.azure.servicebus.message.messageid) (message-id)                | 訊息識別碼是應用程式定義的值，可唯一識別訊息及其承載。 識別碼是一個自由格式的字串，可以反映 GUID 或衍生自應用程式內容的識別碼。 如果啟用[重複偵測](duplicate-detection.md)功能，就可以識別並移除具有相同 **MessageId** 之訊息的第二次及後續提交。                                                                |
| [PartitionKey](/dotnet/api/microsoft.azure.servicebus.message.partitionkey)                          | 針對[分割的實體](service-bus-partitioning.md)，設定此值可將相關訊息指派到相同的內部資料分割，如此便能正確記錄提交順序。 資料分割會由雜湊函式透過此值來選擇，無法直接選擇。 對於工作階段感知的實體，**SessionId** 屬性會覆寫此值。                                                                                                       |
| [ReplyTo](/dotnet/api/microsoft.azure.servicebus.message.replyto) (reply-to)                    | 這個選擇性且由應用程式定義的值，乃是向訊息接收者表示回覆路徑的標準方法。 當傳送者預期收到回覆時，會將值設為預期回覆要送往的佇列或主題絕對或相對路徑。                                                                                                                                           |
| [ReplyToSessionId](/dotnet/api/microsoft.azure.servicebus.message.replytosessionid) (reply-to-group-id)  | 這個值可以擴充 **ReplyTo** 資訊，並指定在傳送到回覆實體時針對回覆所應設定的 **SessionId**。                                                                                                                                                                                                                                                                            |
| [ScheduledEnqueueTimeUtc](/dotnet/api/microsoft.azure.servicebus.message.scheduledenqueuetimeutc)               | 對於僅能在延遲之後可供擷取的訊息，這個屬性會定義以邏輯方式將訊息加入佇列、排序，並使其可供擷取的 UTC 時刻。                                                                                                                                                                                                                 |
| [SequenceNumber](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.sequencenumber)                        | 序號是指派給訊息的唯一 64 位元整數，會由訊息代理程式和函式所接收並儲存，以作為真正的識別碼。 對於分割的實體，最前面的 16 位元會反映資料分割識別碼。 序號會以無間斷方式循序遞增。 它們會在 48 - 64 位元範圍用盡時回復為 0。 這個屬性是唯讀的。                                                                |
| [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid) (group-id)                  | 對於工作階段感知的實體，這個應用程式定義的值會指定訊息的工作階段關係。 具有相同工作階段識別碼的訊息會受限於摘要鎖定，並且能夠確實地依序處理和分離信號。 對於不是工作階段感知的實體，會忽略此值。                                                                                                                                     |
| [大小](/dotnet/api/microsoft.azure.servicebus.message.size)                                  | 將訊息在訊息代理程式記錄中儲存的大小反映為位元組計數，因為它會計入儲存體配額。 這個屬性是唯讀的。                                                                                                                                                                                                                                                                                                       |
| [State](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.state)                                 | 指示記錄中訊息的狀態。 這個屬性只有在訊息瀏覽 (查看) 期間才有意義。它可判斷訊息是否為「作用中」且在到達佇列頂端時可供擷取、訊息是否延遲，或者正等待納入排程。 這個屬性是唯讀的。                                                                                                                                           |
| [TimeToLive](/dotnet/api/microsoft.azure.servicebus.message.timetolive)                            | 這個值是訊息到期之前的相對持續期間，從訊息代理程式接受並儲存訊息的瞬間開始，正如 **EnqueueTimeUtc** 所擷取的時間。 未明確設定時，所使用的值便是對應佇列或主題的 **DefaultTimeToLive**。 訊息層級的 **TimeToLive** 值不能比實體的 **DefaultTimeToLive** 設定還長。 如果較長，則會以無訊息方式加以調整。 |
| [To](/dotnet/api/microsoft.azure.servicebus.message.to) (to)                               | 這個屬性會保留以便日後供路由傳送案例使用，訊息代理程式目前會忽略這個屬性。 應用程式可在規則驅動的自動轉送鏈結案例中使用此值，以指出訊息預期的邏輯目的地。                                                                                                                                                                                   |
| [ViaPartitionKey](/dotnet/api/microsoft.azure.servicebus.message.viapartitionkey)                       | 如果訊息會透過交易範圍內的傳輸佇列來傳送，這個值就會選取傳輸佇列資料分割。                                                                                                                                                                                                                                                                                                                 |

抽象訊息模型可讓訊息透過 HTTP (實際上一律為 HTTPS) 來發佈至佇列，並且讓訊息透過 AMQP 來擷取。 不論是何種狀況，訊息在對應通訊協定的內容中看起來都很正常。 訊息代理程式屬性會視需要加以轉譯，使用者屬性也會對應至相對通訊協定訊息模型上最適當的位置。 在 HTTP 中，使用者屬性會直接與 HTTP 標頭相互對應；在 AMQP 中，使用者屬性則會與 **application-properties** 對應相互對應。

## <a name="message-routing-and-correlation"></a>訊息路由傳送和相互關聯

先前提及的部分訊息代理程式屬性 (亦即 [To](/dotnet/api/microsoft.azure.servicebus.message.to)、[ReplyTo](/dotnet/api/microsoft.azure.servicebus.message.replyto)、[ReplyToSessionId](/dotnet/api/microsoft.azure.servicebus.message.replytosessionid)、[MessageId](/dotnet/api/microsoft.azure.servicebus.message.messageid)、[CorrelationId](/dotnet/api/microsoft.azure.servicebus.message.correlationid) 及 [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid)) 可用來協助應用程式將訊息路由傳送到特定目的地。 為了進一步說明，請考慮下列數個模式：

- **簡單要求/回覆**：發行者將訊息傳送到佇列，並預期會有來自訊息取用者的回覆。 為了接收回覆，發行者會擁有一個佇列，並預期回覆會傳遞到此佇列。 該佇列的位址會以輸出訊息的 **ReplyTo** 屬性來表示。 當取用者回應時，會將已處理訊息的 **MessageId** 複製到回覆訊息的 **CorrelationId** 屬性，並將訊息傳遞到 **ReplyTo** 屬性所指示的目的地。 根據應用程式內容而定，一個訊息可能會產生多個回覆。
- **多點傳送要求/回覆**：這是前一個模式的變形。發行者會將訊息傳送至主題，讓多個訂閱者能夠取用訊息。 每個訂閱者可以使用先前所述的方式來回應。 此模式適用於探索或點名案例，而回應者通常會透過使用者屬性或在承載內部中來識別自己的身分。 如果 **ReplyTo** 指向主題，就能將這類探索回應散發給目標對象。
- **多工**：此工作階段功能可透過單一佇列或訂用帳戶，啟用相關訊息的多工作業，使得特定接收者將工作階段維持鎖定之際，相關訊息的每個工作階段或群組 (藉由符合的 **SessionId** 值來識別) 會路由傳送給接收者。 如需工作階段的詳細資料，請在[這裡](message-sessions.md)進行深入了解。
- **多工要求/回覆**：此工作階段功能會啟用多工回覆，讓數個發行者可以共用一個回覆佇列。 發行者可以設定 **ReplyToSessionId**，以指示取用者將該值複製到回覆訊息的 **SessionId** 屬性。 發行佇列或主題不需是工作階段感知的。 傳送訊息時，發行者接著可以有條件地接受工作階段接收者，以特地等候佇列上具有指定 **SessionId** 的工作階段具體化。 

在服務匯流排命名空間內進行路由傳送，可以使用自動轉送鏈結和主題訂用帳戶規則來實現。 在命名空間之間進行路由傳送，可以[使用 Azure LogicApps](https://azure.microsoft.com/services/logic-apps/) 來實現。 如同上述清單中所述，**To** 屬性會保留以供日後使用，且最終會由訊息代理程式藉由特別啟用的功能來解譯。 希望實作路由傳送的應用程式應該根據使用者屬性來執行此動作，而不是依賴 **To** 屬性，不過，此方式目前並不會造成相容性問題。

## <a name="payload-serialization"></a>承載序列化

承載不論是在傳輸或儲存於服務匯流排時，都是不透明的二進位區塊。 [ContentType](/dotnet/api/microsoft.azure.servicebus.message.contenttype) 屬性可讓應用程式使用適用於屬性值的建議格式來描述承載 (根據 IETF RFC2045 為 MIME 內容類型的描述)，例如 `application/json;charset=utf-8`。

與 Java 或.NET 標準變數不同，服務匯流排 API 的 .NET Framework 版本支援將任意 .NET 物件傳遞到建構函式來建立 **BrokeredMessage** 執行個體。 

使用舊版 SBMP 通訊協定時，接著會使用預設的二進位序列化程式來將那些物件序列化，或是使用外部提供的序列化程式來序列化。 使用 AMQP 通訊協定時，會將物件序列化為 AMQP 物件。 接收者可以利用 [GetBody<T>()](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.getbody#Microsoft_ServiceBus_Messaging_BrokeredMessage_GetBody__1) 方法擷取那些物件，並提供預期的類型。 使用 AMQP，將物件序列化為 **ArrayList** 和 **IDictionary<string,object>** 物件的 AMQP 圖形，而任何 AMQP 用戶端都可將它們解碼。 

儘管這個隱藏的序列化魔法相當便利，但應用程式還是應該明確控制物件序列化，並先將其物件圖形轉換為資料流，之後再將它們納入訊息中，在接收者端則以相反順序來執行。 這會產生可交互作用的結果。 此外，也應該注意，當 AMQP 具有功能強大的二進位編碼模型時，系統會將它繫結到 AMQP 傳訊生態系統，而 HTTP 用戶端將在為這類承載進行解碼時遇到問題。 

我們通常會建議針對結構化資料使用 JSON 和 Apache Avro 作為承載格式。

.NET 標準和 Java API 變數只接受位元組陣列，這表示應用程式必須處理物件序列化控制。 

## <a name="next-steps"></a>後續步驟

若要深入了解服務匯流排傳訊，請參閱下列主題：

* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)
