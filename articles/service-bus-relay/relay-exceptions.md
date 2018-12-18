---
title: Azure 轉送例外狀況以及解決方式 | Microsoft Docs
description: Azure 轉送例外狀況以及您可以採取來協助解決這些例外狀況之建議動作的清單。
services: service-bus-relay
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: 5f9dd02c-cce0-43b3-8eb8-744f0c27f38c
ms.service: service-bus-relay
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/20/2017
ms.author: spelluru
ms.openlocfilehash: 0fe30fe95e77adceaa5013f89206b08daf2a58a2
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43702090"
---
# <a name="azure-relay-exceptions"></a>Azure 轉送例外狀況

本文列出 Azure 轉送 API 所產生的一些例外狀況。 此參考可能有所變更，請不定期查看更新。

## <a name="exception-categories"></a>例外狀況類別

轉送 API 產生的例外狀況分成下列類別。 同時列出您可以採取來解決例外狀況的建議動作。

*   **使用者程式碼撰寫錯誤**：[System.ArgumentException](https://msdn.microsoft.com/library/system.argumentexception.aspx)、[System.InvalidOperationException](https://msdn.microsoft.com/library/system.invalidoperationexception.aspx)、[System.OperationCanceledException](https://msdn.microsoft.com/library/system.operationcanceledexception.aspx)、[System.Runtime.Serialization.SerializationException](https://msdn.microsoft.com/library/system.runtime.serialization.serializationexception.aspx)。 

    **一般動作**：請先嘗試修正此程式碼，再繼續執行。
*   **設定/組態錯誤**：[System.UnauthorizedAccessException](https://msdn.microsoft.com/library/system.unauthorizedaccessexception.aspx)。 

    **一般動作**：檢閱您的組態。 視需要變更組態。
*   **暫時性例外狀況**：[Microsoft.ServiceBus.Messaging.MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception)、[Microsoft.ServiceBus.Messaging.ServerBusyException](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception)、[Microsoft.ServiceBus.Messaging.MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception)。 

    **一般動作**：重試此作業或通知使用者。
*   **其他例外狀況**：[System.Transactions.TransactionException](https://msdn.microsoft.com/library/system.transactions.transactionexception.aspx)、[System.TimeoutException](https://msdn.microsoft.com/library/system.timeoutexception.aspx)。 

    **一般動作**︰例外狀況類型特定。 請參閱下列各節中的資料表。 

## <a name="exception-types"></a>例外狀況類型

下表列出傳訊例外狀況類型及其原因。 同時列出您可以採取來解決例外狀況的建議動作。

| **例外狀況類型** | **說明** | **建議動作** | **自動或立即重試附註** |
| --- | --- | --- | --- |
| [逾時](https://msdn.microsoft.com/library/system.timeoutexception.aspx) |伺服器未在 [OperationTimeout](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings.operationtimeout)控制的指定時間內回應要求的作業。 伺服器可能已完成要求的作業。 這可能是由於網路或其他基礎結構延遲所導致。 |檢查系統狀態的一致性，然後視需要重試。 請參閱 [TimeoutException](#timeoutexception)。 |在某些情況下，重試也許有幫助；將重試邏輯新增至程式碼。 |
| [作業無效](https://msdn.microsoft.com/library/system.invalidoperationexception.aspx) |不允許在伺服器或服務內執行要求的使用者作業。 如需詳細資訊，請參閱例外狀況訊息。 |檢查程式碼和文件。 確定要求的作業有效。 |重試將無助益。 |
| [已取消作業](https://msdn.microsoft.com/library/system.operationcanceledexception.aspx) |嘗試在已關閉、中止或處置的物件上叫用作業。 極少數的情況下，環境交易是已處置狀態。 |檢查程式碼，確定它不會在已處置物件上叫用作業。 |重試將無助益。 |
| [未經授權的存取](https://msdn.microsoft.com/library/system.unauthorizedaccessexception.aspx) |[TokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider) 物件無法取得權杖、權杖無效，或權杖不包含執行作業所需的宣告。 |確定權杖提供者是以正確的值建立。 檢查存取控制服務的組態。 |在某些情況下，重試也許有幫助；將重試邏輯新增至程式碼。 |
| [引數例外狀況](https://msdn.microsoft.com/library/system.argumentexception.aspx)<br /> [引數 Null](https://msdn.microsoft.com/library/system.argumentnullexception.aspx)<br />[引數超出範圍](https://msdn.microsoft.com/library/system.argumentoutofrangeexception.aspx) |發生下列一或多個情況︰<br />提供給方法的一個或多個引數無效。<br /> 提供給 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) 或 [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory.create) 的 URI 包含一或多個路徑區段。<br />提供給 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) 或 [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory.create) 的 URI 配置無效。 <br />屬性值大於 32 KB。 |檢查呼叫程式碼，並確定引數正確無誤。 |重試將無助益。 |
| [伺服器忙碌](/dotnet/api/microsoft.servicebus.messaging.serverbusyexception) |服務目前無法處理要求。 |用戶端可以等待一段時間，然後再重試作業。 |用戶端可以在特定間隔之後重試。 如果重試產生不同的例外狀況，請檢查該例外狀況的重試行為。 |
| [超過配額](/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception) |傳訊實體已達到允許的大小上限。 |從實體或其子佇列接收訊息，在實體中建立空間。 請參閱 [QuotaExceededException](#quotaexceededexception)。 |如果在此同時已移除訊息，重試可能會有幫助。 |
| [超過訊息大小](/dotnet/api/microsoft.servicebus.messaging.messagesizeexceededexception) |訊息承載超過 256 KB 的限制。 請注意，256 KB 的限制是總訊息大小。 總訊息大小可包括系統屬性和任何 Microsoft .NET 負荷。 |減少訊息裝載大小，然後再重試作業。 |重試將無助益。 |

## <a name="quotaexceededexception"></a>QuotaExceededException

[QuotaExceededException](/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception) 指出已超過某特定實體的配額。

若為轉送，這個例外狀況會包裝 [System.ServiceModel.QuotaExceededException](https://msdn.microsoft.com/library/system.servicemodel.quotaexceededexception.aspx)，指出已超過這個端點接聽程式的最大數目。 這會表示在例外狀況訊息的 **MaximumListenersPerEndpoint** 值中。

## <a name="timeoutexception"></a>TimeoutException
[TimeoutException](https://msdn.microsoft.com/library/system.timeoutexception.aspx) 表示使用者啟始作業所用的時間長過作業逾時。 

檢查 [ServicePointManager.DefaultConnectionLimit](https://msdn.microsoft.com/library/system.net.servicepointmanager.defaultconnectionlimit) 屬性的值。 觸達此限制也會導致 [TimeoutException](https://msdn.microsoft.com/library/system.timeoutexception.aspx)。

若為轉送，您可能會在第一次開啟轉送傳送者連線時收到逾時例外狀況。 這個例外狀況有兩個常見的原因︰

*   [OpenTimeout](https://msdn.microsoft.com/library/wcf.opentimeout.aspx) 值可能太小 (甚至幾分之一秒)。
* 內部部署轉送接聽程式可能沒有回應 (或可能遇到禁止接聽程式接受新用戶端連線的防火牆規則問題)，而 [OpenTimeout](https://msdn.microsoft.com/library/wcf.opentimeout.aspx) 值約小於 20 秒。

範例：

```
'System.TimeoutException’: The operation did not complete within the allotted timeout of 00:00:10.
The time allotted to this operation may have been a portion of a longer timeout.
```

### <a name="common-causes"></a>常見的原因
這個錯誤有兩個常見的原因︰

*   **組態不正確**
    
    操作條件的作業逾時可能太小。 用戶端 SDK 的作業逾時預設值為 60 秒。 請檢查在程式碼中是否將值設定過小。 請注意，CPU 使用量和網路的條件會影響完成作業所花費的時間。 不要將作業逾時設為非常小的值是很好的想法。
*   **暫時性服務錯誤**

    有時候，轉送服務可能會遇到延遲處理要求。 例如，這可能會在高流量期間發生。 如果發生這種情況，您可以在延遲後重試作業，直到作業成功為止。 如果多次嘗試同一作業之後持續失敗，請瀏覽 [Azure 服務狀態網站](https://azure.microsoft.com/status/)，看看是否有任何已知的服務中斷。

## <a name="next-steps"></a>後續步驟
* [Azure 轉送常見問題集](relay-faq.md)
* [建立轉送命名空間](relay-create-namespace-portal.md)
* [開始使用 Azure 轉送和 .NET](relay-hybrid-connections-dotnet-get-started.md)
* [開始使用 Azure 轉送和節點](relay-hybrid-connections-node-get-started.md)

