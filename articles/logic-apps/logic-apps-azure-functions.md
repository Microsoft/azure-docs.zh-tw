---
title: 使用 Azure Functions 在 Azure Logic Apps 中新增和執行自訂程式碼 | Microsoft Docs
description: 了解如何使用 Azure Functions 在 Azure Logic Apps 中新增和執行自訂程式碼片段
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
manager: jeconnoc
ms.topic: article
ms.date: 08/20/2018
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: a63bd8e3b071ed996db8ad5aeaeb5e451b4d92e9
ms.sourcegitcommit: 974c478174f14f8e4361a1af6656e9362a30f515
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42144036"
---
# <a name="add-and-run-custom-code-snippets-in-azure-logic-apps-with-azure-functions"></a>使用 Azure Functions 在 Azure Logic Apps 中新增和執行自訂程式碼片段

當您想要執行剛剛好能夠執行邏輯應用程式中特定作業的程式碼時，您可以使用 [Azure Functions](../azure-functions/functions-overview.md) 建立自己的函式。 這項服務可協助您建立 Node.js、C# 和 F # 程式碼片段，因此您不必建置完整的應用程式或基礎結構來執行程式碼。 Azure Functions 可提供在雲端進行的無伺服器運算，適合用來執行以下舉例的各種工作：

* 使用 Node.js 或 C# 中的函式來擴充邏輯應用程式的行為。
* 在邏輯應用程式工作流程中執行計算。
* 在邏輯應用程式中套用進階格式設定或計算欄位。

您也可以[從 Azure 函式內呼叫邏輯應用程式](#call-logic-app)。

## <a name="prerequisites"></a>必要條件

若要遵循本文，您需要這些項目：

* 如果您還沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* Azure 函式應用程式 (也就是，Azure 函式的容器) 和 Azure 函式。 如果您沒有函式應用程式，[請先建立該函式應用程式](../azure-functions/functions-create-first-azure-function.md)。 然後，您可以在邏輯應用程式設計工具中建立函式，不論是要[在邏輯應用程式外另外](#create-function-external)建立，還是要[從邏輯應用程式內部](#create-function-designer)建立都行。

  現有的和新的函式應用程式與函式在使用邏輯應用程式上有相同的需求：

  * 函式應用程式必須有與邏輯應用程式相同的 Azure 訂用帳戶。

  * 您的函式會使用 HTTP 觸發程序，例如，適用於 **JavaScript**或 **C#** 的 **HTTP 觸發程序**函式範本。 

    HTTP 觸發程序範本可以接受內容中有來自邏輯應用程式的 `application/json` 類型。 
    當您將 Azure 函式新增至邏輯應用程式時，邏輯應用程式設計工具會顯示 Azure 訂用帳戶內透過此範本建立的自訂函式。 

  * 您的函式不會使用自訂路由，除非您已定義 [OpenAPI 定義](../azure-functions/functions-openapi-definition.md) (先前稱為 [Swagger 檔案](http://swagger.io/))。 
  
  * 如果您已為函式定義 OpenAPI 定義，邏輯應用程式設計工具會在使用函式參數上提供更豐富的體驗。 邏輯應用程式若要尋找和存取具有 OpenAPI 定義的函式，必須先[遵循下列步驟來設定函式應用程式](#function-swagger)。

* 您要在其中新增函式的邏輯應用程式，並包括作為邏輯應用程式中第一個步驟的[觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts) 

  邏輯應用程式必須以觸發程序作為開頭，才能新增可執行函式的動作。

  如果您還不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)和[快速入門：建立第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

<a name="create-function-external"></a>

## <a name="create-functions-outside-logic-apps"></a>在邏輯應用程式外部建立函式

在 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>中建立 Azure 函式應用程式 (必須有和邏輯應用程式相同的 Azure 訂用帳戶)，然後建立 Azure 函式。
如果您還不熟悉如何建立 Azure Functions，請了解如何[在 Azure 入口網站中建立您的第一個函式](../azure-functions/functions-create-first-azure-function.md)，但請注意下列需求，滿足後才能建立可從邏輯應用程式呼叫的函式：

* 確定您有選取適用於 **JavaScript** 或 **C#** 的 **HTTP 觸發程序**函式範本。

  ![HTTP 觸發程序 - JavaScript 或 C#](./media/logic-apps-azure-functions/http-trigger-function.png)

<a name="function-swagger"></a>

* (選擇性) 如果您為函式[產生 API 定義](../azure-functions/functions-openapi-definition.md) (先前稱為 [Swagger 檔案](http://swagger.io/))，當您在邏輯應用程式設計工具中使用函式參數時將可獲得更豐富的體驗。 若要設定函式應用程式，讓邏輯應用程式可以尋找和使用具有 Swagger 描述的函式，請遵循下列步驟：

  1. 確定函式應用程式正在執行。

  2. 若要在函式應用程式中設定[跨原始資源共用 (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)，以允許所有來源，請遵循下列步驟：

     1. 從 [函式應用程式] 清單中，選取您的函式應用程式 > [平台功能] > [CORS]。

        ![選取您的函式應用程式 > [平台功能] > [CORS]](./media/logic-apps-azure-functions/function-platform-features-cors.png)

     2. 在 [CORS] 底下新增 `*` 萬用字元，但在清單中移除所有其他來源，然後選擇 [儲存]。

        ![將 "CORS* 設為萬用字元 "*"](./media/logic-apps-azure-functions/function-platform-features-cors-origins.png)

### <a name="access-property-values-inside-http-requests"></a>存取 HTTP 要求內的屬性值

Webhook 函式可以透過輸入接受 HTTP 要求，並將這些要求傳遞給其他函式。 例如，雖然 Logic Apps 有[會轉換日期時間值的函式](../logic-apps/workflow-definition-language-functions-reference.md)，但這個基本範例 JavaScript 函式會說明如何存取傳遞給函式的要求物件內所含的屬性，並對該屬性值執行作業。 為了存取物件內的屬性，這個範例使用[點 (.) 運算子](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Property_accessors)： 

```javascript
function convertToDateString(request, response){
   var data = request.body;
   response = {
      body: data.date.ToDateString();
   }
}
```

以下是此函式內部的運作情形：

1. 函式會建立 `data` 變數，並將 `request` 物件內的 `body` 物件指派給該變數。 此函式會使用點 (.) 運算子來參考 `request` 物件內的 `body` 物件： 

   ```javascript
   var data = request.body;
   ```

2. 此函式現在可以透過 `data` 變數存取 `date` 屬性，並藉由呼叫 `ToDateString()` 函式將該屬性值從 DateTime 類型轉換為 DateString 類型。 此函式也會透過函式回應中的 `body` 屬性傳回結果： 

   ```javascript
   body: data.date.ToDateString();
   ```

您已建立 Azure 函式，接下來請遵循如何[將函式新增至邏輯應用程式](#add-function-logic-app)的步驟。

<a name="create-function-designer"></a>

## <a name="create-functions-inside-logic-apps"></a>在邏輯應用程式內部建立函式

若要在邏輯應用程式設計工具中從邏輯應用程式內部開始建立 Azure 函式，您必須先擁有 Azure 函式應用程式，以作為函式的容器。 如果您沒有函式應用程式，請先建立該函式應用程式。 請參閱[在 Azure 入口網站中建立您的第一個函式](../azure-functions/functions-create-first-azure-function.md)。 

1. 在 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>的邏輯應用程式設計工具中，開啟邏輯應用程式。 

2. 若要建立並新增函式，請遵循適用於您案例的步驟：

   * 在邏輯應用程式工作流程的最後一個步驟中，選擇 [新增步驟]。

   * 在邏輯應用程式工作流程的現有步驟之間，將滑鼠移到箭號，並選擇加號 (+)，然後選取 [新增動作]。

3. 在搜尋方塊中，輸入「azure functions」作為篩選條件。
從動作清單中選取此動作：**選擇 Azure 函式 – Azure Functions** 

   ![尋找 "Azure functions"](./media/logic-apps-azure-functions/find-azure-functions-action.png)

4. 從函式應用程式清單中，選取您的函式應用程式。 動作清單開啟後，請選取此動作：**Azure Functions - 建立新函式**

   ![選取函式應用程式](./media/logic-apps-azure-functions/select-function-app-create-function.png)

5. 在函式定義編輯器中，定義您的函式：

   1. 在 [函式名稱] 方塊中，為您的函式提供名稱。 

   2. 在 [程式碼] 方塊中，於函式範本內新增程式碼，包括要在函式執行完成後傳回給邏輯應用程式的回應和承載。 

      ![定義函式](./media/logic-apps-azure-functions/function-definition.png)

      在範本的程式碼中，`context` 物件代表邏輯應用程式透過**要求本文** (位於後面的步驟中) 欄位傳送的訊息。 
      若要從您的函式內存取 `context` 物件的屬性，請使用此語法： 

      `context.body.<property-name>`

      例如，若要參考 `context` 物件內的 `content` 屬性，請使用此語法： 

      `context.body.content`

      範本程式碼也包含 `input` 變數，其中儲存著來自 `data` 參數的值，因此函式可以根據該值執行作業。 
      在 JavaScript 函數中，`data` 變數也是 `context.body` 的捷徑。

      > [!NOTE]
      > 這裡的 `body` 屬性適用於 `context` 物件，與動作輸出中可能也會傳遞給函式的**本文**權杖不同。 
 
   3. 完成之後，請選擇 [建立]。

6. 在 [要求本文] 方塊中，提供函式的輸入 (必須以 JavaScript 物件標記法 (JSON) 物件進行格式化)。 

   此輸入是邏輯應用程式傳送給函式的「內容物件」或訊息。 當您在 [要求本文] 欄位內按一下時，動態內容清單會隨即出現，讓您可以選取前面步驟中的輸出權杖。 此範例會指明內容承載包含名為 `content` 的屬性，而此屬性具有電子郵件觸發程序中的**寄件者**權杖：

   ![「要求本文」範例 - 內容物件承載](./media/logic-apps-azure-functions/function-request-body-example.png)

   在此處，內容物件不會轉換為字串，所以物件內容會直接新增至 JSON 承載。 不過，如果內容物件不是傳遞字串、JSON 物件或 JSON 陣列的 JSON 權杖，您就會收到錯誤。 因此，如果此範例改為使用**接收時間**權仗，則您可以在內容物件上加上雙引號，將內容物件轉換為字串：  

   ![將物件轉換為字串](./media/logic-apps-azure-functions/function-request-body-string-cast-example.png)

7. 若要指定其他詳細資料 (例如，要使用的方法、要求標頭或查詢參數)，請選擇 [顯示進階選項]。

<a name="add-function-logic-app"></a>

## <a name="add-existing-functions-to-logic-apps"></a>在邏輯應用程式中新增現有函式

若要從邏輯應用程式呼叫現有 Azure 函式，您可以在邏輯應用程式設計工具中新增 Azure 函式，例如任何其他動作。 

1. 在 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>的邏輯應用程式設計工具中，開啟邏輯應用程式。 

2. 在想要新增函式的步驟底下，選擇 [新增步驟] > [新增動作]。 

3. 在搜尋方塊中，輸入「azure functions」作為篩選條件。
從動作清單中選取此動作：**選擇 Azure 函式 – Azure Functions** 

   ![尋找 "Azure functions"](./media/logic-apps-azure-functions/find-azure-functions-action.png)

4. 從函式應用程式清單中，選取您的函式應用程式。 函式清單出現後，選取您的函式。 

   ![選取函式應用程式和 Azure 函式](./media/logic-apps-azure-functions/select-function-app-existing-function.png)

   函式若具有 API 定義 (Swagger 描述)，並已[進行設定以讓邏輯應用程式可以尋找和存取這些函式](#function-swagger)，您就可以選取 [Swagger 動作]：

   ![選取函式應用程式、「Swagger 動作」和 Azure 函式](./media/logic-apps-azure-functions/select-function-app-existing-function-swagger.png)

5. 在 [要求本文] 方塊中，提供函式的輸入 (必須以 JavaScript 物件標記法 (JSON) 物件進行格式化)。 

   此輸入是邏輯應用程式傳送給函式的「內容物件」或訊息。 當您在 [要求本文] 欄位內按一下時，動態內容清單會隨即出現，讓您可以選取前面步驟中的輸出權杖。 此範例會指明內容承載包含名為 `content` 的屬性，而此屬性具有電子郵件觸發程序中的**寄件者**權杖：

   ![「要求本文」範例 - 內容物件承載](./media/logic-apps-azure-functions/function-request-body-example.png)

   在此處，內容物件不會轉換為字串，所以物件內容會直接新增至 JSON 承載。 不過，如果內容物件不是傳遞字串、JSON 物件或 JSON 陣列的 JSON 權杖，您就會收到錯誤。 因此，如果此範例改為使用**接收時間**權仗，則您可以在內容物件上加上雙引號，將內容物件轉換為字串： 

   ![將物件轉換為字串](./media/logic-apps-azure-functions/function-request-body-string-cast-example.png)

6. 若要指定其他詳細資料 (例如，要使用的方法、要求標頭或查詢參數)，請選擇 [顯示進階選項]。

<a name="call-logic-app"></a>

## <a name="call-logic-apps-from-functions"></a>從函式呼叫邏輯應用程式

若要從 Azure 函式內觸發邏輯應用程式，該邏輯應用程式必須以能提供可呼叫端點的觸發程序作為開頭。 例如，您可以使用 **HTTP**、**要求**、**Azure 佇列**或**事件方格**觸發程序來起始邏輯應用程式。 在函式內部，將 HTTP POST 要求傳送給該觸發程序的 URL，並包含您想要讓該邏輯應用程式處理的承載。 如需詳細資訊，請參閱[呼叫、觸發或巢狀邏輯應用程式](../logic-apps/logic-apps-http-endpoint.md)。 

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解[Logic Apps 連接器](../connectors/apis-list.md)
