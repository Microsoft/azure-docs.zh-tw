---
title: Azure Front Door Service - 後端健康情況監視 | Microsoft Docs
description: 本文將協助您了解 Azure Front Door Service 如何監視後端的健康情況
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
ms.openlocfilehash: 256d530590fadc9e2aeb1ea1efb7a52608014978
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46988558"
---
# <a name="health-probes"></a>健康狀態探查

為了判斷每個後端的健康情況，每個 Front Door 環境會定期向您設定的每個後端傳送一個綜合的 HTTP/HTTPS 要求。 接著，Front Door 會使用來自這些探查的回應，來判斷它應該要路由傳送實際用戶端要求的「最佳」後端。


## <a name="supported-protocols"></a>支援的通訊協定

Front Door 支援透過 HTTP 或 HTTPS 通訊協定傳送探查。 這些探查透過設定用於路由傳送用戶端要求之相同的 TCP 連接埠傳送，並且無法覆寫。

## <a name="health-probe-responses"></a>健全狀況探查回應

| 回應  | 說明 | 
| ------------- | ------------- |
| 判斷健全狀況  |  200 OK 狀態碼表示後端狀況良好。 所有其他項目會視為失敗。 如果基於任何原因 (包括網路失敗) 未收到有效的 HTTP 探查回應，則探查將會視為失敗。|
| 測量延遲  | 延遲是從目前我們傳送探查要求的那一刻起，到我們收到回應的最後一個位元組之前測量的時鐘時間。 我們針對每個要求使用新的 TCP 連線，因此該度量不會偏向使用現有連線的後端。  |

## <a name="how-front-door-determines-backend-health"></a>Front Door 如何判斷後端的健康狀態

Azure Front Door Service 在所有演算法之間使用下述相同的三步驟程序來判斷健全狀況。

1. 排除已停用後端。

2. 排除有健康情況探查錯誤的後端：
    * 藉由查看最後 _n_ 個健全狀況探查回應來完成此選取。 如果至少 _x_ 狀況良好，則後端會視為狀況良好。

    * 透過變更負載平衡設定中的 SampleSize 屬性來設定 _n_。

    * 透過變更負載平衡設定中的 SuccessfulSamplesRequired 屬性來設定 _x_。

3. 在後端集區中狀況良好的集合中，Front Door 還會測量並維護每個後端的延遲 (來回時間)。


## <a name="complete-health-probe-failure"></a>完整的健全狀況探查失敗

如果後端集區中的每個後端的健全狀況探查失敗，那麼 Front Door 會將所有的後端視為狀況良好，並在所有後端的循環配置資源散發中路由傳送流量。

一旦任何後端恢復狀況良好狀態，Front Door 將會繼續正常負載平衡演算法。

## <a name="next-steps"></a>後續步驟

- 了解如何[建立 Front Door](quickstart-create-front-door.md)。
- 了解 [Front Door 的運作方式](front-door-routing-architecture.md)。
