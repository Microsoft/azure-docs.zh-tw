---
title: 將 SQL Server 資料庫備份至 Azure | Microsoft Docs
description: 此教學課程說明如何將 SQL Server 備份至 Azure。 此文章也將說明 SQL Server 復原。
services: backup
documentationcenter: ''
author: markgalioto
manager: carmonm
editor: ''
keywords: ''
ms.assetid: ''
ms.service: backup
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/02/2018
ms.author: markgal;anuragm
ms.custom: ''
ms.openlocfilehash: aab0ac2dfba47741eaf5a75ef46d9ca5f8873d50
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434240"
---
# <a name="back-up-sql-server-databases-to-azure"></a>將 SQL Server 資料庫備份到 Azure

SQL Server 資料庫是需要低復原點目標 (RPO) 和長期保留的重要工作負載。 Azure 備份提供不需要任何基礎結構的 SQL Server 備份解決方案：您無需管理複雜的備份伺服器、管理代理程式和備份儲存體。 Azure 備份可針對在所有執行 SQL Server 的伺服器上或甚至不同工作負載上進行的備份，提供集中式管理。

在此文章中，您將了解：

> [!div class="checklist"]
> * 將 SQL Server 執行個體備份至 Azure 的必要條件。
> * 如何建立及使用復原服務保存庫。
> * 如何設定 SQL Server 資料庫備份。
> * 如何設定復原點的備份 (或保留) 原則。
> * 如何還原資料庫。

開始此文章中的程序之前，您應該具有在 Azure 中執行的 SQL Server 執行個體。 [使用 SQL Marketplace 虛擬機器快速建立 SQL Server 執行個體](../sql-database/sql-database-get-started-portal.md)。

## <a name="public-preview-limitations"></a>公開預覽限制

下列項目是公開預覽的已知限制：

- SQL 虛擬機器 (VM) 需要網際網路連線才能存取 Azure 公用 IP 位址。 如需詳細資訊，請參閱[建立網路連線](backup-azure-sql-database.md#establish-network-connectivity)。
- 在一個復原服務保存庫中保護多達 2,000 個 SQL 資料庫。 額外的 SQL 資料庫應該儲存在不同的復原服務保存庫中。
- [分散式可用性群組備份](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/distributed-availability-groups?view=sql-server-2017)具有限制。
- 不支援 SQL Server Always On 容錯移轉叢集執行個體 (FCI)。
- 請使用 Azure 入口網站設定 Azure 備份以保護 SQL Server 資料庫。 目前不支援 Azure PowerShell、Azure CLI 和 REST API。

如需支援/不支援案例的詳細資訊，請參閱[常見問題集](https://docs.microsoft.com/azure/backup/backup-azure-sql-database#faq)一節。

## <a name="support-for-azure-geos"></a>Azure 地區的支援

下列地區支援 Azure 備份：

- 澳大利亞東南部 (ASE) 
- 巴西南部 (BRS)
- 加拿大中部 (CNC)
- 加拿大東部 (CE)
- 美國中部 (CUS)
- 東亞 (EA)
- 澳大利亞東部 (AE) 
- 美國東部 (EUS)
- 美國東部 2 (EUS2)
- 印度中部 (INC) 
- 印度南部 (INS)
- 日本東部 (JPE)
- 日本西部 (JPW)
- 南韓中部 (KRC)
- 南韓 (KRS)
- 美國中北部 (NCUS) 
- 北歐 (NE) 
- 美國中南部 (SCUS) 
- 東南亞 (SEA)
- 英國南部 (UKS) 
- 英國西部 (UKW) 
- 美國中西部 (WCUS)
- 西歐 (WE) 
- 美國西部 (WUS)
- 美國西部 2 (WUS 2) 

## <a name="support-for-operating-systems-and-sql-server-versions"></a>作業系統和 SQL Server 版本的支援

本節說明 Azure 備份對於作業系統和 SQL Server 版本的支援。 支援 SQL Marketplace Azure 虛擬機器，以及非 Marketplace 虛擬機器 (需要手動安裝 SQL Server)。

### <a name="supported-operating-systems"></a>受支援的作業系統

- Windows Server 2012
- Windows Server 2012 R2
- Windows Server 2016

目前不支援 Linux。

### <a name="supported-sql-server-versions-and-editions"></a>支援的 SQL Server 版本

- SQL Server 2012 Enterprise、Standard、Web、Developer、Express
- SQL Server 2014 Enterprise、Standard、Web、Developer、Express
- SQL Server 2016 Enterprise、Standard、Web、Developer、Express
- SQL Server 2017 Enterprise、Standard、Web、Developer、Express

## <a name="prerequisites"></a>必要條件

備份 SQL Server 資料庫之前，請先檢查下列條件：

- 在與裝載 SQL Server 執行個體的虛擬機器相同的區域或地區設定中，識別或[建立復原服務保存庫](backup-azure-sql-database.md#create-a-recovery-services-vault)。
- 檢查備份 SQL 資料庫所需的[虛擬機器權限](backup-azure-sql-database.md#set-permissions-for-non-marketplace-sql-vms)。
- 確認 [SQL 虛擬機器具有網路連線](backup-azure-sql-database.md#establish-network-connectivity)。

> [!NOTE]
> 您一次只能使用一個備份解決方案來備份 SQL Server 資料庫。 請先停用所有其他 SQL 備份再使用此功能；否則，備份會受到影響並失敗。 您可以啟用適用於 IaaS VM 的 Azure 備份並搭配 SQL 備份，而不會有任何衝突。
>

如果環境中存在這些條件，請繼續[設定 SQL Server 資料庫的備份](backup-azure-sql-database.md#configure-backup-for-sql-server-databases)。 如有任何必要條件不存在，則請繼續閱讀。


## <a name="establish-network-connectivity"></a>建立網路連線

若要執行所有作業，SQL 虛擬機器需要連線到 Azure 公用 IP 位址。 SQL 虛擬機器作業 (例如資料庫探索、設定備份、排程備份、還原復原點等) 由於未連線到公用 IP 位址而失敗。 使用下列選項之一來提供清楚的備份流量路徑：

- 將 Azure 資料中心 IP 範圍列入白名單：若要將 Azure 資料中心 IP 範圍列入白名單，請使用[下載中心頁面以取得 IP 範圍的詳細資料和指示](https://www.microsoft.com/download/details.aspx?id=41653)。 
- 部署 HTTP Proxy 伺服器以路由傳送流量：當您在 VM 中備份 SQL 資料庫時，VM 上的備份延伸模組會使用 HTTPS API 將管理命令傳送至 Azure 備份，並將資料傳送至 Azure 儲存體。 備份延伸模組也會使用 Azure Active Directory (Azure AD) 進行驗證。 透過 HTTP Proxy 路由傳送這三項服務的備份延伸模組流量。 延伸模組是唯一為了要存取公用網際網路而設定的元件。

這些選項之間的權衡取捨依據包括管理能力、細微控制和成本。

> [!NOTE]
> Azure 備份的服務標記應該已正式運作。
>

| 選項 | 優點 | 缺點 |
| ------ | ---------- | ------------- |
| 將 IP 範圍列入允許清單 | 沒有額外的成本。 <br/> 如需在某個 NSG 中開啟存取權，請使用 **Set-AzureNetworkSecurityRule** Cmdlet。 | 由於受影響的 IP 範圍會隨著時間改變，因此難以管理。 <br/>提供整個 Azure 的存取權，而不只是 Azure 儲存體的存取權。|
| 使用 HTTP Proxy   | 允許在 Proxy 中精確控制儲存體 URL。 <br/>VM 的單一網際網路存取點。 <br/> 不會隨著 Azure IP 位址變更。 | 使用 Proxy 軟體執行 VM 時的額外成本。 |

## <a name="set-permissions-for-non-marketplace-sql-vms"></a>設定非 Marketplace SQL VM 的權限

若要備份虛擬機器，Azure 備份需要安裝 **AzureBackupWindowsWorkload** 延伸模組。 如果您使用 Azure Marketplace 虛擬機器，請繼續前往[探索 SQL Server 資料庫](backup-azure-sql-database.md#discover-sql-server-databases)。 如果裝載 SQL 資料庫的虛擬機器不是從 Azure Marketplace 建立的，請完成下列程序來安裝延伸模組並設定適當的權限。 除了 **AzureBackupWindowsWorkload** 延伸模組，Azure 備份還需要 SQL 系統管理員權限才能保護 SQL 資料庫。 為了探索虛擬機器上的資料庫，Azure 備份會建立帳戶 **NT Service\AzureWLBackupPluginSvc**。 若要讓 Azure 備份探索 SQL 資料庫，**NT Service\AzureWLBackupPluginSvc** 帳戶必須具有 SQL 和 SQL 系統管理員權限。 下列程序說明如何提供這些權限。

若要設定權限：

1. 在 [Azure 入口網站](https://portal.azure.com)中，開啟您用來保護 SQL 資料庫的復原服務保存庫。

2. 在 [復原服務保存庫] 儀表板上，選取 [備份]。 [備份目標] 功能表隨即開啟。

   ![選取 [備份] 以開啟 [備份目標] 功能表](./media/backup-azure-sql-database/open-backup-menu.png)

3. 在 [備份目標] 功能表上，將 [工作負載的執行位置] 設定為預設值：**Azure**。

4. 展開 [您要備份什麼] 下拉式清單方塊，然後選取 [Azure VM 中的 SQL Server]。

    ![選取 Azure VM 中的 SQL Server 來進行備份](./media/backup-azure-sql-database/choose-sql-database-backup-goal.png)

    [備份目標] 功能表會顯示兩個步驟：[探索 VM 中的 DB] 和 [設定備份]。 [探索 VM 中的 DB] 步驟會開始搜尋 Azure 虛擬機器。

    ![檢閱兩個備份目標步驟](./media/backup-azure-sql-database/backup-goal-menu-step-one.png)

5. 在 [探索 VM 中的 DB] 底下，選取 [開始探索] 以搜尋訂用帳戶中未受保護的虛擬機器。 搜尋所有虛擬機器可能需要一段時間。 搜尋時間取決於訂用帳戶中未受保護的虛擬機器數目。

    ![在搜尋 VM 中的 DB 期間會將備份擱置](./media/backup-azure-sql-database/discovering-sql-databases.png)
 
    找到未受保護的虛擬機器之後，該虛擬機器會出現在清單中。 未受保護的虛擬機器會依其虛擬機器名稱和資源群組列出。 多部虛擬機器可能會有相同的名稱。 不過，相同名稱的虛擬機器會屬於不同的資源群組。 如果未列出預期的虛擬機器，請確定是否已為保存庫保護該虛擬機器。

6. 在虛擬機器清單中，選取要備份的 SQL 資料庫所在的 VM，然後選取 [探索 DB]。 

    探索程序會在虛擬機器上安裝 **AzureBackupWindowsWorkload** 延伸模組。 延伸模組可讓 Azure 備份服務與虛擬機器通訊，以備份 SQL 資料庫。 安裝延伸模組之後，Azure 備份會在虛擬機器上建立 Windows 虛擬服務帳戶 **NT Service\AzureWLBackupPluginSvc**。 此虛擬服務帳戶需要 SQL 系統管理員權限。 在虛擬服務帳戶安裝過程中，如果您看到錯誤 `UserErrorSQLNoSysadminMembership`，請參閱[修正 SQL 系統管理員權限](backup-azure-sql-database.md#fix-sql-sysadmin-permissions)。

    [通知] 區域會顯示資料庫探索的進度。 此作業可能需要一些時間來完成。 作業時間取決於虛擬機器上的資料庫數目。 探索到選取的資料庫之後，即會出現成功訊息。

    ![部署成功訊息](./media/backup-azure-sql-database/notifications-db-discovered.png)

建立資料庫與復原服務保存庫的關聯之後，下一個步驟是[設定備份作業](backup-azure-sql-database.md#configure-backup-for-sql-server-databases)。

### <a name="fix-sql-sysadmin-permissions"></a>修正 SQL 系統管理員權限

在安裝過程中，如果您收到錯誤 `UserErrorSQLNoSysadminMembership`，請使用具有 SQL Server 系統管理員權限的帳戶登入 SQL Server Management Studio (SSMS)。 除非您需要特殊權限，否則 Windows 驗證應該能運作。

1. 在 SQL Server 上，開啟 [安全性]/[登入] 資料夾。

    ![開啟 [安全性]/[登入] 資料夾來查看帳戶](./media/backup-azure-sql-database/security-login-list.png)

2. 以滑鼠右鍵按一下 [登入] 資料夾，然後選取 [新增登入]。 在 [登入 - 新增] 對話方塊中，選取 [搜尋]。

    ![在 [登入 - 新增] 對話方塊中，選取 [搜尋]](./media/backup-azure-sql-database/new-login-search.png)

3. Windows 虛擬服務帳戶**NT Service\AzureWLBackupPluginSvc** 已於虛擬機器註冊期間和 SQL 探索階段建立。 請輸入 [輸入要選取的物件名稱] 方塊中所顯示的帳戶名稱。 選取 [檢查名稱] 以解析名稱。 

    ![選取 [檢查名稱] 以解析未知的服務名稱](./media/backup-azure-sql-database/check-name.png)

4. 選取 [確定] 關閉對話方塊。

5. 在 [伺服器角色] 方塊中，確定已選取 [系統管理員] 角色。 選取 [確定] 關閉對話方塊。

    ![確定已選取系統管理員伺服器角色](./media/backup-azure-sql-database/sysadmin-server-role.png)

    現在應該存在必要權限。

6. 雖然您已修正權限錯誤，但您仍然需要建立資料庫與復原服務保存庫的關聯。 在 Azure 入口網站的 [受保護的伺服器] 清單中，以滑鼠右鍵按一下處於錯誤狀態的伺服器，然後選取 [重新探索 DB]。

    ![確認伺服器具有適當的權限](./media/backup-azure-sql-database/check-erroneous-server.png)

    [通知] 區域會顯示資料庫探索的進度。 此作業可能需要一些時間來完成。 作業時間取決於虛擬機器上的資料庫數目。 找到選取的資料庫之後，即會出現成功訊息。

    ![部署成功訊息](./media/backup-azure-sql-database/notifications-db-discovered.png)

建立資料庫與復原服務保存庫的關聯之後，下一個步驟是[設定備份作業](backup-azure-sql-database.md#configure-backup-for-sql-server-databases)。

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="discover-sql-server-databases"></a>探索 SQL Server 資料庫

Azure 備份會探索 SQL Server 執行個體上的所有資料庫。 您可以根據備份需求來保護資料庫。 請使用下列程序來識別裝載 SQL 資料庫的虛擬機器。 識別虛擬機器之後，Azure 備份會安裝輕量型延伸模組來探索 SQL Server 資料庫。

1. 在 [Azure 入口網站](https://portal.azure.com/)中登入您的訂用帳戶。

2. 在左側功能表上，選取 [所有服務]。

    ![選取所有服務](./media/backup-azure-sql-database/click-all-services.png) <br/>

3. 在 [所有服務] 對話方塊中，輸入 **Recovery Services**。 當您輸入時，您的輸入會篩選資源清單。 選取清單中的 [復原服務保存庫]。

    ![輸入並選擇復原服務保存庫](./media/backup-azure-sql-database/all-services.png) <br/>

    隨即會在訂用帳戶中出現 [復原服務保存庫] 清單。 

4. 在 [復原服務保存庫] 清單中，選取要用來保護 SQL 資料庫的保存庫。

5. 在 [復原服務保存庫] 儀表板上，選取 [備份]。 [備份目標] 功能表隨即開啟。

   ![選取 [備份] 以開啟 [備份目標] 功能表](./media/backup-azure-sql-database/open-backup-menu.png)

6. 在 [備份目標] 功能表上，將 [工作負載的執行位置] 設定為預設值：**Azure**。

7. 展開 [您要備份什麼] 下拉式清單方塊，然後選取 [Azure VM 中的 SQL Server]。

    ![選取 Azure VM 中的 SQL Server 來進行備份](./media/backup-azure-sql-database/choose-sql-database-backup-goal.png)

    [備份目標] 功能表會顯示兩個步驟：[探索 VM 中的 DB] 和 [設定備份]。
    
    ![檢閱兩個備份目標步驟](./media/backup-azure-sql-database/backup-goal-menu-step-one.png)

8. 在 [探索 VM 中的 DB] 底下，選取 [開始探索] 以搜尋訂用帳戶中未受保護的虛擬機器。 搜尋所有虛擬機器可能需要一段時間。 搜尋時間取決於訂用帳戶中未受保護的虛擬機器數目。

    ![在搜尋 VM 中的 DB 期間會將備份擱置](./media/backup-azure-sql-database/discovering-sql-databases.png)
 
    找到未受保護的虛擬機器之後，該虛擬機器會出現在清單中。 多部虛擬機器可以有相同的名稱。 不過，相同名稱的虛擬機器會屬於不同的資源群組。 未受保護的虛擬機器會依其虛擬機器名稱和資源群組列出。 如果未列出預期的虛擬機器，請確定是否已為保存庫保護該虛擬機器。

9. 在虛擬機器清單中，選取要備份的 SQL 資料庫所在的 VM，然後選取 [探索 DB]。

    Azure 備份會探索虛擬機器上的所有 SQL 資料庫。 如需資料庫探索階段期間發生狀況的資訊，請參閱[背景作業](backup-azure-sql-database.md#background-operations)。 探索到 SQL 資料庫之後，即可開始[設定備份作業](backup-azure-sql-database.md#configure-backup-for-sql-server-databases)。

### <a name="background-operations"></a>背景作業

當您使用 [Discover DBs] \(探索 DB\) 工具時，Azure 備份會在背景執行下列作業：

- 向復原服務保存庫註冊用於工作負載備份的虛擬機器。 已註冊虛擬機器上的所有資料庫只能備份至此復原服務保存庫。 

- 在虛擬機器上安裝 **AzureBackupWindowsWorkload** 延伸模組。 SQL 資料庫的備份是無代理程式解決方案。 此延伸模組會安裝在虛擬機器上，SQL 資料庫上則不會安裝任何代理程式。

- 在虛擬機器上建立服務帳戶 **NT Service\AzureWLBackupPluginSvc**。 所有備份和還原作業都會使用此服務帳戶。 **NT Service\AzureWLBackupPluginSvc** 需要 SQL 系統管理員權限。 所有 SQL Marketplace 虛擬機器一上線就已安裝了 **SqlIaaSExtension**。 **AzureBackupWindowsWorkload** 延伸模組會使用 **SQLIaaSExtension** 來自動取得必要的權限。 如果虛擬機器未安裝 **SqlIaaSExtension**，「探索 DB」作業會失敗，並出現錯誤訊息 `UserErrorSQLNoSysAdminMembership`。 若要為備份新增系統管理員權限，請遵循[設定非 Marketplace SQL VM 的 Azure 備份權限](backup-azure-sql-database.md#set-permissions-for-non-marketplace-sql-vms)中的指示進行。

    ![選取 VM 和資料庫](./media/backup-azure-sql-database/registration-errors.png)

## <a name="configure-backup-for-sql-server-databases"></a>設定 SQL Server 資料庫的備份

Azure 備份提供管理服務來保護您的 SQL Server 資料庫及管理備份工作。 管理和監視功能取決於您的復原服務保存庫。 

> [!NOTE]
> 您一次只能使用一個備份解決方案來備份 SQL Server 資料庫。 請先停用所有其他 SQL 備份再使用此功能；否則，備份會受到影響並失敗。 您可以啟用適用於 IaaS VM 的 Azure 備份並搭配 SQL 備份，而不會有任何衝突。
>

若要為 SQL 資料庫設定保護：

1. 開啟向 SQL 虛擬機器註冊的復原服務保存庫。

2. 在 [復原服務保存庫] 儀表板上，選取 [備份]。 [備份目標] 功能表隨即開啟。

   ![選取 [備份] 以開啟 [備份目標] 功能表](./media/backup-azure-sql-database/open-backup-menu.png)

3. 在 [備份目標] 功能表上，將 [工作負載的執行位置] 設定為預設值：**Azure**。

4. 展開 [您要備份什麼] 下拉式清單方塊，然後選取 [Azure VM 中的 SQL Server]。

    ![選取 Azure VM 中的 SQL Server 來進行備份](./media/backup-azure-sql-database/choose-sql-database-backup-goal.png)

    [備份目標] 功能表會顯示兩個步驟：[探索 VM 中的 DB] 和 [設定備份]。
    
    如果您已依序完成此文章中的步驟，您即已探索到未受保護的虛擬機器，而且此保存庫已向虛擬機器註冊。 您現在可以設定 SQL 資料庫的保護。
    
5. 在 [備份目標] 功能表中，選取 [設定備份]。

    ![選取 [設定備份]](./media/backup-azure-sql-database/backup-goal-configure-backup.png)

    Azure 備份服務會顯示具有獨立資料庫的所有 SQL Server 執行個體，以及 SQL Server Always On 可用性群組。 若要檢視 SQL Server 執行個體中的獨立資料庫，請選取執行個體名稱左邊的＞形箭號。 下圖顯示獨立執行個體和 AlwaysOn 可用性群組的範例。

    > [!NOTE]
    > SQL Server Always On 可用性群組可接受 SQL 備份喜好設定。 但是因為 SQL 平台的限制，完整和差異備份必須從主要節點進行。 備份喜好設定會決定是否記錄備份。 由於此限制，必須一律為主要節點註冊可用性群組。
    >

    ![SQL 執行個體中的資料庫清單](./media/backup-azure-sql-database/discovered-databases.png)

    選取 Always On 可用性群組左邊的＞形箭號以檢視資料庫清單。

    ![Always On 可用性群組中的資料庫清單](./media/backup-azure-sql-database/discovered-database-availability-group.png)

6. 在資料庫清單中，選取要保護的所有資料庫，然後選取 [確定]。

    ![選取要保護的多個資料庫](./media/backup-azure-sql-database/select-multiple-database-protection.png)

    一次最多可選取 50 個資料庫。 若要保護的資料庫超過 50 個，請分次進行。 保護前 50 個資料庫之後，請重複此步驟來保護下一組資料庫。

    > [!Note] 
    > 為了最佳化備份載入，Azure 備份會將大型備份工作分成多個批次。 每個備份工作的資料庫數目上限為 50。
    >
    >

7. 若要建立或選擇備份原則，請在 [備份] 功能表中，選取 [備份原則]。 [備份原則] 功能表隨即開啟。

    ![選取備份原則](./media/backup-azure-sql-database/select-backup-policy.png)

8. 在 [選擇備份原則] 下拉式清單方塊中，選擇備份原則，然後選取 [確定]。 如需如何建立備份原則的資訊，請參閱[定義備份原則](backup-azure-sql-database.md#define-a-backup-policy)。

   > [!NOTE]
   > 在預覽期間，您無法編輯備份原則。 若您想要使用清單上沒有提供的原則，便必須建立該原則。 如需建立新備份原則的相關資訊，請參閱[定義備份原則](backup-azure-sql-database.md#define-a-backup-policy)一節。

    ![從清單中選擇備份原則](./media/backup-azure-sql-database/select-backup-policy-steptwo.png)

    在 [備份原則] 功能表的 [選擇備份原則] 下拉式清單方塊中，您可以： 
    - 選取預設原則：**HourlyLogBackup**。
    - 選擇之前為 SQL 建立的現有備份原則。
    - 根據 RPO 和保留範圍[定義新的原則](backup-azure-sql-database.md#define-a-backup-policy)。 

    > [!Note]
    > Azure 備份支援以 Grandfather-Father-Son 備份配置為基礎的長期保留。 此配置會將後端儲存體使用量最佳化，同時符合合規性需求。
    >

9. 在選擇備份原則後，請於 [備份] 功能表中，選取 [啟用備份]。

    ![啟用所選備份原則](./media/backup-azure-sql-database/enable-backup-button.png)

    在入口網站的 [通知] 區域中，追蹤設定進度。

    ![[通知] 區域](./media/backup-azure-sql-database/notifications-area.png)


### <a name="define-a-backup-policy"></a>定義備份原則

備份原則會定義由備份的進行時間和備份的保留時間長度所構成的矩陣。 請使用 Azure 備份來排程 SQL 資料庫的三種備份類型：

* 完整備份：完整資料庫備份會備份整個資料庫。 完整備份包含特定資料庫中的所有資料或是一組檔案群組或檔案，以及用以復原該資料的足夠記錄。 您每天最多可以觸發一次完整備份。 您可以選擇依每日或每週間隔進行完整備份。 
* 差異備份：差異備份是以先前最近的完整資料備份為基礎。 差異備份只會擷取自完整備份後已變更的資料。 您每天最多可以觸發一次差異備份。 您無法在同一天設定完整備份和差異備份。
* 交易記錄備份：記錄備份最高允許特定秒的時間點還原。 您最多可以每隔 15 分鐘設定交易記錄備份一次。

原則會在復原服務保存庫層級建立。 多個保存庫可以使用相同的備份原則，但您必須將備份原則套用至每個保存庫。 建立備份原則時，每日完整備份是預設值。 您可以新增差異備份，但僅限於將完整備份設定為每週進行時。 下列程序說明如何在 Azure 虛擬機器中建立 SQL Server 執行個體的備份原則。 

> [!NOTE]
> 在預覽中，您無法編輯備份原則。 您必須改為建立具有所需詳細資料的新原則。  
 
若要建立備份原則：

1. 在保護 SQL 資料庫的復原服務保存庫中，按一下 [備份原則]，然後按一下 [新增]。 

   ![開啟建立新備份原則對話方塊](./media/backup-azure-sql-database/new-policy-workflow.png)

   [新增] 功能表隨即出現。

2. 在 [新增] 功能表中，按一下 [Azure VM 中的 SQL Server]。

   ![針對新備份原則選擇原則類型](./media/backup-azure-sql-database/policy-type-details.png)

   選取 [Azure VM 中的 SQL Server] 會定義原則類型，並開啟 [備份原則] 功能表。 [備份原則] 功能表會顯示新 SQL Server 備份原則所需的欄位。

3. 在 [原則名稱] 中，為新原則輸入名稱。

4. 由於完整備份是必要的，因此您無法關閉 [完整備份] 選項。 按一下 [完整備份] 以檢視及編輯原則。 即使您不對 [備份原則] 進行任何變更，您也應該檢視原則詳細資料。

    ![新的備份原則欄位](./media/backup-azure-sql-database/full-backup-policy.png)

    在 [完整備份原則] 功能表中，針對 [備份頻率] 選擇 [每日] 或 [每週]。 若選擇 [每日]，請選取備份作業的開始時間和時區。 您無法為每日完整備份建立差異備份。

   ![每日間隔設定](./media/backup-azure-sql-database/daily-interval.png)

    若選擇 [每週]，請選取備份作業的開始日期 (星期幾)、時間和時區。

   ![每週間隔設定](./media/backup-azure-sql-database/weekly-interval.png)

5. 預設會選取所有 [保留範圍] 選項：每日、每週、每月和每年。 取消選取任何不想要的保留範圍限制。 設定要使用的間隔。 在 [完整備份原則] 功能表中，選取 [確定] 接受設定。

   ![保留範圍間隔設定](./media/backup-azure-sql-database/retention-range-interval.png)

    復原點會根據其保留範圍標記為保留。 例如，如果您選取每日完整備份，每天只會觸發一次完整備份。 系統會根據每週保留範圍和每週保留設定，標記和保留特定日期的備份。 每月和每年保留範圍會以類似方式運作。

6. 若要新增差異備份原則，請選取 [差異備份]。 [差異備份原則] 功能表隨即開啟。 

   ![開啟 [差異備份原則] 功能表](./media/backup-azure-sql-database/backup-policy-menu-choices.png)

    在 [差異備份原則] 功能表中，選取 [啟用] 以開啟頻率和保留控制項。 您每天最多可以觸發一次差異備份。
    
    > [!Important] 
    > 差異備份最多可以保留 180 天。 如果您需要保留更久，則必須使用完整備份。
    >

   ![編輯差異備份原則](./media/backup-azure-sql-database/enable-differential-backup-policy.png)

    選取 [確定] 以儲存原則，然後返回主要 [備份原則] 功能表。

7. 若要新增交易記錄備份原則，請選取 [記錄備份]。 [記錄備份] 功能表隨即開啟。

    在 [記錄備份] 功能表中，選取 [啟用]，然後設定頻率和保留控制項。 記錄備份可以每隔 15 分鐘頻繁地出現，而且最多可以保留 35 天。 選取 [確定] 以儲存原則，然後返回主要 [備份原則] 功能表。

   ![編輯記錄備份原則](./media/backup-azure-sql-database/log-backup-policy-editor.png)

8. 在 [備份原則] 功能表上，選擇是否要啟用 [SQL 備份壓縮]。 預設會停用壓縮。

    Azure 備份在後端使用 SQL 原生備份壓縮。

9. 完成備份原則的編輯之後，請選取 [確定]。 

   ![接受新的備份原則](./media/backup-azure-sql-database/backup-policy-click-ok.png)

## <a name="restore-a-sql-database"></a>還原 SQL 資料庫
Azure 備份提供使用交易記錄備份將個別資料庫還原至特定日期或時間 (以秒計) 的功能。 Azure 備份會根據還原時間，自動決定適當的完整差異和還原資料所需的記錄備份鏈結。

您也可以選取特定完整或差異備份來還原至特定復原點，而不是特定時間。

### <a name="pre-requisite-before-triggering-a-restore"></a>觸發還原的必要條件

1. 您可以將資料庫還原至相同 Azure 區域中的 SQL Server 執行個體。 目的地伺服器必須註冊到和來源相同的復原服務保存庫。  
2. 若要將 TDE 加密資料庫還原至另一部 SQL 伺服器，請依照[這裡](https://docs.microsoft.com/sql/relational-databases/security/encryption/move-a-tde-protected-database-to-another-sql-server?view=sql-server-2017)所記錄的步驟，先將憑證還原至目的地伺服器。
3. 在觸發 "master" 資料庫的還原作業之前，請使用啟動選項 `-m AzureWorkloadBackup` 以單一使用者模式啟動 SQL Server 執行個體。 `-m` 選項的引數是用戶端名稱。 只有此用戶端能夠開啟連線。 針對所有系統資料庫 (模型、master、msdb)，請在觸發還原之前停止 SQL Agent 服務。 關閉可能會嘗試奪取這些資料庫連線的任何應用程式。

### <a name="steps-to-restore-a-database"></a>還原資料庫的步驟：

1. 開啟向 SQL 虛擬機器註冊的復原服務保存庫。

2. 在 [復原服務保存庫] 儀表板上的 [使用方式] 底下，選取 [備份項目] 以開啟 [備份項目] 功能表。

    ![開啟 [備份項目] 功能表](./media/backup-azure-sql-database/restore-sql-vault-dashboard.png).

3. 在 [備份項目] 功能表上的 [備份管理類型] 底下，選取 [Azure VM 中的 SQL]。 

    ![選取 Azure VM 中的 SQL](./media/backup-azure-sql-database/sql-restore-backup-items.png)

    [備份項目] 功能表會顯示 SQL 資料庫清單。 

4. 在 SQL 資料庫清單中，選取要還原的資料庫。

    ![選取要還原的資料庫](./media/backup-azure-sql-database/sql-restore-sql-in-vm.png)

    當您選取資料庫時，其功能表隨即開啟。 此功能表提供資料庫的備份詳細資料，包括：

    * 最早和最近的還原點。
    * 過去 24 小時的記錄備份狀態 (如果設定為交易記錄備份，則適用於完整和大量記錄復原模式的資料庫)。

5. 在所選資料庫的功能表中，選取 [還原 DB]。 [還原] 功能表隨即開啟。

    ![選取 [還原 DB]](./media/backup-azure-sql-database/restore-db-button.png)

    [還原] 功能表開啟時，[還原設定] 功能表也會開啟。 [還原設定] 功能表是設定還原的第一個步驟。 請使用此功能表來選取要將資料還原到哪個位置。 可用選項包括：
    - **替代位置**：將資料庫還原至替代位置，並保留原始來源資料庫。
    - **覆寫 DB**：將資料還原至與原始來源相同的 SQL Server 執行個體。 此選項的效果是要覆寫原始資料庫。

    > [!Important]
    > 如果選取的資料庫屬於 AlwaysOn 可用性群組，SQL Server 不允許覆寫資料庫。 在此情況下，只會啟用 [替代位置] 選項。
    >

    ![[還原設定] 功能表](./media/backup-azure-sql-database/restore-restore-configuration-menu.png)

### <a name="restore-to-an-alternate-location"></a>還原至替代位置

此程序會逐步引導您將資料還原至替代位置。 若要在還原期間覆寫資料庫，請繼續閱讀[還原並覆寫資料庫](backup-azure-sql-database.md#restore-and-overwrite-the-database)。 在此階段，復原服務保存庫已開啟，並可看見 [還原設定] 功能表。 如果您不是在這個階段，請從[還原 SQL 資料庫](backup-azure-sql-database.md#restore-a-sql-database)來開始。

> [!NOTE]
> 您可以將資料庫還原至相同 Azure 區域中的 SQL Server 執行個體。 目的地伺服器必須註冊到復原服務保存庫。 
>

在 [還原設定] 功能表上，[伺服器] 下拉式清單方塊只會顯示已向復原服務保存庫註冊的 SQL Server 執行個體。 如果您需要的伺服器不在清單中，請參閱[探索 SQL Server 資料庫](backup-azure-sql-database.md#discover-sql-server-databases)以尋找伺服器。 在探索過程中，新的伺服器都會註冊到復原服務保存庫。

1. 在 [還原設定] 功能表中：

    * 在 [還原位置] 底下，選取 [替代位置]。
    * 開啟 [伺服器] 下拉式清單方塊，然後選擇要作為資料庫還原目的地的 SQL Server 執行個體。
    * 開啟 [執行個體] 下拉式清單方塊，然後選擇 SQL Server 執行個體。
    * 在 [還原的 DB 名稱] 方塊中，輸入目標資料庫的名稱。
    * 如果適用的話，請選取 [若選取的 SQL 執行個體上已存在相同名稱的 DB 則覆寫]。
    * 選取 [確定] 完成設定目的地，然後繼續選擇還原點。

    ![提供 [還原設定] 功能表的值](./media/backup-azure-sql-database/restore-configuration-menu.png)

2. 在 [選取還原點] 功能表中，選擇 [記錄 (時間點)] 或 [完整與差異] 作為還原點。 若要還原至特定時間點的記錄，請繼續執行此步驟。 若要還原完整與差異還原點，請繼續進行步驟 3。

    ![[選取還原點] 功能表](./media/backup-azure-sql-database/recovery-point-menu.png)

    還原時間點僅適用於具有完整和大量記錄復原模式的資料庫記錄備份。 若要還原至特定時間點：

    1. 選取 [記錄 (時間點)] 作為還原類型。

        ![選擇還原點類型](./media/backup-azure-sql-database/recovery-point-logs.png)

    2. 在 [還原日期/時間] 下，選取迷你行事曆以開啟 [行事曆]。 在 [行事曆] 上，粗體日期表示具有復原點，且會醒目提示目前的日期。 選取具有復原點的日期。 無法選取沒有復原點的日期。

        ![開啟行事曆](./media/backup-azure-sql-database/recovery-point-logs-calendar.png)

        選取日期之後，時間軸圖表會顯示連續範圍中可用的復原點。

    3. 使用時間軸圖表或 [時間] 對話方塊，指定復原點的特定時間。 選取 [確定] 完成還原點步驟。
    
       ![開啟行事曆](./media/backup-azure-sql-database/recovery-point-logs-graph.png)

        這會關閉 [選取還原點] 功能表，並開啟 [進階設定] 功能表。

       ![進階設定功能表](./media/backup-azure-sql-database/restore-point-advanced-configuration.png)

    4. 在 [進階設定] 功能表上：

        * 若要讓資料庫在還原後保持不運作，請將 [使用 NORECOVERY 還原] 設定為 [已啟用]。
        * 若要變更目的地伺服器上的還原位置，請在 [目標] 欄中輸入新的路徑。
        * 選取 [確定] 來核准設定。 關閉 [進階設定] 功能表。

    5. 在 [還原] 功能表上，選取 [還原] 以啟動還原作業。 在 [通知] 區域中追蹤還原進度，或透過選取資料庫功能表上的 [還原作業] 來進行追蹤。

       ![還原作業進度](./media/backup-azure-sql-database/restore-job-notification.png)

3. 在 [選取還原點] 功能表中，選擇 [記錄 (時間點)] 或 [完整與差異] 作為還原點。 若要還原時間點記錄，請回到步驟 2。 此步驟會還原特定完整或差異還原點。 您可以查看過去 30 天的所有完整和差異復原點。 若要查看 30 天前的復原點，請選取 [篩選] 以開啟 [篩選還原點] 功能表。 對於差異復原點，Azure 備份會先還原適當的完整復原點，再套用選取的差異復原點。

    ![[選取還原點] 功能表](./media/backup-azure-sql-database/recovery-point-menu.png)

    1. 在 [選取還原點] 功能表中，選取 [Full & Differential] \(完整與差異\)。

       ![選取完整和差異](./media/backup-azure-sql-database/recovery-point-logs-fd.png)

        此功能表會顯示可用復原點的清單。

    2. 從清單中選取復原點，然後選取 [確定] 以完成還原點程序。 

        ![選擇完整復原點](./media/backup-azure-sql-database/choose-fd-recovery-point.png)

        這會關閉 [還原點] 功能表，並開啟 [進階設定] 功能表。

        ![進階設定功能表](./media/backup-azure-sql-database/restore-point-advanced-configuration.png)

    3. 在 [進階設定] 功能表上：

        * 若要讓資料庫在還原後保持不運作，請將 [使用 NORECOVERY 還原] 設定為 [已啟用]。 預設會停用 [Restore with NORECOVERY] \(使用 NORECOVERY 還原\)。
        * 若要變更目的地伺服器上的還原位置，請在 [目標] 欄中輸入新的路徑。
        * 選取 [確定] 來核准設定。 關閉 [進階設定] 功能表。

    4. 在 [還原] 功能表上，選取 [還原] 以啟動還原作業。 在 [通知] 區域中追蹤還原進度，或透過選取資料庫功能表上的 [還原作業] 來進行追蹤。

       ![還原作業進度](./media/backup-azure-sql-database/restore-job-notification.png)

### <a name="restore-and-overwrite-the-database"></a>還原並覆寫資料庫

此程序會逐步引導您還原資料並覆寫資料庫。 若要還原至替代位置，請繼續閱讀[還原至替代位置](backup-azure-sql-database.md#restore-to-an-alternate-location)。 在此階段，復原服務保存庫已開啟，並可看見 [還原設定] 功能表 (請參閱下圖)。 如果您不是在這個階段，請從[還原 SQL 資料庫](backup-azure-sql-database.md#restore-a-sql-database)來開始。

![[還原設定] 功能表](./media/backup-azure-sql-database/restore-overwrite-db.png)

在 [還原設定] 功能表上，[伺服器] 下拉式清單方塊只會顯示已向復原服務保存庫註冊的 SQL Server 執行個體。 如果您需要的伺服器不在清單中，請參閱[探索 SQL Server 資料庫](backup-azure-sql-database.md#discover-sql-server-databases)以尋找伺服器。 在探索過程中，新的伺服器都會註冊到復原服務保存庫。

1. 在 [還原設定] 功能表中，選取 [覆寫 DB]，然後選取 [確定] 完成設定目的地。 

   ![選取 [覆寫 DB]](./media/backup-azure-sql-database/restore-configuration-overwrite-db.png)

    [伺服器]、[執行個體] 和 [還原的 DB 名稱] 設定並非必要項目。

2. 在 [選取還原點] 功能表中，選擇 [記錄 (時間點)] 或 [完整與差異] 作為還原點。 若要還原至特定時間點的記錄，請繼續執行此步驟。 若要還原完整與差異還原點，請繼續進行步驟 3。

    ![[選取還原點] 功能表](./media/backup-azure-sql-database/recovery-point-menu.png)

    還原時間點僅適用於具有完整和大量記錄復原模式的資料庫記錄備份。 若要還原至特定秒數：

    1. 選取 [記錄 (時間點)] 作為還原點。

        ![選擇還原點類型](./media/backup-azure-sql-database/recovery-point-logs.png)

    2. 在 [還原日期/時間] 下，選取迷你行事曆以開啟 [行事曆]。 在 [行事曆] 上，粗體日期表示具有復原點，且會醒目提示目前的日期。 選取具有復原點的日期。 無法選取沒有復原點的日期。

        ![開啟行事曆](./media/backup-azure-sql-database/recovery-point-logs-calendar.png)

        選取日期之後，時間軸圖表會顯示可用的復原點。

    3. 使用時間軸圖表或 [時間] 對話方塊，指定復原點的特定時間。 選取 [確定] 完成還原點步驟。
    
       ![開啟行事曆](./media/backup-azure-sql-database/recovery-point-logs-graph.png)

        這會關閉 [選取還原點] 功能表，並開啟 [進階設定] 功能表。

       ![進階設定功能表](./media/backup-azure-sql-database/restore-point-advanced-configuration.png)

    4. 在 [進階設定] 功能表上：

        * 若要讓資料庫在還原後保持不運作，請將 [使用 NORECOVERY 還原] 設定為 [已啟用]。
        * 若要變更目的地伺服器上的還原位置，請在 [目標] 欄中輸入新的路徑。
        * 選取 [確定] 來核准設定。 關閉 [進階設定] 功能表。

    5. 在 [還原] 功能表上，選取 [還原] 以啟動還原作業。 在 [通知] 區域中追蹤還原進度，或透過選取資料庫功能表上的 [還原作業] 來進行追蹤。

       ![還原作業進度](./media/backup-azure-sql-database/restore-job-notification.png)

3. 在 [選取還原點] 功能表中，選擇 [記錄 (時間點)] 或 [完整與差異] 作為還原點。 若要還原時間點記錄，請回到步驟 2。 此步驟會還原特定完整或差異還原點。 您可以查看過去 30 天的所有完整和差異復原點。 若要查看 30 天前的復原點，請選取 [篩選] 以開啟 [篩選還原點] 功能表。 對於差異復原點，Azure 備份會先還原適當的完整復原點，再套用選取的差異復原點。

    ![[選取還原點] 功能表](./media/backup-azure-sql-database/recovery-point-menu.png)

    1. 在 [選取還原點] 功能表中，選取 [Full & Differential] \(完整與差異\)。

       ![選取完整和差異](./media/backup-azure-sql-database/recovery-point-logs-fd.png)

        此功能表會顯示可用復原點的清單。

    2. 從清單中選取復原點，然後選取 [確定] 以完成還原點程序。 

        ![選擇完整復原點](./media/backup-azure-sql-database/choose-fd-recovery-point.png)

        這會關閉 [還原點] 功能表，並開啟 [進階設定] 功能表。

        ![進階設定功能表](./media/backup-azure-sql-database/restore-point-advanced-configuration.png)

    3. 在 [進階設定] 功能表上：

        * 若要讓資料庫在還原後保持不運作，請將 [使用 NORECOVERY 還原] 設定為 [已啟用]。 預設會停用 [Restore with NORECOVERY] \(使用 NORECOVERY 還原\)。
        * 若要變更目的地伺服器上的還原位置，請在 [目標] 欄中輸入新的路徑。
        * 選取 [確定] 來核准設定。 關閉 [進階設定] 功能表。

    4. 在 [還原] 功能表上，選取 [還原] 以啟動還原作業。 在 [通知] 區域中追蹤還原進度，或透過選取資料庫功能表上的 [還原作業] 來進行追蹤。

       ![還原作業進度](./media/backup-azure-sql-database/restore-job-notification.png)

## <a name="manage-azure-backup-operations-for-sql-on-azure-vms"></a>管理 Azure VM 上 SQL 的 Azure 備份作業

本節提供 Azure 虛擬機器上 SQL 可用之各種 Azure 備份管理作業的相關資訊。 存在下列高階作業：

* 監視工作
* 備份警示
* 停止保護 SQL 資料庫
* 繼續保護 SQL 資料庫
* 觸發臨機操作備份作業
* 將執行 SQL Server 的伺服器取消註冊

### <a name="monitor-backup-jobs"></a>監視備份作業
Azure 備份是企業級解決方案，可為任何失敗提供進階的備份警示和通知。 (請參閱[檢視備份警示](backup-azure-sql-database.md#view-backup-alerts))。若要監視特定作業，請根據需求來使用下列任何選項。

#### <a name="use-the-azure-portal-for-adhoc-operations"></a>使用 Azure 入口網站進行臨機操作作業
Azure 備份會在**備份作業**入口網站中顯示所有手動觸發或臨機操作的作業。 **備份作業**入口網站中可用的作業包括：
- 所有設定備份作業。
- 手動觸發的備份作業。
- 還原作業。
- 註冊並探索資料庫作業。
- 停止備份作業。 

![備份作業入口網站](./media/backup-azure-sql-database/jobs-list.png)

> [!NOTE]
> **備份作業**入口網站不會顯示所有已排程的備份作業，包括完整、差異及記錄備份。 請使用 SQL Server Management Studio 來監視已排程的備份作業，如下一節中所述。
>

#### <a name="use-sql-server-management-studio-for-backup-jobs"></a>針對備份作業使用 SQL Server Management Studio
Azure 備份使用 SQL 原生 API 來執行所有備份作業。 使用原生 API 從 msdb 資料庫中的 [SQL backupset 資料表](https://docs.microsoft.com/sql/relational-databases/system-tables/backupset-transact-sql?view=sql-server-2017)擷取所有作業資訊。

下列查詢範例會針對名稱為 **DB1** 的資料庫，擷取其所有備份作業。 自訂查詢以進行進階監視。

```
select CAST (
Case type
                when 'D' 
                                 then 'Full'
                when  'I'
                               then 'Differential' 
                ELSE 'Log'
                END         
                AS varchar ) AS 'BackupType',
database_name, 
server_name,
machine_name,
backup_start_date,
backup_finish_date,
DATEDIFF(SECOND, backup_start_date, backup_finish_date) AS TimeTakenByBackupInSeconds,
backup_size AS BackupSizeInBytes
  from msdb.dbo.backupset where user_name = 'NT SERVICE\AzureWLBackupPluginSvc' AND database_name =  <DB1>  
 
```

### <a name="view-backup-alerts"></a>檢視備份警示

記錄備份會每隔 15 分鐘出現一次，因此監視備份作業有時可能很繁瑣。 Azure 備份可在此情況下提供協助。 所有備份失敗都會觸發電子郵件警示。 這些警示會在資料庫層級依錯誤碼合併。 資料庫第一次備份失敗時，才會傳送電子郵件警示。 登入 Azure 入口網站來監視資料庫的所有失敗。 

若要監視備份警示：

1. 在 [Azure 入口網站](https://portal.azure.com)中登入您的 Azure 訂用帳戶。

2. 開啟向 SQL 虛擬機器註冊的復原服務保存庫。

3. 在 [復原服務保存庫] 儀表板上，選取 [警示與事件]。 

   ![選取 [警示和事件]](./media/backup-azure-sql-database/vault-menu-alerts-events.png)

4. 在 [警示與事件] 功能表中，選取 [備份警示] 以檢視警示清單。

   ![選取 [備份警示]](./media/backup-azure-sql-database/backup-alerts-dashboard.png)

### <a name="stop-protection-for-a-sql-server-database"></a>停止保護 SQL Server 資料庫

當您停止保護 SQL Server 資料庫時，Azure 備份會要求是否要保留復原點。 有兩種方式可停止保護 SQL 資料庫︰

* 停止所有未來的備份作業並刪除所有復原點。
* 停止所有未來的備份作業但保留復原點。

保留復原點會產生成本。 SQL 的復原點會產生 SQL 受保護執行個體定價費用，加上所使用的儲存體費用。 如需 SQL 之 Azure 備份定價的詳細資訊，請參閱 [Azure 備份定價頁面](https://azure.microsoft.com/pricing/details/backup/)。 

若要停止保護資料庫：

1. 開啟向 SQL 虛擬機器註冊的復原服務保存庫。

2. 在 [復原服務保存庫] 儀表板上的 [使用方式] 底下，選取 [備份項目] 以開啟 [備份項目] 功能表。

    ![開啟 [備份項目] 功能表](./media/backup-azure-sql-database/restore-sql-vault-dashboard.png).

3. 在 [備份項目] 功能表上的 [備份管理類型] 底下，選取 [Azure VM 中的 SQL]。 

    ![選取 Azure VM 中的 SQL](./media/backup-azure-sql-database/sql-restore-backup-items.png)

    [備份項目] 功能表會顯示 SQL 資料庫清單。 

4. 在 SQL 資料庫清單中，選取要停止保護的資料庫。

    ![選取要停止保護的資料庫](./media/backup-azure-sql-database/sql-restore-sql-in-vm.png)

    當您選取資料庫時，其功能表隨即開啟。

5. 在所選資料庫的功能表中，選取 [停止備份]。 

    ![選取 [停止備份]](./media/backup-azure-sql-database/stop-db-button.png)

    [停止備份] 功能表隨即開啟。

6. 在 [停止備份] 功能表中，選擇 [保留備份資料] 或 [刪除備份資料]。 您也可以選擇提供停止保護的原因和註解。

    ![[停止備份] 功能表](./media/backup-azure-sql-database/stop-backup-button.png)

7. 選取 [停止備份] 以停止保護資料庫。 

### <a name="resume-protection-for-a-sql-database"></a>繼續保護 SQL 資料庫

如果在停止保護 SQL 資料庫時選取 [保留備份資料] 選項，則可以繼續保護。 如果未保留備份資料，則無法繼續保護。 

1. 若要繼續保護 SQL 資料庫，請開啟備份項目並選取 [繼續備份]。

    ![選取 [繼續備份] 以繼續保護資料庫](./media/backup-azure-sql-database/resume-backup-button.png)

   [備份原則] 功能表隨即開啟。

2. 在 [備份原則] 功能表上選取原則，然後選取 [儲存]。

### <a name="trigger-an-adhoc-backup"></a>觸發臨機操作備份

視需要觸發臨機操作備份。 臨機操作備份的類型共有四種：

* 完整備份
* 僅複製完整備份
* 差異備份
* 記錄備份

如需每種類型的詳細資料，請參閱 [SQL 備份類型](https://docs.microsoft.com/sql/relational-databases/backup-restore/backup-overview-sql-server?view=sql-server-2017#types-of-backups)。

### <a name="unregister-a-sql-server-instance"></a>將 SQL Server 執行個體取消註冊

若要在移除保護之後但在刪除保存庫之前，將 SQL Server 執行個體取消註冊：

1. 開啟向 SQL 虛擬機器註冊的復原服務保存庫。

2. 在 [復原服務保存庫] 儀表板上，於 [管理] 底下，選取 [備份基礎結構]。  

   ![選取 [備份基礎結構]](./media/backup-azure-sql-database/backup-infrastructure-button.png)

3. 在 [管理伺服器] 底下，選取 [受保護的伺服器]。

   ![選取 [受保護的伺服器]](./media/backup-azure-sql-database/protected-servers.png)

    [受保護的伺服器] 功能表隨即開啟。 

4. 在 [受保護的伺服器] 功能表中，選取要取消註冊的伺服器。 若要刪除保存庫，您必須將所有伺服器取消註冊。

5. 在 [受保護的伺服器] 功能表中，以滑鼠右鍵按一下受保護的伺服器，然後選取 [刪除]。 

   ![選取 [刪除]](./media/backup-azure-sql-database/delete-protected-server.png)

## <a name="faq"></a>常見問題集

下一節提供 SQL Database 備份的其他相關資訊。

### <a name="can-i-throttle-the-speed-of-the-sql-server-backup-policy"></a>是否可以限制 SQL Server 備份原則的速度？

是。 您可以限制備份原則的執行速率，以儘量降低對 SQL Server 執行個體的影響。

變更設定：

1. 在 SQL Server 執行個體上的 C:\Program Files\Azure Workload Backup\bin 資料夾中，開啟 **TaskThrottlerSettings.json** 檔案。

2. 在 TaskThrottlerSettings.json 檔案中，將 **DefaultBackupTasksThreshold** 設定變更為較低的值 (例如 5)。

3. 儲存您的變更。 關閉檔案。

4. 在 SQL Server 執行個體上，開啟 [工作管理員]。 重新啟動 [Azure 備份工作負載協調器服務]。

### <a name="can-i-run-a-full-backup-from-a-secondary-replica"></a>是否可以從次要複本執行完整備份？

否。 不支援此功能。

### <a name="do-successful-backup-jobs-create-alerts"></a>成功的備份作業是否會建立警示？

否。 成功的備份作業不會產生警示。 只有失敗的備份作業會傳送警示。

### <a name="can-i-see-details-for-scheduled-backup-jobs-in-the-jobs-menu"></a>是否可以在 [作業] 功能表中看到已排程備份作業的詳細資料？

否。 [備份作業] 功能表會顯示臨機操作工作的詳細資訊，但不會顯示已排程的備份作業。 如果已排程的備份作業失敗，則可以在失敗的作業警示中取得詳細資料。 若要監視所有已排程與臨機操作備份作業，請使用 [SQL Server Management Studio](backup-azure-sql-database.md#use-sql-server-management-studio-for-backup-jobs)。

### <a name="when-i-select-a-sql-server-instance-are-future-databases-automatically-added"></a>當我選取 SQL Server 執行個體時，系統是否會自動新增之後的資料庫？

否。 在為 SQL Server 執行個體設定保護時，如果您選取伺服器層級選項，系統會新增所有資料庫。 如果在設定保護之後將資料庫新增至 SQL Server 執行個體，則必須手動新增新的資料庫來保護它們。 資料庫不會自動包含在已設定的保護中。

### <a name="if-i-change-the-recovery-model-how-do-i-restart-protection"></a>如果我變更復原模式，要如何重新啟動保護？

觸發完整備份。 記錄備份會如預期地開始。

### <a name="can-i-protect-sql-always-on-availability-groups-where-the-primary-replica-is-on-premises"></a>是否可以保護主要複本位於內部部署環境的 SQL Always On 可用性群組？

否。 Azure 備份會保護在 Azure 中執行的 SQL Server。 如果可用性群組 (AG) 分散在 Azure 與內部部署機器之間，則只有當主要複本是在 Azure 中執行時，才可保護 AG。 此外，Azure 備份只會保護與復原服務保存庫位於相同 Azure 區域中的執行中節點。

### <a name="can-i-protect-sql-always-on-availability-groups-which-are-spread-across-azure-regions"></a>我是否可以保護散佈在各個 Azure 區域的 SQL Always On 可用性群組
「Azure 備份復原服務保存庫」可以偵測並保護與「復原服務保存庫」位於相同區域中的所有節點。 如果您的「SQL Always On 可用性群組」跨多個 Azure 區域，您就必須從具有主要節點的區域設定備份。 「Azure 備份」將能夠依據備份喜好設定，偵測並保護可用性群組中的所有資料庫。 如果不符合備份喜好設定，備份就會失敗，而您將會收到失敗警示。


## <a name="next-steps"></a>後續步驟

若要深入了解 Azure 備份，請參閱用來備份加密虛擬機器的 Azure PowerShell 範例。

> [!div class="nextstepaction"]
> [備份加密的 VM](./scripts/backup-powershell-sample-backup-encrypted-vm.md)
