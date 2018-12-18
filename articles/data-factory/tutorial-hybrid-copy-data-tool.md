---
title: 使用 Azure 複製資料工具複製內部部署資料 | Microsoft Docs
description: 建立 Azure 資料處理站，然後使用「複製資料」工具將資料從內部部署 SQL Server 資料庫複製到 Azure Blob 儲存體。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 01/04/2018
ms.author: jingwang
ms.openlocfilehash: a6c17fc897dae765f9789840262cb001d598b731
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44297734"
---
# <a name="copy-data-from-an-on-premises-sql-server-database-to-azure-blob-storage-by-using-the-copy-data-tool"></a>使用複製資料工具將資料從內部部署 SQL Server 資料庫複製到 Azure Blob 儲存體
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [目前的版本](tutorial-hybrid-copy-data-tool.md)

在這個教學課程中，您會使用 Azure 入口網站來建立資料處理站。 接著，您會使用複製資料工具建立管線，以將資料從內部部署 SQL Server 資料庫複製到 Azure Blob 儲存體。

> [!NOTE]
> - 如果您不熟悉 Azure Data Factory，請參閱 [Data Factory 簡介](introduction.md)。
在本教學課程中，您會執行下列步驟：

> [!div class="checklist"]
> * 建立資料處理站。
> * 使用複製資料工具建立管線。
> * 監視管線和活動執行。

## <a name="prerequisites"></a>必要條件
### <a name="azure-subscription"></a>Azure 訂用帳戶
開始之前，如果您還沒有 Azure 訂用帳戶，[請建立免費帳戶](https://azure.microsoft.com/free/)。

### <a name="azure-roles"></a>Azure 角色
若要建立資料處理站執行個體，您用來登入 Azure 的使用者帳戶必須具備「參與者」或「擁有者」角色，或必須是 Azure 訂用帳戶的「管理員」。 

若要檢視您在訂用帳戶中擁有的權限，請前往 Azure 入口網站。 在右上角中，選取使用者名稱，然後選取 [權限]。 如果您有多個訂用帳戶的存取權，請選取適當的訂用帳戶。 如需如何將使用者新增至角色的範例指示，請參閱[使用 RBAC 和 Azure 入口網站來管理存取權](../role-based-access-control/role-assignments-portal.md)。

### <a name="sql-server-2014-2016-and-2017"></a>SQL Server 2014、2016 和 2017
在本教學課程中，您會使用內部部署 SQL 資料庫作為「來源」資料存放區。 您在本教學課程中建立於 Data Factory 的管線會將資料從此內部部署 SQL Server 資料庫 (來源) 複製到 Blob 儲存體 (接收)。 然後在 SQL Server 資料庫中建立名為 **emp** 的資料表，並在資料表中插入幾個範例項目。 

1. 啟動 SQL Server Management Studio。 如果它尚未安裝在機器上，請移至[下載 SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)。 

1. 使用您的認證連線到 SQL Server 執行個體。 

1. 建立範例資料庫。 在樹狀檢視中，以滑鼠右鍵按一下 [資料庫]，然後選取 [新增資料庫]。 

1. 在 [新增資料庫] 視窗中，輸入資料庫的名稱，然後選取 [確定]。 

1. 若要建立 **emp** 資料表並在其中插入一些範例資料，請針對資料庫執行下列查詢指令碼。 在樹狀檢視中，以滑鼠右鍵按一下您建立的資料庫，然後選取 [新增查詢]。

    ```sql
    CREATE TABLE dbo.emp
    (
        ID int IDENTITY(1,1) NOT NULL,
        FirstName varchar(50),
        LastName varchar(50)
    )
    GO
    
    INSERT INTO emp (FirstName, LastName) VALUES ('John', 'Doe')
    INSERT INTO emp (FirstName, LastName) VALUES ('Jane', 'Doe')
    GO
    ```

### <a name="azure-storage-account"></a>Azure 儲存體帳戶
在本教學課程中，您可以使用一般用途的 Azure 儲存體帳戶 (特別是 Blob 儲存體) 作為目的地/接收資料存放區。 如果您沒有一般用途的儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)來按照指示建立帳戶。 您在本教學課程中建立於 Data Factory 的管線會將資料從內部部署 SQL Server 資料庫 (來源) 複製到此 Blob 儲存體 (接收)。 

#### <a name="get-the-storage-account-name-and-account-key"></a>取得儲存體帳戶名稱和帳戶金鑰
您會在此教學課程中使用儲存體帳戶的名稱和金鑰。 採取下列步驟，以取得儲存體帳戶的名稱和金鑰： 

1. 使用您的使用者名稱和密碼登入 [Azure 入口網站](https://portal.azure.com)。 

1. 在左側窗格中，選取 [更多服務]。 使用 **Storage** 關鍵字進行篩選，然後選取 [儲存體帳戶]。

    ![儲存體帳戶搜尋](media/tutorial-hybrid-copy-powershell/search-storage-account.png)

1. 在儲存體帳戶清單中，篩選您的儲存體帳戶 (如有需要)。 然後選取您的儲存體帳戶。 

1. 在 [儲存體帳戶] 視窗中，選取 [存取金鑰]。

    ![存取金鑰](media/tutorial-hybrid-copy-powershell/storage-account-name-key.png)

1. 在 [儲存體帳戶名稱] 和 [金鑰1] 方塊中複製值，然後將它們貼到 [記事本] 或另一個編輯器中，以供稍後在教學課程中使用。 

#### <a name="create-the-adftutorial-container"></a>建立 adftutorial 容器 
在這一節中，您會在 Blob 儲存體中建立一個名為 **adftutorial** 的 Blob 容器。 

1. 在 [儲存體帳戶] 視窗中，切換至 [概觀]，然後選取 [Blob]。 

    ![選取 Blob 選項](media/tutorial-hybrid-copy-powershell/select-blobs.png)

1. 在 [Blob 服務] 視窗中，選取 [容器]。 

    ![容器按鈕](media/tutorial-hybrid-copy-powershell/add-container-button.png)

1. 在 [新容器] 視窗的 [名稱] 方塊中，輸入 **adftutorial**，然後選取 [確定]。 

    ![新的容器](media/tutorial-hybrid-copy-powershell/new-container-dialog.png)

1. 在容器清單中選取 [adftutorial]。

    ![容器選取](media/tutorial-hybrid-copy-powershell/seelct-adftutorial-container.png)

1. 保留 [容器] 視窗以供 **adftutorial** 開啟。 您可以在本快速入門結尾處使用它來確認輸出。 Data Factory 會在此容器中自動建立輸出資料夾，因此您不需要建立輸出資料夾。

    ![容器視窗](media/tutorial-hybrid-copy-powershell/container-page.png)


## <a name="create-a-data-factory"></a>建立 Data Factory

1. 在左側功能表中，選取 [新增] > [資料 + 分析] > [資料處理站]。 
  
   ![新資料處理站的建立](./media/tutorial-hybrid-copy-data-tool/new-azure-data-factory-menu.png)
1. 在 [新增資料處理站] 頁面的 [名稱] 下，輸入 **ADFTutorialDataFactory**。 
   
     ![新增 Data Factory](./media/tutorial-hybrid-copy-data-tool/new-azure-data-factory.png)

   資料處理站的名稱必須是「全域唯一」的名稱。 如果您在 [名稱] 欄位看到下列錯誤訊息，請變更資料處理站的名稱 (例如 yournameADFTutorialDataFactory)。 如需 Data Factory 成品的命名規則，請參閱 [Data Factory 命名規則](naming-rules.md)。

   ![新增資料處理站名稱](./media/tutorial-hybrid-copy-data-tool/name-not-available-error.png)
1. 選取您要在其中建立資料處理站的 Azure **訂用帳戶**。 
1. 針對 [資源群組]，採取下列其中一個步驟︰
  
      - 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。

      - 選取 [建立新的] ，然後輸入資源群組的名稱。 
        
      若要了解資源群組，請參閱[使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-overview.md)。
1. 在 [版本] 下，選取 **[V2]**。
1. 在 [位置] 下，選取資料處理站的位置。 只有受到支援的位置會顯示在下拉式清單中。 Data Factory 所使用的資料存放區 (例如 Azure 儲存體和 SQL Database) 和計算 (例如 Azure HDInsight) 可位於其他地區/區域。
1. 選取 [釘選到儀表板]。 
1. 選取 [建立] 。
1. 在儀表板上，您會看到 [正在部署 Data Factory] 狀態的下列圖格︰

    ![部署資料處理站圖格](media/tutorial-hybrid-copy-data-tool/deploying-data-factory.png)
1. 建立完成之後，您會看到如圖中所示的 [Data Factory] 頁面。
  
    ![Data Factory 首頁](./media/tutorial-hybrid-copy-data-tool/data-factory-home-page.png)
1. 選取 [編寫與監視]，以在另一個索引標籤中啟動 Data Factory 使用者介面。 

## <a name="use-the-copy-data-tool-to-create-a-pipeline"></a>使用複製資料工具建立管線

1. 在 [現在就開始吧] 頁面中，選取 [複製資料] 以啟動複製資料工具。 

   ![複製資料工具圖格](./media/tutorial-hybrid-copy-data-tool/copy-data-tool-tile.png)

1. 在複製資料工具的 [屬性] 頁面中上，於 [工作名稱] 下輸入 **CopyFromOnPremSqlToAzureBlobPipeline**。 然後，選取 [下一步]。 複製資料工具會使用您在此欄位指定的名稱建立管線。 

   ![工作名稱](./media/tutorial-hybrid-copy-data-tool/properties-page.png)

1. 在 [來源資料存放區] 頁面上，按一下 [建立新連線]。 

   ![建立新的連結服務](./media/tutorial-hybrid-copy-data-tool/create-new-source-data-store.png)

1. 在 [新增連結服務] 下搜尋 [SQL Server]，然後選取 [下一步]。 

   ![SQL Server 選取](./media/tutorial-hybrid-copy-data-tool/select-source-data-store.png)

1. 在 [新增連結服務 (SQL Server)] 的 **[名稱]**** 下，輸入 **SqlServerLinkedService**。 在 [透過整合執行階段連線] 下選取 [+新增]。 您必須建立自我裝載的整合執行階段、將其下載到您的機器，並使用 Data Factory 進行註冊。 自我裝載的整合執行階段會在內部部署環境與雲端之間複製資料。

   ![建立自我裝載的整合執行階段](./media/tutorial-hybrid-copy-data-tool/create-integration-runtime-link.png)

1. 在 [Integration Runtime 安裝] 對話方塊中，選取 [私人網路]。 然後，選取 [下一步]。 

   ![](./media/tutorial-hybrid-copy-data-tool/create-integration-runtime-dialog0.png)

1. 在 [Integration Runtime 安裝] 對話方塊中的 [名稱] 下，輸入 **TutorialIntegrationRuntime**。 然後，選取 [下一步]。 

   ![整合執行階段名稱](./media/tutorial-hybrid-copy-data-tool/create-integration-runtime-dialog.png)

1. 選取 [按一下這裡啟動此電腦的快速安裝]。 此動作會在您的機器上安裝整合執行階段，並使用 Data Factory 進行註冊。 或者，您可以使用手動安裝選項來下載安裝檔案、執行它，並使用金鑰來註冊整合執行階段。 

    ![[在這台電腦上啟動快速安裝] 連結](./media/tutorial-hybrid-copy-data-tool/launch-express-setup-link.png)

1. 執行下載的應用程式。 您會在視窗中看到快速安裝的狀態。 

    ![快速安裝狀態](./media/tutorial-hybrid-copy-data-tool/express-setup-status.png)

1. 確認已為 [整合執行階段] 欄位選取 [TutorialIntegrationRuntime]。

    ![已選取整合執行階段](./media/tutorial-hybrid-copy-data-tool/integration-runtime-selected.png)

1. 在 [指定內部部署 SQL Server 資料庫] 中，採取下列步驟： 

      a. 在 [名稱] 下輸入 **SqlServerLinkedService**。

      b. 在 [伺服器名稱] 下，輸入內部部署 SQL Server 執行個體的名稱。

      c. 在 [資料庫名稱] 下，輸入內部部署資料庫名稱。

      d. 在 [驗證類型] 下，選取適當的驗證。

      e. 在 [使用者名稱] 下，輸入具有內部部署 SQL Server 存取權的使用者名稱。

      f. 輸入使用者的 [密碼]。 選取 [完成]。 

1. 選取 [下一步] 。

     ![](./media/tutorial-hybrid-copy-data-tool/select-source-linked-service.png)

1. 在 [選取要從中複製資料的資料表或使用自訂查詢] 頁面中，選取清單中的 **[dbo].[emp]** 資料表，然後選取 [下一步]。 您可以根據自己的資料庫選取任何其他資料表。

     ![生產資料表選取項目](./media/tutorial-hybrid-copy-data-tool/select-emp-table.png)

1. 在 [目的地資料存放區] 頁面上，選取 [建立新連線]

     //image create-new-sink-connection.png

     ![建立目的地連結服務](./media/tutorial-hybrid-copy-data-tool/create-new-sink-connection.png)

1. 在 [新增連結服務] 中，搜尋並選取 [Azure Blob]，然後選取 [繼續]。 

     ![Blob 儲存體選取](./media/tutorial-hybrid-copy-data-tool/select-destination-data-store.png)

1. 在 [新增連結服務 (Azure Blob 儲存體)] 對話方塊上，執行下列步驟： 

     a. 在 **[名稱]**** 下，輸入 **AzureStorageLinkedService**。

     b. 在 [透過整合執行階段連線] 下，選取 [TutorialIntegrationRuntime]

     c. 在 [儲存體帳戶名稱] 下，從下拉式清單中選取您的儲存體帳戶。 

     d. 選取 [下一步] 。

     ![指定儲存帳號](./media/tutorial-hybrid-copy-data-tool/specify-azure-blob-storage-account.png)

1. 在 [目的地資料存放區] 對話方塊中，選取 [下一步]。 在 [連線屬性] 中，選取 [Azure 儲存體服務] 作為 [Azure Blob 儲存體]。 選取 [下一步] 。 

     ![連線屬性](./media/tutorial-hybrid-copy-data-tool/select-connection-properties.png)

1. 在 [選擇輸出檔案或資料夾] 對話方塊中的 [資料夾路徑] 下，輸入 **adftutorial/fromonprem**。 您已建立 **adftutorial** 容器作為必要條件的一部分。 如果輸出資料夾不存在 (在此案例中為 **fromonprem**)，Data Factory 會自動加以建立。 您也可以使用 [瀏覽] 按鈕以瀏覽 Blob 儲存體和其容器/資料夾。 如果您未在 [檔案名稱] 下指定任何值，依預設將會使用來自來源的名稱 (在此案例中為 **dbo.emp**)。
           
     ![選擇輸出檔案或資料夾](./media/tutorial-hybrid-copy-data-tool/choose-output-file-folder.png)

1. 在 [檔案格式設定] 對話方塊上，選取 [下一步]。 

     ![檔案格式設定頁面](./media/tutorial-hybrid-copy-data-tool/file-format-settings-page.png)

1. 在 [設定] 對話方塊上，選取 [下一步]。 

     ![設定頁面](./media/tutorial-hybrid-copy-data-tool/settings-page.png)

1. 在 [摘要] 對話方塊上，檢閱所有設定的值，然後選取 [下一步]。 

     ![摘要頁面](./media/tutorial-hybrid-copy-data-tool/summary-page.png)

1. 在 [部署] 頁面中，選取 [監視] 來監視您建立的管線或工作。

     ![部署頁面](./media/tutorial-hybrid-copy-data-tool/deployment-page.png)

1. 在 [監視] 索引標籤上，您可以檢視您所建立管線的狀態。 您可以使用 [動作] 資料行中的連結檢視與此管線執行相關聯的活動執行，以及重新執行管線。 

     ![監視管線回合](./media/tutorial-hybrid-copy-data-tool/monitor-pipeline-runs.png)

1. 選取 [動作] 資料行中的 [檢視活動執行] 連結，以查看與此管線執行相關聯的活動執行。 若要看與關於複製作業的詳細資料，請選取 [動作] 資料行中的 [詳細資料] 連結 (眼鏡圖示)。 若要切換回 [管線執行] 檢視，請選取頂端的 [管線]。

     ![監視活動回合](./media/tutorial-hybrid-copy-data-tool/monitor-activity-runs.png)

1. 確認您在 **adftutorial** 容器的 **fromonprem** 資料夾中看到輸出檔案。 

     ![輸出 Blob](./media/tutorial-hybrid-copy-data-tool/output-blob.png)

1. 選取左側的 [編輯] 索引標籤以切換至編輯器模式。 您可以使用編輯器更新此工具所建立的連結服務、資料集和管線。 選取 [程式碼] 以檢視與編輯器中開啟之實體相關聯的 JSON 程式碼。 如需如何在 Data Factory 使用者介面中編輯這些實體的詳細資訊，請參閱[本教學課程的 Azure 入口網站版本](tutorial-copy-data-portal.md)。

     ![編輯索引標籤](./media/tutorial-hybrid-copy-data-tool/edit-tab.png)


## <a name="next-steps"></a>後續步驟
此範例中的管線會將資料從內部部署 SQL Server 資料庫複製到 Blob 儲存體。 您已了解如何︰ 

> [!div class="checklist"]
> * 建立資料處理站。
> * 使用複製資料工具建立管線。
> * 監視管線和活動執行。

如需 Data Factory 所支援的資料存放區清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。

若要了解如何將大量資料從來源複製到目的地，請進入下列教學課程：

> [!div class="nextstepaction"]
>[複製大量資料](tutorial-bulk-copy-portal.md)
