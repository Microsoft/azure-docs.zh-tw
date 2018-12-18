---
title: 使用 Hive 查詢來瀏覽 Hive 資料表的資料 | Microsoft Docs
description: 使用 Hive 查詢來瀏覽 Hive 資料表的資料。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 0d46cea5-2b4c-4384-9bfa-fa20f6f75148
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/09/2017
ms.author: deguhath
ms.openlocfilehash: 79d40617ae4f9cd83d04cad213e5d8fd76b03876
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42145157"
---
# <a name="explore-data-in-hive-tables-with-hive-queries"></a>使用 Hive 查詢來瀏覽 Hive 資料表的資料
本文件提供 Hive 指令碼範例，可用來瀏覽 HDInsight Hadoop 叢集的 Hive 資料表中的資料。

下列 **功能表** 所連結的主題會說明如何從各種不同的儲存體環境使用工具來瀏覽資料。

[!INCLUDE [cap-explore-data-selector](../../../includes/cap-explore-data-selector.md)]

## <a name="prerequisites"></a>必要條件
本文假設您已經：

* 建立 Azure 儲存體帳戶。 如需指示，請參閱[建立 Azure 儲存體帳戶](../../storage/common/storage-quickstart-create-account.md)
* 佈建含有 HDInsight 服務的自訂 Hadoop 叢集。 如需指示，請參閱 [自訂適用於進階分析的 Azure HDInsight Hadoop 叢集](customize-hadoop-cluster.md)。
* 已將資料上傳至 Azure HDInsight Hadoop 叢集中的 Hive 資料表。 如果沒有，請遵循 [建立資料並載入 Hive 資料表](move-hive-tables.md) 中的指示，先將資料上傳至 Hive 資料表。
* 啟用叢集的遠端存取。 如需指示，請參閱 [存取 Hadoop 叢集的前端節點](customize-hadoop-cluster.md)。
* 如果您需要如何提交 Hive 查詢的指示，請參閱 [如何提交 Hive 查詢](move-hive-tables.md#submit)

## <a name="example-hive-query-scripts-for-data-exploration"></a>資料探索的 Hive 查詢指令碼範例
1. 取得每個分割區的觀察計數 `SELECT <partitionfieldname>, count(*) from <databasename>.<tablename> group by <partitionfieldname>;`
2. 取得每天的觀察計數 `SELECT to_date(<date_columnname>), count(*) from <databasename>.<tablename> group by to_date(<date_columnname>);`
3. 取得類別資料行中的層級   
    `SELECT  distinct <column_name> from <databasename>.<tablename>`
4. 取得兩個類別資料行組合中的層級數目 `SELECT <column_a>, <column_b>, count(*) from <databasename>.<tablename> group by <column_a>, <column_b>`
5. 取得數值資料行的分佈  
    `SELECT <column_name>, count(*) from <databasename>.<tablename> group by <column_name>`
6. 聯結兩個資料表來擷取記錄
   
        SELECT
            a.<common_columnname1> as <new_name1>,
            a.<common_columnname2> as <new_name2>,
            a.<a_column_name1> as <new_name3>,
            a.<a_column_name2> as <new_name4>,
            b.<b_column_name1> as <new_name5>,
            b.<b_column_name2> as <new_name6>
        FROM
            (
            SELECT <common_columnname1>,
                <common_columnname2>,
                <a_column_name1>,
                <a_column_name2>,
            FROM <databasename>.<tablename1>
            ) a
            join
            (
            SELECT <common_columnname1>,
                <common_columnname2>,
                <b_column_name1>,
                <b_column_name2>,
            FROM <databasename>.<tablename2>
            ) b
            ON a.<common_columnname1>=b.<common_columnname1> and a.<common_columnname2>=b.<common_columnname2>

## <a name="additional-query-scripts-for-taxi-trip-data-scenarios"></a>計程車路線資料案例的其他查詢指令碼
[GitHub 存放庫](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts)中也會提供 [NYC 計程車車程資料](http://chriswhong.com/open-data/foil_nyc_taxi/)案例的專屬查詢範例。 這些查詢已經具備指定的資料結構描述，且準備好進行提交來執行。

