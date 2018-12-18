---
title: 連線到 IBM DB2 - Azure Logic Apps | Microsoft Docs
description: 使用 IBM DB2 REST API 和 Azure Logic Apps 管理資源
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
ms.reviewer: plarsen, LADocs
ms.suite: integration
ms.topic: article
ms.date: 08/23/2018
tags: connectors
ms.openlocfilehash: 354e67183a36f511811d74a0685dea2e23d6c0e2
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818870"
---
# <a name="manage-ibm-db2-resources-with-azure-logic-apps"></a>使用 Azure Logic Apps 管理 IBM DB2 資源

使用 Azure Logic Apps 和 IBM DB2 連接器，您即可根據儲存在 DB2 資料庫中的資源，建立自動化的工作和工作流程。 工作流程可以連線到資料庫中的資源、讀取和列出資料庫資料表、新增資料列、變更資料列、刪除資料列，不一而足。 您可以將動作納入邏輯應用程式中，以取得資料庫回應，並提供輸出給其他動作使用。 

本文說明如何建立邏輯應用程式來執行各種資料庫作業。 如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)。

## <a name="supported-platforms-and-versions"></a>支援的平台和版本

DB2 連接器內含 Microsoft 用戶端，以便透過 TCP/IP 網路來與遠端 DB2 伺服器通訊。 您可以使用此連接器來存取雲端資料庫 (例如在 Azure 虛擬化中執行的 IBM Bluemix dashDB 或 IBM DB2 for Windows)。 您也可以在[安裝和設定內部部署資料閘道](../logic-apps/logic-apps-gateway-connection.md)後，存取內部部署 DB2 資料庫。 

IBM DB2 連接器支援以下 IBM DB2 平台和版本，以及 IBM DB2 相容產品 (例如 IBM Bluemix dashDB)，而這些產品支援分散式關聯資料庫架構 (DRDA) SQL 存取管理員 (SQLAM) 版本 10 和 11：

| 平台 | 版本 | 
|----------|---------|
| IBM DB2 for z/OS | 11.1, 10.1 |
| IBM DB2 for i | 7.3, 7.2, 7.1 |
| IBM DB2 for LUW | 11, 10.5 |
|||

## <a name="supported-database-operations"></a>支援的資料庫作業

IBM DB2 連接器支援以下資料庫作業，這些作業對應至連接器中的對應動作：

| 資料庫作業 | 連接器動作 | 
|--------------------|------------------|
| 列出資料庫資料表 | 取得資料表 | 
| 使用 SELECT 讀取一個資料列 | 取得單一資料列 | 
| 使用 SELECT 讀取所有資料列 | 取得多個資料列 | 
| 使用 INSERT 加入一個資料列 | 插入資料列 | 
| 使用 UPDATE 編輯一個資料列 | 更新資料列 | 
| 使用 DELETE 移除一個資料列 | 刪除資料列 | 
|||

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* IBM DB2 資料庫，雲端式或內部部署皆可

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 要在其中存取 DB2 資料庫的邏輯應用程式。 此連接器只提供動作，因此若要啟動邏輯應用程式，請選取個別觸發程序，例如**週期**觸發程序。
本文範例使用**週期**觸發程序。

<a name="add-db2-action"></a>

## <a name="add-db2-action---get-tables"></a>新增 DB2 動作 - 取得資料表

1. 在 [Azure 入口網站](https://portal.azure.com)中，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟邏輯應用程式。

1. 在觸發程序下方，選擇 [新增步驟]。

1. 在搜尋方塊中，輸入「db2」作為篩選條件。 針對此範例，請選取 [動作] 清單下的這個動作：**取得資料表 (預覽)**
   
   ![選取動作](./media/connectors-create-api-db2/select-db2-action.png)

   系統現在會提示您提供 DB2 資料庫的連線詳細資料。 

1. 遵循用於建立[雲端資料庫](#cloud-connection)或[內部部署資料庫](#on-premises-connection)連線的步驟。

<a name="cloud-connection"></a>

## <a name="connect-to-cloud-db2"></a>連線到雲端 DB2

若要設定連線，請在出現提示時提供以下連線詳細資料，選擇 [建立]，然後儲存邏輯應用程式：

| 屬性 | 必要 | 說明 | 
|----------|----------|-------------| 
| **透過內部部署閘道連線** | 否 | 僅適用於內部部署連線。 | 
| 連線名稱 | 是 | 連線的名稱，例如「MyLogicApp-DB2-connection」 |
| **伺服器** | 是 | DB2 伺服器的位址或別名冒號連接埠號碼，例如「myDB2server.cloudapp.net:50000」 <p><p>**注意**：這個值是字串，代表 TCP/IP 位址或別名 (採用 IPv4 或 IPv6 格式)，後面接著冒號和 TCP/IP 連接埠號碼。 |
| **資料庫** | 是 | 資料庫的名稱 <p><p>**注意**：這個值是字串，代表 DRDA 關聯式資料庫名稱 (RDBNAM)： <p>- DB2 for z/OS 接受 16 個位元組的字串 (其中，此資料庫也稱為「IBM DB2 for z/OS」位置)。 <br>- DB2 for i 接受 18 個位元組的字串 (其中，此資料庫也稱為「IBM DB2 for i」關聯式資料庫)。 <br>- DB2 for LUW 接受 8 個位元組的字串。 |
| **使用者名稱** | 是 | 資料庫的使用者名稱 <p><p>**注意**：這個值是字串，其長度以特定資料庫為基礎： <p><p>- DB2 for z/OS 接受 8 個位元組的字串。 <br>- DB2 for i 接受 10 個位元組的字串。 <br>- DB2 for Linux 或 UNIX 接受 8 個位元組的字串。 <br>- DB2 for Windows 接受 30 個位元組的字串。 | 
| **密碼** | 是 | 資料庫的密碼 | 
|||| 

例如︰

![雲端式資料庫的連線詳細資料](./media/connectors-create-api-db2/create-db2-cloud-connection.png)

<a name="on-premises-connection"></a>

## <a name="connect-to-on-premises-db2"></a>連線至內部部署 DB2

在建立連線前，必須先安裝好內部部署資料閘道。 否則，就無法完成連線設定。 如果有安裝閘道，請提供以下連線詳細資料來繼續進行，然後選擇 [建立]。

| 屬性 | 必要 | 說明 | 
|----------|----------|-------------| 
| **透過內部部署閘道連線** | 是 | 適用於當您想要使用內部部署連線，並顯示內部部署連線屬性時。 | 
| 連線名稱 | 是 | 連線的名稱，例如「MyLogicApp-DB2-connection」 | 
| **伺服器** | 是 | DB2 伺服器的位址或別名冒號連接埠號碼，例如「myDB2server:50000」 <p><p>**注意**：這個值是字串，代表 TCP/IP 位址或別名 (採用 IPv4 或 IPv6 格式)，後面接著冒號和 TCP/IP 連接埠號碼。 | 
| **資料庫** | 是 | 資料庫的名稱 <p><p>**注意**：這個值是字串，代表 DRDA 關聯式資料庫名稱 (RDBNAM)： <p>- DB2 for z/OS 接受 16 個位元組的字串 (其中，此資料庫也稱為「IBM DB2 for z/OS」位置)。 <br>- DB2 for i 接受 18 個位元組的字串 (其中，此資料庫也稱為「IBM DB2 for i」關聯式資料庫)。 <br>- DB2 for LUW 接受 8 個位元組的字串。 | 
| **驗證** | 是 | 連線的驗證類型，例如「基本」 <p><p>**注意**：請從清單選取此值，可選項目包括「基本」或「Windows (Kerberos)」。 | 
| **使用者名稱** | 是 | 資料庫的使用者名稱 <p><p>**注意**：這個值是字串，其長度以特定資料庫為基礎： <p><p>- DB2 for z/OS 接受 8 個位元組的字串。 <br>- DB2 for i 接受 10 個位元組的字串。 <br>- DB2 for Linux 或 UNIX 接受 8 個位元組的字串。 <br>- DB2 for Windows 接受 30 個位元組的字串。 | 
| **密碼** | 是 | 資料庫的密碼 | 
| **閘道** | 是 | 所安裝內部部署資料閘道的名稱 <p><p>**注意**：請從清單選取此值，可選項目包括 Azure 訂用帳戶和資源群組內的所有已安裝資料閘道。 | 
|||| 

例如︰

![內部部署資料庫的連線詳細資料](./media/connectors-create-api-db2/create-db2-on-premises-connection.png)

### <a name="view-output-tables"></a>檢視輸出資料表

若要手動執行邏輯應用程式，請在設計工具的工具列上，選擇 [執行]。 邏輯應用程式執行完成後，您即可檢視執行的輸出。

1. 在邏輯應用程式功能表上，選取 [概觀]。 

1. 在 [摘要] 底下的 [執行歷程記錄] 區段中，選取最新的執行，也就是清單中的第一個項目。 

   ![檢視執行歷程記錄](./media/connectors-create-api-db2/run-history.png)

1. 您現在可以在 [邏輯應用程式執行] 底下，檢閱邏輯應用程式中每個步驟的狀態、輸入和輸出。 展開 [取得資料表] 動作。

   ![展開動作](./media/connectors-create-api-db2/expand-action-step.png)

1. 若要檢視輸入，請選擇 [顯示原始輸入]。 

1. 若要檢視輸出，請選擇 [顯示原始輸出]。 

   輸出中會包含資料表清單。 
   
   ![檢視輸出資料表](./media/connectors-create-api-db2/db2-connector-get-tables-outputs.png)

## <a name="get-row"></a>取得單一資料列

若要擷取 DB2 資料庫資料表中的一筆記錄，請在邏輯應用程式中使用 [取得資料列] 動作。 這個動作會執行 DB2 `SELECT WHERE` 陳述式，例如 `SELECT FROM AREA WHERE AREAID = '99999'`。

1. 如果您之前從未在邏輯應用程式中使用過 DB2 動作，請檢閱[新增 DB2 動作 - 取得資料表](#add-db2-action)一節中的步驟，但改為新增**取得資料列**動作，然後再返回此處繼續。 

   新增 [取得資料列] 動作之後，以下是邏輯應用程式範例的樣貌：

   ![取得資料列動作](./media/connectors-create-api-db2/db2-get-row-action.png)

1. 指定所有必要屬性 (*) 的值。 選取資料表之後，此動作就會顯示該資料表中記錄特有的相關屬性。

   | 屬性 | 必要 | 說明 | 
   |----------|----------|-------------| 
   | **資料表名稱** | 是 | 具有所需記錄的資料表，例如此例中的「AREA」 | 
   | **區域識別碼** | 是 | 所需記錄的識別碼，例如此例中的「99999」 | 
   |||| 

   ![選取資料表](./media/connectors-create-api-db2/db2-get-row-action-select-table.png)

1. 當您完成時，請在設計工具的工具列上，選擇 [儲存]。 

### <a name="view-output-row"></a>檢視輸出資料列

若要手動執行邏輯應用程式，請在設計工具的工具列上，選擇 [執行]。 邏輯應用程式執行完成後，您即可檢視執行的輸出。

1. 在邏輯應用程式功能表上，選取 [概觀]。 

1. 在 [摘要] 底下的 [執行歷程記錄] 區段中，選取最新的執行，也就是清單中的第一個項目。 

1. 您現在可以在 [邏輯應用程式執行] 底下，檢閱邏輯應用程式中每個步驟的狀態、輸入和輸出。 展開 [取得資料列] 動作。

1. 若要檢視輸入，請選擇 [顯示原始輸入]。 

1. 若要檢視輸出，請選擇 [顯示原始輸出]。 

   輸出中會包含所指定的資料列。 
   
   ![檢視輸出資料列](./media/connectors-create-api-db2/db2-connector-get-row-outputs.png)

## <a name="get-rows"></a>取得多個資料列

若要擷取 DB2 資料庫資料表中的所有記錄，請在邏輯應用程式中使用 [取得資料列] 動作。 這個動作會執行 DB2 `SELECT` 陳述式，例如 `SELECT * FROM AREA`。

1. 如果您之前從未在邏輯應用程式中使用過 DB2 動作，請檢閱[新增 DB2 動作 - 取得資料表](#add-db2-action)一節中的步驟，但改為新增**取得資料列**動作，然後再返回此處繼續。 

   新增 [取得資料列] 動作之後，以下是邏輯應用程式範例的樣貌：

   ![取得資料列動作](./media/connectors-create-api-db2/db2-get-rows-action.png)

1. 開啟 [資料表名稱] 清單，然後選取所需的資料表，也就是此例中的「AREA」： 

   ![選取資料表](./media/connectors-create-api-db2/db2-get-rows-action-select-table.png)

1. 若要指定篩選條件或查詢結果，請選擇 [顯示進階選項]。

1. 當您完成時，請在設計工具的工具列上，選擇 [儲存]。 

### <a name="view-output-rows"></a>檢視輸出資料列

若要手動執行邏輯應用程式，請在設計工具的工具列上，選擇 [執行]。 邏輯應用程式執行完成後，您即可檢視執行的輸出。

1. 在邏輯應用程式功能表上，選取 [概觀]。 

1. 在 [摘要] 底下的 [執行歷程記錄] 區段中，選取最新的執行，也就是清單中的第一個項目。 

1. 您現在可以在 [邏輯應用程式執行] 底下，檢閱邏輯應用程式中每個步驟的狀態、輸入和輸出。 展開 [取得資料列] 動作。

1. 若要檢視輸入，請選擇 [顯示原始輸入]。 

1. 若要檢視輸出，請選擇 [顯示原始輸出]。 

   輸出中會包含所指定資料表中的所有記錄。
   
   ![檢視輸出資料列](./media/connectors-create-api-db2/db2-connector-get-rows-outputs.png)

## <a name="insert-row"></a>插入資料列

若要對 DB2 資料庫資料表新增一筆記錄，請在邏輯應用程式中使用 [插入資料列] 動作。 這個動作會執行 DB2 `INSERT` 陳述式，例如 `INSERT INTO AREA (AREAID, AREADESC, REGIONID) VALUES ('99999', 'Area 99999', 102)`。

1. 如果您之前從未在邏輯應用程式中使用過 DB2 動作，請檢閱[新增 DB2 動作 - 取得資料表](#add-db2-action)一節中的步驟，但改為新增**插入資料列**動作，然後再返回此處繼續。 

   新增 [插入資料列] 動作之後，以下是邏輯應用程式範例的樣貌：

   ![插入資料列動作](./media/connectors-create-api-db2/db2-insert-row-action.png)

1. 指定所有必要屬性 (*) 的值。 選取資料表之後，此動作就會顯示該資料表中記錄特有的相關屬性。 

   此範例的屬性如下：

   | 屬性 | 必要 | 說明 | 
   |----------|----------|-------------| 
   | **資料表名稱** | 是 | 用來新增記錄的資料表，例如「AREA」 | 
   | **區域識別碼** | 是 | 所要新增區域的識別碼，例如「99999」 | 
   | **區域說明** | 是 | 所要新增區域的說明，例如「Area 99999」 | 
   | **區域識別碼** | 是 | 所要新增區域的識別碼，例如「102」 | 
   |||| 

   例如︰

   ![選取資料表](./media/connectors-create-api-db2/db2-insert-row-action-select-table.png)

1. 當您完成時，請在設計工具的工具列上，選擇 [儲存]。 

### <a name="view-insert-row-outputs"></a>檢視插入資料列輸出

若要手動執行邏輯應用程式，請在設計工具的工具列上，選擇 [執行]。 邏輯應用程式執行完成後，您即可檢視執行的輸出。

1. 在邏輯應用程式功能表上，選取 [概觀]。 

1. 在 [摘要] 底下的 [執行歷程記錄] 區段中，選取最新的執行，也就是清單中的第一個項目。 

1. 您現在可以在 [邏輯應用程式執行] 底下，檢閱邏輯應用程式中每個步驟的狀態、輸入和輸出。 展開 [插入資料列] 動作。

1. 若要檢視輸入，請選擇 [顯示原始輸入]。 

1. 若要檢視輸出，請選擇 [顯示原始輸出]。 

   輸出中會包含新增至指定資料表的記錄。
   
   ![檢視有所插入資料列的輸出](./media/connectors-create-api-db2/db2-connector-insert-row-outputs.png)

## <a name="update-row"></a>更新資料列

若要在 DB2 資料庫資料表中更新一筆記錄，請在邏輯應用程式中使用 [更新資料列] 動作。 這個動作會執行 DB2 `UPDATE` 陳述式，例如 `UPDATE AREA SET AREAID = '99999', AREADESC = 'Updated 99999', REGIONID = 102)`。

1. 如果您之前從未在邏輯應用程式中使用過 DB2 動作，請檢閱[新增 DB2 動作 - 取得資料表](#add-db2-action)一節中的步驟，但改為新增**更新資料列**動作，然後再返回此處繼續。 

   新增 [更新資料列] 動作之後，以下是邏輯應用程式範例的樣貌：

   ![更新資料列動作](./media/connectors-create-api-db2/db2-update-row-action.png)

1. 指定所有必要屬性 (*) 的值。 選取資料表之後，此動作就會顯示該資料表中記錄特有的相關屬性。 

   此範例的屬性如下：

   | 屬性 | 必要 | 說明 | 
   |----------|----------|-------------| 
   | **資料表名稱** | 是 | 用來更新記錄的資料表，例如「AREA」 | 
   | **資料列識別碼** | 是 | 所要更新記錄的識別碼，例如「99999」 | 
   | **區域識別碼** | 是 | 新區域的識別碼，例如「99999」 | 
   | **區域說明** | 是 | 新區域的說明，例如「Updated 99999」 | 
   | **區域識別碼** | 是 | 新區域的識別碼，例如「102」 | 
   |||| 

   例如︰

   ![選取資料表](./media/connectors-create-api-db2/db2-update-row-action-select-table.png)

1. 當您完成時，請在設計工具的工具列上，選擇 [儲存]。 

### <a name="view-update-row-outputs"></a>檢視更新資料列輸出

若要手動執行邏輯應用程式，請在設計工具的工具列上，選擇 [執行]。 邏輯應用程式執行完成後，您即可檢視執行的輸出。

1. 在邏輯應用程式功能表上，選取 [概觀]。 

1. 在 [摘要] 底下的 [執行歷程記錄] 區段中，選取最新的執行，也就是清單中的第一個項目。 

1. 您現在可以在 [邏輯應用程式執行] 底下，檢閱邏輯應用程式中每個步驟的狀態、輸入和輸出。 展開 [更新資料列] 動作。

1. 若要檢視輸入，請選擇 [顯示原始輸入]。 

1. 若要檢視輸出，請選擇 [顯示原始輸出]。 

   輸出中會包含指定資料表中所更新的記錄。
   
   ![檢視有所更新資料列的輸出](./media/connectors-create-api-db2/db2-connector-update-row-outputs.png)

## <a name="delete-row"></a>刪除資料列

若要從 DB2 資料庫資料表中刪除一筆記錄，請在邏輯應用程式中使用 [刪除資料列] 動作。 這個動作會執行 DB2 `DELETE` 陳述式，例如 `DELETE FROM AREA WHERE AREAID = '99999'`。

1. 如果您之前從未在邏輯應用程式中使用過 DB2 動作，請檢閱[新增 DB2 動作 - 取得資料表](#add-db2-action)一節中的步驟，但改為新增**刪除資料列**動作，然後再返回此處繼續。 

   新增 [刪除資料列] 動作之後，以下是邏輯應用程式範例的樣貌：

   ![刪除資料列動作](./media/connectors-create-api-db2/db2-delete-row-action.png)

1. 指定所有必要屬性 (*) 的值。 選取資料表之後，此動作就會顯示該資料表中記錄特有的相關屬性。 

   此範例的屬性如下：

   | 屬性 | 必要 | 說明 | 
   |----------|----------|-------------| 
   | **資料表名稱** | 是 | 用來刪除記錄的資料表，例如「AREA」 | 
   | **資料列識別碼** | 是 | 所要刪除記錄的識別碼，例如「99999」 | 
   |||| 

   例如︰

   ![選取資料表](./media/connectors-create-api-db2/db2-delete-row-action-select-table.png)

1. 當您完成時，請在設計工具的工具列上，選擇 [儲存]。 

### <a name="view-delete-row-outputs"></a>檢視刪除資料列輸出

若要手動執行邏輯應用程式，請在設計工具的工具列上，選擇 [執行]。 邏輯應用程式執行完成後，您即可檢視執行的輸出。

1. 在邏輯應用程式功能表上，選取 [概觀]。 

1. 在 [摘要] 底下的 [執行歷程記錄] 區段中，選取最新的執行，也就是清單中的第一個項目。 

1. 您現在可以在 [邏輯應用程式執行] 底下，檢閱邏輯應用程式中每個步驟的狀態、輸入和輸出。 展開 [刪除資料列] 動作。

1. 若要檢視輸入，請選擇 [顯示原始輸入]。 

1. 若要檢視輸出，請選擇 [顯示原始輸出]。 

   輸出中不會再包含已從指定資料表中刪除的記錄。
   
   ![檢視沒有所刪除資料列的輸出](./media/connectors-create-api-db2/db2-connector-delete-row-outputs.png)

## <a name="connector-reference"></a>連接器參考

如需連接器的 Swagger 檔案所敘述的技術詳細資料 (例如，觸發程序、動作和限制)，請參閱[連接器的參考頁面](/connectors/db2/)。 

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)
