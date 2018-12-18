---
title: 在 Azure HDInsight Hive 資料表中進行資料取樣 | Microsoft Docs
description: 在 Azure HDInsight (Hadopop) Hive 資料表中進行資料向下取樣
services: machine-learning,hdinsight
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: f31e8d01-0fd4-4a10-b1a7-35de3c327521
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2017
ms.author: deguhath
ms.openlocfilehash: 558f7d684453c8b5040f586820bd2a8a9ac0f9c8
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34838427"
---
# <a name="sample-data-in-azure-hdinsight-hive-tables"></a>在 Azure HDInsight Hive 資料表中進行資料取樣
本文說明如何使用 Hive 查詢，對 Azure HDInsight Hive 資料表中儲存的資料向下取樣，以縮減至更適合操控分析的大小。 文中討論三個普遍使用的取樣方法：

* 統一隨機取樣
* 依群組隨機取樣
* 分層取樣

以下**功能表**所連結的主題會說明如何從各種不同儲存體環境進行資料取樣。

[!INCLUDE [cap-sample-data-selector](../../../includes/cap-sample-data-selector.md)]

**為何要對您的資料進行取樣？**
如果您規劃分析的資料集很龐大，通常最好是對資料進行向下取樣，將資料縮減為更小但具代表性且更容易管理的大小。 向下取樣有助於資料的了解、探索和功能工程。 它在 Team Data Science Process 中扮演的角色是，能夠快速建立資料處理函式與機器學習服務模型的原型。

這個取樣工作是 [Team Data Science Process (TDSP)](https://azure.microsoft.com/documentation/learning-paths/cortana-analytics-process/)中的一個步驟。

## <a name="how-to-submit-hive-queries"></a>如何提交 Hive 查詢
Hive 查詢可以從 Hadoop 叢集前端節點上的 Hadoop 命令列主控台提交。 若要執行這個動作，請登入 Hadoop 叢集的前端節點、開啟 Hadoop 命令列主控台，然後從該處提交 Hive 查詢。 如需在 Hadoop 命令列主控台中提交 Hive 查詢的相關指示，請參閱[如何提交 Hive 查詢](move-hive-tables.md#submit)。

## <a name="uniform"></a> 統一隨機取樣
統一隨機取樣表示資料集中的每個資料列都具有相等的取樣機率。 藉由在內部 "select" 查詢中，以及在外部 "select" 查詢 (在該隨機欄位中設定條件) 中，將額外的欄位 rand() 新增至資料集中，即可實作此取樣。

查詢範例如下：

    SET sampleRate=<sample rate, 0-1>;
    select
        field1, field2, …, fieldN
    from
        (
        select
            field1, field2, …, fieldN, rand() as samplekey
        from <hive table name>
        )a
    where samplekey<='${hiveconf:sampleRate}'

在此處， `<sample rate, 0-1>` 會指定使用者想要取樣的記錄比例。

## <a name="group"></a> 依群組隨機取樣
對類別資料進行取樣時，您可能想要包含或排除類別變數中某些值的所有執行個體。 這種取樣稱為「依群組取樣」。 例如，如果您有一個類別變數 "State"，擁有 NY、MA、CA、NJ、PA 等值，您想要讓相同狀態的記錄一律在一起，而不論是否要對它們取樣。

以下是依群組取樣的查詢範例：

    SET sampleRate=<sample rate, 0-1>;
    select
        b.field1, b.field2, …, b.catfield, …, b.fieldN
    from
        (
        select
            field1, field2, …, catfield, …, fieldN
        from <table name>
        )b
    join
        (
        select
            catfield
        from
            (
            select
                catfield, rand() as samplekey
            from <table name>
            group by catfield
            )a
        where samplekey<='${hiveconf:sampleRate}'
        )c
    on b.catfield=c.catfield

## <a name="stratified"></a>分層取樣
若取得的樣本具有的類別值，呈現的比率與它們在母體中的相同，則隨機取樣就會在類別變數方面分層。 使用上述同一個範例，假設您的資料依狀態具有下列觀察：NJ 具有 100 個觀察、NY 具有 60 個觀察，而 WA 具有 300 個觀察。 如果您將分層取樣的比率指定為 0.5，則針對 NJ、NY 及 WA 所獲得的樣本分別應大約有 50、30 及 150 個觀察。

查詢範例如下：

    SET sampleRate=<sample rate, 0-1>;
    select
        field1, field2, field3, ..., fieldN, state
    from
        (
        select
            field1, field2, field3, ..., fieldN, state,
            count(*) over (partition by state) as state_cnt,
              rank() over (partition by state order by rand()) as state_rank
          from <table name>
        ) a
    where state_rank <= state_cnt*'${hiveconf:sampleRate}'


如需可在 Hive 中使用的進一步進階取樣方法相關資訊，請參閱 [LanguageManual 取樣](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Sampling)。

