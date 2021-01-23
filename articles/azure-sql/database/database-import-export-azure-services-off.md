---
title: 匯入或匯出 Azure SQL Database，但不允許 Azure 服務存取伺服器。
description: 匯入或匯出 Azure SQL Database，但不允許 Azure 服務存取伺服器。
services: sql-database
ms.service: sql-database
ms.subservice: migration
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/08/2020
ms.openlocfilehash: 3a02876234d43df2e98a3a4e60453fc3f1f74ef6
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98724164"
---
# <a name="import-or-export-an-azure-sql-database-without-allowing-azure-services-to-access-the-server"></a>匯入或匯出 Azure SQL Database，但不允許 Azure 服務存取伺服器
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

本文說明當伺服器上的 [ *允許 Azure 服務* ] 設為 [ *關閉* ] 時，如何匯入或匯出 Azure SQL Database。 工作流程會使用 Azure 虛擬機器來執行 SqlPackage，以執行匯入或匯出作業。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-the-azure-virtual-machine"></a>建立 Azure 虛擬機器

選取 [ **部署至 Azure** ] 按鈕，以建立 azure 虛擬機器。

此範本可讓您使用最新的修補版本，針對 Windows 版本使用幾個不同的選項來部署簡單的 Windows 虛擬機器。 這會將 A2 大小的 VM 部署在資源群組位置，並傳回 VM 的完整功能變數名稱。
<br><br>

[![顯示標示為「部署至 Azure」的按鈕影像。](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-vm-simple-windows%2Fazuredeploy.json)

如需詳細資訊，請參閱 [非常簡單的 WINDOWS VM 部署](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows)。

## <a name="connect-to-the-virtual-machine"></a>連接至虛擬機器

下列步驟說明如何使用遠端桌面連線來連接到您的虛擬機器。

1. 部署完成之後，請移至虛擬機器資源。

   ![螢幕擷取畫面顯示具有 [連接] 按鈕的虛擬機器總覽頁面。](./media/database-import-export-azure-services-off/vm.png)  

2. 選取 [連接]。

   將顯示遠端桌面通訊協定檔案 (.rdp 檔案) 表單，其中包含虛擬機器的公用 IP 位址與連接埠號碼。

   ![RDP 表單](./media/database-import-export-azure-services-off/rdp.png)  

3. 選取 [下載 RDP 檔案]。

   > [!NOTE]
   > 您也可以使用 SSH 來連線到您的 VM。

4. 關閉 **連線至虛擬機器** 表單。
5. 若要連線至您的 VM，請開啟下載的 RDP 檔案。
6. 出現提示時，請選取 [連接]。 在 Mac 上，您需要 RDP 用戶端，例如來自 Mac App Store 的[遠端桌面用戶端](https://apps.apple.com/app/microsoft-remote-desktop-10/id1295203466?mt=12)。

7. 輸入在建立虛擬機器時指定的使用者名稱和密碼，然後選擇 [確定]。

8. 您可能會在登入過程中收到憑證警告。 選擇 [是] 或 [繼續] 以繼續進行連線。

## <a name="install-sqlpackage"></a>安裝 SqlPackage

[下載並安裝最新版本的 SqlPackage](/sql/tools/sqlpackage-download)。

如需詳細資訊，請參閱 [SqlPackage.exe](/sql/tools/sqlpackage)。

## <a name="create-a-firewall-rule-to-allow-the-vm-access-to-the-database"></a>建立防火牆規則以允許 VM 存取資料庫

將虛擬機器的公用 IP 位址新增至伺服器的防火牆。

下列步驟會為您虛擬機器的公用 IP 位址建立伺服器層級 IP 防火牆規則，並啟用虛擬機器的連線能力。

1. 從左側功能表中選取 **[sql 資料庫** ]，然後在 [ **sql 資料庫** ] 頁面上選取您的資料庫。 您資料庫的 [總覽] 頁面隨即開啟，顯示完整的伺服器名稱 (例如 **servername.database.windows.net**) ，並提供進一步設定的選項。

2. 當連接到您的伺服器及其資料庫時，請複製此完整伺服器名稱以供使用。

   ![伺服器名稱](./media/database-import-export-azure-services-off/server-name.png)

3. 在工具列上選取 [設定伺服器防火牆]。 伺服器的 [防火牆設定] 頁面隨即開啟。

   ![伺服器層級 IP 防火牆規則](./media/database-import-export-azure-services-off/server-firewall-rule.png)

4. 選擇工具列上的 [ **新增用戶端 ip** ]，將虛擬機器的公用 ip 位址新增至新的伺服器層級 ip 防火牆規則。 伺服器層級 IP 防火牆規則可以針對單一 IP 位址或 IP 位址範圍開啟連接埠 1433。

5. 選取 [儲存]。 系統會為您虛擬機器的公用 IP 位址建立伺服器層級 IP 防火牆規則，以在伺服器上開啟埠1433。

6. 關閉 [防火牆設定] 頁面。

## <a name="export-a-database-using-sqlpackage"></a>使用 SqlPackage 匯出資料庫

若要使用 [SqlPackage](/sql/tools/sqlpackage) 命令列公用程式匯出 Azure SQL Database，請參閱 [匯出參數和屬性](/sql/tools/sqlpackage#export-parameters-and-properties)。 SqlPackage 公用程式隨附 [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) 和 [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt)的最新版本，您也可以下載最新版本的 [SqlPackage](/sql/tools/sqlpackage-download)。

建議您在大多數生產環境中使用 SqlPackage 公用程式來調整規模和效能。 如需 SQL Server 客戶諮詢小組部落格中有關使用 BACPAC 檔案進行移轉的主題，請參閱[使用 BACPAC 檔案從 SQL Server 移轉至 Azure SQL Database](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files)。

此範例示範如何使用具有 Active Directory 通用驗證的 SqlPackage.exe 來匯出資料庫。 取代為您的環境特有的值。

```cmd
SqlPackage.exe /a:Export /tf:testExport.bacpac /scs:"Data Source=<servername>.database.windows.net;Initial Catalog=MyDB;" /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="import-a-database-using-sqlpackage"></a>使用 SqlPackage 匯入資料庫

若要使用 [SqlPackage](/sql/tools/sqlpackage) 命令列公用程式匯入 SQL Server 資料庫，請參閱[匯入參數和屬性](/sql/tools/sqlpackage#import-parameters-and-properties)。 SqlPackage 具有最新的 [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) 和 [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt)。 您也可以下載最新版本的 [SqlPackage](/sql/tools/sqlpackage-download)。

為了規模和效能，我們建議在大部分生產環境中使用 SqlPackage，而不使用 Azure 入口網站。 如需 SQL Server 客戶諮詢小組部落格中有關使用 `BACPAC` 檔案進行移轉的主題，請參閱[使用 BACPAC 檔案從 SQL Server 移轉至 Azure SQL Database](/archive/blogs/sqlcat/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files) \(英文\)。

下列 SqlPackage 命令會將 **AdventureWorks2017** 資料庫從本機儲存體匯入至 Azure SQL Database。 此命令會建立名為 **myMigratedDatabase**、且具有 **進階** 服務層級和 **P6** 服務目標的新資料庫。 請針對您的環境適當變更這些值。

```cmd
sqlpackage.exe /a:import /tcs:"Data Source=<serverName>.database.windows.net;Initial Catalog=myMigratedDatabase>;User Id=<userId>;Password=<password>" /sf:AdventureWorks2017.bacpac /p:DatabaseEdition=Premium /p:DatabaseServiceObjective=P6
```

> [!IMPORTANT]
> 若要連線到 tAzure SQL Database 從公司防火牆後方，防火牆必須開啟埠1433。

此範例說明如何使用 SqlPackage 與 Active Directory 通用驗證來匯入資料庫。

```cmd
sqlpackage.exe /a:Import /sf:testExport.bacpac /tdn:NewDacFX /tsn:apptestserver.database.windows.net /ua:True /tid:"apptest.onmicrosoft.com"
```

## <a name="performance-considerations"></a>效能考量

匯出速度會因許多因素而有所不同 (例如，資料圖形) 因此無法預測預期的速度。 SqlPackage 可能需要相當長的時間，特別是針對大型資料庫。

若要獲得最佳效能，您可以嘗試下列策略：

1. 請確定沒有其他工作負載正在資料庫上執行。 在匯出之前建立複本可能是確保沒有其他工作負載正在執行的最佳解決方案。
2. 提高 (SLO) 的資料庫服務等級目標，以更妥善處理匯出工作負載 (主要是讀取 i/o) 。 如果資料庫目前 GP_Gen5_4，則業務關鍵層可能有助於讀取工作負載。
3. 請確定有針對大型資料表的叢集索引。
4. 虛擬機器 (Vm) 應該與資料庫位於相同的區域，以協助避免網路限制。
5. 在上傳至 blob 儲存體之前，Vm 應該具有適當大小的 SSD，才能產生暫存成品。
6. 針對特定資料庫，Vm 應該有足夠的核心和記憶體設定。

## <a name="store-the-imported-or-exported-bacpac-file"></a>儲存匯入或匯出。BACPAC 檔案

，.BACPAC 檔案可以儲存在 [Azure blob](../../storage/blobs/storage-blobs-overview.md)或 [Azure 檔案儲存體](../../storage/files/storage-files-introduction.md)中。

若要達到最佳效能，請使用 Azure 檔案儲存體。 SqlPackage 會操作檔案系統，以便直接存取 Azure 檔案儲存體。

若要降低成本，請使用 Azure Blob，其成本低於 premium Azure 檔案共用。 不過，它會要求您複製 [。](/sql/relational-databases/data-tier-applications/data-tier-applications#bacpac) 匯入或匯出作業之前，blob 和本機檔案系統之間的 BACPAC 檔案。 因此，程式需要較長的時間。

以上傳或下載。BACPAC 檔案，請參閱 [使用 AzCopy 和 Blob 儲存體傳輸資料](../../storage/common/storage-use-azcopy-v10.md#transfer-data)，以及 [使用 AzCopy 和檔案儲存體傳輸資料](../../storage/common/storage-use-azcopy-files.md)。

視您的環境而定，您可能需要 [設定 Azure 儲存體防火牆和虛擬網路](../../storage/common/storage-network-security.md)。

## <a name="next-steps"></a>後續步驟

- 若要瞭解如何連接並查詢匯入的 SQL Database，請參閱 [快速入門： Azure SQL Database：使用 SQL Server Management Studio 連接和查詢資料](connect-query-ssms.md)。
- 如需 SQL Server 客戶諮詢小組部落格中有關使用 BACPAC 檔案進行移轉的主題，請參閱[使用 BACPAC 檔案從 SQL Server 移轉至 Azure SQL Database](https://techcommunity.microsoft.com/t5/DataCAT/Migrating-from-SQL-Server-to-Azure-SQL-Database-using-Bacpac/ba-p/305407)。
- 如需有關整個 SQL Server 資料庫移轉程序的討論，包括效能建議，請參閱[將 SQL Server 資料庫移轉至 Azure SQL Database](migrate-to-database-from-sql-server.md)。
- 若要了解如何管理和共用儲存體金鑰，以及安全地共用存取簽章，請參閱 [Azure 儲存體安全性指南](../../storage/blobs/security-recommendations.md)。
