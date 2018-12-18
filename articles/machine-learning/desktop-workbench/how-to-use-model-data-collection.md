---
title: 在 Azure Machine Learning Workbench 中使用模型資料收集功能 | Microsoft Docs
description: 本文章說明如何在 Azure Machine Learning Workbench 中使用模型資料收集功能
services: machine-learning
author: aashishb
ms.author: aashishb
manager: hjerez
ms.reviewer: jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 09/12/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 435ef29192682f1bf19a9a53923dd043676256ab
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46954937"
---
# <a name="collect-model-data-by-using-data-collection"></a>使用資料收集來收集模型資料

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

您可以使用 Azure Machine Learning 中的模型資料收集功能，以封存來自 Web 服務的模型輸入和預測。

## <a name="install-the-data-collection-package"></a>安裝資料收集套件
您可以在於 Linux 和 Windows 上使用原生方式安裝資料收集程式庫。

### <a name="windows"></a>Windows
在 Windows 上，可以使用下列命令來安裝資料收集器模組：

    pip install azureml.datacollector

### <a name="linux"></a>Linux
在 Linux 上，請先安裝 libxml++ 程式庫。 執行下列命令，此命令必須在 sudo 下方發出：

    sudo apt-get install libxml++2.6-2v5

然後，執行下列命令：

    pip install azureml.datacollector

## <a name="set-environment-variables"></a>設定環境變數

模型資料收集取決於兩個環境變數。 AML_MODEL_DC_STORAGE_ENABLED 必須設為 **true** (全部小寫)，且 AML_MODEL_DC_STORAGE 必須設為要儲存資料之 Azure 儲存體帳戶的連接字串。

在 Web 服務是在 Azure 中的叢集上執行時，系統將會為您設定這些環境變數。 在本機執行時，您必須自行設定它們。 如果您使用 Docker，請使用 docker run 命令的 -e 參數來傳遞環境變數。

## <a name="collect-data"></a>收集資料

若要使用模型資料收集，請對計分檔進行下列變更：

1. 在檔案開頭處新增下列程式碼：
   
    ```python
    from azureml.datacollector import ModelDataCollector
    ```

1. 將下列程式碼行新增至 `init()` 函式：
    
    ```python
    global inputs_dc, prediction_dc
    inputs_dc = ModelDataCollector('model.pkl',identifier="inputs")
    prediction_dc = ModelDataCollector('model.pkl', identifier="prediction")
    ```

1. 將下列程式碼行新增至 `run(input_df)` 函式：
    
    ```python
    global inputs_dc, prediction_dc
    inputs_dc.collect(input_df)
    prediction_dc.collect(pred)
    ```

    請確定變數 `input_df` 和 `pred` (來自 `model.predict()` 的預測值) 會先初始化，然後您才能在其上呼叫 `collect()` 函式。

1. 使用 `az ml service create realtime` 命令搭配 `--collect-model-data true` 參數，以建立即時 Web 服務。 此步驟可確保會在服務執行時收集模型資料。

     ```batch
    c:\temp\myIris> az ml service create realtime -f iris_score.py --model-file model.pkl -s service_schema.json -n irisapp -r python --collect-model-data true 
    ```
    
1. 若要測試資料收集，請執行 `az ml service run realtime` 命令：

    ```
    C:\Temp\myIris> az ml service run realtime -i irisapp -d "ADD YOUR INPUT DATA HERE!!" 
    ``` 
    
## <a name="view-the-collected-data"></a>檢視收集的資料
檢視 Blob 儲存體中收集的資料：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 選取 [所有服務]。
1. 在 [搜尋] 方塊中，輸入**儲存體帳戶**，然後選取 Enter 鍵。
1. 從 [儲存體帳戶] 搜尋刀鋒視窗中，選取 [儲存體帳戶] 資源。 若要判斷您的儲存體帳戶，請使用下列步驟：

    a. 移至 Azure Machine Learning Workbench，選取您正在處理的專案，然後從 [檔案] 功能表開啟命令提示字元。
    
    b. 輸入 `az ml env show -v`，並檢查 *storage_account* 值。 這是您儲存體帳戶的名稱。

1. 選取 [資源] 刀鋒視窗功能表上的 [容器]，然後選取名為 **modeldata** 的容器。 若要看到資料開始傳播至儲存體帳戶，您可能需要在第一個 Web 服務要求之後最多等候 10 分鐘。 資料會透過下列容器路徑流入 Blob：

    `/modeldata/<subscription_id>/<resource_group_name>/<model_management_account_name>/<webservice_name>/<model_id>-<model_name>-<model_version>/<identifier>/<year>/<month>/<day>/data.csv`

您可以使用 Microsoft 軟體和開放原始碼工具，透過各種不同方式從 Azure Blob 取用資料。 這裡有一些範例：
- Azure Machine Learning Workbench：藉由將 .csv 檔案新增為資料來源，在 Azure Machine Learning Workbench 中開啟 .csv 檔案。
- Excel：開啟每日的 .csv 檔案作為試算表。
- [Power BI](https://powerbi.microsoft.com/documentation/powerbi-azure-and-power-bi/)：使用從 Blob 中之 .csv 資料提取的資料建立圖表。
- [Spark](https://docs.microsoft.com/azure/hdinsight/hdinsight-apache-spark-overview)：使用大部分的 .csv 資料建立資料框架。
    ```python
    var df = spark.read.format("com.databricks.spark.csv").option("inferSchema","true").option("header","true").load("wasb://modeldata@<storageaccount>.blob.core.windows.net/<subscription_id>/<resource_group_name>/<model_management_account_name>/<webservice_name>/<model_id>-<model_name>-<model_version>/<identifier>/<year>/<month>/<date>/*")
    ```
- [Hive](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-linux-tutorial-get-started)：將 .csv 資料載入至 Hive 資料表，並直接針對 Blob 執行 SQL 查詢。

