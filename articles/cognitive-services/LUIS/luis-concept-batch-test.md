---
title: LUIS 應用程式批次測試 - Language Understanding
titleSuffix: Azure Cognitive Services
description: 使用批次測試持續調整您的應用程式，改良應用程式及其語言理解能力。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 414862d7ba1956f9449fde3ba063906fae6ef01b
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47041603"
---
# <a name="batch-testing-in-luis"></a>LUIS 的批次測試

批次測試會驗證您[使用中](luis-concept-version.md#active-version)的已訓練模型來評估其預測的精確度。 批次測試可協助您以圖表的型式，檢視目前已訓練模型每個意圖和實體的精確度。 檢閱批次測試結果，採取適當動作以改善精確度，例如，如果您的應用程式經常無法識別正確的意圖，可新增更多意圖的範例語句。

## <a name="group-data-for-batch-test"></a>批次測試的群組資料
重點是用於批次測試的語句，必須是 LUIS 中所沒有的。 如果您有語句的資料集，將這些語句分為三組：加入到意圖的語句、從發佈的端點收到的語句，以及對於已訓練 LUIS 進行批次測試所用的語句。 

## <a name="a-dataset-of-utterances"></a>語句的資料集
提交語句的批次檔 (也稱為*資料集*) 進行批次測試。 資料集是 JSON 格式的檔案，其中包含最多 1000 個標示為**非重複**的語句。 您可以在應用程式中測試最多 10 個資料集。 如果您需要測試更多的資料集，可刪除一個資料集，然後加入一個不同的資料集。

|**規則**|
|--|
|*無重複的語句|
|無任何階層式實體子項目|
|1000 個語句或更少|

*重複項目會被視為完全相同的相符字串項目，而不是先權杖化的相符項目。 

## <a name="entities-allowed-in-batch-tests"></a>批次測試中允許的實體
實體包含簡單、階層式父代及複合。 即使批次檔中沒有對應的實體，這些類型的所有實體還是都會出現在批次測試實體篩選條件中。


<a name="json-file-with-no-duplicates"></a>
<a name="example-batch-file"></a>
## <a name="batch-file-format"></a>批次檔格式
批次檔由語句組成。 每個語句都必須有預期的意圖預測，以及您想要偵測的[機器學習實體](luis-concept-entity-types.md#types-of-entities)。 

以下是使用適當語法的批次檔範例：

```JSON
[
  {
    "text": "Are there any janitorial jobs currently open?",
    "intent": "GetJobInformation",
    "entities": 
    [
        {
            "entity": "Job",
            "startPos": 14,
            "endPos": 23
        }
    ]
  },
  {
    "text": "I would like a fullstack typescript programming with azure job",
    "intent": "GetJobInformation",
    "entities": 
    [
        {
            "entity": "Job",
            "startPos": 15,
            "endPos": 46
        }
    ]
  },
  {
    "text": "Is there a database position open in Los Colinas?",
    "intent": "GetJobInformation",
    "entities": 
    [
        {
            "entity": "Job",
            "startPos": 11,
            "endPos": 18
        }
    ]
  },
  {
    "text": "Please find database jobs open today in Seattle",
    "intent": "GetJobInformation",
    "entities": 
    [
        {
            "entity": "Job",
            "startPos": 12,
            "endPos": 19
        }
    ]
  }
]
```

## <a name="batch-syntax-template"></a>批次語法範本

使用下列範本來啟動您的批次檔：

```JSON
[
  {
    "text": "example utterance goes here",
    "intent": "intent name goes here",
    "entities": 
    [
        {
            "entity": "entity name 1 goes here",
            "startPos": 14,
            "endPos": 23
        },
        {
            "entity": "entity name 2 goes here",
            "startPos": 14,
            "endPos": 23
        }
    ]
  }
]
```

批次檔會使用 **startPos** 和 **endPos** 屬性，來記錄實體的開頭與結尾。 值是以零為起始的，而且不應以空格作為開頭或結尾。 

這與查詢記錄不同，後者使用 startIndex 與 endIndex 屬性。 


## <a name="common-errors-importing-a-batch"></a>匯入批次的常見錯誤
常見錯誤包括： 

> * 超過 1000 個語句
> * 沒有實體屬性的語句 JSON 物件
> * 在多個實體中標示的文字

## <a name="batch-test-state"></a>批次測試狀態
LUIS 會追蹤每個資料集的最後一次測試所呈現的狀態。 這包括大小 (批次中的語句數目)、上次執行日期及最後結果 (成功預測的語句數目)。

<a name="sections-of-the-results-chart"></a>
## <a name="batch-test-results"></a>批次測試結果
批次測試結果會是散佈圖，也稱為錯誤矩陣。 此圖表是對於檔案中的語句以及目前模型的預測意圖和實體進行的 4 向比較。 

**誤判**和**漏判**區段的資料點表示錯誤，應該加以調查。 如果所有資料點是在**真肯定**和**真否定**區段上，則針對這個資料集，您應用程式的精確度是完美的。

![圖表的 4 個區段](./media/luis-concept-batch-test/chart-sections.png)

這個圖表有助於您找到 LUIS 依照其目前訓練內容而錯誤預測的語句。 結果會依區域顯示在圖表上。 選取圖表上各點來檢閱語句資訊，或選取區域名稱檢閱該區域中的語句結果。

![批次測試](./media/luis-concept-batch-test/batch-testing.png)

## <a name="errors-in-the-results"></a>結果中的錯誤
批次測試中顯示錯誤，表示並未預測出如批次檔中所述之意圖。 圖表的兩個紅色區段會顯示錯誤。 

誤判區段顯示語句對應到不應該對應的意圖或實體。 漏判區段顯示語句未對應到應該對應的意圖或實體。 

## <a name="fixing-batch-errors"></a>修正批次錯誤
如果批次測試有錯誤，您可以將更多語句加入到意圖，及/或對於更多語句標示實體，幫助 LUIS 區別不同的意圖。 如果您加入並標示語句，但是測試批次仍然出現預測錯誤，請考慮加入具有特定領域詞彙的[片語清單](luis-concept-feature.md)功能，幫助 LUIS 加速學習。 

## <a name="next-steps"></a>後續步驟

* 了解如何[測試批次](luis-how-to-batch-test.md)
