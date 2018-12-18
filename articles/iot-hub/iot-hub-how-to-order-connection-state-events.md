---
title: 使用 Azure Cosmos DB 排序來自 Azure IoT 中樞的裝置連線事件 | Microsoft Docs
description: 本文說明如何使用 Azure Cosmos DB 排序和記錄來自 Azure IoT 中樞的裝置連線事件，以維持最新的連接狀態
services: iot-hub
documentationcenter: ''
author: ash2017
manager: briz
editor: ''
ms.service: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/06/2018
ms.author: asrastog
ms.openlocfilehash: dd3c750e93646624e46c46afd871ef75af905bf0
ms.sourcegitcommit: a1140e6b839ad79e454186ee95b01376233a1d1f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43145452"
---
# <a name="order-device-connection-events-from-azure-iot-hub-using-azure-cosmos-db"></a>使用 Azure Cosmos DB 排序來自 Azure IoT 中樞的裝置連線事件

Azure 事件方格可協助您建置以事件為基礎的應用程式，並輕鬆將 IoT 事件整合至您的商業解決方案中。 本文將逐步引導您完成可用來在 Cosmos DB 中追蹤和儲存最新裝置連線狀態的設定。 我們將使用「裝置連線」和「裝置中斷連線」事件中的可用序號，並將最新狀態儲存在 Cosmos DB 中。 我們將使用預存程序，也就是對 Cosmos DB 中的集合執行的應用程式邏輯。

序號是十六進位數字的字串表示法。 您可以使用字串比較來識別較大的號碼。 如果您想要將字串轉換為十六進位，號碼將是 256 位元的數字。 序號會持續增加，且最新事件的號碼會高於其他事件。 如果您的裝置經常連線和中斷連線，而您想要確保一律會以最新的事件來觸發下游動作，這將有其效用，因為 Azure 事件方格不支援事件的排序。

## <a name="prerequisites"></a>必要條件

* 使用中的 Azure 帳戶。 如果您沒有帳戶，您可以[建立免費帳戶](http://azure.microsoft.com/pricing/free-trial/)。
* 使用中的 Azure Cosmos DB SQL API 帳戶。 如果您尚未建立此帳戶，請參閱[建立資料庫帳戶](https://docs.microsoft.com/azure/cosmos-db/create-sql-api-dotnet#create-a-database-account)以取得逐步解說。
* 您資料庫中的集合。 請參閱[新增集合](https://docs.microsoft.com/azure/cosmos-db/create-sql-api-dotnet#add-a-collection)以取得逐步解說。
* Azure 中的 IoT 中樞。 若尚未建立，請參閱[開始使用 IoT 中樞](../iot-hub/iot-hub-csharp-csharp-getstarted.md)的逐步解說。 

## <a name="create-a-stored-procedure"></a>建立預存程序

首先，建立預存程序並加以設定，使其執行會比較傳入事件的序號，並將每個裝置的最新事件記錄在資料庫中的邏輯。

1. 在您的 Cosmos DB SQL API 中，選取 [資料總管] > [項目] > [新增預存程序]。

   ![建立預存程序](./media/iot-hub-how-to-order-connection-state-events/create-stored-procedure.png)

2. 輸入預存程序識別碼，然後將下列內容貼到「預存程序主體」中。 請注意，此程式碼應取代預存程序主體中任何現有的程式碼。 此程式碼會為每個裝置識別碼維護一個資料列，並藉由識別最高的序號，來記錄該裝置識別碼的最新連線狀態。 

    ```javascript
    // SAMPLE STORED PROCEDURE
    function UpdateDevice(deviceId, moduleId, hubName, connectionState, connectionStateUpdatedTime, sequenceNumber) {
      var collection = getContext().getCollection();
      var response = {};
      
      var docLink = getDocumentLink(deviceId, moduleId);

      var isAccepted = collection.readDocument(docLink, function(err, doc) {
        if (err) {
          console.log('Cannot find device ' + docLink + ' - ');
          createDocument();
        } else {
          console.log('Document Found - ');
          replaceDocument(doc);
        }
      });

      function replaceDocument(document) {
        console.log(
          'Old Seq :' +
            document.sequenceNumber +
            ' New Seq: ' +
            sequenceNumber +
            ' - '
        );
        if (sequenceNumber > document.sequenceNumber) {
          document.connectionState = connectionState;
          document.connectionStateUpdatedTime = connectionStateUpdatedTime;
          document.sequenceNumber = sequenceNumber;

          console.log('replace doc - ');

          isAccepted = collection.replaceDocument(docLink, document, function(
            err,
            updated
          ) {
            if (err) {
              getContext()
                .getResponse()
                .setBody(err);
            } else {
              getContext()
                .getResponse()
                .setBody(updated);
            }
          });
        } else {
          getContext()
            .getResponse()
            .setBody('Old Event - current: ' + document.sequenceNumber + ' Incoming: ' + sequenceNumber);
        }
      }
      function createDocument() {
        document = {
          id: deviceId + '-' + moduleId,
          deviceId: deviceId,
          moduleId: moduleId,
          hubName: hubName,
          connectionState: connectionState,
          connectionStateUpdatedTime: connectionStateUpdatedTime,
          sequenceNumber: sequenceNumber
        };
        console.log('Add new device - ' + collection.getAltLink());
        isAccepted = collection.createDocument(
          collection.getAltLink(),
          document,
          function(err, doc) {
            if (err) {
              getContext()
                .getResponse()
                .setBody(err);
            } else {
              getContext()
                .getResponse()
                .setBody(doc);
            }
          }
        );
      }

      function getDocumentLink(deviceId, moduleId) {
        return collection.getAltLink() + '/docs/' + deviceId + '-' + moduleId;
      }
    }
    ```

3. 儲存預存程序： 

    ![儲存預存程序](./media/iot-hub-how-to-order-connection-state-events/save-stored-procedure.png)

## <a name="create-a-logic-app"></a>建立邏輯應用程式

首先，建立邏輯應用程式，並新增可監視您虛擬機器資源群組的 Event Grid 觸發程序。 

### <a name="create-a-logic-app-resource"></a>建立邏輯應用程式資源

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取 [新增] > [整合] > [邏輯應用程式]。

   ![建立邏輯應用程式](./media/iot-hub-how-to-order-connection-state-events/select-logic-app.png)

2. 在訂用帳戶中為邏輯應用程式指定唯一的名稱，然後選取與您的 IoT 中樞相同的訂用帳戶、資源群組和位置。 
3. 當您準備就緒，選取 [釘選到儀表板]，然後選擇 [建立]。

   您現在已經為您的應用程式邏輯建立一項 Azure 資源。 在 Azure 部署您的邏輯應用程式之後，Logic Apps 設計工具會顯示常見模式的範本，以便您更快開始使用。

   > [!NOTE] 
   > 當您選取 [釘選到儀表板] 時，邏輯應用程式會自動在 Logic Apps 設計工具中開啟。 不然，您可以手動尋找並開啟您的邏輯應用程式。

4. 在 [範本] 下的 [邏輯應用程式設計工具] 中，選擇 [空白邏輯應用程式]，以便從頭建置邏輯應用程式。

### <a name="select-a-trigger"></a>選取觸發程序

觸發程序是啟動邏輯應用程式的特定事件。 在本教學課程中，啟動工作流程的觸發程序會透過 HTTP 接收要求。  

1. 在連接器和觸發程序搜尋列中，輸入 **HTTP**。
2. 選取 [要求 - 收到 HTTP 要求時] 作為觸發程序。 

   ![選取 HTTP 要求觸發程序](./media/iot-hub-how-to-order-connection-state-events/http-request-trigger.png)

3. 選取 [使用範例承載來產生結構描述]。 

   ![選取 HTTP 要求觸發程序](./media/iot-hub-how-to-order-connection-state-events/sample-payload.png)

4. 將下列 JSON 程式碼範例貼到文字方塊中，然後選取 [完成]：

   ```json
   [{
    "id": "fbfd8ee1-cf78-74c6-dbcf-e1c58638ccbd",
    "topic":
      "/SUBSCRIPTIONS/DEMO5CDD-8DAB-4CF4-9B2F-C22E8A755472/RESOURCEGROUPS/EGTESTRG/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/MYIOTHUB",
    "subject": "devices/Demo-Device-1",
    "eventType": "Microsoft.Devices.DeviceConnected",
    "eventTime": "2018-07-03T23:20:11.6921933+00:00",
    "data": {
      "deviceConnectionStateEventInfo": {
        "sequenceNumber":
          "000000000000000001D4132452F67CE200000002000000000000000000000001"
      },
      "hubName": "MYIOTHUB",
      "deviceId": "48e44e11-1437-4907-83b1-4a8d7e89859e",
      "moduleId": ""
    },
    "dataVersion": "1",
    "metadataVersion": "1"
   }]
   ```

5. 您可能會收到快顯通知，指出**請務必在您的要求中加入 Content-Type 標頭，並將其設為 application/json。** 您可以放心地忽略這項建議，並移至下一節。 

### <a name="create-a-condition"></a>建立條件

條件有助於在特定條件通過後，在您的邏輯應用程式工作流程中執行特定動作。 一旦符合條件，就可以定義所需的動作。 在本教學課程中，條件是要檢查 eventType 是裝置連線還是中斷連線。 此動作將會在您的資料庫中執行預存程序。 

1. 選取 [新增步驟]，然後選取 [內建項目] 和 [條件]。 

2. 依照下列方式填入條件，以僅對「裝置連線」和「裝置中斷連線」事件執行此動作：
  * 選擇值：**eventType**
  * 將「等於」變更為**結尾是**
  * 選擇值：**nected**

   ![填入條件](./media/iot-hub-how-to-order-connection-state-events/condition-detail.png)

3. 如果條件為 true，請按一下 [新增動作]。
  
   ![若為 true 則新增動作](./media/iot-hub-how-to-order-connection-state-events/action-if-true.png)

4. 搜尋 Cosmos DB，然後按一下 [Azure Cosmos DB - 執行預存程序]

   ![搜尋 CosmosDB](./media/iot-hub-how-to-order-connection-state-events/cosmosDB-search.png)

5. 從您的資料庫中選取值，以填入「執行」預存程序的表單。 輸入分割區索引鍵值和參數，如下所示。 

   ![填入邏輯應用程式動作](./media/iot-hub-how-to-order-connection-state-events/logicapp-stored-procedure.png)

6. 儲存您的邏輯應用程式。 

### <a name="copy-the-http-url"></a>複製 HTTP URL

離開 Logic Apps 設計工具之前，為觸發程序複製邏輯應用程式正在接聽的 URL。 您可以使用此 URL 來設定 Event Grid。 

1. 按一下以展開 [收到 HTTP 要求時] 觸發程序設定方塊。 
2. 選取 [HTTP POST URL] 旁的複製按鈕來複製其值。 

   ![複製 HTTP POST URL](./media/iot-hub-how-to-order-connection-state-events/copy-url.png)

3. 儲存這個 URL，以便在下一節參考它。 

## <a name="configure-subscription-for-iot-hub-events"></a>設定 IoT 中樞事件的訂用帳戶

在本節中，您可將 IoT 中樞設定為在事件發生時進行發佈。 

1. 在 Azure 入口網站中，瀏覽至您的 IoT 中樞。 
2. 選取 [事件]。

   ![開啟 Event Grid 詳細資料](./media/iot-hub-how-to-order-connection-state-events/event-grid.png)

3. 選取 [事件訂用帳戶]。 

   ![建立新的事件訂用帳戶](./media/iot-hub-how-to-order-connection-state-events/event-subscription.png)

4. 使用下列值來建立事件訂用帳戶： 
   * **事件類型**：取消核取 [訂閱所有事件類型]，然後從功能表中選取 [裝置連線] 和 [裝置中斷連線]。
   * **端點詳細資料**：選取 [Web Hook] 作為 [端點類型]，然後點選端點，再貼上您從邏輯應用程式複製的 URL，並確認您的選取。

   ![選取端點 URL](./media/iot-hub-how-to-order-connection-state-events/endpoint-url.png)

   * **事件訂用帳戶詳細資料**：提供描述性的名稱，然後選取 [事件格線結構描述]。作業完成後，表單應會如下列範例所示： 

   ![事件訂用帳戶表單範例](./media/iot-hub-how-to-order-connection-state-events/subscription-form.png)

5. 選取 [建立] 以儲存事件訂用帳戶。

## <a name="observe-events"></a>觀察事件
現在您已設定事件訂用帳戶，接下來我們將連接裝置以進行測試。

### <a name="register-a-device-in-iot-hub"></a>在 IoT 中樞登錄裝置

1. 從您的 IoT 中樞，選取 [IoT 裝置]。 
2. 選取 [新增] 。
3. 在 [裝置識別碼] 中，輸入 `Demo-Device-1`。
4. 選取 [ **儲存**]。 
5. 您可以使用不同的裝置識別碼來新增多個裝置。

   ![操作結果](./media/iot-hub-how-to-order-connection-state-events/AddIoTDevice.png)

6. 複製**連接字串 -- 主索引鍵**，以供日後使用。

   ![操作結果](./media/iot-hub-how-to-order-connection-state-events/DeviceConnString.png)

### <a name="start-raspberry-pi-simulator"></a>啟動 Raspberry Pi 模擬器

1. 我們將使用 Raspberry Pi Web 模擬器來模擬裝置連線。

[啟動 Raspberry Pi 模擬器](https://azure-samples.github.io/raspberry-pi-web-simulator/#Getstarted)

### <a name="run-a-sample-applciation-on-the-raspberry-pi-web-simulator"></a>在 Raspberry Pi Web 模擬器上執行範例應用程式
這將會觸發裝置連線事件。

1. 在編碼區域中，將行 15 中的預留位置取代為您的 Azure IoT 中樞裝置連接字串。

   ![操作結果](./media/iot-hub-how-to-order-connection-state-events/raspconnstring.png)

2. 按一下 [執行] 以執行應用程式。

您應該會看見下列輸出，顯示傳送至 IoT 中樞的感應器資料和訊息。

   ![操作結果](./media/iot-hub-how-to-order-connection-state-events/raspmsg.png)

   按一下 [停止] 以停止模擬器，然後觸發 [裝置中斷連線] 事件。

現在，您已執行範例應用程式以收集感應器資料，並將其傳送至 IoT 中樞。 

### <a name="observe-events-in-cosmos-db"></a>觀察 Cosmos DB 中的事件

您可以在 Cosmos DB 文件中查看執行預存程序的結果。 它將顯示如下。 請注意，每個資料列分別包含每個裝置的最新裝置連線狀態

   ![操作結果](./media/iot-hub-how-to-order-connection-state-events/cosmosDB-outcome.png)

## <a name="use-the-azure-cli"></a>使用 Azure CLI

您不必使用 Azure 入口網站，可改用 Azure CLI 來完成 IoT 中樞步驟。 如需詳細資訊，請參閱 Azure CLI 頁面：[建立事件訂用帳戶](https://docs.microsoft.com/cli/azure/eventgrid/event-subscription)和[建立 IoT 裝置](https://docs.microsoft.com/cli/azure/iot/device)。

## <a name="clean-up-resources"></a>清除資源

本教學課程使用了會使您的 Azure 訂用帳戶產生費用的資源。 完成教學課程試用和結果測試時，請停用或刪除不想保留的資源。 

如果不想遺失您在應用程式邏輯上所做的工作，請將它停用，而非刪除它。 

1. 瀏覽至您的邏輯應用程式。
2. 在 [概觀] 刀鋒視窗上選取 [刪除]或 [停用]。 

每個訂用帳戶都可以有一個免費的 IoT 中樞。 如果您為本教學課程建立了免費中樞，則不必為了避免費用而將它刪除。

1. 瀏覽至您的 IoT 中樞。 
2. 在 [概觀] 刀鋒視窗上選取 [刪除]。 

即使要保留 IoT 中樞，您也可以刪除您所建立的事件訂用帳戶。 

1. 在 IoT 中樞中，選取 [Event Grid]。
2. 選取您想要移除的事件訂用帳戶。 
3. 選取 [刪除] 。 

若要從 Azure 入口網站中移除 Azure Cosmos DB 帳戶，請以滑鼠右鍵按一下帳戶名稱，然後按一下 [刪除帳戶]。 請參閱[刪除 Azure Cosmos DB 帳戶](https://docs.microsoft.com/azure/cosmos-db/manage-account#delete)中的詳細指示。

## <a name="next-steps"></a>後續步驟

* 深入了解如何[使用 Event Grid 來觸發動作以回應 IoT 中樞事件](../iot-hub/iot-hub-event-grid.md)
* [試試 IoT 中樞事件教學課程](../event-grid/publish-iot-hub-events-to-logic-apps.md)
* 了解您還可使用[事件方格](../event-grid/overview.md)執行哪些作業


