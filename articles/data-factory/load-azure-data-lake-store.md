---
title: 使用 Azure Data Factory 將資料載入 Azure Data Lake Storage Gen1 | Microsoft Docs
description: 使用 Azure Data Factory 將資料複製到 Azure Data Lake Storage Gen1
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/17/2018
ms.author: jingwang
ms.openlocfilehash: e401508fc5ffc1de666f727ffbb7790005384fc1
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39003792"
---
# <a name="load-data-into-azure-data-lake-storage-gen1-by-using-azure-data-factory"></a>使用 Azure Data Factory 將資料載入 Azure Data Lake Storage Gen1

[Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md) (先前稱為 Azure Data Lake Store) 是容納巨量資料分析工作負載的企業級超大規模存放庫。 Azure Data Lake 可讓您擷取任何大小、類型和擷取速度的資料。 系統會在單一位置擷取資料以供作業及探勘分析之用。

Azure Data Factory 是完全受控的雲端式資料整合服務。 您可以使用此服務，在建置分析解決方案時於資料湖中填入現有系統的資料並節省時間。

Azure Data Factory 可針對將資料載入 Azure Data Lake Store 的作業提供下列優勢：

* **容易設定**：不需要編寫指令碼的直覺式 5 步驟精靈。
* **豐富的資料存放區支援**︰一組豐富內部部署和雲端式資料存放區的內部支援。 如需詳細清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)的資料表。
* **安全且符合規範**：資料會透過 HTTPS 或 ExpressRoute 來傳送。 具有全域服務，可確保資料絕不會離開地理界限。
* **高效能**：將資料載入 Azure Data Lake Store 中的速度高達 1 GB/s。 如需詳細資料，請參閱[複製活動效能](copy-activity-performance.md)。

本文將示範如何使用 Data Factory 複製資料工具，將資料從 Amazon S3 載入 Azure Data Lake Store 中。 您可以依照類似的步驟，從其他類型的資料存放區複製資料。

> [!NOTE]
> 如需詳細資訊，請參閱[使用 Azure Data Factory 將資料複製到 Azure Data Lake Store 或從該處複製資料](connector-azure-data-lake-store.md)。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶：如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。
* Azure Data Lake Store：如果您沒有 Data Lake Store 帳戶，請參閱[建立 Data Lake Store 帳戶](../data-lake-store/data-lake-store-get-started-portal.md#create-an-azure-data-lake-store-account)中的指示。
* Amazon S3：本文示範如何從 Amazon S3 複製資料。 您可以依照類似的步驟來使用其他資料存放區。

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 在左側功能表中，選取 [新增] > [資料 + 分析] > [資料處理站]：
   
   ![建立新的資料處理站](./media/load-data-into-azure-data-lake-store/new-azure-data-factory-menu.png)
2. 在 [新增資料處理站] 頁面中，為下圖所示的欄位提供值： 
      
   ![新增資料處理站頁面](./media/load-data-into-azure-data-lake-store//new-azure-data-factory.png)
 
    * **名稱**：輸入 Azure 資料處理站的全域唯一名稱。 如果您收到「資料處理站名稱 \"LoadADLSDemo\" 無法使用」的錯誤，請為資料處理站輸入其他名稱。 例如，您可以使用**您的名稱****ADFTutorialDataFactory**。 請嘗試再次建立資料處理站。 如需 Data Factory 成品的命名規則，請參閱 [Data Factory 命名規則](naming-rules.md)。
    * **訂用帳戶**：選取用來在其中建立資料處理站的 Azure 訂用帳戶。 
    * **資源群組**：從下拉式清單中選取現有資源群組，或選取 [新建] 選項，然後輸入資源群組的名稱。 若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-overview.md)。  
    * **版本**：選取 [V2]。
    * **位置**：選取資料處理站的位置。 只有受到支援的位置會顯示在下拉式清單中。 資料處理站所使用的資料存放區可位於其他位置和區域。 這些資料存放區包含 Azure Data Lake Store、Azure 儲存體、Azure SQL Database 等等。

3. 選取 [建立] 。
4. 建立完成後，請移至資料處理站。 您會看到如下圖所示的 [Data Factory] 首頁： 
   
   ![Data Factory 首頁](./media/load-data-into-azure-data-lake-store/data-factory-home-page.png)

   選取 [編寫與監視] 圖格，以在另一個索引標籤中啟動資料整合應用程式。

## <a name="load-data-into-azure-data-lake-store"></a>將資料載入 Azure Data Lake Store 中

1. 在 [開始使用] 頁面中，選取 [複製資料] 圖格以啟動複製資料工具： 

   ![複製資料工具圖格](./media/load-data-into-azure-data-lake-store/copy-data-tool-tile.png)
2. 在 [屬性] 頁面中，對 [工作名稱] 欄位指定 [CopyFromAmazonS3ToADLS]，然後選取 [下一步]：

    ![屬性頁面](./media/load-data-into-azure-data-lake-store/copy-data-tool-properties-page.png)
3. 在 [來源資料存放區] 頁面中，按一下 [+ Create new connection] \(+ 建立新連線\)：

    ![來源資料存放區頁面](./media/load-data-into-azure-data-lake-store/source-data-store-page.png)
    
    選取 [Amazon S3]，然後選取 [繼續]
    
    ![來源資料存放區 s3 頁面](./media/load-data-into-azure-data-lake-store/source-data-store-page-s3.png)
    
4. 在 [指定 Amazon S3 連線] 頁面中，執行下列步驟： 
   1. 指定 [存取金鑰識別碼] 值。
   2. 指定 [祕密存取金鑰] 值。
   3. 選取 [完成]。
   
   ![指定 Amazon S3 帳戶](./media/load-data-into-azure-data-lake-store/specify-amazon-s3-account.png)
   
   4. 您會看到新的連線。 選取 [下一步] 。
   
   ![指定 Amazon S3 帳戶](./media/load-data-into-azure-data-lake-store/specify-amazon-s3-account-created.png)
   
5. 在 [選擇輸入檔案或資料夾] 頁面中，瀏覽至您要複製過去的資料夾和檔案。 選取資料夾/檔案，選取 [選擇]，然後選取 [下一步]：

    ![選擇輸入檔案或資料夾](./media/load-data-into-azure-data-lake-store/choose-input-folder.png)

6. 選取 [以遞迴方式複製檔案] 和 [二進位複製] (依原樣複製檔案) 選項來選擇複製行為。 選取 [下一步]：

    ![指定輸出資料夾](./media/load-data-into-azure-data-lake-store/specify-binary-copy.png)
    
7. 在 [目的地資料存放區] 頁面中，按一下 [+ Create new connection] \(+ 建立新連線\)，然後選取 [Azure Data Lake Store]，並選取 [繼續]：

    ![目的地資料存放區頁面](./media/load-data-into-azure-data-lake-store/destination-data-storage-page.png)

8. 在 [指定 Data Lake Store 連線] 頁面中，執行下列步驟： 

   1. 選取您的 Data Lake Store 作為 [Data Lake Store 帳戶名稱]。
   2. 指定 [租用戶]，然後選取 [完成]。
   3. 選取 [下一步] 。
   
   > [!IMPORTANT]
   > 在本逐步解說中，您會使用_受控服務識別_來驗證 Data Lake Store。 請務必遵循[這些指示](connector-azure-data-lake-store.md#using-managed-service-identity-authentication)，以對 MSI 授與 Azure Data Lake Store 中的適當權限。
   
   ![指定 Azure Data Lake Store 帳戶](./media/load-data-into-azure-data-lake-store/specify-adls.png)
9. 在 [選擇輸出檔案或資料夾] 頁面中，輸入 [copyfroms3] 作為輸出資料夾名稱，然後選取 [下一步]： 

    ![指定輸出資料夾](./media/load-data-into-azure-data-lake-store/specify-adls-path.png)

10. 在 [設定] 頁面中，選取 [下一步]：

    ![設定頁面](./media/load-data-into-azure-data-lake-store/copy-settings.png)
11. 在 [摘要] 頁面中檢閱設定，然後選取 [下一步]：

    ![摘要頁面](./media/load-data-into-azure-data-lake-store/copy-summary.png)
12. 在**部署頁面**上選取 [監視] 來監視管線 (工作)：

    ![部署頁面](./media/load-data-into-azure-data-lake-store/deployment-page.png)
13. 請注意，系統會自動選取左側的 [監視] 索引標籤。 [動作] 資料行中會有連結可供檢視活動執行詳細資料，以及重新執行管線：

    ![監視管線回合](./media/load-data-into-azure-data-lake-store/monitor-pipeline-runs.png)
14. 若要檢視與此管線執行相關聯的活動執行，請選取 [動作] 資料行中的 [檢視活動執行] 連結。 管線中只有一個活動 (複製活動)，所以您只會看到一個項目。 若要切換回 [管線執行] 檢視，請選取頂端的 [管線] 連結。 選取 [重新整理] 即可重新整理清單。 

    ![監視活動回合](./media/load-data-into-azure-data-lake-store/monitor-activity-runs.png)

15. 若要監視每個複製活動的執行詳細資料，請選取活動監控檢視中 [動作] 底下的 [詳細資料] 連結。 您可以監視的詳細資料包括從來源複製到接收的資料量、資料輸送量、執行步驟與對應的持續期間，以及所使用的設定：

    ![監視活動執行詳細資料](./media/load-data-into-azure-data-lake-store/monitor-activity-run-details.png)

16. 確認資料已複製到 Azure Data Lake Store 中： 

    ![確認 Data Lake Store 輸出](./media/load-data-into-azure-data-lake-store/adls-copy-result.png)

## <a name="next-steps"></a>後續步驟

前往下列文章，以了解 Azure Data Lake Store 支援： 

> [!div class="nextstepaction"]
>[Azure Data Lake Store 連接器](connector-azure-data-lake-store.md)
