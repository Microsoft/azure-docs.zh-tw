---
title: 錯誤和例外狀況處理 - Azure Logic Apps | Microsoft Docs
description: 了解 Azure Logic Apps 中的錯誤和例外狀況處理模式
services: logic-apps
ms.service: logic-apps
author: dereklee
ms.author: deli
manager: jeconnoc
ms.date: 01/31/2018
ms.topic: article
ms.reviewer: klam, LADocs
ms.suite: integration
ms.openlocfilehash: 7ce5c7007414bfe8e17727c25de9712e7993dc1e
ms.sourcegitcommit: a5eb246d79a462519775a9705ebf562f0444e4ec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39263747"
---
# <a name="handle-errors-and-exceptions-in-azure-logic-apps"></a>處理 Azure Logic Apps 中的錯誤和例外狀況

適當地處理相依系統所造成的停機情況或問題，對任何整合架構來說都是種挑戰。 為了協助您建立穩固且具有恢復能力的整合方案來處理問題與失敗，Logic Apps 提供了一流的錯誤和例外狀況處理體驗。 

<a name="retry-policies"></a>

## <a name="retry-policies"></a>重試原則

對於大部分基本的例外狀況和錯誤處理，您可以在支援的動作或觸發程序中使用「重試原則」。 此重試原則會定義當原始要求逾時或失敗時 (也就是，任何造成 408、429 或 5xx 回應的要求)，動作或觸發程序是否要重試要求，以及要如何重試。 如果未使用任何其他重試原則，則會使用預設原則。 

以下是重試原則的類型： 

| 類型 | 說明 | 
|------|-------------| 
| [**預設值**](#default-retry) | 此原則會以[*指數漸增*](#exponential-retry)間隔 (依 7.5 秒調整，但限制在 5 到 45 秒之間) 傳送最多 4 次重試。 | 
| [**指數間隔**](#exponential-retry)  | 此原則會先等待選自指數成長範圍內的隨機間隔時間，再傳送下一個要求。 | 
| [**固定間隔**](#fixed-retry)  | 此原則會先等待指定的間隔時間，再傳送下一個要求。 | 
| [**無**](#no-retry)  | 不重新傳送要求。 | 
||| 

如需有關重試原則限制的資訊，請參閱 [Logic Apps 限制和設定](../logic-apps/logic-apps-limits-and-config.md#request-limits)。 

### <a name="change-retry-policy"></a>變更重試原則

若要選取不同的重試原則，請遵循下列步驟： 

1. 在邏輯應用程式設計工具中開啟邏輯應用程式。 

2. 開啟動作或觸發程序的 [設定]。

3. 如果動作或觸發程序支援重試原則，請於 [重試原則] 底下選取您想要的類型。 

或者，您也可以針對支援重試原則的動作或觸發程序，在其 `inputs` 區段中手動指定重試原則。 如果您未指定重試原則，則動作會使用預設原則。

```json
"<action-name>": {
   "type": "<action-type>", 
   "inputs": {
      "<action-specific-inputs>",
      "retryPolicy": {
         "type": "<retry-policy-type>",
         "interval": "<retry-interval>",
         "count": <retry-attempts>,
         "minimumInterval": "<minimum-interval>",
         "maximumInterval": "<maximun-interval>"
      },
      "<other-action-specific-inputs>"
   },
   "runAfter": {}
}
```

*必要*

| 值 | 類型 | 說明 |
|-------|------|-------------|
| <retry-policy-type> | 字串 | 您想要使用的重試原則類型：「預設值」、「無」、「固定」或「指數」 | 
| <retry-interval> | 字串 | 值必須使用 [ISO 8601 格式](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations)的重試間隔。 預設最小間隔是 `PT5S`，最大間隔則是 `PT1D`。 當您使用指數間隔原則時，您可以指定不同的最小和最大值。 | 
| <retry-attempts> | 整數  | 重試次數必須介於 1 到 90 之間 | 
||||

*選擇性*

| 值 | 類型 | 說明 |
|-------|------|-------------|
| <minimum-interval> | 字串 | 對於指數間隔原則，此為隨機選取間隔的最小間隔，且採用 [ISO 8601 格式](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) | 
| <maximum-interval> | 字串 | 對於指數間隔原則，此為隨機選取間隔的最大間隔，且採用 [ISO 8601 格式](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) | 
|||| 

以下是不同原則類型的詳細資訊。

<a name="default-retry"></a>

### <a name="default"></a>預設值

若您未指定重試原則，則動作會使用預設原則 (實際上就是[指數間隔原則](#exponential-interval))，此原則會以指數漸增間隔 (以 7.5 秒作調整) 傳送最多四次重試。 間隔的最小值與最大值為 5 秒和 45 秒。 

雖然動作或觸發程序中未明確定義，但以下是 HTTP 動作範例中的預設原則行為：

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "http://myAPIendpoint/api/action",
      "retryPolicy" : {
         "type": "exponential",
         "interval": "PT7S",
         "count": 4,
         "minimumInterval": "PT5S",
         "maximumInterval": "PT1H"
      }
   },
   "runAfter": {}
}
```

### <a name="none"></a>None

若要將動作或觸發程序指定為不會重試失敗的要求，請將 <retry-policy-type> 設定為 `none`。

### <a name="fixed-interval"></a>固定間隔

若要將動作或觸發程序指定為先等候指定間隔再傳送下一個要求，請將 <retry-policy-type> 設定為 `fixed`。

*範例*

此重試原則會嘗試在第一次失敗要求後再取得最新消息兩次，每次嘗試之間有 30 秒的延遲：

```json
"Get_latest_news": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "https://mynews.example.com/latest",
      "retryPolicy": {
         "type": "fixed",
         "interval": "PT30S",
         "count": 2
      }
   }
}
```

<a name="exponential-interval"></a>

### <a name="exponential-interval"></a>指數間隔

若要將動作或觸發程序指定為先等候隨機間隔再傳送下一個要求，請將 <retry-policy-type> 設定為 `exponential`。 隨機間隔會從指數成長範圍來選取。 (選擇性) 您也可以藉由指定自己的最小和最大間隔，覆寫預設的最小和最大間隔。

**隨機變數範圍**

此資料表說明 Logic Apps 如何針對每次重試 (最高可達重試次數 (含))，產生指定範圍內的統一隨機變數：

| 重試數目 | 最小間隔 | 最大間隔 |
|--------------|------------------|------------------|
| 1 | max(0, <minimum-interval>) | min(interval, <maximum-interval>) |
| 2 | max(interval, <minimum-interval>) | min(2 * interval, <maximum-interval>) |
| 3 | max(2 * interval, <minimum-interval>) | min(4 * interval, <maximum-interval>) |
| 4 | max(4 * interval, <minimum-interval>) | min(8 * interval, <maximum-interval>) |
| .... | .... | .... | 
|||| 

## <a name="catch-and-handle-failures-with-the-runafter-property"></a>使用 RunAfter 屬性來擷取和處理失敗

每個邏輯應用程式動作都會在動作開始之前，宣告必須先完成的一些動作，就類似於在工作流程中指定步驟順序的方式。 在動作定義中，**runAfter** 屬性會定義此順序，並且是描述哪些動作和動作狀態會執行動作的物件。

根據預設，所有您在「邏輯應用程式設計工具」中新增的動作都會設定為在前一個步驟執行後才執行，但前提是前一個步驟結果為 **Succeeded (成功)**。 不過，您可以自訂 **runAfter** 值，讓動作可以在上一個動作結果為 **Failed (失敗)**、**Skipped (略過)** 或這些值的一些組合時引發。 例如，若要在特定的 **Insert_Row** 動作失敗後，將項目新增至特定的服務匯流排主題，您可以使用此 **runAfter** 定義範例：

```json
"Send_message": {
    "inputs": {
        "body": {
            "ContentData": "@{encodeBase64(body('Insert_Row'))}",
            "ContentType": "{ \"content-type\" : \"application/json\" }"
        },
        "host": {
            "api": {
                "runtimeUrl": "https://logic-apis-westus.azure-apim.net/apim/servicebus"
            },
            "connection": {
                "name": "@parameters('$connections')['servicebus']['connectionId']"
            }
        },
        "method": "post",
        "path": "/@{encodeURIComponent('failures')}/messages"
    },
    "runAfter": {
        "Insert_Row": [
            "Failed"
        ]
    }
}
```

當 **Insert_Row** 動作狀態為 **Failed (失敗)** 時，**RunAfter** 屬性會設為執行。 若要在動作狀態為 **Succeeded**、**Failed** 或 **Skipped** 時執行動作，請使用此語法︰

```json
"runAfter": {
        "Insert_Row": [
            "Failed", "Succeeded", "Skipped"
        ]
    }
```

> [!TIP]
> 在前一個動作失敗後所執行並順利完成的動作會標示為 **Succeeded (成功)**。 此行為表示如果您成功擷取到工作流程中的所有失敗，該次執行本身會標示為 **Succeeded (成功)**。

<a name="scopes"></a>

## <a name="evaluate-actions-with-scopes-and-their-results"></a>以範圍和其結果評估動作

類似於使用 **runAfter** 屬性在個別動作之後執行步驟，您可以在 [scope (範圍)](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md) 內將動作群組在一起。 當您想要以邏輯方式將動作群組在一起、評估範圍的彙總狀態，並根據這些狀態執行動作時，即可使用範圍。 當範圍中的所有動作都執行完成之後，範圍本身會取得自己的狀態。 

若要檢查範圍的狀態，您可以使用檢查邏輯應用程式執行狀態時所用的相同準則，例如 **Succeeded (成功)**、**Failed (失敗)** 等。 

根據預設，當範圍的所有動作都成功時，範圍的狀態會標示為 **Succeeded (成功)**。 如果範圍內的最後動作結果為 **Failed (失敗)** 或 **Aborted (中止)**，範圍的狀態會標示 **Failed (失敗)**。 

若要攔截 **Failed (失敗)** 範圍的例外狀況，以及執行處理這些錯誤的動作，您可以針對該 **Failed (失敗)** 範圍使用 **runAfter** 屬性。 這樣一來，如果範圍內的「任何」動作失敗，而您對該範圍使用 **runAfter** 屬性，您便可建立單一動作來擷取失敗。

如需範圍的限制，請參閱[限制和設定](../logic-apps/logic-apps-limits-and-config.md)。

### <a name="get-context-and-results-for-failures"></a>取得失敗的內容和結果

雖然從範圍擷取失敗很實用，但還是建議您取得內容以協助解實際上有哪些動作失敗，以及所傳回的任何錯誤或狀態碼。 @result() 運算式會提供範圍內所有動作結果的相關內容。

@result() 運算式會接受單一參數 (範圍名稱)，並傳回該範圍內所有動作結果的陣列。 這些動作物件包含與 **@actions()** 物件相同的屬性，例如動作的開始時間、結束時間、狀態、輸入、相互關聯識別碼和輸出。 若要傳送在某個範圍內失敗之任何動作的內容，您可以輕鬆地將 **@result()** 函式與 **runAfter** 屬性配對。

若要針對範圍中結果為 **Failed** 的每個動作執行動作，以及篩選結果陣列以顯示失敗的動作，您可以將 **@result()** 與**[篩選陣列](../connectors/connectors-native-query.md)** 動作及 [**For each**](../logic-apps/logic-apps-control-flow-loops.md) 迴圈配對。 您可以取得篩選後的結果陣列，並使用 **For each** 迴圈對每個失敗執行動作。 

以下範例 (隨後並有詳細說明) 會傳送 HTTP POST 要求，其中含有任何在 "My_Scope" 範圍內失敗之動作的回應本文：

```json
"Filter_array": {
   "type": "Query",
   "inputs": {
      "from": "@result('My_Scope')",
      "where": "@equals(item()['status'], 'Failed')"
   },
   "runAfter": {
      "My_Scope": [
         "Failed"
      ]
    }
},
"For_each": {
   "type": "foreach",
   "actions": {
      "Log_exception": {
         "type": "Http",
         "inputs": {
            "method": "POST",
            "body": "@item()['outputs']['body']",
            "headers": {
               "x-failed-action-name": "@item()['name']",
               "x-failed-tracking-id": "@item()['clientTrackingId']"
            },
            "uri": "http://requestb.in/"
         },
         "runAfter": {}
      }
   },
   "foreach": "@body('Filter_array')",
   "runAfter": {
      "Filter_array": [
         "Succeeded"
      ]
   }
}
```

以下是說明此範例中所發生情況的詳細逐步解說︰

1. 若要從 "My_Scope" 內的所有動作取得結果，**篩選陣列**動作會使用這個篩選條件運算式："@result('My_Scope')"

2. **篩選陣列**的條件是任何狀態等於 **Failed** 的 "@result()" 項目。 此條件會將具有 "My_Scope" 所有動作結果的陣列篩選成只剩失敗動作結果的陣列。

3. 對篩選後陣列的輸出執行 **For each** 迴圈動作。 此步驟會對之前篩選的每個失敗動作結果執行動作。

   如果範圍中的單一動作失敗，**For each** 迴圈中的動作只會執行一次。 
   如果有多個失敗動作，則會導致對每個失敗執行一個動作。

4. 在 **For each** 項目回應本文 (即 "@item()['outputs']['body']" 運算式) 上傳送 HTTP POST。 

   "@result()" 項目圖形和 "@actions()" 圖形相同，並可透過相同方式剖析。

5. 包含兩個具有失敗動作名稱 ("@item()['name']") 和失敗執行用戶端追蹤識別碼 ("@item()['clientTrackingId']") 的自訂標頭。

供您參考，以下範例是單一 "@result()" 項目，會顯示前一個範例中剖析的 **name**、**body**和 **clientTrackingId** 屬性。 在 **For each** 動作之外，"@result()" 會傳回這些物件的陣列。

```json
{
   "name": "Example_Action_That_Failed",
   "inputs": {
      "uri": "https://myfailedaction.azurewebsites.net",
      "method": "POST"
   },
   "outputs": {
      "statusCode": 404,
      "headers": {
         "Date": "Thu, 11 Aug 2016 03:18:18 GMT",
         "Server": "Microsoft-IIS/8.0",
         "X-Powered-By": "ASP.NET",
         "Content-Length": "68",
         "Content-Type": "application/json"
      },
      "body": {
         "code": "ResourceNotFound",
         "message": "/docs/folder-name/resource-name does not exist"
      }
   },
   "startTime": "2016-08-11T03:18:19.7755341Z",
   "endTime": "2016-08-11T03:18:20.2598835Z",
   "trackingId": "bdd82e28-ba2c-4160-a700-e3a8f1a38e22",
   "clientTrackingId": "08587307213861835591296330354",
   "code": "NotFound",
   "status": "Failed"
}
```

若要執行不同的例外狀況處理模式，您可以使用本文前面描述的運算式。 您可以選擇在範圍外執行單一例外狀況處理動作，以接受整個篩選後的失敗陣列，並移除 **For each** 動作。 您也可以包含 **@result()** 回應中的其他有用屬性，如先前所述。

## <a name="azure-diagnostics-and-metrics"></a>Azure 診斷和計量

上述模式非常適合處理執行內的錯誤和例外狀況，但您也可以識別和回應與執行本身無關的錯誤。 
[Azure 診斷](../logic-apps/logic-apps-monitor-your-logic-apps.md) 提供一個簡單的方法，可讓您將所有工作流程事件 (包括所有執行和動作狀態) 傳送至 Azure 儲存體帳戶或使用「Azure 事件中樞」建立的某個事件中樞。 

若要評估執行狀態，您可以監視記錄和度量，或將它們發佈至您偏好使用的任何監視工具。 其中一個可能的選項是透過「事件中樞」將所有事件串流到 [Azure 串流分析](https://azure.microsoft.com/services/stream-analytics/)。 在「串流分析」中，您可以根據診斷記錄檔中的任何異常、平均或失敗，撰寫即時查詢。 您可以使用「串流分析」將資訊傳送到其他資料來源，例如傳送到佇列、主題、SQL、Azure Cosmos DB 或 Power BI。

## <a name="next-steps"></a>後續步驟

* [看看客戶如何使用 Azure Logic Apps 建置錯誤處理](../logic-apps/logic-apps-scenario-error-and-exception-handling.md)
* [尋找其他 Logic Apps 範例和案例](../logic-apps/logic-apps-examples-and-scenarios.md)

<!-- References -->
[retryPolicyMSDN]: https://docs.microsoft.com/rest/api/logic/actions-and-triggers#Anchor_9