---
title: 使用 HDFS CLI 搭配 Azure Data Lake Storage Gen2 預覽
description: 適用於 Data Lake Storage Gen2 預覽的 HDFS CLI 簡介
services: storage
author: artemuwka
ms.service: storage
ms.topic: article
ms.date: 06/27/2018
ms.author: artek
ms.component: data-lake-storage-gen2
ms.openlocfilehash: c59331c772e140fccfefb89eef086a35837171e1
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45576970"
---
# <a name="using-the-hdfs-cli-with-data-lake-storage-gen2"></a>使用 HDFS CLI 搭配 Data Lake Storage Gen2

Azure Data Lake Storage Gen2 預覽可讓您管理及存取資料，就如同使用 [Hadoop 分散式檔案系統 (HDFS)](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html) 一樣。 無論您是連結 HDInsight 叢集，或是使用 Azure Databricks、針對儲存於 Azure Data Lake Storage Gen2 的資料執行分析，從而執行 Apache Spark 作業，都可以使用命令列介面 (CLI) 來擷取和操作所載入的資料。 本文的其餘部分將概述在 [Azure 儲存體小組在新增 Azure 儲存體總管和 Azure 入口網站的支援](https://azure.microsoft.com/roadmap/)時，您有哪些選項可以使用。

## <a name="hdfs-cli-with-hdinsight"></a>HDFS CLI 搭配 HDInsight

HDInsight 可以存取本機連接至計算節點的分散式檔案系統。 您可以使用與 HDFS 直接互動的殼層，以及 Hadoop 支援的其他檔案系統，來存取此檔案系統。 以下是常用的命令和實用資源的連結。

>[!IMPORTANT]
>HDInsight 叢集的計費起自叢集建立時，終至叢集刪除時。 計費是以每分鐘按比例計算，因此不再使用時，請一律刪除您的叢集 (了解如何[刪除叢集](../../hdinsight/hdinsight-delete-cluster.md))。 不過，即使將 HDInsight 叢集刪除，Azure Data Lake Storage Gen2 中儲存的資料仍然存在。

若要取得檔案清單或目錄：

    hdfs dfs -ls <args>
若要建立目錄：

    hdfs dfs -mkdir [-p] <paths>
若要刪除檔案或目錄：

    hdfs dfs -rm [-skipTrash] URI [URI ...]


現在讓我們利用 Linux 上的 HDInsight Hadoop 叢集作為範例。 若要使用 HDFS CLI，您必須先建立[對服務的遠端存取](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-linux-information#remote-access-to-services)。 如果您挑選 [SSH](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix)，則範例 PowerShell 程式碼會看起來像這樣：
```PowerShell
#Connect to the cluster via SSH.
ssh sshuser@clustername-ssh.azurehdinsight.net
#Execute basic HDFS commands. Display the hierarchy.
hdfs dfs -ls /
#Create a sample directory.
hdfs dfs -mkdir /samplefolder
```

您可以在 Azure 入口網站中 HDInsight 叢集刀鋒視窗的「SSH + 叢集登入」區段找到連接字串。 在叢集建立時，就已指定 SSH 認證。

如需有關 HDFS CLI 的詳細資訊，請參閱[官方文件](https://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-common/FileSystemShell.html)和 [HDFS 使用權限指南](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html)。

## <a name="hdfs-cli-with-azure-databricks"></a>HDFS CLI 搭配 Azure Databricks

Databricks 提供以 Databricks REST API 為基礎所建置的 CLI，讓您能輕鬆使用。 開放原始碼專案裝載於 [GitHub](https://github.com/databricks/databricks-cli)。 以下是常用的命令。

若要取得檔案清單或目錄：

    dbfs ls [-l]
若要建立目錄：

    dbfs mkdirs
若要刪除檔案：

    dbfs rm [-r]

與 Databricks 互動的另一種方法是筆記本。 雖然筆記本有主要語言，但您可以在資料格的開頭指定語言魔術命令 %language 來混合語言。 具體而言，%sh 可讓您在筆記本中執行殼層程式碼，與本文中稍早的 HDInsight 範例很像。

若要取得檔案清單或目錄：

    %sh ls <args>
若要建立目錄：

    %sh mkdir [-p] <paths>
若要刪除檔案或目錄：

    %sh rm [-skipTrash] URI [URI ...]

啟動 Azure Databricks 中的 Spark 叢集之後，您將建立新的筆記本。 範例筆記本指令碼會看起來像這樣：

    #Execute basic HDFS commands invoking the shell. Display the hierarchy.
    %sh ls /
    #Create a sample directory.
    %sh mkdir /samplefolder

如需關於 Databricks CLI 的詳細資訊，請參閱[官方文件](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html)。 如需有關筆記本的詳細資訊，請參閱文件的[筆記本](https://docs.azuredatabricks.net/user-guide/notebooks/index.html)小節。

## <a name="next-steps"></a>後續步驟

- [建立搭配 Azure Data Lake Storage Gen2 的 HDInsight 叢集](./quickstart-create-connect-hdi-cluster.md)
- [在 Azure Databricks 中使用具有 Azure Data Lake Storage Gen2 功能的帳戶](./quickstart-create-databricks-account.md) 