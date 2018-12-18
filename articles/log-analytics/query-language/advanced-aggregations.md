---
title: Azure Log Analytics 查詢中的進階彙總| Microsoft Docs
description: 說明 Log Analytics 查詢中可使用的一些更進階的彙總選項。
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
ms.date: 08/16/2018
ms.author: bwren
ms.component: na
ms.openlocfilehash: 288af0eae50634f44d6af8c787b56112bb3119ff
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998588"
---
# <a name="advanced-aggregations-in-log-analytics-queries"></a>Log Analytics 查詢中的進階彙總

> [!NOTE]
> 您應該先完成[Log Analytics 查詢中的彙總](./aggregations.md)，再完成此課程。

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

此文章說明 Log Analytics 查詢中可使用的一些更進階的彙總選項。

## <a name="generating-lists-and-sets"></a>產生清單與集合
您可以使用 `makelist` 依特定欄中的值順序瀏覽樞紐資料。 例如，您可能想要探索您的機器上發生的最長見訂購事件。 基本上，您能依每部機器上的事件識別碼順序來瀏覽樞紐資料。 

```Kusto
Event
| where TimeGenerated > ago(12h)
| order by TimeGenerated desc
| summarize makelist(EventID) by Computer
```
|電腦|list_EventID|
|---|---|
| computer1 | [704,701,1501,1500,1085,704,704,701] |
| computer2 | [326,105,302,301,300,102] |
| ... | ... |

`makelist` 會產生資料傳遞到其中的順序清單。 若要從最舊到最新為事件排序，請在順序陳述式中使用 `asc`，而非 `desc`. 

建立只包含相異值的清單也很實用。 這稱為「集合」，而且可以使用 `makeset` 來產生：

```Kusto
Event
| where TimeGenerated > ago(12h)
| order by TimeGenerated desc
| summarize makeset(EventID) by Computer
```
|電腦|list_EventID|
|---|---|
| computer1 | [704,701,1501,1500,1085] |
| computer2 | [326,105,302,301,300,102] |
| ... | ... |

就像 `makelist` 一樣，`makeset` 也適用於已排序的資料，而且金會根據傳遞到其中的列順序來產生陣列。

## <a name="expanding-lists"></a>展開清單
`makelist` 或 `makeset` 的反向作業是 `mvexpand`，它會展開值清單以分隔列。 它可以跨任何數目的動態欄 (包括 JSON 與陣列) 展開。 例如，您可以檢查「活動訊號」表格以了解過去一小時內從已傳送活動訊號之電腦傳送資料的解決方案：

```Kusto
Heartbeat
| where TimeGenerated > ago(1h)
| project Computer, Solutions
```

| 電腦 | 解決方案 | 
|--------------|----------------------|
| computer1 | 「安全性」、「更新」、「變更追蹤」 |
| computer2 | 「安全性」、「更新」 |
| computer3 | 「反惡意程式碼」、「變更追蹤」 |
| ... | ... | ... |

使用 `mvexpand` 以在個別列中顯示每個值，而不是顯示逗號分隔清單：

```Kusto
Heartbeat
| where TimeGenerated > ago(1h)
| project Computer, split(Solutions, ",")
| mvexpand Solutions
```

| 電腦 | 解決方案 | 
|--------------|----------------------|
| computer1 | 「安全性」 |
| computer1 | 「更新」 |
| computer1 | 「變更追蹤」 |
| computer2 | 「安全性」 |
| computer2 | 「更新」 |
| computer3 | 「反惡意程式碼」 |
| computer3 | 「變更追蹤」 |
| ... | ... | ... |


接著，您可以再次使用 `makelist` 以將項目組成群組，而且這次您會看到每個解決方案的電腦清單：

```Kusto
Heartbeat
| where TimeGenerated > ago(1h)
| project Computer, split(Solutions, ",")
| mvexpand Solutions
| summarize makelist(Computer) by tostring(Solutions) 
```
|解決方案 | list_Computer |
|--------------|----------------------|
| 「安全性」 | ["computer1", "computer2"] |
| 「更新」 | ["computer1", "computer2"] |
| 「變更追蹤」 | ["computer1", "computer3"] |
| 「反惡意程式碼」 | ["computer3"] |
| ... | ... |

## <a name="handling-missing-bins"></a>處理遺失的間隔
填寫遺失間隔的預設值是 `mvexpand` 的其中一個實用應用。例如，假設您正在透過探索特定機器的活動訊號以尋找其運作時間。 您可能也想要查看活動訊號來源，這位於「類別」欄。 一般而言，我們會使用簡單的 summarize 陳述式，如下所示：

```Kusto
Heartbeat
| where TimeGenerated > ago(12h)
| summarize count() by Category, bin(TimeGenerated, 1h)
```
| 類別 | TimeGenerated | count_ |
|--------------|----------------------|--------|
| 直接代理程式 | 2017-06-06T17:00:00Z | 15 |
| 直接代理程式 | 2017-06-06T18:00:00Z | 60 |
| 直接代理程式 | 2017-06-06T20:00:00Z | 55 |
| 直接代理程式 | 2017-06-06T21:00:00Z | 57 |
| 直接代理程式 | 2017-06-06T22:00:00Z | 60 |
| ... | ... | ... |

在這些結果中，雖然與 "2017-06-06T19:00:00Z" 關聯的貯體遺失 (因為那一小時內沒有任何活動訊號資料)。 使用 `make-series` 函式來指派預設值給空白貯體。 這將會為每個類別產生一列，每一列都有四個額外的陣列欄，一個適用於值，而一個適用於比較時間貯體：

```Kusto
Heartbeat
| make-series count() default=0 on TimeGenerated in range(ago(1d), now(), 1h) by Category 
```

| 類別 | count_ | TimeGenerated |
|---|---|---|
| 直接代理程式 | [15,60,0,55,60,57,60,...] | ["2017-06-06T17:00:00.0000000Z","2017-06-06T18:00:00.0000000Z","2017-06-06T19:00:00.0000000Z","2017-06-06T20:00:00.0000000Z","2017-06-06T21:00:00.0000000Z",...] |
| ... | ... | ... |

*count_* 陣列的第三個元素如預期是 0，而且 _TimeGenerated_ 陣列中有符合的時間戳記項目 "2017-06-06T19:00:00.0000000Z"。 但此陣列格式難以閱讀。 使用 `mvexpand` 來展開陣列，並產生與 `summarize`所產生之格式輸出相同的格式輸出：

```Kusto
Heartbeat
| make-series count() default=0 on TimeGenerated in range(ago(1d), now(), 1h) by Category 
| mvexpand TimeGenerated, count_
| project Category, TimeGenerated, count_
```
| 類別 | TimeGenerated | count_ |
|--------------|----------------------|--------|
| 直接代理程式 | 2017-06-06T17:00:00Z | 15 |
| 直接代理程式 | 2017-06-06T18:00:00Z | 60 |
| 直接代理程式 | 2017-06-06T19:00:00Z | 0 |
| 直接代理程式 | 2017-06-06T20:00:00Z | 55 |
| 直接代理程式 | 2017-06-06T21:00:00Z | 57 |
| 直接代理程式 | 2017-06-06T22:00:00Z | 60 |
| ... | ... | ... |



## <a name="narrowing-results-to-a-set-of-elements-let-makeset-toscalar-in"></a>將結果縮小為元素集合：`let`、`makeset`、`toscalar`、`in`
常見案例是根據一組條件選取某些特定實體的名稱，然後將不同的資料集篩選為該實體集合。 例如，您可以尋找已知缺少更新的電腦，並找出這些電腦的 IP：


```Kusto
let ComputersNeedingUpdate = toscalar(
    Update
    | summarize makeset(Computer)
    | project set_Computer
);
WindowsFirewall
| where Computer in (ComputersNeedingUpdate)
```

## <a name="next-steps"></a>後續步驟

請參閱其他課程以了解如何使用 Log Analytics 查詢語言：

- [字串作業](string-operations.md)
- [日期和時間作業](datetime-operations.md)
- [彙總函式](aggregations.md)
- [進階彙總](advanced-aggregations.md)
- [JSON 與資料結構](json-data-structures.md)
- [聯結](joins.md)
- [圖表](charts.md)