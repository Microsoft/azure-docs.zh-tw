---
title: 深入了解 BizTalk 服務各版本的功能 | Microsoft Docs
description: 比較 BizTalk 服務各版本的功能：免費、開發人員、基本、標準和高級。 MABS，WABS。
services: biztalk-services
documentationcenter: ''
author: MandiOhlinger
manager: anneta
editor: ''
ms.assetid: c589629f-06b1-44bb-b8ca-1db71826ea59
ms.service: biztalk-services
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 11/07/2016
ms.author: mandia
ms.openlocfilehash: 718b57a801a9ba62a0154ae42da2ac0c0741f203
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22706643"
---
# <a name="biztalk-services-editions-chart"></a>BizTalk 服務：版本圖表

> [!INCLUDE [BizTalk Services is being retired, and replaced with Azure Logic Apps](../../includes/biztalk-services-retirement.md)]

Azure BizTalk 服務提供數個版本。 使用本文決定最適合您的案例和商務需求的版本。

## <a name="compare-the-editions"></a>比較版本
**免費 (預覽)**

功能包括建立和管理混合式連線。 混合式連線能讓您將 Azure 網站輕鬆連線到內部部署系統，例如 SQL Server。

**開發人員**

包括混合式連線、EAI 和 EDI 訊息處理與易於使用的交易夥伴管理入口網站、支援一般 EDI 結構描述，以及透過 X12 和 AS2 的豐富 EDI 處理。 建立一般 EAI 案例，以利用任何 HTTP/S、REST、FTP、WCF 和 SFTP 通訊協定連接雲端中的服務，來讀取和寫入訊息。  利用易於使用的 SAP、Oracle eBusiness、Oracle DB、Siebel 和 SQL Server 介面卡，來使用內部部署 LOB 系統的連線。 在開發人員為主的環境中，使用 Visual Studio 工具輕鬆開發和部署。 僅限沒有服務等級協定 (SLA) 的開發和測試目的。

**基本**

包括大部分的開發人員功能，加上混合式連線、EAI 橋接器、EDI 協議以及 BizTalk Adapter Pack 連線。 此版本也提供高可用性，以及透過服務等級協定 (SLA) 進行擴充的選項。

**標準**

包括所有基本功能，加上混合式連線、EAI 橋接器、EDI 協議以及 BizTalk Adapter Pack 連線。 此版本也提供高可用性，以及透過服務等級協定 (SLA) 進行擴充的選項。

**高級**

包括所有標準功能，加上混合式連線、EAI 橋接器、EDI 協議以及 BizTalk Adapter Pack 連線。 此版本也包含封存、高可用性，以及透過服務等級協定 (SLA) 進行擴充的選項。

## <a name="editions-chart"></a>版本圖表
下表列出差異。

<table border="1">
<tr bgcolor="FAF9F9">
        <th></th>
        <th>免費 (預覽)</th>
        <th>開發人員</th>
        <th>基本</th>
        <th>標準</th>
        <th>進階</th>
</tr>

<tr>
<td><strong>起始價格</strong></td>
<td colspan="5"><a HREF="http://go.microsoft.com/fwlink/p/?LinkID=304011">Azure BizTalk 服務價格</a> <br/><br/> <a HREF="http://azure.microsoft.com/pricing/calculator/?scenario=full">Azure 價格計算機</a></td>
</tr>
<tr>
<td><strong>預設最小組態</strong></td>
<td>1 個免費單位</td>
<td>1 個開發人員單位</td>
<td>1 個基本單位</td>
<td>1 個標準單位</td>
<td>1 個高級單位</td>
</tr>
<tr>
<td><strong>調整</strong></td>
<td>無調整</td>
<td>無調整</td>
<td>是，增加 1 個基本單位</td>
<td>是，增加 1 個標準單位</td>
<td>是，增加 1 個高級單位</td>
</tr>
<tr>
<td><strong>允許的最大相應放大</strong></td>
<td>無調整</td>
<td>無調整</td>
<td>最多 8 個單位</td>
<td>最多 8 個單位</td>
<td>最多 8 個單位</td>
</tr>
<tr>
<td><strong>每個單位的 EAI 橋接器</strong></td>
<td>未包括</td>
<td>25</td>
<td>25</td>
<td>125</td>
<td>500</td>
</tr>
<tr>
<td><strong>EDI、AS2</strong>
<br/><br/>
包括 TPM 協議</td>
<td>未包括</td>
<td>已包括。 每個單位 10 個協定。</td>
<td>已包括。 每個單位 50 個協定。</td>
<td>已包括。 每個單位 250 個協定。</td>
<td>已包括。 每個單位 1000 個協定。</td>
</tr>
<tr>
<td><strong>每個單位的混合式連線</strong></td>
<td>5</td>
<td>5</td>
<td>10</td>
<td>50</td>
<td>100</td>
</tr>
<tr>
<td><strong>每個單位的混合式連線資料傳輸 (GB)</strong></td>
<td>5</td>
<td>5</td>
<td>50</td>
<td>250</td>
<td>500</td>
</tr>
<tr>
<td><strong>內部部署 LOB 系統的 BizTalk 介面卡服務連線</strong></td>
<td>未包括</td>
<td>1 個連接</td>
<td>2 個連接</td>
<td>5 個連接</td>
<td>25 個連接</td>
</tr>
<tr>
<td align="left"><strong>支援的通訊協定/系統：</strong>
<ul>
<li>HTTP</li>
<li>HTTPS</li>
<li>FTP</li>
<li>SFTP</li>
<li>WCF</li>
<li>服務匯流排 (SB)</li>
<li>Azure Blob</li>
<li>REST API</li>
</ul>
</td>
<td>未包括</td>
<td>已包括</td>
<td>已包括</td>
<td>已包括</td>
<td>已包括</td>
</tr>
<tr>
<td><strong>高可用性</strong>
<br/><br/>
有關服務等級協定 (SLA)，請參閱 <a HREF="http://go.microsoft.com/fwlink/p/?LinkID=304011">Azure BizTalk 服務價格</a>。
</td>
<td>未包括</td>
<td>未包括</td>
<td>已包括</td>
<td>已包括</td>
<td>已包括</td>
</tr>
<tr>
<td><strong>備份與還原</strong></td>
<td>未包括</td>
<td>已包括</td>
<td>已包括</td>
<td>已包括</td>
<td>已包括</td>
</tr>
<tr>
<td><strong>追蹤</strong></td>
<td>未包括</td>
<td>已包括</td>
<td>已包括</td>
<td>已包括</td>
<td>已包括</td>
</tr>
<tr>
<td><strong>封存</strong><br/><br/>
包括收據的不可否認性 (NRR) 和下載追蹤的訊息</td>
<td>未包括</td>
<td>已包括</td>
<td>未包括</td>
<td>未包括</td>
<td>已包括</td>
</tr>
<tr>
<td><strong>使用自訂程式碼</strong></td>
<td>未包括</td>
<td>已包括</td>
<td>已包括</td>
<td>已包括</td>
<td>已包括</td>
</tr>
<tr>
<td><strong>使用轉換，包括自訂 XSLT</strong></td>
<td>未包括</td>
<td>已包括</td>
<td>已包括</td>
<td>已包括</td>
<td>已包括</td>
</tr>
</table>

> [!NOTE]
> 針對硬體故障時的備援能力，高可用性即代表在單一 BizTalk 單位內具有多個 VM。
> 
> 

## <a name="faqs"></a>常見問題集
#### <a name="what-is-a-biztalk-unit"></a>何謂 BizTalk 單位？
「單位」為 Azure BizTalk 服務部署的原子層級。 每個版本都隨附一個具有不同計算容量和記憶體的單位。 例如，基本單位的計算能力超過開發人員、標準的計算能力超過基本，依此類推。 當調整 BizTalk 服務時，您是根據單位調整。

#### <a name="what-is-the-difference-between-biztalk-services-and-azure-biztalk-vm"></a>BizTalk 服務與 Azure BizTalk VM 有何差異？
BizTalk 服務提供真正的平台即服務 (PaaS) 架構，可在雲端中組建整合方案。 您可以利用 PaaS 模型，完全專注於應用程式邏輯，並將所有基礎結構管理留給 Microsoft，包括：

* 不需要管理或修補虛擬機器。
* Microsoft 確保了可用性。
* 您可以根據需求調整，只需透過 Azure 入口網站要求更多或更少容量。

虛擬機器上的 BizTalk 伺服器提供基礎架構即服務 (IaaS) 架構。 您可以建立虛擬機器，並設定它們完全和您的內部部署環境一樣，這使得在雲端中執行現有的應用程式更為容易，而不需變更程式碼。 使用 IaaS 時，您仍須負責設定虛擬機器、管理虛擬機器 (例如，安裝軟體和修補程式)，以及架構高可用性的應用程式。

如果您正在查看如何組建新的整合方案，以將您的基礎結構管理努力降至最低，請使用 BizTalk 服務。 如果您正在查看如何快速移轉現有的 BizTalk 方案，或尋找根據需求環境，以開發並測試 BizTalk 伺服器應用程式，請在 Azure 虛擬機器上使用 BizTalk 伺服器。

#### <a name="what-is-the-difference-between-biztalk-adapter-service-and-hybrid-connections"></a>BizTalk 配接器服務與混合式連線有何差異？
BizTalk 配接器服務是由 Azure BizTalk 服務使用。 BizTalk 配接器服務使用 BizTalk Adapter Pack 來連線至內部部署的企業營運 (LOB) 系統。 混合式連線則可讓您將 Azure 應用程式 (例如 Azure App Service 的 Web Apps 和 Azure 行動服務) 連線至內部部署資源，既輕鬆又方便。

#### <a name="what-does-hybrid-connection-data-transfer-gb-per-unit-mean-is-this-per-minutehourdayweekmonth-what-happens-when-the-limit-is-reached"></a>「每單位混合式連線資料傳輸 (GB)」代表什麼意思？ 是指每分鐘/小時/日/週/月嗎？ 達到限制時會發生什麼事？
混合式連線每單位的成本要視 BizTalk 服務版本而定。 簡言之，成本取決於您傳輸的資料量。 例如，每天傳輸 10 GB 資料的成本低於每天傳輸 100 GB 的成本。 使用 BizTalk 服務的 [定價計算機](https://azure.microsoft.com/pricing/calculator/?scenario=full) 來決定特定的成本。 一般而言，系統會每日強制執行該限制。 如果超過這個限制，每 GB 會收取 $1 的超額費。

#### <a name="when-i-create-an-agreement-in-biztalk-services-why-does-the-number-of-bridges-go-up-by-two-instead-of-just-one"></a>當我在 BizTalk 服務上建立協定時，為什麼橋接器的數目增加二，而不是只增加一？
每個協定由兩個不同協定組成，即傳送端通訊橋接器和接收端通訊橋接器。

#### <a name="what-happens-when-i-hit-the-quota-limit-on-the-number-of-bridges-or-agreements"></a>當我達到橋接器或合約數目的配額限制時將發生什麼情況？
您將無法部署任何新的橋接器，或建立任何新的合約。 為了能夠部署更多，您需要相應增加為更多的 BizTalk 服務單位，或升級至更高版本。

#### <a name="how-do-i-migrate-from-one-tier-of-biztalk-services-to-another"></a>如何從 BizTalk 服務的某層移轉至另一層？
免費版本無法移轉或「調高」到另一層，而且無法備份及還原到另一層。 如果您需要另一層，請使用新的服務層建立新的 BizTalk 服務。 必須在新的 BizTalk 服務中重建任何使用免費版本建立的成品 (包括混合式連線)。 

對於其餘的版本，使用備份與還原將您的成品從某一層移轉至另一層。 例如，在標準層中備份成品，然後將它們還原至高階層。 [BizTalk 服務：備份與還原](biztalk-backup-restore.md) 說明支援的移轉路徑和備份的成品清單。 請注意，混合式連線無法備份。 備份及還原至新的服務層之後，您會重建混合式連線。  

#### <a name="is-the-biztalk-adapter-service-included-in-the-service-how-do-i-receive-the-software"></a>服務中是否包括 BizTalk 介面卡服務？ 如何接收軟體？
是，BizTalk 介面卡服務 (含 BizTalk 介面卡套件) 隨附於 Azure BizTalk 服務 SDK [下載](http://www.microsoft.com/download/details.aspx?id=39087)(英文)。

## <a name="next-steps"></a>後續步驟
若要在 Azure 入口網站中建立 Azure BizTalk 服務，請移至 [BizTalk 服務：使用 Azure 入口網站進行佈建](biztalk-provision-services.md)。 若要開始建立應用程式，請移至 [Azure BizTalk 服務](http://go.microsoft.com/fwlink/p/?LinkID=235197)(英文)。

## <a name="additional-resources"></a>其他資源
* [BizTalk 服務：使用 Azure 入口網站進行佈建](biztalk-provision-services.md)<br/>
* [BizTalk 服務：佈建狀態圖](biztalk-service-state-chart.md)<br/>
* [BizTalk 服務：儀表板、監視和調整索引標籤](biztalk-dashboard-monitor-scale-tabs.md)<br/>
* [BizTalk Services: Backup and restore](biztalk-backup-restore.md)<br/>
* [BizTalk 服務：節流](biztalk-throttling-thresholds.md)<br/>
* [BizTalk 服務：簽發者名稱和簽發者金鑰](biztalk-issuer-name-issuer-key.md)<br/>
* [如何開始使用 Azure BizTalk 服務 SDK](http://go.microsoft.com/fwlink/p/?LinkID=302335)<br/>

