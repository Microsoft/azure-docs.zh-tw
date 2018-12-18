---
title: 在 Machine Learning 中評估模型效能 | Microsoft Docs
description: 說明如何在 Azure Machine Learning 中評估模型效能
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: 5dc5348a-4488-4536-99eb-ff105be9b160
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2017
ms.openlocfilehash: bb49fd2fe7f72e211fbbda7cffdd2308c2c36fba
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34834228"
---
# <a name="how-to-evaluate-model-performance-in-azure-machine-learning"></a>如何在 Azure Machine Learning 中評估模型效能
本文示範如何在 Azure Machine Learning Studio 中評估模型的效能，並提供這項工作中可用度量的簡短說明。 提供三種常見的受監督的學習案例： 

* 迴歸
* 二進位分類 
* 多元分類

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

評估模型的效能是資料科學流程中的核心階段之一。 它會指出定型模型如何成功地為資料集評分 (預測)。 

Azure Machine Learning 支援透過兩個主要的機器學習服務模組來評估模型：[評估模型][evaluate-model]和[交叉驗證模型][cross-validate-model]。 這些模組可讓您根據 Machine Learning 和統計資料中常用的一些度量，查看您模型的運作方式。

## <a name="evaluation-vs-cross-validation"></a>評估與交叉驗證
評估與交叉驗證是測量模型效能的標準方式。 它們都會產生您可以對照其他模型的度量檢查或比較的評估度量。

[評估模型][evaluate-model]需要一個計分資料集作為輸入 (如果您想要比較 2 個不同模型的效能，則需要 2 個計分資料集)。 這表示，您需要先使用[訓練模型][train-model]模組來訓練模型，並使用[計分模型][score-model]模組在一些資料集上進行預測，然後才能評估結果。 評估是以評分標籤/機率及真實性標籤為基礎，這些全都由[評分模型][score-model]模組來輸出。

或者，您可以使用交叉驗證，對不同的輸入資料子集自動執行一「訓練-評分-評估」作業 (10 次交疊)。 輸入資料會分割成 10 個部分，其中 1 個部分保留供測試之用，其他 9 個部分則供訓練之用。 此程序會重複 10 次，然後取得評估度量的平均值。 這有助於判斷將模型一般化為新資料集的程度。 [交叉驗證模型][cross-validate-model]模組接受非定型模型和一些標註的資料集，除了輸出平均結果，還會輸出各有 10 次交疊的評估結果。

在以下幾節中，我們將同時使用[評估模型][evaluate-model]和[交叉驗證模型][cross-validate-model]模組，建置簡單的迴歸和分類模型，並評估其效能。

## <a name="evaluating-a-regression-model"></a>評估迴歸模型
假設我們想要使用某些功能 (例如尺寸、馬力、引擎規格等等) 預測汽車的價格。 這是典型的迴歸問題，其中的目標變數 (*價格*) 是連續的數值。 假設已知特定汽車的功能值，我們可以放入簡單的線性迴歸模型，就可以預測該汽車的價格。 這個迴歸模型可用來為我們所訓練的相同資料集評分。 一旦我們擁有所有汽車的預測價格之後，就可以藉由查看預測與實際價格的平均誤差，來評估模型的效能。 為了說明這一點，我們使用 Azure Machine Learning Studio 的 [ *已儲存的資料集* ] 區段中提供的 [ **汽車價格資料 (原始) 資料集** ]。

### <a name="creating-the-experiment"></a>建立實驗
將下列模組加入至您在 Azure Machine Learning Studio 中的工作區：

* 汽車價格資料 (原始)
* [線性迴歸][linear-regression]
* [訓練模型][train-model]
* [計分模型][score-model]
* [評估模型][evaluate-model]

如下圖 1 所示連接連接埠，並將[訓練模型][train-model]模組的 [標籤] 資料行設定為 [價格]。

![評估迴歸模型](./media/evaluate-model-performance/1.png)

圖 1. 評估迴歸模型。

### <a name="inspecting-the-evaluation-results"></a>檢查評估結果
執行實驗之後，您可以按一下[評估模型][evaluate-model]模組的輸出連接埠，然後選取 [視覺化] 以查看評估結果。 適用於迴歸模型的評估度量包括：「平均絕對誤差」、「平均根絕對誤差」、「相對絕對誤差」、「相對平方誤差」和「決定係數」。

「誤差」一詞代表預測的值和真正的值之間的差異。 系統通常會計算這項差異的絕對值或平方來擷取所有案例中的總誤差大小，因為在某些情況下，預測的值和真正的值之間的差異可能是負數。 誤差度量會根據預測與真正的值之間的平均偏差，測量迴歸模型的預測性效能。 誤差值越低，表示進行預測的模型越精確。 整體誤差度量為 0 時，表示模型完全符合資料。

決定係數亦稱為「R 平方值」，也是測量模型對於資料的適用程度的一種標準方式。 它可以解譯為模型所說明的變化的比例。 在此情況下，比例越高越好，其中 1 表示完全符合。

![線性迴歸評估度量](./media/evaluate-model-performance/2.png)

圖 2. 線性迴歸評估度量。

### <a name="using-cross-validation"></a>使用交叉驗證
如先前所述，您可以使用[交叉驗證模型][cross-validate-model]模組，自動執行反覆的訓練、計分和評估。 在這種情況下，您只需要一個資料集、一個非定型模型，以及一個[交叉驗證模型][cross-validate-model]模組 (請參閱下圖)。 請注意，您必須在[交叉驗證模型][cross-validate-model]模組的屬性中，將 [標籤] 資料行設定為 [價格]。

![交叉驗證迴歸模型](./media/evaluate-model-performance/3.png)

圖 3. 交叉驗證迴歸模型。

執行實驗之後，您可以按一下[交叉驗證模型][cross-validate-model]模組右側的輸出連接埠，以查看評估結果。 這將為每個反覆項目 (交疊) 提供度量的詳細檢視，以及每個度量的平均結果 (圖 4)。

![迴歸模型的交叉驗證結果](./media/evaluate-model-performance/4.png)

圖 4. 迴歸模型的交叉驗證結果。

## <a name="evaluating-a-binary-classification-model"></a>評估二進位分類模型
例如，在二元分類的情況下，目標變數只有兩個可能的結果：{0, 1} 或 {false, true}, {negative, positive}。 假設您有一個成人員工的資料集，其中包含一些人口統計和就業變數，而且您必須預測收入層級，也就是包含值 {“<=50K”, “>50K”} 的二元變數。 換句話說，負數類別表示年收入低於或等於 5 萬元的員工，而正數類別則表示其他所有員工。 如同在迴歸情況下，我們會訓練模型、為某些資料評分，並評估結果。 此處的主要差異在於 Azure Machine Learning 計算和輸出的度量選擇。 為說明收入層級的預測情況，我們將使用 [成人](http://archive.ics.uci.edu/ml/datasets/Adult) 資料集來建立 Azure Machine Learning 實驗，並評估二元羅吉斯迴歸模型 (這是一種常用的二元分類器) 的效能。

### <a name="creating-the-experiment"></a>建立實驗
將下列模組加入至您在 Azure Machine Learning Studio 中的工作區：

* 成人收入普查二進位分類資料集
* [二元羅吉斯迴歸][two-class-logistic-regression]
* [定型模型][train-model]
* [計分模型][score-model]
* [評估模型][evaluate-model]

如下圖 5 所示連接連接埠，並將[訓練模型][train-model]模組的 [標籤] 資料行設定為 [收入]。

![評估二進位分類模型](./media/evaluate-model-performance/5.png)

圖 5. 評估二進位分類模型。

### <a name="inspecting-the-evaluation-results"></a>檢查評估結果
執行實驗之後，您可以按一下[評估模型][evaluate-model]模組的輸出連接埠，然後選取 [視覺化] 以查看評估結果 (圖 7)。 可用於二元分類模型的評估度量包括：「正確性」、「精確度」、「回收」、「F1 分數」和 AUC。 此外，這個模組會輸出一個混淆矩陣，其中顯示真肯定、誤否定、誤肯定、真否定，以及 ROC、「精確度/回收和「升力」曲線的數目。

準確性只是正確分類的案例的比例。 它通常是評估分類器時，您看到的第一個度量。 不過，當測試資料處於不平衡狀態 (其中大部分的案例都屬於其中一個類別) 時，或者您對於其中一個類別的效能更有興趣時，準確性不一定會擷取分類器的效率。 在收入層級的分類案例中，假設您要測試的特定資料中，99% 的案例代表年收入少於或等於 5 萬元的人。 預測所有案例的類別 “<=50K”，可能會達到 0.99 準確性。 在此案例中，此分類器整體而言似乎做得不錯，但事實上，它無法正確分類任何高收入的個人 (1%)。

因此，最好先計算可擷取更明確的評估層面的其他度量。 在進入這類度量的細節之前，最好先了解二進位分類評估的混淆矩陣。 定型集中的類別標籤僅能採用 2 個可能的值，我們通常指的是正或負。 分類器正確預測的正和負案例分別稱為真肯定 (TP) 和真否定 (TN)。 同樣地，分類不正確的案例稱為誤肯定 (FP) 和誤否定 (FN)。 混淆矩陣只是一個表格，其中顯示分別落在這 4 個類別的案例數目。 Azure Machine Learning 會自動決定資料集的兩個類別中，那個類別是正類別。 如果類別標籤為布林值或整數，則標示為 'true' 或 '1' 的案例會被指派正類別。 如果標籤為字串 (如以下的收入資料集)，標籤會依字母順序排序，且選擇的第一個層級為負類別，第二個層級則是正類別。

![二元分類混淆矩陣](./media/evaluate-model-performance/6a.png)

圖 6. 二進位分類混淆矩陣。

回到收入分類問題，我們會想要提出幾個評估問題，幫助我們了解所使用的分類器的效能。 很自然的問題是：「在模型預測收入 5 萬元以上 (TP+FP) 的個人中，有多少人的分類正確 (TP)？ 」 透過查看模型的 **精確度** ，也就是正確分類的正數比例，可以回答這個問題：TP/(TP+FP)。 另一個常見問題是：「在收入高於 5 萬元 (TP+FN) 的高收入員工中，分類器正確分類的員工有多少 (TP)？ 」實際上，這是 **回收**，或真肯定比率：分類器的 TP/(TP+FN)。 您可能會注意到在精確度與回收之間有明顯的取捨。 例如，假設是相當對稱資料集，預測大部分是正案例的分類器會有高回收，但是因為許多負案例分類錯誤造成的精確度低會導致大量的誤肯定。 若要查看這兩個度量如何變化的繪圖，您可以按一下 [評估結果輸出] 頁面中的「精確度/回收」曲線 (圖 7 的左側部分)。

![二元分類評估結果](./media/evaluate-model-performance/7.png)

圖 7. 二進位分類評估結果。

常用的另一個相關度量是 **F1 分數**，這會將精確度和回收同時列入考量。 這是這 2 個度量的調和平均數，其計算方式如下：F1 = 2 (精確度 x 回收) / (精確度 + 回收)。 F1 分數是總結單一數字評估很好的方式，但同時查看精確度與回收，以便更加了解分類器運作方式永遠是一個不錯的做法。

此外，您還可以在**受測者操作特徵 (ROC)** 曲線及對應的**曲線下面積 (AUC)** 值中，查看真肯定比率與誤肯定比率的比較。 此曲線越接近左上角，分類器的效能越好 (亦即，將真肯定比率提至最高，同時將誤肯定比率降至最低)。 接近繪圖對角線的曲線是因為分類器想要進行接近隨機猜測的預測所造成。

### <a name="using-cross-validation"></a>使用交叉驗證
如同在迴歸範例中，我們可以執行交叉驗證以自動重複訓練、評分和評估不同的資料子集。 同樣地，我們可以使用[交叉驗證模型][cross-validate-model]模組、非定型的羅吉斯迴歸模型和資料集。 在[交叉驗證模型][cross-validate-model]模組的屬性中，[標籤] 資料行必須設定為 [收入]。 執行實驗並按一下[交叉驗證模型][cross-validate-model]模組右側的輸出連接埠之後，除了每次交疊的平均值和標準差，我們還可以看到每次交疊的二元分類度量值。 

![交叉驗證二元分類模型](./media/evaluate-model-performance/8.png)

圖 8. 交叉驗證二進位分類模型。

![二元分類器的交叉驗證結果](./media/evaluate-model-performance/9.png)

圖 9. 二進位分類器的交叉驗證結果。

## <a name="evaluating-a-multiclass-classification-model"></a>評估多元分類模型
在這個實驗中，我們將使用熱門的[鳶尾](http://archive.ics.uci.edu/ml/datasets/Iris "鳶尾")資料集，其中包含 3 種不同類型 (類別) 鳶尾屬植物的案例。 每個案例有 4 個特性值 (萼片長度/寬度和花瓣長度/寬度)。 在上一個實驗中，我們使用相同的資料集訓練並測試模型。 在此，我們將使用[分割資料][split]模組建立 2 個資料子集，訓練第一個子集，然後計分和評估第二個子集。 [UCI Machine Learning Repository](http://archive.ics.uci.edu/ml/index.html) 上公開提供鳶尾資料集，可透過[匯入資料][import-data]模組來下載。

### <a name="creating-the-experiment"></a>建立實驗
將下列模組加入至您在 Azure Machine Learning Studio 中的工作區：

* [匯入資料][import-data]
* [多元決策樹系][multiclass-decision-forest]
* [分割資料][split]
* [定型模型][train-model]
* [計分模型][score-model]
* [評估模型][evaluate-model]

連接連接埠，如以下圖 10 中所示。

將[訓練模型][train-model]模組的 [標籤] 資料行索引設定為 5。 此資料集沒有標頭資料列，但是我們知道類別標籤位於第五個資料行中。

按一下 [匯入資料][import-data] 模組，並將 [資料來源] 屬性設定為 [透過 HTTP 的 Web URL]，並將 *URL* 設為 http://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data。

在[分割資料][split]模組中，設定要用於訓練的執行個體分數 (例如 0.7)。

![評估多元分類器](./media/evaluate-model-performance/10.png)

圖 10. 評估多元分類器

### <a name="inspecting-the-evaluation-results"></a>檢查評估結果
執行實驗，然後按一下[評估模型][evaluate-model]的輸出連接埠。 在此案例中，評估結果會以混淆矩陣的形式呈現。 此矩陣會針對全部 3 個類別，顯示實際與預測的案例。

![多元分類評估結果](./media/evaluate-model-performance/11.png)

圖 11. 多元分類評估結果。

### <a name="using-cross-validation"></a>使用交叉驗證
如先前所述，您可以使用[交叉驗證模型][cross-validate-model]模組，自動執行反覆的訓練、計分和評估。 您需要一個資料集、一個非定型模型，以及一個[交叉驗證模型][cross-validate-model]模組 (請參閱下圖)。 同樣地，您必須設定[交叉驗證模型][cross-validate-model]模組的 [標籤] 資料行 (在本例中為資料行索引 5)。 執行實驗並按一下[交叉驗證模型][cross-validate-model]右側的輸出連接埠之後，您可以查看每次交疊的度量值，以及平均值和標準差。 在此顯示的度量類似於在二進位分類案例中討論的度量。 但是請注意，在多元分類中，運算真肯定/否定以及誤肯定/否定是透過根據每個類別計算來完成，因為沒有整體的正或負類別。 例如，運算 ‘Iris-setosa’ 類別的精確度或回收時，假設這是正類別，其他所有類別則是負類別。

![交叉驗證多元分類模型](./media/evaluate-model-performance/12.png)

圖 12. 交叉驗證多元分類模型。

![多元分類模型的交叉驗證結果](./media/evaluate-model-performance/13.png)

圖 13. 多元分類模型的交叉驗證結果。

<!-- Module References -->
[cross-validate-model]: https://msdn.microsoft.com/library/azure/75fb875d-6b86-4d46-8bcc-74261ade5826/
[evaluate-model]: https://msdn.microsoft.com/library/azure/927d65ac-3b50-4694-9903-20f6c1672089/
[linear-regression]: https://msdn.microsoft.com/library/azure/31960a6f-789b-4cf7-88d6-2e1152c0bd1a/
[multiclass-decision-forest]: https://msdn.microsoft.com/library/azure/5e70108d-2e44-45d9-86e8-94f37c68fe86/
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
[score-model]: https://msdn.microsoft.com/library/azure/401b4f92-e724-4d5a-be81-d5b0ff9bdb33/
[split]: https://msdn.microsoft.com/library/azure/70530644-c97a-4ab6-85f7-88bf30a8be5f/
[train-model]: https://msdn.microsoft.com/library/azure/5cc7053e-aa30-450d-96c0-dae4be720977/
[two-class-logistic-regression]: https://msdn.microsoft.com/library/azure/b0fd7660-eeed-43c5-9487-20d9cc79ed5d/

