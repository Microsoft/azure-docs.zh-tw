---
title: 使用 REST API 檢閱 Azure Enterprise 註冊計費資料 | Microsoft Docs
description: 了解如何使用 Azure REST API 來檢閱 Enterprise 註冊計費資訊。
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
ms.author: alleonar
ms.openlocfilehash: 71143549916fc7440d5f21bcb03f1f795ddc73ac
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42141282"
---
# <a name="review-enterprise-enrollment-billing-using-rest-apis"></a>使用 REST API 檢閱 Enterprise 註冊計費

Azure 報告 API 可協助您檢閱及管理 Azure 費用。

在本文中，您會了解如何使用 Azure REST API，來擷取與計費帳戶、部門或 Enterprise 合約 (EA) 註冊帳戶相關聯的帳單資訊。 

## <a name="individual-account-billing"></a>個別帳戶的帳單

若要取得某部門帳戶的使用量詳細資料：

```http
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/providers/Microsoft.Consumption/usageDetails?api-version=2018-06-30
Content-Type: application/json   
Authorization: Bearer
```

`{billingAccountId}` 是必要參數，其中應該包含帳戶的識別碼。

以下是必要標頭： 

|要求標頭|說明|  
|--------------------|-----------------|  
|*Content-Type:*|必要。 設定為 `application/json`。|  
|*Authorization:*|必要。 設定為有效的 `Bearer` [API 金鑰](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#asynchronous-call-polling-based)。 |  

此範例示範傳回目前計費週期詳細資料的同步呼叫。 基於效能的考量，同步呼叫會傳回過去一個月的資訊。  您也可以[非同步呼叫 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#asynchronous-call-polling-based) 傳回 36 個月的資料。


## <a name="response"></a>Response  

傳回狀態碼 200 (確定) 表示成功回應，其中包含帳戶的詳細費用清單。

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/BillingAccounts/1234/providers/Microsoft.Billing/billingPeriods/201702/providers/Microsoft.Consumption/usageDetails/usageDetailsId1",
      "name": "usageDetailsId1",
      "type": "Microsoft.Consumption/usageDetails",
      "properties": {
        ...
        "usageStart": "2017-02-13T00:00:00Z",
        "usageEnd": "2017-02-13T23:59:59Z",
        "instanceName": "shared1",
        "instanceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Default-Web-eastasia/providers/Microsoft.Web/sites/shared1",
        "currency": "USD",
        "usageQuantity": 0.00328,
        "billableQuantity": 0.00328,
        "pretaxCost": 0.67,
        "isEstimated": false,
        ...
      }
    }
  ]
}
```  

此範例已經過縮減；如需每個回應欄位和錯誤處理的完整說明，請參閱[取得帳單帳戶的使用量詳細資料](/rest/api/consumption/usagedetails/listbybillingaccount)。

## <a name="department-billing"></a>部門帳單 

取得針對某部門所有帳戶所彙總的使用量詳細資料。 

```http
GET https://management.azure.com/providers/Microsoft.Billing/departments/{departmentId}/providers/Microsoft.Consumption/usageDetails?api-version=2018-06-30
Content-Type: application/json   
Authorization: Bearer
```

`{departmentId}` 是必要參數，其中應該包含註冊帳戶中的部門識別碼。

以下是必要標頭： 

|要求標頭|說明|  
|--------------------|-----------------|  
|*Content-Type:*|必要。 設定為 `application/json`。|  
|*Authorization:*|必要。 設定為有效的 `Bearer` [API 金鑰](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#asynchronous-call-polling-based)。 |  

此範例示範傳回目前計費週期詳細資料的同步呼叫。 基於效能的考量，同步呼叫會傳回過去一個月的資訊。  您也可以[非同步呼叫 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#asynchronous-call-polling-based) 傳回 36 個月的資料。

### <a name="response"></a>Response  

成功的回應會傳回狀態碼 200 (確定)，回應中會列出指定帳單期間的詳細使用量資訊和成本以及部門的發票識別碼。


下列範例顯示 `1234` 部門的 REST API 輸出。

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/Departments/1234/providers/Microsoft.Billing/billingPeriods/201702/providers/Microsoft.Consumption/usageDetails/usageDetailsId1",
      "name": "usageDetailsId1",
      "type": "Microsoft.Consumption/usageDetails",
      "properties": {
        "billingPeriodId": "/providers/Microsoft.Billing/Departments/1234/providers/Microsoft.Billing/billingPeriods/201702",
        "invoiceId": "/providers/Microsoft.Billing/Departments/1234/providers/Microsoft.Billing/invoices/201703-123456789",
        "usageStart": "2017-02-13T00:00:00Z",
        "usageEnd": "2017-02-13T23:59:59Z",
        "instanceName": "shared1",
        "instanceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/Default-Web-eastasia/providers/Microsoft.Web/sites/shared1",
        "instanceLocation": "eastasia",
        "currency": "USD",
        "usageQuantity": 0.00328,
        "billableQuantity": 0.00328,
        "pretaxCost": 0.67,
        ...
      }
    }
  ]
}
```  

此範例已經過縮減；如需每個回應欄位和錯誤處理的完整說明，請參閱[取得部門的使用量詳細資料](/rest/api/consumption/usagedetails/listbydepartment)。

## <a name="enrollment-account-billing"></a>註冊帳戶帳單

取得針對註冊帳戶所彙總的使用量詳細資料。

```http
GET GET https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts/{enrollmentAccountId}/providers/Microsoft.Consumption/usageDetails?api-version=2018-06-30
Content-Type: application/json   
Authorization: Bearer
```

`{enrollmentAccountId}` 是必要參數，其中應該包含註冊帳戶的識別碼。

以下是必要標頭： 

|要求標頭|說明|  
|--------------------|-----------------|  
|*Content-Type:*|必要。 設定為 `application/json`。|  
|*Authorization:*|必要。 設定為有效的 `Bearer` [API 金鑰](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#asynchronous-call-polling-based)。 |  

此範例示範傳回目前計費週期詳細資料的同步呼叫。 基於效能的考量，同步呼叫會傳回過去一個月的資訊。  您也可以[非同步呼叫 API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#asynchronous-call-polling-based) 傳回 36 個月的資料。

### <a name="response"></a>Response  

成功的回應會傳回狀態碼 200 (確定)，回應中會列出指定帳單期間的詳細使用量資訊和成本以及部門的發票識別碼。

下列範例顯示 `1234` 企業註冊的 REST API 輸出。

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/EnrollmentAccounts/1234/providers/Microsoft.Billing/billingPeriods/201702/providers/Microsoft.Consumption/usageDetails/usageDetailsId1",
      "name": "usageDetailsId1",
      "type": "Microsoft.Consumption/usageDetails",
      "properties": {
        "billingPeriodId": "/providers/Microsoft.Billing/EnrollmentAccounts/1234/providers/Microsoft.Billing/billingPeriods/201702",
        "invoiceId": "/providers/Microsoft.Billing/EnrollmentAccounts/1234/providers/Microsoft.Billing/invoices/201703-123456789",
        "usageStart": "2017-02-13T00:00:00Z",
        "usageEnd": "2017-02-13T23:59:59Z",
        ....
        "currency": "USD",
        "usageQuantity": 0.00328,
        "billableQuantity": 0.00328,
        "pretaxCost": 0.67,
        ...
      }
    }
  ]
}
``` 

此範例已經過縮減；如需每個回應欄位和錯誤處理的完整說明，請參閱[取得註冊帳戶的使用量詳細資料](/rest/api/consumption/usagedetails/listbyenrollmentaccount)。

## <a name="next-steps"></a>後續步驟 
- 檢閱[企業報告概觀](https://docs.microsoft.com/azure/billing/billing-enterprise-api)
- 調查[企業計費 REST API](https://docs.microsoft.com/rest/api/billing/)   
- [Get started with Azure REST API](https://docs.microsoft.com/rest/api/azure/) (開始使用 Azure REST API)   
