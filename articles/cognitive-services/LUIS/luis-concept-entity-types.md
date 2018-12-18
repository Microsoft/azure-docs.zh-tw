---
title: LUIS 應用程式中的實體類型 - Language Understanding
titleSuffix: Azure Cognitive Services
description: 在 Language Understanding Intelligent Service (LUIS) 應用程式中新增實體 (您應用程式定義域中的關鍵資料)。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 3ed10ac428b7ce2e528ccf46e34c1d394523bdec
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47042443"
---
# <a name="entities-in-luis"></a>LUIS 中的實體

實體是語句中作為您應用程式定義域關鍵資料的單字或片語。

## <a name="entity-compared-to-intent"></a>實體與意圖的比較
實體代表語句中您想要擷取的單字或片語。 一個語句可以包含許多實體，也可以完全不包含實體。 一個實體代表一個類別，其中包含相似物件 (地點、物件、人員、事件或概念) 的集合。 實體描述與意圖有關的資訊，有時是您應用程式執行其工作的必備要素。 例如，「新聞搜尋」應用程式可能包含「主題」、「來源」、「關鍵字」及「發佈日期」等實體，這些都是搜尋新聞的關鍵資料。 在旅遊預訂應用程式中，「地點」、「日期」、「航空公司」、「艙等」及「機票」則是航班預訂 (與「預訂航班」意圖相關) 的關鍵資訊。

相較之下，意圖代表了整個語句的預測。 

## <a name="entities-represent-data"></a>實體代表資料
實體是您想要從語句中提取的資料。 這可以是名稱、日期、產品名稱或任何單字的群組。 

|語句|實體|資料|
|--|--|--|
|購買 3 張到紐約的機票|預先建置的號碼<br>Location.Destination|3<br>紐約|
|購買 1 張 3 月 5 日從紐約到倫敦的機票|Location.Origin<br>Location.Destination<br>預先建置的 datetimeV2|紐約<br>倫敦<br>2018 年 3 月 5 日|

## <a name="entities-are-optional-but-highly-recommended"></a>實體是選擇性的，但強烈建議使用
意圖是必要的，實體則為選擇性。 您不需要為應用程式中的每個概念建立實體，只需針對應用程式執行動作所需的部分建立實體即可。 

如果您的語句中沒有 Bot 繼續工作所需的詳細資料，就無須新增這些資料。 您可以隨著應用程式趨於成熟，稍後再新增這些資料。 

如果您不確定會如何使用該資訊，可以新增一些常見的預先建置實體，例如 datetimeV2、序數、電子郵件及電話號碼。

## <a name="label-for-word-meaning"></a>單字意義的標籤
如果單字選擇或單字排列相同，但意義不同，請勿以實體標記它。 

下列語句中的 `fair` 一字是一個同形異義字。 其拼字相同，但意義不同：

|語句|
|--|
|What kind of county fairs are happening in the Seattle area this summer?|
|Is the current rating for the Seattle review fair?|

如果您想要讓事件實體尋找所有事件資料，請標記第一個語句中的 `fair` 一字，但不要標記第二個語句中的該字。

## <a name="entities-are-shared-across-intents"></a>時體會跨意圖共用
時體會在意圖間共用。 它們不屬於任何單一意圖。 意圖和實體可以在語意上相關，但並不具有獨佔關聯性。

在「為我預訂到巴黎的機票」語句中，「巴黎」是地點類型的實體。 LUIS 可透過辨識使用者輸入中提到的實體，協助您選擇要執行以滿足某個意圖的特定動作。

## <a name="assign-entities-in-none-intent"></a>指派 None 意圖中的實體
您應該標記所有意圖 (包括 **None** 意圖) 的實體。 這可協助 LUIS 進一步了解實體在語句中的哪裡，以及實體周圍有哪些單字。 

## <a name="types-of-entities"></a>實體類型
LUIS 提供許多類型的實體；預先建置的實體、自訂的機器學習實體及清單實體。

| 名稱 | 可標記 | 說明 |
| -- |--|--|
| **預先建置** <br/>[自訂](#prebuilt)| |  **定義**<br>代表常見概念的內建類型。 <br><br>**清單**<br/>關鍵片語號碼、序數、溫度、維度、金額、年齡、百分比、電子郵件、URL、電話號碼及關鍵片語。 <br><br>預先建置的實體名稱為保留名稱。 <br><br>所有已新增至應用程式的預先建置實體都會在[端點](luis-glossary.md#endpoint)查詢中傳回。 如需詳細資訊，請參閱[預先建置的實體](./luis-prebuilt-entities.md)。 <br/><br/>[實體的範例回應](luis-concept-data-extraction.md#prebuilt-entity-data)|
|<!-- added week of 3/21/08 --> **規則運算式**<br/>[RegEx](#regex)||**定義**<br>已格式化原始語句文字的自訂規則運算式。 這會忽略大小寫並忽略文化特性變體。  <br><br>此實體適用於以任何一致的變化來一致地格式化的單字或片語。<br><br>在拼字檢查修改之後，才會套用規則運算式比對。 <br><br>如果規則運算式太複雜 (例如使用許多方括號)，您便無法將運算式新增到模型中。 <br><br>**範例**<br>`kb[0-9]{6,}` 會比對出 kb123456。<br/><br/>[快速入門](luis-quickstart-intents-regex-entity.md)<br>[實體的範例回應](luis-concept-data-extraction.md)|
| **簡單** <br/>[機器學習](#machine-learned) | ✔ | **定義**<br>簡單實體是描述單一概念並從機器學習內容學習到的一般實體。 內容包括單字選擇、單字位置及語句長度。<br/><br/>這適用於未以一致方式格式化但表示相同事物的單字或片語。 <br/><br/>[快速入門](luis-quickstart-primary-and-secondary-data.md)<br/>[實體的範例回應](luis-concept-data-extraction.md#simple-entity-data)|  
| **清單** <br/>[完全相符](#exact-match)|| **定義**<br>清單實體代表系統中一組固定的封閉式相關單字及其同義字。 <br><br>每個清單實體可以有一或多種形式。 最適用於代表相同概念的一組已知變化。<br/><br/>LUIS 並不會探索清單實體的額外值。 使用**建議**功能，以根據目前的清單查看適用於新字組的建議。<br/><br>如果有多個清單實體具有相同的值，則在端點查詢中會傳回每個實體。 <br/><br/>[快速入門](luis-quickstart-intent-and-list-entity.md)<br>[實體的範例回應](luis-concept-data-extraction.md#list-entity-data)| 
| **Pattern.any** <br/>[混合](#mixed) | ✔|**定義**<br>Patterns.any 是僅用於模式範本語句的可變長度預留位置，用來標記實體開始及結束的位置。  <br><br>**範例**<br>根據標題說出語句來搜尋書籍時，pattern.any 會擷取完整的標題。 使用 pattern.any 的範本語句是 `Who wrote {BookTitle}[?]`。<br/><br/>[教學課程](luis-tutorial-pattern.md)<br>[實體的範例回應](luis-concept-data-extraction.md#composite-entity-data)|  
| **複合** <br/>[機器學習](#machine-learned) | ✔|**定義**<br>複合實體是由預先建置的實體、簡單、RegEx、清單、階層式等其他實體所組成的。 個別實體會構成一個完整的提體。 <br><br>**範例**<br>名為 PlaneTicketOrder 的複合實體可能會有預先建置的 `number` 和 `ToLocation` 子實體。 <br/><br/>[教學課程](luis-tutorial-composite-entity.md)<br>[實體的範例回應](luis-concept-data-extraction.md#composite-entity-data)|  
| **階層式** <br/>[機器學習](#machine-learned) |✔ | **定義**<br>階層式實體是一個依內容學習之簡單實體的分類。<br><br>**範例**<br>假設有一個子系為 `ToLocation` 和 `FromLocation` 的階層式實體 `Location`，便可根據語句內的**內容**來判斷每個子系。 在 `Book 2 tickets from Seattle to New York` 語句中，`ToLocation` 和 `FromLocation` 會根據其周圍的單字而有內容上的不同。 <br/><br/>**不應使用此實體的情況**<br>如果您要尋找不論內容為何只要子系全文相符即可的實體，則應該使用「清單」實體。 如果您要尋找與其他實體類型的父子關係，您應該使用「複合」實體。<br/><br/>[快速入門](luis-quickstart-intent-and-hier-entity.md)<br>[實體的範例回應](luis-concept-data-extraction.md#hierarchical-entity-data)|

<a name="prebuilt"></a>
**預先建置**實體是 LUIS 所提供的自訂實體。 在開放原始碼 [Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) 專案中已定義部分這些實體。 /Specs 目錄中有許多適用於所支援文化特性的[範例](https://github.com/Microsoft/Recognizers-Text/tree/master/Specs)。 如果目前不支援您的特定文化特性或實體，請向專案提出。 

<a name="machine-learned"></a>
**機器學習**實體在透過[端點查詢](luis-concept-test.md#endpoint-testing)及[檢閱端點語句](luis-how-to-review-endoint-utt.md)來進行測試時，效果最佳。 

<a name="regex"></a>
**規則運算式**實體會由使用者在實體定義中所提供的規則運算式來定義。 

<a name="exact-match"></a>
**完全相符**實體會使用實體中所提供的文字來進行全文比對。

<a name="mixed"></a>
**混合**實體會使用實體偵測方法的組合。

## <a name="entity-limits"></a>實體限制
請檢閱[限制](luis-boundaries.md#model-boundaries)，以了解您可以將多少個每一種類型的實體新增到模型中。

## <a name="entity-roles"></a>實體角色
實體[角色](luis-concept-roles.md)僅可在模式中使用。 

## <a name="composite-vs-hierarchical-entities"></a>複合實體與階層式實體的比較
複合實體和階層式實體都有父子關係，且都是機器學習實體。 機器學習可讓 LUIS 根據不同的內容 (單字的排列) 理解實體。 複合實體較具彈性，因為它們允許以不同的實體類型作為子系。 階層實體的子系僅限簡單實體。 

|類型|目的|範例|
|--|--|--|
|階層式|簡單實體的父子|Location.Origin=New York<br>Location.Destination=London|
|複合|父子實體：預先建置、清單、簡單、階層式| number=3<br>list=first class<br>prebuilt.datetimeV2=March 5|

## <a name="data-matching-multiple-entities"></a>與多個實體相符的資料
如果單字或片語與多個實體相符，端點查詢會傳回每個實體。 如果您同時新增預先建置的號碼實體和預先建置的 datetimeV2，且語句為 `create meeting on 2018/03/12 for lunch with wayne`，LUIS 就會辨識所有實體，並在 JSON 端點回應中傳回實體陣列： 

```JSON
{
  "query": "create meeting on 2018/03/12 for lunch with wayne",
  "topScoringIntent": {
    "intent": "Calendar.Add",
    "score": 0.9333419
  },
  "entities": [
    {
      "entity": "2018/03/12",
      "type": "builtin.datetimeV2.date",
      "startIndex": 18,
      "endIndex": 27,
      "resolution": {
        "values": [
          {
            "timex": "2018-03-12",
            "type": "date",
            "value": "2018-03-12"
          }
        ]
      }
    },
    {
      "entity": "2018",
      "type": "builtin.number",
      "startIndex": 18,
      "endIndex": 21,
      "resolution": {
        "value": "2018"
      }
    },
    {
      "entity": "03/12",
      "type": "builtin.number",
      "startIndex": 23,
      "endIndex": 27,
      "resolution": {
        "value": "0.25"
      }
    }
  ]
}
```

## <a name="data-matching-multiple-list-entities"></a>與多個清單實體相符的資料
如果單字或片語與多個清單實體相符，端點查詢會傳回每個清單實體。

如果查詢為 `when is the best time to go to red rock?`，且應用程式在多個清單中有 `red` 一字，LUIS 就會辨識所有實體，並在 JSON 端點回應中傳回實體陣列： 

```JSON
{
  "query": "when is the best time to go to red rock?",
  "topScoringIntent": {
    "intent": "Calendar.Find",
    "score": 0.06701678
  },
  "entities": [
    {
      "entity": "red",
      "type": "Colors",
      "startIndex": 31,
      "endIndex": 33,
      "resolution": {
        "values": [
          "Red"
        ]
      }
    },
    {
      "entity": "red rock",
      "type": "Cities",
      "startIndex": 31,
      "endIndex": 38,
      "resolution": {
        "values": [
          "Destinations"
        ]
      }
    }
  ]
}
```

## <a name="if-you-need-more-than-the-maximum-number-of-entities"></a>如果您所需的實體超出實體數目上限 

您可能需要使用階層式和複合實體。 階層式實體會反映共用特性或具有某個分類成員身分之實體間的關係。 子實體全都是其父系分類的成員。 例如，名為 PlaneTicketClass 的階層式實體可能會有 EconomyClass 和 FirstClass 子實體。 階層所跨的深度只有一層。  

複合實體代表一個整體的多個部分。 例如，名為 PlaneTicketOrder 的複合實體可能會有 Airline、Destination、DepartureCity、DepartureDate 及 PlaneTicketClass 子實體。 您可以從既有的簡單實體、階層式實體的子系或預先建置的實體，建置複合實體。  

LUIS 也提供非機器學習但可讓您的 LUIS 應用程式指定固定值清單的清單實體類型。 請參閱 [LUIS 界限](luis-boundaries.md)參考，以檢閱「清單」實體類型的限制。 

如果您在考慮階層式、複合及清單實體之後，所需的實體仍然超出限制，請與支援人員連絡。 若要這樣做，請收集關於您系統的詳細資訊、前往 [LUIS](luis-reference-regions.md#luis-website) 網站，然後選取 [支援]。 如果您的 Azure 訂用帳戶包含支援服務，請與 [Azure 技術支援人員](https://azure.microsoft.com/support/options/)連絡。 

## <a name="best-practices"></a>最佳作法

當呼叫應用程式或 Bot 需要一些來自執行應用程式所需語句的參數或資料時，請建立[實體](luis-concept-entity-types.md)。 實體是語句中您需要擷取的單字或片語 -- 也許用來作為函式的參數。 

為了選取正確類型的實體來新增至您的應用程式，您必須知道使用者如何輸入該資料。 每一種實體類型的尋找方式各使用不同的機制，例如機器學習、封閉式清單或規則運算式。 如果您不確定要使用哪一種，請從簡單實體開始著手，然後標記所有意圖 (包括 None 意圖) 之所有語句中代表該資料的單字或片語。  

請定期檢閱端點語句，以找出所含實體可被識別為規則運算式或有全文相符的常見用法。  

在檢閱過程中，請考慮針對那些對您定義域而言相當重要、但非完全相符且 LUIS 信賴度不高的單字或片語，新增片語清單以向 LUIS 新增訊號。  

如需詳細資訊，請參閱[最佳做法](luis-concept-best-practices.md)。

## <a name="next-steps"></a>後續步驟

了解良好[語句](luis-concept-utterance.md)的相關概念。 

請參閱[新增實體](luis-how-to-add-entities.md)，以深入了解如何將實體新增至 LUIS 應用程式。