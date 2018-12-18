---
title: 步驟 4：訓練和評估預測分析模型 | Microsoft Docs
description: 開發預測解決方案逐步解說的步驟 4：在 Azure Machine Learning Studio 中定型、計分和評估多個模型。
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: d905f6b3-9201-4117-b769-5f9ed5ee1cac
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/23/2017
ms.openlocfilehash: 95937ee74be3f6facbc482c3cfd80a8af9128ea3
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34836004"
---
# <a name="walkthrough-step-4-train-and-evaluate-the-predictive-analytic-models"></a>逐步解說步驟 4：定型和評估預測分析模型
此主題包含[在 Azure Machine Learning 中開發預測性分析解決方案](walkthrough-develop-predictive-solution.md)逐步解說的第四個步驟

1. [建立機器學習服務工作區](walkthrough-1-create-ml-workspace.md)
2. [上傳現有資料](walkthrough-2-upload-data.md)
3. [建立新實驗](walkthrough-3-create-new-experiment.md)
4. **訓練及評估模型**
5. [部署 Web 服務](walkthrough-5-publish-web-service.md)
6. [存取 Web 服務](walkthrough-6-access-web-service.md)

- - -
使用 Azure Machine Learning Studio 來建立機器學習服務模型的優點之一，是能夠在單一實驗中一次嘗試多個模型類型，並比較結果。 這類實驗可協助您針對問題找到最佳解決方案。

在我們正於這個逐步解說中開發的實驗內，將會建立兩種不同的模型，然後比較其計分結果，以決定要用於最終實驗的演算法。  

有各種不同的模型可供我們選擇。 若要查看可用的模型，請在模組選擇區展開 [機器學習] 節點，然後展開 [初始化模型]，再選擇其下方的節點。 基於本實驗的目的，我們將選取[二元支援向量機器][two-class-support-vector-machine] (SVM) 和[二元促進式決策樹][two-class-boosted-decision-tree]模組。    

> [!TIP]
> 如需取得說明以決定哪一種機器學習服務演算法最適合您正在嘗試解決的特定問題，請參閱 [如何選擇 Microsoft Azure Machine Learning 的演算法](algorithm-choice.md)。
> 
> 

## <a name="train-the-models"></a>訓練模型

我們將在此實驗中新增[二元促進式決策樹][two-class-boosted-decision-tree]模組和[二元支援向量機器][two-class-support-vector-machine]模組。

### <a name="two-class-boosted-decision-tree"></a>二元促進式決策樹

首先，讓我們設定促進式決策樹模型。

1. 在模組選擇區中找到 [[二元促進式決策樹]][two-class-boosted-decision-tree] 模組，將其拖曳到畫布上。

2. 找到 [[訓練模型]][train-model] 模組，將它拖曳到畫布上，然後將[ [二元促進式決策樹]][two-class-boosted-decision-tree] 模組的輸出連接到 [[訓練模型]][train-model] 模組的左側輸入連接埠。
   
   [[二元促進式決策樹]][two-class-boosted-decision-tree] 模組會將一般模組初始化，而 [[訓練模型]][train-model] 則會使用訓練資料來訓練模型。 

3. 將左側 [[執行 R 指令碼]][execute-r-script] 模組的左輸出連接到 [[訓練模型]][train-model] 模組的右側輸入連接埠 (我們在本逐步解說[步驟 3](walkthrough-3-create-new-experiment.md)決定的做法，使用「分割資料」模組左側所傳來的資料用於訓練)。
   
   > [!TIP]
   > 在這項實驗中，我們不需要 [[執行 R 指令碼]][execute-r-script] 模組的兩個輸入和一個輸出，因此可以讓它們保持未連接。 
   > 
   > 

實驗的這部分目前看起來如下：  

![Training a model][1]

現在我們要告訴 [[訓練模型]][train-model] 模組，我們要讓模型預測信用風險值。

1. 選取 [[訓練模型]][train-model] 模組。 按一下 [屬性] 窗格中的 [啟動資料行選取器]。

2. 在 [選取單一資料行] 對話方塊中，在 [可用的資料行] 下的 [搜尋] 欄位中輸入「信用風險」，然後選取下方的 [信用風險]，按一下向右箭號按鈕 (**>**) 將 [信用風險] 移至 [選取的資料行]。 

    ![為訓練模型模組選取信用風險資料行][0]

3. 按一下 [確定] \(打勾記號)。

### <a name="two-class-support-vector-machine"></a>二元支援向量機器

接下來，我們設定 SVM 模型。  

首先，簡單說明一下 SVM。 強化的決策樹適合處理任何類型的特性。 不過，因為 SVM 模組會產生線性分類器，而它所產生的模型在所有數值特性都有相同的尺度時，將具有最佳檢定誤差。 為了將所有數值特徵轉換成相同的尺度，我們使用 Tanh 轉換 (搭配[標準化資料][normalize-data]模組)。 這會將我們的數字轉換到 [0,1] 範圍內。 SVM 模組會將字串特徵轉換為類別特性，再轉換為二進位 0/1 特徵，因此我們無須手動轉換字串特徵。 此外，我們不想要轉換 [信用風險] 資料行 (資料行 21) - 它是數值，但這是我們定型模型來預測的值，因此必須維持原狀。  

若要設定 SVM 模型，請執行下列動作：

1. 在模組選擇區中找到 [[二元支援向量機器]][two-class-support-vector-machine] 模組，將它推曳到畫布上。

2. 以滑鼠右鍵按一下 [[訓練模型]][train-model] 模組，選取 [複製]，然後以滑鼠右鍵按一下畫布並選取 [貼上]。 [[訓練模型]][train-model] 模組複本的資料行選擇與原始模組相同。

3. 將 [[二元支援向量機器]][two-class-support-vector-machine] 模組的輸出連接到第二個 [[訓練模型]][train-model] 模組的左側輸入連接埠。

4. 找到 [[標準化資料]][normalize-data] 模組，將其拖曳到畫布上。

5. 將左側 [[執行 R 指令碼]][execute-r-script] 模組的左側輸出連接到此模組的輸出 (請注意，模組的輸出連接埠可能連接到多個其他模組)。

6. 將 [[標準化資料]][normalize-data] 模組的左側輸出連接埠連接到第二個 [[訓練模型]][train-model] 模組的右側輸入連接埠。

實驗的這部分目前看起來如下：  

![Training the second model][2]  

現在，設定 [[標準化資料]][normalize-data] 模組︰

1. 按一下以選取 [[標準化資料]][normalize-data] 模組。 在 [屬性] 窗格中，選取 [Tanh] 做為 [轉換方法] 參數。

2. 按一下 [啟動資料行選取器]，選取 [開始於] 的 [無資料行]，在第一個下拉式清單中選取 [包含]，在第二個下拉式清單中選取 [資料行類型]，然後在第三個下拉式清單中選取 [數值]。 這樣會指定轉換所有數值資料行 (且僅限數值)。

3. 按一下此資料列右側的加號 (+) - 這會建立一排下拉式清單。 在第一個下拉式清單中選取 [排除]，在第二個下拉式清單中選取 [資料行名稱]，然後在文字欄位中輸入「信用風險」。 這會指定應忽略 [信用風險] 資料行 (需要這麼做是因為此資料行為數值，如未排除，它會被轉換)。

4. 按一下 **[確定]** \(打勾記號)。  

    ![選取標準化資料模組的資料行][5]

[[標準化資料]][normalize-data] 模組現在已設定為在所有數值資料行上執行 Tanh 轉換 ([信用風險] 資料行除外)。  

## <a name="score-and-evaluate-the-models"></a>計分及評估模型

我們將使用由[[資料分割]][split] 模組所分開的測試資料，給我們訓練的模型評分。 然後，我們就可以比較兩個模型的結果，了解何者產生的結果較佳。  

### <a name="add-the-score-model-modules"></a>新增評分模型模組

1. 找到 [[評分模型]][score-model] 模組並拖曳到畫布上。

2. 將已連接至 [[二元促進式決策樹]][two-class-boosted-decision-tree] 模組的 [[訓練模型]][train-model] 模組，連接到 [[評分模型]][score-model] 模組的左側輸入連接埠。

3. 將右側 [[執行 R 指令碼]][execute-r-script] 模組 (或測試資料) 連接到 [[評分模型]][score-model] 模組的右側輸入連接埠。

    ![已連接評分模型模組][6]
   
   [[評分模型]][score-model] 模組可以立即從測試資料中採取信用資訊，並且將模型產生的預測情況與測試資料中的實際信用風險資料行進行比較。

4. 複製並貼上 [[評分模型]][score-model] 模組來建立第二個複本。

5. 將 SVM 模型的輸出 (亦即，已連接到 [[二元支援向量機器]][two-class-support-vector-machine] 的 [[訓練模型]][train-model] 模組的輸出連接埠)，連接到第二個 [[評分模型]][score-model] 模組的輸入連接埠。

6. 在 SVM 模型中，我們必須像是轉換定型資料一樣，對測試資料進行相同的轉換。 因此，請複製並貼上 [[標準化資料]][normalize-data] 模組來建立第二個複本，再將其連接到右邊 [[執行 R 指令碼]][execute-r-script] 模組。

7. 將第二個 [[標準化資料]][normalize-data] 模組的左邊輸出連接到第二個 [[評分模型]][score-model] 模組的右邊輸入連接埠。

    ![已連接兩個評分模型模組][7]

### <a name="add-the-evaluate-model-module"></a>新增評估模型模組

為了評估兩個評計分結果並加以比較，我們使用[評估模型][evaluate-model]模組。  

1. 找到 [[評估模型]][evaluate-model] 模組並拖曳到畫布上。

2. 將與促進式決策樹模型相關聯的 [[評分模型]][score-model]模組的輸出連接埠，連接到 [[評估模型]][evaluate-model] 模組的左側輸入連接埠。

3. 將其他 [[評分模型]][score-model] 模組連接到右側輸入連接埠。  

    ![已連接評估模型模組][8]

### <a name="run-the-experiment-and-check-the-results"></a>執行實驗並檢查結果

若要執行實驗，請按一下畫布下方的 [執行] 按鈕。 可能需要數分鐘的時間。 每個模組上的旋轉指示器表示正在執行，模組完成時會出現綠色打勾記號。 當所有模組都出現核取記號時，表示實驗執行完成。

實驗目前看起來如下：  

![Evaluating both models][3]

若要檢查結果，按一下 [[評估模型]][evaluate-model] 模組的輸出連接埠，然後選取 [視覺化]。  

[[評估模型]][evaluate-model] 模組會產生一對曲線和度量，讓您比較兩個評分模型的結果。 您可以將結果顯示成「受測者操作特徵 (ROC)」曲線、「正確性/召回」曲線或「升力」曲線。 其他顯示的資料還包括混淆矩陣、曲線下面積 (AUC) 的累計值，以及其他度量。 您可以將滑動軸左右移動來變更臨界值，觀察這樣如何影響度量組。  

在圖形的右邊，按一下 [已計分的資料集] 或 [要比較的已計分資料集]，以醒目提示相關聯的曲線，並在下方顯示相關聯的度量。 在曲線的圖例中，"Scored dataset" 對應至 [[評估模型]][evaluate-model] 模組的左側輸入埠 - 在我們的案例中，這是促進式決策樹模型。 「要比較的已計分資料集」會對應至右側輸入連接埠 (在本例中為 SVM 模型)。 按一下其中一個標籤時，該模型的曲線會反白顯示，並顯示如下圖所示的相對應度量。  

![ROC curves for models][4]

您可以檢查這些值，以判斷哪個模型最可能提供您想要的結果。 您可以返回並變更不同模型中的參數值，以反覆執行實驗。 

解讀這些結果和微調模型效能的藝術與科學超出本逐步解說的範圍。 如需更多說明，請參閱下列文章：
- [如何在 Azure Machine Learning 中評估模型效能](evaluate-model-performance.md)
- [選擇參數來最佳化 Azure Machine Learning 中的演算法](algorithm-parameters-optimize.md)
- [在 Azure Machine Learning 中解讀模型結果](interpret-model-results.md)

> [!TIP]
> 每次執行實驗，[執行歷程記錄] 中就會保留該筆逐一查看的記錄。 您可以檢視這些反覆運算，按一下畫布下方的 [檢視執行歷程記錄]  即可回到其中任何一個。 您也可以按一下 [屬性] 窗格中的 [先前執行]，回到您目前開啟的反覆運算之前的那一個反覆運算。
> 
> 您可以按一下畫布下方的 [另存新檔]  ，為實驗的任何反覆項目製作一個複本。 
> 使用實驗的 [摘要] 和 [描述] 屬性，以記錄在您實驗反覆運算中已嘗試的動作。
> 
> 如需詳細資訊，請參閱 [在 Azure Machine Learning Studio 中管理實驗逐一查看](manage-experiment-iterations.md)。  
> 
> 

- - -
**下一步：[部署 Web 服務](walkthrough-5-publish-web-service.md)**

[0]: ./media/walkthrough-4-train-and-evaluate-models/train-model-select-column.png
[1]: ./media/walkthrough-4-train-and-evaluate-models/experiment-with-train-model.png
[2]: ./media/walkthrough-4-train-and-evaluate-models/svm-model-added.png
[3]: ./media/walkthrough-4-train-and-evaluate-models/final-experiment.png
[4]: ./media/walkthrough-4-train-and-evaluate-models/roc-curves.png
[5]: ./media/walkthrough-4-train-and-evaluate-models/normalize-data-select-column.png
[6]: ./media/walkthrough-4-train-and-evaluate-models/score-model-connected.png
[7]: ./media/walkthrough-4-train-and-evaluate-models/both-score-models-added.png
[8]: ./media/walkthrough-4-train-and-evaluate-models/evaluate-model-added.png


<!-- Module References -->
[evaluate-model]: https://msdn.microsoft.com/library/azure/927d65ac-3b50-4694-9903-20f6c1672089/
[execute-r-script]: https://msdn.microsoft.com/library/azure/30806023-392b-42e0-94d6-6b775a6e0fd5/
[normalize-data]: https://msdn.microsoft.com/library/azure/986df333-6748-4b85-923d-871df70d6aaf/
[score-model]: https://msdn.microsoft.com/library/azure/401b4f92-e724-4d5a-be81-d5b0ff9bdb33/
[train-model]: https://msdn.microsoft.com/library/azure/5cc7053e-aa30-450d-96c0-dae4be720977/
[two-class-boosted-decision-tree]: https://msdn.microsoft.com/library/azure/e3c522f8-53d9-4829-8ea4-5c6a6b75330c/
[two-class-support-vector-machine]: https://msdn.microsoft.com/library/azure/12d8479b-74b4-4e67-b8de-d32867380e20/
[split]: https://msdn.microsoft.com/library/azure/70530644-c97a-4ab6-85f7-88bf30a8be5f/
