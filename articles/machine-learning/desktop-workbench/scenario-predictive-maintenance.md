---
title: 真實案例的預測性維護 | Microsoft Docs
description: 使用 PySpark 之真實案例的預測性維護
services: machine-learning
author: ehrlinger
ms.author: jehrling
manager: jhubbard
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.custom: mvc
ms.date: 10/05/2017
ROBOTS: NOINDEX
ms.openlocfilehash: c154b0124acb5bee93211adb611356555526d2c0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996208"
---
# <a name="predictive-maintenance-for-real-world-scenarios"></a>真實案例的預測性維護

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



對所有企業來說，設備在非排程的時間停機將會造成不利影響。 讓現場設備保持運作以充分發揮使用率和效能，並將代價高昂的排程外停機時間降到最低，至關重要。 及早發現問題有助於以更符合成本效益的方式配置為數不多的維護資源，並增強品質和供應鏈流程。 

此案例探索相對[大規模模擬資料集](https://github.com/Microsoft/SQL-Server-R-Services-Samples/tree/master/PredictiveMaintanenceModelingGuide/Data)，以從資料擷取、功能工程、模型建置以及模型運作和部署來逐步解說預測性維護資料科學專案。 整個程序的程式碼是在 Azure Machine Learning Workbench 中利用 PySpark 以 Jupyter Notebook 撰寫的。 部署最終模型時，會使用「Azure Machine Learning 模型管理」來部署，以進行即時設備故障預測。   

### <a name="cortana-intelligence-gallery-github-repository"></a>Cortana Intelligence 資源庫 GitHub 存放庫

適用於 PM 教學課程的 Cortana Intelligence 資源庫是公用 GitHub 存放庫 ([https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance](https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance))，您可以在此回報問題並進行貢獻。


## <a name="use-case-overview"></a>使用案例概觀

在擁有龐大固定資產的產業中，企業所面臨的其中一個主要問題，就是與因機械故障而導致生產延遲相關的重大成本。 大多數企業都想要了解如何預測這些問題的可能發生時機，以便事先主動預防。 目標是要藉由縮短停機時間及可能提升安全性來降低成本。 

此案例的構想來自[預測性維護腳本](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/cortana-analytics-playbook-predictive-maintenance)，用以示範如何為模擬的資料集建置預測性模型。 範例資料衍生自在許多預測性維護使用案例中觀察到的常見元素。

此模擬資料的企業問題是要預測元件故障所造成的問題。 企業問題是「機器因為元件故障而當機的機率有多高？」 這個問題會格式化為多類別分類問題 (每部機器多個元件)。 機器學習演算法會用來建立預測性模型。 我們會使用從機器收集到的歷史資料來訓練模型。 在此案例中，使用者會經歷各種步驟，以在 Machine Learning Workbench 環境中實作此模型。

## <a name="prerequisites"></a>必要條件

* [Azure 帳戶](https://azure.microsoft.com/free/) (有提供免費試用)。
* 已安裝 [Azure Machine Learning Workbench](../service/overview-what-is-azure-ml.md)。 請依照[快速入門安裝指南](quickstart-installation.md)來安裝程式及建立工作區。
* 「Azure Machine Learning 操作化」需要一個本機部署環境，以及一個 [Azure Machine Learning 模型管理帳戶](model-management-overview.md)。

此範例可在任何 Machine Learning Workbench 計算環境中執行。 不過，建議至少以 16 GB 的記憶體來執行它。 此案例已在執行遠端 DS4_V2 標準[適用於 Linux (Ubuntu) 的資料科學虛擬機器 (DSVM)](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.linux-data-science-vm-ubuntu) 的 Windows 10 電腦上建置並測試。

模型的操作化已使用 0.1.0a22 版的 Azure Machine Learning CLI 來完成。

## <a name="create-a-new-workbench-project"></a>建立新的 Workbench 專案

使用此範例作為範本來建立新專案：
1.  開啟 Machine Learning Workbench。
2.  在 [專案] 頁面上，選取 **+**，然後選取 [新增專案]。
3.  在 [建立新專案] 窗格中，填入新專案的資訊。
4.  在 [搜尋專案範本] 搜尋方塊中輸入「預測性維護」，然後選取 [預測性維護] 範本。
5.  選取 [建立] 。

## <a name="prepare-the-notebook-server-computation-target"></a>準備筆記本伺服器計算目標

若要在您的本機電腦上執行，請從 Machine Learning Workbench [檔案] 功能表中，選取 [開啟命令提示字元] 或 [開啟 PowerShell CLI]。 CLI 介面可讓您使用 `az` 命令來存取您的 Azure 服務。 首先，使用下列命令來登入 Azure 帳戶：

```
az login
``` 

這個命令會提供要與 https:\\aka.ms\devicelogin URL 搭配使用的驗證金鑰。 CLI 會等候直到裝置登入作業傳回並提供一些連線資訊。 接著，如果您已安裝本機 [Docker](https://www.docker.com/get-docker)，請使用下列命令來準備本機計算環境：

```
az ml experiment prepare --target docker --run-configuration docker
```

最好是在[適用於 Linux (Ubuntu) 的 DSVM](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.linux-data-science-vm-ubuntu) 上執行，以符合記憶體和磁碟需求。 設定 DSVM 之後，請使用下列兩個命令來準備遠端 Docker 環境：

```
az ml computetarget attach remotedocker --name [Connection_Name] --address [VM_IP_Address] --username [VM_Username] --password [VM_UserPassword]
```

在連線到遠端 Docker 容器之後，請使用下列命令來準備 DSVM Docker 計算環境： 

```
az ml experiment prepare --target [Connection_Name] --run-configuration [Connection_Name]
```

備妥 Docker 計算環境之後，請從 Azure Machine Learning Workbench 的 [筆記本] 索引標籤中開啟 Jupyter 筆記本伺服器，或是使用下列命令來啟動瀏覽器型伺服器： 

```
az ml notebook start
```

範例筆記本會儲存在 [Code] 目錄中。 筆記本會設定成從第一個 (Code\1_data_ingestion.ipynb) 筆記本開始循序執行。 當您開啟每個筆記本時，系統都會提示您選取計算核心。 請選擇 [Project_Name]_Template [Connection_Name] 核心，以在先前設定的 DSVM 上執行。

## <a name="data-description"></a>資料說明

[模擬資料](https://github.com/Microsoft/SQL-Server-R-Services-Samples/tree/master/PredictiveMaintanenceModelingGuide/Data)包含五個逗號分隔值 (.csv) 檔案。 請使用下列連結來取得資料集的相關詳細描述。

* [機器](https://pdmmodelingguide.blob.core.windows.net/pdmdata/machines.csv)：區別每部機器的特徵，例如存留期和型號。
* [錯誤](https://pdmmodelingguide.blob.core.windows.net/pdmdata/errors.csv)：此錯誤記錄檔包含機器仍在運作時所擲回的非重大錯誤。 系統並不將這些錯誤視為故障，但這些錯誤可能是未來故障事件的預兆。 由於遙測資料的收集頻率是每小時一次，因此系統會將錯誤的日期時間值四捨五入到最接近的整點。
* [維護](https://pdmmodelingguide.blob.core.windows.net/pdmdata/maint.csv)：維護記錄同時包含排程和非排程的維護記錄。 排定的維護會與定期的元件檢查對應。 排程外的維護可能起因於機械故障或其他效能降低情況。 由於遙測資料的收集頻率是每小時一次，因此系統會將維護的日期時間值四捨五入到最接近的整點。
* [遙測](https://pdmmodelingguide.blob.core.windows.net/pdmdata/telemetry.csv)：遙測資料是由每部機器內多個感應器的時間序列量值所組成。 資料是藉由平均每小時間隔的感應器值來記錄的。
* [故障](https://pdmmodelingguide.blob.core.windows.net/pdmdata/failures.csv)：故障會與維護記錄中的元件更換事件對應。 每一筆記錄都包含機器識別碼、元件類型和更換日期與時間。 我們會使用這些記錄來建立將由模型試著預測的機器學習標籤。

若要從 GitHub 存放庫下載未經處理的資料集，並建立要供此分析使用的 PySpark 資料集，請參閱 [Code] 資料夾中的[資料擷取](https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance/blob/master/Code/1_data_ingestion.ipynb) \(英文\) Jupyter 筆記本案例。

## <a name="scenario-structure"></a>案例結構
案例內容可於 [GitHub 存放庫](https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance)取得。 

[讀我檔案](https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance/blob/master/README.md)檔案概述了從準備資料、建置模型，再到部署解決方案來實際執行的工作流程。 此工作流程的每個步驟都封裝在存放庫內 [Code](https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance/tree/master/Code) 資料夾的 Jupyter 筆記本中。   

[Code\1_data_ingestion.ipynb](https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance/blob/master/Code/1_data_ingestion.ipynb)：此筆記本會下載 5 個輸入 .csv 檔案，並執行一些初步的資料清理和視覺化。 此筆記本會將每個資料集轉換成 PySpark 格式，然後將其儲存在 Azure Blob 容器中以在「功能工程筆記本」中使用。

[Code\2_feature_engineering.ipynb](https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance/blob/master/Code/2_feature_engineering.ipynb)：這會從來自 Azure Blob 儲存體的原始資料集，針對遙測、錯誤及維護資料使用標準的時間序列方法，來建構模型功能。 故障相關的元件更換會用來建構描述故障元件的模型標籤。 已加上標籤的功能資料會儲存在 Azure Blob 中供「模型建置筆記本」使用。

[Code\3_model_building.ipynb](https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance/blob/master/Code/3_model_building.ipynb)：「模型建置筆記本」會使用已加上標籤的功能資料集，然後將資料沿著時間戳記分割成定型和開發資料集。 筆記本會設定成 pyspark.ml.classification 模型的實驗。 定型資料會向量化。 使用者可以進行 **DecisionTreeClassifier** 或 **RandomForestClassifier** 的實驗來操作超參數，以找出效能最佳的模型。 效能是藉由評估開發資料集的相關度量統計資料來判斷的。 這些統計資料會往回記錄到 Machine Learning Workbench 執行階段畫面中以供追蹤。 每次執行時，筆記本都會將產生的模型儲存至執行 Jupyter 筆記本核心的本機磁碟。 

[Code\4_operationalization.ipynb](https://github.com/Azure/MachineLearningSamples-PredictiveMaintenance/blob/master/Code/4_operationalization.ipynb)：此筆記本會使用儲存至本機 (Jupyter 筆記本核心) 磁碟的最後一個模型來建置元件，以將模型部署至 Azure Web 服務中。 完整的運作資產會壓縮至儲存在另一個 Azure Blob 容器內的 o16n.zip 檔案中。 此 ZIP 壓縮檔案包含︰

* **service_schema.json**:用於部署的結構描述定義檔。 
* **pdmscore.py**：Azure Web 服務所需的 **init()** 和 **run()** 函式。
* **pdmrfull.model**：模型定義目錄。
    
此筆記本會先使用模型定義來測試函式，然後才封裝操作化資產以供部署。 部署指示包含在此筆記本的結尾。

## <a name="conclusion"></a>結論

此案例概略說明了如何在 Machine Learning Workbench 中，於 Jupyter 筆記本環境內使用 PySpark 來建置端對端的預測性維護解決方案。 此範例案例也詳細描述如何透過「Machine Learning 模型管理」環境來執行模型部署，以進行即時設備故障預測。

## <a name="references"></a>參考

下列參考資料提供其他適用於各種平台的預測性維護使用案例範例：

* [預測性維護解決方案範本](https://docs.microsoft.com/azure/machine-learning/cortana-analytics-playbook-predictive-maintenance)
* [預測性維護建模指南](https://gallery.cortanaintelligence.com/Collection/Predictive-Maintenance-Modelling-Guide-1)
* [使用 SQL R 服務的預測性維護建模指南](https://gallery.cortanaintelligence.com/Tutorial/Predictive-Maintenance-Modeling-Guide-using-SQL-R-Services-1)
* [預測性維護建模指南 Python 筆記本](https://gallery.cortanaintelligence.com/Notebook/Predictive-Maintenance-Modelling-Guide-Python-Notebook-1)
* [使用 PySpark 的預測性維護](https://gallery.cortanaintelligence.com/Tutorial/Predictive-Maintenance-using-PySpark)
* [深入學習預測性維護](https://docs.microsoft.com/azure/machine-learning/desktop-workbench/scenario-deep-learning-for-predictive-maintenance)

## <a name="next-steps"></a>後續步驟

Machine Learning Workbench 中有提供其他示範此產品額外功能的範例案例。 
