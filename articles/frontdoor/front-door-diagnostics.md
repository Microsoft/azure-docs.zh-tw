---
title: Azure Front Door Service - 計量和記錄 | Microsoft Docs
description: 此文章可協助您了解 Azure Front Door Service 支援的不同計量和存取記錄
services: frontdoor
documentationcenter: ''
author: sharad4u
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/18/2018
ms.author: sharadag
ms.openlocfilehash: 643ae03a9350868b172d99b2af7ce23e34fedf47
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46997993"
---
# <a name="monitoring-metrics-for-front-door"></a>監視 Front Door 的計量

使用 Azure Front Door Service，您可以監視主要計量來驗證 Front Door 設定，以及您 Front Door 的使用狀況、健康情況與效能。

## <a name="metrics"></a>度量

計量是某些 Azure 資源的功能，可供您在入口網站中檢視效能計數器。 針對 Front Door，有下列計量可用：

| 計量 | 計量顯示名稱 | 單位 | 維度 | 說明 |
| --- | --- | --- | --- | --- |
| RequestCount | 要求計數 | 計數 | HttpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | 由 Front Door 提供服務的用戶端要求數。  |
| RequestSize | 要求大小 | 位元組 | HttpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | 從用戶端傳送到 Front Door 之要求的位元組數。 |
| ResponseSize | 回應大小 | 位元組 | HttpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | 從 Front Door 傳送到用戶端之回應的位元組數。 |
| TotalLatency | 延遲總計 | 毫秒 | HttpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | 從 Front Door 接收用戶端要求一直到用戶端認可來自 Front Door 的最後回應位元組所經歷的時間。 |
| BackendRequestCount | 後端要求計數 | 計數 | HttpStatus</br>HttpStatusGroup</br>後端 | 從 Front Door 傳送至後端的要求數。 |
| BackendRequestLatency | 後端要求延遲 | 毫秒 | 後端 | 從 Front Door 傳送要求到後端一直到 Front Door 從後端接收最後回應位元組所經歷的時間。 |
| BackendHealthPercentage | 後端健康情況百分比 | 百分比 | 後端</br>BackendPool | 從 Front Door 到後端的健康情況探查成功百分比。 |
| WebApplicationFirewallRequestCount | Web 應用程式防火牆要求計數 | 計數 | PolicyName</br>RuleName</br>動作 | Front Door 應用程式層安全性處理的用戶端要求數。 |


## <a name="next-steps"></a>後續步驟

- 了解如何[建立 Front Door](quickstart-create-front-door.md)。
- 了解 [Front Door 的運作方式](front-door-routing-architecture.md)。
