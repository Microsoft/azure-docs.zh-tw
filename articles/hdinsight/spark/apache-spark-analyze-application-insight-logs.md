---
title: 使用 Spark 分析 Application Insights 記錄 - Azure HDInsight
description: 了解如何將 Application Insights 記錄檔匯出至 blob 儲存體，並接著使用 HDInsight 上的 Spark 分析記錄檔。
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/09/2018
ms.openlocfilehash: 51123fee9a3ab3c79e57af12a64e1c0b05daf79a
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041600"
---
# <a name="analyze-application-insights-telemetry-logs-with-spark-on-hdinsight"></a>使用 HDInsight 上的 Spark 分析 Application Insights 遙測記錄檔

了解如何在 HDInsight 上使用 Spark 來分析 Application Insights 遙測資料。

[Visual Studio Application Insights](../../application-insights/app-insights-overview.md) 是一項分析服務，可監視您的 Web 應用程式。 可以將 Application Insights 產生的遙測資料匯出至 Azure 儲存體。 資料一旦位於 Azure 儲存體中，HDInsight 便可用於進行分析。

## <a name="prerequisites"></a>必要條件

* 設定要使用 Application Insights 的應用程式。

* 熟悉以 Linux 為基礎的 HDInsight 叢集的建立程序。 如需詳細資訊，請參閱[在 HDInsight 中建立 Spark](apache-spark-jupyter-spark-sql.md)。

  > [!IMPORTANT]
  > 此文件中的步驟需要使用 Linux 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

* 網頁瀏覽器。

開發和測試本文件時使用了以下資源：

* Application Insights 遙測資料，由 [設定為使用 Application Insights 的 Node.js Web 應用程式](../../application-insights/app-insights-nodejs.md)產生。

* 在 Linux 上使用 HDInsight 叢集版本 3.5 上的 Spark 分析資料。

## <a name="architecture-and-planning"></a>架構與規劃

下圖說明此範例的服務架構：

![顯示資料從 Application Insights 傳輸至 blob 儲存體，然後由 HDInsight 上的 Spark 處理的圖](./media/apache-spark-analyze-application-insight-logs/appinsightshdinsight.png)

### <a name="azure-storage"></a>Azure 儲存體

Application Insights 可以設定為持續將遙測資訊匯出到 blob。 HDInsight 接著便可讀取儲存在 blob 中的資料。 不過，有一些您必須遵守的需求︰

* **位置**︰如果儲存體帳戶和 HDInsight 位於不同位置，可能就會增加延遲。 此外，將輸出費用套用到在區域間移動的資料時也會增加成本。

    > [!WARNING]
    > 不支援在與 HDInsight 不同的位置使用儲存體帳戶。

* **blob 類型**：HDInsight 僅支援區塊 blob。 Application Insights 預設為使用區塊 blob，因此應該使用預設項目來搭配 HDInsight。

如需將儲存體新增至現有 叢集的資訊，請參閱[新增其他儲存體帳戶](../hdinsight-hadoop-add-storage.md)。

### <a name="data-schema"></a>資料結構描述

Application Insights 提供 [匯出資料模型](../../application-insights/app-insights-export-data-model.md) 資訊，做為匯出至 blob 之遙測資料的格式依據。 這份文件中的步驟使用 Spark SQL 來處理資料。 Spark SQL 可以為 Application Insights 所記錄的 JSON 資料結構自動產生結構描述。

## <a name="export-telemetry-data"></a>匯出遙測資料

依照 [設定連續匯出](../../application-insights/app-insights-export-telemetry.md) 中的步驟設定您的 Application Insights 將遙測資訊匯出到 Azure 儲存體 blob。

## <a name="configure-hdinsight-to-access-the-data"></a>設定 HDInsight 來存取資料

如果您要建立 HDInsight 叢集，請在叢集建立期間新增儲存體帳戶。

若要將 Azure 儲存體帳戶新增至現有的叢集，請使用[新增其他儲存體帳戶](../hdinsight-hadoop-add-storage.md)文件中的資訊。

## <a name="analyze-the-data-pyspark"></a>分析資料︰PySpark

1. 在 [Azure 入口網站](https://portal.azure.com)中選取您 HDInsight 叢集上的 Spark。 在 [快速連結] 區段中，選取 [叢集儀表板]，然後選取 [叢集儀表板] 區段中的 [Jupyter Notebook]。

    ![叢集儀表板](./media/apache-spark-analyze-application-insight-logs/clusterdashboards.png)

2. 在 Jupyter 頁面右上角依序選取 [新增]、[PySpark]。 隨即開啟新的瀏覽器索引標籤，其中包含以 Python 為基礎的 Jupyter Notebook。

3. 在頁面的第一個欄位 (稱為**儲存格**) 中，輸入下列文字：

   ```python
   sc._jsc.hadoopConfiguration().set('mapreduce.input.fileinputformat.input.dir.recursive', 'true')
   ```

    這個程式碼會設定 Spark 以遞迴方式存取輸入資料的目錄結構。 Application Insights 遙測會記錄到類似 `/{telemetry type}/YYYY-MM-DD/{##}/` 的目錄結構中。

4. 使用 **SHIFT + ENTER** 執行程式碼。 在儲存格左邊，方括號之間出現 '\*' 即表示正在執行此儲存格中的程式碼。 當執行完成之後，'\*' 就會變成數字，而儲存格下方會顯示類似以下文字的輸出：

        Creating SparkContext as 'sc'

        ID    YARN Application ID    Kind    State    Spark UI    Driver log    Current session?
        3    application_1468969497124_0001    pyspark    idle    Link    Link    ✔

        Creating HiveContext as 'sqlContext'
        SparkContext and HiveContext created. Executing user code ...
5. 新的儲存格會建立在第一個儲存格之下。 在新的儲存格中輸入下列文字。 將 `CONTAINER` 和 `STORAGEACCOUNT` 取代為 Azure 儲存體帳戶名稱和包含 Application Insights 資料的 Blob 容器名稱。

   ```python
   %%bash
   hdfs dfs -ls wasb://CONTAINER@STORAGEACCOUNT.blob.core.windows.net/
   ```

    使用 **SHIFT + ENTER** 執行程此儲存格。 您會看到類似以下文字的結果：

        Found 1 items
        drwxrwxrwx   -          0 1970-01-01 00:00 wasb://appinsights@contosostore.blob.core.windows.net/contosoappinsights_2bededa61bc741fbdee6b556571a4831

    傳回的 wasb 路徑是 Application Insights 遙測資料的位置。 將儲存格中的 `hdfs dfs -ls` 這一行變更為使用傳回的 wasb 路徑，然後使用 **SHIFT + ENTER** 再執行一次儲存格。 此時，結果應該會顯示包含遙測資料的目錄。

   > [!NOTE]
   > 本節中步驟的其餘部分使用 `wasb://appinsights@contosostore.blob.core.windows.net/contosoappinsights_{ID}/Requests` 目錄。 您的目錄結構可能不同。

6. 在下一個儲存格中，輸入下列程式碼︰將 `WASB_PATH` 取代為前一個步驟中的路徑。

   ```python
   jsonFiles = sc.textFile('WASB_PATH')
   jsonData = sqlContext.read.json(jsonFiles)
   ```

    這個程式碼會從連續匯出程序匯出的 JSON 檔案建立資料框架。 使用 **SHIFT + ENTER** 執行此儲存格。
7. 在下一個儲存格中輸入並執行下列命令，以檢視 Spark 為 JSON 檔案建立的結構描述︰

   ```python
   jsonData.printSchema()
   ```

    每種類型的遙測結構描述皆不同。 以下範例是針對 Web 要求產生的結構描述 (資料儲存在 `Requests` 子目錄中)：

        root
        |-- context: struct (nullable = true)
        |    |-- application: struct (nullable = true)
        |    |    |-- version: string (nullable = true)
        |    |-- custom: struct (nullable = true)
        |    |    |-- dimensions: array (nullable = true)
        |    |    |    |-- element: string (containsNull = true)
        |    |    |-- metrics: array (nullable = true)
        |    |    |    |-- element: string (containsNull = true)
        |    |-- data: struct (nullable = true)
        |    |    |-- eventTime: string (nullable = true)
        |    |    |-- isSynthetic: boolean (nullable = true)
        |    |    |-- samplingRate: double (nullable = true)
        |    |    |-- syntheticSource: string (nullable = true)
        |    |-- device: struct (nullable = true)
        |    |    |-- browser: string (nullable = true)
        |    |    |-- browserVersion: string (nullable = true)
        |    |    |-- deviceModel: string (nullable = true)
        |    |    |-- deviceName: string (nullable = true)
        |    |    |-- id: string (nullable = true)
        |    |    |-- osVersion: string (nullable = true)
        |    |    |-- type: string (nullable = true)
        |    |-- location: struct (nullable = true)
        |    |    |-- city: string (nullable = true)
        |    |    |-- clientip: string (nullable = true)
        |    |    |-- continent: string (nullable = true)
        |    |    |-- country: string (nullable = true)
        |    |    |-- province: string (nullable = true)
        |    |-- operation: struct (nullable = true)
        |    |    |-- name: string (nullable = true)
        |    |-- session: struct (nullable = true)
        |    |    |-- id: string (nullable = true)
        |    |    |-- isFirst: boolean (nullable = true)
        |    |-- user: struct (nullable = true)
        |    |    |-- anonId: string (nullable = true)
        |    |    |-- isAuthenticated: boolean (nullable = true)
        |-- internal: struct (nullable = true)
        |    |-- data: struct (nullable = true)
        |    |    |-- documentVersion: string (nullable = true)
        |    |    |-- id: string (nullable = true)
        |-- request: array (nullable = true)
        |    |-- element: struct (containsNull = true)
        |    |    |-- count: long (nullable = true)
        |    |    |-- durationMetric: struct (nullable = true)
        |    |    |    |-- count: double (nullable = true)
        |    |    |    |-- max: double (nullable = true)
        |    |    |    |-- min: double (nullable = true)
        |    |    |    |-- sampledValue: double (nullable = true)
        |    |    |    |-- stdDev: double (nullable = true)
        |    |    |    |-- value: double (nullable = true)
        |    |    |-- id: string (nullable = true)
        |    |    |-- name: string (nullable = true)
        |    |    |-- responseCode: long (nullable = true)
        |    |    |-- success: boolean (nullable = true)
        |    |    |-- url: string (nullable = true)
        |    |    |-- urlData: struct (nullable = true)
        |    |    |    |-- base: string (nullable = true)
        |    |    |    |-- hashTag: string (nullable = true)
        |    |    |    |-- host: string (nullable = true)
        |    |    |    |-- protocol: string (nullable = true)
8. 使用下列命令將資料框架註冊為暫存資料表，並針對資料執行查詢︰

   ```python
   jsonData.registerTempTable("requests")
   df = sqlContext.sql("select context.location.city from requests where context.location.city is not null")
   df.show()
   ```

    此查詢會傳回 context.location.city 不是 null 的前 20 筆記錄的 city 資訊。

   > [!NOTE]
   > context 結構會出現在 Application Insights 記錄的所有遙測中。 您的記錄中可能不會填入 city 元素。 使用結構描述找出您可以查詢可能包含您的記錄檔資料的其他元素。

    此查詢會傳回類似以下文字的資訊：

        +---------+
        |     city|
        +---------+
        | Bellevue|
        |  Redmond|
        |  Seattle|
        |Charlotte|
        ...
        +---------+

## <a name="analyze-the-data-scala"></a>分析資料︰Scala

1. 在 [Azure 入口網站](https://portal.azure.com)中選取您 HDInsight 叢集上的 Spark。 在 [快速連結] 區段中，選取 [叢集儀表板]，然後選取 [叢集儀表板] 區段中的 [Jupyter Notebook]。

    ![叢集儀表板](./media/apache-spark-analyze-application-insight-logs/clusterdashboards.png)
2. 在 Jupyter 頁面右上角依序選取 [新增]、[Scala]。 新的瀏覽器索引標籤隨即出現，其中包含以 Scala 為基礎的 Jupyter Notebook。
3. 在頁面的第一個欄位 (稱為**儲存格**) 中，輸入下列文字：

   ```scala
   sc.hadoopConfiguration.set("mapreduce.input.fileinputformat.input.dir.recursive", "true")
   ```

    這個程式碼會設定 Spark 以遞迴方式存取輸入資料的目錄結構。 Application Insights 遙測會記錄到類似 `/{telemetry type}/YYYY-MM-DD/{##}/` 的目錄結構中。

4. 使用 **SHIFT + ENTER** 執行程式碼。 在儲存格左邊，方括號之間出現 '\*' 即表示正在執行此儲存格中的程式碼。 當執行完成之後，'\*' 就會變成數字，而儲存格下方會顯示類似以下文字的輸出：

        Creating SparkContext as 'sc'

        ID    YARN Application ID    Kind    State    Spark UI    Driver log    Current session?
        3    application_1468969497124_0001    spark    idle    Link    Link    ✔

        Creating HiveContext as 'sqlContext'
        SparkContext and HiveContext created. Executing user code ...
5. 新的儲存格會建立在第一個儲存格之下。 在新的儲存格中輸入下列文字。 將 `CONTAINER` 和 `STORAGEACCOUNT` 取代為 Azure 儲存體帳戶名稱和包含 Application Insights 記錄的 Blob 容器名稱。

   ```scala
   %%bash
   hdfs dfs -ls wasb://CONTAINER@STORAGEACCOUNT.blob.core.windows.net/
   ```

    使用 **SHIFT + ENTER** 執行程此儲存格。 您會看到類似以下文字的結果：

        Found 1 items
        drwxrwxrwx   -          0 1970-01-01 00:00 wasb://appinsights@contosostore.blob.core.windows.net/contosoappinsights_2bededa61bc741fbdee6b556571a4831

    傳回的 wasb 路徑是 Application Insights 遙測資料的位置。 將儲存格中的 `hdfs dfs -ls` 這一行變更為使用傳回的 wasb 路徑，然後使用 **SHIFT + ENTER** 再執行一次儲存格。 此時，結果應該會顯示包含遙測資料的目錄。

   > [!NOTE]
   > 本節中步驟的其餘部分使用 `wasb://appinsights@contosostore.blob.core.windows.net/contosoappinsights_{ID}/Requests` 目錄。 這個目錄可能不存在，除非您的遙測資料是用於 Web 應用程式。

6. 在下一個儲存格中，輸入下列程式碼︰將 `WASB\_PATH` 取代為前一個步驟中的路徑。

   ```scala
   var jsonFiles = sc.textFile('WASB_PATH')
   val sqlContext = new org.apache.spark.sql.SQLContext(sc)
   var jsonData = sqlContext.read.json(jsonFiles)
   ```

    這個程式碼會從連續匯出程序匯出的 JSON 檔案建立資料框架。 使用 **SHIFT + ENTER** 執行此儲存格。

7. 在下一個儲存格中輸入並執行下列命令，以檢視 Spark 為 JSON 檔案建立的結構描述︰

   ```scala
   jsonData.printSchema
   ```

    每種類型的遙測結構描述皆不同。 以下範例是針對 Web 要求產生的結構描述 (資料儲存在 `Requests` 子目錄中)：

        root
        |-- context: struct (nullable = true)
        |    |-- application: struct (nullable = true)
        |    |    |-- version: string (nullable = true)
        |    |-- custom: struct (nullable = true)
        |    |    |-- dimensions: array (nullable = true)
        |    |    |    |-- element: string (containsNull = true)
        |    |    |-- metrics: array (nullable = true)
        |    |    |    |-- element: string (containsNull = true)
        |    |-- data: struct (nullable = true)
        |    |    |-- eventTime: string (nullable = true)
        |    |    |-- isSynthetic: boolean (nullable = true)
        |    |    |-- samplingRate: double (nullable = true)
        |    |    |-- syntheticSource: string (nullable = true)
        |    |-- device: struct (nullable = true)
        |    |    |-- browser: string (nullable = true)
        |    |    |-- browserVersion: string (nullable = true)
        |    |    |-- deviceModel: string (nullable = true)
        |    |    |-- deviceName: string (nullable = true)
        |    |    |-- id: string (nullable = true)
        |    |    |-- osVersion: string (nullable = true)
        |    |    |-- type: string (nullable = true)
        |    |-- location: struct (nullable = true)
        |    |    |-- city: string (nullable = true)
        |    |    |-- clientip: string (nullable = true)
        |    |    |-- continent: string (nullable = true)
        |    |    |-- country: string (nullable = true)
        |    |    |-- province: string (nullable = true)
        |    |-- operation: struct (nullable = true)
        |    |    |-- name: string (nullable = true)
        |    |-- session: struct (nullable = true)
        |    |    |-- id: string (nullable = true)
        |    |    |-- isFirst: boolean (nullable = true)
        |    |-- user: struct (nullable = true)
        |    |    |-- anonId: string (nullable = true)
        |    |    |-- isAuthenticated: boolean (nullable = true)
        |-- internal: struct (nullable = true)
        |    |-- data: struct (nullable = true)
        |    |    |-- documentVersion: string (nullable = true)
        |    |    |-- id: string (nullable = true)
        |-- request: array (nullable = true)
        |    |-- element: struct (containsNull = true)
        |    |    |-- count: long (nullable = true)
        |    |    |-- durationMetric: struct (nullable = true)
        |    |    |    |-- count: double (nullable = true)
        |    |    |    |-- max: double (nullable = true)
        |    |    |    |-- min: double (nullable = true)
        |    |    |    |-- sampledValue: double (nullable = true)
        |    |    |    |-- stdDev: double (nullable = true)
        |    |    |    |-- value: double (nullable = true)
        |    |    |-- id: string (nullable = true)
        |    |    |-- name: string (nullable = true)
        |    |    |-- responseCode: long (nullable = true)
        |    |    |-- success: boolean (nullable = true)
        |    |    |-- url: string (nullable = true)
        |    |    |-- urlData: struct (nullable = true)
        |    |    |    |-- base: string (nullable = true)
        |    |    |    |-- hashTag: string (nullable = true)
        |    |    |    |-- host: string (nullable = true)
        |    |    |    |-- protocol: string (nullable = true)

8. 使用下列命令將資料框架註冊為暫存資料表，並針對資料執行查詢︰

   ```scala
   jsonData.registerTempTable("requests")
   var city = sqlContext.sql("select context.location.city from requests where context.location.city is not null limit 10").show()
   ```

    此查詢會傳回 context.location.city 不是 null 的前 20 筆記錄的 city 資訊。

   > [!NOTE]
   > context 結構會出現在 Application Insights 記錄的所有遙測中。 您的記錄中可能不會填入 city 元素。 使用結構描述找出您可以查詢可能包含您的記錄檔資料的其他元素。
   >
   >

    此查詢會傳回類似以下文字的資訊：

        +---------+
        |     city|
        +---------+
        | Bellevue|
        |  Redmond|
        |  Seattle|
        |Charlotte|
        ...
        +---------+

## <a name="next-steps"></a>後續步驟

如需在 Azure 中使用 Spark 處理資料和服務的範例，請參閱下列文件︰

* [Spark 和 BI：在 HDInsight 中搭配使用 Spark 和 BI 工具執行互動式資料分析](apache-spark-use-bi-tools.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark，利用 HVAC 資料來分析建築物溫度](apache-spark-ipython-notebook-machine-learning.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark 來預測食品檢查結果](apache-spark-machine-learning-mllib-ipython.md)
* [使用 HDInsight 中的 Spark 進行網站記錄分析](apache-spark-custom-library-website-log-analysis.md)

如需建立和執行 Spark 應用程式的詳細資訊，請參閱下列文件︰

* [使用 Scala 來建立獨立的應用程式](apache-spark-create-standalone-application.md)
* [利用 Livy 在 Spark 叢集上遠端執行作業](apache-spark-livy-rest-interface.md)
