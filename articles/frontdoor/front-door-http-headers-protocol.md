---
title: Azure Front Door Service - HTTP 標頭通訊協定支援 | Microsoft Docs
description: 本文可協助您了解 Front Door 支援的 HTTP 標頭通訊協定
services: frontdoor
documentationcenter: ''
author: sharad4u
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: sharadag
ms.openlocfilehash: 0dcb769627714be9da55faf2a8e82c8750789498
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47038845"
---
# <a name="azure-front-door-service---http-headers-protocol-support"></a>Azure Front Door Service - HTTP 標頭通訊協定支援
本文件概述 Azure Front Door Service 以下圖所述呼叫路徑的各種組件支援的通訊協定。 下列各節針對 Front Door 支援的 HTTP 標頭提供更多見解。

![Azure Front Door Service HTTP 標頭通訊協定][1]

>[!WARNING]
>Azure Front Door Service 不會對這裡未記載的任何 HTTP 標頭提供保證。

## <a name="1-client-to-front-door"></a>1.用戶端至 Front Door
Front Door 會接受來自傳入要求的大多數標頭 (但不會加以修改)，不過，有一些保留的標頭將會從傳入要求中移除 (如果已傳送)。 這包括具有下列前置詞的標頭：
 - X-FD*
 - X-MS*

## <a name="2-front-door-to-backend"></a>2.Front Door 至後端

Front Door 會包含來自傳入要求的標頭，除非這些標頭由於上述的限制而遭到移除。 Front Door 也會新增下列標題：

| 頁首  | 範例和描述 |
| ------------- | ------------- |
| X-MS-Ref |  *X-MS-Ref: 0WrHgWgAAAACFupORp/8MS6vxhG/WUvawV1NURURHRTAzMjEARWRnZQ==* </br> 這是唯一的參考字串，可識別由 Front Door 提供服務的要求。 這對於疑難排解極為重要，因為它用來搜尋存取記錄。|
| X-MS-RequestChain |  *X-MS-RequestChain: hops=1* </br> 這是 Front Door 用來偵測要求迴圈的標頭，使用者不應該相依於它。 |
| X-MS-Via |  *X-MS-Via: Azure* </br> 這是由 Front Door 所新增，表示 Azure/Front Door 曾是用戶端與後端之間要求的中繼接收者。 |

## <a name="3-front-door-to-client"></a>3.Front Door 至用戶端

以下是從 Front Door 傳送至用戶端的標頭。 從後端傳送至 Front Door 的任何標頭也會傳遞給用戶端。

| 頁首  | 範例 |
| ------------- | ------------- |
| X-MS-Ref |  *X-MS-Ref: 0WrHgWgAAAACFupORp/8MS6vxhG/WUvawV1NURURHRTAzMjEARWRnZQ==* </br> 這是唯一的參考字串，可識別由 Front Door 提供服務的要求。 這對於疑難排解極為重要，因為它用來搜尋存取記錄。|

## <a name="next-steps"></a>後續步驟

- 了解如何[建立 Front Door](quickstart-create-front-door.md)。
- 了解 [Front Door 的運作方式](front-door-routing-architecture.md)。

<!--Image references-->
[1]: ./media/front-door-http-headers-protocol/front-door-protocol-summary.png