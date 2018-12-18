---
title: 混合式連線與兩層式應用程式 | Microsoft Docs
description: 了解如何部署虛擬設備和 UDR 以在 Azure 中建立多層式的應用程式環境
services: virtual-network
documentationcenter: na
author: jimdial
manager: carmonm
editor: tysonn
ms.assetid: 1f509bec-bdd1-470d-8aa4-3cf2bb7f6134
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/05/2016
ms.author: jdial
ms.openlocfilehash: 544ba6484b23da425d53594622122b1e18b92359
ms.sourcegitcommit: e5355615d11d69fc8d3101ca97067b3ebb3a45ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/31/2017
ms.locfileid: "23643862"
---
# <a name="virtual-appliance-scenario"></a>虛擬設備的案例
在較大的 Azure 客戶中，常見的案例是需要對網際網路公開兩層式的應用程式，同時允許從內部部署資料中心存取後層。 本文件會逐步引導您完成案例，使用使用者定義路由 (UDR)、VPN 閘道和網路虛擬裝置來部署符合下列需求的兩層式環境︰

* Web 應用程式必須只允許從公用網際網路存取。
* 裝載應用程式的網頁伺服器必須能夠存取後端應用程式伺服器。
* 從網際網路到 Web 應用程式的所有流量都必須通過防火牆虛擬設備。 這個虛擬設備只用於網際網路流量。
* 經過應用程式伺服器的所有流量都必須通過防火牆虛擬設備。 這個虛擬設備會用於存取後端伺服器，以及存取透過 VPN 閘道來自內部部署的網路。
* 系統管理員必須能夠使用管理用途專用的第三個防火牆虛擬設備，從他們的內部部署電腦來管理防火牆虛擬設備。

這是標準的 DMZ 和受保護網路的 DMZ 案例。 您可利用 NSG、防火牆虛擬設備或兩者的組合，在 Azure 中建構這類案例。 下表會顯示 NSG 和防火牆虛擬設備之間的優缺點。

|  | 優點 | 缺點 |
| --- | --- | --- |
| NSG |無成本。 <br/>整合到 Azure RBAC。 <br/>可在 ARM 範本中建立規則。 |大型環境中的複雜性各有不同。 |
| 防火牆 |完全掌控資料面。 <br/>透過防火牆主控台集中管理。 |防火牆設備的成本。 <br/>不與 Azure RBAC 相整合。 |

下列解決方案使用防火牆虛擬設備實作 DMZ/受保護網路案例。

## <a name="considerations"></a>考量
您可以使用目前可用的不同功能，在 Azure 中部署上述環境，如下所示。

* **虛擬網路 (VNet)**。 Azure VNet 運作的方式與內部部署網路相似，可以切割成一或多個子網路提供流量隔離並區隔問題。
* **虛擬設備**。 Azure Marketplace 中有數家合作夥伴提供的虛擬設備，可用為上述三種防火牆。 
* **使用者定義路由 (UDR)**。 路由表可以包含 Azure 網路用來控制 VNet 中封包流程的 UDR。 這些路由表可以套用至子網路。 Azure 其中一項最新功能是能夠將路由表套用至 GatewaySubnet，讓您能夠將從混合式連線進入 Azure VNet 的所有流量都轉送至虛擬設備。
* **IP 轉送**。 根據預設，只有封包目的地 IP 位址符合 NIC IP 位址時，Azure 網路引擎才會將封包轉送到虛擬網路介面卡 (NIC)。 因此，如果 UDR 定義封包必須傳送到指定的虛擬設備，則 Azure 網路引擎就會卸除此封包。 為確保將封包遞送到不是封包實際目的地的 VM (本例中為虛擬設備)，您需要為虛擬設備啟用 [IP 轉送]。
* **網路安全性群組 (NSG)**。 下列範例不會使用 NSG，但是您可以在此解決方案中使用子網路所套用的 NSG 和/或 NIC，以進一步篩選出入這些子網路和 NIC 的流量。

![IPv6 網路連線](./media/virtual-network-scenario-udr-gw-nva/figure01.png)

此範例中，有一個訂用帳戶包含下列項目：

* 2 個資源群組，不顯示在圖表中。 
  * **ONPREMRG**。 包含模擬內部部署網路所需要的所有資源。
  * **AZURERG**。 包含 Azure 虛擬網路環境所需要的所有資源。 
* 名為 **onpremvnet** 的 VNet，用於模仿內部部署資料中心分段，如下所示。
  * **onpremsn1**。 子網路，內含執行 Ubuntu 以模擬內部部署伺服器的虛擬機器 (VM)。
  * **onpremsn2**。 子網路，內含執行 Ubuntu 以模擬系統管理員所用之內部部署電腦的 VM。
* **onpremvnet** 上有一個防火牆虛擬設備名為 **OPFW**，用來維護 **azurevnet** 的通道。
* 名為 **azurevnet** 的 VNet 分段，如下所示。
  * **azsn1**。 專門用於外部防火牆的外部防火牆子網路。 所有的網際網路流量都會從這個子網路進入。 這個子網路只包含連結至外部防火牆的 NIC。
  * **azsn2**。 前端子網路，裝載的 VM 會作為從網際網路存取的 Web 伺服器執行。
  * **azsn3**。 後端子網路，裝載的 VM 所執行的後端應用程式伺服器會由前端 Web 伺服器存取。
  * **azsn4**。 專門提供所有防火牆虛擬設備管理存取的管理子網路。 這個子網路只包含解決方案中所用之每個防火牆虛擬設備的 NIC。
  * **GatewaySubnet**。 ExpressRoute 和 VPN 閘道所需要的 Azure 混合式連線子網路，以提供 Azure Vnet 和其他網路間的連線。 
* **azurevnet** 網路有 3 個防火牆虛擬裝置。 
  * **AZF1**。 在 Azure 中使用公用 IP 位址資源對公用網際網路公開的外部防火牆。 您需確定具備來自 Marketplace 或直接來自設備廠商的佈建 3-NIC 的虛擬設備範本。
  * **AZF2**。 用以控制 **azsn2** 和 **azsn3** 間流量的內部防火牆。 這也是 3-NIC 的虛擬設備。
  * **AZF3**。 內部部署資料中心系統管理員可存取的管理防火牆，且已連線到用來管理所有防火牆設備的管理子網路。 您可以在 Marketplace 中找到 2-NIC 虛擬設備的範本，或直接向您的設備廠商要求一份。

## <a name="user-defined-routing-udr"></a>使用者定義的路由 (UDR)
Azure 中的每個子網路都可以連結至 UDR 資料表，這份資料表用以定義如何路由在該子網路中起始的流量。 如未定義任何 UDR，Azure 就會使用預設的路由允許流量在子網路之間流動。 如需進一步了解 UDR，請瀏覽 [什麼是使用者定義路由和 IP 轉送](virtual-networks-udr-overview.md)。

為確保通訊經由正確的防火牆設備完成，根據上述的最後一個需求，您需要建立下列路由表，將 UDR 包含在 **azurevnet**中。

### <a name="azgwudr"></a>azgwudr
在這個案例中，會透過連接到 **AZF3**，使用唯一從內部部署流至 Azure 的流量來管理防火牆，且流量必須通過內部防火牆 **AZF2**。 因此， **GatewaySubnet** 中只有一個必要路由，如下所示。

| 目的地 | 下一個躍點 | 說明 |
| --- | --- | --- |
| 10.0.4.0/24 |10.0.3.11 |允許內部部署流量觸達管理防火牆 **AZF3** |

### <a name="azsn2udr"></a>azsn2udr
| 目的地 | 下一個躍點 | 說明 |
| --- | --- | --- |
| 10.0.3.0/24 |10.0.2.11 |允許流量透過 **AZF2** |
| 0.0.0.0/0 |10.0.2.10 |允許透過 **AZF1** |

### <a name="azsn3udr"></a>azsn3udr
| 目的地 | 下一個躍點 | 說明 |
| --- | --- | --- |
| 10.0.2.0/24 |10.0.3.10 |允許 **azsn2** 的流量透過 **AZF2** 從應用程式伺服器流向 Web 伺服器 |

您也必須為 **onpremvnet** 中的子網路建立路由表，以模擬內部部署資料中心。

### <a name="onpremsn1udr"></a>onpremsn1udr
| 目的地 | 下一個躍點 | 說明 |
| --- | --- | --- |
| 192.168.2.0/24 |192.168.1.4 |允許 **onpremsn2** 的流量通過 **OPFW** |

### <a name="onpremsn2udr"></a>onpremsn2udr
| 目的地 | 下一個躍點 | 說明 |
| --- | --- | --- |
| 10.0.3.0/24 |192.168.2.4 |允許流量透過 **OPFW** |
| 192.168.1.0/24 |192.168.2.4 |允許 **onpremsn1** 的流量通過 **OPFW** |

## <a name="ip-forwarding"></a>IP 轉送
UDR 和 IP 轉送兩種功能可以組合使用，以使用虛擬設備來控制 Azure VNet 中的流量流程。  虛擬應用裝置無非就是一個 VM，可執行用來以某種方式處理網路流量的應用程式，例如防火牆或 NAT 裝置。

此虛擬應用裝置 VM 必須能夠接收未定址到本身的連入流量。 若要讓 VM 接收定址到其他目的地的流量，您必須針對 VM 啟用 IP 轉送。 這是 Azure 設定，不是客體作業系統中的設定。 虛擬設備仍然需要執行某些類型的應用程式，以處理連入流量並正確予以路由。

如需深入了解 IP 轉送，請瀏覽 [什麼是使用者定義路由和 IP 轉送？](virtual-networks-udr-overview.md)。

例如，假設 Azure 虛擬網路中有下列安裝程式︰

* 子網路 **onpremsn1** 包含名為 **onpremvm1** 的 VM。
* 子網路 **onpremsn2** 包含名為 **onpremvm2** 的 VM。
* 名為 **OPFW** 的虛擬設備連線到 **onpremsn1** 和 **onpremsn2**。
* 連結至 **onpremsn1** 的使用者定義路由，指定 **onpremsn2** 的所有流量都必須傳送至 **OPFW**。

此時，如果 **onpremvm1** 嘗試建立與 **onpremvm2** 的連線，則會使用 UDR，並將流量傳送至 **OPFW** 作為下一個躍點。 請記住，實際的封包目的地並未變更，目的地仍顯示 **onpremvm2** 。 

若 **OPFW**未啟用 [IP 轉送]，Azure 虛擬網路邏輯會卸除封包，因為如果 VM 的 IP 位址是封包的目的地，其只允許將封包傳送到 VM。

若使用 [IP 轉送]，Azure 虛擬網路邏輯會將封包轉送到 OPFW，而不會變更其原始的目的地位址。 **OPFW** 必須處理封包，並決定如何加以處理。

為使上述案例得以運作，您必須對用於路由的 **OPFW**、**AZF1**、**AZF2** 和 **AZF3** 的 NIC 啟用 [IP 轉送] \(所有 NIC，連結到管理子網路的 NIC 除外)。 

## <a name="firewall-rules"></a>防火牆規則
如上所述，[IP 轉送] 只確保將封包傳送到虛擬設備。 您的設備仍然需要決定如何處理這些封包。 在上述案例中，您需要在設備中建立下列規則︰

### <a name="opfw"></a>OPFW
OPFW 代表包含下列規則的內部部署裝置︰

* **路由**：10.0.0.0/16 (**azurevnet**) 的所有流量都必須通過通道 **ONPREMAZURE** 傳送。
* **原則**︰允許 **port2** 和 **ONPREMAZURE** 之間所有的雙向流量。

### <a name="azf1"></a>AZF1
AZF1 代表包含下列規則的 Azure 虛擬設備︰

* **原則**︰允許 **port1** 和 **port2** 之間所有的雙向流量。

### <a name="azf2"></a>AZF2
AZF2 代表包含下列規則的 Azure 虛擬設備︰

* **路由**：10.0.0.0/16 (**onpremvnet**) 的所有流量都必須通過 **port1** 傳送至 Azure 閘道 IP 位址 (即 10.0.0.1)。
* **原則**︰允許 **port1** 和 **port2** 之間所有的雙向流量。

## <a name="network-security-groups-nsgs"></a>網路安全性群組 (NSG)
這個案例中不會使用 NSG。 不過，您可以將 NSG 套用到每個子網路來限制流量的出入。 例如，您可以將下列 NSG 規則套用至外部 FW 子網路。

**連入**

* 允許所有來自網際網路的 TCP 流量流向子網路任何 VM 的連接埠 80。
* 拒絕來自網際網路的所有其他流量。

**連出**

* 拒絕所有流向網際網路的流量。

## <a name="high-level-steps"></a>高階步驟
若要部署這個案例，請遵循下列的高階步驟。

1. 登入您的 Azure 訂用帳戶。
2. 如果您想要部署 VNet 來模擬內部部署網路，請佈建屬於 **ONPREMRG**的資源。
3. 佈建屬於 **AZURERG**的資源。
4. 佈建 **onpremvnet** 到 **azurevnet** 的通道。
5. 佈建好所有資源之後，請登入 **onpremvm2** 並 ping 10.0.3.101 來測試 **onpremsn2** 和 **azsn3** 之間的連線。

