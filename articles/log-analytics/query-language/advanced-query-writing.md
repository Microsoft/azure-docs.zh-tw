---
title: Azure Log Analytics 中的進階查詢 | Microsoft Docs
description: 本文提供教學課程來說明如何使用 Analytics 入口網站在 Log Analytics 中撰寫查詢。
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
ms.openlocfilehash: 2f9868abd0eb8bf96928aeba6f96c10bcb91c4e2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46958544"
---
# <a name="writing-advanced-queries-in-log-analytics"></a>在 Log Analytics 中撰寫進階查詢

> [!NOTE]
> 您應該先完成[開始使用 Analytics 入口網站](get-started-analytics-portal.md)與[開始使用查詢](get-started-queries.md)，再完成此課程。

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

## <a name="reusing-code-with-let"></a>透過 let 重複使用程式碼
使用 `let` 來指派結果給變數，並稍後在查詢中使用該變數：

```Kusto
// get all events that have level 2 (indicates warning level)
let warning_events=
Event
| where EventLevel == 2;
// count the number of warning events per computer
warning_events
| summarize count() by Computer 
```

您也可以將常數值指派給變數。 這支援一種方法，讓您針對每次執行查詢時都必須變更的欄位設定參數。 視需要修改那些參數。 例如，若要計算給定時段內的可用磁碟空間與可用記憶體 (以百分位數表示)：

```Kusto
let startDate = datetime(2018-08-01T12:55:02);
let endDate = datetime(2018-08-02T13:21:35);
let FreeDiskSpace =
Perf
| where TimeGenerated between (startDate .. endDate)
| where ObjectName=="Logical Disk" and CounterName=="Free Megabytes"
| summarize percentiles(CounterValue, 50, 75, 90, 99);
let FreeMemory =
Perf
| where TimeGenerated between (startDate .. endDate)
| where ObjectName=="Memory" and CounterName=="Available MBytes Memory"
| summarize percentiles(CounterValue, 50, 75, 90, 99);
union FreeDiskSpace, FreeMemory
```

這可讓您在下次執行查詢時輕鬆變更開始與結束時間。

### <a name="local-functions-and-parameters"></a>區域函式與參數
使用 `let` 陳述式來建立可在相同查詢中使用的函式。 例如，定義一個會取得日期時間欄位值 (國際標準時間 (UTC) 格式) 並將它轉換為標準美國格式的函式。 

```Kusto
let utc_to_us_date_format = (t:datetime)
{
  strcat(getmonth(t), "/", dayofmonth(t),"/", getyear(t), " ",
  bin((t-1h)%12h+1h,1s), iff(t%24h<12h, "AM", "PM"))
};
Event 
| where TimeGenerated > ago(1h) 
| extend USTimeGenerated = utc_to_us_date_format(TimeGenerated)
| project TimeGenerated, USTimeGenerated, Source, Computer, EventLevel, EventData 
```

## <a name="functions"></a>Functions
您可以使用函式別名來儲存查詢，以便它可以由其他查詢參考。 例如，下列標準查詢會傳回過去一天內所回報的所有缺少的安全性更新：

```Kusto
Update
| where TimeGenerated > ago(1d) 
| where Classification == "Security Updates" 
| where UpdateState == "Needed"
```

您可以將此查詢另存為函式並為它指定別名，例如 _security_updates_last_day_。 接著，您可以在另一個查詢中使用它來搜尋必要 SQL 相關安全性更新：

```Kusto
security_updates_last_day | where Title contains "SQL"
```

若要將查詢另存為函式，請選取入口網站中的 [儲存] 按鈕，並將 [另存新檔] 變更為 [函式]。 函式別名可以包含字母、數字或底線，而且開頭必須是字母或底線。

> [!NOTE]
> 您可以在 Log Analytics 查詢中儲存函式，但目前不適用於 Application Insights 查詢。


## <a name="print"></a>列印
`print` 將會傳回具有單欄與單列的表格，顯示計算的結果。 這通常用於需要簡單計算的案例。 例如，尋找太平洋標準時間 (PST) 的目前時間並加上美國東部標準時間 (EST) 欄：

```Kusto
print nowPst = now()-8h
| extend nowEst = nowPst+3h
```

## <a name="datatable"></a>Datatable
`datatable` 可讓您定義資料集合。 您可以提供結構描述與一組值，然後以管道方式將該表格傳送到另一個查詢元素。 例如，建立 RAM 使用狀況表並每小時計算其平均值：

```Kusto
datatable (TimeGenerated: datetime, usage_percent: double)
[
  "2018-06-02T15:15:46.3418323Z", 15.5,
  "2018-06-02T15:45:43.1561235Z", 20.2,
  "2018-06-02T16:16:49.2354895Z", 17.3,
  "2018-06-02T16:46:44.9813459Z", 45.7,
  "2018-06-02T17:15:41.7895423Z", 10.9,
  "2018-06-02T17:44:23.9813459Z", 24.7,
  "2018-06-02T18:14:59.7283023Z", 22.3,
  "2018-06-02T18:45:12.1895483Z", 25.4
]
| summarize avg(usage_percent) by bin(TimeGenerated, 1h)
```

建立查閱表時，Datatable 建構也很實用。 若要將表格資料 (例如來自 _SecurityEvent_ 表格的事件識別碼) 對應到任意處列出的事件型別，請使用 `datatable` 建立具有事件型別的查閱表，並將此 Datatable 與 _SecurityEvent_ 資料聯結：

```Kusto
let eventCodes = datatable (EventID: int, EventType:string)
[
    4625, "Account activity",
    4688, "Process action",
    4634, "Account activity",
    4672, "Access",
    4624, "Account activity",
    4799, "Access management",
    4798, "Access management",
    5059, "Key operation",
    4648, "A logon was attempted using explicit credentials",
    4768, "Access management",
    4662, "Other",
    8002, "Process action",
    4904, "Security event management",
    4905, "Security event management",
];
SecurityEvent
| where TimeGenerated > ago(1h) 
| join kind=leftouter (
  eventCodes
) on EventID
| project TimeGenerated, Account, AccountType, Computer, EventType
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