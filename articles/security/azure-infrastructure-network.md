---
title: Azure 網路架構
description: 本文提供 Microsoft Azure 基礎結構網路的一般描述。
services: security
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: TomSh
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/28/2018
ms.author: terrylan
ms.openlocfilehash: 5a0f0594a9ccb27a0f76a679e454e9a3ffe19a43
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39505376"
---
# <a name="azure-network-architecture"></a>Azure 網路架構
Azure 網路架構會遵循改良版業界標準核心/散發/存取模型 (具有不同的硬體層級)。 這些層級包括：

- 核心 (資料中心路由器)
- 散發 (存取路由器和 L2 彙總)。 分散層區隔 L3 路由 L2 切換。
- 存取 (L2 主機交換器)

網路架構有兩個層級的第 2 層交換器。 彙總其他層流量。 第二層迴圈合併冗餘。 這可提供更有彈性的 VLAN 使用量，以及改善連接埠調整。 此架構讓 L2 與 L3 有所區別，允許在網路的各個不同層中使用硬體，以及將一層中的錯誤降至最低，以免影響其他層。 使用主幹容許進行資源共用，例如 L3 基礎結構的連線。

## <a name="network-configuration"></a>網路組態
資料中心內 Azure 叢集的網路架構包含下列裝置：

- 路由器 (資料中心、存取路由器和邊界分葉路由器)
- 交換器 (彙總與機櫃頂端交換器)
- Digi CM
- 電源分配單元

Azure 有兩個不同的架構。 某些現有的 Azure 客戶與共用服務位於預設 LAN 架構 (DLA)，而新的區域和虛擬客戶常駐於 Quantum 10 (Q10) 架構。 DLA 架構是傳統樹狀結構設計，具有作用中的被動存取路由器以及套用至存取路由器的安全性存取控制清單 (ACL)。 Quantum 10 架構為封閉式/網格設計的路由器，其中的 ACL 不會在路由器套用。 相反地，ACL 是在路由底下，透過軟體負載平衡 (SLB)，或軟體定義的 VLAN 來套用。

下圖提供 Azure 叢集中的網路架構高階概觀：

![Azure 網路圖表][1]

### <a name="quantum-10-devices"></a>Quantum 10 裝置
Quantum 10 設計會進行遍佈於關閉/網格設計中多個裝置的第 3 層交換。 Q10 設計的優點包括容量更大，以及更有能力調整現有的網路基礎結構。 此設計運用邊界分葉路由器、主幹式交換器和機櫃頂端交換器，來將流量跨多個路由傳遞至叢集，進而允許容錯。 軟體負載平衡，而不是硬體裝置，處理安全性服務，例如網路位址轉譯。

### <a name="access-routers"></a>存取路由器
散發/存取 L3 路由器 (AR) 可執行散發和存取層的主要路由功能。 這些裝置會成對部署，而且是子網路的預設閘道。 視容量而定，每個 AR 組對都可以支援多個 L2 彙總交換器組。 最大數目取決於裝置的容量，以及失敗網域。 典型的數字是每個 AR 組配三個 L2 彙總交換器組。

### <a name="l2-aggregation-switches"></a>L2 彙總交換器  
這些裝置可當作 L2 流量的彙總點。 它們是 L2 網狀架構的散發層，可以處理大量流量。 因為這些裝置彙總流量，所以它們需要 802.1q 功能和高頻寬技術，例如連接埠彙總和 10GE。

### <a name="l2-host-switches"></a>L2 主機交換器
主機會直接連線到這些交換器。 它們可以是固定在機櫃上的交換器，或部署於底座。 802.1q 標準容許將一個 VLAN 指定為原生 VLAN，並將該 VLAN 視為正常 (未標記) 乙太網路框架。 在正常情況下，原生 VLAN 上的訊框會在 802.1q 主幹連接埠上傳輸或接收 (未標記)。 此功能專為移轉到 802.1q 並與非 802.1q 裝置相容而設計。 在此架構中，只有網路基礎結構會利用原生 VLAN。

此架構會指定一個標準，用於選取原生 VLAN。 這個標準會儘可能確保 AR 裝置對每個主幹和 L2Aggregation 至 L2Aggregation 主幹都有唯一的原生 VLAN。 L2Aggregation 至 L2Host 交換器主幹具有非預設的原生 VLAN。

### <a name="link-aggregation-8023ad"></a>連結彙總 (802.3ad)
連結彙總可讓多個個別的連結組合在一起，並視為單一邏輯連結。 為了方便操作偵錯，用來指定連接埠通道介面的數字，應該予以標準化。 其餘的網路會在連接埠通道的兩端使用相同的數字。

針對 L2Agg 至 L2Host 交換器指定的號碼，就是在 L2Agg 端使用的連接埠-通道號碼。 因為 L2Host 端的號碼範圍更加受限，所以標準是在 L2Host 端使用號碼 1 和 2。 這些分別指向進入 "a" 和 "b" 端的連接埠-通道。

### <a name="vlans"></a>VLAN
網路架構會使用 VLAN 將多個伺服器組成單一廣播網域。 VLAN 號碼符合 802.1q 標準，可支援編號 1 - 4094 的 VLAN。

### <a name="customer-vlans"></a>客戶 VLAN
您有各種可透過 Azure 入口網站部署的 VLAN 實作選項，以符合解決方案的隔離和架構需求。 您可以透過虛擬機器來部署這些解決方案。 如需客戶參考架構範例，請參閱 [Azure 參考架構](https://docs.microsoft.com/azure/architecture/reference-architectures/)。

### <a name="edge-architecture"></a>邊緣架構
Azure 資料中心是以高度備援且妥善佈建的網路基礎結構為基礎。 Microsoft 會在 Azure 資料中心內實作「需求加一」(N+1) 備援性架構或更佳的架構。 資料中心之內和之間的完整容錯移轉功能，有助於確保網路和服務可用性。 在外部，會提供專用、高頻寬的網路線路供資料中心使用。 這些備援的線路，將資產與全球的多個對等互連點，超過 1200 個網際網路服務提供者串連起來。 這在網路上提供超過 2,000 Gbps 的潛在邊緣容量。

在 Azure 網路的邊緣和存取層篩選路由器，可在封包層級提供完善的安全性。 這有助於防止有人未經授權便想連線到 Azure。 這些路由器有助於確保封包的實際內容包含預期格式的資料，而且符合預期的用戶端/伺服器通訊配置。 Azure 會實作由下列網路隔離和存取控制元件所組成的階層式架構：

- **邊緣路由器** 這些會區隔應用程式環境與網際網路。 邊緣路由器的設計訴求是提供反詐騙保護，並使用存取控制清單 (ACL) 來限制存取權。
- **發佈 (存取) 的路由器。** 這些只允許 Microsoft 核准的 IP 位址，以提供防詐騙功能，以及使用 ACL 來建立連線。

### <a name="a10-ddos-mitigation-architecture"></a>A10 DDOS 風險降低架構
阻斷服務攻擊會繼續對線上服務的可靠性帶來實質威脅。 隨著攻擊變得更具針對性、更複雜，以及 Microsoft 提供的服務遍及各地時，首要之務就是識別及降低這些攻擊的影響。 下列詳細資料從網路架構的觀點說明如何實作 A10 DDOS 風險降低系統。

Azure 會在資料中心路由器 (DCR) 使用 A10 網路裝置，以提供自動化偵測和風險降低。 A10 解決方案會利用 Azure 網路監視，進行流動封包取樣以及判斷是否有任何攻擊。 一旦偵測到攻擊，A10 裝置就會介入來降低攻擊的風險。 然後只允許乾淨的流量直接從 DCR 進入 Azure 資料中心。 Microsoft 使用 A10 解決方案來保護 Azure 網路基礎結構。

A10 解決方案中的 DDoS 保護包括：

- UDP IPv4 和 IPv6 洪水保護
- ICMP IPv4 和 IPv6 洪水保護
- TCP IPv4 和 IPv6 洪水保護
- IPv4 和 IPv6 的 TCP SYN 攻擊保護
- 片段攻擊

> [!NOTE]
> 依預設，Microsoft 會為所有的 Azure 客戶提供 DDoS 保護。
>
>

## <a name="network-connection-rules"></a>網路連線規則
Azure 會在其網路上部署可在封包層級提供安全性的邊緣路由器，以防止未經授權嘗試連線到 Azure。 邊緣路由器能確保封包的實際內容包含預期格式的資料，而且符合預期的用戶端/伺服器通訊配置。

邊緣路由器會區隔應用程式環境與網際網路。 這些路由器的設計訴求是提供反詐騙保護，並使用存取控制清單 (ACL) 來限制存取權。 Microsoft 會使用階層式 ACL 方法來設定邊緣路由器，以限制能夠傳輸邊緣路由器及存取路由器的網路通訊協定。

Microsoft 會將網路架設在存取和邊緣位置，以當作套用輸入或輸出篩選條件的邊界點。

## <a name="next-steps"></a>後續步驟
若要深入了解 Microsoft 為協助保護 Azure 基礎結構執行了哪些動作，請參閱：

- [Azure 設備、廠房以及實體安全性](azure-physical-security.md)
- [Azure 基礎結構可用性](azure-infrastructure-availability.md)
- [Azure 資訊系統元件與界限](azure-infrastructure-components.md)
- [Azure 生產網路](azure-production-network.md)
- [Azure SQL Database 安全性功能](azure-infrastructure-sql.md)
- [Azure 生產環境運作與管理](azure-infrastructure-operations.md)
- [Azure 基礎結構監視](azure-infrastructure-monitoring.md)
- [Azure 基礎結構完整性](azure-infrastructure-integrity.md)
- [Azure 客戶資料保護](azure-protection-of-customer-data.md)

<!--Image references-->
[1]: ./media/azure-infrastructure-network/network-arch.png
