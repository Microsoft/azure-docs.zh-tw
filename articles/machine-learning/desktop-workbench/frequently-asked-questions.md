---
title: Azure Machine Learning 2017 預覽版常見問題集 | Microsoft Docs
description: 本文包含 Azure Machine Learning 預覽版功能的常見問題集與解答
services: machine-learning
author: serinakaye
ms.author: serinak
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 08/30/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 4785f87515663831b1177c6c0958d3d965351797
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996565"
---
# <a name="azure-machine-learning-frequently-asked-questions"></a>Azure Machine Learning 常見問題集

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 

Azure Machine Learning 是一個完全受控的 Azure 服務，可讓您建立、測試、管理及部署機器學習服務和 AI 模型。 我們的服務和可下載應用程式提供一個程式碼優先方法，其運用雲端、內部部署環境及邊緣來提供具有強大功能、速度與彈性的訓練、部署、管理和監視模型。 或者，Azure Machine Learning Studio 也提供一個無須編寫任何程式碼的瀏覽器型視覺拖放撰寫環境。 

## <a name="general-product-questions"></a>一般產品問題

**需要哪些其他 Azure 服務？**

Azure Blob 儲存體和 Azure Container Registry 會由 Azure Machine Learning 使用。 此外，您必須佈建運算資源，例如資料科學 VM 或 HDInsight 叢集。 部署您的 Web 服務時也需要運算和裝載，例如 [Azure Container Service](https://docs.microsoft.com/azure/aks)。

**Azure Machine Learning 與 SQL Server 2017 中的 Microsoft Machine Learning 服務如何相關？**   

SQL Server 2017 中的「Machine Learning 服務」是一個可延伸、可調整的平台，用來將機器學習服務工作整合到資料庫工作流程中。 它特別適用於需要內部部署解決方案的情況，例如資料移動成本太高或無法支援。 對比之下，雲端或混合式工作負載則最適用於我們的新 Azuer 服務。 

**Azure Machine Learning 與適用於 Spark 的 Microsoft Machine Learning 如何相關？**

MMLSpark 提供適用於 Apache Spark 的深層學習和資料科學工具，強調生產力、容易測試及最先進的演算法。 MMLSpark 提供「Spark 機器學習服務」管線與 Microsoft Cognitive Toolkit 和 OpenCV 的整合。 您可以為影像和文字資料建立強大、具備高度靈活調整能力的預測和分析模型。 MMLSpark 是以開放原始碼授權的形式提供，並且包含在 AML Workbench 中作為一組可取用的模型和演算法。 如需有關 MMLSpark 的詳細資訊，請瀏覽我們的產品文件。 

**新工具和服務支援哪些 Spark 版本？**

Workbench 目標包含並支援 MMLSpark 0.8 版，此版本與 Apache Spark 2.1 相容。 您也可以選擇在 Linux 虛擬機器上使用支援 GPU 的 MMLSpark 0.8 Docker 映像。

## <a name="experimentation-service"></a>測試服務

**什麼是 Azure Machine Learning 測試服務？**

「測試服務」是一個將機器學習體驗提升到下一層次的受控 Azure 服務。 您可以在本機或雲端建立測試。 在桌上型電腦上快速建立原型，然後擴充至虛擬機器或 Spark 叢集。 具有最新 GPU 技術的 Azure VM 可讓您迅速有效地進行深層學習。 我們也包含了與 Git 的深度整合，因此您可以輕鬆插入到現有的工作流程中來進行程式碼追蹤、設定及共同作業。 

**測試服務的收費方式為何？**

前兩個與您「Azure Machine Learning 測試服務」關聯的使用者是免費的。 針對額外的使用者，將會依「公開預覽版」費率每月收取 50 美元的費用。 如需有關定價和計費的詳細資訊，請瀏覽我們的「定價」頁面。

**是否會依我執行測試的次數向我收費？**

否，「測試服務」可讓您視需要執行許多測試，而且只依使用者數目收費。 測試計算資源會另外收費。 建議您執行多次測試，以便找出最適合您解決方案的模型。   

**我可以使用哪些特定類型的計算和儲存體資源？**

測試服務可以在本機電腦 (直接或 Docker 型)、[Azure 虛擬機器](https://azure.microsoft.com/services/virtual-machines/)及 [HDInsight](https://azure.microsoft.com/services/hdinsight/) 上執行您的測試。 此服務也會存取 [Azure 儲存體](https://azure.microsoft.com/services/storage/)帳戶來儲存執行輸出，並且可利用 [Visual Studio Team Service](https://azure.microsoft.com/services/visual-studio-team-services/) 帳號來進行版本控制和 Git 儲存。 請注意，針對任何取用的計算和儲存體資源，會依其個別定價向您另外收費。  


## <a name="model-management"></a>模型管理

**什麼是 Azure Machine Learning 模型管理？**

「Azure Machine Learning 模型管理」是一個受控 Azure 服務，可讓資料科學人員和 DevOps 小組將預測模型可靠地部署到各種環境。 Git 儲存機制和 Docker 容器可提供可追蹤性和可重複性。 模型可以可靠地部署在雲端、內部部署環境或邊緣。 在生產環境中之後，您將可以管理模型效能，然後在效能降低時，主動重新訓練。 您可以將本機電腦上的模型部署到 [Azure VM](https://azure.microsoft.com/services/virtual-machines/)、Spark on [HDInsight](https://azure.microsoft.com/services/hdinsight/) 或 Kubernetes 協調的 [Azure Container Service](https://azure.microsoft.com/services/container-service/) 叢集。  

**什麼是「模型」？**

模型是已升階來進行模型管理之測試執行回合的輸出。 在裝載帳戶中註冊的模型會計入您的方案中，包括透過重新訓練或版本反覆運算更新的模型。

**什麼是「受控模型」？**

模型是訓練程序的輸出，也是機器學習演算法在訓練資料上的應用。 「模型管理」可讓您將模型部署成 Web 服務、管理各種模型版本，以及監視其效能和計量。 「受控」模型是已向「Azure Machine Learning 模型管理」帳戶註冊的模型。 舉例來說，想像一個您嘗試預測銷售額的情況。 在測試階段，您會使用不同的資料集或演算法來產生許多模型。 您已產生四個具有不同精確度的模型，但選擇只註冊精確度最高的模型。 這個註冊的模型會成為您的第一個受控模型。
 
**什麼是「部署」？**

「模型管理」可讓您將模型部署成 Azure 中已封裝的 Web 服務容器。 叫用這些 Web 服務時，可以使用 REST API 來叫用。 每個 Web 服務都會視為單一部署，而作用中部署的總數會計入您的方案中。 以銷售額預測為例，當您部署效能最佳的模型時，您的方案會增加一個部署。 如果您接著重新訓練並部署另一個版本，您就會有兩個部署。 如果您判斷新的模型較佳，並刪除原始模型，您的部署計數就會減一。  

**有哪些特定的計算資源可供我的部署使用？** 

「模型管理」可以將部署當作已向 [Azure Container Service](https://azure.microsoft.com/services/container-service/) 註冊的 Docker 容器、當作 [Azure 虛擬機器](https://azure.microsoft.com/services/virtual-machines/)來執行，或是在本機電腦上執行。 很快就會再新增其他部署目標。 請注意，針對任何取用的計算資源，會依其個別定價向您另外收費。     

**我是否可以使用 Azure Machine Learning 模型管理，來部署運用測試服務以外工具建置的模型？**

當您部署使用「測試服務」建立的模型時，可以獲得最佳體驗。 不過，您也可以部署運用其他架構和工具建置的模型。 我們支援各種模型，包括 MMLSpark、TensorFlow、Microsoft Cognitive Toolkit、scikit-learn、Keras 等。 

**我是否可以使用自己的 Azure 資源？**

是，「測試服務」和「模型管理」可以與多個 Azure 資料存放區、計算工作負載及其他服務搭配運作。 如需有關所需 Azure 服務的進一步詳細資料，請參考我們的技術文件。

**是否同時支援內部部署和雲端部署案例？**

是。 我們透過 Docker 容器支援內部部署和雲端部署情節。 本機執行目標包括：單一節點 Docker 部署、[搭配 ML 服務的 Microsoft SQL Server](https://docs.microsoft.com/sql/advanced-analytics/r/r-services)、Hadoop 或 Spark。 我們也支援透過 Docker 進行雲端部署，包括：透過 Azure Container Service 和 Kubernetes、HDInsight 或 Spark 叢集進行叢集部署。 針對邊緣案例，則是透過 Docker 容器和 Azure IOT Edge 提供支援。 

**我是否可以在另一部主機上執行使用 Azure Machine Learning CLI 建立的 Docker 映像？**

是。 您可以使用該映像作為任何 Docker 主機上的 Web 服務，只要主機擁有足夠的運算資源來裝載該 Docker 映像即可。

**是否支援重新訓練已部署的模型？**

是，您可以部署同一模型的多個版本。 「模型管理」將可支援所有已更新之模型和映像的服務更新。

## <a name="workbench"></a>Workbench

**什麼是 Azure Machine Learning Workbench？**

Azure Machine Learning Workbench 是為專業資料科學人員建置的隨附應用程式。 Machine Learning Workbench 適用於 Windows 和 Mac，提供機器學習解決方案的概觀、管理及控制。 Machine Learning Workbench 可讓您存取來自 Microsoft 和開放原始碼社群的頂尖 AI 架構。 我們包含了最常用的資料科學工具組，包括 TensorFlow、Microsoft Cognitive Toolkit、Spark ML、scikit-learn 等。 我們也啟用了與常用資料科學 IDE (例如 Jupyter Notebook、PyCharm 及 Visual Studio Code) 的整合。 Machine Learning Workbench 有內建的資料準備功能，可快速取樣、理解及準備資料，不論該資料是結構化還是非結構化資料。 我們的新資料準備工具稱為 [PROSE](https://microsoft.github.io/prose/)，是以來自「Microsoft 研究」的頂尖技術為基礎建置的。  

**Workbench 是否是一種 IDE？**

否。 Machine Learning Workbench 已設計為常用 IDE (例如 Jupyter Notebook、Visual Studio Code 及 PyCharm) 的隨附應用程式，但它不是功能完整的 IDE。 Machine Learning Workbench 提供一些基本的文字編輯功能，但不支援偵錯、IntelliSense 及其他常用的 IDE 功能。 建議您使用慣用的 IDE 來進行程式碼開發、編輯及偵錯。 您也可以嘗試使用 [Visual Studio Code Tools for AI](https://www.visualstudio.com/downloads/ai-tools-vscode)。

**使用 Azure Machine Learning Workbench 是否需要付費？**

否。 Azure Machine Learning Workbench 是免費的應用程式。 您可以視需要將它下載到許多部電腦，並供許多使用者使用。 若要使用 Azure Machine Learning Workbench，您必須擁有一個測試帳戶。 .  

**是否支援命令列功能？**

是，Azure Machine Learning 提供完整的 CLI 介面。 Machine Learning CLI 預設會隨 Azure Machine Learning Workbench 一起安裝。 它也隨附於 Azure 上的「Linux 資料科學」虛擬機器，並且將整合至 [Azure CLI](https://docs.microsoft.com/cli/azure?view=azure-cli-latest)


**我是否可以將 Jupyter Notebook 與 Workbench 搭配使用？**

可以！ 您可以在 Workbench 中執行 Jupyter Notebook，其中以 Workbench 作為用戶端裝載應用程式，就像您會使用瀏覽器作為用戶端一樣。 

**支援哪些 Jupyter Notebook 核心？**

Workbench 隨附的目前 Jupyter 版本會啟動 Python 3 核心，並針對您 aml_config 資料夾中的每個 "runconfig" 檔案啟動一個額外的核心。 支援的組態包括：
- 本機 Python
- 本機或遠端 Docker 中的 Python

## <a name="data-formats-and-capabilities"></a>資料格式和功能

**Workbench 中的資料內嵌目前支援哪些檔案格式？**

Workbench 中的資料準備工具目前支援從下列格式內嵌： 
- 分隔符號檔案 (例如 CSV、TSV 等)  
- 固定寬度檔案
- 純文字檔案
- Excel (.xls/xlsx)
- JSON 檔案
- Parquet 檔案 
- 自訂檔案 (指令碼) - 如果您的解決方案要求從其他來源內嵌資料，Python 程式碼可以用來... 

**目前支援哪些資料儲存位置？**

針對公開預覽版，Workbench 支援從下列位置內嵌資料： 
- 本機硬碟或對應的網路儲存位置
- Azure BLOB 或 Azure 儲存體 (需要 Azure 訂用帳戶)
- Azure SQL Server
- 連接字串


**有哪些資料整理、準備及轉換可供使用？**

針對公開預覽版，Workbench 支援「依範例衍生資料行」、「依範例分割資料行」、「文字叢集」、「處理遺漏值」等。  Workbench 也支援資料類型轉換、資料彙總 (COUNT、MEAN、VARIANCE 等)，以及複雜的資料聯結。 如需完整的支援功能清單，請瀏覽我們的產品文件。 

**Azure Machine Learning Workbench、測試或模型管理是否有強制執行任何資料大小限制？**

否，新服務並未強制執行任何資料限制。 不過，您執行資料準備、模型訓練、測試或部署的環境會導入限制。 例如，如果您以本機環境作為訓練目標，就會受到硬碟中的可用空間限制。 或者，如果您以 HDInsight 作為目標，則會受到任何關聯的大小或計算約束條件限制。 

## <a name="algorithms-and-libraries"></a>演算法和程式庫

**Azure Machine Learning Workbench 中支援哪些演算法？**

我們的預覽版產品和服務包含開放原始碼社群的最佳演算法。 我們支援各種演算法和程式庫，包括 TensorFlow、scikit-learn、Apache Spark 及 Microsoft Cognitive Toolkit。 Azure Machine Learning Workbench 也封裝了 [Microsoft revoscalepy](https://docs.microsoft.com/sql/advanced-analytics/python/what-is-revoscalepy) 套件。

**Azure Machine Learning 與 Microsoft Cognitive Toolkit 如何相關？**

[Microsoft Cognitive Toolkit](https://www.microsoft.com/cognitive-toolkit/) 是我們新工具和服務所支援的眾多架構之一。 Cognitive Toolkit 是一個整合的深層學習工具組，可讓您取用和結合常用的機器學習模型，包括「前饋深層類神經網路」(Feed-Forward Deep Neural Networks)、卷積類神經網路 (Convolutional Nets)、「序列到序列」(Sequence-to-Sequence) 及「循環式網路網路」。 如需有關 Microsoft Cognitive Toolkit 的詳細資訊，請瀏覽我們的[產品文件](https://docs.microsoft.com/cognitive-toolkit/)。 