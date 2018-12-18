---
title: Azure 中的輸出連線 (傳統) | Microsoft Docs
description: 本文說明 Azure 如何讓雲端服務與公用網際網路服務進行通訊。
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/13/2018
ms.author: kumud
ms.openlocfilehash: bd446923f84d22537b7a49a8ef6124f343141d73
ms.sourcegitcommit: 0b05bdeb22a06c91823bd1933ac65b2e0c2d6553
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39069905"
---
# <a name="outbound-connections-classic"></a>輸出連線 (傳統)

Azure 透過數個不同的機制，提供客戶部署的輸出連線。 本文說明案例為何、套用時機、運作方式，以及如何管理它們。

>[!NOTE]
>本文僅涵蓋傳統部署。  檢閱[輸出連線](load-balancer-outbound-connections.md)以取得 Azure 中的所有 Resource Manager 部署案例。

Azure 中的部署可以與公用 IP 位址空間中的 Azure 外部端點進行通訊。 當執行個體向公用 IP 位址空間中的目的地起始輸出流程時，Azure 會動態地將私人 IP 位址對應至公用 IP 位址。 建立此對應之後，此輸出起源流程的傳回流量便也可以觸達流程起源的私人 IP 位址。

Azure 會使用來源網路位址轉譯 (SNAT) 執行這項功能。 當多個私人 IP 位址在單一公用 IP 位址後面偽裝時，Azure 會使用[連接埠位址轉譯 (PAT)](#pat) 來偽裝私人 IP 位址。 暫時連接埠會用於 PAT，並且根據集區大小[預先配置](#preallocatedports)。

有多個[輸出案例](#scenarios)。 您可以視需要合併這些案例。 仔細檢閱它們以了解當它們套用到您的部署模型和應用程式案例時的功能、限制和模式。 檢閱[管理這些案例](#snatexhaust)的指引。

## <a name="scenarios"></a>案例概觀

Azure 提供三種不同的方法來達成輸出連線傳統部署。  並非所有的傳統部署都具備這三個案例：

| 案例 | 方法 | IP 通訊協定 | 說明 | Web 背景工作角色 | IaaS | 
| --- | --- | --- | --- | --- | --- |
| [1.VM 具有執行個體層級的公用 IP 位址](#ilpip) | SNAT，未使用連接埠偽裝 | TCP、UDP、ICMP、ESP | Azure 會使用指派了公用 IP 的虛擬機器。 執行個體有所有可用的暫時連接埠。 | 否 | 是 |
| [2. 負載平衡的公用端點](#publiclbendpoint) | SNAT 與連接埠偽裝 (PAT)，連至公用端點 | TCP、UDP | Azure 會與多個私人端點共用公用 IP 位址的公用端點。 Azure 使用公用端點的暫時連接埠來進行 PAT。 | 是 | 是 |
| [3.獨立 VM](#defaultsnat) | SNAT 與連接埠偽裝 (PAT) | TCP、UDP | Azure 會自動指定 SNAT 的公用 IP 位址，與整個部署共用此公用 IP 位址，並使用此公用端點 IP 位址的暫時連接埠來進行 PAT。 這是上述案例的後援案例。 如果您需要可見性和控制權，則不建議使用此方式。 | 是 | 是 |

這是 Azure 中 Resource Manager 部署可使用之輸出連線功能的子集。  

傳統中的不同部署具有不同功能：

| 傳統部署 | 提供的功能 | 
| --- | --- |
| 虛擬機器 | 案例 [1](#ilpip)、[2](#publiclbendpoint) 或 [3](#defaultsnat) |
| Web 背景工作角色 | 只有案例 [2](#publiclbendpoint)、[3](#defaultsnat) | 

[風險降低策略](#snatexhaust)也有相同的差異。

用於傳統部署之 PAT 的[預先配置暫時連接埠](#ephemeralports)的演算法，與用於 Azure Resource Manager 資源部署的演算法相同。

### <a name="ilpip"></a>案例 1：VM 與執行個體層級公用 IP 位址

在此案例中，VM 具有指派給它的執行個體層級公用 IP (ILPIP)。 就輸出連線而言，VM 是否有負載平衡的端點並不重要。 此案例的優先順序高於其他案例。 使用 ILPIP 時，VM 會針對所有輸出流程使用 ILPIP。  

指派給虛擬機器的公用 IP 是 1:1 關聯 (而非 1:多) 而且會實作為無狀態 1:1 NAT。  不使用連接埠偽裝 (PAT)，VM 會有所有可用的暫時連接埠。

如果您的應用程式起始許多輸出流程而發生 SNAT 連接埠耗盡的情況，請考慮指派 [ILPIP 來緩和 SNAT 限制的問題](#assignilpip)。 從整體面檢閱[管理 SNAT 耗盡](#snatexhaust)。

### <a name="publiclbendpoint"></a>案例 2：負載平衡的公用端點

在此案例中，VM 或 Web 背景工作角色是透過負載平衡的端點與公用 IP 位址相關聯。 VM 沒有指派給它的公用 IP 位址。 

當負載平衡的 VM 建立輸出流程時，Azure 會將輸出流量的私人來源 IP 位址轉譯為負載平衡公用端點的公用 IP 位址。 Azure 會使用 SNAT 來執行此功能。 Azure 也會使用 [PAT](#pat) 來偽裝公用 IP 位址背後的多個私人 IP 位址。 

負載平衡器公用 IP 位址前端的暫時連接埠可用來區分 VM 所產生的個別流程。 建立輸出流程時，SNAT 會動態使用[預先配置暫時連接埠](#preallocatedports)。 在此情況下，用於 SNAT 的暫時連接埠稱為 SNAT 連接埠。

SNAT 連接埠會預先配置，如[了解 SNAT 和 PAT](#snat) 一節所述。 它們是可能耗盡的有限資源。 請務必了解[取用](#pat)它們的方式。 若要了解如何針對此取用方式進行設計及視需要降低風險，請檢閱[管理 SNAT 耗盡](#snatexhaust)。

當存在[多個負載平衡的公用端點](load-balancer-multivip.md)時，這些公用 IP 位址都是[輸出流程的候選項](#multivipsnat)，會隨機選取其中一個。  

### <a name="defaultsnat"></a>案例 3：沒有相關聯的公用 IP 位址

在此案例中，VM 或 Web 背景工作角色不屬於負載平衡的公用端點。  對 VM 來說，系統沒有指派任何 ILPIP 位址給它。 當 VM 建立輸出流程時，Azure 會將輸出流量的公用來源 IP 位址轉譯為私用來源 IP 位址。 此輸出流量所用的公用 IP 位址無法進行設定，而且不利於訂用帳戶的公用 IP 資源限制。  Azure 會自動配置此位址。

Azure 會使用 SNAT 搭配位址偽裝 ([PAT](#pat)) 來執行此功能。 這個案例類似於[案例 2](#lb)，差別在於沒有所使用 IP 位址的控制權。 這是案例 1 和 2 不存在時的後援案例。 如果您想要能夠控制輸出位址，則不建議採用此案例。 如果輸出連線對您的應用程式很重要，您應該選擇其他案例。

SNAT 連接埠會預先配置，如[了解 SNAT 和 PAT](#snat) 一節所述。  共用公用 IP 位址之 VM 或 Web 背景工作角色的數目，會決定預先配置的臨時連接埠數目。   請務必了解[取用](#pat)它們的方式。 若要了解如何針對此取用方式進行設計及視需要降低風險，請檢閱[管理 SNAT 耗盡](#snatexhaust)。

## <a name="snat"></a>了解 SNAT 和 PAT

### <a name="pat"></a>連接埠偽裝 SNAT (PAT)

當部署建立輸出連線時，會重寫每個輸出連線的來源。 來源會由私人 IP 位址空間重寫為與部署相關聯的公用 IP (根據上述案例)。 在公用 IP 位址空間中，5 tuple 流程 (來源 IP 位址、來源連接埠、IP 傳輸通訊協定、目的地 IP 位址、目的地連接埠) 必須是唯一的。  

在重寫私人來源 IP 位址之後，會使用暫時連接埠 (SNAT 連接埠) 來達到這個目的，因為多個流程來自單一公用 IP 位址。 

每個流程都會取用一個 SNAT 連接埠至單一目的地 IP 位址、連接埠和通訊協定。 針對相同目的地 IP 位址、連接埠和通訊協定的多個流程，每個流程都會取用單一 SNAT 連接埠。 這可確保流程在從相同公用 IP 位址產生並前往相同目的地 IP 位址、連接埠及通訊協定時，會是唯一的。 

多個各自前往不同目的地 IP 位址、連接埠及通訊協定的流程會共用單一 SNAT 連接埠。 目的地 IP 位址、連接埠及通訊協定可讓流程成為唯一的，而無須使用額外的來源連接埠來區別公用 IP 位址空間中的流程。

當 SNAT 連接埠資源耗盡時，輸出流程會失敗，直到現有的流程釋出 SNAT 連接埠為止。 當流程關閉並使用 [4 分鐘閒置逾時](#idletimeout)來從閒置流程回收 SNAT 連接埠時，Load Balancer 會回收 SNAT 連接埠。

如需了解可緩和通常會導致 SNAT 連接埠耗盡情況的模式，請檢閱[管理 SNAT](#snatexhaust) 一節。

### <a name="preallocatedports"></a>連接埠偽裝 SNAT (PAT) 的暫時連接埠預先配置

Azure 會使用演算法在使用連接埠偽裝 SNAT ([PAT](#pat)) 時，根據後端集區的大小決定可用的預先配置 SNAT 連接埠數目。 SNAT 連接埠是可供特定公用 IP 來源位址使用的暫時連接埠。

部署執行個體時，Azure 會根據有多少個 VM 或 Web 背景工作角色執行個體共用指定的公用 IP 位址，預先配置 SNAT 連接埠。  建立輸出流程時，[PAT](#pat) 會動態取用 (最多可達預先配置的限制) 這些連接埠，並且在流程關閉或[閒置逾時](#ideltimeout)時釋出這些連接埠。

下表說明各個後端集區大小層級的 SNAT 連接埠預先配置：

| 執行個體 | 每個執行個體預先配置的 SNAT 連接埠 |
| --- | --- |
| 1-50 | 1,024 |
| 51-100 | 512 |
| 101-200 | 256 |
| 201-400 | 128 |

請記住，可用的 SNAT 連接埠數目不會直接轉譯成流程數目。 單一 SNAT 連接埠可以重複使用於多個唯一目的地。 只有在必須使用連接埠來讓流程具有唯一性的情況下，才會取用連接埠。 如需有關設計和降低風險措施的指引，請參閱[如何管理這個可耗盡的資源](#snatexhaust)，以及描述 [PAT](#pat) 的小節。

變更部署的大小可能會影響您已建立的部分流程。 如果後端集區大小增加並轉換到下一層，在轉換至下一個較大後端集區層的期間，會回收您預先配置的一半 SNAT 連接埠。 與已回收 SNAT 連接埠關聯的流程會逾時，而必須重新建立。 嘗試建立新流程時，只要預先配置的連接埠可用，流程就會立即成功。

如果部署大小縮減並轉換成較低的層級，可用的 SNAT 連接埠數目就會增加。 在此情況下，現有已配置 SNAT 連接埠及其各自流程都不會受到影響。

如果將雲端服務重新部署或變更，基礎結構可能會暫時將後端集區報告為實際的兩倍，而 Azure 則會在每個執行個體預先配置比預期較少的 SNAT 連接埠。  這可以暫時增加 SNAT 連接埠耗盡的機率。 集區的大小終究會轉換成實際大小，且 Azure 會根據上表將預先配置的 SNAT 連接埠自動增加至預期的數目。  這是設計上就會進行的行為，且不可設定。

SNAT 連接埠配置為 IP 傳輸通訊協定專屬 (TCP 和 UDP 會個別維護)，並且在下列條件之下釋出：

### <a name="tcp-snat-port-release"></a>TCP SNAT 連接埠釋出

- 如果伺服器/用戶端均傳送 FIN/ACK，則會在 240 秒之後釋出 SNAT 連接埠。
- 如果看到 RST，則會在 15 秒之後釋出 SNAT 連接埠。
- 已達到閒置逾時

### <a name="udp-snat-port-release"></a>UDP SNAT 連接埠釋出

- 已達到閒置逾時

## <a name="problemsolving"></a>解決問題 

本節的用意是協助您降低 Azure 中輸出連線可能會發生的 SNAT 耗盡與其他案例。

### <a name="snatexhaust"></a>管理 SNAT (PAT) 連接埠耗盡
用於 [PAT](#pat) 的[暫時連接埠](#preallocatedports)十分耗費資源，如[沒有相關聯的公用 IP](#defaultsnat) 和[負載平衡的公用端點](#publiclbendpoint)中所述。

如果您知道將會對相同的目的地 IP 位址和連接埠起始許多輸出 TCP 或 UDP 連線，並且觀察到失敗的輸出連線，或是支援人員告知您 SNAT 連接埠 ([PAT](#pat) 使用的預先配置[暫時連接埠](#preallocatedports)) 將耗盡，您有數個可緩和這些問題的一般選項。 請檢閱這些選項並判斷哪一個可用且最適合您的案例。 可能會有一或多個選項有助於管理此案例。

如果您在了解輸出連線行為方面遇到問題，您可以使用 IP 堆疊統計資料 (netstat)。 或是使用封包擷取來觀察連線行為，也會很有幫助。

#### <a name="connectionreuse"></a>將應用程式修改成重複使用連線 
您可以在應用程式中重複使用連線，以降低對用於 SNAT 之暫時連接埠的需求。 對於 HTTP/1.1 之類的通訊協定尤其如此，因為預設會重複使用連線。 而其他使用 HTTP 作為其傳輸方式的通訊協定 (例如 REST) 也會因而受益。 

重複使用一律比每個要求的獨立且不可部分完成的 TCP 連線來得好。 重複使用可提供效能更佳又非常有效率的 TCP 傳輸。

#### <a name="connection pooling"></a>將應用程式修改成使用連線共用
您可以在應用程式中採用連線集區配置，這樣要求會在內部分布到一固定的連線集合 (每個都盡可能地重複使用)。 這個配置會限制使用中的暫時連接埠數目，而建立較可預測的環境。 這個配置也可在單一連線阻斷某個作業的回應時，藉由允許多個作業同時進行，來增加要求輸送量。  

連線共用可能已存在於您用來開發應用程式的架構中，或是您應用程式的組態設定中。 您可以將連線共用與連線重複使用搭配使用。 這樣，您的多個要求就會將數目固定且可預測的連接埠取用至相同的目的地 IP 位址和連接埠。 這些要求也會因為系統有效率地使用 TCP 交易來降低延遲和資源使用量而受益。 UDP 交易也有幫助，因為管理 UDP 流程數目可接著避免發生耗盡狀況，並管理 SNAT 連接埠使用量。

#### <a name="retry logic"></a>將應用程式修改成使用較不積極的重試邏輯
當用於 [PAT](#pat) 的[預先配置暫時連接埠](#preallocatedports)耗盡或發生應用程式失敗時，不含衰減和降速邏輯的積極或暴力重試會造成耗盡的情況發生或持續存在。 您可以使用較不積極的重試邏輯，以降低對暫時連接埠的需求。 

暫時連接埠有 4 分鐘的閒置逾時 (無法調整)。 如果重試太過積極，耗盡情況就沒有機會進行自我清理。 因此，考慮應用程式重試交易的方式和頻率，是設計的一個重要部分。

#### <a name="assignilpip"></a>指派執行個體層級公用 IP 給每個 VM
指派 ILPIP 會將您的案例變更為 [VM 的執行個體層級公用 IP](#ilpip)。 用於每個 VM 的所有公用 IP 暫時連接埠都可供 VM 使用。 (與公用 IP 暫時連接埠會與個別部署之所有相關 VM 共用的案例相反。)有一些要考慮的取捨，例如將大量的個別 IP 位址加入白名單的潛在影響。

>[!NOTE] 
>此選項不適用於 Web 背景工作角色。

### <a name="idletimeout"></a>使用 Keepalive 來重設輸出閒置逾時

連出連線有 4 分鐘的閒置逾時。 此逾時無法調整。 不過，您可以使用傳輸 (例如 TCP Keepalive) 或應用程式層 Keepalive 來重新整理閒置流程，然後視需要重設此閒置逾時。  請諮詢封裝軟體的供應商，了解是否支援此功能，或如何啟用此功能。  通常只有一端需要產生 Keepalive 來重設閒置逾時。 

## <a name="discoveroutbound"></a>探索 VM 使用的公用 IP
有許多方式可判斷輸出連線的公用來源 IP 位址。 OpenDNS 提供可顯示您 VM 公用 IP 位址的服務。 

藉由使用 nslookup 命令，您便可以將名稱 myip.opendns.com 的 DNS 查詢傳送給 OpenDNS 解析程式。 服務會傳回用來傳送查詢的來源 IP 位址。 當您從 VM 執行下列查詢時，回應會是用於該 VM 的公用 IP：

    nslookup myip.opendns.com resolver1.opendns.com


## <a name="next-steps"></a>後續步驟

- 深入了解在 Resource Manager 部署中使用的[負載平衡器](load-balancer-overview.md)。
- 深入了解 Resource Manager 部署中的[輸出連線](load-balancer-outbound-connections.md)情節。
