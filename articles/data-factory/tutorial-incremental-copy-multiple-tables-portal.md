---
title: 使用 Azure Data Factory 以累加方式複製多個資料表 | Microsoft Docs
description: 在本教學課程中，您會建立 Azure Data Factory 管線，透過累加方式將差異資料從內部部署 SQL Server 資料庫中的多個資料表，複製到 Azure SQL Database。
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
ms.date: 01/20/2018
ms.author: yexu
ms.openlocfilehash: ea1e3ca76f779f442c9d22478ea93de3d5ab83f2
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43088101"
---
# <a name="incrementally-load-data-from-multiple-tables-in-sql-server-to-an-azure-sql-database"></a>以累加方式將 SQL Server 中多個資料表的資料載入到 Azure SQL Database
在本教學課程中，您會建立 Azure Data Factory 與管線，以將差異資料從內部部署 SQL Server 中的多個資料表，載入到 Azure SQL Database。    

您會在本教學課程中執行下列步驟：

> [!div class="checklist"]
> * 準備來源和目的地資料存放區。
> * 建立資料處理站。
> * 建立自我裝載整合執行階段。
> * 安裝整合執行階段。 
> * 建立連結的服務。 
> * 建立來源、接收及水位線資料集。
> * 建立、執行及監視管線。
> * 檢閱結果。
> * 新增或更新來源資料表中的資料。
> * 重新執行，並監視管線。
> * 檢閱最終結果。

## <a name="overview"></a>概觀
以下是建立此解決方案的重要步驟： 

1. **選取水位線資料行**。
    
    針對來源資料存放區中的每個資料表各選取一個資料行，以便用於識別每次執行時新增或更新的記錄。 一般來說，當建立或更新資料列時，這個選取的資料行 (例如，last_modify_time 或 ID) 中的資料會持續增加。 此資料行中的最大值就作為水位線。

1. **準備資料存放區來儲存水位線值**。   
    
    在本教學課程中，您會將水位線值儲存在 SQL 資料庫中。

1. **使用下列活動建立管線**： 
    
    a. 建立 ForEach 活動，逐一查看以參數形式傳遞到管線的來源資料表名稱清單。 此活動會針對每個來源資料表叫用下列活動，以對該資料表執行差異載入。

    b. 建立兩個查閱活動。 使用第一個查閱活動來取出最後一個水位線值。 使用第二個查閱活動來取出新的水位線值。 這些水位線值會傳遞給複製活動。

    c. 建立複製活動，以複製來源資料存放區的資料列，而這些資料列的水位線資料行值大於舊水位線值，且小於新水位線值。 然後，它會將來源資料存放區的差異資料複製到 Azure Blob 儲存體作為新檔案。

    d. 建立 StoredProcedure 活動，以更新下次執行的管線水位線值。 

    高階解決方案圖表如下： 

    ![以累加方式載入資料](media\tutorial-incremental-copy-multiple-tables-portal\high-level-solution-diagram.png)


如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先決條件
* **SQL Server**。 在本教學課程中，您會使用內部部署 SQL 資料庫作為來源資料存放區。 
* **Azure SQL Database**。 您會使用 SQL 資料庫作為接收資料存放區。 如果您沒有 SQL 資料庫，請參閱[建立 Azure SQL Database](../sql-database/sql-database-get-started-portal.md)，按照步驟來建立 SQL 資料庫。 

### <a name="create-source-tables-in-your-sql-server-database"></a>在 SQL Server 資料庫中建立來源資料表

1. 開啟 SQL Server Management Studio，然後連線到內部部署 SQL Server 資料庫。

1. 在**伺服器總管**中，以滑鼠右鍵按一下資料庫，然後選擇 [新增查詢]。

1. 對您的資料庫執行下列 SQL 命令，以建立名為 `customer_table` 和 `project_table` 的資料表：

    ```sql
    create table customer_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );
    
    create table project_table
    (
        Project varchar(255),
        Creationtime datetime
    );
        
    INSERT INTO customer_table
    (PersonID, Name, LastModifytime)
    VALUES
    (1, 'John','9/1/2017 12:56:00 AM'),
    (2, 'Mike','9/2/2017 5:23:00 AM'),
    (3, 'Alice','9/3/2017 2:36:00 AM'),
    (4, 'Andy','9/4/2017 3:21:00 AM'),
    (5, 'Anny','9/5/2017 8:06:00 AM');
    
    INSERT INTO project_table
    (Project, Creationtime)
    VALUES
    ('project1','1/1/2015 0:00:00 AM'),
    ('project2','2/2/2016 1:23:00 AM'),
    ('project3','3/4/2017 5:16:00 AM');
    
    ```

### <a name="create-destination-tables-in-your-azure-sql-database"></a>在 Azure SQL 資料庫中建立目的地資料表
1. 開啟 SQL Server Management Studio，然後連線到 Azure SQL 資料庫。

1. 在**伺服器總管**中，以滑鼠右鍵按一下資料庫，然後選擇 [新增查詢]。

1. 對您的資料庫執行下列 SQL 命令，以建立名為 `customer_table` 和 `project_table` 的 SQL 資料表：  
    
    ```sql
    create table customer_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );
    
    create table project_table
    (
        Project varchar(255),
        Creationtime datetime
    );

    ```

### <a name="create-another-table-in-the-azure-sql-database-to-store-the-high-watermark-value"></a>在 Azure SQL Database 中建立另一個資料表來儲存高水位線值
1. 對 SQL 資料庫執行下列 SQL 命令，以建立名為 `watermarktable` 的資料表來儲存水位線值： 
    
    ```sql
    create table watermarktable
    (
    
        TableName varchar(255),
        WatermarkValue datetime,
    );
    ```
1. 在水位線資料表中插入這兩個來源資料表的初始水位線值。

    ```sql

    INSERT INTO watermarktable
    VALUES 
    ('customer_table','1/1/2010 12:00:00 AM'),
    ('project_table','1/1/2010 12:00:00 AM');
    
    ```

### <a name="create-a-stored-procedure-in-the-azure-sql-database"></a>在 Azure SQL 資料庫中建立預存程序 

執行下列命令，在您的 SQL 資料庫中建立預存程序。 這個預存程序會在每次管線執行之後更新水位線值。 

```sql
CREATE PROCEDURE sp_write_watermark @LastModifiedtime datetime, @TableName varchar(50)
AS

BEGIN

    UPDATE watermarktable
    SET [WatermarkValue] = @LastModifiedtime 
WHERE [TableName] = @TableName

END

```

### <a name="create-data-types-and-additional-stored-procedures-in-azure-sql-database"></a>在 Azure SQL 資料庫中建立資料類型和其他預存程序
執行下列查詢，在您的 SQL 資料庫中建立兩個預存程序和兩個資料類型。 它們用來將來源資料表的資料合併到目的地資料表。

```sql
CREATE TYPE DataTypeforCustomerTable AS TABLE(
    PersonID int,
    Name varchar(255),
    LastModifytime datetime
);

GO

CREATE PROCEDURE sp_upsert_customer_table @customer_table DataTypeforCustomerTable READONLY
AS

BEGIN
  MERGE customer_table AS target
  USING @customer_table AS source
  ON (target.PersonID = source.PersonID)
  WHEN MATCHED THEN
      UPDATE SET Name = source.Name,LastModifytime = source.LastModifytime
  WHEN NOT MATCHED THEN
      INSERT (PersonID, Name, LastModifytime)
      VALUES (source.PersonID, source.Name, source.LastModifytime);
END

GO

CREATE TYPE DataTypeforProjectTable AS TABLE(
    Project varchar(255),
    Creationtime datetime
);

GO

CREATE PROCEDURE sp_upsert_project_table @project_table DataTypeforProjectTable READONLY
AS

BEGIN
  MERGE project_table AS target
  USING @project_table AS source
  ON (target.Project = source.Project)
  WHEN MATCHED THEN
      UPDATE SET Creationtime = source.Creationtime
  WHEN NOT MATCHED THEN
      INSERT (Project, Creationtime)
      VALUES (source.Project, source.Creationtime);
END

```

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 啟動 **Microsoft Edge** 或 **Google Chrome** 網頁瀏覽器。 目前，只有 Microsoft Edge 和 Google Chrome 網頁瀏覽器支援 Data Factory UI。
1. 按一下左邊功能表上的 [新增]、[資料 + 分析]，再按一下 [Data Factory]。 
   
   ![新增->DataFactory](./media/tutorial-incremental-copy-multiple-tables-portal/new-azure-data-factory-menu.png)
1. 在 [新增資料處理站] 頁面中，輸入 **ADFMultiIncCopyTutorialDF** 作為 [名稱]。 
      
     ![新增資料處理站頁面](./media/tutorial-incremental-copy-multiple-tables-portal/new-azure-data-factory.png)
 
   Azure Data Factory 的名稱必須是 **全域唯一的**。 如果您收到下列錯誤，請變更資料處理站的名稱 (例如 yournameADFMultiIncCopyTutorialDF)，然後試著重新建立。 請參閱 [Data Factory - 命名規則](naming-rules.md)一文，以了解 Data Factory 成品的命名規則。
  
       `Data factory name ADFMultiIncCopyTutorialDF is not available`
1. 選取您要在其中建立資料處理站的 Azure **訂用帳戶**。 
1. 針對 [資源群組]，請執行下列其中一個步驟︰
     
      - 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。 
      - 選取 [建立新的] ，然後輸入資源群組的名稱。   
         
        若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-overview.md)。  
1. 對 [版本] 選取 [V2 (預覽)]。
1. 選取 Data Factory 的 [位置]  。 只有受到支援的位置會顯示在下拉式清單中。 資料處理站所使用的資料存放區 (Azure 儲存體、Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。
1. 選取 [釘選到儀表板]。     
1. 按一下頁面底部的 [新增] 。      
1. 在儀表板上，您會看到狀態如下的下列圖格︰**正在部署資料處理站**。 

    ![部署資料處理站圖格](media/tutorial-incremental-copy-multiple-tables-portal/deploying-data-factory.png)
1. 建立完成之後，您會看到如圖中所示的 [Data Factory] 頁面。
   
   ![Data Factory 首頁](./media/tutorial-incremental-copy-multiple-tables-portal/data-factory-home-page.png)
1. 按一下 [編寫與監視] 圖格，在個別索引標籤中啟動 Azure Data Factory 使用者介面 (UI)。
1. 在 Azure Data Factory UI 的 [開始使用] 頁面中按一下 [建立管線]，或切換至 [編輯] 索引標籤。 

   ![開始使用頁面](./media/tutorial-incremental-copy-multiple-tables-portal/get-started-page.png)

## <a name="create-self-hosted-integration-runtime"></a>建立自我裝載的整合執行階段
當您將資料從私人網路中的資料存放區 (內部部署) 移至 Azure 資料存放區時，請在內部部署環境中安裝自我裝載整合執行階段 (IR)。 自我裝載 IR 會在您的私人網路與 Azure 之間移動資料。 

1. 按一下左窗格底部的 [連線]，並切換至 [連線] 視窗中的 [整合執行階段]。 

   ![連線索引標籤](./media/tutorial-incremental-copy-multiple-tables-portal/connections-tab.png)
1. 在 [整合執行階段] 索引標籤中，按一下 [+ 新增]。 

   ![新增整合執行階段 - 按鈕](./media/tutorial-incremental-copy-multiple-tables-portal/new-integration-runtime-button.png)
1. 在 [整合執行階段設定] 視窗中，選取 [執行資料移動，並分派活動到外部計算]，然後按 [下一步]。 

   ![選取整合執行階段類型](./media/tutorial-incremental-copy-multiple-tables-portal/select-integration-runtime-type.png)
1. 選取 **私人網路**，然後按 [下一步]。 

   ![選取私人網路](./media/tutorial-incremental-copy-multiple-tables-portal/select-private-network.png)
1. 輸入 **MySelfHostedIR** 作為 [名稱]，然後按一下 [下一步]。 

   ![自我裝載 IR 名稱](./media/tutorial-incremental-copy-multiple-tables-portal/self-hosted-ir-name.png)
1. 在 [選項 1：快速安裝] 區段中，按 [按一下這裡啟動此電腦的快速安裝]。 

   ![按一下 [快速安裝] 連結](./media/tutorial-incremental-copy-multiple-tables-portal/click-exress-setup.png)
1. 在 [Integration Runtime (自我裝載) 快速安裝] 視窗中，按一下 [關閉]。 

   ![整合執行階段安裝 - 成功](./media/tutorial-incremental-copy-multiple-tables-portal/integration-runtime-setup-successful.png)
1. 在網頁瀏覽器的 [整合執行階段安裝] 視窗中，按一下 [完成]。 

   ![整合執行階段安裝 - 完成](./media/tutorial-incremental-copy-multiple-tables-portal/click-finish-integration-runtime-setup.png)
1. 確認您在整合執行階段的清單中看到 **MySelfHostedIR**。

       ![Integration runtimes - list](./media/tutorial-incremental-copy-multiple-tables-portal/integration-runtimes-list.png)

## <a name="create-linked-services"></a>建立連結的服務
您在資料處理站中建立的連結服務會將您的資料存放區和計算服務連結到資料處理站。 在本節中，您會對內部部署 SQL Server 資料庫和 SQL 資料庫建立連結服務。 

### <a name="create-the-sql-server-linked-service"></a>建立 SQL Server 連結服務
在此步驟中，您要將內部部署 SQL Server 資料庫連結至資料處理站。

1. 在 [連線] 視窗中，從 [整合執行階段] 索引標籤切換至 [連結服務] 索引標籤，然後按一下 [+ 新增]。

    ![新增連結服務按鈕](./media/tutorial-incremental-copy-multiple-tables-portal/new-sql-server-linked-service-button.png)
1. 在 [新增連結服務] 視窗中，選取 [SQL Server]，然後按一下 [繼續]。 

    ![選取 SQL Server](./media/tutorial-incremental-copy-multiple-tables-portal/select-sql-server.png)
1. 在 [新增連結服務] 視窗中，執行下列步驟：

    1. 輸入 **SqlServerLinkedService** 作為 [名稱]。 
    1. 在 [透過整合執行階段連線] 選取 [MySelfHostedIR]。 這是**重要**步驟。 預設整合執行階段無法連線到內部部署資料存放區。 請使用您先前建立的自我裝載整合執行階段。 
    1. 針對 [伺服器名稱]，輸入具有 SQL Server 資料庫的電腦名稱。
    1. 針對 [資料庫名稱]，輸入 SQL Server 中具有來源資料的資料庫名稱。 您已建立資料表，並在此資料庫中插入資料，作為必要條件的一部分。 
    1. 針對 [驗證類型]，選取您連線到資料庫時所要使用的**驗證類型**。 
    1. 針對 [使用者名稱]，輸入可存取 SQL Server 資料庫的使用者名稱。 如果您需要在使用者帳戶或伺服器名稱中使用斜線字元 (`\`)，請使用逸出字元 (`\`)。 例如 `mydomain\\myuser`。
    1. 針對 [密碼]，輸入使用者的**密碼**。 
    1. 若要測試 Data Factory 是否可連線到 SQL Server 資料庫，請按一下 [測試連線]。 修正任何錯誤，直到連線成功。 
    1. 若要儲存連結服務，按一下 [儲存]。

        ![SQL Server 連結服務 - 設定](./media/tutorial-incremental-copy-multiple-tables-portal/sql-server-linked-service-settings.png)

### <a name="create-the-azure-sql-database-linked-service"></a>建立 Azure SQL Database 連結服務。
在最後一個步驟中，您會建立連結服務，將來源 SQL Server 資料庫連結到資料處理站。 在此步驟中，您會將目的地/接收 Azure SQL 資料庫連結到資料處理站。 

1. 在 [連線] 視窗中，從 [整合執行階段] 索引標籤切換至 [連結服務] 索引標籤，然後按一下 [+ 新增]。

    ![新增連結服務按鈕](./media/tutorial-incremental-copy-multiple-tables-portal/new-sql-server-linked-service-button.png)
1. 在 [新增連結服務] 視窗中，選取 [Azure SQL Database]，然後按一下 [繼續]。 
1. 在 [新增連結服務] 視窗中，執行下列步驟：

    1. 輸入 **AzureSqlDatabaseLinkedService** 作為 [名稱]。 
    1. 針對 [伺服器名稱]，從下拉式清單中選取您 Azure SQL 伺服器的名稱。 
    1. 針對 [資料庫名稱]，選取您在其中建立 customer_table 和 project_table 作為必要條件一部分的 Azure SQL 資料庫。 
    1. 針對 [使用者名稱]，輸入可存取 Azure SQL 資料庫的使用者名稱。 
    1. 針對 [密碼]，輸入使用者的**密碼**。 
    1. 若要測試 Data Factory 是否可連線到 SQL Server 資料庫，請按一下 [測試連線]。 修正任何錯誤，直到連線成功。 
    1. 若要儲存連結服務，按一下 [儲存]。

        ![Azure SQL 連結服務 - 設定](./media/tutorial-incremental-copy-multiple-tables-portal/azure-sql-linked-service-settings.png)
1. 確認您在清單中看到兩個連結服務。 
   
    ![兩個連結服務](./media/tutorial-incremental-copy-multiple-tables-portal/two-linked-services.png) 

## <a name="create-datasets"></a>建立資料集
在此步驟中，您會建立資料集以代表資料來源和資料目的地，以及要儲存水位線的位置。

### <a name="create-a-source-dataset"></a>建立來源資料集

1. 按一下左窗格中的 [+\] (加號\)，然後按一下 [資料集]。

   ![新增資料集功能表](./media/tutorial-incremental-copy-multiple-tables-portal/new-dataset-menu.png)
1. 在 [新增資料集] 視窗中選取 [SQL Server]，然後按一下 [完成]。 

   ![選取 SQL Server](./media/tutorial-incremental-copy-multiple-tables-portal/select-sql-server-for-dataset.png)
1. 您會看到網頁瀏覽器中開啟了用來設定資料集的新索引標籤。 你也會在樹狀檢視中看到資料集。 在底部 [屬性] 視窗的 [一般] 索引標籤中，輸入 **SourceDataset** 作為 [名稱]。 

   ![來源資料集 - 名稱](./media/tutorial-incremental-copy-multiple-tables-portal/source-dataset-general.png)
1. 在 [屬性] 視窗中切換至 [連線] 索引標籤，然後針對 [連結服務] 選取 **SqlServerLinkedService**。 您在此處不會選取資料表。 管線中的複製活動會使用 SQL 查詢來載入資料，而不會載入整個資料表。

   ![來源資料集 - 連線](./media/tutorial-incremental-copy-multiple-tables-portal/source-dataset-connection.png)


### <a name="create-a-sink-dataset"></a>建立接收資料集
1. 按一下左窗格中的 [+\] (加號\)，然後按一下 [資料集]。

   ![新增資料集功能表](./media/tutorial-incremental-copy-multiple-tables-portal/new-dataset-menu.png)
1. 在 [新增資料集] 視窗中選取 [Azure SQL Database]，然後按一下 [完成]。 

   ![選取 Azure SQL Database](./media/tutorial-incremental-copy-multiple-tables-portal/select-azure-sql-database.png)
1. 您會看到網頁瀏覽器中開啟了用來設定資料集的新索引標籤。 你也會在樹狀檢視中看到資料集。 在底部 [屬性] 視窗的 [一般] 索引標籤中，輸入 **SinkDataset** 作為 [名稱]。

   ![接收資料集 - 一般](./media/tutorial-incremental-copy-multiple-tables-portal/sink-dataset-general.png)
1. 在 [屬性] 視窗中切換至 [參數] 索引標籤，並執行下列步驟： 

    1. 按一下 [建立/更新參數] 區段中的 [新增]。 
    1. 輸入 **SinkTableName** 作為**名稱**，並輸入 **String** 作為**類型**。 此資料集採用 **SinkTableName** 作為參數。 SinkTableName 參數是由管線在執行階段動態設定的。 管線中的 ForEach 活動會逐一查看資料表名稱清單，並將資料表名稱傳遞至每個反覆項目中的這個資料集。
   
       ![接收資料集 - 屬性](./media/tutorial-incremental-copy-multiple-tables-portal/sink-dataset-parameters.png)
1. 在 [屬性] 視窗中切換至 [連線] 索引標籤，然後針對 [連結服務] 選取 **AzureSqlLinkedService**。 針對 [資料表] 屬性，按一下 [新增動態內容]。 

   ![接收資料集 - 連線](./media/tutorial-incremental-copy-multiple-tables-portal/sink-dataset-connection.png)
    
    
1. 選取 [參數] 區段中的 [SinkTableName]
   
   ![接收資料集 - 連線](./media/tutorial-incremental-copy-multiple-tables-portal/sink-dataset-connection-dynamicContent.png)

   
 1. 按一下 [完成] 後，您會看到資料表名稱顯示為 **@dataset().SinkTableName**。
   
   ![接收資料集 - 連線](./media/tutorial-incremental-copy-multiple-tables-portal/sink-dataset-connection-completion.png)

### <a name="create-a-dataset-for-a-watermark"></a>建立水位線的資料集
在此步驟中，您會建立資料集來儲存高水位線值。 

1. 按一下左窗格中的 [+\] (加號\)，然後按一下 [資料集]。

   ![新增資料集功能表](./media/tutorial-incremental-copy-multiple-tables-portal/new-dataset-menu.png)
1. 在 [新增資料集] 視窗中選取 [Azure SQL Database]，然後按一下 [完成]。 

   ![選取 Azure SQL Database](./media/tutorial-incremental-copy-multiple-tables-portal/select-azure-sql-database.png)
1. 在底部 [屬性] 視窗的 [一般] 索引標籤中，輸入 **WatermarkDataset** 作為 [名稱]。
1. 切換至 [連線] 索引標籤，然後執行下列步驟： 

    1. 選取 [AzureSqlDatabaseLinkedService] 作為 [連結服務]。
    1. 選取 **[dbo].[watermarktable]** 作為 [資料表]。

       ![水位線資料集 - 連線](./media/tutorial-incremental-copy-multiple-tables-portal/watermark-dataset-connection.png)

## <a name="create-a-pipeline"></a>建立管線
管線會使用資料表名稱清單作為參數。 ForEach 活動會逐一查看資料表名稱清單，並執行下列作業： 

1. 使用查閱活動擷取舊的水位線值 (初始值，或最後一次反覆運算中使用的值)。

1. 使用查閱活動擷取新的水位線值 (來源資料表中水位線資料行的最大值)。

1. 使用複製活動，在來源資料庫到目的地資料庫的這兩個水位線值之間複製資料。

1. 使用 StoredProcedure 活動，更新要在下一個反覆項目的第一個步驟中使用的舊水位線值。 

### <a name="create-the-pipeline"></a>建立管線

1. 按一下左窗格中的 [+] (加號)，然後按一下 [管線]。

    ![新增管線 - 功能表](./media/tutorial-incremental-copy-multiple-tables-portal/new-pipeline-menu.png)
1. 在 [屬性] 視窗的 [一般] 索引標籤中，輸入 **IncrementalCopyPipeline** 作為 [名稱]。 

    ![管線名稱](./media/tutorial-incremental-copy-multiple-tables-portal/pipeline-name.png)
1. 在 [屬性] 視窗中，執行下列步驟： 

    1. 按一下 [+ 新增]。 
    1. 輸入 **tableList** 作為參數的 [名稱]。 
    1. 選取 [物件] 作為參數**類型**。

    ![管線參數](./media/tutorial-incremental-copy-multiple-tables-portal/pipeline-parameters.png) 
1. 在 [活動] 工具箱中展開 [反覆項目與條件]，並將 [ForEach] 活動拖放至管線設計工具介面。 在 [屬性] 視窗的 [一般] 索引標籤中，輸入 **IterateSQLTables**。 

    ![ForEach 活動 - 名稱](./media/tutorial-incremental-copy-multiple-tables-portal/foreach-name.png)
1. 在 [屬性] 視窗中切換至 [設定] 索引標籤，然後針對 [項目] 輸入 `@pipeline().parameters.tableList`。 ForEach 活動會逐一查看資料表清單，並執行累加式複製作業。 

    ![ForEach 活動 - 設定](./media/tutorial-incremental-copy-multiple-tables-portal/foreach-settings.png)
1. 在管線中選取 **ForEach** 活動 (如果尚未選取)。 按一下 [編輯 (鉛筆圖示)] 按鈕。

    ![ForEach 活動 - 編輯](./media/tutorial-incremental-copy-multiple-tables-portal/edit-foreach.png)
1. 在 [活動] 工具箱中展開 [一般]，並將 [查閱] 活動拖放至管線設計工具介面，然後輸入 **LookupOldWaterMarkActivity** 作為 [名稱]。

    ![第一個查閱活動 - 名稱](./media/tutorial-incremental-copy-multiple-tables-portal/first-lookup-name.png)
1. 切換 [屬性] 視窗中的 [設定] 索引標籤，並執行下列步驟： 

    1. 選取 **WatermarkDataset** 作為 [來源資料集]。
    1. 為 [使用查詢] 選取 [查詢]。 
    1. 輸入下列 SQL 查詢作為 [查詢]。 

        ```sql
        select * from watermarktable where TableName  =  '@{item().TABLE_NAME}'
        ```

        ![第一個查閱活動 - 設定](./media/tutorial-incremental-copy-multiple-tables-portal/first-lookup-settings.png)
1. 拖放 [活動] 工具箱中的 [查閱] 活動，並輸入 **LookupNewWaterMarkActivity** 作為 [名稱]。
        
    ![第二個查閱活動 - 名稱](./media/tutorial-incremental-copy-multiple-tables-portal/second-lookup-name.png)
1. 切換到 [設定]  索引標籤。

    1. 選取 [SourceDataset] 作為 [來源資料集]。 
    1. 為 [使用查詢] 選取 [查詢]。
    1. 輸入下列 SQL 查詢作為 [查詢]。

        ```sql    
        select MAX(@{item().WaterMark_Column}) as NewWatermarkvalue from @{item().TABLE_NAME}
        ```
    
        ![第二個查閱活動 - 設定](./media/tutorial-incremental-copy-multiple-tables-portal/second-lookup-settings.png)
1. 拖放 [活動] 工具箱中的 [複製] 活動，並輸入 **IncrementalCopyActivity** 作為 [名稱]。 

    ![複製活動 - 名稱](./media/tutorial-incremental-copy-multiple-tables-portal/copy-activity-name.png)
1. 逐一將 [查閱] 活動連線至 [複製] 活動。 若要連線，請先將連結至 [查閱] 活動的**綠色**方塊拖放到 [複製] 活動上。 當 [複製] 活動的框線顏色變為**藍色**時，即鬆開滑鼠按鈕。

    ![將 [查閱] 活動連線至 [複製] 活動](./media/tutorial-incremental-copy-multiple-tables-portal/connect-lookup-to-copy.png)
1. 選取管線中的 [複製] 活動。 在 [屬性] 視窗中切換至 [來源] 索引標籤。 

    1. 選取 [SourceDataset] 作為 [來源資料集]。 
    1. 為 [使用查詢] 選取 [查詢]。 
    1. 輸入下列 SQL 查詢作為 [查詢]。

        ```sql
        select * from @{item().TABLE_NAME} where @{item().WaterMark_Column} > '@{activity('LookupOldWaterMarkActivity').output.firstRow.WatermarkValue}' and @{item().WaterMark_Column} <= '@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}'        
        ```

        ![複製活動 - 來源設定](./media/tutorial-incremental-copy-multiple-tables-portal/copy-source-settings.png)
1. 切換至 [接收] 索引標籤，然後選取 **SinkDataset** 作為 [接收資料集]。 
        
    ![複製活動 - 接收設定](./media/tutorial-incremental-copy-multiple-tables-portal/copy-sink-settings.png)
1. 切換至 [參數] 索引標籤，並執行下列步驟：

    1. 針對 [接收預存程序名稱] 屬性，輸入 `@{item().StoredProcedureNameForMergeOperation}`。
    1. 針對 [接收資料表類型] 屬性，輸入 `@{item().TableType}`。
    1. 在 [接收資料集] 區段中，針對 **SinkTableName** 參數輸入 `@{item().TABLE_NAME}`。

        ![複製活動 - 參數](./media/tutorial-incremental-copy-multiple-tables-portal/copy-activity-parameters.png)
1. 將 [活動] 工具箱中的 [預存程序] 活動拖放至管線設計工具介面。 將 [複製] 活動連線至 [預存程序] 活動。 

    ![複製活動 - 參數](./media/tutorial-incremental-copy-multiple-tables-portal/connect-copy-to-sproc.png)
1. 在管線中選取 [預存程序] 活動，然後在 [屬性] 視窗的 [一般] 索引標籤中輸入 **StoredProceduretoWriteWatermarkActivity** 作為 [名稱]。 

    ![預存程序活動 - 名稱](./media/tutorial-incremental-copy-multiple-tables-portal/sproc-activity-name.png)
1. 切換至 [SQL 帳戶] 索引標籤，然後選取 **AzureSqlDatabaseLinkedService** 作為 [連結服務]。

    ![預存程序活動 - SQL 帳戶](./media/tutorial-incremental-copy-multiple-tables-portal/sproc-activity-sql-account.png)
1. 切換至 [預存程序] 索引標籤，然後執行下列步驟：

    1. 針對 [預存程序名稱]，選取 `sp_write_watermark`。 
    1. 選取 [匯入參數]。 
    1. 指定參數的下列值︰ 

        | 名稱 | 類型 | 值 | 
        | ---- | ---- | ----- |
        | LastModifiedtime | Datetime | `@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}` |
        | TableName | 字串 | `@{activity('LookupOldWaterMarkActivity').output.firstRow.TableName}` |
    
        ![預存程序活動 - 預存程序設定](./media/tutorial-incremental-copy-multiple-tables-portal/sproc-activity-sproc-settings.png)
1. 在左窗格中按一下 [發佈]。 此動作會將您建立的實體發佈至 Data Factory 服務。 

    ![發佈按鈕](./media/tutorial-incremental-copy-multiple-tables-portal/publish-button.png)
1. 請靜待 [發佈成功] 訊息顯示。 若要檢視通知，請按一下 [顯示通知] 連結。 按一下 **X** 以關閉通知視窗。

    ![顯示通知](./media/tutorial-incremental-copy-multiple-tables-portal/notifications.png)

 
## <a name="run-the-pipeline"></a>執行管道

1. 在管線的工具列上按一下 [觸發]，然後按一下 [立即觸發]。     

    ![立即觸發](./media/tutorial-incremental-copy-multiple-tables-portal/trigger-now.png)
1. 在 [管線執行] 視窗中，為 **tableList** 參數輸入下列值，然後按一下 [完成]。 

    ```
    [
        {
            "TABLE_NAME": "customer_table",
            "WaterMark_Column": "LastModifytime",
            "TableType": "DataTypeforCustomerTable",
            "StoredProcedureNameForMergeOperation": "sp_upsert_customer_table"
        },
        {
            "TABLE_NAME": "project_table",
            "WaterMark_Column": "Creationtime",
            "TableType": "DataTypeforProjectTable",
            "StoredProcedureNameForMergeOperation": "sp_upsert_project_table"
        }
    ]
    ```

    ![管線執行引數](./media/tutorial-incremental-copy-multiple-tables-portal/pipeline-run-arguments.png)

## <a name="monitor-the-pipeline"></a>監視管線

1. 切換至左側的 [監視] 索引標籤。 您會看到**手動觸發程序**所觸發的管線執行。 按一下 [重新整理] 按鈕即可重新整理清單。 [動作] 資料行中的連結可讓您檢視與此管線執行相關聯的活動執行，以及重新執行管線。 

    ![管線執行](./media/tutorial-incremental-copy-multiple-tables-portal/pipeline-runs.png)
1. 按一下 [動作] 資料行中的 [檢視活動執行] 連結。 您會看到所有與選取的管線執行相關聯的活動執行。 

    ![活動執行](./media/tutorial-incremental-copy-multiple-tables-portal/activity-runs.png)

## <a name="review-the-results"></a>檢閱結果
在 SQL Server Management Studio 中，對目標 SQL 資料庫執行下列查詢，以確認資料已從來源資料表複製到目的地資料表： 

**查詢** 
```sql
select * from customer_table
```

**輸出**
```
===========================================
PersonID    Name    LastModifytime
===========================================
1           John    2017-09-01 00:56:00.000
2           Mike    2017-09-02 05:23:00.000
3           Alice   2017-09-03 02:36:00.000
4           Andy    2017-09-04 03:21:00.000
5           Anny    2017-09-05 08:06:00.000
```

**查詢**

```sql
select * from project_table
```

**輸出**

```
===================================
Project     Creationtime
===================================
project1    2015-01-01 00:00:00.000
project2    2016-02-02 01:23:00.000
project3    2017-03-04 05:16:00.000
```

**查詢**

```sql
select * from watermarktable
```

**輸出**

```
======================================
TableName       WatermarkValue
======================================
customer_table  2017-09-05 08:06:00.000
project_table   2017-03-04 05:16:00.000
```

請注意，這兩個資料表的水位線值已更新。 

## <a name="add-more-data-to-the-source-tables"></a>新增更多資料至來源資料表

對來源 SQL Server 資料庫執行下列查詢，以更新 customer_table 中的現有資料列。 在 project_table 中插入新的資料列。 

```sql
UPDATE customer_table
SET [LastModifytime] = '2017-09-08T00:00:00Z', [name]='NewName' where [PersonID] = 3

INSERT INTO project_table
(Project, Creationtime)
VALUES
('NewProject','10/1/2017 0:00:00 AM');
``` 

## <a name="rerun-the-pipeline"></a>重新執行管線
1. 在網頁瀏覽器視窗中，切換至左側的 [編輯] 索引標籤。 
1. 在管線的工具列上按一下 [觸發]，然後按一下 [立即觸發]。   

    ![立即觸發](./media/tutorial-incremental-copy-multiple-tables-portal/trigger-now.png)
1. 在 [管線執行] 視窗中，為 **tableList** 參數輸入下列值，然後按一下 [完成]。 

    ```
    [
        {
            "TABLE_NAME": "customer_table",
            "WaterMark_Column": "LastModifytime",
            "TableType": "DataTypeforCustomerTable",
            "StoredProcedureNameForMergeOperation": "sp_upsert_customer_table"
        },
        {
            "TABLE_NAME": "project_table",
            "WaterMark_Column": "Creationtime",
            "TableType": "DataTypeforProjectTable",
            "StoredProcedureNameForMergeOperation": "sp_upsert_project_table"
        }
    ]
    ```

## <a name="monitor-the-pipeline-again"></a>重新監視管線

1. 切換至左側的 [監視] 索引標籤。 您會看到**手動觸發程序**所觸發的管線執行。 按一下 [重新整理] 按鈕即可重新整理清單。 [動作] 資料行中的連結可讓您檢視與此管線執行相關聯的活動執行，以及重新執行管線。 

    ![管線執行](./media/tutorial-incremental-copy-multiple-tables-portal/pipeline-runs.png)
1. 按一下 [動作] 資料行中的 [檢視活動執行] 連結。 您會看到所有與選取的管線執行相關聯的活動執行。 

    ![活動執行](./media/tutorial-incremental-copy-multiple-tables-portal/activity-runs.png) 

## <a name="review-the-final-results"></a>檢閱最終結果
在 SQL Server Management Studio 中，對目標資料庫執行下列查詢，以確認經過更新/全新的資料已從來源資料表複製到目的地資料表。 

**查詢** 
```sql
select * from customer_table
```

**輸出**
```
===========================================
PersonID    Name    LastModifytime
===========================================
1           John    2017-09-01 00:56:00.000
2           Mike    2017-09-02 05:23:00.000
3           NewName 2017-09-08 00:00:00.000
4           Andy    2017-09-04 03:21:00.000
5           Anny    2017-09-05 08:06:00.000
```

請注意，**PersonID** 為 3 的 **Name** 和 **LastModifytime** 的新值。 

**查詢**

```sql
select * from project_table
```

**輸出**

```
===================================
Project     Creationtime
===================================
project1    2015-01-01 00:00:00.000
project2    2016-02-02 01:23:00.000
project3    2017-03-04 05:16:00.000
NewProject  2017-10-01 00:00:00.000
```

請注意，project_table 中已新增 **NewProject** 項目。 

**查詢**

```sql
select * from watermarktable
```

**輸出**

```
======================================
TableName       WatermarkValue
======================================
customer_table  2017-09-08 00:00:00.000
project_table   2017-10-01 00:00:00.000
```

請注意，這兩個資料表的水位線值已更新。
     
## <a name="next-steps"></a>後續步驟
在本教學課程中，您已執行下列步驟： 

> [!div class="checklist"]
> * 準備來源和目的地資料存放區。
> * 建立資料處理站。
> * 建立自我裝載的整合執行階段 (IR)。
> * 安裝整合執行階段。
> * 建立連結的服務。 
> * 建立來源、接收及水位線資料集。
> * 建立、執行及監視管線。
> * 檢閱結果。
> * 新增或更新來源資料表中的資料。
> * 重新執行，並監視管線。
> * 檢閱最終結果。

進入下列教學課程，以了解如何在 Azure 上使用 Spark 叢集來轉換資料：

> [!div class="nextstepaction"]
>[使用變更追蹤技術，以累加方式將資料從 Azure SQL Database 載入到 Azure Blob 儲存體](tutorial-incremental-copy-change-tracking-feature-portal.md)


