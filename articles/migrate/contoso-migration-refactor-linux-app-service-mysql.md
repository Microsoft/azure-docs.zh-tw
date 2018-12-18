---
title: 將 Contoso Linux 服務台應用程式重構至 Azure App Service 和 Azure MySQL | Microsoft Docs
description: 了解 Contoso 如何使用適用於 Web 層和 Azure SQL Database 的 GitHub，將內部部署 Linux 應用程式遷移至 Azure App Service，以便重構。
author: rayne-wiselman
ms.service: site-recovery
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: raynew
ms.openlocfilehash: 3f85d9d18aa49a378c63fa1f1692c7dc665489be
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44296513"
---
# <a name="contoso-migration-refactor-a-contoso-linux-service-desk-app-to-multiple-regions-with-azure-app-service-traffic-manager-and-azure-mysql"></a>Contoso 移轉：將 Contoso Linux 服務台應用程式重構至具有 Azure App Service、流量管理員和 Azure MySQL 的多個區域

本文說明 Contoso 如何將其內部部署兩層式 Linux 服務台應用程式 (osTicket) 遷移到含 GitHub 整合的 Azure App Service 和 Azure MySQL，以便重構。

這份文件是系列文章中的其中一篇，說明 Contoso 這家虛構的公司如何將其內部部署資源遷移至 Microsoft Azure 雲端。 整個系列文章中包含背景資訊以及一些案例，用以說明如何設定移轉基礎結構，以及執行不同類型的移轉。 案例的複雜性會提高。 我們會不定期新增其他文章。

**文章** | **詳細資料** | **狀態**
--- | --- | ---
[文章 1：概觀](contoso-migration-overview.md) | 系列文章的概觀、Contoso 的移轉策略，以及此系列中所使用的應用程式範例。 | 可用
[文章 2：部署 Azure 基礎結構](contoso-migration-infrastructure.md) | Contoso 會準備其內部部署基礎結構和其 Azure 基礎結構以進行移轉。 此系列中的所有移轉文章都使用相同的基礎結構。 | 可用
[文章 3：存取內部部署資源以移轉至 Azure](contoso-migration-assessment.md)  | Contoso 會評估其在 VMware 上執行的內部部署 SmartHotel360 應用程式。 Contoso 會使用 Azure Migrate 服務來評估應用程式 VM，並使用 Database Migration Assistant 來評估應用程式 SQL Server 資料庫。 | 可用
[文章 4：在 Azure VM 和 SQL Database 受控執行個體上重新裝載應用程式](contoso-migration-rehost-vm-sql-managed-instance.md) | Contoso 會為其內部部署 SmartHotel360 應用程式執行隨即移轉至 Azure 的作業。 Contoso 會使用 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview) 來移轉應用程式的前端 VM。 Contoso 會使用 [Azure 資料庫移轉服務](https://docs.microsoft.com/azure/dms/dms-overview)將應用程式資料庫移轉至 Azure SQL Database 受控執行個體。 | 可用   
[文章 5：在 Azure VM 上重新裝載應用程式](contoso-migration-rehost-vm.md) | Contoso 會使用 Site Recovery 服務，將其 SmartHotel360 應用程式 VM 移轉至 Azure VM。 | 可用
[文章 6：在 Azure VM 上和 SQL Server AlwaysOn 可用性群組中重新裝載應用程式](contoso-migration-rehost-vm-sql-ag.md) | Contoso 會移轉 SmartHotel360 應用程式。 Contoso 會使用 Site Recovery 來遷移應用程式的 VM。 它會使用資料庫移轉服務，將應用程式資料庫遷移至受到 SQL Server 可用性群組保護的 SQL Server 叢集。 | 可用 
[文章 7：將 Linux 應用程式重新裝載至 Azure VM](contoso-migration-rehost-linux-vm.md) | Contoso 會使用 Site Recovery 服務，來完成將其 Linux osTicket 應用程式隨即移轉至 Azure VM 的作業 | 可用
[文章 8：在 Azure VM 和 Azure MySQL 上重新裝載 Linux 應用程式](contoso-migration-rehost-linux-vm-mysql.md) | Contoso 會使用 Azure Site Recovery 將 Linux osTicket 應用程式移轉至 Azure VM，並使用 MySQL Workbench 將應用程式資料庫移轉至 Azure MySQL Server 執行個體。 | 可用
[文章 9：在 Azure Web 應用程式和 Azure SQL 資料庫上重構應用程式](contoso-migration-refactor-web-app-sql.md) | Contoso 會將 SmartHotel360 應用程式移轉至 Azure Web 應用程式，以及使用 Database Migration Assistant 將應用程式資料庫移轉至 Azure SQL Server 執行個體 | 可用
文章 10：在 Azure Web Apps 和 Azure MySQL 上重構 Linux 應用程式 | Contoso 會使用 Azure 流量管理員，將 Linux osTicket 應用程式移轉至多個 Azure 區域的 Azure Web 應用程式，與 GitHub 整合以進行持續傳遞。 Contoso 會將應用程式資料庫遷移至適用於 MySQL 的 Azure 資料庫執行個體。 | 本文
[文章 11：重構 Azure DevOps Services 上的 TFS](contoso-migration-tfs-vsts.md) | Contoso 會將其內部部署 Team Foundation Server 部署，移轉至 Azure 中的 Azure DevOps Services。 | 可用
[文章 12：在 Azure 容器和 Azure SQL Database 上重新建構應用程式](contoso-migration-rearchitect-container-sql.md) | Contoso 會將其 SmartHotel 應用程式移轉至 Azure。 然後，它會重新建構應用程式 Web 層，作為在 Azure Service Fabric 中執行的 Windows 容器，以及具有 Azure SQL Database 的資料庫。 | 可用
[文章 13：在 Azure 中重建應用程式](contoso-migration-rebuild.md) | Contoso 會藉由使用各種 Azure 功能和服務 (包括 Azure App Service、Azure Kubernetes Service (AKS)、Azure Functions、Azure 認知服務及 Azure Cosmos DB) 重建其 SmartHotel360 應用程式。 | 可用

在本文中，Contoso 會將兩層式 Linux Apache MySQL PHP (LAMP) 服務台應用程式 (osTicket) 遷移至 Azure。 如果想使用這個開放原始碼的應用程式，您可以從 [github](https://github.com/osTicket/osTicket) 進行下載。


## <a name="business-drivers"></a>商業動機

IT 領導小組與商務合作夥伴密切合作，了解他們想要達成什麼目標：

- **解決業務成長**：Contoso 正在成長並轉向新的市場。 需要額外的客戶服務代理程式。 
- **擴展**：應該建置解決方案，好讓 Contoso 隨著商務擴展而新增更多客戶服務代理程式。
- **提升復原能力**：過去的系統問題只會影響內部使用者。 使用新的商業模式，外部使用者會受到影響，而 Contoso 需要應用程式隨時保持在運作狀態。

## <a name="migration-goals"></a>移轉目標

Contoso 雲端小組已針對此次移轉擬定好各項目標，以便決定最佳移轉方法：

- 應用程式應該擴展超過目前的內部部署容量和效能。  Contoso 正在移動應用程式，以充分利用 Azure 的隨選縮放。
- Contoso 希望能將應用程式的程式碼基底移至持續傳遞管線，  當應用程式變更推送至 GitHub 時，Contoso 希望操作人員不需要進行任何工作，便可部署這些變更。
- 應用程式必須先具備成長與容錯移轉的能力，才算是具備復原性。 Contoso 想要在兩個不同的 Azure 區域中部署應用程式，並設定為自動調整。
- 應用程式移到雲端後，Contoso 希望能將資料庫管理的工作減到最少。

## <a name="solution-design"></a>解決方案設計
擬定好目標和需求後，Contoso 會設計和檢閱部署解決方案，並識別移轉程序，包括將用於移轉的 Azure 服務。


## <a name="current-architecture"></a>目前架構

- 這個應用程式橫跨兩層 VM (OSTICKETWEB 和 OSTICKETMYSQL)。
- VM 位於 VMware ESXi 主機 **contosohost1.contoso.com** (6.5 版)。
- VMware 環境是由 VM 上執行的 vCenter Server 6.5 (**vcenter.contoso.com**) 進行管理。
- Contoso 有內部部署資料中心 (contoso-datacenter) 以及內部部署網域控制站 (**contosodc1**)。

![目前架構](./media/contoso-migration-refactor-linux-app-service-mysql/current-architecture.png) 


## <a name="proposed-architecture"></a>建議的架構

以下是建議的架構：

- 在兩個 Azure 區域中建置 Azure App Service，並藉此遷移 OSTICKETWEB 上的 Web 層應用程式。 將會使用 PHP 7.0 Docker 容器來實作適用於 Linux 的 Azure App Service。
- 應用程式程式碼將會移至 GitHub，而且會設定 Azure Web 應用程式以便透過 GitHub 持續傳遞。
- 主要 (美國東部 2) 和次要 (美國中部) 區域都會部署 Azure App Server。
- 在這兩個區域中，流量管理員將會設定於兩個 Azure Web Apps 前面。
- 若要強制流量通過美國東部 2，請以優先順序模式設定流量管理員。
- 如果美國東部 2 的 Azure App Server 離線，使用者可以存取美國中部已容錯移轉的應用程式。
- 應用程式資料庫將會使用 MySQL Workbench 工具遷移到 Azure MySQL PaaS 服務。 內部部署資料庫將在本機進行備份，並直接還原到 Azure MySQL。
- 此資料庫將位於主要美國東部 2 區域，置於生產網路 (VNET-PROD-EUS2) 的資料庫子網路 (PROD-DB-EUS2) 中。
- 因為他們要遷移生產工作負載，所以應用程式的 Azure 資源會位於生產資源群組 **ContosoRG** 中。
- 流量管理員資源會部署在 Contoso 的基礎結構資源群組 **ContosoInfraRG** 中。
- 移轉完成之後，將會解除委任 Contoso 資料中心的內部部署 VM。


![案例架構](./media/contoso-migration-refactor-linux-app-service-mysql/proposed-architecture.png) 


## <a name="migration-process"></a>移轉程序

Contoso 會按照下列方式完成移轉程序：

1. 在第一個步驟中，Contoso 管理員會設定 Azure 基礎結構，包括佈建 Azure App Service、設定流量管理員，以及佈建 Azure MySQL 執行個體。
2. 準備好 Azure 之後，他們會使用 MySQL Workbench 來遷移資料庫。 
3. 在 Azure 中執行資料庫之後，他們會設定 GitHub 私人存放庫以便 Azure App Service 持續傳遞，並使用 osTicket 應用程式將它載入。
4. 在 Azure 入口網站中，它們會將應用程式從 GitHub 載入至執行 Azure App Service 的 Docker 容器。 
5. 它們會調校 DNS 設定，並且為應用程式設定自動調整。

![移轉程序](./media/contoso-migration-refactor-linux-app-service-mysql/migration-process.png) 


### <a name="azure-services"></a>Azure 服務

**服務** | **說明** | **成本**
--- | --- | ---
[Azure App Service](https://azure.microsoft.com/services/app-service/) | 服務會使用適用於網站的 Azure PaaS 服務，執行和調整應用程式。  | 定價是以執行個體的大小和所需的功能為基礎。 [深入了解](https://azure.microsoft.com/pricing/details/app-service/windows/)。
[流量管理員](https://azure.microsoft.com/services/traffic-manager/) | 使用 DNS 將使用者導向 Azure 或外部網站和服務的負載平衡器。 | 定價是以收到的 DNS 查詢數目和已監視的端點數目為基礎。 | [深入了解](https://azure.microsoft.com/pricing/details/traffic-manager/)。
[適用於 MySQL 的 Azure 資料庫](https://docs.microsoft.com/azure/mysql/) | 此資料庫是以開放原始碼 MySQL 伺服器引擎為基礎。 它可為應用程式的開發與部署，提供完全受控、符合企業需求的社群 MySQL 資料庫即服務。 | 定價是以計算、儲存和備份需求為基礎。 [深入了解](https://azure.microsoft.com/pricing/details/mysql/)。

 
## <a name="prerequisites"></a>必要條件

以下是 Contoso 要執行此案例所需的項目。

**需求** | **詳細資料**
--- | ---
**Azure 訂用帳戶** | Contoso 稍早已在本文章系列中建立訂用帳戶。 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。<br/><br/> 如果您建立免費帳戶，您就是訂用帳戶的管理員，並可執行所有動作。<br/><br/> 如果您使用現有訂用帳戶，而且您不是系統管理員，則需要與系統管理員合作，讓其指派擁有者或參與者權限給您。 
**Azure 基礎結構** | Contoso 會如[適用於移轉的 Azure 基礎結構](contoso-migration-infrastructure.md)中所述，設定其 Azure 基礎結構。



## <a name="scenario-steps"></a>案例步驟

以下是 Contoso 完成移轉的方式：

> [!div class="checklist"]
> * **步驟 1：佈建 Azure App Service**：Contoso 管理員會在主要和次要區域中佈建 Web Apps。
> * **步驟 2：設定流量管理員**：他們會在 Web Apps 前面設定流量管理員，以便路由傳送及平衡流量負載。
> * **步驟 3：佈建 MySQL**：在 Azure 中，他們會佈建 Azure MySQL 資料庫執行個體。
> * **步驟 4：遷移資料庫**：他們會使用 MySQL Workbench 來遷移資料庫。 
> * **步驟 5︰設定 GitHub**：他們會設定應用程式網站/程式碼的本機 GitHub 存放庫。
> * **步驟 6：部署 Web 應用程式**：他們會從 GitHub 部署 Web 應用程式。




## <a name="step-1-provision-azure-app-services"></a>步驟 1：佈建 Azure App Services

Contoso 管理員會使用 Azure App Service 佈建兩個 Web 應用程式 (每個區域一個)。

1. 他們會從 Azure Marketplace，在主要美國東部 2 區域 (**osticket-eus2**) 建立 Web 應用程式資源。
2. 他們會將資源放在生產資源群組 **ContosoRG** 中。

    ![Azure 應用程式](./media/contoso-migration-refactor-linux-app-service-mysql/azure-app1.png) 

3. 他們會使用標準大小，在主要區域 (**APP-SVP-EUS2**) 中建立新的 App Service 方案。

     ![Azure 應用程式](./media/contoso-migration-refactor-linux-app-service-mysql/azure-app2.png) 
    
4. 他們會選取具有 PHP 7.0 執行階段堆疊的 Linux OS，這是一個 Docker 容器。

    ![Azure 應用程式](./media/contoso-migration-refactor-linux-app-service-mysql/azure-app3.png) 

5. 他們會為美國中部區域建立第二個 Web 應用程式 (**osticket-cus**) 和應用程式服務方案。

    ![Azure 應用程式](./media/contoso-migration-refactor-linux-app-service-mysql/azure-app4.png) 


**需要其他協助？**

- 了解 [Azure App Service Web 應用程式](https://docs.microsoft.com/azure/app-service/app-service-web-overview)。
- 了解 [Linux 上的 Azure App Service](https://docs.microsoft.com/azure/app-service/containers/app-service-linux-intro)。


## <a name="step-2-set-up-traffic-manager"></a>步驟 2：設定流量管理員

Contoso 管理員會設定流量管理員，將輸入的 Web 要求導向至在 osTicket Web 層上執行的 Web Apps。

1. 他們會從 Azure Marketplace 建立流量管理員資源 (**osticket.trafficmanager.net**)。 他們採用優先順序路由，因此美國東部 2 是主要網站。 他們將此資源放在其基礎結構資源群組 (**ContosoInfraRG**).中。 請注意，流量管理員為全域服務，無法繫結至特定位置。

    ![流量管理員](./media/contoso-migration-refactor-linux-app-service-mysql/traffic-manager1.png) 

2. 現在，他們會使用端點來設定流量管理員。 他們會新增美國東部 2 Web 應用程式作為主要網站 (**osticket-eus2**)，以及新增美國中部應用程式作為次要網站 (**osticket-cus**)。

    ![流量管理員](./media/contoso-migration-refactor-linux-app-service-mysql/traffic-manager2.png) 

3. 新增端點之後，它們可以進行監視。

    ![流量管理員](./media/contoso-migration-refactor-linux-app-service-mysql/traffic-manager3.png)

**需要其他協助？**

- 了解[流量管理員](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-overview)。
- 了解如何[將流量路由傳送至優先端點](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-configure-priority-routing-method)。
 
## <a name="step-3-provision-azure-database-for-mysql"></a>步驟 3：佈建適用於 MySQL 的 Azure 資料庫

Contoso 管理員會在主要美國東部 2 區域中，佈建 MySQL 資料庫執行個體。

1. 在 Azure 入口網站中，他們可會建立適用於 MySQL 的 Azure 資料庫資源。 

    ![MySQL](./media/contoso-migration-refactor-linux-app-service-mysql/mysql-1.png)

2. 他們會為 Azure 資料庫新增名稱 **contosoosticket**。 他們會將資料庫新增至生產資源群組 **ContosoRG**，然後為它指定認證。
3. 內部部署 MySQL 資料庫的版本為 5.7，因此他們會為了相容性而選取這個版本。 他們會使用預設大小，以符合其資料庫需求。

     ![MySQL](./media/contoso-migration-refactor-linux-app-service-mysql/mysql-2.png)

4. 針對 [備份備援選項]，他們會選擇使用 [異地備援]。 此選項可讓他們在發生中斷時，在其次要美國中部區域中還原資料庫。 他們在佈建資料庫時，只能設定這個選項。

    ![備援性](./media/contoso-migration-refactor-linux-app-service-mysql/db-redundancy.png)

4. 他們會設定連線安全性。 在資料庫 > [連線安全性] 中，他們會設定防火牆規則，允許資料庫存取 Azure 服務。
5. 他們會將本機工作站用戶端 IP 位址新增到開始和結束 IP 位址。 這可讓 Web 應用程式存取 MySQL 資料庫，以及存取執行移轉的資料庫用戶端。

    ![MySQL](./media/contoso-migration-refactor-linux-app-service-mysql/mysql-3.png)



## <a name="step-4-migrate-the-database"></a>步驟 4：遷移資料庫

Contoso 管理員會利用 MySQL 工具，使用備份與還原來遷移資料庫。 他們會安裝 MySQL Workbench，從 OSTICKETMYSQL 備份資料庫，然後將它還原至 適用於 MySQL 的 Azure 資料庫伺服器。

### <a name="install-mysql-workbench"></a>安裝 MySQL Workbench

1. 他們會檢查[必要條件以及下載 MySQL Workbench](https://dev.mysql.com/downloads/workbench/?utm_source=tuicool) (英文)。
2. 他們會按照[安裝指示](https://dev.mysql.com/doc/workbench/en/wb-installing.html)，安裝適用於 Windows 的 MySQL Workbench。 OSTICKETMYSQL VM 和 Azure 必須能透過網際網路存取安裝所在的電腦。
3. 在 MySQL Workbench 中，他們會建立 OSTICKETMYSQL 的 MySQL 連線。 

    ![MySQL Workbench](./media/contoso-migration-refactor-linux-app-service-mysql/workbench1.png)

4. 他們會將資料庫當作 **osticket** 匯出至本機獨立檔案。

    ![MySQL Workbench](./media/contoso-migration-refactor-linux-app-service-mysql/workbench2.png)

5. 在本機備份資料庫之後，他們會建立適用於 MySQL 的 Azure 資料庫執行個體連線。

    ![MySQL Workbench](./media/contoso-migration-refactor-linux-app-service-mysql/workbench3.png)

6. 他們現在可以在 Azure MySQL 執行個體中，從該獨立檔案匯入 (還原) 資料庫。 系統會為此執行個體建立新的結構描述 (osticket)。

    ![MySQL Workbench](./media/contoso-migration-refactor-linux-app-service-mysql/workbench4.png)

7. 還原資料之後，可以使用 Workbench 來查詢資料，而該資料會出現在 Azure 入口網站中。

    ![MySQL Workbench](./media/contoso-migration-refactor-linux-app-service-mysql/workbench5.png)

    ![MySQL Workbench](./media/contoso-migration-refactor-linux-app-service-mysql/workbench6.png)

8. 最後，他們必須更新 Web 應用程式上的資料庫資訊。 在 MySQL 執行個體上，他們會開啟**連接字串**。 

     ![MySQL Workbench](./media/contoso-migration-refactor-linux-app-service-mysql/workbench7.png)

9. 在字串清單中，他們會找出 Web 應用程式設定，並且按一下加以複製。

    ![MySQL Workbench](./media/contoso-migration-refactor-linux-app-service-mysql/workbench8.png)

10. 他們會開啟 [記事本] 視窗並將字串貼入新檔案，然後加以更新它，以符合 osticket 資料庫、MySQL 執行個體和認證設定。

     ![MySQL Workbench](./media/contoso-migration-refactor-linux-app-service-mysql/workbench9.png)

11. 他們可以在 Azure 入口網站的 MySQL 執行個體中，確認 [概觀] 中的伺服器名稱和登入。

    ![MySQL Workbench](./media/contoso-migration-refactor-linux-app-service-mysql/workbench10.png)


## <a name="step-5-set-up-github"></a>步驟 5︰設定 GitHub

Contoso 管理員會建立新的私人 GitHub 存放庫，並設定連到 Azure MySQL 中 osTicket 資料庫的連線。 接著，它們會隨著應用程式載入 Azure Web 應用程式。  

1.  他們會瀏覽至 OsTicket 軟體公用 GitHub 存放庫，並將它分支處理至 Contoso GitHub 帳戶。

    ![GitHub](./media/contoso-migration-refactor-linux-app-service-mysql/github1.png)

2. 分支處理之後，他們會導覽至 **include** 資料夾，然後尋找 **ost-config.php** 檔案。

    ![GitHub](./media/contoso-migration-refactor-linux-app-service-mysql/github2.png)


3. 檔案會在瀏覽器中開啟，他們會加以編輯。

    ![GitHub](./media/contoso-migration-refactor-linux-app-service-mysql/github3.png)

4. 他們會在編輯器中更新資料庫詳細資料，具體來說是 **DBHOST** 和 **DBUSER**。 

    ![GitHub](./media/contoso-migration-refactor-linux-app-service-mysql/github4.png)

5. 他們會接著認可變更。

    ![GitHub](./media/contoso-migration-refactor-linux-app-service-mysql/github5.png)

6. 針對每個 Web 應用程式 (**osticket-eus2** 和 **osticket-cus**)，他們會在 Azure 入口網站中修改 [應用程式設定]。

    ![GitHub](./media/contoso-migration-refactor-linux-app-service-mysql/github6.png)

7. 他們會輸入名稱為 **osticket** 的連接字串，並將此字串從 [記事本] 複製到**值區域**中。 他們會選取字串旁邊下拉式清單中的 **MySQL**，並儲存設定。

    ![GitHub](./media/contoso-migration-refactor-linux-app-service-mysql/github7.png)

## <a name="step-6-configure-the-web-apps"></a>步驟 6: 設定 Web Apps

在移轉程序的最後一個步驟中，Contoso 管理員會使用 osTicket 網站來設定 Web 應用程式。



1. 在主要 Web 應用程式 (**osticket-eus2**) 中，他們會開啟 [部署選項] 並將來源設定為 **GitHub**。

    ![設定應用程式](./media/contoso-migration-refactor-linux-app-service-mysql/configure-app1.png)

2. 他們會選取部署選項。

    ![設定應用程式](./media/contoso-migration-refactor-linux-app-service-mysql/configure-app2.png)

3. 設定好選項之後，Azure 入口網站中的組態會顯示為擱置。

    ![設定應用程式](./media/contoso-migration-refactor-linux-app-service-mysql/configure-app3.png)

4. 更新組態而且 osTicket Web 應用程式從 GitHub 載入至執行 Azure App Service 的 Docket 容器之後，網站就會顯示為作用中。

    ![設定應用程式](./media/contoso-migration-refactor-linux-app-service-mysql/configure-app4.png)

5. 他們會針對第二個 Web 應用程式 (**osticket-cus**)，重複上述步驟。
6. 設定好網站之後，即可透過流量管理員設定檔進行存取。 DNS 名稱是 osTicket 應用程式的新位置。 [深入了解](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)。

    ![設定應用程式](./media/contoso-migration-refactor-linux-app-service-mysql/configure-app5.png)
    
7. Contoso 想要有好記的 DNS 名稱。 他們會在其網域控制站的 DNS 中，建立可指向流量管理員名稱的別名記錄 (CNAME) **osticket.contoso.com**。

    ![設定應用程式](./media/contoso-migration-refactor-linux-app-service-mysql/configure-app6.png)

8. 他們會設定 **osticket-eus2** 和 **osticket-cus** Web 應用程式，以允許自訂主機名稱。

    ![設定應用程式](./media/contoso-migration-refactor-linux-app-service-mysql/configure-app7.png)

### <a name="set-up-autoscaling"></a>設定自動調整

最後，他們會設定應用程式的自動調整。 這可確保當代理程式使用應用程式時，應用程式執行個體會根據業務需求增加和減少。 

1. 在 App Service **APP-SRV-EUS2** 中，他們會開啟 [縮放單位]。
2. 他們以單一規則設定新的自動調整設定，以便在目前執行個體的 CPU 百分比超過 70% 達到 10 分鐘時，將執行個體計數遞增一。

    ![Autoscale](./media/contoso-migration-refactor-linux-app-service-mysql/autoscale1.png)

3. 他們會在 **APP-SRV-CUS** 上設定相同的設定，確保應用程式容錯移轉到次要區域時會套用相同的行為。 唯一的差異在於他們將執行個體限制設定為 1，因為這僅適用於容錯移轉。

   ![Autoscale](./media/contoso-migration-refactor-linux-app-service-mysql/autoscale2.png)

##  <a name="clean-up-after-migration"></a>移轉之後進行清除

移轉完成後，osTicket 應用程式會重構，以使用私人 GitHub 存放庫透過持續傳遞方式在 Azure Web 應用程式中執行。 應用程式會在兩個區域中執行，以提升復原能力。 在遷移到 PaaS 平台之後，osTicket 資料庫會在適用於 MySQL 的 Azure 資料庫中執行。

若要清除，Contoso 必須執行下列作業： 
- 從 vCenter 清查中移除 VMware VM。
- 從本機備份作業中移除內部部署 VM。
- 更新內部文件以顯示新的位置和 IP 位址。 
- 檢閱與內部部署 VM 互動的任何資源，並更新任何相關的設定或文件，以反映新的組態。
- 將監視重新設定為指向 osticket-trafficmanager.net UR，以追蹤應用程式正在運作中。

## <a name="review-the-deployment"></a>檢閱部署

應用程式正在執行中，Contoso 必須能發揮一切功能並保護其新的基礎結構。

### <a name="security"></a>安全性

Contoso 安全性小組會檢查 Azure VM，判斷是否有任何的安全疑慮。 他們發現 osTicket 應用程式與 MySQL 資料庫執行個體之間的通訊並未針對 SSL 進行設定。 他們必須這麼做，才能確保資料庫流量不會遭到駭客入侵。 [深入了解](https://docs.microsoft.com/azure/mysql/howto-configure-ssl)。

### <a name="backups"></a>備份

- osTicket Web 應用程式不包含狀態資料，因此不需要進行備份。
- 他們不需要設定資料庫的備份。 適用於 MySQL 的 Azure 資料庫會自動建立伺服器備份和存放區。 他們選擇對資料庫使用異地備援，所以資料庫可復原並已準備好用於生產。 備份可以用來將伺服器還原至某個時間點。 [深入了解](https://docs.microsoft.com/azure/mysql/concepts-backup)。


### <a name="licensing-and-cost-optimization"></a>授權和成本最佳化

- PaaS 部署沒有任何授權問題。
- Contoso 會啟用 Microsoft 子公司 Cloudyn 授權的 Azure 成本管理。 它是一種多雲端成本管理解決方案，可協助您利用和管理 Azure 和其他雲端資源。  [深入了解](https://docs.microsoft.com/azure/cost-management/overview) Azure 成本管理。



