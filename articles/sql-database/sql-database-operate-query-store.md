---
title: 操作 Azure SQL Database 中的查詢存放區
description: 了解如何操作 Azure SQL Database 中的查詢存放區
services: sql-database
author: bonova
manager: craigg
ms.service: sql-database
ms.custom: monitor & tune
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: bonova
ms.openlocfilehash: 37cb77b6738ba1354034dcf77d22a19b96c4ef23
ms.sourcegitcommit: d551ddf8d6c0fd3a884c9852bc4443c1a1485899
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2018
ms.locfileid: "37903093"
---
# <a name="operating-the-query-store-in-azure-sql-database"></a>操作 Azure SQL Database 中的查詢存放區
Azure 中的查詢資料存放區是完全受控的資料庫功能，可持續收集及呈現有關所有查詢的詳細歷程記錄資訊。 您可以將查詢存放區視為類似於飛航資料記錄器，可大幅簡化雲端選項與內部部署客戶的查詢效能疑難排解。 這篇文章說明在 Azure 中操作查詢存放區的特定層面。 您可以使用此預先收集的查詢資料，快速地診斷並解決效能問題，因此能夠花更多時間專注於業務上。 

查詢存放區已自 2015 年 11 月在 Azure SQL Database 中 [通用版本上市](https://azure.microsoft.com/updates/general-availability-azure-sql-database-query-store/) 。 查詢存放區是效能分析及調整功能的基礎，例如 [SQL Database 建議程式和效能儀表板](https://azure.microsoft.com/updates/sqldatabaseadvisorga/)。 在本文發行時，查詢存放區正於 Azure 中 200,000 個以上的使用者資料庫內執行，不間斷地收集數個月的查詢相關資訊。

> [!IMPORTANT]
> Microsoft 正在為所有 Azure SQL Database (現有和新的) 啟用「查詢存放區」。 
> 
> 

## <a name="optimal-query-store-configuration"></a>最佳查詢存放區組態
本節描述最佳的組態預設值，其設計目的是確保查詢存放區及相依功能 (例如 [SQL Database 建議程式和效能儀表板](https://azure.microsoft.com/updates/sqldatabaseadvisorga/)) 能夠可靠地運作。 預設組態已針對持續收集資料最佳化，也就是在 OFF/READ_ONLY 狀態花費最少的時間。

| 組態 | 說明 | 預設值 | 註解 |
| --- | --- | --- | --- |
| MAX_STORAGE_SIZE_MB |指定查詢存放區在客戶資料庫內佔用的資料空間限制 |100 |對新資料庫強制執行 |
| INTERVAL_LENGTH_MINUTES |定義彙總和保存查詢計畫所收集到的執行階段統計資料的時段大小。 對於此組態定義的一段時間，每個使用中的查詢計劃最多會有一個資料列 |60 |對新資料庫強制執行 |
| STALE_QUERY_THRESHOLD_DAYS |以時間為基礎的清理原則，可控制保存執行階段統計資料和非使用中查詢的保留期限 |30 |對新資料庫和具有先前的預設值 (367) 的資料庫強制執行 |
| SIZE_BASED_CLEANUP_MODE |指定當查詢存放區資料大小接近限制時，是否進行自動資料清理 |AUTO |對所有資料庫強制執行 |
| QUERY_CAPTURE_MODE |指定是否會追蹤所有查詢或只有查詢的子集 |AUTO |對所有資料庫強制執行 |
| FLUSH_INTERVAL_SECONDS |指定擷取的執行階段統計資料在排清到磁碟之前，保留在記憶體中的最大期間 |900 |對新資料庫強制執行 |
|  | | | |

> [!IMPORTANT]
> 在所有 Azure SQL Database 中查詢存放區啟用的最後階段會自動套用這些預設值 (請參閱上面的重要附註)。 在這次推出之後，Azure SQL Database 不會變更客戶所設定的組態值，除非該組態值會對主要工作負載或查詢存放區的可靠操作造成負面影響。
> 
> 

如果您想要繼續使用自訂設定，請使用 [ALTER DATABASE 搭配查詢存放區選項](https://msdn.microsoft.com/library/bb522682.aspx) ，以將組態還原到先前的狀態。 請查看 [使用查詢存放區的最佳作法](https://msdn.microsoft.com/library/mt604821.aspx) ，以了解如何選擇最佳的組態參數。

## <a name="next-steps"></a>後續步驟
[SQL Database 效能深入解析](sql-database-performance.md)

## <a name="additional-resources"></a>其他資源
如需詳細資訊，請參閱下列文章：

* [您的資料庫的航班資料錄製器](https://azure.microsoft.com/blog/query-store-a-flight-data-recorder-for-your-database) 
* [使用查詢存放區來監視效能](https://msdn.microsoft.com/library/dn817826.aspx)
* [查詢存放區使用案例](https://msdn.microsoft.com/library/mt614796.aspx)
 

