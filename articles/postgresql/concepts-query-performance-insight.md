---
title: 適用於 PostgreSQL 的 Azure 資料庫中的查詢效能深入解析
description: 此文章描述在適用於 PostgreSQL 的 Azure 資料庫中使用查詢效能深入解析功能。
services: postgresql
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/26/2018
ms.openlocfilehash: cc041104169ca8c4344b9d3de597283d122e63db
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47394755"
---
# <a name="query-performance-insight"></a>查詢效能深入解析 

**適用對象：** 適用於 PostgreSQL 的 Azure 資料庫 9.6 和 10

> [!IMPORTANT]
> 查詢效能深入解析功能僅在有限的區域處於公開預覽狀態。 

查詢效能深入解析可協助您快速找出執行時間最長的查詢、一段時間後的變化情形，以及受到哪些等候的影響。

## <a name="permissions"></a>權限
需要**擁有者**或**參與者**權限，才能檢視查詢效能深入解析中的查詢文字。 **讀取者**可以檢視圖表和資料表，但不能檢視查詢文字。

## <a name="prerequisites"></a>必要條件
資料必須存在於[查詢存放區](concepts-query-store.md)中，查詢效能深入解析才能運作。

## <a name="viewing-performance-insights"></a>檢視效能深入解析
Azure 入口網站中的[查詢效能深入解析](concepts-query-performance-insight.md)檢視會以視覺效果呈現來自查詢存放區的重要資訊。 

在適用於 PostgreSQL 的 Azure 資料庫伺服器的入口網站頁面中，在功能表列的 [支援與疑難排解] 區段下選取的 [查詢效能深入解析]。

![查詢效能深入解析長時間執行的查詢](./media/concepts-query-performance-insight/query-performance-insight-landing-page.png)

[長時間執行的查詢] 索引標籤會每隔 15 分鐘彙總一次，依每次執行的平均持續時間，顯示前 5 個查詢。 您可以從**查詢數目**下拉式清單中選取，來檢視更多查詢。 當您這麼做時，特定查詢識別碼的圖表色彩可能會有所變更。

您可以在圖表中按一下並拖曳來縮小到特定時間範圍。 或者，使用放大和縮小圖示，分別檢視一段較短或較長的時間。

圖表下方的資料表會提供該時間範圍內長時間執行之查詢的詳細資訊。

選取 [等候統計資料] 索引標籤，檢視伺服器中等候的對應視覺效果。

![查詢效能深入解析等候統計資料](./media/concepts-query-performance-insight/query-performance-insight-wait-statistics.png)

## <a name="next-steps"></a>後續步驟
- 深入了解在適用於 PostgreSQL 的 Azure 資料庫進行[監視和微調](concepts-monitoring.md)。


