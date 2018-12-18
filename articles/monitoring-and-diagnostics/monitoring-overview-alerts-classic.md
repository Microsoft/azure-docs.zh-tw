---
title: Microsoft Azure 和 Azure 監視器中的傳統警示概觀
description: 傳統警示即將淘汰。 警示可讓您監視 Azure 資源度量、事件或記錄檔，並在您所指定條件符合時收到通知。
author: rboucher
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: robb
ms.component: alerts
ms.openlocfilehash: 7046a0c6ac84ad5f156098a26dcef2b8accd50af
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987640"
---
# <a name="what-are-classic-alerts-in-microsoft-azure"></a>什麼是 Microsoft Azure 中的傳統警示？

> [!NOTE]
> 本文說明如何建立舊版傳統計量警示。 「Azure 監視器」現已支援[新版的近乎即時計量警示和新的警示體驗](monitoring-overview-alerts.md)。 
>

警示可讓您對資料設定條件，並在最近的監視資料符合條件時收到通知。

## <a name="old-and-new-alerting-capabilities"></a>舊的和新的警示功能

在舊的 Azure 監視器中，Application Insights、Log Analytics 和服務健康狀態具有個別的警示功能。 隨著時間進展，Azure 改善並結合使用者介面與不同的警示方法。 這樣的整併仍在持續進行中。 警示

您只能在 Azure 入口網站的傳統警示使用者畫面中檢視傳統警示。 您可以從警示畫面上的 [檢視傳統警示] 按鈕取得此畫面。 

 ![Azure 入口網站中的警示選擇](./media/monitoring-overview-alerts-classic/monitor-alert-screen2.png) 

新的警示使用者體驗比傳統警示體驗多了下列優點：
-   **更好的通知系統**- 所有新版警示都使用動作群組，這些是可在多個警示中重複使用的具名通知及動作群組。 傳統計量警示及舊版 Log Analytics 警示並不使用動作群組。
-   **統一的撰寫體驗** - Azure 監視器、Log Analytics 及 Application Insights 之計量、記錄和活動記錄的所有警示建立作業都在一個地方完成。
-   **在 Azure 入口網站中檢視引發的 Log Analytics 警示** - 您現在也能在訂用帳戶中查看引發的 Log Analytics 警示。 先前這些警示是在個別的入口網站中。
-   **分隔引發的警示和警示規則** - 區分警示規則 (定義觸發警示的條件) 和引發的警示 (警示規則引發的執行個體)，因此將操作與設定檢視分開。
-   **更好的工作流程** - 新的警示編寫體驗會引導使用者進行設定警示規則的程序，而能更容易找到要取得警示的正確項目。
-   **智慧型警示彙總**和**設定警示狀態** - 新版警示包含自動群組功能，可將類似的警示顯示在一起，以減少使用者介面中的多載。 

新版計量警示比傳統計量警示多了下列優點：
-   **改善延遲**：新版計量警示的執行頻率可以達到每分鐘一次。 舊版計量警示的執行頻率一律是每 5 分鐘一次。 新版警示從發生問題到通知或動作的延遲較短 (3 至 5 分鐘)。 舊版警示則是 5 至 15 分鐘，視類型而定。  記錄警示通常會有 10 到 15 分鐘的延遲，這是因為內嵌記錄需要時間，但新版的處理方法可縮短該時間。 
-   **支援多維度計量**：您可以針對維度計量發出警示，以便監視計量中某個您感興趣的區段。
-   **對計量條件有更多的控制**：您可以定義更豐富的警示規則。 新版警示支援監視計量的最大值、最小值、平均及總計值。
-   **可合併監視多個計量**：您可以使用單一規則來監視多個計量 (目前最多兩個計量)。 若兩個計量在指定的期間內都超出其個別閾值，就會觸發警示。
-   **更好的通知系統**：所有新版警示都使用[動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)，這些是可在多個警示中重複使用的具名通知及動作群組。  傳統計量警示及舊版 Log Analytics 警示並不使用動作群組。 
-   **來自記錄的計量** (公開預覽)：現在可將進入 Log Analytics 的記錄資料擷取並轉換成「Azure 監視器」計量，然後針對這些計量發出警示，就像任何其他計量一樣。 如需傳統警示專用的術語，請參閱[警示 (傳統)](monitoring-overview-alerts-classic.md)。 


## <a name="classic-alerts-on-azure-monitor-data"></a>Azure 監視器資料的傳統警示
可用的傳統警示類型有兩種：計量警示和活動記錄警示。

* **傳統計量警示**：當指定的計量值超出您指派的閾值時，就會觸發此警示。 當警示「已啟動」(超過閾值且符合警示條件) 時，會產生通知。 當警示「已解決」 (再次超出閾值但不再符合條件) 時，它會產生另一個通知。

* **傳統活動記錄警示** - 產生符合您所指派篩選準則之「活動記錄」事件時會觸發的資料流記錄警示。 這些警示只有「已啟動」這個狀態，因為警示引擎只會將篩選準則套用至任何新的事件。 您可以使用這些警示，在發生新的服務健康狀態事件時，或是在使用者或應用程式於您的訂用帳戶中執行作業時 (例如「刪除虛擬機器」)，收到通知。

針對透過 Azure 監視器提供的診斷記錄資料，請將資料路由傳送至 Log Analytics (先前稱為 OMS) 並使用 Log Analytics 查詢警示。 Log Analytics 現在使用[新的警示方法](monitoring-overview-unified-alerts.md) 

下圖摘要說明 Azure 監視器中的資料來源，以及就概念而言如何發出該資料的警示。

![警示的說明](./media/monitoring-overview-alerts-classic/Alerts_Overview_Resource_v5.png)

## <a name="taxonomy-of-alerts-classic"></a>警示 (傳統) 的分類法
Azure 使用下列詞彙來描述傳統警示及其功能：
* **警示** - 符合時會啟動之準則 (一或多個規則或條件) 的定義。
* **作用中** - 符合傳統警示所定義準則時的狀態。
* **已解決** - 先前符合傳統警示所定義的準則，但之後已不再符合時的狀態。
* **通知** - 根據變成作用中之傳統警示而採取的動作。
* **動作** - 傳送給通知接收者的特定呼叫 (例如以電子郵件傳送位址或張貼到 Webhook URL)。 通知通常可觸發多個動作。

## <a name="how-do-i-receive-a-notification-from-an-azure-monitor-classic-alert"></a>如何收到 Azure 監視器傳統警示傳來的通知？
在過去，Azure 的警示來自不同的服務，各自使用其專屬的內建通知方法。 

Azure 監視器會建立可重複使用的通知群組，稱為*動作群組*。 動作群組會指定一組通知接收者，每當啟動參考此動作群組的警示時，所有接收者都會收到該通知。 動作群組可讓您在許多警示物件之間重複使用一組接收者 (例如您隨時待命的工程師清單)。 除了電子郵件地址、簡訊號碼和一些其他動作，動作群組的通知支援還包括張貼到 Webhook URL。  如需詳細資訊，請參閱[動作群組](monitoring-action-groups.md)。 

舊版傳統活動記錄警示會使用動作群組。

不過，舊版計量警示不會使用動作群組。 您可以改為設定下列動作： 
- 將電子郵件通知傳送至服務管理員、共同管理員或您指定的其他電子郵件。
- 呼叫 webhook，可讓您啟動其他自動化動作。

Webhook 可啟用自動化和修復，例如使用：
    - Azure 自動化 Runbook
    - Azure Function
    - Azure 邏輯應用程式
    - 第三方服務

## <a name="next-steps"></a>後續步驟
使用下列項目取得有關警示規則和設定這些規則的資訊：

* 深入了解[計量](monitoring-overview-metrics.md)
* [透過 Azure 入口網站設定傳統的計量警示](insights-alerts-portal.md)
* 設定[傳統的計量警示 PowerShell](insights-alerts-powershell.md)
* 設定[傳統的計量警示命令列介面 (CLI)](insights-alerts-command-line-interface.md)
* 設定[傳統的計量警示 Azure 監視器 REST API](https://msdn.microsoft.com/library/azure/dn931945.aspx)
* 深入了解[活動記錄](monitoring-overview-activity-logs.md)
* [透過 Azure 入口網站設定活動記錄警示](monitoring-activity-log-alerts.md)
* [透過 Resource Manager 設定活動記錄警示](monitoring-create-activity-log-alerts-with-resource-manager-template.md)
* 檢閱[活動記錄警示 webhook 結構描述](monitoring-activity-log-alerts-webhook.md)
* 深入了解[動作群組](monitoring-action-groups.md)
* 設定[較新的警示](monitor-alerts-unified-usage.md)