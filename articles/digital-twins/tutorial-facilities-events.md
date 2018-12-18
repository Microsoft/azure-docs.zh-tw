---
title: 擷取來自 Azure Digital Twins 空間的事件 | Microsoft Docs
description: 了解如何使用本教學課程中的步驟，藉由整合 Azure Digital Twins 與 Logic Apps，接收來自空間的通知。
services: digital-twins
author: dsk-2015
ms.service: digital-twins
ms.topic: tutorial
ms.date: 10/15/2018
ms.author: dkshir
ms.openlocfilehash: 91dd16938efbd1e24419352f66e3238646a77e8a
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49323205"
---
# <a name="tutorial-receive-notifications-from-your-azure-digital-twins-spaces-using-logic-apps"></a>教學課程：使用 Logic Apps 接收來自 Azure Digital Twins 空間的通知

在部署了 Azure Digital Twins 執行個體、佈建空間並實作自訂函式來監視特定情況後，您便可以在所監視的情況發生時，透過電子郵件通知辦公室的管理員。 

在[第一個教學課程](tutorial-facilities-setup.md)中，您已設定虛構建築物的空間圖形，該建築物中有一個空間含有可監控運動、二氧化碳和溫度的感應器。 在[第二個教學課程](tutorial-facilities-udf.md)中，您已佈建圖形和稱為使用者定義函式的自訂函式，以便監視這些感應器數值，並在房間沒人、且溫度和二氧化碳濃度在舒適範圍內時觸發通知。 本教學課程說明如何整合這些通知與 Azure Logic Apps，以在空出這樣的房間時傳送電子郵件。 辦公室管理員可以使用這項資訊，來協助員工預約最具生產力的會議室。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 整合事件與事件方格
> * 使用 Logic Apps 發出事件通知
    
## <a name="prerequisites"></a>必要條件

本教學課程假設您已經[設定](tutorial-facilities-setup.md)並[佈建](tutorial-facilities-udf.md) Azure Digital Twins 設定。 請確定您符合下列條件後，再繼續本課程：
- [Azure 帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
- 執行中的 Digital Twins 執行個體。
- 在工作電腦上下載並解壓縮 [Digital Twins C# 範例](https://github.com/Azure-Samples/digital-twins-samples-csharp)。
- 在開發電腦上安裝 [.NET Core SDK 2.1.403 版或更新版本](https://www.microsoft.com/net/download)，以執行範例。 執行 `dotnet --version` 來確認所安裝的版本是否正確。 
- 用來傳送通知電子郵件的 Office 365 帳戶。

## <a name="integrate-events-with-event-grid"></a>整合事件與事件方格 
在本節中，您會設定[事件方格](../event-grid/overview.md)以收集來自 Digital Twins 執行個體的事件，並將這些事件重新導向至[事件處理常式](../event-grid/event-handlers.md)，例如 Logic Apps。

### <a name="create-event-grid-topic"></a>建立事件方格主題
[事件方格主題](../event-grid/concepts.md#topics)會提供介面供您路由傳送使用者定義函式所產生的事件。 

1. 登入 [Azure 入口網站](https://portal.azure.com)。

1. 在左側導覽面板中，按一下 [建立資源]。 

1. 搜尋並選取 [事件方格主題]。 按一下頁面底部的 [新增] 。

1. 輸入事件方格主題的 [名稱]，然後選擇 [訂用帳戶]。 選取針對 Digital Twins 執行個體所使用或建立的 [資源群組]，以及 [位置]。 按一下頁面底部的 [新增] 。 

    ![建立事件方格主題](./media/tutorial-facilities-events/create-event-grid-topic.png)

1. 從資源群組瀏覽至事件方格主題，按一下 [概觀]，然後將 [主題端點] 的值複製到暫存檔。 下一節會用到此 URL。 

1. 按一下 [存取金鑰]，然後將 [金鑰 1] 和 [金鑰 2] 複製到暫存檔。 下一節在建立端點時需要用到這些值。

    ![事件方格金鑰](./media/tutorial-facilities-events/event-grid-keys.png)

### <a name="create-an-endpoint-for-the-event-grid-topic"></a>建立事件方格主題的端點

1. 在 [命令] 視窗中，確定您位於 Digital Twins 範例的 **occupancy-quickstart\src** 資料夾中。

1. 在 Visual Studio Code 編輯器中開啟 **actions\createEndpoints.yaml** 檔案。 請確定此檔案具有下列內容︰

    ```yaml
    - type: EventGrid
      eventTypes:
      - SensorChange
      - SpaceChange
      - TopologyOperation
      - UdfCustom
      connectionString: Primary_connection_string_for_your_Event_Grid
      secondaryConnectionString: Secondary_connection_string_for_your_Event_Grid
      path: Event_Grid_Topic_Path
    ```

1. 將預留位置 `Primary_connection_string_for_your_Event_Grid` 替換為 [金鑰 1] 的值。 

1. 將預留位置 `Secondary_connection_string_for_your_Event_Grid` 替換為 [金鑰 2] 的值。

1. 將預留位置 `Event_Grid_Topic_Path` 替換為事件方格主題的路徑。 若要取得此路徑，請從**主題端點** URL 中移除 https:// 和尾端的資源路徑。 此路徑的格式應該會類似：**yourEventGridName.yourLocation.eventgrid.azure.net**。 

    > [!IMPORTANT]
    > 所有值在輸入時不得有任何引號。 請確定 YAML 檔案中的所有冒號後面都至少有一個空格字元。 您也可以使用任何線上的 YAML 驗證器 (例如[此工具](https://onlineyamltools.com/validate-yaml)) 來驗證 YAML 檔案的內容。

1. 儲存並關閉檔案。 在命令視窗中執行下列命令，並在出現提示時登入。 

    ```cmd/sh
    dotnet run CreateEndpoints
    ```

   此命令會建立事件方格的端點。 

   ![事件方格的端點](./media/tutorial-facilities-events/dotnet-create-endpoints.png)


## <a name="notify-events-with-logic-app"></a>使用 Logic Apps 發出事件通知
[Azure Logic Apps](../logic-apps/logic-apps-overview.md) 服務可讓您為接收自其他服務的事件建立自動化工作。 在本節中，您會設定 Logic Apps，藉助[事件方格主題](../event-grid/overview.md)來為空間感應器所路由傳送過來的事件建立電子郵件通知。

1. 在 [Azure 入口網站](https://portal.azure.com)的左側導覽窗格中，按一下 [建立資源]。

1. 搜尋並選取新的**邏輯應用程式**資源。 按一下頁面底部的 [新增] 。

1. 輸入邏輯應用程式的 [名稱]，然後選取 [訂用帳戶]、[資源群組] 和 [位置]。 按一下頁面底部的 [新增] 。

    ![建立邏輯應用程式](./media/tutorial-facilities-events/create-logic-app.png)

1. 開啟部署好的邏輯應用程式，然後開啟 [邏輯應用程式設計工具] 窗格。 

1. 選取 [發生事件方格事件時] 觸發程序。 當系統提示時，使用 Azure 帳戶**登入**租用戶。 出現提示時，確認要針對事件方格 [允許存取]。 按一下 [繼續]。

1. 在 [發生資源事件時 (預覽)] 視窗中： 
    1. 選取先前用來建立事件方格的 [訂用帳戶]。

    1. 選取 [Microsoft.EventGrid.Topics] 作為 [資源類型]。

    1. 從 [資源名稱] 下拉式方塊中選取事件方格資源。

    ![建立邏輯應用程式](./media/tutorial-facilities-events/logic-app-resource-event.png)

1. 按一下 [新增步驟] 按鈕。

1. 在 [選擇動作] 視窗中：
    1. 搜尋「剖析 json」片語，然後選取 [剖析 JSON] 動作。

    1. 在 [內容] 欄位內按一下，然後從 [動態內容] 清單中選取 [主體]。

    1. 按一下 [使用承載範例來產生結構描述]。 貼上下列 JSON 承載，然後按一下 [完成]。

        ```JSON
        {
        "id": "32162f00-a8f1-4d37-aee2-9312aabba0fd",
        "subject": "UdfCustom",
        "data": {
          "TopologyObjectId": "20efd3a8-34cb-4d96-a502-e02bffdabb14",
          "ResourceType": "Space",
          "Payload": "\"Air quality is poor.\"",
          "CorrelationId": "32162f00-a8f1-4d37-aee2-9312aabba0fd"
        },
        "eventType": "UdfCustom",
        "eventTime": "0001-01-01T00:00:00Z",
        "dataVersion": "1.0",
        "metadataVersion": "1",
        "topic": "/subscriptions/a382ee71-b48e-4382-b6be-eec7540cf271/resourceGroups/HOL/providers/Microsoft.EventGrid/topics/DigitalTwinEventGrid"
        }
        ```
    
    此承載有虛構值。 邏輯應用程式會使用此承載範例來產生 [結構描述]。
    
    ![事件方格的邏輯應用程式剖析 JSON](./media/tutorial-facilities-events/logic-app-parse-json.png)

1. 按一下 [新增步驟] 按鈕。

1. 在 [選擇動作] 視窗中：
    1. 在 [動作] 清單中搜尋並選取 [條件控制]。 

    1. 在第一個 [選擇值] 文字方塊內按一下，然後從 [剖析 JSON] 視窗的 [動態內容] 清單中選取 [eventType]。

    1. 在第二個 [選擇值] 文字方塊內按一下，然後輸入「UdfCustom」。

    ![事件方格的邏輯應用程式剖析 JSON](./media/tutorial-facilities-events/logic-app-condition.png)

1. 在 [若為 true] 視窗中：
    1. 按一下 [新增動作]，然後選取 [Office 365 Outlook]。

    1. 在 [動作] 的清單中，選取 [傳送電子郵件]。 按一下 [登入]，並使用電子郵件帳戶的認證。 出現提示時，按一下 [允許存取]。

    1. 在 [收件者] 方塊中，輸入要用來接收通知的電子郵件識別碼。 在 [主旨] 中，輸入「Digital Twins 針對空間內空氣品質不良的通知」這段文字，然後從 [剖析 JSON] 的 [動態內容] 清單中選取 [TopologyObjectId]。

    1. 在相同視窗的 [主體] 中，輸入如下文字：「房間內所偵測到的空氣品質不良，且需要調整溫度」。 您可以使用 [動態內容] 清單中的元素，任意地加上詳細說明，如下所示。

    ![邏輯應用程式傳送電子郵件](./media/tutorial-facilities-events/logic-app-send-email.png)

1. 按一下 [邏輯應用程式設計工具] 窗格頂端的 [儲存] 按鈕。

1. 請務必藉由在命令視窗中瀏覽至 Digital Twins 範例的 **device-connectivity** 資料夾，並執行 `dotnet run`，來模擬感應器資料。

在幾分鐘內，您應該就會開始從這個邏輯應用程式收到電子郵件通知。 

   ![邏輯應用程式傳送電子郵件](./media/tutorial-facilities-events/logic-app-notification.png)

若要停止接收這些電子郵件，請在入口網站中瀏覽至 [邏輯應用程式]，然後選取 [概觀] 窗格。 按一下 [停用]。


## <a name="clean-up-resources"></a>清除資源

如果您想要停止探索超過此時間點的 Azure Digital Twins，請放心地刪除在本教學課程中建立的資源：

1. 從 [Azure 入口網站](http://portal.azure.com)的左側功能表中，按一下 [所有資源]，選取您的 Digital Twins 資源群組，然後將它 [刪除]。
2. 如有需要，您可以繼續刪除您工作電腦上的範例應用程式。 


## <a name="next-steps"></a>後續步驟

您可以繼續進行下一個教學課程，以了解如何將感應器資料視覺化、分析趨勢並找出異常狀況。 
> [!div class="nextstepaction"]
> [教學課程：使用時間序列深入解析對來自 Azure Digital Twins 空間的事件進行視覺化檢視和分析](tutorial-facilities-analyze.md)

您也可以繼續進行，以深入了解 Azure Digital Twins 中的空間智慧圖形和物件模型。 
> [!div class="nextstepaction"]
> [了解 Digital Twins 物件模型和空間智慧圖形](concepts-objectmodel-spatialgraph.md)