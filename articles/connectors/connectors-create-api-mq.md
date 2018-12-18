---
title: 連線到 MQ 伺服器 - Azure Logic Apps | Microsoft Docs
description: 使用 Azure 或內部部署 MQ 伺服器和 Azure Logic Apps 來傳送及取出訊息
author: valthom
manager: jeconnoc
ms.author: valthom
ms.date: 06/01/2017
ms.topic: article
ms.service: logic-apps
services: logic-apps
ms.reviewer: klam, LADocs
ms.suite: integration
tags: connectors
ms.openlocfilehash: 6b34bd7b286ca3b206c611343217c90e0d57fbfb
ms.sourcegitcommit: 6f6d073930203ec977f5c283358a19a2f39872af
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35295905"
---
# <a name="connect-to-an-ibm-mq-server-from-logic-apps-using-the-mq-connector"></a>使用 MQ 連接器從 Logic Apps 連線到 IBM MQ Server 

Microsoft Connector for MQ 會傳送並取出儲存在 MQ Server 內部部署或在 Azure 中的訊息。 此連接器包含透過 TCP/IP 網路與遠端 IBM MQ Server 通訊的 Microsoft MQ 用戶端。 本文件是使用 MQ 連接器的入門指南。 建議您從瀏覽佇列上的單一訊息開始，然後再嘗試其他動作。    

MQ 連接器包含下列動作。 但不包含觸發程序。

-   瀏覽單一訊息，而不從 IBM MQ Server 刪除訊息
-   瀏覽批次訊息，而不從 IBM MQ Server 刪除訊息
-   接收單一訊息，並從 IBM MQ Server 刪除訊息
-   接收批次訊息，並從 IBM MQ Server 刪除訊息
-   將單一訊息傳送至 IBM MQ Server 

## <a name="prerequisites"></a>先決條件

* 如果是使用內部部署 MQ Server，請在網路內之伺服器上[安裝內部部署資料閘道](../logic-apps/logic-apps-gateway-install.md)。 如果 MQ Server 可公開使用或可在 Azure 內使用，就是沒有使用資料閘道或不需要資料閘道。

    > [!NOTE]
    > 安裝內部部署資料閘道的伺服器也必須安裝適用於 MQ 連接器的 .Net Framework 4.6 才能運作。

* 建立內部部署資料閘道的 Azure 資源 - [設定資料閘道連線](../logic-apps/logic-apps-gateway-connection.md)。

* 正式支援的 IBM WebSphere MQ 版本：
   * MQ 7.5
   * MQ 8.0

## <a name="create-a-logic-app"></a>建立邏輯應用程式

1. 在 **Azure 開始面板**中，依序選取 **+** (加號)、[Web + 行動] 和 [邏輯應用程式]。 
2. 輸入 [名稱]，例如 MQTestApp、[訂用帳戶]、[資源群組]，和 [位置](使用已設定內部部署資料閘道連線的位置)。 選取 [釘選到儀表板]，然後選取 [建立]。  
![建立邏輯應用程式](media/connectors-create-api-mq/Create_Logic_App.png)

## <a name="add-a-trigger"></a>新增觸發程序

> [!NOTE]
> MQ 連接器並沒有任何觸發程序。 因此，使用另一個觸發程序來啟動邏輯應用程式，例如 [定期] 觸發程序。 

1. **Logic Apps Designer** 隨即開啟，在一般觸發程序的清單中選取 [定期]。
2. 在定期觸發程序內選取 [編輯]。 
3. 將 [頻率] 設定為 [天]，將 [間隔] 設定為 **7**。 

## <a name="browse-a-single-message"></a>瀏覽單一訊息
1. 選取 [+ 新增步驟]，再選取 [新增動作]。
2. 在 [搜尋] 方塊中，輸入 `mq`，然後選取 [MQ - 瀏覽訊息]。  
![瀏覽訊息](media/connectors-create-api-mq/Browse_message.png)

3. 如果沒有現有的 MQ 連線，就建立連線：  

    1. 選取 [透過內部部署資料閘道連線]，並輸入 MQ Server 的屬性。  
    針對 [伺服器]，您可以輸入 MQ Server 的名稱，或輸入 IP 位址，後面接著冒號和連接埠號碼。 
    2. [閘道] 下拉式清單中會列出已設定的任何現有閘道連線。 選取您的閘道。
    3. 完成後，請選取 [建立]。 連線看起來如下所示：   
    ![連線屬性](media/connectors-create-api-mq/Connection_Properties.png)

4. 在動作屬性中，您可以：  

    * 使用 **Queue** 屬性來存取與連線中所定義不同的佇列名稱
    * 使用 **MessageId**、**CorrelationId**、**GroupId** 和其他屬性，來瀏覽以不同 MQ 訊息屬性作為基礎的訊息
    * 將 **IncludeInfo** 設定為 **True**，可在輸出中包含額外的訊息資訊。 或者，將它設定為 **False**，可在輸出中排除額外的訊息資訊。
    * 輸入 [逾時] 值，來決定訊息到達空白查詢中的等候時間。 如果未輸入任何內容，就會取出佇列中的第一個訊息，且不會花費時間等候要顯示的訊息。  
    ![瀏覽訊息屬性](media/connectors-create-api-mq/Browse_message_Props.png)

5. [儲存] 您的變更，然後 [執行] 邏輯應用程式：  
![儲存並執行](media/connectors-create-api-mq/Save_Run.png)

6. 幾秒之後，會顯示執行的步驟，且您可以查看輸出。 選擇綠色核取記號可查看每個步驟的詳細資料。 選取 [查看原始輸出] 可查看輸出資料上的其他詳細資料。  
![瀏覽訊息輸出](media/connectors-create-api-mq/Browse_message_output.png)  

    原始輸出：  
    ![瀏覽訊息原始輸出](media/connectors-create-api-mq/Browse_message_raw_output.png)

7. 當 **IncludeInfo** 選項設定為 true 時，會顯示下列輸出：  
![瀏覽訊息包含資訊](media/connectors-create-api-mq/Browse_message_Include_Info.png)

## <a name="browse-multiple-messages"></a>瀏覽多個訊息
[瀏覽訊息] 動作包含 **BatchSize** 選項，指出應該從佇列傳回的訊息數量。  如果 **BatchSize** 沒有項目，就會傳回所有的訊息。 傳回的輸出是陣列訊息。

1. 新增 [瀏覽訊息] 動作時，預設會選取所設定的第一個連線。 選取 [變更連線] 來建立新的連線，或選取不同的連線。

2. 瀏覽訊息的輸出會顯示：  
![瀏覽訊息輸出](media/connectors-create-api-mq/Browse_messages_output.png)

## <a name="receive-a-single-message"></a>接收單一訊息
[接收訊息] 動作具有與 [瀏覽訊息] 動作相同的輸入和輸出。 使用 [接收訊息] 時，會從佇列將訊息刪除。

## <a name="receive-multiple-messages"></a>接收多個訊息
[接收訊息] 動作具有與 [瀏覽訊息] 動作相同的輸入和輸出。 使用 [接收訊息] 時，會從佇列將訊息刪除。

如果進行瀏覽或接收時佇列中沒有任何訊息，步驟就會失敗，並出現下列輸出：  
![MQ 無訊息錯誤](media/connectors-create-api-mq/MQ_No_Msg_Error.png)

## <a name="send-a-message"></a>傳送訊息
1. 新增 [傳送訊息] 動作時，預設會選取所設定的第一個連線。 選取 [變更連線] 來建立新的連線，或選取不同的連線。 有效的 [訊息類型] 是 [資料包]、[回覆] 或 [要求]。  
![傳送訊息屬性](media/connectors-create-api-mq/Send_Msg_Props.png)

2. 傳送訊息的輸出顯示如下：  
![傳送訊息輸出](media/connectors-create-api-mq/Send_Msg_Output.png)

## <a name="connector-specific-details"></a>連接器特定的詳細資料

檢視 Swagger 中定義的任何觸發程序和動作，另請參閱[連接器詳細資料](/connectors/mq/)的所有限制。

## <a name="next-steps"></a>後續步驟
[建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 請到我們的 [API 清單](apis-list.md)探索 Logic Apps 中其他可用的連接器。
