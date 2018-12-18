---
title: Azure SQL Database 安全性功能
description: 本文提供 Azure SQL Database 如何保護 Azure 中客戶資料的一般描述。
services: security
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: TomSh
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/28/2018
ms.author: terrylan
ms.openlocfilehash: cce1ff1102c42bd1627caeba7b2c86432b228607
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39170849"
---
# <a name="azure-sql-database-security-features"></a>Azure SQL Database 安全性功能    
Azure SQL Database 在 Azure 中提供關聯式資料庫服務。 為了保護客戶資料，並提供強大的安全性功能，以滿足提供客戶對於關聯式資料庫服務的期待，SQL Database 有一套自己的安全性功能。 這些功能都是根據從 Azure 繼承的控制項為基礎而建置。

## <a name="security-capabilities"></a>安全性功能

### <a name="usage-of-the-tds-protocol"></a>TDS 通訊協定的使用方式
Azure SQL Database 僅支援表格式資料流 (TDS) 通訊協定，該通訊協定會要求只能透過預設連接埠 TCP/1433 存取資料庫。

### <a name="azure-sql-database-firewall"></a>Azure SQL Database 防火牆
為了協助保護客戶資料，Azure SQL Database 會包含防火牆功能，該功能預設可防止對於 SQL Database 伺服器的所有存取，如下所示。

![Azure SQL Database 防火牆][1]

閘道防火牆可限制位址，讓客戶有精細的控制權，可以指定可接受 IP 位址的範圍。 此防火牆會根據每一個要求的原始 IP 位址來授與存取權。

客戶可以使用管理入口網站，或使用 Azure SQL Database 管理 REST API 以程式設計方式來完成防火牆組態。 Azure SQL Database 閘道防火牆預設會防止所有客戶以 TDS 存取 Azure SQL 資料庫執行個體。 客戶必須使用存取控制清單 (ACL) 來設定存取，以依據來源和目的地網際網路位址、通訊協定和連接埠號碼允許 Azure SQL Database 連線。

### <a name="dosguard"></a>DoSGuard
稱為 DoSGuard 的 SQL Database 閘道服務會減少拒絕服務的攻擊 (DoS)。 DoSGuard 會主動追蹤來自 IP 位址的失敗登入。 如果在一段時間內特定 IP 位址有多個失敗登入，該 IP 位址會遭到封鎖，而無法在預先定義的時段存取服務中的任何資源。

此外，Azure SQL Database 閘道會執行下列項目：

- 安全通道功能交涉，在連線至資料庫伺服器時實作 TDS FIPS 140-2 經驗證的加密連線。
- 在接受來自用戶端連線的同時進行具狀態 TDS 封包檢查。 閘道會驗證連線資訊，並且根據連接字串中指定的資料庫名稱，將 TDS 封包傳遞至適當的實體伺服器。

Azure SQL Database 供應項目的網路安全性首要原則，是只允許讓服務得以運作的必要連線和通訊。 其他所有連接埠、通訊協定及連線預設都會遭到封鎖。 虛擬區域網路 (VLAN) 和 ACL 是用來依據來源和目的地網路、通訊協定和連接埠號碼限制網路通訊。

已核准用來實作網路型 ACL 的機制包括：路由器和負載平衡器上的 ACL。 這些機制是由 Azure 網路、客體 VM 防火牆及 Azure SQL Database 閘道防火牆規則 (均由客戶設定) 進行管理。

## <a name="data-segregation-and-customer-isolation"></a>資料隔離和客戶隔離
Azure 生產網路經過結構化，可公開存取的系統元件會從內部資源中隔離出來。 提供公眾對應 Azure 入口網站和基礎 Azure 虛擬基礎結構 (客戶應用程式執行個體和客戶資料位於該位置) 存取權的 Web 伺服器之間有實體和邏輯界限存在。

所有可公開存取的資訊在 Azure 生產網路中都會受到控制。 生產網路受限於雙因素驗證和界限保護機制，會使用上一節中所述的防火牆和安全性功能集，以及使用下一節中所示的資料隔離函式。

### <a name="unauthorized-systems-and-isolation-of-the-fc"></a>未經授權的系統和隔離 FC
由於網狀架構控制器 FC 是 Azure Fabric 的中央協調器，因此具有大量控制項以減輕威脅，尤其是來自客戶應用程式內可能遭盜用 FA 的威脅。 FC 無法辨識其裝置資訊 (例如，MAC 位址) 未預先載入 FC 的任何硬體。 FC 上的 DHCP 伺服器已設定它們願意開機的節點 MAC 位址清單。 未經授權的系統即使連線到網狀架構，也不會併入網狀架構清查，因此不會連線到網狀架構清查內的任何系統，也不會得到授權與網狀架構清查內的任何系統通訊。 這樣可以降低未經授權的系統與 FC 通訊，以及取得 VLAN 和 Azure 存取權的風險。

### <a name="vlan-isolation"></a>VLAN 隔離
Azure 生產網路以邏輯方式分成三個主要 VLAN：

- 主要 VLAN：互連未受信任的客戶節點。
- FC VLAN：包含受信任的 FC 和支援系統。
- 裝置 VLAN：包含受信任的網路和其他基礎結構裝置。

### <a name="packet-filtering"></a>封包篩選
IP 篩選器和根 OS 與客體 OS 上實作的軟體防火牆，會強制執行連線限制，並且防止虛擬機器之間未經授權的流量。

### <a name="hypervisor-root-os-and-guest-vms"></a>Hypervisor、根 OS 和客體 VM
隔離根 OS 與客體 VM，以及客體 VM 之間的彼此隔離，是由 Hypervisor 和根 OS 所管理。

### <a name="types-of-rules-on-firewalls"></a>防火牆上的規則類型
規則定義為：

{安全性回應中心 (Src) IP、Src 連接埠，目的地 IP、目的地連接埠、目的地通訊協定，輸入/輸出、具狀態/無狀態、具狀態流程逾時}。

同步閒置字元 (SYN) 封包只有在任何一個規則允許時，才可以輸入或輸出。 針對 TCP，Azure 會使用無狀態規則，原則是只允許所有非 SYN 封包輸入或輸出 VM。 安全性前提是任何主機堆疊具有彈性，如果它先前沒有見過 SYN 封包，則略過非 SYN。 TCP 通訊協定本身具狀態，與無狀態 SYN 型規則搭配，以達成具狀態實作的整體行為。

針對使用者資料包通訊協定 (UDP)，Azure 會使用具狀態規則。 每次 UDP 封包符合規則時，就會在另一個方向建立反向流程。 此流程有內建的逾時。

客戶須負責在 Azure 所提供功能的基礎上，設定自己的防火牆。 在這裡客戶就能夠定義輸入和輸出流量的規則。

### <a name="production-configuration-management"></a>生產組態管理
標準安全組態是由 Azure 和 Azure SQL Database 中的個別作業小組維護。 系統會記載對生產系統的所有組態變更，並且透過中央追蹤系統追蹤。 軟體和硬體變更是透過中央追蹤系統追蹤的。 與 ACL 相關的網路變更是使用 ACL 管理服務來追蹤的。

對 Azure 的所有組態變更是在預備環境中開發及測試，然後在生產環境中部署。 軟體組建檢閱是測試的一部分。 安全性和隱私權檢查是項目檢查清單準則的一部分。 變更會由個別部署小組以排程間隔進行部署。 版本在部署到生產環境之前，會由個別部署小組人員檢閱及簽核。

系統會監視變更是否成功。 在失敗案例中，變更會回復為其先前狀態，或者經由指定人員的核准，部署 Hotfix 以解決失敗。 Source Depot、Git、TFS、Master Data Services (MDS)、執行器、Azure 安全性監視、FC 和 WinFabric 平台，是用來集中管理、套用及確認 Azure 虛擬環境中的組態設定。

同樣地，硬體和網路變更也有既定的驗證步驟，來評估是否遵循組建需求。 這些版本是透過跨堆疊個別群組的協調變更諮詢委員會 (CAB)，進行檢閱及授權。

## <a name="next-steps"></a>後續步驟
若要深入了解 Microsoft 為保護 Azure 基礎結構執行了哪些動作，請參閱：

- [Azure 設備、廠房以及實體安全性](azure-physical-security.md)
- [Azure 基礎結構可用性](azure-infrastructure-availability.md)
- [Azure 資訊系統元件和界限](azure-infrastructure-components.md)
- [Azure 網路架構](azure-infrastructure-network.md)
- [Azure 生產網路](azure-production-network.md)
- [Azure 生產環境運作與管理](azure-infrastructure-operations.md)
- [Azure 基礎結構監視](azure-infrastructure-monitoring.md)
- [Azure 基礎結構完整性](azure-infrastructure-integrity.md)
- [Azure 客戶資料保護](azure-protection-of-customer-data.md)

<!--Image references-->
[1]: ./media/azure-infrastructure-sql/sql-database-firewall.png
