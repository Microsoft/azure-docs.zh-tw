---
title: 跨訂用帳戶將 Azure 活動記錄收集到 Log Analytics | Microsoft Docs
description: 使用事件中樞和 Logic Apps 收集來自 Azure 活動記錄的資料，並將它傳送至不同租用戶中的 Azure Log Analytics 工作區。
services: log-analytics, logic-apps, event-hubs
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: richrund; bwren
ms.component: na
ms.openlocfilehash: c2bb802213d903290a0168623d7e6a302ba0e324
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37127436"
---
# <a name="collect-azure-activity-logs-into-log-analytics-across-subscriptions"></a>跨訂用帳戶將 Azure 活動記錄收集到 Log Analytics

本文會逐步執行方法，以使用適用於 Logic Apps 的 Azure Log Analytics 資料收集器連接器，將 Azure 活動記錄收集到 Log Analytics 工作區。 當您需要將記錄傳送至不同 Azure Active Directory 中的工作區時，請使用本文中的程序。 例如，如果您是受控服務提供者，則可能會想要收集客戶訂用帳戶的活動記錄，並將其儲存在您專屬訂用帳戶的 Log Analytics 工作區中。

如果 Log Analytics 工作區位在相同的 Azure 訂用帳戶中，或者在不同的訂用帳戶但在相同的 Azure Active Directory 中，請使用 [Azure 活動記錄解決方案](../log-analytics/log-analytics-activity.md)中的步驟來收集 Azure 活動記錄。

## <a name="overview"></a>概觀

此案例中所使用的策略是讓 Azure 活動記錄將事件傳送至[事件中樞](../event-hubs/event-hubs-what-is-event-hubs.md)其中，[邏輯應用程式](../logic-apps/logic-apps-overview.md)會將它們傳送至 Log Analytics 工作區。 

![從活動記錄到 Log Analytics 之資料流程的影像](media/log-analytics-activity-logs-subscriptions/data-flow-overview.png)

此方法的優點包含：
- 將 Azure 活動記錄串流至事件中樞之後的低延遲。  接著會觸發邏輯應用程式，然後將資料公佈到 Log Analytics。 
- 需要最少的程式碼，而且沒有任何要部署的伺服器基礎結構。

本文逐步引導您如何：
1. 建立事件中樞。 
2. 使用 Azure 活動記錄匯出設定檔，將活動記錄匯出至事件中樞。
3. 建立邏輯應用程式，以從事件中樞讀取事件，並將事件傳送至 Log Analytics。

## <a name="requirements"></a>需求
以下是此案例中所使用 Azure 資源的需求。

- 事件中樞命名空間不需要與訂用帳戶發出記錄位在相同的訂用帳戶中。 進行此設定的使用者必須具有這兩個訂用帳戶的適當存取權。 如果相同的 Azure Active Directory 中有多個訂用帳戶，則可以將所有訂用帳戶的活動記錄傳送至單一事件中樞。
- 邏輯應用程式可以在事件中樞的不同訂用帳戶中，而且不需要在相同的 Azure Active Directory 中。 邏輯應用程式會使用事件中樞的共用存取金鑰，以從事件中樞進行讀取。
- Log Analytics 工作區可以位在邏輯應用程式的不同訂用帳戶和 Azure Active Directory 中，但為求簡單，建議將它們放在相同的訂用帳戶中。 邏輯應用程式會使用 Log Analytics 工作區識別碼和金鑰來傳送至 Log Analytics。



## <a name="step-1---create-an-event-hub"></a>步驟 1 - 建立事件中樞

<!-- Follow the steps in [how to create an Event Hubs namespace and Event Hub](../event-hubs/event-hubs-create.md) to create your event hub. -->

1. 在 Azure 入口網站中，選取 [建立資源] > [物聯網] > [事件中樞]。

   ![Marketplace 新的事件中樞](media/log-analytics-activity-logs-subscriptions/marketplace-new-event-hub.png)

3. 在 [建立命名空間] 下方，輸入新的命名空間，或選取現有命名空間。 系統會立即檢查此名稱是否可用。

   ![建立事件中樞對話方塊的影像](media/log-analytics-activity-logs-subscriptions/create-event-hub1.png)

4. 選擇新資源的定價層 (基本或標準)、Azure 訂用帳戶、資源群組和位置。  按一下 [建立]  來建立命名空間。 您可能必須等候幾分鐘，讓系統完整佈建資源。
6. 從該清單中，按一下您剛建立的命名空間。
7. 選取 [共用存取原則]，然後按一下 [RootManageSharedAccessKey]。

   ![事件中樞共用存取原則的影像](media/log-analytics-activity-logs-subscriptions/create-event-hub7.png)
   
8. 按一下複製按鈕，將 **RootManageSharedAccessKey** 連接字串複製到剪貼簿。 

   ![事件中樞共用存取金鑰的影像](media/log-analytics-activity-logs-subscriptions/create-event-hub8.png)

9. 在暫存位置中 (例如 [記事本])，複製事件中樞名稱以及主要或次要事件中樞連接字串。 邏輯應用程式需要這些值。  針對事件中樞連接字串，您可以使用 **RootManageSharedAccessKey** 連接字串，或建立不同的連接字串。  您使用的連接字串開頭必須是 `Endpoint=sb://`，並且適用於具有 [管理] 存取原則的原則。


## <a name="step-2---export-activity-logs-to-event-hub"></a>步驟 2 - 將活動記錄匯出事件中樞

若要啟用活動記錄的串流，您可以挑選事件中樞命名空間以及該命名空間的共用存取原則。 第一個新的活動記錄事件發生時，會在該命名空間中建立事件中樞。 

您可以使用與訂用帳戶發出記錄位在不同訂用帳戶中的事件中樞命名空間，不過，訂用帳戶必須位在相同的 Azure Active Directory 中。 進行此設定的使用者必須具有適當的 RBAC 才能存取這兩個訂用帳戶。 

1. 在 Azure 入口網站中，選取 [監視] > [活動記錄]。
3. 按一下頁面頂端的 [匯出] 按鈕。

   ![導覽中 Azure 監視器的影像](media/log-analytics-activity-logs-subscriptions/activity-log-blade.png)

4. 選取要從中匯出的 [訂用帳戶]，然後按一下 [區域] 下拉式清單中的 [全選]，以選取所有區域中資源的事件。 按一下 [Export to an event hub] \(匯出至事件中樞) 核取方塊。
7. 按一下 [服務匯流排命名空間]，然後選取含事件中樞、[事件中樞命名空間] 和 [事件中樞原則名稱] 的 [訂用帳戶]。

    ![匯出至事件中樞頁面的影像](media/log-analytics-activity-logs-subscriptions/export-activity-log2.png)

11. 按一下 [確定]，然後按一下 [儲存]，來儲存這些設定。 您的訂用帳戶時會立即套用設定。

<!-- Follow the steps in [stream the Azure Activity Log to Event Hubs](../monitoring-and-diagnostics/monitoring-stream-activity-logs-event-hubs.md) to configure a log profile that writes activity logs to an event hub. -->

## <a name="step-3---create-logic-app"></a>步驟 3 - 建立邏輯應用程式

活動記錄寫入至事件中樞之後，請建立邏輯應用程式，以從事件中樞收集記錄檔，並將它們寫入至 Log Analytics。

邏輯應用程式包含下列各項：
- 要從事件中樞讀取的[事件中樞連接器](https://docs.microsoft.com/connectors/eventhubs/)觸發程序。
- 擷取 JSON 事件的[剖析 JSON 動作](../logic-apps/logic-apps-content-type.md)。
- 將 JSON 轉換為物件的[撰寫動作](../logic-apps/logic-apps-workflow-actions-triggers.md#compose-action)。
- 將資料張貼至 Log Analytics 的 [Log Analytics 傳送資料連接器](https://docs.microsoft.com/connectors/azureloganalyticsdatacollector/)。

   ![在 Logic Apps 中新增事件中樞觸發程序的影像](media/log-analytics-activity-logs-subscriptions/log-analytics-logic-apps-activity-log-overview.png)

### <a name="logic-app-requirements"></a>邏輯應用程式需求
建立邏輯應用程式之前，請確定您有先前步驟中的下列資訊：
- 事件中樞名稱
- 事件中樞命名空間的事件中樞連接字串 (主要或次要)。
- Log Analytics 工作區識別碼
- Log Analytics 共用金鑰

若要取得事件中樞名稱和連接字串，請遵循[檢查事件中樞命名空間權限並尋找連接字串](../connectors/connectors-create-api-azure-event-hubs.md#permissions-connection-string)中的步驟。


### <a name="create-a-new-blank-logic-app"></a>建立新的空白邏輯應用程式

1. 在 Azure 入口網站中，選擇 [建立資源] > [企業整合] > [邏輯應用程式]。

    ![Marketplace 新的邏輯應用程式](media/log-analytics-activity-logs-subscriptions/marketplace-new-logic-app.png)

2. 在下表中，輸入設定。

    ![建立邏輯應用程式](media/log-analytics-activity-logs-subscriptions/create-logic-app.png)

   |設定 | 說明  |
   |:---|:---|
   | Name           | 邏輯應用程式的唯一名稱。 |
   | 訂用帳戶   | 選取將包含邏輯應用程式的 Azure 訂用帳戶。 |
   | 資源群組 | 選取現有 Azure 資源群組，或建立邏輯應用程式的新 Azure 資源群組。 |
   | 位置       | 選取資料中心區域，用於部署邏輯應用程式。 |
   | Log Analytics  | 選取是否要記錄 Log Analytics 中每次執行邏輯應用程式的狀態。  |

    
3. 選取 [建立] 。 顯示 [部署成功] 通知時，請按一下 [前往資源] 開啟邏輯應用程式。

4. 在 [範本] 底下，選擇 [空白邏輯應用程式]。 

邏輯應用程式設計工具現在會顯示您可用的連接器和其觸發程序，以用於啟動邏輯應用程式工作流程。

<!-- Learn [how to create a logic app](../logic-apps/quickstart-create-first-logic-app-workflow.md). -->

### <a name="add-event-hub-trigger"></a>新增事件中樞觸發程序

1. 在邏輯應用程式設計工具的搜尋方塊中，鍵入 *event hubs* 作為篩選條件。 選取 [Event Hubs - When events are available in Event Hub] \(事件中樞 - 當事件可用於事件中樞時) 觸發程序。

   ![在 Logic Apps 中新增事件中樞觸發程序的影像](media/log-analytics-activity-logs-subscriptions/logic-apps-event-hub-add-trigger.png)

2. 系統提示您輸入認證時，請連線至事件中樞命名空間。 輸入您連線的名稱，然後輸入您複製的連接字串。  選取 [建立] 。

   ![在 Logic Apps 中新增事件中樞連線的影像](media/log-analytics-activity-logs-subscriptions/logic-apps-event-hub-add-connection.png)

3. 建立連線之後，請編輯觸發程序的設定。 從 [事件中樞名稱] 下拉式清單中選取 **insights-operational-logs** 來開始。

   ![當事件可用時對話方塊](media/log-analytics-activity-logs-subscriptions/logic-apps-event-hub-read-events.png)

5. 展開 [顯示進階選項]，並將 [內容類型] 變更為 [application/json]

   ![事件中樞設定對話方塊](media/log-analytics-activity-logs-subscriptions/logic-apps-event-hub-configuration.png)

### <a name="add-parse-json-action"></a>新增剖析 JSON 動作

事件中樞的輸出包含具有記錄陣列的 JSON 承載。 [剖析 JSON](../logic-apps/logic-apps-content-type.md) 動作用來只擷取要傳送至 Log Analytics 的記錄陣列。

1. 按一下 [新增步驟] > [新增動作]
2. 在搜尋方塊中，鍵入 parse json 作為篩選條件。 選取 [Data Operations - Parse JSON] \(資料作業 - 剖析 JSON) 動作。

   ![在 Logic Apps 中新增剖析 JSON 動作](media/log-analytics-activity-logs-subscriptions/logic-apps-add-parse-json-action.png)

3. 按一下 [內容] 欄位，然後選取 [本文]。

4. 複製下列結構描述，並將其貼入 [結構描述] 欄位。  此結構描述符合事件中樞動作中的輸出。  

   ![剖析 JSON 對話方塊](media/log-analytics-activity-logs-subscriptions/logic-apps-parse-json-configuration.png)

``` json-schema
{
    "properties": {
        "body": {
            "properties": {
                "ContentData": {
                    "type": "string"
                },
                "Properties": {
                    "properties": {
                        "ProfileName": {
                            "type": "string"
                        },
                        "x-opt-enqueued-time": {
                            "type": "string"
                        },
                        "x-opt-offset": {
                            "type": "string"
                        },
                        "x-opt-sequence-number": {
                            "type": "number"
                        }
                    },
                    "type": "object"
                },
                "SystemProperties": {
                    "properties": {
                        "EnqueuedTimeUtc": {
                            "type": "string"
                        },
                        "Offset": {
                            "type": "string"
                        },
                        "PartitionKey": {},
                        "SequenceNumber": {
                            "type": "number"
                        }
                    },
                    "type": "object"
                }
            },
            "type": "object"
        },
        "headers": {
            "properties": {
                "Cache-Control": {
                    "type": "string"
                },
                "Content-Length": {
                    "type": "string"
                },
                "Content-Type": {
                    "type": "string"
                },
                "Date": {
                    "type": "string"
                },
                "Expires": {
                    "type": "string"
                },
                "Location": {
                    "type": "string"
                },
                "Pragma": {
                    "type": "string"
                },
                "Retry-After": {
                    "type": "string"
                },
                "Timing-Allow-Origin": {
                    "type": "string"
                },
                "Transfer-Encoding": {
                    "type": "string"
                },
                "Vary": {
                    "type": "string"
                },
                "X-AspNet-Version": {
                    "type": "string"
                },
                "X-Powered-By": {
                    "type": "string"
                },
                "x-ms-request-id": {
                    "type": "string"
                }
            },
            "type": "object"
        }
    },
    "type": "object"
}
```

>[!TIP]
> 您可以按一下 [執行]，並從事件中樞中查看 [Raw Output] \(原始輸出)，以取得範例承載。  您接著可以使用含有 [剖析 JSON] 活動中 [請使用範例承載產生結構描述] 的這個輸出，來產生結構描述。

### <a name="add-compose-action"></a>新增撰寫動作
[撰寫](../logic-apps/logic-apps-workflow-actions-triggers.md#compose-action)動作採用 JSON 輸出，並建立 Log Analytics 動作可使用的物件。

1. 按一下 [新增步驟] > [新增動作]
2. 鍵入 *compose* 作為篩選條件，然後選取 [Data Operations - Compose] \(資料作業 - 撰寫) 動作。

    ![新增撰寫動作](media/log-analytics-activity-logs-subscriptions/logic-apps-add-compose-action.png)

3. 按一下 [輸入] 欄位，然後選取 [剖析 JSON] 活動下方的 [本文]。


### <a name="add-log-analytics-send-data-action"></a>新增 Log Analytics 傳送資料動作
[Azure Log Analytics 資料收集器](https://docs.microsoft.com/connectors/azureloganalyticsdatacollector/)動作採用 [撰寫] 動作中的物件，並將它傳送至 Log Analytics。

1. 按一下 [新增步驟] > [新增動作]
2. 鍵入 log analytics 作為篩選條件，然後選取 [Azure Log Analytics Data Collector - Send Data] \(Azure Log Analytics 資料收集器 - 傳送資料) 動作。

   ![在 Logic Apps 中新增 Log Analytics 傳送資料動作](media/log-analytics-activity-logs-subscriptions/logic-apps-send-data-to-log-analytics-connector.png)

3. 輸入您連線的名稱，並貼入 Log Analytics 工作區的 [工作區識別碼] 和 [工作區金鑰]。  按一下頁面底部的 [新增] 。

   ![在 Logic Apps 中新增 Log Analytics 連線](media/log-analytics-activity-logs-subscriptions/logic-apps-log-analytics-add-connection.png)

4. 建立連線之後，請編輯下表中的設定。 

    ![設定傳送資料動作](media/log-analytics-activity-logs-subscriptions/logic-apps-send-data-to-log-analytics-configuration.png)

   |設定        | 值           | 說明  |
   |---------------|---------------------------|--------------|
   |JSON 要求本文  | [撰寫] 動作的 [輸出] | 從 [撰寫] 動作的本文中擷取記錄。 |
   | 自訂記錄名稱 | AzureActivity | 要在 Log Analytics 中建立以保留所匯入資料的自訂記錄表名稱。 |
   | time-generated-field | 分析 | 請不要針對**時間**選取 JSON 欄位 - 只要鍵入 word 時間即可。 如果您選取 JSON 欄位，則設計工具會將 [傳送資料] 動作放入 [For Each] 迴圈，但這不是您要的作業。 |




10. 按一下 [儲存] 儲存您對邏輯應用程式所做的變更。

## <a name="step-4---test-and-troubleshoot-the-logic-app"></a>步驟 4 - 針對邏輯應用程式進行測試和疑難排解
工作流程完成時，即可在設計工具中測試，以驗證它運作正常未發生錯誤。

在邏輯應用程式設計工具中，按一下 [執行] 測試應用程式。 邏輯應用程式中的每個步驟都會顯示狀態圖示，而綠色圓圈內有白色勾號指出成功。

   ![測試邏輯應用程式](media/log-analytics-activity-logs-subscriptions/test-logic-app.png)

若要查看每個步驟的詳細資訊，請按一下步驟名稱予以展開。 按一下 [Show raw inputs] \(顯示原始輸入) 和 [Show raw outputs] \(顯示原始輸出)，以查看每個步驟所接收和傳送資料的詳細資訊。

## <a name="step-5---view-azure-activity-log-in-log-analytics"></a>步驟 5 - 在 Log Analytics 中檢視 Azure 活動記錄
最後一個步驟是檢查 Log Analytics 工作區，確定正在如預期收集資料。

1. 在 Azure 入口網站中，按一下左上角的 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。
2. 在 Log Analytics 工作區清單中，選取您的工作區。
3.  按一下 [記錄搜尋] 圖格，然後在 [記錄搜尋] 窗格的查詢欄位中輸入 `AzureActivity_CL`，再按 Enter 鍵，或按一下查詢欄位右邊的搜尋按鈕。 如果您未將自訂記錄命名為 *AzureActivity*，則請鍵入選擇的名稱，並附加 `_CL`。

>[!NOTE]
> 第一次將新的自訂記錄傳送至 Log Analytics 時，最多可能需要一個小時的時間來搜尋自訂記錄。

>[!NOTE]
> 活動記錄會寫入自訂資料表，而且不會出現在[活動記錄解決方案](./log-analytics-activity.md)中。


![測試邏輯應用程式](media/log-analytics-activity-logs-subscriptions/log-analytics-results.png)

## <a name="next-steps"></a>後續步驟

在本文中，您已建立邏輯應用程式來讀取事件中樞的 Azure 活動記錄，並將它們傳送至 Log Analytics 以進行分析。 若要深入了解如何在 Log Analytics 中視覺化資料 (包含建立儀表板)，請檢閱視覺化資料的教學課程。

> [!div class="nextstepaction"]
> [視覺化記錄搜尋資料教學課程](./log-analytics-tutorial-dashboards.md)
