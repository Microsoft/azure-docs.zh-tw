---
title: 在 Azure Data Factory 中使用 Databricks Notebook 活動執行 Databricks Notebook
description: 了解如何在 Azure data factory 中使用 Databricks Notebook 活動，針對 databricks 作業叢集執行 Databricks Notebook。
services: data-factory
documentationcenter: ''
author: nabhishek
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/12/2018
ms.author: abnarain
ms.reviewer: douglasl
ms.openlocfilehash: 6b0a4b7a8b2a30b9572ecfc488e2af7554b46346
ms.sourcegitcommit: 7bc4a872c170e3416052c87287391bc7adbf84ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/02/2018
ms.locfileid: "48017733"
---
# <a name="run-a-databricks-notebook-with-the-databricks-notebook-activity-in-azure-data-factory"></a>在 Azure Data Factory 中使用 Databricks Notebook 活動執行 Databricks Notebook

在本教學課程中，您會使用 Azure 入口網站來建立 Azure Data Factory 管線，以便針對 databricks 作業叢集執行 Databricks Notebook。 它也會在執行期間將 Azure Data Factory 參數傳遞至 Databricks Notebook。

您會在本教學課程中執行下列步驟：

  - 建立資料處理站。

  - 建立使用 Databricks Notebook 活動的管線。

  - 觸發管線執行。

  - 監視管道執行。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/) 。

如需此功能的簡介與示範，請觀看下列 11 分鐘長的影片：

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/ingest-prepare-and-transform-using-azure-databricks-and-data-factory/player]

## <a name="prerequisites"></a>必要條件

  - **Azure Databricks 工作區**。 [建立 Databricks 工作區](https://docs.microsoft.com/azure/azure-databricks/quickstart-create-databricks-workspace-portal)或使用現有的工作區。 您可在 Azure Databricks 工作區中建立 Python Notebook。 然後執行 Notebook，並使用 Azure Data Factory 將參數傳遞給該 Notebook。

## <a name="create-a-data-factory"></a>建立 Data Factory

1.  啟動 **Microsoft Edge** 或 **Google Chrome** 網頁瀏覽器。 目前，只有 Microsoft Edge 和 Google Chrome 網頁瀏覽器支援 Data Factory UI。

1.  選取左側功能表上的 [建立資源]、選取 [分析]，然後選取 [資料處理站]。

    ![建立新的資料處理站](media/transform-data-using-databricks-notebook/new-azure-data-factory-menu.png)

1.  在 [新增資料處理站] 窗格的 [名稱] 下，輸入 **ADFTutorialDataFactory**。

    Azure Data Factory 的名稱必須是 *全域唯一的*。 如果您看到下列錯誤，請變更資料處理站的名稱。 (例如，使用 **\<yourname\>ADFTutorialDataFactory**)。 如需 Data Factory 成品的命名規則，請參閱 [Data Factory - 命名規則](https://docs.microsoft.com/azure/data-factory/naming-rules)一文。

    ![提供新資料處理站的名稱](media/transform-data-using-databricks-notebook/new-azure-data-factory.png)

1.  針對 [訂用帳戶]，選取您要用來建立資料處理站的 Azure 訂用帳戶。

1.  針對 [資源群組]，採取下列其中一個步驟︰
    
    - 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。
    
    - 選取 [建立新的] ，然後輸入資源群組的名稱。

    本快速入門的某些步驟是假設您使用 **ADFTutorialResourceGroup** 作為資源群組名稱。 若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)。

1.  針對 [版本]，選取 [V2]。

1.  針對 [位置]，選取資料處理站的位置。

    如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析] 以找出 [Data Factory]：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 Data Factory 所使用的資料存放區 (Azure 儲存體和 Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。
1.  選取 [建立] 。


1.  建立完成之後，您會看到 [Data Factory] 頁面。 選取 [編寫與監視] 圖格，以在個別的索引標籤上啟動 Data Factory UI 應用程式。

    ![啟動資料處理站 UI 應用程式](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image4.png)

## <a name="create-linked-services"></a>建立連結的服務

在本節中，您會撰寫 Databricks 連結服務。 此連結服務包含 Databricks 叢集的連線資訊：

### <a name="create-an-azure-databricks-linked-service"></a>建立 Azure Databricks 連結服務

1.  在 [現在就開始吧] 頁面上，切換至左面板中的 [編輯] 索引標籤。

    ![編輯新的連結服務](media/transform-data-using-databricks-notebook/get-started-page.png)

1.  選取視窗底部的 [連線]，然後選取 [+ 新增]。
    
    ![建立新的連線](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image6.png)

1.  在 [新增連結服務] 視窗中，選取 [計算] \> [Azure Databricks]，然後選取 [繼續]。
    
    ![指定 Databricks 連結服務](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image7.png)

1.  在 [新增連結服務] 視窗中，完成下列步驟：
    
    1.  針對 [名稱]，輸入 ***AzureDatabricks\_LinkedService***
    
    1.  選取您要在其中執行 Notebook 的適當 **Databricks 工作區**

    1.  針對 [選取叢集]，選取 [新增作業叢集]
    
    1.  針對 [網域/區域]，資訊應該會自動填入

    1.  針對 [存取權杖]，從 Azure Databricks 工作區產生它。 您可以在[這裡](https://docs.databricks.com/api/latest/authentication.html#generate-token)找到步驟。

    1.  針對 [叢集版本]，選取 [4.0] (含 Apache Spark 2.3.0、Scala 2.11)

    1.  針對 [叢集節點類型]，請為本教學課程選取 [一般用途 (HDD)] 分類下的 [標準\_D3\_v2]。 
    
    1.  針對 [背景工作角色]，輸入 **2**。
    
    1.  選取 [完成]。

        ![完成連結服務建立](media/transform-data-using-databricks-notebook/new-databricks-linkedservice.png)

## <a name="create-a-pipeline"></a>建立管線

1.  選取 **+** (加號) 按鈕，然後選取功能表上的 [管線]。

    ![用於建立新管線的按鈕](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image9.png)

1.  建立要用於 [管線] 的**參數**。 稍後您會將此參數傳遞給 Databricks Notebook 活動。 在空的管線中，按一下 [參數] 索引標籤，然後按一下 [新增] 並將它命名為 '**name**'。

    ![建立新的參數](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image10.png)

    ![建立名稱參數](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image11.png)

1.  在 [活動] 工具箱中，展開 [Databricks]。 將 [Notebook] 工具箱中的 [Spark] 活動拖到管線設計工具介面。

    ![將 Notebook 拖曳至設計工具介面](media/transform-data-using-databricks-notebook/new-adf-pipeline.png)

1.  在 [Databricks Notebook] 活動視窗底部的屬性中，完成下列步驟：

    a. 切換至 [Azure Databricks] 索引標籤。

    b. 選取 **AzureDatabricks\_LinkedService** (已在上一個程序中建立)。

    c. 切換至 [設定] 索引標籤

    c. 瀏覽以選取 Databricks **Notebook 路徑**。 讓我們建立 Notebook 並在此指定路徑。 您可以依照下列步驟取得 Notebook 路徑。

       1. 啟動您的 Azure Databricks 工作區

       1. 在工作區中建立**新資料夾**，並將它稱為 **adftutorial**。

          ![建立新的資料夾](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image13.png)

       1. [建立新的 Notebook](https://docs.databricks.com/user-guide/notebooks/index.html#creating-a-notebook) (Python)，在 **adftutorial** 資料夾下將它稱為 **mynotebook**，按一下 [建立]。

          ![建立新的 Notebook](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image14.png)

          ![設定新 Notebook 的屬性](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image15.png)

       1. 在新建立的 Notebook "mynotebook" 中，新增下列程式碼：

           ```
           # Creating widgets for leveraging parameters, and printing the parameters

           dbutils.widgets.text("input", "","")
           dbutils.widgets.get("input")
           y = getArgument("input")
           print "Param -\'input':"
           print y
           ```

           ![建立參數的小工具](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image16.png)

       1. 在此例中，[Notebook 路徑] 是 **/adftutorial/mynotebook**

1.  切換回 **Data Factory UI 撰寫工具**。 瀏覽至 [Notebook1 活動] 之下的 [設定] 索引標籤。 
    
    a.  [新增參數] 至 Notebook 活動。 您可使用先前新增至 [管線] 的相同參數。

       ![新增參數](media/transform-data-using-databricks-notebook/new-adf-parameters.png)

    b.  將參數命名為 **input**，並提供此值作為 **@pipeline().parameters.name** 運算式。

1.  若要驗證管線，選取工具列上的 [驗證] 按鈕。 若要關閉驗證視窗，請選取 **\>\>** (向右箭頭) 按鈕。

    ![驗證管線](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image18.png)

1.  選取 [全部發佈]。 Data Factory UI 會將實體 (連結服務和管線) 發佈至 Azure Data Factory 服務。

    ![發佈新的資料處理站實體](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image19.png)

## <a name="trigger-a-pipeline-run"></a>觸發管線執行

選取工具列上的 [觸發程序]，然後選取 [立即觸發]。

![選取 [立即觸發] 命令](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image20.png)

[管線執行] 對話方塊會要求 **name** 參數。 在此使用 **/path/filename** 作為參數。 按一下 [完成]。

![提供 name 參數的值。](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image21.png)

## <a name="monitor-the-pipeline-run"></a>監視管道執行

1.  切換至 [監視] 索引標籤。確認您看到管線執行。 大約需要 5-8 分鐘的時間建立 Databricks 作業叢集 (執行 Notebook 的位置)。

    ![監視管線](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image22.png)

1.  定期選取 [重新整理] 以檢查管線執行的狀態。

1.  若要檢視與管線執行相關聯的活動執行，請選取 [動作] 資料行中的 [檢視活動執行]。

    ![檢視活動執行](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image23.png)

您可以選取頂端的 [管線] 連結，切換回管線執行檢視。

## <a name="verify-the-output"></a>驗證輸出

您可以登入 **Azure Databricks 工作區**，移至 [叢集] 並可看到 [作業] 狀態為「暫止執行」、「執行中」或「已終止」。

![檢視作業叢集和作業](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image24.png)

您可以按一下**作業名稱**並瀏覽以查看進一步的詳細資料。 對於成功的執行，您可以驗證所傳遞的參數和 Python Notebook 的輸出。

![檢視執行詳細資料和輸出](media/transform-data-using-databricks-notebook/databricks-output.png)

## <a name="next-steps"></a>後續步驟

此範例中的管線會觸發 Databricks Notebook 活動並將參數傳遞給它。 您已了解如何︰

  - 建立資料處理站。

  - 建立使用 Databricks Notebook 活動的管線。

  - 觸發管線執行。

  - 監視管道執行。
