---
title: 在 Azure Data Factory 管道中分支 | Microsoft Docs
description: 了解如何將活動分支和鏈結來控制 Azure Data Factory 中的資料流程。
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/11/2018
ms.author: shlo
ms.openlocfilehash: 4cb133cc617ecc121fb93a4da816120986e131e8
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43086921"
---
# <a name="branching-and-chaining-activities-in-a-data-factory-pipeline"></a>在 Data Factory 管道中將活動分支和鏈結
在本教學課程中，您會建立 Data Factory 管道來展示部分的控制流程功能。 這個管道只是簡單地從 Azure Blob 儲存體中的一個容器複製到相同儲存體帳戶中的另一個容器。 如果複製活動成功，管線會在成功電子郵件中傳送成功複製作業的詳細資料 (例如寫入的資料量)。 如果複製活動失敗，管線會在失敗電子郵件中傳送複製失敗的詳細資料 (例如錯誤訊息)。 在整個教學課程中，您會看到如何傳遞參數。

情節的高階概觀：![概觀](media/tutorial-control-flow-portal/overview.png)

您會在本教學課程中執行下列步驟：

> [!div class="checklist"]
> * 建立資料處理站。
> * 建立 Azure 儲存體連結服務。
> * 建立 Azure Blob 資料集
> * 建立其中包含複製活動和網路活動的管線
> * 將活動的輸出傳送至後續的活動
> * 利用參數傳遞和系統變數
> * 啟動管道執行
> * 監視管道和活動執行

本教學課程使用 Azure 入口網站。 您可以使用其他機制來與 Azure Data Factory 互動，請參閱目錄中的「快速入門」。

## <a name="prerequisites"></a>必要條件

* **Azure 訂用帳戶**。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。
* **Azure 儲存體帳戶**。 您會使用 Blob 儲存體作為**來源**資料存放區。 如果您沒有 Azure 儲存體帳戶，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)一文，按照步驟來建立帳戶。
* **Azure SQL Database**。 您會使用資料庫作為**接收**資料存放區。 如果您沒有 Azure SQL Database，請參閱[建立 Azure SQL 資料庫](../sql-database/sql-database-get-started-portal.md)一文，按照步驟建立資料庫。

### <a name="create-blob-table"></a>建立 Blob 資料表

1. 啟動 [記事本]。 複製下列文字，並在磁碟上儲存為 **input.txt** 檔案。

    ```
    John,Doe
    Jane,Doe
    ```
2. 使用 [Azure 儲存體總管](http://storageexplorer.com/)之類的工具來執行以下步驟： 
    1. 建立 **adfv2branch** 容器。
    2. 在 **adfv2branch** 容器中建立 **input** 資料夾。
    3. 將 **input.txt** 檔案上傳至容器。

## <a name="create-email-workflow-endpoints"></a>建立電子郵件工作流程端點
若要從管線觸發電子郵件的傳送，您需要使用 [Logic Apps](../logic-apps/logic-apps-overview.md) 來定義工作流程。 如需有關建立邏輯應用程式工作流程的詳細資訊，請參閱[如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 

### <a name="success-email-workflow"></a>成功電子郵件工作流程 
建立名為 `CopySuccessEmail` 的邏輯應用程式工作流程。 將工作流程觸發程序定義為 `When an HTTP request is received`，並新增動作 `Office 365 Outlook – Send an email`。

![成功電子郵件工作流程](media/tutorial-control-flow-portal/success-email-workflow.png)

針對您的要求觸發程序，在 `Request Body JSON Schema` 中填寫下列 JSON：

```json
{
    "properties": {
        "dataFactoryName": {
            "type": "string"
        },
        "message": {
            "type": "string"
        },
        "pipelineName": {
            "type": "string"
        },
        "receiver": {
            "type": "string"
        }
    },
    "type": "object"
}
```

在邏輯應用程式設計工具中，要求看起來應該如下圖所示： 

![邏輯應用程式設計工具 - 要求](media/tutorial-control-flow-portal/logic-app-designer-request.png)

對於**傳送電子郵件**動作，利用要求本文 JSON 結構描述中傳遞的屬性，以自訂您要如何格式化電子郵件。 下列是一個範例：

![邏輯應用程式設計工具 - 傳送電子郵件動作](media/tutorial-control-flow-portal/send-email-action-2.png)

儲存工作流程。 請記下成功電子郵件工作流程的 HTTP Post 要求 URL：

```
//Success Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

### <a name="fail-email-workflow"></a>失敗電子郵件工作流程 
依照相同的步驟來建立另一個 Logic Apps 工作流程 **CopyFailEmail**。 在要求觸發程序中，`Request Body JSON schema` 相同。 變更電子郵件的格式 (例如 `Subject`)，以調整為適合失敗電子郵件。 下列是一個範例：

![邏輯應用程式設計工具 - 失敗電子郵件工作流程](media/tutorial-control-flow-portal/fail-email-workflow-2.png)

儲存工作流程。 請記下失敗電子郵件工作流程的 HTTP Post 要求 URL：

```
//Fail Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

您現在應該有兩個工作流程 URL：

```
//Success Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000

//Fail Request Url
https://prodxxx.eastus.logic.azure.com:443/workflows/000000/triggers/manual/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=000000
```

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 啟動 **Microsoft Edge** 或 **Google Chrome** 網頁瀏覽器。 目前，只有 Microsoft Edge 和 Google Chrome 網頁瀏覽器支援 Data Factory UI。
1. 按一下左邊功能表上的 [新增]、[資料 + 分析]，再按一下 [Data Factory]。 
   
   ![新增->DataFactory](./media/tutorial-control-flow-portal/new-azure-data-factory-menu.png)
2. 在 [新增資料處理站] 頁面中，輸入 **ADFTutorialDataFactory** 作為 [名稱]。 
      
     ![新增資料處理站頁面](./media/tutorial-control-flow-portal/new-azure-data-factory.png)
 
   Azure Data Factory 的名稱必須是 **全域唯一的**。 如果您收到錯誤，請變更 Data Factory 名稱 (例如 yournameADFTutorialDataFactory)，然後試著重新建立。 請參閱 [Data Factory - 命名規則](naming-rules.md)一文，以了解 Data Factory 成品的命名規則。
  
       `Data factory name “ADFTutorialDataFactory” is not available`
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

    ![部署資料處理站圖格](media/tutorial-control-flow-portal/deploying-data-factory.png)
9. 建立完成之後，您會看到如圖中所示的 [Data Factory] 頁面。
   
   ![Data Factory 首頁](./media/tutorial-control-flow-portal/data-factory-home-page.png)
10. 按一下 [撰寫與監視] 圖格，以在另一個索引標籤中啟動 Azure Data Factory 使用者介面 (UI)。


## <a name="create-a-pipeline"></a>建立管線
在此步驟中，您會建立具有一個複製活動與兩個 Web 活動的管線。 您會使用下列功能來建立管線：

- 管線參數，該參數由資料集所存取。 
- Web 活動，可叫用邏輯應用程式工作流程來傳送成功/失敗電子郵件。 
- 將一個活動與另一個活動連線 (成功和失敗時)
- 使用一個活動的輸出作為後續活動的輸入

1. 在 Data Factory 使用者介面的 [開始使用] 頁面中，按一下 [建立管線] 圖格。  

   ![開始使用頁面](./media/tutorial-control-flow-portal/get-started-page.png) 
3. 在管線的 [屬性] 視窗中，切換至 [參數] 索引標籤，然後使用 [新增] 按鈕即可新增 String 類型的下列三個參數：sourceBlobContainer、sinkBlobContainer 和 receiver。 

    - **sourceBlobContainer** - 管線中由來源 Blob 資料集所取用的參數。
    - **sinkBlobContainer** – 管線中由接收 Blob 資料集所取用的參數
    - **接收者** – 管線中的兩項 Web 活動會使用此參數，將成功或失敗電子郵件傳送給其電子郵件地址由此參數指定的接收者。

   ![新增管線功能表](./media/tutorial-control-flow-portal/pipeline-parameters.png)
4. 在 [活動] 工具箱中展開 [資料流程]，並將 [複製] 活動拖曳至管線設計工具介面。 

   ![拖放複製活動](./media/tutorial-control-flow-portal/drag-drop-copy-activity.png)
5. 在底部 [複製] 活動的 [屬性] 視窗中，切換到 [來源] 索引標籤，然後按一下 [+ 新增]。 您在此步驟中為複製活動建立來源資料集。 

   ![來源資料集](./media/tutorial-control-flow-portal/new-source-dataset-button.png)
6. 在 [新增資料集] 視窗中選取 [Azure Blob 儲存體]，然後按一下 [完成]。 

   ![選取 Azure Blob 儲存體](./media/tutorial-control-flow-portal/select-azure-blob-storage.png)
7. 您會看到標題為 **AzureBlob1** 的新**索引標籤**。 將資料集的名稱變更為 **SourceBlobDataset**。

   ![資料集一般設定](./media/tutorial-control-flow-portal/dataset-general-page.png)
8. 在 [屬性] 視窗中切換至 [連線] 索引標籤，針對 [連結服務] 按一下 [新增]。 在此步驟中，您會建立連結服務，將 Azure 儲存體帳戶連結到資料處理站。 
    
   ![資料集連線 - 新的連結服務](./media/tutorial-control-flow-portal/dataset-connection-new-button.png)
9. 在 [新增連結服務] 視窗中，執行下列步驟： 

    1. 輸入 **AzureStorageLinkedService** 作為 [名稱]。
    2. 針對 [儲存體帳戶名稱] 選取您的 Azure 儲存體帳戶。
    3. 按一下 [檔案] 。

   ![新增 Azure 儲存體連結服務](./media/tutorial-control-flow-portal/new-azure-storage-linked-service.png)
12. 針對資料夾輸入 `@pipeline().parameters.sourceBlobContainer`，針對檔案名稱輸入 `emp.txt`。 您可以使用 sourceBlobContainer 管線參數來為資料集設定資料夾路徑。 

    ![來源資料集設定](./media/tutorial-control-flow-portal/source-dataset-settings.png)
13. 切換至 [管線] 索引標籤 (或) 按一下樹狀檢視中的 [管線]。 確認已為 [來源資料集] 選取 [SourceBlobDataset]。 

   ![來源資料集](./media/tutorial-control-flow-portal/pipeline-source-dataset-selected.png)
13. 在 [屬性] 視窗中，切換至 [接收] 索引標籤，然後為 [接收資料集] 按一下 [+ 新增]。 您在此步驟中為複製活動建立的接收資料集與您建立來源資料集的方式類似。 

    ![新增接收資料集按鈕](./media/tutorial-control-flow-portal/new-sink-dataset-button.png)
14. 在 [新增資料集] 視窗中選取 [Azure Blob 儲存體]，然後按一下 [完成]。 
15. 在資料集的 [一般] 設定頁面上，輸入 **SinkBlobDataset** 作為 [名稱]。
16. 切換至 [連線] 索引標籤，然後執行下列步驟： 

    1. 針對 [連結服務] 選取 [AzureStorageLinkedService]。
    2. 針對資料夾輸入 `@pipeline().parameters.sinkBlobContainer`。
    1. 針對檔案名稱輸入 `@CONCAT(pipeline().RunId, '.txt')`。 運算式會使用目前的管線執行識別碼供檔案名稱使用。 如需支援的系統變數和運算式清單，請參閱[系統變數](control-flow-system-variables.md)和[運算式語言](control-flow-expression-language-functions.md)。

        ![接收資料集設定](./media/tutorial-control-flow-portal/sink-dataset-settings.png)
17. 切換至頂端的 [管線] 索引標籤。 在 [活動] 工具箱中展開 [一般]，並將 [Web] 活動拖放至管線設計工具表面。 將活動的名稱設定為 **SendSuccessEmailActivity**。 「網路活動」允許呼叫任何 REST 端點。 如需活動的詳細資訊，請參閱[網路活動](control-flow-web-activity.md)。 這個管道會使用「網路活動」來呼叫 Logic Apps 電子郵件工作流程。 

   ![拖放第一個 Web 活動](./media/tutorial-control-flow-portal/success-web-activity-general.png)
18. 從 [一般] 索引標籤切換至 [設定] 索引標籤，並執行下列步驟： 
    1. 針對 [URL]，指定傳送成功電子郵件之邏輯應用程式工作流程的 URL。  
    2. 針對 [方法] 選取 [POST]。 
    3. 按一下 [標頭] 區段中的 [+ 新增標頭] 連結。 
    4. 新增標頭 [Content-type] 並將它設定為 **application/json**。 
    5. 為 [主體] 指定下列 JSON。 

        ```json
        {
            "message": "@{activity('Copy1').output.dataWritten}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
        }
        ```
        訊息主體包含下列屬性：

        - 訊息 – 傳遞 `@{activity('Copy1').output.dataWritten` 的值。 存取先前複製活動的屬性，並傳遞 dataWritten 的值。 對於失敗案例，請傳遞錯誤輸出，而不是 `@{activity('CopyBlobtoBlob').error.message`。
        - 資料處理站名稱 – 傳遞 `@{pipeline().DataFactory}` 的值。這是系統變數，可讓您存取對應的資料處理站名稱。 如需系統變數的清單，請參閱[系統變數](control-flow-system-variables.md)一文。
        - 管道名稱 - 傳遞 `@{pipeline().Pipeline}` 的值。 這也是系統變數，可讓您存取對應的管道名稱。 
        - 接收者 – 傳遞 "\@pipeline().parameters.receiver") 的值。 存取管道參數。
    
        ![第一個 Web 活動的設定](./media/tutorial-control-flow-portal/web-activity1-settings.png)         
19. 將 [複製] 活動連線至 [Web] 活動，方法是將複製活動旁的綠色按鈕拖放在 Web 活動上。 

    ![使用第一個 Web 活動連線複製活動](./media/tutorial-control-flow-portal/connect-copy-web-activity1.png)
20. 將另一個 [Web] 活動從 [活動] 工具箱拖放至管線設計工具表面，並將 [名稱] 設定為 **SendFailureEmailActivity**。

    ![第二個 Web 活動的名稱](./media/tutorial-control-flow-portal/web-activity2-name.png)
21. 切換至 [設定] 索引標籤，並執行下列步驟：

    1. 針對 [URL]，指定傳送失敗電子郵件之邏輯應用程式工作流程的 URL。  
    2. 針對 [方法] 選取 [POST]。 
    3. 按一下 [標頭] 區段中的 [+ 新增標頭] 連結。 
    4. 新增標頭 [Content-type] 並將它設定為 **application/json**。 
    5. 為 [主體] 指定下列 JSON。 

        ```json
        {
            "message": "@{activity('Copy1').error.message}",
            "dataFactoryName": "@{pipeline().DataFactory}",
            "pipelineName": "@{pipeline().Pipeline}",
            "receiver": "@pipeline().parameters.receiver"
        }
        ```

        ![第二個 Web 活動的設定](./media/tutorial-control-flow-portal/web-activity2-settings.png)         
22. 在管線設計工具中選取 [複製] 活動，按一下 +-> 按鈕，然後選取 [錯誤]。  

    ![第二個 Web 活動的設定](./media/tutorial-control-flow-portal/select-copy-failure-link.png)
23. 將複製活動旁邊的**紅色**按鈕拖曳至第二個 Web 活動 **SendFailureEmailActivity**。 您可以將活動四處移動，讓管線看起來如下圖所示： 

    ![具有所有活動的完整管線](./media/tutorial-control-flow-portal/full-pipeline.png)
24. 若要驗證管線，按一下工具列上的 [驗證] 按鈕。 按一下 >> 按鈕，來關閉 [管線驗證輸出] 視窗。

    ![驗證管線](./media/tutorial-control-flow-portal/validate-pipeline.png)
24. 若要將實體 (資料集、管線等等) 發佈至 Data Factory 服務，請選取 [全部發佈]。 請靜待 [發佈成功] 訊息顯示。

    ![發佈](./media/tutorial-control-flow-portal/publish-button.png)
 
## <a name="trigger-a-pipeline-run-that-succeeds"></a>觸發成功的管線執行
1. 若要**觸發**管狀執行，按一下工具列上的 [觸發]，然後按一下 [立即觸發]。 

    ![觸發管線執行](./media/tutorial-control-flow-portal/trigger-now-menu.png)
2. 在 [管線執行] 視窗中，執行下列步驟： 

    1. 輸入 **adftutorial/adfv2branch/input** 作為 **sourceBlobContainer** 參數。 
    2. 輸入 **adftutorial/adfv2branch/output** 作為 **sourceBlobContainer** 參數。 
    3. 輸入**接收者**的**電子郵件地址**。 
    4. 按一下 [完成]

        ![管線執行參數](./media/tutorial-control-flow-portal/pipeline-run-parameters.png)

## <a name="monitor-the-successful-pipeline-run"></a>監視成功的管線執行

1. 若要監視管線執行，請切換到左側的 [監視] 索引標籤。 您會看到由您手動觸發的管線執行。 使用 [重新整理] 按鈕可重新整理清單。 
    
    ![成功的管線執行](./media/tutorial-control-flow-portal/monitor-success-pipeline-run.png)
2. 若要檢視與此管線執行相關聯的**活動執行**，請按一下 [動作] 資料行中的第一個連結。 您可以按一下頂端的 [管線]，來切換回到前一個檢視。 使用 [重新整理] 按鈕可重新整理清單。 

    ![活動執行](./media/tutorial-control-flow-portal/activity-runs-success.png)

## <a name="trigger-a-pipeline-run-that-fails"></a>觸發失敗的管線執行
1. 切換至左側的 [編輯] 索引標籤。 
2. 若要**觸發**管狀執行，按一下工具列上的 [觸發]，然後按一下 [立即觸發]。 
3. 在 [管線執行] 視窗中，執行下列步驟： 

    1. 輸入 **adftutorial/dummy/input** 作為 **sourceBlobContainer** 參數。 請確定 adftutorial 容器中不存在 dummy 資料夾。 
    2. 輸入 **adftutorial/dummy/output** 作為 **sourceBlobContainer** 參數。 
    3. 輸入**接收者**的**電子郵件地址**。 
    4. 按一下 [完成] 。

## <a name="monitor-the-failed-pipeline-run"></a>監視失敗的管線執行

1. 若要監視管線執行，請切換到左側的 [監視] 索引標籤。 您會看到由您手動觸發的管線執行。 使用 [重新整理] 按鈕可重新整理清單。 
    
    ![失敗管線執行](./media/tutorial-control-flow-portal/monitor-failure-pipeline-run.png)
2. 按一下管線執行的 [錯誤] 連結以查看錯誤的詳細資料。 

    ![管線錯誤](./media/tutorial-control-flow-portal/pipeline-error-message.png)
2. 若要檢視與此管線執行相關聯的**活動執行**，請按一下 [動作] 資料行中的第一個連結。 使用 [重新整理] 按鈕可重新整理清單。 請注意，管線中的複製活動失敗了。 Web 活動成功地將失敗電子郵件傳送給指定收件者。 

    ![活動執行](./media/tutorial-control-flow-portal/activity-runs-failure.png)
4. 按一下 [動作] 資料行中的 [錯誤] 連結，以查看錯誤的詳細資料。 

    ![活動執行錯誤](./media/tutorial-control-flow-portal/activity-run-error.png)

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已執行下列步驟： 

> [!div class="checklist"]
> * 建立資料處理站。
> * 建立 Azure 儲存體連結服務。
> * 建立 Azure Blob 資料集
> * 建立包含複製活動和網路活動的管道
> * 將活動的輸出傳送至後續的活動
> * 利用參數傳遞和系統變數
> * 啟動管道執行
> * 監視管道和活動執行

您現在可以繼續閱讀＜概念＞一節，以了解 Azure Data Factory 的詳細資訊。
> [!div class="nextstepaction"]
>[管線和活動](concepts-pipelines-activities.md)
