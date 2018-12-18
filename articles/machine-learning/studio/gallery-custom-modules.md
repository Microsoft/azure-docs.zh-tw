---
title: Azure AI 資源庫自訂模組 | Microsoft Docs
description: 探索 Azure AI 資源庫中的自訂機器學習服務模組。
services: machine-learning
documentationcenter: ''
author: YasinMSFT
ms.author: yahajiza
manager: hjerez
editor: cgronlun
ms.assetid: 16037a84-dad0-4a8c-9874-a1d3bd551cf0
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/14/2017
ms.openlocfilehash: c53bab2e838425dfdd124e64c3d7d3114fa30429
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34834425"
---
# <a name="discover-custom-machine-learning-modules-in-azure-ai-gallery"></a>探索 Azure AI 資源庫中的自訂機器學習服務模組
[!INCLUDE [machine-learning-gallery-item-selector](../../../includes/machine-learning-gallery-item-selector.md)]

## <a name="custom-modules-for-machine-learning-studio"></a>Machine Learning studio 的自訂模組
Azure AI 資源庫會提供數個[自訂模組](https://gallery.cortanaintelligence.com/customModules)，可擴充 Azure Machine Learning Studio 的功能。 您可以匯入這些模組用於您的實驗，以便開發更進階的預測性分析解決方案。

目前，Gallery 提供*時間序列分析**關聯規則**叢集演算法*(k-means 除外) 和*視覺效果*，以及其他的主力公用程式模組。


## <a name="discover"></a>探索
若要瀏覽 [Gallery 中的](http://gallery.cortanaintelligence.com)自訂模組，請在 [更多] 下方選取 [自訂模組]。

![選取 Gallery 首頁上的自訂模組](./media/gallery-custom-modules/select-custom-modules-in-gallery.png)

**[自訂模組](https://gallery.cortanaintelligence.com/customModules)** 頁面會顯示最近新增及熱門的模組清單。 若要檢視所有的自訂模組，請選取 [查看全部] 按鈕。 若要搜尋特定的自訂模組，請選取 [查看全部]，然後選取篩選準則。 您也可以在資源庫頁面頂端的 [搜尋] 方塊中輸入搜尋字詞。

![選取 [查看全部] 可瀏覽所有的自訂模組](./media/gallery-custom-modules/click-see-all-for-all-custom-modules.png)

### <a name="understand"></a>了解

若要了解已發佈的自訂模組如何運作，請選取自訂模組開啟模組詳細資料頁面。 詳細資料頁面可提供一致且資訊豐富的學習經驗。 例如，詳細資料頁面會反白顯示模組目的，並列出預期的輸入、輸出和參數。 詳細資料頁面也提供基礎原始程式碼的連結，讓您可以檢查和自訂程式碼。

### <a name="comment-and-share"></a>註解和共用
在自訂模組詳細資料頁面上的 [註解] 區段中，您可以留言、提供意見反應，或詢問有關模組的問題。 您甚至可以在 Twitter 或 LinkedIn 上與朋友或同事分享模組。 您也可以用電子郵件寄出模組詳細資料頁面連結，以邀請其他使用者檢視該頁面。

![與朋友分享此項目](./media/gallery-how-to-use-contribute-publish/share-links.png)

![新增您自己的留言](./media/gallery-how-to-use-contribute-publish/comments.png)

## <a name="import"></a>Import
您可以從 Gallery 將任何自訂模組匯入到您自己的實驗中。

Azure AI 資源庫提供兩種匯入模組複本的方式：

* **從 Gallery**。 當您從 Gallery 匯入自訂模組，您也會獲得範例實驗，以舉例說明如何使用模組。
* **從 Machine Learning Studio 內**。 您可以在使用 Machine Learning Studio 時，可以匯入任何自訂模組 (在此狀況下，您無法取得範例實驗)。

### <a name="from-the-gallery"></a>從 Gallery

1. 在 Gallery 中開啟模組詳細資料頁面。 
2. 選取 [在 Studio 中開啟]。
   
    ![從 Gallery 開啟自訂模組](./media/gallery-custom-modules/open-custom-module-from-gallery.png)
   
每個自訂模組都包含示範如何使用模組的範例實驗。 當您選取 [在 Studio 中開啟]時，範例實驗會在 Machine Learning Studio 工作區中開啟。 (如果您尚未登入 Studio，會提示您先使用您的 Microsoft 帳戶登入。)

除了範例實驗外，自訂模組也會複製到您的工作區。 此外，自訂模組會連同所有內建或自訂 Machine Learning Studio 模組一起放在模組選擇區中。 現在，您可以將自訂模組用於您自己的實驗中，就如同工作區中任何其他的模組。

### <a name="from-within-machine-learning-studio"></a>從 Machine Learning Studio 內

1. 在 Machine Learning Studio 中，選取 [新增]。
2. 選取 [模組]。 您可以從 Gallery 模組清單中選擇，或使用 [搜尋] 方塊尋找特定模組。
3. 將滑鼠指向模組，然後選取 [匯入模組]。 (若要取得模組的相關資訊，請選取 [View in Gallery (在 Gallery 中檢視)]。 這會將您帶往 Gallery 中的模組詳細資料頁面。)
   
    ![將自訂模組匯入 Machine Learning Studio](./media/gallery-custom-modules/add-custom-module-in-studio.png)

自訂模組會複製到您的工作區，並連同內建或自訂的 Machine Learning 模組一起放在模組選擇區中。 現在，您可以將自訂模組用於您自己的實驗中，就如同工作區中任何其他的模組。

## <a name="use"></a>使用

無論您選擇哪一種方式匯入自訂模組，當您匯入模組時，模組會放在 Machine Learning Studio 的模組選擇區中。 從模組選擇區中，您可以將自訂模組用於工作區中的任何實驗中，就如同任何其他模組一般。

若要使用匯入的模組：

1. 建立實驗，或開啟現有的實驗。
2. 若要展開工作區中的自訂模組清單，在模組選擇區中 [自訂]。 模組選擇區位在實驗畫布左側。
   
    ![Studio 調色盤中的自訂模組清單](./media/gallery-custom-modules/custom-module-in-studio-palette.png)
3. 選取您匯入的模組，並將它拖曳至您的實驗。


**[移至資源庫](http://gallery.cortanaintelligence.com)**

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

