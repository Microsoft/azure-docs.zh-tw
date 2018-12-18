---
title: Machine Learning Studio 中的簡易實驗 | Microsoft Docs
description: 此機器學習服務教學課程會引導您輕鬆進行資料科學實驗。 我們將使用迴歸演算法預測汽車價格。
keywords: 實驗、線性迴歸、機器學習服務演算法、機器學習服務教學課程、預測性模型技術、資料科學實驗
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: b6176bb2-3bb6-4ebf-84d1-3598ee6e01c6
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/20/2017
ms.openlocfilehash: ac7b208c97ad82d06b8cdc75e04ccd0c787d335f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46951316"
---
# <a name="machine-learning-tutorial-create-your-first-data-science-experiment-in-azure-machine-learning-studio"></a>機器學習服務教學課程：在 Azure Machine Learning Studio 中建立您的第一個資料科學實驗

如果您從未使用過 **Azure Machine Learning Studio**，本教學課程很適合您。

在本教學課程中，我們將逐步解說第一次如何使用 Studio 建立機器學習服務實驗。 此實驗將測試一個分析模型，該模型會根據使用製造和技術規格等不同變數，來預測汽車的價格。

> [!NOTE]
> 本教學課程說明如何拖放模組到您的實驗、將它們連接在一起、執行實驗及檢視結果的基本概念。 我們不討論機器學習服務或如何選取及使用 100 個以上內建的演算法和資料操作模組的一般主題。
>
>如果您是機器學習服務的新手，[初學者資料科學](data-science-for-beginners-the-5-questions-data-science-answers.md)影片系列可能是不錯的起點。 此影片系列使用日常用語和概念，是絕佳的機器學習服務介紹。
>
>如果您已熟悉機器學習服務，但是想了解更多關於 Machine Learning Studio 的一般資訊及其所包含的演算法，這裡有一些不錯的資源：
>
- [什麼是 Machine Learning Studio？](what-is-ml-studio.md) - 這是 Studio 的高階概觀。
- [機器學習服務基本概念 (含演算法範例)](basics-infographic-with-algorithm-examples.md) - 如果您想深入了解 Machine Learning Studio 包含的不同類型機器學習服務演算法，此資訊圖很有用。
- [機器學習服務指南](https://gallery.cortanaintelligence.com/Tutorial/Machine-Learning-Guide-1) - 本指南涵蓋類似上述資訊圖的資訊，但採用互動式格式。
- [機器學習服務演算法小祕技](algorithm-cheat-sheet.md)和[如何選擇 Microsoft Azure Machine Learning 的演算法](algorithm-choice.md) - 這個可下載的海報和隨附的文章深入探討 Studio 演算法。
- [Machine Learning Studio︰演算法和模組說明](https://msdn.microsoft.com/library/azure/dn905974.aspx) - 這是所有 Studio 模組的完整參考，還包括機器學習服務演算法。

<!-- -->

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="how-does-machine-learning-studio-help"></a>Machine Learning Studio 如何協助您？

Machine Learning Studio 可讓您輕鬆地使用以預測性模型技術預先程式化的拖放模組來設定實驗。

使用互動式的視覺化工作區，可以拖放***資料集***和***分析***模組到互動式畫布。 將它們連接在一起，可組成在 Machine Learning Studio 中執行的***實驗***。
您可以***建立模型***、***訓練模型***以及***評分與測試模型***。

您可以逐一查看模型設計、編輯並執行實驗，直到它能提供您想要的結果。 當您的模型就緒後，您可以將其發行為 ***Web 服務***，讓其他人可以傳送新資料到該服務，並取得預測結果。

## <a name="open-machine-learning-studio"></a>開啟 Machine Learning Studio

若要開始使用 Studio，請移至 [https://studio.azureml.net](https://studio.azureml.net)。 如果您之前已登入過 Machine Learning Studio，請按一下 [登入]。 否則，請按一下 [由此註冊] 並選擇免費或付費選項。

![登入 Machine Learning Studio][sign-in-to-studio]
<br/>
***登入 Machine Learning Studio***

## <a name="five-steps-to-create-an-experiment"></a>建立實驗的五個步驟

在這個機器學習教學課程中，您可以遵循下列五個基本步驟，在 Machine Learning Studio 中建置實驗，來建立、訓練及評分您的模型：

- **建立模型**
    - [步驟 1：取得資料]
    - [步驟 2︰ 準備資料]
    - [步驟 3：定義功能]
- **訓練模型**
    - [步驟 4：選擇及套用學習演算法]
- **對模型評分與測試**
    - [步驟 5：預測新的汽車價格]

[步驟 1：取得資料]: #step-1-get-data
[步驟 2︰ 準備資料]: #step-2-prepare-the-data
[步驟 3：定義功能]: #step-3-define-features
[步驟 4：選擇及套用學習演算法]: #step-4-choose-and-apply-a-learning-algorithm
[步驟 5：預測新的汽車價格]: #step-5-predict-new-automobile-prices

> [!TIP] 
> 您可以在 [Azure AI 資源庫](https://gallery.cortanaintelligence.com)中找到下列實驗的工作複本。 移至**[您的第一個資料科學實驗 - 汽車價格預測](https://gallery.cortanaintelligence.com/Experiment/Your-first-data-science-experiment-Automobile-price-prediction-1)**，按一下 [在 Studio 中開啟]，以將實驗的複本下載到您的 Machine Learning Studio 工作區。


## <a name="step-1-get-data"></a>步驟 1：取得資料

要執行機器學習服務，首先您需要的是資料。
Machine Learning Studio 隨附多個範例資料集供您使用，或者，您可以從許多來源匯入資料。 在此範例中，我們將使用您的工作區內含的範例資料集**汽車價格資料 (原始)**。
此資料集將包含許多個別汽車的項目，包括製造、模型、技術規格和價格等資訊。

以下說明如何匯入資料集到您的實驗。

1. 按一下 Machine Learning Studio 視窗底部的 [+新增]，並選取 [實驗]，然後選取 [空白實驗] 以建立新的實驗。

2. 您可以在畫布頂端看到實驗的預設名稱。 選取這段文字，然後重新命名為有意義的名稱，例如**汽車價格預測**。 此名稱不必是唯一的。

    ![將實驗重新命名][rename-experiment]

2. 實驗畫布左側是資料集和模組的調色盤。 在此調色盤頂端的 [搜尋] 方塊中輸入**汽車**，可尋找標示為**汽車價格資料 (原始)** 的資料集。 將此資料集拖曳到實驗畫布。

    ![找出汽車資料集，並將它拖曳到實驗畫布上][type-automobile]
    <br/>
    ***找出汽車資料集，並將它拖曳到實驗畫布上***

若想知道此資料的呈現情形，請按一下汽車資料集底部的輸出連接埠，然後選取 [視覺化] 。

![按一下輸出連接埠，然後選取 [視覺化]][select-visualize]
<br/>
***按一下輸出連接埠，然後選取 [視覺化]***

> [!TIP]
> 資料集和模組以小圓圈代表輸入和輸出連接埠，輸入連接埠位在頂端，輸出連接埠在底端。
若要在您的實驗中建立資料流程，您要將某個模組的輸出連接埠連接到另一個模組的輸入連接埠。
您隨時可以按一下資料集或模組的輸出連接埠，以查看資料於該時間點在資料流程中的型態。

在此範例資料集中，汽車的每個執行個體會顯示為資料列，而與每部汽車相關聯的變數會顯示為資料行。 使用特定一款汽車的變數，我們要嘗試預測最右側資料行中的價格 (資料行 26，標題為「價格」)。

![檢視資料視覺效果視窗中的汽車資料][visualize-auto-data]
<br/>
***檢視資料視覺效果視窗中的汽車資料***

按一下右上角的 "**x**"，以關閉視覺化視窗。

## <a name="step-2-prepare-the-data"></a>步驟 2︰準備資料

資料集通常必須先經過某些前置處理，才能進行分析。 例如，您可能已經注意到在各種不同資料列的資料行中有遺漏的值。 必須清除這些遺漏的值，讓模型才能正確地分析資料。 在我們的案例中，我們將移除含有遺漏值的所有資料列。 此外， **自負虧損** 資料行含有比例很高的遺漏值，因此我們會將該資料行從模型中完全排除。

> [!TIP]
> 在使用大部分的模組時，都必須從輸入資料中清除遺漏值。

我們先新增一個模組以完整移除 [自負虧損] 資料行，然後再新增另一個模組以移除含有遺漏資料的任何資料列。

1. 在模組調色盤頂端的 [搜尋] 方塊中輸入**選取資料行**以尋找[選取資料集中的資料行][select-columns]模組，然後將其拖曳到實驗畫布。 此模組可讓我們選取要將哪些資料行包含在模型中，或是從模型中排除。

2. 將**汽車價格資料 (原始)** 資料集的輸出連接埠連接到[選取資料集中的資料行][select-columns]模組的輸入連接埠。

    ![將 [選取資料集中的資料行] 模組新增到實驗畫布並連接它][type-select-columns]
    <br/>
    ***將「選取資料集中的資料行」模組新增到實驗畫布並連接它***

3. 按一下 [選取資料集中的資料行][select-columns] 模組，然後按一下 [屬性] 窗格中的 [啟動資料行選取器]。

    - 在左側按一下 [套用規則] 
    - 在 [開始於] 下，按一下 [所有資料行]。 這會引導 [選取資料集中的資料行][select-columns] 通過所有資料行 (但即將排除的資料行除外)。
    - 在下拉式清單中，選取 [排除] 和 [資料行名稱]，然後按一下文字方塊內部。 資料行清單隨即顯示。 選取 [自負虧損]，該資料行就會新增到文字方塊中。
    - 按一下核取記號 ([確定]) 按鈕，以關閉資料行選取器 (位於右下方)。

    ![啟動資料行選取器，並排除 [自負虧損] 資料行][launch-column-selector]
    <br/>
    ***啟動資料行選取器，並排除 [自負虧損] 資料行***

    現在，[選取資料集中的資料行] 的屬性窗格指出它會傳遞資料集中的所有資料行，但 [自負虧損] 除外。

    ![[屬性] 窗格顯示 [自負虧損] 資料行已被排除][showing-excluded-column]
    <br/>
    ***[屬性] 窗格顯示 [自負虧損] 資料行已被排除***

    > [!TIP]
    您可以按兩下模組並輸入文字，為模組新增註解。 這有助於您快速檢視模組在您實驗中的執行情況。 在此案例中，按兩下 [選取資料集中的資料行][select-columns] 模組，然後輸入註解「排除自負虧損」。

    ![按兩下模組以新增註解][add-comment]
    <br/>
    ***按兩下模組以新增註解***

3. 將[清除遺漏的資料][clean-missing-data] 模組拖曳到實驗畫布，然後將其連接到 [選取資料集中的資料行][select-columns] 模組。 在 [屬性] 窗格中，選取 [清除模式] 底下的 [移除整個資料列]。 這會指示[清除遺漏的資料][clean-missing-data]藉由移除含任何遺漏值的資料列以清除資料。 按兩下模組，並輸入註解「移除遺漏值資料列」。

    ![將 [清除遺漏的資料] 的清除模式設為 [移除整個資料列]][set-remove-entire-row]
    <br/>
    ***將 [清除遺漏的資料] 的清除模式設為 [移除整個資料列]***

4. 按一下頁面底部的 [執行]，以執行實驗。

    實驗執行完成時，所有模組都會呈現綠色核取標記，表示它們已順利完成。 同時也請留意位於右上角的 **執行完成** 狀態。

![執行後，實驗看起來應如下所示：][early-experiment-run]
<br/>
***執行後，實驗看起來應如下所示：***

> [!TIP]
> 為什麼要立即執行實驗？ 藉由執行實驗，就會從資料集傳遞我們資料的資料行定義 (透過[選取資料集中的資料行][select-columns]模組，及透過[清除遺漏的資料][clean-missing-data]模組)。 這表示，我們連接到[清除遺漏的資料][clean-missing-data]的任何模組也會有相同的資訊。

我們的實驗到目前為止所完成的是清除資料。 若要檢視已清除的資料集，請按一下[清除遺漏的資料][clean-missing-data]模組的左側輸出連接埠，然後選取 [視覺化]。 請注意，此時已不包含 **自負虧損** 資料行，而且也沒有遺漏值。

現在我們已清除資料，而可以指定要在預測模型中使用哪些功能。

## <a name="step-3-define-features"></a>步驟 3：定義功能

在機器學習中， *功能* 是您感興趣的某項目的個別可測量的屬性。 在我們的資料集中，每個資料列都代表一部汽車，而每個資料行是該汽車的功能。

要找到一組理想的功能來建立預測模型，必須針對您要解決的問題進行實驗，並具備相關知識。 有些功能會比其他功能更適合用來預測目標。 此外，某些功能與其他功能具有強相關性，而且可以移除。 例如，市區油耗和高速公路油耗密切相關，我們可以保留其中一項而移除另一項，這對預測不會有大幅影響。

我們將建置一個模型，並在其中使用我們資料集中的功能子集。 您稍後可以重新選取不同功能、再次執行實驗，並確認是否會有較理想的結果。 但是若要開始，讓我們試用下列功能︰

    make, body-style, wheel-base, engine-size, horsepower, peak-rpm, highway-mpg, price


1. 將另一個[選取資料集中的資料行][select-columns]模組拖曳到實驗畫布。 將[清除遺漏資料][clean-missing-data]模組的左側輸出連接埠連接到[選取資料集中的資料行][select-columns]模組的輸入。

    ![將 [選取資料集中的資料行] 模組連接到 [清除遺漏的資料] 模組][connect-clean-to-select]
    <br/>
    ***將「選取資料集中的資料行」模組連接到「清除遺漏的資料」模組***

2. 按兩下模組，並輸入「選取要預測的功能」。

2. 按一下 [屬性] 窗格中的 [啟動資料行選取器]。

3. 按一下 [套用規則] 。

4. 在 [開始於] 下，按一下 [無資料行]。 在 [篩選] 資料列中，選取 [包含] 和 [資料行名稱]，然後在文字方塊中選取資料行名稱的清單。 這會指示模組不要傳遞我們所指定以外的任何資料行 (功能)。

5. 按一下核取記號 ([確定]) 按鈕。

    ![選取要包含在預測中的資料行 (功能)][select-columns-to-include]
    <br/>
    ***選取要包含在預測中的資料行 (功能)***

這會產生已篩選的資料集，其中只包含我們想傳遞至下一個步驟中將使用之學習服務演算法的功能。 稍後您可以返回，並選取不同的功能重新嘗試。

## <a name="step-4-choose-and-apply-a-learning-algorithm"></a>步驟 4：選擇及套用學習演算法

現在，資料已備妥，訓練和測試是建構預測模型的要素。 我們將使用我們的資料來訓練模型，然後測試模型，以檢驗它預測價格的精準度。
<!-- For now, don't worry about *why* we need to train and then test a model.-->

*分類*和*迴歸*是兩種受監督的機器學習服務演算法。 「分類」可從一組已定義的類別預測答案，例如色彩 (紅色、藍色或綠色)。 「迴歸」可用來預測數字。

因為要預測價格，也就是一個數字，因此我們將使用迴歸演算法。 在此範例中，我們將使用簡單*線性迴歸*模型。

> [!TIP]
> 如果您想深入了解不同類型的機器學習服務演算法和使用時機，您可以檢視初學者資料科學系列中的第一部影片[資料科學解答的五個問題](data-science-for-beginners-the-5-questions-data-science-answers.md)。 您也可以查看資訊圖[機器學習服務基本概念 (含演算法範例)](basics-infographic-with-algorithm-examples.md)，或查看[機器學習服務演算法小祕技](algorithm-cheat-sheet.md)。

我們藉由提供一組包含價格的資料來訓練模型。 模型會掃描的資料，然後尋找汽車功能與價格之間的關聯性。 然後，我們將測試模型 - 提供它一組我們熟悉的汽車功能，接著檢驗模型預測已知價格的精準度。

我們將資料分割成個別的訓練和測試資料集，用來訓練和測試模型。

1. 選取 [分割資料][split] 模組並拖曳到實驗畫布，然後將其連接到最後一個 [選取資料集中的資料行][select-columns] 模組。

2. 按一下[分割資料][split]模組以選取它。 尋找 [第一個輸出資料集中的資料列分數] \(在畫布右邊的 [屬性] 窗格)，並將它設為 0.75。 如此，我們將使用百分之 75 的資料來訓練模型，並保留百分之 25 做為測試之用 (稍後，您可以使用不同的百分比來實驗)。

    ![將 [分割資料] 模組的分割分數設為 0.75][set-split-data-percentage]
    <br/>
    ***將「分割資料」模組的分割分數設為 0.75***

    > [!TIP]
    > 變更 [ **隨機種子** ] 參數，可讓您為訓練和測試產生不同的隨機範例。 此參數會控制虛擬隨機數字產生器的植入。

2. 執行實驗。 執行實驗時，[選取資料集中的資料行][select-columns]和[分割資料][split]模組會將資料行定義傳遞至我們接下來要新增的模組。  

3. 若要選取學習演算法，請在畫布左側的模組調色盤中展開 [機器學習服務] 類別，然後展開 [初始化模型]。 這會顯示數個可用來初始化機器學習演算法的模組類別。 在此實驗中，請選取 [迴歸] 類別下的[線性迴歸][linear-regression]模組，然後將其拖曳到實驗畫布。
(您也可以在調色盤搜尋方塊中輸入「線性迴歸」以尋找模組。)

4. 找出[訓練模型][train-model]模組，並將其拖曳到實驗畫布。 將[線性迴歸][linear-regression]模組的輸出連接到[訓練模型][train-model]模組的左側輸入，並將[分割資料][split]模組的訓練資料輸出 (左側連接埠) 連接到[訓練模型][train-model]模組的右側輸入。

    ![將「評分模型」模組連接到「線性迴歸」和「分割資料」模組][connect-train-model]
    <br/>
    ***將「定型模型」模組連接到「線性迴歸」和「分割資料」模組***

5. 按一下[訓練模型][train-model]模組，按一下 [屬性] 窗格中的 [啟動資料行選取器]，然後選取 [價格] 資料行。 這是我們的模型所將預測的值。

    在資料行選取器中選取 [價格] 資料行 (將它從 [可用的資料行] 清單 移到 [選取的資料行] 清單)。

    ![選取 [訓練模型] 模組的價格資料行][select-price-column]
    <br/>
    ***選取「定型模型」模組的價格資料行***

6. 執行實驗。

我們現在有經過訓練的迴歸模型可用來為新的汽車資料評分，以藉此預測價格。

![執行後，實驗此時看起來應如下所示：][second-experiment-run]
<br/>
***執行後，實驗此時看起來應如下所示：***

## <a name="step-5-predict-new-automobile-prices"></a>步驟 5：預測新的汽車價格

現在已完成使用百分之 75 資料模型的訓練，我們可以用它來為其他百分之 25 的資料評分，以了解模型的運作是否理想。

1. 找出[評分模型][score-model]模組，並將其拖曳到實驗畫布。 將[訓練模型][train-model]模組的輸出連接到[評分模型][score-model]的左側輸入連接埠。 將[分割資料][split]模組的測試資料輸出 (右側連接埠) 連接到[評分模型][score-model]的右側輸入連接埠。

    ![將「評分模型」模組連接到「訓練模型」和「分割資料」模組][connect-score-model]
    <br/>
    ***將「評分模型」模組連接到「定型模型」和「分割資料」模組***

2. 執行實驗並檢視[評分模型][score-model]模組的輸出 (按一下[評分模型][score-model]的輸出連接埠，然後選取 [視覺化])。 輸出會顯示價格的預測值，以及來自測試資料的已知值。  

    ![「評分模型」模組的輸出][score-model-output]
    <br/>
    ***「評分模型」模組的輸出***

3. 最後，我們要測試結果的品質。 選取[評估模型][evaluate-model]模組，並將其拖曳至實驗畫布，然後將[評分模型][score-model]模組的輸出連接到[評估模型][evaluate-model]的左側輸入。

    > [!TIP]
    > [評估模型][evaluate-model]模組有兩個輸入連接埠，因為它可以並排方式來比較兩個模型。 稍後，您可以在實驗中新增其他演算法，並使用[評估模型][evaluate-model]查看哪一個提供更好的結果。

4. 執行實驗。

若要檢視[評估模型][evaluate-model]模組的輸出，請按一下輸出連接埠，然後選取 [視覺化]。

![實驗的評估結果][evaluation-results]
<br/>
***實驗的評估結果***

我們的模型會顯示下列統計資料：

- **平均絕對誤差** (MAE)：絕對誤差的平均值 ( *誤差* 是指預測值與實際值之間的差異)。
- **均方根誤差** (RMSE)：對測試資料集所做之預測的平方誤差的評分根平均值。
- **相對絕對誤差**：相對於實際值與所有實際值之平均值之間的絕對差異的絕對誤差平均值。
- **相對平方誤差**：相對於實際值與所有實際值之平均值之間的平方差異的平方誤差平均值。
- **決定係數**：也稱為 **R 平方值**，這是一個統計度量，可指出模型對於資料的適用程度。

針對每個誤差統計資料，越小越好。 值越小，表示預測越接近實際值。 就 [決定係數] 而言，其值愈接近一 (1.0)，預測就愈精準。

## <a name="final-experiment"></a>最終實驗

實驗最終應呈現如下：

![最終實驗][complete-linear-regression-experiment]
<br/>
***最終實驗***

## <a name="next-steps"></a>後續步驟

現在您已經完成第一個機器學習服務教學課程並已設定實驗，您可以繼續改善模型，然後再將它部署為預測型 Web 服務。

- **逐一檢測以嘗試改善模型** - 例如，您可以變更在預測中使用的功能。 或者，您可以修改[線性迴歸][linear-regression]演算法的屬性，或嘗試完全不同的演算法。 您甚至可以同時在實驗中新增多個機器學習演算法，並使用[評估模型][evaluate-model]模組進行比較 (兩兩相比)。
如需如何在單一實驗中比較多個模型的範例，請參閱 [Azure AI 資源庫](https://gallery.cortanaintelligence.com)中的[比較迴歸輸入變數](https://gallery.cortanaintelligence.com/Experiment/Compare-Regressors-5)。

    > [!TIP]
    > 若要複製實驗的任何反覆運算，請使用頁面底部的 [另存新檔] 按鈕。 您可以按一下頁面底部的 [檢視執行歷程記錄]，以檢視實驗的所有反覆運算。 如需詳細資訊，請參閱[在 Azure Machine Learning Studio 中管理實驗逐一查看][runhistory]。

[runhistory]: manage-experiment-iterations.md

- **將模型部署為預測性 Web 服務** - 如果您對模型感到滿意，您可以將其部署為 Web 服務，用以透過新資料預測汽車價格。 如需詳細資訊，請參閱[部署 Azure Machine Learning Web 服務][publish]。

[publish]: publish-a-machine-learning-web-service.md

要深入了解？ 如需建立、訓練、評分及部署模型之程序的更廣泛且更詳細的逐步解說，請參閱[使用 Azure Machine Learning 開發預測性解決方案][walkthrough]。

[walkthrough]: walkthrough-develop-predictive-solution.md

<!-- Images -->
[sign-in-to-studio]: ./media/create-experiment/sign-in-to-studio.png
[rename-experiment]: ./media/create-experiment/rename-experiment.png
[visualize-auto-data]:./media/create-experiment/visualize-auto-data.png
[select-visualize]: ./media/create-experiment/select-visualize.png
[showing-excluded-column]:./media/create-experiment/showing-excluded-column.png
[set-remove-entire-row]:./media/create-experiment/set-remove-entire-row.png
[early-experiment-run]:./media/create-experiment/early-experiment-run.png
[select-columns-to-include]:./media/create-experiment/select-columns-to-include.png
[second-experiment-run]:./media/create-experiment/second-experiment-run.png
[connect-score-model]:./media/create-experiment/connect-score-model.png
[evaluation-results]:./media/create-experiment/evaluation-results.png
[complete-linear-regression-experiment]:./media/create-experiment/complete-linear-regression-experiment.png

<!-- temporarily switching GIFs to PNGs to remove animation --> 
[type-automobile]:./media/create-experiment/type-automobile.png
[type-select-columns]:./media/create-experiment/type-select-columns.png
[launch-column-selector]:./media/create-experiment/launch-column-selector.png
[add-comment]:./media/create-experiment/add-comment.png
[connect-clean-to-select]:./media/create-experiment/connect-clean-to-select.png

[set-split-data-percentage]:./media/create-experiment/set-split-data-percentage.png

<!-- temporarily switching GIFs to PNGs to remove animation --> 
[connect-train-model]:./media/create-experiment/connect-train-model.png
[select-price-column]:./media/create-experiment/select-price-column.png

[score-model-output]:./media/create-experiment/score-model-output.png

<!-- Module References -->
[evaluate-model]: https://msdn.microsoft.com/library/azure/927d65ac-3b50-4694-9903-20f6c1672089/
[linear-regression]: https://msdn.microsoft.com/library/azure/31960a6f-789b-4cf7-88d6-2e1152c0bd1a/
[clean-missing-data]: https://msdn.microsoft.com/library/azure/d2c5ca2f-7323-41a3-9b7e-da917c99f0c4/
[select-columns]: https://msdn.microsoft.com/library/azure/1ec722fa-b623-4e26-a44e-a50c6d726223/
[score-model]: https://msdn.microsoft.com/library/azure/401b4f92-e724-4d5a-be81-d5b0ff9bdb33/
[split]: https://msdn.microsoft.com/library/azure/70530644-c97a-4ab6-85f7-88bf30a8be5f/
[train-model]: https://msdn.microsoft.com/library/azure/5cc7053e-aa30-450d-96c0-dae4be720977/
