---
title: 在 Azure Log Analytics 查詢中處理字串 | Microsoft Docs
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
ms.openlocfilehash: ff0514a3432ed74baa6df2574157066daff895bb
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46961258"
---
# <a name="working-with-json-and-data-structures-in-log-analytics-queries"></a>在 Log Analytics 查詢中處理 JSON 與資料結構

> [!NOTE]
> 您應該先完成[開始使用 Analytics 入口網站](get-started-analytics-portal.md)與[開始使用查詢](get-started-queries.md)，再完成此課程。

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

巢狀物件是在陣列或機碼值組中包含其他物件的物件。 這些物件是以 JSON 字串來表示。 此文章說明如何使用 JSON 來擷取資料並分析巢狀物件。

## <a name="working-with-json-strings"></a>處理 JSON 字串
使用 `extractjson` 來存取已知路徑中的特定 JSON 元素。 此功能需要使用下列慣例的路徑運算式。

- _$_ 以代表根資料夾
- 使用括弧或點標記法來代表索引與元素，如下列範例中所示。


使用括弧來代表索引，並使用點來分隔元素：

```Kusto
let hosts_report='{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"location":"South_DC", "status":"stopped", "rate":3}]}';
print hosts_report
| extend status = extractjson("$.hosts[0].status", hosts_report)
```

這與只使用括弧標記法的結果相同：

```Kusto
let hosts_report='{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"location":"South_DC", "status":"stopped", "rate":3}]}';
print hosts_report 
| extend status = extractjson("$['hosts'][0]['status']", hosts_report)
```

若只有一個元素，您可以只使用點標記法：

```Kusto
let hosts_report='{"location":"North_DC", "status":"running", "rate":5}';
print hosts_report 
| extend status = hosts_report.status
```


## <a name="working-with-objects"></a>處理物件

### <a name="parsejson"></a>parsejson
若要存取您 json 結構中的多個元素，以動態物件方式來存取比較簡單。 使用 `parsejson` 將文字資料轉換為動態物件。 一旦轉換為動態型別，就能使用額外函式來分析資料。

```Kusto
let hosts_object = parsejson('{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"location":"South_DC", "status":"stopped", "rate":3}]}');
print hosts_object 
| extend status0=hosts_object.hosts[0].status, rate1=hosts_object.hosts[1].rate
```



### <a name="arraylength"></a>arraylength
使用 `arraylength` 來計算陣列中的元素數目：

```Kusto
let hosts_object = parsejson('{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"location":"South_DC", "status":"stopped", "rate":3}]}');
print hosts_object 
| extend hosts_num=arraylength(hosts_object.hosts)
```

### <a name="mvexpand"></a>mvexpand
使用 `mvexpand` 將物件的屬性分成數列。

```Kusto
let hosts_object = parsejson('{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"location":"South_DC", "status":"stopped", "rate":3}]}');
print hosts_object 
| mvexpand hosts_object.hosts[0]
```

![mvexpand](media/json-data-structures/mvexpand.png)

### <a name="buildschema"></a>buildschema
使用 `buildschema` 來取得認可物件所有值的結構描述：

```Kusto
let hosts_object = parsejson('{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"location":"South_DC", "status":"stopped", "rate":3}]}');
print hosts_object 
| summarize buildschema(hosts_object)
```

輸出是 JSON 格式的結構描述：
```json
{
    "hosts":
    {
        "indexer":
        {
            "location": "string",
            "rate": "int",
            "status": "string"
        }
    }
}
```
此輸出描述物件欄位的名稱與其對應的資料類型。 

巢狀物件可能有不同的結構描述，如下列範例中所示：

```Kusto
let hosts_object = parsejson('{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"status":"stopped", "rate":"3", "range":100}]}');
print hosts_object 
| summarize buildschema(hosts_object)
```


![建置結構描述](media/json-data-structures/buildschema.png)

## <a name="next-steps"></a>後續步驟
請參閱其他課程以了解如何使用 Log Analytics 查詢語言：

- [字串作業](string-operations.md)
- [日期和時間作業](datetime-operations.md)
- [彙總函式](aggregations.md)
- [進階彙總](advanced-aggregations.md)
- [進階查詢撰寫](advanced-query-writing.md)
- [聯結](joins.md)
- [圖表](charts.md)