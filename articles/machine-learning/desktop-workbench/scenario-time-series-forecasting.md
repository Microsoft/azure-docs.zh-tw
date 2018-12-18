---
title: 能源需求時間序列預測 | Microsoft Docs
description: 如何運用機器學習在 Azure Machine Learning Workbench 中預測能源需求時間序列。
services: machine-learning
documentationcenter: ''
author: anta
manager: ireiter
editor: anta
ms.assetid: ''
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/15/2017
ms.author: anta
ROBOTS: NOINDEX
ms.openlocfilehash: 934d37783165c2e57dcabd0ff764747e1ab4b65e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946849"
---
# <a name="energy-demand-time-series-forecasting"></a>能源需求時間序列預測

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 




時間序列預測這項工作可在按時間排序的觀察值序列中預測未來值。 這是常見的問題，且運用於許多產業中。 例如，零售公司需要預測未來的產品銷售，以便有效地組織其供應鏈來滿足需求。 同樣地，包裹遞送公司需要評估其服務的需求，以便事先規劃人力需求和遞送路線。 在許多情況下，預測失準的財務風險可能很明顯。 因此，預測通常是商務關鍵活動。

這個範例示範如何透過運用機器學習技術來執行時間序列預測。 系統會引導您執行模型化程序的每個步驟，包括：
- 資料準備，以清理及格式化資料；
- 從未經處理的時間序列資料，建立機器學習模型的特徵；
- 訓練各種機器學習模型；
- 藉由比較對於留出測試資料集的效能來評估模型；以及
- 將最佳模型運算化，使其能透過 Web 服務提供以產生隨選預測。

Azure Machine Learning Workbench 有助於每個步驟的模型化程序： 
- 此範例示範 Jupyter Notebook 環境 (直接透過 Workbench 取得) 如何讓 Python 程式碼開發變得更簡單。 使用 Markdown 附註和圖表，可以更清楚地對給其他人說明模型開發程序。 從 Workbench 可以直接檢視、編輯及執行這些 Notebook。
- 定型的模型可以保存下來並上傳至 blob 儲存體。 這可協助資料科學家追蹤定型的模型物件，並確定它們已定型並可在需要時擷取。
- 執行 Python 指令碼時可以記錄計量，讓資料科學家得以保留模型效能分數的記錄。
- Workbench 會產生可自訂的已記錄計量資料表，讓資料科學家能輕鬆地比較模型效能計量。 系統會自動顯示圖表，以便立即識別最佳執行模型。
- 最後，此範例示範如何在即時 Web 服務中部署定型的模型，進而將該模型運算化。

## <a name="link-to-the-gallery-github-repository"></a>連結至資源庫 GitHub 存放庫
此實際案例的公用 GitHub 存放庫包含此範例所需的所有資料，包括程式碼範例：

[https://github.com/Azure/MachineLearningSamples-EnergyDemandTimeSeriesForecasting](https://github.com/Azure/MachineLearningSamples-EnergyDemandTimeSeriesForecasting)


## <a name="use-case-overview"></a>使用案例概觀

這個案例著重於能源需求預測，其目的在於預測未來的能源網格負載。 對於能源部門的公司而言，這是商務關鍵作業，因為操作員需要維護網格所耗用的能源與供應給網格的能源間的微妙平衡。 供應太多電力給網格會造成能源浪費或技術錯誤。 不過，如果供應太少電力，則可能會導致停電，讓客戶沒有電力。 一般而言，網格操作員可以採取短期決策，管理網格的能源供應並保持負載平衡。 因此，必須有能源需求的精確短期預測，操作員才能有信心地進行這些決策。

此案例詳述機器學習能源需求預測解決方案的建構。 此解決方案是在 [New York Independent System Operator (NYISO)](http://www3.dps.ny.gov/W/PSCWeb.nsf/All/298372E2CE4764E885257687006F39DF?OpenDocument) 的公用資料集進行訓練，該公司負責經營紐約州的電力網格。 此資料集包含紐約市過去五年的每小時電力需求資料。 另外包含紐約市同期每小時天氣狀況的資料集是取自 [darksky.net](https://darksky.net)。

## <a name="prerequisites"></a>必要條件

- [Azure 帳戶](https://azure.microsoft.com/free/) (有提供免費試用)。
- 遵循[快速入門安裝指南](quickstart-installation.md)所安裝的 [Azure Machine Learning Workbench](../service/overview-what-is-azure-ml.md) 版本，以便安裝程式並建立 Workbench。
- 此範例假設您在 Windows 10 (已於本機安裝 [Docker 引擎](https://www.docker.com/)) 上執行 Azure ML Workbench。 如果您是使用 macOS，指示大致相同。
- 隨著本機部署環境設定安裝的 Azure Machine Learning Operationalization，以及依本[指南](./model-management-configuration.md)所述建立的模型管理帳戶。
- 這個範例要求您將 Pandas 安裝更新為 0.20.3 版或更新版本，並且安裝 matplotlib。 從 Workbench 的 [檔案] 功能表按一下 [開啟命令提示字元]，並執行下列命令來安裝這些相依性：

    ```
    conda install "pandas>=0.21.1"
    ```
    
## <a name="create-a-new-workbench-project"></a>建立新的 Workbench 專案

使用此範例作為範本來建立新專案：
1.  開啟 Azure Machine Learning Workbench
2.  在 [專案] 頁面上，按一下 **+** 符號，然後選取 [新增專案]
3.  在 [建立新專案] 窗格中，填入新專案的資訊
4.  在 [搜尋專案範本] 搜尋方塊中，輸入「能源需求時間序列預測」並選取範本
5.  按一下 [建立] 


## <a name="data-description"></a>資料說明

我們透過這個範例提供了兩個資料集，並且使用 `1-data-preparation.ipynb` 筆記型電腦下載：`nyc_demand.csv` 和 `nyc_weather.csv`。

**nyc_demand.csv** 包含紐約市 2012-2017 年的能源需求值。 此資料具有下列簡單的結構：

| timeStamp | demand |
| --- | --- |
| 2012-01-01 00:00:00 | 4937.5 |
| 2012-01-01 01:00:00 | 4752.1 |
| 2012-01-01 02:00:00 | 4542.6 |
| 2012-01-01 03:00:00 | 4357.7 |

需求值以兆瓦時 (MWh) 為單位。 以下是 2017 年 7 月為期 7 天的能源需求圖表：

![能源需求](./media/scenario-time-series-forecasting/energy_demand.png  "能源需求")

**nyc_weather.csv** 包含紐約市 2012-2017 年的每小時天氣值：

| timeStamp | precip | temp
| --- | --- | --- |
| 2012-01-01 00:00:00 | 0.0 | 46.13 |
| 2012-01-01 01:00:00 | 0.01 | 45.89 |
| 2012-01-01 02:00:00 | 0.05 | 45.04 |
| 2012-01-01 03:00:00 | 0.02 | 45.03 |

*precip* 是降雨程度的百分比測量。 *temp* (溫度) 值已重新調整，讓所有的值都落在間隔 [0, 100] 內。

## <a name="scenario-structure"></a>案例結構

當您第一次在 Azure Machine Learning Workbench 中開啟此專案時，請瀏覽至左側的 [檔案] 窗格。 下列程式碼檔案會隨著這個範例提供：
- `1-data-preparation.ipynb` - 此 Jupyter Notebook 會下載和處理資料，準備進行模型化。 這是您將執行的第一個 Notebook。
- `2-linear-regression.ipynb` - 此 Notebook 會在定型的資料上訓練線性迴歸模型。
- `3-ridge.ipynb` - 訓練山脊型迴歸模型。
- `4-ridge-poly2.ipynb` - 在二階多項式特徵上訓練山脊型迴歸模型。
- `5-mlp.ipynb` - 訓練多層式認知類神經網路。
- `6-dtree.ipynb` - 訓練決策樹模型。
- `7-gbm.ipynb` - 訓練梯度提升機器模型。
- `8-evaluate-model.py` - 此指令碼可載入定型的模型，並將它用於對留出測試資料集進行預測。 它可產生模型評估計量，因此可以比較不同模型的效能。
- `9-forecast-output-exploration.ipynb` - 此 Notebook 會產生機器學習模型所產生之預測的視覺效果。
- `10-deploy-model.ipynb` - 此 Notebook 示範如何在即時 Web 服務中將定型的模型運算化。
- `evaluate-all-models.py` - 此指令碼可評估所有定型的模型。 它會提供針對每個的定型模型個別執行 `8-evaluate-model.py` 的替代方式。

### <a name="1-data-preparation-and-cleaning"></a>1.資料準備和清除

若要開始執行範例，先按一下 `1-data-preparation.ipynb` 以在預覽模式中開啟 Notebook。 按一下 [啟動 Notebook 伺服器]，以在您的電腦上啟動 Jupyter Notebook 伺服器和 Python 核心。 您可以從 Workbench 執行 Notebook，或者可以使用瀏覽器瀏覽至 [http://localhost:8888](http://localhost:8888)。 請注意，您必須將核心變更為 PROJECT_NAME local。 您可以在 Notebook 中從 [核心] 功能表的 [變更核心] 之下執行此作業。 按 ***shift+Enter*** 可在個別 Notebook 資料格中執行程式碼，或按一下 [資料格] 功能表中的 [全部執行] 可執行整個 Notebook。

Notebook 會先從裝載於 Azure 的 blob 儲存體容器下載資料。 資料會儲存在電腦的 `AZUREML_NATIVE_SHARE_DIRECTORY` 中。 從任何您從 Workbench 執行的 Notebook 或指令碼均可存取這個位置，所以這是儲存資料和定型模型的好位置。

下載資料後，Notebook 會填滿時間序列中的間隙，並以插補方式填入遺漏值，藉此清除資料。 以此方式清除時間序列資料，可讓訓練模型可用的資料量最大化。

從已清除的未經處理資料會建立數個模型特徵。 這些特徵可以分成兩個群組：

- **時間驅動特徵**衍生自 timestamp 變數，例如 month、dayofweek 和 hour。
- **延隔特徵**是實際需求或溫度值的時間偏移值。 這些特徵旨在擷取模型中連續時間週期之間的條件式相依性。

然後在開發預測模型時，可以使用所產生的特徵資料集。

### <a name="2-model-development"></a>2.模型開發

第一個模型化方法可能是簡單的線性迴歸模型。 `2-linear-regression.ipynb` Notebook 會顯示如何訓練未來能源需求的線性迴歸預測模型。 我們將特別訓練此模型，以預測目前時間週期 (*t*) 前一小時 (*t+1*) 的能源需求。 但是，我們可以在測試階段以遞迴方式套用此「單一步驟」模型，以產生未來時間週期 *t+n* 的預測。

為了訓練模型，我們會建立牽涉三個步驟的預測性管線：

- **資料轉換**：視機器學習演算法而定，輸入資料所需的格式有所不同。 在此情況下，線性迴歸模型需要一個類別變數才能進行獨熱 (One-hot) 編碼。
- **交叉驗證常式**：機器學習模型通常會有一或多個需要透過實驗微調的超參數。 交叉驗證可用來尋找一組最佳的參數值。 此模型會在不同折 (fold) 的資料集上重複進行訓練和評估。 最佳參數就是在平均分配於交叉驗證折時達到最佳模型效能的參數。 `2-linear-regression.ipynb` 會更詳細說明此程序。
- **訓練最終模型**：使用最佳的超參數集，在整個資料集上訓練模型。

我們在 Notebook 編號 2-7 中包含一系列的 6 個不同模型。 每個 Notebook 會訓練不同的模型，並將其儲存在 `AZUREML_NATIVE_SHARE_DIRECTORY` 位置。 在您訓練為此案例開發的所有模型後，我們可以如下一節所述來評估和比較這些模型。

### <a name="3-model-evaluation-and-comparison"></a>3.模型評估和比較

我們可以使用定型的模型來進行未來時間週期的預測。 最好根據留出測試資料集來評估這些模型。 藉由根據相同的未顯示資料集來評估不同的模型，我們可以公平地比較其效能，並找出最佳模型。 在此案例中，我們以遞迴方式套用定型的模型，以預測未來的多個時間步距。 特別的是，我們會針對目前小時 *t* 的前六小時 *t+6* 產生預測。 這些預測會針對每個時間期間觀察到的實際需求進行評估。

若要評估模型，請從 Workbench 中的 [檔案] 窗格開啟 `8-evaluate-model.py` 指令碼。 請檢查 [執行組態] 是否設為 [本機]，然後在 [引數] 欄位中輸入模型名稱。 模型名稱必須完全符合在訓練模型的 Notebook 開頭設定的 *model_name* 變數。 例如，輸入 "linear_regression" 來評估定型的線性迴歸模型。 或者，訓練所有模型後，您可執行 `evaluate-all-models.py` 以一個命令評估所有的模型。 若要執行此指令碼，請從 Workbench 開啟命令提示字元並執行下列命令：

```
python evaluate-all-models.py
```
`8-evaluate-model.py` 指令碼會執行下列作業：

- 從磁碟載入定型的模型管線
- 根據測試資料進行預測
- 計算模型效能計量和並加以記錄
- 將測試資料集預測儲存至 `AZUREML_NATIVE_SHARE_DIRECTORY` 以供日後檢查和分析
- 將定型的模型管線儲存至 *outputs* 資料夾。

> [!Note]
> 將模型儲存至 *outputs* 資料夾，可將模型物件自動複製到與您的實驗帳戶相關聯的 Azure Blob 儲存體帳戶。 這表示您一律能夠從 blob 擷取已儲存的模型物件，即使您變更電腦或計算內容。

瀏覽至 [執行歷程記錄] 窗格並按一下 `8-evaluate-model.py`。 您會看到顯示數個模型效能計量的圖表：

- **ME**：預測的平均誤差。 這可以解譯為預測的平均偏差。
- **MPE**：[平均百分比誤差](https://en.wikipedia.org/wiki/Mean_percentage_error) (ME 會以實際需求的百分比表示)
- **MSE**：[均方差](https://en.wikipedia.org/wiki/Mean_squared_error)
- **RMSE**：均方根誤差 (MSE 的方根)
- **MAPE**：[平均絕對百分比誤差](https://en.wikipedia.org/wiki/Mean_absolute_percentage_error)
- **sMAPE**：[對成平均絕對百分比誤差](https://en.wikipedia.org/wiki/Symmetric_mean_absolute_percentage_error)
- **MAPE_base**：基準預測的 MAPE，其預測等於最後一個已知的需求值。
- **MdRAE**：相對絕對誤差中位數。 預測錯誤與基準預測誤差的中位數比例。 值小於 1 表示預測的效能優於基準。

上述所有計量都會參考 *t+1* 預測。 除了上述的計量，您也會看到一組具有 *_horizon* 後置詞的對應計量。 這是平均分配於預測範圍 (從值間 *t+1* 至期間 *t+6*) 中所有期間的計量。

如果計量未顯示在圖表區域中，請按一下右上角的齒輪符號。 確定已選取您感興趣的模型效能計量。 度量也會出現在圖表下方的資料表中。 同樣地，按一下齒輪符號即可自訂此資料表。 依效能計量來排序資料表，以找出最佳模型。 如果您有興趣查看從期間 *t+1* 至 *t+6* 的預測效能變化，請按一下資料表中的模型項目。 顯示預測期間內 MAPE、MPE 和 MdRAE 計量的圖表會顯示在 [視覺效果] 之下。

在評估預測模型時，它很適合用來將輸出預測視覺化。 這可協助資料科學家判斷所產生的預測是否切合實際。 它也有助於識別預測的問題，例如，預測在某些時間週期中效能不佳。 `9-forecast-output-exploration.ipynb` Notebook 會製作針對測試資料集產生之預測的視覺效果。

### <a name="4-deployment"></a>4.部署

將最佳模型部署為即時 Web 服務，即可將該模型運算化。 然後可叫用此 Web 服務，以便在有新資料可用時產生隨選預測。 在此案例中，必須每小時產生新的預測，才能預測後續一小時的能源需求。 若要執行這項工作，可以部署 Web 服務，該服務會以指定小時時間週期的特徵陣列作為輸入，並傳回預測的需求作為輸出。

在此範例中，Web 服務會部署到 Windows 10 電腦。 確定您已完成在 Operationalization CLI 的[入門指南](https://github.com/Azure/Machine-Learning-Operationalization/blob/master/documentation/getting-started.md)中針對本機部署設定的必要步驟。 設定本機環境和模型管理帳戶後，請執行 `10-deploy-model.ipynb` Notebook 來部署 Web 服務。

## <a name="conclusion"></a>結論

這個範例示範如何建置端對端時間序列預測解決方案，以便預測能源需求。 在此範例中探索的許多原則都可以延伸至其他預測案例和產業。

此案例示範 Azure Machine Learning Workbench 如何協助資料科學家開發具有實用功能 (例如 Jupyter Notebook 環境和計量記錄功能) 的真實解決方案。 此範例也會引導讀者如何使用 Azure Machine Learning Operationalization CLI 來運算化和部署模型。 部署之後，Web 服務 API 可讓開發人員或資料工程師將預測模型整合到較寬的資料管線中。
