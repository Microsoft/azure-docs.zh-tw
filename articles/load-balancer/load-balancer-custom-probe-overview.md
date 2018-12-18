---
title: 使用 Load Balancer 健康情況探查來保護您的服務 | Microsoft Docs
description: 了解如何使用健康情況探查來監視 Load Balancer 後方的執行個體
services: load-balancer
documentationcenter: na
author: KumudD
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: 46b152c5-6a27-4bfc-bea3-05de9ce06a57
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/04/2018
ms.author: kumud
ms.openlocfilehash: ecc33fc6078dac4affe3942f1be7e039ae9e9e70
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43695420"
---
# <a name="load-balancer-health-probes"></a>Load Balancer 健康情況探查

Azure Load Balancer 會使用健康情況探查，來決定哪個後端集區執行個體會接收新流程。 您可以使用健康情況探查來偵測後端執行個體上的應用程式失敗。 您也可以對健康情況探查產生自訂回應，而且將健康情況探查用於流量控制，並通知 Load Balancer 是否要繼續傳送新流量，或停止將新流量傳送到後端執行個體。 這可用來管理負載或計畫性停機。 當健康情況探查失敗時，Load Balancer 就會停止傳送新流量給狀況不良的個別執行個體。

可用的健康情況探查類型和健康情況探查運作的方式，皆取決於您使用的 Load Balancer SKU。 例如，新的和現有流程的行為取決於流程為 TCP 或 UDP，以及您目前使用哪個 Load Balancer SKU。

| | 標準 SKU | 基本 SKU |
| --- | --- | --- |
| [探查類型](#types) | TCP, HTTP, HTTPS | TCP, HTTP |
| [探查關閉行為](#probedown) | 關閉所有探查、繼續所有 TCP 流程。 | 關閉所有探查、終止所有 TCP 流程。 | 

> [!IMPORTANT]
> Load Balancer 健康情況探查源自於 IP 位址 168.63.129.16，而且不得封鎖探查以將您的執行個體標示為已啟動。  檢閱[探查來源 IP 位址](#probesource)，以取得詳細資料。

## <a name="types"></a>探查類型

健康情況探查可以觀察後端執行個體上的任何連接埠，包括在其上提供實際服務的連接埠。 您可以針對三種不同的健康情況探查類型，來設定健康情況探查通訊協定：

- [接聽程式](#tcpprobe)
- [HTTP 端點](#httpprobe)
- [HTTPS 端點](#httpsprobe)

可用的健康情況探查類型會因為選取的 Load Balancer SKU 不同而有所差異：

|| TCP | HTTP | HTTPS |
| --- | --- | --- | --- |
| 標準 SKU |    &#9989; |   &#9989; |   &#9989; |
| 基本 SKU |   &#9989; |   &#9989; | &#10060; |

若要讓 UDP 負載平衡，您應該為使用 TCP、HTTP 或 HTTPS 健康情況探查的後端執行個體，產生自訂健康情況探查訊號。

使用 [HA 連接埠負載平衡規則](load-balancer-ha-ports-overview.md)搭配 [Standard Load Balancer](load-balancer-standard-overview.md) 時，所有連接埠都會進行負載平衡，而單一健康情況探查回應必須反映出整個執行個體的狀態。  

您不應該透過接受健康情況探查的執行個體，將健康情況探查經由 NAT 或 Proxy 傳送到 VNet 中的另一個執行個體，因為這可能導致您的案例發生連鎖性失敗。

如果您想要測試健康情況探查失敗或將個別執行個體標示為已關閉，您可以使用安全性群組來明確封鎖健康情況探查 (目的地或[來源](#probesource))。

### <a name="tcpprobe"></a> TCP 探查

TCP 探查會利用定義的連接埠執行三向開放 TCP 交握，藉此初始化連線。  接著執行四向關閉 TCP 交握。

最小探查間隔為 5 秒，狀況不良的回應數目下限為 2。  總持續時間不能超過 120 秒。

在以下情況，TCP 探查會失敗：
* 執行個體上的 TCP 接聽程式在逾時期間完全沒有回應。  系統會根據失敗探查要求數目將探查標示為已關閉，而將探查標示為已關閉之前，這些要求會設定為未獲得回應。
* 探查會從執行個體接收 TCP 重設。

#### <a name="resource-manager-template"></a>Resource Manager 範本

```json
    {
      "name": "tcp",
      "properties": {
        "protocol": "Tcp",
        "port": 1234,
        "intervalInSeconds": 5,
        "numberOfProbes": 2
      },
```

### <a name="httpprobe"></a> <a name="httpsprobe"></a> HTTP / HTTPS 探查

> [!NOTE]
> HTTPS 探查僅適用於 [Standard Load Balancer](load-balancer-standard-overview.md)。

HTTP 和 HTTPS 探查會建立 TCP 連線，以及發出含有指定路徑的 HTTP GET。 這兩個探查皆支援 HTTP GET 的相對路徑。 HTTPS 探查就是加入傳輸層安全性 (TLS，之前稱為 SSL) 包裝函式的 HTTP 探查。 當執行個體在逾時期限內以 HTTP 狀態 200 回應時，健康情況探查會標示為已啟動。  依預設，這些健康情況探查會每隔 15 秒嘗試檢查一次已設定的健康情況探查連接埠。 最小探查間隔為 5 秒。 總持續時間不能超過 120 秒。 

HTTP / HTTPS 探查也很適合在您想要實作自己的邏輯，以從負載平衡器循環中移除執行個體時使用。 例如，如果執行個體使用超過 90% CPU，並傳回非 200 HTTP 狀態，您可以決定移除執行個體。 

如果您使用雲端服務而且有使用 w3wp.exe 的 Web 角色，也能讓網站的監視自動化。 網站程式碼中的失敗，會將非 200 的狀態傳回給負載平衡器探查。  HTTP 探查會覆寫預設的客體代理程式探查。 

在以下情況，HTTP / HTTPS 探查會失敗：
* 探查端點會傳回 200 以外的 HTTP 回應碼 (例如，403、404 或 500)。 這會立即將健康情況探查標示為已關閉。 
* 探查端點在 31 秒的逾時期間內完全沒有回應。 根據設定的逾時值，在探查標示為非執行中之前 (也就是說，在傳送 SuccessFailCount 探查之前)，多個探查要求並未獲得回應。
* 探查端點會透過 TCP 重設關閉連線。

#### <a name="resource-manager-templates"></a>Resource Manager 範本

```json
    {
      "name": "http",
      "properties": {
        "protocol": "Http",
        "port": 80,
        "requestPath": "/",
        "intervalInSeconds": 5,
        "numberOfProbes": 2
      },
```

```json
    {
      "name": "https",
      "properties": {
        "protocol": "Https",
        "port": 443,
        "requestPath": "/",
        "intervalInSeconds": 5,
        "numberOfProbes": 2
      },
```

### <a name="guestagent"></a>客體代理程式探查 (僅限傳統)

依預設，雲端服務角色 (背景工作角色和 Web 角色) 會使用客體代理程式進行探查監視。   您應該考慮將這個選項作為最後手段。  您一律應該明確定義採用 TCP 或 HTTP 探查的健康情況探查。 在大部分的應用程式案例中，客體代理程式探查的效果不如明確定義的探查。  

客體代理程式探查是虛擬機器內客體代理程式的檢查。 然後，只有在執行個體處於就緒狀態時，它才會接聽並以「HTTP 200 確定」回應。 (其他狀態為忙碌、正在回收或正在停止。)

如需詳細資訊，請查看[設定適用於健全狀況探查的服務定義檔案 (csdef)](https://msdn.microsoft.com/library/azure/ee758710.aspx) 或[開始為雲端服務建立公用負載平衡器](load-balancer-get-started-internet-classic-cloud.md#check-load-balancer-health-status-for-cloud-services)。

如果客體代理程式無法以「HTTP 200 確定」回應，則負載平衡器會將執行個體標示為沒有回應。 然後，它會停止將流量傳送到該執行個體。 負載平衡器會繼續檢查執行個體。 

如果客體代理程式以 HTTP 200 回應，則負載平衡器會再次將新流量傳送到該執行個體。

使用 Web 角色時，網站程式碼通常會在不受 Azure 網狀架構或客體代理程式監視的 w3wp.exe 中執行。 在 w3wp.exe 中的失敗 (例如 HTTP 500 回應) 不會向客體代理程式報告。 因此，負載平衡器不會將該執行個體從循環中剔除。

## <a name="probehealth"></a>探查健康情況

TCP、HTTP 和 HTTPS 健康情況探查於下列狀況時會視為狀況良好，並將角色執行個體標示為狀況良好：

* 健康情況探查會在 VM 第一次開機時成功。
* SuccessFailCount 的數目 (如上所述) 可定義將角色執行個體標示為狀況良好所需的成功探查值。 如果已移除角色執行個體，成功且連續的探查數目必須大於或等於 SuccessFailCount 的值才能將角色執行個體標示為執行中。

> [!NOTE]
> 如果角色執行個體的健全狀況出現變動，負載平衡器會先等候較長的時間，才將角色執行個體重新放回狀況良好的狀態。 此額外等候時間可保護使用者和基礎結構，且為刻意設計的原則。

## <a name="probe-count-and-timeout"></a>探查計數及逾時

探查行為取決於︰

* 允許執行個體標示為已啟動的成功探查數目。
* 導致執行個體標示為已關閉的失敗探查數目。

在 SuccessFailCount 中設定的逾時和頻率值會決定執行個體是否確認為執行中或非執行中。 在 Azure 入口網站中，逾時設定為頻率值的兩倍。

負載平衡規則有已定義個別後端集區的單一健康情況探查。

## <a name="probedown"></a>探查關閉行為

### <a name="tcp-connections"></a>TCP 連線

新的 TCP 連線會接續至狀況良好且其客體作業系統和應用程式能夠接受新流量的後端執行個體。

如果後端執行個體的健康情況探查失敗，則這個後端執行個體的已建立 TCP 連線會繼續。

如果後端集區中所有執行個體的所有探查都失敗，則不會有任何新流量傳送至後端集區。 Standard Load Balancer 將允許已建立的 TCP 流量繼續。  Basic Load Balancer 會終止後端集區的所有現有 TCP 流量。
 
流量一律介於用戶端與 VM 的客體 OS 之間，因此如果沒有狀況良好的後端執行個體可接收流量，則所有探查都已關閉的集區會導致前端對 TCP 連線開啟嘗試沒有回應。

### <a name="udp-datagrams"></a>UDP 資料包

UDP 資料包會傳遞至狀況良好的後端執行個體。

UDP 是不需連線的，因此 UDP 沒有流程狀態追蹤。 如有任何後端執行個體的健康情況探查失敗，則現有的 UDP 流量可能會移到後端集區中另一個狀況良好的執行個體。

如果後端集區中所有執行個體的所有探查都失敗，則 Basic 和 Standard Load Balancer 的現有 UDP 流量將會終止。

## <a name="probesource"></a>探查來源 IP 位址

Load Balancer 會為其內部健康情況模型使用分散式探查服務。 每個 VM 所在的主機都可以透過程式設計方式來為每個客戶的組態產生健康情況探查。 健康情況探查流量會直接在產生健康情況探查的基礎結構元件和客戶 VM 之間產生。 所有 Load Balancer 健康情況探查都源自作為其來源的 IP 位址 168.63.129.16。  當您將自有的 IP 位址帶到 Azure 的虛擬網路時，此健康情況探查來源 IP 位址保證是唯一的，因為它為 Microsoft 全域保留。  此位址在所有區域中皆相同，而且不會變更。 它不應被視為安全性風險，因為只有內部 Azure 平台可以從此 IP 位址取得封包來源。 

除了 Load Balancer 健康情況探查外，下列作業會使用此 IP 位址：

- 啟用 VM 代理程式來與平台通訊，藉此表示它處於「就緒」狀態
- 啟用與 DNS 虛擬伺服器的通訊，以提供篩選後的名稱解析給未定義自訂 DNS 伺服器的客戶。  此篩選可確保客戶只可以解析其部署的主機名稱。

為了讓 Load Balancer 的健康情況探查將您的執行個體標示為已啟動，您**必須**在任何 Azure [安全性群組](../virtual-network/security-overview.md)和本機防火牆原則中允許此 IP 位址。

如果您不允許防火牆原則中有此 IP 位址，則健康情況探查將會失敗，因為它無法觸達您的執行個體。  接著，Load Balancer 會因為健康情況探查失敗而將您的執行個體標示為已關閉。  這會導致已負載平衡的服務失敗。 

此外，您不得使用 Microsoft 擁有的 IP 位址範圍 (其中包含 168.63.129.16) 來設定您的 VNet。  這會與健康情況探查的 IP 位址相衝突。

如果您的 VM 上有多個介面，您必須確保在您接收探查的介面上進行回應。  您可能需要在每個介面上，將 VM 中的這個位址設為唯一來源 NAT。

## <a name="monitoring"></a>監視

公用和內部 [Standard Load Balancer](load-balancer-standard-overview.md) 會透過 Azure 監視器將每個端點和後端執行個體的健康情況探查狀態公開為多維度計量。 然後，讓其他 Azure 服務或第三方應用程式使用這些資料。 

公用 Basic Load Balancer 會透過 Log Analytics 公開每個後端集區的健康情況探查狀態總結。  此功能不適用於內部 Basic Load Balancer。  您可以使用[記錄分析](load-balancer-monitor-log.md)來檢查公用負載平衡器探查健全狀況和探查計數。 記錄可以與 Power BI 或 Azure Operation Insights 搭配使用，以提供負載平衡器健康狀態。

## <a name="limitations"></a>限制

-  HTTPS 探查不支援使用用戶端憑證進行相互驗證。
-  目前，SDK、PowerShell 不支援 HTTPS 探查。

## <a name="next-steps"></a>後續步驟

- 深入了解[標準 Load Balancer](load-balancer-standard-overview.md)
- [開始使用 PowerShell 在資源管理員中建立公用負載平衡器](load-balancer-get-started-internet-arm-ps.md)
- [適用於健康情況探查的 REST API](https://docs.microsoft.com/rest/api/load-balancer/loadbalancerprobes/)
- 透過 [Load Balancer 的 Uservoice](https://aka.ms/lbuservoice) 要求新的健康情況探查功能
