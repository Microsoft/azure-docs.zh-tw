---
title: 使用 Azure Data Factory 中的 Pig 活動轉換資料 | Microsoft Docs
description: 了解如何使用 Azure 資料處理站中的 Pig 活動，以在隨選/您自己的 HDInsight 叢集上執行 Pig 指令碼。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 5af07a1a-2087-455e-a67b-a79841b4ada5
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 5abd0b07037559b14158a3c314b6ca6ce30ab655
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37045111"
---
# <a name="transform-data-using-pig-activity-in-azure-data-factory"></a>使用 Azure Data Factory 中的 Pig 活動轉換資料
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
> 本文適用於 Data Factory 的第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[在 Data Factory 中使用 Pig 活動來轉換資料](../transform-data-using-hadoop-pig.md)。


Data Factory [管線](data-factory-create-pipelines.md)中的 HDInsight Pig 活動會在[您自己](data-factory-compute-linked-services.md#azure-hdinsight-linked-service)或[隨選](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service)的 Windows/Linux 架構 HDInsight 叢集上執行 Pig 查詢。 本文是根據 [資料轉換活動](data-factory-data-transformation-activities.md) 一文，它呈現資料轉換和支援的轉換活動的一般概觀。

> [!NOTE] 
> 如果您是 Azure Data Factory 的新手，請在閱讀本文章之前閱讀 [Azure Data Factory 簡介](data-factory-introduction.md)，以及進行教學課程：[建置您的第一個資料管線](data-factory-build-your-first-pipeline.md)。 

## <a name="syntax"></a>語法

```JSON
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
## <a name="syntax-details"></a>語法詳細資料
| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| name |活動的名稱 |yes |
| 說明 |說明活動用途的文字 |否 |
| type |HDInsightPig |yes |
| 輸入 |Pig 活動所取用的一或多項輸入 |否 |
| 輸出 |Pig 活動所產生的一或多項輸出 |yes |
| 預設容器 |參考 HDInsight 叢集註冊為 Data Factory 中的連結服務 |yes |
| script |指定 Pig 指令碼內嵌 |否 |
| 指令碼路徑 |在 Azure blob 儲存體中儲存 Pig 指令碼，並提供檔案的路徑。 使用 'script' 或 'scriptPath' 屬性。 兩者無法同時使用。 檔案名稱有區分大小寫。 |否 |
| 定義 |在使用 Pig 指令碼內指定參數做為參考的機碼/值組 |否 |

## <a name="example"></a>範例
讓我們思考一下一個遊戲記錄檔分析範例，在此範例中，您想要了解遊戲玩家花費多少時間玩貴公司所推出的遊戲。

下列範例遊戲記錄檔是一個逗點 (,) 分隔檔。 它包含下列欄位 – ProfileID、SessionStart、Duration、SrcIPAddress 及 GameType。

```
1809,2014-05-04 12:04:25.3470000,14,221.117.223.75,CaptureFlag
1703,2014-05-04 06:05:06.0090000,16,12.49.178.247,KingHill
1703,2014-05-04 10:21:57.3290000,10,199.118.18.179,CaptureFlag
1809,2014-05-04 05:24:22.2100000,23,192.84.66.141,KingHill
.....
```

用來處理此資料的「Pig 指令碼」  ：

```
PigSampleIn = LOAD 'wasb://adfwalkthrough@anandsub14.blob.core.windows.net/samplein/' USING PigStorage(',') AS (ProfileID:chararray, SessionStart:chararray, Duration:int, SrcIPAddress:chararray, GameType:chararray);

GroupProfile = Group PigSampleIn all;

PigSampleOut = Foreach GroupProfile Generate PigSampleIn.ProfileID, SUM(PigSampleIn.Duration);

Store PigSampleOut into 'wasb://adfwalkthrough@anandsub14.blob.core.windows.net/sampleoutpig/' USING PigStorage (',');
```

若要在 Data Factory 管線中執行此 Pig 指令碼，請執行下列步驟：

1. 建立連結服務以註冊[您自己的 HDInsight 計算叢集](data-factory-compute-linked-services.md#azure-hdinsight-linked-service)或設定[隨選 HDInsight 計算叢集](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service)。 讓我們將此連結服務命名為 **HDInsightLinkedService**。
2. 建立 [連結服務](data-factory-azure-blob-connector.md) 以設定裝載資料之 Azure Blob 儲存體的連接。 讓我們將此連結服務命名為 **StorageLinkedService**。
3. 建立指向輸入和輸出資料的 [資料集](data-factory-create-datasets.md) 。 讓我們將此輸出資料集命名為 **PigSampleIn**，以及將輸出資料集命名為 **PigSampleOut**。
4. 複製「Azure Blob 儲存體」在步驟 #2 所設定之檔案中的Pig 查詢。 如果裝載資料的 Azure 儲存體與裝載查詢檔案的儲存體不同，請建立個別的「Azure 儲存體」連結服務。 請參考活動組態中的連結服務。 使用 **scriptPath ** 來指定 Pig 指令碼檔案和 **scriptLinkedService** 的路徑。 
   
   > [!NOTE]
   > 您也可以使用 **script** 屬性，在活動定義中以內嵌方式提供 Pig 指令碼。 不過，不建議使用此方法，因為必須逸出指令碼中的所有特殊字元，而且可能造成偵錯問題。 最佳做法是遵循步驟 #4。
   > 
   > 
5. 建立具有 HDInsightPig 活動的管線。 此活動會透過在 HDInsight 叢集上執行 Pig 指令碼來處理輸入資料。

    ```JSON   
    {
      "name": "PigActivitySamplePipeline",
      "properties": {
        "activities": [
          {
            "name": "PigActivitySample",
            "type": "HDInsightPig",
            "inputs": [
              {
                "name": "PigSampleIn"
              }
            ],
            "outputs": [
              {
                "name": "PigSampleOut"
              }
            ],
            "linkedServiceName": "HDInsightLinkedService",
            "typeproperties": {
              "scriptPath": "adfwalkthrough\\scripts\\enrichlogs.pig",
              "scriptLinkedService": "StorageLinkedService"
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
6. 部署管線。 如需詳細資料，請參閱〈 [建立管線](data-factory-create-pipelines.md) 〉文章。 
7. 使用資料處理站監視和管理檢視來監視管線。 如需詳細資料，請參閱〈 [監視及管理 Data Factory 管線](data-factory-monitor-manage-pipelines.md) 〉文章。

## <a name="specifying-parameters-for-a-pig-script"></a>指定 Pig 指令碼的參數
請思考一下下列範例：遊戲記錄檔每天都會被擷取至「Azure Blob 儲存體」，並儲存在根據日期和時間分割的資料夾中。 您想要參數化 Pig 指令碼，在執行階段期間以動態方式傳遞輸入資料夾位置，並且產生使用日期和時間分割的輸出。

若要使用參數化 Pig 指令碼，請執行下列動作：

* 定義 **defines**中的參數。

    ```JSON  
    {
        "name": "PigActivitySamplePipeline",
          "properties": {
        "activities": [
            {
                "name": "PigActivitySample",
                "type": "HDInsightPig",
                "inputs": [
                      {
                        "name": "PigSampleIn"
                      }
                ],
                "outputs": [
                      {
                        "name": "PigSampleOut"
                      }
                ],
                "linkedServiceName": "HDInsightLinkedService",
                "typeproperties": {
                      "scriptPath": "adfwalkthrough\\scripts\\samplepig.hql",
                      "scriptLinkedService": "StorageLinkedService",
                      "defines": {
                        "Input": "$$Text.Format('wasb: //adfwalkthrough@<storageaccountname>.blob.core.windows.net/samplein/yearno={0: yyyy}/monthno={0:MM}/dayno={0: dd}/',SliceStart)",
                        "Output": "$$Text.Format('wasb://adfwalkthrough@<storageaccountname>.blob.core.windows.net/sampleout/yearno={0:yyyy}/monthno={0:MM}/dayno={0:dd}/', SliceStart)"
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
* 在 Pig 指令碼中，使用 '**$parameterName**' 參考參數，如下列範例所示：

    ```  
    PigSampleIn = LOAD '$Input' USING PigStorage(',') AS (ProfileID:chararray, SessionStart:chararray, Duration:int, SrcIPAddress:chararray, GameType:chararray);    
    GroupProfile = Group PigSampleIn all;        
    PigSampleOut = Foreach GroupProfile Generate PigSampleIn.ProfileID, SUM(PigSampleIn.Duration);        
    Store PigSampleOut into '$Output' USING PigStorage (','); 
    ```
## <a name="see-also"></a>另請參閱
* [Hive 活動](data-factory-hive-activity.md)
* [MapReduce 活動](data-factory-map-reduce.md)
* [Hadoop 串流活動](data-factory-hadoop-streaming-activity.md)
* [叫用 Spark 程式](data-factory-spark.md)
* [叫用 R 指令碼](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/RunRScriptUsingADFSample)

