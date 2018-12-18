---
title: 教學課程：使用 Visual Studio 建立具有複製活動的管線 | Microsoft Docs
description: 在本教學課程中，您會使用 Visual Studio，建立具有複製活動的 Azure Data Factory 管線。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 1751185b-ce0a-4ab2-a9c3-e37b4d149ca3
ms.service: data-factory
ms.workload: data-services
ms.custom: vs-azure
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 1152074f6dd45d9169f507aedf01541332d6f8f2
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43096838"
---
# <a name="tutorial-create-a-pipeline-with-copy-activity-using-visual-studio"></a>教學課程：使用 Visual Studio 建立具有複製活動的管線
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
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[複製活動教學課程](../quickstart-create-data-factory-dot-net.md)。 

在本文中，您會了解如何使用 Microsoft Visual Studio 建立資料處理站，其中有管線可將資料從 Azure Blob 儲存體複製到 Azure SQL 資料庫。 如果您不熟悉 Azure Data Factory，請先詳閱 [Azure Data Factory 簡介](data-factory-introduction.md)一文，再進行本教學課程。   

在本教學課程中，您可以建立包含一個活動的管線：複製活動。 複製活動會將資料從支援的資料存放區複製到支援的接收資料存放區。 如需作為來源和接收區支援的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 此活動是由全域可用的服務所提供，可以使用安全、可靠及可調整的方式，在各種不同的資料存放區之間複製資料。 如需複製活動的詳細資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md)。

一個管線中可以有多個活動。 您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 如需詳細資訊，請參閱[管線中的多個活動](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)。

> [!NOTE] 
> 本教學課程中的資料管線會將資料從來源資料存放區，複製到目的地資料存放區。 如需如何使用 Azure Data Factory 轉換資料的教學課程，請參閱[教學課程︰使用 Hadoop 叢集建置管線來轉換資料](data-factory-build-your-first-pipeline.md)。

## <a name="prerequisites"></a>必要條件
1. 詳讀 [教學課程概觀](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) 一文並完成 **必要** 步驟。       
2. 若要建立 Data Factory 執行個體，您必須是訂用帳戶/資源群組層級的 [Data Factory 參與者](../../role-based-access-control/built-in-roles.md#data-factory-contributor) 角色成員。
3. 您必須已在電腦上安裝下列項目： 
   * Visual Studio 2013 或 Visual Studio 2015
   * 下載 Azure SDK for Visual Studio 2013 或 Visual Studio 2015。 瀏覽至 [Azure 下載頁面](https://azure.microsoft.com/downloads/)，然後按一下 [.NET] 區段中的 [VS 2013] 或 [VS 2015]。
   * 下載適用於 Visual Studio 的最新 Azure Data Factory 外掛程式：[VS 2013](https://visualstudiogallery.msdn.microsoft.com/754d998c-8f92-4aa7-835b-e89c8c954aa5) 或 [VS 2015](https://visualstudiogallery.msdn.microsoft.com/371a4cf9-0093-40fa-b7dd-be3c74f49005)。 您也可以執行下列步驟來更新外掛程式：在功能表中按一下 [工具] -> [擴充功能和更新] -> [線上] -> [Visual Studio 組件庫] -> [Microsoft Azure Data Factory Tools for Visual Studio] -> [更新]。

## <a name="steps"></a>步驟
以下是您會在本教學課程中執行的步驟：

1. 此資料處理站中建立**連結服務**。 在此步驟中，您會建立兩種連結服務：Azure 儲存體和 Azure SQL Database。 
    
    AzureStorageLinkedService 會將 Azure 儲存體帳戶連結至資料處理站。 您已建立容器並將資料上傳到此儲存體帳戶，作為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)的一部分。   

    AzureSqlLinkedService 會將 Azure SQL Database 連結至資料處理站。 從 Blob 儲存體複製的資料會儲存在此資料庫中。 您在此資料庫中建立了 SQL 資料表，作為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)的一部分。     
2. 在資料處理站中建立輸入和輸出**資料集**。  
    
    Azure 儲存體連結服務會指定 Data Factory 服務在執行階段用來連線到 Azure 儲存體帳戶的連接字串。 而且，輸入 Blob 資料集會指定包含輸入資料的容器和資料夾。  

    同樣第，Azure SQL Database 連結服務會指定 Data Factory 在執行階段用來連線到 Azure SQL Database 的連接字串。 而且，輸出 SQL 資料表資料集會指定資料庫中作為 Blob 儲存體資料複製目的地的資料表。
3. 在資料處理站中建立**管線**。 在此步驟中，您會建立具有複製活動的管線。   
    
    複製活動會將資料從 Azure Blob 儲存體中的 Blob 複製到 Azure SQL Database 中的資料表。 您可以在管線中使用複製活動，將資料從任何支援的來源複製到任何支援的目的地。 如需支援的資料存放區清單，請參閱[資料移動活動](data-factory-data-movement-activities.md#supported-data-stores-and-formats)一文。 
4. 在部署 Data Factory 實體 (連結服務、資料集/資料表和管線) 時，建立 Azure **Data Factory**。 

## <a name="create-visual-studio-project"></a>建立 Visual Studio 專案
1. 啟動 **Visual Studio 2015**。 按一下 [檔案]，指向 [新增]，然後按一下 [專案]。 您應該會看到 [新增專案]  對話方塊。  
2. 在 [新增專案] 對話方塊中，選取 **DataFactory** 範本，然後按一下 [空白 Data Factory 專案]。  
   
    ![[新增專案] 對話方塊](./media/data-factory-copy-activity-tutorial-using-visual-studio/new-project-dialog.png)
3. 指定專案名稱、方案位置和方案名稱，然後按一下 [確定]。
   
    ![Controllers\HomeController.cs](./media/data-factory-copy-activity-tutorial-using-visual-studio/solution-explorer.png)    

## <a name="create-linked-services"></a>建立連結的服務
您在資料處理站中建立的連結服務會將您的資料存放區和計算服務連結到資料處理站。 在本教學課程中，您不會使用任何計算服務，例如 Azure HDInsight 或 Azure Data Lake Analytics。 您可以使用兩種類型的資料存放區：Azure 儲存體 (來源) 和 Azure SQL Database (目的地)。 

因此，您會建立兩種連結服務：AzureStorage 和 AzureSqlDatabase。  

Azure 儲存體已連結的服務會連結 Azure 儲存體帳戶至資料處理站。 此儲存體帳戶是您在其中建立容器並將資料上傳為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)一部分的帳戶。   

Azure SQL 連結服務可將 Azure SQL Database 連結到資料處理站。 從 Blob 儲存體複製的資料會儲存在此資料庫中。 您在此資料庫中建立了 emp 資料表，作為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)的一部分。

連結服務會將資料存放區或計算服務連結至 Azure Data Factory。 如需複製活動支援的所有來源和接收，請參閱 [支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats) 。 如需 Data Factory 支援的計算服務清單，請參閱 [計算連結服務](data-factory-compute-linked-services.md) 。 在本教學課程中，您不會使用任何計算服務。 

### <a name="create-the-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務
1. 以滑鼠右鍵按一下 [方案總管] 中的 [連結服務]，指向 [新增]，然後按一下 [新增項目]。      
2. 在 [新增新項目] 對話方塊中，從清單選取 [Azure 儲存體連結服務]，然後按一下 [新增]。 
   
    ![新的連結服務](./media/data-factory-copy-activity-tutorial-using-visual-studio/new-linked-service-dialog.png)
3. 使用 Azure 儲存體帳戶的名稱及其金鑰來取代 `<accountname>` 和 `<accountkey>`。 
   
    ![Azure 儲存體連結服務](./media/data-factory-copy-activity-tutorial-using-visual-studio/azure-storage-linked-service.png)
4. 儲存 **AzureStorageLinkedService1.json** 檔案。

    如需連結服務定義中 JSON 屬性的詳細資訊，請參閱 [Azure Blob 儲存體連接器](data-factory-azure-blob-connector.md#linked-service-properties)一文。

### <a name="create-the-azure-sql-linked-service"></a>建立 Azure SQL 連結服務。
1. 再次以滑鼠右鍵按一下 [方案總管] 中的 [連結服務] 節點，指向 [新增]，然後按一下 [新增項目]。 
2. 這次，請選取 [Azure SQL 連結服務]，然後按一下 [新增]。 
3. 在 **AzureSqlLinkedService1.json 檔案**中，以您的 Azure SQL Server 名稱、資料庫名稱、使用者帳戶名稱和密碼取代 `<servername>`、`<databasename>`、`<username@servername>` 和 `<password>`。    
4. 儲存 **AzureSqlLinkedService1.json** 檔案。 
    
    如需這些 JSON 屬性的詳細資訊，請參閱 [Azure SQL Database 連接器](data-factory-azure-sql-connector.md#linked-service-properties)。


## <a name="create-datasets"></a>建立資料集
在上一個步驟中，您已建立可將 Azure 儲存體帳戶和 Azure SQL Database 連結至資料處理站的連結服務。 在此步驟中，您會定義名為 InputDataset 和 OutputDataset 的兩個資料集，它們分別代表 AzureStorageLinkedService1 和 AzureSqlLinkedService1 所參照資料存放區中儲存的輸入和輸出資料。

Azure 儲存體連結服務會指定 Data Factory 服務在執行階段用來連線到 Azure 儲存體帳戶的連接字串。 而且，輸入 Blob 資料集 (InputDataset) 會指定包含輸入資料的容器和資料夾。  

同樣第，Azure SQL Database 連結服務會指定 Data Factory 在執行階段用來連線到 Azure SQL Database 的連接字串。 而且，輸出 SQL 資料表資料集 (OututDataset) 會指定資料庫中作為 Blob 儲存體資料複製目的地的資料表。 

### <a name="create-input-dataset"></a>建立輸入資料集
在此步驟中，您將在 AzureStorageLinkedService1 連結服務所代表的 Azure 儲存體中，建立名為 InputDataset 的資料集，該資料集會指向 Blob 容器 (adftutorial) 根資料夾中的 Blob 檔案 (emp.txt)。 如果您未指定 (或跳過) fileName 的值，則輸入資料夾中所有 Blob 資料都會複製到目的地。 在本教學課程中，您可指定 fileName 的值。 

在此，您會使用「資料表」一詞，而不是「資料集」。 資料表是矩形的資料集，而且是目前唯一受支援的資料集類型。 

1. 以滑鼠右鍵按一下 [方案總管] 中的 [資料表]，指向 [新增]，然後按一下 [新增項目]。
2. 在 [新增新項目] 對話方塊中，選取 [Azure Blob]，然後按一下 [新增]。   
3. 將 JSON 文字取代為下列文字並儲存 **AzureBlobLocation1.json** 檔案。 

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
      "linkedServiceName": "AzureStorageLinkedService1",
      "typeProperties": {
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

### <a name="create-output-dataset"></a>建立輸出資料集
在此步驟中，您會建立名為 **OutputDataset**的輸出資料集。 此資料集指向 Azure SQL Database 中 **AzureSqlLinkedService1**所代表的 SQL 資料表。 

1. 再次以滑鼠右鍵按一下 [方案總管] 中的 [資料表]，指向 [新增]，然後按一下 [新增項目]。
2. 在 [新增新項目] 對話方塊中，選取 [Azure SQL]，然後按一下 [新增]。 
3. 將 JSON 文字取代成下列 JSON 並儲存 **AzureSqlTableLocation1.json** 檔案。

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
       "linkedServiceName": "AzureSqlLinkedService1",
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

## <a name="create-pipeline"></a>建立管線
在此步驟中您會建立管線，其中含有使用 **InputDataset** 作為輸入和使用 **OutputDataset** 作為輸出的**複製活動**。

目前，驅動排程的是輸出資料集。 在本教學課程中，輸出資料集設定成一小時產生一次配量。 管線具有相隔一天 (也就是 24 小時) 的開始時間和結束時間。 因此，管線會產生輸出資料集的 24 個配量。 

1. 以滑鼠右鍵按一下 [方案總管] 中的 [管線]，指向 [新增]，然後按一下 [新增項目]。  
2. 選取 [新增新項目] 對話方塊中的 [複製資料管線]，並按一下 [新增]。 
3. 將 JSON 取代為下列 JSON 並儲存 **CopyActivity1.json** 檔案。

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
             "style": "StartOfInterval",
             "retry": 0,
             "timeout": "01:00:00"
           }
         }
       ],
       "start": "2017-05-11T00:00:00Z",
       "end": "2017-05-12T00:00:00Z",
       "isPaused": false
     }
    }
    ```   
    - 在活動區段中，只會有一個 **type** 設為 **Copy** 的活動。 如需複製活動的詳細資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md)。 在 Data Factory 解決方案中，您也可以使用[資料轉換活動](data-factory-data-transformation-activities.md)。
    - 活動的輸入設定為 **InputDataset**，活動的輸出則設定為 **OutputDataset**。 
    - 在 **typeProperties** 區段中，來源類型指定為 **BlobSource**，接收類型指定為 **SqlSink**。 如需複製活動作為來源和接收器支援的資料存放區完整清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 若要了解如何使用特定支援的資料存放區作為來源/接收器，請按一下資料表中的連結。  
     
    將 **start** 屬性的值替換為目前日期，並將 **end**值替換為隔天的日期。 在日期時間中，您只指定日期部分，並略過時間部分。 例如，"2016-02-03"，這相當於 "2016-02-03T00:00:00Z"
     
    開始和結束日期時間都必須是 [ISO 格式](http://en.wikipedia.org/wiki/ISO_8601)。 例如：2016-10-14T16:32:41Z。 **end** 時間為選擇性項目，但在本教學課程中會用到。 
     
    如果您未指定 **end** 屬性的值，則會以「**start + 48 小時**」計算。 若要無限期地執行管線，請指定 **9999-09-09** 做為 **end** 屬性的值。
     
    在上述範例中，由於每小時即產生一個資料配量，共會有 24 個資料配量。

    如需管線定義中 JSON 屬性的說明，請參閱[建立管線](data-factory-create-pipelines.md)一文。 如需複製活動定義中 JSON 屬性的說明，請參閱[資料移動活動](data-factory-data-movement-activities.md)。 如需 BlobSource 所支援 JSON 屬性的說明，請參閱 [Azure Blob 連接器](data-factory-azure-blob-connector.md)一文。 如需 SqlSink 支援的 JSON 屬性說明，請參閱 [Azure SQL Database 連接器](data-factory-azure-sql-connector.md)一文。

## <a name="publishdeploy-data-factory-entities"></a>發佈/部署 Data Factory 實體
在此步驟中，您會發佈稍早建立的 Data Factory 實體 (連結的服務、資料集和管線)。 您也可以指定要建立來保存這些實體的新 Data Factory 名稱。  

1. 在 [方案總管] 中，以滑鼠右鍵按一下專案，再按一下 [發佈] 。 
2. 如果您看到 [登入您的 Microsoft 帳戶] 對話方塊，請輸入具有 Azure 訂用帳戶的帳戶認證，然後按一下 [登入]。
3. 您應該會看到下列對話方塊：
   
   ![發佈對話方塊](./media/data-factory-copy-activity-tutorial-using-visual-studio/publish.png)
4. 在 [設定 Data Factory] 頁面中，執行下列步驟： 
   
   1. 選取 [建立新的 Data Factory]  選項。
   2. 針對 [名稱] 輸入 **VSTutorialFactory**。  
      
      > [!IMPORTANT]
      > Azure Data Factory 的名稱在全域必須是唯一的。 如果您在發佈時收到與 Data Factory 名稱有關的錯誤，請變更 Data Factory 的名稱 (例如 yournameVSTutorialFactory) 並嘗試再發佈一次。 請參閱 [Data Factory - 命名規則](data-factory-naming-rules.md) 主題，以了解 Data Factory 成品的命名規則。        
      > 
      > 
   3. 針對 [訂用帳戶]  欄位選取您的 Azure 訂用帳戶。
      
      > [!IMPORTANT]
      > 如果看不到任何訂用帳戶，請確定您是使用訂用帳戶的管理員或共同管理員的帳戶進行登入。  
      > 
      > 
   4. 針對要建立的 Data Factory 選取 [資源群組]  。 
   5. 選取 Data Factory 的 [區域]  。 下拉式清單中只會顯示 Data Factory 服務支援的區域。
   6. 按 [下一步]，切換至 [發佈項目] 頁面。
      
       ![設定 Data Factory 頁面](media/data-factory-copy-activity-tutorial-using-visual-studio/configure-data-factory-page.png)   
5. 在 [發佈項目] 頁面上，確認所有 Data Factory 實體都已選取，並按 [下一步] 切換至 [摘要] 頁面。
   
   ![發佈項目頁面](media/data-factory-copy-activity-tutorial-using-visual-studio/publish-items-page.png)     
6. 檢閱摘要，然後按 [下一步] 開始部署程序，並檢視 [部署狀態]。
   
   ![發佈摘要頁面](media/data-factory-copy-activity-tutorial-using-visual-studio/publish-summary-page.png)
7. 在 [部署狀態]  頁面上，您應該會看到部署程序的狀態。 部署完成後按一下 [完成]。
 
   ![部署狀態頁面](media/data-factory-copy-activity-tutorial-using-visual-studio/deployment-status.png)

請注意下列幾點： 

* 如果您收到錯誤：「此訂用帳戶未註冊為使用命名空間 Microsoft.DataFactory」，請執行下列其中一項，然後嘗試再次發佈︰ 
  
  * 在 Azure PowerShell 中，執行下列命令以註冊 Data Factory 提供者。 

    ```PowerShell    
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```
    您可以執行下列命令來確認已註冊 Data Factory 提供者。 
    
    ```PowerShell
    Get-AzureRmResourceProvider
    ```
  * 使用 Azure 訂用帳戶登入 [Azure 入口網站](https://portal.azure.com) 並瀏覽至 [Data Factory] 刀鋒視窗 (或) 在 Azure 入口網站中建立 Data Factory。 此動作會自動為您註冊提供者。
* Data Factory 的名稱未來可能會註冊為 DNS 名稱，因此會變成公開可見的名稱。

> [!IMPORTANT]
> 若要建立 Data Factory 執行個體，您必須是 Azure 訂用帳戶的管理員/共同管理員

## <a name="monitor-pipeline"></a>監視管線
瀏覽至您的資料處理站首頁：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下左側功能表上的 [更多服務]，然後按一下 [Data Factory]。

    ![瀏覽 Data Factory](media/data-factory-copy-activity-tutorial-using-visual-studio/browse-data-factories.png)
3. 開始輸入您的資料處理站名稱。

    ![資料處理站名稱](media/data-factory-copy-activity-tutorial-using-visual-studio/enter-data-factory-name.png) 
4. 按一下結果清單中您的資料處理站，以查看您的資料處理站首頁。

    ![Data Factory 首頁](media/data-factory-copy-activity-tutorial-using-visual-studio/data-factory-home-page.png)
5. 請遵循[監視資料集和管線](data-factory-copy-activity-tutorial-using-azure-portal.md#monitor-pipeline)中的指示，監視您在本教學課程中建立的管線和資料集。 Visual Studio 目前不支援監視 Data Factory 管線。 

## <a name="summary"></a>總結
在本教學課程中，您已建立要將資料從 Azure Blob 複製到 Azure SQL 資料庫的 Azure Data Factory。 您已使用 Visual Studio 建立 Data Factory、連結服務、資料集和管線。 以下是您在本教學課程中執行的高階步驟：  

1. 建立 Azure **Data Factory**。
2. 建立 **連結服務**：
   1. **Azure 儲存體** 連結服務可連結保留輸入資料的 Azure 儲存體帳戶。     
   2. **Azure SQL** 連結服務可連結保留輸出資料的 Azure SQL Database。 
3. 建立可描述管線輸入資料和輸出資料的 **資料集**。
4. 建立具有**複製活動**的**管線**，以 **BlobSource** 做為來源並以 **SqlSink** 做為接收器。 

若要了解如何使用 HDInsight Hive 活動以利用 Azure HDInsight 叢集轉換資料，請參閱[教學課程︰使用 Hadoop 叢集建置第一個管線來轉換資料](data-factory-build-your-first-pipeline.md)。

您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 如需詳細資訊，請參閱[在 Data Factory 中排程和執行](data-factory-scheduling-and-execution.md)。 

## <a name="view-all-data-factories-in-server-explorer"></a>在 Server Explorer 中檢視所有資料處理站
本節說明如何使用 Visual Studio 中的 Server Explorer，檢視 Azure 訂用帳戶中的所有資料處理站並根據現有的資料處理站建立 Visual Studio 專案。 

1. 在 **Visual Studio** 中，按一下功能表上的 [檢視]，然後按一下 [伺服器總管]。
2. 在 [伺服器總管] 視窗中，依序展開 **Azure** 和 **Data Factory**。 如果您看到 [登入 Visual Studio]，請輸入和 Azure 訂用帳戶相關聯的**帳戶**，然後按一下 [繼續]。 輸入**密碼**，然後按一下 [登入]。 Visual Studio 會嘗試取得訂用帳戶中所有 Azure Data Factory 的相關資訊。 您會在 [Data Factory 工作清單] 視窗中看到這項作業的狀態。

    ![Server Explorer](./media/data-factory-copy-activity-tutorial-using-visual-studio/server-explorer.png)

## <a name="create-a-visual-studio-project-for-an-existing-data-factory"></a>建立現有資料處理站的 Visual Studio 專案

- 在 Server Explorer 中的資料處理站上按一下滑鼠右鍵，並選取 [將 Data Factory 匯出至新的專案]，以便根據現有的資料處理站建立 Visual Studio 專案。

    ![匯出 Data Factory 至 VS 專案](./media/data-factory-copy-activity-tutorial-using-visual-studio/export-data-factory-menu.png)  

## <a name="update-data-factory-tools-for-visual-studio"></a>更新 Visual studio 的 Data Factory 工具
若要更新 Visual Studio 的 Azure Data Factory 工具，請執行下列步驟：

1. 按一下功能表上的 [工具]，然後選取 [擴充功能和更新]。 
2. 選取左窗格中的 [更新]，然後選取 [Visual Studio 組件庫]。
3. 選取 [Visual Studio 的 Azure Data Factory 工具] 並按一下 [更新]。 如果您看不到此項目，代表您已經有最新版本的工具。 

## <a name="use-configuration-files"></a>使用組態檔
您可以在 Visual Studio 中使用組態檔，針對各個環境分別設定連結服務/資料表/管線的屬性。

請針對 Azure 儲存體連結服務考量下列 JSON 定義。 根據您部署 Data Factory 實體的環境 (開發/測試/生產)，針對 accountname 和 accountkey 指定具有不同值的 **connectionString** 。 您可以針對每個環境使用個別的組態檔來達成此行為。

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

### <a name="add-a-configuration-file"></a>新增組態檔
藉由執行下列步驟來新增每個環境的組態檔：   

1. 在 Visual Studio 解決方案中以滑鼠右鍵按一下 Data Factory 專案，指向 [新增]，然後按一下 [新增項目]。
2. 從左側的已安裝範本清單中選取 [設定]、選取 [設定檔]、輸入設定檔的 [名稱]，然後按一下 [新增]。

    ![新增組態檔](./media/data-factory-build-your-first-pipeline-using-vs/add-config-file.png)
3. 以下列格式新增設定參數和其值：

    ```json
    {
        "$schema": "http://datafactories.schema.management.azure.com/vsschemas/V1/Microsoft.DataFactory.Config.json",
        "AzureStorageLinkedService1": [
            {
                "name": "$.properties.typeProperties.connectionString",
                "value": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
            }
        ],
        "AzureSqlLinkedService1": [
            {
                "name": "$.properties.typeProperties.connectionString",
                "value":  "Server=tcp:<Azure SQL server name>.database.windows.net,1433;Database=<Azure SQL datbase>;User ID=<Username>;Password=<Password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
            }
        ]
    }
    ```

    這個範例會設定 Azure 儲存體連結服務和 Azure SQL 連結服務的 connectionString 屬性。 請注意，指定名稱的語法是 [JsonPath](http://goessner.net/articles/JsonPath/)。   

    如果 JSON 具有屬性，該屬性具有如下列程式碼所示的值陣列：  

    ```json
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
    ```

    設定如下列組態檔所示的屬性 (使用以零為起始的索引)︰

    ```json
    {
        "name": "$.properties.structure[0].name",
        "value": "FirstName"
    }
    {
        "name": "$.properties.structure[0].type",
        "value": "String"
    }
    {
        "name": "$.properties.structure[1].name",
        "value": "LastName"
    }
    {
        "name": "$.properties.structure[1].type",
        "value": "String"
    }
    ```

### <a name="property-names-with-spaces"></a>包含空格的屬性名稱
如果屬性名稱內含空格，請使用如下列範例 (資料庫伺服器名稱) 中所示的方括號：

```json
 {
     "name": "$.properties.activities[1].typeProperties.webServiceParameters.['Database server name']",
     "value": "MyAsqlServer.database.windows.net"
 }
```

### <a name="deploy-solution-using-a-configuration"></a>使用組態部署解決方案
當您在 VS 中發佈 Azure Data Factory 實體時，您可以指定想要用於該發佈作業的組態。

若要使用組態檔在 Azure Data Factory 專案中發佈實體：   

1. 以滑鼠右鍵按一下 Data Factory 專案，然後按一下 [發佈] 以查看 [發佈項目] 對話方塊。
2. 選取現有的 Data Factory，或指定值以在 [設定 Data Factory] 頁面上建立 Data Factory，然後按 [下一步]。   
3. 在 [發佈項目] 頁面：您會看到下拉式清單，其中具有 [選取部署設定] 欄位的可用設定。

    ![選取組態檔](./media/data-factory-build-your-first-pipeline-using-vs/select-config-file.png)
4. 選取您想要使用的**設定檔**，然後按 [下一步]。
5. 確認您在 [摘要] 頁面上看到 JSON 檔案的名稱，然後按 [下一步]。
6. 部署作業完成後按一下 [完成]  。

部署時，在將實體部署至 Azure Data Factory 服務之前，會使用組態檔的值來設定 JSON 檔案中的屬性值。   

## <a name="use-azure-key-vault"></a>使用 Azure 金鑰保存庫
不建議認可機密資料 (例如將字串連線至程式碼存放庫)，且通常會違反安全性原則。 請參閱 GitHub 上的 [ADF 安全發佈](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/ADFSecurePublish)範例，了解在 Azure Key Vault 中儲存機密資訊和在發行 Data Factory 實體時使用它。 Visual Studio 的安全發佈擴充功能可在 Key Vault 中儲存機密資料，且僅在連結服務 / 部署組態中指定時才予以參考。 當您將 Data Factory 實體發佈至 Azure 時，會解析這些參考。 接著這些檔案可以認可至來源存放庫而不公開任何機密資訊。


## <a name="next-steps"></a>後續步驟
在本教學課程中，您可使用 Azure Blob 儲存體作為來源資料存放區以及使用 Azure SQL Database 作為複製作業的目的地資料存放區。 下表提供複製活動所支援作為來源或目的地的資料存放區清單： 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

若要深入了解如何從資料存放區雙向複製資料，請按一下資料表中資料存放區的連結。