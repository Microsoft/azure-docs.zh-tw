---
title: Azure 服務匯流排傳訊例外狀況 | Microsoft Docs
description: 服務匯流排傳訊例外狀況和建議的動作清單。
services: service-bus-messaging
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: 3d8526fe-6e47-4119-9f3e-c56d916a98f9
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/31/2018
ms.author: spelluru
ms.openlocfilehash: eab36c9bf38249fd7e2a2b24914190b86800693f
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43699197"
---
# <a name="service-bus-messaging-exceptions"></a>服務匯流排傳訊例外狀況
本文列出一些 Microsoft Azure 服務匯流排傳訊 API 產生的例外狀況。 此參考可能有所變更，請不定期查看更新。

## <a name="exception-categories"></a>例外狀況類別
傳訊 API 會產生下列類別的例外狀況，以及可用來嘗試修正它們的相關動作。 請注意，例外狀況的意義和原因會隨著傳訊實體的類型而異：

1. 使用者程式碼撰寫錯誤 ([System.ArgumentException](https://msdn.microsoft.com/library/system.argumentexception.aspx)、[System.InvalidOperationException](https://msdn.microsoft.com/library/system.invalidoperationexception.aspx)、[System.OperationCanceledException](https://msdn.microsoft.com/library/system.operationcanceledexception.aspx)、[System.Runtime.Serialization.SerializationException](https://msdn.microsoft.com/library/system.runtime.serialization.serializationexception.aspx))。 一般動作：請先嘗試修正此程式碼，再繼續執行。
2. 設定/組態錯誤 ([Microsoft.ServiceBus.Messaging.MessagingEntityNotFoundException](/dotnet/api/microsoft.azure.servicebus.messagingentitynotfoundexception)、[System.UnauthorizedAccessException](https://msdn.microsoft.com/library/system.unauthorizedaccessexception.aspx))。 一般動作：檢閱您的組態並視需要進行變更。
3. 暫時性例外狀況 ([Microsoft.ServiceBus.Messaging.MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception)、[Microsoft.ServiceBus.Messaging.ServerBusyException](/dotnet/api/microsoft.azure.servicebus.serverbusyexception)、[Microsoft.ServiceBus.Messaging.MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception))。 一般動作：重試此操作或通知使用者。
4. 其他例外狀況 ([System.Transactions.TransactionException](https://msdn.microsoft.com/library/system.transactions.transactionexception.aspx)、[System.TimeoutException](https://msdn.microsoft.com/library/system.timeoutexception.aspx)、[Microsoft.ServiceBus.Messaging.MessageLockLostException](/dotnet/api/microsoft.azure.servicebus.messagelocklostexception)、[Microsoft.ServiceBus.Messaging.SessionLockLostException](/dotnet/api/microsoft.azure.servicebus.sessionlocklostexception))。 一般動作︰依例外狀況類型而異；請參閱下一節中的表格。 

## <a name="exception-types"></a>例外狀況類型
下表列出傳訊例外狀況類型及其原因，並指出您可以採取的建議動作。

| **例外狀況類型** | **描述/原因/範例** | **建議的動作** | **自動/立即重試附註** |
| --- | --- | --- | --- |
| [TimeoutException](https://msdn.microsoft.com/library/system.timeoutexception.aspx) |伺服器未在 [OperationTimeout](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings#Microsoft_ServiceBus_Messaging_MessagingFactorySettings_OperationTimeout)控制的指定時間內回應要求的作業。 伺服器可能已完成要求的作業。 這可能是由於網路或其他基礎結構延遲所導致。 |檢查系統狀態的一致性，並視需要重試。 請參閱 [逾時例外狀況](#timeoutexception)。 |在某些情況下，重試也許有幫助；將重試邏輯新增至程式碼。 |
| [InvalidOperationException](https://msdn.microsoft.com/library/system.invalidoperationexception.aspx) |不允許在伺服器或服務內執行要求的使用者作業。 如需詳細資訊，請參閱例外狀況訊息。 例如，如果是在 [ReceiveAndDelete](/dotnet/api/microsoft.azure.servicebus.receivemode) 模式收到訊息，[Complete()](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync) 將會產生這個例外狀況。 |檢查程式碼和文件。 確定要求的作業無效。 |重試將無助益。 |
| [OperationCanceledException](https://msdn.microsoft.com/library/system.operationcanceledexception.aspx) |嘗試在已關閉、中止或處置的物件上叫用作業。 極少數的情況下，環境交易是已處置狀態。 |檢查程式碼，確定它不會在已處置物件上叫用作業。 |重試將無助益。 |
| [UnauthorizedAccessException](https://msdn.microsoft.com/library/system.unauthorizedaccessexception.aspx) |[TokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider) 物件無法取得權杖、權杖無效，或權杖不包含執行作業所需的宣告。 |確定權杖提供者是以正確的值建立。 檢查存取控制服務的組態。 |在某些情況下，重試也許有幫助；將重試邏輯新增至程式碼。 |
| [ArgumentException](https://msdn.microsoft.com/library/system.argumentexception.aspx)<br /> [ArgumentNullException](https://msdn.microsoft.com/library/system.argumentnullexception.aspx)<br />[ArgumentOutOfRangeException](https://msdn.microsoft.com/library/system.argumentoutofrangeexception.aspx) |提供給方法的一個或多個引數無效。<br /> 提供給 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) 或 [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory#Microsoft_ServiceBus_Messaging_MessagingFactory_Create_System_Collections_Generic_IEnumerable_System_Uri__) 的 URI 包含路徑區段。<br /> 提供給 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) 或 [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory#Microsoft_ServiceBus_Messaging_MessagingFactory_Create_System_Collections_Generic_IEnumerable_System_Uri__) 的 URI 配置無效。 <br />屬性值大於 32 KB。 |檢查呼叫程式碼，並確定引數正確無誤。 |重試將無助益。 |
| [MessagingEntityNotFoundException](/dotnet/api/microsoft.azure.servicebus.messagingentitynotfoundexception) |與作業相關聯的實體不存在或已被刪除。 |確定實體已存在。 |重試將無助益。 |
| [MessageNotFoundException](/dotnet/api/microsoft.servicebus.messaging.messagenotfoundexception) |嘗試接收具有特定序號的訊息。 找不到此訊息。 |確定尚未收到訊息。 檢查寄不出信件佇列，查看訊息是否已停止傳送。 |重試將無助益。 |
| [MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception) |用戶端無法建立服務匯流排連線。 |確定提供的主機名稱正確，且主機可以連線。 |如果有間歇性的連線問題，重試也許有幫助。 |
| [ServerBusyException](/dotnet/api/microsoft.azure.servicebus.serverbusyexception) |服務目前無法處理要求。 |用戶端可以等待一段時間，然後再重試作業。 |用戶端可以在特定間隔後重試。 如果重試產生不同的例外狀況，請檢查該例外狀況的重試行為。 |
| [MessageLockLostException](/dotnet/api/microsoft.azure.servicebus.messagelocklostexception) |與訊息相關的鎖定權杖已過期，或找不到鎖定權杖。 |處置訊息。 |重試將無助益。 |
| [SessionLockLostException](/dotnet/api/microsoft.azure.servicebus.sessionlocklostexception) |與此工作階段相關聯的鎖定遺失。 |中止 [MessageSession](/dotnet/api/microsoft.servicebus.messaging.messagesession) 物件。 |重試將無助益。 |
| [MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception) |可能會在下列情況中擲回的一般傳訊例外狀況：<br /> 利用屬於不同實體類型 (例如主題) 的名稱或路徑嘗試建立 [QueueClient](/dotnet/api/microsoft.azure.servicebus.queueclient) 。<br />  嘗試傳送大於 256 KB 的訊息。 處理要求時伺服器或服務發生錯誤。 如需詳細資訊，請參閱例外狀況訊息。 這通常是暫時性例外狀況。 |查看程式碼，並確定訊息內文只使用可序列化的物件 (或使用自訂序列化程式)。 查看文件來了解支援的屬性值類型，並且只使用支援的類型。 查看 [IsTransient](/dotnet/api/microsoft.servicebus.messaging.messagingexception#Microsoft_ServiceBus_Messaging_MessagingException_IsTransient) 屬性。 如果該屬性為 **True**，您就可以重試作業。 |重試行為未定義，而且可能沒有幫助。 |
| [MessagingEntityAlreadyExistsException](/dotnet/api/microsoft.servicebus.messaging.messagingentityalreadyexistsexception) |嘗試在該服務命名空間中以另一個實體已在使用的名稱建立實體。 |刪除現有的實體，或選擇不同的名稱來建立實體。 |重試將無助益。 |
| [QuotaExceededException](/dotnet/api/microsoft.azure.servicebus.quotaexceededexception) |傳訊實體已達到允許的大小上限，或已超過命名空間的連線數目上限。 |從實體或其子佇列接收訊息，在實體中建立空間。 請參閱 [QuotaExceededException](#quotaexceededexception)。 |如果在此同時已移除訊息，重試可能會有幫助。 |
| [RuleActionException](/dotnet/api/microsoft.servicebus.messaging.ruleactionexception) |如果您嘗試建立無效的規則動作，服務匯流排會傳回此例外狀況。 如果處理停止傳送的訊息的規則動作時發生錯誤，服務匯流排會將此例外狀況附加至該訊息。 |檢查規則動作的正確性。 |重試將無助益。 |
| [FilterException](/dotnet/api/microsoft.servicebus.messaging.filterexception) |如果您嘗試建立無效的篩選，服務匯流排會傳回此例外狀況。 如果處理停止傳送的訊息的篩選時發生錯誤，服務匯流排會將此例外狀況附加至該訊息。 |檢查篩選的正確性。 |重試將無助益。 |
| [SessionCannotBeLockedException](/dotnet/api/microsoft.servicebus.messaging.sessioncannotbelockedexception) |嘗試接受含有特定工作階段識別碼的工作階段，但該工作階段目前被另一個用戶端鎖定。 |確定其他用戶端已解除鎖定工作階段。 |如果工作階段在過渡期間被解除鎖定，重試可能會有幫助。 |
| [TransactionSizeExceededException](/dotnet/api/microsoft.servicebus.messaging.transactionsizeexceededexception) |交易包含太多作業。 |減少此交易的作業數目。 |重試將無助益。 |
| [MessagingEntityDisabledException](/dotnet/api/microsoft.azure.servicebus.messagingentitydisabledexception) |在停用的實體上要求執行階段作業。 |啟用實體。 |如實體在過渡期間被啟用，重試可能會有幫助。 |
| [NoMatchingSubscriptionException](/dotnet/api/microsoft.servicebus.messaging.nomatchingsubscriptionexception) |如果您將訊息傳送至已啟用預先篩選的主題，但沒有符合的篩選，服務匯流排就會傳回此例外狀況。 |確定至少有一個篩選相符。 |重試將無助益。 |
| [MessageSizeExceededException](/dotnet/api/microsoft.servicebus.messaging.messagesizeexceededexception) |訊息承載超過 256 KB 的限制。 請注意，256 KB 的限制是總訊息大小，可包括系統屬性和任何 .NET 負荷。 |減少訊息裝載大小，然後再重試作業。 |重試將無助益。 |
| [TransactionException](https://msdn.microsoft.com/library/system.transactions.transactionexception.aspx) |環境交易 (*Transaction.Current*) 無效。 其可能已完成或中止。 內部例外狀況可能會提供其他資訊。 | |重試將無助益。 |
| [TransactionInDoubtException](https://msdn.microsoft.com/library/system.transactions.transactionindoubtexception.aspx) |嘗試在不確定的交易上執行作業，或嘗試認可交易而交易變得不確定。 |應用程式必須處理這個例外狀況 (當成特殊狀況)，因為交易可能已被認可。 |- |

## <a name="quotaexceededexception"></a>QuotaExceededException
[QuotaExceededException](/dotnet/api/microsoft.azure.servicebus.quotaexceededexception) 指出已超過某特定實體的配額。

### <a name="queues-and-topics"></a>佇列和主題
對佇列和主題而言，這通常是佇列的大小。 錯誤訊息屬性會包含進一步的詳細資訊，如下例所示：

```Output
Microsoft.ServiceBus.Messaging.QuotaExceededException
Message: The maximum entity size has been reached or exceeded for Topic: ‘xxx-xxx-xxx’. 
    Size of entity in bytes:1073742326, Max entity size in bytes:
1073741824..TrackingId:xxxxxxxxxxxxxxxxxxxxxxxxxx, TimeStamp:3/15/2013 7:50:18 AM
```

訊息指出主題超過其大小限制，本例為 1 GB (預設大小限制)。 

### <a name="namespaces"></a>命名空間

針對命名空間，[QuotaExceededException](/dotnet/api/microsoft.azure.servicebus.quotaexceededexception) 可能表示應用程式已超過命名空間的連線數目上限。 例如︰

```Output
Microsoft.ServiceBus.Messaging.QuotaExceededException: ConnectionsQuotaExceeded for namespace xxx.
<tracking-id-guid>_G12 ---> 
System.ServiceModel.FaultException`1[System.ServiceModel.ExceptionDetail]: 
ConnectionsQuotaExceeded for namespace xxx.
```

#### <a name="common-causes"></a>常見的原因
這個錯誤有兩個常見的原因︰無效信件佇列和訊息接收者未作用。

1. **[無效信件佇列](service-bus-dead-letter-queues.md)**：讀取器無法完成訊息，當鎖定過期後，訊息會傳回佇列/主題。 如果讀取器遇到例外狀況，阻止它呼叫 [BrokeredMessage.Complete](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.complete)，就會發生這個錯誤。 訊息讀取 10 次後，預設會移至無效信件佇列。 這種行為由 [QueueDescription.MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) 屬性控制，預設值是 10。 訊息堆積在無效信件佇列中會佔用空間。
   
    若要解決此問題，請閱讀和完成無效信件佇列中的訊息，就像您處理任何其他佇列一樣。 您可以使用 [FormatDeadLetterPath](/dotnet/api/microsoft.azure.servicebus.entitynamehelper.formatdeadletterpath) 方法來協助格式化無效信件佇列路徑。
2. **收件者停止** ：收件者停止接收佇列或訂用帳戶的訊息。 識別這個原因的方法是查看 [QueueDescription.MessageCountDetails](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails) 屬性，它會顯示訊息的完整解析。 如果 [ActiveMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.activemessagecount) 屬性很高或不斷增加，表示訊息撰寫的速度超過讀取的速度。

## <a name="timeoutexception"></a>TimeoutException
[TimeoutException](https://msdn.microsoft.com/library/system.timeoutexception.aspx) 表示使用者啟始作業所用的時間長過作業逾時。 

您應該檢查 [ServicePointManager.DefaultConnectionLimit](https://msdn.microsoft.com/library/system.net.servicepointmanager.defaultconnectionlimit) 屬性的值，因為達到這個限制可能也會造成 [TimeoutException](https://msdn.microsoft.com/library/system.timeoutexception.aspx)。

### <a name="queues-and-topics"></a>佇列和主題
佇列和主題的逾時是在 [MessagingFactorySettings.OperationTimeout](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings#Microsoft_ServiceBus_Messaging_MessagingFactorySettings_OperationTimeout) 屬性中指定，作為連接字串的一部分，或透過 [ServiceBusConnectionStringBuilder](/dotnet/api/microsoft.azure.servicebus.servicebusconnectionstringbuilder) 指定。 錯誤訊息本身可能不盡相同，但它一定會包含目前作業的指定逾時值。 



## <a name="next-steps"></a>後續步驟

如需完整的服務匯流排 .NET API 參考資料，請參閱 [Azure .NET API 參考資料](/dotnet/api/overview/azure/service-bus)。

若要深入了解[服務匯流排](https://azure.microsoft.com/services/service-bus/)，請參閱下列文章：

* [服務匯流排訊息概觀](service-bus-messaging-overview.md)
* [服務匯流排基本概念](service-bus-fundamentals-hybrid-solutions.md)
* [服務匯流排架構](service-bus-architecture.md)

