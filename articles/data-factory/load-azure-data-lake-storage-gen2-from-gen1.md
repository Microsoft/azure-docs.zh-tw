---
title: 使用 Azure Data Factory 將資料從 Azure Data Lake Storage Gen1 複製到 Gen2 (預覽)
description: 使用 Azure Data Factory 將資料從 Azure Data Lake Storage Gen1 複製到 Gen2 (預覽)
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 07/06/2018
ms.author: jingwang
ms.openlocfilehash: a160c47e12db3c4ef9cefc5cd70293468ddf8234
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39011356"
---
# <a name="copy-data-from-azure-data-lake-storage-gen1-to-gen2-preview-with-azure-data-factory"></a>使用 Azure Data Factory 將資料從 Azure Data Lake Storage Gen1 複製到 Gen2 (預覽)

[Azure Data Lake Storage Gen2 (預覽)](../storage/data-lake-storage/introduction.md)會將具有階層式檔案系統命名空間和安全性功能的通訊協定，新增至 Azure Blob 儲存體，使其易於將分析架構連接到耐久儲存層。 在 Data Lake Storage Gen2 (預覽) 中，所有物件儲存體的品質都維持不變，另外還增加了檔案系統介面的優點。

如果您目前使用 Azure Data Lake Storage Gen1，您可以藉由使用 Azure Data Factory 將資料從 Data Lake Storage Gen1 複製到 Gen2 來評估 Gen2 的新功能。

Azure Data Factory 是完全受控的雲端式資料整合服務。 您可以使用此服務，在建置分析解決方案時，於 Lake 中置入來自豐富的內部部署集合和雲端式資料存放區的資料，並節省時間。 如需受支援連接器的詳細清單，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)資料表。

Azure Data Factory 提供可向外延展的受控資料移動解決方案。 由於 ADF 具有相應放大架構，因此能以高輸送量來內嵌資料。 如需詳細資料，請參閱[複製活動效能](copy-activity-performance.md)。

本文示範如何使用 Data Factory 複製資料工具，將資料從 Azure Data Lake Storage Gen1 複製到 Azure Data Lake Storage Gen2。 您可以依照類似的步驟，從其他類型的資料存放區複製資料。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶：如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。
* 內有資料的 Azure Data Lake Storage Gen1 帳戶。
* 已啟用 Data Lake Storage Gen2 的 Azure 儲存體帳戶：如果您還沒有儲存體帳戶，請按一下[這裡](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM)建立一個帳戶。

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 在左側功能表中，選取 [新增] > [資料 + 分析] > [資料處理站]：
   
   ![建立新的資料處理站](./media/load-azure-data-lake-storage-gen2-from-gen1/new-azure-data-factory-menu.png)
2. 在 [新增資料處理站] 頁面中，為下圖所示的欄位提供值： 
      
   ![新增資料處理站頁面](./media/load-azure-data-lake-storage-gen2-from-gen1/new-azure-data-factory.png)
 
    * **名稱**：輸入 Azure 資料處理站的全域唯一名稱。 如果您收到「資料處理站名稱 \"LoadADLSDemo\" 無法使用」的錯誤，請為資料處理站輸入其他名稱。 例如，您可以使用**您的名稱****ADFTutorialDataFactory**。 請嘗試再次建立資料處理站。 如需 Data Factory 成品的命名規則，請參閱 [Data Factory 命名規則](naming-rules.md)。
    * **訂用帳戶**：選取用來在其中建立資料處理站的 Azure 訂用帳戶。 
    * **資源群組**：從下拉式清單中選取現有資源群組，或選取 [新建] 選項，然後輸入資源群組的名稱。 若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-overview.md)。  
    * **版本**：選取 [V2]。
    * **位置**：選取資料處理站的位置。 只有受到支援的位置會顯示在下拉式清單中。 資料處理站所使用的資料存放區可位於其他位置和區域。 

3. 選取 [建立] 。
4. 建立完成後，請移至資料處理站。 您會看到如下圖所示的 [Data Factory] 首頁： 
   
   ![Data Factory 首頁](./media/load-azure-data-lake-storage-gen2-from-gen1/data-factory-home-page.png)

   選取 [編寫與監視] 圖格，以在另一個索引標籤中啟動資料整合應用程式。

## <a name="load-data-into-azure-data-lake-storage-gen2"></a>將資料載入 Azure Data Lake Storage Gen2 中

1. 在 [開始使用] 頁面中，選取 [複製資料] 圖格以啟動複製資料工具： 

   ![複製資料工具圖格](./media/load-azure-data-lake-storage-gen2-from-gen1/copy-data-tool-tile.png)
2. 在 [屬性] 頁面中，對 [工作名稱] 欄位指定 [CopyFromADLSGen1ToGen2]，然後選取 [下一步]：

    ![屬性頁面](./media/load-azure-data-lake-storage-gen2-from-gen1/copy-data-tool-properties-page.png)
3. 在 [來源資料存放區] 頁面中，按一下 [+ Create new connection] \(+ 建立新連線\)：

    ![來源資料存放區頁面](./media/load-azure-data-lake-storage-gen2-from-gen1/source-data-store-page.png)
    
    從連接器資源庫選取 [Azure Data Lake Storage Gen1]，然後選取 [繼續]
    
    ![來源資料存放區 Azure Data Lake Storage Gen1 頁面](./media/load-azure-data-lake-storage-gen2-from-gen1/source-data-store-page-adls-gen1.png)
    
4. 在 [指定 Azure Data Lake Storage Gen1 連線] 頁面中，執行下列步驟：
   1. 選取 [Data Lake Storage Gen1] 作為帳戶名稱。
   2. 指定或驗證 [租用戶]，然後選取 [完成]。
   3. 選取 [下一步] 。
   
   > [!IMPORTANT]
   > 在本逐步解說中，您會使用_受控服務識別_來驗證 Data Lake Storage Gen1。 請務必遵循[這些指示](connector-azure-data-lake-store.md#using-managed-service-identity-authentication)，以對 MSI 授與 Azure Data Lake Storage Gen1 中的適當權限。
   
   ![指定 Azure Data Lake Storage Gen1 帳戶](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-adls-gen1-account.png)
   
   4. 您會看到新的連線隨即建立。 選取 [下一步] 。
   
5. 在 [選擇輸入檔案或資料夾] 頁面中，瀏覽至您要複製過去的資料夾和檔案。 選取資料夾/檔案，選取 [選擇]：

    ![選擇輸入檔案或資料夾](./media/load-azure-data-lake-storage-gen2-from-gen1/choose-input-folder.png)

6. 勾選 [以遞迴方式複製檔案] 和 [二進位複製] 選項來選取複製行為。 選取 [下一步]：

    ![指定輸出資料夾](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-binary-copy.png)
    
7. 在 [目的地資料存放區] 頁面中，按一下 [+ 建立新連線]，然後選取 [Azure Data Lake Storage Gen2 (預覽)]，並選取 [繼續]：

    ![目的地資料存放區頁面](./media/load-azure-data-lake-storage-gen2-from-gen1/destination-data-storage-page.png)

8. 在 [指定 Azure Data Lake Storage Gen2 連線] 頁面中，執行下列步驟：

   1. 從 [儲存體帳戶名稱] 下拉式清單中選取您的 Data Lake Storage Gen2 可使用帳戶。
   2. 選取 [下一步] 。
   
   ![指定 Azure Data Lake Storage Gen2 帳戶](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-adls-gen2-account.png)

9. 在 [選擇輸出檔案或資料夾] 頁面中，輸入 [copyfromadlsgen1] 作為輸出資料夾名稱，然後選取 [下一步]： 

    ![指定輸出資料夾](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-adls-gen2-path.png)

10. 在 [設定] 頁面中選取 [下一步]，以使用預設設定。

11. 在 [摘要] 頁面中檢閱設定，然後選取 [下一步]：

    ![摘要頁面](./media/load-azure-data-lake-storage-gen2-from-gen1/copy-summary.png)
12. 在 [部署] 頁面中選取 [監視] 來監視管線：

    ![部署頁面](./media/load-azure-data-lake-storage-gen2-from-gen1/deployment-page.png)
13. 請注意，系統會自動選取左側的 [監視] 索引標籤。 [動作] 資料行中會有連結可供檢視活動執行詳細資料，以及重新執行管線：

    ![監視管線回合](./media/load-azure-data-lake-storage-gen2-from-gen1/monitor-pipeline-runs.png)

14. 若要檢視與此管線執行相關聯的活動執行，請選取 [動作] 資料行中的 [檢視活動執行] 連結。 管線中只有一個活動 (複製活動)，所以您只會看到一個項目。 若要切換回 [管線執行] 檢視，請選取頂端的 [管線] 連結。 選取 [重新整理] 即可重新整理清單。 

    ![監視活動回合](./media/load-azure-data-lake-storage-gen2-from-gen1/monitor-activity-runs.png)

15. 若要監視每個複製活動的執行詳細資料，請選取活動監控檢視中 [動作] 底下的 [詳細資料] 連結 (眼鏡圖示)。 您可以監視的詳細資料包括從來源複製到接收的資料量、資料輸送量、執行步驟與對應的持續期間，以及所使用的設定：

    ![監視活動執行詳細資料](./media/load-azure-data-lake-storage-gen2-from-gen1/monitor-activity-run-details.png)

16. 確認資料已複製到 Data Lake Storage Gen2 帳戶中。

## <a name="best-practices"></a>最佳作法

從檔案型資料存放區複製大量資料時，建議您：

- 將檔案分割成 10TB 到 30TB 的檔案集。
- 請勿觸發太多並行複製執行，以避免來源或接收資料存放區的節流。 您可以從一個複製執行開始並且監視輸送量，然後視需要逐漸增加。

## <a name="next-steps"></a>後續步驟

* [複製活動概觀](copy-activity-overview.md)
* [Azure Data Lake Storage Gen2 連接器](connector-azure-data-lake-storage.md)