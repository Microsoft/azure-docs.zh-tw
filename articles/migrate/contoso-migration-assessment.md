---
title: 評定可供 Contoso 移轉到 Azure 的內部部署工作負載 | Microsoft Docs
description: 了解 Contoso 如何評定其內部部署機器，以便使用 Azure Migrate 和 Data Migration Assistant 移轉至 Azure。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: raynew
ms.openlocfilehash: 3969286b4a748a9dde8c126f91a5b19ee14a7c0d
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44303309"
---
# <a name="contoso-migration-assess-on-premises-workloads-for-migration-to-azure"></a>Contoso 移轉：評定要移轉至 Azure 的內部部署工作負載

在本文中，Contoso 會在將其內部部署 SmartHotel360 應用程式移轉至 Azure 時，對該應用程式進行評估。

本文是一系列文章中的一篇，描述 Contoso 這家虛構的公司如何將其內部部署資源遷移至 Microsoft Azure 雲端。 整個系列文章中包含背景資訊和詳細的部署案例，說明如何設定移轉基礎結構、評定內部部署資源移轉的適合性，以及執行不同類型的移轉。 案例的複雜性會提高。 隨著時間的推移，此系列會新增其他文章。

文章 | 詳細資料 | 狀態
--- | --- | ---
[文章 1：概觀](contoso-migration-overview.md) | 系列文章的概觀、Contoso 的移轉策略，以及此系列中所使用的應用程式範例。 | 可用
[文章 2：部署 Azure 基礎結構](contoso-migration-infrastructure.md) | Contoso 會準備其內部部署基礎結構和其 Azure 基礎結構以進行移轉。 此系列中的所有文章都使用相同的基礎結構。 | 可用
文章 3：存取內部部署資源以移轉至 Azure | Contoso 會評估其在 VMware 上執行的內部部署 SmartHotel360 應用程式。 Contoso 會使用 Azure Migrate 服務來評估應用程式 VM，並使用 Database Migration Assistant 來評估應用程式 SQL Server 資料庫。 | 本文
[文章 4：在 Azure VM 和 SQL Database 受控執行個體上重新裝載應用程式](contoso-migration-rehost-vm-sql-managed-instance.md) | Contoso 會為其內部部署 SmartHotel360 應用程式執行隨即移轉至 Azure 的作業。 它會使用 Azure Site Recovery 服務來遷移應用程式前端。 它會使用 Azure 資料庫移轉服務，將應用程式資料庫遷移至 Azure SQL Database 受控執行個體。 | 可用
[文章 5：在 Azure VM 上重新裝載應用程式](contoso-migration-rehost-vm.md) | Contoso 會使用 Site Recovery 服務，將其 SmartHotel360 應用程式 VM 移轉至 Azure VM。 | 可用
[文章 6：在 Azure VM 上和 SQL Server AlwaysOn 可用性群組中重新裝載應用程式](contoso-migration-rehost-vm-sql-ag.md) | Contoso 會使用 Site Recovery 來移轉應用程式 VM，以移轉 SmartHotel360 應用程式，以及使用資料庫移轉服務，將應用程式資料庫移轉至受到 AlwaysOn 可用性群組保護的 SQL Server 叢集。 | 可用
[文章 7：將 Linux 應用程式重新裝載至 Azure VM](contoso-migration-rehost-linux-vm.md) | Contoso 會使用 Site Recovery 服務，來完成將其 Linux osTicket 應用程式隨即移轉至 Azure VM 的作業。 | 可用
[文章 8：在 Azure VM 和適用於 MySQL 的 Azure 資料庫上重新裝載 Linux 應用程式](contoso-migration-rehost-linux-vm-mysql.md) | Contoso 會使用 Site Recovery，將其 Linux osTicket 應用程式遷移至 Azure VM。 它會使用 MySQL Workbench，將應用程式資料庫遷移至適用於 MySQL 的 Azure 資料庫。 | 可用
[文章 9：在 Azure Web 應用程式和 Azure SQL Database 中重構應用程式](contoso-migration-refactor-web-app-sql.md) | Contoso 會將其 SmartHotel360 應用程式移轉至 Azure Web 應用程式，以及使用 Database Migration Assistant 將應用程式資料庫移轉至 Azure SQL Server 執行個體。 | 可用
[文章 10：在 Azure Web 應用程式和適用於 MySQL 的 Azure 資料庫中重構 Linux 應用程式](contoso-migration-refactor-linux-app-service-mysql.md) | Contoso 會使用 Azure 流量管理員，將 Linux osTicket 應用程式移轉至多個 Azure 區域的 Azure Web 應用程式，與 GitHub 整合以進行持續傳遞。 Contoso 會將應用程式資料庫遷移至適用於 MySQL 的 Azure 資料庫執行個體。 | 可用
[文章 11：在 Azure DevOps Services 上重構 Team Foundation Server](contoso-migration-tfs-vsts.md) | Contoso 會將其內部部署 Team Foundation Server 部署移轉至 Azure 中的 Azure DevOps Services。 | 可用
[文章 12：在 Azure 容器和 Azure SQL Database 中重新建構應用程式](contoso-migration-rearchitect-container-sql.md) | Contoso 會將其 SmartHotel 應用程式移轉至 Azure。 然後，它會重新建構應用程式 Web 層，作為在 Azure Service Fabric 中執行的 Windows 容器，以及具有 Azure SQL Database 的資料庫。 | 可用
[文章 13：在 Azure 中重建應用程式](contoso-migration-rebuild.md) | Contoso 會藉由使用各種 Azure 功能和服務 (包括 Azure App Service、Azure Kubernetes Service (AKS)、Azure Functions、Azure 認知服務及 Azure Cosmos DB) 重建其 SmartHotel 應用程式。 | 可用


## <a name="overview"></a>概觀

Contoso 公司在考慮遷移至 Azure 時，想要進行技術和財務方面的評量，以判斷其內部部署工作負載是否適合移轉至雲端。 尤其是，Contoso 小組想要評定機器和資料庫是否能與移轉作業相容。 該小組想要預估在 Azure 中執行 Contoso 的資源所需的容量和成本。

為了開始進行和更加了解其中所涉及的技術，Contoso 會評定兩個內部部署應用程式 (在下表中摘要說明)。 該公司會評定可重新裝載和重構應用程式以供移轉的移轉案例。 在 [Contoso 移轉概觀](contoso-migration-overview.md)中深入了解重新裝載與重構。

應用程式名稱 | 平台 | 應用程式層 | 詳細資料
--- | --- | --- | ---
SmartHotel360<br/><br/> (管理 Contoso 旅遊需求) | 在含有 SQL Server 資料庫的 Windows 上執行 | 兩層式應用程式。 前端 ASP.NET 網站在一個 VM (**WEBVM**) 上執行，SQL Server 在另一個 VM (**SQLVM**) 上執行。 | VM 為 VMware，其在 vCenter Server 所管理的 ESXi 主機上執行。<br/><br/> 您可以從 [GitHub](https://github.com/Microsoft/SmartHotel360) 下載應用程式範例。
osTicket<br/><br/> (Contoso 服務台應用程式) | 在含有 MySQL PHP (LAMP) 的 Linux/Apache 上執行 | 兩層式應用程式。 前端 PHP 網站在一個 VM (**OSTICKETWEB**) 上執行，MySQL 資料庫在另一個 VM (**OSTICKETMYSQL**) 上執行。 | 客戶服務應用程式會使用該應用程式來追蹤內部員工和外部客戶的問題。<br/><br/> 您可以從 [GitHub](https://github.com/osTicket/osTicket) 下載範例。

## <a name="current-architecture"></a>目前架構

下圖顯示目前的 Contoso 內部部署基礎結構：

![目前的 Contoso 架構](./media/contoso-migration-assessment/contoso-architecture.png)  

- Contoso 有一個主要的資料中心。 該資料中心位於美國東部的紐約市。
- Contoso 在全美另有三家地區性分公司。
- 主要資料中心透過光纖都會乙太網路連線 (500 MBps) 連到網際網路。
- 每家分公司皆使用企業級連線從本機連到網際網路，並透過 IPsec VPN 通道連回主要資料中心。 此設定可讓 Contoso 的整個網路永久連線，並將網際網路連線最佳化。
- 主要資料中心已透過 VMware 完全虛擬化。 Contoso 有兩部 ESXi 6.5 虛擬化主機，均由 vCenter Server 6.5 管理。
- Contoso 使用 Active Directory 來管理身分識別。 Contoso 會使用內部網路上的 DNS 伺服器。
- 資料中心的網域控制站會在 VMware VM 上執行。 當地分公司的網域控制站會在實體伺服器上執行。

## <a name="business-drivers"></a>商業動機

Contoso 的 IT 領導小組已與該公司的企業合作夥伴密切合作，以了解這些企業想從此次移轉實現什麼目標：

- **解決業務成長**：Contoso 正在成長。 因此，該公司的內部部署系統和基礎結構所承受的壓力變大了。
- **提高效率**：Contoso 必須移除不必要的程序，並簡化其開發人員和使用者的程序。 該公司需要快速且不浪費時間或金錢的 IT 服務，以便能夠更快滿足客戶需求。
- **提高靈活性**：Contoso IT 必須能夠更快因應企業的需求。 其因應速度必須能夠比市場變化更快，才能讓該公司在全球經濟中獲致成功。 Contoso 的 IT 小組不得礙事，或成為企業的絆腳石。
- **調整**：隨著該公司的業務順利成長，Contoso IT 必須提供能夠同步成長的系統。

## <a name="assessment-goals"></a>評量目標

Contoso 雲端小組已識別其移轉評量的目標：

- 移轉過後，應用程式不管是在 Azure 或 Contoso 的內部部署 VMWare 環境中，都應具有相同的效能。 移到雲端，並不表示應用程式效能比較不重要。
- Contoso 必須了解其應用程式和資料庫與 Azure 需求的相容性。 Contoso 也必須了解其在 Azure 中的裝載選項。
- 在應用程式移到雲端之後，Contoso 的資料庫管理工作應會減到最少。  
- Contoso 不僅想要了解其移轉選項，還想了解在移到雲端之後與基礎結構相關的成本。

## <a name="assessment-tools"></a>評量工具

Contoso 會使用 Microsoft 工具進行其移轉評量。 這些工具與該公司的目標相符，應可提供 Contoso 所需的全部資訊。

Technology | 說明 | 成本
--- | --- | ---
[資料移轉小幫手](https://docs.microsoft.com/sql/dma/dma-overview?view=ssdt-18vs2017) | Contoso 會使用 Data Migration Assistant 來評定和偵測可能影響其在 Azure 中資料庫功能的相容性問題。 Data Migration Assistant 會評定 SQL 來源和目標之間的功能同位。 其會建議如何改善效能和可靠性。 | Data Migration Assistant 是免費、可下載的工具。
[Azure Migrate](https://docs.microsoft.com/azure/migrate/migrate-overview) | Contoso 會使用 Azure Migrate 服務來評定其 VMware VM。 Azure Migrate 會評定機器是否適合移轉。 它會提供在 Azure 中執行的大小調整建議和成本估計。  | 截至 2018 年 5 月，Azure Migrate 是一項免費服務。
[服務對應](https://docs.microsoft.com/azure/operations-management-suite/operations-management-suite-service-map) | Azure Migrate 會使用服務對應來顯示該公司想要遷移的機器彼此之間的相依性。 | 服務對應是 Azure Log Analytics 的一部分。 目前，Contoso 可免費使用服務對應 180 天。

在此案例中，Contoso 會下載並執行 Data Migration Assistant，以評定其旅遊應用程式的內部部署 SQL Server 資料庫。 Contoso 會使用 Azure Migrate 搭配相依性對應來評定應用程式 VM，再將其遷移至 Azure。

## <a name="assessment-architecture"></a>評量架構

![移轉評估架構](./media/contoso-migration-assessment/migration-assessment-architecture.png)

- Contoso 是代表典型企業組織的虛構名稱。
- Contoso 有內部部署資料中心 (**contoso-datacenter**) 以及內部部署網域控制站 (**CONTOSODC1**、**CONTOSODC2**)。
- VMware VM 位於執行 6.5 版的 VMware ESXI 主機 (**contosohost1**、**contosohost2**) 上。
- VMware 環境是由 VM 上執行的 vCenter Server 6.5 (**vcenter.contoso.com**) 進行管理。
- SmartHotel360 旅遊應用程式具有下列特性：
    - 這個應用程式橫跨兩層 VMware VM (**WEBVM** 和 **SQLVM**)。
    - VM 位於 VMware ESXi 主機 **contosohost1.contoso.com**。
    - VM 正在執行 Windows Server 2008 R2 Datacenter with SP1。
- VMware 環境是由 VM 上所執行的 VCenter Server (**vcenter.contoso.com**) 進行管理。
- osTicket 服務台應用程式：
    - 這個應用程式橫跨兩層 VM (**OSTICKETWEB** 和 **OSTICKETMYSQL**)。
    - VM 正在 Ubuntu Linux Server 16.04-LTS 上執行。
    - **OSTICKETWEB** 正在執行 Apache 2 和 PHP 7.0。
    - **OSTICKETMYSQL** 正在執行 MySQL 5.7.22。

## <a name="prerequisites"></a>必要條件

Contoso 和其他使用者都必須符合下列先決條件才能進行評量：

- 對於 Azure 訂用帳戶或 Azure 訂用帳戶中資源群組的擁有者或參與者權限。
- 執行 6.5、6.0 或 5.5 版的內部部署 vCenter 伺服器執行個體。
- vCenter Server 中的唯讀帳戶，或是可供建立帳戶的權限。
- 可在 vCenter Server 執行個體上使用 .ova 範本建立 VM 的權限。
- 至少一個執行 5.0 版或更新版本的 ESXi 主機。
- 至少兩個內部部署 VMware VM，且其中一個要執行 SQL Server 資料庫。
- 可在每部 VM 上安裝 Azure Migrate 代理程式的權限。
- VM 應該要能直接連線到網際網路。  
        - 您可以限制只能透過網際網路存取[所需的 URL](https://docs.microsoft.com/azure/migrate/concepts-collector#collector-pre-requisites)。  
        - 如果您的 VM 沒有網際網路連線，則 VM 上必須安裝 Azure [Log Analytics 閘道](../log-analytics/log-analytics-oms-gateway.md)，而且代理程式流量會透過該閘道導向。
- 執行 SQL Server 執行個體的 VM 所具有的 FQDN，以供評估資料庫。
- SQL Server VM 上所執行的 Windows 防火牆應該在 TCP 通訊埠 1433 (預設值) 允許外部連線。 此設定可讓 Data Migration Assistant 能夠連線。

## <a name="assessment-overview"></a>評量概觀

以下是 Contoso 執行評量的方式：

> [!div class="checklist"]
> * **步驟 1：下載並安裝 Data Migration Assistant**：Contoso 會準備 Data Migration Assistant 以便評量內部部署 SQL Server 資料庫。
> * **步驟 2：使用 Data Migration Assistant 評定資料庫**：Contoso 會執行及分析資料庫評量。
> * **步驟 3：使用 Azure Migrate 準備 VM 評量**：Contoso 會設定內部部署帳戶並調整 VMware 設定。
> * **步驟 4：使用 Azure Migrate 探索內部部署 VM**：Contoso 會建立 Azure Migrate 收集器 VM。 然後，Contoso 會執行收集器以探索要評量的 VM。
> * **步驟 5：使用 Azure Migrate 準備相依性分析**：Contoso 會在 VM 上安裝 Azure Migrate 代理程式，以便能夠了解 VM 之間的相依性對應。
> * **步驟 6：使用 Azure Migrate 評定 VM**：Contoso 會檢查相依性、將 VM 分組，並執行評量。 當評量就緒時，Contoso 會分析評量以準備進行移轉。

## <a name="step-1-download-and-install-data-migration-assistant"></a>步驟 1：下載並安裝 Data Migration Assistant

1. Contoso 可從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=53595)下載 Data Migration Assistant。
    - Data Migration Assistant 可以安裝於任何可以連線到 SQL Server 執行個體的機器上。 Contoso 不需要在 SQL Server 機器上執行此工具。
    - Data Migration Assistant 不得在 SQL Server 主機機器上執行。
2. Contoso 會執行所下載的安裝檔案 (DownloadMigrationAssistant.msi)，開始進行安裝。
3. 在 [完成] 頁面上，Contoso 會先選取 [啟動 Microsoft Data Migration Assistant]，再完成精靈。

## <a name="step-2-run-and-analyze-the-database-assessment-for-smarthotel360"></a>步驟 2：執行 SmartHotel360 的資料庫評估並加以分析

Contoso 現在可以執行評估，以便針對 SmartHotel360 應用程式分析其內部部署 SQL Server 資料庫。

1. 在 Data Migration Assistant 中，Contoso 會選取 [新增] > [評估]，然後為評估提供專案名稱。
2. Contoso 會選取 [Azure 虛擬機器上的 SQL Server] 作為 [來源伺服器類型]。

    ![Data Migration Assistant - 選取來源](./media/contoso-migration-assessment/dma-assessment-1.png)

    > [!NOTE]
      Data Migration Assistant 目前不支援就是否可遷移至 Azure SQL Database 受控執行個體進行評量。 為解決此問題，Contoso 會使用「Azure VM 上的 SQL Server」作為假定的評量目標。

3. 在 [選取目標版本] 中，Contoso 會選取 SQL Server 2017 作為目標版本。 這是 SQL Database 受控執行個體所使用的版本，所以 Contoso 必須選取此版本。
4. Contoso 選取報告以協助自身探索相容性和新功能的相關資訊：
    - [相容性問題] 會指明可能會中斷移轉的變更，或指明必須略做調整再移轉。 此報告會讓 Contoso 得知任何目前使用、但已過時的功能。 所有問題會以相容性層級來加以組織。
    - [新功能的建議] 會指明資料庫可於移轉後所使用的目標 SQL Server 平台中，有什麼新功能。 新功能的建議會組織在 [效能]、[安全性] 和 [儲存體] 標題底下。

    ![Data Migration Assistant - 相容性問題和新功能](./media/contoso-migration-assessment/dma-assessment-2.png)

2. 在 [連線到伺服器] 中，Contoso 會輸入執行資料庫的 VM 名稱，以及用來存取它的認證。 Contoso 會啟用 [信任伺服器憑證] 以確保 VM 可以存取 SQL Server。 接著，Contoso 會選取 [連線]。

    ![Data Migration Assistant - 連線至伺服器](./media/contoso-migration-assessment/dma-assessment-3.png)

3. 在 [新增來源] 中，Contoso 會新增想要評定的資料庫，然後選取 [下一步] 開始進行評量。
4. 評量隨即建立。

    ![Data Migration Assistant - 建立評量](./media/contoso-migration-assessment/dma-assessment-4.png)

5. 在 [檢閱結果] 中，Contoso 會檢視評量結果。

### <a name="analyze-the-database-assessment"></a>分析資料庫評估結果

一有結果就會顯示出來。 如果 Contoso 將問題修復，則必須選取 [重新開始評量] 以重新執行評量。

1. 在 [相容性問題] 報告中，Contoso 會檢查每個相容性層級的所有問題。 相容性層級與 SQL Server 版本的對應如下：

    - 100：SQL Server 2008/Azure SQL Database
    - 110：SQL Server 2012/Azure SQL Database
    - 120：SQL Server 2014/Azure SQL Database
    - 130：SQL Server 2016/Azure SQL Database
    - 140：SQL Server 2017/Azure SQL Database

    ![Data Migration Assistant - 相容性問題報告](./media/contoso-migration-assessment/dma-assessment-5.png)

2. 在 [功能建議] 報告中，Contoso 會檢視評量在移轉後建議的效能、安全性和儲存功能。 該報告會提出各種建議功能，包括記憶體內部 OLTP、資料行存放區索引、Stretch Database、Always Encrypted、動態資料遮罩和透明資料加密。

    ![Data Migration Assistant - 功能建議報告](./media/contoso-migration-assessment/dma-assessment-6.png)

    > [!NOTE]
    > Contoso 應針對所有 SQL Server 資料庫[啟用透明資料加密](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption?view=sql-server-2017)。 相較於裝載在內部部署環境時，當資料庫位於雲端時，這一點會更加重要。 請只在移轉後才啟用透明資料加密。 如果透明資料加密已啟用，Contoso 就必須將憑證或非對稱金鑰移至目標伺服器的 master 資料庫。 了解如何[將受到透明資料加密保護的資料庫移到其他 SQL Server 執行個體](https://docs.microsoft.com/sql/relational-databases/security/encryption/move-a-tde-protected-database-to-another-sql-server?view=sql-server-2017)。

2. Contoso 可以匯出 JSON 或 CSV 格式的評量。

> [!NOTE]
> 如需進行大規模的評量：
> - 同時執行多個評量，並在 [所有評量] 頁面檢視這些評量的狀態。
> - 將評量合併至 [SQL Server 資料庫](https://docs.microsoft.com/sql/dma/dma-consolidatereports?view=ssdt-18vs2017#import-assessment-results-into-a-sql-server-database)。
> - 將評量合併至 [PowerBI 報告](https://docs.microsoft.com/sql/dma/dma-powerbiassesreport?view=ssdt-18vs2017)。

## <a name="step-3-prepare-for-vm-assessment-by-using-azure-migrate"></a>步驟 3：使用 Azure Migrate 準備 VM 評量

Contoso 必須建立 VMware 帳戶，以供 Azure Migrate 用來自動探索要評量的 VM、確認建立 VM 的權限、記下必須開啟的連接埠，以及設定統計資料設定層級。

### <a name="set-up-a-vmware-account"></a>設定 VMware 帳戶

VM 探索需要在 vCenter Server 中擁有唯讀帳戶，且該帳戶要具有下列屬性：

- **使用者類型**：至少是唯讀使用者。
- **權限**：針對資料中心物件，選取 [傳播至子物件] 核取方塊。 針對 [角色]，選取 [唯讀]。
- **詳細資料**：使用者是在資料中心層級指派的，具有資料中心內所有物件的存取權。
- 如果要限制存取權，請將具備 [傳播至子物件] 權限的 [沒有存取權] 角色指派給子物件 (vSphere 主機、資料存放區、VM 及網路)。

### <a name="verify-permissions-to-create-a-vm"></a>確認用來建立 VM 的權限

Contoso 會確認其有權限可藉由匯入 .ova 格式的檔案來建立 VM。 了解如何[建立及指派具有權限的角色](https://kb.vmware.com/s/article/1023189?other.KM_Utility.getArticleLanguage=1&r=2&other.KM_Utility.getArticleData=1&other.KM_Utility.getArticle=1&ui-comm-runtime-components-aura-components-siteforce-qb.Quarterback.validateRoute=1&other.KM_Utility.getGUser=1)。

### <a name="verify-ports"></a>確認連接埠

Contoso 評量會使用相依性對應。 若要進行相依性對應，所要評定的 VM 上必須安裝代理程式。 該代理程式必須能從每個 VM 上的 TCP 連接埠 443 連線到 Azure。 了解[連線需求](https://docs.microsoft.com/azure/log-analytics/log-analytics-concept-hybrid)。

### <a name="set-statistics-settings"></a>設定統計資料設定

Contoso 在開始部署之前，必須將 vCenter Server 的統計資料設定設為層級 3。 

> [!NOTE]
> - 設定層級之後，Contoso 必須至少等待一天才能執行評量。 否則，評量可能不會如預期般運作。
> - 如果層級高於 3，評估仍會運作，但：
>    - 不會收集磁碟和網路的效能資料。
>    - 針對儲存，Azure Migrate 的建議是在 Azure 中使用標準磁碟，且大小要與內部部署磁碟相同。
>    - 針對網路功能，每個內部部署網路介面卡都建議在 Azure 中有一個對應的網路介面卡。
>    - 針對計算，Azure Migrate 會查看 VM 核心和記憶體大小，並建議使用有相同設定的 Azure VM。 如果有多個符合資格的 Azure VM 大小，建議使用成本最低的那個。
> - 如需使用層級 3 來調整大小的詳細資訊，請參閱[調整大小](https://docs.microsoft.com/azure/migrate/concepts-assessment-calculation#sizing)。

若要設定層級：

1. 在 vSphere Web 用戶端中，Contoso 會開啟 SQL Server 執行個體。
2. Contoso 會選取 [管理] > [設定] > [一般] > [編輯]。
3. 在 [統計資料] 中，Contoso 會將統計資料層級的設定設為 [層級 3]。

    ![vCenter Server 統計資料層級](./media/contoso-migration-assessment/vcenter-statistics-level.png)

## <a name="step-4-discover-vms"></a>步驟 4：探索 VM

為了探索 VM，Contoso 會建立一個 Azure Migrate 專案。 Contoso 會下載並設定收集器 VM。 然後，Contoso 會執行收集器以探索其內部部署 VM。

### <a name="create-a-project"></a>建立專案

1. 在 [Azure 入口網站](https://portal.azure.com)中，Contoso 會搜尋 **Azure Migrate**。 接著，Contoso 會建立專案。
2. Contoso 會指定專案名稱 (**ContosoMigration**) 和 Azure 訂用帳戶。 它會建立新的 Azure 資源群組 (**ContosoFailoverRG**)。 
    > [!NOTE]
    > - 您只能在「美國中西部」或「美國東部」區域建立 Azure Migrate 專案。
    > - 您可以針對任何目標位置規劃移轉。
    > - 專案位置只會用來儲存從內部部署 VM 收集到的中繼資料。

    ![Azure Migrate - 建立移轉專案](./media/contoso-migration-assessment/project-1.png)

### <a name="download-the-collector-appliance"></a>下載收集器設備

Azure Migrate 會建立稱為「收集器設備」的內部部署 VM。 此 VM 會探索內部部署 VMware VM，並將 VM 的相關中繼資料傳送至 Azure Migrate 服務。 為了設定收集器設備，Contoso 會下載 OVA 範本，然後將其匯入至內部部署 vCenter Server 執行個體來建立 VM。

1. 在 Azure Migrate 專案中，Contoso 會選取 [開始使用] > [探索及評定] > [探索機器]。 Contoso 會下載 OVA 範本檔案。
2. Contoso 會複製專案識別碼與金鑰。 必須有專案和識別碼才能設定收集器。

    ![Azure Migrate - 下載 OVA 檔案](./media/contoso-migration-assessment/download-ova.png)

### <a name="verify-the-collector-appliance"></a>確認收集器設備

在部署 VM 之前，Contoso 會確認 OVA 檔案是安全的：

1. 在存放所下載檔案的機器上，Contoso 會開啟系統管理員命令提示字元視窗。
2. Contoso 會執行下列命令以產生 OVA 檔案的雜湊：

    ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    
    **範例** 
    
    ```C:\>CertUtil -HashFile C:\AzureMigrate\AzureMigrate.ova SHA256```
3. 產生的雜湊應符合這些設定 (1.0.9.14 版)：

    **演算法** | **雜湊值**
    --- | ---
    MD5 | 6d8446c0eeba3de3ecc9bc3713f9c8bd
    SHA1 | e9f5bdfdd1a746c11910ed917511b5d91b9f939f
    SHA256 | 7f7636d0959379502dfbda19b8e3f47f3a4744ee9453fc9ce548e6682a66f13c

### <a name="create-the-collector-appliance"></a>建立收集器設備

現在，Contoso 可以將下載的檔案匯入 vCenter Server 執行個體並佈建收集器設備 VM：

1. 在 vSphere 用戶端主控台中，Contoso 會選取 [檔案] > [部署 OVF 範本]。

    ![vSphere Web 用戶端 - 部署 OVF 範本](./media/contoso-migration-assessment/vcenter-wizard.png)

2. 在 [部署 OVF 範本精靈] 中，Contoso 會選取 [來源]，然後指定 OVA 檔案的位置。
3. 在 [名稱和位置] 中，Contoso 會指定收集器 VM 的顯示名稱。 然後，它會選取用來裝載 VM 的清查位置。 Contoso 也會指定用來執行收集器設備的主機或叢集。
4. 在 [儲存體] 中，Contoso 會指定儲存位置。 在 [磁碟格式] 中，Contoso 會選取儲存體的佈建方式。
5. 在 [網路對應] 中，Contoso 會指定要用來連接收集器 VM 的網路。 此網路必須能夠連線到網際網路，以將中繼資料傳送至 Azure。
6. Contoso 會檢閱設定，然後選取 [在部署後開啟電源] > [完成]。 當設備建立好，就會出現訊息確認作業已成功完成。

### <a name="run-the-collector-to-discover-vms"></a>執行收集器來探索 VM

現在，Contoso 會執行收集器以探索 VM。 收集器目前只支援以**英文 (美國)** 作為作業系統語言和收集器介面語言。

1. 在 vSphere 用戶端主控台中，Contoso 會選取 [開啟主控台]。 Contoso 會指定收集器 VM 的語言、時區和密碼喜好設定。
2. 在桌面上，Contoso 會選取 [執行收集器] 捷徑。

    ![vSphere 用戶端主控台 - 收集器捷徑](./media/contoso-migration-assessment/collector-shortcut.png)

3. 在 Azure Migrate 收集器中，Contoso 會選取 [設定必要條件]。 Contoso 會接受授權條款，並閱讀第三方資訊。
4. 收集器會確認 VM 是否可存取網際網路、時間是否同步，以及收集器服務是否正在執行。 (收集器服務已依預設安裝在 VM 上)。Contoso 也會安裝 VMware PowerCLI。

    > [!NOTE]
    > 我們會假設 VM 可直接存取網際網路，而不會使用 Proxy。

    ![Azure Migrate 收集器 - 確認必要條件](./media/contoso-migration-assessment/collector-verify-prereqs.png)

5. 在 [指定 vCenter Server 詳細資料] 中，Contoso 會輸入 vCenter Server 執行個體的名稱 (FQDN) 或 IP 位址，以及用於探索的唯讀認證。
6. Contoso 會選取 VM 探索的範圍。 收集器只能探索指定範圍內的虛擬機器。 範圍可以設定為特定資料夾、資料中心或叢集。 範圍不應包含超過 1,500 個 VM。

    ![指定 vCenter Server 詳細資料](./media/contoso-migration-assessment/collector-connect-vcenter.png)

7. 在 [指定移轉專案] 中，Contoso 會輸入從入口網站所複製的 Azure Migrate 專案識別碼和金鑰。 為了取得專案識別碼和金鑰，Contoso 可以移至專案的 [概觀] 頁面 > [探索機器]。  

    ![指定移轉專案](./media/contoso-migration-assessment/collector-connect-azure.png)

8. 在 [檢視收集進度] 中，Contoso 可以監視探索，然後確認從虛擬機器收集到的中繼資料位於範圍內。 收集器會提供概略的探索時間。

    ![檢視收集進度](./media/contoso-migration-assessment/collector-collection-process.png)

### <a name="verify-vms-in-the-portal"></a>在入口網站中確認 VM

收集完成時，Contoso 會確認 VM 出現在入口網站中：

1. 在 Azure Migrate 專案中，Contoso 會選取 [管理] > [機器]。 Contoso 會檢查其想要探索的 VM 是否已顯示。

    ![Azure Migrate - 探索到的機器](./media/contoso-migration-assessment/discovery-complete.png)

2. 機器目前未安裝 Azure Migrate 代理程式。 Contoso 必須安裝代理程式才能檢視相依性。

    ![Azure Migrate - 必須安裝代理程式](./media/contoso-migration-assessment/machines-no-agent.png)

## <a name="step-5-prepare-for-dependency-analysis"></a>步驟 5︰準備相依性分析

Contoso 為了檢視其所要評定 VM 之間的相依性，會在應用程式 VM 上下載並安裝代理程式。 Contoso 會在其應用程式的所有 VM (Windows 和 Linux) 上安裝代理程式。

### <a name="take-a-snapshot"></a>擷取快照集

為了在修改 VM 前保留其複本，Contoso 會在安裝代理程式之前建立快照集。

![機器的快照集](./media/contoso-migration-assessment/snapshot-vm.png)

### <a name="download-and-install-the-vm-agents"></a>下載並安裝虛擬機器代理程式

1. 在 [機器] 中，Contoso 會選取機器。 在 [相依性] 資料行中，Contoso 會選取 [需要安裝]。
2. 在 [探索機器] 窗格中，Contoso 會：
    - 下載每部 Windows VM 的 Microsoft Monitoring Agent (MMA) 和 Dependency Agent。
    - 下載每部 Linux VM 的 MMA 和 Dependency Agent。
3. Contoso 會複製工作區識別碼和金鑰。 Contoso 在安裝 MMA 時，需要工作區識別碼和金鑰。

    ![代理程式下載](./media/contoso-migration-assessment/download-agents.png)

### <a name="install-the-agents-on-windows-vms"></a>在 Windows VM 上安裝代理程式

Contoso 會在每部 VM 上執行安裝。

#### <a name="install-the-mma-on-windows-vms"></a>在 Windows VM 上安裝 MMA

1. Contoso 會按兩下所下載的代理程式。
2. 在 [目的資料夾] 中，Contoso 會保留預設的安裝資料夾，然後選取 [下一步]。
3. 在 [代理程式安裝選項] 中，Contoso 會選取 [將代理程式連線至 Azure Log Analytics] > [下一步]。

    ![Microsoft Monitoring Agent 安裝程式 - 代理程式安裝程式選項](./media/contoso-migration-assessment/mma-install.png)

4. 在 [Azure Log Analytics] 中，Contoso 會貼上從入口網站複製的工作區識別碼和金鑰。

    ![Microsoft Monitoring Agent 安裝程式 - Azure Log Analytics](./media/contoso-migration-assessment/mma-install2.png)

5. 在 [準備安裝] 中，Contoso 會安裝 MMA。

#### <a name="install-the-dependency-agent-on-windows-vms"></a>在 Windows VM 上安裝 Dependency Agent

1. Contoso 會按兩下所下載的 Dependency Agent。
2. Contoso 會接受授權條款並等候安裝完成。

    ![Dependency Agent 安裝程式 - 安裝中](./media/contoso-migration-assessment/dependency-agent.png)

### <a name="install-the-agents-on-linux-vms"></a>在 Linux VM 上安裝代理程式

Contoso 會在每部 VM 上執行安裝。

#### <a name="install-the-mma-on-linux-vms"></a>在 Linux VM 上安裝 MMA

1. Contoso 會使用下列命令，在每個 VM 上安裝 Python ctypes 程式庫：

    `sudo apt-get install python-ctypeslib`
2. Contoso 必須執行此命令，以 root 身分安裝 MMA 代理程式。 為了變成 root 身分，Contoso 會執行下列命令，然後輸入 root 密碼：

    `sudo -i`
3. Contoso 會安裝 MMA：
    - Contoso 會在命令中輸入工作區識別碼和金鑰。
    - 命令是針對 64 位元。
    - 工作區識別碼和主要金鑰位於 Microsoft Operations Management Suite (OMS) 入口網站。 依序選取 [設定] 和 [連接的來源] 索引標籤。
    - 執行下列命令來下載 OMS 代理程式、驗證總和檢查碼，然後安裝代理程式並使其上線：

    ```
    wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w 6b7fcaff-7efb-4356-ae06-516cacf5e25d -s k7gAMAw5Bk8pFVUTZKmk2lG4eUciswzWfYLDTxGcD8pcyc4oT8c6ZRgsMy3MmsQSHuSOcmBUsCjoRiG2x9A8Mg==
    ```

#### <a name="install-the-dependency-agent-on-linux-vms"></a>在 Linux VM 上安裝 Dependency Agent

安裝 MMA 之後，Contoso 會在 Linux VM 上安裝 Dependency Agent：

1. 使用 InstallDependencyAgent-Linux64.bin (具有自我解壓縮二進位檔的殼層指令碼)，即可在 Linux 電腦上安裝 Dependency Agent。 Contoso 會使用 sh 來執行檔案，或對檔案本身新增執行權限。

2. Contoso 會以 root 身分安裝 Linux Dependency Agent：

    ```
    wget --content-disposition https://aka.ms/dependencyagentlinux -O InstallDependencyAgent-Linux64.bin && sudo sh InstallDependencyAgent-Linux64.bin -s
    ```

## <a name="step-6-run-and-analyze-the-vm-assessment"></a>步驟 6：執行 VM 評量並加以分析

Contoso 現在可以確認機器相依性並建立群組。 接著，會執行群組的評量。

### <a name="verify-dependencies-and-create-a-group"></a>確認相依性和建立群組

1. 為了判斷所要分析的機器，Contoso 會選取 [檢視相依性]。

    ![Azure Migrate - 檢視機器相依性](./media/contoso-migration-assessment/view-machine-dependencies.png)

2. SQLVM 的相依性對應顯示下列詳細資料：

    - 在指定的時間期間 (預設為一小時) 內，有作用中網路連線在 SQLVM 上執行的處理序群組或處理序。
    - 所有相依機器的輸入 (用戶端) 和輸出 (伺服器) TCP 連線。
    - 已安裝 Azure Migrate 代理程式的相依機器會顯示為不同的方塊。
    - 未安裝代理程式的機器則會顯示連接埠和 IP 位址資訊。

3. 針對已安裝代理程式的機器 (WEBVM)，Contoso 會選取機器方塊以檢視詳細資訊。 這些資訊包括 FQDN、作業系統和 MAC 位址。

    ![Azure Migrate - 檢視群組相依性](./media/contoso-migration-assessment/sqlvm-dependencies.png)

4. Contoso 會選取要新增至群組的 VM (SQLVM 和 WEBVM)。 Contoso 會使用「CTRL+按一下滑鼠左鍵」來選取多個 VM。
5. Contoso 會選取 [建立群組]，然後輸入名稱 (**smarthotelapp**)。

    > [!NOTE]
    > 若要檢視更細微的相依性，您可以展開時間範圍。 您可以選取特定持續期間，也可以選取開始和結束日期。

### <a name="run-an-assessment"></a>執行評估

1. 在 [群組] 中，Contoso 會開啟群組 (**smarthotelapp**)，然後選取 [建立評量]。

    ![Azure Migrate - 建立評估](./media/contoso-migration-assessment/run-vm-assessment.png)

2. 為了檢視評量，Contoso 會選取 [管理] > [評量]。

Contoso 會使用預設評量設定，但您可以[自訂設定](how-to-modify-assessment.md)。

### <a name="analyze-the-vm-assessment"></a>分析 VM 評估

Azure Migrate 評量包括內部部署與 Azure 的相容性、建議的 Azure VM 適當縮放，以及預估的 Azure 每月成本等相關資訊。

![Azure Migrate - 評量報告](./media/contoso-migration-assessment/assessment-overview.png)

#### <a name="review-confidence-rating"></a>檢閱信賴評等

![Azure Migrate - 評量顯示](./media/contoso-migration-assessment/assessment-display.png)

評量會獲得 1 顆星到 5 顆星的信賴評等 (1 顆星最低，5 顆星最高)。
- 根據計算評量所需的資料點可用性，每個評量都會指派信賴評等。
- 此評等可協助您預估 Azure Migrate 所提供大小建議的可靠性。
- 信賴評等在您進行「以效能為基礎的大小調整」時非常實用。 Azure Migrate 可能沒有足夠的資料點可供進行以使用率為基礎的大小調整。 對於「內部部署形式」的大小調整，信賴評等一律是 5 顆星，原因是 Azure Migrate 擁有調整 VM 大小所需的所有資料點。
- 根據可用資料點的百分比提供評量的信賴評等：

   資料點的可用性 | 信賴評等
   --- | ---
   0%-20% | 1 顆星
   21%-40% | 2 顆星
   41%-60% | 3 顆星
   61%-80% | 4 顆星
   81%-100% | 5 顆星

#### <a name="verify-azure-readiness"></a>確認 Azure 移轉整備程度

![Azure Migrate - 評量整備程度](./media/contoso-migration-assessment/azure-readiness.png)  

評估報告會顯示彙總在資料表中的資訊。 為了顯示以效能為基礎的大小調整，Azure Migrate 需要下列資訊。 如果無法收集此資訊，大小調整評估可能不會正確。

- CPU 和記憶體的使用量資料。
- 連結至 VM 的每個磁碟，其所具有的讀取/寫入 IOPS 及輸送量。
- 連結至 VM 的每個網路介面卡，其所具有的網路輸入/輸出資訊。

設定 | 指示 | 詳細資料
--- | --- | ---
**Azure VM 整備程度** | 指出 VM 是否已可移轉。 | 可能的狀態：</br><br/>- 可供 Azure 使用<br/><br/>- 有條件的備妥 <br/><br/>- 未備妥供 Azure 使用<br/><br/>- 整備程度未知<br/><br/> 如果 VM 未準備好，Azure Migrate 會示範一些補救步驟。
**Azure VM 大小** | 對於準備好的 VM，Azure Migrate 會提供 Azure VM 大小建議。 | 大小調整建議取決於評估屬性：<br/><br/>- 如果您之前使用以效能為基礎的大小調整，大小調整會考慮 VM 的效能記錄。<br/><br/>- 如果您之前使用「內部部署形式」，則大小調整會以內部部署 VM 的大小為基礎，而不會使用使用量資料。
**建議的工具** | 因為 Azure 機器會執行代理程式，Azure Migrate 會查看在機器內部執行的程序。 它會識別機器是否為資料庫機器。
**VM 資訊** | 此報告會顯示內部部署 VM 的設定，包括作業系統、開機類型、磁碟和儲存體資訊。

#### <a name="review-monthly-cost-estimates"></a>檢閱每月成本預估值

此檢視會顯示在 Azure 中執行 VM 的計算和儲存總成本。 它也會顯示每部機器的詳細資料。

![Azure Migrate -  Azure 成本](./media/contoso-migration-assessment/azure-costs.png)

- 會使用機器的大小建議來計算成本預估值。
- 系統會彙總群組內所有 VM 之計算和儲存的每月預估成本。

## <a name="clean-up-after-assessment"></a>評量之後進行清除

- 評量完成時，Contoso 會保留 Azure Migrate 設備以供未來評估使用。
- Contoso 會關閉 VMware VM。 Contoso 會在評估其他 VM 時再次使用它。
- Contoso 會在 Azure 中保留 **Contoso Migration** 專案。 該專案目前部署在東部美國 Azure 區域的 **ContosoFailoverRG** 資源群組中。
-  收集器 VM 有 180 天的評估授權。 如果超過此限制，Contoso 就需要再次下載並設定收集器。

## <a name="conclusion"></a>結論

在此案例中，Contoso 會使用「資料移轉評估」工具來評估其 SmartHotel360 應用程式的資料庫。 它會藉由使用 Azure Migrate 服務來評定內部部署 VM。 Contoso 會檢閱評量，以確保內部部署資源已可供移轉至 Azure。

## <a name="next-steps"></a>後續步驟

在這一系列的下一篇文章中，Contoso 會在 Azure 中使用隨即轉移來重新裝載其 SmartHotel360 應用程式。 Contoso 會藉由使用 Azure Site Recovery 來遷移應用程式的前端 WEBVM。 它會藉由使用「資料庫移轉服務」，將應用程式資料庫遷移至 Azure SQL Database 受控執行個體。 [開始使用](contoso-migration-rehost-vm-sql-managed-instance.md)此部署。
