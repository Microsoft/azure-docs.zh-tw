---
title: 建立以事件為基礎的工作流程或動作 - Azure Logic Apps | Microsoft Docs
description: 使用 Webhook 和 Azure Logic Apps 自動執行以事件為基礎的工作流程或動作
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, jehollan, LADocs
ms.assetid: 71775384-6c3a-482c-a484-6624cbe4fcc7
ms.topic: article
tags: connectors
ms.date: 07/21/2016
ms.openlocfilehash: 7b1886321ca4afd4b4710bd9fddf16d2d5eb224b
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43126582"
---
# <a name="create-event-based-workflows-or-actions-by-using-webhooks-and-azure-logic-apps"></a>使用 Webhook 和 Azure Logic Apps 建立以事件為基礎的工作流程或動作

透過使用 webhook 動作和觸發程序，您可以啟動、暫停和繼續流程來執行這些工作︰

* 收到項目時，即從 [Azure 事件中樞](https://github.com/logicappsio/EventHubAPI) 觸發
* 等待核准再繼續工作流程

深入了解關於[如何建立支援 webhook 的自訂 API](../logic-apps/logic-apps-create-api-app.md)。

## <a name="use-the-webhook-trigger"></a>使用 webhook 觸發程序

[觸發程序](connectors-overview.md)是啟動邏輯應用程式工作流程的事件。 webhook 觸發程序是以事件為基礎，而不是依賴對新項目進行輪詢。 如同[要求觸發程序](connectors-native-reqres.md)一般，邏輯應用程式會觸發事件發生的瞬間。 webhook 觸發程序會註冊服務的*回呼 URL*，並依需求使用該 URL 來觸發邏輯應用程式。

以下是如何在邏輯應用程式設計工具中設定 HTTP 觸發程序的範例。 這個步驟假設您已部署或正在存取的 API 會遵循[邏輯應用程式中的 webhook 訂閱和取消訂閱模式](../logic-apps/logic-apps-create-api-app.md#webhook-triggers)。 每當邏輯應用程式隨新的 webhook 儲存或由停用切換為啟用時，就會進行訂閱呼叫。 每當系統移除並儲存邏輯應用程式 webhook 觸發程序時，就會進行取消訂閱呼叫。

**新增 webhook 觸發程序**

1. 將 **HTTP Webhook** 觸發程序新增為邏輯應用程式中的第一個步驟。
2. 填入 webhook 訂閱和取消訂閱呼叫的參數。

   此步驟所遵循的模式與 [HTTP 動作](connectors-native-http.md)格式相同。

     ![HTTP 觸發程序](./media/connectors-native-webhook/using-trigger.png)

3. 加入至少一個動作。
4. 按一下 [儲存] 來發佈邏輯應用程式。 此步驟會呼叫含有觸發此邏輯應用程式所需的回呼 URL 訂閱端點。
5. 每當服務對回呼 URL 發出 `HTTP POST` 時，就會觸發邏輯應用程式，並包含要求中所傳遞的任何資料。

## <a name="use-the-webhook-action"></a>使用 webhook 動作

[「動作」](connectors-overview.md)是由邏輯應用程式中定義的工作流程所執行的作業。 Webhook 動作會註冊服務的*回呼 URL*，並等到呼叫該 URL 後再繼續。 [傳送核准電子郵件](connectors-create-api-office365-outlook.md)是遵循這個模式的連接器範例。 您可以透過 webhook 動作將此模式擴充到任何服務。 

以下是如何在邏輯應用程式設計工具中設定 webhook 動作的範例。 這些步驟假設您已部署或正在存取的 API 會遵循[邏輯應用程式中的 webhook 訂閱和取消訂閱模式](../logic-apps/logic-apps-create-api-app.md#webhook-actions)。 每當邏輯應用程式執行 webhook 動作時，就會進行訂閱呼叫。 每當執行在等待回應時取消，或在邏輯應用程式逾時前取消，就會進行取消訂閱呼叫。

**若要新增 webhook 動作**

1. 選擇 [新增步驟] > [新增動作]。

2. 在搜尋方塊中，輸入 "webhook" 以找出 **HTTP Webhook** 動作。

    ![選取查詢動作](./media/connectors-native-webhook/using-action-1.png)

3. 填入 webhook 訂閱和取消訂閱呼叫的參數

   此步驟所遵循的模式與 [HTTP 動作](connectors-native-http.md)格式相同。

     ![完整查詢動作](./media/connectors-native-webhook/using-action-2.png)
   
   在執行階段中，邏輯應用程式會進行該步驟之後呼叫訂閱端點。

4. 按一下 [儲存] 來發佈邏輯應用程式。

## <a name="technical-details"></a>技術詳細資訊

以下是更多關於 webhook 支援的觸發程序和動作詳細資料。

## <a name="webhook-triggers"></a>WebHook 觸發程序

| 動作 | 說明 |
| --- | --- |
| HTTP Webhook |訂閱服務的回呼 URL，就能視需要呼叫該 URL 以觸發邏輯應用程式。 |

### <a name="trigger-details"></a>觸發程序詳細資料

#### <a name="http-webhook"></a>HTTP Webhook

訂閱服務的回呼 URL，就能視需要呼叫該 URL 以觸發邏輯應用程式。
代表必要欄位 * 。

| 顯示名稱 | 屬性名稱 | 說明 |
| --- | --- | --- |
| 訂閱方法* |method |用於訂閱要求的 HTTP 方法 |
| 訂閱 URI* |uri |用於訂閱要求的 HTTP URI |
| 取消訂閱方法* |method |用於取消訂閱要求的 HTTP 方法 |
| 取消訂閱 URI* |uri |用於取消訂閱要求的 HTTP URI |
| 訂閱本文 |body |訂閱的 HTTP 要求本文 |
| 訂閱標頭 |headers |訂閱的 HTTP 要求標頭 |
| 訂閱驗證 |驗證 |訂閱要使用的 HTTP 驗證。 如需詳細資訊，[請參閱 HTTP 連接器](connectors-native-http.md#authentication) |
| 取消訂閱頁面 |body |取消訂閱的 HTTP 要求本文 |
| 取消訂閱標頭 |headers |取消訂閱的 HTTP 要求標頭 |
| 取消訂閱驗證 |驗證 |取消訂閱要使用的 HTTP 驗證。 如需詳細資訊，[請參閱 HTTP 連接器](connectors-native-http.md#authentication) |

**輸出詳細資料**

Webhook 要求

| 屬性名稱 | 資料類型 | 說明 |
| --- | --- | --- |
| headers |物件 |Webhook 要求標頭 |
| body |物件 |Webhook 要求物件 |
| 狀態碼 |int |Webhook 要求狀態碼 |

## <a name="webhook-actions"></a>Webhook 動作

| 動作 | 說明 |
| --- | --- |
| HTTP Webhook |訂閱服務的回呼 URL，就能視需要呼叫繼續工作流程步驟的 URL。 |

### <a name="action-details"></a>動作詳細資料

#### <a name="http-webhook"></a>HTTP Webhook

訂閱服務的回呼 URL，就能視需要呼叫繼續工作流程步驟的 URL。
代表必要欄位 * 。

| 顯示名稱 | 屬性名稱 | 說明 |
| --- | --- | --- |
| 訂閱方法* |method |用於訂閱要求的 HTTP 方法 |
| 訂閱 URI* |uri |用於訂閱要求的 HTTP URI |
| 取消訂閱方法* |method |用於取消訂閱要求的 HTTP 方法 |
| 取消訂閱 URI* |uri |用於取消訂閱要求的 HTTP URI |
| 訂閱本文 |body |訂閱的 HTTP 要求本文 |
| 訂閱標頭 |headers |訂閱的 HTTP 要求標頭 |
| 訂閱驗證 |驗證 |訂閱要使用的 HTTP 驗證。 如需詳細資訊，[請參閱 HTTP 連接器](connectors-native-http.md#authentication) |
| 取消訂閱頁面 |body |取消訂閱的 HTTP 要求本文 |
| 取消訂閱標頭 |headers |取消訂閱的 HTTP 要求標頭 |
| 取消訂閱驗證 |驗證 |取消訂閱要使用的 HTTP 驗證。 如需詳細資訊，[請參閱 HTTP 連接器](connectors-native-http.md#authentication) |

**輸出詳細資料**

Webhook 要求

| 屬性名稱 | 資料類型 | 說明 |
| --- | --- | --- |
| headers |物件 |Webhook 要求標頭 |
| body |物件 |Webhook 要求物件 |
| 狀態碼 |int |Webhook 要求狀態碼 |

## <a name="next-steps"></a>後續步驟

* [建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* [尋找其他連接器](apis-list.md)