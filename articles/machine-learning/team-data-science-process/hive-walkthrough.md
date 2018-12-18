---
title: 瀏覽 Hadoop 叢集中的資料，並在 Azure Machine Learning 中建立模型 | Microsoft Docs
description: 對採用 HDInsight Hadoop 叢集來建置和部署模型的端對端案例使用 Team Data Science Process。
services: machine-learning,hdinsight
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: e9e76c91-d0f6-483d-bae7-2d3157b86aa0
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/29/2017
ms.author: deguhath
ms.openlocfilehash: ff4daf350783e02141a6afea815165ccecfe0116
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42140300"
---
# <a name="the-team-data-science-process-in-action-use-azure-hdinsight-hadoop-clusters"></a>Team Data Science Process 實務：使用 Azure HDInsight Hadoop 叢集
在此逐步解說中，我們在端對端案例中使用 [Team Data Science Process (TDSP)](overview.md)。 我們使用 [Azure HDInsight Hadoop 叢集](https://azure.microsoft.com/services/hdinsight/)，以對 [NYC 計程車車程](http://www.andresmh.com/nyctaxitrips/) \(英文\) 資料集內可公開使用的資料進行儲存、探索和特徵工程設計，並縮減取樣資料。 若要處理二元和多元分類和迴歸預測工作，我們使用 Azure Machine Learning 建置資料的模型。 

如需示範如何處理較大資料集的逐步解說，請參閱 [Team Data Science Process - 在 1 TB 的資料集上使用 Azure HDInsight Hadoop 叢集](hive-criteo-walkthrough.md)。

您也可以使用 IPython Notebook 來完成本逐步解說中使用 1 TB 資料集的工作。 如需詳細資訊，請參閱[使用 Hive ODBC 連接的 Criteo 逐步解說](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-hive-walkthrough-criteo.ipynb)。

## <a name="dataset"></a>NYC 計程車車程資料集說明
NYC 計程車車程資料是約 20 GB 的壓縮逗點分隔值 (CSV) 檔案 (未壓縮約 48 GB)。 它有超過 1.73 億趟個別的車程，而且包含每趟車程的費用。 每趟車程記錄包括上車和下車的位置與時間、匿名的計程車司機駕照號碼，以及圓形徽章號碼 (計程車的唯一識別碼)。 資料涵蓋 2013 年的所有車程，且每月提供於下列兩個資料集中：

- Trip_data CSV 檔案包含車程詳細資料。 這包括乘客數、上車和下車地點、車程持續時間以及車程長度。 以下是一些範例記錄：
   
        medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude
        89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868
- Trip_fare CSV 檔案包含每趟車程的費用詳細資料。 這包括付款類型、費用金額、附加費和稅金、小費和通行費及支付總額。 以下是一些範例記錄：
   
        medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount
        89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5

加入 trip\_data and trip\_fare 的唯一索引鍵由下列欄位組成：medallion、hack\_license 和 pickup\_datetime。 若要取得特定車程的所有詳細資訊，加入這三個索引鍵便已足夠。

## <a name="mltasks"></a>預測工作的範例
決定您想根據資料分析執行的預測種類。 這有助於釐清需要包含在程序中的工作。 以下是我們在本逐步解說中解決的三個預測問題範例。 這些是以 *tip\_amount* 為根據：

- **二元分類**：預測是否已支付某趟車程的小費。 也就是說，大於 $0 的 *tip\_amount* 是正數範例，而 $0 的 *tip\_amount* 是負數範例。
   
        Class 0: tip_amount = $0
        Class 1: tip_amount > $0
- **多元分類**：預測針對該趟車程支付的小費金額範圍。 我們將 tip\_amount 分成五個類別：
   
        Class 0: tip_amount = $0
        Class 1: tip_amount > $0 and tip_amount <= $5
        Class 2: tip_amount > $5 and tip_amount <= $10
        Class 3: tip_amount > $10 and tip_amount <= $20
        Class 4: tip_amount > $20
- **迴歸工作**：預測已針對某趟車程支付的小費金額。  

## <a name="setup"></a>設定進階分析的 HDInsight Hadoop 叢集
> [!NOTE]
> 這通常是管理工作。
> 
> 

您可以採取三個步驟，為利用 HDInsight 叢集的進階分析設定 Azure 環境：

1. [建立儲存體帳戶](../../storage/common/storage-quickstart-create-account.md)：這個儲存體帳戶是用來將資料儲存在 Azure Blob 儲存體中。 HDInsight 叢集中使用的資料也位於此處。
2. [針對進階分析程序和技術自訂 Azure HDInsight Hadoop 叢集](customize-hadoop-cluster.md)。 這個步驟會建立已在所有節點上安裝 64 位元 Anaconda Python 2.7 的 HDInsight Hadoop 叢集。 自訂 HDInsight 叢集時應注意兩個重要的步驟。
   
   * 建立步驟 1 中的儲存體帳戶時，請務必將之與您的 HDInsight 叢集連結。 這個儲存體帳戶可存取在叢集內處理的資料。
   * 建立叢集之後，請對叢集的前端節點啟用遠端存取。 瀏覽至 [組態] 索引標籤，然後選取 [啟用遠端]。 這個步驟可指定使用於遠端登入的使用者認證。
3. [建立 Azure Machine Learning 工作區](../studio/create-workspace.md)：您可以使用這個工作區來建置機器學習模型。 使用 HDInsight 叢集完成初始資料探索和縮小取樣之後，會處理這項工作。

## <a name="getdata"></a>從公用來源取得資料
> [!NOTE]
> 這通常是管理工作。
> 
> 

若要從 [NYC 計程車車程](http://www.andresmh.com/nyctaxitrips/) \(英文\) 資料集的公用位置將該資料集複製到您的電腦，請使用[將資料移進和移出 Azure Blob 儲存體](move-azure-blob.md)中所述的任何一種方法。

我們在這裡說明如何使用 AzCopy 來傳輸含有資料的檔案。 若要下載並安裝 AzCopy，請遵循[開始使用 AzCopy 命令列公用程式](../../storage/common/storage-use-azcopy.md)的指示。

1. 從命令提示字元視窗中執行下列 AzCopy 命令，以所需的目的地取代 *<path_to_data_folder>*：

        "C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:https://nyctaxitrips.blob.core.windows.net/data /Dest:<path_to_data_folder> /S

1. 複製完成時，您會在選擇的資料夾中看到總共 24 個壓縮檔。 將下載的檔案解壓縮到您本機電腦上的相同目錄。 記下未壓縮檔案所在的資料夾。 接下來會將這個資料夾稱為 *<path\_to\_unzipped_data\_files\>*。

## <a name="upload"></a>將資料上傳至 HDInsight Hadoop 叢集的預設容器
> [!NOTE]
> 這通常是管理工作。
> 
> 

在下列 AzCopy 命令中，以建立 Hadoop 叢集和解壓縮資料檔案時指定的實際值取代下列參數。

* ***<path_to_data_folder>***：您的電腦上包含未解壓縮之資料檔案的目錄 (以及路徑)。  
* ***<storage account name of Hadoop cluster>*** 與您的 HDInsight 叢集相關聯的儲存體帳戶。
* ***<default container of Hadoop cluster>*** 您的叢集所使用的預設容器。 請注意，預設容器的名稱通常與叢集本身的名稱相同。 例如，如果叢集稱為 "abc123.azurehdinsight.net"，預設容器即為 abc123。
* ***<storage account key>*** 您的叢集所使用的儲存體帳戶金鑰。

從命令提示字元或 Windows PowerShell 視窗，執行下列兩個 AzCopy 命令。

這個命令會將車程資料上傳至 Hadoop 叢集之預設容器中的 ***nyctaxitripraw*** 目錄。

        "C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:<path_to_unzipped_data_files> /Dest:https://<storage account name of Hadoop cluster>.blob.core.windows.net/<default container of Hadoop cluster>/nyctaxitripraw /DestKey:<storage account key> /S /Pattern:trip_data_*.csv

這個命令會將費用資料上傳至 Hadoop 叢集之預設容器中的 ***nyctaxifareraw*** 目錄。

        "C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:<path_to_unzipped_data_files> /Dest:https://<storage account name of Hadoop cluster>.blob.core.windows.net/<default container of Hadoop cluster>/nyctaxifareraw /DestKey:<storage account key> /S /Pattern:trip_fare_*.csv

資料現在應該位於 Blob 儲存體，而且準備好在 HDInsight 叢集內使用。

## <a name="#download-hql-files"></a>登入 Hadoop 叢集的前端節點，並準備進行探索資料分析
> [!NOTE]
> 這通常是管理工作。
> 
> 

若要存取叢集的前端節點以進行資料的探索資料分析和縮小取樣，請遵循[存取 Hadoop 叢集的前端節點](customize-hadoop-cluster.md)中所述的程序進行。

在本逐步解說中，我們主要是使用 [Hive](https://hive.apache.org/)中撰寫的查詢 (類似 SQL 的查詢語言) 來執行初步資料探索。 Hive 查詢會儲存在 .hql 檔案。 我們接著會縮小取樣這份資料，以便在 Machine Learning 中用來建置模型。

為了準備探索資料分析的叢集，我們從 [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts) 將包含相關 Hive 指令碼的 .hql 檔案下載至前端節點上的本機目錄 (C:\temp)。 若要這樣做，請從叢集的前端節點內開啟命令提示字元，並執行下列兩個命令：

    set script='https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/DataScienceProcess/DataScienceScripts/Download_DataScience_Scripts.ps1'

    @powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString(%script%))"

這兩個命令會將本逐步解說中所需的所有 .hql 檔案，下載到前端節點的本機目錄 ***C:\temp&#92;***。

## <a name="#hive-db-tables"></a>建立依月份分割的 Hive 資料庫和資料表
> [!NOTE]
> 這通常是管理工作。
> 
> 

您現在已準備好為 NYC 計程車資料集建立 Hive 資料表。
在 Hadoop 叢集的前端節點中，於前端節點的桌面上開啟 Hadoop 命令列。 執行下列命令以進入 Hive 目錄：

    cd %hive_home%\bin

> [!NOTE]
> 請從 Hive bin/ 目錄提示字元執行本逐步解說中的所有 Hive 命令。 如此可自動處理路徑相關問題。 我們將在本逐步解說中交替使用「Hive 目錄提示字元」、「Hive bin/ 目錄提示字元」、「Hadoop 命令列」等詞語。
> 
> 

從 Hive 目錄提示字元，在前端節點的 Hadoop 命令列執行下列命令。 此動作會提交 Hive 查詢以建立 Hive 資料庫和資料表：

    hive -f "C:\temp\sample_hive_create_db_and_tables.hql"

以下是 **C:\temp\sample\_hive\_create\_db\_and\_tables.hql** 檔案的內容。 這會建立 Hive 資料庫 **nyctaxidb** 和資料表 **trip** 和 **fare**。

    create database if not exists nyctaxidb;

    create external table if not exists nyctaxidb.trip
    (
        medallion string,
        hack_license string,
        vendor_id string,
        rate_code string,
        store_and_fwd_flag string,
        pickup_datetime string,
        dropoff_datetime string,
        passenger_count int,
        trip_time_in_secs double,
        trip_distance double,
        pickup_longitude double,
        pickup_latitude double,
        dropoff_longitude double,
        dropoff_latitude double)  
    PARTITIONED BY (month int)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n'
    STORED AS TEXTFILE LOCATION 'wasb:///nyctaxidbdata/trip' TBLPROPERTIES('skip.header.line.count'='1');

    create external table if not exists nyctaxidb.fare
    (
        medallion string,
        hack_license string,
        vendor_id string,
        pickup_datetime string,
        payment_type string,
        fare_amount double,
        surcharge double,
        mta_tax double,
        tip_amount double,
        tolls_amount double,
        total_amount double)
    PARTITIONED BY (month int)
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n'
    STORED AS TEXTFILE LOCATION 'wasb:///nyctaxidbdata/fare' TBLPROPERTIES('skip.header.line.count'='1');

這個 Hive 指令碼會建立兩個資料表：

* **trip** 資料表包含每趟車程的詳細資料 (駕駛詳細資料、上車時間、車程距離和時間)。
* **fare** 資料表包含費用詳細資料 (費用金額、小費金額、通行費及附加費)。

如果您需要這些程序的任何額外協助，或想要調查替代程序，請參閱[從 Hadoop 命令列直接提交 Hive 查詢](move-hive-tables.md#submit)一節。

## <a name="#load-data"></a>依資料分割將資料載入 Hive 資料表
> [!NOTE]
> 這通常是管理工作。
> 
> 

NYC 計程車資料集會依月份自然分割資料，可讓我們更快速處理和查詢時間。 下列 PowerShell 命令 (使用 Hadoop 命令列從 Hive 目錄發出) 會將資料載入至依月份分割的 trip 和 fare Hive 資料表。

    for /L %i IN (1,1,12) DO (hive -hiveconf MONTH=%i -f "C:\temp\sample_hive_load_data_by_partitions.hql")

**sample\_hive\_load\_data\_by\_partitions.hql** 檔案包含下列 **LOAD** 命令：

    LOAD DATA INPATH 'wasb:///nyctaxitripraw/trip_data_${hiveconf:MONTH}.csv' INTO TABLE nyctaxidb.trip PARTITION (month=${hiveconf:MONTH});
    LOAD DATA INPATH 'wasb:///nyctaxifareraw/trip_fare_${hiveconf:MONTH}.csv' INTO TABLE nyctaxidb.fare PARTITION (month=${hiveconf:MONTH});

請注意，在此處的探索程序中使用的 Hive 查詢只涉及查看一或兩個資料分割。 但這些查詢可跨整個資料集執行。

### <a name="#show-db"></a>顯示 HDInsight Hadoop 叢集中的資料庫
若要在 Hadoop 命令列視窗內顯示在 HDInsight Hadoop 叢集中建立的資料庫，可在 Hadoop 命令列中執行下列命令：

    hive -e "show databases;"

### <a name="#show-tables"></a>顯示 **nyctaxidb** 資料庫中的 Hive 資料表
若要顯示 **nyctaxidb** 資料庫中的資料表，可在 Hadoop 命令列中執行下列命令：

    hive -e "show tables in nyctaxidb;"

我們可執行下列命令來確認資料表已分割：

    hive -e "show partitions nyctaxidb.trip;"

以下是預期的輸出：

    month=1
    month=10
    month=11
    month=12
    month=2
    month=3
    month=4
    month=5
    month=6
    month=7
    month=8
    month=9
    Time taken: 2.075 seconds, Fetched: 12 row(s)

同樣地，我們可以執行下列命令來確保 fare 資料表已分割：

    hive -e "show partitions nyctaxidb.fare;"

以下是預期的輸出：

    month=1
    month=10
    month=11
    month=12
    month=2
    month=3
    month=4
    month=5
    month=6
    month=7
    month=8
    month=9
    Time taken: 1.887 seconds, Fetched: 12 row(s)

## <a name="#explore-hive"></a>Hive 中的資料探索和特徵工程
> [!NOTE]
> 這通常是資料科學家工作。
> 
> 

您可以針對載入到 Hive 資料表的資料使用 Hive 查詢來完成資料探索和功能工程工作。 以下是這類工作的範例：

* 檢視這兩個資料表中的前 10 筆記錄。
* 在變動的時間範圍中探索數個欄位的資料分佈。
* 調查經度和緯度欄位的資料品質。
* 根據小費金額產生二元和多元分類標籤。
* 藉由計算車程的直線距離來產生功能。

### <a name="exploration-view-the-top-10-records-in-table-trip"></a>探索：檢視 trip 資料表中的前 10 筆記錄
> [!NOTE]
> 這通常是資料科學家工作。
> 
> 

為了查看資料的外觀，請檢查每個資料表的 10 筆記錄。 若要檢查記錄，請從 Hadoop 命令列主控台的 Hive 目錄提示字元，個別執行下列兩個查詢。

若要從 trip 資料表取得第一個月的前 10 筆記錄：

    hive -e "select * from nyctaxidb.trip where month=1 limit 10;"

若要從 fare 資料表取得第一個月的前 10 筆記錄：

    hive -e "select * from nyctaxidb.fare where month=1 limit 10;"

您可以將記錄儲存到檔案，以方便檢視。 對上述查詢進行微幅變更以完成這項作業：

    hive -e "select * from nyctaxidb.fare where month=1 limit 10;" > C:\temp\testoutput

### <a name="exploration-view-the-number-of-records-in-each-of-the-12-partitions"></a>探索：檢視 12 個資料分割中每一個資料分割的記錄數目。
> [!NOTE]
> 這通常是資料科學家工作。
> 
> 

令人感興趣的是車程數目在日曆年度的變化。 依月份群組顯示車程的分佈。

    hive -e "select month, count(*) from nyctaxidb.trip group by month;"

這會提供下列輸出：

    1       14776615
    2       13990176
    3       15749228
    4       15100468
    5       15285049
    6       14385456
    7       13823840
    8       12597109
    9       14107693
    10      15004556
    11      14388451
    12      13971118
    Time taken: 283.406 seconds, Fetched: 12 row(s)

這裡的第一個資料行是月份，而第二個資料行是該月份的車程數目。

我們也可以在 Hive 目錄提示字元中執行下列命令，計算車程資料集的記錄總數：

    hive -e "select count(*) from nyctaxidb.trip;"

這會產生：

    173179759
    Time taken: 284.017 seconds, Fetched: 1 row(s)

我們可以使用針對車程資料集所顯示的類似命令，從 Hive 目錄提示字元發出費用資料集的 Hive 查詢，以驗證記錄數目。

    hive -e "select month, count(*) from nyctaxidb.fare group by month;"

這會提供下列輸出：

    1       14776615
    2       13990176
    3       15749228
    4       15100468
    5       15285049
    6       14385456
    7       13823840
    8       12597109
    9       14107693
    10      15004556
    11      14388451
    12      13971118
    Time taken: 253.955 seconds, Fetched: 12 row(s)

請注意，兩個資料集會傳回完全相同的每月車程數。 這會提供已正確載入資料的第一次驗證。

您可以在 Hive 目錄提示字元中執行下列命令，計算費用資料集的記錄總數：

    hive -e "select count(*) from nyctaxidb.fare;"

這會產生：

    173179759
    Time taken: 186.683 seconds, Fetched: 1 row(s)

兩個資料表中的記錄總數也會相同。 這會提供資料已正確載入的第二次驗證。

### <a name="exploration-trip-distribution-by-medallion"></a>探索：依據 medallion 的車程分佈
> [!NOTE]
> 這通常是資料科學家工作。
> 
> 

此範例會識別在特定期間內超過 100 趟車程的圓形徽章 (計程車數目)。 由於這個查詢受到資料分割變數 **month** 的條件約束，因此使用資料分割資料表會很有幫助。 查詢結果會寫入前端節點上 `C:\temp` 中的本機檔案 **queryoutput.tsv**。

    hive -f "C:\temp\sample_hive_trip_count_by_medallion.hql" > C:\temp\queryoutput.tsv

以下是要檢查之 **sample\_hive\_trip\_count\_by\_medallion.hql** 檔案的內容。

    SELECT medallion, COUNT(*) as med_count
    FROM nyctaxidb.fare
    WHERE month<=3
    GROUP BY medallion
    HAVING med_count > 100
    ORDER BY med_count desc;

NYC 計程車資料集中的圓形徽章會識別唯一的計程車。 您可以詢問哪些計程車在特定時段超過特定的車程數目，以識別哪些計程車相對忙碌。 下列範例會識別在前三個月中超過幾百趟車程的計程車，並將查詢結果儲存至本機檔案 **C:\temp\queryoutput.tsv**。

以下是要檢查之 **sample\_hive\_trip\_count\_by\_medallion.hql** 檔案的內容。

    SELECT medallion, COUNT(*) as med_count
    FROM nyctaxidb.fare
    WHERE month<=3
    GROUP BY medallion
    HAVING med_count > 100
    ORDER BY med_count desc;

從 Hive 目錄提示字元執行下列命令：

    hive -f "C:\temp\sample_hive_trip_count_by_medallion.hql" > C:\temp\queryoutput.tsv

### <a name="exploration-trip-distribution-by-medallion-and-hack-license"></a>探索：依據計程車牌照和計程車駕照的車程分佈
> [!NOTE]
> 這通常是資料科學家工作。
> 
> 

當探索資料集時，我們通常會想要檢查共同出現的值群組數目。 本節提供如何為計程車和駕駛執行這項作業的範例。

**sample\_hive\_trip\_count\_by\_medallion\_license.hql** 檔案會將費用資料集分組成 **medallion** 和 **hack_license**，並傳回每個組合的計數。 以下是其內容：

    SELECT medallion, hack_license, COUNT(*) as trip_count
    FROM nyctaxidb.fare
    WHERE month=1
    GROUP BY medallion, hack_license
    HAVING trip_count > 100
    ORDER BY trip_count desc;

這個查詢會依車程數目的遞減順序，傳回計程車和駕駛的組合。

從 Hive 目錄提示字元執行：

    hive -f "C:\temp\sample_hive_trip_count_by_medallion_license.hql" > C:\temp\queryoutput.tsv

查詢結果會寫入本機檔案 **C:\temp\queryoutput.tsv**。

### <a name="exploration-assessing-data-quality-by-checking-for-invalid-longitude-or-latitude-records"></a>探索：藉由檢查無效的經度或緯度記錄來評估資料品質
> [!NOTE]
> 這通常是資料科學家工作。
> 
> 

常見的探索資料分析目標是去掉無效或不正確的記錄。 本節的範例會判斷經度或緯度欄位是否包含遠離 NYC 區域的值。 由於這類記錄可能具有錯誤的經度-緯度值，因此我們想要從用來建立模型的任何資料中排除這些錯誤值。

以下是要檢查之 sample\_hive\_quality\_assessment.hql 檔案的內容。

        SELECT COUNT(*) FROM nyctaxidb.trip
        WHERE month=1
        AND  (CAST(pickup_longitude AS float) NOT BETWEEN -90 AND -30
        OR    CAST(pickup_latitude AS float) NOT BETWEEN 30 AND 90
        OR    CAST(dropoff_longitude AS float) NOT BETWEEN -90 AND -30
        OR    CAST(dropoff_latitude AS float) NOT BETWEEN 30 AND 90);


從 Hive 目錄提示字元執行：

    hive -S -f "C:\temp\sample_hive_quality_assessment.hql"

這個命令中包含的 *-S* 引數會隱藏 Hive Map/Reduce 工作的狀態畫面顯示。 這非常有用，因為它讓 Hive 查詢輸出的螢幕顯示更容易閱讀。

### <a name="exploration-binary-class-distributions-of-trip-tips"></a>探索：車程小費的二元類別分佈
> [!NOTE]
> 這通常是資料科學家工作。
> 
> 

對於 [預測工作的範例](hive-walkthrough.md#mltasks) 一節中所述的二元分類問題而言，了解是否已指定小費會很有幫助。 小費是二元分佈：

* tip given (Class 1, tip\_amount > $0)  
* no tip (Class 0, tip\_amount = $0)

以下的 **sample\_hive\_tipped\_frequencies.hql** 檔案會執行這項作業：

    SELECT tipped, COUNT(*) AS tip_freq
    FROM
    (
        SELECT if(tip_amount > 0, 1, 0) as tipped, tip_amount
        FROM nyctaxidb.fare
    )tc
    GROUP BY tipped;

從 Hive 目錄提示字元執行：

    hive -f "C:\temp\sample_hive_tipped_frequencies.hql"


### <a name="exploration-class-distributions-in-the-multiclass-setting"></a>探索：多元設定中的類別分佈
> [!NOTE]
> 這通常是資料科學家工作。
> 
> 

對於[預測工作的範例](hive-walkthrough.md#mltasks)一節中所述的多元分類問題，這個資料集也可用於自然分類，以預測指定的小費金額。 我們可以使用二元分類來定義查詢中的小費範圍。 為了取得各種小費範圍的類別分佈，請使用 **sample\_hive\_tip\_range\_frequencies.hql** 檔案。 以下是其內容。

    SELECT tip_class, COUNT(*) AS tip_freq
    FROM
    (
        SELECT if(tip_amount=0, 0,
            if(tip_amount>0 and tip_amount<=5, 1,
            if(tip_amount>5 and tip_amount<=10, 2,
            if(tip_amount>10 and tip_amount<=20, 3, 4)))) as tip_class, tip_amount
        FROM nyctaxidb.fare
    )tc
    GROUP BY tip_class;

從 Hadoop 命令列主控台執行下列命令：

    hive -f "C:\temp\sample_hive_tip_range_frequencies.hql"

### <a name="exploration-compute-the-direct-distance-between-two-longitude-latitude-locations"></a>探索：計算兩個經度-緯度位置之間的直線距離
> [!NOTE]
> 這通常是資料科學家工作。
> 
> 

您可能想知道兩個位置之間的直線距離，與計程車實際的車程距離之間是否有差異。 如果乘客認為駕駛刻意繞遠路時，他們可能比較不會給小費。

為了查看兩個經度-緯度點 (「大圓」距離) 之間的實際車程距離與 [Haversine 距離](http://en.wikipedia.org/wiki/Haversine_formula)的比較，您可以使用 Hive 中的可用三角函數：

    set R=3959;
    set pi=radians(180);

    insert overwrite directory 'wasb:///queryoutputdir'

    select pickup_longitude, pickup_latitude, dropoff_longitude, dropoff_latitude, trip_distance, trip_time_in_secs,
    ${hiveconf:R}*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
     *${hiveconf:pi}/180/2),2)-cos(pickup_latitude*${hiveconf:pi}/180)
     *cos(dropoff_latitude*${hiveconf:pi}/180)*pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2)))
     /sqrt(pow(sin((dropoff_latitude-pickup_latitude)*${hiveconf:pi}/180/2),2)
     +cos(pickup_latitude*${hiveconf:pi}/180)*cos(dropoff_latitude*${hiveconf:pi}/180)*
     pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2))) as direct_distance
    from nyctaxidb.trip
    where month=1
    and pickup_longitude between -90 and -30
    and pickup_latitude between 30 and 90
    and dropoff_longitude between -90 and -30
    and dropoff_latitude between 30 and 90;

在上述查詢中，R 是地球的半徑 (英哩)，而 pi 會轉換成弧度。 請注意，經度-緯度點會經過篩選，以移除遠離 NYC 區域的值。

在本例中，我們將結果寫入 **queryoutputdir** 目錄。 以下一連串命令會先建立這個輸出目錄，然後再執行 Hive 命令。

從 Hive 目錄提示字元執行：

    hdfs dfs -mkdir wasb:///queryoutputdir

    hive -f "C:\temp\sample_hive_trip_direct_distance.hql"


查詢結果會寫入 Hadoop 叢集的預設容器下的 9 個 Azure Blob (**queryoutputdir/000000\_0** 至 **queryoutputdir/000008\_0**)。

為了查看個別 Blob 的大小，我們從 Hive 目錄提示字元執行下列命令：

    hdfs dfs -ls wasb:///queryoutputdir

為了查看指定檔案的內容 (假設是 **000000\_0**)，請使用 Hadoop 的 `copyToLocal` 命令。

    hdfs dfs -copyToLocal wasb:///queryoutputdir/000000_0 C:\temp\tempfile

> [!WARNING]
> 遇到大型檔案，`copyToLocal` 可能會很慢，因此不建議使用。  
> 
> 

將此資料放在 Azure Blob 中的主要優點在於，我們可以使用[匯入資料][import-data]模組在 Machine Learning 內探索資料。

## <a name="#downsample"></a>在 Machine Learning 中縮小取樣和建置模型
> [!NOTE]
> 這通常是資料科學家工作。
> 
> 

在探索資料分析階段之後，我們現在已準備好縮小取樣資料，以便在 Machine Learning 中建置模型。 在本節中，我們會示範如何使用 Hive 查詢縮小取樣資料。 Machine Learning 再從[匯入資料][import-data]模組存取它。

### <a name="down-sampling-the-data"></a>縮小取樣資料
這個程序包含兩個步驟。 首先我們在所有記錄都會出現的三個索引鍵 (**medallion**、**hack\_license** 和 **pickup\_datetime**) 上加入 **nyctaxidb.trip** 和 **nyctaxidb.fare** 資料表。 接著產生二元分類標籤 **tipped** 和多元分類標籤 **tip\_class**。

為了能夠直接從 Machine Learning 中的[匯入資料][import-data]模組使用縮小取樣的資料，應將上述查詢的結果儲存至內部 Hive 資料表。 接下來我們將建立內部 Hive 資料表，並以加入和縮小取樣的資料來填入其內容。

查詢會直接套用標準 Hive 函式，以從 **pickup\_datetime** 欄位產生下列資訊：
- 時
- 年中的週
- 工作日 (1 代表星期一，7 代表星期日)

查詢也會產生上車和下車位置之間的直線距離。 如需這類函式的完整清單，請參閱 [LanguageManual UDF](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF)。

查詢會接著縮小取樣資料，以將查詢結果填入 Azure Machine Learning Studio。 只有約 1% 的原始資料集會匯入 Studio。

以下是 **sample\_hive\_prepare\_for\_aml\_full.hql** 檔案的內容，該檔案會準備在 Machine Learning 中建置模型所要使用的資料：

        set R = 3959;
        set pi=radians(180);

        create table if not exists nyctaxidb.nyctaxi_downsampled_dataset (

        medallion string,
        hack_license string,
        vendor_id string,
        rate_code string,
        store_and_fwd_flag string,
        pickup_datetime string,
        dropoff_datetime string,
        pickup_hour string,
        pickup_week string,
        weekday string,
        passenger_count int,
        trip_time_in_secs double,
        trip_distance double,
        pickup_longitude double,
        pickup_latitude double,
        dropoff_longitude double,
        dropoff_latitude double,
        direct_distance double,
        payment_type string,
        fare_amount double,
        surcharge double,
        mta_tax double,
        tip_amount double,
        tolls_amount double,
        total_amount double,
        tipped string,
        tip_class string
        )
        row format delimited fields terminated by ','
        lines terminated by '\n'
        stored as textfile;

        --- now insert contents of the join into the above internal table

        insert overwrite table nyctaxidb.nyctaxi_downsampled_dataset
        select
        t.medallion,
        t.hack_license,
        t.vendor_id,
        t.rate_code,
        t.store_and_fwd_flag,
        t.pickup_datetime,
        t.dropoff_datetime,
        hour(t.pickup_datetime) as pickup_hour,
        weekofyear(t.pickup_datetime) as pickup_week,
        from_unixtime(unix_timestamp(t.pickup_datetime, 'yyyy-MM-dd HH:mm:ss'),'u') as weekday,
        t.passenger_count,
        t.trip_time_in_secs,
        t.trip_distance,
        t.pickup_longitude,
        t.pickup_latitude,
        t.dropoff_longitude,
        t.dropoff_latitude,
        t.direct_distance,
        f.payment_type,
        f.fare_amount,
        f.surcharge,
        f.mta_tax,
        f.tip_amount,
        f.tolls_amount,
        f.total_amount,
        if(tip_amount>0,1,0) as tipped,
        if(tip_amount=0,0,
        if(tip_amount>0 and tip_amount<=5,1,
        if(tip_amount>5 and tip_amount<=10,2,
        if(tip_amount>10 and tip_amount<=20,3,4)))) as tip_class

        from
        (
        select
        medallion,
        hack_license,
        vendor_id,
        rate_code,
        store_and_fwd_flag,
        pickup_datetime,
        dropoff_datetime,
        passenger_count,
        trip_time_in_secs,
        trip_distance,
        pickup_longitude,
        pickup_latitude,
        dropoff_longitude,
        dropoff_latitude,
        ${hiveconf:R}*2*2*atan((1-sqrt(1-pow(sin((dropoff_latitude-pickup_latitude)
        *${hiveconf:pi}/180/2),2)-cos(pickup_latitude*${hiveconf:pi}/180)
        *cos(dropoff_latitude*${hiveconf:pi}/180)*pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2)))
        /sqrt(pow(sin((dropoff_latitude-pickup_latitude)*${hiveconf:pi}/180/2),2)
        +cos(pickup_latitude*${hiveconf:pi}/180)*cos(dropoff_latitude*${hiveconf:pi}/180)*pow(sin((dropoff_longitude-pickup_longitude)*${hiveconf:pi}/180/2),2))) as direct_distance,
        rand() as sample_key

        from nyctaxidb.trip
        where pickup_latitude between 30 and 90
            and pickup_longitude between -90 and -30
            and dropoff_latitude between 30 and 90
            and dropoff_longitude between -90 and -30
        )t
        join
        (
        select
        medallion,
        hack_license,
        vendor_id,
        pickup_datetime,
        payment_type,
        fare_amount,
        surcharge,
        mta_tax,
        tip_amount,
        tolls_amount,
        total_amount
        from nyctaxidb.fare
        )f
        on t.medallion=f.medallion and t.hack_license=f.hack_license and t.pickup_datetime=f.pickup_datetime
        where t.sample_key<=0.01

若要從 Hive 目錄提示字元執行這個查詢：

    hive -f "C:\temp\sample_hive_prepare_for_aml_full.hql"

我們現在有內部資料表 **nyctaxidb.nyctaxi_downsampled_dataset**，使用 Machine Learning 中的[匯入資料][import-data]模組即可存取此資料表。 此外，我們可使用這個資料集來建置 Machine Learning 模型。  

### <a name="use-the-import-data-module-in-machine-learning-to-access-the-down-sampled-data"></a>使用 Machine Learning 中的「匯入資料」模組來存取縮小取樣的資料
若要在 Machine Learning 的[匯入資料][ import-data]模組發出 Hive 查詢，您需要存取 Machine Learning 工作區。 您也需要存取叢集和其相關聯儲存體帳戶的認證。

以下是[匯入資料][import-data]模組及輸入參數的一些詳細資料：

**HCatalog 伺服器 URI**：如果叢集名稱是 **abc123**，則為：https://abc123.azurehdinsight.net。

**Hadoop 使用者帳戶名稱**：為叢集選擇的使用者名稱 (非遠端存取使用者名稱)。

**Hadoop 使用者帳戶密碼**：為叢集選擇的密碼 (非遠端存取密碼)。

**輸出資料的位置**：選擇為 Azure。

**Azure 儲存體帳戶名稱**：與叢集建立關聯的預設儲存體帳戶名稱。

**Azure 容器名稱**：這是叢集的預設容器名稱，且通常與叢集名稱相同。 如果叢集為 **abc123**，即為 abc123。

> [!IMPORTANT]
> 任何我們想要使用 Machine Learning 中的[匯入資料][import-data]模組來查詢的資料表，都必須是內部資料表。
> 
> 

以下說明如何判斷資料庫 **D.db** 中的資料表 **T** 是否為內部資料表。 從 Hive 目錄提示字元執行下列命令：

    hdfs dfs -ls wasb:///D.db/T

如果資料表是內部資料表並已填妥，其內容必須顯示在這裡。

判斷資料表是否為內部資料表的另一種方式是使用 Azure 儲存體總管。 您可以使用它巡覽至叢集的預設容器名稱，然後依資料表名稱篩選。 如果有顯示資料表及其內容，則可確認它是內部資料表。

以下是 Hive 查詢和[匯入資料][import-data]模組的螢幕擷取畫面：

![匯入資料模組的 Hive 查詢螢幕擷取畫面](./media/hive-walkthrough/1eTYf52.png)

因為我們縮小取樣的資料位於預設容器中，從 Machine Learning 產生的 Hive 查詢就非常簡單。 它就只是 **SELECT * FROM nyctaxidb.nyctaxi\_downsampled\_data**。

資料集現在可做為建置 Machine Learning 模型的起點。

### <a name="mlmodel"></a>在 Machine Learning 中建置模型
您現在可在 [Machine Learning](https://studio.azureml.net) 中繼續建置和部署模型。 資料已就緒，可用來解決稍早指出的預測問題：

- **二元分類**：預測是否已支付某趟車程的小費。

  **已使用學習者：** 二元羅吉斯迴歸

  a. 對於這個問題，目標 (或類別) 標籤是 **tipped**。 縮小取樣的原始資料集有幾個資料行會顯示這個分類實驗的目標流失。 特別是 **tip\_class**、**tip\_amount** 和 **total\_amount**，可揭示測試時不會提供的目標標籤相關資訊。 我們使用[選取資料集中的資料行][select-columns]模組，從考量範圍中移除這些資料行。

  下圖顯示我們的實驗，目的是預測是否會支付指定車程的小費：

  ![實驗圖表](./media/hive-walkthrough/QGxRz5A.png)

  b. 對於這項實驗，我們的目標標籤分佈大約是 1:1。

   下圖顯示二元分類問題之 tip 類別標籤的分佈：

  ![tip 類別標籤的分佈圖](./media/hive-walkthrough/9mM4jlD.png)

    因此，我們在 0.987 的曲線 (AUC) 底下得到一個區域，如下圖所示：

  ![AUC 值圖](./media/hive-walkthrough/8JDT0F8.png)

- **多元分類**：使用先前定義的類別，預測針對該趟車程支付的小費金額範圍。

  **已使用學習者：** 多元羅吉斯迴歸

  a. 對於這個問題，我們的目標 (或類別) 標籤是 **tip\_class**，可能採用五個值 (0、1、2、3、4) 的其中一個。 如二元分類案例所示，我們有幾個資料行會顯示這個實驗的目標。 特別是 **tipped**、**tip\_amount** 和 **total\_amount**，可揭示測試時無法取得的目標標籤相關資訊。 我們使用[選取資料集中的資料行][select-columns]模組來移除這些資料行。

  下圖顯示預測小費可能落在哪個容器的實驗。 容器為：Class 0: tip = $0、Class 1: tip > $0 and tip <= $5、Class 2: tip > $5 and tip <= $10、Class 3: tip > $10 and tip <= $20 及 Class 4: tip > $20。

  ![實驗圖表](./media/hive-walkthrough/5ztv0n0.png)

  現在會顯示實際的測試類別分佈。 類別 0 和類別 1 很普遍，其他類別則很罕見。

  ![測試類別分佈圖](./media/hive-walkthrough/Vy1FUKa.png)

  b. 對於這項實驗，我們使用混淆矩陣來查看預測精確度。 其如下所示：

  ![混淆矩陣](./media/hive-walkthrough/cxFmErM.png)

  請注意，雖然常見類別上的類別精確度很高，但模型對較罕見類別沒有很好的「學習」效果。

- **迴歸工作**：預測針對某趟車程支付的小費金額。

  **已使用學習者：** 推進式決策樹

  a. 對於這個問題，目標 (或類別) 標籤是 **tip\_amount**。 在本例中，目標流失為：**tipped****tip\_class** 及 **total\_amount**。 這所有變數都會揭示測試時通常無法取得的小費金額相關資訊。 我們使用[選取資料集中的資料行][select-columns]模組來移除這些資料行。

  下圖顯示預測指定小費金額的實驗：

  ![實驗圖表](./media/hive-walkthrough/11TZWgV.png)

  b. 對於迴歸問題，我們會藉由查看預測中的平方誤差和決定係數，來測量預測的精確度：

  ![預測統計資料螢幕擷取畫面](./media/hive-walkthrough/Jat9mrz.png)

  此處的決定係數是 0.709，其中隱含的變異大約有 71% 是由模型係數所造成。

> [!IMPORTANT]
> 若要深入了解 Machine Learning，以及如何存取與使用，請參閱[什麼是 Machine Learning](../studio/what-is-machine-learning.md)。 此外，[Azure AI 資源庫](https://gallery.cortanaintelligence.com/) \(英文\) 涵蓋了所有實驗，並且完整介紹 Machine Learning 的功能範圍。
> 
> 

## <a name="license-information"></a>授權資訊
此範例逐步解說及其隨附的指令碼是在 MIT 授權下由 Microsoft 所共用。 如需詳細資料，請查看 GitHub 上範例程式碼目錄中的 **LICENSE.txt** 檔案。

## <a name="references"></a>參考
•    [Andrés Monroy NYC 計程車車程下載頁面](http://www.andresmh.com/nyctaxitrips/) \(英文\)  
•    [FOILing NYC 的計程車車程資料 (作者為 Chris Whong)](http://chriswhong.com/open-data/foil_nyc_taxi/)   
•    [NYC 計程車和禮車委託研究和統計資料](http://www.nyc.gov/html/tlc/html/technology/aggregated_data.shtml)

[2]: ./media/hive-walkthrough/output-hive-results-3.png
[11]: ./media/hive-walkthrough/hive-reader-properties.png
[12]: ./media/hive-walkthrough/binary-classification-training.png
[13]: ./media/hive-walkthrough/create-scoring-experiment.png
[14]: ./media/hive-walkthrough/binary-classification-scoring.png
[15]: ./media/hive-walkthrough/amlreader.png

<!-- Module References -->
[select-columns]: https://msdn.microsoft.com/library/azure/1ec722fa-b623-4e26-a44e-a50c6d726223/
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/



