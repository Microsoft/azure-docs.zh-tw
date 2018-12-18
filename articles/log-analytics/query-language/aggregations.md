---
title: Azure Log Analytics 查詢中的彙總| Microsoft Docs
description: 說明 Log Analytics 查詢中提供實用方式來分析資料的彙總函式。
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
ms.openlocfilehash: 764c43a382442096a5d130334e54afdc135ba419
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46966670"
---
# <a name="aggregations-in-log-analytics-queries"></a>Log Analytics 查詢中的彙總

> [!NOTE]
> 您應該先完成[開始使用 Analytics 入口網站](get-started-analytics-portal.md)與[開始使用查詢](get-started-queries.md)，再完成此課程。

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

此文章說明 Log Analytics 查詢中提供實用方式來分析資料的彙總函式。 這些函式都可搭配 `summarize` 運算子使用，以產生具有輸入表格之彙總結果的表格。

## <a name="counts"></a>計數

### <a name="count"></a>count
計算套用任何篩選之後結果集中的列數。 下列範例會傳回過去 30 分鐘內 _Perf_ 表中的總列數。 除非您指定特定欄名，否則結果會以名為 *count_* 的欄傳回：


```Kusto
Perf
| where TimeGenerated > ago(30m) 
| summarize count()
```

```Kusto
Perf
| where TimeGenerated > ago(30m) 
| summarize num_of_records=count() 
```

查看一段時間的趨勢時，時間表視覺效果很實用：

```Kusto
Perf 
| where TimeGenerated > ago(30m) 
| summarize count() by bin(TimeGenerated, 5m)
| render timechart
```

來自此範例的輸出會顯示 5 分鐘間隔的效能記錄計數趨勢線：

![計算趨勢](media/aggregations/count-trend.png)


### <a name="dcount-dcountif"></a>dcount, dcountif
使用 `dcount` 與 `dcountif` 來計算特定欄中的相異值。 下列查詢會評估過去一小時內傳送活動訊號的相異電腦數：

```Kusto
Heartbeat 
| where TimeGenerated > ago(1h) 
| summarize dcount(Computer)
```

若只要計算傳送活動訊號的 Linux 電腦數，請使用 `dcountif`：

```Kusto
Heartbeat 
| where TimeGenerated > ago(1h) 
| summarize dcountif(Computer, OSType=="Linux")
```

### <a name="evaluating-subgroups"></a>評估子群組
若要為您資料中的子群組執行計算或其他彙總，請使用 `by` 關鍵字。 例如，若要計算在每個國家/地區傳送活動訊號的相異 Linux 電腦數目：

```Kusto
Heartbeat 
| where TimeGenerated > ago(1h) 
| summarize distinct_computers=dcountif(Computer, OSType=="Linux") by RemoteIPCountry
```

|RemoteIPCountry  | distinct_computers  |
------------------|---------------------|
|美國    | 19                  |
|加拿大           | 3                   |
|愛爾蘭          | 0                   |
|英國   | 0                   |
|荷蘭      | 2                   |


若要分析更小的資料子群組，請將額外欄名加到 `by` 區段。 例如，您可能想要計算每個國家/地區依 OSType 的相異電腦數：

```Kusto
Heartbeat 
| where TimeGenerated > ago(1h) 
| summarize distinct_computers=dcountif(Computer, OSType=="Linux") by RemoteIPCountry, OSType
```

## <a name="percentiles-and-variance"></a>百分位數和變異數
當評估數值時，常見實務是使用 `summarize avg(expression)` 來計算其平均值。 平均值會受僅描繪少數案例的極端值影響。 為解決該問題，您可以使用較不敏感的函式，例如 `median` 或 `variance`。

### <a name="percentile"></a>百分位數
若要尋找中位數，請使用 `percentile` 函式搭配值來指定百分位數：

```Kusto
Perf
| where TimeGenerated > ago(30m) 
| where CounterName == "% Processor Time" and InstanceName == "_Total" 
| summarize percentiles(CounterValue, 50) by Computer
```

您也可以指定不同的百分位數來取得每個值的彙總結果：

```Kusto
Perf
| where TimeGenerated > ago(30m) 
| where CounterName == "% Processor Time" and InstanceName == "_Total" 
| summarize percentiles(CounterValue, 25, 50, 75, 90) by Computer
```

這可能會顯示某些電腦 CPU 有相同的中位數，但某些電腦 CPU 都持續接近中位數，而其他電腦則回報低很多與高很多的 CPU 值，表示它們遇到峰值。

### <a name="variance"></a>Variance
若要直接評估值的變異數，請使用標準差與變異數方法：

```Kusto
Perf
| where TimeGenerated > ago(30m) 
| where CounterName == "% Processor Time" and InstanceName == "_Total" 
| summarize stdev(CounterValue), variance(CounterValue) by Computer
```

分析 CPU 使用狀況穩定性的一個好方式是結合 stdev 與中位數計算：

```Kusto
Perf
| where TimeGenerated > ago(130m) 
| where CounterName == "% Processor Time" and InstanceName == "_Total" 
| summarize stdev(CounterValue), percentiles(CounterValue, 50) by Computer
```

請參閱其他課程以了解如何使用 Log Analytics 查詢語言：

- [字串作業](string-operations.md)
- [日期和時間作業](datetime-operations.md)
- [進階彙總](advanced-aggregations.md)
- [JSON 與資料結構](json-data-structures.md)
- [進階查詢撰寫](advanced-query-writing.md)
- [聯結](joins.md)
- [圖表](charts.md)
