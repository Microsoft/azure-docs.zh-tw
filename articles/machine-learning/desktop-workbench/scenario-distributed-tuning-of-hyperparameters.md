---
title: 使用 Azure Machine Learning Workbench 的分散式超參數微調 | Microsoft Docs
description: 這個案例示範如何使用 Azure Machine Learning Workbench 進行分散式超參數微調
services: machine-learning
author: pechyony
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.author: dmpechyo
manager: mwinkle
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.date: 09/20/2017
ROBOTS: NOINDEX
ms.openlocfilehash: f74889cdf727bc132723d16df295849769001ce9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46951962"
---
# <a name="distributed-tuning-of-hyperparameters-using-azure-machine-learning-workbench"></a>使用 Azure Machine Learning Workbench 的分散式超參數微調

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



這個案例示範如何使用 Azure Machine Learning Workbench，相應放大實作 scikit-learn API 之機器學習演算法的超參數微調。 我們示範如何設定及使用遠端 Docker 容器和 Spark 叢集作為微調超參數時的執行後端。

## <a name="link-of-the-gallery-github-repository"></a>資源庫 GitHub 存放庫的連結
以下是公用 GitHub 存放庫的連結： 

[https://github.com/Azure/MachineLearningSamples-DistributedHyperParameterTuning](https://github.com/Azure/MachineLearningSamples-DistributedHyperParameterTuning)

## <a name="use-case-overview"></a>使用案例概觀

很多機器學習演算法都有一個或多個旋鈕 (Knob)，稱之為超參數。 這些旋鈕能夠微調演算法，使其對於未來資料的效能最佳化，而效能是根據使用者指定的計量 (例如精確度、AUC、RMSE) 來測量。 資料科學家必須在建立訓練資料的模型以及查看未來測試資料之前，提供超參數的值。 我們如何根據已知的訓練資料來設定超參數的值，讓模型對未知的測試資料有良好的效果？ 
    
微調超參數的常用技術為「結合交叉驗證的網格搜尋」。 交叉驗證技術可評估模型的效果，在訓練集上進行訓練，以及對測試集進行預測。 使用這項技術時，我們會先將資料集分成 K 個折，然後以循環配置資源的方式訓練演算法 K 次。 我們會對所有的折這樣做，但其中名為留出 (held-out) 的折除外。 我們可對對 K 個留出折計算 K 模型的計量平均值。 這個平均值 (名為「交叉驗證效能評估」) 取決於建立 K 模型時所用的超參數值。 微調超參數時，我們會搜尋候選超參數值的整個空間，尋找可最佳化交叉驗證效能估計值的值。 網格搜尋是一個常見的搜尋技術。 在網格搜尋中，多個超參數的候選值空間是個別超參數候選值集的交叉乘積。 

使用交叉驗證的網格搜尋可能很費時。 如果演算法有五個超參數，每個超參數有五個候選值，我們會使用 K=5 個折。 然後我們會透過訓練 5<sup>6</sup>=15625 個模型，以完成網格搜尋。 幸運的是，使用交叉驗證的網格搜尋是窘迫平行程序，而這些模型全都可以平行訓練。

## <a name="prerequisites"></a>必要條件

* [Azure 帳戶](https://azure.microsoft.com/free/) (有提供免費試用)。
* 已安裝的 [Azure Machine Learning Workbench](../service/overview-what-is-azure-ml.md) 複本，請依照[安裝和建立快速入門](quickstart-installation.md)安裝 Workbench 並建立帳戶。
* 此案例假設您在 Windows 10 或 MacOS (已於本機安裝 Docker 引擎) 上執行 Azure ML Workbench。 
* 若要執行具有遠端 Docker 容器的案例，請依照[指示](https://docs.microsoft.com/azure/machine-learning/machine-learning-data-science-provision-vm)佈建 Ubuntu 資料科學虛擬機器 (DSVM)。 我們建議使用至少 8 個核心和 28 GB 記憶體的虛擬機器。 虛擬機器的 D4 執行個體有此容量。 
* 若要搭配 Spark 叢集執行此案例，請依照這些[指示](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-provision-linux-clusters)來佈建 Spark HDInsight 叢集。 建議使用一個在標頭和背景工作節點中採用下列設定的叢：集：
    - 四個背景工作節點
    - 八個核心
    - 28 GB 記憶體  
      
  虛擬機器的 D4 執行個體有此容量。 

     **疑難排解**：您的 Azure 訂用帳戶有可用的核心數目配額。 Azure 入口網站不允許建立核心總數超過配額的叢集。 若要找出您的配額，請進入 Azure 入口網站的 [訂用帳戶] 區段，按一下用來部署叢集的訂用帳戶，然後按一下 [使用量 + 配額]。 通常會針對每個 Azure 區域定義配額，您可以選擇在有足夠可用核心的區域中部署 Spark 叢集。 

* 建立用來儲存資料集的 Azure 儲存體帳戶。 請遵循這些[指示](https://docs.microsoft.com/azure/storage/common/storage-create-storage-account)以建立儲存體帳戶。

## <a name="data-description"></a>資料說明

我們使用 [TalkingData 資料集](https://www.kaggle.com/c/talkingdata-mobile-user-demographics/data)。 此資料集有來自行動電話中應用程式的事件。 目標在於依照電話的類型及使用者最近產生的事件，預測行動電話使用者的性別和年齡類別。  

## <a name="scenario-structure"></a>案例結構
這個案例在 GitHub 存放庫中有多個資料夾。 程式碼和組態檔位於 **Code** 資料夾中，所有文件位於 **Docs** 資料夾中，而所有映像位於 **Images** 資料夾中。 根資料夾具有讀我檔案，其中包含此案例的簡短摘要。

### <a name="getting-started"></a>開始使用
按一下 Azure Machine Learning Workbench 圖示，執行 Azure Machine Learning Workbench 並從「分散式微調超參數」範本建立專案。 您可以在[安裝和建立快速入門](quickstart-installation.md)中找到有關如何建立新專案的詳細指示。   

### <a name="configuration-of-execution-environments"></a>執行環境的設定
我們會示範如何在遠端 Docker 容器和 Spark 叢集中執行程式碼。 我們一開始會說明兩種環境通用的設定。 

我們會使用 Azure Machine Learning Workbench 的預設 Docker 容器中所未提供的 [scikit-learn](https://anaconda.org/conda-forge/scikit-learn)、[xgboost](https://anaconda.org/conda-forge/xgboost) 和 [azure-storage](https://pypi.python.org/pypi/azure-storage) 套件。 azure-storage 套件需要安裝 [cryptography](https://pypi.python.org/pypi/cryptography) 和 [azure](https://pypi.python.org/pypi/azure) 套件。 為了在 Docker 容器和 Spark 叢集的節點中安裝這些套件，我們會修改 conda_dependencies.yml 檔案：

    name: project_environment
    channels:
      - conda-forge
    dependencies:
      - python=3.5.2
      - scikit-learn
      - xgboost
      - pip:
        - cryptography
        - azure
        - azure-storage

修改的 conda\_dependencies.yml 檔案會儲存在教學課程的 aml_config 目錄中。 

在後續步驟中，我們會將執行環境連線至 Azure 帳戶。 按一下 AML Workbench 左上角的 [檔案] 功能表。 然後選擇 [開啟命令提示字元]。 然後在 CLI 中執行

    az login

您會收到一則訊息

    To sign in, use a web browser to open the page https://aka.ms/devicelogin and enter the code <code> to authenticate.

移至這個網頁，輸入程式碼並登入您的 Azure 帳戶。 這個步驟之後，請在 CLI 中執行

    az account list -o table

並尋找具有您 AML Workbench 工作區帳戶的 Azure 訂用帳戶 ID。 最後，在 CLI 中執行

    az account set -s <subscription ID>

完成對 Azure 訂用帳戶的連線。

在後續兩節中，我們會示範如何完成遠端 Docker 和 Spark 叢集的設定。

#### <a name="configuration-of-remote-docker-container"></a>遠端 Docker 容器的設定

 若要設定遠端 Docker 容器，請在 CLI 中執行

    az ml computetarget attach remotedocker --name dsvm --address <IP address> --username <username> --password <password> 

使用 DSVM 中的 IP 位址、使用者名稱和密碼。 在 Azure 入口網站中 DSVM 頁面的 [概觀] 區段中，可以找到 DSVM 的 IP 位址：

![VM IP](media/scenario-distributed-tuning-of-hyperparameters/vm_ip.png)

#### <a name="configuration-of-spark-cluster"></a>Spark 叢集的組態

若要設定 Spark 環境，請在 CLI 中執行

    az ml computetarget attach cluster --name spark --address <cluster name>-ssh.azurehdinsight.net  --username <username> --password <password> 

使用叢集的名稱、叢集的 SSH 使用者名稱和密碼。 SSH 使用者名稱的預設值是 `sshuser`，除非您在叢集的佈建期間變更此值。 在 Azure 入口網站中您叢集頁面的 [屬性] 區段中，可以找到叢集的名稱：

![叢集名稱](media/scenario-distributed-tuning-of-hyperparameters/cluster_name.png)

我們使用 spark-sklearn 套件，讓 Spark 成為分散式超參數微調的執行環境。 我們修改了 spark_dependencies.yml 檔案，以在使用 Spark 執行環境時安裝這個套件：

    configuration: 
      #"spark.driver.cores": "8"
      #"spark.driver.memory": "5200m"
      #"spark.executor.instances": "128"
      #"spark.executor.memory": "5200m"  
      #"spark.executor.cores": "2"
  
    repositories:
      - "https://mmlspark.azureedge.net/maven"
      - "https://spark-packages.org/packages"
    packages:
      - group: "com.microsoft.ml.spark"
        artifact: "mmlspark_2.11"
        version: "0.7.91"
      - group: "databricks"
        artifact: "spark-sklearn"
        version: "0.2.0"

修改的 spark\_dependencies.yml 檔案會儲存在教學課程的 aml_config 目錄中。 

### <a name="data-ingestion"></a>資料擷取
此案例中的程式碼假設資料儲存在 Azure blob 儲存體中。 我們一開始示範如何從 Kaggle 網站將資料下載到您的電腦，並將它上傳至 blob 儲存體。 然後我們會示範如何從 blob 儲存體讀取資料。 

若要從 Kaggle 下載資料，請移至[資料集頁面](https://www.kaggle.com/c/talkingdata-mobile-user-demographics/data)並且按 [下載] 按鈕。 系統會要求您登入 Kaggle。 登入之後，系統會將您重新導回資料集頁面。 然後選取左欄中的前七個檔案並且按 [下載] 按鈕，下載這些檔案。 所下載檔案的大小總計為 289 Mb。 若要將這些檔案上傳至 blob 儲存體，請在您的儲存體帳戶中建立 blob 儲存體容器 'dataset'。 移至您儲存體帳戶的 Azure 頁面，按一下 Blob，然後按一下 [+容器]，即可進行該作業。 輸入 'dataset' 作為名稱，然後按一下 [確定]。 以下螢幕擷取畫面說明這些步驟：

![開啟 blob](media/scenario-distributed-tuning-of-hyperparameters/open_blob.png)
![開啟容器](media/scenario-distributed-tuning-of-hyperparameters/open_container.png)

之後，從清單中選取資料集容器，並按一下 [上傳] 按鈕。 Azure 入口網站允許同時上傳多個檔案。 在 [上傳 blob] 區段中按一下資料夾按鈕、選取資料集中的所有檔案、按一下 [開啟]，然後按一下 [上傳]。 以下螢幕擷取畫面會說明這些步驟：

![上傳 Blob](media/scenario-distributed-tuning-of-hyperparameters/upload_blob.png) 

視您的網際網路連線而定，上傳檔案需要數分鐘的時間。 

在我們的程式碼中，我們使用 [Azure 儲存體 SDK](https://docs.microsoft.com/python/azure/) 將資料集從 blob 儲存體下載到目前的執行環境。 下載會在 load_data.py 檔案的 load\_data() 函式中執行。 若要使用此程式碼，您必須以主控資料集之儲存體帳戶的名稱與主要金鑰取代 <ACCOUNT_NAME> and <ACCOUNT_KEY>。 您可以在您儲存體帳戶的 Azure 頁面左上角看到帳戶名稱。 若要取得帳戶，請在儲存體帳戶的 Azure 頁面中選取 [存取金鑰] (請參閱「資料擷取」一節中的第一個螢幕擷取畫面)，然後複製金鑰欄的第一列中的長字串：
 
![存取金鑰](media/scenario-distributed-tuning-of-hyperparameters/access_key.png)

來自 load_data() 函式的下列程式碼會下載單一檔案：

    from azure.storage.blob import BlockBlobService

    # Define storage parameters 
    ACCOUNT_NAME = "<ACCOUNT_NAME>"
    ACCOUNT_KEY = "<ACCOUNT_KEY>"
    CONTAINER_NAME = "dataset"

    # Define blob service     
    my_service = BlockBlobService(account_name=ACCOUNT_NAME, account_key=ACCOUNT_KEY)

    # Load blob
    my_service.get_blob_to_path(CONTAINER_NAME, 'app_events.csv.zip', 'app_events.csv.zip')

請注意，您不需要手動執行 load_data.py 檔案。 稍後將從其他檔案呼叫它。

### <a name="feature-engineering"></a>特徵設計
用於計算所有特徵的程式碼位於 feature\_engineering.py 檔案。 您不需要手動執行 feature_engineering.py 檔案。 稍後將從其他檔案進行呼叫。

我們會建立多個特徵集：
* 行動電話的廠牌和型號的獨熱 (One-hot) 編碼 (one\_hot\_brand_model 函式)
* 使用者在每個週間日產生的事件片段 (weekday\_hour_features 函式)
* 使用者在每個小時產生的事件片段 (weekday\_hour_features 函式)
* 使用者在每個週間日和小時組合中產生的事件片段 (weekday\_hour_features 函式)
* 使用者在每個應用程式中產生的事件片段 (one\_hot\_app_labels 函式)
* 使用者在每個應用程式標籤中產生的事件片段 (one\_hot\_app_labels 函式)
* 使用者在每個應用程式類別中產生的事件片段 (text\_category_features 函式)
* 用來產生事件之應用程式類別的指標特徵 (one\_hot_category 函式)

這些特徵是由 Kaggle 核心 ([應用程式和標籤的線性模型](https://www.kaggle.com/dvasyukova/a-linear-model-on-apps-and-labels)) 所啟發。

計算這些特徵時需要大量的記憶體。 一開始，我們會嘗試在具有 16-GB RAM 的本機環境中計算特徵。 我們可以計算前四個特徵集，但是在運算第五個特徵集時收到「記憶體不足」錯誤。 前四個特徵集的計算是在 singleVMsmall.py 檔案中，並可以在本機環境中執行，方法是執行 

     az ml experiment submit -c local .\singleVMsmall.py   

在 CLI 視窗中。

由於本機環境對於計算所有特徵集而言太小，所以我們會切換至具有較大記憶體的遠端 DSVM。 在 DSVM 內部執行是在 AML Workbench 所管理的 Docker 容器內部進行。 使用這個 DSVM，我們就能夠計算所有特徵、訓練模型，以及微調超參數 (請參閱下一節)。 singleVM.py 檔案具有完整特徵計算和模型化程式碼。 在下一節中，我們將示範如何在遠端 DSVM 中執行 singleVM.py。 

### <a name="tuning-hyperparameters-using-remote-dsvm"></a>使用遠端 DSVM 微調超參數
我們會使用梯度樹提升的 [xgboost](https://anaconda.org/conda-forge/xgboost) 實作 [1]。 我們也會使用 [scikit-learn](http://scikit-learn.org/) 套件來微調 xgboost 的超參數。 雖然 xgboost 不屬於 scikit-learn 套件，但它會實作 scikit-learn API，因此可以搭配 scikit-learn 的超參數微調函式使用。 

Xgboost 有八個超參數，如[這裡](https://github.com/dmlc/xgboost/blob/master/doc/parameter.md)所述：
* n_estimators
* max_depth
* reg_alpha
* reg_lambda
* colsample\_by_tree
* learning_rate
* colsample\_by_level
* subsample
* objective  
 
一開始，我們會使用遠端 DSVM 並從小型候選值網格中微調超參數：

    tuned_parameters = [{'n_estimators': [300,400], 'max_depth': [3,4], 'objective': ['multi:softprob'], 'reg_alpha': [1], 'reg_lambda': [1], 'colsample_bytree': [1],'learning_rate': [0.1], 'colsample_bylevel': [0.1,], 'subsample': [0.5]}]  

此網格有四個超參數的值組合。 我們使用 5 折交叉驗證，這會產生 4x5=20 次的 xgboost 執行。 為了測量模型的效能，我們使用負值對數損失計量。 下列程式碼會從網格中尋找可將交叉驗證負值對數損失最大化的超參數值。 此程式碼也會使用這些值來訓練完整訓練集的最終模型：

    clf = XGBClassifier(seed=0)
    metric = 'neg_log_loss'
    
    clf_cv = GridSearchCV(clf, tuned_parameters, scoring=metric, cv=5, n_jobs=8)
    model = clf_cv.fit(X_train,y_train)

建立模型之後，我們會儲存超參數微調的結果。 我們會使用 AML Workbench 的記錄 API 來儲存超參數的最佳值，以及負值對數損失的對應交叉驗證估計值：

    from azureml.logging import get_azureml_logger

    # initialize logger
    run_logger = get_azureml_logger()

    ...

    run_logger.log(metric, float(clf_cv.best_score_))

    for key in clf_cv.best_params_.keys():
        run_logger.log(key, clf_cv.best_params_[key]) 

我們也會建立 sweeping_results.txt 檔案，其中包含網格中所有超參數值組合的交叉驗證負值對數損失。

    if not path.exists('./outputs'):
        makedirs('./outputs')
    outfile = open('./outputs/sweeping_results.txt','w')

    print("metric = ", metric, file=outfile)
    for i in range(len(model.grid_scores_)):
        print(model.grid_scores_[i], file=outfile)
    outfile.close()

這個檔案會儲存在特別的 ./outputs 目錄中。 稍後我們會示範如何下載它。  

 第一次在遠端 DSVM 中執行 singleVM.py 之前，我們會在那裡建立 Docker 容器，方法是執行 

    az ml experiment prepare -c dsvm

在 CLI 視窗中。 建立 Docker 容器需要數分鐘的時間。 之後，我們會在 DSVM 中執行 singleVM.py：

    az ml experiment submit -c dsvm .\singleVM.py

如果 DSVM 有 8 個核心和 28 Gb 記憶體，此命令會在 1 小時 38 分鐘內完成。 您可以在 AML Workbench 的 [執行歷程記錄] 視窗中檢視已記錄的值：

![執行歷程記錄](media/scenario-distributed-tuning-of-hyperparameters/run_history.png)

根據預設，[執行歷程記錄] 視窗會顯示第 1-2 個記錄值的值和圖形。 若要查看所選超參數值的完整清單，請按一下在上一個螢幕擷取畫面中以紅色圓圈標記的設定圖示。 然後選取要在資料表中顯示的超參數。 此外，若要選取 [執行歷程記錄] 視窗最上方顯示的圖形，按一下以藍色圓形標記的設定圖示，並選取清單中的圖形。 

在 [執行屬性] 視窗中也可以檢查所選的超參數值： 

![執行屬性](media/scenario-distributed-tuning-of-hyperparameters/run_properties.png)

[執行屬性] 視窗的右上角有一個 [輸出檔案] 區段，內含在 '.\output' 資料夾中建立之所有檔案的清單。 選取 sweeping\_results.txt 並且按 [下載] 按鈕，即可下載該檔案。 sweeping_results.txt 應該有下列輸出：

    metric =  neg_log_loss
    mean: -2.29096, std: 0.03748, params: {'colsample_bytree': 1, 'learning_rate': 0.1, 'subsample': 0.5, 'n_estimators': 300, 'reg_alpha': 1, 'objective': 'multi:softprob', 'colsample_bylevel': 0.1, 'reg_lambda': 1, 'max_depth': 3}
    mean: -2.28712, std: 0.03822, params: {'colsample_bytree': 1, 'learning_rate': 0.1, 'subsample': 0.5, 'n_estimators': 400, 'reg_alpha': 1, 'objective': 'multi:softprob', 'colsample_bylevel': 0.1, 'reg_lambda': 1, 'max_depth': 3}
    mean: -2.28706, std: 0.03863, params: {'colsample_bytree': 1, 'learning_rate': 0.1, 'subsample': 0.5, 'n_estimators': 300, 'reg_alpha': 1, 'objective': 'multi:softprob', 'colsample_bylevel': 0.1, 'reg_lambda': 1, 'max_depth': 4}
    mean: -2.28530, std: 0.03927, params: {'colsample_bytree': 1, 'learning_rate': 0.1, 'subsample': 0.5, 'n_estimators': 400, 'reg_alpha': 1, 'objective': 'multi:softprob', 'colsample_bylevel': 0.1, 'reg_lambda': 1, 'max_depth': 4}

### <a name="tuning-hyperparameters-using-spark-cluster"></a>使用 Spark 叢集微調超參數
我們使用 Spark 相應放大微調超參數及使用較大的網格。 我們新的網格是

    tuned_parameters = [{'n_estimators': [300,400], 'max_depth': [3,4], 'objective': ['multi:softprob'], 'reg_alpha': [1], 'reg_lambda': [1], 'colsample_bytree': [1], 'learning_rate': [0.1], 'colsample_bylevel': [0.01, 0.1], 'subsample': [0.5, 0.7]}]

此網格有 16 個超參數的值組合。 因為我們使用 5 折交叉驗證，所以我們會執行 xgboost 16x5=80 次。

scikit-learn 套件沒有使用 Spark 叢集微調超參數的原生支援。 幸運的是，Databricks 提供的 [spark-sklearn](https://spark-packages.org/package/databricks/spark-sklearn) 套件填補此缺口。 此套件提供 GridSearchCV 函式，其具有與 scikit-learn 中的 GridSearchCV 函式幾乎相同的 API。 為了使用 spark-sklearn 及使用 Spark 來微調超參數，我們必須建立 Spark 內容

    from pyspark import SparkContext
    sc = SparkContext.getOrCreate()

然後我們會將 

    from sklearn.model_selection import GridSearchCV

取代為 

    from spark_sklearn import GridSearchCV

此外，我們也將從 scikit-learn 對 GridSearchCV 的呼叫取代為從 spark-sklearn 對 GridSearchCV 的呼叫：

    clf_cv = GridSearchCV(sc = sc, param_grid = tuned_parameters, estimator = clf, scoring=metric, cv=5)

使用 Spark 微調超參數的最終程式碼位於 distributed\_sweep.py 檔案。 singleVM.py 和 distributed_sweep.py 之間的差異，在於網格的定義以及其他四行程式碼。 另請注意，由於 AML Workbench 服務，將執行環境從遠端 DSVM 變更為 Spark 叢集時，記錄程式碼不會變更。

第一次在 Spark 叢集中執行 distributed_sweep.py 之前，我們需要安裝在那裡 Python 套件。 達成方式為執行 

    az ml experiment prepare -c spark

在 CLI 視窗中。 此安裝可能需要數分鐘的時間。 之後，我們會在 Spark 叢集中執行 distributed_sweep.py:

    az ml experiment submit -c spark .\distributed_sweep.py

如果 DSVM 有 6 個背景工作節點和 28 Gb 記憶體，此命令會在 1 小時 6 分鐘內完成。 超參數微調結果可以在 Azure Machine Learning Workbench 中存取，存取方式和遠端 DSVM 執行相同。 (也就是記錄檔、超參數的最佳值及 sweeping_results.txt 檔案)

### <a name="architecture-diagram"></a>架構圖表

下圖顯示整體工作流程：![架構](media/scenario-distributed-tuning-of-hyperparameters/architecture.png) 

## <a name="conclusion"></a>結論 

在此案例中，我們示範了如何使用 Azure Machine Learning Workbench，在遠端虛擬機器和 Spark 叢集中執行超參數微調。 我們發現 Azure Machine Learning Workbench 所提供的工具，可讓您輕鬆設定執行環境。 也可以在它們之間輕鬆切換。 

## <a name="references"></a>參考

[1] T. Chen and C. Guestrin. [XGBoost: A Scalable Tree Boosting System](https://arxiv.org/abs/1603.02754). KDD 2016.




