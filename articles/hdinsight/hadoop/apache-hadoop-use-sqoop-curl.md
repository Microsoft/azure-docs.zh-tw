---
title: 在 HDInsight 中搭配使用 Hadoop Sqoop 與 Curl - Azure
description: 了解如何使用 Curl 從遠端提交 Sqoop 工作到 HDInsight。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: 44b151cdd66bdcb5bb2ec005298f58186d98e97a
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43050063"
---
# <a name="run-sqoop-jobs-with-hadoop-in-hdinsight-with-curl"></a>使用 Curl 在 HDInsight 中以 Hadoop 執行 Sqoop 作業
[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

了解如何使用 Curl 在 HDInsight 中的 Hadoop 叢集上執行 Sqoop 作業。

本文件使用 Curl 示範如何使用未經處理的 HTTP 要求來與 HDInsight 互動，以便執行、監視和擷取 Sqoop 作業的結果。 要想執行這些作業，就要使用 HDInsight 叢集所提供的 WebHCat REST API (先前稱為 Templeton)。

## <a name="prerequisites"></a>必要條件
若要完成本文中的步驟，您需要下列項目：

* 完成[在 HDInsight 上將 Sqoop 與 Hadoop 搭配使用](hdinsight-use-sqoop.md#create-cluster-and-sql-database)，以使用 HDInsight 叢集和 Azure SQL Database 設定環境。
* [Curl](http://curl.haxx.se/)。 Curl 是將資料傳送至 HDInsight 叢集或從 HDInsight 叢集傳送資料的工具。
* [jq](http://stedolan.github.io/jq/)。 jq 公用程式用來處理從 REST 要求傳回的 JSON 資料。

## <a name="submit-sqoop-jobs-by-using-curl"></a>使用 Curl 提交 Sqoop 作業
> [!NOTE]
> 在使用 Curl 或與 WebHCat 進行任何其他 REST 通訊時，您必須提供 HDInsight 叢集系統管理員的使用者名稱和密碼來驗證要求。 您也必須在用來將要求傳送至伺服器的統一資源識別項 (URI) 中使用叢集名稱。
> 
> 在本節的所有命令中，將 **USERNAME** 取代為用來驗證叢集的使用者，並將 **PASSWORD** 取代為使用者帳戶的密碼。 將 **CLUSTERNAME** 取代為您叢集的名稱。
> 
> 透過 [基本驗證](http://en.wikipedia.org/wiki/Basic_access_authentication)來保護 REST API 的安全。 您應該一律使用安全 HTTP (HTTPS) 提出要求，確保認證安全地傳送至伺服器。
> 
> 

1. 從命令列中，使用下列命令來確認您可以連線到 HDInsight 叢集：

    ```bash   
    curl -u USERNAME:PASSWORD -G https://CLUSTERNAME.azurehdinsight.net/templeton/v1/status
    ```

    您應該會收到如下所示的回應：

    ```json   
    {"status":"ok","version":"v1"}
    ```
   
    此命令中使用的參數如下：
   
   * **-u** - 用來驗證要求的使用者名稱和密碼。
   * **-G** - 指出這是 GET 要求。
     
     URL 的開頭 **https://CLUSTERNAME.azurehdinsight.net/templeton/v1** 適用於所有的要求。 路徑 **/status** 指出要求是要傳回伺服器之 WebHCat (也稱為 Templeton) 的狀態。 
2. 使用以下命令提交 Sqoop 作業：

    ```bash
    curl -u USERNAME:PASSWORD -d user.name=USERNAME -d command="export --connect jdbc:sqlserver://SQLDATABASESERVERNAME.database.windows.net;user=USERNAME@SQLDATABASESERVERNAME;password=PASSWORD;database=SQLDATABASENAME --table log4jlogs --export-dir /example/data/sample.log --input-fields-terminated-by \0x20 -m 1" -d statusdir="wasb:///example/data/sqoop/curl" https://CLUSTERNAME.azurehdinsight.net/templeton/v1/sqoop
    ```

    此命令中使用的參數如下：

    * **-d** - 因為未使用 `-G`，要求會依預設使用 POST 方法。 `-d` 可指定與要求一起傳送的資料值。

        * **user.name** - 執行命令的使用者。

        * **命令** - 要執行的 Sqoop 命令。

        * **statusdir** - 要寫入此工作狀態的目錄。

    此命令應該會傳回可用來檢查工作狀態的工作識別碼。

        ```json
        {"id":"job_1415651640909_0026"}
        ```

3. 若要檢查工作的狀態，請使用下列命令。 將 **JOBID** 取代為上一個步驟中所傳回的值。 例如，如果傳回值為 `{"id":"job_1415651640909_0026"}`，則 **JOBID** 會是 `job_1415651640909_0026`。

    ```bash
    curl -G -u USERNAME:PASSWORD -d user.name=USERNAME https://CLUSTERNAME.azurehdinsight.net/templeton/v1/jobs/JOBID | jq .status.state
    ```

    如果工作已完成，則狀態會是 [ **成功**]。
   
   > [!NOTE]
   > 此 Curl 要求會傳回含有工作資訊的 JavaScript Object Notation (JSON) 文件；jq 可用來僅擷取狀態值。
   > 
   > 
4. 工作狀態變更為 [成功] 之後，即可從 Azure Blob 儲存體擷取工作結果。 隨查詢一起傳送的 `statusdir` 參數包含輸出檔案的位置；在此案例中為 **wasb:///example/data/sqoop/curl**。 此位址會將作業輸出儲存至 HDInsight 叢集所使用之預設儲存體容器的 **example/data/sqoop/curl** 目錄中。
   
    您可以使用 Azure 入口網站存取 stderr 和 stdout Blob。  您也可以使用 Microsoft SQL Server Management Studio 檢查上傳到 log4jlogs 資料表的資料。

## <a name="limitations"></a>限制
* 大量匯出 - 使用 Linux 型 HDInsight，用來將資料匯出至 Microsoft SQL Server 或 Azure SQL Database 的 Sqoop 連接器目前不支援大量插入。
* 批次處理 - 使用 Linux 型 HDInsight，執行插入時若使用 `-batch` 參數，Sqoop 將會執行多個插入，而不是批次處理插入作業。

## <a name="summary"></a>總結
如這份文件所示，您可以使用原始 HTTP 要求來執行、監視和檢視 HDInsight 叢集上的 Sqoop 作業結果。

如需本文中使用的 REST 介面的詳細資訊，請參閱 <a href="https://sqoop.apache.org/docs/1.99.3/RESTAPI.html" target="_blank">Sqoop REST API 指南</a>。

## <a name="next-steps"></a>後續步驟
Hive 與 HDInsight 搭配使用的一般資訊：

* [在 HDInsight 上將 Sqoop 與 Hadoop 搭配使用](hdinsight-use-sqoop.md)

如需您可以在 HDInsight 上使用 Hadoop 之其他方式的詳細資訊：

* [搭配使用 Hive 與 HDInsight 上的 Hadoop](hdinsight-use-hive.md)
* [搭配 HDInsight 上的 Hadoop 使用 Pig](hdinsight-use-pig.md)
* [搭配使用 MapReduce 與 HDInsight 上的 Hadoop](hdinsight-use-mapreduce.md)

如需涉及 curl 的其他 HDInsight 文章：
 
* [使用 Azure REST API 建立 Hadoop 叢集](../hdinsight-hadoop-create-linux-clusters-curl-rest.md)
* [使用 REST 以 HDInsight 中的 Hadoop 執行 Hive 查詢](apache-hadoop-use-hive-curl.md)
* [使用 REST 搭配 HDInsight 上的 Hadoop 執行 MapReduce 作業](apache-hadoop-use-mapreduce-curl.md)
* [使用 Curl 搭配執行 Pig 工作與 HDInsight 上的 Hadoop](apache-hadoop-use-pig-curl.md)



