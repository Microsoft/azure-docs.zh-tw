---
title: Azure 標準 Load Balancer 概觀 | Microsoft Docs
description: Azure 標準 Load Balancer 功能概觀
services: load-balancer
documentationcenter: na
author: KumudD
manager: timlt
editor: ''
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: kumud
ms.openlocfilehash: 17b4bc68b2dc996134626b1822cfd17f0a9a7572
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161636"
---
# <a name="azure-load-balancer-standard-overview"></a>Azure Load Balancer Standard 概觀

Azure Load Balancer 可讓您調整您的應用程式，並為您的服務建立高可用性。 對於所有的 TCP 和 UDP 應用程式，Load Balancer 皆可用於輸入和輸出案例，並可提供低延遲和高輸送量，且最多可相應增加為數百萬個流程。 

本文將著重於標準 Load Balancer。  如需 Azure Load Balancer 的一般概觀，請另行檢閱[負載平衡器概觀](load-balancer-overview.md)。

## <a name="what-is-standard-load-balancer"></a>什麼是標準 Load Balancer？

標準 Load Balancer 是適用於所有 TCP 和 UDP 應用程式的新款 Load Balancer 產品，具有比基本 Load Balancer 更豐富、更細密的功能。  雖然兩者有許多相似之處，但請務必熟悉本文所說明的差異。

您可以使用標準 Load Balancer 作為公用或內部 Load Balancer。 此外，虛擬機器可以連接至一個公用和一個內部 Load Balancer 資源。

Load Balancer 資源的運作方式，一律以前端、規則、健康情況探查和後端集區定義來表示。  一個資源可包含多項規則。 您可以從虛擬機器的 NIC 資源指定後端集區，以將虛擬機器放入後端集區中。  使用虛擬機器擴展集時，此參數會透過網路設定檔傳遞並進行擴充。

資源的虛擬網路範圍，是重要層面之一。  基本 Load Balancer 存在於可用性設定組的範圍內，而標準 Load Balancer 則與虛擬網路範圍完全整合，並且適用所有的虛擬網路概念。

Load Balancer 資源是一種物件，而您可以在其中表示 Azure 應如何對其多租用戶基礎結構進行設計程式，以達到您想要建立的案例。  Load Balancer 資源與實際的基礎結構之間沒有直接關聯性；建立 Load Balancer 並不會建立執行個體，容量一直都在，且無須考量啟動或調整延遲的問題。 

>[!NOTE]
> Azure 可為您的案例提供一套受到完整管理的負載平衡解決方案。  如果您要尋找 TLS 終止 (「SSL 卸載」) 或每一 HTTP/HTTPS 要求的應用程式層處理，請檢閱[應用程式閘道](../application-gateway/application-gateway-introduction.md)。  如果您要尋找全域 DNS 負載平衡，請檢閱[流量管理員](../traffic-manager/traffic-manager-overview.md)。  視需要結合這些解決方案，您的端對端案例可能因而受益。

## <a name="why-use-standard-load-balancer"></a>為何要使用標準 Load Balancer？

標準負載平衡器可讓您將小規模部署的應用程式調整為大型且複雜的多重區域架構，並建立高可用性。

請檢閱下表，大致了解標準 Load Balancer 與基本 Load Balancer 之間的差異：

>[!NOTE]
> 新的設計應該使用 Standard Load Balancer。 

[!INCLUDE [comparison table](../../includes/load-balancer-comparison-table.md)]

請檢閱 [Load Balancer 的服務限制](https://aka.ms/lblimits)、[定價](https://aka.ms/lbpricing)和 [SLA](https://aka.ms/lbsla)。


### <a name="backend"></a>後端集區

標準 Load Balancer 後端集區可擴充至虛擬網路中的任何虛擬機器資源。  它最多可包含 1000 個後端執行個體。  後端執行個體是 IP 組態，也就是 NIC 資源的屬性。

後端集區可包含獨立虛擬機器、可用性設定組或虛擬機器擴展集。  您也可以混合搭配後端集區中的資源。 每個負載平衡器的後端集區可結合最多 150 項資源。

在考量如何設計您的後端集區時，您可以將最少量的個別後端集區資源作為設計原則，以進一步最佳化執行管理作業所需的時間。  在資料平面的效能或規模方面沒有任何差異。

### <a name="probes"></a>健康情況探查
  
Standard Load Balancer 會新增對 [HTTPS 健康情況探查](load-balancer-custom-probe-overview.md#httpprobe) (使用傳輸層安全性 (TLS) 包裝函式的 HTTP 探查) 的支援，以便精確地監視 HTTPS 應用程式。  

此外，當整個後端集區[向下探查](load-balancer-custom-probe-overview.md#probedown)時，Standard Load Balancer 允許所有已建立的 TCP 連線繼續。 (Basic Load Balancer 會終止對所有執行個體的現有 TCP 連線)。

檢閱[負載平衡器健康情況探查](load-balancer-custom-probe-overview.md)中的詳細資料。

### <a name="az"></a>可用性區域

標準 Load Balancer 在可使用「可用性區域」的區域中可支援較多功能。  這些功能會累加於所有標準 Standard Load Balancer 所提供的功能之上。  公用和內部標準 Load Balancer 皆可進行可用性區域設定。

區域性前端在部署於具有可用性區域的區域時，會依預設成為區域備援前端。   區域備援前端不受區域失敗的影響，且所有區域中的專用基礎結構會同時為其提供服務。 

此外，您可以確保前端可用於特定區域。 區域性前端與個別的區域會有共同的結果，且只有單一區域中的專用基礎結構會為其提供服務。

跨區域負載平衡適用於後端集區，Vnet 中的任何虛擬機器資源都可以是後端集區的一部分。

請檢閱[可用性區域相關功能的詳細討論](load-balancer-standard-availability-zones.md)。

### <a name="diagnostics"></a> 診斷

標準 Load Balancer 可透過 Azure 監視器提供多維度計量。  您可針對指定維度篩選、分組及劃分這些計量，  且您可深入檢視服務目前和過去的效能和健康情況。  此外也支援資源健康情況。  以下將簡單說明支援的診斷：

| 計量 | 說明 |
| --- | --- |
| VIP 可用性 | Load Balancer Standard 會持續運用從區域內到 Load Balancer 前端再一直到支援您 VM 之 SDN 堆疊的資料路徑。 只要狀況良好的執行個體持續存在，測量就會依循與您應用程式的負載平衡流量相同的路徑。 此外，也會驗證您客戶所使用的資料路徑。 此測量對您的應用程式來說是看不見的，也不會干擾到其他作業。|
| DIP 可用性 | Load Balancer Standard 使用分散式健康情況探查服務，可根據您的組態設定監視應用程式端點的健康情況。 這個計量會提供 Load Balancer 集區中每個個別執行個體端點的彙總檢視，或各端點篩選過的檢視。  您可以看到 Load Balancer 藉由健康情況探查設定如何檢視應用程式的健康情況。
| SYN 封包 | Load Balancer Standard 不會終止 TCP 連線，或是與 TCP 或 UDP 封包流程互動。 流程及其交握一律是在來源與 VM 執行個體之間進行。 若要針對您的 TCP 通訊協定案例進行進一步疑難排解，您可使用 SYN 封包計數器來了解已進行多少次 TCP 連線嘗試。 此計量會回報已收到的 TCP SYN 封包數。|
| SNAT 連線 | Load Balancer Standard 會回報偽裝為「公用 IP」位址前端的連出流程數目。 SNAT 連接埠是可耗盡的資源。 此計量可以指出應用程式有多依賴 SNAT 來處理連出的起始流程。  系統會回報成功和失敗之連出 SNAT 流程的計數器，而且可用來對連出流程的健康情況進行疑難排解及了解。|
| 位元組計數器 | Load Balancer Standard 會回報每個前端處理的資料。|
| 封包計數器 | Load Balancer Standard 會回報每個前端處理的封包。|

請檢閱[標準 Load Balancer 診斷的詳細討論](load-balancer-standard-diagnostics.md)。

### <a name="haports"></a>HA 連接埠

標準 Load Balancer 支援新類型的規則。  

您可以設定負載平衡規則，讓應用程式延展並變得高度可靠。 當您使用 HA 連接埠負載平衡規則時，標準 Load Balancer 將會在內部標準 Load Balancer 之前端 IP 位址的每個暫時連接埠上提供以流量為準的負載平衡。  對於那些指定個別連接埠並不實際或不符合需求的其他案例來說，此功能相當實用。

HA 連接埠負載平衡規則可讓您為網路虛擬設備和任何需要大範圍輸入連接埠的應用程式，建立主動-被動或主動-主動 n + 1 的案例。  健康情況探查可用來判斷哪個後端應接收新的流量。  您可以使用網路安全性群組來模擬連接埠範圍案例。

>[!IMPORTANT]
> 如果您打算使用網路虛擬設備，請要求廠商提供相關指引以了解是否其產品是否經過 HA 連接埠的測試，並遵循其指引進行實作。 

請檢閱 [HA 連接埠的詳細討論](load-balancer-ha-ports-overview.md)。

### <a name="securebydefault"></a>預設保護

標準 Load Balancer 完全架構於虛擬網路上。  虛擬網路是封閉的私人網路。  由於標準 Load Balancer 和標準公用 IP 位址設計為允許從虛擬網路外部存取此虛擬網路，因此這些資源目前預設為關閉，除非您加以開啟。 這表示目前會使用網路安全性群組 (NSG) 來明確許可和放行允許的流量。  您可以在建立整個虛擬資料中心之後，透過 NSG 決定該中心可供使用的項目和時間。  如果您沒有子網路的 NSG 或虛擬機器資源的 NIC，系統不會允許流量存取此資源。

若要深入了解 NSG 及如何將其套用至您的案例，請參閱[網路安全性群組](../virtual-network/security-overview.md)。

### <a name="outbound"></a> 輸出連線

Load Balancer 支援輸入和輸出案例。  標準 Load Balancer 在輸出連線方面與基本 Load Balancer 有很大的不同。

系統會使用來源網路位址轉譯 (SNAT)，將您虛擬網路上的內部、私人 IP 位址對應至 Load Balancer 前端的公用 IP 位址。

標準 Load Balancer 基於[更為穩健、可擴充且可預期的 SNAT 演算法](load-balancer-outbound-connections.md#snat)的目標導入了新的演算法，並且啟用了新功能、去除模糊性，以及強制執行不產生負面影響的明確設定。 必須要有這些變更，才會有新的功能。 

使用標準 Load Balancer 時須注意的重要原則如下：

- Load Balancer 資源的運用端賴於完整的規則。  Azure 的所有程式設計皆衍生自該規則的設定。
- 有多個可用的前端時，將會使用所有前端，而每個前端分別會使可用的 SNAT 連接埠數目隨之增加
- 您可以選擇及控制是否不要讓特定前端用於輸出連線。
- 輸出案例是明確的，且必須經由指定才會有輸出連線存在。
- 負載平衡規則會推斷 SNAT 的程式設計方式。 負載平衡規則會隨通訊協定而不同。 SNAT 會隨通訊協定而不同，且設定應反映這一點，而非產生負面影響。

#### <a name="multiple-frontends"></a>多個前端
如果您因為預期會有或已面臨大量輸出連線需求而需要更多 SNAT 連接埠，您也可以藉由對相同的虛擬機器資源設定額外的前端、規則和後端集區，來加入增量的 SNAT 連接埠數量。

#### <a name="control-which-frontend-is-used-for-outbound"></a>控制用於輸出的前端
如果您想要將輸出連線限定為僅來自特定的前端 IP 位址，您可以選擇性地在表示輸出對應的規則中停用輸出 SNAT。

#### <a name="control-outbound-connectivity"></a>控制輸出連線
標準 Load Balancer 存在於虛擬網路的內容中。  虛擬網路是隔離的私人網路。  若未與公用 IP 位址有所關聯，就不會允許公用連線。  您可以存取 [VNet 服務端點](../virtual-network/virtual-network-service-endpoints-overview.md)，因為它們位於您虛擬網路內部的本機位置。  如果您想要對您虛擬網路外部的目的地建立輸出連線，您有兩個選項：
- 將標準 SKU 公用 IP 位址作為執行個體層級的公用 IP 位址，指派給虛擬機器資源，或
- 將虛擬機器資源放在公用標準 Load Balancer 的後端集區中。

這兩種方式都允許從虛擬網路到虛擬網路外部的輸出連線。 

如果您_只有_內部標準 Load Balancer 與您虛擬機器資源所在的後端集區相關聯，則您的虛擬機器將只能存取虛擬網路資源和 [VNet 服務端點](../virtual-network/virtual-network-service-endpoints-overview.md)。  您可以依照上一段說明的步驟來建立輸出連線。

與標準 SKU 無關聯之虛擬機器資源的輸出連線會維持原狀。

請檢閱[輸出連線的詳細討論](load-balancer-outbound-connections.md)。

### <a name="multife"></a>多個前端
Load Balancer 支援為多個前端使用多項規則。  標準 Load Balancer 可將此支援擴及至輸出案例。  輸出案例基本上就是與輸入負載平衡規則相反的情形。  輸入負載平衡規則也會建立輸出連線的關聯性。 標準 Load Balancer 會透過負載平衡規則使用所有與虛擬機器資源相關聯的前端。  此外，您可以透過負載平衡規則的參數來抑制輸出連線用途的負載平衡規則，而使特定前端的可用選項變成空的。

相對地，基本 Load Balancer 會隨機選取單一前端，且無法控制能夠選取哪一個。

請檢閱[輸出連線的詳細討論](load-balancer-outbound-connections.md)。

### <a name="operations"></a>管理作業

標準 Load Balancer 資源存在於全新的基礎結構平台上。  這可大幅提升標準 SKU 的管理作業執行速度，且每項標準 SKU 資源的管理通常都可在 30 秒內完成。  請注意，當後端集區的大小增加，變更後端集區的所需時間也會隨之增加。

您修改標準 Load Balancer 資源，以及在不同虛擬機器之間移動標準公用 IP 位址的速度，都可快上許多。

## <a name="migration-between-skus"></a>在 SKU 之間移轉

SKU 是不可變動的。 請依照本節中的步驟從一個資源 SKU 移到另一個 SKU。

>[!IMPORTANT]
>請完整檢閱這份文件以了解 SKU 之間的差異，並小心檢查您的案例。  您可能需要根據您的案例進行額外的變更。

### <a name="migrate-from-basic-to-standard-sku"></a>從 Basic 移轉到 Standard SKU

1. 視需要針對 Load Balancer 和「公用 IP」建立一個新的 Standard 資源。 重新建立您的規則和探查定義。  如果您先前使用對 443/tcp 的 TCP 探查，請考慮將此探查通訊協定變更為 HTTPS 探查並新增路徑。

2. 在 NIC 或子網路上建立新的或更新現有的 NSG，以將已負載平衡的流量、探查及任何其他您想要允許的流量列入允許清單。

3. 從所有 VM 執行個體中移除 Basic SKU 資源 (看 Load Balancer 和「公用 IP」哪一個適用)。 請務必也移除可用性設定組的所有 VM 執行個體。

4. 將所有 VM 執行個體附加至新的 Standard SKU 資源。

### <a name="migrate-from-standard-to-basic-sku"></a>從 Standard 移轉到 Basic SKU

1. 視需要針對 Load Balancer 和「公用 IP」建立一個新的 Basic 資源。 重新建立您的規則和探查定義。  將 HTTPS 探查變更為對 443/tcp 的 TCP 探查。 

2. 從所有 VM 執行個體中移除 Standard SKU 資源 (看 Load Balancer 和「公用 IP」哪一個適用)。 請務必也移除可用性設定組的所有 VM 執行個體。

3. 將所有 VM 執行個體附加至新的 Basic SKU 資源。

>[!IMPORTANT]
>
>在 Basic 和 Standard SKU 的使用上有一些限制。
>
>只有在 Standard SKU 中才有提供「HA 連接埠」和 Standard SKU 的「診斷」。 您無法既從 Standard SKU 移轉到 Basic SKU 又同時保留這些功能。
>
>如本文所述，基本和標準 SKU 之間有許多差異。  請確實加以了解，並做好相關準備。
>
>Load Balancer 和 Public IP 資源必須使用相符的 SKU。 您無法將 Basic SKU 資源與 Standard SKU 資源混用。 您無法將獨立虛擬機器、可用性設定組資源中的虛擬機器或虛擬機器擴展集資源同時連結到這兩個 SKU。

## <a name="region-availability"></a>區域可用性

Load Balancer Standard 目前已在所有公用雲端地區推出。

## <a name="sla"></a>SLA

標準 Load Balancer 提供 99.99% SLA。  如需詳細資訊，請檢閱[標準 Load Balancer SLA](https://aka.ms/lbsla)。

## <a name="pricing"></a>價格

標準 Load Balancer 為計費產品，根據所設定的負載平衡規則數目以及處理的所有輸入和輸出資料計算費用。 如需標準 Load Balancer 的定價資訊，請瀏覽 [Load Balancer 定價](https://aka.ms/lbpricing)頁面。

## <a name="limitations"></a>限制

- SKU 是不可變動的。 您無法變更現有資源的 SKU。
- 獨立虛擬機器資源、可用性設定組資源或虛擬機器擴展集資源都只能參考一個 SKU，絕不會同時參考兩者。
- Load Balancer 規則無法跨越兩個虛擬網路。  前端和其相關的後端執行個體必須位於相同的虛擬網路。  
- 無法跨全域虛擬網路對等互連存取 Load Balancer 前端。
- 標準 SKU LB 與 PIP 資源不支援[移動訂用帳戶作業](../azure-resource-manager/resource-group-move-resources.md)。
- 由於預先 VNet 服務和其他平台服務的運作方式產生副作用，而只使用內部標準 Load Balancer 時，才可存取沒有 VNet 和其他 Microsoft 平台服務的 Web 背景工作角色。 請勿以此作為個別服務本身，否則基礎平台可能會在不經通知的情況下變更。 如果在只使用內部標準 Load Balancer 時有需要，請一律假設您需要明確建立[輸出連線](load-balancer-outbound-connections.md)。
- Load Balancer 是一款 TCP 或 UDP 產品，用於針對特定的 IP 通訊協定，進行負載平衡和連接埠轉送作業。  負載平衡規則和 NAT 傳入規則均支援 TCP 和 UDP ，但不支援包含 ICMP 在內的其他 IP 通訊協定。 Load Balancer 並不會終止、回應或與 UDP 或 TCP 流程的承載互動。 Load Balancer 並非 Proxy。 前端連線能力必須在與負載平衡或是 NAT 傳入規則 (TCP 或 UDP) 所使用的相同通訊協定中，成功進行頻內驗證，而且至少要有一個虛擬機器必須對用戶端產生回應，以查看來自前端的回應。  未從 Load Balancer 前端接收到頻內回應，即表示沒有任何虛擬機器能夠回應。  在虛擬機器未回應的情況下，無法和 Load Balancer 前端互動。  這也適用於傳出連線，其中[連接埠偽裝 SNAT](load-balancer-outbound-connections.md#snat) 僅支援 TCP 和 UDP，包括 ICMP 在內的任何其他 IP 通訊協定也會失敗。  指派執行個體層級的公用 IP 可減輕負擔。
- 公用 Load Balancer 從虛擬網路內的私人 IP 位址轉換至公用 IP 位址時會提供[傳出連線](load-balancer-outbound-connections.md)，但是內部 Load Balancers 與公用 Load Balancer 不同，不會將傳出的起源連線轉譯至內部 Load Balancer，因為兩者都位於私人 IP 位址空間內。  如此可避免在無需轉譯的專屬內部 IP 位址空間中將 SNAT 耗盡。  副作用是，如果後端集區 VM 的傳出流程嘗試將流程傳送到其所在集區之內部 Load Balancer 前端的，並且對應回本身，則這兩個流程互不相符，而且流程會失敗。  如果流程未對應回建立流程至前端之後端集區中的相同 VM，則流程將會成功。   當流程對應回本身時，傳出流程似乎是來自 VM 而傳至前端，而對應的傳入流程似乎來自 VM 至而傳至本身。 以客體作業系統來看，相同流程的傳入和傳出部分在虛擬機器內部不相符。 由於來源和目的地不同，TCP 堆疊無法將其中半數的流程視為相同流程的一部分。  當流程對應至後端集區中的任何其他 VM 時，半數流程將會相符，而 VM 就能成功回應流程。  此案例的症狀是間歇性出現連線逾時。 有數個常見的因應措施能夠可靠地實現此情節 (將來自後端集區的流程產生至個別內部 Load Balancer 前端的後端集區)，包括在內部 Load Balancer 後插入協力廠商 Proxy 或[使用 DSR 樣式規則](load-balancer-multivip-overview.md)。  儘管可以使用公用 Load Balancer 來減輕負擔，但產生的情節容易造成 [SNAT 耗盡](load-balancer-outbound-connections.md#snat)，除非謹慎控管，否則應該避免。

## <a name="next-steps"></a>後續步驟

- 了解如何使用[標準 Load Balancer 和可用性區域](load-balancer-standard-availability-zones.md)。
- 深入了解[健康情況探查](load-balancer-custom-probe-overview.md)。
- 深入了解[可用性區域](../availability-zones/az-overview.md)。
- 了解[標準 Load Balancer 診斷](load-balancer-standard-diagnostics.md)。
- 參閱[支援的多維度計量](../monitoring-and-diagnostics/monitoring-supported-metrics.md#microsoftnetworkloadbalancers)，以了解 [Azure 監視器](../monitoring-and-diagnostics/monitoring-overview.md)中的診斷。
- 了解如何使用 [Load Balancer 來進行輸出連線](load-balancer-outbound-connections.md)。
- 深入了解[輸出規則](load-balancer-outbound-rules-overview.md)。
- 深入了解[閒置時重設 TCP](load-balancer-tcp-reset.md)。
- 了解[具有 HA 連接埠負載平衡規則的標準 Load Balancer](load-balancer-ha-ports-overview.md)。
- 了解如何搭配使用 [Load Balancer 與多個前端](load-balancer-multivip-overview.md)。
- 了解[虛擬網路](../virtual-network/virtual-networks-overview.md)。
- 深入了解[網路安全性群組](../virtual-network/security-overview.md)。
- 了解 [VNET 服務端點](../virtual-network/virtual-network-service-endpoints-overview.md)。
- 了解 Azure 中的一些其他重要[網路功能](../networking/networking-overview.md)。
- 深入了解 [Load Balancer](load-balancer-overview.md)。
