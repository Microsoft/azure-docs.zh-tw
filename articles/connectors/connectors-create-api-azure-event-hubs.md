---
title: 連線至 Azure 事件中樞 - Azure Logic Apps | Microsoft Docs
description: 使用 Azure 事件中樞與 Azure Logic Apps 管理和監視事件
author: ecfan
manager: jeconnoc
ms.author: estfan
ms.date: 05/21/2018
ms.topic: article
ms.service: logic-apps
services: logic-apps
ms.reviewer: klam, LADocs
ms.suite: integration
tags: connectors
ms.openlocfilehash: 86fc1c3542bea1be840041bb73df15631c066c7e
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35294967"
---
# <a name="monitor-receive-and-send-events-with-azure-event-hubs-and-azure-logic-apps"></a>使用 Azure 事件中樞與 Azure Logic Apps 監視、接收和傳送事件 

本文說明如何使用 Azure 事件中樞連接器，從邏輯應用程式內部監視和管理傳送至 [Azure 事件中樞](../event-hubs/event-hubs-what-is-event-hubs.md)的事件。 這樣一來，您就可以建立邏輯應用程式，來自動執行從事件中樞檢查、傳送和接收事件的工作和工作流程。 

如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 如果您還不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)和[快速入門：建立第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。
如需連接器特定的技術資訊，請參閱 <a href="https://docs.microsoft.com/connectors/eventhubs/" target="blank">Azure 事件中樞連接器參考</a>。

## <a name="prerequisites"></a>先決條件

* [Azure 事件中樞命名空間和事件中樞](../event-hubs/event-hubs-create.md)

* 您要存取事件中樞的邏輯應用程式。 若要使用 Azure 事件中樞觸發程序來啟動邏輯應用程式，您需要[空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 

<a name="permissions-connection-string"></a>

## <a name="check-permissions-and-get-connection-string"></a>檢查使用權限並取得連接字串

若要讓邏輯應用程式存取事件中樞，請確認您有權取得事件中樞命名空間的連接字串。

1. 登入 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>。 

2. 移至您的事件中樞「命名空間」，而不是特定事件中樞。 在命名空間頁面的 [設定] 之下，選擇 [共用存取原則]。 在 [宣告] 之下，確認您有該命名空間的 [管理] 權限。

   ![管理事件中樞命名空間的權限](./media/connectors-create-api-azure-event-hubs/event-hubs-namespace.png)

3. 如果您想要在稍後手動輸入您的連線資訊，請取得事件中樞命名空間的連接字串。 

   1. 在 [原則] 下，選擇 [RootManageSharedAccessKey]。 

   2. 尋找主索引鍵的連接字串。 選擇 [複製] 按鈕，然後儲存連接字串以供稍後使用。

      ![複製事件中樞命名空間連接字串](media/connectors-create-api-azure-event-hubs/find-event-hub-namespace-connection-string.png)

      > [!TIP]
      > 若要確認您的連接字串是否與事件中樞命名空間或特定事件中樞相關聯，請確認連接字串沒有 `EntityPath` 參數。 如果您發現這個參數，此連接字串適用於特定事件中樞「實體」，而不是使用您的邏輯應用程式的正確字串。

4. 現在繼續[新增事件中樞觸發程序](#add-trigger)或[新增事件中樞動作](#add-action)。

<a name="add-trigger"></a>

## <a name="add-an-event-hubs-trigger"></a>新增事件中樞觸發程序

在 Azure Logic Apps 中，每個邏輯應用程式都必須使用[觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)啟動，而該觸發程序會在特定事件發生或符合特定條件時引發。 每次引發觸發程序時，Logic Apps 引擎都會建立邏輯應用程式執行個體，並開始執行應用程式的工作流程。

此範例會示範當新的事件傳送到事件中樞時，如何啟動邏輯應用程式工作流程。 

1. 在 Azure 入口網站或 Visual Studio 中，建立空白的邏輯應用程式，以開啟 Logic Apps 設計工具。 這個範例會使用 Azure 入口網站。

2. 在搜尋方塊中，輸入「事件中樞」作為篩選條件。 從觸發程序清單中，選取您想要的觸發程序。 

   此範例使用以下觸發程序：**事件中樞 - 當事件可用於事件中樞時**

   ![選取觸發程序](./media/connectors-create-api-azure-event-hubs/find-event-hubs-trigger.png)

3. 如果系統提示您輸入連線詳細資料，請[立即建立事件中樞連線](#create-connection)。 或者，如果連線已存在，請提供觸發程序的必要資訊。

   1. 從**事件中樞名稱**清單中，選取您要監視的事件中樞。

      ![指定事件中樞或取用者群組](./media/connectors-create-api-azure-event-hubs/select-event-hub.png)

   2. 選取要讓觸發程序檢查事件中樞的間隔和頻率。
 
   3. 若要選擇性地選取部分進階觸發程序選項，請選擇 [顯示進階選項]。
   
      ![觸發程序進階選項](./media/connectors-create-api-azure-event-hubs/event-hubs-trigger-advanced.png)

      | 屬性 | 詳細資料 | 
      |----------|---------| 
      | 內容類型  | 選取事件的內容類型。 預設為 "application/octet-stream"。 |
      | 內容結構描述 | 輸入從事件中樞所讀取事件的內容結構描述 (JSON 格式)。 |
      | 取用者群組名稱 | 輸入要讀取事件的事件中樞[取用者群組名稱](../event-hubs/event-hubs-features.md#consumer-groups)。 若未指定，就會使用預設取用者群組。 |
      | 最小分割區索引鍵 | 輸入要讀取的最小[分割區](../event-hubs/event-hubs-features.md#partitions)識別碼。 預設會讀取所有分割區。 |
      | 最大分割區索引鍵 | 輸入要讀取的最大[分割區](../event-hubs/event-hubs-features.md#partitions)識別碼。 預設會讀取所有分割區。 |
      | 最大事件計數 | 輸入事件數目的最大值。 觸發程序傳回的事件數目會介於 1 到這個屬性指定的數目之間。 |
      |||

4. 當您完成時，請在設計工具的工具列上，選擇 [儲存]。

5. 現在，繼續針對您想要使用觸發程序結果來執行的工作，於邏輯應用程式中新增一或多個動作。

> [!NOTE]
> 所有事件中樞觸發程序都是*長時間輪詢*觸發程序；這表示當引發觸發程序時，觸發程序會處理所有事件，然後等候 30 秒等事件中樞中出現更多事件。
> 如果在 30 秒內沒收到任何事件，就會略過觸發程序執行。 否則，觸發程序會繼續讀取事件，直到事件中樞沒有任何事件為止。
> 下一次發生觸發程序輪詢的時間，會根據您在觸發程序屬性中指定的循環間隔而定。

<a name="add-action"></a>

## <a name="add-an-event-hubs-action"></a>新增事件中樞動作

在 Azure Logic Apps 中，[動作](../logic-apps/logic-apps-overview.md#logic-app-concepts)是工作流程中跟隨在觸發程序或另一個動作之後的步驟。 在此範例中，邏輯應用程式會從事件中樞觸發程序開始，其會檢查事件中樞內的新事件。 

1. 在 Azure 入口網站或 Visual Studio 的邏輯應用程式設計工具中，開啟邏輯應用程式。 這個範例會使用 Azure 入口網站。

2. 在觸發程序或動作底下，選擇 [新增步驟] > [新增動作]。

   若要在現有步驟之間新增動作，請將滑鼠放在連接箭頭上。 
   選擇顯示的加號 (**+**)，然後選擇 [新增動作]。

3. 在搜尋方塊中，輸入「事件中樞」作為篩選條件。
從 [動作] 清單中，選取您想要的動作。 

   在此範例中，選取此動作：**事件中樞 - 傳送事件**

   ![選取「事件中樞 - 傳送事件」](./media/connectors-create-api-azure-event-hubs/find-event-hubs-action.png)

4. 如果系統提示您輸入連線詳細資料，請[立即建立事件中樞連線](#create-connection)。 或者，如果連線已存在，請提供動作的必要資訊。

   | 屬性 | 必要 | 說明 | 
   |----------|----------|-------------|
   | 事件中樞名稱 | yes | 選取要傳送事件的事件中樞 | 
   | 事件內容 | 否 | 您要傳送事件的內容 | 
   | properties | 否 | 要傳送的應用程式屬性與值 | 
   |||| 

   例如︰ 

   ![選取事件中樞名稱，並提供事件內容](./media/connectors-create-api-azure-event-hubs/event-hubs-send-event-action.png)

5. 當您完成時，請在設計工具的工具列上，選擇 [儲存]。

<a name="create-connection"></a>

## <a name="connect-to-your-event-hub"></a>連線至事件中樞

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)] 

1. 當系統提示您需要連線資訊時，請提供以下詳細資料：

   | 屬性 | 必要 | 值 | 說明 | 
   |----------|----------|-------|-------------|
   | 連線名稱 | yes | <*connection-name*> | 要為連線建立的名稱 |
   | 事件中樞命名空間 | yes | <*event-hubs-namespace*> | 選取您想要使用的事件中樞命名空間。 | 
   |||||  

   例如︰

   ![建立事件中樞連線](./media/connectors-create-api-azure-event-hubs/create-event-hubs-connection-1.png)

   若要手動輸入連接字串，請選擇 [手動輸入連線資訊]。 
   了解[如何尋找您的連接字串](#permissions-connection-string)。

2. 選取要使用的事件中樞原則 (若尚未選取)。 選擇 [建立] 。

   ![建立事件中樞連線 (第二部分)](./media/connectors-create-api-azure-event-hubs/create-event-hubs-connection-2.png)

3. 您在建立連線之後，請繼續[新增事件中樞觸發程序](#add-trigger)或[新增事件中樞動作](#add-action)。

## <a name="connector-reference"></a>連接器參考

如需連接器的 Swagger 檔案所敘述的技術詳細資料 (例如，觸發程序、動作和限制)，請參閱[連接器的參考頁面](/connectors/eventhubs/)。 

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)