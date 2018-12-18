---
title: 適用於 Azure Logic Apps 的連接器 | Microsoft Docs
description: 使用 Azure Logic Apps 的連接器 (包括內建、受控、內部部署、整合帳戶和企業連接器) 將工作流程自動化
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.suite: integration
ms.topic: article
ms.date: 08/23/2018
ms.openlocfilehash: 6b31882ec3916e60ac7dc7b8117328176abef1b4
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818529"
---
# <a name="connectors-for-azure-logic-apps"></a>適用於 Azure Logic Apps 的連接器

當您使用 Azure Logic Apps 建立自動化工作流程時，連接器扮演不可或缺的部分。 在邏輯應用程式中使用連接器，即可擴充內部部署和雲端應用程式的功能，以便對您建立和已有的資料執行工作。 

儘管 Logic Apps 提供 [200 個以上的連接器](https://docs.microsoft.com/connectors)，但本文說明由數千個應用程式及數百萬次執行成功用於處理資料和資訊的熱門常用連接器。 連接器可以內建項目或受控連接器的形式提供。 

> [!NOTE]
> 如需連接器的完整清單和每個連接器的參考資訊，例如動作、任何觸發程序和限制，您可以在[連接器概觀](https://docs.microsoft.com/connectors)下找到完整清單。

* [**內建項目**](#built-ins)：這些內建動作和觸發程序可協助您建立可執行下列作業的邏輯應用程式：依照自訂排程執行、與其他端點通訊、接收和回應要求，以及呼叫 Azure 函式、Azure API Apps (Web 應用程式)，透過 Azure API 管理控管和發佈的自有 API，以及可接收要求的巢狀邏輯應用程式。 您也可以使用內建動作，協助您組織及控制邏輯應用程式的工作流程，以及處理資料。

* **受控連接器**：這些連接器會提供觸發程序和動作，以便存取其他服務和系統。 某些連接器會要求您先建立 Azure Logic Apps 所管理的連線。 受控連接器可歸納為以下群組：

  |   |   |
  |---|---|
  | [**受控 API 連接器**](#managed-api-connectors) | 建立可使用 Azure Blob 儲存體、Office 365、Dynamics、Power BI、OneDrive、Salesforce、SharePoint Online 等其他服務的邏輯應用程式。 | 
  | [**內部部署連接器**](#on-premises-connectors) | 在您安裝並設定[內部部署資料閘道][gateway-doc]之後，這些連接器可協助您的邏輯應用程式存取內部部署系統，例如 SQL Server、SharePoint Server、Oracle DB、檔案共用及其他系統。 | 
  | [**整合帳戶連接器**](#integration-account-connectors) | 當您建立並支付整合帳戶時，這些連接器會轉換和驗證 XML、進行一般檔案的編碼和解碼，以及透過 AS2、EDIFACT 和 X12 通訊協定處理企業對企業 (B2B) 訊息。 | 
  | [**企業連接器**](#enterprise-connectors) | 可供存取企業系統，例如 SAP 和 IBM MQ (額外付費)。 |
  ||| 

  例如，如果您使用 Microsoft BizTalk Server，則邏輯應用程式可以使用 [BizTalk Server 連接器](#on-premises-connectors)連線到 BizTalk Server 並且進行通訊。 
  您可以使用[整合帳戶連接器](#integration-account-connectors)，接著在邏輯應用程式中擴充或執行類似 BizTalk 的作業。 

> [!NOTE] 
> 如需連接器的完整清單和每個連接器的參考資訊，例如動作和任何觸發程序 (由 Swagger 描述定義) 以及任何限制，您可以在[連接器概觀](/connectors/)下找到完整清單。 如需定價資訊，請參閱 [Logic Apps 定價詳細資料](https://azure.microsoft.com/pricing/details/logic-apps/)和 [Logic Apps 定價模型](../logic-apps/logic-apps-pricing.md)。 

<a name="built-ins"></a>

## <a name="built-ins"></a>內建項目

Logic Apps 會提供內建觸發程序和動作，所以您可以建立以排程為基礎的工作流程，協助邏輯應用程式與其他應用程式和服務通訊、透過邏輯應用程式控制工作流程，以及管理或操作資料。 

|   |   |   |   | 
|---|---|---|---| 
| [![API 圖示][schedule-icon]<br/>**排程**][recurrence-doc] | - 透過 [週期] 觸發程序，依照指定的排程執行邏輯應用程式 (範圍從基本到複雜週期)。 <p>- 透過 [延遲] 動作，在指定的持續時間內暫停邏輯應用程式。 <p>- 透過 [延遲直到] 動作，將邏輯應用程式暫停直到指定的日期和時間為止。 | [![API 圖示][http-icon]<br/>**HTTP**][http-doc] | 利用適用於 HTTP、HTTP + Swagger 以及 HTTP + Webhook 的觸發程序和動作，透過 HTTP 與任何端點通訊。 | 
| [![API 圖示][http-request-icon]<br/>**要求**][http-request-doc] | - 透過 [要求] 觸發程序，讓邏輯應用程式得以從其他應用程式或服務呼叫、在 Event Grid 資源事件上觸發，或在 Azure 資訊安全中心警示的回應上觸發。 <p>- 透過 [回應] 動作，將回應傳送到應用程式或服務。 | [![API 圖示][batch-icon]<br/>**批次**][batch-doc] | - 透過 [批次訊息] 觸發程序，分批處理訊息。 <p>- 透過 [將訊息傳送至批次] 動作，呼叫具有現有批次觸發程序的邏輯應用程式。 | 
| [![API 圖示][azure-functions-icon]<br/>**Azure Functions**][azure-functions-doc] | 呼叫 Azure 函式，以從邏輯應用程式執行自訂程式碼片段 (C# 或 Node.js)。 | [![API 圖示][azure-api-management-icon]</br>**Azure API 管理**][azure-api-management-doc] | 呼叫透過 Azure API 管理控管和發佈的自有 API 所定義的觸發程序和動作。 | 
| [![API 圖示][azure-app-services-icon]<br/>**Azure App Service**][azure-app-services-doc] | 呼叫 API Apps 或 Web Apps (裝載於 Azure App Service)。 如果包含 Swagger，這些應用程式所定義的動作與觸發程序看起來像是任何其他第一級觸發程序和動作。 | [![API 圖示][azure-logic-apps-icon]<br/>**Azure<br/>Logic Apps**][nested-logic-app-doc] | 呼叫透過 [要求] 觸發程序啟動的其他邏輯應用程式。 | 
||||| 

### <a name="control-workflow"></a>控制工作流程

以下的內建動作可供建構及控制邏輯應用程式工作流程中的動作：

|   |   |   |   | 
|---|---|---|---| 
| [![內建圖示][condition-icon]<br/>**條件**][condition-doc] | 根據條件為 true 或 false，評估條件並執行不同的動作。 | [![內建圖示][for-each-icon]</br>**For each**][for-each-doc] | 對陣列中的每個項目執行相同的動作。 | 
| [![內建圖示][scope-icon]<br/>**範圍**][scope-doc] | 將動作分成數個「範圍」，這些範圍會在範圍中的動作執行完成後取得自己的狀態。 | [![內建圖示][switch-icon]</br>**Switch**][switch-doc] | 將動作分成數個「案例」，這些案例會獲派唯一值 (預設案例除外)。 僅執行其指派值符合運算式、物件或語彙基元結果的案例。 如果沒有相符項目存在，請執行預設案例。 | 
| [![內建圖示][terminate-icon]<br/>**終止**][terminate-doc] | 停止正在執行的邏輯應用程式工作流程。 | [![內建圖示][until-icon]<br/>**Until**][until-doc] | 重複動作，直到符合指定的條件或某些狀態已改變為止。 | 
||||| 

### <a name="manage-or-manipulate-data"></a>管理或操作資料

以下的內建動作可供處理資料輸出及其格式：  

|   |   | 
|---|---| 
| ![內建圖示][data-operations-icon]<br/>**資料作業** | 執行資料相關作業： <p>- **撰寫**：從多個輸入與各種類型建立單一輸出。 <br>- **建立 CSV 資料表**：從 JSON 物件陣列建立以逗號分隔 (CSV) 的資料表。 <br>- **建立 HTML 資料表**：從 JSON 物件陣列建立 HTML 資料表。 <br>- **篩選陣列**：從符合您準則的另一個陣列建立陣列。 <br>- **加入**：從陣列中的所有項目建立字串，並使用指定的分隔符號來分隔這些項目。 <br>- **剖析 JSON**：從 JSON 內容中的屬性及其值，建立使用者易記的語彙基元，以便在您的工作流程中使用這些屬性。 <br>- **選取**：轉換另一個陣列中的項目或值，並將這些項目對應至指定的屬性，以建立 JSON 物件陣列。 | 
| ![內建圖示][date-time-icon]<br/>**日期時間** | 執行時間戳記相關作業： <p>- **新增至時間**：將指定的單位數新增至時間戳記。 <br>- **轉換時區**：將時間戳記從來源時區轉換為目標時區。 <br>- **目前時間**：傳回目前的時間戳記作為字串。 <br>- **取得未來時間**：傳回目前時間戳記加上指定時間單位的結果。 <br>- **取得過去時間**：傳回目前時間戳記減去指定時間單位的結果。 <br>- **從時間減去**：從時間戳記減去時間單位數字。 |
| [![內建圖示][variables-icon]<br/>**變數**][variables-doc] | 執行變數相關作業： <p>- **附加到陣列變數**：插入一個值，作為變數所儲存陣列中的最後一個項目。 <br>- **附加到字串變數**：插入一個值，作為變數所儲存字串中的最後一個字元。 <br>- **遞減變數**：依常數值遞減變數。 <br>- **遞增變數**：依常數值遞增變數。 <br>- **初始化變數**：建立變數，並宣告其資料類型和初始值。 <br>- **設定變數**：將不同的值指派給現有的變數。 |
|  |  | 

<a name="managed-api-connectors"></a>

## <a name="managed-api-connectors"></a>受控 API 連接器

以下是比較熱門的連接器，可供自動執行下列服務或系統的工作、程序和工作流程：

|   |   |   |   | 
|---|---|---|---| 
| [![API 圖示][azure-service-bus-icon]<br/>**Azure 服務匯流排**][azure-service-bus-doc] | 利用 Logic Apps 中最常用的連接器，管理非同步訊息、工作階段和主題訂用帳戶。 | [![API 圖示][sql-server-icon]<br/>**SQL Server**][sql-server-doc] | 連線到內部部署 SQL Server 或雲端中的 Azure SQL Database，以便管理記錄、執行預存程序，或執行查詢。 | 
| [![API 圖示][office-365-outlook-icon]<br/>**Office 365<br/>Outlook**][office-365-outlook-doc] | 連線到 Office 365 電子郵件帳戶，以便建立和管理電子郵件、工作、行事曆事件和會議、連絡人、要求等等。 | [![API 圖示][azure-blob-storage-icon]<br/>**Azure Blob<br/>儲存體**][azure-blob-storage-doc] | 連線到儲存體帳戶，以便建立和管理 blob 內容。 | 
| [![API 圖示][sftp-icon]<br/>**SFTP**][sftp-doc] | 連線到可從網際網路存取的 SFTP 伺服器，以便使用您的檔案和資料夾。 | [![API 圖示][sharepoint-online-icon]<br/>**SharePoint<br/>Online**][sharepoint-online-doc] | 連線到 SharePoint Online，以便管理檔案、附件、資料夾等等。 | 
| [![API 圖示][dynamics-365-icon]<br/>**Dynamics 365<br/>CRM Online**][dynamics-365-doc] | 連線到 Dynamics 365 帳戶，以便建立和管理記錄、項目等等。 | [![API 圖示][ftp-icon]<br/>**FTP**][ftp-doc] | 連線到可從網際網路存取的 FTP 伺服器，以便使用您的檔案和資料夾。 | 
| [![API 圖示][salesforce-icon]<br/>**Salesforce**][salesforce-doc] | 連線到 Salesforce 帳戶，以便建立和管理記錄、作業、物件等項目。 | [![API 圖示][twitter-icon]<br/>**Twitter**][twitter-doc] | 連線到 Twitter 帳戶，以便管理推文、追蹤者、您的時間軸等等。 將您的推文儲存到 SQL、Excel 或 SharePoint。 | 
| [![API 圖示][azure-event-hubs-icon]<br/>**Azure 事件中樞**][azure-event-hubs-doc] | 透過事件中樞取用和發佈事件。 例如，透過事件中樞從邏輯應用程式取得輸出，然後將輸出傳送給即時分析提供者。 | [![API 圖示][azure-event-grid-icon]<br/>**Azure Event**</br>**Grid**][azure-event-grid-doc] | 當 Azure 資源或第三方資源變更時，監視 Event Grid 所發佈的事件。 | 
||||| 

<a name="on-premises-connectors"></a>

## <a name="on-premises-connectors"></a>內部部署連接器 

以下是一些常用的連接器，可供存取內部部署系統中的資料和資源。 您必須先[下載、安裝及設定內部部署資料閘道][gateway-doc]，才可建立內部部署系統連線。 此閘道會提供安全的通訊通道，而不需設定必要的網路基礎結構。 

|   |   |   |   |   | 
|---|---|---|---|---| 
| ![API 圖示][biztalk-server-icon]<br/>**BizTalk**</br> **伺服器** | [![API 圖示][file-system-icon]<br/>**檔案</br>系統**][file-system-doc] | [![API 圖示][ibm-db2-icon]<br/>**IBM DB2**][ibm-db2-doc] | [![API 圖示][ibm-informix-icon]<br/>**IBM**</br> **Informix**][ibm-informix-doc] | ![API 圖示][mysql-icon]<br/>**MySQL** | 
| [![API 圖示][oracle-db-icon]<br/>**Oracle DB**][oracle-db-doc] | ![API 圖示][postgre-sql-icon]<br/>**PostgreSQL** | [![API 圖示][sharepoint-server-icon]<br/>**SharePoint</br> Server**][sharepoint-server-doc] | [![API 圖示][sql-server-icon]<br/>**SQL</br> Server**][sql-server-doc] | ![API 圖示][teradata-icon]<br/>**Teradata** | 
||||| 

<a name="integration-account-connectors"></a>

## <a name="integration-account-connectors"></a>整合帳戶連接器 

當您建立並支付[整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) (可透過 Azure 中的企業整合套件 (EIP) 取得) 時，以下的連接器可供利用邏輯應用程式建置企業對企業 (B2B) 解決方案。 使用此帳戶，您可以建立並儲存一些 B2B 構件，例如交易夥伴、合約、地圖、結構描述、憑證等等。 若要使用這些構件，請建立您的邏輯應用程式與整合帳戶的關聯。 如果您目前使用 BizTalk Server，可能已經熟悉這些連接器。

|   |   |   |   | 
|---|---|---|---| 
| [![API 圖示][as2-icon]<br/>**AS2</br> 解碼**][as2-decode-doc] | [![API 圖示][as2-icon]<br/>**AS2</br>編碼**][as2-encode-doc] | [![API 圖示][edifact-icon]<br/>**EDIFACT</br>解碼**][edifact-decode-doc] | [![API 圖示][edifact-icon]<br/>**EDIFACT</br>編碼**][edifact-encode-doc] | 
| [![API 圖示][flat-file-decode-icon]<br/>**一般檔案</br>解碼**][flat-file-decode-doc] | [![API 圖示][flat-file-encode-icon]<br/>**一般檔案</br>編碼**][flat-file-encode-doc] | [![API 圖示][integration-account-icon]<br/>**整合<br/>帳戶**][integration-account-doc] | [![API 解碼][liquid-icon]<br/>**Liquid**</br>**轉換**][json-liquid-transform-doc] | 
| [![API 圖示][x12-icon]<br/>**X12</br>解碼**][x12-decode-doc] | [![API 圖示][x12-icon]<br/>**X12</br>編碼**][x12-encode-doc] | [![API 圖示][xml-transform-icon]<br/>**XML**</br>**轉換**][xml-transform-doc] | [![API 圖示][xml-validate-icon]<br/>**XML<br/>驗證**][xml-validate-doc] |  
||||| 

<a name="enterprise-connectors"></a>

## <a name="enterprise-connectors"></a>企業連接器

您的邏輯應用程式可以存取企業系統，例如 SAP 和 IBM MQ：

|   |   | 
|---|---| 
| [![API 圖示][ibm-mq-icon]<br/>**IBM MQ**][ibm-mq-doc] | [![API 圖示][sap-icon]<br/>**SAP**][sap-connector-doc] |
||| 

## <a name="more-about-triggers-and-actions"></a>觸發程序動作的詳細資訊

有些連接器會提供「觸發程序」，以在發生特定事件時通知您的邏輯應用程式。 所以發生這些事件時，觸發程序會建立並執行邏輯應用程式的執行個體。 例如，FTP 連接器提供「新增或修改檔案時」觸發程序，該觸發程序會在檔案更新時啟動您的邏輯應用程式。 

Logic Apps 會提供下列幾種觸發程序：  

* 輪詢觸發程序：這些觸發程序會以指定的頻率輪詢您的服務，並檢查是否有新資料。 

  有新資料可用時，就會建立邏輯應用程式的新執行個體，並以當作輸入傳入的資料執行。 

* 推送觸發程序：這些觸發程序會接聽端點上的新資料或要發生的事件，以建立和執行邏輯應用程式的新執行個體。

* 週期性觸發程序︰此觸發程序會依照指定的排程建立和執行邏輯應用程式的執行個體。

連接器也會提供一些「動作」，以在邏輯應用程式的工作流程中執行工作。 比方說，邏輯應用程式可以讀取資料，並且在邏輯應用程式的後續步驟中使用此資料。 更具體來說，邏輯應用程式可以從 SQL 資料庫中尋找客戶資料，稍後在邏輯應用程式的工作流程中處理此資料。 

如需觸發程序和動作的詳細資訊，請參閱[連接器概觀](connectors-overview.md)。 

## <a name="custom-apis-and-connectors"></a>自訂 API 和連接器 

若要呼叫可執行自訂程式碼或不可作為連接器的 API，您可[建立自訂 API Apps](../logic-apps/logic-apps-create-api-app.md)，以擴充 Logic Apps 平台。 您也可以針對「任何」以 REST 或 SOAP 為基礎的 API [建立自訂連接器](../logic-apps/custom-connector-overview.md)，以便將這些 API 提供給 Azure 訂用帳戶中的任何邏輯應用程式使用。
若要對任何人公開自訂 API Apps 或連接器以使用於 Azure，您可以[提交連接器進行 Microsoft 認證](../logic-apps/custom-connector-submit-certification.md)。

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。

* 若要提出或票選 Azure Logic Apps 和連接器的想法，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

* 這些文件是否遺漏您認為重要的文章或詳細資訊？ 如果是，您可以將遺漏的資訊新增至現有文章或撰寫您自己的文章。 文件是開放原始碼並存放於 GitHub。 從 Azure 文件的 [GitHub 存放庫](https://github.com/Microsoft/azure-docs)著手。 

## <a name="next-steps"></a>後續步驟

* 尋找[連接器的完整清單](https://docs.microsoft.com/connectors)
* [建立第一個邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)
* [建立邏輯應用程式的自訂連接器](https://docs.microsoft.com/connectors/custom-connectors/)
* [建立邏輯應用程式的自訂 API](../logic-apps/logic-apps-create-api-app.md)

<!--Misc doc links-->
[gateway-doc]: ../logic-apps/logic-apps-gateway-connection.md "使用內部部署資料閘道從邏輯應用程式連線至內部部署資料來源"

<!--Built-ins doc links-->
[azure-functions-doc]: ../logic-apps/logic-apps-azure-functions.md "整合邏輯應用程式與 Azure Functions"
[azure-api-management-doc]: ../api-management/get-started-create-service-instance.md "建立 Azure API 管理服務執行個體以便管理和發佈您的 API"
[azure-app-services-doc]: ../logic-apps/logic-apps-custom-hosted-api.md "整合邏輯應用程式與 App Service API Apps"
[azure-service-bus-doc]: ./connectors-create-api-servicebus.md "從「服務匯流排佇列和主題」傳送訊息，並接收來自「服務匯流排佇列和訂用帳戶」的訊息"
[batch-doc]: ../logic-apps/logic-apps-batch-process-send-receive-messages.md "以群組或批次方式處理訊息"
[condition-doc]: ../logic-apps/logic-apps-control-flow-conditional-statement.md "根據條件為 true 或 false，評估條件並執行不同的動作"
[delay-doc]: ./connectors-native-delay.md "執行延遲的動作"
[for-each-doc]: ../logic-apps/logic-apps-control-flow-loops.md#foreach-loop "對陣列中的每個項目執行相同的動作"
[http-doc]: ./connectors-native-http.md "使用 HTTP 連接器進行 HTTP 呼叫"
[http-request-doc]: ./connectors-native-reqres.md "將 HTTP 要求和回應的動作新增至邏輯應用程式"
[http-response-doc]: ./connectors-native-reqres.md "將 HTTP 要求和回應的動作新增至邏輯應用程式"
[http-swagger-doc]: ./connectors-native-http-swagger.md "使用 HTTP + Swagger 連接器進行 HTTP 呼叫"
[http-webook-doc]: ./connectors-native-webhook.md "將 HTTP Webhook 動作和觸發程序新增至邏輯應用程式"
[nested-logic-app-doc]: ../logic-apps/logic-apps-http-endpoint.md "整合邏輯應用程式與巢狀工作流程"
[query-doc]: ./connectors-native-query.md "透過查詢動作選取和篩選陣列"
[recurrence-doc]:  ./connectors-native-recurrence.md "觸發邏輯應用程式的重複執行動作"
[scope-doc]: ../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md "將動作組織成群組，這些群組會在群組中的動作執行完成後取得自己的狀態" 
[switch-doc]: ../logic-apps/logic-apps-control-flow-switch-statement.md "將動作組織成案例，這些案例會獲派唯一值。僅執行其值符合運算式、物件或語彙基元結果的案例。如果沒有相符項目存在，請執行預設案例"
[terminate-doc]: ../logic-apps/logic-apps-workflow-actions-triggers.md#terminate-action "停止或取消邏輯應用程式正在執行中的工作流程"
[until-doc]: ../logic-apps/logic-apps-control-flow-loops.md#until-loop "重複動作，直到符合指定的條件或某些狀態已改變為止"
[variables-doc]: ../logic-apps/logic-apps-create-variables-store-values.md "執行變數相關作業，例如初始化、設定、遞增、遞減及附加至字串或陣列變數"

<!--Managed API doc links-->
[azure-blob-storage-doc]: ./connectors-create-api-azureblobstorage.md "使用 Azure Blob 儲存體連接器管理 Blob 容器中的檔案"
[azure-event-grid-doc]: ../event-grid/monitor-virtual-machine-changes-event-grid-logic-app.md " 當 Azure 資源或第三方資源變更時，監視 Event Grid 所發佈的事件"
[azure-event-hubs-doc]: ./connectors-create-api-azure-event-hubs.md "連線到 Azure 事件中樞。在邏輯應用程式與事件中樞之間接收和傳送事件"
[box-doc]: ./connectors-create-api-box.md "連線至 Box。上傳、取得、刪除、列出您的檔案等等"
[dropbox-doc]: ./connectors-create-api-dropbox.md "連線至 Dropbox。上傳、取得、刪除、列出您的檔案等等"
[dynamics-365-doc]: ./connectors-create-api-crmonline.md "連線至 Dynamics CRM Online，以便使用 CRM Online 資料"
[facebook-doc]: ./connectors-create-api-facebook.md "連線至 Facebook。張貼在動態時報上、取得頁面摘要等等"
[file-system-doc]: ../logic-apps/logic-apps-using-file-connector.md "連線至內部部署檔案系統"
[ftp-doc]: ./connectors-create-api-ftp.md "連線至 FTP / FTPS 伺服器以便執行 FTP 工作，像是上傳、取得、刪除檔案等等"
[github-doc]: ./connectors-create-api-github.md "連線至 GitHub 並追蹤問題"
[google-calendar-doc]: ./connectors-create-api-googlecalendar.md "連接到 Google Calendar 並可管理行事曆。"
[google-drive-doc]: ./connectors-create-api-googledrive.md "連接至 GoogleDrive 以便使用資料"
[google-sheets-doc]: ./connectors-create-api-googlesheet.md "連線至 Google 試算表以便修改工作表"
[google-tasks-doc]: ./connectors-create-api-googletasks.md "連線至 Google Tasks 以便管理工作"
[ibm-db2-doc]: ./connectors-create-api-db2.md "連線至雲端或內部部署中的 IBM DB2。更新資料列、取得資料表等等"
[ibm-informix-doc]: ./connectors-create-api-informix.md "連線至雲端或內部部署中的 Informix。讀取資料列、列出資料表等等"
[ibm-mq-doc]: ./connectors-create-api-mq.md "連線到 IBM MQ 內部部署或 Azure 以傳送及接收訊息"
[instagram-doc]: ./connectors-create-api-instagram.md "連線至 Instagram。觸發事件或對其採取行動"
[mailchimp-doc]: ./connectors-create-api-mailchimp.md "連線至您的 MailChimp 帳戶。管理及自動處理郵件"
[mandrill-doc]: ./connectors-create-api-mandrill.md "連線至 Mandrill 進行通訊"
[microsoft-translator-doc]: ./connectors-create-api-microsofttranslator.md "連線至 Microsoft Translator。翻譯文字、偵測語言等等" 
[office-365-outlook-doc]: ./connectors-create-api-office365-outlook.md "連線至 Office 365 帳戶。傳送和接收電子郵件、管理行事曆和連絡人等等"
[office-365-users-doc]: ./connectors-create-api-office365-users.md 
[office-365-video-doc]: ./connectors-create-api-office365-video.md "取得影片資訊、影片清單和頻道，以及 Office 365 影片的播放 URL"
[onedrive-doc]: ./connectors-create-api-onedrive.md "連線至您的個人 Microsoft OneDrive。上傳、刪除、列出檔案等等"
[onedrive-for-business-doc]: ./connectors-create-api-onedriveforbusiness.md "連線至公司 Microsoft OneDrive。上傳、刪除、列出您的檔案等等"
[oracle-db-doc]: ./connectors-create-api-oracledatabase.md "連線至 Oracle 資料庫來新增、插入、刪除資料列等等"
[outlook.com-doc]: ./connectors-create-api-outlook.md "連線至 Outlook 信箱。管理電子郵件、行事曆、連絡人等等"
[project-online-doc]: ./connectors-create-api-projectonline.md "連線至 Microsoft Project Online。管理專案、工作、資源等等"
[rss-doc]: ./connectors-create-api-rss.md "發佈和擷取摘要項目，在新項目發佈到 RSS 時觸發作業。"
[salesforce-doc]: ./connectors-create-api-salesforce.md "連線至 Salesforce 帳戶。管理帳戶、潛在客戶、機會等等"
[sap-connector-doc]: ../logic-apps/logic-apps-using-sap-connector.md "連線至內部部署 SAP 系統"
[sendgrid-doc]: ./connectors-create-api-sendgrid.md "連線至 SendGrid。傳送電子郵件及管理收件者清單"
[sftp-doc]: ./connectors-create-api-sftp.md "連線至 SFTP 帳戶。上傳、取得、刪除檔案等等"
[sharepoint-server-doc]: ./connectors-create-api-sharepointserver.md "連線至 SharePoint 內部部署伺服器。管理文件、列出項目等等"
[sharepoint-online-doc]: ./connectors-create-api-sharepointonline.md "連線至 SharePoint Online。管理文件、列出項目等等"
[slack-doc]: ./connectors-create-api-slack.md "連線至 Slack，並將訊息張貼至 Slack 通道"
[smtp-doc]: ./connectors-create-api-smtp.md "連線至 SMTP 伺服器，以及傳送帶有附件的電子郵件"
[sparkpost-doc]: ./connectors-create-api-sparkpost.md "連線至 SparkPost 進行通訊"
[sql-server-doc]: ./connectors-create-api-sqlazure.md "連線至 Azure SQL Database 或 SQL Server。建立、更新、取得和刪除 SQL Database 資料表中的項目。"
[trello-doc]: ./connectors-create-api-trello.md "連線至 Trello。管理您的專案並組織任何項目與任何人"
[twilio-doc]: ./connectors-create-api-twilio.md "連線至 Twilio。傳送和取得訊息、取得可用的號碼、管理撥入的電話號碼等等"
[twitter-doc]: ./connectors-create-api-twitter.md "連線至 Twitter。取得時間軸、張貼推文等等"
[wunderlist-doc]: ./connectors-create-api-wunderlist.md "連線至 Wunderlist。管理您的工作和待辦事項清單、讓您的生活保持同步等等"
[yammer-doc]: ./connectors-create-api-yammer.md "連線至 Yammer。張貼訊息、取得新訊息等等"
[youtube-doc]: ./connectors-create-api-youtube.md "連線至 YouTube。管理您的影片和頻道"

<!--Enterprise Intregation Pack doc links-->
[as2-doc]: ../logic-apps/logic-apps-enterprise-integration-as2.md "深入了解企業整合 AS2。"
[as2-decode-doc]: ../logic-apps/logic-apps-enterprise-integration-as2-decode.md "深入了解企業整合 AS2 解碼"
[as2-encode-doc]:../logic-apps/logic-apps-enterprise-integration-as2-encode.md "深入了解企業整合 AS2 編碼"
[edifact-decode-doc]: ../logic-apps/logic-apps-enterprise-integration-EDIFACT-decode.md "深入了解企業整合 EDIFACT 解碼"
[edifact-encode-doc]: ../logic-apps/logic-apps-enterprise-integration-EDIFACT-encode.md "深入了解企業整合 EDIFACT 編碼"
[flat-file-decode-doc]:../logic-apps/logic-apps-enterprise-integration-flatfile.md "深入了解企業整合一般檔案。"
[flat-file-encode-doc]:../logic-apps/logic-apps-enterprise-integration-flatfile.md "深入了解企業整合一般檔案。"
[integration-account-doc]: ../logic-apps/logic-apps-enterprise-integration-metadata.md "查閱整合帳戶中的結構描述、對應、合作夥伴和其他資訊"
[json-liquid-transform-doc]: ../logic-apps/logic-apps-enterprise-integration-liquid-transform.md "深入了解使用 Liquid 進行 JSON 轉換"
[x12-doc]: ../logic-apps/logic-apps-enterprise-integration-x12.md "深入了解企業整合 X12"
[x12-decode-doc]: ../logic-apps/logic-apps-enterprise-integration-X12-decode.md "深入了解企業整合 X12 解碼"
[x12-encode-doc]: ../logic-apps/logic-apps-enterprise-integration-X12-encode.md "深入了解企業整合 X12 編碼"
[xml-transform-doc]: ../logic-apps/logic-apps-enterprise-integration-transform.md "深入了解企業整合轉換。"
[xml-validate-doc]: ../logic-apps/logic-apps-enterprise-integration-xml-validation.md "深入了解企業整合 XML 驗證。"

<!-- Built-ins icons -->
[azure-api-management-icon]: ./media/apis-list/azure-api-management.png
[azure-app-services-icon]: ./media/apis-list/azure-app-services.png
[azure-functions-icon]: ./media/apis-list/azure-functions.png
[azure-logic-apps-icon]: ./media/apis-list/azure-logic-apps.png
[batch-icon]: ./media/apis-list/batch.png
[condition-icon]: ./media/apis-list/condition.png
[data-operations-icon]: ./media/apis-list/data-operations.png
[date-time-icon]: ./media/apis-list/date-time.png
[for-each-icon]: ./media/apis-list/for-each-loop.png
[http-icon]: ./media/apis-list/http.png
[http-request-icon]: ./media/apis-list/request.png
[http-response-icon]: ./media/apis-list/response.png
[http-swagger-icon]: ./media/apis-list/http-swagger.png
[http-webhook-icon]: ./media/apis-list/http-webhook.png
[schedule-icon]: ./media/apis-list/recurrence.png
[scope-icon]: ./media/apis-list/scope.png
[switch-icon]: ./media/apis-list/switch.png
[terminate-icon]: ./media/apis-list/terminate.png
[until-icon]: ./media/apis-list/until.png
[variables-icon]: ./media/apis-list/variables.png

<!--Managed API icons-->
[appfigures-icon]: ./media/apis-list/appfigures.png
[asana-icon]: ./media/apis-list/asana.png
[azure-automation-icon]: ./media/apis-list/azure-automation.png
[azure-blob-storage-icon]: ./media/apis-list/azure-blob-storage.png
[azure-cognitive-services-text-analytics-icon]: ./media/apis-list/azure-cognitive-services-text-analytics.png
[azure-data-lake-icon]: ./media/apis-list/azure-data-lake.png
[azure-document-db-icon]: ./media/apis-list/azure-document-db.png
[azure-event-grid-icon]: ./media/apis-list/azure-event-grid.png
[azure-event-grid-publish-icon]: ./media/apis-list/azure-event-grid-publish.png
[azure-event-hubs-icon]: ./media/apis-list/azure-event-hubs.png
[azure-ml-icon]: ./media/apis-list/azure-ml.png
[azure-queues-icon]: ./media/apis-list/azure-queues.png
[azure-resource-manager-icon]: ./media/apis-list/azure-resource-manager.png
[azure-service-bus-icon]: ./media/apis-list/azure-service-bus.png
[basecamp-3-icon]: ./media/apis-list/basecamp.png
[bitbucket-icon]: ./media/apis-list/bitbucket.png
[bitly-icon]: ./media/apis-list/bitly.png
[biztalk-server-icon]: ./media/apis-list/biztalk.png
[blogger-icon]: ./media/apis-list/blogger.png
[box-icon]: ./media/apis-list/box.png
[campfire-icon]: ./media/apis-list/campfire.png
[common-data-service-icon]: ./media/apis-list/common-data-service.png
[dropbox-icon]: ./media/apis-list/dropbox.png
[dynamics-365-icon]: ./media/apis-list/dynamics-crm-online.png
[dynamics-365-financials-icon]: ./media/apis-list/dynamics-365-financials.png
[dynamics-365-operations-icon]: ./media/apis-list/dynamics-365-operations.png
[easy-redmine-icon]: ./media/apis-list/easyredmine.png
[facebook-icon]: ./media/apis-list/facebook.png
[file-system-icon]: ./media/apis-list/file-system.png
[ftp-icon]: ./media/apis-list/ftp.png
[github-icon]: ./media/apis-list/github.png
[google-calendar-icon]: ./media/apis-list/google-calendar.png
[google-drive-icon]: ./media/apis-list/google-drive.png
[google-sheets-icon]: ./media/apis-list/google-sheet.png
[google-tasks-icon]: ./media/apis-list/google-tasks.png
[hipchat-icon]: ./media/apis-list/hipchat.png
[ibm-db2-icon]: ./media/apis-list/ibm-db2.png
[ibm-informix-icon]: ./media/apis-list/ibm-informix.png
[ibm-mq-icon]: ./media/apis-list/ibm-mq.png
[insightly-icon]: ./media/apis-list/insightly.png
[instagram-icon]: ./media/apis-list/instagram.png
[instapaper-icon]: ./media/apis-list/instapaper.png
[jira-icon]: ./media/apis-list/jira.png
[mailchimp-icon]: ./media/apis-list/mailchimp.png
[mandrill-icon]: ./media/apis-list/mandrill.png
[microsoft-translator-icon]: ./media/apis-list/microsoft-translator.png
[mysql-icon]: ./media/apis-list/mysql.png
[office-365-outlook-icon]: ./media/apis-list/office-365.png
[office-365-users-icon]: ./media/apis-list/office-365-users.png
[office-365-video-icon]: ./media/apis-list/office-365-video.png
[onedrive-icon]: ./media/apis-list/onedrive.png
[onedrive-for-business-icon]: ./media/apis-list/onedrive-business.png
[oracle-db-icon]: ./media/apis-list/oracle-db.png
[outlook.com-icon]: ./media/apis-list/outlook.png
[pagerduty-icon]: ./media/apis-list/pagerduty.png
[pinterest-icon]: ./media/apis-list/pinterest.png
[postgre-sql-icon]: ./media/apis-list/postgre-sql.png
[project-online-icon]: ./media/apis-list/projecton-line.png
[redmine-icon]: ./media/apis-list/redmine.png
[rss-icon]: ./media/apis-list/rss.png
[salesforce-icon]: ./media/apis-list/salesforce.png
[sap-icon]: ./media/apis-list/sap.png
[send-grid-icon]: ./media/apis-list/sendgrid.png
[sftp-icon]: ./media/apis-list/sftp.png
[sharepoint-online-icon]: ./media/apis-list/sharepoint-online.png
[sharepoint-server-icon]: ./media/apis-list/sharepoint-server.png
[slack-icon]: ./media/apis-list/slack.png
[smartsheet-icon]: ./media/apis-list/smartsheet.png
[smtp-icon]: ./media/apis-list/smtp.png
[sparkpost-icon]: ./media/apis-list/sparkpost.png
[sql-server-icon]: ./media/apis-list/sql.png
[teradata-icon]: ./media/apis-list/teradata.png
[todoist-icon]: ./media/apis-list/todoist.png
[trello-icon]: ./media/apis-list/trello.png
[twilio-icon]: ./media/apis-list/twilio.png
[twitter-icon]: ./media/apis-list/twitter.png
[vimeo-icon]: ./media/apis-list/vimeo.png
[visual-studio-team-services-icon]: ./media/apis-list/visual-studio-team-services.png
[wordpress-icon]: ./media/apis-list/wordpress.png
[wunderlist-icon]: ./media/apis-list/wunderlist.png
[yammer-icon]: ./media/apis-list/yammer.png
[youtube-icon]: ./media/apis-list/youtube.png

<!-- Enterprise Integration Pack icons -->
[as2-icon]: ./media/apis-list/as2.png
[edifact-icon]: ./media/apis-list/edifact.png
[flat-file-encode-icon]: ./media/apis-list/flat-file-encoding.png
[flat-file-decode-icon]: ./media/apis-list/flat-file-decoding.png
[integration-account-icon]: ./media/apis-list/integration-account.png
[liquid-icon]: ./media/apis-list/liquid-transform.png
[x12-icon]: ./media/apis-list/x12.png
[xml-validate-icon]: ./media/apis-list/xml-validation.png
[xml-transform-icon]: ./media/apis-list/xsl-transform.png