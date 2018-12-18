---
title: 在 Data Factory 中建立/排程管線和鏈結活動 | Microsoft Docs
description: 了解如何在 Azure Data Factory 中建立用來移動和轉換資料的資料管線。 建立資料導向工作流程以產生可供使用的資訊。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 13b137c7-1033-406f-aea7-b66f25b313c0
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 09fd2f38c3746cf92d576325058dc36221ae50cd
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38668022"
---
# <a name="pipelines-and-activities-in-azure-data-factory"></a>Azure Data Factory 中的管線及活動
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-create-pipelines.md)
> * [第 2 版 (目前的版本)](../concepts-pipelines-activities.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[第 2 版中的管線](../concepts-pipelines-activities.md)。

本文協助您了解 Azure Data Factory 中的管線和活動，並使用這些項目來為您的資料移動和資料處理案例建構端對端的資料導向工作流程。  

> [!NOTE]
> 本文假設您已經看過 [Azure Data Factory 簡介](data-factory-introduction.md)。 如果您沒有建立 Data Factory 的實作經驗，瀏覽[資料轉換教學課程](data-factory-build-your-first-pipeline.md)和/或[資料移動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)將可協助您進一步了解這篇文章。  

## <a name="overview"></a>概觀
資料處理站可以有一或多個管線。 管線是一起執行某個工作的活動所組成的邏輯群組。 管線中的活動會定義要在資料上執行的動作。 例如，您可以使用複製活動將資料從內部部署 SQL Server 複製到「Azure Blob 儲存體」。 接著，使用在 Azure HDInsight 叢集上執行 Hive 指令碼的 Hive 活動，來處理/轉換來自 Blob 儲存體的資料以產生輸出資料。 最後，使用第二個複製活動將輸出資料複製到 Azure SQL 資料倉儲，以在該處建置商業智慧 (BI) 報表解決方案。 

一個活動可以接受零個或多個輸入[資料集](data-factory-create-datasets.md)，並且會產生一個或多個輸出[資料集](data-factory-create-datasets.md)。 下圖顯示 Data Factory 中管線、活動及資料集之間的關聯性： 

![管線、活動及資料集之間的關聯性](media/data-factory-create-pipelines/relationship-pipeline-activity-dataset.png)

管線可讓您可以將活動當作一個集合來管理，而不是個別管理每個活動。 例如，您可以部署、排定、暫止及繼續管線，而不是以單獨的方式處理管線中的活動。

Data Factory 支援兩種活動類型︰資料移動活動和資料轉換活動。 每個活動可以有零個或多個輸入[資料集](data-factory-create-datasets.md)，並且會產生一個或多個輸出資料集。

輸入資料集表示管線中的活動輸入，而輸出資料集表示活動的輸出。 資料集可識別資料表、檔案、資料夾和文件等各種資料存放區中的資料。 建立資料集之後，您可以將其與管線中的活動搭配使用。 例如，資料集可以是複製活動或 HDInsightHive 活動的輸入/輸出資料集。 如需有關資料集的詳細資訊，請參閱 [Azure Data Factory 中的資料集](data-factory-create-datasets.md)一文。

### <a name="data-movement-activities"></a>資料移動活動
Data Factory 中的複製活動會將資料從來源資料存放區複製到接收資料存放區。 Data Factory 支援下列資料存放區。 可將來自任何來源的資料寫入任何接收器。 按一下資料存放區，即可了解如何將資料複製到該存放區，以及從該存放區複製資料。

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

> [!NOTE]
> 具有 * 的資料存放區可以在內部部署環境或 Azure IaaS 上，並且需要您在內部部署/Azure IaaS 機器上安裝 [資料管理閘道](data-factory-data-management-gateway.md) 。

如需詳細資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md)文章。

### <a name="data-transformation-activities"></a>資料轉換活動
[!INCLUDE [data-factory-transformation-activities](../../../includes/data-factory-transformation-activities.md)]

如需詳細資訊，請參閱[資料轉換活動](data-factory-data-transformation-activities.md)文章。

### <a name="custom-net-activities"></a>自訂 .NET 活動 
如果您需要將資料移入/移出「複製活動」不支援的資料存放區，或使用您自己的邏輯來轉換資料，請建立**自訂 .NET 活動**。 如需有關建立及使用自訂活動的詳細資料，請參閱 [在 Azure Data Factory 管線中使用自訂活動](data-factory-use-custom-activities.md)。

## <a name="schedule-pipelines"></a>建立管線排程
管線僅在其「開始」時間與「結束」時間之間有作用。 在開始時間之前或結束時間之後就不會執行。 如果管線已暫停，不論其開始和結束時間為何，都不會執行。 若要執行管線，則不該將它暫停。 請參閱 [排程和執行](data-factory-scheduling-and-execution.md) ，以了解如何在 Azure Data Factory 中排程和執行。

## <a name="pipeline-json"></a>管線 JSON
讓我們來深入探討如何定義 JSON 格式的管線。 管線的一般結構如下所示：

```json
{
    "name": "PipelineName",
    "properties": 
    {
        "description" : "pipeline description",
        "activities":
        [

        ],
        "start": "<start date-time>",
        "end": "<end date-time>",
        "isPaused": true/false,
        "pipelineMode": "scheduled/onetime",
        "expirationTime": "15.00:00:00",
        "datasets": 
        [
        ]
    }
}
```

| Tag | 說明 | 必要 |
| --- | --- | --- |
| name |管線的名稱。 指定代表管線所執行之動作的名稱。 <br/><ul><li>字元數目上限︰260</li><li>開頭必須為字母、數字或底線 (_)</li><li>不允許使用下列字元：“.”、“+”、“?”、“/”、“<”、”>”、”*”、”%”、”&”、”:”、”\\”</li></ul> |yes |
| 說明 | 指定說明管線用途的文字。 |yes |
| 活動 | [ **活動** ] 區段內可以有一或多個已定義的活動。 如需有關活動 JSON 元素的詳細資料，請參閱下一節。 | yes |  
| start | 管線的開始日期時間。 必須使用 [ISO 格式](http://en.wikipedia.org/wiki/ISO_8601)。 例如：`2016-10-14T16:32:41Z`。 <br/><br/>您可以指定本地時間，如 EST 時間。 範例如下︰`2016-02-27T06:00:00-05:00`，這是美加東部標準時間上午 6 點。<br/><br/>管線的 start 和 end 屬性共同指定管線的作用中期間。 輸出配量只會在作用中期間內產生。 |否<br/><br/>如果您指定 end 屬性的值，也必須指定 start 屬性的值。<br/><br/>開始和結束時間都可以是空白來建立管線。 必須指定兩個值，才能設定執行管線的作用中時間。 如果您建立管線時未指定開始和結束時間，您可以在稍後使用 Set-AzureRmDataFactoryPipelineActivePeriod Cmdlet 進行設定。 |
| end | 管線的結束日期時間。 如果已指定，則必須使用 ISO 格式。 例如：`2016-10-14T17:32:41Z` <br/><br/>您可以指定本地時間，如 EST 時間。 範例如下︰`2016-02-27T06:00:00-05:00`，這是美加東部標準時間上午 6 點。<br/><br/>若要無限期地執行管線，請指定 9999-09-09 做為 end 屬性的值。 <br/><br/> 管線僅在其開始時間與結束時間之間有作用。 在開始時間之前或結束時間之後就不會執行。 如果管線已暫停，不論其開始和結束時間為何，都不會執行。 若要執行管線，則不該將它暫停。 請參閱 [排程和執行](data-factory-scheduling-and-execution.md) ，以了解如何在 Azure Data Factory 中排程和執行。 |否 <br/><br/>如果您指定 start 屬性的值，也必須指定 end 屬性的值。<br/><br/>請參閱 **start** 屬性的註釋。 |
| isPaused | 如果設定為 true，管線就不會執行。 它會處於暫停狀態。 預設值 = false。 您可以使用此屬性來啟用或停用管線。 |否 |
| pipelineMode | 排程管線執行的方法。 允許的值包括：scheduled (預設值)、onetime。<br/><br/>‘Scheduled’ 表示管線會根據其作用中期間 (開始和結束時間) 依指定的時間間隔執行。 ‘Onetime’ 表示管線只會執行一次。 目前，Onetime 管線在建立之後即無法進行修改/更新。 如需 onetime 設定的詳細資料，請參閱 [Onetime 管線](#onetime-pipeline)。 |否 |
| expirationTime | 建立之後，[單次管線](#onetime-pipeline)有效且應該維持佈建狀態的持續時間。 如果管線沒有任何作用中、失敗或擱置中的執行，系統就會在管線達到到期時間時，自動將它刪除。 預設值：`"expirationTime": "3.00:00:00"`|否 |
| 資料集 |要供管線中已定義活動所使用的資料集清單。 此屬性可用來定義此管線所特有但未在 Data Factory 內定義的資料集。 此管線內定義的資料集只有此管線才能使用，並無法共用。 如需詳細資訊，請參閱 [範圍資料集](data-factory-create-datasets.md#scoped-datasets) 。 |否 |

## <a name="activity-json"></a>活動 JSON
[ **活動** ] 區段內可以有一或多個已定義的活動。 每個活動都有下列最上層結構：

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
    },
    "scheduler":
    {
    }
}
```

下表說明活動 JSON 定義內的屬性：

| Tag | 說明 | 必要 |
| --- | --- | --- |
| name | 活動的名稱。 指定代表活動所執行之動作的名稱。 <br/><ul><li>字元數目上限︰260</li><li>開頭必須為字母、數字或底線 (_)</li><li>不允許使用下列字元：“.”、“+”、“?”、“/”、“<”、”>”、”*”、”%”、”&”、”:”、”\\”</li></ul> |yes |
| 說明 | 說明活動用途的文字 |yes |
| type | 活動的類型。 如需了解不同類型的活動，請參閱[資料移動活動](#data-movement-activities)和[資料轉換活動](#data-transformation-activities)小節。 |yes |
| 輸入 |活動所使用的輸入資料表<br/><br/>`// one input table`<br/>`"inputs":  [ { "name": "inputtable1"  } ],`<br/><br/>`// two input tables` <br/>`"inputs":  [ { "name": "inputtable1"  }, { "name": "inputtable2"  } ],` |yes |
| 輸出 |活動所使用的輸出資料表。<br/><br/>`// one output table`<br/>`"outputs":  [ { "name": "outputtable1" } ],`<br/><br/>`//two output tables`<br/>`"outputs":  [ { "name": "outputtable1" }, { "name": "outputtable2" }  ],` |yes |
| 預設容器 |活動所使用的連結服務名稱。 <br/><br/>活動可能會要求您指定可連結至所需計算環境的連結服務。 |是：適用於 HDInsight 活動和 Azure Machine Learning Batch 評分活動  <br/><br/>否：所有其他 |
| typeProperties |**typeProperties** 區段中的屬性會視活動的類型而定。 若要查看活動的類型屬性，請按一下先前小節中的活動連結。 | 否 |
| 原則 |會影響活動之執行階段行為的原則。 如果未指定，則會使用預設原則。 |否 |
| scheduler | “scheduler” 屬性用來定義所要的活動排程。 其子屬性與 [資料集中的可用性屬性](data-factory-create-datasets.md#dataset-availability)中的屬性相同。 |否 |


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

## <a name="sample-copy-pipeline"></a>範例複製管線
在以下的範例管線中， **Copy** in the **活動** 類型的活動。 在此範例中，[複製活動](data-factory-data-movement-activities.md)會將資料從 Azure Blob 儲存體複製到 Azure SQL Database。 

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
    "start": "2016-07-12T00:00:00Z",
    "end": "2016-07-13T00:00:00Z"
  }
} 
```

請注意下列幾點：

* 在活動區段中，只會有一個 **type** 設為 **Copy** 的活動。
* 活動的輸入設定為 **InputDataset**，活動的輸出則設定為 **OutputDataset**。 若要了解如何以 JSON 定義資料集，請參閱[資料集](data-factory-create-datasets.md)一文。 
* 在 **typeProperties** 區段中，來源類型指定為 **BlobSource**，接收類型指定為 **SqlSink**。 在[資料移動活動](#data-movement-activities)一節中，按一下您想要用來作為來源或接收器的資料存放區，以深入了解如何將資料移入/移出該資料存放區。 

如需建立此管線的完整逐步解說，請參閱 [教學課程：將資料從 Blob 儲存體複製到 SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。 

## <a name="sample-transformation-pipeline"></a>範例轉換管線
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
        "start": "2016-04-01T00:00:00Z",
        "end": "2016-04-02T00:00:00Z",
        "isPaused": false
    }
}
```

請注意下列幾點： 

* 在活動區段中，只會有一個 **type** 設為 **HDInsightHive** 的活動。
* Hive 指令碼檔案 **partitionweblogs.hql** 儲存於 Azure 儲存體帳戶 (透過名為 **AzureStorageLinkedService** 的 scriptLinkedService 指定)，且位於 **adfgetstarted** 容器的 **script** 資料夾中。
* `defines` 區段可用來指定執行階段設定，該設定將傳遞給 Hive 指令碼作為 Hive 設定值 (例如 `${hiveconf:inputtable}`、`${hiveconf:partitionedtable}`)。

每個轉換活動的 **typeProperties** 區段都不同。 若要深入了解某個轉換活動支援的類型屬性，請按一下[資料轉換活動](#data-transformation-activities)表格中的該轉換活動。 

如需建立此管線的完整逐步解說，請參閱 [教學課程：使用 Hadoop 叢集建置您的第一個管線來處理資料](data-factory-build-your-first-pipeline.md)。 

## <a name="multiple-activities-in-a-pipeline"></a>管線中的多個活動
前兩個範例管線都只包含一個活動。 您可以在一個管線中包含多個活動。  

如果您在管線中有多個活動，而且活動的輸出不是另一個活動的輸入，則當活動的輸入資料分割已備妥時，活動可能會平行執行。 

您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動。 只有當第一個活動執行成功完成時，第二個活動才會執行。

![相同管線中的鏈結活動](./media/data-factory-create-pipelines/chaining-one-pipeline.png)

在此範例中，管理有兩個活動：Activity1 和 Activity2。 Activity1 會以 Dataset1 作為輸入，並產生輸出 Dataset2。 Activity2 會以 Dataset2 作為輸入，並產生輸出 Dataset3。 由於 Activity1 的輸出 (Dataset2) 是 Activity2 的輸入，因此只有在 Activity1 順利完成並產生 Dataset2 配量之後，才會執行 Activity2。 如果 Activity1 因某種原因而失敗，而未產生 Dataset2 配量，Activity2 就不會針對該配量 (例如：上午 9 點到上午 10 點) 執行。 

您可以將不同管線中的活動建立鏈結。

![兩個管線中的鏈結活動](./media/data-factory-create-pipelines/chaining-two-pipelines.png)

在此範例中，Pipeline1 只有一個活動，此活動是以 Dataset1 作為輸入，並產生 Dataset2 作為輸出。 Pipeline2 同樣只有一個活動，此活動是以 Dataset2 作為輸入，並產生 Dataset3 作為輸出。 

如需詳細資訊，請參閱 [排程和執行](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)。 

## <a name="create-and-monitor-pipelines"></a>建立和監視管線
您可以使用下列其中一項工具或 SDK 來建立管線。 

- 複製精靈 
- Azure 入口網站
- Visual Studio
- Azure PowerShell
- Azure Resource Manager 範本
- REST API
- .NET API

如需使用上述其中一項工具或 SDK 來建立管線的逐步指示，請參閱下列教學課程。
 
- [使用資料轉換活動來建置管線](data-factory-build-your-first-pipeline.md)
- [使用資料移動活動來建置管線](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)

建置/部署管線之後，您便可以使用 Azure 入口網站刀鋒視窗或「監視與管理」應用程式，來管理及監視您的管線。 如需逐步指示，請參閱下列主題。 

- [使用 Azure 入口網站刀鋒視窗來監視和管理管線](data-factory-monitor-manage-pipelines.md)
- [使用監視與管理應用程式來監視和管理管線](data-factory-monitor-manage-app.md)


## <a name="onetime-pipeline"></a>Onetime 管線
您可以建立和排程管線，以在管線定義中指定的開始和結束時間內定期執行 (例如：每小時或每日)。 如需詳細資訊，請參閱 [排程活動](#scheduling-and-execution) 。 您也可以建立只執行一次的管線。 若要這樣做，您需將管線定義中的 **pipelineMode** 屬性設定為 **onetime** (如下列 JSON 範例所示)。 這個屬性的預設值是 **scheduled**。

```json
{
    "name": "CopyPipeline",
    "properties": {
        "activities": [
            {
                "type": "Copy",
                "typeProperties": {
                    "source": {
                        "type": "BlobSource",
                        "recursive": false
                    },
                    "sink": {
                        "type": "BlobSink",
                        "writeBatchSize": 0,
                        "writeBatchTimeout": "00:00:00"
                    }
                },
                "inputs": [
                    {
                        "name": "InputDataset"
                    }
                ],
                "outputs": [
                    {
                        "name": "OutputDataset"
                    }
                ]
                "name": "CopyActivity-0"
            }
        ]
        "pipelineMode": "OneTime"
    }
}
```

請注意：

* 未指定管線的**開始**和**結束**時間。
* 有指定輸入和輸出資料集的**可用性** (**頻率**和**間隔**)，即使 Data Factory 未使用這些值也是一樣。  
* 圖表檢視不會顯示一次性管線。 這是設計的行為。
* 一次性管線無法更新。 您可以複製一次性管線、將其重新命名、更新屬性，以及加以部署來建立另一個管線。


## <a name="next-steps"></a>後續步驟
- 如需有關資料集的詳細資訊，請參閱[建立資料集](data-factory-create-datasets.md)一文。 
- 如需有關管線的排程和執行方式的詳細資訊，請參閱 [Azure Data Factory 中的排程和執行](data-factory-scheduling-and-execution.md)一文。 
  

