---
title: 「適用於 MySQL 的 Azure 資料庫」中的讀取複本。
description: 此文將說明「適用於 MySQL 的 Azure 資料庫」的讀取複本。
services: mysql
author: ajlam
ms.author: andrela
editor: jasonwhowell
ms.service: mysql
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 283b529aa8c6431ea725b066c9b5cb3db19a929b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46969439"
---
# <a name="read-replicas-in-azure-database-for-mysql"></a>讀取「適用於 MySQL 的 Azure 資料庫」中的複本

讀取複本功能 (公開預覽) 可讓您將資料從「適用於 MySQL 的 Azure 資料庫伺服器」(主要) 複寫到相同 Azure 區域內的最多五部唯讀伺服器 (複本)。 唯讀複本會使用 MySQL 引擎的原生二進位記錄 (binlog) 檔案位置型複寫技術來進行非同步更新。 若要深入了解二進位記錄 (binlog) 複寫，請參閱 [MySQL 二進位記錄 (binlog) 複寫概觀](https://dev.mysql.com/doc/refman/5.7/en/binlog-replication-configuration-overview.html) \(英文\)。

在「適用於 MySQL 的 Azure 資料庫」服務中建立的複本是新的伺服器，而且可使用與正常/獨立 MySQL 伺服器一樣的方式來管理。 這些伺服器的費率與標準伺服器的費率相同。

若要深入了解 MySQL 複寫功能與問題，請參閱 [MySQL 複寫文件](https://dev.mysql.com/doc/refman/5.7/en/replication-features.html) \(英文\)。

## <a name="when-to-use-read-replicas"></a>何時應該使用讀取複本

會使用大量讀取資源的應用程式與工作負載可由唯讀複本來服務。 相較於只使用單一伺服器來為讀取與寫入要求提供服務，讀取複本有助於增加可用讀取容量。 讀取工作負載可隔離到複本，而寫入工作負載可以導向到主要伺服器。

常見的案例是讓 BI 與分析工作負載使用讀取複本做為報告的資料來源。

## <a name="considerations-and-limitations"></a>考量與限制

### <a name="pricing-tiers"></a>定價層

目前只有「一般用途」與「記憶體最佳化」定價層提供讀取複本。

### <a name="stopping-replication"></a>停止複寫

您可以選擇停止主要伺服器與複本伺服器之間的複寫。 停止複寫會移除主要伺服器與複本伺服器之間的複寫關聯性。

一旦停止複寫，複本伺服器就會成為獨立伺服器。 獨立伺服器中的資料是起始「停止複寫」命令之後，複本上可用的資料。 獨立伺服器未跟上主要伺服器。 此伺服器無法再次設定為複本。

### <a name="replicas-are-new-servers"></a>複本是新伺服器

複本會建立為新的「適用於 MySQL 的 Azure 資料庫」伺服器。 現有的伺服器無法設定為複本。

### <a name="replica-server-configuration"></a>複本伺服器設定

複本伺服器是使用與主要伺服器相同的伺服器設定所建立，包括下列設定：

- 定價層
- 計算世代
- 虛擬核心
- 儲存體
- 備份保留期限
- 備份備援選項
- MySQL 引擎版本

建立複本之後，您可以變更為與獨立伺服器不一樣定價層 (無法變更為基本或從基本變更為其他定價層)、計算世代、vCore、儲存體與備份保留。

### <a name="master-server-configuration"></a>主要伺服器設定

若主要伺服器設定 (例如 vCore 或儲存體) 已更新，複本的設定應該也會更新為相等或更大的值。 若沒有，複本伺服器可能無法同步主要伺服器上發生的變更，而且可能會因此當機。 

### <a name="deleting-the-master-server"></a>刪除主要伺服器

刪除主要伺服器時，會在所有讀取複本上停止複寫。 這些複本會成為獨立伺服器。 主要伺服器本身會被刪除。

### <a name="user-accounts"></a>使用者帳戶

系統會將主要伺服器上的使用者複寫到讀取複本。 您只能使用主要伺服器上可用的使用者帳戶來連線到讀取複本。

### <a name="other"></a>其他

- 不支援全域交易識別碼 (GTID)。
- 不支援建立複本的複本。
- 記憶體內資料表可能會導致複本不同步。這是 MySQL 複寫技術的限制。 如需詳細資訊，請參閱 [MySQL 參考文件](https://dev.mysql.com/doc/refman/5.7/en/replication-features-memory.html) \(英文\)。
- 在建立複本伺服器之後調整主要伺服器上的 [`innodb_file_per_table`](https://dev.mysql.com/doc/refman/5.7/en/innodb-multiple-tablespaces.html) 參數可能會導致複本不同步。複本伺服器不會發覺不同的資料表空間。
- 檢閱 [MySQL 文件](https://dev.mysql.com/doc/refman/5.7/en/replication-features.html) \(英文\) 中的完整 MySQL 複寫限制清單


## <a name="next-steps"></a>後續步驟

- 了解如何[使用 Azure 入口網站來建立及管理讀取複本](howto-read-replicas-portal.md)

<!--
- Learn how to [create and manage read replicas using the Azure CLI](howto-read-replicas-using-cli.md)
-->
