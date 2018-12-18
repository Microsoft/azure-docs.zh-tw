---
title: Azure Migrate - 常見問題集 (FAQ) | Microsoft Docs
description: 解說 Azure Migrate 的相關常見問題
author: snehaamicrosoft
ms.service: azure-migrate
ms.topic: conceptual
ms.date: 09/03/2018
ms.author: snehaa
ms.openlocfilehash: ce9dc4aab26b99bbb1e9f24f018354b8c91f66f4
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43699959"
---
# <a name="azure-migrate---frequently-asked-questions-faq"></a>Azure Migrate - 常見問題集 (FAQ)

本文包含有關 Azure Migrate 的常見問題集。 如果您在閱讀本文後仍有其他問題，請將問題張貼在 [Azure Migrate 論壇](http://aka.ms/AzureMigrateForum)。

## <a name="general"></a>一般

### <a name="does-azure-migrate-support-assessment-of-only-vmware-workloads"></a>Azure Migrate 是否僅支援 VMware 工作負載的評估？

是，Azure Migrate 目前僅支援 VMware 工作負載的評估。 未來將啟用對 Hyper-V 和實體伺服器的支援。

### <a name="does-azure-migrate-need-vcenter-server-to-discover-a-vmware-environment"></a>Azure Migrate 是否需要 vCenter Server 才能探索 VMware 環境？

是，Azure Migrate 需要 vCenter Server 才能探索 VMware 環境， 且不支援探索非由 vCenter Server 管理的 ESXi 主機。

### <a name="how-is-azure-migrate-different-from-azure-site-recovery"></a>Azure Migrate 和 Azure Site Recovery 有何不同？

Azure Migrate 是一項評估服務，可協助您探索內部部署工作負載，並規劃移轉至 Azure 的作業。 搭配 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/migrate-tutorial-on-premises-azure) 作為災難復原解決方案，可協助您將內部部署工作負載移轉至 Azure 中的 IaaS 虛擬機器。

### <a name="whats-the-difference-between-using-azure-migrate-for-assessments-and-the-map-toolkit"></a>使用 Azure Migrate 評量和 Map Toolkit 之間的差異為何？

[Azure Migrate](migrate-overview.md) 提供專為協助移轉整備與評估內部部署工作負載至 Azure 的移轉評量。 [Microsoft Assessment and Planning (MAP) Toolkit](https://www.microsoft.com/en-us/download/details.aspx?id=7826) 有其他功能。 例如，較新版本之 Windows 用戶端和伺服器作業系統的移轉規劃、軟體使用情況追蹤等。若有這些情況，請繼續使用 MAP Toolkit。


### <a name="how-is-azure-migrate-different-from-azure-site-recovery-deployment-planner"></a>Azure Migrate 和 Azure Site Recovery 部署規劃工具有何不同？

Azure Migrate 是移轉規劃工具，而 Azure Site Recovery 部署規劃工具則是災難復原 (DR) 規劃工具。

**從 VMware 移轉至 Azure**：如果您要將內部部署工作負載移轉至 Azure，請使用 Azure Migrate 規劃移轉作業。 Azure Migrate 會評估內部部署工作負載，並提供指引、見解和機制，協助您移轉至 Azure。 將移轉計劃準備就緒後，您可以使用 Azure Site Recovery 和 Azure 資料庫移轉服務等服務，將機器移轉至 Azure。

**從 Hyper-V 移轉至 Azure**：Azure Migrate 目前僅支援評估 VMware 虛擬機器至 Azure 的移轉。 Hyper-V 支援已列在 Azure Migrate 藍圖規劃中。 在此過渡期間，您可使用 Site Recovery 部署規劃工具。 一旦 Azure Migrate 啟用 Hyper-V 支援，您即可使用 Azure Migrate 規劃移轉 Hyper-V 的工作負載。

**從 VMware/Hyper-V 移轉至 Azure 的災難復原**：如果您預計使用 Azure Site Recovery (Site Recovery) 在 Azure 上執行災難復原 (DR)，請使用 Site Recovery 部署規劃工具進行 DR 規劃。 Site Recovery 部署規劃工具可針對您的內部部署環境進行深層的 ASR 特定評估。 此外，還可針對如何成功執行複寫、虛擬機器容錯移轉等 DR 作業提供必要的 Site Recovery 建議。  

### <a name="which-azure-regions-are-supported-by-azure-migrate"></a>Azure Migrate 支援哪些 Azure 區域？

Azure Migrate 目前支援以美國東部和美國中西部作為移轉專案位置。 請注意，即使您只能在美國中西部和美國東部建立移轉專案，您還是可以針對[多個目標位置](https://docs.microsoft.com/azure/migrate/how-to-modify-assessment#edit-assessment-properties)來評估機器。 專案位置只會用來儲存探索到的資料。

### <a name="how-does-the-on-premises-site-connect-to-azure-migrate"></a>內部部署站台如何連線至 Azure Migrate？

連線可以透過網際網路或使用 ExpressRoute 與公用對等互連。

### <a name="can-i-harden-the-vm-set-up-with-the-ova-template"></a>我可以使用 .OVA 範本強化虛擬機器設定嗎？

只要 Azure Migrate 設備運作所需的通訊和防火牆規則保持不變，即可將其他元件 (例如防毒軟體) 新增至 .OVA 範本中。   

## <a name="discovery-and-assessment"></a>探索和評估

### <a name="what-data-is-collected-by-azure-migrate"></a>Azure Migrate 會收集哪些資料？

Azure Migrate 支援兩種探索，分別是設備為基礎及代理程式為基礎的探索。
設備為基礎的探索會收集有關內部部署虛擬機器的中繼資料，設備收集的中繼資料完整清單如下：

**虛擬機器的組態資料**
- 虛擬機器顯示名稱 (在 vCenter 上)
- 虛擬機器清查路徑 (vCenter 中的主機/叢集/資料夾)
- IP 位址
- MAC 位址
- 作業系統
- 核心、磁碟、NIC 數目
- 記憶體大小、磁碟大小

**虛擬機器的效能資料**
- CPU 使用率
- 記憶體使用量
- 附加至虛擬機器的每個磁碟：
  - 磁碟讀取輸送量
  - 磁碟寫入輸送量
  - 每秒的磁碟讀取作業數
  - 每秒的磁碟寫入作業數
- 附加至虛擬機器的每個網路介面卡：
  - 網路輸入
  - 網路輸出

代理程式為基礎的探索則是基於於設備型探索的選項，可協助客戶[視覺化內部部署虛擬機器的相依性](how-to-create-group-machine-dependencies.md)。 相依性代理程式會收集 FQDN、作業系統、IP 位址、MAC 位址、在虛擬機器內執行的程序，以及從虛擬機器傳出/傳入的 TCP 連線等詳細資料。 代理程式為基礎的探索是選擇性做法，如果您不想要視覺化虛擬機器的相依性，則可選擇不要安裝代理程式。

### <a name="where-is-the-collected-data-stored-and-for-how-long"></a>資料收集後會儲存在哪裡、儲存多久？

收集器設備收集的資料會儲存在您建立移轉專案時所指定的 Azure 位置。 資料會安全地儲存在 Microsoft 訂用帳戶中，而使用者刪除 Azure Migrate 專案後，資料即會一併刪除。

針對相依性視覺化，如果您在虛擬機器上安裝代理程式，則相依性代理程式收集的資料會儲存在美國由使用者訂用帳戶建立的 OMS 工作區中。 刪除訂用帳戶中的 OMS 工作區時，會刪除此資料。 [深入了解](https://docs.microsoft.com/azure/migrate/concepts-dependency-visualization)。

### <a name="is-the-data-encrypted-at-rest-and-while-in-transit"></a>待用資料和傳輸中的資料是否會加密？

是，收集的資料在待用和傳輸時都會加密。 設備收集的中繼資料會安全地傳送至 Azure Migrate 服務 (使用 https 透過網際網路傳送)。 收集的中繼資料會儲存在 Microsoft 訂用帳戶的 [Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/database-encryption-at-rest) 和 [Azure Blob 儲存體](https://docs.microsoft.com/azure/storage/common/storage-service-encryption)中，且待用時會加密。

相依性代理程式收集的資料傳輸時也會加密 (安全的 https 通道)，且會儲存在使用者訂用帳戶的 Log Analytics 工作區中； 資料待用時也會加密。

### <a name="how-does-the-collector-communicate-with-the-vcenter-server-and-the-azure-migrate-service"></a>收集器如何與 vCenter Server 和 Azure Migrate 服務進行通訊？

收集器設備會以使用者在設備中提供的認證連線至 vCenter Server (連接埠 443)。 該收集器要求 vCenter Server 使用 VMware PowerCLI 收集 vCenter Server 管理的虛擬機器相關中繼資料， 並且會從 vCenter Server 收集虛擬機器 組態資料 (核心、記憶體、磁碟、NIC 等)，以及每個虛擬機器上個月的記錄。 收集的中繼資料則會傳送至 Azure Migrate 服務 (使用 https 透過網際網路傳送) 進行評估。 [深入了解](concepts-collector.md)

### <a name="can-i-connect-the-same-collector-appliance-to-multiple-vcenter-servers"></a>我是否可以將相同的收集器設備連線到多部 vCenter 伺服器？

可以，單一收集器設備可用來探索多部 vCenter 伺服器，但無法同時進行。 需要逐一執行探索。

### <a name="is-the-ova-template-used-by-site-recovery-integrated-with-the-ova-used-by-azure-migrate"></a>Site Recovery 使用的 .OVA 範本是否與 Azure Migrate 使用的 .OVA 整合？

目前未整合。 Site Recovery 中的 .OVA 範本用來設定 VMware VM/實體伺服器複寫的 Site Recovery 設定伺服器。 基於移轉評估目的，Azure Migrate 使用的 .OVA 用於探索 vCenter 伺服器管理的 VMware VM。

### <a name="i-changed-my-machine-size-can-i-rerun-the-assessment"></a>我已變更機器大小。 是否可以重新執行評量？

如果您已針對要評估的虛擬機器變更設定，觸發程序將使用收集器設備再次探索。 請在設備中使用 [重新開始收集] 選項來執行此動作。 收集完成後，請在入口網站中選取 [重新計算] 選項進行評估，以取得更新的評估結果。

### <a name="how-can-i-discover-a-multi-tenant-environment-in-azure-migrate"></a>如何在 Azure Migrate 中探索多租用戶環境？

如果您有一個在租用戶之間共用的環境，而且不想在一個租用戶的訂用帳戶中探索另一個租用戶的虛擬機器，您可以使用收集器設備中的 [範圍] 欄位來設定探索的範圍。 如果租用戶間要共用主機，請只針對屬於特定租用戶的虛擬機器，建立具有唯讀存取權的認證，然後在收集器設備中使用此認證，並將範圍指定為要進行探索的主機。 或者，您也可以在 vCenter Server 中建立資料夾 (例如 tenant1 對 folder1 及 tenant2 對 folder2)，在共用主機下，將 tenant1 的虛擬機器移到 folder1、並將 tenant2 的虛擬機器移到 tenant2，然後據以指定適當的資料夾來設定收集器中的探索範圍。

### <a name="how-many-virtual-machines-can-be-discovered-in-a-single-migration-project"></a>單一移轉專案中可以探索多少個虛擬機器？

您在單一移轉專案中可探索 1500 台虛擬機器。 如果您的內部部署環境中有多台機器，請[進一步了解](how-to-scale-assessment.md)如何在 Azure Migrate 中探索大型環境。

### <a name="does-azure-migrate-support-enterprise-agreement-ea-based-cost-estimation"></a>Azure Migrate 是否支援以 Enterprise 合約 (EA) 為基礎的成本估計？

Azure Migrate 目前不支援 [Enterprise 合約支援方案供應項目](https://azure.microsoft.com/offers/enterprise-agreement-support/)的成本估計。 因應措施是指定「預付型方案」做為供應項目，並在評量屬性的 [折扣] 欄位中手動指定折扣百分比 (適用於訂用帳戶)。

  ![折扣](./media/resources-faq/discount.png)

## <a name="dependency-visualization"></a>相依性視覺效果

### <a name="do-i-need-to-pay-to-use-the-dependency-visualization-feature"></a>我是否需要付費才能使用相依性虛擬化功能？

不須額外費用即可使用 Azure Migrate。 [在此](https://azure.microsoft.com/pricing/details/azure-migrate/)深入了解 Azure Migrate 定價。

### <a name="can-i-use-an-existing-workspace-for-dependency-visualization"></a>是否可使用現有的工作區進行相依性視覺化？

Azure Migrate 不支援使用現有工作區進行相依性視覺化，不過，Microsoft Monitoring Agent (MMA) 支援多重首頁，可讓您將資料傳送至多個工作區。 因此，如果您已部署代理程式並將其設定為工作區，則可利用 MMA 代理程式中的多重首頁功能，將其設定為 Azure Migrate 工作區 (除了現有的工作區)，然後開始運作。 [此部落格](https://blogs.technet.microsoft.com/msoms/2016/05/26/oms-log-analytics-agent-multi-homing-support/)文章說明如何在 MMA 代理程式中啟用多重首頁。

## <a name="next-steps"></a>後續步驟

- 請參閱 [Azure Migrate 概觀](migrate-overview.md)
- 了解如何[探索和評定](tutorial-assessment-vmware.md) VMware 環境
