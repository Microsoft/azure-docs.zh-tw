---
title: 將 Contoso 內部部署應用程式遷移至 Azure VM 和 Azure SQL Database 受控執行個體，以重新裝載此應用程式 | Microsoft Docs
description: 了解 Contoso 如何在 Azure VM 和 Azure SQL Database 受控執行個體上重新裝載內部部署應用程式。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: raynew
ms.openlocfilehash: 12baa21c8661012cd7ef96217724150a3d8c56f3
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44303419"
---
# <a name="contoso-migration-rehost-an-on-premises-app-on-an-azure-vm-and-sql-database-managed-instance"></a>Contoso 移轉：在 Azure VM 和 SQL Database 受控執行個體上重新裝載內部部署應用程式

在本文中，Contoso 會使用 Azure Site Recovery 服務，將其 SmartHotel360 應用程式前端 VM 移轉至 Azure。 Contoso 也會將應用程式資料庫遷移至 Azure SQL Database 受控執行個體。

> [!NOTE]
> Azure SQL Database 受控執行個體目前處於預覽狀態。

本文是一系列文章中的一篇，描述 Contoso 這家虛構的公司如何將其內部部署資源遷移至 Microsoft Azure 雲端。 整個系列文章中包含背景資訊以及一系列案例，用以說明如何設定移轉基礎結構，以及執行不同類型的移轉。 案例的複雜性會提高。 隨著時間的推移，此系列會新增其他文章。


**文章** | **詳細資料** | **狀態**
--- | --- | ---
[文章 1：概觀](contoso-migration-overview.md) | Contoso 移轉策略概觀、文章系列，以及此系列中所使用的應用程式範例。 | 可用
[文章 2：部署 Azure 基礎結構](contoso-migration-infrastructure.md) | Contoso 會準備其內部部署基礎結構和其 Azure 基礎結構以進行移轉。 此系列中的所有移轉文章都使用相同的基礎結構。 | 可用
[文章 3：存取內部部署資源以移轉至 Azure](contoso-migration-assessment.md) | Contoso 會評定其在 VMware 上執行的內部部署兩層式 SmartHotel 應用程式。 Contoso 會藉由使用 [Azure Migrate](migrate-overview.md) 服務來評定應用程式的 VM。 Contoso 會藉由使用 [Data Migration Assistant](https://docs.microsoft.com/sql/dma/dma-overview?view=sql-server-2017) 來評定應用程式的 SQL Server 資料庫。 | 可用
文章 4：在 Azure VM 和 SQL Database 受控執行個體上重新裝載應用程式 | Contoso 會為其內部部署 SmartHotel 應用程式執行隨即移轉至 Azure 的作業。 Contoso 會藉由使用 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview) 來遷移應用程式的前端 VM。 Contoso 會藉由使用 [Azure 資料庫移轉服務](https://docs.microsoft.com/azure/dms/dms-overview)將應用程式資料庫遷移至 Azure SQL Database 受控執行個體。 | 本文
[文章 5：在 Azure VM 上重新裝載應用程式](contoso-migration-rehost-vm.md) | Contoso 會藉由使用 Site Recovery 服務將其 SmartHotel 應用程式 VM 遷移至 Azure VM。 | 可用
[文章 6：在 Azure VM 上和 SQL Server AlwaysOn 可用性群組中重新裝載應用程式](contoso-migration-rehost-vm-sql-ag.md) | Contoso 會遷移 SmartHotel 應用程式。 Contoso 會使用 Site Recovery 來遷移應用程式的 VM。 它會使用資料庫移轉服務，將應用程式資料庫遷移至受到 SQL Server 可用性群組保護的 SQL Server 叢集。 | 可用
[文章 7：將 Linux 應用程式重新裝載至 Azure VM](contoso-migration-rehost-linux-vm.md) | Contoso 會使用 Site Recovery 來完成將其 Linux osTicket 應用程式隨即移轉至 Azure VM 的作業。 | 可用
[文章 8：在 Azure VM 和適用於 MySQL 的 Azure 資料庫上重新裝載 Linux 應用程式](contoso-migration-rehost-linux-vm-mysql.md) | Contoso 會藉由使用 Site Recovery 將其 Linux osTicket 應用程式遷移至 Azure VM。 它會使用 MySQL Workbench 將應用程式資料庫遷移至適用於 MySQL 的 Azure 資料庫。 | 可用
[文章 9：在 Azure Web 應用程式和 Azure SQL Database 中重構應用程式](contoso-migration-refactor-web-app-sql.md) | Contoso 會將其 SmartHotel 應用程式遷移至 Azure Web 應用程式，以及將應用程式資料庫遷移至 Azure SQL Server 執行個體。 | 可用
[文章 10：在 Azure Web 應用程式和適用於 MySQL 的 Azure 資料庫中重構 Linux 應用程式](contoso-migration-refactor-linux-app-service-mysql.md) | Contoso 會將其 Linux osTicket 應用程式遷移至多個網站上的 Azure Web 應用程式。 Web 應用程式會與 GitHub 整合以提供持續傳遞能力。 Contoso 會將應用程式資料庫遷移至適用於 MySQL 的 Azure 資料庫執行個體。 | 可用
[文章 11：在 Azure DevOps Services 上重構 Team Foundation Server](contoso-migration-tfs-vsts.md) | Contoso 會將其內部部署 Team Foundation Server 部署移轉至 Azure 中的 Azure DevOps Services。 | 可用
[文章 12：在 Azure 容器和 Azure SQL Database 中重新建構應用程式](contoso-migration-rearchitect-container-sql.md) | Contoso 會將其 SmartHotel 應用程式遷移至 Azure，然後重新建構該應用程式。 Contoso 會將應用程式的 Web 層重新建構為 Windows 容器，以及使用 Azure SQL Database 重新建構應用程式資料庫。 | 可用
[文章 13：在 Azure 中重建應用程式](contoso-migration-rebuild.md) | Contoso 會藉由使用各種 Azure 功能和服務 (包括 Azure App Service、Azure Kubernetes Service、Azure Functions、Azure 認知服務及 Azure Cosmos DB) 重建其 SmartHotel 應用程式。 | 可用




您可以從 [GitHub](https://github.com/Microsoft/SmartHotel360) 下載本文中所用的範例 SmartHotel360 應用程式。



## <a name="business-drivers"></a>商業動機

Contoso 的 IT 領導小組已與該公司的企業合作夥伴密切合作，以了解這些企業想從此次移轉實現什麼目標：

- **解決業務成長**：Contoso 正在成長。 因此，該公司的內部部署系統和基礎結構所承受的壓力變大了。
- **提高效率**：Contoso 必須移除不必要的程序，並簡化其開發人員和使用者的程序。 該公司需要快速且不浪費時間或金錢的 IT 服務，以便能夠更快滿足客戶需求。
- **提高靈活性**：Contoso IT 必須能夠更快因應企業的需求。 其因應速度必須能夠比市場變化更快，才能讓該公司在全球經濟中獲致成功。 Contoso 的 IT 小組不得礙事，或成為企業的絆腳石。
- **調整**：隨著該公司的業務順利成長，Contoso IT 必須提供能夠同步成長的系統。

## <a name="migration-goals"></a>移轉目標

Contoso 雲端小組已確定此次移轉的目標。 該公司會使用移轉目標來判斷最合適的移轉方法。

- 移轉過後，應用程式不管是在 Azure 或 Contoso 的內部部署 VMWare 環境中，都應具有相同的效能。 移到雲端，並不表示應用程式效能比較不重要。
- Contoso 不想在這個應用程式上投注資源。 此應用程式對企業很重要，但是 Contoso 只想要以目前的格式將應用程式移至雲端。
- 在應用程式遷移之後，資料庫管理工作應會減到最少。
- Contoso 不想對此應用程式使用 Azure SQL Database， 且正在尋求替代方案。


## <a name="solution-design"></a>解決方案設計

擬定好目標和需求後，Contoso 會設計和檢閱部署解決方案，並識別移轉程序，包括將用於移轉的 Azure 服務。

### <a name="current-architecture"></a>目前架構 

- Contoso 有一個主要的資料中心 (**contoso-datacenter**)。 該資料中心位於美國東部的紐約市。
- Contoso 在全美另有三家地區性分公司。
- 主要資料中心透過光纖都會乙太網路連線 (500 MBps) 連到網際網路。
- 每家分公司皆使用企業級連線從本機連到網際網路，並透過 IPsec VPN 通道連回主要資料中心。 此設定可讓 Contoso 的整個網路永久連線，並將網際網路連線最佳化。
- 主要資料中心已透過 VMware 完全虛擬化。 Contoso 有兩部 ESXi 6.5 虛擬化主機，均由 vCenter Server 6.5 管理。
- Contoso 使用 Active Directory 來管理身分識別。 Contoso 會使用內部網路上的 DNS 伺服器。
- Contoso 有一個內部部署網域控制站 (**contosodc1**)。
- 網域控制站會在 VMware VM 上執行。 地區分公司的網域控制站會在實體伺服器上執行。
- SmartHotel360 應用程式橫跨兩層 VM (**WEBVM** 和 **SQLVM**)，而且位於 VMware ESXi 6.5 版主機 (**contosohost1.contoso.com**)。 
- VMware 環境是由 VM 上執行的 vCenter Server 6.5 (**vcenter.contoso.com**) 進行管理。

![目前的 Contoso 架構](./media/contoso-migration-rehost-vm-sql-managed-instance/contoso-architecture.png)  

### <a name="proposed-architecture"></a>建議的架構

在此案例中，Contoso 想要遷移其兩層式內部部署旅遊應用程式，如下所示：

- 將應用程式資料庫 (**SmartHotelDB**) 遷移到 Azure SQL Database 受控執行個體。
- 將前端 **WebVM** 遷移到 Azure VM。
- 移轉完成後，將會解除委任 Contoso 資料中心的內部部署 VM。

![案例架構](media/contoso-migration-rehost-vm-sql-managed-instance/architecture.png) 

### <a name="database-considerations"></a>資料庫考量

在解決方案設計過程中，Contoso 會進行 Azure SQL Database 與 SQL Server 受控執行個體的功能比較。 下列考量協助他們決定使用受控執行個體。

- 受控執行個體的目標是為最新版內部部署 SQL Server 提供幾乎 100% 的相容性。 如果在內部部署環境或在 IaaS VM 上執行 SQL Server 的客戶想要在幾乎不需要變更設計的情況下，將其應用程式遷移至完全受控服務，Microsoft 建議使用受控執行個體。
- Contoso 計劃將大量應用程式從內部部署環境遷移到 IaaS。 有許多都是由 ISV 提供。 Contoso 意識到使用受控執行個體將有助於確保這些應用程式的資料庫相容性，而不是使用可能不支援的 SQL Database。
- Contoso 可以使用完全自動化的資料移轉服務 (DMS)，直接執行隨即轉移至受控執行個體。 備妥此服務，Contoso 可以將它重複使用於未來的資料庫移轉。
- SQL 受控執行個體支援 SQL Server Agent，也就是 SmartHotel360 應用程式的重要問題。 Contoso 需要此相容性，否則就必須重新設計應用程式所需的維護方案。
- Contoso 可以透過軟體保證使用適用於 SQL Server 的 Azure Hybrid Benefit，以折扣優惠在 SQL Database 受控執行個體上交換執行個體的現有授權。 這可讓 Contoso 最多節省 30% 的受控執行個體。
- 受控執行個體完全包含在虛擬網路中，因此可為 Contoso 的資料提供高層次的隔離和安全性。 Contoso 可享有公用雲端的優勢，同時讓環境與公用網際網路保持隔離。
- 受控執行個體支援許多安全性功能，包括永遠加密、動態資料遮罩、資料列層級的安全性，以及威脅偵測。


### <a name="solution-review"></a>解決方案檢閱

Contoso 會透過比較一份優缺點清單，來評估建議設計。

**考量** | **詳細資料**
--- | ---
**優點** |  WEBVM 會移至 Azure (不需變更)，讓移轉變得更簡單。<br/><br/> SQL 受控執行個體可援 Contoso 的技術需求和目標。<br/><br/> 受控執行個體將提供與其目前部署的 100% 相容性，同時將其從 SQL Server 2008 R2 中移走。<br/><br/>  他們可以使用 SQL Server 和 Windows Server 的 Azure Hybrid Benefit 來妥善運用軟體保證中的投資。<br/><br/> 他們可以重複使用資料庫移轉服務來進行其他未來的移轉作業。<br/><br/> SQL 受控執行個體具有 Contoso 不需設定的內建容錯功能。 這可確保資料層不再是單一的容錯移轉點。
**缺點** | WEBVM 會執行 Windows Server 2008 R2。  雖然 Azure 支援此作業系統，但它不再是支援的平台。 [深入了解](https://support.microsoft.com/en-us/help/956893)。<br/><br/> Web 層會保留只有 WEBVM 提供服務的單一容錯移轉點。<br/><br/> Contoso 必須繼續支持此應用程式 Web 層作為 VM，而不是轉向 Azure App Service 等受控服務。<br/><br/> 在資料層中，如果 Contoso 想要自訂作業系統或資料庫伺服器，或如果他們想要執行第三方應用程式以及 SQL Server，受控執行個體可能不是最佳解決方案。 在 IaaS VM 上執行 SQL Server 可提供此種彈性。 

### <a name="migration-process"></a>移轉程序

Contoso 會完成下列步驟，以將 SmartHotel360 應用程式的 Web 和資料層遷移至 Azure：

1. Contoso 的 Azure 基礎結構已經備妥，所以只需要為此案例新增幾個特定 Azure 元件即可。
2. 資料層會使用資料轉換服務進行遷移。 資料轉換服務會透過 Contoso 資料中心與 Azure 之間的站對站 VPN 連線來連線至內部部署 SQL Server VM。 接著，資料移轉服務會遷移資料庫。
3. Web 層將會透過 Site Recovery 使用隨即轉移進行遷移。 此程序需要準備內部部署 VMware 環境、設定和啟用複寫，以及將 VM 容錯移轉至 Azure 以便遷移。

     ![移轉架構](media/contoso-migration-rehost-vm-sql-managed-instance/migration-architecture.png) 

### <a name="azure-services"></a>Azure 服務

服務 | 說明 | 成本
--- | --- | ---
[Database Migration Service](https://docs.microsoft.com/azure/dms/dms-overview) | 資料庫移轉服務能夠從多個資料庫來源無縫移轉到 Azure 資料平台，將停機時間降到最低。 | 深入了解[支援的區域](https://docs.microsoft.com/azure/dms/dms-overview#regional-availability)和[資料庫移轉服務定價](https://azure.microsoft.com/pricing/details/database-migration/)。
[Azure SQL Database 受控執行個體](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance) | 受控執行個體是一種受控資料庫服務，可代表 Azure 雲端中的完全受控 SQL Server 執行個體。 它會使用與最新版 SQL Server 資料庫引擎相同的程式碼，並擁有最新的功能、效能增強功能和安全性修補程式。 | 使用在 Azure 中執行的 SQL Database 受控執行個體會根據所用容量產生費用。 深入了解[受控執行個體定價](https://azure.microsoft.com/pricing/details/sql-database/managed/)。 
[Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/) | Site Recovery 服務可協調和管理 Azure VM 以及內部部署 VM 和實體伺服器的移轉和災害復原。  | 複寫至 Azure 的期間會產生 Azure 儲存體費用。  發生容錯移轉時，系統會建立 Azure VM 並產生費用。 深入了解 [Site Recovery 費用和定價](https://azure.microsoft.com/pricing/details/site-recovery/)。

 

## <a name="prerequisites"></a>必要條件

Contoso 和其他使用者都必須符合此案例的下列先決條件：

需求 | 詳細資料
--- | ---
**註冊受控執行個體預覽版** | 您必須已在 SQL Database 受控執行個體有限公開預覽版中註冊。 您必須有 Azure 訂用帳戶才能[註冊](https://portal.azure.com#create/Microsoft.SQLManagedInstance)。 註冊可能需要幾天的時間才能完成，因此務必在開始部署此案例之前完成註冊。
**Azure 訂用帳戶** | 當您在這系列的第一篇文章中執行評量時，您應該已經建立訂用帳戶。 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。<br/><br/> 如果您建立免費帳戶，您就是訂用帳戶的管理員，並可執行所有動作。<br/><br/> 如果您使用現有的訂用帳戶，而且您不是訂用帳戶的系統管理員，則需要與系統管理員合作，讓其指派擁有者或參與者權限給您。<br/><br/> 如果您需要更細微的權限，請參閱[使用角色型存取控制來管理 Site Recovery 存取](../site-recovery/site-recovery-role-based-linked-access-control.md)。 
**Site Recovery (內部部署)** | 內部部署 vCenter Server 執行個體應執行 5.5、6.0 或 6.5 版<br/><br/> 執行 5.5、6.0 或 6.5 版的 ESXi 主機<br/><br/> 一或多部在 ESXi 主機上執行的 VMware VM。<br/><br/> VM 必須符合 [Azure 需求](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#azure-vm-requirements)。<br/><br/> 支援的[網路](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#network)和[儲存體](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#storage)組態。
**Database Migration Service** | 針對資料庫移轉服務，您需要[相容的內部部署 VPN 裝置](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-devices)。<br/><br/> 您必須能夠設定內部部署 VPN 裝置。 它必須有對外開放的公用 IPv4 位址。 此位址不能位於 NAT 裝置後方。<br/><br/> 請確定您可以存取內部部署 SQL Server 資料庫。<br/><br/> Windows 防火牆應該要能存取來源資料庫引擎。 了解如何[設定用於 Database Engine 存取的 Windows 防火牆](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。<br/><br/> 如果資料庫機器前面有防火牆，請新增規則以允許存取資料庫，以及允許透過 SMB 連接埠 445 存取檔案。<br/><br/> 用來連線至來源 SQL Server 執行個體和目標受控執行個體的認證，必須是 sysadmin 伺服器角色的成員。<br/><br/> 內部部署資料庫中必須有可供資料庫移轉服務用來備份來源資料庫的網路共用。<br/><br/> 請確定執行來源 SQL Server 執行個體的服務帳戶擁有網路共用的寫入權限。<br/><br/> 請記下擁有網路共用完整控制權限的 Windows 使用者和密碼。 資料庫移轉服務會模擬這些使用者認證，以便將備份檔案上傳至 Azure 儲存體容器。<br/><br/> SQL Server Express 安裝程序預設會將 TCP/IP 通訊協定設定為**停用**。 請務必將其啟用。

## <a name="scenario-steps"></a>案例步驟

以下說明 Contoso 打算如何設定部署：

> [!div class="checklist"]
> * **步驟 1：設定 SQL Database 受控執行個體**：Contoso 需要預先建立受控執行個體，以作為內部部署 SQL Server 資料庫的遷移目的地。
> * **步驟 2：準備資料庫移轉服務**：Contoso 必須註冊資料庫移轉提供者、建立執行個體，然後建立資料庫移轉服務專案。 Contoso 也必須為資料庫移轉服務設定共用存取簽章 (SAS) 統一資源識別項 (URI)。 SAS URI 可提供 Contoso 儲存體帳戶資源的委派存取權，以便 Contoso 對儲存體物件授與有限的權限。 Contoso 會設定 SAS URI，以便資料庫移轉服務存取儲存體帳戶容器 (此為服務在上傳 SQL Server 備份檔案時的上傳目的地)。
> * **步驟 3：針對 Site Recovery 準備 Azure**：Contoso 必須建立儲存體帳戶來保存 Site Recovery 的複寫資料。 它也必須建立 Azure 復原服務保存庫。
> * **步驟 4：針對 Site Recovery 準備內部部署 VMware**：Contoso 會準備帳戶以便探索 VM 和安裝代理程式，進而在容錯移轉後連線至 Azure VM。
> * **步驟 5：複寫 VM**：為了設定複寫，Contoso 會設定 Site Recovery 來源和目標環境、設定複寫原則，並開始將 VM 複寫至 Azure 儲存體。
> * **步驟 6：使用資料庫移轉服務遷移資料庫**：Contoso 會遷移資料庫。
> * **步驟 7：使用 Site Recovery 遷移 VM**：Contoso 會執行測試容錯移轉，確定一切都能正常運作。 接著，Contoso 會執行完整的容錯移轉，將 VM 遷移至 Azure。

## <a name="step-1-prepare-a-sql-database-managed-instance"></a>步驟 1：準備 SQL Database 受控執行個體

若要設定 Azure SQL Database 受控執行個體，Contoso 需要一個符合下列要求的子網路：

- 此子網路必須是專用的。 它必須空白，不得包含任何其他雲端服務。 子網路不能是閘道子網路。
- 建立受控執行個體後，Contoso 就不得在該子網路中新增資源。
- 子網路不能有與其相關聯的網路安全性群組。
- 子網路必須有使用者定義的路由 (UDR) 路由表。 唯一指派的路由應該是 0.0.0.0/0 下一個躍點網際網路。 
- 選擇性自訂 DNS：如果在 Azure 虛擬網路上指定自訂 DNS，則必須將 Azure 的遞迴解析程式 IP 位址 (例如 168.63.129.16) 新增至清單。 了解如何[為受控執行個體設定自訂 DNS](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-custom-dns)。
- 子網路不得有相關聯的服務端點 (儲存體或 SQL)。 虛擬網路上應該停用服務端點。
- 子網路必須至少具有 16 個 IP 位址。 了解如何[調整受控執行個體子網路的大小](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-vnet-configuration#determine-the-size-of-subnet-for-managed-instances)。
- 在 Contoso 的混合式環境中，需要有自訂 DNS 設定。 Contoso 會將 DNS 設定配置為使用公司的其中一或多部 Azure DNS 伺服器。 深入了解 [DNS 自訂](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-custom-dns)。

### <a name="set-up-a-virtual-network-for-the-managed-instance"></a>設定受控執行個體的虛擬網路

Contoso 管理員會設定虛擬網路，如下所示： 

1. 他們會在主要美國東部 2 區域中建立新的虛擬網路 (**VNET-SQLMI-EU2**)。 它會將此虛擬網路新增至 **ContosoNetworkingRG** 資源群組。
2. 他們會指派 10.235.0.0/24 位址空間。 他們會確保範圍不會與其企業中的任何其他網路重疊。
3. 他們會將兩個子網路新增到網路：
    - **SQLMI-DS-EUS2** (10.235.0.0.25)
    - **SQLMI-SAW-EUS2** (10.235.0.128/29). 此子網路用來將目錄連結到受控執行個體。

    ![受控執行個體 - 建立虛擬網路](media/contoso-migration-rehost-vm-sql-managed-instance/mi-vnet.png)

4. 部署虛擬網路和子網路之後，他們會將網路對等互連，如下所示：

    - 對等互連 **VNET-SQLMI-EUS2** 與 **VNET-HUB-EUS2** (美國東部 2 的中樞虛擬網路)。
    - 對等互連 **VNET-SQLMI-EUS2** 與 **VNET-PROD-EUS2** (生產網路)。

    ![網路對等互連](media/contoso-migration-rehost-vm-sql-managed-instance/mi-peering.png)

5. 他們可設定自訂 DNS 設定。 DNS 會先指向 Contoso 的 Azure 網域控制站。 而後指向 Azure DNS。 Contoso Azure 網域控制站的位置如下所示：

    - 位於美國東部 2 生產網路 (**VNET-PROD-EUS2**) 的 **PROD-DC-EUS2** 子網路中
    - **CONTOSODC3** 位址：10.245.42.4
    - **CONTOSODC4** 位址：10.245.42.5
    - Azure DNS 解析程式：168.63.129.16

     ![網路 DNS 伺服器](media/contoso-migration-rehost-vm-sql-managed-instance/mi-dns.png)

*需要其他協助？*

- 取得 [SQL Database 受控執行個體](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance)的概觀。
- 了解如何[建立 SQL Database 受控執行個體的虛擬網路](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-vnet-configuration#create-a-new-virtual-network-for-managed-instances)。
- 了解如何[設定對等互連](https://docs.microsoft.com/azure/virtual-network/virtual-network-manage-peering)。
- 了解如何[更新 Azure Active Directory DNS 設定](https://docs.microsoft.com/azure/active-directory-domain-services/active-directory-ds-getting-started-dns)。

### <a name="set-up-routing"></a>設定路由

受控執行個體會置於私人虛擬網路中。 Contoso 需要路由表，以供虛擬網路與 Azure 管理服務通訊。 如果虛擬網路無法與管理它的服務通訊，則會變成無法存取。

Contoso 會考量下列因素：

- 路由表包含一組規則 (路由)，可指定從受控執行個體傳送的封包應如何在虛擬網路中路由傳送。
- 路由表會與受控執行個體部署所在的子網路相關聯。 每個離開子網路的封包都會依據相關聯的路由表進行處理。
- 一個子網路只能與一個路由表相關聯。
- 在 Microsoft Azure 中建立路由表，沒有任何額外的費用。

 若要設定路由，Contoso 管理員可執行下列動作：

1. 他們會在 **ContosoNetworkingRG** 資源群組中建立 UDR (路由) 表。

    ![路由表](media/contoso-migration-rehost-vm-sql-managed-instance/mi-route-table.png)

2. 若要符合受控執行個體需求，在部署路由表 (**MIRouteTable**) 之後，他們會新增位址前置詞為 0.0.0.0/0 的路由。 [下一個躍點類型] 選項會設定為 [網際網路]。

    ![路由表前置詞](media/contoso-migration-rehost-vm-sql-managed-instance/mi-route-table-prefix.png)
    
3. 他們會讓路由表與 **SQLMI-DB-EUS2** 子網路 (在 **VNET-SQLMI-EUS2** 網路中) 建立關聯。 

    ![路由表子網路](media/contoso-migration-rehost-vm-sql-managed-instance/mi-route-table-subnet.png)
    
*需要其他協助？*

了解如何[設定受控執行個體的路由](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-create-tutorial-portal#create-new-route-table-and-a-route)。

### <a name="create-a-managed-instance"></a>建立受控執行個體

Contoso 管理員現在可以佈建 SQL Database 受控執行個體：

1. 因為受控執行個體會提供一個商務應用程式，所以他們會在公司的主要美國東部 2 區域中部署受控執行個體。 他們會將受控執行個體新增至 **ContosoRG** 資源群組。
2. 他們會選取執行個體的定價層、大小計算和儲存體。 深入了解[受控執行個體定價](https://azure.microsoft.com/pricing/details/sql-database/managed/)。

    ![受控執行個體](media/contoso-migration-rehost-vm-sql-managed-instance/mi-create.png)

3. 部署受控執行個體之後，**ContosoRG** 資源群組中會出現兩個新的資源：

    - 虛擬叢集 (假使 Contoso 有多個受控執行個體)。
    - SQL Server Database 受控執行個體。 

    ![受控執行個體](media/contoso-migration-rehost-vm-sql-managed-instance/mi-resources.png)

*需要其他協助？*

了解如何[佈建受控執行個體](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-create-tutorial-portal)。

## <a name="step-2-prepare-the-database-migration-service"></a>步驟 2︰準備資料庫移轉服務

若要準備資料庫移轉服務，Contoso 管理員需要執行一些作業：

- 在 Azure 中註冊資料庫移轉服務提供者。
- 將 Azure 儲存體存取權提供給資料庫移轉服務，以便上傳用來遷移資料庫的備份檔案。 為了提供 Azure 儲存體的存取權，他們會建立 Azure Blob 儲存體容器。 他們會產生 Blob 儲存體容器的 SAS URI。 
- 建立資料庫移轉服務專案。

然後，他們會完成下列步驟：

1. 他們會在其訂用帳戶之下註冊資料庫移轉提供者。
    ![資料庫移轉服務 - 註冊](media/contoso-migration-rehost-vm-sql-managed-instance/dms-subscription.png)

2. 他們會建立 Blob 儲存體容器。 Contoso 會產生 SAS URI，以便資料庫移轉服務存取它。

    ![資料庫移轉服務 - 產生 SAS URI](media/contoso-migration-rehost-vm-sql-managed-instance/dms-sas.png)

3. 他們會建立資料庫移轉服務執行個體。 

    ![資料庫移轉服務 - 建立執行個體](media/contoso-migration-rehost-vm-sql-managed-instance/dms-instance.png)

4. 他們會將資料庫移轉服務執行個體置於 **VNET-PROD-DC-EUS2** 虛擬網路的 **PROD-DC-EUS2** 子網路中。
    - 資料庫移轉服務會放在該處，因為此服務必須在可透過 VPN 閘道存取內部部署 SQL Server VM 的虛擬網路中。
    - **VNET-PROD-EUS2** 會與 **VNET-HUB-EUS2** 對等互連，並且能夠使用遠端閘道。 [使用遠端閘道] 選項確保資料庫移轉服務可以視需要進行通訊。

        ![資料庫移轉服務 - 設定網路](media/contoso-migration-rehost-vm-sql-managed-instance/dms-network.png)

*需要其他協助？*

- 了解如何[設定資料移轉服務](https://docs.microsoft.com/azure/dms/quickstart-create-data-migration-service-portal)。
- 了解如何[建立和使用 SAS](https://docs.microsoft.com/azure/storage/blobs/storage-dotnet-shared-access-signature-part-2)。


## <a name="step-3-prepare-azure-for-the-site-recovery-service"></a>步驟 3：針對 Site Recovery 服務準備 Azure

Contoso 需要數個 Azure 元素，才能夠設定 Site Recovery 以便遷移其 Web 層 VM (**WEBMV**)：

- 一個虛擬網路，容錯移轉的資源位於其中。
- 用來保存複寫資料的儲存體帳戶。 
- Azure 中的復原服務保存庫。

Contoso 管理員會按照下列方式設定 Site Recovery：

1. 因為 VM 是 SmartHotel360 應用程式的 Web 前端，所以 Contoso 會將 VM 容錯移轉到其現有的生產網路 (**VNET-PROD-EUS2**) 和子網路 (**PROD-FE-EUS2**)。 此網路和子網路都位於主要美國東部 2 區域中。 Contoso 會在[部署 Azure 基礎結構](contoso-migration-infrastructure.md)時設定此網路。
2. 他們會建立儲存體帳戶 (**contosovmsacc20180528**)。 Contoso 會使用一般用途的帳戶。 Contoso 會選取標準儲存體和本地備援儲存體複寫。

    ![Site Recovery - 建立儲存體帳戶](media/contoso-migration-rehost-vm-sql-managed-instance/asr-storage.png)

3. 網路和儲存體帳戶準備就緒之後，他們會建立保存庫 (**ContosoMigrationVault**)。 Contoso 會將此保存庫放在主要美國東部 2 區域的 **ContosoFailoverRG** 資源群組中。

    ![復原服務 - 建立保存庫](media/contoso-migration-rehost-vm-sql-managed-instance/asr-vault.png)

*需要其他協助？*

了解如何[為 Azure 進行 Site Recovery 的設定](https://docs.microsoft.com/azure/site-recovery/tutorial-prepare-azure)。


## <a name="step-4-prepare-on-premises-vmware-for-site-recovery"></a>步驟 4：為內部部署 VMware 進行 Site Recovery 的準備工作

若要針對 Site Recovery 準備 VMware，Contoso 管理員必須完成下列工作：

- 在 vCenter Server 執行個體或 vSphere ESXi 主機上，準備一個帳戶。 此帳戶會自動探索 VM。
- 準備一個帳戶，其允許在 Contoso 想要複寫的 VMware VM 上自動安裝行動服務。
- 準備內部部署 VM，以便在容錯移轉之後於建立時連線到 Azure VM。


### <a name="prepare-an-account-for-automatic-discovery"></a>準備帳戶以進行自動探索

Site Recovery 需要存取 VMware 伺服器才能：

- 自動探索 VM。 需要至少一個唯讀帳戶。
- 協調複寫、容錯移轉和容錯回復。 Contoso 需要可執行建立和移除磁碟以及開啟 VM 等作業的帳戶。

Contoso 管理員會藉由完成下列工作來設定帳戶：

1. 在 vCenter 層級建立一個角色。
2. 將必要權限指派給該角色。

*需要其他協助？*

了解如何[建立及指派自動探索的角色](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial-prepare-on-premises#prepare-an-account-for-automatic-discovery)。

### <a name="prepare-an-account-for-mobility-service-installation"></a>準備一個用來安裝行動服務的帳戶

行動服務必須安裝在 Contoso 想要複寫的 VM 上。 Contoso 會考慮有關行動服務的這些因素：

- 當 Contoso 啟用 VM 的複寫功能時，Site Recovery 可以自動推送安裝此元件。
- 若要自動推送安裝，Contoso 必須準備一個可供 Site Recovery 用來存取 VM 的帳戶。
- 在 Azure 主控台中設定複寫時會指定此帳戶。
- Contoso 必須有權限可以在 VM 上安裝的網域或本機帳戶。

*需要其他協助？*

了解如何[建立用於行動服務推送安裝的帳戶](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial-prepare-on-premises#prepare-an-account-for-mobility-service-installation)。

### <a name="prepare-to-connect-to-azure-vms-after-failover"></a>準備在容錯移轉後連接到 Azure VM

容錯移轉至 Azure 之後，Contoso 希望能夠連線到 Azure 中複寫的 VM。 若要連線到 Azure 中複寫的 VM，Contoso 管理員必須在移轉之前，在內部部署 VM 上完成一些工作： 

1. 為了透過網際網路存取，他們會在內部部署 VM 上啟用 RDP，再進行容錯移轉。 他們會確定已針對 [公用] 設定檔新增 TCP 和 UDP 規則，且在 [Windows 防火牆] > [允許的應用程式] 中已針對所有設定檔允許 RDP。
2. 如需透過站對站 VPN 存取，他們會在內部部署機器上啟用 RDP。 在 [Windows 防火牆] > [允許的應用程式與功能] 中，他們允許 [網域和私人] 網路使用 RDP。
3. 他們會將內部部署 VM 的作業系統 SAN 原則設定成 **OnlineAll**。

Contoso 管理員在執行容錯移轉時，也必須檢查下列項目：

- 觸發容錯移轉時，VM 上不應該有擱置的 Windows 更新。 如果 Windows 更新擱置，在更新完成前，Contoso 使用者無法登入虛擬機器。
- 容錯移轉之後，管理員應勾選 [開機診斷] 以檢視 VM 的螢幕擷取畫面。 如果他們無法檢視開機診斷，則應檢查 VM 是否正常執行，然後檢閱[疑難排解祕訣](http://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx)。

## <a name="step-5-replicate-the-on-premises-vms-to-azure"></a>步驟 5：將內部部署 VM 複寫到 Azure

在執行移轉至 Azure 之前，Contoso 管理員必須為內部部署 VM 設定和啟用複寫。

### <a name="set-a-replication-goal"></a>設定複寫目標

1. 在保存庫的保存庫名稱 (**ContosoVMVault**) 底下，他們會設定一個複寫目標 ([開始使用] > [Site Recovery] > [準備基礎結構])。
2. 他們指定機器位於內部部署環境、屬於 VMware VM，並複寫至 Azure。

    ![準備基礎結構 - 保護目標](./media/contoso-migration-rehost-vm-sql-managed-instance/replication-goal.png)

### <a name="confirm-deployment-planning"></a>確認部署規劃

若要繼續，Contoso 管理員會確認他們已完成部署規劃。 他們會選取 [是，我已完成]。 在此部署中，Contoso 只會遷移單一 VM，所以不需要部署規劃。

### <a name="set-up-the-source-environment"></a>設定來源環境

Contoso 管理員現在會設定來源環境。 若要設定其來源環境，他們需要下載 OVF 範本，然後用它來將組態伺服器與其相關聯的元件部署為高可用性、內部部署 VMware VM。 伺服器上的元件包括︰

- 組態伺服器，可協調內部部署基礎結構與 Azure 之間的通訊。 組態伺服器會管理資料複寫。
- 處理序伺服器，可作為複寫閘道。 處理序伺服器：
    - 接收複寫資料。
    - 使用快取、壓縮和加密，將複寫資料最佳化。
    - 將複寫日期傳送至 Azure 儲存體。
- 處理序伺服器也會在將要複寫的 VM 上安裝行動服務。 處理序伺服器會在內部部署 VMware VM 上執行自動探索。
- 建立並啟動組態伺服器 VM 之後，Contoso 會在保存庫中註冊伺服器。

若要設定來源環境，Contoso 管理員可執行下列動作：

1. 他們會從 Azure 入口網站 ([準備基礎結構] > [來源] > [組態伺服器]) 下載 OVF 範本。
    
    ![新增組態伺服器](./media/contoso-migration-rehost-vm-sql-managed-instance/add-cs.png)

2. 他們將範本匯入 VMware 中，以建立和部署 VM。

    ![部署 OVF 範本](./media/contoso-migration-rehost-vm-sql-managed-instance/vcenter-wizard.png)

3.  當他們第一次啟動 VM 時，VM 會開始進行 Windows Server 2016 安裝程序。 他們接受授權合約，並輸入系統管理員密碼。
4. 安裝完成後，他們會以系統管理員身分登入 VM。 第一次登入時，Azure Site Recovery 設定工具會自動執行。
5. 在 Site Recovery 設定工具中，他們會輸入一個名稱，以在保存庫中註冊組態伺服器。
6. 此工具會檢查 VM 與 Azure 的連線。 建立連線之後，他們會選取 [登入] 以登入 Azure 訂用帳戶。 認證必須能夠存取在其中註冊組態伺服器的保存庫。 

    ![註冊組態伺服器](./media/contoso-migration-rehost-vm-sql-managed-instance/config-server-register2.png)

7. 此工具會執行一些設定工作，而後重新開機。 他們會再次登入機器。 組態伺服器管理精靈會自動啟動。
8. 在此精靈中，他們選取要接收複寫流量的 NIC。 這項設定在完成之後即無法變更。
9. 他們會選取訂用帳戶、資源群組，以及要在其中註冊組態伺服器的復原服務保存庫。

    ![選取復原服務保存庫](./media/contoso-migration-rehost-vm-sql-managed-instance/cswiz1.png)

10. 他們下載並安裝 MySQL Server 和 VMWare PowerCLI。 然後，他們會驗證伺服器設定。
11. 驗證之後，他們會指定 vCenter Server 執行個體或 vSphere 主機的 FQDN 或 IP 位址。 他們會保留預設的連接埠，並且在 Azure 中輸入 vCenter Server 執行個體的顯示名稱。
12. 他們會指定先前所建立的帳戶，以便 Site Recovery 自動探索可用於複寫的 VMware VM。 
13. 他們會輸入認證，以便在啟用複寫時自動安裝行動服務。 對於 Windows 機器，此帳戶需具備 VM 的本機系統管理員權限。 

    ![設定 vCenter Server](./media/contoso-migration-rehost-vm-sql-managed-instance/cswiz2.png)

7. 註冊完成後，他們會在 Azure 入口網站中再次確認組態伺服器和 VMware 伺服器都已列在保存庫中的 [來源] 頁面上。 探索可能需要 15 分鐘以上的時間。 
8. Site Recovery 會使用指定的設定連線至 VMware 伺服器並探索 VM。

### <a name="set-up-the-target"></a>設定目標

Contoso 管理員現在會設定目標複寫環境：

1. 在 [準備基礎結構] > [目標] 中，他們選取目標設定。
2. Site Recovery 會確認在指定的目標中有儲存體帳戶和網路。

### <a name="create-a-replication-policy"></a>建立複寫原則

設定好來源和目標之後，Contoso 管理員會建立複寫原則，並將讓此原則與組態伺服器建立關聯：

1. 在 [準備基礎結構] > [複寫設定] > [複寫原則] >  [建立和關聯] 中，他們會建立 **ContosoMigrationPolicy** 原則。
2. 他們使用預設設定：
    - **RPO 閾值**：預設值是 60 分鐘。 這個值可定義復原點的建立頻率。 連續複寫超過此限制時會產生警示。
    - **復原點保留**：預設值為 24 小時。 此值會指定每個復原點的保留週期有多長。 複寫的 VM 可以還原至一個週期內的任何時間點。
    - **應用程式一致的快照頻率**：預設值為 1 小時。 此值會指定應用程式一致快照的建立頻率。
 
    ![複寫原則 - 建立](./media/contoso-migration-rehost-vm-sql-managed-instance/replication-policy.png)

3. 此原則會自動與設定伺服器產生關聯。 

    ![複寫原則 - 關聯](./media/contoso-migration-rehost-vm-sql-managed-instance/replication-policy2.png)

*需要其他協助？*

- 您可以在[設定內部部署 VMware VM 的災害復原](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial)中，閱讀這些步驟的詳細逐步解說。
- 您會得到詳細的操作說明，協助您[設定來源環境](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-source)、[部署組態伺服器](https://docs.microsoft.com/azure/site-recovery/vmware-azure-deploy-configuration-server)以及[指定複寫設定](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-replication)。

### <a name="enable-replication"></a>啟用複寫

現在，Contoso 管理員可以開始複寫 WebVM。

1. 在 [複寫應用程式] > [來源] > [複寫] 中，他們選取來源設定。
2. 他們表示想要啟用虛擬機器、選取 vCenter Server 執行個體，然後設定組態伺服器。

    ![啟用複寫 - 來源](./media/contoso-migration-rehost-vm-sql-managed-instance/enable-replication1.png)
 
3. 他們會指定目標設定，包括 Azure VM 在容錯移轉之後所在的資源群組和網路。 他們會指定要儲存所複寫資料的儲存體帳戶。

    ![啟用複寫 - 目標](./media/contoso-migration-rehost-vm-sql-managed-instance/enable-replication2.png)

4. 他們會選取 **WebVM** 來進行複寫。 Site Recovery 會在複寫啟用時在每部 VM 上安裝行動服務。 

    ![啟用複寫 - 選取 VM](./media/contoso-migration-rehost-vm-sql-managed-instance/enable-replication3.png)

5. 他們會確認已選取正確的複寫原則，並針對 **WEBVM** 啟用複寫。 他們在 [作業] 中追踨複寫進度。 執行 [完成保護] 作業之後，機器即準備好進行容錯移轉。
6. 在 Azure 入口網站的 [基本資訊] 中，他們可以看到複寫到 Azure 的 VM 結構：

    ![基礎結構檢視](./media/contoso-migration-rehost-vm-sql-managed-instance/essentials.png)

*需要其他協助？*

您可以在[啟用複寫](https://docs.microsoft.com/azure/site-recovery/vmware-azure-enable-replication)中閱讀這些步驟的完整逐步解說。

## <a name="step-6-migrate-the-database"></a>步驟 6：遷移資料庫 

Contoso 管理員必須建立資料庫移轉服務專案，然後遷移資料庫。

### <a name="create-a-database-migration-service-project"></a>建立資料庫移轉服務專案

1. 他們會建立資料庫移轉服務專案。 他們會選取 [SQL Server] 來源伺服器類型，並以 [Azure SQL Database 受控執行個體] 作為目標。

     ![資料庫移轉服務 - 新增移轉專案](./media/contoso-migration-rehost-vm-sql-managed-instance/dms-project.png)

2. 移轉精靈隨即開啟。

### <a name="migrate-the-database"></a>遷移資料庫 

1. 在移轉精靈中，他們會指定內部部署資料庫所在的來源 VM。 他們會提供用於存取資料庫的認證。

    ![資料庫移轉服務 - 來源詳細資料](./media/contoso-migration-rehost-vm-sql-managed-instance/dms-wizard-source.png)

2. 他們會選取要遷移的資料庫 (**SmartHotel.Registration**)：

    ![資料庫移轉服務 - 選取來源資料庫](./media/contoso-migration-rehost-vm-sql-managed-instance/dms-wizard-sourcedb.png)

3. 對此目標，他們會輸入 Azure 中的受控執行個體名稱，以及存取認證。

    ![資料庫移轉服務 - 目標詳細資料](./media/contoso-migration-rehost-vm-sql-managed-instance/dms-target-details.png)

4. 在 [新增活動] > [執行移轉] 中，他們會指定下列設定來執行移轉：
    - 來源和目標認證。
    - 要遷移的資料庫。
    - 在內部部署 VM 上建立的網路共用。 資料庫移轉服務會將來源備份擷取到此共用。 
        - 執行來源 SQL Server 執行個體的服務帳戶必須擁有此共用的寫入權限。
        - 必須使用共用的 FQDN 路徑。
    - SAS URI，此 URI 可供資料庫移轉服務存取儲存體帳戶容器，而此服務會將備份檔案上傳至該容器以供移轉。

        ![資料庫移轉服務 - 設定移轉設定](./media/contoso-migration-rehost-vm-sql-managed-instance/dms-migration-settings.png)

5. 他們會儲存移轉設定，然後執行移轉。
6. 在 [概觀] 中，他們會監視移轉狀態。

    ![資料庫移轉服務 - 監視](./media/contoso-migration-rehost-vm-sql-managed-instance/dms-monitor1.png)

7. 完成移轉後，他們會確認受控執行個體上有目標資料庫。

    ![資料庫移轉服務 - 驗證資料庫移轉](./media/contoso-migration-rehost-vm-sql-managed-instance/dms-monitor2.png)

## <a name="step-7-migrate-the-vm"></a>步驟 7：遷移 VM

Contoso 管理員會執行快速測試容錯移轉，然後移轉 VM。

### <a name="run-a-test-failover"></a>執行測試容錯移轉

在遷移 WEBVM 之前，測試容錯移轉有助於確保一切運作正常。 管理員會完成下列步驟：

1. 他們會針對最新的可用時間點 (**最近處理的**) 執行一次測試容錯移轉。
2. 他們會選取 [先將機器關機再開始容錯移轉]。 選取這個選項後，Site Recovery 嘗試先關閉來源 VM，再觸發容錯移轉。 即使關機失敗，仍會繼續容錯移轉。 
3. 執行測試容錯移轉： 
    1. 系統會執行先決條件檢查，以確保移轉所需的全部條件都已準備就緒。
    2. 容錯移轉會處理資料，以便建立 Azure VM。 如果選取了最新的復原點，則會根據資料建立復原點。
    3.  使用先前步驟中處理的資料來建立 Azure VM。
3. 容錯移轉完成後，Azure VM 複本會出現在 Azure 入口網站中。 他們會確認一切運作正常：VM 為適當的大小、其已連線到正確的網路，而且正在執行中。 
4. 確認容錯移轉測試之後，他們可以清除容錯移轉，並記錄任何觀察到的結果。 

### <a name="migrate-the-vm"></a>移轉 VM

1. 驗證測試容錯移轉如預期般正常運作之後，Contoso 管理員會針對移轉建立復原方案，並將 WEBVM 新增到此方案：

     ![建立復原計畫 - 選取項目](./media/contoso-migration-rehost-vm-sql-managed-instance/recovery-plan.png)

2. 他們會對此方案執行容錯移轉，並選取最新的復原點。 他們指定 Site Recovery 應該在觸發容錯移轉之前，先嘗試關閉內部部署 VM。

    ![容錯移轉](./media/contoso-migration-rehost-vm-sql-managed-instance/failover1.png)

3. 容錯移轉之後，他們可以確認 Azure VM 會如預期般出現在 Azure 入口網站中。

   ![復原方案詳細資料](./media/contoso-migration-rehost-vm-sql-managed-instance/failover2.png)

4. 確認之後，他們會完成移轉程序、停止 VM 複寫，以及停止 VM 的 Site Recovery 計費。

    ![容錯移轉 - 完成移轉](./media/contoso-migration-rehost-vm-sql-managed-instance/failover3.png)

### <a name="update-the-connection-string"></a>更新連接字串

在移轉程序的最後一個步驟中，Contoso 管理員會將應用程式的連接字串更新為指向在 Contoso 受控執行個體上執行的遷移後資料庫。

1. 在 Azure 入口網站中，他們會選取 [設定] > [連接字串] 來尋找連接字串。

    ![連接字串](./media/contoso-migration-rehost-vm-sql-managed-instance/failover4.png)  

2. 他們會使用 SQL Database 受控執行個體的使用者名稱與密碼來更新此字串。
3. 設定此字串之後，他們會取代其應用程式的 web.config 檔案中目前的連接字串。
4. 更新該檔案並加以儲存後，他們會在命令提示字元視窗中執行 `IISRESET /RESTART`，以在 WEBVM 上重新啟動 IIS。
5. 重新啟動 IIS 後，應用程式會使用在 SQL Database 受控執行個體上執行的資料庫。
6. 此時，他們可以關閉內部部署 SQLVM 機器。 已完成移轉。

*需要其他協助？*

- 了解如何[執行測試容錯移轉](https://docs.microsoft.com/azure/site-recovery/tutorial-dr-drill-azure)。 
- 了解如何[建立復原方案](https://docs.microsoft.com/azure/site-recovery/site-recovery-create-recovery-plans)。
- 了解如何[容錯移轉至 Azure](https://docs.microsoft.com/azure/site-recovery/site-recovery-failover)。

## <a name="clean-up-after-migration"></a>移轉之後進行清除

移轉完成時，SmartHotel360 應用程式會在 Azure VM 上執行，而 SmartHotel360 資料庫可適用於 Azure SQL Database 受控執行個體。  

現在，Contoso 必須執行下列清除工作：  

- 從 vCenter Server 清查中移除 WEBVM 機器。
- 從 vCenter Server 清查中移除 SQLVM 機器。
- 從本機備份作業移除 WEBVM 和 SQLVM。
- 更新內部文件以顯示 WEBVM 的新位置和 IP 位址。
- 從內部文件中移除 SQLVM。 或者，Contoso 可以修改文件，將 SQLVM 顯示為已刪除且已不在 VM 清查中。
- 檢閱與已解除委任 VM 互動的任何資源。 更新任何相關的設定或文件，以反映新的組態。

## <a name="review-the-deployment"></a>檢閱部署

對於 Azure 中的遷移後資源，Contoso 必須能執行一切功能並保護其新的基礎結構。

### <a name="security"></a>安全性

Contoso 安全性小組會檢閱 Azure VM 和 SQL Database 受控執行個體，檢查其實作是否有任何的安全疑慮：

- 小組會檢閱用來控制 VM 存取權的網路安全性群組。 網路安全性群組有助於確保只可以傳遞該應用程式允許的流量。
- Contoso 安全性小組也會考慮使用 Azure 磁碟加密和 Azure KeyVault 來保護磁碟上的資料。
- 小組會在受控執行個體上啟用威脅偵測。 如果偵測到威脅，威脅偵測會將警示傳送至 Contoso 的安全性小組/服務台系統。 深入了解[受控執行個體的威脅偵測](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-threat-detection)。

     ![受控執行個體安全性 - 威脅偵測](./media/contoso-migration-rehost-vm-sql-managed-instance/mi-security.png)  

若要深入了解 VM 的安全性做法，請參閱 [Azure 中 IaaS 工作負載的安全性最佳做法](https://docs.microsoft.com/azure/security/azure-security-best-practices-vms#vm-authentication-and-access-control)。

### <a name="bcdr"></a>BCDR

針對商務持續性和災害復原 (BCDR)，Contoso 會採取下列動作：

- 保護資料安全：Contoso 會使用 Azure 備份服務來備份 VM 上的資料。 [深入了解]https://docs.microsoft.com/azure/backup/backup-introduction-to-azure-backup?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
- 保持應用程式啟動及執行：Contoso 會使用 Site Recovery，在 Azure 中將應用程式 VM 複寫至次要區域。 [深入了解](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-quickstart)。
- Contoso 深入了解如何管理 SQL 受控執行個體，包括[資料庫備份](https://docs.microsoft.com/azure/sql-database/sql-database-automated-backups)。


### <a name="licensing-and-cost-optimization"></a>授權和成本最佳化

- Contoso 有 WEBVM 的現有授權。 為了利用 Azure Hybrid Benefit 的定價，Contoso 會轉換現有的 Azure VM。
- Contoso 會啟用 Microsoft 子公司 Cloudyn 授權的 Azure 成本管理。 成本管理是多雲端成本管理解決方案，可協助 Contoso 使用及管理 Azure 和其他雲端資源。 深入了解 [Azure 成本管理](https://docs.microsoft.com/azure/cost-management/overview)。 


## <a name="conclusion"></a>結論

在本文中，Contoso 使用 Site Recovery 服務將應用程式前端 VM 移轉至 Azure，以重新裝載 SmartHotel360 應用程式。 Contoso 會使用 Azure 資料庫移轉服務，將內部部署資料庫遷移至 Azure SQL Database 受控執行個體。

## <a name="next-steps"></a>後續步驟

在這系列的下一篇文章中，Contoso 會使用 Azure Site Recovery 服務，[在 Azure VM 上重新裝載 SmartHotel360 應用程式](contoso-migration-rehost-vm.md)。

