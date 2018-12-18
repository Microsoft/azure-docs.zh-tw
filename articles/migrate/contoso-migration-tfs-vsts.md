---
title: 將 Team Foundation Server 部署重構到 Azure 中的 Azure DevOps Services | Microsoft Docs
description: 了解 Contoso 如何藉由將內部部署的 TFS 部署遷移至 Azure 中的 Azure DevOps Services，以重構部署。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: raynew
ms.openlocfilehash: a304cb08ec001587af5e6ea740853bd8435824e7
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44297920"
---
# <a name="contoso-migration--refactor-a-team-foundation-server-deployment-to-azure-devops-services"></a>Contoso 移轉：將 Team Foundation Server 部署重構到 Azure DevOps Services

本文說明 Contoso 如何將其內部部署 Team Foundation Server (TFS) 部署遷移至 Azure 中的 Azure DevOps Services，以進行重構。 Contoso 的開發小組在過去五年均使用 TFS 進行小組共同作業和原始檔控制。 現在，他們想要移轉至雲端式解決方案以進行開發和測試工作，以及原始檔控制。 當他們移轉至 Azure DevOps 模型，並開發新的雲端原生應用程式時，Azure DevOps Services 將扮演重要角色。

這份文件是系列文章中的其中一篇，說明 Contoso 這家虛構的公司如何將其內部部署資源遷移至 Microsoft Azure 雲端。 整個系列文章中包含背景資訊以及一些案例，用以說明如何設定移轉基礎結構，以及執行不同類型的移轉。 案例的複雜性會提高。 我們會不定期新增其他文章。

**文章** | **詳細資料** | **狀態**
--- | --- | ---
[文章 1：概觀](contoso-migration-overview.md) | 簡單介紹 Contoso 的移轉策略、文章系列以及我們使用的範例應用程式。 | 可用
[文章 2：部署 Azure 基礎結構](contoso-migration-infrastructure.md) | 描述 Contoso 如何準備其內部部署和 Azure 基礎結構來進行移轉。 所有 Contoso 移轉案例都使用相同的基礎結構。 | 可用
[文章 3：評估內部部署資源](contoso-migration-assessment.md)  | 說明 Contoso 如何評定他們在 VMware 上執行的內部部署兩層式 SmartHotel 應用程式。 他們利用 [Azure Migrate](migrate-overview.md) 服務來評定應用程式 VM，以及利用 [Azure Database Migration Assistant](https://docs.microsoft.com/sql/dma/dma-overview?view=sql-server-2017) 來評定應用程式 SQL Server 資料庫。 | 可用
[文章 4：重新裝載至 Azure VM 和 SQL 受控執行個體](contoso-migration-rehost-vm-sql-managed-instance.md) | 示範 Contoso 如何將 SmartHotel 應用程式遷移至 Azure。 他們使用 [Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview) 來遷移應用程式 Web VM，以及使用 [Azure 資料庫移轉](https://docs.microsoft.com/azure/dms/dms-overview)服務，將應用程式資料庫遷移至 SQL 受管理的執行個體。 | 可用
[文章 5：重新裝載至 Azure VM](contoso-migration-rehost-vm.md) | 說明 Contoso 如何使用 Site Recovery 服務，將其 SmartHotel 遷移至 Azure IaaS VM。
[文章 6：重新裝載至 Azure VM 和 SQL Server 可用性群組](contoso-migration-rehost-vm-sql-ag.md) | 說明 Contoso 如何遷移 SmartHotel 應用程式。 他們使用 Site Recovery 來遷移應用程式 VM，以及使用資料庫移轉服務，將應用程式資料庫遷移至 SQL Server 可用性群組。 | 可用
[文章 7：將 Linux 應用程式重新裝載至 Azure VM](contoso-migration-rehost-linux-vm.md) | 說明 Contoso 如何使用 Azure Site Recovery，將其 osTicket Linux 應用程式遷移至 Azure IaaS VM。
[文章 8：將 Linux 應用程式重新裝載至 Azure VM 和 Azure MySQL Server](contoso-migration-rehost-linux-vm-mysql.md) | 示範 Contoso 如何遷移 osTicket Linux 應用程式。 他們會使用 Site Recovery 進行 VM 遷移，以及使用 MySQL Workbench 來遷移至 Azure MySQL Server 執行個體。 | 可用
[文章 9：將應用程式重構到 Azure Web 應用程式和 Azure SQL 資料庫](contoso-migration-refactor-web-app-sql.md) | 說明 Contoso 如何將 SmartHotel 應用程式移轉至 Azure 容器型 Web 應用程式，以及如何將應用程式資料庫移轉至 Azure SQL Server。 | 可用
[文章 10：將 Linux 應用程式重構到 Azure App Service 和 Azure MySQL Server](contoso-migration-refactor-linux-app-service-mysql.md) | 說明 Contoso 如何使用 PHP 7.0 Docker 容器，將 osTicket Linux 應用程式移轉至 Azure App Service。 部署的程式碼基底會移轉至 GitHub。 應用程式資料庫會移轉至 Azure MySQL。 | 可用
文章 11：在 Azure DevOps Services 中重構 TFS 部署 | 將 TFS 開發應用程式遷移至 Azure 中的 Azure DevOps Services | 本文
[文章 12：在 Azure 容器 和 Azure SQL Database 上重新建構應用程式](contoso-migration-rearchitect-container-sql.md) | 說明 Contoso 如何將其 SmartHotel 應用程式移轉和重新建構至 Azure。 它們會將應用程式的 Web 層重新建構為 Windows 容器及 Azure SQL Database 中的應用程式資料庫。 | 可用
[文章 13：重建 Azure 中的應用程式](contoso-migration-rebuild.md) | 示範 Contoso 如何使用各種 Azure 功能和服務 (包括 App Services、Azure Kubernetes、Azure Functions、認知服務及 Cosmos DB) 重建其 SmartHotel 應用程式。 | 可用


## <a name="business-drivers"></a>商業動機

IT 領導小組與商務夥伴密切合作，試圖找出未來的目標。 商業夥伴並不太擔心開發工具和技術，但是他們歸納出以下幾點：

- **軟體**：無論核心業務為何，現今所有的公司都可說是軟體公司，包括 Contoso 在內。 企業領導者想知道 IT 如何引領公司為使用者提供嶄新的工作方式，以及新的客戶體驗。
- **效率**：Contoso 必須精簡流程，並且為開發人員和使用者去除不必要的程序。 這可讓公司更有效率地達到客戶的要求。 企業需要的是快速、但不浪費時間或金錢的 IT。
- **靈活度**：Contoso IT 必須快速因應商業需求，並且比市場更早做出回應，才能在全球經濟態勢中勝出。 IT 不可成為企業的絆腳石。

## <a name="migration-goals"></a>移轉目標

Contoso 雲端小組已擬定好移轉至 Azure DevOps Services 的目標：

- 小組需要將資料遷移至雲端的工具。 有幾項程序必須手動執行。
- 他們必須移轉去年的工作項目資料和歷程記錄。
- 他們不想設定新的使用者名稱和密碼。 目前的所有系統指派都必須保留。
- 他們想要從 Team Foundation 版本控制 (TFVC) 移轉至 Git，以進行原始檔控制。
- 完全移轉至 Git 的作業將是「頂層移轉」，只會匯入最新版的原始程式碼。 此移轉將在所有工作都因程式碼基底替換而停止的停機期間執行。 他們了解，在移動之後，將只有目前的主要分支歷程記錄可供使用。
- 他們想知道會有何變更，並且想在進行完整的移轉之前先進行測試。 他們想在移轉至 Azure DevOps Services 之後仍保留對 TFS 的存取。
- 他們有多個集合，但是想先從只有幾個專案的集合開始嘗試，以進一步了解程序。
- 他們了解 TFS 集合具有與 Azure DevOps Services 組織間的一對一關聯性，因此會有多個 URL。 不過，這與他們目前程式碼基底和專案分離的模型相符。


## <a name="proposed-architecture"></a>建議的架構

- Contoso 會將其 TFS 專案移至雲端，並不會再將其專案或原始檔控制裝載於內部部署。
- TFS 將會遷移至 Azure DevOps Services。
- Contoso 目前有一個名為 **ContosoDev** 的 TFS 集合，將會遷移至稱為 **contosodevmigration.visualstudio.com** 的 Azure DevOps Services 組織。
- 去年的專案、工作項目、錯誤 (bug) 和反覆項目將會遷移至 Azure DevOps Services。
- Contoso 將會使用他們在移轉規劃之初[部署其 Azure 基礎結構](contoso-migration-infrastructure.md)時所設定的 Azure Active Directory。 


![案例架構](./media/contoso-migration-tfs-vsts/architecture.png) 


## <a name="migration-process"></a>移轉程序

Contoso 會按照下列方式完成移轉程序：

1. 相關準備工作非常多。 首先，Contoso 必須將其 TFS 實作升級至支援的層級。 Contoso 目前執行 TFS 2017 Update 3，但若要使用資料庫移轉，就必須執行受支援且具有最新更新的 2018 版。
2. 升級之後，Contoso會執行 TFS 移轉工具，並驗證其集合。
3. Contoso 會建置一組準備檔案，並執行移轉的試執行以進行測試。
4. 然後，Contoso 會執行另一項移轉，這次將是包含工作項目、Bug、衝刺和程式碼的完整移轉。
5. 在移轉之後，Contoso 會將程式碼從 TFVC 移至 Git。

![移轉程序](./media/contoso-migration-tfs-vsts/migration-process.png) 


## <a name="scenario-steps"></a>案例步驟

以下是 Contoso 完成移轉的方式：

> [!div class="checklist"]
> * **步驟 1：建立 Azure 儲存體帳戶**：在執行移轉程序期間將使用此儲存體帳戶。
> * **步驟 2：升級 TFS**：Contoso 會將其部署升級至 TFS 2018 Upgrade 2。 
> * **步驟 3：驗證集合**：Contoso 會在準備移轉時驗證其 TFS 集合。
> * **步驟 4：建置準備檔案**：Contoso 會使用 TFS 移轉工具建立移轉檔案。 


## <a name="step-1-create-a-storage-account"></a>步驟 1：建立儲存體帳戶

1. 在 Azure 入口網站中，Contoso 管理員會建立儲存體帳戶 (**contosodevmigration**)。
2. 他們將這些帳戶放在用於容錯移轉的次要區域中 - 美國中部。 他們使用具有本機備援儲存體的一般用途標準帳戶。

    ![儲存體帳戶](./media/contoso-migration-tfs-vsts/storage1.png) 


**需要其他協助？**

- [Azure 儲存體簡介](https://docs.microsoft.com/azure/storage/common/storage-introduction)。
- [建立儲存體帳戶](https://docs.microsoft.com/azure/storage/common/storage-create-storage-account)。


## <a name="step-2-upgrade-tfs"></a>步驟 2：升級 TFS

Contoso 管理員會將 TFS 伺服器升級至 TFS 2018 Update 2。 在他們開始之前：

- 他們下載了 [TFS 2018 Update 2](https://visualstudio.microsoft.com/downloads/)
- 他們驗證[硬體需求](https://docs.microsoft.com/tfs/server/requirements)，並詳閱[版本資訊](https://docs.microsoft.com/visualstudio/releasenotes/tfs2018-relnotes)和[升級注意事項](https://docs.microsoft.com/tfs/server/upgrade/get-started#before-you-upgrade-to-tfs-2018)。

他們會依照下列方式進行升級：

1. 首先，他們會備份其 TFS 伺服器 (執行於 VMware VM)，並建立 VMware 快照集。

    ![TFS](./media/contoso-migration-tfs-vsts/upgrade1.png) 

2. TFS 安裝程式啟動後，他們會選擇安裝位置。 安裝程式需要存取網際網路。

    ![TFS](./media/contoso-migration-tfs-vsts/upgrade2.png) 

3. 安裝完成後，會啟動 [伺服器組態精靈]。

    ![TFS](./media/contoso-migration-tfs-vsts/upgrade3.png) 

4. 驗證之後，精靈會完成升級。

     ![TFS](./media/contoso-migration-tfs-vsts/upgrade4.png) 

5. 他們會檢閱專案、工作項目和程式碼，以驗證 TFS 安裝。

     ![TFS](./media/contoso-migration-tfs-vsts/upgrade5.png) 

> [!NOTE]
> 在執行某些 TFS 升級時，必須在升級完成後執行 [設定功能精靈]。 [深入了解](https://docs.microsoft.com/azure/devops/reference/configure-features-after-upgrade?utm_source=ms&utm_medium=guide&utm_campaign=vstsdataimportguide&view=vsts)。

**需要其他協助？**

了解如何[升級 TFS](https://docs.microsoft.com/tfs/server/upgrade/get-started)。

## <a name="step-3-validate-the-tfs-collection"></a>步驟 3：驗證 TFS 集合

Contoso 管理員會對 ContosoDev 集合資料庫執行 TFS 移轉工具而加以驗證，再進行移轉。

1. 他們會下載並解壓縮 [TFS 移轉工具](https://www.microsoft.com/download/details.aspx?id=54274)。 務必要下載要執行的 TFS 更新適用的版本。 他們可在管理主控台中檢查版本。

    ![TFS](./media/contoso-migration-tfs-vsts/collection1.png)

2. 他們會指定專案集合的 URL 並執行此工具，以執行驗證：

        **TfsMigrator validate /collection:http://contosotfs:8080/tfs/ContosoDev**


3. 此工具會顯示錯誤。

    ![TFS](./media/contoso-migration-tfs-vsts/collection2.png)

5. 他們在工具所在位置上一層的 **Logs** 資料夾中找到記錄檔。 每個主要驗證都會產生一個記錄檔。 **TfsMigration.log** 包含主要資訊。

    ![TFS](./media/contoso-migration-tfs-vsts/collection3.png)

4. 他們會尋找這個與身分識別有關的項目。

    ![TFS](./media/contoso-migration-tfs-vsts/collection4.png)

5. 他們在命令列上執行 **TfsMigration validate /help**，並發現可能必須執行 **/tenantDomainName** 命令，才能驗證身分識別。

     ![TFS](./media/contoso-migration-tfs-vsts/collection5.png)

6. 他們再次執行驗證命令，並納入此值和他們的 Azure AD 名稱：**TfsMigrator validate /collection:http://contosotfs:8080/tfs/ContosoDev /tenantDomainName:contosomigration.onmicrosoft.com**。

    ![TFS](./media/contoso-migration-tfs-vsts/collection7.png)

7. Azure AD 的 [登入] 畫面出現後，他們輸入全域管理員使用者的認證。

     ![TFS](./media/contoso-migration-tfs-vsts/collection8.png)

8. 驗證通過，且工具加以確認。

    ![TFS](./media/contoso-migration-tfs-vsts/collection9.png)



## <a name="step-4-create-the-migration-files"></a>步驟 4：建立移轉檔案

驗證完成後，Contoso 管理員即可使用 TFS 移轉工具建置移轉檔案。

1. 他們在此工具中執行準備步驟。

    **TfsMigrator prepare /collection:http://contosotfs:8080/tfs/ContosoDev /tenantDomainName:contosomigration.onmicrosoft.com /accountRegion:cus**

     ![準備](./media/contoso-migration-tfs-vsts/prep1.png)

    執行的準備工作如下：
    - 掃描集合以找出所有使用者的清單，並填入身分識別對應記錄 (**IdentityMapLog.csv**])
    - 準備連線至 Azure Active Directory，以尋找每個身分識別的相符項目。
    - Contoso 已部署 Azure AD 並使用 AD Connect 加以同步處理，因此「準備工作」應可找到相符的身分識別，並將其標示為「作用中」。

2. Azure AD 的 [登入] 畫面出現後，他們輸入全域管理員的認證。

    ![準備](./media/contoso-migration-tfs-vsts/prep2.png)

3. 準備完成後，此工具會報告已順利產生匯入檔案。

    ![準備](./media/contoso-migration-tfs-vsts/prep3.png)

4. 此時，他們會看到 IdentityMapLog.csv 和 import.json 檔案已建立於新資料夾中。

    ![準備](./media/contoso-migration-tfs-vsts/prep4.png)

5. Import.json 檔案會提供匯入設定。 其中包含所需組織名稱之類的資訊，以及儲存體帳戶資訊。 大部分的欄位會自動填入。 某些欄位需由使用者輸入。 Contoso 會開啟檔案，並新增要建立的 Azure DevOps Services 組織名稱：**contosodevmigration**。 使用此名稱時，其 Azure DevOps Services URL 將是 **contosodevmigration.visualstudio.com**。

    ![準備](./media/contoso-migration-tfs-vsts/prep5.png)

    > [!NOTE]
    > 組織必須在移轉之前建立，並且可在移轉完成後變更。

6. 他們檢閱身分識別記錄對應檔，其中顯示將在匯入期間移至 Azure DevOps Services 的帳戶。 

    - 作用中的身分識別是指在匯入後會在 Azure DevOps Services 中成為使用者的身分識別。
    - 在移轉之後，這些身分識別將在 Azure DevOps Services 上獲得授權，並且顯示為組織中的使用者。
    - 這些身分識別檔案的 [預期的匯入狀態] 資料行中會標示 [作用中]。

    ![準備](./media/contoso-migration-tfs-vsts/prep6.png)



## <a name="step-5-migrate-to-azure-devops-services"></a>步驟 5：遷移至 Azure DevOps Services

在準備就緒後，Contoso 管理員此時已可專注於移轉工作。 執行移轉之後，他們會從使用 TFVC 改為使用 Git，以進行版本控制。

在開始之前，管理員會排定開發小組的停機時間，使集合離線以進行移轉。 以下是移轉程序的步驟：

1. **中斷集合的連結**：當集合已連結且連線時，集合的身分識別資料會位於 TFS 伺服器組態資料庫中。 當集合從 TFS 伺服器中斷連結時，它會取得該身分識別資料的複本，並將它封裝於集合以進行傳輸。 若沒有這項資料，則無法執行匯入的身分識別部分。 建議應在匯入完成前讓集合持續中斷連結，因為在匯入期間發生的變更是無法匯入的。
2. **產生備份**：移轉程序的下一個步驟，是產生可匯入 Azure DevOps Services 中的備份。 資料層應用程式元件封裝 (DACPAC) 是一項 SQL Server 功能，可讓資料庫變更封裝成單一檔案，並部署至其他 SQL 執行個體。 它也可以直接還原至 Azure DevOps Services，因此可作為將集合資料放入雲端中的封裝方法。 Contoso 會使用 SqlPackage.exe 工具來產生 DACPAC。 此工具隨附於 SQL Server Data Tools 中。
3. **上傳至儲存體**：DACPAC 建立後，他們將其上傳至 Azure 儲存體。 上傳之後，他們會取得共用存取簽章 (SAS)，以允許 TFS 移轉工具存取儲存體。
4. **填寫匯入**：接著，Contoso 可以在匯入檔案中填寫遺漏的欄位，包括 DACPAC 設定。 為此，他們會先指定要執行 **DryRun** 匯入，以確認一切都正常運作，再進行完整移轉。
5. **執行試執行**：試執行匯入有助於測試集合移轉。 試執行的存留期間有限，且會在生產環境移轉執行之前刪除。 試執行會在一段時間後自動刪除。 在匯入完成後收到成功電子郵件中，會包含何時將刪除試執行的相關附註。 請記下該時間並據以進行規劃。
6. **完成生產環境移轉**：在試執行移轉完成後，Contoso 管理員會更新 import.json，並再次執行匯入，以進行最後的移轉。



### <a name="detach-the-collection"></a>中斷集合的連結

開始之前，Contoso 管理員會先建立本機 SQL Server 備份，並建立 TFS 伺服器的 VMware 快照集，再中斷連結。

1.  在 TFS 管理主控台中，他們會選取要中斷連結的集合 (**ContosoDev**)。

    ![移轉](./media/contoso-migration-tfs-vsts/migrate1.png)

2. 在 [一般] 中，他們選取 [中斷連結集合]

    ![移轉](./media/contoso-migration-tfs-vsts/migrate2.png)

3. 在 [中斷連結 Team 專案集合精靈] > [服務訊息] 中，他們為可能嘗試對集合中的專案進行連線的使用者提供訊息。

    ![移轉](./media/contoso-migration-tfs-vsts/migrate3.png)

4. 在 [中斷連結進度] 中，他們監視進度，並在程序完成後按 [下一步]。

    ![移轉](./media/contoso-migration-tfs-vsts/migrate4.png)

5. 在 [整備檢查] 中，他們在檢查完成後按一下 [中斷連結]。

    ![移轉](./media/contoso-migration-tfs-vsts/migrate5.png)

6. 他們按一下 [關閉] 以完成作業。

    ![移轉](./media/contoso-migration-tfs-vsts/migrate6.png)

6. TFS 管理主控台已不再參考該集合。

    ![移轉](./media/contoso-migration-tfs-vsts/migrate7.png)


### <a name="generate-a-dacpac"></a>產生 DACPAC

Contoso 會建立可匯入 Azure DevOps Services 中的備份 (DACPAC)。

- SQL Server Data Tools 中的 SqlPackage.exe 會用來建立 DACPAC。 使用 SQL Server Data Tools 安裝的 SqlPackage.exe 有多個不同版本，位於具有 120、130 和 140 等名稱的資料夾下。 務必要使用正確的版本來準備 DACPAC。
- TFS 2018 匯入需要使用 140 資料夾中的 SqlPackage.exe 或更高版本。  就 CONTOSOTFS 而言，此檔案位於下列資料夾中：**C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\Extensions\Microsoft\SQLDB\DAC\140**。


Contoso 管理員會產生 DACPAC，如下所示：

1. 他們會開啟命令提示字元，並瀏覽至 SQLPackage.exe 的所在位置。 他們會輸入下列命令以產生 DACPAC：

    **SqlPackage.exe /sourceconnectionstring:"Data Source=SQLSERVERNAME\INSTANCENAME;Initial Catalog=Tfs_ContosoDev;Integrated Security=True" /targetFile:C:\TFSMigrator\Tfs_ContosoDev.dacpac /action:extract /p:ExtractAllTableData=true /p:IgnoreUserLoginMappings=true /p:IgnorePermissions=true /p:Storage=Memory** 

    ![Backup ](./media/contoso-migration-tfs-vsts/backup1.png)

2. 此命令執行後，會出現下列訊息。

    ![Backup ](./media/contoso-migration-tfs-vsts/backup2.png)

3. 他們會確認 DACPAC 檔案的屬性

    ![Backup ](./media/contoso-migration-tfs-vsts/backup3.png)

### <a name="update-the-file-to-storage"></a>將檔案更新至儲存體

DACPAC 建立後，Contoso 會將其上傳至 Azure 儲存體。

1. 他們會[下載並安裝](https://azure.microsoft.com/features/storage-explorer/) Azure 儲存體總管。

    ![上傳](./media/contoso-migration-tfs-vsts/backup5.png)

4. 他們會連線至其訂用帳戶，並找出他們為移轉建立的儲存體帳戶 (**contosodevmigration**)。 他們會建立新的 Blob 容器 **azuredevopsmigration**。

    ![上傳](./media/contoso-migration-tfs-vsts/backup6.png)

5. 他們會以區塊 Blob 的形式指定要上傳的 DACPAC 檔案。

    ![上傳](./media/contoso-migration-tfs-vsts/backup7.png)
    
7. 檔案上傳後，他們會按一下檔案名稱 > [產生 SAS]。 他們會展開儲存體帳戶下的 Blob 容器、選取含有匯入檔案的容器，然後按一下 [取得共用存取簽章]。

    ![上傳](./media/contoso-migration-tfs-vsts/backup8.png)

8. 他們接受預設值，然後按一下 [建立]。 這可以啟用 24 小時的存取。

    ![上傳](./media/contoso-migration-tfs-vsts/backup9.png)

9. 他們會複製共用存取簽章 URL，使其可供 TFS 移轉工具使用。

    ![上傳](./media/contoso-migration-tfs-vsts/backup10.png)

> [!NOTE]
> 移轉必須在允許的時間範圍內執行，否則權限將會過期。
> 請勿從 Azure 入口網站產生 SAS 金鑰。 以此方式產生的金鑰會以帳戶為範圍，且無法用於匯入。

### <a name="fill-in-the-import-settings"></a>填入匯入設定

Contoso 管理員稍早已填寫匯入規格檔案 (import.json) 的某些部分。 現在，他們必須新增其餘設定。

他們開啟 import.json 檔案，並填寫下列欄位：• 位置：上面產生的 SAS 金鑰所在的位置。
• Dacpac：將名稱設定為您上傳至儲存體帳戶的 DACPAC 檔案。 必須包含 ".dacpac" 副檔名。
• ImportType：暫時設定為 DryRun。


![匯入設定](./media/contoso-migration-tfs-vsts/import1.png)


### <a name="do-a-dry-run-migration"></a>進行試執行移轉

Contoso 管理員會先進行試執行移轉，以確定一切都正常運作。

1. 他們會開啟命令提示字元，並找出 TfsMigration 的所在位置 (C:\TFSMigrator)。
2. 第一個步驟是驗證匯入檔案。 他們想要確定檔案的格式正確，且 SAS 金鑰有效用。

    **TfsMigrator import /importFile:C:\TFSMigrator\import.json /validateonly**

3. 驗證傳回錯誤，指出 SAS 金鑰需要較長的到期時間。

    ![試執行](./media/contoso-migration-tfs-vsts/test1.png)

3. 他們使用 Azure 儲存體總管建立新的 SAS 金鑰，且到期日設定為七天。

    ![試執行](./media/contoso-migration-tfs-vsts/test2.png)

3. 他們更新 import.json 檔案，並重新執行驗證。 這次的驗證順利完成。

    **TfsMigrator import /importFile:C:\TFSMigrator\import.json /validateonly**

    ![試執行](./media/contoso-migration-tfs-vsts/test3.png)
    
7. 他們啟動試執行：

    **TfsMigrator import /importFile:C:\TFSMigrator\import.json**

8. 系統發出確認移轉的訊息。 請留意在試執行之後將保留暫存資料的時間長度。

    ![試執行](./media/contoso-migration-tfs-vsts/test4.png)

9. 此時會出現 Azure AD 的 [登入] 畫面，應使用 Contoso 管理員的登入資料加以完成。

    ![試執行](./media/contoso-migration-tfs-vsts/test5.png)

10. 訊息顯示匯入的相關資訊。

    ![試執行](./media/contoso-migration-tfs-vsts/test6.png)

11. 約 15 分鐘後，他們瀏覽至 URL，並看到下列資訊：

     ![試執行](./media/contoso-migration-tfs-vsts/test7.png)

12. 移轉完成後，Contoso 開發部門主管可登入 Azure DevOps Services，以確認試執行正常運作。 經過驗證之後，Azure DevOps Services 服務需要一些詳細資料來確認組織。

    ![試執行](./media/contoso-migration-tfs-vsts/test8.png)

13. 在 Azure DevOps 服務中，開發部門主管可以看到專案已遷移至 Azure DevOps Services。 有通知指出組織將在 15 天後刪除。

    ![試執行](./media/contoso-migration-tfs-vsts/test9.png)

14. 開發部門主管開啟其中一個專案，並開啟 [工作項目] > [指派給我]。 這表示工作項目資料已連同身分識別一起遷移。

    ![試執行](./media/contoso-migration-tfs-vsts/test10.png)

15. 主管也檢查了其他專案和程式碼，以確認原始程式碼和歷程記錄皆已遷移。

    ![試執行](./media/contoso-migration-tfs-vsts/test11.png)


### <a name="run-the-production-migration"></a>執行生產環境移轉

在試執行完成後，Contoso 管理員接著執行生產環境移轉。 他們會刪除試執行、更新匯入設定，然後重新執行匯入。

1. 在 Azure DevOps Services 入口網站中，他們會刪除試執行組織。
2. 他們更新 import.json 檔案，以將 **ImportType** 設定為 **ProductionRun**。

    ![Production](./media/contoso-migration-tfs-vsts/full1.png)

3. 他們依照試執行的相同方式開始進行移轉：**TfsMigrator import /importFile:C:\TFSMigrator\import.json**。
4. 此時會出現確認移轉的訊息，並警告資料可保留在安全的暫存區域中，最多七天。

    ![Production](./media/contoso-migration-tfs-vsts/full2.png)

5. 在 Azure AD 的 [登入] 中，他們指定 Contoso 管理員登入。

    ![Production](./media/contoso-migration-tfs-vsts/full3.png)

6. 訊息顯示匯入的相關資訊。

    ![Production](./media/contoso-migration-tfs-vsts/full4.png)

7. 約 15 分鐘後，他們瀏覽至 URL，並看到下列資訊：

    ![Production](./media/contoso-migration-tfs-vsts/full5.png)

8. 移轉完成後，Contoso 開發部門主管可登入 Azure DevOps Services，以確認試移轉正常運作。 登入之後，他可以看到專案已遷移。

    ![Production](./media/contoso-migration-tfs-vsts/full6.png)

8. 開發部門主管開啟其中一個專案，並開啟 [工作項目] > [指派給我]。 這表示工作項目資料已連同身分識別一起遷移。

    ![Production](./media/contoso-migration-tfs-vsts/full7.png)

9. 主管會檢查其他工作項目資料加以確認。

    ![Production](./media/contoso-migration-tfs-vsts/full8.png)

15. 主管也檢查了其他專案和程式碼，以確認原始程式碼和歷程記錄皆已遷移。

    ![Production](./media/contoso-migration-tfs-vsts/full9.png)


### <a name="move-source-control-from-tfvc-to-git"></a>將原始檔控制從 TFVC 移轉至 GIT

移轉完成後，Contoso 想要從 TFVC 移轉至 Git 以進行原始程式碼管理。 他們需要將目前 Azure DevOps Services 組織中的原始程式碼匯入為相同組織中的 Git 存放庫。

1. 在 Azure DevOps Services 入口網站中，他們會開啟其中一個 TFVC 存放庫 (**$/PolicyConnect**)，並加以檢視。

    ![Git](./media/contoso-migration-tfs-vsts/git1.png)

2. 他們按一下 [來源] 下拉式清單 > [匯入]。

    ![Git](./media/contoso-migration-tfs-vsts/git2.png)

3. 在 [來源類型] 中，他們選取 [TFVC]，並指定存放庫的路徑。 他們決定不移轉歷程記錄。

    ![Git](./media/contoso-migration-tfs-vsts/git3.png)

    > [!NOTE]
    > 由於 TFVC 和 Git 儲存版本控制資訊的方式有所差異，建議 Contoso 不要遷移歷程記錄。 這是 Microsoft 將 Windows 和其他產品從集中式版本控制遷移至 Git 時所採取的方法。

4. 匯入之後，管理員會檢閱程式碼。

    ![Git](./media/contoso-migration-tfs-vsts/git4.png)

5. 他們會對第二個存放庫 (**$/SmartHotelContainer**) 重複此程序。

    ![Git](./media/contoso-migration-tfs-vsts/git5.png)

6. 檢閱來源後，開發部門主管認定移轉至 Azure DevOps Services 的作業已完成。 Azure DevOps Services 此時已成為小組內所有涉及移轉的開發來源。

    ![Git](./media/contoso-migration-tfs-vsts/git6.png)



**需要其他協助？**

[深入了解](https://docs.microsoft.com/azure/devops/repos/git/import-from-TFVC?view=vsts)如何從 TFVC 匯入。

##  <a name="clean-up-after-migration"></a>移轉之後進行清除

移轉完成後，Contoso 必須執行下列作業：

- 檢閱[匯入之後](https://docs.microsoft.com/azure/devops/articles/migration-post-import?view=vsts)一文，以了解其他匯入活動的相關資訊。
- 刪除 TFVC 存放庫，或將其置於唯讀模式中。 不得使用程式碼基底，但可以參考其歷程記錄。

## <a name="next-steps"></a>後續步驟

Contoso 將必須為相關小組成員提供 Azure DevOps Services 和 Git 訓練。



