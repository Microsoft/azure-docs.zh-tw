---
title: 在 Azure Log Analytics 查詢中處理日期時間值 | Microsoft Docs
description: 說明如何在 Log Analytics 查詢中處理日期和時間資料。
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
ms.openlocfilehash: 9b0c58fdbfb0d55b3b8998f4edfc1222b9a3d4aa
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46988592"
---
# <a name="working-with-date-time-values-in-log-analytics-queries"></a>在 Log Analytics 查詢中處理日期時間值

> [!NOTE]
> 您應該先完成[開始使用 Analytics 入口網站](get-started-analytics-portal.md)與[開始使用查詢](get-started-queries.md)，再完成此課程。

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

此文章說明如何在 Log Analytics 查詢中處理日期和時間資料。


## <a name="date-time-basics"></a>日期時間基礎
Log Analytics 查詢語言有兩個與日期和時間關聯的主要資料類型：「日期時間」與「時間範圍」。 所有日期都以國際標準時間 (UTC) 表示。 雖然我們支援多種日期時間格式，但建議使用 ISO8601 格式。 

時間範圍是以十進位數加上時間單位來表示：

|縮寫   | 時間單位    |
|:---|:---|
|d           | day          |
|h           | hour         |
|m           | 分鐘       |
|s           | 秒       |
|ms          | 毫秒  |
|microsecond | 微秒  |
|tick        | 奈秒   |

您可以透過使用 `todatetime` 運算子轉換字串來建立。 例如，若要檢閱在特定時間範圍內傳送的 VM 活動訊號，您可以使用 [between 運算子](https://docs.loganalytics.io/docs/Language-Reference/Scalar-operators/between-operator)以方便地指定時間範圍。

```Kusto
Heartbeat
| where TimeGenerated between(datetime("2018-06-30 22:46:42") .. datetime("2018-07-01 00:57:27"))
```

另一個常見案例是將日期時間與現在做比較。 例如，若要查看過去兩分鐘內的所有活動訊號，您可以使用 `now` 運算子結合代表兩分鐘的時間範圍：

```Kusto
Heartbeat
| where TimeGenerated > now() - 2m
```

也提供此函式的捷徑：
```Kusto
Heartbeat
| where TimeGenerated > now(-2m)
```

然而，最快且最可讀的方法是使用 `ago` 運算子：
```Kusto
Heartbeat
| where TimeGenerated > ago(2m)
```

假設您不知道開始與結束時間，但知道開始時間與時間長度。 您可以將查詢重寫為下面這樣：

```Kusto
let startDatetime = todatetime("2018-06-30 20:12:42.9");
let duration = totimespan(25m);
Heartbeat
| where TimeGenerated between(startDatetime .. (startDatetime+duration) )
| extend timeFromStart = TimeGenerated - startDatetime
```

## <a name="converting-time-units"></a>轉換時間單位
使用預設時間單位以外的時間單位來表示日期時間或時間範圍非常實用。 例如，假設您正在檢閱過去 30 分鐘內的錯誤事件，而且需要顯示事件在多久之前發生的計算結果欄：

```Kusto
Event
| where TimeGenerated > ago(30m)
| where EventLevelName == "Error"
| extend timeAgo = now() - TimeGenerated 
```

您可以看到 _timeAgo_ 欄存放 "00:09:31.5118992" 之類的值，表示其格式為 hh:mm:ss.fffffff。 若您不想要將這些值的格式設定為從開始時間算起的分鐘數 _numver_，只要將該值除以「1 分鐘」即可：

```Kusto
Event
| where TimeGenerated > ago(30m)
| where EventLevelName == "Error"
| extend timeAgo = now() - TimeGenerated
| extend timeAgoMinutes = timeAgo/1m 
```


## <a name="aggregations-and-bucketing-by-time-intervals"></a>依時間間隔的彙總與貯體處理
另一個非常常見的案例是以特定時間粒紋取得一段特定間期間統計資料的需求。 針對這個目的，可以使用 `bin` 運算子做為摘要子句的一部分。

使用下列查詢來取得過去半小時內每 5 分鐘發生的事件數目：

```Kusto
Event
| where TimeGenerated > ago(30m)
| summarize events_count=count() by bin(TimeGenerated, 5m) 
```

這會產生下列表格：  
|TimeGenerated(UTC)|events_count|
|--|--|
|2018-08-01T09:30:00.000|54|
|2018-08-01T09:35:00.000|41|
|2018-08-01T09:40:00.000|42|
|2018-08-01T09:45:00.000|41|
|2018-08-01T09:50:00.000|41|
|2018-08-01T09:55:00.000|16|

建立結果貯體的另一個方式是使用函式，例如 `startofday`：

```Kusto
Event
| where TimeGenerated > ago(4d)
| summarize events_count=count() by startofday(TimeGenerated) 
```

這會產生下列結果：

|timestamp|count_|
|--|--|
|2018-07-28T00:00:00.000|7,136|
|2018-07-29T00:00:00.000|12,315|
|2018-07-30T00:00:00.000|16,847|
|2018-07-31T00:00:00.000|12,616|
|2018-08-01T00:00:00.000|5,416  |


## <a name="time-zones"></a>時區
因為所有日期時間值都是國際標準時間 (UTC) 表示，將這些時間轉換為當地時區時非常實用。 例如，使用此計算將國際標準時間 (UTC) 轉換為太平洋標準時間 (PST) 時間：

```Kusto
Event
| extend localTimestamp = TimeGenerated - 8h
```

## <a name="related-functions"></a>相關函式

| 類別 | 函式 |
|:---|:---|
| 轉換資料類型 | [todatetime](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/todatetime())  [totimespan](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/totimespan())  |
| 將值四捨五入為間隔大小 | [bin](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/bin()) |
| 取得特定日期或時間 | [ago](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/ago()) [now](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/now())   |
| 取得值的某部分 | [datetime_part](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/datetime_part()) [getmonth](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/getmonth()) [monthofyear](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/monthofyear()) [getyear](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/getyear()) [dayofmonth](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/dayofmonth()) [dayofweek](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/dayofweek()) [dayofyear](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/dayofyear()) [weekofyear](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/weekofyear()) |
| 取得與值相對的日期  | [endofday](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/endofday()) [endofweek](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/endofweek()) [endofmonth](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/endofmonth()) [endofyear](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/endofyear()) [startofday](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/startofday()) [startofweek](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/startofweek()) [startofmonth](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/startofmonth()) [startofyear](https://docs.loganalytics.io/docs/Language-Reference/Scalar-functions/startofyear()) |

## <a name="next-steps"></a>後續步驟
請參閱其他課程以了解如何使用 Log Analytics 查詢語言：

- [字串作業](string-operations.md)
- [彙總函式](aggregations.md)
- [進階彙總](advanced-aggregations.md)
- [JSON 與資料結構](json-data-structures.md)
- [進階查詢撰寫](advanced-query-writing.md)
- [聯結](joins.md)
- [圖表](charts.md)