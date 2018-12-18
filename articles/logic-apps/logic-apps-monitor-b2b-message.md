---
title: 監視 B2B 訊息並設定記錄 - Azure Logic Apps | Microsoft Docs
description: 監視 AS2、X12 和 EDIFACT 訊息。 在 Azure Logic Apps 中設定整合帳戶的診斷記錄。
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: bb7d9432-b697-44db-aa88-bd16ddfad23f
ms.date: 07/21/2017
ms.openlocfilehash: fd48dfaf7b863ab026da1758751509d9911ae01e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948004"
---
# <a name="monitor-b2b-messages-and-set-up-logging-for-integration-accounts-in-azure-logic-apps"></a>在 Azure Logic Apps 中監視 B2B 訊息並設定整合帳戶的診斷記錄

在您透過整合帳戶設定兩個執行中商務程序或應用程式之間的 B2B 通訊之後，這些實體也可以彼此交換訊息。 若要確認此通訊如預期運作，您可以設定 AS2、X12 和 EDIFACT 訊息的監視，以及透過 [Azure Log Analytics](../log-analytics/log-analytics-overview.md) 服務之整合帳戶的診斷記錄。 這個服務會監視您的雲端和內部部署環境，協助您維護其可用性和效能，同時收集執行階段詳細資料和事件，進行更豐富的偵錯。 您也可以[搭配使用診斷資料與其他服務](#extend-diagnostic-data)，例如 Azure 儲存體和 Azure 事件中樞。

## <a name="requirements"></a>需求

* 已設定診斷記錄的邏輯應用程式。 了解[如何設定該邏輯應用程式的記錄](../logic-apps/logic-apps-monitor-your-logic-apps.md#azure-diagnostics)。

  > [!NOTE]
  > 在您符合這項需求之後，Log Analytics 中應該有工作區。 當您設定整合帳戶的記錄時，應該使用相同的 Log Analytics 工作區。 如果您沒有 Log Analytics 工作區，請了解[如何建立 Log Analytics 工作區](../log-analytics/log-analytics-quick-create-workspace.md)。

* 連結至邏輯應用程式的整合帳戶。 了解[如何建立具有邏輯應用程式連結的整合帳戶](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md)。

## <a name="turn-on-diagnostics-logging-for-your-integration-account"></a>開啟整合帳戶的診斷記錄

您可以直接從整合帳戶或[透過 Azure 監視器服務](#azure-monitor-service)來開啟記錄。 Azure 監視器提供基礎結構層級資料的基本監視。 深入了解 [Azure 監視器](../azure-monitor/overview.md)。

### <a name="turn-on-diagnostics-logging-directly-from-your-integration-account"></a>直接從整合帳戶開啟診斷記錄

1. 在 [Azure 入口網站](https://portal.azure.com)中，尋找並選取整合帳戶。 在 [監視] 下，選擇 [診斷記錄]，如下所示：

   ![尋找並選取整合帳戶，然後選擇 [診斷記錄]。](media/logic-apps-monitor-b2b-message/integration-account-diagnostics.png)

2. 在您選取整合帳戶之後，會自動選取下列值。 如果這些值正確，請選擇 [開啟診斷]。 否則，請選取您想要的值：

   1. 在 [訂用帳戶] 下，選取您要與整合帳戶搭配使用的 Azure 訂用帳戶。
   2. 在 [資源群組] 下，選取您要與整合帳戶搭配使用的資源群組。
   3. 在 [資源類型] 下，選取 [整合帳戶]。 
   4. 在 [資源] 下，選取整合帳戶。 
   5. 選擇 [開啟診斷]。

   ![針對整合帳戶設定診斷](media/logic-apps-monitor-b2b-message/turn-on-diagnostics-integration-account.png)

3. 在 [診斷設定] 的 [狀態] 下，選擇 [開啟]。

   ![開啟 Azure 診斷](media/logic-apps-monitor-b2b-message/turn-on-diagnostics-integration-account-2.png)

4. 現在選取要用於記錄的 Log Analytics 工作區和資料，如下所示：

   1. 選取 [傳送至 Log Analytics]。 
   2. 在 [Log Analytics] 下，選擇 [設定]。 
   3. 在 [OMS 工作區] 下，選取要用於記錄的 Log Analytics 工作區。
   4. 在 [記錄] 下，選取 [IntegrationAccountTrackingEvents] 分類。
   5. 選擇 [儲存]。

   ![設定 Log Analytics 以將診斷資料傳送至記錄](media/logic-apps-monitor-b2b-message/send-diagnostics-data-log-analytics-workspace.png)

5. 現在[在 Log Analytics 中設定 B2B 訊息的追蹤](../logic-apps/logic-apps-track-b2b-messages-omsportal.md)。

<a name="azure-monitor-service"></a>

### <a name="turn-on-diagnostics-logging-through-azure-monitor"></a>透過 Azure 監視器開啟診斷記錄

1. 在 [Azure 入口網站](https://portal.azure.com)中，於主要 Azure 功能表上依序選擇 [監視器] 和 [診斷記錄]。 然後選取整合帳戶，如下所示：

   ![依序選擇 [監視器] 和 [診斷記錄]，然後選取整合帳戶](media/logic-apps-monitor-b2b-message/monitor-service-diagnostics-logs.png)

2. 在您選取整合帳戶之後，會自動選取下列值。 如果這些值正確，請選擇 [開啟診斷]。 否則，請選取您想要的值：

   1. 在 [訂用帳戶] 下，選取您要與整合帳戶搭配使用的 Azure 訂用帳戶。
   2. 在 [資源群組] 下，選取您要與整合帳戶搭配使用的資源群組。
   3. 在 [資源類型] 下，選取 [整合帳戶]。
   4. 在 [資源] 下，選取整合帳戶。
   5. 選擇 [開啟診斷]。

   ![針對整合帳戶設定診斷](media/logic-apps-monitor-b2b-message/turn-on-diagnostics-integration-account.png)

3. 在 [診斷設定] 下，選擇 [開啟]。

   ![開啟 Azure 診斷](media/logic-apps-monitor-b2b-message/turn-on-diagnostics-integration-account-2.png)

4. 現在選取用於記錄的 Log Analytics 工作區和事件類別目錄，如下所示：

   1. 選取 [傳送至 Log Analytics]。 
   2. 在 [Log Analytics] 下，選擇 [設定]。 
   3. 在 [OMS 工作區] 下，選取要用於記錄的 Log Analytics 工作區。
   4. 在 [記錄] 下，選取 [IntegrationAccountTrackingEvents] 分類。
   5. 完成之後，請選擇 [儲存]。

   ![設定 Log Analytics 以將診斷資料傳送至記錄](media/logic-apps-monitor-b2b-message/send-diagnostics-data-log-analytics-workspace.png)

5. 現在[在 Log Analytics 中設定 B2B 訊息的追蹤](../logic-apps/logic-apps-track-b2b-messages-omsportal.md)。

## <a name="extend-how-and-where-you-use-diagnostic-data-with-other-services"></a>延伸搭配使用診斷資料與其他服務的方式和位置

使用 Azure Log Analytics，即可延伸如何搭配使用邏輯應用程式的診斷資料與其他 Azure services，例如： 

* [在 Azure 儲存體中封存 Azure 診斷記錄](../monitoring-and-diagnostics/monitoring-archive-diagnostic-logs.md)
* [將 Azure 診斷記錄串流至 Azure 事件中樞](../monitoring-and-diagnostics/monitoring-stream-diagnostic-logs-to-event-hubs.md) 

您可以接著使用其他服務的遙測和分析來取得即時監視，例如 [Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md) 和 [Power BI](../log-analytics/log-analytics-powerbi.md)。 例如︰

* [將資料從事件中樞串流至串流分析](../stream-analytics/stream-analytics-define-inputs.md)
* [使用串流分析分析串流資料並在 Power BI 中建立即時分析儀表板](../stream-analytics/stream-analytics-power-bi-dashboard.md)

根據您要設定的選項，確定您先[建立 Azure 儲存體帳戶](../storage/common/storage-create-storage-account.md)或[建立 Azure 事件中樞](../event-hubs/event-hubs-create.md)。 然後選取您要傳送診斷資料的選項：

![將資料傳送至 Azure 儲存體帳戶或事件中樞](./media/logic-apps-monitor-b2b-message/storage-account-event-hubs.png)

> [!NOTE]
> 只有在您選擇使用儲存體帳戶時，才會套用保留期間。

## <a name="supported-tracking-schemas"></a>支援的追蹤結構描述

Azure 支援下列追蹤結構描述類型，其中除了自訂類型外都有固定的結構描述。

* [AS2 追蹤結構描述](../logic-apps/logic-apps-track-integration-account-as2-tracking-schemas.md)
* [X12 追蹤結構描述](../logic-apps/logic-apps-track-integration-account-x12-tracking-schema.md)
* [自訂追蹤結構描述](../logic-apps/logic-apps-track-integration-account-custom-tracking-schema.md)

## <a name="next-steps"></a>後續步驟

* [在 Log Analytics 中追蹤 B2B 訊息](../logic-apps/logic-apps-track-b2b-messages-omsportal.md "在 OMS 中追蹤 B2B 訊息")
* [深入了解企業整合套件](../logic-apps/logic-apps-enterprise-integration-overview.md "了解企業整合套件")

