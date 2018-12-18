---
title: 教學課程︰建立 Azure Data Factory 管線來複製資料 (Azure 入口網站) |Microsoft 文件
description: 在本教學課程中，您會使用 Azure 入口網站建立具有複製活動的 Azure Data Factory 管線，以將資料從 Azure Blob 儲存體複製到 Azure SQL Database。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: d9317652-0170-4fd3-b9b2-37711272162b
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 1373882fa64ac334b92dc772fc04d4b40260cc25
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43093403"
---
# <a name="tutorial-use-azure-portal-to-create-a-data-factory-pipeline-to-copy-data"></a>本教學課程︰使用 Azure 入口網站建立 Data Factory 管線來複製資料 
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

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱[複製活動教學課程](../quickstart-create-data-factory-dot-net.md)。 

在本文中，您會了解如何使用 [Azure 入口網站](https://portal.azure.com)建立資料處理站，其中有管線可將資料從 Azure Blob 儲存體複製到 Azure SQL 資料庫。 如果您不熟悉 Azure Data Factory，請先詳閱 [Azure Data Factory 簡介](data-factory-introduction.md)一文，再進行本教學課程。   

在本教學課程中，您可以建立包含一個活動的管線：複製活動。 複製活動會將資料從支援的資料存放區複製到支援的接收資料存放區。 如需作為來源和接收區支援的資料存放區清單，請參閱[支援的資料存放區](data-factory-data-movement-activities.md#supported-data-stores-and-formats)。 此活動是由全域可用的服務所提供，可以使用安全、可靠及可調整的方式，在各種不同的資料存放區之間複製資料。 如需複製活動的詳細資訊，請參閱[資料移動活動](data-factory-data-movement-activities.md)。

一個管線中可以有多個活動。 您可以將一個活動的輸出資料集設為另一個活動的輸入資料集，藉此鏈結兩個活動 (讓一個活動接著另一個活動執行)。 如需詳細資訊，請參閱[管線中的多個活動](data-factory-scheduling-and-execution.md#multiple-activities-in-a-pipeline)。 

> [!NOTE] 
> 本教學課程中的資料管線會將資料從來源資料存放區，複製到目的地資料存放區。 如需如何使用 Azure Data Factory 轉換資料的教學課程，請參閱[教學課程︰使用 Hadoop 叢集建置管線來轉換資料](data-factory-build-your-first-pipeline.md)。

## <a name="prerequisites"></a>必要條件
請先完成[教學課程必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)一文中所列的必要條件，再執行本教學課程。

## <a name="steps"></a>步驟
以下是您會在本教學課程中執行的步驟：

1. 建立 Azure **資料處理站**。 在此步驟中，您會建立名為 ADFTutorialDataFactory 的資料處理站。 
2. 在此資料處理站中建立**連結服務**。 在此步驟中，您會建立兩種連結服務：Azure 儲存體和 Azure SQL Database。 
    
    AzureStorageLinkedService 會將 Azure 儲存體帳戶連結至資料處理站。 您已建立容器並將資料上傳到此儲存體帳戶，作為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)的一部分。   

    AzureSqlLinkedService 會將 Azure SQL Database 連結至資料處理站。 從 Blob 儲存體複製的資料會儲存在此資料庫中。 您在此資料庫中建立了 SQL 資料表，作為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)的一部分。   
3. 在資料處理站中建立輸入和輸出**資料集**。  
    
    Azure 儲存體連結服務會指定 Data Factory 服務在執行階段用來連線到 Azure 儲存體帳戶的連接字串。 而且，輸入 Blob 資料集會指定包含輸入資料的容器和資料夾。  

    同樣第，Azure SQL Database 連結服務會指定 Data Factory 在執行階段用來連線到 Azure SQL Database 的連接字串。 而且，輸出 SQL 資料表資料集會指定資料庫中作為 Blob 儲存體資料複製目的地的資料表。
4. 在資料處理站中建立**管線**。 在此步驟中，您會建立具有複製活動的管線。   
    
    複製活動會將資料從 Azure Blob 儲存體中的 Blob 複製到 Azure SQL Database 中的資料表。 您可以在管線中使用複製活動，將資料從任何支援的來源複製到任何支援的目的地。 如需支援的資料存放區清單，請參閱[資料移動活動](data-factory-data-movement-activities.md#supported-data-stores-and-formats)一文。 
5. 監視管線。 在此步驟中，您會使用 Azure 入口網站來**監視**輸入和輸出資料集的配量。 

## <a name="create-data-factory"></a>建立資料處理站
> [!IMPORTANT]
> 如果您尚未完成[教學課程的必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)，請先這麼做。   

資料處理站可以有一或多個管線。 其中的管線可以有一或多個活動。 例如，「複製活動」會從來源將資料複製到目的地資料存放區，HDInsight Hive 活動則是執行 Hive 指令碼來轉換輸入資料，以產生輸出資料。 讓我們在這個步驟中開始建立 Data Factory。

1. 登入 [Azure 入口網站](https://portal.azure.com/)後，按一下左功能表上的 [建立資源]，按一下 [資料 + 分析]，然後按一下 [Data Factory]。 
   
   ![新增->DataFactory](./media/data-factory-copy-activity-tutorial-using-azure-portal/NewDataFactoryMenu.png)    
2. 在 [ **新增 Data Factory** ] 刀鋒視窗中：
   
   1. 輸入 **ADFTutorialDataFactory** 做為**名稱**。 
      
         ![新增 Data Factory 刀鋒視窗](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-new-data-factory.png)
      
       Azure Data Factory 的名稱必須是 **全域唯一的**。 如果您收到錯誤，請變更 Data Factory 名稱 (例如 yournameADFTutorialDataFactory)，然後試著重新建立。 請參閱 [Data Factory - 命名規則](data-factory-naming-rules.md) 主題，以了解 Data Factory 成品的命名規則。
      
           Data factory name “ADFTutorialDataFactory” is not available  
      
       ![Data Factory 名稱無法使用](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-data-factory-not-available.png)
   2. 選取您要在其中建立資料處理站的 Azure **訂用帳戶**。 
   3. 針對 [資源群組]，請執行下列其中一個步驟︰
      
      - 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。 
      - 選取 [建立新的] ，然後輸入資源群組的名稱。   
         
          本教學課程的某些步驟是假設您使用 **ADFTutorialResourceGroup** 做為資源群組名稱。 若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](../../azure-resource-manager/resource-group-overview.md)。  
   4. 選取 Data Factory 的 [位置]  。 下拉式清單中只會顯示 Data Factory 服務支援的區域。
   5. 選取 [釘選到儀表板]。     
   6. 按一下頁面底部的 [新增] 。
      
      > [!IMPORTANT]
      > 若要建立 Data Factory 執行個體，您必須是訂用帳戶/資源群組層級的 [Data Factory 參與者](../../role-based-access-control/built-in-roles.md#data-factory-contributor) 角色成員。
      > 
      > Data Factory 的名稱未來可能會註冊為 DNS 名稱，因此會變成公開可見的名稱。                
      > 
      > 
3. 在儀表板上，您會看到狀態如下的下列圖格︰**正在部署資料處理站**。 

    ![部署資料處理站圖格](media/data-factory-copy-activity-tutorial-using-azure-portal/deploying-data-factory.png)
4. 建立完成之後，您會看到如圖所示的 [Data Factory]  刀鋒視窗。
   
   ![Data Factory 首頁](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-data-factory-home-page.png)

## <a name="create-linked-services"></a>建立連結的服務
您在資料處理站中建立的連結服務會將您的資料存放區和計算服務連結到資料處理站。 在本教學課程中，您不會使用任何計算服務，例如 Azure HDInsight 或 Azure Data Lake Analytics。 您可以使用兩種類型的資料存放區：Azure 儲存體 (來源) 和 Azure SQL Database (目的地)。 

因此，您可以建立名為 AzureStorageLinkedService 和 AzureSqlLinkedService 的兩個連結服務︰類型為 AzureStorage 和 AzureSqlDatabase。  

AzureStorageLinkedService 會將 Azure 儲存體帳戶連結至資料處理站。 此儲存體帳戶是您在其中建立容器並將資料上傳為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)一部分的帳戶。   

AzureSqlLinkedService 會將 Azure SQL Database 連結至資料處理站。 從 Blob 儲存體複製的資料會儲存在此資料庫中。 您在此資料庫中建立了 emp 資料表，作為[必要條件](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)的一部分。  

### <a name="create-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務
在此步驟中，您會將您的 Azure 儲存體帳戶連結到您的資料處理站。 在此區段中指定您 Azure 儲存體帳戶的名稱和金鑰。  

1. 在 [Data Factory] 刀鋒視窗中，按一下 [製作和部署] 圖格。
   
   ![[製作和部署] 磚](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-author-deploy-tile.png) 
2. 您會看到如下圖所示的 [Data Factory 編輯器]： 

    ![Data Factory 編輯器](./media/data-factory-copy-activity-tutorial-using-azure-portal/data-factory-editor.png)
3. 在編輯器中，按一下工具列上的 [新增資料存放區] 按鈕，然後從下拉式功能表中選取 [Azure 儲存體]。 在右窗格中，您應該會看到用來建立 Azure 儲存體連結服務的 JSON 範本。 
   
    ![編輯器 [新增資料存放區] 按鈕](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-editor-newdatastore-button.png)    
3. 將 `<accountname>` 和 `<accountkey>` 取代為 Azure 儲存體帳戶的帳戶名稱和帳戶金鑰值。 
   
    ![編輯器 Blob 儲存體 JSON](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-editor-blob-storage-json.png)    
4. 按一下工具列上的 [部署]。 您現在應該會在樹狀檢視中看到已部署的 **AzureStorageLinkedService**。 
   
    ![編輯器 Blob 儲存體部署](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-editor-blob-storage-deploy.png)

    如需連結服務定義中 JSON 屬性的詳細資訊，請參閱 [Azure Blob 儲存體連接器](data-factory-azure-blob-connector.md#linked-service-properties)一文。

### <a name="create-a-linked-service-for-the-azure-sql-database"></a>建立 Azure SQL Database 的連結服務
在此步驟中，您會將您的 Azure SQL Database 連結到您的 Data Factory。 在此區段中指定 Azure SQL 伺服器名稱、資料庫名稱、使用者名稱和使用者密碼。 

1. 在 [Data Factory 編輯器] 中，按一下工具列上的 [新增資料存放區] 按鈕，然後從下拉式功能表中選取 [Azure SQL Database]。 在右窗格中，您應該會看到用來建立 Azure SQL 連結服務的 JSON 範本。
2. 以您的 Azure SQL Server 名稱、資料庫名稱、使用者帳戶名稱和密碼取代 `<servername>`、`<databasename>`、`<username>@<servername>` 和 `<password>`。 
3. 按一下工具列上的 [部署]，以建立並部署 **AzureSqlLinkedService**。
4. 確認您已在 [連結服務] 下的樹狀檢視中看到 **AzureSqlLinkedService**。  

    如需這些 JSON 屬性的詳細資訊，請參閱 [Azure SQL Database 連接器](data-factory-azure-sql-connector.md#linked-service-properties)。

## <a name="create-datasets"></a>建立資料集
在上一個步驟中，您已建立可將 Azure 儲存體帳戶和 Azure SQL Database 連結至資料處理站的連結服務。 在此步驟中，您會定義名為 InputDataset 和 OutputDataset 的兩個資料集，它們分別代表 AzureStorageLinkedService 和 AzureSqlLinkedService 所參照資料存放區中儲存的輸入和輸出資料。

Azure 儲存體連結服務會指定 Data Factory 服務在執行階段用來連線到 Azure 儲存體帳戶的連接字串。 而且，輸入 Blob 資料集 (InputDataset) 會指定包含輸入資料的容器和資料夾。  

同樣第，Azure SQL Database 連結服務會指定 Data Factory 在執行階段用來連線到 Azure SQL Database 的連接字串。 而且，輸出 SQL 資料表資料集 (OututDataset) 會指定資料庫中作為 Blob 儲存體資料複製目的地的資料表。 

### <a name="create-input-dataset"></a>建立輸入資料集
在此步驟中，您將在 AzureStorageLinkedService 連結服務所代表的 Azure 儲存體中，建立名為 InputDataset 的資料集，該資料集會指向 Blob 容器 (adftutorial) 根資料夾中的 Blob 檔案 (emp.txt)。 如果您未指定 (或跳過) fileName 的值，則輸入資料夾中所有 Blob 資料都會複製到目的地。 在本教學課程中，您可指定 fileName 的值。 

1. 在 Data Factory 的 [編輯器] 中，依序按一下下拉式功能表中的 **[...更多]**、[新增資料集] 和 [Azure Blob 儲存體]。 
   
    ![新增資料集功能表](./media/data-factory-copy-activity-tutorial-using-azure-portal/new-dataset-menu.png)
2. 將右窗格中的 JSON 替換為以下 JSON 片段： 
   
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
3. 按一下工具列上的 [部署]，以建立並部署 **InputDataset** 資料集。 確認您已在樹狀檢視中看到 **InputDataset** 。

### <a name="create-output-dataset"></a>建立輸出資料集
Azure SQL Database 連結服務會指定 Data Factory 在執行階段用來連線到 Azure SQL Database 的連接字串。 您在此步驟中建立的輸出 SQL 資料表資料集 (OututDataset) 會指定資料庫中作為 Blob 儲存體資料複製目的地的資料表。

1. 在 Data Factory 的 [編輯器] 中，依序按一下下拉式功能表中的 **[...更多]**、[新增資料集] 和 [Azure SQL]。 
2. 將右窗格中的 JSON 替換為以下 JSON 片段：

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
3. 按一下工具列上的 [部署]，以建立並部署 **OutputDataset** 資料集。 確認您已在 **Datasets** 之下的樹狀檢視中看到 **OutputDataset**。 

## <a name="create-pipeline"></a>建立管線
在此步驟中您會建立管線，其中含有使用 **InputDataset** 作為輸入和使用 **OutputDataset** 作為輸出的**複製活動**。

目前，驅動排程的是輸出資料集。 在本教學課程中，輸出資料集設定成一小時產生一次配量。 管線具有相隔一天 (也就是 24 小時) 的開始時間和結束時間。 因此，管線會產生輸出資料集的 24 個配量。 

1. 在 Data Factory 的 [編輯器] 中，依序按一下下拉式功能表中的 **[...更多]** 和 [新增管線]。 或者，您也可以在樹狀檢視中，以滑鼠右鍵按一下 [管線]，再按一下 [新增管線]。
2. 將右窗格中的 JSON 替換為以下 JSON 片段： 

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
    - 開始和結束日期時間都必須是 [ISO 格式](http://en.wikipedia.org/wiki/ISO_8601)。 例如：2016-10-14T16:32:41Z。 **end** 時間為選擇性項目，但在本教學課程中會用到。 如果您未指定 **end** 屬性的值，則會以「**start + 48 小時**」計算。 若要無限期地執行管線，請指定 **9999-09-09** 做為 **end** 屬性的值。
     
    在上述範例中，由於每小時即產生一個資料配量，共會有 24 個資料配量。

    如需管線定義中 JSON 屬性的說明，請參閱[建立管線](data-factory-create-pipelines.md)一文。 如需複製活動定義中 JSON 屬性的說明，請參閱[資料移動活動](data-factory-data-movement-activities.md)。 如需 BlobSource 所支援 JSON 屬性的說明，請參閱 [Azure Blob 連接器](data-factory-azure-blob-connector.md)一文。 如需 SqlSink 支援的 JSON 屬性說明，請參閱 [Azure SQL Database 連接器](data-factory-azure-sql-connector.md)一文。
3. 按一下工具列上的 [部署]，建立並部署 **ADFTutorialPipeline**。 確認您在樹狀檢視中看到管線。 
4. 現在，按一下 **X** 關閉 [編輯器] 刀鋒視窗。再次按一下 **X**，以查看 **ADFTutorialDataFactory** 的 **Data Factory** 首頁。

**恭喜！** 您已成功建立 Azure Data Factory，其中有管線可將資料從 Azure Blob 儲存體複製到 Azure SQL Database。 


## <a name="monitor-pipeline"></a>監視管線
在此步驟中，您會使用 Azure 入口網站來監視 Azure Data Factory 的運作情形。    

### <a name="monitor-pipeline-using-monitor--manage-app"></a>使用監視及管理應用程式來監視管線
下列步驟顯示如何使用「監視及管理」應用程式來監視資料處理站中的管線︰ 

1. 在 Data Factory 首頁上按一下 [監視及管理] 圖格。
   
    ![監視及管理圖格](./media/data-factory-copy-activity-tutorial-using-azure-portal/monitor-manage-tile.png) 
2. 您應該會在個別的索引標籤中看到 [監視及管理應用程式]。 

    > [!NOTE]
    > 如果您看到網頁瀏覽器停滯在「授權中...」，請執行下列其中一個動作：清除 [封鎖第三方 Cookie 和網站資料] 核取方塊，或建立 **login.microsoftonline.com** 的例外狀況，然後再試一次開啟應用程式。

    ![監視及管理應用程式](./media/data-factory-copy-activity-tutorial-using-azure-portal/monitor-and-manage-app.png)
3. 變更 [開始時間] 和 [結束時間] 以包含您管線的開始 (2017-05-11) 和結束時間 (2017-05-12)，然後按一下 [套用]。       
3. 您會在中間窗格的清單中看到與管線開始和結束時間之間的每小時相關聯的**活動時段**。 
4. 若要查看活動時段的詳細資訊，請選取 [活動時段] 清單中的活動時段。 
    ![活動時段詳細資料](./media/data-factory-copy-activity-tutorial-using-azure-portal/activity-window-details.png)

    在右邊的 [活動時段總管] 中，您會看到截至目前 UTC 時間 (下午 8:12) 為止的所有配量都已處理 (綠色)。 尚未處理 8-9 PM、9-10 PM、10-11 PM、11 PM-12 AM 配量。

    右窗格中的 [嘗試] 區段會提供資料配量的活動執行相關資訊。 如果發生錯誤，它會提供有關錯誤的詳細資訊。 例如，如果輸入資料夾或容器不存在，而且配量處理失敗，您看到錯誤訊息，說明容器或資料夾不存在。

    ![活動執行嘗試](./media/data-factory-copy-activity-tutorial-using-azure-portal/activity-run-attempts.png) 
4. 啟動 **SQL Server Management Studio**，並連接到 Azure SQL Database，然後確認資料列已插入資料庫的 **emp** 資料表中。
    
    ![SQL 查詢結果](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-sql-query-results.png)

如需使用此應用程式的詳細資訊，請參閱 [使用監視及管理應用程式來監視和管理 Azure Data Factory 管線](data-factory-monitor-manage-app.md)。

### <a name="monitor-pipeline-using-diagram-view"></a>使用圖表檢視監視管線
您也可以使用 [圖表] 檢視來監視資料管線。  

1. 在 [Data Factory] 刀鋒視窗中，按一下 [圖表]。
   
    ![Data Factory 刀鋒視窗：圖表磚](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-datafactoryblade-diagramtile.png)
2. 您應該會看到如下圖所示的圖表： 
   
    ![圖表檢視](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-diagram-blade.png)  
5. 在 [圖表] 檢視中，按兩下 **InputDataset** 以查看資料集的配量。  
   
    ![已選取 InputDataset 的資料集](./media/data-factory-copy-activity-tutorial-using-azure-portal/DataSetsWithInputDatasetFromBlobSelected.png)   
5. 按一下 [更多資訊] 連結來查看所有資料配量。 您會看到管線開始和結束時間之間的 24 個每小時配量。 
   
    ![所有輸入資料配量](./media/data-factory-copy-activity-tutorial-using-azure-portal/all-input-slices.png)  
   
    請注意，截至目前 UTC 時間為止的所有資料配量都已 [就緒]，因為 **emp.txt** 檔案一直都存在於 Blob 容器中：**adftutorial\input**。 未來時間的配量還不是就緒狀態。 確認下方的 [最近失敗的配量]  區段中沒有任何配量。
6. 關閉刀鋒視窗，直到您看到 [圖表] 檢視 (或) 向左捲動來查看 [圖表] 檢視。 然後，按兩下 **OutputDataset**。 
8. 按一下 **OutputDataset** 的 [資料表] 刀鋒視窗上的 [更多資訊] 連結，以查看所有配量。

    ![資料配量刀鋒視窗](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-dataslices-blade.png) 
9. 請注意，截至目前 UTC 時間為止的所有配量都會從 [暫止執行] 狀態 => [進行中] ==> [就緒] 狀態移動。 過去 (目前時間之前) 的配量預設會從最新到最舊的順序處理。 比方說，如果目前時間為 8:12 PM UTC，則會在 6 PM - 7 PM 配量之前處理 7 PM - 8 PM 配量。 8 PM - 9 PM 配量預設會在時間間隔結束時 (也就是 9 PM 之後) 處理。  
10. 按一下清單中的任何資料配量，您應該會看到 [資料配量]  刀鋒視窗。 與活動時段相關聯的資料稱為配量。 配量可以是一或多個檔案。  
    
     ![資料配量刀鋒視窗](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-dataslice-blade.png)
    
     若配量不是處於 [就緒] 狀態，您可以在 [未就緒的上游配量] 清單中看到未就緒且阻礙目前配量執行的上游配量。
11. 在 [資料配量]  刀鋒視窗中，您應該會看到底部清單中的所有活動執行。 按一下 [活動執行] 查看 [活動執行詳細資料] 刀鋒視窗。 
    
    ![活動執行詳細資料](./media/data-factory-copy-activity-tutorial-using-azure-portal/ActivityRunDetails.png)

    在此刀鋒視窗中，您會看到複製作業花費多少時間、輸送量為何、已讀取和寫入多少個位元組的資料、執行開始時間、執行結束時間等。  
12. 按一下 **X** 關閉所有刀鋒視窗，直到您回到 **ADFTutorialDataFactory** 的起始刀鋒視窗。
13. (選擇性) 按一下 [資料集] 圖格或 [管線] 圖格，以取得您在上述步驟所見的刀鋒視窗。 
14. 啟動 **SQL Server Management Studio**，並連接到 Azure SQL Database，然後確認資料列已插入資料庫的 **emp** 資料表中。
    
    ![SQL 查詢結果](./media/data-factory-copy-activity-tutorial-using-azure-portal/getstarted-sql-query-results.png)


## <a name="summary"></a>總結
在本教學課程中，您已建立要將資料從 Azure Blob 複製到 Azure SQL 資料庫的 Azure Data Factory。 您已使用 Azure 入口網站建立 Data Factory、連結服務、資料集和管線。 以下是您在本教學課程中執行的高階步驟：  

1. 建立 Azure **Data Factory**。
2. 建立 **連結服務**：
   1. **Azure 儲存體** 連結服務可連結保留輸入資料的 Azure 儲存體帳戶。     
   2. **Azure SQL** 連結服務可連結保留輸出資料的 Azure SQL Database。 
3. 建立可描述管線輸入資料和輸出資料的 **資料集** 。
4. 建立具有**複製活動**的**管線**，以 **BlobSource** 做為來源並以 **SqlSink** 做為接收器。  

## <a name="next-steps"></a>後續步驟
在本教學課程中，您可使用 Azure Blob 儲存體作為來源資料存放區以及使用 Azure SQL Database 作為複製作業的目的地資料存放區。 下表提供複製活動所支援作為來源或目的地的資料存放區清單： 

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

若要深入了解如何從資料存放區雙向複製資料，請按一下資料表中資料存放區的連結。