---
title: 包含檔案
description: 包含檔案
services: frontdoor
author: sharad4u
ms.service: frontdoor
ms.topic: include
ms.date: 05/09/2019
ms.author: sharadag
ms.custom: include file
ms.openlocfilehash: 37ebe2f0c5cbbaca712e69ab4484379ecf0f4830
ms.sourcegitcommit: 42748f80351b336b7a5b6335786096da49febf6a
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2019
ms.locfileid: "72237303"
---
| Resource | 預設/最大限制 |
| --- | --- |
| 每個訂用帳戶的 Azure Front Door 服務資源 | 100 |
| 前端主機，其中包括每個資源的自訂網域 | 100 |
| 每個資源的路由規則 | 100 |
| 每個資源的後端集區數量 | 50 |
| 每個後端集區的後端數量 | 100 |
| 要符合路由規則的路徑模式 | 25 |
| 每個原則的自訂 Web 應用程式防火牆規則 | 10 |
| 每個訂用帳戶的 Web 應用程式防火牆原則 | 100 |
| 每個自訂規則的 Web 應用程式防火牆比對條件 | 10 |
| 每個符合條件的 Web 應用程式防火牆 IP 位址範圍 | 600 |
| Web 應用程式防火牆字串符合每個符合條件的值 | 10 |
| Web 應用程式防火牆字串符合值長度 | 256 |
| Web 應用程式防火牆張貼主體參數名稱長度 | 256 |
| Web 應用程式防火牆 HTTP 標頭名稱長度 | 256 |
| Web 應用程式防火牆 cookie 名稱長度 | 256 |
| Web 應用程式防火牆 HTTP 要求主體大小已檢查 | 128 KB |
| Web 應用程式防火牆自訂回應主體長度 | 2 KB |

### <a name="timeout-values"></a>超時值
#### <a name="client-to-front-door"></a>用戶端至 Front Door
- Front Door 的閒置 TCP 連線逾時為 61 秒。

#### <a name="front-door-to-application-back-end"></a>Front Door 至應用程式的後端
- 如果回應是區塊回應，則會在收到第一個區塊時傳回200。
- 將 HTTP 要求轉送至後端之後，Front Door 會等待後端第一個封包的30秒。 然後，它會將503錯誤傳回給用戶端。
- 從後端接收到第一個封包之後，Front Door 會等待30秒的閒置時間。 然後，它會將503錯誤傳回給用戶端。
- 後端 TCP 會話超時的前端為30分鐘。

### <a name="upload-and-download-data-limit"></a>上傳和下載資料限制

|  | 使用區塊傳輸編碼（CTE） | 沒有 HTTP 區塊化 |
| ---- | ------- | ------- |
| **下載** | 下載大小沒有任何限制。 | 下載大小沒有任何限制。 |
| **上傳** |  只要每個 CTE 上傳小於 2 GB，就不會有任何限制。 | 大小不能大於 2 GB。 |

### <a name="other-limits"></a>其他限制
- URL 大小上限-8192 位元組-指定原始 URL 的最大長度（配置 + 主機名稱 + 埠 + 路徑 + URL 的查詢字串）
- 查詢字串大小上限-4096 個位元組-指定查詢字串的最大長度（以位元組為單位）。
