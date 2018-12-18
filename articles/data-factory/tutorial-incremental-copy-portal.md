---
title: 使用 Azure Data Factory 以累加方式複製資料表 | Microsoft Docs
description: 在本教學課程中，您會建立 Azure Data Factory 管線，以累加方式將資料從 Azure SQL Database 複製到 Azure Blob 儲存體。
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
ms.date: 01/11/2018
ms.author: yexu
ms.openlocfilehash: 342fdce9a0e9b47380a8d8c975703ebb7f57e3b6
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43087124"
---
# <a name="incrementally-load-data-from-an-azure-sql-database-to-azure-blob-storage"></a>以累加方式將資料從 Azure SQL Database 載入到 Azure Blob 儲存體
在本教學課程中，您會建立 Azure Data Factory 與管線，以將差異資料從 Azure SQL Database 中的資料表載入到 Azure Blob 儲存體。 

您會在本教學課程中執行下列步驟：

> [!div class="checklist"]
> * 準備資料存放區來儲存水位線值。
> * 建立資料處理站。
> * 建立連結的服務。 
> * 建立來源、接收及水位線資料集。
> * 建立管道。
> * 執行管道。
> * 監視管道執行。 
> * 檢閱結果
> * 將更多資料新增至來源。
> * 再次執行管線。
> * 監視第二次管線執行
> * 檢閱第二次執行的結果


## <a name="overview"></a>概觀
高階解決方案圖表如下： 

![以累加方式載入資料](media\tutorial-Incremental-copy-portal\incrementally-load.png)

以下是建立此解決方案的重要步驟： 

1. **選取水位線資料行**。
    選取來源資料存放區中的一個資料行，可用於切割每次執行時新增或更新的記錄。 一般來說，當建立或更新資料列時，這個選取的資料行 (例如，last_modify_time 或 ID) 中的資料會持續增加。 此資料行中的最大值就作為水位線。

2. **準備資料存放區來儲存水位線值**。 在本教學課程中，您會將水位線值儲存在 SQL 資料庫中。
    
3. **使用下列工作流程建立管線**： 
    
    此解決方案中的管道有下列活動：
  
    * 建立兩個查閱活動。 使用第一個查閱活動來取出最後一個水位線值。 使用第二個查閱活動來取出新的水位線值。 這些水位線值會傳遞給複製活動。 
    * 建立複製活動，以複製來源資料存放區的資料列，而這些資料列的水位線資料行值大於舊水位線值，且小於新水位線值。 然後，它會將來源資料存放區的差異資料複製到 Blob 儲存體作為新檔案。 
    * 建立 StoredProcedure 活動，以更新下次執行之管線的水位線值。 


如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件
* **Azure SQL Database**。 您需要使用資料庫作為來源資料存放區。 如果您沒有 SQL 資料庫，請參閱[建立 Azure SQL Database](../sql-database/sql-database-get-started-portal.md)，按照步驟來建立 SQL 資料庫。
* **Azure 儲存體**。 您需要使用 Blob 儲存體作為接收資料存放區。 如果您沒有儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)，按照步驟來建立儲存體帳戶。 建立名為 adftutorial 的容器。 

### <a name="create-a-data-source-table-in-your-sql-database"></a>在 SQL 資料庫中建立資料來源資料表
1. 開啟 SQL Server Management Studio。 在 [伺服器總管] 中，以滑鼠右鍵按一下資料庫，然後選擇 [新增查詢]。

2. 對 SQL 資料庫執行下列 SQL 命令，以建立名為 `data_source_table` 的資料表作為資料來源存放區： 
    
    ```sql
    create table data_source_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );

    INSERT INTO data_source_table
    (PersonID, Name, LastModifytime)
    VALUES
    (1, 'aaaa','9/1/2017 12:56:00 AM'),
    (2, 'bbbb','9/2/2017 5:23:00 AM'),
    (3, 'cccc','9/3/2017 2:36:00 AM'),
    (4, 'dddd','9/4/2017 3:21:00 AM'),
    (5, 'eeee','9/5/2017 8:06:00 AM');
    ```
    在本教學課程中，您會使用 LastModifytime 作為水位線資料行。 下表顯示資料來源存放區中的資料：

    ```
    PersonID | Name | LastModifytime
    -------- | ---- | --------------
    1 | aaaa | 2017-09-01 00:56:00.000
    2 | bbbb | 2017-09-02 05:23:00.000
    3 | cccc | 2017-09-03 02:36:00.000
    4 | dddd | 2017-09-04 03:21:00.000
    5 | eeee | 2017-09-05 08:06:00.000
    ```

### <a name="create-another-table-in-your-sql-database-to-store-the-high-watermark-value"></a>在 SQL 資料庫中建立另一個資料表來儲存高水位線值
1. 對 SQL 資料庫執行下列 SQL 命令，以建立名為 `watermarktable` 的資料表來儲存水位線值：  
    
    ```sql
    create table watermarktable
    (
    
    TableName varchar(255),
    WatermarkValue datetime,
    );
    ```
2. 使用來源資料存放區的資料表名稱來設定高水位線的預設值。 在本教學課程中，資料表名稱是 data_source_table。

    ```sql
    INSERT INTO watermarktable
    VALUES ('data_source_table','1/1/2010 12:00:00 AM')    
    ```
3. 檢閱資料表 `watermarktable` 中的資料。
    
    ```sql
    Select * from watermarktable
    ```
    輸出： 

    ```
    TableName  | WatermarkValue
    ----------  | --------------
    data_source_table | 2010-01-01 00:00:00.000
    ```

### <a name="create-a-stored-procedure-in-your-sql-database"></a>在 SQL 資料庫中建立預存程序 

執行下列命令，在您的 SQL 資料庫中建立預存程序：

```sql
CREATE PROCEDURE sp_write_watermark @LastModifiedtime datetime, @TableName varchar(50)
AS

BEGIN
    
    UPDATE watermarktable
    SET [WatermarkValue] = @LastModifiedtime 
WHERE [TableName] = @TableName
    
END
```

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 啟動 **Microsoft Edge** 或 **Google Chrome** 網頁瀏覽器。 目前，只有 Microsoft Edge 和 Google Chrome 網頁瀏覽器支援 Data Factory UI。
1. 按一下左邊功能表上的 [新增]、[資料 + 分析]，再按一下 [Data Factory]。 
   
   ![新增->DataFactory](./media/tutorial-incremental-copy-portal/new-azure-data-factory-menu.png)
2. 在 [新增 Data Factory] 頁面中，輸入 [ADFTutorialOnPremDF] 作為 [名稱]。 
      
     ![新增資料處理站頁面](./media/tutorial-incremental-copy-portal/new-azure-data-factory.png)
 
   Azure Data Factory 的名稱必須是 **全域唯一的**。 如果您看到有以下錯誤的紅色驚嘆號，請變更 Data Factory 名稱 (例如 yournameADFTutorialDataFactory)，然後試著重新建立。 請參閱 [Data Factory - 命名規則](naming-rules.md)一文，以了解 Data Factory 成品的命名規則。
  
       `Data factory name "ADFIncCopyTutorialDF" is not available`
3. 選取您要在其中建立資料處理站的 Azure **訂用帳戶**。 
4. 針對 [資源群組]，請執行下列其中一個步驟︰
     
      - 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。 
      - 選取 [建立新的] ，然後輸入資源群組的名稱。   
         
        若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-overview.md)。  
4. 針對 [版本] 選取 [V2]。
5. 選取 Data Factory 的 [位置]  。 只有受到支援的位置會顯示在下拉式清單中。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。
6. 選取 [釘選到儀表板]。     
7. 按一下頁面底部的 [新增] 。      
8. 在儀表板上，您會看到狀態如下的下列圖格︰**正在部署資料處理站**。 

    ![部署資料處理站圖格](media/tutorial-incremental-copy-portal/deploying-data-factory.png)
9. 建立完成之後，您會看到如圖中所示的 [Data Factory] 頁面。
   
   ![Data Factory 首頁](./media/tutorial-incremental-copy-portal/data-factory-home-page.png)
10. 按一下 [撰寫與監視] 圖格，以在另一個索引標籤中啟動 Azure Data Factory 使用者介面 (UI)。

## <a name="create-a-pipeline"></a>建立管線
在本教學課程中，您會建立具有兩個查閱活動、一個複製活動和一個 StoredProcedure 活動的管線，這些活動都在一個管線中鏈結。 

1. 在 Data Factory 使用者介面的 [開始使用] 頁面中，按一下 [建立管線] 圖格。 

   ![Data Factory 使用者介面的開始使用網頁](./media/tutorial-incremental-copy-portal/get-started-page.png)    
3. 在管線之 [屬性] 視窗的 [一般] 頁面中，輸入 [IncrementalCopyPipeline] 名稱。 

   ![管線名稱](./media/tutorial-incremental-copy-portal/pipeline-name.png)
4. 讓我們新增第一個查閱活動來取得舊的浮水印值。 在 [活動] 工具箱中展開 [一般]，並將 [查閱] 活動拖放至管線設計工具介面。 將活動名稱變更為 **LookupOldWaterMarkActivity**。

   ![第一個查閱活動 - 名稱](./media/tutorial-incremental-copy-portal/first-lookup-name.png)
5. 切換至 [設定] 索引標籤，然後按一下 [+ 新增] 以新增**來源資料集**。 在此步驟中，您會建立資料集來代表**浮水印資料表**中的資料。 此資料表包含先前複製作業中所使用的舊浮水印。 

   ![新增資料集功能表 - 舊浮水印](./media/tutorial-incremental-copy-portal/new-dataset-old-watermark.png)
6. 在 [新增資料集] 視窗中，選取 [Azure SQL Database]，然後按一下 [完成]。 您會看到為該資料集開啟的新索引標籤。 

   ![選取 Azure SQL Database](./media/tutorial-incremental-copy-portal/select-azure-sql-database-old-watermark.png)
7. 在該資料集的 [屬性] 視窗中，輸入 **WatermarkDataset** 作為 [名稱]。

   ![浮水印資料集 - 名稱](./media/tutorial-incremental-copy-portal/watermark-dataset-name.png)
8. 切換至 [連線] 索引標籤，然後按一下 [+ 新增] 即可與 Azure SQL 資料庫進行連線 (建立連結的服務)。 

   ![新增連結服務按鈕](./media/tutorial-incremental-copy-portal/watermark-dataset-new-connection-button.png)
9. 在 [新增連結服務] 視窗中，執行下列步驟：

    1. 輸入 **AzureSqlDatabaseLinkedService** 作為 [名稱]。 
    2. 選取要供**伺服器名稱**使用的 Azure SQL 伺服器。
    3. 輸入 [使用者名稱] 來存取 Azure SQL 伺服器。 
    4. 輸入使用者的 [密碼]。 
    5. 若要測試與 Azure SQL 資料庫的連線，請按一下 [測試連線]。
    6. 按一下 [檔案] 。
    7. 在 [連線] 索引標籤上，確認已為**連結服務**選取 **AzureSqlDatabaseLinkedService**。
       
        ![新增連結服務視窗](./media/tutorial-incremental-copy-portal/azure-sql-linked-service-settings.png)
10. 選取 **[dbo].[watermarktable]** 作為 [資料表]。 如果您想要預覽資料表中的資料，請按一下 [預覽資料]。

    ![浮水印資料集 - 連線設定](./media/tutorial-incremental-copy-portal/watermark-dataset-connection-settings.png)
11. 按一下頂端的 [管線] 索引標籤或左側樹狀檢視中的管線名稱，即可切換到管線編輯器。 在 [查閱] 活動的 [屬性] 視窗中，確認已為 [來源資料集] 欄位選取 **WatermarkDataset**。 

    ![管線 - 舊的浮水印資料集](./media/tutorial-incremental-copy-portal/pipeline-old-watermark-dataset-selected.png)
12. 在 [活動] 工具箱中展開 [一般]，並將另一個 [查閱] 活動拖放至管線設計工具介面，然後在 [屬性] 視窗的 [一般] 索引標籤中，將名稱設為 **LookupNewWaterMarkActivity**。 此查閱活動會從資料表取得新浮水印值，該資料表具備要複製到目的地的來源資料。 

    ![第二個查閱活動 - 名稱](./media/tutorial-incremental-copy-portal/second-lookup-activity-name.png)
13. 在第二個 [查閱] 活動的 [屬性] 視窗中，切換到 [設定] 索引標籤，然後按一下 [新增]。 您建立的資料集會指向來源資料表，其中包含新浮水印值 (LastModifyTime 最大值)。 

    ![第二個查閱活動 - 新增資料集](./media/tutorial-incremental-copy-portal/second-lookup-activity-settings-new-button.png)
14. 在 [新增資料集] 視窗中，選取 [Azure SQL Database]，然後按一下 [完成]。 您會看到為此資料集而開啟的新索引標籤。 你也會在樹狀檢視中看到該資料集。 
15. 在 [屬性] 視窗的 [一般] 索引標籤中，輸入 [SourceDataset] 作為 [名稱]。 

    ![來源資料集 - 名稱](./media/tutorial-incremental-copy-portal/source-dataset-name.png)
16. 切換至 [連線] 索引標籤，然後執行下列步驟： 

    1. 選取 [AzureSqlDatabaseLinkedService] 作為 [連結服務]。
    2. 選取 **[dbo].[data_source_table]** 作為 [資料表]。 您稍後可在本教學課程中指定對此資料集的查詢。 查詢會優先於您在此步驟中指定的資料表。 

        ![第二個查閱活動 - 新增資料集](./media/tutorial-incremental-copy-portal/source-dataset-connection.png)
17. 按一下頂端的 [管線] 索引標籤或左側樹狀檢視中的管線名稱，即可切換到管線編輯器。 在 [查閱] 活動的 [屬性] 視窗中，確認已為 [來源資料集] 欄位選取 [SourceDataset]。 
18. 為[使用查詢] 欄位選取 [查詢]，並輸入下列查詢：您從 **data_source_table** 中選取的只有 **LastModifytime** 的最大值。 如果您沒有這項查詢，因你已在資料集定義中指定資料表名稱 (data_source_table)，資料集會從資料表取得所有資料列。

    ```sql
    select MAX(LastModifytime) as NewWatermarkvalue from data_source_table
    ```

    ![第二個查閱活動 - 查詢](./media/tutorial-incremental-copy-portal/query-for-new-watermark.png)
19. 在 活動 工具箱中，展開 資料流程，並從 活動 工具箱中拖放 複製 活動，以及將名稱設定為 **IncrementalCopyActivity**。 

    ![複製活動 - 名稱](./media/tutorial-incremental-copy-portal/copy-activity-name.png)
20. 透過將 [查閱] 活動所附加的**綠色按鈕**拖曳至 [複製] 活動 **，即可將兩個 [查閱] 活動同時連線至 [複製] 活動**。 當您看到 [複製] 活動的框線顏色變為藍色時即鬆開滑鼠按鈕。 

    ![將 [查閱] 活動連線至 [複製] 活動](./media/tutorial-incremental-copy-portal/connection-lookups-to-copy.png)
21. 選取 [複製] 活動並確認您在 [屬性] 視窗中看到活動的屬性。 

    ![複製活動屬性](./media/tutorial-incremental-copy-portal/back-to-copy-activity-properties.png)
22. 在 [屬性] 視窗中切換至 [來源] 索引標籤，並執行下列步驟：

    1. 為 [來源資料集] 欄位選取 [SourceDataset]。 
    2. 為 [使用查詢] 欄位選取 [查詢]。 
    3. 為 [查詢] 欄位輸入下列 SQL 查詢。 

        ```sql
        select * from data_source_table where LastModifytime > '@{activity('LookupOldWaterMarkActivity').output.firstRow.WatermarkValue}' and LastModifytime <= '@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}'
        ```
    
        ![複製活動 - 來源](./media/tutorial-incremental-copy-portal/copy-activity-source.png)
23. 切換至 [接收] 索引標籤，然後按一下 [接收資料集] 欄位的 [+ 新增]。 

    ![新增資料集按鈕](./media/tutorial-incremental-copy-portal/new-sink-dataset-button.png)
24. 在本教學課程中，接收資料存放區是 Azure Blob 儲存體類型。 因此，選取 [Azure Blob 儲存體]，然後按一下 [新增資料集] 視窗中的 [完成]。 

    ![選取 Azure Blob 儲存體](./media/tutorial-incremental-copy-portal/select-azure-blob-storage.png)
25. 在該資料集 [屬性] 視窗中的 [一般] 索引標籤，輸入 [SinkDataset] 作為 [名稱]。 

    ![接收資料集 - 名稱](./media/tutorial-incremental-copy-portal/sink-dataset-name.png)
26. 切換至 [連線] 索引標籤，然後按一下 [+ 新增]。 在此步驟中，您將建立與 **Azure Blob 儲存體**的連線 (連結的服務)。

    ![接收資料集 - 新增連線](./media/tutorial-incremental-copy-portal/sink-dataset-new-connection.png)
26. 在 [新增連結服務] 視窗中，執行下列步驟： 

    1. 輸入 **AzureStorageLinkedService** 作為 [名稱]。 
    2. 為 [儲存體帳戶名稱] 選取 Azure 儲存體帳戶。
    3. 按一下 [檔案] 。 

        ![Azure 儲存體連結服務 - 設定](./media/tutorial-incremental-copy-portal/azure-storage-linked-service-settings.png)
27. 在 [連線] 索引標籤中，執行下列步驟：

    1. 確認已為 [連結服務] 選取 [AzureStorageLinkedService]。 
    2. 在 [檔案路徑] 欄位的**資料夾**部分中，輸入 **adftutorial/incrementalcopy**。 **adftutorial** 是 blob 容器名稱而 **incrementalcopy** 是資料夾名稱。 此程式碼片段假設您在 Blob 儲存體中有一個名為 adftutorial 的 Blob 容器。 建立容器 (若不存在)，或設為現有容器的名稱。 如果輸出資料夾 **incrementalcopy** 不存在，Azure Data Factory 將會自動建立。 您也可以對**檔案路徑**使用 [瀏覽] 按鈕來瀏覽至 blob 容器中的資料夾。 .RunId, '.txt')`.
    3. 在 [檔案路徑] 欄位的 **filename** 部分中，輸入 `@CONCAT('Incremental-', pipeline().RunId, '.txt')`。 系統會使用運算式來動態產生此檔案名稱。 每個管線執行都有唯一的識別碼。 複製活動會使用執行識別碼來產生檔案名稱。 

        ![接收資料集 - 連線設定](./media/tutorial-incremental-copy-portal/sink-dataset-connection-settings.png)
28. 按一下頂端的 [管線] 索引標籤或左側樹狀檢視中的管線名稱，即可切換到**管線**編輯器。 
29. 在 [活動] 工具箱中展開 [一般]，並將 [預存程序] 活動從 [活動] 工具箱拖放至管線設計工具介面。 將 [複製] 活動的綠色 (成功) 輸出**連線**至 [預存程序] 活動。 
    
    ![複製活動 - 來源](./media/tutorial-incremental-copy-portal/connect-copy-to-stored-procedure-activity.png)
24. 選取管線設計工具中的 [預存程序活動]，將其名稱變更為 **StoredProceduretoWriteWatermarkActivity**。 

    ![預存程序活動 - 名稱](./media/tutorial-incremental-copy-portal/stored-procedure-activity-name.png)
25. 切換至 [SQL 帳戶] 索引標籤，然後為選取 [AzureSqlDatabaseLinkedService]* 作為 [連結服務]。 

    ![預存程序活動 - SQL 帳戶](./media/tutorial-incremental-copy-portal/sp-activity-sql-account-settings.png)
26. 切換至 [預存程序] 索引標籤，然後執行下列步驟： 

    1. 針對 [預存程序名稱]，選取 **sp_write_watermark**。 
    2. 若要指定預存程序參數的值，請按一下 [匯入參數]，然後輸入參數的下列值： 

        | 名稱 | 類型 | 值 | 
        | ---- | ---- | ----- | 
        | LastModifiedtime | Datetime | @{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue} |
        | TableName | 字串 | @{activity('LookupOldWaterMarkActivity').output.firstRow.TableName} |

    ![預存程序活動 - 預存程序設定](./media/tutorial-incremental-copy-portal/sproc-activity-stored-procedure-settings.png)
27. 若要驗證管線設定，請按一下工具列上的 [驗證]。 確認沒有任何驗證錯誤。 若要關閉 [管線驗證報告] 視窗，請按一下 [>>]。   

    ![驗證管線](./media/tutorial-incremental-copy-portal/validate-pipeline.png)
28. 選取 [全部發佈] 按鈕，將實體 (連結的服務、資料集和管線) 發佈至 Azure Data Factory 服務。 請等候直至您看見成功發佈的訊息。 

    ![發佈按鈕](./media/tutorial-incremental-copy-portal/publish-button.png)

## <a name="trigger-a-pipeline-run"></a>觸發管線執行
1. 按一下工具列上 [觸發]，然後按一下 [立即觸發]。 

    ![立即觸發按鈕](./media/tutorial-incremental-copy-portal/trigger-now.png)
2. 在 [管線執行] 視窗中，選取 [完成]。 

## <a name="monitor-the-pipeline-run"></a>監視管道執行

1. 切換至左側的 [監視] 索引標籤。 您可以看到手動觸發程序所觸發的管線執行狀態。 按一下 [重新整理] 按鈕即可重新整理清單。 
    
    ![管線執行](./media/tutorial-incremental-copy-portal/pipeline-runs.png)
2. 若要檢視與此管線執行相關聯的所有活動執行，請按一下 [動作] 資料行中的第一個連結 ([檢視活動執行])。 您可以按一下頂端的 [管線]，來回到前一個檢視。 按一下 [重新整理] 按鈕即可重新整理清單。

    ![活動執行](./media/tutorial-incremental-copy-portal/activity-runs.png)

## <a name="review-the-results"></a>檢閱結果
1. 透過使用 [Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/)等工具來連線到您的 Azure 儲存體帳戶。 確認輸出檔案已建立於 **adftutorial** 容器的 **incrementalcopy** 資料夾中。

    ![第一個輸出檔案](./media/tutorial-incremental-copy-portal/first-output-file.png)
2. 開啟輸出檔，請注意，所有的資料都會從 **data_source_table** 複製到 blob 檔案。 

    ```
    1,aaaa,2017-09-01 00:56:00.0000000
    2,bbbb,2017-09-02 05:23:00.0000000
    3,cccc,2017-09-03 02:36:00.0000000
    4,dddd,2017-09-04 03:21:00.0000000
    5,eeee,2017-09-05 08:06:00.0000000
    ```
3. 檢查 `watermarktable` 中的最新值。 您會看到水位線值已更新。

    ```sql
    Select * from watermarktable
    ```
    
    輸出如下：
 
    | TableName | WatermarkValue |
    | --------- | -------------- |
    | data_source_table | 2017-09-05    8:06:00.000 | 

## <a name="add-more-data-to-source"></a>將更多資料新增至來源

將新資料插入 SQL 資料庫 (資料來源存放區)。

```sql
INSERT INTO data_source_table
VALUES (6, 'newdata','9/6/2017 2:23:00 AM')

INSERT INTO data_source_table
VALUES (7, 'newdata','9/7/2017 9:01:00 AM')
``` 

SQL 資料庫中更新的資料如下：

```
PersonID | Name | LastModifytime
-------- | ---- | --------------
1 | aaaa | 2017-09-01 00:56:00.000
2 | bbbb | 2017-09-02 05:23:00.000
3 | cccc | 2017-09-03 02:36:00.000
4 | dddd | 2017-09-04 03:21:00.000
5 | eeee | 2017-09-05 08:06:00.000
6 | newdata | 2017-09-06 02:23:00.000
7 | newdata | 2017-09-07 09:01:00.000
```


## <a name="trigger-another-pipeline-run"></a>觸發另一個管線執行
1. 切換至 [編輯] 索引標籤。如果管線沒有在設計工具中開啟，請在樹狀檢視中按一下它。 

    ![立即觸發按鈕](./media/tutorial-incremental-copy-portal/edit-tab.png)
2. 按一下工具列上 [觸發]，然後按一下 [立即觸發]。 

    ![立即觸發按鈕](./media/tutorial-incremental-copy-portal/trigger-now.png)

## <a name="monitor-the-second-pipeline-run"></a>監視第二次管線執行

1. 切換至左側的 [監視] 索引標籤。 您可以看到手動觸發程序所觸發的管線執行狀態。 按一下 [重新整理] 按鈕即可重新整理清單。 
    
    ![管線執行](./media/tutorial-incremental-copy-portal/pipeline-runs-2.png)
2. 若要檢視與此管線執行相關聯的所有活動執行，請按一下 [動作] 資料行中的第一個連結 ([檢視活動執行])。 您可以按一下頂端的 [管線]，來回到前一個檢視。 按一下 [重新整理] 按鈕即可重新整理清單。

    ![活動執行](./media/tutorial-incremental-copy-portal/activity-runs-2.png)

## <a name="verify-the-second-output"></a>確認第二個輸出

1. 在 blob 儲存體中，您會看到已建立另一個檔案。 在本教學課程中，新的檔案名稱是 `Incremental-<GUID>.txt`。 開啟該檔案，您會在其中看到兩列記錄。

    ```
    6,newdata,2017-09-06 02:23:00.0000000
    7,newdata,2017-09-07 09:01:00.0000000    
    ```
2. 檢查 `watermarktable` 中的最新值。 您會看到水位線值再次更新。

    ```sql
    Select * from watermarktable
    ```
    範例輸出： 
    
    | TableName | WatermarkValue |
    | --------- | --------------- |
    | data_source_table | 2017-09-07 09:01:00.000 |


     
## <a name="next-steps"></a>後續步驟
在本教學課程中，您已執行下列步驟： 

> [!div class="checklist"]
> * 準備資料存放區來儲存水位線值。
> * 建立資料處理站。
> * 建立連結的服務。 
> * 建立來源、接收及水位線資料集。
> * 建立管道。
> * 執行管道。
> * 監視管道執行。 
> * 檢閱結果
> * 將更多資料新增至來源。
> * 再次執行管線。
> * 監視第二次管線執行
> * 檢閱第二次執行的結果

在本教學課程中，管線已從 SQL 資料庫中的單一資料表將資料複製到 Blob 儲存體。 請前進到下列教學課程，了解如何將資料從內部部署 SQL Server 資料庫中的多個資料表複製到 SQL 資料庫。 

> [!div class="nextstepaction"]
>[以累加方式將 SQL Server 中多個資料表的資料載入到 Azure SQL Database](tutorial-incremental-copy-multiple-tables-portal.md)



