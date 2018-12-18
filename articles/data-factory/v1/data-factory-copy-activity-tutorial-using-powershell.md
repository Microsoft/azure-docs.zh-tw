---
title: 教學課程：使用 Azure PowerShell 建立管線來移動資料 | Microsoft Docs
description: 在本教學課程中，您會使用 Azure PowerShell，建立具有複製活動的 Azure Data Factory 管線。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: 71087349-9365-4e95-9847-170658216ed8
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: bfe1d022364455f6c3e22872358b6e18b0806e6a
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43109331"
---
# <a name="tutorial-create-a-data-factory-pipeline-that-moves-data-by-using-azure-powershell"></a>教學課程︰使用 Azure PowerShell 建立 Data Factory 管線來移動資料
> [!div class="op_single_selector"]
> * [概觀和必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [複製精靈](data-factory-copy-data-wizard-tutorial.md)
> * [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)
> * [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md)
> * [PowerShell](data-factory-copy-activity-tutorial-using-powershell.md)
> * [Azure Resource Manager 範本](data-factory-copy-activity-tutorial-using-azure-resource-manager-template.md)
> * [REST API](data-factory-copy-activity-tutorial-using-rest-api.md)
> * [.NET API](data-factory-copy-activity-tutorial-using-dotnet-api.md)

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[複製活動教學課程](../quickstart-create-data-factory-powershell.md)。 

在本文中，您會了解如何使用 PowerShell 建立資料處理站，其中有管線可將資料從 Azure Blob 儲存體複製到 Azure SQL 資料庫。 如果您不熟悉 Azure Data Factory，請先詳閱 [Azure Data Factory 簡介](data-factory-introduction.md)一文，再進行本教學課程。   

在本教學課程中，您可以建立包含一個活動的管線：複製活動。 複製活動會將資料從支援的資料存放區複製到支援的接收資料存放區。 如需作為來源和接收區支援的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 此活動是由全域可用的服務所提供，可以使用安全、可靠及可調整的方式，在各種不同的資料存放區之間複製資料。 如需複製活動的詳細資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md)。

一個管線中可以有多個活動。 您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 如需詳細資訊，請參閱[管線中的多個活動](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)。

> [!NOTE]
> 這篇文章並未涵蓋所有的 Data Factory Cmdlet。 如需這些 Cmdlet 的完整文件，請參閱 [Data Factory Cmdlet 參考](/powershell/module/azurerm.datafactories)。
> 
> 本教學課程中的資料管線會將資料從來源資料存放區，複製到目的地資料存放區。 如需如何使用 Azure Data Factory 轉換資料的教學課程，請參閱[教學課程︰使用 Hadoop 叢集建置管線來轉換資料](data-factory-build-your-first-pipeline.md)。

## <a name="prerequisites"></a>必要條件
- 請完成[教學課程必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)一文中所列的必要條件。
- 安裝 **Azure PowerShell**。 遵循[如何安裝並設定 Azure PowerShell](/powershell/azure/install-azurerm-ps) 中的指示。

## <a name="steps"></a>步驟
以下是您會在本教學課程中執行的步驟：

1. 建立 Azure **Data Factory**。 在此步驟中，您會建立名為 ADFTutorialDataFactoryPSH 的資料處理站。 
1. 此資料處理站中建立**連結服務**。 在此步驟中，您會建立兩種連結服務：Azure 儲存體和 Azure SQL Database。 
    
    AzureStorageLinkedService 會將 Azure 儲存體帳戶連結至資料處理站。 您已建立容器並將資料上傳到此儲存體帳戶，作為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)的一部分。   

    AzureSqlLinkedService 會將 Azure SQL Database 連結至資料處理站。 從 Blob 儲存體複製的資料會儲存在此資料庫中。 您在此資料庫中建立了 SQL 資料表，作為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)的一部分。   
1. 在資料處理站中建立輸入和輸出**資料集**。  
    
    Azure 儲存體連結服務會指定 Data Factory 服務在執行階段用來連線到 Azure 儲存體帳戶的連接字串。 而且，輸入 Blob 資料集會指定包含輸入資料的容器和資料夾。  

    同樣第，Azure SQL Database 連結服務會指定 Data Factory 在執行階段用來連線到 Azure SQL Database 的連接字串。 而且，輸出 SQL 資料表資料集會指定資料庫中作為 Blob 儲存體資料複製目的地的資料表。
1. 在資料處理站中建立**管線**。 在此步驟中，您會建立具有複製活動的管線。   
    
    複製活動會將資料從 Azure Blob 儲存體中的 Blob 複製到 Azure SQL Database 中的資料表。 您可以在管線中使用複製活動，將資料從任何支援的來源複製到任何支援的目的地。 如需支援的資料存放區清單，請參閱[資料移動活動](data-factory-data-movement-activities.md#supported-data-stores-and-formats)一文。 
1. 監視管線。 在此步驟中，您會使用 PowerShell 來**監視**輸入和輸出資料集的配量。

## <a name="create-a-data-factory"></a>建立 Data Factory
> [!IMPORTANT]
> 如果您尚未完成[教學課程的必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)，請先這麼做。   

資料處理站可以有一或多個管線。 其中的管線可以有一或多個活動。 例如，「複製活動」會從來源將資料複製到目的地資料存放區，HDInsight Hive 活動則是執行 Hive 指令碼來轉換輸入資料，以產生輸出資料。 讓我們在這個步驟中開始建立 Data Factory。

1. 啟動 **PowerShell**。 將 Azure PowerShell 維持在開啟狀態，直到本教學課程結束為止。 如果您關閉並重新開啟，則需要再次執行這些命令。

    執行下列命令，並輸入您用來登入 Azure 入口網站的使用者名稱和密碼：

    ```PowerShell
    Connect-AzureRmAccount
    ```   
   
    執行下列命令以檢視此帳戶的所有訂用帳戶：

    ```PowerShell
    Get-AzureRmSubscription
    ```

    執行下列命令以選取您要使用的訂用帳戶。 以您的 Azure 訂用帳戶名稱取代 **&lt;NameOfAzureSubscription**&gt;：

    ```PowerShell
    Get-AzureRmSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzureRmContext
    ```
1. 執行以下命令，建立名為 **ADFTutorialResourceGroup** 的 Azure 資源群組：

    ```PowerShell
    New-AzureRmResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```
    
    本教學課程的某些步驟會假設您使用名為 **ADFTutorialResourceGroup**的資源群組。 如果使用不同的資源群組，您必須以該群組取代本教學課程中的 ADFTutorialResourceGroup。
1. 執行 **New-AzureRmDataFactory** Cmdlet，以建立名為 **ADFTutorialDataFactoryPSH** 的 Data Factory：  

    ```PowerShell
    $df=New-AzureRmDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH –Location "West US"
    ```
    此名稱可能已被使用。 因此，加上前置詞或後置詞 (例如︰ADFTutorialDataFactoryPSH05152017) 並再次執行命令，讓資料處理站的名稱成為唯一的。  

請注意下列幾點：

* Azure Data Factory 的名稱在全域必須是唯一的。 如果您收到錯誤，請變更名稱 (例如 yournameADFTutorialDataFactoryPSH)。 執行本教學課程中的步驟時，請使用此名稱來取代 ADFTutorialFactoryPSH。 請參閱 [Data Factory - 命名規則](data-factory-naming-rules.md)，以了解 Data Factory 成品的命名規則。

    ```
    Data factory name “ADFTutorialDataFactoryPSH” is not available
    ```
* 若要建立 Data Factory 執行個體，您必須是 Azure 訂用帳戶的參與者或系統管理員。
* Data Factory 的名稱未來可能會註冊為 DNS 名稱，因此會變成公開可見的名稱。
* 您可能會收到下列錯誤︰「**未註冊此訂用帳戶，無法使用命名空間 Microsoft.DataFactory。**」 請執行下列其中一個動作，並試著重新發佈一次︰

  * 在 Azure PowerShell 中，執行下列命令以註冊 Data Factory 提供者：

    ```PowerShell
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```

    執行下列命令來確認已註冊 Data Factory 提供者：

    ```PowerShell
    Get-AzureRmResourceProvider
    ```
  * 使用 Azure 訂用帳戶登入 [Azure 入口網站](https://portal.azure.com)。 請移至 Data Factory 刀鋒視窗，或在 Azure 入口網站中建立 Data Factory。 此動作會自動為您註冊提供者。

## <a name="create-linked-services"></a>建立連結的服務
您在資料處理站中建立的連結服務會將您的資料存放區和計算服務連結到資料處理站。 在本教學課程中，您不會使用任何計算服務，例如 Azure HDInsight 或 Azure Data Lake Analytics。 您可以使用兩種類型的資料存放區：Azure 儲存體 (來源) 和 Azure SQL Database (目的地)。 

因此，您可以建立名為 AzureStorageLinkedService 和 AzureSqlLinkedService 的兩個連結服務︰類型為 AzureStorage 和 AzureSqlDatabase。  

AzureStorageLinkedService 會將 Azure 儲存體帳戶連結至資料處理站。 此儲存體帳戶是您在其中建立容器並將資料上傳為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)一部分的帳戶。   

AzureSqlLinkedService 會將 Azure SQL Database 連結至資料處理站。 從 Blob 儲存體複製的資料會儲存在此資料庫中。 您在此資料庫中建立了 emp 資料表，作為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)的一部分。 

### <a name="create-a-linked-service-for-an-azure-storage-account"></a>建立 Azure 儲存體帳戶的連結服務
在此步驟中，您會將您的 Azure 儲存體帳戶連結到您的資料處理站。

1. 在 **C:\ADFGetStartedPSH** 資料夾中，使用以下內容建立名為 **AzureStorageLinkedService.json** 的 JSON 檔案：(如果 ADFGetStartedPSH 資料夾不存在，則加以建立。)

    > [!IMPORTANT]
    > 儲存檔案前，以 Azure 儲存體帳戶的名稱和金鑰取代 &lt;accountname&gt; 和 &lt;accountkey&gt;。 

    ```json
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
1. 在 **Azure PowerShell** 中，切換到 **ADFGetStartedPSH** 資料夾。
1. 執行 **New-AzureRmDataFactoryLinkedService** Cmdlet 建立連結服務：**AzureStorageLinkedService**。 此 Cmdlet 和您在本教學課程中使用的其他 Data Factory Cmdlet，皆需要您將值傳給 **ResourceGroupName** 和 **DataFactoryName** 參數。 或者，您可以傳遞 New-AzureRmDataFactory Cmdlet 所傳回的 DataFactory 物件，就不需要在每次執行 Cmdlet 時輸入 ResourceGroupName 和 DataFactoryName。 

    ```PowerShell
    New-AzureRmDataFactoryLinkedService $df -File .\AzureStorageLinkedService.json
    ```
    以下是範例輸出：

    ```
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.LinkedServiceProperties
    ProvisioningState : Succeeded
    ``` 

    其他建立此連結服務的方法就是指定資源群組名稱和資料處理站名稱，而不是指定 DataFactory 物件。  

    ```PowerShell
    New-AzureRmDataFactoryLinkedService -ResourceGroupName ADFTutorialResourceGroup -DataFactoryName <Name of your data factory> -File .\AzureStorageLinkedService.json
    ```

### <a name="create-a-linked-service-for-an-azure-sql-database"></a>建立 Azure SQL Database 的連結服務
在此步驟中，您會將您的 Azure SQL Database 連結到您的 Data Factory。

1. 在 C:\ADFGetStartedPSH 資料夾中，使用以下內容建立名為 AzureSqlLinkedService.json 的 JSON 檔案：

    > [!IMPORTANT]
    > 將 &lt;servername&gt;、&lt;databasename&gt;、&lt;username@servername&gt; 和 &lt;password&gt; 替換為您的 Azure SQL 伺服器名稱、資料庫名稱、使用者帳戶和密碼。
    
    ```json
    {
        "name": "AzureSqlLinkedService",
        "properties": {
            "type": "AzureSqlDatabase",
            "typeProperties": {
                "connectionString": "Server=tcp:<server>.database.windows.net,1433;Database=<databasename>;User ID=<user>@<server>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        }
     }
    ```
1. 執行以下命令建立連結服務：

    ```PowerShell
    New-AzureRmDataFactoryLinkedService $df -File .\AzureSqlLinkedService.json
    ```
    
    以下是範例輸出：

    ```
    LinkedServiceName : AzureSqlLinkedService
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.LinkedServiceProperties
    ProvisioningState : Succeeded
    ```

   確認 Azure SQL Server 已開啟 [允許存取 Azure 服務] 設定。 若要確認並開啟此設定，請執行下列步驟：

    1. 登入 [Azure 入口網站](https://portal.azure.com)
    1. 按一下左邊的 [更多服務 >]，然後按一下 [資料庫] 類別中的 [SQL Server]。
    1. 在 SQL Server 清單中選取您的伺服器。
    1. 在 [SQL Server] 刀鋒視窗中，按一下 [顯示防火牆設定] 連結。
    1. 在 [防火牆設定] 刀鋒視窗中，對 [允許存取 Azure 服務] 按一下 [開啟]。
    1. 按一下工具列的 [儲存]  。 

## <a name="create-datasets"></a>建立資料集
在上一個步驟中，您已建立可將 Azure 儲存體帳戶和 Azure SQL Database 連結至資料處理站的連結服務。 在此步驟中，您會定義名為 InputDataset 和 OutputDataset 的兩個資料集，它們分別代表 AzureStorageLinkedService 和 AzureSqlLinkedService 所參照資料存放區中儲存的輸入和輸出資料。

Azure 儲存體連結服務會指定 Data Factory 服務在執行階段用來連線到 Azure 儲存體帳戶的連接字串。 而且，輸入 Blob 資料集 (InputDataset) 會指定包含輸入資料的容器和資料夾。  

同樣第，Azure SQL Database 連結服務會指定 Data Factory 在執行階段用來連線到 Azure SQL Database 的連接字串。 而且，輸出 SQL 資料表資料集 (OututDataset) 會指定資料庫中作為 Blob 儲存體資料複製目的地的資料表。 

### <a name="create-an-input-dataset"></a>建立輸入資料集
在此步驟中，您將在 AzureStorageLinkedService 連結服務所代表的 Azure 儲存體中，建立名為 InputDataset 的資料集，該資料集會指向 Blob 容器 (adftutorial) 根資料夾中的 Blob 檔案 (emp.txt)。 如果您未指定 (或跳過) fileName 的值，則輸入資料夾中所有 Blob 資料都會複製到目的地。 在本教學課程中，您可指定 fileName 的值。  

1. 在 **C:\ADFGetStartedPSH** 資料夾中，使用下列內容建立名為 **InputDataset.json** 的 JSON 檔案：

    ```json
    {
        "name": "InputDataset",
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
                "fileName": "emp.txt",
                "folderPath": "adftutorial/",
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
1. 執行以下命令建立 Data Factory 資料集。

    ```PowerShell  
    New-AzureRmDataFactoryDataset $df -File .\InputDataset.json
    ```
    以下是範例輸出：

    ```
    DatasetName       : InputDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Availability      : Microsoft.Azure.Management.DataFactories.Common.Models.Availability
    Location          : Microsoft.Azure.Management.DataFactories.Models.AzureBlobDataset
    Policy            : Microsoft.Azure.Management.DataFactories.Common.Models.Policy
    Structure         : {FirstName, LastName}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DatasetProperties
    ProvisioningState : Succeeded
    ```

### <a name="create-an-output-dataset"></a>建立輸出資料表
在此步驟的這個部分中，您會建立名為 **OutputDataset**的輸出資料集。 此資料集指向 Azure SQL Database 中 **AzureSqlLinkedService**所代表的 SQL 資料表。 

1. 在 **C:\ADFGetStartedPSH** 資料夾中，使用下列內容建立名為 **OutputDataset.json** 的 JSON 檔案：

    ```json
    {
        "name": "OutputDataset",
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
1. 執行下列命令來建立 Data Factory 資料集。

    ```PowerShell   
    New-AzureRmDataFactoryDataset $df -File .\OutputDataset.json
    ```

    以下是範例輸出：

    ```
    DatasetName       : OutputDataset
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Availability      : Microsoft.Azure.Management.DataFactories.Common.Models.Availability
    Location          : Microsoft.Azure.Management.DataFactories.Models.AzureSqlTableDataset
    Policy            :
    Structure         : {FirstName, LastName}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DatasetProperties
    ProvisioningState : Succeeded
    ```

## <a name="create-a-pipeline"></a>建立管線
在此步驟中您會建立管線，其中含有使用 **InputDataset** 作為輸入和使用 **OutputDataset** 作為輸出的**複製活動**。

目前，驅動排程的是輸出資料集。 在本教學課程中，輸出資料集設定成一小時產生一次配量。 管線具有相隔一天 (也就是 24 小時) 的開始時間和結束時間。 因此，管線會產生輸出資料集的 24 個配量。 


1. 在 **C:\ADFGetStartedPSH** 資料夾中，使用下列內容建立名為 **ADFTutorialPipeline.json** 的 JSON 檔案：

    ```json   
    {
      "name": "ADFTutorialPipeline",
      "properties": {
        "description": "Copy data from a blob to Azure SQL table",
        "activities": [
          {
            "name": "CopyFromBlobToSQL",
            "type": "Copy",
            "inputs": [
              {
                "name": "InputDataset"
              }
            ],
            "outputs": [
              {
                "name": "OutputDataset"
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
    - 活動的輸入設定為 **InputDataset**，活動的輸出則設定為 **OutputDataset**。 
    - 在 **typeProperties** 區段中，來源類型指定為 **BlobSource**，接收類型指定為 **SqlSink**。 如需複製活動作為來源和接收器支援的資料存放區完整清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 若要了解如何使用特定支援的資料存放區作為來源/接收器，請按一下資料表中的連結。  
     
    將 **start** 屬性的值替換為目前日期，並將 **end**值替換為隔天的日期。 在日期時間中，您只指定日期部分，並略過時間部分。 例如，"2016-02-03"，這相當於 "2016-02-03T00:00:00Z"
     
    開始和結束日期時間都必須是 [ISO 格式](http://en.wikipedia.org/wiki/ISO_8601)。 例如：2016-10-14T16:32:41Z。 **end** 時間為選擇性項目，但在本教學課程中會用到。 
     
    如果您未指定 **end** 屬性的值，則會以「**start + 48 小時**」計算。 若要無限期地執行管線，請指定 **9999-09-09** 做為 **end** 屬性的值。
     
    在上述範例中，由於每小時即產生一個資料配量，共會有 24 個資料配量。

    如需管線定義中 JSON 屬性的說明，請參閱[建立管線](data-factory-create-pipelines.md)一文。 如需複製活動定義中 JSON 屬性的說明，請參閱[資料移動活動](data-factory-data-movement-activities.md)。 如需 BlobSource 所支援 JSON 屬性的說明，請參閱 [Azure Blob 連接器](data-factory-azure-blob-connector.md)一文。 如需 SqlSink 支援的 JSON 屬性說明，請參閱 [Azure SQL Database 連接器](data-factory-azure-sql-connector.md)一文。
1. 執行下列命令來建立 Data Factory 資料表。

    ```PowerShell   
    New-AzureRmDataFactoryPipeline $df -File .\ADFTutorialPipeline.json
    ```

    以下是範例輸出： 

    ```
    PipelineName      : ADFTutorialPipeline
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    Properties        : Microsoft.Azure.Management.DataFactories.Models.PipelinePropertie
    ProvisioningState : Succeeded
    ```

**恭喜！** 您已成功建立 Azure Data Factory，其中有管線可將資料從 Azure Blob 儲存體複製到 Azure SQL Database。 

## <a name="monitor-the-pipeline"></a>監視管線
在此步驟中，您會使用 Azure PowerShell 來監視 Azure Data Factory 的運作情形。

1. 以您的資料處理站名稱取代 &lt;DataFactoryName&gt; 並執行 **Get-AzureRmDataFactory**，然後將輸出指派給變數 $df。

    ```PowerShell  
    $df=Get-AzureRmDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name <DataFactoryName>
    ```

    例如︰
    ```PowerShell
    $df=Get-AzureRmDataFactory -ResourceGroupName ADFTutorialResourceGroup -Name ADFTutorialDataFactoryPSH0516
    ```
    
    然後，執行列印 $df 的內容可看到下列輸出︰ 
    
    ```
    PS C:\ADFGetStartedPSH> $df
    
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DataFactoryId     : 6f194b34-03b3-49ab-8f03-9f8a7b9d3e30
    ResourceGroupName : ADFTutorialResourceGroup
    Location          : West US
    Tags              : {}
    Properties        : Microsoft.Azure.Management.DataFactories.Models.DataFactoryProperties
    ProvisioningState : Succeeded
    ```
1. 執行 **Get-AzureRmDataFactorySlice**，以取得 **OutputDataset** (管線的輸出資料表) 所有配量的詳細資料。  

    ```PowerShell   
    Get-AzureRmDataFactorySlice $df -DatasetName OutputDataset -StartDateTime 2017-05-11T00:00:00Z
    ```

   此設定應符合管線 JSON 中的 **Start** 值。 您應該會看到 24 個配量，從今天 12 AM 到隔天 12 AM，每小時各一個。

   以下是輸出中的三個範例配量︰ 

    ``` 
    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 11:00:00 PM
    End               : 5/12/2017 12:00:00 AM
    RetryCount        : 0
    State             : Ready
    SubState          :
    LatencyStatus     :
    LongRetryCount    : 0

    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 9:00:00 PM
    End               : 5/11/2017 10:00:00 PM
    RetryCount        : 0
    State             : InProgress
    SubState          :
    LatencyStatus     :
    LongRetryCount    : 0   

    ResourceGroupName : ADFTutorialResourceGroup
    DataFactoryName   : ADFTutorialDataFactoryPSH0516
    DatasetName       : OutputDataset
    Start             : 5/11/2017 8:00:00 PM
    End               : 5/11/2017 9:00:00 PM
    RetryCount        : 0
    State             : Waiting
    SubState          : ConcurrencyLimit
    LatencyStatus     :
    LongRetryCount    : 0
    ```
1. 執行 **Get-AzureRmDataFactoryRun**，以取得**特定**配量的活動執行詳細資料。 從前一個命令的輸出複製日期時間值，以指定 StartDateTime 參數的值。 

    ```PowerShell  
    Get-AzureRmDataFactoryRun $df -DatasetName OutputDataset -StartDateTime "5/11/2017 09:00:00 PM"
    ```

   以下是範例輸出： 

    ```
    Id                  : c0ddbd75-d0c7-4816-a775-704bbd7c7eab_636301332000000000_636301368000000000_OutputDataset
    ResourceGroupName   : ADFTutorialResourceGroup
    DataFactoryName     : ADFTutorialDataFactoryPSH0516
    DatasetName         : OutputDataset
    ProcessingStartTime : 5/16/2017 8:00:33 PM
    ProcessingEndTime   : 5/16/2017 8:01:36 PM
    PercentComplete     : 100
    DataSliceStart      : 5/11/2017 9:00:00 PM
    DataSliceEnd        : 5/11/2017 10:00:00 PM
    Status              : Succeeded
    Timestamp           : 5/16/2017 8:00:33 PM
    RetryAttempt        : 0
    Properties          : {}
    ErrorMessage        :
    ActivityName        : CopyFromBlobToSQL
    PipelineName        : ADFTutorialPipeline
    Type                : Copy  
    ```

如需 Data Factory Cmdlet 的完整文件，請參閱 [Data Factory Cmdlet 參考](/powershell/module/azurerm.datafactories)。

## <a name="summary"></a>總結
在本教學課程中，您已建立要將資料從 Azure Blob 複製到 Azure SQL 資料庫的 Azure Data Factory。 您已使用 PowerShell 建立 Data Factory、連結服務、資料集和管線。 以下是您在本教學課程中執行的高階步驟：  

1. 建立 Azure **Data Factory**。
1. 建立 **連結服務**：

   a. **Azure 儲存體**連結服務可連結保留輸入資料的 Azure 儲存體帳戶。     
   b. **Azure SQL** 連結服務可連結保留輸出資料的 SQL Database。
1. 建立可描述管線輸入資料和輸出資料的 **資料集** 。
1. 建立具有**複製活動**的**管線**，以 **BlobSource** 做為來源並以 **SqlSink** 做為接收器。

## <a name="next-steps"></a>後續步驟
在本教學課程中，您可使用 Azure Blob 儲存體作為來源資料存放區以及使用 Azure SQL Database 作為複製作業的目的地資料存放區。 下表提供複製活動所支援作為來源或目的地的資料存放區清單： 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

若要深入了解如何從資料存放區雙向複製資料，請按一下資料表中資料存放區的連結。 

