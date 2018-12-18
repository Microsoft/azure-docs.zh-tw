---
title: 將 Azure CDN 設定檔從 Verizon 標準移轉至 Verizon 進階 | Microsoft Docs
description: 了解如何將設定檔從 Verizon 標準移轉至 Verizon 進階的詳細資料。
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
ms.topic: article
ms.date: 06/21/2018
ms.author: magattus
ms.custom: ''
ms.openlocfilehash: 7768dde424aedc295b53512db50c9dfc9db9ab8c
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49091465"
---
# <a name="migrate-an-azure-cdn-profile-from-standard-verizon-to-premium-verizon"></a>將 Azure CDN 設定檔從標準 Verizon 移轉至進階 Verizon

當您建立 Azure 內容傳遞網路 (CDN) 設定檔來管理端點時，Azure CDN 提供四個不同的供應項目供您選擇。 如需不同產品及其可用功能的相關資訊，請參閱[比較 Azure CDN 產品功能](cdn-features.md)。

如果您已建立**來自 Verizon 的 Azure CDN 標準**設定檔，並使用它來管理您的 CDN 端點，可選擇將它升級為**來自 Verizon 的 Azure CDN 進階**設定檔。 當您升級時，將保留您的 CDN 端點和所有資料。 

> [!IMPORTANT]
> 一旦升級為**來自 Verizon 的 Azure CDN 進階**設定檔之後，稍後就無法將它轉換回**來自 Verizon 的 Azure CDN 標準**設定檔。
> 

若要升級**來自 Verizon 的 Azure CDN 標準**設定檔，請連絡[Microsoft 支援服務](https://azure.microsoft.com/support/options/)。

## <a name="profile-comparison"></a>設定檔比較
**來自 Verizon 的 Azure CDN 進階**設定檔與**來自 Verizon 的 Azure CDN 標準**設定檔之間具有下列重要差異：
- 針對某些 Azure CDN 功能 (例如[壓縮](cdn-improve-performance.md)、[快取規則](cdn-caching-rules.md)和[地區篩選](cdn-restrict-access-by-country.md))，您無法使用 Azure CDN 介面，而是必須透過 [管理] 按鈕使用 Verizon 入口網站。
- API：不同於標準 Verizon，您無法使用 API 來控制可從進階 Verizon 入口網站存取的功能。 不過，您可以使用 API 來控制其他常見功能，例如，建立/刪除端點、清除/載入快取的資產，以及啟用/停用自訂網域。
- 定價：進階 Verizon 的資料傳輸定價結構與標準 Verizon 的不同。 如需詳細資訊，請參閱[內容傳遞網路定價](https://azure.microsoft.com/pricing/details/cdn/)。

**來自 Verizon 的 Azure CDN 進階**設定檔具有下列其他功能：
- [權杖驗證](cdn-token-auth.md)：允許使用者取得並使用權杖來擷取安全的資源。
- [規則引擎](cdn-rules-engine.md)：允許您自訂 HTTP 要求的處理方式。
- 進階分析工具：
   - [詳細的 HTTP 分析](cdn-advanced-http-reports.md)
   - [邊緣效能分析](cdn-edge-performance.md)
   - [即時分析](cdn-real-time-alerts.md)


## <a name="next-steps"></a>後續步驟
若要深入了解規則引擎，請參閱 [Azure CDN 規則引擎參考](cdn-rules-engine-reference.md)。

