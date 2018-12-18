---
title: 在 Azure 時間序列深入解析查詢中成形 JSON 的最佳做法。
description: 了解如何改善您的時間序列深入解析查詢效率。
services: time-series-insights
author: ashannon7
manager: cshankar
ms.service: time-series-insights
ms.topic: article
ms.date: 05/24/2018
ms.author: anshan
ms.openlocfilehash: 11bea78315ff7ebb4b0c167dbb687ce940907527
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39628907"
---
# <a name="how-to-shape-json-to-maximize-query-performance"></a>如何成形 JSON 來將查詢效能最大化 

本文提供成形 JSON 來將 Azure 時間序列深入解析 (TSI) 查詢效率最大化的指引。

## <a name="best-practices"></a>最佳作法

請務必考慮如何將事件傳送至時間序列深入解析。 也就是說，您應該一律：

1. 盡可能有效率地透過網路傳送資料。
2. 確定您的資料儲存方式可讓您執行適合您情況的彙總。
3. 確定您未達到 TSI 的最大屬性限制
   - S1 環境為 600 個屬性 (資料行)。
   - S2 環境為 800 個屬性 (資料行)。

下列指引有助於確保盡可能獲得最佳查詢效能：

1. 請勿使用動態屬性，例如以標記識別項作為屬性名稱，因為這樣做會導致達到最大屬性限制。
2. 請勿傳送不必要的屬性。 如果不需要某個查詢屬性，最好不要傳送它，以避免達到儲存空間限制。
3. 使用[參考資料](time-series-insights-add-reference-data-set.md)，以避免透過網路傳送靜態資料。
4. 在多個事件之間共用維度屬性，以更有效率地透過網路傳送資料。
5. 請勿使用深層陣列巢狀結構。 TSI 最多支援兩層包含物件的巢狀陣列。 TSI 會將訊息中的陣列壓平合併為具有屬性值組的多個事件。
6. 如果所有或大部分的事件只存在少數的量值，最好將這些量值當作相同物件內的個別屬性來傳送。 個別傳送可減少事件數目，而且由於需要處理的事件數目較少，因此查詢效能可能會較佳。 有數個量值時，將它們當作單一屬性中的值來傳送，可降低達到最大屬性限制的可能性。

## <a name="examples"></a>範例

下列兩個範例示範傳送事件，並強調上述建議。 在每個範例之後，您可以查看如何套用建議。

這些範例是根據多部裝置傳送度量或訊號的案例而來。 這些度量或訊號可以是流動率、引擎機油壓力、溫度和溼度。 在第一個範例中，有幾個度量會跨所有裝置。 在第二個範例中，有許多裝置，每部裝置會傳送許多唯一的度量。

### <a name="scenario-only-a-few-measurementssignals-exist"></a>案例：只存在幾個度量/訊號

**建議：** 將每個度量當作個別屬性/資料行來傳送。

在下列範例中，有一個 IoT 中樞訊息，其中外部陣列包含通用維度值的共用區段。 外部陣列使用參考資料來提高訊息的效率。 參考資料包含裝置中繼資料，不會隨每個事件變更，但會提供有用的屬性以進行資料分析。 批次處理通用維度值，以及採用參考資料，都可以節省透過網路傳送的位元組數目，讓訊息更有效率。

範例 JSON 承載：

```json
[
    {
        "deviceId": "FXXX",
        "timestamp": "2018-01-17T01:17:00Z",
        "series": [
            {
                "Flow Rate ft3/s": 1.0172575712203979,
                "Engine Oil Pressure psi ": 34.7
            },
            {
                "Flow Rate ft3/s": 2.445906400680542,
                "Engine Oil Pressure psi ": 49.2
            }
        ]
    },
    {
        "deviceId": "FYYY",
        "timestamp": "2018-01-17T01:18:00Z",
        "series": [
            {
                "Flow Rate psi": 0.58015072345733643,
                "Engine Oil Pressure psi ": 22.2
            }
        ]
    }
]
```

參考資料表 (索引鍵屬性為 deviceId)：

| deviceId | messageId | deviceLocation |
| --- | --- | --- |
| FXXX | LINE\_DATA | EU |
| FYYY | LINE\_DATA | US |

時間序列見解事件資料表 (壓平合併後)：

| deviceId | messageId | deviceLocation | timestamp | series.Flow Rate ft3/s | series.Engine Oil Pressure psi |
| --- | --- | --- | --- | --- | --- |
| FXXX | LINE\_DATA | EU | 2018-01-17T01:17:00Z | 1.0172575712203979 | 34.7 |
| FXXX | LINE\_DATA | EU | 2018-01-17T01:17:00Z | 2.445906400680542 | 49.2 |
| FYYY | LINE\_DATA | US | 2018-01-17T01:18:00Z | 0.58015072345733643 | 22.2 |

請注意上述範例中的下列重點：

- **deviceId** 資料行是車隊中各種裝置的資料行標頭。 嘗試將 deviceId 值設為其本身的屬性名稱，搭配其他五個資料行，會將總裝置數限制為 595 (S1 環境) 或 795 (S2 環境)。

- 避免不必要的屬性，例如廠牌和型號資訊等。由於未來不會查詢這些屬性，因此予以排除可改善網路和儲存空間效率。

- 使用參考資料來減少透過網路傳輸的位元組數目。 兩個屬性 **messageId** 和 **deviceLocation** 會透過索引鍵屬性 **deviceId** 聯結。 此資料會在輸入時與遙測資料聯結，並接著儲存在 TSI 中以供查詢。

- 使用兩層的巢狀結構，這是 TSI 支援的最大巢狀結構數量。 請務必避免巢狀結構很深的陣列。

- 量值會當作相同物件內的個別屬性來傳送，因為量值數目很少。 在本例中，**series.Flow Rate psi** 和 **series.Engine Oil Pressure ft3/s** 是唯一的資料行。

### <a name="scenario-several-measures-exist"></a>案例：存在數個量值

**建議：** 將度量當作 "type"、"unit"、"value" Tuple 來傳送。

範例 JSON 承載：

```json
[
    {
        "deviceId": "FXXX",
        "timestamp": "2018-01-17T01:17:00Z",
        "series": [
            {
                "tagId": "pumpRate",
                "value": 1.0172575712203979
            },
            {
                "tagId": "oilPressure",
                "value": 49.2
            },
            {
                "tagId": "pumpRate",
                "value": 2.445906400680542
            },
            {
                "tagId": "oilPressure",
                "value": 34.7
            }
        ]
    },
    {
        "deviceId": "FYYY",
        "timestamp": "2018-01-17T01:18:00Z",
        "series": [
            {
                "tagId": "pumpRate",
                "value": 0.58015072345733643
            },
            {
                "tagId": "oilPressure",
                "value": 22.2
            }
        ]
    }
]
```

參考資料 (索引鍵屬性為 deviceId 和 series.tagId)：

| deviceId | series.tagId | messageId | deviceLocation | type | unit |
| --- | --- | --- | --- | --- | --- |
| FXXX | pumpRate | LINE\_DATA | EU | 流動率 | ft3/s |
| FXXX | oilPressure | LINE\_DATA | EU | 引擎機油壓力 | psi |
| FYYY | pumpRate | LINE\_DATA | US | 流動率 | ft3/s |
| FYYY | oilPressure | LINE\_DATA | US | 引擎機油壓力 | psi |

時間序列見解事件資料表 (壓平合併後)：

| deviceId | series.tagId | messageId | deviceLocation | type | unit | timestamp | series.value |
| --- | --- | --- | --- | --- | --- | --- | --- |
| FXXX | pumpRate | LINE\_DATA | EU | 流動率 | ft3/s | 2018-01-17T01:17:00Z | 1.0172575712203979 |
| FXXX | oilPressure | LINE\_DATA | EU | 引擎機油壓力 | psi | 2018-01-17T01:17:00Z | 34.7 |
| FXXX | pumpRate | LINE\_DATA | EU | 流動率 | ft3/s | 2018-01-17T01:17:00Z | 2.445906400680542 |
| FXXX | oilPressure | LINE\_DATA | EU | 引擎機油壓力 | Psi | 2018-01-17T01:17:00Z | 49.2 |
| FYYY | pumpRate | LINE\_DATA | US | 流動率 | ft3/s | 2018-01-17T01:18:00Z | 0.58015072345733643 |
| FYYY | oilPressure | LINE\_DATA | US | 引擎機油壓力 | psi | 2018-01-17T01:18:00Z | 22.2 |

請注意上述範例中的下列重點，這些重點與第一個範例類似：

- 資料行 **deviceId** 和 **series.tagId** 是車隊中各種裝置和標記的資料行標頭。 使用每個標頭作為其本身的屬性，搭配其他六個資料行，會將查詢限制為總裝置數 594 (S1 環境) 或 794 (S2 環境)。

- 基於第一個範例中所述的原因，已避免不必要的屬性。

- 使用參考資料來減少透過網路傳輸的位元組數目，其做法是為唯一的 **messageId** 和 **deviceLocation** 組合引進 **deviceId**。 針對唯一的 **type** 和 **unit** 組合，則會使用複合索引鍵 **series.tagId**。 複合索引鍵允許使用 **deviceId** 和 **series.tagId** 組來參考四個值：**messageId、deviceLocation、type** 和 **unit**。 此資料會在輸入時與遙測資料聯結，並接著儲存在 TSI 中以供查詢。

- 基於第一個範例中所述的原因，使用兩層的巢狀結構。

### <a name="for-both-scenarios"></a>針對這兩個案例

如果您的屬性具有大量可能值，最好當作單一資料行內的相異值來傳送，而不是為每個值建立新的資料行。 在前兩個範例中：
  - 在第一個範例中，具有數個值的屬性並不多，因此適合將每個屬性當作個別的屬性。 
  - 不過，在第二個範例中，您可以看到量值並未指定為個別屬性，而是指定為通用數列屬性下的值/量值陣列。 新的索引鍵 **tagId** 會送出，以在扁平化資料表中建立新的資料行 **series.tagId**。 使用參考資料可建立新的屬性 **type** 和 **unit**，以避免達到屬性限制。

## <a name="next-steps"></a>後續步驟

若要實際執行這些方針，請參閱 [Azure 時間序列見解查詢語法](/rest/api/time-series-insights/time-series-insights-reference-query-syntax)，以深入了解 TSI 資料存取 REST API 的查詢語法。