---
title: 使用 Spark 以 Azure Databricks 存取 Azure Data Lake Storage Gen2 預覽版資料 | Microsoft Docs
description: 了解如何在 Azure Databricks 叢集上執行 Spark 查詢，以存取 Azure Data Lake Storage Gen2 儲存體帳戶中的資料。
services: hdinsight,storage
author: dineshm
ms.component: data-lake-storage-gen2
ms.service: storage
ms.topic: tutorial
ms.date: 6/27/2018
ms.author: dineshm
ms.openlocfilehash: fd9dfaa2042cae0923c919f4e76d7b59a170918e
ms.sourcegitcommit: 06724c499837ba342c81f4d349ec0ce4f2dfd6d6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46466025"
---
# <a name="tutorial-access-azure-data-lake-storage-gen2-preview-data-with-azure-databricks-using-spark"></a>教學課程：使用 Spark 以 Azure Databricks 存取 Azure Data Lake Storage Gen2 預覽版資料

在此教學課程中，您會了解如何在 Azure Databricks 叢集上執行 Spark 查詢，以在具有 Azure Data Lake Storage Gen2 預覽版功能的帳戶中查詢資料。

> [!div class="checklist"]
> * 建立 Databricks 叢集
> * 將非結構化的資料內嵌到儲存體帳戶
> * 在 Blob 儲存體中對資料執行分析

## <a name="prerequisites"></a>先決條件

此教學課程示範如何取用及查詢航班資料 (來源為[美國運輸部](https://transtats.bts.gov/Tables.asp?DB_ID=120&DB_Name=Airline%20On-Time%20Performance%20Data&DB_Short_Name=On-Time))。 下載至少兩年份的航線資料 (選取所有欄位)，並將結果儲存至機器。 請務必記下所下載資料的檔案名稱與路徑；稍後的步驟需要此資訊。

> [!NOTE]
> 按一下 [預先壓縮的檔案] 核取方塊來選取所有資料欄位。 所下載的資料會有數 GB 的大小，但必須有這麼大量的資料才能進行分析。

## <a name="create-an-azure-data-lake-storage-gen2-account"></a>建立 Azure Data Lake Storage Gen2 帳戶

若要開始，請建立新的 [Azure Data Lake Storage Gen2 帳戶](quickstart-create-account.md)，並對它賦予唯一的名稱。 然後瀏覽至儲存體帳戶，以擷取組態設定。

1. 按一下 [設定] 下的 [存取金鑰]。
2. 按一下 [key1] 旁的 [複製] 按鈕以複製金鑰值。

此教學課程的後續步驟需要用到帳戶名稱和金鑰。 請開啟文字編輯器，並設定帳戶名稱和金鑰以供日後參考。

## <a name="create-a-databricks-cluster"></a>建立 Databricks 叢集

下一個步驟是建立 [Databricks 叢集](https://docs.azuredatabricks.net/)以建立資料工作區。

1. 建立 [Databricks 服務](https://ms.portal.azure.com/#create/Microsoft.Databricks)，並將其命名為 **myFlightDataService** (請務必在建立服務時勾選 [釘選到儀表板] 核取方塊)。
2. 按一下 [啟動工作區] 以在新的瀏覽器視窗中開啟工作區。
3. 按一下左側瀏覽列中的 [叢集]。
4. 按一下 [建立叢集]。
5. 在 [叢集名稱] 欄位中，輸入 *myFlightDataCluster*。
6. 在 [背景工作類型] 欄位中選取 [Standard_D8s_v3]。
7. 將 [背景工作數下限] 值變更為 [4]。
8. 按一下頁面頂端的 [建立叢集] (此程序最多可能需要 5 分鐘才能完成)。
9. 程序完成時，選取瀏覽列左上角的 [Azure Databricks]。
10. 在頁面下半部的 [新增] 區段下選取 [Notebook]。
11. 在 [名稱] 欄位中輸入您所選擇的名稱，然後選取 [Python] 作為語言。
12. 其他所有欄位可保留其預設值。
13. 選取 [建立] 。
14. 在 **Cmd 1** 資料格中貼入下列程式碼，並將其中的值取代為您從儲存體帳戶保留的值。

    ```bash
    spark.conf.set("fs.azure.account.key.<account_name>.dfs.core.windows.net", "<account_key>") 
    spark.conf.set("fs.azure.createRemoteFileSystemDuringInitialization", "true")
    dbutils.fs.ls("abfss://<file_system>@<account_name>.dfs.core.windows.net/")
    spark.conf.set("fs.azure.createRemoteFileSystemDuringInitialization", "false")
    ```

## <a name="ingest-data"></a>擷取資料

### <a name="copy-source-data-into-the-storage-account"></a>將來源資料複製到儲存體帳戶

下一個工作是使用 AzCopy 將資料從 .csv 檔案複製到 Azure 儲存體。 開啟命令提示字元視窗，並輸入下列命令。 請務必要將預留位置 `<DOWNLOAD_FILE_PATH>` 和 `<ACCOUNT_KEY>` 取代為您在上一個步驟中保留的對應值。

```bash
set ACCOUNT_NAME=<ACCOUNT_NAME>
set ACCOUNT_KEY=<ACCOUNT_KEY>
azcopy cp "<DOWNLOAD_FILE_PATH>" https://<ACCOUNT_NAME>.dfs.core.windows.net/dbricks/folder1/On_Time --recursive 
```

### <a name="use-databricks-notebook-to-convert-csv-to-parquet"></a>使用 Databricks Notebook 將 CSV 轉換成 Parquet

在瀏覽器中重新開啟 Databricks，然後執行下列步驟：

1. 選取瀏覽列左上角的 [Azure Databricks]。
2. 在頁面下半部的 [新增] 區段下選取 [Notebook]。
3. 在 [名稱] 欄位中輸入 **CSV2Parquet**。
4. 其他所有欄位可保留其預設值。
5. 選取 [建立] 。
6. 在 **Cmd 1** 資料格中貼入下列程式碼 (此程式碼會自動儲存到編輯器中)。

    ```python
    #mount Azure Blob Storage as an HDFS file system to your databricks cluster
    #you need to specify a storage account and container to connect to. 
    #use a SAS token or an account key to connect to Blob Storage.  
    accountname = "<insert account name>"
    accountkey = " <insert account key>"
    fullname = "fs.azure.account.key." +accountname+ ".dfs.core.windows.net"
    accountsource = "abfs://dbricks@" +accountname+ ".dfs.core.windows.net/folder1"
    #create a dataframe to read data
    flightDF = spark.read.format('csv').options(header='true', inferschema='true').load(accountsource + "/On_Time_On_Time*.csv")
    #read the all the airline csv files and write the output to parquet format for easy query
    flightDF.write.mode("append").parquet(accountsource + '/parquet/flights')

    #read the flight details parquet file 
    #flightDF = spark.read.format('parquet').options(header='true', inferschema='true').load(accountsource + "/parquet/flights")
    print("Done")
    ```

## <a name="explore-data-using-hadoop-distributed-file-system"></a>使用 Hadoop 分散式檔案系統探索資料

返回 Databricks 工作區，然後按一下左側瀏覽列中的 [最近] 圖示。

1. 按一下 [航班資料分析] Notebook。
2. 按 **Ctrl + Alt + N** 來建立新的資料格。

在 **Cmd 1** 中輸入下列每個程式碼區塊，然後按 **Cmd + Enter** 來執行 Python 指令碼。

若要取得透過 AzCopy 上傳的 CSV 檔案清單，請執行下列指令碼：

```python
import os.path
import IPython
from pyspark.sql import SQLContext
source = "abfs://<FILE_SYSTEM_NAME>@<ACCOUNT_NAME>.dfs.core.windows.net/"
dbutils.fs.ls(source + "/temp")
display(dbutils.fs.ls(source + "/temp/"))
```

若要建立新檔案，並列出 *parquet/flights* 資料夾中的檔案，請執行此指令碼：

```python
source = "abfs://<FILE_SYSTEM_NAME>@<ACCOUNT_NAME>.dfs.core.windows.net/"

dbutils.fs.help()

dbutils.fs.put(source + "/temp/1.txt", "Hello, World!", True)
dbutils.fs.ls(source + "/temp/parquet/flights")
```
藉由這些程式碼範例，您已使用具有 Azure Data Lake Storage Gen2 功能之帳戶中所儲存的資料來探索 HDFS 的階層式本質。

## <a name="query-the-data"></a>查詢資料

接下來，您可以開始查詢您上傳到 Azure Data Lake Storage 的資料。 在 **Cmd 1** 中輸入下列每個程式碼區塊，然後按 **Cmd + Enter** 來執行 Python 指令碼。

### <a name="simple-queries"></a>範例查詢

若要建立資料來源的資料框架，請執行下列指令碼：

> [!IMPORTANT]
> 請務必將 **<YOUR_CSV_FILE_NAME>** 預留位置取代為您在此教學課程剛開始時下載的檔案名稱。

```python
#Copy this into a Cmd cell in your notebook.
acDF = spark.read.format('csv').options(header='true', inferschema='true').load(accountsource + "/<YOUR_CSV_FILE_NAME>.csv")
acDF.write.parquet(accountsource + '/parquet/airlinecodes')

#read the existing parquet file for the flights database that was created earlier
flightDF = spark.read.format('parquet').options(header='true', inferschema='true').load(accountsource + "/parquet/flights")

#print the schema of the dataframes
acDF.printSchema()
flightDF.printSchema()

#print the flight database size
print("Number of flights in the database: ", flightDF.count())

#show the first 20 rows (20 is the default)
#to show the first n rows, run: df.show(n)
acDF.show(100, False)
flightDF.show(20, False)

#Display to run visualizations
#preferrably run this in a separate cmd cell
display(flightDF)
```

若要對資料執行分析查詢，請執行下列指令碼：

```python
#Run each of these queries, preferrably in a separate cmd cell for separate analysis
#create a temporary sql view for querying flight information
FlightTable = spark.read.parquet(accountsource + '/parquet/flights')
FlightTable.createOrReplaceTempView('FlightTable')

#create a temporary sql view for querying airline code information
AirlineCodes = spark.read.parquet(accountsource + '/parquet/airlinecodes')
AirlineCodes.createOrReplaceTempView('AirlineCodes')

#using spark sql, query the parquet file to return total flights in January and February 2016
out1 = spark.sql("SELECT * FROM FlightTable WHERE Month=1 and Year= 2016")
NumJan2016Flights = out1.count()
out2 = spark.sql("SELECT * FROM FlightTable WHERE Month=2 and Year= 2016")
NumFeb2016Flights=out2.count()
print("Jan 2016: ", NumJan2016Flights," Feb 2016: ",NumFeb2016Flights)
Total= NumJan2016Flights+NumFeb2016Flights
print("Total flights combined: ", Total)

# List out all the airports in Texas
out = spark.sql("SELECT distinct(OriginCityName) FROM FlightTable where OriginStateName = 'Texas'") 
print('Airports in Texas: ', out.show(100))

#find all airlines that fly from Texas
out1 = spark.sql("SELECT distinct(Carrier) FROM FlightTable WHERE OriginStateName='Texas'")
print('Airlines that fly to/from Texas: ', out1.show(100, False))
```
### <a name="complex-queries"></a>複雜的查詢

若要執行下列更複雜的查詢，請在 Notebook 中一次執行一個區段，並檢查結果。

```python
#find the airline with the most flights

#create a temporary view to hold the flight delay information aggregated by airline, then select the airline name from the Airlinecodes dataframe
spark.sql("DROP VIEW IF EXISTS v")
spark.sql("CREATE TEMPORARY VIEW v AS SELECT Carrier, count(*) as NumFlights from FlightTable group by Carrier, UniqueCarrier order by NumFlights desc LIMIT 10")
output = spark.sql("SELECT AirlineName FROM AirlineCodes WHERE AirlineCode in (select Carrier from v)")

#show the top row without truncation
output.show(1, False)

#show the top 10 airlines
output.show(10, False)

#Determine which is the least on time airline

#create a temporary view to hold the flight delay information aggregated by airline, then select the airline name from the Airlinecodes dataframe
spark.sql("DROP VIEW IF EXISTS v")
spark.sql("CREATE TEMPORARY VIEW v AS SELECT Carrier, count(*) as NumFlights from FlightTable WHERE DepDelay>60 or ArrDelay>60 group by Carrier, UniqueCarrier order by NumFlights desc LIMIT 10")
output = spark.sql("select * from v")
#output = spark.sql("SELECT AirlineName FROM AirlineCodes WHERE AirlineCode in (select Carrier from v)")
#show the top row without truncation
output.show(1, False)

#which airline improved its performance
#find the airline with the most improvement in delays
#create a temporary view to hold the flight delay information aggregated by airline, then select the airline name from the Airlinecodes dataframe
spark.sql("DROP VIEW IF EXISTS v1")
spark.sql("DROP VIEW IF EXISTS v2")
spark.sql("CREATE TEMPORARY VIEW v1 AS SELECT Carrier, count(*) as NumFlights from FlightTable WHERE (DepDelay>0 or ArrDelay>0) and Year=2016 group by Carrier order by NumFlights desc LIMIT 10")
spark.sql("CREATE TEMPORARY VIEW v2 AS SELECT Carrier, count(*) as NumFlights from FlightTable WHERE (DepDelay>0 or ArrDelay>0) and Year=2017 group by Carrier order by NumFlights desc LIMIT 10")
output = spark.sql("SELECT distinct ac.AirlineName, v1.Carrier, v1.NumFlights, v2.NumFlights from v1 INNER JOIN v2 ON v1.Carrier = v2.Carrier INNER JOIN AirlineCodes ac ON v2.Carrier = ac.AirlineCode WHERE v1.NumFlights > v2.NumFlights")
#show the top row without truncation
output.show(10, False)

#display for visual analysis
display(output)
```

## <a name="next-steps"></a>後續步驟

* [使用 Azure HDInsight 上的 Apache Hive 來擷取、轉換和載入資料](tutorial-extract-transform-load-hive.md)
