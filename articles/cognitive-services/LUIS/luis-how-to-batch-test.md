---
title: 使用 1000 個範例語句的批次測試
titleSuffix: Azure Cognitive Services
description: 使用 Language Understanding (LUIS) 批次測試集，找出具有錯誤意圖和實體的語句。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/06/2018
ms.author: diberry
ms.openlocfilehash: b4c58a13c8b66add8ebd7e535ecac9d7b0f1075b
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47032116"
---
# <a name="batch-testing-with-a-set-of-example-utterances"></a>使用一組範例語句的批次測試
 批次測試是針對您目前已定型模型的完整測試，以測量其於 LUIS 中的效能。 

<a name="batch-testing"></a>
## <a name="import-a-dataset-file-for-batch-testing"></a>針對批次測試匯入資料集檔案

1. 選取頂端列中的 [測試]，然後選取 [批次測試面板]。

    ![批次測試連結](./media/luis-how-to-batch-test/batch-testing-link.png)

2. 選取 [匯入資料集]。 [匯入新資料集] 對話方塊隨即出現。 選取 [選擇檔案]，並找出具備正確 [JSON 格式](luis-concept-batch-test.md#batch-file-format)且包含*不超過 1,000 個*要測試語句的 JSON 檔案。

    若有匯入錯誤，系統會在瀏覽器頂端的紅色通知列中回報。 當匯入發生錯誤時，將不會建立任何資料集。 如需詳細資訊，請參閱[常見錯誤](luis-concept-batch-test.md#common-errors-importing-a-batch)。

3. 在 [ 資料集名稱] 欄位中，為資料集檔案輸入一個名稱。 資料集檔案包含**語句的陣列**，其中包括標記的意圖和實體。 針對語法，請檢閱[範例批次檔](luis-concept-batch-test.md#batch-file-format)。 

4. 選取 [完成] 。 系統會新增資料集檔案。

## <a name="run-rename-export-or-delete-dataset"></a>執行、重新命名、匯出或刪除資料集
若要執行、重新命名、匯出或刪除資料集，請使用位於資料集列尾端的省略符號 (***...***) 按鈕。

![資料集動作](./media/luis-how-to-batch-test/batch-testing-options.png)

## <a name="run-a-batch-test-on-your-trained-app"></a>對已定型的應用程式執行批次測試

若要執行測試，請選取資料集名稱。 當測試完成時，此列會顯示該資料集的測試結果。

![批次測試結果](./media/luis-how-to-batch-test/run-test.png)

可下載的資料集和針對批次測試所上傳的檔案相同。

|State|意義|
|--|--|
|![測試成功的綠色圓圈圖示](./media/luis-how-to-batch-test/batch-test-result-green.png)|所有語句皆成功。|
|![測試失敗紅色 x 圖示](./media/luis-how-to-batch-test/batch-test-result-red.png)|至少有一個語句意圖不符合預測。|
|![已準備就緒進行測試圖示](./media/luis-how-to-batch-test/batch-test-result-blue.png)|測試已準備好開始執行。|

<a name="access-batch-test-result-details-in-a-visualized-view"></a>
## <a name="view-batch-test-results"></a>檢視批次測試結果 
若要檢閱批次測試結果，請選取 [查看結果]。

![批次測試結果](./media/luis-how-to-batch-test/run-test-results.png)

<!-- Select the **See results** link that appears after you run the test. A scatter graph known as an error matrix displays. The data points represent the utterances in the dataset. Green points indicate correct prediction, and red ones indicate incorrect prediction. The filtering panel on the right side of the screen displays a list of all intents and entities in the app, with a green point for intents/entities that were predicted correctly in all dataset utterances, and a red point for those items with errors. Also, for each intent/entity, you can see the number of correct predictions out of the total utterances.-->


<a name="filter-chart-results-by-intent-or-entity"></a>  
## <a name="filter-chart-results"></a>篩選圖表結果

若要依特定意圖或實體篩選圖表，請於右側的篩選面板中選取意圖或實體。 資料點及其分佈會根據您的選取範圍在圖表中更新。 
 
![視覺化的批次測試結果](./media/luis-how-to-batch-test/filter-by-entity.png) 

## <a name="view-single-point-utterance-data"></a>檢視單一點語句資料
在圖表中，將滑鼠暫留於資料點上，以查看其預測的確定性分數。 選取資料點以擷取它在頁面底部語句清單中的相對應語句。 

![選取的語句](./media/luis-how-to-batch-test/selected-utterance.png)


<a name="relabel-utterances-and-retrain"></a>
<a name="false-test-results"></a>
## <a name="view-section-data"></a>檢視區段資料
在四個區段的圖表中，選取圖表右上角的區段名稱 (例如 **False Positive**)。 該區段中的所有語句會以清單的形式顯示在圖表下方。 

![依區段選取的語句](./media/luis-how-to-batch-test/selected-utterances-by-section.png)

在上面這個影像中，語句 `switch on` 具有 TurnAllOn 意圖的標記，但接收到 None 意圖的預測。 這代表 TurnAllOn 意圖需要更多範例語句，以取得預期的預測結果。 

圖表中兩個以紅色顯示的區段，表示沒有符合預期預測的語句。 這代表它們是 LUIS 需要進行更多定型的語句。 

圖表中另外兩個以綠色顯示的區段則有符合預期的預測。

## <a name="next-steps"></a>後續步驟

若測試顯示出您的 LUIS 應用程式無法識別正確的意圖和實體，則您可以透過標示更多語句或新增功能，以提升 LUIS 應用程式的效能。 

* [使用 LUIS 標示建議的語調](luis-how-to-review-endoint-utt.md) 
* [使用功能來改善 LUIS 應用程式效能](luis-how-to-add-features.md) 
* [透過此教學課程來了解批次測試](luis-tutorial-batch-testing.md)
* [了解批次測試概念](luis-concept-batch-test.md).
