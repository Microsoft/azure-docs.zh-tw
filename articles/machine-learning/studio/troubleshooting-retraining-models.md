---
title: 針對重新訓練 Azure Machine Learning Classic Web 服務進行疑難排解 | Microsoft Docs
description: 找出您在為 Azure Machine Learning Web 服務重新訓練模型時所遇到的常見問題，並加以修正。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: 75cac53c-185c-437d-863a-5d66d871921e
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/01/2017
ms.openlocfilehash: 989bf010320501050a37fbf2f0799f50a5a3e2ba
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835768"
---
# <a name="troubleshooting-the-retraining-of-an-azure-machine-learning-classic-web-service"></a>針對 Azure Machine Learning 傳統 Web 服務的重新訓練進行疑難排解
## <a name="retraining-overview"></a>重新訓練概觀
當您將預測性實驗部署為評分 Web 服務時，它會是靜態模型。 當有新資料可用或 API 取用者有自己的資料時，模型就必須重新訓練。 

如需傳統 Web 服務重新訓練程序的完整逐步解說，請參閱[以程式設計方式重新訓練 Machine Learning 模型](retrain-models-programmatically.md)。

## <a name="retraining-process"></a>重新訓練程序
當您需要重新訓練 Web 服務時，您必須另外加入某些部分︰

* 從訓練實驗部署的 Web 服務。 實驗必須將 **Web 服務輸出**模組附加至**訓練模型**模組的輸出。  
  
    ![將 Web 服務的輸出附加至訓練模型。][image1]
* 加入至評分 Web 服務的新端點。  您可以透過程式設計方式，使用《以程式設計方式重新訓練機器學習服務模型》主題中所參照的範例程式碼新增端點，或是透過 Azure Machine Learning Web 服務入口網站來新增。

接著，您可以使用訓練 Web 服務的 API 說明頁面中的範例 C# 程式碼重新訓練模型。 在評估結果後，如果您感到滿意，就可以使用新加入的端點更新已訓練的模型評分 Web 服務。

準備好所有部分之後，為了重新訓練模型所必須採取的主要步驟如下︰

1. 呼叫訓練 Web 服務︰呼叫的對象為批次執行服務 (BES)，而非求回應服務 (RRS)。 您可以使用 API 說明頁面中的範例 C# 程式碼來進行呼叫。 
2. 尋找 *BaseLocation*、*RelativeLocation* 和 *SasBlobToken* 的值︰在您呼叫訓練 Web 服務的輸出中會傳回這些值。 
   ![顯示重新訓練範例的輸出和 BaseLocation、RelativeLocation 及 SasBlobToken 的值。][image6]
3. 從評分 Web 服務使用訓練好的新模型更新加入的端點︰使用《以程式設計方式重新訓練機器學習服務模型》所提供的範例程式碼，從評分 Web 服務使用訓練好的新模型更新加入到評分模型的新端點。

## <a name="common-obstacles"></a>常見障礙
### <a name="check-to-see-if-you-have-the-correct-patch-url"></a>檢查是否有正確的 PATCH URL
所使用的 PATCH URL 必須是與加入到評分 Web 服務的新評分端點相關聯的 URL。 有幾個方法可取得 PATCH URL：

**選項 1︰程式設計方式**

若要取得正確的 PATCH URL：

1. 執行 [AddEndpoint](https://github.com/raymondlaghaeian/AML_EndpointMgmt/blob/master/Program.cs) 範例程式碼。
2. 從 AddEndpoint 的輸出中找出「HelpLocation」  值並複製 URL。
   
   ![addEndpoint 範例之輸出中的 HelpLocation。][image2]
3. 將此 URL 貼到瀏覽器中，瀏覽到提供 Web 服務說明連結的頁面。
4. 按一下 [更新資源] 連結以開啟修補說明頁面。

**選項 2：使用 Azure Machine Learning Web 服務入口網站**

1. 登入 [Azure Machine Learning Web 服務](https://services.azureml.net/)入口網站。
2. 按一下頂端的 [Web 服務] 或 [傳統 Web 服務]。
4. 按一下您正在處理的評分 Web 服務 (如果您並未修改 Web 服務的預設名稱，它的結尾是 "[Scoring Exp.]")。
5. 按一下 [+新增]。
6. 在新增端點之後，按一下其端點名稱。
7. 在 [修補程式] 之下，按一下 [API 說明] 以開啟修補說明頁面。

> [!NOTE]
> 如果您已將端點新增至訓練 Web 服務，而不是預測性 Web 服務，當您按一下 [更新資源] 連結，就會收到下列錯誤︰「很抱歉，但這項功能在此內容中不支援或不可使用。 此 Web 服務有沒有可更新的資源。 造成您的不便我們深感抱歉，並將致力於改善這個工作流程。」
> 
> 

PATCH 說明頁面包含必須使用的 PATCH URL，並提供可用來呼叫它的範例程式碼。

![修補 URL。][image5]

### <a name="check-to-see-that-you-are-updating-the-correct-scoring-endpoint"></a>檢查正在更新的評分端點是否正確
* 請勿修補訓練 Web 服務︰修補作業必須是對評分 Web 服務來執行。
* 請勿修補 Web 服務上的預設端點︰修補作業必須是對您新增的評分 Web 服務端點來執行。

您可以造訪 Web 服務入口網站來確認端點位於哪個 Web 服務上。 

> [!NOTE]
> 請確定您將端點新增至預測性 Web 服務，而不是定型 Web 服務。 如果您正確部署定型和預測性 Web 服務，您應該會看到列出兩個不同的 Web 服務。 預測性 Web 服務應是以 "[predictive exp.]" 結尾。
> 
> 

1. 登入 [Azure Machine Learning Web 服務](https://services.azureml.net/)入口網站。
2. 按一下 [Web 服務] 或 [傳統 Web 服務]。
3. 選取您的預測性 Web 服務。
4. 確認新的端點已加入到 Web 服務。

### <a name="check-that-your-workspace-is-in-the-same-region-as-the-web-service"></a>檢查您的工作區是否位於與 Web 服務相同的區域
1. 登入 [Machine Learning Studio](https://studio.azureml.net/)。
2. 在頂端，按一下您工作區的下拉式清單。

   ![機器學習服務區域 UI。][image4]

3. 確認您的工作區所在的區域。

<!-- Image Links -->

[image1]: ./media/troubleshooting-retraining-a-model/ml-studio-tm-connnected-to-web-service-out.png
[image2]: ./media/troubleshooting-retraining-a-model/addEndpoint-output.png
[image3]: ./media/troubleshooting-retraining-a-model/azure-portal-update-resource.png
[image4]: ./media/troubleshooting-retraining-a-model/check-workspace-region.png
[image5]: ./media/troubleshooting-retraining-a-model/ml-help-page-patch-url.png
[image6]: ./media/troubleshooting-retraining-a-model/retraining-output.png
[image7]: ./media/troubleshooting-retraining-a-model/web-services-tab.png
