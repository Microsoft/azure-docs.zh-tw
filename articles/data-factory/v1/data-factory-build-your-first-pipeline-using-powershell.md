---
title: 建置您的第一個 Data Factory (PowerShell) | Microsoft Docs
description: 在本教學課程中，您將使用 Azure PowerShell，建立範例 Azure Data Factory 管線。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: ''
editor: ''
ms.assetid: 22ec1236-ea86-4eb7-b903-0e79a58b90c7
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: 3972dd7031e25f20a2ef6bfb66b63fe3f3b0290b
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45732477"
---
# <a name="tutorial-build-your-first-azure-data-factory-using-azure-powershell"></a>教學課程：使用 Azure PowerShell 建置您的第一個 Azure Data Factory
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
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[快速入門：使用 Azure Data Factory 來建立資料處理站](../quickstart-create-data-factory-powershell.md)。

在本文中，您會使用 Azure PowerShell 來建立您的第一個 Azure Data Factory。 若要使用其他工具/SDK 進行本教學課程，請選取下拉式清單的其中一個選項。

本教學課程中的管線有一個活動︰**HDInsight Hive 活動**。 此活動會在 Azure HDInsight 叢集上執行 Hive 指令碼，以轉換輸入資料來產生輸出資料。 管線已排定每個月在指定的開始與結束時間之間執行一次。 

> [!NOTE]
> 本教學課程中的資料管線會轉換輸入資料來產生輸出資料， 它不是將資料從來源資料存放區，複製到目的地資料存放區。 如需說明如何使用 Azure Data Factory 複製資料的教學課程，請參閱[教學課程：將資料從 Blob 儲存體複製到 SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。
> 
> 一個管線中可以有多個活動。 您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 如需詳細資訊，請參閱 [Data Factory 排程和執行](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)。

## <a name="prerequisites"></a>必要條件
* 詳讀 [教學課程概觀](data-factory-build-your-first-pipeline.md) 一文並完成 **必要** 步驟。
* 按照 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 一文中的指示，在您的電腦上安裝最新版的 Azure PowerShell。
* (選用) 這篇文章並未涵蓋所有的 Data Factory Cmdlet。 如需 Data Factory Cmdlet 的完整文件，請參閱 [Data Factory Cmdlet 參考](/powershell/module/azurerm.datafactories) 。

## <a name="create-data-factory"></a>建立資料處理站
在此步驟中，您會使用 Azure PowerShell 建立名為 **FirstDataFactoryPSH**的 Azure Data Factory。 資料處理站可以有一或多個管線。 其中的管線可以有一或多個活動。 例如，「複製活動」會從來源將資料複製到目的地資料存放區，HDInsight Hive 活動則是執行 Hive 指令碼來轉換輸入資料。 讓我們在這個步驟中開始建立 Data Factory。

1. 啟動 Azure PowerShell 並執行下列命令。 將 Azure PowerShell 維持在開啟狀態，直到本教學課程結束為止。 如果您關閉並重新開啟，則需要再次執行這些命令。
   * 執行下列命令並輸入您用來登入 Azure 入口網站的使用者名稱和密碼。
    ```PowerShell
    Connect-AzureRmAccount
    ```    
   * 執行下列命令以檢視此帳戶的所有訂用帳戶。
    ```PowerShell
    Get-AzureRmSubscription 
    ```
   * 執行下列命令以選取您要使用的訂用帳戶。 此訂用帳戶應該與您在 Azure 入口網站中使用的相同。
    ```PowerShell
    Get-AzureRmSubscription -SubscriptionName <SUBSCRIPTION NAME> | Set-AzureRmContext
    ```     
2. 執行以下命令，建立名為 **ADFTutorialResourceGroup** 的 Azure 資源群組：
    
    ```PowerShell
    New-AzureRmResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```
    本教學課程的某些步驟假設您使用名為 ADFTutorialResourceGroup 的資源群組。 如果使用不同的資源群組，您必須以該群組取代本教學課程中的 ADFTutorialResourceGroup。
3. 執行 **New-AzureRmDataFactory** Cmdlet，建立名為 **FirstDataFactoryPSH** 的 Data Factory。

    ```PowerShell
    New-AzureRmDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name FirstDataFactoryPSH –Location "West US"
    ```
請注意下列幾點：

* Azure Data Factory 的名稱在全域必須是唯一的。 如果您收到錯誤：「Data Factory 名稱 “FirstDataFactoryPSH” 無法使用」 ，請變更名稱 (例如 yournameFirstDataFactoryPSH)。 執行本教學課程中的步驟時，請使用此名稱來取代 ADFTutorialFactoryPSH。 請參閱 [Data Factory - 命名規則](data-factory-naming-rules.md) 主題，以了解 Data Factory 成品的命名規則。
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

建立管線之前，您必須先建立一些 Data Factory 項目。 首先，您要先建立連結的服務，以便將資料存放區/電腦連結到您的資料存放區；並定義輸入和輸出資料集，表示位於連結的資料存放區中的輸入/輸出資料，然後建立具有使用這些資料集的活動之管線。

## <a name="create-linked-services"></a>建立連結的服務
在此步驟中，您會將您的 Azure 儲存體帳戶和隨選 Azure HDInsight 叢集連結到您的 Data Factory。 Azure 儲存體帳戶會保留此範例中管線的輸入和輸出資料。 HDInsight 連結服務是用來執行此範例中管線活動所指定的 Hive 指令碼。 識別案例中使用的資料存放區/計算服務，並建立連結的服務將這些服務連結到 Data Factory。

### <a name="create-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務
在此步驟中，您會將您的 Azure 儲存體帳戶連結到您的 Data Factory。 您會使用相同的 Azure 儲存體帳戶來存放輸入/輸出資料及 HQL 指令碼檔案。

1. 在 C:\ADFGetStarted 資料夾中，使用以下內容建立名為 StorageLinkedService.json 的 JSON 檔案。 如果不存在，請建立 ADFGetStarted 資料夾。

    ```json
    {
        "name": "StorageLinkedService",
        "properties": {
            "type": "AzureStorage",
            "description": "",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
            }
        }
    }
    ```
    以您的 Azure 儲存體帳戶名稱取代**帳戶名稱**，並以 Azure 儲存體帳戶的存取金鑰取代**帳戶金鑰**。 若要了解如何取得您的儲存體存取金鑰，請參閱[管理儲存體帳戶](../../storage/common/storage-account-manage.md#access-keys)中說明如何檢視、複製和重新產生儲存體存取金鑰的資訊。
2. 在 Azure PowerShell 中，切換到 ADFGetStarted 資料夾。
3. 您可以使用 **New-AzureRmDataFactoryLinkedService** Cmdlet 建立連結服務。 此 Cmdlet 和您在本教學課程中使用的其他 Data Factory Cmdlet，皆需要您將值傳給 *ResourceGroupName* 和 *DataFactoryName* 參數。 或者，您可以使用 **Get-AzureRmDataFactory** 取得 **DataFactory** 物件，並傳遞此物件，就不需要在每次執行 Cmdlet 時輸入 *ResourceGroupName* 和 *DataFactoryName*。 執行以下命令，將 **Get-AzureRmDataFactory** Cmdlet 的輸出指派給 **$df** 變數。

    ```PowerShell
    $df=Get-AzureRmDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name FirstDataFactoryPSH
    ```
4. 現在，執行 **New-AzureRmDataFactoryLinkedService** Cmdlet 以建立連結的 **StorageLinkedService** 服務。

    ```PowerShell
    New-AzureRmDataFactoryLinkedService $df -File .\StorageLinkedService.json
    ```
    如果您還沒執行 **Get-AzureRmDataFactory** Cmdlet 和指派輸出給 **$df** 變數，您就必須指定 *ResourceGroupName* 和 *DataFactoryName* 參數的值，如下所示。

    ```PowerShell
    New-AzureRmDataFactoryLinkedService -ResourceGroupName ADFTutorialResourceGroup -DataFactoryName FirstDataFactoryPSH -File .\StorageLinkedService.json
    ```
    如果您在教學課程途中關閉 Azure PowerShell，則下次啟動 Azure PowerShell 時必須執行 **Get-AzureRmDataFactory** Cmdlet，才能完成教學課程。

### <a name="create-azure-hdinsight-linked-service"></a>建立 Azure HDInsight 連結服務
在此步驟中，您可將隨選 HDInsight 叢集連結至 Data Factory。 HDInsight 叢集會在執行階段自動建立，並在處理完成之後刪除，且會閒置一段時間。 您可以使用自己的 HDInsight 叢集，不必使用隨選的 HDInsight 叢集。 請參閱 [計算連結服務](data-factory-compute-linked-services.md) 以取得詳細資料。

1. 在 **C:\ADFGetStarted** 資料夾中，使用以下內容建立名為 **HDInsightOnDemandLinkedService**.json 的 JSON 檔案。

    ```json
    {
        "name": "HDInsightOnDemandLinkedService",
        "properties": {
            "type": "HDInsightOnDemand",
            "typeProperties": {
                "version": "3.5",
                "clusterSize": 1,
                "timeToLive": "00:05:00",
                "osType": "Linux",
                "linkedServiceName": "StorageLinkedService"
            }
        }
    }
    ```
    下表提供程式碼片段中所使用之 JSON 屬性的描述：

   | 屬性 | 說明 |
   |:--- |:--- |
   | ClusterSize |指定 HDInsight 叢集的大小。 |
   | TimeToLive |指定 HDInsight 叢集在被刪除之前的閒置時間。 |
   | linkedServiceName |指定用來儲存 HDInsight 產生之記錄檔的儲存體帳戶 |

    請注意下列幾點：

   * Data Factory 會使用 JSON，為您建立**以 Linux 為基礎的** HDInsight 叢集。 如需詳細資訊，請參閱 [HDInsight 隨選連結服務](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) 。
   * 您可以使用 **自己的 HDInsight 叢集** ，不必使用隨選的 HDInsight 叢集。 如需詳細資訊，請參閱 [HDInsight 連結服務](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) 。
   * HDInsight 叢集會在您於 JSON 中指定的 Blob 儲存體 (**linkedServiceName**) 建立**預設容器**。 HDInsight 不會在刪除叢集時刪除此容器。 這是設計的行為。 在使用 HDInsight 隨選連結服務時，除非有現有的即時叢集 (**timeToLive**)，否則每次處理配量時，就會建立 HDInsight 叢集。 此叢集會在處理完成時自動刪除。

       隨著處理的配量越來越多，您會在 Azure Blob 儲存體中看到許多容器。 如果在疑難排解作業時不需要這些容器，建議您加以刪除以降低儲存成本。 這些容器的名稱遵循下列模式："adf**yourdatafactoryname**-**linkedservicename**-datetimestamp"。 請使用 [Microsoft 儲存體總管](http://storageexplorer.com/) 之類的工具刪除 Azure Blob 儲存體中的容器。

     如需詳細資訊，請參閱 [HDInsight 隨選連結服務](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service) 。
2. 執行 **New-AzureRmDataFactoryLinkedService** Cmdlet，建立名為 HDInsightOnDemandLinkedService 的連結服務。
    
    ```PowerShell
    New-AzureRmDataFactoryLinkedService $df -File .\HDInsightOnDemandLinkedService.json
    ```

## <a name="create-datasets"></a>建立資料集
在此步驟中，您會建立資料集來代表 Hive 處理的輸入和輸出資料。 這些資料集是您稍早在本教學課程中建立的 **StorageLinkedService** 。 連結的服務會指向 Azure 儲存體帳戶，而資料集則會指定保留輸入和輸出資料儲存體中的容器、資料夾和檔案名稱。

### <a name="create-input-dataset"></a>建立輸入資料集
1. 在 **C:\ADFGetStarted** 資料夾中，使用下列內容建立名為 **InputTable.json** 的 JSON 檔案：

    ```json
    {
        "name": "AzureBlobInput",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "StorageLinkedService",
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
   | columnDelimiter |記錄檔案中的資料行會以逗號字元 (,) 分隔。 |
   | frequency/interval |頻率設為「每月」且間隔為 1，表示每個月都會可取得輸入配量。 |
   | external |如果輸入資料不是由 Data Factory 服務產生，此屬性會設為 true。 |
2. 在 Azure PowerShell 中執行以下命令來建立 Data Factory 資料集：

    ```PowerShell
    New-AzureRmDataFactoryDataset $df -File .\InputTable.json
    ```

### <a name="create-output-dataset"></a>建立輸出資料集
現在，您會建立輸出資料集來代表 Azure Blob 儲存體中儲存的輸出資料。

1. 在 **C:\ADFGetStarted** 資料夾中，使用下列內容建立名為 **OutputTable.json** 的 JSON 檔案：

    ```json
    {
      "name": "AzureBlobOutput",
      "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
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
2. 在 Azure PowerShell 中執行以下命令來建立 Data Factory 資料集：

    ```PowerShell
    New-AzureRmDataFactoryDataset $df -File .\OutputTable.json
    ```

## <a name="create-pipeline"></a>建立管線
在此步驟中，您會建立第一個具有 **HDInsightHive** 活動的管線。 您每個月都可取得輸入配量資訊 (頻率：每月，間隔：1)，輸出配量則是每用產生，而活動的排程器屬性也設為每月。 輸出資料集設定及活動排程器必須相符。 目前，輸出資料集會影響排程，因此即使活動並未產生任何輸出，您都必須建立輸出資料集。 如果活動沒有任何輸入，您可以略過建立輸入資料集。 本節結尾會說明下列 JSON 中使用的屬性。

1. 在 C:\ADFGetStarted 資料夾中，使用下列內容建立名為 MyFirstPipelinePSH.json 的 JSON 檔案：

   > [!IMPORTANT]
   > 在 JSON 中使用您的儲存體帳戶名稱取代 **storageaccountname** 。
   >
   >

    ```json
    {
        "name": "MyFirstPipeline",
        "properties": {
            "description": "My first Azure Data Factory pipeline",
            "activities": [
                {
                    "type": "HDInsightHive",
                    "typeProperties": {
                        "scriptPath": "adfgetstarted/script/partitionweblogs.hql",
                        "scriptLinkedService": "StorageLinkedService",
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
            "start": "2017-07-01T00:00:00Z",
            "end": "2017-07-02T00:00:00Z",
            "isPaused": false
        }
    }
    ```
    您會在 JSON 程式碼片段中建立一個管線，其中包括在 HDInsight 叢集上使用 Hive 處理「資料」的單一活動。

    Hive 指令碼檔案 **partitionweblogs.hql** 儲存於 Azure 儲存體帳戶 (透過 scriptLinkedService 指定，名為 **StorageLinkedService**)，且位於 **adfgetstarted** 容器的 **script** 資料夾中。

    **defines** 區段可用來指定執行階段設定，該設定會傳遞到 Hive 指令碼做為 Hive 設定值 (例如 ${hiveconf:inputtable}、${hiveconf:partitionedtable})。

    管線的 **start** 和 **end** 屬性會指定管線的作用中期間。

    在活動 JSON 中，您會指定 Hive 指令碼要在透過 **linkedServiceName** – **HDInsightOnDemandLinkedService** 指定的計算上執行。

   > [!NOTE]
   > 如需範例使用之 JSON 屬性的詳細資料，請參閱 [Azure Data Factory 中的管線及活動](data-factory-create-pipelines.md)中的「管線 JSON」。

2. 確認您在 Azure Blob 儲存體的 **adfgetstarted/inputdata** 資料夾中看到了 **input.log** 檔案，並執行下列命令以部署管線。 由於 **start** 和 **end** 時間設定在過去，且 **isPaused** 設為 false，管線 (管線中的活動) 會在部署之後立即執行。

    ```PowerShell
    New-AzureRmDataFactoryPipeline $df -File .\MyFirstPipelinePSH.json
    ```
3. 恭喜，您已經成功使用 Azure PowerShell 建立您的第一個管線！

## <a name="monitor-pipeline"></a>監視管線
在此步驟中，您會使用 Azure PowerShell 來監視 Azure Data Factory 的運作情形。

1. 執行 **Get-AzureRmDataFactory** 並指派輸出給 **$df** 變數。

    ```PowerShell
    $df=Get-AzureRmDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name FirstDataFactoryPSH
    ```
2. 執行 **Get-AzureRmDataFactorySlice**，以取得 **EmpSQLTable** (管線的輸出資料表) 所有配量的詳細資料。

    ```PowerShell
    Get-AzureRmDataFactorySlice $df -DatasetName AzureBlobOutput -StartDateTime 2017-07-01
    ```
    請注意，您在此處指定的 StartDateTime 與在管線 JSON 中指定的開始時間是相同的。 以下是範例輸出：

    ```PowerShell
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : FirstDataFactoryPSH
    DatasetName       : AzureBlobOutput
    Start             : 7/1/2017 12:00:00 AM
    End               : 7/2/2017 12:00:00 AM
    RetryCount        : 0
    State             : InProgress
    SubState          :
    LatencyStatus     :
    LongRetryCount    : 0
    ```
3. 執行 **Get-AzureRmDataFactoryRun** ，以取得特定配量的活動執行詳細資料。

    ```PowerShell
    Get-AzureRmDataFactoryRun $df -DatasetName AzureBlobOutput -StartDateTime 2017-07-01
    ```

    以下是範例輸出： 

    ```PowerShell
    Id                  : 0f6334f2-d56c-4d48-b427-d4f0fb4ef883_635268096000000000_635292288000000000_AzureBlobOutput
    ResourceGroupName   : ADFTutorialResourceGroup
    DataFactoryName     : FirstDataFactoryPSH
    DatasetName         : AzureBlobOutput
    ProcessingStartTime : 12/18/2015 4:50:33 AM
    ProcessingEndTime   : 12/31/9999 11:59:59 PM
    PercentComplete     : 0
    DataSliceStart      : 7/1/2017 12:00:00 AM
    DataSliceEnd        : 7/2/2017 12:00:00 AM
    Status              : AllocatingResources
    Timestamp           : 12/18/2015 4:50:33 AM
    RetryAttempt        : 0
    Properties          : {}
    ErrorMessage        :
    ActivityName        : RunSampleHiveActivity
    PipelineName        : MyFirstPipeline
    Type                : Script
    ```
    您可以繼續執行此 Cmdlet 直到您看到配量處於**就緒**狀態或**失敗**狀態。 當配量處於 [就緒] 狀態時，檢查您 Blob 儲存體中 **adfgetstarted** 容器內 **partitioneddata** 資料夾的輸出資料。  建立隨選 HDInsight 叢集通常需要一些時間。

    ![輸出資料](./media/data-factory-build-your-first-pipeline-using-powershell/three-ouptut-files.png)

> [!IMPORTANT]
> 建立隨選 HDInsight 叢集通常需要一些時間 (大約 20 分鐘)。 因此，管線預計需要 **大約 30 分鐘** 的時間來處理配量。
>
> 配量處理成功時就會刪除輸入檔案。 因此，如果您想要重新執行配量或再次進行本教學課程，請將輸入檔案 (input.log) 上傳至 adfgetstarted 容器的 inputdata 資料夾。
>
>

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
| [Data Factory Cmdlet 參考](/powershell/module/azurerm.datafactories) |請參閱 Data Factory Cmdlet 中的完整文件 |
| [管線](data-factory-create-pipelines.md) |本文協助您了解 Azure Data Factory 中的管線和活動，以及如何使用這些來為您的案例或業務建構端對端的資料導向工作流程。 |
| [資料集](data-factory-create-datasets.md) |本文協助您了解 Azure Data Factory 中的資料集。 |
| [排程和執行](data-factory-scheduling-and-execution.md) |本文說明 Azure Data Factory 應用程式模型的排程和執行層面。 |
| [使用監視應用程式來監視和管理管線](data-factory-monitor-manage-app.md) |本文說明如何使用監視及管理應用程式，來監視、管理管線及進行偵錯。 |
