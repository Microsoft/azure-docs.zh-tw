---
title: 包含檔案
description: 包含檔案
services: frontdoor
author: sharad4u
ms.service: frontdoor
ms.topic: include
ms.date: 9/17/2018
ms.author: sharadag
ms.custom: include file
ms.openlocfilehash: f0c2d1501b9aa19dec8c4ad157e004a57e0e5070
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47006486"
---
| 資源 | 預設限制 |
| --- | --- |
| 每個訂用帳戶的 Front Door 資源 | 100 |
| 每個資源的前端主機 (包括自訂網域) | 100 |
| 每個資源的路由規則 | 100 |
| 每個資源的後端集區 | 50 |
| 每個後端集區的後端 | 100 |
| 要符合路由規則的路徑模式 | 25 |
| 每個原則的自訂 Web 應用程式防火牆規則 | 10 |
| 每個資源的 Web 應用程式防火牆原則 | 100 |

### <a name="timeout-values"></a>逾時值
#### <a name="client-to-front-door"></a>用戶端至 Front Door
- Front Door 的閒置 TCP 連線逾時為 61 秒。
#### <a name="front-door-to-application-backend"></a>Front Door 至應用程式後端
- 如果回應是區塊式回應，則若/在收到第一個區塊時會傳回 200。
- 在 HTTP 要求轉送到後端之後，Front Door 會等候來自後端的第一個封包 30 秒，然後才會傳回 503 錯誤給用戶端。
- 後端收到第一個封包後，Front Door 會等候 30 秒 (閒置逾時)，然後才會傳回 503 錯誤給用戶端。
- Front Door 至後端 TCP 工作階段的逾時為 30 分鐘。

### <a name="upload--download-data-limit"></a>上傳/下載資料限制

|  | 使用區塊傳輸編碼 (CTE) | 不使用 HTTP 區塊處理 |
| ---- | ------- | ------- |
| **下載** | 下載大小沒有限制 | 下載大小沒有限制 |
| **上傳** |  只要每個 CTE 上傳小於 28.6 MB 就沒有限制 | 大小不能大於 28.6 MB。 |
