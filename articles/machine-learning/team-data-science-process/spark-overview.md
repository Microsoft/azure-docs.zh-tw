---
title: 在 Azure HDInsight 上使用 Spark 的資料科學概觀 | Microsoft Docs
description: Spark MLlib 工具組將可觀的機器學習模型化功能引進分散式 HDInsight 環境中。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: a4e1de99-a554-4240-9647-2c6d669593c8
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2017
ms.author: deguhath
ms.openlocfilehash: f15596bbd8a9b8bab96eeae738b63df934620648
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34838488"
---
# <a name="overview-of-data-science-using-spark-on-azure-hdinsight"></a>在 Azure HDInsight 上使用 Spark 的資料科學概觀
[!INCLUDE [machine-learning-spark-modeling](../../../includes/machine-learning-spark-modeling.md)]

這組主題說明如何使用 HDInsight Spark 來完成常見的資料科學工作，例如資料擷取、特徵工程、模型化和模型評估。 所使用的資料是 2013 NYC 計程車車程和費用資料集的抽樣樣本。 建立的模型包括羅吉斯和線性迴歸、隨機樹系和漸層停駐推進式決策樹。 這些主題也會說明如何將這些模型儲存至 Azure Blob 儲存體 (WASB)，以及如何評分及評估模型的預測效能。 更進階的主題會討論如何使用交叉驗證和超參數掃掠來訓練模型。 此概觀主題也參考說明了如何設定所需的 Spark 叢集，以完成逐步解說中的步驟的主題。 

## <a name="spark-and-mllib"></a>Spark 及 MLlib
[Spark](http://spark.apache.org/) 是一個開放原始碼平行處理架構，可支援記憶體內部處理，大幅提升巨量資料分析應用程式的效能。 Spark 處理引擎是專為速度、易用性及精密分析打造的產品。 Spark 的記憶體內分散式計算功能，使其成為機器學習和圖表計算中所使用反覆演算法的絕佳選擇。 [MLlib](http://spark.apache.org/mllib/) 是將演算法模型化功能引進此分散式環境的 Spark 可調整機器學習程式庫。 

## <a name="hdinsight-spark"></a>HDInsight Spark

  [HDInsight Spark](../../hdinsight/spark/apache-spark-overview.md) 是開放原始碼 Spark 的 Azure 託管供應項目。 它也支援 Spark 叢集上的 **Jupyter PySpark notebooks**，可執行 Spark SQL 互動式查詢以轉換、篩選和視覺化 Azure Blob (WASB) 中儲存的資料。 PySpark 是適用於 Spark 的 Python API。 程式碼片段提供了解決方案，並且顯示相關的繪圖，進而將安裝在 Spark 叢集上的 Jupyter Notebook 資料加以視覺化。 這些主題中的模型化步驟包括程式碼，以示範如何訓練、評估、儲存和使用各類模型。 

## <a name="setup-spark-clusters-and-jupyter-notebooks"></a>設定：Spark 叢集和 Jupyter Notebook
此逐步解說所提供的設定步驟和程式碼適用於使用 HDInsight Spark 1.6。 不過 Jupyter Notebook 可供 HDInsight Spark 1.6 版和 Spark 2.0 叢集兩者使用。 Notebook 的描述及它們的連結已在包含它們的 GitHub 儲存機制的 [Readme.md](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Readme.md) 中提供。 此外，此處及連結的 Notebook 內的程式碼皆屬泛型程式碼，而且應該能在任何 Spark 叢集上運作。 若您不是使用 HDInsight Spark，叢集設定和管理步驟可能與這裡顯示的稍有不同。 為了方便起見，以下是可讓 Spark 1.6 版 (在 Jupyter Notebook 伺服器的 pySpark 核心中執行) 和 Spark 2.0 版 (在 Jupyter Notebook 伺服器的 pySpark3 核心中執行) 的 Jupyter Notebook 連結：

### <a name="spark-16-notebooks"></a>Spark 1.6 Notebook
這些 Notebook 是在 Jupyter Notebook 伺服器的 pySpark 核心中執行。

- [pySpark-machine-learning-data-science-spark-data-exploration-modeling.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/pySpark-machine-learning-data-science-spark-data-exploration-modeling.ipynb)：提供如何利用數個不同的演算法來執行資料瀏覽、模型化和評分的相關資訊。
- [pySpark-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/pySpark-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb)：包含Notebook #1 中的主題，以及使用超參數微調和交叉驗證的模型開發。
- [pySpark-machine-learning-data-science-spark-model-consumption.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/pySpark-machine-learning-data-science-spark-model-consumption.ipynb)：示範如何在 HDInsight 叢集上使用 Python 將儲存的模型實際運作。

### <a name="spark-20-notebooks"></a>Spark 2.0 Notebook
這些 Notebook 是在 Jupyter Notebook 伺服器的 pySpark3 核心中執行。

- [Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0/Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb)：此檔案使用在[這裡](https://docs.microsoft.com/azure/machine-learning/machine-learning-data-science-spark-overview#the-nyc-2013-taxi-data)描述的 NYC 計程車車程和費用資料集，提供如何在 Spark 2.0 叢集中執行資料瀏覽、模型化和評分的相關資訊。 Notebook 可能是很好的起點，可快速瀏覽我們針對 Spark 2.0 所提供的程式碼。 如需更多分析 NYC 計程車資料的 Notebook 詳細資訊，請參閱這份清單中的下一個 Notebook。 請參閱此清單之後比較這些 Notebook 的附註。 
- [Spark2.0 pySpark3_NYC_Taxi_Tip_Regression.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0/Spark2.0_pySpark3_NYC_Taxi_Tip_Regression.ipynb)：這個檔案會顯示如何使用[這裡](https://docs.microsoft.com/azure/machine-learning/machine-learning-data-science-spark-overview#the-nyc-2013-taxi-data)所述的 NYC 計程車車程和費用資料集，執行資料爭議 (Spark SQL 和資料框架作業)、瀏覽、模型化和評分。
- [Spark2.0 pySpark3_Airline_Departure_Delay_Classification.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0/Spark2.0_pySpark3_Airline_Departure_Delay_Classification.ipynb)：這個檔案會顯示如何使用已知的 2011 年和 2012 年航班準時出發資料集，執行資料爭議 (Spark SQL 和資料框架作業)、瀏覽、模型化和評分。 我們在模型化之前將航班資料集與機場天氣資料 (例如風速、溫度、高度等等) 整合，因此可以在模型中包含這些天氣功能。

<!-- -->

> [!NOTE]
> Spark 2.0 Notebook 中新增了航班資料集，以更清楚地說明使用的分類演算法。 請參閱下列連結，以取得航班準時出發資料集和天氣資料集的相關資訊：

>- 航班準時出發資料：[http://www.transtats.bts.gov/ONTIME/](http://www.transtats.bts.gov/ONTIME/)

>- 機場天氣資料：[https://www.ncdc.noaa.gov/](https://www.ncdc.noaa.gov/) 
> 
> 

<!-- -->

<!-- -->

> [!NOTE]
NYC 計程車和飛行航班延遲資料集上的 Spark 2.0 Notebook 需要 10 分鐘或更久 (取決於 HDI 叢集的大小) 才能執行。 上述清單中的第一個 Notebook 說明 Notebook 中許多層面的資料瀏覽、視覺效果和 ML 模型訓練，會使用向下取樣 NYC 資料集以較短時間執行，其中計程車和車資檔案已預先聯結︰[Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark2.0/Spark2.0-pySpark3-machine-learning-data-science-spark-advanced-data-exploration-modeling.ipynb)此 Notebook 會採用較短的時間來完成 (2-3 分鐘)，並可能是快速瀏覽我們針對 Spark 2.0 所提供之程式碼的一個良好起點。 

<!-- -->

如需 Spark 2.0 模型和評分的模型耗用量實際運作的相關指引，請參閱[有關耗用量的 Spark 1.6 文件](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/pySpark/Spark1.6/pySpark-machine-learning-data-science-spark-model-consumption.ipynb)，以取得概述所需步驟的範例。 若要在 Spark 2.0 上使用此檔案，將使用[這個檔案](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/Spark/Python/Spark2.0_ConsumeRFCV_NYCReg.py)來取代 Python 程式碼。

### <a name="prerequisites"></a>先決條件
下列程序與 Spark 1.6 相關。 對於 Spark 2.0 版本，請使用先前說明和連結的 Notebook。 

1. 您必須擁有 Azure 訂用帳戶。 如果還沒有訂用帳戶，請參閱 [取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。

2. 您需要 Spark 1.6 叢集才能完成這個逐步解說。 若要建立該項目，請參閱 [開始使用：在 Azure HDInsight 上建立 Apache Spark](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md)所提供的指示。 叢集類型和版本是由 [選取叢集類型]  功能表來指定。 

![設定叢集](./media/spark-overview/spark-cluster-on-portal.png)

<!-- -->

> [!NOTE]
> 如需示範如何使用 Scala 而非 Python 完成端對端資料科學程序工作的主題，請參閱 [在 Azure 上使用 Spark 與 Spark 的資料科學](scala-walkthrough.md)。
> 
> 

<!-- -->

> [!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]
> 
> 

## <a name="the-nyc-2013-taxi-data"></a>NYC 2013 計程車資料
「NYC 計程車車程」資料大約是 20GB 以逗號分隔值 (CSV) 的壓縮檔 (未壓縮時可達 48GB)，其中包含超過 1 億 7300 萬筆個別車程及針對每趟車程支付的費用。 每趟車程記錄包括上車和下車的位置與時間、匿名的計程車司機駕照號碼，以及圓形徽章 (計程車的唯一識別碼) 號碼。 資料涵蓋 2013 年的所有車程，並且每月會在下列兩個資料集中加以提供：

1. 'trip_data' CSV 檔案包含車程詳細資訊，例如乘客數、上車和下車地點、車程持續時間，以及車程長度。 以下是一些範例記錄：
   
        medallion,hack_license,vendor_id,rate_code,store_and_fwd_flag,pickup_datetime,dropoff_datetime,passenger_count,trip_time_in_secs,trip_distance,pickup_longitude,pickup_latitude,dropoff_longitude,dropoff_latitude
        89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,1,N,2013-01-01 15:11:48,2013-01-01 15:18:10,4,382,1.00,-73.978165,40.757977,-73.989838,40.751171
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-06 00:18:35,2013-01-06 00:22:54,1,259,1.50,-74.006683,40.731781,-73.994499,40.75066
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,1,N,2013-01-05 18:49:41,2013-01-05 18:54:23,1,282,1.10,-74.004707,40.73777,-74.009834,40.726002
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:54:15,2013-01-07 23:58:20,2,244,.70,-73.974602,40.759945,-73.984734,40.759388
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,1,N,2013-01-07 23:25:03,2013-01-07 23:34:24,1,560,2.10,-73.97625,40.748528,-74.002586,40.747868
2. 'trip_data' CSV 檔案包含針對每趟車程所支付的費用詳細資料，例如付款類型、費用金額、附加費和稅金、小費和通行費，以及支付的總金額。 以下是一些範例記錄：
   
        medallion, hack_license, vendor_id, pickup_datetime, payment_type, fare_amount, surcharge, mta_tax, tip_amount, tolls_amount, total_amount
        89D227B655E5C82AECF13C3F540D4CF4,BA96DE419E711691B9445D6A6307C170,CMT,2013-01-01 15:11:48,CSH,6.5,0,0.5,0,0,7
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-06 00:18:35,CSH,6,0.5,0.5,0,0,7
        0BD7C8F5BA12B88E0B67BED28BEA73D8,9FD8F69F0804BDB5549F40E9DA1BE472,CMT,2013-01-05 18:49:41,CSH,5.5,1,0.5,0,0,7
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:54:15,CSH,5,0.5,0.5,0,0,6
        DFD2202EE08F7A8DC9A57B02ACB81FE2,51EE87E3205C985EF8431D850C786310,CMT,2013-01-07 23:25:03,CSH,9.5,0.5,0.5,0,0,10.5

我們已取得這些檔案的 0.1% 樣本，並將 trip\_data 和 trip\_fare CSV 檔聯結成單一資料集，做為此逐步解說的輸入資料集。 聯結 trip\_data 和 trip\_fare 的唯一索引鍵是由下列欄位組成：medallion、hack\_licence、pickup\_datetime。 資料集的每一筆記錄會包含代表 NYC 計程車車程的下列屬性︰

| 欄位 | 簡短描述 |
| --- | --- |
| medallion |匿名的計程車圓形徽章 (唯一的計程車識別碼) |
| hack_license |匿名的 Hackney 歸位字元駕照號碼 |
| vendor_id |計程車廠商識別碼 |
| rate_code |NYC 計程車費率 |
| store_and_fwd_flag |儲存和轉寄旗標 |
| pickup_datetime |上車日期和時間 |
| dropoff_datetime |下車日期和時間 |
| pickup_hour |於幾點上車 |
| pickup_week |於一年中的第幾星期上車 |
| weekday |工作日 (範圍 1-7) |
| passenger_count |計程車車程的乘客數目 |
| trip_time_in_secs |往返時間 (秒) |
| trip_distance |以英哩計的車程距離 |
| pickup_longitude |上車處經度 |
| pickup_latitude |上車處緯度 |
| dropoff_longitude |下車經度 |
| dropoff_latitude |下車緯度 |
| direct_distance |上車與下車位置之間的直線距離 |
| payment_type |付款類型 (cas、信用卡等) |
| fare_amount |費用金額 |
| surcharge |額外費用 |
| mta_tax |Mta 稅額 |
| tip_amount |小費金額 |
| tolls_amount |收費金額 |
| total_amount |總金額 |
| tipped |已收到小費 (用 0 或 1 表示否或是) |
| tip_class |小費類別 (0：$0、1：$0-5、2：$6-10、3：$11-20、4：> $20) |

## <a name="execute-code-from-a-jupyter-notebook-on-the-spark-cluster"></a>從 Spark 叢集的 Jupyter Notebook 中執行程式碼
您可以從 Azure 入口網站啟動 Jupyter Notebook。 在儀表板上尋找 Spark 叢集，並按一下該項目以進入您的叢集管理頁面。 若要開啟與 Spark 叢集相關聯的 Notebook，按一下 [叢集儀表板]  ->  [Jupyter Notebook]。

![叢集儀表板](./media/spark-overview/spark-jupyter-on-portal.png)

若要存取 Jupyter Notebook，您也可以瀏覽 ***https://CLUSTERNAME.azurehdinsight.net/jupyter***。 以您的叢集名稱來取代此 URL 內的 CLUSTERNAME 部分。 您需要有系統管理員帳戶的密碼才能存取 Notebook。

![瀏覽 Jupyter Notebooks](./media/spark-overview/spark-jupyter-notebook.png)

選取 [PySpark] 來查看包含一些預先封裝 Notebook 範例的目錄，這些範例使用了 PySpark API。適用於這組 Spark 主題的 Notebook (包含程式碼範例) 可在 [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/Spark/pySpark) 上找到。

您可以將 Notebook 直接從 [GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/Spark/pySpark) 上傳至 Spark 叢集上的 Jupyter Notebook 伺服器。 在您的 Jupyter 首頁，按一下螢幕右側的 [上傳]  按鈕。 [檔案總管] 隨即開啟。 您可以在這裡貼上 Notebook 的 GitHub (原始內容) URL，然後按一下 [開啟]。 

您會在 Jupyter 檔案清單上看到檔案名稱，並重新出現 [上傳]  按鈕。 按一下此 [上傳]  按鈕。 現在您已匯入 Notebook。 重複這些步驟，使用此逐步解說上傳其他 Notebook。

> [!TIP]
> 您可以用滑鼠右鍵按一下瀏覽器上的下列連結，然後選取 [複製連結] 取得 Github 原始內容 URL。 您可以將此 URL 貼到 Jupyter 的 [上傳] 檔案總管對話方塊。
> 
> 

現在您可以：

* 按一下 Notebook，查看程式碼。
* 按 **SHIFT-ENTER** 執行每個儲存格。
* 按一下 [儲存格]  ->  [執行] 執行整個筆記本。
* 使用查詢的自動視覺效果。

> [!TIP]
> PySpark 核心會將 SQL (HiveQL) 查詢的輸出自動視覺化。 在 Notebook 內使用 [類型]  功能表按鈕，系統提供您幾種不同類型的視覺效果 (資料表、圓形圖、折線圖、區域圖或橫條圖) 可選擇。
> 
> 

![泛型方法的羅吉斯迴歸 ROC 曲線](./media/spark-overview/pyspark-jupyter-autovisualization.png)

## <a name="whats-next"></a>後續步驟
現在您已使用 HDInsight Spark 叢集進行設定，並已上傳 Jupyter 筆記本，您已準備要逐步執行對應至這三個 PySpark Notebook 的主題。 這些主題示範如何瀏覽資料、建立和取用模型。 進階的資料探索和模型化 Notebook 顯示如何包括交叉驗證、超參數清除和模型評估。 

**使用 Spark 資料探索和模型化：**：遵循[使用 Spark MLlib 工具組來建立資料的二進位分類和迴歸模型](spark-data-exploration-modeling.md)主題的內容，來探索資料集，以及建立、評分、評估 Machine Learning 模型。

**模型耗用量︰** 若要瞭解如何評分本主題中所建立的分類和迴歸模型，請參閱 [評分及評估 Spark 建置機器學習服務模型](spark-model-consumption.md)。

**交叉驗證和超參數清除**：請參閱 [使用 Spark 進階資料探索和模型化](spark-advanced-data-exploration-modeling.md) 有關如何使用交叉驗證和超參數清除訓練模型

