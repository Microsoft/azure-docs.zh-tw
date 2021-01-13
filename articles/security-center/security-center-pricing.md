---
title: Azure 資訊安全中心的定價
description: 在是否包含 Azure Defender 的基礎下，Azure 資訊安全中心分成兩種模式。
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 4d1364cd-7847-425a-bb3a-722cb0779f78
ms.service: security-center
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/13/2020
ms.author: memildin
ms.openlocfilehash: 99f47df86d89e9daf2bc8878b868b04b7038ffd5
ms.sourcegitcommit: 3af12dc5b0b3833acb5d591d0d5a398c926919c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2021
ms.locfileid: "98071199"
---
# <a name="pricing-of-azure-security-center"></a>Azure 資訊安全中心的定價
Azure 資訊安全中心提供統一的安全性管理和進階的威脅防護，保護 Azure、內部部署和其他雲端中執行的工作負載。 其提供了針對混合式雲端工作負載的可見性和控制能力、可降低威脅曝露度的主動防禦措施、還有智慧型偵測可幫助您跟上網路風險快速發展的腳步。


## <a name="free-option-vs-azure-defender-enabled"></a>免費選項及已啟用 Azure Defender

資訊安全中心提供兩個模式：

- **關閉 Azure Defender** (免費) - 當您第一次造訪 Azure 入口網站中的 [Azure 資訊安全中心] 儀表板，或透過 API 以程式設計方式啟用此功能時，您所有的 Azure 訂用帳戶都會免費啟用不包含 Azure Defender 的資訊安全中心。 此免費模式會提供安全性原則、持續的安全性評估，以及可操作的安全性建議，以幫助您保護 Azure 資源。

- **開啟 Azure Defender** - 啟用 Azure Defender 會將免費模式的功能擴展到在私人雲端和其他公用雲端中執行的工作負載，為您的混合式雲端工作負載提供統一的安全性管理和威脅防護。 Azure Defender 的一些主要功能：

    - **適用於端點的 Microsoft Defender** - 適用於伺服器的 Azure Defender 包含 [適用於端點的 Microsoft Defender](https://www.microsoft.com/microsoft-365/security/endpoint-defender)，可讓您獲得完整的端點偵測和回應 (EDR) 功能。 若要深入了解搭配使用適用於端點的 Microsoft Defender 與 Azure Defender 的優點，請參閱[使用資訊安全中心的整合式 EDR 解決方案](security-center-wdatp.md)。
    - **虛擬機器和容器登錄的弱點掃描** - 輕鬆地將掃描器部署到您所有的虛擬機器，以提供業界最先進的弱點管理解決方案。 直接在資訊安全中心內檢視、調查和補救結果。 
    - **混合式安全性** – 取得您所有內部部署和雲端工作負載中安全性的統一檢視。 套用安全性原則，並持續評估您混合式雲端工作負載的安全性，以確保符合安全性標準。 從多種來源 (包括防火牆和其他合作夥伴解決方案) 收集、搜尋及分析安全性資料。
    - **威脅防護警示** - 進階的行為分析和 Microsoft Intelligent Security Graph 可勝過不斷演進的網路攻擊。 內建行為分析和機器學習服務可識別攻擊和零時差惡意探索。 監視網路、機器和雲端服務中是否有傳入攻擊和侵入後活動。 使用互動式工具和內容相關威脅情報來簡化調查。
    - **存取與應用程式控制** (AAC) - 套用依特定工作負載調整並由機器學習服務提供支援的建議，以建立允許與拒絕清單來封鎖惡意程式碼和其他不想要的應用程式。 使用 Just-In-Time 來控制針對 Azure VM 上管理連接埠的存取，以減少您網路的受攻擊面。 AAC 會大幅減少暴力密碼破解和其他網路攻擊的風險。
    - **容器安全性功能** - 受益於您容器化環境的弱點管理和即時威脅防護。 啟用 **適用於容器登錄的 Azure Defender** 時，最多可能需要 12 小時才能啟用所有功能。 我們會以推送至已連線登錄的唯一容器映像數目來計費。 在映像經過一次掃描之後，您就不需要再向此支付費用，除非進行修改並再推送一次。
    - **為連線到 Azure 環境的資源提供完整的威脅防護** - Azure Defender 包含Azure 原生完整威脅防護，適用於所有資源通用的 Azure 服務，包括：Azure Resource Manager、Azure DNS、Azure 網路層，以及 Azure Key Vault。 Azure Defender 對 Azure 管理層和 Azure DNS 層具有獨特的可見度，因此可保護已連線到這些層的雲端資源。


## <a name="try-azure-defender-free-for-30-days"></a>免費試用 Azure Defender 30 天
Azure Defender 在前 30 天免費。 在 30 天結束時，如果您選擇繼續使用服務，我們會自動開始針對使用量計費。

## <a name="enable-azure-defender"></a>啟用 Azure Defender
您可以使用 Azure Defender 來保護整個 Azure 訂用帳戶，而且訂用帳戶內的所有資源都會繼承保護。

若要啟用 Azure Defender：

1. 在資訊安全中心的主要功能表中，選取 [定價和設定]。
1. 選取您要升級的訂用帳戶。
1. 選取 [開啟 Azure Defender] 以進行升級。
1. 選取 [儲存]。

以下是範例訂用帳戶的定價頁面。 您會發現，Azure Defender 中的每個方案都是分開定價，而且可以個別設定為開啟或關閉。

:::image type="content" source="./media/security-center-pricing/pricing-tier-page.png" alt-text="入口網站中的資訊安全中心定價頁面":::

> [!NOTE]
> 若要啟用所有資訊安全中心功能 (包括威脅防護功能)，您必須在包含適用工作負載的訂用帳戶上啟用 Azure Defender。 在工作區層級上啟用並不會啟用 Just-In-Time VM 存取、自適性應用程式控制，以及 Azure 資源的網路偵測。 此外，在工作區層級可用的唯一 Azure Defender 方案是適用於伺服器的 Azure Defender，以及適用於機器上 SQL 伺服器的 Azure Defender。
>
> 您可以在訂用帳戶層級或資源層級啟用 **適用於儲存體帳戶的 Azure Defender**。
> 您可以在訂用帳戶層級或資源層級啟用 **適用於 SQL 的 Azure Defender**。
> 您只能在資源層級為 **適用於 MariaDB/MySQL/PostgreSQL 的 Azure 資料庫** 啟用威脅防護。


## <a name="faq---pricing-and-billing"></a>常見問題 - 定價和計費 

- [如何追蹤組織中哪些人在 Azure 資訊安全中心中啟用 Azure Defender 變更？](#how-can-i-track-who-in-my-organization-enabled-azure-defender-changes-in-security-center)
- [資訊安全中心提供哪些方案？](#what-are-the-plans-offered-by-security-center)
- [如何為我的訂用帳戶啟用 Azure Defender？](#how-do-i-enable-azure-defender-for-my-subscription)
- [我是否可為訂用帳戶中的部分伺服器啟用適用於伺服器的 Azure Defender？](#can-i-enable-azure-defender-for-servers-on-a-subset-of-servers-in-my-subscription)
- [如果我已經有「適用於端點的 Microsoft Defender」授權，我可以擁有 Azure Defender 的折扣嗎？](#if-i-already-have-a-license-for-microsoft-defender-for-endpoint-can-i-get-a-discount-for-azure-defender)
- [我的訂用帳戶已啟用適用於伺服器的 Azure Defender，是否需要支付非執行中伺服器的費用？](#my-subscription-has-azure-defender-for-servers-enabled-do-i-pay-for-not-running-servers)
- [我是否需支付未安裝 Log Analytics 代理程式的電腦費用？](#will-i-be-charged-for-machines-without-the-log-analytics-agent-installed)
- [如果 Log Analytics 代理程式向多個工作區報告，是否需要支付兩次費用？](#if-a-log-analytics-agent-reports-to-multiple-workspaces-will-i-be-charged-twice)
- [如果 Log Analytics 代理程式向多個工作區報告，是否有 500 MB 的免費資料擷取可供使用？](#if-a-log-analytics-agent-reports-to-multiple-workspaces-is-the-500-mb-free-data-ingestion-available-on-all-of-them)
- [要針對整個工作區計算 500 MB 的免費資料擷取，或是僅針對每台電腦計算擷取作業？](#is-the-500-mb-free-data-ingestion-calculated-for-an-entire-workspace-or-strictly-per-machine)

### <a name="how-can-i-track-who-in-my-organization-enabled-azure-defender-changes-in-security-center"></a>如何追蹤組織中哪些人在資訊安全中心中啟用 Azure Defender 變更？
Azure 訂用帳戶可能有多個管理員具有變更定價設定的權限。 若要找出哪位使用者進行了變更，請使用 Azure 活動記錄。

:::image type="content" source="media/security-center-pricing/logged-change-to-pricing.png" alt-text="顯示價格變更事件的 Azure 活動記錄檔":::

如果使用者的資訊未列在 **依起始事件排序** 的資料行中，請探索事件的 JSON 以了解相關詳細資料。

:::image type="content" source="media/security-center-pricing/tracking-pricing-changes-in-activity-log.png" alt-text="Azure 活動記錄 JSON 總管":::


### <a name="what-are-the-plans-offered-by-security-center"></a>資訊安全中心提供哪些方案？ 
資訊安全中心有兩個供應項目： 

- 免費 Azure 資訊安全中心 
- Azure Defender  

### <a name="how-do-i-enable-azure-defender-for-my-subscription"></a>如何為我的訂用帳戶啟用 Azure Defender？ 
您可以使用下列任何一種方式，為您的訂用帳戶啟用 Azure Defender： 

|方法  |指示  |
|---------|---------|
|Azure 入口網站的 Azure 資訊安全中心頁面|[啟用 Azure Defender](#enable-azure-defender)|
|REST API|[定價 API](/rest/api/securitycenter/pricings)|
|Azure CLI|[az security pricing](/cli/azure/security/pricing)|
|PowerShell|[Set-AzSecurityPricing](/powershell/module/az.security/set-azsecuritypricing)|
|Azure 原則|[搭售方案定價](https://github.com/Azure/Azure-Security-Center/blob/master/Pricing%20%26%20Settings/ARM%20Templates/Set-ASC-Bundle-Pricing.json)|
|||

### <a name="can-i-enable-azure-defender-for-servers-on-a-subset-of-servers-in-my-subscription"></a>我是否可為訂用帳戶中的部分伺服器啟用適用於伺服器的 Azure Defender？
否。 在訂用帳戶上啟用[適用於伺服器的 Azure Defender](defender-for-servers-introduction.md) 時，Azure Defender 將會保護訂用帳戶中的所有伺服器。 

另一種方式是在 Log Analytics 工作區層級啟用適用於伺服器的 Azure Defender。 如果您這樣做，只有向該工作區報告的伺服器才會受到保護和計費。 不過，有幾項功能將無法使用。 包括Just-In-Time VM 存取、網路偵測、法規合規性、彈性網路強化、彈性應用程式控制等等。 

### <a name="if-i-already-have-a-license-for-microsoft-defender-for-endpoint-can-i-get-a-discount-for-azure-defender"></a>如果我已經有「適用於端點的 Microsoft Defender」授權，我可以擁有 Azure Defender 的折扣嗎？
如果您已經有「適用於端點的 Microsoft Defender」授權，就不需要支付該部分的 Azure Defender 授權。

若要確認您的折扣，請洽詢資訊安全中心的支援小組，並提供相關的工作區識別碼、區域和授權資訊。

### <a name="my-subscription-has-azure-defender-for-servers-enabled-do-i-pay-for-not-running-servers"></a>我的訂用帳戶已啟用適用於伺服器的 Azure Defender，是否需要支付非執行中伺服器的費用？ 
否。 當您啟用訂用帳戶上 [適用於伺服器的 Azure Defender](defender-for-servers-introduction.md) 時，只需支付執行中伺服器的每小時費用。 您無需支付任何關閉的伺服器費用。 

> [!TIP]
> 這也適用於資訊安全中心所保護的其他資源類型。 

### <a name="will-i-be-charged-for-machines-without-the-log-analytics-agent-installed"></a>我是否需支付未安裝 Log Analytics 代理程式的電腦費用？
是。 當您啟用[適用於伺服器的 Azure Defender](defender-for-servers-introduction.md) 時，該訂用帳戶中的電腦即使尚未安裝 Log Analytics 代理程式，也會受到一定範圍的保護。

### <a name="if-a-log-analytics-agent-reports-to-multiple-workspaces-will-i-be-charged-twice"></a>如果 Log Analytics 代理程式向多個工作區報告，是否需要支付兩次費用？ 
是。 如果您已將 Log Analytics 代理程式設定為將資料傳送至兩個或多個不同的 Log Analytics 工作區 (多路連接)，則會針對每個工作區向您收取已安裝「安全性」或「反惡意程式碼軟體」解決方案的費用。 

### <a name="if-a-log-analytics-agent-reports-to-multiple-workspaces-is-the-500-mb-free-data-ingestion-available-on-all-of-them"></a>如果 Log Analytics 代理程式向多個工作區報告，是否有 500 MB 的免費資料擷取可供使用？
是。 如果您已將 Log Analytics 代理程式設定為將資料傳送至兩個或多個不同的 Log Analytics 工作區 (多路連接)，則會取得 500 MB 的免費資料擷取。 這會依每個節點、每個報告的工作區每天計算，並適用於已安裝「安全性」或「反惡意程式碼軟體」解決方案的每個工作區。 您需支付超過 500 MB 的所有資料擷取費用。

### <a name="is-the-500-mb-free-data-ingestion-calculated-for-an-entire-workspace-or-strictly-per-machine"></a>要針對整個工作區計算 500 MB 的免費資料擷取，或是僅針對每台電腦計算擷取作業？
每一部連線到工作區的電腦，每天都會取得 500 MB 的免費資料擷取。 特別是針對 Azure 資訊安全中心直接收集的安全性資料類型。

此資料是所有節點的平均每日速率。 因此，即使有些電腦傳送了 100 MB，而其他電腦傳送了 800 MB，如果總計未超過 **[電腦數目] x 500 MB** 的免費限制，則不需支付額外費用。

## <a name="next-steps"></a>後續步驟
本文說明資訊安全中心的定價選項。 如需相關內容，請參閱：

- [如何最佳化您的 Azure 工作負載成本](https://azure.microsoft.com/blog/how-to-optimize-your-azure-workload-costs/)
- [以所選貨幣和區域為依據的定價詳細資料](https://azure.microsoft.com/pricing/details/security-center/)
- 您可能會想要管理您的成本，並透過將解決方案限制在一組特定的代理程式，來限制針對該解決方案所收集的資料量。 [解決方案目標](../azure-monitor/insights/solution-targeting.md)可讓您將某個範圍套用至解決方案，並將工作區中的電腦子集設定為目標。 如果您使用解決方案目標鎖定，資訊安全中心會將工作區列為沒有解決方案。