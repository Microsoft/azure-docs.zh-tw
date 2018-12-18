---
title: Azure Event Grid 安全性與驗證
description: 說明 Azure Event Grid 與其概念。
services: event-grid
author: banisadr
manager: timlt
ms.service: event-grid
ms.topic: conceptual
ms.date: 08/13/2018
ms.author: babanisa
ms.openlocfilehash: 257f7cbd20d21903f4cf7daf68b5f185d0af10bc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46965446"
---
# <a name="event-grid-security-and-authentication"></a>Event Grid 安全性與驗證 

Azure Event Grid 有三種驗證方法：

* 事件訂閱
* 事件發佈
* WebHook 事件傳遞

## <a name="webhook-event-delivery"></a>WebHook 事件傳遞

Webhook 是從 Azure 事件方格接收事件的眾多方法之一。 當新的事件準備好時，EventGrid 服務就會對設定的端點發佈 HTTP 要求，並在要求本文內夾帶事件。

和其他許多支援 Webhook 的服務一樣，EventGrid 也會先要求您證明 Webhook 端點的「擁有權」，才開始將事件傳遞至該端點。 此要求是為了防止非預期的端點成為從 EventGrid 傳遞事件的目標端點。 不過，當您使用下列三種 Azure 服務的任一種時，Azure 基礎結構將會自動處理這項驗證：

* Azure Logic Apps、
* Azure 自動化、
* 適用於 EventGrid 觸發程序的 Azure Functions。

如果您使用任何其他類型的端點 (例如，以 HTTP 觸發程序為基礎的 Azure 函式)，則您的端點程式碼必須參與和 EventGrid 的驗證交握。 EventGrid 支援兩種不同的驗證交握模型：

1. **ValidationCode 交握**：在事件訂閱建立期間，EventGrid 會將「訂閱驗證事件」發佈至您的端點。 此事件的結構描述類似於其他 EventGridEvent，而此事件的資料部分會包含 `validationCode` 屬性。 當您的應用程式確認驗證要求是針對預期的事件訂閱而發出時，應用程式的程式碼必須將驗證碼傳回至 EventGrid，作為回應。 所有 EventGrid 版本都支援此交握機制。

2. **ValidationURL 交握 (手動交握)**：在某些情況下，您可能沒有端點原始程式碼的控制權，而無法實作以 ValidationCode 為基礎的交握。 例如，如果您使用第三方服務 (例如 [Zapier](https://zapier.com) 或 [IFTTT](https://ifttt.com/))，就可能無法以程式設計方式在回應中提供驗證碼。 因此，從 2018-05-01-preview 版開始，EventGrid 已可支援手動驗證交握。 如果您以採用這個新 API 版本 (2018-05-01-preview) 的 SDK/工具建立事件訂閱，EventGrid 會傳送 `validationUrl` 屬性 (除了 `validationCode` 屬性) 作為訂閱驗證事件資料部分的一部分。 若要完成交握，您只需透過 REST 用戶端或使用網頁瀏覽器，對該 URL 執行 GET 要求即可。 提供的驗證 URL 的有效期大約只有 10 分鐘。 在那段時間，事件訂閱的佈建狀態為 `AwaitingManualAction`。 如果您未在 10 分鐘內完成手動驗證，則會將佈建狀態設為 `Failed`。 您將必須重新嘗試建立事件訂閱，然後再次嘗試執行手動驗證。

此一手動驗證機制目前為預覽狀態。 若要使用它，您必須為 [Azure CLI](/cli/azure/install-azure-cli) 安裝[事件格線延伸模組](/cli/azure/azure-cli-extensions-list)。 您可以使用 `az extension add --name eventgrid` 進行安裝。 如果您使用的是 REST API，請確定使用的是 `api-version=2018-05-01-preview`。

### <a name="validation-details"></a>驗證詳細資料

* 在事件訂閱建立/更新時，事件方格會將訂閱驗證事件發佈至目標端點。 
* 事件包含標頭值 "aeg-event-type: SubscriptionValidation"。
* 此事件主體之結構描述與其他 Event Grid 事件相同。
* 事件的 eventType 屬性為 "Microsoft.EventGrid.SubscriptionValidationEvent"。
* 事件的資料屬性包含 "validationCode" 屬性，內含隨機產生的字串。 例如，"validationCode: acb13…"。
* 如果您使用 2018-05-01-preview API 版本，事件資料也會包含具有 URL 的 `validationUrl` 屬性，以供手動驗證訂閱之用。
* 此陣列只包含驗證事件。 在您回應驗證程式碼之後，其他事件會在不同的要求中傳送。
* EventGrid DataPlane SDK 具有對應於訂閱驗證事件資料和訂閱驗證回應的類別。

下列範例顯示 SubscriptionValidationEvent 的範例：

```json
[{
  "id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66",
  "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "subject": "",
  "data": {
    "validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6",
    "validationUrl": "https://rp-eastus2.eventgrid.azure.net:553/eventsubscriptions/estest/validate?id=B2E34264-7D71-453A-B5FB-B62D0FDC85EE&t=2018-04-26T20:30:54.4538837Z&apiVersion=2018-05-01-preview&token=1BNqCxBBSSE9OnNSfZM4%2b5H9zDegKMY6uJ%2fO2DFRkwQ%3d"
  },
  "eventType": "Microsoft.EventGrid.SubscriptionValidationEvent",
  "eventTime": "2018-01-25T22:12:19.4556811Z",
  "metadataVersion": "1",
  "dataVersion": "1"
}]
```

如需證明端點擁有權，請在 validationResponse 內容中回應驗證代碼，如下列範例所示：

```json
{
  "validationResponse": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"
}
```

或者，您可以藉由將 GET 要求傳送至驗證 URL，以手動方式驗證訂閱。 事件訂用帳戶會保持擱置狀態，直到通過驗證為止。

您可以在 https://github.com/Azure-Samples/event-grid-dotnet-publish-consume-events/blob/master/EventGridConsumer/EventGridConsumer/Function1.cs 找到說明如何處理訂閱驗證交握的 C# 範例。

### <a name="checklist"></a>檢查清單

在事件訂閱建立期間，如果您看到「嘗試驗證提供的端點 https://your-endpoint-here 失敗。 如需詳細資訊，請瀏覽 https://aka.ms/esvalidation」之類的錯誤訊息，即表示驗證交握失敗。 若要解決此錯誤，請執行以下幾方面的驗證：

* 您是否有權控制目標端點中的應用程式程式碼？ 例如，如果您要撰寫以 HTTP 觸發程序為基礎的 Azure 函式，您是否能夠存取應用程式的程式碼而加以變更？
* 如果您有應用程式程式碼的存取權，請實作以 ValidationCode 為基礎的交握機制，如上述範例所示。

* 如果您沒有應用程式程式碼的存取權 (例如，如果您使用支援 Webhook 的第三方服務)，您可以使用手動交握機制。 若要這樣做，請確定您使用的是 2018-05-01-preview API 版本 (例如，使用前述的 EventGrid CLI 擴充功能)，以接收驗證事件中的 validationUrl。 若要完成手動驗證交握，請取得 "validationUrl" 屬性的值，並在網頁瀏覽器中瀏覽該 URL。 如果驗證成功，您應該會在網頁瀏覽器中看到驗證成功的訊息，且您會看到該事件訂閱的 provisioningState 顯示為「成功」。 

### <a name="event-delivery-security"></a>事件傳遞安全性

您可以在建立事件訂閱時將查詢參數新增至 Webhook URL，以保護您的 Webhook 端點。 將其中一個查詢參數設定為祕密，例如[存取權杖](https://en.wikipedia.org/wiki/Access_token)，Webhook 可用其來辨識事件是否來自具有效權限的事件格線。 事件格線會在傳遞至 Webhook 的每個事件中包含這些查詢參數。

編輯事件訂閱時，除非在 Azure [CLI](https://docs.microsoft.com/cli/azure?view=azure-cli-latest) 中使用 [--include-full-endpoint-url](https://docs.microsoft.com/cli/azure/eventgrid/event-subscription?view=azure-cli-latest#az-eventgrid-event-subscription-show) 參數，否則不會顯示或傳回查詢參數。

最後請務必注意，Azure Event Grid 只支援 HTTPS Webhook 端點。

## <a name="event-subscription"></a>事件訂閱

要訂閱事件，您必須擁有所需資源的 **Microsoft.EventGrid/EventSubscriptions/Write** 授權。 因為您要在該資源的範圍下寫入新的訂用帳戶，所以需要授權。 依您訂閱的是系統主題或自訂主題而定，所需資源會有所不同。 本節會說明這兩種類型。

### <a name="system-topics-azure-service-publishers"></a>系統主題 (Azure 服務發行者)

若您訂閱的是系統主題，您需要取得在資源發佈範圍下寫入新事件訂閱的權限。 資源的格式為：`/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{resource-provider}/{resource-type}/{resource-name}`

舉例來說，若要訂閱名為 **myacct** 之儲存體帳戶上的事件，您需要 Microsoft.EventGrid/EventSubscriptions/Write 授予您此權限：`/subscriptions/####/resourceGroups/testrg/providers/Microsoft.Storage/storageAccounts/myacct`

### <a name="custom-topics"></a>自訂主題

若是自訂主題，您需要取得在 Event Grid 主題範圍下寫入新事件訂用帳戶的權限。 資源的格式為：`/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.EventGrid/topics/{topic-name}`

舉例來說，若要訂閱名為**mytopic** 之自訂主題，您需要 Microsoft.EventGrid/EventSubscriptions/Write 授予您此權限：`/subscriptions/####/resourceGroups/testrg/providers/Microsoft.EventGrid/topics/mytopic`

## <a name="topic-publishing"></a>主題發佈

主題使用共用存取簽章 (SAS) 或金鑰驗證。 我們建議使用 SAS，但金鑰驗證提供簡單的程式編寫，並且與許多現有的 Webhook 發佈者相容。 

您要在 HTTP 標題包含驗證值。 若選擇 SAS，請使用 **aeg-sas-token** 作為標題的值。 若選擇金鑰驗證，請使用 **aeg-sas-key** 作為標題的值。

### <a name="key-authentication"></a>金鑰驗證

金鑰驗證是最簡單的驗證方法。 請使用此格式：`aeg-sas-key: <your key>`

舉例來說，您將金鑰與此一起傳遞：

```
aeg-sas-key: VXbGWce53249Mt8wuotr0GPmyJ/nDT4hgdEj9DpBeRr38arnnm5OFg==
```

### <a name="sas-tokens"></a>SAS 權杖

Event Grid 的 SAS 權杖包含資源、過期時間及簽章。 SAS 權杖的格式為：`r={resource}&e={expiration}&s={signature}`。

這裡的資源是您傳送事件之目標 Event Grid 主題的路徑。 以下為一個有效資源路徑的例子：`https://<yourtopic>.<region>.eventgrid.azure.net/eventGrid/api/events`

簽章由您從金鑰產生。

以下為一個有效 **aeg-sas-token** 值的例子：

```http
aeg-sas-token: r=https%3a%2f%2fmytopic.eventgrid.azure.net%2feventGrid%2fapi%2fevent&e=6%2f15%2f2017+6%3a20%3a15+PM&s=a4oNHpRZygINC%2fBPjdDLOrc6THPy3tDcGHw1zP4OajQ%3d
```

下列範例建立了供 Event Grid 使用的 SAS 權杖：

```cs
static string BuildSharedAccessSignature(string resource, DateTime expirationUtc, string key)
{
    const char Resource = 'r';
    const char Expiration = 'e';
    const char Signature = 's';

    string encodedResource = HttpUtility.UrlEncode(resource);
    var culture = CultureInfo.CreateSpecificCulture("en-US");
    var encodedExpirationUtc = HttpUtility.UrlEncode(expirationUtc.ToString(culture));

    string unsignedSas = $"{Resource}={encodedResource}&{Expiration}={encodedExpirationUtc}";
    using (var hmac = new HMACSHA256(Convert.FromBase64String(key)))
    {
        string signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(unsignedSas)));
        string encodedSignature = HttpUtility.UrlEncode(signature);
        string signedSas = $"{unsignedSas}&{Signature}={encodedSignature}";

        return signedSas;
    }
}
```

## <a name="management-access-control"></a>Management Access Control

Azure Event Grid 讓您能控制給予不同使用者進行各種管理作業的存取層級，這些作業包含列出事件訂閱、建立新訂閱及產生金鑰等等。 Event Grid 會使用 Azure 的角色型存取檢查 (RBAC)。

### <a name="operation-types"></a>作業類型

Azure Event Grid 支援下列動作：

* Microsoft.EventGrid/*/read
* Microsoft.EventGrid/*/write
* Microsoft.EventGrid/*/delete
* Microsoft.EventGrid/eventSubscriptions/getFullUrl/action
* Microsoft.EventGrid/topics/listKeys/action
* Microsoft.EventGrid/topics/regenerateKey/action

最後三種作業可能會回傳秘密資訊，這種資訊在一般讀取作業時是會被篩選掉的。 最好的做法是限制這幾項作業的存取。 自訂角色可以使用 [Azure PowerShell](../role-based-access-control/role-assignments-powershell.md)、[Azure 命令列介面 (CLI)](../role-based-access-control/role-assignments-cli.md) 和 [REST API](../role-based-access-control/role-assignments-rest.md) 建立。

### <a name="enforcing-role-based-access-check-rbac"></a>強制執行 Role Based Access Check (RBAC)

進行下列步驟以強制不同使用者執行 RBAC：

#### <a name="create-a-custom-role-definition-file-json"></a>建立自訂的角色定義檔案 (.json)

以下是允許使用者進行不同組事件的 Event Grid 角色定義範例。

**EventGridReadOnlyRole.json**：只允許唯讀作業。

```json
{
  "Name": "Event grid read only role",
  "Id": "7C0B6B59-A278-4B62-BA19-411B70753856",
  "IsCustom": true,
  "Description": "Event grid read only role",
  "Actions": [
    "Microsoft.EventGrid/*/read"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<Subscription Id>"
  ]
}
```

**EventGridNoDeleteListKeysRole.json**：允許限制的張貼動作，但不允許刪除。

```json
{
  "Name": "Event grid No Delete Listkeys role",
  "Id": "B9170838-5F9D-4103-A1DE-60496F7C9174",
  "IsCustom": true,
  "Description": "Event grid No Delete Listkeys role",
  "Actions": [
    "Microsoft.EventGrid/*/write",
    "Microsoft.EventGrid/eventSubscriptions/getFullUrl/action"
    "Microsoft.EventGrid/topics/listkeys/action",
    "Microsoft.EventGrid/topics/regenerateKey/action"
  ],
  "NotActions": [
    "Microsoft.EventGrid/*/delete"
  ],
  "AssignableScopes": [
    "/subscriptions/<Subscription id>"
  ]
}
```

**EventGridContributorRole.json**：允許所有 Grid 動作。

```json
{
  "Name": "Event grid contributor role",
  "Id": "4BA6FB33-2955-491B-A74F-53C9126C9514",
  "IsCustom": true,
  "Description": "Event grid contributor role",
  "Actions": [
    "Microsoft.EventGrid/*/write",
    "Microsoft.EventGrid/*/delete",
    "Microsoft.EventGrid/topics/listkeys/action",
    "Microsoft.EventGrid/topics/regenerateKey/action",
    "Microsoft.EventGrid/eventSubscriptions/getFullUrl/action"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/<Subscription id>"
  ]
}
```

#### <a name="create-and-assign-custom-role-with-azure-cli"></a>使用 Azure CLI 建立並指派自訂角色

若要建立自訂角色，請使用：

```azurecli
az role definition create --role-definition @<file path>
```

若要指派角色給使用者，請使用：

```azurecli
az role assignment create --assignee <user name> --role "<name of role>"
```

## <a name="next-steps"></a>後續步驟

* 如需 Event Grid 的簡介，請參閱[關於 Event Grid](overview.md)
