---
title: 在 Azure Log Analytics 中開始使用查詢 | Microsoft Docs
description: 本文提供教學課程來說明如何在 Log Analytics 中開始撰寫查詢。
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
ms.date: 08/06/2018
ms.author: bwren
ms.component: na
ms.openlocfilehash: b56a75074af239f60b82edbe1d074c6384c4aef1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46982969"
---
# <a name="get-started-with-queries-in-log-analytics"></a>在 Log Analytics 中開始使用查詢


> [!NOTE]
> 請先完成[開始使用 Analytics 入口網站](get-started-analytics-portal.md)再完成本教學課程。

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

在本教學課程中，您會了解如何撰寫 Azure Log Analytics 查詢。 它會告訴您如何：

- 了解查詢的結構
- 排序查詢結果
- 篩選查詢結果
- 指定時間範圍
- 選取要包含在結果中的欄位
- 定義和使用自訂欄位
- 彙總和群組結果


## <a name="writing-a-new-query"></a>撰寫新的查詢
查詢可以透過資料表名稱或 *search* 命令來開始。 請從資料表名稱開始，原因是它會定義清楚的查詢範圍，並改善查詢效能和結果的相關性。

> [!NOTE]
> Azure Log Analytics 查詢語言有區分大小寫。 語言關鍵字通常會以小寫來撰寫。 當查詢中使用到資料表或資料行的名稱時，請務必使用正確的大小寫，如結構描述窗格所示。

### <a name="table-based-queries"></a>以資料表為基礎的查詢
Azure Log Analytics 會將資料組織到資料表中，每個資料表都包含多個資料行。 所有資料表和資料行都會顯示在 Analytics 入口網站中的 [結構描述] 窗格。 請找出您感興趣的資料表，然後看看其中的資料：

```Kusto
SecurityEvent
| take 10
```

上述查詢會傳回「SecurityEvent」資料表中的 10 個結果 (沒有特定順序)。 這是瀏覽資料表並了解其結構和內容的常見方式。 讓我們檢查其建置方式：

* 此查詢會從「SecurityEvent」資料表名稱來開始，這部分會定義查詢的範圍。
* 垂直線 (|) 字元會分隔命令，下列命令輸入中第一個項目的輸出也是如此。 您可以新增任意數目的垂直線元素。
* 垂直線後面是 **take** 命令，會從資料表傳回特定數目的任意記錄。

即使未新增 `| take 10`，我們實際上仍可執行查詢，查詢仍會有效，但它可能會傳回高達 10,000 個結果。

### <a name="search-queries"></a>搜尋查詢
搜尋查詢的結構較鬆散，通常更適合用於尋找其資料行中包含特定值的記錄：

```Kusto
search in (SecurityEvent) "Cryptographic"
| take 10
```

此查詢會搜尋「SecurityEvent」資料表，看看其中是否有記錄包含「Cryptographic」片語。 系統會傳回並顯示這些記錄的其中 10 筆。 如果我們省略了 `in (SecurityEvent)` 部分，只執行 `search "Cryptographic"`，則搜尋會找遍「所有」資料表，因此需要較長時間，且較沒效率。

> [!NOTE]
> 預設的設定為「過去 24 小時」的時間範圍。 若要使用不同範圍，請使用時間選擇器 (位於 [執行] 按鈕旁)，或在查詢中新增明確的時間範圍篩選條件。

## <a name="sort-and-top"></a>Sort 和 top
雖然 **take** 適合用來取得一些記錄，但所選取和顯示的結果並沒有依特定順序來排列。 若要取得已排序的檢視，您可以依所慣用的資料行來**排序**：

```Kusto
SecurityEvent   
| sort by TimeGenerated desc
```

不過，這麼做會傳回太多結果，而且也可能需要一些時間。 上述查詢會依 TimeGenerated 資料行來排序「整個」SecurityEvent 資料表。 Analytics 入口網站接著會將顯示限制為只顯示 10,000 筆記錄。 這當然不是最佳方法。

若要只取得最近 10 筆記錄，最佳方法是使用 **top**，其會在伺服器端排序整個資料表，然後傳回排序在前的記錄：

```Kusto
SecurityEvent
| top 10 by TimeGenerated
```

預設的排序順序是遞減，因此，我們通常會忽略 **desc** 引數。輸出看起來會像這樣：

![前 10 個](media/get-started-queries/top10.png)


## <a name="where-filtering-on-a-condition"></a>Where︰針對條件進行篩選
顧名思義，篩選會根據特定條件來篩選資料。 這種方式最常用來限制查詢結果，以便只顯示相關資訊。

若要對查詢新增篩選，請使用 **where** 運算子，後面加上一或多個條件。 例如，下列查詢只會傳回 Level 等於 8 的 SecurityEvent 記錄：

```Kusto
SecurityEvent
| where Level == 8
```

在撰寫篩選條件時，您可以使用下列運算式：

| 運算是 | 說明 | 範例 |
|:---|:---|:---|
| == | 檢查是否相等<br>(區分大小寫) | `Level == 8` |
| =~ | 檢查是否相等<br>(不區分大小寫) | `EventSourceName =~ "microsoft-windows-security-auditing"` |
| !=、<> | 檢查是否不相等<br>(這兩個運算式同義) | `Level != 4` |
| and、or | 條件之間必須有此項目| `Level == 16 or CommandLine != ""` |

若要依多個條件進行篩選，您可以使用 **and**：

```Kusto
SecurityEvent
| where Level == 8 and EventID == 4672
```

也可以透過垂直線將多個 **where** 元素一個接一個串連在一起：

```Kusto
SecurityEvent
| where Level == 8 
| where EventID == 4672
```
    
> [!NOTE]
> 值可以有不同類型，因此，您可能需要轉換值才能對正確類型進行比較。 例如，SecurityEvent 的 *Level* 資料行是字串類型，因此，您必須先將其轉換為數值類型 (例如，int 或 long)，才能對其使用數值運算子：`SecurityEvent | where toint(Level) >= 10`

## <a name="specify-a-time-range"></a>指定時間範圍

### <a name="time-picker"></a>時間選擇器
時間選擇器位於左上角，會指出我們只要查詢過去 24 小時內的記錄。 這是適用於所有查詢的預設時間範圍。 若只要取得過去 1 小時的記錄，請選取 [過去 1 小時]，然後再次執行查詢。

![時間選擇器](media/get-started-queries/timepicker.png)


### <a name="time-filter-in-query"></a>查詢中的時間篩選
您也可以對查詢新增時間篩選，藉以定義您自己的時間範圍。 時間篩選最好直接放在資料表名稱之後： 

```Kusto
SecurityEvent
| where TimeGenerated > ago(30m) 
| where toint(Level) >= 10
```

在上述時間篩選中，`ago(30m)` 表示「30 分鐘前」，因此，這個查詢只會傳回過去 30 分鐘的記錄。 其他時間單位包括天 (2d)、分鐘 (25m) 和秒 (10s)。


## <a name="project-and-extend-select-and-compute-columns"></a>Project 和 Extend：選取和計算資料行
使用 **project** 可選取要包含在結果中的特定資料行：

```Kusto
SecurityEvent 
| top 10 by TimeGenerated 
| project TimeGenerated, Computer, Activity
```

上述範例會產生以下輸出：

![Log Analytics 專案結果](media/get-started-queries/project.png)

您也可以使用 **project** 將資料行重新命名並定義新的資料行。 下列範例會使用 project 來執行下列作業：

* 僅選取 Computer 和 TimeGenerated 原始資料行。
* 將 Activity 資料行重新命名為 EventDetails。
* 建立名為 EventCode 的新資料行。 **substring()** 函式可用來僅取得 [活動] 欄位中的前四個字元。


```Kusto
SecurityEvent
| top 10 by TimeGenerated 
| project Computer, TimeGenerated, EventDetails=Activity, EventCode=substring(Activity, 0, 4)
```

**extend** 會在結果集內保留所有原始資料行，並定義其他資料行。 下列查詢會使用 **extend** 來新增 localtime 資料行，其中包含當地語系化的 TimeGenerated 值。

```Kusto
SecurityEvent
| top 10 by TimeGenerated
| extend localtime = TimeGenerated-8h
```

## <a name="summarize-aggregate-groups-of-rows"></a>Summarize：彙總資料列群組
使用 **summarize** 可識別記錄群組 (根據一或多個資料行)，並對其套用彙總。 **summarize** 最常見的用法是 count，其會傳回每個群組中的結果數目。

下列查詢會檢閱過去 1 小時的所有 Perf 記錄、依 ObjectName 將這些記錄分組，並計算每個群組中的記錄數目： 
```Kusto
Perf
| where TimeGenerated > ago(1h)
| summarize count() by ObjectName
```

依多個維度來定義群組有時很合理。 這些值的每個唯一組合可定義出不同的群組：

```Kusto
Perf
| where TimeGenerated > ago(1h)
| summarize count() by ObjectName, CounterName
```

另一個常見用途是針對每個群組進行數學或統計計算。 例如，下列運算式會計算每部電腦的 CounterValue 平均值：

```Kusto
Perf
| where TimeGenerated > ago(1h)
| summarize avg(CounterValue) by Computer
```

不幸的是，此查詢的結果並無意義，原因是我們混搭使用不同的效能計數器。 若要讓結果更有意義，我們應該針對 CounterName 和 Computer 的每個組合分別計算平均值：

```Kusto
Perf
| where TimeGenerated > ago(1h)
| summarize avg(CounterValue) by Computer, CounterName
```

### <a name="summarize-by-a-time-column"></a>依時間資料行彙總
群組結果也可以根據時間資料行或其他連續值來進行。 不過，只彙總 `by TimeGenerated` 會針對時間範圍內的每一毫秒建立群組，原因是這些是唯一值。 

若要根據連續值建立群組，最好是使用 **bin** 將範圍分成可管理的單位。 下列查詢會分析 Perf 記錄，這些記錄會測量特定電腦上的可用記憶體 (可用的 MB 數)。 它會計算過去 2 天內每個期間 (假設為 1 小時) 的平均值：

```Kusto
Perf 
| where TimeGenerated > ago(2d)
| where Computer == "ContosoAzADDS2" 
| where CounterName == "Available MBytes" 
| summarize count() by bin(TimeGenerated, 1h)
```

若要讓輸出更加清楚，您可以選取將它顯示為時間圖表，以顯示一段時間內的可用記憶體：

![一段時間內的 Log Analytics 記憶體](media/get-started-queries/chart.png)



## <a name="next-steps"></a>後續步驟

- 了解如何[撰寫搜尋查詢](search-queries.md)