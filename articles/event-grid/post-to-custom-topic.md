---
title: 張貼事件以自訂 Azure Event Grid 主題
description: 本文說明如何將事件張貼到自訂主題。 它會顯示貼文與事件資料的格式。
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: ed126487938e524264c94544903460854ffc4d41
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98681595"
---
# <a name="post-to-custom-topic-for-azure-event-grid"></a>針對 Azure Event Grid 張貼到自訂主題

本文說明如何將事件張貼到自訂主題。 它會顯示貼文與事件資料的格式。 [服務等級協定 (SLA)](https://azure.microsoft.com/support/legal/sla/event-grid/v1_0/) 只適用於符合預期格式的貼文。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="endpoint"></a>端點

將 HTTP POST 傳送至自訂主題時，請使用 URI 格式：`https://<topic-endpoint>?api-version=2018-01-01`。

例如，`https://exampletopic.westus2-1.eventgrid.azure.net/api/events?api-version=2018-01-01` 便是有效的 URI。

若要使用 Azure CLI 取得自訂主題的端點，請使用：

```azurecli-interactive
az eventgrid topic show --name <topic-name> -g <topic-resource-group> --query "endpoint"
```

若要使用 Azure PowerShell 取得自訂主題的端點，請使用：

```powershell
(Get-AzEventGridTopic -ResourceGroupName <topic-resource-group> -Name <topic-name>).Endpoint
```

## <a name="header"></a>標頭

在要求中，包含名為 `aeg-sas-key` 的標頭值，其中包含用於驗證的金鑰。

例如，有效的標頭值為 `aeg-sas-key: VXbGWce53249Mt8wuotr0GPmyJ/nDT4hgdEj9DpBeRr38arnnm5OFg==`。

若要使用 Azure CLI 取得自訂主題的索引鍵，請使用：

```azurecli
az eventgrid topic key list --name <topic-name> -g <topic-resource-group> --query "key1"
```

若要使用 PowerShell 取得自訂主題的索引鍵，請使用：

```powershell
(Get-AzEventGridTopicKey -ResourceGroupName <topic-resource-group> -Name <topic-name>).Key1
```

## <a name="event-data"></a>事件資料

對於自訂主題，最上層的資料包含與標準資源定義事件相同的欄位。 其中一個屬性是資料屬性，其中包含自訂主題特有的屬性。 身為事件發行者，您必須判斷該資料物件的屬性。 使用下列結構描述：

```json
[
  {
    "id": string,    
    "eventType": string,
    "subject": string,
    "eventTime": string-in-date-time-format,
    "data":{
      object-unique-to-each-publisher
    },
    "dataVersion": string
  }
]
```

如需這些屬性的說明，請參閱 [Azure Event Grid 事件結構描述](event-schema.md)。 張貼事件到事件方格主題時，陣列總大小最大為 1 MB。 事件允許的大小上限也是 1 MB。 超過 64 KB 的事件會依 64-KB 的遞增計費。 

例如，有效的事件資料結構描述為：

```json
[{
  "id": "1807",
  "eventType": "recordInserted",
  "subject": "myapp/vehicles/motorcycles",
  "eventTime": "2017-08-10T21:03:07+00:00",
  "data": {
    "make": "Ducati",
    "model": "Monster"
  },
  "dataVersion": "1.0"
}]
```

## <a name="response"></a>回應

在張貼到主題端點之後，您會收到回應。 回應是標準的 HTTP 回應碼。 一些常見回應有：

|結果  |回應  |
|---------|---------|
|Success  | 200 確定  |
|事件資料的格式不正確 | 400 不正確的要求 |
|存取金鑰無效 | 401 未經授權 |
|端點不正確 | 404 找不到 |
|陣列或事件超過大小限制 | 413 承載太大 |

對於錯誤，訊息內文的格式如下：

```json
{
    "error": {
        "code": "<HTTP status code>",
        "message": "<description>",
        "details": [{
            "code": "<HTTP status code>",
            "message": "<description>"
    }]
  }
}
```

## <a name="next-steps"></a>後續步驟

* 如需關於監視事件傳遞的資訊，請參閱[監視 Event Grid 訊息傳遞](monitor-event-delivery.md)。
* 如需驗證金鑰的詳細資訊，請參閱 [Event Grid 安全性和驗證](security-authentication.md)。
* 若要了解 Event Grid 訂用帳戶的建立，請參閱 [Event Grid 訂用帳戶結構描述](subscription-creation-schema.md)。
