---
title: Azure Log Analytics 查詢中的聯結 | Microsoft Docs
description: 此文章包括在 Log Analytics 查詢語言中使用聯結的課程。
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
ms.openlocfilehash: 39a461a27e8d9d6d1b9712449586bfabf6124d22
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46989442"
---
# <a name="joins-in-log-analytics-queries"></a>Log Analytics 查詢中的聯結

> [!NOTE]
> 您應該先完成[開始使用 Analytics 入口網站](get-started-analytics-portal.md)與[開始使用查詢](get-started-queries.md)，再完成此課程。

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

聯結可讓您在相同的查詢中分析多個資料表的資料。 它們會透過比對所指定資料行的值來合併兩個資料集的資料列。


```Kusto
SecurityEvent 
| where EventID == 4624     // sign-in events
| project Computer, Account, TargetLogonId, LogonTime=TimeGenerated
| join kind= inner (
    SecurityEvent 
    | where EventID == 4634     // sign-out events
    | project TargetLogonId, LogoffTime=TimeGenerated
) on TargetLogonId
| extend Duration = LogoffTime-LogonTime
| project-away TargetLogonId1 
| top 10 by Duration desc
```

在此範例中，第一個資料集會篩選所有登入事件。 這與會篩選所有登出事件的第二個資料集聯結。 投影的資料行是 _Computer_、_Account_、_TargetLogonId_ 與 _TimeGenerated_。 兩個資料集會以共用資料行 _TargetLogonId_ 彼此關聯。 輸出是每個關聯的單一記錄，它同時包含登入與登出時間。

若兩個資料集都有具有相同名稱的資料行，系統將會為右側資料集的資料行指定索引編號，因此在此範例中，結果會顯示具有來自左側資料表之值的 _TargetLogonId_，以及具有來自右側資料表之值的 _TargetLogonId1_ 。 在此案例中，第二個 _TargetLogonId1_ 資料行會使用 `project-away` 運算子移除。

> [!NOTE]
> 若要改進效能，請使用 `project` 運算子，只保留聯結資料集的相關資料行。


使用下列語法來聯結兩個資料集，而且聯結的索引鍵在兩個資料表有不同的名稱：
```
Table1
| join ( Table2 ) 
on $left.key1 == $right.key2
```

## <a name="lookup-tables"></a>查閱資料表
聯結的其中一個常用用途是使用值的靜態對應 (使用有助於將結果轉換為更易讀之格式的 `datatable`)。 例如，為每個事件識別碼加上事件名稱以讓安全性事件資料更豐富。

```Kusto
let DimTable = datatable(EventID:int, eventName:string)
  [
    4625, "Account activity",
    4688, "Process action",
    4634, "Account activity",
    4658, "The handle to an object was closed",
    4656, "A handle to an object was requested",
    4690, "An attempt was made to duplicate a handle to an object",
    4663, "An attempt was made to access an object",
    5061, "Cryptographic operation",
    5058, "Key file operation"
  ];
SecurityEvent
| join kind = inner
 DimTable on EventID
| summarize count() by eventName
```

![與資料表聯結](media/joins/dim-table.png)

## <a name="join-kinds"></a>聯結類型
使用 _kind_ 引數指定聯結類型。 每種類型都會在給定資料表的記錄之間執行不同的比對，如下表所示。

| 聯結類型 | 說明 |
|:---|:---|
| innerunique | 這是預設聯結模式。 首先，系統會找到左側資料表上的相符資料行，然後移除重複值。  接著，會和右側資料表比對唯一值集合。 |
| inner | 只有兩個資料表中都有的相符記錄才會包含在結果中。 |
| leftouter | 左側資料表中的所有記錄與右側資料表中的相符記錄都會包括在結果中。 不相符的輸出屬性包含 Null。  |
| leftanti | 來自左側且未與右側相符的記錄會包括在結果中。 結果資料表只有來自左側的資料行。 |
| leftsemi | 來自左側且與右側相符的記錄會包括在結果中。 結果資料表只有來自左側的資料行。 |


## <a name="best-practices"></a>最佳作法

為獲得最佳效能，請考慮以下各點：

- 在每個資料表上使用時間篩選以減少必須針對聯結評估的記錄數目。
- 在聯結之前，使用 `where` 與 `project` 來減少輸入資料表中的資料列與資料行數目。
* 如果某個資料表永遠都比另一個資料表還小，請將它做為聯結的左側。


## <a name="next-steps"></a>後續步驟
請參閱其他課程以了解如何使用 Log Analytics 查詢語言：

- [字串作業](string-operations.md)
- [彙總函式](aggregations.md)
- [進階彙總](advanced-aggregations.md)
- [JSON 與資料結構](json-data-structures.md)
- [進階查詢撰寫](advanced-query-writing.md)
- [圖表](charts.md)