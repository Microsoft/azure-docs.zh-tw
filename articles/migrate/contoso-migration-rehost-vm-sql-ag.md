---
title: 將 Contoso 應用程式移轉至 Azure VM 和 SQL Server AlwaysOn 可用性群組，以重新裝載該應用程式 | Microsoft Docs
description: 了解 Contoso 如何將內部部署應用程式移轉至 Azure VM 和 SQL Server AlwaysOn 可用性群組，以重新裝載該應用程式
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: raynew
ms.openlocfilehash: f744b9bacfb43b5cf4ba81e19d8e543561bcec61
ms.sourcegitcommit: 3d0295a939c07bf9f0b38ebd37ac8461af8d461f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "43842748"
---
# <a name="contoso-migration-rehost-an-on-premises-app-on-azure-vms-and-sql-server-alwayson-availability-group"></a>Contoso 移轉：在 Azure VM 和 SQL Server AlwaysOn 可用性群組上重新裝載內部部署應用程式

本文說明 Contoso 如何在 Azure 中重新裝載 SmartHotel360 應用程式。 Contoso 會將應用程式前端 VM 移轉至 Azure VM，並將應用程式資料庫移轉至 Azure SQL Server VM，在具有 SQL Server AlwaysOn 可用性群組的 Windows Server 容錯移轉叢集中執行。

這份文件是系列文章中的其中一篇，說明 Contoso 這家虛構的公司如何將內部部署資源移轉至 Microsoft Azure 雲端。 整個系列文章中包含背景資訊和說明如何設定移轉基礎結構的案例、評估內部部署資源的移轉，以及執行不同類型的移轉。 案例的複雜性會提高。 我們會不定期新增其他文章。

**文章** | **詳細資料** | **狀態**
--- | --- | ---
[文章 1：概觀](contoso-migration-overview.md) | 系列文章的概觀、Contoso 的移轉策略，以及此系列中所使用的應用程式範例。 | 可用
[文章 2：部署 Azure 基礎結構](contoso-migration-infrastructure.md) | Contoso 會準備其內部部署基礎結構和其 Azure 基礎結構以進行移轉。 此系列中的所有移轉文章都使用相同的基礎結構。 | 可用
[文章 3：存取內部部署資源以移轉至 Azure](contoso-migration-assessment.md)  | Contoso 會評估其在 VMware 上執行的內部部署 SmartHotel360 應用程式。 Contoso 會使用 Azure Migrate 服務來評估應用程式 VM，並使用 Database Migration Assistant 來評估應用程式 SQL Server 資料庫。 | 可用
[文章 4：在 Azure VM 和 SQL Database 受控執行個體上重新裝載應用程式](contoso-migration-rehost-vm-sql-managed-instance.md) | Contoso 會為其內部部署 SmartHotel360 應用程式執行隨即移轉至 Azure 的作業。 Contoso 會使用 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview) 來移轉應用程式的前端 VM。 Contoso 會使用 [Azure 資料庫移轉服務](https://docs.microsoft.com/azure/dms/dms-overview)將應用程式資料庫移轉至 Azure SQL Database 受控執行個體。 | 可用   
[文章 5：在 Azure VM 上重新裝載應用程式](contoso-migration-rehost-vm.md) | Contoso 會使用 Site Recovery 服務，將其 SmartHotel360 應用程式 VM 移轉至 Azure VM。 | 可用
文章 6：在 Azure VM 上和 SQL Server AlwaysOn 可用性群組中重新裝載應用程式 | Contoso 會移轉 SmartHotel360 應用程式。 Contoso 會使用 Site Recovery 來遷移應用程式的 VM。 它會使用資料庫移轉服務，將應用程式資料庫遷移至受到 SQL Server 可用性群組保護的 SQL Server 叢集。 | 本文
[文章 7：將 Linux 應用程式重新裝載至 Azure VM](contoso-migration-rehost-linux-vm.md) | Contoso 會使用 Site Recovery 服務，來完成將其 Linux osTicket 應用程式隨即移轉至 Azure VM 的作業 | 可用
[文章 8：在 Azure VM 和 Azure MySQL Server 上重新裝載 Linux 應用程式](contoso-migration-rehost-linux-vm-mysql.md) | Contoso 會使用 Azure Site Recovery 將 Linux osTicket 應用程式移轉至 Azure VM，並使用 MySQL Workbench 將應用程式資料庫移轉至 Azure MySQL Server 執行個體。 | 可用
[文章 9：在 Azure Web 應用程式和 Azure SQL 資料庫上重構應用程式](contoso-migration-refactor-web-app-sql.md) | Contoso 會將 SmartHotel360 應用程式移轉至 Azure Web 應用程式，以及使用 Database Migration Assistant 將應用程式資料庫移轉至 Azure SQL Server 執行個體 | 可用
[文章 10：在 Azure Web Apps 和 Azure MySQL 上重構 Linux 應用程式](contoso-migration-refactor-linux-app-service-mysql.md) | Contoso 會使用 Azure 流量管理員，將 Linux osTicket 應用程式移轉至多個 Azure 區域的 Azure Web 應用程式，與 GitHub 整合以進行持續傳遞。 Contoso 會將應用程式資料庫遷移至適用於 MySQL 的 Azure 資料庫執行個體。 | 可用 
[文章 11：在 VSTS 上重構 TFS](contoso-migration-tfs-vsts.md) | Contoso 會將其內部部署 Team Foundation Server 部署，移轉至 Azure 中的 Visual Studio Team Services。 | 可用
[文章 12：在 Azure 容器和 Azure SQL Database 上重新建構應用程式](contoso-migration-rearchitect-container-sql.md) | Contoso 會將其 SmartHotel360 應用程式移轉至 Azure。 然後，它會重新建構應用程式 Web 層，作為在 Azure Service Fabric 中執行的 Windows 容器，以及具有 Azure SQL Database 的資料庫。 | 可用
[文章 13：在 Azure 中重建應用程式](contoso-migration-rebuild.md) | Contoso 會藉由使用各種 Azure 功能和服務 (包括 Azure App Service、Azure Kubernetes Service (AKS)、Azure Functions、Azure 認知服務及 Azure Cosmos DB) 重建其 SmartHotel360 應用程式。 | 可用


在本文中，Contoso 會將在 VMware VM 上執行的兩層 Windows .NET SmartHotel360 應用程式移轉至 Azure。 如果您想使用此應用程式，您可以從 [GitHub](https://github.com/Microsoft/SmartHotel360) 下載其開放原始碼。

## <a name="business-drivers"></a>商業動機

IT 領導小組與商務合作夥伴密切合作，以了解此次移轉所要實現的目標：

- **因應業務成長**：Contoso 的業務量日益增多，對內部部署系統和基礎結構造成了壓力。
- **提高效率**：Contoso 必須移除不必要的程序，並且簡化開發人員和使用者的程序。  企業亟需快速且不浪費時間或金錢的 IT 服務，進而更快滿足客戶的需求。
- **提高靈活性**：Contoso IT 必須能夠更快因應企業的需求。 其因應速度必須能夠比市場變化更快，才能更在全球經濟中獲致成功。  它不得礙事，或成為企業的絆腳石。
- **調整**：隨著企業順利成長，Contoso IT 必須提供能夠同步成長的系統。

## <a name="migration-goals"></a>移轉目標

Contoso 雲端小組已針對此次移轉擬定好各項目標。 這些目標用來判斷最合適的移轉方法：

- 移轉之後，應用程式不管是在 Azure 還是在目前的 VMware 中，應具有相同效能。  應用程式不管是在雲端中或在內部部署，都一樣重要。
- Contoso 不想在這個應用程式上投注資源。  這對企業很重要，但以其目前的形式而言，Contoso 只想安全地移至雲端。
- 應用程式的內部部署資料庫有可用性方面問題。 Contoso 想要將其部署於 Azure 中，作為具有容錯移轉能力的高可用性叢集。
- Contoso 想要從目前的 SQL Server 2008 R2 平台升級至 SQL Server 2017。
- Contoso 不想對此應用程式使用 Azure SQL Database，且正在尋求替代方案。


## <a name="solution-design"></a>解決方案設計

擬定好目標和需求後，Contoso 會設計和檢閱部署解決方案，並識別移轉程序，包括將用於移轉的 Azure 服務。

### <a name="current-architecture"></a>目前架構

- 應用程式分層至兩個 VM (WEBVM 和 SQLVM)。
- 這些 VM 位於 VMware ESXi 主機 **contosohost1.contoso.com** (6.5 版)
- VMware 環境是由 VM 上執行的 vCenter Server 6.5 (**vcenter.contoso.com**) 進行管理。
- Contoso 有內部部署資料中心 (contoso-datacenter) 以及內部部署網域控制站 (**contosodc1**)。


### <a name="proposed-architecture"></a>建議的架構

在此情節中：

- Contoso 會將應用程式前端 WEBVM 移轉至 Azure IaaS VM。
    - Azure 中的前端 VM 將部署在 ContosoRG 資源群組中 (用於生產資源)。
    -  它將位於美國東部 2 主要區域的 Azure 生產網路中 (VNET-PROD-EUS2)。
- 應用程式資料庫將移轉至 Azure SQL Server VM。
    - 它將位於 Contoso 在美國東部 2 主要區域的 Azure 資料庫網路中 (PROD-DB-EUS2)。
    - 它將放置在具有兩個節點、且使用 SQL Server Always On 可用性群組的 Windows Server 容錯移轉叢集中。
    - 在 Azure 中，叢集中的兩個 SQL Server VM 節點將部署在 ContosoRG 資源群組中。
    - 這些 VM 節點將位於美國東部 2 主要區域的 Azure 生產網路中 (VNET-PROD-EUS2)。
    - VM 會執行具有 SQL Server 2017 Enterprise Edition 的 Windows Server 2016。 Contoso 不具備此作業系統的授權，因此將使用 Azure Marketplace 中以 Azure EA 承諾用量計費的形式提供授權的映像。
    - 除了唯一名稱以外，這兩個 VM 所使用的設定皆相同。
- Contoso 將部署會接聽叢集上的流量，並將其導向至適當叢集節點的內部負載平衡器。
    - 內部負載平衡器將部署在 ContosoNetworkingRG 中 (用於網路資源)。
- 移轉完成之後，將會解除委任 Contoso 資料中心的內部部署 VM。

![案例架構](media/contoso-migration-rehost-vm-sql-ag/architecture.png) 

### <a name="database-considerations"></a>資料庫考量

在解決方案設計過程中，Contoso 會進行 Azure SQL Database 與 SQL Server 的功能比較。 下列考量協助他們決定採用執行 SQL Server 的 Azure IaaS VM：

 - 如果 Contoso 需要自訂作業系統或資料庫伺服器，或者想要在相同的 VM 上共置並執行第三方應用程式，使用執行 SQL Server 的 Azure VM 似乎是最佳解決方案。
 - 透過 Data Migration Assistant，Contoso 可輕易評估及移轉至 Azure SQL Database。
 

### <a name="solution-review"></a>解決方案檢閱

Contoso 會透過比較一份優缺點清單，來評估其建議設計。

**考量** | **詳細資料**
--- | ---
**優點** | WEBVM 會移至 Azure (不需變更)，讓移轉變得更簡單。<br/><br/> SQL Server 層將會在 SQL Server 2017 和 Windows Server 2016 上執行。 這會淘汰其目前的 Windows Server 2008 R2 作業系統，且執行 SQL Server 2017 將可支援 Contoso 的技術需求和目標。 IT 從 SQL Server 2008 R2 進行移轉時，可提供 100% 的相容性。<br/><br/> Contoso 可以使用 Azure Hybrid Benefit，充分發揮軟體保證的投資效益。<br/><br/> Azure 中高可用性 SQL Server 部署可提供容錯功能，使應用程式資料層不再是單一的容錯移轉點。
**缺點** | WEBVM 會執行 Windows Server 2008 R2。 Azure 對此作業系統的支援僅限於特定角色 (2018 年 7 月)。 [深入了解](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines)。<br/><br/> 應用程式的 Web 層會保留單一容錯移轉點。</br><br/> Contoso 必須繼續支持此 Web 層作為 Azure VM，而非移轉至 Azure App Service 等受控服務。<br/><br/> 透過選擇的解決方案，Contoso 必須繼續管理兩個 SQL Server VM，而非移轉至受控平台，例如 Azure SQL 受控執行個體。 此外，Contoso 可透過軟體保證，以折扣優惠在 Azure SQL 受控執行個體上交換其現有的授權。


### <a name="azure-services"></a>Azure 服務

**服務** | **說明** | **成本**
--- | --- | ---
[Database Migration Assistant](https://docs.microsoft.com/sql/dma/dma-overview?view=ssdt-18vs2017) | DMA 會從內部部署的 SQL Server 電腦本機執行，並透過站對站 VPN 將資料庫移轉至 Azure。 | DMA 是可下載的免費工具。
[Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/) | Site Recovery 可協調和管理 Azure VM 以及內部部署 VM 和實體伺服器的移轉和災害復原。  | 複寫至 Azure 的期間會產生 Azure 儲存體費用。  發生容錯移轉時，系統會建立 Azure VM 並產生費用。 [深入了解](https://azure.microsoft.com/pricing/details/site-recovery/)費用和定價。

 

## <a name="migration-process"></a>移轉程序

Contoso 管理員會將應用程式 VM 移轉至 Azure。

- 他們會使用 Site Recovery 將前端 VM 移轉至 Azure VM：
    - 在第一個步驟中，他們會準備以及設定 Azure 元件，然後準備內部部署 VMware 基礎結構。
    - 一切就緒後，他們就可以開始複寫 VM。
    - 複寫功能啟用且正常運作後，他們會將 VM 容錯移轉至 Azure，而加以移轉。
- 他們會使用 Data Migration Assistant (DMA)，將資料庫移轉至 Azure 中的 SQL Server 叢集。
    - 在第一個步驟中，他們必須在 Azure 中佈建 SQL Server VM、設定叢集與內部負載平衡器，並設定 AlwaysOn 可用性群組。
    - 這些作業完成後，他們即可移轉資料庫
- 移轉完成後，他們會為資料庫啟用 AlwaysOn 保護。

![移轉程序](media/contoso-migration-rehost-vm-sql-ag/migration-process.png) 
 
## <a name="prerequisites"></a>必要條件

以下是 Contoso 在此案例中應該準備好的事項。

**需求** | **詳細資料**
--- | ---
**Azure 訂用帳戶** | Contoso 已在本系列稍早的文章中建立訂用帳戶。 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。<br/><br/> 如果您建立免費帳戶，您就是訂用帳戶的管理員，並可執行所有動作。<br/><br/> 如果您使用現有訂用帳戶，而且您不是系統管理員，則需要與系統管理員合作，讓其指派擁有者或參與者權限給您。<br/><br/> 如果您需要更細微的權限，請檢閱[此文章](../site-recovery/site-recovery-role-based-linked-access-control.md)。 
**Azure 基礎結構** | [了解](contoso-migration-infrastructure.md) Contoso 如何設定 Azure 基礎結構。<br/><br/> 深入了解 Site Recovery 具體的[網路](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#network)和[儲存體](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#storage)需求。
**站台復原 (內部部署)** | 內部部署 vCenter 伺服器應執行 5.5、6.0 或 6.5 版<br/><br/> 執行 5.5、6.0 或 6.5 版的 ESXi 主機<br/><br/> 一或多部在 ESXi 主機上執行的 VMware VM。<br/><br/> VM 必須符合 [Azure 需求](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#azure-vm-requirements)。<br/><br/> 支援的[網路](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#network)和[儲存體](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#storage)組態。<br/><br/> 您要複寫的 VM 必須符合 [Azure 需求](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#azure-vm-requirements)。



## <a name="scenario-steps"></a>案例步驟

以下是 Contoso 執行移轉的方式：

> [!div class="checklist"]
> * **步驟 1：準備叢集**：建立叢集，以在 Azure 中部署兩個 SQL Server VM 節點。
> * **步驟 2：部署並設定叢集**：準備 Azure SQL Server 叢集。  資料庫會移轉至這個預先建立的叢集中。
> * **步驟 3：部署負載平衡器**：部署負載平衡器，以平衡傳輸至 SQL Server 節點的流量。
> * **步驟 4：為 Azure 進行 Site Recovery 的準備工作**：建立一個 Azure 儲存體帳戶來保存複寫的資料，並建立復原服務保存庫。 
> * **步驟 5：為內部部署 VMware 進行 Site Recovery 的準備工作**：為帳戶進行 VM 探索和代理程式安裝的準備工作。 準備內部部署 VM，讓使用者在移轉之後可連線至 Azure VM。
> * **步驟 6：複寫 VM**：啟用對 Azure 複寫 VM 的功能。
> * **步驟 7：安裝 DMA**：下載並安裝 Database Migration Assistant。
> * **步驟 7：使用 DMA 移轉資料庫**：將資料庫移轉至 Azure。
> * **步驟 9：保護資料庫**：為叢集建立 Always On 可用性群組。
> * **步驟 10：移轉 Web 應用程式 VM**：執行測試容錯移轉，確定一切都正常運作。 然後，執行對 Azure 的完整容錯移轉。 


## <a name="step-1-prepare-a-sql-server-alwayson-availability-group-cluster"></a>步驟 1：準備 SQL Server AlwaysOn 可用性群組叢集

Contoso 管理員會依照下列方式設定叢集：

1. 他們在 Azure Marketplace 中選取 SQL Server 2017 Enterprise Windows Server 2016 映像，以建立兩個 SQL Server VM。 

    ![SQL VM SKU](media/contoso-migration-rehost-vm-sql-ag/sql-vm-sku.png)

2. 在 [建立虛擬機器精靈] > [基本] 中，他們設定：

    - VM 的名稱：**SQLAOG1** 和 **SQLAOG2**。
    - 由於機器是攸關業務的重要元件，因此其 VM 磁碟類型可以是 SSD。
    - 他們指定機器認證。
    - 他們在主要美國東部 2 區域，將 VM 部署在 ContosoRG 資源群組中。

3. 在 [大小] 中，他們先為兩個 VM 都設定 D2s_V3 SKU。 其後他們將視需要加以調整。
4. 在 [設定] 中，他們執行下列作業：

    - 由於這些 VM 對應用程式而言都是重要的資料庫，因此他們使用受控磁碟。
    - 他們將機器放在美國東部 2 主要區域的生產網路中 (**VNET-PROD-EUS2**)，置於資料庫子網路內 (**PROD-DB-EUS2**)。
    - 他們建立新的可用性設定組：**SQLAOGAVSET**，具有兩個容錯網域和五個更新網域。

    ![SQL VM](media/contoso-migration-rehost-vm-sql-ag/sql-vm-settings.png)

4. 在 [SQL Server 設定] 中，他們將 SQL 對虛擬網路 (私人) 的連線限定於預設連接埠 1433。 針對驗證，他們使用站上所使用的相同認證 (**contosoadmin**)。

    ![SQL VM](media/contoso-migration-rehost-vm-sql-ag/sql-vm-db.png)

**需要其他協助？**

- [取得說明](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-server-provision#1-configure-basic-settings)以了解如何佈建 SQL Server VM。
- [了解](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-prereq#create-sql-server-vms)如何為不同的 SQL Server SKU 設定 VM。

## <a name="step-2-deploy-and-set-up-the-cluster"></a>步驟 2︰部署並設定叢集

以下是 Contoso 管理員設定叢集的方式：

1. 他們設定 Azure 儲存體帳戶作為雲端見證。
2. 他們將 SQL Server VM 新增至 Contoso 內部部署資料中心的 Active Directory 網域。
3. 他們在 Azure 中建立叢集。
4. 他們設定雲端見證。
5. 最後，他們啟用 SQL Always On 可用性群組。

### <a name="set-up-a-storage-account-as-cloud-witness"></a>將儲存體帳戶設定為雲端見證

為了設定雲端見證，Contoso 必須要有 Azure 儲存體帳戶，用來保存用於叢集仲裁的 Blob 檔案。 相同的儲存體帳戶也可用來設定多個叢集的雲端見證。 

Contoso 管理員會建立儲存體帳戶，如下所示：

1. 他們為帳戶指定可辨識的名稱 (**contosocloudwitness**)。
2. 他們部署具有 LRS 的一般通用帳戶。
3. 他們將帳戶放置於第三個區域 - 美國中南部。 他們將它放在主要和次要區域外，使其在發生區域性失敗時仍可供使用。
4. 他們將它放在用來保存基礎結構資源的資源群組中 - **ContosoInfraRG**。

    ![雲端見證](media/contoso-migration-rehost-vm-sql-ag/witness-storage.png)

5. 當他們建立儲存體帳戶時，系統會產生該帳戶的主要和次要存取金鑰。 他們必須要有主要存取金鑰才能建立雲端見證。 金鑰會出現在儲存體帳戶名稱 > [存取金鑰] 下方。

    ![存取金鑰](media/contoso-migration-rehost-vm-sql-ag/access-key.png)

### <a name="add-sql-server-vms-to-contoso-domain"></a>將 SQL Server VM 新增至 Contoso 網域

1. Contoso 將 SQLAOG1 和 SQLAOG2 新增至 contoso.com 網域。
2. 然後，他們在每個 VM 上安裝 Windows 容錯移轉叢集功能和工具。

### <a name="set-up-the-cluster"></a>設定叢集

在設定叢集之前，Contoso 管理員為每個機器的作業系統磁碟建立快照集。

![快照集](media/contoso-migration-rehost-vm-sql-ag/snapshot.png)

2. 然後，他們執行已整合的指令碼，以建立 Windows 容錯移轉叢集。

    ![建立叢集](media/contoso-migration-rehost-vm-sql-ag/create-cluster1.png)

3. 在建立叢集之後，他們確認 VM 已呈現為叢集節點。

     ![建立叢集](media/contoso-migration-rehost-vm-sql-ag/create-cluster2.png)

## <a name="configure-the-cloud-witness"></a>設定雲端見證

1. Contoso 管理員使用容錯移轉叢集管理員中的**仲裁設定精靈**來設定雲端見證。
2. 在此精靈中，他們選擇使用儲存體帳戶來建立雲端見證。
3. 雲端見證設定後，會出現在容錯移轉叢集管理員嵌入式管理單元中。

    ![雲端見證](media/contoso-migration-rehost-vm-sql-ag/cloud-witness.png)
            

### <a name="enable-sql-server-always-on-availability-groups"></a>啟用 SQL Server Always On 可用性群組

Contoso 管理員現在已可啟用 Always On：

1. 在 SQL Server 組態管理員中，他們為 **SQL Server (MSSQLSERVER)** 服務啟用 **AlwaysOn 可用性群組**。

    ![啟用 AlwaysOn](media/contoso-migration-rehost-vm-sql-ag/enable-alwayson.png)

2. 他們重新啟動服務讓變更生效。

在 AlwaysOn 啟用後，Contoso 可以設定將會保護 SmartHotel360 資料庫的 AlwaysOn 可用性群組。

**需要其他協助？**

- [深入了解](https://docs.microsoft.com/windows-server/failover-clustering/deploy-cloud-witness)雲端見證以及如何為其設定儲存體帳戶。
- [取得指示](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-tutorial)以了解如何設定叢集和建立可用性群組。

## <a name="step-3-deploy-the-azure-load-balancer"></a>步驟 3：部署 Azure Load Balancer

Contoso 管理員此時想要部署位於叢集節點前面的內部負載平衡器。 負載平衡器會接聽流量，並將其導向至適當的節點。

![負載平衡](media/contoso-migration-rehost-vm-sql-ag/architecture-lb.png)

他們依照下列方式建立負載平衡器︰

1. 在 Azure 入口網站 > [網路] > [負載平衡器] 中，他們設定新的內部負載平衡器：**ILB-PROD-DB-EUS2-SQLAOG**。
2. 他們將負載平衡器放在生產網路 **VNET-PROD-EUS2** 的資料庫子網路 **PROD-DB-EUS2** 中。
3. 他們為其指派靜態 IP 位址：10.245.40.100。
4. 他們在網路資源群組 **ContosoNetworkingRG** 中部署負載平衡器，作為網路元素。

    ![負載平衡](media/contoso-migration-rehost-vm-sql-ag/lb-create.png)

部署內部負載平衡器後，他們必須加以設定。 他們建立後端位址集區、設定健康情況探查，並設定負載平衡規則。

### <a name="add-a-backend-pool"></a>新增後端集區

為了將流量分散至叢集中的各個 VM，Contoso 管理員設定了後端位址集區，其中包含將會從負載平衡器接收網路流量的 VM NIC IP 位址。

1. 在入口網站的負載平衡器設定中，Contoso 新增了後端集區：**ILB-PROD-DB-EUS-SQLAOG-BEPOOL**。
2. 他們建立了該集區與可用性設定組 SQLAOGAVSET 的關聯。 設定組中的 VM (**SQLAOG1** 和 **SQLAOG2**) 會新增至集區。

    ![後端集區](media/contoso-migration-rehost-vm-sql-ag/backend-pool.png)

### <a name="create-a-health-probe"></a>建立健康狀態探查

Contoso 管理員建立了健康情況探查，讓負載平衡器能夠監視應用程式的健康情況。 此探查會根據 VM 對健康情況檢查的回應，以動態方式從負載平衡器輪替中新增或移除 VM。

他們依照下列方式建立探查： 

1. 在入口網站的負載平衡器設定中，Contoso 建立健康情況探查：**SQLAlwaysOnEndPointProbe**。
2. 他們設定探查以監視 TCP 連接埠 59999 上的 VM。
3. 他們在探查間設定 5 秒的間隔，並將閾值設定為 2。 如果有兩個探查失敗，VM 即會被視為狀況不良。

    ![探查](media/contoso-migration-rehost-vm-sql-ag/nlb-probe.png)

### <a name="configure-the-load-balancer-to-receive-traffic"></a>設定要接收流量的負載平衡器


此時，Contoso 管理員必須設定負載平衡器規則，以定義如何將流量分散至 VM。

- 前端 IP 位址會處理傳入流量。
- 後端 IP 集區會接收流量。

他們依照下列方式建立規則：

1. 在入口網站的負載平衡器設定中，他們新增負載平衡規則：**SQLAlwaysOnEndPointListener**。
2. 他們設定了前端接聽程式，以在 TCP 1433 上接收傳入的 SQL 用戶端流量。
3. 他們指定流量所將路由到的後端集區，以及 VM 用來接聽流量的連接埠。
4. 他們啟用了浮動 IP (伺服器直接回傳)。 SQL AlwaysOn 一律需要此功能。

    ![探查](media/contoso-migration-rehost-vm-sql-ag/nlb-probe.png)

**需要其他協助？**

- 對 Azure Load Balancer [取得概觀](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview)。
- [了解](https://docs.microsoft.com/azure/load-balancer/tutorial-load-balancer-basic-internal-portal)如何建立負載平衡器。



## <a name="step-4-prepare-azure-for-the-site-recovery-service"></a>步驟 4：為 Azure 進行 Site Recovery 服務的準備工作

以下是 Contoso 部署 Site Recovery 時所需的 Azure 元件：

- 在容錯移轉時建立 VM 時，這些 VM 所在的 VNet。
- 一個 Azure 儲存體帳戶，用來保存複寫的資料。 
- Azure 中的復原服務保存庫。

Contoso 管理員的設定方式如下：

1.  Contoso 在[部署 Azure 基礎結構](contoso-migration-rehost-vm-sql-ag.md)時，即已建立可用於 Site Recovery 的網路/子網路。

    - SmartHotel360 應用程式是一個生產應用程式，而 WEBVM 將會移轉至美國東部 2 主要區域中的 Azure 生產網路 (VNET-PROD-EUS2)。
    - WEBVM 將會置於 ContosoRG 資源群組中 (用於生產資源)，並且置於生產子網路中 (PROD-FE-EUS2)。

2. Contoso 管理員會在主要區域中建立 Azure 儲存體帳戶 (contosovmsacc20180528)。

    - 他們使用一般用途的帳戶，並配備標準儲存體和 LRS 複寫。
    - 此帳戶必須位於與保存庫相同的區域中。

    ![Site Recovery 儲存體](media/contoso-migration-rehost-vm-sql-ag/asr-storage.png)

3. 網路和儲存體帳戶準備就緒之後，他們現在會建立復原服務保存庫 (**ContosoMigrationVault**)，然後將其置於美國東部 2 主要區域的 **ContosoFailoverRG** 資源群組中。

    ![復原服務保存庫](media/contoso-migration-rehost-vm-sql-ag/asr-vault.png)

**需要其他協助？**

[了解](https://docs.microsoft.com/azure/site-recovery/tutorial-prepare-azure)如何為 Azure 進行 Site Recovery 的設定。


## <a name="step-5-prepare-on-premises-vmware-for-site-recovery"></a>步驟 5：為內部部署 VMware 進行 Site Recovery 的準備工作

以下是 Contoso 管理員為內部部署所做的準備：

- vCenter Server 或 vSphere ESXi 主機上的一個帳戶，用以自動執行 VM 探索。
- 一個帳戶，用以在您想要複寫的 VMware VM 上自動安裝行動服務。
- 內部部署 VM 設定，讓 Contoso 在容錯移轉之後可以連線至複寫的 Azure VM。


### <a name="prepare-an-account-for-automatic-discovery"></a>準備帳戶以進行自動探索

Site Recovery 需要存取 VMware 伺服器才能：

- 自動探索 VM。 
- 協調複寫、容錯移轉和容錯回復。
- 需要至少一個唯讀帳戶。 您需要可執行建立和移除磁碟以及開啟 VM 等作業的帳戶。

Contoso 管理員會依照下列方式設定帳戶：

1. 他們在 vCenter 層級建立一個角色。
2. 然後，他們會指派必要權限給角色。



### <a name="prepare-an-account-for-mobility-service-installation"></a>準備一個用來安裝行動服務的帳戶

行動服務必須安裝在每一個 VM 上。

- 當 VM 的複寫功能啟用時，Site Recovery 可以執行此元件的自動推送安裝。
- 您必須有一個帳戶可供 Site Recovery 用來存取 VM 以進行推送安裝。 當您在 Azure 主控台中設定複寫時，要指定此帳戶。
- 此帳戶可以是網域帳戶或本機帳戶，且具有在 VM 上進行安裝的權限。




### <a name="prepare-to-connect-to-azure-vms-after-failover"></a>準備在容錯移轉後連接到 Azure VM

容錯移轉之後，Contoso 想要能夠連線至 Azure VM。 為此，Contoso 管理員在移轉之前執行了下列作業：

1. 為了透過網際網路存取，他們：

 - 先在內部部署 VM 上啟用 RDP，然後再容錯移轉
 - 確定已為**公用**設定檔新增 TCP 和 UDP 規則。
 - 針對所有設定檔，在 [Windows 防火牆] > [允許的應用程式] 中檢查是否允許 RDP。
 
2. 為了透過站對站 VPN 存取，他們：

 - 在內部部署機器上啟用 RDP。
 - 在 [Windows 防火牆] -> [允許的應用程式與功能] 中，允許 [網域和私人] 網路使用 RDP。
 - 將內部部署 VM 的作業系統 SAN 原則設定為 **OnlineAll**。

此外，當他們執行容錯移轉時，需要檢查以下各項：

- 觸發容錯移轉時，VM 上不應該有擱置的 Windows 更新。 如果有，在更新完成之前，使用者將無法登入 VM。
- 在容錯移轉之後，他們可以勾選 [開機診斷] 以檢視 VM 的螢幕擷取畫面。 若未解決問題，他們應確認 VM 是否執行中，並檢閱這些[疑難排解祕訣](http://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx)。


**需要其他協助？**

- [了解](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial-prepare-on-premises#prepare-an-account-for-automatic-discovery)如何建立和指派自動探索所需的角色。
- [深入了解](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial-prepare-on-premises#prepare-an-account-for-mobility-service-installation)建立行動服務推送安裝的帳戶。


## <a name="step-6-replicate-the-on-premises-vms-to-azure-with-site-recovery"></a>步驟 6：使用 Site Recovery 將內部部署 VM 複寫至 Azure

Contoso 管理員必須先設定並啟用複寫，才能執行移轉至 Azure 的作業。

### <a name="set-a-replication-goal"></a>設定複寫目標

1. 在保存庫中的保存庫名稱 (ContosoVMVault) 下，他們選取複寫目標 ([開始使用] > [Site Recovery] > [準備基礎結構])。
2. 他們指定其機器都是內部部署、在 VMware 上執行，並複寫至 Azure。

    ![複寫目標](./media/contoso-migration-rehost-vm-sql-ag/replication-goal.png)

### <a name="confirm-deployment-planning"></a>確認部署規劃

若要繼續，他們必須選取 [是，我已經完成]，確認自己已經完成部署規劃。 在此案例中，Contoso 僅移轉一個 VM，不需要部署計劃。

### <a name="set-up-the-source-environment"></a>設定來源環境

Contoso 管理員必須設定其來源環境。 為此，他們下載 OVF 範本，然後用它將 Site Recovery 組態伺服器部署為高可用性、內部部署的 VMware VM。 當組態伺服器開始執行之後，他們將它註冊到保存庫。

組態伺服器會執行多項元件：

- 組態伺服器元件會協調內部部署與 Azure 之間的通訊，以及管理資料複寫。
- 處理序伺服器，可作為複寫閘道。 負責接收複寫資料，以快取、壓縮和加密進行最佳化，然後將複寫資料傳送至 Azure 儲存體。
- 處理序伺服器也會在您要複寫的 VM 上安裝行動服務，並且在內部部署 VMware VM 上執行自動探索。

Contoso 管理員會依照下列方式執行這些步驟：


1. 在保存庫中，他們從 [準備基礎結構] > [來源] > [組態伺服器] 下載 OVF 範本。
    
    ![下載 OVF](./media/contoso-migration-rehost-vm-sql-ag/add-cs.png)

2. 他們將範本匯入 VMware 中，以建立和部署 VM。

    ![OVF 範本](./media/contoso-migration-rehost-vm-sql-ag/vcenter-wizard.png)

3. 當他們第一次啟動 VM 時，VM 會在開機後進入 Windows Server 2016 安裝程序。 他們接受授權合約，並輸入系統管理員密碼。
4. 安裝完成之後，他們以系統管理員身分登入 VM。 第一次登入時，依預設會執行 Azure Site Recovery 設定工具。
5. 在此工具中，他們指定一個名稱，用以在保存庫中註冊組態伺服器。
6. 此工具會檢查 VM 是否可連線到 Azure。 連線建立後，他們登入 Azure 訂用帳戶。 認證必須能夠存取您要在其中註冊組態伺服器的保存庫。

    ![註冊設定伺服器](./media/contoso-migration-rehost-vm-sql-ag/config-server-register2.png)

7. 此工具會執行一些設定工作，而後重新開機。
8. 他們再次登入機器，組態伺服器管理精靈隨即自動啟動。
9. 在此精靈中，他們選取要接收複寫流量的 NIC。 這項設定在完成之後即無法變更。
10. 他們選取訂用帳戶、資源群組，以及在其中註冊組態伺服器的保存庫。
        ![保存庫](./media/contoso-migration-rehost-vm-sql-ag/cswiz1.png) 

10. 接著，他們下載並安裝 MySQL Server 和 VMWare PowerCLI。 
11. 驗證之後，他們指定 vCenter 伺服器或 vSphere 主機的 FQDN 或 IP 位址。 他們保留預設的連接埠，並且為 vCenter 伺服器指定易記名稱。
12. 他們指定先前為自動探索建立的帳戶，以及用來自動安裝行動服務的認證。 對於 Windows 機器，此帳戶需具備 VM 的本機系統管理員權限。

    ![vCenter](./media/contoso-migration-rehost-vm-sql-ag/cswiz2.png)

7. 註冊完成後，在 Azure 入口網站中確認組態伺服器和 VMware 伺服器都已列在保存庫中的 [來源] 頁面上。 探索可能需要 15 分鐘以上的時間。 
8. 接下來，Site Recovery 會使用指定的設定連線至 VMware 伺服器，並探索 VM。

### <a name="set-up-the-target"></a>設定目標

Contoso 管理員現在會指定目標複寫設定。

1. 在 [準備基礎結構] > [目標] 中，他們選取目標設定。
2. Site Recovery 會確認在指定的目標中有 Azure 儲存體帳戶和網路。

### <a name="create-a-replication-policy"></a>建立複寫原則

現在，Contoso 管理員可以建立複寫原則。

1. 在 [準備基礎結構] > [複寫設定] > [複寫原則] >  [建立和關聯] 中，他們建立 **ContosoMigrationPolicy** 原則。
2. 他們使用預設設定：
    - **RPO 閾值**：預設值是 60 分鐘。 這個值可定義復原點的建立頻率。 連續複寫超過此限制時會產生警示。
    - **復原點保留**。 預設值為 24 小時。 此值會指定每個復原點的保留週期有多長。 複寫的 VM 可以還原至一個週期內的任何時間點。
    - **應用程式一致快照頻率**。 預設值是一小時。 此值會指定應用程式一致快照的建立頻率。
 
        ![建立複寫原則](./media/contoso-migration-rehost-vm-sql-ag/replication-policy.png)

5. 此原則會自動與設定伺服器產生關聯。 

    ![關聯複寫原則](./media/contoso-migration-rehost-vm-sql-ag/replication-policy2.png)



### <a name="enable-replication"></a>啟用複寫

現在，Contoso 管理員可以開始複寫 WebVM。

1. 在 [複寫應用程式] > [來源] > [+複寫] 中，他們選取來源設定。
2. 他們表示想要啟用 VM、選取 vCenter 伺服器和組態伺服器。

    ![啟用複寫](./media/contoso-migration-rehost-vm-sql-ag/enable-replication1.png)

3. 現在，他們指定目標設定 (包括資源群組和 VNet)，以及用來儲存複寫資料的儲存體帳戶。

     ![啟用複寫](./media/contoso-migration-rehost-vm-sql-ag/enable-replication2.png)

3. 他們選取要複寫的 WebVM、檢查複寫原則，並啟用複寫。 Site Recovery 會在複寫啟用時在 VM 上安裝行動服務。
 
    ![啟用複寫](./media/contoso-migration-rehost-vm-sql-ag/enable-replication3.png)

4. 他們在 [作業] 中追踨複寫進度。 執行 [完成保護] 作業之後，機器即準備好進行容錯移轉。
5. 在 Azure 入口網站的 [基本資訊] 中，他們可以看到 VM 複寫至 Azure 的結構。

    ![基礎結構檢視](./media/contoso-migration-rehost-vm-sql-ag/essentials.png)


**需要其他協助？**

- 您可以在[設定內部部署 VMware VM 的災害復原](https://docs.microsoft.com/azure/site-recovery/vmware-azure-tutorial)中，閱讀所有步驟的詳細逐步解說。
- 您會得到詳細的操作說明，協助您[設定來源環境](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-source)、[部署組態伺服器](https://docs.microsoft.com/azure/site-recovery/vmware-azure-deploy-configuration-server)以及[指定複寫設定](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-replication)。
- 您可以深入了解如何[啟用複寫](https://docs.microsoft.com/azure/site-recovery/vmware-azure-enable-replication)。


## <a name="step-7-install-the-database-migration-assistant-dma"></a>步驟 7：安裝 Database Migration Assistant (DMA)

Contoso 管理員會使用 DMA 將 SmartHotel360 資料庫移轉至 Azure VM **SQLAOG1**。 他們依照下列方式設定 DMA：

1. 他們從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=53595)將工具下載至內部部署的 SQL Server VM (**SQLVM**)。
2. 他們在 VM 上執行安裝程式 (DownloadMigrationAssistant.msi)。
3. 在 [完成] 頁面上，他們會先選取 [啟動 Microsoft Data Migration Assistant]，再完成精靈。

## <a name="step-8-migrate-the-database-with-dma"></a>步驟 8：使用 DMS 移轉資料庫

1. 在 DMA 中，他們執行新的移轉 - **SmartHotel**。
2. 他們選取 [Azure 虛擬機器上的 SQL Server] 作為 [目標伺服器類型]。 

    ![DMA](media/contoso-migration-rehost-vm-sql-ag/dma-1.png)

3. 在移轉詳細資料中，他們新增 **SQLVM** 作為來源伺服器，並新增 **SQLAOG1** 作為目標。 他們指定每個機器的認證。

     ![DMA](media/contoso-migration-rehost-vm-sql-ag/dma-2.png)

4. 他們建立資料庫和組態資訊的本機共用。 SQLVM 和 SQLAOG1 的 SQL 服務帳戶必須可透過寫入存取權來存取該共用位置。

    ![DMA](media/contoso-migration-rehost-vm-sql-ag/dma-3.png)

5. Contoso 選取應移轉的登入，並開始進行移轉。 完成之後，DMA 會顯示移轉成功。

    ![DMA](media/contoso-migration-rehost-vm-sql-ag/dma-4.png)

6. 他們確認資料庫執行於 **SQLAOG1** 上。

    ![DMA](media/contoso-migration-rehost-vm-sql-ag/dma-5.png)

DMS 會透過 Contoso 資料中心與 Azure 之間的站對站 VPN 連線來連線至內部部署 SQL Server VM，然後再遷移資料庫。

## <a name="step-7-protect-the-database-with-alwayson"></a>步驟 7：使用 AlwaysOn 保護資料庫

在應用程式資料庫執行於 **SQLAOG1** 的情況下，Contoso 管理員現在可以使用 AlwaysOn 可用性群組加以保護。 他們使用 SQL Management Studio 設定 AlwaysOn，然後使用 Windows 叢集指派接聽程式。 

### <a name="create-an-alwayson-availability-group"></a>建立 AlwaysOn 可用性群組

1. 在 SQL Management Studio 中，他們以滑鼠右鍵按一下 [Always On 高可用性] 以啟動 [新增可用性群組精靈]。
2. 在 [指定選項] 中，他們將可用性群組命名為 **SHAOG**。 在 [選取資料庫] 中，他們選取 SmartHotel360 資料庫。

    ![AlwaysOn 可用性群組](media/contoso-migration-rehost-vm-sql-ag/aog-1.png)

3. 在 [指定複本] 中，他們新增兩個 SQL 節點作為可用性複本，並將其設定為透過同步認可提供自動容錯移轉。

     ![AlwaysOn 可用性群組](media/contoso-migration-rehost-vm-sql-ag/aog-2.png)

4. 他們設定群組 (**SHAOG**) 和連接埠的接聽程式。 內部負載平衡器的 IP 位址會新增為靜態 IP 位址 (10.245.40.100)。

    ![AlwaysOn 可用性群組](media/contoso-migration-rehost-vm-sql-ag/aog-3.png)

5. 在 [選取資料同步處理] 中，他們啟用自動植入。 透過此選項，SQL Server 會自動為群組中的每個資料庫中建立次要複本，因此 Contoso 不需要手動加以備份和還原。 驗證之後，會建立可用性群組。

    ![AlwaysOn 可用性群組](media/contoso-migration-rehost-vm-sql-ag/aog-4.png)

6. Contoso 在建立群組時發生問題。 他們未使用 Active Directory Windows 整合式安全性，因此需要授與權限給 SQL 登入，以建立 Windows 容錯移轉叢集角色。

    ![AlwaysOn 可用性群組](media/contoso-migration-rehost-vm-sql-ag/aog-5.png)

6. 群組建立後，Contoso 可以在 SQL Management Studio 中加以檢視。

### <a name="configure-a-listener-on-the-cluster"></a>在叢集上設定接聽程式

在設定 SQL 部署的最後一個步驟中，Contoso 管理員將內部負載平衡器設定為叢集上的接聽程式，並使接聽程式上線。 他們使用指令碼來執行此作業。

![叢集接聽程式](media/contoso-migration-rehost-vm-sql-ag/cluster-listener.png)


### <a name="verify-the-configuration"></a>驗證組態

在所有設定完成後，Contoso 此時在使用移轉後資料庫的 Azure 中已有可運作的可用性群組。 管理員可藉由在 SQL Management Studio 中連線至內部負載平衡器來確認這一點。

![ILB 連線](media/contoso-migration-rehost-vm-sql-ag/ilb-connect.png)

**需要其他協助？**
- 了解如何建立[可用性群組](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-tutorial#create-the-availability-group)和[接聽程式](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-tutorial#configure-listener)。
- 手動[設定叢集以使用負載平衡器 IP 位址](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener#configure-the-cluster-to-use-the-load-balancer-ip-address)。
- [深入了解](https://docs.microsoft.com/azure/storage/blobs/storage-dotnet-shared-access-signature-part-2)如何建立和使用 SAS。


## <a name="step-8-migrate-the-vm-with-site-recovery"></a>步驟 8：使用 Site Recovery 移轉 VM

Contoso 管理員會執行快速測試容錯移轉，然後移轉 VM。

### <a name="run-a-test-failover"></a>執行測試容錯移轉

在移轉之前執行測試容錯移轉，有助於確保一切運作正常。 

1. 他們會針對最新的可用時間點 (**最近處理的**) 執行一次測試容錯移轉。
2. 如果想在觸發容錯移轉之前，讓 Site Recovery 嘗試將來源 VM 關機，請選取 [先將機器關機再開始容錯移轉]。 即使關機失敗，仍會繼續容錯移轉。 
3. 執行測試容錯移轉： 

    - 系統會執行先決條件檢查，以確保所有移轉所需的條件都已準備就緒。
    - 容錯移轉會處理資料，以便建立 Azure VM。 若選取最新的復原點，則會根據資料建立復原點。
    - 將會使用先前步驟中處理的資料來建立 Azure VM。

3. 容錯移轉完成後，Azure VM 複本會出現在 Azure 入口網站中。 他們會確認 VM 為適當的大小、其已連線到正確的網路，而且正在執行中。 
4. 確認之後，他們可以清除容錯移轉，並記錄與儲存任何觀察到的結果。 

### <a name="run-a-failover"></a>執行容錯移轉

1. 驗證測試容錯移轉如預期般正常運作之後，Contoso 管理員會建立移轉的復原計畫，並將 WEBVM 新增至計畫。

     ![復原計畫](./media/contoso-migration-rehost-vm-sql-ag/recovery-plan.png)

2. 他們根據此計畫執行容錯移轉。 他們選取最新的復原點，然後指定 Site Recovery 應該在嘗試觸發容錯移轉之前，先關閉內部部署 VM。

    ![容錯移轉](./media/contoso-migration-rehost-vm-sql-ag/failover1.png)

3. 容錯移轉之後，他們可以確認 Azure VM 會如預期般出現在 Azure 入口網站中。

    ![復原計畫](./media/contoso-migration-rehost-vm-sql-ag/failover2.png)

6. 在 Azure 中確認 VM 之後，他們會完成移轉程序、停止 VM 複寫，以及停止 VM 的 Site Recovery 計費。

    ![容錯移轉](./media/contoso-migration-rehost-vm-sql-ag/failover3.png)

### <a name="update-the-connection-string"></a>更新連接字串

在移轉程序的最後一個步驟中，Contoso 管理員會將應用程式的連接字串更新為指向在其 SHAOG 接聽程式上執行的移轉後資料庫。 此組態會在目前執行於 Azure 中的 WEBVM 上變更。  此組態位於ASP 應用程式的 web.config 中。 

1. 在 C:\inetpub\SmartHotelWeb\web.config 上找出檔案。變更伺服器的名稱以反映 AOG 的 FQDN：shaog.contoso.com。

    ![容錯移轉](./media/contoso-migration-rehost-vm-sql-ag/failover4.png)  

2. 更新該檔案並加以儲存後，他們會在 WEBVM 上重新啟動 IIS。 他們從 CMD 命令提示字元使用 IISRESET /RESTART 執行此動作。
2. IIS 重新啟動後，應用程式此時會使用在 SQL MI 上執行的資料庫。


**需要其他協助？**

- [了解](https://docs.microsoft.com/azure/site-recovery/tutorial-dr-drill-azure)如何執行測試容錯移轉。 
- [了解](https://docs.microsoft.com/azure/site-recovery/site-recovery-create-recovery-plans)如何建立復原計畫。
- [了解](https://docs.microsoft.com/azure/site-recovery/site-recovery-failover)如何容錯移轉至 Azure。

## <a name="clean-up-after-migration"></a>移轉之後進行清除

移轉之後，SmartHotel360 應用程式會在 Azure VM 上執行，而 SmartHotel360 資料庫會位於 Azure SQL 叢集中。

現在，Contoso 必須完成以下的清除步驟：  

- 從 vCenter 清查中移除內部部署 VM。
- 從本機備份作業中移除 VM。
- 更新內部文件以顯示 VM 的新位置和 IP 位址。
- 檢閱與已解除委任 VM 互動的任何資源，並更新任何相關的設定或文件，以反映新的組態。
- 新增兩個應新增至生產監視系統的新 VM (SQLAOG1 和 SQLAOG2)。

## <a name="review-the-deployment"></a>檢閱部署

對於 Azure 中的移轉後資源，Contoso 必須能執行一切功能並保護其新的基礎結構。

### <a name="security"></a>安全性

Contoso 安全性小組會檢查 WEBVM、SQLAOG1 和 SQLAOG2 等 Azure VM，以判斷是否有任何的安全疑慮。 

- 小組會檢閱 VM 的網路安全性群組 (NSG) 以控制存取權。 NSG 可用來確保只可以傳遞該應用程式允許的流量。
- 小組考慮使用 Azure 磁碟加密和 KeyVault 來保護磁碟上的資料。
- 小組應評估透明資料加密 (TDE)，然後在執行於新的 SQL AOG 的 SmartHotel360 資料庫上加以啟用。 [深入了解](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption?view=sql-server-2017)。

[深入了解](https://docs.microsoft.com/azure/security/azure-security-best-practices-vms#vm-authentication-and-access-control) VM 的安全措施。


## <a name="bcdr"></a>BCDR

 針對商務持續性和災害復原 (BCDR)，Contoso 採取下列動作：
 - 保護資料安全：Contoso 會使用 Azure 備份服務，備份 WEBVM、SQLAOG1 和 SQLAOG2 VM 上的資料。 [深入了解]。
(https://docs.microsoft.com/azure/backup/backup-introduction-to-azure-backup?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
- Contoso 也將了解如何使用 Azure 儲存體將 SQL Server 直接備份至 Blob 儲存體。 [深入了解](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-use-storage-sql-server-backup-restore)。
- 保持應用程式啟動及執行：Contoso 會使用 Site Recovery，在 Azure 中將應用程式 VM 複寫至次要區域。 [深入了解](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-quickstart)。


### <a name="licensing-and-cost-optimization"></a>授權和成本最佳化

1. Contoso 具有 WEBVM 的現有授權，而且會利用 Azure Hybrid Benefit。  Contoso 會轉換現有的 Azure VM，以便充分利用這個定價。
2. Contoso 會啟用 Microsoft 子公司 Cloudyn 授權的 Azure 成本管理。 它是一種多雲端成本管理解決方案，可協助您利用和管理 Azure 和其他雲端資源。  [深入了解](https://docs.microsoft.com/azure/cost-management/overview) Azure 成本管理。 

## <a name="conclusion"></a>結論

在本文中，Contoso 使用 Site Recovery 服務將應用程式前端 VM 移轉至 Azure，以重新裝載 SmartHotel360 應用程式。 Contoso 將應用程式資料庫移轉至佈建在 Azure 中的 SQL Server 叢集，並在 SQL Server AlwaysOn 可用性群組中加以保護。

## <a name="next-steps"></a>後續步驟

在此系列的下一篇文章中，我們將說明 Contoso 如何重新裝載在 Linux 上執行的服務台 osTicket 應用程式，並使用 MySQL 資料庫進行部署。


