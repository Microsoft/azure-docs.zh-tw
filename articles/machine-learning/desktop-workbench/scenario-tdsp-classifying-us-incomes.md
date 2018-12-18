---
title: 所得分類 - Team Data Science Process - Azure Machine Learning | Microsoft Docs
description: 如何使用 Team Data Science Process 範本，在 Azure Machine Learning 中建立可將美國所得分類的專案。
services: machine-learning
documentationcenter: ''
author: deguhath
ms.author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: ''
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/17/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 8533023a16cb350828127d0603e6add0876b37a8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948844"
---
# <a name="income-classification-with-team-data-science-process-tdsp-project"></a>使用 Team Data Science Process (TDSP) 專案進行所得分類

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



## <a name="introduction"></a>簡介

結構的標準化和資料科學專案的記載 (錨定至已建立的[資料科學生命週期](https://github.com/Azure/Microsoft-TDSP/blob/master/Docs/lifecycle-detail.md)) 是促進資料科學小組有效共同作業的關鍵。 使用 [Team Data Science Process (TDSP)](https://github.com/Azure/Microsoft-TDSP) 範本建立 Azure Machine Learning 專案，可提供這類標準化的架構。

我們先前發行了[適用於 TDSP 專案結構和範本的 GitHub 存放庫](https://github.com/Azure/Azure-TDSP-ProjectTemplate)。 但到目前為止，仍無法在資料科學工具中將 TDSP 結構和範本具現化。 我們現在已能夠建立使用 [Azure Machine Learning 的 TDSP 結構和文件範本](https://github.com/amlsamples/tdsp)具現化的 Azure Machine Learning 專案。 [這裡](https://aka.ms/how-to-use-tdsp-in-aml)提供如何在 Azure Machine Learning 中使用 TDSP 結構和範本的指示。 我們在此提供的範例是關於如何使用 TDSP 結構建立實際的機器學習專案、填入專案專屬程式碼、構件和文件，以及在 Azure Machine Learning 中執行該專案。

## <a name="link-to-github-repository"></a>連結至 GitHub 存放庫
我們在[這裡](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome)提供範例的相關摘要文件。 在 GitHub 網站上可以找到更廣泛的文件。

### <a name="purpose"></a>目的
此範例的主要目的在於示範如何在 Azure Machine Learning 中使用 [Team Data Science Process (TDSP)](https://github.com/Azure/Microsoft-TDSP) 結構和範本，具現化和執行機器學習專案。 基於此目的，我們使用[來自 UCI 機器學習存放庫的 1994 年美國人口普查資料](https://archive.ics.uci.edu/ml/datasets/adult)。 模型化工作可從美國人口普查資訊 (例如年齡、種族、教育程度、原屬國家等)，預測美國年度所得類別。

### <a name="scope"></a>影響範圍
 * 機器學習模型的資料探勘、訓練和部署，可處理「使用案例概觀」中所述的預測問題。 
 * 使用 Azure Machine Learning 中此專案的 Team Data Science Process (TDSP) 範本，在 Azure Machine Learning 中執行專案。 對於專案執行和報告，我們將會使用 TDSP 生命週期。
 * 直接從 Azure Container Services 中的 Azure Machine Learning 將解決方案運算化。

 此專案強調 Azure Machine Learning 的好幾項功能，例如 TDSP 結構具現化和使用、執行 Jupyter Notebook 和 Python 檔案中的程式碼，以及使用 Docker 和 Kubernetes 在 Azure Container Services 中輕易進行運算化。


## <a name="team-data-science-process-tdsp-lifecycle"></a>Team Data Science Process (TDSP) 生命週期
請參閱 [Team Data Science Process (TDSP) 生命週期](https://github.com/Azure/Microsoft-TDSP/blob/master/Docs/lifecycle-detail.md)

![](./media/scenario-tdsp-classifying-us-incomes/tdsp-lifecycle.jpg)

## <a name="prerequisites"></a>必要條件
### <a name="required-subscription-hardware-software"></a>必要：訂用帳戶、硬體、軟體
1. Azure [訂用帳戶](https://azure.microsoft.com)。 您也可取得[免費訂用帳戶](https://azure.microsoft.com/free/?v=17.16&WT.srch=1&WT.mc_id=AID559320_SEM_cZGgGOIg)來執行這個範例。
2. [Azure 資料科學虛擬機器 (DSVM) Windows Server 2016](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.windows-data-science-vm)、(VM 大小：[DS3_V2](https://docs.microsoft.com/azure/virtual-machines/windows/sizes)具有 4 個虛擬 CPU 和 14 Gb 的 RAM)。 雖然是在 Azure DSVM 上測試，但有可能在任何 Windows 10 電腦上運作。
3. 檢閱 Azure Machine Learning 及其相關服務的說明文件 (請參閱下面的連結)。
4. 確定您已正確安裝由[快速入門安裝指南](quickstart-installation.md)所提供的 Azure Machine Learning。

此範例的資料集來自 UCI ML 存放庫 [[連結]](https://archive.ics.uci.edu/ml/datasets/adult)。 它取自 1994 年美國人口普查資料庫，包含大約 50,000 個人的人口普查和所得資訊。 這是具有數值和分類特徵的結構化資料集，以及包含兩個所得類別的分類目標 ('>50 K' 或 '<=50 K')。 

### <a name="optional-version-control-repository"></a>選擇性：版本控制存放庫
如果您想要儲存專案及其內容並控制其版本，則必須具有版本控制儲存庫才能進行。 在 Azure Machine Learning 中使用 TDSP 範本建立新專案時，您可以輸入 Git 儲存庫位置。 請參閱[如何在 Azure Machine Learning 中使用 Git](using-git-ml-project.md)，以取得進一步的詳細資訊。

### <a name="informational-about-azure-machine-learning"></a>告知性：關於 Azure Machine Learning
* [常見問題集 - 如何開始使用](frequently-asked-questions.md)
* [概觀](../service/overview-what-is-azure-ml.md)
* [安裝](quickstart-installation.md)
* [執行](experimentation-service-configuration.md)
* [使用 TDSP](https://aka.ms/how-to-use-tdsp-in-aml)
* [讀取和寫入檔案](how-to-read-write-files.md)
* [使用 Git 搭配 Azure 機器學習](using-git-ml-project.md)
* [將 ML 模型部署為 Web 服務](model-management-service-deploy.md)

### <a name="create-a-new-workbench-project"></a>建立新的 Workbench 專案

使用此範例作為範本來建立新專案：
1.  開啟 Azure Machine Learning Workbench
2.  在 [專案] 頁面上，按一下 **+** 符號，然後選取 [新增專案]
3.  在 [建立新專案] 窗格中，填入新專案的資訊
4.  在 [搜尋專案範本] 搜尋方塊中，輸入「將美國所得分類 - TDSP 專案」並選取範本
5.  按一下 [建立] 

如果您在建立專案期間 (在適當的方塊中) 提供空的 Git 存放庫位置，則會在建立專案之後，使用專案結構和內容填入該存放庫。

## <a name="use-case-overview"></a>使用案例概觀
問題在於了解在美國人口普查中擷取的社經資料如何協助預測美國的年度個人所得。 根據這類人口普查特徵，機器學習工作可預測個人所得是否高於 $50,000 (二元分類工作)。

## <a name="data-description"></a>資料說明
如需資料的詳細資訊，請參閱 UCI 存放庫中的[說明](https://archive.ics.uci.edu/ml/machine-learning-databases/adult/adult.names)。 

此資料擷取自美國人口普查局資料庫： https://www.census.gov/en.html。 


* 共有 48,842 個執行個體 (篩選之前)、混合連續和離散 (訓練=32,561，測試=16,281)
* 標籤 '>50 K' 的機率: 23.93% / 24.78% (不含未知數)
* 標籤 '<=50 K' 的機率: 76.07% / 75.22% (不含未知數)  

* **目標**：所得類別 '>50 K'、'<=50 K'。 在資料準備階段，這些分別由 1 和 0 取代。
* **特徵**：年齡、工作類別、教育程度、種族、性別、每週工作時數等。


## <a name="project-structure-execution-and-reporting"></a>專案結構、執行和報告

### <a name="structure"></a>Structure
我們在此專案中使用 TDSP 資料夾結構和文件範本 (下方)，其遵循 [TDSP 生命週期](https://github.com/Azure/Microsoft-TDSP/blob/master/Docs/lifecycle-detail.md)。 

我們會根據[這裡](https://aka.ms/how-to-use-tdsp-in-aml)提供的指示建立專案。 在填入專案的程式碼和構件之後，此結構如下所示 (請參閱下圖中以紅色框線框住的專案結構)。


<img src="./media/scenario-tdsp-classifying-us-incomes/instantiation-4.png" width="900" height="700">

### <a name="execution"></a>執行
在此範例中，我們會在**本機計算環境**中執行程式碼。 請參考 Azure Machine Learning 文件，以取得[執行選項](experimentation-service-configuration.md)的詳細資訊。

在本機 Python 執行階段中執行 Python 指令碼很容易：

    az ml experiment submit -c local my_script.py

從 Azure Machine Learning UI 左側的專案結構中可以按兩下 IPython Notebook 檔案並在 Jypyter Notebook Server 中執行。


逐步資料科學工作流程如下所示：

* [**資料取得與認知**](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome/tree/master/code/01_data_acquisition_and_understanding)

已從 UCI ML 存放庫[[連結]](https://archive.ics.uci.edu/ml/datasets/adult)的 URL 下載 .csv 格式的資料。 ProjectReport.md 檔案會詳細描述特徵、目標及其轉換。

資料取得與認知的程式碼位於：/code/01_data_acquisition_and_understanding。

系統會使用發佈為[資料科學工具的 TDSP 套件](https://github.com/Azure/Azure-TDSP-Utilities)一部分的 Python 3 [IDEAR (互動式資料探勘和報告) 公用程式](https://github.com/Azure/Azure-TDSP-Utilities/tree/master/DataScienceUtilities/DataReport-Utils/Python)，執行資料探勘。 此公用程式有助於包含數值和分類特徵及目標的資料產生標準化資料探勘報告。 下面提供 Python 3 IDEAR 公用程式的使用方式詳細資訊。 

最終資料探勘報告的位置為 [IDEAR.html](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome/tree/master/docs/deliverable_docs)。 IDEAR 報告檢視如下所示：

![](./media/scenario-tdsp-classifying-us-incomes/idear.png)

* [**模型化**](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome/tree/master/code/02_modeling)

我們建立了採用 3 折交叉驗證的兩個模型：彈性網路和隨機樹系。 我們使用 [59 點取樣](http://www.jmlr.org/papers/volume13/bergstra12a/bergstra12a.pdf)進行隨機網格搜尋，以作為交叉驗證和模型參數最佳化的策略。 已使用測試資料集的 AUC (曲線下面積) 來測量模型的精確度。 

模型化的程式碼位於：/code/02_modeling。

彈性網路和隨機樹系模型的 AUC 均 > 0.85。 我們將這兩個模型儲存在 pickled.pkl 檔案中，並輸出這兩種模型的 ROC 圖。 隨機樹系模型的 AUC 為 0.92，而彈性網路模型的 AUC 為 0.90。 此外，為了解譯模型，隨機樹系模型的特徵重要性會輸出於 .csv 檔案並繪製成 pdf (僅限前 20 個預測特徵)。

測試資料之**隨機樹系模型**的 ROC 曲線如下所示。 這是所部署的模型：

![](./media/scenario-tdsp-classifying-us-incomes/rf-auc.png)

隨機樹系模型的特徵重要性 (前 20 個) 如下所示。 它會顯示資本利得金額、教育、婚姻狀態 (具有最高的特徵重要性) 等特徵。

![](./media/scenario-tdsp-classifying-us-incomes/featImportance.png)

* [**部署**](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome/tree/master/code/03_deployment)

我們已在 [Azure Container Service (ACS)](https://azure.microsoft.com/services/container-service/) 中的叢集上部署隨機樹系模型作為 Web 服務。 運算化環境會在叢集中佈建 Docker 和 Kubernetes，以管理 Web 服務部署。 您可以在[這裡](model-management-service-deploy.md)找到有關運算化程序的進一步資訊。 

部署的程式碼位於：/code/03_deployment。


### <a name="final-project-reporthttpsgithubcomazuremachinelearningsamples-tdspuciadultincomeblobmasterdocsdeliverabledocsprojectreportmd"></a>[最終專案報告](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome/blob/master/docs/deliverable_docs/ProjectReport.md)
已編譯的最終專案報告 [ProjectReport](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome/blob/master/docs/deliverable_docs/ProjectReport.md) 中會提供以上各節的詳細資訊。 專案報告也包含有關使用案例、模型效能計量、部署，以及專案開發和部署所在基礎結構的進一步詳細資料。

專案報告，連同整個專案資料夾的內容，以及版本控制存放庫可能會傳遞至用戶端。


## <a name="conclusion"></a>結論

在此範例中，我們示範了如何在 Azure Machine Learning 中使用 TDSP 結構和範本。 透過文件和構件範本，您可以：
1. 正確定義專案的目的和範圍
2. 建立具有分散式角色和責任的專案小組
3. 根據 TDSP 生命週期階段來建構並執行專案
4. 使用 TDSP 資料科學公用程式 (如 IDEAR 資料探勘和視覺效果報告) 來開發標準化報告。
5. 準備可傳遞給用戶端的最終資料科學專案報告

我們希望您使用 Azure Machine Learning 的這項功能，促進您的資料科學小組的標準化和共同作業。

## <a name="next-steps"></a>後續步驟

請參閱下面的參考資料以開始進行：

[如何在 Azure Machine Learning 中使用 Team Data Science Process (TDSP)](https://aka.ms/how-to-use-tdsp-in-aml)

[Team Data Science Process (TDSP)](https://github.com/Azure/Microsoft-TDSP)

[Azure Machine Learning 的 TDSP 專案範本](https://aka.ms/tdspamlgithubrepo)

[UCI ML存放庫中的美國所得資料集](https://archive.ics.uci.edu/ml/datasets/adult)