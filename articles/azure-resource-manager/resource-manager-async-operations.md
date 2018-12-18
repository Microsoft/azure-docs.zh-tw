---
title: Azure 非同步作業 | Microsoft Docs
description: 描述如何在 Azure 中追蹤非同步作業。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
ms.assetid: ''
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/21/2018
ms.author: tomfitz
ms.openlocfilehash: 601f4a899393d8ddd5ea698d4d01ade7141ee91f
ms.sourcegitcommit: 76797c962fa04d8af9a7b9153eaa042cf74b2699
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42139891"
---
# <a name="track-asynchronous-azure-operations"></a>追蹤非同步 Azure 作業
某些 Azure REST 作業因為無法快速完成，而以非同步方式執行。 本文說明如何透過回應中傳回的值，以追蹤非同步作業的狀態。  

## <a name="status-codes-for-asynchronous-operations"></a>非同步作業的狀態碼
非同步作業最初會傳回下列其中一個 HTTP 狀態碼︰

* 201 (已建立)
* 202 (已接受) 

作業成功完成時會傳回下列其中一個狀態碼︰

* 200 (確定)
* 204 (沒有內容) 

請參閱 [REST API 文件](/rest/api/)，查看您執行作業的回應。

## <a name="monitor-status-of-operation"></a>監視作業的狀態
非同步 REST 作業傳回的標頭值，可用來判斷作業的狀態。 可能有三個標頭值可檢查︰

* `Azure-AsyncOperation` - 用於檢查作業進行中狀態的 URL。 如果您的作業傳回此值，一律使用它 (而非 Location) 來追蹤作業的狀態。
* `Location` - 用於判斷作業何時完成的 URL。 只在未傳回 Azure-AsyncOperation 時才使用此值。
* `Retry-After` - 在檢查非同步作業的狀態之前等待的秒數。

不過，並非每個非同步作業會傳回所有這些值。 例如，您可能需要評估一項作業的 Azure AsyncOperation 標頭值，但要評估另一項作業的 Location 標頭值。 

擷取標頭值的方式如同擷取要求的任何標頭值一樣。 例如，在 C# 中，您可以使用下列程式碼從名為 `response` 的 `HttpWebResponse` 物件擷取標頭值︰

```cs
response.Headers.GetValues("Azure-AsyncOperation").GetValue(0)
```

## <a name="azure-asyncoperation-request-and-response"></a>Azure-AsyncOperation 要求和回應

若要取得非同步作業的狀態，請將 GET 要求傳送至 Azure-AsyncOperation 標頭值中的 URL。

這項作業的回應內文包含作業的相關資訊。 下列範例顯示作業所傳回的可能值︰

```json
{
    "id": "{resource path from GET operation}",
    "name": "{operation-id}", 
    "status" : "Succeeded | Failed | Canceled | {resource provider values}", 
    "startTime": "2017-01-06T20:56:36.002812+00:00",
    "endTime": "2017-01-06T20:56:56.002812+00:00",
    "percentComplete": {double between 0 and 100 },
    "properties": {
        /* Specific resource provider values for successful operations */
    },
    "error" : { 
        "code": "{error code}",  
        "message": "{error description}" 
    }
}
```

所有回應只傳回 `status`。 當狀態為 Failed 或 Canceled 時會傳回錯誤物件。 其他所有值都是選擇性，因此，您收到的回應可能不同於範例。

## <a name="provisioningstate-values"></a>provisioningState 值

建立、更新或刪除資源的作業 (PUT、PATCH、DELETE) 通常會傳回 `provisioningState` 值。 當作業完成時會傳回下列三個值之一︰ 

* Succeeded
* Failed
* Canceled

其他所有值表示作業仍在執行中。 資源提供者可以傳回自訂值來指出其狀態。 例如，當要求已收到且正在執行時，您可能會收到 **Accepted**。

## <a name="example-requests-and-responses"></a>要求和回應範例

### <a name="start-virtual-machine-202-with-azure-asyncoperation"></a>啟動虛擬機器 (202 與 Azure-AsyncOperation)
這個範例示範如何判斷虛擬機器**啟動**作業的狀態。 初始要求是下列格式︰

```HTTP
POST 
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Compute/virtualMachines/{vm-name}/start?api-version=2016-03-30
```

它傳回狀態碼 202。 在標頭值之中，您會看到：

```HTTP
Azure-AsyncOperation : https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Compute/locations/{region}/operations/{operation-id}?api-version=2016-03-30
```

若要檢查非同步作業的狀態，請將另一個要求傳送至該 URL。

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Compute/locations/{region}/operations/{operation-id}?api-version=2016-03-30
```

回應內文包含作業的狀態︰

```json
{
  "startTime": "2017-01-06T18:58:24.7596323+00:00",
  "status": "InProgress",
  "name": "9a062a88-e463-4697-bef2-fe039df73a02"
}
```

### <a name="deploy-resources-201-with-azure-asyncoperation"></a>部署資源 (201 與 Azure-AsyncOperation)

這個範例示範將資源部署至 Azure 時，如何判斷**部署**作業的狀態。 初始要求是下列格式︰

```HTTP
PUT
https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/microsoft.resources/deployments/{deployment-name}?api-version=2016-09-01
```

它傳回狀態碼 201。 回應內文包含︰

```json
"provisioningState":"Accepted",
```

在標頭值之中，您會看到：

```HTTP
Azure-AsyncOperation: https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Resources/deployments/{deployment-name}/operationStatuses/{operation-id}?api-version=2016-09-01
```

若要檢查非同步作業的狀態，請將另一個要求傳送至該 URL。

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.Resources/deployments/{deployment-name}/operationStatuses/{operation-id}?api-version=2016-09-01
```

回應內文包含作業的狀態︰

```json
{"status":"Running"}
```

部署完成時，回應包含︰

```json
{"status":"Succeeded"}
```

### <a name="create-storage-account-202-with-location-and-retry-after"></a>建立儲存體帳戶 (202 與 Location 和 Retry-After)

這個範例示範如何判斷儲存體帳戶**建立**作業的狀態。 初始要求是下列格式︰

```HTTP
PUT
https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-name}?api-version=2016-01-01
```

要求內文包含儲存體帳戶的屬性︰

```json
{ "location": "South Central US", "properties": {}, "sku": { "name": "Standard_LRS" }, "kind": "Storage" }
```

它傳回狀態碼 202。 在標頭值之中，您會看到下列兩個值︰

```HTTP
Location: https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Storage/operations/{operation-id}?monitor=true&api-version=2016-01-01
Retry-After: 17
```

等待 Retry-After 中指定的秒數經過之後，將另一個要求傳送至該 URL，以檢查非同步作業的狀態。

```HTTP
GET 
https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Storage/operations/{operation-id}?monitor=true&api-version=2016-01-01
```

如果要求仍在執行中，您會收到狀態碼 202。 如果要求已完成，您會收到狀態碼 200，回應內文會包含已建立之儲存體帳戶的屬性。

## <a name="next-steps"></a>後續步驟

* 如需每個 REST 作業的相關文件，請參閱 [REST API 文件](/rest/api/)。
* 如需透過 Resource Manager REST API 部署範本的相關資訊，請參閱[使用 Resource Manager 範本和 Resource Manager REST API 部署資源](resource-group-template-deploy-rest.md)。