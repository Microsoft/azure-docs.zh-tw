---
title: Azure 串流分析輸出至 Azure SQL Database
description: 了解從 Azure 串流分析將資料輸出至 Azure SQL Database，達到更高的寫入輸送速率。
services: stream-analytics
author: chetang
ms.author: chetang
manager: katicad
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 09/21/2018
ms.openlocfilehash: 623d03c96866392ef245fb924cbf6600e7850ffe
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47057705"
---
# <a name="azure-stream-analytics-output-to-azure-sql-database"></a>Azure 串流分析輸出至 Azure SQL Database

本文會討論使用 Azure 串流分析將資料載入 Azure SQL Dtabase 時，達到更佳寫入輸送效能的祕訣。

Azure 串流分析中的 SQL 輸出支援平行寫入作為選項。 此選項允許[完全平行](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-parallelization#embarrassingly-parallel-jobs)作業拓撲，可將多個輸出資料分割平行寫入目的地資料表。 但是，在 Azure 串流分析中啟用此選項可能仍不足以達到更高的輸送量，因為它相當倚賴您的 Azure SQL Database 資料庫設定及資料表結構描述。 索引、叢集索引鍵、索引填滿因數及壓縮的選擇會影響載入資料表所需的時間。 如需如何最佳化您 Azure SQL Database，以根據內部效能評定改善查詢及載入效能的詳細資訊，請參閱 [SQL 資料庫效能指導](https://docs.microsoft.com/azure/sql-database/sql-database-performance-guidance)。

以下是每一種服務中的某些設定，可協助改善您解決方案的整體輸送量。

## <a name="azure-stream-analytics"></a>Azure 串流分析

- **繼承資料分割** – 此 SQL 輸出設定選項可繼承您先前查詢步驟或輸入的資料分割配置。 啟用此選項時，寫入磁碟式資料表及針對您的作業採用[完全平行](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-parallelization#embarrassingly-parallel-jobs)拓撲，可預期取得更佳的輸送量。 此資料分割已會針對許多其他[輸出](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-parallelization#partitions-in-sources-and-sinks)自動進行。 使用此選項進行的大量插入 (Bulk Insert) 也會停用資料表鎖定 (TABLOCK)。

> [!NOTE] 
> 當輸入資料分割超過 8 個時，繼承輸入資料分割配置可能不會是適當的選擇。 此上限可在擁有單一識別欄位和叢集索引的資料表上觀察到。 根據您的結構描述和選擇的索引，您的觀察結果可能會有所不同。

- **批次大小** - SQL 輸出設定允許您根據目的地資料表/工作負載的性質，指定 Azure 串流分析 SQL 輸出中的批次大小上限。 批次大小是每一次大量插入交易中傳送的記錄數上限。 在叢集資料行存放區索引中，使用約 [100K](https://docs.microsoft.com/sql/relational-databases/indexes/columnstore-indexes-data-loading-guidance) 的批次大小可允許更多平行化、最小記錄及鎖定最佳化。 在磁碟式資料表中，10K (預設) 或更低的批次大小可能會是您解決方案的最佳選擇，因為更大的批次大小可能會在大量插入時觸發鎖定擴大。

- **輸入訊息調整** – 若您已使用繼承資料分割及批次大小進行最佳化，增加每個資料分割每個訊息的輸入事件數可更進一步協助提高寫入輸送量。 輸入訊息調整允許 Azure 串流分析中之批次大小提高到指定的批次大小，以改善輸送量。 這可透過使用[壓縮](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-define-inputs)或 Premium EventHub SKU 中更大的訊息大小來達成。

## <a name="sql-azure"></a>SQL Azure

- **資料分割資料表及索引** – 使用[資料分割](https://docs.microsoft.com/sql/relational-databases/partitions/partitioned-tables-and-indexes?view=sql-server-2017) SQL 資料表及資料表上具有與您分割區索引鍵相同資料行的資料分割索引 (例如 PartitionId) 可大幅減少寫入期間資料分割間的競爭。 針對資料分割資料表，您將需要在 PRIMARY 檔案群組上建立[資料分割函數](https://docs.microsoft.com/sql/t-sql/statements/create-partition-function-transact-sql?view=sql-server-2017)及[資料分割配置](https://docs.microsoft.com/sql/t-sql/statements/create-partition-scheme-transact-sql?view=sql-server-2017)。 這也會在載入新資料時，增加現有資料的可用性。 根據資料分割數可能會到達記錄 IO 限制，您可以透過升級 SKU 來提高此限制。

- **避免唯一索引鍵違規** – 若您在 Azure 串流分析活動記錄中收到[多重索引鍵違規警告訊息](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-common-troubleshooting-issues#handle-duplicate-records-in-azure-sql-database-output)，請確認您的作業並未受到復原案例期間容易發生的唯一條件約束違規影響。 這可透過在您的索引上設定 [IGNORE\_DUP\_KEY](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-common-troubleshooting-issues#handle-duplicate-records-in-azure-sql-database-output) 選項來避免。

## <a name="azure-data-factory-and-in-memory-tables"></a>Azure Data Factory 及記憶體內部資料表

- **記憶體內部資料表作為暫存資料表** – [記憶體內部資料表](https://docs.microsoft.com/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)允許非常高速的資料載入，但資料必須要能容納在記憶體中。 效能評定顯示從記憶體內部資料表大量載入磁碟式資料表的速度，大約是使用單一寫入器直接大量插入具有識別欄位和叢集索引磁碟式資料表的 10 倍。 若要利用此大量插入效能，請[使用 Azure Data Factory 設定複製作業](https://docs.microsoft.com/azure/data-factory/connector-azure-sql-database)，從記憶體內部資料表將資料複製到以磁碟為基礎的資料表。

## <a name="summary"></a>總結

總結來說，針對 SQL 輸出使用 Azure 串流分析中的資料分割輸出功能，利用 Azure SQL Database 中的資料分割資料表來對您的作業進行校準平行化，可大幅改善您的輸送量。 利用 Azure Data Factory 協調資料移動，將資料從記憶體內部資料表移動到以磁碟為基礎的資料表，可為您的輸送量帶來以指數增加的改善。 若可行的話，改善訊息密度也可以在改善整體輸送量中扮演重要角色。
