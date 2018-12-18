---
title: 使用 Azure 服務匯流排傳送及接收訊息 - Azure Logic Apps | Microsoft Docs
description: 在 Azure Logic Apps 中使用 Azure 服務匯流排來設定企業雲端傳訊
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.assetid: d6d14f5f-2126-4e33-808e-41de08e6721f
ms.topic: article
tags: connectors
ms.date: 08/25/2018
ms.openlocfilehash: 813df5b4ef37ad1264df48863aa8f0ed5a4d4789
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048769"
---
# <a name="exchange-messages-in-the-cloud-with-azure-service-bus-and-azure-logic-apps"></a>使用 Azure 服務匯流排與 Azure Logic Apps 在雲端中交換訊息

您可以使用 Azure Logic Apps 和 Azure 服務匯流排連接器，建立自動化的工作和工作流程，以便傳輸資料，例如銷售訂單和採購單、日誌及貴組織應用程式之間的存貨移動。 連接器不只會監視、傳送及管理訊息，也會執行佇列、工作階段、主題、訂用帳戶等等的動作，例如：

* 監視訊息何時在佇列、主題及主題訂用帳戶中送達 (自動完成) 或接收 (查看鎖定)。 
* 傳送訊息。
* 建立和刪除主題訂用帳戶。
* 管理佇列和主題訂用帳戶中的訊息，例如，取得、延後、完成、延遲、放棄和無效信件。
* 更新佇列和主題訂用帳戶中訊息和工作階段的鎖定。
* 關閉佇列和主題中的工作階段。

您可以使用觸發程序，從服務匯流排收到回應，並且讓輸出可供邏輯應用程式中的其他動作使用。 您也可以讓其他動作使用服務匯流排動作的輸出。 如果您是服務匯流排和 Logic Apps 的新手，請檢閱[什麼是 Azure 服務匯流排？](../service-bus-messaging/service-bus-messaging-overview.md) 以及[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* 服務匯流排命名空間和傳訊實體，例如佇列。 如果您沒有這些項目，請了解如何[建立您的服務匯流排命名空間和佇列](../service-bus-messaging/service-bus-create-namespace-portal.md)。 

  與使用這些項目的邏輯應用程式所用的同一個 Azure 訂用帳戶中，必須要有這些項目存在。

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中使用服務匯流排的邏輯應用程式。 邏輯應用程式必須存在於與您的服務匯流排相同的 Azure 訂用帳戶。 若要開始使用服務匯流排觸發程序，請[建立空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 若要使用服務匯流排動作，請使用其他觸發程序來啟動邏輯應用程式，例如「週期」觸發程序。

<a name="permissions-connection-string"></a>

## <a name="check-permissions"></a>檢查權限

確認您的邏輯應用程式有權可以存取您的服務匯流排命名空間。 

1. 登入 [Azure 入口網站](https://portal.azure.com)。 

2. 移至您的服務匯流排「命名空間」。 在命名空間頁面的 [設定] 之下，選取 [共用存取原則]。 在 [宣告] 之下，確認您有該命名空間的 [管理] 權限

   ![管理服務匯流排命名空間的權限](./media/connectors-create-api-azure-service-bus/azure-service-bus-namespace.png)

3. 取得服務匯流排命名空間的連接字串。 當您在邏輯應用程式中輸入連線資訊時，需要這個字串。

   1. 選取 [RootManageSharedAccessKey]。 
   
   1. 選擇您主要連接字串旁邊的 [複製] 按鈕。 儲存連接字串以供稍後使用。

      ![複製服務匯流排命名空間連接字串](./media/connectors-create-api-azure-service-bus/find-service-bus-connection-string.png)

   > [!TIP]
   > 若要確認您的連接字串，是否與您的服務匯流排命名空間或傳訊實體 (例如佇列) 相關聯，請搜尋連接字串中是否有 `EntityPath` 參數。 如果您發現這個參數，表示此連接字串適用於特定實體，但不是可用於您邏輯應用程式的正確字串。

## <a name="add-trigger-or-action"></a>新增觸發程序或動作

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. 登入 [Azure 入口網站](https://portal.azure.com)，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 若要將「觸發程序」新增至空白邏輯應用程式，請在搜尋方塊中，輸入 "Azure Service Bus" 作為篩選條件。 請在觸發程序清單底下，選取您想要的觸發程序。 

   例如，若要在新項目傳送至「服務匯流排」佇列時，觸發邏輯應用程式，則選取此觸發程序：**在佇列中收到訊息時 (自動完成)**

   ![選取服務匯流排觸發程序](./media/connectors-create-api-azure-service-bus/select-service-bus-trigger.png)

   > [!NOTE]
   > 部分觸發程序會傳回一或多個訊息，像是觸發程序：**一或多個訊息到達佇列時 (自動完成)**。 引發這些觸發程序時，它們傳回的訊息數會介於 1 和此觸發程序**訊息計數上限**屬性指定的訊息數目之間。

   所有服務匯流排觸發程序都是長時間輪詢觸發程序，這表示當引發觸發程序時，觸發程序會處理所有訊息，然後等候 30 秒讓佇列或主題訂閱中出現更多訊息。 
   如果在 30 秒內未出現任何訊息，就會略過觸發程序執行。 
   如果有，觸發程序會繼續讀取訊息，直到佇列或主題訂閱是空的。 下一次的觸發程序輪詢，會根據觸發程序的屬性指定的循環間隔。

1. 若要將「動作」新增至現有的邏輯應用程式，請遵循下列步驟： 

   1. 請在想要新增動作的最後一個步驟底下，選擇 [新增步驟]。 

      若要在步驟之間新增動作，將指標移至步驟之間的箭號。 
      選擇顯示的加號 (**+**)，然後選取 [新增動作]。

   1. 在搜尋方塊中，輸入 "Azure Service Bus" 作為篩選條件。 
   請在動作清單底下，選取您想要的動作。 
 
      例如，選取此動作：**傳送訊息**

      ![選取服務匯流排動作](./media/connectors-create-api-azure-service-bus/select-service-bus-send-message-action.png) 

1. 如果您是第一次將邏輯應用程式連線至服務匯流排命名空間，邏輯應用程式設計工具現在會提示您輸入連線資訊。 

   1. 請為您的連線提供名稱，然後選取您的服務匯流排命名空間。

      ![建立服務匯流排連線 (第一部分)](./media/connectors-create-api-azure-service-bus/create-service-bus-connection-1.png)

      若要改為手動輸入連接字串，請選擇 [手動輸入連線資訊]。 
      如果您還沒有連接字串，請了解[如何尋找您的連接字串](#permissions-connection-string)。

   1. 現在選取您的服務匯流排原則，然後選擇 [建立]。

      ![建立服務匯流排連線 (第二部分)](./media/connectors-create-api-azure-service-bus/create-service-bus-connection-2.png)

1. 針對這個範例，請選取您想要的傳訊實體，例如佇列或主題。 在此範例中，選取您的服務匯流排佇列。 
   
   ![選取服務匯流排佇列](./media/connectors-create-api-azure-service-bus/service-bus-select-queue.png)

1. 為您的觸發程序或動作提供必要的詳細資料。 針對這個範例，請遵循觸發程序或動作的相關步驟： 

   * **針對範例觸發程序**：設定輪詢間隔和檢查佇列的頻率。

     ![設定輪詢間隔](./media/connectors-create-api-azure-service-bus/service-bus-trigger-details.png)

     當您完成時，藉由新增您想要的動作，可繼續建置邏輯應用程式的工作流程。 例如，您可以新增在新訊息送達時，會傳送電子郵件的動作。
     當觸發程序檢查您的佇列並且發現新訊息時，您的邏輯應用程式會針對找到的訊息，執行已選取的動作。

   * **針對範例動作**：輸入訊息內容和任何其他詳細資料。 

     ![提供訊息內容和詳細資料](./media/connectors-create-api-azure-service-bus/service-bus-send-message-details.png)

     當您完成時，藉由新增您想要的任何其他動作，可繼續建置邏輯應用程式的工作流程。 例如，您可以新增動作，該動作會傳送電子郵件以確認訊息已傳送。

1. 儲存您的邏輯應用程式。 在設計工具的工具列上，選擇 [儲存]。

## <a name="connector-reference"></a>連接器參考

如需觸發程序、動作和限制的技術詳細資訊，它們是由連接器的 OpenAPI (以前稱為 Swagger) 來描述，請檢閱連接器的[參考頁面](/connectors/servicebus/)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)