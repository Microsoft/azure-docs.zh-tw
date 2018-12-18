---
title: Azure Log Analytics 中的 DNS 分析解決方案 | Microsoft Docs
description: 在 Log Analytics 中設定並使用 DNS 分析解決方案，以收集關於 DNS 基礎結構在安全性、效能及作業方面的深入解析。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: f44a40c4-820a-406e-8c40-70bd8dc67ae7
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/20/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 56ad633807a59eaa7ae75dc334aa630fb3bf94a3
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39436721"
---
# <a name="gather-insights-about-your-dns-infrastructure-with-the-dns-analytics-preview-solution"></a>收集搭配 DNS 分析預覽版解決方案使用 DNS 基礎結構的深入解析

![DNS 分析符號](./media/log-analytics-dns/dns-analytics-symbol.png)

本文描述如何在 Log Analytics 中設定並使用 Azure DNS 分析解決方案，以收集關於 Azure DNS 基礎結構在安全性、效能及作業方面的深入解析。

DNS 分析可協助您︰

- 指出嘗試解析惡意網域名稱的用戶端。
- 指出過時的資源記錄。
- 指出經常查詢的網域名稱和 Talkative DNS 用戶端。
- 檢視 DNS 伺服器上的要求負載。
- 檢視動態 DNS 註冊失敗。

此解決方案會收集、分析 Windows DNS 分析和稽核記錄，並將它與 DNS 伺服器中的其他相關資料關聯。

## <a name="connected-sources"></a>連接的來源

下表描述此方案支援的連線來源：

| **連線的來源** | **支援** | **說明** |
| --- | --- | --- |
| [Windows 代理程式](log-analytics-windows-agent.md) | 是 | 此解決方案會收集來自 Windows 代理程式的 DNS 資訊。 |
| [Linux 代理程式](log-analytics-linux-agents.md) | 否 | 此解決方案不會收集來自直接 Linux 代理程式的 DNS 資訊。 |
| [System Center Operations Manager 管理群組](log-analytics-om-agents.md) | 是 | 此解決方案會收集來自連線 Operations Manager 管理群組的代理程式之中的 DNS 資訊。 Operations Manager 代理程式不需要直接連線到 Operations Management Suite。 資料會從管理群組轉送至 Operations Management Suite 存放庫。 |
| [Azure 儲存體帳戶](log-analytics-azure-storage.md) | 否 | 此解決方案沒有使用 Azure 儲存體。 |

### <a name="data-collection-details"></a>資料收集詳細資料

此解決方案會從已安裝 Log Analytics 代理程式的 DNS 伺服器收集 DNS 清查和 DNS 事件相關資料。 這項資料會再上傳至 Log Analytics，並顯示在解決方案儀表板中。 清查相關資料 (例如 DNS 伺服器數目、區域和資源記錄) 的收集方式是執行 DNS PowerShell Cmdlet。 此資料每兩天會更新一次。 事件相關資料是以接近即時的方式，從 Windows Server 2012 R2 增強的 DNS 記錄與診斷功能所提供的[分析和稽核記錄](https://technet.microsoft.com/library/dn800669.aspx#enhanc)進行收集。

## <a name="configuration"></a>組態

請使用下列資訊來設定此解決方案：

- 在每部要監視的 DNS 伺服器上，都必須安裝 [Windows](log-analytics-windows-agent.md) 或 [Operations Manager](log-analytics-om-agents.md) 代理程式。
- 您可以從 [Azure Marketplace](https://aka.ms/dnsanalyticsazuremarketplace) 將 DNS 分析解決方案新增至您的 Operations Management Suite 工作區。 您也可以使用[從方案庫新增 Log Analytics 解決方案](log-analytics-add-solutions.md)中所述的程序。

不需要進一步設定，此解決方案就會開始收集資料。 不過，您可以使用下列組態來自訂資料收集。

### <a name="configure-the-solution"></a>設定方案

在解決方案儀表板中，按一下 [組態] 以開啟 [DNS 分析組態] 頁面。 您可以進行兩種類型的組態變更︰

- **列入白名單的網域名稱**。 該解決方案不會處理所有查閱查詢。 它會維護一份網域名稱尾碼的白名單。 若查閱查詢解析為符合此允許清單中之網域名稱尾碼的網域名稱，此解決方案就不會處理它們。 不處理列入允許清單的網域名稱，有助於最佳化傳送至 Log Analytics 的資料。 預設白名單包含熱門的公用網域名稱，例如 www.google.com 和 www.facebook.com。 您可以捲動來檢視完整的預設清單。

 您可以修改清單，將您想要檢視查閱深入解析的任何網域名稱尾碼加以新增。 您也可以將您不想要檢視查閱深入解析的任何網域名稱尾碼加以移除。

- **Talkative 用戶端閾值**。 超過查閱要求數目閾值的 DNS 用戶端，在 [DNS 用戶端] 刀鋒視窗中會反白顯示。 預設閾值為 1,000。 您可以編輯閾值。

    ![列入白名單的網域名稱](./media/log-analytics-dns/dns-config.png)

## <a name="management-packs"></a>管理組件

如果您使用 Microsoft Monitoring Agent 連線到您的 Operations Management Suite 工作區，則會安裝下列管理組件：

- Microsoft DNS 資料收集器智慧套件 (Microsft.IntelligencePacks.Dns)

如果 Operations Manager 管理群組已連線到 Operations Management Suite 工作區，當您新增此方案時，下列管理組件會安裝在 Operations Manager 中。 這些管理組件不需要任何組態或維護：

- Microsoft DNS 資料收集器智慧套件 (Microsft.IntelligencePacks.Dns)
- Microsoft System Center Advisor DNS 分析組態 (Microsoft.IntelligencePack.Dns.Configuration)

如需有關方案管理組件如何更新的詳細資訊，請參閱 [將 Operations Manager 連接到 Log Analytics](log-analytics-om-agents.md)。

## <a name="use-the-dns-analytics-solution"></a>使用 DNS 分析解決方案

本節說明所有儀表板函式及其使用方式。

將解決方案新增至您的工作區之後，[Operations Management Suite 概觀] 頁面上的解決方案圖格會提供您 DNS 基礎結構的快速摘要。 它包含收集資料所在的 DNS 伺服器之數目。 它也包含用戶端在過去 24 小時內所提出的解決惡意網域之要求數目。 當您按一下圖格時，方案儀表板隨即開啟。

![DNS 分析圖格](./media/log-analytics-dns/dns-tile.png)

### <a name="solution-dashboard"></a>解決方案儀表板

解決方案儀表板會顯示解決方案的各種功能的摘要資訊。 它也包含鑑識分析和診斷之詳細檢視的連結。 根據預設，會顯示過去七天的資料。 您可以使用**日期-時間選取控制項**來變更日期與時間範圍，如下圖所示：

![時間選取控制項](./media/log-analytics-dns/dns-time.png)

解決方案儀表板會顯示下列刀鋒視窗：

**DNS 安全性**。 報告嘗試與惡意網域進行通訊的 DNS 用戶端。 藉由使用 Microsoft 威脅情報摘要，DNS 分析可以偵測嘗試存取惡意網域的用戶端 IP。 在許多情況下，受惡意程式碼感染的裝置會透過解析惡意程式碼網域名稱「撥出」至惡意網域的「命令與控制項」中心。

![[DNS 安全性] 刀鋒視窗](./media/log-analytics-dns/dns-security-blade.png)

當您按一下清單中的用戶端 IP 時，記錄搜尋就會開啟，其中顯示個別查詢的查閱詳細資料。 在下列範例中，DNS 分析偵測到透過 [IRCbot](https://www.microsoft.com/security/portal/threat/encyclopedia/entry.aspx?Name=Win32/IRCbot) 進行的通訊：

![顯示 ircbot 的記錄搜尋結果](./media/log-analytics-dns/ircbot.png)

此資訊可協助您識別︰

- 起始通訊的用戶端 IP。
- 解析為惡意 IP 的網域名稱。
- 網域名稱解析成的 IP 位址。
- 惡意 IP 位址。
- 問題的嚴重性。
- 將惡意 IP 列入黑名單的原因。
- 偵測時間。

**查詢的網域**。 提供您環境中的 DNS 用戶端最常查詢的網域名稱。 您可以檢視查詢的所有網域名稱清單。 您也可以在記錄搜尋中向下切入至特定網域名稱的查閱要求詳細資料。

![[查詢的網域] 刀鋒視窗](./media/log-analytics-dns/domains-queried-blade.png)

**DNS 用戶端**。 報告違反閾值的用戶端在所選時間週期內的查詢數目。 您可以在記錄搜尋中檢視所有 DNS 用戶端的清單和它們所執行查詢的詳細資料。

![[DNS 用戶端] 刀鋒視窗](./media/log-analytics-dns/dns-clients-blade.png)

**動態 DNS 註冊**。 報告名稱註冊失敗。 位址[資源記錄](https://en.wikipedia.org/wiki/List_of_DNS_record_types)的所有註冊失敗 (類型 A 和 AAAA) 會連同提出註冊要求的用戶端 IP 一起反白顯示。 您可以接著依照下列步驟，使用此資訊來尋找註冊失敗的根本原因︰

1. 尋找對於用戶端嘗試更新之名稱具有權威性的區域。

1. 使用此解決方案來檢查該區域的清查資訊。

1. 確認已啟用該區域的動態更新。

1. 檢查該區域是否已設定安全動態更新。

    ![[動態 DNS 註冊] 刀鋒視窗](./media/log-analytics-dns/dynamic-dns-reg-blade.png)

**名稱註冊要求**。 上方的圖格會顯示成功和失敗之 DNS 動態更新要求的趨勢線。 下方的圖格所列出的前 10 個用戶端，是將失敗的 DNS 更新要求傳送給 DNS 伺服器 (依失敗數目排序)。

![[名稱註冊要求] 刀鋒視窗 ](./media/log-analytics-dns/name-reg-req-blade.png)

**範例 DDI 分析查詢**。 包含最常見的搜尋查詢清單，會直接擷取未經處理的分析資料。


![範例查詢](./media/log-analytics-dns/queries.png)

您可以使用這些查詢做為起點，建立自己的查詢以供自訂報告。 查詢會連結到顯示結果的 [DNS 分析記錄搜尋] 頁面︰

- **DNS 伺服器清單**。 顯示所有 DNS 伺服器的清單，包含其相關聯的 FQDN、網域名稱、樹系名稱和伺服器的 IP。
- **DNS 區域清單**。 顯示所有 DNS 區域的清單，及其相關聯的區域名稱、動態更新狀態、名稱伺服器及 DNSSEC 簽署狀態。
- **未使用的資源記錄**。 顯示所有未使用/過時資源記錄的清單。 此清單包含資源記錄名稱、資源記錄類型、相關聯的 DNS 伺服器、記錄產生時間和區域名稱。 您可以使用此清單來找出不再使用的 DNS 資源記錄。 根據這項資訊，您可以接著從 DNS 伺服器將這些項目移除。
- **DNS 伺服器查詢負載**。 可顯示資訊，以便您能在 DNS 伺服器上取得 DNS 負載的檢視方塊。 這項資訊可協助您規劃伺服器的容量。 您可以移至 [計量] 索引標籤，將檢視變更為圖形化視覺效果。 此檢視可協助您了解您的 DNS 伺服器的 DNS 負載分佈狀況。 它會顯示每部伺服器的 DNS 查詢速率趨勢。

    ![DNS 伺服器查詢記錄搜尋結果](./media/log-analytics-dns/dns-servers-query-load.png)

- **DNS 區域的查詢負載**。 顯示受解決方案管理的 DNS 伺服器上所有區域的 DNS 區域每秒查詢統計資料。 按一下 [計量] 索引標籤，以將檢視從詳細記錄變更為圖形化的視覺效果結果。
- **設定事件**。 顯示所有的 DNS 設定變更事件和相關聯的訊息。 您可以根據事件的時間、事件識別碼、DNS 伺服器或工作類別進一步篩選這些事件。 這些資料可協助您稽核 DNS 伺服器在特定時間發生的變更。
- **DNS 分析記錄**。 顯示解決方案所管理的所有 DNS 伺服器上的所有分析事件。 您可以根據事件的時間、事件識別碼、DNS 伺服器、執行查閱查詢的用戶端 IP 和查詢類型工作類別，將這些事件進一步篩選。 DNS 伺服器分析事件可追蹤 DNS 伺服器上的活動。 每次伺服器傳送或接收 DNS 資訊時，都會記錄分析事件。

### <a name="search-by-using-dns-analytics-log-search"></a>使用 DNS 分析記錄搜尋來搜尋

在 [記錄搜尋] 頁面上，您可以建立查詢。 您可以使用 facet 控制項來篩選搜尋結果。 您也可以建立進階查詢來轉換、篩選和報告結果。 使用下列查詢來開始：

1. 在**搜尋查詢方塊**中，輸入 `DnsEvents` 以檢視此解決方案管理的 DNS 伺服器所產生的所有 DNS 事件。 結果會列出與查閱查詢、動態註冊和設定變更相關之所有事件的記錄資料。

    ![DnsEvents 記錄搜尋](./media/log-analytics-dns/log-search-dnsevents.png)  

    a. 若要檢視查閱查詢的記錄資料，請從左側 facet 控制項選取 **LookUpQuery** 作為 **Subtype** 篩選條件。 隨即顯示列出所選時間週期之查閱查詢事件的資料表。

    b. 若要檢視動態註冊的記錄資料，請從左側 facet 控制項選取 **DynamicRegistration** 作為 **Subtype** 篩選條件。 隨即顯示列出所選時間週期之動態註冊事件的資料表。

    c. 若要檢視設定變更的記錄資料，請從左側 facet 控制項選取 **ConfigurationChange** 作為 **Subtype** 篩選條件。 隨即顯示列出所選時間週期之設定變更事件的資料表。

1. 在**搜尋查詢方塊**中，輸入 `DnsInventory` 以檢視此解決方案管理的 DNS 伺服器的所有 DNS 清查相關資料。 結果會列出 DNS 伺服器、DNS 區域和資源記錄的記錄資料。

    ![DnsInventory 記錄搜尋](./media/log-analytics-dns/log-search-dnsinventory.png)

## <a name="feedback"></a>意見反應

提供意見反應的方式有兩種：

- **UserVoice**. 發表您對 DNS 分析功能的想法。 請瀏覽 [Operations Management Suite UserVoice 頁面](https://aka.ms/dnsanalyticsuservoice)。
- **加入我們的行列**。 我們始終歡迎新客戶加入我們的行列以搶先試用新功能，並協助我們改善 DNS 分析。 如果您想加入我們的行列，請填妥這份[快速問卷調查 (英文)](https://aka.ms/dnsanalyticssurvey)。

## <a name="next-steps"></a>後續步驟

[搜尋記錄](log-analytics-log-searches.md)以檢視詳細的 DNS 記錄。
