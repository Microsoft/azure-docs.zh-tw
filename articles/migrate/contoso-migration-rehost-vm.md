---
title: 利用 Azure Site Recovery 來移轉至 Azure VM，以便重新裝載 Contoso 應用程式 | Microsoft 文件
description: 了解如何使用 Azure Site Recovery 服務，透過將內部部署機器隨即轉移至 Azure，以重新裝載內部部署應用程式。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: raynew
ms.openlocfilehash: 40efd1b556db5cbebff80293a91a228afe17eab5
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44297412"
---
# <a name="contoso-migration-rehost-an-on-premises-app-to-azure-vms"></a>Contoso 移轉：將內部部署應用程式重新裝載至 Azure VM


本文說明 Contoso 如何將應用程式 VM 移轉至 Azure VM，藉以在 Azure 中重新裝載內部部署 SmartHotel360 應用程式。


這份文件是系列文章中的其中一篇，說明 Contoso 這家虛構的公司如何將內部部署資源移轉至 Microsoft Azure 雲端。 整個系列文章中包含背景資訊和說明如何設定移轉基礎結構的案例、評估內部部署資源的移轉，以及執行不同類型的移轉。 案例的複雜性會提高。 我們會不定期新增其他文章。

**文章** | **詳細資料** | **狀態**
--- | --- | ---
[文章 1：概觀](contoso-migration-overview.md) | 系列文章的概觀、Contoso 的移轉策略，以及此系列中所使用的應用程式範例。 | 可用
[文章 2：部署 Azure 基礎結構](contoso-migration-infrastructure.md) | Contoso 會準備其內部部署基礎結構和其 Azure 基礎結構以進行移轉。 此系列中的所有移轉文章都使用相同的基礎結構。 | 可用
[文章 3：存取內部部署資源以移轉至 Azure](contoso-migration-assessment.md)  | Contoso 會評估其在 VMware 上執行的內部部署 SmartHotel360 應用程式。 Contoso 會使用 Azure Migrate 服務來評估應用程式 VM，並使用 Database Migration Assistant 來評估應用程式 SQL Server 資料庫。 | 可用
[文章 4：在 Azure VM 和 SQL Database 受控執行個體上重新裝載應用程式](contoso-migration-rehost-vm-sql-managed-instance.md) | Contoso 會為其內部部署 SmartHotel360 應用程式執行隨即移轉至 Azure 的作業。 Contoso 會使用 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview) 來移轉應用程式的前端 VM。 Contoso 會使用 [Azure 資料庫移轉服務](https://docs.microsoft.com/azure/dms/dms-overview)將應用程式資料庫移轉至 Azure SQL Database 受控執行個體。 | 可用   
文章 5：在 Azure VM 上重新裝載應用程式 | Contoso 會使用 Site Recovery 服務，將其 SmartHotel360 應用程式 VM 移轉至 Azure VM。 | 本文
[文章 6：在 Azure VM 上和 SQL Server AlwaysOn 可用性群組中重新裝載應用程式](contoso-migration-rehost-vm-sql-ag.md) | Contoso 會移轉 SmartHotel360 應用程式。 Contoso 會使用 Site Recovery 來遷移應用程式的 VM。 它會使用資料庫移轉服務，將應用程式資料庫遷移至受到 SQL Server 可用性群組保護的 SQL Server 叢集。 | 可用 
[文章 7：將 Linux 應用程式重新裝載至 Azure VM](contoso-migration-rehost-linux-vm.md) | Contoso 會使用 Site Recovery 服務，來完成將其 Linux osTicket 應用程式隨即移轉至 Azure VM 的作業 | 可用
[文章 8：在 Azure VM 和 Azure MySQL 上重新裝載 Linux 應用程式](contoso-migration-rehost-linux-vm-mysql.md) | Contoso 會使用 Azure Site Recovery 將 Linux osTicket 應用程式移轉至 Azure VM，並使用 MySQL Workbench 將應用程式資料庫移轉至 Azure MySQL Server 執行個體。 | 可用
[文章 9：在 Azure Web 應用程式和 Azure SQL 資料庫上重構應用程式](contoso-migration-refactor-web-app-sql.md) | Contoso 會將 SmartHotel360 應用程式移轉至 Azure Web 應用程式，以及使用 Database Migration Assistant 將應用程式資料庫移轉至 Azure SQL Server 執行個體 | 可用
[文章 10：在 Azure Web Apps 和 Azure MySQL 上重構 Linux 應用程式](contoso-migration-refactor-linux-app-service-mysql.md) | Contoso 會使用 Azure 流量管理員，將 Linux osTicket 應用程式移轉至多個 Azure 區域的 Azure Web 應用程式，與 GitHub 整合以進行持續傳遞。 Contoso 會將應用程式資料庫遷移至適用於 MySQL 的 Azure 資料庫執行個體。 | 可用 
[文章 11：重構 Azure DevOps Services 上的 TFS](contoso-migration-tfs-vsts.md) | Contoso 會將其內部部署 Team Foundation Server 部署，移轉至 Azure 中的 Azure DevOps Services。 | 可用
[文章 12：在 Azure 容器和 Azure SQL Database 上重新建構應用程式](contoso-migration-rearchitect-container-sql.md) | Contoso 會將其 SmartHotel 應用程式移轉至 Azure。 然後，它會重新建構應用程式 Web 層，作為在 Azure Service Fabric 中執行的 Windows 容器，以及具有 Azure SQL Database 的資料庫。 | 可用
[文章 13：在 Azure 中重建應用程式](contoso-migration-rebuild.md) | Contoso 會藉由使用各種 Azure 功能和服務 (包括 Azure App Service、Azure Kubernetes Service (AKS)、Azure Functions、Azure 認知服務及 Azure Cosmos DB) 重建其 SmartHotel 應用程式。 | 可用


在本文中，Contoso 會移轉兩層式 Windows。 NET SmartHotel360 應用程式 (執行於 VMware VM)，移轉至 Azure。 如果想使用這個應用程式，目前可以取得開放原始碼的格式，請從 [github](https://github.com/Microsoft/SmartHotel360) 下載。



## <a name="business-drivers"></a>商業動機

IT 領導小組與商務合作夥伴密切合作，以了解此次移轉所要實現的目標：

- **因應業務成長**：Contoso 的業務量日益增多，對內部部署系統和基礎結構造成了壓力。
- **限制風險**：SmartHotel360 應用程式對 Contoso 的業務影響甚大。 Contoso 想要在亳無風險的情況下，將應用程式移至 Azure。
- **擴充**：Contoso 不想要修改應用程式，但想要確保其穩定性。


## <a name="migration-goals"></a>移轉目標

Contoso 雲端小組已針對此次移轉擬定好各項目標。 這些目標是用來判斷最合適的移轉方法：

- 移轉之後，應用程式不管是在 Azure 或 VMware 中，應具有相同效能。  應用程式不管是在雲端中或在內部部署，都一樣重要。 
- Contoso 不想在這個應用程式上投注資源。  這對企業很重要，但以其目前的格式而言，Contoso 只想安全地移至雲端。
- Contoso 不想變更這個應用程式的 ops 模型。 Contoso 想要按照目前的方式，與其在雲端中互動。
- Contoso 不想變更任何應用程式的功能。 只會變更應用程式位置。


## <a name="solution-design"></a>解決方案設計

擬定好目標和需求後，Contoso 會設計和檢閱部署解決方案，並識別移轉程序，包括 Contoso 會用於移轉的 Azure 服務。

### <a name="current-app"></a>目前的應用程式

- 這個應用程式橫跨兩層 VM (**WEBVM** 和 **SQLVM**)。
- VM 位於 VMware ESXi 主機 **contosohost1.contoso.com** (6.5 版)。
- VMware 環境是由 VM 上執行的 vCenter Server 6.5 (**vcenter.contoso.com**) 進行管理。
- Contoso 有內部部署資料中心 (contoso-datacenter) 以及內部部署網域控制站 (**contosodc1**)。

### <a name="proposed-architecture"></a>建議的架構

- 因為應用程式為生產工作負載，所以 Azure 中的應用程式 VM 會位於生產資源群組 ContosoRG 中。
- 應用程式 VM 將會遷移到主要 Azure 區域 (美國東部 2)，並且放在生產網路 (VNET-PROD-EUS2) 中。
- Web 前端 VM 會位於生產網路中的前端子網路 (PROD-FE-EUS2)。
- 資料庫 VM 會位於生產網路中的資料庫子網路 (PROD-DB-EUS2)。
- 移轉完成之後，將會解除委任 Contoso 資料中心的內部部署 VM。

![案例架構](./media/contoso-migration-rehost-vm/architecture.png) 

### <a name="database-considerations"></a>資料庫考量

在解決方案設計過程中，Contoso 會進行 Azure SQL Database 與 SQL Server 的功能比較。 下列考量協助他們決定使用在 Azure IaaS VM 上執行的 SQL Server： 

- 如果 Contoso 需要自訂作業系統或資料庫伺服器，或者想要在相同的 VM 上共置並執行第三方應用程式，使用執行 SQL Server 的 Azure VM 似乎是最佳解決方案。
- 透過軟體保證，Contoso 未來可以使用適用於 SQL Server 的 Azure Hybrid Benefit，以折扣優惠在 SQL Database 受控執行個體上交換執行個體的現有授權。 最多可節省 30% 的受控執行個體。



### <a name="solution-review"></a>解決方案檢閱

Contoso 會透過比較一份優缺點清單，來評估建議設計。

**考量** | **詳細資料**
--- | ---
**優點** | 這兩個應用程式 VM 都會移至 Azure (不需變更)，讓移轉變簡單。<br/><br/> 由於 Contoso 對這兩個應用程式 VM 使用隨即轉移，所以應用程式資料庫不需要任何特殊設定或移轉工具。<br/><br/> Contoso 可以使用 Azure Hybrid Benefit 來充分利用軟體保證的投資。<br/><br/> Contoso 會保留 Azure 中應用程式 VM 的完整控制權。 
**缺點** | WEBVM 和 SQLVM 會執行 Windows Server 2008 R2。 Azure 對此作業系統的支援僅限於特定角色 (2018 年 7 月)。 [深入了解](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines)。<br/><br/> 應用程式的 Web 和資料層會保留單一容錯移轉點。</br><br/> SQLVM 是在非主流支援的 SQL Server 2008 R2 上執行。 不過，Azure VM 受到支援 (2018 年 7 月)。 [深入了解](https://support.microsoft.com/en-us/help/956893)。<br/><br/> Contoso 必須繼續支持此應用程式作為 Azure VM，而不是轉向 Azure App Service 與 Azure SQL Database 等受控服務。



### <a name="migration-process"></a>移轉程序

Contoso 會透過 Site Recovery，將應用程式前端和資料庫 VM 遷移至 Azure VM：

- 在第一個步驟中，Contoso 會準備以及設定 Site Recovery 的 Azure 元件，然後準備內部部署 VMware 基礎結構。
- 他們的 [Azure 基礎結構](contoso-migration-infrastructure.md)已經準備到位，所以 Contoso 只需要特別為 Site Recovery 增加幾個 Azure 元件即可。
- 等一切就緒，Contoso 就可以開始複寫 VM。
- 複寫功能啟用且正常運作後，Contoso 會將 VM 容錯移轉至 Azure，而加以遷移。

![移轉程序](./media/contoso-migration-rehost-vm/migraton-process.png) 



### <a name="azure-services"></a>Azure 服務

**服務** | **說明** | **成本**
--- | --- | ---
[Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/) | 此服務可協調和管理 Azure VM 以及內部部署 VM 和實體伺服器的移轉和災害復原。  | 複寫至 Azure 的期間會產生 Azure 儲存體費用。  發生容錯移轉時，系統會建立 Azure VM 並產生費用。 [深入了解](https://azure.microsoft.com/pricing/details/site-recovery/)費用和定價。


## <a name="prerequisites"></a>必要條件

以下是 Contoso 要執行此案例所需的項目。

**需求** | **詳細資料**
--- | ---
**Azure 訂用帳戶** | Contoso 已在本系列稍早的文章中建立訂用帳戶。 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。<br/><br/> 如果您建立免費帳戶，您就是訂用帳戶的管理員，並可執行所有動作。<br/><br/> 如果您使用現有訂用帳戶，而且您不是系統管理員，則需要與系統管理員合作，讓其指派擁有者或參與者權限給您。<br/><br/> 如果您需要更細微的權限，請檢閱[此文章](../site-recovery/site-recovery-role-based-linked-access-control.md)。 
**Azure 基礎結構** | [了解](contoso-migration-infrastructure.md) Contoso 如何設定 Azure 基礎結構。<br/><br/> 深入了解 Site Recovery 具體的[網路](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#network)和[儲存體](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#storage)需求。
**內部部署伺服器** | 內部部署 vCenter 伺服器應該執行版本 5.5、6.0 或 6.5<br/><br/> ESXi 主機應該執行版本 5.5、6.0 或 6.5<br/><br/> 一或多部在 ESXi 主機上執行的 VMware VM。
**內部部署 VM** | VM 必須符合 [Azure 需求](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#azure-vm-requirements)。


## <a name="scenario-steps"></a>案例步驟

以下是 Contoso 管理員執行移轉的方式：

> [!div class="checklist"]
> * **步驟 1：針對 Site Recovery 準備 Azure**：他們要建立一個 Azure 儲存體帳戶來保存複寫的資料，還要建立一個復原服務保存庫。
> * **步驟 2：針對 Site Recovery 準備內部部署 VMware**：他們要準備帳戶以便探索 VM 和安裝代理程式，並準備在容錯移轉後連線至 Azure VM。
> * **步驟 3：複寫 VM**：他們要設定複寫，然後開始將 VM 複寫至 Azure 儲存體。
> * **步驟 4：使用 Site Recovery 移轉 VM**：他們會執行容錯移轉測試，確定一切都能正常運作，然後執行完整的容錯移轉，以便將 VM 移轉至 Azure。




## <a name="step-1-prepare-azure-for-the-site-recovery-service"></a>步驟 1：針對 Site Recovery 服務準備 Azure

以下是 Contoso 將 VM 移轉至 Azure 時，所需的 Azure 元件：

- 容錯移轉期間建立 Azure VM 時，這些 VM 所在的 VNet。
- 用來保存複寫資料的 Azure 儲存體帳戶。 
- Azure 中的復原服務保存庫。

他們依照下列方式進行其設定：

1. 設定網路 - Contoso 已經設定好網路，當他們[部署 Azure 基礎結構](contoso-migration-infrastructure.md)時，就可以用於 Site Recovery。

    - SmartHotel360 應用程式為生產應用程式，而且 VM 會被移轉至美國東部 2 主要區域的 Azure 生產網路 (VNET-PROD-EUS2)。
    - 這兩個 VM 會置於可作為生產資源的 ContosoRG 資源群組中。
    - 應用程式前端 VM (WEBVM) 將移轉至生產網路中的前端子網路 (PROD-FE-EUS2)。
    - 應用程式前端 VM (SQLVM) 將移轉至生產網路中的資料庫子網路 (PROD-DB-EUS2)。

2. 設定儲存體帳戶 - Contoso 在主要區域中建立 Azure 儲存體帳戶 (contosovmsacc20180528)。
    - 儲存體帳戶與復原服務保存庫必須位於相同的區域。
    - 他們使用一般用途的帳戶，並配備標準儲存體和 LRS 複寫。 

    ![Site Recovery 儲存體](./media/contoso-migration-rehost-vm/asr-storage.png)

3. 建立保存庫 - 網路和儲存體帳戶準備就緒之後，Contoso 現在會建立復原服務保存庫 (ContosoMigrationVault)，然後將它放在美國東部 2 主要區域的 ContosoFailoverRG 資源群組中。

    ![復原服務保存庫](./media/contoso-migration-rehost-vm/asr-vault.png)

**需要其他協助？**

[了解如何](https://docs.microsoft.com/azure/site-recovery/tutorial-prepare-azure)針對 Site Recovery 設定 Azure。


## <a name="step-2-prepare-on-premises-vmware-for-site-recovery"></a>步驟 2：針對 Site Recovery 準備內部部署 VMware

以下是 Contoso 針對內部部署所準備的內容：

- 在 vCenter Server 或 vSphere ESXi 主機上準備一個帳戶，用於自動執行 VM 探索。
- 一個帳戶，允許在 VMware VM 上自動安裝行動服務。 
- 內部部署 VM 設定，讓 Contoso 在容錯移轉之後，可以連線到複寫的 Azure VM。


### <a name="prepare-an-account-for-automatic-discovery"></a>準備帳戶以進行自動探索

Site Recovery 需要存取 VMware 伺服器才能：

- 自動探索 VM。 
- 協調 VM 的複寫、容錯移轉和容錯回復。
- 需要至少一個唯讀帳戶。 這個帳戶應該可以執行各種操作，例如建立和移除磁碟以及啟動 VM。

Contoso 管理員會依照下列方式設定帳戶：

1. 他們在 vCenter 層級建立一個角色。
2. 他們會指派必要權限給角色。



### <a name="prepare-an-account-for-mobility-service-installation"></a>準備一個用來安裝行動服務的帳戶

行動服務必須安裝在每一個 VM 上。

- 啟用 VM 複寫功能時，Site Recovery 可以自動推送安裝行動服務。
- 需要一個帳戶，以便 Site Recovery 可以存取推入安裝的 VM。 當您設定複寫時，需要指定此帳戶。
- 帳戶可以是網域帳戶或本機帳戶，而且擁有 VM 的安裝權限。


### <a name="prepare-to-connect-to-azure-vms-after-failover"></a>準備在容錯移轉後連接到 Azure VM

容錯移轉之後，Contoso 會想要連線到 Azure VM。 為此，Contoso 管理員在移轉之前執行了下列作業：

1. 為了透過網際網路存取，他們：

 - 先在內部部署 VM 上啟用 RDP，然後再容錯移轉。
 - 確定已為**公用**設定檔新增 TCP 和 UDP 規則。
 - 針對所有設定檔，在 [Windows 防火牆] > [允許的應用程式] 中檢查是否允許 RDP。
 
2. 為了透過站對站 VPN 存取，他們：

 - 在內部部署機器上啟用 RDP。
 - 在 [Windows 防火牆] -> [允許的應用程式與功能] 中，允許 [網域和私人] 網路使用 RDP。
 - 將內部部署 VM 的作業系統 SAN 原則設定為 **OnlineAll**。

此外，當他們執行容錯移轉時，需要檢查以下各項：

- 觸發容錯移轉時，VM 上不應該有擱置的 Windows 更新。 如果有，在更新完成之前，他們將無法登入 VM。
- 在容錯移轉之後，他們可以勾選 [開機診斷] 以檢視 VM 的螢幕擷取畫面。 若未解決問題，他們應確認 VM 是否執行中，並檢閱這些[疑難排解祕訣](http://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx)。


**需要其他協助？**

- [了解](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial-prepare-on-premises#prepare-an-account-for-automatic-discovery)如何建立和指派自動探索所需的角色。
- [深入了解](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial-prepare-on-premises#prepare-an-account-for-mobility-service-installation)建立行動服務推送安裝的帳戶。


## <a name="step-3-replicate-the-on-premises-vms"></a>步驟 3：複寫內部部署 VM

Contoso 管理員必須先設定並啟用複寫，才能執行移轉至 Azure 的作業。

### <a name="set-a-replication-goal"></a>設定複寫目標

1. 在保存庫中的保存庫名稱 (ContosoVMVault) 下，他們選取複寫目標 ([開始使用] > [Site Recovery] > [準備基礎結構])。
2. 他們指定其機器都是內部部署、在 VMware 上執行，並複寫至 Azure。

    ![複寫目標](./media/contoso-migration-rehost-vm/replication-goal.png)

### <a name="confirm-deployment-planning"></a>確認部署規劃

若要繼續，他們會選取 [是，我已經完成]，確認自己已經完成部署計劃。 在這個案例中，Contoso 只移轉兩個 VM，因此不需要部署規劃。


### <a name="set-up-the-source-environment"></a>設定來源環境

Contoso 管理員必須設定來源環境。 為此，他們下載 OVF 範本，然後用它將 Site Recovery 組態伺服器部署為高可用性、內部部署的 VMware VM。 當組態伺服器開始執行之後，他們將它註冊到保存庫。

組態伺服器會執行多項元件：

- 組態伺服器元件會協調內部部署與 Azure 之間的通訊，以及管理資料複寫。
- 處理序伺服器，可作為複寫閘道。 負責接收複寫資料，以快取、壓縮和加密進行最佳化，然後將複寫資料傳送至 Azure 儲存體。
- 處理序伺服器也會在您要複寫的 VM 上安裝行動服務，並且在內部部署 VMware VM 上執行自動探索。



Contoso 管理員會依照下列方式執行這些步驟：

1. 在保存庫中，他們從 [準備基礎結構] > [來源] > [組態伺服器] 下載 OVF 範本。
    
    ![下載 OVF](./media/contoso-migration-rehost-vm/add-cs.png)

2. 他們將範本匯入 VMware 中，以建立和部署 VM。

    ![OVF 範本](./media/contoso-migration-rehost-vm/vcenter-wizard.png)

3.  當他們第一次啟動 VM 時，VM 會在開機後進入 Windows Server 2016 安裝程序。 他們接受授權合約，並輸入系統管理員密碼。
4. 安裝完成之後，他們以系統管理員身分登入 VM。 第一次登入時，預設會執行 Azure Site Recovery 設定工具。
5. 在這個工具中，他們會指定一個名稱，以在保存庫中註冊組態伺服器。
6. 此工具會檢查 VM 是否可連線到 Azure。 連線建立後，他們登入 Azure 訂用帳戶。 認證必須能夠存取他們要在其中註冊組態伺服器的保存庫。

    ![註冊設定伺服器](./media/contoso-migration-rehost-vm/config-server-register2.png)

7. 此工具會執行一些設定工作，而後重新開機。
8. 他們再次登入機器，組態伺服器管理精靈隨即自動啟動。
9. 在此精靈中，他們選取要接收複寫流量的 NIC。 這項設定調整好之後，便無法變更。
10. 他們選取訂用帳戶、資源群組，以及在其中註冊組態伺服器的保存庫。
        ![保存庫](./media/contoso-migration-rehost-vm/cswiz1.png) 

10. 他們下載並安裝 MySQL Server 和 VMWare PowerCLI。 
11. 驗證之後，他們會指定 vCenter 伺服器或 vSphere 主機的 FQDN 或 IP 位址。 他們會保留預設的連接埠，並為 Azure 中的伺服器指定易記名稱。
12. 他們會指定為自動探索而建立的帳戶，以及用來自動安裝行動服務的認證。 對於 Windows 機器，此帳戶需具備 VM 的本機系統管理員權限。

    ![vCenter](./media/contoso-migration-rehost-vm/cswiz2.png)

7. 註冊完成後，在 Azure 入口網站中確認組態伺服器和 VMware 伺服器都已列在保存庫中的 [來源] 頁面上。 探索可能需要 15 分鐘以上的時間。 
8. 接下來，Site Recovery 會使用指定的設定連線至 VMware 伺服器，並探索 VM。

### <a name="set-up-the-target"></a>設定目標

Contoso 管理員現在會指定目標複寫設定。

1. 在 [準備基礎結構] > [目標] 中，他們選取目標設定。
2. Site Recovery 會檢查指定目標位置是否有 Azure 儲存體帳戶與網路。

### <a name="create-a-replication-policy"></a>建立複寫原則

現在，Contoso 管理員可以建立複寫原則。

1. 在 [準備基礎結構] > [複寫設定] > [複寫原則] >  [建立和關聯] 中，他們建立 **ContosoMigrationPolicy** 原則。
2. 他們使用預設設定：
    - **RPO 閾值**：預設值是 60 分鐘。 這個值可定義復原點的建立頻率。 連續複寫超過此限制時會產生警示。
    - **復原點保留**。 預設值為 24 小時。 此值會指定每個復原點的保留週期有多長。 複寫的 VM 可以還原至一個週期內的任何時間點。
    - **應用程式一致快照頻率**。 預設值是一小時。 此值會指定應用程式一致快照的建立頻率。

        ![建立複寫原則](./media/contoso-migration-rehost-vm/replication-policy.png)

5. 此原則會自動與設定伺服器產生關聯。 

    ![關聯複寫原則](./media/contoso-migration-rehost-vm/replication-policy2.png)

### <a name="enable-replication-for-webvm"></a>啟用 WEBVM 的複寫

一切就緒後，Contoso 管理員可以立即啟用 VM 的複寫。 他們會從 WebVM 開始。

1. 在 [複寫應用程式] > [來源] > [+複寫] 中，他們會選取來源設定。
2. 他們表示想要啟用 VM、選取 vCenter 伺服器和組態伺服器。

    ![啟用複寫](./media/contoso-migration-rehost-vm/enable-replication1.png)

3. 他們會選取目標設定，包括資源群組和 Azure 網路以及儲存體帳戶。

     ![啟用複寫](./media/contoso-migration-rehost-vm/enable-replication2.png)

4. 他們會選取 **WebVM** 來進行複寫、檢查複寫原則並啟用複寫。

    - 在這個階段中，因為必須選取 VNet 和子網路，而且應用程式 VM 放在不同的子網路中，所以他們只會選取 WEBVM。
    - 啟用複寫時，Site Recovery 會自動在 VM 上安裝行動服務。

    ![啟用複寫](./media/contoso-migration-rehost-vm/enable-replication3.png)

5. 他們在 [作業] 中追踨複寫進度。 執行 [完成保護] 作業之後，機器即準備好進行容錯移轉。
6. 在 Azure 入口網站的 [基本資訊] 中，他們可以看到 VM 複寫至 Azure 的結構。


### <a name="enable-replication-for-sqlvm"></a>啟用 SQLVM 的複寫

現在 Contoso 管理員可以使用上述相同程序，開始複寫 SQLVM 機器。

1. 他們會選取來源設定。

    ![啟用複寫](./media/contoso-migration-rehost-vm/enable-replication1.png)

2. 然後，他們會指定目標設定。

     ![啟用複寫](./media/contoso-migration-rehost-vm/enable-replication2-sqlvm.png)

3. 他們會選取 SQLVM 來進行複寫。 

    ![啟用複寫](./media/contoso-migration-rehost-vm/enable-replication3-sqlvm.png)

4. 他們會套用針對 WEBVM 所使用的同一套複寫原則，並且啟用複寫。

    ![基礎結構檢視](./media/contoso-migration-rehost-vm/essentials.png)

**需要其他協助？**

- 您可以在[設定內部部署 VMware VM 的災害復原](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial)中，閱讀所有步驟的詳細逐步解說。
- 您會得到詳細的操作說明，協助您[設定來源環境](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-source)、[部署組態伺服器](https://docs.microsoft.com/azure/site-recovery/vmware-azure-deploy-configuration-server)以及[指定複寫設定](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-replication)。
- 您可以深入了解如何[啟用複寫](https://docs.microsoft.com/azure/site-recovery/vmware-azure-enable-replication)。


## <a name="step-4-migrate-the-vms"></a>步驟 4：遷移 VM 

Contoso 管理員會執行一次快速的容錯移轉測試，然後再執行一次完整的容錯移轉來遷移 VM。

### <a name="run-a-test-failover"></a>執行測試容錯移轉

容錯移轉測試有助於保證一切如預期般正常運作。 

1. 他們會針對最新的可用時間點 (**最近處理的**) 執行一次測試容錯移轉。
2. 如果想在觸發容錯移轉之前，讓 Site Recovery 嘗試將來源 VM 關機，請選取 [先將機器關機再開始容錯移轉]。 即使關機失敗，仍會繼續容錯移轉。 
3. 執行測試容錯移轉： 

    - 系統會執行先決條件檢查，以確保所有移轉所需的條件都已準備就緒。
    - 容錯移轉會處理資料，以便建立 Azure VM。 若選取最新的復原點，則會根據資料建立復原點。
    - 將會使用先前步驟中處理的資料來建立 Azure VM。
    
3. 容錯移轉完成後，Azure VM 複本會出現在 Azure 入口網站中。 他們會檢查 VM 的大小是否合適、是否連線到正確的網路，而且正常執行中。 
4. 確認容錯移轉測試之後，他們可以清除容錯移轉，並記錄與儲存任何觀察到的結果。 

### <a name="create-and-customize-a-recovery-plan"></a>建立並自訂復原方案

 驗證測試容錯移轉如預期般正常運作之後，Contoso 管理員會針對移轉建立復原方案。 

- 復原計劃會指定容錯移轉的發生順序，並指出 Azure VM 如何在 Azure 中正常執行。
- 由於這是兩層式應用程式，他們會自訂復原計劃，因此資料 VM (SQLVM) 會比前端 (WEBVM) 先啟動。

1. 在 [復原方案 (Site Recovery)] > [+復原方案] 中，他們會建立一個方案，並將 VM 加入其中。

    ![復原計畫](./media/contoso-migration-rehost-vm/recovery-plan.png)

2. 建立計劃之後，他們會自訂它 ([復原計劃] > [SmartHotelMigrationPlan] > [自訂])。
2.  他們會從 [群組 1：啟動] 移除 WEBVM。  這樣可確保第一個啟動動作，只會影響 SQLVM。
3.  在 [+群組] > [新增受保護項目] 中，他們會將 WEBVM 新增至 [群組 2：開始]。  VM 必須分屬兩個不同的群組。


### <a name="migrate-the-vms"></a>遷移 VM


Contoso 管理員現在會執行一次完整的容錯移轉，以完成移轉。

1. 他們會選取復原計劃 > [容錯移轉]。
2. 他們會選取容錯移轉至最新的復原點，然後 Site Recovery 應該會在嘗試觸發容錯移轉之前，先關閉內部部署 VM。 他們可以在 [作業] 頁面上追蹤容錯移轉進度。

    ![容錯移轉](./media/contoso-migration-rehost-vm/failover1.png)


3. 容錯移轉之後，他們可以確認 Azure VM 會如預期般出現在 Azure 入口網站中。

    ![容錯移轉](./media/contoso-migration-rehost-vm/failover2.png)  

3. 驗證之後，他們會完成每一個 VM 的移轉。 這樣會停止 VM 的複寫功能，並停止 VM 的 Site Recovery 計費。

    ![容錯移轉](./media/contoso-migration-rehost-vm/failover3.png)

**需要其他協助？**

- [了解](https://docs.microsoft.com/azure/site-recovery/tutorial-dr-drill-azure)如何執行測試容錯移轉。 
- [了解](https://docs.microsoft.com/azure/site-recovery/site-recovery-create-recovery-plans)如何建立復原計畫。
- [了解](https://docs.microsoft.com/azure/site-recovery/site-recovery-failover)如何容錯移轉至 Azure。

## <a name="clean-up-after-migration"></a>移轉之後進行清除

移轉完成之後，SmartHotel360 應用程式層現在會在 Azure VM 上執行。

現在，Contoso 必須完成以下的清除步驟：  

- 從 vCenter 庫存移除 WEBVM 機器。
- 從 vCenter 清查中移除 SQLVM 機器。
- 從本機備份工作移除 WEBVM 和 SQLVM。
- 更新內部文件以顯示 VM 的新位置和 IP 位址。
- 檢閱與 VM 互動的任何資源，並更新任何相關的設定或文件，以反映新的組態。

## <a name="review-the-deployment"></a>檢查部署

應用程式正在執行中，Contoso 現在必須能在 Azure 中發揮一切功能並保護它。

### <a name="security"></a>安全性

Contoso 安全性小組會檢查 Azure VM，判斷是否有任何的安全疑慮。

- 若要控制存取權，小組會檢閱 VM 的網路安全性群組 (NSG)。 NSG 是用來確保只有該應用程式被允許的流量，才可以與其連線。
- 小組也會考慮使用 Azure 磁碟加密和 KeyVault 來保護磁碟上的資料。

[深入了解](https://docs.microsoft.com/azure/security/azure-security-best-practices-vms#vm-authentication-and-access-control) VM 的安全措施。

## <a name="bcdr"></a>BCDR

針對商務持續性和災害復原 (BCDR)，Contoso 會採取下列動作：

- 保護資料安全：Contoso 會使用 Azure 備份服務來備份 VM 上的資料。 [深入了解](https://docs.microsoft.com/azure/backup/backup-introduction-to-azure-backup?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
- 保持應用程式啟動及執行：Contoso 會使用 Site Recovery，在 Azure 中將應用程式 VM 複寫至次要區域。 [深入了解](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-quickstart)。



### <a name="licensing-and-cost-optimization"></a>授權和成本最佳化

1. Contoso 為他們的 VM 準備了現有的授權，而且會利用 Azure Hybrid Benefit。  Contoso 會轉換現有的 Azure VM，以便充分利用這個定價。
2. Contoso 會啟用 Microsoft 子公司 Cloudyn 授權的 Azure 成本管理。 它是一種多雲端成本管理解決方案，可協助您使用和管理 Azure 和其他雲端資源。 [深入了解](https://docs.microsoft.com/azure/cost-management/overview) Azure 成本管理。 

## <a name="conclusion"></a>結論

在本文中，Contoso 會使用 Site Recovery 服務，將應用程式 VM 移轉至 Azure VM，以便重新裝載 SmartHotel360。 


## <a name="next-steps"></a>後續步驟

在系列文章的[下一篇文章](contoso-migration-rehost-vm-sql-ag.md)中，我們將說明 Contoso 如何將 SmartHotel360 應用程式前端 VM 重新裝載在 Azure VM 上，並將資料庫移轉至 Azure 中的 SQL Server AlwaysOn 可用性群組。

