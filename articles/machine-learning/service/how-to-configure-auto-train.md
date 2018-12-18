---
title: 設定您的自動化機器學習實驗 - Azure Machine Learning
description: 自動化機器學習服務可為您挑選演算法，並產生馬上可進行部署的模型。 了解可用來設定自動化機器學習實驗的選項。
author: nacharya1
ms.author: nilesha
ms.reviewer: sgilley
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 33126c094a55bc57edd49a54fbc4f5acd7401998
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078999"
---
# <a name="configure-your-automated-machine-learning-experiment"></a>設定您的自動化機器學習實驗

自動化機器學習服務 (自動化 ML) 可為您挑選演算法和超參數，並產生馬上可進行部署的模型。 您也可以下載模型以進一步自訂。 有數個選項可用來設定自動化 ML 實驗。 在本指南中，您將了解如何定義各種不同的組態設定。

若要檢視自動化 ML 的範例，請參閱[教學課程：使用自動化機器學習訓練分類模型](tutorial-auto-train-models.md)或[使用雲端中的自動化機器學習來訓練模型](how-to-auto-train-remote.md)。

自動化機器學習服務中的可用設定選項：

* 選取您的實驗類型，例如分類、迴歸 
* 資料來源、格式和擷取資料
* 選擇您的計算目標 (本機或遠端)
* 自動化 ML 實驗設定
* 執行自動化 ML 實驗
* 探索模型計量
* 註冊和部署模型

## <a name="select-your-experiment-type"></a>選取您的實驗類型
在開始實驗之前，您應先決定所要解決的機器學習問題類型。 自動化 ML 支援兩種監督式學習：分類和迴歸。 在自動化和調整程序期間，自動化 ML 支援下列演算法。 身為使用者，您不需要指定演算法。
分類 | 迴歸
--|--
sklearn.linear_model.LogisticRegression | sklearn.linear_model.ElasticNet
sklearn.linear_model.SGDClassifier  | sklearn.ensemble.GradientBoostingRegressor
sklearn.naive_bayes.BernoulliNB | sklearn.tree.DecisionTreeRegressor
sklearn.naive_bayes.MultinomialNB | sklearn.neighbors.KNeighborsRegressor
sklearn.svm.SVC | sklearn.linear_model.LassoLars
sklearn.svm.LinearSVC | sklearn.linear_model.SGDRegressor
sklearn.calibration.CalibratedClassifierCV |    sklearn.ensemble.RandomForestRegressor
sklearn.neighbors.KNeighborsClassifier |    sklearn.ensemble.ExtraTreesRegressor
sklearn.tree.DecisionTreeClassifier |   lightgbm.LGBMRegressor
sklearn.ensemble.RandomForestClassifier |
sklearn.ensemble.ExtraTreesClassifier   |
sklearn.ensemble.GradientBoostingClassifier |
lightgbm.LGBMClassifier |


## <a name="data-source-and-format"></a>資料來源和格式
自動化 ML 支援位於本機桌面或雲端中的 Azure Blob 儲存體所包含的資料。 資料可讀取到支援 scikit-learn 的資料格式中。 您可以將資料讀取到 1) Numpy 陣列 X (特徵) 和 y (目標變數，或者也稱為標籤) 或 2) Pandas 資料框架中。 

範例：

1.  Numpy 陣列

    ```python
    digits = datasets.load_digits()
    X_digits = digits.data 
    y_digits = digits.target
    ```

2.  Pandas 資料框架

    ```python
    Import pandas as pd
    df = pd.read_csv("https://automldemods.blob.core.windows.net/datasets/PlayaEvents2016,_1.6MB,_3.4k-rows.cleaned.2.tsv", delimiter="\t", quotechar='"') 
    # get integer labels 
    le = LabelEncoder() 
    le.fit(df["Label"].values) 
    y = le.transform(df["Label"].values) 
    df = df.drop(["Label"], axis=1) 
    df_train, _, y_train, _ = train_test_split(df, y, test_size=0.1, random_state=42)
    ```

## <a name="fetch-data-for-running-experiment-on-remote-compute"></a>擷取用來對遠端計算執行實驗的資料

如果您使用遠端計算來執行實驗，則必須將資料擷取包覆在個別的 python 指令碼 `get_data()` 中。 此指令碼會在自動化 ML 實驗執行所在的遠端計算上執行。 若使用 `get_data`，您即無須在每次反覆運算時透過連線擷取資料。 若未使用 `get_data`，您在遠端計算上執行的實驗將會失敗。

以下是 `get_data` 的範例：

```python
%%writefile $project_folder/get_data.py 
import pandas as pd 
from sklearn.model_selection 
import train_test_split 
from sklearn.preprocessing import LabelEncoder 
def get_data(): # Burning man 2016 data 
    df = pd.read_csv("https://automldemods.blob.core.windows.net/datasets/PlayaEvents2016,_1.6MB,_3.4k-rows.cleaned.2.tsv", delimiter="\t", quotechar='"') 
    # get integer labels 
    le = LabelEncoder() 
    le.fit(df["Label"].values) 
    y = le.transform(df["Label"].values) 
    df = df.drop(["Label"], axis=1) 
    df_train, _, y_train, _ = train_test_split(df, y, test_size=0.1, random_state=42) 
    return { "X" : df, "y" : y }
```

在您的 `AutoMLConfig` 物件中，您會指定 `data_script` 參數，並提供如下的 `get_data` 指令碼檔案路徑：

```python
automl_config = AutoMLConfig(****, data_script=project_folder + "/get_data.py", **** )
```

`get_data` 指令碼可能會傳回下列結果：
Key | 類型 |    互斥項目 | 說明
---|---|---|---
X | Pandas 資料框架或 Numpy 陣列 | data_train、標籤、資料行 |  所有要用於訓練的特徵
y | Pandas 資料框架或 Numpy 陣列 |   標籤   | 要用於訓練的標籤資料。 就分類而言，應為整數的陣列。
X_valid | Pandas 資料框架或 Numpy 陣列   | data_train、標籤 | _選擇性_ 所有要用於驗證的特徵。 如果未指定，則 X 會分割至訓練和驗證之間
y_valid |   Pandas 資料框架或 Numpy 陣列 | data_train、標籤 | _選擇性_ 要用於驗證的標籤資料。 如果未指定，則 y 會分割至訓練和驗證之間
sample_weight | Pandas 資料框架或 Numpy 陣列 |   data_train、標籤、資料行| _選擇性_ 每個範例的加權值。 如果您想要為資料點指派不同的加權，則應使用 
sample_weight_valid | Pandas 資料框架或 Numpy 陣列 | data_train、標籤、資料行 |    _選擇性_ 每個驗證範例的加權值。 如果未指定，則 sample_weight 會分割至訓練和驗證之間
data_train |    Pandas 資料框架 |  X、y、X_valid、y_valid |    所有要用於訓練的資料 (特徵+標籤)
標籤 | 字串  | X、y、X_valid、y_valid |  data_train 中的哪個資料行代表標籤
columns | 字串的陣列  ||  _選擇性_ 要用於特徵的資料行白名單
cv_splits_indices   | 一連串整數 ||  _選擇性_ 用來分割交叉驗證資料的索引清單

## <a name="train-and-validation-data"></a>訓練和驗證資料

您可以透過 get_data() 個別的訓練和驗證集，或直接在 `AutoMLConfig` 方法中指定。

## <a name="cross-validation-split-options"></a>交叉驗證分割選項

### <a name="k-folds-cross-validation"></a>k 疊交叉驗證

使用 `n_cross_validations` 設定來指定交叉驗證的數目。 訓練資料集會隨機分割為相同大小的 `n_cross_validations` 疊。 在每個交叉驗證回合中，將使用其中一個疊來驗證以其餘幾疊進行訓練的模型。 此程序會重複執行 `n_cross_validations` 回合，直到每一疊都已作為驗證集執行一次。 最後將會報告 `n_cross_validations` 個回合的總平均分數，並以整個訓練資料集重新訓練對應的模型。

### <a name="monte-carlo-cross-validation-aka-repeated-random-sub-sampling"></a>蒙地卡羅交叉驗證 (也稱為 重複隨機子取樣)

使用 `validation_size` 指定應該用於驗證的訓練資料集百分比，並使用 `n_cross_validations` 指定交叉驗證的數目。 在每個交叉驗證回合中，將隨機選取 `validation_size` 大小的子集來驗證以其餘資料進行訓練的模型。 最後將會報告 `n_cross_validations` 個回合的總平均分數，並以整個訓練資料集重新訓練對應的模型。

### <a name="custom-validation-dataset"></a>自訂驗證資料集

如果隨機分割不可行，請使用自訂驗證資料集 (通常是時間序列資料或不平衡的資料)。 使用此方式時，您可以指定自己的驗證資料集。 系統會根據指定的驗證資料集來評估模型，而不是隨機的資料集。

## <a name="compute-to-run-experiment"></a>要執行實驗的計算

接下來，請決定要在何處訓練模型。 自動化 ML 訓練實驗可在您擁有及管理的計算目標上執行。 

支援的計算選項包括：
1.  您的本機電腦，例如本機桌上型電腦或膝上型電腦 – 通常適用於資料集並不大，且您尚在探索階段時。
2.  雲端中的遠端電腦 (執行 Linux 的 [Azure 資料科學虛擬機器](https://azure.microsoft.com/services/virtual-machines/data-science-virtual-machines/)) – 您有大型資料集，而且想要相應增加至可在 Azure 雲端中使用的大型機器。 
3.  Azure Batch AI 叢集 – 您可以經由設定相應放大，並以平行方式執行自動化 ML 反覆運算的受控叢集。 


## <a name="configure-your-experiment-settings"></a>設定您的實驗設定

有數個途徑可用來設定自動化 ML 實驗。 將 `AutoMLConfig` 物件具現化即可設定這些參數。

部分範例包括：

1.  使用加權的 AUC 作為主要計量的分類實驗，每次反覆運算的時間上限為 12,000 秒，並且在 50 次反覆運算和 2 疊的交叉驗證之後結束實驗。

    ```python
    automl_classifier = AutoMLConfig(
        task='classification',
        primary_metric='AUC_weighted',
        max_time_sec=12000,
        iterations=50,
        X=X, 
        y=y,
        n_cross_validations=2)
    ```
2.  以下範例說明設定在 100 次反覆運算之後結束的迴歸實驗，每次反覆運算的持續時間上限為 600 秒，並執行 5 疊的交叉驗證。

    ````python
    automl_regressor = AutoMLConfig(
        task='regression',
        max_time_sec=600,
        iterations=100,
        primary_metric='r2_score',
        X=X, 
        y=y,
        n_cross_validations=5)
    ````

下表列出您的實驗適用的參數設定及其預設值。

屬性 |  說明 | 預設值
--|--|--
`task`  |指定機器學習問題的類型。 允許的值包括 <li>分類</li><li>迴歸</li>    | None |
`primary_metric` |您在建置模型時想要最佳化的計量。 例如，如果您將精確度指定為 primary_metric，則自動化 ML 會尋找具有最大精確度的模型。 在每個實驗中只能指定一個 primary_metric。 允許的值包括 <br/>**分類**：<br/><li> 精確度  </li><li> AUC_weighted</li><li> precision_score_weighted </li><li> balanced_accuracy </li><li> average_precision_score_weighted </li><br/>**迴歸**： <br/><li> normalized_mean_absolute_error </li><li> spearman_correlation </li><li> normalized_root_mean_squared_error </li><li> normalized_root_mean_squared_log_error</li><li> R2_score    </li> | 分類：精確度 <br/>迴歸：spearman_correlation <br/> |
`exit_score` |  您可以設定 primary_metric 的目標值。 一旦發現模型符合 primary_metric 目標，自動化 ML 即會停止反覆運算，且實驗會終止。 如果未設定此值 (預設值)，自動化 ML 實驗將會繼續依據反覆運算中指定的次數執行反覆運算。 採用雙精度浮點數值。 如果尚未達到目標，則自動化 ML 將會繼續執行，直到達到反覆運算中指定的反覆運算次數。|   None
`iterations` |反覆運算次數上限。 每次反覆運算相當於一個產生管線的訓練作業。 管線是資料前置處理和模型。 若要取得高品質的模型，請使用 250 或更高的值 | 100
`Concurrent_iterations`|    要平行執行的反覆運算次數上限。 這項設定僅適用於遠端計算。|    1
`max_cores_per_iteration`   | 指出在計算目標上將使用多少個核心來訓練單一管線。 如果演算法可以使用多個核心，這將可提高多核心電腦的效能。 您可以將它設定為 -1，以使用電腦上所有可用的核心。|  1
`max_time_sec` |    限制特定反覆運算所花費的時間量 (秒)。 如果反覆運算超過指定的時間量，則會取消該反覆運算。 如果未設定，則反覆運算會繼續執行直到完成。 |   None
`n_cross_validations`   |交叉驗證分割數目| None
`validation_size`   |驗證集的大小，以所有訓練範例的百分比表示。|  None
`preprocess` | True/False <br/>True 會啟用實驗，以對輸入執行前置處理。 以下是前置處理的子集<li>遺漏資料：插補遺漏資料 - 平均數值，最常出現的文字 </li><li>類別值：如果資料類型是數值，而唯一值的數目小於 5%，則會轉換為 one-hot 編碼 </li><li>如需完整清單，請查看 [GitHub 存放庫](https://aka.ms/aml-notebooks)</li><br/>注意：如果是疏鬆資料，則無法使用 preprocess = true |  False | 
`blacklist_algos`   | 自動化 ML 實驗中有許多它會嘗試的不同演算法。 設定自動化 ML 以從實驗中排除特定演算法。 如果您知道特定演算法不適用於您的資料集，則有所幫助。 排除演算法可為您節省計算資源和訓練時間。<br/>允許的分類值<br/><li>羅吉斯迴歸</li><li>SGD 分類器</li><li>MultinomialNB</li><li>BernoulliNB</li><li>SVM</li><li>LinearSVM</li><li>kNN</li><li>DT</li><li>RF</li><li>額外樹狀結構</li><li>漸層停駐提升</li><li>lgbm_classifier</li><br/>允許的迴歸值<br/><li>彈性網路</li><li>漸層停駐提升迴歸輸入變數</li><li>DT 迴歸輸入變數</li><li>kNN 迴歸輸入變數</li><li>Lasso lars</li><li>SGD 迴歸輸入變數</li><li>RF 迴歸輸入變數</li><li>額外樹狀結構迴歸輸入變數</li>|   None
`verbosity` |控制記錄層級，INFO 最詳細，CRITICAL 最簡要。<br/>允許的值包括：<br/><li>logging.INFO</li><li>logging.WARNING</li><li>logging.ERROR</li><li>logging.CRITICAL</li>  | logging.INFO</li> 
`X` | 所有要用於訓練的特徵 |  None
`y` |   要用於訓練的標籤資料。 就分類而言，應為整數的陣列。|  None
`X_valid`|_選擇性_ 所有要用於驗證的特徵。 如果未指定，則 X 會分割至訓練和驗證之間 |   None
`y_valid`   |_選擇性_ 要用於驗證的標籤資料。 如果未指定，則 y 會分割至訓練和驗證之間    | None
`sample_weight` |   _選擇性_ 每個範例的加權值。 如果您想要為資料點指派不同的加權，則應使用 |   None
`sample_weight_valid`   |   _選擇性_ 每個驗證範例的加權值。 如果未指定，則 sample_weight 會分割至訓練和驗證之間   | None
`run_configuration` |   RunConfiguration 物件。  用於遠端回合。 |None
`data_script`  |    包含 get_data 方法的檔案路徑。  遠端回合的必要項目。   |None


## <a name="run-experiment"></a>執行實驗

接下來，我們可以起始要執行的實驗，以產生模型。 將 `AutoMLConfig` 傳至 `submit` 方法以產生模型。

```python
run = experiment.submit(automl_config, show_output=True)
```

>[!NOTE]
>相依性會先安裝在新的 DSVM 上。  最多可能需要 10 分鐘才會顯示輸出。
>將 `show_output` 設定為 True，會使輸出顯示在主控台上。


## <a name="explore-model-metrics"></a>探索模型計量
如果您使用 Notebook，您可以在小工具中或以內嵌方式檢視結果。 請檢視「追蹤和評估模型」的詳細資料。 (請確定 AML 內容包含自動化 ML 的相關資訊)

在每次反覆運算時會儲存下列計量
* AUC_macro
* AUC_micro
* AUC_weighted
* 精確度
* average_precision_score_macro
* average_precision_score_micro
* average_precision_score_weighted
* balanced_accuracy
* f1_score_macro
* f1_score_micro
* f1_score_weighted
* log_loss
* norm_macro_recall
* precision_score_macro
* precision_score_micro
* precision_score_weighted
* recall_score_macro
* recall_score_micro
* recall_score_weighted
* weighted_accuracy

## <a name="next-steps"></a>後續步驟

深入了解[模型部署的方式和位置](how-to-deploy-and-where.md)。

深入了解[如何使用自動化 ML 訓練分類模型](tutorial-auto-train-models.md)或[如何使用自動化 ML 對遠端資源進行訓練](how-to-auto-train-remote.md)。 