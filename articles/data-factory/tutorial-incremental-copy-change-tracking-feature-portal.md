---
title: 使用變更追蹤和 Azure Data Factory 以累加方式複製資料 | Microsoft Docs
description: '在本教學課程中，您會建立 Azure Data Factory 管線，透過累加方式將差異資料從內部部署 SQL Server 資料庫中的多個資料表，複製到 Azure SQL Database。 '
services: data-factory
documentationcenter: ''
author: dearandyxu
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/12/2018
ms.author: yexu
ms.openlocfilehash: f06094fb82f10276f7a41d1b22f6dd99836a497f
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43095505"
---
# <a name="incrementally-load-data-from-azure-sql-database-to-azure-blob-storage-using-change-tracking-information"></a>使用變更追蹤資訊，以累加方式將資料從 Azure SQL Database 載入到 Azure Blob 儲存體 
在本教學課程中，您會建立一個 Azure Data Factory 並讓其具有管線，以根據來源 Azure SQL Database 中的**變更追蹤**資訊，將差異資料載入到 Azure Blob 儲存體。  

您會在本教學課程中執行下列步驟：

> [!div class="checklist"]
> * 準備來源資料存放區
> * 建立資料處理站。
> * 建立連結的服務。 
> * 建立來源、接收和變更追蹤資料集。
> * 建立、執行和監視完整複製管線
> * 新增或更新來源資料表中的資料
> * 建立、執行和監視累加複製管線

## <a name="overview"></a>概觀
在資料整合解決方案中，我們經常會見到在初次載入資料之後，再以累加方式載入資料的情形。 在某些情況下，您的來源資料存放區於某一期間內所變更的資料，可以輕鬆地到加以切割 (例如，LastModifyTime、CreationTime)。 但也有某些情況是，您並無法明確地識別自上次處理資料後所產生的差異資料。 Azure SQL Database 和 SQL Server 等資料存放區所支援的變更追蹤技術，可供您用來識別差異資料。  本教學課程說明如何搭配使用 Azure Data Factory 與 SQL 變更追蹤技術，以透過累加方式從 Azure SQL Database 將差異資料載入至 Azure Blob 儲存體。  如需更為具體的 SQL 變更追蹤技術資訊，請參閱 [SQL Server 中的變更追蹤](/sql/relational-databases/track-changes/about-change-tracking-sql-server)。 

## <a name="end-to-end-workflow"></a>端對端工作流程
以下是使用變更追蹤技術，以累加方式載入資料的典型端對端工作流程步驟。

> [!NOTE]
> Azure SQL Database 和 SQL Server 都支援變更追蹤技術。 本教學課程會使用 Azure SQL Database 作為來源資料存放區。 但您也可以使用內部部署的 SQL Server。 

1. **初始載入歷史資料** (執行一次)：
    1. 在來源 Azure SQL Database 中啟用變更追蹤技術。
    2. 取得 Azure SQL Database 中的 SYS_CHANGE_VERSION 初始值作為基準，以擷取變更的資料。
    3. 將完整資料從 Azure SQL Database 載入到 Azure Blob 儲存體。 
2. **依排程累加載入差異資料** (在初始載入資料後定期執行)：
    1. 取得 SYS_CHANGE_VERSION 的舊值和新值。
    3. 藉由將 **sys.change_tracking_tables** 中有所變更資料列 (介於兩個 SYS_CHANGE_VERSION 值之間) 的主索引鍵，聯結到**來源資料表**中的資料來載入差異資料，然後將差異資料移動到目的地。
    4. 針對要在下一次載入的差異更新其 SYS_CHANGE_VERSION。

## <a name="high-level-solution"></a>高階解決方案
在本教學課程中，您會建立兩個管線以執行下列兩項作業：  

1. **初始載入：** 您會使用複製活動建立管線，以將整個資料從來源資料存放區 (Azure SQL Database) 複製到目的地資料存放區 (Azure Blob 儲存體)。

    ![完整載入資料](media/tutorial-incremental-copy-change-tracking-feature-portal/full-load-flow-diagram.png)
1.  **累加載入：** 您會使用下列活動建立管線，然後定期執行該管線。 
    1. 建立**兩個查閱活動**，從 Azure SQL Database 取得舊的和新的 SYS_CHANGE_VERSION，並將其傳遞給複製活動。
    2. 建立**一個複製活動**，將兩個 SYS_CHANGE_VERSION 值之間所插入/更新/刪除的資料，從 Azure SQL Database 複製到 Azure Blob 儲存體。
    3. 建立**一個預存程序活動**，以更新下一次管線執行的 SYS_CHANGE_VERSION 值。

    ![累加載入流程圖](media/tutorial-incremental-copy-change-tracking-feature-portal/incremental-load-flow-diagram.png)


如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件
* **Azure SQL Database**。 您需要使用資料庫作為**來源**資料存放區。 如果您沒有 Azure SQL Database，請參閱[建立 Azure SQL 資料庫](../sql-database/sql-database-get-started-portal.md)一文，按照步驟建立資料庫。
* **Azure 儲存體帳戶**。 您需要使用 Blob 儲存體作為**接收**資料存放區。 如果您沒有 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)一文，按照步驟來建立帳戶。 建立名為 **adftutorial** 的容器。 

### <a name="create-a-data-source-table-in-your-azure-sql-database"></a>在 Azure SQL 資料庫中建立資料來源資料表
1. 啟動 **SQL Server Management Studio**，然後連線到 Azure SQL Server。 
2. 在**伺服器總管**中，以滑鼠右鍵按一下您的**資料庫**，然後選擇 [新增查詢]。
3. 對 Azure SQL 資料庫執行下列 SQL 命令，以建立名為 `data_source_table` 的資料表作為資料來源存放區。  
    
    ```sql
    create table data_source_table
    (
        PersonID int NOT NULL,
        Name varchar(255),
        Age int
        PRIMARY KEY (PersonID)
    );

    INSERT INTO data_source_table
        (PersonID, Name, Age)
    VALUES
        (1, 'aaaa', 21),
        (2, 'bbbb', 24),
        (3, 'cccc', 20),
        (4, 'dddd', 26),
        (5, 'eeee', 22);

    ```
4. 執行下列 SQL 查詢，在您的資料庫和來源資料表 (data_source_table) 啟用**變更追蹤**機制： 

    > [!NOTE]
    > - 將 &lt;your database name&gt; 替換為具有 data_source_table 的 Azure SQL Database 名稱。 
    > - 在目前的範例中，有變更的資料會保留兩天。 如果您每隔三天以上才會載入變更的資料，則裡面可能不會包含某些已變更的資料。  您必須將 CHANGE_RETENTION 的值變更為更大的數字。 或者，請確保您用來載入已變更資料的期間是在兩天內。 如需詳細資訊，請參閱[啟用資料庫的變更追蹤](/sql/relational-databases/track-changes/enable-and-disable-change-tracking-sql-server#enable-change-tracking-for-a-database)
 
    ```sql
    ALTER DATABASE <your database name>
    SET CHANGE_TRACKING = ON  
    (CHANGE_RETENTION = 2 DAYS, AUTO_CLEANUP = ON)  
  
    ALTER TABLE data_source_table
    ENABLE CHANGE_TRACKING  
    WITH (TRACK_COLUMNS_UPDATED = ON)
    ```
5. 執行下列查詢，建立新資料表並使用預設值儲存 ChangeTracking_version： 

    ```sql
    create table table_store_ChangeTracking_version
    (
        TableName varchar(255),
        SYS_CHANGE_VERSION BIGINT,
    );

    DECLARE @ChangeTracking_version BIGINT
    SET @ChangeTracking_version = CHANGE_TRACKING_CURRENT_VERSION();  

    INSERT INTO table_store_ChangeTracking_version
    VALUES ('data_source_table', @ChangeTracking_version)
    ```
    
    > [!NOTE]
    > 如果在您啟用 SQL Database 的變更追蹤後，資料並未變更，變更追蹤版本的值會是 0。
6. 執行下列查詢，在您的 Azure SQL Database 中建立預存程序。 該管線會叫用此預存程序，以更新您在上一個步驟建立的資料表變更追蹤版本。 

    ```sql
    CREATE PROCEDURE Update_ChangeTracking_Version @CurrentTrackingVersion BIGINT, @TableName varchar(50)
    AS
    
    BEGIN
    
        UPDATE table_store_ChangeTracking_version
        SET [SYS_CHANGE_VERSION] = @CurrentTrackingVersion
    WHERE [TableName] = @TableName
    
    END    
    ```

### <a name="azure-powershell"></a>Azure PowerShell
依照[如何安裝和設定 Azure PowerShell](/powershell/azure/install-azurerm-ps)中的指示，安裝最新的 Azure PowerShell 模組。

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 啟動 **Microsoft Edge** 或 **Google Chrome** 網頁瀏覽器。 目前，只有 Microsoft Edge 和 Google Chrome 網頁瀏覽器支援 Data Factory UI。
1. 按一下左邊功能表上的 [新增]、[資料 + 分析]，再按一下 [Data Factory]。 
   
   ![新增->DataFactory](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-azure-data-factory-menu.png)
2. 在 [新增資料處理站] 頁面中，輸入 **ADFTutorialDataFactory** 作為 [名稱]。 
      
     ![新增資料處理站頁面](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-azure-data-factory.png)
 
   Azure Data Factory 的名稱必須是 **全域唯一的**。 如果您收到錯誤，請變更 Data Factory 名稱 (例如 yournameADFTutorialDataFactory)，然後試著重新建立。 請參閱 [Data Factory - 命名規則](naming-rules.md)一文，以了解 Data Factory 成品的命名規則。
  
       `Data factory name “ADFTutorialDataFactory” is not available`
3. 選取您要在其中建立資料處理站的 Azure **訂用帳戶**。 
4. 針對 [資源群組]，請執行下列其中一個步驟︰
     
      - 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。 
      - 選取 [建立新的] ，然後輸入資源群組的名稱。   
         
        若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-overview.md)。  
4. 對 [版本] 選取 [V2 (預覽)]。
5. 選取 Data Factory 的 [位置]  。 只有受到支援的位置會顯示在下拉式清單中。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。
6. 選取 [釘選到儀表板]。     
7. 按一下頁面底部的 [新增] 。      
8. 在儀表板上，您會看到狀態如下的下列圖格︰**正在部署資料處理站**。 

    ![部署資料處理站圖格](media/tutorial-incremental-copy-change-tracking-feature-portal/deploying-data-factory.png)
9. 建立完成之後，您會看到如圖中所示的 [Data Factory] 頁面。
   
   ![Data Factory 首頁](./media/tutorial-incremental-copy-change-tracking-feature-portal/data-factory-home-page.png)
10. 按一下 [撰寫與監視] 圖格，以在另一個索引標籤中啟動 Azure Data Factory 使用者介面 (UI)。
11. 在 [開始使用] 頁面中，切換至左面板中的 [編輯] 索引標籤，如下圖所示： 

    ![建立管線按鈕](./media/tutorial-incremental-copy-change-tracking-feature-portal/get-started-page.png)

## <a name="create-linked-services"></a>建立連結的服務
您在資料處理站中建立的連結服務會將您的資料存放區和計算服務連結到資料處理站。 在本節中，您會對 Azure 儲存體帳戶和 Azure SQL 資料庫建立連結服務。 

### <a name="create-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務。
在此步驟中，您會將您的 Azure 儲存體帳戶連結到 Data Factory。

1. 按一下 [連線]，然後按一下 [+ 新增]。

   ![新增連線按鈕](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-connection-button-storage.png)
2. 在 [新增連結服務] 視窗中，選取 [Azure Blob 儲存體]，然後按一下 [繼續]。 

   ![選取 Azure Blob 儲存體](./media/tutorial-incremental-copy-change-tracking-feature-portal/select-azure-storage.png)
3. 在 [新增連結服務] 視窗中，執行下列步驟： 

    1. 輸入 **AzureStorageLinkedService** 作為 [名稱]。 
    2. 為 [儲存體帳戶名稱] 選取 Azure 儲存體帳戶。 
    3. 按一下 [檔案] 。 
    
   ![Azure 儲存體帳戶設定](./media/tutorial-incremental-copy-change-tracking-feature-portal/azure-storage-linked-service-settings.png)


### <a name="create-azure-sql-database-linked-service"></a>建立 Azure SQL Database 連結服務。
在此步驟中，您會將您的 Azure SQL Database 連結到您的 Data Factory。

1. 按一下 [連線]，然後按一下 [+ 新增]。
2. 在 [新增連結服務] 視窗中，選取 [Azure SQL Database]，然後按一下 [繼續]。 
3. 在 [新增連結服務] 視窗中，執行下列步驟： 

    1. 在 [名稱] 欄位輸入 **AzureSqlDatabaseLinkedService**。 
    2. 在 [伺服器名稱] 欄位選取您的 Azure SQL 伺服器。
    4. 在 [資料庫名稱] 欄位選取您的 Azure SQL 資料庫。 
    5. 在 [使用者名稱] 欄位輸入使用者的名稱。 
    6. 在 [密碼] 欄位輸入使用者的密碼。 
    7. 按一下 [測試連線] 以測試連線。
    8. 按一下 [儲存] 以儲存連結服務。 
    
       ![Azure SQL Database 連結服務設定](./media/tutorial-incremental-copy-change-tracking-feature-portal/azure-sql-database-linked-service-settings.png)

## <a name="create-datasets"></a>建立資料集
在此步驟中，您會建立資料集以代表資料來源和資料目的地。 以及用來儲存 SYS_CHANGE_VERSION 的位置。

### <a name="create-a-dataset-to-represent-source-data"></a>建立資料集來代表來源資料 
在此步驟中，您會建立資料集來代表來源資料。 

1. 在樹狀檢視中，按一下 [+] (加號)，然後按一下 [資料集]。 

   ![新增資料集功能表](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-dataset-menu.png)
2. 選取 [Azure SQL Database]，然後按一下 [下一步]。 

   ![來源資料集類型 - Azure SQL Database](./media/tutorial-incremental-copy-change-tracking-feature-portal/select-azure-sql-database.png)
3. 您會看到用於設定資料集的新索引標籤。 你也會在樹狀檢視中看到該資料集。 在 [屬性] 視窗中，將資料集的名稱變更為 **SourceDataset**。

   ![來源資料集名稱](./media/tutorial-incremental-copy-change-tracking-feature-portal/source-dataset-name.png)    
4. 切換至 [連線] 索引標籤，執行下列步驟： 
    
    1. 選取 [AzureSqlDatabaseLinkedService] 作為 [連結服務]。 
    2. 選取 [[dbo].[data_source_table]] 作為 [資料表]。 

   ![來源連線](./media/tutorial-incremental-copy-change-tracking-feature-portal/source-dataset-connection.png)

### <a name="create-a-dataset-to-represent-data-copied-to-sink-data-store"></a>建立資料集來表示要複製到接收資料存放區的資料。 
在此步驟中，您會建立資料集來代表從來源資料存放區複製的資料。 您會在 Azure Blob 儲存體中建立 adftutorial 容器，以滿足其中一項必要條件。 建立容器 (若不存在)，或設為現有容器的名稱。 在本教學課程中，您會使用 `@CONCAT('Incremental-', pipeline().RunId, '.txt')` 運算式來動態產生輸出檔案名稱。

1. 在樹狀檢視中，按一下 [+] (加號)，然後按一下 [資料集]。 

   ![新增資料集功能表](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-dataset-menu.png)
2. 選取 [Azure Blob 儲存體]，然後按一下 [完成]。 

   ![接收資料集類型：Azure Blob 儲存體](./media/tutorial-incremental-copy-change-tracking-feature-portal/source-dataset-type.png)
3. 您會看到用於設定資料集的新索引標籤。 你也會在樹狀檢視中看到該資料集。 在 [屬性] 視窗中，將資料集的名稱變更為 **SinkDataset**。

   ![接收資料集 - 名稱](./media/tutorial-incremental-copy-change-tracking-feature-portal/sink-dataset-name.png)
4. 在 [屬性] 視窗中切換至 [連線] 索引標籤，執行下列步驟：

    1. 選取 [AzureStorageLinkedService] 作為 [連結服務]。
    2. 在 [檔案路徑] 的**資料夾**部分輸入 **adftutorial/incchgtracking**。
    3. 在 [檔案路徑] 的**檔案**部分輸入 **@CONCAT('Incremental-', pipeline().RunId, '.txt')**。  

       ![接收資料集 - 連線](./media/tutorial-incremental-copy-change-tracking-feature-portal/sink-dataset-connection.png)

### <a name="create-a-dataset-to-represent-change-tracking-data"></a>建立資料集來代表變更追蹤資料 
在此步驟中，您會建立資料集來儲存變更追蹤版本。  您必須建立 table_store_ChangeTracking_version 資料表以滿足其中一項必要條件。

1. 在樹狀檢視中，按一下 [+] (加號)，然後按一下 [資料集]。 
2. 選取 [Azure SQL Database]，然後按一下 [下一步]。 
3. 您會看到用於設定資料集的新索引標籤。 你也會在樹狀檢視中看到該資料集。 在 [屬性] 視窗中，將資料集的名稱變更為 **ChangeTrackingDataset**。
4. 切換至 [連線] 索引標籤，執行下列步驟： 
    
    1. 選取 [AzureSqlDatabaseLinkedService] 作為 [連結服務]。 
    2. 選取 [[dbo].[table_store_ChangeTracking_version]] 作為 [資料表]。 

## <a name="create-a-pipeline-for-the-full-copy"></a>建立完整複本的管線
在此步驟中，您會使用複製活動建立管線，將整個資料從來源資料存放區 (Azure SQL Database) 複製到目的地資料存放區 (Azure Blob 儲存體)。

1. 按一下左窗格中的 [+] (加號)，然後按一下 [管線]。 

    ![新增管線功能表](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-pipeline-menu.png)
2. 您會看到用於設定管線的新索引標籤。 你也會在樹狀檢視中看到該管線。 在 [屬性] 視窗中，將管線的名稱變更為 **FullCopyPipeline**。

    ![新增管線功能表](./media/tutorial-incremental-copy-change-tracking-feature-portal/full-copy-pipeline-name.png)
3. 在 [活動] 工具箱中展開 [資料流程]，並將 [複製] 活動拖放至管線設計工具介面，然後將名稱設為 **FullCopyActivity**。 

    ![FullCopyActivity - 名稱](./media/tutorial-incremental-copy-change-tracking-feature-portal/full-copy-activity-name.png)
4. 切換至 [來源] 索引標籤，在 [來源資料集] 欄位選取 [SourceDataset]。 

    ![複製活動 - 來源](./media/tutorial-incremental-copy-change-tracking-feature-portal/copy-activity-source.png)
5. 切換至 [接收] 索引標籤，在 [接收資料集] 欄位選取 [SinkDataset]。 

    ![複製活動 - 接收](./media/tutorial-incremental-copy-change-tracking-feature-portal/copy-activity-sink.png)
6. 若要驗證管線定義，按一下工具列上的 [驗證]。 確認沒有任何驗證錯誤。 按一下 **>>** 關閉 [管線驗證報告]。 

    ![驗證管線](./media/tutorial-incremental-copy-change-tracking-feature-portal/full-copy-pipeline-validate.png)
7. 若要發佈實體 (連結服務、資料集、管線)，按一下 [發佈]。 等待發佈成功。 

    ![發佈按鈕](./media/tutorial-incremental-copy-change-tracking-feature-portal/publish-button.png)
8. 請靜待 [發佈成功] 訊息顯示。 

    ![發佈成功](./media/tutorial-incremental-copy-change-tracking-feature-portal/publishing-succeeded.png)
9. 您也可以按一下左邊的 [顯示通知] 按鈕查看通知。 若要關閉通知視窗，按一下 [X]。

    ![顯示通知](./media/tutorial-incremental-copy-change-tracking-feature-portal/show-notifications.png)


### <a name="run-the-full-copy-pipeline"></a>執行完整的複製管線
按一下管線工具列上的 [觸發]，然後按一下 [立即觸發]。 

![立即觸發功能表](./media/tutorial-incremental-copy-change-tracking-feature-portal/trigger-now-menu.png)

### <a name="monitor-the-full-copy-pipeline"></a>監視完整的複製管線

1. 按一下左側的 [監視] 索引標籤。 您會在清單中看到管線執行和其狀態。 若要重新整理清單，按一下 [重新整理]。 [動作] 資料行中的連結可讓您檢視與此管線執行相關聯的活動執行，以及重新執行管線。 

    ![管線執行](./media/tutorial-incremental-copy-change-tracking-feature-portal/monitor-full-copy-pipeline-run.png)
2. 若要檢視與此管線執行相關聯的活動執行，按一下 [動作] 資料行中的 [檢視活動執行] 連結。 管線中只有一個活動，所以在清單中只會看到一個項目。 若要切換回 [管線執行] 檢視，按一下頂端的 [管線] 連結。 

    ![活動執行](./media/tutorial-incremental-copy-change-tracking-feature-portal/activity-runs-full-copy.png)

### <a name="review-the-results"></a>檢閱結果
您會在 `adftutorial` 容器的 `incchgtracking` 資料夾中看到名為 `incremental-<GUID>.txt` 的檔案。 

![完整複製的輸出檔案](media\tutorial-incremental-copy-change-tracking-feature-portal\full-copy-output-file.png)

此檔案應該會有 Azure SQL Database 中的資料：

```
1,aaaa,21
2,bbbb,24
3,cccc,20
4,dddd,26
5,eeee,22
```

## <a name="add-more-data-to-the-source-table"></a>新增更多資料至來源資料表

對 Azure SQL Database 執行下列查詢，以新增資料列並加以更新。 

```sql
INSERT INTO data_source_table
(PersonID, Name, Age)
VALUES
(6, 'new','50');


UPDATE data_source_table
SET [Age] = '10', [name]='update' where [PersonID] = 1

``` 

## <a name="create-a-pipeline-for-the-delta-copy"></a>建立差異複本的管線
在此步驟中，您會建立具有下列活動的管線，並定期執行此管線。 **查閱活動**會從 Azure SQL Database 取得舊的和新的 SYS_CHANGE_VERSION，並將它傳遞給複製活動。 **複製活動**會將兩個 SYS_CHANGE_VERSION 值之間所插入/更新/刪除的資料，從 Azure SQL Database 複製到 Azure Blob 儲存體。 **預存程序活動**會更新下一次管線執行的 SYS_CHANGE_VERSION 值。

1. 在 [Data Factory] 使用者介面中，切換至 [編輯] 索引標籤。按一下左窗格中的 [+] (加號)，然後按一下 [管線]。 

    ![新增管線功能表](./media/tutorial-incremental-copy-change-tracking-feature-portal/new-pipeline-menu-2.png)
2. 您會看到用於設定管線的新索引標籤。 你也會在樹狀檢視中看到該管線。 在 [屬性] 視窗中，將管線的名稱變更為 **IncrementalCopyPipeline**。

    ![管線名稱](./media/tutorial-incremental-copy-change-tracking-feature-portal/incremental-copy-pipeline-name.png)
3. 在 [活動] 工具箱中展開 [一般]，並將 [查閱] 活動拖放至管線設計工具介面。 將活動的名稱設定為 **LookupLastChangeTrackingVersionActivity**。 此活動會取得在上次複製作業中使用的變更追蹤版本，這項資訊儲存在 **table_store_ChangeTracking_version** 資料表中。

    ![查閱活動 - 名稱](./media/tutorial-incremental-copy-change-tracking-feature-portal/first-lookup-activity-name.png)
4. 切換至 [屬性] 視窗中的 [設定]，在 [來源資料集] 欄位選取 [ChangeTrackingDataset]。 

    ![查閱活動 - 設定](./media/tutorial-incremental-copy-change-tracking-feature-portal/first-lookup-activity-settings.png)
5. 將 [活動] 工具箱中的 [查閱] 活動拖放至管線設計工具介面。 將活動的名稱設定為 **LookupCurrentChangeTrackingVersionActivity**。 此活動會取得目前的變更追蹤版本。

    ![查閱活動 - 名稱](./media/tutorial-incremental-copy-change-tracking-feature-portal/second-lookup-activity-name.png)
6. 在 [屬性] 視窗中切換至 [設定] ，執行下列步驟：

    1. 在 [來源資料集] 欄位選取 [SourceDataset]。
    2. 為 [使用查詢] 選取 [查詢]。 
    3. 輸入下列 SQL 查詢作為 [查詢]。 

        ```sql
        SELECT CHANGE_TRACKING_CURRENT_VERSION() as CurrentChangeTrackingVersion
        ```

    ![查閱活動 - 設定](./media/tutorial-incremental-copy-change-tracking-feature-portal/second-lookup-activity-settings.png)
7. 在 [活動] 工具箱中展開 [資料流程]，並將 [複製] 活動拖放至管線設計工具介面。 將活動的名稱設定為 **IncrementalCopyActivity**。 此活動會將上次變更追蹤版本和目前變更追蹤版本之間的資料，複製到目的地資料存放區。 

    ![複製活動 - 名稱](./media/tutorial-incremental-copy-change-tracking-feature-portal/incremental-copy-activity-name.png)
8. 在 [屬性] 視窗中切換至 [來源] 索引標籤，執行下列步驟：

    1. 選取 [SourceDataset] 作為 [來源資料集]。 
    2. 為 [使用查詢] 選取 [查詢]。 
    3. 輸入下列 SQL 查詢作為 [查詢]。 

        ```sql
        select data_source_table.PersonID,data_source_table.Name,data_source_table.Age, CT.SYS_CHANGE_VERSION, SYS_CHANGE_OPERATION from data_source_table RIGHT OUTER JOIN CHANGETABLE(CHANGES data_source_table, @{activity('LookupLastChangeTrackingVersionActivity').output.firstRow.SYS_CHANGE_VERSION}) as CT on data_source_table.PersonID = CT.PersonID where CT.SYS_CHANGE_VERSION <= @{activity('LookupCurrentChangeTrackingVersionActivity').output.firstRow.CurrentChangeTrackingVersion}
        ```
    
    ![複製活動 - 來源設定](./media/tutorial-incremental-copy-change-tracking-feature-portal/inc-copy-source-settings.png)
9. 切換至 [接收] 索引標籤，在 [接收資料集] 欄位選取 [SinkDataset]。 

    ![複製活動 - 接收設定](./media/tutorial-incremental-copy-change-tracking-feature-portal/inc-copy-sink-settings.png)
10. 逐一**將兩個查詢活動連線至複製活動**。 將連結 [查閱] 活動**綠色**按鈕拖曳至 [複製] 活動。 

    ![將查閱和複製活動連線](./media/tutorial-incremental-copy-change-tracking-feature-portal/connect-lookup-and-copy.png)
11. 將 [活動] 工具箱中的 [預存程序] 活動拖放至管線設計工具介面。 將活動的名稱設定為 **StoredProceduretoUpdateChangeTrackingActivity**。 此活動會更新 **table_store_ChangeTracking_version** 資料表中的變更追蹤版本。

    ![預存程序活動 - 名稱](./media/tutorial-incremental-copy-change-tracking-feature-portal/stored-procedure-activity-name.png)
12. 切換至 [SQL 帳戶]* 索引標籤，選取 [AzureSqlDatabaseLinkedService] 作為 [連結服務]。 

    ![預存程序活動 - SQL 帳戶](./media/tutorial-incremental-copy-change-tracking-feature-portal/sql-account-tab.png)
13. 切換至 [預存程序] 索引標籤，然後執行下列步驟： 

    1. 針對 [預存程序名稱]，選取 **Update_ChangeTracking_Version**。  
    2. 選取 [匯入參數]。 
    3. 在 [預存程序參數] 區段中，指定參數的下列值： 

        | 名稱 | 類型 | 值 | 
        | ---- | ---- | ----- | 
        | CurrentTrackingVersion | Int64 | @{activity('LookupCurrentChangeTrackingVersionActivity').output.firstRow.CurrentChangeTrackingVersion} | 
        | TableName | 字串 | @{activity('LookupLastChangeTrackingVersionActivity').output.firstRow.TableName} | 
    
        ![預存程序活動 - 參數](./media/tutorial-incremental-copy-change-tracking-feature-portal/stored-procedure-parameters.png)
14. **將複製活動連線至預存程序活動**。 將連結 [複製] 活動的**綠色**按鈕拖曳至 [預存程序] 活動。 

    ![將複製和預存程序活動連線](./media/tutorial-incremental-copy-change-tracking-feature-portal/connect-copy-stored-procedure.png)
15. 按一下工具列上的 [驗證]。 確認沒有任何驗證錯誤。 按一下 **>>** 關閉 [管線驗證報告] 視窗。 

    ![驗證按鈕](./media/tutorial-incremental-copy-change-tracking-feature-portal/validate-button.png)
16.  按一下 [全部發佈] 按鈕，將實體 (連結的服務、資料集、管線) 發佈至 Data Factory 服務。 請靜待 [發佈成功] 訊息顯示。 

        ![發佈按鈕](./media/tutorial-incremental-copy-change-tracking-feature-portal/publish-button-2.png)    

### <a name="run-the-incremental-copy-pipeline"></a>執行累加複製管線
1. 按一下管線工具列上的 [觸發]，然後按一下 [立即觸發]。 

    ![立即觸發功能表](./media/tutorial-incremental-copy-change-tracking-feature-portal/trigger-now-menu-2.png)
2. 在 [管線執行] 視窗中，選取 [完成]。

### <a name="monitor-the-incremental-copy-pipeline"></a>監視累加複製管線
1. 按一下左側的 [監視] 索引標籤。 您會在清單中看到管線執行和其狀態。 若要重新整理清單，按一下 [重新整理]。 [動作] 資料行中的連結可讓您檢視與此管線執行相關聯的活動執行，以及重新執行管線。 

    ![管線執行](./media/tutorial-incremental-copy-change-tracking-feature-portal/inc-copy-pipeline-runs.png)
2. 若要檢視與此管線執行相關聯的活動執行，按一下 [動作] 資料行中的 [檢視活動執行] 連結。 管線中只有一個活動，所以在清單中只會看到一個項目。 若要切換回 [管線執行] 檢視，按一下頂端的 [管線] 連結。 

    ![活動執行](./media/tutorial-incremental-copy-change-tracking-feature-portal/inc-copy-activity-runs.png)


### <a name="review-the-results"></a>檢閱結果
您會在 `adftutorial` 容器的 `incchgtracking` 資料夾中看到第二個檔案。 

![累加複製的輸出檔案](media\tutorial-incremental-copy-change-tracking-feature-portal\incremental-copy-output-file.png)

此檔案應該只會有 Azure SQL Database 中的差異資料。 具有 `U` 的記錄是資料庫中更新的資料列，具有 `I` 的記錄則是一個新增的資料列。 

```
1,update,10,2,U
6,new,50,1,I
```
前三個資料行是 data_source_table 中已變更的資料。 最後兩個資料行是變更追蹤系統資料表中的中繼資料。 第四個資料行是每個已變更資料列的 SYS_CHANGE_VERSION。 第五個資料行是作業：U = 更新、I = 插入。  如需變更追蹤資訊的詳細資料，請參閱 [CHANGETABLE](/sql/relational-databases/system-functions/changetable-transact-sql)。 

```
==================================================================
PersonID Name    Age    SYS_CHANGE_VERSION    SYS_CHANGE_OPERATION
==================================================================
1        update  10     2                     U
6        new     50     1                     I
```

    
## <a name="next-steps"></a>後續步驟
進入下列教學課程，以了解如何在 Azure 上使用 Spark 叢集來轉換資料：

> [!div class="nextstepaction"]
>[在雲端中使用 Spark 叢集轉換資料](tutorial-transform-data-spark-portal.md)



