---
title: Azure SQL Database 的查詢效能深入解析 | Microsoft Docs
description: 查詢效能監視可識別 Azure SQL Database 的大部分 CPU 取用的查詢。
services: sql-database
author: danimir
manager: craigg
ms.service: sql-database
ms.custom: monitor & tune
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: v-daljep
ms.reviewer: carlrab
ms.openlocfilehash: 5069b4e69c53ed93e9018cef2517f6125b838d12
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39524160"
---
# <a name="azure-sql-database-query-performance-insight"></a>Azure SQL Database 查詢效能深入解析
管理和調整關聯式資料庫效能是一項具挑戰性的工作，需要投入大量的專業知識和時間。 「查詢效能深入解析」提供了下列各項，讓您得以花費較少時間來對資料庫效能進行疑難排解：

* 更深入的資料庫資源 (DTU) 取用分析。 
* 依 CPU/持續時間/執行計數排名最前面的查詢，進行微調有可能會改善效能。
* 能夠向下鑽研查詢的詳細資料，以檢視其文字和資源使用量的歷程記錄。 
* 效能微調註解，可顯示 [SQL Azure Database 建議程式](sql-database-advisor.md)  



## <a name="prerequisites"></a>必要條件
* 「查詢效能深入解析」要求 [查詢存放區](https://msdn.microsoft.com/library/dn817826.aspx) 在您的資料庫上為作用中狀態。 如果查詢存放區不在執行中，則入口網站會提示您將它開啟。

## <a name="permissions"></a>權限
需有下列 [角色型存取控制](../role-based-access-control/overview.md) 權限，才能使用「查詢效能深入解析」︰ 

* 檢視排名最前面的資源取用查詢和圖表時，需具備**讀取器**、**擁有者**、**參與者**、**SQL DB 參與者** 或 **SQL Server 參與者** 權限。 
* 檢視查詢文字時，需具備**擁有者**、**參與者**、**SQL DB 參與者**或 **SQL Server 參與者**權限。

## <a name="using-query-performance-insight"></a>使用查詢效能深入解析
「查詢效能深入解析」很容易使用︰

* 開啟 [Azure 入口網站](https://portal.azure.com/) ，然後尋找您想要檢查的資料庫。 
  * 從左側功能表中，[支援和疑難排解] 下方，選取 [查詢效能深入解析]。
* 在第一個索引標籤上，檢閱排名最前面的資源取用查詢清單。
* 選取個別的查詢來檢視其詳細資料。
* 開啟 [SQL Azure Database 建議程式](sql-database-advisor.md) ，並查看是否有任何可用的建議。
* 使用滑桿或縮放圖示來變更所觀測的間隔。
  
    ![效能儀表板](./media/sql-database-query-performance/performance.png)

> [!NOTE]
> SQL Database 的查詢存放區需要擷取數個小時的資料，才能提供查詢效能深入解析。 如果在某段時間內資料庫沒有任何作用中的活動或查詢存放區，則在顯示該期間時圖表會是空的。 如果查詢存放區不在執行中，您可隨時加以啟用。   
> 
> 

## <a name="review-top-cpu-consuming-queries"></a>檢閱排名最前面的 CPU 取用查詢
在 [入口網站](http://portal.azure.com) 中執行下列動作：

1. 瀏覽至 SQL Database，然後按一下 [所有設定]  >  [支援 + 疑難排解]  >  [查詢效能深入解析]。 
   
    ![查詢效能深入解析][1]
   
    排名最前面的查詢檢視會隨即開啟，並列出排名最前面的 CPU 取用查詢。
2. 按一下圖表周圍以取得詳細資料。<br>最上面的線條顯示資料庫的整體 DTU %，而長條則顯示已選取查詢在所選間隔內 (例如，如果已選取 [過去一週]，則每個長條代表一天) 所取用的 CPU %。
   
    ![排名最前面的查詢][2]
   
    底部格線則代表可見查詢的彙總資訊。
   
   * 查詢識別碼 - 資料庫內部查詢的唯一識別碼。
   * 在可觀察時間間隔期間每個查詢的 CPU (取決於彙總函式)。
   * 每個查詢的持續時間 (取決於彙總函式)。
   * 特定查詢的總執行次數。
     
     使用核取方塊，選取或清除圖表要包含或排除的個別查詢。
3. 如果您的資料過期了，請按一下 [重新整理]  按鈕。
4. 您可以使用滑桿和縮放按鈕來變更觀測間隔，並調查尖峰︰![設定](./media/sql-database-query-performance/zoom.png)
5. (選擇性) 如果您想要不同的檢視，您可以選取 [自訂]  索引標籤，然後設定：
   
   * 度量 (CPU、持續時間、執行計數)
   * 時間間隔 (過去 24 小時、過去一週、過去一個月)。 
   * 查詢數目。
   * 彙總函式。
     
     ![settings](./media/sql-database-query-performance/custom-tab.png)

## <a name="viewing-individual-query-details"></a>檢視個別查詢詳細資料
若要檢視查詢詳細資料︰

1. 按一下排名最前面的查詢清單中的任何查詢。
   
    ![詳細資料](./media/sql-database-query-performance/details.png)
2. 詳細資料檢視隨即開啟，而查詢的 CPU 取用量/持續時間/執行計數會根據時間來細分。
3. 按一下圖表周圍以取得詳細資料。
   
   * 最上層圖表顯示一條含有整體資料庫 DTU % 的線條，而長條是已選取查詢所耗用的 CPU %。
   * 第二個圖表顯示已選取查詢的總持續時間。
   * 底層圖表則顯示已選取查詢的執行總數。
     
     ![查詢詳細資料][3]
4. (選擇性) 使用滑桿、縮放按鈕或按一下 [設定]  ，來自訂查詢資料的顯示方式或挑選不同的期間。

## <a name="review-top-queries-per-duration"></a>針對每段持續時間檢閱排名最前面的查詢
在「查詢效能深入解析」的最新查詢中，我們引進了兩個新的度量，可協助您識別潛在的瓶頸︰持續時間和執行計數。<br>

長時間執行的查詢最有可能會鎖定資源較長的時間、封鎖其他使用者，以及限制擴充性。 它們也是最適合進行最佳化的項目。<br>

識別長時間執行的查詢：

1. 針對選取的資料庫，在 [查詢效能深入解析] 中開啟 [自訂]  索引標籤
2. 將度量變更為 [持續時間] 
3. 選取查詢數目和觀測間隔
4. 選取彙總函式
   
   * **Sum** 會加總整個觀測間隔內的所有查詢執行時間。
   * **Max** 會尋找在整個觀測間隔內執行時間最大的查詢。
   * **Avg** 會尋找所有查詢執行的平均執行時間，並顯示這些平均值中排名最高者。 
     
     ![查詢持續時間][4]

## <a name="review-top-queries-per-execution-count"></a>針對每個執行計數檢閱排名最前面的查詢
執行數目很高可能不會影響資料庫本身且資源使用量可能很低，但整體應用程式可能會變慢。

在某些情況下，極高的執行計數可能會導致網路往返次數增加。 往返次數會大幅影響效能。 它們會受限於網路延遲和下游伺服器延遲。 

例如，許多資料導向的網站會針對每個使用者要求大量存取資料庫。 儘管有連線集區的協助，但是資料庫伺服器上增加的網路流量和處理負載可能會對效能產生不利的影響。  通常會建議您將往返次數維持在絕對最小值。

識別經常執行查詢 (「多對話」) 的查詢：

1. 針對選取的資料庫，在 [查詢效能深入解析] 中開啟 [自訂]  索引標籤
2. 將度量變更為 [執行計數] 
3. 選取查詢數目和觀測間隔
   
    ![查詢執行計數][5]

## <a name="understanding-performance-tuning-annotations"></a>了解效能微調註解
在「查詢效能深入解析」中瀏覽您的工作負載時，您可能會注意到圖表最上方含有垂直線的圖示。<br>

這些圖示是註解。它們代表 [SQL Azure Database 建議程式](sql-database-advisor.md)所執行會影響效能的動作。 您可以藉由停留在註解上方來取得與動作有關的基本資訊：

![查詢註解][6]

如果您想要深入了解或套用建議程式的建議，請按一下圖示。 隨即會開啟動作的詳細資料。 如果是有效的建議，您就能使用命令立即套用動作。

![查詢註解詳細資料][7]

### <a name="multiple-annotations"></a>多個註解。
這是可能的，由於縮放等級的緣故，使得彼此相近的註解將會摺疊成一個。 這將會利用特殊的圖示來表示，按一下該圖示就會開啟新的刀鋒視窗，其中顯示已分組的註解清單。
使查詢和效能調整動作相互關聯，有助於深入了解您的工作負載。 

## <a name="optimizing-the-query-store-configuration-for-query-performance-insight"></a>針對查詢效能深入解析來讓查詢存放區組態最佳化
您在使用查詢效能深入解析的期間，可能會看到下列查詢存放區訊息：

* 「此資料庫的查詢存放區未正確設定。 請按一下這裡深入了解。」
* 「此資料庫的查詢存放區未正確設定。 請按一下這裡變更設定。」 

這些訊息通常會在查詢存放區無法收集新資料時出現。 

當查詢存放區處於唯讀狀態且以最佳方式設定參數時，就會發生第一種狀況。 您可以藉由增加查詢存放區的大小，或清除查詢存放區來修正此問題。

![qds 按鈕][8]

當查詢存放區已關閉或參數並不是以最佳方式所設定時，就會發生第二種狀況。 <br>您可以執行下列命令或直接從入口網站，變更保留期和擷取原則並啟用查詢存放區：

![qds 按鈕][9]

### <a name="recommended-retention-and-capture-policy"></a>建議使用的保留期和擷取原則
保留期原則有兩種：

* 容量大小式：設定為 [AUTO] 時，該原則會在收集的資料量接近容量上限時，自動清除資料。
* 時間式：預設設定為 30 天，這代表當查詢存放區的空間用完時，就會刪除 30 天之前的查詢資訊

擷取原則可以設定為：

* **All**：擷取所有的查詢。
* **Auto**：會忽略不常執行的查詢，以及編譯和執行持續時間微不足道的查詢。 執行次數、編譯及執行階段持續時間的臨界值是由內部決定的。 這是預設選項。
* **None**：「查詢存放區」會停止擷取新的查詢，不過仍然會收集已擷取查詢的執行階段統計資料。

我們建議您將所有原則設定為 [AUTO]，並將清除原則設定為 30 天：

    ALTER DATABASE [YourDB] 
    SET QUERY_STORE (SIZE_BASED_CLEANUP_MODE = AUTO);

    ALTER DATABASE [YourDB] 
    SET QUERY_STORE (CLEANUP_POLICY = (STALE_QUERY_THRESHOLD_DAYS = 30));

    ALTER DATABASE [YourDB] 
    SET QUERY_STORE (QUERY_CAPTURE_MODE = AUTO);

增加查詢存放區的大小， 方法是連線至資料庫，然後發出下列查詢：

    ALTER DATABASE [YourDB]
    SET QUERY_STORE (MAX_STORAGE_SIZE_MB = 1024);

套用這些設定，最終會使查詢存放區收集新的查詢，不過，如果您不想等待，可以清除查詢存放區。 

> [!NOTE]
> 執行下列查詢將會刪除查詢存放區中所有目前的資訊。 
> 
> 

    ALTER DATABASE [YourDB] SET QUERY_STORE CLEAR;


## <a name="summary"></a>總結
「查詢效能深入解析」可協助您了解您的查詢工作負載的影響，以及其與資料庫資源取用量的關係。 使用此功能，您將了解排名最前面的取用查詢，並且在發生問題前輕鬆地找出要修正的項目。

## <a name="next-steps"></a>後續步驟
如需有關改善 SQL Database 效能的其他建議，請按一下 [查詢效能深入解析] 刀鋒視窗上的 [建議](sql-database-advisor.md)。

![效能建議程式](./media/sql-database-query-performance/ia.png)

<!--Image references-->
[1]: ./media/sql-database-query-performance/tile.png
[2]: ./media/sql-database-query-performance/top-queries.png
[3]: ./media/sql-database-query-performance/query-details.png
[4]: ./media/sql-database-query-performance/top-duration.png
[5]: ./media/sql-database-query-performance/top-execution.png
[6]: ./media/sql-database-query-performance/annotation.png
[7]: ./media/sql-database-query-performance/annotation-details.png
[8]: ./media/sql-database-query-performance/qds-off.png
[9]: ./media/sql-database-query-performance/qds-button.png

