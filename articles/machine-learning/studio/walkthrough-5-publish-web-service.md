---
title: 步驟 5：部署 Machine Learning Web 服務 | Microsoft Docs
description: 開發預測解決方案逐步解說的步驟 5：在 Machine Learning Studio 中將預測實驗部署為 Web 服務。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: 3fca74a3-c44b-4583-a218-c14c46ee5338
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/23/2017
ms.openlocfilehash: 436656195e00311dd350a5526b01fffa56ac02ca
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "40246531"
---
# <a name="walkthrough-step-5-deploy-the-azure-machine-learning-web-service"></a>逐步解說步驟 5：部署 Azure Machine Learning Web 服務
這是 [在 Azure Machine Learning 中為信用風險評估開發預測性分析解決方案](walkthrough-develop-predictive-solution.md)

1. [建立機器學習服務工作區](walkthrough-1-create-ml-workspace.md)
2. [上傳現有資料](walkthrough-2-upload-data.md)
3. [建立新實驗](walkthrough-3-create-new-experiment.md)
4. [訓練及評估模型](walkthrough-4-train-and-evaluate-models.md)
5. **部署 Web 服務**
6. [存取 Web 服務](walkthrough-6-access-web-service.md)

- - -
為了讓其他人有機會使用此逐步解說中開發的預測模型，我們可以將它部署為 Azure 上的 Web 服務。

截至目前為止，我們一直在試驗如何定型模型。 但是已部署的服務就不會再進行訓練，它會根據我們的模型，對使用者的輸入計分以產生預測。 所以我們要進行一些準備工作，將此實驗從***訓練***實驗轉換為***預測***實驗。 

這是一個三步驟程序：  

1. 移除其中一個模型
2. 將我們所建立的*訓練實驗*轉換成*預測實驗*
3. 將預測實驗部署為 Web 服務

## <a name="remove-one-of-the-models"></a>移除其中一個模型

首先，我們需要更精簡此實驗。 我們目前在實驗中有兩個不同的模型，但將此實驗部署為 Web 服務時，我們只要一個模型。  

假設我們已判斷出促進式樹狀模型比 SVM 模型更適合。 因此，首要之務是移除[二元支援向量機器][two-class-support-vector-machine]模組，以及用於訓練該模組的其他模組。 您可能需要按一下實驗畫布底部的 [ **另存新檔** ]，先建立一份實驗複本。

我們必須刪除下列模組：  

* [二元支援向量機器][two-class-support-vector-machine]
* [訓練模型][train-model]和與之連接的[評分模型][score-model]模組
* [標準化資料][normalize-data] (兩者)
* [評估模型][evaluate-model] (因為我們已完成模型的評估)

選取每個模組然後按 Delete 鍵，或用右鍵按一下模組並選取 [刪除]。 

![已移除 SVM 模型][3a]

我們的模型現在看起來應該像這樣：

![已移除 SVM 模型][3]

現在我們已經準備好使用[二元促進式決策樹][two-class-boosted-decision-tree]來部署此模型。

## <a name="convert-the-training-experiment-to-a-predictive-experiment"></a>將訓練實驗轉換為預測實驗

若要備妥此模型以進行部署，我們需要將此訓練實驗轉換為預測實驗。 這牽涉到三個步驟：

1. 儲存已定型的模型，並取代我們的定型模組
2. 精簡實驗，移除只有定型才需要的模組
3. 定義 Web 服務將接受輸入的位置和產生輸出的位置

我們可以手動進行此動作，但幸好上述三個步驟只要按一下實驗畫布底端的 [設定 Web 服務] (然後選取 [預測性 Web 服務] 選項) 即可完成。

> [!TIP]
> 如需將訓練實驗轉換成預測實驗的詳細資訊，請參閱[如何準備您的模型以在 Azure Machine Learning Studio 中部署](convert-training-experiment-to-scoring-experiment.md)。

當您按一下 [設定 Web 服務] ，會發生幾件事：

* 定型的模型會轉換為單一**定型模型**模組，並存放在實驗畫布左側的模組選擇區中 (您可以在 [定型模型] 下找到這個模組)
* 用於定型的模組已遭到移除，尤其是：
  * [二元促進式決策樹][two-class-boosted-decision-tree]
  * [定型模型][train-model]
  * [資料分割][split]
  * 用於測試資料的第二個[執行 R 指令碼][execute-r-script]模組
* 儲存的定型模型會加回實驗中
* 會新增 **Web 服務輸入**和 **Web 服務輸出**模組 (這些可用來辨識使用者的資料將在何處輸入模型中、傳回什麼資料、在何時存取 Web 服務)

> [!NOTE]
> 您可以看到，實驗已儲存在已新增至實驗畫布頂端之索引標籤下的兩個部分。 原始的訓練實驗位於 [訓練實驗] 索引標籤底下，新建立的預測實驗位於 [預測實驗] 底下。 我們將預測實驗部署為 Web 服務。

我們需要對此特別的實驗採取一個額外步驟。
我們新增了兩個[執行 R 指令碼][execute-r-script]模組來替資料提供加權函式。 這只是我們為了訓練和測試所用的技巧，因此我們可以從最終模型中拿掉這些模組。
Machine Learning Studio 已在移除[分割][split]模組時移除一個[執行 R 指令碼][execute-r-script]模組。 現在我們可以移除另一個模組，並直接將[中繼資料編輯器][metadata-editor]連接至[評分模型][score-model]。    

實驗現在看起來如下：  

![Scoring the trained model][4]  

> [!NOTE]
> 您可能驚訝我們為什麼要在預測實驗中留下「UCI 德國信用卡資料」資料集。 服務即將評分使用者的資料，而不是原始資料集，所以為何要讓原始資料集留在模型中？
> 
> 服務的確不需要原始信用卡資料。 但卻需要該資料的結構描述，例如，有多少個資料行及哪些資料行是數值等資訊。 需要此結構描述資訊才能解譯使用者的資料。 我們保持連接這些元件，以便服務執行時，評分模型才會有資料集結構描述。 不會使用資料，只是使用結構描述。  
> 
>一個值得注意的重要事項是若您的原始資料集包含標籤，則來自 Web 輸入的預期結構描述也預期應該有具有標籤的資料行！ 避免此問題的一個因應措施是移除標籤，以及在將 Web 輸入與定型資料集連結到通用模組之前，移除位於定型資料集但將不會位於 Web 輸入中的任何其他資料。 
> 

最後一次執行實驗 (按一下 **[執行]**)。如果要確認模型仍然有效，請按一下 [評分模型][score-model]模組的輸出結果，並選取 [檢視結果]。 您可以看到原始資料出現，也會看到信用風險值 ("Scored Labels") 和評分機率值 ("Scored Probabilities")。 

## <a name="deploy-the-web-service"></a>部署 Web 服務
您可以將實驗部署為傳統 Web 服務或架構在 Azure Resource Manager 上的新式 Web 服務。

### <a name="deploy-as-a-classic-web-service"></a>部署為傳統 Web 服務
若要部署衍生自實驗的傳統 Web 服務，請按一下畫布下方的 [部署 Web 服務]，然後選取 [部署 Web 服務 [傳統]]。 Machine Learning Studio 會將實驗當做 Web 服務部署，並帶您前往 Web 服務的儀表板。 您可以從此頁面返回實驗 ([檢視快照] 或 [檢視最新])，並執行簡單的 Web 服務測試 (請參閱下面的**測試 Web 服務**)。 另外這裡還有建立可存取 Web 服務之應用程式的資訊 (此逐步說明的下一個步驟中有該資訊的詳細資料)。

![Web 服務儀表板][6]

您可以按一下 [設定]  索引標籤來設定服務。您可以在這裡修改服務名稱 (預設會指定實驗名稱) 並輸入描述。 您也可以為輸入和輸出資料指定更好記的標籤。  

![設定 Web 服務][5]  

### <a name="deploy-as-a-new-web-service"></a>部署為新式 Web 服務

> [!NOTE] 
> 若要部署新式 Web 服務，您必須在要部署 Web 服務的訂用帳戶中具備足夠的權限。 如需詳細資訊，請參閱[使用 Azure Machine Learning Web 服務入口網站管理 Web 服務](manage-new-webservice.md)。 

部署衍生自實驗的新式 Web 服務：

1. 按一下畫布底下的 [部署 Web 服務]，然後選取 [部署 Web 服務 [新式]]。 Machine Learning Studio 會帶您到 Azure Machine Learning Web 服務的 [部署實驗] 頁面。

2. 輸入 Web 服務的名稱。 

3. 在 **價格方案**，您可以選取現有的定價方案，或選取 [建立新的] 並指定新的方案的名稱，然後選取每月方案選項。 方案層預設為您預設區域的方案，而您的 Web 服務會部署到該區域。

4. 按一下 [ **部署**]。

幾分鐘後，Web 服務的 [快速入門] 頁面就會開啟。

您可以按一下 [設定] 索引標籤來設定服務。您可以在這裡修改服務標題並輸入描述。 

若要測試 Web 服務，按一下 [測試] 索引標籤 (請參閱下面的**測試 Web 服務**)。 如需建立應用程式以存取 Web 服務的相關資訊，請按一下 [取用] 索引標籤 (本逐步解說的下一個步驟將提供更多詳細資料)。

> [!TIP]
> 您可以在部署 Web 服務之後進行更新。 例如，如果想要變更模型，則可以編輯訓練實驗、調整模型參數，然後按一下 [部署 Web 服務]，選取 [部署 Web 服務 [傳統]] 或 [部署 Web 服務 [新式]]。 重新部署實驗時，將會取代 Web 服務 (現在使用的是已更新的模型)。  
> 
> 

## <a name="test-the-web-service"></a>測試 Web 服務

存取 Web 服務時，使用者的資料會透過 **Web 服務輸入**模組輸入，再透過它傳遞給[評分模型][score-model]模組進行評分。 基於我們設定預測實驗的方式，模型預期會得到與原始信用風險資料集相同格式的資料。
接著透過 **Web 服務輸出**模組，將結果從 Web 服務傳回給使用者。

> [!TIP]
> 基於我們設定預測實驗的方式，會傳回[評分模型][score-model]模組的整個結果。 這包括所有輸入的資料以及信用風險值和評分機率。 但您可以視需要傳回不同的項目，例如，您可以只傳回信用風險值。 若要這樣做，請在[評分模型][score-model]和 **Web 服務輸出**之間插入[專案資料行][project-columns]模組，以排除您不想讓 Web 服務傳回的資料行。 
> 
> 

您可以在 **Machine Learning Studio** 或 **Azure Machine Learning Web 服務**入口網站中測試傳統 Web 服務。
您只能在 **Machine Learning Web 服務**入口網站中測試新式 Web 服務。

> [!TIP]
> 在 Azure Machine Learning Web 服務入口網站中測試時，您可以用入口網站建立範例資料以用於測試要求-回應服務。 在 [設定] 頁面上，[啟用範例資料嗎?] 選取 [是]。 當您開啟 [測試] 頁面上的 [要求-回應] 索引標籤時，入口網站會填入取自原始信用風險資料集的範例資料。

### <a name="test-a-classic-web-service"></a>測試傳統 Web 服務

您可以在 Machine Learning Studio 或 Machine Learning Web 服務入口網站中測試傳統 Web 服務。 

#### <a name="test-in-machine-learning-studio"></a>在 Machine Learning Studio 中測試

1. 在 Web 服務的 [儀表板] 頁面上，按一下 [預設端點] 下的 [測試] 按鈕。 對話方塊隨即顯示，要求您提供服務的輸入資料。 這些就是在原始的信用風險資料集中出現的資料行。  

2. 輸入一組資料，然後按一下 [確定] 。 

#### <a name="test-in-the-machine-learning-web-services-portal"></a>在 Machine Learning Web 服務入口網站中測試

1. 在 Web 服務的 [儀表板] 頁面上，按一下 [預設端點] 下的 [測試預覽] 連結。 Azure Machine Learning Web 服務入口網站中的 Web 服務端點測試頁面隨即開啟，並要求您提供服務的輸入資料。 這些就是在原始的信用風險資料集中出現的資料行。

2. 按一下 [測試要求-回應]。 

### <a name="test-a-new-web-service"></a>測試新式 Web 服務

您只能在 Machine Learning Web 服務入口網站中測試新式 Web 服務。

1. 在 [Azure Machine Learning Web 服務](https://services.azureml.net/quickstart)入口網站中，按一下頁面頂端的 [測試]。 [測試] 頁面會開啟，您可以輸入服務的資料。 顯示的輸入欄位會對應到在原始的信用風險資料集中出現的資料行。 

2. 輸入一組資料，然後按一下 [測試要求-回應] 。

測試的結果會顯示在頁面右邊的輸出資料行上。 


## <a name="manage-the-web-service"></a>管理 Web 服務

一旦部署 Web 服務 (傳統或新式) 之後，您就可以從 [Microsoft Azure Machine Learning Web 服務](https://services.azureml.net/quickstart)入口網站管理它。

監視 Web 服務的效能：

1. 登入 [Microsoft Azure Machine Learning Web 服務](https://services.azureml.net/quickstart)入口網站
2. 按一下 [Web 服務]
3. 按一下您的 Web 服務
4. 按一下 [儀表板]

- - -
**下一步：[存取 Web 服務](walkthrough-6-access-web-service.md)**

[3]: ./media/walkthrough-5-publish-web-service/publish3.png
[3a]: ./media/walkthrough-5-publish-web-service/publish3a.png
[4]: ./media/walkthrough-5-publish-web-service/publish4.png
[5]: ./media/walkthrough-5-publish-web-service/publish5.png
[6]: ./media/walkthrough-5-publish-web-service/publish6.png


<!-- Module References -->
[evaluate-model]: https://msdn.microsoft.com/library/azure/927d65ac-3b50-4694-9903-20f6c1672089/
[execute-r-script]: https://msdn.microsoft.com/library/azure/30806023-392b-42e0-94d6-6b775a6e0fd5/
[metadata-editor]: https://msdn.microsoft.com/library/azure/370b6676-c11c-486f-bf73-35349f842a66/
[normalize-data]: https://msdn.microsoft.com/library/azure/986df333-6748-4b85-923d-871df70d6aaf/
[score-model]: https://msdn.microsoft.com/library/azure/401b4f92-e724-4d5a-be81-d5b0ff9bdb33/
[split]: https://msdn.microsoft.com/library/azure/70530644-c97a-4ab6-85f7-88bf30a8be5f/
[train-model]: https://msdn.microsoft.com/library/azure/5cc7053e-aa30-450d-96c0-dae4be720977/
[two-class-boosted-decision-tree]: https://msdn.microsoft.com/library/azure/e3c522f8-53d9-4829-8ea4-5c6a6b75330c/
[two-class-support-vector-machine]: https://msdn.microsoft.com/library/azure/12d8479b-74b4-4e67-b8de-d32867380e20/
[project-columns]: https://msdn.microsoft.com/library/azure/1ec722fa-b623-4e26-a44e-a50c6d726223/
