---
title: 工作流程定義語言的函式參考 - Azure Logic Apps | Microsoft Docs
description: 了解 Azure Logic Apps 的工作流程定義語言函式
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
manager: jeconnoc
ms.topic: reference
ms.date: 08/15/2018
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: 8a2e06d2e6cf3e470d4e0909e5559ac0411292fd
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43307108"
---
# <a name="functions-reference-for-workflow-definition-language-in-azure-logic-apps"></a>Azure Logic Apps 中工作流程定義語言的函式參考

[Azure Logic Apps](../logic-apps/logic-apps-overview.md) 中的某些[運算式](../logic-apps/logic-apps-workflow-definition-language.md#expressions)會從邏輯應用程式工作流程定義開始執行時可能還不存在的執行階段動作中取得其值。 若要在運算式中參考或使用這些值，您可以使用[工作流程定義語言](../logic-apps/logic-apps-workflow-definition-language.md)所提供的「函式」。 例如，您可以使用數學函式進行計算，例如 [add()](../logic-apps/workflow-definition-language-functions-reference.md#add) 函式，以傳回整數或浮點數總和。 以下是您可以使用函式執行的另外幾個範例工作：

| Task | 函式語法 | 結果 | 
| ---- | --------------- | ------ | 
| 傳回小寫格式的字串。 | toLower('<*text*>') <p>例如：toLower('Hello') | "hello" | 
| 傳回全域唯一識別碼 (GUID)。 | guid() |"c2ecc88d-88c8-4096-912c-d6f2e2b138ce" | 
|||| 

本文會說明您可以在建立邏輯應用程式定義時使用的函式。
若要[根據其一般目的](#ordered-by-purpose)來尋找函式，請繼續看後面的資料表。 或者，如需每個函式的詳細資訊，請參閱[依字母順序排列的清單](#alphabetical-list)。 

> [!NOTE]
> 在參數定義的語法中，參數後方出現問號 (？) 表示此參數是選擇性的。 例如，請參閱 [getFutureTime()](#getFutureTime)。

## <a name="functions-in-expressions"></a>運算式中的函式

為了說明如何在運算式中使用函式，此範例示範如何藉由在運算式中使用 [parameters()](#parameters) 函式，從 `customerName` 參數取得值並將該值指派給 `accountName` 屬性：

```json
"accountName": "@parameters('customerName')"
```

以下是您可以在運算式中使用函式的一些其他一般方式：

| Task | 運算式中的函式語法 | 
| ---- | -------------------------------- | 
| 將該項目傳遞至函式，以執行項目處理。 | "\@<functionName>(<item>)" | 
| 1.使用巢狀 `parameters()` 函式來取得 parameterName 的值。 </br>2.將該值傳遞至 functionName，以執行結果處理。 | "\@<functionName>(parametersparameterName('<>'))" | 
| 1.從巢狀內部函式 functionName 取得結果。 </br>2.將結果傳遞至外部函式 functionName2。 | "\@<functionName2>(<functionName>(<item>))" | 
| 1.從 functionName 取得結果。 </br>2.假設結果是具有 propertyName 屬性的物件，則會取得該屬性的值。 | "\@<functionName>(<item>).<propertyName>" | 
||| 

例如，`concat()` 函式可以採用兩個或多個字串值作為參數。 此函式會將這些字串合併成一個字串。 您可以傳入字串常值，例如，"Sophia" 和 "Owen"，以便取得合併字串 "SophiaOwen"：

```json
"customerName": "@concat('Sophia', 'Owen')"
```

或者，您可以從參數取得字串值。 這個範例在每個 `concat()` 參數以及 `firstName` 和 `lastName`參數中使用 `parameters()` 函式。 接著，您會將所產生的字串傳遞至 `concat()` 函式，以便取得合併字串，例如 "SophiaOwen"：

```json
"customerName": "@concat(parameters('firstName'), parameters('lastName'))"
```

無論如何，這兩個範例都會將結果指派給 `customerName` 屬性。 

以下是依其一般目的排序的可用函式，或者，您也可以根據[字母順序](#alphabetical-list)瀏覽函式。

<a name="ordered-by-purpose"></a>
<a name="string-functions"></a>

## <a name="string-functions"></a>字串函數

若要處理字串，您可以使用這些字串函式以及一些[集合函式](#collection-functions)。 字串函式只能用於字串。 

| 字串函式 | Task | 
| --------------- | ---- | 
| [concat](../logic-apps/workflow-definition-language-functions-reference.md#concat) | 結合兩個或多個字串，並傳回合併的字串。 | 
| [endsWith](../logic-apps/workflow-definition-language-functions-reference.md#endswith) | 檢查字串是否以指定的子字串結束。 | 
| [guid](../logic-apps/workflow-definition-language-functions-reference.md#guid) | 以字串形式產生全域唯一識別碼 (GUID)。 | 
| [indexOf](../logic-apps/workflow-definition-language-functions-reference.md#indexof) | 傳回子字串的起始位置。 | 
| [lastIndexOf](../logic-apps/workflow-definition-language-functions-reference.md#lastindexof) | 傳回子字串最後一次出現的起始位置。 | 
| [replace](../logic-apps/workflow-definition-language-functions-reference.md#replace) | 使用指定字串取代子字串，並傳回已更新的字串。 | 
| [分割](../logic-apps/workflow-definition-language-functions-reference.md#split) | 傳回具有字串中所有字元的陣列，並以特定分隔符號字元將每個字元隔開。 | 
| [startsWith](../logic-apps/workflow-definition-language-functions-reference.md#startswith) | 檢查字串是否以特定的子字串開始。 | 
| [substring](../logic-apps/workflow-definition-language-functions-reference.md#substring) | 傳回字串中的字元 (從指定的位置起始)。 | 
| [toLower](../logic-apps/workflow-definition-language-functions-reference.md#toLower) | 傳回小寫格式的字串。 | 
| [toUpper](../logic-apps/workflow-definition-language-functions-reference.md#toUpper) | 傳回大寫格式的字串。 | 
| [修剪](../logic-apps/workflow-definition-language-functions-reference.md#trim) | 移除字串的開頭和尾端空白字元，並傳回更新後的字串。 | 
||| 

<a name="collection-functions"></a>

## <a name="collection-functions"></a>集合函式

若要處理集合 (通常為陣列、字串，而有時候為字典)，您可以使用這些集合函式。 

| 集合函式 | Task | 
| ------------------- | ---- | 
| [contains](../logic-apps/workflow-definition-language-functions-reference.md#contains) | 檢查集合是否具有特定項目。 |
| [empty](../logic-apps/workflow-definition-language-functions-reference.md#empty) | 檢查集合是否是空的。 | 
| [first](../logic-apps/workflow-definition-language-functions-reference.md#first) | 傳回集合中的第一個項目。 | 
| [intersection](../logic-apps/workflow-definition-language-functions-reference.md#intersection) | 在指定的多個集合中，傳回「只有」共同項目的集合。 | 
| [join](../logic-apps/workflow-definition-language-functions-reference.md#join) | 傳回具有陣列中「所有」項目 (以指定的字元隔開) 的字串。 | 
| [last](../logic-apps/workflow-definition-language-functions-reference.md#last) | 傳回集合中的最後一個項目。 | 
| [length](../logic-apps/workflow-definition-language-functions-reference.md#length) | 傳回字串或陣列中的項目數目。 | 
| [skip](../logic-apps/workflow-definition-language-functions-reference.md#skip) | 移除集合前端的項目，並傳回「其他所有」項目。 | 
| [take](../logic-apps/workflow-definition-language-functions-reference.md#take) | 傳回集合中的前端項目。 | 
| [union](../logic-apps/workflow-definition-language-functions-reference.md#union) | 傳回具有指定集合中「所有」項目的集合。 | 
||| 

<a name="comparison-functions"></a>

## <a name="logical-comparison-functions"></a>邏輯比較函式

若要處理條件、比較值和運算式結果，或評估各種不同的邏輯，您可以使用下列邏輯比較函式。 如需有關每個函式的完整參考，請參閱[依字母順序排列的清單](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list)。

| 邏輯比較函式 | Task | 
| --------------------------- | ---- | 
| [and](../logic-apps/workflow-definition-language-functions-reference.md#and) | 檢查是否所有運算式都是 True。 | 
| [equals](../logic-apps/workflow-definition-language-functions-reference.md#equals) | 檢查兩個值是否相等。 | 
| [greater](../logic-apps/workflow-definition-language-functions-reference.md#greater) | 檢查第一個值是否大於第二個值。 | 
| [greaterOrEquals](../logic-apps/workflow-definition-language-functions-reference.md#greaterOrEquals) | 檢查第一個值是否大於或等於第二個值。 | 
| [if](../logic-apps/workflow-definition-language-functions-reference.md#if) | 檢查運算式是 True 或 False。 根據結果，傳回指定的值。 | 
| [less](../logic-apps/workflow-definition-language-functions-reference.md#less) | 檢查第一個值是否小於第二個值。 | 
| [lessOrEquals](../logic-apps/workflow-definition-language-functions-reference.md#lessOrEquals) | 檢查第一個值是否小於或等於第二個值。 | 
| [not](../logic-apps/workflow-definition-language-functions-reference.md#not) | 檢查運算式是否為 False。 | 
| [or](../logic-apps/workflow-definition-language-functions-reference.md#or) | 檢查是否至少有一個運算式是 True。 |
||| 

<a name="conversion-functions"></a>

## <a name="conversion-functions"></a>轉換函式

若要變更值的類型或格式，您可以使用這些轉換函式。 例如，您可以將一個值從布林值變更為整數。 若要深入了解 Logic Apps 在轉換期間如何處理內容類型，請參閱[處理內容類型](../logic-apps/logic-apps-content-type.md)。 如需有關每個函式的完整參考，請參閱[依字母順序排列的清單](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list)。

| 轉換函式 | Task | 
| ------------------- | ---- | 
| [array](../logic-apps/workflow-definition-language-functions-reference.md#array) | 從單一指定輸入傳回的陣列。 關於多個輸入的資訊，請參閱 [createArray](../logic-apps/workflow-definition-language-functions-reference.md#createArray)。 | 
| [base64](../logic-apps/workflow-definition-language-functions-reference.md#base64) | 傳回字串的 base64 編碼版本。 | 
| [base64ToBinary](../logic-apps/workflow-definition-language-functions-reference.md#base64ToBinary) | 傳回 base64 編碼字串的二進位版本。 | 
| [base64ToString](../logic-apps/workflow-definition-language-functions-reference.md#base64ToString) | 傳回 base64 編碼字串的字串版本。 | 
| [binary](../logic-apps/workflow-definition-language-functions-reference.md#binary) | 傳回輸入值的二進位版本。 | 
| [bool](../logic-apps/workflow-definition-language-functions-reference.md#bool) | 傳回輸入值的布林值版本。 | 
| [createArray](../logic-apps/workflow-definition-language-functions-reference.md#createArray) | 從多個輸入傳回陣列。 | 
| [dataUri](../logic-apps/workflow-definition-language-functions-reference.md#dataUri) | 傳回輸入值的資料 URI。 | 
| [dataUriToBinary](../logic-apps/workflow-definition-language-functions-reference.md#dataUriToBinary) | 傳回資料 URI 的二進位版本。 | 
| [dataUriToString](../logic-apps/workflow-definition-language-functions-reference.md#dataUriToString) | 傳回資料 URI 的字串版本。 | 
| [decodeBase64](../logic-apps/workflow-definition-language-functions-reference.md#decodeBase64) | 傳回 base64 編碼字串的字串版本。 | 
| [decodeDataUri](../logic-apps/workflow-definition-language-functions-reference.md#decodeDataUri) | 傳回資料 URI 的二進位版本。 | 
| [decodeUriComponent](../logic-apps/workflow-definition-language-functions-reference.md#decodeUriComponent) | 傳回以已解碼版本取代逸出字元的字串。 | 
| [encodeUriComponent](../logic-apps/workflow-definition-language-functions-reference.md#encodeUriComponent) | 傳回以逸出字元取代 URL 中 Unsafe 字元的字串。 | 
| [float](../logic-apps/workflow-definition-language-functions-reference.md#float) | 傳回輸入值的浮點數。 | 
| [int](../logic-apps/workflow-definition-language-functions-reference.md#int) | 傳回字串的整數版本。 | 
| [json](../logic-apps/workflow-definition-language-functions-reference.md#json) | 傳回字串或 XML 的 JavaScript 物件標記法 (JSON) 類型值或物件。 | 
| [字串](../logic-apps/workflow-definition-language-functions-reference.md#string) | 傳回輸入值的字串版本。 | 
| [uriComponent](../logic-apps/workflow-definition-language-functions-reference.md#uriComponent) | 藉由以逸出字元取代 URL 中的 Unsafe 字元，傳回輸入值的 URI 編碼版本。 | 
| [uriComponentToBinary](../logic-apps/workflow-definition-language-functions-reference.md#uriComponentToBinary) | 傳回 URI 編碼字串的二進位版本。 | 
| [uriComponentToString](../logic-apps/workflow-definition-language-functions-reference.md#uriComponentToString) | 傳回 URI 編碼字串的字串版本。 | 
| [xml](../logic-apps/workflow-definition-language-functions-reference.md#xml) | 傳回字串的 SML 版本。 | 
||| 

<a name="math-functions"></a>

## <a name="math-functions"></a>數學函式

若要處理整數和浮點數，您可以使用這些數學函式。 如需有關每個函式的完整參考，請參閱[依字母順序排列的清單](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list)。

| 數學函式 | Task | 
| ------------- | ---- | 
| [新增](../logic-apps/workflow-definition-language-functions-reference.md#add) | 傳回兩個數字相加的結果。 | 
| [div](../logic-apps/workflow-definition-language-functions-reference.md#div) | 傳回兩個數字相除的結果。 | 
| [max](../logic-apps/workflow-definition-language-functions-reference.md#max) | 從數字集合或陣列中傳回最大值。 | 
| [min](../logic-apps/workflow-definition-language-functions-reference.md#min) | 從數字集合或陣列中傳回最小值。 | 
| [mod](../logic-apps/workflow-definition-language-functions-reference.md#mod) | 傳回兩數相除的餘數。 | 
| [mul](../logic-apps/workflow-definition-language-functions-reference.md#mul) | 傳回將兩數相乘的乘積。 | 
| [rand](../logic-apps/workflow-definition-language-functions-reference.md#rand) | 從指定範圍傳回隨機整數。 | 
| [range](../logic-apps/workflow-definition-language-functions-reference.md#range) | 傳回從指定整數開始的整數陣列。 | 
| [sub](../logic-apps/workflow-definition-language-functions-reference.md#sub) | 傳回第一個數字減去第二個數字的結果。 | 
||| 

<a name="date-time-functions"></a>

## <a name="date-and-time-functions"></a>日期和時間函式

若要處理日期和時間，您可以使用這些日期和時間函式。
如需有關每個函式的完整參考，請參閱[依字母順序排列的清單](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list)。

| 日期或時間函式 | Task | 
| --------------------- | ---- | 
| [addDays](../logic-apps/workflow-definition-language-functions-reference.md#addDays) | 將天數加入時間戳記。 | 
| [addHours](../logic-apps/workflow-definition-language-functions-reference.md#addHours) | 將時數加入時間戳記。 | 
| [addMinutes](../logic-apps/workflow-definition-language-functions-reference.md#addMinutes) | 將分鐘數加入時間戳記。 | 
| [addSeconds](../logic-apps/workflow-definition-language-functions-reference.md#addSeconds) | 將秒數加入時間戳記。 |  
| [addToTime](../logic-apps/workflow-definition-language-functions-reference.md#addToTime) | 將時間單位數字加入時間戳記。 另請參閱 [getFutureTime](../logic-apps/workflow-definition-language-functions-reference.md#getFutureTime)。 | 
| [convertFromUtc](../logic-apps/workflow-definition-language-functions-reference.md#convertFromUtc) | 將時間戳記從國際標準時間 (UTC) 轉換為目標時區。 | 
| [convertTimeZone](../logic-apps/workflow-definition-language-functions-reference.md#convertTimeZone) | 將時間戳記從來源時區轉換為目標時區。 | 
| [convertToUtc](../logic-apps/workflow-definition-language-functions-reference.md#convertToUtc) | 將時間戳記從來源時區轉換為國際標準時間 (UTC)。 | 
| [dayOfMonth](../logic-apps/workflow-definition-language-functions-reference.md#dayOfMonth) | 傳回時間戳記中的當月日期元件。 | 
| [dayOfWeek](../logic-apps/workflow-definition-language-functions-reference.md#dayOfWeek) | 傳回時間戳記中的星期幾元件。 | 
| [dayOfYear](../logic-apps/workflow-definition-language-functions-reference.md#dayOfYear) | 傳回時間戳記中一年的第幾天元件。 | 
| [formatDateTime](../logic-apps/workflow-definition-language-functions-reference.md#formatDateTime) | 傳回時間戳記中的日期。 | 
| [getFutureTime](../logic-apps/workflow-definition-language-functions-reference.md#getFutureTime) | 傳回目前時間戳記加上指定時間單位的結果。 另請參閱 [addToTime](../logic-apps/workflow-definition-language-functions-reference.md#addToTime)。 | 
| [getPastTime](../logic-apps/workflow-definition-language-functions-reference.md#getPastTime) | 傳回目前時間戳記減去指定時間單位的結果。 另請參閱 [subtractFromTime](../logic-apps/workflow-definition-language-functions-reference.md#subtractFromTime)。 | 
| [startOfDay](../logic-apps/workflow-definition-language-functions-reference.md#startOfDay) | 傳回時間戳記中當天的起始點。 | 
| [startOfHour](../logic-apps/workflow-definition-language-functions-reference.md#startOfHour) | 傳回時間戳記中小時的起始點。 | 
| [startOfMonth](../logic-apps/workflow-definition-language-functions-reference.md#startOfMonth) | 傳回時間戳記中月份的起始點。 | 
| [subtractFromTime](../logic-apps/workflow-definition-language-functions-reference.md#subtractFromTime) | 從時間戳記減去時間單位數字。 另請參閱 [getPastTime](../logic-apps/workflow-definition-language-functions-reference.md#getPastTime)。 | 
| [ticks](../logic-apps/workflow-definition-language-functions-reference.md#ticks) | 傳回指定時間戳記的 `ticks` 屬性值。 | 
| [utcNow](../logic-apps/workflow-definition-language-functions-reference.md#utcNow) | 傳回目前的時間戳記作為字串。 | 
||| 

<a name="workflow-functions"></a>

## <a name="workflow-functions"></a>工作流程函式

這些工作流程函式可協助您：

* 在執行階段取得有關工作流程執行個體的詳細資料。 
* 處理用於具現化邏輯應用程式的輸入。
* 參考來自觸發程序和動作的輸出。

例如，您可以參考來自一個動作的輸出，並在稍後動作中使用該資料。 如需有關每個函式的完整參考，請參閱[依字母順序排列的清單](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list)。

| 工作流程函式 | Task | 
| ----------------- | ---- | 
| [action](../logic-apps/workflow-definition-language-functions-reference.md#action) | 傳回目前動作在執行階段的輸出，或來自其他 JSON 名稱與值配對中的值。 另請參閱 [actions](../logic-apps/workflow-definition-language-functions-reference.md#actions)。 | 
| [actionBody](../logic-apps/workflow-definition-language-functions-reference.md#actionBody) | 傳回動作在執行階段的 `body` 輸出。 另請參閱 [body](../logic-apps/workflow-definition-language-functions-reference.md#body)。 | 
| [actionOutputs](../logic-apps/workflow-definition-language-functions-reference.md#actionOutputs) | 傳回動作在執行階段的輸出。 請參閱 [actions](../logic-apps/workflow-definition-language-functions-reference.md#actions)。 | 
| [actions](../logic-apps/workflow-definition-language-functions-reference.md#actions) | 傳回動作在執行階段的輸出，或來自其他 JSON 名稱與值配對中的值。 另請參閱 [action](../logic-apps/workflow-definition-language-functions-reference.md#action)。  | 
| [body](#body) | 傳回動作在執行階段的 `body` 輸出。 另請參閱 [actionBody](../logic-apps/workflow-definition-language-functions-reference.md#actionBody)。 | 
| [formDataMultiValues](../logic-apps/workflow-definition-language-functions-reference.md#formDataMultiValues) | 建立帶有值的陣列，這些值會符合「表單資料」或「表單編碼」動作輸出中的索引鍵名稱。 | 
| [formDataValue](../logic-apps/workflow-definition-language-functions-reference.md#formDataValue) | 傳回單一值，此值會符合動作「表單資料」或「表單編碼」輸出的索引鍵名稱。 | 
| [item](../logic-apps/workflow-definition-language-functions-reference.md#item) | 若用於透過陣列進行的重複動作中，則會在動作的目前反覆運算期間，傳回陣列中的目前項目。 | 
| [items](../logic-apps/workflow-definition-language-functions-reference.md#items) | 若用於 for-each 或 do-until-loop 內，則會從指定的迴圈傳回目前的項目。| 
| [listCallbackUrl](../logic-apps/workflow-definition-language-functions-reference.md#listCallbackUrl) | 傳回呼叫觸發程序或動作的「回呼 URL」。 | 
| [multipartBody](../logic-apps/workflow-definition-language-functions-reference.md#multipartBody) | 在具有多個部分的動作輸出中，傳回特定部分的內容。 | 
| [參數](../logic-apps/workflow-definition-language-functions-reference.md#parameters) | 傳回邏輯應用程式定義中描述的參數值。 | 
| [trigger](../logic-apps/workflow-definition-language-functions-reference.md#trigger) | 傳回觸發程序在執行階段的輸出，或來自其他 JSON 名稱與值配對中的值。 另請參閱 [triggerOutputs](#triggerOutputs) 和 [triggerBody](../logic-apps/workflow-definition-language-functions-reference.md#triggerBody)。 | 
| [triggerBody](../logic-apps/workflow-definition-language-functions-reference.md#triggerBody) | 傳回觸發程序在執行階段的 `body` 輸出。 請參閱 [trigger](../logic-apps/workflow-definition-language-functions-reference.md#trigger)。 | 
| [triggerFormDataValue](../logic-apps/workflow-definition-language-functions-reference.md#triggerFormDataValue) | 在「表單資料」或「表單編碼」觸發程序輸出中，傳回符合索引鍵名稱的單一值。 | 
| [triggerMultipartBody](../logic-apps/workflow-definition-language-functions-reference.md#triggerMultipartBody) | 在觸發程序的多部分輸出中，傳回特定部分的內容。 | 
| [triggerFormDataMultiValues](../logic-apps/workflow-definition-language-functions-reference.md#triggerFormDataMultiValues) | 建立帶有值的陣列，這些值會符合「表單資料」或「表單編碼」觸發程序輸出中的索引鍵名稱。 | 
| [triggerOutputs](../logic-apps/workflow-definition-language-functions-reference.md#triggerOutputs) | 傳回觸發程序在執行階段的輸出，或來自其他 JSON 名稱與值配對中的值。 請參閱 [trigger](../logic-apps/workflow-definition-language-functions-reference.md#trigger)。 | 
| [變數](../logic-apps/workflow-definition-language-functions-reference.md#variables) | 傳回指定變數的值。 | 
| [workflow](../logic-apps/workflow-definition-language-functions-reference.md#workflow) | 傳回執行階段期間與工作流程本身相關的所有詳細資料。 | 
||| 

<a name="uri-parsing-functions"></a>

## <a name="uri-parsing-functions"></a>URI 剖析函式

若要處理統一資源識別項 (URI) 並取得這些 URI 的各種屬性值，您可以使用這些 URI 剖析函式。 如需有關每個函式的完整參考，請參閱[依字母順序排列的清單](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list)。

| URI 剖析函式 | Task | 
| -------------------- | ---- | 
| [uriHost](../logic-apps/workflow-definition-language-functions-reference.md#uriHost) | 傳回統一資源識別項 (URI) 的 `host` 值。 | 
| [uriPath](../logic-apps/workflow-definition-language-functions-reference.md#uriPath) | 傳回統一資源識別項 (URI) 的 `path` 值。 | 
| [uriPathAndQuery](../logic-apps/workflow-definition-language-functions-reference.md#uriPathAndQuery) | 傳回統一資源識別項 (URI) 的 `path` 和 `query` 值。 | 
| [uriPort](../logic-apps/workflow-definition-language-functions-reference.md#uriPort) | 傳回統一資源識別項 (URI) 的 `port` 值。 | 
| [uriQuery](../logic-apps/workflow-definition-language-functions-reference.md#uriQuery) | 傳回統一資源識別項 (URI) 的 `query` 值。 | 
| [uriScheme](../logic-apps/workflow-definition-language-functions-reference.md#uriScheme) | 傳回統一資源識別項 (URI) 的 `scheme` 值。 | 
||| 

<a name="manipulation-functions"></a>

## <a name="manipulation-functions-json--xml"></a>操作函式：JSON 和 XML

若要處理 JSON 物件和 XML 節點，您可以使用這些操作函式。 如需有關每個函式的完整參考，請參閱[依字母順序排列的清單](../logic-apps/workflow-definition-language-functions-reference.md#alphabetical-list)。

| 操作函式 | Task | 
| --------------------- | ---- | 
| [addProperty](../logic-apps/workflow-definition-language-functions-reference.md#addProperty) | 將屬性和其值或成對的名稱和數值新增至 JSON 物件，並傳回更新的物件。 | 
| [coalesce](../logic-apps/workflow-definition-language-functions-reference.md#coalesce) | 從一個或多個參數中傳回第一個非 Null 值。 | 
| [removeProperty](../logic-apps/workflow-definition-language-functions-reference.md#removeProperty) | 從 JSON 物件中移除屬性，並傳回更新的物件。 | 
| [setProperty](../logic-apps/workflow-definition-language-functions-reference.md#setProperty) | 設定 JSON 物件屬性的值，並傳回更新的物件。 | 
| [xpath](../logic-apps/workflow-definition-language-functions-reference.md#xpath) | 檢查 XML 中是否有符合 XPath (XML 路徑語言) 運算式的節點或值，並傳回符合的節點或值。 | 
||| 

<a name="alphabetical-list"></a>
<a name="action"></a>

### <a name="action"></a>動作

傳回執行階段上「目前」動作的輸出，或來自其他 JSON 名稱-值配對的值，而您可以將其指派給運算式。 根據預設，此函式會參考整個動作物件，但是您可以選擇性地指定其中有所需值的屬性。 另請參閱 [actions()](../logic-apps/workflow-definition-language-functions-reference.md#actions)。

您只能在下列位置使用 `action()` 函式： 

* Webhook 動作的 `unsubscribe` 屬性，讓您可以存取來自原始 `subscribe` 要求的結果
* 動作的 `trackedProperties` 屬性
* 動作的 `do-until` 迴圈條件

```
action()
action().outputs.body.<property> 
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*property*> | 否 | 字串 | 您需要其值的動作物件屬性名稱：**name**、**startTime** **endTime** **inputs** **outputs** **status** **code** **trackingId** 和 **clientTrackingId**。 在 Azure 入口網站中，您可以藉由檢閱特定執行歷程記錄的詳細資料來找出這些屬性。 如需詳細資訊，請參閱 [REST API - 工作流程執行動作](https://docs.microsoft.com/rest/api/logic/workflowrunactions/get)。 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | -----| ----------- | 
| <*action-output*> | 字串 | 目前動作或屬性的輸出 | 
|||| 

<a name="actionBody"></a>

### <a name="actionbody"></a>actionBody

傳回執行階段上動作的 `body` 輸出。 `actions('<actionName>').outputs.body` 的簡略版。 請參閱 [body()](#body) 和 [actions()](#actions)。

```
actionBody('<actionName>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*actionName*> | 是 | 字串 | 所需動作的 `body` 輸出名稱 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | -----| ----------- | 
| <*action-body-output*> | 字串 | 來自指定動作的 `body` 輸出 | 
|||| 

*範例*

此範例會從 Twitter 動作 `Get user` 取得 `body` 輸出： 

```
actionBody('Get_user')
```

並傳回此結果：

```json
"body": {
  "FullName": "Contoso Corporation",
  "Location": "Generic Town, USA",
  "Id": 283541717,
  "UserName": "ContosoInc",
  "FollowersCount": 172,
  "Description": "Leading the way in transforming the digital workplace.",
  "StatusesCount": 93,
  "FriendsCount": 126,
  "FavouritesCount": 46,
  "ProfileImageUrl": "https://pbs.twimg.com/profile_images/908820389907722240/gG9zaHcd_400x400.jpg"
}
```

<a name="actionOutputs"></a>

### <a name="actionoutputs"></a>actionOutputs

傳回動作在執行階段的輸出。 `actions('<actionName>').outputs` 的簡略版。 請參閱 [actions()](#actions)。

```
actionOutputs('<actionName>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*actionName*> | 是 | 字串 | 所需動作輸出的名稱 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | -----| ----------- | 
| <*output*> | 字串 | 來自指定動作的輸出 | 
|||| 

*範例*

此範例會從 Twitter 動作 `Get user` 取得輸出： 

```
actionOutputs('Get_user')
```

並傳回此結果：

```json
{ 
  "statusCode": 200,
  "headers": {
    "Pragma": "no-cache",
    "Vary": "Accept-Encoding",
    "x-ms-request-id": "a916ec8f52211265d98159adde2efe0b",
    "X-Content-Type-Options": "nosniff",
    "Timing-Allow-Origin": "*",
    "Cache-Control": "no-cache",
    "Date": "Mon, 09 Apr 2018 18:47:12 GMT",
    "Set-Cookie": "ARRAffinity=b9400932367ab5e3b6802e3d6158afffb12fcde8666715f5a5fbd4142d0f0b7d;Path=/;HttpOnly;Domain=twitter-wus.azconn-wus.p.azurewebsites.net",
    "X-AspNet-Version": "4.0.30319",
    "X-Powered-By": "ASP.NET",
    "Content-Type": "application/json; charset=utf-8",
    "Expires": "-1",
    "Content-Length": "339"
  },
  "body": {
    "FullName": "Contoso Corporation",
    "Location": "Generic Town, USA",
    "Id": 283541717,
    "UserName": "ContosoInc",
    "FollowersCount": 172,
    "Description": "Leading the way in transforming the digital workplace.",
    "StatusesCount": 93,
    "FriendsCount": 126,
    "FavouritesCount": 46,
    "ProfileImageUrl": "https://pbs.twimg.com/profile_images/908820389907722240/gG9zaHcd_400x400.jpg"
  }
}
```

<a name="actions"></a>

### <a name="actions"></a>動作

傳回執行階段上動作的輸出，或來自其他 JSON 成對名稱和數值中的值，而您可以將其指派給運算式。 根據預設，函式會參考整個動作物件，但是您可以選擇性地指定其中有所需值的屬性。 若要了解簡略版，請參閱 [actionBody()](#actionBody)、[actionOutputs()](#actionOutputs) 和 [body()](#body)。 若要了解目前的動作，請參閱 [action()](#action)。

> [!NOTE] 
> 先前，指定動作以另一個動作的輸出為基礎來執行時，您可以使用 `actions()` 函式或 `conditions` 元素。 但是，若要明確宣告動作之間的相依性，您現在必須使用相依動作的 `runAfter` 屬性。 若要深入了解 `runAfter` 屬性，請參閱[使用 runAfter 屬性來擷取和處理失敗](../logic-apps/logic-apps-workflow-definition-language.md)。

```
actions('<actionName>')
actions('<actionName>').outputs.body.<property> 
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*actionName*> | 是 | 字串 | 您需要其輸出的動作物件名稱  | 
| <*property*> | 否 | 字串 | 您需要其值的動作物件屬性名稱：**name**、**startTime** **endTime** **inputs** **outputs** **status** **code** **trackingId** 和 **clientTrackingId**。 在 Azure 入口網站中，您可以藉由檢閱特定執行歷程記錄的詳細資料來找出這些屬性。 如需詳細資訊，請參閱 [REST API - 工作流程執行動作](https://docs.microsoft.com/rest/api/logic/workflowrunactions/get)。 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | -----| ----------- | 
| <*action-output*> | 字串 | 來自指定動作或屬性的輸出 | 
|||| 

*範例*

此範例會從執行階段上的 Twitter 動作 `Get user` 取得 `status` 屬性值： 

```
actions('Get_user').outputs.body.status 
```

並傳回此結果：`"Succeeded"`

<a name="add"></a>

### <a name="add"></a>新增

傳回兩個數字相加的結果。

```
add(<summand_1>, <summand_2>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*summand_1*>, <*summand_2*> | 是 | 整數、浮點數或混合 | 要相加的數字 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | -----| ----------- | 
| <*result-sum*> | 整數或浮點數 | 指定數字相加的結果 | 
|||| 

*範例*

此範例會相加指定的數字：

```
add(1, 1.5)
```

並傳回此結果：`2.5`

<a name="addDays"></a>

### <a name="adddays"></a>addDays

將天數加入時間戳記。

```
addDays('<timestamp>', <days>, '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*days*> | 是 | 整數  | 要加入的天數 (正數或負數) | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-timestamp*> | 字串 | 時間戳記加上指定的天數  | 
|||| 

*範例 1*

此範例會在指定的時間戳記中加上 10 天：

```
addDays('2018-03-15T13:00:00Z', 10)
```

並傳回此結果：`"2018-03-25T00:00:0000000Z"`

*範例 2*

此範例會從指定的時間戳記中減去五天：

```
addDays('2018-03-15T00:00:00Z', -5)
```

並傳回此結果：`"2018-03-10T00:00:0000000Z"`

<a name="addHours"></a>

### <a name="addhours"></a>addHours

將時數加入時間戳記。

```
addHours('<timestamp>', <hours>, '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*hours*> | 是 | 整數  | 要加入的時數 (正數或負數) | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-timestamp*> | 字串 | 時間戳記加上指定的時數  | 
|||| 

*範例 1*

此範例會在指定的時間戳記中加上 10 小時：

```
addHours('2018-03-15T00:00:00Z', 10)
```

並傳回此結果：`"2018-03-15T10:00:0000000Z"`

*範例 2*

此範例會從指定的時間戳記中減去五小時：

```
addHours('2018-03-15T15:00:00Z', -5)
```

並傳回此結果：`"2018-03-15T10:00:0000000Z"`

<a name="addMinutes"></a>

### <a name="addminutes"></a>addMinutes

將分鐘數加入時間戳記。

```
addMinutes('<timestamp>', <minutes>, '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*minutes*> | 是 | 整數  | 要加入的分鐘數 (正數或負數) | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-timestamp*> | 字串 | 時間戳記加上指定的分鐘數 | 
|||| 

*範例 1*

此範例會在指定的時間戳記中加上 10 分鐘：

```
addMinutes('2018-03-15T00:10:00Z', 10)
```

並傳回此結果：`"2018-03-15T00:20:00.0000000Z"`

*範例 2*

此範例會從指定的時間戳記中減去五分鐘：

```
addMinutes('2018-03-15T00:20:00Z', -5)
```

並傳回此結果：`"2018-03-15T00:15:00.0000000Z"`

<a name="addProperty"></a>

### <a name="addproperty"></a>addProperty

將屬性和其值或成對的名稱和數值新增至 JSON 物件，並傳回更新的物件。 如果物件已存在於執行階段中，函式會擲回錯誤。

```
addProperty(<object>, '<property>', <value>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*object*> | 是 | Object | 您要對其新增屬性的 JSON 物件 | 
| <*property*> | 是 | 字串 | 要新增的屬性名稱 | 
| <*value*> | 是 | 任意 | 屬性的值 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-object*> | Object | 具有指定屬性的更新 JSON 物件 | 
|||| 

*範例*

此範例會將 `accountNumber` 屬性新增至 `customerProfile` 物件，此物件是以 [JSON()](#json) 函式轉換為 JSON。 函式會指派由 [guid()](#guid) 函式產生的值，並傳回更新的物件：

```
addProperty(json('customerProfile'), 'accountNumber', guid())
```

<a name="addSeconds"></a>

### <a name="addseconds"></a>addSeconds

將秒數加入時間戳記。

```
addSeconds('<timestamp>', <seconds>, '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*seconds*> | 是 | 整數  | 要加入的秒數 (正數或負數) | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-timestamp*> | 字串 | 時間戳記加上指定的秒數  | 
|||| 

*範例 1*

此範例會在指定的時間戳記中加上 10 秒鐘：

```
addSeconds('2018-03-15T00:00:00Z', 10)
```

並傳回此結果：`"2018-03-15T00:00:10.0000000Z"`

*範例 2*

此範例會在指定的時間戳記中減去 5 秒鐘：

```
addSeconds('2018-03-15T00:00:30Z', -5)
```

並傳回此結果：`"2018-03-15T00:00:25.0000000Z"`

<a name="addToTime"></a>

### <a name="addtotime"></a>addToTime

將時間單位數字加入時間戳記。 另請參閱 [getFutureTime()](#getFutureTime)。

```
addToTime('<timestamp>', <interval>, '<timeUnit>', '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*interval*> | 是 | 整數  | 要加入的指定時間單位數字 | 
| <*timeUnit*> | 是 | 字串 | 與 *interval* 搭配使用的時間單位："Second"、"Minute"、"Hour"、"Day"、"Week"、"Month"、"Year" | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-timestamp*> | 字串 | 時間戳記加上指定的時間單位數字  | 
|||| 

*範例 1*

此範例會在指定的時間戳記中加上 1 天：

```
addToTime('2018-01-01T00:00:00Z', 1, 'Day') 
```

並傳回此結果：`"2018-01-02T00:00:00:0000000Z"`

*範例 2*

此範例會在指定的時間戳記中加上 1 天：

```
addToTime('2018-01-01T00:00:00Z', 1, 'Day', 'D')
```

並傳回選用 "D" 格式的結果：`"Tuesday, January 2, 2018"`

<a name="and"></a>

### <a name="and"></a>和

檢查是否所有運算式都是 True。 在所有運算式都是 True 時，傳回 True，或至少有一個運算式為 False 時傳回 False。

```
and(<expression1>, <expression2>, ...)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*expression1*>, <*expression2*>, ... | 是 | BOOLEAN | 要檢查的運算式 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | -----| ----------- | 
| True 或 False | BOOLEAN | 所有運算式都是 True 時，即傳回 True。 至少一個運算式為 False 時，則傳回 False。 | 
|||| 

*範例 1*

這些範例會檢查指定的布林值是否都是 True：

```
and(true, true)
and(false, true)
and(false, false)
```

並傳回下列結果：

* 第一個範例：兩個運算式都是 True，所以會傳回 `true`。 
* 第二個範例：一個運算式為 False，所以會傳回 `false`。
* 第一個範例：兩個運算式都是 False，所以會傳回 `false`。

*範例 2*

這些範例會檢查指定的運算式是否都是 True：

```
and(equals(1, 1), equals(2, 2))
and(equals(1, 1), equals(1, 2))
and(equals(1, 2), equals(1, 3))
```

並傳回下列結果：

* 第一個範例：兩個運算式都是 True，所以會傳回 `true`。 
* 第二個範例：一個運算式為 False，所以會傳回 `false`。
* 第一個範例：兩個運算式都是 False，所以會傳回 `false`。

<a name="array"></a>

### <a name="array"></a>array

從單一指定輸入傳回的陣列。 關於多個輸入的資訊，請參閱 [createArray()](#createArray)。 

```
array('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 建立陣列的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| [<*value*>] | 陣列 | 包含單一指定輸入的陣列 | 
|||| 

*範例*

此範例會從 "hello" 字串建立陣列：

```
array('hello')
```

並傳回此結果：`["hello"]`

<a name="base64"></a>

### <a name="base64"></a>base64

傳回字串的 base64 編碼版本。

```
base64('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 輸入字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*base64-string*> | 字串 | 輸入字串的 base64 編碼版本 | 
|||| 

*範例*

此範例會將 "hello" 字串轉換為 base64 編碼的字串：

```
base64('hello')
```

並傳回此結果：`"aGVsbG8="`

<a name="base64ToBinary"></a>

### <a name="base64tobinary"></a>base64ToBinary

傳回 base64 編碼字串的二進位版本。

```
base64ToBinary('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要轉換的 base64 編碼字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*binary-for-base64-string*> | 字串 | base64 編碼字串的二進位版本 | 
|||| 

*範例*

此範例會將 "aGVsbG8=" base64 編碼的字串轉換為二進位字串：

```
base64ToBinary('aGVsbG8=')
```

並傳回此結果： 

`"0110000101000111010101100111001101100010010001110011100000111101"`

<a name="base64ToString"></a>

### <a name="base64tostring"></a>base64ToString

傳回 base64 編碼字串的字串版本，也就是有效地解碼 base64 字串。 使用此函式而非 [decodeBase64()](#decodeBase64)。 雖然這兩個函數的運作方式相同，但是較常使用 `base64ToString()`。

```
base64ToString('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要解碼的 base64 編碼字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*decoded-base64-string*> | 字串 | 傳回 base64 編碼字串的字串版本 | 
|||| 

*範例*

此範例會將 "aGVsbG8=" base64 編碼的字串轉換為單純字串：

```
base64ToString('aGVsbG8=')
```

並傳回此結果：`"hello"`

<a name="binary"></a>

### <a name="binary"></a>binary 

傳回字串的二進位版本。

```
binary('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要轉換的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*binary-for-input-value*> | 字串 | 指定字串的二進位版本 | 
|||| 

*範例*

此範例會將 "hello" 字串轉換為二進位字串：

```
binary('hello')
```

並傳回此結果： 

`"0110100001100101011011000110110001101111"`

<a name="body"></a>

### <a name="body"></a>body

傳回動作在執行階段的 `body` 輸出。 `actions('<actionName>').outputs.body` 的簡略版。 請參閱 [actionBody()](#actionBody) 和 [actions()](#actions)。

```
body('<actionName>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*actionName*> | 是 | 字串 | 所需動作的 `body` 輸出名稱 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | -----| ----------- | 
| <*action-body-output*> | 字串 | 來自指定動作的 `body` 輸出 | 
|||| 

*範例*

此範例會從 `body` Twitter 動作取得 `Get user` 輸出： 

```
body('Get_user')
```

並傳回此結果： 

```json
"body": {
    "FullName": "Contoso Corporation",
    "Location": "Generic Town, USA",
    "Id": 283541717,
    "UserName": "ContosoInc",
    "FollowersCount": 172,
    "Description": "Leading the way in transforming the digital workplace.",
    "StatusesCount": 93,
    "FriendsCount": 126,
    "FavouritesCount": 46,
    "ProfileImageUrl": "https://pbs.twimg.com/profile_images/908820389907722240/gG9zaHcd_400x400.jpg"
}
```

<a name="bool"></a>

### <a name="bool"></a>布林

傳回值的布林值版本。

```
bool(<value>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 任意 | 要轉換的值 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| True 或 False | BOOLEAN | 指定值的布林值版本 | 
|||| 

*範例*

這些範例會將指定的值轉換為布林值： 

```
bool(1)
bool(0)
```

並傳回下列結果： 

* 第一個範例：`true` 
* 第二個範例：`false`

<a name="coalesce"></a>

### <a name="coalesce"></a>coalesce

從一個或多個參數中傳回第一個非 Null 值。 空白字串、空白陣列和空白物件不是 null。

```
coalesce(<object_1>, <object_2>, ...)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*object_1*>, <*object_2*>, ... | 是 | 任何類型，可以是混合類型 | 要檢查是否有 Null 的一個或多個項目 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*first-non-null-item*> | 任意 | 第一個不是 Null 的項目或值。 如果所有參數都是 Null，則此函式會傳回 Null。 | 
|||| 

*範例*

這些範例會從指定值傳回第一個非 Null 的值，或是，如果所有值都是 Null，則傳回 Null：

```
coalesce(null, true, false)
coalesce(null, 'hello', 'world')
coalesce(null, null, null)
```

並傳回下列結果： 

* 第一個範例：`true` 
* 第二個範例：`"hello"`
* 第三個範例：`null`

<a name="concat"></a>

### <a name="concat"></a>concat

結合兩個或多個字串，並傳回合併的字串。 

```
concat('<text1>', '<text2>', ...)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*text1*>, <*text2*>, ... | 是 | 字串 | 要結合的至少兩個字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*text1text2...*> | 字串 | 從合併輸入字串中建立的字串 | 
|||| 

*範例*

此範例會合併 "Hello" 和 "World" 字串：

```
concat('Hello', 'World')
```

並傳回此結果：`"HelloWorld"`

<a name="contains"></a>

### <a name="contains"></a>contains

檢查集合是否具有特定項目。 找到項目時，傳回 True，或找不到項目時，傳回 False。 此函式會區分大小寫。

```
contains('<collection>', '<value>')
contains([<collection>], '<value>')
```

具體而言，此函數會用在這些集合類型上： 

* 要尋找「子字串」的「字串」
* 要尋找「值」的「陣列」
* 要尋找「索引碼」的「字典」

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*collection*> | 是 | 字串、陣列或字典 | 要檢查的集合 | 
| <*value*> | 是 | 個別的字串、陣列或字典 | 要尋找的項目 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| True 或 False | BOOLEAN | 找到項目時，傳回 True。 找不到項目時，傳回 False。 |
|||| 

*範例 1*

此範例會檢查字串 "hello world" 是否有子字串 "world"，並傳回 True：

```
contains('hello world', 'world')
```

*範例 2*

此範例會檢查字串 "hello world" 是否有子字串 "universe"，並傳回 False：

```
contains('hello world', 'universe')
```

<a name="convertFromUtc"></a>

### <a name="convertfromutc"></a>convertFromUtc

將時間戳記從國際標準時間 (UTC) 轉換為目標時區。

```
convertFromUtc('<timestamp>', '<destinationTimeZone>', '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*destinationTimeZone*> | 是 | 字串 | 目標時區的名稱。 如需詳細資訊，請參閱[時區識別碼](https://docs.microsoft.com/previous-versions/windows/embedded/gg154758(v=winembedded.80))。 | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*converted-timestamp*> | 字串 | 轉換為目標時區的時間戳記 | 
|||| 

*範例 1*

此範例會將時間戳記轉換為指定的時區： 

```
convertFromUtc('2018-01-01T08:00:00.0000000Z', 'Pacific Standard Time')
```

並傳回此結果：`"2018-01-01T00:00:00.0000000"`

*範例 2*

此範例會將時間戳記轉換為指定的時區和格式：

```
convertFromUtc('2018-01-01T08:00:00.0000000Z', 'Pacific Standard Time', 'D')
```

並傳回此結果：`"Monday, January 1, 2018"`

<a name="convertTimeZone"></a>

### <a name="converttimezone"></a>convertTimeZone

將時間戳記從來源時區轉換為目標時區。

```
convertTimeZone('<timestamp>', '<sourceTimeZone>', '<destinationTimeZone>', '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*sourceTimeZone*> | 是 | 字串 | 來源時區的名稱。 如需詳細資訊，請參閱[時區識別碼](https://docs.microsoft.com/previous-versions/windows/embedded/gg154758(v=winembedded.80))。 | 
| <*destinationTimeZone*> | 是 | 字串 | 目標時區的名稱。 如需詳細資訊，請參閱[時區識別碼](https://docs.microsoft.com/previous-versions/windows/embedded/gg154758(v=winembedded.80))。 | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*converted-timestamp*> | 字串 | 轉換為目標時區的時間戳記 | 
|||| 

*範例 1*

此範例會將來源時區轉換為目標時區： 

```
convertTimeZone('2018-01-01T08:00:00.0000000Z', 'UTC', 'Pacific Standard Time')
```

並傳回此結果：`"2018-01-01T00:00:00.0000000"`

*範例 2*

此範例會將時區轉換為指定的時區和格式：

```
convertTimeZone('2018-01-01T80:00:00.0000000Z', 'UTC', 'Pacific Standard Time', 'D')
```

並傳回此結果：`"Monday, January 1, 2018"`

<a name="convertToUtc"></a>

### <a name="converttoutc"></a>convertToUtc

將時間戳記從來源時區轉換為國際標準時間 (UTC)。

```
convertToUtc('<timestamp>', '<sourceTimeZone>', '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*sourceTimeZone*> | 是 | 字串 | 來源時區的名稱。 如需詳細資訊，請參閱[時區識別碼](https://docs.microsoft.com/previous-versions/windows/embedded/gg154758(v=winembedded.80))。 | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*converted-timestamp*> | 字串 | 轉換為 UTC 的時間戳記 | 
|||| 

*範例 1*

此範例會將時間戳記轉換為 UTC： 

```
convertToUtc('01/01/2018 00:00:00', 'Pacific Standard Time')
```

並傳回此結果：`"2018-01-01T08:00:00.0000000Z"`

*範例 2*

此範例會將時間戳記轉換為 UTC：

```
convertToUtc('01/01/2018 00:00:00', 'Pacific Standard Time', 'D')
```

並傳回此結果：`"Monday, January 1, 2018"`

<a name="createArray"></a>

### <a name="createarray"></a>createArray

從多個輸入傳回陣列。 如需單一輸入陣列的資訊，請參閱 [array()](#array)。

```
createArray('<object1>', '<object2>', ...)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*object1*>, <*object2*>, ... | 是 | 任何類型，但不能是混合 | 用來建立陣列的至少兩個項目 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| [<*object1*>, <*object2*>, ...] | 陣列 | 從所有輸入項目建立的陣列 | 
|||| 

*範例*

此範例會從以下輸入建立陣列：

```
createArray('h', 'e', 'l', 'l', 'o')
```

並傳回此結果：`["h", "e", "l", "l", "o"]`

<a name="dataUri"></a>

### <a name="datauri"></a>dataUri

傳回字串的資料統一資源識別項 (URI)。 

```
dataUri('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要轉換的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*data-uri*> | 字串 | 輸入字串的資料 URI | 
|||| 

*範例*

此範例會針對 "hello" 字串建立資料 URI：

```
dataUri('hello') 
```

並傳回此結果：`"data:text/plain;charset=utf-8;base64,aGVsbG8="`

<a name="dataUriToBinary"></a>

### <a name="datauritobinary"></a>dataUriToBinary

傳回資料統一資源識別項 (URI) 的二進位版本。 使用此函式而非 [decodeDataUri()](#decodeDataUri)。 雖然這兩個函數的運作方式相同，但是較常使用 `decodeDataUri()`。

```
dataUriToBinary('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要轉換的資料 URI | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*binary-for-data-uri*> | 字串 | 資料 URI 的二進位版本 | 
|||| 

*範例*

此範例會建立此資料 URI 的二進位版本：

```
dataUriToBinary('data:text/plain;charset=utf-8;base64,aGVsbG8=')
```

並傳回此結果： 

`"01100100011000010111010001100001001110100111010001100101011110000111010000101111011100000
1101100011000010110100101101110001110110110001101101000011000010111001001110011011001010111
0100001111010111010101110100011001100010110100111000001110110110001001100001011100110110010
10011011000110100001011000110000101000111010101100111001101100010010001110011100000111101"`

<a name="dataUriToString"></a>

### <a name="datauritostring"></a>dataUriToString

傳回資料統一資源識別項 (URI) 的字串版本。

```
dataUriToString('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要轉換的資料 URI | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*string-for-data-uri*> | 字串 | 資料 URI 的字串版本 | 
|||| 

*範例*

此範例會建立此資料 URI 的字串：

```
dataUriToString('data:text/plain;charset=utf-8;base64,aGVsbG8=')
```

並傳回此結果：`"hello"`

<a name="dayOfMonth"></a>

### <a name="dayofmonth"></a>dayOfMonth

傳回時間戳記中的當月日期。 

```
dayOfMonth('<timestamp>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*day-of-month*> | 整數  | 在指定時間戳記中的當月日期 | 
|||| 

*範例*

此範例會傳回此時間戳記中的當月日期數字：

```
dayOfMonth('2018-03-15T13:27:36Z')
```

並傳回此結果：`15`

<a name="dayOfWeek"></a>

### <a name="dayofweek"></a>dayOfWeek

從時間戳記傳回當週的第幾天。  

```
dayOfWeek('<timestamp>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*day-of-week*> | 整數  | 在指定時間戳記上那一週的第幾天，其中星期日是 0、星期一是 1，依此類推 | 
|||| 

*範例*

此範例會從此時間戳記傳回當週的第幾天：

```
dayOfWeek('2018-03-15T13:27:36Z')
```

並傳回此結果：`3`

<a name="dayOfYear"></a>

### <a name="dayofyear"></a>dayOfYear

從時間戳記傳回一年的第幾天。 

```
dayOfYear('<timestamp>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*day-of-year*> | 整數  | 在指定時間戳記上那一年的第幾天 | 
|||| 

*範例*

此範例會從此時間戳記傳回一年的第幾天：

```
dayOfYear('2018-03-15T13:27:36Z')
```

並傳回此結果：`74`

<a name="decodeBase64"></a>

### <a name="decodebase64"></a>decodeBase64

傳回 base64 編碼字串的字串版本，也就是有效地解碼 base64 字串。 請考慮使用 [base64ToString()](#base64ToString)，而非 `decodeBase64()`。 雖然這兩個函數的運作方式相同，但是較常使用 `base64ToString()`。

```
decodeBase64('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要解碼的 base64 編碼字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*decoded-base64-string*> | 字串 | 傳回 base64 編碼字串的字串版本 | 
|||| 

*範例*

此範例會建立 base64 編碼字串的字串：

```
decodeBase64('aGVsbG8=')
```

並傳回此結果：`"hello"`

<a name="decodeDataUri"></a>

### <a name="decodedatauri"></a>decodeDataUri

傳回資料統一資源識別項 (URI) 的二進位版本。 請考慮使用 [dataUriToBinary()](#dataUriToBinary)，而非 `decodeDataUri()`。 雖然這兩個函數的運作方式相同，但是較常使用 `dataUriToBinary()`。

```
decodeDataUri('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要解碼的資料 URI 字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*binary-for-data-uri*> | 字串 | 資料 URI 字串的二進位版本 | 
|||| 

*範例*

此範例會傳回此資料 URI 的二進位版本：

```
decodeDataUri('data:text/plain;charset=utf-8;base64,aGVsbG8=')
```

並傳回此結果： 

`"01100100011000010111010001100001001110100111010001100101011110000111010000101111011100000
1101100011000010110100101101110001110110110001101101000011000010111001001110011011001010111
0100001111010111010101110100011001100010110100111000001110110110001001100001011100110110010
10011011000110100001011000110000101000111010101100111001101100010010001110011100000111101"`

<a name="decodeUriComponent"></a>

### <a name="decodeuricomponent"></a>decodeUriComponent

傳回以已解碼版本取代逸出字元的字串。 

```
decodeUriComponent('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 其逸出字元需要解碼的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*decoded-uri*> | 字串 | 更新後的字串，其中逸出字元已解碼 | 
|||| 

*範例*

此範例會以已解碼的版本取代此字串中的逸出字元：

```
decodeUriComponent('http%3A%2F%2Fcontoso.com')
```

並傳回此結果：`"https://contoso.com"`

<a name="div"></a>

### <a name="div"></a>div

傳回兩數相除的整數結果。 若要取得餘數，請參閱 [mod()](#mod)。

```
div(<dividend>, <divisor>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*dividend*> | 是 | 整數或浮點數 | 要除以「除數」的數字 | 
| <*divisor*> | 是 | 整數或浮點數 | 要除「被除數」的數字，但不能為 0 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*quotient-result*> | 整數  | 第一個數字除以第二個數字的整數結果 | 
|||| 

*範例*

這兩個範例會將第一個數字除以第二個數字：

```
div(10, 5)
div(11, 5)
```

並傳回此結果：`2`

<a name="encodeUriComponent"></a>

### <a name="encodeuricomponent"></a>encodeUriComponent

傳回字串的統一資源識別項 (URI) 編碼版本，以逸出字元取代 URL 中的 Unsafe 字元。 請考慮使用 [uriComponent()](#uriComponent)，而非 `encodeUriComponent()`。 雖然這兩個函數的運作方式相同，但是較常使用 `uriComponent()`。

```
encodeUriComponent('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要轉換成 URI 編碼格式的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*encoded-uri*> | 字串 | 具有逸出字元的 URI 編碼字串 | 
|||| 

*範例*

此範例會建立此字串的 URI 編碼版本：

```
encodeUriComponent('https://contoso.com')
```

並傳回此結果：`"http%3A%2F%2Fcontoso.com"`

<a name="empty"></a>

### <a name="empty"></a>empty

檢查集合是否是空的。 集合若是空的，傳回 True，或集合若不是空的，則傳回 False。

```
empty('<collection>')
empty([<collection>])
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*collection*> | 是 | 字串、陣列或物件 | 要檢查的集合 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| True 或 False | BOOLEAN | 若集合是空的，傳回 True。 若不是空的，傳回 False。 | 
|||| 

*範例* 

這些範例會檢查指定的集合是否是空的：

```
empty('')
empty('abc')
```

並傳回下列結果： 

* 第一個範例：傳遞空字串，所以函數傳回 `true`。 
* 第二個範例：傳遞 "abc" 字串，所以函數傳回 `false`。 

<a name="endswith"></a>

### <a name="endswith"></a>endsWith

檢查字串是否以特定的子字串結束。 找到子字串時，傳回 True，或找不到子字串時，傳回 False。 此函式不區分大小寫。

```
endsWith('<text>', '<searchText>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*text*> | 是 | 字串 | 要檢查的字串 | 
| <*searchText*> | 是 | 字串 | 要尋找的結尾子字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| True 或 False  | BOOLEAN | 找到結尾子字串時，傳回 True。 找不到項目時，傳回 False。 | 
|||| 

*範例 1* 

此範例會檢查 "hello world" 字串是否以 "world" 字串結尾：

```
endsWith('hello world', 'world')
```

並傳回此結果：`true`

*範例 2*

此範例會檢查 "hello world" 字串是否以 "universe" 字串結尾：

```
endsWith('hello world', 'universe')
```

並傳回此結果：`false`

<a name="equals"></a>

### <a name="equals"></a>equals

檢查兩個值、運算式或物件是否相等。 兩個項目相等時，傳回 True，或兩個項目不相等時，傳回 False。

```
equals('<object1>', '<object2>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*object1*>, <*object2*> | 是 | 各種類型 | 要比較的值、運算式或物件 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| True 或 False | BOOLEAN | 當兩個項目相等時，傳回 True。 當兩個項目不相等時，傳回 False。 | 
|||| 

*範例*

這些範例會檢查指定的輸入是否相等。 

```
equals(true, 1)
equals('abc', 'abcd')
```

並傳回下列結果： 

* 第一個範例：兩個值相等，所以函數傳回 `true`。
* 第二個範例：兩個值不相等，所以函數傳回 `false`。

<a name="first"></a>

### <a name="first"></a>first

傳回字串或陣列中的第一個項目。

```
first('<collection>')
first([<collection>])
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*collection*> | 是 | 字串或陣列 | 要從中尋找第一個項目的集合 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*first-collection-item*> | 任意 | 集合中的第一個項目 | 
|||| 

*範例*

這些範例會尋找以下集合中第一個項目：

```
first('hello')
first([0, 1, 2])
```

並傳回下列結果： 

* 第一個範例：`"h"`
* 第二個範例：`0`

<a name="float"></a>

### <a name="float"></a>float

將浮點數的字串版本轉換為實際浮點數。 您只可在將自訂參數傳遞給應用程式 (例如邏輯應用程式) 時使用這個函式。

```
float('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 具有有效浮點數要轉換的字串 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*float-value*> | Float | 所指定字串的浮點數 | 
|||| 

*範例*

此範例會建立此浮點數的字串版本：

```
float('10.333')
```

並傳回此結果：`10.333`

<a name="formatDateTime"></a>

### <a name="formatdatetime"></a>formatDateTime

傳回指定格式的時間戳記。

```
formatDateTime('<timestamp>', '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*reformatted-timestamp*> | 字串 | 所指定格式的更新時間戳記 | 
|||| 

*範例*

此範例會將時間戳記轉換為指定格式：

```
formatDateTime('03/15/2018 12:00:00', 'yyyy-MM-ddTHH:mm:ss')
```

並傳回此結果：`"2018-03-15T12:00:00"`

<a name="formDataMultiValues"></a>

### <a name="formdatamultivalues"></a>formDataMultiValues

傳回帶有值的陣列，此值會符合動作「表單資料」或「表單編碼」輸出的索引鍵名稱。 

```
formDataMultiValues('<actionName>', '<key>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*actionName*> | 是 | 字串 | 其輸出中有您所需索引鍵值的動作 | 
| <*key*> | 是 | 字串 | 您需要其值的索引鍵名稱 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| [<*array-with-key-values*>] | 陣列 | 此陣列具有符合指定索引鍵的所有值 | 
|||| 

*範例* 

此範例會從「主旨」索引鍵的值建立陣列，而索引鍵值位於指定動作的表單資料或表單編碼輸出中：  

```
formDataMultiValues('Send_an_email', 'Subject')
```

然後傳回陣列中的主旨文字，例如：`["Hello world"]`

<a name="formDataValue"></a>

### <a name="formdatavalue"></a>formDataValue

傳回單一值，此值會符合動作「表單資料」或「表單編碼」輸出的索引鍵名稱。 如果函式找到一個以上的相符項目，函式會擲回錯誤。

```
formDataValue('<actionName>', '<key>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*actionName*> | 是 | 字串 | 其輸出中有您所需索引鍵值的動作 | 
| <*key*> | 是 | 字串 | 您需要其值的索引鍵名稱 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*key-value*> | 字串 | 所指定索引鍵中的值  | 
|||| 

*範例* 

此範例會從「主旨」索引鍵的值建立字串，而索引鍵值位於指定動作的表單資料或表單編碼輸出中：  

```
formDataValue('Send_an_email', 'Subject')
```

然後以字串形式傳回主旨文字，例如：`"Hello world"`

<a name="getFutureTime"></a>

### <a name="getfuturetime"></a>getFutureTime

傳回目前時間戳記加上指定時間單位的結果。

```
getFutureTime(<interval>, <timeUnit>, <format>?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*interval*> | 是 | 整數  | 要減去的指定時間單位數字 | 
| <*timeUnit*> | 是 | 字串 | 與 *interval* 搭配使用的時間單位："Second"、"Minute"、"Hour"、"Day"、"Week"、"Month"、"Year" | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-timestamp*> | 字串 | 目前時間戳記加上指定的時間單位數字 | 
|||| 

*範例 1*

假設目前的時間戳記是 "2018-03-01T00:00:00.0000000Z"。 此範例會在時間戳記中加上 5 天：

```
getFutureTime(5, 'Day')
```

並傳回此結果：`"2018-03-06T00:00:00.0000000Z"`

*範例 2*

假設目前的時間戳記是 "2018-03-01T00:00:00.0000000Z"。 此範例會加上五天，並將結果轉換為 "D" 格式：

```
getFutureTime(5, 'Day', 'D')
```

並傳回此結果：`"Tuesday, March 6, 2018"`

<a name="getPastTime"></a>

### <a name="getpasttime"></a>getPastTime

傳回目前時間戳記減去指定時間單位的結果。

```
getPastTime(<interval>, <timeUnit>, <format>?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*interval*> | 是 | 整數  | 要減去的指定時間單位數字 | 
| <*timeUnit*> | 是 | 字串 | 與 *interval* 搭配使用的時間單位："Second"、"Minute"、"Hour"、"Day"、"Week"、"Month"、"Year" | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-timestamp*> | 字串 | 目前時間戳記減去指定的時間單位數字 | 
|||| 

*範例 1*

假設目前的時間戳記是 "2018-02-01T00:00:00.0000000Z"。 此範例會從此時間戳記中減去五天：

```
getPastTime(5, 'Day')
```

並傳回此結果：`"2018-01-27T00:00:00.0000000Z"`

*範例 2*

假設目前的時間戳記是 "2018-02-01T00:00:00.0000000Z"。 此範例會減去五天，並將結果轉換為 "D" 格式：

```
getPastTime(5, 'Day', 'D')
```

並傳回此結果：`"Saturday, January 27, 2018"`

<a name="greater"></a>

### <a name="greater"></a>greater

檢查第一個值是否大於第二個值。 當第一個值比較大時，傳回 True，或當第一個值比較小時，傳回 False。

```
greater(<value>, <compareTo>)
greater('<value>', '<compareTo>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 整數、浮點數或字串 | 要檢查其是否大於第二個值的第一個值 | 
| <*compareTo*> | 是 | 個別的整數、浮點數或字串 | 比較值 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| True 或 False | BOOLEAN | 當第一個值大於第二個值時，傳回 True。 當第一個值等於或小於第二個值時，傳回 False。 | 
|||| 

*範例*

以下範例會檢查第一個值是否大於第二個值：

```
greater(10, 5)
greater('apple', 'banana')
```

並傳回下列結果： 

* 第一個範例：`true`
* 第二個範例：`false`

<a name="greaterOrEquals"></a>

### <a name="greaterorequals"></a>greaterOrEquals

檢查第一個值是否大於或等於第二個值。
當第一個值較大或相等時，傳回 True，或當第一個值較小時，傳回 False。

```
greaterOrEquals(<value>, <compareTo>)
greaterOrEquals('<value>', '<compareTo>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 整數、浮點數或字串 | 要檢查其是否大於或等於第二個值的第一個值 | 
| <*compareTo*> | 是 | 個別的整數、浮點數或字串 | 比較值 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| True 或 False | BOOLEAN | 當第一個值大於或等於第二個值時，傳回 True。 當第一個值小於第二個值時，傳回 False。 | 
|||| 

*範例*

以下範例會檢查第一個值是否大於或等於第二個值：

```
greaterOrEquals(5, 5)
greaterOrEquals('apple', 'banana')
```

並傳回下列結果： 

* 第一個範例：`true`
* 第二個範例：`false`

<a name="guid"></a>

### <a name="guid"></a>GUID

以字串形式產生唯一識別碼 (GUID)，例如 "c2ecc88d-88c8-4096-912c-d6f2e2b138ce"： 

```
guid()
```

此外，除了預設格式 "D" (以連字號分隔的 32 個數字)，您可以指定不同格式的 GUID。

```
guid('<format>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*format*> | 否 | 字串 | 所傳回 GUID 的單一[格式規範](https://msdn.microsoft.com/library/97af8hh4)。 預設格式為 "D"，但您可以使用 "N"、"D"、"B"、"P" 或 "X"。 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*GUID-value*> | 字串 | 隨機產生的 GUID | 
|||| 

*範例* 

此範例會產生相同 GUID，不過格式是 32 個以連字號分隔的數字，並以括號括住： 

```
guid('P')
```

並傳回此結果：`"(c2ecc88d-88c8-4096-912c-d6f2e2b138ce)"`

<a name="if"></a>

### <a name="if"></a>if

檢查運算式是 True 或 False。 根據結果，傳回指定的值。

```
if(<expression>, <valueIfTrue>, <valueIfFalse>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*expression*> | 是 | BOOLEAN | 要檢查的運算式 | 
| <*valueIfTrue*> | 是 | 任意 | 運算式為 True 時要傳回的值 | 
| <*valueIfFalse*> | 是 | 任意 | 運算式為 False 時要傳回的值 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*specified-return-value*> | 任意 | 根據運算式為 True 或 False，傳回的指定值 | 
|||| 

*範例* 

此範例會傳回 `"yes"`，因為指定的運算式傳回 True。 否則，此範例會傳回 `"no"`：

```
if(equals(1, 1), 'yes', 'no')
```

<a name="indexof"></a>

### <a name="indexof"></a>indexOf

傳回子字串的起始位置或索引值。 此函式不區分大小寫，而且索引以數字 0 開頭。 

```
indexOf('<text>', '<searchText>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*text*> | 是 | 字串 | 具有子字串要尋找的字串 | 
| <*searchText*> | 是 | 字串 | 要尋找的子字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*index-value*>| 整數  | 所指定子字串的起始位置或索引值。 <p>如果找不到該字串，傳回數字 -1。 | 
|||| 

*範例* 

此範例會在 "hello world" 字串中，尋找 "world" 子字串的起始索引值：

```
indexOf('hello world', 'world')
```

並傳回此結果：`6`

<a name="int"></a>

### <a name="int"></a>int

傳回字串的整數版本。

```
int('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要轉換的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*integer-result*> | 整數  | 所指定字串的整數版本 | 
|||| 

*範例* 

此範例會建立字串 "10" 的整數版本：

```
int('10')
```

並傳回此結果：`10`

<a name="item"></a>

### <a name="item"></a>item

若用於透過陣列進行的重複動作中，則會傳回動作進行目前反覆項目期間，目前在陣列中的項目。 您也可以從該項目的屬性取得這些值。 

```
item()
```

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*current-array-item*> | 任意 | 進行動作的目前反覆項目時，目前存在陣列中的項目 | 
|||| 

*範例* 

此範例會從目前的 "Send_an_email" 動作訊息取得 `body` 元素，該動作在 for-each 迴圈的目前反覆項目中：

```
item().body
```

<a name="items"></a>

### <a name="items"></a>items

從 for-each 迴圈中的每個循環傳回目前項目。 在 for each 迴圈內使用此函式。

```
items('<loopName>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*loopName*> | 是 | 字串 | for-each 迴圈名稱 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*item*> | 任意 | 來自所指定 for each 迴圈中目前循環的項目 | 
|||| 

*範例* 

此範例會從指定 for each 迴圈中取得目前項目：

```
items('myForEachLoopName')
```

<a name="json"></a>

### <a name="json"></a>json

傳回字串或 XML 的 JavaScript 物件標記法 (JSON) 類型值或物件。

```
json('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串或 XML | 要轉換的字串或 XML | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*JSON-result*> | JSON 原生類型或物件 | 所指定字串或 XML 的 JSON 原生類型值或物件。 如果字串為 Null，函式會傳回空物件。 | 
|||| 

*範例 1* 

此範例會將此字串轉換為 JSON 值：

```
json('[1, 2, 3]')
```

並傳回此結果：`[1, 2, 3]`

*範例 2*

此範例會將此字串轉換為 JSON： 

```
json('{"fullName": "Sophia Owen"}')
```

並傳回此結果：

```
{
  "fullName": "Sophia Owen"
}
```

*範例 3*

此範例會將此 XML 轉換為 JSON： 

```
json(xml('<?xml version="1.0"?> <root> <person id='1'> <name>Sophia Owen</name> <occupation>Engineer</occupation> </person> </root>'))
```

並傳回此結果：

```json
{ 
   "?xml": { "@version": "1.0" }, 
   "root": { 
      "person": [ { 
         "@id": "1", 
         "name": "Sophia Owen", 
         "occupation": "Engineer" 
       } ] 
   } 
}
```

<a name="intersection"></a>

### <a name="intersection"></a>交集

在指定的多個集合中，傳回「只有」共同項目的集合。 項目若要出現在結果中，必須出現在所有傳遞至此函式的集合中。 如果一個或多個項目有相同的名稱，則具有該名稱的最後一個項目會出現在結果中。

```
intersection([<collection1>], [<collection2>], ...)
intersection('<collection1>', '<collection2>', ...)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*collection1*>, <*collection2*>, ... | 是 | 陣列或物件，但不可以兩者並存 | 您想要其中「只有」共同項目的集合 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*common-items*> | 個別的陣列或物件 | 在指定的多個集合中，「只有」共同項目的集合 | 
|||| 

*範例* 

此範例會從以下陣列中找出共同項目：  

```
intersection([1, 2, 3], [101, 2, 1, 10], [6, 8, 1, 2])
```

並傳回的「只有」這些項目的陣列：`[1, 2]`

<a name="join"></a>

### <a name="join"></a>Join

傳回具有陣列中所有項目的字串，並以「分隔符號」將每個字元隔開。

```
join([<collection>], '<delimiter>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*collection*> | 是 | 陣列 | 要將其項目聯結的陣列 |  
| <*delimiter*> | 是 | 字串 | 在結果字串中，要出現在每個字元之間的分隔符號 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*char1*><*delimiter*><*char2*><*delimiter*>... | 字串 | 從指定陣列中所有項目建立的結果字串 |
|||| 

*範例* 

此範例會從此陣列中所有項目建立字串，並以指定字元作為分隔符號：

```
join([a, b, c], '.')
```

並傳回此結果：`"a.b.c"`

<a name="last"></a>

### <a name="last"></a>last

傳回集合中的最後一個項目。

```
last('<collection>')
last([<collection>])
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*collection*> | 是 | 字串或陣列 | 要從中尋找最後一個項目的集合 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*last-collection-item*> | 個別的字串或陣列 | 集合中的最後一個項目 | 
|||| 

*範例* 

這些範例會尋找以下集合中最後一個項目：

```
last('abcd')
last([0, 1, 2, 3])
```

並傳回下列結果： 

* 第一個範例：`"d"`
* 第二個範例：`3`

<a name="lastindexof"></a>

### <a name="lastindexof"></a>lastIndexOf

傳回子字串最後一次出現的起始位置或索引值。 此函式不區分大小寫，而且索引以數字 0 開頭。

```
lastIndexOf('<text>', '<searchText>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*text*> | 是 | 字串 | 具有子字串要尋找的字串 | 
| <*searchText*> | 是 | 字串 | 要尋找的子字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*ending-index-value*> | 整數  | 指定的子字串最後一次出現的起始位置或索引值。 <p>如果找不到該字串，傳回數字 -1。 | 
|||| 

*範例* 

此範例會在 "hello world" 字串中，尋找 "world" 子字串最後一次出現的起始索引值：

```
lastIndexOf('hello world', 'world')
```

並傳回此結果：`6`

<a name="length"></a>

### <a name="length"></a>length

傳回集合中的項目數目。

```
length('<collection>')
length([<collection>])
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*collection*> | 是 | 字串或陣列 | 要計算其項目數的集合 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*length-or-count*> | 整數  | 集合中的項目數目 | 
|||| 

*範例*

這些範例會計算以下集合中的項目數： 

```
length('abcd')
length([0, 1, 2, 3])
```

並傳回此結果：`4`

<a name="less"></a>

### <a name="less"></a>less

檢查第一個值是否小於第二個值。
當第一個值較小時，傳回 True，或當第一個值較大時，傳回 False。

```
less(<value>, <compareTo>)
less('<value>', '<compareTo>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 整數、浮點數或字串 | 要檢查其是否小於第二個值的第一個值 | 
| <*compareTo*> | 是 | 個別的整數、浮點數或字串 | 比較項目 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| True 或 False | BOOLEAN | 當第一個值小於第二個值時，傳回 True。 當第一個值等於或大於第二個值時，傳回 False。 | 
|||| 

*範例*

以下範例會檢查第一個值是否小於第二個值。

```
less(5, 10)
less('banana', 'apple')
```

並傳回下列結果： 

* 第一個範例：`true`
* 第二個範例：`false`

<a name="lessOrEquals"></a>

### <a name="lessorequals"></a>lessOrEquals

檢查第一個值是否小於或等於第二個值。
當第一個值較小或相等時，傳回 True，或當第一個值較大時，傳回 False。

```
lessOrEquals(<value>, <compareTo>)
lessOrEquals('<value>', '<compareTo>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 整數、浮點數或字串 | 要檢查其是否小於或等於第二個值的第一個值 | 
| <*compareTo*> | 是 | 個別的整數、浮點數或字串 | 比較項目 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| True 或 False  | BOOLEAN | 當第一個值小於或等於第二個值時，傳回 True。 當第一個值大於第二個值時，傳回 False。 |  
|||| 

*範例*

以下範例會檢查第一個值是否小於或等於第二個值。

```
lessOrEquals(10, 10)
lessOrEquals('apply', 'apple')
```

並傳回下列結果： 

* 第一個範例：`true`
* 第二個範例：`false`

<a name="listCallbackUrl"></a>

### <a name="listcallbackurl"></a>listCallbackUrl

傳回呼叫觸發程序或動作的「回呼 URL」。 此函式僅適用於 **HttpWebhook** 和 **ApiConnectionWebhook** 連接器類型的觸發程序和動作，但不適用於**手動**、**循環**、**HTTP** 和 **APIConnection** 類型。 

```
listCallbackUrl()
```

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*callback-URL*> | 字串 | 觸發程序或動作的回呼 URL |  
|||| 

*範例*

此範例會顯示此函數可能會傳回的回呼 URL 範例：

`"https://prod-01.westus.logic.azure.com:443/workflows/<*workflow-ID*>/triggers/manual/run?api-version=2016-10-01&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=<*signature-ID*>"`

<a name="max"></a>

### <a name="max"></a>max

從具有數字的清單或陣列中傳回最大值 (包含首尾兩端的值)。 

```
max(<number1>, <number2>, ...)
max([<number1>, <number2>, ...])
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*number1*>, <*number2*>, ... | 是 | 整數、浮點數或兩者並存 | 您需要其中最大值的數字集合 | 
| [<*number1*>, <*number2*>, ...] | 是 | 陣列 - 整數、浮點數或兩者並存 | 您需要其中最大值的數字陣列 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*max-value*> | 整數或浮點數 | 所指定陣列或數字集合中的最大值 | 
|||| 

*範例* 

這些範例會從數字集合和陣列中取得最大值：

```
max(1, 2, 3)
max([1, 2, 3])
```

並傳回此結果：`3`

<a name="min"></a>

### <a name="min"></a>Min

從數字集合或陣列中傳回最小值。

```
min(<number1>, <number2>, ...)
min([<number1>, <number2>, ...])
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*number1*>, <*number2*>, ... | 是 | 整數、浮點數或兩者並存 | 您需要其中最小值的數字集合 | 
| [<*number1*>, <*number2*>, ...] | 是 | 陣列 - 整數、浮點數或兩者並存 | 您需要其中最小值的數字陣列 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*min-value*> | 整數或浮點數 | 所指定數字集合或陣列中的最小值 | 
|||| 

*範例* 

這些範例會取得數字集合和陣列中的最小值：

```
min(1, 2, 3)
min([1, 2, 3])
```

並傳回此結果：`1`

<a name="mod"></a>

### <a name="mod"></a>mod

傳回兩數相除的餘數。 若要取得整數結果，請參閱 [div()](#div)。

```
mod(<dividend>, <divisor>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*dividend*> | 是 | 整數或浮點數 | 要除以「除數」的數字 | 
| <*divisor*> | 是 | 整數或浮點數 | 要除「被除數」的數字，但不能為 0。 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*modulo-result*> | 整數或浮點數 | 第一個數字除以第二個數字的餘數 | 
|||| 

*範例* 

此範例會將第一個數字除以第二個數字：

```
mod(3, 2)
```

並傳回此結果：`1`

<a name="mul"></a>

### <a name="mul"></a>mul

傳回將兩數相乘的乘積。

```
mul(<multiplicand1>, <multiplicand2>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*multiplicand1*> | 是 | 整數或浮點數 | 要與「被乘數 2」 相乘的數字 | 
| <*multiplicand2*> | 是 | 整數或浮點數 | 要與「被乘數 1」 相乘的數字 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*product-result*> | 整數或浮點數 | 第一個數字與第二個數字相乘的乘積 | 
|||| 

*範例* 

這些範例會將第一個數字與第二個數字相乘：

```
mul(1, 2)
mul(1.5, 2)
```

並傳回下列結果：

* 第一個範例：`2`
* 第二個範例：`3`

<a name="multipartBody"></a>

### <a name="multipartbody"></a>multipartBody

在具有多個部分的動作輸出中，傳回特定部分的內容。

```
multipartBody('<actionName>', <index>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*actionName*> | 是 | 字串 | 其輸出具有多個部分的動作名稱 | 
| <*index*> | 是 | 整數  | 所需部分的索引值 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*body*> | 字串 | 所指定部分的內容 | 
|||| 

<a name="not"></a>

### <a name="not"></a>否

檢查運算式是否為 False。 運算式為 False 時，傳回 True，或運算式為 True 時，傳回 False。

```
not(<expression>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*expression*> | 是 | BOOLEAN | 要檢查的運算式 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| True 或 False | BOOLEAN | 運算式為 False 時，傳回 True。 運算式為 True 時，傳回 False。 |  
|||| 

*範例 1*

這些範例會檢查指定的運算式是否為 False： 

```
not(false)
not(true)
```

並傳回下列結果：

* 第一個範例：運算式為 False，所以函數傳回 `true`。
* 第一個範例：運算式為 True，所以函數傳回 `false`。

*範例 2*

這些範例會檢查指定的運算式是否為 False： 

```
not(equals(1, 2))
not(equals(1, 1))
```

並傳回下列結果：

* 第一個範例：運算式為 False，所以函數傳回 `true`。
* 第一個範例：運算式為 True，所以函數傳回 `false`。

<a name="or"></a>

### <a name="or"></a>或

檢查是否至少有一個運算式是 True。 至少有一個運算式是 True 時，傳回 True，或所有運算式都是 False 時，傳回 False。

```
or(<expression1>, <expression2>, ...)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*expression1*>, <*expression2*>, ... | 是 | BOOLEAN | 要檢查的運算式 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| True 或 False | BOOLEAN | 至少有一個運算式是 True 時，傳回 True。 所有運算式都是 False 時，即傳回 False。 |  
|||| 

*範例 1*

這些範例會檢查是否至少有一個運算式是 True：

```
or(true, false)
or(false, false)
```

並傳回下列結果：

* 第一個範例：至少有一個運算式為 True，所以函數傳回 `true`。
* 第二個範例：兩個運算式都是 False，所以函式傳回 `false`。

*範例 2*

這些範例會檢查是否至少有一個運算式是 True：

```
or(equals(1, 1), equals(1, 2))
or(equals(1, 2), equals(1, 3))
```

並傳回下列結果：

* 第一個範例：至少有一個運算式為 True，所以函數傳回 `true`。
* 第二個範例：兩個運算式都是 False，所以函式傳回 `false`。

<a name="parameters"></a>

### <a name="parameters"></a>parameters

傳回邏輯應用程式定義中描述的參數值。 

```
parameters('<parameterName>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*parameterName*> | 是 | 字串 | 您需要其值的參數名稱 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*parameter-value*> | 任意 | 所指定參數的值 | 
|||| 

*範例* 

假設您有此 JSON 值：

```json
{
  "fullName": "Sophia Owen"
}
```

此範例會取得指定參數的值：

```
parameters('fullName')
```

並傳回此結果：`"Sophia Owen"`

<a name="rand"></a>

### <a name="rand"></a>rand

從指定範圍傳回隨機整數 (不包含範圍中的末端數字)。

```
rand(<minValue>, <maxValue>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*minValue*> | 是 | 整數  | 範圍中的最小整數 | 
| <*maxValue*> | 是 | 整數  | 在範圍中，在函式所能傳回最大整數後面的整數 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*random-result*> | 整數  | 從指定範圍傳回的隨機整數 |  
|||| 

*範例*

此範例會從指定範圍取得隨機整數，但不包含最大值： 

```
rand(1, 5)
```

並傳回這些數字的其中一個作為結果：`1`、`2`、`3` 或 `4` 

<a name="range"></a>

### <a name="range"></a>range

傳回從指定整數開始的整數陣列。

```
range(<startIndex>, <count>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*startIndex*> | 是 | 整數  | 作為第一個項目起始陣列的整數值 | 
| <*count*> | 是 | 整數  | 陣列中的整數數量 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| [<*range-result*>] | 陣列 | 從指定索引開始的整數陣列 |  
|||| 

*範例*

此範例會建立從指定索引開始的整數陣列，且其中有指定數量的整數：

```
range(1, 4)
```

並傳回此結果：`[1, 2, 3, 4]`

<a name="replace"></a>

### <a name="replace"></a>取代

使用指定字串取代子字串，並傳回結果字串。 此函式會區分大小寫。

```
replace('<text>', '<oldText>', '<newText>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*text*> | 是 | 字串 | 有子字串要取代的字串 | 
| <*oldText*> | 是 | 字串 | 要取代的子字串 | 
| <*newText*> | 是 | 字串 | 取代字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-text*> | 字串 | 取代子字串後的更新字串 <p>如果找不到子字串，則傳回原始字串。 | 
|||| 

*範例* 

此範例會尋找 "the old string" 中的 "old" 字串，然後以"new" 取代 "old"： 

```
replace('the old string', 'old', 'new')
```

並傳回此結果：`"the new string"`

<a name="removeProperty"></a>

### <a name="removeproperty"></a>removeProperty

從物件中移除屬性，並傳回更新的物件。

```
removeProperty(<object>, '<property>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*object*> | 是 | Object | 您要從中移除屬性的 JSON 物件 | 
| <*property*> | 是 | 字串 | 要移除的屬性名稱 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-object*> | Object | 沒有指定屬性的更新 JSON 物件 | 
|||| 

*範例*

此範例會從以 [JSON()](#json) 函式轉換為 JSON 的 `"customerProfile"` 物件中移除 `"accountLocation"` 屬性，並傳回更新的物件：

```
removeProperty(json('customerProfile'), 'accountLocation')
```

<a name="setProperty"></a>

### <a name="setproperty"></a>setProperty

設定物件屬性的值，並傳回更新的物件。 若要新增屬性，您可以使用此函式或 [addProperty()](#addProperty) 函式。

```
setProperty(<object>, '<property>', <value>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*object*> | 是 | Object | 您想要設定其屬性的 JSON 物件 | 
| <*property*> | 是 | 字串 | 要設定的現有屬性或新屬性名稱 | 
| <*value*> | 是 | 任意 | 要為指定屬性設定的值 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-object*> | Object | 您已設定其屬性的更新 JSON 物件 | 
|||| 

*範例*

此範例會設定 `"customerProfile"` 物件上的 `"accountNumber"` 屬性，此物件是以 [JSON()](#json) 函式轉換為 JSON。 函式會指派由 [guid()](#guid) 函式產生的值，並傳回更新的 JSON 物件：

```
setProperty(json('customerProfile'), 'accountNumber', guid())
```

<a name="skip"></a>

### <a name="skip"></a>skip

移除集合前端的項目，並傳回「其他所有」項目。

```
skip([<collection>], <count>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*collection*> | 是 | 陣列 | 您想要從中移除項目的集合 | 
| <*count*> | 是 | 整數  | 正整數，表示要移除的前端項目數量 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| [<*updated-collection*>] | 陣列 | 移除指定項目之後的更新集合 | 
|||| 

*範例*

此範例會從指定陣列的前端移除一個項目，也就是數字 0： 

```
skip([0, 1, 2, 3], 1)
```

並傳回具有剩餘項目的此陣列：`[1,2,3]`

<a name="split"></a>

### <a name="split"></a>split

傳回具有字串中所有字元的陣列，並以「分隔符號」將每個字元隔開。

```
split('<text>', '<separator>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*text*> | 是 | 字串 | 要分割其中字元的字串 |  
| <*separator*> | 是 | 字串 | 在結果陣列中，要出現在每個字元之間的分隔符號 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| [<*char1*><*separator*><*char2*><*separator*>...] | 陣列 | 從指定字串中所有項目建立的結果陣列 |
|||| 

*範例* 

此範例會從指定字串建立陣列，並以逗號作為分隔符號來分隔每個字元：

```
split('abc', ',')
```

並傳回此結果：`[a, b, c]`

<a name="startOfDay"></a>

### <a name="startofday"></a>startOfDay

傳回時間戳記中當天的起始點。 

```
startOfDay('<timestamp>', '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-timestamp*> | 字串 | 指定的時間戳記，但以當天的零小時標記開始 | 
|||| 

*範例* 

此範例會尋找此時間戳記中當天的起始點：

```
startOfDay('2018-03-15T13:30:30Z')
```

並傳回此結果：`"2018-03-15T00:00:00.0000000Z"`

<a name="startOfHour"></a>

### <a name="startofhour"></a>startOfHour

傳回時間戳記中小時的起始點。 

```
startOfHour('<timestamp>', '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-timestamp*> | 字串 | 指定的時間戳記，但以小時的零分鐘標記開始 | 
|||| 

*範例* 

此範例會尋找此時間戳記中小時的起始點：

```
startOfHour('2018-03-15T13:30:30Z')
```

並傳回此結果：`"2018-03-15T13:00:00.0000000Z"`

<a name="startOfMonth"></a>

### <a name="startofmonth"></a>startOfMonth

傳回時間戳記中月份的起始點。 

```
startOfMonth('<timestamp>', '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-timestamp*> | 字串 | 指定的時間戳記，但以當月第一天的零小時標記開始 | 
|||| 

*範例* 

此範例會傳回此時間戳記中月份的起始點：

```
startOfMonth('2018-03-15T13:30:30Z')
```

並傳回此結果：`"2018-03-01T00:00:00.0000000Z"`

<a name="startswith"></a>

### <a name="startswith"></a>startsWith

檢查字串是否以特定的子字串開始。 找到子字串時，傳回 True，或找不到子字串時，傳回 False。 此函式不區分大小寫。

```
startsWith('<text>', '<searchText>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*text*> | 是 | 字串 | 要檢查的字串 | 
| <*searchText*> | 是 | 字串 | 要尋找的起始字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| True 或 False  | BOOLEAN | 找到起始子字串時，傳回 True。 找不到項目時，傳回 False。 | 
|||| 

*範例 1* 

此範例會檢查 "hello world" 字串是否以 "hello" 子字串開始：

```
startsWith('hello world', 'hello')
```

並傳回此結果：`true`

*範例 2*

此範例會檢查 "hello world" 字串是否以 "greetings" 子字串開始：

```
startsWith('hello world', 'greetings')
```

並傳回此結果：`false`

<a name="string"></a>

### <a name="string"></a>字串

傳回值的字串版本。

```
string(<value>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 任意 | 要轉換的值 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*string-value*> | 字串 | 指定值的字串版本 | 
|||| 

*範例 1* 

此範例會建立此數字的字串版本：

```
string(10)
```

並傳回此結果：`"10"`

*範例 2*

此範例會為指定的 JSON 物件建立字串，並使用反斜線字元 (\\) 作為雙引號 (") 的逸出字元。

```
string( { "name": "Sophie Owen" } )
```

並傳回此結果：`"{ \\"name\\": \\"Sophie Owen\\" }"`

<a name="sub"></a>

### <a name="sub"></a>sub

傳回第一個數字減去第二個數字的結果。

```
sub(<minuend>, <subtrahend>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*minuend*> | 是 | 整數或浮點數 | 要從中減去「減數」的數字 | 
| <*subtrahend*> | 是 | 整數或浮點數 | 從「被減數」中減去的數字 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*result*> | 整數或浮點數 | 從第一個數字減去第二個數字的結果 | 
|||| 

*範例* 

此範例會從第一個數字減去第二個數字：

```
sub(10.3, .3)
```

並傳回此結果：`10`

<a name="substring"></a>

### <a name="substring"></a>substring

從字串中傳回從指定位置或索引起始的字元。 索引值會以數字 0 開頭。 

```
substring('<text>', <startIndex>, <length>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*text*> | 是 | 字串 | 您需要其中字元的字串 | 
| <*startIndex*> | 是 | 整數  | 作為起始位置或索引值的正數 | 
| <*length*> | 是 | 整數  | 子字串中您需要的字元數正數 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*substring-result*> | 字串 | 具有指定字元數目的子字串，並以來源字串中的指定索引位置起始 | 
|||| 

*範例* 

此範例會從指定字串建立 5 個字元的子字串，並且從索引值 6 起始：

```
substring('hello world', 6, 5)
```

並傳回此結果：`"world"`

<a name="subtractFromTime"></a>

### <a name="subtractfromtime"></a>subtractFromTime

從時間戳記減去時間單位數字。 另請參閱 [getPastTime](#getPastTime)。

```
subtractFromTime('<timestamp>', <interval>, '<timeUnit>', '<format>'?)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 包含時間戳記的字串 | 
| <*interval*> | 是 | 整數  | 要減去的指定時間單位數字 | 
| <*timeUnit*> | 是 | 字串 | 與 *interval* 搭配使用的時間單位："Second"、"Minute"、"Hour"、"Day"、"Week"、"Month"、"Year" | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updated-timestamp*> | 字串 | 時間戳記減去指定的時間單位數字 | 
|||| 

*範例 1*

此範例會從此時間戳記中減去一天：

```
subtractFromTime('2018-01-02T00:00:00Z', 1, 'Day') 
```

並傳回此結果：`"2018-01-01T00:00:00:0000000Z"`

*範例 2*

此範例會從此時間戳記中減去一天：

```
subtractFromTime('2018-01-02T00:00:00Z', 1, 'Day', 'D') 
```

並傳回選用 "D" 格式的此結果：`"Monday, January, 1, 2018"`

<a name="take"></a>

### <a name="take"></a>take

傳回集合中的前端項目。 

```
take('<collection>', <count>)
take([<collection>], <count>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*collection*> | 是 | 字串或陣列 | 您需要其中項目的集合 | 
| <*count*> | 是 | 整數  | 正整數，表示您需要的前端項目數量 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*subset*> 或 [<*subset*>] | 個別的字串或陣列 | 從原始集合前端取得指定項目數量的字串或陣列 | 
|||| 

*範例*

此範例會從這些集合的前端取得指定數目的項目：

```
take('abcde`, 3)
take([0, 1, 2, 3, 4], 3)
```

並傳回下列結果：

* 第一個範例：`"abc"`
* 第二個範例：`[0, 1, 2]`

<a name="ticks"></a>

### <a name="ticks"></a>刻度

傳回指定時間戳記的 `ticks` 屬性值。 一「刻度」是 100 奈秒的間隔。

```
ticks('<timestamp>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*timestamp*> | 是 | 字串 | 時間戳記的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*ticks-number*> | 整數  | 自指定時間戳記以來的刻度數目 | 
|||| 

<a name="toLower"></a>

### <a name="tolower"></a>toLower

傳回小寫格式的字串。 如果字串中的字元沒有小寫版本，則該字元在傳回的字串中會保持不變。

```
toLower('<text>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*text*> | 是 | 字串 | 要以小寫格式傳回的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*lowercase-text*> | 字串 | 小寫格式的原始字串 | 
|||| 

*範例* 

此範例會將此字串轉換為小寫： 

```
toLower('Hello World')
```

並傳回此結果：`"hello world"`

<a name="toUpper"></a>

### <a name="toupper"></a>toUpper

傳回大寫格式的字串。 如果字串中的字元沒有大寫版本，則該字元在傳回的字串中會保持不變。

```
toUpper('<text>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*text*> | 是 | 字串 | 要以大寫格式傳回的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*uppercase-text*> | 字串 | 大寫格式的原始字串 | 
|||| 

*範例* 

此範例會將此字串轉換為大寫：

```
toUpper('Hello World')
```

並傳回此結果：`"HELLO WORLD"`

<a name="trigger"></a>

### <a name="trigger"></a>觸發程序

傳回執行階段上觸發程序的輸出，或來自其他 JSON 成對名稱和數值中的值，而您可以將其指派給運算式。 

* 在觸發程序的輸入內，此函式會傳回先前執行的輸出。 

* 在觸發程序的條件內，此函式會傳回目前執行的輸出。 

根據預設，函式會參考整個觸發程序物件，但是您可以選擇性地指定其中有所需值的屬性。 此外，此函式有簡略版可用，請參閱 [triggerOutputs()](#triggerOutputs) 和 [triggerBody()](#triggerBody)。 

```
trigger()
```

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*trigger-output*> | 字串 | 執行階段上觸發程序的輸出 | 
|||| 

<a name="triggerBody"></a>

### <a name="triggerbody"></a>triggerBody

傳回觸發程序在執行階段的 `body` 輸出。 `trigger().outputs.body` 的簡略版。 請參閱 [trigger()](#trigger)。 

```
triggerBody()
```

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*trigger-body-output*> | 字串 | 來自觸發程序的 `body` 輸出 | 
|||| 

<a name="triggerFormDataMultiValues"></a>

### <a name="triggerformdatamultivalues"></a>triggerFormDataMultiValues

傳回帶有值的陣列，此值會符合觸發程序「表單資料」或「表單編碼」輸出的索引鍵名稱。 

```
triggerFormDataMultiValues('<key>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*key*> | 是 | 字串 | 您需要其值的索引鍵名稱 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| [<*array-with-key-values*>] | 陣列 | 此陣列具有符合指定索引鍵的所有值 | 
|||| 

*範例* 

此範例會從 "feedUrl" 索引鍵值建立陣列，而此索引鍵值位於 RSS 觸發程序的表單資料或表單編碼輸出中： 

```
triggerFormDataMultiValues('feedUrl')
```

並傳回此陣列作為範例結果：`["http://feeds.reuters.com/reuters/topNews"]`

<a name="triggerFormDataValue"></a>

### <a name="triggerformdatavalue"></a>triggerFormDataValue

傳回具有單一值的字串，此單一值會符合觸發程序「表單資料」或「表單編碼」輸出的索引鍵名稱。 如果函式找到一個以上的相符項目，函式會擲回錯誤。

```
triggerFormDataValue('<key>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*key*> | 是 | 字串 | 您需要其值的索引鍵名稱 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*key-value*> | 字串 | 所指定索引鍵中的值 | 
|||| 

*範例* 

此範例會從 "feedUrl" 索引鍵值建立字串，而此索引鍵值位於 RSS 觸發程序的表單資料或表單編碼輸出中：

```
triggerFormDataValue('feedUrl')
```

並傳回此字串作為範例結果：`"http://feeds.reuters.com/reuters/topNews"` 

<a name="triggerMultipartBody"></a>

### <a name="triggermultipartbody"></a>triggerMultipartBody

從具有多個部分的觸發程序輸出中，傳回特定部分的內容。 

```
triggerMultipartBody(<index>)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*index*> | 是 | 整數  | 所需部分的索引值 |
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*body*> | 字串 | 在觸發程序的多部分輸出中，指定部分的內容 | 
|||| 

<a name="triggerOutputs"></a>

### <a name="triggeroutputs"></a>triggerOutputs

傳回觸發程序在執行階段的輸出，或來自其他 JSON 名稱與值配對中的值。 `trigger().outputs` 的簡略版。 請參閱 [trigger()](#trigger)。 

```
triggerOutputs()
```

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*trigger-output*> | 字串 | 執行階段上觸發程序的輸出  | 
|||| 

<a name="trim"></a>

### <a name="trim"></a>修剪

移除字串的開頭和尾端空白字元，並傳回更新後的字串。

```
trim('<text>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*text*> | 是 | 字串 | 要為其移除開頭和尾端空白字元的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updatedText*> | 字串 | 不含開頭或尾端空白字元的原始字串更新版本 | 
|||| 

*範例* 

此範例會從 " Hello World  " 字串中移除開頭和尾端空白字元：  

```
trim(' Hello World  ')
```

並傳回此結果：`"Hello World"`

<a name="union"></a>

### <a name="union"></a>union

傳回具有指定集合中「所有」項目的集合。 出現在結果中的項目，可以出現在任何傳遞至此函式的集合中。 如果一個或多個項目有相同的名稱，則具有該名稱的最後一個項目會出現在結果中。 

```
union('<collection1>', '<collection2>', ...)
union([<collection1>], [<collection2>], ...)
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*collection1*>, <*collection2*>, ...  | 是 | 陣列或物件，但不可以兩者並存 | 您想要其中「所有」項目的集合 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*updatedCollection*> | 個別的陣列或物件 | 具有指定集合中所有項目的集合 - 不含重複值 | 
|||| 

*範例* 

此範例會從以下集合取得「所有」項目： 

```
union([1, 2, 3], [1, 2, 10, 101])
```

並傳回此結果：`[1, 2, 3, 10, 101]`

<a name="uriComponent"></a>

### <a name="uricomponent"></a>uriComponent

傳回字串的統一資源識別項 (URI) 編碼版本，以逸出字元取代 URL 中的 Unsafe 字元。 使用此函式而非 [encodeUriComponent()](#encodeUriComponent)。 雖然這兩個函數的運作方式相同，但是較常使用 `uriComponent()`。

```
uriComponent('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要轉換成 URI 編碼格式的字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*encoded-uri*> | 字串 | 具有逸出字元的 URI 編碼字串 | 
|||| 

*範例*

此範例會建立此字串的 URI 編碼版本：

```
uriComponent('https://contoso.com')
```

並傳回此結果：`"http%3A%2F%2Fcontoso.com"`

<a name="uriComponentToBinary"></a>

### <a name="uricomponenttobinary"></a>uriComponentToBinary

傳回統一資源識別項 (URI) 元件的二進位版本。

```
uriComponentToBinary('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要轉換的 URI 編碼字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*binary-for-encoded-uri*> | 字串 | URI 編碼字串的二進位版本。 二進位內容是以 base64 編碼，而且由 `$content` 表示。 | 
|||| 

*範例*

此範例會建立此 URI 編碼字串的二進位版本： 

```
uriComponentToBinary('http%3A%2F%2Fcontoso.com')
```

並傳回此結果： 

`"001000100110100001110100011101000111000000100101001100
11010000010010010100110010010001100010010100110010010001
10011000110110111101101110011101000110111101110011011011
110010111001100011011011110110110100100010"`

<a name="uriComponentToString"></a>

### <a name="uricomponenttostring"></a>uriComponentToString

傳回統一資源識別項 (URI) 編碼字串的字串版本，也就是有效地解碼 URI 編碼字串。

```
uriComponentToString('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 要解碼的 URI 編碼字串 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*decoded-uri*> | 字串 | URI 編碼字串的已解碼版本 | 
|||| 

*範例*

此範例會建立此 URI 編碼字串的已解碼字串版本： 

```
uriComponentToString('http%3A%2F%2Fcontoso.com')
```

並傳回此結果：`"https://contoso.com"` 

<a name="uriHost"></a>

### <a name="urihost"></a>uriHost

傳回統一資源識別項 (URI) 的 `host` 值。

```
uriHost('<uri>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*uri*> | 是 | 字串 | 您需要其 `host` 值的 URI | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*host-value*> | 字串 | 所指定 URI 的 `host` 值 | 
|||| 

*範例*

此範例會尋找此 URI 的 `host` 值： 

```
uriHost('https://www.localhost.com:8080')
```

並傳回此結果：`"www.localhost.com"`

<a name="uriPath"></a>

### <a name="uripath"></a>uriPath

傳回統一資源識別項 (URI) 的 `path` 值。 

```
uriPath('<uri>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*uri*> | 是 | 字串 | 您需要其 `path` 值的 URI | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*path-value*> | 字串 | 所指定 URI 的 `path` 值。 如果 `path` 沒有值，則傳回 "/" 字元。 | 
|||| 

*範例*

此範例會尋找此 URI 的 `path` 值： 

```
uriPath('http://www.contoso.com/catalog/shownew.htm?date=today')
```

並傳回此結果：`"/catalog/shownew.htm"`

<a name="uriPathAndQuery"></a>

### <a name="uripathandquery"></a>uriPathAndQuery

傳回統一資源識別項 (URI) 的 `path` 和 `query` 值。

```
uriPathAndQuery('<uri>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*uri*> | 是 | 字串 | 您需要其 `path` 和 `query` 值的 URI | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*path-query-value*> | 字串 | 所指定 URI 的 `path` 和 `query` 值。 如果 `path` 未指定值，則傳回 "/" 字元。 | 
|||| 

*範例*

此範例會尋找此 URI 的 `path` 和 `query` 值：

```
uriPathAndQuery('http://www.contoso.com/catalog/shownew.htm?date=today')
```

並傳回此結果：`"/catalog/shownew.htm?date=today"`

<a name="uriPort"></a>

### <a name="uriport"></a>uriPort

傳回統一資源識別項 (URI) 的 `port` 值。

```
uriPort('<uri>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*uri*> | 是 | 字串 | 您需要其 `port` 值的 URI | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*port-value*> | 整數  | 所指定 URI 的 `port` 值。 如果 `port` 未指定值，則傳回通訊協定的預設連接埠。 | 
|||| 

*範例*

此範例會傳回此 URI 的 `port` 值：

```
uriPort('http://www.localhost:8080')
```

並傳回此結果：`8080`

<a name="uriQuery"></a>

### <a name="uriquery"></a>uriQuery

傳回統一資源識別項 (URI) 的 `query` 值。

```
uriQuery('<uri>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*uri*> | 是 | 字串 | 您需要其 `query` 值的 URI | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*query-value*> | 字串 | 所指定 URI 的 `query` 值 | 
|||| 

*範例*

此範例會傳回此 URI 的 `query` 值： 

```
uriQuery('http://www.contoso.com/catalog/shownew.htm?date=today')
```

並傳回此結果：`"?date=today"`

<a name="uriScheme"></a>

### <a name="urischeme"></a>uriScheme

傳回統一資源識別項 (URI) 的 `scheme` 值。

```
uriScheme('<uri>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*uri*> | 是 | 字串 | 您需要其 `scheme` 值的 URI | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*scheme-value*> | 字串 | 所指定 URI 的 `scheme` 值 | 
|||| 

*範例*

此範例會傳回此 URI 的 `scheme` 值：

```
uriScheme('http://www.contoso.com/catalog/shownew.htm?date=today')
```

並傳回此結果：`"http"`

<a name="utcNow"></a>

### <a name="utcnow"></a>utcNow

傳回目前的時間戳記。 

```
utcNow('<format>')
```

您可以選擇性地以 <*format*> 參數指定不同格式。 


| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*format*> | 否 | 字串 | [單一格式規範](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings)或[自訂格式模式](https://docs.microsoft.com/dotnet/standard/base-types/custom-date-and-time-format-strings)。 時間戳記的預設格式為 ["o"](https://docs.microsoft.com/dotnet/standard/base-types/standard-date-and-time-format-strings) (yyyy-MM-ddT:mm:ss:fffffffK)，其符合 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 並保留時區資訊。 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*current-timestamp*> | 字串 | 目前的日期和時間 | 
|||| 

*範例 1*

假設今天是 2018 年 4 月 15 日下午 1:00:00。 此範例會取得目前的時間戳記： 

```
utcNow()
```

並傳回此結果：`"2018-04-15T13:00:00.0000000Z"`

*範例 2*

假設今天是 2018 年 4 月 15 日下午 1:00:00。 此範例會使用選擇性的 "D" 格式取得目前時間戳記：

```
utcNow('D')
```

並傳回此結果：`"Sunday, April 15, 2018"`

<a name="variables"></a>

### <a name="variables"></a>variables

傳回指定變數的值。 

```
variables('<variableName>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*variableName*> | 是 | 字串 | 您需要其值的變數名稱 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*variable-value*> | 任意 | 所指定變數的值 | 
|||| 

*範例*

假設 "numItems" 變數的目前值為 20。 此範例會取得此變數的整數值：

```
variables('numItems')
```

並傳回此結果：`20`

<a name="workflow"></a>

### <a name="workflow"></a>工作流程

傳回執行階段期間與工作流程本身相關的所有詳細資料。 

```
workflow().<property>
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*property*> | 否 | 字串 | 您需要其值的工作流程屬性名稱 <p>工作流程物件具有下列屬性：**name**、**type** **id** **location** 和 **run**。 **run** 屬性值也是具有這些屬性的物件：**name**、**type**和 **id**。 | 
||||| 

*範例*

此範例會傳回工作流程的目前執行名稱：

```
workflow().run.name
```

<a name="xml"></a>

### <a name="xml"></a>xml

傳回包含 JSON 物件的 XML 版字串。 

```
xml('<value>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*value*> | 是 | 字串 | 其中有 JSON 物件要轉換的字串 <p>JSON 物件必須只能有一個根屬性。 <br>使用反斜線字元 (\\) 作為雙引號 (") 的逸出字元。 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*xml-version*> | Object | 所指定字串或 JSON 物件的編碼 XML | 
|||| 

*範例 1*

此範例會為包含 JSON 物件的此字串建立 XML 版本： 

`xml( '{ \"name\": \"Sophia Owen\" }' )`

並傳回此結果 XML： 

```xml
<name>Sophia Owen</name>
```

*範例 2*

假設您有此 JSON 物件：

```json
{ 
  "person": { 
    "name": "Sophia Owen", 
    "city": "Seattle" 
  } 
}
```

此範例會為包含此 JSON 物件的字串建立 XML：

`xml( '{ \"person\": { \"name\": \"Sophia Owen\", \"city\": \"Seattle\" } }' )`

並傳回此結果 XML： 

```xml
<person>
  <name>Sophia Owen</name>
  <city>Seattle</city>
<person>
```

<a name="xpath"></a>

### <a name="xpath"></a>xpath

檢查 XML 中是否有符合 XPath (XML 路徑語言) 運算式的節點或值，並傳回符合的節點或值。 XPath 運算式 (或 "XPath") 可協助您瀏覽 XML 文件結構，讓您可以在 XML 內容中選取節點或計算值。

```
xpath('<xml>', '<xpath>')
```

| 參數 | 必要 | 類型 | 說明 | 
| --------- | -------- | ---- | ----------- | 
| <*xml*> | 是 | 任意 | XML 字串，將對其搜尋是否有符合 XPath 運算式的值或節點 | 
| <*xpath*> | 是 | 任意 | 用來尋找相符 XML 節點或值的 XPath 運算式 | 
||||| 

| 傳回值 | 類型 | 說明 | 
| ------------ | ---- | ----------- | 
| <*xml-node*> | XML | 只有單一節點符合指定的 XPath 運算式時會傳回 XML 節點 | 
| <*value*> | 任意 | 只有單一值符合指定的 XPath 運算式時，會傳回 XML 節點的值 | 
| [<*xml-node1*>, <*xml-node2*>, ...] </br>-或- </br>[<*value1*>, <*value2*>, ...] | 陣列 | 陣列，其中有符合指定 XPath 運算式的 XML 節點或值 | 
|||| 

*範例 1*

此範例會尋找是否有節點符合指定引數中的 `<name></name>` 節點，並傳回包含這些節點值的陣列： 

`xpath(xml(parameters('items')), '/produce/item/name')`

以下是引數：

* 包含此 XML 的 "items" 字串：

  `"<?xml version="1.0"?> <produce> <item> <name>Gala</name> <type>apple</type> <count>20</count> </item> <item> <name>Honeycrisp</name> <type>apple</type> <count>10</count> </item> </produce>"`

  此範例會使用 [parameters()](#parameters) 函式來取得 "items" 引數中的 XML 字串，但也必須使用 [xml()](#xml) 函式將字串轉換為 XML 格式。 

* 作為字串傳遞的此 XPath 運算式：

  `"/produce/item/name"`

以下是結果陣列，其中有符合 `<name></name`的節點：

`[ <name>Gala</name>, <name>Honeycrisp</name> ]`

*範例 2*

以範例 1 為基礎，此範例會尋找符合 `<count></count>` 節點的節點，並使用 `sum()` 函式將這些節點值相加：

`xpath(xml(parameters('items')), 'sum(/produce/item/count)')`

並傳回此結果：`30`

*範例 3*

在此範例中，有兩個運算式會在指定的引數中 (其中包含具有命名空間的 XML) 尋找符合 `<location></location>` 節點的節點。 此運算式使用反斜線字元 (\\) 作為雙引號 (") 的逸出字元。

* 運算式 1

  `xpath(xml(body('Http')), '/*[name()=\"file\"]/*[name()=\"location\"]')`

* 運算式2  

  `xpath(xml(body('Http')), '/*[local-name=()=\"file\"] and namespace-uri()=\"http://contoso.com\"/*[local-name()]=\"location\" and namespace-uri()=\"\"]')`

以下是引數：

* 包含 XML 文件命名空間的此 XML `xmlns="http://contoso.com"`： 

  ```xml
  <?xml version="1.0"?> <file xmlns="http://contoso.com"> <location>Paris</location> </file>
  ```

* 此處其中一個 XPath 運算式：

  * `/*[name()=\"file\"]/*[name()=\"location\"]`

  * `/*[local-name=()=\"file\"] and namespace-uri()=\"http://contoso.com\"/*[local-name()]=\"location\" and namespace-uri()=\"\"]`

以下是符合 `<location></location` 節點的結果節點：

```xml
<location xmlns="https://contoso.com">Paris</location>
```

*範例 4*

以範例 3 為基礎，此範例會尋找 `<location></location>` 節點中的值： 

`xpath(xml(body('Http')), 'string(/*[name()=\"file\"]/*[name()=\"location\"])')`

並傳回此結果：`"Paris"`

## <a name="next-steps"></a>後續步驟

了解[工作流程定義語言](../logic-apps/logic-apps-workflow-definition-language.md)
