---
title: 針對使用 Hive 查詢之 Hadoop 叢集中的資料建立特性 | Microsoft Docs
description: Hive 查詢的範例，產生儲存在 Azure HDInsight Hadoop 叢集中之資料中的特性。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: e8a94c71-979b-4707-b8fd-85b47d309a30
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/21/2017
ms.author: deguhath
ms.openlocfilehash: 0e46ce327bb4beffd631ef6369864a0888580c11
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34836658"
---
# <a name="create-features-for-data-in-a-hadoop-cluster-using-hive-queries"></a>針對使用 Hive 查詢之 Hadoop 叢集中的資料建立特徵
本文件說明如何使用 Hive 查詢，針對儲存在 Azure HDInsight Hadoop 叢集中的資料建立特徵。 這些 Hive 查詢會使用針對其提供指令碼的內嵌 Hive 使用者定義函式 (UDF)。

建立特徵所需的作業可能耗用大量記憶體。 在此情況下，Hive 查詢的效能會變得十分重要，可微調某些參數來改善。 最後一節討論如何微調這些參數。

[GitHub 存放庫](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts)中也會提供 [NYC 計程車車程資料](http://chriswhong.com/open-data/foil_nyc_taxi/)案例特定的查詢範例。 這些查詢已經具備指定的資料結構描述，且準備好進行提交來執行。 最後一節也會討論使用者可以微調的參數，以改善 Hive 查詢的效能。

[!INCLUDE [cap-create-features-data-selector](../../../includes/cap-create-features-selector.md)]

這個 **功能表** 所連結的主題會說明如何在各種環境中建立資料的特徵。 此工作是 [Team Data Science Process (TDSP)](https://azure.microsoft.com/documentation/learning-paths/cortana-analytics-process/)中的一個步驟。

## <a name="prerequisites"></a>先決條件
本文假設您已經：

* 建立 Azure 儲存體帳戶。 如需指示，請參閱[建立 Azure 儲存體帳戶](../../storage/common/storage-create-storage-account.md#create-a-storage-account)
* 佈建含有 HDInsight 服務的自訂 Hadoop 叢集。  如需指示，請參閱 [自訂適用於進階分析的 Azure HDInsight Hadoop 叢集](customize-hadoop-cluster.md)。
* 已將資料上傳至 Azure HDInsight Hadoop 叢集中的 Hive 資料表。 如果沒有，則遵循[建立資料並載入到 Hive 資料表](move-hive-tables.md) ，先將資料上傳至 Hive 資料表。
* 啟用叢集的遠端存取。 如需指示，請參閱 [存取 Hadoop 叢集的前端節點](customize-hadoop-cluster.md)。

## <a name="hive-featureengineering"></a>特徵產生
在本節中，說明可以使用 Hive 查詢特性之數個方式的範例。 一旦產生額外功能之後，就可以將它們當成資料行新增至現有的資料表，或是建立具有其他功能和主索引鍵的新資料表 (然後與原始資料表聯結)。 以下是顯示的範例：

1. [以頻率為基礎的功能產生](#hive-frequencyfeature)
2. [二進位分類中類別變數的風險](#hive-riskfeature)
3. [從日期時間欄位擷取功能](#hive-datefeatures)
4. [從文字欄位擷取功能](#hive-textfeatures)
5. [計算 GPS 座標間的距離](#hive-gpsdistance)

### <a name="hive-frequencyfeature"></a>以頻率為基礎的特徵產生
計算類別變數層級的頻率，或是來自多個類別變數之特定層級組合的頻率，通常很實用。 使用者可以使用下列指令碼來計算這些頻率：

        select
            a.<column_name1>, a.<column_name2>, a.sub_count/sum(a.sub_count) over () as frequency
        from
        (
            select
                <column_name1>,<column_name2>, count(*) as sub_count
            from <databasename>.<tablename> group by <column_name1>, <column_name2>
        )a
        order by frequency desc;


### <a name="hive-riskfeature"></a>二元分類中類別變數的風險
在二元分類中，若使用的模型只會採用數值特徵，則必須將非數值類別變數轉換成數值特徵。 此轉換是透過使用數值風險來取代每個非數值層級來完成。 本節將說明一些計算類別變數風險值 (記錄機率) 的泛型 Hive 查詢。

        set smooth_param1=1;
        set smooth_param2=20;
        select
            <column_name1>,<column_name2>,
            ln((sum_target+${hiveconf:smooth_param1})/(record_count-sum_target+${hiveconf:smooth_param2}-${hiveconf:smooth_param1})) as risk
        from
            (
            select
                <column_nam1>, <column_name2>, sum(binary_target) as sum_target, sum(1) as record_count
            from
                (
                select
                    <column_name1>, <column_name2>, if(target_column>0,1,0) as binary_target
                from <databasename>.<tablename>
                )a
            group by <column_name1>, <column_name2>
            )b

這個範例會設定變數 `smooth_param1` 和 `smooth_param2`，以減緩從資料計算得來的風險值。 風險範圍介於-Inf 和 Inf 之間。 風險 > 0 表示目標等於 1 的機率大於 0.5。

計算出風險資料表之後，使用者就可以藉由將資料表聯結至風險資料表，來將風險值指派給該資料表。 Hive 聯結查詢已在上一節中提供。

### <a name="hive-datefeatures"></a>從日期時間欄位擷取特徵
Hive 會和一組 UDF 一起出現，用來處理日期時間欄位。 在 Hive 中，預設的日期時間格式是 'yyyy-MM-dd 00:00:00' (例如 '1970-01-01 12:21:32')。 本節會顯示擷取月份日期和來自日期時間欄位的月份範例，以及其他可將預設格式以外格式的日期時間字串轉換為預設格式的日期時間字串範例。

        select day(<datetime field>), month(<datetime field>)
        from <databasename>.<tablename>;

這個 Hive 查詢假設 *<datetime field>* 是預設的日期時間格式。

如果日期時間欄位不是預設格式，您需要先將日期時間欄位轉換為 Unix 時間戳記，然後將 Unix 時間戳記轉換為預設格式的日期時間字串。 將日期時間為預設格式之後，使用者就可以套用內嵌的日期時間 UDF 來擷取功能。

        select from_unixtime(unix_timestamp(<datetime field>,'<pattern of the datetime field>'))
        from <databasename>.<tablename>;

在此查詢中，如果 *<datetime field>* 的模式類似於 *03/26/2015 12:04:39*，則 *<pattern of the datetime field>'* 應為 `'MM/dd/yyyy HH:mm:ss'`。 若要進行測試，使用者可以執行

        select from_unixtime(unix_timestamp('05/15/2015 09:32:10','MM/dd/yyyy HH:mm:ss'))
        from hivesampletable limit 1;

佈建叢集時，這個查詢中的 *hivesampletable* 預設會預先安裝於所有 Azure HDInsight Hadoop 叢集中。

### <a name="hive-textfeatures"></a>從文字欄位擷取特徵
當 Hive 資料表具有一個文字欄位且其中包含以空格分隔的文字字串時，下列查詢便會擷取字串長度，以及字串中的字數。

        select length(<text field>) as str_len, size(split(<text field>,' ')) as word_num
        from <databasename>.<tablename>;

### <a name="hive-gpsdistance"></a>計算 GPS 座標組之間的距離
本節中提供的查詢可直接套用至「NYC 計程車車程資料」。 此查詢的目的是示範如何在 Hive 中套用內嵌的數學函式來產生功能。

這個查詢中所使用的欄位是上車與下車位置的 GPS 座標，其名稱為 *pickup\_longitude*、*pickup\_latitude*、*dropoff\_longitude* 和 *dropoff\_latitude*。 計算上車與下車座標間直線距離的查詢如下：

        set R=3959;
        set pi=radians(180);
        select pickup_longitude, pickup_latitude, dropoff_longitude, dropoff_latitude,
            ${hiveconf:R}*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
            *${hiveconf:pi}/180/2),2)-cos(pickup_latitude*${hiveconf:pi}/180)
            *cos(dropoff_latitude*${hiveconf:pi}/180)*pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2)))
            /sqrt(pow(sin((dropoff_latitude-pickup_latitude)*${hiveconf:pi}/180/2),2)
            +cos(pickup_latitude*${hiveconf:pi}/180)*cos(dropoff_latitude*${hiveconf:pi}/180)*
            pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2))) as direct_distance
        from nyctaxi.trip
        where pickup_longitude between -90 and 0
        and pickup_latitude between 30 and 90
        and dropoff_longitude between -90 and 0
        and dropoff_latitude between 30 and 90
        limit 10;

您可以在<a href="http://www.movable-type.co.uk/scripts/latlong.html" target="_blank">可移動的類型指令碼</a>網站 (作者為 Peter Lapisu) 上找到計算兩個 GPS 座標間之距離的數學方程式。 在此 JavaScript 中，函式 `toRad()` 只是 *lat_or_lon*pi/180*，可將角度轉換為弧度。 此處的 *lat_or_lon* 為緯度或經度。 由於 Hive 不提供函式 `atan2`，但提供函式 `atan`，因此 `atan2` 函式是由上述 Hive 查詢中的 `atan` 函式以 <a href="http://en.wikipedia.org/wiki/Atan2" target="_blank">Wikipedia</a> 中提供的定義來實作。

![建立工作區](./media/create-features-hive/atan2new.png)

您可以在 <a href="https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-MathematicalFunctions" target="_blank">Apache Hive wiki</a> 上的**內建函式**一節中找到 Hive 內嵌 UDF 的完整清單。  

## <a name="tuning"></a> 進階主題：微調 Hive 參數以提升查詢速度
Hive 叢集的預設參數設定可能不適合 Hive 查詢以及查詢正在處理的資料。 本節將討論一些使用者可以微調以提升 Hive 查詢效能的參數。 使用者需要在處理資料的查詢之前新增參數微調查詢。

1. **Java 堆積空間**：對於涉及聯結大型資料集或處理長記錄的查詢，常見的一項錯誤是**堆積空間不足**。 透過將參數 *mapreduce.map.java.opts* 和 *mapreduce.task.io.sort.mb* 設定為所需的值可避免此錯誤。 下列是一個範例：
   
        set mapreduce.map.java.opts=-Xmx4096m;
        set mapreduce.task.io.sort.mb=-Xmx1024m;

    這個參數會配置 4 GB 記憶體給 Java 堆積空間，也會藉由配置更多記憶體來使排序更有效率。 如果發生任何與堆積空間相關的工作失敗錯誤，那麼使用這些配置會是個好主意。

1. **DFS 區塊大小**：這個參數會設定檔案系統所儲存的最小資料單位。 例如，如果 DFS 區塊大小為 128 MB，則大小小於和等於 128 MB 的任何資料都會儲存在單一區塊中。 系統會分配額外的區塊給大於 128 MB 的資料。 
2. 選擇小的區塊大小會在 Hadoop 中造成極大的負荷，因為名稱節點必須處理更多要求，以尋找與檔案有關的相關區塊。 在處理 GB (或更大型) 資料時，建議的設定如下：

        set dfs.block.size=128m;

2. **將 Hive 中的聯結作業最佳化**：儘管 Map/Reduce 架構中的聯結作業通常是在縮減階段執行，但有時可藉由在對應階段中排程聯結 (亦稱為 "mapjoin") 來得到大量的收穫。 若要在適當時機引導 Hive 執行這個動作，請設定：
   
       set hive.auto.convert.join=true;

3. **指定 Hive 的對應程式數目**：儘管 Hadoop 允許使用者設定縮減程式的數目，但使用者通常不會設定對應程式的數目。 允許對這個數目進行某種程度控制的技巧是選擇 Hadoop 變數 *mapred.min.split.size* 和 *mapred.max.split.size*，因為每個對應工作的大小由下列項目決定：
   
        num_maps = max(mapred.min.split.size, min(mapred.max.split.size, dfs.block.size))
   
    一般而言，預設值如下：
    
    - *mapred.min.split.size* 的預設值為 0
    - *mapred.max.split.size* 的預設值為 **Long.MAX** 
    - *dfs.block.size* 的預設值為 64 MB。

    誠如所見，若指定了資料大小，則藉由「設定」這些參數來微調它們，讓我們能夠微調所使用的對應程式數目。

4. 以下是最佳化 Hive 效能的其他數個更**進階的選項**。 這些選項讓您能夠設定配置的記憶體來對應和縮減工作，而且在調整效能時非常實用。 請記住，*mapreduce.reduce.memory.mb* 不能大於 Hadoop 叢集中每個背景工作角色節點的實際記憶體大小。
   
        set mapreduce.map.memory.mb = 2048;
        set mapreduce.reduce.memory.mb=6144;
        set mapreduce.reduce.java.opts=-Xmx8192m;
        set mapred.reduce.tasks=128;
        set mapred.tasktracker.reduce.tasks.maximum=128;

