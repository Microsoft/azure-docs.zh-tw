---
title: Azure Resource Manager 要求限制 | Microsoft Docs
description: 描述如何在到達訂用帳戶限制時，對 Azure Resource Manager 要求使用節流。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: e1047233-b8e4-4232-8919-3268d93a3824
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/10/2018
ms.author: tomfitz
ms.openlocfilehash: f3dcb0c5036b2cfc38ef2a6a16269a8697bbd9e6
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34358858"
---
# <a name="throttling-resource-manager-requests"></a>對 Resource Manager 要求進行節流
針對每個訂用帳戶和租用戶，Resource Manager 限制每小時只能有 15000 個讀取要求和 1200 個寫入要求。 這些限制適用於每個 Azure Resource Manager 執行個體。 每個 Azure 區域中都有多個執行個體，且 Azure Resource Manager 會部署到所有 Azure 區域。  因此，實際上的限制比這些限制還要高，因為通常是由多個不同的執行個體來服務使用者要求。

如果應用程式或指令碼到達這些限制，便需要對要求進行節流。 本文說明如何判斷觸達限制前還剩下多少要求，以及在觸達限制時該如何應對。

當您到達限制時，您會收到 HTTP 狀態碼 **429 太多要求**。

要求數會受訂用帳戶或租用戶所限制。 如果訂用帳戶中有多個並行應用程式提出要求，來自這些應用程式的要求會加總起來，以判斷剩餘的要求數。

受訂用帳戶限制的要求是涉及傳遞訂用帳戶識別碼的要求，例如擷取訂用帳戶中的資源群組。 受租用戶限制的要求則未包含訂用帳戶識別碼，例如擷取有效的 Azure 位置。

## <a name="remaining-requests"></a>剩餘的要求
您可以藉由檢查回應標頭來判斷剩餘的要求數。 每個要求都包含剩餘之讀取和寫入要求數的值。 下表描述可供檢查這些值的回應標頭︰

| 回應標頭 | 說明 |
| --- | --- |
| x-ms-ratelimit-remaining-subscription-reads |受訂用帳戶限制的剩餘讀取。 讀取作業會傳回此值。 |
| x-ms-ratelimit-remaining-subscription-writes |受訂用帳戶限制的剩餘寫入。 寫入作業會傳回此值。 |
| x-ms-ratelimit-remaining-tenant-reads |受租用戶限制的剩餘讀取 |
| x-ms-ratelimit-remaining-tenant-writes |受租用戶限制的剩餘寫入 |
| x-ms-ratelimit-remaining-subscription-resource-requests |受訂用帳戶限制的剩餘資源類型要求。<br /><br />只有在服務已覆寫預設限制時，才會傳回此標頭值。 Resource Manager 會新增此值，而非訂用帳戶讀取或寫入。 |
| x-ms-ratelimit-remaining-subscription-resource-entities-read |受訂用帳戶限制的剩餘資源類型集合要求。<br /><br />只有在服務已覆寫預設限制時，才會傳回此標頭值。 這個值會提供剩餘的集合要求 (列出資源) 數目。 |
| x-ms-ratelimit-remaining-tenant-resource-requests |受租用戶限制的剩餘資源類型要求。<br /><br />只有租用戶層級的要求且在服務已覆寫預設限制時，才會新增此標頭。 Resource Manager 會新增此值，而非租用戶讀取或寫入。 |
| x-ms-ratelimit-remaining-tenant-resource-entities-read |受租用戶限制的剩餘資源類型集合要求。<br /><br />只有租用戶層級的要求且在服務已覆寫預設限制時，才會新增此標頭。 |

## <a name="retrieving-the-header-values"></a>擷取標頭值
在程式碼或指令碼中擷取這些標頭值與擷取任何標頭值並無不同。 

例如，在 **C#** 中，您可以使用下列程式碼從 **HttpWebResponse** 物件 (名為 **response**) 擷取標頭值︰

```cs
response.Headers.GetValues("x-ms-ratelimit-remaining-subscription-reads").GetValue(0)
```

在 **PowerShell** 中，您可以從 Invoke-WebRequest 作業擷取標頭值。

```powershell
$r = Invoke-WebRequest -Uri https://management.azure.com/subscriptions/{guid}/resourcegroups?api-version=2016-09-01 -Method GET -Headers $authHeaders
$r.Headers["x-ms-ratelimit-remaining-subscription-reads"]
```

或者，如果您想要查看可用於偵錯的剩餘要求，您可以在 **PowerShell** Cmdlet 提供 **-Debug** 參數。

```powershell
Get-AzureRmResourceGroup -Debug
```

這會傳回許多值，包括下列回應值︰

```powershell
DEBUG: ============================ HTTP RESPONSE ============================

Status Code:
OK

Headers:
Pragma                        : no-cache
x-ms-ratelimit-remaining-subscription-reads: 14999
```

若要取得寫入限制，請使用寫入作業： 

```powershell
New-AzureRmResourceGroup -Name myresourcegroup -Location westus -Debug
```

這會傳回許多值，包括下列值︰

```powershell
DEBUG: ============================ HTTP RESPONSE ============================

Status Code:
Created

Headers:
Pragma                        : no-cache
x-ms-ratelimit-remaining-subscription-writes: 1199
```

在 **Azure CLI** 中，您可以使用更詳細的選項擷取標頭值。

```azurecli
az group list --verbose --debug
```

這會傳回許多值，包括下列值︰

```azurecli
msrest.http_logger : Response status: 200
msrest.http_logger : Response headers:
msrest.http_logger :     'Cache-Control': 'no-cache'
msrest.http_logger :     'Pragma': 'no-cache'
msrest.http_logger :     'Content-Type': 'application/json; charset=utf-8'
msrest.http_logger :     'Content-Encoding': 'gzip'
msrest.http_logger :     'Expires': '-1'
msrest.http_logger :     'Vary': 'Accept-Encoding'
msrest.http_logger :     'x-ms-ratelimit-remaining-subscription-reads': '14998'
```

若要取得寫入限制，請使用寫入作業： 

```azurecli
az group create -n myresourcegroup --location westus --verbose --debug
```

這會傳回許多值，包括下列值︰

```azurecli
msrest.http_logger : Response status: 201
msrest.http_logger : Response headers:
msrest.http_logger :     'Cache-Control': 'no-cache'
msrest.http_logger :     'Pragma': 'no-cache'
msrest.http_logger :     'Content-Length': '163'
msrest.http_logger :     'Content-Type': 'application/json; charset=utf-8'
msrest.http_logger :     'Expires': '-1'
msrest.http_logger :     'x-ms-ratelimit-remaining-subscription-writes': '1199'
```

## <a name="waiting-before-sending-next-request"></a>在傳送下一個要求前先稍候
當您到達要求限制時，Resource Manager 會在標頭中傳回 **429** HTTP 狀態碼和 **Retry-After** 值。 **Retry-After** 值會指定應用程式在傳送下一個要求之前所應等待 (或睡眠) 的秒數。 如果重試值沒過您就傳送要求，則不會處理要求，並且會傳回新的重試值。

## <a name="next-steps"></a>後續步驟

* 如需有關限制和配額的詳細資訊，請參閱 [Azure 訂用帳戶和服務限制、配額及條件約束](../azure-subscription-service-limits.md)。
* 若要了解如何處理非同步 REST 要求，請參閱[追蹤非同步 Azure 作業 (英文)](resource-manager-async-operations.md)。
