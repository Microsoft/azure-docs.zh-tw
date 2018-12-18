---
title: 案例 - 使用 Azure Functions 與 Azure 服務匯流排觸發邏輯應用程式 | Microsoft Docs
description: 使用 Azure Functions 和 Azure 服務匯流排建立可觸發邏輯應用程式的函式
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.reviewer: jehollan, klam, LADocs
ms.topic: article
ms.assetid: 19cbd921-7071-4221-ab86-b44d0fc0ecef
ms.date: 08/25/2018
ms.openlocfilehash: 8d6f2ecaec7bf7ae7e4415ccd60fc6ec3ff487f3
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43126167"
---
# <a name="scenario-trigger-logic-apps-with-azure-functions-and-azure-service-bus"></a>案例：使用 Azure Functions 和 Azure 服務匯流排觸發邏輯應用程式

當您需要部署長時間執行的接聽程式或工作時，可以使用 Azure Functions 來建立邏輯應用程式的觸發程序。 例如，您可以建立一個將會在佇列上接聽的函數，接著立即引發邏輯應用程式成為推送觸發程序。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識 

* 先[建立函式應用程式](../azure-functions/functions-create-function-app-portal.md)，才能建立 Azure 函式。

## <a name="create-logic-app"></a>建立邏輯應用程式

在此範例中，您會有針對需要觸發的每個邏輯應用程式執行的函數。 首先，建立具有 HTTP 要求觸發程序的邏輯應用程式。 每當收到佇列訊息時，函數即會呼叫該端點。  

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後建立空白的邏輯應用程式。 

   如果您還不熟悉邏輯應用程式，請檢閱[快速入門：如何建立第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

1. 在搜尋方塊中，輸入 "http request"。 在觸發程序清單下選取此觸發程序：**收到 HTTP 要求時**

   ![選取觸發程序](./media/logic-apps-scenario-function-sb-trigger/when-http-request-received-trigger.png)

1. 針對 [要求] 觸發程序，您可以選擇性地輸入 JSON 結構描述，以便搭配佇列訊息使用。 JSON 結構描述可協助 Logic App Designer 了解輸入資料的結構，並且讓您在整個工作流程中更輕鬆地選取輸出。 

   若要指定結構描述，請在 [要求本文 JSON 結構描述] 方塊中輸入結構描述，例如： 

   ![指定 JSON 結構描述](./media/logic-apps-scenario-function-sb-trigger/when-http-request-received-trigger-schema.png)

   如果您沒有結構描述，但是有 JSON 格式的範例承載，即可從該承載產生 JSON 結構描述。

   1. 在要求觸發程序中，選擇 [使用範例承載來產生結構描述]。

   1. 在 [輸入或貼上範例 JSON 承載] 之下，輸入您的範例承載，然後選擇 [完成]。
      
      ![輸入範例承載](./media/logic-apps-scenario-function-sb-trigger/enter-sample-payload.png)

   此範例承載會產生出現在觸發程序中的這個結構描述：

   ```json
   {
      "type": "object",
      "properties": {
         "address": {
            "type": "object",
            "properties": {
               "number": {
                  "type": "integer"
               },
               "street": {
                  "type": "string"
               },
               "city": {
                  "type": "string"
               },
               "postalCode": {
                  "type": "integer"
               },
               "country": {
                  "type": "string"
               }
            }
         }
      }
   }
   ```

1. 新增您希望在收到佇列訊息之後發生的任何其他動作。 

   例如，您可以使用 Office 365 Outlook 連接器傳送電子郵件。

1. 儲存邏輯應用程式，進而為此邏輯應用程式中的觸發程序產生回呼 URL。 此 URL 會出現在 **HTTP POST URL** 屬性中。

   ![為觸發程序產生的回呼 URL](./media/logic-apps-scenario-function-sb-trigger/callback-URL-for-trigger.png)

## <a name="create-azure-function"></a>建立 Azure 函式

接著，建立一個函式，作為觸發程序並接聽佇列。 

1. 在 Azure 入口網站中，開啟並展開函式應用程式 (如果尚未開啟)。 

1. 在函式應用程式名稱之下，展開 [Functions]。 在 [函式] 窗格上，選擇 [新增函式]。 選取此範本：**服務匯流排佇列觸發程序 - C#**
   
   ![選取 Azure Functions 入口網站](./media/logic-apps-scenario-function-sb-trigger/newqueuetriggerfunction.png)

1. 為您的觸發程序提供名稱，然後設定服務匯流排佇列的連接，其可使用服務匯流排 SDK `OnMessageReceive()` 接聽程式。

1. 撰寫基本函式，以使用佇列訊息作為觸發程序來呼叫先前建立的邏輯應用程式端點，例如： 
   
   ```CSharp
   using System;
   using System.Threading.Tasks;
   using System.Net.Http;
   using System.Text;
   
   private static string logicAppUri = @"https://prod-05.westus.logic.azure.com:443/.........";
   
   public static void Run(string myQueueItem, TraceWriter log)
   {
       log.Info($"C# ServiceBus queue trigger function processed message: {myQueueItem}");

       using (var client = new HttpClient())
       {
           var response = client.PostAsync(logicAppUri, new StringContent(myQueueItem, Encoding.UTF8, "application/json")).Result;
       }
   }
   ```

   此範例會使用 `application/json` 訊息內容類型，但是您可以視需要變更此類型。

1. 若要測試此函式，請使用 [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer) 等工具來新增佇列訊息。 

   在函式收到訊息之後，邏輯應用程式會立即觸發。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

