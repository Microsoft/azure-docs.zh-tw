---
title: 租用戶資源使用情況 API | Microsoft Docs
description: 資源使用情況 API (用以擷取 Azure Stack 使用情況資訊) 的參考。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: b9d7c7ee-e906-4978-92a3-a2c52df16c36
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/26/2018
ms.author: mabrigg
ms.reviewer: alfredop
ms.openlocfilehash: 8472d8ce733c07641a7fa6d53aeb6909cd709990
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37048385"
---
# <a name="tenant-resource-usage-api"></a>租用戶資源使用情況 API

租用戶可以使用租用戶 API 來檢視租用戶的資源使用量資料。 此 API 與 Azure 使用情況 API (目前處於私人預覽狀態) 一致。

您可以像在 Azure 中一樣，使用 Windows PowerShell Cmdlet **Get-UsageAggregates** 取得使用量資料。

## <a name="api-call"></a>API 呼叫
### <a name="request"></a>要求
要求會取得所要求的訂用帳戶在要求的時間範圍內的耗用量詳細資料。 沒有要求主體。

| **方法** | **要求 URI** |
| --- | --- |
| GET |https://{armendpoint}/subscriptions/{subId}/providers/Microsoft.Commerce/usageAggregates?reportedStartTime={reportedStartTime}&reportedEndTime={reportedEndTime}&aggregationGranularity={granularity}&api-version=2015-06-01-preview&continuationToken={token-value} |

### <a name="arguments"></a>引數
| **引數** | **說明** |
| --- | --- |
| *Armendpoint* |您 Azure Stack 環境的 Azure Resource Manager 端點。 依 Azure Stack 慣例，Azure Resource Manager 端點名稱的格式是 `https://management.{domain-name}`。 例如，如果是開發套件，則網域名稱會是 local.azurestack.external，而 Resource Manager 端點會是 `https://management.local.azurestack.external`。 |
| *subId* |正在進行呼叫之使用者的訂用帳戶識別碼。 您只能使用此 API 查詢單一訂用帳戶的使用情況。 提供者可以使用提供者資源使用情況 API 查詢所有租用戶的使用情況。 |
| *reportedStartTime* |查詢的開始時間。 *DateTime* 值應該以 UTC 格式及小時開始時的時間呈現，例如 13:00。 對於每日彙總，請將這個值設定為 UTC 午夜。 格式是*逸出的* ISO 8601，例如 2015-06-16T18%3a53%3a11%2b00%3a00Z，其中冒號會逸出為 %3a 而加號會逸出為 %2b，使其符合 URI 規範。 |
| *reportedEndTime* |查詢的結束時間。 適用於 *reportedStartTime* 的限制也適用於此引數。 *reportedEndTime* 的值不能是未來的時間。 |
| *aggregationGranularity* |這是選擇性引數，它可以有兩個截然不同的可能值 (每日及每小時)。 如同以上兩個值所暗示，一個會每日傳回資料，另一個則會每小時傳回資料。 預設值為 [每日] 選項。 |
| *api-version* |用來提出此要求的通訊協定版本。 您必須使用 2015-06-01-preview。 |
| *continuationToken* |從上次呼叫使用情況 API 提供者所擷取的權杖。 回應大於 1000 行時就需要這個權杖，可作為進度的書籤。 若無此權杖，則會從一天或小時開始時的時間擷取資料，取決於所傳遞的細微性。 |

### <a name="response"></a>Response
GET /subscriptions/sub1/providers/Microsoft.Commerce/UsageAggregates?reportedStartTime=reportedStartTime=2014-05-01T00%3a00%3a00%2b00%3a00&reportedEndTime=2015-06-01T00%3a00%3a00%2b00%3a00&aggregationGranularity=Daily&api-version=1.0

```json
{
"value": [
{

"id":
"/subscriptions/sub1/providers/Microsoft.Commerce/UsageAggregate/sub1-meterID1",
"name": "sub1-meterID1",
"type": "Microsoft.Commerce/UsageAggregate",

"properties": {
"subscriptionId":"sub1",
"usageStartTime": "2015-03-03T00:00:00+00:00",
"usageEndTime": "2015-03-04T00:00:00+00:00",
"instanceData":"{\"Microsoft.Resources\":{\"resourceUri\":\"resourceUri1\",\"location\":\"Alaska\",\"tags\":null,\"additionalInfo\":null}}",
"quantity":2.4000000000,
"meterId":"meterID1"

}
},

…
```

### <a name="response-details"></a>回應詳細資料
| **引數** | **說明** |
| --- | --- |
| *id* |使用情況彙總的唯一識別碼 |
| *name* |使用情況彙總的名稱 |
| *type* |資源定義 |
| *subscriptionId* |Azure 使用者的訂用帳戶識別碼 |
| *usageStartTime* |此使用情況彙總所屬的使用情況貯體 UTC 開始時間 |
| *usageEndTime* |此使用情況彙總所屬的使用情況貯體 UTC 結束時間 |
| *instanceData* |執行個體詳細資料的機碼值組 (新格式)：<br>  *resourceUri*：完整資源識別碼，包含資源群組和執行個體名稱 <br>  *location*：此服務執行所在的區域 <br>  *tags*：使用者指定的資源標記 <br>  *additionalInfo*：更多關於所取用資源的詳細資料 (例如，作業系統版本或映像類型) |
| *quantity* |此時間範圍內發生的資源取用數量 |
| *meterId* |所取用資源的唯一識別碼 ( *ResourceID*) |


## <a name="next-steps"></a>後續步驟
[提供者資源使用狀況 API](azure-stack-provider-resource-api.md)

[使用狀況相關常見問題集](azure-stack-usage-related-faq.md)

