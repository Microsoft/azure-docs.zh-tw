---
title: Azure Machine Learning 中的 ALM | Microsoft Docs
description: 在 Azure Machine Learning Studio 中套用應用程式生命週期管理最佳作法
keywords: ALM、AML、Azure ML、應用程式生命週期管理、版本控制
services: machine-learning
documentationcenter: ''
author: hning86
ms.author: haining
manager: mwinkle
editor: cgronlun
ms.assetid: 1be6577d-f2c7-425b-b6b9-d5038e52b395
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/27/2016
ms.openlocfilehash: 49d1a228132cc220b30091481bb542623b1e222d
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835860"
---
# <a name="application-lifecycle-management-in-azure-machine-learning-studio"></a>Azure Machine Learning Studio 中的應用程式生命週期管理
Azure Machine Learning Studio 是用於開發機器學習實驗的工具，可在 Azure 雲端平台中運作。 它就像是合併到單一平台的 Visual Studio IDE 和可調整的雲端服務。 您可以將標準應用程式生命週期管理 (ALM) 作法 (從控制各種資產的版本，以至自動執行和部署) 合併到 Azure Machine Learning Studio。 本文會討論其中的部分選項和方法。

## <a name="versioning-experiment"></a>版本控制實驗
有兩個建議方法可控制您的實驗版本。 您可以仰賴內建的執行歷程記錄或以 JSON 格式匯出實驗，以便在外部進行管理。 每種方法都有其優缺點。

### <a name="experiment-snapshots-using-run-history"></a>使用執行歷程記錄的實驗快照
在 Azure Machine Learning Studio 學習實驗的執行模式中，只要在實驗編輯器中按一下 [執行]，不可變的實驗快照就會提交至工作排程器。 若要檢視這份快照清單，請在實驗編輯器的畫面中，按一下命令列上的 [執行歷程記錄] 按鈕。

![執行歷程記錄按鈕](./media/version-control/runhistory.png)

然後，在實驗提交執行和擷取快照時，按一下實驗名稱，即可在鎖定模式中開啟快照。 請注意，只有清單中的第一個項目 (代表目前的實驗) 處於「可編輯」狀態。 也請注意，每個快照也可以有各種狀態，包括「已完成 (部分執行)」、「失敗」、「失敗 (部分執行)」或「草稿」。

![執行歷程記錄清單](./media/version-control/runhistorylist.png)

開啟後，您就可以將快照實驗儲存為新的實驗，然後加以修改。 如果實驗快照包含具有已更新版本的資產 (如定型模型、轉換或資料集)，此快照會在擷取快照時保留原始版本的參考。 如果您將鎖定的快照儲存為新的實驗，Azure Machine Learning Studio 會偵測到這些資產有更新的版本存在，而會在新實驗中自動加以更新。

如果您刪除此實驗，該實驗的所有快照會遭到刪除。

### <a name="exportimport-experiment-in-json-format"></a>以 JSON 格式匯出/匯入實驗
每次將實驗提交執行時，執行歷程記錄快照會在 Azure Machine Learning Studio 中保留實驗的不可變版本。 您也可以在本機儲存實驗的複本，並將其簽入到您最喜愛的原始檔控制系統 (例如 Team Foundation Server)，稍後再從該本機檔案重新建立實驗。 您可以使用 [Azure Machine Learning PowerShell](http://aka.ms/amlps) Cmdlet [*Export-AmlExperimentGraph*](https://github.com/hning86/azuremlps#export-amlexperimentgraph) 和 [*Import-AmlExperimentGraph*](https://github.com/hning86/azuremlps#import-amlexperimentgraph) 來完成此作業。

JSON 檔案是實驗圖形的文字表示法，但可能在工作區中包含資產的參考，例如資料集或定型模型。 它不會包含資產的序列化版本。 如果您嘗試將 JSON 文件匯回工作區中，所參考的資產必須已經存在且具有實驗中所參考的相同資產識別碼。 否則您無法存取匯入的實驗。

## <a name="versioning-trained-model"></a>定型模型版本控制
Azure Machine Learning 中的定型模型會序列化為一種稱為 .iLearner 檔案的格式 (`.iLearner`)，並儲存在與工作區相關聯的 Azure Blob 儲存體帳戶中。 取得 .iLearner 檔案複本的方法之一是透過重新訓練 API。 [本文](retrain-models-programmatically.md)說明重新訓練 API 的運作方式。 高階步驟：

1. 設定您的訓練實驗。
2. 將 Web 服務輸出連接埠新增至「訓練模型」模組，或是可產生定型模型的模組，例如「微調模型參數」或「建立 R 模型」。
3. 執行訓練實驗，然後將它部署為模型訓練 Web 服務。
4. 呼叫訓練 Web 服務的 BES 端點，並指定所需的 .iLearner 檔案名稱及其儲存所在的 Blob 儲存體帳戶位置。
5. 在 BES 呼叫完成後，取得所產生的 .iLearner 檔案。

擷取 .iLearner 檔案的另一種方式是透過 PowerShell Cmdlet [*Download-AmlExperimentNodeOutput*](https://github.com/hning86/azuremlps#download-amlexperimentnodeoutput)。 如果您只想取得 iLearner 檔案的複本，而不需要以程式設計方式重新訓練模型，這可能比較容易。

擁有內含定型模型的 .iLearner 檔案之後，您便可以採用自己的版本控制策略。 策略可以非常簡單，像是將前置/後置套用為命名慣例，而只在 Blob 儲存體中保留 .iLearner 檔案，或將其複製/匯入到版本控制系統。

儲存的 .iLearner 檔案可透過已部署的 Web 服務用於評分。

## <a name="versioning-web-service"></a>控制 Web 服務版本
您可以從 Azure Machine Learning 實驗部署兩種類型的 Web 服務。 傳統 Web 服務會與實驗及工作區緊密結合。 新的 Web 服務會使用 Azure Resource Manager 架構，而且不再與原始實驗或工作區結合。

### <a name="classic-web-service"></a>傳統 Web 服務
若要控制傳統 Web 服務的版本，您可以利用 Web 服務端點建構。 以下是典型的流程︰

1. 在預測性實驗中部署新的傳統 Web 服務，其中包含預設端點。
2. 建立名為 ep2 的新端點，該端點會公開目前版本的實驗/定型模型。
3. 返回並更新預測性實驗和定型模型。
4. 重新部署預測性實驗，該實驗會接著更新預設端點。 但這並不會更改 ep2。
5. 您可以建立名為 ep3 的額外端點，該端點會公開新版的實驗和定型模型。
6. 視需要回到步驟 3。

經過一段時間，您可能會在相同的 Web 服務中建立許多端點。 每一個端點都代表實驗的時間點複本，其中包含定型模型的時間點版本。 您可以接著使用外部邏輯來判斷要呼叫哪一個端點，實際上是表示選取定型模型的版本以便進行評分。

您也可以建立許多相同的 Web 服務端點，然後將不同版本的 .iLearner 檔案修補至端點，以達到類似的效果。 [本文](create-models-and-endpoints-with-powershell.md)更詳細地說明如何完成此項作業。

### <a name="new-web-service"></a>新的 Web 服務
如果您建立以 Azure Resource Manager 為基礎的新 Web 服務，就無法再使用端點建構。 然而，您可以從預測性實驗使用 [Export-AmlWebServiceDefinitionFromExperiment](https://github.com/hning86/azuremlps#export-amlwebservicedefinitionfromexperiment) PowerShell Cmdlet，或從已經部署的 Resource Manager 型 Web 服務使用 [*Export-AzureRmMlWebservice*](https://msdn.microsoft.com/library/azure/mt767935.aspx) PowerShell Cmdlet 來產生 Web 服務定義 (WSD) 檔案 (採用 JSON 格式)。

在擁有匯出的 WSD 檔案並對其進行版本控制後，您也可以將 WSD 部署為不同 Azure 區域中不同 Web 服務方案的新 Web 服務。 只要確定您提供適當的儲存體帳戶組態，以及新的 Web 服務方案識別碼。 若要在不同的 .iLearner 檔案中修補，您可以修改 WSD 檔案和更新定型模型的位置參考，並且部署為新的 Web 服務。

## <a name="automate-experiment-execution-and-deployment"></a>自動進行實驗執行和部署
ALM 的重要層面就是能夠自動進行應用程式的執行和部署程序。 在 Azure Machine Learning 中，您可以使用 [PowerShell 模組](http://aka.ms/amlps)來完成此作業。 以下是使用 [Azure Machine Learning Studio PowerShell 模組](http://aka.ms/amlps)之標準 ALM 自動化執行/部署程序相關的端對端步驟範例。 每個步驟都會連結至一或多個 PowerShell Cmdlet，您可以使用它來完成該步驟。

1. [上傳資料集](https://github.com/hning86/azuremlps#upload-amldataset)。
2. 將訓練實驗從[工作區](https://github.com/hning86/azuremlps#copy-amlexperiment)或[資源庫](https://github.com/hning86/azuremlps#copy-amlexperimentfromgallery)複製到工作區，或從本機磁碟[匯入](https://github.com/hning86/azuremlps#import-amlexperimentgraph)一個[已匯出的](https://github.com/hning86/azuremlps#export-amlexperimentgraph)實驗。
3. 在訓練實驗中[更新資料集](https://github.com/hning86/azuremlps#update-amlexperimentuserasset)。
4. [執行訓練實驗](https://github.com/hning86/azuremlps#start-amlexperiment)。
5. [提升定型模型](https://github.com/hning86/azuremlps#promote-amltrainedmodel)。
6. [複製預測性實驗](https://github.com/hning86/azuremlps#copy-amlexperiment)到工作區中。
7. 在預測性實驗中[更新定型模型](https://github.com/hning86/azuremlps#update-amlexperimentuserasset)。
8. [執行預測性實驗](https://github.com/hning86/azuremlps#start-amlexperiment)。
9. 從預測性實驗[部署 Web 服務](https://github.com/hning86/azuremlps#new-amlwebservice)。
10. 測試 Web 服務 [RRS](https://github.com/hning86/azuremlps#invoke-amlwebservicerrsendpoint) 或 [BES](https://github.com/hning86/azuremlps#invoke-amlwebservicebesendpoint) 端點。

## <a name="next-steps"></a>後續步驟
* 下載 [Azure Machine Learning Studio PowerShell](http://aka.ms/amlps) 模組並開始自動執行 ALM 工作。
* 了解如何透過 PowerShell 和重新訓練 API [只使用單一實驗建立和管理大量 ML 模型](create-models-and-endpoints-with-powershell.md)。
* 深入了解如何[部署 Azure Machine Learning Web 服務](publish-a-machine-learning-web-service.md)。
