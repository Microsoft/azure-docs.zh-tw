---
title: Azure Machine Learning 模型如何變成 Web 服務 | Microsoft Docs
description: 機制的概觀，說明 Azure Machine Learning 模型如何從開發實驗進展為實際運作的 Web 服務。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: 25e0c025-f8b0-44ab-beaf-d0f2d485eb91
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2017
ms.openlocfilehash: 7b9aec2815f836b3b220de37fe6428c54d39c3e5
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835567"
---
# <a name="how-a-machine-learning-model-progresses-from-an-experiment-to-an-operationalized-web-service"></a>Machine Learning 模型如何從實驗進展為實際運作的 Web 服務
Azure Machine Learning Studio 提供互動式畫布，可讓您開發、執行、測試及反覆運算代表預測性分析模型的***實驗***。 有各種可用的模組可以︰

* 將資料輸入到實驗
* 處理資料
* 使用機器學習服務演算法來訓練模型
* 評分模型
* 評估結果
* 輸出最終值

一旦滿意實驗，您就可以將其部署為***傳統 Azure Machine Learning Web 服務***或***新式 Azure Machine Learning Web 服務***，讓使用者可以將新的資料傳送給它並接收結果。

在本文中我們將提供機制的概觀，說明 Machine Learning 模型如何從開發實驗進展為實際運作的 Web 服務。

> [!NOTE]
> 有其他方式可開發和部署機器學習模型，但本文著重在如何使用 Machine Learning Studio。 例如，若要閱讀如何使用 R 來建立傳統預測性 Web 服務的說明，請參閱部落格文章[使用 RStudio 和 Azure ML 建置和部署預測性 Web Apps (英文)](http://blogs.technet.com/b/machinelearning/archive/2015/09/25/build-and-deploy-a-predictive-web-app-using-rstudio-and-azure-ml.aspx)。
> 
> 

雖然 Azure Machine Learning Studio 的設計是為了協助您開發及部署 *預測性分析模型*，但可以使用 Studio 來開發未包含預測性分析模型的實驗。 比方說，實驗可能只是輸入資料、操作資料，然後輸出結果。 如同預測性分析實驗，您可以將這個非預測實驗部署為 Web 服務，但此程序更簡單，因為實驗不會對機器學習模型進行定型或計分。 雖然通常不會這樣使用 Studio，我們還是將它納入討論，以便提供 Studio 運作方式的完整說明。

## <a name="developing-and-deploying-a-predictive-web-service"></a>開發及部署預測性 Web 服務
以下是使用 Machine Learning Studio 進行開發及部署時一般解決方案所遵循的階段：

![部署流程](./media/model-progression-experiment-to-web-service/model-stages-from-experiment-to-web-service.png)

*圖 1 - 一般預測性分析模型的階段*

### <a name="the-training-experiment"></a>訓練實驗
***訓練實驗***Machine Learning Studio 中開發 Web 服務的初始階段。 訓練實驗的目的是要提供您開發、測試、逐一查看和最終定型機器學習模型的位置。 尋求最佳的解決方案時，您甚至可以同時定型多個模型，但是一旦完成實驗，您將選取單一定型的模型，並從實驗消除其餘部分。 如需開發預測性分析實驗的範例，請參閱 [在 Azure Machine Learning 中為信用風險評估開發預測性分析解決方案](walkthrough-develop-predictive-solution.md)。

### <a name="the-predictive-experiment"></a>預測性實驗
當您在訓練實驗中產生定型模型之後，請在 Machine Learning Studio 中按一下 [設定 Web 服務]，並選取 [預測性 Web 服務]將訓練實驗轉換為***預測實驗***。 預測實驗的目的是要使用您的定型模型對新資料進行計分，以最終目標成為實際運作的 Azure Web 服務。

會透過下列步驟為您完成這項轉換：

* 轉換用於定型成單一模組的模組集，並將它儲存為定型的模型
* 排除與計分無關的任何多餘模組
* 新增最終的 Web 服務將使用的輸入和輸出連接埠

可能有您想要進行的變更，以讓預測實驗備妥供部署為 Web 服務。 例如，如果想要 Web 服務輸出結果的子集，您可以在輸出連接埠之前新增篩選模組。

在此轉換程序中不會捨棄訓練實驗。 程序完成時您在 Studio 中將有兩個索引標籤：一個用於訓練實驗，而一個用於預測實驗。 以此方式，在部署您的 Web 服務之前，您可以對訓練實驗進行變更，並重建預測實驗。 或者您可以儲存一份訓練實驗，以開始另一行的實驗。

> [!NOTE]
> 按一下 [預測性 Web 服務] 時，就會開始自動化的程序，將訓練實驗轉換為預測實驗，而且這在大部分的情況下可正常運作。 如果訓練實驗很複雜 (例如，聯結在一起的訓練有多個路徑)，您可能會想要手動進行這項轉換。 如需詳細資訊，請參閱[如何準備您的模型以在 Azure Machine Learning Studio 中部署](convert-training-experiment-to-scoring-experiment.md)。
> 
> 

### <a name="the-web-service"></a>Web 服務
一旦您認為您的預測實驗已經就緒，您就可以根據 Azure Resource Manager，將服務部署為傳統 Web 服務或新式 Web 服務。 若要將您的模型部署為*傳統 Machine Learning Web 服務*來運作，請按一下 [部署 Web 服務]，然後選取 [部署 Web 服務 [傳統]]。 若要部署為*新式 Machine Learning Web 服務*，請按一下 [部署 Web 服務]，然後選取 [部署 Web 服務 [新式]]。 使用者現在可以使用 Web 服務 REST API 將資料傳送至您的模型中並收回結果。 如需詳細資訊，請參閱 [如何取用 Azure Machine Learning Web 服務](consume-web-services.md)。

## <a name="the-non-typical-case-creating-a-non-predictive-web-service"></a>非一般的情況：建立非預測性 Web 服務
如果實驗不會訓練預測性分析模型，則您不需要同時建立訓練實驗和評分實驗 - 只有一個實驗，而您可以將它部署為 Web 服務。 Machine Learning Studio 可分析您所使用的模組，以偵測您的實驗是否包含預測性模型。

逐一查看實驗並滿足於該實驗之後：

1. 按一下 [設定 Web 服務]，並選取 [重新訓練 Web] - 自動會新增輸入和輸出節點
2. 按一下 [執行] 
3. 按一下 [部署 Web 服務]，然後根據您要部署的環境而定，選取 [部署 Web 服務 [傳統]] 或 [部署 Web 服務 [新式]]。

現在已部署您的 Web 服務，而且您可以如同預測性 Web 服務一般存取及管理它。

## <a name="updating-your-web-service"></a>更新您的 Web 服務
現在您已將實驗部署為 Web 服務，如果需要更新它，該怎麼做？

視您需要更新的項目而定：

**您想要變更輸入或輸出，或您想要修改 Web 服務操縱資料的方式**

如果您不變更模型，但只是要變更 Web 服務處理資料的方式，您可以編輯預測實驗，然後按一下 [部署 Web 服務]，再次選取 [部署 Web 服務 [傳統]] 或 [部署 Web 服務 [新式]]。 將會停止 Web 服務、部署已更新的預測實驗，然後重新啟動 Web 服務。

以下是範例：假設預測性實驗會傳回輸入資料的整個資料列與預測結果。 您可能決定您想要 Web 服務只傳回結果。 因此，您可以在預測性實驗中加入 **專案資料行** 模組，緊接在輸出連接埠之前，以便排除資料行而非結果。 當您按一下 [部署 Web 服務]，然後再次選取 [部署 Web 服務 [傳統]] 或 [部署 Web 服務 [新式]] 時，Web 服務就會更新。

**您想要使用新資料重新定型模型**

如果您想要保留您的機器學習模型，但您想要以新的資料重新定型，您有兩個選擇：

1. **Web 服務執行時重新訓練模型** - 如果您想要在預測性 Web 服務執行時重新訓練模型，您可以藉由對訓練實驗進行幾個修改，使它成為***重新訓練實驗***，然後可以將它部署為***重新訓練 Web* 服務**。 如需如何執行這項操作的指示，請參閱 [以程式設計方式重新定型機器學習服務模型](retrain-models-programmatically.md)。
2. **返回原始訓練實驗並使用不同的訓練資料來開發您的模型** - 您的預測實驗連結至 Web 服務，但訓練實驗未以這種方式直接連結。 如果您修改原始的訓練實驗，並按一下 [設定 Web 服務]，它會建立「新的」預測實驗，部署時將會建立「新的」Web 服務。 它不只是更新原始的 Web 服務。
   
   如果您需要修改訓練實驗，請開啟它並按一下 [另存新檔] 以製作複本。 這將會原始的訓練實驗、預測實驗和 Web 服務維持不變。 您現在可以利用您的變更建立新的 Web 服務。 部署新的 Web 服務之後，可以再決定是否要停止先前的 Web 服務，或讓它隨著新的服務一起執行。

**您想要定型不同的模型**

如果您想要對原始預測性實驗進行變更，例如選取不同的機器學習演算法、嘗試不同的定型方法等，則您必須遵循上述用於重新定型模型的第二個程序：開啟訓練實驗、按一下 [另存新檔]  來製作副本，然後開始開發模型的新路徑、建立預測性實驗和部署 Web 服務。 這會建立與原始無關的新 Web 服務 - 您可以決定要繼續執行哪一個或兩者。

## <a name="next-steps"></a>後續步驟
如需開發和實驗程序的詳細資訊，請參閱下列文章︰

* 轉換實驗 - [如何準備您的模型以在 Azure Machine Learning Studio 中部署](convert-training-experiment-to-scoring-experiment.md)
* 部署 Web 服務 - [部署 Azure Machine Learning Web 服務](publish-a-machine-learning-web-service.md)
* 重新定型模型 - [以程式設計方式重新定型機器學習服務模型](retrain-models-programmatically.md)

如需整個程序的範例，請參閱：

* [機器學習教學課程：在 Azure Machine Learning Studio 中建立您的第一個實驗](create-experiment.md)
* [逐步解說：在 Azure Machine Learning 中為信用風險評估開發預測性分析解決方案](walkthrough-develop-predictive-solution.md)

