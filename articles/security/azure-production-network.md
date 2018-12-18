---
title: Azure 生產網路
description: 本文提供 Azure 生產網路的一般描述。
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
ms.openlocfilehash: 710792c890c3e48fc54507f93eeaee529ca839f8
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39114023"
---
# <a name="the-azure-production-network"></a>Azure 生產網路
Azure 生產網路的使用者同時包括存取自有 Azure 應用程式的外部客戶，以及管理生產網路的內部 Azure 支援人員。 本文會討論用於建立 Azure 生產網路連線的安全性存取方法和保護機制。

## <a name="internet-routing-and-fault-tolerance"></a>網際網路路由和容錯
全域備援的內部和外部 Azure 網域名稱服務 (DNS) 基礎結構，結合多個主要和次要 DNS 伺服器叢集以提供容錯功能。 同時，會使用額外的 Azure 網路安全性控制 (例如 NetScaler) 來防止分散式阻斷服務 (DDoS) 攻擊，並保護 Azure DNS 服務的完整性。

Azure DNS 伺服器位於多個資料中心設施。 Azure DNS 實作會結合次要和主要 DNS 伺服器的階層，以公開解析 Azure 客戶網域名稱。 網域名稱通常會解析成 CloudApp.net 位址，其會針對客戶的服務包裝虛擬 IP (VIP) 位址。 對應至租用戶轉譯的內部專用 IP (DIP) 位址的 VIP，是由負責該 VIP 的 Microsoft 負載平衡器進行，此為 Azure 特有的現象。

Azure 裝載於美國境內分散各地的 Azure 資料中心，而且建置於能實作強大、可擴充之架構標準的最新路由平台上。 其中一些值得注意的功能為：

- 以多重通訊協定標籤交換 (MPLS) 為基礎的流量工程，可在發生中斷情形的情況下提供有效的連結利用和適度的效能降低。
- 網路會經由「需求加一」(N + 1) 備援性架構或更佳的架構來實作。
- 在外部，資料中心是由專用的高頻寬網路線路提供支援，這些線路能於多個對等互連點上透過全球 1,200 個網際網路服務提供者重複連線資產。 此連線可提供每秒超過 2,000 GB (GBps) 的邊緣容量。

因為 Microsoft 在資料中心之間擁有自己的網路線路，這些特性可協助 Azure 供應項目達成超過 99.9% 的網路可用性，而不需要傳統的第三方網際網路服務提供者。

## <a name="connection-to-production-network-and-associated-firewalls"></a>連線至生產網路及相關聯的防火牆
Azure 網路網際網路流量流程原則會將流量導向位於美國境內最近區域資料中心的 Azure 生產網路。 由於 Azure 生產資料中心會維護一致的網路架構和硬體，因此下列的流量流程說明會一致地套用至所有的資料中心。

將 Azure 的網際網路流量路由傳送至最接近的資料中心之後，就能與存取路由器建立連線。 這些存取路由器用於隔離 Azure 節點與客戶具現化 VM 之間的流量。 位於存取與邊緣位置的網路基礎結構裝置是套用輸入和輸出篩選條件的邊界點。 這些路由器是透過階層式存取控制清單 (ACL) 來設定，以篩選不必要的網路流量並套用流量速率限制 (如有必要的話)。 ACL 所允許的流量會路由傳送至負載平衡器。 散發路由器的設計訴求是只允許 Microsoft 核准的 IP 位址、提供防詐騙功能，以及建立使用 ACL 的 TCP 連線。

外部負載平衡裝置位於存取路由器後方，以執行從網際網路可路由傳送的 IP 至 Azure 內部 IP 的網路位址轉譯 (NAT)。 裝置也會將封包路由傳送至有效的生產內部 IP 和連接埠，並它們會作為保護機制來限制內部生產網路位址空間的公開。

根據預設，Microsoft 會對傳輸到客戶網頁瀏覽器的所有流量 (包括登入以及之後的所有流量)，強制使用超文字安全傳輸通訊協定 (HTTPS)。 使用 TLS v1.2 可讓流量流經安全的通道。 存取和核心路由器上的 ACL 可確保流量的來源如同預期。

相較於傳統的安全性架構，此架構的重要區別在於它在連線至 Azure 生產環境之前，不會有一般會預期的專用硬體防火牆、特製化入侵偵測/預防裝置，或其他安全性設備。 客戶通常會預期 Azure 網路中會有這些硬體防火牆裝置，不過 Azure 內並不會運用它們。 這些安全性功能幾乎都內建於執行 Azure 環境的軟體中，以提供強大的多層式安全性機制，包括防火牆功能。 此外，界限的範圍，以及重要安全性裝置的相關聯擴充將更易於管理及清查，因為如上述內容所示，它是由執行 Azure 的軟體所管理。

## <a name="core-security-and-firewall-features"></a>核心安全性和防火牆功能
Azure 會在各種層級實作強固的軟體安全性和防火牆功能，以強制執行通常會在傳統環境中預期的安全性功能，以保護核心安全性授權界限。

### <a name="azure-security-features"></a>Azure 安全性功能
Azure 會在生產網路內實作以主機為基礎的軟體防火牆。 核心 Azure 環境內有數個核心安全性和防火牆功能。 這些安全性功能反映出 Azure 環境中的深度防禦策略。 Azure 中的客戶資料會受到下列防火牆保護：

**Hypervisor 防火牆 (封包篩選器)**：此防火牆會在 Hypervisor 中實作，並由網狀架構控制器 (FC) 代理程式設定。 此防火牆會保護在 VM 內執行的租用戶，以防止未經授權的存取。 根據預設，系統在 VM 建立時會封鎖所有流量，然後 FC 代理程式會在篩選器中新增規則及例外狀況，以允許經過授權的流量。

系統在此會設定兩種類型的規則：

- **機器設定或基礎結構規則**：根據預設，會封鎖所有通訊。 有存在例外狀況，可允許 VM 傳送和接收動態主機設定通訊協定 (DHCP) 通訊及 DNS 資訊，並將傳送至「公用」網際網路的流量輸出到 FC 叢集內的其他 VM 和 OS 啟用伺服器。 由於 VM 的連出目的地允許清單不包含 Azure 路由器子網路和其他 Microsoft 屬性，因此這些規則會作為其防禦層。
- **角色設定檔規則**：這會根據租用戶的服務模型定義輸入 ACL。 例如，如果租用戶在特定 VM 的連接埠 80 上有 Web 前端，則系統會對所有 IP 位址開放連接埠 80。 如果 VM 有執行中的背景工作角色，則系統只會對相同租用戶內的 VM 開放該背景工作角色。

**原生主機防火牆**：Azure 服務網狀架構和 Azure 儲存體會在沒有 Hypervisor 的原生 OS 上執行，因此系統會使用上述兩組規則來設定 Windows 防火牆。

**主機防火牆**：主機防火牆會保護執行 Hypervisor 的主機分割區。 規則設計成僅允許 FC，並以跳躍方式在特定連接埠上與主機分割區通訊。 其他例外狀況則會允許 DHCP 回應與 DNS 回覆。 Azure 會使用機器設定檔，該檔案具有主機分割區的防火牆規則範本。 同時也存在主機防火牆例外狀況，能允許 VM 透過特定通訊協定/連接埠來與主機元件、線路伺服器和中繼資料伺服器通訊。

**客體防火牆**：客體 OS 的 Windows 防火牆片段，可由客戶在客戶 VM 和儲存體上進行設定。

內建於 Azure 功能的其他安全性功能包括：

- 會獲派來自 DIP 之 IP 位址的基礎結構元件。 網際網路上的攻擊者無法處理那些位址的流量，因為該流量並無法觸達 Microsoft。 網際網路閘道路由器會篩選單獨送至內部位址的封包，因此它們並不會進入生產網路。 負載平衡器是唯一會接受針對 VIP 導向之流量的元件。
- 實作於所有內部節點上的防火牆，針對任何指定案例都有三個主要安全性架構考量：

   - 防火牆會被置於負載平衡器後方，並接受來自任何地方的封包。 這些封包預期會對外公開，而且會對應至傳統周邊防火牆中開啟的連接埠。
   - 防火牆只會接受來自有限位址集合的封包。 此考量為針對 DDoS 攻擊之防禦深入策略的一部分。 這類連線會以密碼編譯方式進行驗證。
   - 防火牆只能從特定的內部節點存取。 它們只會接受來自來源 IP 位址列舉清單中的封包，而這些 IP 全都是 Azure 網路內的 DIP。 例如，針對公司網路的攻擊可以將要求導向這些位址，但除非封包的來源位址位於 Azure 網路內的列舉清單中，否則該攻擊將會被封鎖。
     - 周邊的存取路由器會因為其已設定的靜態路由，而封鎖送至 Azure 網路內部之位址的輸出封包。

## <a name="next-steps"></a>後續步驟
若要深入了解 Microsoft 為保護 Azure 基礎結構執行了哪些動作，請參閱：

- [Azure 設施、廠房以及實體安全性](azure-physical-security.md)
- [Azure 基礎結構可用性](azure-infrastructure-availability.md)
- [Azure 資訊系統元件與界限](azure-infrastructure-components.md)
- [Azure 網路架構](azure-infrastructure-network.md)
- [Azure SQL Database 安全性功能](azure-infrastructure-sql.md)
- [Azure 生產環境運作與管理](azure-infrastructure-operations.md)
- [Azure 基礎結構監視](azure-infrastructure-monitoring.md)
- [Azure 基礎結構完整性](azure-infrastructure-integrity.md)
- [Azure 客戶資料保護](azure-protection-of-customer-data.md)
