---
title: 監視和效能微調 - Azure SQL Database | Microsoft Docs
description: 透過評估和改進來調整 Azure SQL Database 效能的秘訣。
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: danimir
ms.author: v-daljep
ms.reviewer: carlrab
manager: craigg
ms.date: 07/16/2018
ms.openlocfilehash: c79285247950510791ede915fcf0e5373792044f
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47165036"
---
# <a name="monitoring-and-performance-tuning"></a>監視和效能微調

Azure SQL Database 是自動管理和彈性的資料服務，您可以輕鬆監視使用狀況、新增或移除資源 (CPU、資源、IO)、尋找可改善資料庫效能的建議，或讓資料庫調整您的工作負載並自動最佳化效能。

## <a name="overview-of-monitoring-database-performance-in-azure-sql-database"></a>監視 Azure SQL Database 中資料庫效能的概觀
在 Azure 中監視 SQL Database 的效能，必須從監視您選擇之資料庫效能等級相關的資源使用率開始。 監視可協助您判斷資料庫是否有超量的容量或因為資源超量而發生問題，然後判斷是否開始調整 [DTU 形式的購買模型](sql-database-service-tiers-dtu.md)或[虛擬核心形式的購買模型](sql-database-service-tiers-vcore.md) 中的資料庫的計算大小和服務層。 您可以在 [Azure 入口網站](https://portal.azure.com)中使用下列圖形化工具，或使用 SQL [動態管理檢視 (DMV)](sql-database-monitoring-with-dmvs.md)，來監視您的資料庫。

Azure SQL Database 可讓您檢閱[效能微調建議](sql-database-advisor.md)，找出不必變更資源卻能改善和最佳化查詢效能的機會。 遺漏索引與查詢最佳化不足是資料庫效能不佳的常見原因。 您可以套用這些調整建議來改善工作負載的效能。
您也可以套用所有發現的建議並驗證能否改善資料庫效能，讓 Azure SQL Database [自動最佳化查詢](sql-database-automatic-tuning.md)。 您有下列選項可以用來針對資料庫效能進行監視和疑難排解：

- 在 [Azure 入口網站](https://portal.azure.com)中，按一下 [SQL Database]、選取資料庫，然後使用監視圖表尋找接近其上限的資源。 預設會顯示 DTU 耗用量。 按一下 [編輯]  來變更所顯示的時間範圍和值。
- 使用[查詢效能深入解析](sql-database-query-performance.md)來找出花費最多資源的查詢。
- 使用 [SQL Database Advisor](sql-database-advisor-portal.md) 來檢視建立和卸除索引、參數化查詢，以及修正結構描述問題的建議。
- 使用 [Azure SQL Intelligent Insights](sql-database-intelligent-insights.md) 來自動監視您的資料庫效能。 在偵測到效能問題之後，系統就會產生包含問題詳細資料和「根本原因分析」(RCA) 的診斷記錄。 在可能的情況下會提供效能改進建議。
- [啟用自動調整](sql-database-automatic-tuning-enable.md)並讓 Azure SQL Database 自動修正發現的效能問題。
- 您可以使用[動態管理檢視 (DMV)](sql-database-monitoring-with-dmvs.md)、[擴充的事件 (`XEvents`)(sql-database/sql-database-xevent-db-diff-from-svr.md) 和[查詢存放區](https://docs.microsoft.com/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store)即時取得效能參數。 如果您使用這些報告或檢視發現某些問題，請參閱[效能指引](sql-database-performance-guidance.md)來尋找可用來改善 Azure SQL Database 效能的技巧。

## <a name="monitor-databases-using-the-azure-portal"></a>使用 Azure 入口網站監視資料庫
在 [Azure 入口網站](https://portal.azure.com/)中，您可以透過選取資料庫，並按一下 [監視] 圖表，監視單一資料庫的使用率。 如此會帶出您可變更的 [度量] 視窗，只要按一下 [編輯圖表] 按鈕即可。 新增下列度量：

* CPU 百分比
* DTU 百分比
* 資料 IO 百分比
* 資料庫大小百分比

新增這些計量之後，您可以在 [計量] 視窗上含有詳細資訊的 [監視] 圖表中繼續檢視它們。 這四個度量都會顯示與資料庫 **DTU** 相對的平均使用率百分比。 如需服務層的相關詳細資訊，請參閱 [DTU 形式的購買模型](sql-database-service-tiers-dtu.md)和[虛擬核心形式的購買模型](sql-database-service-tiers-vcore.md)文章。  

![資料庫效能的服務層監視。](./media/sql-database-single-database-monitoring/sqldb_service_tier_monitoring.png)

您也可以在效能度量中設定警示。 按一下 [度量] 視窗中的 [新增警示] 按鈕。 遵循精靈的指示以設定警示。 您可以選擇在度量超出或低於特定臨界值時發出警示。

例如，如果您預期資料庫中的工作負載會成長，可以選擇設定電子郵件警示，以便在資料庫的任何效能度量達到 80% 時收到警示。 您可以使用此警示做為早期警告，協助您判斷何時需要切換至更高的計算大小。

效能度量也可協助您判斷您是否能夠降級至較低的計算大小。 假設您使用標準 S2 資料庫，且所有效能度量皆顯示在指定時間內，資料庫平均使用率沒有超過 10%。 則該資料庫可能在標準 S1 中會運作得不錯。 不過，在您決定將資料庫移至較低計算大小前，請先注意暴增或大幅變動的工作負載。

## <a name="improving-database-performance-with-more-resources"></a>使用更多資源提升資料庫效能

最後，如果沒有可改善資料庫效能的可操作項目，您可以變更 Azure SQL Database 中可用的資源數量。 您可以隨時變更單一資料庫的 [DTU 服務層](sql-database-service-tiers-dtu.md)或增加彈性集區的 eDTU。 或者，如果您使用[虛擬核心形式的購買模型](sql-database-service-tiers-vcore.md)，便可以變更服務層或增加配置給資料庫的資源。 
1. 若是單一資料庫，您可以視需要[變更服務層級](sql-database-service-tiers-dtu.md)或[計算資源](sql-database-service-tiers-vcore.md)以提升資料庫效能。
2. 若是多個資料庫，請考慮使用[彈性集區](sql-database-elastic-pool-guidance.md)來自動調整資源。

## <a name="tune-and-refactor-application-or-database-code"></a>調整和重構應用程式或資料庫程式碼

您可以變更應用程式程式碼，針對工作負載以最佳方式使用資料庫、變更索引、強制執行計劃，或使用提示來手動調整資料庫。 在[效能指引主題](sql-database-performance-guidance.md)文章中可找到手動調整或重寫程式碼的一些方針和秘訣。


## <a name="next-steps"></a>後續步驟

- 若要在 Azure SQL Database 中啟用自動調整，並讓自動調整功能完全管理您的工作負載，請參閱[啟用自動調整](sql-database-automatic-tuning-enable.md)。
- 若要使用手動調整，您可以檢閱 [Azure 入口網站中的調整建議](sql-database-advisor-portal.md)並手動套用可改善查詢效能的建議。
- 變更 [Azure SQL Database 服務層](sql-database-performance-guidance.md)來變更資料庫中可用的資源
