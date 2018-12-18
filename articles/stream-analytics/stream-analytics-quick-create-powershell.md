---
title: 使用 Azure PowerShell 建立串流分析作業
description: 本快速入門詳細說明如何使用 Azure PowerShell 模組來部署和執行 Azure 串流分析作業。
services: stream-analytics
author: sidramadoss
ms.author: sidram
ms.date: 05/14/2018
ms.topic: quickstart
ms.service: stream-analytics
ms.custom: mvc
manager: kfile
ms.openlocfilehash: 18903dfbe187de73a6edb14196fa29e02d35dbca
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39185375"
---
# <a name="quickstart-create-a-stream-analytics-job-by-using-azure-powershell"></a>快速入門：使用 Azure PowerShell 建立串流分析作業

使用 Azure PowerShell 模組，即可利用 PowerShell Cmdlet 或指令碼建立和管理 Azure 資源。 本快速入門詳細說明如何使用 Azure PowerShell 模組來部署和執行 Azure 串流分析作業。 
 
範例作業會從 Azure blob 儲存體中的 blob 讀取串流資料。 本快速入門中所使用的輸入資料檔案包含僅供說明之用的靜態資料。 在實際案例中，您可以使用串流輸入資料來進行串流分析作業。 接下來，作業會使用串流分析查詢語言來轉換資料，以計算超過 100° 的平均溫度。 最後，它會將產生的輸出事件寫入另一個檔案中。 

## <a name="before-you-begin"></a>開始之前

* 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/)。  

* 本快速入門需要 Azure PowerShell 模組 3.6 版或更新版本。 執行 `Get-Module -ListAvailable AzureRM` 來尋找本機電腦上所安裝的版本。 如果您需要安裝或升級，請參閱[安裝 Azure PowerShell 模組](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)一文。 


## <a name="sign-in-to-azure"></a>登入 Azure

使用 `Connect-AzureRmAccount` 命令登入 Azure 訂用帳戶，並在快顯瀏覽器中輸入您的 Azure 認證：

```powershell
# Log in to your Azure account
Connect-AzureRmAccount
```

登入之後，如果您有多個訂用帳戶，請執行下列 Cmdlet 以選取您想要用於本快速入門的訂用帳戶。 務必要以您的訂用帳戶名稱取代 <your subscription name>：  

```powershell
# List all available subscriptions.
Get-AzureRmSubscription

# Select the Azure subscription you want to use to create the resource group and resources.
Get-AzureRmSubscription -SubscriptionName "<your subscription name>" | Select-AzureRmSubscription
```

## <a name="create-a-resource-group"></a>建立資源群組

使用 [New-AzureRmResourceGroup](https://docs.microsoft.com/powershell/module/azurerm.resources/new-azurermresourcegroup) 建立 Azure 資源群組。 資源群組是在其中部署與管理 Azure 資源的邏輯容器。

```powershell
$resourceGroup = "StreamAnalyticsRG"
$location = "WestUS2"
New-AzureRmResourceGroup `
   -Name $resourceGroup `
   -Location $location 
```

## <a name="prepare-the-input-data"></a>準備輸入資料

定義串流分析作業前，先準備設定為作業輸入的資料。

1. 從 GitHub 下載[感應器資料範例](https://raw.githubusercontent.com/Azure/azure-stream-analytics/master/Samples/GettingStarted/HelloWorldASA-InputStream.json)。 以滑鼠右鍵按一下連結和 [另存連結為...] 或 [另存目標為]。

2. 下列 PowerShell 程式碼區塊會執行多個命令，以準備作業所需的輸入資料。 檢閱區段以了解程式碼。 

   1. 使用 [New-AzureRmStorageAccount](https://docs.microsoft.com/powershell/module/azurerm.storage/New-AzureRmStorageAccount) Cmdlet 來建立標準的一般用途儲存體帳戶。  這個範例會建立名為 mystorageaccount 的儲存體帳戶，其具有本機備援儲存體 (LRS) 和 blob 加密 (預設為啟用)。  

   2. 取出儲存體帳戶內容 `$storageAccount.Context`，定義要使用的儲存體帳戶。 使用儲存體帳戶時，會參考內容而非重複提供認證。 

   3. 使用 [New-AzureStorageContainer](https://docs.microsoft.com/powershell/module/azure.storage/new-azurestoragecontainer) 建立儲存體容器，以及上傳您稍早下載的[感應器資料範例](https://github.com/Azure/azure-stream-analytics/blob/master/Samples/GettingStarted/HelloWorldASA-InputStream.json)。 

   4. 複製程式碼所輸出的儲存體金鑰，並在 JSON 檔案中貼上該金鑰，以便稍後建立串流作業的輸入和輸出。

   ```powershell
   $storageAccountName = "mystorageaccount"
   $storageAccount = New-AzureRmStorageAccount `
     -ResourceGroupName $resourceGroup `
     -Name $storageAccountName `
     -Location $location `
     -SkuName Standard_LRS `
     -Kind Storage
   
   $ctx = $storageAccount.Context
   $containerName = "streamanalytics"
   
   New-AzureStorageContainer `
     -Name $containerName `
     -Context $ctx
   
   Set-AzureStorageBlobContent `
     -File "c:\HelloWorldASA-InputStream.json" `
     -Blob "input/HelloWorldASA-InputStream.json" `
     -Container $containerName `
     -Context $ctx  
   
   $storageAccountKey = (Get-AzureRmStorageAccountKey `
     -ResourceGroupName $resourceGroup `
     -Name $storageAccountName).Value[0]
   
   Write-Host "The <storage account key> placeholder needs to be replaced in your input and output json files with this key value:" 
   Write-Host $storageAccountKey -ForegroundColor Cyan
   ```

## <a name="create-a-stream-analytics-job"></a>建立串流分析作業

使用 [New-AzureRmStreamAnalyticsJob](https://docs.microsoft.com/powershell/module/azurerm.streamanalytics/new-azurermstreamanalyticsjob?view=azurermps-5.4.0) Cmdlet 建立串流分析作業。 此 Cmdlet 會採用作業名稱、資源群組名稱和作業定義來作為參數。 作業名稱可以是任何可識別作業的易記名稱。 只可包含英數字元、連字號與底線，且其長度必須介於 3 到 63 個字元之間。 作業定義是 JSON 檔案，其中包含建立作業所需的屬性。 在本機電腦上，建立名為 `JobDefinition.json` 的檔案，並於其中新增下列 JSON 資料：

```json
{    
   "location":"WestUS2",  
   "properties":{    
      "sku":{    
         "name":"standard"  
      },  
      "eventsOutOfOrderPolicy":"adjust",  
      "eventsOutOfOrderMaxDelayInSeconds":10,  
      "compatibilityLevel": 1.1
   }
}
```

接下來，執行 `New-AzureRmStreamAnalyticsJob` Cmdlet。 務必將 `jobDefinitionFile` 變數的值，取代為作業定義 JSON 檔案儲存所在的路徑。 

```powershell
$jobName = "MyStreamingJob"
$jobDefinitionFile = "C:\JobDefinition.json"
New-AzureRmStreamAnalyticsJob `
  -ResourceGroupName $resourceGroup `
  -File $jobDefinitionFile `
  -Name $jobName `
  -Force 
```

## <a name="configure-input-to-the-job"></a>設定作業的輸入

使用 [New-AzureRmStreamAnalyticsInput](https://docs.microsoft.com/powershell/module/azurerm.streamanalytics/new-azurermstreamanalyticsinput?view=azurermps-5.4.0) Cmdlet 新增作業的輸入。 此 Cmdlet 會採用作業名稱、作業輸入名稱、資源群組名稱和作業輸入定義來作為參數。 作業輸入定義是 JSON 檔案，其中包含設定作業輸入所需的屬性。 在此範例中，您將建立 blob 儲存體作為輸入。 

在本機電腦上，建立名為 `JobInputDefinition.json` 的檔案，並於其中新增下列 JSON 資料。 務必要將 `accountKey` 的值取代為儲存體帳戶的存取金鑰 (也就是 $storageAccountKey 值內所儲存的值)。 

```json
{
    "properties": {
        "type": "Stream",
        "datasource": {
            "type": "Microsoft.Storage/Blob",
            "properties": {
                "storageAccounts": [
                {
                   "accountName": "mystorageaccount",
                   "accountKey":"<storage account key>"
                }],
                "container": "streamanalytics",
                "pathPattern": "input/",
                "dateFormat": "yyyy/MM/dd",
                "timeFormat": "HH"
            }
        },
        "serialization": {
            "type": "Json",
            "properties": {
                "encoding": "UTF8"
            }
        }
    },
    "name": "MyBlobInput",
    "type": "Microsoft.StreamAnalytics/streamingjobs/inputs"
}
```

接下來，執行 `New-AzureRmStreamAnalyticsInput` Cmdlet，務必將 `jobDefinitionFile` 變數的值，取代為作業輸入定義 JSON 檔案儲存所在的路徑。 

```powershell
$jobInputName = "MyBlobInput"
$jobInputDefinitionFile = "C:\JobInputDefinition.json"
New-AzureRmStreamAnalyticsInput `
  -ResourceGroupName $resourceGroup `
  -JobName $jobName `
  -File $jobInputDefinitionFile `
  -Name $jobInputName 
```

## <a name="configure-output-to-the-job"></a>設定作業的輸出

使用 [New-AzureRmStreamAnalyticsOutput](https://docs.microsoft.com/powershell/module/azurerm.streamanalytics/new-azurermstreamanalyticsoutput?view=azurermps-5.4.0) Cmdlet 新增作業的輸出。 此 Cmdlet 會採用作業名稱、作業輸出名稱、資源群組名稱和作業輸出定義來作為參數。 作業輸出定義是 JSON 檔案，其中包含設定作業輸出所需的屬性。 這個範例會使用 blob 儲存體作為輸出。 

在本機電腦上，建立名為 `JobOutputDefinition.json` 的檔案，並於其中新增下列 JSON 資料。 務必要將 `accountKey` 的值取代為儲存體帳戶的存取金鑰 (也就是 $storageAccountKey 值內所儲存的值)。 

```json
{
    "properties": {
        "datasource": {
            "type": "Microsoft.Storage/Blob",
            "properties": {
                "storageAccounts": [
                    {
                      "accountName": "mystorageaccount",
                      "accountKey": "<storage account key>"
                    }],
                "container": "streamanalytics",
                "pathPattern": "output/",
                "dateFormat": "yyyy/MM/dd",
                "timeFormat": "HH"
            }
        },
        "serialization": {
            "type": "Json",
            "properties": {
                "encoding": "UTF8",
                "format": "LineSeparated"
            }
        }
    },
    "name": "MyBlobOutput",
    "type": "Microsoft.StreamAnalytics/streamingjobs/outputs"
}
```

接下來，執行 `New-AzureRmStreamAnalyticsOutput` Cmdlet。 務必將 `jobOutputDefinitionFile` 變數的值，取代為作業輸出定義 JSON 檔案儲存所在的路徑。 

```powershell
$jobOutputName = "MyBlobOutput"
$jobOutputDefinitionFile = "C:\JobOutputDefinition.json"
New-AzureRmStreamAnalyticsOutput `
  -ResourceGroupName $resourceGroup `
  -JobName $jobName `
  -File $jobOutputDefinitionFile `
  -Name $jobOutputName -Force 
```

## <a name="define-the-transformation-query"></a>定義轉換查詢

使用 [New-AzureRmStreamAnalyticsTransformation](https://docs.microsoft.com/powershell/module/azurerm.streamanalytics/new-azurermstreamanalyticstransformation?view=azurermps-5.4.0) Cmdlet 新增作業的轉換。 此 Cmdlet 會採用作業名稱、作業轉換名稱、資源群組名稱和作業轉換定義來作為參數。 在本機電腦上，建立名為 `JobTransformationDefinition.json` 的檔案，並於其中新增下列 JSON 資料。 JSON 檔案包含可定義轉換查詢的查詢參數：

```json
{     
   "name":"MyTransformation",  
   "type":"Microsoft.StreamAnalytics/streamingjobs/transformations",  
   "properties":{    
      "streamingUnits":1,  
      "script":null,  
      "query":" SELECT System.Timestamp AS OutputTime, dspl AS SensorName, Avg(temp) AS AvgTemperature INTO MyBlobOutput FROM MyBlobInput TIMESTAMP BY time GROUP BY TumblingWindow(second,30),dspl HAVING Avg(temp)>100"  
   }  
}
```

接下來，執行 `New-AzureRmStreamAnalyticsTransformation` Cmdlet。 務必將 `jobTransformationDefinitionFile` 變數的值，取代為作業轉換定義 JSON 檔案儲存所在的路徑。 

```powershell
$jobTransformationName = "MyJobTransformation"
$jobTransformationDefinitionFile = "C:\JobTransformationDefinition.json"
New-AzureRmStreamAnalyticsTransformation `
  -ResourceGroupName $resourceGroup `
  -JobName $jobName `
  -File $jobTransformationDefinitionFile `
  -Name $jobTransformationName -Force
```

## <a name="start-the-stream-analytics-job-and-check-the-output"></a>啟動串流分析工作並查看輸出

使用 [Start-AzureRmStreamAnalyticsJob](https://docs.microsoft.com/powershell/module/azurerm.streamanalytics/start-azurermstreamanalyticsjob?view=azurermps-5.4.0) Cmdlet 啟動作業。 此 Cmdlet 會採用作業名稱、資源群組名稱、輸出啟動模式和啟動時間來作為參數。 `OutputStartMode` 可接受 `JobStartTime`、`CustomTime` 或 `LastOutputEventTime`。 若要了解這些值各自代表什麼，請參閱 PowerShell 文件中的[參數](https://docs.microsoft.com/powershell/module/azurerm.streamanalytics/start-azurermstreamanalyticsjob?view=azurermps-5.4.0)一節。 在此範例中，將模式指定為 `CustomTime`，並且為 `OutputStartTime` 提供值。 

針對時間值，選取 `2018-01-01`。 之所以選擇這個開始日期，是因為它位於範例資料的事件時間戳記之前。 執行下列 Cmdlet 後，如果作業啟動，它會在輸出中傳回 `True`。 在儲存體容器中，建立的輸出資料夾包含已轉換的資料。 

```powershell
Start-AzureRmStreamAnalyticsJob `
  -ResourceGroupName $resourceGroup `
  -Name $jobName `
  -OutputStartMode CustomTime `
  -OutputStartTime 2018-01-01T00:00:00Z 
```

## <a name="clean-up-resources"></a>清除資源

若不再需要，可刪除資源群組、串流作業和所有相關資源。 刪除作業可避免因為作業使用串流單位而產生費用。 如果您計劃在未來使用該作業，您可以跳過刪除動作，且目前停止作業。 如果您不會再繼續使用此作業，請執行下列 Cmdlet，以刪除本快速入門所建立的所有資源：

```powershell
Remove-AzureRmResourceGroup `
  -Name $resourceGroup 
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已部署了簡單的串流分析作業。 若要了解如何設定其他輸入來源及執行即時偵測，請前往下列文章：

> [!div class="nextstepaction"]
> [使用 Azure 串流分析進行即時詐騙偵測](stream-analytics-real-time-fraud-detection.md)
