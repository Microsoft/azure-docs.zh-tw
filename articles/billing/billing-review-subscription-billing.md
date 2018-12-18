---
title: 使用 REST API 檢閱 Azure 訂用帳戶計費資料 | Microsoft Docs
description: 了解如何使用 Azure REST API 檢閱訂用帳戶計費詳細資料。
services: billing
documentationcenter: na
author: lleonard-msft
manager: MBaldwin
editor: ''
ms.assetid: 82D50B98-40F2-44B1-A445-4391EA9EBBAA
ms.service: billing
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/06/2018
ms.author: erikre
ms.openlocfilehash: d0199711370dae6a23626337946e45e84fd33047
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47394710"
---
# <a name="review-subscription-billing-using-rest-apis"></a>使用 REST API 檢閱訂用帳戶計費

Azure 報告 API 可協助您檢閱及管理 Azure 費用。  

篩選有助於自訂結果來符合您的需求。

在本文中，您將了解如何使用 REST API 傳回指定日期範圍的訂用帳戶計費詳細資料。

``` http
GET https://management.azure.com/subscriptions/${subscriptionID}/providers/Microsoft.Billing/billingPeriods/${billingPeriod}/providers/Microsoft.Consumption/usageDetails?$filter=properties/usageEnd ge '${startDate}' AND properties/usageEnd le '${endDate}'
Content-Type: application/json   
Authorization: Bearer
```

## <a name="build-the-request"></a>建立要求  

`{subscriptionID}` 是必要參數，用於識別目標訂用帳戶。

`{billingPeriod}` 是必要參數，用於指定目前的[計費週期](https://docs.microsoft.com/rest/api/billing/billingperiods/get#billingperiod)。

`${startDate}` 和 `${endDate}` 在此範例中為必要參數，但對於端點則非必要。  它以 YYYY-MM-DD 格式的字串來指定日期範圍 (例如：`'20180501'` 和 `'20180615'`)。 

以下是必要標頭： 

|要求標頭|說明|  
|--------------------|-----------------|  
|*Content-Type:*|必要。 設定為 `application/json`。|  
|*Authorization:*|必要。 設定為無效的 `Bearer` [存取權杖](https://docs.microsoft.com/rest/api/azure/#authorization-code-grant-interactive-clients)。 |  

## <a name="response"></a>Response  

傳回狀態碼 200 (確定) 表示成功回應，其中包含您帳戶的詳細費用清單。

``` json
{
  "value": [
    {
      "id": "/subscriptions/{$subscriptionID}/providers/Microsoft.Billing/billingPeriods/201702/providers/Microsoft.Consumption/usageDetails/{$detailsID}",
      "name": "{$detailsID}",
      "type": "Microsoft.Consumption/usageDetails",
      "properties": {
        "billingPeriodId": "/subscriptions/${subscriptionID}/providers/Microsoft.Billing/billingPeriods/${billingPeriod}",
        "invoiceId": "/subscriptions/${subscriptionID}/providers/Microsoft.Billing/invoices/${invoiceID}",
        "usageStart": "${startDate}}",
        "usageEnd": "${endDate}",
        "currency": "USD",
        "usageQuantity": ${usageQuantity},
        "billableQuantity": ${billableQuantity},
        "pretaxCost": ${cost},
        "meterId": "${meterID}",
        "meterDetails": ${meterDetails}
      }
    }
    ],
    "nextLink": "${nextLinkURL}"
} 
```  

[值] 中的每個項目代表與使用服務相關的詳細資料：

|回應屬性|說明|
|----------------|----------|
|**subscriptionGuid** | 訂用帳戶的全域唯一識別碼。 | 
|**startDate** | 開始使用的日期。 |
|**endDate** | 結束使用的日期。 |
|**useageQuantity** | 使用的數量。 | 
|**billableQuantity** | 實際計費的數量。 |
|**pretaxCost** | 加上適用稅金之前的發票費用。 | 
|**meterDetails** | 使用的相關詳細資訊。 |
|**nextLink**| 設定時，指定「下一頁」詳細資料的 URL。 如果該頁面是最後一頁，則為空白。 |  
||
  
此範例已經過縮減；如需每個回應欄位的完整描述，請參閱 [List usage details](https://docs.microsoft.com/rest/api/consumption/usagedetails/listbybillingperiod#usagedetailslistresult) (列出使用情況詳細資料)。 

其他狀態碼表示錯誤狀況。 在這些情況下，回應物件會說明要求為何失敗。

``` json
{  
  "error": [  
    { "code": "Error type." 
      "message": "Error response describing why the operation failed."  
    }  
  ]  
}  
```  

## <a name="next-steps"></a>後續步驟 
- 檢閱[企業報告概觀](https://docs.microsoft.com/azure/billing/billing-enterprise-api)
- 調查[企業計費 REST API](https://docs.microsoft.com/rest/api/billing/)   
- [Get started with Azure REST API](https://docs.microsoft.com/rest/api/azure/) (開始使用 Azure REST API)   
