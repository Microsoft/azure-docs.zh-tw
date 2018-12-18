---
title: 監視虛擬機器變更 - Azure Event Grid 和 Logic Apps | Microsoft Docs
description: 使用 Azure Event Grid 和 Logic Apps 來檢查虛擬機器 (VM) 的組態變更
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: tutorial
ms.date: 11/30/2017
ms.openlocfilehash: 29b28b0d81314d062c1b334092979cc9bccbeb31
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43127654"
---
# <a name="monitor-virtual-machine-changes-with-azure-event-grid-and-logic-apps"></a>使用 Azure Event Grid 和 Logic Apps 監視虛擬機器變更

當 Azure 資源或第三方資源發生特定事件時，您可以啟動自動化[邏輯應用程式工作流程](../logic-apps/logic-apps-overview.md)。 這些資源可以將這些事件發佈至 [Azure Event Grid](../event-grid/overview.md)。 接著，Event Grid 會將這些事件發送給以佇列、Webhook 或[事件中心](../event-hubs/event-hubs-what-is-event-hubs.md)作為端點的訂閱者。 身為訂閱者，邏輯應用程式可以先等待來自 Event Grid 的這些事件，再執行自動化工作流程來執行工作 - 您無須等待任何程式碼。

例如，以下是發行者可以透過 Azure Event Grid 服務傳送給訂閱者的一些事件：

* 建立、讀取、更新或刪除資源。 例如，您可以監視可能會產生 Azure 訂用帳戶費用並會影響帳單的變更。 
* 從 Azure 訂用帳戶中新增或移除人員。
* 您的應用程式會執行特定動作。
* 新的訊息會出現在佇列中。

本教學課程會建立邏輯應用程式，可監視虛擬機器的變更，並傳送有關這些變更的電子郵件。 當您建立的邏輯應用程式具有 Azure 資源的事件訂用帳戶時，事件會透過 Event Grid 從該資源流向邏輯應用程式。 本教學課程會引導您建立此邏輯應用程式：

![概觀 - 使用 Event Grid 和邏輯應用程式來監視虛擬機器](./media/monitor-virtual-machine-changes-event-grid-logic-app/monitor-virtual-machine-event-grid-logic-app-overview.png)

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立可從 Event Grid 監視事件的邏輯應用程式。
> * 新增可特別檢查虛擬機器變更的條件。
> * 在虛擬機器變更時傳送電子郵件。

## <a name="prerequisites"></a>必要條件

* 來自 [Azure Logic Apps 所支援的任何電子郵件提供者](../connectors/apis-list.md)的電子郵件帳戶，例如 Office 365 Outlook、Outlook.com 或 Gmail，以傳送通知。 本教學課程是使用 Office 365 Outlook。

* [虛擬機器](https://azure.microsoft.com/services/virtual-machines)。 如果您尚未這樣做，請透過[建立 VM 教學課程](https://docs.microsoft.com/azure/virtual-machines/)來建立虛擬機器。 若要讓虛擬機器發佈事件，您[不需要執行任何其他作業](../event-grid/overview.md)。

## <a name="create-a-logic-app-that-monitors-events-from-an-event-grid"></a>建立可從 Event Grid 監視事件的邏輯應用程式

首先，建立邏輯應用程式，並新增可監視您虛擬機器資源群組的 Event Grid 觸發程序。 

1. 登入 [Azure 入口網站](https://portal.azure.com)。 

2. 從 Azure 主功能表左上角，選擇 [建立資源] > [企業整合] > [邏輯應用程式]。

   ![建立邏輯應用程式](./media/monitor-virtual-machine-changes-event-grid-logic-app/azure-portal-create-logic-app.png)

3. 使用下表中指定的設定來建立應用程式邏輯：

   ![提供邏輯應用程式詳細資料](./media/monitor-virtual-machine-changes-event-grid-logic-app/create-logic-app-for-event-grid.png)

   | 設定 | 建議的值 | 說明 | 
   | ------- | --------------- | ----------- | 
   | **名稱** | {your-logic-app-name} | 提供唯一的邏輯應用程式名稱。 | 
   | **訂用帳戶** | {your-Azure-subscription} | 在本教學課程中，針對所有服務選取相同的 Azure 訂用帳戶。 | 
   | **資源群組** | {your-Azure-resource-group} | 在本教學課程中，針對所有服務選取相同的 Azure 訂用帳戶。 | 
   | **位置** | {your-Azure-region} | 在本教學課程中，針對所有服務選取相同的區域。 | 
   | | | 

4. 當您準備就緒，選取 [釘選到儀表板]，然後選擇 [建立]。

   您現在已經為您的應用程式邏輯建立一項 Azure 資源。 
   在 Azure 部署您的邏輯應用程式之後，Logic Apps 設計工具會顯示常見模式的範本，以便您更快開始使用。

   > [!NOTE] 
   > 當您選取 [釘選到儀表板] 時，邏輯應用程式會自動在 Logic Apps 設計工具中開啟。 不然，您可以手動尋找並開啟您的邏輯應用程式。

5. 立即選擇邏輯應用程式範本。 在 [範本] 之下，選擇 [空白邏輯應用程式]，以便從頭建置邏輯應用程式。

   ![選擇 [邏輯應用程式] 範本](./media/monitor-virtual-machine-changes-event-grid-logic-app/choose-logic-app-template.png)

   Logic Apps 設計工具現在會顯示[連接器](../connectors/apis-list.md)和[觸發程序](../logic-apps/logic-apps-overview.md#logic-app-concepts)，以便您用來啟動邏輯應用程式，以及您可以新增在觸發程序之後以執行工作的動作。 觸發程序是可建立邏輯應用程式執行個體並啟動邏輯應用程式工作流程的事件。 
   邏輯應用程式需要觸發程序作為第一個項目。

6. 在搜尋方塊中，輸入 "event grid" 作為篩選條件。 選取此觸發程序：**Azure Event Grid - 在資源事件上**

   ![選取此觸發程序：「Azure Event Grid - 在資源事件上」](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-event-grid-trigger.png)

7. 出現提示時，使用您的 Azure 認證登入 Azure Event Grid。

   ![利用您的 Azure 認證登入](./media/monitor-virtual-machine-changes-event-grid-logic-app/sign-in-event-grid.png)

   > [!NOTE]
   > 如果您已使用個人 Microsoft 帳戶登入，例如 @outlook.com 或 @hotmail.com，Event Grid 觸發程序可能無法正確顯示。 如需因應措施，請選擇[與服務主體連線](../azure-resource-manager/resource-group-create-service-principal-portal.md)，或以您與 Azure 訂用帳戶相關聯之 Azure Active Directory 的成員身分進行驗證，例如 user-name@emailoutlook.onmicrosoft.com。

8. 現在以邏輯應用程式訂閱發行者事件。 提供如下表中所指定的事件訂用帳戶之詳細資料：

   ![提供事件訂閱的詳細資料](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-event-grid-trigger-details-generic.png)

   | 設定 | 建議的值 | 說明 | 
   | ------- | --------------- | ----------- | 
   | **訂用帳戶** | {virtual-machine-Azure-subscription} | 選取事件發行者的 Azure 訂用帳戶。 在本教學課程中，選取虛擬機器的 Azure 訂用帳戶。 | 
   | **資源類型** | Microsoft.Resources.resourceGroups | 選取事件發行者的資源類型。 在本教學課程中，選取指定的值，讓您的邏輯應用程式只監視資源群組。 | 
   | **資源名稱** | {virtual-machine-resource-group-name} | 選取發行者的資源名稱。 在此教學課程中，選取虛擬機器的資源群組名稱。 | 
   | 針對選擇性設定，選擇 [顯示進階選項]。 | {see descriptions} | * **前置詞篩選條件**：在此教學課程中，將此設定保留空白。 預設行為會比對所有的值。 不過，您可以指定前置詞字串作為篩選條件，例如，特定資源的路徑和參數。 <p>* **後置詞篩選條件**：在此教學課程中，將此設定保留空白。 預設行為會比對所有的值。 不過，您可以指定前置詞字串作為篩選條件，例如，副檔名 (如果只想要特定檔案類型)。<p>* **訂用帳戶名稱**：提供事件訂用帳戶的唯一名稱。 |
   | | | 

   當您完成時，Event Grid 觸發程序可能如此範例所示︰
   
   ![範例 Event Grid 觸發程序詳細資料](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-event-grid-trigger-details.png)

9. 儲存您的邏輯應用程式。 在設計工具的工具列上，選擇 [儲存]。 若要在邏輯應用程式中摺疊並隱藏動作的詳細資料，請選擇動作的標題列。

   ![儲存您的邏輯應用程式](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-event-grid-save.png)

   當您儲存應用程式邏輯與 Event Grid 觸發程序時，Azure 會自動針對您的邏輯應用程式建立所選取資源的事件訂用帳戶。 因此當資源將事件發佈到 Event Grid 時，該 Event Grid 會將事件自動推送至邏輯應用程式。 此事件會觸發邏輯應用程式，然後建立並執行您在後續步驟中定義之工作流程的執行個體。

邏輯應用程式現在已上架，並且會接聽來自 Event Grid 的事件，但是在您將動作新增至工作流程以前，不會執行任何動作。 

## <a name="add-a-condition-that-checks-for-virtual-machine-changes"></a>新增可檢查虛擬機器變更的條件

若只要在發生特定事件時執行邏輯應用程式工作流程，請新增可檢查虛擬機器「寫入」作業的條件。 當此條件為 true 時，應用程式邏輯會傳送電子郵件給您，其中包含已更新虛擬機器的詳細資料。

1. 在 Logic Apps 設計工具中，於 Event Grid 觸發程序之下，選擇 [新增步驟] > [新增條件]。

   ![將條件新增至邏輯應用程式](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-event-grid-add-condition-step.png)

   Logic Apps 設計工具會將空白條件新增至您的工作流程，包括根據條件為 true 或 false 所要遵循的動作路徑。

   ![空白條件](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-add-empty-condition.png)

2. 在 [條件] 方塊中，選擇 [在進階模式中編輯]。
輸入此運算式：

   `@equals(triggerBody()?['data']['operationName'], 'Microsoft.Compute/virtualMachines/write')`

   您的條件現在看起來就像下面這個範例︰

   ![空白條件](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-condition-expression.png)

   此運算式會針對 `operationName` 屬性為 `Microsoft.Compute/virtualMachines/write` 作業的 `data` 物件，檢查事件 `body`。 
   深入了解 [Event Grid 事件結構描述](../event-grid/event-schema.md)。

3. 若要提供條件的描述，請選擇條件圖形上的**省略符號** (**...**) 按鈕，然後選擇 [重新命名]。

   > [!NOTE] 
   > 本教學課程稍後的範例也會提供邏輯應用程式工作流程中的步驟描述。

4. 現在選擇 [在基本模式中編輯]，以便運算式會自動解析，如下所示：

   ![邏輯應用程式條件](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-condition-1.png)

5. 儲存您的邏輯應用程式。

## <a name="send-email-when-your-virtual-machine-changes"></a>在虛擬機器變更時傳送電子郵件

現在新增[動作](../logic-apps/logic-apps-overview.md#logic-app-concepts)，以在指定的條件為 true 時收到電子郵件。

1. 在條件的 [若為 true] 方塊中，選擇 [新增動作]。

   ![新增條件為 true 時的動作](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-condition-2.png)

2. 在搜尋方塊中，輸入 "email" 作為篩選條件。 根據您的電子郵件提供者，尋找並選取相符的連接器。 然後針對您的連接器選取 [傳送電子郵件] 動作。 例如︰ 

   * 對於 Azure 公司或學校帳戶，選取 Office 365 Outlook 連接器。 
   * 對於個人 Microsoft 帳戶，選取 Outlook.com 連接器。 
   * 對於 Gmail 帳戶，選取 Gmail 連接器。 

   我們將使用 Office 365 Outlook 連接器繼續操作。 
   如果您使用不同的提供者，步驟維持不變，但您的 UI 可能會有所不同。 

   ![選取 [傳送電子郵件] 動作](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-send-email.png)

3. 如果您還沒有電子郵件提供者的連線，請在系統要求您進行驗證時登入您的電子郵件帳戶。

4. 如下表中所指定，提供電子郵件的詳細資料：

   ![空白的電子郵件動作](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-empty-email-action.png)

   > [!TIP]
   > 若要從您工作流程的可用欄位中選取，請按一下編輯方塊，隨即開啟 [動態內容] 清單，或選擇 [新增動態內容]。 如需更多的欄位，在清單中的每個區段選擇 [更多資訊]。 若要關閉 [動態內容] 清單，請選擇 [新增動態內容]。

   | 設定 | 建議的值 | 說明 | 
   | ------- | --------------- | ----------- | 
   | **To** | {recipient-email-address} |輸入收件者的電子郵件地址。 為了測試用途，您可以使用自己的電子郵件地址。 | 
   | **主旨** | 更新的資源：**主旨**| 輸入電子郵件主旨的內容。 在本教學課程中，輸入建議的文字並選取事件的 [主旨] 欄位。 在這裡，您的電子郵件主旨包含更新資源 (虛擬機器) 的名稱。 | 
   | **內文** | 資源群組：**主題** <p>事件類型：**事件類型**<p>事件識別碼：**識別碼**<p>時間：**事件時間** | 輸入電子郵件內文的內容。 選取 [主題]、[事件類型]、[識別碼] 和 [事件時間] 欄位，使您的電子郵件包含資源群組名稱、事件類型、事件戳記，以及更新的事件識別碼。 <p>若要在內容中新增空白的行，請按 Shift + Enter。 | 
   | | | 

   > [!NOTE] 
   > 如果您選取代表陣列的欄位，設計工具會自動新增以參考該陣列的動作為主的 **For each** 迴圈。 如此一來，應用程式邏輯會在每個陣列項目上執行該動作。

   現在，您的電子郵件動作看起來可能就像下面這個範例︰

   ![選取要納入電子郵件中的輸出](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-send-email-details.png)

   完成的邏輯應用程式可能如此範例所示︰

   ![完成的邏輯應用程式](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-completed.png)

5. 儲存您的邏輯應用程式。 若要在邏輯應用程式中摺疊並隱藏每個動作的詳細資料，請選擇動作的標題列。

   ![儲存您的邏輯應用程式](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-event-grid-save-completed.png)

   邏輯應用程式現在已上限，但是在執行任何作業之前，等候您的虛擬機器變更。 
   若要立即測試應用程式邏輯，請繼續下一節。

## <a name="test-your-logic-app-workflow"></a>測試邏輯應用程式工作流程

1. 若要檢查邏輯應用程式會取得指定的事件，請更新您的虛擬機器。 

   例如，您可以在 Azure 入口網站中調整您的虛擬機器大小，或[使用 Azure PowerShell 調整您的 VM 大小](../virtual-machines/windows/resize-vm.md)。 

   一會兒之後，您應可取得電子郵件。 例如︰

   ![關於虛擬機器更新的電子郵件](./media/monitor-virtual-machine-changes-event-grid-logic-app/email.png)

2. 若要檢閱邏輯應用程式的執行和觸發歷程記錄，請在邏輯應用程式功能表上選擇 [概觀]。 若要檢視執行的更多詳細資料，請選擇執行的資料列。

   ![邏輯應用程式執行記錄](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-run-history.png)

3. 若要檢視每個步驟的輸入和輸出，請展開您需要檢閱的步驟。 此資訊可協助您診斷和偵錯應用程式邏輯中的問題。
 
   ![邏輯應用程式執行記錄詳細資料](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-run-history-details.png)

恭喜，您已建立並執行邏輯應用程式，該邏輯應用程式可透過 Event Grid 監視資源事件，以及在這些事件發生時監視電子郵件。 您也了解如何輕鬆地建立工作流程，以自動執行程序並整合系統與雲端服務。

您可以使用事件格線和邏輯應用程式來監視其他組態變更，例如：

* 虛擬機器取得角色型存取控制 (RBAC) 權限。
* 對網路介面 (NIC) 上的網路安全性群組 (NSG) 進行變更。
* 已新增或移除虛擬機器的磁碟。
* 已將公用 IP 位址指派給虛擬機器 NIC。

## <a name="clean-up-resources"></a>清除資源

本教學課程使用會資源並執行會產生 Azure 訂用帳戶費用的動作。 當您完成本教學課程和測試時，請務必停用或刪除您不想產生費用的任何資源。

* 若要停止執行邏輯應用程式但是不刪除您的工作，請停用您的應用程式。 在邏輯應用程式功能表上，選擇 [概觀]。 在工具列上，選擇 [刪除]。

  ![關閉應用程式邏輯](./media/monitor-virtual-machine-changes-event-grid-logic-app/turn-off-disable-logic-app.png)

  > [!TIP]
  > 如果您沒有看到邏輯應用程式功能表，請嘗試返回 Azure 儀表板，然後重新開啟邏輯應用程式。

* 若要永久刪除邏輯應用程式，請在邏輯應用程式的功能表上，選擇 [概觀]。 在工具列上，選擇 [刪除]。 確認您想要刪除邏輯應用程式，然後選擇 [刪除]。

## <a name="next-steps"></a>後續步驟

* [使用 Event Grid 建立和路由傳送自訂事件](../event-grid/custom-event-quickstart.md)
