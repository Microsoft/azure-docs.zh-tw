---
title: Azure Data Lake 中可調整的資料科學︰完整的逐步解說 | Microsoft Docs
description: 如何使用 Azure Data Lake 在資料集上進行資料探索和二元分類工作。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: 91a8207f-1e57-4570-b7fc-7c5fa858ffeb
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2017
ms.author: deguhath
ms.openlocfilehash: 62ca89ffe7507c2dc0a0f1a86750fb2bb996a5bd
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34836965"
---
# <a name="scalable-data-science-with-azure-data-lake-an-end-to-end-walkthrough"></a>Azure Data Lake 中可調整的資料科學︰完整的逐步解說
本逐步解說示範如何使用 Azure Data Lake，在 NYC 計程車車程和車費資料集上執行資料探索和二元分類，以預測車費是否會支付小費。 其中，從取得資料開始，經過模型訓練，然後部署 Web 服務來發佈模型，從頭到尾逐步引導您完成 [Team Data Science Process](http://aka.ms/datascienceprocess)。

### <a name="azure-data-lake-analytics"></a>Azure Data Lake Analytics
[Microsoft Azure Data Lake](https://azure.microsoft.com/solutions/data-lake/) 具備所有必要的功能，讓資料科學家能夠輕易地儲存任何大小、形狀和速度的資料，並以高延展性且符合成本效益的方式，進行資料處理、進階分析和建構機器學習服務模型。   只在實際處理資料時，才需要依個別作業付費。 Azure Data Lake Analytics 包括 U-SQL，此語言融合 SQL 的宣告性質和 C# 表達能力，提供可調整的分散式查詢功能。 它可讓您在讀取、插入自訂邏輯和使用者定義函式 (UDF) 上套用結構描述，以處理非結構化資料，並包含擴充性，可精細控制如何大規模執行。 若要深入了解 U-SQL 背後的設計原理，請參閱 [Visual Studio 部落格文章](https://blogs.msdn.microsoft.com/visualstudio/2015/09/28/introducing-u-sql-a-language-that-makes-big-data-processing-easy/)。

Data Lake Analytics 也是 Cortana Analytics 套件的重要組成部分，可搭配 Azure SQL 資料倉儲、Power BI 與 Data Factory 一起使用。 這讓您有一個完整的雲端巨量資料和進階分析平台。

本逐步解說首先描述如何安裝完成資料科學程序工作所需的必要條件和資源。 接著，將概述使用 U-SQL 的資料處理步驟，最後將示範如何搭配 Azure Machine Learning Studio 使用 Python 和 Hive 來建置和部署預測模型。 

### <a name="u-sql-and-visual-studio"></a>U-SQL 和 Visual Studio
本逐步解說建議使用 Visual Studio 編輯 U-SQL 指令碼來處理資料集。 這些 U-SQL 指令碼說明於此，也在個別檔案中提供 。 過程包括擷取、探索和取樣資料。 同時還會示範如何從 Azure 入口網站執行 U-SQL 指令碼作業。 相關聯的 HDInsight 叢集中會為資料建立 Hive 資料表，以利於 Azure Machine Learning Studio 中建置和部署二元分類模型。  

### <a name="python"></a>Python
本逐步解說也包含一個小節，示範如何搭配 Azure Machine Learning Studio 使用 Python 來建置和部署預測模型。 它針對此程序中的步驟提供 Jupyter Notebook 和 Python 指令碼。 除了此處所述的二元分類模型，此 Notebook 還包含一些額外特徵工程步驟和模型建構的程式碼，例如多類別分類和迴歸模型。 迴歸工作是根據其他小費特徵來預測小費金額。 

### <a name="azure-machine-learning"></a>Azure Machine Learning
Azure Machine Learning Studio 可用來建置和部署預測模型。 這是使用下列兩種方法來完成︰首先會使用 Python 指令碼，接著使用 HDInsight (Hadoop) 叢集上的 Hive 資料表。

### <a name="scripts"></a>指令碼
本逐步解說只概述主要步驟。 您可以從 [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/AzureDataLakeWalkthrough) 下載完整的 **U-SQL 指令碼**和 **Jupyter Notebook**。

## <a name="prerequisites"></a>先決條件
開始運用這些主題之前，您必須符合下列條件：

* Azure 訂用帳戶。 如果還沒有訂用帳戶，請參閱 [取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* [建議] Visual Studio 2013 或更新版本。 如果您還沒有安裝這些版本的其中一個，您可以從 [Visual Studio 社群](https://www.visualstudio.com/vs/community/)下載免費的社群版本。

> [!NOTE]
> 除了 Visual Studio，您也可以使用 Azure 入口網站提交 Azure Data Lake 查詢。 **使用 U-SQL 處理資料**一節會提供指示，說明如何使用 Visual Studio 以及在入口網站上執行此動作。 
> 
> 


## <a name="prepare-data-science-environment-for-azure-data-lake"></a>準備 Azure Data Lake 的資料科學環境
若要準備本逐步解說的資料科學環境，請建立下列資源︰

* Azure Data Lake Store (ADLS) 
* Azure Data Lake Analytics (ADLA)
* Azure Blob 儲存體帳戶
* Azure Machine Learning Studio 帳戶
* Azure Data Lake Tools for Visual Studio (建議)

本節提供如何建立這些資源的指示。 如果您選擇搭配 Azure Machine Learning 使用 Hive 資料表 (而不是 Python) 來建置模型，您也必須佈建 HDInsight (Hadoop) 叢集。 這個替代程序將於＜選項 2＞一節中加以說明。


> [!NOTE]
> **Azure Data Lake Store** 可以個別建立，或者當您建立 **Azure Data Lake Analytics** 做為預設儲存體時加以建立。 參考的指示會個別建立這其中的每一個資源，但不需要個別建立 Data Lake 儲存體帳戶。
>
> 

### <a name="create-an-azure-data-lake-store"></a>建立 Azure Data Lake Store


從 [Azure 入口網站](http://portal.azure.com)建立 ADLS。 如需詳細資訊，請參閱 [使用 Azure 入口網站建立 HDInsight 叢集與 Data Lake Store](../../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md)。 請務必依該文件所述，在 [選擇性組態] 刀鋒視窗的 [資料來源] 刀鋒視窗中設定 [叢集 AAD 身分識別]。 

 ![3](./media/data-lake-walkthrough/3-create-ADLS.PNG)

### <a name="create-an-azure-data-lake-analytics-account"></a>建立 Azure Data Lake Analytics 帳戶
從 [Azure 入口網站](http://portal.azure.com)建立 ADLA 帳戶。 如需詳細資訊，請參閱 [教學課程：使用 Azure 入口網站開始使用 Azure Data Lake Analytics](../../data-lake-analytics/data-lake-analytics-get-started-portal.md)。 

 ![4](./media/data-lake-walkthrough/4-create-ADLA-new.PNG)

### <a name="create-an-azure-blob-storage-account"></a>建立 Azure Blob 儲存體帳戶
從 [Azure 入口網站](http://portal.azure.com)建立 Azure Blob 儲存體帳戶。 如需詳細資訊，請參閱 [關於 Azure 儲存體帳戶](../../storage/common/storage-create-storage-account.md)中的＜建立儲存體帳戶＞一節。

 ![5](./media/data-lake-walkthrough/5-Create-Azure-Blob.PNG)

### <a name="set-up-an-azure-machine-learning-studio-account"></a>設定 Azure Machine Learning Studio 帳戶
從 [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) 頁面註冊/登入 Azure Machine Learning Studio。 按一下 [立即開始使用] 按鈕，然後選擇 [免費工作區] 或 [標準工作區]。 您現在能夠在 Azure ML Studio 中建立實驗。  

### <a name="install-azure-data-lake-tools-recommended"></a>安裝 Azure Data Lake 工具 [建議]
依照您的 Visual Studio 版本，從 [Azure Data Lake Tools for Visual Studio (Visual Studio 適用的 Azure Data Lake 工具)](https://www.microsoft.com/download/details.aspx?id=49504)中安裝適合的 Azure Data Lake 工具。

 ![6](./media/data-lake-walkthrough/6-install-ADL-tools-VS.PNG)

在成功完成安裝之後，開啟 Visual Studio。 您在頂端的功能表應該會看到 Data Lake 索引標籤。 當您登入 Azure 帳戶時，您的 Azure 資源應該會出現在左面板中。

 ![7](./media/data-lake-walkthrough/7-install-ADL-tools-VS-done.PNG)

## <a name="the-nyc-taxi-trips-dataset"></a>NYC 計程車車程資料集
此處使用的資源集是公開可用的資料集 -- [NYC Taxi Trips (NYC 計程車車程)](http://www.andresmh.com/nyctaxitrips/) 資料集。 「NYC 計程車車程」資料是由約 20GB 的 CSV 壓縮檔 (未壓縮時可達 48GB) 所組成，裡面記錄了超過 1 億 7300 萬筆個別車程及針對每趟車程所支付的費用。 每趟車程記錄包括上車和下車的位置與時間、匿名的計程車司機駕照號碼，以及計程車牌照 (計程車的唯一識別碼) 號碼。 資料涵蓋 2013 年的所有車程，並且每月會在下列兩個資料集中加以提供：

「trip_data」CSV 檔案包含車程的詳細資訊，例如，乘客數、上車和下車地點、車程持續時間，以及車程長度。 以下是一些範例記錄：

       medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude
       89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171
       0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066
       0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002
       DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388
       DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868



「trip_fare」CSV 檔案包含針對每趟車程所支付之費用的詳細資訊，例如付款類型、費用金額、銷售稅和稅金、小費和服務費，以及支付的總金額。 以下是一些範例記錄：

       medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount
       89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7
       0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7
       0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7
       DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6
       DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5

聯結 trip\_data 和 trip\_fare 的唯一索引鍵是由下列三個欄位所組成：medallion、hack\_license 和 pickup\_datetime。 從公用 Azure 儲存體 blob 中可以存取原始 CSV 檔案。 適用於此聯結的 U-SQL 指令碼位於 [聯結車程和費用資料表](#join) 一節中。

## <a name="process-data-with-u-sql"></a>使用 U-SQL 處理資料
本節所述的資料處理工作包括擷取、檢查品質、探索和取樣資料。 同時說明如何聯結車程和費用資料表。 最後一節說明從 Azure 入口網站執行 U-SQL 指令碼作業。 以下是各小節的連結︰

* [資料擷取：從公用 blob 讀取資料](#ingest)
* [資料品質檢查](#quality)
* [資料探索](#explore)
* [如何聯結車程和費用資料表](#join)
* [資料取樣](#sample)
* [執行 U-SQL 作業](#run)

這些 U-SQL 指令碼說明於此，也在個別檔案中提供 。 您可以從 **GitHub** 下載完整的 [U-SQL 指令碼](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/AzureDataLakeWalkthrough)。

若要執行 U-SQL，請開啟 Visual Studio、按一下 [檔案] --> [新增] --> [專案]、選擇 [U-SQL 專案]，命名並儲存到資料夾。

![8](./media/data-lake-walkthrough/8-create-USQL-project.PNG)

> [!NOTE]
> 您可以使用 Azure 入口網站來執行 U-SQL，而不是 Visual Studio。 您可以瀏覽到入口網站上的 Azure Data Lake Analytics 資源並直接提交查詢 (如下圖所示)：
> 
> 

![9](./media/data-lake-walkthrough/9-portal-submit-job.PNG)

### <a name="ingest"></a>資料擷取：從公用 Blob 讀取資料
Azure Blob 中的資料位置是以 **wasb://container_name@blob_storage_account_name.blob.core.windows.net/blob_name** 來參考，可透過 **Extractors.Csv()** 來擷取。 在下列指令碼中，以您自己的容器名稱和儲存體帳戶名稱來替換 wasb 位址中的 container_name@blob_storage_account_name。 由於檔案名稱的格式相同，所以有可能使用 **trip\_data_{\*\}.csv** 來讀取全部 12 個車程檔案。 

    ///Read in Trip data
    @trip0 =
        EXTRACT 
        medallion string,
        hack_license string,
        vendor_id string,
        rate_code string,
        store_and_fwd_flag string,
        pickup_datetime string,
        dropoff_datetime string,
        passenger_count string,
        trip_time_in_secs string,
        trip_distance string,
        pickup_longitude string,
        pickup_latitude string,
        dropoff_longitude string,
        dropoff_latitude string
    // This is reading 12 trip data from blob
    FROM "wasb://container_name@blob_storage_account_name.blob.core.windows.net/nyctaxitrip/trip_data_{*}.csv"
    USING Extractors.Csv();

由於第一列有標頭，您必須移除標頭，並將資料行類型變更為適當的類型。 您可以使用 **swebhdfs://data_lake_storage_name.azuredatalakestorage.net/folder_name/file_name**_，將已處理的資料儲存至 Azure Data Lake 儲存體，或使用 **wasb://container_name@blob_storage_account_name.blob.core.windows.net/blob_name**，儲存至 Azure Blob 儲存體帳戶。 

    // change data types
    @trip =
        SELECT 
        medallion,
        hack_license,
        vendor_id,
        rate_code,
        store_and_fwd_flag,
        DateTime.Parse(pickup_datetime) AS pickup_datetime,
        DateTime.Parse(dropoff_datetime) AS dropoff_datetime,
        Int32.Parse(passenger_count) AS passenger_count,
        Double.Parse(trip_time_in_secs) AS trip_time_in_secs,
        Double.Parse(trip_distance) AS trip_distance,
        (pickup_longitude==string.Empty ? 0: float.Parse(pickup_longitude)) AS pickup_longitude,
        (pickup_latitude==string.Empty ? 0: float.Parse(pickup_latitude)) AS pickup_latitude,
        (dropoff_longitude==string.Empty ? 0: float.Parse(dropoff_longitude)) AS dropoff_longitude,
        (dropoff_latitude==string.Empty ? 0: float.Parse(dropoff_latitude)) AS dropoff_latitude
    FROM @trip0
    WHERE medallion != "medallion";

    ////output data to ADL
    OUTPUT @trip   
    TO "swebhdfs://data_lake_storage_name.azuredatalakestore.net/nyctaxi_folder/demo_trip.csv"
    USING Outputters.Csv(); 

    ////Output data to blob
    OUTPUT @trip   
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_trip.csv"
    USING Outputters.Csv();  

同樣地，您可以讀取費用資料集。 以滑鼠右鍵按一下 [Azure Data Lake Store]，您可以選擇在 [Azure 入口網站] --> [資料總管] 或 Visual Studio 內的 [檔案總管] 中查看您的資料。 

 ![10](./media/data-lake-walkthrough/10-data-in-ADL-VS.PNG)

 ![11](./media/data-lake-walkthrough/11-data-in-ADL.PNG)

### <a name="quality"></a>資料品質檢查
讀取車程和費用資料表之後，即可依下列方式檢查資料品質。 產生的 CSV 檔案可以輸出至 Azure Blob 儲存體或 Azure Data Lake Store。 

尋找獎章數目和卓越的獎章數目：

    ///check the number of medallions and unique number of medallions
    @trip2 =
        SELECT
        medallion,
        vendor_id,
        pickup_datetime.Month AS pickup_month
        FROM @trip;

    @ex_1 =
        SELECT
        pickup_month, 
        COUNT(medallion) AS cnt_medallion,
        COUNT(DISTINCT(medallion)) AS unique_medallion
        FROM @trip2
        GROUP BY pickup_month;
        OUTPUT @ex_1   
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_1.csv"
    USING Outputters.Csv(); 

尋找有超過 100 個車程的牌照：

    ///find those medallions that had more than 100 trips
    @ex_2 =
        SELECT medallion,
               COUNT(medallion) AS cnt_medallion
        FROM @trip2
        //where pickup_datetime >= "2013-01-01t00:00:00.0000000" and pickup_datetime <= "2013-04-01t00:00:00.0000000"
        GROUP BY medallion
        HAVING COUNT(medallion) > 100;
        OUTPUT @ex_2   
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_2.csv"
    USING Outputters.Csv(); 

找到 pickup_longitude 方面的無效記錄︰

    ///find those invalid records in terms of pickup_longitude
    @ex_3 =
        SELECT COUNT(medallion) AS cnt_invalid_pickup_longitude
        FROM @trip
        WHERE
        pickup_longitude <- 90 OR pickup_longitude > 90;
        OUTPUT @ex_3   
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_3.csv"
    USING Outputters.Csv(); 

尋找一些變數的遺漏值︰

    //check missing values
    @res =
        SELECT *,
               (medallion == null? 1 : 0) AS missing_medallion
        FROM @trip;

    @trip_summary6 =
        SELECT 
            vendor_id,
        SUM(missing_medallion) AS medallion_empty, 
        COUNT(medallion) AS medallion_total,
        COUNT(DISTINCT(medallion)) AS medallion_total_unique  
        FROM @res
        GROUP BY vendor_id;
    OUTPUT @trip_summary6
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_16.csv"
    USING Outputters.Csv();



### <a name="explore"></a>資料探索
使用下列指令碼稍微探索資料，可以更充分了解資料。

尋找有小費和無小費車程的分佈︰

    ///tipped vs. not tipped distribution
    @tip_or_not =
        SELECT *,
               (tip_amount > 0 ? 1: 0) AS tipped
        FROM @fare;

    @ex_4 =
        SELECT tipped,
               COUNT(*) AS tip_freq
        FROM @tip_or_not
        GROUP BY tipped;
        OUTPUT @ex_4   
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_4.csv"
    USING Outputters.Csv(); 

尋找有截止值的小費金額分佈︰0、5、10 和 20 美金。

    //tip class/range distribution
    @tip_class =
        SELECT *,
               (tip_amount >20? 4: (tip_amount >10? 3:(tip_amount >5 ? 2:(tip_amount > 0 ? 1: 0)))) AS tip_class
        FROM @fare;
    @ex_5 =
        SELECT tip_class,
               COUNT(*) AS tip_freq
        FROM @tip_class
        GROUP BY tip_class;
        OUTPUT @ex_5   
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_5.csv"
    USING Outputters.Csv(); 

尋找車程距離的基本統計資料︰

    // find basic statistics for trip_distance
    @trip_summary4 =
        SELECT 
            vendor_id,
            COUNT(*) AS cnt_row,
            MIN(trip_distance) AS min_trip_distance,
            MAX(trip_distance) AS max_trip_distance,
            AVG(trip_distance) AS avg_trip_distance 
        FROM @trip
        GROUP BY vendor_id;
    OUTPUT @trip_summary4
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_14.csv"
    USING Outputters.Csv();

尋找車程距離的百分位數︰

    // find percentiles of trip_distance
    @trip_summary3 =
        SELECT DISTINCT vendor_id AS vendor,
                        PERCENTILE_DISC(0.25) WITHIN GROUP(ORDER BY trip_distance) OVER(PARTITION BY vendor_id) AS median_trip_distance_disc,
                        PERCENTILE_DISC(0.5) WITHIN GROUP(ORDER BY trip_distance) OVER(PARTITION BY vendor_id) AS median_trip_distance_disc,
                        PERCENTILE_DISC(0.75) WITHIN GROUP(ORDER BY trip_distance) OVER(PARTITION BY vendor_id) AS median_trip_distance_disc
        FROM @trip;
       // group by vendor_id;
    OUTPUT @trip_summary3
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_13.csv"
    USING Outputters.Csv(); 


### <a name="join"></a>聯結車程和費用資料表
車程和費用資料表可以由 medallion、hack_license 和 pickup_time 聯結。

    //join trip and fare table

    @model_data_full =
    SELECT t.*, 
    f.payment_type, f.fare_amount, f.surcharge, f.mta_tax, f.tolls_amount,  f.total_amount, f.tip_amount,
    (f.tip_amount > 0 ? 1: 0) AS tipped,
    (f.tip_amount >20? 4: (f.tip_amount >10? 3:(f.tip_amount >5 ? 2:(f.tip_amount > 0 ? 1: 0)))) AS tip_class
    FROM @trip AS t JOIN  @fare AS f
    ON   (t.medallion == f.medallion AND t.hack_license == f.hack_license AND t.pickup_datetime == f.pickup_datetime)
    WHERE   (pickup_longitude != 0 AND dropoff_longitude != 0 );

    //// output to blob
    OUTPUT @model_data_full   
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_7_full_data.csv"
    USING Outputters.Csv(); 

    ////output data to ADL
    OUTPUT @model_data_full   
    TO "swebhdfs://data_lake_storage_name.azuredatalakestore.net/nyctaxi_folder/demo_ex_7_full_data.csv"
    USING Outputters.Csv(); 


對於每個層級的乘客計數，計算記錄數、平均小費金額、小費金額變異數、小費車程的百分比。

    // contigency table
    @trip_summary8 =
        SELECT passenger_count,
               COUNT(*) AS cnt,
               AVG(tip_amount) AS avg_tip_amount,
               VAR(tip_amount) AS var_tip_amount,
               SUM(tipped) AS cnt_tipped,
               (float)SUM(tipped)/COUNT(*) AS pct_tipped
        FROM @model_data_full
        GROUP BY passenger_count;
        OUTPUT @trip_summary8
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_17.csv"
    USING Outputters.Csv();


### <a name="sample"></a>資料取樣
首先，我們從聯結的資料表隨機選取 0.1% 的資料︰

    //random select 1/1000 data for modeling purpose
    @addrownumberres_randomsample =
    SELECT *,
            ROW_NUMBER() OVER() AS rownum
    FROM @model_data_full;

    @model_data_random_sample_1_1000 =
    SELECT *
    FROM @addrownumberres_randomsample
    WHERE rownum % 1000 == 0;

    OUTPUT @model_data_random_sample_1_1000   
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_7_random_1_1000.csv"
    USING Outputters.Csv(); 

接著，以二元變數 tip_class 進行分層取樣：

    //stratified random select 1/1000 data for modeling purpose
    @addrownumberres_stratifiedsample =
    SELECT *,
            ROW_NUMBER() OVER(PARTITION BY tip_class) AS rownum
    FROM @model_data_full;

    @model_data_stratified_sample_1_1000 =
    SELECT *
    FROM @addrownumberres_stratifiedsample
    WHERE rownum % 1000 == 0;
    //// output to blob
    OUTPUT @model_data_stratified_sample_1_1000   
    TO "wasb://container_name@blob_storage_account_name.blob.core.windows.net/demo_ex_9_stratified_1_1000.csv"
    USING Outputters.Csv(); 
    ////output data to ADL
    OUTPUT @model_data_stratified_sample_1_1000   
    TO "swebhdfs://data_lake_storage_name.azuredatalakestore.net/nyctaxi_folder/demo_ex_9_stratified_1_1000.csv"
    USING Outputters.Csv(); 


### <a name="run"></a>執行 U-SQL 作業
完成編輯 U-SQL 指令碼時，您可以使用 Azure Data Lake Analytics 帳戶將它們提交到伺服器。 依序按一下 [Data Lake] 和 [提交作業]，並選取您的 [Analytics 帳戶]，然後選擇 [平行處理原則]，再按一下 [提交] 按鈕。  

 ![12](./media/data-lake-walkthrough/12-submit-USQL.PNG)

成功編譯作業後，您作業的狀態會顯示在 Visual Studio 中供您監視。 作業執行完成後，您甚至可以重播作業執行程序，找出瓶頸步驟來提升作業效率。 您也可以移至 Azure 入口網站來檢查 U-SQL 作業的狀態。

 ![13](./media/data-lake-walkthrough/13-USQL-running-v2.PNG)

 ![14](./media/data-lake-walkthrough/14-USQL-jobs-portal.PNG)

現在，您可以在 Azure Blob 儲存體或 Azure 入口網站中檢查輸出檔案。 在下一個步驟中，使用分層取樣資料進行模型化。

 ![15](./media/data-lake-walkthrough/15-U-SQL-output-csv.PNG)

 ![16](./media/data-lake-walkthrough/16-U-SQL-output-csv-portal.PNG)

## <a name="build-and-deploy-models-in-azure-machine-learning"></a>在 Azure Machine Learning 中建置和部署模型
您有兩個可用的選項，可將資料提取到要建置的 Azure Machine Learning 

* 在第一個選項中，您可以使用已寫入 Azure Blob 的取樣資料 (來自上述的 **資料取樣** 步驟)，然後使用 Python，從 Azure Machine Learning 建置並部署模型。 
* 在第二個選項中，您會直接使用 Hive 查詢來查詢 Azure Data Lake 中的資料。 此選項需要您建立新的 HDInsight 叢集，或使用現有的 HDInsight 叢集，其中 Hive 資料表指向 Azure Data Lake Storage 中的 NY 計程車資料。  下列各節中會討論這兩個選項。 

## <a name="option-1-use-python-to-build-and-deploy-machine-learning-models"></a>選項 1︰使用 Python 建置和部署機器學習服務模型
若要使用 Python 建置和部署機器學習服務模型，請在您的本機電腦上或在 Azure Machine Learning Studio 中建立 Jupyter Notebook。 [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/AzureDataLakeWalkthrough) 上提供的 Jupyter Notebook 包含完整的程式碼，可探索、視覺化資料、特徵工程、模型化和部署。 本文只涵蓋模型化和部署。 

### <a name="import-python-libraries"></a>匯入 Python 程式庫
若要執行範例 Jupyter Notebook 或 Python 指令碼檔案，您需要下列 Python 封裝。 如果您使用 AzureML Notebook 服務，則已預先安裝這些封裝。

    import pandas as pd
    from pandas import Series, DataFrame
    import numpy as np
    import matplotlib.pyplot as plt
    from time import time
    import pyodbc
    import os
    from azure.storage.blob import BlobService
    import tables
    import time
    import zipfile
    import random
    import sklearn
    from sklearn.linear_model import LogisticRegression
    from sklearn.cross_validation import train_test_split
    from sklearn import metrics
    from __future__ import division
    from sklearn import linear_model
    from azureml import services


### <a name="read-in-the-data-from-blob"></a>從 blob 讀取資料
* 連接字串   
  
        CONTAINERNAME = 'test1'
        STORAGEACCOUNTNAME = 'XXXXXXXXX'
        STORAGEACCOUNTKEY = 'YYYYYYYYYYYYYYYYYYYYYYYYYYYY'
        BLOBNAME = 'demo_ex_9_stratified_1_1000_copy.csv'
        blob_service = BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
* 以文字形式讀取
  
        t1 = time.time()
        data = blob_service.get_blob_to_text(CONTAINERNAME,BLOBNAME).split("\n")
        t2 = time.time()
        print(("It takes %s seconds to read in "+BLOBNAME) % (t2 - t1))
  
  ![17](./media/data-lake-walkthrough/17-python_readin_csv.PNG)    
* 加入資料行名稱和分隔資料行
  
        colnames = ['medallion','hack_license','vendor_id','rate_code','store_and_fwd_flag','pickup_datetime','dropoff_datetime',
        'passenger_count','trip_time_in_secs','trip_distance','pickup_longitude','pickup_latitude','dropoff_longitude','dropoff_latitude',
        'payment_type', 'fare_amount', 'surcharge', 'mta_tax', 'tolls_amount',  'total_amount', 'tip_amount', 'tipped', 'tip_class', 'rownum']
        df1 = pd.DataFrame([sub.split(",") for sub in data], columns = colnames)
* 將某些資料行變更為數值
  
        cols_2_float = ['trip_time_in_secs','pickup_longitude','pickup_latitude','dropoff_longitude','dropoff_latitude',
        'fare_amount', 'surcharge','mta_tax','tolls_amount','total_amount','tip_amount', 'passenger_count','trip_distance'
        ,'tipped','tip_class','rownum']
        for col in cols_2_float:
            df1[col] = df1[col].astype(float)

### <a name="build-machine-learning-models"></a>建置機器學習服務模型
在此，您可建置二元分類模型，預測一趟車程是否收到小費。 在 Jupyter Notebook 中，您可以發現其他兩個模型︰多類別分類和迴歸模型。

* 首先，您必須建立可在 scikit-learn 模型中使用的虛擬變數
  
        df1_payment_type_dummy = pd.get_dummies(df1['payment_type'], prefix='payment_type_dummy')
        df1_vendor_id_dummy = pd.get_dummies(df1['vendor_id'], prefix='vendor_id_dummy')
* 建立模型化的資料框架
  
        cols_to_keep = ['tipped', 'trip_distance', 'passenger_count']
        data = df1[cols_to_keep].join([df1_payment_type_dummy,df1_vendor_id_dummy])
  
        X = data.iloc[:,1:]
        Y = data.tipped
* 訓練和測試 60-40 分割
  
        X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=0.4, random_state=0)
* 訓練集中的羅吉斯迴歸
  
        model = LogisticRegression()
        logit_fit = model.fit(X_train, Y_train)
        print ('Coefficients: \n', logit_fit.coef_)
        Y_train_pred = logit_fit.predict(X_train)
  
       ![c1](./media/data-lake-walkthrough/c1-py-logit-coefficient.PNG)
* 評分測試資料集
  
        Y_test_pred = logit_fit.predict(X_test)
* 計算評估度量
  
        fpr_train, tpr_train, thresholds_train = metrics.roc_curve(Y_train, Y_train_pred)
        print fpr_train, tpr_train, thresholds_train
  
        fpr_test, tpr_test, thresholds_test = metrics.roc_curve(Y_test, Y_test_pred) 
        print fpr_test, tpr_test, thresholds_test
  
        #AUC
        print metrics.auc(fpr_train,tpr_train)
        print metrics.auc(fpr_test,tpr_test)
  
        #Confusion Matrix
        print metrics.confusion_matrix(Y_train,Y_train_pred)
        print metrics.confusion_matrix(Y_test,Y_test_pred)
  
       ![c2](./media/data-lake-walkthrough/c2-py-logit-evaluation.PNG)

### <a name="build-web-service-api-and-consume-it-in-python"></a>建置 Web 服務 API 並在 Python 中使用
您希望機器學習服務模型在建置後開始運作。 我們在此以二元羅吉斯模型為例。 請確定本機電腦中的 scikit-learn 版本是 0.15.1。 如果您使用 Azure ML Studio 服務，則不必擔心這一點。

* 從 Azure ML Studio 設定中尋找您的工作區認證。 在 Azure Machine Learning Studio 中，按一下 [設定] --> [名稱] --> [授權權杖]。 
  
    ![c3](./media/data-lake-walkthrough/c3-workspace-id.PNG)

        workspaceid = 'xxxxxxxxxxxxxxxxxxxxxxxxxxx'
        auth_token = 'xxxxxxxxxxxxxxxxxxxxxxxxxxx'

* 建立 Web 服務
  
        @services.publish(workspaceid, auth_token) 
        @services.types(trip_distance = float, passenger_count = float, payment_type_dummy_CRD = float, payment_type_dummy_CSH=float, payment_type_dummy_DIS = float, payment_type_dummy_NOC = float, payment_type_dummy_UNK = float, vendor_id_dummy_CMT = float, vendor_id_dummy_VTS = float)
        @services.returns(int) #0, or 1
        def predictNYCTAXI(trip_distance, passenger_count, payment_type_dummy_CRD, payment_type_dummy_CSH,payment_type_dummy_DIS, payment_type_dummy_NOC, payment_type_dummy_UNK, vendor_id_dummy_CMT, vendor_id_dummy_VTS ):
            inputArray = [trip_distance, passenger_count, payment_type_dummy_CRD, payment_type_dummy_CSH, payment_type_dummy_DIS, payment_type_dummy_NOC, payment_type_dummy_UNK, vendor_id_dummy_CMT, vendor_id_dummy_VTS]
            return logit_fit.predict(inputArray)
* 取得 Web 服務認證
  
        url = predictNYCTAXI.service.url
        api_key =  predictNYCTAXI.service.api_key
  
        print url
        print api_key
  
        @services.service(url, api_key)
        @services.types(trip_distance = float, passenger_count = float, payment_type_dummy_CRD = float, payment_type_dummy_CSH=float,payment_type_dummy_DIS = float, payment_type_dummy_NOC = float, payment_type_dummy_UNK = float, vendor_id_dummy_CMT = float, vendor_id_dummy_VTS = float)
        @services.returns(float)
        def NYCTAXIPredictor(trip_distance, passenger_count, payment_type_dummy_CRD, payment_type_dummy_CSH,payment_type_dummy_DIS, payment_type_dummy_NOC, payment_type_dummy_UNK, vendor_id_dummy_CMT, vendor_id_dummy_VTS ):
            pass
* 呼叫 Web 服務 API。 執行上一個步驟後，您必須等候 5-10 秒。
  
        NYCTAXIPredictor(1,2,1,0,0,0,0,0,1)
  
       ![c4](./media/data-lake-walkthrough/c4-call-API.PNG)

## <a name="option-2-create-and-deploy-models-directly-in-azure-machine-learning"></a>選項 2：直接在 Azure Machine Learning 中建立和部署模型
Azure Machine Learning Studio 可以直接從 Azure Data Lake Store 讀取資料，然後用來建立和部署模型。 這個方法會使用指向 Azure Data Lake Store 的 Hive 資料表。 這必須佈建不同的 Azure HDInsight 叢集，而 Hive 資料表將建立於其中。 下列各節將示範如何執行這項操作。 

### <a name="create-an-hdinsight-linux-cluster"></a>建立 HDInsight Linux 叢集
從 [Azure 入口網站](http://portal.azure.com)建立 HDInsight 叢集 (Linux)。 如需詳細資訊，請參閱[使用 Azure 入口網站建立 HDInsight 叢集與 Data Lake Store](../../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md) 中的**建立可存取 Azure Data Lake Store 的 HDInsight 叢集**一節。

 ![18](./media/data-lake-walkthrough/18-create_HDI_cluster.PNG)

### <a name="create-hive-table-in-hdinsight"></a>在 HDInsight 中建立 Hive 資料表
現在，您可使用上一個步驟中儲存於 Azure Data Lake Store 中的資料，在 HDInsight 叢集中建立要於 Azure Machine Learning Studio 中使用的 Hive 資料表。 移至所建立的 HDInsight 叢集。 按一下 [設定] --> [屬性] --> [叢集 AAD 身分識別] --> [ADLS 存取]，確定已將您的 Azure Data Lake Store 帳戶新增清單中，且具有讀取、寫入及執行權限。 

 ![19](./media/data-lake-walkthrough/19-HDI-cluster-add-ADLS.PNG)

然後，按一下 [設定] 按鈕旁邊的 [儀表板]，隨即出現一個視窗。 按一下頁面右上角的 [Hive 檢視]，您應看見 [查詢編輯器]。

 ![20](./media/data-lake-walkthrough/20-HDI-dashboard.PNG)

 ![21](./media/data-lake-walkthrough/21-Hive-Query-Editor-v2.PNG)

貼上下列 Hive 指令碼來建立資料表。 在 Azure Data Lake Store 中以這種方式參考資料來源的位置：**adl://data_lake_store_name.azuredatalakestore.net:443/folder_name/file_name**。

    CREATE EXTERNAL TABLE nyc_stratified_sample
    (
        medallion string,
        hack_license string,
        vendor_id string,
        rate_code string,
        store_and_fwd_flag string,
        pickup_datetime string,
        dropoff_datetime string,
        passenger_count string,
        trip_time_in_secs string,
        trip_distance string,
        pickup_longitude string,
        pickup_latitude string,
        dropoff_longitude string,
        dropoff_latitude string,
      payment_type string,
      fare_amount string,
      surcharge string,
      mta_tax string,
      tolls_amount string,
      total_amount string,
      tip_amount string,
      tipped string,
      tip_class string,
      rownum string
      )
    ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' lines terminated by '\n'
    LOCATION 'adl://data_lake_storage_name.azuredatalakestore.net:443/nyctaxi_folder/demo_ex_9_stratified_1_1000_copy.csv';


當查詢完成執行時，您應看到如下所示的結果︰

 ![22](./media/data-lake-walkthrough/22-Hive-Query-results.PNG)

### <a name="build-and-deploy-models-in-azure-machine-learning-studio"></a>在 Azure Machine Learning Studio 中建置和部署模型
您現在已準備好要建置和部署使用 Azure Machine Learning 來預測是否會支付小費的模型。 分層範例資料已經備妥，可在這個二元分類 (有無小費) 問題中使用。 使用多類別分類 (tip_class) 和迴歸 (tip_amount) 的預測模型也會使用 Azure Machine Learning Studio 來建置和部署，但是這裡只會示範如何使用二元分類模型來處理此案例。

1. 使用**匯入資料**模組 (可從**資料輸入和輸出**一節取得)，將資料匯入 Azure ML。 如需詳細資訊，請參閱 [匯入資料](https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/) 模組參考頁面。
2. 在 [屬性] 面板中，選取 [Hive 查詢] 做為 [資料來源]。
3. 將下列 Hive 指令碼貼到 [Hive 資料庫查詢] 編輯器中
   
        select * from nyc_stratified_sample;
4. 輸入 HDInsight 叢集的 URI (這可以在 Azure 入口網站中找到)、Hadoop 認證、輸出資料的位置，以及 Azure 儲存體帳戶名稱/金鑰/容器名稱。
   
   ![23](./media/data-lake-walkthrough/23-reader-module-v3.PNG)  

下圖顯示從 Hive 資料表讀取資料的二元分類實驗範例：

 ![24](./media/data-lake-walkthrough/24-AML-exp.PNG)

建立實驗之後，按一下 [設定 Web 服務] --> [預測性 Web 服務]

 ![25](./media/data-lake-walkthrough/25-AML-exp-deploy.PNG)

執行自動建立的評分實驗，完成後，按一下 [部署 Web 服務]

 ![26](./media/data-lake-walkthrough/26-AML-exp-deploy-web.PNG)

很快就會顯示 Web 服務儀表板︰

 ![27](./media/data-lake-walkthrough/27-AML-web-api.PNG)

## <a name="summary"></a>總結
完成這個逐步解說之後，您就已經建立了資料科學環境，可在 Azure Data Lake 中建置可調整的端對端解決方案。 這個環境可用來分析大型公用資料集，透過資料科學程序的標準步驟來取得此公用資料集，過程中從取得資料開始、經過模型訓練，最後將模型部署成 Web 服務。 您可以使用 U-SQL 來處理、探索和取樣資料。 Python 和 Hive 會與 Azure Machine Learning Studio 搭配使用來建置和部署預測模型。

## <a name="whats-next"></a>後續步驟
[Team Data Science Process (TDSP)](http://aka.ms/datascienceprocess) 的學習路徑提供可說明進階分析程序中每個步驟的主題連結。 [Team Data Science Process 逐步解說](walkthroughs.md) 頁面上分項列出一系列逐步解說，示範如何在各種不同預測性分析案例中使用資源和服務：

* [Team Data Science Process 實務：使用 SQL 資料倉儲](sqldw-walkthrough.md)
* [Team Data Science Process 實務：使用 HDInsight Hadoop 叢集](hive-walkthrough.md)
* [Team Data Science Process：使用 SQL Server](sql-walkthrough.md)
* [在 Azure HDInsight 上使用 Spark 的資料科學程序概觀](spark-overview.md)
