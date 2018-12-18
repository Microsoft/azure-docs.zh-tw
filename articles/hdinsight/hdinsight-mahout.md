---
title: 從 PowerShell 使用 Mahout HDInsight 產生推薦 - Azure
description: 了解如何從您的用戶端上執行的 PowerShell 指令碼搭配 HDInsight (Hadoop) 使用 Apache Mahout 機器學習庫來產生電影推薦。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 04/23/2018
ms.author: jasonh
ms.openlocfilehash: 66bcddf478a717c13ff450bc48e52be0aaf7a6ef
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43108728"
---
# <a name="generate-movie-recommendations-by-using-apache-mahout-with-hadoop-in-hdinsight-powershell"></a>透過在 HDInsight 上將 Apache Mahout 與 Hadoop 搭配使用來產生電影推薦 (PowerShell)

[!INCLUDE [mahout-selector](../../includes/hdinsight-selector-mahout.md)]

了解如何使用搭配 Azure HDInsight 的 [Apache Mahout](http://mahout.apache.org) 機器學習庫產生電影推薦。 本文件中的範例使用 Azure PowerShell 來執行 Mahout 作業。

## <a name="prerequisites"></a>必要條件

* 以 Linux 為基礎的 HDInsight 叢集。 如需有關建立叢集的資訊，請參閱[開始在 HDInsight 中使用以 Linux 為基礎的 Hadoop][getstarted]。

    > [!IMPORTANT]
    > Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

* [Azure PowerShell](/powershell/azure/overview)

## <a name="recommendations"></a>使用 Azure PowerShell 產生推薦

> [!WARNING]
> 本節中的作業是使用 Azure PowerShell 運作。 Mahout 提供的許多類別目前無法搭配 Azure PowerShell 使用。 如需無法搭配 Azure PowerShell 使用的類別清單，請參閱 [疑難排解](#troubleshooting) 一節。
>
> 如需使用 SSH 連線到 HDInsight 並在叢集上直接執行 Mahout 範例的範例，請參閱[使用 Mahout 和 HDInsight (SSH) 來產生電影推薦](hadoop/apache-hadoop-mahout-linux-mac.md)。

Mahout 提供的其中一項功能是推薦引擎。 這個引擎接受 `userID``itemId` 和 `prefValue` (使用者偏好的項目) 格式的資料。 Mahout 會使用資料以偏好的類似項目判斷使用者，並以此做出推薦。

下列範例是簡化的逐步解說，說明建議程序如何運作：

* **共生**：Joe、Alice 和 Bob 都喜歡 *《星際大戰》*、*《帝國大反擊》* 和 *《絕地大反攻》*。 Mahout 將判斷喜歡上述任何一部電影的使用者，也會喜歡另外兩部電影。

* **共生**：Bob 和 Alice 同時也喜歡 *《威脅潛伏》*、*《複製人全面進攻》* 和 *《西斯大帝的復仇》*。 Mahout 將判斷喜歡前三部電影的使用者，也會喜歡這些電影。

* **相似性推薦**：因為 Joe 喜歡前三部電影，Mahout 會查看具有相似偏好的其他使用者所喜歡但 Joe 還沒看過 (喜歡/評價) 的電影。 在此情況下，Mahout 將會推薦 *《威脅潛伏》*、*《複製人全面進攻》* 和 *《西斯大帝的復仇》*。

### <a name="understanding-the-data"></a>了解資料

[GroupLens 研究][movielens]提供 Mahout 相容格式的電影評分資料。 您可在位於 `/HdiSamples/HdiSamples/MahoutMovieData` 的叢集預設儲存體取得這份資料。

有兩份檔案：`moviedb.txt` (電影相關資訊) 和 `user-ratings.txt`。 `user-ratings.txt` 檔案是用於分析期間。 `moviedb.txt` 檔案是在顯示分析結果時，用來提供易懂的文字。

user-ratings.txt 內包含的資料具有 `userID`、`movieID`、`userRating` 和 `timestamp` 結構，可說明每位使用者對於影片的評價為何。 以下是資料範例：

    196    242    3    881250949
    186    302    3    891717742
    22     377    1    878887116
    244    51     2    880606923
    166    346    1    886397596

### <a name="run-the-job"></a>執行工作

使用下列 Windows PowerShell 指令碼執行工作，以透過 Mahout 推薦引擎來處理影片資料：

> [!NOTE]
> 這個檔案會提示您輸入用來連接到 HDInsight 叢集並執行作業的資訊。 可能需要幾分鐘的時間來完成作業，並且下載 output.txt 檔案。

[!code-powershell[main](../../powershell_scripts/hdinsight/mahout/use-mahout.ps1?range=5-98)]

> [!NOTE]
> Mahout 工作不會移除處理工作時所建立的暫存資料。 範例工作中所指定的 `--tempDir` 參數會將暫存檔隔離到特定的目錄。

Mahout 工作不會將輸出傳回 STDOUT。 相反地，其會將該輸出儲存在指定的輸出目錄 **part-r-00000**中。 指令碼會下載這個檔案到您工作站上目前目錄的 **output.txt** 檔。

下列文字是此檔案內容的範例：

    1    [234:5.0,347:5.0,237:5.0,47:5.0,282:5.0,275:5.0,88:5.0,515:5.0,514:5.0,121:5.0]
    2    [282:5.0,210:5.0,237:5.0,234:5.0,347:5.0,121:5.0,258:5.0,515:5.0,462:5.0,79:5.0]
    3    [284:5.0,285:4.828125,508:4.7543354,845:4.75,319:4.705128,124:4.7045455,150:4.6938777,311:4.6769233,248:4.65625,272:4.649266]
    4    [690:5.0,12:5.0,234:5.0,275:5.0,121:5.0,255:5.0,237:5.0,895:5.0,282:5.0,117:5.0]

第一欄是 `userID`。 '[' 和 ']' 中包含的值是 `movieId`:`recommendationScore`。

指令碼也會下載使輸出格式更具可讀性所需的 `moviedb.txt` 和 `user-ratings.txt` 檔案。

### <a name="view-the-output"></a>檢視輸出

雖然產生的輸出可以在應用程式中使用，但使用者不容易判讀。 來自伺服器的 `moviedb.txt`可用來解決電影名稱的 `movieId`。 使用下列 PowerShell 指令碼來顯示建議的影片名稱︰

[!code-powershell[main](../../powershell_scripts/hdinsight/mahout/use-mahout.ps1?range=106-180)]

使用下列命令，以易懂的格式來顯示建議： 

```powershell
.\show-recommendation.ps1 -userId 4 -userDataFile .\user-ratings.txt -movieFile .\moviedb.txt -recommendationFile .\output.txt
```

輸出大致如下：

    Reading movies descriptions
    Reading rated movies
    Reading recommendations
    Rated movies
    ---------------------------
    Movie                                    Rating
    -----                                    ------
    Devil's Own, The (1997)                  1
    Alien: Resurrection (1997)               3
    187 (1997)                               2
    (lines ommitted)

    ---------------------------
    Recommended movies
    ---------------------------

    Movie                                    Score
    -----                                    -----
    Good Will Hunting (1997)                 4.6504064
    Swingers (1996)                          4.6862745
    Wings of the Dove, The (1997)            4.6666665
    People vs. Larry Flynt, The (1996)       4.834559
    Everyone Says I Love You (1996)          4.707071
    Secrets & Lies (1996)                    4.818182
    That Thing You Do! (1996)                4.75
    Grosse Pointe Blank (1997)               4.8235292
    Donnie Brasco (1997)                     4.6792455
    Lone Star (1996)                         4.7099237

## <a name="troubleshooting"></a>疑難排解

### <a name="cannot-overwrite-files"></a>無法覆寫檔案

Mahout 工作不會清除在處理期間所建立的暫存檔。 此外，作業也不會覆寫現有的輸出檔。

為了避免執行 Mahout 作業時發生錯誤，請在每次執行之前刪除暫存檔和輸出檔。 若要移除本文件中先前指令碼所建立的檔案，請使用下列 PowerShell 指令碼︰

```powershell
# Login to your Azure subscription
# Is there an active Azure subscription?
$sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
if(-not($sub))
{
    Connect-AzureRmAccount
}

# Get cluster info
$clusterName = Read-Host -Prompt "Enter the HDInsight cluster name"
$creds=Get-Credential -Message "Enter the login for the cluster"

#Get the cluster info so we can get the resource group, storage, etc.
$clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
$resourceGroup = $clusterInfo.ResourceGroup
$storageAccountName = $clusterInfo.DefaultStorageAccount.split('.')[0]
$container = $clusterInfo.DefaultStorageContainer
$storageAccountKey = (Get-AzureRmStorageAccountKey `
    -Name $storageAccountName `
-ResourceGroupName $resourceGroup)[0].Value

#Create a storage context and upload the file
$context = New-AzureStorageContext `
    -StorageAccountName $storageAccountName `
    -StorageAccountKey $storageAccountKey

#Azure PowerShell can't delete blobs using wildcard,
#so have to get a list and delete one at a time
# Start with the output
$blobs = Get-AzureStorageBlob -Container $container -Context $context -Prefix "example/out"
foreach($blob in $blobs)
{
    Remove-AzureStorageBlob -Blob $blob.Name -Container $container -context $context
}
# Next the temp files
$blobs = Get-AzureStorageBlob -Container $container -Context $context -Prefix "example/temp"
foreach($blob in $blobs)
{
    Remove-AzureStorageBlob -Blob $blob.Name -Container $container -context $context
}
```

### <a name="nopowershell"></a>不適用於 Azure PowerShell 的類別

當從 Windows PowerShell 中使用的 Mahout 作業利用到下列類別時，會傳回各種錯誤訊息：

* org.apache.mahout.utils.clustering.ClusterDumper
* org.apache.mahout.utils.SequenceFileDumper
* org.apache.mahout.utils.vectors.lucene.Driver
* org.apache.mahout.utils.vectors.arff.Driver
* org.apache.mahout.text.WikipediaToSequenceFile
* org.apache.mahout.clustering.streaming.tools.ResplitSequenceFiles
* org.apache.mahout.clustering.streaming.tools.ClusterQualitySummarizer
* org.apache.mahout.classifier.sgd.TrainLogistic
* org.apache.mahout.classifier.sgd.RunLogistic
* org.apache.mahout.classifier.sgd.TrainAdaptiveLogistic
* org.apache.mahout.classifier.sgd.ValidateAdaptiveLogistic
* org.apache.mahout.classifier.sgd.RunAdaptiveLogistic
* org.apache.mahout.classifier.sequencelearning.hmm.BaumWelchTrainer
* org.apache.mahout.classifier.sequencelearning.hmm.ViterbiEvaluator
* org.apache.mahout.classifier.sequencelearning.hmm.RandomSequenceGenerator
* org.apache.mahout.classifier.df.tools.Describe

若要執行用到這些類別的作業，請使用 SSH 連接至 HDInsight 叢集，然後從命令列執行作業。 如需使用 SSH 來執行 Mahout 作業的範例，請參閱[使用 Mahout 和 HDInsight (SSH) 來產生電影推薦](hadoop/apache-hadoop-mahout-linux-mac.md)。

## <a name="next-steps"></a>後續步驟

您現在已了解如何使用 Mahout，請繼續探索在 HDInsight 上使用資料的其他方法：

* [搭配 HDInsight 使用 Hive](hadoop/hdinsight-use-hive.md)
* [搭配 HDInsight 使用 Pig](hadoop/hdinsight-use-pig.md)
* [搭配 HDInsight 使用 MapReduce](hadoop/hdinsight-use-mapreduce.md)

[build]: http://mahout.apache.org/developers/buildingmahout.html
[aps]: /powershell/azureps-cmdlets-docs
[movielens]: http://grouplens.org/datasets/movielens/
[100k]: http://files.grouplens.org/datasets/movielens/ml-100k.zip
[getstarted]:hadoop/apache-hadoop-linux-tutorial-get-started.md
[upload]: hdinsight-upload-data.md
[ml]: http://en.wikipedia.org/wiki/Machine_learning
[forest]: http://en.wikipedia.org/wiki/Random_forest
[enableremote]: ./media/hdinsight-mahout/enableremote.png
[connect]: ./media/hdinsight-mahout/connect.png
[hadoopcli]: ./media/hdinsight-mahout/hadoopcli.png
[tools]: https://github.com/Blackmist/hdinsight-tools
