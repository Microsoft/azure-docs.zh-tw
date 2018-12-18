---
title: Azure 串流分析中的異常偵測 (預覽)
description: 本文說明如何一起使用 Azure 串流分析和 Azure Machine Learning 來偵測異常行為。
services: stream-analytics
author: dubansal
ms.author: dubansal
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 04/09/2018
ms.openlocfilehash: e7274e4507d901a209ed5832e98ca630feefda4f
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31420090"
---
# <a name="anomaly-detection-in-azure-stream-analytics"></a>Azure 串流分析中的異常偵測

> [!IMPORTANT]
> 這項功能處於預覽狀態，不建議使用於生產工作負載。

**AnomalyDetection** 運算子用來偵測事件串流中不同類型的異常。 例如，長時間可用記憶體緩慢減少可能表示記憶體流失，或範圍內穩定的 Web 服務要求數量可能有急劇的增加或減少。  

AnomalyDetection 運算子可偵測三種類型的異常： 

* **雙向層級變更**：持續增加或減少值層級 (向上和向下)。 此值不同於峰值和谷值，峰值和谷值均為瞬間或短期性變更。  

* **緩慢正向趨勢**：一段時間緩慢增加的趨勢。  

* **緩慢負向趨勢**：一段時間緩慢下降的趨勢。  

使用 AnomalyDetection 運算子時，您必須指定 **Limit Duration** 子句。 此子句可指定在偵測異常時應該考量的時間間隔 (在歷程記錄中從目前事件回推的距離)。 使用  **When**  子句，此運算子可選擇性地僅限於符合特定屬性或條件的事件。 此運算子也可以根據  **Partition by**  子句中指定的索引鍵，選擇性地分開處理事件群組。 每個分割區會獨立進行訓練和預測。 

## <a name="syntax-for-anomalydetection-operator"></a>AnomalyDetection 運算子的語法

`ANOMALYDETECTION(<scalar_expression>) OVER ([PARTITION BY <partition key>] LIMIT DURATION(<unit>, <length>) [WHEN boolean_expression])` 

**使用方式範例**  

`SELECT id, val, ANOMALYDETECTION(val) OVER(PARTITION BY id LIMIT DURATION(hour, 1) WHEN id > 100) FROM input`

### <a name="arguments"></a>引數

* **scalar_expression** - 執行異常偵測的純量運算式。 此參數允許的值包含可傳回單一 (純量) 值的 Float 或 Bigint 資料類型。 不允許萬用字元運算式 **\*** 。 純量運算式不能包含其他分析函式或外部函式。 

* **partition_by_clause** - `PARTITION BY <partition key>` 子句會將學習和訓練劃分為單獨的分割區。 換句話說，`<partition key>` 的每個值會使用單獨的模型，只有具備該值的事件才會用於該模型中的學習與訓練。 例如，下列查詢僅對相同感應器的其他讀數來訓練和評分某一讀數：

  `SELECT sensorId, reading, ANOMALYDETECTION(reading) OVER(PARTITION BY sensorId LIMIT DURATION(hour, 1)) FROM input`

* **limit_duration 子句** `DURATION(<unit>, <length>)` - 指定在偵測異常時應該考量的時間間隔 (在歷程記錄中從目前事件回推的距離)。 如需支援單位及其縮寫的詳細說明，請參閱 [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/datediff-azure-stream-analytics)。 

* **when_clause** - 指定異常偵測運算中考量之事件的布林條件。

### <a name="return-types"></a>傳回型別

AnomalyDetection 運算子會傳回包含所有三個分數做為輸出的記錄。 與不同類型的異常偵測器相關聯的屬性如下：

- BiLevelChangeScore
- SlowPosTrendScore
- SlowNegTrendScore

若要從記錄中擷取個別值，請使用 **GetRecordPropertyValue** 函數。 例如︰

`SELECT id, val FROM input WHERE (GetRecordPropertyValue(ANOMALYDETECTION(val) OVER(LIMIT DURATION(hour, 1)), 'BiLevelChangeScore')) > 3.25` 

當其中一個異常分數超過閾值時，就會檢測到某種類型的異常。 閾值可以是任何大於或等於 0 的浮點數。 閾值是敏感度與信賴之間的取捨。 例如，較低的閾值會讓偵測對於變化更加敏感並產生更多警示，而較高的閾值讓偵測較不敏感而更有信心，但會遮罩某些異常。 要使用的確切閾值視情況而定。 沒有上限，但建議的範圍是 3.25-5。 

範例中顯示的值 3.25 只是建議的起始點。 對您自己的資料集執行作業，並觀察輸出值，直到您觸達可容忍的臨界值，將值進行微調。

## <a name="anomaly-detection-algorithm"></a>異常偵測演算法

* AnomalyDetection 運算子會使用**非監督式學習**方法，該方法不會假設任何類型的事件分佈。 一般而言，在任何時候都會平行維護兩個模型，其中一個模型用於評分，而另一個模型則會在背景中進行訓練。 異常偵測模型會使用目前串流中的資料進行訓練，而不是使用頻外機制進行訓練。 用於訓練的資料數量取決於使用者在 Limit Duration 參數內指定的時間範圍大小 d。 每個模型最終會根據 d 到 2d 的事件進行訓練。 建議每個時間範圍內至少有 50 個事件，以獲得最佳結果。 

* AnomalyDetection 運算子會使用**滑動時間範圍語意**來訓練模型和分數事件。 這表示每個事件都會進行異常評估，且系統會傳回分數。 此分數表示該異常的信賴等級。 

* AnomalyDetection 運算子會提供**可重複性保證**，也就是不論作業輸出開始時間為何，相同的輸入一律會產生相同的分數。 作業輸出開始時間表示作業產生第一個輸出事件的時間。 使用者會將它設定為目前時間、自訂值，或上次輸出時間 (如果作業先前已產生輸出)。 

### <a name="training-the-models"></a>訓練模型 

隨著時間前進，系統會使用不同的資料訓練模型。 為了讓分數產生意義，這有助於了解用以訓練模型的基礎機制。 在這裡，**t<sub>0</sub>** 是**作業輸出開始時間**，而 **d** 是來自 Limit Duration 參數的**時間範圍大小**。 假設時間劃分成**多個大小為 d 的躍點** (從 `01/01/0001 00:00:00` 開始)。 下列步驟用來訓練模型和分數事件：

* 當作業開始時，它會讀取資料 (開始時間為 t<sub>0</sub> – 2d)。  
* 當時間達到下一個躍點時，系統會建立新模型 M1 並開始進行訓練。  
* 當時間向前移到另一個躍點時，系統會建立新模型 M2 並開始進行訓練。  
* 當時間達到 t<sub>0</sub> 時，M1 會變成作用中狀態，而其分數會開始進行輸出。  
* 在下一個躍點，同時會發生三件事：  

  * 不再需要 M1 並予以捨棄。  
  * M2 已經過充分訓練，所以可用於評分。  
  * 系統會建立新模型 M3 並開始在背景中進行訓練。  

* 這個週期會針對每個躍點重複：作用中模型會遭到捨棄、切換到平行模式，並開始在背景中訓練第三個模型。 

用圖表表示，其步驟如下所示： 

![訓練模型](media/stream-analytics-machine-learning-anomaly-detection/training_model.png)

|**模型** | **訓練開始時間** | **開始使用其分數的時間** |
|---------|---------|---------|
|M1     | 11:20   | 11:33   |
|M2     | 11:30   | 11:40   |
|M3     | 11:40   | 11:50   |

* 模型 M1 於上午 11:20 開始訓練，這是在作業於上午 11:13 開始讀取之後的下一個躍點。 在訓練 13 分鐘的資料之後，第一個輸出會於上午 11:33 從 M1 產生。 

* 新模型 M2 也會在上午 11:30 開始訓練，但是直到上午 11:40 (也就是使用 10 分鐘的資料進行訓練之後) 才會使用其分數。 

* M3 遵循會與 M2 相同的模式。 

### <a name="scoring-with-the-models"></a>使用模型進行評分 

在機器學習服務層級，異常偵測演算法會藉由比較每個內送事件與歷程記錄時間範圍中的事件來計算其奇異 (Strangeness) 值。 每種異常類型的奇異計算方式不同。  

讓我們來詳細檢閱奇異計算 (假設有一組包含事件的過去時間範圍)： 

1. **雙向層級變更：** 根據歷程記錄時間範圍，正常的作業範圍計算為 [第 10 個百分位數、第 90 個百分位數]，也就是以第 10 個百分位數值作為下限，以第 90 個百分位數值作為上限。 目前事件的奇異值會計算為：  

   - 0，如果 event_value 處於正常作業範圍內  
   - event_value/90th_percentile，如果 event_value 大於 90th_percentile  
   - 10th_percentile/event_value，如果 event_value 小於 10th_percentile  

2. **緩慢正向趨勢：** 系統會計算歷程記錄時間範圍內事件值的趨勢線，而作業可尋找趨勢線的正向趨勢。 奇異值會計算為：  

   - Slope，如果斜率為正數  
   - 不然為 0 

3. **緩慢負向趨勢：** 系統會計算歷程記錄時間範圍內事件值的趨勢線，而作業可尋找趨勢線的負向趨勢。 奇異值會計算為： 

   - Slope，如果斜率為負數  
   - 不然為 0  

計算內送事件的奇異值後，系統會根據奇異值計算鞅 (Martingale) 值 (如需如何計算鞅值的詳細資訊，請參閱[機器學習服務部落格](https://blogs.technet.microsoft.com/machinelearning/2014/11/05/anomaly-detection-using-machine-learning-to-detect-abnormalities-in-time-series-data/))。 系統會將此鞅值傳回為異常分數。 鞅值會緩慢增加以回應奇怪的值，這可讓偵測器對偶發變更維持強固並減少錯誤警示。 它也有一個實用的屬性： 

可能性 [有 t 存在，以致 M<sub>t</sub> > λ ] < 1/λ，其中 M<sub>t</sub> 是即時 t 的鞅值，而 λ 是實際值。 例如，如果在 M<sub>t</sub>>100 時出現警示，則誤判的可能性會小於 1/100。  

## <a name="guidance-for-using-the-bi-directional-level-change-detector"></a>使用雙向層級變更偵測器的指引 

雙向層級變更偵測器可使用於電源中斷和復原或尖峰時刻流量等案例。不過，其運作方式是一旦使用特定資料訓練模型，則只有在新值高於前一個上限 (向上層級變更案例) 或低於前一個下限 (向下層級變更案例)，另一個層級變更才是異常。 因此，模型不應在其訓練時間範圍內看到新層級 (向上或向下) 範圍中的資料值，因為這些資料值被視為異常。 

使用此偵測器時，應該考量下列幾點： 

1. 當時間序列突然看到數值增加或下滑時，AnomalyDetection 運算子就會偵測到它。 但是偵測恢復正常狀態則需要更詳細的規劃。 如果時間序列在異常前處於穩定狀態，將 AnomalyDetection 運算子的偵測時間範圍設定為異常時間長度的最多一半即可達成。 此案例假設可以事先估計異常的最小持續時間，而且該時間範圍內有足夠的事件可充分訓練模型 (至少 50 個事件)。 

   這顯示於下面使用上限變更的圖 1 和 2 中 (相同的邏輯適用於下限變更)。 在這兩個圖中，波形都屬於異常層級變更。 橙色垂直線表示躍點界限，而且躍點大小與 AnomalyDetection 運算子中指定的偵測時間範圍相同。 綠色線條表示訓練時間範圍的大小。 在圖 1 中，躍點大小與異常持續時間相同。 在圖 2 中，躍點大小是異常持續時間的一半。 在所有情況下，系統都會偵測到向上變更，因為用於評分的模型是使用正常資料進行訓練。 但是，根據雙向層級變更偵測器的運作方式，必須從進行恢復正常狀態評分之模型所用的訓練時間範圍中排除正常值。 在圖 1 中，評分模型的訓練包含一些正常事件，所以無法偵測到恢復正常狀態。 但在圖 2 中，訓練只包含異常部分，以確保偵測到恢復正常狀態。 基於相同的理由，任何小於一半的項目也適用，然而任何較大的項目終將包含少數正常事件。 

   ![時間範圍大小等於異常時間長度的 AD](media/stream-analytics-machine-learning-anomaly-detection/windowsize_equal_anomaly_length.png)

   ![時間範圍大小等於異常時間長度一半的 AD](media/stream-analytics-machine-learning-anomaly-detection/windowsize_equal_half_anomaly_length.png)

2. 在無法預測異常時間長度的情況下，此偵測器會全力運作。 不過，選擇較窄的時間範圍會限制訓練資料，而增加偵測到恢復正常狀態的可能性。 

3. 在下列案例中，並未偵測到時間較長的異常，因為訓練時間範圍已經包含值同樣高的異常。 

   ![大小相同的異常](media/stream-analytics-machine-learning-anomaly-detection/anomalies_with_same_length.png)

## <a name="example-query-to-detect-anomalies"></a>用以偵測異常行為的範例查詢 

如果偵測到異常，下列查詢可用於輸出警示。
當輸入資料流不統一時，彙總步驟可協助將其轉換成統一的時間序列。 此範例會使用 AVG，但特定類型的彙總視使用者情況而定。 此外，當時間序列具有大於彙總時間範圍的間隙時，時間序列中沒有任何事件會觸發異常偵測 (根據滑動時間範圍語意)。 因此，當下一個事件來臨，就會打破一致性的假設。 在這類情況下，應填滿時間序列中的間隙。 其中一個方法是在每個跳動視窗中採用最後一個事件，如下所示。

```sql
    WITH AggregationStep AS 
    (
         SELECT
               System.Timestamp as tumblingWindowEnd,

               AVG(value) as avgValue

         FROM input
         GROUP BY TumblingWindow(second, 5)
    ),

    FillInMissingValuesStep AS
    (
          SELECT
                System.Timestamp AS hoppingWindowEnd,

                TopOne() OVER (ORDER BY tumblingWindowEnd DESC) AS lastEvent

         FROM AggregationStep
         GROUP BY HOPPINGWINDOW(second, 300, 5)

    ),

    AnomalyDetectionStep AS
    (

          SELECT
                hoppingWindowEnd,
                lastEvent.tumblingWindowEnd as lastTumblingWindowEnd,
                lastEvent.avgValue as lastEventAvgValue,
                system.timestamp as anomalyDetectionStepTimestamp,

                ANOMALYDETECTION(lastEvent.avgValue) OVER (LIMIT DURATION(hour, 1)) as
                scores

          FROM FillInMissingValuesStep
    )

    SELECT
          alert = 1,
          hoppingWindowEnd,
          lastTumblingWindowEnd,
          lastEventAvgValue,
          anomalyDetectionStepTimestamp,
          scores

    INTO output

    FROM AnomalyDetectionStep

    WHERE

        CAST(GetRecordPropertyValue(scores, 'BiLevelChangeScore') as float) >= 3.25

        OR CAST(GetRecordPropertyValue(scores, 'SlowPosTrendScore') as float) >=
        3.25

       OR CAST(GetRecordPropertyValue(scores, 'SlowNegTrendScore') as float) >=
       3.25
```

## <a name="performance-guidance"></a>效能指南

* 將六個串流單位用於處理作業。 
* 傳送事件時至少間隔 1 秒。
* 使用 AnomalyDetection 運算子的非分割查詢可能在平均計算延遲大約 25 毫秒的情況下產生結果。
* 隨著計算次數的增加，資料分割查詢所經歷的延遲與分割數略有不同。 不過，延遲與所有分割中可比較事件總數的非資料分割情況大致相同。
* 讀取非即時資料時，會快速內嵌大量資料。 目前處理此資料比較慢。 這類情況下的延遲是隨著時間範圍內的資料點數目 (而非時間範圍大小或事件間隔) 呈線性增加。 若要降低非即時情況的延遲，請考慮使用較小的視窗大小。 或者，請考慮從目前時間開始您的作業。 非資料分割查詢中延遲的一些範例： 
    - 偵測視窗中的 60 個資料點可能會導致 3 MBps 輸送量出現 10 秒的延遲時間。 
    - 在 600 個資料點時，0.4 MBps 的輸送量可能會達到大約 80 秒的延遲時間。

## <a name="limitations-of-the-anomalydetection-operator"></a>AnomalyDetection 運算子的限制 

* 此運算子目前不支援峰值和谷值偵測。 峰值和谷值均為時間序列中的瞬間或短期性變更。 

* 此運算子目前不會處理季節性模式。 季節性模式是資料中的重複模式，例如週末期間不同的網路流量行為或黑色星期五期間不同的購物趨勢，這不是異常，而是預期的行為模式。 

* 此運算子預期輸入時間序列是統一的。 事件資料流可藉由輪轉彙總或跳動視窗來進行統一。 在事件之間的間隙總是小於彙總視窗的情況下，輪轉視窗足以讓時間序列一致。 當間隙可以更大時，可以透過使用跳動視窗重複最後一個值來填滿間隙。 

## <a name="references"></a>參考

* [異常偵測 – 使用機器學習服務來偵測時間序列資料中的異常](https://blogs.technet.microsoft.com/machinelearning/2014/11/05/anomaly-detection-using-machine-learning-to-detect-abnormalities-in-time-series-data/)
* [機器學習服務異常偵測 API](https://docs.microsoft.com/en-gb/azure/machine-learning/machine-learning-apps-anomaly-detection-api)
* [時間序列異常偵測](https://msdn.microsoft.com/library/azure/mt775197.aspx)

## <a name="next-steps"></a>後續步驟

* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [開始使用 Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

