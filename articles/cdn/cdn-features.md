---
title: 比較 Azure 內容傳遞網路 (CDN) 產品功能 | Microsoft Docs
description: 了解每項 Azure 內容傳遞網路 (CDN) 產品所支援的功能。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 09/13/2018
ms.author: magattus
ms.custom: mvc
ms.openlocfilehash: 8152add4fb28f7f2536859ea7dcdae379a014f90
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46951758"
---
# <a name="compare-azure-cdn-product-features"></a>比較 Azure CDN 產品功能

Azure 內容傳遞網路 (CDN) 包含四種產品：**來自 Microsoft 的標準 Azure CDN**、**來自 Akamai 的 Azure CDN 標準**、**來自 Verizon 的 Azure CDN 標準**和**來自 Verizon 的 Azure CDN 進階**。 

如需將 [來自 Verizon 的標準 Azure CDN] 設定檔移轉至 [來自 Verizon 的進階 Azure CDN] 的相關資訊，請參閱[將 Azure CDN 設定檔從標準 Verizon 移轉至進階 Verizon](cdn-migrate.md)。

下表將比較各項產品的可用功能。

| **效能功能和最佳化** | **標準 Microsoft** | **標準 Akamai** | **標準 Verizon** | **進階 Verizon** |
| --- | --- | --- | --- | --- |
| [動態網站加速](https://docs.microsoft.com/azure/cdn/cdn-dynamic-site-acceleration)  |  | **&#x2713;**  | **&#x2713;** | **&#x2713;** |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[動態網站加速 - 調整映像壓縮](https://docs.microsoft.com/azure/cdn/cdn-dynamic-site-acceleration#adaptive-image-compression-akamai-only)  |  | **&#x2713;**  |  |  |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[動態網站加速 - 物件預先擷取](https://docs.microsoft.com/azure/cdn/cdn-dynamic-site-acceleration#object-prefetch-akamai-only)  |  | **&#x2713;**  |  |  |
| [影片串流最佳化](https://docs.microsoft.com/azure/cdn/cdn-media-streaming-optimization)  | \* | **&#x2713;**  | \* |  \* |
| [大型檔案最佳化](https://docs.microsoft.com/azure/cdn/cdn-large-file-optimization)  | \* | **&#x2713;**  | \* |  \* |
| [全域伺服器負載平衡 (GSLB)](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-load-balancing-azure)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [快速清除](cdn-purge-endpoint.md)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [資產預先載入](cdn-preload-endpoint.md)  |  | |**&#x2713;** |**&#x2713;** |
| 快取/標頭設定 (使用[快取規則](cdn-caching-rules.md))  |  |**&#x2713;** |**&#x2713;** | |
| 快取/標頭設定 (使用 [規則引擎](cdn-rules-engine.md))  |  | | |**&#x2713;** |
| [查詢字串快取](cdn-query-string.md)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| IPv4/IPv6 雙重堆疊 | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [HTTP/2 支援](cdn-http2.md)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
||||
 **安全性** | **標準 Microsoft** | **標準 Akamai** | **標準 Verizon** | **進階 Verizon** | 
| CDN 端點的 HTTPS 支援 | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [自訂網域 HTTPS](cdn-custom-ssl.md)  | **&#x2713;** | |**&#x2713;** |**&#x2713;** |
| [自訂網域名稱支援](cdn-map-content-to-custom-domain.md)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [地理位置篩選](cdn-restrict-access-by-country.md)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [權杖驗證](cdn-token-auth.md)  |  |  |  |**&#x2713;**| 
| [DDOS 保護](https://www.us-cert.gov/ncas/tips/ST04-015)  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [使用您自己的憑證](cdn-custom-ssl.md?tabs=option-2-enable-https-with-your-own-certificate#ssl-certificates) |**&#x2713;** |  |  |  |
||||
| **分析和報告** | **標準 Microsoft** | **標準 Akamai** | **標準 Verizon** | **進階 Verizon** | 
| [Azure 診斷記錄](cdn-azure-diagnostic-logs.md)  | **&#x2713;** | **&#x2713;** |**&#x2713;** |**&#x2713;** |
| [來自 Verizon 的 Core 報告](cdn-analyze-usage-patterns.md)  |  | |**&#x2713;** |**&#x2713;** |
| [來自 Verizon 的自訂報告](cdn-verizon-custom-reports.md)  |  | |**&#x2713;** |**&#x2713;** |
| [進階 HTTP 報告](cdn-advanced-http-reports.md)  |  | | |**&#x2713;** |
| [即時統計資料](cdn-real-time-stats.md)  |  | | |**&#x2713;** |
| [邊緣節點效能](cdn-edge-performance.md)  |  | | |**&#x2713;** |
| [即時警示](cdn-real-time-alerts.md)  |  | | |**&#x2713;** |
||||
| **容易使用** | **標準 Microsoft** | **標準 Akamai** | **標準 Verizon** | **進階 Verizon** | 
| 輕鬆地與 Azure 服務 (例如[儲存體](cdn-create-a-storage-account-with-cdn.md)、[Web Apps](cdn-add-to-web-app.md) 和[媒體服務](../media-services/media-services-portal-manage-streaming-endpoints.md)) 整合  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| 透過 [REST API](https://msdn.microsoft.com/library/mt634456.aspx)、[.NET](cdn-app-dev-net.md)、[Node.js](cdn-app-dev-node.md) 或 [PowerShell](cdn-manage-powershell.md) 管理  | **&#x2713;** |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [可自訂的、規則式內容傳遞引擎](cdn-rules-engine.md)  |  | | |**&#x2713;** |
| URL 重新導向/重寫 (使用 [規則引擎](cdn-rules-engine.md))  |  | | |**&#x2713;** |
| 行動裝置規則 (使用 [規則引擎](cdn-rules-engine.md))  |  | | |**&#x2713;** |

\* Microsoft 和 Verizon 支援透過一般 Web 傳遞最佳化直接傳遞大型檔案和媒體。



