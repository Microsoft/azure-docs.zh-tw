---
title: 適用於 PostgreSQL 的 Azure 資料庫中的效能建議
description: 此文章描述可在適用於 PostgreSQL 的 Azure 資料庫中取得的效能建議。
services: postgresql
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/26/2018
ms.openlocfilehash: 6de302dbcfa9d6d1d2b311f41b03d8e54aeb63f6
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47395435"
---
# <a name="performance-recommendations-in-azure-database-for-postgresql"></a>適用於 PostgreSQL 的 Azure 資料庫中的效能建議

**適用對象：** 適用於 PostgreSQL 的 Azure 資料庫 9.6 和 10

> [!IMPORTANT]
> 效能建議僅在有限的區域處於公開預覽狀態。

效能建議功能可找出最上層的索引，以在適用於 PostgreSQL 的 Azure 資料庫伺服器中建立來改善效能。 若要產生索引建議，此功能會考量各種資料庫特性，包括查詢存放區所報告的結構描述和工作負載。 實作任何效能建議後，客戶應測試效能，以評估這些變更的影響。 

## <a name="permissions"></a>權限
需要**擁有者**或**參與者**權限，才能使用效能建議功能執行分析。

## <a name="performance-recommendations"></a>效能建議
[效能建議](concepts-performance-recommendations.md)功能可分析整部伺服器的工作負載，來找出可能會改善效能的索引。

在 PostgreSQL 伺服器的 Azure 入口網站頁面上，從功能表列的 [支援與疑難排解] 區段開啟 [效能建議]。

![效能建議登陸頁面](./media/concepts-performance-recommendations/performance-recommendations-landing-page.png)

選取 [分析] 並選擇資料庫。 隨即會開始分析。 根據您的工作負載，這可能需要幾分鐘才能完成。 分析完成後，入口網站中會有通知。

[效能建議] 視窗會以清單顯示找到的任何建議。 建議會顯示相關的 [資料庫]、[資料表]、[資料行] 及 [索引大小] 資訊。

![效能建議新頁面](./media/concepts-performance-recommendations/performance-recommendations-result.png)

若要實作建議，請複製查詢文字並在您選擇的用戶端執行該文字。

## <a name="next-steps"></a>後續步驟
- 深入了解在適用於 PostgreSQL 的 Azure 資料庫進行[監視和微調](concepts-monitoring.md)。

