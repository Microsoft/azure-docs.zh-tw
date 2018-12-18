---
title: 建置您的第一個 Data Factory (REST) | Microsoft Docs
description: 在本教學課程中，您會使用 Data Factory REST API 建立範例 Azure Data Factory 管線。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 7e0a2465-2d85-4143-a4bb-42e03c273097
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 11/01/2017
ms.author: shlo
robots: noindex
ms.openlocfilehash: 91a85505eda34e11f0db9c5f3c46e33aad992ffe
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45736694"
---
# <a name="tutorial-build-your-first-azure-data-factory-using-data-factory-rest-api"></a>教學課程：使用 Data Factory REST API 建置您的第一個 Azure Data Factory
> [!div class="op_single_selector"]
> * [概觀和必要條件](data-factory-build-your-first-pipeline.md)
> * [Azure 入口網站](data-factory-build-your-first-pipeline-using-editor.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [Resource Manager 範本](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)
>
>


> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[快速入門：使用 Azure Data Factory 來建立資料處理站](../quickstart-create-data-factory-rest-api.md)。

在本文中，您會使用 Data Factory REST API 來建立第一個 Azure Data Factory。 若要使用其他工具/SDK 進行本教學課程，請選取下拉式清單的其中一個選項。

本教學課程中的管線有一個活動︰**HDInsight Hive 活動**。 此活動會在 Azure HDInsight 叢集上執行 Hive 指令碼，以轉換輸入資料來產生輸出資料。 管線已排定每個月在指定的開始與結束時間之間執行一次。

> [!NOTE]
> 這篇文章並未涵蓋所有的 REST API。 如需 REST API 的完整文件，請參閱 [Data Factory REST API 參考](/rest/api/datafactory/)。
> 
> 一個管線中可以有多個活動。 您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 如需詳細資訊，請參閱 [Data Factory 排程和執行](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)。


## <a name="prerequisites"></a>必要條件
* 詳讀 [教學課程概觀](data-factory-build-your-first-pipeline.md) 一文並完成 **必要** 步驟。
* 在您的電腦上安裝 [Curl](https://curl.haxx.se/dlwiz/) 。 您可搭配使用 CURL 工具與 REST 命令來建立 Data Factory。
* 請依照 [本文](../../azure-resource-manager/resource-group-create-service-principal-portal.md) 的指示：
  1. 在 Azure Active Directory 中建立名為 **ADFGetStartedApp** 的 Web 應用程式。
  2. 取得**用戶端識別碼**和**秘密金鑰**。
  3. 取得 **租用戶識別碼**。
  4. 將 **ADFGetStartedApp** 應用程式指派給 **Data Factory 參與者**角色。
* 安裝 [Azure PowerShell](/powershell/azure/overview)。
* 啟動 **PowerShell** 並執行下列命令。 將 Azure PowerShell 維持在開啟狀態，直到本教學課程結束為止。 如果您關閉並重新開啟，則需要再次執行這些命令。
  1. 執行 **Connect-AzureRmAccount**，並輸入您用來登入 Azure 入口網站的使用者名稱和密碼。
  2. 執行 **Get-AzureRmSubscription** 以檢視此帳戶的所有訂用帳戶。
  3. 執行 **Get-AzureRmSubscription -SubscriptionName NameOfAzureSubscription | Set-AzureRmContext** 來選取您想要使用的訂用帳戶。 以您的 Azure 訂用帳戶名稱取代 **NameOfAzureSubscription** 。
* 在 PowerShell 中執行以下命令，建立名為 **ADFTutorialResourceGroup** 的 Azure 資源群組：

    ```PowerShell
    New-AzureRmResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```

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
    "name": "FirstDataFactoryREST",
    "location": "WestUS"
}
```

### <a name="azurestoragelinkedservicejson"></a>azurestoragelinkedservice.json
> [!IMPORTANT]
> 以 Azure 儲存體帳戶的名稱和金鑰取代 **accountname** 和 **accountkey**。 若要了解如何取得您的儲存體存取金鑰，請參閱[管理儲存體帳戶](../../storage/common/storage-account-manage.md#access-keys)中說明如何檢視、複製和重新產生儲存體存取金鑰的資訊。
>
>

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

### <a name="hdinsightondemandlinkedservicejson"></a>hdinsightondemandlinkedservice.json

```JSON
{
    "name": "HDInsightOnDemandLinkedService",
    "properties": {
        "type": "HDInsightOnDemand",
        "typeProperties": {
            "version": "3.5",
            "clusterSize": 1,
            "timeToLive": "00:05:00",
            "osType": "Linux",
            "linkedServiceName": "AzureStorageLinkedService"
        }
    }
}
```

下表提供程式碼片段中所使用之 JSON 屬性的描述：

| 屬性 | 說明 |
|:--- |:--- |
| ClusterSize |HDInsight 叢集的大小。 |
| TimeToLive |指定 HDInsight 叢集在被刪除之前的閒置時間。 |
| linkedServiceName |指定用來儲存 HDInsight 產生之記錄檔的儲存體帳戶 |

請注意下列幾點：

* Data Factory 會使用上述 JSON，為您建立**以 Linux 為基礎的** HDInsight 叢集。 如需詳細資訊，請參閱 [HDInsight 隨選連結服務](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) 。
* 您可以使用 **自己的 HDInsight 叢集** ，不必使用隨選的 HDInsight 叢集。 如需詳細資訊，請參閱 [HDInsight 連結服務](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) 。
* HDInsight 叢集會在您於 JSON 中指定的 Blob 儲存體 (**linkedServiceName**) 建立**預設容器**。 HDInsight 不會在刪除叢集時刪除此容器。 這是設計的行為。 在使用 HDInsight 隨選連結服務時，除非有現有的即時叢集 (**timeToLive**)，否則每次處理配量時，就會建立 HDInsight 叢集，並在處理完成時予以刪除。

    隨著處理的配量越來越多，您會在 Azure Blob 儲存體中看到許多容器。 如果在疑難排解作業時不需要這些容器，建議您加以刪除以降低儲存成本。 這些容器的名稱遵循下列模式："adf**yourdatafactoryname**-**linkedservicename**-datetimestamp"。 請使用 [Microsoft 儲存體總管](http://storageexplorer.com/) 之類的工具刪除 Azure Blob 儲存體中的容器。

如需詳細資訊，請參閱 [HDInsight 隨選連結服務](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) 。

### <a name="inputdatasetjson"></a>inputdataset.json

```JSON
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

JSON 會定義名為 **AzureBlobInput**的資料集，以表示管線中活動的輸入資料。 此外，它也會指定將輸入資料放在名為 **adfgetstarted** 的 Blob 容器及名為 **inputdata** 的資料夾中。

下表提供程式碼片段中所使用之 JSON 屬性的描述：

| 屬性 | 說明 |
|:--- |:--- |
| type |類型屬性設為 AzureBlob，因為資料位於 Azure Blob 儲存體。 |
| 預設容器 |表示您稍早建立的 StorageLinkedService。 |
| fileName |這是選用屬性。 如果您省略此屬性，會挑選位於 folderPath 的所有檔案。 在這種情況下，只會處理 input.log。 |
| type |記錄檔為文字格式，因此我們會使用 TextFormat。 |
| columnDelimiter |記錄檔案中的資料行會以逗號字元 (,) 分隔 |
| frequency/interval |頻率設為「每月」且間隔為 1，表示每個月都會可取得輸入配量。 |
| external |如果輸入資料不是由 Data Factory 服務產生，此屬性會設為 true。 |

### <a name="outputdatasetjson"></a>outputdataset.json

```JSON
{
    "name": "AzureBlobOutput",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "folderPath": "adfgetstarted/partitioneddata",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ","
            }
        },
        "availability": {
            "frequency": "Month",
            "interval": 1
        }
    }
}
```

JSON 會定義名為 **AzureBlobOutput**的資料集，以表示管線中活動的輸出資料。 此外，它也會指定將結果儲存在名為 **adfgetstarted** 的 Blob 容器及名為 **partitioneddata** 的資料夾中。 **availability** 區段指定每個月產生一次輸出資料集。

### <a name="pipelinejson"></a>pipeline.json
> [!IMPORTANT]
> 以您的 Azure 儲存體帳戶名稱取代 **storageaccountname** 。
>
>

```JSON
{
    "name": "MyFirstPipeline",
    "properties": {
        "description": "My first Azure Data Factory pipeline",
        "activities": [{
            "type": "HDInsightHive",
            "typeProperties": {
                "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                "scriptLinkedService": "AzureStorageLinkedService",
                "defines": {
                    "inputtable": "wasb://adfgetstarted@<stroageaccountname>.blob.core.windows.net/inputdata",
                    "partitionedtable": "wasb://adfgetstarted@<stroageaccountname>t.blob.core.windows.net/partitioneddata"
                }
            },
            "inputs": [{
                "name": "AzureBlobInput"
            }],
            "outputs": [{
                "name": "AzureBlobOutput"
            }],
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
        }],
        "start": "2017-07-10T00:00:00Z",
        "end": "2017-07-11T00:00:00Z",
        "isPaused": false
    }
}
```

您會在 JSON 程式碼片段中建立一個管線，其中包括在 HDInsight 叢集上使用 Hive 處理資料的單一活動。

Hive 指令碼檔案 **partitionweblogs.hql** 儲存於 Azure 儲存體帳戶 (透過 scriptLinkedService 指定，名為 **StorageLinkedService**)，且位於 **adfgetstarted** 容器的 **script** 資料夾中。

**defines** 區段可指定執行階段設定，這些設定會傳遞到 Hive 指令碼做為 Hive 設定值 (例如 ${hiveconf:inputtable}、${hiveconf:partitionedtable})。

管線的 **start** 和 **end** 屬性會指定管線的作用中期間。

在活動 JSON 中，您會指定 Hive 指令碼要在透過 **linkedServiceName** – **HDInsightOnDemandLinkedService** 指定的計算上執行。

> [!NOTE]
> 如需上述範例使用之 JSON 屬性的詳細資料，請參閱 [Azure Data Factory 中的管線及活動](data-factory-create-pipelines.md)中的「管線 JSON」。
>
>

## <a name="set-global-variables"></a>設定全域變數
在 Azure PowerShell 中，將值取代為您自己的值之後，執行下列命令：

> [!IMPORTANT]
> 如需取得用戶端識別碼、用戶端密碼、租用戶識別碼及訂用帳戶識別碼的指示，請參閱 [必要條件](#prerequisites) 一節。
>
>

```PowerShell
$client_id = "<client ID of application in AAD>"
$client_secret = "<client key of application in AAD>"
$tenant = "<Azure tenant ID>";
$subscription_id="<Azure subscription ID>";

$rg = "ADFTutorialResourceGroup"
$adf = "FirstDataFactoryREST"
```


## <a name="authenticate-with-aad"></a>使用 AAD 驗證

```PowerShell
$cmd = { .\curl.exe -X POST https://login.microsoftonline.com/$tenant/oauth2/token  -F grant_type=client_credentials  -F resource=https://management.core.windows.net/ -F client_id=$client_id -F client_secret=$client_secret };
$responseToken = Invoke-Command -scriptblock $cmd;
$accessToken = (ConvertFrom-Json $responseToken).access_token;

(ConvertFrom-Json $responseToken)
```


## <a name="create-data-factory"></a>建立資料處理站
在此步驟中，您會建立名為 **FirstDataFactoryREST**的 Azure Data Factory。 資料處理站可以有一或多個管線。 其中的管線可以有一或多個活動。 例如，「複製活動」會從來源將資料複製到目的地資料存放區，HDInsight Hive 活動則是執行 Hive 指令碼來轉換資料。 執行以下命令以建立 Data Factory：

1. 將命令指派給名為 **cmd**的變數。

    確認您在此指定的名稱 (ADFCopyTutorialDF) 符合在 **datafactory.json**指定的名稱。

    ```PowerShell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data “@datafactory.json” https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/FirstDataFactoryREST?api-version=2015-10-01};
    ```
2. 使用 **Invoke-Command**執行命令。

    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. 檢視結果。 如果已成功建立 Data Factory，您會在 **結果**中看到 Data Factory 的 JSON；不然，您會看到一則錯誤訊息。

    ```PowerShell
    Write-Host $results
    ```

請注意下列幾點：

* Azure Data Factory 的名稱在全域必須是唯一的。 如果您在結果中看到錯誤︰「Data factory 名稱 “FirstDataFactoryREST” 無法使用」 ，請執行下列步驟︰
  1. 在 **datafactory.json** 檔案中變更名稱 (例如，yournameFirstDataFactoryREST)。 請參閱 [Data Factory - 命名規則](data-factory-naming-rules.md) 主題，以了解 Data Factory 成品的命名規則。
  2. 在指派 **$cmd** 變數值的第一個命令中，以新的名稱取代 FirstDataFactoryREST 並執行命令。
  3. 執行下面兩個命令來叫用 REST API，以建立 Data Factory 和列印作業的結果。
* 若要建立 Data Factory 執行個體，您必須是 Azure 訂用帳戶的參與者/系統管理員
* Data Factory 的名稱未來可能會註冊為 DNS 名稱，因此會變成公開可見的名稱。
* 如果您收到錯誤：「此訂用帳戶未註冊為使用命名空間 Microsoft.DataFactory」，請執行下列其中一項，然後嘗試再次發佈︰

  * 在 Azure PowerShell 中，執行下列命令以註冊 Data Factory 提供者：

    ```PowerShell
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```

      您可以執行下列命令來確認已註冊 Data Factory 提供者：
    ```PowerShell
    Get-AzureRmResourceProvider
    ```
  * 使用 Azure 訂用帳戶登入 [Azure 入口網站](https://portal.azure.com) 並瀏覽至 [Data Factory] 刀鋒視窗 (或) 在 Azure 入口網站中建立 Data Factory。 此動作會自動為您註冊提供者。

建立管線之前，您必須先建立一些 Data Factory 項目。 首先，您要先建立連結的服務，以便將資料存放區/電腦連結到您的資料存放區；並定義輸入和輸出資料集，表示資料位於連結的資料存放區中。

## <a name="create-linked-services"></a>建立連結的服務
在此步驟中，您會將您的 Azure 儲存體帳戶和隨選 Azure HDInsight 叢集連結到您的 Data Factory。 Azure 儲存體帳戶會保留此範例中管線的輸入和輸出資料。 HDInsight 連結服務是用來執行此範例中管線活動所指定的 Hive 指令碼。

### <a name="create-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務
在此步驟中，您會將您的 Azure 儲存體帳戶連結到您的 Data Factory。 在本教學課程中，您會使用相同的 Azure 儲存體帳戶來存放輸入/輸出資料及 HQL 指令碼檔案。

1. 將命令指派給名為 **cmd**的變數。

    ```PowerShell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data “@azurestoragelinkedservice.json” https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/AzureStorageLinkedService?api-version=2015-10-01};
    ```
2. 使用 **Invoke-Command**執行命令。

    ```PowerShell
    $results = Invoke-Command -scriptblock $cmd;
    ```
3. 檢視結果。 如果已成功建立連結服務，您會在 **結果**中看到連結服務的 JSON；不然，您會看到一則錯誤訊息。

    ```PowerShell
    Write-Host $results
    ```

### <a name="create-azure-hdinsight-linked-service"></a>建立 Azure HDInsight 連結服務
在此步驟中，您可將隨選 HDInsight 叢集連結至 Data Factory。 HDInsight 叢集會在執行階段自動建立，並在處理完成之後刪除，且會閒置一段時間。 您可以使用自己的 HDInsight 叢集，不必使用隨選的 HDInsight 叢集。 請參閱 [計算連結服務](data-factory-compute-linked-services.md) 以取得詳細資料。

1. 將命令指派給名為 **cmd**的變數。

    ```PowerShell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@hdinsightondemandlinkedservice.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/linkedservices/hdinsightondemandlinkedservice?api-version=2015-10-01};
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
在此步驟中，您會建立資料集來代表 Hive 處理的輸入和輸出資料。 這些資料集是您稍早在本教學課程中建立的 **StorageLinkedService** 。 連結的服務會指向 Azure 儲存體帳戶，而資料集則會指定保留輸入和輸出資料儲存體中的容器、資料夾和檔案名稱。

### <a name="create-input-dataset"></a>建立輸入資料集
在此步驟中，您會建立輸入資料集來代表 Azure Blob 儲存體中儲存的輸入資料。

1. 將命令指派給名為 **cmd**的變數。

    ```PowerShell
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
在此步驟中，您會建立輸出資料集來代表 Azure Blob 儲存體中儲存的輸出資料。

1. 將命令指派給名為 **cmd**的變數。

    ```PowerShell
    $cmd = {.\curl.exe -X PUT -H "Authorization: Bearer $accessToken" -H "Content-Type: application/json" --data "@outputdataset.json" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/AzureBlobOutput?api-version=2015-10-01};
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
在此步驟中，您會建立第一個具有 **HDInsightHive** 活動的管線。 您每個月都可取得輸入配量資訊 (頻率：每月，間隔：1)，輸出配量則是每用產生，而活動的排程器屬性也設為每月。 輸出資料集設定及活動排程器必須相符。 目前，輸出資料集會影響排程，因此即使活動並未產生任何輸出，您都必須建立輸出資料集。 如果活動沒有任何輸入，您可以略過建立輸入資料集。

確認您在 Azure Blob 儲存體的 **adfgetstarted/inputdata** 資料夾中看到了 **input.log** 檔案，並執行下列命令以部署管線。 由於 **start** 和 **end** 時間設定在過去，且 **isPaused** 設為 false，管線 (管線中的活動) 會在部署之後立即執行。

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
4. 恭喜，您已經成功使用 Azure PowerShell 建立您的第一個管線！

## <a name="monitor-pipeline"></a>監視管線
在此步驟中，您會使用 Data Factory REST API 來監視管線所產生的配量。

```PowerShell
$ds ="AzureBlobOutput"

$cmd = {.\curl.exe -X GET -H "Authorization: Bearer $accessToken" https://management.azure.com/subscriptions/$subscription_id/resourcegroups/$rg/providers/Microsoft.DataFactory/datafactories/$adf/datasets/$ds/slices?start=1970-01-01T00%3a00%3a00.0000000Z"&"end=2016-08-12T00%3a00%3a00.0000000Z"&"api-version=2015-10-01};

$results2 = Invoke-Command -scriptblock $cmd;

IF ((ConvertFrom-Json $results2).value -ne $NULL) {
    ConvertFrom-Json $results2 | Select-Object -Expand value | Format-Table
} else {
        (convertFrom-Json $results2).RemoteException
}
```

> [!IMPORTANT]
> 建立隨選 HDInsight 叢集通常需要一些時間 (大約 20 分鐘)。 因此，管線預計需要 **大約 30 分鐘** 的時間來處理配量。
>
>

執行 Invoke-Command 和下一個命令，直到您看到配量處於 [就緒] 狀態或 [失敗] 狀態。 當配量處於 [就緒] 狀態時，檢查您 Blob 儲存體中 **adfgetstarted** 容器內 **partitioneddata** 資料夾的輸出資料。  建立隨選 HDInsight 叢集通常需要一些時間。

![輸出資料](./media/data-factory-build-your-first-pipeline-using-rest-api/three-ouptut-files.png)

> [!IMPORTANT]
> 配量處理成功時就會刪除輸入檔案。 因此，如果您想要重新執行配量或再次進行本教學課程，請將輸入檔案 (input.log) 上傳至 adfgetstarted 容器的 inputdata 資料夾。
>
>

您也可以使用 Azure 入口網站來監視配量及排解任何疑難問題。 如需詳細，請參閱 [使用 Azure 入口網站監視管線](data-factory-build-your-first-pipeline-using-editor.md#monitor-a-pipeline) 。

## <a name="summary"></a>總結
在本教學課程中，您會在 HDInsight hadoop 叢集上執行 Hive 指令碼，以建立 Azure Data Factory 來處理資料。 您會在使用 Azure 入口網站中使用 Data Factory 編輯器來執行下列步驟︰

1. 建立 Azure **Data Factory**。
2. 建立兩個 **連結服務**：
   1. **Azure 儲存體** 連結服務可將保留輸入/輸出檔案的 Azure Blob 儲存體連結至 Data Factory。
   2. **Azure HDInsight** 隨選連結服務可將 HDInsight Hadoop 隨選叢集連結至 Data Factory。 Azure Data Factory 會即時建立 HDInsight Hadoop 叢集以處理輸入資料及產生輸出資料。
3. 建立兩個 **資料集**，以說明管線中 HDInsight Hive 活動的輸入和輸出資料。
4. 建立具有 **HDInsight Hive** 活動的**管線**。

## <a name="next-steps"></a>後續步驟
在本文中，您已經建立可在隨選 Azure HDInsight 叢集上執行 Hive 指令碼，含有轉換活動 (HDInsight 活動) 的管線。 若要了解如何使用「複製活動」從 Azure Blob 將資料複製到 Azure SQL，請參閱 [教學課程：從 Azure Blob 將資料複製到 Azure SQL](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。

## <a name="see-also"></a>另請參閱
| 話題 | 說明 |
|:--- |:--- |
| [Data Factory REST API 參考](/rest/api/datafactory/) |請參閱 Data Factory Cmdlet 中的完整文件 |
| [管線](data-factory-create-pipelines.md) |本文協助您了解 Azure Data Factory 中的管線和活動，以及如何使用這些來為您的案例或業務建構端對端的資料導向工作流程。 |
| [資料集](data-factory-create-datasets.md) |本文協助您了解 Azure Data Factory 中的資料集。 |
| [排程和執行](data-factory-scheduling-and-execution.md) |本文說明 Azure Data Factory 應用程式模型的排程和執行層面。 |
| [使用監視應用程式來監視和管理管線](data-factory-monitor-manage-app.md) |本文說明如何使用監視及管理應用程式，來監視、管理管線及進行偵錯。 |
