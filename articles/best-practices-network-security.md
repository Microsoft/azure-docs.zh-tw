---
title: Azure 網路安全性最佳作法 |Microsoft Docs
description: 了解 Azure 中可用來協助建立安全網路環境中的一些重要功能
services: virtual-network
documentationcenter: na
author: tracsman
manager: rossort
editor: ''
ms.assetid: d169387a-1243-4867-a602-01d6f2d8a2a1
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/03/2017
ms.author: jonor
ms.openlocfilehash: cf015f4857a22b755813d0be1af5a55a8b7b6535
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34360467"
---
# <a name="microsoft-cloud-services-and-network-security"></a>Microsoft 雲端服務和網路安全性
Microsoft 雲端服務提供超大規模的服務和基礎結構、企業級的功能，以及許多混合式連線選項。 客戶可以選擇透過網際網路或透過 Azure ExpressRoute (提供私人網路連線能力) 存取這些服務。 Microsoft Azure 平台可讓客戶順暢地將基礎結構延伸至雲端並建置多層式架構。 另外，協力廠商可以提供安全性服務和虛擬設備，以啟用增強的功能。 當客戶使用透過 ExpressRoute 存取的 Microsoft 雲端服務，這份白皮書提供他們應該考慮的安全性和架構性問題的概觀。 也包括在 Azure 虛擬網路中建立更安全的服務。

## <a name="fast-start"></a>快速開始
以下邏輯圖表以具體範例說明 Azure 平台可用的許多安全性技術。 如需快速參考，請找出最符合您情況的範例。 如需延伸說明，請繼續閱讀本文。
[![0]][0]

[範例 1︰建置周邊網路 (也稱為 DMZ 或屏蔽式子網路)，使用網路安全性群組 (NSG) 協助保護應用程式。](#example-1-build-a-perimeter-network-to-help-protect-applications-with-nsgs)</br>
[範例 2：建置周邊網路，使用防火牆和 NSG 協助保護應用程式。](#example-2-build-a-perimeter-network-to-help-protect-applications-with-a-firewall-and-nsgs)</br>
[範例 3︰建置周邊網路，使用防火牆、使用者定義的路由 (UDR) 及 NSG 協助保護網路。](#example-3-build-a-perimeter-network-to-help-protect-networks-with-a-firewall-and-udr-and-nsg)</br>
[範例 4：新增使用站對站、虛擬設備的虛擬私人網路 (VPN) 混合式連接。](#example-4-add-a-hybrid-connection-with-a-site-to-site-virtual-appliance-vpn)</br>
[範例 5：新增使用站對站 Azure VPN 閘道的混合式連接。](#example-5-add-a-hybrid-connection-with-a-site-to-site-azure-vpn-gateway)</br>
[範例 6：新增使用 ExpressRoute 的混合式連接。](#example-6-add-a-hybrid-connection-with-expressroute)</br>
本文會在未來幾個月內，陸續加入在虛擬網路、高可用性及服務鏈結之間加入連接的範例。

## <a name="microsoft-compliance-and-infrastructure-protection"></a>Microsoft 法規遵循與基礎結構保護
為了協助組織符合控管個人資料收集和使用之國家、區域和業界特定需求的標準，Microsoft 提供超過 40 個認證和證明。 所有雲端服務提供者中最完整的組合。

如需詳細資訊，請參閱 [Microsoft 信任中心][TrustCenter]的合規性資訊。

Microsoft 有完整的方法來保護執行超大規模全域服務所需的雲端基礎結構。 Microsoft 雲端基礎結構除實體資料中心之外，還包括硬體、軟體、網路、管理和操作人員。

![2]

這個方法為客戶提供更安全的基礎，將他們的服務部署在 Microsoft 雲端。 接著就由客戶設計和建立安全性架構來保護這些服務。

## <a name="traditional-security-architectures-and-perimeter-networks"></a>傳統的安全性架構和周邊網路
雖然 Microsoft 大量投資在保護雲端基礎結構，客戶也必須保護其雲端服務和資源群組。 安全性的多層式方法提供最佳的防護。 周邊網路安全區可防止不受信任的網路存取內部網路資源。 周邊網路是指座落在網際網路和受保護的企業 IT 基礎結構之間的網路邊緣或部分地帶。

在典型的企業網路中，核心基礎結構的周邊有多層的安全性裝置，嚴加防禦。 每一層的界限都由裝置和原則強制執行點組成。 每一層可能包含以下網路安全性裝置的組合：防火牆、阻斷服務 (DoS) 預防、入侵偵測或保護系統 (IDS/IPS) 和 VPN 裝置。 原則可能以防火牆原則、存取控制清單 (ACL) 或特定路由等形式強制執行。 網路的第一防線 (直接接受來自網際網路的連入流量) 由這些機制聯合起來阻擋攻擊和有害的流量，但允許合法的要求進入網路。 此流量會直接路由至周邊網路的資源中。 該資源可能與網路中更深處的資源「交談」，傳送至下一個界限先進行驗證。 最外層稱為周邊網路，因為網路的這個部分對網際網路公開，而兩側通常會有某種形式的保護。 下圖顯示公司網路中的單一子網路周邊網路範例，有兩個安全性界限。

![3]

有許多架構用來實作周邊網路。 這些架構從簡單的負載平衡器到多子網路的周邊網路都有，每個界限上設有各種機制來阻擋流量，並保護公司網路的更深層級。 如何組建周邊網路取決於組織的特定需求和整體的風險承受度。

當客戶將工作負載移到公用雲端時，Azure 中的周邊網路架構必須支援類似的功能，才能符合法規遵循和安全性需求。 本文提供客戶如何在 Azure 中組建安全網路環境方法的指導方針。 它著重在周邊網路，但也包含網路安全性各層面的完整討論。 下列問題會通知這項討論︰

* 如何組建 Azure 的周邊網路？
* 組建周邊網路需要哪些 Azure 功能？
* 如何保護後端工作負載？
* 如何控制網際網路對 Azure 中的工作負載進行通訊？
* 如何保護內部部署網路不受 Azure 中的部署所影響？
* 何時該使用原生 Azure 安全性功能？何時又該使用協力廠商設備或服務？

下圖顯示 Azure 提供給客戶的各種安全性層級。 這些層級既是 Azure 平台本身的原生功能，也是客戶定義的功能︰

![4]

從網際網路往內，Azure DDoS 會協防針對 Azure 的大規模攻擊。 下一層是客戶定義的公用 IP 位址 (端點)，用來判斷什麼流量可通過雲端服務進入虛擬網路。 原生 Azure 虛擬網路隔離可確保與其他所有網路完全隔離，而且流量只能流經使用者設定的路徑和方法。 這些路徑和方法就是下一層，可利用 NSG、UDR 和網路虛擬設備來建立安全性界限，以保護受保護網路中的應用程式部署。

下一節提供 Azure 虛擬網路的概觀。 這些虛擬網路由客戶建立，也是部署的工作負載所連接的目的地。 虛擬網路是建立周邊網路來保護 Azure 中的客戶部署時，所需的一切網路安全性功能的基礎。

## <a name="overview-of-azure-virtual-networks"></a>Azure 虛擬網路的概觀
在網際網路流量能進入 Azure 虛擬網路之前，Azure 平台本身就有兩層安全性：

1.    **DDoS 保護**：DDoS 保護是 Azure 實體網路層，保護 Azure 平台本身免於大規模的網際網路攻擊。 這些攻擊會使用多個 "Bot" 節點嘗試灌爆網際網路服務。 Azure 有一層很強大的 DDoS 保護網可篩選所有輸入、輸出及跨 Azure 區域的連線。 這個 DDoS 保護層沒有可讓使用者設定的屬性，客戶也無法接觸。 DDoS 保護層級會將 Azure 當做平台來保護，防範大規模攻擊，它也會監視輸出流量和跨 Azure 區域的流量。 在虛擬網路上使用網路虛擬設備，可以由客戶針對較小規模的攻擊設定多幾層的彈性，不會令平台層級保護破功。 DDoS 的實際執行範例；如果網際網路對向的 IP 位址遭到大規模 DDoS 攻擊，Azure 會偵測攻擊的來源，並在有問題的流量到達其目的地之前中止它。 在大多數情況下，受到攻擊的端點不會被攻擊影響。 在少數端點會受影響的情況下，其流量不會影響到其他端點，只會影響受攻擊的端點。 因此，其他客戶和服務也不會看到此攻擊所造成的影響。 請務必注意，Azure DDoS 只尋找大規模攻擊。 在超過平台層級保護臨界值之前，您的特定服務有可能被覆蓋。 例如，單一 A0 IIS 伺服器上的網站，在 Azure 平台層級 DDoS 保護註冊威脅之前，可能被 DDoS 攻擊而離線。

2.  **公用 IP 位址**︰公用 IP 位址允許雲端服務或資源群組將公用的網際網路 IP 位址和連接埠公開；公用 IP 位址是透過服務端點、公用 IP 位址、應用程式閘道器以及其他 Azure 功能 (可向路由至您的資源的網際網路顯示公用 IP 位址) 啟用。 端點使用網路位址轉譯 (NAT) 將流量路由到 Azure 虛擬網路的內部位址和連接埠。 這個路徑是外部流量進入虛擬網路的主要方式。 公用 IP 位址可由使用者設定，用於判斷傳入什麼流量，以及該流量在虛擬網路上如何轉譯及轉譯至何處。

一旦流量到達虛擬網路，就有許多功能可派上用場。 Azure 虛擬網路是客戶連接工作負載的基礎，也是套用基本網路層級安全性的位置。 它是客戶在 Azure 中的私人網路 (虛擬網路覆疊)，具有下列功能和特性：

* **流量隔離**：虛擬網路是 Azure 平台上的流量隔離界限。 一個虛擬網路中的虛擬機器 (VM) 無法與不同虛擬網路中的 VM 直接通訊，即使兩個虛擬網路是由同一位客戶所建立也一樣。 隔離是很重要的屬性，可確保客戶 VM 和通訊仍然隱蔽於虛擬網路內。

>[!NOTE]
>流量隔離只會參考「輸入」虛擬網路的流量。 依預設，系統允許從 VNet 到網際網路的輸出流量，但如果 NSG 有需要可加以阻止。
>
>

* **多層拓撲**：虛擬網路可讓客戶配置子網路並為工作負載的不同元素或「層」指定個別位址空間，以定義多層拓撲。 這些邏輯分群和拓撲可讓客戶根據工作負載類型來定義不同的存取原則，還可控制各層之間的流量。
* **跨單位連線**：客戶可以在虛擬網路和多個內部部署網站或 Azure 中的其他虛擬網路之間，建立跨單位連線。 若要建立連線，客戶可以使用 VNet 對等互連、Azure VPN 閘道、協力廠商的網路虛擬設備或 ExpressRoute。 Azure 支援使用標準 IPsec/IKE 通訊協定和 ExpressRoute 私人連線能力的站對站 (S2S) VPN。
* **NSG** 可讓客戶依所需的精細度建立規則 (ACL)：網路介面、個別的 VM 或虛擬子網路。 客戶可以從客戶網路上的系統，透過跨單位連線或直接網際網路通訊，允許或拒絕虛擬網路內的工作負載，以控制存取權。
* **UDR** 和 **IP 轉送**可讓客戶定義虛擬網路內的不同層之間的通訊路徑。 客戶可以部署防火牆、IDS/IPS 和其他虛擬設備，並透過這些安全性設備來路由傳送網路流量，以強制執行安全性界限原則、稽核和檢查。
* Azure Marketplace 中的**網路虛擬設備**：Azure Marketplace 和 VM 映像資源庫中提供防火牆、負載平衡器和 IDS/IPS 等安全性設備。 客戶可以將這些設備部署到其虛擬網路，特別是安全性界限 (包括周邊網路子網路)，以完成多層式安全網路環境。

根據這些功能，下圖提供一個如何在 Azure 中建構周邊網路架構的範例：

![5]

## <a name="perimeter-network-characteristics-and-requirements"></a>周邊網路特性和需求
周邊網路是網路的前端，直接與來自網際網路的通訊正面接觸。 連入封包應該會先流經安全性設備，如防火牆、IDS 和 IPS 等，才會到達後端伺服器。 來自工作負載的網際網路繫結封包可能也會流經周邊網路中的安全性設備，經過原則強制執行、檢查和稽核之後，才會離開網路。 此外，周邊網路也可以裝載客戶虛擬網路與內部部署網路之間的跨單位 VPN 閘道。

### <a name="perimeter-network-characteristics"></a>周邊網路特性
請參見上圖，好的周邊網路應有下列特性︰

* 網際網路對向︰
  * 周邊網路子網路本身是網際網路對向，直接與網際網路進行通訊。
  * 公用 IP 位址、VIP 及/或服務端點會將網際網路流量傳遞給前端網路和裝置。
  * 來自網際網路的輸入流量會先通過安全性裝置，才會通過前端網路上其他資源。
  * 如果已啟用輸出安全性，流量會通過安全性裝置 (最後一個步驟)，然後才會傳遞到網際網路。
* 受保護的網路︰
  * 網際網路到核心基礎結構之間沒有直接的路徑。
  * 連到核心基礎結構的通道必須通過 NSG、防火牆或 VPN 裝置等安全性裝置。
  * 其他裝置不得橋接網際網路和核心基礎結構。
  * 在網際網路對向和受保護網路面向的周邊網路界限上的安全性裝置 (例如，上圖中顯示的兩個防火牆圖示)，實際上可能是單一虛擬設備，但針對每個界限各有不同的規則或介面。 例如，一部實體裝置，邏輯上隔開，可處理周邊網路兩個界限的負載。
* 其他常見的做法和條件約束：
  * 工作負載不得儲存商務關鍵資訊。
  * 只有獲授權的系統管理員才能存取和更新周邊網路設定和部署。

### <a name="perimeter-network-requirements"></a>周邊網路需求
為了發揮這些特性，請依照虛擬網路需求的這些指導方針，實作成功的周邊網路：

* **子網路架構：** 指定虛擬網路，使整個子網路專門作為周邊網路，與相同虛擬網路中的其他子網路分開。 這樣隔開可確保周邊網路與其他內部或私人子網路層之間的流量，一定會流經防火牆或 IDS/IPS 虛擬設備。  界限子網路上使用者定義的路由，需將此流量轉送到虛擬設備。
* **NSG：** 周邊網路子網路本身應該開啟，允許與網際網路通訊，但這不表示客戶應該略過 NSG。 遵循一般的安全性做法，以減少網路介面暴露在網際網路中。 鎖定可以存取部署或特定應用程式通訊協定和已開啟連接埠的遠端位址範圍。 但可能在某些情況下，完全鎖定不一定行得通。 例如，如果客戶在 Azure 中有外部網站，則周邊網路應該允許從任何公開 IP 位址傳入的 Web 要求，但只應該開啟 Web 應用程式連接埠：80 連接埠 上的 TCP 和/或 443 連接埠上的 TCP。
* **路由表：** 周邊網路子網路本身必須能夠直接與網際網路通訊，但不應該允許未通過防火牆或安全性設備，就直接與後端或內部部署網路之間往返通訊。
* **安全性設備設定：** 為了路由傳送和檢查周邊網路與受保護網路其餘部分之間的封包，安全性設備 (如防火牆、IDS 和 IPS 裝置) 可以有多重主目錄。 但周邊網路和後端子網路可能有個別的 NIC。 周邊網路的 NIC會使用相應的 NSG 和周邊網路路由表，直接與網際網路往返通訊。 對於相應的後端子網路，連接到後端子網路的 NIC 會有更受限制的 NSG 和路由表。
* **安全性設備功能：** 部署在周邊網路中的安全性設備通常會執行下列功能：
  * 防火牆：對連入要求強制執行防火牆規則或存取控制原則。
  * 威脅偵測和防止：偵測並減輕來自網際網路的惡意攻擊。
  * 稽核和記錄：維護詳細記錄供稽核和分析。
  * 反向 Proxy︰將連入要求重新導向至對應的後端伺服器。 此重新導向包括將前端裝置上的目的地位址 (通常是防火牆) 對應和轉譯到後端伺服器位址。
  * 正向 Proxy：提供 NAT 並稽核從虛擬網路內開始到網際網路的通訊。
  * 路由器：轉送虛擬網路內的傳入流量和跨子網路的流量。
  * VPN 裝置：作為內部部署網路上的客戶與 Azure 虛擬網路之間跨單位 VPN 連線的跨單位 VPN 閘道。
  * VPN 伺服器：接受連接到 Azure 虛擬網路的 VPN 用戶端。

> [!TIP]
> 隔開下列兩個群組：獲授權可存取周邊網路安全性裝置的個人，與獲授權成為應用程式開發、部署或作業系統管理員的個人。 將這些群組隔開可區分權責，防止單一個人略過應用程式安全性和網路安全性控制。
>
>

### <a name="questions-to-be-asked-when-building-network-boundaries"></a>組建網路界限時的疑問
在本節中，除非特別說明，否則「網路」一詞是指由訂用帳戶管理員所建立的私人 Azure 虛擬網路。 此詞非指 Azure 內的基礎實體網路。

此外，Azure 虛擬網路通常用來擴充傳統的內部部署網路。 站對站或 ExpressRoute 混合式網路功能解決方案可以與周邊網路架構合併。 此混合連結在組建網路安全性界限時，是很重要的考量。

當您建立有周邊網路和多個安全性界限的網路時，需要回答下列三個重要問題。

#### <a name="1-how-many-boundaries-are-needed"></a>1) 需要多少界限？
第一個決策點是決定在特定案例下需要多少安全性界限：

* 單一界限：虛擬網路與網際網路之間的前端周邊網路是一個界限。
* 兩個界限：一個在周邊網路的網際網路端，另一個在 Azure 虛擬網路的周邊網路子網路和後端子網路之間。
* 三個界限：一個在周邊網路的網際網路端，一個在周邊網路和後端子網路之間，還有一個在後端子網路和內部部署網路之間。
* N 個界限︰可變的數字。 視安全性需求而定，特定網路中可套用的安全性界限數量沒有限制。

需要的界限類型和數量，依照公司的風險承受度和實作的特定案例而有所不同。 這通常是組織內的多個團隊共同達成的決策，通常包括風險和合規性小組、網路和平台小組及應用程式開發小組。 了解安全性、所涉及的資料及運用之技術的人，在此決策上應該要有發言權，以確保對每個實作表達適當的安全性立場。

> [!TIP]
> 應該儘可能以最少的界限來滿足特定情況下的安全性需求。 界限愈多，操作和疑難排解就愈困難，後續管理多個界限原則時的管理負荷也越沈重。 不過，沒有足夠的界限會增加風險。 取得平衡相當重要。
>
>

![6]

上圖中顯示有三個安全性界限網路的高層級檢視。 界限介於周邊網路和網際網路、Azure 前端和後端私人子網路，以及 Azure 後端子網路和內部部署公司網路之間。

#### <a name="2-where-are-the-boundaries-located"></a>2) 界限位於何處？
一旦決定界限數量，下一個決策點就是在何處實作它們。 通常有三個選項︰

* 使用網際網路型的中繼服務 (例如，雲端 Web 應用程式防火牆，本文中不討論)
* 在 Azure 中使用原生功能及/或網路虛擬設備
* 在內部部署網路上使用實體裝置

在純粹的 Azure 網路上，選項包括原生的 Azure 功能 (例如 Azure 負載平衡器)，或由 Azure 豐富的合作夥伴生態系統提供的網路虛擬設備 (例如檢查點防火牆)。

如果 Azure 與內部部署網路之間需要界限，安全性裝置可以位於連線的任一端 (或兩端)。 因此，必須決定放置安全性裝置的位置。

上圖中，網際網路到周邊網路及前端到後端的邊界完全在 Azure 內，而且必須是原生的 Azure 功能或網路虛擬設備。 在 Azure (後端子網路) 與公司網路之間的界限上，安全性裝置可以在 Azure 端或內部部署端，甚至是兩端的裝置組合。 任一選項都可能有顯著的優點和缺點，必須認真考慮。

例如，在內部部署網路端上使用現有的實體安全性裝置，就不需要新的裝置。 它只需要重新設定。 但缺點則是所有流量必須從 Azure 回到內部部署網路，才能被安全性裝置看到。 因此，如果將 Azure 到 Azure 流量強制送回內部部署網路來強制執行安全性原則，可能會造成顯著的延遲，影響應用程式效能和使用者體驗。

#### <a name="3-how-are-the-boundaries-implemented"></a>3) 如何實作界限？
每個安全性界限可能有不同的功能需求 (例如，周邊網路的網際網路端需要 IDS 和防火牆規則，但周邊網路與後端子網路之間只需要 ACL)。 決定使用哪個裝置 (或多少個裝置)，取決於案例和安全性需求。 在下一節中，範例 1、2 和 3 會討論一些可用的選項。 檢閱 Azure 原生網路功能和 Azure 中由合作夥伴生態系統提供的裝置，將可以找到大量選項來解決幾乎任何的案例。

另一個重要的實作決策點就是如何連接內部部署網路與 Azure。 您應該使用 Azure 虛擬閘道還是網路虛擬設備呢？ 下節 (範例 4、5 和 6) 對這些選項會有更詳細的描述和討論。

您可能也需要 Azure 虛擬網路之間的流量。 這些案例都會在日後加入。

一旦解答上述問題， [快速開始](#fast-start) 一節可協助您識別最適合特定案例的範例。

## <a name="examples-building-security-boundaries-with-azure-virtual-networks"></a>範例：使用 Azure 虛擬網路組建安全性界限
### <a name="example-1-build-a-perimeter-network-to-help-protect-applications-with-nsgs"></a>範例 1：建置周邊網路，使用 NSG 協助保護應用程式
[回到快速開始](#fast-start) | [此範例的詳細組建指示][Example1]

[![7]][7]

#### <a name="environment-description"></a>環境描述
此範例中的訂用帳戶包含下列資源：

- 單一資源群組
- 含 “FrontEnd” 和 “BackEnd” 兩個子網路的虛擬網路
- 套用至這兩個子網路的單一網路安全性群組
- 一個代表應用程式 Web 伺服器的 Windows Server (“IIS01”)
- 兩個代表應用程式後端伺服器的 Windows Server (“AppVM01”、“AppVM02”)
- 一個代表 DNS 伺服器的 Windows Server ("DNS01")
- 與應用程式 Web 伺服器相關聯的公用 IP

如需指令碼和 Azure Resource Manager 範本，請參閱[詳細的組建指示][Example1]。

#### <a name="nsg-description"></a>NSG 描述
此範例會組建 NSG 群組，然後載入六個規則。

> [!TIP]
> 一般而言，您應該先建立特定的「允許」規則，再建立較一般的「拒絕」規則。 指定的優先順序決定要先評估哪些規則。 一旦發現流量適用特定規則，就不會再評估後續規則。 NSG 規則可以套用在輸入或輸出方向 (從子網路的觀點出發)。
>
>

指令碼會以宣告方式為輸入流量建置下列規則：

1. 允許內部 DNS 流量 (連接埠 53)。
2. 允許從網際網路到任何虛擬機器的 RDP 流量 (連接埠 3389)。
3. 允許從網際網路到 Web 伺服器 (IIS01) 的 HTTP 流量 (連接埠 80)。
4. 允許從 IIS01 到 AppVM1 的任何流量 (所有連接埠)。
5. 拒絕從網際網路到整個虛擬網路 (兩個子網路) 的任何流量 (所有連接埠)。
6. 拒絕從前端子網路到後端子網路的任何流量 (所有連接埠)。

這些規則繫結至每個子網路，如果 HTTP 要求是從網際網路輸入 Web 伺服器，則 3 (允許) 和 5 (拒絕) 兩個規則都會套用。 但因規則 3 具有較高的優先順序，所以只會套用規則 3，規則 5 不起作用。 因此會允許 HTTP 要求送往 Web 伺服器。 如果相同的流量嘗試抵達 DNS01 伺服器，規則 5 (拒絕) 會先適用，因此不會允許流量傳遞給伺服器。 規則 6 (拒絕) 封鎖前端子網路和後端子網路交談 (規則 1 和 4 所允許的流量除外)。 萬一攻擊者危害前端的 Web 應用程式，這條規則可以保護後端網路。 攻擊者對後端「受保護」網路的存取受到限制 (僅限對 AppVM01 伺服器公開的資源)。

有一個預設輸出規則可允許流量外流到網際網路。 在此範例中，我們會允許輸出流量，並不會修改任何輸出規則。 若要鎖定兩個方向的流量，則需要使用者定義的路由 (請見範例 3)。

#### <a name="conclusion"></a>結論
這個範例隔離後端子網路與輸入流量的方式相當直接簡單。 如需詳細資訊，請參閱[詳細的組建指示][Example1]。 這些指示包括：

* 如何使用傳統 PowerShell 指令碼組建這個周邊網路。
* 如何使用 Azure Resource Manager 範本組建這個周邊網路。
* 每個 NSG 命令的詳細描述。
* 詳細的流量流程案例，顯示每一層允許或拒絕流量的方式。


### <a name="example-2-build-a-perimeter-network-to-help-protect-applications-with-a-firewall-and-nsgs"></a>範例 2：建置周邊網路，使用防火牆和 NSG 協助保護應用程式
[回到快速開始](#fast-start) | [此範例的詳細組建指示][Example2]

[![8]][8]

#### <a name="environment-description"></a>環境描述
此範例中的訂用帳戶包含下列資源：

* 單一資源群組
* 含 “FrontEnd” 和 “BackEnd” 兩個子網路的虛擬網路
* 套用至這兩個子網路的單一網路安全性群組
* 一個網路虛擬設備，在此範例中為防火牆，連接到 Frontend 子網路
* 一個代表應用程式 Web 伺服器的 Windows Server (“IIS01”)
* 兩個代表應用程式後端伺服器的 Windows Server (“AppVM01”、“AppVM02”)
* 一個代表 DNS 伺服器的 Windows Server ("DNS01")

如需指令碼和 Azure Resource Manager 範本，請參閱[詳細的組建指示][Example2]。

#### <a name="nsg-description"></a>NSG 描述
此範例會組建 NSG 群組，然後載入六個規則。

> [!TIP]
> 一般而言，您應該先建立特定的「允許」規則，再建立較一般的「拒絕」規則。 指定的優先順序決定要先評估哪些規則。 一旦發現流量適用特定規則，就不會再評估後續規則。 NSG 規則可以套用在輸入或輸出方向 (從子網路的觀點出發)。
>
>

指令碼會以宣告方式為輸入流量建置下列規則：

1. 允許內部 DNS 流量 (連接埠 53)。
2. 允許從網際網路到任何虛擬機器的 RDP 流量 (連接埠 3389)。
3. 允許從網際網路到網路虛擬設備 (防火牆) 的任何流量 (所有連接埠)。
4. 允許從 IIS01 到 AppVM1 的任何流量 (所有連接埠)。
5. 拒絕從網際網路到整個虛擬網路 (兩個子網路) 的任何流量 (所有連接埠)。
6. 拒絕從前端子網路到後端子網路的任何流量 (所有連接埠)。

這些規則繫結至每個子網路，如果 HTTP 要求是從網際網路輸入防火牆，則 3 (允許) 和 5 (拒絕) 兩個規則都會套用。 但因規則 3 具有較高的優先順序，所以只會套用規則 3，規則 5 不起作用。 因此會允許 HTTP 要求送往防火牆。 如果相同的流量嘗試到達 IIS01 伺服器，即使是在 Frontend 子網路上，則會套用規則 5 (拒絕)，因此不會允許流量傳遞給伺服器。 規則 6 (拒絕) 封鎖前端子網路和後端子網路交談 (規則 1 和 4 所允許的流量除外)。 萬一攻擊者危害前端的 Web 應用程式，這條規則可以保護後端網路。 攻擊者對後端「受保護」網路的存取受到限制 (僅限對 AppVM01 伺服器公開的資源)。

有一個預設輸出規則可允許流量外流到網際網路。 在此範例中，我們會允許輸出流量，並不會修改任何輸出規則。 若要鎖定兩個方向的流量，則需要使用者定義的路由 (請見範例 3)。

#### <a name="firewall-rule-description"></a>防火牆規則描述
防火牆上應該建立轉送規則。 此範例只會將網際網路流量往內路由傳送到防火牆，再傳送到 Web 伺服器，因此只需要一個轉送網路位址轉譯 (NAT) 規則。

轉寄規則接受任何叫用防火牆嘗試連線到 HTTP (連接埠 80 或 HTTPS 為 443) 的輸入來源位址。 它會傳出防火牆的本機介面，重新導向至 IP 位址 10.0.1.5 的 Web 伺服器。

#### <a name="conclusion"></a>結論
這個範例使用防火牆保護應用程式並隔離後端子網路與輸入流量的方式相當直接簡單。 如需詳細資訊，請參閱[詳細的組建指示][Example2]。 這些指示包括：

* 如何使用傳統 PowerShell 指令碼組建這個周邊網路。
* 如何使用 Azure Resource Manager 範本組建這個範例。
* 每個 NSG 命令和防火牆規則的詳細描述。
* 詳細的流量流程案例，顯示每一層允許或拒絕流量的方式。

### <a name="example-3-build-a-perimeter-network-to-help-protect-networks-with-a-firewall-and-udr-and-nsg"></a>範例 3︰建置周邊網路，使用防火牆、UDR 及 NSG 協助保護網路
[回到快速開始](#fast-start) | [此範例的詳細組建指示][Example3]

[![9]][9]

#### <a name="environment-description"></a>環境描述
此範例中的訂用帳戶包含下列資源：

* 單一資源群組
* 一個虛擬網路，包含三個子網路：“SecNet”、“FrontEnd”、“BackEnd”
* 一個網路虛擬設備，在本例中為防火牆，連接到 SecNet 子網路
* 一個代表應用程式 Web 伺服器的 Windows Server (“IIS01”)
* 兩個代表應用程式後端伺服器的 Windows Server (“AppVM01”、“AppVM02”)
* 一個代表 DNS 伺服器的 Windows Server ("DNS01")

如需指令碼和 Azure Resource Manager 範本，請參閱[詳細的組建指示][Example3]。

#### <a name="udr-description"></a>UDR 描述
根據預設，下列系統路由的定義如下：

        Effective routes :
         Address Prefix    Next hop type    Next hop IP address Status   Source     
         --------------    -------------    ------------------- ------   ------     
         {10.0.0.0/16}     VNETLocal                            Active   Default    
         {0.0.0.0/0}       Internet                             Active   Default    
         {10.0.0.0/8}      Null                                 Active   Default    
         {100.64.0.0/10}   Null                                 Active   Default    
         {172.16.0.0/12}   Null                                 Active   Default    
         {192.168.0.0/16}  Null                                 Active   Default

VNETLocal 一定是一或多個定義的位址首碼，這些首碼組成該特定網路定的虛擬網路 (也就是它會依虛擬網路而變，取決於各個特定虛擬網路的定義)。 其餘系統路由則為靜態和預設值，如表中所述。

本例中會建立兩個路由表，分別用於前端和後端子網路。 每個資料表都會載入適用於給定子網路的靜態路由。 在此範例中，每個資料表會有三個路由，負責將所有流量 (0.0.0.0/0) 導向防火牆 (下一個躍點 = 虛擬設備 IP 位址)：

1. 未定義下一個躍點的本機子網路流量，以允許本機子網路流量略過防火牆。
2. 下一個躍點定義為防火牆的虛擬網路。 這個「下一個躍點」會覆寫允許本機虛擬網路流量直接路由的預設規則。
3. 下一個躍點定義為防火牆的所有其餘流量 (0/0)。

> [!TIP]
> UDR 中沒有本機子網路項目，會中斷本機子網路通訊。
>
> * 在我們的範例中，10.0.1.0/24 指向 VNETLocal 相當重要 ！ 若不這麼做，離開 Web 伺服器 (10.0.1.4)、目的地是另一部本機伺服器 (例如 10.0.1.25) 的封包都會失敗，因為它們將會傳送至 NVA。 NVA 會將它傳送至子網路，而子網路會再將它傳送至 NVA，形成無限迴圈。
> * 在連線至不同子網路且有多個 NIC 的設備上 (通常屬於傳統的內部部署設備)，形成路由迴圈的機會通常比較高。
>
>

路由表一經建立就必須將繫結至其子網路。 前端子網路路由表在建立並繫結至子網路後，應該會像下面的輸出：

        Effective routes :
         Address Prefix    Next hop type    Next hop IP address Status   Source     
         --------------    -------------    ------------------- ------   ------     
         {10.0.1.0/24}     VNETLocal                            Active
         {10.0.0.0/16}     VirtualAppliance 10.0.0.4            Active    
         {0.0.0.0/0}       VirtualAppliance 10.0.0.4            Active

> [!NOTE]
> UDR 現在可套用至 ExpressRoute 線路所連線的閘道子網路。
>
> 範例 3 和 4 示範如何使用 ExpressRoute 或站對站網路啟用周邊網路。
>
>

#### <a name="ip-forwarding-description"></a>IP 轉送描述
IP 轉送是 UDR 的隨附功能。 IP 轉送是虛擬設備的一項設定，以允許它接收不是要特別傳送到設備的流量，再將流量轉送到其最終目的地。

舉例來說，如果 AppVM01 對 DNS01 伺服器提出要求，UDR 會將此流量路由傳送到防火牆。 在啟用 IP 轉送時，目的地為 DNS01 (10.0.2.4) 的流量會被設備 (10.0.0.4) 所接受，然後轉送到其最終目的地 (10.0.2.4)。 若防火牆未啟用 IP 轉送，即使路由表將防火牆作為下一個躍點，流量也不會被設備所接受。 如果要使用虛擬設備，請務必記得一同啟用 IP 轉送和 UDR。

#### <a name="nsg-description"></a>NSG 描述
本例會組建 NSG 群組，然後在其中載入單一規則。 這個群組接著只會繫結到前端和後端子網路 (不會繫結到 SecNet)。 指令碼會以宣告方式建置下列規則：

* 拒絕從網際網路到整個虛擬網路 (所有子網路) 的任何流量 (所有連接埠)。

雖然本例中使用 NSG，但它的主要目的是作為防止人為設定錯誤的第二道防線。 目標是要封鎖從網際網路到前端或後端子網路的所有輸入流量。 流量應該只從 SecNet 子網路到防火牆 (然後，如果合適，再到前端或後端子網路)。 此外，在具有 UDR 規則時，確實能夠流往前端或後端子網路的流量皆會被引導出來並流向防火牆 (多虧有了 UDR)。 防火牆會將此流量視為非對稱流量，並且會捨棄輸出流量。 因此子網路有三層的安全性保護︰

* 任何前端或後端 NIC 上沒有公用 IP 位址。
* NSG 拒絕來自網際網路的流量。
* 防火牆卸除對稱流量。

此範例的 NSG 有一個有趣的地方，那就是它只有一個規則，此規則是為了要拒絕由網際網路流往整個虛擬網路 (包括安全性子網路) 的流量。 不過，由於 NSG 只會繫結至前端和後端子網路，因此不會對流往安全性子網路的輸入流量處理此規則。 如此一來，流量就會傳送到安全性子網路。

#### <a name="firewall-rules"></a>防火牆規則
防火牆上應該建立轉送規則。 因為防火牆會封鎖或轉送所有輸入、輸出和內部虛擬網路流量，所以需要許多防火牆規則。 此外，所有輸入流量都會抵達安全性服務的公用 IP 位址 (在不同連接埠上)，並由防火牆進行處理。 最佳做法是先繪製邏輯流量圖，再設定子網路和防火牆規則，以避免日後還要修改。 下圖是此範例中的防火牆規則的邏輯視圖：

![10]

> [!NOTE]
> 根據所使用的網路虛擬應用裝置，會有不同的管理連接埠。 在此範例中，所參考的 Barracuda NextGen 防火牆使用連接埠 22、801 和 807。 請參閱設備廠商的說明文件來尋找用於管理所使用裝置的確切連接埠。
>
>

#### <a name="firewall-rules-description"></a>防火牆規則描述
上述的邏輯圖中沒有顯示安全性子網路，因為子網路上只有防火牆這項資源。 圖中顯示防火牆規則以及這些規則在邏輯上是如何允許或拒絕流量的流動，而非顯示實際路由路徑。 此外，針對 RDP 流量選取的外部連接埠皆為較高範圍的連接埠 (8014 – 8026)，且在選取時會稍微配合本機 IP 位址的最後兩個八位元數字，以方便閱讀 (例如，本機伺服器位址 10.0.1.4 會與外部連接埠 8014 相關聯)。 不過您可以使用任何更高範圍的連接埠，只要連接埠未衝突即可。

本例中，我們需要七種類型的規則︰

* 外部規則 (適用於輸入流量)：
  1. 防火牆管理規則：此應用程式重新導向規則允許流量傳遞至網路虛擬設備的管理連接埠。
  2. RDP 規則 (適用於每部 Windows 伺服器)：這四個規則 (每部伺服器一個) 允許透過 RDP 管理個別伺服器。 根據所使用的網路虛擬設備功能而定，也可將四個 RDP 規則結合成一個規則。
  3. 應用程式流量規則：這些規則有兩個，第一個適用於前端 Web 流量，第二個適用於後端流量 (例如，Web 伺服器流往資料層)。 這些規則的設定取決於網路架構 (伺服器的放置位置) 和流量流動行為 (流量的流動方向，以及使用的連接埠)。
     * 第一個規則允許實際的應用程式流量抵達應用程式伺服器。 其他規則所考慮的是安全性和管理等方面的事項，而應用程式規則則是用來允許外部使用者或服務存取應用程式。 本例中，連接埠 80 上有單一 Web 伺服器。 因此單一防火牆應用程式規則會將流往外部 IP 的輸入流量，重新導向到 Web 伺服器的內部 IP 位址。 重新導向的流量工作階段會透過 NAT 轉譯到內部伺服器。
     * 第二個規則是後端規則，允許 Web 伺服器透過任何連接埠與 AppVM01 伺服器 (而非 AppVM02) 對談。
* 內部規則 (適用於內部虛擬網路流量)
  1. 輸出到網際網路規則：這個規則允許來自任何網路的流量傳遞至選定的網路。 此規則通常是防火牆上既有的預設規則，但會處於停用狀態。 在此範例中，應啟用此規則。
  2. DNS 規則：此規則只會允許 DNS (連接埠 53) 流量傳遞至 DNS 伺服器。 在這種環境中，大部分從前端到後端流量都會被封鎖。 這個規則特別允許來自任何本機子網路的 DNS。
  3. 子網路對子網路規則：此規則會允許後端子網路上的任何伺服器連接至前端子網路上的任何伺服器 (但不允許反向連接)。
* 保險規則 (適用於不符合上述任一規則的流量)：
  1. 拒絕所有流量規則：請一律將這一條拒絕規則作為最終規則 (以優先順序來看)，這樣一來，如果流量的流動行為無法符合上述任何規則時，便會由此規則將其捨棄。 這是預設規則，通常為現成啟用狀態。 通常不需要對此規則進行任何修改。

> [!TIP]
> 為簡化這個範例，第二個應用程式流量規則允許任何連接埠。 在真實案例中，應使用最特定的連接埠和位址範圍來降低這項規則被攻擊的範圍。
>
>

建立好上述所有規則後，請務必檢閱每個規則的優先順序，以確保可依需要允許或拒絕流量。 在此範例中，規則是依照優先順序排列。

#### <a name="conclusion"></a>結論
本範例較為複雜，但卻比之前的範例能更完整地保護和隔離網路。 (範例 2 只保護應用程式，範例 1 只隔離子網路)。 這項設計允許同時監視兩個方向的流量，且不只保護輸入應用程式伺服器，還會對此網路上的所有伺服器強制執行網路安全性原則。 此外，根據使用的應用裝置而定，還能達成完整的流量稽核和感知。 如需詳細資訊，請參閱[詳細的組建指示][Example3]。 這些指示包括：

* 如何使用傳統 PowerShell 指令碼組建這個範例周邊網路。
* 如何使用 Azure Resource Manager 範本組建這個範例。
* 每個 UDR、NSG 命令和防火牆規則的詳細描述。
* 詳細的流量流程案例，顯示每一層允許或拒絕流量的方式。

### <a name="example-4-add-a-hybrid-connection-with-a-site-to-site-virtual-appliance-vpn"></a>範例 4 新增使用站對站虛擬設備 VPN 的混合式連接
[回到快速開始](#fast-start) | 詳細的組建指示即將推出

[![11]][11]

#### <a name="environment-description"></a>環境描述
使用網路虛擬設備 (NVA) 的混合式網路可以加入至範例 1、2 或 3 所描述的任何周邊網路類型。

如上圖所示，透過網際網路的 VPN 連線 (站對站) 用於透過 NVA，將內部部署網路連接到 Azure 虛擬網路。

> [!NOTE]
> 使用 ExpressRoute 時如果啟用了 [Azure 公用對等] 選項，應該會建立靜態的路由。 這個靜態路由應該會路由至您公司網際網路外的 NVA VPN IP 位址，而且不是透過 ExpressRoute 連線。 ExpressRoute [Azure 公用對等] 選項所需要的 NAT 可中斷 VPN 工作階段。
>
>

一旦 VPN 備妥，NVA 就會變成所有網路和子網路的中央中樞。 防火牆轉送規則會決定允許、由 NAT 轉譯、重新導向或捨棄哪些流量流程 (即使是內部部署網路與 Azure 之間的流量流程也是這樣)。

應該謹慎考量流量流程，它們可能因為這種設計模式而最佳化或退化，視特定的使用案例而定。

使用範例 3 中組建的環境，然後加入站對站 VPN 混合式網路連線，會產生下列設計：

[![12]][12]

內部部署路由器或與用於 VPN 的 NVA 相容的任何其他網路裝置，就是 VPN 用戶端。 此實體裝置負責使用 NVA 起始並維護您的 VPN 連線。

就 NVA 而言，網路在邏輯上就像四個不同的「安全性區域 」，而以 NVA 上的規則為總指揮，統管這些區域之間的流量：

![13]

#### <a name="conclusion"></a>結論
將站對站 VPN 混合式網路連線加入至 Azure 虛擬網路，能夠安全地將內部部署網路擴充至 Azure。 使用 VPN 連線時，流量會加密並透過網際網路來路由傳送。 本例中使用 NVA 集中強制執行和管理安全性原則。 如需詳細資訊，請參閱＜詳細的組建指示＞(即將推出)。 這些指示包括：

* 如何使用 PowerShell 指令碼組建這個範例周邊網路。
* 如何使用 Azure Resource Manager 範本組建這個範例。
* 顯示此設計中流量如何流通的詳細流量流程案例。

### <a name="example-5-add-a-hybrid-connection-with-a-site-to-site-azure-vpn-gateway"></a>範例 5 新增使用站對站 Azure VPN 閘道的混合式連接
[回到快速開始](#fast-start) | 詳細的組建指示即將推出

[![14]][14]

#### <a name="environment-description"></a>環境描述
使用 Azure VPN 閘道的混合式網路可以加入至範例 1 或 2 所描述的任一種周邊網路類型。

如上圖所示，透過網際網路的 VPN 連線 (站對站) 用於透過 Azure VPN 閘道，將內部部署網路連接到 Azure 虛擬網路。

> [!NOTE]
> 使用 ExpressRoute 時如果啟用了 [Azure 公用對等] 選項，應該會建立靜態的路由。 這個靜態路由應該會路由至您公司網際網路外的 NVA VPN IP 位址，而且不是透過 ExpressRoute WAN。 ExpressRoute [Azure 公用對等] 選項所需要的 NAT 可中斷 VPN 工作階段。
>
>

下圖顯示這個範例的兩個網路邊緣。 在第一個邊緣，NVA 和 NSG 控制 Azure 內部網路以及 Azure 和網際網路之間的流量流程。 第二個邊緣是 Azure VPN 閘道，這是內部部署與 Azure 之間分開隔離的網路邊緣。

應該謹慎考量流量流程，它們可能因為這種設計模式而最佳化或退化，視特定的使用案例而定。

使用範例 1 中組建的環境，然後加入站對站 VPN 混合式網路連線，會產生下列設計：

[![15]][15]

#### <a name="conclusion"></a>結論
將站對站 VPN 混合式網路連線加入至 Azure 虛擬網路，能夠安全地將內部部署網路擴充至 Azure。 使用原生 Azure VPN 閘道時，流量會由 IPSec 加密並透過網際網路來路由傳送。 此外，使用 Azure VPN 閘道的成本也較低 (不像協力廠商 NVA 一樣需要額外的授權成本)。 這個選項是範例 1 最經濟實惠的地方，因為不使用任何 NVA。 如需詳細資訊，請參閱＜詳細的組建指示＞(即將推出)。 這些指示包括：

* 如何使用 PowerShell 指令碼組建這個範例周邊網路。
* 如何使用 Azure Resource Manager 範本組建這個範例。
* 顯示此設計中流量如何流通的詳細流量流程案例。

### <a name="example-6-add-a-hybrid-connection-with-expressroute"></a>範例 6：新增使用 ExpressRoute 的混合式連接
[回到快速開始](#fast-start) | 詳細的組建指示即將推出

[![16]][16]

#### <a name="environment-description"></a>環境描述
使用 ExpressRoute 私用對等連線的混合式網路可以加入至範例 1 或 2 所描述的任一種周邊網路類型。

如上圖所示，ExpressRoute 私用對等提供內部部署網路與 Azure 虛擬網路之間的直接連線。 流量只會在服務提供者網路和 Microsoft Azure 網路上流動，永遠不會到達網際網路。

> [!TIP]
> 使用 ExpressRoute 讓公司網路流量遠離網際網路。 也能符合 ExpressRoute 提供者的服務等級協定。 Azure 閘道在 ExpressRoute 上的傳遞速度高達 10 Gbps，而使用站對站 VPN 時，Azure 閘道的最大輸送量為 200 Mbps。
>
>

如下圖所示，使用此選項後，環境現在有兩個網路邊緣。 NVA 和 NSG 控制 Azure 網路內部及 Azure 與網際網路之間的流量流程，而閘道是內部部署與 Azure 之間的一個獨立且隔離的網路邊緣。

應該謹慎考量流量流程，它們可能因為這種設計模式而最佳化或退化，視特定的使用案例而定。

使用範例 1 中組建的環境，然後加入 ExpressRoute 混合式網路連線，會產生下列設計：

[![17]][17]

#### <a name="conclusion"></a>結論
加入 ExpressRoute 私用對等網路連線，能夠以安全、短延遲、高效能的方式將內部部署網路擴充至 Azure。 此外，如這個範例所示，使用原生 Azure 閘道的成本也較低 (不像協力廠商 NVA 一樣需要額外授權)。 如需詳細資訊，請參閱＜詳細的組建指示＞(即將推出)。 這些指示包括：

* 如何使用 PowerShell 指令碼組建這個範例周邊網路。
* 如何使用 Azure Resource Manager 範本組建這個範例。
* 顯示此設計中流量如何流通的詳細流量流程案例。

## <a name="references"></a>參考
### <a name="helpful-websites-and-documentation"></a>實用的網站和文件
* 使用 Azure Resource Manager 存取 Azure：
* 使用 PowerShell 存取 Azure：[https://docs.microsoft.com/powershell/azureps-cmdlets-docs/](/powershell/azure/overview)
* 虛擬網路文件：[https://docs.microsoft.com/azure/virtual-network/](https://docs.microsoft.com/azure/virtual-network/)
* 網路安全性群組文件：[https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg](virtual-network/security-overview.md)
* 使用者定義的路由文件：[https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview](virtual-network/virtual-networks-udr-overview.md)
* Azure 虛擬閘道：[https://docs.microsoft.com/azure/vpn-gateway/](https://docs.microsoft.com/azure/vpn-gateway/)
* 站對站 VPN：[https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell](vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)
* ExpressRoute 文件 (請務必閱讀〈使用者入門〉和〈作法〉兩節)：[https://docs.microsoft.com/azure/expressroute/](https://docs.microsoft.com/azure/expressroute/)

<!--Image References-->
[0]: ./media/best-practices-network-security/flowchart.png "Security Options Flowchart"
[2]: ./media/best-practices-network-security/azuresecurityfeatures.png "Azure Security Features"
[3]: ./media/best-practices-network-security/dmzcorporate.png "A DMZ in a Corporate network"
[4]: ./media/best-practices-network-security/azuresecurityarchitecture.png "Azure Security Architecture"
[5]: ./media/best-practices-network-security/dmzazure.png "A DMZ in an Azure Virtual Network"
[6]: ./media/best-practices-network-security/dmzhybrid.png "Hybrid Network with Three Security Boundaries"
[7]: ./media/best-practices-network-security/example1design.png "Inbound DMZ with NSG"
[8]: ./media/best-practices-network-security/example2design.png "Inbound DMZ with NVA and NSG"
[9]: ./media/best-practices-network-security/example3design.png "Bi-directional DMZ with NVA, NSG, and UDR"
[10]: ./media/best-practices-network-security/example3firewalllogical.png "Logical View of the Firewall Rules"
[11]: ./media/best-practices-network-security/example3designoptions.png "DMZ with NVA Connected Hybrid Network"
[12]: ./media/best-practices-network-security/example4designs2s.png "DMZ with NVA Connected Using a Site-to-Site VPN"
[13]: ./media/best-practices-network-security/example4networklogical.png "Logical Network from NVA Perspective"
[14]: ./media/best-practices-network-security/example5designoptions.png "DMZ with Azure Gateway Connected Site-to-Site Hybrid Network"
[15]: ./media/best-practices-network-security/example5designs2s.png "DMZ with Azure Gateway Using Site-to-Site VPN"
[16]: ./media/best-practices-network-security/example6designoptions.png "DMZ with Azure Gateway Connected ExpressRoute Hybrid Network"
[17]: ./media/best-practices-network-security/example6designexpressroute.png "DMZ with Azure Gateway Using an ExpressRoute Connection"

<!--Link References-->
[TrustCenter]: https://azure.microsoft.com/support/trust-center/compliance/
[Example1]: ./virtual-network/virtual-networks-dmz-nsg.md
[Example2]: ./virtual-network/virtual-networks-dmz-nsg-fw-asm.md
[Example3]: ./virtual-network/virtual-networks-dmz-nsg-fw-udr-asm.md
[Example4]: ./virtual-network/virtual-networks-hybrid-s2s-nva-asm.md
[Example5]: ./virtual-network/virtual-networks-hybrid-s2s-agw-asm.md
[Example6]: ./virtual-network/virtual-networks-hybrid-expressroute-asm.md
[Example7]: ./virtual-network/virtual-networks-vnet2vnet-direct-asm.md
[Example8]: ./virtual-network/virtual-networks-vnet2vnet-transit-asm.md
