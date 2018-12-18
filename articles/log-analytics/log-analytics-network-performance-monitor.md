---
title: Azure 中的網路效能監控解決方案 | Microsoft Docs
description: Azure 中的「網路效能監視器」可協助您近乎即時地監視網路的效能，以偵測和找出網路效能瓶頸。
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
ms.openlocfilehash: 3640f5bb2c2e9457e269bd189cbec3b627ee7349
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39626867"
---
# <a name="network-performance-monitor-solution-in-azure"></a>Azure 中的網路效能監控解決方案

![網路效能監控符號](./media/log-analytics-network-performance-monitor/npm-symbol.png)


網路效能監視器是雲端式的混合網路監視解決方案，可協助您監視網路基礎結構中多個端點間的網路效能。 也可協助您監視服務和應用程式端點的網路連線能力，以及監視 Azure ExpressRoute 的效能。 

網路效能監視器會偵測網路問題，例如流量黑洞、路由錯誤，以及傳統網路監視方法無法偵測的問題。 此解決方案會在違反網路連結的臨界值時產生警示並通知您。 它可確實而及時地偵測網路效能問題，並將問題的來源限縮於特定網路區段或裝置。 

網路效能監視器提供三種廣泛功能： 

* [效能監視器](log-analytics-network-performance-monitor-performance-monitor.md)：您可以監視雲端部署和內部部署位置、多個資料中心和分公司，以及任務關鍵性多層式應用程式或微服務之間的網路連線。 利用效能監視器，您可以在使用者提出抱怨之前偵測到網路問題。

* [服務連線能力監視](log-analytics-network-performance-monitor-service-endpoint.md)：您可以監視從使用者到所關心服務的連線、判斷路徑中有哪些基礎結構，以及識別發生網路瓶頸之處。 您可以搶先在使用者之前得知中斷情況，並查看網路路徑上發生問題的確切位置。 

    此功能可協助您執行以 HTTP、HTTPS、TCP 與 ICMP 為基礎的測試，以近乎即時或記錄的方式監視服務的可用性與回應時間。 您也可以監視因網路而造成封包遺失與延遲問題的比重。 透過網路拓撲地圖，您可以解析網速變慢的問題。 您可以利用每個躍點的延遲資料，識別從節點到服務的網路路徑沿途所發生的問題點。 透過內建測試，您完全不需任何預先設定，即可監視 Office365 和 Dynamics CRM 的網路連線。 使用此功能，您可以監視任何支援 TCP 端點的網路連線，例如網站、SaaS 應用程式、PaaS 應用程式和 SQL 資料庫。

* [ExpressRoute 監視器](log-analytics-network-performance-monitor-expressroute.md)：透過 Azure ExpressRoute 監視分公司與 Azure 之間的端對端連線與效能。  

您可以在線上深入了解[網路效能監視器](https://docs.microsoft.com/azure/networking/network-monitoring-overview)所支援的各種功能。
 
## <a name="supported-regions"></a>支援的區域
NPM 可使用在下列其中一個區域裝載的工作區，監視世界各地的網路和應用程式之間的連線情形：
* 西歐
* 美國中西部
* 美國東部
* 日本東部
* 東南亞
* 澳大利亞東南部
* 英國南部
* 美國維吉尼亞州政府

ExpressRoute 監視器的支援區域清單提供於[文件](https://docs.microsoft.com/azure/expressroute/how-to-npm?utm_swu=8117#regions)中。


## <a name="set-up-and-configure"></a>安裝及設定

### <a name="install-and-configure-agents"></a>安裝及設定代理程式 

使用[將 Windows 電腦連接到 Azure Log Analytics](log-analytics-windows-agents.md) 和[將 Operations Manager 連接到 Log Analytics](log-analytics-om-agents.md) 中的基本程序，進行代理程式安裝。

### <a name="where-to-install-the-agents"></a>安裝代理程式的位置 

* **效能監視器**：至少在一個節點上安裝 Operations Management Suite 代理程式，您即可針對與該節點連線的每個子網路監視和其他子網路的網路連線。

    若要監視網路連結，請在該連結的兩個端點上安裝代理程式。 如果您不確定您的網路拓撲，請在具有重要工作負載、且您想要監視其間網路效能的伺服器上安裝代理程式。 例如，如果您想要監視 Web 伺服器和執行 SQL 的伺服器 之間的網路連線，請在這兩部伺服器上安裝代理程式。 代理程式會監視主機之間的網路連線 (連結)，而不是主機本身。 

* **服務連線能力監視**：在您想要的每個節點上安裝 Operations Management Suite 代理程式，監視它與服務端點之間的網路連線。 舉例來說，如果您想要監視辦公室網站 O1、O2 和 O3 對 Office 365 的網路連線。 請分別在 O1、O2 和 O3 中的至少一個節點上安裝 Operations Management Suite 代理程式。 

* **ExpressRoute 監視器**：在您的 Azure 虛擬網路中安裝至少一個 Operations Management Suite 代理程式。 此外，也在透過 ExpressRoute 私人對等互連進行連線的內部部署子網路中至少安裝一個代理程式。  

### <a name="configure-operations-management-suite-agents-for-monitoring"></a>設定用於監視的 Operations Management Suite 代理程式 

網路效能監視器會使用綜合交易來監視來源與目的地代理程式之間的網路效能。 您可以從 TCP 和 ICMP 中擇一作為要在監視「效能監視器」和「服務連線能力監視」功能中用於監視的通訊協定。 只有 TCP 可以當作 ExpressRoute 監視的監視通訊協定使用。 請確定防火牆允許 Operations Management Suite 代理程式 (這些代理程式將用於在您選擇的通訊協定上進行監視) 之間的通訊。 

* **TCP 通訊協定**：如果您選擇 TCP 作為用於監視的通訊協定，請在用於「網路效能監視器」和「ExpressRoute 監視器」的代理程式上開啟防火牆連接埠，以確保代理程式可以彼此連線。 若要開啟連接埠，請使用系統管理權限執行 [EnableRules.ps1](https://aka.ms/npmpowershellscript) PowerShell 指令碼，而不在 PowerShell 視窗中使用任何參數。

    使指令碼會建立解決方案所需的登錄機碼。 此外也會建立 Windows 防火牆規則，以允許代理程式建立彼此的 TCP 連線。 此指令碼所建立的登錄機碼會指定是否要記錄偵錯記錄，以及記錄檔的路徑。 此指令碼也會定義用於通訊的代理程式 TCP 連接埠。 此指令碼會自動設定這些機碼的值。 請勿手動變更這些機碼。 預設開啟的連接埠是 8084。 您可以為指令碼提供 portNumber 參數來使用自訂連接埠。 請在指令碼執行所在的所有電腦上使用相同的連接埠。 

    >[!NOTE]
    > 指令碼只會在本機設定 Windows 防火牆。 如果您有網路防火牆，請確定它允許傳至網路效能監視器所用 TCP 連接埠的流量。

    >[!NOTE]
    > 您不需要為「服務端點監視器」執行 [EnableRules.ps1](https://aka.ms/npmpowershellscript ) PowerShell 指令碼。

    

* **ICMP 通訊協定**：如果您選擇 ICMP 作為用於監視的通訊協定，請啟用下列防火牆規則以穩定使用 ICMP：
    
   ```
   netsh advfirewall firewall add rule name="NPMDICMPV4Echo" protocol="icmpv4:8,any" dir=in action=allow 
   netsh advfirewall firewall add rule name="NPMDICMPV6Echo" protocol="icmpv6:128,any" dir=in action=allow 
   netsh advfirewall firewall add rule name="NPMDICMPV4DestinationUnreachable" protocol="icmpv4:3,any" dir=in action=allow 
   netsh advfirewall firewall add rule name="NPMDICMPV6DestinationUnreachable" protocol="icmpv6:1,any" dir=in action=allow 
   netsh advfirewall firewall add rule name="NPMDICMPV4TimeExceeded" protocol="icmpv4:11,any" dir=in action=allow 
   netsh advfirewall firewall add rule name="NPMDICMPV6TimeExceeded" protocol="icmpv6:3,any" dir=in action=allow 
   ```
 

### <a name="configure-the-solution"></a>設定方案 

1. 從 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.NetworkMonitoringOMS?tab=Overview) 將網路效能監視器解決方案新增至您的 OMS 工作區。 您也可以使用[從方案庫新增 Log Analytics 解決方案](log-analytics-add-solutions.md)中說明的程序。 
2. 開啟 Log Analytics 工作區，然後選取 [概觀] 圖格。 
3. 選取含有 *「解決方案需要其他設定」* 訊息的 [網路效能監視器]  圖格。

   ![網路效能監視器圖格](media/log-analytics-network-performance-monitor/npm-config.png)

4. 在 [設定] 頁面上，您會看到安裝 Operations Management Suite 代理程式的選項，並且可在 [一般設定] 檢視中設定用於監視的代理程式。 如前所述，如果您安裝並設定了 Operations Management Suite 代理程式，請選取 [設定] 檢視以設定您要使用的功能。 

   **效能監視器**：選擇**預設**效能監視器規則中的綜合交易所要使用的通訊協定，然後選取 [儲存並繼續]。 此通訊協定選取項目只會用於系統產生的預設規則。 您每次建立效能監視器規則時，都需要明確地選擇通訊協定。 其後，您可以隨時移至 [效能監視器] 索引標籤上的 [預設] 規則設定 (這會在您完成初始設定之後出現)，並變更通訊協定。 如果您不想使用效能監視器功能，您可以從 [效能監視器] 索引標籤上的 [預設] 規則設定停用預設規則。

   ![效能監視器檢視](media/log-analytics-network-performance-monitor/npm-synthetic-transactions.png)
    
   **服務連線能力監視**：此功能提供內建的預先設定測試，可從代理程式監視 Office 365 和 Dynamics 365 的網路連線。 請選取您要監視的 Office 365 和 Dynamics 365 服務旁邊的核取方塊，以選擇這些服務。 若要選擇您要用來監視的代理程式，請選取 [新增代理程式]。 如果您不想要使用此功能，或想要稍後再設定，請不要選擇任何項目，而直接選取 [儲存並繼續]。

   ![服務端點監視器檢視](media/log-analytics-network-performance-monitor/npm-service-endpoint-monitor.png)

   **ExpressRoute 監視器**：選取 [立即探索]，以探索在與此 Log Analytics 工作區連結的 Azure 訂用帳戶中，連線到虛擬網路的所有 ExpressRoute 私人對等互連。 

   >[!NOTE] 
   > 目前，解決方案只會探索 ExpressRoute 私人對等互連。 

   >[!NOTE] 
   > 只會探索已連線至虛擬網路 (這些虛擬網路與連結到此 Log Analytics 工作區的訂用帳戶相關聯) 的私人對等互連。 如果 ExpressRoute 連線到與此工作區連結之訂用帳戶以外的虛擬網路，請在那些訂用帳戶中建立 Log Analytics 工作區。 請使用網路效能監控來監視那些對等互連。

   ![ExpressRoute 監視器檢視](media/log-analytics-network-performance-monitor/npm-express-route.png)

   完成探索之後，探索到的私人對等互連會列在表格中。 

   ![網路效能監視器組態頁面](media/log-analytics-network-performance-monitor/npm-private-peerings.png)
    
這些對等互連的監視一開始會處於停用狀態。 請選取您要監視的每個對等互連，然後從右側的詳細資料檢視設定其監視。 選取 [儲存] 以儲存組態。 若要深入了解，請參閱「設定 ExpressRoute 監視」一文。 

設定完成之後，需要 30 分鐘到一小時的時間來填入資料。 當解決方案從您的網路中彙總資料時，您會在網路效能監視器的 [概觀] 圖格上看到「解決方案需要其他設定」訊息。 在收集資料並編製索引之後，[概觀] 圖格會變更，並對您通知網路的健康情況摘要。 接著，您可以編輯 Operations Management Suite 代理程式安裝所在節點的監視，以及從環境中探索到的子網路監視。

#### <a name="edit-monitoring-settings-for-subnets-and-nodes"></a>編輯子網路與節點的監視設定 

所有至少已安裝一個代理程式的子網路，都會列在設定頁面的  **[子網路]**  索引標籤上。 


若要啟用或停用特定子網路的監視：

1. 選取或清除  **[子網路識別碼]** 旁的核取方塊。 然後，確定已視情況選取或清除  **用於監視**  . 您可以選取或清除多個子網路。 停用後，子網路就不受監視，因為代理程式將會更新為停止 ping 其他代理程式。 
2. 選擇您要在特定子網路中監視的節點。 請從清單中選取特定子網路，並且在包含非受控與受控節點的清單之間移動所需的節點。 您可以將自訂描述新增至子網路。
3. 選取 [儲存] 以儲存組態。 

#### <a name="choose-nodes-to-monitor"></a>選擇要監視的節點

所有已安裝代理程式的節點會都列在 [節點] 索引標籤上。 

1. 選取或清除您要監視或停止監視的節點。 
2. 視需要選取 [用於監視]，或加以清除。 
3. 選取 [ **儲存**]。 


設定您想要的功能：

- [效能監視器](log-analytics-network-performance-monitor-performance-monitor.md#configuration)
- [服務端點監視](log-analytics-network-performance-monitor-performance-monitor.md#configuration)
- [ExpressRoute 監視](log-analytics-network-performance-monitor-expressroute.md#configuration)

 

## <a name="data-collection-details"></a>資料收集詳細資料
為了收集遺失和延遲資訊，當您選擇 TCP 作為通訊協定時，網路效能監視器會使用 TCP SYN-SYNACK-ACK 交握封包。 當您選擇 ICMP 作為通訊協定時，網路效能監視器會使用 ICMP ECHO ICMP ECHO REPLY。 追蹤路由也可用來取得拓撲資訊。

下表顯示網路效能監視器的資料收集方法，以及有關如何收集資料的其他詳細資料。

| 平台 | 直接代理程式 | System Center Operations Manager 代理程式 | Azure 儲存體 | 是否需要 Operations Manager？ | 透過管理群組傳送的 Operations Manager 代理程式資料 | 收集頻率 |
| --- | --- | --- | --- | --- | --- | --- |
| Windows | &#8226; | &#8226; |  |  |  |TCP 交握/ICMP ECHO 訊息會每 5 秒進行一次，而資料會每 3 分鐘傳送一次 |
 

 
此方案會使用綜合交易來評估網路的健全狀況。 Operations Management Suite 代理程式已安裝在網路交換 TCP 封包或 ICMP Echo 相互執行的各種端點上。 代理程式會使用 TCP 封包還是 ICMP Echo，取決於您為監視選取的通訊協定。 在過程中，代理程式會了解來回行程時間和封包遺失 (如果有的話)。 此外，每個代理程式也會定期執行其他代理程式的路徑追蹤，以找出網路中必須測試的所有各種路由。 使用這項資料，代理程式就能夠推論網路延遲和封包遺失數字。 測試會每五秒重複一次。 代理程式會先彙總資料約三分鐘，再將資料上傳至 Log Analytics 服務。



>[!NOTE]
> 雖然代理程式會頻繁地相互通訊，但是在進行測試時並不會產生大量的網路流量。 代理程式只依賴 TCP SYN-SYNACK-ACK 交握封包來判斷遺失和延遲。 並不會交換任何資料封包。 在此過程中，代理程式只有在需要時才會相互通訊。 代理程式通訊拓撲已最佳化，而可減少網路流量。

## <a name="use-the-solution"></a>使用解決方案 

### <a name="network-performance-monitor-overview-tile"></a>網路效能監視器概觀圖格 

啟用網路效能監視器解決方案後，[概觀] 頁面上的解決方案圖格會提供網路健康情況的快速概觀。 

 ![網路效能監視器概觀圖格](media/log-analytics-network-performance-monitor/npm-overview-tile.png)

### <a name="network-performance-monitor-dashboard"></a>網路效能監視器儀表板 

* **頂端網路健康情況事件**：此頁面會列出系統中最近的健康情況事件和警示，以及事件變成作用中的時間。 每當針對監視規則所選擇的計量值 (遺失、延遲、回應時間或頻寬使用率) 超過閾值時，就會產生健康情況事件或警示。 

* **ExpressRoute 監視器**：此頁面會提供包含解決方案監視器之各種不同 ExpressRoute 對等互連連線的健康情況摘要。 [拓撲] 圖格會透過您的網路中受監視的 ExpressRoute 線路顯示網路路徑數目。 選取此圖格可移至 [拓撲] 檢視。

* **服務連線能力監視**：此頁面會提供您所建立不同測試的健康情況摘要。 [拓撲] 圖格會顯示受監視的端點數目。 選取此圖格可移至 [拓撲] 檢視。

* **效能監視器**：此頁面會提供解決方案所監視的**網路**連結和**子網路**連結的健康情況摘要。 [拓撲] 圖格會顯示網路中受監視的網路路徑數目。 選取此圖格可移至 [拓撲] 檢視。 

* **常用查詢**：此頁面包含一組搜尋查詢，可直接擷取原始的網路監視資料。 您可以使用這些查詢作為起點，建立自己的查詢以供自訂報告之用。 

   ![網路效能監視器儀表板](media/log-analytics-network-performance-monitor/npm-dashboard.png)

 

### <a name="drill-down-for-depth"></a>深入鑽研 

您可以選取解決方案儀表板上的各種連結，更深入鑽研您感興趣的任何領域。 例如，當您看到儀表板上出現警示或狀況不良的網路連結時，您可以選取該項目以進一步調查。 接著顯示的頁面會列出特定網路連結的所有子網路連結。 您可以查看每個子網路連結的遺失、延遲和健康情況狀態。 您可以快速找出導致問題的子網路連結。 選取 [檢視節點連結] ，可查看所有節點連結中狀況不良的子網路連結。 然後，您可以看到個別的節點間連結並尋找狀況不良的節點連結。 

選取  **[檢視拓撲]** ，可在來源與目的地節點之間檢視路由的逐一躍點拓撲。 狀況不良的路由會以紅色顯示。 您可以檢視每個躍點產生的延遲，以便快速識別特定網路部分的問題。

 

### <a name="network-state-recorder-control"></a>網路狀態錄製器控制項

每個檢視會在特定時間點顯示您網路健全狀況的快照集。 根據預設，會顯示最新狀態。 位於頁面頂端的列，會顯示狀態顯示的時間點。 若要檢視過去的網路健康情況的快照集，請選取 [動作]。 您也可以為任何頁面啟用或停用在檢視最新狀態時的自動重新整理。 

 ![網路狀態錄製器](media/log-analytics-network-performance-monitor/network-state-recorder.png)

 

### <a name="trend-charts"></a>趨勢圖 

在您深入鑽研的每個層級上，您可以檢視適用計量的趨勢。 此計量可以是遺失、延遲、回應時間或頻寬使用率。 若要變更趨勢的時間間隔，請使用圖表頂端的時間控制項。 

趨勢圖會顯示效能計量效能的歷程記錄觀點。 某些網路問題的本質是暫時的，很難單憑查看網路的目前狀態就找出來。 問題可能會在有人注意前迅速浮現而後消失，並只會在稍後的時間點再度出現。 應用程式管理員也可能難以發現這類暫時性問題。 這類問題常會在應用程式的回應時間莫名增加時出現，即使所有的應用程式元件都看似順暢執行。 

藉由查看趨勢圖表，可以輕易偵測到這類問題。 問題在網路延遲或封包遺失的情況突然激增時出現。 若要調查問題，您可以使用網路狀態錄製器控制項，檢視發生問題當時的網路快照集和拓撲。

 
![趨勢圖](media/log-analytics-network-performance-monitor/trend-charts.png)
 

### <a name="topology-map"></a>拓撲圖 

網路效能監視器會在互動式拓撲圖上顯示來源與目的地端點之間路由的逐一躍點拓撲。 若要檢視拓撲圖，請選取方案儀表板上的 [拓撲] 圖格。 您也可以選取深入鑽研頁面上的 [檢視拓撲] 連結。 

拓撲圖會顯示來源與目的地之間有多少個路由，以及資料封包所採用的路徑。 您也可看見每個網路躍點所產生的延遲。 路徑延遲總計超過閾值 (在對應監視規則中所設定) 的所有路徑都會顯示為紅色。 

當您在拓撲圖上選取節點或將滑鼠移至該節點上方時，您會看到節點屬性，例如 FQDN 和 IP 位址。 選取躍點可查看其 IP 位址。 您可經由網路躍點所產生的延遲，識別出發生問題的躍點。 若要篩選特定路徑，請使用可摺疊動作窗格中的篩選器。 若要簡化網路拓樸，請使用動作窗格中的滑桿隱藏中繼躍點。 您可以使用滑鼠滾輪來放大或縮小拓撲圖。 

圖中顯示的拓撲是第 3 層拓撲，不包含第 2 層裝置和連線。 

 
![拓撲圖](media/log-analytics-network-performance-monitor/topology-map.png)
 

## <a name="log-analytics-search"></a>Log Analytics 搜尋 

以圖形方式透過網路效能監視器儀表板和深入鑽研頁面公開的所有資料，也都可以在 [Log Analytics 搜尋](log-analytics-log-search-new.md)中取得。 您可以對存放庫中的資料執行互動式分析，並且讓不同來源的資料相互關聯。 您也可以建立自訂警示和檢視，並將資料匯出至 Excel、Power BI 或可共用的連結。 儀表板中的  **[常用查詢]**  區域有一些實用的查詢，您可以使用這些查詢作為起點來建立自己的查詢和報告。 

## <a name="alerts"></a>警示

網路效能監控使用 [Azure 監視器](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts)的警示功能。

這表示所有警示都可以透過[動作群組](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-action-groups#overview)加以管理。  

如果您是透過 OMS 建立警示的 NPM 使用者： 
1. 將您重新導向 Azure 入口網站的連結將會出現。 按一下可存取入口網站。
2. 按一下 [網路效能監控] 解決方案圖格。 
3. 導覽至 [設定]。  
4. 選取您要建立警示的測試，然後遵循下文所述的步驟。

如果您是透過 Azure 入口網站建立警示的 NPM 使用者：  
1. 您可以選擇直接輸入電子郵件，或選擇透過動作群組建立警示。
2. 如果您選擇直接輸入電子郵件，系統會建立名為 **NPM 電子郵件動作群組**的動作群組，並將電子郵件識別碼加入該動作群組。
3. 如果您選擇使用動作群組，必須選取先前建立的動作群組。 您可以在[這裡](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-action-groups#create-an-action-group-by-using-the-azure-portal)了解如何建立動作群組。 
4. 成功建立警示後，您可以使用「管理警示」連結管理警示。 

##<a name="pricing"></a>價格

定價資訊可在[線上](log-analytics-network-performance-monitor-pricing-faq.md)查詢。

## <a name="provide-feedback"></a>提供意見反應 

* **UserVoice**：對於您希望我們改善的網路效能監視器功能，您可以張貼您的想法。 請瀏覽 [UserVoice 頁面](https://feedback.azure.com/forums/267889-log-analytics/category/188146-network-monitoring)。 

* **加入我們的社群**：我們竭誠歡迎新客戶加入我們的社群。 加入其中，您就能夠及早存取新功能，並有機會協助我們改善網路效能監視器。 如果您想加入，請填妥這份 [快速問卷調查](https://aka.ms/npmcohort)。 

## <a name="next-steps"></a>後續步驟 
深入了解[效能監視器](log-analytics-network-performance-monitor-performance-monitor.md)、[服務連線能力監視](log-analytics-network-performance-monitor-performance-monitor.md)與 [ExpressRoute 監視](log-analytics-network-performance-monitor-expressroute.md)。 
