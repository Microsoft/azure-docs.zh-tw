---
title: 連線到 Dynamics 365 - Azure Logic Apps | Microsoft Docs
description: 使用 Dynamics 365 (線上) REST API 和 Azure Logic Apps 建立及管理記錄
author: Mattp123
ms.author: matp
ms.service: logic-apps
services: logic-apps
ms.reviewer: estfan, LADocs
ms.suite: integration
ms.topic: article
ms.date: 08/18/2018
tags: connectors
ms.openlocfilehash: b1ff93f1e03e047ad5ac00259c1aa53afda0c76d
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818935"
---
# <a name="manage-dynamics-365-records-with-azure-logic-apps"></a>使用 Azure Logic Apps 管理 Dynamics 365 記錄

使用 Azure Logic Apps 和 Dynamics 365 連接器，您即可根據 Dynamics 365 中的記錄，建立自動化的工作和工作流程。 工作流程可以在 Dynamics 365 帳戶中建立記錄、更新項目、傳回記錄，以及進行其他各項工作。 您可以將動作納入邏輯應用程式中，以取得 Dynamics 365 的回應，並提供輸出給其他動作使用。 舉例來說，當 Dynamics 365 中有項目更新時，您可以利用 Office 365 來傳送電子郵件。

本文說明如何建置邏輯應用程式，以在每次 Dynamics 365 中建立了新的潛在客戶記錄時，便在 Dynamics 365 建立工作。
如果您不熟悉邏輯應用程式，請檢閱[什麼是 Azure Logic Apps？](../logic-apps/logic-apps-overview.md)。

## <a name="prerequisites"></a>必要條件

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請先<a href="https://azure.microsoft.com/free/" target="_blank">註冊免費的 Azure 帳戶</a>。 

* [Dynamics 365 帳戶](https://dynamics.microsoft.com)

* [如何建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)的基本知識

* 您要在其中存取 Dynamics 365 帳戶的邏輯應用程式。 若要使用 Dynamics 365 觸發程序啟動邏輯應用程式，您需要[空白邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)。 

## <a name="add-dynamics-365-trigger"></a>新增 Dynamics 365 觸發程序

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

首先，新增會在 Dynamics 365 中出現新的潛在客戶記錄時引發的 Dynamics 365 觸發程序。

1. 在 [Azure 入口網站](https://portal.azure.com)中，如果邏輯應用程式尚未開啟，請在邏輯應用程式設計工具中開啟空白邏輯應用程式。

1. 在搜尋方塊中，輸入「Dynamics 365」作為篩選條件。 在此範例中，請在 [觸發程序] 清單下選取此觸發程序：**建立記錄時**

   ![選取觸發程序](./media/connectors-create-api-crmonline/select-dynamics-365-trigger.png)

1. 如果系統提示您登入 Dynamics 365，請立即登入。

1. 提供以下觸發程序詳細資料：

   | 屬性 | 必要 | 說明 | 
   |----------|----------|-------------| 
   | **組織名稱** | 是 | 要監視的組織 Dynamics 365 執行個體名稱，例如「Contoso」 |
   | **實體名稱** | 是 | 要監視的實體名稱，例如「Leads」 | 
   | **頻率** | 是 | 在檢查是否有觸發程序相關更新時要對間隔使用的時間單位 |
   | **間隔** | 是 | 在下一次檢查之前要經過的秒數、分鐘數、小時數天數、週數或月數 |
   ||| 

   ![觸發程序詳細資料](./media/connectors-create-api-crmonline/trigger-details.png)

## <a name="add-dynamics-365-action"></a>新增 Dynamics 365 動作

現在，新增 Dynamics 365 動作，為新的潛在客戶記錄建立工作記錄。

1. 在觸發程序下方，選擇 [新增步驟]。

1. 在搜尋方塊中，輸入「Dynamics 365」作為篩選條件。 從 [動作] 清單中，選取此動作：**建立新記錄**

   ![選取動作](./media/connectors-create-api-crmonline/select-action.png)

1. 提供以下動作詳細資料：

   | 屬性 | 必要 | 說明 | 
   |----------|----------|-------------| 
   | **組織名稱** | 是 | 要在其中建立記錄的 Dynamics 365 執行個體，不一定要與觸發程序中的執行個體相同，但在本例中為「Contoso」 |
   | **實體名稱** | 是 | 要在其中建立記錄的實體，例如「Tasks」 | 
   | | |

   ![動作詳細資料](./media/connectors-create-api-crmonline/action-details.png)

1. 當動作中出現 [主旨] 方塊時，按一下 [主旨] 方塊內部，讓動態內容清單出現。 從這份清單中，選取欄位值以包含在與新的潛在客戶記錄相關聯的工作記錄中：

   | 欄位 | 說明 | 
   |-------|-------------| 
   | **姓氏** | 作為記錄中主要連絡人的潛在客戶姓氏 |
   | **主題** | 記錄中潛在客戶的描述性名稱 | 
   | | | 

   ![工作記錄詳細資料](./media/connectors-create-api-crmonline/create-record-details.png)

1. 在設計工具的工具列上，針對邏輯應用程式選擇 [儲存]。 

1. 若要手動啟動邏輯應用程式，請在設計工具的工具列上，選擇 [執行]。

   ![執行邏輯應用程式](./media/connectors-create-api-crmonline/designer-toolbar-run.png)

1. 現在，在 Dynamics 365 中建立潛在客戶記錄，以便能夠觸發邏輯應用程式的工作流程。

## <a name="add-filter-or-query"></a>新增篩選條件或查詢

若要指定如何篩選 Dynamics 365 動作中的資料，請選擇該動作的 [顯示進階選項]。 您接著可以新增篩選條件或依查詢來排序。
例如，您可以使用篩選條件查詢只取得作用中的帳戶，並依帳戶名稱排序這些記錄。 針對這項工作，請遵循下列步驟：

1. 在 [篩選查詢] 底下，輸入此 OData 篩選查詢：`statuscode eq 1`

2. 在 [排序依據] 底下，當動態內容清單出現時，選取 [帳戶名稱]。 

   ![指定篩選條件和順序](./media/connectors-create-api-crmonline/advanced-options.png)

如需詳細資訊，請參閱下列 Dynamics 365 Customer Engagement Web API 系統查詢選項： 

* [$filter](https://docs.microsoft.com/dynamics365/customer-engagement/developer/webapi/query-data-web-api#filter-results)
* [$orderby](https://docs.microsoft.com/dynamics365/customer-engagement/developer/webapi/query-data-web-api#order-results)

### <a name="best-practices-for-advanced-options"></a>進階選項的最佳做法

當您在動作或觸發程序中指定欄位的值時，無論您以手動方式輸入值，還是從動態內容清單中選取值，值的資料類型都必須符合欄位類型。

下表描述部分欄位類型和其值的必要資料類型。

| 欄位類型 | 必要資料類型 | 說明 | 
|------------|--------------------|-------------|
| 文字欄位 | 單行文字 | 這些欄位需要單行文字或具有文字類型的動態內容。 <p><p>範例欄位：**描述**和**類別** | 
| 整數欄位 | 整數 | 某些欄位需要整數或具有整數類型的動態內容。 <p><p>範例欄位：**完成百分比**和**持續時間** | 
| 日期欄位 | 日期和時間 | 某些欄位需要 mm/dd/yyyy 格式的日期或具有日期類型的動態內容。 <p><p>範例欄位：**建立日期**、**開始日期**、**實際開始時間**、**實際結束時間**和**到期日** | 
| 同時需要記錄識別碼和查閱類型的欄位 | 主索引鍵 | 某些參考另一個實體記錄的欄位同時需要記錄識別碼和查閱類型。 | 
||||

為了詳述這些欄位類型，以下是 Dynamics 365 觸發程序和動作中同時需要記錄識別碼和查閱類型的欄位範例。 此需求代表從動態清單選取的值不會有作用。 

| 欄位 | 說明 | 
|-------|-------------|
| **擁有者** | 必須是有效的使用者識別碼或小組記錄識別碼。 | 
| **擁有者類型** | 必須是**系統使用者**或**小組**。 | 
| **關於** | 必須是有效的記錄識別碼，例如帳戶識別碼或連絡人記錄識別碼。 | 
| **關於類型** | 必須是查閱類型，例如**帳戶**或**連絡人**。 | 
| **客戶** | 必須是有效的記錄識別碼，例如帳戶識別碼或連絡人記錄識別碼。 | 
| **客戶類型** | 必須是查閱類型，例如**帳戶**或**連絡人**。 | 
|||

在此範例中，名為**建立新記錄**的動作會建立新的工作記錄： 

![使用記錄識別碼和查閱類型建立工作記錄](./media/connectors-create-api-crmonline/create-record-advanced.png)

此動作會將工作記錄指派給特定使用者識別碼或小組記錄識別碼 (根據 [擁有者] 欄位中的記錄識別碼，以及 [擁有者類型] 欄位中的查閱類型)：

![擁有者記錄識別碼和查閱類型](./media/connectors-create-api-crmonline/owner-record-id-and-lookup-type.png)

此動作也會將與記錄識別碼相關聯的帳戶記錄新增到 [關於] 欄位，以及將查閱類型新增到 [關於類型] 欄位： 

![關於記錄識別碼和查閱類型](./media/connectors-create-api-crmonline/regarding-record-id-lookup-type-account.png)

## <a name="find-record-id"></a>尋找記錄識別碼

若要尋找記錄識別碼，請遵循下列步驟： 

1. 在 Dynamics 365 中開啟記錄，例如帳戶記錄。

2. 在 [動作] 工具列中，選擇下列其中一個步驟：

   * 選擇 [快顯] ![彈出記錄](./media/connectors-create-api-crmonline/popout-record.png)。 
   * 選擇 [以電子郵件傳送連結]，以便將完整 URL 複製到預設電子郵件程式。

   記錄識別碼會出現在 URL 中的 `%7b` 和 `%7d` 編碼字元之間：

   ![尋找記錄識別碼](./media/connectors-create-api-crmonline/find-record-ID.png)

## <a name="troubleshoot-failed-runs"></a>針對失敗的執行進行疑難排解

若要尋找並檢閱邏輯應用程式中失敗的步驟，您可以檢視邏輯應用程式的執行歷程記錄、狀態、輸入、輸出等項目。

1. 在 Azure 入口網站中，於邏輯應用程式的主功能表上選取 [概觀]。 在 [執行歷程記錄] 區段中 (會顯示邏輯應用程式的所有執行狀態)，選取失敗的執行以獲得詳細資訊。

   ![邏輯應用程式執行狀態](./media/connectors-create-api-crmonline/run-history.png)

1. 展開失敗的步驟，以便檢視詳細資訊。 

   ![展開失敗的步驟](./media/connectors-create-api-crmonline/expand-failed-step.png)

1. 檢閱步驟的詳細資料 (例如輸入和輸出)，其可協助您找出失敗背後的原因。

   ![失敗的步驟 - 輸入和輸出](./media/connectors-create-api-crmonline/expand-failed-step-inputs-outputs.png)

如需針對邏輯應用程式進行疑難排解的詳細資訊，請參閱[診斷邏輯應用程式失敗](../logic-apps/logic-apps-diagnosing-failures.md)。

## <a name="connector-reference"></a>連接器參考

如需連接器的 Swagger 檔案所敘述的技術詳細資料 (例如，觸發程序、動作和限制)，請參閱[連接器的參考頁面](/connectors/crm/)。 

## <a name="get-support"></a>取得支援

* 如有問題，請瀏覽 [Azure Logic Apps 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps)。
* 若要提交或票選功能構想，請造訪 [Logic Apps 使用者意見反應網站](http://aka.ms/logicapps-wish)。

## <a name="next-steps"></a>後續步驟

* 了解其他 [Logic Apps 連接器](../connectors/apis-list.md)
