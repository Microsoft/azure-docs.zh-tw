---
title: Azure Machine Learning 服務如何運作？
description: 了解組成 Azure Machine Learning 服務的架構、技術與概念。 您也會了解使用服務的一般工作流程，以及 Azure Machine Learning 服務所使用的 Azure 服務。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: haining
author: hning86
ms.reviewer: larryfr
ms.date: 09/24/2018
ms.openlocfilehash: 3011fa85dbac2135f4d9113c6b76a8b667ee4013
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46952118"
---
# <a name="architecture-and-concepts-how-does-azure-machine-learning-service-work"></a>架構與概念：Azure Machine Learning 服務如何運作？ 

此文件說明 Azure Machine Learning 服務的架構與概念。 下圖顯示服務的主要元件，並說明使用服務時的一般工作流程： 

[![Azure Machine Learning 架構與工作流程](./media/concept-azure-machine-learning-architecture/workflow.png)](./media/concept-azure-machine-learning-architecture/workflow.png#lightbox)

工作流程一般遵循下列步驟：

1. 使用 __Python__ 開發機器學習服務定型指令碼。
1. 建立並設定__計算目標__。
1. __提交指令碼__至設定的計算目標以在該環境中執行。 在定型期間，計算目標會將執行記錄儲存至__資料存放區__。 在該處，記錄會儲存至__實驗__。
1. __查詢實驗__取得目前和過去回合的記錄計量。 如果計量未指出所要的結果，請回到步驟 1 並逐一查看您的指令碼。
1. 找到令人滿意的回合時，請在__模型登錄__中註冊保存的模型。
1. 開發評分指令碼。
1. __建立映像__並在__映像登錄__中註冊該映像。 
1. __將映像__ 部署為 Azure 中的 __Web 服務__。


[!INCLUDE [aml-preview-note](../../../includes/aml-preview-note.md)]

> [!NOTE]
> 雖然此文件定義了 Azure Machine Learning 使用的詞彙與概念，但並沒有定義 Azure 平台的詞彙與概念。 如需 Azure 平台技術的詳細資訊，請參閱 [Microsoft Azure 詞彙](https://docs.microsoft.com/azure/azure-glossary-cloud-terminology)。

## <a name="workspace"></a>工作區

工作區是 Azure Machine Learning 服務的最上層資源。 它可在您使用 Azure Machine Learning 時，提供集中式位置以處理您建立的所有成品。

工作區會保留一份可用於定型模型的計算目標清單。 它也會保留定型執行記錄，包括記錄、計量、輸出，以及指令碼快照集。 此資訊可用來判斷哪一個定型回合會產生最佳模型。

模型會在工作區註冊。 註冊的模型與評分指令碼會用來建立映像。 映像可以接著部署到 Azure 容器執行個體、Azure Kubernetes Service 或線現場可程式化閘陣列 (FPGA) 中，作為 REST 型 HTTP 輸出。 它也可以部署到 Azure IoT Edge 裝置作為模組。

您可以建立多個工作區，而且每個工作區都可由多人共用。 共用工作區時，請透過指派下列角色給使用者以控制對工作區的存取：

* 擁有者
* 參與者
* 讀取者

建立新的工作區時，會自動建立工作區使用的幾個 Azure 資源：

* [Azure Container Registry](https://azure.microsoft.com/services/container-registry/) - 註冊在定型期間與部署模型時使用的 Docker 容器。
* [Azure 儲存體](https://azure.microsoft.com/services/storage/) - 用來作為工作區的預設資料存放區。
* [Azure Application Insights](https://azure.microsoft.com/services/application-insights/) - 儲存與模型有關的監視資訊。
* [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) - 儲存計算目標所使用的密碼與工作區所需的其他機密資訊。

> [!NOTE]
> 您也可以使用現有的 Azure 服務，不需要建立新的版本。 

下圖為工作區的分類：

[![工作區分類](./media/concept-azure-machine-learning-architecture/taxonomy.png)](./media/concept-azure-machine-learning-architecture/taxonomy.png#lightbox)

## <a name="model"></a>模型

簡單來說，模型是可接受輸入並產生輸出的一段程式碼。 機器學習服務模型的建立作業涵蓋選取演算法、提供資料給演算法，以及調整超參數。 定型是一種反覆進行的程序，可產生定型的模型，其中會封裝模型在定型程序期間學到項目。

模型是由 Azure Machine Learning 中的回合所產生的。 您也可以使用在 Azure Machine Learning 外部定型的模型。 模型可以在 Azure Machine Learning 工作區底下註冊。

Azure Machine Learning 與架構無關。 您可以使用任何受歡迎機器學習服務架構建立模型，例如 scikit-learn、xgboost、PyTorch、TensorFlow、Chainer 與 CNTK。

如需定型模型的範例，請參閱[快速入門：建立機器學習服務工作區](quickstart-get-started.md)文件。

### <a name="model-registry"></a>模型登錄

模型登錄可記錄您 Azure Machine Learning 工作區中的所有模型。 

模型是透過名稱與版本來識別的。 每次註冊與現有模型名稱相同的模型時，登錄都會假設它是一個新版本。 版本會累加，新模型則會以該名稱來註冊。

您可以在註冊模型時提供額外的中繼資料標記，然後在搜尋模型時使用這些標記。

您無法刪除映像正在使用的模型。

如需模型註冊範例，請參閱[使用 Azure Machine Learning 將映像分類模型定型](tutorial-train-models-with-aml.md)文件。

## <a name="image"></a>映像

映像提供一個可靠的方式，以部署模型和使用模型時所需的所有元件。 映像包含下列項目：

* 模型。
* 評分指令碼或應用程式。 此指令碼可用來傳送輸入給模型，並傳回模型的輸出。
* 模型或評分指令碼/應用程式所需的相依性。  例如，您可以會包含列出 Python 套件相依性的 Conda 環境檔案。

Azure Machine Learning 可建立兩種類型的映像：

* FPGA 映像：在部署至 Azure 雲端中的現場可程式化閘陣列時使用。
* Docker 映像：在部署至 FPGA 以外的計算目標時使用。 例如，Azure 容器執行個體與 Azure Kubernetes Service。

如需映像建立範例，請參閱[在 Azure 容器執行個體中部署映像分類模型](tutorial-deploy-models-with-aml.md)文件。

### <a name="image-registry"></a>映像登錄

映像登錄可記錄從您的模型建立的映像。 您可以在建立映像時提供額外的中繼資料標記。 映像登錄會儲存中繼資料標記，並可透過查詢標記尋找您的映像。

## <a name="deployment"></a>部署

部署是指在可能裝載於雲端的 Web 服務，或在用於整合式裝置部署的 IoT 模組中將影像具現化。 

### <a name="web-service"></a>Web 服務

已部署的 Web 服務可以使用 Azure 容器執行個體、Azure Kubernetes Service 或現場可程式化閘陣列 (FPGA)。
服務是從封裝您的模型、指令碼與相關聯檔案的映像建立的。 映像包含經過負載平衡的 HTTP 端點，可接收傳送至 Web 服務的評分要求。

如果已選擇啟用 Application Insight 遙測和/或模型遙測收集功能，Azure 將可以透過此功能協助您監視 Web 服務。 遙測資料只有您才能存取，而且會儲存在您的 Application Insights 與儲存體帳戶執行個體中。

如果已經啟用自動調整功能，Azure 將會自動調整您的部署。

如需將模型部署為 Web 服務的範例，請參閱[在 Azure 容器執行個體中部署映像分類模型](tutorial-deploy-models-with-aml.md)文件。

### <a name="iot-module"></a>IoT 模組

已部署的 IoT 模組是一個 Docker 容器，其中包含您的模型與相關聯的指令碼或應用程式，以及任何額外的相依性。 這些模型是在邊緣裝置上使用 Azure IoT Edge 部署的。 

如果您已經啟用監視功能，Azure 就會從 Azure IoT Edge 模組內部的模型收集遙測資料。 遙測資料只有您才能存取，而且會儲存在您的儲存體帳戶執行個體中。

Azure IoT Edge 會確保模組正在執行，並監視裝載模組的裝置。

## <a name="datastore"></a>資料存放區

資料存放區是 Azure 儲存體帳戶的儲存體抽象概念。 資料存放區可以使用 Azure Blob 容器或 Azure 檔案共用作為後端儲存體。 每個工作區都有預設資料存放區，而且您也可以註冊額外的資料存放區。 

使用 Python SDK API 或 Azure Machine Learning CLI 在資料存放區中儲存及擷取檔案。 

## <a name="run"></a>回合

回合是包含下列資訊的記錄：

* 與回合有關的中繼資料 (時間戳記、持續時間等等)
* 由您的指令碼記錄的計量
* 由實驗自動收集，或由您明確上傳的輸出檔案。
* 包含您指令碼之目錄的快照 (執行之前)

當您提交指令碼以將模型定型時，就會產生回合。 回合可以有 0 或多個子回合。 因此最上層回合可能有 2 個子回合，其中每個可能都有自己的子回合。

如需檢視透過將模型定型所產生之回合的範例，請參閱[快速入門：開始使用 Azure Machine Learning 服務](quickstart-get-started.md)文件。

## <a name="experiment"></a>實驗

實驗是給定指令碼之許多回合組成的群組。 它一律屬於某個工作區。 當您提交回合時，必須提供一個實驗名稱。 回合的資訊會儲存在該實驗底下。 如果提交回合並指定不存在的實驗名稱，就會自動以該名稱建立一個新的實驗。

如需實驗使用範例，請參閱[快速入門：開始使用 Azure Machine Learning 服務](quickstart-get-started.md)文件。

## <a name="compute-target"></a>計算目標

計算目標是用來執行定型指令碼，或裝載 Web 服務部署的計算資源。 支援的計算目標包括： 

* 您的本機電腦
* Azure 中的 Linux VM (例如資料科學虛擬機器)
* Azure Batch AI 叢集
* Apache Spark for HDInsight
* Azure 容器執行個體
* Azure Kubernetes Service

計算目標會附加至工作區。 本機電腦以外的計算目標會由工作區的使用者共用。

大部分計算目標都可使用 Azure 入口網站、Azure Machine Learning SDK 或 Azure CLI，透過工作區直接建立。 如果您有透過其他程序 (例如 Azure 入口網站或 Azure CLI) 建立的計算目標，可以將它們新增 (附加) 至工作區。 某些計算目標必須在工作區外部建立，然後再附加。

如需選取計算目標以進行定型的詳細資訊，請參閱[選取並使用計算目標以將模型定型](how-to-set-up-training-targets.md)文件。

如需選取計算目標以進行部署的詳細資訊，請參閱[使用 Azure Machine Learning 服務部署模型](how-to-deploy-and-where.md)文件。

## <a name="run-configuration"></a>回合組態

回合組態是定義指令碼在指定計算目標中應如何執行的一組指示。 它包含一組豐富的行為定義，例如是否使用現有的 Python 環境，或使用依據規格建立的 Conda 環境。

回合組態可保存在檔案 (位於包含您定型指令碼的目錄內) 中，或建構為記憶體內部物件並用來提交回合。

如需回合組態範例，請參閱[選取及使用計算目標將模型定型](how-to-set-up-training-targets.md)文件。

## <a name="training-script"></a>定型指令碼

您可以指定包含定型指令碼與相關聯檔案的目錄，以將模型定型。 您也可以指定用來儲存定型期間所收集資訊的實驗名稱。 在定型期間，整個目錄會複製到定型環境 (計算目標)，然後啟動回合組態指定的指令碼。 目錄的快照集也會儲存在工作區中的實驗底下。

如需使用指令碼將模型定型的範例，請參閱[使用 Python 建立工作區](quickstart-get-started.md)

## <a name="logging"></a>記錄

開發解決方案時，請在 Python 指令碼中使用 Azure Machine Learning Python SDK 以記錄任意計量。 在該回合之後，請查詢計量以判斷該回合是否產生您要部署的模型。 

## <a name="snapshot"></a>快照

提交回合時，Azure Machine Learning 會將包含指令碼的目錄壓縮成 zip 檔案，然後傳送至計算目標。 接著會在計算目標中將 zip 檔案解壓縮並執行指令碼。 Azure Machine Learning 也會將 zip 檔案以快照方式儲存在回合記錄中。 任何擁有工作區存取權的人都能瀏覽回合記錄並下載快照集。

## <a name="activity"></a>活動

活動代表長時間執行的作業。 下列作業為活動範例：

* 建立或刪除計算目標
* 在計算目標上執行指令碼

活動可透過 SDK 或 Web UI 提供通知，方便您輕鬆監視作業進度。

## <a name="next-steps"></a>後續步驟

使用下列連結開始使用 Azure Machine Learning：

* [什麼是 Azure Machine Learning 服務](overview-what-is-azure-ml.md)
* [快速入門：使用 Python 建立工作區](quickstart-get-started.md)
* [教學課程：將模型定型](tutorial-train-models-with-aml.md)
