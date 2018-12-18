---
title: 套用效能建議 - Azure SQL Database |Microsoft Docs
description: 使用 Azure 入口網站以找出可最佳化 Azure SQL Database 的效能建議。
services: sql-database
author: danimir
manager: craigg
ms.service: sql-database
ms.custom: monitor & tune
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: v-daljep
ms.reviewer: carlrab
ms.openlocfilehash: 67432c8751cd8809649d41195957a7d68b392074
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43665594"
---
# <a name="find-and-apply-performance-recommendations"></a>尋找和套用效能建議

您可以使用 Azure 入口網站，找出可最佳化 Azure SQL Database 的效能建議，或修正在工作負載中找到的一些問題。 您可以在 Azure 入口網站中的 [效能建議] 頁面找到根據潛在影響所提出排名最前面的建議。 

## <a name="viewing-recommendations"></a>檢視建議

若要檢視和套用效能建議，您在 Azure 中必須有正確的[角色型存取控制](../role-based-access-control/overview.md)權限。 需要**讀取者**、**SQL DB 參與者**權限，才能檢視建議，以及需要**擁有者**、**SQL DB 參與者**權限，才能執行任何動作；建立或卸除索引並取消建立索引。

使用下列步驟在 Azure 入口網站上尋找效能建議：

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 移至 [所有服務] > [SQL Database]，然後選取您的資料庫。
3. 瀏覽至 [效能建議] 來檢視適用於所選資料庫的可用建議。

資料表中顯示的效能建議類似於下圖：

![建議](./media/sql-database-advisor-portal/recommendations.png)

依照可能帶來的效能影響排序，建議分成下列類別：

| 影響 | 說明 |
|:--- |:--- |
| 高 |高影響建議提供最明顯的效能影響。 |
| 中 |中度影響建議會改善效能，但不顯著。 |
| 低 |低影響建議比沒有建議時提供更好的效能，但改善可能不顯著。 |


> [!NOTE]
> Azure SQL Database 必須至少監視活動一整天，才能找出一些建議。 相較於隨機蹦出的零星活動，一致的查詢模式更有利於 Azure SQL Database 最佳化。 如果 [效能建議]  頁面中目前沒有可用的建議，該頁面會提供訊息說明原因。
> 

您也可以檢視歷程記錄作業的狀態。 選取建議或狀態來查看其詳細資訊。

以下是 Azure 入口網站中「建立索引」建議的範例。

![建立索引](./media/sql-database-advisor-portal/sql-database-performance-recommendation.png)

## <a name="applying-recommendations"></a>套用建議
Azure SQL Database 可讓您使用下列 3 個選項的其中任一選項來控制建議的啟用方式： 

* 一次套用一個個別的建議。
* 啟用自動調整功能，以自動套用建議。
* 若要手動實作建議，請針對您的資料庫執行建議的 T-SQL 指令碼。

選取任何建議來檢視其詳細資料，然後按一下 [檢視指令碼]  來檢閱將如何建立建議的確切詳細資料。

套用建議時，資料庫仍保持連線 -- 使用效能建議或自動調整功能不會使資料庫離線。

### <a name="apply-an-individual-recommendation"></a>套用個別的建議
您可以一次檢閱並接受一個建議。

1. 選取 [建議] 頁面上的某個建議。
2. 在 [詳細資料] 頁面上按一下 [套用] 按鈕。
   
    ![套用建議](./media/sql-database-advisor-portal/apply.png)

針對資料庫套用選取的建議。

### <a name="removing-recommendations-from-the-list"></a>從清單中移除建議
如果建議的清單中包含您想從清單中移除的項目，您可以捨棄該建議：

1. 在 [建議] 清單中選取建議，以開啟詳細資料。
2. 在 [詳細資料] 頁面上按一下 [捨棄]。

如有需要，您可以將捨棄的項目加回到 **建議** 清單：

1. 在 [建議] 頁面上按一下 [檢視已捨棄]。
2. 從清單中選取捨棄的項目以檢視其詳細資料。
3. (選擇性) 按一下 [復原捨棄]，將索引加回到**建議**的主要清單。

> [!NOTE]
> 請注意，如果啟用 SQL Database [自動微調](sql-database-automatic-tuning.md)，且您以手動方式捨棄清單中的建議，就永遠不會自動套用這類建議。 捨棄建議是一個便利的方式，可在要求不得套用特定建議時，讓使用者可以啟用自動調整。
> 您可以選取 [復原捨棄] 選項，將捨棄的建議新增回 [建議] 清單，從而還原這個行為。
> 

### <a name="enable-automatic-tuning"></a>啟用自動微調
您可以將 Azure SQL Database 設為自動實作建議。 當建議可供使用時會自動套用建議。 因為所有建議都由服務管理，所以若對效能產生負面影響，就會還原該建議。

1. 在 [建議] 頁面上按一下 [自動化]：
   
    ![建議程式設定](./media/sql-database-advisor-portal/settings.png)
2. 選取要自動執行的動作：
   
    ![建議的索引](./media/sql-database-automatic-tuning-enable/server.png)

> [!NOTE]
> 請注意，此時 [DROP_INDEX] 選項與使用分割區切換和索引提示的應用程式不相容，而且不應該在這些情況下啟用。
>

選取所需的組態後，按一下 [套用]。

### <a name="manually-run-the-recommended-t-sql-script"></a>手動執行建議的 T-SQL 指令碼
選取任何建議，然後按一下 [檢視指令碼] 。 對資料庫執行這個指令碼，以手動套用建議。

 ，因此建議您在建立這些索引之後監視索引，以確認它們能夠提高效能，且於必要時調整或刪除它們。 如需有關建立索引的詳細資訊，請參閱 [CREATE INDEX (Transact-SQL)](https://msdn.microsoft.com/library/ms188783.aspx)。

### <a name="canceling-recommendations"></a>取消建議
可以取消處於**擱置中**、**驗證中**或**成功**狀態的建議。 狀態為 **執行中** 的建議無法取消。

1. 在 [調整歷程記錄] 區域中選取建議，以開啟 [建議詳細資料] 頁面。
2. 按一下 [取消]  以中止套用建議的程序。

## <a name="monitoring-operations"></a>監視作業
套用建議時可能不會立即執行。 入口網站會提供有關建議狀態的詳細資料。 索引有下列可能的狀態：

| 狀態 | 說明 |
|:--- |:--- |
| Pending |已收到套用建議命令，且已排程執行。 |
| 執行中 |正在套用建議。 |
| Validating |成功套用建議，而服務正在衡量益處。 |
| 成功 |已成功套用建議，並證實有益處。 |
| Error |套用建議程序期間發生錯誤。 這可能是暫時性問題，也可能是資料表的結構描述變更，造成指令碼不再有效。 |
| 還原 |已套用建立但被認為無助於效能，正在自動還原。 |
| 已還原 |已還原建議。 |

按一下清單中正在處理的建議以查看其詳細資訊：

![建議的索引](./media/sql-database-advisor-portal/operations.png)

### <a name="reverting-a-recommendation"></a>還原建議
如果您使用效能建議來套用建議 (表示您未手動執行 T-SQL 指令碼)，如果建議程式發現會對效能造成負面影響，它將會自動還原變更。 如果您因為任何原因想要還原建議，您可以執行以下步驟：

1. 在 [調整歷程記錄]  區域中選取已成功套用的建議。
2. 在 [建議詳細資料] 頁面上按一下 [還原]。

![建議的索引](./media/sql-database-advisor-portal/details.png)

## <a name="monitoring-performance-impact-of-index-recommendations"></a>監視索引建議的效能影響
成功實作建議之後 (目前僅提供索引作業和參數化查詢建議)，您可以按一下 [建議詳細資料] 頁面上的 **查詢深入解析** 來開啟 [查詢效能深入解析](sql-database-query-performance.md)，並查看排名最前面查詢對效能的影響。

![監視效能影響](./media/sql-database-advisor-portal/query-insights.png)

## <a name="summary"></a>總結
Azure SQL Database 會提供可改善 SQL Database 效能的建議。 藉由提供 T-SQL 指令碼，您會獲得最佳化資料庫的協助，並最終改善查詢效能。

## <a name="next-steps"></a>後續步驟
監視建議，並繼續套用建議以改善效能。 資料庫工作負載會動態地持續變更。 Azure SQL Database 會繼續監視並提供可能改善資料庫效能的建議。 

* 若要深入了解 Azure SQL Database 中的自動調整功能，請參閱[自動調整](sql-database-automatic-tuning.md)。
* 如需 Azure SQL Database 效能建議的概觀，請參閱[效能建議](sql-database-advisor.md)。
* 請參閱[查詢效能深入解析](sql-database-query-performance.md)，以了解如何檢視排名最前面查詢的效能影響。

## <a name="additional-resources"></a>其他資源
* [查詢存放區](https://msdn.microsoft.com/library/dn817826.aspx)
* [CREATE INDEX](https://msdn.microsoft.com/library/ms188783.aspx)
* [角色型存取控制](../role-based-access-control/overview.md)

