---
title: 使用 Azure 資料庫移轉服務在線上將 SQL Server 移轉至 Azure SQL Database | Microsoft Docs
description: 了解如何使用 Azure 資料庫移轉服務，在線上從內部部署的 SQL Server 移轉至 Azure SQL Database。
services: dms
author: HJToland3
ms.author: jtoland
manager: craigg
ms.reviewer: ''
ms.service: dms
ms.workload: data-services
ms.custom: mvc, tutorial
ms.topic: article
ms.date: 08/31/2018
ms.openlocfilehash: b4cbc7fc7e031fcbd25229792488dbb4002ea23e
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43340206"
---
# <a name="migrate-sql-server-to-azure-sql-database-online-using-dms"></a>使用 DMS 在線上將 SQL Server 移轉至 Azure SQL Database
您可以使用 Azure 資料庫移轉服務，在最短的停止運作時間內將資料庫從內部部署 SQL Server 執行個體移轉至 [Azure SQL Database](https://docs.microsoft.com/azure/sql-database/)。 在本教學課程中，您要使用 Azure 資料庫移轉服務，將已還原至內部部署 SQL Server 2016 (或更新版本) 執行個體的 **Adventureworks2012** 資料庫移轉到 Azure SQL Database。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 使用 Data Migration Assistant 評估您的內部部署資料庫。
> * 使用 Data Migration Assistant 移轉範例結構描述。
> * 建立 Azure 資料庫移轉服務的執行個體。
> * 使用 Azure 資料庫移轉服務來建立移轉專案。
> * 執行移轉。
> * 監視移轉。
> * 下載移轉報告。

> [!IMPORTANT]
> 為了獲得最佳的移轉體驗，Microsoft 建議在目標資料庫所在的同一個 Azure 區域中，建立 Azure 資料庫移轉服務的執行個體。 跨區域或地理位置移動資料可能使移轉程序變慢，並產生錯誤。

## <a name="prerequisites"></a>必要條件
若要完成本教學課程，您需要：

- 下載並安裝 [SQL Server 2012 或更新版本](https://www.microsoft.com/sql-server/sql-server-downloads) (任何版本)。
- 啟用 TCP/IP 通訊協定，在 SQL Server Express 安裝期間預設會停用，方法是遵循[啟用或停用伺服器網路通訊協定](https://docs.microsoft.com/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure)一文中的指示。
- 您可以依照[在 Azure 入口網站中建立 Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-get-started-portal) 一文中的詳細資料來建立 Azure SQL Database 執行個體。
- 下載並安裝 [Data Migration Assistant](https://www.microsoft.com/download/details.aspx?id=53595) (DMA) v3.3 或更新版本。
- 使用 Azure Resource Manager 部署模型來建立 Azure 資料庫移轉服務的 VNET，其使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 提供站對站連線能力給您的內部部署來源伺服器。
- 確定您的 Azure 虛擬網路 (VNET) 網路安全性群組規則不會封鎖下列通訊埠 443、53、9354、445、12000。 如需 Azure VNET NSG 流量篩選的詳細資訊，請參閱[使用網路安全性群組來篩選網路流量](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg)。
- 設定[用於 Database Engine 存取的 Windows 防火牆](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
- 開啟您的 Windows 防火牆以允許 Azure 資料庫移轉服務存取來源 SQL Server，其預設會通過 TCP 連接埠 1433。
- 如果您使用動態連接埠執行多個具名 SQL Server 執行個體，您可以啟用 SQL Browser 服務並允許通過防火牆存取 UDP 連接埠 1434，讓 Azure 資料庫移轉服務連線來源伺服器上的具名執行個體。
- 使用來源資料庫前面的防火牆應用裝置時，您可能必須新增防火牆規則，才能讓 Azure 資料庫移轉服務存取來源資料庫，以進行移轉。
- 針對 Azure SQL Database 伺服器建立伺服器層級的[防火牆規則](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)，以允許 Azure 資料庫移轉服務存取目標資料庫。 提供用於 Azure 資料庫移轉服務之 VNET 的子網路範圍。
- 確定用來連線至來源 SQL Server 執行個體的認證具有 [CONTROL SERVER](https://docs.microsoft.com/sql/t-sql/statements/grant-server-permissions-transact-sql) 權限。
- 確定用來連線至目標 Azure SQL DB 執行個體的認證，在目標 Azure SQL 資料庫上具有 CONTROL DATABASE 權限。
- 來源 SQL Server 版本必須是 SQL Server 2005 或更新版本。 若要確認您的 SQL Server 執行個體正在執行的版本，請參閱[如何判斷 SQL Server 及其元件的版本、版次及更新層級](https://support.microsoft.com/help/321185/how-to-determine-the-version-edition-and-update-level-of-sql-server-an)一文。
- 資料庫必須處於大量記錄或完整復原模式。 若要確認您為 SQL Server 執行個體設定的復原模式，請參閱[檢視或變更資料庫的復原模式 (SQL Server)](https://docs.microsoft.com/sql/relational-databases/backup-restore/view-or-change-the-recovery-model-of-a-database-sql-server?view=sql-server-2017) 一文。
- 確實製作資料庫的完整資料庫備份。 若要建立完整資料庫備份，請參閱[如何： 建立完整資料庫備份 (Transact-SQL)](https://docs.microsoft.com/previous-versions/sql/sql-server-2008-r2/ms191304(v=sql.105)) 一文。
- 若有任何資料表沒有主索引鍵，請在資料庫和特定資料表上啟用異動資料擷取 (CDC)。
    > [!NOTE]
    > 您可以使用下列指令碼來尋找任何沒有主索引鍵的資料表。
    ```
    USE <DBName>;
    go
    SELECT is_tracked_by_cdc, name AS TableName
    FROM sys.tables WHERE type = 'U' and is_ms_shipped = 0 AND
    OBJECTPROPERTY(OBJECT_ID, 'TableHasPrimaryKey') = 0;
     ```
    >若結果顯示有一或多個資料表的 'is_tracked_by_cdc' 為 '0'，請使用[啟用和停用異動資料擷取 (SQL Server)](https://docs.microsoft.com/sql/relational-databases/track-changes/enable-and-disable-change-data-capture-sql-server?view=sql-server-2017) 一文說明的程序，為資料庫和特定資料表啟用異動擷取。

- 設定來源 SQL Server 的散發者角色。

    >[!NOTE]
    > 您可以使用下方的查詢確認複寫元件是否已安裝。
    ```
    USE master;
    DECLARE @installed int;
    EXEC @installed = sys.sp_MS_replication_installed;
    SELECT @installed as installed;
    ```
    若結果傳回建議安裝複寫元件的錯誤訊息，請使用[安裝 SQL Server 複寫](https://docs.microsoft.com/sql/database-engine/install-windows/install-sql-server-replication?view=sql-server-2017)一文中的程序安裝 SQL Server 複寫元件。

    若複寫已安裝，請檢查是否已在來源 SQL Server 上使用下方的 T-SQL 命令設定散發角色。
    ```
    EXEC sp_get_distributor;
    ```

    如果未設定散發 (在上述命令輸出中，散發伺服器顯示為 NULL)，請使用[設定散發](https://docs.microsoft.com/sql/relational-databases/replication/configure-publishing-and-distribution?view=sql-server-2017)一文中提供的指導方針來設定散發。

- 在目標 Azure SQL Database 上停用資料庫觸發程序。
    >[!NOTE]
    > 您可以使用下列查詢，在目標 Azure SQL Database 上尋找資料庫觸發程序：
    ```
    Use <Database name>
    select * from sys.triggers
    DISABLE TRIGGER (Transact-SQL)
    ```
    如需詳細資訊，請參閱 [DISABLE TRIGGER (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/disable-trigger-transact-sql?view=sql-server-2017) 一文。 

## <a name="assess-your-on-premises-database"></a>評估您的內部部署資料庫
將資料從內部部署 SQL Server 執行個體移轉到 Azure SQL Database 之前，您必須評估 SQL Server 資料庫是否有任何可能會阻礙移轉的問題。 使用資料移轉小幫手 v3.3 或更新版本，依照[執行 SQL Server 移轉評估](https://docs.microsoft.com/sql/dma/dma-assesssqlonprem)一文中所描述的步驟，完成內部部署資料庫評估。

若要評估內部部署資料庫，請執行下列步驟：
1.  在 DMA 中選取 [新增] (+) 圖示，然後選取 [評估] 專案類型。
2.  在 [Source server type] \(來源伺服器類型\) 文字方塊中指定專案名稱，選取 [SQL Server]，並在 [Target server type] \(目標伺服器類型\) 文字方塊中，選取 [Azure SQL Database]，然後選取 [建立] 以建立專案。

    當您評估移轉至 Azure SQL Database 的來源 SQL Server 資料庫時，您可以選擇下列其中一種或兩種評估報告類型：
    - 檢查資料庫相容性
    - 檢查功能同位

    預設會選取這兩種報告類型。

3.  在 DMA 中的 [選項] 畫面上，選取 [下一步]。
4.  在 [Select sources] \(選取來源\) 畫面的 [Connect to a server] \(連線到伺服器\) 對話方塊中，提供您 SQL Server 的連線詳細資料，然後選取 [Connect] \(連線\)。
5.  在 [新增資源] 對話方塊中，依序選取 [AdventureWorks2012]、[新增]，然後選取 [開始評估]。

    評估完成時，結果會如下圖所示：

    ![評估資料移轉](media\tutorial-sql-server-to-azure-sql-online\dma-assessments.png)

    對於 Azure SQL Database，評估會識別功能同位問題，以及阻礙移轉的問題。

    - **SQL Server 功能同位**類別提供一組完整的建議、Azure 中可使用的替代方法以及補救步驟，協助您規劃移轉專案所需的時間和精力。
    - **相容性問題**類別則識別反映出相容性問題的部分支援或不支援功能，這些相容性問題可能會阻礙將內部部署 SQL Server 資料庫移轉到 Azure SQL Database。 同時也提供協助您解決這些問題的建議。

6.  選取特定的選項，檢閱評估結果是否有阻礙移轉的問題和功能同位問題。

## <a name="migrate-the-sample-schema"></a>移轉範例結構描述
當您滿意評估結果，且認為選取的資料庫也適合移轉至 Azure SQL Database 之後，請使用 DMA 將結構描述移轉至 Azure SQL Database。

> [!NOTE]
> 在 DMA 中建立移轉專案之前，請務必先確認您已依照必要條件中的說明佈建 Azure SQL Database。 基於本教學課程的目的，Azure SQL Database 的名稱會假設為 **AdventureWorksAzure**，但您可以命名為不同的名稱。

若要將 **AdventureWorks2012** 結構描述移轉到 Azure SQL Database，請執行下列步驟：

1.  在資料移轉小幫手中，選取新增 (+) 圖示，然後在 [專案類型] 底下選取 [移轉]。
2.  在 [Source server type] \(來源伺服器類型\) 文字方塊中指定專案名稱，選取 [SQL Server]，然後在 [Target server type] \(目標伺服器類型\) 文字方塊中，選取 [Azure SQL Database]。
3.  在 [Migration Scope] \(移轉範圍\) 下，選取 [Schema only] \(僅結構描述\)。

    執行先前的步驟之後，DMA 介面應會如下圖所示：
    
    ![建立 Data Migration Assistant 專案](media\tutorial-sql-server-to-azure-sql-online\dma-create-project.png)

4.  選取 [Create] \(建立\) 以建立專案。
5.  在 DMA 中，為您的 SQL Server 指定來源連線詳細資料，選取 [連線]，然後選取 [AdventureWorks2012] 資料庫。

    ![Data Migration Assistant 來源連線詳細資料](media\tutorial-sql-server-to-azure-sql-online\dma-source-connect.png)

6.  在 [連線到目標伺服器] 下方選取 [下一步]，指定 Azure SQL Database 的目標連線詳細資料、選取 [連線]，然後選取您已在 Azure SQL Database 中預先佈建的 [AdventureWorksAzure] 資料庫。

    ![Data Migration Assistant 目標連線詳細資料](media\tutorial-sql-server-to-azure-sql-online\dma-target-connect.png)

7.  選取 [Next] \(下一步\) 前進到 [Select objects] \(選取物件\) 畫面，您可以指定 **AdventureWorks2012** 資料庫中需要部署至 Azure SQL Database 的結構描述物件。

    預設會選取所有物件。

    ![產生 SQL 指令碼](media\tutorial-sql-server-to-azure-sql-online\dma-assessment-source.png)

8.  選取 [Generate SQL script] \(產生 SQL 指令碼\) 來建立 SQL 指令碼，然後檢閱指令碼是否有任何錯誤。

    ![結構描述指令碼](media\tutorial-sql-server-to-azure-sql-online\dma-schema-script.png)

9.  選取 [Deploy schema] \(部署結構描述\) 以將結構描述部署至 Azure SQL Database，並在結構描述部署好之後，檢查目標伺服器是否有任何異常狀況。

    ![部署結構描述](media\tutorial-sql-server-to-azure-sql-online\dma-schema-deploy.png)

## <a name="register-the-microsoftdatamigration-resource-provider"></a>註冊 Microsoft.DataMigration 資源提供者
1. 登入 Azure 入口網站，選取 [所有服務]，然後選取 [訂用帳戶]。
 
   ![顯示入口網站訂用帳戶](media\tutorial-sql-server-to-azure-sql-online\portal-select-subscription1.png)
       
2. 選取您要在其中建立 Azure 資料庫移轉服務執行個體的訂用帳戶，然後選取 [資源提供者]。
 
    ![顯示資源提供者](media\tutorial-sql-server-to-azure-sql-online\portal-select-resource-provider.png)
    
3.  搜尋移轉，然後在 [Microsoft.DataMigration] 的右邊，選取 [註冊]。
 
    ![註冊資源提供者](media\tutorial-sql-server-to-azure-sql-online\portal-register-resource-provider.png)    

## <a name="create-an-instance"></a>建立執行個體
1.  在 Azure 入口網站中，選取 [+ 建立資源]，搜尋「Azure 資料庫移轉服務」，然後從下拉式清單選取 [Azure 資料庫移轉服務]。

    ![Azure Marketplace](media\tutorial-sql-server-to-azure-sql-online\portal-marketplace.png)

2.  在 [Azure 資料庫移轉服務] 畫面上，選取 [建立]。
 
    ![建立 Azure 資料庫移轉服務執行個體](media\tutorial-sql-server-to-azure-sql-online\dms-create1.png)
  
3.  在 [建立移轉服務] 畫面上，指定服務的名稱、訂用帳戶，以及新的或現有的資源群組。

4. 選取您要在其中建立 Azure 資料庫移轉服務執行個體的位置。 

5. 選取現有的虛擬網路 (VNET) 或建立新的虛擬網路。

    VNET 會為 Azure 資料庫移轉服務提供來源 SQL Server 和目標 Azure SQL Database 執行個體的存取權。

    如需有關如何在 Azure 入口網站中建立 VNET 的詳細資訊，請參閱[使用 Azure 入口網站建立虛擬網路](https://aka.ms/DMSVnet)一文。

6. 選取定價層。

    如需成本和定價層的詳細資訊，請參閱[定價分頁](https://aka.ms/dms-pricing)。

    如果您需要選擇適當 Azure 資料庫移轉服務層的協助，請參閱張貼在[這裡](https://go.microsoft.com/fwlink/?linkid=861067)的建議。  

     ![設定 Azure 資料庫移轉服務執行個體設定](media\tutorial-sql-server-to-azure-sql-online\dms-settings2.png)

7.  選取 [建立] 以建立服務。

## <a name="create-a-migration-project"></a>建立移轉專案
建立服務之後，請在 Azure 入口網站中找出該服務，然後建立新的移轉專案。

1. 在 Azure 入口網站中，選取 [所有服務]，搜尋 Azure 資料庫移轉服務，然後選取 [Azure 資料庫移轉服務]。
 
      ![找出 Azure 資料庫移轉服務的所有執行個體](media\tutorial-sql-server-to-azure-sql-online\dms-search.png)

2. 在 [Azure 資料庫移轉服務] 畫面上，搜尋您建立的 Azure 資料庫移轉服務執行個體名稱，然後選取該執行個體。
 
     ![找出您的 Azure 資料庫移轉服務執行個體](media\tutorial-sql-server-to-azure-sql-online\dms-instance-search.png)
 
3. 選取 [+ 新增移轉專案]。
4. 在 [新增移轉專案] 畫面上指定專案名稱，並在 [來源伺服器類型] 文字方塊中中選取 [SQL Server]，然後在 [目標伺服器類型] 文字方塊中選取 [Azure SQL Database]。
5. 在 [選擇活動類型] 區段中，選取 [線上資料移轉]

    ![建立資料庫移轉服務專案](media\tutorial-sql-server-to-azure-sql-online\dms-create-project3.png)

    > [!NOTE]
    > 或者，您可以選擇 [僅建立專案] 以立即建立移轉專案，並於後續再執行移轉。

6. 選取 [ **儲存**]。

7. 選取 [建立及執行活動]，以建立專案並執行移轉活動。

    ![建立並執行資料庫移轉服務活動](media\tutorial-sql-server-to-azure-sql-online\dms-create-and-run-activity.png)
 
## <a name="specify-source-details"></a>指定來源詳細資料
1. 在 [移轉來源詳細資料] 畫面上，指定來源 SQL Server 執行個體的連線詳細資料。
 
    請務必使用來源 SQL Server 執行個體名稱的完整網域名稱 (FQDN)。 如果無法解析 DNS 名稱，您也可以使用 IP 位址。

2. 如果您尚未在來源伺服器上安裝信任的憑證，請選取 [信任伺服器憑證] 核取方塊。

    未安裝信任的憑證時，SQL Server 會在執行個體啟動時產生自我簽署憑證。 此憑證用來加密用戶端連線的認證。

    > [!CAUTION]
    > 使用自我簽署憑證加密的 SSL 連線不會提供增強式安全性。 這種連線容易受到攔截式攻擊。 在生產環境中，或在連線到網際網路的伺服器上，您不應該仰賴使用自我簽署憑證的 SSL。

   ![來源詳細資料](media\tutorial-sql-server-to-azure-sql-online\dms-source-details3.png)

## <a name="specify-target-details"></a>指定目標詳細資料
1. 選取 [儲存]，然後在 [移轉目標詳細資料] 畫面上指定目標 Azure SQL Database 伺服器的連線詳細資料，此伺服器是使用 DMA 將 **AdventureWorks2012** 結構描述部署到的預先佈建 Azure SQL Database。

    ![選取目標](media\tutorial-sql-server-to-azure-sql-online\dms-select-target3.png)

2. 選取 [儲存]，然後在 [對應到目標資料庫] 畫面上，對應要進行移轉的來源和目標資料庫。

    如果目標資料庫包含與來源資料庫相同的資料庫名稱，Azure 資料庫移轉服務依預設會選取目標資料庫。

    ![對應到目標資料庫](media\tutorial-sql-server-to-azure-sql-online\dms-map-targets-activity3.png)

3. 在 [選取資料表] 畫面上選取 [儲存]，展開資料表清單，然後檢閱受影響欄位的清單。

    Azure 資料庫移轉服務會自動選取存在於目標 Azure SQL Database 執行個體上的所有空來源資料表。 如果您想要重新移轉已包含資料的資料表，就必須在此刀鋒視窗上明確地選取資料表。

    ![選取資料表](media\tutorial-sql-server-to-azure-sql-online\dms-configure-setting-activity3.png)

4.  選取 [儲存]，在 [移轉摘要] 畫面的 [活動名稱] 文字方塊中，指定移轉活動的名稱，然後檢閱摘要，以確定來源和目標詳細資料都與您先前的指定相符。

    ![移轉摘要](media\tutorial-sql-server-to-azure-sql-online\dms-migration-summary.png)

## <a name="run-the-migration"></a>執行移轉
- 選取 [執行移轉]。

    [移轉活動] 視窗隨即出現，且活動的 [狀態] 為 [正在初始化]。

    ![活動狀態 - 正在初始化](media\tutorial-sql-server-to-azure-sql-online\dms-activity-status2.png)

## <a name="monitor-the-migration"></a>監視移轉
1. 在移轉活動畫面上，選取 [重新整理] 以更新顯示，直到移轉的 [狀態] 顯示為 [執行中] 為止。

2. 按一下特定資料庫，以顯示 [載入完整資料] 和 [累加式資料同步] 作業的移轉狀態。

    ![活動狀態 - 進行中](media\tutorial-sql-server-to-azure-sql-online\dms-activity-in-progress.png)

## <a name="perform-migration-cutover"></a>執行完全移轉
初始完整載入完成後，資料庫會標示為 [已可執行完全移轉]。

1. 當您準備好要完成資料庫移轉後，請選取 [開始完全移轉]。

    ![開始完全移轉](media\tutorial-sql-server-to-azure-sql-online\dms-start-cutover.png)
 
2.  確實停止所有傳入來源資料庫的交易；等到 [暫止的變更] 計數器顯示為 **0** 為止。
3.  選取 [確認]，然後選取 [套用]。
4. 當資料庫移轉狀態顯示為 [已完成] 後，請將應用程式連線至新的目標 Azure SQL Database。
 
    ![活動狀態 - 已完成](media\tutorial-sql-server-to-azure-sql-online\dms-activity-completed.png)

## <a name="next-steps"></a>後續步驟
- 如需在線上移轉至「適用於 MySQL 的 Azure 資料庫」時的已知問題和限制，請參閱 [Azure SQL Database 線上移轉的已知問題和因應措施](known-issues-azure-sql-online.md)一文。
- 如需 Azure 資料庫移轉服務的相關資訊，請參閱[什麼是 Azure 資料庫移轉服務？](https://docs.microsoft.com/azure/dms/dms-overview)一文。
- 如需 Azure SQL Database 的相關資訊，請參閱[什麼是 Azure SQL Database 服務？](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)。
