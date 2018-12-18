---
title: Azure AI 資源庫實驗 | Microsoft Docs
description: 探索與分享 Azure AI 資源庫中的實驗。
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: f4248922-c961-4d3a-9e1b-aec743210166
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/26/2017
ms.openlocfilehash: d57e780fbf5187f2481d40bde28b113181c15b4a
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34834653"
---
# <a name="discover-experiments-in-azure-ai-gallery"></a>探索 Azure AI 資源庫中的實驗
[!INCLUDE [machine-learning-gallery-item-selector](../../../includes/machine-learning-gallery-item-selector.md)]

## <a name="experiments-for-machine-learning-studio"></a>Machine Learning studio 的實驗
Gallery 包含各種在 [Azure Machine Learning Studio](https://studio.azureml.net) 中開發的[實驗](https://gallery.cortanaintelligence.com/experiments)。 小至示範特定機器學習服務技術的快速概念證明實驗，大至用於解決複雜機器學習服務問題的完整開發解決方案，都在這些實驗的涵蓋範圍內。

> [!NOTE]
> ***實驗***是 Machine Learning Studio 中的畫布，可用來建構預測分析模型。 您可以將資料與各種分析模組連接，以建立模型。 您可以嘗試不同的想法、執行試用，最後部署您的模型做為 Azure 中的 Web 服務。 如需建立基本實驗的範例，請參閱[機器學習服務教學課程：在 Azure Machine Learning Studio 中建立您的第一個資料科學實驗](create-experiment.md)。 如需建立預測性分析方案的更完整逐步解說，請參閱[逐步解說：在 Azure Machine Learning 中為信用風險評估開發預測分析解決方案](walkthrough-develop-predictive-solution.md)。
>
>

## <a name="discover"></a>探索
若要瀏覽 [Gallery 中](http://gallery.cortanaintelligence.com)的實驗，請在 Gallery 首頁頂端選取 [實驗]。

**[實驗](https://gallery.cortanaintelligence.com/experiments)** 頁面會顯示最近新增與熱門實驗清單。 若要查看所有實驗，請選取 [查看全部] 按鈕。 若要搜尋特定的實驗，請選取 [查看全部]，然後選取篩選準則。 您也可以在資源庫頁面頂端的 [搜尋] 方塊中輸入搜尋字詞。

您可以實驗詳細資料頁面上取得實驗的詳細資訊。 若要開啟實驗詳細資料頁面，請選取該實驗。 在實驗詳細資料頁面上的 [註解] 區段中，您可以留言、提供意見反應，或詢問有關實驗的問題。 您甚至可以在 Twitter 或 LinkedIn 上與朋友或同事分享實驗。 您也可以用電子郵件寄出實驗詳細資料頁面連結，以邀請其他使用者檢視該頁面。

![與朋友分享此項目](./media/gallery-how-to-use-contribute-publish/share-links.png)

![新增您自己的留言](./media/gallery-how-to-use-contribute-publish/comments.png)

## <a name="download"></a>下載
您可以將 Gallery 中任何實驗的複本下載到您的 Machine Learning Studio 工作區。 然後，您可以修改複本以建立您自己的解決方案。

Azure AI 資源庫提供兩種匯入實驗複本的方式：

* **從 Gallery**。 如果您在 Gallery 中發現所需實驗，您可以下載其複本，並在 Machine Learning Studio 工作區中開啟。
* **從 Machine Learning Studio 內**。 在 Machine Learning Studio 中，您可以將 Gallery 中的任何實驗當做範本來建立新的實驗。

### <a name="from-the-gallery"></a>從 Gallery

1. 在 Gallery 中，開啟實驗詳細資料頁面。
2. 選取 [在 Studio 中開啟]。

    ![從 Gallery 開啟實驗](./media/gallery-experiments/open-experiment-from-gallery.png)

當您選取 [在 Studio 中開啟]時，實驗會在 Machine Learning Studio 工作區中開啟。 (如果您尚未登入 Machine Learning Studio，會提示您先使用您的 Microsoft 帳戶登入。)

### <a name="from-within-machine-learning-studio"></a>從 Machine Learning Studio 內

1. 在 Machine Learning Studio 中，選取 [新增]。
2. 選取 [實驗]。 您可以從 Gallery 實驗清單中選擇，或使用 [搜尋] 方塊尋找特定實驗。
3. 將滑鼠指向實驗，然後選取 [在 Studio 中開啟]。 (若要檢視實驗的相關資訊，請選取 [View in Gallery (在 Gallery 中檢視)]。 這會將您帶往 Gallery 中的實驗詳細資料頁面。)

    ![在 Machine Learning Studio 中開啟 Gallery 實驗](./media/gallery-experiments/open-experiment-from-studio.png)

您可以自訂、反覆執行和部署下載的實驗，就如同您在 Machine Learning Studio 中建立的任何其他實驗。

![在 Studio 中開啟的實驗](./media/gallery-experiments/experiment-open-in-studio.png)

## <a name="contribute"></a>參與
當您登入 Gallery 時，您就成為 Gallery 社群的成員。 身為社群的成員，您可以貢獻您自己的實驗，讓其他使用者可以從您您發現的解決方案中獲益。

### <a name="publish-your-experiment-to-the-gallery"></a>將實驗發佈至 Gallery

1. 使用 Microsoft 帳戶登入 Machine Learning Studio。
2. 建立您的實驗，然後執行。
3. 當您準備好在 Gallery 中發佈實驗時，在實驗畫布下方的動作清單中，選取 [Publish to Gallery (發佈到 Gallery)]。

    ![選取 [Publish to Gallery (發佈到 Gallery)]](./media/gallery-experiments/publish-experiment-to-gallery.png)
4. 在 [Experiment Description (實驗描述)] 頁面上，輸入標題和標記。 讓標題和標記具有描述性。 反白顯示您所使用的技術或您正在解決的實際問題。 舉例來說，「二元分類法：Twitter 情感分析」即為具描述性的實驗標題。

    ![輸入標題和標記進行發佈](./media/gallery-experiments/experiment-description.png)
5. 在 [摘要] 方塊中，輸入您的實驗摘要。 簡短說明此實驗要解決的問題，以及您的處理方式。
6. 在 [DETAILED DESCRIPTION (詳細描述)] 方塊中，描述您在實驗的每個部分中採取的步驟。 以下為一些實用的主題：
   * 實驗圖表的螢幕擷取畫面
   * 資料來源和說明
   * 資料處理
   * 特徵設計
   * 模型說明
   * 成果和模型效能評估

   您可以使用 Markdown 來設定描述的格式。 若要了解您在實驗描述頁面上的輸入在實驗發佈時會是什麼樣子，請選取 [預覽]。

   ![選取 [預覽] 了解文字如何呈現](./media/gallery-experiments/preview-markdown-text.png)

   > [!TIP]
   > 提供用於編輯和預覽 Markdown 的文字方塊相當小。 因此建議您在 Markdown 編輯器中撰寫實驗文件，複製文件，然後將完成的文件貼到 Gallery 的文字方塊中。 發佈實驗後，您可以採用任何使用 Markdown 的標準 Web 型工具進行編輯與預覽，以進行任何更正。

7. 在 [影像選取] 頁面上，為您的實驗選取縮圖影像。 縮圖影像會顯示在實驗詳細資料頁面與實驗磚頂端。 其他使用者在瀏覽 Gallery 時會看到縮圖影像。 您可以上傳電腦中的影像，或選取 Gallery 中的內建影像。
    </br>
    ![上傳或選取 Gallery 的影像](./media/gallery-experiments/select-gallery-image.png)
8. 在 [設定] 頁面的 [可視性] 下，選擇要公開發佈您的內容 ([公開])，或只讓擁有該頁面連結的人存取 ([未公開]))。

    ![選擇要公開發佈還是以未列出狀態發佈](./media/gallery-experiments/choose-public-or-unlisted.png)

    <!-- -->

   > [!TIP]
   > 如果您想要先確定文件看起來是正確的，再公開發佈，您可以先將實驗發佈為 [未公開]。 之後，您可以在實驗詳細資料頁面上將可視性設定變更為 [公開]。
   >
   >
9. 若要將實驗發佈到 Gallery，請選取 [確定] 勾選記號。

    ![選取 [確定] 勾選記號發佈實驗](./media/gallery-experiments/ok-checkmark.png)

如需如何發佈高品質 Gallery 實驗的秘訣，請參閱[記錄與發佈實驗的秘訣](#tips-for-documenting-and-publishing-your-experiment)。

說明到此就算結束了。

您現在可以在 Gallery 中檢視您的實驗，並與其他人分享連結。 如果您使用 [公開] 可視性設定發佈您的實驗，則實驗會出現在 Gallery 的瀏覽與搜尋結果中。 在您登入 Gallery 後，您可以隨時在實驗詳細資料頁面上編輯您的實驗文件。

若要檢視您的貢獻清單，請在任何 Gallery 頁面的右上角選取您的影像。 接著，選取您的名稱以開啟您的帳戶頁面。

![選取您的帳戶名稱](./media/gallery-experiments/click-account-name.png)

### <a name="update-your-experiment"></a>更新您的實驗
若有需要，您可以在您發佈到 Gallery 的實驗中變更工作流程 (模組、參數等等)。 在 Machine Learning Studio 中，對實驗進行任何您想要的變更，然後重新發佈。 系統會以您所做的變更來更新發佈的實驗。

您可以直接在 Gallery 中變更實驗的以下任一資訊：

* 實驗名稱
* 摘要或描述
* 標記
* 映像
* 可視性設定 ([公開] 或 [未公開])

您也可以將實驗從 Gallery 中刪除。

您可以從 Gallery 的實驗詳細資料頁面或設定檔頁面中，進行這些變更或刪除實驗。


#### <a name="from-the-experiment-details-page"></a>從實驗詳細資料頁面
在實驗詳細資料頁面上若要變更實驗的詳細資料，請選取 [編輯]。

![選取 [編輯] 以編輯實驗](./media/gallery-experiments/edit-button.png)

詳細資料頁面進入編輯模式。 若要進行變更，請選取實驗名稱、摘要或標記旁邊的 [編輯]。 當您完成變更時，請選取 [完成]。

![選取 [編輯] 以編輯詳細資料，當完成時請選取 [完成]](./media/gallery-experiments/edit-details-page.png)

若要變更實驗的可視性設定 ([公開] 或 [未公開])，或要將實驗從 Gallery 中刪除，請選取 [設定] 圖示。

![選取 [設定] 變更可視性或刪除實驗](./media/gallery-experiments/settings-button.png)

#### <a name="from-your-profile-page"></a>從設定檔頁面
在設定檔頁面上，選取實驗的向下箭頭，然後選取 [編輯]。 這會帶您前往實驗的詳細資料頁面，並進入編輯模式。 當您完成變更時，請選取 [完成]。

若要將實驗從 Gallery 中刪除，請選取 [刪除]。

![選取 [編輯] 或 [刪除]](./media/gallery-experiments/edit-delete-buttons.png)

### <a name="tips-for-documenting-and-publishing-your-experiment"></a>記錄與發佈實驗的秘訣
* 您可以假設讀者先前已有資料科學方面的經驗，但簡化措辭仍然有所助益。 盡可能詳細說明其內容。
* Cortana Intelligence Suite 是較新的服務， 並非所有讀者都熟悉。 因此，提供足夠的資訊和逐步解說可協助讀者了解您的實驗過程。
* 視覺效果可幫助讀者正確理解和使用您的實驗。 視覺效果包括實驗圖和資料的螢幕擷取畫面。 如需如何在實驗文件中放入影像的詳細資訊，請參閱[發佈方針與範例集合](https://gallery.cortanaintelligence.com/Collection/Publishing-Guidelines-and-Examples-1)。
* 如果您在實驗中納入資料集 (也就是您未透過 [匯入資料] 模組匯入資料集)，則資料集為您實驗的一部分，並已在 Gallery 中發佈。 請確定您發佈的資料集具有授權條款可允許任何人分享與下載。 資源庫發表內容在 Azure [使用規定](https://azure.microsoft.com/support/legal/website-terms-of-use/)涵蓋範圍內。

## <a name="frequently-asked-questions"></a>常見問題集
**提交或編輯實驗的影像有何需求？**

您連同實驗一起提交的影像用於為您的貢獻建立實驗磚。 建議使用小於 500 KB 的影像，長寬比 3:2，解析度 960 &#215; 640。

**我在實驗中使用的資料集會發生什麼狀況？資料集是否也會在 Gallery 中發佈？**

如果您的資料集是實驗的一部分，且未透過 [匯入資料] 模組匯入，則資料集會在 Gallery 中發佈成為您實驗的一部分。 請確定隨同您實驗發佈的資料集具有適當的授權條款。 該授權條款應該允許任何人分享與下載資料。

**我有一個實驗是使用 [匯入資料] 模組從 Azure HDInsight 或 SQL Server 提取資料。它使用我的認證擷取資料。我可以發佈這種實驗嗎？我該如何確定我的認證將不會與他人分享？**

目前，您無法在 Gallery 中發佈使用認證的實驗。

**如何輸入多個標籤？**

輸入標記之後若要輸入另一個標記，請按 Tab 鍵。

**[移至資源庫](http://gallery.cortanaintelligence.com)**

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]
