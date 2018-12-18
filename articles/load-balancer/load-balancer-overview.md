---
title: Azure 負載平衡器概觀 | Microsoft Docs
description: Azure 負載平衡器功能、架構和實作的概觀。 了解 Load Balancer 的運作方式，並將其用於雲端。
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: load-balancer
Customer intent: As an IT administrator, I want to learn more about the Azure Load Balancer service and what I can use it for.
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/20/2018
ms.author: kumud
ms.custom: mvc
ms.openlocfilehash: 618b00906a799e1b8cfcfac5ee6bcc3a714c2f87
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918737"
---
# <a name="what-is-azure-load-balancer"></a>什麼是 Azure Load Balancer？

您可以使用 Azure Load Balancer 調整您的應用程式，並為您的服務建立高可用性。 對於所有 TCP 和 UDP 應用程式，Load Balancer 可支援傳入和傳出的場合、提供低延遲和高輸送量，且最多可相應增加為數百萬個流程。  

Load Balancer 會根據規則和健康情況探查，將抵達 Load Balancer 前端的新輸入流程，分送給各個後端集區執行個體。 

此外，公用 Load Balancer 可以將虛擬網路內虛擬機器 (VM) 的私人 IP 位址轉譯成公用 IP 位址，為虛擬機器提供輸出連線。

Azure Load Balancer 有兩種 SKU：基本和標準。 它們的規模、功能、價格不同。 任何可在基本 Load Balancer 建立的案例，也可以使用標準 Load Balancer 來建立，雖然方法可能稍有不同。 在您摸索認識 Load Balancer 時，請務必熟悉基本功能以及 SKU 專屬的差異。

## <a name="why-use-load-balancer"></a>為什麼要使用 Load Balancer？ 

您可以使用 Azure Load Balancer：

* 對 VM 的連入網際網路流量進行負載平衡。 這種設定稱為[公用 Load Balancer](#publicloadbalancer)。
* 對虛擬網路內 VM 之間的流量進行負載平衡。 在混合式的案例中，您也可以從內部部署網路連線到 Load Balancer 前端。 這兩種案例使用的設定稱為[內部 Load Balancer](#internalloadbalancer)。
* 利用輸入網路位址轉譯 (NAT) 規則，從連接埠將流量轉送至特定 VM 上的特定連接埠。
* 藉由使用公用 Load Balancer，為虛擬網路內的 VM 提供[輸出連線能力](load-balancer-outbound-connections.md)。


>[!NOTE]
> Azure 會為您的案例提供一套完整受控的負載平衡解決方案。 如果您要尋找傳輸層安全性 (TLS) 通訊協定終止 (「SSL 卸載」) 或每一 HTTP/HTTPS 要求的應用程式層處理，請檢閱[應用程式閘道](../application-gateway/application-gateway-introduction.md)。 如果您要尋找全域 DNS 負載平衡，請檢閱[流量管理員](../traffic-manager/traffic-manager-overview.md)。 視需要結合這些解決方案，可能有益於您的端對端案例。

## <a name="what-are-load-balancer-resources"></a>什麼是 Load Balancer 資源？

Load Balancer 資源可以公用 Load Balancer 或內部 Load Balancer 的形式存在。 Load Balancer 資源的運作方式可用前端、規則、健康情況探查和後端集區定義來表示。 從虛擬機器指定後端集區，藉以將虛擬機器放在後端集區中。

Load Balancer 資源是一種物件，而您可以在其中表示 Azure 應如何對其多租用戶基礎結構設計程式，以達到您想要建立的案例。 Load Balancer 資源與實際的基礎結構之間沒有直接關聯性。 建立 Load Balancer 並不會建立執行個體，而且隨時都有容量可用。 

## <a name="fundamental-load-balancer-features"></a>Load Balancer 的基本功能

Load Balancer 為 TCP 和 UDP 應用程式提供下列基本功能：

* **負載平衡**

    有了 Azure Load Balancer，您就能建立負載平衡規則，將抵達前端的流量分送給各個後端集區執行個體。 Load Balancer 使用雜湊型演算法來分送輸入流程，並據以重寫流向後端集區執行個體的流程標頭。 當健康情況探查表示後端端點狀況良好時，伺服器便可以接收新的流程。
    
    Load Balancer 預設會使用 5-tuple 的雜湊 (由來源 IP 位址、來源連接埠、目的地 IP 位址、目的地連接埠、IP 通訊協定數目所組成)，將流程對應至可用的伺服器。 您可以選擇建立特定來源 IP 位址的親和性，做法是替指定規則加入 2-tuple 或 3-tuple 的雜湊。 相同封包流程的所有封包會抵達相同執行個體，執行個體位於已負載平衡的前端之後。 當用戶端從相同的來源 IP 起始新的流程時，來源連接埠會變更。 結果是產生的 5-tuple 可能會將流量送往不同的後端端點。

    如需詳細資訊，請參閱 [Load Balancer 分送模式](load-balancer-distribution-mode.md)。 下列影像顯示雜湊型分送：

    ![雜湊型分散](./media/load-balancer-overview/load-balancer-distribution.png)

    *圖：雜湊型分送*

* **連接埠轉送**

    有了 Load Balancer，您就能建立輸入 NAT 規則，將特定前端 IP 位址特定連接埠的流量，從連接埠轉送到虛擬網路內特定後端執行個體的特定連接埠。 這也是使用和負載平衡同樣的雜湊型分送來完成的。 此功能的常見案例是透過遠端桌面通訊協定 (RDP) 或安全殼層 (SSH) 工作階段連線至 Azure 虛擬網路內的個別 VM 執行個體。 您可以將多個內部端點對應至相同前端 IP 位址的各個連接埠。 您可以使用這些端點透過網際網路遠端管理 VM，而不需要額外的跳箱。

* **透明且相容的應用程式**

    Load Balancer 不會直接與 TCP 或 UDP 或應用程式層互動，可以支援任何 TCP 或 UDP 應用程式案例。  Load Balancer 不會終止或產生流程，也不會與流程承載互動，不提供任何應用程式層的閘道功能，而且通訊協定交握一律直接在用戶端與後端集區的執行個體之間發生。  輸入流程的回應一律是來自虛擬機器的回應。  當流程到達虛擬機器時，也會保留原始的來源 IP 位址。  以下透過幾個範例來進一步說明透明度：
    - 每個端點只會由 VM 回應。  例如，TCP 交握一律是在用戶端和選取的後端虛擬機器之間發生。  前端要求的回應是由後端虛擬機器所產生的回應。 當您已順利驗證前端連線時，就代表至少驗證了一個後端虛擬機器的端對端連線。
    - 對 Load Balancer 而言，應用程式承載均 透明可見，且可支援任何 UDP 或 TCP 應用程式。 至於需要對處理或操作每個 HTTP 要求之應用程式層承載的工作負載 (如 HTTP URL 剖析)，則應使用[應用程式閘道](https://azure.microsoft.com/services/application-gateway)這類第 7 層 Load Balancer。
    - 由於 Load Balancer 無從驗證 TCP 承載，而且未提供 TLS 卸載 (「SSL」)，因此可以使用 Load Balancer 建立端對端加密的案例，並藉由終止 VM 本身的 TLS 連線來相應放大 TLS 應用程式。  舉例而言，您的 TLS 工作階段金鑰處理容量只會受限於您新增至後端集區的虛擬機器類型和數量。  如果您需要「SSL 卸載」、應用程式層處理，或是想要將憑證管理委派給 Azure，應改用 Azure 的第 7 層 Load Balancer [應用程式閘道](https://azure.microsoft.com/services/application-gateway)。
        

* **自動重新設定**

    當您相應增加或減少執行個體時，Load Balancer 本身會立即重新設定。 從後端集區新增或移除 VM 會重新設定 Load Balancer，而不需要在 Load Balancer 資源上進行其他作業。

* **健康情況探查**

    若要判斷後端集區中執行個體的健康情況，Load Balancer 會使用您定義的健康情況探查。 當探查無法回應時，Load Balancer 會停止傳送新的連線至狀況不良的執行個體。 現有連線不會受到影響，會繼續維持連線直到應用程式終止流程、發生閒置逾時或 VM 關機。
     
    Load Balancer 會為 TCP、HTTP 和 HTTPS 端點提供[不同的健康情況探查類型](load-balancer-custom-probe-overview.md#types)。

    此外，使用傳統雲端服務時，允許使用其他類型：[客體代理程式](load-balancer-custom-probe-overview.md#guestagent)。  這應該被視為健康情況探查的最後手段，而且有其他可行的選項時建議不要使用。
    
* **輸出連線 (SNAT)**

    所有從虛擬網路內私人 IP 位址送往網際網路上公用 IP 位址的輸出流程，皆可以轉譯為 Load Balancer 的前端 IP 位址。 當公用前端透過負載平衡規則繫結到後端虛擬機器時，Azure 會自動將輸出連線轉譯成公用前端 IP 位址。

    * 能夠輕鬆進行服務的升級及災害復原，因為前端可以動態對應到服務的另一個執行個體。
    * 讓存取控制清單 (ACL) 管理變得更容易。 當服務相應增加、相應減少或重新部署時，根據前端 IP 表示的 ACL 不會變更。  將輸出連線轉譯成數量比機器更少的 IP 地址，可以減少白清單的負擔。

    如需詳細資訊，請參閱[輸出連線](load-balancer-outbound-connections.md)。

除了這些基本功能，標準 Load Balancer 還有其他的 SKU 專屬功能。 請檢閱本文的其餘部分以了解詳情。

## <a name="skus"></a>Load Balancer SKU 的比較

Load Balancer 支援基本和標準 SKU，兩者在案例規模、功能、價格方面各不相同。 任何可在基本 Load Balancer 建立的案例，也可以使用標準 Load Balancer 來建立。 事實上，這兩個 SKU 的 API 類似，都是透過指定 SKU 來叫用。 從 2017-08-01 API 開始，已提供可支援 Load Balancer 和公用 IP 之 SKU 的 API。 這兩個 SKU 具有相同的一般 API 和結構。

不過，根據選擇的 SKU 不同，完整的案例設定可能稍有不同。 當文章僅適用於特定 SKU 時，Load Balancer 文件才會加以標注。 若要比較並了解其中的差異，請參閱下表。 如需詳細資訊，請參閱[標準 Load Balancer 概觀](load-balancer-standard-overview.md)。

>[!NOTE]
> 新的設計應該使用 Standard Load Balancer。 

獨立 VM、可用性設定組和虛擬機器擴展集都只能和一個 SKU 連線，永遠不能和兩者同時連線。 與公用 IP 位址搭配使用時，Load Balancer 和公用 IP 位址的 SKU 必須相符。 Load Balancer 和公用 IP 的 SKU 不可變動。

雖無強制性，但您最好明確地指定 SKU。  現階段，會將所需的變更保持在最少量的狀態。 若未指定 SKU，系統會認為您打算使用基本 SKU 的 2017-08-01 API 版本。

>[!IMPORTANT]
>標準 Load Balancer 是新的 Load Balancer 產品，大體上是基本 Load Balancer 的超集。 兩種產品間有重要且刻意製造的差別。 任何可在基本 Load Balancer 建立的端對端案例，也可以使用標準 Load Balancer 來建立。 如果您已經習慣使用基本 Load Balancer，請讓自己熟悉標準 Load Balancer，以了解標準和基本在行為上的最新變更以及其影響。 請仔細檢閱這一節。

[!INCLUDE [comparison table](../../includes/load-balancer-comparison-table.md)]

如需詳細資訊，請參閱 [Load Balancer 的服務限制](https://aka.ms/lblimits)。 如需標準 Load Balancer 詳細資料，請參閱[概觀](load-balancer-standard-overview.md)、[定價](https://aka.ms/lbpricing)及 [SLA](https://aka.ms/lbsla)。

## <a name="concepts"></a>概念

### <a name = "publicloadbalancer"></a>公用 Load Balancer

公用 Load Balancer 會將連入流量的公用 IP 位址和連接埠號碼，對應至 VM 的私人 IP 位址和連接埠號碼，來自 VM 的回應流量也是如此。 藉由套用負載平衡規則，即可將特定類型的流量分散至多個 VM 或服務。 例如，您可以將 Web 要求的流量負載分散在多個 Web 伺服器。

下圖顯示在三部 VM 中共用，且公用和私人 TCP 通訊埠均為 80 的 Web 流量負載平衡端點。 這三部 VM 均位在負載平衡集中。

![公用 Load Balancer 範例](./media/load-balancer-overview/IC727496.png)

圖：使用公用 Load Balancer 來進行網路流量負載平衡

當網際網路用戶端在 TCP 通訊埠 80 上傳送網頁要求至 Web 應用程式的公用 IP 位址時，Azure Load Balancer 會在負載平衡集中，將要求分配至這三部 VM。 如需 Load Balancer 演算法的詳細資訊，請參閱本文的 [Load Balancer 功能](load-balancer-overview.md##fundamental-load-balancer-features)一節。

根據預設，Azure Load Balancer 會在多個 VM 執行個體之間均分網路流量。 您也可以設定工作階段親和性。 如需詳細資訊，請參閱 [Load Balancer 分送模式](load-balancer-distribution-mode.md)。

### <a name = "internalloadbalancer"></a>內部 Load Balancer

內部 Load Balancer 只會將流量導向位於虛擬網路內的資源，或導向會使用 VPN 來存取 Azure 基礎結構的資源。 在這一方面，內部 Load Balancer 與公用 Load Balancer 不同。 Azure 基礎結構會限制存取虛擬網路的負載平衡前端 IP 位址。 前端 IP 位址與虛擬網路永不會直接公開至網際網路端點。 內部企業營運應用程式會在 Azure 中執行，並且可從 Azure 內或內部部署資源內存取。

內部 Load Balancer 可提供下列幾種類型的負載平衡：

* **虛擬網路內**：從虛擬網路中的 VM 至一組位於相同虛擬網路內之 VM 的負載平衡。
* **在跨單位虛擬網路中**：從內部部署電腦至一組位於相同虛擬網路內之 VM 的負載平衡。 
* **多層應用程式**：網際網路對向多層式應用程式 (其中的後端層並非網際網路對向) 的負載平衡。 後端層需要來自網際網路對向層的流量負載平衡 (請見下一張圖)。
* **針對企業營運應用程式**：在 Azure 中代管的企業營運應用程式的負載平衡，而不額外負載平衡器硬體或軟體。 此情節包括流量已負載平衡之電腦集合中的內部部署伺服器。

![內部 Load Balancer 範例](./media/load-balancer-overview/IC744147.png)

圖：使用公用和內部 Load Balancer 來進行多層式應用程式負載平衡

## <a name="pricing"></a>價格
標準 Load Balancer 使用量的收費是根據設定的負載平衡規則數量，以及處理輸入和輸出的資料量而定。 如需標準 Load Balancer 的定價資訊，請瀏覽 [Load Balancer 定價](https://azure.microsoft.com/pricing/details/load-balancer/)頁面。

基本 Load Balancer 則是免費提供。

## <a name="sla"></a>SLA

如需標準 Load Balancer SLA 的相關資訊，請瀏覽 [Load Balancer SLA](https://aka.ms/lbsla) 網頁。 

## <a name="limitations"></a>限制

- Load Balancer 是一款 TCP 或 UDP 產品，用於針對特定的 IP 通訊協定，進行負載平衡和連接埠轉送作業。  負載平衡規則和 NAT 傳入規則均支援 TCP 和 UDP ，但不支援包含 ICMP 在內的其他 IP 通訊協定。 Load Balancer 並不會終止、回應或與 UDP 或 TCP 流程的承載互動。 Load Balancer 並非 Proxy。 前端連線能力必須在與負載平衡或是 NAT 傳入規則 (TCP 或 UDP) 所使用的相同通訊協定中，成功進行頻內驗證，「而且」至少要有一個虛擬機器必須對用戶端產生回應，以查看來自前端的回應。  未從 Load Balancer 前端接收到頻內回應，即表示沒有任何虛擬機器能夠回應。  在虛擬機器未回應的情況下，無法和 Load Balancer 前端互動。  這也適用於傳出連線，其中[連接埠偽裝 SNAT](load-balancer-outbound-connections.md#snat) 僅支援 TCP 和 UDP，包括 ICMP 在內的任何其他 IP 通訊協定也會失敗。  指派執行個體層級的公用 IP 可減輕負擔。
- 公用 Load Balancer 從虛擬網路內的私人 IP 位址轉換至公用 IP 位址時會提供[傳出連線](load-balancer-outbound-connections.md)，但是內部 Load Balancers 與公用 Load Balancer 不同，不會將傳出的起源連線轉譯至內部 Load Balancer，因為兩者都位於私人 IP 位址空間內。  如此可避免在無需轉譯的專屬內部 IP 位址空間中將 SNAT 連接埠耗盡。  副作用是，如果後端集區虛擬機器的傳出流程嘗試將流程傳送到其所在集區之內部 Load Balancer 前端，「而且」對應回本身，則這兩個流程互不相符，而且流程會失敗。  如果流程未對應回位於後端集區中，且建立了到前端流程的相同虛擬機器，則流程將會成功。   當流程對應回本身時，傳出流程會看來像是從虛擬機器傳至前端，而對應的傳入流程會看來像是從虛擬機器傳至本身。 以客體作業系統來看，相同流程的傳入和傳出部分在虛擬機器內部不相符。 由於來源和目的地不同，TCP 堆疊無法將其中半數的流程視為相同流程的一部分。  當流程對應至後端集區中的任何其他虛擬機器時，半數流程將會相符，而虛擬機器就能成功回應流程。  此案例的徵兆是當流量回到其源自的相同後端時，發生間歇性連線逾時。 若要將來自後端集區的流程導向各自內部 Load Balancer 前端的後端集區，有數個常見的因應措施能夠可靠地實現這種情況，包括在內部 Load Balancer 後插入 Proxy 層或[使用 DSR 樣式規則](load-balancer-multivip-overview.md)。  客戶可以結合內部 Load Balancer 與任何第 3 方 Proxy，或用內部[應用程式閘道](../application-gateway/application-gateway-introduction.md)代替受限於 HTTP/HTTPS 的 Proxy 案例。 儘管可以使用公用 Load Balancer 來減輕負擔，但產生的情節容易造成 [SNAT 耗盡](load-balancer-outbound-connections.md#snat)，除非謹慎控管，否則應該避免。

## <a name="next-steps"></a>後續步驟

您現在已概略了解 Azure Load Balancer。 若要開始使用 Load Balancer，請先建立 Load Balancer，再建立安裝好自訂 IIS 擴充功能的 VM，並對 VM 之間的 Web 應用程式進行負載平衡。 若要了解如何進行，請參閱[建立基本 Load Balancer](quickstart-create-basic-load-balancer-portal.md)的快速入門。
