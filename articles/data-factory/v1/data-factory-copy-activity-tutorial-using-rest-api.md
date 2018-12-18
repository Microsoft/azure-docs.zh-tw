---
title: 教學課程︰使用 REST API 建立 Azure Data Factory 管線 |Microsoft Docs
description: 在本教學課程中，您會使用 REST API 建立具有複製活動的 Azure Data Factory 管線，以將資料從 Azure Blob 儲存體複製到 Azure SQL Database。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: 1704cdf8-30ad-49bc-a71c-4057e26e7350
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 3bff6f00c67a2fa523113171855dce4e7b807f0b
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45734449"
---
# <a name="tutorial-use-rest-api-to-create-an-azure-data-factory-pipeline-to-copy-data"></a>教學課程︰使用 REST API 建立 Azure Data Factory 管線來複製資料 
> [!div class="op_single_selector"]
> * [概觀和必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [複製精靈](data-factory-copy-data-wizard-tutorial.md)
> * [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Azure Resource Manager 範本](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [.NET API](data-factory-copy-activity-tutorial-using-dotnet-api.md)
> 
> 

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[複製活動教學課程](../quickstart-create-data-factory-rest-api.md)。 

在本文中，您會了解如何使用 REST API 建立資料處理站，其中有管線可將資料從 Azure Blob 儲存體複製到 Azure SQL 資料庫。 如果您不熟悉 Azure Data Factory，請先詳閱 [Azure Data Factory 簡介](data-factory-introduction.md)一文，再進行本教學課程。   

在本教學課程中，您可以建立包含一個活動的管線：複製活動。 複製活動會將資料從支援的資料存放區複製到支援的接收資料存放區。 如需作為來源和接收區支援的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 此活動是由全域可用的服務所提供，可以使用安全、可靠及可調整的方式，在各種不同的資料存放區之間複製資料。 如需複製活動的詳細資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md)。

一個管線中可以有多個活動。 您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 如需詳細資訊，請參閱[管線中的多個活動](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)。

> [!NOTE]
> 這篇文章並未涵蓋所有的 Data Factory REST API。 如需 Data Factory Cmdlet 的完整文件，請參閱 [Data Factory REST API 參考](/rest/api/datafactory/) 。
>  
> 本教學課程中的資料管線會將資料從來源資料存放區，複製到目的地資料存放區。 如需如何使用 Azure Data Factory 轉換資料的教學課程，請參閱[教學課程︰使用 Hadoop 叢集建置管線來轉換資料](data-factory-build-your-first-pipeline.md)。

## <a name="prerequisites"></a>必要條件
* 請檢閱 [教學課程概觀](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) 並完成 **必要** 步驟。
* 在您的電腦上安裝 [Curl](https://curl.haxx.se/dlwiz/) 。 您可搭配使用 Curl 工具與 REST 命令來建立 Data Factory。 
* 請依照 [本文](../../azure-resource-manager/resource-group-create-service-principal-portal.md) 的指示： 
  1. 在 Azure Active Directory 中建立名為 **ADFCopyTutorialApp** 的 Web 應用程式。
  2. 取得**用戶端識別碼**和**秘密金鑰**。 
  3. 取得 **租用戶識別碼**。 
  4. 將 **ADFCopyTutorialApp** 應用程式指派給 **Data Factory 參與者**角色。  
* 安裝 [Azure PowerShell](/powershell/azure/overview)。  
* 啟動 **PowerShell** 並執行下列步驟。 將 Azure PowerShell 維持在開啟狀態，直到本教學課程結束為止。 如果您關閉並重新開啟，則需要再次執行這些命令。
  
  1. 執行下列命令並輸入您用來登入 Azure 入口網站的使用者名稱和密碼：
    
    ```PowerShell 
    Connect-AzureRmAccount
    ```   
  2. 執行下列命令以檢視此帳戶的所有訂用帳戶：

    ```PowerShell     
    Get-AzureRmSubscription
    ``` 
  3. 執行下列命令以選取您要使用的訂用帳戶。 以您的 Azure 訂用帳戶名稱取代 **&lt;NameOfAzureSubscription**&gt;。 
     
    ```PowerShell
    Get-AzureRmSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzureRmContext
    ```
  4. 在 PowerShell 中執行以下命令，建立名為 **ADFTutorialResourceGroup** 的 Azure 資源群組：  

    ```PowerShell     
      New-AzureRmResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```
     
      如果資源群組已存在，您可指定是否要更新 (Y) 或予以保留 (N)。 
     
      本教學課程的某些步驟假設您使用名為 ADFTutorialResourceGroup 的資源群組。 如果使用不同的資源群組，您必須以資源群組的名稱取代本教學課程中的 ADFTutorialResourceGroup。

## <a name="create-json-definitions"></a>建立 JSON 定義
在 curl.exe 所在的資料夾中建立下列 JSON 檔案。 

### <a name="datafactoryjson"></a>datafactory.json
> [!IMPORTANT]
> 名稱必須是全域唯一的，所以建議您使用 ADFCopyTutorialDF 做為前置詞/後置詞，使其成為唯一的名稱。 
> 
> 

```JSON
{  
    "name": "ADFCopyTutorialDF",  
    "location": "WestUS"
}  
```

### <a name="azurestoragelinkedservicejson"></a>azurestoragelinkedservice.json
> [!IMPORTANT]
> 以 Azure 儲存體帳戶的名稱和金鑰取代 **accountname** 和 **accountkey**。 若要了解如何取得儲存體存取金鑰，請參閱 [檢視、複製和重新產生儲存體存取金鑰](../../storage/common/storage-account-manage.md#access-keys)

```JSON
{
    "name": "AzureStorageLinkedService",
    "properties": {
        "type": "AzureStorage",
        "typeProperties": {
            "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
    }
}
```

如需 JSON 屬性的詳細資訊，請參閱 [Azure 儲存體連結服務](data-factory-azure-blob-connector.md#azure-storage-linked-service)。

### <a name="azuersqllinkedservicejson"></a>azuersqllinkedservice.json
> [!IMPORTANT]
> 將 **servername**、**databasename**、**username** 和 **password** 替換為您的 Azure SQL Server 名稱、SQL Database 名稱、使用者帳戶及帳戶的密碼。  
> 
>

```JSON
{
    "name": "AzureSqlLinkedService",
    "properties": {
        "type": "AzureSqlDatabase",
        "description": "",
        "typeProperties": {
            "connectionString": "Data Source=tcp:<servername>.database.windows.net,1433;Initial Catalog=<databasename>;User ID=<username>;Password=<password>;Integrated Security=False;Encrypt=True;Connect Timeout=30"
        }
    }
}
```

如需 JSON 屬性的詳細資訊，請參閱 [Azure SQL 連結服務](data-factory-azure-sql-connector.md#linked-service-properties)。

### <a name="inputdatasetjson"></a>inputdataset.json

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "structure": [
      {
        "name": "FirstName",
        "type": "String"
      },
      {
        "name": "LastName",
        "type": "String"
      }
    ],
    "type": "AzureBlob",
    "linkedServiceName": "AzureStorageLinkedService",
    "typeProperties": {
      "folderPath": "adftutorial/",
      "fileName": "emp.txt",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ","
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

下表提供程式碼片段中所使用之 JSON 屬性的描述：

| 屬性 | 說明 |
|:--- |:--- |
| type | type 屬性會設為 **AzureBlob**，因為資料位於 Azure Blob 儲存體中。 |
| 預設容器 | 表示您稍早建立的 **AzureStorageLinkedService**。 |
| folderPath | 指定包含輸入 Blob 的 Blob **容器**和**資料夾**。 在本教學課程中，adftutorial 是 blob 容器，而資料夾是根資料夾。 | 
| fileName | 這是選用屬性。 如果您省略此屬性，則會挑選 folderPath 中的所有檔案。 在本教學課程中，會針對 fileName 指定 **emp.txt**，因此只會挑選該檔案進行處理。 |
| format -> type |輸入檔為文字格式，因此我們會使用 **TextFormat**。 |
| columnDelimiter | 輸入檔案中的資料行會以**逗號字元 (`,`)** 分隔。 |
| frequency/interval | frequency 會設為 **Hour** 且 interval 會設為 **1**，表示**每小時**都可取得輸入配量。 換句話說，Data Factory 服務會每小時都在您指定之 Blob 容器 (**adftutorial**) 的根資料夾中尋找輸入資料。 它會尋找管線開始和結束時間內 (而非這些時間之前或之後) 的資料。  |
| external | 如果資料不是由此管線產生，此屬性會設為 **true**。 本教學課程中的輸入資料位於 emp.txt 檔案中，該檔案不是由此管線產生，因此我們會將此屬性設定為 true。 |

如需這些 JSON 屬性的詳細資訊，請參閱 [Azure Blob 連接器](data-factory-azure-blob-connector.md#dataset-properties)一文。

### <a name="outputdatasetjson"></a>outputdataset.json

```JSON
{
  "name": "AzureSqlOutput",
  "properties": {
    "structure": [
      {
        "name": "FirstName",
        "type": "String"
      },
      {
        "name": "LastName",
        "type": "String"
      }
    ],
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "emp"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
下表提供程式碼片段中所使用之 JSON 屬性的描述：

| 屬性 | 說明 |
|:--- |:--- |
| type | type 屬性會設為 **AzureSqlTable**，因為資料已複製到 Azure SQL Database 中的資料表。 |
| 預設容器 | 表示您稍早建立的 **AzureSqlLinkedService**。 |
| tableName | 指定作為資料複製目的地的**資料表**。 | 
| frequency/interval | frequency 會設為**Hour** 且 interval 為**1**，這表示會在管線開始和結束時間之間 (而非這些時間之前或之後) **每小時**產生輸出配量。  |

資料庫的 emp 資料表中有三個資料行 – **ID**、**FirstName** 和 **LastName**。 ID 是識別資料行，所以您只需在此指定 **FirstName** 和 **LastName**。

如需這些 JSON 屬性的詳細資訊，請參閱 [Azure SQL 連接器](data-factory-azure-sql-connector.md#dataset-properties)一文。

### <a name="pipelinejson"></a>pipeline.json

```JSON
{
  "name": "ADFTutorialPipeline",
  "properties": {
    "description": "Copy data from a blob to Azure SQL table",
    "activities": [
      {
        "name": "CopyFromBlobToSQL",
        "description": "Push Regional Effectiveness Campaign data to Azure SQL database",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlOutput"
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
    "start": "2017-05-11T00:00:00Z",
    "end": "2017-05-12T00:00:00Z"
  }
}
```

請注意下列幾點：

- 在活動區段中，只會有一個 **type** 設為 **Copy** 的活動。 如需複製活動的詳細資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md)。 在 Data Factory 解決方案中，您也可以使用[資料轉換活動](data-factory-data-transformation-activities.md)。
- 活動的輸入設為 **AzureBlobInput**，活動的輸出則設為 **AzureSqlOutput**。 
- 在 **typeProperties** 區段中，來源類型指定為 **BlobSource**，接收類型指定為 **SqlSink**。 如需複製活動作為來源和接收器支援的資料存放區完整清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 若要了解如何使用特定支援的資料存放區作為來源/接收器，請按一下資料表中的連結。  
 
將 **start** 屬性的值替換為目前日期，並將 **end**值替換為隔天的日期。 在日期時間中，您只指定日期部分，並略過時間部分。 例如，"2017-02-03"，這相當於 "2017-02-03T00:00:00Z"
 
開始和結束日期時間都必須是 [ISO 格式](http://en.wikipedia.org/wiki/ISO_8601)。 例如：2016-10-14T16:32:41Z。 **end** 時間為選擇性項目，但在本教學課程中會用到。 
 
如果您未指定 **end** 屬性的值，則會以「**start + 48 小時**」計算。 若要無限期地執行管線，請指定 **9999-09-09** 做為 **end** 屬性的值。
 
在上述範例中，由於每小時即產生一個資料配量，共會有 24 個資料配量。

如需管線定義中 JSON 屬性的說明，請參閱[建立管線](data-factory-create-pipelines.md)一文。 如需複製活動定義中 JSON 屬性的說明，請參閱[資料移動活動](data-factory-data-movement-activities.md)。 如需 BlobSource 所支援 JSON 屬性的說明，請參閱 [Azure Blob 連接器](data-factory-azure-blob-connector.md)一文。 如需 SqlSink 支援的 JSON 屬性說明，請參閱 [Azure SQL Database 連接器](data-factory-azure-sql-connector.md)一文。

## <a name="set-global-variables"></a>設定全域變數
在 Azure PowerShell 中，將值取代為您自己的值之後，執行下列命令：

> [!IMPORTANT]
> 如需取得用戶端識別碼、用戶端密碼、租用戶識別碼及訂用帳戶識別碼的指示，請參閱 [必要條件](#prerequisites) 一節。   
> 
> 

```JSON
$client_id = "<client ID of application in AAD>"
$client_secret = "<client key of application in AAD>"
$tenant = "<Azure tenant ID>";
$subscription_id="<Azure subscription ID>";

$rg = "ADFTutorialResourceGroup"
```

更新您正在使用的資料處理站名稱之後，執行下列命令︰ 

```
$adf = "ADFCopyTutorialDF"
```

## <a name="authenticate-with-aad"></a>使用 AAD 驗證
執行下列命令以驗證 Azure Active Directory (AAD)： 

```PowerShell
$cmd = { .\curl.exe -X POST https://login.microsoftonline.com/$tenant/oauth2/token  -F grant_type=client_credentials  -F resource=https://management.core.windows.net/ -F client_id=$client_id -F client_secret=$client_secret };
$responseToken = Invoke-Command -scriptblock $cmd;
$accessToken = (ConvertFrom-Json $responseToken).access_token;

(ConvertFrom-Json $responseToken) 
```

## <a name="create-data-factory"></a>建立資料處理站
在此步驟中，您會建立名為 **ADFCopyTutorialDF**的 Azure Data Factory。 資料處理站可以有一或多個管線。 其中的管線可以有一或多個活動。 例如，「複製活動」會從來源複製資料到目的地資料存放區。 HDInsight Hive 活動會執行 Hive 指令碼來轉換輸入資料，以產生輸出資料。 執行以下命令以建立 Data Factory： 

1. 將命令指派給名為 **cmd**的變數。 
   
    > [!IMPORTANT]
    > 確認您在此指定的名稱 (ADFCopyTutorialDF) 符合在 **datafactory.json**指定的名稱。 
   
    ```PowerShell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data “@datafactory.json” https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/ADFCopyTutorialDF0411?api-version=2015-10-01};
    ```
2. 使用 **Invoke-Command**執行命令。
   
    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. 檢視結果。 如果已成功建立 Data Factory，您會在 **結果**中看到 Data Factory 的 JSON；不然，您會看到一則錯誤訊息。  
   
    ```
    Write-Host $results
    ```

請注意下列幾點：

* Azure Data Factory 的名稱在全域必須是唯一的。 如果您在結果中看到錯誤︰「Data factory 名稱 "ADFCopyTutorialDF" 無法使用」 ，請執行下列步驟︰  
  
  1. 在 **datafactory.json** 檔案中變更名稱 (例如，yournameADFCopyTutorialDF)。
  2. 在指派 **$cmd** 變數值的第一個命令中，以新的名稱取代 ADFCopyTutorialDF 並執行命令。 
  3. 執行下面兩個命令來叫用 REST API，以建立 Data Factory 和列印作業的結果。 
     
     請參閱 [Data Factory - 命名規則](data-factory-naming-rules.md) 主題，以了解 Data Factory 成品的命名規則。
* 若要建立 Data Factory 執行個體，您必須是 Azure 訂用帳戶的參與者/系統管理員
* Data Factory 的名稱未來可能會註冊為 DNS 名稱，因此會變成公開可見的名稱。
* 如果您收到錯誤：「此訂用帳戶未註冊為使用命名空間 Microsoft.DataFactory」，請執行下列其中一項，然後嘗試再次發佈︰ 
  
  * 在 Azure PowerShell 中，執行下列命令以註冊 Data Factory 提供者： 

    ```PowerShell    
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```
    您可以執行下列命令來確認已註冊 Data Factory 提供者。 
    
    ```PowerShell
    Get-AzureRmResourceProvider
    ```
  * 使用 Azure 訂用帳戶登入 [Azure 入口網站](https://portal.azure.com) 並瀏覽至 [Data Factory] 刀鋒視窗 (或) 在 Azure 入口網站中建立 Data Factory。 此動作會自動為您註冊提供者。

建立管線之前，您必須先建立一些 Data Factory 項目。 您會先建立連結服務，將來源和目的地資料存放區連結至您的資料存放區。 然後，定義輸入和輸出資料集，以表示連結資料存放區中的資料。 最後，建立會使用這些資料集之活動的管線。

## <a name="create-linked-services"></a>建立連結的服務
您在資料處理站中建立的連結服務會將您的資料存放區和計算服務連結到資料處理站。 在本教學課程中，您不會使用任何計算服務，例如 Azure HDInsight 或 Azure Data Lake Analytics。 您可以使用兩種類型的資料存放區：Azure 儲存體 (來源) 和 Azure SQL Database (目的地)。 因此，您可以建立名為 AzureStorageLinkedService 和 AzureSqlLinkedService 的兩個連結服務︰類型為 AzureStorage 和 AzureSqlDatabase。  

AzureStorageLinkedService 會將 Azure 儲存體帳戶連結至資料處理站。 此儲存體帳戶是您在其中建立容器並將資料上傳為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)一部分的帳戶。   

AzureSqlLinkedService 會將 Azure SQL Database 連結至資料處理站。 從 Blob 儲存體複製的資料會儲存在此資料庫中。 您在此資料庫中建立了 emp 資料表，作為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)的一部分。  

### <a name="create-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務
在此步驟中，您會將您的 Azure 儲存體帳戶連結到您的資料處理站。 在此區段中指定您 Azure 儲存體帳戶的名稱和金鑰。 如需用來定義 Azure 儲存體連結服務之 JSON 屬性的詳細資料，請參閱 [Azure 儲存體連結服務](data-factory-azure-blob-connector.md#azure-storage-linked-service)。  

1. 將命令指派給名為 **cmd**的變數。 

    ```PowerShell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@azurestoragelinkedservice.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/AzureStorageLinkedService?api-version=2015-10-01};
    ```
2. 使用 **Invoke-Command**執行命令。

    ```PowerShell   
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. 檢視結果。 如果已成功建立連結服務，您會在 **結果**中看到連結服務的 JSON；不然，您會看到一則錯誤訊息。

    ```PowerShell   
    Write-Host $results
    ```

### <a name="create-azure-sql-linked-service"></a>建立 Azure SQL 連結服務
在此步驟中，您會將您的 Azure SQL Database 連結到您的 Data Factory。 在此區段中指定 Azure SQL 伺服器名稱、資料庫名稱、使用者名稱和使用者密碼。 如需用來定義 Azure SQL 連結服務之 JSON 屬性的詳細資料，請參閱 [Azure SQL 連結服務](data-factory-azure-sql-connector.md#linked-service-properties)。

1. 將命令指派給名為 **cmd**的變數。 
   
    ```PowerShell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data “@azuresqllinkedservice.json” https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/AzureSqlLinkedService?api-version=2015-10-01};
    ```
2. 使用 **Invoke-Command**執行命令。
   
    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. 檢視結果。 如果已成功建立連結服務，您會在 **結果**中看到連結服務的 JSON；不然，您會看到一則錯誤訊息。
   
    ```PowerShell
    Write-Host $results
    ```

## <a name="create-datasets"></a>建立資料集
在上一個步驟中，您已建立可將 Azure 儲存體帳戶和 Azure SQL Database 連結至資料處理站的連結服務。 在此步驟中，您會定義名為 AzureBlobInput 和 AzureSqlOutput 的兩個資料集，它們分別代表 AzureStorageLinkedService 和 AzureSqlLinkedService 所參照資料存放區中儲存的輸入和輸出資料。

Azure 儲存體連結服務會指定 Data Factory 服務在執行階段用來連線到 Azure 儲存體帳戶的連接字串。 而且，輸入 Blob 資料集 (AzureBlobInput) 會指定包含輸入資料的容器和資料夾。  

同樣第，Azure SQL Database 連結服務會指定 Data Factory 在執行階段用來連線到 Azure SQL Database 的連接字串。 而且，輸出 SQL 資料表資料集 (OututDataset) 會指定資料庫中作為 Blob 儲存體資料複製目的地的資料表。 

### <a name="create-input-dataset"></a>建立輸入資料集
在此步驟中，您將在 AzureStorageLinkedService 連結服務所代表的 Azure 儲存體中，建立名為 AzureBlobInput 的資料集，該資料集會指向 Blob 容器 (adftutorial) 根資料夾中的 Blob 檔案 (emp.txt)。 如果您未指定 (或跳過) fileName 的值，則輸入資料夾中所有 Blob 資料都會複製到目的地。 在本教學課程中，您可指定 fileName 的值。 

1. 將命令指派給名為 **cmd**的變數。 

    ```PowerSHell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@inputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureBlobInput?api-version=2015-10-01};
    ```
2. 使用 **Invoke-Command**執行命令。
   
    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. 檢視結果。 如果已成功建立資料集，您會在 **結果**中看到資料集的 JSON；不然，您會看到一則錯誤訊息。
   
    ```PowerShell
    Write-Host $results
    ```

### <a name="create-output-dataset"></a>建立輸出資料集
Azure SQL Database 連結服務會指定 Data Factory 在執行階段用來連線到 Azure SQL Database 的連接字串。 您在此步驟中建立的輸出 SQL 資料表資料集 (OututDataset) 會指定資料庫中作為 Blob 儲存體資料複製目的地的資料表。

1. 將命令指派給名為 **cmd**的變數。

    ```PowerShell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@outputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureSqlOutput?api-version=2015-10-01};
    ```
2. 使用 **Invoke-Command**執行命令。
    
    ```PowerShell   
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. 檢視結果。 如果已成功建立資料集，您會在 **結果**中看到資料集的 JSON；不然，您會看到一則錯誤訊息。
   
    ```PowerShell
    Write-Host $results
    ``` 

## <a name="create-pipeline"></a>建立管線
在此步驟中，您會建立管線，其中含有使用 **AzureBlobInput** 作為輸入和使用 **AzureSqlOutput** 作為輸出的**複製活動**。

目前，驅動排程的是輸出資料集。 在本教學課程中，輸出資料集設定成一小時產生一次配量。 管線具有相隔一天 (也就是 24 小時) 的開始時間和結束時間。 因此，管線會產生輸出資料集的 24 個配量。 

1. 將命令指派給名為 **cmd**的變數。

    ```PowerShell   
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@pipeline.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datapipelines/MyFirstPipeline?api-version=2015-10-01};
    ```
2. 使用 **Invoke-Command**執行命令。

    ```PowerShell   
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. 檢視結果。 如果已成功建立資料集，您會在 **結果**中看到資料集的 JSON；不然，您會看到一則錯誤訊息。  

    ```PowerShell   
    Write-Host $results
    ```

**恭喜！** 您已成功建立 Azure Data Factory，其中有管線可將資料從 Azure Blob 儲存體複製到 Azure SQL Database。

## <a name="monitor-pipeline"></a>監視管線
在此步驟中，您會使用 Data Factory REST API 來監視管線所產生的配量。

```PowerShell
$ds ="AzureSqlOutput"
```

> [!IMPORTANT] 
> 請確定在下列命令中指定的開始和結束時間符合管線的開始和結束時間。 

```PowerShell
$cmd = {.\curl.exe -X GET -H "Authorization: Bearer $accessToken" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/$ds/slices?start=2017-05-11T00%3a00%3a00.0000000Z"&"end=2017-05-12T00%3a00%3a00.0000000Z"&"api-version=2015-10-01};
```

```PowerShell
$results2 = Invoke-Command -scriptblock $cmd;
```

```PowerShell
IF ((ConvertFrom-Json $results2).value -ne $NULL) {
    ConvertFrom-Json $results2 | Select-Object -Expand value | Format-Table
} else {
        (convertFrom-Json $results2).RemoteException
}
```

執行 Invoke-Command 和下一個命令，直到您看到配量處於 [就緒] 狀態或 [失敗] 狀態。 當配量處於 [就緒] 狀態時，請檢查您的 Azure SQL Database 的 **emp** 資料表中的輸出資料。 

對於每個配量，來源檔案中有兩個資料列會複製到 Azure SQL Database 中的 emp 資料表。 因此，成功處理所有配量 (處於 [就緒] 狀態) 後，您會在 emp 資料表中看到 24 筆新記錄。 

## <a name="summary"></a>總結
在本教學課程中，您已使用 REST API 建立要將資料從 Azure Blob 複製到 Azure SQL 資料庫的 Azure Data Factory。 以下是您在本教學課程中執行的高階步驟：  

1. 建立 Azure **Data Factory**。
2. 建立 **連結服務**：
   1. Azure 儲存體連結服務可連結保留輸入資料的 Azure 儲存體帳戶。     
   2. Azure SQL 連結服務可連結保留輸出資料的 Azure SQL Database。 
3. 建立可描述管線輸入資料和輸出資料的 **資料集**。
4. 建立具有複製活動的 **管線** ，以 BlobSource 做為來源並以 SqlSink 做為接收器。 

## <a name="next-steps"></a>後續步驟
在本教學課程中，您可使用 Azure Blob 儲存體作為來源資料存放區以及使用 Azure SQL Database 作為複製作業的目的地資料存放區。 下表提供複製活動所支援作為來源或目的地的資料存放區清單： 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

若要深入了解如何從資料存放區雙向複製資料，請按一下資料表中資料存放區的連結。
