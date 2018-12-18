---
title: 使用 REST API 取得 Azure 資源健康狀態事件 | Microsoft Docs
description: 使用 Azure REST API 取得 Azure 資源的健康狀態事件。
services: Resource health
author: rloutlaw
ms.reviewer: routlaw
manager: angerobe
ms.service: service-health
ms.custom: REST
ms.topic: article
ms.date: 06/06/2017
ms.author: routlaw
ms.openlocfilehash: bbbaa4c44a7c0d6da189f0c49d73adfa6142cdee
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47095786"
---
# <a name="get-resource-health-using-the-rest-api"></a>使用 REST API 取得資源健康狀態 

此文章範例示範如何在訂用帳戶中使用 [Azure REST API](/rest/api/azure/)，擷取 Azure 資源的健康狀態事件清單。

REST API 的完整參考文件和其他範例可於 [Azure 監視器 REST 參考](/rest/api/monitor)中取得。 

## <a name="build-the-request"></a>建立要求

使用下列 `GET` HTTP 要求，針對 `2018-05-16` 和 `2018-06-20` 之間的時間範圍，列出訂用帳戶的健康狀態事件。

```http
https://management.azure.com/subscriptions/{subscription-id}/providers/microsoft.insights/eventtypes/management/values?api-version=2015-04-01&%24filter=eventTimestamp%20ge%20'2018-05-16T04%3A36%3A37.6407898Z'%20and%20eventTimestamp%20le%20'2018-06-20T04%3A36%3A37.6407898Z'
```

### <a name="request-headers"></a>要求標頭

以下是必要標頭： 

|要求標頭|說明|  
|--------------------|-----------------|  
|*Content-Type:*|必要。 設定為 `application/json`。|  
|*Authorization:*|必要。 設定為無效的 `Bearer` [存取權杖](/rest/api/azure/#authorization-code-grant-interactive-clients)。 |  

### <a name="uri-parameters"></a>URI 參數

| 名稱 | 說明 |
| :--- | :---------- |
| subscriptionId | 可識別 Azure 訂用帳戶的訂用帳戶識別碼。 如果您有多個訂用帳戶，請參閱[使用多個訂用帳戶](https://docs.microsoft.com/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest#working-with-multiple-subscriptions)。 |
| api-version | 要用於要求的 API 版本。<br /><br /> 此文件涵蓋 api-version `2015-04-01`，內含於上述 URL 中。  |
| $filter | 可減少傳回之結果集的篩選選項。 此參數允許的模式可用於[活動記錄作業的參考](/rest/api/monitor/activitylogs/list#uri-parameters)。 顯示的範例將會擷取 2018 年 5 月 16 日到 2018 年 6 月 20 日之間的時間範圍內的所有事件 |
| &nbsp; | &nbsp; |

### <a name="request-body"></a>Request body

此作業不需要要求本文。

## <a name="handle-the-response"></a>處理回應

傳回狀態碼 200，以及對應至篩選參數之健康狀態事件值與 `nextlink` URI 的清單，以擷取下一頁的結果。

## <a name="example-response"></a>範例回應 

```json
{
  "value": [
    {
      "correlationId": "1e121103-0ba6-4300-ac9d-952bb5d0c80f",
      "eventName": {
        "value": "EndRequest",
        "localizedValue": "End request"
      },
      "id": "/subscriptions/{subscription-id}/resourceGroups/MSSupportGroup/providers/microsoft.support/supporttickets/115012112305841/events/44ade6b4-3813-45e6-ae27-7420a95fa2f8/ticks/635574752669792776",
      "resourceGroupName": "MSSupportGroup",
      "resourceProviderName": {
        "value": "microsoft.support",
        "localizedValue": "microsoft.support"
      },
      "operationName": {
        "value": "microsoft.support/supporttickets/write",
        "localizedValue": "microsoft.support/supporttickets/write"
      },
      "status": {
        "value": "Succeeded",
        "localizedValue": "Succeeded"
      },
      "eventTimestamp": "2015-01-21T22:14:26.9792776Z",
      "submissionTimestamp": "2015-01-21T22:14:39.9936304Z",
      "level": "Informational"
    }
  ],
  "nextLink": "https://management.azure.com/########-####-####-####-############$skiptoken=######"
}
```