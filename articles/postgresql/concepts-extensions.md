---
title: 使用適用於 PostgreSQL 的 Azure 資料庫中的 PostgreSQL 擴充功能
description: 描述下列功能：在適用於 PostgreSQL 的 Azure 資料庫中，使用擴充功能來擴充資料庫功能。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.topic: article
ms.date: 05/30/2018
ms.openlocfilehash: 92321929c7929cc62321317c7e94e74ef77034ad
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38970078"
---
# <a name="postgresql-extensions-in-azure-database-for-postgresql"></a>適用於 PostgreSQL 的 Azure 資料庫中的 PostgreSQL 擴充功能
PostgreSQL 提供下列功能：使用擴充功能來擴充您的資料庫功能。 擴充功能可在單一封裝中一併統合多個相關的 SQL 物件，其可使用單一命令從您的資料庫加以載入或移除。 載入資料庫之後，擴充功能就可如內建功能般運作。 如需 PostgreSQL 擴充功能的詳細資訊，請參閱[將相關物件封裝成擴充功能 (英文)](https://www.postgresql.org/docs/9.6/static/extend-extensions.html)。

## <a name="how-to-use-postgresql-extensions"></a>如何使用 PostgreSQL 擴充功能
您必須先在資料庫中安裝 PostgreSQL 擴充功能，然後才能使用它們。 若要安裝特定的擴充功能，請從 psql 工具執行 [CREATE EXTENSION](https://www.postgresql.org/docs/9.6/static/sql-createextension.html) 命令，以將封裝的物件載入至您的資料庫。

適用於 PostgreSQL 的 Azure 資料庫目前支援的部分重要擴充功能如下所列。 不支援未列出的擴充功能；您無法利用適用於 PostgreSQL 的 Azure 資料庫服務，自行建立擴充功能。

## <a name="extensions-supported-by-azure-database-for-postgresql"></a>適用於 PostgreSQL 的 Azure 資料庫支援的擴充功能
下表列出適用於 PostgreSQL 的 Azure 資料庫目前支援的標準 PostgreSQL 擴充功能。 此資訊也可以藉由執行 `SELECT * FROM pg_available_extensions;` 來取得。

### <a name="data-types-extensions"></a>資料類型擴充功能

> [!div class="mx-tableFixed"]
| **擴充功能** | **說明** |
|---|---|
| [chkpass](https://www.postgresql.org/docs/9.6/static/chkpass.html) | 為自動加密密碼提供資料類型。 |
| [citext](https://www.postgresql.org/docs/9.6/static/citext.html) | 提供不區分大小寫的字元字串類型。 |
| [cube](https://www.postgresql.org/docs/9.6/static/cube.html) | 提供多維度 Cube 的資料類型。 |
| [hstore](https://www.postgresql.org/docs/9.6/static/hstore.html) | 提供用來儲存索引鍵/值組之集合的資料類型。 |
| [isn](https://www.postgresql.org/docs/9.6/static/isn.html) | 提供國際產品編號標準的資料類型。 |
| [ltree](https://www.postgresql.org/docs/9.6/static/ltree.html) | 提供階層式樹狀結構的資料類型。 |

### <a name="functions-extensions"></a>函數擴充功能

> [!div class="mx-tableFixed"]
| **擴充功能** | **說明** |
|---|---|
| [earthdistance](https://www.postgresql.org/docs/9.6/static/earthdistance.html) | 提供方法來計算地球表面上的大圓距離。 |
| [fuzzystrmatch](https://www.postgresql.org/docs/9.6/static/fuzzystrmatch.html) | 提供數個函數來判斷字串之間的相似性與距離。 |
| [intarray](https://www.postgresql.org/docs/9.6/static/intarray.html) | 提供函數和運算子來操作無 null 的整數陣列。 |
| [pgcrypto](https://www.postgresql.org/docs/9.6/static/pgcrypto.html) | 提供密碼編譯函式。 |
| [pg\_partman](https://pgxn.org/dist/pg_partman/doc/pg_partman.html) | 依時間或識別碼管理分割的資料表。 |
| [pg\_trgm](https://www.postgresql.org/docs/9.6/static/pgtrgm.html) | 提供可根據三併詞比對判斷英數文字相似度的函式和運算子。 |
| [tablefunc](https://www.postgresql.org/docs/9.6/static/tablefunc.html) | 提供操縱整個資料表 (包括交叉資料表) 的函式。 |
| [uuid-ossp](https://www.postgresql.org/docs/9.6/static/uuid-ossp.html) | 產生通用唯一識別碼 (UUID)。 |

### <a name="full-text-search-extensions"></a>全文檢索搜尋擴充功能

> [!div class="mx-tableFixed"]
| **擴充功能** | **說明** |
|---|---|
| [dict\_int](https://www.postgresql.org/docs/9.6/static/dict-int.html) | 提供整數的文字搜尋字典範本。 |
| [unaccent](https://www.postgresql.org/docs/9.6/static/unaccent.html) | 文字搜尋字典，可從詞彙中移除重音符號 (變音符號)。 |

### <a name="index-types-extensions"></a>索引類型擴充功能

> [!div class="mx-tableFixed"]
| **擴充功能** | **說明** |
|---|---|
| [btree\_gin](https://www.postgresql.org/docs/9.6/static/btree-gin.html) | 提供範例 GIN 運算子類別，可針對特定資料類型實作類似 B 型樹狀結構的行為。 |
| [btree\_gist](https://www.postgresql.org/docs/9.6/static/btree-gist.html) | 提供可實作 B 型樹狀結構的 GiST 索引運算子類別。 |

### <a name="language-extensions"></a>語言擴充功能

> [!div class="mx-tableFixed"]
| **擴充功能** | **說明** |
|---|---|
| [plpgsql](https://www.postgresql.org/docs/9.6/static/plpgsql.html) | PL/pgSQL 可載入的程序性語言。 |

### <a name="miscellaneous-extensions"></a>其他擴充功能

> [!div class="mx-tableFixed"]
| **擴充功能** | **說明** |
|---|---|
| [pg\_buffercache](https://www.postgresql.org/docs/9.6/static/pgbuffercache.html) | 提供一種方法，即時檢查共用緩衝區快取中發生的狀況。 |
| [pg\_prewarm](https://www.postgresql.org/docs/9.6/static/pgprewarm.html) | 提供一種方式，來將關聯資料載入至緩衝區快取。 |
| [pg\_stat\_statements](https://www.postgresql.org/docs/9.6/static/pgstatstatements.html) | 提供一種方法，來追蹤伺服器所執行之所有 SQL 陳述式的執行統計資料。 (請參閱以下關於此延伸模組的注意事項)。 |
| [pgrowlocks](https://www.postgresql.org/docs/9.6/static/pgrowlocks.html) | 提供方法來顯示資料列層級鎖定的資訊。 |
| [pgstattuple](https://www.postgresql.org/docs/9.6/static/pgstattuple.html) | 提供方法來顯示 Tuple 層級統計資料。 |
| [postgres\_fdw](https://www.postgresql.org/docs/9.6/static/postgres-fdw.html) | 可用來存取儲存於外部 PostgreSQL 伺服器之資料的外部資料包裝函式。 |
| [hypopg](https://hypopg.readthedocs.io/en/latest/) | 提供方法建立不使用 CPU 或磁碟的假設索引。 |

### <a name="postgis-extensions"></a>PostGIS 擴充功能

> [!div class="mx-tableFixed"]
| **擴充功能** | **說明** |
|---|---|
| [PostGIS](http://www.postgis.net/)、postgis\_topology、postgis\_tiger\_geocoder、postgis\_sfcgal | 適用於 PostgreSQL 的空間與地理物件。 |
| address\_standardizer、address\_standardizer\_data\_us | 用來將位址剖析為組成項目。 用來支援對位址進行地理編碼的正規化步驟。 |
| [pgrouting](http://pgrouting.org/) | 擴充 PostGIS / PostgreSQL 地理空間資料庫，以提供地理空間路由功能。 |


### <a name="using-pgstatstatements"></a>使用 pg_stat_statements
每個適用於 PostgreSQL 的 Azure 資料庫已預先載入 [Pg\_stat\_ 陳述式延伸模組](https://www.postgresql.org/docs/9.6/static/pgstatstatements.html)，以便 PostgreSQL 伺服器提供追蹤 SQL 陳述式執行統計資料的方法。
控制延伸模組計入哪些陳述式的設定 `pg_stat_statements.track` 預設為 `top`，這表示會追蹤用戶端直接發出的所有陳述式。 其他兩個會追蹤層級 `none` 和 `all`。 這項設定是可透過 [Azure 入口網站](https://docs.microsoft.com/azure/postgresql/howto-configure-server-parameters-using-portal)或 [Azure CLI](https://docs.microsoft.com/azure/postgresql/howto-configure-server-parameters-using-cli) 設定的伺服器參數。

pg_stat_statements 提供查詢執行資訊，不過會對於伺服器效能造成影響，因為它會記錄每個 SQL 陳述式。 如果您不主動使用 pg_stat_statements 延伸模組，我們建議您將 `pg_stat_statements.track` 設定為 `none`。 請注意，某些協力廠商監視服務可能會需要 pg_stat_statements 傳遞查詢效能深入解析，因此請確認這是否為您的情況。


## <a name="next-steps"></a>後續步驟
如果您未看見想要使用的擴充功能，請讓我們知道。 在我們的[客戶意見反應論壇](https://feedback.azure.com/forums/597976-azure-database-for-postgresql) \(英文\) 中投票給現有要求，或建立新的意見反應和要求。
