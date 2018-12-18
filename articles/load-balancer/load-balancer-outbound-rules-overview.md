---
title: Azure Load Balancer 中的輸出規則 | Microsoft Docs
description: 使用輸出規則來定義輸出網路位址轉譯
services: load-balancer
documentationcenter: na
author: KumudD
manager: jpconnock
tags: azure-resource-manager
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/24/2018
ms.author: kumud
ms.openlocfilehash: 63c193b4757c28f809a33b917058df36467d4db4
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47163013"
---
# <a name="load-balancer-outbound-rules"></a>Load Balancer 輸出規則

除了輸入之外，Azure Load Balancer 還需要提供來自虛擬網路的輸出連線。  輸出規則可以簡單地設定公用 [Standard Load Balancer](load-balancer-standard-overview.md) 的輸出網路位址轉譯。  您可以完整宣告式控制輸出連線，以針對您的特定需求擴充和調整此功能。

![Load Balancer 輸出規則](media/load-balancer-outbound-rules-overview/load-balancer-outbound-rules.png)

運用輸出規則，您可以使用 Load Balancer： 
- 從頭定義輸出 NAT。
- 擴充並調整現有輸出 NAT 的行為。 

輸出規則可讓您控制：
- 哪些虛擬機器應該轉譯成哪些公用 IP 位址。 
- 應該如何配置[輸出 SNAT 連接埠](load-balancer-outbound-connections.md#snat)。
- 要提供其輸出轉譯的通訊協定。
- 要用於輸出連線閒置逾時的持續時間。
- 是否傳送閒置逾時時的 TCP 重設 (在公開預覽中)。 

輸出規則會擴充[輸出連線](load-balancer-outbound-connections.md)一文中所述的[情節 2](load-balancer-outbound-connections.md#lb)，而且情節優先順序會保持現狀。

## <a name="outbound-rule"></a>輸出規則

如同所有 Load Balancer 規則，輸出規則會遵循與負載平衡和輸入 NAT 規則相同的熟悉語法：

**前端** + **參數** + **後端集區**

輸出規則會將「後端集區所識別的所有虛擬機器」的輸出 NAT 設定為轉譯成「前端」。  而且，「參數」提供對輸出 NAT 演算法的額外精細控制。

API 版本 "2018-07-01" 允許輸出規則定義建構如下：

```json
      "outboundRules": [
        {
          "frontendIPConfigurations": [ list_of_frontend_ip_configuations ],
          "allocatedOutboundPorts": number_of_SNAT_ports,
          "idleTimeoutInMinutes": 4 through 66,
          "enableTcpReset": true | false,
          "protocol": "Tcp" | "Udp" | "All",
          "backendAddressPool": backend_pool_reference,
        }
      ]
```

>[!NOTE]
>有效的輸出 NAT 設定包含所有輸出規則和負載平衡規則。 輸出規則會累加到負載平衡規則。 請檢閱[停用負載平衡規則的輸出 NAT](#disablesnat)，以管理將多個規則套用至 VM 時的有效輸出 NAT 轉譯。 所定義輸出規則使用與負載平衡規則相同的公用 IP 位址時，您必須[停用輸出 SNAT](#disablesnat)。

### <a name="scale"></a> 使用多個 IP 位址擴充輸出 NAT

雖然輸出規則只能與單一公用 IP 位址搭配使用，但是輸出規則可減輕擴充輸出 NAT 的設定負擔。 您可以使用多個 IP 位址來規劃大規模情節，也可以使用輸出規則來減輕易於 [SNAT 消耗](load-balancer-outbound-connections.md#snatexhaust)模式。  

前端所提供的每個其他 IP 位址都會提供 64,000 個暫時連接埠，供 Load Balancer 當成 SNAT 連接埠使用。 負載平衡或輸入 NAT 規則有單一前端，而輸出規則會擴充前端概念，並允許每個規則的多個前端。  運用每個規則的多個前端，每個公用 IP 位址都會有成倍的可用 SNAT 連接埠數量，而且可以支援非常大的情節。

此外，您還可以直接搭配使用[公用 IP 首碼](https://aka.ms/lbpublicipprefix)與輸出規則。  這樣可以更輕鬆地擴充和簡化源自 Azure 部署的流程白名單。 您可以在 Load Balancer 資源內設定前端 IP 設定，直接參考公用 IP 位址首碼。  這可讓 Load Balancer 獨佔公用 IP 首碼，而輸出規則將會自動使用輸出連線之公用 IP 首碼內包含的所有公用 IP 位址。  公用 IP 首碼範圍內的每個 IP 位址都會根據 IP 位址提供 64,000 個暫時連接埠，供 Load Balancer 當成 SNAT 連接埠使用。   

使用此選項作為輸出規則必須完整控制公用 IP 首碼時，您不能有從公用 IP 首碼建立的個別公用 IP 位址資源。  如果您需要更精細的控制，則可以從公用 IP 首碼建立個別公用 IP 位址資源，並將多個公用 IP 位址個別指派給輸出規則的前端。

### <a name="snatports"></a> 調整 SNAT 連接埠配置

您可以使用輸出規則來調整[根據後端集區大小的自動 SNAT 連接埠配置](load-balancer-outbound-connections.md#preallocatedports)，並配置比自動 SNAT 連接埠配置所提供的更多或更少。

使用下列參數，為每個 VM 配置 10,000 個 SNAT 連接埠 (NIC IP 設定)。
 

          "allocatedOutboundPorts": 10000

來自輸出規則之所有前端的每個公用 IP 位址都會提供最多 64,000 個暫時連接埠，以當成 SNAT 連接埠使用。  Load Balancer 會以 8 的倍數來配置 SNAT 連接埠數目。 如果您提供的值無法與 8 整除，則會拒絕設定作業。  如果您嘗試根據公用 IP 位址數目來配置比可用 SNAT 連接埠更多的 SNAT 連接埠，則會拒絕設定作業。  例如，如果您為每個 VM 配置 10,000 個連接埠，而且後端集區中的 7 部 VM 將會共用單一公用 IP 位址，則會拒絕設定 (7 x 10,0000 個 SNAT 連接埠 > 64,000 個 SNAT 連接埠)。  您可以將更多公用 IP 位址新增至輸出規則的前端，以啟用此情節。

您可以指定連接埠數目 0，以還原為[根據後端集區大小的自動 SNAT 連接埠配置](load-balancer-outbound-connections.md#preallocatedports)。

### <a name="idletimeout"></a> 控制輸出流程閒置逾時

輸出規則提供設定參數來控制輸出流程閒置逾時，並使其符合您應用程式的需求。  輸出閒置逾時預設為 4 分鐘。  此參數接受從 4 到 66 到特定分鐘數的值，作為符合此特定規則之流程的閒置逾時。

使用下列參數，將輸出閒置逾時設定為 1 小時：

          "idleTimeoutInMinutes": 60

### <a name="tcprst"></a> <a name="tcpreset"></a> 啟用閒置逾時的 TCP 重設 (預覽)

Load Balancer 的預設行為，是在達到輸出閒置逾時的當時以無訊息方式捨棄流程。  運用 enableTCPReset 參數，您可以啟用更可預測的應用程式行為，並控制是否在輸出閒置逾時的當時傳送雙向 TCP 重設 (TCP RST)。 

使用下列參數，以在輸出規則上啟用 TCP 重設：

           "enableTcpReset": true

如需詳細資料 (包含區域可用性)，請檢閱[閒置逾時的 TCP 重設 (預覽)](https://aka.ms/lbtcpreset)。

### <a name="proto"></a> 使用單一規則支援 TCP 和 UDP 傳輸通訊協定

建議您針對輸出規則的傳輸通訊協定使用 "All"，但您也可以在需要時將輸出規則套用至特定傳輸通訊協定。

使用下列參數，將通訊協定設定為 TCP 和 UDP：

          "protocol": "All"

### <a name="disablesnat"></a> 停用負載平衡規則的輸出 NAT

如先前所述，負載平衡規則會提供輸出 NAT 的自動程式設計。 不過，某些情節有所助益，或需要您透過負載平衡規則停用輸出 NAT 的自動程式設計，以控制或精簡行為。  輸出規則的情節對於停止自動輸出 NAT 程式設計十分重要。

您可以透過兩種方式來使用此參數：
- 將輸入 IP 位址用於輸出 NAT 的選擇性隱藏項目。  輸出規則會累加到負載平衡規則；而且，如果設定此參數，則會控制輸出規則。
  
- 調整同時用於輸入和輸出的 IP 位址輸出 NAT 參數。  必須停用自動輸出 NAT 程式設計，以允許輸出規則取得控制。  例如，若要變更位址的 SNAT 連接埠配置也用於輸入，此參數必須設定為 true。  如果您嘗試使用輸出規則來重新定義也用於輸入的 IP 位址參數，但尚未發行負載平衡規則的輸出 NAT 程式設計，則設定輸出規則的作業會失敗。

>[!IMPORTANT]
> 如果您將此參數設定為 true，但沒有輸出規則 (或[執行個體層級公用 IP 情節](load-balancer-outbound-connections.md#ilpip)) 定義輸出連線，則您的虛擬機器不會有輸出連線。  您 VM 或應用程式的某些作業可能取決於可用輸出連線。 請確定您了解情節的相依性，並考慮這項變更的影響。

您可以使用此設定參數，在負載平衡規則上停用輸出 SNAT：

```json
      "loadBalancingRules": [
        {
          "disableOutboundSnat": true
        }
      ]
```

disableOutboundSNAT 參數預設為 false，這表示負載平衡規則**確實**提供自動輸出 NAT 以作為負載平衡規則設定的鏡像映像。  

如果您在負載平衡規則上將 disableOutboundSnat 設定為 true，則負載平衡規則會發行自動輸出 NAT 程式設計的控制。  停用因負載平衡規則的輸出 SNAT。

### <a name="reuse-existing-or-define-new-backend-pools"></a>重複使用現有後端集區或定義新的後端集區

輸出規則未引進用於定義應套用規則的 VM 群組新概念。  相反地，它們會重複使用也用於負載平衡規則的後端集區概念。 您可以使用此來簡化設定，方法是重複使用現有後端集區定義，或建立輸出規則特定的後端集區定義。

## <a name="scenarios"></a>案例

### <a name="groom"></a> 清理與一組特定公用 IP 位址的輸出連線

您可以使用輸出規則來清理似乎源自一組特定公用 IP 位址的輸出連線，以簡化白名單情節。  此來源公用 IP 位址可以與負載平衡規則所使用的來源公用 IP 位址相同，或與負載平衡規則所使用的一組不同公用 IP 位址。  

1. 建立[公用 IP 首碼](https://aka.ms/lbpublicipprefix) (或從公用 IP 首碼的公用 IP 位址)
2. 建立公用標準 Load Balancer
3. 建立可參考您要使用的公用 IP 首碼 (或公用 IP 位址) 前端
4. 重複使用後端集區，或建立後端集區，並將 VM 放入公用 Load Balancer 的後端集區
5. 在公用 Load Balancer 上設定輸出規則，以使用前端對這些 VM 的輸出 NAT 進行程式設計
   
如果您不想要將負載平衡規則用於輸出，則需要在負載平衡規則上[停用輸出 SNAT](#disablesnat)。

### <a name="modifysnat"></a> 修改 SNAT 連接埠配置

您可以使用輸出規則來調整[根據後端集區大小的自動 SNAT 連接埠配置](load-balancer-outbound-connections.md#preallocatedports)。

例如，如果您有兩部虛擬機器共用輸出 NAT 的單一公用 IP 位址，則建議您在遇到 SNAT 消耗時增加從預設 1024 個連接埠配置的 SNAT 連接埠數目。 每個公用 IP 位址都會提供最多 64,000 個暫時連接埠。  如果您使用單一公用 IP 位址前端來設定輸出規則，則可以將共 64,000 個 SNAT 連接埠散發到後端集區中的 VM。  針對兩個 VM，最多可以使用輸出規則來配置 32,000 個 SNAT 連接埠 (2x 32,000 = 64,000)。

請檢閱[輸出連線](load-balancer-outbound-connections.md)，以及如何配置和使用 [SNAT](load-balancer-outbound-connections.md#snat) 連接埠的詳細資料。

### <a name="outboundonly"></a> 僅啟用輸出

您可以使用公用 Standard Load Balancer 提供一組 VM 的輸出 NAT。 在此情節中，您可以使用輸出規則本身，而不需要任何其他規則。

#### <a name="outbound-nat-for-vms-only-no-inbound"></a>僅 VM 的輸出 NAT (無輸入)

定義公用 Standard Load Balancer，並將 VM 放入後端集區，以及設定輸出規則以對輸出 NAT 進行程式設計並清理輸出連線以源自特定公用 IP 位址。 您也可以使用公用 IP 首碼，簡化將輸出連線來源設為白名單。

1. 建立公用 Standard Load Balancer。
2. 建立後端集區，並將 VM 放入公用 Load Balancer 的後端集區。
3. 在公用 Load Balancer 上設定輸出規則，以對這些 VM 的輸出 NAT 進行程式設計。

#### <a name="outbound-nat-for-internal-standard-load-balancer-scenarios"></a>內部 Standard Load Balancer 情節的輸出 NAT

使用內部 Standard Load Balancer 時，除非同時設定公用 Standard Load Balancer，否則無法使用輸出 NAT。 變更此作業的方式是使用輸出規則來建立受內部 Standard Load Balancer 保護的 VM 輸出連線。

1. 建立公用 Standard Load Balancer。
2. 建立後端集區，並將 VM 放入公用 Load Balancer 的後端集區。
3. 在公用 Load Balancer 上設定輸出規則，以對這些 VM 的輸出 NAT 進行程式設計。

#### <a name="enable-both-tcp--udp-protocols-for-outbound-nat-with-a-public-standard-load-balancer"></a>使用公用 Standard Load Balancer 啟用輸出 NAT 的 TCP 和 UDP 通訊協定

- 使用公用 Standard Load Balancer 時，所提供的自動輸出 NAT 程式設計會符合負載平衡規則的傳輸通訊協定。  

   1. 在負載平衡規則上停用輸出 SNAT。
   2. 在相同的 Load Balancer 上設定輸出規則。
   3. 重複使用 VM 已使用的後端集區。
   4. 指定 "protocol": "All" 作為輸出規則的一部分。

- 僅使用輸出 NAT 規則時，不會提供輸出 NAT。

   1. 將 VM 放入後端集區中。
   2. 使用公用 IP 位址或公用 IP 首碼，定義一或多個前端 IP 設定。
   3. 在相同的 Load Balancer 上設定輸出規則。
   4. 指定 "protocol": "All" 作為輸出規則的一部分

## <a name="limitations"></a>限制

- 每個前端 IP 位址的可用暫時連接埠數目上限為 64,000。
- 可設定輸出閒置逾時的範圍為 4 到 66 分鐘 (240 到 4000 秒)。
- Load Balancer 不支援輸出 NAT 的 ICMP。
- 入口網站無法用來設定或檢視輸出規則。  請改為使用範本、REST API、Az CLI 2.0 或 PowerShell。

## <a name="next-steps"></a>後續步驟

- 了解如何使用 [Load Balancer 來進行輸出連線](load-balancer-outbound-connections.md)。
- 深入了解 [Standard Load Balancer](load-balancer-standard-overview.md)。
- 了解[閒置逾時的雙向 TCP 重設](load-balancer-tcp-reset.md)。
- [使用 Azure CLI 2.0 設定輸出規則](configure-load-balancer-outbound-cli.md)。
