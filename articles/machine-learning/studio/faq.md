---
title: Azure Machine Learning 常見問題集 (FAQ) | Microsoft Docs
description: Azure Machine Learning 簡介：常見問題集，涵蓋計費、功能，以及適用於簡化預測性模型化之雲端服務的限制。
keywords: 機器學習服務簡介,建立預測模型,什麼是機器學習服務
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: a4a32a06-dbed-4727-a857-c10da774ce66
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/02/2017
ms.openlocfilehash: 22d060ca38663f21bef977f5d506f215fd947a50
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998316"
---
# <a name="azure-machine-learning-frequently-asked-questions-billing-capabilities-limitations-and-support"></a>Azure Machine Learning 常見問題集：計費、功能、限制及支援
以下是有關 Azure Machine Learning 的一些常見問題和對應解答，而 Azure Machine Learning 是適合透過 Web 服務開發預測性模型和運作方案的雲端服務。 這個常見問題涵蓋如何使用服務的相關問題，包括計費模型、功能、限制及支援。

**是否有您無法在這裡找到的問題？**

Azure Machine Learning 在 MSDN 上有一個論壇，資料科學社群的成員可在其中詢問有關 Azure Machine Learning 的問題。 此論壇由 Azure Machine Learning 團隊控管。 移至 [Azure Machine Learning 論壇](http://social.msdn.microsoft.com/Forums/windowsazure/home?forum=MachineLearning)，搜尋答案或張貼您自己的新問題。

## <a name="general-questions"></a>一般問題
**什麼是 Azure Machine Learning 服務？**

Azure Machine Learning 是完全受控的服務，可用來建立、測試、操作及管理雲端中的預測分析方案。 僅使用瀏覽器，您即可以登入、上傳資料，以及立即開始機器學習實驗。 拖放式預測性模型化、大型模組和用以啟動範本的程式庫，讓您得以簡便而快速地執行一般機器學習工作。 如需詳細資訊，請參閱 [Azure Machine Learning 服務概觀](https://azure.microsoft.com/services/machine-learning/)。 如需說明重要術語和概念的機器學習服務簡介，請參閱 [Azure Machine Learning 簡介](what-is-machine-learning.md)。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

**什麼是 Machine Learning Studio？**

Machine Learning Studio 是您利用網頁瀏覽器存取的工作區環境。 Machine Learning Studio 在視覺化結構介面中裝載一組模組，協助您以實驗的形式建置端對端資料科學工作流程。

如需 Machine Learning Studio 的詳細資訊，請參閱 [什麼是 Machine Learning Studio？](what-is-ml-studio.md)

**什麼是 Machine Learning API 服務？**

Machine Learning API 服務可讓您將預測模型 (例如 Machine Learning Studio 內建的預測模型) 部署為可調整、容錯的 Web 服務。 Machine Learning API 服務所建立的 Web 服務是 REST API，此類 API 提供的介面可用於外部應用程式與您的預測性分析模型之間的通訊。

如需詳細資訊，請參閱 [如何取用 Azure Machine Learning Web 服務](consume-web-services.md)。

**傳統 Web 服務會在哪裡列出？新型 (Azure Resource Manager 架構) Web 服務會在哪裡列出？**

使用傳統部署模型建立的 Web 服務和使用新型 Azure Resource Manager 部署模型建立的 Web 服務會在 [Microsoft Azure Machine Learning Web 服務](https://services.azureml.net/)入口網站中列出。

傳統 Web 服務也會在 [Web 服務] 索引標籤的 [Machine Learning Studio](http://studio.azureml.net) 中列出。

## <a name="azure-machine-learning-questions"></a>Azure Machine Learning 問題
**Azure Machine Learning Web 服務是什麼？**

Machine Learning Web 服務可在應用程式與機器學習服務工作流程計分模型之間提供介面。 外部應用程式可以使用 Azure Machine Learning 即時與 Machine Learning 工作流程計分模型通訊。 Machine Learning Web 服務呼叫會將預測結果傳回外部應用程式。 若要進行 Web 服務呼叫，您可以傳遞在部署 Web 服務時所建立的 API 金鑰。 Machine Learning Web 服務以 REST 為基礎，這是一種常見的 Web 程式設計專案架構。

Azure Machine Learning 有兩種 Web 服務類型：

* 要求-回應服務 (RRS)：這是一種低延遲、調整性高的服務，針對使用 Machine Learning Studio 建立和部署的無狀態模型提供介面。
* 批次執行服務 (BES)：這是一種非同步的服務，為一批資料記錄進行計分。

有數種方法可以使用 REST API 和存取 Web 服務。 例如，您可以使用當您部署 Web 服務時為您產生的範例程式碼，在 C#、R 或 Python 中寫入應用程式。

可用的範例程式碼位於：
- Azure Machine Learning Web 服務入口網站中 Web 服務的 [取用] 頁面
- Machine Learning Studio 中 Web 服務儀表板中的 [API 說明] 頁面

您也可以使用為您建立並可在 Machine Learning Studio 中的 Web 服務儀表板取得的範例 Microsoft Excel 活頁簿。

**Azure Machine Learning 的主要更新為何？**

如需最新更新，請參閱 [Azure Machine Learning 新增功能](../../active-directory/fundamentals/whats-new.md)。

## <a name="machine-learning-studio-questions"></a>Machine Learning Studio 問題
### <a name="import-and-export-data-for-machine-learning"></a>匯入和匯出 Machine Learning 的資料
**機器學習服務支援何種資料來源？**

將資料下載至 Machine Learning Studio 實驗的方法有三種︰

- 以資料集的形式上傳本機檔案
- 使用模組從雲端資料服務匯入資料
- 匯入從另一個實驗儲存的資料集

若要深入了解支援的檔案格式，請參閱 [將訓練資料匯入 Azure Machine Learning Studio](import-data.md)。

#### <a id="ModuleLimit"></a>我的模組適用多大的資料集？
Machine Learning Studio 中的模組對常見使用案例支援最多 10 GB 的密集數值資料的資料集。 如果模組接受一個以上的輸入，10 GB 值是所有輸入的大小總計。 您也可以使用 Hive 或 Azure SQL Database 查詢來取樣較大型資料集，或者在擷取前使用「以計數學習」前置處理。  

下列資料類型可以在功能正規化期間擴充為較大型資料集，並限制在小於 10 GB：

* 疏鬆
* 類別
* 字串
* 二進位資料

下列模組限制在小於 10 GB 的資料集：

* Recommender 模組
* 合成少數類過採樣技術 (SMOTE) 模組
* 指令碼模組：R、Python SQL
* 輸出資料大小可以大於輸入資料大小的模組，例如聯結或功能雜湊
* 當反覆運算數目非常大時的交叉驗證、微調模型超參數、序數迴歸和一對多的多類別

#### <a id="UploadLimit"></a>資料上傳的限制為何？
針對大於 2 GB 的資料集，請將資料上傳至 Azure 儲存體或 Azure SQL Database，或是使用 Azure HDInsight，而不要直接從本機檔案上傳。

**可以從 Amazon S3 讀取資料嗎？**

如果您有少量的資料，而且想要透過 HTTP URL 公開，則您可以使用[匯入資料][import-data]模組。 若為較大量的資料，請先傳送至 Azure 儲存體，然後使用[匯入資料][import-data]模組將它帶入實驗中。
<!--

<SEE CLOUD DS PROCESS>
-->

**有內建的影像輸入功能嗎？**

您可以在[匯入影像][image-reader]參考中了解影像輸入功能。

### <a name="modules"></a>模組
**我在尋找的演算法、資料來源、資料格式或資料轉換作業不在 Azure Machine Learning Studio 中。我有哪些選擇？**

您可以前往[使用者意見反應論壇](http://go.microsoft.com/fwlink/?LinkId=404231)，檢視我們所追蹤的功能要求。 如果您要找的功能已有人要求，請投票給該要求。 如果您要找的功能不存在，請建立新要求。 您也可以在此論壇中檢視您的要求狀態。 我們會密切追蹤此清單，並經常更新功能可用性的狀態。 此外，您可以利用對 R 和 Python 的內建支援，視需要建立自訂轉換。

**是否可將我現有的程式碼放入 Machine Learning Studio 中？**

是的，您可以在 Machine Learning Studio 中放入現有的 R 或 Python 程式碼，使用 Azure Machine Learning 學習模組在相同的實驗中加以執行，然後透過 Azure Machine Learning 將解決方案部署為 Web 服務。 如需詳細資訊，請參閱[透過 R 擴展您的實驗](extend-your-experiment-with-r.md)和[在 Azure Machine Learning Studio 中執行 Python 機器學習服務指令碼](execute-python-scripts.md)。

**可使用 [PMML](http://en.wikipedia.org/wiki/Predictive_Model_Markup_Language) 之類來定義模型嗎？**

否，不支援預測模型標記語言 (PMML)。 您可以使用自訂 R 和 Python 程式碼來定義模組。

**在我的實驗中可以平行執行多少個模組？**  

您可以在實驗中最多平行執行 4 個模組。

### <a name="data-processing"></a>資料處理
**有能力在實驗內將資料以互動方式視覺化嗎 (R 視覺效果外)？**

按一下模組的輸出，即可將資料視覺化並取得統計資料。

**在瀏覽器中預覽結果或資料時，資料列和資料行的數目很有限。原因為何？**

因為大量資料可能會傳送至瀏覽器，限制資料大小是為了防止 Machine Learning Studio 變慢。 若要將所有資料/結果視覺化，最好是下載資料並使用 Excel 或其他工具。

### <a name="algorithms"></a>演算法
**Machine Learning Studio 中支援哪些現有的演算法？**

Machine Learning Studio 提供頂級演算法，例如 Scalable Boosted Decision 樹、Bayesian Recommendation 系統、Deep Neural Networks 和 Decision Jungles (由 Microsoft Research 開發)。 此外也包含可調整的開放原始碼機器學習套件，例如 Vowpal Wabbit。 Machine Learning Studio 支援多類別與二進位分類、迴歸和叢集的機器學習演算法。 請參閱[機器學習服務單元][machine-learning-modules]的完整清單。

**您會自動建議我的資料適用的機器學習演算法嗎？**

不會，但 Machine Learning Studio 有各種方法可比較每個演算法的結果，以決定適合您的問題的演算法。

**針對從提供的演算法中挑選，是否有任何準則？**

請參閱 [如何選擇演算法](algorithm-choice.md)。

**提供的演算法是以 R 或 Python 撰寫？**

否，這些演算法大部分是以編譯的語言的撰寫，以提供更好的效能。

**有提供演算法之任何詳細資料嗎？**

文件提供一些演算法的相關資訊，並說明可供微調的參數，以針對您的使用最佳化演算法。  

**線上學習是否有任何支援？**

否，目前只支援以程式設計方式進行重新訓練。

**可以使用內建模組來視覺化類神經網路模型的層級嗎？**

否。

**可以以 C# 或其他語言建立自己的模組嗎？**

目前，您只能使用 R 來建立新的自訂模組。

### <a name="r-module"></a>R 模組
**Machine Learning Studio 中可使用什麼 R 套件？**

Machine Learning Studio 目前支援 400 個以上的 CRAN 套件，以下是所有內含套件的[目前清單](http://az754797.vo.msecnd.net/docs/RPackages.xlsx)。 此外，若要了解如何自行擷取此清單，請參閱 [透過 R 擴展您的實驗](extend-your-experiment-with-r.md) 。 如果您要的套件不在此清單中，請在 [使用者意見反映論壇](http://go.microsoft.com/fwlink/?LinkId=404231)提供套件名稱。

**是否可以建置自訂的 R 模組？**

是，如需詳細資訊，請參閱 [在 Azure Machine Learning 中撰寫自訂 R 模組](custom-r-modules.md) 。

**是否有 R 適用的 REPL 環境？**

否，Studio 中沒有 R 適用的 Read-Eval-Print-Loop (REPL) 環境。

### <a name="python-module"></a>Python 模組
**是否可以建置自訂的 Python 模組？**

目前不可以，但是您可以使用一或多個[執行 Python 指令碼][python]模組來取得相同的結果。

**是否有 Python 適用的 REPL 環境？**

您可以在 Machine Learning Studio 中使用 Jupyter Notebook。 如需詳細資訊，請參閱 [介紹 Azure Machine Learning Studio 中的 Jupyter Notebook](http://blogs.technet.com/b/machinelearning/archive/2015/07/24/introducing-jupyter-notebooks-in-azure-ml-studio.aspx)。

## <a name="web-service"></a>Web 服務
### <a name="retrain"></a>重新訓練
**如何以程式設計方式重新訓練 Azure Machine Learning 模型？**

使用重新訓練 API。 如需詳細資訊，請參閱 [以程式設計方式重塑機器學習模型](retrain-models-programmatically.md)。 [Microsoft Azure Machine Learning 重新訓練示範](https://azuremlretrain.codeplex.com/)也會提供範例程式碼。

### <a name="create"></a>建立
**可以在本機或在沒有網際網路連線的應用程式中部署模型嗎？**

否。

**所有 Web 服務是否有預期的基準延遲？**

請參閱 [Azure 訂用帳戶限制](../../azure-subscription-service-limits.md)。

### <a name="use"></a>使用
**何時該以「批次執行服務」的形式執行預測模型？何時該以要求回應服務的形式執行？**

「要求回應服務 (RRS)」是低延遲性的高階 Web 服務，可用來為從實驗環境建立並部署的無狀態模型提供介面。 批次執行服務 (BES) 是可為一批資料記錄進行非同步評分的服務。 BES 的輸入就像 RRS 使用的資料輸入。 主要差異在於，BES 會從各種來源讀取記錄區塊，例如 Azure Blob 儲存體、Azure 表格儲存體、Azure SQL Database、HDInsight (Hive 查詢) 和 HTTP 來源。 如需詳細資訊，請參閱 [如何取用 Azure Machine Learning Web 服務](consume-web-services.md)。

**如何為已部署的 Web 服務更新模型？**

若要為已部署的服務更新預測模型，請修改並重新執行用來撰寫和儲存已訓練模型的實驗。 在您有新版的已訓練模型後，Machine Learning Studio 會詢問您是否要更新 Web 服務。 如需有關如何更新已部署 Web 服務的詳細資訊，請參閱[部署 Machine Learning Web 服務](publish-a-machine-learning-web-service.md)。

您也可以使用重新訓練 API。
如需詳細資訊，請參閱 [以程式設計方式重塑機器學習模型](retrain-models-programmatically.md)。 [Microsoft Azure Machine Learning 重新訓練示範](https://azuremlretrain.codeplex.com/)也會提供範例程式碼。

**如何監控部署在實際執行環境中的 Web 服務？**

部署預測模型之後，您可以從 Azure Machine Learning Web 服務入口網站加以監視。 每個已部署的服務都有其本身的儀表板，您可以在此處檢視該服務的監控資訊。 如需如何管理已部署 Web 服務的詳細資訊，請參閱[使用 Azure Machine Learning Web 服務入口網站管理 Web 服務](manage-new-webservice.md)和[管理 Azure Machine Learning 工作區](manage-workspace.md)。

**是否有哪個位置，可讓我查看我的 RRS/BES 輸出？**

針對 RRS，通常您可在 Web 服務回應查看此結果。 您可以也將結果寫入至 Azure Blob 儲存體。 BES 的輸出預設會寫入至 Blob。 您也可以使用[匯出資料][export-data]模組，將輸出寫入資料庫或資料表。

**只能從 Machine Learning Studio 中建立的模型來建立 Web 服務嗎？**

不，您也可以直接使用 Jupyter Notebook 和 RStudio 來建立 Web 服務。

**哪裡可以找到有關錯誤碼的詳細資訊？**

如需錯誤碼與說明的清單，請參閱 [Machine Learning 模組錯誤碼](https://msdn.microsoft.com/library/azure/dn905910.aspx) 。

## <a name="scalability"></a>延展性
**什麼是 Web 服務的延展性？**

目前的預設端點是每個端點上佈建 20 個並行 RRS 要求。 您可以將每個端點的並行要求調整為 200 個，也可以將每個 Web 服務的端點調整為 10,000 個，如[調整 Web 服務](scaling-webservice.md)所述。 對於 BES，每個端點一次可以處理 40 個要求，超過 40 個的其他要求則會排入佇列。 這些排入佇列的要求會在佇列清空後自動執行。

**R 作業會分散於節點嗎？**

否。  

**我可以將多少資料用於訓練？**

Machine Learning Studio 中的模組對常見使用案例支援最多 10 GB 的密集數值資料的資料集。 如果模組接受一個以上的輸入，則所有輸入的大小總計為 10 GB。 您也可以取樣較大型資料集，方法是透過 Hive 查詢或 Azure SQL Database 查詢，或在擷取前利用[以計數學習][counts]模組進行前置處理。  

下列資料類型可以在功能正規化期間擴充為較大型資料集，並限制在小於 10 GB：

* 疏鬆
* 類別
* 字串
* 二進位資料

下列模組限制在小於 10 GB 的資料集：

* Recommender 模組
* 合成少數類過採樣技術 (SMOTE) 模組
* 指令碼模組：R、Python SQL
* 輸出資料大小可以大於輸入資料大小的模組，例如聯結或功能雜湊
* 當反覆運算數目非常大時的交叉驗證、微調模型超參數、序數迴歸和一對多的多類別

針對大於幾 GB 的資料集，將資料上傳至 Azure 儲存體或 Azure SQL Database，或是使用 HDInsight，而不要直接從本機檔案上傳。

**是否有任何向量大小限制嗎？**

每個資料列和資料行的限制在 .NET 的最大 Int 限制：2,147,483,647。

**可以調整用來執行 Web 服務的虛擬機器大小嗎？**

否。  

## <a name="security-and-availability"></a>安全性和可用性
**根據預設，哪些人可以存取 Web 服務的 HTTP 端點？如何限制對此端點的存取？**

部署 Web 服務之後，我們會建立該服務的預設端點。 該預設端點可使用其 API 金鑰進行呼叫。 您可以從 Web 服務入口網站或使用 Web 服務管理 API 以程式設計方式新增更多具有專屬金鑰的端點。 必須要有存取金鑰，才能呼叫 Web 服務。 如需詳細資訊，請參閱 [如何取用 Azure Machine Learning Web 服務](consume-web-services.md)。

**如果找不到我的 Azure 儲存體帳戶，會發生什麼情況？**

Machine Learning Studio 依賴使用者提供的 Azure 儲存體帳戶，才能在執行工作流程時儲存中繼資料。 此儲存體帳戶會在建立工作區時提供給 Machine Learning Studio。 建立工作區之後，如果刪除儲存體帳戶，而且不再能找到，工作區會停止運作，並且該工作區中的所有實驗將會失敗。

如果您不小心刪除儲存體帳戶，請使用與所刪除儲存體帳戶相同的區域中的相同名稱來重新建立儲存體帳戶。 在那之後，重新同步處理存取金鑰。

**如果我的儲存體帳戶存取金鑰未同步，會發生什麼情況？**

Machine Learning Studio 依賴使用者提供的 Azure 儲存體帳戶，才能在執行工作流程時儲存中繼資料。 此儲存體帳戶是在建立工作區時提供給 Machine Learning Studio，並且存取金鑰會與該工作區相關聯。 建立工作區之後，如果變更存取金鑰，該工作區便無法再存取儲存體帳戶。 它會停止運作，而該工作區中的所有實驗將會失敗。

如果您變更了儲存體帳戶存取金鑰，請使用 Azure 入口網站在工作區中重新同步處理存取金鑰。  

## <a name="support-and-training"></a>支援和訓練
**哪裡可以取得 Azure Machine Learning 的訓練？**

[Azure Machine Learning 文件中心](https://azure.microsoft.com/services/machine-learning/)包含視訊教學課程和操作方式指南。 這些逐步操作指南提供了服務簡介，並說明匯入資料、清除資料、建置預測模型以及使用 Azure Machine Learning 在生產環境中部署這些模型的資料科學生命週期。

我們會持續更新 Machine Learning Center 的內容。 您可以在 [使用者意見反應論壇](https://windowsazure.uservoice.com/forums/257792-machine-learning)中，提交關於 Machine Learning Center 的其他學習內容要求。

您也可以在 [Microsoft Virtual Academy](http://www.microsoftvirtualacademy.com/training-courses/getting-started-with-microsoft-azure-machine-learning)尋找訓練。

**如何取得 Azure Machine Learning 的支援？**

若要取得 Azure Machine Learning 的技術支援，請移至 [Azure 支援](https://azure.microsoft.com/support/options/)，並選取 [Machine Learning]。

Azure Machine Learning 在 MSDN 上也設有社群論壇，可供您詢問 Azure Machine Learning 的相關問題。 此論壇由 Azure Machine Learning 團隊控管。 請移至 [Azure 論壇](http://social.msdn.microsoft.com/Forums/windowsazure/home?forum=MachineLearning)。

## <a name="billing-questions"></a>計費問題
**機器學習服務如何計費？**

Azure Machine Learning 有兩個元件：Machine Learning Studio 與 Machine Learning Web 服務。

當您在評估 Machine Learning Studio 時，您可以使用免費計費層。 免費層也可讓您部署容量有限的傳統 Web 服務。

如果您斷定 Azure Machine Learning 符合需求，即可註冊標準層。 若要註冊，您必須擁有 Microsoft Azure 訂用帳戶。

標準層會針對您在 Machine Learning Studio 中定義的每個工作區，每個月向您收費。 當您在 Studio 中執行實驗時，您必須針對執行實驗時的計算資源付費。 當您部署傳統 Web 服務時，交易和計算時數則是以隨用隨付方式來收費。

新型 (Resource Manager 架構) Web 服務引進了可提高成本可預測性的計費方案。 分層式價格適用於需要許多容量的客戶，可提供折扣費率。

當您建立方案時，即表示您承諾支付固定費用，以換取方案所包含的 API 計算時數和 API 交易的數量。 如果您需要更多的包含數量，您可以在方案中新增執行個體。 如果您需要非常多的包含數量，您可以選擇較高層級的方案，其可提供非常多的包含數量和更好的折扣費率。

在現有執行個體的包含數量用完後，額外的使用量將需要費用，其收費依據是計費方案層所關聯的超額費率。

> [!NOTE]
每 30 天會重新配置包含的數量，未使用的包含數量不會展期至下一期。

如需其他計費和價格資訊，請參閱 [機器學習服務價格](https://azure.microsoft.com/pricing/details/machine-learning/)。

**機器學習服務是否有免費試用版？**

 Azure Machine Learning 有 [Machine Learning 價格](https://azure.microsoft.com/pricing/details/machine-learning/)中說明的免費訂用帳戶選項。 當您登入 [Machine Learning Studio](https://studio.azureml.net/?selectAccess=true&o=2) 時，Machine Learning Studio 提供八小時的快速評估試用版。

 此外，註冊 Azure 免費試用版後，您可以試用任何 Azure 服務一個月。 若要深入了解 Azure 免費試用版，請造訪 [Azure 免費試用常見問題集](https://azure.microsoft.com/pricing/free-trial-faq/)。

**什麼是交易？**

交易代表 Azure Machine Learning 所回應的 API 呼叫。 來自要求回應服務 (RRS) 和批次執行服務 (BES) 呼叫的交易會彙總起來，並根據計費方案來收費。

**方案中包含的交易數量是否可用於 RRS 和 BES 交易？**

是，來自 RRS 和 BES 的交易會彙總起來，並根據計費方案來收費。

**什麼是 API 計算時數？**

API 計算時數是 API 呼叫使用 Machine Learning 計算資源來執行所需時間的計費單位。 系統會彙總所有呼叫以便計費。

**典型的生產 API 呼叫需要花費多久時間？**

生產 API 呼叫時間的差別可能很大，通常從數百毫秒到幾秒鐘。 視資料處理和機器學習模型的複雜度而定，有些 API 呼叫可能需要數分鐘。 估計生產 API 呼叫時間最好的方法是在機器學習服務上建立基準模型。

**什麼是 Studio 計算時數？**

Studio 計算時數是實驗在 Studio 中使用計算資源的彙總時間計費單位。

**在新型 (Azure Resource Manager 架構) Web 服務中，開發/測試層的用途為何？**

Resource Manager 架構 Web 服務提供多個層級供您佈建計費方案。 開發/測試定價層提供有限的包含數量，可讓您以 Web 服務的形式測試實驗，而不會產生費用。 您將有機會查看其運作情形。

**儲存體需要另外付費嗎？**

機器學習服務免費層不需要或不允許個別儲存體。 機器學習服務標準層要求使用者必須有 Azure 儲存體帳戶。 Azure 儲存體是[另外收費](https://azure.microsoft.com/pricing/details/storage/)。

**Machine Learning 是否支援高可用性？**

是。 如需詳細資訊，請參閱[機器學習服務價格](https://azure.microsoft.com/pricing/details/machine-learning/)，以取得服務等級協定 (SLA) 的說明。

**我的生產 API 呼叫將使用哪些特定種類的計算資源來執行？**

Machine Learning 服務是多租用戶服務。 後端實際使用的計算資源有所不同，並依效能和可預測性而最佳化。

### <a name="management-of-new-resource-manager-based-web-services"></a>新型 (Resource Manager 架構) Web 服務的管理
**如果我刪除方案會發生什麼事？**

方案將會從訂用帳戶中移除，並以按比例計算的使用量向您收費。

> [!NOTE]
您無法刪除 Web 服務正在使用的方案。 若要刪除方案，您必須指派新方案給 Web 服務或刪除 Web 服務。

**什麼是方案執行個體？**

方案執行個體是可在計費方案中新增的包含數量單位。 為計費方案選取計費層時，方案便會隨附一個執行個體。 如果您需要更多的包含數量，您可以在方案中新增所選計費層的執行個體。

**可以新增多少個方案執行個體？**

在訂用帳戶的開發/測試定價層可以有一個執行個體。

在標準 S1、標準 S2 和標準 S3 層中，您可以視需要新增多個執行個體。

> [!NOTE]
根據預期的使用量而定，更符合成本效益的方式可能是升級為有更多包含數量的層級，而非在目前的層級中新增執行個體。

**變更方案層級 (升級/降級) 時會發生什麼事？**

舊方案會遭到刪除，目前的使用量將按比例計費。 系統會針對剩餘期間建立新的方案，其中將會有升級/降級層的完整包含數量。

> [!NOTE]
包含的數量是針對每個期間來配置，未使用的數量不會展期。

**在方案中新增執行個體時會發生什麼事？**

包含的數量會按比例包含，而且可能需要 24 小時才會生效。

**刪除方案執行個體時會發生什麼事？**

執行個體將會從訂用帳戶中移除，並以按比例計算的使用量向您收費。

### <a name="sign-up-for-new-resource-manager-based-web-services-plans"></a>註冊新型 (Resource Manager 架構) Web 服務方案
**如何註冊方案？**

有兩種方式可供建立計費方案。

當您第一次部署 Resource Manager 架構 Web 服務時，您可以選擇現有方案，或建立新方案。

以這種方式建立的方案會位於預設區域中，而且 Web 服務將會部署到該區域。

如果您想要將服務部署到預設區域以外的區域，您可以在部署服務前定義計費方案。

在此情況下，您可以登入 Azure Machine Learning Web 服務入口網站，然後移至 [方案] 頁面。 您可以在該處新增方案、刪除方案，以及修改現有方案。

**我應該選擇從哪一個方案來開始？**

建議您從標準 S1 層來開始，並監視服務的使用量。 如果您發現包含的數量使用快速，則可以新增執行個體，或改用較高的方案層並獲得更好的折扣費率。 在整個計費週期內，您都可以視需要調整計費方案。

**哪些區域有提供新的方案？**

支援新的 Web 服務的三個生產區域中有提供新的計費方案︰

* 美國中南部
* 西歐
* 東南亞

**我在多個區域擁有 Web 服務。是否每個區域都需要一個方案？**

是。 不同區域有不同的方案價格。 當您將 Web 服務部署到其他區域時，您必須對服務指派該區域特定的方案。 如需詳細資訊，請參閱[不同區域提供的產品]( https://azure.microsoft.com/regions/services/)。

### <a name="new-web-services-overages"></a>新型 Web 服務：超額
**如何檢查 Web 服務使用量是否超額？**

您可以在 Azure Machine Learning Web 服務入口網站的 [方案] 頁面，檢視所有方案的使用量。 請登入該入口網站，然後按一下 [方案] 功能表選項。

在資料表的 [交易] 和 [計算] 資料行，您可以看到方案的包含數量和已使用的百分比。

**開發/測試定價層的包含數量用完時會發生什麼事？**

獲指派開發/測試定價層的服務會停止，直到下一個週期來臨，或直到您將它們移至付費層為止。

**針對傳統 Web 服務和超額的新 (Resource Manager 架構) Web 服務，如何計算要求回應 (RRS) 和批次 (BES) 工作負載的價格？**

對於 RRS 工作負載，我們會向您收取每個 API 交易呼叫的費用，以及與這些要求相關之計算時間的費用。 RRS 生產 API 交易費用的計算方式為：API 呼叫總數乘以每 1,000 筆交易的定價 (依個別交易的比例計算)。 RRS API 生產 API 計算時數費用的計算方式為：每個 API 呼叫所需的執行時間總數乘以 API 交易總數，再乘以每個生產 API 計算時數的定價。

例如，對於標準 S1 超額來說，執行時間各為 0.72 秒的 1,000,000 個 API 交易，將會產生 (1,000,000 個 * 0.50 美元/1,000 個 API 交易) 500 美元的生產 API 交易費用和 (1,000,000 個 * 0.72 秒 * 2 美元/小時) 400 美元的生產 API 計算時數，總計 900 美元。

BES 工作負載採用相同的計費方式。 不過，API 交易費用代表您提交的批次工作數目，而計算費用則代表與這些批次工作相關的計算時間。 BES 生產 API 交易費用的計算方式為：提交的工作總數乘以每 1,000 筆交易的定價 (依個別交易的比例計算)。 BES API 生產 API 計算時數費用的計算方式為：作業中每個資料列所需的執行時間乘以作業中的資料列總數、乘以工作總數，再乘以每個生產 API 計算時數的價格。 當您使用 Machine Learning 計算機時，交易計量代表您計劃提交的作業數量，而每筆交易時間欄位則代表每個作業中所有資料列執行所需時間的總數。

例如，假設採用標準 S1 超額，而且您每天提交 100 個作業，每個作業包含 500 個資料列，且執行時間各為 0.72 秒。 您的每月超額費用會是 (每天 100 個作業 = 每月 3,100 個作業 * 0.50 美元/1,000 個 API 交易) 1.55 美元的生產 API 交易費用和 (500 個資料列 * 0.72 秒 * 3,100 個作業 * 2 美元/小時) 620 美元的生產 API 計算時數，總計 621.55 美元。

### <a name="azure-machine-learning-classic-web-services"></a>Azure Machine Learning 傳統 Web 服務
**是否還有提供隨用隨付？**

是，Azure Machine Learning 中仍然提供傳統的 Web 服務。  

### <a name="azure-machine-learning-free-and-standard-tier"></a>Azure Machine Learning 的免費層和標準層
**Azure Machine Learning 免費層包含什麼？**

Azure Machine Learning 免費層主要是用來提供 Azure Machine Learning Studio 的深入介紹。 您只需要 Microsoft 帳戶即可註冊。 免費層可讓每個 [Microsoft 帳戶](https://account.microsoft.com/account)免費存取一個 Azure Machine Learning Studio 工作區。 在此層中，您可以使用最多 10 GB 的儲存體，以及讓模型能以預備 API 運作。 免費層工作負載不包含在 SLA 內，且僅供開發與個人使用。 

免費層工作區有下列限制：

* 工作負載無法藉由連線至執行 SQL Server 之內部部署伺服器來存取資料。
* 您無法部署新的 Resource Manager 基底 Web 服務。


**Azure Machine Learning 的標準層和方案包含什麼？**

Azure Machine Learning 標準層是 Azure Machine Learning Studio 的付費正式版本。 Azure Machine Learning Studio 月費會每月依據個別工作區收費，不足的月份則按比例收費。 Azure Machine Learning Studio 實驗時數會依據進行中實驗的每個計算時數收費。 不足的時數會按比例收費。  

Azure Machine Learning API 服務會根據它是傳統 Web 服務還是新型 (Resource Manager 架構) Web 服務來收費。

下列費用依訂用帳戶的每個工作區合計。

* Machine Learning 工作區訂用帳戶：Machine Learning 工作區訂用帳戶是用於存取 Machine Learning Studio 工作區的月費。 要在 Studio 中執行實驗並使用生產 API 的話，就需要此訂用帳戶。
* Studio 實驗時數：此計量彙總在 Machine Learning Studio 中執行實驗和在預備環境中執行生產 API 呼叫而產生的所有計算費用。
* 連接到在您的模型中執行 SQL Server 的內部部署伺服器以存取資料，供訓練和評分使用。
* 傳統 Web 服務：
  * 生產 API 計算時數：此計量包含在生產環境中執行的 Web 服務所產生的計算費用。
  * 生產 API 交易 (以 1000 個為單位)：此計量包含每次呼叫生產 Web 服務所產生的費用。

除了上述費用，如果是 Resource Manager 架構 Web 服務，費用會彙總至選取的方案︰

* 標準 S1/S2/S3 API 方案 (單位)：此計量表示針對 Resource Manager 架構 Web 服務選取的執行個體類型。
* 標準 S1/S2/S3 超額 API 計算時數：此計量包含現有執行個體中包含的數量用完後，在生產環境中執行的 Resource Manager 架構 Web 服務所產生的計算費用。 額外的使用量會以與 S1/S2/S3 方案層相關聯的超額費率收費。
* 標準 S1/S2/S3 超額 API 交易 (在 1,000 秒內)：此計量包含現有執行個體中包含的數量用完後，對生產環境的 Resource Manager 架構 Web 服務所進行的每個呼叫所產生的費用。 額外的使用量會以與 S1/S2/S3 方案層相關聯的超額費率收費。
* 包含的數量 API 計算時數：在 Resource Manager 架構 Web 服務中，此計量表示 API 計算時數的包含數量。
* 包含的數量 API 交易 (在 1,000 秒內)：在 Resource Manager 架構 Web 服務中，此計量表示 API 交易的包含數量。

**如何註冊 Azure Machine Learning 免費層？**

您只需要 Microsoft 帳戶。 前往 [Azure Machine Learning 首頁](https://azure.microsoft.com/services/machine-learning/)，然後按一下 [立即開始]。 使用 Microsoft 帳戶登入，系統便會為您建立免費層工作區。 您可以立即開始探索，並建立機器學習服務實驗。

**如何註冊 Azure Machine Learning 標準層？**

您必須先取得 Azure 訂用帳戶的存取權，才能建立標準 Machine Learning 工作區。 您可以註冊 30 天的免費試用 Azure 訂用帳戶，之後再升級為付費 Azure 訂用帳戶，也可以直接購買付費的 Azure 訂用帳戶。 您可以在取得訂用帳戶的存取權後，從 Microsoft Azure 入口網站建立 Machine Learning 工作區。 請檢視 [逐步指示](https://azure.microsoft.com/trial/get-started-machine-learning-b/)。

或者，您也可以受到標準 Machine Learning 工作區擁有者的邀請，存取擁有者的工作區。

**我可以在免費層中指定使用我自己的 Azure Blob 儲存體帳戶嗎？**

否，標準層相當於推出層次之前可用的機器學習服務版本。

**我可以 API 形式將機器學習服務模型部署在免費層嗎？**

是，在免費層中，您可以將機器學習模型轉變成預備 API 服務來運作。 若要讓預備 API 服務進入正式運作並取得操作化服務的生產端點，您必須使用標準層。

**Azure 免費試用版和 Azure Machine Learning 免費層有何差異？**

[Microsoft Azure 免費試用版](https://azure.microsoft.com/free/)提供適用於任何 Azure 服務的一個月點數。 Azure Machine Learning 免費層特別為非生產工作負載，提供連續存取 Azure Machine Learning。

**要如何將實驗從免費層移至標準層？**

若要將實驗從免費層複製到標準層：

1. 登入 Azure Machine Learning Studio，確定您可以在上方導覽列的工作區選取器中看到免費工作區和標準工作區。
2. 如果您目前位於標準工作區，請切換至免費工作區。
3. 在實驗清單檢視中，選取想要複製的實驗，然後按一下 [複製] 命令按鈕。
4. 在開啟的對話方塊中選取 [標準] 工作區，然後按一下 [複製] 按鈕。
   所有相關聯的資料集、訓練好的模型等項目，會連同實驗複製到標準工作區中。
5. 您必須在標準工作區中重新執行實驗並重新發行 Web 服務。

### <a name="studio-workspace"></a>Studio 工作區
**不同的工作區會有不同的帳單嗎？**

一份帳單上，工作區費用會依每個適用的度量而個別細分。

**我的實驗作業會在何種特定類型的計算資源上進行？**

Machine Learning 服務是多租用戶服務。 後端實際使用的計算資源有所不同，並依效能和可預測性而最佳化。

### <a name="guest-access"></a>來賓存取
**何謂 Azure 機器學習 Studio 的來賓存取？**

「來賓存取」是有限制的試用經驗。 您可以在不經驗證的情況下，於 Azure Machine Learning Studio 中免費建立及執行實驗。 來賓工作階段為非持續性工作階段 (無法儲存) 且僅限 8 小時。 其他限制包括缺少 R 和 Python 支援、缺少預備 API，以及有限的資料集大小和儲存體容量。 透過比較，選擇以 Microsoft 帳戶登入的使用者享有前述 Machine Learning Studio 免費層的完整存取權，包括持續性工作區和更全面的功能。 若要選擇免費的 Machine Learning 體驗，請在 [https://studio.azureml.net](https://studio.azureml.net) 上按一下 [開始使用]，然後選取 [來賓存取] 或以 Microsoft 帳戶登入。

<!-- Module References -->
[image-reader]: https://msdn.microsoft.com/library/azure/893f8c57-1d36-456d-a47b-d29ae67f5d84/
[join]: https://msdn.microsoft.com/library/azure/124865f7-e901-4656-adac-f4cb08248099/
[machine-learning-modules]: https://msdn.microsoft.com/library/azure/6d9e2516-1343-4859-a3dc-9673ccec9edc/
[partition-and-sample]: https://msdn.microsoft.com/library/azure/a8726e34-1b3e-4515-b59a-3e4a475654b8/
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
[export-data]: https://msdn.microsoft.com/library/azure/7A391181-B6A7-4AD4-B82D-E419C0D6522C
[split]: https://msdn.microsoft.com/library/azure/70530644-c97a-4ab6-85f7-88bf30a8be5f/
[python]: https://msdn.microsoft.com/library/azure/CDB56F95-7F4C-404D-BDE7-5BB972E6F232
[counts]: https://msdn.microsoft.com/library/azure/dn913056.aspx
