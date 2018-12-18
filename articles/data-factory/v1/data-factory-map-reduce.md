---
title: 從 Azure Data Factory 叫用 MapReduce 程式
description: 了解如何從 Azure Data Factory，在 Azure HDInsight 叢集上執行 MapReduce 程式以處理資料。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: c34db93f-570a-44f1-a7d6-00390f4dc0fa
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: edbef08eaa100248368d7f0b23171f15b52ec56a
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37050942"
---
# <a name="invoke-mapreduce-programs-from-data-factory"></a>從 Data Factory 叫用 MapReduce 程式
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
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[在 Data Factory 中使用 MapReduce 活動來轉換資料](../transform-data-using-hadoop-map-reduce.md)。


Data Factory [管線](data-factory-create-pipelines.md)中的 HDInsight MapReduce 活動會在[您自己](data-factory-compute-linked-services.md#azure-hdinsight-linked-service)或[隨選](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service)的 Windows/Linux 架構 HDInsight 叢集上執行 MapReduce 程式。 本文是根據 [資料轉換活動](data-factory-data-transformation-activities.md) 一文，它呈現資料轉換和支援的轉換活動的一般概觀。

> [!NOTE] 
> 如果您是 Azure Data Factory 的新手，請在閱讀本文章之前閱讀 [Azure Data Factory 簡介](data-factory-introduction.md)，以及研習教學課程：[建置您的第一個資料管線](data-factory-build-your-first-pipeline.md)。  

## <a name="introduction"></a>簡介
Azure Data Factory 中的「管線」會使用連結的計算服務，來處理連結的儲存體服務中的資料。 它包含一系列活動，其中每個活動都會執行特定的處理作業。 本文說明如何使用「HDInsight MapReduce 活動」。

若要了解如何使用 HDInsight 的 Pig 和 Hive 活動，在 Windows/Linux 型 HDInsight 叢集上從管線執行 Pig/Hive 指令碼，請參閱 [Pig](data-factory-pig-activity.md) 和 [Hive](data-factory-hive-activity.md)。 

## <a name="json-for-hdinsight-mapreduce-activity"></a>「HDInsight MapReduce 活動」的 JSON
在 HDInsight 活動的 JSON 定義中： 

1. 將 **activity** 的 **type** 設為 **HDInsight**。
2. 指定 **className** 屬性的類別名稱。
3. 為 **jarFilePath** 屬性指定包含檔案名稱的 JAR 檔案路徑。
4. 為 **jarLinkedService** 屬性指定連結服務，此連結服務參考包含 JAR 檔案的 Azure Blob 儲存體。   
5. 在 **arguments** 區段中，為 MapReduce 程式指定所有引數。 在執行階段，您會看到幾個來自 MapReduce 架構的額外引數 (例如：mapreduce.job.tags)。 若要區分您的引數與 MapReduce 引數，請考慮同時使用選項和值作為引數，如下列範例所示 (-s、--input、--output 等等是後面接著其值的選項)。

    ```JSON   
    {
        "name": "MahoutMapReduceSamplePipeline",
        "properties": {
            "description": "Sample Pipeline to Run a Mahout Custom Map Reduce Jar. This job calcuates an Item Similarity Matrix to determine the similarity between 2 items",
            "activities": [
                {
                    "type": "HDInsightMapReduce",
                    "typeProperties": {
                        "className": "org.apache.mahout.cf.taste.hadoop.similarity.item.ItemSimilarityJob",
                        "jarFilePath": "adfsamples/Mahout/jars/mahout-examples-0.9.0.2.2.7.1-34.jar",
                        "jarLinkedService": "StorageLinkedService",
                        "arguments": [
                            "-s",
                            "SIMILARITY_LOGLIKELIHOOD",
                            "--input",
                            "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/input",
                            "--output",
                            "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/output/",
                            "--maxSimilaritiesPerItem",
                            "500",
                            "--tempDir",
                            "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/temp/mahout"
                        ]
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
            "start": "2017-01-03T00:00:00Z",
            "end": "2017-01-04T00:00:00Z"
        }
    }
    ```
您可以使用「HDInsight MapReduce 活動」，在 HDInsight 叢集上執行任何 MapReduce Jar 檔案。 在下列管線的範例 JSON 定義中，已設定讓「HDInsight 活動」執行 Mahout JAR 檔案。

## <a name="sample-on-github"></a>GitHub 上的範例
您可以從 [GitHub 上的 Data Factory 範例](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/JSON/MapReduce_Activity_Sample)下載使用「HDInsight MapReduce 活動」的範例。  

## <a name="running-the-word-count-program"></a>執行字數統計程式
本範例中的管線會在 Azure HDInsight 叢集上執行字數統計 Map/Reduce 程式。   

### <a name="linked-services"></a>連結的服務
首先，建立連結的服務，將 Azure HDInsight 叢集使用的 Azure 儲存體連結到  Azure Data Factory。 如果您複製/貼上下列程式碼，請記得使用 Azure 儲存體的名稱和金鑰來取代**帳戶名稱**和**帳戶金鑰**。 

#### <a name="azure-storage-linked-service"></a>Azure 儲存體連結服務

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
接著，建立連結的服務，將 Azure HDInsight 叢集連結到 Azure Data Factory。 如果您複製/貼上下列程式碼，請使用您的 HDInsight 叢集的名稱來取代 **HDInsight 叢集名稱** ，並變更使用者名稱和密碼值。   

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
此範例中的管線不需要取得任何輸入。 您需指定「HDInsight MapReduce 活動」的輸出資料集。 這個資料集只是一個驅動管線排程所需的虛設資料集。  

```JSON
{
    "name": "MROutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "fileName": "WordCountOutput1.txt",
            "folderPath": "example/data/",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

### <a name="pipeline"></a>管線
此範例中的管線只含有一個類型為 HDInsightMapReduce 的活動。 JSON 中的幾個重要屬性如下： 

| 屬性 | 注意 |
|:--- |:--- |
| type |類型必須設為 **HDInsightMapReduce**。 |
| className |類別的名稱是： **wordcount** |
| jarFilePath |包含類別的 Jar 檔案路徑。 如果您複製/貼上下列程式碼，請記得變更叢集的名稱。 |
| jarLinkedService |包含 Jar 檔案的 Azure 儲存體連結服務。 這個連結服務會參考與 HDInsight 叢集關聯的儲存體。 |
| arguments |字數統計程式會採用輸入和輸出兩個引數。 輸入檔為 davinci.txt 檔案。 |
| frequency/interval |這些屬性的值符合輸出資料集。 |
| 預設容器 |表示您先前建立的 HDInsight 連結服務。 |

```JSON
{
    "name": "MRSamplePipeline",
    "properties": {
        "description": "Sample Pipeline to Run the Word Count Program",
        "activities": [
            {
                "type": "HDInsightMapReduce",
                "typeProperties": {
                    "className": "wordcount",
                    "jarFilePath": "<HDInsight cluster name>/example/jars/hadoop-examples.jar",
                    "jarLinkedService": "StorageLinkedService",
                    "arguments": [
                        "/example/data/gutenberg/davinci.txt",
                        "/example/data/WordCountOutput1"
                    ]
                },
                "outputs": [
                    {
                        "name": "MROutput"
                    }
                ],
                "policy": {
                    "timeout": "01:00:00",
                    "concurrency": 1,
                    "retry": 3
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "MRActivity",
                "linkedServiceName": "HDInsightLinkedService"
            }
        ],
        "start": "2014-01-03T00:00:00Z",
        "end": "2014-01-04T00:00:00Z"
    }
}
```

## <a name="run-spark-programs"></a>執行 Spark 程式
您可以使用 MapReduce 活動，在 HDInsight Spark 叢集上執行 Spark 程式。 如需詳細資訊，請參閱 [從 Azure Data Factory 叫用 Spark 程式](data-factory-spark.md) 。  

[developer-reference]: http://go.microsoft.com/fwlink/?LinkId=516908
[cmdlet-reference]: http://go.microsoft.com/fwlink/?LinkId=517456


[adfgetstarted]: data-factory-copy-data-from-azure-blob-storage-to-sql-database.md
[adfgetstartedmonitoring]:data-factory-copy-data-from-azure-blob-storage-to-sql-database.md#monitor-pipelines 

[Developer Reference]: http://go.microsoft.com/fwlink/?LinkId=516908
[Azure Portal]: http://portal.azure.com

## <a name="see-also"></a>另請參閱
* [Hive 活動](data-factory-hive-activity.md)
* [Pig 活動](data-factory-pig-activity.md)
* [Hadoop 串流活動](data-factory-hadoop-streaming-activity.md)
* [叫用 Spark 程式](data-factory-spark.md)
* [叫用 R 指令碼](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/RunRScriptUsingADFSample)

