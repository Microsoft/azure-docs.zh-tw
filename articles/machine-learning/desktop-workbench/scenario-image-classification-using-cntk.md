---
title: 使用 Azure Machine Learning Workbench 內部的 CNTK 進行影像分類 |Microsoft Docs
description: 使用 Azure ML Workbench 來訓練、評估及部署自訂影像分類模型。
services: machine-learning
documentationcenter: ''
author: PatrickBue
ms.author: pabuehle
manager: mwinkle
ms.reviewer: marhamil, mldocs, garyericson, jasonwhowell
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 10/17/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 83d6f529330a05e6a7c46ad45b19f0338f93bfc7
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46995086"
---
# <a name="image-classification-using-azure-machine-learning-workbench"></a>使用 Azure Machine Learning Workbench 進行影像分類

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



影像分類方法可用來解決大量電腦視覺問題。
其中包括建置模型，用來回答下列問題，例如：*影像中是否有物件？* 其中物件舉例來說可能是「狗」、「汽車」或「船艦」。 或是更複雜的問題，像是：「此病患的視網膜掃描顯示的眼睛疾病嚴重性等級為何？」。

本教學課程說明如何解決這類問題。 我們示範如何使用 [Microsoft Cognitive Toolkit (CNTK)](https://docs.microsoft.com/cognitive-toolkit/) 來訓練、評估及部署您自己的影像分類模型，以進行深入了解。
會提供範例影像，但讀取器也可以攜帶自己的資料集，並訓練自己的自訂模型。

電腦視覺解決方案傳統上需要專業知識，以手動方式找出並實作所謂的「特徵」，用來強調影像中的所需資訊。
這個手動方法在 2012 年因知名的 [AlexNet](https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf) [1] 深度學習論文而改變；目前，深度類神經網路 (DNN) 用來自動找出這些特徵。
DNN 不僅在影像分類的領域上有重大的改善，在物件偵測和影像相似度等其他電腦視覺問題方面也有重大的改善。


## <a name="link-to-the-gallery-github-repository"></a>連結至資源庫 GitHub 存放庫
[https://github.com/Azure/MachineLearningSamples-ImageClassificationUsingCNTK](https://github.com/Azure/MachineLearningSamples-ImageClassificationUsingCNTK)

## <a name="overview"></a>概觀

本教學課程分成三個部分：

- 第 1 部分示範如何使用預先訓練的 DNN 作為 Featurizer 並根據其輸出訓練 SVM，以訓練、評估和部署影像分類系統。
- 第 2 部分顯示如何透過調整 DNN (舉例來說)，而不是使用它作為固定 Featurizer 來提高精確度。
- 第 3 部分提供如何使用您自己的資料集，而不是提供的範例影像，並在需要時，說明如何從網路擷取影像來產生您自己的資料集。

雖然不需要使用機器學習和 CNTK 的先前經驗，但其有助於了解基本準則。 教學課程中報告的精確數字、訓練時間等僅供參考，執行程式碼時的實際值幾乎不一樣。


## <a name="prerequisites"></a>必要條件

要執行此範例所需符合的必要條件如下：

1. [Azure 帳戶](https://azure.microsoft.com/free/) (有提供免費試用)。
2. 遵循[快速入門安裝指南](quickstart-installation.md)的 [Azure Machine Learning Workbench](../service/overview-what-is-azure-ml.md) 版本，以便安裝此程式並建立 Workbench。  
3. 一部 Windows 電腦。 Windows 作業系統是必要項目，因為 Workbench 僅支援 Windows 和 MacOS，而 Microsoft 的 Cognitive Toolkit (我們使用它作為深入學習程式庫) 只支援 Windows 和 Linux。
4. 不需要專用 GPU 來執行第 1 部分的 SVM 訓練，但需要它來進行第 2 部分所描述的 DNN 調整作業。 如果您缺乏強大的 GPU、想要在多個 GPU 上訓練或沒有一部 Windows 電腦，請考慮搭配使用 Azure 的深度學習虛擬機器與 Windows 作業系統。 請參閱[這裡](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning)以取得一鍵部署指南。 一旦部署之後，請透過遠端桌面連線連接到 VM，在該處安裝 Workbench，並在 VM 的本機上執行程式碼。
5. 需要安裝各種 Python 程式庫，例如 OpenCV。 從 Workbench 的 [檔案] 功能表按一下 [開啟命令提示字元]，並執行下列命令來安裝這些相依性：  
    - `pip install https://cntk.ai/PythonWheel/GPU/cntk-2.2-cp35-cp35m-win_amd64.whl`  
    - 從 http://www.lfd.uci.edu/~gohlke/pythonlibs/ 下載 OpenCV Wheel 之後，執行 `pip install opencv_python-3.3.1-cp35-cp35m-win_amd64.whl` (確切的檔名和版本可能變更)
    - `conda install pillow`
    - `pip install -U numpy`
    - `pip install bqplot`
    - `jupyter nbextension enable --py --sys-prefix bqplot`
    - `jupyter nbextension enable --py widgetsnbextension`

### <a name="troubleshooting--known-bugs"></a>疑難排解/已知錯誤
- 第 2 部分需要 GPU，否則在嘗試調整 DNN 時會擲回錯誤「尚未實作 CPU 上的批次正規化訓練」。
- 透過降低迷你批次的大小 (`PARAMETERS.py`中的變數 `cntk_mb_size`)，即可避免在 DNN 訓練期間發生記憶體不足錯誤。
- 此程式碼已使用 CNTK 2.2 進行測試，並且也應該在沒有任何微幅變更的情況下，在較舊 (最高到 v2.0) 和較新版本上執行。
- 在撰寫本文時，Azure Machine Learning Workbench 出現筆記本大於 5 MB 的問題。 如果在顯示所有儲存格輸出的情況下儲存筆記本，便可能會出現這種大型的筆記本。 如果您遇到這個錯誤，則從 Workbench 內部的 [檔案] 功能表開啟命令提示字元、執行 `jupyter notebook`、開啟筆記本、清除所有輸出，然後儲存筆記本。 執行這些步驟之後，筆記本會在 Azure Machine Learning Workbench 內部再次正確開啟。
- 此範例中提供的所有指令碼必須在本機執行，而不是在像 Docker 這樣的遠端環境中執行。 所有筆記本都必須使用設定為本機專案核心的核心來執行，該核心名為 "PROJECTNAME local" (例如 "myImgClassUsingCNTK local")。

    
## <a name="create-a-new-workbench-project"></a>建立新的 Workbench 專案

使用此範例作為範本來建立新專案：
1.  開啟 Azure Machine Learning Workbench。
2.  在 [專案] 頁面上，按一下 **+** 符號，然後選取 [新增專案]。
3.  在 [建立新專案] 窗格中，填入新專案的資訊。
4.  在 [搜尋專案範本] 搜尋方塊中，輸入「影像分類」並選取範本。
5.  按一下頁面底部的 [新增] 。

執行這些步驟會建立專案結構，如下所示。 專案目錄限制為小於 25 MB，因為 Azure Machine Learning Workbench 在每次執行後都會建立這個資料夾的複本 (以啟用執行歷程記錄)。 因此，所有影像和暫存檔案都是在目錄 *~/Desktop/imgClassificationUsingCntk_data* (本文件中稱為 *DATA_DIR*) 中來回儲存。

  資料夾| 說明
  ---|---
  aml_config/|                           包含 Azure Machine Learning Workbench 設定檔的目錄
  libraries/|                              包含所有 Python 和 Jupyter 協助程式函式的目錄
  notebooks/|                              包含所有筆記本的目錄
  resources/|                              包含所有資源的目錄 (例如時裝影像的 URL)
  scripts/|                              包含所有指令碼的目錄
  PARAMETERS.py|                       指定所有參數的 Python 指令碼
  readme.md|                           這個讀我檔案文件


## <a name="data-description"></a>資料說明

本教學課程使用最多由 428 個影像所組成的上衣紋理資料集作為執行範例。 每個影像都會標註為三種不同紋理 (圖點、條紋、豹紋) 的其中一個。 我們將影像數目保持在少量，以便可以快速執行本教學課程。 不過，程式碼已經過完整測試，可處理數以萬計的影像或更多影像。 所有影像都已手動標註，如[第 3 部分](#using-a-custom-dataset)中所述。 影像 URL 及其各自屬性都列在 */resources/fashionTextureUrls.tsv* 檔案中。

指令碼 `0_downloadData.py` 會將所有影像下載到 *DATA_DIR/images/fashionTexture/* 目錄。 428 個 URL 中可能有某些 URL 已中斷。 這不構成問題，而只表示我們進行訓練和測試的影像稍微較少。 此範例中提供的所有指令碼必須在本機執行，而不是在像 Docker 這樣的遠端環境中執行。

下圖顯示圓點 (左側)、條紋 (中間) 和豹紋 (右側) 特徵的範例。 會根據上衣項目完成註解。

<p align="center">
<img src="media/scenario-image-classification-using-cntk/examples_all.jpg"  alt="alt text" width="700">
</p>


## <a name="part-1---model-training-and-evaluation"></a>第 1 部分 - 模型訓練和評估

在本教學課程的第一個部分中，我們要訓練一個系統，此系統會使用但不會修改預先訓練的深度類神經網路。 使用這個預先訓練的 DNN 作為 Featurizer，而線性 SVM 則經過訓練，用來預測指定影像的屬性 (dotted、striped 或 leopard)。

現在，我們會詳細且逐步描述這種方式，並顯示必須執行的指令碼。 我們建議在每個步驟之後檢查寫入了哪些檔案，以及這些檔案寫入的位置。

所有重要參數的指定及簡短說明的提供都在單一位置中進行：`PARAMETERS.py` 檔案。




### <a name="step-1-data-preparation"></a>步驟 1：資料準備
`Script: 1_prepareData.py. Notebook: showImages.ipynb`

`showImages.ipynb` 筆記本可用來將影像視覺化，並視需要修正其註解。 若要執行筆記本，請在 Azure Machine Learning Workbench 中開啟它，在 [啟動筆記本伺服器] 顯示時按一下此選項，變更為名為 "PROJECTNAME local" (例如 "myImgClassUsingCNTK local") 的本機專案核心，然後執行筆記本中的所有儲存格。 如果您收到提報筆記本太大而無法顯示的錯誤，請參閱本文件中的＜疑難排解＞一節。
<p align="center">
<img src="media/scenario-image-classification-using-cntk/notebook_showImages.jpg" alt="alt text" width="700"/>
</p>

現在，執行名為 `1_prepareData.py` 的指令碼，其可將所有影像指派給訓練集或測試集。 這項指派是互斥的 - 沒有訓練影像會同時用於測試，反之亦然。 根據預設，每個屬性類別中隨機的 75%　影像指派給訓練，而剩餘的 25% 則指派給測試。 指令碼產生的所有資料都會儲存在 *DATA_DIR/proc/fashionTexture/* 資料夾中。

<p align="center">
<img src="media/scenario-image-classification-using-cntk/output_script_1_white.jpg" alt="alt text" width="700"/>
</p>



### <a name="step-2-refining-the-deep-neural-network"></a>步驟 2：調整深度類神經網路
`Script: 2_refineDNN.py`

如同我們在本教學課程的第 1 部分所說明，預先訓練的 DNN 會保持固定 (也就是說，它不會調整)。 不過，名為 `2_refineDNN.py` 的指令碼仍會在第 1 部分中執行，因為它會載入預先訓練的 [ResNet](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/He_Deep_Residual_Learning_CVPR_2016_paper.pdf) [2] 模型並加以修改；舉例來說，用以允許更高的輸入影像解析度。 這個步驟的速度很快 (秒)，而且不需要 GPU。

在本教學課程的第 2 部分中，對 PARAMETERS.py 檔案的修改會造成 `2_refineDNN.py` 指令碼也調整預先訓練的 DNN。 根據預設，我們會在調整期間執行 45 個訓練 Epoch。

在這兩種情況下，最終模型接著會寫入至檔案 *DATA_DIR/proc/fashionTexture/cntk_fixed.model*。

### <a name="step-3-evaluate-dnn-for-all-images"></a>步驟 3：評估所有影像的 DNN
`Script: 3_runDNN.py`

現在，我們可以使用最後一個步驟中可能已調整的 DNN，為我們的影像賦予特徵。 指定某個影像作為 DNN 的輸入，則輸出是從模型的倒數第二層產生的 512 浮點數向量。 此向量的維度比影像本身小得多。 不過，它應該包含 (甚至強調) 影像中與辨識影像屬性相關的所有資訊；也就是說，服裝項目具有圓點、條紋或豹紋紋理的話。

所有 DNN 影像表示都會儲存到 *DATA_DIR/proc/fashionTexture/cntkFiles/features.pickle* 檔案中。

<p align="center">
<img src="media/scenario-image-classification-using-cntk/output_script_4_white.jpg" alt="alt text" width="700"/>
</p>


### <a name="step-4-support-vector-machine-training"></a>步驟 4：支援向量機器訓練
`Script: 4_trainSVM.py`

最後一個步驟中所計算的 512 浮點數表示現在用於訓練 SVM 分類器：指定某個影像作為輸入，SVM 會輸出要顯示之每個屬性的分數。 在我們的範例資料集中，這表示 'striped'、'dotted' 和 'leopard' 的分數。

`4_trainSVM.py` 指令碼會載入訓練影像、針對正則化 (放寬) 參數 C 的不同值訓練 SVM，以及使用最高的精確度保留 SVM。 分類精確度會列印到主控台，並在 Workbench.中進行繪製。 對於提供的紋理資料，這些值應該分別是約 100% 和 88%。 最後，已訓練的 SVM 會寫入到 *DATA_DIR/proc/fashionTexture/cntkFiles/svm.np* 檔案中。

<p align="center">
<img src="media/scenario-image-classification-using-cntk/vienna_svm_log_zoom.jpg" alt="alt text" width="700"/>
</p>



### <a name="step-5-evaluation-and-visualization"></a>步驟 5：評估和視覺效果
`Script: 5_evaluate.py. Notebook: showResults.ipynb`

您可以使用 `5_evaluate.py` 指令碼測量已訓練影像分類的精確度。 此指令碼會使用訓練的 SVM 分類器為所有測試影像評分、為每個影像指派具有最高分數的屬性，以及比較具有地面實測資料註解的預測屬性。

`5_evaluate.py` 指令碼的輸出如下所示。 會計算每個個別類別的分類精確度，以及完整測試集的精確度 ('overall accuracy') 和個別精確度的平均值 ('overall class-averaged accuracy')。 100% 對應於最佳的可能精確度，而 0% 則對應於最差。 隨機猜測一般會產生屬性數目的類別平均精確度 1：在我們的案例中，此精確度是 33.33%。 當您使用較高的輸入解析度 (例如 `rf_inputResoluton = 1000`) 時，這些結果會大幅改善，但代價是較長的 DNN 計算時間。

<p align="center">
<img src="media/scenario-image-classification-using-cntk/output_script_6_white.jpg" alt="alt text" width="700"/>
</p>

除了精確度以外，還會繪製曲線下具有各別區域的 ROC 曲線 (左)；並且顯示混淆矩陣 (右)：

<p align="center">
<img src="media/scenario-image-classification-using-cntk/roc_confMat.jpg" alt="alt text" width="700"/>
</p>

最後，提供 `showResults.py` 筆記本來捲動瀏覽測試影像，並將其各自的分類分數視覺化。 依據步驟 1 中的說明，此範例中的每個筆記本都需要使用名為 "PROJECTNAME local" 的本機專案核心：
<p align="center">
<img src="media/scenario-image-classification-using-cntk/notebook_showResults.jpg" alt="alt text" width="700"/>
</p>





### <a name="step-6-deployment"></a>步驟 6：部署
`Scripts: 6_callWebservice.py, deploymain.py. Notebook: deploy.ipynb`

訓練的系統現在可以發行為 REST API。 筆記本 `deploy.ipynb` 中會說明部署，並以 Azure Machine Learning Workbench (請記得設為名為 "PROJECTNAME local" 的本機專案核心) 內的功能為基礎。 另請參閱[鳶尾花教學課程](tutorial-classifying-iris-part-3.md)中的絕佳部署一節，以了解更多部署相關資訊。

一旦部署之後，就可以使用 `6_callWebservice.py` 指令碼來呼叫 Web 服務。 請注意，必須先在指令碼中設定 Web 服務的 IP 位址 (本機或雲端上)。 `deploy.ipynb` 筆記本說明如何尋找此 IP 位址。








## <a name="part-2---accuracy-improvements"></a>第 2 部分 - 提高精確度

在第 1 部分中，我們示範了如何根據深度類神經網路的 512 浮動數輸出訓練線性的支援向量機器，以便分類影像。 此 DNN 已根據數百萬的影像進行預先訓練，而倒數第二層會傳回為特徵向量。 因為 DNN 是以現狀使用，所以這個方法很快速，但卻通常可以提供良好的結果。

現在，我們提供數種方式來提高第 1 部分模型的精確度。 最值得注意的是我們會調整 DNN，而不是保持固定。

### <a name="dnn-refinement"></a>DNN 調整

可以直接在類神經網路中進行分類，而不是使用 SVM。 將新的最後一層新增至預先訓練的 DNN，進而採用倒數第二層的 512 浮點數作為輸入，即可達到此目的。 在 DNN 中進行分類的好處是：現在可以使用反向傳播重新訓練完整網路。 相較於以現況使用預先訓練的 DNN，這種方法通常會產生更好的分類精確度，不過代價是更長的訓練時間 (即使具有 GPU)。

藉由將 `PARAMETERS.py` 中的變數 `classifier` 從 `svm` 變更為 `dnn`，即可訓練類神經網路而不是 SVM。 接著，如第 1 部分所述，必須再執行一次除了資料準備 (步驟 1) 和 SVM 訓練 (步驟 4) 以外的所有指令碼。 DNN 調整需要有 GPU。 如果找不到 GPU，或者如果 GPU 已鎖定 (例如由先前的 CNTK 執行)，則 `2_refineDNN.py` 指令碼會擲回錯誤。 透過降低迷你批次的大小 (`PARAMETERS.py` 中的變數 `cntk_mb_size`)，即可避免在 DNN 訓練期間發生記憶體不足錯誤。

訓練完成之後，調整的模型會儲存到 *DATA_DIR/proc/fashionTexture/cntk_refined.model* 中，並繪製一個繪圖，以顯示訓練和測試分類錯誤在訓練期間如何變化。 在該繪圖中，請注意訓練集的錯誤遠小於測試集的錯誤。 比方說，藉由對中輟速率 `rf_dropoutRate` 使用較高的值，可降低這個所謂的過度調整行為。
<p align="center">
<img src="media/scenario-image-classification-using-cntk/output_script_3_plot.png" alt="alt text" height="300"/>
</p>

從下列繪圖中可以看出，對提供的資料集使用 DNN 調整和使用之前的精確度分別是 92.35%與 88.92% (第 1 部分)。 特別是，'dotted' 影像會大幅改善，其 ROC 曲線下的區域在調整和調整之前各為 0.98 與0.94。 我們使用的是小型資料集，因此執行程式碼的實際精確度會不同。 這項差異是由於隨機效果 (例如隨機將影像分成訓練集和測試集) 所導致。
<p align="center">
<img src="media/scenario-image-classification-using-cntk/roc_confMat_dnn.jpg" alt="alt text" width="700"/>
</p>

### <a name="run-history-tracking"></a>執行歷程記錄追蹤

Azure Machine Learning Workbench 會儲存 Azure上每次執行的歷程記錄，以允許比較甚至相隔幾週的兩個或多個執行。 這在[光圈教學課程](tutorial-classifying-iris-part-2.md)中有詳細說明。 下列螢幕擷取畫面也會說明此項，我們在該畫面中會使用 DNN 調整即 `classifier = "dnn"`(執行編號 148) 或 SVM 訓練即 `classifier = "svm"` (執行編號 150) 來比較 `5_evaluate.py` 指令碼的兩個執行。

在第一個螢幕擷取畫面中，DNN 調整會針對所有類別，產生比 SVM 訓練更佳的精確度。 第二個螢幕擷取畫面會顯示所要追蹤的所有度量，包括分類器為何。 這項追蹤會藉由呼叫 Azure Machine Learning Workbench 記錄器，在指令碼 `5_evaluate.py` 中完成。 此外，指令碼也會將 ROC 曲線和混淆矩陣儲存到 *outputs* 資料夾。 這個 *outputs* 資料夾的特殊之處，在於其內容也是由 Workbench 歷程記錄功能進行追蹤，因此不論本機複本是否已被覆寫，都可以隨時存取輸出檔案。

<p align="center">
<img src="media/scenario-image-classification-using-cntk/run_comparison1.jpg" alt="alt text" width="700"/>
</p>

<p align="center">
<img src="media/scenario-image-classification-using-cntk/run_comparison2b.jpg" alt="alt text" width="700"/>
</p>


### <a name="parameter-tuning"></a>參數調整

與大部分的機器學習專案一樣，若要為新資料集取得良好結果，則需要仔細調整參數，以及評估不同的設計決策。 為了協助完成這些作業，所有重要參數的指定及簡短說明的提供都在單一位置中進行：`PARAMETERS.py` 檔案。

一些最有可能改善的途徑如下：

- 資料品質：確保訓練和測試集具有高品質。 也就是說，影像已正確標註、移除了模稜兩可的影像 (例如，具有條紋和圓點的服裝項目)，而且屬性互斥 (即選擇的每個影像只屬於一個屬性)。

- 如果影像中的相關物件很小，影像分類方法已知無法正常運作。 在這種情況下，請考慮使用物件偵測方法，如本[教學課程](https://github.com/Azure/ObjectDetectionUsingCntk)中所述。
- DNN 調整：正常運行的最重要參數可以說是學習速率 `rf_lrPerMb`。 如果訓練集的精確度 (第 2 部分中的第一個圖形) 未接近於 0-5%，最有可能是因為學習速率錯誤。 其他以 `rf_` 為開頭的參數較不重要。 一般而言，訓練錯誤應該以指數方式遞減，而且在訓練之後接近於 0%。

- 輸入解析度：預設的影像解析度為 224 x 224 像素。 使用較高的影像解析度 (參數：`rf_inputResoluton`)，例如 448 x 448 或 896 x 896 像素，通常會大幅提高精確度，但會使 DNN 調整變慢。 **使用較高的影像解析度近乎免費，而且幾乎可以一律提升精確度**。

- DNN 過度調整：在 DNN 調整期間避免訓練和測試精確度之間有太大的間距 (第 2 部分的第一個圖形)。 此間距可以使用中輟速率 `rf_dropoutRate` 0.5 以上，並藉由增加正則化程式加權 `rf_l2RegWeight` 來降低。 如果 DNN 輸入影像解析度很高，則使用較高的中輟速率可能特別有用。

- 請藉由將 `rf_pretrainedModelFilename` 從 `ResNet_18.model` 變更為 `ResNet_34.model` 或 `ResNet_50.model`，嘗試使用更深入的 DNN。 Resnet 50 模型不僅更深入，而且其倒數第二層的輸出大小為 2048 浮點數 (相對於ResNet 18 和 ResNet 34 模型的 512 浮點數)。 訓練 SVM 分類器時，這個增加的維度可能特別有用。

## <a name="part-3---custom-dataset"></a>第 3 部分 - 自訂資料集

在第 1 和 2 部分中，我們已使用提供的上衣紋理影像來訓練和評估影像分類模型。 現在，我們會示範如何改為使用自訂的使用者提供資料集。 

### <a name="using-a-custom-dataset"></a>使用自訂資料集

首先，讓我們看一下服裝紋理資料的資料夾結構。 請注意不同屬性的所有影像如何位在 *DATA_DIR/images/fashionTexture/* 的個別子資料中；*dotted*、*leopard* 和 *striped*。 另請注意影像資料夾名稱如何同時出現在 `PARAMETERS.py` 檔案中：
```python
datasetName = "fashionTexture"
```

使用自訂資料集就像複製此資料夾結構一樣簡單；在其中，所有影像都會根據其屬性位在子子資料夾中，並將這些子資料夾複製到使用者指定的新目錄 *DATA_DIR/images/newDataSetName/* 中。 唯一需要的程式碼變更是將 `datasetName` 變數設定為 *newDataSetName*。 接著可以依序執行指令碼 1-5，而所有中繼檔案會寫入到 *DATA_DIR/proc/newDataSetName/* 中。 不需要進行其他程式碼變更。

請務必確認每個影像只能指派給一個屬性。 例如，具有 'animal' 和 'leopard' 的屬性是錯誤的，因為 'leopard' 影像也屬於 'animal'。 此外，最好移除模稜兩可而難以標註的影像。



### <a name="image-scraping-and-annotation"></a>影像擷取和註解

收集夠大量的標註影像來進行訓練和測試可能相當困難。 解決這個問題的方法之一是從網際網路擷取影像。

> [!IMPORTANT] 
> 針對您使用的任何影像，請確定您沒有違反影像的著作權與授權。 

<!--
For example, see below the Bing Image Search results for the query *t-shirt striped*. As expected, most images indeed are striped t-shirts. The few incorrect or ambiguous images (such as column 1, row 1; or column 3, row 2) can be identified and removed easily:
<p align="center">
<img src="media/scenario-image-classification-using-cntk/bing_search_striped.jpg" alt="alt text" width="600"/>
</p>
-->

若要產生大量的各種資料集，應該使用多個查詢。 例如，使用服裝項目 {blouse、hoodie、pullover、sweater、shirt、t-shirt、vest} 和 {striped、dotted、leopard} 屬性的所有組合，可以自動合成 7\*3 = 21 個查詢。 下載每個查詢的前 50 個影像則會導致最多 21 * 50 = 1050 個影像。

<!--
Rather than manually downloading images from Bing Image Search, it is much easier to instead use the [Cognitive Services Bing Image Search API](https://www.microsoft.com/cognitive-services/bing-image-search-api) which returns a set of image URLs given a query string.
-->

一些下載的影像是確切的重複項目或接近重複項目 (例如，只是影像解析度或 jpg 成品不同)。 應移除這些重複項目，讓訓練集和測試分割不會包含相同的影像。 移除重複的影像可以使用以雜湊為基礎的方法來達成，它分成兩個步驟：(i) 首先，針對所有影像計算雜湊字串；(i) 在影像的第二個階段中，只保留尚未看到雜湊字串的那些影像。 所有其他影像都會被捨棄。 我們發現 Python 程式庫 `imagehash` 和此[部落格](http://www.hackerfactor.com/blog/index.php?/archives/529-Kind-of-Like-That.html)中說明的 `dhash` 方法，只要將參數 `hash_size` 設定為 16 就能正確執行。 錯誤地移除一些非重複影像也沒有關係，只要移除大部分的實際重複項目即可。





## <a name="conclusion"></a>結論

此範例的一些重點精華如下：
- 可訓練、評估及部署影像分類模型的程式碼。
- 提供示範影像，但可輕鬆調整 (一行變更) 為使用自己的影像資料集。
- 實作最新穎的專家功能，根據傳輸學習來訓練高精確度模型。
- 使用 Azure Machine Learning Workbench 和 Jupyter Notebook 來開發互動式模型。


## <a name="references"></a>參考

[1] Alex Krizhevsky、Ilya Sutskever 和 Geoffrey E.Hinton：[_ImageNet Classification with Deep Convolutional Neural Networks_](https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf). NIPS 2012。  
[2] Kaiming He、Xiangyu Zhang、Shaoqing Ren 和 Jian Sunn：[_Deep Residual Learning for Image Recognition_](https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/He_Deep_Residual_Learning_CVPR_2016_paper.pdf). CVPR 2016。
