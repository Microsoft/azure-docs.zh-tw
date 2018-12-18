---
title: 在 Azure 入口網站中開始使用 Log Analytics | Microsoft Docs
description: 本文提供教學課程來說明如何在 Analytics 入口網站中使用 Log Analytics 撰寫查詢。
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/20/2018
ms.author: bwren
ms.component: na
ms.openlocfilehash: 3bcc5368a99dc7c7c32381ca0226119d81fc2c0a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978194"
---
# <a name="get-started-with-log-analytics-in-the-azure-portal"></a>在 Azure 入口網站中開始使用 Log Analytics

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

在本教學課程中，您會了解如何在 Azure 入口網站中使用 Log Analytics 頁面 (目前為預覽功能) 來撰寫 Log Analytics 查詢。 它會告訴您如何：

- 撰寫簡單的查詢
- 了解資料的結構描述
- 篩選、排序和群組結果
- 套用時間範圍
- 建立圖表
- 儲存及載入查詢
- 匯出及共用查詢


## <a name="meet-the-log-analytics-page"></a>認識 Log Analytics 頁面 
Log Analytics 頁面是一種 Web 工具，可用來撰寫及執行 Azure Log Analytics 查詢。 在 Log Analytics 功能表中選取 [記錄 (預覽)] 來加以開啟。 它會從新的空白查詢來開始。

![首頁](media/get-started-analytics-portal/homepage.png)



## <a name="basic-queries"></a>基本查詢
查詢可用來搜尋字詞、識別趨勢、分析模式，以及提供許多其他以資料為基礎的深入解析。 請從基本查詢來開始：

```Kusto
Event | search "error"
```

此查詢會搜尋「Event」資料表，看看其中的記錄是否有任何屬性包含「error」字詞。

查詢可以透過資料表名稱或 **search** 命令來開始。 上述範例會從「Event」資料表名稱來開始，其會定義查詢的範圍。 垂直線 (|) 字元會分隔命令，下列命令輸入中第一個項目的輸出也是如此。 您可以在單一查詢中新增任意數目的命令。

另一種撰寫該相同查詢的方式會是：

```Kusto
search in (Event) "error"
```

在此範例中，**search** 的範圍會侷限在「Event」資料表，系統會對該資料表中的所有記錄進行搜尋，看看是否有「error」字詞。

## <a name="running-a-query"></a>執行查詢
按一下 [執行] 按鈕或按 **Shift+Enter**，即可執行查詢。 請考慮下列詳細資料，以決定要執行的程式碼和傳回的資料：

- 分行符號：分行可讓查詢變得更清楚。 多個分行符號可將查詢分割為不同的查詢。
- 游標：將游標放在查詢內的某個地方即可執行查詢。 系統會將目前的查詢視為程式碼，直到其發現空白行。
- 時間範圍：預設會設定「過去 24 小時」的時間範圍。 若要使用不同範圍，請使用時間選擇器，或在查詢中新增明確的時間範圍篩選條件。


## <a name="understand-the-schema"></a>了解結構描述
結構描述會將以視覺化方式分組在邏輯類別之下的資料表集合起來。 有幾個類別來自監視解決方案。 LogManagement 類別包含一般資料，例如 Windows 和 Syslog 事件、效能資料與用戶端活動訊號。

![結構描述](media/get-started-analytics-portal/schema.png)

在每個資料表中，資料會組織到不同資料類型 (如資料行名稱旁的圖示所示) 的資料行中。 例如，螢幕擷取畫面中所示的「Event」資料表就包含了「Computer」資料行 (這是文字)、「EventCategory」資料行 (這是數字)，和「TimeGenerated」資料行 (這是日期/時間)。

## <a name="filter-the-results"></a>篩選結果
一開始請先取得「Event」資料表中的所有項目。

```Kusto
Event
```

Log Analytics 頁面會自動根據下列條件來限制結果的範圍：

- 時間範圍：根據預設，系統會將查詢限制在過去 24 小時。
- 結果數：結果數上限為 10,000 筆記錄。

這是一般查詢，會傳回太多結果，所以並不實用。 您可以透過資料表元素來篩選結果，也可以明確地對查詢新增篩選來篩選結果。 透過資料表元素來篩選結果適用於現有結果集，而針對查詢本身的篩選則會傳回新的篩選結果集，並可能因此產生更精確的結果。

### <a name="add-a-filter-to-the-query"></a>對查詢新增篩選
每筆記錄的左邊會有一個箭號。 按一下此箭號就能開啟特定記錄的詳細資料。

將游標停在某個資料行名稱上即可顯示 "+" 和 "-" 圖示。 若要新增篩選，只傳回具有相同值的記錄，請按一下 "+" 符號。 按一下 "-" 則可排除具有此值的記錄，然後按一下 [執行] 以再次執行查詢。

![將篩選新增至查詢](media/get-started-analytics-portal/add-filter.png)

### <a name="filter-through-the-table-elements"></a>對各個資料表元素進行篩選
現在，讓我們將注意力放在嚴重性為「錯誤」的事件上。 名為「EventLevelName」的資料行會指出嚴重性。 您必須捲動到右邊才能看到此資料行。

按一下資料行標題旁的 [篩選] 圖示，然後在快顯視窗中選取「開頭為」「錯誤」文字的值：

![Filter](media/get-started-analytics-portal/filter.png)


## <a name="sort-and-group-results"></a>排序和群組結果
結果現在會縮小為只包含過去 24 小時內所建立、來自 SQL Server 的錯誤事件。 不過，結果並未進行任何排序。 若要依特定資料行來排序結果 (舉例來說，「時間戳記」)，請按一下資料行標題。 按一下會依遞增順序排序，按第二下則會依遞減順序排序。

![排序資料行](media/get-started-analytics-portal/sort-column.png)

群組是另一種組織結果的方式。 若要依特定資料行來群組結果，只要將資料行標頭拖曳到其他資料行上方即可。 若要建立子群組，也請將其他資料行拖曳到上方列。

![群組](media/get-started-analytics-portal/groups.png)

## <a name="select-columns-to-display"></a>選取要顯示的資料行
結果資料表通常包含許多資料行。 您可能會發現某些傳回的資料行依預設並不會顯示出來，或者，您可能想要移除某些顯示出來的資料行。 若要選取要顯示的資料行，請按一下 [資料行] 按鈕：

![Select columns](media/get-started-analytics-portal/select-columns.png)


## <a name="select-a-time-range"></a>選取時間範圍
根據預設，Log Analytics 頁面會套用「過去 24 小時」的時間範圍。 若要使用不同範圍，請透過時間選擇器來選取另一個值，然後按一下 [執行]。 除了預設值外，您也可以使用 [自訂時間範圍] 選項，來選取查詢的絕對範圍。

![時間選擇器](media/get-started-analytics-portal/time-picker.png)

在選取自訂時間範圍時，所選取的值會採用 UTC 格式，這可能會與您當地的時區不同。

如果查詢明確包含「TimeGenerated」的篩選，則時間選擇器標題會顯示 [在查詢中設定]。 系統會停用手動選取以免發生衝突。


## <a name="charts"></a>圖表
除了在資料表中傳回結果外，查詢結果還可以透過視覺化格式來呈現。 請使用下列查詢作為範例：

```Kusto
Event 
| where EventLevelName == "Error" 
| where TimeGenerated > ago(1d) 
| summarize count() by Source 
```

根據預設，結果會顯示在資料表中。 按一下 [圖表] 以透過圖形檢視查看結果：

![長條圖](media/get-started-analytics-portal/bar-chart.png)

結果會顯示在堆疊長條圖中。 按一下 [堆疊直條圖]，然後選取 [圓形圖] 來顯示結果的另一種檢視：

![圓形圖](media/get-started-analytics-portal/pie-chart.png)

您可以從控制列手動變更檢視的不同屬性，例如 x 和 y 軸，或分組及分割喜好設定。

您也可以使用 render 運算子對查詢本身設定慣用檢視。

### <a name="smart-diagnostics"></a>智慧型診斷
在時間圖表上，如果您的資料突然激增或升級，您可能會在線上看到反白顯示的點。 這表示「智慧型診斷」已識別出可將突然變更篩選掉的屬性組合。 按一下點可取得篩選條件的詳細資料，以及查看篩選的版本。 這可能有助於您找出造成變更的原因：

![智慧型診斷](media/get-started-analytics-portal/smart-diagnostics.png)

## <a name="pin-to-dashboard"></a>釘選到儀表板
若要將圖表或資料表釘選至其中一個共用的 Azure 儀表板，請按一下 [釘選] 圖示。

![釘選到儀表板](media/get-started-analytics-portal/pin-dashboard.png)

當您將圖表釘選到儀表板時，圖表會套用某些簡化效果：

- 資料表的資料行和資料列：若要將資料表釘選到儀表板，它必須具有四個以下的資料行。 只會顯示前七個資料列。
- 時間限制：查詢會自動限制為過去 14 天。
- 量化計數限制：如果您顯示的是有許多不連續量化的圖表，所佔比例較少的量化會自動分組到單一的「其他」量化。

## <a name="save-queries"></a>儲存查詢
建立了實用查詢後，您可以儲存起來，也可以與他人共用。 [儲存] 圖示位於頂端列。

您可以儲存整個查詢頁面，也可以將單一查詢儲存為函式。 函式形式的查詢也可以供其他查詢參考。 若要將查詢儲存為函式，您必須提供函式別名，此名稱可用來在其他查詢參考此查詢時，對其進行呼叫。

![儲存函式](media/get-started-analytics-portal/save-function.png)

Log Analytics 查詢一律會儲存至選取的工作區中，並與該工作區的其他使用者共用。

## <a name="load-queries"></a>載入查詢
[查詢總管] 圖示位於右上方區域中。 這會依類別來列出所有已儲存的查詢。 它也可讓您將特定查詢標示為我的最愛，以供日後快速找到。 對已儲存的查詢按兩下，即可將它新增至目前的視窗。

![查詢總管](media/get-started-analytics-portal/query-explorer.png)

## <a name="export-and-share-as-link"></a>以連結形式匯出及共用
Log Analytics 頁面支援數種匯出方法：

- Excel：將結果儲存為 CSV 檔案。
- Power BI：將結果匯出至 Power BI。 如需詳細資訊，請參閱[將 Azure Log Analytics 資料匯入至 Power BI](../log-analytics-powerbi.md)。
- 共用連結：查詢本身可以透過連結形式來共用，可存取相同工作區的其他使用者便可傳送並執行該連結。

## <a name="next-steps"></a>後續步驟

- 深入了解如何[撰寫 Log Analytics 查詢](get-started-queries.md)。
