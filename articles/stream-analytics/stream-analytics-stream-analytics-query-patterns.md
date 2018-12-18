---
title: Azure 串流分析中常見的查詢模式
description: 本文說明一些常見的查詢模式和設計，在 Azure 串流分析作業中很實用。
services: stream-analytics
author: jseb225
manager: kfile
ms.author: jeanb
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/08/2017
ms.openlocfilehash: 7f171fa1eb8c91b55119d0308b57fe3d3e70261b
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39578886"
---
# <a name="query-examples-for-common-stream-analytics-usage-patterns"></a>一般串流分析使用模式的查詢範例

## <a name="introduction"></a>簡介
Azure 串流分析的查詢會以類似 SQL 的查詢語言表達。 語言建構記載在[串流分析查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)指南中。 

查詢設計能傳達簡單的傳遞邏輯，將事件資料從某個輸入資料流移動到輸出資料存放區。 也可以執行豐富的模式比對和時態分析，計算不同時間範圍內的彙總值 (如 TollApp 範例所示)。 您可以聯結多個輸入的資料來合併串流事件，以及查閱靜態參考資料，從而擴充事件值。 此外，您也可以將資料寫入多個輸出。

本文章根據真實世界案例概述幾個常見查詢模式的解決方案。 這是進行中的工作，會繼續不間斷使用新模式進行更新。

## <a name="work-with-complex-data-types-in-json-and-avro"></a>在 JSON 和 AVRO 中使用複雜資料類型 
Azure 串流分析可處理資料格式為 CSV、JSON 和 Avro 的事件。
JSON 和 Avro 可能包含巢狀物件 (記錄) 或陣列等複雜類型。 若要使用這些複雜的資料類型，請參閱[剖析 JSON 和 AVRO 資料](stream-analytics-parsing-json.md)一文。


## <a name="query-example-convert-data-types"></a>查詢範例：轉換資料類型
**描述**：在輸入資料流上定義屬性類型。
例如，汽車重量即將以字串形式出現在輸入資料流，並且需要轉換為 **INT** 以執行 **SUM** 來將它加總。

**輸入**：

| 請確定 | 時間 | Weight |
| --- | --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |"1000" |
| Honda |2015-01-01T00:00:02.0000000Z |"2000" |

**輸出**：

| 請確定 | Weight |
| --- | --- |
| Honda |3000 |

**解決方案**：

    SELECT
        Make,
        SUM(CAST(Weight AS BIGINT)) AS Weight
    FROM
        Input TIMESTAMP BY Time
    GROUP BY
        Make,
        TumblingWindow(second, 10)

**說明**：在 [Weight]\(加權\) 欄位中使用 **CAST** 陳述式來指定其資料類型。 請參閱[資料類型 (Azure 串流分析)](https://msdn.microsoft.com/library/azure/dn835065.aspx) 中的支援資料類型清單。

## <a name="query-example-use-likenot-like-to-do-pattern-matching"></a>查詢範例：使用 Like/Not like 進行模式比對
**描述**：檢查事件的欄位值是否符合特定模式。
例如，檢查結果是否會傳回開頭為 A 且結尾為 9 的車牌。

**輸入**：

| 請確定 | LicensePlate | 時間 |
| --- | --- | --- |
| Honda |ABC-123 |2015-01-01T00:00:01.0000000Z |
| Toyota |AAA-999 |2015-01-01T00:00:02.0000000Z |
| Nissan |ABC-369 |2015-01-01T00:00:03.0000000Z |

**輸出**：

| 請確定 | LicensePlate | 時間 |
| --- | --- | --- |
| Toyota |AAA-999 |2015-01-01T00:00:02.0000000Z |
| Nissan |ABC-369 |2015-01-01T00:00:03.0000000Z |

**解決方案**：

    SELECT
        *
    FROM
        Input TIMESTAMP BY Time
    WHERE
        LicensePlate LIKE 'A%9'

**說明**：使用 **LIKE** 陳述式檢查 [LicensePlate] 欄位值。 開頭應該為 A，接著是長度為零或更多字元的字串，並以 9 結尾。 

## <a name="query-example-specify-logic-for-different-casesvalues-case-statements"></a>查詢範例：為不同的案例/值指定邏輯 (CASE 陳述式)
**描述**：根據特定準則，提供不同的欄位計算方式。
例如，針對通過的相同廠牌汽車中有多少部符合 1 的特殊情況提供字串描述。

**輸入**：

| 請確定 | 時間 |
| --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |
| Toyota |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:03.0000000Z |

**輸出**：

| CarsPassed | 時間 |
| --- | --- | --- |
| 1 Honda |2015-01-01T00:00:10.0000000Z |
| 2 Toyotas |2015-01-01T00:00:10.0000000Z |

**解決方案**：

    SELECT
        CASE
            WHEN COUNT(*) = 1 THEN CONCAT('1 ', Make)
            ELSE CONCAT(CAST(COUNT(*) AS NVARCHAR(MAX)), ' ', Make, 's')
        END AS CarsPassed,
        System.TimeStamp AS Time
    FROM
        Input TIMESTAMP BY Time
    GROUP BY
        Make,
        TumblingWindow(second, 10)

**說明**：**CASE** 運算式會比較運算式和一組簡單運算式來決定結果。 在此範例中，計數為 1 的車輛傳回的字串描述與計數並非為 1 的車輛所傳回者不同。 

## <a name="query-example-send-data-to-multiple-outputs"></a>查詢範例：將資料傳送至多個輸出
**描述**：將資料從單一工作傳送到多個輸出目標。
例如，分析臨界值警示的資料，並將所有事件封存到 Blob 儲存體。

**輸入**：

| 請確定 | 時間 |
| --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |
| Honda |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:01.0000000Z |
| Toyota |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:03.0000000Z |

**Output1**：

| 請確定 | 時間 |
| --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |
| Honda |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:01.0000000Z |
| Toyota |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:03.0000000Z |

**Output2**：

| 請確定 | 時間 | Count |
| --- | --- | --- |
| Toyota |2015-01-01T00:00:10.0000000Z |3 |

**解決方案**：

    SELECT
        *
    INTO
        ArchiveOutput
    FROM
        Input TIMESTAMP BY Time

    SELECT
        Make,
        System.TimeStamp AS Time,
        COUNT(*) AS [Count]
    INTO
        AlertOutput
    FROM
        Input TIMESTAMP BY Time
    GROUP BY
        Make,
        TumblingWindow(second, 10)
    HAVING
        [Count] >= 3

**說明**：**INTO** 子句會告訴串流分析要從此陳述式將資料寫入哪個輸出。
第一個查詢將接收到的資料傳遞至名稱為 **ArchiveOutput** 的輸出。
第二個查詢會執行一些簡單的彙總和篩選，並將結果傳送至下游的警示系統。

請注意，您也可以在多個輸出陳述式中重複使用通用資料表運算式 (CTE) 的結果 (例如 **WITH** 陳述式)。 此選項多了一項優點，就是對輸入來源開放的讀取器較少。
例如︰ 

    WITH AllRedCars AS (
        SELECT
            *
        FROM
            Input TIMESTAMP BY Time
        WHERE
            Color = 'red'
    )
    SELECT * INTO HondaOutput FROM AllRedCars WHERE Make = 'Honda'
    SELECT * INTO ToyotaOutput FROM AllRedCars WHERE Make = 'Toyota'

## <a name="query-example-count-unique-values"></a>查詢範例：計算唯一值
**描述**：計算某個時間範圍內在串流中所出現唯一欄位值的數目。
例如，在 2 秒鐘時間範圍內有多少部某一獨特廠牌的汽車通過收費亭？

**輸入**：

| 請確定 | 時間 |
| --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |
| Honda |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:01.0000000Z |
| Toyota |2015-01-01T00:00:02.0000000Z |
| Toyota |2015-01-01T00:00:03.0000000Z |

**輸出：**

| CountMake | 時間 |
| --- | --- |
| 2 |2015-01-01T00:00:02.000Z |
| 1 |2015-01-01T00:00:04.000Z |

**解決方案：**

````
SELECT
     COUNT(DISTINCT Make) AS CountMake,
     System.TIMESTAMP AS TIME
FROM Input TIMESTAMP BY TIME
GROUP BY 
     TumblingWindow(second, 2)
````


**說明：**
**COUNT(DISTINCT Make)** 會傳回一個時間範圍內 **Make** 資料行的相異值數目。

## <a name="query-example-determine-if-a-value-has-changed"></a>查詢範例：判斷值是否已變更
**描述**：查看前一個值來判斷該值是否與目前的值不同。
例如收費道路上前一輛汽車的品牌，是否與目前汽車的品牌相同？

**輸入**：

| 請確定 | 時間 |
| --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |
| Toyota |2015-01-01T00:00:02.0000000Z |

**輸出**：

| 請確定 | 時間 |
| --- | --- |
| Toyota |2015-01-01T00:00:02.0000000Z |

**解決方案**：

    SELECT
        Make,
        Time
    FROM
        Input TIMESTAMP BY Time
    WHERE
        LAG(Make, 1) OVER (LIMIT DURATION(minute, 1)) <> Make

**說明**：使用 **LAG** 查看前一個事件的輸入資料流，並取得 **Make** 值。 然後將其和目前事件中的 **Make** 值比較，並且在兩者不同時輸出事件。

## <a name="query-example-find-the-first-event-in-a-window"></a>查詢範例：尋找時間範圍內的第一個事件
**描述**：是否要每隔 10 分鐘尋找第一輛車。

**輸入**：

| LicensePlate | 請確定 | 時間 |
| --- | --- | --- |
| DXE 5291 |Honda |2015-07-27T00:00:05.0000000Z |
| YZK 5704 |Ford |2015-07-27T00:02:17.0000000Z |
| RMV 8282 |Honda |2015-07-27T00:05:01.0000000Z |
| YHN 6970 |Toyota |2015-07-27T00:06:00.0000000Z |
| VFE 1616 |Toyota |2015-07-27T00:09:31.0000000Z |
| QYF 9358 |Honda |2015-07-27T00:12:02.0000000Z |
| MDR 6128 |BMW |2015-07-27T00:13:45.0000000Z |

**輸出**：

| LicensePlate | 請確定 | 時間 |
| --- | --- | --- |
| DXE 5291 |Honda |2015-07-27T00:00:05.0000000Z |
| QYF 9358 |Honda |2015-07-27T00:12:02.0000000Z |

**解決方案**：

    SELECT 
        LicensePlate,
        Make,
        Time
    FROM 
        Input TIMESTAMP BY Time
    WHERE 
        IsFirst(minute, 10) = 1

現在讓我們來變更問題，每隔 10 分鐘尋找特定廠牌的第一輛車。

| LicensePlate | 請確定 | 時間 |
| --- | --- | --- |
| DXE 5291 |Honda |2015-07-27T00:00:05.0000000Z |
| YZK 5704 |Ford |2015-07-27T00:02:17.0000000Z |
| YHN 6970 |Toyota |2015-07-27T00:06:00.0000000Z |
| QYF 9358 |Honda |2015-07-27T00:12:02.0000000Z |
| MDR 6128 |BMW |2015-07-27T00:13:45.0000000Z |

**解決方案**：

    SELECT 
        LicensePlate,
        Make,
        Time
    FROM 
        Input TIMESTAMP BY Time
    WHERE 
        IsFirst(minute, 10) OVER (PARTITION BY Make) = 1

## <a name="query-example-find-the-last-event-in-a-window"></a>查詢範例：尋找時間範圍內的上一個事件
**描述**：每隔 10 分鐘尋找上一輛車。

**輸入**：

| LicensePlate | 請確定 | 時間 |
| --- | --- | --- |
| DXE 5291 |Honda |2015-07-27T00:00:05.0000000Z |
| YZK 5704 |Ford |2015-07-27T00:02:17.0000000Z |
| RMV 8282 |Honda |2015-07-27T00:05:01.0000000Z |
| YHN 6970 |Toyota |2015-07-27T00:06:00.0000000Z |
| VFE 1616 |Toyota |2015-07-27T00:09:31.0000000Z |
| QYF 9358 |Honda |2015-07-27T00:12:02.0000000Z |
| MDR 6128 |BMW |2015-07-27T00:13:45.0000000Z |

**輸出**：

| LicensePlate | 請確定 | 時間 |
| --- | --- | --- |
| VFE 1616 |Toyota |2015-07-27T00:09:31.0000000Z |
| MDR 6128 |BMW |2015-07-27T00:13:45.0000000Z |

**解決方案**：

    WITH LastInWindow AS
    (
        SELECT 
            MAX(Time) AS LastEventTime
        FROM 
            Input TIMESTAMP BY Time
        GROUP BY 
            TumblingWindow(minute, 10)
    )
    SELECT 
        Input.LicensePlate,
        Input.Make,
        Input.Time
    FROM
        Input TIMESTAMP BY Time 
        INNER JOIN LastInWindow
        ON DATEDIFF(minute, Input, LastInWindow) BETWEEN 0 AND 10
        AND Input.Time = LastInWindow.LastEventTime

**說明**：查詢中有兩個步驟。 第一個步驟會尋找 10 分鐘時間範圍內最新的時間戳記。 第二個步驟會將第一個查詢的結果與原始串流聯結在一起，在每個時間範圍內尋找符合最後一個時間戳記的事件。 

## <a name="query-example-detect-the-absence-of-events"></a>查詢範例：偵測到事件不存在
**描述**：檢查串流中是否沒有和特定準則相符的值。
例如，在最後的 90 秒內連續有 2 部相同廠牌的車輛進入收費道路？

**輸入**：

| 請確定 | LicensePlate | 時間 |
| --- | --- | --- |
| Honda |ABC-123 |2015-01-01T00:00:01.0000000Z |
| Honda |AAA-999 |2015-01-01T00:00:02.0000000Z |
| Toyota |DEF-987 |2015-01-01T00:00:03.0000000Z |
| Honda |GHI-345 |2015-01-01T00:00:04.0000000Z |

**輸出**：

| 請確定 | 時間 | CurrentCarLicensePlate | FirstCarLicensePlate | FirstCarTime |
| --- | --- | --- | --- | --- |
| Honda |2015-01-01T00:00:02.0000000Z |AAA-999 |ABC-123 |2015-01-01T00:00:01.0000000Z |

**解決方案**：

    SELECT
        Make,
        Time,
        LicensePlate AS CurrentCarLicensePlate,
        LAG(LicensePlate, 1) OVER (LIMIT DURATION(second, 90)) AS FirstCarLicensePlate,
        LAG(Time, 1) OVER (LIMIT DURATION(second, 90)) AS FirstCarTime
    FROM
        Input TIMESTAMP BY Time
    WHERE
        LAG(Make, 1) OVER (LIMIT DURATION(second, 90)) = Make

**說明**：使用 **LAG** 查看前一個事件的輸入資料流，並取得 **Make** 值。 將該值和目前事件中的 **MAKE** 值比較，如果相同則輸出該事件。 您也可以使用 **LAG** 取得上一輛車的相關資料。

## <a name="query-example-detect-the-duration-between-events"></a>查詢範例：偵測事件與事件之間的持續時間
**描述**：尋找指定事件的持續時間。 例如，指定某一網頁點選流，以判斷在某一功能上花費的時間。

**輸入**：  

| 使用者 | 功能 | Event | 時間 |
| --- | --- | --- | --- |
| user@location.com |RightMenu |Start |2015-01-01T00:00:01.0000000Z |
| user@location.com |RightMenu |End |2015-01-01T00:00:08.0000000Z |

**輸出**：  

| 使用者 | 功能 | Duration |
| --- | --- | --- |
| user@location.com |RightMenu |7 |

**解決方案**：

````
    SELECT
        [user], feature, DATEDIFF(second, LAST(Time) OVER (PARTITION BY [user], feature LIMIT DURATION(hour, 1) WHEN Event = 'start'), Time) as duration
    FROM input TIMESTAMP BY Time
    WHERE
        Event = 'end'
````

**說明**：當事件類型為 **Start** 時，使用 **LAST** 函式取出最後一個 **TIME** 值。 **LAST** 函式使用 **PARTITION BY [user]** 來表達會為個別使用者分別計算結果。 查詢的 **Start** 與 **Stop** 事件之間時間差的上限為 1 小時，但該值可以在必要時變更 **(LIMIT DURATION(hour, 1)**。

## <a name="query-example-detect-the-duration-of-a-condition"></a>查詢範例：偵測某個情況的持續時間
**描述**：找出某個情況的持續時間。
例如，假設有個錯誤導致所有車輛的重量不正確 (超過 20,000 磅)，而且必須計算該錯誤的持續時間。

**輸入**：

| 請確定 | 時間 | Weight |
| --- | --- | --- |
| Honda |2015-01-01T00:00:01.0000000Z |2000 |
| Toyota |2015-01-01T00:00:02.0000000Z |25000 |
| Honda |2015-01-01T00:00:03.0000000Z |26000 |
| Toyota |2015-01-01T00:00:04.0000000Z |25000 |
| Honda |2015-01-01T00:00:05.0000000Z |26000 |
| Toyota |2015-01-01T00:00:06.0000000Z |25000 |
| Honda |2015-01-01T00:00:07.0000000Z |26000 |
| Toyota |2015-01-01T00:00:08.0000000Z |2000 |

**輸出**：

| StartFault | EndFault |
| --- | --- |
| 2015-01-01T00:00:02.000Z |2015-01-01T00:00:07.000Z |

**解決方案**：

````
    WITH SelectPreviousEvent AS
    (
    SELECT
    *,
        LAG([time]) OVER (LIMIT DURATION(hour, 24)) as previousTime,
        LAG([weight]) OVER (LIMIT DURATION(hour, 24)) as previousWeight
    FROM input TIMESTAMP BY [time]
    )

    SELECT 
        LAG(time) OVER (LIMIT DURATION(hour, 24) WHEN previousWeight < 20000 ) [StartFault],
        previousTime [EndFault]
    FROM SelectPreviousEvent
    WHERE
        [weight] < 20000
        AND previousWeight > 20000
````

**說明**：使用 **LAG** 來檢視 24 小時內的輸入資料流，並尋找其中的 **StartFault** 和 **StopFault** 被 weight < 20000 合併的執行個體。

## <a name="query-example-fill-missing-values"></a>查詢範例：填入遺漏值
**描述**：對於有遺漏值的事件串流，以固定間隔產生事件串流。
例如，每隔 5 秒產生事件，報告最近所見的資料點。

**輸入**：

| t | value |
| --- | --- |
| "2014-01-01T06:01:00" |1 |
| "2014-01-01T06:01:05" |2 |
| "2014-01-01T06:01:10" |3 |
| "2014-01-01T06:01:15" |4 |
| "2014-01-01T06:01:30" |5 |
| "2014-01-01T06:01:35" |6 |

**輸出 (前 10 個資料列)**：

| windowend | lastevent.t | lastevent.value |
| --- | --- | --- |
| 2014-01-01T14:01:00.000Z |2014-01-01T14:01:00.000Z |1 |
| 2014-01-01T14:01:05.000Z |2014-01-01T14:01:05.000Z |2 |
| 2014-01-01T14:01:10.000Z |2014-01-01T14:01:10.000Z |3 |
| 2014-01-01T14:01:15.000Z |2014-01-01T14:01:15.000Z |4 |
| 2014-01-01T14:01:20.000Z |2014-01-01T14:01:15.000Z |4 |
| 2014-01-01T14:01:25.000Z |2014-01-01T14:01:15.000Z |4 |
| 2014-01-01T14:01:30.000Z |2014-01-01T14:01:30.000Z |5 |
| 2014-01-01T14:01:35.000Z |2014-01-01T14:01:35.000Z |6 |
| 2014-01-01T14:01:40.000Z |2014-01-01T14:01:35.000Z |6 |
| 2014-01-01T14:01:45.000Z |2014-01-01T14:01:35.000Z |6 |

**解決方案**：

    SELECT
        System.Timestamp AS windowEnd,
        TopOne() OVER (ORDER BY t DESC) AS lastEvent
    FROM
        input TIMESTAMP BY t
    GROUP BY HOPPINGWINDOW(second, 300, 5)


**說明**：此查詢會每隔 5 秒產生事件，並且會輸出之前收到的最後一個事件。 [跳動視窗](https://msdn.microsoft.com/library/dn835041.aspx "跳動視窗 - Azure 串流分析")持續時間會決定查詢回溯到多久之前以找出最新的事件 (在此範例中為 300 秒)。


## <a name="query-example-correlate-two-event-types-within-the-same-stream"></a>查詢範例：將相同串流中的兩個事件類型相互關聯
**描述**：有時需要根據特定時間範圍內發生的多種事件類型來產生警示。
例如，在家用烤爐的 IoT 案例中，風扇溫度低於 40 且過去 3 分鐘的最大功率低於 10 時，必須發出警示。

**輸入**：

| 分析 | deviceId | sensorName | value |
| --- | --- | --- | --- |
| "2018-01-01T16:01:00" | "Oven1" | "temp" |120 |
| "2018-01-01T16:01:00" | "Oven1" | "power" |15 |
| "2018-01-01T16:02:00" | "Oven1" | "temp" |100 |
| "2018-01-01T16:02:00" | "Oven1" | "power" |15 |
| "2018-01-01T16:03:00" | "Oven1" | "temp" |70 |
| "2018-01-01T16:03:00" | "Oven1" | "power" |15 |
| "2018-01-01T16:04:00" | "Oven1" | "temp" |50 |
| "2018-01-01T16:04:00" | "Oven1" | "power" |15 |
| "2018-01-01T16:05:00" | "Oven1" | "temp" |30 |
| "2018-01-01T16:05:00" | "Oven1" | "power" |8 |
| "2018-01-01T16:06:00" | "Oven1" | "temp" |20 |
| "2018-01-01T16:06:00" | "Oven1" | "power" |8 |
| "2018-01-01T16:07:00" | "Oven1" | "temp" |20 |
| "2018-01-01T16:07:00" | "Oven1" | "power" |8 |
| "2018-01-01T16:08:00" | "Oven1" | "temp" |20 |
| "2018-01-01T16:08:00" | "Oven1" | "power" |8 |

**輸出**：

| eventTime | deviceId | temp | alertMessage | maxPowerDuringLast3mins |
| --- | --- | --- | --- | --- | 
| "2018-01-01T16:05:00" | "Oven1" |30 | "Short circuit heating elements" |15 |
| "2018-01-01T16:06:00" | "Oven1" |20 | "Short circuit heating elements" |15 |
| "2018-01-01T16:07:00" | "Oven1" |20 | "Short circuit heating elements" |15 |

**解決方案**：

````
WITH max_power_during_last_3_mins AS (
    SELECT 
        System.TimeStamp AS windowTime,
        deviceId,
        max(value) as maxPower
    FROM
        input TIMESTAMP BY t
    WHERE 
        sensorName = 'power' 
    GROUP BY 
        deviceId, 
        SlidingWindow(minute, 3) 
)

SELECT 
    t1.t AS eventTime,
    t1.deviceId, 
    t1.value AS temp,
    'Short circuit heating elements' as alertMessage,
    t2.maxPower AS maxPowerDuringLast3mins
    
INTO resultsr

FROM input t1 TIMESTAMP BY t
JOIN max_power_during_last_3_mins t2
    ON t1.deviceId = t2.deviceId 
    AND t1.t = t2.windowTime
    AND DATEDIFF(minute,t1,t2) between 0 and 3
    
WHERE
    t1.sensorName = 'temp'
    AND t1.value <= 40
    AND t2.maxPower > 10
````

**說明**：第一個查詢 `max_power_during_last_3_mins` 會使用[滑動時間範圍](https://msdn.microsoft.com/azure/stream-analytics/reference/sliding-window-azure-stream-analytics)來尋找過去 3 分鐘內每個裝置的功率感應器最大值。 系統會將第二個查詢加入第一個查詢，以找出目前事件最近相關時間範圍內的功率值。 然後，假如條件符合，就會針對裝置產生警示。

## <a name="query-example-process-events-independent-of-device-clock-skew-substreams"></a>查詢範例：與裝置時鐘誤差 (子串流) 無關的處理事件
**描述**：事件會由於事件產生器之間的時鐘誤差、分割之間的時鐘誤差或網路延遲而晚發生或順序錯誤。 在下列範例中，TollID 2 的裝置時鐘比 TollID 1 晚 10 秒，而且 TollID 3 的裝置時鐘比 TollID 1 晚 5 秒。 


**輸入**：
| LicensePlate | 請確定 | 時間 | TollID |
| --- | --- | --- | --- |
| DXE 5291 |Honda |2015-07-27T00:00:01.0000000Z | 1 |
| YHN 6970 |Toyota |2015-07-27T00:00:05.0000000Z | 1 |
| QYF 9358 |Honda |2015-07-27T00:00:01.0000000Z | 2 |
| GXF 9462 |BMW |2015-07-27T00:00:04.0000000Z | 2 |
| VFE 1616 |Toyota |2015-07-27T00:00:10.0000000Z | 1 |
| RMV 8282 |Honda |2015-07-27T00:00:03.0000000Z | 3 |
| MDR 6128 |BMW |2015-07-27T00:00:11.0000000Z | 2 |
| YZK 5704 |Ford |2015-07-27T00:00:07.0000000Z | 3 |

**輸出**：
| TollID | Count |
| --- | --- |
| 1 | 2 |
| 2 | 2 |
| 1 | 1 |
| 3 | 1 |
| 2 | 1 |
| 3 | 1 |

**解決方案**：

````
SELECT
      TollId,
      COUNT(*) AS Count
FROM input
      TIMESTAMP BY Time OVER TollId
GROUP BY TUMBLINGWINDOW(second, 5), TollId

````

**說明**：[TIMESTAMP BY OVER](https://msdn.microsoft.com/azure/stream-analytics/reference/timestamp-by-azure-stream-analytics#over-clause-interacts-with-event-ordering) 子句會使用子串流個別查看每個裝置時間軸。 計算時會產生每個 TollID 的輸出事件，這表示事件的順序均與每個 TollID 有關，而不會重新排列順序，就像所有裝置都依據同一個時鐘一般。


## <a name="get-help"></a>取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)。

## <a name="next-steps"></a>後續步驟
* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [開始使用 Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

