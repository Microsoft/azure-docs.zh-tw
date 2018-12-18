---
title: 使用 Hadoop 資料流活動轉換資料 - Azure | Microsoft Docs
description: 了解如何使用 Azure Data Factory 中的 Hadoop 資料流活動，以在隨選/您自己的 HDInsight 叢集上執行 Hadoop 資料流程式來轉換資料。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 4c3ff8f2-2c00-434e-a416-06dfca2c41ec
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 3d5832f63a3ebe7583d18fcd863c8cc60b9b045d
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37048759"
---
# <a name="transform-data-using-hadoop-streaming-activity-in-azure-data-factory"></a>使用 Azure Data Factory 中的 Hadoop 資料流活動轉換資料
> [!div class="op_single_selector" title1="Transformation Activities"]
> * [Hive 活動](data-factory-hive-activity.md) 
> * [Pig 活動](data-factory-pig-activity.md)
> * [MapReduce 活動](data-factory-map-reduce.md)
> * [Hadoop 串流活動](data-factory-hadoop-streaming-activity.md)
> * [Spark 活動](data-factory-spark.md)
> * [Machine Learning Batch 執行活動](data-factory-azure-ml-batch-execution-activity.md)
> * [Machine Learning 更新資源活動](data-factory-azure-ml-update-resource-activity.md)
> * [預存程序活動](data-factory-stored-proc-activity.md)
> * [Data Lake Analytics U-SQL 活動](data-factory-usql-activity.md)
> * [.NET 自訂活動](data-factory-use-custom-activities.md)

> [!NOTE]
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[在 Data Factory 中使用 Hadoop 串流來轉換資料](../transform-data-using-hadoop-streaming.md)。


您可以使用 HDInsightStreamingActivity 活動從 Azure Data Factory 管線叫用 Hadoop 串流工作。 下列 JSON 片段會示範在管線 JSON 檔案中使用 HDInsightStreamingActivity 的語法。 

Data Factory [管線](data-factory-create-pipelines.md)中的 HDInsight 串流活動會在[您自己](data-factory-compute-linked-services.md#azure-hdinsight-linked-service)或[隨選](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service)的 Windows/Linux 架構 HDInsight 叢集上執行 Hadoop 串流程式。 本文是根據 [資料轉換活動](data-factory-data-transformation-activities.md) 一文，它呈現資料轉換和支援的轉換活動的一般概觀。

> [!NOTE] 
> 如果您是 Azure Data Factory 的新手，請在閱讀本文章之前閱讀 [Azure Data Factory 簡介](data-factory-introduction.md)，以及研習教學課程：[建置您的第一個資料管線](data-factory-build-your-first-pipeline.md)。 

## <a name="json-sample"></a>JSON 範例
HDInsight 叢集會使用範例程式 (wc.exe 和 cat.exe) 和資料 (將 davinci.txt) 自動填入。 根據預設，HDInsight 叢集所使用的容器名稱是叢集本身的名稱。 例如，如果您的叢集名稱是 myhdicluster，相關聯的 Blob 容器名稱為 myhdicluster。 

```JSON
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
                    "filePaths": [
                        "<nameofthecluster>/example/apps/wc.exe",
                        "<nameofthecluster>/example/apps/cat.exe"
                    ],
                    "fileLinkedService": "AzureStorageLinkedService",
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
        "start": "2014-01-04T00:00:00Z",
        "end": "2014-01-05T00:00:00Z"
    }
}
```

請注意下列幾點：

1. 將連結服務的名稱設定為 **linkedServiceName** ，該服務指向您的 HDInsight 叢集，串流 mapreduce 作業會在該叢集上執行。
2. 將活動的類型設為 **HDInsightStreaming**。
3. 針對 **mapper** 屬性，指定對應程式可執行檔的名稱。 在範例中，cat.exe 是對應程式可執行檔。
4. 針對 **reducer** 屬性，指定減壓器可執行檔的名稱。 在範例中，cat.exe 是減壓器可執行檔。
5. 針對 **input** 類型屬性，指定對應程式的輸入檔 (包括位置)。 在 "wasb://adfsample@<account name>.blob.core.windows.net/example/data/gutenberg/davinci.txt" 範例中：adfsample 是 blob 容器，example/data/Gutenberg 是資料夾，而 davinci.txt 是 blob。
6. 針對 **output** 類型屬性，指定減壓器的輸出檔 (包括位置)。 Hadoop 串流作業的輸出會寫入針對這個屬性指定的位置。
7. 在 **filePaths** 區段中，指定對應程式和減壓器可執行檔的路徑。 在 "adfsample/example/apps/wc.exe" 範例中，adfsample 是 blob 容器，example/apps 是資料夾，而 wc.exe 是可執行檔。
8. 針對 **fileLinkedService** 屬性，指定代表 Azure 儲存體 (包含 filePaths 區段中指定的檔案) 的 Azure 儲存體連結服務。
9. 針對 **arguments** 屬性，指定串流工作的引數。
10. **getDebugInfo** 屬性是選擇性的元素。 該屬性設定為 [失敗] 時，只能在執行失敗時下載記錄檔。 當其設定為「永遠」時，無論執行狀態為何，一律下載記錄檔。

> [!NOTE]
> 如範例所示，您必須為 Hadoop 串流活動的 **outputs** 屬性指定輸出資料集。 這個資料集只是一個驅動管線排程所需的虛設資料集。 您不需要為活動的 **input** 屬性指定任何輸入資料集。  
> 
> 

## <a name="example"></a>範例
本逐步解說中的管線會在 Azure HDInsight 叢集上執行字數統計串流 Map/Reduce 程式。 

### <a name="linked-services"></a>連結的服務
#### <a name="azure-storage-linked-service"></a>Azure 儲存體連結服務
首先，建立連結的服務，將 Azure HDInsight 叢集使用的 Azure 儲存體連結到  Azure Data Factory。 如果您複製/貼上下列程式碼，請記得使用 Azure 儲存體的名稱和金鑰來取代帳戶名稱和帳戶金鑰。 

```JSON
{
    "name": "StorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>"
        }
    }
}
```

#### <a name="azure-hdinsight-linked-service"></a>Azure HDInsight 連結服務
接著，建立連結的服務，將 Azure HDInsight 叢集連結到 Azure Data Factory。 如果您複製/貼上下列程式碼，請使用您的 HDInsight 叢集的名稱來取代 HDInsight 叢集名稱，然後變更使用者名稱和密碼值。 

```JSON
{
    "name": "HDInsightLinkedService",
    "properties": {
        "type": "HDInsight",
        "typeProperties": {
            "clusterUri": "https://<HDInsight cluster name>.azurehdinsight.net",
            "userName": "admin",
            "password": "**********",
            "linkedServiceName": "StorageLinkedService"
        }
    }
}
```

### <a name="datasets"></a>資料集
#### <a name="output-dataset"></a>輸出資料集
此範例中的管線不需要取得任何輸入。 您必須指定 HDInsight 串流活動的輸出資料集。 這個資料集只是一個驅動管線排程所需的虛設資料集。 

```JSON
{
    "name": "StreamingOutputDataset",
    "properties": {
        "published": false,
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "folderPath": "adftutorial/streamingdata/",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            },
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

### <a name="pipeline"></a>管線
此範例中的管線只含有一個類型為 **HDInsightStreaming**的活動。 

HDInsight 叢集會使用範例程式 (wc.exe 和 cat.exe) 和資料 (將 davinci.txt) 自動填入。 根據預設，HDInsight 叢集所使用的容器名稱是叢集本身的名稱。 例如，如果您的叢集名稱是 myhdicluster，相關聯的 Blob 容器名稱為 myhdicluster。  

```JSON
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
                    "input": "wasb://<blobcontainer>@spestore.blob.core.windows.net/example/data/gutenberg/davinci.txt",
                    "output": "wasb://<blobcontainer>@spestore.blob.core.windows.net/example/data/StreamingOutput/wc.txt",
                    "filePaths": [
                        "<blobcontainer>/example/apps/wc.exe",
                        "<blobcontainer>/example/apps/cat.exe"
                    ],
                    "fileLinkedService": "StorageLinkedService"
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
        "start": "2017-01-03T00:00:00Z",
        "end": "2017-01-04T00:00:00Z"
    }
}
```
## <a name="see-also"></a>另請參閱
* [Hive 活動](data-factory-hive-activity.md)
* [Pig 活動](data-factory-pig-activity.md)
* [MapReduce 活動](data-factory-map-reduce.md)
* [叫用 Spark 程式](data-factory-spark.md)
* [叫用 R 指令碼](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/RunRScriptUsingADFSample)

