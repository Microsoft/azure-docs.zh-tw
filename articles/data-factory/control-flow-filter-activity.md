---
title: Azure Data Factory 中的篩選活動 | Microsoft Docs
description: 篩選活動會篩選輸入。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/04/2018
ms.author: shlo
ms.openlocfilehash: b3b26869a84b8519ced19a4c93a6d39d6ed20f9b
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37050347"
---
# <a name="filter-activity-in-azure-data-factory"></a>Azure Data Factory 中的篩選活動
您可以在管線中使用篩選活動，將篩選運算式套用至輸入陣列。 

## <a name="syntax"></a>語法

```json
{
    "name": "MyFilterActivity",
    "type": "filter",
    "typeProperties": {
        "condition": "<condition>",
        "items": "<input array>"
    }
}
```

## <a name="type-properties"></a>類型屬性

屬性 | 說明 | 允許的值 | 必要
-------- | ----------- | -------------- | --------
name | `Filter` 活動的名稱。 | 字串 | yes
type | 必須設定為**篩選**。 | 字串 | yes
condition | 要用來篩選輸入的條件。 | 運算是 | yes
項目 | 應套用篩選條件的輸入陣列。 | 運算是 | yes

## <a name="example"></a>範例

在此範例中，管線有兩個活動：**篩選**與 **ForEach**。 篩選活動設定為會對值大於 3 之項目的輸入陣列進行篩選。 接著，ForEach 活動會逐一查看篩選值，然後等候目前的值所指定的秒數。

```json
{
    "name": "PipelineName",
    "properties": {
        "activities": [{
                "name": "MyFilterActivity",
                "type": "filter",
                "typeProperties": {
                    "condition": "@greater(item(),3)",
                    "items": "@pipeline().parameters.inputs"
                }
            },
            {
                "name": "MyForEach",
                "type": "ForEach",
                "typeProperties": {
                    "isSequential": "false",
                    "batchCount": 1,
                    "items": "@activity('MyFilterActivity').output.value",
                    "activities": [{
                        "type": "Wait",
                        "typeProperties": {
                            "waitTimeInSeconds": "@item()"
                        },
                        "name": "MyWaitActivity"
                    }]
                },
                "dependsOn": [{
                    "activity": "MyFilterActivity",
                    "dependencyConditions": ["Succeeded"]
                }]
            }
        ],
        "parameters": {
            "inputs": {
                "type": "Array",
                "defaultValue": [1, 2, 3, 4, 5, 6]
            }
        }
    }
}
```

## <a name="next-steps"></a>後續步驟
請參閱 Data Factory 支援的其他控制流程活動： 

- [If Condition 活動](control-flow-if-condition-activity.md)
- [執行管道活動](control-flow-execute-pipeline-activity.md)
- [For Each 活動](control-flow-for-each-activity.md)
- [取得中繼資料活動](control-flow-get-metadata-activity.md)
- [查閱活動](control-flow-lookup-activity.md)
- [Web 活動](control-flow-web-activity.md)
- [Until 活動](control-flow-until-activity.md)
