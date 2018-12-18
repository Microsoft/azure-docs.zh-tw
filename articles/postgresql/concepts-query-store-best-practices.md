---
title: 適用於 PostgreSQL 的 Azure 資料庫中的查詢存放區最佳做法
description: 此文章描述在適用於 PostgreSQL 的 Azure 資料庫中使用查詢存放區的最佳做法。
services: postgresql
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/26/2018
ms.openlocfilehash: 54a86a7ea1852efba0776451291820f4174a1f1f
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49377567"
---
# <a name="best-practices-for-query-store"></a>查詢存放區的最佳做法

**適用於：** 適用於 PostgreSQL 的 Azure 資料庫 9.6 和 10

> [!IMPORTANT]
> 查詢存放區功能處於公開預覽狀態。


此文章概述在適用於 PostgreSQL 的 Azure 資料庫中使用查詢存放區的最佳做法。

## <a name="set-the-optimal-query-capture-mode"></a>設定最佳查詢擷取模式
讓查詢存放區擷取對您而言重要的資料。 

|**pg_qs.query_capture_mode** | **案例**|
|---|---|
|_所有_  |根據您的所有查詢與其執行頻率與其他統計資料徹底分析您的工作負載。 識別您工作負載中的新查詢。 偵測是否使用臨機操作查詢來識別使用者或自動參數化的機會。 _全都_有增加的資源耗用成本。 |
|_前幾個_  |專注在前幾個查詢 - 那些由客戶發出的查詢。
|_None_ |您已擷取查詢集與一段時間，您想要在這段時間內調查並減少其他查詢可能會產生之令人困惑之事。 _無_適用於測試及效能評定環境。 _無_應該謹慎使用，因為您可能可能會錯過追蹤及最佳化重要新查詢的機會。 我們無法復原過去時段內的資料。 |

查詢存放區也包含等候統計資料的存放區。 有一個額外的擷取模式查詢，此查詢規範等候統計資料：**pgms_wait_sampling.query_capture_mode** 可設定為 _none_ 或 _all_。 

> [!NOTE] 
> **pg_qs.query_capture_mode** 已取代 **pgms_wait_sampling.query_capture_mode**。 若 pg_qs.query_capture_mode 是 _none_，則 pgms_wait_sampling.query_capture_mode 設定沒有影響。 


## <a name="keep-the-data-you-need"></a>保留您所需的資料
**pg_qs.retention_period_in_days** 參數可指定查詢存放區的資料保留期間 (天)。 較舊的查詢與統計資料會被刪除。 根據預設，查詢存放區是設定為將資料保留 7 天。 避免保留您不打算使用的歷史資料。 若需要保留資料較長的時間，請增加該值。


## <a name="set-the-frequency-of-wait-stats-sampling"></a>設定等候統計資料取樣的頻率 
**pgms_wait_sampling.history_period** 參數可指定等候事件取樣頻率 (毫秒)。 期間越短，取樣頻率就越頻繁。 會擷取更多資訊，但會也產生較高的資源耗用成本。 若伺服器負載不足或您不需要精細度，您可以增加此期間長度


## <a name="get-quick-insights-into-query-store"></a>取得查詢存放區的快速見解
您可以使用 Azure入口網站中的[查詢效能深入解析](concepts-query-performance-insight.md)來取得查詢存放區資料的快速見解。 視覺效果會呈現一段時間內執行時間最長的查詢，以及最長的等候時間。

## <a name="next-steps"></a>後續步驟
- 了解如何使用 [Azure 入口網站](howto-configure-server-parameters-using-portal.md) 或 [Azure CLI](howto-configure-server-parameters-using-cli.md) 來取得或設定參數。