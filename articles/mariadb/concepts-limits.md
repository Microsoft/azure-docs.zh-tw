---
title: 限制 - 適用於 MariaDB 的 Azure 資料庫
description: 本文說明適用於 MariaDB 的 Azure 資料庫有何限制，例如連線數量和儲存引擎選項。
author: savjani
ms.author: pariks
ms.service: jroth
ms.topic: conceptual
ms.date: 10/2/2020
ms.openlocfilehash: de561f0fdea7ea7085a4a1d3ec6f95071c36f57e
ms.sourcegitcommit: 52e3d220565c4059176742fcacc17e857c9cdd02
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98664294"
---
# <a name="limitations-in-azure-database-for-mariadb"></a>適用於 MariaDB 的 Azure 資料庫相關限制
下列各節說明資料庫服務中的容量、儲存引擎支援、權限支援、資料操作陳述式支援，以及功能限制。

## <a name="server-parameters"></a>伺服器參數

> [!NOTE]
> 如果您要尋找伺服器參數的最小值/最大值（如 `max_connections` 和 `innodb_buffer_pool_size` ），這些資訊已移至 **[伺服器參數](./concepts-server-parameters.md)** 文章。

適用於 MariaDB 的 Azure 資料庫支援調整伺服器參數的值。 某些參數的最小值和最大值 (例如。 `max_connections`、 `join_buffer_size` `query_cache_size`) 是由伺服器的定價層和虛擬核心所決定。 如需這些限制的詳細資訊，請參閱 [伺服器參數](./concepts-server-parameters.md) 。

在初始部署時，Azure for 適用于 mariadb 伺服器包含時區資訊的系統資料表，但不會填入這些資料表。 時區資料表可藉由從 MySQL 命令列或 MySQL Workbench 等工具呼叫 `mysql.az_load_timezone` 預存程序來填入。 請參閱 [Azure 入口網站](howto-server-parameters.md#working-with-the-time-zone-parameter)或 [Azure CLI](howto-configure-server-parameters-cli.md#working-with-the-time-zone-parameter) 文章，以了解如何呼叫預存程序，以及設定全域或工作階段層級的時區。

服務不支援密碼外掛程式，例如 "validate_password" 和 "caching_sha2_password"。

## <a name="storage-engine-support"></a>儲存引擎支援

### <a name="supported"></a>支援
- [InnoDB](https://mariadb.com/kb/en/library/xtradb-and-innodb/) \(英文\)
- [MEMORY](https://mariadb.com/kb/en/library/memory-storage-engine/) \(英文\)

### <a name="unsupported"></a>不支援
- [MyISAM](https://mariadb.com/kb/en/library/myisam-storage-engine/) \(英文\)
- [BLACKHOLE](https://mariadb.com/kb/en/library/blackhole/) \(英文\)
- [ARCHIVE](https://mariadb.com/kb/en/library/archive/) \(英文\)

## <a name="privileges--data-manipulation-support"></a>& 資料操作支援的許可權

許多伺服器參數與設定可能會不慎降低伺服器效能，或否定適用于 mariadb 伺服器的 ACID 屬性。 為了維護產品層級的服務完整性與 SLA，此服務不會公開多個角色。 

適用于 mariadb 服務不允許直接存取基礎檔案系統。 不支援某些資料操作命令。 

## <a name="privilege-support"></a>權限支援

### <a name="unsupported"></a>不支援

以下是不支援的：
- DBA 角色：受限制。 或者，您可以使用在新的伺服器建立期間建立的系統管理員使用者 () ，讓您可以執行大部分的 DDL 和 DML 語句。 
- 超級許可權：同樣地，也會限制 [超級許可權](https://mariadb.com/kb/en/library/grant/#global-privileges) 。
- 定義：需要進階的權限才能建立，而且受限制。 如果使用備份匯入資料，執行 mysqldump 時以手動方式或使用 `--skip-definer` 命令移除 `CREATE DEFINER` 命令。
- 系統資料庫： [mysql 系統資料庫](https://mariadb.com/kb/en/the-mysql-database-tables/) 是唯讀的，用來支援各種 PaaS 功能。 您無法對 `mysql` 系統資料庫進行變更。
- `SELECT ... INTO OUTFILE`：服務中不支援。

### <a name="supported"></a>支援
- 支援 `LOAD DATA INFILE`，但必須指定 `[LOCAL]` 參數並導向至 UNC 路徑 (透過 SMB 掛接的 Azure 儲存體)。

## <a name="functional-limitations"></a>功能限制：

### <a name="scale-operations"></a>調整作業
- 目前不支援基本定價層的雙向動態調整。
- 不支援減少伺服器儲存體大小。

### <a name="server-version-upgrades"></a>伺服器版本升級
- 目前不支援在主要資料庫引擎版本之間進行自動轉換。

### <a name="point-in-time-restore"></a>還原時間點
- 使用 PITR 功能時，所建立新伺服器的設定會與作為新伺服器基礎的伺服器設定相同。
- 不支援還原已刪除的伺服器。

### <a name="subscription-management"></a>訂用帳戶管理
- 目前不支援跨訂用帳戶和資源群組動態移動預先建立的伺服器。

### <a name="vnet-service-endpoints"></a>VNet 服務端點
- VNet 服務端點的支援僅適用於一般用途伺服器和記憶體最佳化伺服器。

### <a name="storage-size"></a>儲存體大小
- 如需每個定價層的儲存體大小限制，請參閱[定價層](concepts-pricing-tiers.md)。

## <a name="current-known-issues"></a>目前已知問題
- MariaDB 伺服器執行個體於建立連線後會顯示不正確的伺服器版本。 若要取得正確的伺服器執行個體引擎版本，請使用 `select version();` 命令。

## <a name="next-steps"></a>後續步驟
- [每個服務層級中可用的項目](concepts-pricing-tiers.md)
- [支援的 MariaDB 資料庫版本](concepts-supported-versions.md)
