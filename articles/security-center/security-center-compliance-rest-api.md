---
title: 使用 REST API 進行 Azure 資訊安全中心合規性監視 | Microsoft Docs
description: 使用 Azure 資訊安全中心 REST API 檢閱自動化合規性掃描的結果。
services: security-center
documentationcenter: na
author: rloutlaw
manager: angerobe
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/23/2018
ms.author: rloutlaw
ms.openlocfilehash: 651d7737258f1b1b17c8392a09882388ddf95635
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42819274"
---
# <a name="review-security-center-compliance-results-using-the-azure-rest-apis"></a>使用 Azure REST API 檢閱資訊安全中心合規性結果

在本文中，您會了解如何使用 Azure REST API 擷取訂用帳戶清單的安全性合規性資訊。

## <a name="review-compliance-for-each-subscription"></a>檢閱每個訂用帳戶的合規性

下列範例使用[取得合規性](/rest/api/securitycenter/compliances/get)作業來取得指定合規性和訂用帳戶的安全性評估資訊。

```http
GET https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Security/compliances/{complianceName}?api-version=2017-08-01-preview
```

`{complianceName}` 是必要參數，且應包含針對 `{subscription-id}` 所評估的合規性名稱。 輸出中會包含評估結果，例如：

```json
{
...
  "properties": {
    "assessmentResult": [
      {
        "segmentType": "Compliant",
        "percentage": 77.777777777777786
      }
    ],
    "resourceCount": 18,
    "assessmentTimestampUtcDate": "2018-01-01T00:00:00Z"
  }
}
```

## <a name="review-compliance-for-multiple-subscriptions"></a>檢閱多個訂用帳戶的合規性

若要取得多個訂用帳戶的資料，請發出下列呼叫，使用[列出訂用帳戶](/rest/api/resources/subscriptions/list)作業先取得訂用帳戶的清單。 針對每個傳回的訂用帳戶識別碼，叫用上述[取得合規性](/rest/api/securitycenter/compliances/get)。

該 API 呼叫為：

```http
GET https://management.azure.com/subscriptions?api-version=2016-06-01
```

這會傳回具有如下值的陣列。 請使用上述呼叫中的 subscriptionId 值，來檢閱所有訂用帳戶的合規性資訊。

```json
"value": [
    {
      "id": "/subscriptions/{subscription-id}",
      "subscriptionId": "{subscription-id}",
      "displayName": "Demo subscription",
      ...
    }
```






