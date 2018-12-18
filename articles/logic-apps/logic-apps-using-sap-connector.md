---
title: 連線到 SAP 系統 - Azure Logic Apps | Microsoft Docs
description: 如何透過 Azure Logic Apps 自動化執行工作流程，以存取和管理 SAP 資源
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: divswa, LADocs
ms.topic: article
ms.date: 09/14/2018
tags: connectors
ms.openlocfilehash: 13268ec865ec72fce23df550619b199389096360
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47056500"
---
# <a name="connect-to-sap-systems-from-azure-logic-apps"></a>從 Azure Logic Apps 連線至 SAP 系統

本文顯示如何使用 SAP ERP Central Component (ECC) 連接器，從邏輯應用程式內部存取內部部署 SAP 資源。 SAP ECC 連接器支援透過中繼文件 (IDoc)、商務應用程式開發介面 (BAPI) 或遠端函式呼叫 (RFC) 來進行訊息或資料與 SAP Netweaver 型系統的雙向整合。

SAP ECC 連接器會使用 <a href="https://help.sap.com/saphelp_nwpi71/helpdata/en/e9/23c80d66d08c4c8c044a3ea11ca90f/frameset.htm">SAP .Net Connector (NCo) 程式庫</a>，並提供下列作業或動作：

- **傳送至 SAP**：在 SAP 系統中透過 tRFC 傳送 IDoc 或呼叫 BAPI 函式。
- **從 SAP 接收**：從 SAP 系統透過 tRFC 接收 IDoc 或 BAPI 函式呼叫。
- **產生結構描述**：針對 IDoc、BAPI 或 RFC 的 SAP 成品產生結構描述。

SAP 連接器會透過[內部部署資料閘道](https://www.microsoft.com/download/details.aspx?id=53127)與內部部署 SAP 系統整合。 例如，在傳送案例中，若從 Logic Apps 傳送訊息到 SAP 系統，資料閘道會作為 RFC 用戶端，並將接收自 Logic Apps 的要求轉送到 SAP。
同樣地，在接收案例中，資料閘道會作為 RFC 伺服器，以從 SAP 接收要求並轉送至 Logic Apps。 

本文說明如何建立會與 SAP 整合的邏輯應用程式範例，同時也會說明先前所述的整合案例。

## <a name="prerequisites"></a>必要條件

若要遵循本文中的範例，您需要以下項目：

* Azure 訂用帳戶。 如果您還沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。

* 您要用來存取 SAP 系統的邏輯應用程式，以及啟動邏輯應用程式工作流程的觸發程序。 如果您還不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)和[快速入門：建立第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。

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

## <a name="send-to-sap"></a>傳送至 SAP

此範例使用您可透過 HTTP 要求觸發的邏輯應用程式。 邏輯應用程式會將中繼文件 (IDoc) 傳送至 SAP 伺服器，並將回應傳回呼叫邏輯應用程式的要求者。 

### <a name="add-http-request-trigger"></a>新增 HTTP 要求觸發程序

在 Azure Logic Apps 中，每個邏輯應用程式都必須使用[觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)啟動，而該觸發程序會在特定事件發生或符合特定條件時引發。 每次引發觸發程序時，Logic Apps 引擎都會建立邏輯應用程式執行個體，並開始執行應用程式的工作流程。

在此範例中，您可使用 Azure 中的端點建立邏輯應用程式，如此即可將 *HTTP POST 要求*傳送至您的邏輯應用程式。 邏輯應用程式收到這些 HTTP 要求後，觸發程序便會在您的工作流程中引發並執行後續步驟。

1. 在 [Azure 入口網站](https://portal.azure.com)中，建立空白的邏輯應用程式，以開啟邏輯應用程式設計工具。 

2. 在搜尋方塊中，輸入「http 要求」做為篩選條件。 從觸發程序清單中選取此觸發程序：[要求 - 收到 HTTP 要求時]

   ![新增 HTTP 要求觸發程序](./media/logic-apps-using-sap-connector/add-trigger.png)

3. 現在請儲存邏輯應用程式，即可產生邏輯應用程式的端點 URL。
在設計工具的工具列上，選擇 [儲存]。 

   端點 URL 現在會出現在您的觸發程序中，例如：

   ![產生端點的 URL](./media/logic-apps-using-sap-connector/generate-http-endpoint-url.png)

<a name="add-action"></a>

### <a name="add-sap-action"></a>新增 SAP 動作

在 Azure Logic Apps 中，[動作](../logic-apps/logic-apps-overview.md#logic-app-concepts)是工作流程中跟隨在觸發程序或另一個動作之後的步驟。 如果您尚未新增觸發程序至邏輯應用程式，並且想要遵循此範例進行，請[按照本節所述指示新增觸發程序](#add-trigger)。

1. 在邏輯應用程式設計工具的觸發程序之下，依序選擇 [新增步驟]  >  [新增動作]。

   ![新增動作](./media/logic-apps-using-sap-connector/add-action.png) 

2. 在搜尋方塊中，輸入「sap」作為篩選條件。 從 [動作] 清單中，選取此動作：**傳送訊息至 SAP**
  
   ![選取 SAP 傳送動作](media/logic-apps-using-sap-connector/select-sap-send-action.png)

   或者不要搜尋，而是選擇 [企業] 索引標籤，然後選取 SAP 動作。

   ![從 [企業] 索引標籤選取 SAP 傳送動作](media/logic-apps-using-sap-connector/select-sap-send-action-ent-tab.png)

3. 如果系統提示您輸入連線詳細資料，請立即建立 SAP 連線。 或者，若您的連線已存在，請繼續執行後續步驟，以設定 SAP 動作。 

   **建立內部部署 SAP 連線**

   1. 提供 SAP 伺服器的連線資訊。 
   針對 [資料閘道] 屬性，請選取您在 Azure 入口網站中建立用於安裝閘道的資料閘道。

      如果 [登入類型] 屬性設定為 [應用程式伺服器]，則需要這些通常會顯示為選擇性項目的屬性：

      ![建立 SAP 應用程式伺服器連線](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png) 

      如果 [登入類型] 屬性設定為 [群組]，則需要這些通常會顯示為選擇性項目的屬性： 

      ![建立 SAP 訊息伺服器連線](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png) 

   2. 完成之後，請選擇 [建立]。 
   
      Logic Apps 會設定並測試連線，以確定連線運作正常。

4. 立即尋找並從 SAP 伺服器選取動作。 

   1. 在 [SAP 動作] 方塊中選擇資料夾圖示。 
   從檔案清單中，尋找並選取您要使用的 SAP 訊息。 
   若要瀏覽清單，請使用箭號。

      此範例會選取類型為 [順序] 的 IDoc。 

      ![尋找並選取 IDoc 動作](./media/logic-apps-using-sap-connector/SAP-app-server-find-action.png)

      如果找不到您需要的動作，您可手動輸入路徑，例如：

      ![手動提供路徑給 IDoc 動作](./media/logic-apps-using-sap-connector/SAP-app-server-manually-enter-action.png)

      > [!TIP]
      > 請透過運算式編輯器提供 SAP 動作的值。 如此一來，您就可以對不同訊息類型使用相同的動作。

      如需有關 IDoc 作業的詳細資訊，請參閱 [IDOC 作業的訊息結構描述](https://docs.microsoft.com/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations)。

   2. 在 [輸入訊息] 方塊內按一下，動態內容清單隨即顯示。 
   在該清單中的 [收到 HTTP 要求時] 之下，選取 [內文] 欄位。 

      此步驟包含來自 HTTP 要求觸發程序的內文內容，且可傳送該輸出結果至您的 SAP 伺服器。

      ![選取 [內文] 欄位](./media/logic-apps-using-sap-connector/SAP-app-server-action-select-body.png)

      完成上述步驟後，SAP 動作看起來便如下列範例︰

      ![完整的 SAP 動作](./media/logic-apps-using-sap-connector/SAP-app-server-complete-action.png)

5. 儲存您的邏輯應用程式。 在設計工具的工具列上，選擇 [儲存]。

<a name="add-response"></a>

### <a name="add-http-response-action"></a>新增 HTTP 回應動作

現在請將回應動作新增至邏輯應用程式的工作流程，並納入 SAP 動作的輸出結果。 如此一來，邏輯應用程式便會從 SAP 伺服器將結果傳回至原始的要求者。 

1. 在邏輯應用程式設計工具的 SAP 動作之下，依序選擇 [新增步驟]  >  [新增動作]。

2. 在搜尋方塊中，輸入「回應」做為篩選條件。 從 [動作] 清單中，選取此動作：[要求 - 回應]

3. 在 [內文] 方塊內按一下，動態內容清單隨即顯示。 從該清單的 [傳送至 SAP] 之下，選取 [內文] 欄位。 

   ![完整的 SAP 動作](./media/logic-apps-using-sap-connector/select-sap-body-for-response-action.png)

4. 儲存您的邏輯應用程式。 

### <a name="test-your-logic-app"></a>測試應用程式邏輯

1. 如果邏輯應用程式尚未啟用，請在邏輯應用程式功能表上，選擇 [概觀]。 在工具列上，選擇 [啟用]。 

2. 在「邏輯應用程式設計工具」的工具列上，選擇 [執行]。 此步驟會手動啟動您的邏輯應用程式。

3. 將 HTTP POST 要求傳送至 HTTP 要求觸發程序中的 URL，以觸發邏輯應用程式，然後再透過要求加入訊息內容。 若要傳送內容，您可使用 [Postman](https://www.getpostman.com/apps) 之類的工具。 

   在本文中，要求會傳送 IDoc 檔案，其必須為 XML 格式並且包含您要使用的 SAP 動作的命名空間，例如： 

   ``` xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <Send xmlns="http://Microsoft.LobServices.Sap/2007/03/Idoc/2/ORDERS05//720/Send">
      <idocData>
         <...>
      </idocData>
   </Send>
   ```

4. 傳送 HTTP 要求後，請等候來自邏輯應用程式的回應。

   > [!NOTE]
   > 如果回應所需的所有步驟未在[要求逾時期限](./logic-apps-limits-and-config.md)內完成，邏輯應用程式可能會逾時。 如果發生此情況，要求可能就會遭到封鎖。 為協助您診斷問題，請了解如何[檢查和監控邏輯應用程式](../logic-apps/logic-apps-monitor-your-logic-apps.md)。

恭喜，您已經成功建立能與 SAP 伺服器通訊的邏輯應用程式。 您現在已為邏輯應用程式設定 SAP 連線，接下來可探索其他可用的 SAP 動作如：BAPI 和 RFC。

## <a name="receive-from-sap"></a>從 SAP 接收

此範例所使用的邏輯應用程式會在從 SAP 系統接收訊息時觸發。 

### <a name="add-sap-trigger"></a>新增 SAP 觸發程序

1. 在 Azure 入口網站中，建立空白的邏輯應用程式，以開啟邏輯應用程式設計工具。 

2. 在搜尋方塊中，輸入「sap」作為篩選條件。 從觸發程序清單中，選取此觸發程序：**從 SAP 收到訊息時**

   ![新增 SAP 觸發程序](./media/logic-apps-using-sap-connector/add-sap-trigger.png)

   或者，您也可以移至 [企業] 索引標籤，並選取觸發程序

   ![從索引標籤新增 SAP 觸發程序](./media/logic-apps-using-sap-connector/add-sap-trigger-ent-tab.png)

3. 如果系統提示您輸入連線詳細資料，請立即建立 SAP 連線。 或者，若您的連線已存在，請繼續執行後續步驟，以設定 SAP 動作。 

   **建立內部部署 SAP 連線**

   1. 提供 SAP 伺服器的連線資訊。 
   針對 [資料閘道] 屬性，請選取您在 Azure 入口網站中建立用於安裝閘道的資料閘道。

      如果 [登入類型] 屬性設定為 [應用程式伺服器]，則需要這些通常會顯示為選擇性項目的屬性：

      ![建立 SAP 應用程式伺服器連線](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png) 

      如果 [登入類型] 屬性設定為 [群組]，則需要這些通常會顯示為選擇性項目的屬性：

      ![建立 SAP 訊息伺服器連線](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)  

4. 根據 SAP 系統設定提供必要的參數。 

   您可以選擇性地提供一或多個 SAP 動作。 
   此動作清單會指定觸發程序透過資料閘道從 SAP 伺服器所接收的訊息。 
   空白清單會指定讓觸發程序接收所有訊息。 
   如果清單有多個訊息，觸發程序則只會接收清單所指定的訊息。 閘道會拒絕任何其他從 SAP 伺服器傳送的訊息。

   您可以從檔案選擇器選取 SAP 動作：

   ![選取 SAP 動作](media/logic-apps-using-sap-connector/select-SAP-action-trigger.png)  

   或者，您也可以手動指定動作：

   ![手動輸入 SAP 動作](media/logic-apps-using-sap-connector/manual-enter-SAP-action-trigger.png) 

   以下舉例說明當您設定觸發程序來接收多個訊息時，此動作的顯示方式。

   ![觸發程序範例](media/logic-apps-using-sap-connector/example-trigger.png)  

   如需 SAP 動作的詳細資訊，請參閱 [IDOC 作業的訊息結構描述](https://docs.microsoft.com/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations)

5. 現在儲存邏輯應用程式，以便可以開始從 SAP 系統接收訊息。
在設計工具的工具列上，選擇 [儲存]。 

邏輯應用程式現在已準備好從 SAP 系統接收訊息。 

> [!NOTE]
> SAP 觸發程序不是輪詢觸發程序，而是以 Webhook 為基礎的觸發程序。 訊息存在時才會從閘道呼叫此觸發程序，所以不必輪詢。 

### <a name="test-your-logic-app"></a>測試應用程式邏輯

1. 若要觸發邏輯應用程式，請從 SAP 系統傳送訊息。

2. 在邏輯應用程式的功能表上，選擇 [概觀]，並檢閱 [執行歷程記錄] 以了解邏輯應用程式是否有任何新的執行。 

3. 開啟最近的執行，其會在觸發程序的輸出區段中顯示從 SAP 系統傳送的訊息。

## <a name="generate-schemas-for-artifacts-in-sap"></a>在 SAP 中產生成品的結構描述

此範例使用您可透過 HTTP 要求觸發的邏輯應用程式。 SAP 動作會傳送要求到 SAP 系統，以產生指定中繼文件 (IDoc) 和 BAPI 的結構描述。 在回應中傳回的結構描述會使用 Azure Resource Manager 連接器上傳到企業整合帳戶。

### <a name="add-http-request-trigger"></a>新增 HTTP 要求觸發程序

1. 在 Azure 入口網站中，建立空白的邏輯應用程式，以開啟邏輯應用程式設計工具。 

2. 在搜尋方塊中，輸入「http 要求」做為篩選條件。 從觸發程序清單中選取此觸發程序：[要求 - 收到 HTTP 要求時]

   ![新增 HTTP 要求觸發程序](./media/logic-apps-using-sap-connector/add-trigger.png)

3. 現在請儲存邏輯應用程式，即可產生邏輯應用程式的端點 URL。
在設計工具的工具列上，選擇 [儲存]。 

   端點 URL 現在會出現在您的觸發程序中，例如：

   ![產生端點的 URL](./media/logic-apps-using-sap-connector/generate-http-endpoint-url.png)

### <a name="add-sap-action-to-generate-schemas"></a>新增 SAP 動作以產生結構描述

1. 在邏輯應用程式設計工具的觸發程序之下，依序選擇 [新增步驟]  >  [新增動作]。

   ![新增動作](./media/logic-apps-using-sap-connector/add-action.png) 

2. 在搜尋方塊中，輸入「sap」作為篩選條件。 從 [動作] 清單中，選取此動作：**產生結構描述**
  
   ![選取 SAP 傳送動作](media/logic-apps-using-sap-connector/select-sap-schema-generator-action.png)

   或者您也可以選擇 [企業] 索引標籤，然後選取 SAP 動作。

   ![從 [企業] 索引標籤選取 SAP 傳送動作](media/logic-apps-using-sap-connector/select-sap-schema-generator-ent-tab.png)

3. 如果系統提示您輸入連線詳細資料，請立即建立 SAP 連線。 或者，若您的連線已存在，請繼續執行後續步驟，以設定 SAP 動作。 

   **建立內部部署 SAP 連線**

   1. 提供 SAP 伺服器的連線資訊。 
   針對 [資料閘道] 屬性，請選取您在 Azure 入口網站中建立用於安裝閘道的資料閘道。

      如果 [登入類型] 屬性設定為 [應用程式伺服器]，則需要這些通常會顯示為選擇性項目的屬性：

      ![建立 SAP 應用程式伺服器連線](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png) 

      如果 [登入類型] 屬性設定為 [群組]，則需要這些通常會顯示為選擇性項目的屬性：
   
      ![建立 SAP 訊息伺服器連線](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png) 

   2. 完成之後，請選擇 [建立]。 Logic Apps 會設定並測試連線，以確定連線運作正常。

4. 針對您要產生結構描述的成品提供其路徑。

   您可以從檔案選擇器選取 SAP 動作：

   ![選取 SAP 動作](media/logic-apps-using-sap-connector/select-SAP-action-schema-generator.png)  

   或者，您也可以手動輸入動作：

   ![手動輸入 SAP 動作](media/logic-apps-using-sap-connector/manual-enter-SAP-action-schema-generator.png) 

   若要針對多個成品產生結構描述，請提供每個成品的 SAP 動作詳細資料，例如：

   ![選取 [新增項目]](media/logic-apps-using-sap-connector/schema-generator-array-pick.png) 

   ![顯示兩個項目](media/logic-apps-using-sap-connector/schema-generator-example.png) 

   如需 SAP 動作的詳細資訊，請參閱 [IDOC 作業的訊息結構描述](https://docs.microsoft.com/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations)。

5. 儲存您的邏輯應用程式。 在設計工具的工具列上，選擇 [儲存]。

### <a name="test-your-logic-app"></a>測試應用程式邏輯

1. 在設計工具的工具列上，選擇 [執行] 來觸發邏輯應用程式的執行。

2. 開啟 [執行]，並檢查 [產生結構描述] 動作的輸出。 

   輸出中會顯示針對所指定訊息清單產生的結構描述。

### <a name="upload-schemas-to-integration-account"></a>將結構描述上傳到企業整合帳戶

(選擇性) 您可以在存放庫 (例如，Blob、儲存體或企業整合帳戶) 下載或儲存產生的結構描述。 企業整合帳戶搭配其他 XML 動作可提供一流的體驗，因此這個範例會說明如何使用 Azure Resource Manager 連接器，將相同邏輯應用程式的結構描述上傳到企業整合帳戶。

1. 在邏輯應用程式設計工具的觸發程序之下，依序選擇 [新增步驟]  >  [新增動作]。 在搜尋方塊中，輸入「resource manager」作為篩選條件。 選取此動作：**建立或更新資源**

   ![選取 Azure Resource Manager 動作](media/logic-apps-using-sap-connector/select-arm-action.png) 

2. 輸入詳細資料，包括 Azure 訂用帳戶、Azure 資源群組以及企業整合帳戶。 至於其他欄位，請遵循下列範例。

   ![輸入 Azure Resource Manager 動作的詳細資料](media/logic-apps-using-sap-connector/arm-action.png)

   SAP 的**產生結構描述**動作會以集合的形式產生結構描述，讓設計工具可以在動作中自動新增 **For each** 迴圈。 
   以下舉例說明此動作的顯示方式：

   ![Azure Resource Manager 動作與 "for each" 迴圈](media/logic-apps-using-sap-connector/arm-action-foreach.png)  

   > [!NOTE]
   > 結構描述會使用 base64 編碼格式。 若要將結構描述上傳到企業整合帳戶，必須先使用 `base64ToString()` 函式將結構描述解碼。 以下舉例說明 `"properties"` 元素的程式碼：
   >
   > ```json
   > "properties": {
   >    "Content": "@base64ToString(items('For_each')?['Content'])",
   >    "ContentType": "application/xml",
   >    "SchemaType": "Xml"
   > }
   > ```

3. 儲存您的邏輯應用程式。 在設計工具的工具列上，選擇 [儲存]。

### <a name="test-your-logic-app"></a>測試應用程式邏輯

1. 在設計工具的工具列上，選擇 [執行] 來手動觸發邏輯應用程式。

2. 成功執行之後，請移至企業整合帳戶，並檢查產生的結構描述是否存在。

## <a name="known-issues-and-limitations"></a>已知問題與限制

以下是 SAP 連接器目前的已知問題和限制：

* SAP 觸發程序不支援從 SAP 接收批次 IDOC。 此動作可能會導致 SAP 系統與資料閘道之間的 RFC 連線失敗。

* SAP 觸發程序不支援資料閘道叢集。 在某些容錯移轉案例中，與 SAP 系統通訊的資料閘道節點可能會與作用中節點不同，而導致非預期的行為。 傳送案例可支援資料閘道叢集。

* 接收案例不支援傳回非 null 回應。 具有觸發程序和回應動作的邏輯應用程式會導致非預期的行為。 

* 單一的「傳送至 SAP」呼叫或訊息才能與 tRFC 搭配運作。 不支援商務應用程式開發介面 (BAPI) 認可模式，例如在相同的工作階段中進行多個 tRFC 呼叫。

* 「傳送到 SAP」和「產生結構描述」這兩個動作都不支援含有附件的 RFC。

* SAP 連接器目前不支援 SAP 路由器字串。 內部部署資料閘道必須和所要連線的 SAP 系統存在於相同 LAN 內。

* 後續針對內部部署資料閘道所進行的更新，可能會讓 DATS 和 TIMS SAP 欄位的不存在 (null)、空白、最小和最大值轉換有所變更。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 從邏輯應用程式[連線至內部部署系統](../logic-apps/logic-apps-gateway-connection.md)
* 了解驗證和轉換方式，以及[企業整合套件](../logic-apps/logic-apps-enterprise-integration-overview.md)中的其他訊息作業
* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)
