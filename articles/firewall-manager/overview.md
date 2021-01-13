---
title: 什麼是 Azure 防火牆管理員？
description: 了解 Azure 防火牆管理員功能
author: vhorne
ms.service: firewall-manager
services: firewall-manager
ms.topic: overview
ms.date: 01/12/2021
ms.author: victorh
ms.openlocfilehash: 63e2aac4c12ecc5d832cb037fda91bd2c6ad0bf1
ms.sourcegitcommit: 431bf5709b433bb12ab1f2e591f1f61f6d87f66c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98132438"
---
# <a name="what-is-azure-firewall-manager"></a>什麼是 Azure 防火牆管理員？

Azure 防火牆管理員是一種安全性管理服務，能為雲端式安全界限提供集中的安全性原則及路由管理。 

防火牆管理員可針對兩種網路架構類型提供安全性管理：

- **安全的虛擬中樞**

   [Azure 虛擬 WAN 中樞](../virtual-wan/virtual-wan-about.md#resources)是一項由 Microsoft 管理的資源，可讓您輕鬆建立中樞和輪輻架構。 當安全性和路由原則與這類中樞相關聯時，我們即稱之為 *[安全虛擬中樞](secured-virtual-hub.md)* 。 
- **中樞虛擬網路**

   這是您自行建立和管理的標準 Azure 虛擬網路。 當安全性原則與這類中樞相關聯時，我們即稱之為「安全虛擬網路」。 目前僅支援 Azure 防火牆原則。 您可以將包含工作負載伺服器和服務的輪輻虛擬網路對等互連。 您也可以在獨立虛擬網路中管理未對等互連到任何輪輻的防火牆。

如需「安全虛擬中樞」和「中樞虛擬網路」架構的詳細比較，請參閱[Azure 防火牆管理員架構選項為何？](vhubs-and-vnets.md)。

![防火牆管理員](media/overview/trusted-security-partners.png)

## <a name="azure-firewall-manager-features"></a>Azure 防火牆管理員功能

Azure 防火牆管理員提供下列功能：

### <a name="central-azure-firewall-deployment-and-configuration"></a>集中的 Azure 防火牆部署和設定

您可以集中部署及設定多個跨不同 Azure 區域和訂用帳戶的 Azure 防火牆執行個體。 

### <a name="hierarchical-policies-global-and-local"></a>階層式原則 (全域和本機)

您可以使用 Azure 防火牆管理員，跨多個受保護的虛擬中樞集中管理 Azure 防火牆原則。 您的中央 IT 小組可以撰寫全域防火牆原則，以在多個小組間強制執行全組織的防火牆原則。 在本機撰寫的防火牆原則可讓 DevOps 自助模型獲得更高的靈活性。

### <a name="integrated-with-third-party-security-as-a-service-for-advanced-security"></a>與第三方安全性即服務整合，以獲得進階安全性

除了 Azure 防火牆之外，您還可以整合第三方安全性即服務 (SECaaS) 提供者，為您的 VNet 和分支網際網路連線提供額外的網路保護。

這項功能僅適用於安全虛擬中樞部署。

- VNet 對網際網路 (V2I) 流量篩選

   - 使用您慣用的第三方安全性提供者來篩選輸出虛擬網路流量。
   - 針對在 Azure 上執行的雲端工作負載，使用進階使用者感知網際網路保護。

- 分支對網際網路 (B2I) 流量篩選

   利用您的 Azure 連線和全域散發，輕鬆地為分支對網際網路案例新增第三方篩選。

如需關於安全性提供者的詳細資訊，請參閱[什麼是 Azure 防火牆管理員安全性合作夥伴提供者？](trusted-security-partners.md)

### <a name="centralized-route-management"></a>集中式路由管理

輕鬆地將流量路由傳送到安全中樞以進行篩選和記錄，而無須在輪輻虛擬網路上手動設定使用者定義路由 (UDR)。 

這項功能僅適用於安全虛擬中樞部署。

您可以使用第三方提供者進行分支對網際網路 (B2I) 流量篩選，且此篩選可以與分支對 VNet (B2V)、VNet 對 VNet (V2V) 和 VNet 對網際網路 (V2I) 的 Azure 防火牆並存。 如果 B2V 或 V2V 不需要 Azure 防火牆，您也可以使用第三方提供者進行 V2I 流量篩選。 

## <a name="region-availability"></a>區域可用性

您可以跨區域使用 Azure 防火牆原則。 例如，您可以在美國西部建立原則，並將其用於美國東部。 

## <a name="known-issues"></a>已知問題

Azure 防火牆管理員有下列已知問題：

|問題  |描述  |降低  |
|---------|---------|---------|
|流量分割|目前不支援 Microsoft 365 和 Azure 公用 PaaS 流量分割。 因此，若為 V2I 或 B2I 選取第三方提供者，也會透過合作夥伴服務傳送所有的 Azure 公用 PaaS 和 Microsoft 365 流量。|正在調查中樞的流量分割。
|每個區域一個安全虛擬中樞|每個區域不能有超過一個安全虛擬中樞。|在區域中建立多個虛擬 WAN。|
|基本原則必須位於與本機原則相同的區域中|在與基底原則相同的區域中建立您所有的本機原則。 您仍可將在某個區域中建立的原則套用到另一個區域的安全中樞上。|正在調查|
|篩選安全虛擬中樞部署內的中樞間流量|目前尚不支援安全虛擬中樞對安全虛擬中樞的通訊篩選。 不過，如果未啟用透過 Azure 防火牆進行的私人流量篩選，則中樞對中樞通訊仍可運作。|正在調查|
|輪輻位於與虛擬中樞不同的區域|不支援讓輪輻位於與虛擬中樞不同的區域。|正在調查<br><br>請在與中樞相同的區域中，建立每個區域的中樞和對等 VNet。|
|已啟用私人流量篩選時的分支對分支流量|已啟用私人流量篩選時，不支援分支對分支流量。 |正在調查。<br><br>如果分支對分支連線能力很重要，則請勿保護私人流量。|
|所有共用相同虛擬 WAN 的安全虛擬中樞，都必須位於相同的資源群組中。|此行為與現今的虛擬 WAN 中樞一致。|建立多個虛擬 WAN，以允許在不同的資源群組中建立安全虛擬中樞。|
|大量 IP 位址新增失敗|如果您新增多個公用 IP 位址，則安全中樞防火牆會進入失敗狀態。|請以遞增方式少量增加公用 IP 位址。 例如，一次新增 10 個。|
|不支援 DDoS 保護標準搭配安全的虛擬中樞|DDoS保護標準不會與 vWAN 整合。|正在調查|
|未完全支援活動記錄|防火牆原則目前不支援活動記錄。|正在調查|
|設定 SNAT 私人 IP 位址範圍|如果已設定 Azure 防火牆原則，則會忽略[私人 IP 範圍設定](../firewall/snat-private-range.md)。 已使用預設 Azure 防火牆行為，當目的地 IP 位址不在私人 IP 位址範圍 [IANA RFC 1918](https://tools.ietf.org/html/rfc1918) 時，不會採用 SNAT 網路規則。|正在調查|
|將防火牆遷移為使用防火牆原則時，不會遷移某些防火牆設定|當您遷移至 Azure 防火牆原則時，不會遷移可用性區域和 SNAT 私人位址。|正在調查| 

## <a name="next-steps"></a>後續步驟

- 檢閱 [Azure 防火牆管理員部署概觀](deployment-overview.md)
- 了解[安全虛擬中樞](secured-virtual-hub.md)。
