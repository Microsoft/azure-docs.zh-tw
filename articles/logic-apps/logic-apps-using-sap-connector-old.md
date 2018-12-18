---
title: 連線到 SAP 系統 - Azure Logic Apps | Microsoft Docs
description: 如何透過 Azure Logic Apps 自動化執行工作流程，以存取和管理 SAP 資源
author: ecfan
manager: jeconnoc
ms.author: estfan
ms.date: 05/31/2018
ms.topic: article
ms.service: logic-apps
services: logic-apps
ms.reviewer: klam, divswa, LADocs
ms.suite: integration
tags: connectors
ms.openlocfilehash: 086aa846e278ddf2d64eca97e781463f73b868d0
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47057700"
---
# <a name="connect-to-sap-systems-from-azure-logic-apps"></a>從 Azure Logic Apps 連線至 SAP 系統

> [!NOTE]
> 此 SAP 連接器即將淘汰。 我們已發行更先進的全新 SAP 連接器，並建議您選擇或移動至[新的 SAP 連接器](./logic-apps-using-sap-connector.md)。
>  

本文顯示如何使用 SAP 應用程式伺服器和 SAP 訊息伺服器連接器，從邏輯應用程式內部存取 SAP 資源。 如此您即可建立邏輯應用程式，以自動化用來管理 SAP 資料和資源的工作、流程和工作流程。

此範例使用您可透過 HTTP 要求觸發的邏輯應用程式。 邏輯應用程式會將中繼文件 (IDoc) 傳送至 SAP 伺服器，並將回應傳回呼叫邏輯應用程式的要求者。
目前的 SAP 連接器有動作，但沒有觸發程序，因此這個範例使用 [HTTP 要求觸發程序](../connectors/connectors-native-reqres.md)做為邏輯應用程式工作流程中的第一個步驟。 如需 SAP 連接器專屬的技術資訊，請參閱這些參考文章。 

* <a href="https://docs.microsoft.com/connectors/sapapplicationserver/" target="blank">SAP 應用程式伺服器連接器</a>
* <a href="https://docs.microsoft.com/connectors/sapmessageserver/" target="blank">SAP 訊息伺服器連接器</a>

如果您還沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。

## <a name="prerequisites"></a>必要條件

若要遵循本文中的範例，您需要以下項目：

* 您要用來存取 SAP 系統的邏輯應用程式，以及啟動邏輯應用程式工作流程的觸發程序。 SAP 連接器目前僅提供動作。 如果您還不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)和[快速入門：建立第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

* 您的 <a href="https://wiki.scn.sap.com/wiki/display/ABAP/ABAP+Application+Server" target="_blank">SAP 應用程式伺服器</a>或 <a href="https://help.sap.com/saphelp_nw70/helpdata/en/40/c235c15ab7468bb31599cc759179ef/frameset.htm" target="_blank">SAP 訊息伺服器</a>

* 下載最新的[內部部署資料閘道](https://www.microsoft.com/download/details.aspx?id=53127)，並安裝在任何內部部署電腦中。 請務必先在 Azure 入口網站中設定閘道再繼續。 閘道可協助您安全地存取內部部署的資料和資源。 如需詳細資訊，請參閱[安裝 Azure Logic Apps 的內部部資料署閘道](../logic-apps/logic-apps-gateway-install.md)。

* 下載最新的 SAP 用戶端程式庫 (目前是<a href="https://softwaredownloads.sap.com/file/0020000000086282018" target="_blank">適用於 Microsoft .NET Framework 4.0 和 Windows 64 位元 (x64) 的 SAP Connector (NCo) 3.0.20.0</a>)，並安裝到與內部部署資料閘道所在的相同電腦上。 安裝此版本或更新版本的原因如下：

  * 同時傳送多則 IDoc 訊息時，較舊的 SAP NCo 版本可能會鎖死。 
  此狀況會封鎖所有傳送到 SAP 目的地的新訊息，導致訊息逾時。

  * 內部部署資料閘道僅能在 64 位元系統上執行。 
  否則，您會遭遇「映像錯誤」錯誤，因為資料閘道主機服務不支援 32 位元組件。

  * 資料閘道主機服務和 Microsoft SAP Adapter 皆使用 .NET Framework 4.5。 適用於 .NET Framework 4.0 的 SAP NCo 可執行採用 .NET runtime 4.0 到 4.7.1 版的流程。 
  適用於 .NET Framework 2.0 的 SAP NCo 可執行採用 .NET runtime 2.0 到 3.5 版的流程，但已無法使用最新版內部部署資料閘道。

* 您可傳送至 SAP 伺服器的訊息內容，如 IDoc 範例檔。 此內容必須是 XML 格式，並且包含您要使用的 SAP 動作的命名空間。

<a name="add-trigger"></a>

## <a name="add-http-request-trigger"></a>新增 HTTP 要求觸發程序

在 Azure Logic Apps 中，每個邏輯應用程式都必須使用[觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)啟動，而該觸發程序會在特定事件發生或符合特定條件時引發。 每次引發觸發程序時，Logic Apps 引擎都會建立邏輯應用程式執行個體，並開始執行應用程式的工作流程。

在此範例中，您可使用 Azure 中的端點建立邏輯應用程式，如此即可將 *HTTP POST 要求*傳送至您的邏輯應用程式。 邏輯應用程式收到這些 HTTP 要求後，觸發程序便會在您的工作流程中引發並執行後續步驟。

1. 在 Azure 入口網站中，建立空白的邏輯應用程式，以開啟邏輯應用程式設計工具。 

2. 在搜尋方塊中，輸入「http 要求」做為篩選條件。 從觸發程序清單中選取此觸發程序：[要求 - 收到 HTTP 要求時]

   ![新增 HTTP 要求觸發程序](./media/logic-apps-using-sap-connector-old/add-trigger.png)

3. 現在請儲存邏輯應用程式，即可產生邏輯應用程式的端點 URL。
在設計工具的工具列上，選擇 [儲存]。 

   端點 URL 現在會出現在您的觸發程序中，例如：

   ![產生端點的 URL](./media/logic-apps-using-sap-connector-old/generate-http-endpoint-url.png)

<a name="add-action"></a>

## <a name="add-sap-action"></a>新增 SAP 動作

在 Azure Logic Apps 中，[動作](../logic-apps/logic-apps-overview.md#logic-app-concepts)是工作流程中跟隨在觸發程序或另一個動作之後的步驟。 如果您尚未新增觸發程序至邏輯應用程式，並且想要遵循此範例進行，請[按照本節所述指示新增觸發程序](#add-trigger)。

1. 在邏輯應用程式設計工具的觸發程序之下，依序選擇 [新增步驟]  >  [新增動作]。

   ![新增動作](./media/logic-apps-using-sap-connector-old/add-action.png) 

2. 在搜尋方塊中，輸入「sap 伺服器」做為篩選條件。 從 [動作] 清單中，為 SAP 伺服器選取動作。 

   * **SAP 應用程式伺服器 - 傳送至 SAP**
   * **SAP 訊息伺服器 - 傳送至 SAP**

   此範例使用此動作：[SAP 應用程式伺服器 - 傳送至 SAP]

   ![選取 [SAP 應用程式伺服器] 或 [SAP 訊息伺服器]](media/logic-apps-using-sap-connector-old/select-sap-action.png)

3. 如果系統提示您輸入連線詳細資料，請立即建立 SAP 連線。 或者，若您的連線已存在，請繼續執行後續步驟，以設定 SAP 動作。 

   **建立內部部署 SAP 連線**

   1. 針對 [閘道]，請選取 [透過內部部署資料閘道連線]，如此就會顯示內部部署連線屬性。

   2. 提供 SAP 伺服器的連線資訊。 
   針對 [閘道] 屬性，請選取您在 Azure 入口網站中建立用於安裝閘道的資料閘道，例如：

      **SAP 應用程式伺服器**

      ![建立 SAP 應用程式伺服器連線](./media/logic-apps-using-sap-connector-old/create-SAP-app-server-connection.png)  

      **SAP 訊息伺服器**

      ![建立 SAP 訊息伺服器連線](media/logic-apps-using-sap-connector-old/create-SAP-message-server-connection.png) 

   2. 完成之後，請選擇 [建立]。

      Logic Apps 會設定並測試連線，以確定連線運作正常。

4. 立即尋找並從 SAP 伺服器選取動作。 

   1. 在 [SAP 動作] 方塊中選擇資料夾圖示。 
   在資料夾清單中，尋找並選取您要使用的動作。 

      此範例為 IDoc 動作選取 **IDOC** 類別。 

      ![尋找並選取 IDoc 動作](./media/logic-apps-using-sap-connector-old/SAP-app-server-find-action.png)

      如果找不到您需要的動作，您可手動輸入路徑，例如：

      ![手動提供路徑給 IDoc 動作](./media/logic-apps-using-sap-connector-old/SAP-app-server-manually-enter-action.png)

      如需有關 IDoc 作業的詳細資訊，請參閱 [IDOC 作業的訊息結構描述](https://docs.microsoft.com/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations)

   2. 在 [輸入訊息] 方塊內按一下，動態內容清單隨即顯示。 
   在該清單中的 收到 HTTP 要求時 之下，選取 內文 欄位。 

      此步驟包含來自 HTTP 要求觸發程序的內文內容，且可傳送該輸出結果至您的 SAP 伺服器。

      ![選取 [內文] 欄位](./media/logic-apps-using-sap-connector-old/SAP-app-server-action-select-body.png)

      完成上述步驟後，SAP 動作看起來便如下列範例︰

      ![完整的 SAP 動作](./media/logic-apps-using-sap-connector-old/SAP-app-server-complete-action.png)

6. 儲存您的邏輯應用程式。 在設計工具的工具列上，選擇 [儲存]。

<a name="add-response"></a>

## <a name="add-http-response-action"></a>新增 HTTP 回應動作

現在請將回應動作新增至邏輯應用程式的工作流程，並納入 SAP 動作的輸出結果。 如此一來，邏輯應用程式便會從 SAP 伺服器將結果傳回至原始的要求者。 

1. 在邏輯應用程式設計工具的 SAP 動作之下，依序選擇 [新增步驟]  >  [新增動作]。

2. 在搜尋方塊中，輸入「回應」做為篩選條件。 從 [動作] 清單中，選取此動作：[要求 - 回應]

3. 在 [內文] 方塊內按一下，動態內容清單隨即顯示。 從該清單的 [傳送至 SAP] 之下，選取 [內文] 欄位。 

   ![完整的 SAP 動作](./media/logic-apps-using-sap-connector-old/select-sap-body-for-response-action.png)

4. 儲存您的邏輯應用程式。 

## <a name="test-your-logic-app"></a>測試應用程式邏輯

1. 如果邏輯應用程式尚未啟用，請在邏輯應用程式功能表上，選擇 [概觀]。 在工具列上，選擇 [啟用]。 

2. 在「邏輯應用程式設計工具」的工具列上，選擇 [執行]。 此步驟會手動啟動您的邏輯應用程式。

3. 將 HTTP POST 要求傳送至 HTTP 要求觸發程序中的 URL，以觸發邏輯應用程式，然後再透過要求加入訊息內容。 若要傳送內容，您可使用 [Postman](https://www.getpostman.com/apps) 之類的工具。 

   在本文中，要求會傳送 IDoc 檔案，其必須為 XML 格式並且包含您要使用的 SAP 動作的命名空間，例如： 

   ``` xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <Send xmlns="http://Microsoft.LobServices.Sap/2007/03/Idoc/3/ORDERS05//620/Send">
      <idocData>
         <...>
      </idocData>
   </Send>
   ```

4. 傳送 HTTP 要求後，請等候來自邏輯應用程式的回應。

> [!NOTE]
> 如果回應所需的所有步驟未在[要求逾時期限](./logic-apps-limits-and-config.md)內完成，邏輯應用程式可能會逾時。 如果發生此情況，要求可能就會遭到封鎖。 為協助您診斷問題，請了解如何[檢查和監控邏輯應用程式](../logic-apps/logic-apps-monitor-your-logic-apps.md)。

恭喜，您已經成功建立能與 SAP 伺服器通訊的邏輯應用程式。 您現在已為邏輯應用程式設定 SAP 連線，接下來可探索其他可用的 SAP 動作如：BAPI 和 RFC。

## <a name="connector-reference"></a>連接器參考

如需連接器 Swagger 檔案中所述的連接器相關技術詳細資料，請參閱以下參考文章： 

* [SAP 應用程式伺服器](/connectors/sapapplicationserver/)
* [SAP 訊息伺服器](/connectors/sapmessageserver/)

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 從邏輯應用程式[連線至內部部署系統](../logic-apps/logic-apps-gateway-connection.md)
* 了解驗證和轉換方式，以及[企業整合套件](../logic-apps/logic-apps-enterprise-integration-overview.md)中的其他訊息作業
* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)
