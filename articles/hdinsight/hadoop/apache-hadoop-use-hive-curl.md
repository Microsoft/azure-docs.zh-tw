---
title: 在 HDInsight 中搭配使用 Hadoop Hive 與 Curl - Azure
description: 了解如何使用 Curl 從遠端提交 Pig 工作到 HDInsight。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 04/23/2018
ms.author: jasonh
ms.openlocfilehash: 8a9dd45a8d5ebf506899c733107ff2fd01b08a2c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46964525"
---
# <a name="run-hive-queries-with-hadoop-in-hdinsight-using-rest"></a>使用 REST 以 HDInsight 中的 Hadoop 執行 Hive 查詢

[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

了解如何使用 WebHCat REST API 以 Azure HDInsight 叢集上的 Hadoop 執行 Hive 查詢。

## <a name="prerequisites"></a>必要條件

* HDInsight 叢集 3.4 版或更新版本上以 Linux 為基礎的 Hadoop。

  > [!IMPORTANT]
  > Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

* REST 用戶端。 本文會使用 Windows PowerShell 和 [Curl](http://curl.haxx.se/) 範例。

    > [!NOTE]
    > Azure PowerShell 提供 HDInsight 上的 Hive 專用的 Cmdlet。 如需詳細資訊，請參閱[搭配使用 Hive 與 Azure PowerShell](apache-hadoop-use-hive-powershell.md) 文件。

本文也使用 Windows PowerShell 和 [Jq](http://stedolan.github.io/jq/) 來處理從 REST 要求傳回的 JSON 資料。

## <a id="curl"></a>執行 Hive 查詢

> [!NOTE]
> 在使用 cURL 或與 WebHCat 進行任何其他 REST 通訊時，您必須提供 HDInsight 叢集系統管理員的使用者名稱和密碼來驗證要求。
>
> 透過 [基本驗證](http://en.wikipedia.org/wiki/Basic_access_authentication)來保護 REST API 的安全。 為確保認證安全地傳送至伺服器，請一律使用安全 HTTP (HTTPS) 提出要求。

1. 若要設定本文中的指令碼所使用的叢集登入，請使用下列其中一個命令：

    ```bash
    read -p "Enter your cluster login account name: " LOGIN
    ```

    ```powershell
    $creds = Get-Credential -UserName admin -Message "Enter the cluster login name and password"
    ```

2. 若要設定叢集名稱，請使用下列其中一個命令：

    ```bash
    read -p "Enter the HDInsight cluster name: " CLUSTERNAME
    ```

    ```powershell
    $clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
    ```

3. 若要確認您可以連線至 HDInsight 叢集，請使用下列其中一個命令：

    ```bash
    curl -u $LOGIN -G https://$CLUSTERNAME.azurehdinsight.net/templeton/v1/status)
    ```
    
    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/status" `
       -Credential $creds `
       -UseBasicParsing
    $resp.Content
    ```

    您應該會收到類似以下文字的回應：

    ```json
    {"status":"ok","version":"v1"}
    ```

    此命令中使用的參數如下：

    * `-u` - 用來驗證要求的使用者名稱和密碼。
    * `-G` - 指出此要求是 GET 作業。

   URL 的開頭 `https://$CLUSTERNAME.azurehdinsight.net/templeton/v1` 適用於所有的要求。 路徑 `/status` 會指出要求是要傳回伺服器之 WebHCat (也稱為 Templeton) 的狀態。 您也可以使用下列命令要求 Hive 的版本：

    ```bash
    curl -u $LOGIN -G https://$CLUSTERNAME.azurehdinsight.net/templeton/v1/version/hive
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/version/hive" `
       -Credential $creds `
       -UseBasicParsing
    $resp.Content
    ```

    此要求會傳回類似以下文字的回應：

    ```json
        {"module":"hive","version":"0.13.0.2.1.6.0-2103"}
    ```

4. 使用下列命令建立名為 **log4jLogs** 的資料表：

    ```bash
    JOBID=`curl -s -u $LOGIN -d user.name=$LOGIN -d execute="set+hive.execution.engine=tez;DROP+TABLE+log4jLogs;CREATE+EXTERNAL+TABLE+log4jLogs(t1+string,t2+string,t3+string,t4+string,t5+string,t6+string,t7+string)+ROW+FORMAT+DELIMITED+FIELDS+TERMINATED+BY+' '+STORED+AS+TEXTFILE+LOCATION+'/example/data/';SELECT+t4+AS+sev,COUNT(*)+AS+count+FROM+log4jLogs+WHERE+t4+=+'[ERROR]'+AND+INPUT__FILE__NAME+LIKE+'%25.log'+GROUP+BY+t4;" -d statusdir="/example/rest" https://$CLUSTERNAME.azurehdinsight.net/templeton/v1/hive | jq .id`
    echo $JOBID
    ```

    ```powershell
    $reqParams = @{"user.name"="admin";"execute"="set hive.execution.engine=tez;DROP TABLE log4jLogs;CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED BY ' ' STORED AS TEXTFILE LOCATION '/example/data/;SELECT t4 AS sev,COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' GROUP BY t4;";"statusdir"="/example/rest"}
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/hive" `
       -Credential $creds `
       -Body $reqParams `
       -Method POST `
       -UseBasicParsing
    $jobID = (ConvertFrom-Json $resp.Content).id
    $jobID
    ```

    此要求會使用 POST 方法，而此方法會將資料視為要求的一部分傳送至 REST API。 下列資料值會與要求一起傳送：

     * `user.name` - 執行命令的使用者。
     * `execute` - 要執行的 HiveQL 陳述式。
     * `statusdir` - 此作業的狀態要寫入的目錄。

   這些陳述式會執行下列動作：
   
   * `DROP TABLE` - 如果資料表已存在，將會被刪除。
   * `CREATE EXTERNAL TABLE` - 在 Hive 中建立新的「外部」資料表。 外部資料表只會將資料表定義儲存在 Hive 中。 資料會留在原來的位置。

     > [!NOTE]
     > 當您預期會由外部來源來更新基礎資料時，請使用外部資料表。 例如，自動化的資料上傳程序，或其他 MapReduce 作業。
     >
     > 捨棄外部資料表並 **不會** 刪除資料，只會刪除資料表定義。

   * `ROW FORMAT` - 設定資料格式的方式。 每個記錄中的欄位會以空格分隔。
   * `STORED AS TEXTFILE LOCATION` - 儲存資料、並將其儲存為文字的位置 (example/data 目錄)。
   * `SELECT` - 選取其資料行 **t4** 包含值 **[ERROR]** 的所有資料列計數。 這個陳述式會傳回值 **3**，因為有三個資料列包含此值。

     > [!NOTE]
     > 請注意，在搭配 Curl 使用時，會以 `+` 字元取代 HiveQL 陳述式之間的空格。 加上引號的值若包含空格，例如分隔符號，則不應以 `+`取代。

      此命令會傳回可用來檢查作業狀態的作業識別碼。

5. 若要檢查作業的狀態，請使用下列命令：

    ```bash
    curl -G -u $LOGIN -d user.name=$LOGIN https://$CLUSTERNAME.azurehdinsight.net/templeton/v1/jobs/$JOBID | jq .status.state
    ```

    ```powershell
    $reqParams=@{"user.name"="admin"}
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/jobs/$jobID" `
       -Credential $creds `
       -Body $reqParams `
       -UseBasicParsing
    # ConvertFrom-JSON can't handle duplicate names with different case
    # So change one to prevent the error
    $fixDup=$resp.Content.Replace("jobID","job_ID")
    (ConvertFrom-Json $fixDup).status.state
    ```

    如果作業已完成，則狀態會是 **SUCCEEDED**。

6. 工作狀態變更為 [成功] 之後，即可從 Azure Blob 儲存體擷取工作結果。 與查詢一起傳遞的 `statusdir` 參數包含輸出檔案的位置；在此案例中為 `/example/rest`。 此位址會將輸出儲存在叢集預設儲存體的 `example/curl` 目錄中。

    您可以使用 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) 列出並下載這些檔案。 如需搭配 Azure 儲存體使用 Azure CLI 的詳細資訊，請參閱[搭配 Azure 儲存體使用 Azure CLI](https://docs.microsoft.com/azure/storage/storage-azure-cli#create-and-manage-blobs) 文件。

## <a id="nextsteps"></a>後續步驟

Hive 與 HDInsight 搭配使用的一般資訊：

* [搭配使用 Hive 與 HDInsight 上的 Hadoop](hdinsight-use-hive.md)

如需您可以在 HDInsight 上使用 Hadoop 之其他方式的詳細資訊：

* [搭配使用 Pig 與 HDInsight 上的 Hadoop](hdinsight-use-pig.md)
* [搭配 HDInsight 上的 Hadoop 使用 MapReduce](hdinsight-use-mapreduce.md)

如果您搭配使用 Tez 和 Hive，請參閱下列文件所提供的偵錯資訊：

* [在以 Linux 為基礎的 HDInsight 上使用 Ambari Tez 檢視](../hdinsight-debug-ambari-tez-view.md)

如需本文件中使用之 REST API 的詳細資訊，請參閱 [WebHCat 參照 (英文)](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference) 文件。

[hdinsight-sdk-documentation]: http://msdnstage.redmond.corp.microsoft.com/library/dn479185.aspx

[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[apache-tez]: http://tez.apache.org
[apache-hive]: http://hive.apache.org/
[apache-log4j]: http://en.wikipedia.org/wiki/Log4j
[hive-on-tez-wiki]: https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez
[import-to-excel]: http://azure.microsoft.com/documentation/articles/hdinsight-connect-excel-power-query/


[hdinsight-use-oozie]: hdinsight-use-oozie.md
[hdinsight-analyze-flight-data]: hdinsight-analyze-flight-delay-data.md




[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-submit-jobs]:submit-apache-hadoop-jobs-programmatically.md
[hdinsight-upload-data]: hdinsight-upload-data.md

[powershell-here-strings]: http://technet.microsoft.com/library/ee692792.aspx


