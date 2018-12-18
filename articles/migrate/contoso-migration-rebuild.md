---
title: 在 Azure 中重建 Contoso 內部部署應用程式 | Microsoft Docs
description: 了解 Contoso 如何使用 Azure App Service、Kubernetes 服務、CosmosDB、Azure Functions 和認知服務在 Azure 中重建應用程式。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: conceptual
ms.date: 09/20/2018
ms.author: raynew
ms.openlocfilehash: f0dc199f8a91ac06993f4ccbc9dff7dfad9f8a19
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47042477"
---
# <a name="contoso-migration-rebuild-an-on-premises-app-to-azure"></a>Contoso 移轉：在 Azure 中重建內部部署應用程式

此文章示範 Contoso 如何在 Azure 中移轉及重建 SmartHotel360 應用程式。 Contoso 會將應用程式的前端 VM 移轉至 Azure App Services Web 應用程式。 應用程式後端的建置，則是使用 Azure Kubernetes Service (AKS) 所管理容器中部署的微服務來進行。 網站會與 Azure Functions 互動以提供寵物相片功能。 

這份文件是系列文章中的其中一篇，說明 Contoso 這家虛構的公司如何將內部部署資源移轉至 Microsoft Azure 雲端。 整個系列文章中包含背景資訊和說明如何設定移轉基礎結構的案例、評估內部部署資源的移轉，以及執行不同類型的移轉。 案例的複雜性會提高。 我們會不定期新增其他文章。


**文章** | **詳細資料** | **狀態**
--- | --- | ---
[文章 1：概觀](contoso-migration-overview.md) | 簡單介紹 Contoso 的移轉策略、文章系列以及我們使用的範例應用程式。 | 可用
[文章 2：部署 Azure 基礎結構](contoso-migration-infrastructure.md) | 說明 Contoso 如何準備其內部部署和 Azure 基礎結構以進行移轉。 所有移轉文章都使用相同的基礎結構。 | 可用
[文章 3：評估內部部署資源](contoso-migration-assessment.md)  | 顯示 Contoso 如何評估在 VMware 上執行的內部部署兩層式 SmartHotel360 應用程式。 Contoso 利用 [Azure Migrate](migrate-overview.md) 服務來評估應用程式 VM，並利用 [Database Migration Assistant](https://docs.microsoft.com/sql/dma/dma-overview?view=sql-server-2017) 來評估應用程式 SQL Server 資料庫。 | 可用
[文章 4：在 Azure VM 和 SQL 受控執行個體上重新裝載應用程式](contoso-migration-rehost-vm-sql-managed-instance.md) | 說明 Contoso 如何為 SmartHotel360 應用程式執行隨即移轉至 Azure 的作業。 Contoso 使用 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview) 來移轉應用程式前端 VM，並使用 [Azure 資料庫移轉服務](https://docs.microsoft.com/azure/dms/dms-overview)將應用程式資料庫移轉至 SQL 受控執行個體。 | 可用
[文章 5：在 Azure VM 上重新裝載應用程式](contoso-migration-rehost-vm.md) | 說明 Contoso 如何單純憑藉 Site Recovery 來移轉 SmartHotel360 應用程式 VM。 | 可用
[文章 6：將應用程式重新裝載至 Azure VM 和 SQL Server AlwaysOn 可用性群組](contoso-migration-rehost-vm-sql-ag.md) | 說明 Contoso 如何移轉 SmartHotel360 應用程式。 Contoso 使用 Site Recovery 來移轉應用程式 VM，並使用資料庫移轉服務，將應用程式資料庫移轉至受到 SQL Server 可用性群組保護的 SQL Server 叢集。 | 可用
[文章 7：將 Linux 應用程式重新裝載至 Azure VM](contoso-migration-rehost-linux-vm.md) | 說明 Contoso 如何使用 Site Recovery 執行將 Linux osTicket 應用程式隨即移轉至 Azure VM 的作業 | 可用
[文章 8：在 Azure VM 和 Azure MySQL Server 上重新裝載 Linux 應用程式](contoso-migration-rehost-linux-vm-mysql.md) | 說明 Contoso 如何使用 Site Recovery 將 Linux osTicket 應用程式移轉至 Azure VM，並使用 MySQL Workbench 將應用程式資料庫移轉至 Azure MySQL Server 執行個體。 | 可用
[文章 9：在 Azure Web 應用程式和 Azure SQL 資料庫上重構應用程式](contoso-migration-refactor-web-app-sql.md) | 說明 Contoso 如何將 SmartHotel360 應用程式移轉至 Azure Web 應用程式，以及如何將應用程式資料庫移轉至 Azure SQL Server 執行個體 | 可用
[文章 10：將 Linux 應用程式重構到 Azure Web Apps 和 Azure MySQL](contoso-migration-refactor-linux-app-service-mysql.md) | 示範 Contoso 如何將 Linux osTicket 應用程式移轉到多個網站中的 Azure Web Apps (與 GitHub 整合以達到持續傳遞)。 他們會將應用程式資料庫移轉到 Azure MySQL 執行個體。 | 可用
[文章 11：在 Azure DevOps Services 上重構 TFS](contoso-migration-tfs-vsts.md) | 說明 Contoso 如何透過將內部部署 Team Foundation Server (TFS) 部署移轉至 Azure 中的 Azure DevOps Services 來對 TFS 進行移轉。 | 可用
[文章 12：在 Azure 容器和 SQL Database 上重新架構應用程式](contoso-migration-rearchitect-container-sql.md) | 示範 Contoso 如何將其 SmartHotel 應用程式移轉和重新架構至 Azure。 它們會將應用程式的 Web 層重新建構為 Windows 容器及 Azure SQL Database 中的應用程式資料庫。 | 可用
文章 13：在 Azure 中重建應用程式 | 示範 Contoso 如何使用各種 Azure 功能和服務 (包括 App Services、Azure Kubernetes、Azure Functions、認知服務及 Cosmos DB) 重建其 SmartHotel 應用程式。 | 此文章。

在此文章中，Contoso 會移轉兩層式 Windows。 NET SmartHotel360 應用程式 (執行於 VMware VM)，移轉至 Azure。 如果您想使用此應用程式，您可以從 [GitHub](https://github.com/Microsoft/SmartHotel360) 下載其開放原始碼。

## <a name="business-drivers"></a>商業動機

IT 領導小組與商務合作夥伴密切合作，以了解此次移轉所要實現的目標：

- **因應業務成長**：Contoso 正在成長，而想要為 Contoso 網站上的客戶提供差異化的體驗。
- **靈活度**：Contoso 的因應速度必須能夠比市場變化更快，才能更在全球經濟中獲致成功。 
- **規模調整**：隨著企業順利成長，Contoso IT 小組必須提供能夠同步成長的系統。
- **成本**：Contoso 想要將授權成本降至最低。

## <a name="migration-goals"></a>移轉目標

Contoso 雲端小組已針對此次移轉擬定好各項應用程式需求。 這些需求已用來判斷最合適的移轉方法：
 - Azure 中的應用程式應與現在一樣重要。 其應該妥善執行並可輕鬆調整。
 - 應用程式不應使用 IaaS 元件。 所有項目皆應建置為使用 PaaS 或無伺服器服務。
 - 應用程式組建應該在雲端服務中執行，且容器應該位於雲端中的全企業私人容器登錄中。
 - 其旅館必須接受應用程式所做的決策，所以用於寵物相片的 API 服務在真實世界中應該要準確可靠。 任何獲准進入的寵物都可以留在旅館內。
 - 為了滿足 DevOps 管線的需求，Contoso 會使用 Azure DevOps 搭配 Git 存放庫來進行「原始程式碼管理」(SCM)。  他們將使用自動化組建和發行來建置程式碼，並將其部署至 Azure Web Apps、Azure Functions 及 AKS。
 - 後端上的微服務與前端上的網站需要不同的 CI/CD 管線。
 - 後端服務的發行週期與前端 Web 應用程式不同。  為了滿足此需求，他們將部署兩個不同的 DevOps 管線。
 - Contoso 需要管理部門核准才能進行所有前端網站部署，因此 CI/CD 管線必須提供此功能。


## <a name="solution-design"></a>解決方案設計

擬定好目標和需求之後，Contoso 會設計和檢閱部署解決方案，並識別移轉程序，包括將用於移轉的 Azure 服務。

### <a name="current-app"></a>目前的應用程式

- SmartHotel360 內部部署應用程式會分層至兩個 VM (WEBVM 和 SQLVM)。
- 這些 VM 位於 VMware ESXi 主機 **contosohost1.contoso.com** (6.5 版)
- VMware 環境是由 VM 上執行的 vCenter Server 6.5 (**vcenter.contoso.com**) 進行管理。
- Contoso 有內部部署資料中心 (contoso-datacenter) 以及內部部署網域控制站 (**contosodc1**)。
- 移轉完成之後，將會解除委任 Contoso 資料中心的內部部署 VM。


### <a name="proposed-architecture"></a>建議的架構

- 應用程式的前端在主要 Azure 區域中部署為 Azure App Services Web 應用程式。
- Azure 函式提供寵物相片上傳功能，而網站則與此功能互動。
- 寵物相片函式會利用認知服務視覺 API 和 CosmosDB。
- 網站的後端會使用微服務來建置。 這些微服務會部署到 Azure Kubernetes Service (AKS) 上所管理的容器。
- 建置容器時將使用 Azure DevOps 來建置，並推送至 Azure Container Registry (ACR)。
- 目前，Contoso 會使用 Visual Studio 來手動部署 Web 應用程式和函式程式碼
- 微服務會使用 PowerShell 指令碼，呼叫 Kubernetes 命令列工具來進行部署。

    ![案例架構](./media/contoso-migration-rebuild/architecture.png) 

  
### <a name="solution-review"></a>解決方案檢閱

Contoso 會透過比較一份優缺點清單，來評估建議設計。

**考量** | **詳細資料**
--- | ---
**優點** | 使用 PaaS 和無伺服器解決方案來進行端對端部署可大幅減少 Contoso 必須提供的管理時間。<br/><br/> 移轉至微服務架構可讓 Contoso 隨著時間的進展輕鬆延伸解決方案。<br/><br/> 在將新功能上線時，不必中斷任何現有的解決方案程式碼基底。<br/><br/> Web 應用程式會設定為配有多個執行個體，所以不會有單一失敗點。<br/><br/> 會啟用自動調整功能，讓應用程式能夠處理不同的流量。<br/><br/> 移往 PaaS 服務後，Contoso 可以淘汰在 Windows Server 2008 R2 作業系統上執行的過時解決方案。<br/><br/> CosmosDB 有內建容錯能力，Contoso 不必進行設定。 這表示資料層不再是單一的容錯移轉點。
**缺點** | 容器比其他移轉選項複雜得多。 其學習曲線可能會是 Contoso 的難題。  儘管有學習曲線的問題，但容器帶來了新的複雜度等級，而可提供許多價值。<br/><br/> Contoso 的營運團隊必須提升能力，以針對應用程式來了解和支援 Azure、容器及微服務。<br/><br/> Contoso 尚未針對整個解決方案完全實作 DevOps。 Contoso 必須在將服務部署至 AKS、函式及 App Services 時，考慮到這一點。



### <a name="migration-process"></a>移轉程序

1. Contoso 會佈建 ACR、AKS 和 CosmosDB。
2. 其會佈建基礎結構以供部署使用，包括 Azure Web 應用程式、儲存體帳戶、函式和 API。 
3. 在備妥基礎結構之後，他們將使用 Azure DevOps 來建置其微服務容器映像，以將映像推送至 ACR。
4. Contoso 會使用 PowerShell 指令碼將這些微服務部署至 ASK。
5. 最後，他們會部署 Azure 函式和 Web 應用程式。

    ![移轉程序](./media/contoso-migration-rebuild/migration-process.png) 

### <a name="azure-services"></a>Azure 服務

**服務** | **說明** | **成本**
--- | --- | ---
[AKS](https://docs.microsoft.com/sql/dma/dma-overview?view=ssdt-18vs2017) | 簡化 Kubernetes 管理、部署和作業。 提供完全受控的 Kubernetes 容器協調流程服務。  | AKS 是免費服務。  只需就取用的虛擬機器以及相關聯的儲存體和網路資源支付費用。 [深入了解](https://azure.microsoft.com/pricing/details/kubernetes-service/)。
[Azure Functions](https://azure.microsoft.com/services/functions/) | 以事件驅動的無伺服器計算體驗，加快開發速度。 依需求進行調整。  | 只需就取用的資源支付費用。 根據每秒的資源取用量和執行次數計算方案的費用。 [深入了解](https://azure.microsoft.com/pricing/details/functions/)。
[Azure Container Registry](https://azure.microsoft.com/services/container-registry/) | 儲存所有容器部署類型的映像。 | 根據功能、儲存體和使用期間計算費用。 [深入了解](https://azure.microsoft.com/pricing/details/container-registry/)。
[Azure App Service](https://azure.microsoft.com/services/app-service/containers/) | 快速建置、部署和調整在任何平台上執行的企業級 Web、行動裝置和 API 應用程式。 | App Service 方案以每秒計費。 [深入了解](https://azure.microsoft.com/pricing/details/app-service/windows/)。

## <a name="prerequisites"></a>必要條件

以下是 Contoso 針對此案例所需的項目：

**需求** | **詳細資料**
--- | ---
**Azure 訂用帳戶** | Contoso 在先前文章期間已建立訂用帳戶。 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。<br/><br/> 如果您建立免費帳戶，您就是訂用帳戶的管理員，並可執行所有動作。<br/><br/> 如果您使用現有訂用帳戶，而且您不是系統管理員，則需要與系統管理員合作，讓其指派擁有者或參與者權限給您。
**Azure 基礎結構** | [了解](contoso-migration-infrastructure.md) Contoso 如何設定 Azure 基礎結構。
**開發人員必要條件** | 在開發人員工作站上，Contoso 需要下列工具：<br/><br/> - [Visual Studio 2017 Community 版本：15.5 版](https://www.visualstudio.com/)<br/><br/> 已啟用 .NET 工作負載。<br/><br/> [Git](https://git-scm.com/)<br/><br/> [Azure PowerShell](https://azure.microsoft.com/downloads/)<br/><br/> [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)<br/><br/> 已設定為使用 Windows 容器的 [Docker CE (Windows 10) 或 Docker EE (Windows Server)](https://docs.docker.com/docker-for-windows/install/)。



## <a name="scenario-steps"></a>案例步驟

以下是 Contoso 執行移轉的方式：

> [!div class="checklist"]
> * **步驟 1：佈建 AKS 和 ACR**：Contoso 會使用 PowerShell 佈建受控 AKS 叢集和 Azure 容器登錄
> * **步驟 2：建置 Docker 容器**：他們會使用 Azure DevOps 來設定 Docker 容器的 CI，然後將其推送至 ACR。
> * **步驟 3：部署後端微服務**：他們會部署基礎結構的其餘部分，以供後端微服務運用。
> * **步驟 4：部署前端基礎結構**：他們會部署前端基礎結構，包括寵物電話的 Blob 儲存體、Cosmos DB 和視覺 API。
> * **步驟 5：移轉後端**：他們會部署微服務並在 AKS 上執行，以移轉後端。
> * **步驟 6：發佈前端**：他們會將 SmartHotel360 應用程式發佈至 Azure App Service 以及寵物服務所會呼叫的函式應用程式。



## <a name="step-1-provision-back-end-resources"></a>步驟 1：佈建後端資源

Contoso 管理員會執行部署指令碼，使用 AKS 和 Azure Container Registry (ACR) 來建立受控 Kubernetes 叢集。

- 本節的指示會使用 **SmartHotel360-Azure-backend** 存放庫。
- **SmartHotel360-Azure-backend** GitHub 存放庫包含這部分部署的所有軟體。

### <a name="prerequisites"></a>必要條件

1. 在開始之前，Contoso 管理員會確保在他們要用於部署的開發機器上已安裝所有先決條件軟體。
2. 他們會使用 Git 將本機存放庫複製到開發機器：**git clone https://github.com/Microsoft/SmartHotel360-Azure-backend.git**


### <a name="provision-aks-and-acr"></a>佈建 AKS 和 ACR

Contoso 管理員會依下列方式進行佈建：

1.  他們會使用 Visual Studio Code 來開啟資料夾，然後移至 **/deploy/k8s** 目錄，其中包含指令碼 **gen-aks-env.ps1**。
2. 他們會執行指令碼，使用 AKS 和 ACR 來建立受控 Kubernetes 叢集。

    ![AKS](./media/contoso-migration-rebuild/aks1.png)
 
3.  在檔案開啟時，他們會將 $location 參數更新為 **eastus2**，並儲存檔案。

    ![AKS](./media/contoso-migration-rebuild/aks2.png)

4. 他們會按一下 [檢視] > [整合式終端機] 以在 Code 中開啟整合式終端機。

    ![AKS](./media/contoso-migration-rebuild/aks3.png)

5. 在 PowerShell 整合式終端機中，他們會使用 Connect-AzureRmAccount 命令登入 Azure。 [深入了解](https://docs.microsoft.com/powershell/azure/get-started-azureps)如何開始使用 PowerShell。

    ![AKS](./media/contoso-migration-rebuild/aks4.png)

6. 他們會執行 **az login** 命令來驗證 Azure CLI，並遵循指示以使用其網頁瀏覽器進行驗證。 [深入了解](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest)如何使用 Azure CLI 來登入。

    ![AKS](./media/contoso-migration-rebuild/aks5.png)

7. 他們會執行下列命令，以傳遞 ContosoRG 的資源群組名稱、AKS 叢集 smarthotel-aks-eus2 的名稱和新的登錄名稱。

    ```
    .\gen-aks-env.ps1  -resourceGroupName ContosoRg -orchestratorName smarthotelakseus2 -registryName smarthotelacreus2
    ```
    ![AKS](./media/contoso-migration-rebuild/aks6.png)

8. Azure 會建立另一個資源群組，其中包含 AKS 叢集的資源。

    ![AKS](./media/contoso-migration-rebuild/aks7.png)

9. 部署完成之後，他們會安裝 **kubectl** 命令列工具。 Azure CloudShell 上已安裝此工具。

    **az aks install-cli**

10. 他們會執行 **kubectl get nodes** 命令來確認對叢集的連線。 節點的名稱和所自動建立資源群組中的 VM 名稱相同。

    ![AKS](./media/contoso-migration-rebuild/aks8.png)

11. 他們會執行下列命令來啟動 Kubernetes 儀表板： 

    **az aks browse --resource-group ContosoRG --name smarthotelakseus2**

12. 隨即會有瀏覽器索引標籤開啟為儀表板。 這是使用 Azure CLI 來建立通道的連線。 

    ![AKS](./media/contoso-migration-rebuild/aks9.png)




## <a name="step-2-configure-the-back-end-pipeline"></a>步驟 2：設定後端管線

### <a name="create-an-azure-devops-project-and-build"></a>建立 Azure DevOps 專案和組建

Contoso 會建立 Azure DevOps 專案，並設定 CI 組建來建立容器，再將它推送至 ACR。 本節中的指示會使用 [SmartHotel360-Azure-Backend](https://github.com/Microsoft/SmartHotel360-Azure-backend) 存放庫。

1. 他們會從 visualstudio.com 建立新的組織 (**contosodevops360.visualstudio.com**)，並將它設定為使用 Git。

2. 他們會建立新的專案 (**SmartHotelBackend**)，此專案使用 Git 來進行版本控制，並使用 Agile 來進行工作流程。

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts1.png) 


3. 他們會匯入 [GitHub 存放庫](https://github.com/Microsoft/SmartHotel360-Azure-backend.git)。

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts2.png)
    
4. 在 [管線] 中，他們會按一下 [建置]，然後使用 Azure Repos Git 作為來源，從存放庫建立一個新管線。 

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts3.png)

6. 他們會選取從空白作業開始著手。

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts4.png)  

7. 他們會選取 [裝載的 Linux 預覽] 作為建置管線。

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts5.png) 
 
8. 在 [第 1 階段] 中，他們會新增 [Docker Compose] 工作。 此工作會建置 Docker Compose。

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts6.png) 

9. 他們會重複相同步驟，以新增另一個 [Docker Compose] 工作。 此工作會將容器推送至 ACR。

     ![Azure DevOps](./media/contoso-migration-rebuild/vsts7.png) 

8. 他們會選取第一個工作 (用來建置)，並為此組件設定 Azure 訂用帳戶、授權和 ACR。 

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts8.png)

9. 他們會指定 **docker-compose.yaml** 檔案在存放庫 [src] 資料夾中的路徑。 他們會選取建置服務映像並包含最新標記。 當動作變更為 [建置服務映像] 時，Azure DevOps 工作的名稱會變更為 [自動建置服務]

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts9.png)

10. 現在，他們會設定第二個 Docker 工作 (用來推送)。 他們會選取訂用帳戶和 [smarthotelacreus2] ACR。 

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts10.png)

11. 他們會再次輸入 docker-compose.yaml 檔案的路徑，然後選取 [推送服務映像] 並包含最新的標記。 當動作變更為 [推送服務映像] 時，Azure DevOps 工作的名稱會變更為 [自動推送服務]

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts11.png)

12. 設定好 Azure DevOps 工作之後，Contoso 會儲存建置管線，並啟動建置程序。

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts12.png)

13. 他們會按一下建置作業以檢查進度。

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts13.png)

14. 建置完成之後，ACR 就會顯示新的存放庫，其中會填入微服務所使用的容器。

    ![Azure DevOps](./media/contoso-migration-rebuild/vsts14.png)


### <a name="deploy-the-back-end-infrastructure"></a>部署後端基礎結構

建立 AKS 叢集並建置 Docker 映像之後，Contoso 管理員現在會部署將供後端微服務運用的基礎結構其餘部分。

- 本節中的指示會使用 [SmartHotel360-Azure-Backend](https://github.com/Microsoft/SmartHotel360-Azure-backend) 存放庫。
- 在 **/deploy/k8s/arm** 資料夾中有一個指令碼可建立所有項目。 

他們會依下列方式進行部署：

1. 他們會開啟開發人員命令提示字元，然後使用 az login 命令來登入 Azure 訂用帳戶。
2. 他們會使用 deploy.cmd 檔案，透過輸入下列命令，將 Azure 資源部署在 ContosoRG 資源群組和 EUS2 區域：

    **.\deploy.cmd azuredeploy ContosoRG -c eastus2**

    ![部署後端](./media/contoso-migration-rebuild/backend1.png)

2. 在 Azure 入口網站 中，他們會擷取每個資料庫的連接字串以供稍後使用。

    ![部署後端](./media/contoso-migration-rebuild/backend2.png)

### <a name="create-the-back-end-release-pipeline"></a>建立後端發行管線

現在，Contoso 管理員會執行下列操作：

- 部署 NGINX 輸入控制器，以允許對服務傳送的輸入流量。
- 將微服務部署至 AKS 叢集。
- 他們的第一步是使用 Azure DevOps 來更新微服務的連接字串。 接著，他們會設定新的 Azure DevOps 發行管線來部署微服務。
- 本節中的指示會使用 [SmartHotel360-Azure-Backend](https://github.com/Microsoft/SmartHotel360-Azure-backend) 存放庫。
- 請注意，此文章未涵蓋某些組態設定 (例如 Active Directory B2C)。 如需有關這些設定的詳細資訊，請參閱存放庫。

他們會建立管線：

1. 他們會使用 Visual Studio，以稍早記下的資料庫連線資訊更新 **/deploy/k8s/config_local.yml** 檔案。

    ![DB 連線](./media/contoso-migration-rebuild/back-pipe1.png)

2. 他們會開啟 Azure DevOps，然後在 SmartHotel360 專案的 [發行] 中，按一下 [+新增管線]。

    ![新增管線](./media/contoso-migration-rebuild/back-pipe2.png)

3. 他們會按一下 [空白作業] 來開始管線，而不使用範本。
4. 他們會提供階段和管線名稱。

      ![階段名稱](./media/contoso-migration-rebuild/back-pipe4.png)

      ![管線名稱](./media/contoso-migration-rebuild/back-pipe5.png)

5. 他們會新增成品。

     ![新增成品](./media/contoso-migration-rebuild/back-pipe6.png)

6. 他們會選取 [Git] 作為來源類型，然後為 SmartHotel360 應用程式指定專案、來源及 master 分支。

    ![成品設定](./media/contoso-migration-rebuild/back-pipe7.png)

7. 他們會按一下工作連結。

    ![工作連結](./media/contoso-migration-rebuild/back-pipe8.png)

8. 他們會新增 Azure PowerShell 工作，以便在 Azure 環境中執行 PowerShell 指令碼。

    ![Azure 中的 PowerShell](./media/contoso-migration-rebuild/back-pipe9.png)

9. 他們會選取工作的 Azure 訂用帳戶，然後從 Git 存放庫中選取 [deploy.ps1] 指令碼。

    ![執行指令碼](./media/contoso-migration-rebuild/back-pipe10.png)


10. 他們會新增指令碼的引數。 此指令碼會刪除所有叢集內容 (**ingress** 和 **ingress controller** 除外)，然後部署微服務。

    ![指令碼引數](./media/contoso-migration-rebuild/back-pipe11.png)

11. 他們會將慣用的 Azure PowerShell 版本設定為最新版本，然後儲存管線。
12. 他們會回到 [發行] 頁面，然後手動建立新的發行。

    ![新增發行](./media/contoso-migration-rebuild/back-pipe12.png)

13. 他們會在建立發行之後按一下該發行，然後在 [動作] 中按一下 [部署]。

      ![部署發行](./media/contoso-migration-rebuild/back-pipe13.png)  

14. 當部署完成時，他們會使用 Azure Cloud Shell 來執行下列命令以檢查服務狀態：**kubectl get services**。


## <a name="step-3-provision-front-end-services"></a>步驟 3：佈建前端服務

Contoso 管理員需要部署將供前端應用程式使用的基礎結構。 他們會建立 Blob 儲存體容器來儲存寵物影像；建立 Cosmos 資料庫來儲存含有寵物資訊的文件；以及建立網站適用的「視覺 API」。 

本節的指示會使用 [SmartHotel360-public-web](https://github.com/Microsoft/SmartHotel360-public-web) 存放庫。

### <a name="create-blob-storage-containers"></a>建立 Blob 儲存體容器

1.  在 Azure 入口網站中，他們會開啟已建立的儲存體帳戶，然後按一下 [Blob]。
2.  他們會建立新的容器 (**Pets**)，並將公用存取層級設定為容器。 使用者會將其寵物相片上傳至此容器。

    ![儲存體 Blob](./media/contoso-migration-rebuild/blob1.png)

3. 他們會建立第二個名為 **settings** 的新容器。 具有所有前端應用程式設定的檔案將會放在此容器中。

    ![儲存體 Blob](./media/contoso-migration-rebuild/blob2.png)

4. 他們會擷取文字檔中的儲存體帳戶存取詳細資料，以供日後參考。

    ![儲存體 Blob](./media/contoso-migration-rebuild/blob2.png)

### <a name="provision-a-cosmos-database"></a>佈建 Cosmos 資料庫

Contoso 管理員會佈建用於存放寵物資訊的 Cosmos 資料庫。

1. 他們會在 Azure Marketplace 中建立 **Azure Cosmos DB**。

    ![Cosmos DB](./media/contoso-migration-rebuild/cosmos1.png)

2. 他們會指定名稱 (**contososmarthotel**)、選取 SQL API，並將其放在美國東部 2 主要區域中的生產資源群組 ContosoRG 內。

    ![Cosmos DB](./media/contoso-migration-rebuild/cosmos2.png)

3. 他們會在資料庫內新增集合，並為其設定預設容量與輸送量。

    ![Cosmos DB](./media/contoso-migration-rebuild/cosmos3.png)


4. 他們會記下資料庫的連線資訊，以供日後參考。 

    ![Cosmos DB](./media/contoso-migration-rebuild/cosmos4.png)


### <a name="provision-computer-vision"></a>佈建電腦視覺

Contoso 管理員會佈建「電腦視覺 API」。 函式會呼叫此 API 來評估使用者所上傳的圖片。

1. 他們會在 Azure Marketplace 中建立**電腦視覺**執行個體。

     ![電腦視覺](./media/contoso-migration-rebuild/vision1.png)

2. 他們會在美國東部 2 主要區域的生產資源群組 ContosoRG 中佈建 API (**smarthotelpets**)。

    ![電腦視覺](./media/contoso-migration-rebuild/vision2.png)

3. 他們會將 API 的連線設定儲存至文字檔，以供日後參考。

     ![電腦視覺](./media/contoso-migration-rebuild/vision3.png)


### <a name="provision-the-azure-web-app"></a>佈建 Azure Web 應用程式

Contoso 管理員會使用 Azure 入口網站來佈建 Web 應用程式。


1. 在入口網站中選取 [Web 應用程式]。

    ![Web 應用程式](media/contoso-migration-rebuild/web-app1.png)

2. 他們會提供應用程式名稱 (**smarthotelcontoso**)、在 Windows 上執行它，並將它放在生產環境資源群組 [ContosoRG] 中。 他們會建立新的 Application Insights 執行個體來進行應用程式監視。

    ![Web 應用程式名稱](media/contoso-migration-rebuild/web-app2.png)

3. 完成之後，他們會瀏覽至應用程式的位址，以檢查是否已成功建立該應用程式。

4. 現在，他們會在 Azure 入口網站中為程式碼建立預備位置。 管線將部署到這個位置。 這可確保會等到管理員執行發行之後，才會將程式碼放到生產環境中。

    ![Web 應用程式預備位置](media/contoso-migration-rebuild/web-app3.png)



### <a name="provision-the-azure-function-app"></a>佈建 Azure 函數應用程式

Contoso 管理員會在 Azure 入口網站中佈建函數應用程式。

1. 他們會選取 [函數應用程式]。

    ![建立函數應用程式](./media/contoso-migration-rebuild/function-app1.png)

2. 他們會提供應用程式名稱 (**smarthotelpetchecker**)。 他們會將應用程式放在生產環境資源群組 [ContosoRG] 中，並將主控方案設定為 [使用情況方案]，然後將應用程式放在 [美國東部 2] 區域中。 系統會建立一個新儲存體帳戶，以及用於進行監視的 Application Insights 執行個體。

    ![函數應用程式設定](./media/contoso-migration-rebuild/function-app2.png)


3. 部署應用程式之後，他們會瀏覽至應用程式位址，以檢查是否已成功建立該應用程式。


## <a name="step-4-set-up-the-front-end-pipeline"></a>步驟 4：設定前端管線

Contoso 管理員會為前端網站建立兩個不同的專案。 

1. 他們會在 Azure DevOps 中建立 **SmartHotelFrontend** 專案。

    ![前端專案](./media/contoso-migration-rebuild/function-app1.png)

2. 他們會將 [SmartHotel360 front-end](https://github.com/Microsoft/SmartHotel360-public-web.git) Git 存放庫匯入至新專案。
3. 針對函數應用程式，他們會建立另一個 Azure DevOps 專案 (SmartHotelPetChecker)，然後將 [PetChecker](https://github.com/Microsoft/SmartHotel360-PetCheckerFunction ) Git 存放庫匯入至此專案。

### <a name="configure-the-web-app"></a>設定 Web 應用程式

Contoso 管理員現在會設定 Web 應用程式以使用 Contoso 資源。

1. 他們會連線至 Azure DevOps 專案，然後從本機將存放庫複製到開發機器。
2. 在 Visual Studio 中，他們會開啟資料夾以顯示存放庫中的所有檔案。

    ![存放庫檔案](./media/contoso-migration-rebuild/configure-webapp1.png)

3. 他們會視需要更新設定變更。

    - 當 Web 應用程式啟動時，它會尋找 **SettingsUrl** 應用程式設定。
    - 此變數必須包含指向設定檔的 URL。
    - 根據預設，所使用的設定會是公用端點。

4.  他們會更新 /config-sample.json/sample.json 檔案。

    - 這是 Web 在使用公用端點時的設定檔。
    - 他們會使用 AKS API 端點、儲存體帳戶及 Cosmos 資料庫的值來編輯 **urls** 和 **pets_config** 區段。
    - URL 應符合 Contoso 所會建立的新 Web 應用程式 DNS 名稱。
    - 對於 Contoso 來說，這是 **smarthotelcontoso.eastus2.cloudapp.azure.com**。

    ![JSON 設定](./media/contoso-migration-rebuild/configure-webapp2.png)

5. 在更新檔案之後，他們會將其重新命名為 **smarthotelsettingsurl**，然後將其上傳至稍早建立的儲存體 Blob 中。

    ![重新命名並上傳](./media/contoso-migration-rebuild/configure-webapp3.png)

6. 他們會按一下檔案以取得 URL。 應用程式會在提取設定檔時使用此 URL。

    ![應用程式 URL](./media/contoso-migration-rebuild/configure-webapp4.png)

7. 在 **appsettings.Production.json** 檔案中，他們會將 **SettingsURL** 更新為新檔案的 URL。

    ![更新 URL](./media/contoso-migration-rebuild/configure-webapp5.png)

### <a name="deploy-the-website-to-the-azure-app-service"></a>將網站部署至 Azure App Service

Contoso 管理員現在已可發佈網站。


1. 他們會開啟 Azure DevOps，然後在 **SmartHotelFrontend** 專案的 [建置及發行] 中，按一下 [+新增管線]。
2. 他們會選取 [Azure DevOps Git] 作為來源。
3. 他們會選取 [ASP.NET Core] 範本。
4. 他們會檢閱管線，並確認是否已選取 [發佈 Web 專案] 和 [壓縮發佈的專案]。

    ![管線設定](./media/contoso-migration-rebuild/vsts-publishfront2.png)

5. 在 [觸發程序] 中，其會啟用持續整合，並新增主要分支。 這可確保每次將解決方案的新程式碼提交給 master 分支時，建置管線都會啟動。

    ![持續整合](./media/contoso-migration-rebuild/vsts-publishfront3.png)

6. 他們會按一下 [儲存並排入佇列] 來啟動建置作業。
7. 建置完成之後，他們會使用 [Azure App Service 部署] 來設定發行管線。
8. 他們會提供一個階段名稱 **Staging**。

    ![環境名稱](./media/contoso-migration-rebuild/vsts-publishfront4.png)

9. 他們會新增成品並選取剛剛設定的組建。

     ![新增成品](./media/contoso-migration-rebuild/vsts-publishfront5.png)

10. 他們會按一下成品上的閃電圖示，然後啟用持續部署。

    ![持續部署](./media/contoso-migration-rebuild/vsts-publishfront6.png)
11. 在 [環境] 中，他們會按一下 [Staging] 底下的 [1 個作業, 1 個工作]。
12. 選取訂用帳戶和應用程式名稱之後，他們會開啟 [Azure App Service 部署] 工作。 此部署已設定成使用 [預備環境] 部署位置。 這會自動在此位置建置要檢閱和核准的程式碼。

     ![位置](./media/contoso-migration-rebuild/vsts-publishfront7.png)

13. 在 [管線] 中，他們會新增新的階段。

    ![新增環境](./media/contoso-migration-rebuild/vsts-publishfront8.png)

14. 他們會選取 [使用位置的 Azure App Service 部署]然後將環境命名為 **Prod**。
15. 他們會按一下 [1 個作業, 2 個工作]，然後選取訂用帳戶、應用程式服務名稱及 [預備環境]**staging** 位置。

    ![環境名稱](./media/contoso-migration-rebuild/vsts-publishfront10.png)

16. 他們會從管線中移除 [將 Azure App Service 部署至位置]。 這是先前的步驟所放置。

    ![從管線中移除](./media/contoso-migration-rebuild/vsts-publishfront11.png)

13. 他們會儲存管線。 他們會在管線上按一下 [部署後的條件]。

    ![部署後](./media/contoso-migration-rebuild/vsts-publishfront12.png)

14. 他們會啟用 [部署後核准]，然後新增開發主管作為核准者。

    ![部署後核准](./media/contoso-migration-rebuild/vsts-publishfront13.png)

15. 在建置管線中，他們會手動啟動建置作業。 這會觸發新的發行管線，而將網站部署至預備位置。 就 Contoso 而言，此位置的 URL 是**https://smarthotelcontoso-staging.azurewebsites.net/**。
16. 在建置完成且發行部署至該位置之後，Azure DevOps 就會傳送電子郵件給開發主管來進行核准。
17. 開發主管會按一下 [檢視核准]，然後便可以在 Azure DevOps 入口網站中核准或拒絕該要求。

    ![核准郵件](./media/contoso-migration-rebuild/vsts-publishfront14.png)

16. 主管會加上註解並核准。 這會啟動 [預備環境] 與 [生產環境] 位置的交換，然後將組建移至生產環境。

    ![核准並交換](./media/contoso-migration-rebuild/vsts-publishfront15.png)

17. 管線會完成交換。

    ![完成交換](./media/contoso-migration-rebuild/vsts-publishfront16.png)

18. 小組會檢查 [生產環境] 位置，以確認該 Web 應用程式已於 **https://smarthotelcontoso.azurewebsites.net/** 投入生產環境。


### <a name="deploy-the-petchecker-function-app"></a>部署 PetChecker 函數應用程式

Contoso 管理員會依下列方式部署應用程式。

1. 他們會透過連線至 Azure DevOps 專案，從本機將存放庫複製到開發機器。
2. 在 Visual Studio 中，他們會開啟資料夾以顯示存放庫中的所有檔案。
3. 他們會開啟 **src/PetCheckerFunction/local.settings.json** 檔案，然後新增用於儲存體、Cosmos 資料庫及「電腦視覺 API」的應用程式設定。

    ![部署函式](./media/contoso-migration-rebuild/function5.png)

4. 他們會認可程式碼，然後透過同步處理將其送回 Azure DevOps，以發佈其變更。
5. 他們會新增一個新的建置管線，然後選取 [Azure DevOps Git] 作為來源。
6. 他們會選取 [ASP.NET Core (.NET Framework)] 範本。
7. 他們會接受範本的預設值。
8. 在 [觸發程序] 中，選取 [啟用持續整合]，然後按一下 [儲存並排入佇列] 來啟動建置作業。
9. 建置成功之後，他們會建置發行管線，其中會新增 [使用位置的 Azure App Service 部署]。
10. 他們會將環境命名為 **Prod**，然後選取訂用帳戶。 他們會將 [應用程式類型] 設定為 [函數應用程式]，並將應用程式服務名稱設定為 **smarthotelpetchecker**。

    ![函式應用程式](./media/contoso-migration-rebuild/petchecker2.png)

11. 他們會新增 [建置] 成品。

    ![構件](./media/contoso-migration-rebuild/petchecker3.png)

12. 他們會啟用 [持續部署觸發程序]，然後按一下 [儲存]。
13. 他們會按一下 [將新組建排入佇列]，以執行完整的 CI/CD 管線。
14. 函式部署完成後會出現在 Azure 入口網站中，且狀態為**執行中**。

    ![部署函式](./media/contoso-migration-rebuild/function6.png)


7. 他們會瀏覽至應用程式以測試 Pet Checker 應用程式是否如預期般運作，其位置在 [http://smarthotel360public.azurewebsites.net/Pets](http://smarthotel360public.azurewebsites.net/Pets)。
8. 他們會按一下虛擬人偶以上傳圖片。

    ![部署函式](./media/contoso-migration-rebuild/function7.png)

9. 他們想要檢查的第一張相片是一頭小型犬。

    ![部署函式](./media/contoso-migration-rebuild/function8.png)

10. 應用程式傳回已接受的訊息。

    ![部署函式](./media/contoso-migration-rebuild/function9.png)






## <a name="review-the-deployment"></a>檢閱部署

在 Azure 中有了所移轉的資源之後，Contoso 現在必須讓新基礎結構完整運作且受到保護。

### <a name="security"></a>安全性

- Contoso 必須確保新資料庫安全無虞。 [深入了解](https://docs.microsoft.com/azure/sql-database/sql-database-security-overview)。
- 應用程式必須更新為搭配使用 SSL 與憑證。 容器執行個體應重新部署為會在 443 上接聽。
- Contoso 應考慮使用 KeyVault 來保護其 Service Fabric 應用程式的祕密。 [深入了解](https://docs.microsoft.com/azure/service-fabric/service-fabric-application-secret-management)。

### <a name="backups-and-disaster-recovery"></a>備份和災害復原

- Contoso 需要檢閱 Azure SQL Database 的備份需求。 [深入了解](https://docs.microsoft.com/azure/sql-database/sql-database-automated-backups)。
- Contoso 應考慮實作 SQL 容錯移轉群組，為資料庫提供區域性容錯移轉。 [深入了解](https://docs.microsoft.com/azure/sql-database/sql-database-geo-replication-overview)。
- Contoso 可以運用適用於 ACR 進階 SKU 的異地複寫功能。 [深入了解](https://docs.microsoft.com/azure/container-registry/container-registry-geo-replication)。
- Cosmos DB 會自動備份。 Contoso 可以[深入了解](https://docs.microsoft.com/azure/cosmos-db/online-backup-and-restore)這個程序。

### <a name="licensing-and-cost-optimization"></a>授權和成本最佳化

- 部署好所有資源之後，Contoso 應根據[基礎結構規劃](contoso-migration-infrastructure.md#set-up-tagging)來指派 Azure 標記。
- 所有授權費用都會併入 Contoso 使用的 PaaS 服務中。 這將會從 EA 中扣除。
- Contoso 會啟用 Microsoft 子公司 Cloudyn 授權的 Azure 成本管理。 它是一種多雲端成本管理解決方案，可協助您利用和管理 Azure 和其他雲端資源。  [深入了解](https://docs.microsoft.com/azure/cost-management/overview) Azure 成本管理。

## <a name="conclusion"></a>結論

在此文章中，Contoso 在 Azure 中重建 SmartHotel360 應用程式。 他們將內部部署應用程式前端 VM 重建成 Azure App Services Web 應用程式。 應用程式後端的建置，則是使用 Azure Kubernetes Service (AKS) 所管理容器中部署的微服務來進行。 Contoso 使用寵物相片應用程式增強了應用程式功能。




