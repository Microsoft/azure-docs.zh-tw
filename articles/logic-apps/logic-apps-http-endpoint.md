---
title: 透過 HTTP 端點呼叫、觸發或巢狀處理工作流程 - Azure Logic Apps | Microsoft Docs
description: 設定 HTTP 端點來呼叫、觸發或巢狀處理 Azure Logic Apps 的工作流程
services: logic-apps
keywords: 工作流程, HTTP 端點
author: jeffhollan
manager: jeconnoc
editor: ''
documentationcenter: ''
ms.assetid: 73ba2a70-03e9-4982-bfc8-ebfaad798bc2
ms.service: logic-apps
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.custom: H1Hack27Feb2017
ms.date: 03/31/2017
ms.author: klam; LADocs
ms.openlocfilehash: 9c02a0f540f52007412a850b9d69e337743fc35f
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35300193"
---
# <a name="call-trigger-or-nest-workflows-with-http-endpoints-in-logic-apps"></a>在邏輯應用程式中透過 HTTP 端點呼叫、觸發或巢狀處理工作流程

您可以在邏輯應用程式中利用原生方式公開同步的 HTTP 端點作為觸發程序，以透過 URL 來觸發或呼叫邏輯應用程式。 您也可以使用可呼叫端點的模式，以在邏輯應用程式中巢狀處理工作流程。

若要建立 HTTP 端點，您可以新增這些觸發程序，以讓邏輯應用程式接收傳入要求︰

* [要求](../connectors/connectors-native-reqres.md)

* [API 連線 Webhook](../logic-apps/logic-apps-workflow-actions-triggers.md#apiconnection-trigger)

* [HTTP Webhook](../connectors/connectors-native-webhook.md)

   > [!NOTE]
   > 雖然這些範例使用 [要求] 觸發程序，但是您可以使用任何列出的 HTTP 觸發程序，而且所有的原則都會同樣地套用到其他觸發程序類型。

## <a name="set-up-an-http-endpoint-for-your-logic-app"></a>設定邏輯應用程式的 HTTP 端點

若要建立 HTTP 端點，請新增可接收傳入要求的觸發程序。

1. 登入 [Azure 入口網站](https://portal.azure.com "Azure 入口網站")。 移至邏輯應用程式，並開啟邏輯應用程式設計工具。

2. 新增可讓邏輯應用程式接收傳入要求的觸發程序。 例如，將**要求**觸發程序新增至邏輯應用程式。

3.  在 [要求本文 JSON 結構描述] 下方，您可以選擇針對預期要觸發程序收到的承載 (資料) 輸入 JSON 結構描述。

    設計工具會使用此結構描述來產生權杖，而邏輯應用程式可以使用權杖以透過工作流程從觸發程序中取用、剖析和傳遞資料。 
    深入了解[從 JSON 結構描述產生的權杖](#generated-tokens)。

    在此範例中，輸入設計工具中所顯示的結構描述︰

    ```json
    {
      "type": "object",
      "properties": {
        "address": {
          "type": "string"
        }
      },
      "required": [
        "address"
      ]
    }
    ```

    ![新增要求動作][1]

    > [!TIP]
    > 
    > 您可以從 [jsonschema.net](http://jsonschema.net/) 這類工具產生範例 JSON 承載的結構描述，或在 [要求] 觸發程序中選擇 [請使用範例承載產生結構描述] 來產生範例 JSON 承載的結構描述。 
    > 輸入您的範例承載，然後選擇 [完成]。

    例如，此範例裝載︰

    ```json
    {
       "address": "21 2nd Street, New York, New York"
    }
    ```

    產生此結構描述︰

    ```json
    {
       "type": "object",
       "properties": {
          "address": {
             "type": "string" 
          }
       }
    }
    ```

4.  儲存您的邏輯應用程式。 在 [此 URL 的 HTTP POST] 下方，您現在應該會找到產生的回呼 URL，如下列範例所示：

    ![為端點產生的回呼 URL](./media/logic-apps-http-endpoint/generated-endpoint-url.png)

    此 URL 的查詢參數中包含用於驗證的共用存取簽章 (SAS) 金鑰。 
    您也可以從 Azure 入口網站的邏輯應用程式概觀中取得 HTTP 端點 URL。 在 [觸發程序記錄] 下方，選取觸發程序：

    ![從 Azure 入口網站取得 HTTP 端點 URL][2]

    或者，您可以進行這個呼叫來取得 URL：

    ```
    POST https://management.azure.com/{logic-app-resourceID}/triggers/{myendpointtrigger}/listCallbackURL?api-version=2016-06-01
    ```

## <a name="change-the-http-method-for-your-trigger"></a>變更觸發程序的 HTTP 方法

根據預設，[要求] 觸發程序會預期 HTTP POST 要求，但您可以使用不同的 HTTP 方法。 

> [!NOTE]
> 您只能指定一種方法類型。

1. 在 [要求] 觸發程序上，選擇 [顯示進階選項]。

2. 開啟 [方法] 清單。 在此範例中，選取 **GET**，稍後測試 HTTP 端點的 URL。

    > [!NOTE]
    > 您可以選取任何其他 HTTP 方法，或指定專屬邏輯應用程式的自訂方法。

    ![變更 HTTP 方法](./media/logic-apps-http-endpoint/change-method.png)

## <a name="accept-parameters-through-your-http-endpoint-url"></a>透過 HTTP 端點 URL 接受參數

當您想要 HTTP 端點 URL 接受參數時，請自訂觸發程序的相對路徑。

1. 在 [要求] 觸發程序上，選擇 [顯示進階選項]。 

2. 在 [方法] 下方，指定您想要要求使用的 HTTP 方法。 在此範例中，選取 **GET** 方法 (如果還沒有這麼做的話)，以測試 HTTP 端點的 URL。

      > [!NOTE]
      > 當您指定觸發程序的相對路徑時，也必須明確地指定觸發程序的 HTTP 方法。

3. 在 [相對路徑] 下方，指定您 URL 應該接受之參數的相對路徑，例如，`customers/{customerID}`。

    ![指定參數的 HTTP 方法和相對路徑](./media/logic-apps-http-endpoint/relativeurl.png)

4. 若要使用參數，請將 [回應] 動作新增至邏輯應用程式 (在觸發程序下方，選擇 [新增步驟] > [新增動作] > [回應])。 

5. 在您回應的 [本文] 中，包括觸發程序相對路徑中所指定參數的權杖。

    例如，若要傳回 `Hello {customerID}`，請使用 `Hello {customerID token}` 來更新回應的 [本文]。 
    動態內容清單應該會出現，並顯示 `customerID` 權杖以供您選取。

    ![將參數新增至回應本文](./media/logic-apps-http-endpoint/relativeurlresponse.png)

    您的 [本文] 應該如下列範例所示：

    ![含參數的回應本文](./media/logic-apps-http-endpoint/relative-url-with-parameter.png)

6. 儲存您的邏輯應用程式。 

    HTTP 端點 URL 現在包括相對路徑，例如︰ 

    https&#58;//prod-00.southcentralus.logic.azure.com/workflows/f90cb66c52ea4e9cabe0abf4e197deff/triggers/manual/paths/invoke/customers/{customerID}...

7. 若要測試 HTTP 端點，請複製更新過的 URL，並將其貼入另一個瀏覽器視窗中，但將 `{customerID}` 取代為 `123456`，然後按 Enter 鍵。

    您的瀏覽器應該顯示下列文字︰ 

    `Hello 123456`

<a name="generated-tokens"></a>

### <a name="tokens-generated-from-json-schemas-for-your-logic-app"></a>從 JSON 結構描述產生邏輯應用程式的權杖

在 [要求] 觸發程序中提供 JSON 結構描述時，邏輯應用程式設計工具會產生該結構描述中屬性的權杖。 您接著可以使用這些權杖，透過邏輯應用程式工作流程來傳遞資料。

在此範例中，如果您將 `title` 和 `name` 屬性新增至 JSON 結構描述，則現在可以將其權杖用於稍後的工作流程步驟。 

以下是完整 JSON 結構描述︰

```json
{
   "type": "object",
   "properties": {
      "address": {
         "type": "string"
      },
      "title": {
         "type": "string"
      },
      "name": {
         "type": "string"
      }
   },
   "required": [
      "address",
      "title",
      "name"
   ]
}
```

## <a name="create-nested-workflows-for-logic-apps"></a>建立邏輯應用程式的巢狀工作流程

您可以新增其他可接收要求的邏輯應用程式，以在邏輯應用程式中巢狀處理工作流程。 若要包括這些邏輯應用程式，請將 [Azure Logic Apps - 選擇 Logic Apps 工作流程] 動作新增至觸發程序。 您接著可以從合格的邏輯應用程式中進行選取。

![新增另一個邏輯應用程式](./media/logic-apps-http-endpoint/choose-logic-apps-workflow.png)

## <a name="call-or-trigger-logic-apps-through-http-endpoints"></a>透過 HTTP 端點呼叫或觸發邏輯應用程式

建立 HTTP 端點之後，您可以透過完整 URL 的 `POST` 方法來觸發邏輯應用程式。 邏輯應用程式具有直接存取端點的內建支援。

> [!NOTE] 
> 若要隨時手動執行邏輯應用程式，請在邏輯應用程式設計工具或邏輯應用程式程式碼檢視工具列上選擇 [執行]。

## <a name="reference-content-from-an-incoming-request"></a>參考來自傳入要求的內容

如果內容的類型是 `application/json`，您可以從傳入要求參考屬性。 否則，會將內容視為可傳遞給其他 API 的單一二進位單位。 若要在工作流程內部參考此內容，您必須轉換該內容。 例如，如果您要傳遞 `application/xml` 內容，可以使用 `@xpath()` 進行 XPath 擷取，或使用 `@json()` 來將 XML 轉換成 JSON。 深入了解如何[使用內容類型](../logic-apps/logic-apps-content-type.md)。

若要取得傳入要求的輸出，您可以使用 `@triggerOutputs()` 函式。 輸出可能如下列範例所示︰

```json
{
    "headers": {
        "content-type" : "application/json"
    },
    "body": {
        "myProperty" : "property value"
    }
}
```

若要特別存取 `body` 屬性，您可以使用 `@triggerBody()` 捷徑。 

## <a name="respond-to-requests"></a>回應要求

您可能想要將內容傳回給呼叫端，以回應可啟動邏輯應用程式的特定要求。 若要建構回應的狀態碼、標頭和本文，您可以使用 [回應] 動作。 這個動作可以出現在邏輯應用程式的任何位置，而不只是工作流程的結尾。

> [!NOTE] 
> 如果邏輯應用程式未包括 [回應]，則 HTTP 端點會「立即」回應 [202 已接受] 狀態。 而且，為了讓原始要求取得回應，除非您呼叫工作流程作為巢狀邏輯應用程式，否則回應所需的所有步驟都必須在[要求逾時限制](./logic-apps-limits-and-config.md)內完成。 如果未在此限制內產生任何回應，傳入要求就會逾時，並收到 HTTP 回應 [408 用戶端逾時]。 針對巢狀邏輯應用程式，無論需要多少時間，父邏輯應用程式都會繼續等候回應，直到完成為止。

### <a name="construct-the-response"></a>建構回應

您可以在回應本文中包括數個標頭和任何類型的內容。 在範例回應中，標頭指定回應的內容類型為 `application/json`。 而根據先前針對 [要求] 觸發程序所更新的 JSON 結構描述，本文會包含 `title` 和 `name`。

![HTTP 回應動作][3]

回應具有下列屬性：

| 屬性 | 說明 |
| --- | --- |
| StatusCode |指定用於回應傳入要求的 HTTP 狀態碼。 此代碼可以是任何以 2xx、4xx 或 5xx 開頭的有效狀態碼。 但是，不允許 3xx 狀態碼。 |
| headers |定義要包含於回應中的標頭，且數目不限。 |
| body |指定本文物件可以是字串、JSON 物件，甚至是上一個步驟中所參考的二進位內容。 |

以下是 [回應] 動作之 JSON 結構描述目前的外觀：

``` json
"Response": {
   "inputs": {
      "body": {
         "title": "@{triggerBody()?['title']}",
         "name": "@{triggerBody()?['name']}"
      },
      "headers": {
           "content-type": "application/json"
      },
      "statusCode": 200
   },
   "runAfter": {},
   "type": "Response"
}
```

> [!TIP]
> 若要檢視邏輯應用程式的完整 JSON 定義，請在邏輯應用程式設計工具上選擇 [程式碼檢視]。

## <a name="q--a"></a>問答集

#### <a name="q-what-about-url-security"></a>問︰URL 安全性如何？

答：Azure 會使用共用存取簽章 (SAS)，安全地產生邏輯應用程式回呼 URL。 這個簽章是以查詢參數的形式傳遞，且必須在引發您的邏輯應用程式之前先驗證。 Azure 會使用每個邏輯應用程式、觸發程序名稱以及要執行作業之秘密金鑰的唯一組合來產生簽章。 因此，除非某人具有邏輯應用程式秘密金鑰的存取權，否則他們無法產生有效的簽章。

   > [!IMPORTANT]
   > 對於生產和安全系統，強烈建議直接從瀏覽器呼叫邏輯應用程式，因為：
   > 
   > * URL 中出現共用存取金鑰。
   > * 您由於邏輯應用程式客戶之間共用網域，而無法管理安全內容原則。

#### <a name="q-can-i-configure-http-endpoints-further"></a>問︰我可以進一步設定 HTTP 端點嗎？

答︰可以，HTTP 端點透過 [**API 管理**](../api-management/api-management-key-concepts.md)來支援更進階的設定。 此服務也可讓您透過一致的方式管理所有 API，包括邏輯應用程式、設定自訂網域名稱、使用其他驗證方法等等，例如︰

* [變更要求方法](https://docs.microsoft.com/azure/api-management/api-management-advanced-policies#SetRequestMethod)
* [變更要求的 URL 區段](https://docs.microsoft.com/azure/api-management/api-management-transformation-policies#RewriteURL)
* 在 [Azure 入口網站](https://portal.azure.com/ "Azure 入口網站")中設定 API 管理網域
* 設定檢查基本驗證的原則

#### <a name="q-what-changed-when-the-schema-migrated-from-the-december-1-2014-preview"></a>問︰從 2014 年 12 月 1 日預覽版升級結構描述時的變更內容為何？

答︰以下是這些變更的摘要︰

| 2014 年 12 月 1 日預覽版 | 2016 年 6 月 1 日 |
| --- | --- |
| 按一下 [HTTP 接聽程式] API 應用程式 |按一下 [手動觸發程序] \(不需要 API 應用程式) |
| HTTP 接聽程式設定 [自動傳送回應] |包括 [回應] 動作，或不在工作流程定義中 |
| 設定基本或 OAuth 驗證 |透過 API 管理 |
| 設定 HTTP 方法 |在 [顯示進階選項] 下方，選擇 HTTP 方法。 |
| 設定相對路徑 |在 [顯示進階選項] 下方，新增相對路徑。 |
| 透過 `@triggerOutputs().body.Content` 參考傳入本文 |透過 `@triggerOutputs().body` 參考 |
| **傳送 HTTP 回應** 動作 |按一下 [回應 HTTP 要求] \(不需要 API 應用程式) |

## <a name="get-help"></a>取得說明

若要提出問題、回答問題以及了解其他 Azure Logic Apps 使用者的做法，請造訪 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。

若要改善 Azure Logic Apps 和連接器，請在 [Azure Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)上票選或提交想法。

## <a name="next-steps"></a>後續步驟

* [撰寫邏輯應用程式定義](./logic-apps-author-definitions.md)
* [處理錯誤和例外狀況](./logic-apps-exception-handling.md)

[1]: ./media/logic-apps-http-endpoint/manualtrigger.png
[2]: ./media/logic-apps-http-endpoint/manualtriggerurl.png
[3]: ./media/logic-apps-http-endpoint/response.png