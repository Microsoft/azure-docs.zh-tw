---
title: 在 Azure 串流分析中處理可設定閾值型規則
description: 本文說明如何使用參考資料在 Azure 串流分析中達成具有可設定閾值型規則的警示解決方案。
services: stream-analytics
author: zhongc
ms.author: zhongc
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 04/30/2018
ms.openlocfilehash: 802be1ad5b1029add249430ee7760002407c4641
ms.sourcegitcommit: 0fa8b4622322b3d3003e760f364992f7f7e5d6a9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37021620"
---
# <a name="process-configurable-threshold-based-rules-in-azure-stream-analytics"></a>在 Azure 串流分析中處理可設定閾值型規則
本文說明如何使用參考資料在 Azure 串流分析中達成使用可設定閾值型規則的警示解決方案。

## <a name="scenario-alerting-based-on-adjustable-rule-thresholds"></a>案例：根據可調整的規則閾值發出警示
當傳入的串流事件達到特定的值，或根據傳入串流事件的彙總值超出特定閾值時，您可能需要產生警示作為輸出。 設定串流分析查詢，來比較值與靜態閾值 (這是固定且預先決定的) 非常簡單。 您可以將固定的閾值以硬式編碼寫在使用簡單數值比較 (大於、小於和等於) 的串流查詢語法中。

在某些情況下，閾值需要能更輕鬆地進行設定，而不需在每次閾值變更時都要編輯查詢語法。 而在其他情況中，您可能需要利用相同的查詢來處理多個裝置或使用者 (每個使用者在每種裝置種類上都有不同的閾值)。 

此模式可用來動態設定閾值、選擇性地依篩選的輸入資料選擇要套用閾值的裝置種類，以及選擇性地選擇輸出中要包括哪些欄位。

## <a name="recommended-design-pattern"></a>建議的設計模式
使用輸入至串流分析工作的參考資料作為警示閾值的查閱：
- 將閾值儲存在參考資料中，每個索引鍵一個值。
- 將串流資料輸入事件與索引鍵資料行上的參考資料聯結。
- 使用從參考資料建立的索引鍵值作為閾值。

## <a name="example-data-and-query"></a>範例資料和查詢
在範例中，警示產生的時機是在一分鐘長度的時間範圍內，來自裝置流入的資料串流彙總，與規則中以參考資料提供的規定值相符時。

在查詢中，針對每個 deviceId 和 deviceId 下的每個 metricName，可對 GROUP BY 設定 0 到 5 的維度。 只有具有對應篩選值的事件會被分組。 分組之後，具範圍限制的 Min、Max、Avg 彙總會透過 60 秒的輪轉時間範圍計算。 彙總值的篩選接著會以參考資料中每個設定的閾值來進行計算，以產生警示輸出事件。

作為範例，我們假設有一個串流分析工作具有名稱為 **rules** 的參考資料輸入，而串流資料輸入的名稱為 **metrics**。 

## <a name="reference-data"></a>參考資料
此範例參考資料說明如何呈現閾值型規則。 JSON 檔案會保留參考資料，並儲存到 Azure Blob 儲存體中，而該 Blob 儲存體容器會用來當作名稱為 **rules** 的參考資料輸入。 您可以隨時間經過覆寫此 JSON 檔案並替換規則設定，無須停止或啟動串流工作。

- 範例規則是用來在 CPU 超出 (平均值大於或等於) 值 `90` 百分比時，表示可調整的警示。 `value` 欄位可視需要進行設定。
- 請注意，規則有 **operator** 欄位，這會在查詢語法後段以動態方式解譯 `AVGGREATEROREQUAL`。 
- 規則會針對具有值 `C1` 的特定維度索引鍵 `2` 篩選資料。 其他的欄位是空白字串，表示不會透過那些事件欄位篩選輸入串流。 您可以設定額外的 CPU 規則，來視需要篩選其他相符的欄位。
- 並非所有的資料行都包含在輸出警示事件中。 在本案例中，`includedDim` 索引鍵的 `2` 號被開啟為 `TRUE`，表示串流當中的事件資料 2 號欄位將會包含在合格的輸出事件中。 其他欄位則不會包含在警示輸出中，但您可以調整欄位清單。


```json
{
    "ruleId": 1234, 
    "deviceId" : "978648", 
    "metricName": "CPU", 
    "alertName": "hot node AVG CPU over 90",
    "operator" : "AVGGREATEROREQUAL",
    "value": 90, 
    "includeDim": {
        "0": "FALSE", 
        "1": "FALSE", 
        "2": "TRUE", 
        "3": "FALSE", 
        "4": "FALSE"
    },
    "filter": {
        "0": "", 
        "1": "",
        "2": "C1", 
        "3": "", 
        "4": ""
    }    
}
```

## <a name="example-streaming-query"></a>範例串流查詢
此範例串流分析查詢會將上述範例的 **rules** 參考資料聯結到名稱為 **metrics** 的資料輸入串流。

```sql
WITH transformedInput AS
(
    SELECT
        dim0 = CASE rules.includeDim.[0] WHEN 'TRUE' THEN metrics.custom.dimensions.[0].value ELSE NULL END,
        dim1 = CASE rules.includeDim.[1] WHEN 'TRUE' THEN metrics.custom.dimensions.[1].value ELSE NULL END,
        dim2 = CASE rules.includeDim.[2] WHEN 'TRUE' THEN metrics.custom.dimensions.[2].value ELSE NULL END,
        dim3 = CASE rules.includeDim.[3] WHEN 'TRUE' THEN metrics.custom.dimensions.[3].value ELSE NULL END,
        dim4 = CASE rules.includeDim.[4] WHEN 'TRUE' THEN metrics.custom.dimensions.[4].value ELSE NULL END,
        metric = metrics.metric.value,
        metricName = metrics.metric.name,
        deviceId = rules.deviceId, 
        ruleId = rules.ruleId, 
        alertName = rules.alertName,
        ruleOperator = rules.operator, 
        ruleValue = rules.value
    FROM 
        metrics
        timestamp by eventTime
    JOIN 
        rules
        ON metrics.deviceId = rules.deviceId AND metrics.metric.name = rules.metricName
    WHERE
        (rules.filter.[0] = '' OR metrics.custom.filters.[0].value = rules.filter.[0]) AND 
        (rules.filter.[1] = '' OR metrics.custom.filters.[1].value = rules.filter.[1]) AND
        (rules.filter.[2] = '' OR metrics.custom.filters.[2].value = rules.filter.[2]) AND
        (rules.filter.[3] = '' OR metrics.custom.filters.[3].value = rules.filter.[3]) AND
        (rules.filter.[4] = '' OR metrics.custom.filters.[4].value = rules.filter.[4])
)

SELECT
    System.Timestamp as time, 
    transformedInput.deviceId as deviceId,
    transformedInput.ruleId as ruleId,
    transformedInput.metricName as metric,
    transformedInput.alertName as alert,
    AVG(metric) as avg,
    MIN(metric) as min, 
    MAX(metric) as max, 
    dim0, dim1, dim2, dim3, dim4
FROM
    transformedInput
GROUP BY
    transformedInput.deviceId,
    transformedInput.ruleId,
    transformedInput.metricName,
    transformedInput.alertName,
    dim0, dim1, dim2, dim3, dim4,
    ruleOperator, 
    ruleValue, 
    TumblingWindow(second, 60)
HAVING
    (
        (ruleOperator = 'AVGGREATEROREQUAL' AND avg(metric) >= ruleValue) OR
        (ruleOperator = 'AVGEQUALORLESS' AND avg(metric) <= ruleValue) 
    )
```

## <a name="example-streaming-input-event-data"></a>範例串流輸入事件資料
此範例 JSON 資料表示用於上述串流查詢中的 **metrics** 輸入資料。 

- 在 1 分鐘的時間範圍中列出了三個範例事件，值 `T14:50`。 
- 三個範例都有相同的 `deviceId` 值 `978648`。
- CPU 的計量值在每個事件中都有所不同，分別是 `98`、`95`、`80`。 只有前兩個範例事件的值超出規則中建立的 CPU 警示規則。
- 警示規則中的 includeDim 欄位是索引鍵 2 號。 範例事件中所對應到的索引鍵 2 欄位名稱為 `NodeName`。 三個範例事件的值分別是 `N024`、`N024` 和 `N014`。 在輸出中，您會看到只有節點 `N024`，因為它是符合高 CPU 警示準則的唯一資料。 `N014` 不符合高 CPU 閾值。
- 使用 `filter` 設定的警示規則只針對索引鍵 2 號，也就是對應到範例事件中的 `cluster` 欄位。 這三個範例事件全都具有值 `C1`，且符合篩選準則。

```json
{
    "eventTime": "2018-04-30T14:50:23.1324132Z",
    "deviceId": "978648",
    "custom": {
        "dimensions": {
            "0": {
                "name": "NodeType",
                "value": "N1"
            },
            "1": {
                "name": "Cluster",
                "value": "C1"
            },
            "2": {
                "name": "NodeName",
                "value": "N024"
            }
        },
        "filters": {
            "0": {
                "name": "application",
                "value": "A1"
            },
            "1": {
                "name": "deviceType",
                "value": "T1"
            },
            "2": {
                "name": "cluster",
                "value": "C1"
            },
            "3": {
                "name": "nodeType",
                "value": "N1"
            }
        }
    },
    "metric": {
        "name": "CPU",
        "value": 98,
        "count": 1.0,
        "min": 98,
        "max": 98,
        "stdDev": 0.0
    }
}
{
    "eventTime": "2018-04-30T14:50:24.1324138Z",
    "deviceId": "978648",
    "custom": {
        "dimensions": {
            "0": {
                "name": "NodeType",
                "value": "N2"
            },
            "1": {
                "name": "Cluster",
                "value": "C1"
            },
            "2": {
                "name": "NodeName",
                "value": "N024"
            }
        },
        "filters": {
            "0": {
                "name": "application",
                "value": "A1"
            },
            "1": {
                "name": "deviceType",
                "value": "T1"
            },
            "2": {
                "name": "cluster",
                "value": "C1"
            },
            "3": {
                "name": "nodeType",
                "value": "N2"
            }
        }
    },
    "metric": {
        "name": "CPU",
        "value": 95,
        "count": 1,
        "min": 95,
        "max": 95,
        "stdDev": 0
    }
}
{
    "eventTime": "2018-04-30T14:50:37.1324130Z",
    "deviceId": "978648",
    "custom": {
        "dimensions": {
            "0": {
                "name": "NodeType",
                "value": "N3"
            },
            "1": {
                "name": "Cluster",
                "value": "C1 "
            },
            "2": {
                "name": "NodeName",
                "value": "N014"
            }
        },
        "filters": {
            "0": {
                "name": "application",
                "value": "A1"
            },
            "1": {
                "name": "deviceType",
                "value": "T1"
            },
            "2": {
                "name": "cluster",
                "value": "C1"
            },
            "3": {
                "name": "nodeType",
                "value": "N3"
            }
        }
    },
    "metric": {
        "name": "CPU",
        "value": 80,
        "count": 1,
        "min": 80,
        "max": 80,
        "stdDev": 0
    }
}
```

## <a name="example-output"></a>範例輸出
此範例輸出 JSON 資料顯示了單一警示事件，此事件是根據參考資料中定義的 CPU 閾值規則而產生。 輸出事件包含警示名稱以及所考量欄位的彙總 (平均、最小、最大)。 由於規則設定的關係，輸出事件資料會包含欄位索引鍵 2 號 `NodeName` 的值 `N024`。 (為了清楚起見，此 JSON 已經過調整以顯示分行符號)。

```JSON
{"time":"2018-05-01T02:03:00.0000000Z","deviceid":"978648","ruleid":1234,"metric":"CPU",
"alert":"hot node AVG CPU over 90","avg":96.5,"min":95.0,"max":98.0,
"dim0":null,"dim1":null,"dim2":"N024","dim3":null,"dim4":null}
```