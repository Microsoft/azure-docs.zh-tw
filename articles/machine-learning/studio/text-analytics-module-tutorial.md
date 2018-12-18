---
title: 在 Azure Machine Learning Studio 中建立文字分析模型 | Microsoft Docs
description: 如何在 Azure Machine Learning Studio 使用文字前置處理、N-Gram 或特徵雜湊來建立文字分析模型
services: machine-learning
documentationcenter: ''
author: rastala
ms.author: roastala
manager: cgronlun
editor: ''
ms.assetid: 08cd6723-3ae6-4e99-a924-e650942e461b
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2018
ms.openlocfilehash: a9d4ee9015b5781301dcc9c0fe4f8944dbf244e8
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34835877"
---
# <a name="create-text-analytics-models-in-azure-machine-learning-studio"></a>在 Azure Machine Learning Studio 中建立文字分析模型
您可以使用 Azure Machine Learning 來建置和實行文字分析模型。 這些模型可協助您解決問題，例如，文件分類或情緒分析問題。

在文字分析實驗中，您通常需要︰

1. 清理和前置處理文字資料集
2. 從已前置處理的文字擷取數值特徵向量
3. 定型分類或迴歸模型
4. 評分和驗證模型
5. 模型部署到生產環境

在此教學課程中，當我們使用「Amazon 書籍評論」資料集逐步解說情緒分析模型時，您會學到這些步驟 (請參閱研究報告 “Biographies, Bollywood, Boom-boxes and Blenders: Domain Adaptation for Sentiment Classification”，作者：Association of Computational Linguistics (ACL) 的 John Blitzer、Mark Dredze 和 Fernando Pereira，2007 年)。此資料集是由評論分數 (1-2 或 4-5) 和自由格式文字所組成。 目標是要預測評論分數︰低 (1-2) 或高 (4-5)。

您可以在 Azure AI 資源庫找到本教學課程中涵蓋的實驗︰

[預測書籍評論](https://gallery.cortanaintelligence.com/Experiment/Predict-Book-Reviews-1)

[預測書籍評論 - 預測性實驗](https://gallery.cortanaintelligence.com/Experiment/Predict-Book-Reviews-Predictive-Experiment-1)

## <a name="step-1-clean-and-preprocess-text-dataset"></a>步驟 1：清理和前置處理文字資料集
一開始我們先將評論分數分為低與高兩類，以將問題公式化為雙類別分類。 我們使用[編輯中繼資料](https://msdn.microsoft.com/library/azure/dn905986.aspx)和[群組類別值](https://msdn.microsoft.com/library/azure/dn906014.aspx)模組。

![建立標籤](./media/text-analytics-module-tutorial/create-label.png)

然後，我們使用 [前置處理文字](https://msdn.microsoft.com/library/azure/mt762915.aspx) 模組清除文字。 清除可減少資料集雜訊、協助您找出最重要的特徵，並改善最終模型的精確度。 我們會移除停用字詞 (例如 "the" 或 "a" 等常見單字)、數字、特殊字元、重複字元、電子郵件地址和 URL。 我們也將文字轉換成小寫、將單字按屈折變化形式歸類，並偵測句子界限，然後在預先處理的文字中以 "| | |" 符號表示這些界限。

![前置處理文字](./media/text-analytics-module-tutorial/preprocess-text.png)

如果想使用自訂的停用字詞清單該怎麼做？ 您可以將它傳入做為選擇性的輸入。 您也可以使用自訂的 C# 語法規則運算式來取代子字串，並按詞性 (名詞、動詞或形容詞) 移除單字。

在前置處理完成之後，我們會將資料分成定型和測試集。

## <a name="step-2-extract-numeric-feature-vectors-from-pre-processed-text"></a>步驟 2：從已前置處理的文字擷取數值特徵向量
若要建置文字資料的模型，您通常需要將自由格式的文字轉換成數值特徵向量。 在此範例中，我們使用 [從文字擷取 N-Gram 特徵](https://msdn.microsoft.com/library/azure/mt762916.aspx) 模組，將文字資料轉換為這種格式。 此模組會採用以空格分隔單字的資料行，並計算出現在您資料集中的單字字典或單字的 N-Gram。 然後，它會計算每個單字或 N-Gram 出現在每筆記錄的次數，並從這些計數建立特徵向量。 在本教學課程中，我們將 N-Gram 大小設為 2，因此我們的特徵向量包含一個單字和兩個後續單字的組合。

![擷取 N-Gram](./media/text-analytics-module-tutorial/extract-ngrams.png)

我們會套用 TF*IDF (Term Frequency Inverse Document Frequency) 加權至 N-Gram 計數。 這個方法會增加經常出現在單一記錄、卻很少在整個資料集出現的單字的權數。 其他選項包括二進位、TF 及圖形加權。

這類文字特徵通常具有高維度。 比方說，如果您的語言資料庫有 100,000 個唯一的單字，特徵空間會有 100,000 個維度，或使用更多的 N-Gram。 「擷取 N-Gram 特徵」模組提供一組減少維度的選項。 您可以選擇排除過短或過長的單字，或是太常見或太頻繁而有重要預測值的單字。 在本教學課程中，我們會排除出現在少於 5 筆記錄或超過 80% 的記錄的 N-Gram。

此外，使用特徵選取可以只選取與您的預測目標最相關的特徵。 我們使用 Chi-Squared 特徵選取來選取 1000 個特徵。 您可以按一下「擷取 N-Gram」模組右側的輸出，即可檢視所選單字或 N-Gram 的詞彙。

另一個方法是使用「擷取 N-Gram 特徵」，您就可以使用「特徵雜湊」模組。 但請注意， [特徵雜湊](https://msdn.microsoft.com/library/azure/dn906018.aspx) 沒有內建的特徵選取功能或 TF*IDF 加權。

## <a name="step-3-train-classification-or-regression-model"></a>步驟 3：定型分類或迴歸模型
現在文字已轉換為數值特徵資料行。 資料集仍包含上一階段中的字串資料行，因此我們使用「選取資料集中的資料行」來排除它們。

接著使用 [二元羅吉斯迴歸](https://msdn.microsoft.com/library/azure/dn905994.aspx) 預測我們的目標︰高或低的評論分數。 此時，文字分析問題已轉換成一般分類問題。 您可以使用 Azure Machine Learning 中可用的工具來改善模型。 例如，您可以試驗不同的分類器以了解它們所提供結果的精確度，或使用超參數調整改善精確度。

![定型和評分](./media/text-analytics-module-tutorial/scoring-text.png)

## <a name="step-4-score-and-validate-the-model"></a>步驟 4：評分和驗證模型
如何驗證定型的模型？ 我們會對照測試資料集來評分，並評估精確度。 不過，此模型已從了解定型資料集學到 N-Gram 和其加權的詞彙。 因此，在從測試資料擷取特徵時，我們應該使用該詞彙和這些加權，而不是重新建立詞彙。 因此，我們在實驗評分分支加入「擷取 N-Gram 特徵」模組、從定型分支連接輸出詞彙，並將詞彙模式設定為唯讀。 我們也透過將最小值設為 1 個執行個體、最大值設為 100% 來停用依頻率篩選 N-Gram，並關閉特徵選取。

在測試資料中的文字資料行轉換成數值特徵資料行之後，我們會排除先前階段 (例如在定型分支中) 中的字串資料行。 接著使用「評分模型」模組進行預測，並使用「評估模型」模組來評估精確度。

## <a name="step-5-deploy-the-model-to-production"></a>步驟 5：將模型部署到生產環境
模型已幾乎可立即部署到生產環境。 部署為 Web 服務時，它會採用自由格式的文字字串做為輸入，並傳回「高」或「低」的預測。 它會使用學習到的 N-Gram 詞彙將文字轉換成特徵，並使用定型的羅吉斯迴歸模型，從這些特徵進行預測。 

為設定預測性實驗，我們先儲存 N-Gram 詞彙做為資料集，並使用實驗的定型分支中的定型羅吉斯迴歸模型。 接著我們使用「另存新檔」儲存實驗，為預測性實驗建立實驗圖形。 我們從實驗中移除「分割資料」模組和定型分支。 我們再將先前儲存的 N-Gram 詞彙和模型分別連接到「擷取 N-Gram 特徵」和「評分模型」模組。 我們也會移除「評估模型」模組。

我們將「選取資料集中的資料行」模組插入「前置處理文字」模組之前以移除標籤資料行，並取消選取「評分模組」中的「 將評分資料行附加到資料集」選項。 這樣一來，Web 服務就不會要求它正嘗試預測的標籤，也不會對回應中的輸入特徵做回應。

![預測性實驗](./media/text-analytics-module-tutorial/predictive-text.png)

現在我們有一個實驗可以發行為 Web 服務，並使用要求-回應或批次執行 API 進行呼叫。

## <a name="next-steps"></a>後續步驟
從 [MSDN 文件](https://msdn.microsoft.com/library/azure/dn905886.aspx)深入了解文字分析模組。

