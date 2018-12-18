---
title: 解決移轉至 Azure SQL Database 期間的 Transact-SQL 差異 | Microsoft Docs
description: 在 Azure SQL Database 中未完整支援 Transact-SQL 陳述式
services: sql-database
ms.service: sql-database
ms.subservice: ''
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 09/14/2018
ms.openlocfilehash: dfff51d7541ffdc2d279b238a6d993d5e29515f0
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47160702"
---
# <a name="resolving-transact-sql-differences-during-migration-to-sql-database"></a>解決移轉至 SQL Database 期間的 Transact-SQL 差異   
[將您的資料庫從 SQL Server 移轉](sql-database-cloud-migrate.md)至 Azure SQL Server 時，您可能會發現您的資料庫必須先進行一些再造，才能移轉 SQL Server。 本文章提供一些指引，以協助您執行此再造作業，以及了解為何必須執行再造的基礎原因。 若要偵測不相容性，請使用 [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595)。

## <a name="overview"></a>概觀
Microsoft SQL Server 和 Azure SQL Database 都支援應用程式使用的大部分 Transact-SQL 功能。 例如，資料類型、運算子、字串、算術、邏輯及資料指標函式等核心 SQL 元件在 SQL Server 與 SQL Database 中都以相同的方式運作。 不過，DDL (資料定義語言) 和 DML (資料操作語言) 元素中有幾個 T-SQL 差異導致對 T-SQL 陳述式和查詢僅提供部分支援 (將在本文章中稍後探討)。

此外，還有一些功能和語法完全不受支援，因為 Azure SQL Database 的設計目的是要將功能隔離，讓它們不與 master 資料庫和作業系統相依。 因此，大多數伺服器層級活動都不適用於 SQL Database。 如果它們設定伺服器層級選項、作業系統元件或指定檔案系統組態，便無法使用 T-SQL 陳述式和選項。 當需要這類功能時，通常會從 SQL Database 或從另一個 Azure 功能或服務以其他方式提供適合的替代方案。 

例如，Azure 已內建高可用性，所以沒有必要設定「永遠開啟」(不過您可以設定作用中異地複寫以在發生災害時更快獲得復原)。 因此，SQL Database 不支援與可用性群組相關的 T-SQL 陳述式，也不支援與「永遠開啟」相關的動態管理檢視。

如需 SQL Database 所支援和不支援的功能清單，請參閱 [Azure SQL Database 功能比較](sql-database-features.md)。 此頁面上的清單可補充該指引和功能文章，並將焦點放在 Transact-SQL 陳述式。

## <a name="transact-sql-syntax-statements-with-partial-differences"></a>具有部分差異的 Transact-SQL 語法陳述式
核心 DDL (資料定義語言) 陳述式可供使用，但有些 DDL 陳述式具有與磁碟放置相關的擴充功能和不支援的功能。 

- CREATE 和 ALTER DATABASE 陳述式有超過三十多個選項。 這些陳述式包括僅適用於 SQL Server 的檔案放置、FILESTREAM 及服務訊息代理程式選項。 如果您是在移轉之前建立資料庫，這可能無關緊要，但如果您要移轉建立資料庫的 T-SQL 程式碼，您就應該將 [CREATE DATABASE (Azure SQL Database)](https://msdn.microsoft.com/library/dn268335.aspx) 與在 [CREATE DATABASE (SQL Server Transact-SQL)](https://msdn.microsoft.com/library/ms176061.aspx) 的 SQL Server 語法做比較，以確定系統支援您使用的所有選項。 Azure SQL Database 的 CREATE DATABASE 也有僅適用於 SQL Database 的服務目標和靈活擴充能力選項。
- CREATE 和 ALTER TABLE 陳述式具有無法在 SQL Database 上使用的 FileTable 選項，因為 FILESTREAM 不受支援。
- CREATE 和 ALTER 登入陳述式受支援，但 SQL Database 並未提供所有選項。 為了讓您的資料庫更具可攜性，SQL Database 建議您儘可能使用自主資料庫使用者，而不要使用登入。 如需詳細資訊，請參閱 [CREATE/ALTER LOGIN](https://msdn.microsoft.com/library/ms189828.aspx) 及[控制和授與資料庫存取權](https://docs.microsoft.com/azure/sql-database/sql-database-manage-logins)。

## <a name="transact-sql-syntax-not-supported-in-azure-sql-database"></a>在 Azure SQL Database 中不支援的 Transact-SQL 語法   
除了與 [Azure SQL Database 功能比較](sql-database-features.md)中所述不支援的功能相關之 Transact-SQL 陳述式以外，下列陳述式和陳述式群組也不受支援。 因此，如果要移轉的資料庫使用下列任何功能，請再造您的 T-SQL 以排除這些 T-SQL 功能和陳述式。

- 系統物件的定序
- 相關連接：端點陳述式。 SQL Database 不支援 Windows 驗證，但支援類似的 Azure Active Directory 驗證。 某些驗證類型需要最新的 SSMS 版本。 如需詳細資訊，請參閱 [使用 Azure Active Directory 驗證連線到 SQL Database 或 SQL 資料倉儲](sql-database-aad-authentication.md)。
- 使用三個或四個組件名稱跨資料庫查詢。 (使用[彈性資料庫查詢](sql-database-elastic-query-overview.md)支援跨資料庫唯讀查詢。)
- 跨資料庫擁有權鏈結，`TRUSTWORTHY` 設定
- `EXECUTE AS LOGIN` 請改用 'EXECUTE AS USER'。
- 除了可延伸金鑰管理之外還支援加密
- 事件服務：事件、事件通知、查詢通知
- 檔案放置：與資料庫檔案放置、大小及資料庫檔案 (由 Microsoft Azure 自動管理) 相關的語法。
- 高可用性：與透過 Microsoft Azure 帳戶管理的高可用性相關的語法。 這包括備份、還原、永遠開啟、資料庫鏡像、記錄傳送、修復模式的語法。
- 記錄讀取器：依賴 SQL Database 上不適用之記錄讀取器的語法：發送複寫、異動資料擷取。 SQL Database 可以是推送複寫文章的訂閱者。
- 函式：`fn_get_sql`、`fn_virtualfilestats`、`fn_virtualservernodes`
- 硬體：與硬體相關伺服器設定相關的語法：例如記憶體、背景工作執行緒、CPU 親和性、追蹤旗標。 改為使用服務層和計算大小。
- `KILL STATS JOB`
- `OPENQUERY`、`OPENROWSET`、`OPENDATASOURCE` 和四部分的名稱
- .NET Framework：與 SQL Server 整合的 CLR
- 語意搜尋
- 伺服器認證：請改用[資料庫範圍認證](https://msdn.microsoft.com/library/mt270260.aspx)。
- 伺服器層級項目：伺服器角色，`sys.login_token`。 雖然某些伺服器層級權限已由資料庫層級權限取代，但是無法使用伺服器層級權限的 `GRANT`、`REVOKE` 和 `DENY`。 一些有用的伺服器層級 DMV 有相同的資料庫層級 DMV。
- `SET REMOTE_PROC_TRANSACTIONS`
- `SHUTDOWN`
- `sp_addmessage`
- `sp_configure` 選項和 `RECONFIGURE`。 有些選項可透過 [變更資料庫範圍組態](https://msdn.microsoft.com/library/mt629158.aspx)來使用。
- `sp_helpuser`
- `sp_migrate_user_to_contained`
- SQL Server Agent：依賴 SQL Server Agent 或 MSDB 資料庫的語法︰警示、運算子、中央管理伺服器。 改用指令碼，例如 Azure PowerShell。
- SQL Server Audit：請改用 SQL Database 稽核。
- SQL Server 追蹤
- 追蹤旗標：有些追蹤旗標項目已移至相容性模式。
- Transact-SQL 偵錯
- 觸發程序：伺服器範圍或登入觸發程序
- `USE` 陳述式：若要將資料庫內容變更為不同的資料庫，您必須建立與新資料庫的連接。

## <a name="full-transact-sql-reference"></a>完整 Transact-SQL 參考
如需 Transact-SQL 文法、使用方式和範例的詳細資訊，請參閱《SQL Server 線上叢書》中的＜ [Transact-SQL 參考 (資料庫引擎)](https://msdn.microsoft.com/library/bb510741.aspx) ＞。 

### <a name="about-the-applies-to-tags"></a>關於「適用於」標記
Transact-SQL 參考包括從 SQL Server 版本 2008 到目前版本的相關文章。 文章標題下方是圖示列，列出四個 SQL Server 平台並指出適用性。 例如，可用性群組是在 SQL Server 2012 中導入。 [建立可用性群組](https://msdn.microsoft.com/library/ff878399.aspx)文章指出陳述式會套用至 **SQL Server (從 2012 年開始)**。 陳述式不適用於 SQL Server 2008、SQL Server 2008 R2、Azure SQL Database、Azure SQL 資料倉儲或平行資料倉儲。

在某些情況下，文章的一般主旨可用於產品中，但產品之間會有些微的差異。 依適當情況會在文章的中間點指出差異。 在某些情況下，文章的一般主旨可用於產品中，但產品之間會有些微的差異。 依適當情況會在文章的中間點指出差異。 例如，SQL Database 中有提供 CREATE TRIGGER 文章。 但伺服器層級觸發程序的 **ALL SERVER** 選項指出無法在 SQL Database 中使用伺服器層級的觸發程序。 請改用資料庫層級的觸發程序。

## <a name="next-steps"></a>後續步驟

如需 SQL Database 所支援和不支援的功能清單，請參閱 [Azure SQL Database 功能比較](sql-database-features.md)。 此頁面上的清單可補充該指引和功能文章，並將焦點放在 Transact-SQL 陳述式。

