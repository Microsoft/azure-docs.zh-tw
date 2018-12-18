---
title: 建置 Azure Machine Learning 服務 (預覽) 的模型教學課程 | Microsoft Docs
description: 這個完整的教學課程將說明如何使用端對端 Azure Machine Learning 服務 (預覽)。 這是第二部分，會討論測試。
services: machine-learning
author: hning86
ms.author: haining
manager: mwinkle
ms.reviewer: jmartens
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc
ms.topic: tutorial
ms.date: 3/15/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 6faabc3003048c93c08451c537496e762352b50b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46997301"
---
# <a name="tutorial-2-classify-iris---build-a-model"></a>教學課程 2：分類鳶尾花：建立模型

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

Azure Machine Learning 服務 (預覽) 是一套整合式資料科學及進階分析解決方案，可供專業資料科學家用來以雲端規模準備資料、開發測試以及部署模型。

本教學課程是**三部分系列的第二部分**。 在本教學課程中，您可以使用 Azure Machine Learning 服務來：

> [!div class="checklist"]
> * 開啟指令碼並檢閱程式碼
> * 在本機環境中執行指令碼
> * 檢閱執行歷程記錄
> * 在本機 Azure CLI 視窗中執行指令碼
> * 在本機 Docker 環境中執行指令碼
> * 在遠端 Docker 環境中執行指令碼
> * 在雲端 Azure HDInsight 環境中執行指令碼

本教學課程使用不受時間影響的[鳶尾花資料集](https://en.wikipedia.org/wiki/Iris_flower_data_set)。 

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您需要：
- Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。 
- 如本[快速入門](quickstart-installation.md)所述安裝的測試帳戶和 Azure Machine Learning Workbench。
- [教學課程第 1 部分](tutorial-classifying-iris-part-1.md)中的專案和備妥鳶尾花資料
- 在本機安裝並執行的 Docker 引擎。 Docker 的 Community 版本就足夠了。 了解如何在此安裝 Docker： https://docs.docker.com/engine/installation/。

## <a name="review-irissklearnpy-and-the-configuration-files"></a>檢閱 iris_sklearn.py 和組態檔

1. 啟動 Azure Machine Learning Workbench 應用程式。

1. 開啟您在[本教學課程系列的第 1 部分](tutorial-classifying-iris-part-1.md)中建立的 **myIris** 專案。

2. 在開啟的專案中，選取最左邊窗格上的 [檔案] 按鈕 (資料夾圖示)，以在您的專案資料夾中開啟檔案清單。

   ![開啟 Azure Machine Learning Workbench 專案](media/tutorial-classifying-iris/2-project-open.png)

3. 選取 **iris_sklearn.py** Python 指令檔。 

   ![選擇指令碼](media/tutorial-classifying-iris/2-choose-iris_sklearn.png)

   程式碼隨即在 Workbench 內的新文字編輯器索引標籤中開啟。 這是您在本教學課程的這一部分中使用的指令碼。 

   >[!NOTE]
   >因為此範例專案會經常更新，您看到的程式碼可能不會與上述程式碼完全相同。
   
   ![開啟檔案](media/tutorial-classifying-iris/open_iris_sklearn.png)

4. 檢查 Python 指令碼，以熟悉程式碼撰寫樣式。 

   **iris_sklearn.py** 指令碼會執行下列工作：

   * 載入名為 **iris.dprep** 的預設資料準備套件，以建立 [pandas DataFrame](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html)。 

   * 新增隨機功能，讓問題更難解決。 隨機性是必要的，因為鳶尾花是能以接近 100% 的精確度輕鬆地分類的小型資料集。

   * 使用 [scikit-learn](http://scikit-learn.org/stable/index.html) 機器學習程式庫來建置羅吉斯迴歸模型。  根據預設，此程式庫隨附 Azure Machine Learning Workbench。

   * 將模型序列化，方法為在 `outputs` 資料夾的檔案中使用 [pickle](https://docs.python.org/3/library/pickle.html) 程式庫。 
   
   * 載入序列化模型，然後將它還原序列化回到記憶體。

   * 使用已還原序列化的模型來對新記錄進行預測。 

   * 繪製兩個圖表、混淆矩陣和多級接收者作業特性 (ROC) 曲線，方法為使用 [matplotlib](https://matplotlib.org/) 程式庫，然後將它們儲存在 `outputs` 資料夾中。 如果沒有此程式庫，您可以在您的環境中加以安裝。

   * 自動在執行歷程記錄中繪製正規化比率和模型精確度。 系統會使用整個 `run_logger` 物件來記錄正規化速率並將精確度制定到記錄內。 


## <a name="run-irissklearnpy-in-your-local-environment"></a>在本機環境中執行 iris_sklearn.py

1. 啟動 Azure Machine Learning 命令列介面 (CLI)：
   1. 啟動 Azure Machine Learning Workbench。

   1. 從 [Workbench] 功能表，選取 [檔案] > [開啟命令提示字元]。 
   
   Azure Machine Learning 命令列介面 (CLI) 視窗會從 Windows 上的專案資料夾 `C:\Temp\myIris\>` 開始。 這個專案與您在本教學課程的第 1 部分中建立的專案相同。

   >[!IMPORTANT]
   >您必須使用此 CLI 視窗來完成下一個步驟。

1. 在 CLI 視窗中，安裝 Python 繪圖程式庫 **matplotlib** (如果您還沒有此程式庫)。

   **Iris_sklearn.py** 指令碼具有兩個 Python 套件的相依性：**scikit-learn** 和 **matplotlib**。  **scikit-learn** 套件是由 Azure Machine Learning Workbench 安裝，方便您使用。 不過，如果您尚未安裝 **matplotlib**，則必須加以安裝。

   如果您在未安裝 **matplotlib** 的情況下繼續進行，本教學課程中的程式碼仍可順利執行。 不過，程式碼就不會產生混淆矩陣輸出和歷程記錄視覺效果中所示的多類別 ROC 曲線圖。

   ```azurecli
   pip install matplotlib
   ```

   此安裝大約需要一分鐘的時間。

1. 返回 Workbench 應用程式。 

1. 尋找名為 **iris_sklearn.py** 的索引標籤。 

   ![尋找含指令碼的索引標籤](media/tutorial-classifying-iris/2-iris_sklearn-tab.png)

1. 在該索引標籤的工具列中，選取 **local**作為執行環境，以及選取 `iris_sklearn.py` 作為要執行的指令碼。 可能已選取這些項目。

   ![本機和指令碼選擇](media/tutorial-classifying-iris/2-local-script.png)

1. 移至工具列的右邊，然後在 [引數] 欄位中輸入 `0.01`。 

   這個值對應於羅吉斯迴歸模型的正則化速率。

   ![本機和指令碼選擇](media/tutorial-classifying-iris/2-local-script-arguments.png)

1. 選取 [執行] 按鈕。 立即會排定作業。 作業會列在 Workbench 視窗右邊的 [作業] 窗格。 

   ![本機和指令碼選擇](media/tutorial-classifying-iris/2-local-script-arguments-run.png)

   在幾分鐘之後，作業的狀態會從 [提交中] 轉換為 [執行中]，最後變成 [已完成]。

1. 選取 [作業] 窗格的作業狀態文字中的 [已完成]。 

   ![執行 sklearn](media/tutorial-classifying-iris/2-completed.png)

   快顯視窗隨即會開啟，並顯示執行的標準輸出 (stdout) 文字。 若要關閉 stdout 文字，請選取快顯視窗上右上方的 [關閉] \(**x**) 按鈕。

   ![標準輸出](media/tutorial-classifying-iris/2-standard-output.png)

9. 在 [作業] 窗格的相同作業狀態中，選取 [已完成] 狀態和開始時間正上方的藍色文字 **iris_sklearn.py [n]** (_n_ 是執行編號)。 [執行屬性] 視窗隨即開啟，並顯示下列該特定執行的資訊：
   - [執行屬性] 資訊
   - **輸出**
   - **計量**
   - **視覺效果**，若有的話
   - **記錄檔** 

   執行完成時，快顯視窗會顯示下列結果：

   >[!NOTE]
   >因為本教學課程稍早已對訓練集引入了一些隨機化項目，確切的結果可能與如下所示的有所不同。

   ```text
   Python version: 3.5.2 |Continuum Analytics, Inc.| (default, Jul  5 2016, 11:41:13) [MSC v.1900 64 bit (AMD64)]
   
   Iris dataset shape: (150, 5)
   Regularization rate is 0.01
   LogisticRegression(C=100.0, class_weight=None, dual=False, fit_intercept=True,
          intercept_scaling=1, max_iter=100, multi_class='ovr', n_jobs=1,
          penalty='l2', random_state=None, solver='liblinear', tol=0.0001,
          verbose=0, warm_start=False)
   Accuracy is 0.6792452830188679
   
   ==========================================
   Serialize and deserialize using the outputs folder.
   
   Export the model to model.pkl
   Import the model from model.pkl
   New sample: [[3.0, 3.6, 1.3, 0.25]]
   Predicted class is ['Iris-setosa']
   Plotting confusion matrix...
   Confusion matrix in text:
   [[50  0  0]
    [ 1 37 12]
    [ 0  4 46]]
   Confusion matrix plotted.
   Plotting ROC curve....
   ROC curve plotted.
   Confusion matrix and ROC curve plotted. See them in Run History details pane.
   ```
    
10. 關閉 [執行屬性] 索引標籤，然後返回 **iris_sklearn.py** 索引標籤。 

11. 針對其他執行重複進行。 

    在範圍從 `0.001` 至 `10` 的 [引數] 欄位中輸入一系列的值。 選取 [執行] 來執行程式碼數次。 您每次變更的引數值會在程式碼中傳送至羅吉斯迴歸模型，每次產生不同的結果。

## <a name="review-the-run-history-in-detail"></a>檢閱詳細的執行歷程記錄
在 Azure Machine Learning Workbench 中，每個指令碼執行都會以執行歷程記錄的形式擷取。 如果您開啟 [執行] 檢視，就可以檢視特定指令碼的執行歷程記錄。

1. 若要開啟 [執行] 的清單，請選取左邊工具列上的 [執行] 按鈕 (時鐘圖示)。 然後選取 **iris_sklearn.py** 以顯示 `iris_sklearn.py` 的 [執行儀表板]。

   ![執行檢視](media/tutorial-classifying-iris/run_view.png)

1. [執行儀表板] 索引標籤隨即開啟。 

   檢閱跨多個執行擷取的統計資料。 索引標籤頂端中的圖形轉譯。每次執行都有連續的數字，且執行詳細資料會列在畫面底部的資料表中。

   ![執行儀表板](media/tutorial-classifying-iris/run_dashboard.png)

1. 篩選資料表，然後選取任何圖表來檢視每個執行的狀態、持續時間、精確度，以及正規化速率。 

1. 選取 [執行] 資料表中兩個或更多執行旁邊的核取方塊。 選取 [比較] 按鈕來開啟詳細的比較窗格。 檢閱並排比較。 

1. 若要返回 [執行儀表板]，選取 [比較] 窗格左上方的 [執行清單] 返回按鈕。

   ![返回執行清單](media/tutorial-classifying-iris/2-compare-back.png)

1. 選取個別執行以查看執行詳細資料檢視。 請注意，所選取執行的統計資料會列在 [執行屬性] 區段中。 寫入到輸出資料夾的檔案會列於 [輸出] 區段，而且您可以從該處下載檔案。

   ![執行詳細資料](media/tutorial-classifying-iris/run_details.png)

   兩個繪圖，混淆矩陣和多類別 ROC 曲線，會在**視覺效果**區段中轉譯。 所有記錄檔也都可在 **Logs** 區段找到。


## <a name="run-scripts-in-local-docker-environments"></a>在本機 Docker 環境中執行指令碼

(選擇性) 您可以試驗針對本機 Docker 容器來執行指令碼。 您可以設定額外的執行環境，例如 Docker，並在這些環境中執行指令碼。 

>[!NOTE]
>如需實驗分派指令碼以在遠端 Azure VM 的 Docker 容器中執行或在 Azure HDInsight Spark 叢集中執行，您可以依照[建立 Ubuntu 型 Azure 資料科學虛擬機器或 HDInsight 叢集的指示](how-to-create-dsvm-hdi.md)來進行。

1. 如果您尚未這麼做，請在 Windows 或 MacOS 電腦本機上安裝和啟動 Docker。 如需詳細資訊，請參閱位於 https://docs.docker.com/install/ 的 Docker 安裝指示。 Community 版本就足夠了。

1. 在左邊窗格上，選取**資料夾**圖示，即可開啟您專案的 [檔案] 清單。 展開 `aml_config` 資料夾。 

2. 有預先設定的數個環境：**docker-python**、**docker-spark** 和 **local**。 

   每個環境都有兩個檔案，例如 `docker.compute` (適用於 **docker-python** 與 **docker-spark**) 和 `docker-python.runconfig`。 開啟每一個檔案，可看到某些選項可以在文字編輯器中設定。  

   若要進行清理，請選取任何所開啟文字編輯器之索引標籤上的 [關閉] \(**x**) 索引標籤。

3. 使用 **docker-python** 環境執行 **iris_sklearn.py** 指令碼： 

   - 在左邊工具列中，選取**時鐘**圖示以開啟 [執行] 窗格。 選取 [所有執行]。 

   - 在 [所有執行] 索引標籤的最上層，選取 **docker-python** 作為目標環境，而非預設的 **local**。 

   - 接下來，移到右邊，並選取 **iris_sklearn.py** 作為要執行的指令碼。 

   - 將 [引數] 欄位保留空白，因為指令碼會指定預設值。 

   - 選取 [執行] 按鈕。

4. 請觀察，會啟動一個新的作業。 它會出現在 Workbench 視窗右邊的 [作業] 窗格。

   當您第一次對 Docker 執行時，此作業需要多幾分鐘的時間才能完成。 

   在幕後，Azure Machine Learning Workbench 會建置新的 Docker 檔案。 
   新的檔案會參考 `docker.compute` 檔案中指定的基礎 Docker 映像，以及 Python 套件在 `conda_dependencies.yml` 檔案中指定的相依性。 
   
   Docker 引擎會執行下列工作：

    - 從 Azure 下載基礎映像。
    - 安裝 `conda_dependencies.yml` 檔案中指定的 Python 套件。
    - 啟動 Docker 容器。
    - 取決於回合組態，複製或參考專案資料夾的本機副本。      
    - 執行 `iris_sklearn.py` 指令碼。

   最後，您應該會看到與目標為 **local** 時完全相同的結果。

5. 現在讓我們來試試 Spark。 Docker 基礎映像包含預先安裝和設定的 Spark 執行個體，您可將其用於執行 PySpark 指令碼。 使用此基礎映像是開發和測試 Spark 程式的簡單方式，而不需自己花費時間安裝和設定 Spark。 

   開啟 `iris_spark.py` 檔案。 此指令碼會載入 `iris.csv` 資料檔案，並使用來自 Spark Machine Learning 程式庫的羅吉斯迴歸演算法，以分類鳶尾花資料集。 現在將執行環境變更為 **docker-spark**，以及將指令碼變更為 **iris_spark.py**，然後再次執行。 因為必須在 Docker 容器內建立並啟動 Spark 工作階段，此程序可能需要較長的時間。 您也可以看到 stdout 與 `iris_spark.py` 的 stdout 不同。

6. 多啟動幾次執行，並使用不同的引數播放。 

7. 開啟 `iris_spark.py` 檔案來查看使用 Spark Machine Learning 程式庫建置的簡單羅吉斯迴歸模型。 

8. 與 [作業] 窗格互動、執行歷程記錄清單檢視，以及執行跨不同執行環境執行的詳細資料檢視。

## <a name="run-scripts-in-the-cli-window"></a>在 CLI 視窗中執行指令碼

1. 啟動 Azure Machine Learning 命令列介面 (CLI)：
   1. 啟動 Azure Machine Learning Workbench。

   1. 從 [Workbench] 功能表，選取 [檔案] > [開啟命令提示字元]。 
   
   CLI 命令提示字元會從 Windows 上的專案資料夾 `C:\Temp\myIris\>` 開始。 這是您在本教學課程的第 1 部分中建立的專案。

   >[!IMPORTANT]
   >您必須使用此 CLI 視窗來完成下一個步驟。

1. 在 CLI 視窗中，登入 Azure。 [進一步了解 az 登入](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest)。

   您可能已經登入。 在此情況下，您可以略過此步驟。

   1. 在命令提示字元中，輸入：
      ```azurecli
      az login
      ```

      此命令會傳回要在您瀏覽器中的 https://aka.ms/devicelogin 使用的程式碼。

   1. 在瀏覽器中移至 https://aka.ms/devicelogin。

   1. 出現提示時，在瀏覽器中輸入您在 CLI 中收到的程式碼。

   對 Azure 資源進行驗證時，Workbench 應用程式和 CLI 會使用獨立的認證快取。 登入之後，在快取的權杖過期前，您都不需要再次驗證。 

1. 如果您的組織有多個 Azure 訂用帳戶 (企業環境)，您必須設定要使用的訂用帳戶。 尋找您的訂用帳戶、使用訂用帳戶識別碼設定它，然後加以測試。

   1. 使用以下命令列出您可存取的每個 Azure 訂用帳戶：
   
      ```azurecli
      az account list -o table
      ```

      **az account list** 命令會傳回可供您的登入使用的訂用帳戶。 
      如果有多個訂用帳戶，請找出您要使用之訂用帳戶的訂用帳戶識別碼值。

   1. 設定您要當作預設帳戶的 Azure 訂用帳戶：
   
      ```azurecli
      az account set -s <your-subscription-id>
      ```
      其中 \<your-subscription-id\> 是您要使用之訂用帳戶的識別碼值。 請勿包含中括弧。

   1. 要求目前訂用帳戶的詳細資料，以確認新的訂用帳戶設定。 

      ```azurecli
      az account show
      ```    

1. 在 CLI 視窗中，安裝 Python 繪圖程式庫 **matplotlib** (如果您還沒有此程式庫)。

   ```azurecli
   pip install matplotlib
   ```

1. 在 CLI 視窗中，提交 **iris_sklearn.py** 指令碼當作實驗。

   Iris_sklearn.py 執行會針對本機計算內容執行。

   + 在 Windows 上：
     ```azurecli
     az ml experiment submit -c local .\iris_sklearn.py
     ```

   + 在 MacOS 上：
     ```azurecli
     az ml experiment submit -c local iris_sklearn.py
     ```
   
   您的輸出應類似於：
    ```text
    RunId: myIris_1521077190506
    
    Executing user inputs .....
    ===========================
    
    Python version: 3.5.2 |Continuum Analytics, Inc.| (default, Jul  2 2016, 17:52:12) 
    [GCC 4.2.1 Compatible Apple LLVM 4.2 (clang-425.0.28)]
    
    Iris dataset shape: (150, 5)
    Regularization rate is 0.01
    LogisticRegression(C=100.0, class_weight=None, dual=False, fit_intercept=True,
              intercept_scaling=1, max_iter=100, multi_class='ovr', n_jobs=1,
              penalty='l2', random_state=None, solver='liblinear', tol=0.0001,
              verbose=0, warm_start=False)
    Accuracy is 0.6792452830188679
        
    ==========================================
    Serialize and deserialize using the outputs folder.
    
    Export the model to model.pkl
    Import the model from model.pkl
    New sample: [[3.0, 3.6, 1.3, 0.25]]
    Predicted class is ['Iris-setosa']
    Plotting confusion matrix...
    Confusion matrix in text:
    [[50  0  0]
     [ 1 37 12]
     [ 0  4 46]]
    Confusion matrix plotted.
    Plotting ROC curve....
    ROC curve plotted.
    Confusion matrix and ROC curve plotted. See them in Run History details page.
    
    Execution Details
    =================
    RunId: myIris_1521077190506
    ```

1. 檢閱輸出。 您的輸出和結果與您使用 Workbench 執行指令碼時的相同。 

1. 在 CLI 視窗中，請使用 Docker 執行環境再次執行 Python 指令碼 **iris_sklearn.py** (如果您已在電腦上安裝 Docker)。

   + 如果您的容器位於 Windows 上： 
     |執行<br/>Environment|Windows 上的命令|
     |---------------------|------------------|
     |Python|`az ml experiment submit -c docker-python .\iris_sklearn.py 0.01`|
     |Spark|`az ml experiment submit -c docker-spark .\iris_spark.py 0.1`|

   + 如果您的容器位於 MacOS 上： 
     |執行<br/>Environment|Windows 上的命令|
     |---------------------|------------------|
     |Python|`az ml experiment submit -c docker-python iris_sklearn.py 0.01`|
     |Spark|`az ml experiment submit -c docker-spark iris_spark.py 0.1`|

1. 返回 Workbench，然後：
   1. 選取左邊窗格上的資料夾圖示以列出專案檔案。
   
   1. 開啟名為 **run.py** 的 Python 指令碼。 此指令碼適合用於透過各種正則化速率的迴圈。 

   ![返回執行清單](media/tutorial-classifying-iris/2-runpy.png)

1. 以此比率執行實驗多次。 

   此指令碼會以 `10.0` 的正規化速率 (非常大的數字) 來啟動 `iris_sklearn.py` 作業。 接著，指令碼會在下列回合中將比率縮減為一半，並依此類推，直到比率不小於 `0.005` 為止。 

   此指令碼包含下列程式碼：

   ![返回執行清單](media/tutorial-classifying-iris/2-runpy-code.png)

1. 從命令列執行 **run.py** 指令碼，如下所示：

   ```cmd
   python run.py
   ```

   此命令會使用不同的正規化比率多次提交 iris_sklearn.py。

   當 `run.py` 完成時，您可以在 Workbench 的執行歷程記錄清單檢視中看到不同計量的圖表。

## <a name="run-scripts-in-a-remote-docker-container"></a>在遠端 Docker 容器中執行指令碼
若要在遠端 Linux 電腦上的 Docker 容器中執行指令碼，您需要具備該遠端電腦的 SSH 存取 (使用者名稱和密碼)。 此外，該電腦必須安裝並執行 Docker 引擎。 取得這類 Linux 電腦最簡單的方式，是在 Azure 上建立以 Ubuntu 為基礎的資料科學虛擬機器 (DSVM)。 深入了解[如何建立要在 Azure ML Workbench 中使用的 Ubuntu DSVM](how-to-create-dsvm-hdi.md#create-an-ubuntu-dsvm-in-azure-portal)。

>[!NOTE] 
>不支援以 CentOS 為基礎的 DSVM。

1. 建立 VM 後，產生一組 `.runconfig` 和 `.compute` 檔案，即可附加 VM 作為執行環境。 使用下列命令來產生檔案。 

 讓我們將新的計算目標命名為 `myvm`。
 
   ```azurecli
   az ml computetarget attach remotedocker --name myvm --address <your-IP> --username <your-username> --password <your-password>
   ```
   
   >[!NOTE]
   >IP 位址也可以是可公開定址的完整網域名稱 (FQDN)，例如 `vm-name.southcentralus.cloudapp.azure.com`。 絕佳的做法是將 FQDN 新增到您的 DSVM，並使用它來代替 IP 位址。 這種做法是個不錯的主意，因為您可能會在某個時間點關閉 VM 來節省成本。 此外，下次您啟動 VM 時，IP 位址可能已變更。

   >[!NOTE]
   >除了使用者名稱和密碼驗證，您可以使用 `--private-key-file` 及 (選擇性) `--private-key-passphrase` 選項來指定私密金鑰和對應的複雜密碼 (如果有的話)。 如果您想要使用在建立 DSVM 時使用的私密金鑰，您應指定 `--use-azureml-ssh-key` 選項。

   接下來，執行以下命令來準備 **myvm** 計算目標。
   
   ```azurecli
   az ml experiment prepare -c myvm
   ```
   
   上述命令會在 VM 中建構 Docker 映像，以準備好執行指令碼。
   
   >[!NOTE]
   >您也可以將 `myvm.runconfig` 中 `PrepareEnvironment` 的值從預設值 `false` 變更為 `true`。 此變更會在第一次執行時自動準備 Docker 容器。

2. 編輯在 `aml_config` 下產生的 `myvm.runconfig` 檔案，並將架構從預設值 `PySpark` 變更為 `Python`：

   ```yaml
   Framework: Python
   ```
   >[!NOTE]
   >雖然 PySpark 應該也能運作，但如果實際上不需要 Spark 工作階段來執行 Python 指令碼，則使用 Python 會有效率。

3. 發出如同您先前在 CLI 視窗中執行的相同命令，但這次使用 _myvm_ 目標在遠端 Docker 容器中執行 iris_sklearn.py：
   ```azurecli
   az ml experiment submit -c myvm iris_sklearn.py
   ```
   命令會如同您是在 `docker-python` 環境中執行，不同之處在於執行是發生在遠端 Linux VM 上。 CLI 視窗會顯示相同的輸出資訊。

4. 讓我們在容器中試試看使用 Spark。 開啟檔案總管。 建立一份 `myvm.runconfig` 檔案並將其命名為 `myvm-spark.runconfig`。 編輯新檔案，將 `Framework` 設定從 `Python` 變更為 `PySpark`：
   ```yaml
   Framework: PySpark
   ```
   不要對 `myvm.compute` 檔案進行任何變更。 會將相同 VM 上的相同 Docker 映像用於 Spark 執行。 在新 `myvm-spark.runconfig` 中，`Target` 欄位會透過其名稱 `myvm` 指向相同的 `myvm.compute` 檔案。

5. 輸入下列命令，以在遠端 Docker 容器內執行的 Spark 執行個體中執行 **iris_spark.py** 指令碼：
   ```azureli
   az ml experiment submit -c myvm-spark .\iris_spark.py
   ```

## <a name="run-scripts-in-hdinsight-clusters"></a>在 HDInsight 叢集中執行指令碼
您也可以在 HDInsight Spark 叢集中執行此指令碼。 了解[如何建立要在 Azure ML Workbench 中使用的 HDInsight Spark 叢集](how-to-create-dsvm-hdi.md#create-an-apache-spark-for-azure-hdinsight-cluster-in-azure-portal)。

>[!NOTE] 
>HDInsight 叢集必須使用 Azure Blob 作為主要儲存體。 尚未支援使用 Azure Data Lake 儲存體。

1. 如果您有適用於 Azure HDInsight 的 Spark 叢集的存取權，請如下所示產生 HDInsight 回合組態命令。 提供 HDInsight 叢集名稱和您的 HDInsight 使用者名稱與密碼作為參數。 

   使用下列命令來建立指向 HDInsight 叢集的計算目標：

   ```azurecli
   az ml computetarget attach cluster --name myhdi --address <cluster head node FQDN> --username <your-username> --password <your-password>
   ```

   若要準備 HDInsight 叢集，請執行以下命令：

   ```
   az ml experiment prepare -c myhdi
   ```

   叢集前端節點 FQDN 通常是 `<your_cluster_name>-ssh.azurehdinsight.net`。

   >[!NOTE]
   >`username` 是在 HDInsight 設定期間定義的叢集 SSH 使用者名稱。 根據預設，此值是 `sshuser`。 值不是 `admin`，它是其他使用者在設定期間建立，用來啟用對叢集管理網站的存取。 

2. 使用以下命令在 HDInsight 叢集中執行 **iris_spark.py** 指令碼：

   ```azurecli
   az ml experiment submit -c myhdi .\iris_spark.py
   ```

   >[!NOTE]
   >針對遠端 HDInsight 叢集執行時，您也可以在 `https://<your_cluster_name>.azurehdinsight.net/yarnui` 使用 `admin` 使用者帳戶檢視 Yet Another Resource Negotiator (YARN) 作業執行的詳細資料。

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [aml-delete-resource-group](../../../includes/aml-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟
在三部分教學課程系列的第二部分中，您已了解如何：
> [!div class="checklist"]
> * 開啟指令碼並且在 Workbench 中檢閱程式碼
> * 在本機環境中執行指令碼
> * 檢閱執行歷程記錄
> * 在本機 Docker 環境中執行指令碼

您現在可以嘗試本教學課程系列的第三部分，您可在其中將您建立的羅吉斯迴歸模型部署為即時 Web 服務。

> [!div class="nextstepaction"]
> [教學課程 3 - 部署模型](tutorial-classifying-iris-part-3.md)
