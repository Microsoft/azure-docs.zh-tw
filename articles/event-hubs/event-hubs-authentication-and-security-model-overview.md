---
title: Azure 事件中樞驗證和安全性模型概觀 | Microsoft Docs
description: 事件中樞驗證和安全性模型概觀。
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
manager: timlt
editor: ''
ms.assetid: 93841e30-0c5c-4719-9dc1-57a4814342e7
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/30/2018
ms.author: shvija
ms.openlocfilehash: 484b5109678b04943e59b0e6bc516926f5d61838
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40003150"
---
# <a name="event-hubs-authentication-and-security-model-overview"></a>事件中樞驗證和安全性模型概觀

Azure 事件中樞安全性模型符合下列需求：

* 只有出示有效認證的用戶端才能將資料傳送到事件中樞。
* 一個用戶端無法模擬另一個用戶端。
* 可封鎖惡意用戶端，讓它無法將資料傳送到事件中樞。

## <a name="client-authentication"></a>用戶端驗證

事件中樞安全性模型是以 [共用存取簽章 (SAS)](../service-bus-messaging/service-bus-sas.md) 權杖和「事件發行者」的組合為基礎。 事件發行者會定義事件中樞的虛擬端點。 發行者只能用來將訊息傳送到事件中樞。 您無法接收來自發佈者的訊息。

一般而言，事件中樞會針對每個用戶端採用一個發行者。 系統會將傳送到事件中樞之任何發行者的所有訊息，排入該事件中樞內的佇列中。 發行者會啟用更細緻的存取控制和節流。

系統會為每個「事件中樞」用戶端指派一個唯一權杖，該權杖會上傳到用戶端。 權杖的產生機制讓每個唯一權杖都能授與相異唯一發佈者的存取權限。 擁有權杖的用戶端只能傳送到一個發行者，無法再傳送到任何其他發行者。 如果多個用戶端共用同一個權杖，它們每一個就會共用一個發行者。

您可以讓裝置具備可授與事件中樞直接存取權的權杖，不過並不建議這樣做。 任何擁有此權杖的裝置都可以將訊息直接傳送到該事件中樞。 這類裝置將不受節流所約束。 此外，您也無法將這類裝置加入黑名單，以禁止其傳送到該事件中樞。

所有權杖都經過 SAS 金鑰簽署。 一般而言，所有權杖都會經過同一個金鑰簽署。 用戶端並不知道該金鑰；這可避免其他用戶端製造權杖。

### <a name="create-the-sas-key"></a>建立 SAS 金鑰

建立事件中樞命名空間時，此服務會自動產生名為 **RootManageSharedAccessKey** 的 256 位元 SAS 金鑰。 此規則具有主要和次要金鑰的相關聯配對，可將傳送、接聽和管理權限授與命名空間。 您也可以建立額外的金鑰。 建議您產生一個可授與對特定事件中樞之傳送權限的金鑰。 在本主題的其餘部分，假設您將此金鑰命名為 **EventHubSendKey**。

以下範例會在建立事件中樞時，建立一個僅限傳送的金鑰：

```csharp
// Create namespace manager.
string serviceNamespace = "YOUR_NAMESPACE";
string namespaceManageKeyName = "RootManageSharedAccessKey";
string namespaceManageKey = "YOUR_ROOT_MANAGE_SHARED_ACCESS_KEY";
Uri uri = ServiceBusEnvironment.CreateServiceUri("sb", serviceNamespace, string.Empty);
TokenProvider td = TokenProvider.CreateSharedAccessSignatureTokenProvider(namespaceManageKeyName, namespaceManageKey);
NamespaceManager nm = new NamespaceManager(namespaceUri, namespaceManageTokenProvider);

// Create event hub with a SAS rule that enables sending to that event hub
EventHubDescription ed = new EventHubDescription("MY_EVENT_HUB") { PartitionCount = 32 };
string eventHubSendKeyName = "EventHubSendKey";
string eventHubSendKey = SharedAccessAuthorizationRule.GenerateRandomKey();
SharedAccessAuthorizationRule eventHubSendRule = new SharedAccessAuthorizationRule(eventHubSendKeyName, eventHubSendKey, new[] { AccessRights.Send });
ed.Authorization.Add(eventHubSendRule); 
nm.CreateEventHub(ed);
```

### <a name="generate-tokens"></a>產生權杖

您可以使用 SAS 金鑰來產生權杖。 您只能為每個用戶端產生一個權杖。 您可以使用下列方法來產生權杖。 所有權杖都是以 **EventHubSendKey** 金鑰產生。 每個權杖均有一個指派的唯一 URI。

```csharp
public static string SharedAccessSignatureTokenProvider.GetSharedAccessSignature(string keyName, string sharedAccessKey, string resource, TimeSpan tokenTimeToLive)
```

在呼叫這個方法時，您應該將 URI 指定為 `//<NAMESPACE>.servicebus.windows.net/<EVENT_HUB_NAME>/publishers/<PUBLISHER_NAME>`。 此 URI 對所有權杖來說是完全相同的，但 `PUBLISHER_NAME`除外，每個權杖的該項目應該是不同的。 在理想的情況下，`PUBLISHER_NAME` 代表收到該權杖之用戶端的識別碼。

這個方法會產生具有下列結構的權杖：

```csharp
SharedAccessSignature sr={URI}&sig={HMAC_SHA256_SIGNATURE}&se={EXPIRATION_TIME}&skn={KEY_NAME}
```

以秒數為單位指定的權杖到期 (從 1970 年 1 月 1 日 開始)。 以下是權杖的範例：

```csharp
SharedAccessSignature sr=contoso&sig=nPzdNN%2Gli0ifrfJwaK4mkK0RqAB%2byJUlt%2bGFmBHG77A%3d&se=1403130337&skn=RootManageSharedAccessKey
```

一般而言，權杖的週期與用戶端的週期相近或更長。 如果用戶端能夠取得新的權杖，就可以使用週期較短的權杖。

### <a name="sending-data"></a>傳送資料

建立權杖之後，系統就會為每個用戶端佈建其自己的唯一權杖。

當用戶端將資料傳送到事件中樞時，會使用權杖標記自己傳送的要求。 為了防止攻擊者竊聽及竊取權杖，用戶端與事件中樞之間的通訊必須透過已加密的通道進行。

### <a name="blacklisting-clients"></a>將用戶端列入黑名單

如果權杖遭攻擊者竊取，攻擊者便可以模擬權杖遭竊的用戶端。 將用戶端列入黑名單可讓用戶端變成無法使用，直到它收到使用不同發行者的新權杖為止。

## <a name="authentication-of-back-end-applications"></a>後端應用程式的驗證

為了驗證取用「事件中樞」用戶端所產生之資料的後端應用程式，「事件中樞」採用一個與「服務匯流排」主題所用模型相似的安全性模型。 事件中樞取用者群組等同於服務匯流排主題的訂用帳戶。 如果建立取用者群組的要求有附帶權杖，而該權杖可授與事件中樞或事件中樞所屬之命名空間的管理權限，用戶端便可以建立取用者群組。 如果接收要求有附帶權杖，而該權杖可授與該取用者群組、事件中樞或事件中樞所屬之命名空間的接收權限，用戶端便可以取用來自取用者群組的資料。

目前版本的服務匯流排不支援個別訂用帳戶的 SAS 規則。 相同情況亦適用於事件中樞取用者群組。 我們會在日後將這兩項功能加入 SAS 支援。

由於個別取用者群組的 SAS 驗證不存在，因此您可以使用 SAS 金鑰來保護所有使用相同金鑰的取用者群組。 這個方法可讓應用程式取用來自事件中樞之任何取用者群組的資料。

## <a name="next-steps"></a>後續步驟

若要深入了解事件中樞，請造訪下列主題：

* [事件中心概觀]
* [共用存取簽章的概觀]
* [使用事件中樞的完整範例應用程式]

[事件中心概觀]: event-hubs-what-is-event-hubs.md
[使用事件中樞的完整範例應用程式]: https://github.com/Azure/azure-event-hubs/tree/master/samples
[共用存取簽章的概觀]: ../service-bus-messaging/service-bus-sas.md

