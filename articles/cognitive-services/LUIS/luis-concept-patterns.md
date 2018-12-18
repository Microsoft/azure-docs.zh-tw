---
title: 了解模式如何提高預測精確度
titleSuffix: Azure Cognitive Services
description: 模式設計用來改善數個語句非常類似時的精確度。 模式可讓您取得更精確的意圖，而不需提供更多的語句。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 5ade15b3f80d725af4ece31a36ea0b670f5f5147
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47031538"
---
# <a name="patterns-improve-prediction-accuracy"></a>模式可改善預測精確度
模式設計用來改善數個語句非常類似時的精確度。  模式可讓您取得更精確的意圖，而不需提供更多的語句。 

## <a name="patterns-solve-low-intent-confidence"></a>模式可解決低意圖信賴度
請考慮使用人力資源應用程式，該應用程式會報告與員工相關的組織圖。 若指定員工的名稱和關係，LUIS 就會傳回涉及的員工。 請考慮使用員工 Tom，其經理名字為 Alice，且其下小組名字為：Michael、Rebecca 和 Carl。

![組織圖的影像](./media/luis-concept-patterns/org-chart.png)

|表達方式|預測的意圖|意圖分數|
|--|--|--|
|Who is Tom's subordinate?|GetOrgChart|.30|
|Who is the subordinate of Tom?|GetOrgChart|.30|

如果應用程式含有的 10 到 20 個語句具有不同的句子長度、不同的字組順序，甚至不同的字組 ("subordinate"、"manage"、"report" 的同義字)，LUIS 可能會傳回很低的信賴分數。 為協助 LUIS 了解字組順序的重要性，請建立模式。 

模式可解決下列狀況： 

* 當意圖分數很低時
* 當正確的意圖不是最高分數，但太接近最高分數時。 

## <a name="patterns-are-not-a-guarantee-of-intent"></a>模式不是意圖保證
模式會混合使用多個預測技術。 在模式中設定範本語句的意圖並不是意圖預測的保證，但卻是很強的示意訊號。 

## <a name="patterns-do-not-improve-entity-detection"></a>模式不會改善實體偵測
雖然模式需要實體時，但模式無助於實體偵測。 模式僅用於協助預測意圖和角色。  

## <a name="patterns-use-entity-roles"></a>模式可使用實體角色
如果模式中有兩個以上的實體與內容相關，則模式會使用實體[角色](luis-concept-roles.md)來擷取關於實體的內容資訊。 這相當於階層式實體子項目，但**僅**適用於模式。 

## <a name="prediction-scores-with-and-without-patterns"></a>使用和不使用模式的預測分數
假設具有足夠的範例語句，LUIS 就能夠在不使用模式的情況下提高預測信賴度。 模式可提高信賴分數，而不需要提供許多語句。  

## <a name="pattern-matching"></a>模式比對
模式比對會先偵測模式內的實體，然後驗證其餘字組和模式的字組順序。 模式中必須要有實體，才能比對模式。 

## <a name="pattern-syntax"></a>模式語法
模式語法是語句的範本。 此範本應該包含您要比對的字組和實體，以及您要忽略的字組和標點符號。 它**不是**一個規則運算式。 

模式中的實體是用大括弧 `{}` 括住。 模式可以包含實體，以及具有角色的實體。 Pattern.any 是僅用於模式的實體。 下列各節將說明此語法。

### <a name="syntax-to-add-an-entity-to-a-pattern-template"></a>將實體新增至模式範本的語法
若要將實體新增至模式範本，請用大括弧括住實體名稱，例如 `Who does {Employee} manage?`。 

|模式與實體|
|--|
|`Who does {Employee} manage?`|

### <a name="syntax-to-add-an-entity-and-role-to-a-pattern-template"></a>將實體和角色新增至模式範本的語法
實體角色以 `{entity:role}` 的形式表示，其格式為實體名稱後接冒號，然後接著角色名稱。 若要將具有角色的實體新增至模式範本，請用大括弧括住實體名稱和角色名稱，例如 `Book a ticket from {Location:Origin} to {Location:Destination}`。 

|模式與實體角色|
|--|
|`Book a ticket from {Location:Origin} to {Location:Destination}`|

### <a name="syntax-to-add-a-patternany-to-pattern-template"></a>將 Pattern.any 新增至模式範本的語法
Pattern.any 實體可讓您將變動長度的實體新增至模式。 只要遵循模式範本，Pattern.any 就可以是任意長度。 

若要將 **Pattern.any** 實體新增至模式範本，請用大括弧括住 Pattern.any 實體，例如 `How much does {Booktitle} cost and what format is it available in?`。  

|模式與 Pattern.any 實體|
|--|
|`How much does {Booktitle} cost and what format is it available in?`|

|模式中的書名|
|--|
|How much does **steal this book** cost and what format is it available in?|
|How much does **ask** cost and what format is it available in?|
|How much does **The Curious Incident of the Dog in the Night-Time** cost and what format is it available in?| 

在這些書名範例中，書名的內容相關字組不會讓 LUIS 感到混淆。 LUIS 知道書名的結束位置，因為它是一個模式，並以 Pattern.any 實體標記。

### <a name="explicit-lists"></a>明確清單
如果您的模式包含 Pattern.any，而且模式語法允許根據語句擷取不正確的實體，請透過編寫 API 來建立[明確清單](https://aka.ms/ExplicitList)，以允許例外狀況。 

例如，假設您有一個模式，其包含選用語法 `[]` 和實體語法 `{}`，且以錯誤擷取資料的方式合併在一起。

請考慮使用模式 `[find] email about {subject} [from {person}]'。 在下列語句中，系統會正確及錯誤地擷取 **subject** 和 **person** 實體：

|語句|實體|正確擷取|
|--|--|:--:|
|email about dogs from Chris|subject=dogs<br>person=Chris|✔|
|email about the man from La Mancha|subject=the man<br>person=La Mancha|X|

在上述資料表的語句 `email about the man from La Mancha` 中，subject 應該是 `the man from La Mancha` (書名)，但因為 subject 包含選用字組 `from`，所以會錯誤地預測書名。 

若要修正模式的這個例外狀況，請使用[撰寫明確清單的 API](https://aka.ms/ExplicitList)，新增 `the man from la mancha` 作為符合 {subject} 實體的明確清單。

### <a name="syntax-to-mark-optional-text-in-a-template-utterance"></a>在範本語句中標記選用文字的語法
您可以使用規則運算式的方括弧語法 `[]`，在語句中標記選用文字。 選用文字最多只能在方括弧中套嵌兩個大括弧。

|模式與選用文字|
|--|
|`[find] email about {subject} [from {person}]`|

`.`、`!` 及 `?` 等標點符號可以使用方括弧加以忽略。 為了忽略這些符號，每個符號都必須位於不同的模式中。 選用語法目前不支援忽略多個項目清單中的項目。

## <a name="patterns-only"></a>僅限模式
LUIS 允許應用程式在意圖中沒有任何範例語句。 只有在使用模式時，才允許此用法。 模式需要在每一個模式中至少有一個實體。 對於僅限模式的應用程式，模式不應該包含機器學習實體，因為它們確實需要範例語句。 

## <a name="best-practices"></a>最佳作法
了解[最佳做法](luis-concept-best-practices.md)。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [了解如何實作本教學課程中的模式](luis-tutorial-pattern.md)