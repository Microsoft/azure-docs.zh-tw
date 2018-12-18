---
title: 使用 Azure Data Factory 中的 Hadoop MapReduce 活動轉換資料 | Microsoft Docs
description: 了解如何從 Azure Data Factory，在 Azure HDInsight 叢集上執行 Hadoop MapReduce 程式以處理資料。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/16/2018
ms.author: douglasl
ms.openlocfilehash: cb7009d0e7f31b2f503ac51d378fd117fff9f9b2
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37049946"
---
# <a name="transform-data-using-hadoop-mapreduce-activity-in-azure-data-factory"></a>使用 Azure Data Factory 中的 Hadoop MapReduce 活動轉換資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-map-reduce.md)
> * [目前的版本](transform-data-using-hadoop-map-reduce.md)

Data Factory [管線](concepts-pipelines-activities.md) 中的 HDInsight MapReduce 活動會在[您自己](compute-linked-services.md#azure-hdinsight-linked-service)或[隨選](compute-linked-services.md#azure-hdinsight-on-demand-linked-service)的 HDInsight 叢集上叫用 MapReduce 程式。 本文是以[資料轉換活動](transform-data.md)一文為基礎而建置，它呈現資料轉換和支援的轉換活動的一般概觀。

如果您是 Azure Data Factory 的新手，請在閱讀本文之前先閱讀 [Azure Data Factory 簡介](introduction.md)，以及研習教學課程：[教學課程：轉換資料](tutorial-transform-data-spark-powershell.md)。 

若要了解如何使用 HDInsight 的 Pig 和 Hive 活動，在 HDInsight 叢集上從管線執行 Pig/Hive 指令碼，請參閱 [Pig](transform-data-using-hadoop-pig.md) 和 [Hive](transform-data-using-hadoop-hive.md)。 

## <a name="syntax"></a>語法

```json
{
    "name": "Map Reduce Activity",
    "description": "Description",
    "type": "HDInsightMapReduce",
    "linkedServiceName": {
        "referenceName": "MyHDInsightLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "className": "org.myorg.SampleClass",
        "jarLinkedService": {
            "referenceName": "MyAzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "jarFilePath": "MyAzureStorage/jars/sample.jar",
        "getDebugInfo": "Failure",
        "arguments": [
          "-SampleHadoopJobArgument1"
        ],
        "defines": {
          "param1": "param1Value"
        }
    }
}
```

## <a name="syntax-details"></a>語法詳細資料

| 屬性          | 說明                              | 必要 |
| ----------------- | ---------------------------------------- | -------- |
| name              | 活動的名稱                     | yes      |
| 說明       | 說明活動用途的文字 | 否       |
| type              | 對於 MapReduce 活動，活動類型為 HDinsightMapReduce | yes      |
| 預設容器 | 參考 HDInsight 叢集註冊為 Data Factory 中的連結服務。 若要深入了解此已連結的服務，請參閱[計算已連結的服務](compute-linked-services.md)一文。 | yes      |
| className         | 要執行的類別名稱         | yes      |
| jarLinkedService  | Azure 儲存體連結服務用來儲存 Jar 檔案的參考。 如果您未指定這項連結服務，則會使用 HDInsight 已連結的服務中定義的 Azure 儲存體已連結的服務。 | 否       |
| jarFilePath       | 提供儲存在 jarLinkedService 引用之 Azure 儲存體中 Jar 檔案的路徑。 檔案名稱有區分大小寫。 | yes      |
| jarlibs           | 儲存在 jarLinkedService 引用之 Azure 儲存體中作業所參考的 Jar 程式庫檔路徑字串陣列。 檔案名稱有區分大小寫。 | 否       |
| getDebugInfo      | 指定何時將記錄檔複製到 HDInsight 叢集所使用 (或) jarLinkedService 所指定的 Azure 儲存體。 允許的值︰None、Always 或 Failure。 預設值：None。 | 否       |
| arguments         | 指定 Hadoop 作業的引數陣列。 引數會以命令列引數的方式傳遞給每項工作。 | 否       |
| 定義           | 指定參數作為機碼/值組，以供在 Hive 指令碼內參考。 | 否       |



## <a name="example"></a>範例
您可以使用「HDInsight MapReduce 活動」，在 HDInsight 叢集上執行任何 MapReduce Jar 檔案。 在下列管線的範例 JSON 定義中，已設定讓「HDInsight 活動」執行 Mahout JAR 檔案。

```json   
{
    "name": "MapReduce Activity for Mahout",
    "description": "Custom MapReduce to generate Mahout result",
    "type": "HDInsightMapReduce",
    "linkedServiceName": {
        "referenceName": "MyHDInsightLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "className": "org.apache.mahout.cf.taste.hadoop.similarity.item.ItemSimilarityJob",
        "jarLinkedService": {
            "referenceName": "MyStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "jarFilePath": "adfsamples/Mahout/jars/mahout-examples-0.9.0.2.2.7.1-34.jar",
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
    }
}
```
您可以在 **arguments** 區段中，為 MapReduce 程式指定所有引數。 在執行階段，您會看到幾個來自 MapReduce 架構的額外引數 (例如：mapreduce.job.tags)。 若要區分您的引數與 MapReduce 引數，請考慮同時使用選項和值作為引數，如下列範例所示 (-s、--input、--output 等等是後面接著其值的選項)。

## <a name="next-steps"></a>後續步驟
請參閱下列文章，其說明如何以其他方式轉換資料： 

* [U-SQL 活動](transform-data-using-data-lake-analytics.md)
* [Hive 活動](transform-data-using-hadoop-hive.md)
* [Pig 活動](transform-data-using-hadoop-pig.md)
* [Hadoop 串流活動](transform-data-using-hadoop-streaming.md)
* [Spark 活動](transform-data-using-spark.md)
* [.NET 自訂活動](transform-data-using-dotnet-custom-activity.md)
* [Machine Learning 批次執行活動](transform-data-using-machine-learning.md)
* [預存程序活動](transform-data-using-stored-procedure.md)
