---
title: 工作流程定義語言的結構描述參考 - Azure Logic Apps | Microsoft Docs
description: 使用工作流程定義語言撰寫 Azure Logic Apps 的自訂工作流程定義
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
manager: jeconnoc
ms.topic: reference
ms.date: 04/30/2018
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: d5dadd054f95e61626942a1cab7d95ba8c9182e1
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141297"
---
# <a name="schema-reference-for-workflow-definition-language-in-azure-logic-apps"></a>Azure Logic Apps 中工作流程定義語言的結構描述參考

當您使用 [Azure Logic Apps](../logic-apps/logic-apps-overview.md) 建立邏輯應用程式工作流程時，您工作流程的基礎定義會描述針對您的邏輯應用程式執行的實際邏輯。 此描述會遵循工作流程定義語言結構描述所定義和驗證的結構，其使用 [JavaScript 物件標記法 (JSON)](https://www.json.org/)。 
  
## <a name="workflow-definition-structure"></a>工作流程定義結構

工作流程定義至少有一個觸發程序可具現化您的應用程式邏輯，再加上您的邏輯應用程式所執行的一或多個動作。 

以下是工作流程定義的高階結構︰  
  
```json
"definition": {
  "$schema": "<workflow-definition-language-schema-version>",
  "contentVersion": "<workflow-definition-version-number>",
  "parameters": { "<workflow-parameter-definitions>" },
  "triggers": { "<workflow-trigger-definitions>" },
  "actions": { "<workflow-action-definitions>" },
  "outputs": { "<workflow-output-definitions>" }
}
```
  
| 元素 | 必要 | 說明 | 
|---------|----------|-------------| 
| 定義 | 是 | 工作流程定義的起始元素 | 
| $schema | 只有在外部參考工作流程定義時 | JSON 結構描述檔案的位置，該檔案說明工作流程定義語言版本，您可以在此找到此版本： <p>`https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json`</p> |   
| contentVersion | 否 | 您的工作流程定義版本號碼，預設為 "1.0.0.0"。 若要在部署工作流程時協助識別及確認正確的定義，請指定要使用的值。 | 
| parameters | 否 | 一或多個參數的定義，此參數可將資料傳入您的工作流程中 <p><p>參數上限：50 | 
| 觸發程序 | 否 | 一或多個觸發程序的定義，此觸發程序可具現化您的工作流程。 您可以定義多個觸發程序，但只能利用工作流程定義語言，而不會透過 Logic Apps 設計工具呈現。 <p><p>觸發程序上限：10 | 
| 動作 | 否 | 要在工作流程執行階段執行之一或多個動作的定義 <p><p>動作上限：250 | 
| 輸出 | 否 | 從工作流程執行傳回之輸出的定義 <p><p>輸出上限：10 |  
|||| 

## <a name="parameters"></a>參數

在 `parameters` 區段中，定義邏輯應用程式在部署時用於接受輸入的所有工作流程參數。 在部署時需要有參數宣告和參數值。 請務必先在這些區段中宣告所有參數，您才可以在其他工作流程區段中使用這些參數。 

以下是參數定義的一般結構︰  

```json
"parameters": {
  "<parameter-name>": {
    "type": "<parameter-type>",
    "defaultValue": "<default-parameter-value>",
    "allowedValues": [ <array-with-permitted-parameter-values> ],
    "metadata": { 
      "key": { 
        "name": "<key-value>"
      } 
    }
  }
},
```

| 元素 | 必要 | 類型 | 說明 |  
|---------|----------|------|-------------|  
| type | 是 | int、float、string、securestring、bool、array、JSON 物件、secureobject <p><p>**注意**：對於所有密碼、金鑰和祕密，使用 `securestring` 和 `secureobject` 類型，因為 `GET` 作業不會傳回這些類型。 | 參數的類型 |
| defaultValue | 否 | 與 `type` 相同 | 在工作流程具現化時，未指定任何值時的預設參數值 | 
| allowedValues | 否 | 與 `type` 相同 | 具有參數可接受值的陣列 |  
| 中繼資料 | 否 | JSON 物件 | 任何參數詳細資訊，例如：邏輯應用程式的名稱或可讀取描述，或是 Visual Studio 或其他工具所使用的設計階段資料 |  
||||

## <a name="triggers-and-actions"></a>觸發程序及動作  

在工作流程定義中，`triggers` 和 `actions` 區段會定義在工作流程執行期間發生的呼叫。 如需這些區段的語法和詳細資訊，請參閱[工作流程觸發程序和動作](../logic-apps/logic-apps-workflow-actions-triggers.md)。
  
## <a name="outputs"></a>輸出 

在 `outputs` 區段中，定義工作流程在完成執行時可傳回的資料。 例如，若要追蹤來自每次執行的特定狀態或值，請指定工作流程輸出會傳回該資料。 

> [!NOTE]
> 當回應來自服務 REST API 的傳入要求時，請勿使用 `outputs`。 請改用 `Response` 動作類型。 如需詳細資訊，請參閱[工作流程觸發程序和動作](../logic-apps/logic-apps-workflow-actions-triggers.md)。

以下是輸出定義的一般結構： 

```json
"outputs": {
  "<key-name>": {  
    "type": "<key-type>",  
    "value": "<key-value>"  
  }
} 
```

| 元素 | 必要 | 類型 | 說明 | 
|---------|----------|------|-------------| 
| <*key-name*> | 是 | 字串 | 輸出傳回值的索引鍵名稱 |  
| type | 是 | int、float、string、securestring、bool、array、JSON 物件 | 輸出傳回值的類型 | 
| value | 是 | 與 `type` 相同 | 輸出傳回值 |  
||||| 

若要取得工作流程執行的輸出，請在 Azure 入口網站中或使用 [Workflow REST API](https://docs.microsoft.com/rest/api/logic/workflows) 檢閱邏輯應用程式的執行歷程記錄和詳細資料。 您也可以將輸出傳遞至外部系統 (例如 PowerBI)，以便建立儀表板。 

<a name="expressions"></a>

## <a name="expressions"></a>運算式

使用 JSON，您可以擁有在設計階段存在的常值，例如：

```json
"customerName": "Sophia Owen", 
"rainbowColors": ["red", "orange", "yellow", "green", "blue", "indigo", "violet"], 
"rainbowColorsCount": 7 
```

您也可以擁有在執行階段才存在的值。 若要表示這些值，您可以使用會在執行階段評估的「運算式」。 運算式是一個序列，可包含一或多個[函式](#functions)[運算子](#operators)、變數、明確值或常數。 在工作流程定義中，您可以在運算式前面加上 (\@) 符號，以便在 JSON 字串值中的任何地方使用運算式。 在評估代表 JSON 值的運算式時，移除 \@ 字元即可擷取運算式主體，而且一律會產生其他 JSON 值。 

例如，對於先前定義的 `customerName` 屬性，您可以在運算式中使用 [parameters()](../logic-apps/workflow-definition-language-functions-reference.md#parameters) 函式來取得此屬性值，並將該值指派給 `accountName` 屬性：

```json
"customerName": "Sophia Owen", 
"accountName": "@parameters('customerName')"
```

「字串內插補點」也可讓您在字串內使用多個運算式，以 \@ 字元和大括號 ({}) 圍住。 語法如下：

```json
@{ "<expression1>", "<expression2>" }
```

結果一律是字串，讓這項功能類似於 `concat()` 函式，例如： 

```json
"customerName": "First name: @{parameters('firstName')} Last name: @{parameters('lastName')}"
```

如果您有開頭為 \@ 字元的常值字串，請在 \@ 字元前面加上另一個 \@ 字元作為逸出字元：\@\@

下列範例顯示如何評估運算式：

| JSON 值 | 結果 |
|------------|--------| 
| "Sophia Owen" | 傳回這些字元：'Sophia Owen' |
| "array[1]" | 傳回這些字元：'array[1]' |
| "\@\@" | 以一個字元的字串形式傳回這些字元：'\@' |   
| " \@" | 以兩個字元的字串形式傳回這些字元：'\@' |
|||

在這些範例中，假設您定義 "myBirthMonth" 等於 "January" 以及 "myAge" 等於數字 42：  
  
```json
"myBirthMonth": "January",
"myAge": 42
```

下列範例顯示如何評估下列運算式：

| JSON 運算式 | 結果 |
|-----------------|--------| 
| "\@parameters('myBirthMonth')" | 傳回這個字串："January" |  
| "\@{parameters('myBirthMonth')}" | 傳回這個字串："January" |  
| "\@parameters('myAge')" | 傳回這個數字：42 |  
| "\@{parameters('myAge')}" | 以字串形式傳回這個數字："42" |  
| "My age is \@{parameters('myAge')}" | 傳回這個字串："My age is 42" |  
| "\@concat('My age is ', string(parameters('myAge')))" | 傳回這個字串："My age is 42" |  
| "My age is \@\@{parameters('myAge')}" | 傳回這個字串，其中包含運算式："My age is \@{parameters('myAge')}` | 
||| 

當您在 Logic Apps 設計工具中以視覺化方式運作時，您可以透過運算式產生器建立運算式，例如： 

![Logic Apps 設計工具 > 運算式產生器](./media/logic-apps-workflow-definition-language/expression-builder.png)

當您完成時，運算式會對工作流程定義中對應的屬性顯示，例如，以下的 `searchQuery` 屬性：

```json
"Search_tweets": {
  "inputs": {
    "host": {
      "connection": {
       "name": "@parameters('$connections')['twitter']['connectionId']"
      }
    }
  },
  "method": "get",
  "path": "/searchtweets",
  "queries": {
    "maxResults": 20,
    "searchQuery": "Azure @{concat('firstName','', 'LastName')}"
  }
},
```

<a name="operators"></a>

## <a name="operators"></a>運算子

在[運算式](#expressions)和[函式](#functions)中，運算子會執行特定工作，例如參考屬性或陣列中的值。 

| 運算子 | Task | 
|----------|------|
| ' | 若要將字串常值作為輸入或使用於運算式和函式中，只用單引號圍住此字串，例如 `'<myString>'`。 請勿使用雙引號 ("")，這會與整個運算式的 JSON 格式設定發生衝突。 例如︰ <p>**是**：length('Hello') </br>**否**：length("Hello") <p>當您傳遞陣列或數字時，您不需要包圍標點符號。 例如︰ <p>**是**：length([1, 2, 3]) </br>**否**：length("[1, 2, 3]") | 
| [] | 若要參考陣列中特定位置 (索引) 的值，請使用方括號。 例如，若要取得陣列中的第二個項目： <p>`myArray[1]` | 
| . | 若要參考物件中的屬性，請使用點運算子。 例如，若要取得 `customer` JSON 物件的 `name` 屬性： <p>`"@parameters('customer').name"` | 
| ? | 若要在不會發生執行階段錯誤的情況下，參考物件中的 null 屬性，請使用問號運算子。 例如，若要處理來自觸發程序的 null 輸出，您可以使用此運算式︰ <p>`@coalesce(trigger().outputs?.body?.<someProperty>, '<property-default-value>')` | 
||| 

<a name="functions"></a>

## <a name="functions"></a>Functions

某些運算式會從邏輯應用程式開始執行時還不存在的執行階段動作中取得其值。 若要在運算式中參考或使用這些值，您可以使用工作流程定義語言所提供的[*函式*](../logic-apps/workflow-definition-language-functions-reference.md)。 

## <a name="next-steps"></a>後續步驟

* 深入了解[工作流程定義語言動作和觸發程序](../logic-apps/logic-apps-workflow-actions-triggers.md)
* 深入了解如何透過 [Workflow REST API](https://docs.microsoft.com/rest/api/logic/workflows) 以程式設計方式建立及管理邏輯應用程式
