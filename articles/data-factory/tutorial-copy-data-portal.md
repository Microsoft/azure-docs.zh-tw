---
title: 使用 Azure 入口網站建立資料處理站管線 | Microsoft Docs
description: 本教學課程提供逐步指示，說明如何使用 Azure 入口網站建立具有管線的資料處理站。 管線會使用複製活動將資料從 Azure Blob 儲存體複製到 SQL 資料庫。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 06/21/2018
ms.author: jingwang
ms.openlocfilehash: f7d6f34c75069f91e06d58c960249d040b2bda8a
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44299184"
---
# <a name="copy-data-from-azure-blob-storage-to-a-sql-database-by-using-azure-data-factory"></a>使用 Azure Data Factory 將資料從 Azure Blob 儲存體複製到 SQL 資料庫
在本教學課程中，您會使用 Azure Data Factory 使用者介面 (UI) 建立資料處理站。 此資料處理站中的管線會將資料從 Azure Blob 儲存體複製到 SQL 資料庫。 本教學課程中的設定模式從以檔案為基礎的資料存放區複製到關聯式資料存放區。 如需支援作為來源和接收的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)表格。

> [!NOTE]
> - 如果您不熟悉 Data Factory，請參閱 [Data Factory 簡介](introduction.md)。

在本教學課程中，您會執行下列步驟：

> [!div class="checklist"]
> * 建立資料處理站。
> * 建立具有複製活動的管線。
> * 對管線執行測試。
> * 手動觸發管線。
> * 觸發排程的管線。
> * 監視管線和活動執行。

## <a name="prerequisites"></a>必要條件
* **Azure 訂用帳戶**。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。
* **Azure 儲存體帳戶**。 您會使用 Blob 儲存體作為*來源*資料存放區。 如果您沒有儲存體帳戶，請參閱[建立 Azure 儲存體帳戶](../storage/common/storage-quickstart-create-account.md)，按照步驟建立此帳戶。
* **Azure SQL Database**。 您會使用資料庫作為*接收*資料存放區。 如果您沒有 SQL 資料庫，請參閱[建立 SQL 資料庫](../sql-database/sql-database-get-started-portal.md)，按照步驟建立此資料庫。

### <a name="create-a-blob-and-a-sql-table"></a>建立 Blob 和 SQL 資料表

現在，請執行下列步驟，準備本教學課程中的 Blob 儲存體和 SQL 資料庫。

#### <a name="create-a-source-blob"></a>建立來源 Blob

1. 啟動 [記事本]。 複製下列文字，並在磁碟上將其儲存為 **emp.txt** 檔案：

    ```
    John,Doe
    Jane,Doe
    ```

1. 在 Blob 儲存體中，建立名為 **adftutorial** 的容器。 在此容器中建立名為 **input** 的資料夾。 然後，將 **emp.txt** 檔案上傳至 **input** 資料夾。 請使用 Azure 入口網站或 [Azure 儲存體總管](http://storageexplorer.com/)之類的工具執行這些工作。

#### <a name="create-a-sink-sql-table"></a>建立接收 SQL 資料表

1. 使用下列 SQL 指令碼，在您的 SQL 資料庫中建立 **dbo.emp** 資料表：

    ```sql
    CREATE TABLE dbo.emp
    (
        ID int IDENTITY(1,1) NOT NULL,
        FirstName varchar(50),
        LastName varchar(50)
    )
    GO

    CREATE CLUSTERED INDEX IX_emp_ID ON dbo.emp (ID);
    ```

1. 允許 Azure 服務存取 SQL Server。 確定 SQL Server 的 [允許存取 Azure 服務] 已**開啟**，讓 Data Factory 能夠將資料寫入至您的 SQL Server。 若要確認並開啟此設定，請採取下列步驟：

    a. 從左側選取 [更多服務] > [SQL Server]。

    b. 選取您的伺服器，然後在 [設定] 下選取 [防火牆]。

    c. 在 [防火牆設定] 頁面上，針對 [允許存取 Azure 服務] 選取 [開啟]。

## <a name="create-a-data-factory"></a>建立 Data Factory
在此步驟中，您可以建立資料處理站，並啟動 Data Factory 使用者介面，在資料處理站中建立管線。 

1. 開啟 **Microsoft Edge** 或 **Google Chrome** 網頁瀏覽器。 目前，只有 Microsoft Edge 和 Google Chrome 網頁瀏覽器支援 Data Factory UI。
1. 在左側功能表中，選取 [新增] > [資料 + 分析] > [資料處理站]。 
  
   ![新資料處理站的建立](./media/tutorial-copy-data-portal/new-azure-data-factory-menu.png)
1. 在 [新增資料處理站] 頁面的 [名稱] 下，輸入 **ADFTutorialDataFactory**。 
      
     ![新增 Data Factory](./media/tutorial-copy-data-portal/new-azure-data-factory.png)
 
   Azure Data Factory 的名稱必須是 *全域唯一的*。 如果您在 [名稱] 欄位看到下列錯誤訊息，請變更資料處理站的名稱 (例如 yournameADFTutorialDataFactory)。 如需 Data Factory 成品的命名規則，請參閱 [Data Factory 命名規則](naming-rules.md)。
  
   ![錯誤訊息](./media/tutorial-copy-data-portal/name-not-available-error.png)
1. 選取您要在其中建立資料處理站的 Azure **訂用帳戶**。 
1. 針對 [資源群組]，採取下列其中一個步驟︰
     
    a. 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。

    b. 選取 [建立新的] ，然後輸入資源群組的名稱。 
         
    若要了解資源群組，請參閱[使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-overview.md)。 
1. 在 [版本] 下，選取 [V2]。
1. 在 [位置] 下，選取資料處理站的位置。 只有受到支援的位置會顯示在下拉式清單中。 資料處理站所使用的資料存放區 (例如 Azure 儲存體和 SQL Database) 和計算 (例如 Azure HDInsight) 可位於其他區域。
1. 選取 [釘選到儀表板]。 
1. 選取 [建立] 。 
1. 在儀表板上，您會看到 [正在部署 Data Factory] 狀態的下列圖格︰ 

    ![部署資料處理站圖格](media/tutorial-copy-data-portal/deploying-data-factory.png)
1. 建立完成之後，您會看到如圖中所示的 [Data Factory] 頁面。
   
    ![Data Factory 首頁](./media/tutorial-copy-data-portal/data-factory-home-page.png)
1. 選取 [編寫與監視]，以在個別索引標籤中啟動 Data Factory 使用者介面。

## <a name="create-a-pipeline"></a>建立管線
在此步驟中，您會在資料處理站中建立具有「複製」活動的管線。 複製活動會將資料從 Blob 儲存體複製到 SQL Database。 在[快速入門教學課程](quickstart-create-data-factory-portal.md)中，您已依照下列步驟建立管線：

1. 建立連結服務。 
1. 建立輸入和輸出資料集。
1. 建立管道。

在本教學課程中，您首先會建立管線。 然後，您會在需要連結服務和資料集以設定管線時，建立這些項目。 

1. 在 [現在就開始吧] 頁面中，選取 [建立管線]。 

   ![建立管線](./media/tutorial-copy-data-portal/create-pipeline-tile.png)
1. 在管線的 [一般] 索引標籤中，輸入 **CopyPipeline** 作為管線的[名稱]。

1. 在 [活動] 工具箱中展開 [資料流程] 類別，並將 [複製] 活動從工具箱拖放至管線設計工具介面。 指定 **CopyFromBlobToSql** 作為 [名稱]。

    ![複製活動](./media/tutorial-copy-data-portal/drag-drop-copy-activity.png)

### <a name="configure-source"></a>設定來源

1. 移至 [來源] 索引標籤。選取 [+ 新增] 以建立來源資料集。 

1. 在 [新增資料集] 視窗中選取 [Azure Blob 儲存體]，然後選取 [完成]。 來源資料位於 Blob 儲存體中，因此您選取 [Azure Blob 儲存體] 作為來源資料集。 

    ![儲存體選擇](./media/tutorial-copy-data-portal/select-azure-blob-dataset.png)

1. 您會看到系統為 Blob 資料集開啟了新的索引標籤。 在 [屬性] 視窗底部的 [一般] 索引標籤上，輸入 **SourceBlobDataset** 作為 [名稱]。

    ![資料集名稱](./media/tutorial-copy-data-portal/dataset-name.png)

1. 移至 [屬性] 視窗的 [連線] 索引標籤。 在 [連結服務] 文字方塊旁，選取 [+ 新增]。 

    ![新增連結服務按鈕](./media/tutorial-copy-data-portal/source-dataset-new-linked-service-button.png)

1. 在 [新增連結服務] 視窗中輸入 **AzureStorageLinkedService** 作為名稱，從 [儲存體帳戶名稱] 清單中選取您的儲存體帳戶，然後選取 [儲存] 以部署連結服務。

    ![新增連結服務](./media/tutorial-copy-data-portal/new-azure-storage-linked-service.png)

1. 在連結服務建立後，您會回到資料集設定。 在 [檔案路徑] 旁，選取 [瀏覽]。

    ![檔案路徑的瀏覽按鈕](./media/tutorial-copy-data-portal/file-browse-button.png)

1. 瀏覽至 **adftutorial/input** 資料夾，選取 **emp.txt** 檔案，然後選取 [完成]。 

    ![選取輸入檔案](./media/tutorial-copy-data-portal/select-input-file.png)

1. 確認 [檔案格式] 設為 [文字格式]，且 [資料行分隔符號] 設為 [逗號 (`,`)]。 如果來源檔案使用不同的資料列和資料行分隔符號，您可以針對 [檔案格式] 選取 [偵測文字格式]。 「複製資料」工具會自動為您偵測檔案格式和分隔符號。 您仍可以覆寫這些值。 若要預覽此頁面上的資料，請選取 [預覽資料]。

    ![偵測文字格式](./media/tutorial-copy-data-portal/detect-text-format.png)

1. 移至 [屬性] 視窗的 [結構描述] 索引標籤，然後選取 [匯入結構描述]。 請留意應用程式偵測到來源檔案中的兩個資料行。 您會在此處匯入結構描述，以便將資料行從來源資料存放區對應至接收資料存放區。 如果您不需要對應資料行，可以略過此步驟。 在本教學課程中，請匯入結構描述。

    ![偵測來源結構描述](./media/tutorial-copy-data-portal/detect-source-schema.png)  

1. 現在，回到管線 -> [來源] 索引標籤，確認已選取 [SourceBlobDataset]。 若要預覽此頁面上的資料，請選取 [預覽資料]。 
    
    ![來源資料集](./media/tutorial-copy-data-portal/source-dataset-selected.png)

### <a name="configure-sink"></a>設定接收

1. 移至 [接收] 索引標籤，然後選取 [+ 新增] 以建立接收資料集。 

    ![接收資料集](./media/tutorial-copy-data-portal/new-sink-dataset-button.png)
1. 在 [新增資料集] 視窗中，於搜尋方塊中輸入 "SQL" 以篩選連接器，並選取 [Azure SQL Database]，然後選取 [完成]。 在本教學課程中，您會將資料複製到 SQL 資料庫。 

    ![SQL 資料庫選取項目](./media/tutorial-copy-data-portal/select-azure-sql-dataset.png)
1. 在 [屬性] 視窗的 [一般] 索引標籤中，於 [名稱] 中輸入 **OutputSqlDataset**。 
    
    ![輸出資料集名稱](./media/tutorial-copy-data-portal/output-dataset-name.png)
1. 移至 [連線] 索引標籤，然後在 [連結服務] 旁選取 [+ 新增]。 資料集必須與連結的服務相關聯。 連結服務具有連接字串，可供 Data Factory 在執行階段中用來連線到 SQL 資料庫。 資料集會指定要作為資料複製目的地的容器、資料夾和檔案 (選擇性)。 
    
    ![連結服務](./media/tutorial-copy-data-portal/new-azure-sql-database-linked-service-button.png)       
1. 在 [新增連結服務] 視窗中，採取下列步驟： 

    a. 在 [名稱] 下，輸入 **AzureSqlDatabaseLinkedService**。

    b. 在 [伺服器名稱] 下，選取您的 SQL Server 執行個體。

    c. 在 [資料庫名稱] 下，選取您的 SQL 資料庫。

    d. 在 [使用者名稱] 下，輸入使用者名稱。

    e. 在 [密碼] 下，輸入使用者的密碼。

    f. 選取 [測試連線] 以測試連線。

    g. 選取 [儲存] 以儲存連結服務。 
    
    ![儲存新的連結服務](./media/tutorial-copy-data-portal/new-azure-sql-linked-service-window.png)

1. 在 [資料表] 中，選取 **[dbo].[emp]**。 

    ![資料表](./media/tutorial-copy-data-portal/select-emp-table.png)
1. 移至 [結構描述] 索引標籤，然後選取 [匯入結構描述]。 

    ![選取匯入結構描述](./media/tutorial-copy-data-portal/import-destination-schema.png)
1. 選取 [識別碼] 資料行，然後選取 [刪除]。 [識別碼] 資料行是 SQL 資料庫中的身分識別資料行，因此複製活動不需要將資料插入此資料行中。

    ![刪除識別碼資料行](./media/tutorial-copy-data-portal/delete-id-column.png)
1. 移至含有管線的索引標籤，然後在 [接收資料集] 中確認已選取 **OutputSqlDataset**。

    ![管線索引標籤](./media/tutorial-copy-data-portal/pipeline-tab-2.png)        

### <a name="confugure-mapping"></a>設定對應

移至 [屬性] 視窗底部的 [對應] 索引標籤，並選取 [匯入結構描述]。 請注意，來源檔案中的第一個和第二個資料行會對應至 SQL 資料庫中的 **FirstName** 和 **LastName**。

![對應結構描述](./media/tutorial-copy-data-portal/map-schemas.png)

## <a name="validate-the-pipeline"></a>驗證管線
若要驗證管線，請從工具列中選取 [驗證]。
 
您可以按一下右上方的 [程式碼]，以檢視與管線相關聯的 JSON 程式碼。

## <a name="debug-and-publish-the-pipeline"></a>偵錯和發佈管線
您可以先對管線執行偵錯，再將成品 (連結服務、資料集和管線) 發佈至 Data Factory 或您自己的 Azure Repos Git 存放庫。 

1. 若要對管線進行偵錯，請選取工具列上的 [偵錯]。 您可以在視窗底部的 [輸出] 索引標籤中檢視管線執行的狀態。 

1. 當管線可成功執行後，請在頂端的工具列中選取 [全部發佈]。 此動作會將您已建立的實體 (資料集和管線) 發佈至 Data Factory。

    ![發佈](./media/tutorial-copy-data-portal/publish-button.png)

1. 請靜待 [發佈成功] 訊息顯示。 若要檢視通知訊息，請按一下右上方的 [顯示通知] (鈴鐺按鈕)。 

## <a name="trigger-the-pipeline-manually"></a>手動觸發管線
在此步驟中，您會手動觸發您在上一個步驟中發佈的管線。 

1. 選取工具列上的 [觸發程序]，然後選取 [立即觸發]。 在 [管線執行] 頁面上，選取 [完成]。  

1. 移至左側的 [監視] 索引標籤。 您會看到手動觸發程序所觸發的管線執行。 您可以使用 [動作] 資料行中的連結來檢視活動詳細資料，以及重新執行管線。

    ![監視管線回合](./media/tutorial-copy-data-portal/monitor-pipeline.png)
1. 若要檢視與此管線執行相關聯的活動執行，請選取 [動作] 資料行中的 [檢視活動執行] 連結。 此範例中只有一個活動，因此您在清單中只會看到一個項目。 如需關於複製作業的詳細資料，請選取 [動作] 資料行中的 [詳細資料] 連結 (眼鏡圖示)。 選取頂端的 [管線] 可回到 [管線執行] 檢視。 若要重新整理檢視，請選取 [重新整理]。

    ![監視活動回合](./media/tutorial-copy-data-portal/view-activity-runs.png)
1. 確認有兩個以上的資料列新增至 SQL 資料庫中的 **emp** 資料表。 

## <a name="trigger-the-pipeline-on-a-schedule"></a>觸發排程上的管線
在此排程中，您會建立管線的排程器觸發程序。 此觸發程序會依照指定的排程 (例如每小時或每天) 執行管線。 在此範例中，您會建立每分鐘執行一次的觸發程序，並讓其在指定的結束日期時間停止。 

1. 移至左側位於監視索引標籤上方的 [作者] 索引標籤。 

1. 移至您的管線，按一下工具列上的 [觸發程序]，然後選取 [新增/編輯]。 

1. 在 [新增觸發程序] 視窗中選取 [選擇觸發程序]，然後選取 [+ 新增]。 

    ![New button](./media/tutorial-copy-data-portal/add-trigger-new-button.png)
1. 在 [新增觸發程序] 視窗中，採取下列步驟： 

    a. 在 [名稱] 下，輸入 **RunEveryMinute**。

    b. 在 [結束] 下，選取 [日期]。

    c. 在 [結束日期] 下，選取下拉式清單。

    d. 選取 [當天日期] 選項。 根據預設，結束日期會設為次日。

    e. 將**分鐘**部分更新為當天日期時間的數分鐘之後。 在您發佈變更之後，才會啟動觸發程序。 如果您將其設為短短幾分鐘之後，且您未在那之前發佈變更，則不會看到觸發程序執行。

    f. 選取 [套用] 。 

    ![觸發程序屬性](./media/tutorial-copy-data-portal/set-trigger-properties.png)

    g. 選取 [已啟動] 選項。 您可以先將其停用，其後再使用此核取方塊加以啟動。

    h. 選取 [下一步] 。

    ![已啟動按鈕](./media/tutorial-copy-data-portal/trigger-activiated-next.png)

    > [!IMPORTANT]
    > 每次執行管線都會產生相關成本，因此請適當設定結束日期。 
1. 在 [觸發程序執行參數] 頁面上檢閱警告，然後選取 [完成]。 此範例中的管線未使用任何參數。 

    ![觸發程序執行參數](./media/tutorial-copy-data-portal/trigger-pipeline-parameters.png)

1. 按一下 [全部發佈] 以發佈變更。 

1. 移至左側的 [監視] 索引標籤，以檢視已觸發的管線執行。 

    ![已觸發的管線執行](./media/tutorial-copy-data-portal/triggered-pipeline-runs.png)    
1. 若要從 [管線執行] 檢視切換至 [觸發程序執行] 檢視，請選取 [管線執行]，然後選取 [觸發程序執行]。
    
    ![觸發程序執行](./media/tutorial-copy-data-portal/trigger-runs-menu.png)
1. 您可以在清單中檢視觸發程序執行。 

    ![觸發程序執行清單](./media/tutorial-copy-data-portal/trigger-runs-list.png)
1. 確認在指定的結束時間之前，每個管線執行每分鐘會在 **emp** 資料表中插入兩個資料列。 

## <a name="next-steps"></a>後續步驟
此範例中的管線會將資料從 Blob 儲存體中的一個位置複製到其他位置。 您已了解如何︰ 

> [!div class="checklist"]
> * 建立資料處理站。
> * 建立具有複製活動的管線。
> * 對管線執行測試。
> * 手動觸發管線。
> * 觸發排程的管線。
> * 監視管線和活動執行。


若要了解如何將資料從內部部署複製到雲端，請進入下列教學課程： 

> [!div class="nextstepaction"]
>[將資料從內部部署複製到雲端](tutorial-hybrid-copy-portal.md)
