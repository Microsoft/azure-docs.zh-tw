---
title: Log Analytics 常見問題集 | Microsoft Docs
description: Azure Log Analytics 服務的相關常見問題的解答。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ad536ff7-2c60-4850-a46d-230bc9e1ab45
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/18/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: d873d28ec772735b8f21551cde1c7d0eab79a18c
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47035134"
---
# <a name="log-analytics-faq"></a>Log Analytics 常見問題集
此 Microsoft 常見問題集是 Microsoft Azure 中 Log Analytics 常見問題的清單。 若您有任何關於 Log Analytics 的其他問題，請前往[討論論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=opinsights)並張貼您的問題。 當問到常見問題時，我們會將其新增至此文章，以便其他人可以快速輕鬆地找到此問題。


## <a name="new-logs-experience"></a>新的 Logs 體驗

### <a name="q-whats-the-difference-between-the-new-logs-experience-and-log-analytics"></a>問：新的 Logs 體驗與 Log Analytics 之間有何差異？

答：兩者相同。 [Log Analytics 會整合為 Azure 監視器中的功能](../azure-monitor/azure-monitor-rebrand.md)，以提供更加統一的監視體驗。 Azure 監視器中的新 Logs 體驗，與許多客戶已在使用的 Log Analytics 查詢完全相同。

### <a name="q-can-i-still-use-log-search"></a>問：我仍可使用「記錄搜尋」嗎？ 

答：「記錄搜尋」目前仍可在 OMS 入口網站和 Azure 入口網站中以 **Logs (傳統)** 名稱取得。 OMS 入口網站將於 2019 年 1 月 15 日正式淘汰。 Azure 入口網站中的傳統 Logs 體驗會逐漸淘汰並取代新的 Logs 體驗。 

### <a name="q-can-i-still-use-advanced-analytics-portal"></a>問： 我仍可使用進階分析入口網站嗎？ 
Azure 入口網站中的新 Logs 體驗是以[進階分析入口網站](https://portal.loganalytics.io/)為基礎，但仍可在 Azure 入口網站外部存取。 即將將宣布淘汰此外部入口網站的藍圖。

### <a name="q-why-cant-i-see-query-explorer-and-save-buttons-in-the-new-logs-experience"></a>問： 何無法在新的 Logs 體驗中看見 [查詢總管] 和 [儲存] 按鈕？

在特定資源的內容中探索 Logs 時，無法使用 [查詢總管]、[儲存] 和 [設定警示] 按鈕。 若要建立警示、儲存或載入查詢，Logs 必須受限於工作區。 若要在工作區內容中開啟 Logs，請選取 [所有服務] > [監視器] > [Logs]。 已選取上次使用的工作區，但您可以選取任何其他工作區。 如需詳細資訊，請參閱[在 Log Analytics 中檢視及分析資料](../log-analytics/log-analytics-log-search-portals.md)。

### <a name="q-how-do-i-extract-custom-fields-in-the-new-logs-experience"></a>問： 如何在新的 Logs 體驗中擷取自訂欄位？ 

答：目前傳統 Logs 體驗才支援自訂欄位擷取。 

### <a name="q-where-do-i-find-list-view-in-the-new-logs"></a>問： 在新的 Logs 中哪裡可以找到 [清單] 檢視？ 

答：新的 Logs 中無法使用 [清單] 檢視。 結果表格中的每筆記錄左邊會有一個箭號。 按一下此箭號就能開啟特定記錄的詳細資料。 

### <a name="q-after-running-a-query-a-list-of-suggested-filters-shows-up-but-it-doesnt-include-all-filters-how-can-i-see-the-rest"></a>問： 執行查詢之後，建議的篩選條件清單隨即出現，但不包含所有篩選條件。 如何查看其餘的篩選條件？ 

答：您目前看到的內容是新的篩選條件實作預覽。 這現在是以您的完整結果集為基礎，而不是受限於 UI 的 10,000 筆記錄限制。 這目前是最熱門篩選條件的清單，以及每個篩選條件最常用的 10 個值。 

### <a name="q-why-am-i-getting-the-error-register-resource-provider-microsoftinsights-for-this-subscription-to-enable-this-query-in-logs-after-drilling-in-from-vm"></a>問： 從 VM 深入鑽研之後，為何會在 Logs 中收到錯誤：「為此訂用帳戶註冊資源提供者 'Microsoft.Insights' 以啟用此查詢」？ 

答：根據預設，許多資源提供者都會自動註冊。不過，您可能需要手動註冊某些資源提供者。 這會將您的訂用帳戶設定為可搭配資源提供者使用。 註冊範圍一律是訂用帳戶。 如需詳細資訊，請參閱[資源提供者和類型](../azure-resource-manager/resource-manager-supported-services.md#portal)。

### <a name="q-why-am-i-am-getting-no-access-error-message-when-accessing-logs-from-a-vm-page"></a>問： 從 VM 頁面存取 Logs 時，為何不會收到任何存取錯誤訊息？ 

答：若要檢視 VM 記錄，您必須獲得存放 VM 記錄的工作區讀取權限。 在這些情況下，系統管理員必須授與您 Azure 權限。

### <a name="q-why-can-i-can-access-my-workspace-in-oms-portal-but-i-get-the-error-you-have-no-access-in-the-azure-portal"></a>問： 為何我可以在 OMS 入口網站中存取我的工作區，但卻在 Azure 入口網站中收到錯誤「您沒有存取權」？  

答：若要在 Azure 中存取工作區，您必須獲得 Azure 權限。 有某些情況下，您可能沒有適當的存取權限。 在這些情況下，系統管理員必須授與您 Azure 權限。如需詳細資訊，請參閱 [OMS 入口網站移至 Azure](../log-analytics/log-analytics-oms-portal-transition.md)。

### <a name="q-why-cant-i-cant-see-view-designer-entry-in-logs"></a>問： 為何在 Logs 中看不到「檢視設計工具」項目？ 
答：Logs 中的「檢視設計工具」僅適用於獲派「參與者」權限或更高權限的使用者。

### <a name="q-can-i-still-use-the-analytics-portal-outside-of-azure"></a>問： 我仍可在 Azure 外部使用 Analytics 入口網站嗎？
A. 是，Azure 中的 Logs 頁面和[進階 Analytics 入口網站](https://portal.loganalytics.io)是以相同的程式碼為基礎。 Log Analytics 會整合為 Azure 監視器中的功能，以提供更加統一的監視體驗。 您仍然可以使用以下 URL 存取 Analytics 入口網站： https://portal.loganalytics.io/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/workspaces/{workspaceName}。



## <a name="general"></a>一般

### <a name="q-how-can-i-see-my-views-and-solutions-in-azure-portal"></a>問： 如何在 Azure 入口網站中查看我的檢視和解決方案？ 

答：在 Azure 入口網站中可取得檢視和已安裝解決方案的清單。 按一下 [所有服務]。 在資源清單中，選取 [監視]，然後按一下 [...更多]。 已選取上次使用的工作區，但您可以選取任何其他工作區。 

### <a name="q-why-i-cant-create-workspaces-in-west-central-us-region"></a>問： 為何無法在美國中西部區域建立工作區？ 

答：此區域有暫存容量限制。 此限制已計劃在 2019 年上半年獲得解決。


### <a name="q-does-log-analytics-use-the-same-agent-as-azure-security-center"></a>問： Log Analytics 會使用相同的代理程式作為 Azure 資訊安全中心嗎？

答：在 2017 年 6 月初，Azure 資訊安全中心開始使用 Microsoft Monitoring Agent 來收集與儲存資料。 若要深入了解，請參閱 [Azure 資訊安全中心平台移轉常見問題集](../security-center/security-center-platform-migration-faq.md)。

### <a name="q-what-checks-are-performed-by-the-ad-and-sql-assessment-solutions"></a>問： AD 和 SQL 評估解決方案會執行哪些檢查？

答：下列查詢會顯示目前執行的所有檢查的描述：

```
(Type=SQLAssessmentRecommendation OR Type=ADAssessmentRecommendation) | dedup RecommendationId | select FocusArea, ActionArea, Recommendation, Description | sort Type, FocusArea,ActionArea, Recommendation
```

然後可將結果匯出至 Excel 供進一步檢閱。

### <a name="q-why-do-i-see-something-different-than-oms-in-the-system-center-operations-manager-console"></a>問： 為什麼我在 System Center Operations Manager 主控台中看到與 OMS 不同的項目？

答：根據您所採用的 Operations Manager 更新彙總套件而定，您可能會看到 *System Center Advisor*、*Operational Insights* 或 *Log Analytics* 節點。

對 *OMS* 文字字串的更新包含在管理組件西，必須手動匯入它。 若要查看目前文字和功能，請遵循最新 System Center Operations Manager 更新彙總套件知識庫文章中的指示，並重新整理主控台。

### <a name="q-is-there-an-on-premises-version-of-log-analytics"></a>問：Log Analytics 是否有內部部署版本？

答：否。 Log Analytics 是一個可擴充的雲端服務，可處理並儲存大量資料。 

### <a name="q-how-do-i-troubleshoot-if-log-analytics-is-no-longer-collecting-data"></a>問： 如何針對 Log Analytics 不再收集資料的問題進行疑難排解？

答：針對 2018 年 4 月 2 日之前建立且位於「免費」定價層的訂用帳戶和工作區，如果每天傳送的資料超過 500 MB，即會針對當日的剩餘時間停止資料收集。 達到每日限制是 Log Analytics 停止收集資料或資料似乎遺失的常見原因。  

Log Analytics 會建立*活動訊號*類型的事件，而且可用於判斷資料收集是否停止。 

在搜尋中執行下列查詢，即可檢查您是否達到每日限制並遺失資料：`Heartbeat | summarize max(TimeGenerated)`

若要檢查特定電腦，請執行下列查詢：`Heartbeat | where Computer=="contosovm" | summarize max(TimeGenerated)`

當資料收集停止時，根據選取的時間範圍，您不會看到任何傳回的記錄。   

下表描述資料收集停止的原因，並建議為繼續資料收集所要採取的動作：

| 資料收集停止的原因                       | 若要繼續資料收集 |
| -------------------------------------------------- | ----------------  |
| 已達免費資料的限制<sup>1</sup>       | 請等到隔月自動重新開始收集，或<br> 變更為付費定價層 |
| Azure 訂用帳戶處於暫停狀態，原因如下： <br> 免費試用已結束 <br> Azure Pass 已過期 <br> 已達每月消費限制 (例如 MSDN 或 Visual Studio 訂閱)                          | 轉換成付費訂閱 <br> 轉換成付費訂閱 <br> 移除限制，或等到限制重設 |

<sup>1</sup> 如果您的工作區位於「免費」定價層，您每天最多可傳送 500 MB 的資料至服務。 當您達到每日限制時，資料收集就會停止，直到隔天再開始。 在資料收集停止時所傳送的資料不會編製索引，而且無法供搜尋使用。 當資料收集繼續時，只會處理傳送的新資料。 

Log Analytics 使用 UTC 時間，而且每天從午夜 UTC 開始。 如果工作區達到每日限制，會在隔天 UTC 的第一個小時繼續處理。

### <a name="q-how-can-i-be-notified-when-data-collection-stops"></a>問： 如何在資料收集停止時收到通知？

答：使用[建立新記錄警示](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md)中所述的步驟，以在資料收集停止時收到通知。

建立要在資料收集停止發出的警示時：

- **定義警示條件**：將您的 Log Analytics 工作區指定為資源目標。
- **警示準則**：指定下列項目：
   - **訊號名稱**：選取 [自訂記錄搜尋]。
   - 將 [搜尋查詢] 設定為 `Heartbeat | summarize LastCall = max(TimeGenerated) by Computer | where LastCall < ago(15m)`
   - [警示邏輯] 為 [根據結果數目]，而 [條件] 為 [大於臨界值 0]
   - [時間週期] 為 30 分鐘，而 [警示頻率] 設定為每隔 10 分鐘
- **定義警示詳細資料**：指定下列項目：
   - 將 [名稱] 設定為「資料收集已停止」
   - 將 [嚴重性] 設定為「警告」

指定現有或建立新的[動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)，以便在記錄警示符合準則時 (如果活動訊號遺失超過 15 分鐘) 收到通知。

## <a name="configuration"></a>組態
### <a name="q-can-i-change-the-name-of-the-tableblob-container-used-to-read-from-azure-diagnostics-wad"></a>問： 可以變更用來從 Azure 診斷 (WAD) 讀取的資料表/Blob 容器的名稱嗎？

A. 不可以，目前無法讀取 Azure 儲存體中的任意資料表或容器。

### <a name="q-what-ip-addresses-does-the-log-analytics-service-use-how-do-i-ensure-that-my-firewall-only-allows-traffic-to-the-log-analytics-service"></a>問： Log Analytics 服務使用哪些 IP 位址？ 如何確保我的防火牆只允許對 Log Analytics 服務的流量？

A. Log Analytics 服務以 Azure 為建置基礎。 Log Analytics IP 位址位於 [Microsoft Azure 資料中心 IP 範圍](http://www.microsoft.com/download/details.aspx?id=41653)內。

進行服務部署時，Log Analytics 服務的實際 IP 位址會變更。 允許通過防火牆的 DNS 名稱會記載於[網路需求](log-analytics-concept-hybrid.md#network-firewall-requirements)中。

### <a name="q-i-use-expressroute-for-connecting-to-azure-does-my-log-analytics-traffic-use-my-expressroute-connection"></a>問： 我可以使用 ExpressRoute 連接到 Azure。 我的 Log Analytics 流量是否會使用我的 ExpressRoute 連線？

A. [ExpressRoute 文件](../expressroute/expressroute-faqs.md#supported-services)中描述不同類型的 ExpressRoute 流量。

通往 Log Analytics 的流量都會使用公用互連 ExpressRoute 電路。

### <a name="q-is-there-a-simple-and-easy-way-to-move-an-existing-log-analytics-workspace-to-another-log-analytics-workspaceazure-subscription"></a>問： 有簡單且輕鬆的方法，可將現有的 Log Analytics 工作區移至另一個 Log Analytics 工作區/Azure 訂用帳戶嗎？

A. `Move-AzureRmResource` Cmdlet 可讓您將 Log Analytics 工作區及自動化帳戶從一個 Azure 訂用帳戶移至另一個 Azure 訂用帳戶。 如需詳細資訊，請參閱 [Move-AzureRmResource](http://msdn.microsoft.com/library/mt652516.aspx)。

這項變更也可在 Azure 入口網站進行。

您無法將資料在不同一個 Log Analytics 工作區間移動，或是變更 Log Analytics 資料儲存所在的區域。

### <a name="q-how-do-i-add-log-analytics-to-system-center-operations-manager"></a>問：如何將 Log Analytics 新增至 System Center Operations Manager？

答：更新為最新的更新彙總套件和匯入管理組件可讓您將 Operations Manager 連線到 Log Analytics。

>[!NOTE]
>只有 System Center Operations Manager 2012 SP1 和更新版本才能將 Operations Manager 連線到 Log Analytics。

### <a name="q-how-can-i-confirm-that-an-agent-is-able-to-communicate-with-log-analytics"></a>問：如何確認代理程式可與 Log Analytics 通訊？

答：若要確保代理程式可以與 OMS 通訊，請移至：[控制台] > [安全性和設定] > [Microsoft Monitoring Agent]。

在 [Azure Log Analytics (OMS)]  索引標籤中，找出綠色的核取記號。 綠色核取記號圖示可確認代理程式能夠與 Azure 服務進行通訊。

黃色警告圖示表示代理程式有與 Log Analytics 通訊的問題。 一個常見原因是 Microsoft Monitoring Agent 服務已停止。 使用服務控制管理員來重新啟動服務。

### <a name="q-how-do-i-stop-an-agent-from-communicating-with-log-analytics"></a>問：如何阻止代理程式與 Log Analytics 通訊？

答：在 System Center Operations Manager 中，從 OMS 受控電腦清單中移除該電腦。 Operations Manager 會將代理程式的設定更新為不再向 Log Analytics 回報。 針對直接連線到 Log Analytics 的代理程式，您可以透過 [控制台] > [安全性和設定] > [Microsoft Monitoring Agent] 來阻止其通訊。
在 **Azure Log Analytics (OMS)** 下，移除所有列出的工作區。

### <a name="q-why-am-i-getting-an-error-when-i-try-to-move-my-workspace-from-one-azure-subscription-to-another"></a>問︰當我試著將工作區從某個 Azure 訂用帳戶移到另一個時，為什麼會發生錯誤？

答：如果您使用 Azure 入口網站，請確定只選取要移動的工作區。 請勿選取解決方案，移動工作區之後會自動移動解決方案。 

請確定您具有這兩個 Azure 訂用帳戶的權限。

## <a name="agent-data"></a>代理程式資料
### <a name="q-how-much-data-can-i-send-through-the-agent-to-log-analytics-is-there-a-maximum-amount-of-data-per-customer"></a>問： 我可以透過代理程式傳送多少資料到 Log Analytics？ 是否有每位客戶最大的資料量？
A. 免費方案每個工作區的每日容量設定為 500 MB。 標準和進階計畫對於所上傳的資料量沒有限制。 作為一項雲端服務，Log Analytics 的設計可自動相應增加，以處理來自客戶的資料量，即使是每日數 TB。

Log Analytics 代理程式的設計是為了確保它的使用量很小。 資料量會視您啟用的解決方案而不同。 您可以在[使用量](log-analytics-usage.md)頁面中找到有關資料量的詳細資訊，並依解決方案查看細目。

如需詳細資訊，您可以閱讀[客戶部落格](http://thoughtsonopsmgr.blogspot.com/2015/09/one-small-footprint-for-server-one.html) \(英文\)，其中顯示評估 OMS 代理程式的資源使用率 (使用量) 之後的結果。

### <a name="q-how-much-network-bandwidth-is-used-by-the-microsoft-management-agent-mma-when-sending-data-to-log-analytics"></a>問： 傳送資料到 Log Analytics 時，Microsoft 管理代理程式 (MMA) 使用多少網路頻寬？

A. 頻寬是關於傳送的資料量的功能。 透過網路傳送資料時，會壓縮資料。

### <a name="q-how-much-data-is-sent-per-agent"></a>問： 每個代理程式會傳送多少資料？

A. 每個代理程式所傳送的資料量取決於：

* 您已啟用的解決方案
* 記錄檔和要收集之效能計數器的數目
* 記錄檔中的資料量

免費定價層是將數個伺服器上架和量測典型資料量的好方法。 [使用量](log-analytics-usage.md) 頁面會顯示整體使用方式。

對於可執行 WireData 代理程式的電腦，請使用下列查詢查看正在傳送的資料量：

```
Type=WireData (ProcessName="C:\\Program Files\\Microsoft Monitoring Agent\\Agent\\MonitoringHost.exe") (Direction=Outbound) | measure Sum(TotalBytes) by Computer
```

## <a name="next-steps"></a>後續步驟
* [開始使用 Log Analytics](log-analytics-get-started.md) 以深入了解 Log Analytics，並幾分鐘內就啟動並執行。
