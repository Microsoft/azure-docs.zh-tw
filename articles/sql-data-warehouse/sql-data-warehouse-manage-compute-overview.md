---
title: 管理 Azure SQL 資料倉儲中的計算資源 | Microsoft Docs
description: 了解 Azure SQL 資料倉儲中的效能相應放大功能。 藉由調整 DWU 來相應放大，或藉由暫停資料倉儲來降低成本。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 04/17/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 53a801a367e6948c3070224b7ff36a013a1faab3
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43300845"
---
# <a name="manage-compute-in-azure-sql-data-warehouse"></a>管理 Azure SQL 資料倉儲中的計算能力
了解如何管理 Azure SQL 資料倉儲中的計算資源。 藉由暫停資料倉儲來降低成本，或調整資料倉儲規模以符合效能需求。 

## <a name="what-is-compute-management"></a>什麼是計算管理？
SQL 資料倉儲的架構分隔儲存體和計算功能，可單獨進行調整。 因此，您可以調整計算規模以符合與資料儲存體無關的效能需求。 您也可以暫停和繼續計算資源。 這個架構的自然結果為計算[計費](https://azure.microsoft.com/pricing/details/sql-data-warehouse/)與儲存體無關。 如果您有一段時間不需要使用資料倉儲，您可暫停計算以節省計算成本。 

## <a name="scaling-compute"></a>調整計算規模
您可藉由調整資料倉儲的[資料倉儲單位](what-is-a-data-warehouse-unit-dwu-cdwu.md)設定，來相應放大或縮小計算。 當您新增更多資料倉儲單位時，載入和查詢效能可以呈線性增加。 

如需相應放大步驟，請參閱 [Azure 入口網站](quickstart-scale-compute-portal.md)、[PowerShell](quickstart-scale-compute-powershell.md) 或 [T-SQL](quickstart-scale-compute-tsql.md) 快速入門。 您也可以使用 [REST API](sql-data-warehouse-manage-compute-rest-api.md#scale-compute) 來執行相應放大作業。

若要執行調整規模作業，SQL 資料倉儲會先刪除所有傳入的查詢，然後復原交易，以確保一致的狀態。 只有在交易復原完成後，才會發生調整。 在調整規模作業中，系統會使儲存層與計算節點中斷連結，新增計算節點，然後將儲存層重新連結至計算層。 每個資料倉儲會儲存為 60 個散發，其平均分散於各計算節點。 新增更多計算節點可增加更多計算能力。 當計算節點數目增加時，每個計算節點的發佈數目也會隨之增加，進而為您的查詢提供計算能力。 同樣地，減少資料倉儲單位可減少計算節點數目，進而減少查詢的計算資源。

下表顯示當資料倉儲單位變更時，每個計算節點的發佈數目如何隨之變更。  DWU6000 會提供 60 個計算節點，並達到比 DWU100 更高的查詢效能。 

| 資料倉儲單位  | \# 個計算節點 | 每節點 \# 個散發 |
| ---- | ------------------ | ---------------------------- |
| 100  | 1                  | 60                           |
| 200  | 2                  | 30                           |
| 300  | 3                  | 20                           |
| 400  | 4                  | 15                           |
| 500  | 5                  | 12                           |
| 600  | 6                  | 10                           |
| 1000 | 10                 | 6                            |
| 1200 | 12                 | 5                            |
| 1500 | 15                 | 4                            |
| 2000 | 20                 | 3                            |
| 3000 | 30                 | 2                            |
| 6000 | 60                 | 1                            |


## <a name="finding-the-right-size-of-data-warehouse-units"></a>尋找適當大小的資料倉儲單位

若要查看相應放大的效能優勢 (尤其是針對較大的資料倉儲單位)，您可使用至少 1-TB 的資料集。 若要尋找資料倉儲的最佳資料倉儲單位數目，請嘗試相應放大和相應縮小。 在載入您的資料之後，使用不同的資料倉儲單位數目來執行一些查詢。 由於調整很快速，您可以在一小時內嘗各種效能層級。 

尋找最佳資料倉儲單位數目的建議︰

- 若是開發過程中的資料倉儲，可從選取少量的資料倉儲單位開始。  DW400 或 DW200 是不錯的起點。
- 監視應用程式效能，觀察比較所選資料倉儲單位數目與您觀察到的效能。
- 假設線性尺度，並判斷您需要增加或減少多少資料倉儲單位。 
- 繼續進行調整，直到達到您業務需求的最佳效能為止。

## <a name="when-to-scale-out"></a>相應放大時機
相應放大資料倉儲單位會影響效能的下列層面：

- 以線性方式改善掃描、彙總和 CTAS 陳述式的系統效能。
- 針對載入作業增加讀取器和寫入器的數目。
- 並行查詢和並行存取插槽數目上限。

相應放大資料倉儲單位時機的建議︰

- 執行大量資料載入或轉換作業之前，相應放大進而讓資料更快速可供使用。
- 在尖峰營業時間內，相應放大以適應大量的並行查詢。 

## <a name="what-if-scaling-out-does-not-improve-performance"></a>如果相應放大並未改善效能，該怎麼辦？

新增資料倉儲單位可提供平行處理能力。 如果工作平均分割於計算節點之間，額外的平行處理能力可改善查詢效能。 如果相應放大並未改變效能，有一些原因致使發生這種情形。 您的資料可能會呈現偏態分佈，或查詢可能會引進大量的資料移動。 若要調查查詢效能問題，請參閱[效能疑難排解](sql-data-warehouse-troubleshoot.md#performance)。 

## <a name="pausing-and-resuming-compute"></a>暫停和繼續計算
暫停計算會導致儲存層與計算節點中斷連結。 系統會從您的帳戶釋出計算資源。 當計算暫停時，您就不需支付計算費用。 繼續計算會將儲存體重新連結至計算節點，並繼續收取計算費用。 當您暫停資料倉儲時︰

* 計算和記憶體資源會傳回資料中心內的可用資源集區
* 暫停期間的資料倉儲單位成本是零。
* 不會影響資料儲存體，您的資料保持不變。 
* SQL 資料倉儲會取消所有執行中或已排入佇列的作業。

當您處理資料倉儲時︰

* SQL 資料倉儲會取得資料倉儲設定的計算和記憶體資源。
* 繼續資料倉儲單位的計算費用。
* 您的資料變為可供使用。
* 在資料倉儲上線後，您需要重新啟動工作負載查詢。

如果您希望隨時可存取資料倉儲，請考慮將其調整到最小的大小，而不是暫停。 

如需暫停和繼續步驟，請參閱 [Azure 入口網站](pause-and-resume-compute-portal.md) 或 [PowerShell](pause-and-resume-compute-powershell.md) 快速入門。 您也可以使用[暫停 REST API](sql-data-warehouse-manage-compute-rest-api.md#pause-compute) 或[繼續 REST API](sql-data-warehouse-manage-compute-rest-api.md#resume-compute)。

## <a name="drain-transactions-before-pausing-or-scaling"></a>暫停或調整之前先清空交易
我們建議在起始暫停或調整規模作業之前，讓現有的交易先完成。

當您暫停或調整您的 SQL 資料倉儲時，您的查詢在您起始暫停或調整要求時會於幕後取消。  取消簡單的 SELECT 查詢是很快的作業，對於暫停或調整執行個體所花費的時間幾乎沒有什麼影響。  不過，交易性查詢 (會修改您的資料或結構) 可能無法快速地停止。  **顧名思義，交易性查詢必須完全完成或回復變更。**  回復交易性查詢已完成的工作可能需要很長時間，甚至比查詢套用原始變更更久。  例如，如果您取消的刪除資料列查詢已經執行一小時，系統可能需要一個小時將已刪除的資料列回復插入。  如果您在交易執行中執行暫停或調整，暫停或調整作業可能需要一些時間，因為暫停和調整必須等回復完成才能繼續。

另請參閱[了解交易](sql-data-warehouse-develop-transactions.md)和[最佳化交易](sql-data-warehouse-develop-best-practices-transactions.md)。

## <a name="automating-compute-management"></a>自動進行計算管理
若要自動執行計算管理作業，請參閱[使用 Azure 函式管理計算](manage-compute-with-azure-functions.md)。

相應放大、暫停和繼續作業可能需要幾分鐘的時間才能完成。 如果您要自動調整/暫停/繼續，建議實作邏輯以確保在進行其他動作之前，先確認特定作業已完成。 透過各種端點檢查資料倉儲狀態可讓您正確地實作這類作業的自動化。 

若要檢查資料倉儲狀態，請參閱 [PowerShell](quickstart-scale-compute-powershell.md#check-data-warehouse-state) 或 [T-SQL](quickstart-scale-compute-tsql.md#check-data-warehouse-state) 快速入門。 您也可以使用 [REST API](sql-data-warehouse-manage-compute-rest-api.md#check-database-state) 來查看資料倉儲狀態。


## <a name="permissions"></a>權限

調整資料倉儲時需要 [ALTER DATABASE](/sql/t-sql/statements/alter-database-azure-sql-data-warehouse) 中所述的權限。  暫停和繼續則需要 [SQL DB 參與者](../role-based-access-control/built-in-roles.md#sql-db-contributor)權限，特別是 Microsoft.Sql/servers/databases/action。


## <a name="next-steps"></a>後續步驟
管理計算資源的另一個層面就是針對個別查詢配置不同的計算資源。 如需詳細資訊，請參閱[適用於工作負載管理的資源類別](resource-classes-for-workload-management.md)。
