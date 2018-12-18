---
title: 分析 Azure CDN 使用模式 | Microsoft Docs
description: 本文說明適用於 Azure CDN 產品的不同分析報告類型。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: 95e18b3c-b987-46c2-baa8-a27a029e3076
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/05/2017
ms.author: magattus
ms.openlocfilehash: 45b3698dd77bda815218b43405d64819c3e4789f
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49091261"
---
# <a name="analyze-azure-cdn-usage-patterns"></a>分析 Azure CDN 使用模式

啟用應用程式的 CDN 後，您可以監視 CDN 使用情況、檢查傳遞的健康情況，並為可能的問題進行疑難排解。 Azure CDN 會以下列方式提供這些功能： 

## <a name="core-analytics-via-azure-diagnostic-logs"></a>透過 Azure 診斷記錄的核心分析

核心分析適用於所有定價層的 CDN 端點。 Azure 診斷記錄可以將核心分析匯出至 Azure 儲存體、事件中樞或 Azure Log Analytics。 Azure Log Analytics 會提供具有圖表的解決方案，這些圖表可由使用者設定和自訂。 如需 Azure 診斷記錄的詳細資訊，請參閱 [Azure 診斷記錄](cdn-azure-diagnostic-logs.md)。

## <a name="verizon-core-reports"></a>Verizon 核心報告

身為具有**來自 Verizon 的 Azure CDN 標準**或**來自 Verizon 的 Azure CDN 進階**設定檔的 Azure CDN 使用者，您可以在 Verizon 補充入口網站中檢視 Verizon 核心報告。 Verizon 核心報告可以透過 Azure 入口網站的 [管理] 選項存取，並提供各式各樣的圖表和檢視。 如需詳細資訊，請參閱 [Verizon 核心報告](cdn-analyze-usage-patterns.md)。

## <a name="verizon-custom-reports"></a>Verizon 自訂報告

身為具有**來自 Verizon 的 Azure CDN 標準**或**來自 Verizon 的 Azure CDN 進階**設定檔的 Azure CDN 使用者，您可以在 Verizon 補充入口網站中檢視 Verizon 自訂報告。 Verizon 自訂報告可以透過 Azure 入口網站的 [管理] 選項存取。 [Verizon 自訂報告] 頁面會針對屬於 Azure CDN 設定檔的每個邊緣 CName 顯示點擊次數或傳輸的資料。 此資料可依照任何一段時間的 HTTP 回應碼或快取狀態進行分組。 如需詳細資訊，請參閱 [Verizon 自訂報告](cdn-verizon-custom-reports.md)。

## <a name="azure-cdn-premium-from-verizon-reports"></a>來自 Verizon 的進階 Azure CDN 報告

您也可以使用**來自 Verizon 的 Azure CDN 進階** 來存取下列報告：
   * [進階 HTTP 報告](cdn-advanced-http-reports.md)
   * [即時統計資料](cdn-real-time-stats.md)
   * [邊緣節點效能](cdn-edge-performance.md)

