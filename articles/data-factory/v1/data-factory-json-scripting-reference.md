---
title: Azure Data Factory - JSON 指令碼參考 | Microsoft Docs
description: 提供 Data Factory 實體的 JSON 結構描述。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: c5909c1f511d3a7816ebafc3ea8b326edb7f14e3
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39629492"
---
# <a name="azure-data-factory---json-scripting-reference"></a>Azure Data Factory - JSON 指令碼參考
> [!NOTE]
> 本文適用於 Data Factory 第 1 版。


本文提供 JSON 結構描述和範例來定義 Azure Data Factory 實體 (管線、活動、資料集和連結服務)。  

## <a name="pipeline"></a>管線 
管線定義的高階結構如下所示︰ 

```json
{
  "name": "SamplePipeline",
  "properties": {
    "description": "Describe what pipeline does",
    "activities": [
    ],
    "start": "2016-07-12T00:00:00",
    "end": "2016-07-13T00:00:00"
  }
} 
```

下表說明管線 JSON 定義內的屬性：

| 屬性 | 說明 | 必要
-------- | ----------- | --------
| name | 管線的名稱。 指定一個名稱，以表示活動或管線設定要進行的動作<br/><ul><li>字元數目上限︰260</li><li>開頭必須為字母、數字或底線 (_)</li><li>不允許使用下列字元：“.”、“+”、“?”、“/”、“<”、”>”、”*”、”%”、”&”、”:”、”\\”</li></ul> |是 |
| 說明 |說明活動或管線用途的文字 | 否 |
| 活動 | 包含活動清單。 | 是 |
| start |管線的開始日期時間。 必須使用 [ISO 格式](http://en.wikipedia.org/wiki/ISO_8601)。 例如︰2014-10-14T16:32:41。 <br/><br/>您可以指定本地時間，如 EST 時間。 範例如下︰`2016-02-27T06:00:00**-05:00`，這是 6 AM EST。<br/><br/>管線的 start 和 end 屬性共同指定管線的作用中期間。 輸出配量只會在作用中期間內產生。 |否<br/><br/>如果您指定 end 屬性的值，也必須指定 start 屬性的值。<br/><br/>開始和結束時間都可以是空白來建立管線。 必須指定兩個值，才能設定執行管線的作用中時間。 如果您建立管線時未指定開始和結束時間，您可以在稍後使用 Set-AzureRmDataFactoryPipelineActivePeriod Cmdlet 進行設定。 |
| end |管線的結束日期時間。 如果已指定，則必須使用 ISO 格式。 例如：2014-10-14T17:32:41 <br/><br/>您可以指定本地時間，如 EST 時間。 範例如下︰`2016-02-27T06:00:00**-05:00`，這是美加東部標準時間上午 6 點。<br/><br/>若要無限期地執行管線，請指定 9999-09-09 做為 end 屬性的值。 |否 <br/><br/>如果您指定 start 屬性的值，也必須指定 end 屬性的值。<br/><br/>請參閱 **start** 屬性的註釋。 |
| isPaused |如果設為 true，管線不會執行。 預設值 = false。 您可以使用此屬性來啟用或停用。 |否 |
| pipelineMode |排程管線執行的方法。 允許的值包括：scheduled (預設值)、onetime。<br/><br/>‘Scheduled’ 表示管線會根據其作用中期間 (開始和結束時間) 依指定的時間間隔執行。 ‘Onetime’ 表示管線只會執行一次。 目前，Onetime 管線在建立之後即無法進行修改/更新。 如需 onetime 設定的詳細資料，請參閱 [Onetime 管線](data-factory-create-pipelines.md#onetime-pipeline)。 |否 |
| expirationTime |建立之後，管線有效且應該保持佈建的期間。 如果管線沒有任何作用中、失敗或擱置執行，則會在到達到期時間之後自動予以刪除。 |否 |


## <a name="activity"></a>活動 
管線定義 (activities 元素) 內活動的高階結構如下所示︰

```json
{
    "name": "ActivityName",
    "description": "description", 
    "type": "<ActivityType>",
    "inputs":  "[]",
    "outputs":  "[]",
    "linkedServiceName": "MyLinkedService",
    "typeProperties":
    {

    },
    "policy":
    {
    }
    "scheduler":
    {
    }
}
```

下表說明活動 JSON 定義內的屬性：

| Tag | 說明 | 必要 |
| --- | --- | --- |
| name |活動的名稱。 指定名稱，代表活動設定要進行的動作<br/><ul><li>字元數目上限︰260</li><li>開頭必須為字母、數字或底線 (_)</li><li>不允許使用下列字元：“.”、“+”、“?”、“/”、“<”、”>”、”*”、”%”、”&”、”:”、”\\”</li></ul> |是 |
| 說明 |說明活動用途的文字。 |否 |
| type |指定活動的類型。 如需了解不同類型的活動，請參閱[資料存放區](#data-stores)和[資料轉換活動](#data-transformation-activities)小節。 |是 |
| 輸入 |活動所使用的輸入資料表<br/><br/>`// one input table`<br/>`"inputs":  [ { "name": "inputtable1"  } ],`<br/><br/>`// two input tables` <br/>`"inputs":  [ { "name": "inputtable1"  }, { "name": "inputtable2"  } ],` |對 HDInsightStreaming 和 SqlServerStoredProcedure 活動來說為非必要 <br/> <br/> 對所有其他活動來說為必要 |
| 輸出 |活動所使用的輸出資料表。<br/><br/>`// one output table`<br/>`"outputs":  [ { "name": “outputtable1” } ],`<br/><br/>`//two output tables`<br/>`"outputs":  [ { "name": “outputtable1” }, { "name": “outputtable2” }  ],` |是 |
| 預設容器 |活動所使用的連結服務名稱。 <br/><br/>活動可能會要求您指定可連結至所需計算環境的連結服務。 |對於 HDInsight 活動、Azure Machine Learning 活動和預存程序活動而言為必要。 <br/><br/>否：所有其他 |
| typeProperties |typeProperties 區段中的屬性會視活動的類型而定。 |否 |
| 原則 |會影響活動之執行階段行為的原則。 如果未指定，則會使用預設原則。 |否 |
| scheduler |“scheduler” 屬性用來定義所要的活動排程。 其子屬性與 [資料集中的可用性屬性](data-factory-create-datasets.md#dataset-availability)中的屬性相同。 |否 |

### <a name="policies"></a>原則
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

### <a name="typeproperties-section"></a>typeProperties 區段
每個活動的 typeProperties 區段不同。 轉換活動只有 type 屬性。 如需在管線中定義轉換活動的 JSON 範例，請參閱本文中的[資料轉換活動](#data-transformation-activities)一節。 

**複製活動**的 typeProperties 區段中有兩個子區段︰**source** 和 **sink**。 如需示範如何使用資料存放區作為來源和/或接收的 JSON 範例，請參閱本文中的[資料存放區](#data-stores)一節。 

### <a name="sample-copy-pipeline"></a>範例複製管線
在以下的範例管線中， **Copy** in the **活動** 類型的活動。 在此範例中， [Copy 活動](data-factory-data-movement-activities.md) 會將資料從 Azure Blob 儲存體複製到 Azure SQL 資料庫。 

```json
{
  "name": "CopyPipeline",
  "properties": {
    "description": "Copy data from a blob to Azure SQL table",
    "activities": [
      {
        "name": "CopyFromBlobToSQL",
        "type": "Copy",
        "inputs": [
          {
            "name": "InputDataset"
          }
        ],
        "outputs": [
          {
            "name": "OutputDataset"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink",
            "writeBatchSize": 10000,
            "writeBatchTimeout": "60:00:00"
          }
        },
        "Policy": {
          "concurrency": 1,
          "executionPriorityOrder": "NewestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ],
    "start": "2016-07-12T00:00:00",
    "end": "2016-07-13T00:00:00"
  }
} 
```

請注意下列幾點：

* 在活動區段中，只會有一個 **type** 設為 **Copy** 的活動。
* 活動的輸入設定為 **InputDataset**，活動的輸出則設定為 **OutputDataset**。
* 在 **typeProperties** 區段中，來源類型指定為 **BlobSource**，接收類型指定為 **SqlSink**。

如需示範如何使用資料存放區作為來源和/或接收的 JSON 範例，請參閱本文中的[資料存放區](#data-stores)一節。    

如需建立此管線的完整逐步解說，請參閱 [教學課程：將資料從 Blob 儲存體複製到 SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

### <a name="sample-transformation-pipeline"></a>範例轉換管線
在以下的範例管線中， **CopyActivity** in the **活動** 類型的活動。 在此範例中， [HDInsight Hive 活動](data-factory-hive-activity.md) 會執行 Azure HDInsight Hadoop 叢集上的 Hive 指令碼檔案，來轉換 Azure Blob 儲存體的資料。 

```json
{
    "name": "TransformPipeline",
    "properties": {
        "description": "My first Azure Data Factory pipeline",
        "activities": [
            {
                "type": "HDInsightHive",
                "typeProperties": {
                    "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                    "scriptLinkedService": "AzureStorageLinkedService",
                    "defines": {
                        "inputtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/inputdata",
                        "partitionedtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/partitioneddata"
                    }
                },
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
                "policy": {
                    "concurrency": 1,
                    "retry": 3
                },
                "scheduler": {
                    "frequency": "Month",
                    "interval": 1
                },
                "name": "RunSampleHiveActivity",
                "linkedServiceName": "HDInsightOnDemandLinkedService"
            }
        ],
        "start": "2016-04-01T00:00:00",
        "end": "2016-04-02T00:00:00",
        "isPaused": false
    }
}
```

請注意下列幾點： 

* 在活動區段中，只會有一個 **type** 設為 **HDInsightHive** 的活動。
* Hive 指令碼檔案 **partitionweblogs.hql** 儲存於 Azure 儲存體帳戶 (透過名為 **AzureStorageLinkedService** 的 scriptLinkedService 指定)，且位於 **adfgetstarted** 容器的 **script** 資料夾中。
* **defines** 區段用來指定執行階段設定，該設定將傳遞到 Hive 指令碼作為 Hive 設定值 (例如，`${hiveconf:inputtable}`、`${hiveconf:partitionedtable}`)。

如需在管線中定義轉換活動的 JSON 範例，請參閱本文中的[資料轉換活動](#data-transformation-activities)一節。

如需建立此管線的完整逐步解說，請參閱 [教學課程：使用 Hadoop 叢集建置您的第一個管線來處理資料](data-factory-build-your-first-pipeline.md)。 

## <a name="linked-service"></a>連結服務
連結服務定義的高階結構如下所示︰

```json
{
    "name": "<name of the linked service>",
    "properties": {
        "type": "<type of the linked service>",
        "typeProperties": {
        }
    }
}
```

下表說明活動 JSON 定義內的屬性：

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- | 
| name | 連結服務的名稱。 | 是 | 
| properties - type | 連結服務的類型。 例如︰Azure 儲存體、Azure SQL Database。 |
| typeProperties | TypeProperties 區段中的元素隨著每個資料存放區或計算環境而不同。 關於所有資料存放區連結服務，請參閱[資料存放區](#datastores)一節，關於所有計算連結服務，請參閱[計算環境](#compute-environments)一節 |   

## <a name="dataset"></a>Dataset 
Azure Data Factory 中的資料集定義如下：

```json
{
    "name": "<name of dataset>",
    "properties": {
        "type": "<type of dataset: AzureBlob, AzureSql etc...>",
        "external": <boolean flag to indicate external data. only for input datasets>,
        "linkedServiceName": "<Name of the linked service that refers to a data store.>",
        "structure": [
            {
                "name": "<Name of the column>",
                "type": "<Name of the type>"
            }
        ],
        "typeProperties": {
            "<type specific property>": "<value>",
            "<type specific property 2>": "<value 2>",
        },
        "availability": {
            "frequency": "<Specifies the time unit for data slice production. Supported frequency: Minute, Hour, Day, Week, Month>",
            "interval": "<Specifies the interval within the defined frequency. For example, frequency set to 'Hour' and interval set to 1 indicates that new data slices should be produced hourly>"
        },
       "policy":
        {      
        }
    }
}
```

下表描述上述 JSON 的屬性：   

| 屬性 | 說明 | 必要 | 預設值 |
| --- | --- | --- | --- |
| name | 資料集的名稱。 請參閱 [Azure Data Factory - 命名規則](data-factory-naming-rules.md) ，以了解命名規則。 |是 |NA |
| type | 資料集的類型。 指定 Azure Data Factory 支援的其中一個類型 (例如︰AzureBlob、AzureSqlTable)。 關於 Data Factory 支援的所有資料存放區和資料集類型，請參閱[資料存放區](#data-stores)一節。 | 
| structure | 資料集的結構描述。 它包含資料行、其類型等。 | 否 |NA |
| typeProperties | 對應至所選類型的屬性。 關於支援的類型和其屬性，請參閱[資料存放區](#data-stores)一節。 |是 |NA |
| external | 用來指定資料集是否由 Data Factory 管線明確產生的布林值旗標。 |否 |false |
| availability | 定義處理時間範圍或資料集生產的切割模型。 如需資料集切割模型的詳細資訊，請參閱 [排程和執行](data-factory-scheduling-and-execution.md) 一文。 |是 |NA |
| 原則 |定義資料集配量必須符合的準則或條件。 <br/><br/>如需詳細資訊，請參閱 [資料集原則](#Policy) 一節。 |否 |NA |

**structure** 區段中的每個資料行包含下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| name |資料行的名稱。 |是 |
| type |資料行的資料類型。  |否 |
| culture |\.NET 型文化特性是在已指定類型 (type) 且是 \.NET 類型 `Datetime` 或 `Datetimeoffset` 時使用。 預設值為 `en-us`。 |否 |
| format |格式字串是在已指定類型且是 .NET 類型 `Datetime` 或 `Datetimeoffset` 時使用。 |否 |

在下列範例中，資料集有三個資料行 `slicetimestamp`、`projectname` 及 `pageviews`，類型分別為：String、String 及 Decimal。

```json
structure:  
[
    { "name": "slicetimestamp", "type": "String"},
    { "name": "projectname", "type": "String"},
    { "name": "pageviews", "type": "Decimal"}
]
```

下表描述您在 **availability** 區段中可使用的屬性：

| 屬性 | 說明 | 必要 | 預設值 |
| --- | --- | --- | --- |
| frequency |指定資料集配量生產的時間單位。<br/><br/><b>支援的頻率</b>：Minute、Hour、Day、Week、Month |是 |NA |
| interval |指定頻率的倍數<br/><br/>「頻率 x 間隔」會決定產生配量的頻率。<br/><br/>如果您需要將資料集以每小時為單位來切割，請將 <b>Frequency</b> 設定為 <b>Hour</b>，將 <b>interval</b> 設定為 <b>1</b>。<br/><br/><b>注意</b>：如果您將 Frequency 指定為 Minute，建議您將 interval 設定為不小於 15 |是 |NA |
| style |指定是否應該在間隔開始/結束時產生配量。<ul><li>StartOfInterval</li><li>EndOfInterval</li></ul><br/><br/>如果 Frequency 設為 Month，style 設為 EndOfInterval，則會在當月最後一天產生配量。 如果 style 設為 StartOfInterval，則會在每月的第一天產生配量。<br/><br/>如果 Frequency 設為 Day，style 設為 EndOfInterval，則會在當天最後一個小時產生配量。<br/><br/>如果 Frequency 設為 Hour，style 設為 EndOfInterval，則會在每小時結束時產生配量。 例如，若為下午 1 – 2 點期間的配量，此配量會在下午 2 點產生。 |否 |EndOfInterval |
| anchorDateTime |定義排程器用來計算資料集配量界限的時間絕對位置。 <br/><br/><b>注意</b>：如果 AnchorDateTime 有比頻率更細微的日期部分，則系統會忽略那些更細微的部分。 <br/><br/>例如，如果 <b>interval</b> 為 <b>hourly</b> (frequency: hour 且 interval: 1)，而且 <b>AnchorDateTime</b> 包含<b>分鐘和秒鐘</b>，則會忽略 AnchorDateTime 的<b>分鐘和秒鐘</b>部分。 |否 |01/01/0001 |
| Offset |所有資料集配量的開始和結束移位所依據的時間範圍。 <br/><br/><b>注意</b>︰如果同時指定 anchorDateTime 和 offset，結果會是合併的位移。 |否 |NA |

下列 availability 區段指定輸出資料集是每小時產生 (或) 輸入資料集是每小時可用：

```json
"availability":    
{    
    "frequency": "Hour",        
    "interval": 1    
}
```

資料集中的 **policy** 區段定義資料集配量必須符合的準則或條件。

| 原則名稱 | 說明 | 適用於 | 必要 | 預設值 |
| --- | --- | --- | --- | --- |
| minimumSizeMB |驗證 **Azure Blob** 中的資料是否符合最小的大小需求 (以 MB 為單位)。 |Azure Blob |否 |NA |
| minimumRows |驗證 **Azure SQL Database** 或 **Azure 資料表**中的資料是否包含最小的資料列數。 |<ul><li>連接字串</li><li>Azure 資料表</li></ul> |否 |NA |

**範例：**

```json
"policy":

{
    "validation":
    {
        "minimumSizeMB": 10.0
    }
}
```

除非資料集是由 Azure Data Factory 產生，否則應該標示為 **外部**。 除非會使用活動或管線鏈結，否則此設定通常會套用到管線中第一個活動的輸入。

| Name | 說明 | 必要 | 預設值 |
| --- | --- | --- | --- |
| dataDelay |延遲指定配量之外部資料可用性檢查的時間。 例如，如果是每小時有資料可用，則可以使用 dataDelay 來延遲查看「外部資料是否可用，且對應的配量是否已就緒」的檢查。<br/><br/>僅適用於目前的時間。  例如，如果現在時間是下午 1:00，且此值是 10 分鐘，驗證就會在下午 1:10 開始。<br/><br/>此設定不會影響過去配量 (配量結束時間 + dataDelay < 現在的配量) 的處理方式而不產生任何延遲。<br/><br/>時間若大於 23:59 小時，則必須使用 `day.hours:minutes:seconds` 格式指定。 例如，若要指定 24 小時，請不要使用 24:00:00；請改用 1.00:00:00。 如果您使用 24:00:00，這會視同 24 天 (24.00:00:00)。 如為 1 天又 4 小時，請指定 1:04:00:00。 |否 |0 |
| retryInterval |失敗與下一步重試嘗試之間的等待時間。 如果嘗試失敗，則下一次嘗試是在 retryInterval 之後。 <br/><br/>如果現在是下午 1:00，我們會開始第一次嘗試。 如果完成第一次驗證檢查的持續時間是 1 分鐘且作業失敗，則下一次重試會在 1:00 + 1 分鐘 (持續時間) + 1 分鐘 (重試間隔) = 1:02 PM。 <br/><br/>若是過去的配量，則不會有任何延遲。 重試會立即發生。 |否 |00:01:00 (1 分鐘) |
| retryTimeout |每次重試嘗試的逾時。<br/><br/>如果此屬性設為 10 分鐘，則必須在 10 分鐘內完成驗證。 如果花超過 10 分鐘來執行驗證，則重試會逾時。<br/><br/>如果驗證的所有嘗試都逾時，配量會標示為 TimedOut。 |否 |00:10:00 (10 分鐘) |
| maximumRetry |檢查外部資料可用性的次數。 允許的最大值為 10。 |否 |3 |


## <a name="data-stores"></a>資料存放區
[連結服務](#linked-service)一節描述所有連結服務類型通用的 JSON 元素。 本節提供每個資料存放區特有的 JSON 元素的詳細資料。

[資料集](#dataset)一節描述所有資料集類型通用的 JSON 元素。 本節提供每個資料存放區特有的 JSON 元素的詳細資料。

[活動](#activity)一節描述所有活動類型通用的 JSON 元素。 本節詳細說明複製活動中作為來源/接收之每個資料存放區特有的 JSON 元素。  

按一下您感興趣之存放區的連結，以查看連結服務的 JSON 結構描述、資料集，以及複製活動的來源/接收。

| 類別 | 資料存放區 
|:--- |:--- |
| **Azure** |[Azure Blob 儲存體](#azure-blob-storage) |
| &nbsp; |[Azure Data Lake Store](#azure-datalake-store) |
| &nbsp; |[Azure Cosmos DB](#azure-cosmos-db) |
| &nbsp; |[Azure SQL Database](#azure-sql-database) |
| &nbsp; |[Azure SQL 資料倉儲](#azure-sql-data-warehouse) |
| &nbsp; |[Azure 搜尋服務](#azure-search) |
| &nbsp; |[Azure 表格儲存體](#azure-table-storage) |
| **資料庫** |[Amazon Redshift](#amazon-redshift) |
| &nbsp; |[IBM DB2](#ibm-db2) |
| &nbsp; |[MySQL](#mysql) |
| &nbsp; |[Oracle](#oracle) |
| &nbsp; |[PostgreSQL](#postgresql) |
| &nbsp; |[SAP Business Warehouse](#sap-business-warehouse) |
| &nbsp; |[SAP HANA](#sap-hana) |
| &nbsp; |[SQL Server](#sql-server) |
| &nbsp; |[Sybase](#sybase) |
| &nbsp; |[Teradata](#teradata) |
| **NoSQL** |[Cassandra](#cassandra) |
| &nbsp; |[MongoDB](#mongodb) |
| **檔案** |[Amazon S3](#amazon-s3) |
| &nbsp; |[檔案系統](#file-system) |
| &nbsp; |[FTP](#ftp) |
| &nbsp; |[HDFS](#hdfs) |
| &nbsp; |[SFTP](#sftp) |
| **其他** |[HTTP](#http) |
| &nbsp; |[OData](#odata) |
| &nbsp; |[ODBC](#odbc) |
| &nbsp; |[Salesforce](#salesforce) |
| &nbsp; |[Web 資料表](#web-table) |

## <a name="azure-blob-storage"></a>Azure Blob 儲存體

### <a name="linked-service"></a>連結服務
有兩種連結服務類型︰Azure 儲存體連結服務和 Azure 儲存體 SAS 連結服務。

#### <a name="azure-storage-linked-service"></a>Azure 儲存體連結服務
若要使用**帳戶金鑰**將 Azure 儲存體帳戶連結至資料處理站，請建立 Azure 儲存體連結服務。 若要定義 Azure 儲存體連結服務，請將連結服務的**類型**設為 **AzureStorage**。 然後，您可以在 **typeProperties** 區段中指定下列屬性：  

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| connectionString |針對 connectionString 屬性指定連接到 Azure 儲存體所需的資訊。 |是 |

##### <a name="example"></a>範例  

```json
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

#### <a name="azure-storage-sas-linked-service"></a>Azure 儲存體 SAS 連結服務
Azure 儲存體 SAS 連結服務可讓您使用共用存取簽章 (SAS)，將 Azure 儲存體帳戶連結到 Azure Data Factory。 它提供受限制/時間界限存取權，讓資料處理站存取儲存體中的所有/特定資源 (blob/容器)。 若要使用共用存取簽章將 Azure 儲存體帳戶連結至資料處理站，請建立 Azure 儲存體 SAS 連結服務。 若要定義 Azure 儲存體 SAS 連結服務，請將連結服務的**類型**設為 **AzureStorageSas**。 然後，您可以在 **typeProperties** 區段中指定下列屬性：   

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| sasUri |指定 Azure 儲存體資源 (例如 Blob、容器或資料表) 的共用存取簽章 URI。 |是 |

##### <a name="example"></a>範例

```json
{  
    "name": "StorageSasLinkedService",  
    "properties": {  
        "type": "AzureStorageSas",  
        "typeProperties": {  
            "sasUri": "<storageUri>?<sasToken>"   
        }  
    }  
}  
```

如需這些連結服務的詳細資訊，請參閱 [Azure Blob 儲存體連接器](data-factory-azure-blob-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 Azure Blob 資料集，請將資料集的 **type** 設為 **AzureBlob**。 然後，在 **typeProperties** 區段中指定下列 Azure Blob 特定屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| folderPath |Blob 儲存體中容器和資料夾的路徑。 範例：myblobcontainer\myblobfolder\ |是 |
| fileName |Blob 的名稱。 fileName 是選擇性的，而且區分大小寫。<br/><br/>如果您指定檔案名稱，活動 (包括複製) 適用於特定的 Blob。<br/><br/>如果您未指定 fileName，複製會包含 folderPath 中的所有 Blob 以做為輸入資料集。<br/><br/>沒有為輸出資料集指定 fileName 時，所產生的檔案名稱會是下列格式：Data.<Guid>.txt (例如：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |否 |
| partitionedBy |partitionedBy 是選擇性的屬性。 您可以用來指定時間序列資料的動態 folderPath 和 filename。 例如，folderPath 可針對每小時的資料進行參數化。 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 <br><br> 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。 支援的層級為：**Optimal** 和 **Fastest**。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |

#### <a name="example"></a>範例

```json
{
    "name": "AzureBlobInput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "fileName": "input.log",
            "folderPath": "adfgetstarted/inputdata",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        },
        "external": true,
        "policy": {}
    }
}
 ```


如需詳細資訊，請參閱 [Azure Blob 連接器](data-factory-azure-blob-connector.md#dataset-properties)文件。

### <a name="blobsource-in-copy-activity"></a>複製活動中的 BlobSource
如果您從 Azure Blob 儲存體複製資料，請將複製活動的 **source type** 設為 **BlobSource**，並在 **source** 區段中指定下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| 遞迴 |表示是否從子資料夾，或只有從指定的資料夾，以遞迴方式讀取資料。 |True (預設值)、False |否 |

#### <a name="example-blobsource"></a>範例：**BlobSource**
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoSQL",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureSqlOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "SqlSink"
                }
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```
### <a name="blobsink-in-copy-activity"></a>複製活動中的 BlobSink
如果您將資料複製到 Azure Blob 儲存體，請將複製活動的 **sink type** 設為 **BlobSink**，並在 **sink** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| copyBehavior |當來源為 BlobSource 或 FileSystem 時，定義複製行為。 |<b>PreserveHierarchy</b>：保留目標資料夾中的檔案階層。 來源檔案到來源資料夾的相對路徑，與目標檔案到目標資料夾的相對路徑相同。<br/><br/><b>FlattenHierarchy</b>：來自來源資料夾的所有檔案都在目標資料夾的第一層中。 目標檔案會有自動產生的名稱。 <br/><br/><b>MergeFiles (預設值)</b>：將來自來源資料夾的所有檔案合併成一個檔案。 如果已指定檔案/Blob 名稱，合併檔案名稱會是指定的名稱；否則，就會是自動產生的檔案名稱。 |否 |

#### <a name="example-blobsink"></a>範例︰BlobSink

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureSQLtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureSQLInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱 [Azure Blob 連接器](data-factory-azure-blob-connector.md#copy-activity-properties)文件。 

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

### <a name="linked-service"></a>連結服務
若要定義 Azure Data Lake Store 連結服務，請將連結服務的 type 設為 **AzureDataLakeStore**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type | type 屬性必須設為： **AzureDataLakeStore** | 是 |
| dataLakeStoreUri | 指定有關 Azure Data Lake Store 帳戶的資訊。 它的格式如下：`https://[accountname].azuredatalakestore.net/webhdfs/v1` 或 `adl://[accountname].azuredatalakestore.net/`。 | 是 |
| subscriptionId | Data Lake Store 所屬的 Azure 訂用帳戶識別碼。 | 接收 (Sink) 的必要項目 |
| resourceGroupName | Data Lake Store 所屬的 Azure 資源群組名稱。 | 接收 (Sink) 的必要項目 |
| servicePrincipalId | 指定應用程式的用戶端識別碼。 | 是 (適用於服務主體驗證) |
| servicePrincipalKey | 指定應用程式的金鑰。 | 是 (適用於服務主體驗證) |
| tenant | 指定您的應用程式所在租用戶的資訊 (網域名稱或租用戶識別碼)。 將滑鼠游標暫留在 Azure 入口網站右上角，即可擷取它。 | 是 (適用於服務主體驗證) |
| 授權 | 按一下 [Data Factory 編輯器] 中的 [授權] 按鈕，然後輸入您的認證，此動作會將自動產生的授權 URL 指派給此屬性。 | 是 (適用於使用者認證驗證)|
| sessionId | OAuth 授權工作階段的 OAuth 工作階段識別碼。 每個工作階段識別碼都是唯一的，只能使用一次。 當您使用 Data Factory 編輯器時便會自動產生此設定。 | 是 (適用於使用者認證驗證) |

#### <a name="example-using-service-principal-authentication"></a>範例：使用服務主體驗證
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info. Example: microsoft.onmicrosoft.com>"
        }
    }
}
```

#### <a name="example-using-user-credential-authentication"></a>範例︰使用使用者認證驗證
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "sessionId": "<session ID>",
            "authorization": "<authorization URL>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

如需詳細資訊，請參閱 [Azure Data Lake Store 連接器](data-factory-azure-datalake-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 Azure Data Lake Store 資料集，請將資料集的 **type** 設為 **AzureDataLakeStore**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| folderPath |Azure Data Lake Store 中容器與資料夾的路徑。 |是 |
| fileName |Azure Data Lake Store 中的檔案名稱。 fileName 是選擇性的，而且區分大小寫。 <br/><br/>如果您指定檔案名稱，活動 (包括複製) 適用於特定的檔案。<br/><br/>如果您未指定 fileName，複製會包含 folderPath 中的所有檔案以做為輸入資料集。<br/><br/>沒有為輸出資料集指定 fileName 時，所產生的檔案名稱會是下列格式：Data<Guid>.txt (例如：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |否 |
| partitionedBy |partitionedBy 是選擇性的屬性。 您可以用來指定時間序列資料的動態 folderPath 和 filename。 例如，folderPath 可針對每小時的資料進行參數化。 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 <br><br> 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。 支援的層級為：**Optimal** 和 **Fastest**。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |

#### <a name="example"></a>範例
```json
{
    "name": "AzureDataLakeStoreInput",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/input/",
            "fileName": "SearchLog.tsv",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            }
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

如需詳細資訊，請參閱 [Azure Data Lake Store 連接器](data-factory-azure-datalake-connector.md#dataset-properties)文件。 

### <a name="azure-data-lake-store-source-in-copy-activity"></a>複製活動中的 Azure Data Lake Store 來源
如果您從 Azure Data Lake Store 複製資料，請將複製活動的 **source type** 設為 **AzureDataLakeStoreSource**，並在 **source** 區段中指定下列屬性︰

**AzureDataLakeStoreSource** 支援下列屬性 **typeProperties** 區段：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| 遞迴 |表示是否從子資料夾，或只有從指定的資料夾，以遞迴方式讀取資料。 |True (預設值)、False |否 |

#### <a name="example-azuredatalakestoresource"></a>範例：AzureDataLakeStoreSource

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureDakeLaketoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureDataLakeStoreInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "AzureDataLakeStoreSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱 [Azure Data Lake Store 連接器](data-factory-azure-datalake-connector.md#copy-activity-properties)文件。

### <a name="azure-data-lake-store-sink-in-copy-activity"></a>複製活動中的 Azure Data Lake Store 接收
如果您將資料複製到 Azure Data Lake Store，請將複製活動的 **sink type** 設為 **AzureDataLakeStoreSink**，並在 **sink** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| copyBehavior |指定複製行為。 |<b>PreserveHierarchy</b>：保留目標資料夾中的檔案階層。 來源檔案到來源資料夾的相對路徑，與目標檔案到目標資料夾的相對路徑相同。<br/><br/><b>FlattenHierarchy</b>：來源資料夾的中所有檔案都會建立在目標資料夾的第一個層級中。 建立的目標檔案會具有自動產生的名稱。<br/><br/><b>MergeFiles</b>：將來源資料夾的所有檔案合併為一個檔案。 如果已指定檔案/Blob 名稱，合併檔案名稱會是指定的名稱；否則，就會是自動產生的檔案名稱。 |否 |

#### <a name="example-azuredatalakestoresink"></a>範例：AzureDataLakeStoreSink
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoDataLake",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureDataLakeStoreOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "AzureDataLakeStoreSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱 [Azure Data Lake Store 連接器](data-factory-azure-datalake-connector.md#copy-activity-properties)文件。 

## <a name="azure-cosmos-db"></a>Azure Cosmos DB  

### <a name="linked-service"></a>連結服務
若要定義 Azure Cosmos DB 連結服務，請將連結服務的 **type** 設定為 **DocumentDb**，並在 **typeProperties** 區段中指定下列屬性︰  

| **屬性** | **說明** | **必要** |
| --- | --- | --- |
| connectionString |指定連接到 Azure Cosmos DB 資料庫所需的資訊。 |是 |

#### <a name="example"></a>範例

```json
{
    "name": "CosmosDBLinkedService",
    "properties": {
        "type": "DocumentDb",
        "typeProperties": {
            "connectionString": "AccountEndpoint=<EndpointUrl>;AccountKey=<AccessKey>;Database=<Database>"
        }
    }
}
```
如需詳細資訊，請參閱 [Azure Cosmos DB 連接器](data-factory-azure-documentdb-connector.md#linked-service-properties)一文。

### <a name="dataset"></a>Dataset
若要定義 Azure Cosmos DB 資料集，請將資料集的 **type** 設定為 **DocumentDbCollection**，並在 **typeProperties** 區段中指定下列屬性︰ 

| **屬性** | **說明** | **必要** |
| --- | --- | --- |
| collectionName |Azure Cosmos DB 集合的名稱。 |是 |

#### <a name="example"></a>範例

```json
{
    "name": "PersonCosmosDBTable",
    "properties": {
        "type": "DocumentDbCollection",
        "linkedServiceName": "CosmosDBLinkedService",
        "typeProperties": {
            "collectionName": "Person"
        },
        "external": true,
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```
如需詳細資訊，請參閱 [Azure Cosmos DB 連接器](data-factory-azure-documentdb-connector.md#dataset-properties)一文。

### <a name="azure-cosmos-db-collection-source-in-copy-activity"></a>複製活動中的 Azure Cosmos DB 集合來源
如果您要從 Azure Cosmos DB 複製資料，請將複製活動的 **source type** 設定為 **DocumentDbCollectionSource**，並在 **source** 區段中指定下列屬性︰


| **屬性** | **說明** | **允許的值** | **必要** |
| --- | --- | --- | --- |
| query |指定查詢來讀取資料。 |Azure Cosmos DB 所支援的查詢字串。 <br/><br/>範例： `SELECT c.BusinessEntityID, c.PersonType, c.NameStyle, c.Title, c.Name.First AS FirstName, c.Name.Last AS LastName, c.Suffix, c.EmailPromotion FROM c WHERE c.ModifiedDate > \"2009-01-01T00:00:00\"` |否 <br/><br/>如果未指定，執行的 SQL 陳述式：`select <columns defined in structure> from mycollection` |
| nestingSeparator |用來表示文件為巢狀文件的特殊字元 |任何字元。 <br/><br/>Azure Cosmos DB 是 JSON 文件的 NoSQL 存放區 (允許巢狀結構)。 Azure Data Factory 可讓使用者透過 nestingSeparator (也就是上述範例中的 “.”) 表示階層 。 使用分隔符號，複製活動將會根據資料表定義中的 “Name.First”、“Name.Middle” 和 “Name.Last”，產生含有三個子元素 (First、Middle 和 Last) 的 "Name" 物件。 |否 |

#### <a name="example"></a>範例

```json
{
    "name": "DocDbToBlobPipeline",
    "properties": {
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "DocumentDbCollectionSource",
                    "query": "SELECT Person.Id, Person.Name.First AS FirstName, Person.Name.Middle as MiddleName, Person.Name.Last AS LastName FROM Person",
                    "nestingSeparator": "."
                },
                "sink": {
                    "type": "BlobSink",
                    "blobWriterAddHeader": true,
                    "writeBatchSize": 1000,
                    "writeBatchTimeout": "00:00:59"
                }
            },
            "inputs": [{
                "name": "PersonCosmosDBTable"
            }],
            "outputs": [{
                "name": "PersonBlobTableOut"
            }],
            "policy": {
                "concurrency": 1
            },
            "name": "CopyFromCosmosDbToBlob"
        }],
        "start": "2016-04-01T00:00:00",
        "end": "2016-04-02T00:00:00"
    }
}
```

### <a name="azure-cosmos-db-collection-sink-in-copy-activity"></a>複製活動中的 Azure Cosmos DB 集合接收器
如果您要將資料複製到 Azure Cosmos DB，請將複製活動的 **sink type** 設定為 **DocumentDbCollectionSink**，並在 **sink** 區段中指定下列屬性︰

| **屬性** | **說明** | **允許的值** | **必要** |
| --- | --- | --- | --- |
| nestingSeparator |來源資料行名稱中用來表示需要巢狀文件的特殊字元。 <br/><br/>就上述範例而言：輸出資料表中的 `Name.First` 會在 Cosmos DB 文件中產生下列 JSON 結構：<br/><br/>"Name": {<br/>    "First": "John"<br/>}, |用來分隔巢狀層級的字元。<br/><br/>預設值為 `.` (點)。 |用來分隔巢狀層級的字元。 <br/><br/>預設值為 `.` (點)。 |
| writeBatchSize |為了建立文件而傳送到 Azure Cosmos DB 服務的平行要求數目。<br/><br/>您可以在將資料複製到 Azure Cosmos DB 或從該處複製資料時，使用這個屬性來微調效能。 增大 writeBatchSize 的值時，預期可以提升效能，因為會對 Azure Cosmos DB 傳送更多平行要求。 不過，您必須避免可能擲回錯誤訊息的節流：「要求速率很高」。<br/><br/>節流是由許多因素所決定，包括文件大小、文件中的詞彙數目、目標集合的檢索原則等。對於複製作業，您可以使用更好的集合 (例如 S3) 以取得最多可用輸送量 (2,500 要求單位/秒)。 |整數  |否 (預設值：5) |
| writeBatchTimeout |在逾時前等待作業完成的時間。 |時間範圍<br/><br/> 範例：“00:30:00” (30 分鐘)。 |否 |

#### <a name="example"></a>範例

```json
{
    "name": "BlobToDocDbPipeline",
    "properties": {
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "DocumentDbCollectionSink",
                    "nestingSeparator": ".",
                    "writeBatchSize": 2,
                    "writeBatchTimeout": "00:00:00"
                },
                "translator": {
                    "type": "TabularTranslator",
                    "ColumnMappings": "FirstName: Name.First, MiddleName: Name.Middle, LastName: Name.Last, BusinessEntityID: BusinessEntityID, PersonType: PersonType, NameStyle: NameStyle, Title: Title, Suffix: Suffix"
                }
            },
            "inputs": [{
                "name": "PersonBlobTableIn"
            }],
            "outputs": [{
                "name": "PersonCosmosDbTableOut"
            }],
            "policy": {
                "concurrency": 1
            },
            "name": "CopyFromBlobToCosmosDb"
        }],
        "start": "2016-04-14T00:00:00",
        "end": "2016-04-15T00:00:00"
    }
}
```

如需詳細資訊，請參閱 [Azure Cosmos DB 連接器](data-factory-azure-documentdb-connector.md#copy-activity-properties)一文。

## <a name="azure-sql-database"></a>連接字串

### <a name="linked-service"></a>連結服務
若要定義 Azure SQL Database 連結服務，請將連結服務的 **type** 設為 **AzureSqlDatabase**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| connectionString |針對 connectionString 屬性指定連接到 Azure SQL Database 執行個體所需的資訊。 |是 |

#### <a name="example"></a>範例
```json
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "type": "AzureSqlDatabase",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        }
    }
}
```

如需詳細資訊，請參閱 [Azure SQL 連接器](data-factory-azure-sql-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 Azure SQL Database 資料集，請將資料集的 **type** 設為 **AzureSqlTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |Azure SQL Database 執行個體中連結服務所參考的資料表或檢視的名稱。 |是 |

#### <a name="example"></a>範例

```json
{
    "name": "AzureSqlInput",
    "properties": {
        "type": "AzureSqlTable",
        "linkedServiceName": "AzureSqlLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```
如需詳細資訊，請參閱 [Azure SQL 連接器](data-factory-azure-sql-connector.md#dataset-properties)文件。 

### <a name="sql-source-in-copy-activity"></a>複製活動中的 SQL 來源
如果您從 Azure SQL Database 複製資料，請將複製活動的 **source type** 設為 **SqlSource**，並在 **source** 區段中指定下列屬性︰


| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| SqlReaderQuery |使用自訂查詢來讀取資料。 |SQL 查詢字串。 範例： `select * from MyTable`. |否 |
| sqlReaderStoredProcedureName |從來源資料表讀取資料的預存程序名稱。 |預存程序的名稱。 |否 |
| storedProcedureParameters |預存程序的參數。 |名稱/值組。 參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 |否 |

#### <a name="example"></a>範例

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureSQLtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureSQLInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```
如需詳細資訊，請參閱 [Azure SQL 連接器](data-factory-azure-sql-connector.md#copy-activity-properties)文件。 

### <a name="sql-sink-in-copy-activity"></a>複製活動中的 SQL 接收
如果您將資料複製到 Azure SQL Database，請將複製活動的 **sink type** 設為 **SqlSink**，並在 **sink** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| writeBatchTimeout |在逾時前等待批次插入作業完成的時間。 |時間範圍<br/><br/> 範例：“00:30:00” (30 分鐘)。 |否 |
| writeBatchSize |當緩衝區大小達到 writeBatchSize 時，將資料插入 SQL 資料表中 |整數 (資料列數目) |否 (預設值：10000) |
| sqlWriterCleanupScript |指定要讓「複製活動」執行的查詢，以便清除特定分割的資料。 |查詢陳述式。 |否 |
| sliceIdentifierColumnName |指定要讓「複製活動」以自動產生的分割識別碼填入的資料行名稱，這可在重新執行時用來清除特定分割的資料。 |資料類型為 binary(32) 之資料行的資料行名稱。 |否 |
| sqlWriterStoredProcedureName |將資料更新插入 (更新/插入) 目標資料表中的預存程序名稱。 |預存程序的名稱。 |否 |
| storedProcedureParameters |預存程序的參數。 |名稱/值組。 參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 |否 |
| sqlWriterTableType |指定要在預存程序中使用的資料表類型名稱。 複製活動可讓正在移動的資料可用於此資料表類型的暫存資料表。 然後，預存程序程式碼可以合併正在複製的資料與現有的資料。 |資料表類型名稱。 |否 |

#### <a name="example"></a>範例

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoSQL",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureSqlOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource",
                    "blobColumnSeparators": ","
                },
                "sink": {
                    "type": "SqlSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱 [Azure SQL 連接器](data-factory-azure-sql-connector.md#copy-activity-properties)文件。 

## <a name="azure-sql-data-warehouse"></a>Azure SQL 資料倉儲

### <a name="linked-service"></a>連結服務
若要定義 Azure SQL 資料倉儲連結服務，請將連結服務的 **type** 設為 **AzureSqlDW**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| connectionString |針對 connectionString 屬性指定連線到 Azure SQL 資料倉儲執行個體所需的資訊。 |是 |



#### <a name="example"></a>範例

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        }
    }
}
```

如需詳細資訊，請參閱 [Azure SQL 資料倉儲連接器](data-factory-azure-sql-data-warehouse-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 Azure SQL 資料倉儲資料集，請將資料集的 **type** 設為 **AzureSqlDWTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |Azure SQL 資料倉儲資料庫中連結服務所參照的資料表名稱或檢視名稱。 |是 |

#### <a name="example"></a>範例

```json
{
    "name": "AzureSqlDWInput",
    "properties": {
    "type": "AzureSqlDWTable",
        "linkedServiceName": "AzureSqlDWLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

如需詳細資訊，請參閱 [Azure SQL 資料倉儲連接器](data-factory-azure-sql-data-warehouse-connector.md#dataset-properties)文件。 

### <a name="sql-dw-source-in-copy-activity"></a>複製活動中的 SQL DW 來源
如果您從 Azure SQL 資料倉儲複製資料，請將複製活動的 **source type** 設為 **SqlDWSource**，並在 **source** 區段中指定下列屬性︰


| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| SqlReaderQuery |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如： `select * from MyTable` 。 |否 |
| sqlReaderStoredProcedureName |從來源資料表讀取資料的預存程序名稱。 |預存程序的名稱。 |否 |
| storedProcedureParameters |預存程序的參數。 |名稱/值組。 參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 |否 |

#### <a name="example"></a>範例

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureSQLDWtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureSqlDWInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlDWSource",
                    "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱 [Azure SQL 資料倉儲連接器](data-factory-azure-sql-data-warehouse-connector.md#copy-activity-properties)文件。 

### <a name="sql-dw-sink-in-copy-activity"></a>複製活動中的 SQL DW 接收
如果您將資料複製到 Azure SQL 資料倉儲，請將複製活動的 **sink type** 設為 **SqlDWSink**，並在 **sink** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| sqlWriterCleanupScript |指定要讓「複製活動」執行的查詢，以便清除特定分割的資料。 |查詢陳述式。 |否 |
| allowPolyBase |指出是否使用 PolyBase (適用的話) 而不是使用 BULKINSERT 機制。 <br/><br/> **建議使用 PolyBase 將資料載入 SQL 資料倉儲。** |True <br/>FALSE (預設值) |否 |
| polyBaseSettings |可以在 **allowPolybase** 屬性設定為 **true** 時指定的一組屬性。 |&nbsp; |否 |
| rejectValue |指定在查詢失敗前可以拒絕的資料列數目或百分比。 <br/><br/>在 **CREATE EXTERNAL TABLE (Transact-SQL)** 主題的 [引數](https://msdn.microsoft.com/library/dn935021.aspx) 一節中，深入了解 PolyBase 的拒絕選項。 |0 (預設值)、1、2、… |否 |
| rejectType |指定要將 rejectValue 選項指定為常值或百分比。 |值 (預設值)、百分比 |否 |
| rejectSampleValue |決定在 PolyBase 重新計算已拒絕的資料列百分比之前，所要擷取的資料列數目。 |1、2、… |是，如果 **rejectType** 是 **percentage** |
| useTypeDefault |指定當 PolyBase 從文字檔擷取資料時，如何處理分隔符號文字檔中的遺漏值。<br/><br/>從 [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](https://msdn.microsoft.com/library/dn935026.aspx) 的＜引數＞一節深入了解這個屬性。 |True/False (預設值為 False) |否 |
| writeBatchSize |當緩衝區大小達到 writeBatchSize 時，將資料插入 SQL 資料表中 |整數 (資料列數目) |否 (預設值：10000) |
| writeBatchTimeout |在逾時前等待批次插入作業完成的時間。 |時間範圍<br/><br/> 範例：“00:30:00” (30 分鐘)。 |否 |

#### <a name="example"></a>範例

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoSQLDW",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureSqlDWOutput"
            }],
            "typeProperties": {
                "source": {
                "type": "BlobSource",
                    "blobColumnSeparators": ","
                },
                "sink": {
                    "type": "SqlDWSink",
                    "allowPolyBase": true
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱 [Azure SQL 資料倉儲連接器](data-factory-azure-sql-data-warehouse-connector.md#copy-activity-properties)文件。 

## <a name="azure-search"></a>Azure 搜尋服務

### <a name="linked-service"></a>連結服務
若要定義 Azure 搜尋服務連結服務，請將連結服務的 **type** 設為 **AzureSearch**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- |
| url | Azure 搜尋服務的 URL。 | 是 |
| 索引鍵 | Azure 搜尋服務的系統管理金鑰。 | 是 |

#### <a name="example"></a>範例

```json
{
    "name": "AzureSearchLinkedService",
    "properties": {
        "type": "AzureSearch",
        "typeProperties": {
            "url": "https://<service>.search.windows.net",
            "key": "<AdminKey>"
        }
    }
}
```

如需詳細資訊，請參閱 [Azure 搜尋服務連接器](data-factory-azure-search-connector.md#linked-service-properties)文件。

### <a name="dataset"></a>Dataset
若要定義 Azure 搜尋服務資料集，請將資料集的 **type** 設為 **AzureSearchIndex**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- |
| type | type 屬性必須設為 **AzureSearchIndex**。| 是 |
| IndexName | Azure 搜尋服務索引的名稱。 Data Factory 不會建立索引。 索引必須存在於 Azure 搜尋服務中。 | 是 |

#### <a name="example"></a>範例

```json
{
    "name": "AzureSearchIndexDataset",
    "properties": {
        "type": "AzureSearchIndex",
        "linkedServiceName": "AzureSearchLinkedService",
        "typeProperties": {
            "indexName": "products"
        },
        "availability": {
            "frequency": "Minute",
            "interval": 15
        }
    }
}
```

如需詳細資訊，請參閱 [Azure 搜尋服務連接器](data-factory-azure-search-connector.md#dataset-properties)文件。

### <a name="azure-search-index-sink-in-copy-activity"></a>複製活動中的 Azure 搜尋服務索引接收
如果您將資料複製到 Azure 搜尋服務索引，請將複製活動的 **sink type** 設為 **AzureSearchIndexSink**，並在 **sink** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| -------- | ----------- | -------------- | -------- |
| WriteBehavior | 指定若文件已經存在於索引中，是否要合併或取代。 | 合併 (預設值)<br/>上傳| 否 |
| WriteBatchSize | 當緩衝區大小達到 writeBatchSize 時，將資料上傳至 Azure 搜尋服務中。 | 1 到 1000。 預設值為 1000。 | 否 |

#### <a name="example"></a>範例

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "SqlServertoAzureSearchIndex",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": " SqlServerInput"
            }],
            "outputs": [{
                "name": "AzureSearchIndexDataset"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "AzureSearchIndexSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱 [Azure 搜尋服務連接器](data-factory-azure-search-connector.md#copy-activity-properties)文件。

## <a name="azure-table-storage"></a>Azure 表格儲存體

### <a name="linked-service"></a>連結服務
有兩種連結服務類型︰Azure 儲存體連結服務和 Azure 儲存體 SAS 連結服務。

#### <a name="azure-storage-linked-service"></a>Azure 儲存體連結服務
若要使用**帳戶金鑰**將 Azure 儲存體帳戶連結至資料處理站，請建立 Azure 儲存體連結服務。 若要定義 Azure 儲存體連結服務，請將連結服務的**類型**設為 **AzureStorage**。 然後，您可以在 **typeProperties** 區段中指定下列屬性：  

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type |類型屬性必須設為： **AzureStorage** |是 |
| connectionString |針對 connectionString 屬性指定連接到 Azure 儲存體所需的資訊。 |是 |

**範例：**  

```json
{  
    "name": "StorageLinkedService",  
    "properties": {  
        "type": "AzureStorage",  
        "typeProperties": {  
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"  
        }  
    }  
}  
```

#### <a name="azure-storage-sas-linked-service"></a>Azure 儲存體 SAS 連結服務
Azure 儲存體 SAS 連結服務可讓您使用共用存取簽章 (SAS)，將 Azure 儲存體帳戶連結到 Azure Data Factory。 它提供受限制/時間界限存取權，讓資料處理站存取儲存體中的所有/特定資源 (blob/容器)。 若要使用共用存取簽章將 Azure 儲存體帳戶連結至資料處理站，請建立 Azure 儲存體 SAS 連結服務。 若要定義 Azure 儲存體 SAS 連結服務，請將連結服務的**類型**設為 **AzureStorageSas**。 然後，您可以在 **typeProperties** 區段中指定下列屬性：   

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type |類型屬性必須設為： **AzureStorageSas** |是 |
| sasUri |指定 Azure 儲存體資源 (例如 Blob、容器或資料表) 的共用存取簽章 URI。 |是 |

**範例：**

```json
{  
    "name": "StorageSasLinkedService",  
    "properties": {  
        "type": "AzureStorageSas",  
        "typeProperties": {  
            "sasUri": "<storageUri>?<sasToken>"   
        }  
    }  
}  
```

如需這些連結服務的詳細資訊，請參閱 [Azure 表格儲存體連接器](data-factory-azure-table-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 Azure 資料表資料集，請將資料集的 **type** 設為 **AzureTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |Azure 資料表資料庫執行個體中連結服務所參照的資料表名稱。 |是。 指定 tableName 時若沒有指定 azureTableSourceQuery，資料表中的所有記錄都會複製到目的地。 如果同時指定了 azureTableSourceQuery，則資料表中符合查詢的記錄會複製到目的地。 |

#### <a name="example"></a>範例

```json
{
    "name": "AzureTableInput",
    "properties": {
        "type": "AzureTable",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

如需這些連結服務的詳細資訊，請參閱 [Azure 表格儲存體連接器](data-factory-azure-table-connector.md#dataset-properties)文件。 

### <a name="azure-table-source-in-copy-activity"></a>複製活動中的 Azure 資料表來源
如果您從 Azure 表格儲存體複製資料，請將複製活動的 **source type** 設為 **AzureTableSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| AzureTableSourceQuery |使用自訂查詢來讀取資料。 |Azure 資料表查詢字串。 請參閱下一節中的範例。 |否。 指定 tableName 時若沒有指定 azureTableSourceQuery，資料表中的所有記錄都會複製到目的地。 如果同時指定了 azureTableSourceQuery，則資料表中符合查詢的記錄會複製到目的地。 |
| azureTableSourceIgnoreTableNotFound |指出是否忍受資料表不存在的例外狀況。 |TRUE<br/>FALSE |否 |

#### <a name="example"></a>範例

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureTabletoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureTableInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "AzureTableSource",
                    "AzureTableSourceQuery": "PartitionKey eq 'DefaultPartitionKey'"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需這些連結服務的詳細資訊，請參閱 [Azure 表格儲存體連接器](data-factory-azure-table-connector.md#copy-activity-properties)文件。 

### <a name="azure-table-sink-in-copy-activity"></a>複製活動中的 Azure 資料表接收
如果您將資料複製到 Azure 表格儲存體，請將複製活動的 **sink type** 設為 **AzureTableSink**，並在 **sink** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| azureTableDefaultPartitionKeyValue |可供接收器使用的預設資料分割索引鍵值。 |字串值。 |否 |
| azureTablePartitionKeyName |指定其值用來作為分割區索引鍵的資料行名稱。 如果未指定，則會使用 AzureTableDefaultPartitionKeyValue 做為資料分割索引鍵。 |資料行名稱。 |否 |
| azureTableRowKeyName |指定其值用來作為資料列索引鍵的資料行名稱。 如果未指定，則會針對每個資料列使用 GUID。 |資料行名稱。 |否 |
| azureTableInsertType |將資料插入 Azure 資料表的模式。<br/><br/>此屬性可控制針對輸出資料表中具有相符分割區和資料列索引鍵的現有資料列，是要取代還是合併其值。 <br/><br/>若要了解這些設定 (合併和取代) 的運作方式，請參閱[插入或合併實體](https://msdn.microsoft.com/library/azure/hh452241.aspx)和[插入或取代實體](https://msdn.microsoft.com/library/azure/hh452242.aspx)主題。 <br/><br> 此設定是在資料列層級套用，而不是在資料表層級套用，而且兩個選項都不會刪除存在於輸出資料表中但不存在於輸入中的資料列。 |合併 (預設值)<br/>取代 |否 |
| writeBatchSize |在達到 WriteBatchSize 或 writeBatchTimeout 時將資料插入 Azure 資料表中。 |整數 (資料列數目) |否 (預設值：10000) |
| writeBatchTimeout |在達到 WriteBatchSize 或 writeBatchTimeout 時將資料插入 Azure 資料表中 |時間範圍<br/><br/>範例：“00:20:00” (20 分鐘) |否 (預設為儲存體用戶端預設逾時值 90 秒) |

#### <a name="example"></a>範例

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoTable",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureTableOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "AzureTableSink",
                    "writeBatchSize": 100,
                    "writeBatchTimeout": "01:00:00"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```
如需這些連結服務的詳細資訊，請參閱 [Azure 表格儲存體連接器](data-factory-azure-table-connector.md#copy-activity-properties)文件。 

## <a name="amazon-redshift"></a>Amazon RedShift

### <a name="linked-service"></a>連結服務
若要定義 Amazon Redshift 連結服務，請將連結服務的 **type** 設為 **AmazonRedshift**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| 伺服器 |Amazon Redshift 伺服器的 IP 位址或主機名稱。 |是 |
| 連接埠 |Amazon Redshift 伺服器用來接聽用戶端連線的 TCP 連接埠號碼。 |否，預設值︰5439 |
| 資料庫 |Amazon Redshift 資料庫的名稱。 |是 |
| username |可存取資料庫之使用者的名稱。 |是 |
| password |使用者帳戶的密碼。 |是 |

#### <a name="example"></a>範例

```json
{
    "name": "AmazonRedshiftLinkedService",
    "properties": {
        "type": "AmazonRedshift",
        "typeProperties": {
            "server": "<Amazon Redshift host name or IP address>",
            "port": 5439,
            "database": "<database name>",
            "username": "user",
            "password": "password"
        }
    }
}
```

如需詳細資訊，請參閱 [Amazon Redshift 連接器](#data-factory-amazon-redshift-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 Amazon Redshift 資料集，請將資料集的 **type** 設為 **RelationalTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |Amazon Redshift 資料庫中連結服務所參照的資料表名稱。 |否 (如果已指定 **RelationalSource** 的 **query**) |


#### <a name="example"></a>範例

```json
{
    "name": "AmazonRedshiftInputDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "AmazonRedshiftLinkedService",
        "typeProperties": {
            "tableName": "<Table name>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
如需詳細資訊，請參閱 [Amazon Redshift 連接器](#data-factory-amazon-redshift-connector.md#dataset-properties)文件。

### <a name="relational-source-in-copy-activity"></a>複製活動中的關聯式來源 
如果您從 Amazon Redshift 複製資料，請將複製活動的 **source type** 設為 **RelationalSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如： `select * from MyTable` 。 |否 (如果已指定 **dataset** 的 **tableName**) |

#### <a name="example"></a>範例

```json
{
    "name": "CopyAmazonRedshiftToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "AmazonRedshiftInputDataset"
            }],
            "outputs": [{
                "name": "AzureBlobOutputDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "AmazonRedshiftToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```
如需詳細資訊，請參閱 [Amazon Redshift 連接器](#data-factory-amazon-redshift-connector.md#copy-activity-properties)文件。

## <a name="ibm-db2"></a>IBM DB2

### <a name="linked-service"></a>連結服務
若要定義 IBM DB2 連結服務，請將連結服務的 **type** 設為 **OnPremisesDB2**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| 伺服器 |DB2 伺服器的名稱。 |是 |
| 資料庫 |DB2 資料庫的名稱。 |是 |
| 結構描述 |在資料庫中的結構描述名稱。 結構描述名稱會區分大小寫。 |否 |
| authenticationType |用來連接到 DB2 資料庫的驗證類型。 可能的值為：匿名、基本和 Windows。 |是 |
| username |如果您使用基本或 Windows 驗證，請指定使用者名稱。 |否 |
| password |指定您為使用者名稱所指定之使用者帳戶的密碼。 |否 |
| gatewayName |Data Factory 服務應該用來連接到內部部署 DB2 資料庫的閘道器名稱。 |是 |

#### <a name="example"></a>範例
```json
{
    "name": "OnPremDb2LinkedService",
    "properties": {
        "type": "OnPremisesDb2",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "schema": "<schema>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```
如需詳細資訊，請參閱 [IBM DB2 連接器](#data-factory-onprem-db2-connector.md#linked-service-properties)文件。

### <a name="dataset"></a>Dataset
若要定義 DB2 資料集，請將資料集的 **type** 設為 **RelationalTable**，並在 **typeProperties** 區段中指定下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |DB2 資料庫執行個體中連結服務所參照的資料表名稱。 tableName 會區分大小寫。 |否 (如果已指定 **RelationalSource** 的 **query**) 

#### <a name="example"></a>範例
```json
{
    "name": "Db2DataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremDb2LinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

如需詳細資訊，請參閱 [IBM DB2 連接器](#data-factory-onprem-db2-connector.md#dataset-properties)文件。

### <a name="relational-source-in-copy-activity"></a>複製活動中的關聯式來源
如果您從 IBM DB2 複製資料，請將複製活動的 **source type** 設為 **RelationalSource**，並在 **source** 區段中指定下列屬性︰


| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如： `"query": "select * from "MySchema"."MyTable""` 。 |否 (如果已指定 **dataset** 的 **tableName**) |

#### <a name="example"></a>範例
```json
{
    "name": "CopyDb2ToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "select * from \"Orders\""
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "inputs": [{
                "name": "Db2DataSet"
            }],
            "outputs": [{
                "name": "AzureBlobDb2DataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "Db2ToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```
如需詳細資訊，請參閱 [IBM DB2 連接器](#data-factory-onprem-db2-connector.md#copy-activity-properties)文件。

## <a name="mysql"></a>MySQL

### <a name="linked-service"></a>連結服務
若要定義 MySQL 連結服務，請將連結服務的 **type** 設為 **OnPremisesMySql**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| 伺服器 |MySQL 伺服器的名稱。 |是 |
| 資料庫 |MySQL 資料庫的名稱。 |是 |
| 結構描述 |在資料庫中的結構描述名稱。 |否 |
| authenticationType |用來連接到 MySQL 資料庫的驗證類型。 可能的值為：`Basic`。 |是 |
| username |指定要連線到 MySQL 資料庫的使用者名稱。 |是 |
| password |指定您所指定使用者帳戶的密碼。 |是 |
| gatewayName |Data Factory 服務應該用來連接到內部部署 MySQL 資料庫的閘道器名稱。 |是 |

#### <a name="example"></a>範例

```json
{
    "name": "OnPremMySqlLinkedService",
    "properties": {
        "type": "OnPremisesMySql",
        "typeProperties": {
            "server": "<server name>",
            "database": "<database name>",
            "schema": "<schema name>",
            "authenticationType": "<authentication type>",
            "userName": "<user name>",
            "password": "<password>",
            "gatewayName": "<gateway>"
        }
    }
}
```

如需詳細資訊，請參閱 [MySQL 連接器](data-factory-onprem-mysql-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 MySQL 資料集，請將資料集的 **type** 設為 **RelationalTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |MySQL 資料庫執行個體中連結服務所參照的資料表名稱。 |否 (如果已指定 **RelationalSource** 的 **query**) |

#### <a name="example"></a>範例

```json
{
    "name": "MySqlDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremMySqlLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```
如需詳細資訊，請參閱 [MySQL 連接器](data-factory-onprem-mysql-connector.md#dataset-properties)文件。 

### <a name="relational-source-in-copy-activity"></a>複製活動中的關聯式來源
如果您從 MySQL 資料庫複製資料，請將複製活動的 **source type** 設為 **RelationalSource**，並在 **source** 區段中指定下列屬性︰


| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如： `select * from MyTable` 。 |否 (如果已指定 **dataset** 的 **tableName**) |


#### <a name="example"></a>範例
```json
{
    "name": "CopyMySqlToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "MySqlDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobMySqlDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "MySqlToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

如需詳細資訊，請參閱 [MySQL 連接器](data-factory-onprem-mysql-connector.md#copy-activity-properties)文件。 

## <a name="oracle"></a>Oracle 

### <a name="linked-service"></a>連結服務
若要定義 Oracle 連結服務，請將連結服務的 **type** 設為 **OnPremisesOracle**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| driverType | 指定要用來從 Oracle 複製資料/將資料複製到 Oracle 資料庫的驅動程式。 允許的值為 **Microsoft** 或 **ODP** (預設值)。 如需驅動程式詳細資料，請參閱[支援的版本和安裝](#supported-versions-and-installation)一節。 | 否 |
| connectionString | 針對 connectionString 屬性指定連接到 Oracle 資料庫執行個體所需的資訊。 | 是 |
| gatewayName | 用來連線內部部署 Oracle 伺服器的閘道名稱 |是 |

#### <a name="example"></a>範例
```json
{
    "name": "OnPremisesOracleLinkedService",
    "properties": {
        "type": "OnPremisesOracle",
        "typeProperties": {
            "driverType": "Microsoft",
            "connectionString": "Host=<host>;Port=<port>;Sid=<sid>;User Id=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```

如需詳細資訊，請參閱 [Oracle 連接器](data-factory-onprem-oracle-connector.md#linked-service-properties)文件。

### <a name="dataset"></a>Dataset
若要定義 Oracle 資料集，請將資料集的 **type** 設為 **OracleTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |Oracle 資料庫中連結服務所參照的資料表名稱。 |否 (如果已指定 **OracleSource** 的 **oracleReaderQuery**) |

#### <a name="example"></a>範例

```json
{
    "name": "OracleInput",
    "properties": {
        "type": "OracleTable",
        "linkedServiceName": "OnPremisesOracleLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "offset": "01:00:00",
            "interval": "1",
            "anchorDateTime": "2016-02-27T12:00:00",
            "frequency": "Hour"
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```
如需詳細資訊，請參閱 [Oracle 連接器](data-factory-onprem-oracle-connector.md#dataset-properties)文件。

### <a name="oracle-source-in-copy-activity"></a>複製活動中的 Oracle 來源
如果您從 Oracle 資料庫複製資料，請將複製活動的 **source type** 設為 **OracleSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| oracleReaderQuery |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如：`select * from MyTable` <br/><br/>如果未指定，執行的 SQL 陳述式：`select * from MyTable` |否 (如果已指定 **dataset** 的 **tableName**) |

#### <a name="example"></a>範例

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "OracletoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": " OracleInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "OracleSource",
                    "oracleReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
            "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱 [Oracle 連接器](data-factory-onprem-oracle-connector.md#copy-activity-properties)文件。

### <a name="oracle-sink-in-copy-activity"></a>複製活動中的 Oracle 接收
如果您將資料複製到 Oracle 資料庫，請將複製活動的 **sink type** 設為 **OracleSink**，並在 **sink** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| writeBatchTimeout |在逾時前等待批次插入作業完成的時間。 |時間範圍<br/><br/> 範例：00:30:00 (30 分鐘)。 |否 |
| writeBatchSize |當緩衝區大小達到 writeBatchSize 時，將資料插入 SQL 資料表中 |整數 (資料列數目) |否 (預設值：100) |
| sqlWriterCleanupScript |指定要讓「複製活動」執行的查詢，以便清除特定分割的資料。 |查詢陳述式。 |否 |
| sliceIdentifierColumnName |指定要讓「複製活動」以自動產生的分割識別碼填入的資料行名稱，這可在重新執行時用來清除特定分割的資料。 |資料類型為 binary(32) 之資料行的資料行名稱。 |否 |

#### <a name="example"></a>範例
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-05T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoOracle",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "OracleOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource"
                },
                "sink": {
                    "type": "OracleSink"
                }
            },
            "scheduler": {
                "frequency": "Day",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```
如需詳細資訊，請參閱 [Oracle 連接器](data-factory-onprem-oracle-connector.md#copy-activity-properties)文件。

## <a name="postgresql"></a>PostgreSQL

### <a name="linked-service"></a>連結服務
若要定義 PostgreSQL 連結服務，請將連結服務的 **type** 設為 **OnPremisesPostgreSql**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| 伺服器 |PostgreSQL 伺服器的名稱。 |是 |
| 資料庫 |PostgreSQL 資料庫的名稱。 |是 |
| 結構描述 |在資料庫中的結構描述名稱。 結構描述名稱會區分大小寫。 |否 |
| authenticationType |用來連接到 PostgreSQL 資料庫的驗證類型。 可能的值為：匿名、基本和 Windows。 |是 |
| username |如果您使用基本或 Windows 驗證，請指定使用者名稱。 |否 |
| password |指定您為使用者名稱所指定之使用者帳戶的密碼。 |否 |
| gatewayName |Data Factory 服務應該用來連接到內部部署 PostgreSQL 資料庫的閘道器名稱。 |是 |

#### <a name="example"></a>範例

```json
{
    "name": "OnPremPostgreSqlLinkedService",
    "properties": {
        "type": "OnPremisesPostgreSql",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "schema": "<schema>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```
如需詳細資訊，請參閱 [PostgreSQL 連接器](data-factory-onprem-postgresql-connector.md#linked-service-properties)文件。

### <a name="dataset"></a>Dataset
若要定義 PostgreSQL 資料集，請將資料集的 **type** 設為 **RelationalTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |PostgreSQL 資料庫執行個體中連結服務所參照的資料表名稱。 tableName 會區分大小寫。 |否 (如果已指定 **RelationalSource** 的 **query**) |

#### <a name="example"></a>範例
```json
{
    "name": "PostgreSqlDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremPostgreSqlLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```
如需詳細資訊，請參閱 [PostgreSQL 連接器](data-factory-onprem-postgresql-connector.md#dataset-properties)文件。

### <a name="relational-source-in-copy-activity"></a>複製活動中的關聯式來源
如果您從 PostgreSQL 資料庫複製資料，請將複製活動的 **source type** 設為 **RelationalSource**，並在 **source** 區段中指定下列屬性︰


| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如："query": "select * from \"MySchema\".\"MyTable\""。 |否 (如果已指定 **dataset** 的 **tableName**) |

#### <a name="example"></a>範例

```json
{
    "name": "CopyPostgreSqlToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "select * from \"public\".\"usstates\""
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "inputs": [{
                "name": "PostgreSqlDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobPostgreSqlDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "PostgreSqlToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

如需詳細資訊，請參閱 [PostgreSQL 連接器](data-factory-onprem-postgresql-connector.md#copy-activity-properties)文件。

## <a name="sap-business-warehouse"></a>SAP Business Warehouse


### <a name="linked-service"></a>連結服務
若要定義 SAP Business Warehouse (BW) 連結服務，請將連結服務的 **type** 設為 **SapBw**，並在 **typeProperties** 區段中指定下列屬性︰  

屬性 | 說明 | 允許的值 | 必要
-------- | ----------- | -------------- | --------
伺服器 | SAP BW 執行個體所在之伺服器的名稱。 | 字串 | 是
systemNumber | SAP BW 系統的系統編號。 | 以字串表示的二位數十進位數字。 | 是
clientId | SAP W 系統中用戶端的用戶端識別碼。 | 以字串表示的三位數十進位數字。 | 是
username | 具有 SAP 伺服器存取權之使用者的名稱 | 字串 | 是
password | 使用者的密碼。 | 字串 | 是
gatewayName | 資料處理站服務應該用來連線至內部部署 SAP BW 執行個體的閘道器名稱。 | 字串 | 是
encryptedCredential | 加密的認證字串。 | 字串 | 否

#### <a name="example"></a>範例

```json
{
    "name": "SapBwLinkedService",
    "properties": {
        "type": "SapBw",
        "typeProperties": {
            "server": "<server name>",
            "systemNumber": "<system number>",
            "clientId": "<client id>",
            "username": "<SAP user>",
            "password": "<Password for SAP user>",
            "gatewayName": "<gateway name>"
        }
    }
}
```

如需詳細資訊，請參閱 [SAP Business Warehouse 連接器](data-factory-sap-business-warehouse-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 SAP BW 資料集，請將資料集的 **type** 設為 **RelationalTable**。 **RelationalTable** 類型的 SAP BW 資料集不支援類型專用的屬性。  

#### <a name="example"></a>範例

```json
{
    "name": "SapBwDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "SapBwLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
如需詳細資訊，請參閱 [SAP Business Warehouse 連接器](data-factory-sap-business-warehouse-connector.md#dataset-properties)文件。 

### <a name="relational-source-in-copy-activity"></a>複製活動中的關聯式來源
如果您從 SAP Business Warehouse 複製資料，請將複製活動的 **source type** 設為 **RelationalSource**，並在 **source** 區段中指定下列屬性︰


| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query | 指定 MDX 查詢從 SAP BW 執行個體讀取資料。 | MDX 查詢。 | 是 |

#### <a name="example"></a>範例

```json
{
    "name": "CopySapBwToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "<MDX query for SAP BW>"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "SapBwDataset"
            }],
            "outputs": [{
                "name": "AzureBlobDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "SapBwToBlob"
        }],
        "start": "2017-03-01T18:00:00",
        "end": "2017-03-01T19:00:00"
    }
}
```

如需詳細資訊，請參閱 [SAP Business Warehouse 連接器](data-factory-sap-business-warehouse-connector.md#copy-activity-properties)文件。 

## <a name="sap-hana"></a>SAP HANA

### <a name="linked-service"></a>連結服務
若要定義 SAP HANA 連結服務，請將連結服務的 **type** 設為 **SapHana**，並在 **typeProperties** 區段中指定下列屬性︰  

屬性 | 說明 | 允許的值 | 必要
-------- | ----------- | -------------- | --------
伺服器 | SAP Hana 執行個體所在之伺服器的名稱。 如果您的伺服器使用自訂連接埠，指定 `server:port`。 | 字串 | 是
authenticationType | 驗證類型。 | 字串。 "Basic" 或 "Windows" | 是 
username | 具有 SAP 伺服器存取權之使用者的名稱 | 字串 | 是
password | 使用者的密碼。 | 字串 | 是
gatewayName | Data Factory 服務應該用來連接到內部部署 SAP Hana 執行個體的閘道器名稱。 | 字串 | 是
encryptedCredential | 加密的認證字串。 | 字串 | 否

#### <a name="example"></a>範例

```json
{
    "name": "SapHanaLinkedService",
    "properties": {
        "type": "SapHana",
        "typeProperties": {
            "server": "<server name>",
            "authenticationType": "<Basic, or Windows>",
            "username": "<SAP user>",
            "password": "<Password for SAP user>",
            "gatewayName": "<gateway name>"
        }
    }
}

```
如需詳細資訊，請參閱 [SAP HANA 連接器](data-factory-sap-hana-connector.md#linked-service-properties)文件。
 
### <a name="dataset"></a>Dataset
若要定義 SAP HANA 資料集，請將資料集的 **type** 設為 **RelationalTable**。 沒有類型特定的屬性支援 **RelationalTable** 的 SAP HANA 資料集類型。 

#### <a name="example"></a>範例

```json
{
    "name": "SapHanaDataset",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "SapHanaLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
如需詳細資訊，請參閱 [SAP HANA 連接器](data-factory-sap-hana-connector.md#dataset-properties)文件。 

### <a name="relational-source-in-copy-activity"></a>複製活動中的關聯式來源
如果您從 SAP HANA 資料存放區複製資料，請將複製活動的 **source type** 設為 **RelationalSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query | 指定 SQL 查詢從 SAP HANA 執行個體讀取資料。 | SQL 查詢。 | 是 |


#### <a name="example"></a>範例


```json
{
    "name": "CopySapHanaToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "<SQL Query for HANA>"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "SapHanaDataset"
            }],
            "outputs": [{
                "name": "AzureBlobDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "SapHanaToBlob"
        }],
        "start": "2017-03-01T18:00:00",
        "end": "2017-03-01T19:00:00"
    }
}
```

如需詳細資訊，請參閱 [SAP HANA 連接器](data-factory-sap-hana-connector.md#copy-activity-properties)文件。


## <a name="sql-server"></a>SQL Server

### <a name="linked-service"></a>連結服務
您可以建立 **OnPremisesSqlServer** 類型的連結服務，以將內部部署 SQL Server 資料庫連結至資料處理站。 下表提供內部部署 SQL Server 連結服務專屬 JSON 元素的描述。

下表提供 SQL Server 連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |類型屬性應設為： **OnPremisesSqlServer**。 |是 |
| connectionString |指定使用 SQL 驗證或 Windows 驗證連接至內部部署 SQL Server 資料庫所需的 connectionString 資訊。 |是 |
| gatewayName |Data Factory 服務應該用來連接到內部部署 SQL Server 資料庫的閘道器名稱。 |是 |
| username |如果您使用「Windows 驗證」，請指定使用者名稱。 範例︰**domainname\\username**。 |否 |
| password |指定您為使用者名稱所指定之使用者帳戶的密碼。 |否 |

您可以使用 **New-AzureRmDataFactoryEncryptValue** Cmdlet 加密認證，並在連接字串中使用這些認證，如下列範例所示 (**EncryptedCredential** 屬性)：  

```json
"connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=True;EncryptedCredential=<encrypted credential>",
```


#### <a name="example-json-for-using-sql-authentication"></a>範例：用於 SQL 驗證的 JSON

```json
{
    "name": "MyOnPremisesSQLDB",
    "properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "connectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=False;User ID=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```
#### <a name="example-json-for-using-windows-authentication"></a>範例：用於 Windows 驗證的 JSON

如果已指定使用者名稱和密碼，閘道就會使用它們來模擬指定的使用者帳戶，以連線到內部部署 SQL Server 資料庫。 否則，閘道會使用閘道的安全性內容 (其啟動帳戶) 直接連線到 SQL Server。

```json
{
    "Name": " MyOnPremisesSQLDB",
    "Properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "ConnectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=True;",
            "username": "<domain\\username>",
            "password": "<password>",
            "gatewayName": "<gateway name>"
        }
    }
}
```

如需詳細資訊，請參閱 [SQL Server 連接器](data-factory-sqlserver-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 SQL Server 資料集，請將資料集的 **type** 設為 **SqlServerTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |SQL Server Database 執行個體中連結服務所參照的資料表或檢視名稱。 |是 |

#### <a name="example"></a>範例
```json
{
    "name": "SqlServerInput",
    "properties": {
        "type": "SqlServerTable",
        "linkedServiceName": "SqlServerLinkedService",
        "typeProperties": {
            "tableName": "MyTable"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

如需詳細資訊，請參閱 [SQL Server 連接器](data-factory-sqlserver-connector.md#dataset-properties)文件。 

### <a name="sql-source-in-copy-activity"></a>複製活動中的 Sql 來源
如果您從 SQL Server 資料庫複製資料，請將複製活動的 **source type** 設為 **SqlSource**，並在 **source** 區段中指定下列屬性︰


| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| SqlReaderQuery |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如： `select * from MyTable` 。 可以參考輸入資料集所參考資料庫中的多個資料表。 如果未指定，執行的 SQL 陳述式：select from MyTable。 |否 |
| sqlReaderStoredProcedureName |從來源資料表讀取資料的預存程序名稱。 |預存程序的名稱。 |否 |
| storedProcedureParameters |預存程序的參數。 |名稱/值組。 參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 |否 |

如果已為 SqlSource 指定 **sqlReaderQuery** ，複製活動會針對 SQL Server 資料庫來源執行這項查詢以取得資料。

或者，您可以藉由指定 **sqlReaderStoredProcedureName** 和 **storedProcedureParameters** (如果預存程序接受參數) 來指定預存程序。

如果您未指定 sqlReaderQuery 或 sqlReaderStoredProcedureName，系統就會使用 structure 區段中定義的資料行來建立一個要對「SQL Server 資料庫」執行的 select 查詢。 如果資料集定義沒有結構，則會從資料表中選取所有資料行。

> [!NOTE]
> 當您使用 **sqlReaderStoredProcedureName** 時，仍必須為資料集 JSON 中的 **tableName** 屬性指定值。 雖然目前尚未針對此資料表來進行驗證。


#### <a name="example"></a>範例
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "SqlServertoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": " SqlServerInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

在此範例中，已為 SqlSource 指定 **sqlReaderQuery** 。 複製活動會針對 SQL Server 資料庫來源執行這項查詢以取得資料。 或者，您可以藉由指定 **sqlReaderStoredProcedureName** 和 **storedProcedureParameters** (如果預存程序接受參數) 來指定預存程序。 sqlReaderQuery 可以參考輸入資料集所參考之資料庫內的多個資料表。 這不限於只有設定為資料集之 tableName typeProperty 的資料表。

如果您未指定 sqlReaderQuery 或 sqlReaderStoredProcedureName，系統就會使用 structure 區段中定義的資料行來建立一個要對「SQL Server 資料庫」執行的 select 查詢。 如果資料集定義沒有結構，則會從資料表中選取所有資料行。

如需詳細資訊，請參閱 [SQL Server 連接器](data-factory-sqlserver-connector.md#copy-activity-properties)文件。 

### <a name="sql-sink-in-copy-activity"></a>複製活動中的 Sql 接收
如果您將資料複製到 SQL Server 資料庫，請將複製活動的 **sink type** 設為 **SqlSink**，並在 **sink** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| writeBatchTimeout |在逾時前等待批次插入作業完成的時間。 |時間範圍<br/><br/> 範例：“00:30:00” (30 分鐘)。 |否 |
| writeBatchSize |當緩衝區大小達到 writeBatchSize 時，將資料插入 SQL 資料表中 |整數 (資料列數目) |否 (預設值：10000) |
| sqlWriterCleanupScript |指定要讓「複製活動」執行的查詢，以便清除特定分割的資料。 如需詳細資訊，請參閱 [可重複性](#repeatability-during-copy) 一節。 |查詢陳述式。 |否 |
| sliceIdentifierColumnName |指定要讓「複製活動」以自動產生的分割識別碼填入的資料行名稱，這可在重新執行時用來清除特定分割的資料。 如需詳細資訊，請參閱 [可重複性](#repeatability-during-copy) 一節。 |資料類型為 binary(32) 之資料行的資料行名稱。 |否 |
| sqlWriterStoredProcedureName |將資料更新插入 (更新/插入) 目標資料表中的預存程序名稱。 |預存程序的名稱。 |否 |
| storedProcedureParameters |預存程序的參數。 |名稱/值組。 參數的名稱和大小寫必須符合預存程序參數的名稱和大小寫。 |否 |
| sqlWriterTableType |指定要在預存程序中使用的資料表類型名稱。 複製活動可讓正在移動的資料可用於此資料表類型的暫存資料表。 然後，預存程序程式碼可以合併正在複製的資料與現有的資料。 |資料表類型名稱。 |否 |

#### <a name="example"></a>範例
此管線包含「複製活動」，該活動已設定為使用這些輸入和輸出資料集，並且排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **BlobSource**，而 **sink** 類型設為 **SqlSink**。

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "AzureBlobtoSQL",
            "description": "Copy Activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": " SqlServerOutput "
            }],
            "typeProperties": {
                "source": {
                    "type": "BlobSource",
                    "blobColumnSeparators": ","
                },
                "sink": {
                    "type": "SqlSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱 [SQL Server 連接器](data-factory-sqlserver-connector.md#copy-activity-properties)文件。 

## <a name="sybase"></a>Sybase

### <a name="linked-service"></a>連結服務
若要定義 Sybase 連結服務，請將連結服務的 **type** 設為 **OnPremisesSybase**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| 伺服器 |Sybase 伺服器的名稱。 |是 |
| 資料庫 |Sybase 資料庫的名稱。 |是 |
| 結構描述 |在資料庫中的結構描述名稱。 |否 |
| authenticationType |用來連接到 Sybase 資料庫的驗證類型。 可能的值為：匿名、基本和 Windows。 |是 |
| username |如果您使用基本或 Windows 驗證，請指定使用者名稱。 |否 |
| password |指定您為使用者名稱所指定之使用者帳戶的密碼。 |否 |
| gatewayName |Data Factory 服務應該用來連接到內部部署 Sybase 資料庫的閘道器名稱。 |是 |

#### <a name="example"></a>範例
```json
{
    "name": "OnPremSybaseLinkedService",
    "properties": {
        "type": "OnPremisesSybase",
        "typeProperties": {
            "server": "<server>",
            "database": "<database>",
            "schema": "<schema>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```

如需詳細資訊，請參閱 [Sybase 連接器](data-factory-onprem-sybase-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 Sybase 資料集，請將資料集的 **type** 設為 **RelationalTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |Sybase 資料庫執行個體中連結服務所參照的資料表名稱。 |否 (如果已指定 **RelationalSource** 的 **query**) |

#### <a name="example"></a>範例

```json
{
    "name": "SybaseDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremSybaseLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

如需詳細資訊，請參閱 [Sybase 連接器](data-factory-onprem-sybase-connector.md#dataset-properties)文件。 

### <a name="relational-source-in-copy-activity"></a>複製活動中的關聯式來源
如果您從 Sybase 資料庫複製資料，請將複製活動的 **source type** 設為 **RelationalSource**，並在 **source** 區段中指定下列屬性︰


| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如： `select * from MyTable` 。 |否 (如果已指定 **dataset** 的 **tableName**) |

#### <a name="example"></a>範例

```json
{
    "name": "CopySybaseToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "select * from DBA.Orders"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "inputs": [{
                "name": "SybaseDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobSybaseDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "SybaseToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

如需詳細資訊，請參閱 [Sybase 連接器](data-factory-onprem-sybase-connector.md#copy-activity-properties)文件。

## <a name="teradata"></a>Teradata

### <a name="linked-service"></a>連結服務
若要定義 Teradata 連結服務，請將連結服務的 **type** 設為 **OnPremisesTeradata**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| 伺服器 |Teradata 伺服器的名稱。 |是 |
| authenticationType |用來連接到 Teradata 資料庫的驗證類型。 可能的值為：匿名、基本和 Windows。 |是 |
| username |如果您使用基本或 Windows 驗證，請指定使用者名稱。 |否 |
| password |指定您為使用者名稱所指定之使用者帳戶的密碼。 |否 |
| gatewayName |Data Factory 服務應該用來連接到內部部署 Teradata 資料庫的閘道器名稱。 |是 |

#### <a name="example"></a>範例
```json
{
    "name": "OnPremTeradataLinkedService",
    "properties": {
        "type": "OnPremisesTeradata",
        "typeProperties": {
            "server": "<server>",
            "authenticationType": "<authentication type>",
            "username": "<username>",
            "password": "<password>",
            "gatewayName": "<gatewayName>"
        }
    }
}
```

如需詳細資訊，請參閱 [Teradata 連接器](data-factory-onprem-teradata-connector.md#linked-service-properties)文件。

### <a name="dataset"></a>Dataset
若要定義 Teradata Blob 資料集，請將資料集的 **type** 設為 **RelationalTable**。 目前沒有支援 Teradata 資料集的類型屬性。 

#### <a name="example"></a>範例
```json
{
    "name": "TeradataDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "OnPremTeradataLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

如需詳細資訊，請參閱 [Teradata 連接器](data-factory-onprem-teradata-connector.md#dataset-properties)文件。

### <a name="relational-source-in-copy-activity"></a>複製活動中的關聯式來源
如果您從 Teradata 資料庫複製資料，請將複製活動的 **source type** 設為 **RelationalSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如： `select * from MyTable` 。 |是 |

#### <a name="example"></a>範例

```json
{
    "name": "CopyTeradataToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', SliceStart, SliceEnd)"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "TeradataDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobTeradataDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "TeradataToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "isPaused": false
    }
}
```

如需詳細資訊，請參閱 [Teradata 連接器](data-factory-onprem-teradata-connector.md#copy-activity-properties)文件。

## <a name="cassandra"></a>Cassandra


### <a name="linked-service"></a>連結服務
若要定義 Cassandra 連結服務，請將連結服務的 **type** 設為 **OnPremisesCassandra**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| host |一或多個 Cassandra 伺服器 IP 位址或主機名稱。<br/><br/>指定以逗號分隔的 IP 位址或主機名稱清單，以同時連線到所有伺服器。 |是 |
| 連接埠 |Cassandra 伺服器用來接聽用戶端連線的 TCP 連接埠。 |否，預設值：9042 |
| authenticationType |基本或匿名 |是 |
| username |指定使用者帳戶的使用者名稱。 |是，如果 authenticationType 設定為 [基本]。 |
| password |指定使用者帳戶的密碼。 |是，如果 authenticationType 設定為 [基本]。 |
| gatewayName |用來連線到內部部署 Cassandra 資料庫的閘道器名稱。 |是 |
| encryptedCredential |由閘道加密認證。 |否 |

#### <a name="example"></a>範例

```json
{
    "name": "CassandraLinkedService",
    "properties": {
        "type": "OnPremisesCassandra",
        "typeProperties": {
            "authenticationType": "Basic",
            "host": "<cassandra server name or IP address>",
            "port": 9042,
            "username": "user",
            "password": "password",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

如需詳細資訊，請參閱 [Cassandra 連接器](data-factory-onprem-cassandra-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 Cassandra 資料集，請將資料集的 **type** 設為 **CassandraTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| keyspace |Cassandra 資料庫中的 Keyspace 或結構描述名稱。 |是 (如果未定義 **CassandraSource** 的**查詢**)。 |
| tableName |Cassandra 資料庫中資料表的名稱。 |是 (如果未定義 **CassandraSource** 的**查詢**)。 |

#### <a name="example"></a>範例

```json
{
    "name": "CassandraInput",
    "properties": {
        "linkedServiceName": "CassandraLinkedService",
        "type": "CassandraTable",
        "typeProperties": {
            "tableName": "mytable",
            "keySpace": "<key space>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

如需詳細資訊，請參閱 [Cassandra 連接器](data-factory-onprem-cassandra-connector.md#dataset-properties)文件。 

### <a name="cassandra-source-in-copy-activity"></a>複製活動中的 SCassandra 來源
如果您從 Cassandra 複製資料，請將複製活動的 **source type** 設為 **CassandraSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL-92 查詢或 CQL 查詢。 請參閱 [CQL 參考資料](https://docs.datastax.com/en/cql/3.1/cql/cql_reference/cqlReferenceTOC.html)。 <br/><br/>在使用 SQL 查詢時，指定 **keyspace name.table 名稱** 來代表您想要查詢的資料表。 |否 (如果已定義資料集上的 tableName 和 keyspace)。 |
| consistencyLevel |一致性層級可指定必須先有多少複本回應讀取要求，才會將資料傳回用戶端應用程式。 Cassandra 會檢查要讓資料滿足讀取要求的指定複本數目。 |ONE、TWO、THREE、QUORUM、ALL、LOCAL_QUORUM、EACH_QUORUM、LOCAL_ONE。 如需詳細資訊，請參閱 [設定資料一致性](https://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_config_consistency_c.html) 。 |否。 預設值為 ONE。 |

#### <a name="example"></a>範例
  
```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "CassandraToAzureBlob",
            "description": "Copy from Cassandra to an Azure blob",
            "type": "Copy",
            "inputs": [{
                "name": "CassandraInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "CassandraSource",
                    "query": "select id, firstname, lastname from mykeyspace.mytable"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱 [Cassandra 連接器](data-factory-onprem-cassandra-connector.md#copy-activity-properties)文件。

## <a name="mongodb"></a>MongoDB

### <a name="linked-service"></a>連結服務
若要定義 MongoDB 連結服務，請將連結服務的 **type** 設為 **OnPremisesMongoDB**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| 伺服器 |MongoDB 伺服器的 IP 位址或主機名稱。 |是 |
| 連接埠 |MongoDB 伺服器用來接聽用戶端連線的 TCP 連接埠。 |選用，預設值︰27017 |
| authenticationType |基本或匿名。 |是 |
| username |用來存取 MongoDB 的使用者帳戶。 |是 (如果使用基本驗證)。 |
| password |使用者的密碼。 |是 (如果使用基本驗證)。 |
| authSource |您想要用來檢查驗證所用之認證的 MongoDB 資料庫名稱。 |選用 (如果使用基本驗證)。 預設值︰使用以 databaseName 屬性指定的系統管理員帳戶和資料庫。 |
| databaseName |您想要存取之 MongoDB 資料庫的名稱。 |是 |
| gatewayName |存取資料存放區之閘道的名稱。 |是 |
| encryptedCredential |由閘道加密的認證。 |選用 |

#### <a name="example"></a>範例

```json
{
    "name": "OnPremisesMongoDbLinkedService",
    "properties": {
        "type": "OnPremisesMongoDb",
        "typeProperties": {
            "authenticationType": "<Basic or Anonymous>",
            "server": "< The IP address or host name of the MongoDB server >",
            "port": "<The number of the TCP port that the MongoDB server uses to listen for client connections.>",
            "username": "<username>",
            "password": "<password>",
            "authSource": "< The database that you want to use to check your credentials for authentication. >",
            "databaseName": "<database name>",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

如需詳細資訊，請參閱 [MongoDB 連接器文件](data-factory-on-premises-mongodb-connector.md#linked-service-properties)

### <a name="dataset"></a>Dataset
若要定義 MongoDB 資料集，請將資料集的 **type** 設為 **MongoDbCollection**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| collectionName |MongoDB 資料庫中集合的名稱。 |是 |

#### <a name="example"></a>範例

```json
{
    "name": "MongoDbInputDataset",
    "properties": {
        "type": "MongoDbCollection",
        "linkedServiceName": "OnPremisesMongoDbLinkedService",
        "typeProperties": {
            "collectionName": "<Collection name>"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

如需詳細資訊，請參閱 [MongoDB 連接器文件](data-factory-on-premises-mongodb-connector.md#dataset-properties)

#### <a name="mongodb-source-in-copy-activity"></a>複製活動中的 MongoDB 來源
如果您從 MongoDB 複製資料，請將複製活動的 **source type** 設為 **MongoDbSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL-92 查詢字串。 例如： `select * from MyTable` 。 |否 (如果已指定 **dataset** 的 **collectionName**) |

#### <a name="example"></a>範例

```json
{
    "name": "CopyMongoDBToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "MongoDbSource",
                    "query": "select * from MyTable"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "MongoDbInputDataset"
            }],
            "outputs": [{
                "name": "AzureBlobOutputDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "MongoDBToAzureBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

如需詳細資訊，請參閱 [MongoDB 連接器文件](data-factory-on-premises-mongodb-connector.md#copy-activity-properties)

## <a name="amazon-s3"></a>Amazon S3


### <a name="linked-service"></a>連結服務
若要定義 Amazon S3 連結服務，請將連結服務的 **type** 設為 **AwsAccessKey**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| accessKeyID |密碼存取金鑰的識別碼。 |字串 |是 |
| secretAccessKey |密碼存取金鑰本身。 |加密的密碼字串 |是 |

#### <a name="example"></a>範例
```json
{
    "name": "AmazonS3LinkedService",
    "properties": {
        "type": "AwsAccessKey",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": "<secret access key>"
        }
    }
}
```

如需詳細資訊，請參閱 [Amazon S3 連接器文件](data-factory-amazon-simple-storage-service-connector.md#linked-service-properties)。

### <a name="dataset"></a>Dataset
若要定義 Amazon S3 資料集，請將資料集的 **type** 設為 **AmazonS3**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| bucketName |S3 貯體名稱。 |字串 |是 |
| 索引鍵 |S3 物件索引鍵。 |字串 |否 |
| prefix |S3 物件索引鍵的前置詞。 系統會選取索引鍵以此前置詞開頭的物件。 只有當索引鍵空白時才適用。 |字串 |否 |
| version |如果已啟用 S3 版本設定功能，則為 S3 物件的版本。 |字串 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 <br><br> 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 | |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。 支援的層級為：**Optimal** 和 **Fastest**。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 | |


> [!NOTE]
> bucketName + key 可指定 S3 物件的位置，其中貯體是 S3 物件的根容器，而索引鍵是 S3 物件的完整路徑。

#### <a name="example-sample-dataset-with-prefix"></a>範例：範例資料集 (含 prefix)

```json
{
    "name": "dataset-s3",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": "link- testS3",
        "typeProperties": {
            "prefix": "testFolder/test",
            "bucketName": "<S3 bucket name>",
            "format": {
                "type": "OrcFormat"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```
#### <a name="example-sample-data-set-with-version"></a>範例︰範例資料集 (含 version)

```json
{
    "name": "dataset-s3",
    "properties": {
        "type": "AmazonS3",
        "linkedServiceName": "link- testS3",
        "typeProperties": {
            "key": "testFolder/test.orc",
            "bucketName": "<S3 bucket name>",
            "version": "XXXXXXXXXczm0CJajYkHf0_k6LhBmkcL",
            "format": {
                "type": "OrcFormat"
            }
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true
    }
}
```

#### <a name="example-dynamic-paths-for-s3"></a>範例：S3 的動態路徑
在此範例中，我們針對 Amazon S3 資料集內的 key 和 bucketName 屬性使用固定的值。

```json
"key": "testFolder/test.orc",
"bucketName": "<S3 bucket name>",
```

您可以藉由使用系統變數 (例如 SliceStart)，讓 Data Factory 在執行階段動態地計算 key 和 bucketName 的值。

```json
"key": "$$Text.Format('{0:MM}/{0:dd}/test.orc', SliceStart)"
"bucketName": "$$Text.Format('{0:yyyy}', SliceStart)"
```

您也可以對 Amazon S3 資料集的 prefix 屬性執行相同的操作。 如需支援的函式和變數清單，請參閱 [Data Factory 函式與系統變數](data-factory-functions-variables.md) 。

如需詳細資訊，請參閱 [Amazon S3 連接器文件](data-factory-amazon-simple-storage-service-connector.md#dataset-properties)。

### <a name="file-system-source-in-copy-activity"></a>複製活動中的檔案系統來源
如果您從 Amazon S3 複製資料，請將複製活動的 **source type** 設為 **FileSystemSource**，並在 **source** 區段中指定下列屬性︰


| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| 遞迴 |指定是否要以遞迴方式列出目錄下的 S3 物件。 |true/false |否 |


#### <a name="example"></a>範例


```json
{
    "name": "CopyAmazonS3ToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource",
                    "recursive": true
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "AmazonS3InputDataset"
            }],
            "outputs": [{
                "name": "AzureBlobOutputDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "AmazonS3ToBlob"
        }],
        "start": "2016-08-08T18:00:00",
        "end": "2016-08-08T19:00:00"
    }
}
```

如需詳細資訊，請參閱 [Amazon S3 連接器文件](data-factory-amazon-simple-storage-service-connector.md#copy-activity-properties)。

## <a name="file-system"></a>檔案系統


### <a name="linked-service"></a>連結服務
您可以利用「內部部署檔案伺服器」已連結服務，將內部部署的檔案系統連結到 Azure Data Factory。 下表說明內部部署檔案伺服器連結服務專屬的 JSON 元素。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |確保 type 屬性設為 **OnPremisesFileServer**。 |是 |
| host |指定想要複製之資料夾的根路徑。 字串中的特殊字元需使用逸出字元 ‘ \ ’。 如需範例，請參閱 [範例連結服務和資料集定義](#sample-linked-service-and-dataset-definitions) 。 |是 |
| userid |指定具有伺服器存取權之使用者的識別碼。 |否 (如果您選擇 encryptedCredential) |
| password |指定使用者 (userid) 的密碼。 |否 (如果您選擇 encryptedCredential) |
| encryptedCredential |指定可以透過執行 New-AzureRmDataFactoryEncryptValue Cmdlet 來取得的加密認證。 |否 (如果您選擇以純文字指定使用者識別碼和密碼) |
| gatewayName |指定 Data Factory 應該用來連接到內部部署檔案伺服器的閘道器名稱。 |是 |

#### <a name="sample-folder-path-definitions"></a>範例資料夾路徑定義 
| 案例 | 連結服務定義中的主機 | 資料集定義中的 folderPath |
| --- | --- | --- |
| 資料管理閘道電腦上的本機資料夾︰ <br/><br/>範例：D:\\\* 或 D:\folder\subfolder\\* |D:\\\\ (適用於資料管理閘道 2.0 和更新版本) <br/><br/> localhost (適用於比資料管理閘道 2.0 更早的版本) |.\\\\ 或 folder\\\\subfolder (適用於資料管理閘道 2.0 和更新版本) <br/><br/>D:\\\\ 或 D:\\\\folder\\\\subfolder (適用低於閘道 2.0 的版本) |
| 遠端共用資料夾︰ <br/><br/>範例︰\\\\myserver\\share\\\* 或 \\\\myserver\\share\\folder\\subfolder\\* |\\\\\\\\myserver\\\\share |.\\\\ 或 folder\\\\subfolder |


#### <a name="example-using-username-and-password-in-plain-text"></a>範例：使用純文字的使用者名稱和密碼

```json
{
    "Name": "OnPremisesFileServerLinkedService",
    "properties": {
        "type": "OnPremisesFileServer",
        "typeProperties": {
            "host": "\\\\Contosogame-Asia",
            "userid": "Admin",
            "password": "123456",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-using-encryptedcredential"></a>範例：使用 encryptedcredential

```json
{
    "Name": " OnPremisesFileServerLinkedService ",
    "properties": {
        "type": "OnPremisesFileServer",
        "typeProperties": {
            "host": "D:\\",
            "encryptedCredential": "WFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCBvbmx5IGJ5xxxxxxxxxxxxxxxxx",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

如需詳細資訊，請參閱[檔案系統連接器文件](data-factory-onprem-file-system-connector.md#linked-service-properties)。

### <a name="dataset"></a>Dataset
若要定義檔案系統資料集，請將資料集的 **type** 設為 **FileShare**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| folderPath |指定資料夾的子路徑。 字串中的特殊字元需使用逸出字元 ‘ \ ’。 如需範例，請參閱 [範例連結服務和資料集定義](#sample-linked-service-and-dataset-definitions) 。<br/><br/>您可以結合此屬性與 **partitionBy**，讓資料夾路徑以配量開始/結束日期時間為基礎。 |是 |
| fileName |如果您想要資料表參考資料夾中的特定檔案，請指定 **folderPath** 中的檔案名稱。 如果沒有為此屬性指定任何值，資料表會指向資料夾中的所有檔案。<br/><br/>若未指定輸出資料集的 fileName，所產生檔案的名稱是下列格式︰ <br/><br/>`Data.<Guid>.txt` (例如： Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |否 |
| fileFilter |指定要用來在 folderPath (而不是所有檔案) 中選取檔案子集的篩選器。 <br/><br/>允許的值為︰`*` (多個字元) 和 `?` (單一字元)。<br/><br/>範例 1："fileFilter": "*.log"<br/>範例 2："fileFilter": 2016-1-?.txt"<br/><br/>請注意，fileFilter 適用於輸入 FileShare 資料集。 |否 |
| partitionedBy |您可以使用 partitionedBy 來指定時間序列資料的動態 folderPath/fileName。 例如，folderPath 可針對每小時的資料進行參數化。 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 <br><br> 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 和 **ZipDeflate**，而支援的層級為：**最佳**和**最快**。 請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |

> [!NOTE]
> 無法同時使用 fileName 和 fileFilter。

#### <a name="example"></a>範例

```json
{
    "name": "OnpremisesFileSystemInput",
    "properties": {
        "type": " FileShare",
        "linkedServiceName": " OnPremisesFileServerLinkedService ",
        "typeProperties": {
            "folderPath": "mysharedfolder/yearno={Year}/monthno={Month}/dayno={Day}",
            "fileName": "{Hour}.csv",
            "partitionedBy": [{
                "name": "Year",
                "value": {
                    "type": "DateTime",
                    "date": "SliceStart",
                        "format": "yyyy"
                }
            }, {
                "name": "Month",
                "value": {
                    "type": "DateTime",
                    "date": "SliceStart",
                    "format": "MM"
                }
            }, {
                "name": "Day",
                "value": {
                    "type": "DateTime",
                    "date": "SliceStart",
                    "format": "dd"
                }
            }, {
                "name": "Hour",
                "value": {
                    "type": "DateTime",
                    "date": "SliceStart",
                    "format": "HH"
                }
            }]
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

如需詳細資訊，請參閱[檔案系統連接器文件](data-factory-onprem-file-system-connector.md#dataset-properties)。

### <a name="file-system-source-in-copy-activity"></a>複製活動中的檔案系統來源
如果您從檔案系統複製資料，請將複製活動的 **source type** 設為 **FileSystemSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| 遞迴 |指出是否從子資料夾、或只有從指定的資料夾，以遞迴方式讀取資料。 |True/False (預設值為 False) |否 |

#### <a name="example"></a>範例

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2015-06-01T18:00:00",
        "end": "2015-06-01T19:00:00",
        "description": "Pipeline for copy activity",
        "activities": [{
            "name": "OnpremisesFileSystemtoBlob",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "OnpremisesFileSystemInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
            "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```
如需詳細資訊，請參閱[檔案系統連接器文件](data-factory-onprem-file-system-connector.md#copy-activity-properties)。

### <a name="file-system-sink-in-copy-activity"></a>複製活動中的檔案系統接收
如果您將資料複製到檔案系統，請將複製活動的 **sink type** 設為 **FileSystemSink**，並在 **sink** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| copyBehavior |當來源為 BlobSource 或 FileSystem 時，定義複製行為。 |**PreserveHierarchy：** 保留目標資料夾中的檔案階層。 亦即，來源檔案到來源資料夾的相對路徑，與目標檔案到目標資料夾的相對路徑相同。<br/><br/>**FlattenHierarchy：** 來源資料夾的中所有檔案都會建立在目標資料夾的第一層中。 建立的目標檔案會具有自動產生的名稱。<br/><br/>**MergeFiles：** 將來源資料夾的所有檔案合併為一個檔案。 如果有指定檔案/Blob 名稱，合併檔案的名稱會是指定的名稱。 否則，就會是自動產生的檔案名稱。 |否 |
auto-

#### <a name="example"></a>範例

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2015-06-01T18:00:00",
        "end": "2015-06-01T20:00:00",
        "description": "pipeline for copy activity",
        "activities": [{
            "name": "AzureSQLtoOnPremisesFile",
            "description": "copy activity",
            "type": "Copy",
            "inputs": [{
                "name": "AzureSQLInput"
            }],
            "outputs": [{
                "name": "OnpremisesFileSystemOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "SqlSource",
                    "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "FileSystemSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 3,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱[檔案系統連接器文件](data-factory-onprem-file-system-connector.md#copy-activity-properties)。

## <a name="ftp"></a>FTP

### <a name="linked-service"></a>連結服務
若要定義 FTP 連結服務，請將連結服務的 **type** 設為 **FtpServer**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 | 預設值 |
| --- | --- | --- | --- |
| host |FTP 伺服器的名稱或 IP 位址 |是 |&nbsp; |
| authenticationType |指定驗證類型 |是 |基本或匿名 |
| username |可存取 FTP 伺服器的使用者 |否 |&nbsp; |
| password |使用者 (使用者名稱) 的密碼 |否 |&nbsp; |
| encryptedCredential |用來存取 FTP 伺服器的加密認證 |否 |&nbsp; |
| gatewayName |要連線至內部部署 FTP 伺服器的資料管理閘道名稱 |否 |&nbsp; |
| 連接埠 |FTP 伺服器所接聽的連接埠 |否 |21 |
| enableSsl |指定是否使用透過 SSL/TLS 的 FTP 通道 |否 |true |
| enableServerCertificateValidation |指定是否在使用透過 SSL/TLS 的 FTP 通道時啟用伺服器 SSL 憑證驗證 |否 |true |

#### <a name="example-using-anonymous-authentication"></a>範例：使用匿名驗證

```json
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
            "typeProperties": {
            "authenticationType": "Anonymous",
            "host": "myftpserver.com"
        }
    }
}
```

#### <a name="example-using-username-and-password-in-plain-text-for-basic-authentication"></a>範例：使用純文字的使用者名稱和密碼進行基本驗證

```json
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "username": "Admin",
            "password": "123456"
        }
    }
}
```

#### <a name="example-using-port-enablessl-enableservercertificatevalidation"></a>範例：使用 port、enableSsl、enableServerCertificateValidation

```json
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",    
            "username": "Admin",
            "password": "123456",
            "port": "21",
            "enableSsl": true,
            "enableServerCertificateValidation": true
        }
    }
}
```

#### <a name="example-using-encryptedcredential-for-authentication-and-gateway"></a>範例：針對驗證和閘道使用 encryptedCredential

```json
{
    "name": "FTPLinkedService",
    "properties": {
        "type": "FtpServer",
        "typeProperties": {
            "host": "myftpserver.com",
            "authenticationType": "Basic",
            "encryptedCredential": "xxxxxxxxxxxxxxxxx",
            "gatewayName": "<onpremgateway>"
        }
      }
}
```

如需詳細資訊，請參閱 [FTP 連接器](data-factory-ftp-connector.md#linked-service-properties)文件。

### <a name="dataset"></a>Dataset
若要定義 FTP 資料集，請將資料集的 **type** 設為 **FileShare**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| folderPath |資料夾的子路徑。 使用逸出字元 ‘ \ ’ 當做字串中的特殊字元。 如需範例，請參閱 [範例連結服務和資料集定義](#sample-linked-service-and-dataset-definitions) 。<br/><br/>您可以結合此屬性與 **partitionBy**，讓資料夾路徑以配量開始/結束日期時間為基礎。 |是 
| fileName |如果您想要資料表參考資料夾中的特定檔案，請指定 **folderPath** 中的檔案名稱。 如果沒有為此屬性指定任何值，資料表會指向資料夾中的所有檔案。<br/><br/>若未指定輸出資料集的 fileName，所產生檔案的名稱是下列格式︰ <br/><br/>Data.<Guid>.txt (範例：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |否 |
| fileFilter |指定要用來在 folderPath (而不是所有檔案) 中選取檔案子集的篩選器。<br/><br/>允許的值為︰`*` (多個字元) 和 `?` (單一字元)。<br/><br/>範例 1：`"fileFilter": "*.log"`<br/>範例 2：`"fileFilter": 2016-1-?.txt"`<br/><br/> fileFilter 適用於輸入 FileShare 資料集。 這個屬性不支援使用 HDFS。 |否 |
| partitionedBy |partitionedBy 可以用來指定時間序列資料的動態 folderPath 和 filename。 例如，folderPath 可針對每小時的資料進行參數化。 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 <br><br> 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 和 **ZipDeflate**，而支援的層級為：**最佳**和**最快**。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |
| useBinaryTransfer |指定是否使用二進位傳輸模式。 二進位模式為 true，ASCII 則為 false。 預設值：True。 只有在相關聯的連結服務類型的類型為 FtpServer 時，才可以使用這個屬性。 |否 |

> [!NOTE]
> 無法同時使用檔名和 fileFilter。

#### <a name="example"></a>範例

```json
{
    "name": "FTPFileInput",
    "properties": {
        "type": "FileShare",
        "linkedServiceName": "FTPLinkedService",
        "typeProperties": {
            "folderPath": "<path to shared folder>",
            "fileName": "test.csv",
            "useBinaryTransfer": true
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

如需詳細資訊，請參閱 [FTP 連接器](data-factory-ftp-connector.md#dataset-properties)文件。

### <a name="file-system-source-in-copy-activity"></a>複製活動中的檔案系統來源
如果您從 FTP 伺服器複製資料，請將複製活動的 **source type** 設為 **FileSystemSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| 遞迴 |表示是否從子資料夾，或只有從指定的資料夾，以遞迴方式讀取資料。 |True/False (預設值為 False) |否 |

#### <a name="example"></a>範例

```json
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "FTPToBlobCopy",
            "inputs": [{
                "name": "FtpFileInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2016-08-24T18:00:00",
        "end": "2016-08-24T19:00:00"
    }
}
```

如需詳細資訊，請參閱 [FTP 連接器](data-factory-ftp-connector.md#copy-activity-properties)文件。


## <a name="hdfs"></a>HDFS

### <a name="linked-service"></a>連結服務
若要定義 HDFS 連結服務，請將連結服務的 **type** 設為 **Hdfs**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |類型屬性必須設為： **Hdfs** |是 |
| Url |到 HDFS 的 URL |是 |
| authenticationType |匿名或 Windows。 <br><br> 若要對 HDFS 連接器使用 **Kerberos 驗證**，請參閱[此章節](#use-kerberos-authentication-for-hdfs-connector)來據以設定您的內部部署環境。 |是 |
| userName |Windows 驗證的使用者名稱。 |是 (適用於 Windows 驗證) |
| password |Windows 驗證的密碼。 |是 (適用於 Windows 驗證) |
| gatewayName |Data Factory 服務應該用來連接到 HDFS 的閘道器名稱。 |是 |
| encryptedCredential |[New-AzureRMDataFactoryEncryptValue](https://docs.microsoft.com/powershell/module/azurerm.datafactories/new-azurermdatafactoryencryptvalue) 輸出。 |否 |

#### <a name="example-using-anonymous-authentication"></a>範例：使用匿名驗證

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "authenticationType": "Anonymous",
            "userName": "hadoop",
            "url": "http://<machine>:50070/webhdfs/v1/",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-using-windows-authentication"></a>範例：使用 Windows 驗證

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "authenticationType": "Windows",
            "userName": "Administrator",
            "password": "password",
            "url": "http://<machine>:50070/webhdfs/v1/",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

如需詳細資訊，請參閱 [HDFS 連接器](#data-factory-hdfs-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 HDFS 資料集，請將資料集的 **type** 設為 **FileShare**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| folderPath |資料夾的路徑。 範例： `myfolder`<br/><br/>使用逸出字元 ‘ \ ’ 當做字串中的特殊字元。 例如︰若為 folder\subfolder，請指定 folder\\\\subfolder；若為 d:\samplefolder，請指定 d:\\\\samplefolder。<br/><br/>您可以結合此屬性與 **partitionBy**，讓資料夾路徑以配量開始/結束日期時間為基礎。 |是 |
| fileName |如果您想要資料表參考資料夾中的特定檔案，請指定 **folderPath** 中的檔案名稱。 如果沒有為此屬性指定任何值，資料表會指向資料夾中的所有檔案。<br/><br/>若未指定輸出資料集的 fileName，所產生檔案的名稱是下列格式︰ <br/><br/>Data<Guid>.txt (例如：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |否 |
| partitionedBy |partitionedBy 可以用來指定時間序列資料的動態 folderPath 和 filename。 範例：folderPath 可針對每小時的資料進行參數化。 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 <br><br> 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。 支援的層級為：**Optimal** 和 **Fastest**。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |

> [!NOTE]
> 無法同時使用檔名和 fileFilter。

#### <a name="example"></a>範例

```json
{
    "name": "InputDataset",
    "properties": {
        "type": "FileShare",
        "linkedServiceName": "HDFSLinkedService",
        "typeProperties": {
            "folderPath": "DataTransfer/UnitTest/"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

如需詳細資訊，請參閱 [HDFS 連接器](#data-factory-hdfs-connector.md#dataset-properties)文件。 

### <a name="file-system-source-in-copy-activity"></a>複製活動中的檔案系統來源
如果您從 HDFS 複製資料，請將複製活動的 **source type** 設為 **FileSystemSource**，並在 **source** 區段中指定下列屬性︰

**FileSystemSource** 支援下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| 遞迴 |表示是否從子資料夾，或只有從指定的資料夾，以遞迴方式讀取資料。 |True/False (預設值為 False) |否 |

#### <a name="example"></a>範例

```json
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "HdfsToBlobCopy",
            "inputs": [{
                "name": "InputDataset"
            }],
            "outputs": [{
                "name": "OutputDataset"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
```

如需詳細資訊，請參閱 [HDFS 連接器](#data-factory-hdfs-connector.md#copy-activity-properties)文件。

## <a name="sftp"></a>SFTP


### <a name="linked-service"></a>連結服務
若要定義 SFTP 連結服務，請將連結服務的 **type** 設為 **Sftp**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- | --- |
| host | SFTP 伺服器的名稱或 IP 位址。 |是 |
| 連接埠 |SFTP 伺服器所接聽的連接埠。 預設值：21 |否 |
| authenticationType |指定驗證類型。 允許的值︰**Basic**、**SshPublicKey**。 <br><br> 請參閱[使用基本驗證](#using-basic-authentication)和[使用 SSH 公開金鑰驗證](#using-ssh-public-key-authentication)章節，分別取得更多屬性和 JSON 範例。 |是 |
| skipHostKeyValidation | 指定是否略過主機金鑰驗證。 | 否。 預設值：false |
| hostKeyFingerprint | 指定主機金鑰的指紋。 | 如果 `skipHostKeyValidation` 設為 false，則為 [是]。  |
| gatewayName |要連線至內部部署 SFTP 伺服器的資料管理閘道名稱。 | 如果從內部部署 SFTP 伺服器複製資料，則為 [是]。 |
| encryptedCredential | 用來存取 SFTP 伺服器的加密認證。 當您在複製精靈或 ClickOnce 快顯對話方塊中指定基本驗證 (使用者名稱 + 密碼) 或 SshPublicKey 驗證 (使用者名稱 + 私密金鑰路徑或內容) 時自動產生。 | 否。 僅當從內部部署 SFTP 伺服器複製資料時才套用。 |

#### <a name="example-using-basic-authentication"></a>範例：使用基本驗證

若要使用基本驗證，將 `authenticationType` 設定為 `Basic`，然後指定上一節中介紹的 SFTP 連接器泛用以外的下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- | --- |
| username | 可存取 SFTP 伺服器的使用者。 |是 |
| password | 使用者 (使用者名稱) 的密碼。 | 是 |

```json
{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<SFTP server name or IP address>",
            "port": 22,
            "authenticationType": "Basic",
            "username": "xxx",
            "password": "xxx",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-basic-authentication-with-encrypted-credential"></a>範例：**採用加密認證的基本驗證**

```json
{
    "name": "SftpLinkedService",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<FTP server name or IP address>",
            "port": 22,
            "authenticationType": "Basic",
            "username": "xxx",
            "encryptedCredential": "xxxxxxxxxxxxxxxxx",
            "skipHostKeyValidation": false,
            "hostKeyFingerPrint": "ssh-rsa 2048 xx:00:00:00:xx:00:x0:0x:0x:0x:0x:00:00:x0:x0:00",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="using-ssh-public-key-authentication"></a>**使用 SSH 公用金鑰驗證：**

若要使用基本驗證，將 `authenticationType` 設定為 `SshPublicKey`，然後指定上一節中介紹的 SFTP 連接器泛用以外的下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- | --- |
| username |可存取 SFTP 伺服器的使用者 |是 |
| privateKeyPath | 指定閘道可以存取之私密金鑰檔案的絕對路徑。 | 指定 `privateKeyPath` 或 `privateKeyContent`。 <br><br> 僅當從內部部署 SFTP 伺服器複製資料時才套用。 |
| privateKeyContent | 私密金鑰內容的序列化字串。 複製精靈可以讀取私密金鑰檔案，並自動解壓縮私密金鑰內容。 如果您使用任何其他工具/SDK，請改為使用 privateKeyPath 屬性。 | 指定 `privateKeyPath` 或 `privateKeyContent`。 |
| passPhrase | 如果金鑰檔案受到複雜密碼保護，請指定複雜密碼/密碼以將私密金鑰解密。 | 如果私密金鑰檔案受到複雜密碼保護，則為 [是]。 |

```json
{
    "name": "SftpLinkedServiceWithPrivateKeyPath",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "<FTP server name or IP address>",
            "port": 22,
            "authenticationType": "SshPublicKey",
            "username": "xxx",
            "privateKeyPath": "D:\\privatekey_openssh",
            "passPhrase": "xxx",
            "skipHostKeyValidation": true,
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-sshpublickey-authentication-using-private-key-content"></a>範例：**使用私密金鑰內容的 SshPublicKey 驗證**

```json
{
    "name": "SftpLinkedServiceWithPrivateKeyContent",
    "properties": {
        "type": "Sftp",
        "typeProperties": {
            "host": "mysftpserver.westus.cloudapp.azure.com",
            "port": 22,
            "authenticationType": "SshPublicKey",
            "username": "xxx",
            "privateKeyContent": "<base64 string of the private key content>",
            "passPhrase": "xxx",
            "skipHostKeyValidation": true
        }
    }
}
```

如需詳細資訊，請參閱 [SFTP 連接器](data-factory-sftp-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 SFTP 資料集，請將資料集的 **type** 設為 **FileShare**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| folderPath |資料夾的子路徑。 使用逸出字元 ‘ \ ’ 當做字串中的特殊字元。 如需範例，請參閱 [範例連結服務和資料集定義](#sample-linked-service-and-dataset-definitions) 。<br/><br/>您可以結合此屬性與 **partitionBy**，讓資料夾路徑以配量開始/結束日期時間為基礎。 |是 |
| fileName |如果您想要資料表參考資料夾中的特定檔案，請指定 **folderPath** 中的檔案名稱。 如果沒有為此屬性指定任何值，資料表會指向資料夾中的所有檔案。<br/><br/>若未指定輸出資料集的 fileName，所產生檔案的名稱是下列格式︰ <br/><br/>Data.<Guid>.txt (範例：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |否 |
| fileFilter |指定要用來在 folderPath (而不是所有檔案) 中選取檔案子集的篩選器。<br/><br/>允許的值為︰`*` (多個字元) 和 `?` (單一字元)。<br/><br/>範例 1：`"fileFilter": "*.log"`<br/>範例 2：`"fileFilter": 2016-1-?.txt"`<br/><br/> fileFilter 適用於輸入 FileShare 資料集。 這個屬性不支援使用 HDFS。 |否 |
| partitionedBy |partitionedBy 可以用來指定時間序列資料的動態 folderPath 和 filename。 例如，folderPath 可針對每小時的資料進行參數化。 |否 |
| format | 支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 將格式下的 **type** 屬性設定為這些值其中之一。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 <br><br> 如果您想要在以檔案為基礎的存放區之間**依原樣複製檔案** (二進位複本)，請在輸入和輸出資料集定義中略過格式區段。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。 支援的層級為：**Optimal** 和 **Fastest**。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |
| useBinaryTransfer |指定是否使用二進位傳輸模式。 二進位模式為 true，ASCII 則為 false。 預設值：True。 只有在相關聯的連結服務類型的類型為 FtpServer 時，才可以使用這個屬性。 |否 |

> [!NOTE]
> 無法同時使用檔名和 fileFilter。

#### <a name="example"></a>範例

```json
{
    "name": "SFTPFileInput",
    "properties": {
        "type": "FileShare",
        "linkedServiceName": "SftpLinkedService",
        "typeProperties": {
            "folderPath": "<path to shared folder>",
            "fileName": "test.csv"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

如需詳細資訊，請參閱 [SFTP 連接器](data-factory-sftp-connector.md#dataset-properties)文件。 

### <a name="file-system-source-in-copy-activity"></a>複製活動中的檔案系統來源
如果您從 SFTP 來源複製資料，請將複製活動的 **source type** 設為 **FileSystemSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| 遞迴 |表示是否從子資料夾，或只有從指定的資料夾，以遞迴方式讀取資料。 |True/False (預設值為 False) |否 |



#### <a name="example"></a>範例

```json
{
    "name": "pipeline",
    "properties": {
        "activities": [{
            "name": "SFTPToBlobCopy",
            "inputs": [{
                "name": "SFTPFileInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "FileSystemSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "NewestFirst",
                "retry": 1,
                "timeout": "00:05:00"
            }
        }],
        "start": "2017-02-20T18:00:00",
        "end": "2017-02-20T19:00:00"
    }
}
```

如需詳細資訊，請參閱 [SFTP 連接器](data-factory-sftp-connector.md#copy-activity-properties)文件。


## <a name="http"></a>HTTP

### <a name="linked-service"></a>連結服務
若要定義 HTTP 連結服務，請將連結服務的 **type** 設為 **Http**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| url | Web 伺服器的基本 URL | 是 |
| authenticationType | 指定驗證類型。 允許的值為︰**匿名**、**基本**、**摘要**、**Windows**、**ClientCertificate**。 <br><br> 請分別參閱此關於更多屬性的下列資料表各節以及這些驗證類型的 JSON 範例。 | 是 |
| enableServerCertificateValidation | 如果來源是 HTTPS Web 伺服器，指定是否啟用伺服器 SSL 憑證驗證 | 否，預設值是 True |
| gatewayName | 連接至內部部署 HTTP 來源的「資料管理閘道」閘道。 | 如果從內部部署 HTTP 來源複製資料，則為是。 |
| encryptedCredential | 用來存取 HTTP 端點的加密認證。 當您在複製精靈或 ClickOnce 快顯對話方塊中設定驗證資訊時會自動產生。 | 否。 僅當從內部部署 HTTP 伺服器複製資料時才套用。 |

#### <a name="example-using-basic-digest-or-windows-authentication"></a>範例︰使用基本、摘要或 Windows 驗證
將 `authenticationType` 設定為 `Basic`、`Digest`或 `Windows`，並指定除了上面介紹的 HTTP 連接器泛用的下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| username | 存取 HTTP 端點的使用者名稱。 | 是 |
| password | 使用者 (使用者名稱) 的密碼。 | 是 |

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "Http",
        "typeProperties": {
            "authenticationType": "basic",
            "url": "https://en.wikipedia.org/wiki/",
            "userName": "user name",
            "password": "password"
        }
    }
}
```

#### <a name="example-using-clientcertificate-authentication"></a>範例：使用 ClientCertificate 驗證

若要使用基本驗證，將 `authenticationType` 設定為 `ClientCertificate`，並指定除了上面介紹的 HTTP 連接器泛用的下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| embeddedCertData | Base 64 編碼的個人資訊交換 (PFX) 檔案之二進位資料內容。 | 指定 `embeddedCertData` 或 `certThumbprint`。 |
| certThumbprint | 憑證指紋已安裝在您閘道器電腦的憑證存放區上。 僅當從內部部署 HTTP 來源複製資料時才套用。 | 指定 `embeddedCertData` 或 `certThumbprint`。 |
| password | 與憑證相關聯的密碼。 | 否 |

如果您使用 `certThumbprint` 進行驗證且憑證已安裝在本機電腦的個人存放區中，您必須授與讀取權限給閘道器服務︰

1. 啟動 Microsoft Management Console (MMC)。 新增目標為 [本機電腦] 的 [憑證] 嵌入式管理單元。
2. 展開 [憑證]，[個人]，然後按一下 [憑證]。
3. 以滑鼠右鍵按一下個人存放區中的 [憑證]，然後選取 [所有工作]->[管理私用金鑰...]
3. 在 [安全性] 索引標籤上，新增資料管理閘道主機服務使用憑證讀取存取執行所在的使用者帳戶。  

**範例：使用用戶端憑證：** 此連結服務會將您的資料處理站連結至內部部署 HTTP Web 伺服器。 它會使用已安裝資料管理閘道的電腦上所安裝的用戶端憑證。

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "Http",
        "typeProperties": {
            "authenticationType": "ClientCertificate",
            "url": "https://en.wikipedia.org/wiki/",
            "certThumbprint": "thumbprint of certificate",
            "gatewayName": "gateway name"
        }
    }
}
```

#### <a name="example-using-client-certificate-in-a-file"></a>範例︰在檔案中使用用戶端憑證
此連結服務會將您的資料處理站與內部部署 HTTP web 伺服器連結。 它會使用已安裝資料管理閘道的電腦上的用戶端憑證檔案。

```json
{
    "name": "HttpLinkedService",
    "properties": {
        "type": "Http",
        "typeProperties": {
            "authenticationType": "ClientCertificate",
            "url": "https://en.wikipedia.org/wiki/",
            "embeddedCertData": "base64 encoded cert data",
            "password": "password of cert"
        }
    }
}
```

如需詳細資訊，請參閱 [HTTP 連接器](data-factory-http-connector.md#linked-service-properties)文件。

### <a name="dataset"></a>Dataset
若要定義 HTTP 資料集，請將資料集的 **type** 設為 **Http**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| relativeUrl | 包含資料之資源的相對 URL。 當路徑未指定時，則只會使用在連結服務定義中指定的 URL。 <br><br> 若要建構動態 URL，您可以使用 [Data Factory 函式和系統變數](data-factory-functions-variables.md)，範例︰`"relativeUrl": "$$Text.Format('/my/report?month={0:yyyy}-{0:MM}&fmt=csv', SliceStart)"`。 | 否 |
| requestMethod | HTTP 方法。 允許的值為 **GET** 或 **POST**。 | 否。 預設值為 `GET`。 |
| additionalHeaders | 其他 HTTP 要求標頭。 | 否 |
| requestBody | HTTP 要求的內文。 | 否 |
| format | 如果您只想要**從 HTTP 端點依現狀擷取資料**而不剖析它，請略過此格式設定。 <br><br> 如果您想要在複製期間剖析 HTTP 回應內容，支援下列格式類型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat**、**ParquetFormat**。 如需詳細資訊，請參閱[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)章節。 |否 |
| compression | 指定此資料的壓縮類型和層級。 支援的類型為：**GZip**、**Deflate**、**BZip2** 及 **ZipDeflate**。 支援的層級為：**Optimal** 和 **Fastest**。 如需詳細資訊，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |

#### <a name="example-using-the-get-default-method"></a>範例︰使用 GET (預設值) 方法

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "XXX/test.xml",
            "additionalHeaders": "Connection: keep-alive\nUser-Agent: Mozilla/5.0\n"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

#### <a name="example-using-the-post-method"></a>範例︰使用 POST 方法

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "/XXX/test.xml",
            "requestMethod": "Post",
            "requestBody": "body for POST HTTP request"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```
如需詳細資訊，請參閱 [HTTP 連接器](data-factory-http-connector.md#dataset-properties)文件。

### <a name="http-source-in-copy-activity"></a>複製活動中的 HTTP 來源
如果您從 HTTP 來源複製資料，請將複製活動的 **source type** 設為 **HttpSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- |
| httpRequestTimeout | HTTP 的逾時 (TimeSpan) 要求取得回應。 逾時會取得回應，而非逾時讀取回應資料。 | 否。 預設值：00:01:40 |


#### <a name="example"></a>範例

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "HttpSourceToAzureBlob",
            "description": "Copy from an HTTP source to an Azure blob",
            "type": "Copy",
            "inputs": [{
                "name": "HttpSourceDataInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "HttpSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱 [HTTP 連接器](data-factory-http-connector.md#copy-activity-properties)文件。

## <a name="odata"></a>OData

### <a name="linked-service"></a>連結服務
若要定義 OData 連結服務，請將連結服務的 **type** 設為 **OData**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| url |OData 服務的 URL。 |是 |
| authenticationType |用來連線到 OData 來源的驗證類型。 <br/><br/> 若為雲端 OData，可能的值為 Anonymous、Basic 和 OAuth (請注意，Azure Data Factory 目前僅支援 Azure Active Directory 架構的 OAuth)。 <br/><br/> 若為內部部署 OData，可能的值為 Anonymous、Basic 和 Windows。 |是 |
| username |如果您要使用 Basic 驗證，請指定使用者名稱。 |是 (只在您使用基本驗證時) |
| password |指定您為使用者名稱所指定之使用者帳戶的密碼。 |是 (只在您使用基本驗證時) |
| authorizedCredential |如果您使用 OAuth，按一下 Data Factory 複製精靈或編輯器中的 [授權] 按鈕，然後輸入您的認證，接著將會自動產生這個屬性的值。 |是 (只有在您使用 OAuth 驗證時) |
| gatewayName |Data Factory 服務應該用來連接到內部部署 OData 服務的閘道器名稱。 只在要從內部部署 OData 來源複製資料時才指定。 |否 |

#### <a name="example---using-basic-authentication"></a>範例 - 使用基本驗證
```json
{
    "name": "inputLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "http://services.odata.org/OData/OData.svc",
            "authenticationType": "Basic",
            "username": "username",
            "password": "password"
        }
    }
}
```

#### <a name="example---using-anonymous-authentication"></a>範例 - 使用匿名驗證

```json
{
    "name": "ODataLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "http://services.odata.org/OData/OData.svc",
            "authenticationType": "Anonymous"
        }
    }
}
```

#### <a name="example---using-windows-authentication-accessing-on-premises-odata-source"></a>範例 - 使用 Windows 驗證存取內部部署 OData 來源

```json
{
    "name": "inputLinkedService",
    "properties": {
        "type": "OData",
        "typeProperties": {
            "url": "<endpoint of on-premises OData source, for example, Dynamics CRM>",
            "authenticationType": "Windows",
            "username": "domain\\user",
            "password": "password",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example---using-oauth-authentication-accessing-cloud-odata-source"></a>範例 - 使用 OAuth 驗證存取雲端 OData 來源
```json
{
    "name": "inputLinkedService",
    "properties":
    {
        "type": "OData",
            "typeProperties":
        {
            "url": "<endpoint of cloud OData source, for example, https://<tenant>.crm.dynamics.com/XRMServices/2011/OrganizationData.svc>",
            "authenticationType": "OAuth",
            "authorizedCredential": "<auto generated by clicking the Authorize button on UI>"
        }
    }
}
```

如需詳細資訊，請參閱 [OData 連接器](data-factory-odata-connector.md#linked-service-properties)文件。

### <a name="dataset"></a>Dataset
若要定義 OData 資料集，請將資料集的 **type** 設為 **ODataResource**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| path |OData 資源的路徑 |否 |

#### <a name="example"></a>範例

```json
{
    "name": "ODataDataset",
    "properties": {
        "type": "ODataResource",
        "typeProperties": {
            "path": "Products"
        },
        "linkedServiceName": "ODataLinkedService",
        "structure": [],
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "retryInterval": "00:01:00",
            "retryTimeout": "00:10:00",
            "maximumRetry": 3
        }
    }
}
```

如需詳細資訊，請參閱 [OData 連接器](data-factory-odata-connector.md#dataset-properties)文件。

### <a name="relational-source-in-copy-activity"></a>複製活動中的關聯式來源
如果您從 OData 來源複製資料，請將複製活動的 **source type** 設為 **RelationalSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 範例 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |「?$select=Name, Description&$top=5」 |否 |

#### <a name="example"></a>範例

```json
{
    "name": "CopyODataToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "?$select=Name, Description&$top=5"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "ODataDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobODataDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "ODataToBlob"
        }],
        "start": "2017-02-01T18:00:00",
        "end": "2017-02-03T19:00:00"
    }
}
```

如需詳細資訊，請參閱 [OData 連接器](data-factory-odata-connector.md#copy-activity-properties)文件。


## <a name="odbc"></a>ODBC


### <a name="linked-service"></a>連結服務
若要定義 ODBC 連結服務，請將連結服務的 **type** 設為 **OnPremisesOdbc**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| connectionString |連接字串的非存取認證部分和選擇性的加密認證。 請參閱下列幾節中的範例。 |是 |
| 認證 |以驅動程式特定「屬性-值」格式指定之連接字串的存取認證部分。 範例：“Uid=<user ID>;Pwd=<password>;RefreshToken=<secret refresh token>;”。 |否 |
| authenticationType |用來連接到 ODBC 資料存放區的驗證類型。 可能的值為：Anonymous 和 Basic。 |是 |
| username |如果您要使用 Basic 驗證，請指定使用者名稱。 |否 |
| password |指定您為使用者名稱所指定之使用者帳戶的密碼。 |否 |
| gatewayName |Data Factory 服務應該用來連接到 ODBC 資料存放區的閘道器名稱。 |是 |

#### <a name="example---using-basic-authentication"></a>範例 - 使用基本驗證

```json
{
    "name": "ODBCLinkedService",
    "properties": {
        "type": "OnPremisesOdbc",
        "typeProperties": {
            "authenticationType": "Basic",
            "connectionString": "Driver={SQL Server};Server=Server.database.windows.net; Database=TestDatabase;",
            "userName": "username",
            "password": "password",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```
#### <a name="example---using-basic-authentication-with-encrypted-credentials"></a>範例 - 使用基本驗證搭配加密認證
您可以使用 [New-AzureRMDataFactoryEncryptValue](https://docs.microsoft.com/powershell/module/azurerm.datafactories/new-azurermdatafactoryencryptvalue) (1.0 版的 Azure PowerShell) Cmdlet 或 [New-AzureDataFactoryEncryptValue](https://msdn.microsoft.com/library/dn834940.aspx) (0.9 或更舊版的 Azure PowerShell) 來加密認證。  

```json
{
    "name": "ODBCLinkedService",
    "properties": {
        "type": "OnPremisesOdbc",
        "typeProperties": {
            "authenticationType": "Basic",
            "connectionString": "Driver={SQL Server};Server=myserver.database.windows.net; Database=TestDatabase;;EncryptedCredential=eyJDb25uZWN0...........................",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

#### <a name="example-using-anonymous-authentication"></a>範例：使用匿名驗證

```json
{
    "name": "ODBCLinkedService",
    "properties": {
        "type": "OnPremisesOdbc",
        "typeProperties": {
            "authenticationType": "Anonymous",
            "connectionString": "Driver={SQL Server};Server={servername}.database.windows.net; Database=TestDatabase;",
            "credential": "UID={uid};PWD={pwd}",
            "gatewayName": "<onpremgateway>"
        }
    }
}
```

如需詳細資訊，請參閱 [ODBC 連接器](data-factory-odbc-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 ODBC 資料集，請將資料集的 **type** 設為 **RelationalTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |ODBC 資料存放區中資料表的名稱。 |是 |


#### <a name="example"></a>範例

```json
{
    "name": "ODBCDataSet",
    "properties": {
        "type": "RelationalTable",
        "linkedServiceName": "ODBCLinkedService",
        "typeProperties": {},
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

如需詳細資訊，請參閱 [ODBC 連接器](data-factory-odbc-connector.md#dataset-properties)文件。 

### <a name="relational-source-in-copy-activity"></a>複製活動中的關聯式來源
如果您從 ODBC 資料存放區複製資料，請將複製活動的 **source type** 設為 **RelationalSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL 查詢字串。 例如： `select * from MyTable` 。 |是 |

#### <a name="example"></a>範例

```json
{
    "name": "CopyODBCToBlob",
    "properties": {
        "description": "pipeline for copy activity",
        "activities": [{
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "$$Text.Format('select * from MyTable where timestamp >= \\'{0:yyyy-MM-ddTHH:mm:ss}\\' AND timestamp < \\'{1:yyyy-MM-ddTHH:mm:ss}\\'', WindowStart, WindowEnd)"
                },
                "sink": {
                    "type": "BlobSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:00"
                }
            },
            "inputs": [{
                "name": "OdbcDataSet"
            }],
            "outputs": [{
                "name": "AzureBlobOdbcDataSet"
            }],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "OdbcToBlob"
        }],
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00"
    }
}
``` 

如需詳細資訊，請參閱 [ODBC 連接器](data-factory-odbc-connector.md#copy-activity-properties)文件。

## <a name="salesforce"></a>Salesforce


### <a name="linked-service"></a>連結服務
若要定義 Salesforce 連結服務，請將連結服務的 **type** 設為 **Salesforce**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| environmentUrl | 指定 Salesforce 執行個體的 URL。 <br><br> - 預設為「 https://login.salesforce.com 」。 <br> - 若要從沙箱複製資料，請指定「 https://test.salesforce.com 」。 <br> - 若要從自訂網域複製資料，舉例來說，請指定 "https://[網域].my.salesforce.com"。 |否 |
| username |指定使用者帳戶的使用者名稱。 |是 |
| password |指定使用者帳戶的密碼。 |是 |
| securityToken |指定使用者帳戶的安全性權杖。 如需如何重設/取得安全性權杖的指示，請參閱 [取得安全性權杖](https://help.salesforce.com/apex/HTViewHelpDoc?id=user_security_token.htm) 。 若要整體了解安全性權杖，請參閱[安全性和 API](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_concepts_security.htm)。 |是 |

#### <a name="example"></a>範例

```json
{
    "name": "SalesforceLinkedService",
    "properties": {
        "type": "Salesforce",
        "typeProperties": {
            "username": "<user name>",
            "password": "<password>",
            "securityToken": "<security token>"
        }
    }
}
```

如需詳細資訊，請參閱 [Salesforce 連接器](data-factory-salesforce-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 Salesforce 資料集，請將資料集的 **type** 設為 **RelationalTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| tableName |Salesforce 中資料表的名稱。 |否 (如果已指定 **RelationalSource** 的 **query**) |

#### <a name="example"></a>範例

```json
{
    "name": "SalesforceInput",
    "properties": {
        "linkedServiceName": "SalesforceLinkedService",
        "type": "RelationalTable",
        "typeProperties": {
            "tableName": "AllDataType__c"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "external": true,
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

如需詳細資訊，請參閱 [Salesforce 連接器](data-factory-salesforce-connector.md#dataset-properties)文件。 

### <a name="relational-source-in-copy-activity"></a>複製活動中的關聯式來源
如果您從 Salesforce 複製資料，請將複製活動的 **source type** 設為 **RelationalSource**，並在 **source** 區段中指定下列屬性︰

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| query |使用自訂查詢來讀取資料。 |SQL-92 查詢或 [Salesforce 物件查詢語言 (SOQL)](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql.htm) 查詢。 例如：`select * from MyTable__c`。 |否 (如果已指定 **dataset** 的 **tableName**) |

#### <a name="example"></a>範例  



```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "SalesforceToAzureBlob",
            "description": "Copy from Salesforce to an Azure blob",
            "type": "Copy",
            "inputs": [{
                "name": "SalesforceInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "RelationalSource",
                    "query": "SELECT Id, Col_AutoNumber__c, Col_Checkbox__c, Col_Currency__c, Col_Date__c, Col_DateTime__c, Col_Email__c, Col_Number__c, Col_Percent__c, Col_Phone__c, Col_Picklist__c, Col_Picklist_MultiSelect__c, Col_Text__c, Col_Text_Area__c, Col_Text_AreaLong__c, Col_Text_AreaRich__c, Col_URL__c, Col_Text_Encrypt__c, Col_Lookup__c FROM AllDataType__c"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

> [!IMPORTANT]
> 任何自訂物件都需要 API 名稱的「__c」部分。

如需詳細資訊，請參閱 [Salesforce 連接器](data-factory-salesforce-connector.md#copy-activity-properties)文件。 

## <a name="web-data"></a>Web 資料 

### <a name="linked-service"></a>連結服務
若要定義 Web 連結服務，請將連結服務的 **type** 設為 **Web**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| Url |Web 來源的 URL |是 |
| authenticationType |匿名。 |是 |
 

#### <a name="example"></a>範例


```json
{
    "name": "web",
    "properties": {
        "type": "Web",
        "typeProperties": {
            "authenticationType": "Anonymous",
            "url": "https://en.wikipedia.org/wiki/"
        }
    }
}
```

如需詳細資訊，請參閱 [Web 資料表連接器](data-factory-web-table-connector.md#linked-service-properties)文件。 

### <a name="dataset"></a>Dataset
若要定義 Web 資料集，請將資料集的 **type** 設為 **WebTable**，並在 **typeProperties** 區段中指定下列屬性︰ 

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| type |資料集的類型。 必須設定為 **WebTable** |是 |
| path |包含資料表之資源的相對 URL。 |否。 當路徑未指定時，則只會使用在連結服務定義中指定的 URL。 |
| index |資源中資料表的索引。 如需如何取得 HTML 網頁中資料表索引的步驟，請參閱 [取得 HTML 網頁中資料表的索引](#get-index-of-a-table-in-an-html-page) 一節。 |是 |

#### <a name="example"></a>範例

```json
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "linkedServiceName": "WebLinkedService",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

如需詳細資訊，請參閱 [Web 資料表連接器](data-factory-web-table-connector.md#dataset-properties)文件。 

### <a name="web-source-in-copy-activity"></a>複製活動中的 Web 來源
如果您從 Web 資料表複製資料，請將複製活動的 **source type** 設為 **WebSource**。 當複製活動中的來源類型為 **WebSource**，目前並未支援任何其他屬性。

#### <a name="example"></a>範例

```json
{
    "name": "SamplePipeline",
    "properties": {
        "start": "2016-06-01T18:00:00",
        "end": "2016-06-01T19:00:00",
        "description": "pipeline with copy activity",
        "activities": [{
            "name": "WebTableToAzureBlob",
            "description": "Copy from a Web table to an Azure blob",
            "type": "Copy",
            "inputs": [{
                "name": "WebTableInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
            "typeProperties": {
                "source": {
                    "type": "WebSource"
                },
                "sink": {
                    "type": "BlobSink"
                }
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "policy": {
                "concurrency": 1,
                "executionPriorityOrder": "OldestFirst",
                "retry": 0,
                "timeout": "01:00:00"
            }
        }]
    }
}
```

如需詳細資訊，請參閱 [Web 資料表連接器](data-factory-web-table-connector.md#copy-activity-properties)文件。 

## <a name="compute-environments"></a>計算環境
下表列出 Data Factory 支援的計算環境和可在這些環境上執行的轉換活動。 按一下您感興趣的計算連結，查看連結服務的 JSON 結構描述，以將它連結至資料處理站。 

| 計算環境 | 活動 |
| --- | --- |
| [隨選 HDInsight 叢集](#on-demand-azure-hdinsight-cluster)或[您自己的 HDInsight 叢集](#existing-azure-hdinsight-cluster) |[.NET 自訂活動](#net-custom-activity)、[Hive 活動](#hdinsight-hive-activity)、[Pig 活動](#hdinsight-pig-activity)、[MapReduce 活動](#hdinsight-mapreduce-activity)、[Hadoop 串流活動](#hdinsight-streaming-activityd)、[Spark 活動](#hdinsight-spark-activity) |
| [Azure Batch](#azure-batch) |[.NET 自訂活動](#net-custom-activity) |
| [Azure Machine Learning](#azure-machine-learning) | [Machine Learning 批次執行活動](#machine-learning-batch-execution-activity)、[Machine Learning 更新資源活動](#machine-learning-update-resource-activity) |
| [Azure Data Lake Analytics](#azure-data-lake-analytics) |[Data Lake Analytics U-SQL](#data-lake-analytics-u-sql-activity) |
| [Azure SQL Database](#azure-sql-database-1)、[Azure SQL 資料倉儲](#azure-sql-data-warehouse-1)、[SQL Server](#sql-server-1) |[預存程序](#stored-procedure-activity) |

## <a name="on-demand-azure-hdinsight-cluster"></a>隨選 Azure HDInsight 叢集
Azure Data Factory 服務可自動建立以 Windows/Linux 為基礎的隨選 HDInsight 叢集來處理資料。 此叢集會建立在與叢集相關聯的儲存體帳戶 (JSON 中的 linkedServiceName 屬性) 相同的區域中。 您可以在此連結服務上執行下列轉換活動：[.NET 自訂活動](#net-custom-activity)、[Hive 活動](#hdinsight-hive-activity)、[Pig 活動](#hdinsight-pig-activity)、[MapReduce 活動](#hdinsight-mapreduce-activity)、[Hadoop 串流活動](#hdinsight-streaming-activityd)、[Spark 活動](#hdinsight-spark-activity)。 

### <a name="linked-service"></a>連結服務 
下表描述隨選 HDInsight 連結服務的 Azure JSON 定義中所使用的屬性。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |type 屬性應設為 **HDInsightOnDemand**。 |是 |
| clusterSize |叢集中的背景工作/資料節點數。 HDInsight 叢集會利用您為此屬性指定的 2 個前端節點以及背景工作節點數目來建立。 節點大小為具有 4 個核心的 Standard_D3，因此 4 個背景工作節點的叢集需要 24 個核心 (4\*4 = 16 個核心用於背景工作節點，加上 2\*4 = 8 個核心用於前端節點)。 如需 Standard_D3 層的詳細資料，請參閱[在 HDInsight 中建立 Linux 型 Hadoop 叢集](../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md)。 |是 |
| timetolive |隨選 HDInsight 叢集允許的閒置時間。 指定在活動執行完成後，如果叢集中沒有其他作用中的作業，隨選 HDInsight 叢集要保持運作多久。<br/><br/>例如，如果活動執行花費 6 分鐘，而 timetolive 設為 5 分鐘，叢集會在處理活動執行的 6 分鐘期間之後保持運作 5 分鐘。 如果 6 分鐘期間內執行了另一個活動執行，它便會由相同叢集來處理。<br/><br/>建立隨選 HDInsight 叢集是昂貴的作業 (可能需要一段時間)，因此請視需要使用這項設定，重複使用隨選 HDInsight 叢集以改善 Data Factory 的效能。<br/><br/>如果您將 timetolive 值設為 0，叢集會在處理活動執行後立即刪除。 另一方面，如果您設定較高的值，叢集可能會有不必要的閒置而導致高成本。 因此，請務必根據您的需求設定適當的值。<br/><br/>如果適當地設定 timetolive 屬性值，則多個管線可以共用相同的隨選 HDInsight 叢集執行個體 |是 |
| version |HDInsight 叢集的版本。 如需詳細資訊，請參閱 [Azure Data Factory 中支援的 HDInsight 版本](data-factory-compute-linked-services.md#supported-hdinsight-versions-in-azure-data-factory)。 |否 |
| 預設容器 |隨選叢集用於儲存及處理資料的 Azure 儲存體連結服務。 <p>目前，您無法建立使用 Azure Data Lake Store 做為儲存體的隨選 HDInsight 叢集。 如果您想要在 Azure Data Lake Store 中儲存 HDInsight 處理的結果資料，可使用複製活動將 Azure Blob 儲存體的資料複製到 Azure Data Lake Store。</p>  | 是 |
| additionalLinkedServiceNames |指定 HDInsight 連結服務的其他儲存體帳戶，讓 Data Factory 服務代表您註冊它們。 |否 |
| osType |作業系統的類型。 允許的值為：Windows (預設值) 和 linux |否 |
| hcatalogLinkedServiceName |指向 HCatalog 資料庫的 Azure SQL 連結服務名稱。 會使用 Azure SQL 資料庫作為中繼存放區，建立隨選 HDInsight 叢集。 |否 |

### <a name="json-example"></a>JSON 範例
下列 JSON 會定義以 Linux 為基礎的隨選 HDInsight 連結服務。 Data Factory 服務會在處理資料配量時自動建立 **以 Linux 為基礎的** HDInsight 叢集。 

```json
{
    "name": "HDInsightOnDemandLinkedService",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "StorageLinkedService"
        }
    }
}
```

如需詳細資訊，請參閱[計算連結服務](data-factory-compute-linked-services.md)一文。 

## <a name="existing-azure-hdinsight-cluster"></a>現有的 Azure HDInsight 叢集
您可以建立 Azure HDInsight 連結服務，以向 Data Factory 註冊自己的 HDInsight 叢集。 您可以在此連結服務上執行下列資料轉換活動：[.NET 自訂活動](#net-custom-activity)、[Hive 活動](#hdinsight-hive-activity)、[Pig 活動](#hdinsight-pig-activity)、[MapReduce 活動](#hdinsight-mapreduce-activity)、[Hadoop 串流活動](#hdinsight-streaming-activityd)、[Spark 活動](#hdinsight-spark-activity)。 

### <a name="linked-service"></a>連結服務
下表描述 Azure HDInsight 連結服務的 Azure JSON 定義中所使用的屬性。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |type 屬性應設為 **HDInsight**。 |是 |
| clusterUri |HDInsight 叢集的 URI。 |是 |
| username |指定要用來連接到現有 HDInsight 叢集的使用者名稱。 |是 |
| password |指定使用者帳戶的密碼。 |是 |
| 預設容器 | 參照 HDInsight 叢集所使用 Azure Blob 儲存體的 Azure 儲存體連結服務名稱。 <p>目前，您無法針對此屬性指定 Azure Data Lake Store 連結服務。 如果 HDInsight 叢集可存取 Data Lake Store，您可以透過 Hive/Pig 指令碼存取 Azure Data Lake Store 中的資料。 </p>  |是 |

如需支援的 HDInsight 叢集版本，請參閱[支援的 HDInsight 版本](data-factory-compute-linked-services.md#supported-hdinsight-versions-in-azure-data-factory)。 

#### <a name="json-example"></a>JSON 範例

```json
{
    "name": "HDInsightLinkedService",
    "properties": {
        "type": "HDInsight",
        "typeProperties": {
            "clusterUri": " https://<hdinsightclustername>.azurehdinsight.net/",
            "userName": "admin",
            "password": "<password>",
            "linkedServiceName": "MyHDInsightStoragelinkedService"
        }
    }
}
```

## <a name="azure-batch"></a>Azure Batch
您可以建立 Azure Batch 連結服務，以向資料處理站註冊虛擬機器 (VM) 的 Batch 集區。 您可以使用 Azure Batch 或 Azure HDInsight 執行 .NET 自訂活動。 您可以在此連結服務上執行 [.NET 自訂活動](#net-custom-activity)。 

### <a name="linked-service"></a>連結服務
下表描述 Azure Batch 連結服務的 Azure JSON 定義中所使用的屬性。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |type 屬性應設為 **AzureBatch**。 |是 |
| accountName |建立 Azure Batch 帳戶。 |是 |
| accessKey |Azure Batch 帳戶的存取金鑰。 |是 |
| poolName |虛擬機器的集區名稱。 |是 |
| 預設容器 |與此 Azure Batch 連結服務相關聯的 Azure 儲存體服務連結名稱。 此連結服務用於執行活動及儲存活動執行記錄檔所需的暫存檔案。 |是 |


#### <a name="json-example"></a>JSON 範例

```json
{
    "name": "AzureBatchLinkedService",
    "properties": {
        "type": "AzureBatch",
        "typeProperties": {
            "accountName": "<Azure Batch account name>",
            "accessKey": "<Azure Batch account key>",
            "poolName": "<Azure Batch pool name>",
            "linkedServiceName": "<Specify associated storage linked service reference here>"
        }
    }
}
```

## <a name="azure-machine-learning"></a>Azure Machine Learning
您可建立 Azure Machine Learning 連結服務，以向資料處理站註冊 Machine Learning 批次評分端點。 您可以在此連結服務上執行兩個資料轉換活動︰[Machine Learning 批次執行活動](#machine-learning-batch-execution-activity)、[Machine Learning 更新資源活動](#machine-learning-update-resource-activity)。 

### <a name="linked-service"></a>連結服務
下表描述 Azure Machine Learning 連結服務的 Azure JSON 定義中所使用的屬性。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| 類型 |type 屬性應設為： **AzureML**。 |是 |
| mlEndpoint |批次評分 URL。 |是 |
| apiKey |已發佈的工作區模型的 API。 |是 |

#### <a name="json-example"></a>JSON 範例

```json
{
    "name": "AzureMLLinkedService",
    "properties": {
        "type": "AzureML",
        "typeProperties": {
            "mlEndpoint": "https://[batch scoring endpoint]/jobs",
            "apiKey": "<apikey>"
        }
    }
}
```

## <a name="azure-data-lake-analytics"></a>Azure Data Lake Analytics
您應建立 **Azure Data Lake Analytics** 連結服務，將 Azure Data Lake Analytics 計算服務連結至 Azure Data Factory，然後再使用管線中的 [Data Lake Analytics U-SQL 活動](data-factory-usql-activity.md) 。

### <a name="linked-service"></a>連結服務

下表描述 Azure Data Lake Analytics 連結服務的 JSON 定義中所使用的屬性。 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| 類型 |type 屬性應設為： **AzureDataLakeAnalytics**。 |是 |
| accountName |Azure Data Lake Analytics 帳戶名稱。 |是 |
| dataLakeAnalyticsUri |Azure Data Lake Analytics URI。 |否 |
| 授權 |按一下 Data Factory 編輯器中的 [授權]  按鈕並完成 OAuth 登入後，即會自動擷取授權碼。 |是 |
| subscriptionId |Azure 訂用帳戶識別碼 |否 (如果未指定，便會使用 Data Factory 的訂用帳戶)。 |
| resourceGroupName |Azure 資源群組名稱 |否 (若未指定，便會使用 Data Factory 的資源群組)。 |
| sessionId |OAuth 授權工作階段的工作階段識別碼。 每個工作階段識別碼都是唯一的，只能使用一次。 使用 Data Factory 編輯器時會自動產生此識別碼。 |是 |


#### <a name="json-example"></a>JSON 範例
下列範例提供 Azure Data Lake Analytics 連結服務的 JSON 定義。

```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "<account name>",
            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
            "authorization": "<authcode>",
            "sessionId": "<session ID>",
            "subscriptionId": "<subscription id>",
            "resourceGroupName": "<resource group name>"
        }
    }
}
```

## <a name="azure-sql-database"></a>連接字串
您可建立 Azure SQL 連結服務，並將其與 [預存程序活動](#stored-procedure-activity) 搭配使用，以叫用 Data Factory 管線中的預存程序。 

### <a name="linked-service"></a>連結服務
若要定義 Azure SQL Database 連結服務，請將連結服務的 **type** 設為 **AzureSqlDatabase**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| connectionString |針對 connectionString 屬性指定連接到 Azure SQL Database 執行個體所需的資訊。 |是 |

#### <a name="json-example"></a>JSON 範例

```json
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "type": "AzureSqlDatabase",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        }
    }
}
```

如需此連結服務的詳細資料，請參閱 [Azure SQL 連接器](data-factory-azure-sql-connector.md#linked-service-properties) 一文。

## <a name="azure-sql-data-warehouse"></a>Azure SQL 資料倉儲
您可以建立 Azure SQL 資料倉儲連結服務，並將其與 [預存程序活動](data-factory-stored-proc-activity.md) 搭配使用，以叫用 Data Factory 管線中的預存程序。 

### <a name="linked-service"></a>連結服務
若要定義 Azure SQL 資料倉儲連結服務，請將連結服務的 **type** 設為 **AzureSqlDW**，並在 **typeProperties** 區段中指定下列屬性︰  

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| connectionString |針對 connectionString 屬性指定連線到 Azure SQL 資料倉儲執行個體所需的資訊。 |是 |

#### <a name="json-example"></a>JSON 範例

```json
{
    "name": "AzureSqlDWLinkedService",
    "properties": {
        "type": "AzureSqlDW",
        "typeProperties": {
            "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
        }
    }
}
```

如需詳細資訊，請參閱 [Azure SQL 資料倉儲連接器](data-factory-azure-sql-data-warehouse-connector.md#linked-service-properties)文件。 

## <a name="sql-server"></a>SQL Server 
您可以建立 SQL Server 連結服務，並將其與 [預存程序活動](data-factory-stored-proc-activity.md) 搭配使用，以叫用 Data Factory 管線中的預存程序。 

### <a name="linked-service"></a>連結服務
您可以建立 **OnPremisesSqlServer** 類型的連結服務，以將內部部署 SQL Server 資料庫連結至資料處理站。 下表提供內部部署 SQL Server 連結服務專屬 JSON 元素的描述。

下表提供 SQL Server 連結服務專屬 JSON 元素的描述。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| type |類型屬性應設為： **OnPremisesSqlServer**。 |是 |
| connectionString |指定使用 SQL 驗證或 Windows 驗證連接至內部部署 SQL Server 資料庫所需的 connectionString 資訊。 |是 |
| gatewayName |Data Factory 服務應該用來連接到內部部署 SQL Server 資料庫的閘道器名稱。 |是 |
| username |如果您使用「Windows 驗證」，請指定使用者名稱。 範例︰**domainname\\username**。 |否 |
| password |指定您為使用者名稱所指定之使用者帳戶的密碼。 |否 |

您可以使用 **New-AzureRmDataFactoryEncryptValue** Cmdlet 加密認證，並在連接字串中使用這些認證，如下列範例所示 (**EncryptedCredential** 屬性)：  

```JSON
"connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=True;EncryptedCredential=<encrypted credential>",
```


#### <a name="example-json-for-using-sql-authentication"></a>範例：用於 SQL 驗證的 JSON

```json
{
    "name": "MyOnPremisesSQLDB",
    "properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "connectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=False;User ID=<username>;Password=<password>;",
            "gatewayName": "<gateway name>"
        }
    }
}
```
#### <a name="example-json-for-using-windows-authentication"></a>範例：用於 Windows 驗證的 JSON

如果已指定使用者名稱和密碼，閘道就會使用它們來模擬指定的使用者帳戶，以連線到內部部署 SQL Server 資料庫。 否則，閘道會使用閘道的安全性內容 (其啟動帳戶) 直接連線到 SQL Server。

```json
{
    "Name": " MyOnPremisesSQLDB",
    "Properties": {
        "type": "OnPremisesSqlServer",
        "typeProperties": {
            "ConnectionString": "Data Source=<servername>;Initial Catalog=MarketingCampaigns;Integrated Security=True;",
            "username": "<domain\\username>",
            "password": "<password>",
            "gatewayName": "<gateway name>"
        }
    }
}
```

如需詳細資訊，請參閱 [SQL Server 連接器](data-factory-sqlserver-connector.md#linked-service-properties)文件。

## <a name="data-transformation-activities"></a>資料轉換活動

活動 | 說明
-------- | -----------
[HDInsight Hive 活動](#hdinsight-hive-activity) | Data Factory 管線中的 HDInsight Hive 活動會在您自己或隨選的 Windows/Linux 架構 HDInsight 叢集上執行 Hive 查詢。 
[HDInsight Pig 活動](#hdinsight-pig-activity) | Data Factory 管線中的 HDInsight Pig 活動會在您自己或隨選的 Windows/Linux 架構 HDInsight 叢集上執行 Pig 查詢。
[HDInsight MapReduce 活動](#hdinsight-mapreduce-activity) | Data Factory 管線中的 HDInsight MapReduce 活動會在您自己或隨選的 Windows/Linux 架構 HDInsight 叢集上執行 MapReduce 程式。
[HDInsight 串流活動](#hdinsight-streaming-activity) | Data Factory 管線中的 HDInsight 串流活動會在您自己或隨選的 Windows/Linux 架構 HDInsight 叢集上執行 Hadoop 串流程式。
[HDInsight Spark 活動](#hdinsight-spark-activity) | Data Factory 管線中的 HDInsight Spark 活動會在您自己的 HDInsight 叢集上執行 Spark 程式。 
[Machine Learning Batch 執行活動](#machine-learning-batch-execution-activity) | Azure Data Factory 可讓您輕鬆地建立管線，使用已發佈的 Azure Machine Learning Web 服務進行預測性分析。 在 Azure Data Factory 管線中使用批次執行活動，您可以叫用 Machine Learning Web 服務來對批次中的資料進行預測。 
[Machine Learning 更新資源活動](#machine-learning-update-resource-activity) | 經過一段時間，必須使用新的輸入資料集重新訓練 Machine Learning 評分實驗中的預測模型。 完成重新訓練之後，您想要使用已重新訓練的 Machine Learning 模型來更新評分 Web 服務。 您可以使用更新資源活動，以新訓練的模型更新 Web 服務。
[預存程序活動](#stored-procedure-activity) | 您可以在 Data Factory 管線中使用預存程序活動，以叫用下列其中一個資料存放區中的預存程序：您的企業或 Azure VM 中的 Azure SQL Database、Azure SQL 資料倉儲、SQL Server 資料庫。 
[Data Lake Analytics U-SQL 活動](#data-lake-analytics-u-sql-activity) | Data Lake Analytics U-SQL 活動會在 Azure Data Lake Analytics 叢集上執行 U-SQL 指令碼。  
[.NET 自訂活動](#net-custom-activity) | 如果您需要以 Data Factory 不支援的方法轉換資料，可以利用自己的資料處理邏輯建立自訂活動，然後在管線中使用活動。 您可以將自訂 .NET 活動設定為使用 Azure Batch 服務或 Azure HDInsight 叢集來執行。 

     
## <a name="hdinsight-hive-activity"></a>HDInsight Hive 活動
您可以在 Hive 活動 JSON 定義中指定下列屬性。 活動的 type 屬性必須是︰**HDInsightHive**。 您必須先建立 HDInsight 連結服務，再指定它的名稱作為 **linkedServiceName** 屬性的值。 當您將活動類型設為 HDInsightHive 時，**typeProperties** 區段中支援下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| script |指定 Hive 指令碼內嵌 |否 |
| 指令碼路徑 |在 Azure Blob 儲存體中儲存 Hive 指令碼，並提供檔案的路徑。 使用 'script' 或 'scriptPath' 屬性。 兩者無法同時使用。 檔案名稱有區分大小寫。 |否 |
| 定義 |在使用 'hiveconf' 的 Hive 指令碼內指定參數做為參考的金鑰/值組 |否 |

只有 Hive 活動才有這些類型屬性。 其他屬性 (在 typeProperties 區段外) 在所有活動中都支援。   

### <a name="json-example"></a>JSON 範例
下列 JSON 定義管線中的 HDInsight Hive 活動。  

```json
{
    "name": "Hive Activity",
    "description": "description",
    "type": "HDInsightHive",
    "inputs": [
      {
        "name": "input tables"
      }
    ],
    "outputs": [
      {
        "name": "output tables"
      }
    ],
    "linkedServiceName": "MyHDInsightLinkedService",
    "typeProperties": {
      "script": "Hive script",
      "scriptPath": "<pathtotheHivescriptfileinAzureblobstorage>",
      "defines": {
        "param1": "param1Value"
      }
    },
   "scheduler": {
      "frequency": "Day",
      "interval": 1
    }
}
```

如需詳細資訊，請參閱 [Hive 活動](data-factory-hive-activity.md)文件。 

## <a name="hdinsight-pig-activity"></a>HDInsight Pig 活動
您可以在 Pig 活動 JSON 定義中指定下列屬性。 活動的 type 屬性必須是︰**HDInsightPig**。 您必須先建立 HDInsight 連結服務，再指定它的名稱作為 **linkedServiceName** 屬性的值。 當您將活動類型設為 HDInsightPig 時，**typeProperties** 區段中支援下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| script |指定 Pig 指令碼內嵌 |否 |
| 指令碼路徑 |在 Azure blob 儲存體中儲存 Pig 指令碼，並提供檔案的路徑。 使用 'script' 或 'scriptPath' 屬性。 兩者無法同時使用。 檔案名稱有區分大小寫。 |否 |
| 定義 |在使用 Pig 指令碼內指定參數做為參考的機碼/值組 |否 |

只有 Pig 活動才有這些類型屬性。 其他屬性 (在 typeProperties 區段外) 在所有活動中都支援。   

### <a name="json-example"></a>JSON 範例

```json
{
    "name": "HiveActivitySamplePipeline",
      "properties": {
    "activities": [
        {
            "name": "Pig Activity",
            "description": "description",
            "type": "HDInsightPig",
            "inputs": [
                  {
                    "name": "input tables"
                  }
            ],
            "outputs": [
                  {
                    "name": "output tables"
                  }
            ],
            "linkedServiceName": "MyHDInsightLinkedService",
            "typeProperties": {
                  "script": "Pig script",
                  "scriptPath": "<pathtothePigscriptfileinAzureblobstorage>",
                  "defines": {
                    "param1": "param1Value"
                  }
            },
               "scheduler": {
                  "frequency": "Day",
                  "interval": 1
            }
          }
    ]
  }
}
```

如需詳細資訊，請參閱 [Pig 活動](#data-factory-pig-activity.md)文件。 

## <a name="hdinsight-mapreduce-activity"></a>HDInsight MapReduce 活動
您可以在 MapReduce 活動 JSON 定義中指定下列屬性。 活動的 type 屬性必須是︰**HDInsightMapReduce**。 您必須先建立 HDInsight 連結服務，再指定它的名稱作為 **linkedServiceName** 屬性的值。 當您將活動類型設為 HDInsightMapReduce 時，**typeProperties** 區段中支援下列屬性︰ 

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| jarLinkedService | 含有 JAR 檔案之 Azure 儲存體的連結服務名稱。 | 是 |
| jarFilePath | Azure 儲存體中的 JAR 檔案路徑。 | 是 | 
| className | JAR 檔案中的主要類別名稱。 | 是 | 
| arguments | MapReduce 程式以逗號分隔的引數清單。 在執行階段，您會看到幾個來自 MapReduce 架構的額外引數 (例如：mapreduce.job.tags)。 若要區分您的引數與 MapReduce 引數，請考慮同時使用選項和值作為引數，如下列範例所示 (-s、--input、--output 等等是後面接著其值的選項) | 否 | 

### <a name="json-example"></a>JSON 範例

```json
{
    "name": "MahoutMapReduceSamplePipeline",
    "properties": {
        "description": "Sample Pipeline to Run a Mahout Custom Map Reduce Jar. This job calculates an Item Similarity Matrix to determine the similarity between two items",
        "activities": [
            {
                "type": "HDInsightMapReduce",
                "typeProperties": {
                    "className": "org.apache.mahout.cf.taste.hadoop.similarity.item.ItemSimilarityJob",
                    "jarFilePath": "adfsamples/Mahout/jars/mahout-examples-0.9.0.2.2.7.1-34.jar",
                    "jarLinkedService": "StorageLinkedService",
                    "arguments": ["-s", "SIMILARITY_LOGLIKELIHOOD", "--input", "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/input", "--output", "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/output/", "--maxSimilaritiesPerItem", "500", "--tempDir", "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/temp/mahout"]
                },
                "inputs": [
                    {
                        "name": "MahoutInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "MahoutOutput"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "retry": 3
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "name": "MahoutActivity",
                "description": "Custom Map Reduce to generate Mahout result",
                "linkedServiceName": "HDInsightLinkedService"
            }
        ],
        "start": "2017-01-03T00:00:00",
        "end": "2017-01-04T00:00:00"
    }
}
```

如需詳細資訊，請參閱 [MapReduce 活動](data-factory-map-reduce.md)文件。 

## <a name="hdinsight-streaming-activity"></a>HDInsight 串流活動
您可以在 Hadoop 串流活動 JSON 定義中指定下列屬性。 活動的 type 屬性必須是︰**HDInsightStreaming**。 您必須先建立 HDInsight 連結服務，再指定它的名稱作為 **linkedServiceName** 屬性的值。 當您將活動類型設為 HDInsightStreaming 時，**typeProperties** 區段中支援下列屬性︰ 

| 屬性 | 說明 | 
| --- | --- |
| mapper | 對應程式可執行檔的名稱。 在範例中，cat.exe 是對應程式可執行檔。| 
| reducer | 歸納器可執行檔的名稱。 在範例中，cat.exe 是減壓器可執行檔。 | 
| input | 對應工具的輸入檔 (包括位置)。 在 "wasb://adfsample@<account name>.blob.core.windows.net/example/data/gutenberg/davinci.txt" 範例中：adfsample 是 blob 容器，example/data/Gutenberg 是資料夾，而 davinci.txt 是 blob。 |
| output | 歸納器的輸出檔 (包括位置)。 Hadoop 串流作業的輸出會寫入針對這個屬性指定的位置。 |
| filePaths | 對應器和歸納器可執行檔的路徑。 在 "adfsample/example/apps/wc.exe" 範例中，adfsample 是 blob 容器，example/apps 是資料夾，而 wc.exe 是可執行檔。 | 
| fileLinkedService | Azure 儲存體連結服務，代表含有 filePaths 區段中指定之檔案的 Azure 儲存體。 | 
| arguments | MapReduce 程式以逗號分隔的引數清單。 在執行階段，您會看到幾個來自 MapReduce 架構的額外引數 (例如：mapreduce.job.tags)。 若要區分您的引數與 MapReduce 引數，請考慮同時使用選項和值作為引數，如下列範例所示 (-s、--input、--output 等等是後面接著其值的選項) | 
| getDebugInfo | 選擇性元素。 該屬性設定為 [失敗] 時，只能在執行失敗時下載記錄檔。 當其設定為「所有」時，無論執行狀態為何，一律下載記錄檔。 | 

> [!NOTE]
> 您必須在 **outputs** 屬性中指定 Hadoop 串流活動的輸出資料集。 這個資料集可能只是驅動管線排程 (每小時、每天等) 所需的虛設資料集。 如果活動不需要有輸入，您可以略過不用在 **inputs** 屬性中指定活動的輸入資料集。  

## <a name="json-example"></a>JSON 範例

```json
{
    "name": "HadoopStreamingPipeline",
    "properties": {
        "description": "Hadoop Streaming Demo",
        "activities": [
            {
                "type": "HDInsightStreaming",
                "typeProperties": {
                    "mapper": "cat.exe",
                    "reducer": "wc.exe",
                    "input": "wasb://<nameofthecluster>@spestore.blob.core.windows.net/example/data/gutenberg/davinci.txt",
                    "output": "wasb://<nameofthecluster>@spestore.blob.core.windows.net/example/data/StreamingOutput/wc.txt",
                    "filePaths": ["<nameofthecluster>/example/apps/wc.exe","<nameofthecluster>/example/apps/cat.exe"],
                    "fileLinkedService": "StorageLinkedService",
                    "getDebugInfo": "Failure"
                },
                "outputs": [
                    {
                        "name": "StreamingOutputDataset"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "RunHadoopStreamingJob",
                "description": "Run a Hadoop streaming job",
                "linkedServiceName": "HDInsightLinkedService"
            }
        ],
        "start": "2014-01-04T00:00:00",
        "end": "2014-01-05T00:00:00"
    }
}
```

如需詳細資訊，請參閱 [Hadoop 串流活動](data-factory-hadoop-streaming-activity.md)文件。 

## <a name="hdinsight-spark-activity"></a>HDInsight Spark 活動
您可以在 Spark 活動 JSON 定義中指定下列屬性。 活動的 type 屬性必須是︰**HDInsightSpark**。 您必須先建立 HDInsight 連結服務，再指定它的名稱作為 **linkedServiceName** 屬性的值。 當您將活動類型設為 HDInsightSpark 時，**typeProperties** 區段中支援下列屬性︰ 

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- |
| rootPath | Spark 檔案所在的 Azure Blob 容器和資料夾。 檔案名稱有區分大小寫。 | 是 |
| entryFilePath | Spark 程式碼/套件之根資料夾的相對路徑。 | 是 |
| className | 應用程式的 Java/Spark 主要類別 | 否 | 
| arguments | Spark 程式的命令列引數清單。 | 否 | 
| proxyUser | 模擬來執行 Spark 程式的使用者帳戶 | 否 | 
| sparkConfig | Spark 組態屬性。 | 否 | 
| getDebugInfo | 指定何時將 Spark 記錄檔複製到 HDInsight 叢集所使用 (或) sparkJobLinkedService 所指定的 Azure 儲存體。 允許的值︰None、Always 或 Failure。 預設值：None。 | 否 | 
| sparkJobLinkedService | 存放 Spark 作業檔案、相依性和記錄的 Azure 儲存體連結服務。  如果您未指定此屬性的值，則會使用與 HDInsight 叢集相關聯的儲存體。 | 否 |

### <a name="json-example"></a>JSON 範例

```json
{
    "name": "SparkPipeline",
    "properties": {
        "activities": [
            {
                "type": "HDInsightSpark",
                "typeProperties": {
                    "rootPath": "adfspark\\pyFiles",
                    "entryFilePath": "test.py",
                    "getDebugInfo": "Always"
                },
                "outputs": [
                    {
                        "name": "OutputDataset"
                    }
                ],
                "name": "MySparkActivity",
                "linkedServiceName": "HDInsightLinkedService"
            }
        ],
        "start": "2017-02-05T00:00:00",
        "end": "2017-02-06T00:00:00"
    }
}
```
請注意下列幾點： 

- **type** 屬性會設為 **HDInsightSpark**。
- **rootPath** 會設為 **adfspark\\pyFiles**，其中 adfspark 是 Azure Blob 容器，而 pyFiles 是該容器中的正常資料夾。 在此範例中，Azure Blob 儲存體是與 Spark 叢集相關聯的儲存體。 您可以將檔案上傳至不同的 Azure 儲存體。 如果您這麼做，請建立 Azure 儲存體連結服務，以將該儲存體帳戶連結至資料處理站。 然後，將連結服務的名稱指定為 **sparkJobLinkedService** 屬性的值。 如需此屬性和 Spark 活動所支援的其他屬性詳細資訊，請參閱 [Spark 活動屬性](#spark-activity-properties)。
- **entryFilePath** 會設為 **test.py**，這就是 python 檔案。 
- **getDebugInfo** 屬性會設為 **Always**，表示永遠產生記錄檔 (不論成功或失敗)。  

    > [!IMPORTANT]
    > 我們建議您不要在生產環境中將這個屬性設定為 Always，除非您要針對問題進行疑難排解。 
- **outputs** 區段有一個輸出資料集。 您必須指定輸出資料集，即使 Spark 程式不會產生任何輸出。 輸出資料集可以驅動管線的排程 (每小時、每天等)。

如需活動的詳細資訊，請參閱 [Spark 活動](data-factory-spark.md)文件。  

## <a name="machine-learning-batch-execution-activity"></a>Machine Learning 批次執行活動
您可以在 Azure ML 批次執行活動 JSON 定義中指定下列屬性。 活動的 type 屬性必須是︰**AzureMLBatchExecution**。 您必須先建立 Azure Machine Learning 連結服務，再指定它的名稱作為 **linkedServiceName** 屬性的值。 當您將活動類型設為 AzureMLBatchExecution 時，**typeProperties** 區段中支援下列屬性︰

屬性 | 說明 | 必要 
-------- | ----------- | --------
webServiceInput | 傳遞作為 Azure ML Web 服務之輸入的資料集。 此資料集也必須包含在活動的輸入中。 |使用 webServiceInput 或 webServiceInputs。 | 
webServiceInputs | 指定要傳遞作為 Azure ML Web 服務之輸入的資料集。 如果 Web 服務接受多個輸入，請使用 webServiceInputs 屬性，而不要使用 webServiceInput 屬性。 **webServiceInputs** 所參考的資料集也必須包含在活動的 **inputs** 中。 | 使用 webServiceInput 或 webServiceInputs。 | 
webServiceOutputs | 指派作為 Azure ML Web 服務之輸出的資料集。 Web 服務會在此資料集中傳回輸出資料。 | 是 | 
globalParameters | 在此區段中指定 Web 服務參數的值。 | 否 | 

### <a name="json-example"></a>JSON 範例
在這個範例中，活動有資料集 **MLSqlInput** 作為輸入，也有 **MLSqlOutput** 作為輸出。 **MLSqlInput** 透過 **webServiceInput** JSON 屬性，傳遞至 Web 服務作為輸入。 **MLSqlOutput** 透過 **webServiceOutputs** JSON 屬性，傳遞至 Web 服務作為輸出。 

```json
{
   "name": "MLWithSqlReaderSqlWriter",
   "properties": {
      "description": "Azure ML model with sql azure reader/writer",
      "activities": [{
         "name": "MLSqlReaderSqlWriterActivity",
         "type": "AzureMLBatchExecution",
         "description": "test",
         "inputs": [ { "name": "MLSqlInput" }],
         "outputs": [ { "name": "MLSqlOutput" } ],
         "linkedServiceName": "MLSqlReaderSqlWriterDecisionTreeModel",
         "typeProperties":
         {
            "webServiceInput": "MLSqlInput",
            "webServiceOutputs": {
               "output1": "MLSqlOutput"
            },
            "globalParameters": {
               "Database server name": "<myserver>.database.windows.net",
               "Database name": "<database>",
               "Server user account name": "<user name>",
               "Server user account password": "<password>"
            }              
         },
         "policy": {
            "concurrency": 1,
            "executionPriorityOrder": "NewestFirst",
            "retry": 1,
            "timeout": "02:00:00"
         }
      }],
      "start": "2016-02-13T00:00:00",
       "end": "2016-02-14T00:00:00"
   }
}
```

在 JSON 範例中，已部署的 Azure Machine Learning Web 服務使用讀取器和寫入器模組，讀取 Azure SQL Database 的資料，或將資料寫入其中。 此 Web 服務會公開下列 4 個參數：資料庫伺服器名稱、資料庫名稱、伺服器使用者帳戶名稱和伺服器使用者帳戶密碼。

> [!NOTE]
> 只有當輸入及輸出屬於 AzureMLBatchExecution 活動時，才可以當做參數傳遞至 Web 服務。 例如，在上面的 JSON 片段中，MLSqlInput 是 AzureMLBatchExecution 活動的輸入，其透過 webServiceInput 參數傳遞至 Web 服務作為輸入。

## <a name="machine-learning-update-resource-activity"></a>Machine Learning 更新資源活動
您可以在 Azure ML 更新資源活動 JSON 定義中指定下列屬性。 活動的 type 屬性必須是︰**AzureMLUpdateResource**。 您必須先建立 Azure Machine Learning 連結服務，再指定它的名稱作為 **linkedServiceName** 屬性的值。 當您將活動類型設為 AzureMLUpdateResource 時，**typeProperties** 區段中支援下列屬性︰

屬性 | 說明 | 必要 
-------- | ----------- | --------
trainedModelName | 重新定型之模型的名稱。 | 是 |  
trainedModelDatasetName | 此資料集指向重新訓練作業所傳回的 iLearner 檔案。 | 是 | 

### <a name="json-example"></a>JSON 範例
管線有兩個活動：**AzureMLBatchExecution** 和 **AzureMLUpdateResource**。 Azure ML 批次執行活動會以訓練資料做為輸入並產生 iLearner 檔案做為輸出。 此活動會使用輸入訓練資料叫用訓練 Web 服務 (公開為 Web 服務的訓練實驗)，並從 Web 服務接收 iLearner 檔案。 PlaceholderBlob 只是 Azure Data Factory 服務執行管線所需的虛擬輸出資料集而已。


```json
{
    "name": "pipeline",
    "properties": {
        "activities": [
            {
                "name": "retraining",
                "type": "AzureMLBatchExecution",
                "inputs": [
                    {
                        "name": "trainingData"
                    }
                ],
                "outputs": [
                    {
                        "name": "trainedModelBlob"
                    }
                ],
                "typeProperties": {
                    "webServiceInput": "trainingData",
                    "webServiceOutputs": {
                        "output1": "trainedModelBlob"
                    }              
                 },
                "linkedServiceName": "trainingEndpoint",
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1,
                    "timeout": "02:00:00"
                }
            },
            {
                "type": "AzureMLUpdateResource",
                "typeProperties": {
                    "trainedModelName": "trained model",
                    "trainedModelDatasetName" :  "trainedModelBlob"
                },
                "inputs": [{ "name": "trainedModelBlob" }],
                "outputs": [{ "name": "placeholderBlob" }],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "retry": 3
                },
                "name": "AzureML Update Resource",
                "linkedServiceName": "updatableScoringEndpoint2"
            }
        ],
        "start": "2016-02-13T00:00:00",
        "end": "2016-02-14T00:00:00"
    }
}
```

## <a name="data-lake-analytics-u-sql-activity"></a>Data Lake Analytics U-SQL 活動
您可以在 U-SQL 活動 JSON 定義中指定下列屬性。 活動的 type 屬性必須是︰**DataLakeAnalyticsU-SQL**。 您必須建立 Azure Data Lake Analytics 連結服務，並指定它的名稱作為 **linkedServiceName** 屬性的值。 當您將活動類型設為 DataLakeAnalyticsU-SQL 時，**typeProperties** 區段中支援下列屬性︰ 

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| scriptPath |包含 U-SQL 指令碼的資料夾的路徑。 檔案的名稱有區分大小寫。 |否 (如果您使用指令碼) |
| scriptLinkedService |連結服務會連結包含 Data Factory 的指令碼的儲存體 |否 (如果您使用指令碼) |
| script |指定內嵌指令碼而不是指定 scriptPath 和 scriptLinkedService。 例如："script": "CREATE DATABASE test"。 |否 (如果您使用 scriptPath 和 scriptLinkedService) |
| degreeOfParallelism |同時用來執行作業的節點數目上限。 |否 |
| 優先順序 |判斷應該選取排入佇列的哪些工作首先執行。 編號愈低，優先順序愈高。 |否 |
| parameters |U-SQL 指令碼的參數 |否 |

### <a name="json-example"></a>JSON 範例

```json
{
    "name": "ComputeEventsByRegionPipeline",
    "properties": {
        "description": "This pipeline computes events for en-gb locale and date less than Feb 19, 2012.",
        "activities": 
        [
            {
                "type": "DataLakeAnalyticsU-SQL",
                "typeProperties": {
                    "scriptPath": "scripts\\kona\\SearchLogProcessing.txt",
                    "scriptLinkedService": "StorageLinkedService",
                    "degreeOfParallelism": 3,
                    "priority": 100,
                    "parameters": {
                        "in": "/datalake/input/SearchLog.tsv",
                        "out": "/datalake/output/Result.tsv"
                    }
                },
                "inputs": [
                    {
                        "name": "DataLakeTable"
                    }
                ],
                "outputs": 
                [
                    {
                        "name": "EventsByRegionTable"
                    }
                ],
                "policy": {
                    "timeout": "06:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "EventsByRegion",
                "linkedServiceName": "AzureDataLakeAnalyticsLinkedService"
            }
        ],
        "start": "2015-08-08T00:00:00",
        "end": "2015-08-08T01:00:00",
        "isPaused": false
    }
}
```

如需詳細資訊，請參閱 [Data Lake Analytics U-SQL 活動](data-factory-usql-activity.md)。 

## <a name="stored-procedure-activity"></a>預存程序活動
您可以在預存程序活動 JSON 定義中指定下列屬性。 活動的 type 屬性必須是︰**SqlServerStoredProcedure**。 您必須建立下列其中一個連結服務，並指定連結服務的名稱作為 **linkedServiceName** 屬性的值：

- SQL Server 
- 連接字串
- Azure SQL 資料倉儲

當您將活動類型設為 SqlServerStoredProcedure 時，**typeProperties** 區段中支援下列屬性︰

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| storedProcedureName |指定 Azure SQL Database 或 Azure SQL 資料倉儲中預存程序的名稱，由輸出資料表使用的連結的服務代表。 |是 |
| storedProcedureParameters |指定預存程序參數的值。 如果您要為參數傳遞 null，請使用語法："param1": null (全部小寫)。 請參閱下列範例以了解如何使用這個屬性。 |否 |

如果您有指定輸入資料集，它必須可供使用 (「就緒」狀態)，預存程序活動才能執行。 在預存程序中輸入資料集無法做為參數取用。 它只會用來在啟動預存程序活動之前檢查相依性。 您必須指定預存程序活動的輸出資料集。 

輸出資料集會指定預存程序活動的 **排程** (每小時、每週、每月等)。 輸出資料集必須使用參考了想在其中執行預存程序之 Azure SQL Database、Azure SQL 資料倉儲或 SQL Server Database 的 **連結服務** 。 輸出資料集可以做為傳遞預存程序結果，以供管線中的另一個活動 ([鏈結活動](data-factory-scheduling-and-execution.md##multiple-activities-in-a-pipeline)) 進行後續處理的方式。 不過，Data Factory 不會自動將預存程序的輸出寫入至此資料集。 它是會寫入至輸出資料集所指向之 SQL 資料表的預存程序。 在某些情況下，輸出資料集可以是 **虛擬資料集**，只會用來指定用於執行預存程序活動的排程。  

### <a name="json-example"></a>JSON 範例

```json
{
    "name": "SprocActivitySamplePipeline",
    "properties": {
        "activities": [
            {
                "type": "SqlServerStoredProcedure",
                "typeProperties": {
                    "storedProcedureName": "sp_sample",
                    "storedProcedureParameters": {
                        "DateTime": "$$Text.Format('{0:yyyy-MM-dd HH:mm:ss}', SliceStart)"
                    }
                },
                "outputs": [{ "name": "sprocsampleout" }],
                "name": "SprocActivitySample"
            }
        ],
         "start": "2016-08-02T00:00:00",
         "end": "2016-08-02T05:00:00",
        "isPaused": false
    }
}
```

如需詳細資訊，請參閱[預存程序活動](data-factory-stored-proc-activity.md)。 

## <a name="net-custom-activity"></a>.NET 自訂活動
您可以在 .NET 自訂活動 JSON 定義中指定下列屬性。 活動的 type 屬性必須是︰**DotNetActivity**。 您必須建立 Azure HDInsight 連結服務，或 Azure Batch 連結服務，然後指定連結服務的名稱作為 **linkedServiceName** 屬性的值。 當您將活動類型設為 DotNetActivity 時，**typeProperties** 區段中支援下列屬性︰
 
| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| AssemblyName | 組件的名稱。 在此範例中，它是︰**MyDotnetActivity.dll**。 | 是 |
| EntryPoint |實作 IDotNetActivity 介面的類別名稱。 在此範例中，它是︰**MyDotNetActivityNS.MyDotNetActivity**，其中 MyDotNetActivityNS 是命名空間，而 MyDotNetActivity 是類別。  | 是 | 
| PackageLinkedService | Azure 儲存體連結服務的名稱，指向包含自訂活動 zip 檔案的 Blob 儲存體。 在此範例中，它是：**AzureStorageLinkedService**。| 是 |
| PackageFile | zip 檔案的名稱。 在此範例中，它是：**customactivitycontainer/MyDotNetActivity.zip**。 | 是 |
| extendedProperties | 可定義並傳遞至 .NET 程式碼的擴充屬性。 在此範例中，**SliceStart** 變數的值是根據 SliceStart 系統變數來設定。 | 否 | 

### <a name="json-example"></a>JSON 範例

```json
{
  "name": "ADFTutorialPipelineCustom",
  "properties": {
    "description": "Use custom activity",
    "activities": [
      {
        "Name": "MyDotNetActivity",
        "Type": "DotNetActivity",
        "Inputs": [
          {
            "Name": "InputDataset"
          }
        ],
        "Outputs": [
          {
            "Name": "OutputDataset"
          }
        ],
        "LinkedServiceName": "AzureBatchLinkedService",
        "typeProperties": {
          "AssemblyName": "MyDotNetActivity.dll",
          "EntryPoint": "MyDotNetActivityNS.MyDotNetActivity",
          "PackageLinkedService": "AzureStorageLinkedService",
          "PackageFile": "customactivitycontainer/MyDotNetActivity.zip",
          "extendedProperties": {
            "SliceStart": "$$Text.Format('{0:yyyyMMddHH-mm}', Time.AddMinutes(SliceStart, 0))"
          }
        },
        "Policy": {
          "Concurrency": 2,
          "ExecutionPriorityOrder": "OldestFirst",
          "Retry": 3,
          "Timeout": "00:30:00",
          "Delay": "00:00:00"
        }
      }
    ],
    "start": "2016-11-16T00:00:00",
    "end": "2016-11-16T05:00:00",
    "isPaused": false
  }
}
```

如需詳細資訊，請參閱[在 Data Factory 中使用自訂活動](data-factory-use-custom-activities.md)文件。 

## <a name="next-steps"></a>後續步驟
請參閱下列教學課程： 

- [教學課程：建立具有複製活動的管線](data-factory-copy-activity-tutorial-using-azure-portal.md)
- [教學課程：建立具有 Hive 活動的管線](data-factory-build-your-first-pipeline-using-editor.md)
