---
title: 使用 Apache Spark 來分析 Azure Data Lake Store 中的資料
description: 執行 Spark 作業來分析 Azure Data Lake Store 中所儲存的資料
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/21/2018
ms.openlocfilehash: aae63b06999c0b8eafa0d42608d32a4d467a900c
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041056"
---
# <a name="use-hdinsight-spark-cluster-to-analyze-data-in-data-lake-store"></a>使用 HDInsight Spark 叢集來分析 Data Lake Store 中的資料

在本教學課程中，您可使用 HDInsight Spark 叢集提供的 Jupyter Notebook 來執行可從 Data Lake Store 帳戶讀取資料的作業。

## <a name="prerequisites"></a>必要條件

* Azure Data Lake Store 帳戶。 遵循[使用 Azure 入口網站開始使用 Azure Data Lake Store](../../data-lake-store/data-lake-store-get-started-portal.md) 的指示。

* 以 Data Lake Store 做為儲存體的 Azure HDInsight Spark 叢集。 請遵循[快速入門：在 HDInsight 中設定叢集](../../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)中的指示。

    
## <a name="prepare-the-data"></a>準備資料

> [!NOTE]
> 如果您建立具 Data Lake Store 的 HDInsight 叢集做為預設儲存體，則不需執行此步驟。 叢集建立程序會在您建立叢集時所指定的 Data Lake Store 帳戶中新增一些範例資料。 請略過[使用具有 Data Lake Store 的 HDInsight Spark 叢集](#use-an-hdinsight-spark-cluster-with-data-lake-store)一節。
>
>

如果您建立具有 Data Lake Store 的 HDInsight Spark 叢集做為額外的儲存體並以 Azure 儲存體 Blob 做為預設儲存體，您應該先將某些範例資料複製到 Data Lake Store 帳戶。 您可以使用與 HDInsight 叢集相關聯之 Azure 儲存體 Blob 中的資料範例。 您可以使用 [ADLCopy 工具](http://aka.ms/downloadadlcopy) 來執行此動作。 從連結下載並安裝此工具。

1. 開啟命令提示字元，並瀏覽至安裝 AdlCopy 的目錄，通常是 `%HOMEPATH%\Documents\adlcopy`。

2. 執行下列命令，將特定的 Blob 從來源容器複製到資料湖存放區：

        AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adls_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container>

    將位於 **/HdiSamples/HdiSamples/SensorSampleData/hvac/** 的 **HVAC.csv** 範例資料檔複製到 Azure Data Lake Store 帳戶。 程式碼片段看起來應該如下：

        AdlCopy /Source https://mydatastore.blob.core.windows.net/mysparkcluster/HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv /dest swebhdfs://mydatalakestore.azuredatalakestore.net/hvac/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ==

   > [!WARNING]
   > 確定檔案和路徑名稱的大小寫正確。
   >
   >
3. 系統會提示您輸入 Data Lake Store 帳戶所在 Azure 訂用帳戶的認證。 您會看到如下列程式碼片段的輸出：

        Initializing Copy.
        Copy Started.
        100% data copied.
        Copy Completed. 1 file copied.

    資料檔 (**HVAC.csv**) 將被複製到 Data Lake Store 帳戶中的 **/hvac** 資料夾底下。

## <a name="use-an-hdinsight-spark-cluster-with-data-lake-store"></a>使用具有 Data Lake Store 的 HDInsight Spark 叢集

1. 在 [Azure 入口網站](https://portal.azure.com/)的開始面板中，按一下您的 Spark 叢集磚 (如果您已將其釘選到開始面板)。 您也可以按一下 [瀏覽全部] > [HDInsight 叢集] 來瀏覽至您的叢集。

2. 在 Spark 叢集刀鋒視窗中按一下 [快速連結]，然後在 [叢集儀表板] 刀鋒視窗中按一下 [Jupyter Notebook]。 出現提示時，輸入叢集的系統管理員認證。

   > [!NOTE]
   > 您也可以在瀏覽器中開啟下列 URL，來連接到您的叢集的 Jupyter Notebook。 使用您叢集的名稱取代 **CLUSTERNAME** ：
   >
   > `https://CLUSTERNAME.azurehdinsight.net/jupyter`
   >
   >

3. 建立新的 Notebook。 按一下 [新增]，然後按一下 [PySpark]。

    ![建立新的 Jupyter Notebook](./media/apache-spark-use-with-data-lake-store/hdinsight-create-jupyter-notebook.png "建立新的 Jupyter Notebook")

4. 您使用 PySpark 核心建立 Notebook，因此不需要明確建立任何內容。 當您執行第一個程式碼儲存格時，系統會自動為您建立 Spark 和 Hive 內容。 首先，您可以匯入此案例所需的類型。 方法是將下列程式碼片段貼到儲存格中，然後按下 **SHIFT + ENTER**。

        from pyspark.sql.types import *

    每當您在 Jupyter 中執行作業時，網頁瀏覽器視窗標題將會顯示 Notebook 標題和 **(忙碌)** 狀態。 您也會在右上角的 **PySpark** 文字旁看到一個實心圓。 工作完成後，實心圓將變成空心圓。

     ![Jupyter Notebook 作業的狀態](./media/apache-spark-use-with-data-lake-store/hdinsight-jupyter-job-status.png "Jupyter Notebook 作業的狀態")

5. 使用您複製到 Data Lake Store 帳戶的 **HVAC.csv** 檔案，將範例資料載入到暫存資料表中。 您可以使用下列 URL 模式，存取 Data Lake Store 帳戶中的資料。

    * 如果您以 Data Lake Store 做為預設儲存體，HVAC.csv 會位於類似下列 URL 的路徑︰

            adl://<data_lake_store_name>.azuredatalakestore.net/<cluster_root>/HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv

        或者，您也可以使用縮短的格式如下︰

            adl:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv

    * 如果您以 Data Lake Store 做為額外儲存體，HVAC.csv 會位於您複製它的位置，例如︰

            adl://<data_lake_store_name>.azuredatalakestore.net/<path_to_file>

     在空白儲存格中，貼上下列程式碼範例、使用您的 Data Lake Store 帳戶名稱取代 **MYDATALAKESTORE**，然後按 **SHIFT + ENTER**。 此程式碼範例會將資料註冊到名為 **hvac**的暫存資料表。

            # Load the data. The path below assumes Data Lake Store is default storage for the Spark cluster
            hvacText = sc.textFile("adl://MYDATALAKESTORE.azuredatalakestore.net/cluster/mysparkcluster/HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

            # Create the schema
            hvacSchema = StructType([StructField("date", StringType(), False),StructField("time", StringType(), False),StructField("targettemp", IntegerType(), False),StructField("actualtemp", IntegerType(), False),StructField("buildingID", StringType(), False)])

            # Parse the data in hvacText
            hvac = hvacText.map(lambda s: s.split(",")).filter(lambda s: s[0] != "Date").map(lambda s:(str(s[0]), str(s[1]), int(s[2]), int(s[3]), str(s[6]) ))

            # Create a data frame
            hvacdf = sqlContext.createDataFrame(hvac,hvacSchema)

            # Register the data fram as a table to run queries against
            hvacdf.registerTempTable("hvac")

6. 由於您使用的是 PySpark 核心，因此現在可直接在您剛才使用 `%%sql` magic 建立的暫存資料表 **hvac** 上執行 SQL 查詢。 如需 `%%sql` magic 及 PySpark 核心提供的其他 magic 的詳細資訊，請參閱 [使用 Spark HDInsight 叢集之 Jupyter Notebook 上可用的核心](apache-spark-jupyter-notebook-kernels.md#parameters-supported-with-the-sql-magic)。

        %%sql
        SELECT buildingID, (targettemp - actualtemp) AS temp_diff, date FROM hvac WHERE date = \"6/1/13\"

7. 一旦工作順利完成後，預設會顯示下列表格式輸出。

      ![查詢結果的資料表輸出](./media/apache-spark-use-with-data-lake-store/jupyter-tabular-output.png "查詢結果的資料表輸出")

     您也可以查看其他視覺效果中的結果。 例如，相同輸出的區域圖看起來會如下所示。

     ![查詢結果的區域圖](./media/apache-spark-use-with-data-lake-store/jupyter-area-output.png "查詢結果的區域圖")

8. 應用程式執行完畢之後，您應該要關閉 Notebook 來釋放資源。 若要這樣做，請從 Notebook 的 [檔案] 功能表中，按一下 [關閉並停止]。 這樣就能夠結束並關閉 Notebook。


## <a name="next-steps"></a>後續步驟

* [建立獨立 Scala 應用程式以在 Apache Spark 叢集上執行](apache-spark-create-standalone-application.md)
* [使用適用於 IntelliJ 的 Azure 工具組中的 HDInsight 工具建立 HDInsight Spark Linux 叢集的 Spark 應用程式](apache-spark-intellij-tool-plugin.md)
* [使用適用於 Eclipse 的 Azure 工具組中的 HDInsight 工具建立 HDInsight Spark Linux 叢集的 Spark 應用程式](apache-spark-eclipse-tool-plugin.md)
