---
title: 使用 SQL Server 和 Azure Site Recovery 複寫應用程式 | Microsoft Docs
description: 本文說明如何使用適用於 SQL Server 災害復原功能的 Azure Site Recovery 來複寫 SQL Server。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: conceptual
ms.date: 07/22/2018
ms.author: raynew
ms.openlocfilehash: a734999f0fa473b5f1ee8ed55c4b73950566490d
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43188768"
---
# <a name="protect-sql-server-using-sql-server-disaster-recovery-and-azure-site-recovery"></a>使用 SQL Server 災害復原和 Azure Site Recovery 保護 SQL Server

本文說明如何使用一套 SQL Server 商務持續性和災害復原 (BCDR) 技術，加上 [Azure Site Recovery](site-recovery-overview.md)，以保護應用程式的 SQL Server 後端。

開始之前，請確定您了解 SQL Server 災害復原功能，包括容錯移轉叢集、Always On 可用性群組、資料庫鏡像和記錄傳送。


## <a name="sql-server-deployments"></a>SQL Server 部署

許多工作負載都使用 SQL Server 做為基礎，還可以與 SharePoint、Dynamics 和 SAP 等應用程式整合，以實作資料服務。  有許多方法可以部署 SQL Server：

* **獨立 SQL Server**：SQL Server 和所有資料庫都裝載在單一電腦 (實體或虛擬) 上。 如果是虛擬，則主機叢集用於高可用性。 不實作來賓層級的高可用性。
* **SQL Server 容錯移轉叢集執行個體 (Always ON FCI)**：在「Windows 容錯移轉」叢集中設定 2 個或更多個具有共用磁碟的 SQL Server 執行個體節點。 如果有一個節點關閉，叢集可以將 SQL Server 容錯移轉至另一個執行個體。 這項設定通常用在主要網站實作高可用性。 這種部署無法防止共用儲存體層中發生失敗或中斷。 共用磁碟可以使用 iSCSI、光纖通道或共用 vhdx 來實作。
* **SQL Always On 可用性群組**：在不共用任何內容的叢集中設定兩個節點，其中此叢集的 SQL Server 資料庫是設定在具有同步複寫和自動容錯移轉功能的可用性群組中。

 本文利用下列原生 SQL 災害復原技術，將資料庫復原至遠端網站︰

* 支援 SQL Server 2012 或 2014 Enterprise 版本之災害復原的「SQL AlwaysOn 可用性群組」。
* SQL Server Standard 版本 (任何版本) 或 SQL Server 2008 R2 的高安全性模式「SQL 資料庫鏡像」。

## <a name="site-recovery-support"></a>Site Recovery 支援

### <a name="supported-scenarios"></a>支援的案例
資料表中摘要說明 Site Recovery 可以保護 SQL Server。

**案例** | **至次要網站** | **至 Azure**
--- | --- | ---
**Hyper-V** | 是 | 是
**VMware** | 是 | 是
**實體伺服器** | 是 | 是
**Azure**|NA| 是

### <a name="supported-sql-server-versions"></a>支援的 SQL Server 版本
在支援的案例中，支援這些 SQL Server 版本：

* SQL Server 2016 Enterprise 和 Standard
* SQL Server 2014 Enterprise 和 Standard
* SQL Server 2012 Enterprise 和 Standard
* SQL Server 2008 R2 Enterprise 和 Standard

### <a name="supported-sql-server-integration"></a>支援的 SQL Server 整合

Site Recovery 可以與資料表中摘要說明的原生 SQL Server BCDR 技術整合，以提供災害復原解決方案。

**功能** | **詳細資料** | **SQL Server** |
--- | --- | ---
**Always On 可用性群組** | 多個 SQL Server 獨立執行個體中，每個都在含有多個節點的容錯移轉叢集中執行。<br/><br/>資料庫可以分組到可以在 SQL Server 執行個體上複製 (鏡像) 的容錯移轉群組，因此不需要任何共用儲存體。<br/><br/>在主要站台與一或多個次要站台之間提供災害復原功能。 使用同步複寫與自動容錯移轉在可用性群組中設定 SQL Server 資料庫時，可以在不共用任何內容的叢集中設定兩個節點。 | SQL Server 2016、SQL Server 2014 和 SQL Server 2012 Enterprise Edition
**容錯移轉叢集 (Always On FCI)** | SQL Server 會針對內部部署 SQL Server 工作負載的高可用性，運用 Windows 容錯移轉叢集。<br/><br/>使用共用磁碟執行 SQL Server 執行個體的節點是在容錯移轉叢集中設定。 如果執行個體關閉，叢集會容錯移轉至另一個節點。<br/><br/>叢集無法防止共用儲存體失敗或中斷。 共用磁碟可以使用 iSCSI、光纖通道或共用 VHDX 實作。 | SQL Server Enterprise Edition<br/><br/>SQL Server Standard 版本 (僅限兩個節點)
**資料庫鏡像 (高安全性模式)** | 將單一資料庫保護為單一次要複本。 提供高安全性 (同步) 和高效能 (非同步) 複寫模式。 不需要容錯移轉叢集。 | SQL Server 2008 R2<br/><br/>SQL Server Enterprise 所有版本
**獨立式 SQL Server** | SQL Server 和資料庫裝載在單一伺服器 (實體或虛擬) 上。 如果是虛擬伺服器，則主機叢集用於高可用性。 沒有來賓層級的高可用性。 | Enterprise 或 Standard Edition

## <a name="deployment-recommendations"></a>部署建議

下表摘要說明我們將 SQL Server BCDR 技術與 Site Recovery 整合的建議。

| **版本** | **版本** | **部署** | **內部部署到內部部置** | **內部部署到 Azure** |
| --- | --- | --- | --- | --- |
| SQL Server 2016、2014 或 2012 |Enterprise |容錯移轉叢集執行個體 |Always On 可用性群組 |Always On 可用性群組 |
|| Enterprise |高可用性的 Always On 可用性群組 |Always On 可用性群組 |Always On 可用性群組 | |
|| 標準 |容錯移轉叢集執行個體 (FCI) |包含本機鏡像的 Site Recovery 複寫 |包含本機鏡像的 Site Recovery 複寫 | |
|| Enterprise 或 Standard |獨立 |Site Recovery 複寫 |Site Recovery 複寫 | |
| SQL Server 2008 R2 或 2008 |Enterprise 或 Standard |容錯移轉叢集執行個體 (FCI) |包含本機鏡像的 Site Recovery 複寫 |包含本機鏡像的 Site Recovery 複寫 |
|| Enterprise 或 Standard |獨立 |Site Recovery 複寫 |Site Recovery 複寫 | |
| SQL Server (任何版本) |Enterprise 或 Standard |容錯移轉叢集執行個體 - DTC 應用程式 |Site Recovery 複寫 |不支援 |

## <a name="deployment-prerequisites"></a>部署必要條件

* 執行支援的 SQL Server 版本的內部部署 SQL Server 部署。 通常，您的 SQL Server 也需要 Active Directory。
* 您要部署之案例的需求。 深入了解[複寫至 Azure](site-recovery-support-matrix-to-azure.md) 和[內部部署](site-recovery-support-matrix.md)的支援需求，以及[部署必要條件](site-recovery-prereq.md)。

## <a name="set-up-active-directory"></a>設定 Active Directory

在次要復原網站上設定 Active Directory，讓 SQL Server 正常運作。

* **小型企業** — 如果只有少數應用程式，且內部部署網站只有一個網域控制站，當您想要容錯移轉整個網站時，建議您使用 Site Recovery 複寫，將網域控制站複寫至次要資料中心或 Azure。
* **中大型企業** — 如果您有大量的應用程式和一個 Active Directory 樹系，而您想要依應用程式或工作負載容錯移轉，建議您在次要資料中心或 Azure 中設定其他網域控制站。 如果您使用 Always On 可用性群組復原至遠端網站，建議您在次要網站或 Azure 上設定其他不同的網域控制站，供已復原的 SQL Server 執行個體使用。

本文中的指示假設在次要位置中有網域控制站。 [閱讀更多](site-recovery-active-directory.md) 有關使用 Site Recovery 保護 Active Directory。


## <a name="integrate-with-sql-server-always-on-for-replication-to-azure"></a>與 SQL Server Always On 整合以複寫至 Azure

以下是您需要採取的動作：

1. 將指令碼匯入您的 Azure 自動化帳戶。 這包括用於將 [Resource Manager 虛擬機器](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/asr-automation-recovery/scripts/ASR-SQL-FailoverAG.ps1)和[傳統虛擬機器](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/asr-automation-recovery/scripts/ASR-SQL-FailoverAGClassic.ps1)中的 SQL 可用性群組容錯移轉的指令碼。

    [![部署至 Azure](https://azurecomcdn.azureedge.net/mediahandler/acomblog/media/Default/blog/c4803408-340e-49e3-9a1f-0ed3f689813d.png)](https://aka.ms/asr-automationrunbooks-deploy)


1. 新增 ASR-SQL-FailoverAG 作為復原方案第一個群組的前置動作。

1. 依照指令碼中的指示建立自動化變數，以提供可用性群組的名稱。

### <a name="steps-to-do-a-test-failover"></a>測試容錯移轉的步驟

SQL Always On 原本就不支援測試容錯移轉。 因此，我們建議您採取下列動作︰

1. 在 Azure 中裝載可用性群組複本的虛擬機器上，設定 [Azure 備份](../backup/backup-azure-arm-vms.md)。

1. 觸發復原計劃的測試容錯移轉之前，從上一步所使用的備份來復原虛擬機器。

    ![從 Azure 備份還原 ](./media/site-recovery-sql/restore-from-backup.png)

1. 在從備份還原的虛擬機器中[強制仲裁](https://docs.microsoft.com/sql/sql-server/failover-clusters/windows/force-a-wsfc-cluster-to-start-without-a-quorum#PowerShellProcedure)。

1. 將接聽程式的 IP 更新為測試容錯移轉網路中可用的 IP 的。

    ![更新接聽程式 IP](./media/site-recovery-sql/update-listener-ip.png)

1. 使接聽程式上線。

    ![使接聽程式上線](./media/site-recovery-sql/bring-listener-online.png)

1. 使用一個 IP 和 SQL 虛擬機器建立負載平衡器；此 IP 是對應至每個可用性群組接聽程式的前端 IP 集區下的 IP，而是虛擬機器是在後端集區中新增的 SQL 虛擬機器。

     ![建立負載平衡器 - 前端 IP 集區 ](./media/site-recovery-sql/create-load-balancer1.png)

    ![建立負載平衡器 - 後端集區 ](./media/site-recovery-sql/create-load-balancer2.png)

1. 執行復原計劃的測試容錯移轉。

### <a name="steps-to-do-a-failover"></a>進行容錯移轉的步驟

一旦您已在復原方案中新增指令碼，並已執行測試容錯移轉驗證過復原計劃執行，便可以進行復原計劃的容錯移轉。


## <a name="integrate-with-sql-server-always-on-for-replication-to-a-secondary-on-premises-site"></a>與 SQL Server Always On 整合以複寫至次要內部部署網站

如果 SQL Server 為了高可用性而使用可用性群組 (或 FCI)，建議您也在復原網站上使用可用性群組。 請注意，這適用於不使用分散式交易的應用程式。

1. [設定資料庫](https://msdn.microsoft.com/library/hh213078.aspx) 。
1. 在次要網站上建立虛擬網路。
1. 在虛擬網路和主要網站之間設定網站間 VPN 連線。
1. 在復原網站上建立虛擬機器，並在此虛擬機器上安裝 SQL Server。
1. 將現有的 Always On 可用性群組擴充至新的 SQL Server VM。 將此 SQL Server 執行個體設定為非同步複本。
1. 建立可用性群組接聽程式，或更新現有的接聽程式，以包含非同步複本虛擬機器。
1. 請確定應用程式伺服器陣列是使用接聽程式來設定。 如果是使用資料庫伺服器名稱來設定，請更新為使用接聽程式，如此您就不需要在容錯移轉後重新設定。

對於使用分散式交易的應用程式，建議您使用 [VMware/實體伺服器站對站複寫](site-recovery-vmware-to-vmware.md)來部署 Site Recovery。

### <a name="recovery-plan-considerations"></a>復原計畫考量
1. 將此範例指令碼新增至主要和次要網站上的 VMM 程式庫。

        Param(
        [string]$SQLAvailabilityGroupPath
        )
        import-module sqlps
        Switch-SqlAvailabilityGroup -Path $SQLAvailabilityGroupPath -AllowDataLoss -force

1. 當您建立應用程式的復原計畫時，將前置動作加入 Group-1 的指令碼步驟，以叫用指令碼來容錯移轉可用性群組。

## <a name="protect-a-standalone-sql-server"></a>保護獨立式 SQL Server

在此案例中，建議您使用 Site Recovery 複寫保護 SQL Server 電腦。 確切步驟將取決於 SQL Server 是 VM 還是實體伺服器，以及您想要複寫至 Azure 還是次要內部部署網站。 了解 [Site Recovery 案例](site-recovery-overview.md)。

## <a name="protect-a-sql-server-cluster-standard-editionwindows-server-2008-r2"></a>保護 SQL Server 叢集 (標準版/Windows Server 2008 R2)

對於執行 SQL Server Standard Edition 或 SQL Server 2008 R2 的叢集，建議您使用 Site Recovery 複寫保護 SQL Server。

### <a name="on-premises-to-on-premises"></a>內部部署至內部部署

* 如果應用程式使用分散式交易，建議您針對 Hyper-V 環境部署 [Site Recovery 搭配 SAN 複寫](site-recovery-vmm-san.md)，或者，針對 VMware 環境，則部署 [VMware/實體伺服器至 VMware](site-recovery-vmware-to-vmware.md)。
* 針對非 DTC 應用程式，請利用本機高安全性資料庫鏡像，依上述方式將叢集復原成獨立伺服器。

### <a name="on-premises-to-azure"></a>內部部署至 Azure

複寫至 Azure 時，Site Recovery 不提供來賓叢集支援。 SQL Server 也不會為 Standard Edition 提供低成本的災害復原解決方案。 在此案例中，建議您將內部部署 SQL Server 叢集保護至獨立式 SQL Server，並在 Azure 中復原。

1. 在內部部署站台上設定一個額外的獨立 SQL Server 執行個體。
1. 設定此執行個體做為您要保護之資料庫的鏡像。 在高安全性模式下設定鏡像。
1. 針對 [Hyper-V](site-recovery-hyper-v-site-to-azure.md) 或 [VMware WM/實體伺服器](site-recovery-vmware-to-azure-classic.md)，在內部部署網站上設定 Site Recovery。
1. 使用 Site Recovery 複寫將新的 SQL Server 執行個體複寫至 Azure。 因為它是高安全性鏡像副本，它會與主要叢集同步處理，但是會使用 Site Recovery 複寫將它複寫至 Azure。


![標準叢集](./media/site-recovery-sql/standalone-cluster-local.png)

### <a name="failback-considerations"></a>容錯回復考量

就 SQL Server 標準叢集而言，若要在未規劃的容錯移轉之後進行容錯回復，必須從鏡像執行個體進行 SQL 備份並還原至原始叢集，並重新建立鏡像。

## <a name="next-steps"></a>後續步驟
[深入了解](site-recovery-components.md) Site Recovery 架構。
