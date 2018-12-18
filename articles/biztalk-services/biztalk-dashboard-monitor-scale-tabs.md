---
title: BizTalk 服務中的儀表板、監視、調整、設定和混合式連接 | Microsoft Docs
description: 深入了解 BizTalk 服務的控制項及監視效能
services: biztalk-services
documentationcenter: ''
author: MandiOhlinger
manager: anneta
editor: ''
ms.assetid: 7a1815db-0de2-4274-8be0-198c1b077324
ms.service: biztalk-services
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/07/2016
ms.author: mandia
ms.openlocfilehash: 351809cd5f165a863dc02bfadf78fa59cbaabfd7
ms.sourcegitcommit: dcf5f175454a5a6a26965482965ae1f2bf6dca0a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/10/2017
ms.locfileid: "24103356"
---
# <a name="review-the-dashboard-monitor-scale-configure-and-hybrid-connection-tabs"></a>檢閱儀表板、監視器、調整、設定和混合式連線索引標籤

> [!INCLUDE [BizTalk Services is being retired, and replaced with Azure Logic Apps](../../includes/biztalk-services-retirement.md)]

在您建立 BizTalk 服務和部署應用程式之後，您可以變更某些 BizTalk 服務設定和監視應用程式效能。 

> [!INCLUDE [Use APIs to manage MABS](../../includes/biztalk-services-retirement-azure-classic-portal.md)]

這樣會開啟新視窗並顯示下列索引標籤。 本主題說明這些索引標籤。

## <a name="quickstart-quickstartquickstart"></a>快速入門 (![快速入門][Quickstart])
部分 BizTalk 服務版本可能並未提供下列所有選項。 

<table border="1">
    <tr>
        <td><strong>取得工具</strong></td>
        <td>下載 BizTalk 服務 SDK，將 Visual Studio 專案範本安裝到內部部署開發電腦上。 這些範本會建立 <strong>BizTalk 服務</strong> (橋接)，以及可部署至 BizTalk 服務的 <strong>BizTalk 服務成品</strong> (轉換) Visual Studio 專案。
        <br/><br/>
        <a HREF="http://go.microsoft.com/fwlink/p/?LinkID=302335">如何開始使用 Azure BizTalk 服務 SDK</a> 和<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=241589">安裝 Azure BizTalk 服務 SDK</a> 列出開始進行的步驟。
        </td>
    </tr>
    <tr>
        <td><strong>建立夥伴協議</strong></td>
        <td>開啟裝載於 Azure 的 Azure BizTalk 服務入口網站，以加入夥伴並建立 X12、AS2 和 EDIFACT EDI 協議。
        <br/><br/>
        <a HREF="http://go.microsoft.com/fwlink/p/?LinkID=303653">在 BizTalk 服務入口網站上設定 EDI 訊息的元件</a>列出開始進行的步驟。
        </td>
    </tr>

<tr>
        <td><strong>深入了解 BizTalk 服務</strong></td>
        <td>前往<a HREF="http://azure.microsoft.com/documentation/services/biztalk-services/">學習中心</a>以深入了解 Azure BizTalk 服務。</td>
</tr>
</table>


在底部的工作列上，您可以：

<table border="1">

<tr>
<td><strong>管理</strong>應用程式部署</td>
<td>開啟 Azure BizTalk 服務入口網站。 BizTalk 服務入口網站是 EDI 組態的入口，包括加入夥伴及建立 X12、AS2 和 EDIFACT 協議。
<br/><br/>
這與 [快速入門]<strong></strong> 索引標籤上的 [建立夥伴合約]<strong></strong> 相同。
<br/><br/>
<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=303653">在 BizTalk 服務入口網站上設定 EDI 訊息的元件</a>提供 BizTalk 服務入口網站的詳細資訊。</td>
</tr>

<tr>
<td>存取控制命名空間的<strong>連線資訊</strong></td>
<td>選取 [連線資訊] 時會顯示 [存取控制命名空間]、[預設核發者] 和 [預設金鑰]。 您可以複製這些值。
<br/><br/>
您也可以開啟存取控制入口網站。 <a HREF="http://go.microsoft.com/fwlink/p/?LinkID=285670">建立存取控制命名空間</a>提供了存取控制入口網站的詳細資訊。</td>
</tr>

<tr>
<td>儲存體帳戶內的<strong>同步金鑰</strong></td>
<td>當建立儲存體帳戶時，會自動建立主要金鑰和次要金鑰。 這些加密金鑰可控制儲存體帳戶的存取。 BizTalk 服務會自動使用主要金鑰。 <strong>同步金鑰</strong>可讓使用者在主要金鑰和次要金鑰之間切換，而不會中斷 BizTalk 服務。
<br/><br/>
例如，您希望 BizTalk 服務對儲存體帳戶使用新的主要金鑰。 作法：
<br/><br/>
<ol>
<li>選取 BizTalk 服務，再選取 [同步金鑰]<strong></strong>。 選取次要金鑰。 這時，BizTalk 服務就會開始使用次要金鑰。</li>
<li>選取您的儲存體帳戶並重新產生主要金鑰。 記住，BizTalk 服務現在使用次要金鑰。</li>
<li>選取 BizTalk 服務，再選取 [同步金鑰]<strong></strong>。 現在，選取主要金鑰。 這是您重新產生的新主要金鑰。</li>
<li>選取您的儲存體帳戶並重新產生次要金鑰。</li>
</ol>
<br/>
此程序稱為「換用金鑰」。 目的是讓使用者在主要金鑰和次要金鑰之間切換，而不會中斷 BizTalk 服務。</td>
</tr>

<tr>
<td><strong>刪除</strong>應用程式</td>
<td>選取 [刪除] 會移除 BizTalk 服務及已部署到它的所有項目。</td>
</tr>
</table>


## <a name="dashboard"></a>儀表板
部分 BizTalk 服務版本可能並未提供下列所有選項。 

選取 BizTalk 服務名稱時會顯示 [儀表板] 索引標籤。 在 [儀表板] 裡，您可以：

##### <a name="usage-overview-shows-the-number-of-used-hybrid-connections"></a>使用量概觀：顯示已使用的混合式連線數量
資料使用量 (單位為 GB) 也會顯示。 

##### <a name="metric-graph-shows-a-fixed-list-of-performance-metrics"></a>度量圖：顯示固定的效能度量清單。
這些度量提供 BizTalk 服務健全狀況的即時值。 您也可以選擇 [相對] 值或 [絕對] 值，以及圖表上顯示的度量時間範圍 [間隔]。 

如需這些效能度量的說明，請移至本主題的 [可用的度量](#Metrics) 。

##### <a name="quick-glance-lists-your-biztalk-service-properties"></a>快速瀏覽：列出 BizTalk 服務屬性
<table border="1">

<tr>
<td><strong>更新追蹤資料庫認證</strong></td>
<td>變更用於登入追蹤資料庫的使用者名稱和密碼。</td>
</tr>
<tr>
<td><strong>更新 SSL 憑證</strong></td>
<td>可更新 BizTalk 服務來使用不同的 SSL 憑證。 在您<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=302280">建立 BizTalk 服務</a>時，自我簽署 SSL 憑證會自動建立。</td>
</tr>
<tr>
<td><strong>下載憑證</strong></td>
<td>您可以將 BizTalk 服務所使用的 SSL 憑證下載到本機電腦上。</td>
</tr>
<tr>
<td><strong>狀態</strong></td>
<td>顯示 BizTalk 服務的目前狀態。 請參閱 <a HREF="http://go.microsoft.com/fwlink/p/?LinkID=329870">BizTalk 服務：服務狀態圖</a> (英文)。 </td>
</tr>
<tr>
<td><strong>服務 URL</strong></td>
<td>您 BizTalk 服務的 URL。 這與建立您的 BizTalk 服務時所輸入的<strong>網域 URL</strong> 相同。</td>
</tr>
<tr>
<td><strong>公用虛擬 IP (VIP) 位址</strong></td>
<td>此 IP 位址指派給您的 BizTalk 服務。 它用於所有輸入端點，也是輸出流量的來源位址。 此 IP 位址屬於您建立的 BizTalk 服務。 若您刪除 BizTalk 服務，則此 IP 位址會指派給其他 BizTalk 服務。</td>
</tr>
<tr>
<td><strong>ACS 命名空間</strong></td>
<td>向 BizTalk 服務驗證。</td>
</tr>
<tr>
<td><strong>版本</strong></td>
<td>列出建立 BizTalk 服務時所輸入的版本。</td>
</tr>
<tr>
<td><strong>位置</strong></td>
<td>顯示裝載 BizTalk 服務的地理區域。</td>
</tr>
<tr>
<td><strong>建立時間</strong></td>
<td>顯示建立 BizTalk 服務的日期和時間。</td>
</tr>
<tr>
<td><strong>追蹤資料庫</strong></td>
<td>儲存 BizTalk 服務所使用之追蹤資料表的 Azure SQL Database 名稱。 
<br/><br/>
<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=302280">需求說明</a>提供追蹤資料庫的詳細資料。</td>
</tr>
<tr>
<td><strong>監視/封存儲存體</strong></td>
<td>儲存 BizTalk 服務之監視輸出的 Azure 儲存體帳戶名稱。
<br/><br/>
<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=302280">需求說明</a>提供儲存體帳戶的詳細資料。</td>
</tr>
<tr>
<td><strong>訂用帳戶名稱</strong></td>
<td>列出裝載 BizTalk 服務的訂用帳戶名稱。 訂用帳戶會控管存取。</td>
</tr>
<tr>
<td><strong>訂用帳戶識別碼</strong></td>
<td>建立訂用帳戶時，會自動產生訂用帳戶識別碼。 使用 REST API 時，您可能需要輸入訂閱識別碼。</td>
</tr>
</table>

[BizTalk 服務：佈建](http://go.microsoft.com/fwlink/p/?LinkID=302280)列出建立 BizTalk 服務的步驟。

##### <a name="manage-connection-information-sync-keys-and-delete-in-the-task-bar"></a>工作列中的管理、連線資訊、同步金鑰和刪除：
<table border="1">

<tr>
<td><strong>管理</strong>應用程式部署</td>
<td>開啟 Azure BizTalk 服務入口網站。 BizTalk 服務入口網站是 EDI 組態的入口，包括加入夥伴及建立 X12、AS2 和 EDIFACT 協議。
<br/><br/>
這與 [快速入門]<strong></strong> 索引標籤上的 [建立夥伴合約]<strong></strong> 相同。
<br/><br/>
<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=303653">在 BizTalk 服務入口網站上設定 EDI 訊息的元件</a>提供 BizTalk 服務入口網站的詳細資訊。</td>
</tr>
<tr>
<td>存取控制命名空間的<strong>連線資訊</strong></td>
<td>顯示 Access Control 命名空間、預設核發者和預設金鑰值，供您複製。
<br/><br/>
您也可以開啟存取控制入口網站。 此存取控制入口網站的功用就與使用左側導覽窗格中的 [Active Directory] 選項一樣。
<br/><br/>
<a HREF="http://go.microsoft.com/fwlink/p/?LinkID=285670">管理您的 ACS 命名空間</a>提供存取控制入口網站的詳細資訊。</td>
</tr>
<tr>
<td>儲存體帳戶內的<strong>同步金鑰</strong></td>
<td>當建立儲存體帳戶時，會自動建立主要金鑰和次要金鑰。 這些加密金鑰可控制儲存體帳戶的存取。 BizTalk 服務會自動使用主要金鑰。 <strong>同步金鑰</strong>可讓使用者在主要金鑰和次要金鑰之間切換，而不會中斷 BizTalk 服務。
<br/><br/>
例如，您希望 BizTalk 服務對儲存體帳戶使用新的主要金鑰。 作法：
<br/><br/>
<ol>
<li>選取 BizTalk 服務，再選取 [同步金鑰]<strong></strong>。 選取次要金鑰。 這時，BizTalk 服務就會開始使用次要金鑰。</li>
<li>選取您的儲存體帳戶並重新產生主要金鑰。 記住，BizTalk 服務現在使用次要金鑰。</li>
<li>選取 BizTalk 服務，再選取 [同步金鑰]<strong></strong>。 現在，選取主要金鑰。 這是您重新產生的新主要金鑰。</li>
<li>選取您的儲存體帳戶並重新產生次要金鑰。</li>
</ol>
<br/>
此程序稱為「換用金鑰」。 目的是讓使用者在主要金鑰和次要金鑰之間切換，而不會中斷 BizTalk 服務。</td>
</tr>

<tr>
<td><strong>刪除</strong>應用程式</td>
<td>已移除 BizTalk 服務與部署至服務的所有項目。</td>
</tr>
</table>


## <a name="monitor"></a>監視
不適用於免費版本。

選取 BizTalk 服務名稱後，即可使用 [監視] 索引標籤且會顯示下列資訊：

##### <a name="metric-graph-displays-the-selected-performance-metrics"></a>度量圖：顯示已選取的效能度量
這些度量提供 BizTalk 服務健全狀況的即時值。 請選擇要顯示哪些效能度量。 一次最多同時顯示六個效能度量。 

您也可以選擇 [相對] 值或 [絕對] 值，以及所顯示的度量時間範圍 [間隔]。 

##### <a name="to-remove-or-display-metrics-in-the-graph"></a>在圖表中移除或顯示度量：
1. 選取 [ **監視** ] 索引標籤。
2. 選取工作列中的 [加入度量]：  
   ![選取 [新增度量]][AddMetrics]
3. 勾選您要顯示的效能度量。
4. 選取勾選記號以回到 [ **監視** ] 索引標籤。
5. 選取度量旁邊的圓圈，將該度量的值顯示在圖表中。  
   
    例如，[CPU 使用率] 度量呈現灰色，其輸出不會出現在圖表中：  
   ![CPU 使用量度量呈現灰色][GrayedMetric]  
   
    選取灰色的圓圈以啟用 [CPU 使用率] 度量，將其輸出顯示在圖表中：  
   ![CPU 使用量度量已啟用][EnabledMetric]
6. 若要從顯示圖表和清單中移除度量，請選取工作列中的 [ **移除度量** ]。 若要將度量加回到清單中，請選取工作列的 [加入度量]，勾選度量，然後選取勾選記號以回到 [監視] 索引標籤。選取灰色圓圈以啟用度量。

## <a name="Metrics"></a>可用的度量
以下是可用的效能計數器/度量：

<table border="1">

<tr>
<td><strong>往返延遲</strong></td>
<td>顯示從收到訊息起，到 BizTalk 服務跨所有橋接器完全處理完為止，處理訊息所花費的平均時間，以毫秒為單位 (ms)。 只會計算成功處理的訊息。<br/><br/>
發生下列事件時會建立時間戳記：
<ul>
<li>訊息進入閘道</li>
<li>訊息已路由傳送至目的地</li>
<li>收到目的地回應</li>
<li>目的地認可回應已傳送至閘道</li>
</ul>
<br/>
此度量顯示下列計算的結果：
<br/><br/>
[目的地認可回應已傳送至閘道] - [訊息進入閘道]</td>
</tr>
<tr>
<td><strong>來源失敗</strong></td>
<td>顯示從來源端點提取訊息時，BizTalk 服務無法處理的訊息總數。</td>
</tr>
<tr>
<td><strong>CPU 使用率</strong></td>
<td>列出所有角色執行個體的平均 % 處理器時間。</td>
</tr>
<tr>
<td><strong>處理延遲</strong></td>
<td>顯示 BizTalk 服務跨所有橋接器來處理某個訊息所花費的平均時間，不包括在目的地所花費的時間，以毫秒為單位 (ms)。 只會計算成功處理的訊息。<br/><br/>
發生下列每一個事件時會建立時間戳記：

<ul>
<li>訊息進入閘道</li>
<li>訊息已路由傳送至目的地</li>
<li>收到目的地回應</li>
<li>目的地認可回應已傳送至閘道</li>
</ul>
<br/>此度量顯示下列計算的結果：<br/><br/>
[目的地認可回應已傳送至閘道] - [訊息進入閘道] - [收到目的地回應] + [訊息已路由傳送至目的地]</td>
</tr>
<tr>
<td><strong>處理時失敗</strong></td>
<td>顯示在一段時間內，BizTalk 服務跨所有橋接器處理訊息時失敗的訊息總數。</td>
</tr>
<tr>
<td><strong>已傳送的訊息</strong></td>
<td>顯示在一段時間內，BizTalk 服務跨所有橋接器傳送的訊息總數。 當管線傳送的訊息抵達路由目的地時，此度量會遞增。 此度量不表示已成功處理訊息。<br/><br/>
在要求回覆案例中，當路由目的地將收到認可傳回至管線時，此度量會遞增。</td>
</tr>
<tr>
<td><strong>已接收的訊息</strong></td>
<td>顯示在一段時間內，BizTalk 服務跨所有橋接器接收的訊息總數。 當管線收到新的訊息時，此度量會遞增。</td>
</tr>
<tr>
<td><strong>處理中的訊息</strong></td>
<td>顯示 BizTalk 服務在一段時間內正在處理的訊息總數。</td>
</tr>
<tr>
<td><strong>已處理的訊息</strong></td>
<td>顯示在一段時間內，BizTalk 服務跨所有橋接器成功處理的訊息總數。 當訊息由管線成功接收且成功地路由傳送至目的地時，此度量會遞增。</td>
</tr>
</table>


## <a name="scale"></a>調整
在 [調整] 索引標籤中，您可以增加或減少 BizTalk 服務使用的單位數。 依預設已設定一個單位。 您可以將更多單位加入至 BizTalk 服務。 增加規模時，也會增量輸送量。 資源數量也會增加，包括部署的橋接器、協議、LOB 連接和處理能力。 例如，將規模從 1 單位增加為 2 單位。 在此情況下，您可以部署兩倍的橋接器、兩倍的協議、兩倍的 LOB 連接，以及兩倍的處理能力。

有些 BizTalk 版本不提供調整選項。 在此情況下，只允許一個單位。 若要判斷您的版本可調整的單位數，請參閱「 [BizTalk 服務：版本圖表](biztalk-editions-feature-chart.md)」。

增加單位數可能會影響定價。 如果增加單位，選取 **[儲存]** 會出現訊息，告知您計費是否受影響。 您可以選擇繼續。 增加單位數時，BizTalk 服務狀態會從 [作用中] 變更為 [正在更新]。 在更新狀態下，BizTalk 服務會繼續執行。

[BizTalk 服務：版本圖表](biztalk-editions-feature-chart.md) 定義「單位」。

## <a name="configure"></a>設定
不適用於混合式連線。

將 [備份狀態] 設為 [無] 或 [自動]。 設為 [無] 時不會自動建立備份。 設為 [自動] 時需設定備份位置、備份頻率和備份檔案的保留時間。 

[BizTalk 服務：備份與還原](biztalk-backup-restore.md) 提供詳細資料。 

## <a name="HybridConnections"></a>混合式連線
混合式連線可將 Azure 應用程式 (例如 Azure App Service 中的 Web Apps 或 Mobile Apps) 連線到使用靜態 TCP 連接埠的內部部署資源，例如 SQL Server、MySQL、HTTP Web API 及大部分的自訂 Web 服務。 在 BizTalk 服務中建立混合式連線。

若要在 Azure BizTalk 服務內建立或管理混合式連線，請參閱 [混合式連線](integration-hybrid-connection-overview.md)。

## <a name="next"></a>下一頁
現在，您已熟悉不同的索引標籤，您可以繼續深入了解 Azure BizTalk 服務的功能：

* [BizTalk 服務：節流](biztalk-throttling-thresholds.md)  
* [BizTalk 服務：簽發者名稱和簽發者金鑰](biztalk-issuer-name-issuer-key.md)  
* [BizTalk 服務：備份與還原](biztalk-backup-restore.md)

## <a name="see-also"></a>另請參閱
* [VNet](integration-hybrid-connection-overview.md)  
* [BizTalk 服務：開發人員、基本、標準和高級版本圖表](biztalk-editions-feature-chart.md)  
* [BizTalk 服務：佈建](biztalk-provision-services.md)  
* [BizTalk 服務：BizTalk 服務狀態圖](biztalk-service-state-chart.md)  
* [如何開始使用 Azure BizTalk 服務 SDK](http://go.microsoft.com/fwlink/p/?LinkID=302335)

[Quickstart]: ./media/biztalk-dashboard-monitor-scale-tabs/QuickStartIcon.png
[AddMetrics]: ./media/biztalk-dashboard-monitor-scale-tabs/WABS_AddMetrics.png
[GrayedMetric]: ./media/biztalk-dashboard-monitor-scale-tabs/WABS_GrayedMetric.png
[EnabledMetric]: ./media/biztalk-dashboard-monitor-scale-tabs/WABS_EnabledMetric.png

