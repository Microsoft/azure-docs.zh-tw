---
title: 複寫至 Azure SQL Database | Microsoft Docs
description: 了解如何搭配 Azure SQL Database 單一資料庫和彈性集區中的資料庫使用 SQL Server 複寫
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: allenwux
ms.author: xiwu
ms.reviewer: mathoma
manager: craigg
ms.date: 09/25/2018
ms.openlocfilehash: 360aa53a02beabccc685df97ad8549dfb88ff5c8
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47228140"
---
# <a name="replication-to-sql-database-single-and-pooled-databases"></a>複寫至 SQL Database 單一資料庫與集區資料庫

可以將 SQL Server 複寫設定為 Azure SQL Database 中[邏輯伺服器](sql-database-logical-servers.md)上的單一資料庫與集區資料庫。  
 
 ### <a name="supported-configurations"></a>**支援的設定：**  
 -  SQL Server 可以是在內部部署執行的 SQL Server 執行個體，或是在雲端 Azure 虛擬機器中執行的 SQL Server 執行個體。 如需詳細資訊，請參閱 [Azure 虛擬機器上的 SQL Server 概觀](https://azure.microsoft.com/documentation/articles/virtual-machines-sql-server-infrastructure-services/)。  
 - Azure SQL Database 必須是 SQL Server 發行者的發送訂閱者。  
 -  散發資料庫和複寫代理程式不能置於 Azure SQL Database 上。  
 - 支援快照集和單向異動複寫。 不支援點對點異動複寫和合併式複寫。
 - 複寫可在 Azure SQL Database 受控執行個體的公開預覽版上使用。 受控執行個體可以裝載發行者、散發者和訂閱者資料庫。 如需詳細資訊，請參閱[使用 SQL Database 受控執行個體進行複寫](replication-with-sql-database-managed-instance.md)。
 
## <a name="versions"></a>版本  
- 發行者與散發者必須至少是下列其中一個版本：  
 - SQL Server 2017 (14.x) 
 - SQL Server 2016 (13.x) 
 - SQL Server 2014 (12.x) SP1 CU3 
 - SQL Server 2014 (12.x) RTM CU10 
 - SQL Server 2012 (11.x) SP2 CU8 or SP3 
   
- 嘗試使用舊版設定複寫可能會導致錯誤號碼 MSSQL_REPL20084 (處理序無法連線到訂閱者。) 和 MSSQL_REPL40532 (無法開啟登入所要求的公開伺服器 \<名稱>。 登入失敗。)。  
- 若要使用 Azure SQL Database 的所有功能，您必須使用最新版的 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) 和 [SQL Server Data Tools](https://docs.microsoft.com//sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-2017)。  
   
 ## <a name="remarks"></a>備註  
- 若要設定複寫，請使用 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) 或在發行者上執行 Transact-SQL 陳述式。 您無法使用 Azure 入口網站來設定複寫。  
- 複寫只能使用 SQL Server 驗證登入來連線到 Azure SQL Database。 
- 複寫的資料表都必須有主索引鍵。  
- 您必須具有現有的 Azure 訂用帳戶。  
- Azure SQL Database 訂閱者可以位在任何區域中。  
- SQL Server 上的單一發行集可同時支援 Azure SQL Database 和 SQL Server (內部部署和 Azure 虛擬機器中的 SQL Server) 訂閱者。  
- 複寫管理、監視和疑難排解都必須從內部部署 SQL Server 執行。  
- 僅支援對 Azure SQL Database 的發送訂閱。  
- 在 SQL Database 的 **sp_addsubscription** 中僅支援 `@subscriber_type = 0`。  
- Azure SQL Database 不支援雙向、即時、可更新或點對點複寫。      
   
 ## <a name="replication-architecture"></a>複寫架構  
 ![replication-to-sql-database](./media/replication-to-sql-database/replication-to-sql-database.png)  
   
 ## <a name="scenarios"></a>案例  
   
 ### <a name="typical-replication-scenario"></a>一般複寫案例  
   
 1.  在內部部署 SQL Server 資料庫上建立異動複寫發行集。  
   
 2.  在內部部署 SQL Server 上使用 [新增訂閱精靈] 或 Transact-SQL 陳述式，來建立對 Azure SQL Database 的發送訂閱。  
   
 3.  初始資料集通常是由快照集代理程式建立且由散發代理程式散發並套用的快照集。 初始資料集也可透過備份或其他方式 (例如 SQL Server Integration Services) 提供。  
   
 ### <a name="data-migration-scenario"></a>資料移轉案例  
   
 1.  若要將資料從內部部署 SQL Server 資料庫複寫到 Azure SQL Database 中，請使用異動複寫。  
   
 2.  重新導向用戶端或中介層應用程式來更新 Azure SQL Database 複本。  
   
 3.  停止更新 SQL Server 版本的資料表，並移除發行集。  
   
## <a name="limitations"></a>限制  
Azure SQL Database 訂閱不支援下列選項：  
-   複製檔案群組關聯  
-   複製資料表資料分割配置  
-   複製索引資料分割配置  
-   複製使用者定義的統計資料  
-   複製預設繫結  
-   複製規則繫結  
-   複製全文檢索索引  
-   複製 XML XSD  
-   複製 XML 索引  
-   複製權限  
-   複製空間索引  
-   複製篩選的索引  
-   複製資料壓縮屬性  
-   複製疏鬆資料行屬性  
-   將 filestream 轉換為 MAX 資料類型  
-   將 hierarchyid 轉換為 MAX 資料類型  
-   將 spatial 轉換為 MAX 資料類型  
-   複製擴充屬性  
-   複製權限  
   
### <a name="limitations-to-be-determined"></a>待決定的限制 
-   複製定序  
-   SP 的序列式交易執行  
   
## <a name="examples"></a>範例  
建立發行集和發送訂閱。 如需詳細資訊，請參閱  
-   [建立發行集](https://docs.microsoft.com/sql/relational-databases/replication/publish/create-a-publication)  
   
-   [建立發送訂閱](https://docs.microsoft.com/sql/relational-databases/replication/create-a-push-subscription/create-a-push-subscription)，方法是使用 Azure SQL Database 邏輯伺服器名稱作為訂閱者 (例如 **N'azuresqldbdns.database.windows.net'**)，並使用 Azure SQL Database 名稱作為目的地資料庫 (例如 **AdventureWorks**)。  
   
 ## <a name="see-also"></a>另請參閱  
 - [建立發行集](https://docs.microsoft.com/sql/relational-databases/replication/publish/create-a-publication)   
 - [建立發送訂閱](https://docs.microsoft.com/sql/relational-databases/replication/create-a-push-subscription/create-a-push-subscription)   
 - [複寫的類型](https://docs.microsoft.com/sql/relational-databases/replication/types-of-replication)   
 - [監視 &#40;複寫&#41;](https://docs.microsoft.com/sql/relational-databases/replication/monitor/monitoring-replication)   
 - [初始化訂閱](https://docs.microsoft.com/sql/relational-databases/replication/initialize-a-subscription)  
