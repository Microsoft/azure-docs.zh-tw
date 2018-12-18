---
title: 處理內容類型 - Azure Logic Apps | Microsoft Docs
description: 了解 Logic Apps 如何在設計階段與執行階段處理內容類型
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
manager: jeconnoc
ms.topic: article
ms.date: 07/20/2018
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: 82eb9c895f016efe569651dc89885d2e4850fd59
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39159086"
---
# <a name="handle-content-types-in-azure-logic-apps"></a>在 Azure 邏輯應用程式中處理內容類型

不同的內容類型可以在整個邏輯應用程式中流動，例如 JSON、XML、一般檔案及二進位資料。 雖然 Logic Apps 支援所有的內容類型，但有些內容類型有原生支援，而不需要在邏輯應用程式中轉換。 有些內容類型則可能視需要進行轉換。 本文說明 Logic Apps 如何處理內容類型，以及您要如何視需要正確地轉換這些類型。

為了決定處理內容類型的適當方式，Logic Apps 會依賴 HTTP 呼叫中的 `Content-Type` 標頭值，例如：

* [application/json](#application-json) (原生類型)
* [text/plain](#text-plain) (原生類型)
* [application/xml 和 application/octet-stream](#application-xml-octet-stream)
* [其他內容類型](#other-content-types)

<a name="application-json"></a>

## <a name="applicationjson"></a>application/json

Logic Apps 會儲存和處理任何以 application/json 類型內容作為 JavaScript 標記法 (JSON) 物件的要求。 根據預設，您可以剖析 JSON 內容，而不需要任何轉換。 若要剖析具有 "application/json" 內容類型標頭的要求，您可以使用運算式。 此範例會從 `animal-type` 陣列傳回值 `dog`，而不需轉換： 
 
`@body('myAction')['animal-type'][0]` 
  
  ```json
  {
    "client": {
       "name": "Fido",
       "animal-type": [ "dog", "cat", "rabbit", "snake" ]
    }
  }
  ```

如果正在處理未指定標頭的 JSON 資料，您可以使用 [json() 函式](../logic-apps/workflow-definition-language-functions-reference.md#json)將該資料手動轉換為 JSON，例如： 
  
`@json(triggerBody())['animal-type']`

### <a name="create-tokens-for-json-properties"></a>建立 JSON 屬性的權杖

Logic Apps 可讓您產生方便使用的權杖來表示 JSON 內容中的屬性，因此您可以在邏輯應用程式的工作流程中更輕鬆地參考和使用這些屬性。

* **要求觸發程序**

  當您在邏輯應用程式設計工具中使用此觸發程序時，您可以提供 JSON 結構描述來描述您預期收到的承載。 
  此設計工具會使用此結構描述來剖析 JSON 內容，並產生方便使用的權杖來表示 JSON 內容中的屬性。 
  您可以在邏輯應用程式的整個工作流程中輕鬆地參考和使用這些屬性。 
  
  如果您沒有結構描述，您可以產生結構描述。 
  
  1. 在要求觸發程序中，選取 [使用範例承載來產生結構描述]。  
  
  2. 在 [輸入或貼上範例 JSON 承載] 之下，提供範例承載，然後選擇 [完成]。 例如︰ 

     ![提供範例 JSON 承載](./media/logic-apps-content-type/request-trigger.png)

     產生的結構描述現在會出現在觸發程序中。

     ![提供範例 JSON 承載](./media/logic-apps-content-type/generated-schema.png)

     以下是在程式碼檢視編輯器中要求觸發程序的基礎定義：

     ```json
     "triggers": { 
        "manual": {
           "type": "Request",
           "kind": "Http",
           "inputs": { 
              "schema": {
                 "type": "object",
                 "properties": {
                    "client": {
                       "type": "object",
                       "properties": {
                          "animal-type": {
                             "type": "array",
                             "items": {
                                "type": "string"
                             },
                          },
                          "name": {
                             "type": "string"
                          }
                       }
                    }
                 }
              }
           }
        }
     }
     ```

  3. 在您的要求中，確定包含 `Content-Type` 標頭並將標頭的值設定為 `application/json`。

* **剖析 JSON 動作**

  當您在邏輯應用程式設計工具中使用此動作時，您可以剖析 JSON 輸出並產生方便使用的權杖來表示 JSON 內容中的屬性。 
  您可以在邏輯應用程式的整個工作流程中輕鬆地參考和使用這些屬性。 這類似於要求觸發程序，您可以提供或產生 JSON 結構描述來描述所要剖析的 JSON 內容。 
  這麼一來，您可以更輕鬆地從 Azure 服務匯流排、Azure Cosmos DB 等取用資料。

  ![剖析 JSON](./media/logic-apps-content-type/parse-json.png)

<a name="text-plain"></a>

## <a name="textplain"></a>text/plain

當邏輯應用程式接收 `Content-Type` 標頭設定為`text/plain`的 HTTP 訊息時，邏輯應用程式會以原始格式來儲存這些訊息。 如果您在後續動作中納入這些訊息 (但未轉換)，要求就會與設定為 `text/plain` 的 `Content-Type` 標頭一同送出。 

例如，當您處理一般檔案時，您可能會收到 `Content-Type` 標頭設定為 `text/plain` 內容類型的 HTTP 要求：

`Date,Name,Address`</br>
`Oct-1,Frank,123 Ave`

如果您接著在後續動作中傳送此要求以作為另一個要求的本文 (例如 `@body('flatfile')`)，該第二個要求也會有設定為 `text/plain` 的 `Content-Type` 標頭。 如果正在處理格式為純文字但並未指定標頭的資料，您可以使用 [string() 函式](../logic-apps/workflow-definition-language-functions-reference.md#string) (如以下運算式)，將該資料手動轉換為文字： 

`@string(triggerBody())`

<a name="application-xml-octet-stream"></a>

## <a name="applicationxml-and-applicationoctet-stream"></a>application/xml 和 application/octet-stream

Logic Apps 一律會在收到的 HTTP 要求或回應中保留 `Content-Type`。 因此，如果邏輯應用程式收到 `Content-Type` 設定為 `application/octet-stream` 的內容，而且您在後續動作中納入該內容，則外送要求的 `Content-Type` 也會設定為 `application/octet-stream`。 這麼一來，Logic Apps 就可以保證資料在整個工作流程中移動時不會遺失。 不過，當動作狀態 (或輸入和輸出) 通過工作流程時，該狀態會儲存於 JSON 物件中。 

## <a name="converter-functions"></a>轉換器函式

為了保留某些資料類型，Logic Apps 會將內容轉換為二進位 Base64 編碼的字串，其中包含可保留 `$content` 承載和 `$content-type` (都會自動轉換) 的適當中繼資料。 

以下清單說明當您使用這些[函式](../logic-apps/workflow-definition-language-functions-reference.md)時，Logic Apps 如何轉換內容：

* `json()`：將資料轉換為 `application/json`
* `xml()`：將資料轉換為 `application/xml`
* `binary()`：將資料轉換為 `application/octet-stream`
* `string()`：將資料轉換為 `text/plain`
* `base64()`：將內容轉換為 Base64 字串
* `base64toString()`：將 Base64 編碼的字串轉換為 `text/plain`
* `base64toBinary()`：將 Base64 編碼的字串轉換為 `application/octet-stream`
* `encodeDataUri()`：將字串編碼為 dataUri 位元組陣列
* `decodeDataUri()`：將 `dataUri` 解碼為位元組陣列

例如，如果您收到 `Content-Type` 設定為 `application/xml` 的 HTTP 要求，例如以下內容：

```html
<?xml version="1.0" encoding="UTF-8" ?>
<CustomerName>Frank</CustomerName>
```

您可以使用 `@xml(triggerBody())` 運算式搭配 `xml()` 和 `triggerBody()` 函式來轉換此內容，稍後再使用此內容。 或者，您可以使用 `@xpath(xml(triggerBody()), '/CustomerName')` 運算式搭配 `xpath()` 和 `xml()` 函式。 

## <a name="other-content-types"></a>其他內容類型

Logic Apps 可處理和支援其他內容類型，但可能會要求您將 `$content` 變數解碼來手動取得訊息內文。

例如，假設邏輯應用程式是由具有 `application/x-www-url-formencoded` 內容類型的要求觸發。 若要保留所有資料，則要求本文中的`$content` 變數具有編碼為 base64 字串的承載：

`CustomerName=Frank&Address=123+Avenue`

因為要求不是純文字或 JSON，所以要求會儲存在動作中，如下所示︰

```json
"body": {
   "$content-type": "application/x-www-url-formencoded",
   "$content": "AAB1241BACDFA=="
}
```

Logic Apps 會提供可供處理表單資料的原生函式，例如： 

* [triggerFormDataValue()](../logic-apps/workflow-definition-language-functions-reference.md#triggerFormDataValue)
* [triggerFormDataMultiValues()](../logic-apps/workflow-definition-language-functions-reference.md#triggerFormDataMultiValues)
* [formDataValue()](../logic-apps/workflow-definition-language-functions-reference.md#formDataValue) 
* [formDataMultiValues()](../logic-apps/workflow-definition-language-functions-reference.md#formDataMultiValues)

或者，您可以使用如以下範例所示的運算式，手動存取資料：

`@string(body('formdataAction'))` 

如果您希望外送要求具有相同的 `application/x-www-url-formencoded` 內容類型標頭，您可以使用 `@body('formdataAction')` 之類的運算式，將要求新增到動作內文，而不需使用任何轉換。 不過，唯有當內文是 `body` 輸入中的唯一參數時，才適用這種方法。 如果您嘗試在 `application/json` 要求中使用 `@body('formdataAction')` 運算式，您會因為傳送已編碼的內文而發生執行階段錯誤。
