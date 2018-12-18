---
title: Azure 計費和成本管理預算案例 | Microsoft Docs
description: 了解如何使用 Azure 自動化來根據特定預算閾值關閉 VM。
services: billing
documentationcenter: ''
author: Erikre
manager: dougeby
editor: ''
tags: billing
ms.assetid: db93f546-6b56-4b51-960d-1a5bf0274fc8
ms.service: billing
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: billing
ms.date: 7/25/2018
ms.author: erikre
ms.openlocfilehash: 19d8fbf82410136fd6a50a2088dd9b774dd6cb27
ms.sourcegitcommit: c2c64fc9c24a1f7bd7c6c91be4ba9d64b1543231
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39259206"
---
# <a name="manage-costs-with-azure-budgets"></a>使用 Azure 預算來管理成本

成本控制是為您在雲端的投資創造最大價值的關鍵。 在幾種案例中，成本可見性、報告和成本型協調流程，對於持續的業務營運而言非常重要。 [Azure 成本管理 API](https://docs.microsoft.com/rest/api/consumption/) \(英文\) 提供一組 API 來支援上述每一種情節。 API 能提供使用情況詳細資料，可讓您檢視更細微的執行個體層級成本。

預算經常被當作成本控制的一部分。 在 Azure 中可以設定預算範圍。 例如，您可以依訂用帳戶、資源群組或資源集合縮小預算檢視。 除了使用預算 API 於達到預算閾值時透過電子郵件通知您之外，您也可以使用 [Azure 監視器動作群組](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-action-groups)來在發生預算事件時，觸發一組已協調的動作。

對於執行非關鍵工作負載的客戶而言，其中一個常見的預算案例為想要針對某個預算進行管理，並同時在查看每月發票時取得可預測的成本。 這種案例需要針對作為 Azure 環境之一部分的資源進行成本型的協調流程。 在此案例中，會針對訂用帳戶設定每月 $1000 美元的預算。 此外，也會設定通知閾值以觸發幾個協調流程。 此案例一開始具有 80% 的成本閾值，這會停止資源群組 **Optional** 中的所有 VM。 然後，在達到 100% 的成本閾值之後，系統就會停止所有 VM 執行個體。
若要設定此案例，請依照本教學課程中各節所提供的步驟來完成下列動作。 

本教學課程所包含的這些動作可以讓您：

- 建立 Azure 自動化 Runbook，以使用 Webhook 來停止 VM。
- 建立會依預算閾值觸發，並使用正確參數呼叫 Runbook 的 Azure 邏輯應用程式。
- 建立 Azure 監視器 動作群組，並將其設定為在達到預算閾值時觸發 Azure 邏輯應用程式。
- 以所需閾值建立 Azure 預算，並將它連接到動作群組。

## <a name="create-an-azure-automation-runbook"></a>建立 Azure 自動化 Runbook

[Azure 自動化](https://docs.microsoft.com/azure/automation/automation-intro)是一個服務，可讓您編寫大部分資源管理工作的指令碼，並以排程或在需要時執行那些工作。 在這個案例中，您將會建立 [Azure 自動化 Runbook](https://docs.microsoft.com/azure/automation/automation-runbook-types)，並將它用來停止 VM。 您將使用來自[資源庫](https://docs.microsoft.com/azure/automation/automation-runbook-gallery)的 [Stop Azure V2 VM](https://gallery.technet.microsoft.com/scriptcenter/Stop-Azure-ARM-VMs-1ba96d5b) \(英文\) 圖形化 Runbook 來建置這個案例。 透過將此 Runbook 匯入您的 Azure 帳戶並發佈它，您將能在達到預算閾值後停止 VM。 

### <a name="create-an-azure-automation-account"></a>建立 Azure 自動化帳戶

1.  使用您的 Azure 帳戶認證登入 [Azure 入口網站](https://portal.azure.com/)。
2.  按一下 Azure 左上角的 [建立資源] 按鈕。
3.  選取 [管理工具] > [自動化]。
    > [!NOTE]
    > 如果您沒有 Azure 帳戶，可以建立一個[免費帳戶](https://azure.microsoft.com/free/)。
4.  輸入您的帳戶資訊。 針對 [建立 Azure 執行身分帳戶]，選擇 [是] 以自動啟用相關設定來簡化 Azure 驗證。
5.  完成後，按一下 [建立] 可啟動自動化帳戶部署。

### <a name="import-the-stop-azure-v2-vms-runbook"></a>匯入 Stop Azure V2 VM Runbook

透過使用 [Azure 自動化 Runbook](https://docs.microsoft.com/azure/automation/automation-runbook-types)，從資源庫匯入 [Stop Azure V2 VM](https://gallery.technet.microsoft.com/scriptcenter/Stop-Azure-ARM-VMs-1ba96d5b) \(英文\) 圖形化 Runbook。

1.  使用您的 Azure 帳戶認證登入 [Azure 入口網站](https://portal.azure.com/)。
2.  選取 [所有服務] > [自動化帳戶] 來開啟您的自動化帳戶。 然後選取您的自動化帳戶。
3.  按一下 [程序自動化] 區段中的 [Runbook 資源庫]。
4.  將 [資源庫來源] 設定為 [指令碼中心]，然後選取 [確定]。
5.  在 Azure 入口網站內找出並選取 [Stop Azure V2 VM](https://gallery.technet.microsoft.com/scriptcenter/Stop-Azure-ARM-VMs-1ba96d5b) \(英文\) 資源庫項目。
6.  按一下 [匯入] 按鈕來顯示 [匯入] 刀鋒視窗，然後選取 [確定]。 Runbook 概觀刀鋒視窗隨即顯示。
7.  Runbook 完成匯入過程之後，請選取 [編輯] 以顯示圖形化 Runbook 編輯器和發佈選項。
    
    ![Azure - [編輯圖形化 Runbook]](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-01.png)
8.  按一下 [發佈] 按鈕來發佈 Runbook，然後在系統提示時選取 [是]。 當您發佈 Runbook 時，您會以草稿版本覆寫任何現有的已發佈版本。 在本範例的情況中，因為您剛剛才建立 Runbook，所以還沒有已發佈的版本。 

    如需發佈 Runbook 的詳細資訊，請參閱[建立圖形化 Runbook](https://docs.microsoft.com/azure/automation/automation-first-runbook-graphical)。

## <a name="create-webhooks-for-the-runbook"></a>建立 Runbook 的 Webhook

透過使用 [Stop Azure V2 VM](https://gallery.technet.microsoft.com/scriptcenter/Stop-Azure-ARM-VMs-1ba96d5b) \(英文\) 圖形化 Runbook，您將會建立兩個 Webhook，以透過單一 HTTP 要求在 Azure 自動化中啟動 Runbook。 當達到 80% 預算閾值的時候，第一個 Webhook 將會以資源群組名稱作為參數來呼叫 Runbook，以允許系統停止選擇性的 VM。 然後，第二個 Webhook 會在不搭配任何參數 (於 100%) 的情況下叫用 Runbook，這會停止所有剩下的 VM 執行個體。

1.  從 [Azure 入口網站](https://portal.azure.com/)中的 [Runbook] 頁面，按一下會顯示 Runbook 概觀刀鋒視窗的 [StopAzureV2Vm] Runbook。 
2.  按一下頁面頂端的 [Webhook] 來開啟 [新增 Webhook] 刀鋒視窗。
3.  按一下 [建立新的 Webhook] 來開啟 [建立新的 Webhook ] 刀鋒視窗。
4.  將 Webhook 的 [名稱] 設定成 **Optional**。 [Enabled] 屬性必須為 [Yes]。 [Expires] 值不需要變更。 如需 Webhook 屬性的詳細資訊，請參閱 [Webhook 的詳細資料](https://docs.microsoft.com/azure/automation/automation-webhooks#details-of-a-webhook)。
5.  在 URL 值的旁邊，按一下複製圖示來複製 Webhook 的 URL。 
    > [!IMPORTANT]
    > 將名為 **Optional** 之 Webhook 的 URL 保存在安全的地方。 您稍後會在本教學課程中使用這個 URL。 為了安全起見，當您建立 Webhook 之後，就無法再次查看或擷取該 URL。
6.  按一下 [確定] 以建立新的 Webhook。
7.  按一下 [設定參數與回合設定] 以檢視 Runbook 的參數值。 
    > [!NOTE]
    > 如果 Runbook 有強制參數，除非提供值，否則您無法建立 Webhook。
8.  按一下 [確定] 以接受 Webhook 參數值。
9.  按一下 [建立] 以建立 Webhook。
10. 接下來，請依照上述步驟建立名為 **Complete** 的第二個 Webhook。 
    > [!IMPORTANT]
    > 請務必儲存這兩個 Webhook URL，以用於本教學課程稍後的步驟。 為了安全起見，當您建立 Webhook 之後，就無法再次查看或擷取該 URL。

您現在應該有兩個已設定的 Webhook，可透過您已儲存的 URL 來使用。 

![Webhook - [Optional] 和 [Complete]](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-02.png)

您現在已完成 Azure 自動化的設定。 若要測試 Webhook，您可以使用簡單的 Postman 測試來驗證 Webhook 是否有效。 接下來，您必須針對協調流程建立邏輯應用程式。

## <a name="create-an-azure-logic-app-for-orchestration"></a>針對協調流程建立 Azure 邏輯應用程式

邏輯應用程式可協助您以工作流程的形式對程序進行建置、排程及自動化，以跨企業或組織整合應用程式、資料、系統及服務。 在此案例中，您將建立之[邏輯應用程式](https://docs.microsoft.com/azure/logic-apps/)的功能，將不僅僅只有呼叫您所建立的自動化 Webhook 而己。 

預算可以設定成在符合指定閾值的情況下觸發通知。 您可以提供多個要進行通知的閾值，然後邏輯應用程式將會示範可讓您根據不同閾值來執行不同動作的能力。 在此範例中，您將設定會讓您接收到幾個通知的案例，第一個通知會在達到 80% 的預算時傳送，而第二個通知則會在達到 100% 的預算時傳送。 邏輯應用程式將會被用來關閉資源群組中的所有 VM。 首先，在 80% 的時候會達到 **Optional** 閾值，接著在達到第二個閾值時，訂用帳戶中的所有 VM 都會被關閉。

邏輯應用程式可讓您針對 HTTP 觸發程序提供範例結構描述，但您必須設定 **Content-type** 標頭。 因為動作群組沒有適用於 Webhook 的自訂標頭，您必須在個別的步驟中對承載進行剖析。 您將會使用 **Parse** 動作，並為它提供範例承載。

### <a name="create-the-logic-app"></a>建立邏輯應用程式

邏輯應用程式會執行數個動作。 下列清單提供邏輯應用程式將會執行的高階動作集合：
- 辨識接收到 HTTP 要求的時機
- 剖析已傳遞的 JSON 資料，以判斷已經達到的閾值
- 使用條件式陳述式來檢查閾值額度是否已達到預算範圍的 80% 或更高，但沒有超過或等於 100%。
    - 如果已達到這個閾值額度，便使用名為 **Optional** 的 Webhook 來傳送 HTTP POST。 這個動作將關閉 "Optional" 群組中的 VM。
- 使用條件陳述式來檢查閾值額度是否已達到或超過預算值的 100%。
    - 如果已達到這個閾值額度，便使用名為 **Complete** 的 Webhook 來傳送 HTTP POST。 這個動作將關閉所有剩下的 VM。

建立將執行上述步驟的邏輯應用程式，必須執行下列步驟：

1.  在 [Azure 入口網站](https://portal.azure.com/)中，選取 [建立資源] > [整合] > [邏輯應用程式]。
    
    ![Azure - 選取 [邏輯應用程式] 資源](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-03.png)
2.  在 [建立邏輯應用程式] 刀鋒視窗中，提供建立邏輯應用程式所需的詳細資料，選取 [釘選到儀表板]，然後按一下 [建立]。 
    
    ![Azure - 建立邏輯應用程式](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-03a.png)

在 Azure 部署您的邏輯應用程式之後，[Logic Apps 設計工具] 隨即開啟，並顯示含有簡介影片和常用觸發程序的刀鋒視窗。 

### <a name="add-a-trigger"></a>新增觸發程序

每個邏輯應用程式都必須使用觸發程序啟動，而該觸發程序會在特定事件發生或符合特定條件時引發。 每次引發觸發程序時，Logic Apps 引擎會建立邏輯應用程式執行個體，啟動並執行您的工作流程。 「動作」是指在觸發程序之後發生的所有步驟。 

1.  在 [Logic Apps 設計工具] 刀鋒視窗的 [範本] 中，選擇 [空白邏輯應用程式]。
2.  在 [Logic Apps 設計工具] 搜尋方塊中輸入「http 要求」來新增[觸發程序](https://docs.microsoft.com/azure/logic-apps/logic-apps-overview#logic-app-concepts)，以尋找並選取名為 [要求 – 接收到 HTTP 要求時] 的觸發程序。
    
    ![Azure - 邏輯應用程式 - Http 觸發程序](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-04.png) 
3.  選取 [新增步驟] > [新增動作]。 
    
    ![Azure - [新增步驟] - [新增動作]](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-05.png) 
4.  在 [Logic Apps 設計工具] 搜尋方塊中搜尋「剖析 JSON」，以尋找並選取 [資料作業 - 剖析 JSON] [動作](https://docs.microsoft.com/azure/logic-apps/logic-apps-overview#logic-app-concepts)。 
    
    ![Azure - 邏輯應用程式 - 新增剖析 JSON 動作](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-06.png) 
5.  輸入「承載」作為剖析 JSON 承載的 [內容] 名稱。
6.  選取 [剖析 JSON] 方塊中的 [請使用範例承載產生結構描述] 選項。
    
    ![Azure - 邏輯應用程式 - 使用範例 JSON 資料來產生結構描述](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-07.png) 
7.  將下列 JSON 範例承載貼到文字方塊中：`{"schemaId":"AIP Budget Notification","data":{"SubscriptionName":"CCM - Microsoft Azure Enterprise - 1","SubscriptionId":"<GUID>","SpendingAmount":"100","BudgetStartDate":"6/1/2018","Budget":"50","Unit":"USD","BudgetCreator":"email@contoso.com","BudgetName":"BudgetName","BudgetType":"Cost","ResourceGroup":"","NotificationThresholdAmount":"0.8"}}`
    
    文字方塊看起來應如下所示：
        
    ![Azure - 邏輯應用程式 - 範例 JSON 承載](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-08.png) 
8.  按一下 [完成]。

### <a name="add-the-first-conditional-action"></a>新增第一個條件式動作

使用條件式陳述式來檢查閾值額度是否已達到預算範圍的 80% 或更高，但沒有超過或等於 100%。 如果已達到這個閾值額度，便使用名為 **Optional** 的 Webhook 來傳送 HTTP POST。 這個動作將關閉 **Optional** 群組中的 VM。

1.  選取 [新增步驟] > [新增條件]。
    
    ![Azure - 邏輯應用程式 - [新增條件]](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-09.png) 
2.  在 [條件] 方塊中，按一下包含 [選擇值] 的文字方塊，以顯示可用值的清單。
    
    ![Azure - 邏輯應用程式 - [條件] 方塊](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-10.png) 
        
3.  按一下清單頂端的 [運算式]，然後在運算式編輯器中輸入下列運算式：`float()`
    
    ![Azure - 邏輯應用程式 - 浮點運算式](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-11.png) 
        
4.  選取 [動態內容]，將游標放在括號 () 裡面，然後從清單中選取 [NotificationThresholdAmount] 來填入完整的運算式。 
    
    運算式會是下列之一：<br>
    `float(body('Parse_JSON')?['data']?['NotificationThresholdAmount'])`
    
5.  選取 [確定] 來設定運算式。 
6.  在 [條件] 的下拉式方塊中，選取 [大於或等於]。
7.  在 [條件] 的 [選擇值] 方塊中，輸入 `.8`。
    
    ![Azure - 邏輯應用程式 - 具有值的浮點運算式](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-12.png) 
        
8.  按一下 [條件] 方塊內的 [新增] > [新增資料列] 來新增條件的額外部分。
9.  在 [條件] 方塊中，按一下包含 [選擇值] 的文字方塊。
10. 按一下清單頂端的 [運算式]，然後在運算式編輯器中輸入下列運算式：`float()`
11. 選取 [動態內容]，將游標放在括號 () 裡面，然後從清單中選取 [NotificationThresholdAmount] 來填入完整的運算式。 
12. 選取 [確定] 來設定運算式。 
13. 在 [條件] 的下拉式方塊中，選取 [小於]。
14. 在條件的 [選擇值] 方塊中，輸入 `1`。
    
    ![Azure - 邏輯應用程式 - 具有值的浮點運算式](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-13.png) 
        
15. 在 [若為 true] 方塊中，選取 [新增動作]。 您將新增一個會關閉選擇性 VM 的 HTTP POST 動作。
    
    ![Azure - 邏輯應用程式 - [新增動作]](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-14.png) 
        
16. 輸入 **HTTP** 來搜尋該 HTTP 動作，然後選取 [HTTP – HTTP] 動作。
    
    ![Azure - 邏輯應用程式 - 新增 HTTP 動作](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-15.png) 
        
17. 選取 [Post] 作為 [方法] 值。
18. 輸入您稍早於本教學課程中所建立之 **Optional** Webhook 的 URL，作為 [Uri] 值。
    
    ![Azure - 邏輯應用程式 - HTTP 動作 URI](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-16.png) 
        
19. 在 [若為 true] 方塊中，選取 [新增動作]。 您將新增電子郵件動作來傳送電子郵件，以通知收件者選擇性 VM 已被關閉。
20. 搜尋「傳送電子郵件」，然後根據您所使用的電子郵件服務，選取其中一個*傳送電子郵件*動作。
        
    ![Azure -邏輯應用程式 - 傳送電子郵件動作](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-17.png) 
    
    對於個人 Microsoft 帳戶，選取 [Outlook.com]。 對於 Azure 公司或學校帳戶，選取 [Office 365 Outlook]。 如果您還沒有連線，系統會要求您登入您的電子郵件帳戶。 Logic Apps 會建立與電子郵件帳戶的連線。
        
    您必須允許邏輯應用程式存取您的電子郵件資訊。
    
    ![Azure - 邏輯應用程式 - 存取通知](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-18.png) 
        
21. 為通知收件者選擇性 VM 已被關閉的電子郵件，新增 [收件者]、[主旨] 和 [本文]。 使用 **BudgetName** 和 **NotificationThresholdAmount** 動態內容來填入主旨和本文欄位。
    
    ![Azure - 邏輯應用程式 - 電子郵件詳細資料](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-19.png) 
 
### <a name="add-the-second-conditional-action"></a>新增第二個條件式動作

使用條件陳述式來檢查閾值額度是否已達到或超過預算值的 100%。 如果已達到這個閾值額度，便使用名為 **Complete** 的 Webhook 來傳送 HTTP POST。 這個動作將關閉所有剩下的 VM。

1.  選取 [新增步驟] > [新增條件]。
    
    ![Azure - 邏輯應用程式 - 新增動作](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-20.png) 
        
2.  在 [條件] 方塊中，按一下包含 [選擇值] 的文字方塊，以顯示可用值的清單。
3.  按一下清單頂端的 [運算式]，然後在運算式編輯器中輸入下列運算式：`float()`
4.  選取 [動態內容]，將游標放在括號 () 裡面，然後從清單中選取 [NotificationThresholdAmount] 來填入完整的運算式。 
    
    運算式會是下列之一：<br>
    `float(body('Parse_JSON')?['data']?['NotificationThresholdAmount'])`
        
5.  選取 [確定] 來設定運算式。 
6.  在 [條件] 下拉式方塊中，選取 [大於或等於]。
7.  在 [條件] 的 [選擇值] 方塊中，輸入 `1`。
    
    ![Azure - 邏輯應用程式 - 設定條件值](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-21.png) 
        
8.  在 [若為 true] 方塊中，選取 [新增動作]。 您將新增一個會關閉所有剩下 VM 的 HTTP POST 動作。
    
    ![Azure - 邏輯應用程式 - [新增動作]](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-22.png) 
    
9.  輸入 **HTTP** 來搜尋該 HTTP 動作，然後選取 [HTTP – HTTP] 動作。
10. 選取 [Post] 作為 [方法] 值。
11. 輸入您稍早於本教學課程中所建立之 **Complete** Webhook 的 URL，作為 [Uri] 值。
    
    ![Azure - 邏輯應用程式 - [新增動作]](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-23.png) 
        
12. 在 [若為 true] 方塊中，選取 [新增動作]。 您將新增電子郵件動作來傳送電子郵件，以通知收件者剩下的 VM 已被關閉。
13. 搜尋「傳送電子郵件」，然後根據您所使用的電子郵件服務，選取其中一個*傳送電子郵件*動作。
14. 為通知收件者選擇性 VM 已被關閉的電子郵件，新增 [收件者]、[主旨] 和 [本文]。 使用 **BudgetName** 和 **NotificationThresholdAmount** 動態內容來填入主旨和本文欄位。
    
    ![Azure -邏輯應用程式 - 傳送電子郵件詳細資料](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-24.png) 
        
15. 按一下 [Logic Apps 設計工具] 刀鋒視窗上方的 [儲存]。

### <a name="logic-app-summary"></a>邏輯應用程式摘要

當您完成時，邏輯應用程式看起來應該如下所示。 在不需要任何以閾值為基礎之協調流程的最基本案例中，您可以從 [監視器] 直接呼叫自動化指令碼，並略過 [邏輯應用程式] 步驟。

   ![Azure - 邏輯應用程式 - 完整檢視](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-25.png) 
 
當您儲存邏輯應用程式時，系統會產生一個您可以呼叫的 URL。 您會在本教學課程的下一節中使用這個 URL。 

## <a name="create-an-azure-monitor-action-group"></a>建立 Azure 監視器動作群組

動作群組是您所定義之通知喜好設定的集合。 觸發警示時，特定的動作群組將能透過通知接收到警示。 Azure 警示會根據特定條件主動發出通知，並提供採取動作的機會。 警示可以使用來自各種來源 (包括計量和記錄) 的資料。

動作群組是您唯一會與預算進行整合的端點。 您可以在數個通道中設定通知，不過在本案例中，您會將會專注在您稍早於本教學課程中建立的邏輯應用程式。 

### <a name="create-an-action-group-in-azure-monitor"></a>在 Azure 監視器中建立動作群組

當您建立動作群組時，您會指向您稍早於本教學課程中建立的邏輯應用程式。 

1.  如果您尚未登入 [Azure 入口網站](https://portal.azure.com/)，請登入並選取 [所有服務] > [監視]。 
2.  從 [設定] 區段中，選取 [動作群組]。
3.  從 [動作群組] 刀鋒視窗中，選取 [新增動作群組]。
4.  新增並確認下列項目：
    - 動作群組名稱
    - 簡短名稱
    - 訂用帳戶
    - 資源群組
    
    ![Azure - 邏輯應用程式 - [新增動作群組]](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-26.png) 
        
5.  在 [新增動作群組] 窗格內，新增 LogicApp 動作。 將該動作命名為 **Budget-BudgetLA**。 在 [邏輯應用程式] 窗格中，選取 [訂用帳戶] 及 [資源群組]。 然後，選取您稍早於本教學課程中建立的**邏輯應用程式**。
6.  按一下 [確定] 來設定邏輯應用程式。 然後，選取 [新增動作群組] 窗格中的 [確定]以建立動作群組。
    
您已完成有效協調預算所需的所有支援元件。 現在您只需要建立預算，並將它設定為使用您所建立的動作群組。

## <a name="create-the-azure-budget"></a>建立 Azure 預算 

預算目前在 Azure 中並沒有提供入口網站體驗。 不過，您可以呼叫 REST API、Powershell Cmdlet，或是使用 CLI。 下列程序會使用 REST API。 在呼叫 REST API 之前，您將會需要授權權杖。 若要建立授權權杖，您可以使用 [ARMClient](https://github.com/projectkudu/ARMClient) \(英文\) 專案。 **ARMClient** 可讓您將自己驗證至 Azure Resource Manager，並取得呼叫 API 的權杖。 

### <a name="create-an-authentication-token"></a>建立驗證權杖

1.  瀏覽至 GitHub 上的 [ARMClient](https://github.com/projectkudu/ARMClient) \(英文\) 專案。
2.  複製存放庫以取得本機複本。
3.  在 Visual Studio 中開啟專案並加以建置。
4.  建置成功之後，可執行檔應該會位於 *\bin\debug* 資料夾中。
5.  執行 ARMClient。 開啟命令提示字元，並從專案根資料夾瀏覽至 *\bin\debug*。
6.  若要登入並進行驗證，請在命令提示字元中輸入下列命令：<br>
    `ARMClient login prod`
7.  從輸出中複製 **subscription guid**。
8.  若要將授權權杖複製到剪貼簿，請在命令提示字元中輸入下列命令，但請務必使用上述步驟中已複製的訂用帳戶識別碼： <br>
    `ARMClient token <subscription GUID from previous step>`
    
    完成上述步驟後，您將會看到以下內容：<br>
    **權杖已成功複製到剪貼簿。**
9.  儲存權杖以用於本教學課程下一節的步驟。

### <a name="create-the-budget"></a>建立預算

接下來，您將設定 **Postman**，以透過呼叫 Azure 使用量 REST API 來建立預算。 Postman 為 API 開發環境。 您會將環境和集合檔案匯入 Postman。 集合包含會呼叫 Azure 使用量 REST API 之 HTTP 要求的分組定義。 環境檔案包含集合所使用的變數。

1.  下載並安裝 [Postman REST 用戶端](https://www.getpostman.com/) \(英文\) 來執行 REST API。
2.  在 Postman 中，建立新的要求。
    
    ![Postman - 建立新的要求](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-27.png) 
        
3.  將新的要求儲存為集合，使新要求上面不具有任何內容。
    
    ![Postman - 儲存新的要求](./media/billing-cost-management-budget-scenario/billing-cost-management-budget-scenario-28.png) 
        
4.  將要求從 `Get` 變更至 `Put` 動作。
5.  將以下 URL 的 `{subscriptionId}` 取代為您於本教學課程上一節中所使用的**訂用帳戶識別碼**。 此外，修改 URL 來包含 "SampleBudget" 作為 `{budgetName}` 的值：`https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Consumption/budgets/{budgetName}?api-version=2018-03-31`
6.  選取 Postman 內的 [Headers] \(標頭\) 索引標籤。
7.  新增名為 "Authorization" 的新 [Key] \(金鑰\)。
8.  將 [Value] \(值\) 設定成於上一節結尾利用 ArmClient 所建立的權杖。 
9.  選取 Postman 內的 [Body] \(本文\) 索引標籤。 
10. 選取 [raw] \(原始\) 按鈕選項。
11. 在文字方塊中，貼上下列範例預算定義，不過您必須將 **subscriptionid**、**budgetname** 和 **actiongroupname** 參數取代為您的訂用帳戶識別碼、您預算的唯一名稱，以及您在 URL 和要求本文中所建立的動作群組名稱：
    
    ```
        {
            "properties": {
                "category": "Cost",
                "amount": 100.00,
                "timeGrain": "Monthly",
                "timePeriod": {
                "startDate": "2018-06-01T00:00:00Z",
                "endDate": "2018-10-31T00:00:00Z"
                },
                "filters": {
                },
            "notifications": {
                "Actual_GreaterThan_80_Percent": {
                    "enabled": true,
                    "operator": "GreaterThan",
                    "threshold": 80,
                    "contactEmails": [
                    ],
                    "contactRoles": [
                    ],
                    "contactGroups": [
                    "/subscriptions/{subscriptionid}/resourceGroups/{resourcegroupname}/providers/microsoft.insights/actionGroups/{actiongroupname}
                    ]
                }, 
            "Actual_EqualTo_100_Percent": {
                    "operator": "EqualTo",
                    "threshold": 100,
                    "contactGroups": [
                "/subscriptions/{subscriptionid}/resourceGroups/{resourcegroupname}/providers/microsoft.insights/actionGroups/{actiongroupname}"
                    ]
                }
            }
        }
    ```
12. 按 [Send] \(傳送\) 來傳送要求。

現在，呼叫[預算 API](https://docs.microsoft.com/rest/api/consumption/budgets)\(英文\) 的所有元件皆已準備完畢。 預算 API 參考針對特定要求有提供其他詳細資料，包括下列項目：
    - **budgetName** - 支援多個預算。  預算名稱必須是唯一的。
    - **category** - 必須是 **Cost** 或 **Usage**。 API 同時支援成本和使用量預算。
    - **timeGrain** - 每月、每季或每年的預算。 系統會於期間結束時重設額度。
    - **filters** - 篩選可讓您將預算縮小到選取範圍內的特定資源集合。 例如，篩選可以是訂用帳戶層級預算的資源群組集合。
    - **notifications** – 決定通知詳細資料和閾值。 您可以設定多個閾值，並提供電子郵件地址或動作群組來接收通知。

## <a name="summary"></a>總結

透過遵循本教學課程的步驟，您已了解：
- 如何建立 Azure 自動化 Runbook 來停止 VM。
- 如何建立會依預算閾值觸發，並使用正確參數呼叫 Runebook 的 Azure 邏輯應用程式。
- 如何建立 Azure 監視器 動作群組，並將其設定為在達到預算閾值時觸發 Azure 邏輯應用程式。
- 如何以所需閾值建立 Azure 預算，並將它連接到動作群組。

您現在已具有適用於您訂用帳戶且功能完整的預算，會在您達到已設定的預算閾值時關閉您的 VM。 

## <a name="next-steps"></a>後續步驟

- 如需 Azure 計費案例的詳細資訊，請參閱[計費和成本管理自動化案例](billing-cost-management-automation-scenarios.md)。
