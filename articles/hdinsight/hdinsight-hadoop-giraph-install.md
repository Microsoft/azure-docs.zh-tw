---
title: 在 HDInsight 的 Hadoop 叢集上安裝和使用 Giraph - Azure
description: 了解如何使用 Giraph 自訂 HDInsight 叢集，以及如何使用 Giraph。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 02/05/2016
ms.author: jasonh
ROBOTS: NOINDEX
ms.openlocfilehash: 0cf866ee472717c6021922927713ef0bf8817271
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43093239"
---
# <a name="install-and-use-giraph-on-windows-based-hdinsight-clusters"></a>在 Windows 型 HDInsight 叢集上安裝和使用 Giraph

了解如何使用指令碼動作來自訂以 Windows 為基礎的 HDInsight 叢集，以及如何使用 Giraph 來處理大型圖形。 如需搭配以 Linux 為基礎的叢集使用 Giraph 的詳細資訊，請參閱 [在 HDInsight Hadoop 叢集上安裝 Giraph (Linux)](hdinsight-hadoop-giraph-install-linux.md)。

> [!IMPORTANT]
> 本文件的步驟只適用於 Windows HDInsight 叢集。 Windows 上的 HDInsight 只提供低於 HDInsight 3.4 的版本。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。 如需有關如何在 Linux 型 HDInsight 叢集上安裝 Giraph 的詳細資訊，請參閱[在 HDInsight Hadoop 叢集上安裝 Giraph (Linux)](hdinsight-hadoop-giraph-install-linux.md)。


您也可以使用「指令碼動作」 ，在 Azure HDInsight 的任一類型的叢集 (Hadoop、Storm、HBase、Spark) 上安裝 Giraph。 您可以從 [https://hdiconfigactions.blob.core.windows.net/giraphconfigactionv01/giraph-installer-v01.ps1](https://hdiconfigactions.blob.core.windows.net/giraphconfigactionv01/giraph-installer-v01.ps1) 的唯讀 Azure 儲存體 blob 中，取得用來在 HDInsight 叢集上安裝 Giraph 的範例指令碼。 範例指令碼只能與 HDInsight 叢集版本 3.1 搭配使用。 如需 HDInsight 叢集版本的詳細資訊，請參閱 [HDInsight 叢集版本](hdinsight-component-versioning.md)。

**相關文章**

* [在 HDInsight Hadoop 叢集上安裝 Giraph (Linux)](hdinsight-hadoop-giraph-install-linux.md)
* [在 HDInsight 叢集中建立 Hadoop](hdinsight-provision-clusters.md)：建立 HDInsight 叢集的一般資訊。
* [使用指令碼動作來自訂 HDInsight 叢集][hdinsight-cluster-customize]：關於使用指令碼動作來自訂 HDInsight 叢集的一般資訊。
* [開發 HDInsight 的指令碼動作指令碼](hdinsight-hadoop-script-actions.md)

## <a name="what-is-giraph"></a>什麼是 Giraph？
<a href="http://giraph.apache.org/" target="_blank">Apache Giraph</a> 可讓您利用 Hadoop 執行圖形處理，且可以搭配 Azure HDInsight 一起使用。 圖形可以模型化物件之間的關聯，例如大型網路 (像是網際網路) 上的路由器之間的連線，或社交網路上的人際關係 (有時稱為社交圖形)。 圖形處理可讓您分析圖形中物件之間的關聯，例如：

* 根據目前的人際關係找出可能的朋友。
* 識別網路中兩台電腦之間的最短路線。
* 計算網頁的頁面排名。

## <a name="install-giraph-using-portal"></a>使用入口網站安裝 Giraph
1. 使用 [自訂建立] 選項，依[在 HDInsight 中建立 Hadoop 叢集](hdinsight-provision-clusters.md)中的描述開始建立叢集。
2. 在精靈的 [指令碼動作] 頁面上，按一下 [加入指令碼動作] 以提供有關指令碼動作的詳細資料，如下所示：

    ![使用指令碼動作以自訂叢集](./media/hdinsight-hadoop-giraph-install/hdi-script-action-giraph.png "使用指令碼動作以自訂叢集")

    <table border='1'>
        <tr><th>屬性</th><th>值</th></tr>
        <tr><td>名稱</td>
            <td>指定指令碼動作的名稱。 例如，<b>安裝 Giraph</b>。</td></tr>
        <tr><td>指令碼 URI</td>
            <td>指定為自訂叢集叫用的指令碼統一資源識別項 (URI)。 例如，<i>https://hdiconfigactions.blob.core.windows.net/giraphconfigactionv01/giraph-installer-v01.ps1</i></td></tr>
        <tr><td>節點類型</td>
            <td>指定執行自訂指令碼的節點。 您可以選擇 [所有節點]<b></b>、[僅限前端節點]<b></b> 或 [僅限背景工作節點]<b></b>。
        <tr><td>參數</td>
            <td>如果指令碼要求，請指定參數。 要安裝 Giraph 的指令碼不需要任何參數，因此可以讓此處空白。</td></tr>
    </table>

    您可以加入一個以上的指令碼動作，以在叢集上安裝多個元件。 加入指令碼之後，請按一下核取記號以開始建立叢集。

## <a name="use-giraph"></a>使用 Giraph
我們使用 SimpleShortestPathsComputation 範例來示範在圖形中的物件之間找出最短路徑的基本<a href = "http://people.apache.org/~edwardyoon/documents/pregel.pdf">Pregel</a> 實作。 請使用下列步驟來上傳範例資料及範例 jar，使用 SimpleShortestPathsComputation 範例執行工作，然後檢視結果。

1. 將範例資料檔案上傳至 Azure Blob 儲存體。 在本機工作站上，建立名為 **tiny_graph.txt** 的新檔案。 應該包含下列幾行：

        [0,0,[[1,1],[3,3]]]
        [1,0,[[0,1],[2,2],[3,1]]]
        [2,0,[[1,2],[4,4]]]
        [3,0,[[0,3],[1,1],[4,4]]]
        [4,0,[[3,4],[2,4]]]

    將 tiny_graph.txt 檔案上傳至 HDInsight 叢集的主要儲存體。 如需有關如何上傳資料的指示，請參閱 [在 HDInsight 上將 Hadoop 工作的資料上傳](hdinsight-upload-data.md)。

    這項資料會使用 [source_id, source_value,[[dest_id], [edge_value],...]][source\_id, source\_value,[[dest\_id], [edge\_value],...]] 格式，描述一個有向圖形中物件之間的關聯性。每一行代表 **source\_id** 物件和一或多個 **dest\_id** 物件之間的關聯性。 **edge\_value** (或權數) 可以視為 **source_id** 和 **dest\_id** 之間的連線強度或距離。

    如果使用值 (或權數) 當做物件之間的距離繪製出來，上述資料可能如下圖所示：

    ![tiny_graph.txt drawn as circles with lines of varying distance between](./media/hdinsight-hadoop-giraph-install/giraph-graph.png)
2. 執行 SimpleShortestPathsComputation 範例。 使用 tiny_graph.txt 檔案做為輸入，即可使用下列 Azure PowerShell Cmdlet 來執行此範例。

    > [!IMPORTANT]
    > 使用 Azure Service Manager 管理 HDInsight 資源的 Azure PowerShell 支援已**被取代**，並已在 2017 年 1 月 1 日移除。 本文件中的步驟會使用可與 Azure Resource Manager 搭配使用的新 HDInsight Cmdlet。
    >
    > 請遵循 [安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs) 中的步驟來安裝最新版的 Azure PowerShell。 如果您需要修改指令碼才能使用適用於 Azure Resource Manager 的新 Cmdlet，請參閱 [移轉至以 Azure Resource Manager 為基礎的開發工具 (適用於 HDInsight 叢集)](hdinsight-hadoop-development-using-azure-resource-manager.md) ，以取得詳細資訊。

    ```powershell
    $clusterName = "clustername"
    # Giraph examples jar
    $jarFile = "wasb:///example/jars/giraph-examples.jar"
    # Arguments for this job
    $jobArguments = "org.apache.giraph.examples.SimpleShortestPathsComputation",
                    "-ca", "mapred.job.tracker=headnodehost:9010",
                    "-vif", "org.apache.giraph.io.formats.JsonLongDoubleFloatDoubleVertexInputFormat",
                    "-vip", "wasb:///example/data/tiny_graph.txt",
                    "-vof", "org.apache.giraph.io.formats.IdWithValueTextOutputFormat",
                    "-op",  "wasb:///example/output/shortestpaths",
                    "-w", "2"
    # Create the definition
    $jobDefinition = New-AzureHDInsightMapReduceJobDefinition
        -JarFile $jarFile
        -ClassName "org.apache.giraph.GiraphRunner"
        -Arguments $jobArguments

    # Run the job, write output to the Azure PowerShell window
    $job = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $jobDefinition
    Write-Host "Wait for the job to complete ..." -ForegroundColor Green
    Wait-AzureHDInsightJob -Job $job
    Write-Host "STDERR"
    Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $job.JobId -StandardError
    Write-Host "Display the standard output ..." -ForegroundColor Green
    Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $job.JobId -StandardOutput
    ```

    在上述範例中，利用您已安裝 Giraph 的 HDInsight 叢集名稱取代 **clustername** 。
3. 檢視結果。 工作完成之後，結果會儲存於 **wasb:///example/out/shotestpaths** 資料夾中的兩個輸出檔案。 這些檔案稱為 **part-m-00001** 和 **part-m-00002**。 執行下列步驟來下載和檢視輸出：

    ```powershell
    $subscriptionName = "<SubscriptionName>"       # Azure subscription name
    $storageAccountName = "<StorageAccountName>"   # Azure Storage account name
    $containerName = "<ContainerName>"             # Blob storage container name

    # Select the current subscription
    Select-AzureSubscription $subscriptionName

    # Create the Storage account context object
    $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
    $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

    # Download the job output to the workstation
    Get-AzureStorageBlobContent -Container $containerName -Blob example/output/shortestpaths/part-m-00001 -Context $storageContext -Force
    Get-AzureStorageBlobContent -Container $containerName -Blob example/output/shortestpaths/part-m-00002 -Context $storageContext -Force
    ```

    這會在工作站上目前的目錄中建立 **example/output/shortestpaths** 目錄結構，並將兩個輸出檔案下載到該位置。

    使用 **Cat** Cmdlet 顯示檔案的內容：

        Cat example/output/shortestpaths/part*

    輸出應該如下所示：

        0    1.0
        4    5.0
        2    2.0
        1    0.0
        3    1.0

    SimpleShortestPathComputation 範例已刻意設計成從物件識別碼 1 開始，尋找前往其他物件的最短路徑。 因此，輸出應該會顯示 `destination_id distance`，其中 distance 是物件識別碼 1 與目標識別碼之間經過的邊緣的值 (或權數)。

    顯現為圖形後，您可以走過識別碼 1 與其他所有物件之間的最短路徑來驗證結果。 請注意，識別碼 1 和識別碼 4 之間的最短路徑為 5。 這是<span style="color:orange">識別碼 1 和 3 之間</span>加上<span style="color:red">識別碼 3 和 4 之間</span>的總距離。

    ![Drawing of objects as circles with shortest paths drawn between](./media/hdinsight-hadoop-giraph-install/giraph-graph-out.png)

## <a name="install-giraph-using-aure-powershell"></a>使用 Aure PowerShell 安裝 Giraph
請參閱[使用指令碼動作來自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster.md#call-scripts-using-azure-powershell)。  此範例示範如何使用 Azure PowerShell 安裝 Spark。 您需要自訂指令碼以使用 [https://hdiconfigactions.blob.core.windows.net/giraphconfigactionv01/giraph-installer-v01.ps1](https://hdiconfigactions.blob.core.windows.net/giraphconfigactionv01/giraph-installer-v01.ps1)。

## <a name="install-giraph-using-net-sdk"></a>使用 .NET SDK 安裝 Giraph
請參閱[使用指令碼動作來自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster.md#call-scripts-using-azure-powershell)。 此範例示範如何使用 .NET SDK 安裝 Spark。 您需要自訂指令碼以使用 [https://hdiconfigactions.blob.core.windows.net/giraphconfigactionv01/giraph-installer-v01.ps1](https://hdiconfigactions.blob.core.windows.net/giraphconfigactionv01/giraph-installer-v01.ps1)。

## <a name="see-also"></a>另請參閱
* [在 HDInsight Hadoop 叢集上安裝 Giraph (Linux)](hdinsight-hadoop-giraph-install-linux.md)
* [在 HDInsight 叢集中建立 Hadoop](hdinsight-provision-clusters.md)：建立 HDInsight 叢集的一般資訊。
* [使用指令碼動作來自訂 HDInsight 叢集][hdinsight-cluster-customize]：關於使用指令碼動作來自訂 HDInsight 叢集的一般資訊。
* [開發 HDInsight 的指令碼動作指令碼](hdinsight-hadoop-script-actions.md)
* [在 HDInsight 叢集上安裝和使用 Spark][hdinsight-install-spark]：關於安裝 Spark 的指令碼動作範例。
* [在 HDInsight 叢集上安裝 Solr](hdinsight-hadoop-solr-install.md)：有關安裝 Solr 的指令碼動作範例。

[tools]: https://github.com/Blackmist/hdinsight-tools
[aps]: http://azure.microsoft.com/documentation/articles/install-configure-powershell/

[powershell-install]: /powershell/azureps-cmdlets-docs
[hdinsight-provision]: hdinsight-provision-clusters.md
[hdinsight-install-spark]: hdinsight-hadoop-spark-install.md
[hdinsight-cluster-customize]: hdinsight-hadoop-customize-cluster.md
