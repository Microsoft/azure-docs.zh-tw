---
title: 使用 Azure 資料庫移轉服務在線上將 SQL Server 移轉至 Azure SQL Database 受控執行個體 | Microsoft Docs
description: 了解如何使用 Azure 資料庫移轉服務，在線上從內部部署的 SQL Server 移轉至 Azure SQL Database 受控執行個體。
services: dms
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: ''
ms.service: dms
ms.workload: data-services
ms.custom: mvc, tutorial
ms.topic: article
ms.date: 10/10/2018
ms.openlocfilehash: 56bd3ab61045ff99601ed50cd27332b370204795
ms.sourcegitcommit: 7b0778a1488e8fd70ee57e55bde783a69521c912
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49068021"
---
# <a name="migrate-sql-server-to-azure-sql-database-managed-instance-online-using-dms"></a>使用 DMS 在線上將 SQL Server 遷移至 Azure SQL Database 受控執行個體
您可以使用 Azure 資料庫移轉服務，以最短停機時間將內部部署 SQL Server 執行個體的資料庫遷移至 [Azure SQL Database 受控執行個體](../sql-database/sql-database-managed-instance.md)。 如需其他可能需要手動操作的方法，請參閱[將 SQL Server 執行個體遷移至 Azure SQL Database 受控執行個體](../sql-database/sql-database-managed-instance-migrate.md)一文。

>[!IMPORTANT]
>在線上將專案從 SQL Server 移轉至 Azure SQL Database 受控執行個體目前為預覽狀態，而且必須遵循 [Microsoft Azure 預覽補充使用規定](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

在本教學課程中，您要使用 Azure 資料庫移轉服務，以最短停機時間將 **Adventureworks2012** 資料庫從內部部署 SQL Server 執行個體遷移至 Azure SQL Database 受控執行個體。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 建立 Azure 資料庫移轉服務的執行個體。
> * 使用 Azure 資料庫移轉服務來建立移轉專案並啟動線上移轉。
> * 監視移轉。
> * 在您準備好時進行完全移轉。

## <a name="prerequisites"></a>必要條件
若要完成本教學課程，您需要：

- 使用 Azure Resource Manager 部署模型來建立 Azure 資料庫移轉服務的 VNET，其使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 提供站對站連線能力給您的內部部署來源伺服器。 [了解使用 Azure 資料庫移轉服務進行 Azure SQL DB 受控執行個體移轉的網路拓樸](https://aka.ms/dmsnetworkformi)。
- 確定您的 Azure 虛擬網路 (VNET) 網路安全性群組規則不會封鎖下列通訊埠 443、53、9354、445、12000。 如需 Azure VNET NSG 流量篩選的詳細資訊，請參閱[使用網路安全性群組來篩選網路流量](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg)。
- 設定[用於來源資料庫引擎存取的 Windows 防火牆](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
- 開啟您的 Windows 防火牆以允許 Azure 資料庫移轉服務存取來源 SQL Server，其預設會通過 TCP 連接埠 1433。
- 如果您使用動態連接埠執行多個具名 SQL Server 執行個體，您可以啟用 SQL Browser 服務並允許通過防火牆存取 UDP 連接埠 1434，讓 Azure 資料庫移轉服務連線來源伺服器上的具名執行個體。
- 如果您是在來源資料庫前面使用防火牆設備，您可能必須新增防火牆規則，才能讓 Azure 資料庫移轉服務存取用於移轉的來源資料庫，以及透過 SMB 連接埠 445 存取檔案。
- 依照[在 Azure 入口網站中建立 Azure SQL Database 受控執行個體](https://aka.ms/sqldbmi) \(英文\) 一文中的詳細資料，建立 Azure SQL Database 受控執行個體。
- 確定用來連線來源 SQL Server 和目標受控執行個體的登入是 sysadmin 伺服器角色的成員。
- 提供 SMB 網路共用，其中包含您所有資料庫的完整資料庫備份檔案及後續交易記錄備份檔案，而 Azure 資料庫移轉服務會使用這些檔案來進行資料庫移轉。
- 確認執行來源 SQL Server 執行個體的服務帳戶在您所建立的網路共用上具有寫入權限，而且來源伺服器的電腦帳戶對相同的共用具備讀取/寫入存取權。
- 記下在您先前建立的網路共用上具有完整控制權限的 Windows 使用者 (和密碼)。 Azure 資料庫移轉服務會模擬該使用者認證，以便將備份檔案上傳至 Azure 儲存體容器以進行還原作業。
- 建立 Azure Active Directory 應用程式識別碼，以產生應用程式識別碼金鑰，讓 DMS 服務可用它來連線至目標 Azure Database 受控執行個體和 Azure 儲存體容器。 如需詳細資訊，請參閱[使用入口網站來建立可存取資源的 Active Directory 應用程式和服務主體](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal)一文。
- 建立或記下 Azure 儲存體帳戶的**標準效能層**，這可讓 DMS 服務將資料庫備份檔案上傳至該處，並用於遷移資料庫。  請務必在建立 DMS 服務的相同區域中建立 Azure 儲存體帳戶。

## <a name="register-the-microsoftdatamigration-resource-provider"></a>註冊 Microsoft.DataMigration 資源提供者

1. 登入 Azure 入口網站，選取 [所有服務]，然後選取 [訂用帳戶]。

    ![顯示入口網站訂用帳戶](media\tutorial-sql-server-to-managed-instance-online\portal-select-subscriptions.png)        
2. 選取您要在其中建立 Azure 資料庫移轉服務執行個體的訂用帳戶，然後選取 [資源提供者]。

    ![顯示資源提供者](media\tutorial-sql-server-to-managed-instance-online\portal-select-resource-provider.png)

3. 搜尋移轉，然後在 [Microsoft.DataMigration] 的右邊，選取 [註冊]。

    ![註冊資源提供者](media\tutorial-sql-server-to-managed-instance-online\portal-register-resource-provider.png)   

## <a name="create-an-azure-database-migration-service-instance"></a>建立 Azure 資料庫移轉服務執行個體

1. 在 Azure 入口網站中，選取 [+ 建立資源]，搜尋 [Azure 資料庫移轉服務]，然後從下拉式清單選取 [Azure 資料庫移轉服務]。

     ![Azure Marketplace](media\tutorial-sql-server-to-managed-instance-online\portal-marketplace.png)

2. 在 [Azure 資料庫移轉服務] 畫面上，選取 [建立]。

    ![建立 Azure 資料庫移轉服務執行個體](media\tutorial-sql-server-to-managed-instance-online\dms-create1.png)

3. 在 [建立移轉服務] 畫面上，指定服務的名稱、訂用帳戶，以及新的或現有的資源群組。

4.  選取您要建立 DMS 執行個體的位置。

5. 選取現有的虛擬網路 (VNET) 或建立虛擬網路。
 
    VNET 會為 Azure 資料庫移轉服務提供來源 SQL Server 和目標 Azure SQL Database 受控執行個體的存取權。

    如需有關如何在 Azure 入口網站中建立 VNET 的詳細資訊，請參閱[使用 Azure 入口網站建立虛擬網路](https://aka.ms/DMSVnet)一文。

    如需其他詳細資訊，請參閱[了解使用 Azure 資料庫移轉服務進行 Azure SQL DB 受控執行個體移轉的網路拓樸](https://aka.ms/dmsnetworkformi)一文。

6. 從「業務關鍵 (預覽)」定價層中選取 SKU。

    > [!NOTE]
    > 僅在使用「業務關鍵 (預覽)」層時才支援線上移轉。 
   
    如需成本和定價層的詳細資訊，請參閱[定價分頁](https://aka.ms/dms-pricing)。
   
    ![建立 DMS 服務](media\tutorial-sql-server-to-managed-instance-online\dms-create-service3.png)

7.  選取 [建立] 以建立服務。

## <a name="create-a-migration-project"></a>建立移轉專案

建立服務執行個體之後，請在 Azure 入口網站中找出該服務，然後建立新的移轉專案。

1. 在 Azure 入口網站中，選取 [所有服務]，搜尋 Azure 資料庫移轉服務，然後選取 [Azure 資料庫移轉服務]。

    ![找出 Azure 資料庫移轉服務的所有執行個體](media\tutorial-sql-server-to-managed-instance-online\dms-search.png)

2. 在 [Azure 資料庫移轉服務] 畫面上，搜尋您建立的執行個體名稱，然後選取該執行個體。
 
3. 選取 [+ 新增移轉專案]。

4. 在 [新增移轉專案] 畫面上指定專案名稱、在 [來源伺服器類型] 文字方塊中選取 [SQL Server]、在 [目標伺服器類型] 文字方塊中選取 [Azure SQL Database 受控執行個體]，然後針對 [選擇活動類型]，選取 [線上資料移轉 (預覽)]。

   ![建立 DMS 專案](media\tutorial-sql-server-to-managed-instance-online\dms-create-project3.png)

5. 選取 [建立及執行活動]，以建立專案。

## <a name="specify-source-details"></a>指定來源詳細資料

1. 在 [移轉來源詳細資料] 畫面上，指定來源 SQL Server 的連線詳細資料。

2. 如果您尚未在伺服器上安裝信任的憑證，請選取 [信任伺服器憑證] 核取方塊。

    未安裝信任的憑證時，SQL Server 會在執行個體啟動時產生自我簽署憑證。 此憑證用來加密用戶端連線的認證。

    > [!CAUTION]
    > 使用自我簽署憑證加密的 SSL 連線不會提供增強式安全性。 這種連線容易受到攔截式攻擊。 在生產環境中，或在連線到網際網路的伺服器上，您不應該仰賴使用自我簽署憑證的 SSL。

   ![來源詳細資料](media\tutorial-sql-server-to-managed-instance-online\dms-source-details2.png)

3. 選取 [ **儲存**]。

## <a name="specify-target-details"></a>指定目標詳細資料

1.  在 [移轉目標詳細資料] 畫面上，指定**應用程式識別碼**和**金鑰**，DMS 執行個體會使用這些項目來連線到 Azure SQL Database 受控執行個體的目標執行個體及 Azure 儲存體帳戶。

    如需詳細資訊，請參閱[使用入口網站來建立可存取資源的 Active Directory 應用程式和服務主體](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal)一文。
    
2. 選取**訂用帳戶** (其中包含 Azure SQL Database 受控執行個體的目標執行個體)，然後選取目標執行個體。

    如果您尚未佈建 Azure SQL Database 受控執行個體，請選取[連結](https://aka.ms/SQLDBMI)來協助您佈建執行個體。 當 Azure SQL Database 受控執行個體的執行個體準備就緒時，返回此特定專案來執行移轉。

3. 提供 **SQL 使用者**和**密碼**，以連線到 Azure SQL Database 受控執行個體的目標執行個體。

       ![Select Target](media\tutorial-sql-server-to-managed-instance-online\dms-target-details3.png)

2.  選取 [ **儲存**]。

## <a name="select-source-databases"></a>選取來源資料庫

1. 在 [選取來源資料庫] 畫面上，選取您要遷移的來源資料庫。

    ![選取來源資料庫](media\tutorial-sql-server-to-managed-instance-online\dms-select-source-databases2.png)

2. 選取 [ **儲存**]。

## <a name="configure-migration-settings"></a>設定移轉設定
 
1. 在 [設定移轉設定] 畫面上，提供下列詳細資料：

    | | |
    |--------|---------|
    |**SMB 網路位置共用** | SMB 網路共用包含完整資料庫備份檔案及交易記錄備份檔案，Azure 資料庫移轉服務會使用這些檔案來進行移轉。 執行來源 SQL Server 執行個體的服務帳戶在此網路共用上必須具有讀取\寫入權限。 在網路共用中提供伺服器的 FQDN 或 IP 位址，例如，'\\\servername.domainname.com\backupfolder' 或 '\\\IP address\backupfolder'。|
    |**使用者名稱** | 請確定 Windows 使用者對於您先前提供的網路共用具有完整控制權限。 Azure 資料庫移轉服務將會模擬該使用者認證，以便將備份檔案上傳至 Azure 儲存體容器以進行還原作業。 |
    |**密碼** | 使用者的密碼。 |
    |**Azure 儲存體帳戶的訂用帳戶** | 選取包含 Azure 儲存體帳戶的訂用帳戶。 |
    |**Azure 儲存體帳戶** | 選取 Azure 儲存體帳戶，讓 DMS 可以將備份檔案從 SMB 網路共用上傳至該處，並用於資料庫移轉。  建議您選取與 DMS 服務位在相同區域的儲存體帳戶，以達到最佳檔案上傳效能。 |
    
    ![設定移轉設定](media\tutorial-sql-server-to-managed-instance-online\dms-configure-migration-settings4.png)

2. 選取 [ **儲存**]。
 
## <a name="review-the-migration-summary"></a>檢閱移轉摘要

1. 在 [移轉摘要] 畫面上的 [活動名稱] 文字方塊中，指定移轉活動的名稱。

2. 檢閱並確認與移轉專案相關聯的詳細資料。
 
    ![移轉專案摘要](media\tutorial-sql-server-to-managed-instance-online\dms-project-summary3.png)

## <a name="run-and-monitor-the-migration"></a>執行並監視移轉

1. 選取 [執行移轉]。

2. 在移轉活動畫面上，選取 [重新整理] 以更新顯示。
 
   ![移轉活動進行中](media\tutorial-sql-server-to-managed-instance-online\dms-monitor-migration2.png)

    您可以進一步展開資料庫和登入類別，以監視個別伺服器物件的移轉狀態。

   ![移轉活動進行中](media\tutorial-sql-server-to-managed-instance-online\dms-monitor-migration-extend2.png)

## <a name="performing-migration-cutover"></a>執行完全移轉

當完整資料庫備份在 Azure SQL Database 受控執行個體的目標執行個體上還原後，此資料庫就可用來執行完全移轉。

1.  當您準備好要完成線上資料庫移轉後，請選取 [開始完全移轉]。

2.  停止對來源資料庫傳入任何流量。

3.  進行 [結尾記錄備份]、在 SMB 網路共用中提供備份檔案，然後等到最後一個交易記錄備份還原完成。

    此時，您會看到 [暫止的變更] 已設為 0。

4.  選取 [確認]，然後選取 [套用]。

    ![準備完成完全移轉](media\tutorial-sql-server-to-managed-instance-online\dms-complete-cutover.png)

5.  當資料庫移轉狀態顯示為 [已完成] 後，請將應用程式連線至 Azure SQL Database 受控執行個體的新目標執行個體。

    ![完全移轉完成](media\tutorial-sql-server-to-managed-instance-online\dms-cutover-complete.png)

## <a name="next-steps"></a>後續步驟

- 如需如何使用 T-SQL 還原命令將資料庫遷移至受控執行個體的教學課程，請參閱[使用還原命令將備份還原到受控執行個體](../sql-database/sql-database-managed-instance-restore-from-backup-tutorial.md)。
- 如需受控執行個體的詳細資訊，請參閱[受控執行個體是什麼](../sql-database/sql-database-managed-instance.md)。
- 如需將應用程式連線至受控執行個體的相關資訊，請參閱[應用程式連線](../sql-database/sql-database-managed-instance-connect-app.md)。
