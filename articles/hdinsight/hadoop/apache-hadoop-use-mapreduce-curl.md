---
title: 搭配使用 MapReduce 和 Curl 與 HDInsight 中的 Hadoop - Azure
description: 了解如何使用 Curl 從遠端搭配執行 MapReduce 工作與 HDInsight 上的 Hadoop。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/27/2018
ms.author: jasonh
ms.openlocfilehash: 9941f442ad546155f58967fb3414eb2e7b7c3647
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46973706"
---
# <a name="run-mapreduce-jobs-with-hadoop-on-hdinsight-using-rest"></a>使用 REST 搭配 HDInsight 上的 Hadoop 執行 MapReduce 作業

學習如何使用 WebHCat REST API 在 HDInsight 叢集的 Hadoop 上執行 MapReduce 作業。 Curl 用來示範如何使用原始 HTTP 要求與 HDInsight 互動，以執行 MapReduce 工作。

> [!NOTE]
> 如果您已熟悉使用以 Linux 為基礎的 Hadoop 伺服器，但剛接觸 HDInsight，請參閱[在以 Linux 為基礎的 HDInsight 上安裝 Hadoop 的須知事項](../hdinsight-hadoop-linux-information.md)文件。


## <a id="prereq"></a>必要條件

* 一個 Hadoop on HDInsight 叢集
* Windows PowerShell 或 [Curl](http://curl.haxx.se/) 和 [jq](http://stedolan.github.io/jq/)

## <a id="curl"></a>執行 MapReduce 作業

> [!NOTE]
> 在使用 Curl 或與 WebHCat 進行任何其他 REST 通訊時，您必須提供 HDInsight 叢集管理員使用者名稱和密碼來驗證要求。 您必須使用叢集名稱，作為用來將要求傳送至伺服器之 URI 的一部分。
>
> 使用 [基本存取驗證](http://en.wikipedia.org/wiki/Basic_access_authentication)來保護 REST API 的安全。 您應該一律使用 HTTPS 提出要求，確保認證安全地傳送至伺服器。

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

3. 從命令列中，使用下列命令來確認您可以連線到 HDInsight 叢集：

    ```bash
    curl -u $LOGIN -G https://$CLUSTERNAME.azurehdinsight.net/templeton/v1/status
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clustername.azurehdinsight.net/templeton/v1/status" `
        -Credential $creds `
        -UseBasicParsing
    $resp.Content
    ```

    您應該會收到類似以下 JSON 的回應：

        {"status":"ok","version":"v1"}

    此命令中使用的參數如下：

   * **-u**：指出用來驗證要求的使用者名稱和密碼
   * **-G**：指出此作業是 GET 要求

   URI 的開頭 **https://CLUSTERNAME.azurehdinsight.net/templeton/v1** 適用於所有要求。

4. 若要提交 MapReduce 工作，請使用下列命令：

    ```bash
    JOBID=`curl -u $LOGIN -d user.name=$LOGIN -d jar=/example/jars/hadoop-mapreduce-examples.jar -d class=wordcount -d arg=/example/data/gutenberg/davinci.txt -d arg=/example/data/output https://$CLUSTERNAME.azurehdinsight.net/templeton/v1/mapreduce/jar | jq .id`
    echo $JOBID
    ```

    ```powershell
    $reqParams = @{}
    $reqParams."user.name" = "admin"
    $reqParams.jar = "/example/jars/hadoop-mapreduce-examples.jar"
    $reqParams.class = "wordcount"
    $reqParams.arg = @()
    $reqParams.arg += "/example/data/gutenberg/davinci.txt"
    $reqparams.arg += "/example/data/output"
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/mapreduce/jar" `
       -Credential $creds `
       -Body $reqParams `
       -Method POST `
       -UseBasicParsing
    $jobID = (ConvertFrom-Json $resp.Content).id
    $jobID
    ```

    URI 結尾 (/mapreduce/jar) 會告訴 WebHCat，此要求會從 jar 檔案中的類別啟動 MapReduce 作業。 此命令中使用的參數如下：

   * **-d**：未使用 `-G`，因此要求會依預設使用 POST 方法。 `-d` 可指定與要求一起傳送的資料值。
    * **user.name**：執行命令的使用者
    * **jar**：包含要執行之類別的 jar 檔案位置
    * **class**：包含 MapReduce 邏輯的類別
    * **arg**︰要傳遞到 MapReduce 作業的引數。 在此案例中，是用於輸出的輸入文字檔和目錄

   此命令應該會傳回可用來檢查工作狀態的工作識別碼：

       job_1415651640909_0026

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

    如果作業已完成，傳回的狀態會是 `SUCCEEDED`。

   > [!NOTE]
   > 此 Curl 要求會傳回含有作業資訊的 JSON 文件。 jq 是用來只擷取狀態值。

6. 當作業狀態變更為 `SUCCEEDED` 之後，您就可以從 Azure Blob 儲存體擷取作業結果。 隨查詢一起傳遞的 `statusdir` 參數包含輸出檔案的位置。 在此範例中，位置是 `/example/curl`。 此位址會將作業的輸出儲存在叢集預設儲存體的 `/example/curl` 中。

您可以使用 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) 列出並下載這些檔案。 如需從 Azure CLI 使用 Blob 的詳細資訊，請參閱[搭配 Azure 儲存體使用 Azure CLI](../../storage/common/storage-azure-cli.md#create-and-manage-blobs) 文件。

## <a id="nextsteps"></a>後續步驟

如需 HDInsight 中 MapReduce 工作的一般資訊：

* [搭配使用 MapReduce 與 HDInsight 上的 Hadoop](hdinsight-use-mapreduce.md)

如需您可以在 HDInsight 上使用 Hadoop 之其他方式的詳細資訊：

* [搭配使用 Hive 與 HDInsight 上的 Hadoop](hdinsight-use-hive.md)
* [搭配 HDInsight 上的 Hadoop 使用 Pig](hdinsight-use-pig.md)

如需本文中使用的 REST 介面的詳細資訊，請參閱 [WebHCat 參照](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference)。
