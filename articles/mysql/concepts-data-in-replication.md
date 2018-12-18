---
title: 將資料複寫至適用於 MySQL 的 Azure 資料庫。
description: 本文將說明適用於 MySQL 的 Azure 資料庫之中的資料帶入複寫 (Data-in Replication)。
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.topic: article
ms.date: 08/31/2018
ms.openlocfilehash: 6135e4a0182f3af7db54eab974e4c307402185ab
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666071"
---
# <a name="replicate-data-into-azure-database-for-mysql"></a>將資料複寫至適用於 MySQL 的 Azure 資料庫

資料帶入複寫可讓您將來自在內部部署執行的 MySQL 伺服器、虛擬機器中或由其他雲端提供者所代管的資料庫服務的資料，同步處理到適用於 MySQL 的 Azure 資料庫服務。 資料帶入複寫是建立在以二進位記錄 (binlog) 檔案位置為基礎的 MySQL 原生複寫之上。 若要深入了解 binlog 複寫，請參閱 [MySQL binlog 複寫概觀](https://dev.mysql.com/doc/refman/5.7/en/binlog-replication-configuration-overview.html) \(英文\)。 

## <a name="when-to-use-data-in-replication"></a>使用資料帶入複寫的時機
考慮使用資料帶入複寫的主要案例包括：

- **混合式資料同步：** 您可使用資料帶入複寫，讓內部部署伺服器與適用於 MySQL 的 Azure 資料庫之間的資料保持同步。 此同步適用於建立混合式應用程式。 當您目前擁有本機資料庫伺服器，但想要將資料移到更接近使用者的區域時，這個方法很吸引人。
- **多重雲端同步處理：** 針對複雜的雲端解決方案，使用資料帶入複寫來同步處理適用於 MySQL 的 Azure 資料庫與不同雲端提供者 (包括這些雲端中所裝載的虛擬機器和資料庫服務) 之間的資料。

## <a name="limitations-and-considerations"></a>限制與注意事項

### <a name="data-not-replicated"></a>不會複寫資料
不會複寫主要伺服器上的 [mysql 系統資料庫](https://dev.mysql.com/doc/refman/5.7/en/system-database.html) \(英文\)。 不會複寫主要伺服器上對於帳戶和權限的變更。 如果您在主要伺服器上建立帳戶且此帳戶需要存取複本伺服器，則需在複本伺服器端手動建立相同的帳戶。 若要了解系統資料庫中包含哪些資料表，請參閱 [MySQL 手冊](https://dev.mysql.com/doc/refman/5.7/en/system-database.html) \(英文\)。

### <a name="requirements"></a>需求
- 主要伺服器版本必須至少是 MySQL 5.6 版。 
- 主要和複本伺服器的版本必須相同。 例如，兩者都必須是 MySQL 5.6 版，或兩者都必須是 MySQL 5.7 版。
- 每個資料表都必須有主索引鍵。
- 主要伺服器應該使用 MySQL InnoDB 引擎。
- 使用者必須有權設定二進位記錄，以及在主要伺服器上建立新的使用者。

### <a name="other"></a>其他
- 資料輸入複寫只適用於一般用途和記憶體最佳化定價層
- 不支援全域交易識別碼 (GTID)。

## <a name="next-steps"></a>後續步驟
- 了解如何[設定資料帶入複寫](howto-data-in-replication.md)
