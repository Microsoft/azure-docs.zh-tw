---
title: Azure Stack 整合式系統的一般資料中心整合考量 | Microsoft Docs
description: 了解您可以執行哪些工作，來立即為資料中心網路與多節點 Azure Stack 的整合做規劃及準備。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/01/2018
ms.author: jeffgilb
ms.reviewer: wfayed
ms.openlocfilehash: 0c43b66a9d6210ea951af3fae5eca8bc6d47c3d9
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35261210"
---
# <a name="datacenter-integration-considerations-for-azure-stack-integrated-systems"></a>Azure Stack 整合式系統的一般資料中心整合考量
如果您對 Azure Stack 整合式系統有興趣，您應瞭解一些有關部署的重大規劃考量，及系統如何融入您的資料中心。 本文提供這些考量的高階概觀，協助您為 Azure Stack 多節點系統做出重要的基礎結構決策。 與您的 OEM 硬體廠商一起將 Azure Stack 部署到您的資料中心時，瞭解這些考量有所助益。  

> [!NOTE]
> 您只能向經過授權的硬體廠商購買 Azure Stack 多節點系統。 

若要部署 Azure Stack，您必須在部署開始之前，先將規劃資訊提供給解決方案提供者，才能讓程序快速且順暢地進行。 所需的資訊涵蓋網路功能、安全性和身分識別資訊，以及許多可能需要很多不同領域和決策制定者相關知識的重要決策。 因此，您必須與組織中多個小組的人員交涉，以確保您在部署開始前已備妥所有必要資訊。 在收集此資訊時，與您的硬體廠商洽談可能有所幫助，因為他們可能會提供有助於進行決策的建議。

在研究並收集必要資訊時，您可能必須對網路環境進行一些部署前的設定變更。 這可能包括保留 Azure Stack 解決方案的 IP 位址空間，設定您的路由器、交換器和防火牆，以便為新的 Azure Stack 解決方案交換器連線預作準備。 確定目前有主題領域專家可協助您進行規劃。

## <a name="capacity-planning-considerations"></a>容量規劃考量
評估是否要取得 Azure Stack 解決方案時，選擇硬體組態是必要的，因為這直接影響到 Azure Stack 解決方案的整體容量。 其中包括 CPU、記憶體密度、儲存體組態和整體解決方案規模 (例如伺服器的數目) 等典型選項。 不同於傳統的虛擬化解決方案，簡單地計算這些元件不適用於決定可用的容量。 第一個原因是 Azure Stack 的架構是解決方案本身就裝載基礎結構或管理元件。 第二個原因是解決方案的部分容量會為支援恢復功能而保留；透過將租用戶工作負載中斷情形降到最低的方式，來更新解決方案的軟體。 

[Azure Stack 容量規劃工具試算表](https://gallery.technet.microsoft.com/Azure-Stack-Capacity-24ccd822)可透過兩個方法協助您在規劃容量方面作出適當的決定：透過選取硬體供應項目和嘗試符合資源組合，或透過定義 Azure Stack 要執行的工作負載，來檢視可支援的可用硬體 SKU。 最後，此試算表是用來協助決定 Azure Stack 規劃和組態等相關項目的指南。 

不適合用試算表取代您自己的調查與分析。  Microsoft 對於試算表內所提供的資訊不做任何明示或暗示的聲明或保證。



## <a name="management-considerations"></a>管理考量
Azure Stack 是密封的系統，已從權限和網路觀點鎖定基礎結構。 會套用網路存取控制清單 (ACL) 來封鎖所有未經授權的連入流量，及基礎結構元件之間所有不必要的通訊。 這會使得未經授權的使用者難以存取系統。

對於每日的管理與作業，系統管理員存取基礎結構不受限制。 Azure Stack 操作員必須透過系統管理員入口網站或 Azure Resource Manager (透過 PowerShell 或 REST API) 管理系統。 其他管理工具 (例如 HYPER-V 管理員或容錯移轉叢集管理員) 不能存取系統。 為了協助保護系統，無法在 Azure Stack 基礎結構元件中安裝協力廠商軟體 (例如，代理程式)。 透過 PowerShell 或 REST API，可與外部的管理與安全性軟體建立互通性。

需要較高層級的存取權來針對無法透過警示中繼步驟解決的問題進行疑難排解時，您必須與「Microsoft 支援服務」合作。 透過支援服務，可提供一個方法來賦予對系統的暫時完整系統管理員存取權，以執行更進階的操作。 

## <a name="identity-considerations"></a>身分識別考量

### <a name="choose-identity-provider"></a>選擇識別提供者
您必須考量要使用於 Azure Stack 部署 (可能是 Azure AD 或 AD FS) 的識別提供者。 重新部署不含完整系統的部署之後，您無法切換識別提供者。 如果您不擁有該 Azure AD 帳戶，且使用雲端服務提供者提供的帳戶，又如果您決定切換提供者並使用不同的 Azure AD 帳戶，此時需要連絡解決方案提供者以重新部署解決方案，而您需要自行負擔相關費用。



您的識別提供者選項與租用戶虛擬機器、識別系統、其使用的帳戶、它們是否能加入 Active Directory 網域等等無關。這是分開的。

您可以在 [Azure Stack 整合式系統連線模型](.\azure-stack-connection-models.md) 一文中深入了解如何選擇識別提供者。

### <a name="ad-fs-and-graph-integration"></a>AD FS 和 Graph 整合
如果您選擇將 AD FS 作為識別提供者來部署 Azure Stack，您必須透過同盟信任來整合 Azure Stack 上的 AD FS 執行個體與現有的 AD FS 執行個體。 這可讓現有 Active Directory 樹系中的身分識別來驗證 Azure Stack 中的資源。

您也可以整合 Azure Stack 中的 Graph 服務與現有的 Active Directory。 這可讓您在 Azure Stack 中管理角色型存取控制 (RBAC)。 委派資源的存取權後，Graph 元件會使用 LDAP 通訊協定來查閱現有 Active Directory 樹系中的使用者帳戶。

下圖顯示整合的 AD FS 和 Graph 流量。
![顯示 AD FS 與 Graph 流量的圖表](media/azure-stack-datacenter-integration/ADFSIntegration.PNG)

## <a name="licensing-model"></a>授權模型
您必須決定要使用哪個授權模型。 可用選項需視您是否部署已連線至網際網路的 Azure Stack：
- 針對[已連線的部署](azure-stack-connected-deployment.md)，您可以選擇使用時付費或以容量為基礎的授權。 使用時付費需要連線至 Azure 來報告使用量，然後透過 Azure 商務計費。 
- 如果您[部署與網際網路中斷連線的 Azure Stack](azure-stack-disconnected-deployment.md)，只能使用以容量為基礎的授權。 

如需授權模式的詳細資訊，請參閱 [Microsoft Azure Stack 封裝和定價](https://azure.microsoft.com/mediahandler/files/resourcefiles/5bc3f30c-cd57-4513-989e-056325eb95e1/Azure-Stack-packaging-and-pricing-datasheet.pdf)。


## <a name="naming-decisions"></a>命名決策

您將需要思考如何規劃 Azure Stack 命名空間，尤其是區域名稱和外部網域名稱。 公開端點之 Azure Stack 部署的外部完整網域名稱 (FQDN) 以下兩個名稱的組合：&lt;*region*&gt;.&lt;*fqdn*&gt;。 例如，*east.cloud.fabrikam.com*。 在此範例中，Azure Stack 入口網站將位於下列 URL：

- https://portal.east.cloud.fabrikam.com
- https://adminportal.east.cloud.fabrikam.com

> [!IMPORTANT]
> 您為 Azure Stack 部署選擇的區域名稱必須是唯一的，且將會在入口網站位址中顯示。 

下表摘要這些網域命名決策。

| Name | 說明 | 
| -------- | ------------- | 
|區域名稱 | 您的第一個 Azure Stack 區域名稱。 這個名稱會用於 Azure Stack 管理的公用虛擬 IP 位址 (VIP) 之 FQDN 的一部分。 一般而言，區域名稱會是資料中心位置之類的實體位置識別碼。 | 
| 外部網域名稱 | 含外部 VIP 之端點的網域名稱系統 (DNS) 區域名稱。 針對這些公用 VIP 在 FQDN 中使用。 | 
| 私人 (內部) 網域名稱 | 在 Azure Stack 上為基礎結構管理建立的網域 (和內部 DNS 區域) 的名稱。 
| | |

## <a name="certificate-requirements"></a>憑證需求

針對部署，您將需要為公開端點提供安全通訊端層 (SSL) 憑證。 概括而言，憑證具有下列需求：

- 您可以使用單一萬用字元憑證或以使用一組專用的憑證，並只針對端點 (例如儲存體和金鑰保存庫) 使用萬用字元。
- 憑證可以由大眾信任的憑證授權單位 (CA) 或由客戶管理的 CA 發出。

如需有關部署 Azure Stack 時需要哪些 PKI 憑證以及如何取得這些憑證的詳細資訊，請參閱 [Azure Stack 公開金鑰基礎結構憑證需求](azure-stack-pki-certs.md)。  


> [!IMPORTANT]
> 所提供的 PKI 憑證資訊應作為一般指引。 取得 Azure Stack 的任何 PKI 憑證之前，請與您的 OEM 硬體合作夥伴接洽。 它們將提供更詳細的憑證指引和需求。


## <a name="time-synchronization"></a>時間同步處理
您必須選擇用來同步處理 Azure Stack 的特定時間伺服器。  時間符號化對於 Azure Stack 及其基礎結構角色而言很重要，因為它用於產生 Kerberos 票證，而這些票證用於內部服務的彼此驗證。

您必須指定時間同步處理伺服器的 IP，雖然基礎結構中的大部分元件都可以解析 URL，但有些元件只支援 IP 位址。 如果您使用已中斷連線的部署選項，您必須指定您公司網路上的時間伺服器，且確定您可從 Azure Stack 中的基礎結構網路觸達該伺服器。

## <a name="connect-azure-stack-to-azure"></a>將 Azure Stack 連線至 Azure

對於混合式雲端情節，您必須規劃如何將 Azure Stack 連線至 Azure。 支援兩種方法將 Azure Stack 中的虛擬網路連線至 Azure 中的虛擬網路： 
- **網站間**。 透過 IPsec (IKE v1 和 IKE v2) 的虛擬私人網路 (VPN) 連線。 此類型的連線需要 VPN 裝置或路由及遠端存取服務 (RRAS)。 如需 Azure 中 VPN 閘道的詳細資訊，請參閱[關於 VPN 閘道](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways)。 會透過這個通道的通訊加密，並且是安全的。 不過，頻寬會受限於通道的最大輸送量 (100-200 Mbps)。
- **輸出 NAT**。 根據預設，Azure Stack 中的所有虛擬機器皆可透過輸出 NAT 連線到外部網路。 Azure Stack 中建立的每個虛擬網路會被指派一個公用 IP 位址。 無論虛擬機器被直接指派公用 IP 位址，或是位於含公用 IP 位址的負載平衡器後方，它皆可使用虛擬網路的 VIP，透過輸出 NAT 進行外部存取。 這只適用於虛擬機器所起始，並針對外部網路 (網際網路或內部網路) 的通訊。 它不能與來自外部的虛擬機器進行通訊。

### <a name="hybrid-connectivity-options"></a>混合式連線能力選項

對於混合式連線能力，務必考慮您想要提供何種部署及其部署位置。 您必須考量是否需要隔離每個租用戶的網路流量，以及是否也有內部網路或網際網路部署。

- **單一租用戶 Azure Stack**。 Azure Stack 部署看起來 (至少從網路觀點) 就像是一個租用戶。 可能有許多租用戶訂用帳戶，但就像任何內部網路服務，所有流量都會透過相同的網路傳輸。 一個訂用帳戶會與另一個訂用帳戶透過相同的網路連線傳輸網路流量，並且不需要透過加密通道隔離。

- **多租用戶 Azure Stack**。 一個 Azure Stack 部署，其中每個租用戶訂用帳戶流量與 Azure Stack 外部的網路繫結，必須將其與其他租用戶的網路流量隔離。
 
- **內部網路部署**。 位於公司內部網路上 Azure Stack 部署，一般位於私人 IP 位址空間與一或多個防火牆之後。 公用 IP 位址並非真正公用，因為它們無法透過公用網際網路直接路由。

- **網際網路部署**。 一個 Azure Stack 部署，其連線到公用網際網路，並針對公用 IP 位址使用網際網路的可路由公用 IP 位址。 部署仍可放在防火牆後面，但公用 VIP 範圍可從公用網際網路和 Azure 直接連線。
 
下表摘要說明混合式連線情節，包含優點、缺點和使用案例。

| 案例 | 連線方法 | 優點 | 缺點 | 適用對象 |
| -- | -- | --| -- | --|
| 單一租用戶 Azure Stack，內部網路部署 | 輸出 NAT | 用於更快傳輸的更佳頻寬。 簡單實作；沒有必要的閘道。 | 不會加密流量；堆疊外部無隔離或加密。 | 同樣信任所有租用戶的企業部署。<br><br>具有至 Azure 的 Azure ExpressRoute 電路的企業。 |
| 多租用戶 Azure Stack，內部網路部署 | 站對站 VPN | 從租用戶 VNet 到目的地的流量是安全的。 | 頻寬會受限於站對站 VPN 通道。<br><br>需要虛擬網路上的閘道和目的地網路上的 VPN 裝置。 | 必須避免其他租用戶接觸其部分租用戶流量的企業部署。 |
| 單一租用戶 Azure Stack，網際網路部署 | 輸出 NAT | 用於更快傳輸的更佳頻寬。 | 不會加密流量；堆疊外部無隔離或加密。 | 裝載情節，其中租用戶取得其自己的 Azure Stack 部署和 Azure Stack 環境的專屬電路。 例如，ExpressRoute 和多重通訊協定標籤交換 (MPLS)。
| 多租用戶 Azure Stack，網際網路部署 | 站對站 VPN | 從租用戶 VNet 到目的地的流量是安全的。 | 頻寬會受限於站對站 VPN 通道。<br><br>需要虛擬網路上的閘道和目的地網路上的 VPN 裝置。 | 裝載情節，其中的提供者要提供多租用戶雲端，讓租用戶彼此不信任且必須加密流量。
|  |  |  |  |  |

### <a name="using-expressroute"></a>使用 ExpressRoute

您可以針對單一租用戶內部網路和多租用戶情節，透過 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 將 Azure Stack 連線至 Azure。 您將需要透過[連線提供者](https://docs.microsoft.com/azure/expressroute/expressroute-locations)佈建的 ExpressRoute 電路。

下圖顯示單一租用戶情節的 ExpressRoute (其中「客戶的連線」是 ExpressRoute 電路)。

![顯示單一租用戶 ExpressRoute 情節的圖表](media/azure-stack-datacenter-integration/ExpressRouteSingleTenant.PNG)

下圖顯示多租用戶情節的 ExpressRoute。

![顯示多租用戶 ExpressRoute 情節的圖表](media/azure-stack-datacenter-integration/ExpressRouteMultiTenant.PNG)

## <a name="external-monitoring"></a>外部監視
若要從您的 Azure Stack 部署和裝置取得所有警示的單一檢視，並針對票證將警示整合至現有的 IT 服務管理工作流程，您可以[整合 Azure Stack 與外部的資料中心監視解決方案](azure-stack-integrate-monitor.md)。

Azure Stack 解決方案隨附的硬體生命週期主機是 Azure Stack 外部的電腦，其執行 OEM 硬體廠商為硬體提供的管理工具。 您可以使用這些工具，或與資料中心內的現有監視解決方案直接整合的其他解決方案。

下表摘要列出目前的可用選項清單。

| 領域 | 外部監視解決方案 |
| -- | -- |
| Azure Stack 軟體 | [Operations Manager 的 Azure Stack 管理組件](https://azure.microsoft.com/blog/management-pack-for-microsoft-azure-stack-now-available/) \(英文\)<br>[Nagios 外掛程式](https://exchange.nagios.org/directory/Plugins/Cloud/Monitoring-AzureStack-Alerts/details) \(英文\)<br>REST 型 API 呼叫 | 
| 實體伺服器 (透過 IPMI 的 BMC) | OEM 硬體 - Operations Manager 廠商管理組件<br>OEM 硬體廠商提供的解決方案<br>硬體廠商 Nagios 外掛程式 | OEM 合作夥伴支援的監視解決方案 (包含) | 
| 網路裝置 (SNMP) | Operations Manager 網路裝置探索<br>OEM 硬體廠商提供的解決方案<br>Nagios 交換器外掛程式 |
| 租用戶訂用帳戶健康情況監視 | [System Center Management Pack for Windows Azure](https://www.microsoft.com/download/details.aspx?id=50013) | 
|  |  | 

請注意下列需求：
- 您使用的解決方案必須是無代理程式。 您無法在 Azure Stack 元件內部安裝協力廠商代理程式。 
- 如果您想要使用 System Center Operations Manager，需要 Operations Manager 2012 R2 或 Operations Manager 2016。

## <a name="backup-and-disaster-recovery"></a>備份和災害復原

規劃備份及災害復原牽涉到規劃這兩個基礎 Azure Stack 基礎結構 (用來裝載 IaaS 虛擬機器和 PaaS 服務)，以及租用戶應用程式和資料。 您必須分開規劃它們。

### <a name="protect-infrastructure-components"></a>保護基礎結構元件

您可以[備份 Azure Stack](azure-stack-backup-back-up-azure-stack.md)基礎結構元件到您指定的 SMB 共用：

- 您在現有的 Windows 型檔案伺服器或協力廠商裝置上需要有外部 SMB 檔案共用。
- 您應該將這個相同的共用使用於網路交換器與硬體生命週期主機的備份。 OEM 硬體廠商將協助提供這些元件的備份和還原指引，因為這些是 Azure Stack 外部的作業。 您負責根據 OEM 廠商的建議來執行備份工作流程。

如果發生重大資料遺失，您可以使用基礎結構來重新植入部署資料，例如部署輸入和識別碼、服務帳戶、CA 根憑證、同盟的資源 (在中斷連線的部署中)、計劃、供應項目、訂用帳戶、配額、RBAC 原則與角色指派，和金鑰保存庫密碼。
 
### <a name="protect-tenant-applications-on-iaas-virtual-machines"></a>保護 IaaS 虛擬機器上的租用戶應用程式

Azure Stack 不會備份租用戶應用程式和資料。 您必須針對 Azure Stack 的外部目標規劃備份及災害復原保護。 租用戶保護是租用戶驅動的活動。 對於 IaaS 虛擬機器，可在來賓技術中使用租用戶來保護檔案資料夾、應用程式資料和系統狀態。 不過，作為企業或服務提供者，您可能想要在相同資料中心或雲端外部提供備份和復原解決方案。

若要備份 Linux 或 Windows IaaS 虛擬機器，您必須使用含來賓作業系統存取權的備份產品來保護檔案、資料夾、作業系統狀態和應用程式資料。 您可以使用 Azure 備份，System Center Data Center Protection Manager 或支援的協力廠商產品。

若要將資料複寫到次要位置並協調應用程式在發生災難時容錯移轉，您可以使用 Azure Site Recovery 或支援的協力廠商產品。 (在初始版本的整合系統中，Azure Site Recovery 不會支援容錯回復。 不過，您可以透過手動程序完成容錯回復。)此外，支援原生複寫 (如 Microsoft SQL Server) 的應用程式可將資料複寫到正在執行應用程式的另一個位置。

> [!IMPORTANT]
> 在初始版本的整合系統中，我們將支援在 IaaS 虛擬機器的來賓層級上工作的保護技術。 您無法在基礎的基礎結構伺服器上安裝代理程式。

## <a name="learn-more"></a>深入了解

- 如需使用案例、購買、合作夥伴和 OEM 硬體廠商的詳細資訊，請參閱 [Azure Stack](https://azure.microsoft.com/overview/azure-stack/) 產品頁面。
- 如需 Azure Stack 整合系統的藍圖和地理可用性的詳細資訊，請參閱白皮書：[Azure Stack：Azure 的延伸模組](https://azure.microsoft.com/resources/azure-stack-an-extension-of-azure/)。 

## <a name="next-steps"></a>後續步驟
[Azure Stack 部署連線模型](azure-stack-connection-models.md)
