---
title: 檢視或分析所收集的 Azure Log Analytics 資料 | Microsoft Docs
description: 本文包含一篇教學課程，說明如何使用記錄搜尋入口網站來建立記錄搜尋，以及分析儲存在 Log Analytics 資源的資料。  教學課程包含執行一些簡單查詢以傳回不同類型的資料並分析結果。
services: log-analytics
documentationcenter: log-analytics
author: mgoedtel
manager: carmonm
editor: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 07/31/2018
ms.author: magoedte
ms.custom: mvc
ms.component: na
ms.openlocfilehash: b5d7b71b76eebc0c14fe1403791c3d4b6cefd7f4
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161032"
---
# <a name="view-or-analyze-data-collected-with-log-analytics-log-search"></a>檢視或分析以 Log Analytics 記錄搜尋所收集的資料

在 Log Analytics 中，您可以建構查詢以分析收集到的資料來利用記錄搜尋，使用可以最有價值搜尋的圖形檢視來自訂的既存儀表板。  您現在已定義從 Azure VM 和活動記錄收集的操作資料，在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 執行事件資料的簡單搜尋，並使用功能來修改和篩選結果 
> * 瞭解如何使用效能資料

若要完成本教學課程中的範例，您目前必須有[連線至 Log Analytics 工作區](log-analytics-quick-collect-azurevm.md)的虛擬機器。  

除了以互動方式使用傳回的資料之外，還可以使用其中一種方式建立和編輯查詢。  若是基本查詢，請在 Azure 入口網站中使用 [記錄搜尋] 頁面，若是進階查詢，則可以使用進階分析入口網站。 若要深入了解兩個入口網站之間的功能差異，請參閱 [Azure Log Analytics 中用於建立和編輯記錄查詢的入口網站](log-analytics-log-search-portals.md)

在本教學課程中，我們將在 Azure 入口網站中使用 [記錄搜尋]。 

## <a name="log-in-to-azure-portal"></a>登入 Azure 入口網站
在 [https://portal.azure.com](https://portal.azure.com) 上登入 Azure 入口網站。 

## <a name="open-the-log-search-portal"></a>開啟記錄搜尋入口網站 
從開啟記錄搜尋入口網站開始。   

1. 在 Azure 入口網站中，按一下 [所有服務]。 在資源清單中輸入 [監視器]。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [監視器]。
2. 在 [監視器] 導覽功能表上，選取 [Log Analytics]，然後選取工作區。

## <a name="create-a-simple-search"></a>建立簡單搜尋
若要擷取某些資料來使用，最快的方式是使用會傳回資料表中所有記錄的簡單查詢。  如果有任何 Windows 或 Linux 用戶端連線到您的工作區，則您會擁有事件 (Windows) 或 Syslog (Linux) 資料表中的資料。

在搜尋方塊中輸入以下其中一項查詢，然後按一下搜尋按鈕。  

```
Event
```
```
Syslog
```

資料會傳回到預設清單檢視中，您可以看到傳回的記錄總數。

![簡單查詢](media/log-analytics-tutorial-viewdata/log-analytics-portal-eventlist-01.png)

每筆記錄只會顯示前幾個屬性。  按一下 [顯示更多] 以顯示特定記錄的所有屬性。

## <a name="filter-results-of-the-query"></a>篩選查詢結果
畫面左側是篩選窗格，可讓您新增篩選條件到查詢中，而不需要直接修改查詢。  對於該記錄類型會顯示數個記錄屬性，您可以選取一個或多個屬性值，以縮小搜尋結果。

如果您使用的是**事件**，選取 [EVENTLEVELNAME] 下方 [錯誤] 旁的核取方塊。   如果您使用的是 **Syslog**，選取 [SEVERITYLEVEL] 下方 [錯誤] 旁的核取方塊。  這會將查詢變更為下列其中一個，以將結果限制為錯誤事件。

```
Event | where (EventLevelName == "Error")
```
```
Syslog | where (SeverityLevel == "err")
```

![Filter](media/log-analytics-tutorial-viewdata/log-analytics-portal-eventlist-02.png)

從其中一個記錄的屬性功能表中選取 [新增至篩選器]，將屬性新增至篩選窗格。

![新增至篩選功能表](media/log-analytics-tutorial-viewdata/log-analytics-portal-eventlist-03.png)

您可以從記錄的屬性功能表選取具有所要篩選值的 [篩選]，以設定相同的篩選條件。  

只有在將滑鼠移至上方時，名稱是藍色的屬性才會有 [篩選] 選項。  這些是可搜尋的欄位，已針對搜尋條件編列索引。  灰色的欄位是「自然語言檢索搜尋旗標」欄位，只有 [顯示參考] 選項。  此選項會傳回在任何屬性中具有該值的記錄。

您可以選取記錄功能表中的 [分組依據] 選項，在單一屬性上群組結果。  這會將[摘要](/azure/kusto/query/summarizeoperator)運算子新增到查詢中，可在圖表中顯示結果。  您可以群組一個以上的屬性，但需要直接編輯查詢。  選取**電腦**屬性旁的記錄功能表，並選取 [Group by 'Computer'] \(依「電腦」分組\)。  

![依電腦分組](media/log-analytics-tutorial-viewdata/log-analytics-portal-eventlist-04.png)

## <a name="work-with-results"></a>處理結果
記錄搜尋入口網站有各種功能，以供使用查詢結果。  您可以排序、篩選和群組結果來分析資料，而不需修改實際的查詢。  根據預設，不會排序查詢的結果。

若要以可提供其他篩選和排序選項的資料表形式來檢視資料，按一下 [資料表]。  

![資料表檢視](media/log-analytics-tutorial-viewdata/log-search-portal-table-01.png)

按一下記錄旁的箭號以檢視該記錄的詳細資訊。

![排序結果](media/log-analytics-tutorial-viewdata/log-search-portal-table-02.png)

按一下資料行標題，對欄位進行排序。

![排序結果](media/log-analytics-tutorial-viewdata/log-search-portal-table-03.png)

按一下篩選按鈕並提供篩選條件，以篩選出資料行中具特定值的結果。

![篩選結果](media/log-analytics-tutorial-viewdata/log-search-portal-table-04.png)

將資料行標題拖曳至結果上方，以群組資料行。  您可以將多個資料行拖曳至上方，以群組多個欄位。

![群組結果](media/log-analytics-tutorial-viewdata/log-search-portal-table-05.png)


## <a name="work-with-performance-data"></a>使用效能資料
Windows 和 Linux 代理程式的效能資料都儲存在 Log Analytics 工作區的**效能**資料表中。  效能記錄看起來就像其他任何記錄，我們即將撰寫會傳回所有效能記錄的簡單查詢，就像使用事件一樣。

```
Perf
```

![效能資料](media/log-analytics-tutorial-viewdata/log-analytics-portal-perfsearch-01.png)

針對所有效能物件和計數器傳回數百萬筆記錄並不太實用。  您可以使用與上述相同的方法來篩選資料，或直接在 [記錄搜尋] 方塊中輸入下列查詢。  這對於 Windows 和 Linux 電腦都只會傳回處理器使用率記錄。

```
Perf | where ObjectName == "Processor"  | where CounterName == "% Processor Time"
```

![處理器使用率](media/log-analytics-tutorial-viewdata/log-analytics-portal-perfsearch-02.png)

這可讓資料限制在特定的計數器，但仍無法以非常實用的形式來呈現資料。  您可透過折線圖顯示資料，但首先需要以 [電腦] 與 [TimeGenerated] 進行群組。  若要群組多個欄位，您需要直接修改查詢，因此，請將查詢修改如下。  這是在 **CounterValue** 屬性上使用 [avg](/azure/kusto/query/avg-aggfunction) 函式來計算每小時的平均值。

```
Perf  
| where ObjectName == "Processor"  | where CounterName == "% Processor Time"
| summarize avg(CounterValue) by Computer, TimeGenerated
```

![效能資料圖表](media/log-analytics-tutorial-viewdata/log-analytics-portal-perfsearch-03.png)

資料既已適當分組，您可以新增[轉譯](/azure/kusto/query/renderoperator)運算子，以視覺圖表來顯示資料。  

```
Perf  
| where ObjectName == "Processor" | where CounterName == "% Processor Time" 
| summarize avg(CounterValue) by Computer, TimeGenerated 
| render timechart
```

![折線圖](media/log-analytics-tutorial-viewdata/log-analytics-portal-linechart-01.png)

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何建立基本的記錄搜尋來分析事件和效能資料。  前進到下一個教學課程，以了解如何建立儀表板，以視覺化方式呈現資料。

> [!div class="nextstepaction"]
> [建立和共用 Log Analytics 儀表板](log-analytics-tutorial-dashboards.md)
