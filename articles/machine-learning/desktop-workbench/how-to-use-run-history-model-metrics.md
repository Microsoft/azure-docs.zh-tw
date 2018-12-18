---
title: 如何在 Azure Machine Learning Workbench 中使用執行歷程記錄和模型計量 | Microsoft Docs
description: 使用 Azure Machine Learning Workbench 的執行歷程記錄和模型計量功能指南
services: machine-learning
author: rastala
ms.author: roastala
manager: haining
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 09/07/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 52b02460d444464211fc74c8982379424abebb5c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46965488"
---
# <a name="how-to-use-run-history-and-model-metrics-in-azure-machine-learning-workbench"></a>如何在 Azure Machine Learning Workbench 中使用執行歷程記錄和模型計量

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



Azure Machine Learning Workbench 透過其**執行歷程記錄**和**模型計量**功能支援資料科學實驗。
**執行歷程記錄**提供方法來追蹤您機器學習實驗的輸出，接著能夠篩選和比較其結果。
**模型計量**可從您指令碼的任何時間點進行記錄，以追蹤您資料科學實驗中最重要的值。
本文說明如何有效地運用這些功能，來增加資料科學實驗的速率及品質。

## <a name="prerequisites"></a>必要條件
若要逐步執行本作法指南，您需要：
* [建立與安裝 Azure Machine Learning](quickstart-installation.md)
- [建立專案](quickstart-installation.md)


## <a name="azure-ml-logging-api-overview"></a>Azure ML 記錄 API 概觀
[Azure ML 記錄 API](reference-logging-api.md) 會透過 Python (隨 Azure ML Workbench 一起安裝) 中的 **azureml.logging** 模組來提供使用。匯入此模組之後，您可以使用 **get_azureml_logger** 方法，來將 **logger** 物件具現化。
接著，您可以使用記錄器的 **log** 方法，儲存 Python 指令碼所產生的索引鍵/值組。
目前支援記錄純量和清單類型的模型計量，如下所示。

```Python
# create a logger instance in already set up environment 
from azureml.logging import get_azureml_logger
logger = get_azureml_logger()

# log scalar (any integer or floating point type is fine)
logger.log("simple value", 7)


# log list
logger.log("all values", [5, 6, 7])
```
您可以輕鬆地在 Azure ML Workbench 專案內使用記錄器，本文將說明如何執行這項操作。

## <a name="create-a-project-in-azure-ml-workbench"></a>在 Azure ML Workbench 中建立專案
如果您還沒有專案，您可以從[建立和安裝快速入門](quickstart-installation.md)建立一個。您可以從 [專案儀表板] 開啟[iris_sklearn.py] 指令碼 (如下所示)。

![從 [檔案] 索引標籤存取指令碼](media/how-to-use-run-history-model-metrics/how-to-use-run-history-model-metrics-01b.png)

您可以使用這個指令碼作為預期在 Azure ML 中實作模型計量記錄的指南。

## <a name="parameterize-and-log-model-metrics-from-script"></a>從指令碼參數化和記錄模型計量
在 [iris_sklearn.py] 指令碼中，預期要在 Python 中匯入和建構記錄器的模式可縮減為下列程式碼行。

```Python
from azureml.logging import get_azureml_logger
run_logger = get_azureml_logger()
```

一旦建立之後，您就能搭配任何名稱/值組叫用 **log** 方法。

開發完成之後，將指令碼參數化通常很有用，因為可透過命令列傳入值。
下列範例示範如何使用標準的 Python 程式庫來接受命令列參數 (如果有的話)。
此指令碼採用「正則化速率」(reg) 的單一參數，可用其來符合試圖提高「精確度」但不會過度學習的分類模型。
這些變數接著會記錄為「正則化速率」和「精確度」，如此就能輕鬆地識別出具有最佳結果的模型。

```Python
# change regularization rate and you will likely get a different accuracy.
reg = 0.01
# load regularization rate from argument if present
if len(sys.argv) > 1:
    reg = float(sys.argv[1])

print("Regularization rate is {}".format(reg))

# log the regularization rate
run_logger.log("Regularization Rate", reg)

# train a logistic regression model on the training set
clf1 = LogisticRegression(C=1/reg).fit(X_train, Y_train)
print (clf1)

# evaluate the test set
accuracy = clf1.score(X_test, Y_test)
print ("Accuracy is {}".format(accuracy))

# log accuracy
run_logger.log("Accuracy", accuracy)
```

在您的指令碼中執行這些步驟，讓它們能夠做出**執行歷程記錄**的最佳使用量。

## <a name="launch-runs-from-project-dashboard"></a>從專案儀表板啟動執行
返回 [專案儀表板]，您可以啟動 [追蹤的執行]，方法是選取 [iris_sklearn.py] 指令碼，然後在 [引數] 編輯方塊中輸入**正則化速率**參數。

![輸入參數並啟動執行](media/how-to-use-run-history-model-metrics/how-to-use-run-history-model-metrics-05.png)

由於啟動追蹤的執行不會封鎖 Azure ML Workbench，因此可以平行方式啟動數個。
每個追蹤執行的狀態都會顯示於 [作業] 面板中，如下所示。

![在 [作業] 面板中追蹤執行](media/how-to-use-run-history-model-metrics/how-to-use-run-history-model-metrics-06.png)

這會啟用最佳資源利用率，而不需要以序列方式執行每個作業。

## <a name="view-results-in-run-history"></a>在執行歷程記錄中檢視結果
追蹤執行的進度和結果可在 Azure ML Workbench 的 [執行歷程記錄] 中用於分析。
**執行歷程記錄**提供三種不同的檢視：
- 儀表板
- 詳細資料
- 比較

[儀表板] 檢視可跨指定指令碼的所有執行顯示資料，並以圖形和表格形式來轉譯。
[詳細資料] 檢視會顯示從指定指令碼的特定執行產生的所有資料，包括記錄的計量和輸出檔 (例如，轉譯的繪圖)。[比較] 檢視可讓要檢視的兩或三個執行的結果並排顯示，同時也包括記錄的計量與輸出檔。

跨 **iris_sklearn.py** 的八個追蹤執行，記錄了**正則化速率**參數和**精確度**結果的值，以說明如何使用 [執行歷程記錄] 檢視。

### <a name="run-history-dashboard"></a>執行歷程記錄儀表板
八個執行的結果全都會顯示於 [執行歷程記錄儀表板] 中。
由於 **iris_sklearn.py** 會記錄「正則化速率」和「精確度」，因此 [執行歷程記錄儀表板] 預設會顯示這些值的圖表。

![執行歷程記錄儀表板](media/how-to-use-run-history-model-metrics/how-to-use-run-history-model-metrics-07.png)

[執行歷程記錄儀表板] 可以自訂，因此，記錄的值也會出現在方格中。  按一下 [自訂] 圖示，即會顯示 [清單檢視自訂] 對話方塊，如下所示。

![自訂執行歷程記錄儀表板方格](media/how-to-use-run-history-model-metrics/how-to-use-run-history-model-metrics-08.png)

追蹤執行期間所記錄的任何值都可顯示，而選取 [正則化速率] 和 [精確度] 會將它們新增至方格。

![自訂方格中記錄的值](media/how-to-use-run-history-model-metrics/how-to-use-run-history-model-metrics-09.png)

您可以將滑鼠停留在圖表中的點上，輕鬆地尋找感興趣的執行。  在此案例中，執行 7 產生了良好的精確度且較低的持續時間。

![尋找感興趣的執行](media/how-to-use-run-history-model-metrics/how-to-use-run-history-model-metrics-10.png)

按一下任意圖表內與執行 7 相關聯的點或方格中連至執行 7 的連結，即會顯示 [執行歷程記錄詳細資料]。

### <a name="run-history-details"></a>執行歷程記錄詳細資料
這個檢視會顯示執行 7 的完整結果以及執行 7 所產生的任何成品。

![執行歷程記錄詳細資料](media/how-to-use-run-history-model-metrics/how-to-use-run-history-model-metrics-11.png)

[執行歷程記錄詳細資料] 檢視也會提供功能來**下載**要寫入 **./outputs** 資料夾的任何檔案 (這些檔案都受到 Azure ML Workbench 中適用於執行歷程記錄的雲端儲存體所支援，此為另一篇文章的主旨)。

最後，[執行歷程記錄詳細資料] 提供方法來將您的專案還原為它在此執行期間的狀態。
按一下 [還原] 按鈕，即會顯示確認對話方塊，如下所示。

![還原執行確認](media/how-to-use-run-history-model-metrics/how-to-use-run-history-model-metrics-13.png)

如果確認，可能會覆寫或移除檔案，因此請小心使用此功能。

### <a name="run-history-comparison"></a>執行歷程記錄比較
在 [執行歷程記錄儀表板] 中選取兩或三個執行，然後按一下 [比較]，即會將您帶到 [執行歷程記錄比較] 檢視。
或者，按一下 [比較]，然後選取 [執行歷程記錄詳細資料] 檢視內的執行，也會將您帶到 [執行歷程記錄比較] 檢視。
在任一情況下，[執行歷程記錄比較] 檢視都會提供方法，以便並排顯示兩或三個執行的記錄結果或成品。

![執行歷程記錄比較](media/how-to-use-run-history-model-metrics/how-to-use-run-history-model-metrics-12.png)

此檢視特別適用於繪圖的比較，但一般而言，執行的任何屬性均可在此處進行比較。

### <a name="command-line-interface"></a>命令列介面
Azure Machine Learning Workbench 也會透過其**命令列介面**提供執行歷程記錄的存取。
若要存取 [命令列介面]，按一下 [開啟命令提示字元] 功能表，如下所示。

![開啟命令提示字元](media/how-to-use-run-history-model-metrics/how-to-use-run-history-model-metrics-14.png)

適用於執行歷程記錄的命令可透過 `az ml history`，加上藉由新增 `-h` 旗標而取得的線上說明來存取。
```
$ az ml history -h

Group
    az ml history: View run history of machine learning experiments.

Commands:
    compare : Compare two runs.
    download: Download all the artifacts from a run into the destination path.
    info    : Details about one run.
    last    : Get detail about most recent run.
    list    : List runs.
    promote : Promote Artifacts.
```
這些命令會提供 [執行歷程記錄檢視] 顯示的相同功能並傳回相同資料。
例如，最後一個執行的結果可顯示為 JSON 物件。
```
$ az ml history last
{
  "Accuracy": 0.6792452830188679,
  "Regularization Rate": 0.078125,
  "attachments": "control_log, control_log.txt, driver_log, driver_log.txt, pid.txt, dataprep/backgroundProcess.log, dataprep/backgroundProcess_Engine.log, dataprep/backgroundProcess_EngineHost.log, dataprep/backgroundProcess_ProjectProvider.log, dataprep/backgroundProcess_Sampling.log, dataprep/backgroundProcess_Telemetry.log, outputs/cm.png, outputs/model.pkl, outputs/sdk_debug.txt, outputs/roc.png",
  "created_utc": "2017-09-08T00:58:01.611105+00:00",
  "description": null,
  "duration": "0:00:04.892389",
  "end_time_utc": "2017-09-08T00:58:07.951403+00:00",
  "experiment_id": "ce92d0a9-3e2c-4d51-85de-93ef22249ce1",
  "heartbeat_enabled": true,
  "hidden": false,
  "name": null,
  "parent_run_id": null,
  "properties": {
    "CommitId": "e77a5f0df2af1a482bbe39b70bfbb16b62228cb3"
  },
  "run_id": "IrisDemo_1504832281116",
  "run_number": 8,
  "script_name": "iris_sklearn.py",
  "start_time_utc": "2017-09-08T00:58:03.059014+00:00",
  "status": "Completed",
  "target": "local",
  "user_id": "e9fafe06-b0e4-4154-8374-aae34f9977b2"
}
```
此外，所有執行的清單均可以表格格式來顯示。
```
$ az ml history list -o table
  Accuracy    Regularization Rate  Duration        Run_id                  Script_name      Start_time_utc                    Status
----------  ---------------------  --------------  ----------------------  ---------------  --------------------------------  ---------
  0.679245               0.078125  0:00:04.892389  IrisDemo_1504832281116  iris_sklearn.py  2017-09-08T00:58:03.059014+00:00  Completed
  0.679245               0.15625   0:00:04.734956  IrisDemo_1504832255198  iris_sklearn.py  2017-09-08T00:57:38.507196+00:00  Completed
  0.660377               0.3125    0:00:04.913762  IrisDemo_1504832234904  iris_sklearn.py  2017-09-08T00:57:16.849881+00:00  Completed
  0.660377               0.625     0:00:06.107764  IrisDemo_1504832210767  iris_sklearn.py  2017-09-08T00:56:54.602813+00:00  Completed
  0.641509               1.25      0:00:04.742727  IrisDemo_1504832171373  iris_sklearn.py  2017-09-08T00:56:13.879280+00:00  Completed
  0.660377               2.5       0:00:04.915401  IrisDemo_1504832148526  iris_sklearn.py  2017-09-08T00:55:52.937083+00:00  Completed
  0.641509               5         0:00:04.730627  IrisDemo_1504832127172  iris_sklearn.py  2017-09-08T00:55:29.612382+00:00  Completed
  0.641509              10         0:00:06.059082  IrisDemo_1504832109906  iris_sklearn.py  2017-09-08T00:55:14.739806+00:00  Completed

```
**命令列介面**是用來存取 Azure Machine Learning Workbench 功能的替代路徑。

## <a name="next-steps"></a>後續步驟
這些功能都可用來協助進行資料科學實驗程序。
我們希望您能夠發現它們很有用，並且非常感謝您提供意見。
這是只我們最初的實作，而我們規劃了大量的增強功能。
我們期望能夠持續將它們傳送至 Azure Machine Learning Workbench 中。 
