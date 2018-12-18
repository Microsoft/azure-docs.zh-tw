---
title: 使用 Azure Data Factory 中的 Spark 轉換資料 | Microsoft Docs
description: 本教學課程提供逐步指示，說明如何使用 Azure Data Factory 中的 Spark 活動來轉換資料。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: douglasl
ms.openlocfilehash: c01da1b667f5a57e9597b77e21dcd9cc95340cb1
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43088221"
---
# <a name="transform-data-in-the-cloud-by-using-spark-activity-in-azure-data-factory"></a>使用 Azure Data Factory 中的 Spark 活動來轉換雲端中的資料
在本教學課程中，您會使用 Azure PowerShell 建立 Data Factory 管道，以使用 Spark 活動和隨選 HDInsight 連結服務來轉換資料。 您會在本教學課程中執行下列步驟：

> [!div class="checklist"]
> * 建立資料處理站。 
> * 撰寫和部署連結服務。
> * 撰寫和部署管道。 
> * 啟動管線執行。
> * 監視管道執行。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件
* **Azure 儲存體帳戶**。 您需要建立 python 指令碼和輸入檔案，並上傳至 Azure 儲存體。 spark 程式的輸出會儲存在這個儲存體帳戶中。 隨選 Spark 叢集與其主要儲存體是使用相同的儲存體帳戶。  
* **Azure PowerShell**(英文)。 遵循[如何安裝並設定 Azure PowerShell](/powershell/azure/install-azurerm-ps) 中的指示。


### <a name="upload-python-script-to-your-blob-storage-account"></a>將 python 指令碼上傳至 Blob 儲存體帳戶
1. 使用下列內容建立名為 **WordCount_Spark.py** 的 python 檔案： 

    ```python
    import sys
    from operator import add
    
    from pyspark.sql import SparkSession
    
    def main():
        spark = SparkSession\
            .builder\
            .appName("PythonWordCount")\
            .getOrCreate()
            
        lines = spark.read.text("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/inputfiles/minecraftstory.txt").rdd.map(lambda r: r[0])
        counts = lines.flatMap(lambda x: x.split(' ')) \
            .map(lambda x: (x, 1)) \
            .reduceByKey(add)
        counts.saveAsTextFile("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/outputfiles/wordcount")
        
        spark.stop()
    
    if __name__ == "__main__":
        main()
    ```
2. 以您的 Azure 儲存體帳戶名稱取代 **&lt;storageAccountName&gt;**。 然後儲存檔案。 
3. 在 Azure Blob 儲存體中，建立名為 **adftutorial** 的容器 (如果不存在)。 
4. 建立名為 **spark** 的資料夾。
5. 在 **spark** 資料夾下，建立名為 **script** 的子資料夾。 
6. 將 **WordCount_Spark.py** 檔案上傳至 **script** 子資料夾。 


### <a name="upload-the-input-file"></a>上傳輸入檔案
1. 建立名為 **minecraftstory.txt** 的檔案並填入一些文字。 Spark 程式會計算這段文字中的字數。 
2. 在 `spark` 資料夾中建立名為 `inputfiles` 的子資料夾。 
3. 將 `minecraftstory.txt` 上傳至 `inputfiles` 子資料夾。 

## <a name="author-linked-services"></a>撰寫連結服務
在本節中，您會撰寫兩個連結服務： 
    
- 將 Azure 儲存體帳戶連結至資料處理站的 Azure 儲存體連結服務。 隨選 HDInsight 叢集會使用此儲存體。 它也包含要執行的 Spark 指令碼。 
- 隨選 HDInsight 連結服務。 Azure Data Factory 會自動建立 HDInsight 叢集、執行 Spark 程式，然後刪除已閒置一段預先設定時間的 HDInsight 叢集。 

### <a name="azure-storage-linked-service"></a>Azure 儲存體連結服務
使用您慣用的編輯器建立 JSON 檔案、複製下列 Azure 儲存體連結服務的 JSON 定義，然後將檔案儲存為 **MyStorageLinkedService.json**。  

```json
{
    "name": "MyStorageLinkedService",
    "properties": {
      "type": "AzureStorage",
      "typeProperties": {
        "connectionString": {
          "value": "DefaultEndpointsProtocol=https;AccountName=<storageAccountName>;AccountKey=<storageAccountKey>",
          "type": "SecureString"
        }
      }
    }
}
```
以您的 Azure 儲存體帳戶名稱和金鑰更新 &lt;storageAccountName&gt; 和 &lt;storageAccountKey&gt;。 


### <a name="on-demand-hdinsight-linked-service"></a>隨選 HDInsight 連結服務
使用您慣用的編輯器建立 JSON 檔案、複製下列 Azure HDInsight 連結服務的 JSON 定義，然後將檔案儲存為 **MyOnDemandSparkLinkedService.json**。  

```json
{
    "name": "MyOnDemandSparkLinkedService",
    "properties": {
      "type": "HDInsightOnDemand",
      "typeProperties": {
        "clusterSize": 2,
        "clusterType": "spark",
        "timeToLive": "00:15:00",
        "hostSubscriptionId": "<subscriptionID> ",
        "servicePrincipalId": "<servicePrincipalID>",
        "servicePrincipalKey": {
          "value": "<servicePrincipalKey>",
          "type": "SecureString"
        },
        "tenant": "<tenant ID>",
        "clusterResourceGroup": "<resourceGroupofHDICluster>",
        "version": "3.6",
        "osType": "Linux",
        "clusterNamePrefix":"ADFSparkSample",
        "linkedServiceName": {
          "referenceName": "MyStorageLinkedService",
          "type": "LinkedServiceReference"
        }
      }
    }
}
```
在連結服務定義中更新下列屬性的值： 

- **hostSubscriptionId**。 以您的 Azure 訂用帳戶識別碼取代 &lt;subscriptionID&gt;。 隨選 HDInsight 叢集會在此訂用帳戶中建立。 
- **tenant**. 以您的 Azure 租用戶識別碼取代 &lt;tenantID&gt;。 
- **servicePrincipalId**、**servicePrincipalKey**。 以您在 Azure Active Directory 中的服務主體識別碼與金鑰，取代 &lt;servicePrincipalID&gt; 和 &lt;servicePrincipalKey&gt;。 此服務主體必須是訂用帳戶之參與者角色的成員，或其中建立叢集之資源群組的成員。 如需詳細資料，請參閱[建立 Azure Active Directory 應用程式和服務主體](../azure-resource-manager/resource-group-create-service-principal-portal.md)。 
- **clusterResourceGroup**。 以需要在其中建立 HDInsight 叢集的資源群組名稱，取代&lt;resourceGroupOfHDICluster&gt;。 

> [!NOTE]
> Azure HDInsight 對您在其支援的每個 Azure 區域中可使用的核心總數有所限制。 對於隨選 HDInsight 連結服務，建立 HDInsight 叢集的位置與作為其主要儲存體之 Azure 儲存體的位置相同。 請確定您有足夠的核心配額，才能成功建立叢集。 如需詳細資訊，請參閱[使用 Hadoop、Spark 及 Kafka 等在 HDInsight 中設定叢集](../hdinsight/hdinsight-hadoop-provision-linux-clusters.md)。 


## <a name="author-a-pipeline"></a>撰寫管道 
在此步驟中，您會建立具有 Spark 活動的新管道。 此活動使用**字數統計**範例。 從這個位置下載內容 (如果尚未這樣做)。

在您慣用的編輯器中建立 JSON 檔案、複製下列管道定義 JSON 定義，然後儲存為 **MySparkOnDemandPipeline.json**。 

```json
{
  "name": "MySparkOnDemandPipeline",
  "properties": {
    "activities": [
      {
        "name": "MySparkActivity",
        "type": "HDInsightSpark",
        "linkedServiceName": {
            "referenceName": "MyOnDemandSparkLinkedService",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
          "rootPath": "adftutorial/spark",
          "entryFilePath": "script/WordCount_Spark.py",
          "getDebugInfo": "Failure",
          "sparkJobLinkedService": {
            "referenceName": "MyStorageLinkedService",
            "type": "LinkedServiceReference"
          }
        }
      }
    ]
  }
}
```

請注意下列幾點： 

- rootPath 指向 adftutorial 容器的 spark 資料夾。 
- entryFilePath 指向 spark 資料夾的 script 子資料夾中的 WordCount_Spark.py 檔案。 


## <a name="create-a-data-factory"></a>建立 Data Factory 
您已在 JSON 檔案中撰寫連結服務和管道定義。 現在，讓我們使用 PowerShell Cmdlet 來建立資料處理站，並部署連結服務和管道 JSON 檔案。 逐一執行下列 PowerShell 命令： 

1. 逐一設定變數。

    **資源群組名稱**
    ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup" 
    ```

    **Data Factory 名稱。必須是全域唯一的** 
    ```powershell
    $dataFactoryName = "MyDataFactory09102017"
    ```
    
    **管線名稱**
    ```powershell
    $pipelineName = "MySparkOnDemandPipeline" # Name of the pipeline
    ```
2. 啟動 **PowerShell**。 保持開啟 Azure PowerShell，直到本快速入門結束為止。 如果您關閉並重新開啟，則需要再次執行這些命令。 如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析] 以找出 [Data Factory]：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。

    執行下列命令，並輸入您用來登入 Azure 入口網站的使用者名稱和密碼：
        
    ```powershell
    Connect-AzureRmAccount
    ```        
    執行下列命令以檢視此帳戶的所有訂用帳戶：

    ```powershell
    Get-AzureRmSubscription
    ```
    執行下列命令以選取您要使用的訂用帳戶。 以您的 Azure 訂用帳戶識別碼取代 **SubscriptionId**：

    ```powershell
    Select-AzureRmSubscription -SubscriptionId "<SubscriptionId>"    
    ```  
3. 建立資源群組：ADFTutorialResourceGroup。 

    ```powershell
    New-AzureRmResourceGroup -Name $resourceGroupName -Location "East Us" 
    ```
4. 建立資料處理站。 

    ```powershell
     $df = Set-AzureRmDataFactoryV2 -Location EastUS -Name $dataFactoryName -ResourceGroupName $resourceGroupName
    ```

    執行下列命令來查看輸出： 

    ```powershell
    $df
    ```
5. 切換至您建立 JSON 檔案的資料夾，然後執行下列命令來部署 Azure 儲存體連結服務： 
       
    ```powershell
    Set-AzureRmDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "MyStorageLinkedService" -File "MyStorageLinkedService.json"
    ```
6. 執行下列命令來部署隨選 Spark 連結服務： 
       
    ```powershell
    Set-AzureRmDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "MyOnDemandSparkLinkedService" -File "MyOnDemandSparkLinkedService.json"
    ```
7. 執行下列命令來部署管道： 
       
    ```powershell
    Set-AzureRmDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name $pipelineName -File "MySparkOnDemandPipeline.json"
    ```
    
## <a name="start-and-monitor-a-pipeline-run"></a>啟動及監視管道執行  

1. 啟動管道執行。 它也會擷取管道執行識別碼，方便後續監視。

    ```powershell
    $runId = Invoke-AzureRmDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName $pipelineName  
    ```
2. 執行下列程式碼以持續檢查管道執行狀態，直到完成為止。

    ```powershell
    while ($True) {
        $result = Get-AzureRmDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)
    
        if(!$result) {
            Write-Host "Waiting for pipeline to start..." -foregroundcolor "Yellow"
        }
        elseif (($result | Where-Object { $_.Status -eq "InProgress" } | Measure-Object).count -ne 0) {
            Write-Host "Pipeline run status: In Progress" -foregroundcolor "Yellow"
        }
        else {
            Write-Host "Pipeline '"$pipelineName"' run finished. Result:" -foregroundcolor "Yellow"
            $result
            break
        }
        ($result | Format-List | Out-String)
        Start-Sleep -Seconds 15
    }

    Write-Host "Activity `Output` section:" -foregroundcolor "Yellow"
    $result.Output -join "`r`n"

    Write-Host "Activity `Error` section:" -foregroundcolor "Yellow"
    $result.Error -join "`r`n" 
    ```  
3. 執行範例的輸出如下： 

    ```
    Pipeline run status: In Progress
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : 
    ActivityName      : MySparkActivity
    PipelineRunId     : 94e71d08-a6fa-4191-b7d1-cf8c71cb4794
    PipelineName      : MySparkOnDemandPipeline
    Input             : {rootPath, entryFilePath, getDebugInfo, sparkJobLinkedService}
    Output            : 
    LinkedServiceName : 
    ActivityRunStart  : 9/20/2017 6:33:47 AM
    ActivityRunEnd    : 
    DurationInMs      : 
    Status            : InProgress
    Error             :
    …
    
    Pipeline ' MySparkOnDemandPipeline' run finished. Result:
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : MyDataFactory09102017
    ActivityName      : MySparkActivity
    PipelineRunId     : 94e71d08-a6fa-4191-b7d1-cf8c71cb4794
    PipelineName      : MySparkOnDemandPipeline
    Input             : {rootPath, entryFilePath, getDebugInfo, sparkJobLinkedService}
    Output            : {clusterInUse, jobId, ExecutionProgress, effectiveIntegrationRuntime}
    LinkedServiceName : 
    ActivityRunStart  : 9/20/2017 6:33:47 AM
    ActivityRunEnd    : 9/20/2017 6:46:30 AM
    DurationInMs      : 763466
    Status            : Succeeded
    Error             : {errorCode, message, failureType, target}
    
    Activity Output section:
    "clusterInUse": "https://ADFSparkSamplexxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.azurehdinsight.net/"
    "jobId": "0"
    "ExecutionProgress": "Succeeded"
    "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)"
    Activity Error section:
    "errorCode": ""
    "message": ""
    "failureType": ""
    "target": "MySparkActivity"
    ```
4. 根據 spark 程式的輸出，確認 adftutorial 容器的 `spark`資料夾中已建立名為 `outputfiles` 的資料夾。 


## <a name="next-steps"></a>後續步驟
在此範例中的管線會將資料從 Azure Blob 儲存體中的一個位置複製到其他位置。 您已了解如何︰ 

> [!div class="checklist"]
> * 建立資料處理站。 
> * 撰寫和部署連結服務。
> * 撰寫和部署管道。 
> * 啟動管線執行。
> * 監視管道執行。

進入下一個教學課程，以了解如何在虛擬網路中的 Azure HDInsight 叢集上執行 Hive 指令碼，以轉換資料。 

> [!div class="nextstepaction"]
> [教學課程：在 Azure 虛擬網路中使用 Hive 來轉換資料](tutorial-transform-data-hive-virtual-network.md)。





