---
title: 在適用於 PostgreSQL 的 Azure 資料庫中進行監視和微調的教學課程
description: 此教學課程會逐步引導您執行在適用於 PostgreSQL 的 Azure 資料庫中進行監視和微調的方法。
services: postgresql
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: tutorial
ms.date: 09/24/2018
ms.openlocfilehash: ef8809afa2da36f44384d784e32532b7b121e3a9
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49379139"
---
# <a name="tutorial-monitor-and-tune-azure-database-for-postgresql"></a>教學課程：針對適用於 PostgreSQL 的 Azure 資料庫進行監視和微調

適用於 PostgreSQL 的 Azure 資料庫具有能協助您了解並改善伺服器效能的功能。 在本教學課程中，您將了解如何：
> [!div class="checklist"]
> * 啟用查詢及等候統計資料的收集
> * 存取並運用所收集的資料
> * 檢視查詢效能與隨時間變化的等候統計資料
> * 分析資料庫以取得效能建議
> * 套用效能建議

## <a name="before-you-begin"></a>開始之前
您需要有 PostgreSQL 9.6 版或 10 版的適用於 PostgreSQL 的 Azure 資料庫伺服器。 您可以依照[建立教學課程](tutorial-design-database-using-azure-portal.md)中的步驟來建立伺服器。

> [!IMPORTANT]
> **查詢存放區**、**查詢效能深入解析**以及**效能建議**都處於公開預覽狀態。

## <a name="enabling-data-collection"></a>啟用資料收集
[查詢存放區](concepts-query-store.md)能擷取伺服器上查詢及等候統計資料的歷程記錄，並將其儲存在您伺服器上的 **azure_sys** 資料庫中。 它是選擇加入的功能。 若要啟用它：

1. 開啟 Azure 入口網站。

2. 選取適用於 PostgreSQL 的 Azure 資料庫伺服器。

3. 在左側功能表的 [設定] 區段中，選取 [伺服器參數]。

4. 將 [pg_qs.query_capture_mode] 設定為 [TOP] 以開始收集查詢效能資料。 將 [pgms_wait_sampling.query_capture_mode] 設定為 [ALL] 以開始收集等候統計資料。 儲存。
   
   ![查詢存放區伺服器參數](./media/tutorial-performance-intelligence/query-store-parameters.png)

5. 請等候 20 分鐘，以讓第一批資料保存在 **azure_sys** 資料庫中。


## <a name="performance-insights"></a>效能深入解析
Azure 入口網站中的[查詢效能深入解析](concepts-query-performance-insight.md)檢視會以視覺效果呈現來自查詢存放區的重要資訊。 

1. 在適用於 PostgreSQL 的 Azure 資料庫伺服器的入口網站頁面中，在左側功能表的 [支援與疑難排解] 區段下，選取 [查詢效能深入解析]。

2. [長時間執行的查詢] 索引標籤會每隔 15 分鐘彙總一次，依每次執行的平均持續時間，顯示前 5 個查詢。 
   
   ![[查詢效能深入解析] 登陸頁面](./media/tutorial-performance-intelligence/query-performance-insight-landing-page.png)

   您可以從 [查詢數目] 下拉式清單中選取，以檢視更多查詢。 當您這樣做時，特定查詢識別碼的圖表色彩可能會有所變更。

3. 您可以在圖表中按一下並拖曳來縮小到特定時間範圍。

4. 使用放大和縮小圖示來分別檢視一段較短或較長的時間。

5. 檢視圖表下方的資料表以了解該時間範圍內長時間執行之查詢的詳細資訊。

6. 選取 [等候統計資料] 索引標籤，以檢視伺服器中等候的對應視覺效果。
   
   ![查詢效能深入解析等候統計資料](./media/tutorial-performance-intelligence/query-performance-insight-wait-statistics.png)

### <a name="permissions"></a>權限
需要**擁有者**或**參與者**權限，才能檢視查詢效能深入解析中的查詢文字。 **讀者**可以檢視圖表與資料表，但無法檢視查詢文字。


## <a name="performance-recommendations"></a>效能建議
[效能建議](concepts-performance-recommendations.md)功能可分析整部伺服器的工作負載，來找出可能可以改善效能的索引。

1. 在 PostgreSQL 伺服器的 Azure 入口網站頁面上，從功能表列的 [支援與疑難排解] 區段開啟 [效能建議]。
   
   ![[效能建議] 登陸頁面](./media/tutorial-performance-intelligence/performance-recommendations-landing-page.png)

2. 選取 [分析] 並選擇資料庫。 隨即開始分析。

3. 根據您的工作負載，這可能需要幾分鐘才能完成。 分析完成後，入口網站中會有通知。

4. [效能建議] 視窗會以清單顯示所找到的任何建議。 

5. 建議會顯示 [資料庫]、[資料表]、[資料行] 與 [索引大小] 等相關資訊。

   ![效能建議結果](./media/tutorial-performance-intelligence/performance-recommendations-result.png)

6. 若要實作建議，請複製查詢文字並從您選擇的用戶端中執行該文字。

### <a name="permissions"></a>權限
需要**擁有者**或**參與者**權限，才能使用 [效能建議] 功能執行分析。

## <a name="next-steps"></a>後續步驟
- 深入了解在適用於 PostgreSQL 的 Azure 資料庫中進行[監視和微調](concepts-monitoring.md)。