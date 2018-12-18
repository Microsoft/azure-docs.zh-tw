---
title: 使用 Data Factory 進行排程和執行 | Microsoft Docs
description: 了解 Azure Data Factory 應用程式模型的排程和執行層面。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 088a83df-4d1b-4ac1-afb3-0787a9bd1ca5
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: bd8b682e073e86bb824d31d6ebab20a80f807730
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37054597"
---
# <a name="data-factory-scheduling-and-execution"></a>Data Factory 排程和執行
> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[管線執行和觸發程序](../concepts-pipeline-execution-triggers.md)一文。

本文說明 Azure Data Factory 應用程式模型的排程和執行層面。 本文假設您已了解 Data Factory 應用程式模型的基本概念，包括活動、管線、連結的服務和資料集。 請看下列文章，了解 Azure Data Factory 的基本概念：

* [Data Factory 服務簡介](data-factory-introduction.md)
* [管線](data-factory-create-pipelines.md)
* [資料集](data-factory-create-datasets.md) 

## <a name="start-and-end-times-of-pipeline"></a>管線的開始和結束時間
管線僅在其「開始」時間與「結束」時間之間有作用。 在開始時間之前或結束時間之後就不會執行。 如果管線已暫停，不論其開始和結束時間為何，都不會執行。 若要執行管線，則不該將它暫停。 您可以在管線定義中找到這些設定 (start、end、paused)： 

```json
"start": "2017-04-01T08:00:00Z",
"end": "2017-04-01T11:00:00Z"
"isPaused": false
```

如需有關這些屬性的詳細資訊，請參閱[建立管線](data-factory-create-pipelines.md)一文。 


## <a name="specify-schedule-for-an-activity"></a>為活動指定排程
執行的不是管線。 在管線的整體內容中，執行的是管線中的活動。 您可以使用活動 JSON 的 **scheduler** 區段，來為活動指定週期性排程。 例如，您可以排定讓活動每小時執行一次，如下：  

```json
"scheduler": {
    "frequency": "Hour",
    "interval": 1
},
```

如下圖所示，為活動指定排程會在管線的開始和結束時間內，建立一系列輪轉時段。 輪轉時段是一系列大小固定、非重疊的連續時間間隔。 活動的這些邏輯輪轉時段稱為「活動時段」。

![活動排程器範例](media/data-factory-scheduling-and-execution/scheduler-example.png)

活動的 **scheduler** 屬性是選擇性的。 如果您指定此屬性，它就必須符合您在活動輸出資料集的定義中指定的頻率。 目前，驅動排程的是輸出資料集。 因此，即使活動不會產生任何輸出，您也必須指定輸出資料集。 

## <a name="specify-schedule-for-a-dataset"></a>為資料集指定排程
Data Factory 管線中的一個活動可以接受零個或多個輸入「資料集」，並且會產生一個或多個輸出資料集。 針對活動，您可以在資料集定義中使用 **availability** 區段，來指定提供輸入資料或產生輸出資料的頻率。 

**availability** 區段中的 **frequency** 會指定時間單位。 允許的 frequency 值為：Minute、Hour、Day、Week 及 Month。 availability 區段中的 **interval** 屬性會指定 frequency 的倍數。 例如：如果將輸出資料集的 frequency 設定為 Day，並將 interval 是設定為 1，就會每天產生輸出資料。 如果您將 frequency 指定為 minute，建議您將 interval 設定為不小於 15。 

在下列範例中，會每小時提供輸入資料，並每小時產生輸出資料 (`"frequency": "Hour", "interval": 1`)。 

**輸入資料集：** 

```json
{
    "name": "AzureSqlInput",
    "properties": {
        "published": false,
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
```


**輸出資料集**

```json
{
    "name": "AzureBlobOutput",
    "properties": {
        "published": false,
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "mypath/{Year}/{Month}/{Day}/{Hour}",
            "format": {
                "type": "TextFormat"
            },
            "partitionedBy": [
                { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
                { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
                { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
                { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "HH" }}
            ]
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

目前，**是由輸出資料集驅動排程**。 換句話說，會使用為輸出資料集指定的排程在執行階段執行活動。 因此，即使活動不會產生任何輸出，您也必須指定輸出資料集。 如果活動沒有任何輸入，您可以略過建立輸入資料集。 

在下列管線定義中，會使用 **scheduler** 屬性來為活動指定排程。 這是選用屬性。 目前，活動的排程必須與為輸出資料集指定的排程相符。
 
```json
{
    "name": "SamplePipeline",
    "properties": {
        "description": "copy activity",
        "activities": [
            {
                "type": "Copy",
                "name": "AzureSQLtoBlob",
                "description": "copy activity",
                "typeProperties": {
                    "source": {
                        "type": "SqlSource",
                        "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 100000,
                        "writeBatchTimeout": "00:05:00"
                    }
                },
                "inputs": [
                    {
                        "name": "AzureSQLInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                }
            }
        ],
        "start": "2017-04-01T08:00:00Z",
        "end": "2017-04-01T11:00:00Z"
    }
}
```

在此範例中，活動會在管線的開始與結束時間之間每小時執行一次。 針對三小時的時段 (上午 8 點 - 上午 9 點、上午 9 點 - 上午 10 點，以及上午 10 點 - 上午 11 點) 會每小時產生一次輸出資料。 

活動執行所取用或產生的每個資料單位稱為「資料配量」。 下表顯示具有一個輸入資料集和一個輸出資料集的活動範例： 

![可用性排程器](./media/data-factory-scheduling-and-execution/availability-scheduler.png)

上圖顯示輸入和輸出資料集的每小時資料配量。 圖中顯示 3 個已經可供處理的輸入配量。 10-11 AM 活動正在進行中，會產生 10-11 AM 輸出配量。 

您可以使用 [SliceStart](data-factory-functions-variables.md#data-factory-system-variables) 和 [SliceEnd](data-factory-functions-variables.md#data-factory-system-variables) 變數，來存取與資料集 JSON 中目前配量關聯的時間間隔。 同樣地，您可以使用 WindowStart 和 WindowEnd，來存取與活動時段關聯的時間間隔。 活動的排程必須與活動之輸出資料集的排程相符。 因此，SliceStart 和 SliceEnd 值會分別與 WindowStart 和 WindowEnd 值相同。 如需有關這些變數的詳細資訊，請參閱 [Data Factory 函式與系統變數](data-factory-functions-variables.md#data-factory-system-variables)文章。  

您可以在活動 JSON 中針對不同用途使用這些變數。 例如，您可以使用它們從代表時間序列資料 (例如：上午 8 點到上午 9 點) 的輸入和輸出資料集選取資料。 此範例也使用 **WindowStart** 和 **WindowEnd** 來選取活動執行的相關資料，並將它複製到具有適當 **folderPath** 的 Blob。 **folderPath** 會參數化為讓每小時具有個別的資料夾。  

在上述範例中，為輸入和輸出資料集指定的排程是相同的 (每小時)。 如果活動的輸入資料集會以不同的頻率提供 (假設是每隔 15 分鐘)，產生此輸出資料集的活動仍然會一小時執行一次，因為驅動活動排程的是輸出資料集。 如需詳細資訊，請參閱[使用不同的頻率模型化資料集](#model-datasets-with-different-frequencies)。

## <a name="dataset-availability-and-policies"></a>資料集可用性和原則
您已了解資料集定義之 availability 區段中 frequency 和 interval 屬性的使用方式。 還有一些影響活動的排程和執行的其他屬性。 

### <a name="dataset-availability"></a>資料集可用性 
下表描述您在 **availability** 區段中可使用的屬性：

| 屬性 | 說明 | 必要 | 預設值 |
| --- | --- | --- | --- |
| frequency |指定資料集配量生產的時間單位。<br/><br/><b>支援的頻率</b>：Minute、Hour、Day、Week、Month |yes |NA |
| interval |指定頻率的倍數<br/><br/>「頻率 x 間隔」會決定產生配量的頻率。<br/><br/>如果您需要將資料集以每小時為單位來切割，請將 <b>Frequency</b> 設定為 <b>Hour</b>，將 <b>interval</b> 設定為 <b>1</b>。<br/><br/><b>注意</b>：如果您將 Frequency 指定為 Minute，建議您將 interval 設定為不小於 15 |yes |NA |
| style |指定是否應該在間隔開始/結束時產生配量。<ul><li>StartOfInterval</li><li>EndOfInterval</li></ul><br/><br/>如果 Frequency 設為 Month，style 設為 EndOfInterval，則會在當月最後一天產生配量。 如果 style 設為 StartOfInterval，則會在每月的第一天產生配量。<br/><br/>如果 Frequency 設為 Day，style 設為 EndOfInterval，則會在當天最後一個小時產生配量。<br/><br/>如果 Frequency 設為 Hour，style 設為 EndOfInterval，則會在每小時結束時產生配量。 例如，若為下午 1 – 2 點期間的配量，此配量會在下午 2 點產生。 |否 |EndOfInterval |
| anchorDateTime |定義排程器用來計算資料集配量界限的時間絕對位置。 <br/><br/><b>注意</b>：如果 AnchorDateTime 有比頻率更細微的日期部分，則系統會忽略那些更細微的部分。 <br/><br/>例如，如果 <b>interval</b> 為 <b>hourly</b> (frequency: hour 且 interval: 1) 而且 <b>AnchorDateTime</b> 包含<b>分鐘和秒鐘</b>，則會忽略 AnchorDateTime 的<b>分鐘和秒鐘</b>部分。 |否 |01/01/0001 |
| Offset |所有資料集配量的開始和結束移位所依據的時間範圍。 <br/><br/><b>注意</b>︰如果同時指定 anchorDateTime 和 offset，結果會是合併的位移。 |否 |NA |

### <a name="offset-example"></a>位移範例
根據預設，每日 (`"frequency": "Day", "interval": 1`) 配量的開始時間是 UTC 時間上午 12 點 (午夜)。 如果您希望將開始時間改為 UTC 時間上午 6 點，請依照下列程式碼片段所示設定位移： 

```json
"availability":
{
    "frequency": "Day",
    "interval": 1,
    "offset": "06:00:00"
}
```
### <a name="anchordatetime-example"></a>anchorDateTime 範例
在下列範例中，會每隔 23 小時產生一次資料集。 第一個配量會在 anchorDateTime 所指定的時間開始，這是設定成 `2017-04-19T08:00:00` (UTC 時間)。

```json
"availability":    
{    
    "frequency": "Hour",        
    "interval": 23,    
    "anchorDateTime":"2017-04-19T08:00:00"    
}
```

### <a name="offsetstyle-example"></a>位移/樣式範例
下列資料集是每月資料集，會在每月 3 號的上午 8:00 (`3.08:00:00`) 產生：

```json
"availability": {
    "frequency": "Month",
    "interval": 1,
    "offset": "3.08:00:00", 
    "style": "StartOfInterval"
}
```

### <a name="dataset-policy"></a>資料集原則
資料集可以具有定義的驗證原則，指定配量執行所產生的資料在供取用之前如何驗證。 在這種情況下，於配量完成執行後，輸出配量狀態就會變更為**等候**，子狀態為**驗證**。 配量通過驗證之後，配量狀態會變更為 **就緒**。 如果已產生資料配量但是未通過驗證，將不會處理相依於此配量的下游配量活動執行。 [監視和管理管線](data-factory-monitor-manage-pipelines.md) 涵蓋 Data Factory 中資料配量的各種狀態。

資料集中的 **policy** 區段定義資料集配量必須符合的準則或條件。 下表說明您可以在 **policy** 區段中使用的屬性：

| 原則名稱 | 說明 | 適用於 | 必要 | 預設值 |
| --- | --- | --- | --- | --- |
| minimumSizeMB | 驗證 **Azure Blob** 中的資料是否符合最小的大小需求 (以 MB 為單位)。 |Azure Blob |否 |NA |
| minimumRows | 驗證 **Azure SQL Database** 或 **Azure 資料表**中的資料是否包含最小的資料列數。 |<ul><li>連接字串</li><li>Azure 資料表</li></ul> |否 |NA |

#### <a name="examples"></a>範例
**minimumSizeMB：**

```json
"policy":

{
    "validation":
    {
        "minimumSizeMB": 10.0
    }
}
```

**minimumRows**

```json
"policy":
{
    "validation":
    {
        "minimumRows": 100
    }
}
```

如需有關這些屬性及範例的詳細資訊，請參閱[建立資料集](data-factory-create-datasets.md)一文。 

## <a name="activity-policies"></a>活動原則
原則會影響活動的執行階段行為，特別是在處理資料表配量的時候。 下表提供詳細資料。

| 屬性 | 允許的值 | 預設值 | 說明 |
| --- | --- | --- | --- |
| 並行 |整數  <br/><br/>最大值：10 |1 |活動的並行執行數目。<br/><br/>它可決定不同配量上可以發生的平行活動執行數目。 例如，如果活動需要處理大量可用的資料，具有較大的並行值會加快資料處理。 |
| executionPriorityOrder |NewestFirst<br/><br/>OldestFirst |OldestFirst |決定正在處理之資料配量的順序。<br/><br/>例如，如果您有 2 個配量 (一個發生在下午 4 點，另一個發生在下午 5 點)，而兩者都暫停執行。 如果您將 executionPriorityOrder 設為 NewestFirst，則會先處理下午 5 點的配量。 同樣地，如果您將 executionPriorityOrder 設為 OldestFIrst，則會處理下午 4 點的配量。 |
| retry |整數 <br/><br/>最大值可以是 10 |0 |在配量的資料處理標示為 [失敗] 前的重試次數。 資料配量的活動執行會一直重試，直到指定的重試計數為止。 在失敗後會儘速完成重試。 |
| timeout |時間範圍 |00:00:00 |活動的逾時。 範例︰00:10:00 (意指逾時 10 分鐘)<br/><br/>如果您未指定值 (或值為 0)，代表無限逾時。<br/><br/>如果配量的資料處理時間超過逾時值，該活動會遭到取消，且系統會嘗試重試處理。 重試次數取決於 retry 屬性。 若發生逾時，狀態會設為 TimedOut。 |
| delay |時間範圍 |00:00:00 |指定配量之資料處理開始之前的延遲。<br/><br/>資料配量的活動執行會在 Delay 超出預期執行時間後開始。<br/><br/>範例︰00:10:00 (意指延遲 10 分鐘) |
| longRetry |整數 <br/><br/>最大值：10 |1 |配量執行失敗之前的長時間重試嘗試次數。<br/><br/>多個 longRetry 嘗試之間以 longRetryInterval 隔開。 所以如果您需要指定重試嘗試之間的時間，請使用 longRetry。 如果您指定 Retry 和 longRetry 兩者，每個 longRetry 嘗試都包含 Retry 嘗試，且最大嘗試次數是 Retry * longRetry。<br/><br/>例如，如果活動原則的設定如下︰<br/>Retry：3<br/>longRetry：2<br/>longRetryInterval：01:00:00<br/><br/>假設只有一個要執行的配量 (狀態是 Waiting)，且活動執行每次都失敗。 一開始會有 3 次連續執行嘗試。 在每次嘗試之後，配量狀態會是 Retry。 在前 3 次嘗試結束之後，配量狀態會是 LongRetry。<br/><br/>一個小時 (也就是 longRetryInteval 的值) 之後，會有另一組 3 次連續執行嘗試。 在那之後，配量狀態會是 Failed，不會再嘗試重試。 因此全部已進行 6 次嘗試。<br/><br/>如果任何執行成功，配量狀態會是 Ready 且不會再嘗試重試。<br/><br/>longRetry 可能用於下列情況：相依資料達到不具決定性的次數，或進行資料處理的整體環境很脆弱。 在這類情況下逐一進行重試並沒有幫助，而在一段時間後進行重試則會導致所要的結果。<br/><br/>提醒：請勿設定較大的 longRetry 或 longRetryInterval 值。 較大的值通常表示其他系統問題。 |
| longRetryInterval |時間範圍 |00:00:00 |長時間重試嘗試之間的延遲 |

如需詳細資訊，請參閱[管線](data-factory-create-pipelines.md)一文。 

## <a name="parallel-processing-of-data-slices"></a>資料配量的平行處理
您可以將管線的開始日期設定為過去的日期。 當您這麼做時，Data Factory 會自動計算 (回補) 過去的所有資料配量，並開始處理它們。 例如：如果您建立一個開始日期為 2017-04-01 的管線，而目前的日期是 2017-04-10。 當輸出資料集的頻率是每天時，Data Factory 會立即開始處理從 2017-04-01 到 2017-04-09 的所有配量，因為開始日期是過去的日期。 從 2017-04-10 起的配量尚未處理，因為 availability 區段中 style 屬性的值預設為 EndOfInterval。 最舊的配量會最先處理，因為 executionPriorityOrder 的預設值為 OldestFirst。 如需 style 屬性的相關描述，請參閱[資料集可用性](#dataset-availability)一節。 如需 executionPriorityOrder 區段的相關描述，請參閱[活動原則](#activity-policies)一節。 

您可以透過設定活動 JSON 之 **policy** 區段中的 **concurrency** 屬性，設定以平行方式處理回補的資料配量。 此屬性可決定不同配量上可以發生的平行活動執行數目。 concurrency 屬性的預設值是 1。 因此，預設會一次處理一個配量。 最大值為 10。 當管線需要處理一組大量的可用資料時，concurrency 的值越大，資料處理的速度就越快。 

## <a name="rerun-a-failed-data-slice"></a>重新執行失敗的資料配量
處理資料配量時，如果發生錯誤，您可以使用 Azure 入口網站刀鋒視窗或「監視與管理」應用程式來找出配量處理失敗的原因。 如需詳細資訊，請參閱[使用 Azure 入口網站刀鋒視窗監視和管理管線](data-factory-monitor-manage-pipelines.md)或[監視和管理應用程式](data-factory-monitor-manage-app.md)。

請思考一下會顯示兩個活動的下列範例。 Activity1 和 Activity 2。 Activity1 會取用 Dataset1 配量並產生 Dataset2 配量，而此配量會由 Activity2 取用來作為輸入以產生 Final Dataset (最終資料集)。

![失敗的配量](./media/data-factory-scheduling-and-execution/failed-slice.png)

圖中顯示 3 個最近的配量當中有失敗，針對 Dataset2 產生 9-10 AM 配量。 Data Factory 會自動追蹤時間序列資料集的相依性。 因此，它不會開始 9-10 AM 下游配量的活動執行。

Data Factory 監視和管理工具可讓您深入診斷記錄以了解失敗的配量，輕鬆地找出問題的根本原因並加以修正。 在您修正問題之後，即可輕易地開始活動執行以產生失敗的配量。 如需有關如何重新執行和了解資料配量的狀態轉換的詳細資訊，請參閱[使用 Azure 入口網站刀鋒視窗監視和管理管線](data-factory-monitor-manage-pipelines.md)或[監視和管理應用程式](data-factory-monitor-manage-app.md)。

在您重新執行 **Dataset2**的 9-10 AM 配量之後，Data Factory 會開始執行最終資料集上 9-10 AM 相依的配量。

![重新執行失敗的配量](./media/data-factory-scheduling-and-execution/rerun-failed-slice.png)

## <a name="multiple-activities-in-a-pipeline"></a>管線中的多個活動
您可以在一個管線中包含多個活動。 如果您的管線中有多個活動，而且活動的輸出不是另一個活動的輸入，則當活動的輸入資料配量已備妥時，活動可能會平行執行。

您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 活動可以在相同的管線中或在不同的管線中。 只有當第一個活動執行成功完成時，第二個活動才會執行。

例如，請思考一下以下情況，其中管線包含兩個活動：

1. 需要外部輸入資料集 D1 並且會產生輸出資料集 D2 的活動 A1。
2. 需要來自資料集 D2 之輸入並且會產生輸出資料集 D3 的活動 A2。

在此案例中，活動 A1 和 A2 是在相同的管線中。 活動 A1 會在有外部資料可供使用且達到排定的可用性頻率時執行。 活動 A2 會在來自 D2 的排定分割可供使用且達到排定的可用性頻率時執行。 如果資料集 D2 中的其中一個分割發生錯誤，則不會針對該分割執行 A2，直到該分割可供使用為止。

兩個活動同時在相同管線中的 [圖表] 檢視看起來如下圖：

![相同管線中的鏈結活動](./media/data-factory-scheduling-and-execution/chaining-one-pipeline.png)

如先前所述，活動可以在不同的管線中。 在這類案例中，圖表檢視看起來會如下圖：

![兩個管線中的鏈結活動](./media/data-factory-scheduling-and-execution/chaining-two-pipelines.png)

如需範例，請參閱附錄中的[循序複製](#copy-sequentially)一節。

## <a name="model-datasets-with-different-frequencies"></a>使用不同的頻率模型化資料集
在範例中，輸入和輸出資料集和活動排程時段的頻率是相同的。 某些案例需要能夠以不同於一或多個輸入的頻率產生輸出。 Data Factory 支援模型化這些案例。

### <a name="sample-1-produce-a-daily-output-report-for-input-data-that-is-available-every-hour"></a>範例 1：對每小時可用的輸入資料產生每日輸出報告
請設想 Azure Blob 儲存體中每小時會有來自感應器的輸入測量資料的案例。 您想要為具有 [Data Factory Hive 活動](data-factory-hive-activity.md)的日子，產生具有統計資料 (例如平均值、最大值及最小值) 的每日彙總報告。

以下是使用 Data Factory 模型化此案例的方式：

**輸入資料集**

每小時輸入檔案會針對指定日期在資料夾中卸除。 輸入的可用性設定為 **小時** (頻率：小時、間隔：1)。

```json
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
      "partitionedBy": [
        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "MM"}},
        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "dd"}}
      ],
      "format": {
        "type": "TextFormat"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
**輸出資料集**

每天會在當天的資料夾中建立一個輸出檔。 輸出的可用性設定為 **日** (頻率：日、間隔：1)。

```json
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
      "partitionedBy": [
        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "MM"}},
        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "dd"}}
      ],
      "format": {
        "type": "TextFormat"
      }
    },
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```

**活動：管線中的 Hive 活動**

Hive 指令碼會收到適當的「日期時間」  資訊，做為使用 **WindowStart** 變數的參數，如下列程式碼片段所示。 Hive 指令碼會使用此變數將資料從正確的資料夾載入，並執行彙總來產生輸出。

```json
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2015-01-01T08:00:00",
    "end":"2015-01-01T11:00:00",
    "description":"hive activity",
    "activities": [
        {
            "name": "SampleHiveActivity",
            "inputs": [
                {
                    "name": "AzureBlobInput"
                }
            ],
            "outputs": [
                {
                    "name": "AzureBlobOutput"
                }
            ],
            "linkedServiceName": "HDInsightLinkedService",
            "type": "HDInsightHive",
            "typeProperties": {
                "scriptPath": "adftutorial\\hivequery.hql",
                "scriptLinkedService": "StorageLinkedService",
                "defines": {
                    "Year": "$$Text.Format('{0:yyyy}',WindowStart)",
                    "Month": "$$Text.Format('{0:MM}',WindowStart)",
                    "Day": "$$Text.Format('{0:dd}',WindowStart)"
                }
            },
            "scheduler": {
                "frequency": "Day",
                "interval": 1
            },            
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 2,
                "timeout": "01:00:00"
            }
         }
     ]
   }
}
```

下圖顯示從資料相依性觀點來看的案例。

![資料相依性](./media/data-factory-scheduling-and-execution/data-dependency.png)

每一天的輸出配量取決於輸入資料集之 24 小時每小時的配量。 Data Factory 會自動計算這些相依性，方法是釐清落在與要產生之輸出配量相同時間期間的輸入資料配量。 如果這 24 個輸入配量中有任何一個無法使用，Data Factory 會先等待輸入配量就緒，再開始每日活動執行。

### <a name="sample-2-specify-dependency-with-expressions-and-data-factory-functions"></a>範例 2：使用運算式和 Data Factory 函數指定相依性
我們來看一下另一種案例。 假設您有處理兩個輸入資料集的 Hive 活動。 其中一個每日有新資料，但是另一個會每週取得新資料。 假設您想要跨兩個輸入進行聯結作業，並且每日產生輸出。

Data Factory 會藉由對齊輸出資料配量的時間期間來自動找出要處理的正確輸入配量的簡單方法不會奏效。

您必須指定對於每個活動執行，Data Factory 應該針對每週輸入資料集使用上一週的資料配量。 使用 Azure Data Factory 的函式來實作此行為，如下列程式碼片段所示。

**Input1：Azure Blob**

第一個輸入是將會每日更新的 Azure Blob。

```json
{
  "name": "AzureBlobInputDaily",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
      "partitionedBy": [
        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "MM"}},
        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "dd"}}
      ],
      "format": {
        "type": "TextFormat"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```

**Input2：Azure Blob**

Input2 是將會每週更新的 Azure Blob。

```json
{
  "name": "AzureBlobInputWeekly",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
      "partitionedBy": [
        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "MM"}},
        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "dd"}}
      ],
      "format": {
        "type": "TextFormat"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Day",
      "interval": 7
    }
  }
}
```

**輸出：Azure Blob**

每天會在資料夾中建立一個當天的輸出檔。 輸出的可用性設定為 **日** (頻率：日、間隔：1)。

```json
{
  "name": "AzureBlobOutputDaily",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/{Year}/{Month}/{Day}/",
      "partitionedBy": [
        { "name": "Year", "value": {"type": "DateTime","date": "SliceStart","format": "yyyy"}},
        { "name": "Month","value": {"type": "DateTime","date": "SliceStart","format": "MM"}},
        { "name": "Day","value": {"type": "DateTime","date": "SliceStart","format": "dd"}}
      ],
      "format": {
        "type": "TextFormat"
      }
    },
    "availability": {
      "frequency": "Day",
      "interval": 1
    }
  }
}
```

**活動：管線中的 Hive 活動**

Hive 活動接受 2 個輸入，並且每日產生輸出配量。 您可以針對每週輸入指定每日的輸出配量是根據上一週的輸入配量，如下所示。

```json
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2015-01-01T08:00:00",
    "end":"2015-01-01T11:00:00",
    "description":"hive activity",
    "activities": [
      {
        "name": "SampleHiveActivity",
        "inputs": [
          {
            "name": "AzureBlobInputDaily"
          },
          {
            "name": "AzureBlobInputWeekly",
            "startTime": "Date.AddDays(SliceStart, - Date.DayOfWeek(SliceStart))",
            "endTime": "Date.AddDays(SliceEnd,  -Date.DayOfWeek(SliceEnd))"  
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutputDaily"
          }
        ],
        "linkedServiceName": "HDInsightLinkedService",
        "type": "HDInsightHive",
        "typeProperties": {
          "scriptPath": "adftutorial\\hivequery.hql",
          "scriptLinkedService": "StorageLinkedService",
          "defines": {
            "Year": "$$Text.Format('{0:yyyy}',WindowStart)",
            "Month": "$$Text.Format('{0:MM}',WindowStart)",
            "Day": "$$Text.Format('{0:dd}',WindowStart)"
          }
        },
        "scheduler": {
          "frequency": "Day",
          "interval": 1
        },            
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 2,  
          "timeout": "01:00:00"
        }
       }
     ]
   }
}
```

如需 Data Factory 所支援的函數與系統變數清單，請參閱 [Data Factory 函式與系統變數](data-factory-functions-variables.md) 。

## <a name="appendix"></a>附錄

### <a name="example-copy-sequentially"></a>範例：循序複製
您可以利用循序/排序的方式，逐一執行多個複製作業。 例如，您在管線中可能有兩個具有下列輸入資料輸出資料集的複製活動 (CopyActivity1 和 CopyActivity2)：   

CopyActivity1

輸入：Dataset1。 輸出：Dataset2。

CopyActivity2

輸入：Dataset2。  輸出：Dataset3。

唯有當 CopyActivity1 成功執行且 Dataset2 可供使用時，CopyActivity2 才會執行。

以下是範例管線 JSON：

```json
{
    "name": "ChainActivities",
    "properties": {
        "description": "Run activities in sequence",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "copyBehavior": "PreserveHierarchy",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "Dataset1"
                    }
                ],
                "outputs": [
                    {
                        "name": "Dataset2"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00"
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "CopyFromBlob1ToBlob2",
                "description": "Copy data from a blob to another"
            },
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "Dataset2"
                    }
                ],
                "outputs": [
                    {
                        "name": "Dataset3"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00"
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "CopyFromBlob2ToBlob3",
                "description": "Copy data from a blob to another"
            }
        ],
        "start": "2016-08-25T01:00:00Z",
        "end": "2016-08-25T01:00:00Z",
        "isPaused": false
    }
}
```

請注意，在此範例中，是將第一個複製活動的輸出資料集 (Dataset2) 指定為第二個活動的輸入。 因此，只有當來自第一個活動的輸出資料集準備就緒時，第二個活動才會執行。  

在範例中，CopyActivity2 可以有不同的輸入 (例如 Dataset3)，但是您必須指定 Dataset2 做為 CopyActivity2 的輸入，因此在 CopyActivity1 完成之前，活動不會執行。 例如︰

CopyActivity1

輸入︰Dataset1。 輸出：Dataset2。

CopyActivity2

輸入︰Dataset3、Dataset2。 輸出︰Dataset4。

```json
{
    "name": "ChainActivities",
    "properties": {
        "description": "Run activities in sequence",
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "copyBehavior": "PreserveHierarchy",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "Dataset1"
                    }
                ],
                "outputs": [
                    {
                        "name": "Dataset2"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00"
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "CopyFromBlobToBlob",
                "description": "Copy data from a blob to another"
            },
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "Dataset3"
                    },
                    {
                        "name": "Dataset2"
                    }
                ],
                "outputs": [
                    {
                        "name": "Dataset4"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00"
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "CopyFromBlob3ToBlob4",
                "description": "Copy data from a blob to another"
            }
        ],
        "start": "2017-04-25T01:00:00Z",
        "end": "2017-04-25T01:00:00Z",
        "isPaused": false
    }
}
```

請注意，在此範例中，為第二個複製活動指定了兩個輸入資料集。 指定多個輸入時，只有第一個輸入資料集會用來複製資料，但是其他資料集會用來做為相依性。 CopyActivity2 只會在符合下列條件後才開始︰

* CopyActivity1 已順利完成且 Dataset2 可供使用。 將資料複製到 Dataset4 時，不會使用此資料集。 它只會用來做為 CopyActivity2 的排程相依性。   
* Dataset3 可供使用。 此資料集代表已複製到目的地的資料。 
