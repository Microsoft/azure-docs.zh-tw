---
title: 常見問題 - 使用 Azure Site Recovery 進行 VMware 到 Azure 的複寫 | Microsoft Docs
description: 本文摘錄了您使用 Azure Site Recovery 將內部部署 VMware VM 複寫至 Azure 時的常見問題
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.date: 07/19/2018
ms.topic: conceptual
ms.author: raynew
ms.openlocfilehash: 487e0c763ca4b247f1818b2beaf3282734fc4e27
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49388437"
---
# <a name="common-questions---vmware-to-azure-replication"></a>常見問題 - VMware 到 Azure 的複寫

本文將解答我們將內部部署 VMware VM 複寫至 Azure 時的常見問題。 如果您在閱讀本文後有問題，請將問題張貼在 [Azure 復原服務論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=hypervrecovmgr)。


## <a name="general"></a>一般
### <a name="how-is-site-recovery-priced"></a>Site Recovery 是如何定價的？
請檢閱[Azure Site Recovery 定價](https://azure.microsoft.com/pricing/details/site-recovery/)詳細資料。

### <a name="how-do-i-pay-for-azure-vms"></a>如何支付 Azure VM 費用？
在複寫期間，資料會複寫至 Azure 儲存體，您無須為任何 VM 變更付費。 當您容錯移轉到 Azure 時，Site Recovery 會自動建立 Azure IaaS 虛擬機器。 之後就會針對您在 Azure 中取用的運算資源進行計費。

### <a name="what-can-i-do-with-vmware-to-azure-replication"></a>VMware 到 Azure 的複寫有何功能？
- **災害復原**：您可以設定完整的災害復原。 在此案例中，您會將內部部署 VMware VM 複寫至 Azure 儲存體。 其後，如果您的內部部署基礎結構無法使用，您即可進行容錯移轉至 Azure。 當您執行容錯移轉時，系統會使用複寫的資料建立 Azure VM。 您可以在 Azure VM 上存取應用程式和工作負載，直到您的內部部署資料中心恢復可用性為止。 接著，您可以從 Azure 容錯回復至內部部署網站。
- **移轉**：您可以使用 Site Recovery 將內部部署 VMware VM 移轉至 Azure。 在此案例中，您會將內部部署 VMware VM 複寫至 Azure 儲存體。 然後，您會從內部部署環境容錯移轉至 Azure。 容錯移轉之後，您的應用程式和工作負載將可在 Azure VM 上使用及執行。



## <a name="azure"></a>Azure
### <a name="what-do-i-need-in-azure"></a>在 Azure 中需要什麼？
您需要 Azure 訂用帳戶、復原服務保存庫、儲存體帳戶和虛擬網路。 保存庫、儲存體帳戶和網路必須位於相同的區域中。

### <a name="what-azure-storage-account-do-i-need"></a>我需要怎樣的 Azure 儲存體帳戶？
您需要 LRS 或 GRS 儲存體帳戶。 我們建議使用 GRS，以便在發生區域性停電或無法復原主要區域時，能夠恢復資料。 可支援進階儲存體。

### <a name="does-my-azure-account-need-permissions-to-create-vms"></a>我的 Azure 帳戶是否需要建立 VM 的權限？
如果您是訂用帳戶管理員，則會具有所需的複寫權限。 如果您不是，則需要必要權限才能在您設定 Site Reocvery 時指定的資源群組和虛擬網路中建立 Azure VM，且需要寫入至所選儲存體帳戶的權限。 [深入了解](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines)。



## <a name="on-premises"></a>內部部署

### <a name="what-do-i-need-on-premises"></a>我的內部部署環境需要什麼？
在內部部署環境中，您需要安裝在單一 VMware VM 上的 Site Recovery 元件。 您也需要至少有一個 ESXi 主機的 VMware 基礎結構，此外也建議要有 vCenter 伺服器。 此外，您需要一或多個要複寫的 VMware VM。 [深入了解](vmware-azure-architecture.md) VMware 到 Azure 的架構。

可利用以下兩種方式之一部署內部部署設定伺服器

1. 使用預先安裝設定伺服器的虛擬機器範本進行部署。 [請在這裡閱讀更多資訊](vmware-azure-tutorial.md#download-the-vm-template)。
2. 在您選擇的 Windows Server 2016 機器上使用設定進行部署。 [請在這裡閱讀更多資訊](physical-azure-disaster-recovery.md#set-up-the-source-environment)。

若要探索在您自己的 Windows Server 機器上部署設定伺服器的入門步驟，請在啟用保護的保護目標中，選擇**至 Azure > 未虛擬化/其他**。

### <a name="where-do-on-premises-vms-replicate-to"></a>內部部署 VM 會複寫到何處？
資料會複寫到 Azure 儲存體。 當您容錯移轉時，Site Recovery 會從儲存體帳戶自動建立 Azure VM。

### <a name="what-apps-can-i-replicate"></a>我可以複寫哪些應用程式？
您可以複寫符合[複寫需求](vmware-physical-azure-support-matrix.md##replicated-machines)的 VMware VM 上執行的任何應用程式或工作負載。 Site Recovery 支援應用程式感知複寫，可讓應用程式容錯移轉及容錯回復至智慧型狀態。 Site Recovery 除了與 Microsoft 應用程式 (例如 SharePoint、Exchange、Dynamics、SQL Server 及 Active Directory) 整合之外，還與產業龍頭 (包括 Oracle、SAP、IBM 及 Red Hat) 密切合作。 [深入了解](site-recovery-workload.md) 工作負載保護。

### <a name="can-i-replicate-to-azure-with-a-site-to-site-vpn"></a>是否可透過站對站 VPN 複寫至 Azure？
Site Recovery 可透過公用端點將資料從內部部署環境複寫至 Azure 儲存體，或使用 ExpressRoute 公用對等互連進行複寫。 不支援透過站對站 VPN 網路的複寫。

### <a name="can-i-replicate-to-azure-with-expressroute"></a>是否可透過 ExpressRoute 複寫至 Azure？
是的，ExpressRoute 可用來將 VM 複寫至 Azure。 Site Recovery 可透過公用端點將資料複寫到 Azure 儲存體帳戶，而您必須設定 Site Recovery 複寫所需的[公用對等互連](../expressroute/expressroute-circuit-peerings.md#azure-public-peering)。 VM 容錯移轉至 Azure 虛擬網路之後，您可以使用[私人對等互連](../expressroute/expressroute-circuit-peerings.md#azure-private-peering)加以存取。


### <a name="why-cant-i-replicate-over-vpn"></a>為何我無法透過 VPN 進行複寫？

在複寫至 Azure 時，複寫流量會送到 Azure 儲存體帳戶的公用端點，因此您只能使用公用網際網路透過 ExpressRoute (公用對等互連) 進行複寫，VPN 無法運作。



## <a name="what-are-the-replicated-vm-requirements"></a>複寫的 VM 有何需求？

若要進行複寫，VMware VM 必須執行支援的作業系統。 此外，VM 必須符合 Azure VM 的需求。 請在[支援矩陣](vmware-physical-azure-support-matrix.md##replicated-machines)中深入了解相關資訊。

## <a name="how-often-can-i-replicate-to-azure"></a>複寫到 Azure 的頻率為何？
從 VMware VM 到 Azure 的複寫會持續執行。

## <a name="can-i-extend-replication"></a>我可以延伸複寫嗎？
不支援延伸的或鏈結的複寫。 請在 [意見反應論壇](http://feedback.azure.com/forums/256299-site-recovery/suggestions/6097959-support-for-exisiting-extended-replication)中提出這項功能的要求。

## <a name="can-i-do-an-offline-initial-replication"></a>是否可執行離線初始複寫？
不支援此做法。 請在 [意見反應論壇](http://feedback.azure.com/forums/256299-site-recovery/suggestions/6227386-support-for-offline-replication-data-transfer-from)中提出這項功能的要求。

### <a name="can-i-exclude-disks"></a>是否可排除磁碟嗎？
是的，您可以將磁碟排除於複寫外。

### <a name="can-i-replicate-vms-with-dynamic-disks"></a>是否可使用動態磁碟來複寫 VM？
動態磁碟可以複寫。 作業系統磁碟必須是基本磁碟。

### <a name="if-i-use-replication-groups-for-multi-vm-consistency-can-i-add-a-new-vm-to-an-existing-replication-group"></a>如果我使用複寫群組以獲取多 VM 一致性，我是否可將新的 VM 新增至現有的複寫群組？
是，您可以將新的 VM 新增至現有的複寫群組，為這些 VM 啟用複寫。 您無法在複寫起始後將 VM 新增至現有的複寫群組，且無法為現有的 VM 建立複寫群組。

### <a name="can-i-modify-vms-that-are-replicating-by-adding-or-resizing-disks"></a>我是否可藉由新增磁碟或調整其大小來修改要複寫的 VM？

對於從 VMware 到 Azure 的複寫，您可以修改磁碟大小。 如果您想要新增磁碟，您必須新增磁碟，並重新啟用對 VM 的保護。

## <a name="configuration-server"></a>組態伺服器

### <a name="what-does-the-configuration-server-do"></a>組態伺服器有何功用？
組態伺服器會執行內部部署 Site Recovery 元件，包括：
- 設定伺服器，會協調內部部署與 Azure 之間的通訊，以及管理資料複寫。
- 處理序伺服器，可作為複寫閘道。 它會接收複寫資料、使用快取、壓縮和加密加以最佳化，然後將其傳送至 Azure 儲存體。處理序伺服器也會在您要複寫的 VM 上安裝行動服務，並且在內部部署 VMware VM 上執行自動探索。
- 主要目標伺服器，會在從 Azure 容錯回復期間處理複寫資料。

### <a name="where-do-i-set-up-the-configuration-server"></a>我應在何處設定組態伺服器？
您需要具有高可用性的單一內部部署 VMware VM，供組態伺服器使用。

### <a name="what-are-the-requirements-for-the-configuration-server"></a>組態伺服器有何需求？

請檢閱[必要條件](vmware-azure-deploy-configuration-server.md#prerequisites)。

### <a name="can-i-manually-set-up-the-configuration-server-instead-of-using-a-template"></a>是否可以手動設定組態伺服器，而不要使用範本？
建議您使用最新版的 OVF 範本來[建立組態伺服器 VM](vmware-azure-deploy-configuration-server.md)。 如果您因故無法這麼做 (例如您沒有 VMware 伺服器的存取權)，您可以從入口網站[下載統一安裝檔案](physical-azure-set-up-source.md)，並在 VM 上執行。

### <a name="can-a-configuration-server-replicate-to-more-than-one-region"></a>組態伺服器是否可複寫到多個區域？
否。 若要這麼做，您必須在每個區域中設定組態伺服器。

### <a name="can-i-host-a-configuration-server-in-azure"></a>是否可將組態伺服器裝載在 Azure 中？
雖然可行，但執行組態伺服器的 Azure VM 將需要與您的內部部署 VMware 基礎結構和 VM 通訊。 這會加入延遲，並影響進行中的複寫。


### <a name="where-can-i-get-the-latest-version-of-the-configuration-server-template"></a>哪裡可以取得最新版的組態伺服器範本？
請從 [Microsoft 下載中心](https://aka.ms/asrconfigurationserver)下載最新版本。

### <a name="how-do-i-update-the-configuration-server"></a>如何更新組態伺服器？
您可以安裝更新彙總套件。 您可以在 [Wiki 更新頁面](https://social.technet.microsoft.com/wiki/contents/articles/38544.azure-site-recovery-service-updates.aspx)中找到最新的更新資訊。

### <a name="should-i-backup-the-deployed-configuration-server"></a>我是否應該備份已部署的組態伺服器？
建議對於組態伺服器進行定期排程的備份。 為了成功容錯回復，進行容錯回復的虛擬機器必須存在於組態伺服器資料庫，而且組態伺服器必須執行中且處於連線狀態。 您可在[此處](vmware-azure-manage-configuration-server.md)深入了解常見組態伺服器管理工作。

## <a name="mobility-service"></a>行動服務

### <a name="where-can-i-find-the-mobility-service-installers"></a>哪裡可以找到行動服務安裝程式？
安裝程式存放在組態伺服器的 **%ProgramData%\ASR\home\svsystems\pushinstallsvc\repository** 資料夾中。

## <a name="how-do-i-install-the-mobility-service"></a>如何安裝行動服務？
您可以使用[推送安裝](vmware-azure-install-mobility-service.md#install-mobility-service-by-push-installation-from-azure-site-recovery)，安裝在您要複寫的每個 VM 上，或從 [UI](vmware-azure-install-mobility-service.md#install-mobility-service-manually-by-using-the-gui) 或[使用 PowerShell](vmware-azure-install-mobility-service.md#install-mobility-service-manually-at-a-command-prompt) 手動安裝。 或者，您可以使用 [System Center Configuration Manager](vmware-azure-mobility-install-configuration-mgr.md) 或 [Azure 自動化和 DSC](vmware-azure-mobility-deploy-automation-dsc.md) 之類的部署工具來安裝。



## <a name="security"></a>安全性

### <a name="what-access-does-site-recovery-need-to-vmware-servers"></a>Site Recovery 需要有 VMware 伺服器的哪些存取權？
Site Recovery 需要存取 VMware 伺服器才能：

- 設定執行組態伺服器與其他內部部署 Site Recovery 元件的 VMware VM。 [深入了解](vmware-azure-deploy-configuration-server.md)
- 自動探索需要複寫的 VM。 了解如何準備帳戶以進行自動探索。 [深入了解](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-automatic-discovery)


### <a name="what-access-does-site-recovery-need-to-vmware-vms"></a>Site Recovery 需要有 VMware VM 的哪些存取權？

- VMware VM 必須已安裝並執行 Site Recovery 行動服務，才能進行複寫。 您可以手動部署工具，或指定 Site Recovery 應在您為 VM 啟用複寫時執行服務的推送安裝。 若要進行推送安裝，Site Recovery 必須要有可用來安裝服務元件的帳戶。 [深入了解](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-mobility-service-installation)。 處理序伺服器 (依預設在組態伺服器上執行) 會執行此安裝。 [深入了解](vmware-azure-install-mobility-service.md)行動服務安裝。
- 在複寫期間，VM 會與 Site Recovery 通訊，如下所示：
    - VM 會在連接埠 HTTPS 443 上與組態伺服器進行通訊，以進行複寫管理。
    - VM 會透過連接埠 HTTPS 9443 (可修改) 將複寫資料傳送到處理序伺服器。
    - 如果您啟用多部 VM 一致性，則 VM 會透過連接埠 20004 互相通訊。


### <a name="is-replication-data-sent-to-site-recovery"></a>複寫資料是否會傳送至 Site Recovery？
否，Site Recovery 不會攔截複寫的資料，也不會擁有任何關於您的 VM 上執行哪些項目的資訊。 複寫資料會在 VMware Hypervisor 與 Azure 儲存體之間交換。 Site Recovery 並不具有攔截該資料的能力。 只有協調複寫與容錯移轉所需的中繼資料會被傳送給 Site Recovery 服務。  

Site Recovery 已通過 ISO 27001:2013、27018、HIPAA、DPA 認證，並且正在進行 SOC2 和 FedRAMP JAB 評定程序。

### <a name="can-we-keep-on-premises-metadata-within-a-geographic-regions"></a>是否可將內部部署中繼資料保留在地理區域內？
是。 當您在某個區域中建立保存庫時，我們會確保 Site Recovery 所使用的所有中繼資料都會保留在該區域的地理界限內。

### <a name="does-site-recovery-encrypt-replication"></a>Site Recovery 會將複寫加密嗎？
是，傳輸中加密和 [Azure 中的加密](https://docs.microsoft.com/azure/storage/storage-service-encryption)均受支援。


## <a name="failover-and-failback"></a>容錯移轉和容錯回復
### <a name="how-far-back-can-i-recover"></a>最多可復原至多久之前？
針對 VMware 到 Azure 的復原，您可以使用的最舊復原點為 72 小時。

### <a name="how-do-i-access-azure-vms-after-failover"></a>在容錯移轉之後如何存取 Azure VM？
容錯移轉後，您可以透過安全的網際網路連線、透過站對站 VPN 或透過 Azure ExpressRoute 存取 Azure VM。 您必須做好一些準備才能連線。 [深入了解](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover)

### <a name="is-failed-over-data-resilient"></a>容錯移轉後的資料可復原嗎？
Azure 是針對復原能力而設計的。 Site Recovery 設計成可根據 Azure SLA 容錯移轉至次要 Azure 資料中心。 當容錯移轉發生時，我們會確保您的中繼資料和保存庫都保留在您為保存庫選擇的相同地理區域中。

### <a name="is-failover-automatic"></a>容錯移轉是自動發生的嗎？
[容錯移轉](site-recovery-failover.md)不會自動執行。 您可以在入口網站中按一下適當按鈕來起始容錯移轉，或是使用 [PowerShell](/powershell/module/azurerm.siterecovery) 來觸發容錯移轉。

### <a name="can-i-fail-back-to-a-different-location"></a>是否可容錯回復至不同的位置？
是的，在容錯移轉至 Azure 之後，如果原始位置無法使用，您可以容錯回復至不同的位置。 [深入了解](concepts-types-of-failback.md#alternate-location-recovery-alr)。

### <a name="why-do-i-need-a-vpn-or-expressroute-to-fail-back"></a>為何需要以 VPN 或 ExpressRoute 進行容錯回復？

您從 Azure 容錯回復時，Azure 中的資料會重新複製到內部部署 VM，因此需要私用存取。



## <a name="automation-and-scripting"></a>自動化和指令碼

### <a name="can-i-set-up-replication-with-scripting"></a>是否可以使用指令碼設定複寫？
是。 您可以使用 Rest API、PowerShell 或 Azure SDK 將 Site Recovery 的工作流程自動化。[深入了解](vmware-azure-disaster-recovery-powershell.md)。

## <a name="performance-and-capacity"></a>效能與容量
### <a name="can-i-throttle-replication-bandwidth"></a>是否可進行複寫頻寬節流？
是。 [深入了解](site-recovery-plan-capacity-vmware.md)。


## <a name="next-steps"></a>後續步驟
* [檢閱](vmware-physical-azure-support-matrix.md)支援需求。
* [設定](vmware-azure-tutorial.md) VMware 到 Azure 的複寫。
