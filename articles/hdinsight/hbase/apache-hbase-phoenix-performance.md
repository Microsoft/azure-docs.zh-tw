---
title: Azure HDInsight 中的 Phoenix 效能
description: 將 Phoenix 效能最佳化的最佳做法。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: ashishth
ms.openlocfilehash: ff194ef7f5ae609eba5334eb5c66db02d660ab08
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049626"
---
# <a name="phoenix-performance-best-practices"></a>Phoenix 效能最佳做法

Phoenix 效能最重要的層面，就是將基礎 HBase 最佳化。 Phoenix 會在負責將 SQL 查詢轉換成 HBase 作業 (例如掃描) 的 HBase 之上建立關聯式資料模型。 資料表結構描述的設計、主索引鍵中欄位的選取和順序以及索引的使用，都會影響 Phoenix 效能。

## <a name="table-schema-design"></a>資料表結構描述設計

當您在 Phoenix 中建立資料表時，該資料表會儲存在 HBase 資料表中。 HBase 資料表包含資料行群組 (資料行系列)，群組內的資料行會一起存取。 Phoenix 資料表中的一個資料列就是 HBase 資料表中的一個資料列，而每個資料列由與一或多個資料行相關聯的已設版本資料格組成。 邏輯上，單一 HBase 資料列是索引鍵/值組的集合，每個組合具有相同的 rowkey 值。 也就是說，每個索引鍵/值組都有一個 rowkey 屬性，而且特定資料列的該 rowkey 屬性值都相同。

Phoenix 資料表的結構描述設計包括主索引鍵設計、資料行系列設計、個別資料行設計、資料的分割方式。

### <a name="primary-key-design"></a>主索引鍵設計

Phoenix 中資料表上定義的主索引鍵，可決定資料儲存在基礎 HBase 資料表的 rowkey 中的方式。 在 HBase 中，存取特定資料列的唯一方式就是使用 rowkey。 而且，儲存在 HBase 資料表中的資料會依照 rowkey 排序。 Phoenix 會串連資料列中每個資料行的值 (依照其在主索引鍵中定義的順序)，以建立 rowkey 值。

例如，連絡人的資料表中有名字、姓氏、電話號碼和地址，全都屬於相同的資料行系列。 您可以遞增的序號來定義主索引鍵：

|rowkey|       位址|   電話| firstName| lastName|
|------|--------------------|--------------|-------------|--------------|
|  1000|1111 San Gabriel Dr.|1-425-000-0002|    John|Dole|
|  8396|5415 San Gabriel Dr.|1-230-555-0191|  Calvin|Raji|

不過，如果您經常依 lastName 查詢，此主索引鍵可能效能不佳，因為每個查詢都需要完整掃描資料表，才能讀取每個 lastName 的值。 然而，您可以在 lastName、firstName、socialSecurityNum (社會安全號碼) 資料行上定義主索引鍵。 此最後一個資料行是用來釐清住在相同地址且同名的兩個居民，例如父與子。

|rowkey|       位址|   電話| firstName| lastName| socialSecurityNum |
|------|--------------------|--------------|-------------|--------------| ---|
|  1000|1111 San Gabriel Dr.|1-425-000-0002|    John|Dole| 111 |
|  8396|5415 San Gabriel Dr.|1-230-555-0191|  Calvin|Raji| 222 |

使用這個新的主索引鍵，Phoenix 所產生的 rowkey 會是：

|rowkey|       位址|   電話| firstName| lastName| socialSecurityNum |
|------|--------------------|--------------|-------------|--------------| ---|
|  Dole-John-111|1111 San Gabriel Dr.|1-425-000-0002|    John|Dole| 111 |
|  Raji-Calvin-222|5415 San Gabriel Dr.|1-230-555-0191|  Calvin|Raji| 222 |

在上表的第一列中，rowkey 的資料如下所示：

|rowkey|       索引鍵|   value| 
|------|--------------------|---|
|  Dole-John-111|位址 |1111 San Gabriel Dr.|  
|  Dole-John-111|電話 |1-425-000-0002|  
|  Dole-John-111|firstName |John|  
|  Dole-John-111|lastName |Dole|  
|  Dole-John-111|socialSecurityNum |111| 

這個 rowkey 現在可儲存資料的複本。 請考量主索引鍵包含的資料行大小和數目，因為基礎 HBase 資料表中的每個資料格都包含這個值。

此外，如果主索引鍵具有單調遞增值 ，您應該建立具有「salt 貯體」的資料表，協助您避免建立寫入熱點；請參閱[分割資料](#partition-data)。

### <a name="column-family-design"></a>資料行系列設計

如果有些資料行比其他資料行的存取更頻繁，您應該建立多個資料行系列，將經常存取的資料行與很少存取的資料行分開。

此外，如果有些資料行通常一起被存取，請將這些資料行放在相同的資料行系列中。

### <a name="column-design"></a>資料行設計

* 由於大型資料行的 I/O 成本，請讓 VARCHAR 資料行保持在大約 1 MB 以下。 在處理查詢時，HBase 會先完整地將資料格具體化，再將其傳送給用戶端，而用戶端完整接收它們後，才會將其交給應用程式的程式碼。
* 使用精簡格式來儲存資料行的值，例如 protobuf、Avro、msgpack 或 BSON。 JSON 比較大，不建議使用。
* 請考慮先壓縮資料再儲存，以縮減延遲和 I/O 成本。

### <a name="partition-data"></a>分割資料

Phoenix 可讓您控制資料散佈的區域數量，進而大幅增加讀/寫效能。 建立 Phoenix 資料表時，您可以將您的資料 salt 化或預先分割。

若要在資料表建立期間 salt 化，請指定 salt 貯體的數目：

    CREATE TABLE CONTACTS (...) SALT_BUCKETS = 16

此 salt 化作業會將資料表沿著主索引鍵的值分割，並自動選擇值。 

若要控制發生資料表分割的位置，只有提供發生分割的範圍值，即可預先分割資料表。 例如，若要建立沿著三個區域分割的資料表：

    CREATE TABLE CONTACTS (...) SPLIT ON ('CS','EU','NA')

## <a name="index-design"></a>索引設計

Phoenix 索引是一個 HBase 資料表，其中儲存索引資料表中部分或全部的資料複本。 索引可改善特定查詢類型的效能。

當您定義了多個索引，然後查詢資料表時，Phoenix 會針對該查詢自動選取最佳索引。 它會根據您選取的主索引鍵自動建立主要索引。

在預期的查詢中，您也可以藉由指定資料行來建立次要索引。

在設計索引時：

* 只建立您所需的索引。
* 限制經常更新之資料表的索引數目。 資料表的更新會轉譯成主資料表和索引資料表的寫入。

## <a name="create-secondary-indexes"></a>建立次要索引

次要索引可藉由將完整資料表掃描轉變成點查閱來提升讀取效能，其代價是儲存空間和寫入速度。 您可以在資料表建立之後新增或移除次要索引，而不需要變更現有的查詢 – 查詢的執行速度會更快。 根據您的需求，考慮要建立涵蓋性索引、功能性索引、或兩者。

### <a name="use-covered-indexes"></a>使用涵蓋性索引

涵蓋性索引除了包含檢索的值，還包含資料列中的資料。 找到所要的索引項目後，就不需要存取主要資料表。

例如，在連絡人資料表範例中，您可以只在 socialSecurityNum 資料行上建立次要索引。 此次要索引可加速依 socialSecurityNum 值篩選的查詢，但是擷取其他欄位值則需要零外讀取主資料表。

|rowkey|       位址|   電話| firstName| lastName| socialSecurityNum |
|------|--------------------|--------------|-------------|--------------| ---|
|  Dole-John-111|1111 San Gabriel Dr.|1-425-000-0002|    John|Dole| 111 |
|  Raji-Calvin-222|5415 San Gabriel Dr.|1-230-555-0191|  Calvin|Raji| 222 |

不過，如果您通常會查閱指定 socialSecurityNum 的 firstName 和 lastName，您可以建立涵蓋性索引，其中包含 firstName 和 lastName 作為索引資料表中的實際資料：

    CREATE INDEX ssn_idx ON CONTACTS (socialSecurityNum) INCLUDE(firstName, lastName);

此涵蓋性索引可讓下列查詢只要讀取包含次要索引的資料表，就可以取得所有資料：

    SELECT socialSecurityNum, firstName, lastName FROM CONTACTS WHERE socialSecurityNum > 100;

### <a name="use-functional-indexes"></a>使用功能性索引

功能性索引可讓您對您希望用於查詢的任意運算式建立索引。 一旦您備妥功能性索引，並有查詢使用該運算式，索引就可用來擷取結果，而不是資料表。

例如，您可以建立一個索引，讓您對人員的名字和姓氏組合執行不區分大小寫的搜尋：

     CREATE INDEX FULLNAME_UPPER_IDX ON "Contacts" (UPPER("firstName"||' '||"lastName"));

## <a name="query-design"></a>查詢設計

查詢設計的主要考量如下：

* 了解查詢計劃並確認其預期行為。
* 有效率地聯結。

### <a name="understand-the-query-plan"></a>了解查詢計劃

在 [SQLLine](http://sqlline.sourceforge.net/) 中，在 SQL 查詢前面使用 EXPLAIN，可以檢視 Phoenix 將執行的作業計畫。 檢查此計劃：

* 適當使用您的主索引鍵。
* 使用適當的次要索引，而不是資料表。
* 可能的話，使用 RANGE SCAN 或 SKIP SCAN，而非 TABLE SCAN。

#### <a name="plan-examples"></a>計劃範例

例如，假設您有一個名為 FLIGHTS 的資料表，其中儲存航班延誤資訊。

若要選取所有 airlineid 為 `19805` 的航班，其中 airlineid 是不在主索引鍵或任何索引中的欄位：

    select * from "FLIGHTS" where airlineid = '19805';

執行 explain (解說) 命令，如下所示︰

    explain select * from "FLIGHTS" where airlineid = '19805';

查詢計劃如下所示：

    CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN FULL SCAN OVER FLIGHTS
        SERVER FILTER BY AIRLINEID = '19805'

在此計劃中，請注意 FULL SCAN OVER FLIGHTS 字詞。 這段字詞表示執行會對資料表中的所有資料列進行 TABLE SCAN，而不是使用更有效率的 RANGE SCAN 或 SKIP SCAN 選項。

現在，假設您要查詢 2014 年 1 月 2 日 `AA` 航空公司且航班編號大於 1 的航班。 假設範例資料表中有 year、month、dayofmonth、carrier、flightnum 資料行，而且這些資料行會組成複合主索引鍵。 查詢應如下所示：

    select * from "FLIGHTS" where year = 2014 and month = 1 and dayofmonth = 2 and carrier = 'AA' and flightnum > 1;

讓我們檢查此查詢的計畫：

    explain select * from "FLIGHTS" where year = 2014 and month = 1 and dayofmonth = 2 and carrier = 'AA' and flightnum > 1;

產生的計畫如下：

    CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN RANGE SCAN OVER FLIGHTS [2014,1,2,'AA',2] - [2014,1,2,'AA',*]

方括號中的值是主索引鍵的值範圍。 在此案例下，範圍值是固定為 year 2014、month 1、dayofmonth 2，但允許 flightnum 的值從 2 和以上 (`*`) 開始。 此查詢計劃確認系統將如預期般使用主索引鍵。

接下來，只在 FLIGHTS 資料表的 carrier 欄位建立名為 `carrier2_idx` 的索引。 此索引也包含 flightdate、tailnum、origin、flightnum 涵蓋性資料行，其資料也會儲存在索引中。

    CREATE INDEX carrier2_idx ON FLIGHTS (carrier) INCLUDE(FLIGHTDATE,TAILNUM,ORIGIN,FLIGHTNUM);

假設您想要取得 carrier 以及 flightdate 和 tailnum，如下列查詢所示：

    explain select carrier,flightdate,tailnum from "FLIGHTS" where carrier = 'AA';

您應會看見系統使用此索引：

    CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN RANGE SCAN OVER CARRIER2_IDX ['AA']

如需 explain 計劃結果中可以出現的完整項目清單，請參閱 [Apache Phoenix 微調指南](https://phoenix.apache.org/tuning_guide.html)中的＜解說計劃＞。

### <a name="join-efficiently"></a>有效率地聯結

一般而言，您應避免聯結，除非一邊很小，尤其是經常用的查詢。

如有必要，您可以使用 `/*+ USE_SORT_MERGE_JOIN */` 提示進行大型聯結，但是大量資料列的大型聯結是所費不貲的作業。 如果所有右側資料表的整體大小超過可用的記憶體，請使用 `/*+ NO_STAR_JOIN */` 提示。

## <a name="scenarios"></a>案例

下列指引說明一些常見的模式。

### <a name="read-heavy-workloads"></a>大量讀取工作負載

在大量讀取的使用案例中，請務必使用索引。 此外，若要節省讀取階段額外負荷，請考慮建立涵蓋式索引。

### <a name="write-heavy-workloads"></a>大量寫入工作負載

針對主索引鍵單調遞增的大量寫入工作負載，建立 salt 貯體可協助您避免寫入熱點，但會因為需要額外的掃描而犧牲整體讀取輸送量。 此外，使用 UPSERT 來寫入大量的記錄時，請關閉 autoCommit 並將記錄批次化。

### <a name="bulk-deletes"></a>大量刪除

刪除大型資料集時，先開啟 autoCommit 再發出 DELETE 查詢，用戶端便不需要記住所有已刪除資料列的資料列索引鍵。 AutoCommit 可以防止用戶端緩衝處理受 DELETE 影響的資料列，因此 Phoenix 可以直接在區域伺服器上將其刪除，免於將其傳回給用戶端。

### <a name="immutable-and-append-only"></a>不可變和僅能附加

如果您的案例偏重於寫入速度而較不重視資料完整性，請考慮在建立資料表時停用預寫記錄檔：

    CREATE TABLE CONTACTS (...) DISABLE_WAL=true;

如需這個選項及其他選項的詳細資訊，請參閱 [Phoenix 文法](http://phoenix.apache.org/language/index.html#options)。

## <a name="next-steps"></a>後續步驟

* [Phoenix 微調指南](https://phoenix.apache.org/tuning_guide.html)
* [次要索引](http://phoenix.apache.org/secondary_indexing.html)
