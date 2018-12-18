---
title: 適用於 MariaDB 的 Azure 資料庫的定價層
description: 本文會說明適用於 MariaDB 的 Azure 資料庫的定價層。
author: jan-eng
ms.author: janeng
editor: jasonwhowell
services: mariadb
ms.service: mariadb
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 92db6442352242d5c7f25d39442d208d6007621b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984329"
---
# <a name="azure-database-for-mariadb-pricing-tiers"></a>適用於 MariaDB 的 Azure 資料庫定價層

適用於 MariaDB 的 Azure 資料庫伺服器可建立於三個不同定價層之一 (基本、一般用途和記憶體最佳化)。 定價層的差別在於虛擬核心中可佈建的計算數量、每個虛擬核心的記憶體，以及用來儲存資料的儲存體技術。 所有資源都會佈建在 MariaDB 伺服器層級。 一個伺服器可以有一個或多個資料庫。

|    | **基本** | **一般用途** | **記憶體最佳化** |
|:---|:----------|:--------------------|:---------------------|
| 計算世代 | <!--Gen 4,--> 第 5 代 |<!--Gen 4,-->第 5 代 |<!--Gen 4,--> 第 5 代 |
| 虛擬核心 | 1、2 | 2、4、8、16、32 |2、4、8、16 |
| 每個虛擬核心的記憶體 | 2 GB | 5 GB | 10 GB |
| 儲存體大小 | 5 GB 至 1 TB | 5 GB 至 4 TB | 5 GB 至 4 TB |
| 儲存體類型 | Azure 標準儲存體 | Azure 進階儲存體 | Azure 進階儲存體 |
| 資料庫備份的保留期限 | 7 至 35 天 | 7 至 35 天 | 7 至 35 天 |

若要選擇定價層，請從下表著手。

| 定價層 | 目標工作負載 |
|:-------------|:-----------------|
| 基本 | 需要輕量計算和 I/O 效能的工作負載。 範例包括用於開發或測試的伺服器，或者不常使用的小規模應用程式。 |
| 一般用途 | 需要平衡的計算和記憶體以及可擴充 I/O 輸送量的大多數商務工作負載。 範例包括用於裝載 Web 和行動應用程式的伺服器，以及其他企業應用程式。|
| 記憶體最佳化 | 需要記憶體內效能來提供更快速交易處理和更高並行性的高效能資料庫工作負載。 範例包括用於處理即時資料的伺服器，以及高效能交易式或分析應用程式。|

建立伺服器之後，虛擬核心數目和定價層 (基本層的來回除外) 可在幾秒內上下變動。 您可以也單獨調高儲存體的數量及延長或縮短備份保留期限，而無須中斷應用程式。 但您無法在建立伺服器之後，變更備份儲存體類型。 如需詳細資訊，請參閱[調整資源](#scale-resources)一節。

## <a name="compute-generations-and-vcores"></a>計算世代和虛擬核心

計算資源會以虛擬核心的形式提供，虛擬核心代表了基礎硬體的邏輯 CPU。<!--Currently, you can choose from two compute generations, Gen 4 and Gen 5. Gen 4 logical CPUs are based on Intel E5-2673 v3 (Haswell) 2.4-GHz processors.--> 「第 5 代」邏輯 CPU 是以 Intel E5-2673 v4 (Broadwell) 2.3-GHz 處理器為基礎。

<!--
| **Azure region** | **Gen 5** |
|:---|:----------:|:--------------------:|
| Central US | X | X |
| East US | X | X |
| East US 2 | X | X |
| North Central US | X | X |
| South Central US | X | X |
| West US | X | X |
| West US 2 |  | X |
| Canada Central | X | X |
| Canada East | X | X |
| Brazil South | X | X |
| North Europe | X | X |
| West Europe |  | X |
| France Central |  | X |
| UK West |  | X |
| UK South |  | X |
| East Asia | X | X |
| Southeast Asia | X | X |
| Australia East |  | X |
| Australia Central |  | X |
| Australia Central 2 |  | X |
| Australia Southeast |  | X |
| Central India | X | X |
| West India | X | X |
| South India |  | X |
| Japan East | X | X |
| Japan West | X | X |
| Korea Central |  | X |
| Korea South |  | X |
-->

## <a name="storage"></a>儲存體

您佈建的儲存體是「適用於 MariaDB 的 Azure 資料庫」伺服器可用的儲存體容量。 儲存體會用於資料庫檔案、暫存檔案、交易記錄和 MariaDB 伺服器記錄。 您佈建的儲存體總數也會定義您伺服器可用的 I/O 容量。

|    | **基本** | **一般用途** | **記憶體最佳化** |
|:---|:----------|:--------------------|:---------------------|
| 儲存體類型 | Azure 標準儲存體 | Azure 進階儲存體 | Azure 進階儲存體 |
| 儲存體大小 | 5 GB 至 1 TB | 5 GB 至 4 TB | 5 GB 至 4 TB |
| 儲存體遞增大小 | 1 GB | 1 GB | 1 GB |
| IOPS | 變數 |3 IOPS/GB<br/>最小值為 100 IOPS<br/>最大值為 6000 IOPS | 3 IOPS/GB<br/>最小值為 100 IOPS<br/>最大值為 6000 IOPS |

您可以在建立伺服器期間或之後，新增額外的儲存體容量。 基本層不提供 IOPS 保證。 在一般用途和記憶體最佳化定價層中，IOPS 與佈建的儲存體大小會以 3:1 的比例調整。

您可以在 Azure 入口網站或使用 Azure CLI 命令來監視 I/O 耗用量。 要監視的相關計量包括儲存體限制、儲存體百分比、已使用的儲存體和 IO 百分比。
<!--[storage limit, storage percentage, storage used, and IO percent](concepts-monitoring.md)-->

### <a name="reaching-the-storage-limit"></a>到達儲存體限制

當可用的儲存體數量低於 5 GB 或 5% 的佈建儲存體時 (以較低者為準)，伺服器會標記為唯讀狀態。 例如，如果您已佈建 100 GB 的儲存體，並且實際的使用率超過 95 GB，伺服器會標示為唯讀。 或者，如果您已佈建 5 GB 的儲存體，則當可用儲存體小於 250 MB時，伺服器會標示為唯讀。  

當服務嘗試讓伺服器變為唯讀時，會封鎖所有新的寫入交易要求，而現有的使用中交易會繼續執行。 當伺服器設為唯讀時，所有後續的寫入作業和交易認可都會失敗。 讀取查詢將會繼續運作，不會中斷。 當您增加佈建的儲存體之後，伺服器就可以再次接受寫入交易。

我們建議您設定警示，讓系統可在伺服器儲存容量接近閾值時發出通知，以避免進入唯讀狀態。 <!--For more information, see the documentation on [how to set up an alert](howto-alert-on-metric.md).-->

## <a name="backup"></a>Backup 

服務會自動採用伺服器的備份。 備份的最小保留期限是七天。 您可以設定的保留期限最多為 35 天。 在伺服器的存留期期間，您可以在任何時間點調整保留期限。 您可以選擇本地備援和異地備援備份。 異地備援備份也會儲存在您伺服器所在建立區域的[地理配對區域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)中。 此備援能力可在發生災害時提供一層保護。 您也可讓伺服器還原到其他任何 Azure 區域，只要其中的服務可使用異地備援備份。 建立伺服器之後，便無法在兩個備份儲存體選項之間做變更。

## <a name="scale-resources"></a>調整資源

建立伺服器之後，您可以單獨變更虛擬核心、<!--the hardware generation,-->定價層 (基本層的來回除外)、儲存體數量及備份保留期限。 但您無法在建立伺服器之後，變更備份儲存體類型。 虛擬核心數目可相應增加或減少。 備份保留期可在 7 到 35 天的範圍內相應增加或減少。 儲存體大小只能增加。 您可以透過入口網站或 Azure CLI 來調整資源。 <!--For an example of scaling by using Azure CLI, see [Monitor and scale an Azure Database for MariaDB server by using Azure CLI](scripts/sample-scale-server.md).-->

當您變更虛擬核心數目、<!--the hardware generation,-->或定價層時，系統會以新的計算配置建立一個原始伺服器複本。 當新伺服器已啟動並執行之後，連線就會切換到新的伺服器。 在系統切換到新伺服器的期間，您無法建立任何新的連線，且所有未認可的交易皆會復原。 此期間長短可能有所不同，但大部分情況下是少於一分鐘。

調整儲存體和變更備份保留期限完全是線上作業。 運作不會中斷，您的應用程式也不會受到影響。 由於 IOPS 會隨著佈建的儲存體大小進行調整，您可以透過相應增加儲存體來增加伺服器可用的 IOPS。

## <a name="pricing"></a>價格

如需最新的定價資訊，請參閱服務的[定價頁面](https://azure.microsoft.com/pricing/details/mariadb/)。 若要查看您所需的設定成本，[Azure 入口網站](https://portal.azure.com/#create/Microsoft.MariaDBServer)會根據您選取的選項，在 [定價層] 索引標籤中顯示每月成本。 如果您沒有 Azure 訂用帳戶，則可以使用 Azure 價格計算機來取得估計的價格。 在 [Azure 價格計算機](https://azure.microsoft.com/pricing/calculator/)網站上選取 [新增項目]，展開 [資料庫] 類別，並選擇 [適用於 MariaDB 的 Azure 資料庫] 以自訂選項。

## <a name="next-steps"></a>後續步驟
- 了解[服務限制](concepts-limits.md)。
- 了解如何[在 Azure 入口網站中建立 MariaDB 伺服器](quickstart-create-mariadb-server-database-using-azure-portal.md)。

<!--
- Learn how to [monitor and scale an Azure Database for MariaDB server by using Azure CLI](scripts/sample-scale-server.md).-->