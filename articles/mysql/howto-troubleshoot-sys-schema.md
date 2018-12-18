---
title: 如何使用 sys_schema 在適用於 MySQL 的 Azure 資料庫中進行效能微調和資料庫維護
description: 本文說明如何使用 sys_schema 在適用於 MySQL 的 Azure 資料庫中找出效能問題及維護資料庫。
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.topic: article
ms.date: 08/01/2018
ms.openlocfilehash: 1e10e3b1b5f4518732408f254eb5767acb8485c6
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39446902"
---
# <a name="how-to-use-sysschema-for-performance-tuning-and-database-maintenance-in-azure-database-for-mysql"></a>如何使用 sys_schema 在適用於 MySQL 的 Azure 資料庫中進行效能微調和資料庫維護

最先在 MySQL 5.5 中導入的 MySQL performance_schema，能針對許多重要伺服器資源 (例如記憶體配置、預存程式、中繼資料鎖定等) 提供檢測功能。不過，performance_schema 包含超過 80 個資料表，且通常需要聯結 performance_schema 內的資料表以及 information_schema 中的資料表，才能取得所需的資訊。 sys_schema 是以 performance_schema 與 information_schema 為基礎而建置的，它於唯讀資料庫中提供功能強大的[易用檢視](https://dev.mysql.com/doc/refman/5.7/en/sys-schema-views.html) \(英文\) 集合，並已於適用於 MySQL 的 Azure 資料庫 5.7 版中完整啟用。

![sys_schema 的檢視](./media/howto-troubleshoot-sys-schema/sys-schema-views.png)

sys_schema 中有 52 個檢視，每個檢視分別具有下列其中一個前置詞：

- Host_summary 或 IO：I/O 相關的延遲。
- InnoDB：InnoDB 緩衝區狀態和鎖定。
- Memory：依主機和使用者分類的記憶體使用量。
- Schema：結構描述相關的資訊，例如自動增量、索引等等。
- Statement：SQL 陳述式相關的資訊；這可以是導致完整資料表掃描或長時間查詢的陳述式。
- User：使用者所耗用並依使用者分類的資源。 範例包括檔案 I/O、連線和記憶體。
- Wait：依主機或使用者分類的等候事件。

現在，讓我們看看 sys_schema 的一些常見使用模式。 首先，我們會將使用模式分成兩個類別：**效能微調**和**資料庫維護**。

## <a name="performance-tuning"></a>效能微調

### <a name="sysusersummarybyfileio"></a>*sys.user_summary_by_file_io*

IO 是資料庫中成本最高的作業。 我們可以藉由查詢 *sys.user_summary_by_file_io* 檢視來找出平均 IO 延遲。 使用 125 GB 的預設佈建儲存體時，我的 IO 延遲大約是 15 秒。

![IO 延遲：125 GB](./media/howto-troubleshoot-sys-schema/io-latency-125GB.png)

由於適用於 MySQL 的 Azure 資料庫會針對儲存體調整 IO，因此將我已佈建的儲存體增加到 1 TB 時，我的 IO 延遲會減少為 571 毫秒。

![IO 延遲：1TB](./media/howto-troubleshoot-sys-schema/io-latency-1TB.png)

### <a name="sysschematableswithfulltablescans"></a>*sys.schema_tables_with_full_table_scans*

儘管經過仔細規劃，許多查詢仍可能導致完整資料表掃描。 如需關於索引類型和如何對它們加以最佳化的詳細資訊，您可以參閱這篇文章：[如何對查詢效能進行疑難排解](./howto-troubleshoot-query-performance.md)。 完整資料表掃描會耗用大量資源，並降低資料庫效能。 透過完整資料表掃描來尋找資料表的最快方式是查詢 *sys.schema_tables_with_full_table_scans* 檢視。

![完整資料表掃描](./media/howto-troubleshoot-sys-schema/full-table-scans.png)

### <a name="sysusersummarybystatementtype"></a>*sys.user_summary_by_statement_type*

對資料庫效能問題進行疑難排解時，識別您的資料庫內發生的事件可能會有幫助，而使用 *sys.user_summary_by_statement_type* 檢視可能可以協助做到這一點。

![依陳述式分類的摘要](./media/howto-troubleshoot-sys-schema/summary-by-statement.png)

在此範例中，適用於 MySQL 的 Azure 資料庫花費了 53 分鐘將 slog 查詢記錄排清 44579 次。 這不僅耗時，也需要許多 IO。 您可以藉由停用慢速查詢記錄，或降低慢速查詢記錄的頻率，在 Azure 入口網站中減少這項活動。

## <a name="database-maintenance"></a>資料庫維護

### <a name="sysinnodbbufferstatsbytable"></a>*sys.innodb_buffer_stats_by_table*

InnoDB 緩衝集區存在於記憶體中，是 DBMS 與儲存體之間的主要快取機制。 InnoDB 緩衝集區的大小會繫結至效能層，除非選擇不同的產品 SKU，否則無法變更。 如同作業系統中的記憶體，系統會移出舊的頁面以騰出空間給最新的資料。 若要了解哪些資料表耗用了大部分的 InnoDB 緩衝集區記憶體，您可以查詢 *sys.innodb_buffer_stats_by_table* 檢視。

![InnoDB 緩衝區狀態](./media/howto-troubleshoot-sys-schema/innodb-buffer-status.png)

從上圖可以明顯看出，除了系統資料表和檢視以外，mysqldatabase033 資料庫 (其裝載其中一個「我的 WordPress」網站) 中的每個資料表都在記憶體中佔用了 16 KB (或 1 頁) 的資料。

### <a name="sysschemaunusedindexes--sysschemaredundantindexes"></a>*Sys.schema_unused_indexes* & *sys.schema_redundant_indexes*

索引是提升讀取效能的理想工具，但它們會造成插入和儲存方面的額外成本。 *Sys.schema_unused_indexes* 和 *sys.schema_redundant_indexes* 可為您提供未使用或重複索引的見解。

![未使用的索引](./media/howto-troubleshoot-sys-schema/unused-indexes.png)

![重複的索引](./media/howto-troubleshoot-sys-schema/redundant-indexes.png)

## <a name="conclusion"></a>結論

總而言之，sys_schema 對效能調整和資料庫維護而言，都是很理想的工具。 請務必在您適用於 MySQL 的 Azure 資料庫中充分運用這項功能。 

## <a name="next-steps"></a>後續步驟
- 若想知道是否有人可解答您最關切的問題，或是要張貼新問題/解答，請造訪 [MSDN 論壇](https://social.msdn.microsoft.com/forums/security/en-US/home?forum=AzureDatabaseforMySQL)或[堆疊溢位](https://stackoverflow.com/questions/tagged/azure-database-mysql)。
