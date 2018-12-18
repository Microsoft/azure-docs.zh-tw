---
title: SAP HANA on Azure (大型執行個體) 的網路架構 | Microsoft Docs
description: SAP HANA on Azure (大型執行個體) 部署方式的網路架構。
services: virtual-machines-linux
documentationcenter: ''
author: RicksterCDN
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/04/2018
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 1262ed841fe8f6f9c2d5339d79abf06c1ab15a25
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392868"
---
# <a name="sap-hana-large-instances-network-architecture"></a>SAP HANA (大型執行個體) 網路架構

Azure 網路服務的架構是在 HANA 大型執行個體上成功部署 SAP 應用程式的關鍵元件。 一般而言，SAP HANA on Azure (大型執行個體) 部署具有較大的 SAP 架構，其中包含數個擁有不同資料庫大小、CPU 資源耗用量及記憶體使用量的不同 SAP 解決方案。 這些 SAP 系統可能並非全都以 SAP HANA 為基礎。 您的 SAP 環境可能會混用下列項目：

- 部署在內部部署環境中的 SAP 系統。 因大小的緣故，這些系統目前無法裝載在 Azure 中。 例如，如果生產 SAP ERP 系統在 SQL Server (作為資料庫) 上執行，且所需的 CPU 或記憶體資源高於 VM 所能提供，就是如此。
- 部署在內部部署環境中以 SAP HANA 為基礎的 SAP 系統。
- 部署在 VM 中的 SAP 系統。 根據資源耗用量與記憶體需求，這些系統可以是任何可在 Azure (在 VM 上) 順利部署之 SAP NetWeaver 型應用程式的開發、測試、沙箱或生產環境執行個體。 這些系統也可能以 SQL Server 之類的資料庫為基礎。 如需詳細資訊，請參閱 [SAP 支援附註 1928533 - Azure 上的 SAP 應用程式︰支援的產品和 Azure VM 類型](https://launchpad.support.sap.com/#/notes/1928533/E)。 此外，這些系統也可能以 SAP HANA 之類的資料庫為基礎。 如需詳細資訊，請參閱[經 SAP HANA 認證的 IaaS 平台](http://global.sap.com/community/ebook/2014-09-02-hana-hardware/enEN/iaas.html)。
- 部署在 Azure (在 VM 上) 中並利用「Azure 大型執行個體」戳記中 SAP HANA on Azure (大型執行個體) 的 SAP 應用程式伺服器。

混合式 SAP 架構 (含有四個以上的不同部署案例) 為典型架構。 此外也有許多在 Azure 中完成 SAP 架構執行的客戶案例。 由於 VM 的功能愈來愈強大，將所有 SAP 解決方案都移至 Azure 的客戶數目也隨之增加。

在部署於 Azure 中之 SAP 系統內容中的 Azure 網路並不複雜。 其依循原則如下：

- Azure 虛擬網路必須連線至會連接到內部部署網路的 ExpressRoute 線路。
- 將內部部署連線至 Azure 的 ExpressRoute 線路通常應有 1 Gbps 或更高的頻寬。 此最低頻寬可讓您有足夠的頻寬在內部部署系統與執行於 VM 的系統之間傳輸資料。 它也讓內部部署使用者有足夠的頻寬可連線至 Azure 系統。
- Azure 中的所有 SAP 系統都必須在虛擬網路中設定妥當，才能彼此通訊。
- 裝載於內部部署環境中的 Active Directory 與 DNS 會透過 ExpressRoute 從內部部署環境延伸到 Azure。


> [!NOTE] 
> 從計費的觀點來看，在特定 Azure 區域中，只有一個 Azure 訂用帳戶可連結至大型執行個體戳記中的一個租用戶。 反之，單一大型執行個體戳記租用戶也只能連結至一個 Azure 訂用帳戶。 Azure 中其他可計費的物件也有此需求。

如果將 SAP HANA on Azure (大型執行個體) 部署在多個不同的 Azure 區域中，則會在大型執行個體戳記中部署個別的租用戶。 只要這些執行個體都是相同 SAP 架構的一部分，您便可以在同一個 Azure 訂用帳戶下執行兩者。 

> [!IMPORTANT] 
> 使用 SAP HANA on Azure (大型執行個體) 時，僅支援 Azure Resource Manager 部署。

 

## <a name="additional-virtual-network-information"></a>其他虛擬網路資訊

若要將虛擬網路連線至 ExpressRoute，必須建立 Azure 閘道。 如需詳細資訊，請參閱[關於 ExpressRoute 的虛擬網路閘道](../../../expressroute/expressroute-about-virtual-network-gateways.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 

Azure 閘道可以與 ExpressRoute 搭配用於 Azure 以外的基礎結構或 Azure 大型執行個體戳記。 Azure 閘道也可用於虛擬網路之間的連線。 如需詳細資訊，請參閱[使用 PowerShell 設定資源管理員的網路對網路連線](../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 您可以將 Azure 閘道連線至最多四個不同的 ExpressRoute 連線，只要這些連線來自不同的 Microsoft Enterprise Edge 路由器即可。 如需詳細資料，請參閱 [Azure 上的 SAP HANA (大型執行個體) 基礎結構和連線](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 

> [!NOTE] 
> Azure 閘道為這兩個使用案例提供的輸送量是不同的。 如需詳細資訊，請參閱[關於 VPN 閘道](../../../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 在使用 ExpressRoute 連線的情況下，虛擬網路閘道可達到的最大輸送量是 10 Gbps。 在位於虛擬網路中的 VM 與內部部署系統之間複製檔案 (以單一複製串流的形式)，並不會達到不同閘道 SKU 的最大輸送量。 若要使用虛擬網路閘道的完整頻寬，請使用多個資料流。 或者，您必須以單一檔案的平行資料流複製不同的檔案。


## <a name="networking-architecture-for-hana-large-instance"></a>適用於 HANA 大型執行個體的網路架構
適用於 HANA 大型執行個體的網路架構，可區分為四個不同的部分：

- 內部部署網路和 Azure 的 ExpressRoute 連線。 這個部分是客戶網域，會透過 ExpressRoute 連線至 Azure。 請參閱下圖右下方的區域。
- 如先前所討論，Azure 網路服務與同樣也有閘道的虛擬網路搭配使用。 這個部分是您需要針對應用程式需求、安全性及合規性需求尋找適當設計的區域。 是否使用 HANA 大型執行個體，是您在虛擬網路的數目和要選擇的 Azure 閘道 SKU 方面所應考量的另一個要點。 請查看圖中的右上方。
- HANA 大型執行個體透過 ExpressRoute 技術連線至 Azure。 這部分已部署且會由 Microsoft 來處理。 您唯一需要做的，是在將資產部署至 HANA 大型執行個體後提供一些 IP 位址範圍，再將 ExpressRoute 線路連線至虛擬網路。 如需詳細資料，請參閱 [Azure 上的 SAP HANA (大型執行個體) 基礎結構和連線](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 
- HANA 大型執行個體中的網路功能，對您而言大部分是透明的。

![連線至 SAP HANA on Azure (大型執行個體) 與內部部署環境的虛擬網路](./media/hana-overview-architecture/image3-on-premises-infrastructure.png)

由於您使用 HANA 大型執行個體，因此內部部署資產必須透過 ExpressRoute 連線至 Azure 的需求並不會變更。 以一或多個虛擬網路執行 VM 的需求也不會變更 (這些 VM 會裝載應用程式層，連線至裝載於 HANA 大型執行個體單位中的 HANA 執行個體)。 

與 Azure 中的 SAP 部署的差異為：

- 客戶租用戶的 HANA 大型執行個體單位會透過另一個 ExpressRoute 線路連線至您的虛擬網路。 為了區隔負載狀況，內部部署到虛擬網路 ExpressRoute 連結，以及虛擬網路與 HANA 大型執行個體之間的連結，並不會共用相同的路由器。
- SAP 應用程式層和 HANA 大型執行個體之間的工作負載設定檔是不同性質的許多小型要求與高載，例如從 SAP HANA 到應用程式層的資料傳輸 (結果集)。
- 相較於在內部部署與 Azure 之間交換資料的典型案例，SAP 應用程式架構對於網路延遲更敏感。
- 虛擬網路閘道至少會有兩個 ExpressRoute 連線。 這兩個連線會共用虛擬網路閘道之連入資料的最大頻寬。

在 VM 與 HANA 大型執行個體單位之間經歷的網路延遲，可能大於典型的 VM 對 VM 網路來回延遲。 相依於 Azure 區域，測量到的值可超過 0.7 毫秒的來回延遲，這在 [SAP 附註 #1100926 - 常見問題集：網路效能](https://launchpad.support.sap.com/#/notes/1100926/E)中已歸類為低於平均值。 根據測量 Azure VM 與 HANA 大型執行個體單位之間的網路來回延遲時所使用的 Azure 區域和工具，測量的延遲最高可達 2 毫秒左右。 不過，客戶可順利在 SAP HANA 大型執行個體上部署以 SAP HANA 為基礎的生產環境 SAP 應用程式。 請務必在 Azure HANA 大型執行個體中徹底測試您的商務程序。
 
若要提供 VM 和 HANA 大型執行個體之間具決定性的網路延遲，則必須選擇虛擬網路閘道 SKU。 不同於內部部署與 VM 之間的流量模式，VM 和 HANA 大型執行個體之間的流量模式可以開發很小但高載的要求和資料磁碟區來進行傳輸。 為了妥善處理這類高載，強烈建議您使用 UltraPerformance 閘道 SKU。 針對類型 II 類別的 HANA 大型執行個體 SKU，使用 UltraPerformance 閘道 SKU 作為虛擬網路閘道是必要的。

> [!IMPORTANT] 
> 考慮到 SAP 應用程式與資料庫層之間的整體網路流量，因此僅支援使用虛擬網路的 HighPerformance 或 UltraPerformance 閘道 SKU 來連線至 SAP HANA on Azure (大型執行個體)。 針對 HANA 大型執行個體類型 II SKU，僅支援使用 UltraPerformance 閘道 SKU 作為虛擬網路閘道。



## <a name="single-sap-system"></a>單一 SAP 系統

先前顯示的內部部署基礎結構會透過 ExpressRoute 連線至 Azure。 ExpressRoute 線路會連線至 Enterprise Edge 路由器。 如需詳細資訊，請參閱 [ExpressRoute 技術概觀](../../../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 路由在建立後，將會連線至 Microsoft Azure 骨幹，而讓所有 Azure 區域都可供存取。

> [!NOTE] 
> 若要在 Azure 中執行 SAP 架構，請連線至距離 SAP 架構中的 Azure 區域最近的 Enterprise Edge 路由器。 「Azure 大型執行個體」戳記會透過專用 Enterprise Edge 路由器裝置連線，以將 Azure IaaS 中的 VM 與「大型執行個體」戳記之間的網路延遲降到最低。

裝載 SAP 應用程式執行個體之 VM 的虛擬網路閘道會連線至 ExpressRoute 線路。 相同的虛擬網路會連線至專門用來連接到大型執行個體戳記的個別 Enterprise Edge 路由器。

此系統是單一 SAP 系統的簡明範例。 SAP 應用程式層裝載於 Azure 中。 SAP HANA 資料庫執行於 SAP HANA on Azure (大型執行個體) 上。 我們的假設是虛擬網路閘道頻寬 2 Gbps 或 10 Gbps 輸送量並不代表瓶頸。

## <a name="multiple-sap-systems-or-large-sap-systems"></a>多個 SAP 系統或大型 SAP 系統

如果將多個 SAP 系統或大型 SAP 系統部署成連線至 SAP HANA on Azure (大型執行個體)，虛擬網路閘道的輸送量就可能成為瓶頸。 在這種情況下，請將應用程式層分成多個虛擬網路。 針對如下的情況，您也可以建立會連線至 HANA 大型執行個體的特殊虛擬網路：

- 直接從「HANA 大型執行個體」中的 HANA 執行個體備份到 Azure 中裝載 NFS 共用的 VM。
- 將大型備份或其他檔案從「HANA 大型執行個體」單位複製到在 Azure 中管理的磁碟空間。

使用個別的虛擬網路來裝載管理儲存體的 VM。 這種配置可避免在透過虛擬網路閘道 (此閘道可為執行 SAP 應用程式層的 VM 提供服務) 將大型檔案或資料從 HANA 大型執行個體傳輸到 Azure 時造成影響。 

讓網路架構更具彈性：

- 針對單一的較大 SAP 應用程式層，運用多個虛擬網路。
- 相較於將所部署的 SAP 系統結合在相同虛擬網路下的個別子網路中，為這些 SAP 系統中的每一個系統部署一個個別的虛擬網路。

 適用於 SAP HANA on Azure (大型執行個體) 的更有彈性網路架構：

![跨多個虛擬網路部署 SAP 應用程式層](./media/hana-overview-architecture/image4-networking-architecture.png)

此圖顯示透過多個虛擬網路進行部署的 SAP 應用程式層或元件。 此組態會在這些虛擬網路所裝載的應用程式之間進行通訊期間，產生無法避免的延遲額外負荷。 根據預設，在此組態中，位於不同虛擬網路中的 VM 之間的網路流量會透過 Enterprise Edge 路由器來傳送。 若要最佳化並縮短兩個虛擬網路之間的通訊延遲，做法是將相同區域內的虛擬網路對等互連。 即使這些虛擬網路在不同的訂用帳戶中，此方法仍適用。 透過虛擬網路對等互連，兩個不同虛擬網路中 VM 之間的通訊就可以使用 Azure 網路骨幹來彼此直接通訊。 此時會出現延遲狀況，如同 VM 位於相同虛擬網路中。 以透過 Azure 虛擬網路閘道器連線的 IP 位址範圍來定址的流量，將透過虛擬網路的個別虛擬網路閘道器進行路由傳送。 

如需關於虛擬網路對等互連的詳細資訊，請參閱[虛擬網路對等互連](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)。


## <a name="routing-in-azure"></a>Azure 中的路由

SAP HANA on Azure (大型執行個體) 有三項重要的網路路由考量：

* SAP HANA on Azure (大型執行個體) 只能透過 VM 和專用 ExpressRoute 連線來存取，而無法從內部部署環境直接存取。 從內部部署環境對 HANA 大型執行個體單位的直接存取 (如 Microsoft 所提供)，是無法立即執行的。 可轉移的路由會因為目前用於 SAP HANA 大型執行個體的 Azure 網路架構而受限。 有些系統管理用戶端及所有需要直接存取權的應用程式 (例如在內部部署環境中執行的 SAP Solution Manager) 會無法連線至 SAP HANA 資料庫。

* 如果您在兩個不同 Azure 區域中部署了 HANA 大型執行個體單位以供災害復原之用，則系統將會套用相同的暫時性路由限制。 換句話說，HANA 大型執行個體單位在一個區域 (例如美國西部) 內的 IP 位址不會路由至在另一個地區 (例如美國東部) 部署的HANA 大型執行個體單位。 這限制與跨區域使用 Azure 網路對等互連或交叉連接 ExpressRoute 線路 (將 HANA 大型執行個體單位連線至虛擬網路) 無關。 如需以圖形檢視，請參閱「在多個區域中使用 HANA 大型執行個體單位」一節中的圖表。 部署架構所隨附的這項限制，會禁止立即使用 HANA 系統複寫作為災害恢復功能。

* SAP HANA on Azure (大型執行個體) 單位會被指派一個 IP 位址，且該位址來自於您所提交的伺服器 IP 集區為址範圍。 如需詳細資料，請參閱 [Azure 上的 SAP HANA (大型執行個體) 基礎結構和連線](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。 透過 Azure 訂用帳戶和將虛擬網路連線至 HANA on Azure (大型執行個體) 的 ExpressRoute，即可存取此 IP 位址。 從該伺服器 IP 集區位址範圍指派的 IP 位址會直接指派給硬體單位。 它*不會*再經過 NAT 處理，因為這是此解決方案的第一次部署才會有的情況。 

> [!NOTE] 
> 若要克服前兩個清單項目中所述之暫時性路由的限制，請使用其他元件進行路由。 可用來克服這項限制的元件可能為：

> * 路由資料的反向 Proxy，往返方向皆有可能。 例如，搭配部署在 Azure 中作為虛擬防火牆/流量路由解決方案之「流量管理員」的 F5 BIG-IP、NGINX。
> * 在 Linux VM 中使用 [IPTables 規則](http://www.linuxhomenetworking.com/wiki/index.php/Quick_HOWTO_%3a_Ch14_%3a_Linux_Firewalls_Using_iptables#.Wkv6tI3rtaQ)，以啟用在內部部署位置和 HANA 大型執行個體單位之間的路由，或不同區域中 HANA 大型執行個體單位之間的路由。

> 請注意，Microsoft 並不提供與協力廠商網路設備或 IPTables 相關自定義解決方案的實作和支援。 必須由所使用元件的廠商或整合者提供支援。 

## <a name="internet-connectivity-of-hana-large-instance"></a>HANA 大型執行個體的網際網路連線
HANA 大型執行個體*無法*直接連線至網際網路。 此限制可能會使您的一些能力受到限制，例如，直接向 OS 廠商註冊 OS 映像的能力。 您可能需要使用本機 SUSE Linux Enterprise Server Subscription Management Tool 伺服器或 Red Hat Enterprise Linux Subscription Manager。

## <a name="data-encryption-between-vms-and-hana-large-instance"></a>VM 與 HANA 大型執行個體之間的資料加密
HANA 大型執行個體與 VM 之間的資料傳輸並未加密。 不過，您可以純粹針對 HANA DBMS 端與 JDBC/ODBC 型應用程式之間的交換，啟用流量加密。 如需詳細資訊，請參閱 [SAP 提供的這份文件](http://help-legacy.sap.com/saphelp_hanaplatform/helpdata/en/db/d3d887bb571014bf05ca887f897b99/content.htm?frameset=/en/dd/a2ae94bb571014a48fc3b22f8e919e/frameset.htm&current_toc=/en/de/ec02ebbb57101483bdf3194c301d2e/plain.htm&node_id=20&show_children=false)。

## <a name="use-hana-large-instance-units-in-multiple-regions"></a>在多個區域中使用 HANA 大型執行個體單位

除了災害復原之外，您可能還有其他理由促使您在多個 Azure 區域中部署 SAP HANA on Azure (大型執行個體)。 也許您想要從部署在這些區域之不同虛擬網路中的每個 VM 存取「HANA 大型執行個體」。 指派給不同 HANA 大型執行個體單位的 IP 位址，不會在透過閘道直接連線的虛擬網路以外傳播至執行個體。 因此，就會在虛擬網路設計中導入些微變更。 虛擬網路閘道可處理四個來自不同 Enterprise Edge 路由器的不同 ExpressRoute 線路。 每個連線至其中一個大型執行個體戳記的虛擬網路，都可以連線至另一個 Azure 區域中的大型執行個體戳記。

![連線至不同 Azure 區域中的 Azure 大型執行個體戳記的虛擬網路](./media/hana-overview-architecture/image8-multiple-regions.png)

此圖顯示兩個區域中的不同虛擬網路如何連線至兩個不同的 ExpressRoute 線路，而這些線路是用來連線至這兩個 Azure 區域中的 SAP HANA on Azure (大型執行個體)。 新導入的連線是矩形的紅線。 透過這些來自虛擬網路的連線，在其中一個虛擬網路中執行的 VM 便可存取部署在這兩個區域中每一個不同的 HANA 大型執行個體單位。 如上圖所示，這是假設您有兩條從內部部署環境連到這兩個 Azure 區域的 ExpressRoute 連線。 基於災害復原的用途，建議您使用此做法。

> [!IMPORTANT] 
> 如果您使用多個 ExpressRoute 線路，就應該在前面加上「AS 路徑」和使用「本機喜好 BGP」設定來確保流量路由正確。

**後續步驟**
- 請參閱 [SAP HANA (大型執行個體) 儲存體架構](hana-storage-architecture.md)