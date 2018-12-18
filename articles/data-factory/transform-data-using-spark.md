---
title: 使用 Azure Data Factory 中的 Spark 活動轉換資料 | Microsoft Docs
description: 了解如何使用 Spark 活動從 Azure Data Factory 管線執行 Spark 程式以轉換資料。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/31/2018
ms.author: douglasl
ms.openlocfilehash: abe2fabc505f94f19d4b15a406fc59bf6d6e7ac1
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37050330"
---
# <a name="transform-data-using-spark-activity-in-azure-data-factory"></a>使用 Azure Data Factory 中的 Spark 活動轉換資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-spark.md)
> * [目前的版本](transform-data-using-spark.md)

Data Factory [管線](concepts-pipelines-activities.md)中的 Spark 活動會在[您自己的](compute-linked-services.md#azure-hdinsight-linked-service) HDInsight 叢集上或[隨選](compute-linked-services.md#azure-hdinsight-on-demand-linked-service)執行 Spark 程式。 本文是根據 [資料轉換活動](transform-data.md) 一文，它呈現資料轉換和支援的轉換活動的一般概觀。 當您使用隨選 Spark 連結的服務時，Data Factory 會自動為您建立 Spark 叢集 Just-In-Time 以處理資料，一旦處理完成之後就會刪除叢集。 

> [!IMPORTANT]
> Spark 活動不支援 Azure Data Lake Store 作為主要儲存體的 HDInsight Spark 叢集。

## <a name="spark-activity-properties"></a>Spark 活動屬性
以下是 Spark 活動的 JSON 定義範例：    

```json
{
    "name": "Spark Activity",
    "description": "Description",
    "type": "HDInsightSpark",
    "linkedServiceName": {
        "referenceName": "MyHDInsightLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "sparkJobLinkedService": {
            "referenceName": "MyAzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "rootPath": "adfspark\\pyFiles",
        "entryFilePath": "test.py",
        "sparkConfig": {
            "ConfigItem1": "Value"
        },
        "getDebugInfo": "Failure",
        "arguments": [
            "SampleHadoopJobArgument1"
        ]
    }
}
```

下表說明 JSON 定義中使用的 JSON 屬性：

| 屬性              | 說明                              | 必要 |
| --------------------- | ---------------------------------------- | -------- |
| name                  | 管線中的活動名稱。    | yes      |
| 說明           | 說明活動用途的文字。  | 否       |
| type                  | 對於 Spark 活動，活動類型為 HDInsightSpark。 | yes      |
| 預設容器     | Spark 程式執行所在的 HDInsight Spark 連結服務名稱。 若要深入了解此已連結的服務，請參閱[計算已連結的服務](compute-linked-services.md)一文。 | yes      |
| SparkJobLinkedService | 存放 Spark 作業檔案、相依性和記錄的 Azure 儲存體連結服務。  如果您未指定此屬性的值，則會使用與 HDInsight 叢集相關聯的儲存體。 這個屬性的值只能是 Azure 儲存體連結服務。 | 否       |
| rootPath              | Spark 檔案所在的 Azure Blob 容器和資料夾。 檔案名稱有區分大小寫。 如需了解此資料夾結構的詳細資料，請參閱資料夾結構一節 (下一節)。 | yes      |
| entryFilePath         | Spark 程式碼/套件之根資料夾的相對路徑。 輸入檔案必須是 Python 檔案或 .jar 檔案。 | yes      |
| className             | 應用程式的 Java/Spark 主要類別      | 否       |
| arguments             | Spark 程式的命令列引數清單。 | 否       |
| proxyUser             | 模擬來執行 Spark 程式的使用者帳戶 | 否       |
| sparkConfig           | 指定下列主題中所列的 Spark 組態屬性值：[Spark 組態 - 應用程式屬性](https://spark.apache.org/docs/latest/configuration.html#available-properties) (英文)。 | 否       |
| getDebugInfo          | 指定何時將 Spark 記錄檔複製到 HDInsight 叢集所使用 (或) sparkJobLinkedService 所指定的 Azure 儲存體。 允許的值︰None、Always 或 Failure。 預設值：None。 | 否       |

## <a name="folder-structure"></a>資料夾結構
Spark 作業比 Pig/Hive 作業更具擴充性。 對於 Spark 作業，您可以提供多個相依性，例如 jar 套件 (置於 java CLASSPATH)、python 檔案 (置於 PYTHONPATH) 和任何其他檔案。

在 HDInsight 連結服務所參考的 Azure Blob 儲存體中，建立下列資料夾結構。 然後，將相依檔案上傳至根資料夾中以 **entryFilePath** 表示的適當子資料夾。 比方說，將 python 檔案上傳至根資料夾的 pyFiles 子資料夾，將 jar 檔案上傳至 jars 子資料夾。 在執行階段，Data Factory 服務會預期 Azure Blob 儲存體中有下列資料夾結構︰     

| Path                  | 說明                              | 必要 | 類型   |
| --------------------- | ---------------------------------------- | -------- | ------ |
| `.` (root)            | Spark 作業在儲存體連結服務中的根路徑 | yes      | 資料夾 |
| &lt;使用者定義&gt; | 指向 Spark 作業輸入檔案的路徑 | yes      | 檔案   |
| ./jars                | 此資料夾下的所有檔案會上傳並放在叢集的 java 類別路徑 | 否       | 資料夾 |
| ./pyFiles             | 此資料夾下的所有檔案會上傳並放在叢集的 PYTHONPATH | 否       | 資料夾 |
| ./files               | 此資料夾下的所有檔案會上傳並放在執行程式工作目錄 | 否       | 資料夾 |
| ./archives            | 此資料夾下的所有檔案未壓縮 | 否       | 資料夾 |
| ./logs                | 此資料夾包含來自 Spark 叢集的記錄。 | 否       | 資料夾 |

以下是 HDInsight 連結服務所參考的 Azure Blob 儲存體中，含有兩個 Spark 作業檔案的儲存體範例。

```
SparkJob1
    main.jar
    files
        input1.txt
        input2.txt
    jars
        package1.jar
        package2.jar
    logs

SparkJob2
    main.py
    pyFiles
        scrip1.py
        script2.py
    logs
```
## <a name="next-steps"></a>後續步驟
請參閱下列文章，其說明如何以其他方式轉換資料： 

* [U-SQL 活動](transform-data-using-data-lake-analytics.md)
* [Hive 活動](transform-data-using-hadoop-hive.md)
* [Pig 活動](transform-data-using-hadoop-pig.md)
* [MapReduce 活動](transform-data-using-hadoop-map-reduce.md)
* [Hadoop 串流活動](transform-data-using-hadoop-streaming.md)
* [Spark 活動](transform-data-using-spark.md)
* [.NET 自訂活動](transform-data-using-dotnet-custom-activity.md)
* [Machine Learning 批次執行活動](transform-data-using-machine-learning.md)
* [預存程序活動](transform-data-using-stored-procedure.md)
