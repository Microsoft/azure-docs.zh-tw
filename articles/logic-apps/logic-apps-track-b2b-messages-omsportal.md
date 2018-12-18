---
title: 使用 Azure Log Analytics 追蹤 B2B 訊息 - Azure Logic Apps | Microsoft Docs
description: 使用 Azure Log Analytics 追蹤整合帳戶和 Azure Logic Apps 的 B2B 通訊
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: bb7d9432-b697-44db-aa88-bd16ddfad23f
ms.date: 06/19/2018
ms.openlocfilehash: 5bf5385824eb9b711a2fee547c29d24d7ef5a01d
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43125763"
---
# <a name="track-b2b-communication-with-azure-log-analytics"></a>使用 Azure Log Analytics 追蹤 B2B 通訊

在您透過整合帳戶設定兩個執行中商務程序或應用程式之間的 B2B 通訊之後，這些實體也可以彼此交換訊息。 若要檢查是否正確處理這些訊息，您可以使用 [Azure Log Analytics](../log-analytics/log-analytics-overview.md) 追蹤 AS2、X12 和 EDIFACT 訊息。 例如，您可以使用這些網頁追蹤功能來追蹤訊息：

* 訊息計數和狀態
* 通知狀態
* 訊息與通知的相互關聯
* 失敗的詳細錯誤描述
* 搜尋功能

## <a name="requirements"></a>需求

* 已設定診斷記錄的邏輯應用程式。 了解[如何建立邏輯應用程式](quickstart-create-first-logic-app-workflow.md)和[如何設定該邏輯應用程式的記錄](../logic-apps/logic-apps-monitor-your-logic-apps.md#azure-diagnostics)。

* 已設定監視和記錄的整合帳戶。 了解[如何建立整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)和[如何設定該帳戶的監視和記錄](../logic-apps/logic-apps-monitor-b2b-message.md)。

* 如果您還沒有這麼做，請[將診斷資料發佈至 Log Analytics](../logic-apps/logic-apps-track-b2b-messages-omsportal.md)。

> [!NOTE]
> 在您符合先前的需求之後，Log Analytics 中應該有工作區。 您應該在 Log Analytics 中使用相同的工作區來追蹤 B2B 通訊。 
>  
> 如果您沒有 Log Analytics 工作區，請了解[如何建立 Log Analytics 工作區](../log-analytics/log-analytics-quick-create-workspace.md)。

## <a name="add-the-logic-apps-b2b-solution-to-log-analytics"></a>將 Logic Apps B2B 解決方案新增至 Log Analytics

若要讓 Log Analytics 追蹤邏輯應用程式的 B2B 訊息，您必須將 **Logic Apps B2B** 解決方案新增至 OMS 入口網站。 深入了解[將解決方案新增至 Log Analytics](../log-analytics/log-analytics-quick-create-workspace.md)。

1. 在 [Azure 入口網站](https://portal.azure.com)中，選擇 [所有服務]。 搜尋 "log analytics"，然後選擇 [Log Analytics]，如下所示：

   ![尋找 Log Analytics](media/logic-apps-track-b2b-messages-omsportal/browseloganalytics.png)

2. 在 [Log Analytics] 下，尋找並選取 Log Analytics 工作區。 

   ![選取 Log Analytics 工作區](media/logic-apps-track-b2b-messages-omsportal/selectla.png)

3. 在 [管理] 下，選擇 [概觀]。

   ![選擇 Log Analytics 入口網站](media/logic-apps-track-b2b-messages-omsportal/omsportalpage.png)

4. 首頁開啟後，選擇 [新增] 以安裝 Logic Apps B2B 解決方案。    
   ![選擇方案庫](media/logic-apps-track-b2b-messages-omsportal/add-b2b-solution.png)

5. 在 [管理解決方案] 下，尋找和建立 [Logic Apps B2B] 解決方案。     
   ![選擇 Logic Apps B2B](media/logic-apps-track-b2b-messages-omsportal/create-b2b-solution.png)

   在首頁上，現在會顯示 [Logic Apps B2B 訊息] 的圖格。 
   處理 B2B 訊息時，此磚會更新訊息計數。

<a name="message-status-details"></a>

## <a name="track-message-status-and-details-in-log-analytics"></a>在 Log Analytics 中追蹤訊息狀態和詳細資料

1. 處理 B2B 訊息之後，即可檢視這些訊息的狀態和詳細資料。 在 [概觀] 頁面上，選擇 [Logic Apps B2B 訊息] 圖格。

   ![更新的訊息計數](media/logic-apps-track-b2b-messages-omsportal/b2b-overview-tile.png)

   > [!NOTE]
   > [Logic Apps B2B 訊息] 磚預設會顯示以一天為基礎的資料。 若要將資料範圍變更為不同的間隔，請選擇頁面頂端的範圍控制項：
   > 
   > ![變更資料範圍](media/logic-apps-track-b2b-messages-omsportal/server-filter.png)
   >

2. 訊息狀態儀表板出現之後，即可檢視特定訊息類型的其他詳細資料，這會顯示以一天為基礎的資料。 選擇 [AS2]、[X12] 或 [EDIFACT] 的磚。

   ![檢視訊息狀態](media/logic-apps-track-b2b-messages-omsportal/omshomepage5.png)

   會針對您所選擇的磚顯示訊息清單。 
   若要深入了解每種訊息類型的屬性，請參閱這些訊息屬性描述：

   * [AS2 訊息屬性](#as2-message-properties)
   * [X12 訊息屬性](#x12-message-properties)
   * [EDIFACT 訊息屬性](#EDIFACT-message-properties)

   例如，AS2 訊息清單可能如下：

   ![檢視 AS2 訊息](media/logic-apps-track-b2b-messages-omsportal/as2messagelist.png)

3. 若要檢視或匯出特定訊息的輸入和輸出，請選取這些訊息，然後選擇 [下載]。 系統提示您時，請將 .zip 檔案儲存至本機電腦，然後解壓縮該檔案。 

   解壓縮的資料夾包含每個所選訊息的資料夾。 
   如果您設定通知，訊息資料夾也會包含具有通知詳細資料的檔案。 
   每個訊息資料夾都會至少有這些檔案： 
   
   * 具有輸入承載和輸出承載詳細資料的人類可讀檔案
   * 具有輸入和輸出的編碼檔案

   您可以在這裡找到每種訊息類型的資料夾和檔案名稱格式：

   * [AS2 資料夾和檔案名稱格式](#as2-folder-file-names)
   * [X12 資料夾和檔案名稱格式](#x12-folder-file-names)
   * [EDIFACT 資料夾和檔案名稱格式](#edifact-folder-file-names)

   ![下載訊息檔案](media/logic-apps-track-b2b-messages-omsportal/download-messages.png)

4. 若要檢視所有具有相同執行識別碼的動作，請在 [記錄搜尋] 頁面的訊息清單中選擇一則訊息。

   您可以依資料行排序這些動作，或搜尋特定結果。

   ![具有相同執行識別碼的動作](media/logic-apps-track-b2b-messages-omsportal/logsearch.png)

   * 若要搜尋具有預先建置查詢的結果，請選擇 [我的最愛]。

   * 了解[如何新增篩選來建置查詢](logic-apps-track-b2b-messages-omsportal-query-filter-control-number.md)。 
   或者深入了解[如何在 Log Analytics 中使用記錄搜尋尋找資料](../log-analytics/log-analytics-log-searches.md)。

   * 若要變更搜尋方塊中的查詢，請使用您想要用作篩選的資料行和值來更新查詢。

<a name="message-list-property-descriptions"></a>

## <a name="property-descriptions-and-name-formats-for-as2-x12-and-edifact-messages"></a>AS2、X12 和 EDIFACT 訊息的屬性描述和名稱格式

對於每種訊息類型，以下是所下載訊息檔案的屬性描述和名稱格式。

<a name="as2-message-properties"></a>

### <a name="as2-message-property-descriptions"></a>AS2 訊息屬性描述

以下是每個 AS2 訊息的屬性描述。

| 屬性 | 說明 |
| --- | --- |
| 傳送者 | 針對 AS2 協議的 [接收設定] 中所指定的來賓合作夥伴，或 [傳送設定] 中所指定的主機合作夥伴 |
| 接收者 | 針對 AS2 協議的 [接收設定] 中所指定的主機合作夥伴，或 [傳送設定] 中所指定的來賓合作夥伴 |
| 邏輯應用程式 | 已設定 AS2 動作的邏輯應用程式 |
| 狀態 | AS2 訊息狀態 <br>成功 = 已接收或傳送有效的 AS2 訊息。 未設定 MDN。 <br>成功 = 已接收或傳送有效的 AS2 訊息。 已設定並接收 MDN，或傳送 MDN。 <br>失敗 = 已接收無效的 AS2 訊息。 未設定 MDN。 <br>暫止 = 已接收或傳送有效的 AS2 訊息。 已設定 MDN，並預期要有 MDN。 |
| Ack | MDN 訊息狀態 <br>接受 = 已接收或傳送正值的 MDN。 <br>暫止 = 等候接收或傳送 MDN。 <br>拒絕 = 已接收或傳送負值的 MDN。 <br>不需要 = 未在協議中設定 MDN。 |
| 方向 | AS2 訊息方向 |
| 相互關連識別碼 | 與邏輯應用程式中所有觸發程序和動作相關聯的識別碼 |
| 訊息識別碼 | 來自 AS2 訊息標頭的 AS2 訊息識別碼 |
| Timestamp | AS2 動作處理訊息的時間 |
|          |             |

<a name="as2-folder-file-names"></a>

### <a name="as2-name-formats-for-downloaded-message-files"></a>所下載訊息檔案的 AS2 名稱格式

以下是每個所下載 AS2 訊息資料夾和檔案的名稱格式。

| 資料夾或檔案 | 名稱格式 |
| :------------- | :---------- |
| 訊息資料夾 | [寄件者]\_[接收者]\_AS2\_[相互關聯識別碼]\_[訊息識別碼]\_[時間戳記] |
| 輸入、輸出和通知檔案 (若已設定) | **輸入承載**：[寄件者]\_[接收者]\_AS2\_[相互關聯識別碼]\_input_payload.txt </p>**輸出承載**：[寄件者]\_[接收者]\_AS2\_[相互關聯識別碼]\_output\_payload.txt </p></p>**輸入**：[寄件者]\_[接收者]\_AS2\_[相互關聯識別碼]\_inputs.txt </p></p>**輸出**：[寄件者]\_[接收者]\_AS2\_[相互關聯識別碼]\_outputs.txt |
|          |             |

<a name="x12-message-properties"></a>

### <a name="x12-message-property-descriptions"></a>X12 訊息屬性描述

以下是每個 X12 訊息的屬性描述。

| 屬性 | 說明 |
| --- | --- |
| 傳送者 | 針對 X12 協議的 [接收設定] 中所指定的來賓合作夥伴，或 [傳送設定] 中所指定的主機合作夥伴 |
| 接收者 | 針對 X12 協議的 [接收設定] 中所指定的主機合作夥伴，或 [傳送設定] 中所指定的來賓合作夥伴 |
| 邏輯應用程式 | 已設定 X12 動作的邏輯應用程式 |
| 狀態 | X12 訊息狀態 <br>成功 = 已接收或傳送有效的 X12 訊息。 未設定任何功能通知。 <br>成功 = 已接收或傳送有效的 X12 訊息。 已設定和接收功能通知，或傳送功能通知。 <br>失敗 = 已接收或傳送有效的 X12 訊息。 <br>暫止 = 已接收或傳送有效的 X12 訊息。 已設定功能通知，並預期要有功能通知。 |
| Ack | 功能認可 (997) 狀態 <br>接受 = 已接收或傳送正值的功能通知。 <br>拒絕 = 已接收或傳送負值的功能通知。 <br>暫止 = 預期要有功能通知但未收到。 <br>暫止 = 已產生功能通知，但無法傳送給合作夥伴。 <br>不需要 = 未設定功能通知。 |
| 方向 | X12 訊息方向 |
| 相互關連識別碼 | 與邏輯應用程式中所有觸發程序和動作相關聯的識別碼 |
| 訊息類型 | EDI X12 訊息類型 |
| ICN | X12 訊息的交換控制編號 |
| TSCN | X12 訊息的交易集控制編號 |
| Timestamp | X12 動作處理訊息的時間 |
|          |             |

<a name="x12-folder-file-names"></a>

### <a name="x12-name-formats-for-downloaded-message-files"></a>所下載訊息檔案的 X12 名稱格式

以下是每個所下載 X12 訊息資料夾和檔案的名稱格式。

| 資料夾或檔案 | 名稱格式 |
| :------------- | :---------- |
| 訊息資料夾 | [寄件者]\_[接收者]\_X12\_[交換控制編號]\_[通用控制編號]\_[交易集控制編號]\_[時間戳記] |
| 輸入、輸出和通知檔案 (若已設定) | **輸入承載**：[寄件者]\_[接收者]\_X12\_[交換控制編號]\_input_payload.txt </p>**輸出承載**：[寄件者]\_[接收者]\_X12\_[交換控制編號]\_output\_payload.txt </p></p>**輸入**：[寄件者]\_[接收者]\_X12\_[交換控制編號]\_inputs.txt </p></p>**輸出**：[寄件者]\_[接收者]\_X12\_[交換控制編號]\_outputs.txt |
|          |             |

<a name="EDIFACT-message-properties"></a>

### <a name="edifact-message-property-descriptions"></a>EDIFACT 訊息屬性描述

以下是每個 EDIFACT 訊息的屬性描述。

| 屬性 | 說明 |
| --- | --- |
| 傳送者 | 針對 EDIFACT 協議的 [接收設定] 中所指定的來賓合作夥伴，或 [傳送設定] 中所指定的主機合作夥伴 |
| 接收者 | 針對 EDIFACT 協議的 [接收設定] 中所指定的主機合作夥伴，或 [傳送設定] 中所指定的來賓合作夥伴 |
| 邏輯應用程式 | 已設定 EDIFACT 動作的邏輯應用程式 |
| 狀態 | EDIFACT 訊息狀態 <br>成功 = 已接收或傳送有效的 EDIFACT 訊息。 未設定任何功能通知。 <br>成功 = 已接收或傳送有效的 EDIFACT 訊息。 已設定和接收功能通知，或傳送功能通知。 <br>失敗 = 已接收或傳送有效的 EDIFACT 訊息。 <br>暫止 = 已接收或傳送有效的 EDIFACT 訊息。 已設定功能通知，並預期要有功能通知。 |
| Ack | 功能認可 (997) 狀態 <br>接受 = 已接收或傳送正值的功能通知。 <br>拒絕 = 已接收或傳送負值的功能通知。 <br>暫止 = 預期要有功能通知但未收到。 <br>暫止 = 已產生功能通知，但無法傳送給合作夥伴。 <br>不需要 = 未設定功能通知。 |
| 方向 | EDIFACT 訊息方向 |
| 相互關連識別碼 | 與邏輯應用程式中所有觸發程序和動作相關聯的識別碼 |
| 訊息類型 | EDIFACT 訊息類型 |
| ICN | EDIFACT 訊息的交換控制編號 |
| TSCN | EDIFACT 訊息的交易集控制編號 |
| Timestamp | EDIFACT 動作處理訊息的時間 |
|          |               |

<a name="edifact-folder-file-names"></a>

### <a name="edifact-name-formats-for-downloaded-message-files"></a>所下載訊息檔案的 EDIFACT 名稱格式

以下是每個所下載 EDIFACT 訊息資料夾和檔案的名稱格式。

| 資料夾或檔案 | 名稱格式 |
| :------------- | :---------- |
| 訊息資料夾 | [寄件者]\_[接收者]\_EDIFACT\_[交換控制編號]\_[通用控制編號]\_[交易集控制編號]\_[時間戳記] |
| 輸入、輸出和通知檔案 (若已設定) | **輸入承載**：[寄件者]\_[接收者]\_EDIFACT\_[交換控制編號]\_input_payload.txt </p>**輸出承載**：[寄件者]\_[接收者]\_EDIFACT\_[交換控制編號]\_output\_payload.txt </p></p>**輸入**：[寄件者]\_[接收者]\_EDIFACT\_[交換控制編號]\_inputs.txt </p></p>**輸出**：[寄件者]\_[接收者]\_EDIFACT\_[交換控制編號]\_outputs.txt |
|          |             |

## <a name="next-steps"></a>後續步驟

* [查詢 Log Analytics 中的 B2B 訊息](../logic-apps/logic-apps-track-b2b-messages-omsportal-query-filter-control-number.md)
* [AS2 追蹤結構描述](../logic-apps/logic-apps-track-integration-account-as2-tracking-schemas.md)
* [X12 追蹤結構描述](../logic-apps/logic-apps-track-integration-account-x12-tracking-schema.md)
* [自訂追蹤結構描述](../logic-apps/logic-apps-track-integration-account-custom-tracking-schema.md)