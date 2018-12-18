---
title: Azure Log Analytics 中的網路效能監視器方案 | Microsoft Docs
description: 使用網路效能監控中的 ExpressRoute 監視功能，透過 Azure ExpressRoute 監視您分公司與 Azure 之間的端對端連線能力和效能。
services: log-analytics
documentationcenter: ''
author: abshamsft
manager: carmonm
editor: ''
ms.assetid: 5b9c9c83-3435-488c-b4f6-7653003ae18a
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 02/20/2018
ms.author: abshamsft
ms.component: na
ms.openlocfilehash: 27169193a468d98be879164b80e63fffde419002
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38633344"
---
# <a name="expressroute-monitor"></a>ExpressRoute 監視

您可以使用[網路效能監控](log-analytics-network-performance-monitor.md)中的 Azure ExpressRoute 監視功能，透過 Azure ExpressRoute 來監視分公司與 Azure 之間的端對端連線能力和效能。 主要優點如下： 

- 自動偵測與您訂用帳戶相關聯的 ExpressRoute 線路。
- 追蹤 ExpressRoute 的頻寬使用率、線路遺失和延遲、對等互連和 Azure 虛擬網路層級。
- 探索 ExpressRoute 線路的網路拓撲。

![ExpressRoute 監視](media/log-analytics-network-performance-monitor/expressroute-intro.png)

## <a name="configuration"></a>組態 
若要開啟網路效能監控的組態，請開啟[網路效能監控解決方案](log-analytics-network-performance-monitor.md)，然後選取 [設定]。

### <a name="configure-network-security-group-rules"></a>設定網路安全性群組規則 
針對 Azure 中用來透過網路效能監控進行監視的伺服器，設定網路安全性群組 (NSG) 規則，以在網路效能監控用來進行綜合交易的連接埠上允許 TCP 流量。 預設連接部是 8084。 此組態允許安裝在 Azure 虛擬機器上的 Operations Management Suite 代理程式與內部部署監視代理程式通訊。 

如需有關 NSG 的詳細資訊，請參閱 [網路安全性群組](../virtual-network/manage-network-security-group.md)。 

>[!NOTE]
> 在您繼續此步驟之前，請安裝內部部署伺服器代理程式和 Azure 伺服器代理程式，並且執行 EnableRules.ps1 PowerShell 指令碼。 

 
### <a name="discover-expressroute-peering-connections"></a>探索 ExpressRoute 對等互連的連線 
 
1. 選取 [ExpressRoute 對等互連] 檢視。
2. 選取 [立即探索]，以探索在與此 Azure Log Analytics 工作區連結的 Azure 訂用帳戶中，連線到虛擬網路的所有 ExpressRoute 私人對等互連。

    >[!NOTE]
    > 目前，解決方案只會探索 ExpressRoute 私人對等互連。 

    >[!NOTE]
    > 只會探索那些已連線到虛擬網路 (這些虛擬網路與連結到此 Log Analytics 工作區的訂用帳戶相關聯) 的私人對等互連。 如果 ExpressRoute 連線到與此工作區連結之訂用帳戶以外的虛擬網路，請在那些訂用帳戶中建立 Log Analytics 工作區。 然後使用網路效能監控來監視那些對等互連。 

    ![ExpressRoute 監視組態](media/log-analytics-network-performance-monitor/expressroute-configure.png)
 
 完成探索之後，探索到的私人對等互連連線會列在資料表中。 這些對等互連的監視一開始會處於停用狀態。 

### <a name="enable-monitoring-of-the-expressroute-peering-connections"></a>啟用 ExpressRoute 對等互連連線監視 

1. 選取您想要監視的私人對等互連連線。
2. 在右窗格中，選取 [監視這個對等互連] 核取方塊。 
3. 如果您要建立此連線的健康情況事件，請選取 [啟用此對等互連的健康狀況監視]。 
4. 選擇監視條件。 您可以輸入閾值，以設定健康情況事件產生的自訂閾值。 只要條件值高於為對等互連連線選取的閾值時，就會產生健康情況事件。 
5. 選取 [新增代理程式] 以選擇您要用來監視此對等互連連線的監視代理程式。 請確定您在連線的兩端都新增代理程式。 您在連線到此對等互連的虛擬網路中需要至少一個代理程式。 您也需要至少一個連線到此對等互連的內部部署代理程式。 
6. 選取 [儲存] 以儲存組態。 

   ![ExpressRoute 監視組態](media/log-analytics-network-performance-monitor/expressroute-configure-discovery.png)


在您啟用規則並選取值和代理程式之後，請等待 30 到 60 分鐘，讓值填入以及讓 [ExpressRoute 監視] 圖格出現。 當您看到監視圖格時，即表示網路效能監控正在監視您的 ExpressRoute 線路和連線資源。 

>[!NOTE]
> 在已升級為新查詢語言的工作區上能可靠運作這項功能。

## <a name="walkthrough"></a>逐步介紹 

網路效能監控儀表板會顯示 ExpressRoute 線路和對等互連連線的健康情況概觀。 

![網路效能監視器儀表板](media/log-analytics-network-performance-monitor/npm-dashboard-expressroute.png) 

### <a name="circuits-list"></a>線路清單 

若要查看所有受監視的 ExpressRoute 線路清單，請選取 [ExpressRoute 線路] 圖格。 您可以選取某個線路，然後檢視其健全狀態，以及封包遺失、頻寬使用率和延遲的趨勢圖表。 這些圖表是互動式圖表。 您可以選取一個自訂的時間範圍來繪製圖表。 在圖表上以滑鼠拖曳出一個區域，以放大並查看更細微的資料點。 

![ExpressRoute 線路清單](media/log-analytics-network-performance-monitor/expressroute-circuits.png) 

### <a name="trends-of-loss-latency-and-throughput"></a>遺失、延遲及輸送量的趨勢 

頻寬使用率、延遲及遺失圖表是互動式圖表。 您可以使用滑鼠控制項，將這些圖表的任何部分放大。 您也可以看到其他間隔的頻寬、延遲及遺失資料。 在左上方的 [動作] 按鈕底下，選取 [日期/時間] ****。 

![ExpressRoute 延遲](media/log-analytics-network-performance-monitor/expressroute-latency.png) 

### <a name="peerings-list"></a>對等互連清單 

選取儀表板上的 [私人對等互連] 圖格，即可顯示透過私人對等互連連線到虛擬網路的所有連線清單。 在這裡，您可以選取某個虛擬網路連線，然後檢視其健全狀態，以及封包遺失、頻寬使用率和延遲的趨勢圖表。 

![ExpressRoute 對等互連](media/log-analytics-network-performance-monitor/expressroute-peerings.png) 

### <a name="circuit-topology"></a>線路拓撲 

若要檢視線路拓撲，請選取 [拓撲] 圖格。 這個動作會將您移至所選線路或對等互連的拓撲檢視。 拓撲圖表會提供網路上每個區段的延遲，而且每個第 3 層躍點都會由一個圖表節點來代表。 選取躍點將會顯示關於該躍點的更多詳細資料。 若要將可見性層級提升到包含內部部署躍點，請移動 [篩選] 底下的滑動軸。 將滑動軸向左或向右移動，以增加或減少拓撲圖表中的躍點數目。 您可以看見每個區段的延遲，這可讓您更快速隔離網路上的高延遲區段。 

![ExpressRoute 拓撲](media/log-analytics-network-performance-monitor/expressroute-topology.png)

### <a name="detailed-topology-view-of-a-circuit"></a>線路的詳細拓撲檢視 

這個檢視會顯示虛擬網路連線。 

![ExpressRoute 虛擬網路連線](media/log-analytics-network-performance-monitor/expressroute-vnet.png)
 

### <a name="diagnostics"></a>診斷 

網路效能監控可協助您診斷幾個線路連線問題。 以下列出其中一些問題。 

**線路關閉。** 一旦您的內部部署資源與 Azure 虛擬網路之間的連線遺失，網路效能監控就會通知您。 這個通知可協助您在收到使用者呈報之前採取主動的動作，並減少停機時間。

![ExpressRoute 線路關閉](media/log-analytics-network-performance-monitor/expressroute-circuit-down.png)
 

**流量不會流經預定的線路。** 每當流量未流經預定的 ExpressRoute 線路時，網路效能監控就會通知您。 如果線路關閉且流量流經備份路由，就會發生這個問題。 有路由問題時也會發生這個問題。 這項資訊可協助您主動管理您路由原則中的任何組態問題，並確保是使用最合適且安全的路由。 

 

**不會流經主要線路的流量。** 當流量流經次要 ExpressRoute 線路時，網路效能監控會通知您。 即使您在此情況下不會遇到任何連線問題，但主動針對主要線路的問題進行疑難排解能讓您更妥善就緒。 

 
![ExpressRoute 流量](media/log-analytics-network-performance-monitor/expressroute-traffic-flow.png)


**由於尖峰使用量而使效能降低。** 您可以將頻寬使用率趨勢與延遲趨勢相互關聯，來識別 Azure 工作負載效能降低是否因頻寬使用率的尖峰所造成。 然後據此採取動作。

![ExpressRoute 頻寬使用率](media/log-analytics-network-performance-monitor/expressroute-peak-utilization.png)

 

## <a name="next-steps"></a>後續步驟
[搜尋記錄檔](log-analytics-log-searches.md)以檢視詳細的網路效能資料記錄。
