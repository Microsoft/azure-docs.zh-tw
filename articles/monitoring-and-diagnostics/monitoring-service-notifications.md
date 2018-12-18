---
title: 什麼是 Azure 服務健康情況通知？
description: 藉由服務健康情況通知，您可以檢視由 Microsoft Azure 發佈的服務健康情況訊息。
author: dkamstra
services: monitoring
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 4/12/2017
ms.author: dukek
ms.component: activitylog
ms.openlocfilehash: d6a87b17041c4ce6cf41da863354ef5a2a37141c
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35264437"
---
# <a name="view-service-health-notifications-by-using-the-azure-portal"></a>使用 Azure 入口網站檢視服務健康情況通知

Azure 會發佈服務健康情況通知，其中包含您訂用帳戶下資源的相關資訊。 這些通知是活動記錄事件的子類別，在活動記錄上也找得到。 根據不同類別，服務健康情況通知可以是告知性質或是可操作的性質。

服務健康情況通知有多種類別：  

- **需採取動作：** Azure 可能注意到您的帳戶中發生異常狀況，並協助您解決這個問題。 Azure 會傳送通知給您，其中會詳述您需要採取的動作，或 Azure 工程或支援的連絡方式。  
- **協助復原︰** 事件已發生，且工程師確認您仍受到影響。 Azure 工程人員需要直接與您一起將服務還原為徹底健康的狀態。  
- **附帶事件 (Incident)︰** 影響服務的事件 (event) 目前正在影響您訂用帳戶下一個或多個資源。  
- **維護︰** 可能會影響您訂用帳戶下一個或多個資源的計劃性維護活動。  
- **資訊：** 潛在的最佳化作業，可能有助於提升資源使用效率。 
- **安全性︰** 緊急的安全性相關資訊，關係到 Azure 上執行的解決方案。

每個服務健康狀態通知皆會包含關於範圍以及資源所受影響的詳細資料。 詳細資料包括：

屬性名稱 | 說明
-------- | -----------
通道 | 為下列其中一個值：**Admin** 或 **Operation**。
correlationId | 通常是字串格式的 GUID。 屬於相同動作的事件通常會共用同一個 correlationId。
eventDataId | 事件的唯一識別碼。
eventName | 事件的標題。
層級 | 事件的層級
resourceProviderName | 受影響資源的資源提供者名稱。
resourceType| 受影響資源的資源類型。
子狀態 | 通常是對應 REST 呼叫的 HTTP 狀態碼，但也可以包含其他描述子狀態的字串。 例如：確定 (HTTP 狀態碼︰200)，已建立 (HTTP 狀態碼︰201)、接受 (HTTP 狀態碼︰202)、沒有內容 (HTTP 狀態碼︰204)、不正確的要求 (HTTP 狀態碼︰400)、找不到 (HTTP 狀態碼︰404)，衝突 (HTTP 狀態碼︰409)、內部伺服器錯誤 (HTTP 狀態碼︰500)、服務無法使用 (HTTP 狀態碼︰503) 和閘道逾時 (HTTP 狀態碼︰504)。
eventTimestamp | Azure 服務處理要求 (與事件相對應) 而產生事件時的時間戳記。
submissionTimestamp | 當事件變成可供查詢時的時間戳記。
subscriptionId | 記錄此事件的 Azure 訂用帳戶。
status | 字串，描述作業的狀態。 常見的值包括︰**Started**、**In Progress**、**Succeeded**、**Failed**、**Active** 和 **Resolved**。
operationName | 作業的名稱。
category | 此屬性一律為 **ServiceHealth**。
ResourceId | 受影響資源的資源識別碼。
Properties.title | 此通訊的當地語系化標題。 預設語言為英文。
Properties.communication | 與 HTML 標記通訊的詳細資料 (已當地語系化)。 預設語言為英文。
Properties.incidentType | 為下列其中一個值：**ActionRequired**、**Information**、**Incident**、**Maintenance** 或 **Security**。
Properties.trackingId | 與此事件 (event) 相關聯的附帶事件 (Incident)。 可用此屬性讓與附帶事件 (Incident) 有關的事件 (event) 相關聯。
Properties.impactedServices | 逸出的 JSON blob，描述受到附帶事件 (Incident) 影響的服務和區域。 屬性包含服務清單 (每一份都有 **ServiceName**) 和受影響區域清單 (每一份都有 **RegionName**)。
Properties.defaultLanguageTitle | 英文的通訊。
Properties.defaultLanguageContent | 英文的通訊，如 html 標記或純文字。
Properties.stage | **Incident** 和 **Security** 的可能值為 **Active**、**Resolved** 或 **RCA**。 針對 **ActionRequired** 或 **Information**，唯一的值為 **Active**。 **Maintenance** 的可能值︰**Active**、**Planned**、**InProgress**、**Canceled**、**Rescheduled**、**Resolved** 或 **Complete**。
Properties.communicationId | 與此事件相關聯的通訊。

### <a name="details-on-service-health-level-information"></a>服務健康情況層級資訊的詳細資料
  <ul>
    <li><b>所需的動作</b> (properties.incidentType == ActionRequired) <dl>
            <dt>資訊</dt>
            <dd>避免影響現有服務所需的系統管理員動作</dd>
        </dl>
    </li>
    <li><b>維護</b> (properties.incidentType == Maintenance) <dl>
            <dt>警告</dt>
            <dd>緊急維護<dd>
            <dt>資訊</dt>
            <dd>標準的計劃性維護</dd>
        </dl>
    </li>
    <li><b>資訊</b> (properties.incidentType == Information) <dl>
            <dt>資訊</dt>
            <dd>系統管理員可能必須避免影響現有服務</dd>
        </dl>
    </li>
    <li><b>安全性</b> (properties.incidentType == Security) <dl>
            <dt>錯誤</dt>
            <dd>跨多個區域存取多個服務的廣泛問題會影響廣泛的客戶。</dd>
            <dt>警告</dt>
            <dd>存取特定服務及/或特定區域的問題會影響一部分客戶。</dd>
            <dt>資訊</dt>
            <dd>影響管理作業和/或延遲的問題，不會影響服務可用性。</dd>
        </dl>
    </li>
    <li><b>服務問題</b> (properties.incidentType == Incident) <dl>
            <dt>錯誤</dt>
            <dd>跨多個區域存取多個服務的廣泛問題會影響廣泛的客戶。</dd>
            <dt>警告</dt>
            <dd>存取特定服務及/或特定區域的問題會影響一部分客戶。</dd>
            <dt>資訊</dt>
            <dd>影響管理作業和/或延遲的問題，不會影響服務可用性。</dd>
        </dl>
    </li>
  </ul>

## <a name="view-your-service-health-notifications-in-the-azure-portal"></a>在 Azure 入口網站中檢視您的服務健康情況通知
1.  在 [Azure 入口網站](https://portal.azure.com)中，選取 [監視]。

    ![Azure 入口網站功能表的螢幕擷取畫面與選取的監視器](./media/monitoring-service-notifications/home-monitor.png)

    Azure 監視器會將您所有的監視設定和資料結合成一個彙總檢視。 它會先開啟 [活動記錄檔]  區段。

3.  選取 [警示] 。

    ![監視器活動記錄的螢幕擷取畫面與選取的警示](./media/monitoring-service-notifications/service-health-summary.png)
4. 選取 [+新增活動記錄警示] 並設定警示，以確保您會在未來收到服務通知。 如需詳細資訊，請參閱[建立服務通知的活動記錄警示](monitoring-activity-log-alerts-on-service-notifications.md)。

## <a name="next-steps"></a>後續步驟
每當[發佈服務健康情況通知時，即接收警示通知](monitoring-activity-log-alerts-on-service-notifications.md)。  
深入了解[活動記錄警示](monitoring-activity-log-alerts.md)。
