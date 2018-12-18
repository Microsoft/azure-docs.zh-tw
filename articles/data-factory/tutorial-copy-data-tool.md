---
title: 使用 Azure 複製資料工具複製資料 | Microsoft Docs
description: 建立 Azure 資料處理站，然後使用「複製資料」工具將資料從 Azure Blob 儲存體複製到 SQL 資料庫。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 09/11/2018
ms.author: jingwang
ms.openlocfilehash: c1f94c04db69ad44203ef1ada1c3b9fa3df2d779
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44718079"
---
# <a name="copy-data-from-azure-blob-storage-to-a-sql-database-by-using-the-copy-data-tool"></a>使用複製資料工具將資料從 Azure Blob 儲存體複製到 SQL 資料庫
> [!div class="op_single_selector" title1="Select the version of the Data Factory service that you're using:"]
> * [第 1 版](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [目前的版本](tutorial-copy-data-tool.md)

在這個教學課程中，您會使用 Azure 入口網站來建立資料處理站。 接著，您會使用複製資料工具來建立管線，以將資料從 Azure Blob 儲存體複製到 SQL 資料庫。 

> [!NOTE]
> 如果您不熟悉 Azure Data Factory，請參閱 [Azure Data Factory 簡介](introduction.md)。

在此教學課程中，您會執行下列步驟：

> [!div class="checklist"]
> * 建立資料處理站。
> * 使用複製資料工具建立管線。
> * 監視管線和活動執行。

## <a name="prerequisites"></a>先決條件

* **Azure 訂用帳戶**：如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。
* **Azure 儲存體帳戶**：使用 Blob 儲存體作為「來源」資料存放區。 如果您沒有 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)中的指示。
* **Azure SQL Database**：使用 Azure SQL 資料庫作為「接收」資料存放區。 如果您沒有 SQL 資料庫，請參閱[建立 SQL 資料庫](../sql-database/sql-database-get-started-portal.md)中的指示。

### <a name="create-a-blob-and-a-sql-table"></a>建立 Blob 和 SQL 資料表

請執行下列步驟，為此教學課程準備 Blob 儲存體和您的 SQL 資料庫。

#### <a name="create-a-source-blob"></a>建立來源 Blob

1. 啟動 [記事本]。 複製下列文字，並將其儲存在磁碟上名為 **inputEmp.txt** 的檔案：

    ```
    John|Doe
    Jane|Doe
    ```

1. 建立名為 **adfv2tutorial** 的容器，並將 inputEmp.txt 檔案上傳至該容器。 您可以使用各種工具來執行這些工作，例如 [Azure 儲存體總管](http://storageexplorer.com/)。

#### <a name="create-a-sink-sql-table"></a>建立接收 SQL 資料表

1. 使用以下 SQL 指令碼，在您的 SQL 資料庫中建立名為 **dbo.emp** 的資料表：

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

2. 允許 Azure 服務存取 SQL Server。 確認已為執行 SQL Database 的伺服器啟用 [允許存取 Azure 服務] 設定。 此設定可讓 Data Factory 將資料寫入您的資料庫執行個體。 若要確認並開啟此設定，請前往您的 Azure SQL Server > [安全性] > [防火牆與虛擬網路] > 將 [允許存取 Azure 服務] 選項設定為 [開啟]。

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 在左側功能表中，選取 [+ 新增] > [資料 + 分析] > [資料處理站]： 
   
   ![新資料處理站的建立](./media/tutorial-copy-data-tool/new-azure-data-factory-menu.png)
1. 在 [新增資料處理站] 頁面的 [名稱] 下，輸入 **ADFTutorialDataFactory**。 
      
     ![新增 Data Factory](./media/tutorial-copy-data-tool/new-azure-data-factory.png)
 
   資料處理站的名稱必須是「全域唯一」的名稱。 您可能會收到下列錯誤訊息：
   
   ![新增資料處理站錯誤訊息](./media/tutorial-copy-data-tool/name-not-available-error.png)

   如果您收到有關名稱值的錯誤訊息，請輸入不同的資料處理站名稱。 例如，使用**您的名稱****ADFTutorialDataFactory**。 如需 Data Factory 成品的命名規則，請參閱 [Data Factory 命名規則](naming-rules.md)。
1. 選取要在其中建立新資料處理站的 Azure **訂用帳戶**。 
1. 針對 [資源群組]，採取下列其中一個步驟︰
     
    a. 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。

    b. 選取 [建立新的] ，然後輸入資源群組的名稱。 
         
    若要了解資源群組，請參閱[使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-overview.md)。

1. 在 [版本] 下，選取 [V2] 作為版本。
1. 在 [位置] 下，選取資料處理站的位置。 只有受到支援的位置會顯示在下拉式清單中。 資料處理站所使用的資料存放區 (例如 Azure 儲存體和 SQL Database) 和計算 (例如 Azure HDInsight) 可位於其他地區和區域。
1. 選取 [釘選到儀表板]。 
1. 選取 [建立] 。
1. 儀表板上的 [部署資料處理站] 圖格會顯示程序狀態。

    ![部署資料處理站圖格](media/tutorial-copy-data-tool/deploying-data-factory.png)
1. 建立完成後，隨即會顯示 **Data Factory** 首頁。
   
    ![Data Factory 首頁](./media/tutorial-copy-data-tool/data-factory-home-page.png)
1. 選取 [編寫與監視] 圖格，可在另一個索引標籤中啟動 Azure Data Factory 使用者介面 (UI)。 

## <a name="use-the-copy-data-tool-to-create-a-pipeline"></a>使用複製資料工具建立管線

1. 在 [現在就開始吧] 頁面上選取 [複製資料] 圖格，以啟動複製資料工具。 

   ![複製資料工具圖格](./media/tutorial-copy-data-tool/copy-data-tool-tile.png)
1. 在 [屬性] 頁面的 [工作名稱] 下，輸入 **CopyFromBlobToSqlPipeline**。 然後，選取 [下一步]。 Data Factory 使用者介面會使用指定的工作名稱建立管線。 

    ![屬性頁面](./media/tutorial-copy-data-tool/copy-data-tool-properties-page.png)
1. 在 [來源資料存放區]  頁面上，完成下列步驟：

    a. 按一下 [+ 建立新連線] 以新增連線

    ![新增來源連結服務](./media/tutorial-copy-data-tool/new-source-linked-service.png)

    b. 從資源庫選取 [Azure Blob 儲存體]，然後選取 [下一步]。

    ![選取 Blob 來源](./media/tutorial-copy-data-tool/select-blob-source.png)

    c. 在 [新增連結服務] 頁面上，從 [儲存體帳戶名稱] 清單中選取您的儲存體帳戶，然後選取 [完成]。

    ![設定 Azure 儲存體](./media/tutorial-copy-data-tool/configure-azure-storage.png)

    d. 選取新建立的連結服務作為來源，然後按一下 [下一步]。

    ![選取來源連結服務](./media/tutorial-copy-data-tool/select-source-linked-service.png)

1. 在 [選擇輸入檔案或資料夾] 頁面上，完成下列步驟︰
    
    a. 按一下 [瀏覽] 以瀏覽至 **adfv2tutorial/input** 資料夾，選取 **inputEmp.txt** 檔案，然後按一下 [選擇]。

    ![選擇輸入檔案或資料夾](./media/tutorial-copy-data-tool/specify-source-path.png)

    b. 按一下 [下一步] 移至下一個步驟。

1. 在 [檔案格式設定] 頁面上，留意到此工具會自動偵測資料行和資料列分隔符號。 選取 [下一步] 。 您也可以預覽資料，並檢視這個頁面上的輸入資料結構描述。 

    ![檔案格式設定](./media/tutorial-copy-data-tool/file-format-settings-page.png)
1. 在 [目的地資料存放區] 頁面上，完成下列步驟：

    a. 按一下 [+ 建立新連線] 以新增連線

    ![新增接收連結服務](./media/tutorial-copy-data-tool/new-sink-linked-service.png)

    b. 從資源庫選取 [Azure Blob 儲存體]，然後選取 [下一步]。

    ![選取 Azure SQL DB](./media/tutorial-copy-data-tool/select-azure-sql-db.png)

    c. 在 [新增連結服務] 頁面上，從下拉式清單中選取您的伺服器名稱和資料庫名稱，並指定使用者名稱和密碼，然後選取 [完成]。    

    ![設定 Azure SQL DB](./media/tutorial-copy-data-tool/config-azure-sql-db.png)

    d. 選取新建立的連結服務作為接收端，然後按一下 [下一步]。

    ![選取接收連結服務](./media/tutorial-copy-data-tool/select-sink-linked-service.png)

1. 在 [資料表對應] 頁面上，選取 **[dbo].[emp]** 資料表，然後選取 [下一步]。 

    ![資料表對應](./media/tutorial-copy-data-tool/table-mapping.png)
1. 在 [結構描述對應] 頁面上，請注意系統會將輸入檔案中的第一個和第二個資料行對應至 **emp** 資料表的 **FirstName** 和 **LastName** 資料行。 選取 [下一步] 。

    ![結構描述對應頁面](./media/tutorial-copy-data-tool/schema-mapping.png)
1. 在 [設定] 頁面上，選取 [下一步]。 
1. 在 [摘要] 頁面上檢閱設定，然後選取 [下一步]。

    ![摘要頁面](./media/tutorial-copy-data-tool/summary-page.png)
1. 在**部署頁面**上選取 [監視] 來監視管線 (工作)。

    ![部署頁面](./media/tutorial-copy-data-tool/deployment-page.png)
1. 請注意，系統會自動選取左側的 [監視] 索引標籤。 [動作] 資料行中會有連結可供檢視活動執行詳細資料，以及重新執行管線。 選取 [重新整理] 即可重新整理清單。 

    ![監視管線回合](./media/tutorial-copy-data-tool/pipeline-monitoring.png)
1. 若要檢視與此管線執行相關聯的活動執行，請選取 [動作] 資料行中的 [檢視活動執行] 連結。 如需關於複製作業的詳細資料，請選取 [動作] 資料行中的 [詳細資料] 連結 (眼鏡圖示)。 若要回到 [管線執行] 檢視，請選取頂端的 [管線] 連結。 若要重新整理檢視，請選取 [重新整理]。 

    ![監視活動回合](./media/tutorial-copy-data-tool/activity-monitoring.png)

    ![複製活動詳細資料](./media/tutorial-copy-data-tool/copy-execution-details.png)

1. 確認資料有插入 SQL 資料庫中的 **emp** 資料表。

    ![確認 SQL 輸出](./media/tutorial-copy-data-tool/verify-sql-output.png)

1. 選取左側的 [作者] 索引標籤以切換至編輯器模式。 您可以使用編輯器更新透過此工具建立的連結服務、資料集和管線。 如需在 Data Factory 使用者介面中編輯這些實體的詳細資訊，請參閱[教學課程的 Azure 入口網站版本](tutorial-copy-data-portal.md)。

## <a name="next-steps"></a>後續步驟
在此範例中，管線會將資料從 Blob 儲存體複製到 SQL 資料庫。 您已了解如何︰ 

> [!div class="checklist"]
> * 建立資料處理站。
> * 使用複製資料工具建立管線。
> * 監視管線和活動執行。

若要了解如何將資料從內部部署複製到雲端，請進入下列教學課程： 

> [!div class="nextstepaction"]
>[將資料從內部部署複製到雲端](tutorial-hybrid-copy-data-tool.md)
