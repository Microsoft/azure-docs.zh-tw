---
title: Azure Machine Learning 模型管理的概念性概觀 | Microsoft Docs
description: 本文件說明 Azure Machine Learning 的模型管理概念。
services: machine-learning
author: hjerezmsft
ms.author: hjerez
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 09/20/2017
ROBOTS: NOINDEX
ms.openlocfilehash: d3f7e206e7f4aa61a8ec1272ff2670d81bb7a33e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46974675"
---
# <a name="azure-machine-learning-model-management"></a>Azure Machine Learning 模型管理

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]


Azure Machine Learning 模型管理可讓您管理和部署機器學習服務的工作流程和模型。 

模型管理可提供下列功能：
- 模型版本設定
- 在生產環境中追蹤模型
- 使用 [Azure Container Service](https://azure.microsoft.com/services/container-service/) 和 [Kubernetes](https://docs.microsoft.com/azure/container-service/kubernetes/container-service-kubernetes-walkthrough) 透過 AzureML 計算環境將模型部署到生產環境
- 使用模型建立 Docker 容器並在本機測試這些容器
- 自動重新訓練模型
- 擷取模型的遙測以獲得可採取相應措施的深入剖析資訊。 

Azure Machine Learning 模型管理可註冊模型版本。 它也會提供自動化的工作流程來封裝 Machine Learning 容器並將容器部署為 REST API。 系統會使用預測 API 將模型和其執行階段相依性封裝在 Linux 型 Docker 容器中。 

Azure Machine Learning 計算環境有助於設定及管理用來裝載模型的可擴充叢集。 計算環境是以 Azure Container Service 作為基礎。 Azure Container Service 可將 Machine Learning API 自動公開為具有下列功能的 REST API 端點：

- 驗證
- 負載平衡
- 自動相應放大
- 加密

Azure Machine Learning 模型管理會透過 CLI、API 和 Azure 入口網站來提供這些功能。 

Azure Machine Learning 模型管理會使用下列資訊：

 - 模型檔案或具有模型檔案的目錄
 - 使用者所建立、會實作模型計分函式的 Python 檔案
 - 列出執行階段相依性的 Conda 相依性檔案
 - 執行階段環境選擇，以及 
 - API 參數的結構描述檔案 

這項資訊會用來執行下列動作：

- 註冊模型
- 建立會在建置容器時使用的資訊清單
- 建置 Docker 容器映像
- 將容器部署至 Azure Container Service
 
下圖顯示如何在叢集中註冊和部署模型的概觀。 

![](media/model-management-overview/modelmanagement.png)

## <a name="create-and-manage-models"></a>建立和管理模型 
您可以在 Azure Machine Learning 模型管理中註冊模型，以在生產環境中追蹤模型版本。 為了方便重現和管理，這項服務會擷取所有相依性和相關聯的資訊。 若要深入了解效能情況，您可以使用提供的 SDK 擷取模型遙測。 模型遙測會封存在使用者提供的儲存體中。 模型遙測可於稍後用來分析模型效能、重新進行訓練，以及深入了解您的公司。

## <a name="create-and-manage-manifests"></a>建立和管理資訊清單 
模型需要其他的構件才能部署到生產環境。 系統會提供功能來讓您建立資訊清單，以在其中包含模型、相依性、推斷指令碼 (也稱為計分指令碼)、資料範例、結構描述等項目。此資訊清單可作為用來建立 Docker 容器映像的配方。 企業可以自動產生資訊清單、建立不同的版本，以及管理其資訊清單。 

## <a name="create-and-manage-docker-container-images"></a>建立和管理 Docker 容器映像 
您可以使用上一個步驟所建立的資訊清單，以在 Docker 型容器映像各自的環境中建置這些映像。 經過容器化的 Docker 型映像可讓企業彈性地在下列計算環境中執行這些映像：

- [Kubernetes 型 Azure Container Service](https://docs.microsoft.com/azure/container-service/kubernetes/container-service-kubernetes-walkthrough)
- 內部部署容器服務
- 開發環境
- IoT 裝置

這些經過容器化的 Docker 型映像都會自行包含要產生預測所需具備的所有必要相依性。 

## <a name="deploy-docker-container-images"></a>部署 Docker 容器映像 
在 Azure Machine Learning 模型管理中，只要使用單一命令就可以將 Docker 型容器映像部署到由 ML 計算環境所管理的 Azure Container Service 中。 這些部署在建立時所使用的前端伺服器會提供下列功能：

- 大規模的低延遲預測
- 負載平衡
- 自動調整 ML 端點
- API 金鑰授權
- API Swagger 文件

您可以透過下列組態設定來控制部署規模和遙測：

- 適用於每個 Web 服務層級的系統記錄和模型遙測。 如果啟用，所有 stdout 記錄都會串流至 [Azure Application Insights](https://azure.microsoft.com/services/application-insights/)。 模型遙測會封存在您提供的儲存體中。 
- 自動調整和並行存取限制。 這些設定會根據現有叢集內的負載，自動增加所部署的容器數目。 這些設定也會控制預測延遲的輸送量和一致性。

## <a name="consumption"></a>耗用量 
Azure Machine Learning 模型管理會為已部署的模型建立 REST API 以及 Swagger 文件。 在企業營運應用程式的執行過程中，您可以藉由使用 API 金鑰和模型輸入來呼叫 REST API，以取用所部署的模型來獲得預測。 GitHub 中有提供 Java、[Python](https://github.com/CortanaAnalyticsGallery-Int/digit-recognition-cnn-tf/blob/master/client.py) 和 C# 等語言的程式碼範例，以供用來呼叫 REST API。 Azure Machine Learning 模型管理 CLI 可讓您以簡單的方式使用這些 REST API。 您可以使用單一 CLI 命令、已啟用 Swagger 功能的應用程式，或是使用 curl 來取用 API。 

## <a name="retraining"></a>重新訓練 
Azure Machine Learning 模型管理會提供 API 供您重新訓練模型。 您也可以使用 API，以更新後的模型版本來更新現有部署。 在資料科學工作流程的進行過程中，您會在實驗環境中重建模型。 然後，您會在模型管理中註冊模型，並更新現有部署。 使用單一的 UPDATE CLI 命令即可執行更新。 UPDATE 命令不必變更 API URL 或金鑰就能更新現有部署。 取用模型的應用程式能夠在不變更任何程式碼的情況下繼續運作，並使用新模型來開始獲得較佳的預測。

下圖擷取了完成的工作流程來說明這些概念：

![](media/model-management-overview/modelmanagementworkflow.png)

## <a name="frequently-asked-questions-faq"></a>常見問題集 (FAQ) 
- **支援何種資料類型？是否可以直接傳遞 NumPy 陣列來作為 Web 服務的輸入？**

   如果您要提供使用 generate_schema SDK 所建立的結構描述檔案，則可以傳遞 NumPy 和 (或) Pandas DF。 您也可以傳遞任何能夠以 JSON 進行序列化的輸入。 您也可以利用二進位編碼字串的形式來傳遞映像。

- **Web 服務是否支援多個輸入或是否會剖析不同的輸入？**

   是，您可以將封裝在一個 JSON 要求中的多個輸入作為字典。 每個輸入會對應至一個唯一的字典索引鍵。

- **由 Web 服務要求所啟用的呼叫是屬於封鎖呼叫還是非同步呼叫？**

   如果服務是在 CLI 或 API 執行期間使用即時選項所建立的，則屬於封鎖/同步呼叫。 服務應該要快到彷彿是即時的。 不過，您可以在用戶端使用非同步的 HTTP 程式庫來呼叫服務以避免用戶端執行緒遭到封鎖。

- **Web 服務可以同時處理多少要求？**

   視叢集和 Web 服務的規模而定。 您可以將服務相應放大到複本的 100 倍，然後它就可以同時處理許多要求。 您也可以設定每個複本的最大並行要求數量，以增加服務的輸送量。

- **Web 服務可以將多少要求排入佇列？**

   數量是可以設定的。 根據預設，它會設定為每個單一複本最多 10 個，但您可以予以增加/減少以符合應用程式的需求。 一般而言，增加排入佇列之要求的數目會增加服務輸送量，但在百分位數較高時會讓延遲時間變嚴重。 若要讓延遲保持一致，您可以將佇列設定為較低的值 (1-5)，並增加用來處理輸送量的複本數目。 您也可以開啟自動調整功能，讓系統根據負載情形自動調整複本數目。 

- **可以將同一個機器或叢集用於多個 Web 服務端點嗎？**

   當然。 同一個叢集上可以執行 100 倍的服務/端點。 

## <a name="next-steps"></a>後續步驟
若要開始使用模型管理，請參閱[設定模型管理](deployment-setup-configuration.md)。
