---
title: LUIS 預先建置的實體 datetimeV2 參考 - Azure| Microsoft Docs
titleSuffix: Azure
description: 本文包含 Language Understanding (LUIS) 中 datetimeV2 預先建置的實體資訊。
services: cognitive-services
author: diberry
manager: cjgronlund
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 06/20/2018
ms.author: diberry
ms.openlocfilehash: 13f62e98a33aac51eae86d5ce1b802d4701ef3f6
ms.sourcegitcommit: 194789f8a678be2ddca5397137005c53b666e51e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2018
ms.locfileid: "39236766"
---
# <a name="datetimev2-entity"></a>DatetimeV2 實體

**datetimeV2** 預先建置的實體會擷取日期和時間值。 這些值會以標準化格式解析，以供用戶端程式取用。 當語句中有不完整的日期或時間時，LUIS 會在端點回應中包括「過去與未來值」。 因為此實體已經定型，所以您不需要將包含 datetimeV2 的範例語句加入應用程式意圖。 

## <a name="types-of-datetimev2"></a>datetimeV2 類型
DatetimeV2 可從 [Recognizers-text](https://github.com/Microsoft/Recognizers-Text/blob/master/Patterns/English/English-DateTime.yaml) GitHub 存放庫管理

## <a name="example-json"></a>範例 JSON 
下列範例 JSON 回應有子類型為 `datetime` 的 `datetimeV2` 實體。 如需其他 datetimeV2 實體類型的範例，請參閱 [datetimeV2 子類型](#subtypes-of-datetimev2)</a>。

```JSON
"entities": [
  {
    "entity": "8am on may 2nd 2017",
    "type": "builtin.datetimeV2.datetime",
    "startIndex": 15,
    "endIndex": 30,
    "resolution": {
      "values": [
        {
          "timex": "2017-05-02T08",
          "type": "datetime",
          "value": "2017-05-02 08:00:00"
        }
      ]
    }
  }
]
  ```

## <a name="json-property-descriptions"></a>JSON 屬性描述

|屬性名稱 |屬性類型和描述|
|---|---|
|實體|**string** - 以日期、時間、日期範圍或時間範圍的類型從語句中擷取的文字。|
|type|**string** - 其中一個 [datetimeV2 子類型](#subtypes-of-datetimev2)
|startIndex|**int** - 實體開始的語句中的索引。|
|endIndex|**int** - 實體結束的語句中的索引。|
|resolution|有一、二或四個[解析值](#values-of-resolution)的 `values` 陣列。|
|end|時間或日期範圍的結束值，使用的格式和 `value` 相同。 只有當 `type` 為 `daterange`、`timerange` 或 `datetimerange` 時才能使用|

## <a name="subtypes-of-datetimev2"></a>datetimeV2 子類型

**datetimeV2** 預先建置的實體具有下列子類型，而個別的範例均在下表提供：
* `date`
* `time`
* `daterange`
* `timerange`
* `datetimerange`
* `duration`
* `set`

## <a name="values-of-resolution"></a>解析值
* 如果語句中指定的日期或時間完整且明確，陣列會有一個元素。
* 如果 datetimeV2 值語意模糊，陣列會有兩個元素。 語意模糊包括缺少特定年份、時間或時間範圍。 如需範例，請參閱[語意模糊的日期](#ambiguous-dates)。 若時間的上午 或下午語意模糊，即會包含這兩個值。
* 如果語句有兩個語意模糊的元素，陣列會有四個元素。 這包括以下語意模糊的元素：
  * 日期或日期範圍的年份語意模糊
  * 時間或時間範圍的上午或下午 語意模糊。 例如，4 月 3 日 3:00。

`values` 陣列的每個元素可能都有下列欄位： 

|屬性名稱|屬性描述|
|--|--|
|timex|以遵循 [ISO 8601 標準](https://en.wikipedia.org/wiki/ISO_8601)的 TIMEX 格式表示的時間、日期或日期範圍，並對註解的 TIMEX3 屬性使用 TimeML 語言。 此註解會在 [TIMEX 指導方針](http://www.timeml.org/tempeval2/tempeval2-trial/guidelines/timex3guidelines-072009.pdf) \(英文\) 中說明。|
|type|可以是下列其中一個項目的子類型：datetime、date、time、daterange、timerange、datetimerange、duration、set。|
|value|**選用。** 使用 yyyy:MM:dd (date)，HH:mm:ss (time) yyyy:MM:dd HH:mm:ss (datetime) 格式的 datetime 物件。 如果 `type` 是 `duration`，則值會是秒數 (duration) <br/> 只有當 `type` 是 `datetime` 或 `date`、`time` 或 `duration 時才能使用。|

## <a name="valid-date-values"></a>有效的日期值

**datetimeV2** 支援的日期範圍如下：

| Min | max |
|----------|-------------|
| 1900 年 1 月 1 日   | 2099 年 12 月 31 日 |

## <a name="ambiguous-dates"></a>語意模糊的日期

如果可以是過去或未來的日期，LUIS 會兩個值都提供。 例如，語句包含月份與日期，但不含年份。  

例如，假設該語句為「5 月 2 日」：
* 如果今天的日期為 2017 年 5 月 3 日，LUIS 會提供 "2017-05-02" 與 "2018-05-02" 這兩個值。 
* 當今天的日期為 2017 年 5 月 1 日時，LUIS 會提供 "2016-05-02" 與 "2017-05-02" 這兩個值。

下列範例顯示「5 月 2 日」實體的解析。 此解析假設今天的日期介於 2017 年 5 月 2 日到 2018 年 5 月 1 日之間。
`timex` 欄位中包含 `X` 的欄位，是語句中未明確指定的日期部分。

```JSON
  "entities": [
    {
      "entity": "may 2nd",
      "type": "builtin.datetimeV2.date",
      "startIndex": 0,
      "endIndex": 6,
      "resolution": {
        "values": [
          {
            "timex": "XXXX-05-02",
            "type": "date",
            "value": "2017-05-02"
          },
          {
            "timex": "XXXX-05-02",
            "type": "date",
            "value": "2018-05-02"
          }
        ]
      }
    }
  ]
```

## <a name="date-range-resolution-examples-for-numeric-date"></a>數值日期的日期範圍解析範例

`datetimeV2` 實體會擷取日期和時間範圍。 `start` 和 `end` 欄位可指定範圍的開頭和結尾。 針對「5 月 2 日至 5 月 5 日」語句，LUIS 會提供目前年度與明年的 **daterange** 值。 在 `timex` 欄位中，`XXXX` 值表示年份的語意模糊。 `P3D` 表示時間週期長達三天。

```JSON
"entities": [
    {
      "entity": "may 2nd to may 5th",
      "type": "builtin.datetimeV2.daterange",
      "startIndex": 0,
      "endIndex": 17,
      "resolution": {
        "values": [
          {
            "timex": "(XXXX-05-02,XXXX-05-05,P3D)",
            "type": "daterange",
            "start": "2017-05-02",
            "end": "2017-05-05"
          },
          {
            "timex": "(XXXX-05-02,XXXX-05-05,P3D)",
            "type": "daterange",
            "start": "2018-05-02",
            "end": "2018-05-05"
          }
        ]
      }
    }
  ]
```

## <a name="date-range-resolution-examples-for-day-of-week"></a>星期幾的日期範圍解析範例

下列範例示範 LUIS 如何使用 **datetimeV2** 解析「星期二至星期四」語句。 在此範例中，目前日期為 6 月 19 日。 LUIS 包含兩個日期範圍的 **daterange** 值，分別在目前日期之前與之後。

```JSON
  "entities": [
    {
      "entity": "tuesday to thursday",
      "type": "builtin.datetimeV2.daterange",
      "startIndex": 0,
      "endIndex": 19,
      "resolution": {
        "values": [
          {
            "timex": "(XXXX-WXX-2,XXXX-WXX-4,P2D)",
            "type": "daterange",
            "start": "2017-06-13",
            "end": "2017-06-15"
          },
          {
            "timex": "(XXXX-WXX-2,XXXX-WXX-4,P2D)",
            "type": "daterange",
            "start": "2017-06-20",
            "end": "2017-06-22"
          }
        ]
      }
    }
  ]
```
## <a name="ambiguous-time"></a>語意模糊的時間
如果時間或時間範圍的語意模糊，值陣列會有兩個時間元素。 有語意模糊的時間時，就會有上午 與下午 兩個時間值。

## <a name="time-range-resolution-example"></a>時間範圍解析範例

下列範例示範 LUIS 如何使用 **datetimeV2** 解析有時間範圍的語句。

```
  "entities": [
    {
      "entity": "6pm to 7pm",
      "type": "builtin.datetimeV2.timerange",
      "startIndex": 0,
      "endIndex": 9,
      "resolution": {
        "values": [
          {
            "timex": "(T18,T19,PT1H)",
            "type": "timerange",
            "start": "18:00:00",
            "end": "19:00:00"
          }
        ]
      }
    }
  ]
```

## <a name="deprecated-prebuilt-datetime"></a>已被取代的預先建置 datetime

`datetime` 預先建置的實體已過時 ，並由 **datetimeV2** 取代。 

若要在 LUIS 應用程式中以 `datetimeV2` 取代 `datetime`，請完成下列步驟：

1. 開啟 LUIS 網頁介面的 [實體] 窗格。 
2. 刪除 **datetime** 預先建置的實體。
3. 按一下 [新增預先建置的實體]
4. 選取 [datetimeV2]，然後按一下 [儲存]。

## <a name="next-steps"></a>後續步驟

了解 [dimension](luis-reference-prebuilt-dimension.md)、[email](luis-reference-prebuilt-email.md) 實體與 [number](luis-reference-prebuilt-number.md) 的相關資訊。 

