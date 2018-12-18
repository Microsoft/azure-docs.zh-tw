---
title: Azure 事件格線訂用帳戶的事件結構描述
description: 描述 Azure 事件格線針對訂用帳戶事件所提供的屬性
services: event-grid
author: tfitzmac
ms.service: event-grid
ms.topic: reference
ms.date: 08/17/2018
ms.author: tomfitz
ms.openlocfilehash: 18f2a64a4354fbd99f1a471c21cc35cbf5df6619
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42140268"
---
# <a name="azure-event-grid-event-schema-for-subscriptions"></a>針對訂用帳戶的 Azure 事件格線事件結構描述

本文提供針對 Azure 訂用帳戶事件的屬性和結構描述。 如需事件結構描述的簡介，請參閱 [Azure 事件格線事件結構描述](event-schema.md)。

Azure 訂用帳戶和資源群組會發出相同的事件類型。 事件類型與資源中的變更相關聯。 主要的差異在於資源群組所發出的事件是針對資源群組內的資源，而 Azure 訂用帳戶發出的事件是針對整個訂用帳戶上的資源。

系統會針對傳送至 `management.azure.com` 的 PUT、PATCH 和 DELETE 作業建立資源事件。 POST 與 GET 作業不會建立事件。 傳送至資料平面作的業 (例如 `myaccount.blob.core.windows.net`) 不會建立事件。

當您訂閱 Azure 訂用帳戶的事件時，您的端點會接收該訂用帳戶的所有事件。 事件可以包含您想要查看的事件，例如更新虛擬機器，也也可以包含對您不重要的事件，例如在部署歷程記錄中寫入新的項目。 您可以在您的端點接收所有事件，以及撰寫程式碼來處理您想要處理的事件，也可以在建立事件訂用帳戶時設定篩選條件。

若要以程式設計方式處理事件，您可藉由查看 `operationName` 值來排序事件。 例如，您的事件端點可能只會處理等於 `Microsoft.Compute/virtualMachines/write` 或 `Microsoft.Storage/storageAccounts/write` 的作業事件。

事件主旨是資源的資源識別碼，而該資源為作業目標。 若要篩選資源的事件，請在建立事件訂用帳戶時提供該資源識別碼。 若要依資源類型進行篩選，請使用以下格式的值：`/subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.Compute/virtualMachines`

如需範例指令碼和教學課程的清單，請參閱 [Azure 訂用帳戶事件來源](event-sources.md#azure-subscriptions)。

## <a name="available-event-types"></a>可用的事件類型

Azure 訂用帳戶會從 Azure Resource Manager 發出管理事件，像是建立 VM 或刪除儲存體帳戶。

| 事件類型 | 說明 |
| ---------- | ----------- |
| Microsoft.Resources.ResourceWriteSuccess | 在資源建立或是更新作業成功時引發。 |
| Microsoft.Resources.ResourceWriteFailure | 在資源建立或是更新作業失敗時引發。 |
| Microsoft.Resources.ResourceWriteCancel | 在資源建立或是更新作業取消時引發。 |
| Microsoft.Resources.ResourceDeleteSuccess | 在資源刪除作業成功時引發。 |
| Microsoft.Resources.ResourceDeleteFailure | 在資源刪除作業失敗時引發。 |
| Microsoft.Resources.ResourceDeleteCancel | 在資源刪除作業取消時引發。 此事件會於範本部署取消時發生。 |

## <a name="example-event"></a>事件範例

下列範例顯示 **ResourceWriteSuccess** 事件的結構描述。 具有不同 `eventType` 值的 **ResourceWriteFailure** 和 **ResourceWriteCancel** 事件使用相同的結構描述。

```json
[{
  "subject": "/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
  "eventType": "Microsoft.Resources.ResourceWriteSuccess",
  "eventTime": "2018-07-19T18:38:04.6117357Z",
  "id": "4db48cba-50a2-455a-93b4-de41a3b5b7f6",
  "data": {
    "authorization": {
      "scope": "/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
      "action": "Microsoft.Storage/storageAccounts/write",
      "evidence": {
        "role": "Subscription Admin"
      }
    },
    "claims": {
      "aud": "{audience-claim}",
      "iss": "{issuer-claim}",
      "iat": "{issued-at-claim}",
      "nbf": "{not-before-claim}",
      "exp": "{expiration-claim}",
      "_claim_names": "{\"groups\":\"src1\"}",
      "_claim_sources": "{\"src1\":{\"endpoint\":\"{URI}\"}}",
      "http://schemas.microsoft.com/claims/authnclassreference": "1",
      "aio": "{token}",
      "http://schemas.microsoft.com/claims/authnmethodsreferences": "rsa,mfa",
      "appid": "{ID}",
      "appidacr": "2",
      "http://schemas.microsoft.com/2012/01/devicecontext/claims/identifier": "{ID}",
      "e_exp": "{expiration}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "{last-name}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "{first-name}",
      "ipaddr": "{IP-address}",
      "name": "{full-name}",
      "http://schemas.microsoft.com/identity/claims/objectidentifier": "{ID}",
      "onprem_sid": "{ID}",
      "puid": "{ID}",
      "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "{ID}",
      "http://schemas.microsoft.com/identity/claims/tenantid": "{ID}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": "{user-name}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "{user-name}",
      "uti": "{ID}",
      "ver": "1.0"
    },
    "correlationId": "{ID}",
    "resourceProvider": "Microsoft.Storage",
    "resourceUri": "/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
    "operationName": "Microsoft.Storage/storageAccounts/write",
    "status": "Succeeded",
    "subscriptionId": "{subscription-id}",
    "tenantId": "{tenant-id}"
  },
  "dataVersion": "2",
  "metadataVersion": "1",
  "topic": "/subscriptions/{subscription-id}"
}]
```

下列範例顯示 **ResourceDeleteSuccess** 事件的結構描述。 具有不同 `eventType` 值的 **ResourceDeleteFailure** 和 **ResourceDeleteCancel** 事件使用相同的結構描述。

```json
[{
  "subject": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
  "eventType": "Microsoft.Resources.ResourceDeleteSuccess",
  "eventTime": "2018-07-19T19:24:12.763881Z",
  "id": "19a69642-1aad-4a96-a5ab-8d05494513ce",
  "data": {
    "authorization": {
      "scope": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
      "action": "Microsoft.Storage/storageAccounts/delete",
      "evidence": {
        "role": "Subscription Admin"
      }
    },
    "claims": {
      "aud": "{audience-claim}",
      "iss": "{issuer-claim}",
      "iat": "{issued-at-claim}",
      "nbf": "{not-before-claim}",
      "exp": "{expiration-claim}",
      "_claim_names": "{\"groups\":\"src1\"}",
      "_claim_sources": "{\"src1\":{\"endpoint\":\"{URI}\"}}",
      "http://schemas.microsoft.com/claims/authnclassreference": "1",
      "aio": "{token}",
      "http://schemas.microsoft.com/claims/authnmethodsreferences": "rsa,mfa",
      "appid": "{ID}",
      "appidacr": "2",
      "http://schemas.microsoft.com/2012/01/devicecontext/claims/identifier": "{ID}",
      "e_exp": "262800",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "{last-name}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "{first-name}",
      "ipaddr": "{IP-address}",
      "name": "{full-name}",
      "http://schemas.microsoft.com/identity/claims/objectidentifier": "{ID}",
      "onprem_sid": "{ID}",
      "puid": "{ID}",
      "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "{ID}",
      "http://schemas.microsoft.com/identity/claims/tenantid": "{ID}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": "{user-name}",
      "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "{user-name}",
      "uti": "{ID}",
      "ver": "1.0"
    },
    "correlationId": "{ID}",
    "httpRequest": {
      "clientRequestId": "{ID}",
      "clientIpAddress": "{IP-address}",
      "method": "DELETE",
      "url": "https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}?api-version=2018-02-01"
    },
    "resourceProvider": "Microsoft.Storage",
    "resourceUri": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}",
    "operationName": "Microsoft.Storage/storageAccounts/delete",
    "status": "Succeeded",
    "subscriptionId": "{subscription-id}",
    "tenantId": "{tenant-id}"
  },
  "dataVersion": "2",
  "metadataVersion": "1",
  "topic": "/subscriptions/{subscription-id}"
}]
```

## <a name="event-properties"></a>事件屬性

事件具有下列的最高層級資料：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| 主題 | 字串 | 事件來源的完整資源路徑。 此欄位不可寫入。 Event Grid 提供此值。 |
| 主旨 | 字串 | 發行者定義事件主體的路徑。 |
| eventType | 字串 | 此事件來源已註冊的事件類型之一。 |
| eventTime | 字串 | 事件產生的時間，以提供者之 UTC 時間為準。 |
| id | 字串 | 事件的唯一識別碼。 |
| data | 物件 | 訂用帳戶事件資料。 |
| dataVersion | 字串 | 資料物件的結構描述版本。 發行者會定義結構描述版本。 |
| metadataVersion | 字串 | 事件中繼資料的結構描述版本。 Event Grid 會定義最上層屬性的結構描述。 Event Grid 提供此值。 |

資料物件具有下列屬性：

| 屬性 | 類型 | 說明 |
| -------- | ---- | ----------- |
| 授權 | 物件 | 作業的所要求授權。 |
| claims | 物件 | 宣告的屬性。 如需詳細資訊，請參閱 [JWT 規格](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html)。 |
| correlationId | 字串 | 用於疑難排解的作業識別碼。 |
| httpRequest | 物件 | 作業的詳細資料。 更新現有資源或刪除資源時，只會包含這個物件。 |
| resourceProvider | 字串 | 執行作業的資源提供者。 |
| resourceUri | 字串 | 作業中資源的 URI。 |
| operationName | 字串 | 已執行的作業。 |
| status | 字串 | 作業狀態。 |
| subscriptionId | 字串 | 資源的訂用帳戶識別碼。 |
| tenantId | 字串 | 資源的租用戶識別碼。 |

## <a name="next-steps"></a>後續步驟

* 若要初步了解 Azure Event Grid，請參閱[什麼是 Event Grid？](overview.md)。
* 若要了解 Event Grid 訂用帳戶的建立，請參閱 [Event Grid 訂用帳戶結構描述](subscription-creation-schema.md)。
