---
title: 使用 Azure 串流分析中的查詢平行化和調整作業
description: 本文說明如何透過設定輸入資料分割、微調查詢定義，及設定工作串流處理單元來調整串流分析工作。
services: stream-analytics
author: JSeb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/07/2018
ms.openlocfilehash: 44a7c0721d8a0683162d2219bff0e4a4ecb117e6
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33777622"
---
# <a name="leverage-query-parallelization-in-azure-stream-analytics"></a>利用 Azure 串流分析中的查詢平行化作業
本文會示範如何利用 Azure 串流分析中的平行化作業。 您可以了解如何透過設定輸入資料分割並調整分析查詢定義來調整串流分析工作。
先決條件是，您必須熟悉[了解及調整串流處理單位](stream-analytics-streaming-unit-consumption.md)中所述的串流處理單位概念。

## <a name="what-are-the-parts-of-a-stream-analytics-job"></a>串流分析工作由哪些部分所組成？
串流分析工作的定義包含輸入、查詢及輸出。 輸入是指作業讀取資料流的來源。 查詢是用來轉換資料輸入資料流，而輸出是作業傳送作業結果的目的地。  

一個工作至少需要一個輸入來源來進行資料串流。 資料流輸入來源可以儲存在 Azure 事件中樞或 Azure Blob 儲存體中。 如需詳細資訊，請參閱 [Azure 串流分析簡介](stream-analytics-introduction.md)和[開始使用 Azure 串流分析](stream-analytics-real-time-fraud-detection.md)。

## <a name="partitions-in-sources-and-sinks"></a>來源與接收的資料分割
調整串流分析作業需要利用輸入或輸出中的分割區。 資料分割可讓您根據分割索引鍵，將資料分成子集。 取用資料的流程 (例如串流分析作業) 可以平行取用和寫入不同的分割區，這樣可以增加輸送量。 

### <a name="inputs"></a>輸入
所有 Azure 串流分析輸入都可以利用資料分割：
-   事件中樞 (需要明確地使用 PARTITION BY 關鍵字設定分割區索引鍵)
-   IoT 中樞 (需要明確地使用 PARTITION BY 關鍵字設定分割區索引鍵)
-   Blob 儲存體

### <a name="outputs"></a>輸出

使用串流分析時，您可以利用輸出中的資料分割：
-   Azure Data Lake 儲存體
-   Azure Functions
-   Azure 資料表
-   Blob 儲存體 (可明確地設定資料分割索引鍵)
-   CosmosDB (需要明確地設定資料分割索引鍵)
-   事件中樞 (需要明確地設定資料分割索引鍵)
-   IoT 中樞 (需要明確地設定資料分割索引鍵)
-   服務匯流排

PowerBI、SQL 和 SQL 資料倉儲輸出不支援資料分割。 不過，您仍然可以分割輸入，如[本節](#multi-step-query-with-different-partition-by-values)中所述 

如需分割區的詳細資訊，請參閱下列文章：

* [事件中樞功能概觀](../event-hubs/event-hubs-features.md#partitions)
* [資料分割](https://docs.microsoft.com/azure/architecture/best-practices/data-partitioning#partitioning-azure-blob-storage)


## <a name="embarrassingly-parallel-jobs"></a>窘迫平行作業
「窘迫平行」作業是我們在 Azure 串流分析中調整性最高的情節。 它將對於查詢執行個體之輸入的某個資料分割，連接到輸出的某個資料分割。 此平行處理原則具有下列需求︰

1. 如果查詢邏輯相依於同一個查詢執行個體所處理的相同索引鍵，則您必須確保事件會傳送至輸入的相同分割區。 對於事件中樞或 IoT 中樞，這表示事件資料必須設定 **PartitionKey** 值。 或者，您可以使用分割的傳送者。 對於 Blob 儲存體，這表示事件會傳送至相同的磁碟分割資料夾。 如果查詢邏輯並不需要同一個查詢執行個體所處理的相同索引鍵，您可以忽略這項需求。 簡單的選取-投影-篩選查詢，即為此邏輯的一個例子。  

2. 當資料放在輸入端時，您必須確保查詢已分割。 所有步驟中都必須使用 **PARTITION BY**。 您可以使用多個步驟，但全部都必須依相同的索引鍵來分割。 目前，資料分割索引鍵必須設定為 **PartitionId**，作業才能完全平行。  

3. 我們大部分的輸出都可以利用資料分割，不過，如果您使用不支援資料分割的輸出類型，您的作業將無法進行平行處理。 如需詳細資訊，請參閱[輸出](#outputs)一節。

4. 輸入分割區的數目必須等於輸出分割區的數目。 Blob 儲存體輸出可支援資料分割，並繼承上游查詢的資料分割配置。 當針對 Blob 儲存體指定資料分割索引鍵時，資料會依每個輸入分割區進行分割，因此結果仍然是完全平行。 以下是允許完全平行作業的分割區值範例：

   * 8 個事件中樞輸入分割區和 8 個事件中樞輸出分割區
   * 8 個事件中樞輸入分割區和 blob 儲存體輸出
   * 8 個事件中樞輸入分割區和 Blob 儲存體輸出，依含有任意基數的自訂欄位分割
   * 8 個 blob 儲存體輸入分割區和 blob 儲存體輸出
   * 8 個 blob 儲存體輸入分割區和 8 個事件中樞輸出分割區

以下幾節討論一些窘迫平行的範例情節。

### <a name="simple-query"></a>簡單查詢

* 輸入：具有 8 個分割區的事件中樞
* 輸出：具有 8 個分割區的事件中樞

查詢：

    SELECT TollBoothId
    FROM Input1 Partition By PartitionId
    WHERE TollBoothId > 100

此查詢是簡單的篩選。 因此，我們不需要擔心將傳送到事件中樞的輸入分割。 請注意，此查詢包含 **PARTITION BY PartitionId**，因此滿足稍早的需求 #2。 對於輸出，我們必須將作業中的事件中樞輸出設定為讓資料分割索引鍵設為 **PartitionId**。 最後一項檢查是確保輸入分割區數目等於輸出分割區數目。

### <a name="query-with-a-grouping-key"></a>利用群組索引鍵的查詢

* 輸入：具有 8 個分割區的事件中樞
* 輸出：Blob 儲存體

查詢：

    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId

此查詢含有群組索引鍵。 因此，分組在一起的事件必須傳送至相同的事件中樞資料分割。 在此範例中我們是依 TollBoothID 分組，因此，我們應該確保在事件傳送至事件中樞時，TollBoothID 會當作資料分割索引鍵使用。 然後在 ASA 中，我們可以使用 **PARTITION BY PartitionId** 從此資料分割配置繼承，並啟用完整的平行化作業。 因為輸出是 blob 儲存體，所以我們不需要擔心設定分割區索引鍵值，以滿足需求 #4。

## <a name="example-of-scenarios-that-are-not-embarrassingly-parallel"></a>*非*窘迫平行的情節範例

在上一節，我們示範一些窘迫平行情節。 在本節中，我們要討論的情節不符合成為窘迫平行的所有需求。 

### <a name="mismatched-partition-count"></a>不相符的分割區計數
* 輸入：具有 8 個分割區的事件中樞
* 輸出：具有 32 個分割區的事件中樞

在此案例中，查詢並不重要。 如果輸入資料分割計數不符合輸出資料分割計數，則拓撲不是窘迫平行。不過，我們仍然可以獲得一定程度的平行化。

### <a name="query-using-non-partitioned-output"></a>使用非資料分割的輸出查詢
* 輸入：具有 8 個分割區的事件中樞
* 輸出：PowerBI

PowerBI 輸出目前不支援資料分割。 因此，此情節不是窘迫平行。

### <a name="multi-step-query-with-different-partition-by-values"></a>具有不同 PARTITION BY 值的多重步驟查詢
* 輸入：具有 8 個分割區的事件中樞
* 輸出：具有 8 個分割區的事件中樞

查詢：

    WITH Step1 AS (
    SELECT COUNT(*) AS Count, TollBoothId, PartitionId
    FROM Input1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
    )

    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1 Partition By TollBoothId
    GROUP BY TumblingWindow(minute, 3), TollBoothId

如您所見，第二個步驟把[TollBoothId]  當做資料分割索引鍵來使用。 此步驟和第一個步驟不同，因此需要變換一下。 

上述範例示範一些符合 (或不符合) 窘迫平行拓撲的串流分析作業。 如果符合，則可能有最大調整幅度。 對於不符合其中一個設定檔的作業，則提供未來更新時的調整指引。 現在，請在下列各節中使用一般指引。

## <a name="calculate-the-maximum-streaming-units-of-a-job"></a>計算工作的最大串流處理單元
資料流分析工作可使用的串流處理單元總數，取決於為工作定義之查詢中的步驟數目，和每個步驟的資料分割數目。

### <a name="steps-in-a-query"></a>查詢中的步驟
一個查詢可以有一或多個步驟。 每個步驟都是使用 **WITH** 關鍵字定義的子查詢。 在 **WITH** 關鍵字外的查詢 (只有一個查詢) 也視為一個步驟，例如下列查詢中的 **SELECT** 陳述式：

查詢：

    WITH Step1 AS (
        SELECT COUNT(*) AS Count, TollBoothId
        FROM Input1 Partition By PartitionId
        GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
    )
    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1
    GROUP BY TumblingWindow(minute,3), TollBoothId

此查詢有兩個步驟。

> [!NOTE]
> 本文稍後會詳細討論此查詢。
>  

### <a name="partition-a-step"></a>分割步驟
要分割步驟必須符合下列條件：

* 必須分割輸入來源。 
* 查詢的 **SELECT** 陳述式必須讀取某個已分割的輸入來源。
* 步驟內的查詢必須有 **PARTITION BY** 關鍵字。

分割查詢時，將會在個別的分割區群組中處理和彙總輸入事件，然後為每個群組產生輸出事件。 如果需要合併的彙總，您必須建立第二個非分割步驟來彙總。

### <a name="calculate-the-max-streaming-units-for-a-job"></a>計算工作的最大串流處理單元
所有非分割步驟合起來，可將串流分析作業調整為最多 6 個串流單元 (SU)。 此外，您可以在分割的步驟中，為每個資料分割新增 6 個 SU。
您可以在下表中看到部分**範例**。

| 查詢                                               | 作業的最多 SU |
| --------------------------------------------------- | ------------------- |
| <ul><li>查詢包含一個步驟。</li><li>步驟未分割。</li></ul> | 6 |
| <ul><li>輸入資料流會分割為 16 個。</li><li>查詢包含一個步驟。</li><li>步驟進行分割。</li></ul> | 96 (6 * 16 個資料分割) |
| <ul><li>查詢包含兩個步驟。</li><li>兩個步驟都不會分割。</li></ul> | 6 |
| <ul><li>輸入資料流分割時會除以 3。</li><li>查詢包含兩個步驟。 輸入步驟會分割，而第二個步驟則否。</li><li><strong>SELECT</strong> 陳述式會讀取分割的輸入。</li></ul> | 24 (18 個用於分割的步驟 + 6 個用於非分割的步驟) |

### <a name="examples-of-scaling"></a>調整的範例

下列查詢會計算三分鐘內通過收費站 (有三個收費亭) 的車流量。 此查詢可以調整為最多 6 個 SU。

    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId

若要讓查詢使用更多 SU，輸入資料流和查詢都必須分割。 因為資料流分割區設為 3，以下修改的查詢可以調整為最多 18 個 SU：

    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId

分割查詢時將會處理輸入事件並將其彙總在個別的資料分割群組中。 也會為每個群組產生輸出事件。 當 **GROUP BY** 欄位不是輸入資料流中的分割區索引鍵時，資料分割可能會導致某些非預期的結果。 例如，上一個查詢中的 **TollBoothId** 欄位不是 **Input1** 的分割區索引鍵。 結果是收費亭 #1 的資料可能分散在多個分割區。

串流分析會個別處理每個 **Input1** 分割區。 因此，在相同的輪轉視窗中，相同收費亭的汽車計數會建立多筆記錄。 如果無法變更輸入分割區索引鍵，可藉由新增非分割步驟彙總資料分割的值來解決此問題，如下列範例所示：

    WITH Step1 AS (
        SELECT COUNT(*) AS Count, TollBoothId
        FROM Input1 Partition By PartitionId
        GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
    )

    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1
    GROUP BY TumblingWindow(minute, 3), TollBoothId

此查詢可以調整為 24 個 SU。

> [!NOTE]
> 如果您要聯結兩個資料流，請確定以您用來建立聯結的資料行的分割區索引鍵來分割資料流。 也請確定兩個資料流中的分割區數目相同。
> 
> 





## <a name="get-help"></a>取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)。

## <a name="next-steps"></a>後續步驟
* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [開始使用 Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

<!--Image references-->

[img.stream.analytics.monitor.job]: ./media/stream-analytics-scale-jobs/StreamAnalytics.job.monitor-NewPortal.png
[img.stream.analytics.configure.scale]: ./media/stream-analytics-scale-jobs/StreamAnalytics.configure.scale.png
[img.stream.analytics.perfgraph]: ./media/stream-analytics-scale-jobs/perf.png
[img.stream.analytics.streaming.units.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsStreamingUnitsExample.jpg
[img.stream.analytics.preview.portal.settings.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsPreviewPortalJobSettings-NewPortal.png   

<!--Link references-->

[microsoft.support]: http://support.microsoft.com
[azure.event.hubs.developer.guide]: http://msdn.microsoft.com/library/azure/dn789972.aspx

[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: http://go.microsoft.com/fwlink/?LinkId=517301

