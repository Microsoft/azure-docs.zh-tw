---
title: 針對 Azure SQL 資料倉儲問題進行疑難排解 | Microsoft Docs
description: 針對 Azure SQL 資料倉儲問題進行疑難排解。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 04/17/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 3500754c7e9cb14ea86e9c0e562ec5f98fc1fc94
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44377763"
---
# <a name="troubleshooting-azure-sql-data-warehouse"></a>針對 Azure SQL 資料倉儲問題進行疑難排解
本文列出常見的疑難排解問題。

## <a name="connecting"></a>連接
| 問題 | 解決方案 |
|:--- |:--- |
| 使用者 'NT AUTHORITY\ANONYMOUS LOGON' 登入失敗。 (Microsoft SQL Server，錯誤：18456) |當 AAD 使用者嘗試連線到主要資料庫，但主要資料庫中沒有使用者時，就會發生此錯誤。  若要修正此問題，請在連線時間指定您需要連線的 SQL 資料倉儲，或將使用者新增到 master 資料庫。  如需詳細資訊，請參閱[安全性概觀][Security overview]一文。 |
| 伺服器主體 "MyUserName" 在目前的資訊安全內容下無法存取「主要」資料庫。 無法開啟使用者預設資料庫。 登入失敗。 使用者 'MyUserName' 登入失敗。 (Microsoft SQL Server，錯誤：916) |當 AAD 使用者嘗試連線到主要資料庫，但主要資料庫中沒有使用者時，就會發生此錯誤。  若要修正此問題，請在連線時間指定您需要連線的 SQL 資料倉儲，或將使用者新增到 master 資料庫。  如需詳細資訊，請參閱[安全性概觀][Security overview]一文。 |
| CTAIP 錯誤 |若已在 SQL Server Master 資料庫上建立登入，但未在 SQL 資料倉儲資料庫上建立，則會發生這個錯誤。  如果您遇到這個錯誤，請查看[安全性概觀][Security overview]一文。  本文說明如何在 master 資料庫上建立登入和使用者，接著如何在 SQL 資料倉儲資料庫上建立使用者。 |
| 遭到防火牆封鎖 |為確保只有已知的 IP 位址擁有資料庫的存取權限，Azure SQL 資料庫受到伺服器及資料庫層級的防火牆所保護。 防火牆預設將會受到保護，因此您在可以連線之前，必須明確啟用單一 IP 位址或位址範圍。  若要設定防火牆的存取，請遵循[佈建指示][Provisioning instructions]中[設定用戶端 IP 的伺服器防火牆存取][Configure server firewall access for your client IP]的步驟。 |
| 無法與工具或驅動程式連線 |SQL 資料倉儲建議使用 [SSMS][SSMS]、[適用於 Visual Studio 的 SSDT][SSDT for Visual Studio] 或 [sqlcmd][sqlcmd] 來查詢您的資料。 如需驅動程式和連接到 SQL 資料倉儲的詳細資訊，請參閱 [Azure SQL 資料倉儲的驅動程式][Drivers for Azure SQL Data Warehouse]和[連接到 Azure SQL 資料倉儲][Connect to Azure SQL Data Warehouse]文章。 |

## <a name="tools"></a>工具
| 問題 | 解決方案 |
|:--- |:--- |
| Visual Studio 物件總管中遺漏 AAD 使用者 |這是已知的問題。  解決方法是在 [sys.database_principals][sys.database_principals] 中檢視使用者。  若要深入了解使用 Azure Active Directory 與 SQL 資料倉儲，請參閱[適用於 Azure SQL 資料倉儲的驗證][Authentication to Azure SQL Data Warehouse]。 |
|手動撰寫指令碼、使用指令碼精靈，或透過 SSMS 連線很緩慢、無回應或產生錯誤| 請確定已在主要資料庫中建立使用者。 在指令碼選項中，也請確定引擎版本已設定為 [Microsoft Azure SQL 資料倉儲版本]，且引擎類型為 [Microsoft Azure SQL Database]。|

## <a name="performance"></a>效能
| 問題 | 解決方案 |
|:--- |:--- |
| 查詢效能疑難排解 |如果您正試著針對特定查詢進行疑難排解，請從[了解如何監視查詢][Learning how to monitor your queries]開始。 |
| 查詢效能和計劃不佳通常是因為遺漏統計資料 |效能不佳最常見的原因是缺乏資料表的統計資料。  如需有關如何建立統計資料，以及這對於效能為何重要，請參閱[維護資料表統計資料][Statistics]。 |
| 並行存取低落/排入佇列的查詢偏少 |為了瞭解如何平衡記憶體配置與並行存取，必須先了解[工作負載管理][Workload management]。 |
| 如何實作最佳作法 |[SQL 資料倉儲最佳作法][SQL Data Warehouse best practices]一文是學習改善查詢效能的最佳起點。 |
| 如何透過調整來提升效能 |有時，只要[調整您的 SQL 資料倉儲][Scaling your SQL Data Warehouse]來提升查詢的計算能力，即可改善效能。 |
| 索引品質不佳導致查詢效能不佳 |有時，查詢會因為[資料行存放區索引品質不佳][Poor columnstore index quality]而變慢。  請參閱這篇文章，以取得詳細資訊及如何[重建索引以提升區段品質][Rebuild indexes to improve segment quality]。 |

## <a name="system-management"></a>系統管理
| 問題 | 解決方案 |
|:--- |:--- |
| 訊息 40847：無法執行這項作業，因為伺服器可能會超過允許的資料庫交易單位配額 45000。 |減少您正在嘗試建立的資料庫 [DWU][DWU]，或是[要求增加配額][request a quota increase]。 |
| 調查空間使用量 |請參閱[資料表大小][Table sizes]，以了解您系統的空間使用量。 |
| 協助管理資料表 |請參閱[資料表概觀][Overview]一文，以協助管理您的資料表。  本文還包含更詳細主題的連結，例如[資料表的資料類型][Data types]、[散發資料表][Distribute]、[編製資料表的索引][Index]、[分割資料表][Partition]、[維護資料表統計資料][Statistics]和[暫存資料表][Temporary]。 |
|Azure 入口網站中的透明資料加密 (TDE) 進度列不會更新|您可以透過 [PowerShell (英文)](/powershell/module/azurerm.sql/get-azurermsqldatabasetransparentdataencryption) 檢視 TDE 的狀態。|

## <a name="polybase"></a>Polybase
| 問題 | 解決方案 |
|:--- |:--- |
| 因為資料列太大而載入失敗 |目前，Polybase 不支援大型的資料列。  這表示如果資料表包含 VARCHAR(MAX)、NVARCHAR(MAX) 或 VARBINARY(MAX)，則無法使用外部資料表來載入您的資料。  目前，只有透過 Azure Data Factory (含 BCP) Azure 串流分析、SSIS、BCP 或 .NET SQLBulkCopy 類別，才支援載入大型資料列。 未來版本將增加讓 PolyBase 支援大型資料列。 |
| 使用 bcp 載入含有 MAX 資料類型的資料表失敗 |已知的問題是在某些情況下，VARCHAR(MAX)、NVARCHAR(MAX) 或 VARBINARY(MAX) 必須放在資料表結尾。  請嘗試將您的 MAX 資料行移到資料表的結尾。 |

## <a name="differences-from-sql-database"></a>與 SQL Database 不同之處
| 問題 | 解決方案 |
|:--- |:--- |
| 不支援的 SQL Database 功能 |請參閱[不支援的資料表功能][Unsupported table features]。 |
| 不支援的 SQL Database 資料類型 |請參閱[不支援的資料類型][Unsupported data types]。 |
| DELETE 和 UPDATE 限制 |請參閱 [UPDATE 因應措施][UPDATE workarounds]、[DELETE 因應措施][DELETE workarounds]和[使用 CTAS 來解決不支援 UPDATE 和 DELETE 語法的問題][Using CTAS to work around unsupported UPDATE and DELETE syntax]。 |
| 不支援 MERGE 陳述式 |請參閱 [MERGE 因應措施][MERGE workarounds]。 |
| 預存程序限制 |請參閱[預存程序限制][Stored procedure limitations]，以了解預存程序的一些限制。 |
| UDF 不支援 SELECT 陳述式 |這是 UDF 目前的限制。  關於我們支援的語法，請參閱 [CREATE FUNCTION][CREATE FUNCTION]。 |

## <a name="next-steps"></a>後續步驟
如需更多尋找問題解決方案的協助，以下是您可以嘗試的其他資源。

* [部落格]
* [功能要求]
* [影片]
* [CAT 小組部落格]
* [建立支援票證]
* [MSDN 論壇]
* [Stack Overflow 論壇]
* [Twitter]

<!--Image references-->

<!--Article references-->
[Security overview]: sql-data-warehouse-overview-manage-security.md
[SSMS]: /sql/ssms/download-sql-server-management-studio-ssms
[SSDT for Visual Studio]: sql-data-warehouse-install-visual-studio.md
[Drivers for Azure SQL Data Warehouse]: sql-data-warehouse-connection-strings.md
[Connect to Azure SQL Data Warehouse]: sql-data-warehouse-connect-overview.md
[建立支援票證]: sql-data-warehouse-get-started-create-support-ticket.md
[Scaling your SQL Data Warehouse]: sql-data-warehouse-manage-compute-overview.md
[DWU]: sql-data-warehouse-overview-what-is.md
[request a quota increase]: sql-data-warehouse-get-started-create-support-ticket.md
[Learning how to monitor your queries]: sql-data-warehouse-manage-monitor.md
[Provisioning instructions]: sql-data-warehouse-get-started-provision.md
[Configure server firewall access for your client IP]: sql-data-warehouse-get-started-provision.md
[SQL Data Warehouse best practices]: sql-data-warehouse-best-practices.md
[Table sizes]: sql-data-warehouse-tables-overview.md#table-size-queries
[Unsupported table features]: sql-data-warehouse-tables-overview.md#unsupported-table-features
[Unsupported data types]: sql-data-warehouse-tables-data-types.md#unsupported-data-types
[Overview]: sql-data-warehouse-tables-overview.md
[Data types]: sql-data-warehouse-tables-data-types.md
[Distribute]: sql-data-warehouse-tables-distribute.md
[Index]: sql-data-warehouse-tables-index.md
[Partition]: sql-data-warehouse-tables-partition.md
[Statistics]: sql-data-warehouse-tables-statistics.md
[Temporary]: sql-data-warehouse-tables-temporary.md
[Poor columnstore index quality]: sql-data-warehouse-tables-index.md#causes-of-poor-columnstore-index-quality
[Rebuild indexes to improve segment quality]: sql-data-warehouse-tables-index.md#rebuilding-indexes-to-improve-segment-quality
[Workload management]: resource-classes-for-workload-management.md
[Using CTAS to work around unsupported UPDATE and DELETE syntax]: sql-data-warehouse-develop-ctas.md#using-ctas-to-work-around-unsupported-features
[UPDATE workarounds]: sql-data-warehouse-develop-ctas.md#ansi-join-replacement-for-update-statements
[DELETE workarounds]: sql-data-warehouse-develop-ctas.md#ansi-join-replacement-for-delete-statements
[MERGE workarounds]: sql-data-warehouse-develop-ctas.md#replace-merge-statements
[Stored procedure limitations]: sql-data-warehouse-develop-stored-procedures.md#limitations
[Authentication to Azure SQL Data Warehouse]: sql-data-warehouse-authentication.md


<!--MSDN references-->
[sys.database_principals]: /sql/relational-databases/system-catalog-views/sys-database-principals-transact-sql
[CREATE FUNCTION]: /sql/t-sql/statements/create-function-sql-data-warehouse
[sqlcmd]: sql-data-warehouse-get-started-connect-sqlcmd.md

<!--Other Web references-->
[部落格]: https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/
[CAT 小組部落格]: https://blogs.msdn.microsoft.com/sqlcat/tag/sql-dw/
[功能要求]: https://feedback.azure.com/forums/307516-sql-data-warehouse
[MSDN 論壇]: https://social.msdn.microsoft.com/Forums/home?forum=AzureSQLDataWarehouse
[Stack Overflow 論壇]: http://stackoverflow.com/questions/tagged/azure-sqldw
[Twitter]: https://twitter.com/hashtag/SQLDW
[影片]: https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse
