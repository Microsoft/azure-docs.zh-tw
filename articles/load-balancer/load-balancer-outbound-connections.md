---
title: Azure 中的輸出連線 | Microsoft Docs
description: 本文說明如何 Azure 如何讓 VM 與公用網際網路服務進行通訊。
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
ms.assetid: 5f666f2a-3a63-405a-abcd-b2e34d40e001
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/27/2018
ms.author: kumud
ms.openlocfilehash: 24eec3b1f3c85384f80823b82962038c235b6dac
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47036985"
---
# <a name="outbound-connections-in-azure"></a>Azure 中的輸出連線

Azure 透過數個不同的機制，提供客戶部署的輸出連線。 本文說明案例為何、套用時機、運作方式，以及如何管理它們。

>[!NOTE] 
>本文僅涵蓋 Resource Manager 部署。 檢閱[輸出連線 (傳統)](load-balancer-outbound-connections-classic.md) 以取得 Azure 中的所有傳統部署案例。

Azure 中的部署可以與公用 IP 位址空間中的 Azure 外部端點進行通訊。 當執行個體向公用 IP 位址空間中的目的地起始輸出流程時，Azure 會動態地將私人 IP 位址對應至公用 IP 位址。 建立此對應之後，此輸出起源流程的傳回流量便也可以觸達流程起源的私人 IP 位址。

Azure 會使用來源網路位址轉譯 (SNAT) 執行這項功能。 當多個私人 IP 位址在單一公用 IP 位址後面偽裝時，Azure 會使用[連接埠位址轉譯 (PAT)](#pat) 來偽裝私人 IP 位址。 暫時連接埠會用於 PAT，並且根據集區大小[預先配置](#preallocatedports)。

有多個[輸出案例](#scenarios)。 您可以視需要合併這些案例。 仔細檢閱它們以了解當它們套用到您的部署模型和應用程式案例時的功能、限制和模式。 檢閱[管理這些案例](#snatexhaust)的指引。

>[!IMPORTANT] 
>Standard Load Balancer 引進新功能以及不同的輸出連線行為。   例如，當內部 Standard Load Balancer 存在時，就不會有[案例 3](#defaultsnat)，且必須採取不同步驟。   請仔細完整檢閱本文件以了解整體概念和 SKU 之間的差異。

## <a name="scenarios"></a>案例概觀

使用 [Azure Resource Manager](#arm) 時，會明確定義 Azure Load Balancer 和相關資源。  Azure 目前提供三個不同的方法，來達成 Azure Resource Manager 資源的輸出連線。 

| 案例 | 方法 | IP 通訊協定 | 說明 |
| --- | --- | --- | --- |
| [1.具有執行個體層級公用 IP 位址的 VM (無論是否有 Load Balancer)](#ilpip) | SNAT，未使用連接埠偽裝 | TCP、UDP、ICMP、ESP | Azure 會使用指派給執行個體 NIC 之 IP 設定的公用 IP。 執行個體有所有可用的暫時連接埠。 |
| [2.與 VM 關聯的公用 Load Balancer (執行個體上沒有執行個體層級公用 IP 位址)](#lb) | SNAT 搭配使用 Load Balancer 前端的連接埠偽裝 (PAT) | TCP、UDP |Azure 會與多個私人 IP 位址共用公用 Load Balancer 前端的公用 IP 位址。 Azure 會使用前端的暫時連接埠來進行 PAT。 |
| [3.獨立 VM (無 Load Balancer、無執行個體層級公用 IP 位址)](#defaultsnat) | SNAT 與連接埠偽裝 (PAT) | TCP、UDP | Azure 會自動指定 SNAT 的公用 IP 位址、與可用性設定組的多個私人 IP 位址共用此公用 IP 位址，以及使用此公用 IP 位址的暫時連接埠。 這是上述案例的後援案例。 如果您需要可見性和控制權，則不建議使用此方式。 |

如果您不想要讓 VM 與公用 IP 位址空間中的 Azure 外部端點進行通訊，您可以使用網路安全性群組 (NSG) 來視需要封鎖存取。 [防止輸出連線](#preventoutbound)一節中會更詳細探討 NSG。 有關設計、實作及管理沒有任何輸出存取之虛擬網路的指引，則不在本文的涵蓋範圍內。

### <a name="ilpip"></a>案例 1：VM 與執行個體層級公用 IP 位址

在此案例中，VM 具有指派給它的執行個體層級公用 IP (ILPIP)。 就輸出連線而言，VM 是否負載平衡並不重要。 此案例的優先順序高於其他案例。 使用 ILPIP 時，VM 會針對所有輸出流程使用 ILPIP。  

指派給虛擬機器的公用 IP 是 1:1 關聯 (而非 1:多) 而且會實作為無狀態 1:1 NAT。  不使用連接埠偽裝 (PAT)，VM 會有所有可用的暫時連接埠。

如果您的應用程式起始許多輸出流程而發生 SNAT 連接埠耗盡的情況，請考慮指派 [ILPIP 來緩和 SNAT 限制的問題](#assignilpip)。 從整體面檢閱[管理 SNAT 耗盡](#snatexhaust)。

### <a name="lb"></a>案例 2：無執行個體層級公用 IP 位址的負載平衡 VM

在此案例中，VM 是公用 Load Balancer 後端集區的一部分。 VM 沒有指派給它的公用 IP 位址。 必須使用負載平衡器規則來建立公用 IP 前端與後端集區之間的連結，從而設定 Load Balancer 資源。

如果您未完成此規則設定，行為就如[不含執行個體層級公用 IP 的獨立 VM](#defaultsnat) 案例中所述。 規則不需要後端集區中有可運作的接聽程式，即可成功探查健康狀態。

當負載平衡的 VM 建立輸出流程時，Azure 會將輸出流量的私用來源 IP 位址轉譯為公用負載平衡器前端的公用 IP 位址。 Azure 會使用 SNAT 來執行此功能。 Azure 也會使用 [PAT](#pat) 來偽裝公用 IP 位址背後的多個私人 IP 位址。 

負載平衡器公用 IP 位址前端的暫時連接埠可用來區分 VM 所產生的個別流程。 建立輸出流程時，SNAT 會動態使用[預先配置暫時連接埠](#preallocatedports)。 在此情況下，用於 SNAT 的暫時連接埠稱為 SNAT 連接埠。

SNAT 連接埠會預先配置，如[了解 SNAT 和 PAT](#snat) 一節所述。 它們是可能耗盡的有限資源。 請務必了解[取用](#pat)它們的方式。 若要了解如何針對此取用方式進行設計及視需要降低風險，請檢閱[管理 SNAT 耗盡](#snatexhaust)。

當[多個公用 IP 位址與 Load Balancer Basic 建立關聯](load-balancer-multivip-overview.md)時，這些公用 IP 位址中的任一位址都是[輸出流程的候選項目](#multivipsnat)，且會隨機選取其中一個位址。  

若要使用 Load Balancer Basic 來監視輸出連線的健康情況，您可以使用 [Load Balancer 的 Log Analytics](load-balancer-monitor-log.md) 和[警示事件記錄檔](load-balancer-monitor-log.md#alert-event-log)，來監視是否有 SNAT 連接埠耗盡訊息。

### <a name="defaultsnat"></a>案例 3：無執行個體層級公用 IP 位址的獨立 VM

在此案例中，VM 不是公用 Load Balancer 集區的一部分 (也不是內部 Standard Load Balancer 集區的一部分)，而且沒有指派給它的 ILPIP 位址。 當 VM 建立輸出流程時，Azure 會將輸出流量的公用來源 IP 位址轉譯為私用來源 IP 位址。 此輸出流量所用的公用 IP 位址無法進行設定，而且不利於訂用帳戶的公用 IP 資源限制。

>[!IMPORTANT] 
>此案例也適用於__僅__連結內部 Basic Load Balancer 時。 當內部 Standard Load Balancer 連結至 VM 時，案例 3 __無法使用__。  除了使用內部 Standard Load Balancer 以外，您必須明確地建立[案例 1](#ilpip) 或[案例 2](#lb)。

Azure 會使用 SNAT 搭配位址偽裝 ([PAT](#pat)) 來執行此功能。 這個案例類似於[案例 2](#lb)，差別在於沒有所使用 IP 位址的控制權。 這是案例 1 和 2 不存在時的後援案例。 如果您想要能夠控制輸出位址，則不建議採用此案例。 如果輸出連線對您的應用程式很重要，您應該選擇其他案例。

SNAT 連接埠會預先配置，如[了解 SNAT 和 PAT](#snat) 一節所述。  共用可用性設定組的 VM 數目會決定要套用哪個預先配置層。  在為沒有可用性設定組的獨立 VM 決定預先配置時，其實際上會屬於大小為 1 的集區 (1024 SNAT 連接埠)。 SNAT 連接埠是可能會耗盡的有限資源。 請務必了解[取用](#pat)它們的方式。 若要了解如何針對此取用方式進行設計及視需要降低風險，請檢閱[管理 SNAT 耗盡](#snatexhaust)。

### <a name="combinations"></a>多個合併案例

您可以合併先前各節所述的案例來達到特定結果。 有多個案例存在時，會套用以下優先順序：[案例 1](#ilpip) 優先順序高於[案例 2](#lb) 和 [3](#defaultsnat)。 [案例 2](#lb) 會覆寫[案例 3](#defaultsnat)。

範例是 Azure Resource Manager 部署，其中應用程式高度依賴與有限數目目的地的輸出連線，也會透過 Load Balancer 前端接收輸入流程。 在此情況下，您可以合併案例 1 和 2 來緩和情況。 如需了解其他模式，請檢閱[管理 SNAT 耗盡](#snatexhaust)。

### <a name="multife"></a>輸出流程的多個前端

#### <a name="load-balancer-standard"></a>Load Balancer Standard

當[多個 (公用) IP 前端](load-balancer-multivip-overview.md)存在時，Load Balancer Standard 會同時使用輸出流程的所有候選項目。 如果已針對輸出連線啟用負載平衡規則，則將每個前端乘以可用預先配置 SNAT 連接埠的數目。

您可以使用新的負載平衡規則選項，選擇隱藏前端 IP 位址，不要用於輸出連線：

```json    
      "loadBalancingRules": [
        {
          "disableOutboundSnat": false
        }
      ]
```

一般來說，此選項預設為 _false_，表示此規則會在負載平衡規則的後端集區中編排相關聯 VM 的輸出 SNAT。  這個選項可以變更為 _true_，防止 Load Balancer 將相關聯前端 IP 位址用於此負載平衡規則之後端集區中 VM 的輸出連線。  您也可以為輸出流程指定一個特定 IP 位址，如[多個合併案例](#combinations)中所述。

#### <a name="load-balancer-basic"></a>Load Balancer Basic

當有[多個 (公用) IP 前端](load-balancer-multivip-overview.md)是輸出流程的候選項目時，Load Balancer Basic 會選擇單一前端來用於輸出流程。 這個選項是無法設定的，您應該將選取演算法視為隨機。 您可以為輸出流程指定一個特定 IP 位址，如[多個合併案例](#combinations)中所述。

### <a name="az"></a>可用性區域

當使用 [Standard Load Balancer 與可用性區域](load-balancer-standard-availability-zones.md)時，區域備援前端可以提供區域備援輸出 SNAT 連線和 SNAT 程式設計來克服區域失敗。  使用區域前端時，輸出 SNAT 連線會與其所屬的區域有著同樣的命運。

## <a name="snat"></a>了解 SNAT 和 PAT

### <a name="pat"></a>連接埠偽裝 SNAT (PAT)

當公用 Load Balancer 資源與 VM 執行個體建立關聯時，系統會改寫每個連出連線來源。 來源會從虛擬網路私人 IP 位址空間改寫成負載平衡器的前端「公用 IP」位址。 在公用 IP 位址空間中，5 tuple 流程 (來源 IP 位址、來源連接埠、IP 傳輸通訊協定、目的地 IP 位址、目的地連接埠) 必須是唯一的。  連接埠偽裝 SNAT 可與 TCP 或 UDP IP 通訊協定搭配使用。

在重寫私人來源 IP 位址之後，會使用暫時連接埠 (SNAT 連接埠) 來達到這個目的，因為多個流程來自單一公用 IP 位址。 連接埠偽裝 SNAT 演算法會以不同的方式針對 UDP 與 TCP 配置 SNAT 連接埠。

#### <a name="tcp"></a>TCP SNAT 連接埠

每個流程會取用一個 SNAT 連接埠至單一目的地 IP 位址、連接埠。 針對相同目的地 IP 位址、連接埠和通訊協定的多個 TCP 流程，每個 TCP 流程都會取用單一 SNAT 連接埠。 這可確保流程在從相同公用 IP 位址產生並前往相同目的地 IP 位址、連接埠及通訊協定時，會是唯一的。 

多個各自前往不同目的地 IP 位址、連接埠及通訊協定的流程會共用單一 SNAT 連接埠。 目的地 IP 位址、連接埠及通訊協定可讓流程成為唯一的，而無須使用額外的來源連接埠來區別公用 IP 位址空間中的流程。

#### <a name="udp"></a> UDP SNAT 連接埠

UDP SNAT 連接埠是由與 TCP SNAT 連接埠不同的演算法管理。  Load Balancer 會對 UDP 使用稱為 "Port-Restricted cone NAT" 的演算法。  每個流程都會取用一個 SNAT 連接埠 (不管目的地 IP 位址、連接埠為何)。

#### <a name="exhaustion"></a>耗盡

當 SNAT 連接埠資源耗盡時，輸出流程會失敗，直到現有的流程釋出 SNAT 連接埠為止。 當流程關閉並使用 [4 分鐘閒置逾時](#idletimeout)來從閒置流程回收 SNAT 連接埠時，Load Balancer 會回收 SNAT 連接埠。

UDP SNAT 連接埠通常比 TCP SNAT 連接埠更快耗盡，因為使用的演算法有所差異。 您必須在設計及調整測試時記住此差異。

如需了解可緩和通常會導致 SNAT 連接埠耗盡情況的模式，請檢閱[管理 SNAT](#snatexhaust) 一節。

### <a name="preallocatedports"></a>連接埠偽裝 SNAT (PAT) 的暫時連接埠預先配置

Azure 會使用演算法在使用連接埠偽裝 SNAT ([PAT](#pat)) 時，根據後端集區的大小決定可用的預先配置 SNAT 連接埠數目。 SNAT 連接埠是可供特定公用 IP 來源位址使用的暫時連接埠。

分別針對 UDP 和 TCP 預先配置相同數目的 SNAT 連接埠，並且對每個 IP 傳輸通訊協定個別使用。  不過，視流程為 UDP 或 TCP 而定，SNAT 連接埠的使用方式會有所不同。

>[!IMPORTANT]
>標準 SKU SNAT 程式設計是針對每個 IP 傳輸通訊協定，並且衍生自負載平衡規則。  如果只有 TCP 負載平衡規則存在，則 SNAT 只適用於 TCP。 如果您只有 TCP 負載平衡規則，而且需要 UDP 的輸出 SNAT，請從同一個前端將 UDP 負載平衡規則建立到相同的後端集區。  這會觸發 UDP 的 SNAT 程式設計。  不需要可運作的規則或健康情況探查。  不論是否已在負載平衡規則中指定傳輸通訊協定，基本 SKU SNAT 一律會針對這兩個 IP 傳輸通訊協定進行 SNAT 程式設計。

Azure 會將 SNAT 連接埠預先配置到每個 VM 之 NIC 的 IP 設定。 當 IP 設定新增至集區時，會根據後端集區大小針對此 IP 設定預先配置 SNAT 連接埠。 建立輸出流程時，[PAT](#pat) 會動態取用 (最多可達預先配置的限制) 這些連接埠，並且在流程關閉或[閒置逾時](#idletimeout)時釋出這些連接埠。

下表說明各個後端集區大小層級的 SNAT 連接埠預先配置：

| 集區大小 (VM 執行個體) | 對每個 IP 設定預先配置 SNAT 連接埠|
| --- | --- |
| 1-50 | 1,024 |
| 51-100 | 512 |
| 101-200 | 256 |
| 201-400 | 128 |
| 401-800 | 64 |
| 801-1,000 | 32 |

>[!NOTE]
> 使用 Standard Load Balancer 與[多個前端](load-balancer-multivip-overview.md)時，[每個前端 IP 位址就要乘以可用 SNAT 連接埠的數目](#multivipsnat)，這些數目如以上表格所述。 例如，50 個 VM 的後端集區具有 2 個負載平衡規則 (分別具有個別前端 IP 位址)，則在每個 IP 組態中會使用 2048 (2x 1024) 個 SNAT 連接埠。 查看[多個前端](#multife)的詳細資料。

請記住，可用的 SNAT 連接埠數目不會直接轉譯成流程數目。 單一 SNAT 連接埠可以重複使用於多個唯一目的地。 只有在必須使用連接埠來讓流程具有唯一性的情況下，才會取用連接埠。 如需有關設計和降低風險措施的指引，請參閱[如何管理這個可耗盡的資源](#snatexhaust)，以及描述 [PAT](#pat) 的小節。

變更您的後端集區大小可能會影響您建立的部分流程。 如果後端集區大小增加並轉換到下一層，在轉換至下一個較大後端集區層的期間，會回收您預先配置的一半 SNAT 連接埠。 與已回收 SNAT 連接埠關聯的流程會逾時，而必須重新建立。 嘗試建立新流程時，只要預先配置的連接埠可用，流程就會立即成功。

如果後端集區大小縮減而轉換成較低的層級，可用的 SNAT 連接埠數目就會增加。 在此情況下，現有已配置 SNAT 連接埠及其各自流程都不會受到影響。

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
如[無執行個體層級公用 IP 位址的獨立 VM](#defaultsnat) 和[無執行個體層級公用 IP 位址的負載平衡 VM](#lb) 中所述，用於 [PAT](#pat) 的[暫時連接埠](#preallocatedports)是可耗盡的資源。

如果您知道將會對相同的目的地 IP 位址和連接埠起始許多輸出 TCP 或 UDP 連線，並且觀察到失敗的輸出連線，或是支援人員告知您 SNAT 連接埠 ([PAT](#pat) 使用的預先配置[暫時連接埠](#preallocatedports)) 將耗盡，您有數個可緩和這些問題的一般選項。 請檢閱這些選項並判斷哪一個可用且最適合您的案例。 可能會有一或多個選項有助於管理此案例。

如果您在了解輸出連線行為方面遇到問題，您可以使用 IP 堆疊統計資料 (netstat)。 或是使用封包擷取來觀察連線行為，也會很有幫助。 您可以在您執行個體的客體 OS 中執行這些封包擷取，或使用[網路監看員來進行封包擷取](../network-watcher/network-watcher-packet-capture-manage-portal.md)。

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
指派 ILPIP 會將您的案例變更為 [VM 的執行個體層級公用 IP](#ilpip)。 用於每個 VM 的所有公用 IP 暫時連接埠都可供 VM 使用。 (與公用 IP 暫時連接埠會與個別後端集區之所有相關 VM 共用的案例相反)。有一些要考慮的取捨，例如公用 IP 位址的額外成本，以及將大量個別 IP 位址加入白名單所造成的潛在影響。

>[!NOTE] 
>此選項不適用於 Web 背景工作角色。

#### <a name="multifesnat"></a>使用多個前端

使用公用 Standard Load Balancer 時，您會指派[多個前端 IP 位址用於輸出連線](#multife)以及[乘以可用的 SNAT 連接埠數目](#preallocatedports)。  您需要建立前端 IP 組態、規則以及後端集區，以觸發 SNAT 到前端公用 IP 的程式設計。  此規則不需要運作，且健康情況探查不需要成功。  如果您也將多個前端用於輸入 (而非只用於輸出)，您應使用自訂健康情況探查以確保可靠性。

>[!NOTE]
>在大部分情況下，SNAT 連接埠耗盡是設計不良的徵兆。  請確定您了解為什麼您在使用更多前端以新增 SNAT 連接埠之前耗盡連接埠。  您有可能會掩蓋之後會導致失敗的問題。

#### <a name="scaleout"></a>相應放大

[預先配置的連接埠](#preallocatedports)會根據後端集區大小進行指派並分組到各層中，以在某些連接埠必須重新配置以便容納下一個較大後端集區大小層時，將中斷時間降至最低。  您可以選擇將後端集區調整到指定層適用的大小上限，以增加指定前端之 SNAT 連接埠使用率的強度。  這需要有效率地將應用程式相應放大。

例如，後端集區中的 2 部虛擬機器有 1024 個 SNAT 連接埠可供每個 IP 組態使用，總計允許 2048 個 SNAT 連接埠用於部署。  如果部署增加到 50 部虛擬機器，即使每部虛擬機器預先配置的連接埠數目維持不變，部署還是可以使用總計 51,200 (50 x 1024) 個 SNAT 連接埠。  如果您想要將部署相應放大，請檢查每層[預先配置的連接埠](#preallocatedports)數目，以確定您針對個別層相應放大到最大值。  在先前範例中，如果您已選擇相應放大到 51 個而不是 50 個執行個體，您會進到下一層且最終在每部虛擬機器及總計上具有較少的 SNAT 連接埠。

如果您相應放大到下一個較大後端集區大小的層級，且必須將已配置的連接埠重新配置，則部分輸出連線可能會逾時。  如果您只使用部分 SNAT 連接埠，則相應放大到下一個較大後端集區大小並無意義。  每當您移至下一個後端集區層級時，會有一半的現有連接埠重新配置。  如果您不希望發生這種情形，則必須讓部署符合層大小。  或確保您的應用程式可以視需要偵測及重試。  TCP 存留可協助在 SNAT 連接埠因為重新配置而無法運作時進行偵測。

### <a name="idletimeout"></a>使用 Keepalive 來重設輸出閒置逾時

連出連線有 4 分鐘的閒置逾時。 此逾時無法調整。 不過，您可以使用傳輸 (例如 TCP Keepalive) 或應用程式層 Keepalive 來重新整理閒置流程，然後視需要重設此閒置逾時。  

使用 TCP 存留時，在連線的一端啟用它們就已足夠。 例如，只在伺服器端啟用它們來重設流程的閒置計時器就已足夠，不需要在兩端都起始 TCP 存留。  應用程式層也有類似概念，包括資料庫用戶端-伺服器組態。  請檢查伺服器端以了解應用程式特定存留有哪些選項存在。

## <a name="discoveroutbound"></a>探索 VM 使用的公用 IP
有許多方式可判斷輸出連線的公用來源 IP 位址。 OpenDNS 提供可顯示您 VM 公用 IP 位址的服務。 

藉由使用 nslookup 命令，您便可以將名稱 myip.opendns.com 的 DNS 查詢傳送給 OpenDNS 解析程式。 服務會傳回用來傳送查詢的來源 IP 位址。 當您從 VM 執行下列查詢時，回應會是用於該 VM 的公用 IP：

    nslookup myip.opendns.com resolver1.opendns.com

## <a name="preventoutbound"></a>防止輸出連線
有時，您會不想要允許 VM 建立輸出流程。 或是可能需要管理可使用輸出流程來連線到哪些目的地，或哪些目的地可以發起輸入流程。 在此情況下，您可以使用[網路安全性群組](../virtual-network/security-overview.md)來管理 VM 可到連線的目的地。 您也可以使用 NSG 來管理哪個公用目的地可以起始輸入流程。

當您將 NSG 套用到經過負載平衡的虛擬機器時，請注意[服務標記](../virtual-network/security-overview.md#service-tags)和[預設安全性規則](../virtual-network/security-overview.md#default-security-rules)。 您必須確定 VM 可以從 Azure Load Balancer 接收健康情況探查要求。 

如果 NSG 封鎖來自 AZURE_LOADBALANCER 預設標籤的健全狀況探查要求，您的 VM 健全狀況探查會失敗，且會將 VM 標示為離線。 負載平衡器會停止將新的流程傳送到該 VM。

## <a name="limitations"></a>限制
- 在入口網站中設定負載平衡規則時，DisableOutboundSnat 無法作為選項使用。  請改用 REST、範本或用戶端工具。
- 由於預先 VNet 服務和其他平台服務的運作方式產生副作用，而只使用內部標準 Load Balancer 時，才可存取沒有 VNet 和其他 Microsoft 平台服務的 Web 背景工作角色。 請勿以此副作用作為個別服務本身，否則基礎平台可能會在不經通知的情況下變更。 如果在只使用內部標準 Load Balancer 時有需要，請一律假設您需要明確建立輸出連線。 您無法使用本文所述的[預設 SNAT](#defaultsnat) 案例 3。

## <a name="next-steps"></a>後續步驟

- 深入了解 [Load Balancer](load-balancer-overview.md)。
- 深入了解[標準負載平衡器](load-balancer-standard-overview.md)。
- 深入了解[網路安全性群組](../virtual-network/security-overview.md)。
- 了解 Azure 中的一些其他重要[網路功能](../networking/networking-overview.md)。
