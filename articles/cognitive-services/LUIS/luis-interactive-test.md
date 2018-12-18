---
title: 在 LUIS 入口網站中測試您的 LUIS 應用程式
titleSuffix: Azure Cognitive Services
description: 使用 Language Understanding (LUIS) 來持續調整您的應用程式，以改善應用程式及提升其語言理解能力。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/06/2018
ms.author: diberry
ms.openlocfilehash: 6bb03975e36e93c2496eb4c6013be21ebc891ed1
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47039696"
---
# <a name="test-your-luis-app"></a>測試 LUIS 應用程式
<a name="train-your-app"></a>
[測試](luis-concept-test.md)應用程式是一種反覆程序。 在您定型 LUIS 應用程式之後，請使用範例語句來測試它，查看它是否能正確地辨識意圖和實體。 如果為否，請更新 LUIS 應用程式，然後重新進行定型和測試。 

<!-- anchors for H2 name changes -->
<a name="test-your-app"></a>
<a name="access-the-test-page"></a>
<a name="luis-interactive-testing"></a>
## <a name="test-an-utterance"></a>測試語句

1. 在 [我的應用程式] 頁面上選取應用程式名稱，來存取應用程式。 

2. 若要存取 [測試] 滑出面板，請選取應用程式上方面板中的 [測試]。

    ![定型和測試應用程式頁面](./media/luis-how-to-interactive-test/test.png)

3. 在文字方塊中輸入語句，並選取 Enter。 您可以針對 [測試] 輸入任意數目的測試語句，但一次只能輸入一個語句。

4. 該語句，連同評分最高的意圖及其分數，會被新增至文字方塊底下的語句清單中。

    ![互動式測試識別出錯誤意圖](./media/luis-how-to-interactive-test/test-weather-1.png)

## <a name="clear-test-panel"></a>清除測試面板
若要從測試主控台清除所有輸入的測試語句及其結果，請選取 [測試] 面板左上角的 [重新開始]。 

## <a name="close-test-panel"></a>關閉測試面板
若要關閉 [測試] 面板，請再次選取 [測試] 按鈕。

## <a name="inspect-score"></a>檢查分數
您可在 [檢查] 面板中檢查測試結果的詳細資料。 
 
1. [測試] 滑出面板開啟時，請針對您想要比較的語句選取 [檢查]。 

    ![[檢查] 按鈕](./media/luis-how-to-interactive-test/inspect.png)

2. [檢查] 面板隨即出現。 該面板包含評分最高的意圖，以及任何已識別的實體。 該面板會顯示所選語句的結果。

    ![[檢查] 按鈕](./media/luis-how-to-interactive-test/inspect-panel.png)

## <a name="correct-top-scoring-intent"></a>修正評分最高的意圖

1. 若評分最高的意圖是錯誤的，請選取 [編輯] 按鈕。

2.  在下拉式清單中，為該語句選取正確的意圖。

    ![選取正確的意圖](./media/luis-how-to-interactive-test/intent-select.png)

## <a name="view-sentiment-results"></a>檢視情感結果

若已在 [[發行](luis-how-to-publish-app.md#enable-sentiment-analysis)] 頁面上設定 [情感分析]，測試結果將會包含在語句中找到的情感。 

![具有情感分析之 [測試] 窗格的影像](./media/luis-how-to-interactive-test/sentiment.png)

## <a name="correct-matched-patterns-intent"></a>修正相符模式的意圖
若您有使用[模式](luis-concept-patterns.md)且語句有符合某個模式，但系統所預測的意圖是錯誤的，請選取模式旁邊的 [編輯] 連結，然後選取正確的意圖。

## <a name="compare-with-published-version"></a>與已發行的版本比較
您可以搭配已發行的[端點](luis-glossary.md#endpoint)版本來測試應用程式的作用中版本。 在 [檢查] 面板中，選取 [與已發行比較]。 針對已發行模型所做的任何測試，將會從您的 Azure 訂用帳戶配額餘額中扣除。 

![[與已發行比較]](./media/luis-how-to-interactive-test/inspect-panel-compare.png)

## <a name="view-endpoint-json-in-test-panel"></a>在 [測試] 面板中檢視端點 JSON
您可以選取 [顯示 JSON 檢視] 來檢視針對比較傳回的端點 JSON。

![已發行的 JSON 回應](./media/luis-how-to-interactive-test/inspect-panel-compare-json.png)

<!--Service name is 'Bing Spell Check v7 API' in the portal-->
## <a name="additional-settings-in-test-panel"></a>[測試] 面板中的其他設定

### <a name="luis-endpoint"></a>LUIS 端點
若您有數個 LUIS 端點，請使用 [測試] 面板 [發行] 窗格上的 [其他設定] 連結，以變更用於測試的端點。 若您不確定該使用哪一個端點，請選取預設的 [Starter_Key]。 

![已醒目提示 [其他選項] 的 [測試] 面板](./media/luis-how-to-interactive-test/interactive-with-spell-check-service-key.png)


### <a name="view-bing-spell-check-corrections-in-test-panel"></a>在 [測試] 面板中檢視 Bing 拼字檢查的修正
檢視拼字修正的需求： 

* 已發行的應用程式
* Bing 拼字檢查[服務金鑰](https://azure.microsoft.com/try/cognitive-services/?api=spellcheck-api)。 該服務金鑰並不會儲存，並需要針對每個瀏覽器工作階段進行重設。 

使用下列程序以在 [測試] 窗格結果中包含 [Bing 拼字檢查 v7](https://azure.microsoft.com/services/cognitive-services/spell-check/) 服務。 

1. 在 [測試] 窗格中輸入語句。 當系統預測出該語句時，請選取位於您所輸入語句下方的 [[檢查](#inspect-score)]。 

2. 當 [檢查] 面板開啟時，選取 [[與已發行比較](#compare-with-published-version)]。 

3. 當 [已發行] 面板開啟時，選取 [[其他設定](#additional-settings-in-test-panel)]。

4. 在快顯對話方塊中，輸入您的 **Bing 拼字檢查**服務金鑰。 
    ![輸入 Bing 拼字檢查服務金鑰](./media/luis-how-to-interactive-test/interactive-with-spell-check-service-key.png)

5. 輸入具有錯誤拼字的查詢 (例如 `book flite to seattle`)，然後選取 Enter。 在傳送至 LUIS 的查詢中，系統會取代單字 `flite` 的錯誤拼字，因此結果的 JSON 會同時顯示原始的查詢 (`query`)，以及已修正拼字的查詢 (`alteredQuery`)。

    ![已修正拼字的 JSON](./media/luis-how-to-interactive-test/interactive-with-spell-check-results.png)

<a name="json-file-with-no-duplicates"></a>
<a name="import-a-dataset-file-for-batch-testing"></a>
<a name="export-rename-delete-or-download-dataset"></a>
<a name="run-a-batch-test-on-your-trained-app"></a>
<a name="access-batch-test-result-details-in-a-visualized-view"></a>
<a name="filter-chart-results-by-intent-or-entity"></a>
<a name="investigate-false-sections"></a>
<a name="view single-point utterance data"></a>
<a name="relabel-utterances-and-retrain"></a>
<a name="false-test-results"></a>
## <a name="batch-testing"></a>批次測試
請參閱批次測試的[概念](luis-concept-batch-test.md)，並了解[如何](luis-how-to-batch-test.md)測試語句批次。

## <a name="next-steps"></a>後續步驟

若測試顯示出您的 LUIS 應用程式無法識別正確的意圖和實體，則您可以透過標示更多語句或新增功能，以提升 LUIS 應用程式的精確度。 

* [使用 LUIS 標示建議的語調](luis-how-to-review-endoint-utt.md) 
* [使用功能來改善 LUIS 應用程式效能](luis-how-to-add-features.md) 
