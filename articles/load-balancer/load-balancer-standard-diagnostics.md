---
title: Azure 標準 Load Balancer 診斷 | Microsoft Docs
description: 使用可用的計量和健康情況資訊診斷 Azure 標準 Load Balancer。
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 46b152c5-6a27-4bfc-bea3-05de9ce06a57
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/27/2018
ms.author: Kumud
ms.openlocfilehash: 43945dc8810151eb701aa9e1aa1be47d4fbb0491
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43125639"
---
# <a name="metrics-and-health-diagnostics-for-standard-load-balancer"></a>標準 Load Balancer 的計量和健康情況診斷

Azure 標準 Load Balancer 為您的資源提供下列診斷功能：
* **多維度計量**：提供新的多維度診斷功能，可供公用和內部負載平衡器組態使用。 您可以監視、管理您的負載平衡器資源，以及進行其疑難排解。

* **資源健康狀態**：Azure 入口網站中的 [Load Balancer] 頁面以及 [監視器] 下的 [資源健康狀態] 頁面，會公開標準 Load Balancer 之公用負載平衡器組態的 [資源健康狀態] 區段。

本文會簡要介紹這些功能，以及如何將這些功能使用於標準 Load Balancer。

## <a name = "MultiDimensionalMetrics"></a>多維度計量

Azure Load Balancer 透過 Azure 入口網站中的新 Azure 計量 (預覽) 提供新的多維度計量，並協助您取得負載平衡器資源的即時診斷見解。 

各種標準 Load Balancer 組態提供下列計量：

| 計量 | 資源類型 | 說明 | 建議的彙總 |
| --- | --- | --- | --- |
| VIP 可用性 (資料路徑可用性) | 公用 Load Balancer | 標準 Load Balancer 會在資料路徑上持續運用，從區域內到 Load Balancer 前端，再一路到支援 VM 的 SDN 堆疊。 只要狀況良好的執行個體持續存在，測量就會依循與您應用程式的負載平衡流量相同的路徑。 此外，也會驗證您客戶所使用的資料路徑。 此測量對您的應用程式來說是看不見的，也不會干擾到其他作業。| 平均值 |
| DIP 可用性 (健康情況探查狀態) |  公用和內部負載平衡器 | 標準 Load Balancer 使用分散式健康情況探查服務，可根據您的組態設定監視應用程式端點的健康情況。 這個計量會提供負載平衡器集區中每個執行個體端點的彙總檢視，或各端點篩選過的檢視。 您可以看到 Load Balancer 藉由健康情況探查設定如何檢視應用程式的健康情況。 |  平均值 |
| SYN (同步) 封包 |  公用 Load Balancer | 標準 Load Balancer 不會終止傳輸控制通訊協定 (TCP) 連線，也不會與 TCP 或 UDP 封包流程互動。 流程及其交握一律是在來源與 VM 執行個體之間進行。 若要針對您的 TCP 通訊協定案例進行進一步疑難排解，您可使用 SYN 封包計數器來了解已進行多少次 TCP 連線嘗試。 此計量會回報已收到的 TCP SYN 封包數。| 平均值 |
| SNAT 連線 |  公用 Load Balancer |標準 Load Balancer 會回報偽裝為公用 IP 位址前端的輸出流程數目。 來源網路位址轉譯 (SNAT) 連接埠是可耗盡的資源。 此計量可以指出應用程式有多依賴 SNAT 來處理連出的起始流程。 系統會回報成功和失敗之連出 SNAT 流程的計數器，而且可用來對連出流程的健康情況進行疑難排解及了解。| 平均值 |
| 位元組計數器 |  公用和內部負載平衡器 | 標準 Load Balancer 會報告每個前端處理的資料。| 平均值 |
| 封包計數器 |  公用和內部負載平衡器 | 標準 Load Balancer 會報告每個前端處理的封包。| 平均值 |

### <a name="view-your-load-balancer-metrics-in-the-azure-portal"></a>在 Azure 入口網站中檢視負載平衡器計量

Azure 入口網站會透過 [計量] (預覽) 頁面公開負載平衡器計量，此頁面可在特定資源的負載平衡器資源頁面中以及 Azure 監視器頁面中找到。 

若要檢視標準 Load Balancer 資源的計量：
1. 移至 [計量 (預覽)] 頁面並執行下列其中一項：
   * 在負載平衡器資源頁面上，選取下拉式清單中的計量類型。
   * 在 Azure 監視器頁面上，選取負載平衡器資源。
2. 設定適當的彙總類型。
3. 選擇性設定必要的篩選和群組。

![標準 Load Balancer 的計量預覽](./media/load-balancer-standard-diagnostics/LBMetrics1.png)

圖 - 標準 Load Balancer 的 DIP 可用性和健康情況探查狀態計量

### <a name="retrieve-multi-dimensional-metrics-programmatically-via-apis"></a>透過 API 以程式設計方式擷取多維度計量

如需可供擷取多維度計量定義和值的 API 指導方針，請參閱 [Azure 監視 REST API 逐步解說](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-rest-api-walkthrough#retrieve-metric-definitions-multi-dimensional-api)。

### <a name = "DiagnosticScenarios"></a>常見診斷案例與建議的檢視

#### <a name="is-the-data-path-up-and-available-for-my-load-balancer-vip"></a>資料路徑已經啟用，且可以讓我的負載平衡器 VIP 使用嗎？

「VIP 可用性」計量會描述區域內資料路徑 (此路徑通往 VM 所在的計算主機) 的健康情況。 此計量是 Azure 基礎結構的健康情況反映。 您可以使用此計量：
- 監視服務的外部可用性
- 挖掘更深入的資料，了解服務部署所在的平台是否狀況良好、您的客體 OS 或應用程式執行個體是否狀況良好。
- 找出事件是否與您的服務或基礎資料層相關。 請勿將此計量與健康情況探查狀態 (「DIP 可用性」) 搞混。

若要取得標準 Load Balancer 資源的 VIP 可用性：
1. 確定已選取正確的負載平衡器資源。 
2. 在 [計量] 下拉式清單中，選取 [VIP 可用性]。 
3. 在 [彙總] 下拉式清單中，選取 [平均]。 
4. 此外，新增 VIP 位址或 VIP 連接埠篩選，作為必要前端 IP 位址或前端連接埠的維度，並依選取的維度進行分組。

![VIP 探查](./media/load-balancer-standard-diagnostics/LBMetrics-VIPProbing.png)

圖 - Load Balancer VIP 探查詳細資料

計量會由作用中的頻內測量所產生。 區域內的探查服務會產生此測量的流量。 此服務會在您使用公用前端建立部署時啟動，並繼續執行到您移除前端為止。 

>[!NOTE]
>目前不支援內部前端。 

系統會定期產生符合部署之前端和規則的封包。 封包會在區域中從來源周遊到主機 (後端集區中 VM 的所在位置)。 負載平衡器基礎結構會執行相同的負載平衡和轉譯作業，如同對所有其他流量所做的一樣。 這個探查是在負載平衡端點的頻內。 在探查抵達計算主機 (後端集區中狀況良好 VM 的所在位置) 之後，計算主機會針對探查服務產生回應。 您的 VM 不會看到這個流量。

VIP 可用性會因為下列原因而失敗：
- 您的部署在後端集區中沒有剩餘的狀況良好 VM。 
- 已發生基礎結構中斷。

您可以使用 [VIP 可用性計量搭配健康情況探查狀態來進行診斷](#vipavailabilityandhealthprobes)。

在大部分的情況下，請使用**平均**彙總。

#### <a name="are-the-back-end-instances-for-my-vip-responding-to-probes"></a>VIP 的後端執行個體會回應探查嗎？

「健康情況探查狀態」計量會描述應用程式部署的健康情況，這個部署是在您設定負載平衡器健康情況探查時由您所設定。 負載平衡器使用健康情況探查的狀態來判斷新流程要傳送到哪裡。 健康情況探查源自 Azure 基礎結構的位址，在 VM 的客體 OS 內可以看到。

若要取得標準 Load Balancer 資源的 DIP 可用性：
1. 選取 [DIP 可用性] 計量和 [平均] 彙總類型。 
2. 套用必要的 VIP IP 位址或連接埠篩選 (或兩者)。

![DIP 可用性](./media/load-balancer-standard-diagnostics/LBMetrics-DIPAvailability.png)

圖 - Load Balancer VIP 可用性

健康情況探查會因為下列原因而失敗：
- 您可對未接聽、未回應或使用錯誤通訊協定的連接埠設定健康情況探查。 如果您的服務使用伺服器直接回傳 (DSR 或浮動 IP) 規則，請確定服務會接聽 NIC 之 IP 組態的 IP 位址，而且不只是接聽使用前端 IP 位址所設定的回送。
- 網路安全性群組、VM 的客體 OS 防火牆或應用程式層篩選不允許您的探查。

在大部分的情況下，請使用**平均**彙總。

#### <a name="how-do-i-check-my-outbound-connection-statistics"></a>如何查看我的輸出連線統計資料？ 

「SNAT 連線」計量會描述[輸出流程](https://aka.ms/lboutbound)的成功和失敗連線數量。

失敗連線數量大於零，表示 SNAT 連接埠耗盡。 您必須進一步調查，以判斷造成失敗的原因。 SNAT 連接埠耗盡的外在表現就是無法建立[輸出流程](https://aka.ms/lboutbound)。 請參閱輸出連線的文章，以了解案例和運作機制，以及了解如何減輕及設計以避免 SNAT 連接埠耗盡。 

若要取得 SNAT 連線統計資料：
1. 選取 [SNAT 連線] 計量類型，並選取 [總和] 作為彙總。 
2. 針對成功和失敗的 SNAT 連線計數 (以不同線條表示) 依 [連線狀態] 進行分組。 

![SNAT 連線](./media/load-balancer-standard-diagnostics/LBMetrics-SNATConnection.png)

圖：Load Balancer SNAT 連線計數


#### <a name="how-do-i-check-inboundoutbound-connection-attempts-for-my-service"></a>如何查看服務的輸入/輸出連線嘗試？

「SYN 封包」計量會描述 TCP SYN 封包的數量，涵括與特定前端相關聯的已抵達或已傳送 (針對[輸出流程](https://aka.ms/lboutbound)) 封包。 您可以使用此計量來了解服務的 TCP 連線嘗試。

在大部分的情況下，請使用**總計**彙總。

![SYN 連線](./media/load-balancer-standard-diagnostics/LBMetrics-SYNCount.png)

圖：Load Balancer SYN 計數


#### <a name="how-do-i-check-my-network-bandwidth-consumption"></a>如何查看我的網路頻寬耗用？ 

「位元組和封包計數器」計量會描述您的服務在每個前端上所傳送或接收的位元組和封包數量。

在大部分的情況下，請使用**總計**彙總。

若要取得位元組或封包計數統計資料：
1. 選取 [位元組計數] 和/或 [封包計數] 計量類型，並選取 [平均] 作為彙總。 
2. 執行下列其中一個動作：
   * 對特定前端 IP、前端連接埠或後端 IP 或後端連接埠套用篩選器。
   * 取得負載平衡器資源的整體統計資料 (不使用任何篩選)。

![位元組計數](./media/load-balancer-standard-diagnostics/LBMetrics-ByteCount.png)

圖：Load Balancer 位元組計數

#### <a name = "vipavailabilityandhealthprobes"></a>我要如何診斷我的負載平衡器部署？

在單一圖表上結合 VIP 可用性和健康情況探查計量，可讓您找出問題所在並解決問題。 您可以確定 Azure 正常運作，並利用此知識確定地判斷設定或應用程式是根本原因。

您可以透過健康情況探查計量來了解 Azure 依據您所提供的設定如何看待部署的健康情況。 查看健康情況探查永遠是監視或判斷原因時最好的第一步驟。

您可以採取進一步的動作，使用 VIP 可用性計量來深入了解針對架構出特定部署的基礎資料層，Azure 是如何看待其健康情況。 當您合併這兩個計量時，就能找出錯誤可能的位置，如此圖中範例所示：

![VIP 診斷](./media/load-balancer-standard-diagnostics/LBMetrics-DIPnVIPAvailability.png)

圖：結合 DIP 和 VIP 可用性計量

此圖表會顯示以下資訊：
- 基礎結構本身狀況良好，裝載 VM 的基礎結構可以連線，而且後端中放置了多個 VM。 此資訊是以藍色線表示 IP 可用性，其為 100%。 
- 但是，在圖中一開始的橘色線顯示健康情況探查狀態 (DIP 可用性) 為 0%。 在綠色線圈起來的地方狀態 (DIP 可用性) 變成狀況良好，客戶的部署此時能夠接受新的流程。

此圖表讓客戶可以自行針對部署進行疑難排解，不必猜測或要求支援找出是否發生其他問題。 由於設定不正確或應用程式失敗導致健康情況探查失敗，所以服務變得無法使用。

### <a name = "Limitations"></a>限制

VIP 可用性目前僅適用於公用前端。

## <a name = "ResourceHealth"></a>資源健康情況狀態

標準 Load Balancer 資源的健康情況狀態會透過 [監視器] > [服務健康狀態] 底下現有的 [資源健康狀態] 公開。

>[!NOTE]
>目前只有標準 Load Balancer 的公用組態能夠使用 Load Balancer 的資源健康狀態。 內部負載平衡器資源或 Load Balancer 資源的基本 SKU 不會公開資源健康狀態。

若要檢視公用標準 Load Balancer 資源的健康情況：
1. 選取 視器 > 服務健康狀態。

   ![監視器頁面](./media/load-balancer-standard-diagnostics/LBHealth1.png)

   圖：Azure 監視器上的服務健康情況連結

2. 選取 [資源健康狀態]，然後確定已選取 [訂用帳戶識別碼] 以及 [資源類型 = 負載平衡器]。

   ![資源健康情況狀態](./media/load-balancer-standard-diagnostics/LBHealth3.png)

   圖：選取資源以檢視健康情況

3. 在清單中，選取 Load Balancer 資源，以檢視其過去的健康情況狀態。

    ![Load Balancer 健康情況狀態](./media/load-balancer-standard-diagnostics/LBHealth4.png)

   圖：Load Balancer 資源的健康情況檢視
 
下表列出各種資源健康狀態及其說明： 

| 資源健康情況狀態 | 說明 |
| --- | --- |
| 可用 | 您的公用標準 Load Balancer 資源狀況良好並可使用。 |
| 無法使用 | 您的公用標準 Load Balancer 資源狀況不良。 請藉由選取 [Azure 監視器] > [計量] 來診斷健康狀態。<br>([無法使用] 狀態也可能表示資源並未與公用標準 Load Balancer 連線。) |
| 不明 | 公用標準 Load Balancer 資源的資源健康狀態尚未更新。<br>([未知] 也可能表示資源並未與公用標準 Load Balancer 連線。)  |

## <a name="next-steps"></a>後續步驟

- 深入了解[標準負載平衡器](load-balancer-standard-overview.md)。
- 深入了解 [Load Balancer 輸出連線能力](https://aka.ms/lboutbound)。
- 了解 [Azure 監視器計量 REST API](https://docs.microsoft.com/rest/api/monitor/metrics/)。


