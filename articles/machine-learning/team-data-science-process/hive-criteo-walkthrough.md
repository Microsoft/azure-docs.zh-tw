---
title: Team Data Science Process 實務 - 在 1 TB 資料集上使用 Azure HDInsight Hadoop 叢集 | Microsoft Docs
description: 對採用 HDInsight Hadoop 叢集來建置和部署使用大型 (1 TB) 公開可用資料集模型的端對端案例使用 Team Data Science Process
services: machine-learning,hdinsight
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 72d958c4-3205-49b9-ad82-47998d400d2b
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/29/2017
ms.author: deguhath
ms.openlocfilehash: 4c368c3f06347b1164731d056a7341bdabb759b4
ms.sourcegitcommit: 3c3488fb16a3c3287c3e1cd11435174711e92126
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/08/2018
ms.locfileid: "34837339"
---
# <a name="the-team-data-science-process-in-action---using-an-azure-hdinsight-hadoop-cluster-on-a-1-tb-dataset"></a>Team Data Science Process 實務 - 在 1 TB 資料集上使用 Azure HDInsight Hadoop 叢集

本逐步解說示範如何在端對端案例中使用 Team Data Science Process 搭配 [Azure HDInsight Hadoop 叢集](https://azure.microsoft.com/services/hdinsight/)進行儲存、探索、特徵工程設計，並從其中一個公開可用的 [Criteo](http://labs.criteo.com/downloads/download-terabyte-click-logs/) 資料集縮減取樣資料。 其使用 Azure Machine Learning 在此資料上建置二進位的分類模型。 也會顯示如何將其中一個模型發佈為 Web 服務。

此外，也可以使用 IPython Notebook 來完成此逐步解說中說明的工作。 想要嘗試這種方法的使用者，應該查閱 [使用 Hive ODBC 連線的 Criteo 逐步解說](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-hive-walkthrough-criteo.ipynb) 主題。

## <a name="dataset"></a>Criteo 資料集說明
Criteo 資料是點選預測的資料集，大約是 370 GB 的 gzip 壓縮 TSV 檔案 (~1.3 TB 未壓縮)，包含超過 43 億筆記錄。 它取自 [Criteo](http://labs.criteo.com/downloads/download-terabyte-click-logs/)提供的 24 天點選資料。 為了方便資料科學家使用，我們已將可供試驗的資料解壓縮。

在此資料集中的每一筆記錄包含 40 個資料行：

* 第一個資料行是標籤資料行，指出使用者是否按一下 **加入** (值 1) 或未按一下 (值 0)
* 接下來 13 個資料行是數字，以及
* 最後 26 個資料行是類別資料行

資料行為匿名，並使用一系列的列舉名稱："Col1" (針對標籤資料行) 至 "Col40" (針對最後一個分類資料行)。            

以下是來自這個資料集的兩個觀察 (資料列) 的前 20 個資料行的摘錄：

    Col1    Col2    Col3    Col4    Col5    Col6    Col7    Col8    Col9    Col10    Col11    Col12    Col13    Col14    Col15            Col16            Col17            Col18            Col19        Col20

    0       40      42      2       54      3       0       0       2       16      0       1       4448    4       1acfe1ee        1b2ff61f        2e8b2631        6faef306        c6fc10d3    6fcd6dcb           
    0               24              27      5               0       2       1               3       10064           9a8cb066        7a06385f        417e6103        2170fc56        acf676aa    6fcd6dcb                      

此資料集中的數值及分類資料行中有遺漏值。 我們會說明用來處理遺漏值的簡單方法。 資料的其他詳細資料會在將它們儲存成 Hive 資料表時加以說明。

**定義：** *點選率 (CTR)：* 這是資料中點選的百分比。 在此 Criteo 資料集中，CTR 是大約 3.3%或 0.033。

## <a name="mltasks"></a>預測工作的範例
本逐步解說將討論兩個範例預測問題：

1. **二進位分類**：預測使用者是否按下加入：
   
   * 類別 0：未按一下
   * 類別 1：按一下
2. **迴歸**：預測使用者按一下廣告機率的功能。

## <a name="setup"></a>為資料科學設定 HDInsight Hadoop 叢集
**附註：** 這通常是**管理**工作。

設定 Azure 資料科學環境，用於使用 HDInsight 叢集以三個步驟建置預測性的分析解決方案：

1. [建立儲存體帳戶](../../storage/common/storage-create-storage-account.md)：此儲存體帳戶用來將資料儲存在 Azure Blob 儲存體中。 HDInsight 叢集中使用的資料會儲存在這裡。
2. [自訂適用於資料科學的 Azure HDInsight Hadoop 叢集](customize-hadoop-cluster.md)：這個步驟將會建立已在所有節點上安裝 64 位元 Anaconda Python 2.7 的 Azure HDInsight Hadoop 叢集。 自訂 HDInsight 叢集時有兩個需完成的重要步驟 (如本主題所述)。
   
   * 建立時您必須將步驟 1 中建立的儲存體帳戶與 HDInsight 叢集連結。 此儲存體帳戶用於存取可以在叢集內處理的資料。
   * 建立後，您必須對叢集的前端節點啟用遠端存取。 請記住您在此處指定的遠端存取認證 (與建立時指定叢集的不同)：您需要認證以完成下列程序。
3. [建立 Azure ML 工作區](../studio/create-workspace.md)：此 Azure Machine Learning 工作區用來在初始資料瀏覽之後建置機器學習模型，並且在 HDInsight 叢集上下載取樣。

## <a name="getdata"></a>取得並從公用來源取用資料
[Criteo](http://labs.criteo.com/downloads/download-terabyte-click-logs/) 資料集可以透過按一下連結、接受使用條款並提供名稱來存取。 其外觀的快照如下所示：

![接受 Criteo 條款](./media/hive-criteo-walkthrough/hLxfI2E.png)

按一下 [繼續下載]  來閱讀資料集的相關資訊和它的可用性。

資料位於公用 [Azure Blob 儲存體](../../storage/blobs/storage-dotnet-how-to-use-blobs.md)位置：wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/。 "wasb" 指的是 Azure Blob 儲存體位置。 

1. 這個公用 Blob 儲存體中的資料是由所解壓縮資料的三個子資料夾所組成。
   
   1. 子資料夾 *raw/count/* 包含前 21 天的資料 - 從 day\_00 到 day\_20
   2. 子資料夾 *raw/train/* 由單一天 day\_21 的資料組成
   3. 子資料夾 *raw/test/* 由兩天 day\_22 和 day\_23 的資料組成
2. 對於想要從原始 gzip 資料開始的使用者，這些也可以在主要資料夾 *raw/* 取得，形式為 day_NN.gz，其中 NN 從 00 到 23。

本逐步解說中稍後會在我們建立 Hive 資料表時說明存取、瀏覽和模型化此資料而不需要任何本機下載的另一種方法。

## <a name="login"></a>登入到叢集前端節點
若要登入叢集的前端節點，請使用 [Azure 入口網站](https://ms.portal.azure.com) 來找出叢集。 在左側按一下 HDInsight 大象圖示，然後按兩下叢集的名稱。 瀏覽至 [組態]  索引標籤上，在頁面底部的 [連接] 圖示上連按兩下，然後在提示時輸入您的遠端存取認證。 這會帶您前往叢集的前端節點。

以下是典型的第一次登入叢集前端節點時看起來的外觀：

![登入叢集](./media/hive-criteo-walkthrough/Yys9Vvm.png)

左邊是「Hadoop 命令列」，這是我們進行資料瀏覽的主要位置。 請注意兩個實用的 URL - "Hadoop Yarn Status" 和 "Hadoop Name Node"。 Yarn 狀態 URL 會顯示工作進度，而名稱節點 URL 則會提供叢集組態的相關詳細資料。

您現在已設定並準備好開始逐步解說的第一個部分：使用 Hive 的資料瀏覽和為 Azure Machine Learning 備妥資料。

## <a name="hive-db-tables"></a> 建立 Hive 資料庫和資料表
若要為我們的 Criteo 資料集建立 Hive 資料表，請在前端節點的桌面上開啟 [Hadoop 命令列]，然後輸入以下命令來進入 Hive 目錄

    cd %hive_home%\bin

> [!NOTE]
> 請從 Hive bin/ 目錄提示字元執行本逐步解說中的所有 Hive 命令。 如此可自動處理路徑相關問題。 您可以交替使用詞彙「Hive 目錄提示」、「Hive bin/ 目錄提示」和「Hadoop 命令列」。
> 
> [!NOTE]
> 若要執行任何 Hive 查詢，您永遠可以使用下列命令︰
> 
> 

        cd %hive_home%\bin
        hive

Hive REPL "hive >" 出現記號後，只需剪下並貼上查詢即可執行。

下列程式碼會建立資料庫 "criteo"，然後產生 4 個資料表：

* 建置於 day\_00 到 day\_20 「用於產生計數的資料表」、
* 建置於 day\_21「用來做為訓練資料集的資料表」，以及
* 分別建置於 day\_22 和 day\_23 的兩個「用來做為測試資料集的資料表」。

將測試資料集分成兩個不同的資料表，因為其中一天為假日。 目標在於判斷此模型是否可以從點擊率來偵測假日與非假日之間的差異。

下面顯示指令碼 [sample&#95;hive&#95;create&#95;criteo&#95;database&#95;and&#95;tables.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_create_criteo_database_and_tables.hql)，以方便使用：

    CREATE DATABASE IF NOT EXISTS criteo;
    DROP TABLE IF EXISTS criteo.criteo_count;
    CREATE TABLE criteo.criteo_count (
    col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE LOCATION 'wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/count';

    DROP TABLE IF EXISTS criteo.criteo_train;
    CREATE TABLE criteo.criteo_train (
    col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE LOCATION 'wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/train';

    DROP TABLE IF EXISTS criteo.criteo_test_day_22;
    CREATE TABLE criteo.criteo_test_day_22 (
    col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE LOCATION 'wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/test/day_22';

    DROP TABLE IF EXISTS criteo.criteo_test_day_23;
    CREATE TABLE criteo.criteo_test_day_23 (
    col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE LOCATION 'wasb://criteo@azuremlsampleexperiments.blob.core.windows.net/raw/test/day_23';

所有這些資料表都是外部的，因此您可以只是指向其 Azure Blob 儲存體 (wasb) 位置。

**有兩種方式可執行任何 Hive 查詢：**

1. **使用 Hive REPL 命令列**：第一個是發出 "hive" 命令，複製查詢並在 Hive REPL 命令列貼上。 若要這樣做，請執行：
   
        cd %hive_home%\bin
        hive
   
     現在，在 REPL 命令列，剪下和貼上查詢即可執行。
2. **將查詢儲存到檔案，並執行命令**：第二個是要將查詢儲存為 .hql 檔案 ([sample&#95;hive&#95;create&#95;criteo&#95;database&#95;and&#95;tables.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_create_criteo_database_and_tables.hql)) 然後發出下列命令以執行查詢：
   
        hive -f C:\temp\sample_hive_create_criteo_database_and_tables.hql

### <a name="confirm-database-and-table-creation"></a>確認資料庫和資料表的建立
接下來，從 Hive bin/ 目錄提示使用下列命令，確認建立資料庫：

        hive -e "show databases;"

這會提供：

        criteo
        default
        Time taken: 1.25 seconds, Fetched: 2 row(s)

這會確認建立新資料庫，也就是 "criteo"。

若要查看已建立的資料表，只要從 Hive bin/ 目錄提示發出以下命令：

        hive -e "show tables in criteo;"

然後，您應會看見下列輸出：

        criteo_count
        criteo_test_day_22
        criteo_test_day_23
        criteo_train
        Time taken: 1.437 seconds, Fetched: 4 row(s)

## <a name="exploration"></a> 在 Hive 中的資料瀏覽
您現在已準備好在 Hive 中執行一些基本的資料瀏覽。 您可從計算訓練中的範例數目和測試資料的資料表開始。

### <a name="number-of-train-examples"></a>訓練範例的數目
[sample&#95;hive&#95;count&#95;train&#95;table&#95;examples.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_count_train_table_examples.hql) 的內容如下所示：

        SELECT COUNT(*) FROM criteo.criteo_train;

這會產生：

        192215183
        Time taken: 264.154 seconds, Fetched: 1 row(s)

或者，使用者也可以從 Hive bin/ 目錄提示發出下列命令：

        hive -f C:\temp\sample_hive_count_criteo_train_table_examples.hql

### <a name="number-of-test-examples-in-the-two-test-datasets"></a>兩個測試資料集中測試範例的數目
現在計算兩個測試資料集中的範例數目。 [sample&#95;hive&#95;count&#95;criteo&#95;test&#95;day&#95;22&#95;table&#95;examples.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_count_criteo_test_day_22_table_examples.hql) 的內容如下所示：

        SELECT COUNT(*) FROM criteo.criteo_test_day_22;

這會產生：

        189747893
        Time taken: 267.968 seconds, Fetched: 1 row(s)

一如往常，您也可以透過發出命令，從 Hive bin/ 目錄提示呼叫指令碼：

        hive -f C:\temp\sample_hive_count_criteo_test_day_22_table_examples.hql

最後，請檢查以 day\_23 為基礎的測試資料集內的測試範例的數目。

要執行此動作的命令類似剛才顯示的命令 (請參閱 [sample&#95;hive&#95;count&#95;criteo&#95;test&#95;day&#95;23&#95;examples.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_count_criteo_test_day_23_examples.hql))：

        SELECT COUNT(*) FROM criteo.criteo_test_day_23;

這會提供：

        178274637
        Time taken: 253.089 seconds, Fetched: 1 row(s)

### <a name="label-distribution-in-the-train-dataset"></a>訓練資料集中的標籤分佈
訓練資料集中的標籤分佈值得一提。 為了看到此結果，請顯示 [sample&#95;hive&#95;criteo&#95;label&#95;distribution&#95;train&#95;table.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_label_distribution_train_table.hql) 的內容：

        SELECT Col1, COUNT(*) AS CT FROM criteo.criteo_train GROUP BY Col1;

這會產生標籤分佈：

        1       6292903
        0       185922280
        Time taken: 459.435 seconds, Fetched: 2 row(s)

請注意，正數標籤的百分比為大約 3.3% (與原始資料集一致)。

### <a name="histogram-distributions-of-some-numeric-variables-in-the-train-dataset"></a>訓練資料集中一些數值變數的長條圖分佈
您可以使用 Hive 的原生 "histogram\_numeric" 函式，來了解數值變數分佈的外觀。 以下是 [sample&#95;hive&#95;criteo&#95;histogram&#95;numeric.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_histogram_numeric.hql) 的內容：

        SELECT CAST(hist.x as int) as bin_center, CAST(hist.y as bigint) as bin_height FROM
            (SELECT
            histogram_numeric(col2, 20) as col2_hist
            FROM
            criteo.criteo_train
            ) a
            LATERAL VIEW explode(col2_hist) exploded_table as hist;

這會產生下列：

        26      155878415
        2606    92753
        6755    22086
        11202   6922
        14432   4163
        17815   2488
        21072   1901
        24113   1283
        27429   1225
        30818   906
        34512   723
        38026   387
        41007   290
        43417   312
        45797   571
        49819   428
        53505   328
        56853   527
        61004   160
        65510   3446
        Time taken: 317.851 seconds, Fetched: 20 row(s)

橫向檢視 - Hive 中的切割組合是用來產生類似 SQL 的輸出，而不是一般的清單。 請注意，在此資料表中，第一個資料行對應至 bin 中心，而第二個對應至 bin 頻率。

### <a name="approximate-percentiles-of-some-numeric-variables-in-the-train-dataset"></a>訓練資料集中一些數值變數的近似百分比
數值變數是近似百分比的計算也值得一提。 Hive 的原生 "percentile\_approx" 會為我們執行此動作。 [sample&#95;hive&#95;criteo&#95;approximate&#95;percentiles.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_approximate_percentiles.hql) 的內容為：

        SELECT MIN(Col2) AS Col2_min, PERCENTILE_APPROX(Col2, 0.1) AS Col2_01, PERCENTILE_APPROX(Col2, 0.3) AS Col2_03, PERCENTILE_APPROX(Col2, 0.5) AS Col2_median, PERCENTILE_APPROX(Col2, 0.8) AS Col2_08, MAX(Col2) AS Col2_max FROM criteo.criteo_train;

這會產生：

        1.0     2.1418600917169246      2.1418600917169246    6.21887086390288 27.53454893115633       65535.0
        Time taken: 564.953 seconds, Fetched: 1 row(s)

百分比的分佈通常與任何數值變數的長條圖分佈相關。         

### <a name="find-number-of-unique-values-for-some-categorical-columns-in-the-train-dataset"></a>尋找訓練資料集中的某些類別資料行的唯一值數目
繼續進行資料瀏覽，我們發現，對於某些類別資料行，他們使用了唯一值數目。 為了執行此動作，請顯示 [sample&#95;hive&#95;criteo&#95;unique&#95;values&#95;categoricals.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_unique_values_categoricals.hql) 的內容：

        SELECT COUNT(DISTINCT(Col15)) AS num_uniques FROM criteo.criteo_train;

這會產生：

        19011825
        Time taken: 448.116 seconds, Fetched: 1 row(s)

請注意，Col15 有 1900 萬個唯一值！ 使用貝氏方法，像是「一個有效編碼」來編碼這類高維度類別變數不可行。 特別是，我們將說明並示範稱為[以計數學習](http://blogs.technet.com/b/machinelearning/archive/2015/02/17/big-learning-made-easy-with-counts.aspx)的強大、穩健技術，以有效率地解決此問題。

最後來看一下一些其他類別資料行的唯一值數目。 [sample&#95;hive&#95;criteo&#95;unique&#95;values&#95;multiple&#95;categoricals.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_unique_values_multiple_categoricals.hql) 的內容為：

        SELECT COUNT(DISTINCT(Col16)), COUNT(DISTINCT(Col17)),
        COUNT(DISTINCT(Col18), COUNT(DISTINCT(Col19), COUNT(DISTINCT(Col20))
        FROM criteo.criteo_train;

這會產生：

        30935   15200   7349    20067   3
        Time taken: 1933.883 seconds, Fetched: 1 row(s)

請注意，除了 Col20 以外，所有其他資料行都有許多唯一值。

### <a name="co-occurrence-counts-of-pairs-of-categorical-variables-in-the-train-dataset"></a>定型資料集中成對類別變數的共生計數

定型資料集中成對類別變數的共生計數也很有趣。 這可以使用 [sample&#95;hive&#95;criteo&#95;paired&#95;categorical&#95;counts.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_paired_categorical_counts.hql) 中的程式碼來判斷：

        SELECT Col15, Col16, COUNT(*) AS paired_count FROM criteo.criteo_train GROUP BY Col15, Col16 ORDER BY paired_count DESC LIMIT 15;

將依其發生計數反轉順序，並在此情況下查看前 15 個。 這會提供：

        ad98e872        cea68cd3        8964458
        ad98e872        3dbb483e        8444762
        ad98e872        43ced263        3082503
        ad98e872        420acc05        2694489
        ad98e872        ac4c5591        2559535
        ad98e872        fb1e95da        2227216
        ad98e872        8af1edc8        1794955
        ad98e872        e56937ee        1643550
        ad98e872        d1fade1c        1348719
        ad98e872        977b4431        1115528
        e5f3fd8d        a15d1051        959252
        ad98e872        dd86c04a        872975
        349b3fec        a52ef97d        821062
        e5f3fd8d        a0aaffa6        792250
        265366bf        6f5c7c41        782142
        Time taken: 560.22 seconds, Fetched: 15 row(s)

## <a name="downsample"></a> 對 Azure Machine Learning 的資料集縮減取樣
探索資料集並示範了如何對任何變數 (包括各種組合) 的這類探索進行的動作之後，請對資料集縮減取樣，以便在 Azure Machine Learning 中建置模型。 回顧一下問題的重點在於：假設有一組範例屬性 (從 Col2 到 Col40 的功能值)，請預測 Col1 是否為 0 (未按一下) 或 1 (按一下)。

為了對訓練和測試資料集縮減取樣至原始大小的 1%，請使用 Hive 的原生 RAND() 函式。 下一個指令碼 ([sample&#95;hive&#95;criteo&#95;downsample&#95;train&#95;dataset.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_downsample_train_dataset.hql)) 會為訓練資料集執行此動作：

        CREATE TABLE criteo.criteo_train_downsample_1perc (
        col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
        LINES TERMINATED BY '\n'
        STORED AS TEXTFILE;

        ---Now downsample and store in this table

        INSERT OVERWRITE TABLE criteo.criteo_train_downsample_1perc SELECT * FROM criteo.criteo_train WHERE RAND() <= 0.01;

這會產生：

        Time taken: 12.22 seconds
        Time taken: 298.98 seconds

[sample&#95;hive&#95;criteo&#95;downsample&#95;test&#95;day&#95;22&#95;dataset.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_downsample_test_day_22_dataset.hql)指令碼會為測試資料 day\_22 執行此動作：

        --- Now for test data (day_22)

        CREATE TABLE criteo.criteo_test_day_22_downsample_1perc (
        col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
        LINES TERMINATED BY '\n'
        STORED AS TEXTFILE;

        INSERT OVERWRITE TABLE criteo.criteo_test_day_22_downsample_1perc SELECT * FROM criteo.criteo_test_day_22 WHERE RAND() <= 0.01;

這會產生：

        Time taken: 1.22 seconds
        Time taken: 317.66 seconds


最後，[sample&#95;hive&#95;criteo&#95;downsample&#95;test&#95;day&#95;23&#95;dataset.hql](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_criteo_downsample_test_day_23_dataset.hql)指令碼會為測試資料 day\_23 執行此動作：

        --- Finally test data day_23
        CREATE TABLE criteo.criteo_test_day_23_downsample_1perc (
        col1 string,col2 double,col3 double,col4 double,col5 double,col6 double,col7 double,col8 double,col9 double,col10 double,col11 double,col12 double,col13 double,col14 double,col15 string,col16 string,col17 string,col18 string,col19 string,col20 string,col21 string,col22 string,col23 string,col24 string,col25 string,col26 string,col27 string,col28 string,col29 string,col30 string,col31 string,col32 string,col33 string,col34 string,col35 string,col36 string,col37 string,col38 string,col39 string,col40 srical feature; tring)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
        LINES TERMINATED BY '\n'
        STORED AS TEXTFILE;

        INSERT OVERWRITE TABLE criteo.criteo_test_day_23_downsample_1perc SELECT * FROM criteo.criteo_test_day_23 WHERE RAND() <= 0.01;

這會產生：

        Time taken: 1.86 seconds
        Time taken: 300.02 seconds

以此方式，您已準備好使用縮減取樣的訓練和測試資料集，在 Azure Machine Learning 模型中建置模型。

在繼續 Azure Machine Learning 之前，最後有一個重要元件是考量計數資料表。 在下一個子節中，將會討論計數資料表的一些細節。

## <a name="count"></a> 計數資料表的簡短討論
如您所見，數個類別變數有極高的維度性。 逐步解說中會提供稱為[以計數學習](http://blogs.technet.com/b/machinelearning/archive/2015/02/17/big-learning-made-easy-with-counts.aspx)的強大技術，以便利用有效率且穩健的方式編碼這些變數。 提供的連結中說明此技術的詳細資訊。

[!NOTE]
>在本逐步解說中，重點在於使用計數資料表來產生高維度類別功能的精簡表示法。 這不是編碼類別功能的唯一方式；如需有關其他技術的詳細資訊，有興趣的使用者可以查看 [one-hot-encoding](http://en.wikipedia.org/wiki/One-hot) 和[特徵雜湊](http://en.wikipedia.org/wiki/Feature_hashing)。
>

為了對計數資料建置計數資料表，請使用 raw/count 資料夾中的資料。 在模型建構區段中，指示使用者如何從頭建置類別功能的這些計數資料表，或者可以使用預先建置的計數資料表進行其探索。 在下面內容中，提到「預先建置的計數資料表」時，表示使用我們提供的計數資料表。 有關如何存取這些資料表的詳細指示會在下一節提供。

## <a name="aml"></a> 使用 Azure Machine Learning 建置模型
在 Azure Machine Learning 建置模型的程序遵循下列步驟：

1. [從 Hive 資料表取得資料到 Azure Machine Learning](#step1)
2. [建立實驗︰清除資料，並使用計數資料表特性化](#step2)
3. [建立、訓練和評分模型](#step3)
4. [評估模型](#step4)
5. [將模型發佈為 Web 服務](#step5)

您現在已準備好在 Azure Machine Learning Studio 中建置模型。 我們縮減取樣的資料會在叢集中儲存為 Hive 資料表。 使用 Azure Machine Learning 的 [匯入資料] 模組來讀取此資料。 可存取此叢集之儲存體帳戶的認證會在下面提供。

### <a name="step1"></a> 步驟 1：使用「匯入資料」模組從 Hive 資料表取得資料並匯入到 Azure Machine Learning，然後選取它來進行機器學習實驗
藉由選取 [+ 新增] -> [實驗] -> [空白實驗] 開始。 接著，從左上方的 [搜尋]  方塊，搜尋「匯入資料」。 將 [匯入資料]  模組拖放到實驗畫布 (螢幕的中間部分) 上，以將模組用於資料存取。

這是從 Hive 資料表取得資料時 **匯入資料** 看起來的樣子：

![「匯入資料」取得資料](./media/hive-criteo-walkthrough/i3zRaoj.png)

針對 **匯入資料** 模組，圖形中提供的參數值都只是您需要提供之該類值的範例。 以下是一些有關如何填寫 **匯入資料** 模組之參數集的一般指引。

1. 對 **資料來源**
2. 在 [Hive 資料庫查詢] 方塊中，簡單的 SELECT * FROM <your\_database\_name.your\_table\_name> - 就已經足夠。
3. **Hcatalog 伺服器 URI**：如果您的叢集是 "abc"，那麼就是：https://abc.azurehdinsight.net
4. **Hadoop 使用者帳戶名稱**：委任叢集時選擇的使用者名稱。 (非遠端存取使用者名稱！)
5. **Hadoop 使用者帳戶密碼**：委任叢集時選擇之使用者名稱的密碼。 (非遠端存取密碼！)
6. **輸出資料的位置**：選擇 "Azure"
7. **Azure 儲存體帳戶名稱**：和叢集相關聯的儲存體帳戶
8. **Azure 儲存體帳戶金鑰**：和叢集相關聯的儲存體帳戶金鑰。
9. **Azure 容器名稱**：如果叢集名稱是 "abc"，則通常就是 "abc"。

在 **匯入資料** 完成資料取得後 (您會在模組上看到綠色勾號)，請將此資料儲存為「資料集」(使用您選擇的名稱)。 看起來像這樣：

![「匯入資料」儲存資料](./media/hive-criteo-walkthrough/oxM73Np.png)

在 **匯入資料** 模組的輸出連接埠上按一下滑鼠右鍵。 這會顯示 [另存為資料集] 選項和 [視覺化] 選項。 **視覺化** 選項，如果按下，會顯示資料 100 個資料列，以及在右側面板顯示某些實用的摘要統計資料。 若要儲存資料，只要選取 [ **另存為資料集** ] 並遵循指示。

若要選取已儲存的資料集，以用於機器學習實驗，請使用下圖所示的 [搜尋]  方塊找出資料集。 然後只要輸入您提供給資料集的部分名稱即可存取它，並拖曳至主面板中。 將它拖放到主面板中，選取它來用於機器學習模型建構。

![將資料集拖曳到主要面板中](./media/hive-criteo-walkthrough/cl5tpGw.png)

> [!NOTE]
> 為訓練和測試資料集執行這項操作。 此外，請記住要使用您為此目的提供的資料庫名稱和資料表名稱。 在圖中所使用的值僅供說明之用。
> 
> 

### <a name="step2"></a> 步驟 2：在 Azure Machine Learning 中建立簡單的實驗來預測按一下/未按一下
我們的 Azure ML 實驗看起來如下所示：

![Machine Learning 實驗](./media/hive-criteo-walkthrough/xRpVfrY.png)

現在檢查這項實驗的關鍵元件。 先將已儲存的訓練和測試資料集拖曳至我們的實驗畫布上。

#### <a name="clean-missing-data"></a>清除遺漏的資料
**清除遺漏的資料**模組的功能如其名稱示：以使用者指定的方式清除遺漏的資料。 查看此模組可看到這個：

![清除遺漏的資料](./media/hive-criteo-walkthrough/0ycXod6.png)

在此，選擇將所有的遺漏值取代為 0。 還有其他選項，可以藉由查看模組中的下拉式清單看到。

#### <a name="feature-engineering-on-the-data"></a>資料上的功能工程
大型資料集的部分類別功能可以有數百萬的唯一值。 使用像是一個有效編碼的單純方法來表示高維度類別功能是完全不可行的。 本逐步解說示範如何透過内建的 Azure Machine Learning 模組來使用計數功能產生這些高維度類別變數的壓縮表示法。 最後結果是較小的模型、更快速的定型時間與相當於使用其他技術的效能度量。

##### <a name="building-counting-transforms"></a>建置計數轉換
若要建置計數功能，請使用 Azure Machine Learning 中可使用的 [建置計數轉換] 模組。 此模組如下所示：

![建置計數轉換模組](./media/hive-criteo-walkthrough/e0eqKtZ.png)
![建置計數轉換模組](./media/hive-criteo-walkthrough/OdDN0vw.png)

> [!IMPORTANT] 
> 在 [計數資料行] 方塊中，輸入您要在其中執行計數的資料行。 一般來說，這些是 (如上所述) 高維度類別資料行。 請記住 Criteo 資料集有 26 個類別資料行：從 Col15 至 Col40。 在這裡，依據這些資料行，並為其提供索引 (從 15 到 40，並以逗號分隔，如下所示)。
> 

若要使用 MapReduce 模式中的模組 (適用於大型資料集)，您必須存取 HDInsight Hadoop 叢集 (用於功能探索的叢集可以重複用於此目的) 和它的認證。 前面的圖表說明填入的值看起來如何 (將提供做說明用途的值取代為您自己使用案例的相關值)。

![模組參數](./media/hive-criteo-walkthrough/05IqySf.png)

上圖說明輸入 blob 位置的輸入方式。 這個位置會保留資料以在其中建置計數資料表。

此模組完成執行時，稍後在模組上以滑鼠右鍵按一下，並選取 [儲存為轉換] 選項以儲存轉換：

![「另存為轉換」選項](./media/hive-criteo-walkthrough/IcVgvHR.png)

在如上所示的實驗架構中，資料集 "ytransform2" 會精確地對應已儲存的計數轉換。 針對這項實驗的其餘部分，假設讀取器在某些資料上使用 [建置計數轉換] 模組以產生計數，然後使用這些計數在訓練和測試資料集上產生計數功能。

##### <a name="choosing-what-count-features-to-include-as-part-of-the-train-and-test-datasets"></a>選擇要將哪些計數功能納為訓練和測試資料集的一部分
一旦備妥計數轉換時，使用者可以使用 [修改計數資料表參數] 模組，選擇要將哪些功能納入其訓練和測試資料集。 為了完整性，這個模組會在此顯示。 但為了簡單起見，請不要真的在實驗中使用它。

![修改計數資料表參數](./media/hive-criteo-walkthrough/PfCHkVg.png)

如您所見，在此情況下，會使用對數機率並忽略撤退資料行。 您也可以設定參數，例如記憶體回收 bin 臨界值，要新增多少虛擬優先範例才能平滑處理，以及是否使用任何 Laplacian 雜訊。 這些都是進階的功能，而且要注意的是對此類功能產生的新手使用者而言，預設值是個好起點。

##### <a name="data-transformation-before-generating-the-count-features"></a>產生計數功能之前的資料轉換
現在焦點在於一個重點，在實際產生計數功能之前要先轉換我們的訓練和測試資料。 請注意，將計數轉換套用至資料之前，已經使用了兩個 [執行 R 指令碼] 模組。

![執行 R 指令碼模組](./media/hive-criteo-walkthrough/aF59wbc.png)

以下是第一個 R 指令碼：

![第一個 R 指令碼](./media/hive-criteo-walkthrough/3hkIoMx.png)

這個 R 指令碼將資料行重新命名為 "Col1" 到 "Col40" 等名稱。 這是因為計數轉換必須要有這種格式的名稱。

第二個 R 指令碼會藉由降低取樣負類別來平衡正和負類別之間的散發 (分別為類別 1 和 0)。 此處的 R 指令碼會示範如何執行這個動作：

![第二個 R 指令碼](./media/hive-criteo-walkthrough/91wvcwN.png)

在這個簡單的 R 指令碼中，"pos\_neg\_ratio" 用於設定正與負類別之間的平衡數目。 這是很重要的動作，因為改善類別失衡通常會具有效能優勢，可處理類別散發扭曲的分類問題 (請記得在此案例中，您有 3.3% 正類別和 96.7% 負類別)。

##### <a name="applying-the-count-transformation-on-our-data"></a>在我們的資料上套用計數轉換
最後，您可以使用 [套用轉換] 模組，將計數轉換套用至訓練和測試資料集。 此模組會將儲存的計數轉換當成輸入，將訓練或測試資料集當成其他輸入，然後利用計數功能傳回資料。 如下所示：

![套用轉換模組](./media/hive-criteo-walkthrough/xnQvsYf.png)

##### <a name="an-excerpt-of-what-the-count-features-look-like"></a>計數功能的摘錄
它對於查看我們的案例中有什麼樣的計數功能很有啟發性。 以下是其摘錄：

![計數功能](./media/hive-criteo-walkthrough/FO1nNfw.png)

此摘錄說明對於依賴的資料行，除了任何相關輪詢之外，您還會取得計數和對數機率。

您現在已準備好使用這些已轉換的資料集建置 Azure Machine Learning 模型。 下一節說明如何完成此作業。

### <a name="step3"></a> 步驟 3：建立、訓練和評分模型

#### <a name="choice-of-learner"></a>選擇學習者
首先，您必須選擇學習者。 使用兩個類別推進式決策樹作為我們的學習者。 以下是這個學習者的預設選項：

![二元促進式決策樹參數](./media/hive-criteo-walkthrough/bH3ST2z.png)

對此試驗，選擇預設值。 請注意，預設值通常是有意義的好方法，可以在效能上取得快速的基準。 一旦您有了基準之後，如果您選擇整理參數，您就可以藉此改善效能。

#### <a name="train-the-model"></a>訓練模型
對於訓練，只需叫用 [訓練模型] 模組。 它的兩個輸入是兩個類別推進式決策樹和我們的訓練資料集。 其如下所示：

![訓練模組](./media/hive-criteo-walkthrough/2bZDZTy.png)

#### <a name="score-the-model"></a>評分模型
一旦您擁有定型模型，您就可以在測試資料集上評分，並評估其效能。 您可以使用下圖所示的 [評分模型]模組，加上 [評估模型] 模組來執行這項作業：

![Score Model module](./media/hive-criteo-walkthrough/fydcv6u.png)

### <a name="step4"></a> 步驟 4：評估模型
最後，您應該分析模型效能。 通常，針對兩個類別 (二進位) 分類的問題，AUC 是良好的測量方式。 為了將此情況視覺化，為此將 [評分模型] 模組連接至 [評估模型] 模組。 在 [評估模型] 模組上按一下 [視覺化] 會產生類似如下的圖形：

![評估模組 BDT 模型](./media/hive-criteo-walkthrough/0Tl0cdg.png)

在二進位檔 (或兩個類別) 分類的問題中，曲線下面積 (AUC) 是預測準確度良好的測量方式。 下列幾節顯示在測試資料集上使用此模型的結果。 若要檢查結果，請以滑鼠右鍵按一下 [評估模型] 模組的輸出連接埠，然後選取 [視覺化]。

![視覺化評估模型模組](./media/hive-criteo-walkthrough/IRfc7fH.png)

### <a name="step5"></a> 步驟 5：將模型發佈為 Web 服務
發佈 Azure Machine Learning 做為引起最少抱怨之 web 服務的功能，乃是使其可廣泛使用的寶貴功能。 完成作業後，任何人都可以利用他們需要預測的輸入資料來呼叫 web 服務，而 web 服務可使用模型傳回這些預測。

若要執行此動作，先將定型模型儲存為定型模型物件。 做法是以滑鼠右鍵按一下 [訓練模型] 模組，並使用 [儲存為訓練模型] 選項。

接下來，建立 web 服務的輸入和輸出連接埠：

* 輸入連接埠會採用的資料形式和您需要預測的資料相同
* 輸出連接埠會傳回評分標籤和相關聯的機率。

#### <a name="select-a-few-rows-of-data-for-the-input-port"></a>為輸入連接埠選取幾個資料列的資料
我們可以方便地使用 **套用 SQL 轉換** 模組，僅選取 10 個資料列做為輸入連接埠資料。 使用這裡顯示的 SQL 查詢，只為我們的輸入連接埠選取資料列：

![輸入連接埠資料](./media/hive-criteo-walkthrough/XqVtSxu.png)

#### <a name="web-service"></a>Web 服務
您現在已準備好執行可以用來發佈 Web 服務的小型試驗。

#### <a name="generate-input-data-for-webservice"></a>產生 Web 服務的輸入資料
作為預備步驟，由於計數資料表很大，所以會取得幾行測試資料，並使用計數功能從它產生輸出資料。 這會做為 web 服務的輸入資料格式。 其如下所示：

![建立 BDT 輸入資料](./media/hive-criteo-walkthrough/OEJMmst.png)

> [!NOTE]
> 針對輸入資料格式，使用 [計數 Featurizer] 模組的輸出。 一旦此實驗執行完成，將輸出從 **計數 Featurizer** 模組儲存為資料集。 此資料集會用於 Web 服務中的輸入資料。
> 
> 

#### <a name="scoring-experiment-for-publishing-webservice"></a>發佈 Web 服務的評分實驗
首先會顯示其外觀。 基本結構是 [評分模型] 模組，其會接受我們的定型模型物件以及在先前步驟中使用 [計數 Featurizer]  模組產生的一些輸入資料行。 使用「選取資料集中的資料行」來投射出評分標籤和評分機率。

![選取資料集中的資料行](./media/hive-criteo-walkthrough/kRHrIbe.png)

請注意如何使用「選取資料集中的資料行」  模組來「篩選」掉資料集中的資料。 內容如下所示：

![使用「選取資料集中的資料行」模組來進行的篩選](./media/hive-criteo-walkthrough/oVUJC9K.png)

若要取得藍色的輸入和輸出連接埠，您只要按一下位於右下方的 [ **準備 Web 服務** ]。 執行這項實驗也可讓我們發佈 Web 服務：按一下右下方的 [發佈 Web 服務]  圖示，如下所示：

![發佈 Web 服務](./media/hive-criteo-walkthrough/WO0nens.png)

發佈 Web 服務之後，您會被重新導向至看起來如下的頁面：

![Web 服務儀表板](./media/hive-criteo-walkthrough/YKzxAA5.png)

請注意左邊的兩個 Web 服務連結：

* [要求/回應] 服務 (或 RRS) 適用於單一預測而且會在這場研討會中使用。
* **批次執行** 服務 (BES) 用於批次預測，而且要求要用來進行預測的輸入資料位於 Azure Blob 儲存體。

按一下連結**要求/回應**會帶我們前往提供我們以 C#、python 和 R 預先定義程式碼的頁面。此程式碼可以方便地用於對 Web 服務進行呼叫。 請注意，需要使用此頁面上的 API 金鑰來進行驗證。

將此 python 程式碼複製到 IPython notebook 中新的儲存格很方便。

以下是一段具有正確的 API 金鑰的 python 程式碼。

![Python 程式碼](./media/hive-criteo-walkthrough/f8N4L4g.png)

請注意，預設 API 金鑰已被取代為 Web 服務的 API 金鑰。 按一下 IPython notebook 中此儲存格上的 [ **執行** ] 會產生下列回應：

![IPython 回應](./media/hive-criteo-walkthrough/KSxmia2.png)

對於要求的兩個測試範例 (在 python 指令碼的 JSON 架構中)，您會以 "Scored Labels, Scored Probabilities" 形式獲得解答。 在此情況下，已選擇預先定義的程式碼所提供的預設值 (所有數值資料行為 0，所有類別資料行為字串 "value")。

這包含我們的逐步解說，其示範如何使用 Azure Machine Learning 處理大型資料集。 您已開始使用 1 TB 的資料、建構預測模型，並將其部署為雲端中的 Web 服務。

