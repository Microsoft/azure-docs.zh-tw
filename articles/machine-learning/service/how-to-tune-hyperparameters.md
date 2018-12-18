---
title: 使用 Azure Machine Learning 為您的模型微調超參數
description: 了解如何使用 Azure Machine Learning 服務為您的深度學習/機器學習模型微調超參數。 您將了解如何定義參數搜尋空間、指定要最佳化的主要計量，並提早終止效能不佳的設定。
ms.author: swatig
author: swatig007
ms.reviewer: sgilley
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: b6370fd9125c5b14df781b27e028c139175b7589
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47405796"
---
# <a name="tune-hyperparameters-for-your-model"></a>為您的模型微調超參數

在本文中，您會了解如何有效率地為您的模型微調超參數。 您將了解如何定義參數搜尋空間、指定要最佳化的主要計量，並提早終止效能不佳的設定。 您也可以視覺化各種定型執行，並為您的模型選取效能最佳的設定。

## <a name="what-are-hyperparameters"></a>什麼是超參數？
超參數是定型模型之前所選擇的可調式參數，用以控管定型程序本身。 例如，定型深度類神經網路之前，您必須決定網路中的隱藏層數目，以及每一層的節點數目。 這些值通常會在定型程序期間保持不變。

在深度學習/機器學習案例中，模型效能高度取決於所選取的超參數值。 超參數探索的目標是在各種超參數設定之間進行搜尋，以找到產生所需效能的設定。 一般而言，超參數探索是很費力的手動程序，因為搜尋空間很大，而且評估每項設定的成本很高。

Azure Machine Learning 可讓您以有效率的方式來自動化此超參數探索作業，為您省下大量時間和資源。 您可以指定要探索的超參數值範圍，以及此探索的定型執行數目上限。 系統會接著自動啟動多個同時但具有不同參數設定的定型執行，並找到產生最佳效能的設定 (以使用者所選擇的計量來測量)。 效能不佳的定型執行會自動提早終止，以減少計算資源浪費情形。 這些資源會改用來探索其他超參數設定。

為了使用 Azure Machine Learning 服務為您的模型微調超參數，您必須執行下列作業 -
* 定義超參數搜尋空間
* 指定要最佳化的主要計量
* 指定提早終止原則
* 配置用於超參數微調的資源
* 使用上述設定啟動實驗

## <a name="define-the-hyperparameter-search-space"></a>定義超參數搜尋空間
Azure Machine Learning 服務會探索為每個超參數定義的一組值，藉此來自動微調超參數。

### <a name="types-of-hyperparameters"></a>超參數類型
每個超參數可以為離散或連續。

#### <a name="discrete-hyperparameters"></a>離散超參數 
離散超參數可以指定為離散值之間的一個 `choice`。 `choice` 可以接受一或多個逗點分隔值、`range` 物件，或是任何任意 `list` 物件。 例如  

```Python
    {    
        "batch_size": choice(16, 32, 64, 128)
        "number_of_hidden_layers": choice(range(1,5))
    }
```

在本案例中，batch_size 可以接受值 [16, 32, 64, 128] 的其中一個，而 number_of_hidden_layers 可以接受值 [1, 2, 3, 4] 的其中一個。

進階離散超參數也可以使用一項分佈來指定。 支援下列分佈 -
* `quniform(low, high, q)` - 傳回 round(uniform(low, high) / q) * q 之類的值
* `qloguniform(low, high, q)` - 傳回 round(exp(uniform(low, high)) / q) * q 之類的值
* `qnormal(mu, sigma, q)` - 傳回 round(normal(mu, sigma) / q) * q 之類的值
* `qlognormal(mu, sigma, q)` - 傳回 round(exp(normal(mu, sigma)) / q) * q 之類的值

#### <a name="continuous-hyperparameters"></a>連續超參數 
連續超參數可以指定為一組連續值的分佈。 支援的分佈包括 -
* `uniform(low, high)` - 傳回在低值與高值之間均勻分佈的值
* `loguniform(low, high)` - 傳回根據 exp(uniform(low, high)) 繪製的值，讓傳回值的對數均勻分佈
* `normal(mu, sigma)` - 傳回以平均值 mu 和標準差 sigma 進行常態分佈的實數值
* `lognormal(mu, sigma)` - 傳回根據 exp(normal(mu, sigma)) 繪製的值，讓傳回值的對數常態分佈

以下是參數空間定義的範例 -

```Python
    {    
        "learning_rate": normal(10, 3),
        "keep_probability": uniform(0.05, 0.1)
    }
```

此範例以兩個參數定義搜尋空間：learning_rate 和 keep_probability。 learning_rate 會有平均值 10 和標準差 3 的常態分佈。 keep_probability 會有最小值 0.05 和最大值 0.1 的均勻分佈。

### <a name="sampling-the-hyperparameter-space"></a>取樣超參數空間
使用者也會指定要對所指定超參數空間定義使用的參數取樣方法。 Azure Machine Learning 服務支援隨機取樣、網格取樣和貝氏取樣。

#### <a name="random-sampling"></a>隨機取樣
在隨機取樣中，超參數值會從定義的搜尋空間隨機選取。 隨機取樣允許搜尋空間同時包含離散和連續超參數。 例如

```Python
from azureml.train.hyperdrive import RandomParameterSampling
param_sampling = RandomParameterSampling( {
        "learning_rate": normal(10, 3),
        "keep_probability": uniform(0.05, 0.1),
        "batch_size": choice(16, 32, 64, 128)
    }
)
```

#### <a name="grid-sampling"></a>網格取樣
網格取樣會對所定義搜尋空間中的所有可行值，執行簡單的網格搜尋。 它只能搭配使用 `choice` 指定的超參數運作。 例如，下列空間共有六個樣本 -

```Python
from azureml.train.hyperdrive import GridParameterSampling
param_sampling = GridParameterSampling( {
        "num_hidden_layers": choice(1, 2, 3),
        "batch_size": choice(16, 32)
    }
)
```

#### <a name="bayesian-sampling"></a>貝氏取樣
貝氏取樣是以貝氏最佳化演算法為基礎，並對下一個要取樣的超參數值做出明智選擇。 它會根據先前樣本的執行方式來挑選此樣本，以確保新樣本可改善回報的主要計量。

使用貝氏取樣時，同時執行的數目會影響微調程序有效性。 一般而言，減少同時執行數目可能會導致更佳的取樣聚合。 這是因為降低平行處理原則程度會讓更多執行受益於先前完成的執行。

貝氏取樣僅支援搜尋空間上的 `choice` 和 `uniform` 分佈。 例如 

```Python
from azureml.train.hyperdrive import BayesianParameterSampling
param_sampling = BayesianParameterSampling( {
        "learning_rate": uniform(0.05, 0.1),
        "batch_size": choice(16, 32, 64, 128)
    }
)
```

> [!NOTE]
> 貝氏取樣目前不支援任何提早終止原則 (請參閱[指定提早終止原則](#specify-an-early-termination-policy))。 如果使用貝氏參數取樣，您必須將原則設定為 `None`。 其效果相當於不指定搭配貝氏取樣使用的終止原則。
>
> ```Python
> early_termination_policy = None
> ```

## <a name="specify-a-primary-metric-to-optimize"></a>指定要最佳化的主要計量
微調超參數時，您必須指定要讓超參數微調實驗最佳化的主要計量。 每個定型執行會針對此主要計量進行評估，而且效能不佳的執行 (其主要計量不符合提早終止原則所設定的準則) 將會終止。 除了指定主要計量名稱，您也必須指定最佳化的目標 (要最大化或最小化主要計量)。
* `primary_metric_name`：要最佳化的主要計量名稱。 主要計量名稱必須與定型指令碼所記錄的計量名稱完全相符。 請參閱[記錄用於超參數微調的計量](#log-metrics-for-hyperparameter-tuning)。
* `primary_metric_goal`：它可以是 `PrimaryMetricGoal.MAXIMIZE` 或 `PrimaryMetricGoal.MINIMIZE`，而且會在評估執行時，決定要最大化或最小化主要計量。 

例如 -
```Python
primary_metric_name="accuracy",
primary_metric_goal=PrimaryMetricGoal.MAXIMIZE
```
這會最佳化執行將「正確性」提高到最大。

### <a name="log-metrics-for-hyperparameter-tuning"></a>記錄用於超參數微調的計量
為了使用 Azure Machine Learning 服務微調超參數，模型之定型指令碼必須回報模型執行時的相關計量。 使用者會指定服務要用於評估執行效能的主要計量，而定型指令碼必須記錄此計量。 請參閱[指定要最佳化的主要計量](#specify-a-primary-metric-to-optimize)。

您可以使用下列範例程式碼片段，將定型指令碼更新為記錄此計量 -

```Python
from azureml.core.run import Run
run_logger = Run.get_submitted_run()
run_logger.log("accuracy", float(val_accuracy))
```

在此範例中，定型指令碼會計算 `val_accuracy` 並記錄此「正確性」，以用作主要計量。 模型開發人員必須決定回報此計量的頻率。

## <a name="specify-an-early-termination-policy"></a>指定提早終止原則
使用 Azure Machine Learning 服務微調超參數時，效能不佳的執行會自動提早終止。 這會減少資源浪費情形，並將這些資源改用於探索其他參數設定。

使用提早終止原則時，使用者可以設定下列參數，控制何時套用原則 -
* `evaluation_interval`：套用原則的頻率。 每次定型指令碼記錄主要計量都算是一個間隔。 因此，`evaluation_interval` 為 1 表示只要定型指令碼回報主要計量就套用原則。 `evaluation_interval` 為 2 表示會在定型指令碼回報主要計量的隔次套用原則。 這是選擇性參數；如果未指定，預設會將 `evaluation_interval` 設定為 1。
* `delay_evaluation`：將第一次原則評估延遲到指定間隔數目之後。 這是選擇性參數，允許所有設定執行最小的初始間隔數目，以避免定型執行過早終止。 如果指定，則會每隔 evaluation_interval (大於或等於 delay_evaluation) 的倍數套用原則一次。

Azure Machine Learning 服務支援下列提早終止原則 -

### <a name="bandit-policy"></a>Bandit 原則
Bandit 原則是以寬限時間因數/寬限時間數量及評估間隔為基準的終止原則。 如果任何執行的主要計量不在為取得效能最佳之定型執行所指定的寬限時間因數/寬限時間數量內，此原則會提早終止執行。 它接受下列設定參數 -
* `slack_factor` 或 `slack_amount`：為取得效能最佳之定型執行所允許的寬限時間。 `slack_factor` 將允許的寬限時間指定為小數比。 `slack_amount` 將允許的寬限時間指定為絕對數量，而不是小數比。

    例如，請思考在間隔 10 套用的 Bandit 原則。 假設在間隔 10 之效能最佳的執行回報主要計量為 0.8，而目標是要最大化主要計量。 如果指定 `slack_factor` 為 0.2 的原則，在間隔 10 之最佳計量小於 0.66 (0.8/(1+`slack_factor`)) 的任何定型執行都會終止。 相反地，如果指定 `slack_amount` 為 0.2 的原則，在間隔 10 之最佳計量小於 0.6 (0.8 - `slack_amount`) 的任何定型執行都會終止。
* `evaluation_interval`：套用原則的頻率 (選擇性參數)。
* `delay_evaluation`：將第一次原則評估延遲到指定間隔數目之後 (選擇性參數)。

請思考此範例 -

```Python
from azureml.train.hyperdrive import BanditPolicy
early_termination_policy = BanditPolicy(slack_factor = 0.1, evaluation_interval=1, delay_evaluation=5)
```

在此範例中，自評估間隔 5 起，每隔一段時間回報計量時，就會套用提早終止原則。 如果任何執行的最佳計量小於效能最佳之執行的 (1/(1+0.1) 或 91%，該執行將會終止。

### <a name="median-stopping-policy"></a>中位數停止原則
中位數停止原則是以執行所回報之主要計量移動平均為基準的提早終止原則。 此原則會跨所有定型執行與終止執行計算移動平均，這些執行的效能比移動平均的中位數還要差。 此原則接受下列設定參數 -
* `evaluation_interval`：套用原則的頻率 (選擇性參數)。
* `delay_evaluation`：將第一次原則評估延遲到指定間隔數目之後 (選擇性參數)。

請思考此範例 -

```Python
from azureml.train.hyperdrive import MedianStoppingPolicy
early_termination_policy = MedianStoppingPolicy(evaluation_interval=1, delay_evaluation=5)
```

在此範例中，自評估間隔 5 起，每隔一段時間，就會套用提早終止原則。 如果執行的最佳計量在所有定型執行之間比間隔 1:5 的移動平均中位數還要差，該執行將會在間隔 5 終止。

### <a name="truncation-selection-policy"></a>截斷選取原則
截斷選取原則會取消每個評估間隔效能最低之執行的指定百分比。 執行會根據其主要計量效能進行比較，並終止最低的 X%。 它接受下列設定參數 -
* `truncation_percentage`：每個評估間隔效能最低之執行所要終止的百分比。 這應該是介於 1 到 99 的整數。
* `evaluation_interval`：套用原則的頻率 (選擇性參數)。
* `delay_evaluation`：將第一次原則評估延遲到指定間隔數目之後 (選擇性參數)。

請思考此範例 -

```Python
from azureml.train.hyperdrive import TruncationSelectionPolicy
early_termination_policy = TruncationSelectionPolicy(evaluation_interval=1, truncation_percentage=20, delay_evaluation=5)
```

在此範例中，自評估間隔 5 起，每隔一段時間，就會套用提早終止原則。 如果執行在間隔 5 的效能落在間隔 5 之所有執行效能的最低 20%，該執行將會在間隔 5 終止。

### <a name="no-termination-policy"></a>無終止原則
如果您想要所有定型執行完成執行，請使用 NoTerminationPolicy。 其效果相當於不套用任何提早終止原則。 例如 

```Python
from azureml.train.hyperdrive import NoTerminationPolicy
early_termination_policy = NoTerminationPolicy()
```

### <a name="default-policy"></a>預設原則
如果未指定任何原則，超參數微調服務預設會使用 `evaluation_interval` 為 1 且 `delay_evaluation` 為 5 的中位數停止原則。 這些是保守的設定，可在不遺失主要計量的情況下省下約 25%-35% (取決於我們的評估資料)。

## <a name="allocate-resources-for-hyperparameter-tuning"></a>配置用於超參數微調的資源
您可以透過指定定型執行總數上限，並選擇性地指定超參數微調實驗的持續時間上限 (以分鐘為單位)，來控制超參數微調實驗的資源預算。 
* `max_total_runs`：要建立的定型執行總數上限。 這是上限 - 如果超參數空間有限並具有較少的樣本，可能會有較少的執行。 必須是介於 1 到 1000 的數字。
* `max_duration_minutes`：超參數微調實驗的持續時間上限 (以分鐘為單位)。 這是選擇性參數；如果存在，在此持續時間之後執行的任何執行都會自動取消。

>[!NOTE] 
>如果同時指定 `max_total_runs` 和 `max_duration_minutes`，在達到這兩個閾值的第一個時，就會終止超參數微調實驗。

此外，您可以指定要在超參數微調搜尋期間同時執行的定型執行數目上限。
* `max_concurrent_runs`：這是可在任何指定時間同時執行的執行數目上限。 如果未指定，則會同時啟動所有 `max_total_runs`。 如果指定，必須是介於 1 到 100 的數字。

>[!NOTE] 
>同時執行之數目會受限於指定計算目標中的可用資源。 因此，您必須確保計算目標具有可供所需並行處理使用的資源。

您可以配置用於超參數微調的資源，如此範例所示 -
```Python
max_total_runs=20,
max_concurrent_runs=4
```
這會將超參數微調實驗設定為使用最多 20 個執行，一次執行 4 個設定。

## <a name="configure-your-hyperparameter-tuning-experiment"></a>設定您的超參數微調實驗
您可以使用上述各節中定義的超參數搜尋空間、提早終止原則、主要計量和資源配置，來設定超參數微調實驗。 此外，您必須提供要透過取樣之超參數來呼叫的 `estimator`。 `estimator` 描述您執行的定型指令碼、每個作業的資源 (單一或多個 GPU)，以及要使用的計算資源。 由於超參數微調實驗的並行處理受限於可用的資源，因此您必須確保 `estimator` 中指定的計算目標具有所需並行處理的足夠資源 (如需估算器的詳細資訊，請參閱[如何將模型定型](how-to-train-ml-models.md))。

以下示範如何設定您的超參數微調實驗 -

```Python
from azureml.train.hyperdrive import HyperDriveRunConfig
hyperdrive_run_config = HyperDriveRunConfig(estimator=estimator,
                          hyperparameter_sampling=param_sampling, 
                          policy=early_termination_policy,
                          primary_metric_name="accuracy", 
                          primary_metric_goal=PrimaryMetricGoal.MAXIMIZE,
                          max_total_runs=100,
                          max_concurrent_runs=4)
```

## <a name="submit-your-hyperparameter-tuning-experiment"></a>提交您的超參數微調實驗
一旦定義超參數微調設定，您可以使用此設定來提交實驗 -

```Python
from azureml.core.experiment import Experiment
experiment = Experiment(workspace, experiment_name)
hyperdrive_run = experiment.submit(hyperdrive_run_config)
```

其中 `experiment_name` 是您要指派給超參數微調實驗的名稱，而 `workspace` 是您要建立實驗的工作區 (如需實驗的詳細資訊，請參閱[連結](/concept-azure-machine-learning-architecture.md))。

## <a name="visualize-your-hyperparameter-tuning-experiment"></a>視覺化您的超參數微調實驗
Azure Machine Learning SDK 提供 Notebook 小工具，可用來視覺化定型執行的進度。 您可以使用下列程式碼片段，在同一個位置視覺化您所有的超參數微調執行 -

```Python
from azureml.train.widgets import RunDetails
RunDetails(hyperdrive_run).show()
```

這會顯示一個表格，其中包含每個超參數設定的定型執行相關詳細資料。 例如

![超參數微調表](media/how-to-tune-hyperparameters/HyperparameterTuningTable.png)

您也可以依照定型進度視覺化每個執行的效能。 例如

![超參數微調圖](media/how-to-tune-hyperparameters/HyperparameterTuningPlot.png)

最後，您可以使用平行座標圖，以視覺化方式識別效能與個別超參數值之間的關聯性。 例如 

![超參數微調平行座標](media/how-to-tune-hyperparameters/HyperparameterTuningParallelCoordinates.png)

或者，您也可以在 Azure 入口網站中，視覺化您所有的超參數微調執行。 如需如何在入口網站中檢視實驗的詳細資訊，請參閱[連結](/how-to-track-experiments.md/#view-the-experiment-in-the-web-portal)。 例如 -

![超參數微調入口網站](media/how-to-tune-hyperparameters/HyperparameterTuningPortal.png)

## <a name="find-the-configuration-that-resulted-in-the-best-performance"></a>找到產生最佳效能的設定
一旦所有的超參數微調執行完成，您可以使用下列程式碼片段，識別效能最佳的設定和對應的超參數值 -

```Python
best_run = hyperdrive_run.get_best_run_by_primary_metric()
best_run_metrics = best_run.get_metrics()
parameter_values = best_run.get_details()['runDefinition']['Arguments']

print('Best Run Id: ', best_run.id)
print('\n Accuracy:', best_run_metrics['accuracy'])
print('\n learning rate:',parameter_values[3])
print('\n keep probability:',parameter_values[5])
print('\n batch size:',parameter_values[7])
```

## <a name="sample-notebook"></a>範例 Notebook
請參閱 
* `training/03.train-hyperparameter-tune-deploy-with-tensorflow/03.train-hyperparameter-tune-deploy-with-tensorflow.ipynb` 以取得為 Tensorflow 模型微調超參數的相關教學課程。 

請取得此筆記本：

[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>後續步驟
* [追蹤實驗](how-to-track-experiments.md)
* [部署定型的模型](how-to-deploy-and-where.md) (英文)
