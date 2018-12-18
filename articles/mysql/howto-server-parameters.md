---
title: 如何在適用於 MySQL 的 Azure 資料庫中設定伺服器參數
description: 本文說明如何使用 Azure 入口網站，在適用於 MySQL 的 Azure 資料庫中設定 MySQL 伺服器參數。
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.topic: article
ms.date: 07/18/2018
ms.openlocfilehash: 686db4dc15eb42abead3eef6798ed2b83e248811
ms.sourcegitcommit: dc646da9fbefcc06c0e11c6a358724b42abb1438
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39136446"
---
# <a name="how-to-configure-server-parameters-in-azure-database-for-mysql-by-using-the-azure-portal"></a>如何使用 Azure 入口網站，在適用於 MySQL 的 Azure 資料庫中設定伺服器參數

MySQL 的 Azure 資料庫支援某些伺服器參數的組態。 本文說明如何使用 Azure 入口網站來設定這些參數。 並非所有伺服器參數皆可調整。

## <a name="navigate-to-server-parameters-on-azure-portal"></a>瀏覽至 Azure 入口網站上的伺服器參數

1. 登入 Azure 入口網站，然後找到適用於 MySQL 的 Azure 資料庫伺服器。
2. 在 [設定] 區段下方，按一下 [伺服器參數] 以開啟適用於 MySQL 伺服器的 Azure 資料庫的 [伺服器參數] 頁面。
![Azure 入口網站伺服器參數頁面](./media/howto-server-parameters/auzre-portal-server-parameters.png)
3. 找出您需要調整的任何設定。 檢閱 [描述] 資料行，以了解用途和允許的值。
![列舉下拉按鈕](./media/howto-server-parameters/3-toggle_parameter.png)
4. 按一下 [儲存] 以儲存變更。
![儲存或捨棄變更](./media/howto-server-parameters/4-save_parameters.png)
5. 儲存新的參數值後，隨時可以選取 [全部重設為預設值] 回復為所有參數的預設值。
![全部重設為預設值](./media/howto-server-parameters/5-reset_parameters.png)

## <a name="list-of-configurable-server-parameters"></a>可設定的伺服器參數清單

支援的伺服器參數清單會不斷成長。 使用 Azure 入口網站中的 [伺服器參數] 索引標籤，即可根據您的應用程式需求取得定義及設定伺服器參數。

## <a name="non-configurable-server-parameters"></a>無法設定的伺服器參數

InnoDB 緩衝集區和連線數目上限無法設定，且與您的[定價層](concepts-service-tiers.md)有關。

|定價層| **計算世代**|**vCore(s)**|**InnoDB 緩衝集區 (MB)**| **連線數目上限**|
|---|---|---|---|--|
|基本| Gen 4| 1| 1024| 50|
|基本| Gen 4| 2| 2560| 100|
|基本| Gen 5| 1| 1024| 50|
|基本| Gen 5| 2| 2560| 100|
|一般用途| Gen 4| 2| 3584| 300|
|一般用途| Gen 4| 4| 7680| 625|
|一般用途| Gen 4| 8| 15360| 1250|
|一般用途| Gen 4| 16| 31232| 2500|
|一般用途| Gen 4| 32| 62976| 5000|
|一般用途| Gen 5| 2| 3584| 300|
|一般用途| Gen 5| 4| 7680| 625|
|一般用途| Gen 5| 8| 15360| 1250|
|一般用途| Gen 5| 16| 31232| 2500|
|一般用途| Gen 5| 32| 62976| 5000|
|記憶體最佳化| Gen 5| 2| 7168| 600|
|記憶體最佳化| Gen 5| 4| 15360| 1250|
|記憶體最佳化| Gen 5| 8| 30720| 2500|
|記憶體最佳化| Gen 5| 16| 62464| 5000|

這些額外的伺服器參數皆無法在系統中設定：

|**參數**|**固定值**|
| :------------------------ | :-------- |
|基本層中的 innodb_file_per_table|關|
|innodb_flush_log_at_trx_commit|1|
|sync_binlog|1|
|innodb_log_file_size|512MB|

未列在此的其他伺服器參數會設定為其在 MySQL [5.7](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html) 和 [5.6](https://dev.mysql.com/doc/refman/5.6/en/innodb-parameters.html) 版中的現成可用預設值。

## <a name="working-with-the-time-zone-parameter"></a>使用時區參數

### <a name="populating-the-time-zone-tables"></a>填入時區資料表

伺服器上的時區資料表，可以藉由從 MySQL 命令列或 MySQL Workbench 等工具呼叫 `az_load_timezone` 預存程序來填入。

> [!NOTE]
> 如果您是從 MySQL Workbench 執行 `az_load_timezone` 命令，您可能需要先執行 `SET SQL_SAFE_UPDATES=0;` 以關閉安全更新模式。

```sql
CALL mysql.az_load_timezone();
```

若要檢視可用的時區值，請執行以下命令：

```sql
SELECT name FROM mysql.time_zone_name;
```

### <a name="setting-the-global-level-time-zone"></a>設定全域層級時區

全域層級時區可從 Azure 入口網站中的 [伺服器參數] 頁面設定。 下方的命令可將全域時區的值設定為 "US/Pacific"。

![設定時區參數](./media/howto-server-parameters/timezone.png)

### <a name="setting-the-session-level-time-zone"></a>設定工作階段層級時區

工作階段層級時區可以藉由從 MySQL 命令列或 MySQL Workbench 等工具呼叫 `SET time_zone` 命令來設定。 以下範例將時區設為 **US/Pacific** 時區。

```sql
SET time_zone = 'US/Pacific';
```

如需[日期和時間函式](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_convert-tz) \(英文\) 的詳細資訊，請參閱 MySQL 文件。

## <a name="next-steps"></a>後續步驟

- [適用於 MySQL 的 Azure 資料庫的連線庫](concepts-connection-libraries.md)。
