---
title: Azure 服務匯流排和 .NET 與 AMQP 1.0 | Microsoft Docs
description: 搭配使用 .NET 的 Azure 服務匯流排與 AMQP
services: service-bus-messaging
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: 332bcb13-e287-4715-99ee-3d7d97396487
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/16/2018
ms.author: spelluru
ms.openlocfilehash: ad789b7a65fd12abb2a6e92c7c8896677de80cec
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43702233"
---
# <a name="use-service-bus-from-net-with-amqp-10"></a>搭配使用 .NET 的服務匯流排與 AMQP 1.0

服務匯流排套件 2.1 版或更新版本提供 AMQP 1.0 支援。 您可以從 [NuGet][NuGet] 下載「服務匯流排」軟體，以確保您擁有最新版本。

## <a name="configure-net-applications-to-use-amqp-10"></a>設定 .NET 應用程式以使用 AMQP 1.0

依預設，服務匯流排 .NET 用戶端程式庫能使用專屬的 SOAP 型通訊協定與服務匯流排服務通訊。 若要使用 AMQP 1.0 (而非預設的通訊協定)，您需要明確地設定服務匯流排連接字串，如下節內容所述。 除了這項變更之外，在使用 AMQP 1.0 時，應用程式程式碼會維持不變。

目前的版本中有幾項在使用 AMQP 時不支援的 API 功能。 不支援的功能皆列在[行為差異](#behavioral-differences)一節中。 在使用 AMQP 時，某些進階組態設定亦有不同的意義。

### <a name="configuration-using-appconfig"></a>使用 App.config 進行設定

讓應用程式使用 App.config 組態檔案來儲存設定是不錯的作法。 對於服務匯流排應用程式，您可以使用 App.config 來儲存服務匯流排連接字串。 範例 App.config 檔案如下所示：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
    <appSettings>
        <add key="Microsoft.ServiceBus.ConnectionString"
             value="Endpoint=sb://[namespace].servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[SAS key];TransportType=Amqp" />
    </appSettings>
</configuration>
```

`Microsoft.ServiceBus.ConnectionString` 設定的值是用來設定服務匯流排連線的服務匯流排連接字串。 其格式如下所示：

`Endpoint=sb://[namespace].servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[SAS key];TransportType=Amqp`

其中 `namespace` 和 `SAS key` 是當您建立服務匯流排命名空間時，取自 [Azure 入口網站][Azure portal]。 如需詳細資訊，請參閱[使用 Azure 入口網站建立服務匯流排命名空間][Create a Service Bus namespace using the Azure portal]。

使用 AMQP 時，在連接字串中附加 `;TransportType=Amqp`。 此標記法會指示用戶端程式庫使用 AMQP 1.0 來連線到「服務匯流排」。

## <a name="message-serialization"></a>訊息序列化

使用預設通訊協定時，.NET 用戶端程式庫的預設序列化行為是使用 [DataContractSerializer][DataContractSerializer] 類型將 [BrokeredMessage][BrokeredMessage] 執行個體序列化，以用於用戶端程式庫與「服務匯流排服務」之間的傳輸。 使用 AMQP 傳輸模式時，用戶端程式庫會使用 AMQP 類型系統，將[代理訊息][BrokeredMessage]序列化為 AMQP 訊息。 此序列化作業讓可能在不同平台上執行的接收應用程式 (例如使用 JMS API 來存取服務匯流排的 Java 應用程式) 能夠接收和解譯此訊息。

當您建構 [BrokeredMessage][BrokeredMessage] 執行個體時，您可以提供 .NET 物件作為建構函式的參數，進而做為訊息的本文。 如果是可以對應至 AMQP 基本類型的物件，本文會序列化為 AMQP 資料類型。 如果物件不能直接對應至 AMQP 基本類型 (也就是，應用程式所定義的自訂類型)，就會使用 [DataContractSerializer][DataContractSerializer] 將物件序列化，然後在 AMQP 資料訊息中傳送序列化的位元組。

若要促進與非 .NET 用戶端的互通性，在訊息本文中僅使用可直接序列化為 AMQP 類型的 .NET 類型。 下表詳細說明這些類型及對應的 AMQP 類型系統。

| .NET 主體物件類型 | 對應的 AMQP 類型 | AMQP 主體區段類型 |
| --- | --- | --- |
| 布林 |布林值 |AMQP 值 |
| byte |ubyte |AMQP 值 |
| ushort |ushort |AMQP 值 |
| uint |uint |AMQP 值 |
| ulong |ulong |AMQP 值 |
| sbyte |byte |AMQP 值 |
| short |short |AMQP 值 |
| int |int |AMQP 值 |
| long |long |AMQP 值 |
| float |float |AMQP 值 |
| double |double |AMQP 值 |
| decimal |decimal128 |AMQP 值 |
| char |char |AMQP 值 |
| Datetime |timestamp |AMQP 值 |
| Guid |uuid |AMQP 值 |
| byte[] |binary |AMQP 值 |
| 字串 |字串 |AMQP 值 |
| System.Collections.IList |list |AMQP 值︰集合中包含的項目只能是在此表格中定義的項目。 |
| System.Array |array |AMQP 值︰集合中包含的項目只能是在此表格中定義的項目。 |
| System.Collections.IDictionary |map |AMQP 值︰集合中包含的項目只能是在此表格中定義的項目。注意︰僅支援 String 索引鍵。 |
| Uri |描述的 string (請參閱下表) |AMQP 值 |
| DateTimeOffset |描述的 long (請參閱下表) |AMQP 值 |
| TimeSpan |描述的 long (請參閱下表) |AMQP 值 |
| Stream |binary |AMQP 資料 (可能有多個)。 Data 區段包含從 Stream 物件讀取的原始位元組。 |
| 其他物件 |binary |AMQP 資料 (可能有多個)。 包含使用 DataContractSerializer 或應用程式所提供序列化程式之物件的序列化二進位資料。 |

| .NET 類型 | 對應的 AMQP 描述類型 | 注意 |
| --- | --- | --- |
| Uri |`<type name=”uri” class=restricted source=”string”> <descriptor name=”com.microsoft:uri” /></type>` |Uri.AbsoluteUri |
| DateTimeOffset |`<type name=”datetime-offset” class=restricted source=”long”> <descriptor name=”com.microsoft:datetime-offset” /></type>` |DateTimeOffset.UtcTicks |
| 時間範圍 |`<type name=”timespan” class=restricted source=”long”> <descriptor name=”com.microsoft:timespan” /></type> ` |TimeSpan.Ticks |

## <a name="behavioral-differences"></a>行為差異

相較於預設通訊協定，使用 AMQP 時，服務匯流排 .NET API 有一些細微的行為差異：

* [OperationTimeout][OperationTimeout] 屬性會被忽略。
* `MessageReceiver.Receive(TimeSpan.Zero)` 會實作為 `MessageReceiver.Receive(TimeSpan.FromSeconds(10))`。
* 只有最初收到訊息的訊息接收者能夠以鎖定權杖完成訊息。

## <a name="control-amqp-protocol-settings"></a>控制 AMQP 通訊協定設定

[.NET API](/dotnet/api/) 會公開數個可控制 AMQP 通訊協定行為的設定：

* **[MessageReceiver.PrefetchCount](/dotnet/api/microsoft.servicebus.messaging.messagereceiver.prefetchcount?view=azureservicebus-4.0.0#Microsoft_ServiceBus_Messaging_MessageReceiver_PrefetchCount)**：控制套用至連結的初始信用額度。 預設值為 0。
* **[MessagingFactorySettings.AmqpTransportSettings.MaxFrameSize](/dotnet/api/microsoft.servicebus.messaging.amqp.amqptransportsettings.maxframesize?view=azureservicebus-4.0.0#Microsoft_ServiceBus_Messaging_Amqp_AmqpTransportSettings_MaxFrameSize)**︰控制在連線開啟時間進行交涉時所提供的 AMQP 框架大小上限。 預設值為 65,536 個位元組。
* **[MessagingFactorySettings.AmqpTransportSettings.BatchFlushInterval](/dotnet/api/microsoft.servicebus.messaging.amqp.amqptransportsettings.batchflushinterval?view=azureservicebus-4.0.0#Microsoft_ServiceBus_Messaging_Amqp_AmqpTransportSettings_BatchFlushInterval)**︰如果傳輸可分批進行，這個值會決定用於傳送配置的延遲上限。 預設由傳送者/接收者繼承。 個別的傳送者/接收者可以覆寫預設值 (20 毫秒)。
* **[MessagingFactorySettings.AmqpTransportSettings.UseSslStreamSecurity](/dotnet/api/microsoft.servicebus.messaging.amqp.amqptransportsettings.usesslstreamsecurity?view=azureservicebus-4.0.0#Microsoft_ServiceBus_Messaging_Amqp_AmqpTransportSettings_UseSslStreamSecurity)**︰控制是否透過 SSL 連線建立 AMQP 連線。 預設值為 **true**。

## <a name="next-steps"></a>後續步驟

準備好進行深入了解嗎？ 請造訪下列連結：

* [服務匯流排 AMQP 概觀]
* [AMQP 1.0 通訊協定指南]

[Create a Service Bus namespace using the Azure portal]: service-bus-create-namespace-portal.md
[DataContractSerializer]: https://msdn.microsoft.com/library/system.runtime.serialization.datacontractserializer.aspx
[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage?view=azureservicebus-4.0.0
[Microsoft.ServiceBus.Messaging.MessagingFactory.AcceptMessageSession]: /dotnet/api/microsoft.servicebus.messaging.messagingfactory.acceptmessagesession?view=azureservicebus-4.0.0#Microsoft_ServiceBus_Messaging_MessagingFactory_AcceptMessageSession
[OperationTimeout]: /dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings.operationtimeout?view=azureservicebus-4.0.0#Microsoft_ServiceBus_Messaging_MessagingFactorySettings_OperationTimeout
[NuGet]: http://nuget.org/packages/WindowsAzure.ServiceBus/
[Azure portal]: https://portal.azure.com
[服務匯流排 AMQP 概觀]: service-bus-amqp-overview.md
[AMQP 1.0 通訊協定指南]: service-bus-amqp-protocol-guide.md

