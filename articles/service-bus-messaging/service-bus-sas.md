---
title: 使用共用存取簽章的 Azure 服務匯流排存取控制 | Microsoft Docs
description: 使用共用存取簽章的服務匯流排存取控制概觀，詳細說明 Azure 服務匯流排之 SAS 授權的相關資訊。
services: service-bus-messaging
documentationcenter: na
author: spelluru
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/14/2018
ms.author: spelluru
ms.openlocfilehash: adc9d64c4756e055b3ff9b772feebd80902d2c9f
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48857868"
---
# <a name="service-bus-access-control-with-shared-access-signatures"></a>使用共用存取簽章的服務匯流排存取控制

*共用存取簽章* (SAS) 是服務匯流排傳訊的主要安全性機制。 本文討論 SAS、其運作方式以及如何以跨平台的方式使用它們。

SAS 會根據授權規則保護對服務匯流排的存取。 這些規則會設定於命名空間或訊息實體 (轉送、佇列或主題)。 授權規則具有名稱、與特定權限相關聯，並且承載了密碼編譯金鑰組。 您可以透過服務匯流排 SDK 使用規則的名稱和金鑰，或用於您自己的程式碼中以產生 SAS 權杖。 接著，用戶端可將權杖傳至服務匯流排，以證明要求之作業的授權。

## <a name="overview-of-sas"></a>SAS 的概觀

共用存取簽章是使用簡易權杖的宣告式授權機制。 使用 SAS 時，金鑰一律不會透過網路傳遞。 金鑰可用來以密碼編譯方式簽署稍後可由服務驗證的資訊。 SAS 的作用可類似於用戶端直接擁有授權規則名稱和相符金鑰的使用者名稱和密碼。 SAS 的作用也可類似於同盟安全性模型；在此模型中，用戶端會收到來自於安全性權杖服務、具有時效性且已簽署的存取權杖，而從未擁有簽署金鑰。

服務匯流排中的 SAS 驗證，會使用具有相關存取權限的具名[共用存取授權規則](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule)以及主要和次要密碼編譯金鑰組進行設定。 金鑰是採用 Base64 表示法的 256 位元值。 您可以在服務匯流排[轉送](../service-bus-relay/relay-what-is-it.md)、[佇列](/service-bus-messaging/service-bus-messaging-overview.md#queues)和[主題](/service-bus-messaging/service-bus-messaging-overview.md#topics)上，於命名空間層級設定規則。

[共用存取簽章](/dotnet/api/microsoft.servicebus.sharedaccesssignaturetokenprovider)權杖包含所選授權規則的名稱、應存取之資源的 URI、到期時間，以及使用所選授權規則的主要或次要密碼編譯金鑰對這些欄位計算的 HMAC RSA-SHA256 密碼編譯簽章。

## <a name="shared-access-authorization-policies"></a>共用存取授權原則

每個服務匯流排命名空間和每個服務匯流排實體，都有一個由規則組成的共用存取授權原則。 命名空間層級的原則會套用至命名空間內的所有實體，無論其個別的原則組態為何。

對於每個授權原則規則，您會決定三項資訊：**名稱**、**範圍**和**權限**。 **名稱** 只是該範圍內的唯一名稱。 範圍也很簡單：它是發生問題之資源的 URI。 針對服務匯流排命名空間，範圍是完整的網域名稱 (FQDN)，例如 `https://<yournamespace>.servicebus.windows.net/`。

原則規則所授與的權限可由下列項目組合而成：

* 「傳送」- 授與將訊息傳送至實體的權限
* 「接聽」- 授與接聽 (轉送) 或接收 (佇列、訂用帳戶) 和所有相關訊息處理的權限
* 「管理」- 授與管理命名空間拓撲的權限，包括建立和刪除實體

「管理」權限包含「傳送」和「接收」權限。

命名空間或實體原則最多可包含 12 個共用存取授權規則，而提供三組規則的空間，每組規則分別涵蓋基本權限以及「傳送」與「接聽」的組合。 此限制強調 SAS 原則存放區不應為使用者或服務帳戶存放區。 如果您的應用程式需要根據使用者或服務身分識別授與服務匯流排的存取權，則應實作會在驗證和存取檢查後發行 SAS 權杖的安全性權杖服務。

授權規則會被指派「主要金鑰」和「次要金鑰」。 這些是密碼編譯增強式金鑰。 請勿遺失或洩漏金鑰 - 它們永遠都可在 [Azure 入口網站][Azure portal]取得。 您可以使用其中一個產生的金鑰，以及您可以隨時重新產生它們。 如果您重新產生或變更原則中的金鑰，所有先前根據該金鑰發行的權杖都將立即無效。 不過，根據這類權杖建立的作用中連線會繼續運作，直到權杖到期。

當您建立服務匯流排命名空間時，系統會自動為命名空間建立名為 **RootManageSharedAccessKey** 的原則規則。 此原則具有整個命名空間的「管理」權限。 建議您將此規則視為系統管理**根**帳戶，且不要將其用於您的應用程式。 您可以在入口網站中，透過 Powershell 或 Azure CLI 在命名空間的 [設定] 索引標籤中建立額外的原則規則。

## <a name="configuration-for-shared-access-signature-authentication"></a>共用存取簽章驗證的設定

您可以在服務匯流排命名空間、佇列或主題上設定 [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) 規則。 目前不支援在服務匯流排訂用帳戶上設定 [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) ，但是您可以使用在命名空間或主題上設定的規則來保護訂用帳戶的存取安全。 如需可說明此程序的實用範例，請參閱 [搭配使用共用存取簽章 (SAS) 驗證與服務匯流排訂用帳戶](http://code.msdn.microsoft.com/Using-Shared-Access-e605b37c) 範例。

![SAS](./media/service-bus-sas/service-bus-namespace.png)

在此圖中，*manageRuleNS*、*sendRuleNS* 和 *listenRuleNS* 授權規則套用至佇列 Q1 和主題 T1，而 *listenRuleQ* 和 *sendRuleQ* 僅套用至佇列 Q1 以及 *sendRuleT* 僅套用至主題 T1。

## <a name="generate-a-shared-access-signature-token"></a>產生共用存取簽章權杖

任何有權存取授權規則名稱及其簽署金鑰之一的用戶端，都可以產生 SAS 權杖。 權杖是以下列格式編寫字串而產生的：

```
SharedAccessSignature sig=<signature-string>&se=<expiry>&skn=<keyName>&sr=<URL-encoded-resourceURI>
```

* **`se`** - 權杖到期時間。 此整數反映自 1970 年 1 月 1 日的 Epoch `00:00:00 UTC` (UNIX Epoch) 起到權杖到期時所經過的秒數。
* **`skn`** - 授權規則個名稱。
* **`sr`** - 所存取之資源的 URI。
* **`sig`** - 簽章。

`signature-string` 是對資源 URI (**範圍**的說明請見上一節) 和權杖到期時間的字串表示 (以 CRLF 分隔) 計算的 SHA-256 雜湊。

雜湊計算看起來類似下列虛擬程式碼，會傳回 256 位元/32 位元組的雜湊值。

```
SHA-256('https://<yournamespace>.servicebus.windows.net/'+'\n'+ 1438205742)
```

權杖會包含非雜湊值，因此接收者可以用相同的參數重新計算雜湊，驗證簽發者擁有有效的簽署金鑰。 

資源 URI 是宣告其存取權之服務匯流排資源的完整 URI。 例如，`http://<namespace>.servicebus.windows.net/<entityPath>` 或 `sb://<namespace>.servicebus.windows.net/<entityPath>`；也就是 `http://contoso.servicebus.windows.net/contosoTopics/T1/Subscriptions/S3`。 URI 必須是[百分比編碼](https://msdn.microsoft.com/library/4fkewx0t.aspx)。

用於簽署的共用存取授權規則必須設定於此 URI 或其中一個階層式上層所指定的實體。 例如，先前範例中的 `http://contoso.servicebus.windows.net/contosoTopics/T1` 或 `http://contoso.servicebus.windows.net`。

SAS 權杖適用於所有以 `signature-string` 中所使用的 `<resourceURI>` 開頭的資源。

## <a name="regenerating-keys"></a>重新產生金鑰

建議您定期重新產生用於 [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) 物件的金鑰。 由於有主要和次要金鑰槽存在，因此您可以逐步輪替金鑰。 如果您的應用程式通常會使用主要金鑰，您可以將主要金鑰複製到次要金鑰槽中，然後再重新產生主要金鑰。 接著，新的主要金鑰值將可設定到可繼續使用次要槽中的舊有主要金鑰進行存取的用戶端應用程式中。 所有的用戶端皆更新後，您即可重新產生次要金鑰，而最終淘汰掉舊的主要金鑰。

如果您確知或懷疑金鑰受到危害而必須撤銷金鑰，您可以重新產生 [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) 的 [PrimaryKey](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule#Microsoft_ServiceBus_Messaging_SharedAccessAuthorizationRule_PrimaryKey) 和 [SecondaryKey](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule#Microsoft_ServiceBus_Messaging_SharedAccessAuthorizationRule_SecondaryKey)，並取代成新的金鑰。 此程序會讓所有以舊金鑰簽章的權杖失效。

## <a name="shared-access-signature-authentication-with-service-bus"></a>服務匯流排的共用存取簽章驗證

下方說明的案例包括授權規則的設定、SAS 權杖的產生以及用戶端授權。

如需服務匯流排應用程式的完整工作範例，以便了解相關設定及使用 SAS 授權，請參閱 [共用存取簽章驗證與服務匯流排](http://code.msdn.microsoft.com/Shared-Access-Signature-0a88adf8)。 以下提供相關的範例，說明如何使用在命名空間或主題上設定的 SAS 授權規則來保護服務匯流排訂用帳戶︰ [使用共用存取簽章 (SAS) 驗證與服務匯流排訂用帳](http://code.msdn.microsoft.com/Using-Shared-Access-e605b37c)。

## <a name="access-shared-access-authorization-rules-on-an-entity"></a>存取實體上的共用存取授權規則

藉由服務匯流排 .NET Framework 程式庫，您可以透過對應的 [QueueDescription](/dotnet/api/microsoft.servicebus.messaging.queuedescription) 或 [TopicDescription](/dotnet/api/microsoft.servicebus.messaging.topicdescription) 中的 [AuthorizationRules](/dotnet/api/microsoft.servicebus.messaging.authorizationrules) 集合，存取服務匯流排佇列或主題上設定的 [Microsoft.ServiceBus.Messaging.SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) 物件。

下列程式碼示範如何新增佇列的授權規則。

```csharp
// Create an instance of NamespaceManager for the operation
NamespaceManager nsm = NamespaceManager.CreateFromConnectionString(
    <connectionString> );
QueueDescription qd = new QueueDescription( <qPath> );

// Create a rule with send rights with keyName as "contosoQSendKey"
// and add it to the queue description.
qd.Authorization.Add(new SharedAccessAuthorizationRule("contosoSendKey",
    SharedAccessAuthorizationRule.GenerateRandomKey(),
    new[] { AccessRights.Send }));

// Create a rule with listen rights with keyName as "contosoQListenKey"
// and add it to the queue description.
qd.Authorization.Add(new SharedAccessAuthorizationRule("contosoQListenKey",
    SharedAccessAuthorizationRule.GenerateRandomKey(),
    new[] { AccessRights.Listen }));

// Create a rule with manage rights with keyName as "contosoQManageKey"
// and add it to the queue description.
// A rule with manage rights must also have send and receive rights.
qd.Authorization.Add(new SharedAccessAuthorizationRule("contosoQManageKey",
    SharedAccessAuthorizationRule.GenerateRandomKey(),
    new[] {AccessRights.Manage, AccessRights.Listen, AccessRights.Send }));

// Create the queue.
nsm.CreateQueue(qd);
```

## <a name="use-shared-access-signature-authorization"></a>使用共用存取簽章授權

搭配使用 Azure .NET SDK 與服務匯流排 .NET 程式庫的應用程式可以透過 [SharedAccessSignatureTokenProvider](/dotnet/api/microsoft.servicebus.sharedaccesssignaturetokenprovider) 類別來使用 SAS 授權。 下列程式碼說明如何使用權杖提供者將訊息傳送至服務匯流排佇列。 除了以下說明的用法外，您也可以將先前發行的權杖傳至權杖提供者 Factory 方法。

```csharp
Uri runtimeUri = ServiceBusEnvironment.CreateServiceUri("sb",
    <yourServiceNamespace>, string.Empty);
MessagingFactory mf = MessagingFactory.Create(runtimeUri,
    TokenProvider.CreateSharedAccessSignatureTokenProvider(keyName, key));
QueueClient sendClient = mf.CreateQueueClient(qPath);

//Sending hello message to queue.
BrokeredMessage helloMessage = new BrokeredMessage("Hello, Service Bus!");
helloMessage.MessageId = "SAS-Sample-Message";
sendClient.Send(helloMessage);
```

您也可以直接使用權杖提供者發行權杖，並傳至其他用戶端。 

連接字串可包含規則名稱 (*SharedAccessKeyName*) 和規則金鑰 (*SharedAccessKey*) 或先前發行的權杖 (*SharedAccessSignature*)。 當這些名稱出現在傳至任何建構函式的連接字串中，或是接受連接字串的 Factory 方法中時，系統會自動建立 SAS 權杖提供者並填入。

請注意，若要搭配使用 SAS 授權與服務匯流排轉送，您可以使用在服務匯流排命名空間上設定的 SAS 金鑰。 如果您在命名空間 (具有 [RelayDescription](/dotnet/api/microsoft.servicebus.messaging.relaydescription) 的 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager)) 物件上明確建立轉送，您可以設定該轉送的 SAS 規則。 若要搭配使用 SAS 授權與服務匯流排訂用帳戶，您可以使用在服務匯流排命名空間或主題上設定的 SAS 金鑰。

## <a name="use-the-shared-access-signature-at-http-level"></a>使用共用存取簽章 (於 HTTP 層級)

既然知道如何為服務匯流排中的任何實體建立共用存取簽章，您可準備執行 HTTP POST：

```http
POST https://<yournamespace>.servicebus.windows.net/<yourentity>/messages
Content-Type: application/json
Authorization: SharedAccessSignature sr=https%3A%2F%2F<yournamespace>.servicebus.windows.net%2F<yourentity>&sig=<yoursignature from code above>&se=1438205742&skn=KeyName
ContentType: application/atom+xml;type=entry;charset=utf-8
``` 

請記住，這適用於所有項目。 您可以為佇列、主題或訂用帳戶建立 SAS。

如果您提供寄件者或用戶端 SAS 權杖，他們不會直接有金鑰，並且他們無法回復雜湊來取得它。 因此，您可以控制他們可以存取的項目，以及時間長度。 要記住的重點是，如果您變更原則中的主要金鑰，從該金鑰建立的任何共用存取簽章將會失效。

## <a name="use-the-shared-access-signature-at-amqp-level"></a>使用共用存取簽章 (於 AMQP 層級)

在前一節中，說明了如何使用 SAS 權杖搭配 HTTP POST 要求傳送資料到服務匯流排。 如您所了解，您可以使用進階訊息佇列通訊協定 (AMQP) 來存取服務匯流排，此通訊協定在許多案例中，都是基於效能考量而做為慣用的通訊協定。 如需使用 SAS 權杖搭配 AMQP 的用法，請參閱 [AMQP 宣告型安全性 1.0 版](https://www.oasis-open.org/committees/download.php/50506/amqp-cbs-v1%200-wd02%202013-08-12.doc) 文件中的說明，該文件是自 2013 年開始的工作草稿，不過現在 Azure 已經提供良好的支援。

開始將資料傳送到服務匯流排之前，發行者必須在 AMQP 訊息內部將 SAS 權杖傳送至正確定義且名為 **$cbs** 的 AMQP 節點 (您可以將它視為一個由服務所使用的「特別」佇列，用來取得並驗證所有的 SAS 權杖)。 發行者必須在 AMQP 訊息中指定 **ReplyTo** 欄位；這是服務將以權杖驗證結果 (發行者與服務之間的簡單要求/回覆模式) 回覆發行者的節點所在。 此回覆節點是「動態」建立，如 AMQP 1.0 規格中所述的「動態建立遠端節點」。 檢查 SAS 權杖有效之後，發行者可以繼續並開始將資料傳送至服務。

下列步驟示範如何使用 [AMQP.Net Lite](https://github.com/Azure/amqpnetlite) 程式庫，搭配 AMQP 通訊協定來傳送 SAS 權杖。 如果您不能使用以 C\# 開發的官方服務匯流排 SDK (例如，在 WinRT、.Net Compact Framework、.Net Micro Framework 和 Mono 上)，這就非常有用。 當然，此程式庫對於了解宣告型安全性如何在 AMQP 層級運作非常有用，如同您了解其如何在 HTTP 層級運作一樣 (使用 HTTP POST 要求以及在標頭 "Authorization" 內部傳送的 SAS 權杖)。 如果您不需要深入了解 AMQP，您可以搭配 .Net Framework 應用程式使用正式服務匯流排 SDK，其將為您做到這點。

### <a name="c35"></a>C&#35;

```csharp
/// <summary>
/// Send claim-based security (CBS) token
/// </summary>
/// <param name="shareAccessSignature">Shared access signature (token) to send</param>
private bool PutCbsToken(Connection connection, string sasToken)
{
    bool result = true;
    Session session = new Session(connection);

    string cbsClientAddress = "cbs-client-reply-to";
    var cbsSender = new SenderLink(session, "cbs-sender", "$cbs");
    var cbsReceiver = new ReceiverLink(session, cbsClientAddress, "$cbs");

    // construct the put-token message
    var request = new Message(sasToken);
    request.Properties = new Properties();
    request.Properties.MessageId = Guid.NewGuid().ToString();
    request.Properties.ReplyTo = cbsClientAddress;
    request.ApplicationProperties = new ApplicationProperties();
    request.ApplicationProperties["operation"] = "put-token";
    request.ApplicationProperties["type"] = "servicebus.windows.net:sastoken";
    request.ApplicationProperties["name"] = Fx.Format("amqp://{0}/{1}", sbNamespace, entity);
    cbsSender.Send(request);

    // receive the response
    var response = cbsReceiver.Receive();
    if (response == null || response.Properties == null || response.ApplicationProperties == null)
    {
        result = false;
    }
    else
    {
        int statusCode = (int)response.ApplicationProperties["status-code"];
        if (statusCode != (int)HttpStatusCode.Accepted && statusCode != (int)HttpStatusCode.OK)
        {
            result = false;
        }
    }

    // the sender/receiver may be kept open for refreshing tokens
    cbsSender.Close();
    cbsReceiver.Close();
    session.Close();

    return result;
}
```

上述 `PutCbsToken()` 方法會接收代表服務之 TCP 連線的 *connection* ([AMQP .NET Lite 程式庫](https://github.com/Azure/amqpnetlite)所提供的 AMQP 連接類別執行個體) 以及要做為 SAS 權杖傳送的 sasToken 參數。 

> [!NOTE]
> 請務必以**設為 ANONYMOUS 的 SASL 驗證機制** (而非當您不需要傳送 SAS 權杖時所使用且包含使用者名稱與密碼的預設 PLAIN) 建立連線。
> 
> 

接下來，發行者會建立兩個 AMQP 連結來傳送 SAS 權杖，並接受來自服務的回覆 (權杖驗證結果)。

AMQP 訊息包含眾多屬性，以及比簡單訊息更多的資訊。 SAS 權杖是訊息的內文 (使用其建構函式)。 **"ReplyTo"** 屬性設定為在接收者連結上接收驗證結果的節點名稱 (您可以視需要變更它的名稱，而服務將會動態建立它)。 最後三個應用程式/自訂屬性是由服務所使用，以指出它必須執行何種類型的作業。 如 CBS 草稿規格所述，它們必須是**作業名稱** (像是 "put-token")、**權杖類型** (在本案例中為 `servicebus.windows.net:sastoken`)，以及套用權杖之**對象的 "name"** (完整實體)。

在寄件者連結上傳送 SAS 權杖之後，發行者必須在接收者連結上讀取回覆。 回覆是包含名稱為 **"status-code"** 之應用程式屬性的簡單 AMQP 訊息，可用來包含做為 HTTP 狀態碼的相同值

## <a name="rights-required-for-service-bus-operations"></a>服務匯流排作業所需的權限

下表顯示服務匯流排資源上各種作業所需的存取權限。

| 作業 | 所需的宣告 | 宣告範圍 |
| --- | --- | --- |
| **命名空間** | | |
| 在命名空間上設定授權規則 |管理 |任何命名空間位址 |
| **服務登錄** | | |
| 列舉私人原則 |管理 |任何命名空間位址 |
| 開始在命名空間上接聽 |接聽 |任何命名空間位址 |
| 將訊息傳送至命名空間上的接聽程式 |傳送 |任何命名空間位址 |
| **佇列** | | |
| 建立佇列 |管理 |任何命名空間位址 |
| 刪除佇列 |管理 |任何有效的佇列位址 |
| 列舉佇列 |管理 |/$Resources/Queues |
| 取得佇列描述 |管理 |任何有效的佇列位址 |
| 設定佇列的授權規則 |管理 |任何有效的佇列位址 |
| 傳送到佇列中 |傳送 |任何有效的佇列位址 |
| 從佇列接收訊息 |接聽 |任何有效的佇列位址 |
| 在 peek-lock 模式中接收訊息後放棄或完成訊息 |接聽 |任何有效的佇列位址 |
| 延遲訊息以便稍後擷取 |接聽 |任何有效的佇列位址 |
| 讓訊息寄不出去 |接聽 |任何有效的佇列位址 |
| 取得與訊息佇列工作階段相關聯的狀態 |接聽 |任何有效的佇列位址 |
| 設定與訊息佇列工作階段相關聯的狀態 |接聽 |任何有效的佇列位址 |
| 排程訊息以供日後傳遞，例如，[ScheduleMessageAsync()](/dotnet/api/microsoft.azure.servicebus.queueclient.schedulemessageasync#Microsoft_Azure_ServiceBus_QueueClient_ScheduleMessageAsync_Microsoft_Azure_ServiceBus_Message_System_DateTimeOffset_) |接聽 | 任何有效的佇列位址
| **主題** | | |
| 建立主題 |管理 |任何命名空間位址 |
| 刪除主題 |管理 |任何有效的主題位址 |
| 列舉主題 |管理 |/$Resources/Topics |
| 取得主題描述 |管理 |任何有效的主題位址 |
| 設定主題的授權規則 |管理 |任何有效的主題位址 |
| 傳送至主題 |傳送 |任何有效的主題位址 |
| **訂用帳戶** | | |
| 建立訂用帳戶 |管理 |任何命名空間位址 |
| 刪除訂用帳戶 |管理 |../myTopic/Subscriptions/mySubscription |
| 列舉訂用帳戶 |管理 |../myTopic/Subscriptions |
| 取得訂用帳戶描述 |管理 |../myTopic/Subscriptions/mySubscription |
| 在 peek-lock 模式中接收訊息後放棄或完成訊息 |接聽 |../myTopic/Subscriptions/mySubscription |
| 延遲訊息以便稍後擷取 |接聽 |../myTopic/Subscriptions/mySubscription |
| 讓訊息寄不出去 |接聽 |../myTopic/Subscriptions/mySubscription |
| 取得與主題工作階段相關聯的狀態 |接聽 |../myTopic/Subscriptions/mySubscription |
| 設定與主題工作階段相關聯的狀態 |接聽 |../myTopic/Subscriptions/mySubscription |
| **規則** | | |
| 建立規則 |管理 |../myTopic/Subscriptions/mySubscription |
| 刪除規則 |管理 |../myTopic/Subscriptions/mySubscription |
| 列舉規則 |管理或接聽 |../myTopic/Subscriptions/mySubscription/Rules 

## <a name="next-steps"></a>後續步驟

若要深入了解服務匯流排訊息，請參閱下列主題。

* [服務匯流排佇列、主題和訂用帳戶](service-bus-queues-topics-subscriptions.md)
* [如何使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)
* [如何使用服務匯流排主題和訂用帳戶](service-bus-dotnet-how-to-use-topics-subscriptions.md)

[Azure portal]: https://portal.azure.com