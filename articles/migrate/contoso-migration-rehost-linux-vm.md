---
title: 將內部部署 Linux 應用程式重新裝載-遷移及重新裝載到 Azure VM | Microsoft Docs
description: 了解 Contoso 如何將內部部署 Linux 應用程式移轉至 Azure VM 進行重新裝載。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: raynew
ms.openlocfilehash: 4378351d6da6943485b7f382d96094543bfffa23
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44297718"
---
# <a name="contoso-migration-rehost-an-on-premises-linux-app-to-azure-vms"></a>Contoso 移轉：將內部部署 Linux 應用程式重新裝載至 Azure VM

本文說明 Contoso 如何將以 Linux 為基礎的內部部署服務台應用程式 (**osTicket**) 重新裝載至 Azure IaaS VM。

這份文件是一系列文章中的其中一篇，描述 Contoso 這家虛構的公司如何將其內部部署資源遷移至 Microsoft Azure 雲端。 整個系列文章中包含背景資訊以及一組案例，用以說明如何設定移轉基礎結構，以及執行不同類型的移轉。 案例的複雜性會提高。 我們會不定期新增其他文章。

**文章** | **詳細資料** | **狀態**
--- | --- | ---
[文章 1：概觀](contoso-migration-overview.md) | 系列文章的概觀、Contoso 的移轉策略，以及此系列中所使用的應用程式範例。 | 可用
[文章 2：部署 Azure 基礎結構](contoso-migration-infrastructure.md) | Contoso 會準備其內部部署基礎結構和其 Azure 基礎結構以進行移轉。 此系列中的所有移轉文章都使用相同的基礎結構。 | 可用
[文章 3：存取內部部署資源以移轉至 Azure](contoso-migration-assessment.md)  | Contoso 會評估其在 VMware 上執行的內部部署 SmartHotel360 應用程式。 Contoso 會使用 Azure Migrate 服務來評估應用程式 VM，並使用 Database Migration Assistant 來評估應用程式 SQL Server 資料庫。 | 可用
[文章 4：在 Azure VM 和 SQL Database 受控執行個體上重新裝載應用程式](contoso-migration-rehost-vm-sql-managed-instance.md) | Contoso 會為其內部部署 SmartHotel360 應用程式執行隨即移轉至 Azure 的作業。 Contoso 會使用 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview) 來移轉應用程式的前端 VM。 Contoso 會使用 [Azure 資料庫移轉服務](https://docs.microsoft.com/azure/dms/dms-overview)將應用程式資料庫移轉至 Azure SQL Database 受控執行個體。 | 可用   
[文章 5：在 Azure VM 上重新裝載應用程式](contoso-migration-rehost-vm.md) | Contoso 會使用 Site Recovery 服務，將其 SmartHotel360 應用程式 VM 移轉至 Azure VM。 | 可用
[文章 6：在 Azure VM 上和 SQL Server AlwaysOn 可用性群組中重新裝載應用程式](contoso-migration-rehost-vm-sql-ag.md) | Contoso 會移轉 SmartHotel360 應用程式。 Contoso 會使用 Site Recovery 來遷移應用程式的 VM。 它會使用資料庫移轉服務，將應用程式資料庫遷移至受到 SQL Server 可用性群組保護的 SQL Server 叢集。 | 可用 
文章 7：將 Linux 應用程式重新裝載至 Azure VM | Contoso 會使用 Site Recovery 服務，來完成將其 Linux osTicket 應用程式隨即移轉至 Azure VM 的作業 | 本文
[文章 8：在 Azure VM 和 Azure MySQL 上重新裝載 Linux 應用程式](contoso-migration-rehost-linux-vm-mysql.md) | Contoso 會使用 Azure Site Recovery 將 Linux osTicket 應用程式移轉至 Azure VM，並使用 MySQL Workbench 將應用程式資料庫移轉至 Azure MySQL Server 執行個體。 | 可用
[文章 9：在 Azure Web 應用程式和 Azure SQL 資料庫上重構應用程式](contoso-migration-refactor-web-app-sql.md) | Contoso 會將 SmartHotel360 應用程式移轉至 Azure Web 應用程式，以及使用 Database Migration Assistant 將應用程式資料庫移轉至 Azure SQL Server 執行個體 | 可用
[文章 10：在 Azure Web Apps 和 Azure MySQL 上重構 Linux 應用程式](contoso-migration-refactor-linux-app-service-mysql.md) | Contoso 會使用 Azure 流量管理員，將 Linux osTicket 應用程式移轉至多個 Azure 區域的 Azure Web 應用程式，與 GitHub 整合以進行持續傳遞。 Contoso 會將應用程式資料庫遷移至適用於 MySQL 的 Azure 資料庫執行個體。 | 可用 
[文章 11：重構 Azure DevOps Services 上的 TFS](contoso-migration-tfs-vsts.md) | Contoso 會將其內部部署 Team Foundation Server 部署，移轉至 Azure 中的 Azure DevOps Services。 | 可用
[文章 12：在 Azure 容器和 Azure SQL Database 上重新建構應用程式](contoso-migration-rearchitect-container-sql.md) | Contoso 會將其 SmartHotel 應用程式移轉至 Azure。 然後，它會重新建構應用程式 Web 層，作為在 Azure Service Fabric 中執行的 Windows 容器，以及具有 Azure SQL Database 的資料庫。 | 可用
[文章 13：在 Azure 中重建應用程式](contoso-migration-rebuild.md) | Contoso 會藉由使用各種 Azure 功能和服務 (包括 Azure App Service、Azure Kubernetes Service (AKS)、Azure Functions、Azure 認知服務及 Azure Cosmos DB) 重建其 SmartHotel 應用程式。 | 可用



在本文中，Contoso 會將在 Linux Apache MySQL PHP (LAMP) 上執行的兩層式 **osTicket** 應用程式遷移至 Azure。 應用程式 VM 將會使用 Azure Site Recovery 服務進行遷移。 如果想使用這個開放原始碼的應用程式，您可以從 [github](https://github.com/osTicket/osTicket) 進行下載。

## <a name="business-drivers"></a>商業動機

IT 領導小組與商務合作夥伴密切合作，以了解此次移轉所要實現的目標：

- **解決業務成長**：Contoso 的業務量日益增多，對內部部署系統和基礎結構造成了壓力。
- **限制風險**：服務台應用程式對 Contoso 的業務影響甚大。 Contoso 想要在亳無風險的情況下，將它移至 Azure。
- **擴充**：Contoso 不想立即變更應用程式。 只想要確保應用程式的穩定性。


## <a name="migration-goals"></a>移轉目標

Contoso 雲端小組已針對此次移轉擬定好各項目標，以決定最佳移轉方法：

- 移轉之後，應用程式不管是在 Azure 或內部部署 VMWare 環境中，都應具有相同的效能。  應用程式不管是在雲端中或在內部部署，都一樣重要。 
- Contoso 不想在這個應用程式上投注資源。  這對企業很重要，但以其目前的格式而言，Contoso 只想安全地移至雲端。
- Contoso 不想變更這個應用程式的 ops 模型。 而想要按照目前的方式，在雲端中與應用程式互動。
- Contoso 不想變更應用程式功能。 只會變更應用程式位置。
- Contoso 已完成幾次 Windows 應用程式移轉，所以想要了解如何在 Azure 中使用以 Linux 為基礎的基礎結構。


## <a name="solution-design"></a>解決方案設計

擬定好目標和需求後，Contoso 會設計和檢閱部署解決方案，並識別移轉程序，包括 Contoso 會用於移轉的 Azure 服務。

### <a name="current-app"></a>目前的應用程式

- OSTicket 應用程式橫跨兩層 VM (**OSTICKETWEB** 和 **OSTICKETMYSQL**)。
- VM 位於 VMware ESXi 主機 **contosohost1.contoso.com** (6.5 版)。
- VMware 環境是由 VM 上執行的 vCenter Server 6.5 (**vcenter.contoso.com**) 進行管理。
- Contoso 有內部部署資料中心 (**contoso-datacenter**) 以及內部部署網域控制站 (**contosodc1**)

### <a name="proposed-architecture"></a>建議的架構

- 因為應用程式為生產工作負載，所以 Azure 中的 VM 會位於生產資源群組 **ContosoRG** 中。
- VM 將會遷移到主要區域 (美國東部 2)，並且放在生產網路 (VNET-PROD-EUS2) 中：
    - Web VM 會位於前端子網路 (PROD-FE-EUS2) 中。
    - 資料庫 VM 會位於資料庫子網路 (PROD-DB-EUS2) 中。
- 移轉完成之後，將會解除委任 Contoso 資料中心的內部部署 VM。

![案例架構](./media/contoso-migration-rehost-linux-vm/architecture.png) 

### <a name="solution-review"></a>解決方案檢閱

Contoso 會透過比較一份優缺點清單，來評估建議設計。

**考量** | **詳細資料**
--- | ---
**優點** | 這兩個應用程式 VM 都會移至 Azure (不需變更)，讓移轉變簡單。<br/><br/> 由於 Contoso 對這兩個應用程式 VM 使用隨即轉移，所以應用程式資料庫不需要任何特殊設定或移轉工具。<br/><br/> Contoso 會保留 Azure 中應用程式 VM 的完整控制權。 </br>/br> 應用程式 VM 會執行 Ubuntu 16.04-TLS，這是經過背書的 Linux 散發套件。 [深入了解](https://docs.microsoft.com/azure/virtual-machines/linux/endorsed-distros)。
**缺點** | 應用程式的 Web 和資料層會保留單一容錯移轉點。 <br/><br/> Contoso 必須繼續支持此應用程式作為 Azure VM，而不是轉向適用於 MySQL 的 Azure App Service 與 Azure Database 等受控服務。<br/><br/> Contoso 意識到透過隨即轉移 VM 移轉讓事情保持簡單，它們就無法充分利用[適用於 MySQL 的 Azure 資料庫](https://docs.microsoft.com/azure/mysql/overview)所提供的功能 (內建的高可用性、可預測的效能、簡單調整、自動備份和內建安全性)。

### <a name="migration-process"></a>移轉程序

Contoso 會按照下列方式進行遷移：

1. 首先，Contoso 會設定 Azure 和部署 Site Recovery 所需的內部部署基礎結構。
2. 準備好 Azure 和內部部署元件之後，Contoso 會為 VM 設定和啟用複寫功能。
3. 在複寫功能正常運作之後，Contoso 會將 VM 容錯遷移至 Azure 以便進行遷移。

![移轉程序](./media/contoso-migration-rehost-linux-vm/migration-process.png)

### <a name="azure-services"></a>Azure 服務

**服務** | **說明** | **成本**
--- | --- | ---
[Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/) | 此服務可協調和管理 Azure VM 以及內部部署 VM 和實體伺服器的移轉和災害復原。  | 複寫至 Azure 的期間會產生 Azure 儲存體費用。  發生容錯移轉時，系統會建立 Azure VM 並產生費用。 [深入了解](https://azure.microsoft.com/pricing/details/site-recovery/)費用和定價。

 
## <a name="prerequisites"></a>必要條件

以下是 Contoso 在此案例中應該準備好的事項。

**需求** | **詳細資料**
--- | ---
**Azure 訂用帳戶** | Contoso 已在本系列稍早的文章中建立訂用帳戶。 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。<br/><br/> 如果您建立免費帳戶，您就是訂用帳戶的管理員，並可執行所有動作。<br/><br/> 如果您使用現有訂用帳戶，而且您不是系統管理員，則需要與系統管理員合作，讓其指派擁有者或參與者權限給您。<br/><br/> 如果您需要更細微的權限，請檢閱[此文章](../site-recovery/site-recovery-role-based-linked-access-control.md)。 
**Azure 基礎結構** | Contoso 會如[適用於移轉的 Azure 基礎結構](contoso-migration-infrastructure.md)中所述，設定其 Azure 基礎結構。<br/><br/> 深入了解 Site Recovery 具體的[網路](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#network)和[儲存體](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#storage)需求。
**內部部署伺服器** | 內部部署 vCenter 伺服器應執行 5.5、6.0 或 6.5 版<br/><br/> 執行 5.5、6.0 或 6.5 版的 ESXi 主機<br/><br/> 一或多部在 ESXi 主機上執行的 VMware VM。
**內部部署 VM** | [檢閱 Linux 機器](https://docs.microsoft.com//azure/site-recovery/vmware-physical-azure-support-matrix#replicated-machines)，這些機器支援透過 Site Recovery 移轉。<br/><br/> 確認支援的 [Linux 檔案和儲存系統](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#linux-file-systemsguest-storage)。<br/><br/> VM 必須符合 [Azure 需求](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#azure-vm-requirements)。


## <a name="scenario-steps"></a>案例步驟

以下是 Contoso 完成移轉的方式：

> [!div class="checklist"]
> * **步驟 1：針對 Site Recovery 準備 Azure**：Contoso 會建立一個 Azure 儲存體帳戶來保存複寫的資料，還會建立一個復原服務保存庫。
> * **步驟 2：針對 Site Recovery 準備內部部署 VMware**：Contoso 會準備要用於探索 VM 和安裝代理程式的帳戶，以及準備在容錯移轉後連線至 Azure VM。
> * **步驟 3：複寫 VM**：Contoso 會設定來源和目標移轉環境、建立複寫原則，並開始將 VM 複寫至 Azure 儲存體。
> * **步驟 4：使用 Site Recovery 遷移 VM**：Contoso 會執行測試容錯移轉，確定一切都能正常運作，然後執行完整的容錯移轉，以便將 VM 遷移至 Azure。


## <a name="step-1-prepare-azure-for-the-site-recovery-service"></a>步驟 1：針對 Site Recovery 服務準備 Azure

Contoso 需要幾個可供 Site Recovery 使用的 Azure 元件：

- 新的 Azure 儲存體帳戶，用來保存複寫的資料。 
- Azure 中的復原服務保存庫。
- 一個 VNet，容錯移轉的資源位於其中。 Contoso 已經在 [Azure 基礎結構部署](contoso-migration-infrastructure.md)期間建立 VNet ，所以他們只需要建立儲存體帳戶和保存庫。

1. Contoso 管理員會在美國東部 2 區域中建立 Azure 儲存體帳戶 (contosovmsacc20180528)。

    - 儲存體帳戶與復原服務保存庫必須位於相同的區域。
    - 他們會使用一般用途的帳戶，並配備標準儲存體和 LRS 複寫。

    ![Site Recovery 儲存體](./media/contoso-migration-rehost-linux-vm/asr-storage.png)

2. 網路和儲存體帳戶準備就緒之後，他們會建立保存庫 (ContosoMigrationVault)，然後將它放在美國東部 2 主要區域的 **ContosoFailoverRG** 資源群組中。

    ![復原服務保存庫](./media/contoso-migration-rehost-linux-vm/asr-vault.png)


**需要其他協助？**

[了解如何](https://docs.microsoft.com/azure/site-recovery/tutorial-prepare-azure)針對 Site Recovery 設定 Azure。


## <a name="step-2-prepare-on-premises-vmware-for-site-recovery"></a>步驟 2：針對 Site Recovery 準備內部部署 VMware

Contoso 管理員會按照下列方式，準備內部部署 VMware 基礎結構：

- 他們會在 vCenter Server 或 vSphere ESXi 主機上建立一個帳戶，將 VM 探索自動化。
- 他們會建立一個帳戶，允許在您想要複寫的 VMware VM 上自動安裝行動服務。
- 他們會準備內部部署 VM，以便在移轉之後於建立時連線到 Azure VM。


### <a name="prepare-an-account-for-automatic-discovery"></a>準備帳戶以進行自動探索

Site Recovery 需要存取 VMware 伺服器才能：

- 自動探索 VM。 需要至少一個唯讀帳戶。
- 協調複寫、容錯移轉和容錯回復。 您需要可執行建立和移除磁碟以及開啟 VM 等作業的帳戶。

Contoso 管理員會依照下列方式設定帳戶：

1. 他們在 vCenter 層級建立一個角色。
2. 他們會指派必要權限給角色。



### <a name="prepare-an-account-for-mobility-service-installation"></a>準備一個用來安裝行動服務的帳戶

行動服務必須安裝在將要遷移的 Linux VM 上。

- 當 VM 的複寫功能啟用時，Site Recovery 可以執行此元件的自動推送安裝。
- 若要自動推送安裝，他們必須準備一個可供 Site Recovery 用來存取 VM 的帳戶。
- 帳戶詳細資料會在複寫設定期間輸入。 
- 此帳戶可以是網域帳戶或本機帳戶，而且擁有在 VM 上安裝的權限。


### <a name="prepare-to-connect-to-azure-vms-after-failover"></a>準備在容錯移轉後連接到 Azure VM

容錯移轉至 Azure 之後，Contoso 希望能夠連線到 Azure 中複寫的 VM。 若要這麼做，Contoso 管理員還需執行一些作業：

- 若要透過網際網路存取 Azure VM，請在移轉前，先在內部部署 Linux VM 上啟用 SSH。  若為 Ubuntu，可以使用下列命令來完成此動作：**Sudo apt-get ssh install -y**。
- 執行移轉 (容錯移轉) 之後，可以勾選 [開機診斷] 來檢視 VM 的螢幕擷取畫面。
- 如果這個不起任何作用，則需要檢查 VM 是否正常執行，並且檢閱這些[疑難排解祕訣](http://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx)。


**需要其他協助？**

- [了解](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial-prepare-on-premises#prepare-an-account-for-automatic-discovery)如何建立和指派自動探索所需的角色。
- [深入了解](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial-prepare-on-premises#prepare-an-account-for-mobility-service-installation)建立行動服務推送安裝的帳戶。


## <a name="step-3-replicate-the-on-premises-vms"></a>步驟 3：複寫內部部署 VM

Contoso 管理員必須先設定及啟用複寫，才可以將 Web VM 遷移至 Azure。

### <a name="set-a-protection-goal"></a>設定保護目標

1. 在保存庫的保存庫名稱 (ContosoVMVault) 底下，他們會設定一個複寫目標 ([開始使用] > [Site Recovery] > [準備基礎結構]。
2. 他們會指定自己的機器都位於內部部署環境、都是 VMware VM，以及想要複寫至 Azure。
    ![複寫目標](./media/contoso-migration-rehost-linux-vm/replication-goal.png)

### <a name="confirm-deployment-planning"></a>確認部署規劃

若要繼續，他們會選取 [是，我已經完成]，確認已經完成部署規劃。 在這個案例中，Contoso 只會移轉單一 VM，所以不需要部署規劃。

### <a name="set-up-the-source-environment"></a>設定來源環境

Contoso 管理員現在必須設定來源環境。 為此，他們下載 OVF 範本，然後用它將 Site Recovery 組態伺服器部署為高可用性、內部部署的 VMware VM。 當組態伺服器開始執行之後，他們將它註冊到保存庫。

組態伺服器會執行多項元件：

- 組態伺服器元件會協調內部部署與 Azure 之間的通訊，以及管理資料複寫。
- 處理序伺服器，可作為複寫閘道。 負責接收複寫資料，以快取、壓縮和加密進行最佳化，然後將複寫資料傳送至 Azure 儲存體。
- 處理序伺服器也會在您要複寫的 VM 上安裝行動服務，並且在內部部署 VMware VM 上執行自動探索。

Contoso 管理員會依照下列方式執行這些步驟：

1. 他們會從 [準備基礎結構] > [來源] > [組態伺服器] 下載 OVF 範本。
    
    ![下載 OVF](./media/contoso-migration-rehost-linux-vm/add-cs.png)

2. 他們會將範本匯入 VMware，以便建立 VM，然後部署 VM。

    ![OVF 範本](./media/contoso-migration-rehost-linux-vm/vcenter-wizard.png)

3. 當他們第一次啟動 VM 時，VM 會在開機後進入 Windows Server 2016 安裝程序。 他們接受授權合約，並輸入系統管理員密碼。
4. 安裝完成之後，他們會以系統管理員身分登入 VM。 第一次登入時，根據預設，Azure Site Recovery 設定工具會啟動。
5. 在此工具中，他們指定一個名稱，用以在保存庫中註冊組態伺服器。
6. 此工具會檢查 VM 是否可連線到 Azure。 連線建立後，他們登入 Azure 訂用帳戶。 認證必須能夠存取您要在其中註冊組態伺服器的保存庫。

    ![註冊設定伺服器](./media/contoso-migration-rehost-linux-vm/config-server-register2.png)

7. 此工具會執行一些設定工作，而後重新開機。
8. 他們再次登入機器，組態伺服器管理精靈隨即自動啟動。
9. 在此精靈中，他們選取要接收複寫流量的 NIC。 這項設定在完成之後即無法變更。
10. 他們選取訂用帳戶、資源群組，以及在其中註冊組態伺服器的保存庫。

    ![保存庫](./media/contoso-migration-rehost-linux-vm/cswiz1.png) 

11. 接著，他們下載並安裝 MySQL Server 和 VMWare PowerCLI。 
12. 驗證之後，他們指定 vCenter 伺服器或 vSphere 主機的 FQDN 或 IP 位址。 他們會保留預設的連接埠，並且為 vCenter Server 指定易記名稱。
13. 他們會指定為自動探索而建立的帳戶，以及應用於自動安裝行動服務的認證。

    ![vCenter](./media/contoso-migration-rehost-linux-vm/cswiz2.png)

14. 註冊完成後，他們會在 Azure 入口網站中檢查組態伺服器和 VMware 伺服器，確認這兩個項目都已列在保存庫中的 [來源] 頁面上。 探索可能需要 15 分鐘以上的時間。 
15. Site Recovery 接下來會連線至 VMware 伺服器並且探索 VM。

### <a name="set-up-the-target"></a>設定目標

Contoso 管理員現在會設定目標複寫設定。

1. 在 [準備基礎結構] > [目標] 中，他們選取目標設定。
2. Site Recovery 會確認在指定的目標中有 Azure 儲存體帳戶和網路。

### <a name="create-a-replication-policy"></a>建立複寫原則

設定好來源和目標之後，他們即可建立複寫原則。

1. 在 [準備基礎結構] > [複寫設定] > [複寫原則] >  [建立和關聯] 中，他們建立 **ContosoMigrationPolicy** 原則。
2. 他們使用預設設定：
    - **RPO 閾值**：預設值是 60 分鐘。 這個值可定義復原點的建立頻率。 連續複寫超過此限制時會產生警示。
    - **復原點保留**。 預設值為 24 小時。 此值會指定每個復原點的保留週期有多長。 複寫的 VM 可以還原至一個週期內的任何時間點。
    - **應用程式一致快照頻率**。 預設值是一小時。 此值會指定應用程式一致快照的建立頻率。
 
        ![建立複寫原則](./media/contoso-migration-rehost-linux-vm/replication-policy.png)

5. 此原則會自動與設定伺服器產生關聯。 

    ![關聯複寫原則](./media/contoso-migration-rehost-linux-vm/replication-policy2.png)

**需要其他協助？**

- 您可以在[設定內部部署 VMware VM 的災害復原](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial)中，閱讀所有步驟的詳細逐步解說。
- 您會得到詳細的操作說明，協助您[設定來源環境](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-source)、[部署組態伺服器](https://docs.microsoft.com/azure/site-recovery/vmware-azure-deploy-configuration-server)以及[指定複寫設定](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-replication)。
- [深入了解](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-linux)適用於 Linux 的 Azure Guest 代理程式。

**需要其他協助？**

- 您可以在[設定內部部署 VMware VM 的災害復原](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial)中，閱讀所有步驟的詳細逐步解說。
- 您會得到詳細的操作說明，協助您[設定來源環境](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-source)、[部署組態伺服器](https://docs.microsoft.com/azure/site-recovery/vmware-azure-deploy-configuration-server)以及[指定複寫設定](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-replication)。
- [深入了解](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-linux)適用於 Linux 的 Azure Guest 代理程式。



### <a name="enable-replication-for-osticketweb"></a>為 OSTICKETWEB 啟用 VM 複寫

Contoso 管理員現在可以開始複寫 **OSTICKETWEB** VM。

1. 在 [複寫應用程式] > [來源] > [+複寫] 中，他們選取來源設定。
2. 他們表示想要啟用虛擬機器、選取來源設定，包括 vCenter Server 和組態伺服器。

    ![啟用複寫](./media/contoso-migration-rehost-linux-vm/enable-replication-source.png)

3. 他們會指定目標設定，包括 Azure VM 在容錯移轉之後所在的資源群組和 VNet，以及要儲存所複寫資料的儲存體帳戶。

     ![啟用複寫](./media/contoso-migration-rehost-linux-vm/enable-replication2.png)

3. 他們會選取 **OSTICKETWEB** VM 進行複寫。 

    - 在這個階段，他們只會選取 **OSTICKETWEB**，因為必須選取 VNet 和子網路，而且 VM 不在相同的子網路中。
    - 啟用 VM 複寫時，Site Recovery 會自動安裝行動服務。

    ![啟用複寫](./media/contoso-migration-rehost-linux-vm/enable-replication3.png)

4. 在 VM 屬性中，他們會選取處理序伺服器所用的帳戶，以在電腦上自動安裝行動服務。

     ![行動服務](./media/contoso-migration-rehost-linux-vm/linux-mobility.png)

5. 在 [複寫設定] > [設定複寫設定] 中，他們會確認已套用正確的複寫原則，然後選取 [啟用複寫]。
6.  他們在 [作業] 中追踨複寫進度。 執行 [完成保護] 作業之後，機器即準備好進行容錯移轉。



### <a name="enable-replication-for-osticketmysql"></a>為 OSTICKETMYSQL 啟用 VM 複寫

Contoso 管理員現在可以開始複寫 **OSTICKETMYSQL**。

1. 在 [複寫應用程式] > [來源] > [+複寫] 中，他們會選取來源和目標設定。
2. 他們會選取 **OSTICKETMYSQL** VM 進行複寫，然後選取要用於行動服務安裝的帳戶。

    ![啟用複寫](./media/contoso-migration-rehost-linux-vm/mysql-enable.png)

3. 他們會套用針對 OSTICKETWEB 所使用的同一套複寫原則，並且啟用複寫。  

**需要其他協助？**

您可以在[啟用複寫](https://docs.microsoft.com/azure/site-recovery/vmware-azure-enable-replication)中閱讀所有步驟的完整逐步解說。


## <a name="step-4-migrate-the-vms"></a>步驟 4：遷移 VM 

Contoso 管理員會執行快速測試容錯移轉，然後遷移 VM。

### <a name="run-a-test-failover"></a>執行測試容錯移轉

在移轉之前執行測試容錯移轉，有助於確保一切運作正常。 

1. 他們會針對最新的可用時間點 (**最近處理的**) 執行一次測試容錯移轉。
2. 如果想在觸發容錯移轉之前，讓 Site Recovery 嘗試將來源 VM 關機，請選取 [先將機器關機再開始容錯移轉]。 即使關機失敗，仍會繼續容錯移轉。 
3. 執行測試容錯移轉： 
    - 系統會執行先決條件檢查，以確保所有移轉所需的條件都已準備就緒。
    - 容錯移轉會處理資料，以便建立 Azure VM。 如果選取了最新的復原點，則會根據資料建立復原點。
    - 將會使用先前步驟中處理的資料來建立 Azure VM。
3. 容錯移轉完成後，Azure VM 複本會出現在 Azure 入口網站中。 他們會確認 VM 為適當的大小、其已連線到正確的網路，而且正在執行中。 
4. 確認之後，他們可以清除容錯移轉，並記錄與儲存任何觀察到的結果。

### <a name="create-and-customize-a-recovery-plan"></a>建立並自訂復原方案

 驗證測試容錯移轉如預期般正常運作之後，Contoso 管理員會針對移轉建立復原方案。 

- 復原方案會指定容錯移轉的發生順序，並指出 Azure VM 如何在 Azure 中正常運作。
- 由於他們想要遷移兩層式應用程式，他們會自訂復原方案，因此資料 VM (SQLVM) 會比前端 (WEBVM) 先啟動。


1. 在 [復原方案 (Site Recovery)] > [+復原方案] 中，他們會建立一個方案，並將 VM 加入其中。

    ![復原計畫](./media/contoso-migration-rehost-linux-vm/recovery-plan.png)

2. 建立方案之後，他們會選取它自訂進行自訂 ([復原方案] > [OsTicketMigrationPlan] > [自訂])。
3.  他們會從 [群組 1：啟動] 移除 **OSTICKETWEB**。  這樣可確保第一個啟動動作只會影響 **OSTICKETMYSQL**。

    ![復原群組](./media/contoso-migration-rehost-linux-vm/recovery-group1.png)

4.  在 [+群組] > [新增受保護項目] 中，他們會將 **OSTICKETWEB** 新增至 [群組 2：開始]。  他們需要將這些項目放在兩個不同的群組中。

    ![復原群組](./media/contoso-migration-rehost-linux-vm/recovery-group2.png)


### <a name="migrate-the-vms"></a>遷移 VM


Contoso 管理員現在已準備好在復原方案上執行容錯移轉，以便遷移 VM。

1. 他們會選取方案 > [容錯移轉]。
2.  他們會選取容錯移轉至最新的復原點，然後指定 Site Recovery 應該在嘗試觸發容錯移轉之前，先關閉內部部署 VM。 他們可以在 [作業] 頁面上追蹤容錯移轉進度。

    ![容錯移轉](./media/contoso-migration-rehost-vm/failover1.png)

3. 在容錯移轉期間，vCenter Server 會發出命令，以停止在 ESXi 主機上執行的兩部 VM。

    ![容錯移轉](./media/contoso-migration-rehost-linux-vm/vcenter-failover.png)

3. 容錯移轉之後，他們可以確認 Azure VM 會如預期般出現在 Azure 入口網站中。

    ![容錯移轉](./media/contoso-migration-rehost-linux-vm/failover2.png)  

3. 在 Azure 中確認 VM 之後，他們會完成移轉，以完成每部 VM 的移轉程序。 這樣會停止 VM 的複寫功能，並停止 VM 的 Site Recovery 計費。

    ![容錯移轉](./media/contoso-migration-rehost-linux-vm/failover3.png)


### <a name="connect-the-vm-to-the-database"></a>將 VM 連線到資料庫

在移轉程序的最後一個步驟中，Contoso 管理員會將應用程式的連接字串更新為指向在 **OSTICKETMYSQL** VM 上執行的應用程式資料庫。 

1. 他們會使用 Putty 或另一個 SSH 用戶端，對 **OSTICKETWEB** VM 進行 SSH 連線。 VM 為私用的，所以會使用私人 IP 位址進行連線。

    ![連線到資料庫](./media/contoso-migration-rehost-linux-vm/db-connect.png)  

    ![連線到資料庫](./media/contoso-migration-rehost-linux-vm/db-connect2.png)  

2. 他們必須確定 **OSTICKETWEB** VM 可以與 **OSTICKETMYSQL** VM 通訊。 組態目前已利用內部部署 IP 位址 172.16.0.43 進行硬式編碼。

    **更新之前**
    
    ![更新 IP](./media/contoso-migration-rehost-linux-vm/update-ip1.png)  

    **更新之後**
    
    ![更新 IP](./media/contoso-migration-rehost-linux-vm/update-ip2.png) 
    
3. 他們會使用 **systemctl restart apache2** 重新啟動服務。

    ![重新啟動](./media/contoso-migration-rehost-linux-vm/restart.png) 

4. 最後，他們會在其中一個 Contoso 網域控制站上，更新 **OSTICKETWEB** 和 **OSTICKETMYSQL** 的 DNS 記錄。

    ![更新 DNS](./media/contoso-migration-rehost-linux-vm-mysql/update-dns.png) 

    ![更新 DNS](./media/contoso-migration-rehost-linux-vm-mysql/update-dns.png) 

**需要其他協助？**

- [了解](https://docs.microsoft.com/azure/site-recovery/tutorial-dr-drill-azure)如何執行測試容錯移轉。 
- [了解](https://docs.microsoft.com/azure/site-recovery/site-recovery-create-recovery-plans)如何建立復原計畫。
- [了解](https://docs.microsoft.com/azure/site-recovery/site-recovery-failover)如何容錯移轉至 Azure。

## <a name="clean-up-after-migration"></a>移轉之後進行清除

移轉完成之後，osTicket 應用程式層現在會在 Azure VM 上執行。

現在，Contoso 必須進行清除，如下所示： 

- 從 vCenter 清查中移除內部部署 VM。
- 從本機備份作業中移除內部部署 VM。
- 更新其內部文件以顯示新位置，以及 OSTICKETWEB 和 OSTICKETMYSQL 的 IP 位址。
- 檢閱與 VM 互動的任何資源，並更新任何相關的設定或文件，以反映新的組態。
- Contoso 使用了 Azure Migrate 服務搭配相依性對應，評定要進行遷移的 VM。 基於此目的，管理員應該從 VM 中移除 Microsoft Monitoring Agent 以及他們所安裝的 Dependency Agent。

## <a name="review-the-deployment"></a>檢閱部署

應用程式正在執行中，Contoso 必須能發揮一切功能並保護其新的基礎結構。

### <a name="security"></a>安全性

Contoso 安全性小組會檢閱 OSTICKETWEB 和 OSTICKETMYSQLVM，判斷是否有任何安全疑慮。

- 小組會檢閱 VM 的網路安全性群組 (NSG) 來控制存取權。 NSG 可用來確保只可以傳遞該應用程式允許的流量。
- 小組也會考慮使用磁碟加密和 Azure KeyVault 來保護 VM 磁碟上的資料。

[深入了解](https://docs.microsoft.com/azure/security/azure-security-best-practices-vms#vm-authentication-and-access-control) VM 的安全措施。

### <a name="bcdr"></a>BCDR

針對商務持續性和災害復原，Contoso 會採取下列動作：

- **保護資料安全**：Contoso 會使用 Azure 備份服務來備份 VM 上的資料。 [深入了解](https://docs.microsoft.com/azure/backup/backup-introduction-to-azure-backup?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
- **保持應用程式啟動及執行**：Contoso 會使用 Site Recovery，在 Azure 中將應用程式 VM 複寫至次要區域。 [深入了解](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-quickstart)。

### <a name="licensing-and-cost-optimization"></a>授權和成本最佳化

- 部署資源之後，Contoso 會指派 [Azure 基礎結構部署](contoso-migration-infrastructure.md#set-up-tagging)期間定義的 Azure 標記。
- Contoso 沒有 Ubuntu 伺服器相關授權問題。
- Contoso 會啟用 Microsoft 子公司 Cloudyn 授權的 Azure 成本管理。 它是一種多雲端成本管理解決方案，可協助您利用和管理 Azure 和其他雲端資源。  [深入了解](https://docs.microsoft.com/azure/cost-management/overview) Azure 成本管理。 


## <a name="next-steps"></a>後續步驟

我們在本文中介紹了 Contoso 如何使用 Azure Site Recovery，將橫跨兩層 Linux VM 的內部部署服務台應用程式遷移至 Azure IaaS VM。

在這系列的下一篇文章中，我們將會說明 Contoso 如何將相同的服務台應用程式遷移至 Azure。 這次 Contoso 會使用 Site Recovery 來遷移應用程式的前端 VM，然後使用 MySQL Workbench 工具將應用程式資料庫 (使用備份與還原) 遷移至適用於 MySQL 的 Azure 資料庫。 [開始使用](contoso-migration-rehost-linux-vm-mysql.md)。
