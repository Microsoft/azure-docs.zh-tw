---
title: 在 Azure Machine Learning 服務 (預覽) 中準備鳶尾花分類教學課程所需的資料 | Microsoft Docs
description: 這個完整的教學課程將說明如何使用端對端 Azure Machine Learning 服務 (預覽)。 這是第一個部分，會討論資料準備工作。
services: machine-learning
author: hning86
ms.author: haining
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs, gcampanella
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc
ms.topic: tutorial
ms.date: 3/7/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 272b8250a80fee42780311dec92f6d47c221c160
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990156"
---
# <a name="tutorial-1-classify-iris---preparing-the-data"></a>教學課程 1：分類鳶尾花 - 準備資料

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

Azure Machine Learning 服務 (預覽) 是一套整合的端對端資料科學以及進階分析解決方案，可供專業資料科學家以雲端規模準備資料、開發測試以及部署模型。

本教學課程是**三部分系列的第一部分**。 在本教學課程中，您可逐步了解 Azure Machine Learning 服務 (預覽) 的基本概念，以及了解如何：

> [!div class="checklist"]
> * 在 Azure Machine Learning Workbench 中建立專案
> * 建立資料準備封裝
> * 產生 Python/PySpark 程式碼來叫用資料準備套件

本教學課程使用不受時間影響的[鳶尾花資料集](https://en.wikipedia.org/wiki/Iris_flower_data_set)。 

[!INCLUDE [aml-preview-note](../../../includes/aml-preview-note.md)]

## <a name="prerequisites"></a>必要條件

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

若要完成本教學課程，您必須具備：
- Azure Machine Learning 測試帳戶
- 已安裝 Azure Machine Learning Workbench

如果您尚未備妥上述必要條件，請遵循[快速入門：安裝和啟動](quickstart-installation.md)一文中的指示來設定此帳戶，以及安裝 Azure Machine Learning Workbench 應用程式。 

## <a name="create-a-new-project-in-workbench"></a>在 Workbench 中建立新的專案

如果您遵循了[快速入門：安裝和啟動](quickstart-installation.md)一文中的步驟，您應該已經有這個專案，所以可以跳至下一節。

1. 開啟 Azure Machine Learning Workbench 應用程式，並登入 (如果需要)。 
   
   + 在 Windows 上，使用 **Machine Learning Workbench** 桌面捷徑來啟動它。 
   + 在 macOS 上，選取啟動列中的 **Azure ML Workbench**。

1. 在 [專案] 窗格中選取加號 (+)，然後選擇 [新增專案]。  

   ![新增工作區](./media/tutorial-classifying-iris/new_ws.png)

1. 填妥表單欄位並選取 [建立] 按鈕，以在 Workbench 中建立新專案。

   欄位|教學課程的建議值|說明
   ---|---|---
   專案名稱 | myIris |輸入可識別您帳戶的唯一名稱。 您可以使用您自己的名稱，或最能識別測試的部門或專案名稱。 這個名稱長度應介於 2 到 32 個字元之間。 應該只包含英數字元及虛線 (-) 字元。 
   專案目錄 | c:\Temp\ | 指定要在其中建立專案的目錄。
   專案描述 | _保留空白_ | 適合用於描述專案的選擇性欄位。
   Visualstudio.com GIT 存放庫 URL |_保留空白_ | 選擇性欄位。 您可以在 Azure DevOps 上讓專案與 Git 存放庫相關聯，以便進行原始檔控制和共同作業。 [了解如何進行設定](https://docs.microsoft.com/azure/machine-learning/desktop-workbench/using-git-ml-project#step-3-set-up-a-machine-learning-project-and-git-repo)。 
   選取的工作區 | IrisGarden (如果存在的話) | 選擇您在 Azure 入口網站中針對測試帳戶建立的工作區。 <br/>如果您已遵循快速入門，您應該有名稱為 IrisGarden 的工作區。 如果沒有，請選取您建立測試帳戶時所建立的工作區，或任何您想使用的其他工作區。
   專案範本 | 分類鳶尾花 | 範本包含您可用來瀏覽產品的指令碼和資料。 此範本包含您在本快速入門及此文件網站中的其他教學課程中需要使用的指令碼和資料。 

   ![新增專案](media/tutorial-classifying-iris/new_project.png)
 
 建立新的專案，專案儀表板隨即開啟並顯示該專案。 此時，您可以瀏覽專案首頁、資料來源、筆記本及原始程式檔。 

   ![開啟專案](media/tutorial-classifying-iris/project-open.png)
 

## <a name="create-a-data-preparation-package"></a>建立資料準備封裝

接下來，您可以在 Azure Machine Learning Workbench 中瀏覽並開始準備資料。 您在 Workbench 中執行的每個轉換都會以 JSON 格式儲存在本機資料準備套件 (*.dprep 檔案) 中。 此資料準備套件是您的資料準備工作在 Workbench 中的主要容器。

此資料準備套件稍後可遞交給執行階段，例如 local-C#/CoreCLR、Scala/Spark 或 Scala/HDI。 

1. 選取資料夾圖示，即可開啟 [檔案] 檢視，然後選取 **iris.csv** 來開啟該檔案。

   此檔案包含具有 5 個資料行和 50 個資料列的資料表。 四個資料行是數值特徵資料行。 第五個資料行是字串目標資料行。 所有資料行都沒有標頭名稱。

   ![iris.csv](media/tutorial-classifying-iris/show_iris_csv.png)

   >[!NOTE]
   > 請勿在您的專案資料夾中包含資料檔案，特別是當檔案大小很大時。 因為 **iris.csv**，資料檔案很小，因此包含在此範本中以供示範。 如需詳細資訊，請參閱[如何讀取及寫入大型資料檔案](how-to-read-write-files.md)。

2. 在 [資料檢視] 中，選取加號 (**+**) 來新增新的資料來源。 [新增資料來源] 頁面隨即開啟。 

   ![Azure Machine Learning Workbench 中的資料檢視](media/tutorial-classifying-iris/data_view.png)

3. 選取 [文字檔] **(\*.csv、\*.json、\*.txt 等)**，然後按 [下一步]。
   ![Azure Machine Learning Workbench 中的資料來源](media/tutorial-classifying-iris/data-source.png)

4. 瀏覽至 **iris.csv** 檔案，然後按一下 [完成]。 這會使用參數的預設值，例如分隔符號和資料類型。

   >[!IMPORTANT]
   >針對此練習，請確定您從目前的專案目錄內選取 **iris.csv** 檔案。 否則，接下來的步驟可能會失敗。
 
   ![選取鳶尾花](media/tutorial-classifying-iris/select_iris_csv.png)
   
5. 名為 **iris-1.dsource** 的新檔案隨即建立。 由於範例專案中隨附未編號的 **iris.dsource** 檔案，因此會以 "-1" 為檔案指定唯一的名稱。  

   檔案隨即開啟，並且顯示資料。 從 **Column1** 至 **Column5** 的一系列資料行標題，會自動新增到此資料集。 捲動到底部，並注意資料集的最後一個資料列是空的。 資料列是空的，因為 CSV 檔案中有額外的分行符號。

   ![鳶尾花資料檢視](media/tutorial-classifying-iris/iris_data_view.png)

1. 選取 [計量] 按鈕。 隨即產生並顯示長條圖。

   選取 [資料] 按鈕，即可切換回到資料檢視。
   
   ![鳶尾花資料檢視](media/tutorial-classifying-iris/iris_data_view_metrics.png)

1. 觀察長條圖。 每個資料行會計算出一組完整的統計資料。 

   ![鳶尾花資料檢視](media/tutorial-classifying-iris/iris_metrics_view.png)

8. 選取 [準備] 按鈕，開始建立資料準備套件。 [準備] 對話方塊隨即開啟。 

   此範例專案包含預設選取的 **iris.dprep** 資料準備檔案。 

   ![鳶尾花資料檢視](media/tutorial-classifying-iris/prepare.png)

1. 從下拉功能表中選取 [+ 新資料準備套件]，以建立新的資料準備套件。

   ![鳶尾花資料檢視](media/tutorial-classifying-iris/prepare_new.png)

1. 輸入新的值作為套件名稱 (使用 **iris-1**)，然後選取 [確定]。

   名為 **iris-1.dprep** 的新資料準備套件隨即建立，並在資料準備編輯器中開啟。

   ![鳶尾花資料檢視](media/tutorial-classifying-iris/prepare_iris-1.png)

   現在，讓我們進行一些基本的資料準備動作。 

1. 選取各個資料行標頭，讓標頭文字可進行編輯。 然後，將每個資料行重新命名，如下所示： 

   分別針對五個資料行，依序輸入 [萼片長度]、[萼片寬度]、[花瓣長度]、[花瓣寬度] 和 [物種]。

   ![將資料行重新命名](media/tutorial-classifying-iris/rename_column.png)

1. 計算相異值：
   1. 選取 [物種] 資料行。
   1. 按一下滑鼠右鍵加以選取。 
   1. 從下拉式功能表中選取 [值計數]。 

   [偵測器] 窗格會在資料下方開啟。 此時會出現有四個橫條的長條圖。 目標資料行有四個相異值：**Iris-virginica**、**Iris-versicolor**、**Iris-setosa** 和 **(null)** 值。

   ![選取值計數](media/tutorial-classifying-iris/value_count.png)

   ![值計數長條圖](media/tutorial-classifying-iris/filter_out.png)

1. 若要篩選出 null 值，請選取 [(null)] 長條，然後選取減號 (**-**)。 

   接著，(null) 資料列會變成灰色，表示它已被篩選掉。 

   ![篩選掉 null](media/tutorial-classifying-iris/filter_out2.png)

1. 請注意 [步驟] 窗格中詳述的個別資料準備步驟。 當您將資料行重新命名並篩選 null 值資料列時，每個動作都會記錄為資料準備步驟。 您可以編輯個別的步驟，以調整其設定、重新排列步驟，並移除步驟。

   ![步驟](media/tutorial-classifying-iris/steps.png)

1. 關閉資料準備編輯器。 在包含圖形圖示的 **iris-1** 索引標籤上，選取 **x** 圖示以關閉索引標籤。您的工作會自動儲存到顯示在 [資料準備] 標題之下的 **iris-1.dprep** 檔案中。

   ![關閉](media/tutorial-classifying-iris/close.png)

## <a name="generate-pythonpyspark-code-to-invoke-a-data-preparation-package"></a>產生 Python/PySpark 程式碼來叫用資料準備套件

 您可直接在 Python 或 Jupyter Notebook 中探索資料準備套件的輸出。 套件可以跨多個執行階段執行，包括本機 Python、Spark (包含於 Docker 中) 及 HDInsight。 

1. 在 [資料準備] 索引標籤下尋找 **iris-1.dprep** 檔案。

1. 以滑鼠右鍵按一下 **iris-1.dprep** 檔案，然後從快顯功能表中選取 [產生資料存取碼檔案]。 

   ![產生程式碼](media/tutorial-classifying-iris/generate_code.png)

   名為 **iris-1.py** 的新檔案隨即開啟，其中包含下列幾行程式碼，以便叫用您建立為資料準備套件的邏輯：

   ```python
   # Use the Azure Machine Learning data preparation package
   from azureml.dataprep import package

   # Use the Azure Machine Learning data collector to log various metrics
   from azureml.logging import get_azureml_logger
   logger = get_azureml_logger()

   # This call will load the referenced package and return a DataFrame.
   # If run in a PySpark environment, this call returns a
   # Spark DataFrame. If not, it will return a Pandas DataFrame.
   df = package.run('iris-1.dprep', dataflow_idx=0)

   # Remove this line and add code that uses the DataFrame
   df.head(10)
   ```

   視執行此程式碼所在的內容而定，`df` 代表不同種類的 DataFrame：
   + 在 Python 執行階段上執行時，會使用 [pandas DataFrame](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html)。
   + 在 Spark 內容中執行時，則會使用 [Spark 資料框架](https://spark.apache.org/docs/latest/sql-programming-guide.html)。 
   
   若要深入了解如何在 Azure Machine Learning Workbench 中準備資料，請參閱[開始進行資料準備](data-prep-getting-started.md)指南。

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [aml-delete-resource-group](../../../includes/aml-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

在本教學課程中，您使用了 Azure Machine Learning Workbench 來：
> [!div class="checklist"]
> * 建立新專案
> * 建立資料準備封裝
> * 產生 Python/PySpark 程式碼來叫用資料準備套件

您已準備好進行本教學課程系列中的下一個部分，將了解如何建置 Azure Machine Learning 模型：
> [!div class="nextstepaction"]
> [教學課程 2 - 建置模型](tutorial-classifying-iris-part-2.md)
