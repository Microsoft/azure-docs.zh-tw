---
title: 如何對適用於 MySQL 的 Azure 資料庫中的查詢效能進行疑難排解
description: 本文說明如何使用 EXPLAIN 對適用於 MySQL 的 Azure 資料庫中的查詢效能進行疑難排解。
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: 72b047c37ac88e4b33c8723f8df14c6794e84399
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35266171"
---
# <a name="how-to-use-explain-to-profile-query-performance-in-azure-database-for-mysql"></a>如何使用 EXPLAIN 剖析適用於 MySQL 的 Azure 資料庫中的查詢效能
**EXPLAIN** 是可最佳化查詢的便利工具。 EXPLAIN 陳述式可用來取得關於 SQL 陳述式如何執行的資訊。 下列輸出顯示 EXPLAIN 陳述式的執行範例。

```sql
mysql> EXPLAIN SELECT * FROM tb1 WHERE id=100\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 995789
     filtered: 10.00
        Extra: Using where
```

在此範例中您可以看到，*key* 的值為 NULL。 此輸出表示 MySQL 找不到任何查詢的最佳化索引，因此執行了完整資料表掃描。 我們將在 [識別碼] 資料行上新增索引，以最佳化此查詢。

```sql
mysql> ALTER TABLE tb1 ADD KEY (id);
mysql> EXPLAIN SELECT * FROM tb1 WHERE id=100\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: ref
possible_keys: id
          key: id
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
```

新的 EXPLAIN 顯示 MySQL 現在會使用索引將列數限定為 1，因而大幅縮短了搜尋時間。
 
## <a name="covering-index"></a>涵蓋索引
涵蓋索引由索引中的所有查詢資料行所組成，可減少從資料表中擷取的值。 以下是下列 **GROUP BY** 陳述式的說明。
 
```sql
mysql> EXPLAIN SELECT MAX(c1), c2 FROM tb1 WHERE c2 LIKE '%100' GROUP BY c1\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 995789
     filtered: 11.11
        Extra: Using where; Using temporary; Using filesort
```

如輸出所顯示，MySQL 並未使用任何索引，因為沒有適當的索引可供使用。 其中也顯示 *Using temporary; Using file sort*，這表示 MySQL 建立了暫存資料表以滿足 **GROUP BY** 子句。
 
單獨建立資料行 **c2** 的索引並無作用，MySQL 仍然需要建立暫存資料表：

```sql 
mysql> ALTER TABLE tb1 ADD KEY (c2);
mysql> EXPLAIN SELECT MAX(c1), c2 FROM tb1 WHERE c2 LIKE '%100' GROUP BY c1\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 995789
     filtered: 11.11
        Extra: Using where; Using temporary; Using filesort
```

在此情況下，可建立 **c1** 和 **c2** 兩者的**涵蓋性索引**，藉以將 **c2**" 的值直接加入索引中，而消除進一步的資料查閱。

```sql 
mysql> ALTER TABLE tb1 ADD KEY covered(c1,c2);
mysql> EXPLAIN SELECT MAX(c1), c2 FROM tb1 WHERE c2 LIKE '%100' GROUP BY c1\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: index
possible_keys: covered
          key: covered
      key_len: 108
          ref: NULL
         rows: 995789
     filtered: 11.11
        Extra: Using where; Using index
```

如上述 EXPLAIN 所示，MySQL 此時會使用涵蓋性索引，而避免建立暫存資料表。 

## <a name="combined-index"></a>結合的索引
結合的索引包含多個資料行的值，可被視為依串連的索引資料行值排序的資料列陣列。 此方法適用於 **GROUP BY** 陳述式。

```sql
mysql> EXPLAIN SELECT c1, c2 from tb1 WHERE c2 LIKE '%100' ORDER BY c1 DESC LIMIT 10\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 995789
     filtered: 11.11
        Extra: Using where; Using filesort
```

MySQL 會執行速度緩慢的*檔案排序*作業，尤其是在必須排序許多資料列時。 若要最佳化此查詢，您可以對這兩個要排序的資料行建立結合的索引。

```sql 
mysql> ALTER TABLE tb1 ADD KEY my_sort2 (c1, c2);
mysql> EXPLAIN SELECT c1, c2 from tb1 WHERE c2 LIKE '%100' ORDER BY c1 DESC LIMIT 10\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: index
possible_keys: NULL
          key: my_sort2
      key_len: 108
          ref: NULL
         rows: 10
     filtered: 11.11
        Extra: Using where; Using index
```

現在，EXPLAIN 顯示 MySQL 已能夠使用結合的索引避免額外的排序，因為索引已排序。
 
## <a name="conclusion"></a>結論
 
使用 EXPLAIN 和不同類型的索引可大幅提升效能。 單憑您已有資料表的索引，不一定表示 MySQL 就可以將它用於您的查詢。 請務必使用 EXPLAIN 驗證您的假設，並使用索引將查詢最佳化。


## <a name="next-steps"></a>後續步驟
- 若想知道是否有人可解答您最關切的問題，或是要張貼新問題/解答，請造訪 [MSDN 論壇](https://social.msdn.microsoft.com/forums/security/en-US/home?forum=AzureDatabaseforMySQL)或[堆疊溢位](https://stackoverflow.com/questions/tagged/azure-database-mysql)。
